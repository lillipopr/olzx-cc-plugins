---
name: spec-offline
description: 功能下线，逆向清理代码和文档
---

# /spec-offline - 功能下线

## 功能

安全下线功能，彻底清理相关代码和文档。

## 使用方式

```
/spec-offline
/spec-offline {功能名称}
```

## 执行流程

```
Stage 3: Spec Modeling（标记删除） → Stage 4: Artifact Derivation（逆向清理）
```

## 启动提示

```
🗑️ 启动功能下线

将按以下流程执行：
1. 分析下线影响范围
2. 标记要删除的状态、不变量、用例
3. 生成清理清单（接口、代码、配置）

请提供：
1. 要下线的功能名称
2. 下线原因
```

## 关联的 Agent

- Stage 3: `agents/senior-domain-architect/AGENT.md` (规格建模部分)
- Stage 4: `agents/senior-fullstack-engineer/AGENT.md`

## 关联的 Skill

- `skills/for-spec-modeler/SKILL.md`
- `skills/for-fullstack-engineer/SKILL.md`

## 关联的 Workflow

- `skills/for-project-manager/workflows/feature-deprecation.md`
