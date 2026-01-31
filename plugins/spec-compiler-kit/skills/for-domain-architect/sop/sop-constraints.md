# 约束定义 SOP（第三部分）

> **跨端一致性**：本部分产出的约束适用于所有端（后端/iOS/前端）。
> 各端实现时将伪代码翻译为对应语言，但约束语义必须保持一致。

## 目标

将战术设计中的不变量转化为可执行的验证逻辑，确保约束可以被代码检查。

## 核心原则

- 使用统一的伪代码/DSL 定义约束，不绑定具体编程语言
- 每个约束必须满足不变量三测试：可执行、可测试、可解释
- 定义各端实现位置，确保约束在各端正确执行

---

## 约束分类体系

| 类型 | 前缀 | 说明 | 执行时机 |
|------|------|------|----------|
| 结构约束 | STR- | 数据结构完整性 | 创建/更新时 |
| 业务约束 | BIZ- | 业务规则正确性 | 操作时 |
| 状态约束 | STA- | 状态转移合法性 | 状态变更时 |
| 版本约束 | VER- | 版本不可变性 | 修改时 |
| 可解释性约束 | EXP- | 结果可追溯性 | 生成报告时 |

---

## 步骤 1：定义状态空间

### 1.1 状态定义表

为每个实体定义完整的状态空间：

```
{实体名称}状态：
- S0: {状态名}（Created）
- S1: {状态名}（Paid）
- S2: {状态名}（Shipped）
- S3: {状态名}（Completed）
```

**状态定义原则**：
- 状态必须互斥（同一时刻只能处于一个状态）
- 状态必须完备（覆盖所有可能情况）
- 使用状态码 + 状态名的格式

### 1.2 状态设计验证

- [ ] 每个实体至少有 2 个状态（初始状态 + 至少一个转移状态）
- [ ] 状态间无重叠（互斥性）
- [ ] 状态覆盖所有可能情况（完备性）
- [ ] 无孤岛状态（每个状态可达可离）

---

## 步骤 2：定义状态转移

### 2.1 状态转移表

| 当前状态 | 触发行为 | 行为类型 | 目标状态 | 约束条件 |
|---------|---------|---------|---------|---------|
| S0 | {行为} | 用户/系统 | S1 | {条件} |
| S1 | {行为} | 用户/系统 | S2 | {条件} |

### 2.2 状态转移伪代码

```
ALLOWED_TRANSITIONS = {
    # (当前状态, 事件) -> (下一状态, [约束列表])
    ("S0", "event_a"): ("S1", [INV-01, INV-02]),
    ("S1", "event_b"): ("S2", [INV-01]),
}

FUNCTION validate_state_transition(current_state, event, next_state) -> bool
    key = (current_state, event)
    ASSERT key IN ALLOWED_TRANSITIONS
    ON_VIOLATION: THROW "STA-01: 未定义的状态转移"

    expected_state, constraints = ALLOWED_TRANSITIONS[key]
    ASSERT next_state == expected_state
    ON_VIOLATION: THROW "STA-02: 状态转移目标错误"

    FOR EACH constraint IN constraints
        constraint.validate()
    END FOR

    RETURN true
END FUNCTION
```

### 2.3 状态转移验证

- [ ] 每个状态转移都有明确的触发行为
- [ ] 每个状态转移都标注了行为类型（用户/系统）
- [ ] 每个状态转移都定义了约束条件
- [ ] 状态转移表无遗漏

---

## 步骤 3：定义不变量

### 3.1 不变量识别

从战术设计中提取不变量，或从业务规则中推导不变量。

**不变量来源**：
- 聚合设计中的实体收敛原则
- 业务规则的约束条件
- 数据完整性要求
- 版本控制需求
- 可解释性需求

### 3.2 不变量三测试

每个不变量必须通过以下三个测试：

#### ✅ 测试 1：可执行

能否写成 assert 语句？

```
# 自然语言描述
INV-1: 只有 M1 状态才能发放点券

# 转换为 assert
assert membership.state == "M1" or coupon_grant_amount == 0
```

#### ✅ 测试 2：可测试

能否设计用例验证它？

```
Case: 验证 INV-1
Given: 会员状态 = M0
When: 触发刷新点券
Then: 点券增加量 = 0
```

#### ✅ 测试 3：可解释

能否用自然语言解释为什么它必须成立？

> INV-1 的原因：点券是会员权益，非会员不应获得，否则违反商业规则。

### 3.3 不变量定义模板

```
### INV-XX: {约束名称}

**约束描述**：{一句话描述}

**理论依据**：{理论来源}

**伪代码**：
```
FUNCTION validate_{constraint_name}(entity) -> bool
    ASSERT {条件表达式}
    ON_VIOLATION: THROW "INV-XX: {错误信息}"
    RETURN true
END FUNCTION
```

**各端实现位置**：
| 端 | 层级 | 文件 | 方法 |
|----|------|------|------|
| 后端 | Domain | XxxDomain.java | validateXxx() |
| iOS | Service | XxxService.swift | validateXxx() |
| 前端 | Service | xxxService.ts | validateXxx() |
```

### 3.4 不变量示例

#### INV-01: 订单金额计算正确性

**约束描述**：订单总金额必须等于所有订单项金额之和

**理论依据**：数据完整性原则

**伪代码**：
```
FUNCTION validate_order_amount(order) -> bool
    calculated_amount = SUM(item.price * item.quantity FOR item IN order.items)
    ASSERT order.total_amount == calculated_amount
    ON_VIOLATION: THROW "INV-01: 订单金额计算错误"
    RETURN true
END FUNCTION
```

**各端实现位置**：
| 端 | 层级 | 文件 | 方法 |
|----|------|------|------|
| 后端 | Domain | OrderDomain.java | validateAmount() |
| iOS | Service | OrderService.swift | validateAmount() |
| 前端 | Service | orderService.ts | validateAmount() |

---

## 步骤 4：定义禁止态

### 4.1 禁止态识别

明确系统不允许进入的状态或状态转移。

**禁止态来源**：
- 状态转移表中未定义的转移
- 业务规则明确禁止的情况
- 数据一致性不能保证的情况

### 4.2 禁止态定义

```
### 禁止态-1: {描述}

**禁止原因**：{原因说明}

**触发场景**：{场景描述}
```

### 4.3 禁止态伪代码

```
PROHIBITED_TRANSITIONS = {
    # (当前状态, 目标状态) -> "禁止原因"
    ("S2", "S0"): "已发货订单不能回退到待支付",
}

FUNCTION validate_not_prohibited(current_state, next_state) -> bool
    key = (current_state, next_state)
    ASSERT key NOT IN PROHIBITED_TRANSITIONS
    ON_VIOLATION: THROW "STA-03: 禁止的状态转移, reason={PROHIBITED_TRANSITIONS[key]}"
    RETURN true
END FUNCTION
```

---

## 步骤 5：定义约束执行时机

### 5.1 执行时机定义

| 约束ID | 执行时机 | 执行位置 | 失败处理 |
|-------|---------|---------|---------|
| STR-01 | 实体创建时 | Domain 层 | 抛出异常，拒绝操作 |
| BIZ-01 | 业务操作前 | Application 层 | 返回错误信息 |
| STA-01 | 状态变更时 | Domain 层 | 抛出异常，回滚状态 |

### 5.2 执行时机选择原则

| 约束类型 | 推荐执行时机 | 推荐执行位置 |
|---------|-------------|-------------|
| 结构约束 | 实体创建/更新时 | Domain 层 |
| 业务约束 | 业务操作前 | Application 层 |
| 状态约束 | 状态变更时 | Domain 层 |
| 版本约束 | 修改时 | Domain 层 |

---

## 步骤 6：定义约束违反处理策略

### 6.1 约束优先级

| 优先级 | 描述 | 处理策略 | 示例 |
|-------|------|----------|------|
| **P0** | 数据一致性破坏 | 立即抛异常，回滚事务 | 库存为负 |
| **P1** | 业务规则违反 | 阻止操作，返回错误 | 非会员发点券 |
| **P2** | 数据完整性问题 | 记录警告，允许操作 | 非必填字段为空 |
| **P3** | 可解释性问题 | 记录日志，人工审核 | 计算结果异常 |

### 6.2 处理策略表

| 约束ID | 优先级 | 违反处理 | 恢复方式 |
|-------|--------|---------|---------|
| INV-01 | P0 | 抛出异常，回滚事务 | 修复输入后重试 |
| INV-02 | P1 | 返回错误信息 | 调整参数后重试 |

---

## 闸口检查

约束定义完成后，必须通过以下检查：

- [ ] 每个不变量至少对应一个约束定义
- [ ] 每个约束都有伪代码定义（可执行）
- [ ] 每个约束都标注了各端实现位置
- [ ] 约束覆盖结构、业务、状态三类
- [ ] P0 级别约束有明确的回滚机制
- [ ] 状态转移表完整
- [ ] 禁止态已全部定义
- [ ] 约束执行时机已明确

---

## 自我验证

在进入下一阶段前，确认：

- [ ] 伪代码语法正确，无歧义
- [ ] 各端实现位置已明确
- [ ] 状态转移表完整
- [ ] 禁止态已定义
- [ ] 无平台特定语法（约束定义不包含任何平台特定的语法）
- [ ] 每个不变量都通过了三测试（可执行、可测试、可解释）

---

## 常见错误

### ❌ 错误 1：约束描述模糊

```
错误："一般情况下用户应该..."
正确："用户必须..."
```

### ❌ 错误 2：使用平台特定语法

```
错误：if (membership.state == "M1") { throw new Error(...) }
正确：ASSERT membership.state == "M1"
      ON_VIOLATION: THROW "..."
```

### ❌ 错误 3：不变量不可执行

```
错误："用户应该填写完整的个人信息"
正确："用户创建时，姓名和邮箱不能为空"
      ASSERT user.name != null AND user.email != null
```

---

## 参考示例

完整的约束定义示例见《领域设计文档模板》第三部分。

---

**下一步** → [用例设计 SOP](sop-use-cases.md)
