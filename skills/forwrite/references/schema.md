# Dwell Graph — Schema 完整定义

## 文件位置

```
~/.dwell/graph.json
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

无 `projects`、无 `matters`。dwell 只装 dwelling 节点。如未来需要把 dwelling 关联到 harness 的 project/matter，由桥接层处理。

---

## Node Schema

```typescript
type DwellingNode = {
  id: string;              // 8-char hex，主键
  kind: "dwelling";        // 当前唯一类型，预留扩展

  // 时间双戳
  visited_at: string;      // ISO 8601，本次 dwell 发生时刻
  dwelt_in: string;        // 文字描述，被访问的未来时刻
                           // 例："稳定运行一个月后的某个周二上午"

  mode: "light" | "heavy";

  // 来源
  trigger: string;         // 触发本次 dwell 的用户原话片段（≤ 200 字）

  // 仪式产物
  scene: string;           // Step 1：周二上午的具体画面
  uses: string[];          // Step 2：在那个未来里你拿它做什么
  without_it: string | null;  // Step 2b（heavy only）：没有它你怎么做

  // 反向校验
  distance_check: {
    gap: string;           // 跟当下差什么
    verdict: Verdict;
  };
};

type Verdict =
  | "already_there"     // 已在梦里
  | "minimum_action"    // 差一个最小动作
  | "real_project"      // 真要做的项目
  | "abandon"           // 放弃
  | "outer_dream";      // 是更大梦的子集
```

### 字段约束

| 字段 | 约束 |
|---|---|
| `id` | 8 字符十六进制，建议从 uuid4 截前 8 位 |
| `visited_at` | ISO 8601，到秒 |
| `dwelt_in` | 自由文本，但需含时间锚（"一个月后"、"半年后") |
| `trigger` | ≤ 200 字。**保留用户原话**，不要 paraphrase |
| `scene` | 自由文本，应含具体感官细节 |
| `uses` | array of string，每条一个用途 |
| `without_it` | light 模式必须为 `null`；heavy 模式必须为 string |

---

## Edge Schema

```typescript
type DwellEdge = {
  from: string;     // node id
  to: string;       // node id
  kind: EdgeKind;
};

type EdgeKind =
  | "subdream_of"   // from 是 to 的更小、更具体的子集
  | "sibling"       // 同一父梦下的兄弟
  | "depends_on"    // from 需要 to 先实现
  | "supersedes"    // from 替代了之前 abandon 的 to
  | "revisit";      // from 是 to 的二次访问
```

### 边类型用法说明

#### `subdream_of`

verdict = `outer_dream` 时自动产生。

例：用户对 "建立 openclaw 通道" 做 dwell，verdict 是 outer_dream（"我其实想要的是更大的——所有 AI 工具的统一调用层"）。重新启动 dwell 后，"openclaw 通道" 节点带 `subdream_of` 边连到 "AI 统一调用层" 节点。

#### `sibling`

不会自动产生。当用户后续 dwell 一个跟现有节点同父的目标时，可手动建立。dwell skill 在 Step 0 重访检查时如果发现疑似 sibling，提示用户。

#### `depends_on`

跨 dwelling 的依赖。dwell 节点 A 的实现需要 dwell 节点 B 先实现。

#### `supersedes`

verdict = `abandon` 后，如果用户在另一次 dwell 中提出替代方案，新节点带 `supersedes` 边连到旧节点。

#### `revisit`

Step 0 检测到相似旧节点，用户确认是 revisit 时产生。**不要 merge 节点**——每次 revisit 都是独立节点，通过 edge 表达访问历史。

---

## 完整示例

### 示例 1：light，verdict = already_there

```json
{
  "id": "a3f7b2c1",
  "kind": "dwelling",
  "visited_at": "2026-04-26T15:30:00",
  "dwelt_in": "稳定使用一周后的某个周二上午",
  "mode": "light",
  "trigger": "我对 openclaw 的观感很好，要有一个可以稳定使用它的通道",
  "scene": "周二上午十点，VPS 上的 openclaw 已经跑着，浏览器扩展挂在 toolbar 上，我点一下就能用，不用想它在哪台机器上。",
  "uses": [
    "把网页内容塞给 Claude 处理",
    "代理一些不能直连的 API"
  ],
  "without_it": null,
  "distance_check": {
    "gap": "其实当前的 openclaw 已经跑在 VPS 上了，扩展也装好了，已经在做那个画面里的事。",
    "verdict": "already_there"
  }
}
```

### 示例 2：heavy，verdict = minimum_action

```json
{
  "id": "c8e1d4a2",
  "kind": "dwelling",
  "visited_at": "2026-04-26T16:00:00",
  "dwelt_in": "vibekanban 上线一个月后的某个周二上午",
  "mode": "heavy",
  "trigger": "我想把 vibekanban 搞到能用的状态",
  "scene": "周二上午，我打开 vibekanban，看到三列任务，鼠标拖一个卡片到下一列，状态就更新了。Claude Code 那边的 session 也跟着更新。",
  "uses": [
    "在多 session agent 任务之间快速切换上下文",
    "可视化看到哪些任务在阻塞"
  ],
  "without_it": "现在用 ~/.harness/graph.json 加 markdown 笔记凑合着记，但每次都要手动同步状态，经常忘记。",
  "distance_check": {
    "gap": "拖拽和列状态切换还没接上 Claude Code 的 session 状态。差一个 hook。",
    "verdict": "minimum_action"
  }
}
```

### 示例 3：heavy，verdict = abandon

```json
{
  "id": "f2a9b6e0",
  "kind": "dwelling",
  "visited_at": "2026-04-26T16:30:00",
  "dwelt_in": "新建框架运行半年后的某个周二上午",
  "mode": "heavy",
  "trigger": "我想自己写一个比 langchain 更轻的 agent 编排框架",
  "scene": "周二上午，我打开自己的框架文档，给一个 agent 配置 yaml，跑起来。",
  "uses": [
    "替代 langchain 在我项目里的位置"
  ],
  "without_it": "现在直接用 Anthropic SDK 加几个 helper 函数，已经能做到 90% 想要的事，剩下 10% 也不是瓶颈。",
  "distance_check": {
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

按时间排序，过滤 verdict ∈ { `real_project`, `minimum_action` }，跟随 `subdream_of` 和 `depends_on` 边形成 DAG 主干。

### 已到达节点（成就感视图）

verdict = `already_there` 的节点。用户经常忘记自己已经走到了哪里，这个视图回答"我有什么"。

### 放弃墓地（防重复跑偏）

verdict = `abandon` 的节点。下次有相似 trigger 时，Step 0 重访检查应优先匹配这里。

### Revisit 链

跟随 `revisit` 边的连通分量。同一未来场景的多次访问构成一条链，可用来观察认知演变。

---

## 演化预留

当前 schema 限定 `kind: "dwelling"`，未来可扩展：

- `kind: "checkpoint"` — 主路径上的实际到达点（已实现的 minimum_action 或 real_project）
- `kind: "anchor"` — 跟 harness graph 的桥接锚点

但这些都属于桥接层和渲染层的关心，**dwell skill 本身永远只产出 `kind: "dwelling"` 节点**。