# 🔥 dev-forge

**English** · [简体中文](README.zh-CN.md)

> **A local-first knowledge base for the bugs you fight — so you never debug the same thing twice.**

![Claude Code Skill](https://img.shields.io/badge/Claude%20Code-Skill-8A2BE2)
![License: MIT](https://img.shields.io/badge/License-MIT-green)
![Local first](https://img.shields.io/badge/data-100%25%20local-blue)

A [Claude Code](https://claude.com/claude-code) skill suite. Capture a bug **once** — *description · root cause · impact · solution · lessons* — in a plain local Markdown file. dev-forge turns that single record into a searchable root-cause memory, a self-building "don't hit this again" checklist, **and** — as byproducts — weekly reports and résumé bullets.

**Log once. Your debugging knowledge compounds instead of evaporating. 100% local — plain Markdown, nothing leaves your machine.**

---

## Why dev-forge?

Your hardest-won asset as an engineer is the **root cause** you dug out at 2 AM — and it's the first thing to evaporate. A month later you hit the same class of bug and start from zero.

dev-forge is a **second brain for bugs**. The moment you solve something, you capture *why it broke* and *what you learned* as structured, greppable, local Markdown. From that one record it then:

- 🧠 **builds a searchable root-cause memory** — `recall` warns you *before* you sink a day re-solving a bug you already cracked;
- 🛡️ **distills a reusable pre-flight checklist** — your lessons become preventive checks you run before the same mistake bites again;
- 📝 **and, for free, drafts your weekly report and résumé bullets** — the record already holds everything those need.

The reports and the résumé are byproducts. The point is that **your debugging knowledge stops leaking.**

## How is this different?

Brag-docs and "git → résumé" tools log *accomplishments* to a cloud, to look good at review time. dev-forge logs *root causes* to **your own disk**, to make you faster and stop you repeating mistakes. The unit of record is `bug → root cause → impact → solution → lessons`, not "things I shipped" — and that difference is the whole product:

- **Root-cause-first, not accomplishment-first** — the unit of value is *why it broke and what you learned*, captured by hand the moment it's fresh.
- **100% local, plain Markdown + YAML** — your data is files you own: greppable, scriptable, Obsidian-friendly, yours forever (see [interop](reference/interop.md)). No SaaS, no account, no lock-in.
- **Claude-native** — it's a skill, not one more app to babysit.

Weekly reports and résumé bullets are an already-solved space — dev-forge hands them to you, but they're not why it exists.

## Features

The heart is **capture → remember → never-repeat** (`log` / `recall` / `lessons`). Everything else is forged from those same records on demand.

| Command | Module | What it does |
|---|---|---|
| `/dev-forge log` | Capture | Record a problem: description / root cause / impact, auto-grabs git context, auto-checks for a duplicate root cause |
| `/dev-forge resolve <id>` | Resolve | Close it out with the solution, lessons, time spent, fix commit |
| `/dev-forge recall <kw>` | Recall | "Have I hit this before?" — root-cause-aware search across history; also auto-runs on `log` |
| `/dev-forge lessons` | Lessons | Distill solved bugs into a reusable **pre-flight checklist** so you stop repeating them |
| `/dev-forge report [weekly\|monthly\|quarterly] [brief\|standard\|detailed]` | Report | Roll records into a report at 3 detail levels — manual or scheduled |
| `/dev-forge resume [zh\|en]` | Portfolio | Forge solved problems into STAR résumé bullets (living doc, JD-tailoring, anonymize) |
| `/dev-forge stats` | Dashboard | Metrics by severity / category / project, resolve-time, recurrence hot spots |
| `/dev-forge open` | Tracker | Mini issue tracker of everything still unresolved, stale items flagged |
| `/dev-forge help` | Help | Print the command menu and a one-line description of each |
| `/dev-forge auto [on\|off\|status\|scope]` | Auto-capture | Silently log the non-trivial bugs you solve as you work — opt-in, off by default ([details](reference/auto-capture.md)) |

## Install

dev-forge is a Claude Code skill. Drop the folder into your skills directory:

```bash
# clone or download, then:
cp -r dev-forge ~/.claude/skills/dev-forge          # macOS / Linux
# Windows (PowerShell):
Copy-Item -Recurse dev-forge "$env:USERPROFILE\.claude\skills\dev-forge"
```

Restart Claude Code (or start a new session). That's it — type `/dev-forge help` to see every command. Data is auto-created under `~/.claude/dev-forge/` on first use.

> Optional: install the [Claude Code CLI](https://claude.com/claude-code) if you want **scheduled** weekly reports (see below). Git is optional — if present, dev-forge auto-captures repo/branch/commit.

## Quick start

```text
# The moment you solve something:
/dev-forge log
  → describe the bug; dev-forge drafts the record, checks whether you've
    hit this root cause before, and saves it.

# Came back to a problem you parked earlier:
/dev-forge resolve P-20260105-01

# "Wait, didn't I solve this once?"
/dev-forge recall "softmax nan"

# Turn accumulated lessons into a checklist you run before coding:
/dev-forge lessons

# Friday afternoon / updating your CV — forged from the same records:
/dev-forge report weekly standard
/dev-forge resume en
```

See [`examples/`](examples/) for a real problem record, the lessons checklist it feeds, the weekly report it rolls up into, and the résumé bullet it becomes.

## Auto-capture (opt-in)

Tired of remembering to `log`? Turn on auto-capture and dev-forge will **silently record the non-trivial bugs you solve while developing**, so problems land on disk as a side-effect of the work — no manual step.

```text
/dev-forge auto on                 # enable (writes a managed block to your global ~/.claude/CLAUDE.md)
/dev-forge auto status             # see scope / threshold / where it's installed
/dev-forge auto scope my-project   # restrict to specific projects
/dev-forge auto off                # disable and remove the managed block
```

It's **off by default**, **silent** (never interrupts you), de-duplicates by root cause, and only logs problems above a severity threshold so trivia stays out. By design the trigger is installed in your **global** `~/.claude/CLAUDE.md` — never your code repo — staying true to dev-forge's "never pollute your repo" rule. It's best-effort: it fires while you develop *through Claude Code*; anything it misses you can still `log` by hand. Full contract: [`reference/auto-capture.md`](reference/auto-capture.md).

## Data model

One Markdown file per problem, under `~/.claude/dev-forge/problems/`, with YAML frontmatter:

```yaml
id: P-20260105-01
project: aurora-trainer
title: "Mixed-precision loss goes NaN, training crashes at epoch 3"
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

Plain Markdown body with five sections: Description / Root cause / Impact / Solution / Lessons. Reports, checklists, and résumés are **derived live** by scanning this folder — there is no second source of truth to drift.

## Your data is open

These are just files you own. The schema is a stable, documented contract you can drive from anything — ripgrep one-liners, `yq` over frontmatter, an Obsidian Dataview table. No export, no API, no lock-in. See [`reference/interop.md`](reference/interop.md) for grep / yq / Dataview / scripting recipes.

## Scheduled reports (optional)

Manual any time, or let your OS scheduler run it headless:

```bash
# macOS / Linux crontab — every Monday 09:00
0 9 * * 1 claude -p "/dev-forge report weekly standard"
```

```text
:: Windows Task Scheduler — every Monday 09:00
schtasks /create /tn "dev-forge weekly" /sc weekly /d MON /st 09:00 ^
  /tr "claude -p \"/dev-forge report weekly standard\""
```

## Privacy

Everything lives in `~/.claude/dev-forge/` on your machine. dev-forge never writes into your code repos and never sends your records anywhere. The `resume --anon` flag can scrub internal project/repo names before you share a CV.

## FAQ

**Where is my data stored?**
Under `~/.claude/dev-forge/` (`%USERPROFILE%\.claude\dev-forge\` on Windows): plain Markdown problem files plus a small `config.json`. Nothing is written into your code repositories.

**Does anything leave my machine?**
No. Your records are local files, read only when you invoke a command, and never uploaded. Use `resume --anon` to scrub project/repo names before sharing a CV.

**`/dev-forge` isn't triggering — what should I check?**
Make sure the folder sits at `~/.claude/skills/dev-forge/` with `SKILL.md` directly inside, then restart Claude Code or start a new session. Type `/dev-forge help` to confirm it's loaded.

**How do I back up or sync my records?**
They're just files — point any backup tool at `~/.claude/dev-forge/`, or `git init` that folder in a **private** repo yourself. dev-forge never does this for you.

**Windows: will my YAML/JSON get a UTF-8 BOM and break?**
No. dev-forge writes UTF-8 **without** BOM by design, deliberately avoiding PowerShell 5.1's BOM-adding `-Encoding utf8`.

**Do I need Git?**
No. Git is optional; when present, repo/branch/commit are auto-captured, otherwise those fields stay empty and nothing is blocked.

**Can I hand-edit the records?**
Yes — they're plain Markdown. Keep the frontmatter schema valid (see [`reference/conventions.md`](reference/conventions.md)) and every command keeps working.

## Contributing

Issues and PRs welcome — new taxonomy categories, output polish, bug fixes. See [CONTRIBUTING.md](CONTRIBUTING.md) and our [Code of Conduct](CODE_OF_CONDUCT.md).

## Project layout

```text
dev-forge/
├── SKILL.md                 # entry point + log/resolve flow
├── templates/               # output skeletons (problem, report, resume)
├── reference/               # conventions, reporting, resume, value modules, interop, auto-capture
├── examples/                # sample problem → lessons → report → résumé
├── README.md · README.zh-CN.md · LICENSE · CHANGELOG.md
```

## Roadmap

- Richer Obsidian Dataview dashboards on top of the open schema
- Trend charts in the `stats` dashboard
- Team mode (shared root-cause base, opt-in)
- Deterministic auto-capture backstop via hooks (guaranteed capture at turn / session end)

## License

MIT © 2026 Frostyume — see [LICENSE](LICENSE).
