---
name: spec-fix
description: Bug 修复，补充用例并推导修复方案
---

# /spec-fix - Bug 修复

## 功能

将 Bug 转化为用例，推导修复方案。

## 使用方式

```
/spec-fix
/spec-fix {Bug 描述}
```

## 执行流程

```
Stage 3: Spec Modeling（补充用例） → Stage 4: Artifact Derivation（推导修复）
```

## 启动提示

```
🐛 启动 Bug 修复

将按以下流程执行：
1. 分析 Bug，转化为用例
2. 补充到规格建模文档
3. 推导修复方案

请描述 Bug：
- 现象是什么？
- 复现步骤是什么？
- 期望结果是什么？
```

## 关联的 Agent

- Stage 3: `agents/senior-domain-architect/AGENT.md` (规格建模部分)
- Stage 4: `agents/senior-fullstack-engineer/AGENT.md`

## 关联的 Skill

- `skills/for-spec-modeler/SKILL.md`
- `skills/for-fullstack-engineer/SKILL.md`

## 关联的 Workflow

- `skills/for-project-manager/workflows/bug-fix.md`
