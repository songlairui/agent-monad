# Forewrite Graph — Schema 完整定义

## 文件位置

```
~/.forewrite/graph.json
```

跟 `~/.harness/graph.json` 完全分离，不互相引用。

---

## 顶层结构

```json
{
  "nodes": [...],
  "edges": [...]
}
```

无 `projects`、无 `matters`。forewrite 只装 record 节点。如未来需要把 record 关联到 harness 的 project/matter，由桥接层处理。

---

## Node Schema

```typescript
type ForewriteRecord = {
  id: string;              // 8-char hex，主键
  kind: "record";          // 当前唯一类型，预留扩展

  written_at: string;      // ISO 8601，本次 forewrite 写下的时刻
  mode: "light" | "heavy";

  trigger: string;         // 触发本次 forewrite 的用户原话片段（≤ 200 字，保留原话不 paraphrase）

  // 当下页
  now_page: {
    why: string;                 // 为什么想做这件事（一句话）
    current_state: string | null;  // 跟这件事相关的现状盘点；light 模式为 null
  };

  // 未来页
  future_page: {
    marker: string;              // 时间锚，例："一个月后的某个周二上午"
    scene: string;               // 一帧画面，应含具体感官细节
    uses: string[];              // 那一帧里你拿它做什么
    without_it: string | null;   // 没有它的时候你怎么做；light 模式为 null
  };

  // 交叉阅读
  cross_reading: {
    gap: string;                 // 两页之间差什么
    verdict: Verdict;
  };
};

type Verdict =
  | "already_there"     // 当下页已经写出未来页的内容
  | "minimum_action"    // 差一个最小动作
  | "real_project"      // 真要做的项目
  | "abandon"           // 未来页不吸引人
  | "outer_dream";      // 是更大 forewrite 的子页
```

### 字段约束

| 字段 | 约束 |
|---|---|
| `id` | 8 字符十六进制，建议从 uuid4 截前 8 位 |
| `written_at` | ISO 8601，到秒 |
| `trigger` | ≤ 200 字。**保留用户原话**，不要 paraphrase |
| `now_page.why` | 一句话，长度限制 ≤ 100 字 |
| `now_page.current_state` | light 模式必须为 `null`；heavy 模式必须为 string |
| `future_page.marker` | 必须含时间锚（"一个月后"、"半年后"） |
| `future_page.scene` | 自由文本，应含具体感官细节 |
| `future_page.uses` | array of string，每条一个用途 |
| `future_page.without_it` | light 模式必须为 `null`；heavy 模式必须为 string |

---

## Edge Schema

```typescript
type ForewriteEdge = {
  from: string;     // node id
  to: string;       // node id
  kind: EdgeKind;
};

type EdgeKind =
  | "subdream_of"   // from 是 to 的更小、更具体的子页
  | "sibling"       // 同一父页下的兄弟
  | "depends_on"    // from 需要 to 先实现
  | "supersedes"    // from 替代了之前 abandon 的 to
  | "revisit";      // from 是 to 的二次书写
```

### 边类型用法说明

#### `subdream_of`

verdict = `outer_dream` 时自动产生。

例：用户对 "建立 openclaw 通道" 写 forewrite，verdict 是 outer_dream（"我其实想要的是更大的——所有 AI 工具的统一调用层"）。重启 forewrite 后，"openclaw 通道" 节点带 `subdream_of` 边连到 "AI 统一调用层" 节点。

#### `sibling`

不会自动产生。当用户后续 forewrite 一个跟现有节点同父的目标时，可手动建立。Step 0 重访检查时如果发现疑似 sibling，提示用户。

#### `depends_on`

跨 record 的依赖。record A 的实现需要 record B 先实现。

#### `supersedes`

verdict = `abandon` 后，如果用户在另一份 forewrite 中提出替代方案，新节点带 `supersedes` 边连到旧节点。

#### `revisit`

Step 0 检测到相似旧节点，用户确认是 revisit 时产生。**不要 merge 节点**——每次 revisit 都是独立节点，通过 edge 表达书写历史。同一未来场景的多次书写构成一条链，可用来观察认知演变。

---

## 完整示例

### 示例 1：light，verdict = already_there

```json
{
  "id": "a3f7b2c1",
  "kind": "record",
  "written_at": "2026-04-26T15:30:00",
  "mode": "light",
  "trigger": "我对 openclaw 的观感很好，要有一个可以稳定使用它的通道",
  "now_page": {
    "why": "想能稳定调用 openclaw 处理一些任务",
    "current_state": null
  },
  "future_page": {
    "marker": "稳定使用一周后的某个周二上午",
    "scene": "周二上午十点，VPS 上的 openclaw 已经跑着，浏览器扩展挂在 toolbar 上，我点一下就能用，不用想它在哪台机器上。",
    "uses": [
      "把网页内容塞给 Claude 处理",
      "代理一些不能直连的 API"
    ],
    "without_it": null
  },
  "cross_reading": {
    "gap": "其实当前的 openclaw 已经跑在 VPS 上了，扩展也装好了，已经在做未来页里的事。",
    "verdict": "already_there"
  }
}
```

### 示例 2：heavy，verdict = minimum_action

```json
{
  "id": "c8e1d4a2",
  "kind": "record",
  "written_at": "2026-04-26T16:00:00",
  "mode": "heavy",
  "trigger": "我想把 vibekanban 搞到能用的状态",
  "now_page": {
    "why": "多 session agent 任务的状态分散，需要一个可视化的 single-pane",
    "current_state": "本地有部署好的 vibekanban 实例，能创建任务和拖拽，但跟 Claude Code 的 session 状态没接通；现在用 ~/.harness/graph.json 加 markdown 笔记凑合记。"
  },
  "future_page": {
    "marker": "vibekanban 上线一个月后的某个周二上午",
    "scene": "周二上午，我打开 vibekanban，看到三列任务，鼠标拖一个卡片到下一列，状态就更新了。Claude Code 那边的 session 也跟着更新。",
    "uses": [
      "在多 session agent 任务之间快速切换上下文",
      "可视化看到哪些任务在阻塞"
    ],
    "without_it": "现在用 ~/.harness/graph.json 加 markdown 笔记凑合着记，但每次都要手动同步状态，经常忘记。"
  },
  "cross_reading": {
    "gap": "拖拽和列状态切换还没接上 Claude Code 的 session 状态。差一个 hook。",
    "verdict": "minimum_action"
  }
}
```

### 示例 3：heavy，verdict = abandon

```json
{
  "id": "f2a9b6e0",
  "kind": "record",
  "written_at": "2026-04-26T16:30:00",
  "mode": "heavy",
  "trigger": "我想自己写一个比 langchain 更轻的 agent 编排框架",
  "now_page": {
    "why": "觉得现有框架太重，想要自己控制抽象层",
    "current_state": "目前直接用 Anthropic SDK 加几个 helper 函数处理调用编排，能覆盖 90% 的需求。"
  },
  "future_page": {
    "marker": "新建框架运行半年后的某个周二上午",
    "scene": "周二上午，我打开自己的框架文档，给一个 agent 配置 yaml，跑起来。",
    "uses": [
      "替代 langchain 在我项目里的位置"
    ],
    "without_it": "现在直接用 SDK 加几个 helper 函数已经能做到 90% 想要的事，剩下 10% 也不是瓶颈。"
  },
  "cross_reading": {
    "gap": "现在的方式已经够用，差的部分不值得做一整个框架。",
    "verdict": "abandon"
  }
}
```

### 示例 4：edges

```json
{
  "edges": [
    {
      "from": "c8e1d4a2",
      "to": "9d3e7f01",
      "kind": "subdream_of"
    },
    {
      "from": "b5a8c2d4",
      "to": "a3f7b2c1",
      "kind": "revisit"
    }
  ]
}
```

---

## 查询模式（用于渲染层 / 桥接层）

### 主路径

按 `written_at` 排序，过滤 verdict ∈ { `real_project`, `minimum_action` }，跟随 `subdream_of` 和 `depends_on` 边形成 DAG 主干。

### 已到达节点（成就感视图）

verdict = `already_there` 的节点。用户经常忘记自己已经走到了哪里，这个视图回答"我有什么"。当下页的 `current_state` 字段在这里特别有用——它记录了"那个时刻"的资产盘点。

### 放弃墓地（防重复跑偏）

verdict = `abandon` 的节点。下次有相似 trigger 时，Step 0 重访检查应优先匹配这里。

### Revisit 链

跟随 `revisit` 边的连通分量。同一未来场景的多次书写构成一条链。可用来观察：
- 当下页的资产盘点随时间增加（"诶上次我没意识到我已经有这个了"）
- 未来页的场景描述变得更具体或被替换（认知演变）
- gap 在缩小或扩大

---

## 演化预留

当前 schema 限定 `kind: "record"`，未来可扩展：

- `kind: "checkpoint"` — 主路径上的实际到达点（已实现的 minimum_action 或 real_project）
- `kind: "anchor"` — 跟 harness graph 的桥接锚点

但这些都属于桥接层和渲染层的关心，**forewrite skill 本身永远只产出 `kind: "record"` 节点**。