# Paperclip Pages · 俯瞰式上报看板

> 草稿状态：构思阶段（口述还原 + 产品草案）
> 关联：`skills/harness-graph`、`skills/sancai-zhen` 的"天位（全景）"延伸

---

## 一、口述意图还原

原始口述（语音转写，含识别误差）：

> 我想到了使用 Paper Clip 的另一个方向，就是自己制作一些技能，在工具当中直接使用。在 CodeX，在 Cloze Code 的 plug in 当中，上报我正在做的事情，来形成一个全局的视角。然后全局视角的看板的设计，就可以直接参考 Paper Clip 的性质模型。当然也可以做一些变种，Paper Clip 是公司模型，而我的视角就可以根据自己上报的内容，然后展开它的上下文信息，对，On the same page，Pages。我的俯瞰视角可以用 Pages 这个模型。那就可以无极变化的纵览视角。其实我所需要的更多的是一个俯瞰的视角。

**校正后的准确表达：**

我想到了 *Universal Paperclips*（回形针游戏）的另一种应用方向 —— 不是做游戏，而是借用它的"属性面板/资源看板"作为信息架构的范式：

1. **载体**：自己制作一批 skill，直接在已有工具里使用 —— 在 **Codex** 里、在 **Claude Code 的 plugin** 里。
2. **机制**：这些 skill 的核心动作是**上报**（报告"我现在在做什么"）。每个工具实例都成为一个上报源。
3. **聚合**：所有上报汇聚到一个**全局看板**，形成跨工具、跨项目的全局视角。
4. **看板范式**：直接参考 Paperclips 的**属性模型**（左栏一组属性，右栏分桶 / 阶段进展 / 资源数值的紧凑面板）。
5. **变体 —— 从"公司模型"到"Pages 模型"**：
   - Paperclips 是 **Company Model**：所有属性都汇入"一家公司"这个单一聚合体。
   - 我的视角是 **Pages Model**：上报内容根据上下文展开，落在不同的 Page 上。"On the same page" 的双关 —— 既是字面的"同一页"，也是协同的"同频"。
   - Pages 之间可以**无极变化地纵览**：缩放、合并、切片、聚焦。
6. **真正的需求**：我要的不是更细的执行视图，而是一个**俯瞰视角（bird's-eye view）** —— 看见整体在做什么，比看见某一处怎么做更重要。

---

## 二、产品草稿

### 2.1 名字（候选）

- **Paperclip Pages**（直白，保留致敬感）
- **Overlook**（强调"俯瞰"）
- **On the Same Page**（强调对齐与共识）

下文暂用 **Paperclip Pages**。

### 2.2 产品形态一句话

> 一组装在 Codex / Claude Code 等工具里的上报 skill，把分散的 AI 工作流上报成 Paperclips 风格的属性面板，再以 Pages 模型组织成可无限缩放的俯瞰看板。

### 2.3 三层结构

```
┌─────────────────────────────────────────────────────────┐
│  Layer 3 · OVERLOOK                俯瞰层（Pages 编织）  │
│  ── 多 Page 共存、缩放、聚合，"无极变化的纵览视角"        │
├─────────────────────────────────────────────────────────┤
│  Layer 2 · PAGE                    单页面板（Paperclips） │
│  ── 一个 Page = 一组属性 + 桶 + 进度，紧凑信息密度        │
├─────────────────────────────────────────────────────────┤
│  Layer 1 · REPORT                  上报源（Skill）        │
│  ── 装在 Codex / Claude Code 里，吐出结构化事件           │
└─────────────────────────────────────────────────────────┘
```

### 2.4 Layer 1 · Report Skill

每个工具实例装一个上报 skill，行为契约：

- **触发**：每次工具开始/切换/完成一个动作时自动调用，也可被 AI 主动调用。
- **输出**：一条结构化事件 —— 谁、在哪、做什么、置信度、关联 Page。
- **运输**：写入本地 jsonl 文件 + 可选推送到聚合服务。

事件 schema（草案）：

```json
{
  "ts": "2026-04-19T10:00:00Z",
  "source": "claude-code | codex | cursor | ...",
  "session_id": "...",
  "page_hint": "auto | <page-slug>",
  "verb": "start | progress | block | finish | seed",
  "what": "迁移通知模块到事件总线",
  "where": "/repo/src/notify/",
  "metrics": { "files_touched": 3, "tests_passing": 12 },
  "confidence": 0.82,
  "links": ["thread:xxx", "issue:#123"]
}
```

**与现有 skill 的关系**：复用 `harness-graph` 的种子/节点模型；上报事件 ≈ 自动播种到 graph，再被 Page 渲染。

### 2.5 Layer 2 · Page（Paperclips 范式）

一个 Page 是一个**紧凑属性面板**，借 Paperclips 的视觉密度：

```
┌── Page · 通知模块重构 ──────────────────────────────┐
│  Intent     重构通知模块，直接调用 → 事件总线         │
│  Phase      [■■■■■■■□□□] 7/10                        │
│  Confidence 82%                                      │
│  ──────────────────────────────────────────────────  │
│  Resources                  Buckets                  │
│   files_touched   17         scanned    7 calls      │
│   tests_passing   42/45      migrated   3 / 7        │
│   blocks_open     1          pending    4            │
│  ──────────────────────────────────────────────────  │
│  Live Sources                                        │
│   ● claude-code (sess#A2)  · 迁移 UserService        │
│   ○ codex       (sess#C1)  · 草拟事件 schema         │
│  ──────────────────────────────────────────────────  │
│  Seeds (未归属)                                      │
│   🌱 优先级队列                                      │
└──────────────────────────────────────────────────────┘
```

设计要点：
- **左栏属性 / 右栏桶**沿用 Paperclips 的双栏密度。
- **Live Sources** 实时显示哪些工具实例正在为这个 Page 工作。
- **Seeds** 是尚未归属的零碎想法（对接 `sancai-zhen` 的"天/播种"）。
- 每个数字都是上报事件聚合而来 —— Page 本身**不存事实，只存视图**。

### 2.6 Layer 3 · Overlook（Pages 模型）

俯瞰层是 Pages 的编织界面，提供"无极纵览"：

| 操作 | 类比 | 说明 |
|------|------|------|
| **Zoom out** | Map → Continent | 多个 Page 聚成一张总图，每个 Page 收缩成一个胶囊 |
| **Zoom in** | Page → Section | 单个 Page 展开为属性 + 时间线 + 上报流 |
| **Slice** | 横切 | 跨 Page 拉出同一维度（如所有 `blocks_open`） |
| **Merge** | Page A + B → C | 两个 Page 合并为新 Page（对应 `agent-merge`） |
| **Split** | Page → Pages | 一个 Page 拆成多个子 Page |
| **Pin** | 固定 | 把某个 Page 钉在视野内 |

*"On the same page" 的双关在这里落地：* 当多个工具实例都在向同一个 Page 上报，它们就是字面意义上"在同一页上"。

### 2.7 与 Paperclips 的对照

| 维度 | Paperclips | Paperclip Pages |
|------|-----------|-----------------|
| 聚合体 | 一家公司 | 多个 Page |
| 属性 | 资源/产线/目标 | 意图/进度/桶/源 |
| 玩家 | 操控公司演化 | 俯瞰自己的 AI 工作流 |
| 时间感 | 长程指数增长 | 实时上报 + 历史快照 |
| 形态 | 单页递进 | 多页共存 + 缩放 |
| 终态 | 把宇宙变成回形针 | 让所有工作"on the same page" |

### 2.8 与本仓现有 skill 的关系

- `harness-graph` —— 提供底层图谱；上报事件即图谱节点。
- `sancai-zhen` —— 提供天/地/人路由；Page 是"天位"的可视化形态。
- `intent-anchor` —— 提供意图溯源；Page 顶部的 *Intent* 字段从这里来。
- `agent-bridge` / `agent-merge` —— Pages 的合并/迁移直接复用其语义。

新增的 skill（草拟）：

| Skill | 作用 |
|-------|------|
| `report-pulse` | 装在工具里的上报心跳；按事件输出 jsonl |
| `page-render` | 把上报流渲染成 Paperclips 式 Page |
| `overlook` | 俯瞰层 —— 多 Page 编织、缩放、切片 |

---

## 三、最小验证路径（MVP）

1. **写 `report-pulse` skill**：在 Claude Code 中跑 → 写本地 `.agent-monad/reports.jsonl`。
2. **写 `page-render` 命令**：读 jsonl → 在终端打印一个 Page 面板（ASCII 即可）。
3. **手工开两个工具实例**：观察是否能聚合到同一个 Page。
4. **再写 `overlook`**：列出多个 Page，支持缩放与切片。

只要前两步成立，"俯瞰视角"这个核心需求就已经有了最小骨架；剩下的都是放大。

---

## 四、待回答的问题

- Page 的归属是**自动推断**还是**用户钉**？（先自动 + 允许手动覆盖？）
- 上报频率：每条动作一报 vs 节流聚合？
- Overlook 是 TUI、Web，还是先做 Markdown 快照？
- 多机/多仓如何同步？本地 jsonl 够不够，还是需要一个轻量服务？
- 隐私边界：哪些上报字段绝不外发？
