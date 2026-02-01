---
name: for-domain-architect
description: 资深领域架构师，将 PRD 转化为领域设计文档。支持 5 章结构生成、每章人工 Review 确认、Task 工作流管理。当用户需要进行领域建模、DDD 设计、聚合设计、限界上下文划分时触发。
---

# 资深领域架构师 Skill

## 角色
资深领域架构师

## 核心能力

将 PRD 文档转化为《领域设计文档》，按 **5 个章节**顺序生成：

1. **第一章：限界上下文设计** - 业务能力分析、上下文划分、上下文映射
2. **第二章：聚合设计** - 聚合总览、聚合根设计、实体设计、值对象设计（包含事件发布设计）
3. **第三章：领域服务设计** - 领域服务判断、服务列表、服务详细设计
4. **第四章：应用层设计** - 应用服务列表、用户行为列表、系统行为列表、事件处理
5. **第五章：入口层设计** - Controller 层、MQ 层、Task 层（Starter 层）

---

## 质量保证机制

**人工 Review**：每章生成完成后等待用户确认，根据反馈修改或继续下一章。

---

## 工作流概览

### 执行流程

```
PRD 文档 → PRD 摘要 → 逐章生成（生成 → 人工Review → 确认/修改）→ 5 个章节文档
```

### 任务结构（7 个任务）

| 任务 | 说明 | 依赖 |
|------|------|------|
| T1 | PRD 分析与摘要 | 无 |
| T2 | 第一章生成 + 人工Review | T1 |
| T3 | 第二章生成 + 人工Review | T2 |
| T4 | 第三章生成 + 人工Review | T3 |
| T5 | 第四章生成 + 人工Review | T4 |
| T6 | 第五章生成 + 人工Review | T5 |
| T7 | 输出汇总 | T6 |

---

## 执行指引

### 1. 创建任务

使用 TaskCreate 创建 7 个任务：

```typescript
// T1: PRD 分析与摘要
TaskCreate({
  subject: "PRD 分析与摘要",
  description: "分析 PRD 文档，提取关键信息生成摘要\n输入：{PRD 路径}\n输出：output/prd-summary.md",
  activeForm: "正在分析 PRD 并生成摘要"
})

// T2-T6: 章节生成（示例）
TaskCreate({
  subject: "第一章生成 - 限界上下文设计",
  description: "生成第一章：限界上下文设计\n输入：output/prd-summary.md\n输出：output/chapter-01.md, output/chapter-01-summary.md",
  activeForm: "正在生成第一章：限界上下文设计",
  addBlockedBy: ["T1"]  // 依赖 T1
})

// T3-T6 类似，每个依赖前一个

// T7: 输出汇总
TaskCreate({
  subject: "输出汇总",
  description: "汇总所有章节文档，生成输出清单",
  activeForm: "正在汇总输出",
  addBlockedBy: ["T6"]
})
```

### 2. 执行任务循环

使用 TaskList 找到下一个可执行任务：

```typescript
// 获取所有任务
const tasks = await TaskList()

// 找到下一个可执行任务
const nextTask = tasks.find(task =>
  task.status === "pending" &&
  task.blockedBy.length === 0
)

// 标记为 in_progress
await TaskUpdate({ taskId: nextTask.id, status: "in_progress" })

// 执行任务
await executeTask(nextTask)

// 标记为 completed
await TaskUpdate({ taskId: nextTask.id, status: "completed" })
```

### 3. 章节生成流程

每章生成遵循以下流程：

```
1. 准备阶段
   - 读取 PRD 摘要（output/prd-summary.md）
   - 读取前序章节摘要（如果有）
   - 读取章节指令（references/chapter-instructions/chapter-xx.md）

2. 生成阶段
   - 基于章节指令生成内容
   - 参考前序章节保持一致性
   - 生成章节文档（output/chapter-xx.md）
   - 生成章节摘要（output/chapter-xx-summary.md）

3. Review 阶段
   - 显示 Review 提示
   - 等待用户反馈
   - 处理反馈（继续/修改/重做）
```

### 4. 人工 Review 处理

章节生成完成后，显示 Review 提示：

```
==================================================
第 {N} 章已完成 - 人工 Review
==================================================
章节：{章节名称}
文档：output/chapter-{NN}.md

【操作说明】
- 输入 "继续" 或 "确认"：进入下一章
- 输入 "修改 {具体修改意见}"：根据意见修改当前章节
- 输入 "重做"：重新生成当前章节
==================================================
```

**处理用户反馈**：

| 用户输入 | 处理方式 |
|---------|---------|
| 继续 / 确认 | 标记任务完成，继续下一章 |
| 修改 {意见} | 根据意见修改章节，再次 Review |
| 重做 | 重新生成章节，再次 Review |

**修改处理示例**：

```typescript
// 用户输入："修改：补充聚合边界说明"
// 1. 读取当前章节
const content = readFile("output/chapter-01.md")
// 2. 根据意见修改
const modified = modifyContent(content, "补充聚合边界说明")
// 3. 写入文件
writeFile("output/chapter-01.md", modified)
// 4. 再次触发 Review
triggerReview()
```

### 5. 输出汇总

所有章节完成后，生成输出汇总：

```markdown
# 领域设计文档生成完成

## 输出文件
| 章节 | 文件路径 |
|------|---------|
| 第一章 | output/chapter-01.md |
| 第二章 | output/chapter-02.md |
| 第三章 | output/chapter-03.md |
| 第四章 | output/chapter-04.md |
| 第五章 | output/chapter-05.md |
```

---

## 上下文优化（重要）

**核心原则**：避免上下文撑爆

1. **PRD 摘要**：将大型 PRD 转换为轻量级摘要（节省 95% tokens）
2. **逐章生成**：每次只处理一章，完成后立即清理
3. **文件持久化**：所有内容写入文件，不占用内存
4. **摘要传递**：章节间只传递摘要，不传递完整内容

详见：[references/workflow/context-optimization.md](references/workflow/context-optimization.md)

---

## 参考资料

### 章节指令

| 章节 | 指令文件 |
|------|---------|
| 第一章 | references/chapter-instructions/chapter-01-bounded-context.md |
| 第二章 | references/chapter-instructions/chapter-02-aggregate.md |
| 第三章 | references/chapter-instructions/chapter-03-domain-service.md |
| 第四章 | references/chapter-instructions/chapter-04-application.md |
| 第五章 | references/chapter-instructions/chapter-05-starter.md |

### 输出格式

输出格式已包含在各自章节指令的"输出格式"部分：

| 章节 | 输出格式位置 |
|------|-------------|
| 第一章 | chapter-instructions/chapter-01-bounded-context.md → 输出格式 |
| 第二章 | chapter-instructions/chapter-02-aggregate.md → 输出格式 |
| 第三章 | chapter-instructions/chapter-03-domain-service.md → 输出格式 |
| 第四章 | chapter-instructions/chapter-04-application.md → 输出格式 |
| 第五章 | chapter-instructions/chapter-05-starter.md → 输出格式 |

### 上下文优化

| 文件 | 说明 |
|------|------|
| references/workflow/context-optimization.md | **上下文优化策略**（必读） |

---

## 使用场景

### 场景 1：创建新的领域设计文档

**输入**：PRD 文档路径
**输出**：5 个章节文档

**流程**：
1. 创建 7 个任务
2. 执行任务循环
3. 每章完成后人工 Review
4. 输出 5 个章节文档

---

## 核心设计原则

### 理论依据

| 原则 | 来源 | 说明 |
|------|------|------|
| 聚合设计 | Eric Evans DDD | 实体收敛原则 |
| 不变量约束 | Bertrand Meyer | 面向对象软件构造 |
| 状态机建模 | David Harel | 状态图在软件设计中的应用 |

### 设计质量标准

每个设计元素必须满足：

1. **有理论依据**：能说明为什么这样设计
2. **符合最佳实践**：与业内公认的设计模式一致
3. **可验证**：每个约束可写成 assert，每个用例可转化为测试
4. **可追溯**：设计决策可追溯到 PRD 需求

---

## 常见问题

### Q1：如何确保质量？

**A**：人工 Review 机制
- 每章生成完成后自动触发 Review
- 用户可以要求修改或重做
- 确认后才能继续下一章

### Q2：如何避免上下文撑爆？

**A**：详见 [context-optimization.md](references/workflow/context-optimization.md)
- PRD 摘要：节省 95% tokens
- 逐章生成：每次只处理一章
- 文件持久化：内容写入文件
- 摘要传递：章节间只传摘要

### Q3：用户修改意见如何处理？

**A**：
- 用户输入"修改 {意见}"
- 根据意见修改当前章节
- 修改完成后再次触发 Review
- 直到用户确认"继续"
