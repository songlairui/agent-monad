---
name: agent-bridge
description: "Monad bind (>>=): 从当前 Agent 的工作中提取意图和进度，更新 .agent-monad/ 上下文胶囊，使下一个 Agent 能无缝接续。当用户要切换 Agent、结束当前会话、或说 'bridge'、'交接'、'切换到 X'、'保存进度'、'我要换个工具' 时触发。"
---

# agent-bridge — Monad Bind (>>=)

> `bind :: AgentReady(a) → (a → AgentReady(b)) → AgentReady(b)`

将 Agent A 的工作成果解包，更新标准化上下文，再包装给 Agent B。
就像 `promise.then(fn)` —— 自动处理解包和重新包装，你只关心转换逻辑。

## 触发条件

- 用户说"交接"、"bridge"、"切换到 [Agent]"、"我要用 Cursor 了"
- 用户说"保存进度"、"记录一下当前状态"
- 当前会话即将结束（上下文窗口接近容量）
- 用户说"我要换个工具继续"

## 前置条件

- `.agent-monad/` 目录必须存在
- 如果不存在，先运行 `agent-init`

## 工作流程

### Phase 1: 收割当前 Agent 的工作成果

从当前会话和项目状态中提取：

```
1. 意图变化
   → 用户在会话中是否修正/扩展/缩小了意图？
   → 与 intent.md 中的描述是否一致？

2. 完成的工作
   → git diff --stat（对比 init 时的 commit）
   → 新增/修改/删除了哪些文件
   → 关键决策和 trade-off（从对话历史提取）

3. 未完成的工作
   → 进行中但未提交的变更
   → 已知但未修复的问题
   → 计划中但未开始的步骤

4. 发现的信息
   → 会话中发现的项目约定（conventions.md 中没有的）
   → 踩过的坑、绕过的问题
   → 对后续工作者有价值的上下文

5. 当前 Agent 原生配置的变化
   → CLAUDE.md / .cursorrules / codex.md 是否被更新？
   → 回写到 conventions.md
```

### Phase 2: 更新 `.agent-monad/`

#### 更新 `intent.md`

仅在意图确实发生变化时更新。保留原始意图作为历史：

```markdown
# Intent

## 当前意图
[更新后的意图，如果变了]

## 长期目标
[不变 / 更新]

## 意图演化
- [时间] 初始意图: ...
- [时间] 修正为: ... （原因: ...）
```

#### 追加 `progress.jsonl`

```jsonl
{"ts":"[ISO8601]","event":"bridge","agent":"claude-code","target":"cursor","summary":"完成了 API 路由重构，切换到 Cursor 继续前端工作","completed":["src/api/ 路由从 Express 迁移到 Hono","添加了 3 个集成测试"],"pending":["前端组件需要适配新 API","需要更新 API 文档"],"decisions":[{"what":"选择 Hono 而非 Fastify","why":"bundle size 更小，Cloudflare Workers 兼容"}],"blockers":[],"discoveries":["数据库连接池在并发 >50 时需要调优"]}
```

#### 更新 `conventions.md`

追加会话中发现的新约定，不删除已有的。

#### 更新 `handoff.md`

```markdown
# Handoff State

## 最后操作
- Agent: claude-code
- 时间: [ISO8601]
- 做了什么: [摘要]

## 当前状态
- 分支: feature/api-refactor
- 未提交变更: 2 个文件（src/api/routes.ts, tests/api.test.ts）
- 未完成工作:
  - [ ] 前端组件适配新 API
  - [ ] API 文档更新
- 阻塞项: 无

## 关键决策记录
- 选择 Hono 而非 Fastify（bundle size）

## 踩坑记录
- 数据库连接池并发 >50 时需要调优

## 给下一个 Agent 的建议
1. 先看 src/api/routes.ts 了解新 API 结构
2. 前端 fetch 调用都在 src/lib/api-client.ts
3. 运行 `pnpm test:api` 确认 API 层没问题再改前端
```

### Phase 3: 为目标 Agent 生成适配配置

如果用户指定了目标 Agent，生成对应的原生配置：

```
Claude Code  → CLAUDE.md（从 .agent-monad/ 生成）
Cursor       → .cursorrules（从 .agent-monad/ 生成）
Codex        → codex.md（从 .agent-monad/ 生成）
Aider        → .aider.conf.yml（从 .agent-monad/ 生成）
通用         → 只更新 .agent-monad/，不生成特定配置
```

每个生成的配置文件顶部标注：

```
<!-- Auto-generated from .agent-monad/ by agent-bridge. Source of truth: .agent-monad/ -->
```

### Phase 4: 确认

```
✅ Bridge 完成: claude-code → cursor

已收割:
  - 完成: 2 项（API 路由重构 + 集成测试）
  - 未完成: 2 项（已记录到 handoff.md）
  - 决策: 1 项（Hono over Fastify）
  - 发现: 1 项（连接池调优）

已更新:
  - .agent-monad/progress.jsonl  (+1 记录)
  - .agent-monad/handoff.md      (重写)
  - .cursorrules                  (为 Cursor 生成)

下一步: 在 Cursor 中打开项目，它会从 .cursorrules 获取完整上下文。
```

## Monad Law 检验

**左单位元**: `agent-init` 后立刻 `agent-bridge` 到同一 Agent → `.agent-monad/` 不变（除了多一条 progress 记录），等价于没操作。

**右单位元**: `agent-bridge` 回自身 → 等价于"保存进度"，不改变工作状态。
