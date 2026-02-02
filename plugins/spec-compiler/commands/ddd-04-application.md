---
description: 调用应用服务设计Skill，根据《聚合设计》《领域服务设计》文档生成《应用服务设计》文档。
calls-skill: spec-compiler:ddd-04-application
---

# /ddd-04-application - 应用服务设计命令

此命令调用 **ddd-04-application** Skill，从《聚合设计》《领域服务设计》文档生成《应用服务设计》文档。

## 使用场景

- 已有《聚合设计》《领域服务设计》文档，需要生成《应用服务设计》
- 需要识别应用服务（用例编排）
- 需要设计事务边界和异常处理
- 需要设计 DTO 和接口定义

## 技术说明

**Command 类型**：直接调用 Skill（非 Agent）

**Skill 路径**：`{CLAUDE_PLUGIN_ROOT}/skills/ddd-04-application/SKILL.md`

**优势**：
- 更轻量：无需启动独立 Agent 进程
- 更快速：直接在当前会话中执行
- 更可控：Skill 包含完整流程指引，便于调试和优化
