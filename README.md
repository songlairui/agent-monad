# Agent Monad

> *A monad is just a monoid in the category of endofunctors.*
>
> 单子不过就是自函子范畴上的幺半群。

**Agent Monad** 提供三个原语 skill，让人类意图与 AI Agent 之间的切换具备**组合性** —— 就像 `Promise` 让异步操作可组合一样。

## 问题

每个 AI Agent（Claude Code、Cursor、Codex、Aider……）都是一个**自函子**：把"项目状态"映射回"项目状态"。但它们各自的内部表示不同（`CLAUDE.md` / `.cursorrules` / `codex.md`），导致：

- 切换 Agent 时上下文丢失
- 多 Agent 并行后无法合并
- 人类意图在翻译中走形

## 解法：三个 Monad 原语

| 操作 | Skill | Monad 对应 | 作用 |
|------|-------|-----------|------|
| **提升** | `agent-init` | `unit / return (η)` | 扫描项目现状，生成标准化上下文胶囊 `.agent-monad/` |
| **绑定** | `agent-bridge` | `bind / >>= ` | 从 Agent A 的遗产中提取意图+进度，转译给 Agent B |
| **合并** | `agent-merge` | `join / μ` | 多 Agent 并行工作后，合并上下文回统一状态 |

## 标准化中间表示：`.agent-monad/`

```
.agent-monad/
├── intent.md           # 人类意图（what & why，不是 how）
├── progress.jsonl      # Agent 无关的进度日志
├── conventions.md      # 从代码推断的项目约定
└── handoff.md          # 当前交接状态摘要
```

这个目录是 **monadic context** —— 包装器不是值本身，而是"意图 + 进度 + 约定"的标准化表示。

## Monad Laws 的工程含义

```
-- 左单位元：init 后立刻 bridge 到同一 Agent = 什么都没发生
unit a >>= f  ≡  f a

-- 右单位元：bridge 回自己 = 什么都没发生
m >>= unit  ≡  m

-- 结合律：A→B→C 和 (A→B)→C 结果相同，切换顺序无损
(m >>= f) >>= g  ≡  m >>= (λx → f x >>= g)
```

工程检验标准：**切换是否无损？顺序是否无关？**

## 安装

将 `skills/` 下的三个目录复制到你的 `~/.claude/skills/` 或项目的 `.claude/skills/` 中。

## License

MIT
