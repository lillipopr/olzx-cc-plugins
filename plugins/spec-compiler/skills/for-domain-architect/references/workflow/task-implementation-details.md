# Task 实现细节

> **说明**：本文档包含 Task 管理的实现细节，包括持久化格式和错误处理逻辑。

---

## Task 持久化

### 文件结构

```
output/
├── tasks.json              # 任务列表和状态
├── task-history.json       # 任务执行历史
├── task-errors.json        # 错误记录
```

### tasks.json 格式

```json
{
  "tasks": [
    {
      "id": "T1",
      "subject": "PRD 分析与摘要",
      "status": "completed",
      "startTime": "2024-02-01T10:00:00Z",
      "endTime": "2024-02-01T10:05:00Z",
      "retryCount": 0
    }
  ],
  "currentTaskId": "T6",
  "progress": {
    "completed": 5,
    "inProgress": 1,
    "pending": 15
  }
}
```

### task-history.json 格式

```json
{
  "history": [
    {
      "taskId": "T2",
      "attempt": 1,
      "startTime": "2024-02-01T10:06:00Z",
      "endTime": "2024-02-01T10:10:00Z",
      "result": "completed",
      "outputFiles": [
        "output/chapter-01-v1.md",
        "output/chapter-01-issues-principles.md"
      ]
    }
  ]
}
```

---

## 错误处理

### 错误类型

| 错误类型 | 说明 | 处理方式 |
|---------|------|----------|
| **文件不存在** | 输入文件缺失 | 中止任务，提示用户 |
| **解析错误** | 文件格式错误 | 中止任务，记录错误 |
| **质量不达标** | 未通过质量关卡 | 重试（最多 3 次） |
| **用户取消** | 用户主动取消 | 标记为 cancelled |

### 错误恢复

```
function handleError(task, error) {
  // 记录错误
  logError(task.id, error);

  // 根据错误类型处理
  switch (error.type) {
    case 'FILE_NOT_FOUND':
      // 文件缺失，提示用户
      return { action: 'abort', message: `文件 ${error.file} 不存在` };

    case 'PARSE_ERROR':
      // 解析错误，提示用户
      return { action: 'abort', message: `文件 ${error.file} 格式错误` };

    case 'QUALITY_GATE_FAILED':
      // 质量不达标，重试
      if (task.retryCount < task.qualityGate.maxRetries) {
        return { action: 'retry', message: '重试中...' };
      } else {
        return { action: 'fail', message: '达到最大重试次数' };
      }

    case 'USER_CANCELLED':
      // 用户取消
      return { action: 'cancel', message: '用户取消' };
  }
}
```

---

## 参考文档

- [Task 管理规范](./task-management.md) - 核心 Task 管理概念和流程
- [README](./README.md) - 工作流文档索引
