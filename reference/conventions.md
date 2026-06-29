# dev-forge 约定与规则（conventions）

执行 dev-forge 任何动作前先读本文件。本套件不写入任何代码仓库，所有数据集中在用户主目录下的私库。

## 1. 数据布局

根目录：`~/.claude/dev-forge/`（Windows 解析为 `%USERPROFILE%\.claude\dev-forge\`）

- `problems/<id>.md` —— 问题记录，唯一真相源
- `config.json` —— 项目登记表与设置
- `reports/`、`resume/` —— 报表与简历产出（按需创建）

报表与简历一律实时扫描 `problems/` 聚合，**不维护**任何会漂移的二级索引。所有文件都是开放纯文本，可被外部 grep / yq / Obsidian Dataview 直接消费——脚本化配方见 `interop.md`。

## 2. frontmatter schema

| 字段 | 必填 | 取值 / 说明 |
|---|---|---|
| id | 是 | `P-YYYYMMDD-NN`，见 §3 |
| project | 是 | 项目名；新项目自动入 config |
| title | 是 | 一句话标题 |
| status | 是 | open / investigating / resolved / wontfix |
| severity | 是 | low / medium / high / critical |
| category | 是 | 见 §4 taxonomy |
| created | 是 | YYYY-MM-DD（录入当天） |
| resolved | 否 | 结案当天；未结案留空 |
| time_spent | 否 | 如 `3h`、`2d`；结案时填 |
| tags | 否 | 字符串数组 |
| repo | 否 | git 仓库名（尽力抓） |
| branch | 否 | 录入时分支 |
| commit | 否 | 录入时 HEAD 短哈希 |
| fix_commit | 否 | 修复 commit 短哈希；结案时填 |
| files | 否 | 相关文件路径数组 |

正文固定五节：`## 问题描述` / `## 根因 (Root Cause)` / `## 影响 (Impact)` / `## 解决方案 (Solution)` / `## 经验教训 (Lessons)`。

## 3. ID 规则

`P-YYYYMMDD-NN`：

- YYYYMMDD = 录入当天日期（取当天日期：会话已知则直接用，否则 `Get-Date -Format 'yyyyMMdd'` / `date +%Y%m%d`）。
- NN = 当天序号，从 01 起，两位补零。算法：列出 `problems/` 中匹配 `P-YYYYMMDD-*.md` 的文件，解析出各自序号，取 **最大值 +1**（当天无记录则为 01）。删除留下的空缺号**不复用**，永远取 max+1，避免与历史 ID 撞号。

## 4. 取值 taxonomy

- **category**（可扩展，先归到最贴近的一类）：config / dependency / env / build / logic / design / numerics(数值稳定) / performance / data / integration / tooling / other
- **severity**：critical(阻断主线/数据损坏) / high(阻塞当前任务) / medium(有 workaround) / low(小瑕疵)
- **status**：open(已记未查) / investigating(排查中) / resolved(已解决) / wontfix(不修，附原因)

## 5. git 上下文抓取（尽力而为，永不阻断）

仅当 git 可用且当前工作目录在某仓库内时抓取；否则相关字段留空，**绝不因此阻断录入/结案**。健壮判定（PowerShell 示例，其他 shell 同理）：

1. 先确认 git 存在：`Get-Command git -ErrorAction SilentlyContinue`。**不存在则全部字段留空，跳过后续。**
2. 是否在仓库：`git rev-parse --is-inside-work-tree`，退出码非 0 即视为非仓库。
3. 仓库名：取 `git rev-parse --show-toplevel` 的末段目录名。
4. 分支：`git rev-parse --abbrev-ref HEAD`；短哈希：`git rev-parse --short HEAD`。
5. fix_commit（结案时）：同上取当前 HEAD 短哈希。

注意：①不要对原生 git 用 `2>&1`；②判定成功用 **`Get-Command` 存在性 + 退出码双保险**——git 未安装时退出码可能保留上一条命令的旧值，单看退出码会误判。若当前目录不在任何仓库内，这些字段留空属正常。

## 6. config.json

首次创建（`problems/` 不存在时一并建）：

```json
{
  "version": 1,
  "projects": [],
  "settings": { "resume_languages": ["zh", "en"] }
}
```

每次 log：若本次 project 不在 `projects` 数组中则追加写回。

## 7. 生成文件的格式规则

适用于 dev-forge 生成的**所有输出 .md**（问题记录、报表、简历），与常见 Markdown lint 习惯一致：

- 正文**禁止** `---` 横向分割线。frontmatter 顶部/底部的 `---` 是 YAML 定界符，不算分割线，照常使用。
- 表格（`|` 开头的行）前后各留一个空行；其余位置一律不留多余空行。
- 代码块（``` 包裹）内部空行保持原样。
- 文件名以 `<id>.md` 为准。

本规则只约束生成的输出文件，不约束本套件自身的 SKILL.md / 模板 / reference 源文件。

## 8. 写入安全（避免两类必现 bug）

- **YAML 转义**：frontmatter 中含特殊字符（`:`、`#`、`[`、`]`、`{`、`}`、引号，或以数字/符号开头）的字符串值必须用双引号包裹，例如 `title: "fix: 梯度爆炸"`。`tags` / `files` 数组元素同理逐个加引号。纯中文不需转义，但中文标题里常混入英文冒号，务必检查。
- **编码**：优先用 Write 工具落盘（UTF-8 无 BOM）。⚠️ Windows PowerShell 5.1 的 `Out-File`/`Set-Content -Encoding utf8` 会写出**带 BOM** 的 UTF-8，破坏 frontmatter 与 JSON；若必须用 PowerShell，请用 `[System.IO.File]::WriteAllText($path, $text, (New-Object System.Text.UTF8Encoding($false)))`。
