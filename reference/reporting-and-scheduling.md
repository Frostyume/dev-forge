# report —— 战报生成与自动调度

`/dev-forge report [weekly|monthly|quarterly] [brief|standard|detailed]`
默认 `weekly standard`。执行前先读 `conventions.md`。

## 1. 时间窗与标签

| 粒度 | 标签 | 范围（含两端） |
|---|---|---|
| weekly | `<YYYY>-W<ww>`（ISO-8601，周一为周首） | 本周一 ~ 本周日 |
| monthly | `<YYYY>-<MM>` | 当月 1 日 ~ 月末 |
| quarterly | `<YYYY>-Q<n>` | 季首 ~ 季末 |

ISO 周号取法：优先 `Get-Date -UFormat %V`（Windows PowerShell 5.1 可用，已实测正确）；跨平台稳妥用 Thursday 规则手算（取"该周周四"所在的 ISO 年与周序）。⚠️ 不要用 `[System.Globalization.ISOWeek]`——它在 .NET Framework / PowerShell 5.1 下不存在，会抛异常。无论用哪种，报告头部都写出**明确的日期范围**，使标签歧义不影响阅读。允许用参数指定历史窗口（如 `report weekly 2026-W25`）。

## 2. 聚合口径

扫描 `problems/`，解析每个文件的 frontmatter，按窗口归类：

- **本期新增**：`created` 落在窗口内。
- **本期解决**：`resolved` 落在窗口内（无论何时创建）。
- **carryover（仍未结案）**：`status ∈ {open, investigating}` 且 `created` 早于窗口结束。
- 统计量：按 severity / category / project 计数；`time_spent` 求和（解析 `3h`/`2d` 等，无法解析则跳过并在脚注注明）；detailed 档再算平均解决时长（resolved − created）。

## 3. 三档详略

见 `templates/weekly-report.md`：

- **brief**：计数 + 解决/仍卡的单行清单。
- **standard**（默认）：概览 + 分布表 + 解决明细 + carryover + 关键经验。
- **detailed**：逐条迷你卡 + 趋势 + 复发提示 + 下周看板。

月报/季报复用同骨架，仅改标题与 period 粒度；月报建议默认 `standard`，季报默认 `detailed`。

## 4. 输出路径与命名

- `reports/weekly/<YYYY>-W<ww>-<level>.md`
- `reports/monthly/<YYYY>-<MM>-<level>.md`
- `reports/quarterly/<YYYY>-Q<n>-<level>.md`

同窗口同档位重复生成则**覆盖**（报告是派生物，可随时重算）。不存在的子目录先创建。

## 5. 生成步骤

1. 读 `conventions.md`；确定粒度与档位（缺省 weekly/standard）。
2. 计算窗口标签与日期范围。
3. 扫 `problems/`，按 §2 口径分桶、统计。
4. 取 `templates/weekly-report.md` 对应骨架，填实占位，遵守 conventions §7/§8。
5. 写到 §4 的路径；回执路径 + 一句话摘要（新增/解决/open 计数）。
6. 若本期无任何活动，仍生成一份"本期无新增/解决"的空报告并说明，而非静默不产出。

## 6. 自动调度（手动 + 无人值守）

**手动**：随时 `/dev-forge report weekly standard`。

**无人值守**：用系统调度器以 headless 模式调用 Claude Code CLI（本地运行，能访问本地私库）。

- Windows 任务计划程序（每周一 09:00 出上周报）：

```
schtasks /create /tn "dev-forge weekly" /sc weekly /d MON /st 09:00 ^
  /tr "claude -p \"/dev-forge report weekly standard\""
```

- macOS / Linux crontab（每周一 09:00）：

```
0 9 * * 1 claude -p "/dev-forge report weekly standard"
```

- 月报（每月 1 日）/ 季报（季首）同理改 `/sc monthly` 或 cron `0 9 1 * *`，命令换成 `report monthly` / `report quarterly`。

前提：已安装并登录 Claude Code CLI。`-p` 为一次性 headless 执行。会话内临时轮询可用 `/loop`，但持久定时建议用上面的系统调度器。调度跑完可选 git 提交或同步，超出本套件范围。
