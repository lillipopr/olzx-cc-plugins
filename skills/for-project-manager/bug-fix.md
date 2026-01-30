---
description: Bug 修复工作流
---

# Bug 修复（Bug Fix）

## 适用场景

- 生产环境 Bug
- 测试发现的缺陷
- 用例未覆盖的边界情况

## 执行流程

```
Stage 3: Spec Modeling（补充用例） → Stage 4: Artifact Derivation（推导修复）
```

## 核心理念

**Bug = 用例缺失**

每个 Bug 都意味着：
1. 存在未覆盖的边界情况
2. 或不变量定义不完整
3. 或 Bad Case 遗漏

## 详细步骤

### Step 1: Bug 分析

1. 复现 Bug，记录触发条件
2. 定位到具体的状态和操作
3. 分析是哪个不变量被违反

**分析模板**：
```markdown
## Bug 分析

### 现象
[描述 Bug 现象]

### 复现步骤
1. 前置状态：[状态]
2. 操作：[操作]
3. 实际结果：[结果]
4. 期望结果：[结果]

### 根因分析
- 违反的不变量：[INV-X]
- 缺失的用例：[描述]
```

### Step 2: Stage 3 - 补充用例

**执行**：
1. 将 Bug 转化为 Bad Case
2. 检查是否需要新增不变量
3. 更新覆盖矩阵

**闸口**：
```
📋 Bug 已转化为用例

新增用例：
| ID | 场景 | 前置状态 | 操作 | 期望结果 |
|----|------|---------|------|---------|
| TC-XX | [Bug 场景] | [状态] | [操作] | [期望] |

请 Review 用例是否正确描述了 Bug，Review 通过后请回复"继续"
```

### Step 3: Stage 4 - 推导修复

**执行**：
1. 根据用例确定修复位置
2. 生成修复代码骨架
3. 确保不影响其他用例

**闸口**：
```
📋 修复方案已推导

修复位置：[文件:行号]
修复内容：
```diff
- [原代码]
+ [修复代码]
```

请 Review 修复方案，Review 通过后可开始编码
```

## 示例

### Bug 描述
> 过期会员仍能领取点券

### Step 1: 分析
```markdown
### 现象
过期会员点击领取点券，系统发放成功

### 复现步骤
1. 前置状态：EXPIRED
2. 操作：领取点券
3. 实际结果：发放成功
4. 期望结果：拒绝发放

### 根因分析
- 违反的不变量：INV-1（只有 ACTIVE 才能发放）
- 缺失的用例：TC-21（过期会员发券）
```

### Step 2: 补充用例
```markdown
| ID | 场景 | 前置状态 | 操作 | 期望结果 | 覆盖不变量 |
|----|------|---------|------|---------|-----------|
| TC-21 | 过期会员发券 | EXPIRED | 发放点券 | 拒绝 | INV-1 |
```

### Step 3: 推导修复
```markdown
修复位置：Domain.CouponService.grant()

```java
public void grant(String membershipId) {
    Membership m = membershipRepository.findById(membershipId);
+   if (m.getStatus() != MembershipStatus.ACTIVE) {
+       throw new BusinessException("MEMBERSHIP_NOT_ACTIVE");
+   }
    // ... 发放逻辑
}
```
```

## 注意事项

1. **先补用例，再写修复** - 确保修复有据可依
2. **回归测试** - 确保修复不破坏其他功能
3. **更新文档** - 将新用例同步到规格建模文档
