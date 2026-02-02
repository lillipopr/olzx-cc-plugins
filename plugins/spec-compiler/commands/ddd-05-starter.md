---
description: 调用启动器设计Skill，根据《应用服务设计》文档生成《启动器设计》文档。
calls-skill: spec-compiler:ddd-05-starter
---

# /ddd-05-starter - 启动器设计命令

此命令调用 **ddd-05-starter** Skill，从《应用服务设计》文档生成《启动器设计》文档。

## 使用场景

- 已有《应用服务设计》文档，需要生成《启动器设计》
- 需要设计模块自动装配
- 需要设计依赖注入配置
- 需要设计启动生命周期

## 技术说明

**Command 类型**：直接调用 Skill（非 Agent）

**Skill 路径**：`{CLAUDE_PLUGIN_ROOT}/skills/ddd-05-starter/SKILL.md`

**优势**：
- 更轻量：无需启动独立 Agent 进程
- 更快速：直接在当前会话中执行
- 更可控：Skill 包含完整流程指引，便于调试和优化
