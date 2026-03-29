---
name: agent-merge
description: "Monad join (μ): 当多个 Agent 并行处理同一项目的不同部分后，合并它们的上下文胶囊为统一状态。当用户说 'merge'、'合并'、'汇总进度'、'合并分支'，或多个 Agent 的 .agent-monad/ 需要协调时触发。"
---

# agent-merge — Monad Join (μ)

> `join :: AgentReady(AgentReady(a)) → AgentReady(a)`

将嵌套的 monadic context 展平为一层。
对应场景：多个 Agent 并行工作于同一项目的不同模块后，合并它们的成果。

## 触发条件

- 用户说"merge"、"合并进度"、"汇总"
- 多个分支/worktree 各自有 `.agent-monad/` 更新需要合并
- 用户说"A 做完了前端、B 做完了后端，现在合在一起"

## 典型场景

```
main
├── worktree-a/  (Claude Code 做 API)
│   └── .agent-monad/  ← 有独立的 progress + handoff
├── worktree-b/  (Cursor 做前端)
│   └── .agent-monad/  ← 有独立的 progress + handoff
└── .agent-monad/      ← 需要合并为统一状态
```

## 工作流程

### Phase 1: 收集所有 monadic contexts

```
1. 扫描来源
   → 当前项目的 .agent-monad/
   → git worktrees 的 .agent-monad/
   → 用户指定的其他路径
   → git branches 中的 .agent-monad/（通过 git show branch:.agent-monad/）

2. 为每个来源构建快照
   → 最后一条 progress.jsonl 记录
   → handoff.md 的当前状态
   → conventions.md 的差异
   → intent.md 的差异
```

### Phase 2: 冲突检测

检查各来源之间的不一致：

```
⚠️  冲突类型：

1. 意图分歧
   → 两个 Agent 对 intent.md 的理解产生了偏差
   → 标记为需要用户裁决

2. 约定冲突
   → Agent A 在 conventions.md 中记录"用 tabs"
   → Agent B 记录"用 spaces"
   → 标记为需要用户裁决

3. 进度重叠
   → 两个 Agent 修改了相同文件
   → 对应 git merge conflict
   → 先解决代码冲突，再合并 .agent-monad/

4. 决策矛盾
   → Agent A 决定用库 X，Agent B 决定用库 Y 解决同一问题
   → 标记为需要用户裁决
```

### Phase 3: 合并

#### `progress.jsonl`

所有来源的记录按时间戳排序合并，追加一条 merge 事件：

```jsonl
{"ts":"[ISO8601]","event":"merge","agent":"claude-code","summary":"合并 worktree-a (API) 和 worktree-b (前端) 的进度","sources":["worktree-a","worktree-b"],"conflicts":0}
```

#### `intent.md`

- 无分歧 → 保留最新版本
- 有分歧 → 列出两版，请用户选择或重新表述

#### `conventions.md`

- 无冲突 → 合并（取并集）
- 有冲突 → 标记冲突段，请用户裁决：

```markdown
## 代码风格

<<<<<<< worktree-a (Claude Code)
- Indentation: tabs
=======
- Indentation: 2 spaces
>>>>>>> worktree-b (Cursor)

<!-- 请解决此冲突 -->
```

#### `handoff.md`

重新生成，合并所有来源的状态：

```markdown
# Handoff State

## 合并来源
- worktree-a (Claude Code): API 层重构，已完成
- worktree-b (Cursor): 前端组件适配，已完成

## 合并后状态
- 分支: main (合并后)
- 未完成工作:
  - [ ] 端到端集成测试
  - [ ] 部署配置更新
- 冲突解决记录:
  - [无 / 列表]

## 合并决策记录
[合并过程中做的选择]
```

### Phase 4: 验证

```
1. 代码层面
   → git status 是否干净
   → 测试是否通过
   → lint 是否通过

2. 上下文层面
   → .agent-monad/ 中是否有未解决的冲突标记
   → intent.md 是否一致
   → conventions.md 是否自洽
```

### Phase 5: 报告

```
✅ Merge 完成: 2 个来源 → 统一状态

来源:
  - worktree-a (Claude Code): 15 条 progress 记录
  - worktree-b (Cursor): 8 条 progress 记录

合并结果:
  - progress.jsonl: 24 条记录（23 历史 + 1 merge）
  - conventions.md: 合并完成，无冲突
  - intent.md: 一致，保留最新
  - handoff.md: 已重新生成

⚠️  需要注意:
  - 2 个未完成项已汇总到 handoff.md
  - 建议运行端到端测试确认集成

下一步: 项目已回到统一状态，可以用任何 Agent 继续。
```

## Monad Law 检验

**结合律**: `(A >>= bridge-to-B) >>= bridge-to-C` 等价于 `A >>= (λx → bridge-to-B(x) >>= bridge-to-C)`。

工程含义：先从 Claude 切到 Cursor 再切到 Codex，和先从 Claude 切到 Cursor+Codex 的流水线，最终 `.agent-monad/` 的状态一致（除了 progress.jsonl 的中间记录）。
