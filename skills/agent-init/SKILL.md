---
name: agent-init
description: "Monad unit (η): 扫描当前项目状态，生成标准化 .agent-monad/ 上下文胶囊，使任何 AI Agent 都能无损接管。无论项目之前是否被 Agent 接管过、被哪个 Agent 接管过，都能归一化。当用户说 'init'、'接管这个项目'、'开始'、'onboard'、'新项目'，或首次在项目中使用 Agent 时触发。"
---

# agent-init — Monad Unit (η)

> `unit :: RawProjectState → AgentReady(ProjectState)`

无论项目处于什么状态，将其提升进标准化的 monadic context。
就像 `Promise.resolve(value)` —— 不管 value 是 raw 值还是已包装的 Promise，都归一化。

## 触发条件

- 用户说"接管这个项目"、"开始"、"init"、"onboard"
- 首次在一个项目中启动 Agent
- 项目中不存在 `.agent-monad/` 目录
- 用户从另一个 Agent 切换过来

## 工作流程

### Phase 1: 状态探测（幂等）

并行执行以下探测，构建项目指纹：

```
1. 已有 Agent 遗产检测
   → .claude/, CLAUDE.md, AGENTS.md
   → .cursor/, .cursorrules
   → codex.md, .codex/
   → .aider*, .continue/, .copilot/
   → .agent-monad/（如已存在则为 re-init）

2. 项目结构快照
   → 顶层两级目录树（忽略 node_modules, .git, dist, build, __pycache__）
   → 入口文件：main.*, index.*, app.*, server.*, cmd/

3. 技术栈指纹
   → 包管理器：package.json, go.mod, Cargo.toml, pyproject.toml, pom.xml
   → 框架：next.config.*, vite.config.*, django, flask, fastapi, rails
   → 语言版本约束

4. 约定推断
   → 代码风格配置：.eslintrc*, .prettierrc*, ruff.toml, .editorconfig
   → 测试框架与结构：tests/, __tests__/, *_test.go, *.spec.ts
   → CI/CD：.github/workflows/, .gitlab-ci.yml, Makefile

5. Git 状态
   → 当前分支、最近 10 条 commit message
   → 未提交变更摘要
   → remote URL
```

### Phase 2: 意图捕获

如果是全新项目（无历史 Agent 遗产），**向用户提问**：

```
我已扫描完项目，检测到 [技术栈摘要]。

在我生成上下文胶囊之前，请用一两句话告诉我：
1. 你现在想做什么？（当前意图）
2. 这个项目最终要达成什么？（长期目标，可选）
```

如果检测到已有 Agent 遗产，**从中提取意图**，然后向用户确认：

```
我从 [CLAUDE.md / .cursorrules / ...] 中提取了以下上下文：
- 项目描述：...
- 约定：...
- 当前状态：...

这些还准确吗？有需要更新的吗？
```

### Phase 3: 生成 `.agent-monad/`

创建或更新以下文件：

#### `intent.md`

```markdown
# Intent

## 当前意图
[用户说的 what & why]

## 长期目标
[可选]

## 约束
[从约定推断 + 用户补充]
```

#### `conventions.md`

```markdown
# Conventions

## 技术栈
- Language: [detected]
- Framework: [detected]
- Package Manager: [detected]

## 代码风格
[从 linter/formatter 配置推断]

## 项目结构
[目录用途映射]

## 测试
[框架、结构、运行命令]

## 构建与部署
[构建命令、CI/CD 概要]
```

#### `progress.jsonl`

```jsonl
{"ts":"[ISO8601]","event":"init","agent":"[当前agent]","summary":"项目初始化，生成上下文胶囊","detail":{"detected_agents":["claude","cursor"],"tech_stack":["typescript","nextjs"]}}
```

#### `handoff.md`

```markdown
# Handoff State

## 最后操作
- Agent: [当前 Agent 名称]
- 时间: [ISO8601]
- 做了什么: 项目初始化

## 当前状态
- 分支: [current branch]
- 未完成工作: [none / 描述]
- 阻塞项: [none / 描述]

## 下一步建议
[基于意图推断]
```

### Phase 4: 适配当前 Agent

根据当前运行的 Agent，**从 `.agent-monad/` 生成该 Agent 的原生配置**：

- 如果是 Claude Code → 生成/更新 `CLAUDE.md`
- 如果是 Cursor → 生成/更新 `.cursorrules`
- 如果是 Codex → 生成/更新 `codex.md`

生成时引用 `.agent-monad/` 作为 source of truth，而非独立维护。

## 幂等性保证

多次运行 `agent-init`：
- 如果 `.agent-monad/` 已存在且一致 → 跳过，只追加 `progress.jsonl`
- 如果项目状态有变化 → 更新差异部分，保留历史
- 绝不覆盖用户手动编辑的 `intent.md`

## 输出

完成后向用户报告：

```
✅ 上下文胶囊已生成 → .agent-monad/

  intent.md       — 你的意图
  conventions.md  — 项目约定
  progress.jsonl  — 进度日志（1 条记录）
  handoff.md      — 交接状态

检测到的已有 Agent 配置: [列表或"无"]
已为当前 Agent 生成适配配置: [文件名]
```
