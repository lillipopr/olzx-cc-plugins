---
name: spec-iter
description: 功能迭代，基于已有 PRD 增量更新
---

# /spec-iter - 功能迭代

## 功能

基于已有 PRD 进行功能迭代。

## 使用方式

```
/spec-iter
/spec-iter {变更描述}
```

## 执行流程

```
Stage 2: DDD Design → Stage 3: Spec Modeling → Stage 4: Artifact Derivation
```

## 启动提示

```
🔄 启动功能迭代

将按以下流程执行：
1. Stage 2: DDD Design（增量更新）
2. Stage 3: Spec Modeling（增量更新）
3. Stage 4: Artifact Derivation（增量更新）

请提供：
1. 已有 PRD 文档（或路径）
2. 本次迭代的变更内容
```

## 关联的 Agent

- Stage 2: `agents/senior-domain-architect/AGENT.md`
- Stage 3: `agents/senior-domain-architect/AGENT.md` (规格建模部分)
- Stage 4: `agents/senior-fullstack-engineer/AGENT.md`

## 关联的 Skill

- `skills/for-domain-architect/SKILL.md`
- `skills/for-spec-modeler/SKILL.md`
- `skills/for-fullstack-engineer/SKILL.md`

## 关联的 Workflow

- `skills/for-project-manager/workflows/feature-iteration.md`
