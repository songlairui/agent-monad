---
name: sancai-tian
description: 三才阵 · 天位。管理 harness 全景图谱，负责播种、全景渲染、压缩。播种和全景操作默认自主完成，不需要逐步确认。
---

# 天 · 全景图谱

> 天者，万物所归之处。种子落地，图谱生长，全景可见。

数据文件：`~/.harness/graph.json`

---

## 子技能（AI 内部路由）

| 信号 | 执行 |
|------|------|
| 模糊想法、新种子、有个点 | → [播种](#播种) |
| 全景、show graph、我在哪 | → [全景](#全景) |
| 节点 > 50 / 人位触发 | → [压缩](#压缩) |

---

## 播种

**Step 0** — 加载图谱：
```bash
cat ~/.harness/graph.json 2>/dev/null || echo '{"version":"1.0","projects":[],"matters":[],"nodes":[],"edges":[],"archived":[]}'
```

**Step 1** — 接收种子，不追问。

**Step 2** — 归属判断：

若图谱中已有 projects/matters，AI **自行推断**最可能的归属。如果推断置信度高（种子内容明显属于某个 project），直接归属，不问。如果模糊，一次提问：

> "这个点，更靠近哪里？
> [列出最相关的 2-3 个]
> 还是新的 project/matter？"

若图谱为空：

> "这属于工作项目还是生活事项？给它一个名字。"

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
```

**Step 4** — 写入图谱。

对于常规播种（单个种子、归属明确），展示后**直接保存**，不等"确认"。用户如有异议会主动说。

对于批量播种或涉及新 project 创建，展示后等一轮反馈。

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

- **孤岛**：project 无 action → "这个项目停滞了"
- **堵塞**：question 阻碍多个 action → "这个问题卡住了不少事"
- **张力**：contradicts 边未解决 → "这里有个未处理的矛盾"
- **收敛**：thought↓ action↑ → "这个项目在收敛"

---

## 压缩

> 进化不是累加，是替换。

触发条件：节点总数 > 50，或人位发出压缩信号。

**压缩规则（按优先级）：**

1. 合并同义节点：同一 parent，内容相似 > 80% → 保留更新的
2. 归档已解决：status=resolved 且 > 14 天未引用 → 移入 archived
3. 合并孤立 thought：未归属 > 30 天无边连接 → 压缩为摘要节点
4. 压缩 insight 链：同 project > 5 个 insight → 提炼为 1 个核心

压缩前展示计划：

```
🗜 压缩计划
  合并: {n} 个同义节点
  归档: {n} 个已解决节点
  摘要: {n} 个孤立碎片

  压缩后：{before} → {after} 个活跃节点

  确认？[Y/n]
```

压缩涉及批量数据变更，需要用户确认。

---

## 节点类型

`thought` · `action` · `question` · `insight`

## 边类型

`belongs_to` · `blocks` · `triggers` · `related_to` · `contradicts`

---

## 参考文件

- `references/graph-schema.md` — 完整 JSON schema 与读写操作
