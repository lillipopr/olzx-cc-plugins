# Hooks 开发规范

## 概述

Hooks 是在特定事件触发时自动执行的脚本，用于实现自动化工作流和质量控制。

## Hook 类型

Claude Code 支持以下 Hook 类型：

| Hook 类型 | 触发时机 | 用途 |
|-----------|----------|------|
| `PreToolUse` | 工具执行前 | 验证、参数修改、权限检查 |
| `PostToolUse` | 工具执行后 | 自动格式化、检查、通知 |
| `SessionStart` | 会话开始 | 初始化、上下文加载 |
| `SessionEnd` | 会话结束 | 清理、持久化、审计 |

## 目录结构

```
.claude-plugin/hooks/
├── pre-tool-use/              # PreToolUse hooks
│   ├── phase-review-check.md
│   ├── architecture-check.md
│   └── document-validation.md
├── post-tool-use/             # PostToolUse hooks
│   ├── auto-format.md
│   └── spec-consistency-check.md
├── session-start/             # SessionStart hooks
│   └── init-context.md
└── session-end/               # SessionEnd hooks
    └── cleanup-temp.md
```

## Hook 配置

在 `plugin.json` 中声明 Hooks：

```json
{
  "hooks": {
    "preToolUse": [
      {
        "name": "phase-review-check",
        "description": "Phase 完成后强制人工审查",
        "script": "./.claude-plugin/hooks/pre-tool-use/phase-review-check.md",
        "enabled": true,
        "priority": "critical",
        "conditions": {
          "tools": ["Write", "Edit"],
          "filePatterns": ["**/*.spec.md", "**/PRD.md"]
        }
      }
    ],
    "postToolUse": [
      {
        "name": "spec-consistency-check",
        "description": "检查规格文档与实现的一致性",
        "script": "./.claude-plugin/hooks/post-tool-use/spec-consistency-check.md",
        "enabled": true
      }
    ],
    "sessionStart": [
      {
        "name": "init-context",
        "description": "初始化规格编译上下文",
        "script": "./.claude-plugin/hooks/session-start/init-context.md",
        "enabled": true
      }
    ],
    "sessionEnd": [
      {
        "name": "audit-changes",
        "description": "审计所有变更",
        "script": "./.claude-plugin/hooks/session-end/audit-changes.md",
        "enabled": true
      }
    ]
  }
}
```

## Hook 编写规范

### Hook 文件格式

每个 Hook 是一个 Markdown 文件，包含以下部分：

```markdown
# {Hook 名称}

## 元数据

- **类型**: PreToolUse / PostToolUse / SessionStart / SessionEnd
- **优先级**: critical / high / medium / low
- **启用状态**: true / false

## 触发条件

描述何时触发此 Hook。

## 执行逻辑

1. 步骤一
2. 步骤二
3. 步骤三

## 阻止条件

如果需要阻止操作继续，描述阻止条件。

## 输出

描述 Hook 的输出或副作用。
```

### PreToolUse Hook 示例

```markdown
# Phase Review Check

## 元数据

- **类型**: PreToolUse
- **优先级**: critical
- **启用状态**: true

## 触发条件

当用户尝试编辑规格文档（`*.spec.md` 或 `PRD.md`）的下一 Phase 内容时，且当前 Phase 尚未标记为 "Reviewed"。

## 执行逻辑

1. 检查文档中是否存在当前 Phase 的 `<!-- REVIEW STATUS: xxx -->` 标记
2. 如果状态不是 "APPROVED"，阻止编辑
3. 提示用户先完成审查

## 阻止条件

- 当前 Phase 未完成人工审查
- 文档中缺少审查状态标记

## 输出

- 如果阻止：返回错误消息，说明需要先完成审查
- 如果放行：无输出
```

### PostToolUse Hook 示例

```markdown
# Spec Consistency Check

## 元数据

- **类型**: PostToolUse
- **优先级**: medium
- **启用状态**: true

## 触发条件

当用户编辑代码文件（`*.java`, `*.swift`, `*.vue`, `*.ts`）后。

## 执行逻辑

1. 解析文件引用的规格文档路径（从注释中提取）
2. 读取规格文档
3. 检查代码实现是否与规格一致
4. 记录不一致的地方

## 输出

- 如果发现不一致：在控制台输出警告
- 如果一致：无输出
```

## Spec Compiler Kit 核心 Hooks

### 1. Phase 审查闸口 (Critical)

**文件**: `pre-tool-use/phase-review-check.md`

**目的**: 确保 4 Phase 流程的每个 Phase 完成后必须经过人工审查才能继续。

**触发条件**:
- 尝试编辑 Phase N+1 内容时
- Phase N 未标记为 "APPROVED"

**阻止逻辑**:
```
如果 Phase N 状态 != "APPROVED":
    阻止编辑
    显示: "请先完成 Phase N 的人工审查，标记为 APPROVED 后再继续"
```

### 2. 架构分层检查 (High)

**文件**: `pre-tool-use/architecture-check.md`

**目的**: 确保代码遵循指定的架构分层规范。

**触发条件**:
- 编辑 Java 文件时，检查 DDD 分层
- 编辑 iOS 文件时，检查 MVVM 分层
- 编辑 Vue 文件时，检查前端分层

**验证规则**:
- Java: Controller → Application → Domain ← Gateway
- iOS: View → ViewModel → Service → Gateway → Network
- Vue: View → Composable → Service → API → Request

### 3. 文档完整性校验 (High)

**文件**: `pre-tool-use/document-validation.md`

**目的**: 确保规格文档包含所有必需章节。

**检查项**:
- 实体定义是否包含唯一 ID、生命周期、状态转移
- 接口定义是否包含入参、返回值、用例、约束、状态转移、错误处理
- 不变量是否完整定义

### 4. 规格一致性检查 (Medium)

**文件**: `post-tool-use/spec-consistency-check.md`

**目的**: 检查代码实现是否与规格文档一致。

**检查项**:
- 实体属性是否匹配
- 状态转移是否正确实现
- 不变量是否被保护

### 5. 自动格式化 (Low)

**文件**: `post-tool-use/auto-format.md`

**目的**: 代码编辑后自动格式化。

**触发**: Write、Edit 操作后

## Hook 开发最佳实践

### 1. 优先级设计

- **Critical**: 必须执行，阻止操作
- **High**: 强烈推荐执行，可配置是否阻止
- **Medium**: 警告级别，不阻止
- **Low**: 信息级别，仅记录

### 2. 性能考虑

- Hook 应快速执行（< 1s）
- 避免重复计算
- 使用缓存优化

### 3. 错误处理

```markdown
## 错误处理

如果检查失败：
1. 记录错误日志
2. 根据 priority 决定是否阻止
3. 提供可操作的错误消息
```

### 4. 可配置性

在 `plugin.json` 中提供 Hook 配置选项：

```json
{
  "hooks": {
    "config": {
      "phaseReviewRequired": true,
      "architectureCheckStrict": false,
      "autoFormatEnabled": true
    }
  }
}
```

## Hook 调试

### 查看日志

Hook 执行日志会输出到 Claude Code 的控制台。

### 测试 Hook

创建测试场景验证 Hook 行为：

```bash
# 测试 Phase 审查闸口
1. 创建测试规格文档
2. 尝试跳过 Phase 1 直接编辑 Phase 2
3. 验证 Hook 是否阻止
```

## Hook 版本管理

- Hook 使用语义化版本控制
- 重大变更需要升级主版本号
- 在变更日志中记录 Hook 修改

## 常见问题

### Q: Hook 阻止了正常操作，如何绕过？

A: 在 `plugin.json` 中设置 Hook `enabled: false`，或调整优先级。

### Q: 如何添加自定义 Hook？

A: 在对应的 `pre-tool-use/` 或 `post-tool-use/` 目录创建 Hook 文件，并在 `plugin.json` 中声明。

### Q: Hook 可以访问哪些信息？

A: Hook 可以访问工具调用的上下文，包括文件路径、编辑内容、会话状态等。

