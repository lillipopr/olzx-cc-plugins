---
description: 新功能开发完整流程，4 个 Phase，每个阶段强制 planner + 执行 + checkpoint
---

# /dev-feature - 新功能开发完整流程

## 功能

启动新功能开发的完整流程，**严格的多阶段规划控制**：每个 Phase 开始前强制使用 planner 拆解任务，执行后强制 checkpoint 验收。

## 使用方式

```
/dev-feature
/dev-feature {功能描述}
```

## 完整流程

```
┌─────────────────────────────────────────────────────────────┐
│ Phase 1: PRD（产品需求文档）                                │
│   planner → product-manager → checkpoint                    │
├─────────────────────────────────────────────────────────────┤
│ Phase 2: DDD Design（领域设计）                             │
│   planner → domain-architect → checkpoint                   │
├─────────────────────────────────────────────────────────────┤
│ Phase 3: Spec Modeling（规格建模）                          │
│   planner → spec-compiler-v3 → checkpoint                   │
├─────────────────────────────────────────────────────────────┤
│ Phase 4: Artifact Derivation（工件推导）                    │
│   planner → {java/ios/frontend}-expert → checkpoint         │
└─────────────────────────────────────────────────────────────┘
```

## 启动提示

```
🚀 启动新功能开发

将按以下流程执行：
1. Phase 1: PRD（产品需求文档）
2. Phase 2: DDD Design（领域设计）
3. Phase 3: Spec Modeling（规格建模）
4. Phase 4: Artifact Derivation（工件推导）

⚠️  严格的多阶段规划控制：
   - 每个 Phase 开始前强制使用 planner 拆解任务
   - 每个 Phase 执行后强制 checkpoint 验收
   - 只有通过验收才能进入下一阶段

请描述您要开发的功能：
```

---

## Phase 1: PRD（产品需求文档）

### Step 1: 调用 Planner

```
📋 Phase 1: PRD 阶段规划

请调用 planner，规划本阶段的执行步骤：

规划要求：
1. 拆解为原子任务（每个任务可独立验收）
2. 定义每个任务的验收标准（可量化/可检查）
3. 标记关键检查点（必须全部通过）
4. 预估每个任务的执行时间
5. 识别潜在风险和应对方案

输出格式参见 /dev 命令中的 "Planner 输出格式"
```

### Step 2: 执行 Agent

- **Agent**: `agents/product-manager/AGENT.md`
- **Skill**: `skills/for-product-manager/SKILL.md`
- **工作流**:
  - `skills/for-product-manager/workflows/requirement-discovery.md`
  - `skills/for-product-manager/workflows/feature-definition.md`

### Step 3: Checkpoint 验收

```
📋 Phase 1 Checkpoint

### 规划回顾
- 规划步骤数: {总数}
- 已完成: {数量} ✅
- 未完成: {数量} ❌

### 逐项验收
- [ ] 需求探索完成
- [ ] 用户故事编写（INVEST原则）
- [ ] 验收标准可测试
- [ ] In/Out Scope 明确
- [ ] 依赖和风险已识别
- [ ] PRD 文档已生成

### 偏差分析
（如有偏差，列出并说明原因）

### 决策
- [ ] ✅ 通过，进入 Phase 2
- [ ] ⚠️ 有偏差，需要补全
- [ ] ❌ 严重偏离，重新规划

用户确认后继续...
```

### 验收标准（必须全部满足）

1. ✅ **需求背景清晰**：为什么要做这个功能，解决什么问题
2. ✅ **目标用户明确**：谁会使用这个功能
3. ✅ **用户故事完整**：遵循 INVEST 原则
4. ✅ **验收标准可测试**：每个验收标准都可以被测试验证
5. ✅ **In/Out Scope 明确**：什么做、什么不做
6. ✅ **依赖和风险已识别**：技术依赖、业务风险
7. ✅ **成功指标定义**：如何衡量功能成功

---

## Phase 2: DDD Design（领域设计）

### Step 1: 调用 Planner

```
📋 Phase 2: DDD Design 阶段规划

请调用 planner，基于 Phase 1 的 PRD 规划本阶段：

规划要求：
1. 分析 PRD，识别领域概念
2. 拆解为原子任务（上下文划分、聚合设计、实体设计等）
3. 定义每个任务的验收标准
4. 标记关键检查点
5. 预估执行时间

输出格式参见 /dev 命令中的 "Planner 输出格式"
```

### Step 2: 执行 Agent

- **Agent**: `agents/domain-architect/AGENT.md`
- **Skill**: `skills/for-domain-architect/SKILL.md`
- **工作流**:
  - `skills/for-domain-architect/sop/sop-domain-modeling.md`
  - `skills/for-domain-architect/sop/sop-context-partition.md`
  - `skills/for-domain-architect/sop/sop-aggregate-design.md`

### Step 3: Checkpoint 验收

```
📋 Phase 2 Checkpoint

### 规划回顾
- 规划步骤数: {总数}
- 已完成: {数量} ✅

### 逐项验收
- [ ] 上下文划分完成
- [ ] 聚合根设计完成
- [ ] 实体和值对象设计完成
- [ ] 领域事件定义完成
- [ ] 上下文映射图完成
- [ ] DDD 设计文档已生成

### 偏差分析
（如有偏差，列出并说明原因）

### 决策
- [ ] ✅ 通过，进入 Phase 3
- [ ] ⚠️ 有偏差，需要补全
- [ ] ❌ 严重偏离，重新规划

用户确认后继续...
```

### 验收标准（必须全部满足）

1. ✅ **上下文边界清晰**：每个上下文的职责明确
2. ✅ **聚合根设计完整**：识别出所有聚合根，生命周期清晰
3. ✅ **实体和值对象区分明确**：什么时候用实体，什么时候用值对象
4. ✅ **领域事件定义**：状态转移的关键事件已定义
5. ✅ **上下文映射关系**：上下文之间的依赖和协作关系
6. ✅ **不变量约束**：每个聚合的不变量已明确

---

## Phase 3: Spec Modeling（规格建模）

### Step 1: 调用 Planner

```
📋 Phase 3: Spec Modeling 阶段规划

请调用 planner，基于 Phase 2 的 DDD Design 规划本阶段：

规划要求：
1. 分析 DDD 设计，识别需要建模的实体
2. 拆解为原子任务（状态空间设计、用例推导、接口设计等）
3. 定义每个任务的验收标准
4. 标记关键检查点
5. 预估执行时间

输出格式参见 /dev 命令中的 "Planner 输出格式"
```

### Step 2: 执行 Agent

- **Agent**: `agents/spec-compiler-v3/AGENT.md`
- **Skill**: `skills/for-spec-compiler-v3/SKILL.md`
- **工作流**:
  - `skills/for-spec-compiler-v3/sop/phase-1-modeling.md`
  - `skills/for-spec-compiler-v3/sop/phase-2-constraints.md`
  - `skills/for-spec-compiler-v3/sop/phase-3-use-cases.md`
  - `skills/for-spec-compiler-v3/sop/phase-4-e2e-design.md`

### Step 3: Checkpoint 验收

```
📋 Phase 3 Checkpoint

### 规划回顾
- 规划步骤数: {总数}
- 已完成: {数量} ✅

### 逐项验收
- [ ] 实体建模完成（唯一 ID + 生命周期 + 状态转移）
- [ ] 状态空间设计完成
- [ ] 用例推导完成（正常/异常/边界）
- [ ] 端到端接口设计完成
- [ ] 跨端统一建模完成
- [ ] 规格文档已生成

### 偏差分析
（如有偏差，列出并说明原因）

### 决策
- [ ] ✅ 通过，进入 Phase 4
- [ ] ⚠️ 有偏差，需要补全
- [ ] ❌ 严重偏离，重新规划

用户确认后继续...
```

### 验收标准（必须全部满足）

1. ✅ **实体唯一 ID 设计**：每个实体有唯一标识
2. ✅ **生命周期完整**：从创建到销毁的完整状态转移
3. ✅ **状态空间设计**：所有可能的状态和转移条件
4. ✅ **用例覆盖完整**：正常、异常、边界情况都覆盖
5. ✅ **端到端接口设计**：入参、返回值、错误处理、状态转移
6. ✅ **跨端一致性**：后端、前端、移动端的数据模型一致

---

## Phase 4: Artifact Derivation（工件推导）

### Step 1: 识别架构

首先询问用户需要实现的架构：

```
请选择目标架构：

1. Java 后端       (DDD 分层架构：Controller → Application → Domain → Gateway → Mapper)
2. iOS 移动端      (MVVM 分层架构：View → ViewModel → Service → Gateway → Network)
3. Vue 前端        (Vue 3 分层架构：View → Composable → Service → API → Request)
4. Fullstack       (组合以上架构)

请输入数字或架构名称：
```

### Step 2: 调用 Planner

```
📋 Phase 4: Artifact Derivation 阶段规划

请调用 planner，基于 Phase 3 的 Spec Modeling 规划本阶段：

规划要求：
1. 分析规格文档，识别需要生成的工件
2. 根据目标架构，拆解为原子任务
3. 定义每个任务的验收标准
4. 标记关键检查点
5. 预估执行时间

输出格式参见 /dev 命令中的 "Planner 输出格式"
```

### Step 3: 执行 Agent

根据选择的架构，调用对应的 Agent：

#### Java 后端
- **Agent**: `agents/java-expert/AGENT.md`
- **Skill**: `skills/for-java-expert/SKILL.md`
- **分层**: Controller → Application → Domain → Gateway → Mapper

#### iOS 移动端
- **Agent**: `agents/ios-expert/AGENT.md`
- **Skill**: `skills/for-ios-expert/SKILL.md`
- **分层**: View → ViewModel → Service → Gateway → Network

#### Vue 前端
- **Agent**: `agents/frontend-expert/AGENT.md`
- **Skill**: `skills/for-frontend-expert/SKILL.md`
- **分层**: View → Composable → Service → API → Request

#### Fullstack
- 组合调用以上 Agent
- 确保跨端接口一致性

### Step 4: Checkpoint 验收

```
📋 Phase 4 Checkpoint

### 规划回顾
- 规划步骤数: {总数}
- 已完成: {数量} ✅

### 逐项验收
- [ ] Controller 层实现完成
- [ ] Application/ViewModel/Composable 层实现完成
- [ ] Domain/Service 层实现完成
- [ ] Gateway/API 层实现完成
- [ ] Mapper/Network/Request 层实现完成
- [ ] 单元测试完成（覆盖率 80%+）
- [ ] 集成测试完成
- [ ] E2E 测试完成
- [ ] 代码文档完成

### 偏差分析
（如有偏差，列出并说明原因）

### 决策
- [ ] ✅ 通过，功能开发完成
- [ ] ⚠️ 有偏差，需要补全
- [ ] ❌ 严重偏离，重新规划

用户确认后完成...
```

### 验收标准（必须全部满足）

1. ✅ **分层架构完整**：所有层都已实现
2. ✅ **接口契约完整**：入参、返回值、错误处理
3. ✅ **单元测试覆盖**：覆盖率 80% 以上
4. ✅ **集成测试通过**：跨层调用测试
5. ✅ **E2E 测试通过**：端到端业务流程测试
6. ✅ **代码可维护**：遵循 SOLID 原则，高内聚低耦合
7. ✅ **文档完整**：API 文档、关键业务逻辑文档

---

## 完成总结

所有 Phase 完成后，输出：

```
🎉 新功能开发完成！

## 执行摘要
- 功能名称: {名称}
- 总 Phase 数: 4
- 总执行时间: {时间}
- 总任务数: {数量}

## 各阶段摘要
- Phase 1 (PRD): {完成度} - {关键产出}
- Phase 2 (DDD): {完成度} - {关键产出}
- Phase 3 (Spec): {完成度} - {关键产出}
- Phase 4 (实现): {完成度} - {关键产出}

## 生成文档
- PRD: {路径}
- DDD 设计: {路径}
- 规格文档: {路径}
- 实现代码: {路径}
- 测试代码: {路径}

## 下一步建议
- [ ] 代码审查（/dev review）
- [ ] 安全审查（/dev security）
- [ ] 创建 PR（/dev pr）
- [ ] 部署验证
```

## 常见问题

### Q: 可以跳过某个 Phase 吗？
**A:** 不建议。每个 Phase 都是严格规划控制的结果，跳过可能导致后续阶段返工。如果确实需要跳过，需要明确说明原因并获得用户确认。

### Q: Phase 4 可以只实现部分架构吗？
**A:** 可以。在 Step 1 选择架构时，可以选择只实现 Java 后端、只实现 iOS 等。

### Q: checkpoint 失败了怎么办？
**A:** 根据失败严重程度：
- ⚠️ 小偏差：补全缺失内容，然后继续
- ❌ 大偏差：重新规划本阶段，然后重新执行
- ❌ 质量问题：重新执行本阶段
