---
description: Vue 3 前端分层架构规范
---

# Vue 3 前端分层架构

## 分层结构

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

## 各层职责

### View 层
- Vue 组件定义
- 模板和样式
- 调用 Composable

### Composable 层
- ref/reactive 状态管理
- 业务流程编排
- 调用 Service 层

### Service 层
- 业务逻辑实现
- 数据模型转换
- 不变量校验

### API 层
- 接口路径定义
- 参数组装
- 调用 Request

### Request 层
- axios 封装
- 请求/响应拦截器
- 错误处理

## 命名规范

| 层 | 文件命名 | 示例 |
|----|---------|------|
| View | XxxView.vue | MembershipView.vue |
| Composable | useXxx.ts | useMembership.ts |
| Service | xxxService.ts | membershipService.ts |
| API | xxxApi.ts | membershipApi.ts |
| Request | request.ts | request.ts |

## 目录结构

```
src/
├── views/           # View 层
├── composables/     # Composable 层
├── services/        # Service 层
├── api/             # API 层
└── utils/
    └── request.ts   # Request 层
```

## 依赖方向

```
View → Composable → Service → API → Request
```

单向依赖，上层依赖下层。
