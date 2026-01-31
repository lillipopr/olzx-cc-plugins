---
name: for-domain-architect
description: 资深领域架构师，负责将 PRD 文档转化为完整的《领域设计文档》。包含战略设计、战术设计、约束定义、用例设计四部分，提供完整的 SOP、评分标准（90 分及格线）、检查清单和模板。使用场景：(1) 创建新的领域设计文档 (2) 迭代现有设计 (3) Review 设计质量
---

# 资深领域架构师 Skill

## 核心能力

将 PRD 文档转化为《领域设计文档》，包含四个部分：

1. **战略设计**：业务能力分析、限界上下文划分、上下文映射
2. **战术设计**：聚合设计、实体/值对象、领域事件、领域服务、仓储接口、应用层接口
3. **约束定义**：状态定义、状态转移、不变量、禁止态、执行时机、处理策略
4. **用例设计**：正向用例、Bad Case、边界用例、覆盖矩阵、优先级

## 质量保证机制

**90 分及格线**：每部分独立评分，加权计算总分，任一部分 < 60 分则直接不合格。

```
总分 = Part-1×20% + Part-2×30% + Part-3×25% + Part-4×25%
```

## 工作流程

完整工作流程见 [sop/sop-document-workflow.md](sop/sop-document-workflow.md)

```
输入：PRD 文档 / 用户需求
  ↓
阶段 1：准备工作（读取 PRD，确定范围）
  ↓
阶段 2：按部分逐个设计（每部分完成后自检评分）
  ├── 2.1 战略设计 → 使用 strategic-checklist.md 自检 → ≥60 分通过
  ├── 2.2 战术设计 → 使用 tactical-checklist.md 自检 → ≥60 分通过
  ├── 2.3 约束定义 → 使用 constraint-checklist.md 自检 → ≥60 分通过
  └── 2.4 用例设计 → 使用 usecase-checklist.md 自检 → ≥60 分通过
  ↓
阶段 3：综合评分（加权计算，总分 ≥90 分才能交付）
  ↓
阶段 4：交付（生成交付报告，等待用户 Review）
```

## 目录结构

### 工作流程与 SOP（按使用顺序）

| 文件 | 使用时机 | 说明 |
|------|----------|------|
| [sop/sop-document-workflow.md](sop/sop-document-workflow.md) | **首先阅读** | 完整的创建/迭代工作流程，包含评分机制 |
| [sop/sop-main-flow.md](sop/sop-main-flow.md) | 设计过程中 | 各部分设计 SOP 的导航文档 |
| [sop/sop-constraints.md](sop/sop-constraints.md) | 第三部分 | 约束定义详细 SOP |
| [sop/sop-use-cases.md](sop/sop-use-cases.md) | 第四部分 | 用例设计详细 SOP |

### 评分标准（每部分完成后使用）

| 文件 | 对应部分 | 满分 | 权重 |
|------|----------|------|------|
| [scoring/strategic-scoring.md](scoring/strategic-scoring.md) | 战略设计 | 100 | 20% |
| [scoring/tactical-scoring.md](scoring/tactical-scoring.md) | 战术设计 | 100 | 30% |
| [scoring/constraint-scoring.md](scoring/constraint-scoring.md) | 约束定义 | 100 | 25% |
| [scoring/usecase-scoring.md](scoring/usecase-scoring.md) | 用例设计 | 100 | 25% |

### 检查清单（每部分完成后自检）

| 文件 | 对应部分 |
|------|----------|
| [checklists/strategic-checklist.md](checklists/strategic-checklist.md) | 战略设计 |
| [checklists/tactical-checklist.md](checklists/tactical-checklist.md) | 战术设计 |
| [checklists/constraint-checklist.md](checklists/constraint-checklist.md) | 约束定义 |
| [checklists/usecase-checklist.md](checklists/usecase-checklist.md) | 用例设计 |
| [checklists/review-checklist.md](checklists/review-checklist.md) | 最终审查 |

### 设计原则（设计时参考）

| 文件 | 说明 |
|------|------|
| [principles/ddd-principles.md](principles/ddd-principles.md) | DDD 核心原则 |
| [principles/aggregate-principles.md](principles/aggregate-principles.md) | 聚合设计原则 |
| [principles/invariant-principles.md](principles/invariant-principles.md) | **不变量原则**（三测试） |
| [principles/modeling-principles.md](principles/modeling-principles.md) | 领域建模原则 |

### 方法论（设计时参考）

| 文件 | 说明 |
|------|------|
| [methodology/entity-extraction.md](methodology/entity-extraction.md) | 实体抽取方法论（三条件筛选） |
| [methodology/aggregate-design.md](methodology/aggregate-design.md) | 聚合设计方法论 |
| [methodology/context-mapping.md](methodology/context-mapping.md) | 上下文映射方法论 |
| [methodology/domain-event.md](methodology/domain-event.md) | 领域事件设计方法论 |

### 设计模式（设计时参考）

| 文件 | 说明 |
|------|------|
| [patterns/ddd-patterns.md](patterns/ddd-patterns.md) | DDD 战略设计模式 |
| [patterns/tactical-patterns.md](patterns/tactical-patterns.md) | DDD 战术设计模式 |

### 模板

| 文件 | 说明 |
|------|------|
| [templates/domain-design-template.md](templates/domain-design-template.md) | **领域设计文档模板**（最终产出） |

## 核心设计原则

### 理论依据

| 原则 | 来源 | 说明 |
|------|------|------|
| 聚合设计 | Eric Evans DDD | 实体收敛原则 |
| 不变量约束 | Bertrand Meyer | 面向对象软件构造（Design by Contract） |
| 状态机建模 | David Harel | 状态图在软件设计中的应用 |
| 约束优先级 | Michael Jackson | 问题框架方法 |

### 跨端一致性原则

- **第一部分（战略设计）**：跨端唯一
- **第二部分（战术设计）**：跨端唯一（建模不绑定具体技术栈）
- **第三部分（约束定义）**：跨端一致（伪代码不绑定语言）
- **第四部分（用例设计）**：跨端一致（相同用例，各端验证）

### 设计质量标准

每个设计元素必须满足：

1. **有理论依据**：能说明为什么这样设计
2. **符合最佳实践**：与业内公认的设计模式一致
3. **可验证**：每个约束可写成 assert，每个用例可转化为测试
4. **可追溯**：设计决策可追溯到 PRD 需求

## 支持的架构

- 后端 DDD 分层（Controller → Application → Domain → Gateway/Infra → Mapper）
- 移动端 MVVM 分层（View → ViewModel → Service → Gateway → Network）
- Vue 3 前端分层（View → Composable → Service → API → Request）

## 版本历史

### v4.1（当前版本）
- 新增完整的评分标准体系（4 个部分独立评分）
- 新增工作流程 SOP（sop-document-workflow.md）
- 新增 strategic-checklist 和 tactical-checklist
- 建立 90 分及格线机制
- 删除空目录（design-patterns, domain-knowledge, stages）

### v4.0
- 整合了 spec-compiler-v4 的 Phase 1-3（问题建模、约束定义、用例设计）
- 新增约束定义 SOP 和用例设计 SOP
- 新增不变量原则、约束检查清单、用例检查清单
