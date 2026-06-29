# 🔥 dev-forge

[English](README.md) · **简体中文**

> **给开发者的本地踩坑知识库——记一次根因，永不重复 debug 同一个坑。**

![Claude Code Skill](https://img.shields.io/badge/Claude%20Code-Skill-8A2BE2)
![License: MIT](https://img.shields.io/badge/License-MIT-green)
![Local first](https://img.shields.io/badge/data-100%25%20local-blue)

一套 [Claude Code](https://claude.com/claude-code) skill。在你踩坑、解决的当下，把一个 bug 的**问题描述 · 根因 · 影响 · 解决方案 · 经验教训**手工记成**一份本地 Markdown**。dev-forge 把这一份记录变成可检索的**根因记忆**、会自我生长的**避坑清单**，并**顺带**产出周报与简历亮点。

**一处记录，多处复用。让调试认知不再蒸发，数据 100% 留在本地——纯 Markdown，绝不外传。**

---

## 为什么用 dev-forge？

工程师最宝贵的资产，是你凌晨两点挖出来的那个**根因**——而它也是最先蒸发的东西。一个月后你撞上同一类 bug，又从零开始。

dev-forge 是**给 bug 的第二大脑**。每当你解决一个问题，就把*它为什么坏*、*你学到了什么*记成结构化、可 grep 的本地 Markdown。由这一份记录，它随后：

- 🧠 **建起可检索的根因记忆**——`recall` 会在你**重新花一天解一个早已解过的坑之前**就提醒你；
- 🛡️ **蒸馏出可复用的避坑清单**——你的经验教训变成下次同样的坑咬到你之前要过的预防检查；
- 📝 **并顺手起草你的周报与简历亮点**——这份记录本就包含它们需要的一切。

周报和简历是副产物。真正的意义在于：**你的调试认知不再流失。**

## 和别的工具有何本质不同？

brag-doc 和"git → 简历"类工具，把**成就**记到云端，为的是绩效评审时好看。dev-forge 把**根因**记到**你自己的磁盘**，为的是让你更快、并阻止你重复犯错。录入单元是 `bug → 根因 → 影响 → 解决 → 经验教训`，不是"我交付了什么"——这个区别就是整个产品：

- **根因优先，而非成就优先**——价值单元是*它为什么坏、你学到了什么*，在记忆最鲜活时手工记下。
- **100% 本地，纯 Markdown + YAML**——你的数据是你拥有的文件：可 grep、可脚本、Obsidian 友好、永远属于你（见 [interop](reference/interop.md)）。无 SaaS、无账号、无锁定。
- **Claude-native**——它是一个 skill，不是又一个要你照看的 app。

周报与简历是早已被解决的赛道——dev-forge 把它们一并给你，但它们不是它存在的理由。

## 功能

核心是 **记录 → 记住 → 不重蹈**（`log` / `recall` / `lessons`）这条主线，其余一切都由同一批记录按需锻造。

| 命令 | 模块 | 作用 |
|---|---|---|
| `/dev-forge log` | 录入 Capture | 记一个坑：描述 / 根因 / 影响，自动抓 git 上下文，自动做根因查重 |
| `/dev-forge resolve <id>` | 结案 Resolve | 补解决方案、教训、耗时、fix commit |
| `/dev-forge recall <词>` | 回溯 Recall | "我以前踩过吗？"——根因级历史检索；`log` 时也会自动跑 |
| `/dev-forge lessons` | 锦囊 Lessons | 把已解决的坑蒸馏成可复用**避坑清单**，让你不再重蹈 |
| `/dev-forge report [weekly\|monthly\|quarterly] [brief\|standard\|detailed]` | 战报 Report | 汇总成周/月/季报，三档详略——手动或定时 |
| `/dev-forge resume [zh\|en]` | 履历 Portfolio | 把已解决问题锻造成 STAR 简历条目（活文档、JD 裁剪、脱敏） |
| `/dev-forge stats` | 仪表盘 Dashboard | 按严重度 / 类别 / 项目的度量、解决时长、复发热点 |
| `/dev-forge open` | 追踪 Tracker | 未结案迷你 issue tracker，过期项高亮 |
| `/dev-forge help` | 帮助 Help | 打印命令菜单与每条的一句话说明 |
| `/dev-forge auto [on\|off\|status\|scope]` | 自动捕获 Auto | 开发中静默记录你解决的非平凡 bug——opt-in，默认关（[详情](reference/auto-capture.md)） |

## 安装

dev-forge 是一个 Claude Code skill。把文件夹放进你的 skills 目录：

```bash
# clone 或下载后：
cp -r dev-forge ~/.claude/skills/dev-forge          # macOS / Linux
# Windows（PowerShell）：
Copy-Item -Recurse dev-forge "$env:USERPROFILE\.claude\skills\dev-forge"
```

重启 Claude Code（或开新会话）。完成——输入 `/dev-forge help` 查看全部命令。数据首次使用时自动建在 `~/.claude/dev-forge/`。

> 可选：想要**定时周报**就装 [Claude Code CLI](https://claude.com/claude-code)（见下）。git 可选——存在则自动抓取 repo/分支/commit。

## 快速上手

```text
# 解决某事的当下：
/dev-forge log
  → 描述这个 bug；dev-forge 起草记录、查你是否踩过同款根因，然后保存。

# 回到之前搁置的问题：
/dev-forge resolve P-20260105-01

# "等等，我是不是解过这个？"
/dev-forge recall "softmax nan"

# 把累积的教训变成开工前要过的清单：
/dev-forge lessons

# 周五下午 / 更新简历——都由同一批记录锻造：
/dev-forge report weekly standard
/dev-forge resume zh
```

样例见 [`examples/`](examples/)：一条真实问题记录、它喂养的避坑清单、汇总成的周报、以及变成的简历亮点。

## 自动捕获（opt-in）

懒得每次记得 `log`？开启自动捕获，dev-forge 会在你**经 Claude Code 开发**的过程中**静默记录你解决的非平凡 bug**，让问题作为干活的副产物自动落盘——无需手动一步。

```text
/dev-forge auto on                 # 开启（把一段受管块写入全局 ~/.claude/CLAUDE.md）
/dev-forge auto status             # 查看范围 / 阈值 / 安装位置
/dev-forge auto scope my-project   # 只在指定项目生效
/dev-forge auto off                # 关闭并移除受管块
```

它**默认关闭**、**静默**（绝不打断你）、按根因去重，且只记严重度达标的问题，让琐碎不入库。触发指令按设计装在你的**全局** `~/.claude/CLAUDE.md`——绝不碰你的代码仓库——延续 dev-forge「不污染仓库」的原则。它是尽力而为：在你**经 Claude Code 开发**时生效，漏掉的随时手动 `log` 补上。完整契约见 [`reference/auto-capture.md`](reference/auto-capture.md)。

## 数据模型

每个问题一份 Markdown，存于 `~/.claude/dev-forge/problems/`，带 YAML frontmatter：

```yaml
id: P-20260105-01
project: aurora-trainer
title: "混合精度下 loss 变 NaN，训练在第 3 个 epoch 崩溃"
status: resolved        # open / investigating / resolved / wontfix
severity: high          # low / medium / high / critical
category: numerics
created: 2026-01-05
resolved: 2026-01-06
time_spent: 4h
repo: aurora-trainer
commit: 9f2a1c4
fix_commit: 3d7be01
```

正文是固定五节：问题描述 / 根因 / 影响 / 解决方案 / 经验教训。报表、清单、简历都由扫描这个目录**实时派生**——没有第二真相源会漂移。

## 你的数据是开放的

它们只是你拥有的文件。schema 是有文档的稳定契约，可被任何工具驱动——ripgrep 一行命令、对 frontmatter 跑 `yq`、Obsidian Dataview 表格。无导出、无 API、无锁定。grep / yq / Dataview / 脚本配方见 [`reference/interop.md`](reference/interop.md)。

## 定时报表（可选）

随时手动，或交给系统调度器无人值守运行：

```bash
# macOS / Linux crontab —— 每周一 09:00
0 9 * * 1 claude -p "/dev-forge report weekly standard"
```

```text
:: Windows 任务计划程序 —— 每周一 09:00
schtasks /create /tn "dev-forge weekly" /sc weekly /d MON /st 09:00 ^
  /tr "claude -p \"/dev-forge report weekly standard\""
```

## 隐私

一切都在你机器上的 `~/.claude/dev-forge/`。dev-forge 绝不写入你的代码仓库，也绝不把你的记录发往任何地方。`resume --anon` 可在你分享简历前抹去内部项目/仓库名。

## 常见问题（FAQ）

**数据存在哪？**
存于 `~/.claude/dev-forge/`（Windows 为 `%USERPROFILE%\.claude\dev-forge\`）：纯 Markdown 问题文件加一个小的 `config.json`。绝不写入你的代码仓库。

**有任何东西会离开我的机器吗？**
不会。你的记录是本地文件，只在你调用命令时被读取，绝不上传。分享简历前可用 `resume --anon` 抹去项目/仓库名。

**`/dev-forge` 不触发，该查什么？**
确认文件夹位于 `~/.claude/skills/dev-forge/` 且 `SKILL.md` 就在其中，然后重启 Claude Code 或开新会话。输入 `/dev-forge help` 确认已加载。

**如何备份或同步记录？**
它们就是文件——把任意备份工具指向 `~/.claude/dev-forge/`，或自己在一个**私有**仓库里对该目录 `git init`。dev-forge 绝不替你做这件事。

**Windows 下 YAML/JSON 会被加 UTF-8 BOM 而损坏吗？**
不会。dev-forge 在设计上写**无 BOM** 的 UTF-8，刻意避开 PowerShell 5.1 会加 BOM 的 `-Encoding utf8`。

**必须装 Git 吗？**
不必。git 可选；存在则自动抓取 repo/分支/commit，否则这些字段留空，不阻断任何流程。

**可以手动改记录吗？**
可以——它们是纯 Markdown。保持 frontmatter schema 合法（见 [`reference/conventions.md`](reference/conventions.md)），每个命令都照常工作。

## 参与贡献

欢迎提 issue 与 PR——新增分类、产出物打磨、bug 修复。见 [CONTRIBUTING.md](CONTRIBUTING.md) 与[行为准则](CODE_OF_CONDUCT.md)。

## 目录结构

```text
dev-forge/
├── SKILL.md                 # 入口 + log/resolve 流程
├── templates/               # 产出骨架（问题、报表、简历）
├── reference/               # 约定、报表、简历、增值模块、interop、auto-capture
├── examples/                # 样例 问题 → 清单 → 周报 → 简历
├── README.md · README.zh-CN.md · LICENSE · CHANGELOG.md
```

## 路线图

- 基于开放 schema 的更丰富 Obsidian Dataview 仪表盘
- `stats` 仪表盘的趋势图
- 团队模式（共享根因库，opt-in）
- 基于 hook 的确定性自动捕获兜底（每轮 / 会话结束保证落盘）

## 许可证

MIT © 2026 Frostyume —— 见 [LICENSE](LICENSE)。
