---
name: harness-graph
description: 从任意种子点出发，建立并累积 harness 全景图。当用户说「播种」「我在想」「这件事是关于」「帮我理清」「新种子」「seed」「我现在脑子里有个点」，或者在 session 开始时有模糊想法还未展开，立即触发此 skill。它将任意入口扩展为图结构，并以项目（工作）或事项（生活）为关联锚点，跨 session 累积生长。
---

# Harness Graph — 种子图谱

## 核心类比

像算法中的随机种子：

> 无论从哪个点切入，只要完成关联展开，最终收敛的图趋于一致。

种子不需要精准，切入点不需要完整。**任意位置开始，图会找到自己的形状。**

---

## 图的数据结构

图持久化存储于用户本地文件：

```
~/.harness/graph.json
```

若文件不存在，首次运行时自动创建。

### 节点类型

| 类型 | 说明 | 示例 |
|------|------|------|
| `project` | 工作中的项目 | "通知模块重构" |
| `matter` | 生活中的事项 | "装修选材" |
| `thought` | 尚未归类的想法碎片 | "感觉这块逻辑有问题" |
| `action` | 明确的下一步动作 | "跟设计确认边界" |
| `question` | 未解决的疑问 | "这个决定谁来拍板？" |
| `insight` | 某个理解的跃升 | "原来瓶颈不在代码，在流程" |

### 边的类型

| 关系 | 含义 |
|------|------|
| `belongs_to` | 这个节点归属于某 project/matter |
| `blocks` | 阻碍关系 |
| `triggers` | 触发关系（做了 A 才能做 B） |
| `related_to` | 松散关联 |
| `contradicts` | 矛盾/张力 |

---

## 播种流程

### Step 0 — 读取现有图谱

```bash
cat ~/.harness/graph.json 2>/dev/null || echo "{}"
```

加载已有的 projects、matters、nodes，作为上下文。

### Step 1 — 接收种子

用户给出任意形式的种子输入：
- 一句话、一个词、一个问题、一段困惑
- 不需要完整，不需要准确

**不要在此步骤追问任何问题。直接进入 Step 2。**

### Step 2 — 展开关联

基于种子 + 现有图谱，向用户提出**一次**展开性问题：

> "这个点，你觉得它更靠近哪里？
> 
> [列出现有的 projects/matters，最多 5 个]
> 
> 还是说，这是一个新的 project/matter？"

如果图谱为空（第一次播种），改为：

> "这个种子，它属于工作中的某个项目，还是生活中的某件事？
> 给它一个名字，哪怕是临时的。"

### Step 3 — 自动扩展节点

用户确认归属后，你来完成图的扩展：

1. 从种子中提取 **1-3 个子节点**（thoughts、actions、questions）
2. 识别与**现有节点**的潜在连接（如果图谱非空）
3. 向用户展示扩展结果，格式见下方

**展示格式：**

```
🌱 种子：{seed_text}
   └─ 归属：{project/matter}

📍 新节点：
   [thought]  {content}
   [action]   {content}  
   [question] {content}

🔗 发现连接：
   "{existing_node}" → related_to → "{new_node}"
   （或：暂无连接）

✏️ 有需要调整的吗？直接说，或者回复「确认」保存。
```

### Step 4 — 写入图谱

用户确认（或无异议超过一轮）后，将新节点写入 `~/.harness/graph.json`：

```bash
# 读取现有图谱，merge 新节点，写回
python3 -c "
import json, sys, os, datetime

graph_path = os.path.expanduser('~/.harness/graph.json')
os.makedirs(os.path.dirname(graph_path), exist_ok=True)

try:
    with open(graph_path) as f:
        graph = json.load(f)
except:
    graph = {'nodes': [], 'edges': [], 'projects': [], 'matters': []}

# [Claude 在此插入实际的 merge 逻辑]
new_data = sys.stdin.read()
updates = json.loads(new_data)

graph['nodes'].extend(updates.get('nodes', []))
graph['edges'].extend(updates.get('edges', []))
for p in updates.get('projects', []):
    if p not in graph['projects']:
        graph['projects'].append(p)
for m in updates.get('matters', []):
    if m not in graph['matters']:
        graph['matters'].append(m)

with open(graph_path, 'w') as f:
    json.dump(graph, f, ensure_ascii=False, indent=2)

print('✅ 图谱已更新')
"
```

---

## 全景图渲染

当用户说「展示图谱」「我现在在哪」「show graph」「全景」时，渲染当前图：

```
═══════════════════════════════════
  HARNESS GRAPH  /  {date}
═══════════════════════════════════

▼ PROJECTS
  ┌─ {project_1}
  │   ├─ [action]   {node}
  │   ├─ [question] {node}  ← ⚠️ 未解决
  │   └─ [insight]  {node}
  │
  └─ {project_2}
      └─ [thought]  {node}

▼ MATTERS  
  └─ {matter_1}
      ├─ [action]   {node}
      └─ [thought]  {node}

▼ 未归属碎片
  · {orphan_thought_1}
  · {orphan_thought_2}

───────────────────────────────────
  节点总数: {n}  |  最近更新: {ts}
```

---

## 全景图的涌现特性

当累积到一定数量的节点后，你应该主动识别并提示：

- **孤岛**：某个 project 长期没有 action 节点 → "这个项目看起来停滞了"
- **堵塞**：某个 question 节点阻碍了多个 action → "这个问题卡住了不少事"
- **张力**：两个节点存在 `contradicts` 关系但都未解决 → "这里有个还没处理的矛盾"
- **收敛信号**：同一 project 的 thought 节点在减少，action 节点在增加 → "这个项目在收敛"

不要主动分析所有节点，只在渲染全景图时，**挑选最显著的 1-2 个**提示。

---

## 种子的随机性原则

任何切入点都有效，包括：

- "我今天很烦" → 展开为 matter 或未归属 thought
- "这个接口设计感觉哪里不对" → 归属到某 project，提取 question
- "我突然想到一个更好的方案" → 归属到 project，提取 insight + action
- "不知道" → 合法种子，展开为孤立 thought，等待后续归属

**没有无效的种子。**

---

## 参考文件

- `references/graph-schema.md` — graph.json 完整 schema 定义
