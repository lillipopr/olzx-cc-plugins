---
description: 入口层（Starter 层）相关原则
---

# 入口层（Starter 层）相关原则

## 概述

入口层（Starter Layer）定义外部系统与领域层的交互入口，包括 HTTP 接口、消息队列、定时任务。

---

## 1. 入口层核心原则

### 1.1 薄入口层原则

**原则**：入口层仅负责参数校验、调用应用层、返回结果，不包含业务逻辑。

```typescript
// ✅ 正确：薄入口层
@Post("/memberships")
async createMembership(@Body() cmd: CreateMembershipRequest): Promise<ApiResponse<MembershipDTO>> {
  // 1. 参数校验
  this.validate(cmd)

  // 2. 调用应用层
  const membership = await this.membershipAppService.createMembership(cmd)

  // 3. 返回结果
  return ApiResponse.success(this.toDTO(membership))
}

// ❌ 错误：入口层包含业务逻辑
@Post("/memberships")
async createMembership(@Body() cmd: CreateMembershipRequest): Promise<ApiResponse<MembershipDTO>> {
  // 业务逻辑应该在应用层或领域层
  if (cmd.level === "VIP" && cmd.years < 1) {
    throw new Error("VIP 会员至少订阅 1 年")
  }

  const membership = await this.membershipAppService.createMembership(cmd)
  return ApiResponse.success(this.toDTO(membership))
}
```

### 1.2 统一响应格式原则

**原则**：所有接口返回统一的响应结构。

```typescript
interface ApiResponse<T> {
  code: number          // 业务状态码
  message: string       // 响应消息
  data?: T             // 业务数据
}
```

### 1.3 异常处理原则

**原则**：统一的异常处理和错误码映射。

| 异常类型 | HTTP 状态码 | 业务状态码 | 错误信息 |
|---------|-----------|-----------|---------|
| 参数校验失败 | 200 | 400 | "参数错误：{详细说明}" |
| 权限不足 | 200 | 403 | "权限不足" |
| 业务规则违反 | 200 | 422 | "{业务错误信息}" |
| 系统错误 | 200 | 500 | "系统错误，请稍后重试" |

---

## 2. Controller 层原则

### 2.1 只允许 POST 接口原则

**原则**：Controller 只允许 POST 接口，其他方法（GET、PUT、DELETE）不使用。

**原因**：
- POST 语义清晰：表示执行操作
- 统一风格：所有接口都是 POST
- 避免歧义：GET、PUT、DELETE 语义容易混淆

### 2.2 接口路径设计原则

**原则**：使用统一路径格式 `/api/版本/端/聚合名/动作`。

#### 路径格式

```
/api/{版本}/{端}/{聚合名}/{动作}
```

| 组成部分 | 说明 | 取值 |
|---------|------|------|
| **api** | 固定前缀 | `api` |
| **版本** | 接口版本 | `v1`, `v2`, `v3` ... |
| **端** | 调用端标识 | `m`（mobile 移动端）, `c`（client 客户端）, `w`（web 网页端）, `a`（admin 管理端） |
| **聚合名** | 聚合根名称 | `wallet`, `user`, `membership`, `order` ... |
| **动作** | 操作动作 | `freeze`, `login`, `create`, `cancel` ... |

#### 端标识定义

| 端标识 | 名称 | 说明 |
|-------|------|------|
| **m** | Mobile | 移动端（iOS/Android App） |
| **c** | Client | 客户端（通用） |
| **w** | Web | 网页端（浏览器） |
| **a** | Admin | 管理端（后台） |

#### 路径示例

```
✅ 好的路径设计
POST /api/v1/m/wallet/freeze        // 移动端：冻结钱包
POST /api/v1/c/user/login           // 客户端：用户登录
POST /api/v1/m/membership/create    // 移动端：创建会员
POST /api/v1/a/order/cancel         // 管理端：取消订单
POST /api/v2/w/payment/query        // 网页端：查询支付（v2 版本）

❌ 差的路径设计
GET /api/wallet/freeze              // 不使用 GET
POST /api/wallet/freeze             // 缺少版本号
POST /api/v1/wallet/freeze          // 缺少端标识
POST /wallet/freeze                 // 缺少前缀
POST /api/v1/m/freezeWallet         // 动作使用驼峰，应使用小写
```

#### 路径命名规范

| 规范 | 说明 | 示例 |
|------|------|------|
| **全小写** | 所有路径使用小写 | `wallet`, `freeze`, `login` |
| **连字符分隔** | 多词使用连字符 | `daily-grant`, `expire-check` |
| **动词结尾** | 动作使用动词 | `create`, `cancel`, `freeze`, `login` |

### 2.3 请求参数设计原则

**原则**：使用 Request Body 传递参数，使用 XxxParam 格式，不使用原始数据类型。

#### 参数命名格式

**原则**：所有接口参数使用 `{动作} + {聚合名} + Param` 格式。

```
格式：{动作} + {聚合名} + Param

示例：
✅ 好的参数命名
- CreateMembershipParam（创建会员）
- FreezeWalletParam（冻结钱包）
- CancelOrderParam（取消订单）
- LoginUserParam（用户登录）

❌ 差的参数命名
- CreateMembershipRequest（不是 Request，是 Param）
- MembershipDTO（DTO 是响应对象，不是请求参数）
- CreateMembershipReq（缩写不规范）
```

#### 禁止使用原始数据类型

**原则**：所有接口参数必须是对象类型，不能使用原始数据类型。

```typescript
// ✅ 正确：使用 Param 对象
@Post("/api/v1/m/wallet/freeze")
async freezeWallet(@Body() param: FreezeWalletParam): Promise<ApiResponse<void>> {
  // param 是对象类型
}

interface FreezeWalletParam {
  walletId: string
  reason: string
}

// ❌ 错误：使用原始数据类型
@Post("/api/v1/m/wallet/freeze")
async freezeWallet(@Body() walletId: string): Promise<ApiResponse<void>> {
  // 不应该直接使用 string
}

// ❌ 错误：使用多个原始类型参数
@Post("/api/v1/m/user/login")
async login(
  @Body() username: string,
  @Body() password: string
): Promise<ApiResponse<LoginDTO>> {
  // 不应该使用多个原始类型参数
}

// ✅ 正确：包装成 Param 对象
@Post("/api/v1/m/user/login")
async login(@Body() param: LoginUserParam): Promise<ApiResponse<LoginDTO>> {
  // param 是对象类型
}

interface LoginUserParam {
  username: string
  password: string
}
```

#### Request Body vs URL 参数

**原则**：使用 Request Body 传递参数，不使用 URL 参数。

```typescript
// ✅ 正确：使用 Request Body + Param
@Post("/api/v1/m/membership/create")
async createMembership(@Body() param: CreateMembershipParam): Promise<ApiResponse<MembershipDTO>> {
  // param 在 Request Body 中
}

// ❌ 错误：使用 URL 参数
@Post("/api/v1/m/membership/create/:userId/:level")
async createMembership(
  @Param("userId") userId: string,
  @Param("level") level: string
): Promise<ApiResponse<MembershipDTO>> {
  // 不使用 URL 参数
}
```

#### 参数对象设计原则

**原则**：参数对象只包含输入字段，不包含业务规则。

```typescript
// ✅ 正确：Param 只包含输入字段
interface CreateMembershipParam {
  userId: string
  level: MembershipLevel
  startDate: Date
  endDate: Date
}

// ❌ 错误：Param 包含业务规则
interface CreateMembershipParam {
  userId: string
  level: MembershipLevel
  startDate: Date
  endDate: Date
  status: MembershipStatus  // 业务规则，应该在领域层
  isActive: boolean         // 业务规则，应该在领域层
}
```

### 2.4 Controller 命名原则

**原则**：使用"聚合名 + Controller"格式。

```
✅ 好的命名
- MembershipController
- CouponController
- PaymentController

❌ 差的命名
- MembershipService（与 Service 层混淆）
- MembershipHandler（不明确）
```

---

## 3. MQ 层原则

### 3.1 消费者设计原则

**原则**：消费者只负责接收消息、调用应用层、确认消息。

```typescript
// ✅ 正确：薄消费者
class MembershipEventHandler {
  async onMembershipActivated(event: MembershipActivated): Promise<void> {
    // 1. 消息校验
    this.validate(event)

    // 2. 调用应用层
    await this.couponAppService.startDailyGrant(event.membershipId)

    // 3. ACK 确认
    this.ack()
  }
}

// ❌ 错误：消费者包含业务逻辑
class MembershipEventHandler {
  async onMembershipActivated(event: MembershipActivated): Promise<void> {
    // 业务逻辑应该在应用层
    const coupon = await this.couponRepo.findByUserId(event.userId)
    coupon.addAmount(100)
    await this.couponRepo.save(coupon)
  }
}
```

### 3.2 幂等性保证原则

**原则**：消费者必须保证幂等性。

```typescript
// ✅ 正确：保证幂等性
class CouponConsumer {
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

### 3.3 异常处理原则

**原则**：消费者异常不应导致消息丢失。

| 异常类型 | 处理方式 | 是否重试 |
|---------|---------|---------|
| 参数错误 | 发送到死信队列，告警 | 否 |
| 业务错误 | 发送到死信队列，告警 | 否 |
| 系统错误 | 重新入队（最多 N 次） | 是 |
| 超过重试次数 | 发送到死信队列，告警 | 否 |

---

## 4. Task 层原则

### 4.1 定时任务设计原则

**原则**：定时任务只负责触发、调用应用层、记录日志。

```typescript
// ✅ 正确：薄定时任务
@Cron("0 0 * * *")  // 每天凌晨执行
class ExpireMembershipTask {
  async execute(): Promise<void> {
    // 1. 获取分布式锁
    const lock = await this.lockManager.acquire("lock:task:expire-membership")
    if (!lock) {
      return  // 获取失败，跳过本次执行
    }

    try {
      // 2. 调用应用层
      await this.membershipAppService.expireMemberships()
    } finally {
      // 3. 释放锁
      await lock.release()
    }
  }
}

// ❌ 错误：定时任务包含业务逻辑
@Cron("0 0 * * *")
class ExpireMembershipTask {
  async execute(): Promise<void> {
    // 业务逻辑应该在应用层
    const memberships = await this.membershipRepo.findActive()
    for (const membership of memberships) {
      if (membership.isExpired()) {
        membership.expire()
        await this.membershipRepo.save(membership)
      }
    }
  }
}
```

### 4.2 分布式锁原则

**原则**：定时任务必须使用分布式锁。

```typescript
// ✅ 正确：使用分布式锁
class ExpireMembershipTask {
  async execute(): Promise<void> {
    const lock = await this.lockManager.acquire(
      `lock:task:expire-membership`,
      { timeout: 30000 }  // 锁超时 30 秒
    )

    if (!lock) {
      return  // 获取失败，跳过本次执行
    }

    try {
      await this.membershipAppService.expireMemberships()
    } finally {
      await lock.release()
    }
  }
}
```

### 4.3 任务监控原则

**原则**：定时任务必须有监控指标。

| 监控指标 | 阈值 | 告警方式 |
|---------|------|---------|
| 执行耗时 | > 30 秒 | 钉钉告警 |
| 失败次数 | > 3 次/天 | 钉钉告警 |
| 跳过次数 | > 10 次/天 | 钉钉告警 |

---

## 5. 入口层与领域层分离原则

### 5.1 数据模型转换原则

**原则**：入口层使用 DTO，不直接使用领域模型。

```typescript
// ✅ 正确：使用 DTO
@Post("/memberships/create")
async createMembership(@Body() cmd: CreateMembershipRequest): Promise<ApiResponse<MembershipDTO>> {
  // 1. Request → Command
  const command = this.toCommand(cmd)

  // 2. 调用应用层
  const membership = await this.membershipAppService.createMembership(command)

  // 3. Domain → DTO
  return ApiResponse.success(this.toDTO(membership))
}

// ❌ 错误：直接使用领域模型
@Post("/memberships/create")
async createMembership(@Body() membership: Membership): Promise<Membership> {
  // 不应该直接暴露领域模型
  await this.membershipAppService.createMembership(membership)
  return membership
}
```

### 5.2 参数校验分层原则

**原则**：入口层负责格式校验，领域层负责业务规则校验。

| 校验类型 | 位置 | 示例 |
|---------|------|------|
| **格式校验** | 入口层 | 邮箱格式、手机号格式 |
| **参数校验** | 入口层 | 必填字段、数值范围 |
| **业务规则校验** | 领域层 | 会员只能有一个生效订阅 |

---

## 6. 接口文档原则

### 6.1 接口文档完整性原则

**原则**：每个接口必须有完整的文档。

| 文档内容 | 说明 |
|---------|------|
| 接口名称 | 清晰的接口名称 |
| 接口描述 | 接口的业务含义 |
| 请求定义 | Request Header、Request Body |
| 响应定义 | 成功响应、失败响应 |
| 请求示例 | JSON 示例 |
| 响应示例 | JSON 示例 |
| 错误码 | 所有可能的错误码 |

### 6.2 接口版本控制原则

**原则**：接口使用路径版本控制。

```
✅ 好的版本控制
POST /api/v1/memberships/create
POST /api/v2/memberships/create

❌ 差的版本控制
POST /api/memberships/create?v=1
POST /api/memberships/create  // 无版本控制
```

---

## 检查清单

入口层设计完成前，确认：

### Controller 层
- [ ] 所有接口都是 POST 方法
- [ ] 路径设计清晰（/api/版本/端/聚合名/动作）
- [ ] 请求参数使用 Request Body
- [ ] 参数使用 XxxParam 格式（{动作} + {聚合名} + Param）
- [ ] 不使用原始数据类型（string, number, boolean）
- [ ] 响应格式统一
- [ ] 异常处理完善
- [ ] 不包含业务逻辑

### MQ 层
- [ ] 每个消费者都有消息定义
- [ ] 每个消费者都有处理逻辑
- [ ] 每个消费者都有异常处理
- [ ] 每个消费者都有幂等性保证

### Task 层
- [ ] 每个任务都有调度策略
- [ ] 每个任务都有执行逻辑
- [ ] 每个任务都有分布式锁
- [ ] 每个任务都有异常处理
- [ ] 每个任务都有监控指标

### 通用
- [ ] 入口层薄，不包含业务逻辑
- [ ] 使用 DTO，不直接使用领域模型
- [ ] 参数校验分层清晰
- [ ] 接口文档完整
