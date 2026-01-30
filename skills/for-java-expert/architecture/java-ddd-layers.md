---
description: 后端 Java DDD 分层架构规范
---

# Java DDD 分层架构

## 分层结构

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

## 各层职责

### Controller 层
- 接收 HTTP 请求
- 参数校验（@Valid）
- 调用 Application 层
- 返回 DTO

### Application 层
- 用例编排
- 事务管理（@Transactional）
- 调用 Domain 层
- 发布领域事件

### Domain 层
- 业务逻辑
- 不变量校验
- 状态转移
- 领域事件定义

### Gateway/Infra 层
- 外部服务调用
- 消息队列发送
- 缓存操作

### Mapper 层
- 数据库 CRUD
- ORM 映射

## 命名规范

| 层 | 类命名 | 示例 |
|----|--------|------|
| Controller | XxxController | MembershipController |
| Application | XxxAppService | MembershipAppService |
| Domain | Xxx (实体) / XxxService | Membership / MembershipService |
| Gateway | XxxGateway | PaymentGateway |
| Mapper | XxxMapper | MembershipMapper |

## 依赖方向

```
Controller → Application → Domain ← Gateway
                              ↑
                           Mapper
```

Domain 层不依赖任何其他层（依赖倒置）。
