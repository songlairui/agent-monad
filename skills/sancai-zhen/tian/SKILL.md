---
name: sancai-tian
description: 三才阵 · 天位。管理 harness 全景图谱，负责播种、全景渲染、图谱压缩三个子技能。由阵眼路由触发，或用户说「播种」「全景」「压缩图谱」时直接触发。
---

# 天 · 全景图谱

> 天者，万物所归之处。种子落地，图谱生长，全景可见。

数据文件：`~/.harness/graph.json`

---

## 子技能路由

| 信号 | 执行 |
|------|------|
| 播种、新想法、有个点、seed | → [播种](#播种) |
| 全景、show graph、我在哪、图谱 | → [全景](#全景) |
| 压缩、太多了、清理图谱 | → [压缩](#压缩) |

---

## 播种

**Step 0** — 加载现有图谱：
```bash
cat ~/.harness/graph.json 2>/dev/null || echo '{"version":"1.0","projects":[],"matters":[],"nodes":[],"edges":[]}'
```

**Step 1** — 接收种子，不追问，直接进入 Step 2。

**Step 2** — 一次归属提问：

> "这个点，更靠近哪里？
> [列出现有 projects/matters，最多 5 个]
> 还是这是新的 project / matter？"

若图谱为空：

> "这属于工作中的项目，还是生活中的事项？给它一个临时名字。"

**Step 3** — 自动扩展，生成展示：

```
🌱 种子：{seed}
   └─ 归属：{project/matter}

📍 新节点：
   [thought]  {content}
   [action]   {content}
   [question] {content}

🔗 发现连接：
   "{existing}" → related_to → "{new}"

✏️ 有调整？或回复「确认」保存。
```

**Step 4** — 用户确认后写入图谱（见 `references/graph-ops.md`）。

---

## 全景

加载图谱，渲染：

```
═══════════════════════════════════
  天 · 全景  /  {date}
═══════════════════════════════════

▼ PROJECTS
  ┌─ {project}  [{active|paused|done}]
  │   ├─ [action]   {content}
  │   ├─ [question] {content}  ← ⚠️ 阻碍中
  │   └─ [insight]  {content}

▼ MATTERS
  └─ {matter}
      └─ [action] {content}

▼ 未归属
  · {orphan}

─────────────────────────────────
  节点: {n}  |  更新: {ts}
```

渲染后，识别并提示**最显著的 1 个**涌现信号：

- 孤岛：project 无 action → "这个项目停滞了"
- 堵塞：question 阻碍多个 action → "这个问题卡住了不少事"
- 张力：contradicts 边未解决 → "这里有个未处理的矛盾"
- 收敛：thought↓ action↑ → "这个项目在收敛"

---

## 压缩

> 进化不是累加，是替换。

触发条件：节点总数 > 50，或人位发出压缩信号。

**压缩规则（按优先级）：**

1. **合并同义节点**：同一 parent 下，内容相似度 > 80% → 保留更新的，归档旧的
2. **归档已解决节点**：status = resolved 且超过 14 天未引用 → 移入 `archived` 数组
3. **合并孤立 thought**：未归属超过 30 天且无边连接 → 压缩为一条摘要节点
4. **压缩 insight 链**：同一 project 超过 5 个 insight → 提炼为 1 个核心 insight，其余归档

压缩前展示计划，用户确认后执行：

```
🗜 压缩计划
  合并: {n} 个同义节点
  归档: {n} 个已解决节点
  摘要: {n} 个孤立碎片
  
  压缩后：{before} → {after} 个活跃节点
  
  确认执行？[Y/n]
```

---

## 节点类型

| 类型 | 含义 |
|------|------|
| `thought` | 模糊想法，尚未成形 |
| `action` | 明确的下一步 |
| `question` | 未解决的疑问 |
| `insight` | 理解的跃升 |

## 边类型

`belongs_to` · `blocks` · `triggers` · `related_to` · `contradicts`

---

## 参考文件

- `references/graph-schema.md` — 完整 JSON schema
- `references/graph-ops.md` — 读写操作代码片段
