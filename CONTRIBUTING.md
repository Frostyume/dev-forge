# Contributing to dev-forge

Thanks for being here! dev-forge is a small, focused Claude Code skill, and it gets better with your bug reports, new taxonomy ideas, and output polish. This guide keeps contributions smooth.

By participating you agree to our [Code of Conduct](CODE_OF_CONDUCT.md).

## Ways to contribute

- **Report a bug** — use the [bug report template](.github/ISSUE_TEMPLATE/bug_report.md). It mirrors dev-forge's own record schema (*description · root cause · impact*), so a good report is already half a fix.
- **Request a feature** — use the [feature request template](.github/ISSUE_TEMPLATE/feature_request.md).
- **Propose a new category** — the `category` taxonomy in [`reference/conventions.md`](reference/conventions.md) §4 is intentionally extensible. Open an issue first describing the class of problems it captures and why the existing buckets don't fit, then send a PR adding it to §4 (and a one-line CHANGELOG note).
- **Improve outputs** — sharper report skeletons, résumé phrasing, or checklist distillation rules in `templates/` and `reference/`.

## Working on the skill locally

dev-forge is plain Markdown — there's no build step.

1. Copy or symlink your working tree into the skills directory so Claude Code can load it:
   ```bash
   # macOS / Linux
   ln -s "$PWD" ~/.claude/skills/dev-forge
   # Windows (PowerShell, admin)
   New-Item -ItemType SymbolicLink -Path "$env:USERPROFILE\.claude\skills\dev-forge" -Target "$PWD"
   ```
2. Start a fresh Claude Code session and exercise the command you changed (`/dev-forge log`, `/dev-forge report`, …).
3. Your test records land in `~/.claude/dev-forge/` — that folder is data, not part of this repo, so it's safe to poke at.

## Output format rules (please follow)

dev-forge's *generated* files (problem records, reports, résumés) follow a strict house style so they render cleanly everywhere — see [`reference/conventions.md`](reference/conventions.md) §7–§8. The essentials:

- **No `---` horizontal rules** in body text. (`---` is fine only as a YAML frontmatter delimiter.)
- **Blank lines only around tables** — one before and after each table, none elsewhere.
- **UTF-8 without BOM.** On Windows PowerShell 5.1, `Out-File`/`Set-Content -Encoding utf8` writes a BOM that breaks frontmatter and JSON — use a no-BOM writer instead.
- **Quote YAML values** containing `:`, `#`, brackets, quotes, or leading digits/symbols.

These rules bind generated output, not the repo's own docs (README, this file, etc.).

## Pull requests

- Keep PRs focused; one logical change per PR.
- If you touch user-facing behavior, **update both `README.md` and `README.zh-CN.md`** so the two languages stay in sync.
- Add a short entry to [`CHANGELOG.md`](CHANGELOG.md) under an *Unreleased* heading (or the current dev version).
- Describe what you changed and how you verified it (which command you ran, what output you checked).

## 中文简版

dev-forge 是一个小而专注的 Claude Code skill，欢迎你的 bug 报告、新分类提议与产出物打磨。

- **报 bug**：用 [bug 模板](.github/ISSUE_TEMPLATE/bug_report.md)，它就是 dev-forge 自己的记录 schema（描述 / 根因 / 影响），写好一份报告等于修了一半。
- **提需求**：用 [feature 模板](.github/ISSUE_TEMPLATE/feature_request.md)。
- **加新分类**：`category` 体系（[`reference/conventions.md`](reference/conventions.md) §4）刻意可扩展；先开 issue 说明它覆盖哪类问题、为何现有桶装不下，再 PR 加进 §4 并补一行 CHANGELOG。
- **本地开发**：纯 Markdown 无需构建；把工作目录软链到 `~/.claude/skills/dev-forge`，开新会话跑你改的命令，测试记录会落在 `~/.claude/dev-forge/`。
- **产出格式**：生成文件遵守 §7–§8（正文无 `---`、表格前后留空行、UTF-8 无 BOM、YAML 特殊值加引号）；改用户可见行为时**中英 README 一并更新**，并补 CHANGELOG。

参与即表示你接受 [行为准则](CODE_OF_CONDUCT.md)。
