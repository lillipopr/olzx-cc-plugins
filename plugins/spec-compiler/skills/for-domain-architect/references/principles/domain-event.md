---
description: 领域事件相关原则
---

# 领域事件相关原则

## 概述

领域事件（Domain Event）是 DDD 中表示领域内发生的重要事情，用于捕获领域内有意义的业务事实。

---

## 1. 领域事件特征原则

### 1.1 领域事件核心特征

**原则**：领域事件必须满足以下特征。

| 特征 | 说明 | 示例 |
|------|------|------|
| **已经发生** | 表示过去的事实 | MembershipActivated（会员已激活） |
| **不可变** | 一旦发生不能改变 | 事件不能被修改 |
| **业务相关** | 表达业务含义 | OrderPaid（订单已支付） |
| **携带数据** | 包含必要业务数据 | 包含 orderId, userId, amount |
| **有唯一标识** | 每个事件有唯一 ID | eventId |

### 1.2 领域事件 vs 命令

**原则**：事件是过去式，命令是将来式。

| 维度 | 命令（Command） | 事件（Event） |
|------|-----------------|---------------|
| 时态 | 将来时（要做） | 过去时（已做） |
| 命名 | ActivateMembership | MembershipActivated |
| 来源 | 用户/系统发起 | 聚合发布 |
| 数量 | 可能多个 | 唯一确定 |
| 处理 | 可以失败 | 必须成功 |
| 示例 | "激活会员" | "会员已激活" |

---

## 2. 领域事件识别原则

### 2.1 事件识别时机

**原则**：在以下时机考虑使用领域事件。

| 时机 | 说明 | 示例 |
|------|------|------|
| **状态变化** | 聚合状态发生重要变化 | 会员激活、过期 |
| **跨边界协作** | 需要通知其他上下文 | 会员激活通知点券上下文 |
| **异步处理** | 触发后台任务 | 支付成功触发发货 |
| **审计追踪** | 记录关键操作 | 敏感操作审计 |

### 2.2 事件识别问题清单

**原则**：通过以下问题判断是否需要领域事件。

```
1. 这个事情重要到需要通知其他部分吗？
   是 → 领域事件候选

2. 这个事情是一个已经发生的事实吗？
   是 → 领域事件候选

3. 其他上下文需要对这个事情做出反应吗？
   是 → 领域事件候选

4. 需要审计追踪这个事情吗？
   是 → 领域事件候选
```

---

## 3. 领域事件命名原则

### 3.1 命名格式

**原则**：使用"聚合根 + 过去式动词 + Event"格式。

```
✅ 好的事件命名
- MembershipActivatedEvent（会员已激活）
- OrderPaidEvent（订单已支付）
- CouponExpiredEvent（点券已过期）

❌ 差的事件命名
- ActivateMembershipEvent（命令，不是事件）
- MembershipActivationEvent（缺少动词）
- 会员激活Event（中文，不符合代码规范）
```

### 3.2 命名决策树

```
如何命名事件？
│
├─ 是什么发生了变化？
│   └─ {聚合根}
│
├─ 发生了什么？
│   └─ {过去式动词}
│
└─ 组合 → {聚合根}{过去式动词}Event

示例：
- 会员 → 激活 → MembershipActivatedEvent
- 订单 → 支付 → OrderPaidEvent
```

---

## 4. 领域事件设计原则

### 4.1 事件结构原则

**原则**：事件必须包含必要的业务数据。

```typescript
// ✅ 正确：事件包含必要数据
interface MembershipActivated {
  eventId: string
  membershipId: string
  userId: string
  level: MembershipLevel
  activatedAt: Date
}

// ❌ 错误：事件数据不足
interface MembershipActivated {
  eventId: string
  // 缺少 userId、level 等关键数据
}
```

### 4.2 事件数据最少原则

**原则**：事件只携带订阅方需要的数据，不携带整个聚合。

```typescript
// ✅ 正确：只携带必要数据
interface MembershipActivated {
  eventId: string
  membershipId: string
  userId: string
  level: MembershipLevel
  activatedAt: Date
}

// ❌ 错误：携带整个聚合
interface MembershipActivated {
  eventId: string
  membership: Membership  // 不应该携带整个聚合
}
```

### 4.3 事件不可变原则

**原则**：事件一旦发布就不能修改。

```typescript
// ✅ 正确：事件不可变
class MembershipActivated {
  readonly eventId: string
  readonly membershipId: string
  readonly userId: string
  readonly level: MembershipLevel
  readonly activatedAt: Date
}
```

---

## 5. 领域事件发布原则

### 5.1 事件发布位置原则

**原则**：在聚合根方法中发布事件。

```typescript
// ✅ 正确：在聚合根方法中发布事件
class Membership {
  activate(): void {
    this.status = MembershipStatus.ACTIVE
    // 发布事件
    this.addEvent(new MembershipActivated(
      uuid(),
      this.id.value,
      this.userId.value,
      this.level,
      new Date()
    ))
  }
}
```

### 5.2 事件发布时机原则

**原则**：在状态变更完成后发布事件。

```typescript
// ✅ 正确：状态变更完成后发布事件
class Membership {
  activate(): void {
    // 先修改状态
    this.status = MembershipStatus.ACTIVE
    // 然后发布事件
    this.addEvent(new MembershipActivated(...))
  }
}

// ❌ 错误：状态变更前发布事件
class Membership {
  activate(): void {
    // 先发布事件
    this.addEvent(new MembershipActivated(...))
    // 然后修改状态（如果修改失败，事件已发布）
    this.status = MembershipStatus.ACTIVE
  }
}
```

---

## 6. 领域事件订阅原则

### 6.1 订阅方处理原则

**原则**：订阅方应该异步处理事件。

```typescript
// ✅ 正确：异步处理事件
class CouponEventHandler {
  @EventListener
  async onMembershipActivated(event: MembershipActivated): Promise<void> {
    await this.couponService.startDailyGrant(event.membershipId)
  }
}
```

### 6.2 幂等性原则

**原则**：事件订阅方必须保证幂等性。

```typescript
// ✅ 正确：保证幂等性
class CouponEventHandler {
  async onMembershipActivated(event: MembershipActivated): Promise<void> {
    const idempotentKey = `membership:activated:${event.eventId}`
    const exists = await this.idempotentRepo.exists(idempotentKey)
    if (exists) {
      return  // 已处理，直接返回
    }

    await this.couponService.startDailyGrant(event.membershipId)
    await this.idempotentRepo.save(idempotentKey)
  }
}
```

### 6.3 异常处理原则

**原则**：订阅方处理异常不应影响事件发布方。

```typescript
// ✅ 正确：订阅方异常不影响发布方
class CouponEventHandler {
  async onMembershipActivated(event: MembershipActivated): Promise<void> {
    try {
      await this.couponService.startDailyGrant(event.membershipId)
    } catch (error) {
      // 记录日志，但不向上抛出异常
      this.logger.error("Failed to process event", error)
    }
  }
}
```

---

## 7. 常见领域事件类型

### 7.1 状态变化事件

| 事件 | 说明 | 示例 |
|------|------|------|
| **创建** | 聚合根被创建 | OrderCreated |
| **状态转移** | 聚合根状态变化 | MembershipActivated, OrderPaid |
| **删除** | 聚合根被删除 | MembershipDeleted |

### 7.2 业务事件

| 事件 | 说明 | 示例 |
|------|------|------|
| **业务操作** | 业务操作完成 | PaymentCompleted |
| **规则触发** | 业务规则被触发 | DiscountApplied |
| **时间触发** | 时间点到达 | MembershipExpired |

---

## 检查清单

领域事件设计完成前，确认：

- [ ] 所有领域事件已识别
- [ ] 事件命名符合规范（过去式）
- [ ] 每个事件都有触发条件
- [ ] 每个事件都有携带数据
- [ ] 每个事件都有订阅方
- [ ] 事件是不可变的
- [ ] 事件是已经发生的事实
- [ ] 事件携带必要业务数据
- [ ] 订阅方保证幂等性
- [ ] 订阅方异常不影响发布方
