# dev-forge 自动捕获（auto-capture）

`auto` 让 dev-forge 在你开发过程中**自动、静默地**把遇到并解决（或确认无法解决）的非平凡问题落盘为问题记录，免去每次手动 `/dev-forge log`。执行 `auto` 任何子动作前先读 `conventions.md`（schema / ID / git 抓取 / 写入安全）与本文件。

本能力为 **opt-in，默认关闭**。它的「自动」等级是 **L1（实时静默）**：可靠性高但非 100%——它依赖 Claude 在会话中按受管触发块的指令行事，且只在你**通过 Claude Code 开发**时生效（纯本地 IDE 手敲、不经过 Claude 解决的问题无法捕获）。要接近「必落盘」需叠加 hook（L2/L3，见 §9 局限），本文件只覆盖 L1。

## 为什么需要「受管触发块」

skill 不会自行运行——它只在被调用时加载。要让 Claude 在开发中**主动**记录，触发指令必须常驻「始终在上下文」的位置，也就是 `CLAUDE.md`。因此 L1 自动 = **config 开关（gate）+ CLAUDE.md 受管块（trigger）**：`auto on/off` 负责把这段块一键装好 / 卸下，并维护 `config.json`。

## 1. config 结构

在 `~/.claude/dev-forge/config.json` 的 `settings` 下维护 `auto_capture`（字段不存在即视为关闭）：

```json
{
  "version": 1,
  "projects": [],
  "settings": {
    "resume_languages": ["zh", "en"],
    "auto_capture": {
      "enabled": true,
      "mode": "silent",
      "scope": "all",
      "min_severity": "medium",
      "capture_states": ["resolved", "open"],
      "dedup": true,
      "installed_in": ["~/.claude/CLAUDE.md"]
    }
  }
}
```

- `enabled` —— 总开关。
- `mode` —— 目前仅 `silent`（不打断、不提问）。
- `scope` —— `"all"`（任意 project 都记）或字符串数组（仅列出的 project 记，按 project 名匹配）。
- `min_severity` —— 严重度闸：仅当推断 severity ≥ 此值才自动记，过滤琐碎。次序 low < medium < high < critical，默认 `medium`。
- `capture_states` —— 允许自动写入的状态，默认当场解决写 `resolved`、未解决写 `open`。
- `dedup` —— 写入前是否静默跑根因级 recall 去重，默认 true。
- `installed_in` —— 受管块已写入哪些文件（供 `off` / `status` 定位）。

## 2. 受管触发块

`auto on` 把以下块写入目标 `CLAUDE.md`，用分隔注释定界（幂等、可被 `auto off` 精确移除）：

```
<!-- dev-forge:auto-capture:start (managed by `/dev-forge auto` — 请勿手改) -->
**dev-forge 自动捕获已开启（L1 静默）。** 当你在开发中**遇到并解决（或确认无法解决）一个非平凡的问题 / bug**——真正调试 / 排查过、根因有信息量、值得日后回顾的，跳过拼写、格式化、琐碎重命名等——请**立即、静默地**把它落盘为一条 dev-forge 问题记录，遵循 dev-forge skill `reference/auto-capture.md` 的契约：
- 先静默跑一次根因级 recall 去重；命中同源根因则**补充进旧记录**而非新建。
- 当场已解决写 `status: resolved`（填根因 / 解决 / 教训），未解决写 `status: open`。
- 写入 `~/.claude/dev-forge/problems/<id>.md`，schema / ID / git / 写入安全一律遵 `reference/conventions.md`。
- 范围与严重度闸以 `~/.claude/dev-forge/config.json` 的 `auto_capture` 为准（scope 不含当前 project、或推断 severity 低于 min_severity，则**不记**）。
**不要为此打断对话、提问或等待确认**；记完仅在本轮回复末尾附一行极简回执，如 `🔨 dev-forge: 已记 P-<id> — <标题>`。若 `auto_capture` 不存在或 `enabled:false`，忽略本块。
<!-- dev-forge:auto-capture:end -->
```

## 3. auto on —— 开启

`/dev-forge auto on [--scope global|project] [--projects <all|名1,名2>]`

1. 读 `conventions.md`；确保 `~/.claude/dev-forge/` 与 `config.json` 存在（不存在则按 conventions §6 初始化）。
2. 选目标 CLAUDE.md：
   - **默认 `global` = `~/.claude/CLAUDE.md`**（个人级，**不写入任何代码仓库，契合 dev-forge「不污染仓库」原则**）。
   - 仅当显式 `--scope project` 才写当前仓库根的 `CLAUDE.md`；此时**先提示用户该文件通常随仓库提交、可能被团队看到**，确认后再写。
3. 幂等写入受管块：若目标文件已含 `dev-forge:auto-capture:start/end`，替换其间内容；否则追加到文件末尾（保留原有内容）。用 Write 工具或无 BOM UTF-8（conventions §8）。
4. 写 `config.json.settings.auto_capture`：`enabled:true`、`mode:"silent"`、`scope`（来自 `--projects`，默认 `"all"`）、`min_severity:"medium"`、`capture_states:["resolved","open"]`、`dedup:true`，并把目标文件路径并入 `installed_in`。**合并语义——读出原 config、仅增改 `auto_capture` 这一节、整体写回，保留 `projects` / `resume_languages` 等其余字段不变，切勿覆盖整个 config。**
5. 回执：开启状态、范围、阈值、受管块写入位置，并提示「下个会话起稳定生效」（当前会话若已加载该 CLAUDE.md，可能即时生效）。

## 4. auto off —— 关闭

`/dev-forge auto off`

1. 置 `config.json.settings.auto_capture.enabled:false`。
2. 从 `installed_in` 记录的每个文件中删除 `start`…`end` 受管块（连同定界注释；保留文件其余内容）。清空 `installed_in`。
3. 回执：已关闭，并列出清理过的文件。

## 5. auto status —— 查看

`/dev-forge auto status`

打印（不写文件）：`enabled`、`scope`、`min_severity`、`capture_states`、`installed_in`，以及「本会话是否可能生效」的判断。`auto_capture` 不存在时明说「未开启」。

## 6. auto scope —— 调整范围

`/dev-forge auto scope <all | 项目名[,项目名…]>`

仅更新 `config.json.settings.auto_capture.scope`，不动受管块。`all` = 任意 project；列表 = 仅这些 project 自动记。回执新范围。

## 7. 静默捕获契约（被触发时如何记）

当受管块在某次开发会话中被触发，按下列规则落盘，**全程不提问、不打断**：

1. **判定是否够格**：只记**非平凡**问题——真正调试 / 排查过、根因有信息量、日后值得回顾。跳过：拼写 / 格式化、无脑重命名、一次过的小调整、纯需求实现而无「踩坑」。推断 severity < `min_severity` 则不记。
2. **范围闸**：当前 project（按 conventions §5 抓仓库名，抓不到则取工作目录名 / 用户语境）不在 `scope` 内则不记。
3. **静默去重**（`dedup:true` 时）：用标题 + 根因关键词跑一次 recall（`value-modules.md`），**结果不抛给用户**。命中同源根因的旧坑 → 补充进旧记录（追加复现 / 触发场景或更新状态），不新建；命中 0 条 → 新建。
4. **写记录**：`resolved`（含根因 / 解决 / 教训）或 `open`（根因 / 解决留「待…」），ID / frontmatter / git / 写入安全全遵 `conventions.md`，落 `problems/<id>.md`。新 project 入 `config.json`。
5. **极简回执**：仅在本轮回复末尾附一行，如 `🔨 dev-forge: 已记 P-… — <标题>`（或「已并入 P-…」）。不展开、不打断主任务。

## 8. 与手动 log 的关系 / 安全

- 手动 `/dev-forge log` 始终可用，且**不受** `min_severity` / `scope` 限制（用户显式要记就一定记）。auto 只是替你免去「想起来要记」。
- auto **只写 dev-forge 私库**，绝不写你的代码仓库（受管块默认装在全局 `~/.claude/CLAUDE.md`）。
- 嫌噪声：调高 `min_severity` 到 `high`，或把 `scope` 收窄到具体项目，或 `auto off`。
- 隐私：所有记录 100% 本地；auto 不外发任何数据。

## 9. 局限（L1）与升级路径

- **非 100% 保证**：依赖 Claude 在会话中遵循受管块；长会话 / 高度专注时可能漏记——可手动补 `log`。
- **仅覆盖经过 Claude Code 的开发**；纯本地 IDE 操作无法捕获。
- **确定性兜底属 L2/L3**（不在本文件实现）：L2 = settings.json 的 `Stop`/`SessionEnd` 钩子，在每轮 / 会话结束确定性地提醒补记；L3 = `SessionEnd` 钩子起一个无头 `claude -p` 复盘整场会话、批量落盘。两者把「触发」变确定，但仍由 LLM 写内容，且 L3 每场会话多一次 Claude 调用（有成本）。
