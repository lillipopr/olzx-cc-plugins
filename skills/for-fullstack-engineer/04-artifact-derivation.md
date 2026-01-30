---
description: Stage 4 - 工件推导（分层架构/接口契约/实现位置）
---

# Stage 4: Artifact Derivation（工件推导）

## 概述

| 项目 | 说明 |
|------|------|
| **执行者** | AI + 人工审核 |
| **输入** | Stage 3 Spec Modeling |
| **产出物** | 《工件推导文档》 |
| **下游依赖** | 代码实现 |
| **闸口条件** | 分层清晰、契约一致、位置明确 |

## 职责边界

**本阶段负责**：
- 各端分层架构设计
- 接口契约定义（API/DTO/VO）
- 实现位置映射
- 代码骨架生成

**本阶段不负责**：
- 业务规则定义（Stage 3 已完成）
- 用例设计（Stage 3 已完成）

## 工件推导内容

### 1. 分层架构

根据项目类型，确定分层结构：

#### 后端 DDD 分层

```
┌─────────────────────────────────────────────────────┐
│ Controller    │ 接收请求、参数校验、调用 Application │
├─────────────────────────────────────────────────────┤
│ Application   │ 用例编排、事务管理、调用 Domain      │
├─────────────────────────────────────────────────────┤
│ Domain        │ 业务逻辑、不变量校验、领域事件       │
├─────────────────────────────────────────────────────┤
│ Gateway/Infra │ 外部服务调用、消息发送              │
├─────────────────────────────────────────────────────┤
│ Mapper        │ 数据库访问、ORM 映射                │
└─────────────────────────────────────────────────────┘
```

#### iOS MVVM 分层

```
┌─────────────────────────────────────────────────────┐
│ View          │ SwiftUI 视图、用户交互              │
├─────────────────────────────────────────────────────┤
│ ViewModel     │ 状态管理、业务编排、调用 Service    │
├─────────────────────────────────────────────────────┤
│ Service       │ 业务逻辑、数据转换                  │
├─────────────────────────────────────────────────────┤
│ Gateway       │ 接口聚合、缓存策略                  │
├─────────────────────────────────────────────────────┤
│ Network       │ HTTP 请求、响应解析                 │
└─────────────────────────────────────────────────────┘
```

#### Vue 3 前端分层

```
┌─────────────────────────────────────────────────────┐
│ View          │ Vue 组件、模板、样式                │
├─────────────────────────────────────────────────────┤
│ Composable    │ 状态管理、业务编排（类似 ViewModel）│
├─────────────────────────────────────────────────────┤
│ Service       │ 业务逻辑、数据转换                  │
├─────────────────────────────────────────────────────┤
│ API           │ 接口定义、参数组装                  │
├─────────────────────────────────────────────────────┤
│ Request       │ HTTP 请求、拦截器                   │
└─────────────────────────────────────────────────────┘
```

### 2. 接口契约

#### API 定义

```markdown
### POST /api/membership/subscribe

**功能**：创建会员订阅

**入参**：
| 字段 | 类型 | 必填 | 说明 |
|------|------|------|------|
| userId | String | Y | 用户 ID |
| periodType | String | Y | MONTHLY/YEARLY |

**返回**：
| 字段 | 类型 | 说明 |
|------|------|------|
| membershipId | String | 订阅 ID |
| status | String | 状态 |
| startDate | String | 生效日期 |
| endDate | String | 到期日期 |

**错误码**：
| 错误码 | 说明 | 触发条件 |
|--------|------|---------|
| ALREADY_SUBSCRIBED | 已订阅 | 当前状态为 ACTIVE |
| INVALID_PERIOD | 无效周期 | periodType 不合法 |

**对应用例**：TC-01, TC-22
**对应不变量**：-
```

### 3. 实现位置映射

```markdown
## 实现位置映射

### 不变量实现位置

| 不变量 | 后端位置 | iOS 位置 | 前端位置 |
|--------|---------|----------|---------|
| INV-1 | Domain.CouponService | Service.CouponService | - |
| INV-2 | Domain.Membership | ViewModel | Composable |
| INV-3 | Domain.Membership | Model | - |

### 用例实现位置

| 用例 | 后端位置 | iOS 位置 | 前端位置 |
|------|---------|----------|---------|
| TC-01 | Application.MembershipAppService.subscribe | ViewModel.subscribe | Composable.subscribe |
| TC-20 | Domain.CouponService.grant | Service.CouponService | - |
```

### 4. 代码骨架

```markdown
## 代码骨架

### 后端 - MembershipAppService

```java
@Service
public class MembershipAppService {

    /**
     * 创建订阅
     * @see TC-01
     */
    @Transactional
    public MembershipDTO subscribe(SubscribeCommand cmd) {
        // 1. 检查是否已订阅 (TC-22)
        // 2. 创建 Membership 实体
        // 3. 保存并发布事件
        // 4. 返回 DTO
    }
}
```

### iOS - MembershipViewModel

```swift
@MainActor
class MembershipViewModel: ObservableObject {

    /// 创建订阅
    /// - SeeAlso: TC-01
    func subscribe(periodType: PeriodType) async throws {
        // 1. 调用 Service
        // 2. 更新状态
        // 3. 处理错误
    }
}
```
```

## 闸口检查清单

进入编码前，必须确认：

- [ ] 分层架构清晰，各层职责明确
- [ ] 接口契约完整（入参/返回/错误码）
- [ ] 每个接口都关联到用例
- [ ] 每个不变量都有实现位置
- [ ] 各端契约一致（字段名、类型、错误码）
- [ ] 代码骨架可直接填充实现

## 模板

详见 [templates/tpl-artifact-derivation.md](../templates/tpl-artifact-derivation.md)
