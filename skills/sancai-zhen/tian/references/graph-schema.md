# graph.json — Schema & 操作

## Schema

```json
{
  "version": "1.0",
  "created_at": "",
  "updated_at": "",
  "projects": [
    { "id": "proj_001", "type": "project", "name": "", "status": "active", "created_at": "", "tags": [] }
  ],
  "matters": [
    { "id": "matt_001", "type": "matter", "name": "", "status": "active", "created_at": "", "tags": [] }
  ],
  "nodes": [
    {
      "id": "node_0001",
      "type": "thought",
      "content": "",
      "parent_id": "proj_001",
      "status": "open",
      "created_at": "",
      "seed_text": "",
      "session_id": ""
    }
  ],
  "edges": [
    { "id": "edge_0001", "from": "node_0001", "to": "node_0002", "relation": "triggers", "note": "", "created_at": "" }
  ],
  "archived": []
}
```

节点类型：`thought` · `action` · `question` · `insight`  
边类型：`belongs_to` · `blocks` · `triggers` · `related_to` · `contradicts`  
状态：`open` · `resolved` · `dropped` · `blocked`

ID 规则：proj_001 / matt_001 / node_0001 / edge_0001（顺序递增）

---

## 读操作

```python
import json, os

def load_graph():
    path = os.path.expanduser("~/.harness/graph.json")
    if not os.path.exists(path):
        return {"version":"1.0","projects":[],"matters":[],"nodes":[],"edges":[],"archived":[]}
    with open(path) as f:
        return json.load(f)
```

## 写操作

```python
import json, os, datetime

def save_graph(graph):
    path = os.path.expanduser("~/.harness/graph.json")
    os.makedirs(os.path.dirname(path), exist_ok=True)
    graph["updated_at"] = datetime.datetime.utcnow().isoformat() + "Z"
    with open(path, "w") as f:
        json.dump(graph, f, ensure_ascii=False, indent=2)

def next_id(graph, prefix):
    # prefix: "node", "edge", "proj", "matt"
    existing = [n["id"] for n in graph.get("nodes", []) + graph.get("edges", []) + graph.get("projects", []) + graph.get("matters", [])]
    nums = [int(i.split("_")[-1]) for i in existing if i.startswith(prefix)]
    n = max(nums) + 1 if nums else 1
    width = 4 if prefix in ("node", "edge") else 3
    return f"{prefix}_{str(n).zfill(width)}"
```

## 归档操作

```python
def archive_node(graph, node_id, reason=""):
    node = next((n for n in graph["nodes"] if n["id"] == node_id), None)
    if node:
        node["archived_at"] = datetime.datetime.utcnow().isoformat() + "Z"
        node["archive_reason"] = reason
        graph["archived"].append(node)
        graph["nodes"] = [n for n in graph["nodes"] if n["id"] != node_id]
```
