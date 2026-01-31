---
description: 聚合相关原则
---

# 聚合相关原则

## 概述

聚合（Aggregate）是 DDD 中一组相关对象的集合，作为数据修改的单元。聚合定义了事务的一致性边界。

---

## 1. 聚合根识别原则

### 1.1 聚合根判断三问法

**原则**：通过三个问题判断聚合根。

| 问题 | 说明 | 示例 |
|------|------|------|
| **有全局唯一标识吗？** | 需要 ID 来区分身份 | Order 有 orderId |
| **有独立生命周期吗？** | 可独立创建、删除 | Order 可独立创建 |
| **维护一致性边界吗？** | 封装业务不变量 | Order 维护订单总额不变量 |

**判断结果**：三个问题都答"是" → 聚合根候选

### 1.2 聚合根 vs 实体

**原则**：通过标识、访问、生命周期区分聚合根和实体。

| 维度 | 聚合根 | 实体 |
|------|--------|------|
| 标识 | 全局唯一 ID | 局部唯一 ID（通常无独立 ID） |
| 访问 | 外部可直接访问 | 只能通过聚合根访问 |
| 生命周期 | 独立 | 依赖聚合根 |
| 仓储 | 有专门仓储 | 无专门仓储 |
| 示例 | Order | OrderItem |

### 1.3 聚合根识别决策树

```
这个对象是聚合根吗？
│
├─ 有全局唯一标识吗？
│   ├─ 否 → 非聚合根
│   └─ 是 → 继续
│
├─ 有独立生命周期吗？
│   ├─ 否 → 非聚合根
│   └─ 是 → 继续
│
├─ 需要维护一致性吗？
│   ├─ 否 → 可能是值对象
│   └─ 是 → 聚合根候选
│
└─ 会被其他对象直接引用吗？
    ├─ 是 → 聚合根
    └─ 否 → 可能是实体
```

---

## 2. 聚合设计核心原则

### 2.1 黄金法则

> **让聚合尽可能小**

### 2.2 聚合根是唯一入口

**原则**：外部只能通过聚合根访问聚合内部对象。

```typescript
// ✅ 正确：只能通过聚合根访问
const order = orderRepository.findById(orderId)
const item = order.getItem(itemId)  // 通过 order 访问

// ❌ 错误：直接访问实体
const item = orderItemRepository.findById(itemId)  // 不应该有实体仓储
```

### 2.3 聚合尽量小

**原则**：一个聚合应该只包含必须一起修改的对象。

```typescript
// ❌ 错误：大聚合
class User {
  id: UserId
  orders: Order[]        // 不应该在 User 聚合中
  memberships: Membership[] // 不应该在 User 聚合中
  coupons: Coupon[]      // 不应该在 User 聚合中
}

// ✅ 正确：小聚合
class User {
  id: UserId
  profile: UserProfile   // 只包含紧密相关的数据
}
```

### 2.4 强一致性边界

**原则**：聚合内部保证强一致性，聚合间接受最终一致性。

```typescript
// ✅ 正确：聚合内强一致
class Order {
  addItem(item: OrderItem): void {
    this.items.push(item)
    this.updateTotal()  // 保持一致性
  }
}

// 聚合间最终一致：通过领域事件
class Membership {
  activate(): void {
    this.status = MembershipStatus.ACTIVE
    // 发布事件，点券上下文异步处理
    this.addEvent(new MembershipActivated(this.id))
  }
}
```

### 2.5 引用通过 ID

**原则**：聚合间引用只存储 ID，不存储对象引用。

```typescript
// ✅ 正确：只存储 ID
class Order {
  id: OrderId
  userId: UserId  // 只存储 ID
  items: OrderItem[]
}

// ❌ 错误：存储对象引用
class Order {
  id: OrderId
  user: User  // 不应该存储整个对象
  items: OrderItem[]
}
```

### 2.6 一个事务修改一个聚合

**原则**：一个事务只修改一个聚合，保证一致性边界。

```typescript
// ✅ 正确：一个事务修改一个聚合
async function activateMembership(membershipId: string): Promise<void> {
  await db.transaction(async (tx) => {
    const membership = await membershipRepo.findById(membershipId, tx)
    membership.activate()
    await membershipRepo.save(membership, tx)
  })
}

// ❌ 错误：一个事务修改多个聚合
async function activateMembershipAndGrantCoupon(membershipId: string): Promise<void> {
  await db.transaction(async (tx) => {
    // 修改会员聚合
    const membership = await membershipRepo.findById(membershipId, tx)
    membership.activate()
    await membershipRepo.save(membership, tx)
    // 修改点券聚合（不应该在同一个事务）
    const coupon = await couponRepo.findByUserId(membership.userId, tx)
    coupon.addAmount(100)
    await couponRepo.save(coupon, tx)
  })
}
```

---

## 3. 聚合边界确定原则

### 3.1 事务一致性原则

**原则**：需要在同一个事务中修改的数据应该在同一个聚合中。

```
问：这些数据需要在同一个事务中修改吗？
├─ 是 → 应该在同一个聚合中
└─ 否 → 应该在不同的聚合中

示例：
- Order 和 OrderItem → 需要一起修改 → 同一聚合
- Order 和 Payment → 不需要一起修改 → 不同聚合
```

### 3.2 并发冲突原则

**原则**：考虑并发冲突，合理拆分聚合。

```
问：两个用户同时修改会冲突吗？
├─ 会冲突 → 考虑拆分聚合
└─ 不会冲突 → 边界合理

示例：
- User 和 Order → 修改 User 不会影响 Order → 不同聚合
- Order 和 OrderItem → 修改 Order 会影响 OrderItem → 同一聚合
```

### 3.3 数据量原则

**原则**：聚合不应该太大。

| 聚合大小 | 实体数量 | 深度 | 建议 |
|----------|----------|------|------|
| 小 | 1-3 | 1-2 | ✅ 推荐 |
| 中 | 4-7 | 2-3 | ⚠️ 需要评估 |
| 大 | 8+ | 3+ | ❌ 需要拆分 |

---

## 4. 实体识别原则

### 4.1 实体识别三条件

**原则**：满足三个条件才是实体。

| 条件 | 说明 | 示例 |
|------|------|------|
| **有标识吗？** | 需要局部唯一 ID | OrderItem 有 itemId |
| **可变吗？** | 属性可以变化 | OrderItem 数量可变 |
| **生命周期依赖聚合根吗？** | 不能独立存在 | OrderItem 依赖 Order |

### 4.2 实体 vs 值对象

**原则**：通过标识、可变性、生命周期区分实体和值对象。

| 维度 | 实体 | 值对象 |
|------|------|--------|
| 标识 | 有 ID | 无 ID，用值判断相等 |
| 可变性 | 可变 | 不可变 |
| 生命周期 | 依赖聚合根 | 可独立存在 |

---

## 5. 值对象识别原则

### 5.1 值对象识别三条件

**原则**：满足三个条件才是值对象。

| 条件 | 说明 | 示例 |
|------|------|------|
| **无标识吗？** | 用值判断相等 | Money 用 amount+currency 判断 |
| **不可变吗？** | 创建后不能修改 | Money 的 amount 不能修改 |
| **可复用吗？** | 可在多处使用 | Money 可用于 Order、Payment |

### 5.2 值对象设计原则

**原则**：值对象必须不可变。

```typescript
// ✅ 正确：值对象不可变
class Money {
  readonly amount: bigint
  readonly currency: string

  add(other: Money): Money {
    return new Money(this.amount + other.amount, this.currency)  // 返回新对象
  }
}

// ❌ 错误：值对象可变
class Money {
  amount: bigint  // 可变
  currency: string

  add(other: Money): void {
    this.amount += other.amount  // 修改原对象
  }
}
```

---

## 6. 聚合不变量原则

### 6.1 不变量定义

**不变量（Invariant）**：聚合必须始终保持的业务规则。

### 6.2 不变量分类

| 类型 | 说明 | 示例 |
|------|------|------|
| **状态约束** | 状态之间的排斥关系 | 过期的订单不能修改 |
| **数量约束** | 数量限制 | 订单不能超过 10 个项目 |
| **值约束** | 属性值范围 | 金额不能为负 |
| **引用约束** | 关联完整性 | 订单项必须关联有效产品 |

### 6.3 不变量实现原则

**原则**：在聚合根方法中强制不变量。

```typescript
class Order {
  addItem(item: OrderItem): void {
    // 不变量：订单不能有 10 个以上的项目
    if (this.items.length >= 10) {
      throw new Error("Cannot add more than 10 items")
    }
    // 不变量：订单项价格不能为负
    if (item.price.isNegative()) {
      throw new Error("Item price cannot be negative")
    }
    this.items.push(item)
    this.recalculateTotal()
  }
}
```

---

## 7. 聚合设计模式

### 7.1 单实体聚合

**原则**：聚合只包含聚合根，没有其他实体。

```typescript
class Membership {
  id: MembershipId
  userId: UserId
  status: MembershipStatus
  level: MembershipLevel

  activate(): void {
    this.status = MembershipStatus.ACTIVE
  }
}
```

### 7.2 父子聚合

**原则**：聚合包含父子关系的实体。

```typescript
class Order {
  id: OrderId
  items: OrderItem[]  // 子实体

  addItem(item: OrderItem): void {
    this.items.push(item)
    this.updateTotal()
  }
}
```

### 7.3 聚合引用

**原则**：聚合通过 ID 引用其他聚合。

```typescript
class Order {
  id: OrderId
  userId: UserId  // 引用 User 聚合
  shippingAddressId: AddressId  // 引用 Address 聚合
}
```

---

## 检查清单

聚合设计完成前，确认：

- [ ] 聚合根有全局唯一标识
- [ ] 聚合边界清晰，大小合理
- [ ] 聚合内强一致，聚合间最终一致
- [ ] 聚合根是唯一入口
- [ ] 聚合间引用只使用 ID
- [ ] 不变量已识别和实现
- [ ] 一个事务只修改一个聚合
- [ ] 考虑了并发冲突处理
- [ ] 实体和值对象已正确区分
- [ ] 值对象不可变
