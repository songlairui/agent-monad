---
name: cli-to-skill
description: "元技能：为一个已有的 CLI 工具生产一份可被 AI Agent 直接使用的 skill，避免 Agent 在每次使用前反复 `--help` / 试错。当用户说『给 X 做个 skill』『封一下 xxx-cli』『把 gh / kubectl / docker / ffmpeg … 做成 skill』『convert CLI to skill』『cli2skill』，或用户抱怨『这个 CLI agent 总猜不对参数』时触发。输入是一个 CLI 名称 + 用户的使用倾向；输出是一个结构化 SKILL.md（可选 references/）。"
---

# cli-to-skill — CLI → Skill 压模器

> 目标不是复述 `man page`，而是**把一个大而全的 CLI 压成一个窄腰适配器**：留下 Agent 真实会用的那一小部分，再附上坑与组合。

## 核心定位

- **skill 不是 CLI 的镜像**：CLI 有几百个子命令没关系，skill 应该只长在"这个用户/这个项目实际用到的那几条路径"上。
- **skill 是带偏见的**：倾向（tilt）是一等公民。不声明倾向的 skill 会退化成复读 `--help`。
- **skill 的价值在 negative space**：告诉 Agent "不要用什么" 和 "哪些是坑"，往往比列举所有可用命令更有用。
- **skill 要能自我降级**：找不到想要的命令时，要告诉 Agent 该去哪里查（而不是让它凭空发明参数）。

## 触发条件

- 用户直说：『给 X 做个 skill』『封一下 xxx』『把 cli 变 skill』『cli2skill』
- 用户抱怨：『Agent 总猜错 xxx 的参数』『每次都要重新 `--help`』
- 用户想要：把一组常用 CLI 操作沉淀下来，避免每个 session 都重学一遍

**不触发**：用户只是问『xxx 命令怎么用』（那是一次性答疑，不是生产 skill）。

## 输入契约（先问，不猜）

开始前必须向用户确认四件事。缺一则停下来问，**不要替用户决定倾向**。

```
1. CLI 名称 / 二进制 / 版本约束
   例：gh 2.40+ / kubectl / ffmpeg / pnpm / my-company-cli
   不要默认用最新版，用户的环境可能被锁版本。

2. 使用倾向（tilt）—— 这是 skill 的灵魂
   询问模板：
   "告诉我你主要用它做什么？三到五件事就够了。
    哪些子命令你从来不用，或禁止 Agent 用？
    有偏好的工作流吗（例如：先 dry-run 再 apply / 只读模式优先）？"

3. 已知的复杂组装（combo）
   "有没有你习惯性串起来的几条命令？
    例如：gh pr list --json ... | jq ... | xargs gh pr merge
    有的话列给我，skill 会把它作为一等示例。"

4. 边界与禁区
   "有没有哪些命令是破坏性/不可逆的，必须二次确认？
    是否有生产环境标识（--prod / NAMESPACE=prod）需要额外守卫？"
```

**四个问题可以在一轮对话里一起问。用户答模糊没关系，记录模糊答案，继续。**

## 生成前探测（Probe）

只有用户授权、且本机确实装了该 CLI 时才探测。否则只基于用户描述 + 公共知识生成。

探测清单（按需执行，不是都要跑）：

```bash
# 身份与版本
<cli> --version
<cli> --help                   # 顶层
<cli> help                     # 有些 CLI 走这条
which <cli>                    # 路径，可能提示是 wrapper

# 子命令结构（只抓一层，不递归）
<cli> <top-sub> --help

# 输出格式能力（为"可管道化"做准备）
<cli> ... --json / --format / -o json / --output yaml

# 配置与认证
<cli> config ... / <cli> auth status / ls ~/.<cli>/
env | grep -i <CLI_PREFIX>

# 负空间信号
grep -i "deprecat\|removed\|legacy" 在 --help 输出里
```

**不要**：

- 不要对未知 CLI 跑 `<cli> <guess-subcommand>` 做黑盒穷举——有副作用风险
- 不要在没问用户的前提下对 `prod`/`production`/`live` 类环境跑任何命令
- 不要把探测到的所有子命令都搬进 skill——违背"带偏见"原则

## Skill 骨架（输出模板）

生成的 skill 文件路径：

- **本项目内复用** → `skills/<cli-name>/SKILL.md`
- **全局复用** → `~/.claude/skills/<cli-name>/SKILL.md`

由用户选。不确定时默认放到项目内。

骨架：

```markdown
---
name: <cli-name>
description: "<一句触发语 + 核心能力摘要>。当用户说『<触发词 1>』『<触发词 2>』或需要 <典型场景> 时使用。"
---

# <cli-name> — <一句定位>

## 适用范围（Tilt）

- ✅ 这个 skill 覆盖：<用户声明的 3–5 件事>
- ❌ 这个 skill 不覆盖：<显式列出不在范围内的能力>
- 不覆盖范围的需求 → Agent 应回退到 `<cli> --help` 或问用户

## 环境与前置

- 版本：<version constraint>
- 认证：<需要怎么 login / 配置文件在哪>
- 必需环境变量：<VAR_NAME / 或"无"><br>
- 识别"我在哪个环境"的方式：<如 `<cli> context current` 或读环境变量>

## 高频动作（Recipes）

每个动作写成 "意图 → 命令" 的一对多映射。命令一定要**可复制即用**，不要半句话。

### <意图 1：用人话描述>

\`\`\`bash
<cli> <args>            # 常规
<cli> <args> --json     # 需要结构化输出时
\`\`\`

输出要点：<Agent 从中提取什么字段>
失败模式：<常见 exit code / 错误串 → 原因>

### <意图 2 …>

...

## 复杂组装（Combos）

命名 + 一段可粘贴的脚本。每条 combo 都要有明确的"何时用"。

### <combo 名称>：<何时用>

\`\`\`bash
<cli> ... --json | jq '...' | xargs -n1 <cli> ...
\`\`\`

注意：<并发/幂等/副作用 提醒>

## 安全边界（Guardrails）

- 破坏性动作清单：<delete / force / apply / merge / push --force 等>
- 这些动作默认**先 dry-run**：<对应 flag，例如 `--dry-run=client` / `-n`>
- 生产环境识别：<特征串>；命中时必须二次向用户确认

## 输出与解析

- 结构化输出开关：<`--json` / `-o json` / `--format=json`>
- 无结构化能力时的解析提示：<用 awk/jq 的锚点>
- 分页：<是否默认分页；如何关闭 `| cat` / `--no-pager`>

## 负空间（不要做）

- 不要使用 <deprecated-subcmd>，已被 <replacement> 替代
- 不要用 <隐藏 flag> ——虽然能跑，语义在下个版本会变
- 不要把 <cli> 的输出直接当 JSON 解析，除非带了 <json flag>

## 降级策略

当 skill 没覆盖到需求时，Agent 的动作顺序：

1. `<cli> <nearest-topic> --help` 读一次
2. 还不清楚 → 问用户，不要乱试破坏性命令
3. 学到新用法 → 建议用户更新这份 skill（可指向 `cli-to-skill` 做增量）

## 版本漂移

- 本 skill 锚定版本：<version>
- 检测漂移的方式：`<cli> --version` 对比
- 漂移后做什么：用 `cli-to-skill` 重跑生成流程，或只增量修改 Recipes 段

## 参考

- `references/cheatsheet.md` —— 完整命令矩阵（可选，只在用户明确要求时生成）
- `references/debugging.md` —— 常见错误与解法（可选）
```

## 生成流程（六步）

1. **收集输入**：按[输入契约](#输入契约先问不猜)四个问题拿答案。
2. **探测（可选）**：在授权内按 [Probe](#生成前探测probe) 清单收集事实，不做黑盒穷举。
3. **起草 Tilt**：把用户的"要做的事"和"不做的事"原话化进 `适用范围` 段。**这是 skill 的压缩函数**，其他段落都围绕它裁剪。
4. **填 Recipes**：每个意图一条，命令必须可粘贴。超过 7 条就应该考虑是不是把 skill 拆成多个，或者把低频的挪到 `references/`。
5. **补 Combos + Guardrails + 负空间**：这三段是 skill 相对于 `--help` 的真正增量，**缺一段就是不合格**。即使只写"暂无"也要留 anchor。
6. **自检 + 写入**：过 [自检清单](#自检清单)，确认无误后写到约定路径，告知用户位置。

## 自检清单

生成完成前，逐条核对：

- [ ] 有 Tilt 段，且显式列出**不覆盖**的能力
- [ ] 每条 Recipe 都给了可粘贴的命令，而不是『使用 xxx 子命令』
- [ ] 至少有 1 条 Combo，或显式声明"本 CLI 没有典型 combo"
- [ ] Guardrails 段列出破坏性动作 + dry-run 开关
- [ ] 负空间段列出至少 1 条『不要做 X』
- [ ] 描述了降级策略：skill 没覆盖时 Agent 该怎么办
- [ ] description 字段在 300 字以内，包含触发词
- [ ] 没有把用户没讲的偏好塞进去（不要替用户编 tilt）

## 反模式（不要做）

- ❌ **把 `--help` 全文复制到 skill**：skill 会变大且迅速过期
- ❌ **写『支持所有 xxx 命令』**：这等于没写 tilt
- ❌ **把可选功能都标为 Recipe**：recipes 是**高频**动作，不是 all 动作
- ❌ **省略版本号**：CLI 翻一个大版本，半数 flag 会变
- ❌ **越权探测**：未经允许跑破坏性/生产命令去"了解 CLI"
- ❌ **替用户起倾向**：tilt 来自用户的真实使用习惯，不是 Agent 的想象

## 与其他 skill 的关系

- 配合 `agent-init`：新项目 onboard 时，可把项目里常用 CLI 批量过一遍，沉淀成一组 skill
- 配合 `intent-anchor`：如果用户的 tilt 不清楚，让 intent-anchor 先把意图锚定出来，再回到本流程
- 配合 `simplify`：Recipe 条数膨胀时，跑一次 simplify 裁掉低价值条目

## 参考

- `references/skeleton.md` —— 可直接填空的 SKILL.md 骨架副本
- `references/examples.md` —— 两个示范产物（精简版 `gh`、精简版 `ffmpeg`），作为样例而非模板
