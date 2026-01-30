---
name: spec-new
description: 首次功能建设，完整执行 4 个 Stage
---

# /spec-new - 首次功能建设

## 功能

启动首次功能建设的完整流程。

## 使用方式

```
/spec-new
/spec-new {功能描述}
```

## 执行流程

```
Stage 1: PRD → Stage 2: DDD Design → Stage 3: Spec Modeling → Stage 4: Artifact Derivation
```

## 启动提示

```
🚀 启动首次功能建设

将按以下流程执行：
1. Stage 1: PRD（产品需求文档）
2. Stage 2: DDD Design（领域设计）
3. Stage 3: Spec Modeling（规格建模）
4. Stage 4: Artifact Derivation（工件推导）

每个 Stage 完成后会等待您 Review 确认。

请描述您要开发的功能：
```

## 关联的 Agent

- Stage 1: `agents/senior-product-manager/AGENT.md`
- Stage 2: `agents/senior-domain-architect/AGENT.md`
- Stage 3: `agents/senior-domain-architect/AGENT.md` (规格建模部分)
- Stage 4: `agents/senior-fullstack-engineer/AGENT.md`

## 关联的 Skill

- `skills/for-product-manager/SKILL.md`
- `skills/for-domain-architect/SKILL.md`
- `skills/for-spec-modeler/SKILL.md`
- `skills/for-fullstack-engineer/SKILL.md`

## 关联的 Workflow

- `skills/for-project-manager/workflows/full-feature.md`
