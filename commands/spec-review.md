---
name: spec-review
description: 审查规格文档，检查完整性和一致性
---

# /spec-review - 规格文档审查

## 功能

审查已有的规格文档，检查完整性、一致性，发现潜在问题。

## 使用方式

```
/spec-review                    # 交互式选择要审查的文档
/spec-review {文档路径}          # 审查指定文档
/spec-review --all {目录}       # 审查目录下所有规格文档
```

## 执行流程

1. **收集文档**
   - 识别文档类型
   - 加载文档内容

2. **执行审查**
   - 单文档审查
   - 跨文档一致性审查

3. **输出报告**
   - 按严重级别排序
   - 给出修复建议

## 启动提示

```
🔍 启动规格文档审查

请提供要审查的文档：
1. 直接粘贴文档内容
2. 提供文档路径
3. 提供包含规格文档的目录

支持审查的文档类型：
- PRD（产品需求文档）
- DDD Design（领域设计文档）
- Spec Modeling（规格建模文档）
- Artifact Derivation（工件推导文档）
```

## 审查维度

| 文档类型 | 审查重点 |
|---------|---------|
| PRD | 需求清晰度、验收标准可测性 |
| DDD | 聚合边界、实体/值对象区分 |
| 规格建模 | 状态完备性、Bad Case 存在性 |
| 工件推导 | 契约一致性、位置明确性 |

## 关联的 Agent

- `agents/senior-project-manager/AGENT.md`

## 关联的 Skill

- `skills/for-project-manager/SKILL.md`

## 关联的 Workflow

- `skills/for-project-manager/workflows/spec-review.md`
