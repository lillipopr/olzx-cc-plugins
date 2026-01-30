---
description: Stage 3 - 规格建模（实体状态/不变量/用例）
---

# Stage 3: Spec Modeling（规格建模）

## 概述

| 项目 | 说明 |
|------|------|
| **执行者** | AI + 人工审核 |
| **输入** | Stage 2 DDD Design |
| **产出物** | 《规格建模文档》 |
| **下游依赖** | Stage 4 Artifact Derivation |
| **闸口条件** | 状态完备、不变量完整、用例覆盖（含 Bad Case） |

## 职责边界

**本阶段负责**：
- 实体状态枚举和状态转移
- 不变量（业务规则）定义
- 用例设计（含 Bad Case）

**本阶段不负责**：
- 接口定义（Stage 4）
- 分层架构（Stage 4）
- 代码实现（Stage 4）

## 规格建模三要素

### 1. 状态定义

从 DDD 设计中的实体，提取状态机：

```markdown
## 实体状态

### Membership 状态
| 状态 | 编码 | 说明 | 可转移到 |
|------|------|------|---------|
| INACTIVE | M0 | 未激活/非会员 | ACTIVE |
| ACTIVE | M1 | 生效中 | EXPIRED, CANCELLED |
| EXPIRED | M2 | 已过期 | ACTIVE |
| CANCELLED | M3 | 已取消 | ACTIVE |

### 状态转移图
```
     subscribe
M0 ─────────────→ M1
 ↑                 │
 │    renew        │ expire
 └────────────── M2 ←─┘
                   │
     resubscribe   │ cancel
 └────────────── M3 ←─┘
```

### 2. 不变量定义

将业务规则转化为可验证的不变量：

```markdown
## 不变量

### INV-1: 点券发放前提
只有 ACTIVE 状态的会员才能发放点券

```pseudo
REQUIRE membership.status == ACTIVE
BEFORE couponGrant.create()
```

### INV-2: 状态互斥
会员在任一时刻只能处于一个状态

```pseudo
ASSERT membership.status IN [INACTIVE, ACTIVE, EXPIRED, CANCELLED]
ASSERT COUNT(membership.status) == 1
```

### INV-3: 时间一致性
生效日期必须早于或等于到期日期

```pseudo
ASSERT membership.startDate <= membership.endDate
```
```

### 3. 用例设计

覆盖所有状态转移和边界：

```markdown
## 用例

### 正向用例（Happy Path）

| ID | 场景 | 前置状态 | 操作 | 期望结果 | 覆盖不变量 |
|----|------|---------|------|---------|-----------|
| TC-01 | 首次订阅 | INACTIVE | subscribe | ACTIVE | - |
| TC-02 | 会员续费 | ACTIVE | renew | ACTIVE, endDate 延长 | INV-3 |
| TC-03 | 过期续费 | EXPIRED | resubscribe | ACTIVE | - |

### 边界用例

| ID | 场景 | 前置状态 | 操作 | 期望结果 | 说明 |
|----|------|---------|------|---------|------|
| TC-10 | 到期当天 | ACTIVE | 系统检查 | EXPIRED | 边界：endDate == today |
| TC-11 | 到期前一天 | ACTIVE | 系统检查 | ACTIVE | 边界：endDate == today + 1 |

### Bad Case（禁止态）

| ID | 场景 | 前置状态 | 操作 | 期望结果 | 覆盖不变量 |
|----|------|---------|------|---------|-----------|
| TC-20 | 非会员发券 | INACTIVE | 发放点券 | 拒绝 | INV-1 |
| TC-21 | 过期会员发券 | EXPIRED | 发放点券 | 拒绝 | INV-1 |
| TC-22 | 重复订阅 | ACTIVE | subscribe | 拒绝 | - |
```

## 用例覆盖矩阵

确保所有不变量都有用例覆盖：

```markdown
## 覆盖矩阵

| 不变量 | 正向用例 | Bad Case | 覆盖状态 |
|--------|---------|----------|---------|
| INV-1 | TC-02 | TC-20, TC-21 | ✅ |
| INV-2 | TC-01~03 | - | ✅ |
| INV-3 | TC-02 | - | ✅ |
```

## 闸口检查清单

进入 Stage 4 前，必须确认：

- [ ] 状态枚举完备且互斥
- [ ] 状态转移图完整，无遗漏路径
- [ ] 每个不变量都有伪代码表达
- [ ] 正向用例覆盖所有 Happy Path
- [ ] **Bad Case 存在且覆盖所有禁止态**
- [ ] 覆盖矩阵显示所有不变量都被覆盖

## 方法论参考

- [实体抽取 DDD 原则](../methodology/entity-extraction.md)
- [状态空间设计](../methodology/state-space-design.md)
- [不变量定义](../methodology/invariants.md)

## 模板

详见 [templates/tpl-spec-modeling.md](../templates/tpl-spec-modeling.md)
