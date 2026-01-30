---
description: 工件推导文档模板
---

# {功能名称} - 工件推导文档

## 文档信息

| 项目 | 内容 |
|------|------|
| 文档版本 | v1.0 |
| 创建日期 | YYYY-MM-DD |
| 关联规格建模 | {规格建模文档链接} |
| 涉及端 | 后端 / iOS / Android / 前端 |

## 1. 分层架构

### 1.1 后端 DDD 分层

| 层 | 职责 | 包路径 |
|----|------|--------|
| Controller | 接收请求、参数校验 | com.xxx.controller |
| Application | 用例编排、事务管理 | com.xxx.application |
| Domain | 业务逻辑、不变量校验 | com.xxx.domain |
| Gateway | 外部服务调用 | com.xxx.gateway |
| Mapper | 数据库访问 | com.xxx.mapper |

### 1.2 iOS MVVM 分层

| 层 | 职责 | 目录 |
|----|------|------|
| View | SwiftUI 视图 | Views/ |
| ViewModel | 状态管理 | ViewModels/ |
| Service | 业务逻辑 | Services/ |
| Gateway | 接口聚合 | Gateways/ |
| Network | HTTP 请求 | Network/ |

### 1.3 前端分层

| 层 | 职责 | 目录 |
|----|------|------|
| View | Vue 组件 | views/ |
| Composable | 状态管理 | composables/ |
| Service | 业务逻辑 | services/ |
| API | 接口定义 | api/ |
| Request | HTTP 请求 | utils/request |

## 2. 接口契约

### 2.1 {接口名称}

**基本信息**：
| 项目 | 内容 |
|------|------|
| 方法 | POST |
| 路径 | /api/xxx |
| 功能 | {功能描述} |

**入参**：
| 字段 | 类型 | 必填 | 说明 |
|------|------|------|------|
| {字段} | {类型} | Y/N | {说明} |

**返回**：
| 字段 | 类型 | 说明 |
|------|------|------|
| {字段} | {类型} | {说明} |

**错误码**：
| 错误码 | 说明 | 触发条件 |
|--------|------|---------|
| {错误码} | {说明} | {条件} |

**关联用例**：TC-01, TC-02

## 3. 实现位置映射

### 3.1 不变量实现位置

| 不变量 | 后端 | iOS | 前端 |
|--------|------|-----|------|
| INV-1 | Domain.XxxService | Service.XxxService | - |

### 3.2 用例实现位置

| 用例 | 后端 | iOS | 前端 |
|------|------|-----|------|
| TC-01 | Application.XxxAppService | ViewModel.xxx | Composable.useXxx |

## 4. 代码骨架

### 4.1 后端

```java
// Application 层
@Service
public class XxxAppService {
    /**
     * {方法描述}
     * @see TC-01
     */
    @Transactional
    public XxxDTO method(XxxCommand cmd) {
        // TODO: 实现
    }
}
```

### 4.2 iOS

```swift
// ViewModel
@MainActor
class XxxViewModel: ObservableObject {
    /// {方法描述}
    /// - SeeAlso: TC-01
    func method() async throws {
        // TODO: 实现
    }
}
```

### 4.3 前端

```typescript
// Composable
export function useXxx() {
  /**
   * {方法描述}
   * @see TC-01
   */
  async function method() {
    // TODO: 实现
  }
  return { method }
}
```

## 变更记录

| 版本 | 日期 | 变更内容 | 作者 |
|------|------|---------|------|
| v1.0 | YYYY-MM-DD | 初始版本 | {作者} |
