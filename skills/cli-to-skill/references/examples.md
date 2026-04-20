# 示范产物（样例，不是模板）

两个示例，用来说明 Tilt / Combos / Guardrails / 负空间 四段在实际 skill 里长什么样。
**不要**把它们当通用的 `gh` / `ffmpeg` skill 直接复制——它们是基于假设的 tilt 写的。

---

## 示例 1：精简版 `gh`（假想用户偏好：PR 工作流，不做仓库管理）

```markdown
---
name: gh
description: "gh CLI 的 PR 审阅与合并工作流 skill。当用户说『看一下 PR』『merge PR』『列出我的 review』或在 PR 相关任务上触发。不覆盖仓库管理、releases、actions 配置。"
---

# gh — PR 工作流窄腰

## 适用范围（Tilt）

- ✅ PR 列表、详情、评论、合并；review 请求与状态
- ❌ 不覆盖：repo create/fork/clone、releases、actions、secrets、gists
- 越界 → `gh --help` 或问用户

## 环境与前置

- 版本：gh 2.40+
- 认证：`gh auth status` 确认已登录
- 环境变量：可选 `GH_REPO=owner/name` 锁定默认仓库

## 高频动作（Recipes）

### 看某个 PR 的全貌

\`\`\`bash
gh pr view <num> --json number,title,state,reviews,statusCheckRollup,mergeable,body
\`\`\`

输出要点：`mergeable` + `statusCheckRollup` 决定能不能 merge
失败模式：404 → 通常是 repo 没锁对

### 列出我需要 review 的 PR

\`\`\`bash
gh pr list --search "review-requested:@me is:open" --json number,title,url
\`\`\`

### 回复 review comment

\`\`\`bash
gh api repos/<owner>/<repo>/pulls/<num>/comments/<cid>/replies \
  -f body='<msg>'
\`\`\`

## 复杂组装（Combos）

### 批量 squash-merge 所有 approved + 绿灯的 PR：CI 维护窗口用

\`\`\`bash
gh pr list --json number,reviewDecision,statusCheckRollup \
  | jq -r '.[] | select(.reviewDecision=="APPROVED" and (.statusCheckRollup|all(.conclusion=="SUCCESS"))) | .number' \
  | xargs -I{} gh pr merge {} --squash --delete-branch
\`\`\`

注意：非幂等；有 race（合并过程中新增的 commit 不会重跑 check）。在维护窗口用。

## 安全边界（Guardrails）

- 破坏性：`pr merge`、`pr close`、`api ... -X DELETE`
- merge 默认先跑 `gh pr checks <num>` 看绿
- 识别生产 repo：仓库名含 `prod` / `live` 时必须二次确认

## 输出与解析

- 结构化：全部命令都支持 `--json <fields>`，必须显式列 fields
- 分页：`gh pr list` 默认 30 条，加 `--limit 100` 或 `| cat` 关闭 TTY 分页

## 负空间（不要做）

- 不要用 `gh pr merge --admin` 绕过 branch protection
- 不要用 `gh pr review --approve` 自动批准（绕过人为审阅）
- 不要用无 `--json` 的 `gh pr view` 解析，列格式会随版本变

## 降级策略

1. `gh <sub> --help`
2. 不清楚 → 问用户
3. 学到新流程 → 回到 `cli-to-skill` 增补 Recipes

## 版本漂移

- 锚定：gh 2.40
- 检测：`gh --version`
- `--json` 字段名在大版本升级时可能变，优先跑 `gh pr view --help` 看可选字段
```

---

## 示例 2：精简版 `ffmpeg`（假想用户偏好：做短视频二次剪辑）

```markdown
---
name: ffmpeg
description: "ffmpeg 的短视频二次剪辑 skill（裁剪、拼接、加字幕、转码）。当用户说『剪个片段』『拼一下这俩视频』『压给小红书』时触发。不覆盖直播推流、滤镜图脚本编写、硬件加速调优。"
---

# ffmpeg — 短视频二次剪辑窄腰

## 适用范围（Tilt）

- ✅ 裁剪时长、拼接视频、加字幕/水印、输出适配平台的尺寸与码率
- ❌ 不覆盖：RTMP 推流、复杂 filter_complex 图、HW 编码器调优、DRM
- 越界 → `ffmpeg -h full | less` 或问用户

## 环境与前置

- 版本：ffmpeg 6.0+（7.0 更佳）
- 可选编码器：`ffmpeg -encoders | grep -E 'libx264|libx265|hevc_videotoolbox'`
- macOS：有 videotoolbox → 优先硬编

## 高频动作（Recipes）

### 无损裁剪时长（关键帧对齐）

\`\`\`bash
ffmpeg -ss 00:01:23 -to 00:02:45 -i in.mp4 -c copy out.mp4
\`\`\`

注意：`-ss` 在 `-i` 前面是 keyframe seek；不完全精准但不重编码。
失败模式：起止点不在 keyframe → 开头黑屏；改用下一条精准裁剪。

### 精准裁剪（重编码）

\`\`\`bash
ffmpeg -i in.mp4 -ss 00:01:23.5 -to 00:02:45.8 \
  -c:v libx264 -crf 20 -preset veryfast -c:a aac -b:a 128k out.mp4
\`\`\`

### 拼接相同编码的两段

\`\`\`bash
printf "file '%s'\n" a.mp4 b.mp4 > list.txt
ffmpeg -f concat -safe 0 -i list.txt -c copy out.mp4
\`\`\`

### 烧录外挂字幕

\`\`\`bash
ffmpeg -i in.mp4 -vf "subtitles=sub.srt:force_style='Fontsize=24'" -c:a copy out.mp4
\`\`\`

## 复杂组装（Combos）

### 小红书 9:16 适配（1080×1920，≤30MB，h264）

\`\`\`bash
ffmpeg -i in.mp4 \
  -vf "scale=1080:1920:force_original_aspect_ratio=decrease,pad=1080:1920:(ow-iw)/2:(oh-ih)/2:color=black" \
  -c:v libx264 -crf 23 -preset medium -maxrate 4M -bufsize 8M \
  -c:a aac -b:a 128k -movflags +faststart out.mp4
\`\`\`

注意：成品不保证 <30MB，超了调 `-crf 26` 或 `-maxrate 2.5M`。

## 安全边界（Guardrails）

- 破坏性：`-y` 覆盖输出；默认**不要**加 `-y`，让 ffmpeg 提示
- 原片保护：输出文件名必须和输入不同；禁止 `in.mp4 → in.mp4`

## 输出与解析

- 结构化：ffmpeg 没有 JSON；需要元数据用 `ffprobe -v error -print_format json -show_streams -show_format in.mp4`
- 进度：加 `-progress pipe:1 -nostats` 拿到 key=value 行

## 负空间（不要做）

- 不要用 `-vcodec copy` + 改分辨率（不重编码改不了分辨率）
- 不要用过时的 `-vframes` 替代 `-frames:v`
- 不要在 macOS 盲目用 `h264_videotoolbox`，某些源会色偏，先 crf/x264 验证

## 降级策略

1. `ffmpeg -h encoder=libx264` / `ffmpeg -h filter=subtitles`
2. 仍不清楚 → 问用户具体想要的输出规格（分辨率/码率/平台）
3. 新组合稳定后 → 补进 Combos

## 版本漂移

- 锚定：ffmpeg 6.0
- 7.0 后 `-filter_complex` 行为微调；升级后先跑一遍 Combos 的最简例
```

---

## 这两个示例想说明什么

- **Tilt 是第一段，且写明不覆盖什么**——这是 skill 相对于 `man` 的主要压缩
- **Recipes 可粘贴**——不留"请加合适的参数"这种让 Agent 抓瞎的话
- **Combos 有"何时用"**——不然就是把脚本堆到一起
- **负空间比覆盖范围更省 token**——告诉 Agent "不要做 X"，省掉它自己去试
- **版本漂移段让 skill 有过期预警**，不至于永远自信地用 3 年前的 flag
