# dev-forge interop —— 你的数据是开放的

dev-forge 的记录不是锁在某个 app 里的私有格式，而是**你磁盘上的纯文本文件**。每个问题是一份带 YAML frontmatter 的 Markdown，放在 `~/.claude/dev-forge/problems/`。这意味着即使你从不再打开 dev-forge，这些数据依然能被任何工具读取、检索、统计、可视化——**无需导出、无 API、无锁定**。

本文件给出把这套 schema 当作开放契约来驱动的具体配方。schema 字段定义见 `conventions.md` §2，是稳定契约。

## 1. 文件长什么样

```text
~/.claude/dev-forge/
├── problems/
│   ├── P-20260105-01.md
│   └── P-20260106-01.md
├── lessons.md           # /dev-forge lessons 生成的避坑清单
└── config.json
```

每个 `P-*.md` 的 frontmatter 是机器可读的：`id / project / title / status / severity / category / created / resolved / time_spent / tags / repo / branch / commit / fix_commit / files`。正文是固定五节（问题描述 / 根因 / 影响 / 解决方案 / 经验教训）。

## 2. ripgrep / grep 配方

按根因检索（跨全库找「同源根因」，正是 recall 在做的事）：

```bash
rg -i "softmax|fp16|溢出" ~/.claude/dev-forge/problems/ -l
```

列出所有未结案的高危问题（按 frontmatter 字段过滤）：

```bash
rg -l "^severity: (high|critical)" ~/.claude/dev-forge/problems/ \
  | xargs rg -L "^status: (open|investigating)" -l
```

PowerShell 等价（Windows，无 rg 时）：

```powershell
Select-String -Path "$env:USERPROFILE\.claude\dev-forge\problems\*.md" `
  -Pattern '^severity: (high|critical)' | Select-Object -ExpandProperty Path -Unique
```

## 3. 用 yq / jq 做统计

frontmatter 是 YAML，用 [mikefarah `yq`](https://github.com/mikefarah/yq) 的 `--front-matter=extract` 即可只取 frontmatter 部分查询。

按 category 计数：

```bash
for f in ~/.claude/dev-forge/problems/*.md; do
  yq --front-matter=extract '.category' "$f"
done | sort | uniq -c | sort -rn
```

把全部记录的 frontmatter 导出成一个 JSON 数组（喂给任意脚本 / BI / jq）：

```bash
for f in ~/.claude/dev-forge/problems/*.md; do
  yq --front-matter=extract -o=json '.' "$f"
done | jq -s '.'
```

> 提示：`/dev-forge stats` 已内置这些聚合；本节是给想接自己流水线的人留的逃生舱。

## 4. Obsidian Dataview

把 `~/.claude/dev-forge/` 作为（或软链入）一个 Obsidian vault，即可用 Dataview 直接查询 frontmatter——零额外配置就有一个实时仪表盘。

未结案问题表：

````markdown
```dataview
TABLE severity, category, created, file.link AS 记录
FROM "problems"
WHERE status = "open" OR status = "investigating"
SORT severity DESC, created ASC
```
````

某根因类别的复发热点：

````markdown
```dataview
TABLE rows.file.link AS 记录, length(rows) AS 次数
FROM "problems"
GROUP BY category
SORT length(rows) DESC
```
````

## 5. 契约稳定性承诺

- 字段只增不删、不改语义；新增字段一律可选，老脚本不会因此挂掉。
- `id` 一旦分配永不复用、永不变更（见 conventions §3），可安全作为外部系统的稳定主键。
- 一切派生物（报表、简历、避坑清单）都从 `problems/` 实时重算，绝无第二真相源会与你的脚本读到的不一致。

你的踩坑数据属于你。dev-forge 只是恰好把它组织得很好。
