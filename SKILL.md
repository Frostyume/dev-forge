---
name: dev-forge
description: 开发复盘日志套件。记录代码项目中遇到的问题(描述/根因/影响/解决方案)，并由记录逐步产出周报/月报/简历项目等。触发场景：用户说"记一个坑/bug/问题"、"给某问题结案/补解决方案"、"写周报"、"包装简历项目"，或输入 /dev-forge 加 help|log|resolve|report|resume|stats|recall|open|lessons|auto；亦可开启 auto 自动捕获，在开发中静默落盘问题。
---

# dev-forge — 开发复盘日志套件

把代码项目中踩的坑（描述/根因/影响/解决/教训）记成结构化本地复盘，沉淀成可检索的根因记忆与避坑清单，并顺带锻造出周报与简历。一处记录，多处复用，永不重复踩同一个坑。

## 数据存放（中央私库，不污染代码仓库）

所有数据集中存于 `~/.claude/dev-forge/`（Windows 解析为 `%USERPROFILE%\.claude\dev-forge\`）：

- `problems/` —— 每个问题一个 `.md` 文件，是唯一真相源
- `config.json` —— 项目登记表与设置
- `reports/`、`resume/` —— 报表与简历产出（按需创建）

读写规则、frontmatter schema、ID 规则、git 抓取与格式规则详见 `reference/conventions.md`。**执行任何动作前先读它。**

## 动作分发

`/dev-forge <动作> [参数]`：

| 动作 | 模块 | 说明 |
|---|---|---|
| `log` | 录入 Capture | 记一个坑：描述/根因/影响 |
| `resolve <id>` | 结案 Resolve | 补解决方案/耗时/fix commit |
| `report [weekly\|monthly\|quarterly] [brief\|standard\|detailed]` | 战报 Report | 见 `reference/reporting-and-scheduling.md` |
| `resume [zh\|en]` | 履历资产 Portfolio | 见 `reference/resume-packaging.md` |
| `stats` / `recall <词>` / `open` / `lessons` | 增值模块 | 见 `reference/value-modules.md` |
| `auto [on\|off\|status\|scope …]` | 自动捕获 Auto-capture | 开发中静默自动落盘问题（opt-in，默认关）；见 `reference/auto-capture.md` |
| `help` | 帮助 Help | 打印命令菜单（见下「help」节） |

无动作时执行 `help`（见下）；动作不明时先打印 `help` 再请用户选择。

## log —— 录入一个问题

目标：低摩擦地记下一个坑。根因/解决方案在录入时未知是正常的，可留空（status=open）。

1. **读 `reference/conventions.md`**，拿到 schema、ID 规则、taxonomy、git 抓取与格式规则。
2. **确保数据目录存在**：若 `problems/` 不存在则创建，并初始化 `config.json`（conventions §6）。
3. **收集信息**。必填：`title` 与 `问题描述`。其余尽量从用户这句话和当前会话上下文（最近报错/改动）推断；缺失且重要的（severity、category、project）简短追问，但**不要为 open 问题强逼根因/解决方案**。
   - 若用户只给一句话（如"记一下刚才那个梯度爆炸的坑"），直接从上下文起草，再让用户确认。
   - **根因级查重（防重闸，勿跳过）**：拿到标题 + 初步根因/关键词后，先跑一次 recall（见 `reference/value-modules.md`），**以「根因」为首要匹配维度**。若命中**同源根因**的旧坑，显式提示「⚠️ 疑似重复踩坑：你 `<D>` 天前在 `<project>` 已踩过同款根因 `<id>`」，并让用户选择：(a) 补充进旧记录而不新建、(b) 确为新问题再新建。命中 0 条则正常继续。这是 dev-forge 的核心价值之一——在你重新花一天去解同一个根因之前拦住你。
4. **确定 project**：当前若在 git 仓库内，默认取仓库名；否则询问或沿用上次。新 project 追加进 `config.json`。
5. **抓 git 上下文**（尽力而为，conventions §5；失败就留空，**绝不因此阻断录入**）。
6. **生成 ID** `P-YYYYMMDD-NN`（conventions §3）。
7. **按 `templates/problem.md` 写文件** 到 `problems/<id>.md`，填好 frontmatter 与各节，遵守格式规则（conventions §7、§8）。
8. **回执**：告诉用户文件路径、id、status；若根因或解决方案留空，提示日后用 `/dev-forge resolve <id>` 结案。

## resolve —— 给问题结案

目标：补全解决方案与教训，翻转状态。

1. **读 `reference/conventions.md`**。
2. **定位文件**：在 `problems/` 中按 `<id>` 找；找不到就列出最近的 open 问题让用户选。
3. **收集**：`解决方案`（必填）、`经验教训`、`time_spent`、`fix_commit`（在仓库内可自动抓当前 HEAD 短哈希）。
4. **更新 frontmatter**：`status: resolved`、`resolved: <今天>`、填 `time_spent`、`fix_commit`。
5. **填正文** 的"解决方案"与"经验教训"两节，遵守格式规则。
6. **回执**：确认已结案，附路径。

若用户要标记为不修（wontfix）或仍在排查（investigating），相应设置 status，并允许"解决方案"节留作说明而非真正修复。

## auto —— 自动捕获（静默落盘，默认关闭）

让 dev-forge 在你开发过程中**自动、静默地**把遇到并解决的非平凡问题落盘，免去每次手动 `log`。opt-in，**默认关闭**；完整契约（config 结构、受管触发块、静默捕获规则）见 `reference/auto-capture.md`，执行前先读它。

- `auto on` —— 开启。默认把一段**受管触发块**写入**全局** `~/.claude/CLAUDE.md`（个人级，**不碰任何代码仓库**，契合「不污染仓库」原则），并在 `config.json` 写入 `auto_capture` 设置。仅显式 `--scope project` 才改项目 CLAUDE.md（会提示该文件可能随仓库提交）。
- `auto off` —— 移除受管块并置 `enabled:false`。
- `auto status` —— 打印开关 / 范围 / 阈值 / 受管块位置。
- `auto scope <all|项目名…>` —— 设定在哪些 project 生效。

本质：skill 不能自跑，触发指令须常驻上下文（CLAUDE.md），`auto` 把它一键装好/卸下。可靠性为 L1（高但非 100%），仅覆盖经过 Claude Code 的开发。

## help —— 命令菜单

`/dev-forge help`；无参数调用 `/dev-forge` 时默认执行本动作。

打印一份简明命令菜单到对话：逐条列出 `log / resolve / recall / lessons / report / resume / stats / open / auto` 的用法与一句话说明（即上方分发表内容），并点出核心主线 **`log → recall → lessons`（记录 → 记住 → 不重蹈）**。纯展示，**不读写任何文件**，无需先读 conventions。结尾给一句引导，如「直接说『记一下刚才那个坑』或 `/dev-forge log` 即可开始」。
