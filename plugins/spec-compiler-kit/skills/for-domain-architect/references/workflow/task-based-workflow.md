# 基于 Task 工具的工作流系统

> **目标**：将庞大的生成过程拆分为多个小任务，使用 Task 工具管理，确保每个任务保质保量完成

---

## 为什么需要 Task 工具

### 问题分析

| 问题 | 说明 | 影响 |
|------|------|------|
| **流程庞大** | 6个章节 + 评估 + 修改 = 10+ 步 | 容易遗漏 |
| **上下文限制** | 单次处理所有内容会撑爆 | 无法完成 |
| **质量难控** | 中间环节无法验证 | 最终质量差 |
| **进度不透明** | 不知道当前进度 | 用户体验差 |
| **难以调试** | 出错后无法定位问题 | 效率低 |

### Task 工具的优势

| 优势 | 说明 | 效果 |
|------|------|------|
| **任务拆分** | 每个章节是一个独立任务 | 清晰可控 |
| **进度跟踪** | 实时显示任务进度 | 透明可见 |
| **质量关卡** | 每个任务完成后检查 | 质量保证 |
| **依赖管理** | 自动处理任务依赖 | 顺序正确 |
| **失败重试** | 失败任务可以重试 | 稳定性高 |
| **上下文隔离** | 每个任务独立上下文 | 避免撑爆 |

---

## 任务拆分策略

### 整体任务树

```
根任务：生成领域设计文档
│
├─ [T1] PRD 分析与摘要（依赖：无）
│
├─ [T2] 第一章：限界上下文设计（依赖：T1）
│
├─ [T3] 第二章：聚合设计（依赖：T2）
│
├─ [T4] 第三章：领域服务设计（依赖：T3）
│
├─ [T5] 第四章：应用层设计（依赖：T4）
│
├─ [T6] 第五章：领域事件（依赖：T5）
│
├─ [T7] 第六章：入口层设计（依赖：T6）
│
├─ [T8] 综合评分（依赖：T1-T7）
│
└─ [T9] 文档组装（依赖：T8）
```

### 任务定义模板

每个任务包含以下信息：

```typescript
interface TaskDefinition {
  id: string              // 任务ID
  subject: string         // 任务名称
  description: string     // 详细描述
  activeForm: string      // 进行中描述
  dependencies: string[]  // 依赖任务ID
  input: {
    prdSummary: string   // PRD 摘要文件
    chapterOutputs: string[]  // 前序章节输出
    instructionFile: string  // 章节指令文件
    principleFile: string   // 原则文件
  }
  output: {
    contentFile: string   // 输出内容文件
    summaryFile: string  // 输出摘要文件
    score: number        // 评分
  }
  qualityGate: {
    checklistFile: string  // 检查清单文件
    scoringFile: string    // 评分标准文件
    passScore: number      // 及格分数
  }
}
```

---

## 任务执行流程

### 流程图

```
┌─────────────────────────────────────────────────────────────────────────┐
│                      任务执行流程                                        │
├─────────────────────────────────────────────────────────────────────────┤
│                                                                          │
│  开始                                                                    │
│    ↓                                                                     │
│  ┌─────────────────────────────────────────────────────────────────┐    │
│  │ Step 1：创建任务列表                                              │    │
│  │   - 创建所有任务                                                  │    │
│  │   - 定义任务依赖                                                  │    │
│  │   - 设置初始状态（pending）                                        │    │
│  └─────────────────────────────────────────────────────────────────┘    │
│    ↓                                                                     │
│  ┌─────────────────────────────────────────────────────────────────┐    │
│  │ Step 2：执行任务循环                                              │    │
│  │                                                                  │    │
│  │  ┌─────────────────────────────────────────────────────────┐    │    │
│  │  │ 2.1 找到下一个可执行任务                                     │    │    │
│  │  │     - 状态：pending                                         │    │    │
│  │  │     - 依赖：已完成                                         │    │    │
│  │  └─────────────────────────────────────────────────────────┘    │    │
│  │    ↓                                                              │    │
│  │  ┌─────────────────────────────────────────────────────────┐    │    │
│  │  │ 2.2 标记任务为 in_progress                                   │    │    │
│  │  └─────────────────────────────────────────────────────────┘    │    │
│  │    ↓                                                              │    │
│  │  ┌─────────────────────────────────────────────────────────┐    │    │
│  │  │ 2.3 执行任务                                                  │    │    │
│  │  │     - 读取指令和原则                                          │    │
│  │  │     - 生成内容                                                │    │    │
│  │  │     - 写入文件                                                │    │    │
│  │  └─────────────────────────────────────────────────────────┘    │    │
│  │    ↓                                                              │    │
│  │  ┌─────────────────────────────────────────────────────────┐    │    │
│  │  │ 2.4 质量检查                                                  │    │    │
│  │  │     - 使用检查清单自检                                         │    │
│  │  │     - 使用评分标准评分                                         │    │
│  │  └─────────────────────────────────────────────────────────┘    │    │
│  │    ↓                                                              │    │
│  │  ┌─────────────────────────────────────────────────────────┐    │    │
│  │  │ 2.5 检查结果                                                  │    │    │
│  │  │     ├─ 评分 ≥60 → 标记为 completed                            │    │    │
│  │  │     └─ 评分 <60 → 标记为 pending（重做）                       │    │    │
│  │  └─────────────────────────────────────────────────────────┘    │    │
│  │    ↓                                                              │    │
│  │  ┌─────────────────────────────────────────────────────────┐    │    │
│  │  │ 2.6 检查是否所有任务完成                                       │    │    │
│  │  │     ├─ 否 → 返回 2.1                                         │    │
│  │  │     └─ 是 → 结束循环                                           │    │
│  │  └─────────────────────────────────────────────────────────┘    │    │
│  └─────────────────────────────────────────────────────────────────┘    │
│    ↓                                                                     │
│  ┌─────────────────────────────────────────────────────────────────┐    │
│  │ Step 3：综合评分                                                    │    │
│  │   - 读取所有章节评分                                               │    │
│  │   - 计算综合评分                                                   │    │
│  │   - 生成评分报告                                                   │    │
│  └─────────────────────────────────────────────────────────────────┘    │
│    ↓                                                                     │
│  ┌─────────────────────────────────────────────────────────────────┐    │
│  │ Step 4：文档组装                                                    │    │
│  │   - 读取所有章节文件                                               │    │
│  │   - 使用模板组装                                                   │    │
│  │   - 生成最终文档                                                   │    │
│  └─────────────────────────────────────────────────────────────────┘    │
│    ↓                                                                     │
│  结束                                                                    │
│                                                                          │
└─────────────────────────────────────────────────────────────────────────┘
```

---

## 详细任务定义

### 任务 1：PRD 分析与摘要

```yaml
id: "task-1"
subject: "PRD 分析与摘要"
description: |
  从 PRD 文档中提取关键信息，生成轻量级摘要（~500 tokens），
  包括功能概述、核心实体、业务流程、外部接口、异步处理、定时任务。
activeForm: "正在分析 PRD 并生成摘要"
dependencies: []
input:
  prdFile: "prd.md"
  templateFile: "templates/prd-summary-template.md"
output:
  summaryFile: "output/prd-summary.md"
qualityGate:
  checklistFile: "checklists/prd-analysis-checklist.md"
  passScore: null  # 无评分要求
```

**执行步骤**：
1. 读取 PRD 文件
2. 提取关键信息
3. 生成 PRD 摘要
4. 写入 `output/prd-summary.md`
5. 自检通过检查清单
6. 标记为 completed

---

### 任务 2：第一章 - 限界上下文设计

```yaml
id: "task-2"
subject: "第一章：限界上下文设计"
description: |
  基于 PRD 摘要，生成第一章内容，包括业务能力分析、
  限界上下文划分、上下文映射。
activeForm: "正在生成第一章：限界上下文设计"
dependencies: ["task-1"]
input:
  prdSummary: "output/prd-summary.md"
  instructionFile: "chapters/chapter-01-bounded-context.md"
  principleFile: "references/principles/bounded-context.md"
output:
  contentFile: "output/chapter-01.md"
  summaryFile: "output/chapter-01-summary.md"
qualityGate:
  checklistFile: "references/checklists/strategic-checklist.md"
  scoringFile: "references/scoring/01-strategic-scoring.md"
  passScore: 60
```

**执行步骤**：
1. 等待依赖任务完成
2. 读取 PRD 摘要
3. 读取章节指令和原则
4. 生成第一章内容
5. 写入 `output/chapter-01.md`
6. 生成章节摘要
7. 写入 `output/chapter-01-summary.md`
8. 自检通过检查清单
9. 评分通过评分标准
10. 检查评分 ≥60 分
    - 是：标记为 completed
    - 否：标记为 pending（重做）

---

### 任务 3-7：第二-六章

格式与任务 2 类似，只需修改：
- `id`：递增（task-3 到 task-7）
- `subject`：对应章节名称
- `dependencies`：依赖前序任务
- `input`：对应的指令和原则文件
- `output`：对应的输出文件
- `qualityGate`：对应的检查清单和评分标准

---

### 任务 8：综合评分

```yaml
id: "task-8"
subject: "综合评分"
description: |
  基于所有章节的评分，计算综合评分，
  生成评分报告。总分 ≥90 分通过。
activeForm: "正在计算综合评分"
dependencies: ["task-2", "task-3", "task-4", "task-5", "task-6", "task-7"]
input:
  chapterScores:
    - "output/chapter-01-score.md"
    - "output/chapter-02-score.md"
    - "output/chapter-03-score.md"
    - "output/chapter-04-score.md"
    - "output/chapter-05-score.md"
    - "output/chapter-06-score.md"
output:
  reportFile: "output/score-report.md"
qualityGate:
  passScore: 90
```

**执行步骤**：
1. 等待所有章节任务完成
2. 读取所有章节评分
3. 计算综合评分：
   ```
   总分 = 第一章×15% + 第二章×25% + 第三章×10% +
         第四章×15% + 第五章×10% + 第六章×10% +
         设计一致性×15%
   ```
4. 生成评分报告
5. 写入 `output/score-report.md`
6. 检查总分 ≥90 分
    - 是：标记为 completed
    - 否：标记为 pending（需修改）

---

### 任务 9：文档组装

```yaml
id: "task-9"
subject: "文档组装"
description: |
  读取所有章节文件，使用模板组装最终文档。
activeForm: "正在组装最终文档"
dependencies: ["task-8"]
input:
  templateFile: "assets/templates/domain-design-template.md"
  chapters:
    - "output/chapter-01.md"
    - "output/chapter-02.md"
    - "output/chapter-03.md"
    - "output/chapter-04.md"
    - "output/chapter-05.md"
    - "output/chapter-06.md"
output:
  finalFile: "output/{功能名称}-领域设计文档.md"
qualityGate:
  checklistFile: "references/checklists/review-checklist.md"
  passScore: null  # 最终审查，无评分要求
```

**执行步骤**：
1. 等待综合评分任务完成
2. 读取模板文件
3. 读取所有章节文件
4. 组装最终文档
5. 写入 `output/{功能名称}-领域设计文档.md`
6. 最终审查通过检查清单
7. 标记为 completed
8. 输出最终文件路径

---

## Task 工具使用示例

### 创建任务列表

```typescript
// 创建所有任务
await TaskCreate({
  id: "task-1",
  subject: "PRD 分析与摘要",
  description: "从 PRD 文档中提取关键信息，生成轻量级摘要",
  activeForm: "正在分析 PRD 并生成摘要"
})

await TaskCreate({
  id: "task-2",
  subject: "第一章：限界上下文设计",
  description: "生成第一章内容：业务能力分析、限界上下文划分、上下文映射",
  activeForm: "正在生成第一章：限界上下文设计",
  addBlockedBy: ["task-1"]
})

// ... 创建其他任务
```

### 执行任务循环

```typescript
while (true) {
  // 1. 获取任务列表
  const tasks = await TaskList()

  // 2. 找到下一个可执行任务
  const nextTask = tasks.find(t =>
    t.status === 'pending' &&
    t.blockedBy.length === 0
  )

  if (!nextTask) {
    break  // 没有可执行任务，退出
  }

  // 3. 标记为 in_progress
  await TaskUpdate({
    taskId: nextTask.id,
    status: 'in_progress'
  })

  // 4. 执行任务
  const result = await executeTask(nextTask)

  // 5. 质量检查
  const passed = await qualityCheck(result)

  // 6. 更新任务状态
  if (passed) {
    await TaskUpdate({
      taskId: nextTask.id,
      status: 'completed'
    })
  } else {
    // 失败，重置为 pending
    await TaskUpdate({
      taskId: nextTask.id,
      status: 'pending'
    })
  }
}
```

### 任务依赖管理

```typescript
// 设置任务依赖
await TaskUpdate({
  taskId: "task-3",
  addBlockedBy: ["task-2"]  // task-3 依赖 task-2
})

// 解除任务依赖（task-2 完成时）
await TaskUpdate({
  taskId: "task-3",
  addBlockedBy: []
})
```

---

## 质量检查机制

### 检查清单验证

```typescript
async function validateByChecklist(content: string, checklistFile: string): boolean {
  // 1. 读取检查清单
  const checklist = await readFile(checklistFile)

  // 2. 逐项检查
  const items = parseChecklist(checklist)
  const results = items.map(item => ({
    item: item.description,
    passed: checkItem(content, item)
  }))

  // 3. 生成检查报告
  const allPassed = results.every(r => r.passed)

  // 4. 输出结果
  console.table(results)

  return allPassed
}
```

### 评分验证

```typescript
async function scoreChapter(content: string, scoringFile: string): number {
  // 1. 读取评分标准
  const scoring = await readFile(scoringFile)

  // 2. 逐项评分
  const criteria = parseScoring(scoring)
  const scores = criteria.map(criterion => ({
    criterion: criterion.name,
    score: scoreItem(content, criterion),
    maxScore: criterion.maxScore
  }))

  // 3. 计算总分
  const totalScore = scores.reduce((sum, s) => sum + s.score, 0)
  const maxScore = scores.reduce((sum, s) => sum + s.maxScore, 0)
  const finalScore = (totalScore / maxScore) * 100

  // 4. 输出结果
  console.table(scores)
  console.log(`总分: ${finalScore}/${maxScore}`)

  return finalScore
}
```

---

## 进度显示

### 实时进度条

```typescript
function showProgress(tasks: Task[]): void {
  const total = tasks.length
  const completed = tasks.filter(t => t.status === 'completed').length
  const inProgress = tasks.filter(t => t.status === 'in_progress').length
  const pending = tasks.filter(t => t.status === 'pending').length

  console.clear()
  console.log('='.repeat(50))
  console.log(`任务进度：${completed}/${total} (${Math.round(completed/total*100)}%)`)
  console.log('='.repeat(50))
  console.log(`✅ 已完成: ${completed}`)
  console.log(`🔄 进行中: ${inProgress}`)
  console.log(`⏳ 待执行: ${pending}`)
  console.log('='.repeat(50))

  // 显示任务列表
  tasks.forEach(task => {
    const icon = {
      pending: '⏳',
      in_progress: '🔄',
      completed: '✅'
    }[task.status]
    console.log(`${icon} ${task.subject}`)
  })
}
```

---

## 错误处理与重试

### 任务失败处理

```typescript
async function executeTaskWithRetry(task: Task, maxRetries: number = 3): Promise<boolean> {
  let retryCount = 0

  while (retryCount < maxRetries) {
    try {
      // 1. 标记为 in_progress
      await TaskUpdate({
        taskId: task.id,
        status: 'in_progress'
      })

      // 2. 执行任务
      const result = await executeTask(task)

      // 3. 质量检查
      const passed = await qualityCheck(result)

      if (passed) {
        // 4. 标记为 completed
        await TaskUpdate({
          taskId: task.id,
          status: 'completed'
        })
        return true
      } else {
        // 质量不通过，重试
        retryCount++
        console.warn(`任务 ${task.id} 质量检查不通过，重试 ${retryCount}/${maxRetries}`)
      }
    } catch (error) {
      // 执行失败，重试
      retryCount++
      console.error(`任务 ${task.id} 执行失败：${error.message}`)
      console.error(`重试 ${retryCount}/${maxRetries}`)
    }
  }

  // 达到最大重试次数，仍然失败
  console.error(`任务 ${task.id} 达到最大重试次数，标记为失败`)
  await TaskUpdate({
    taskId: task.id,
    status: 'failed',
    metadata: { errorMessage: `重试 ${maxRetries} 次后仍然失败` }
  })
  return false
}
```

---

## 完整示例代码

### 主流程

```typescript
async function generateDomainDesignDocument(prdFile: string): Promise<string> {
  // 1. 创建任务列表
  await createTasks()

  // 2. 执行任务循环
  await executeTaskLoop()

  // 3. 综合评分
  await calculateOverallScore()

  // 4. 文档组装
  const finalDoc = await assembleDocument()

  return finalDoc
}

async function createTasks(): Promise<void> {
  const tasks = [
    {
      id: "task-1",
      subject: "PRD 分析与摘要",
      description: "从 PRD 文档中提取关键信息，生成轻量级摘要",
      activeForm: "正在分析 PRD 并生成摘要"
    },
    {
      id: "task-2",
      subject: "第一章：限界上下文设计",
      description: "生成第一章内容：业务能力分析、限界上下文划分、上下文映射",
      activeForm: "正在生成第一章：限界上下文设计",
      addBlockedBy: ["task-1"]
    },
    // ... 其他任务
  ]

  for (const task of tasks) {
    await TaskCreate(task)
  }
}

async function executeTaskLoop(): Promise<void> {
  let consecutiveCompleted = 0
  const totalTasks = 9

  while (consecutiveCompleted < totalTasks) {
    // 1. 获取任务列表
    const tasks = await TaskList()
    showProgress(tasks)

    // 2. 找到下一个可执行任务
    const nextTask = tasks.find(t =>
      t.status === 'pending' &&
      t.blockedBy.length === 0
    )

    if (!nextTask) {
      console.log("没有可执行的任务，检查是否有任务失败")
      const hasFailed = tasks.some(t => t.status === 'failed')
      if (hasFailed) {
        throw new Error("有任务执行失败，无法继续")
      }
      break
    }

    // 3. 执行任务（带重试）
    const success = await executeTaskWithRetry(nextTask)

    if (success) {
      consecutiveCompleted++
    } else {
      throw new Error(`任务 ${nextTask.id} 执行失败，无法继续`)
    }
  }
}
```

---

## 最佳实践

### DO ✅

- ✅ **任务拆分**：将大任务拆分为多个小任务
- ✅ **依赖明确**：明确定义任务依赖关系
- ✅ **质量关卡**：每个任务完成后进行质量检查
- ✅ **进度透明**：实时显示任务进度
- ✅ **错误处理**：失败任务自动重试
- ✅ **上下文隔离**：每个任务独立上下文

### DON'T ❌

- ❌ **任务过大**：单个任务包含多个章节
- ❌ **依赖不清**：任务依赖关系不明确
- ❌ **缺少检查**：任务完成后不进行质量检查
- ❌ **进度隐藏**：不显示任务进度
- ❌ **错误忽略**：任务失败后不处理
- ❌ **上下文累积**：在单个任务中处理所有内容

---

## 总结

### 优势对比

| 维度 | 不使用 Task 工具 | 使用 Task 工具 |
|------|-----------------|--------------|
| **进度可见性** | ❌ 黑盒 | ✅ 实时可见 |
| **质量可控性** | ❌ 最后才发现问题 | ✅ 每步验证 |
| **错误处理** | ❌ 全盘重来 | ✅ 单点重试 |
| **上下文管理** | ❌ 容易撑爆 | ✅ 隔离管理 |
| **可维护性** | ❌ 难以调试 | ✅ 易于定位 |

### 核心要点

1. **任务拆分**：9个独立任务，每个任务职责单一
2. **依赖管理**：通过 `addBlockedBy` 定义任务依赖
3. **质量关卡**：每个任务完成后进行自检和评分
4. **进度跟踪**：实时显示任务进度
5. **错误处理**：失败任务自动重试，最多3次
6. **上下文隔离**：每个任务独立上下文，避免累积

通过 Task 工具，可以将庞大的生成过程拆分为多个小任务，确保每个任务保质保量完成，同时保持进度透明和质量可控。
