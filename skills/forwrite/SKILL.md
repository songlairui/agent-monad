---
name: forewrite
description: 当用户提出项目级或基础设施级的想法（"我想做 X"、"我打算搞 Y"、"我要建立 Z"、"我对 W 观感很好准备开始"），主动启动一次"双页同书"——用一支笔同时书写两页：当下页（为什么想做、现在什么状态）和未来页（完成后某个具体时刻的场景、用途）。然后交叉阅读两页，落到五种 verdict 之一：already_there（已到达，无需启动）、minimum_action（差一个小动作）、real_project（真要做的项目）、abandon（不吸引人放弃）、outer_dream（是更大梦的子集）。专门用来在兴致涌起、跃跃欲试的时刻插一脚，发现很多事根本不需要启动。区别于 intent-anchor（session 锚定）和 refine-goal（目标本身澄清）——forewrite 是唯一同时书写两个时间点的 skill。支持文件系统的 agent 写入 ~/.forewrite/graph.json；无文件系统的 agent 产出 forewrite-ingest JSON 块供后续 ingest。
---

# Forewrite — 编写未来

## 核心隐喻

一支笔，同时在两页上书写：

- **当下页**：你现在想做什么、为什么想做、跟这件事相关的当前状态是什么
- **未来页**：完成后某个具体时刻——比如某个周二上午十点——你打开了什么、看到了什么、它对你做了什么

写完之后，把两页摊开**交叉阅读**。

forewrite 的产物不是某一页，是**两页之间的距离**。这个距离决定要不要启动新的执行通道。

---

## 与邻近 skill 的位置

时间方向是 forewrite 的唯一独立轴。

| skill | 工作位置 |
|---|---|
| refine-goal | 当下：澄清目标本身 |
| harness-graph | 当下：从种子展开图 |
| intent-anchor | 当下：锚定 session |
| **forewrite** | **同时书写当下与未来 → 交叉阅读 → 决定要不要启动** |

---

## 第一性原理

**P1 — 两页同时存在**：当下页和未来页都是一等公民。只写一页都是失败的——只写当下退化为 to-do，只写未来退化为白日梦。

**P2 — 未来页要落到一帧**：未来页必须是一个**具体时刻**的场景，不是抽象状态。"我可以稳定使用 X" 是状态；"周二上午十点我打开 X，看到三列任务" 是一帧。

**P3 — 距离就是产物**：交叉阅读两页之间的差距，比单独读哪一页都重要。距离很小 → 不需要启动；距离很大但方向错了 → 也不需要启动。

**P4 — 不启动是合法结局**：五种 verdict 中三种导向"不启动新执行通道"。这是 skill 的设计目的，不是退化模式。

---

## 触发条件

### 主动触发

用户提出**项目级或基础设施级**的想法时，立即触发。信号：

- **名词**：通道、系统、框架、工具、流程、平台、stack、套件
- **动词**：建立、搭建、做一个、开始、搞一个、铺一套
- **兴致**：观感很好、跃跃欲试、觉得很妙、想试试、准备开始

**NOT 触发**：

- 已在执行中的单步动作（"修改这个样式"、"改一下这个函数"）
- 小工具调用（"帮我搜一下 X"）
- 信息查询（"X 是什么"）
- 用户已经在 intent-anchor 锚定的 session 中执行任务

### 被动触发（仅 filesystem-capable agent）

用户说「ingest 这个 forewrite」、「合并 forewrite」，或贴入 ` ```forewrite-ingest ` 标记的 JSON 块时，跳过书写仪式直接进入"节点写入"步骤。

---

## 自动选档

档位由 agent 自动判断，**不询问用户**。用户可在任意步骤说「升档」、「降档」切换。

### light — 默认档

满足全部：
- 单一工具或单一组件
- 跟现有系统无大改动
- 一天内可见效

light 模式：
- 当下页省略 `current_state`，只写 `trigger` + `why`
- 未来页省略 `without_it`
- 其余照常

### heavy — 基建档

满足任一即升档：
- 跨多组件、多设备、多系统
- 涉及"稳定使用"、"长期运行"
- 引入新抽象、新概念
- 用户语气表现出强烈兴致（最容易跑偏的信号）

heavy 模式两页都写完整。

---

## 仪式流程

### Step 0 — 重访检查（仅 filesystem-capable agent，graph 非空时）

读取 `~/.forewrite/graph.json`，按 trigger 关键词扫描相似 record。

如果找到：

> "你之前在 {written_at} 写过类似的未来：{future_page.scene 摘要}，verdict 是 {verdict}。
> 
> 这次是 revisit 吗？还是新一份 forewrite？"

revisit 模式下，本次新节点附带 `revisit` 边连到旧节点。

无文件系统场景跳过此步。

---

### Step 1 — 写当下页

向用户提问：

> "先写当下页。三个字段：
> 
> 1. 为什么想做这件事？（一句话）
> 2. 跟这件事相关的现状是什么？已经有哪些东西在跑、哪些在 manual、哪些缺失？
> 
> （light 模式只问第 1 项。）"

记录为 `now_page.why` 和 `now_page.current_state`。

**关键约束：当下页要在写未来页之前写。** 这一顺序不能颠倒。先写当下是为了让用户**先盘点资产**——很多时候盘点完就发现"诶，已经有了"，不用再走未来页。

如果用户在写当下页时已经自己说出"等一下其实已经有了"——直接跳到 Step 3，verdict 提议 `already_there`。

---

### Step 2 — 写未来页

向用户提问：

> "现在写未来页。
> 
> 假设这件事已经做完，稳定运行了一段时间。挑一个**最普通的时刻**——比如某个周二上午十点。那一刻你打开什么、看到什么、它对你做了什么？
> 
> 不要写状态，写场景。要具体到能闻到。"

等待回答。记录为 `future_page.scene`。

如果用户写的是状态而非画面（"我可以稳定使用 X"），引导一次：

> "这是状态。我要的是一帧画面——那个状态下，周二上午十点你具体在做什么？"

只引导一次。第二次仍是状态，记下来继续。**不要替用户补全画面。**

#### Step 2b — 用途追问

> "在那一帧画面里，你拿它**做什么**？为什么是它，不是别的？"

记录为 `future_page.uses`（list）。

#### Step 2c — heavy 档专属

> "没有它的时候，这件事你是怎么做的？"

记录为 `future_page.without_it`。

**这一问是 forewrite 最锋利的一刀。不要软化、不要诱导答案、不要在用户回答后立刻安抚或解释。** 它的作用就是戳出"其实现在的方式也行"——如果戳出来了，就让它戳出来。

---

### Step 3 — 交叉阅读

把当下页和未来页摊在一起。向用户提问：

> "把两页摊开比一下。
> 
> 当下页和未来页之间，差什么？差的部分，跟你最初想做的事，是同一件事吗？"

根据回答归类到下表（你提议，用户确认或修改）：

| verdict | 含义 | 出口 |
|---|---|---|
| `already_there` | 当下页已经写出未来页的内容 | 不启动新事，可能调小细节 |
| `minimum_action` | 两页之间差一个最小动作 | 直接做那个动作，跳过 intent-anchor |
| `real_project` | 两页之间是个真要做的项目 | 进入 intent-anchor 锚定 session |
| `abandon` | 未来页不吸引人 | 放弃，记入 graph，结束 |
| `outer_dream` | 这一页其实是另一份更大 forewrite 的子页 | 重启 forewrite 从外层开始，新节点附带 `subdream_of` 边 |

**呈现选项，让用户选。不要诱导某个 verdict。**

记录为 `cross_reading: { gap, verdict }`。

---

## 节点写入

### Filesystem-capable agent

写入 `~/.forewrite/graph.json`：

```bash
python3 <<'PY'
import json, os, datetime, uuid

graph_path = os.path.expanduser('~/.forewrite/graph.json')
os.makedirs(os.path.dirname(graph_path), exist_ok=True)

try:
    with open(graph_path) as f:
        graph = json.load(f)
except (FileNotFoundError, json.JSONDecodeError):
    graph = {'nodes': [], 'edges': []}

# agent 在此处依据本次 forewrite 的实际结果填充字段
new_node = {
    'id': str(uuid.uuid4())[:8],
    'kind': 'record',
    'written_at': datetime.datetime.now().isoformat(timespec='seconds'),
    'mode': 'light',  # 或 'heavy'
    'trigger': '...',
    'now_page': {
        'why': '...',
        'current_state': None  # heavy 模式填充
    },
    'future_page': {
        'marker': '...',  # 例："稳定运行一个月后的某个周二上午"
        'scene': '...',
        'uses': [],
        'without_it': None  # heavy 模式填充
    },
    'cross_reading': {
        'gap': '...',
        'verdict': '...'
    }
}
new_edges = []  # 例：[{'from': new_node['id'], 'to': '<old_id>', 'kind': 'revisit'}]

graph['nodes'].append(new_node)
graph['edges'].extend(new_edges)

with open(graph_path, 'w') as f:
    json.dump(graph, f, ensure_ascii=False, indent=2)

print(f"✅ Forewrite {new_node['id']} 已记入 ~/.forewrite/graph.json | verdict: {new_node['cross_reading']['verdict']}")
PY
```

写入后简短告知用户：

> "✅ 两页已落定。Verdict: {verdict}。"

然后**根据 verdict 触发对应出口**（见 Step 3 表格）。不要追加额外解释。

### No-filesystem agent

完成仪式后，输出标准 ingest 块：

````
```forewrite-ingest
{
  "kind": "record",
  "written_at": "<ISO timestamp>",
  "mode": "light",
  "trigger": "...",
  "now_page": {
    "why": "...",
    "current_state": null
  },
  "future_page": {
    "marker": "...",
    "scene": "...",
    "uses": ["..."],
    "without_it": null
  },
  "cross_reading": { "gap": "...", "verdict": "..." }
}
```
````

附一行说明：

> "复制这个块，粘贴到 filesystem-capable agent，forewrite skill 会在被动模式下识别并 ingest 进 ~/.forewrite/graph.json。"

---

## Schema 速查

```
Node {
  id, kind: "record",
  written_at, mode, trigger,
  now_page:    { why, current_state }
  future_page: { marker, scene, uses[], without_it }
  cross_reading: { gap, verdict }
}

Edge { from, to, kind }
  kind ∈ { subdream_of, sibling, depends_on, supersedes, revisit }
```

完整 schema 与示例见 `references/schema.md`。

---

## 与其它 skill 的衔接

- **之前**：目标本身没说清 → 先 `refine-goal` 再 forewrite
- **之后**（verdict = `real_project`）：进入 `intent-anchor` 锚定 session
- **之后**（verdict = `minimum_action`）：直接做最小动作，**跳过 intent-anchor 重型流程**
- **之后**（verdict = `already_there` / `abandon`）：结束。不启动任何新执行通道。
- **跟 harness-graph 的关系**：forewrite 不写 harness graph。两图分离。未来如需关联，由独立桥接层处理。

---

## 仪式期间的规则

- **当下页先写**：顺序不可颠倒。资产盘点要在场景想象之前。
- **不替用户描绘 scene**：场景必须由用户给出。agent 只能澄清"这是状态不是场景"。
- **不引导 verdict**：呈现选项，让用户选。
- **`without_it` 一问不要软化**（heavy 模式）：软化它就废了它。
- **不安抚**：`abandon` 不需要"放弃也是好的"这种话。verdict 就是 verdict。
- **3 分钟内完成**：light 通常 1 分钟，heavy 不超过 3 分钟。用户开始展开讨论时说"我记下来了，继续"。
- **不致歉、不用 social lubricant**：错了直接改方向。

---

## 主动触发判定示例

| 用户原话 | 触发 | 档位 |
|---|:---:|---|
| "我想做一个稳定使用 openclaw 的通道" | ✓ | heavy |
| "我打算搭一个 vibekanban 的 staging 环境" | ✓ | heavy |
| "我要给自己做一套早晨工作流" | ✓ | heavy |
| "我对这个 X 库观感很好，准备开始用" | ✓ | light |
| "我想试试用 Y 替代 Z" | ✓ | light |
| "帮我修改这个 CSS" | ✗ | — |
| "我想用 ripgrep 搜一下 ABC" | ✗ | — |
| "X 是什么意思" | ✗ | — |

---

## 参考文件

- `references/schema.md` — 完整 schema、字段定义、示例 JSON、查询模式
