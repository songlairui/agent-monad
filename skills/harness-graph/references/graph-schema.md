# graph.json — 完整 Schema

存储路径：`~/.harness/graph.json`

---

## 顶层结构

```json
{
  "version": "1.0",
  "created_at": "2025-06-12T10:00:00Z",
  "updated_at": "2025-06-12T14:23:00Z",
  "projects": [],
  "matters": [],
  "nodes": [],
  "edges": []
}
```

---

## Project / Matter

```json
{
  "id": "proj_001",
  "type": "project",       // "project" | "matter"
  "name": "通知模块重构",
  "status": "active",      // "active" | "paused" | "done"
  "created_at": "2025-06-01T00:00:00Z",
  "tags": ["Q2", "留存项目"]
}
```

---

## Node

```json
{
  "id": "node_0042",
  "type": "thought",       // thought | action | question | insight
  "content": "感觉这块逻辑有问题，但说不清在哪",
  "parent_id": "proj_001", // 归属的 project/matter id，null = 未归属
  "status": "open",        // "open" | "resolved" | "dropped"
  "created_at": "2025-06-12T14:23:00Z",
  "seed_text": "这块逻辑感觉哪里不对",  // 原始种子文本，保留溯源
  "session_id": "sess_20250612_001"
}
```

---

## Edge

```json
{
  "id": "edge_0010",
  "from": "node_0042",
  "to": "node_0038",
  "relation": "blocks",    // belongs_to | blocks | triggers | related_to | contradicts
  "note": "这个问题解决之前，重构方案无法确认",
  "created_at": "2025-06-12T14:25:00Z"
}
```

---

## ID 生成规则

- project/matter：`proj_001`、`matt_001`，三位数字，顺序递增
- node：`node_0001`，四位数字
- edge：`edge_0001`，四位数字
- session：`sess_YYYYMMDD_NNN`

---

## 最小有效图（首次播种后）

```json
{
  "version": "1.0",
  "created_at": "2025-06-12T14:20:00Z",
  "updated_at": "2025-06-12T14:23:00Z",
  "projects": [
    {
      "id": "proj_001",
      "type": "project",
      "name": "通知模块重构",
      "status": "active",
      "created_at": "2025-06-12T14:20:00Z",
      "tags": []
    }
  ],
  "matters": [],
  "nodes": [
    {
      "id": "node_0001",
      "type": "thought",
      "content": "感觉这块逻辑有问题，但说不清在哪",
      "parent_id": "proj_001",
      "status": "open",
      "created_at": "2025-06-12T14:23:00Z",
      "seed_text": "这块逻辑感觉哪里不对",
      "session_id": "sess_20250612_001"
    },
    {
      "id": "node_0002",
      "type": "question",
      "content": "边界条件是否覆盖了并发场景？",
      "parent_id": "proj_001",
      "status": "open",
      "created_at": "2025-06-12T14:23:00Z",
      "seed_text": "这块逻辑感觉哪里不对",
      "session_id": "sess_20250612_001"
    }
  ],
  "edges": [
    {
      "id": "edge_0001",
      "from": "node_0001",
      "to": "node_0002",
      "relation": "triggers",
      "note": "",
      "created_at": "2025-06-12T14:23:00Z"
    }
  ]
}
```
