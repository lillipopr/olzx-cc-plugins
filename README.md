# Spec Compiler Kit

规格编译器套件：将模糊需求编译为确定性规格文档，实现"人管变化，AI 写实现"。

## 核心理念

```
传统模式：需求 → 人写代码 → 测试 → 交付
新范式：  需求 → 人写文档 → AI 编译代码 → 测试 → 交付
```

- **人负责**：需求定义、领域设计、规格审核、验收
- **AI 负责**：规格建模、工件推导、代码生成、测试实现
- **代码是可替换工件，规格 + 用例才是资产**

## 完整编排流程

```
┌─────────────────────────────────────────────────────────────────┐
│                         完整功能建设流程                          │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  Stage 1: PRD                                                   │
│  ├─ 执行者: PM                                                   │
│  ├─ 产出: 《产品需求文档》                                         │
│  └─ 闸口: 需求清晰、边界明确、验收标准可测                           │
│           ↓                                                     │
│  Stage 2: DDD Design                                            │
│  ├─ 执行者: 架构师                                                │
│  ├─ 产出: 《DDD 设计文档》                                         │
│  └─ 闸口: 领域边界清晰、聚合设计合理、上下文映射完整                  │
│           ↓                                                     │
│  Stage 3: Spec Modeling                                         │
│  ├─ 执行者: AI + 人工审核                                         │
│  ├─ 产出: 《规格建模文档》                                         │
│  └─ 闸口: 状态完备、不变量完整、用例覆盖（含 Bad Case）              │
│           ↓                                                     │
│  Stage 4: Artifact Derivation                                   │
│  ├─ 执行者: AI + 人工审核                                         │
│  ├─ 产出: 《工件推导文档》                                         │
│  └─ 闸口: 分层清晰、契约一致、可直接编码                            │
│           ↓                                                     │
│  Stage 5: Test Generation                                       │
│  ├─ 执行者: AI + 人工审核                                         │
│  ├─ 产出: 《测试代码》                                            │
│  └─ 闸口: 测试覆盖所有用例、TDD RED 阶段就绪                        │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

## 插件结构

```
spec-compiler-kit/
├── .claude-plugin/
│   └── plugin.json              # 插件元数据
├── README.md                     # 项目说明
├── commands/                     # 【Command 层】用户命令入口
│   ├── spec.md                  # 主命令（智能路由）
│   ├── spec-new.md              # 首次功能建设
│   ├── spec-iter.md             # 功能迭代
│   ├── spec-fix.md              # Bug 修复
│   ├── spec-offline.md          # 功能下线
│   └── spec-review.md           # 规格审查
├── agents/                       # 【Agent 层】按角色命名的执行代理
│   ├── senior-product-manager/  # 资深产品经理
│   ├── senior-domain-architect/ # 资深领域架构师
│   ├── senior-fullstack-engineer/ # 资深全栈工程师
│   ├── senior-test-engineer/    # 资深测试工程师
│   ├── senior-project-manager/  # 资深项目经理
│   ├── senior-java-expert/      # Java 后端专家
│   ├── senior-ios-expert/       # iOS 开发专家
│   ├── senior-frontend-expert/  # 前端开发专家
│   └── senior-database-expert/  # 数据库专家
├── skills/                       # 【Skill 层】按角色配备的知识库
│   ├── for-product-manager/     # 产品经理知识库
│   ├── for-domain-architect/    # 领域架构师知识库
│   ├── for-spec-modeler/        # 规格建模知识库
│   ├── for-fullstack-engineer/  # 全栈工程师知识库
│   ├── for-test-engineer/       # 测试工程师知识库
│   ├── for-project-manager/     # 项目经理知识库
│   ├── for-java-expert/         # Java 专家知识库
│   ├── for-ios-expert/          # iOS 专家知识库
│   ├── for-frontend-expert/     # 前端专家知识库
│   └── for-database-expert/     # 数据库专家知识库
└── tools/                        # 【Tools 层】工具封装
    ├── file-tools/              # 文件操作工具
    ├── search-tools/            # 搜索工具
    └── validation-tools/        # 验证工具
```

### 各层职责说明

| 层级 | 目录 | 职责 | 关系 |
|------|------|------|------|
| **Command 层** | `commands/` | 面向用户的 `/spec-*` 命令入口 | 路由到合适的 Agent |
| **Agent 层** | `agents/` | 实际执行任务、多次 tool 调用、决策、状态管理 | 按角色命名，引用对应 Skill |
| **Skill 层** | `skills/` | 提供领域知识、方法论、SOP、模板、设计模式、原则 | 按角色配备 |
| **Tools 层** | `tools/` | 读写文件、搜索、执行命令等 | 主动操作资源 |

## 使用方式

### 安装

```bash
# 方法 1：符号链接（推荐，本地开发）
ln -s /Users/zxq/data/github/spec-compiler-kit ~/.claude/plugins/local/spec-compiler-kit

# 方法 2：使用 /plugin install 命令
/plugin install local /Users/zxq/data/github/spec-compiler-kit
```

### 命令

```bash
/spec              # 交互式选择场景
/spec new          # 首次功能建设
/spec iter         # 功能迭代
/spec fix          # Bug 修复
/spec offline      # 功能下线
/spec review       # 审查规格文档
```

## 场景支持

| 场景 | 命令 | 执行阶段 | 说明 |
|------|------|---------|------|
| **首次功能建设** | `/spec-new` | 1 → 2 → 3 → 4 | 完整流程 |
| **功能迭代** | `/spec-iter` | 2 → 3 → 4 | 基于已有 PRD 增量更新 |
| **Bug 修复** | `/spec-fix` | 3 → 4 | 定位问题 → 补充用例 → 推导修复 |
| **功能下线** | `/spec-offline` | 3 → 4（逆向） | 删除用例 → 删除工件 → 清理代码 |

## 支持架构

| 架构类型 | 分层结构 | 规范文件 |
|---------|---------|---------|
| **后端 DDD** | Controller → Application → Domain → Gateway/Infra → Mapper | `java-ddd-layers.md` |
| **iOS MVVM** | View → ViewModel → Service → Gateway → Network | `ios-mvvm-layers.md` |
| **Vue 3** | View → Composable → Service → API → Request | `vue3-layers.md` |

## 核心原则

### 闸口控制

每个 Stage 完成后**必须等待用户确认**：

```
Stage N 完成 → 输出产出物 → 提醒用户 Review → 等待确认 → 进入 Stage N+1
```

**禁止行为**：
- 不能在用户未确认时自动进入下一 Stage
- 不能跳过 Review 闸口
- 不能假设用户已同意而继续执行

**必须行为**：
- 每个 Stage 完成后明确提醒用户 Review
- 清晰列出 Review 要点
- 等待用户明确的正面确认
- 修改后再次提醒 Review

### 质量闸口

| 闸口 | 检查点 | 不通过则 |
|------|--------|----------|
| **PRD 闸** | 需求清晰、边界明确 | 不进入 DDD 设计 |
| **DDD 闸** | 领域边界清晰、聚合合理 | 不进入规格建模 |
| **建模闸** | 状态完备、不变量完整、用例覆盖 | 不进入工件推导 |
| **工件闸** | 分层清晰、契约一致 | 不允许生成代码 |
| **熵控闸** | 功能下线时删 case + 删代码 | 不允许遗留死代码 |

## 快速开始示例

**输入**：用户订阅会员，每天刷新 100 点券

**Stage 1 PRD**：
- 功能描述、用户故事、验收标准

**Stage 2 DDD**：
- 聚合根：Membership
- 实体：CouponGrant
- 限界上下文：会员上下文、点券上下文

**Stage 3 Spec Modeling**：
- 状态：M0(非会员) → M1(生效中) → M2(已过期)
- 不变量：INV-1 只有 M1 才能发放点券
- 用例：覆盖所有状态转移 + Bad Case

**Stage 4 Artifact Derivation**：
- 后端：Controller/Application/Domain/Gateway 分层
- 接口契约：API 定义、DTO、错误码
- 实现位置映射

## 文件说明

### Agents（执行代理）

- `spec-prd-agent.md` - PRD 产出代理
- `spec-ddd-agent.md` - DDD 设计代理
- `spec-modeling-agent.md` - 规格建模代理
- `spec-artifact-agent.md` - 工件推导代理
- `spec-test-agent.md` - 测试生成代理
- `spec-review-agent.md` - 规格审查代理

### Commands（用户命令）

- `spec.md` - 主入口（智能路由）
- `spec-new.md` - 首次功能建设
- `spec-iter.md` - 功能迭代
- `spec-fix.md` - Bug 修复
- `spec-offline.md` - 功能下线
- `spec-review.md` - 审查规格文档

### Skills（技能库层）

#### spec-compiler/
- `SKILL.md` - 入口：理念 + 流程编排
- `rules/` - 核心原则
- `workflows/` - 完整工作流（按场景）
- `stages/` - 阶段详解（01-prd ~ 05-test-generation）
- `methodology/` - 方法论文档（不变量、实体提取、状态空间设计等）
- `domain/` - 领域特化
- `templates/` - 文档模板

## 许可证

MIT License
