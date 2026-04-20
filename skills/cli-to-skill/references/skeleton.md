# SKILL.md 骨架（可直接填空）

拷贝到 `skills/<cli-name>/SKILL.md` 后替换所有 `<...>` 占位符。

---

```markdown
---
name: <cli-name>
description: "<一句触发语 + 核心能力摘要>。当用户说『<触发词 1>』『<触发词 2>』或需要 <典型场景> 时使用。"
---

# <cli-name> — <一句定位>

## 适用范围（Tilt）

- ✅ 覆盖：<意图 1>、<意图 2>、<意图 3>
- ❌ 不覆盖：<能力 A>、<能力 B>
- 越界时 → `<cli> --help` 或问用户

## 环境与前置

- 版本：<x.y+>
- 认证：<login 方式 / 配置路径>
- 环境变量：<VAR / 或"无">
- 环境识别：<命令 / 环境变量>

## 高频动作（Recipes）

### <意图 1>

\`\`\`bash
<cli> <args>
\`\`\`

输出要点：<...>
失败模式：<...>

### <意图 2>

\`\`\`bash
<cli> <args>
\`\`\`

## 复杂组装（Combos）

### <combo 名>：<何时用>

\`\`\`bash
<pipeline>
\`\`\`

注意：<并发 / 幂等 / 副作用>

## 安全边界（Guardrails）

- 破坏性动作：<list>
- 默认 dry-run：<flag>
- 生产识别：<signal>；命中必须二次确认

## 输出与解析

- 结构化：<flag>
- 分页：<flag>
- 无结构化时的锚点：<awk/jq tip>

## 负空间（不要做）

- 不要用 <X>（已弃用 / 语义不稳定）
- 不要假设 <Y>

## 降级策略

1. `<cli> <topic> --help`
2. 仍不清楚 → 问用户
3. 学到新用法 → 更新本 skill

## 版本漂移

- 锚定版本：<version>
- 检测：`<cli> --version`
- 漂移处理：用 `cli-to-skill` 重跑

## 参考

- `references/cheatsheet.md`（可选）
- `references/debugging.md`（可选）
```
