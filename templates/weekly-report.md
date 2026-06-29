# 战报模板（weekly / monthly / quarterly 通用，三档详略）

下面三个骨架对应 `brief / standard / detailed` 三档。生成时按所选档位取对应骨架，把占位填实，并遵守 conventions §7（表格前后空行、无 `---` 分割线、无多余空行）。月报/季报复用同结构，仅把标题与 period 改为对应粒度。

## 档位 A：brief

适合往群里贴、给自己留痕。约 6–12 行。

```
---
type: weekly-report
period: <2026-W26>
range: <2026-06-22 ~ 2026-06-28>
level: brief
generated: <YYYY-MM-DD>
---
# 周报 <2026-W26>（<06-22 ~ 06-28>）
本周新增 <N> · 解决 <M> · 仍 open <K> · 累计耗时 <Xh>。
解决：
- <P-id> <标题>
- ...
仍卡：
- <P-id> <标题>（已 open <D> 天）
```

## 档位 B：standard（默认）

适合正式周报上交。含分布、解决明细、carryover、经验。

```
---
type: weekly-report
period: <2026-W26>
range: <2026-06-22 ~ 2026-06-28>
level: standard
generated: <YYYY-MM-DD>
projects: [<...>]
---
# 周报 <2026-W26>（<06-22 ~ 06-28>）
## 概览
本周新增 <N> 个问题，解决 <M> 个，仍 open <K> 个，累计耗时 <Xh>。

| 维度 | 分布 |
|---|---|
| 严重度 | critical <a> · high <b> · medium <c> · low <d> |
| 类别 | <category>:<n> · ... |

## 本周解决
- **<P-id> <标题>**（<severity>，<耗时>）根因：<一句话> → 方案：<一句话>
- ...
## 仍未结案（carryover）
- <P-id> <标题>（<severity>，已 open <D> 天，下一步：<...>）
## 关键经验
- <从本周 Lessons 提炼，1–3 条>
```

## 档位 C：detailed

适合月度复盘的素材、写进项目文档。每个问题一张迷你卡 + 趋势 + 下周看板。

```
---
type: weekly-report
period: <2026-W26>
range: <2026-06-22 ~ 2026-06-28>
level: detailed
generated: <YYYY-MM-DD>
projects: [<...>]
---
# 周报 <2026-W26>（<06-22 ~ 06-28>）
## 概览
新增 <N> · 解决 <M> · 仍 open <K> · 累计耗时 <Xh> · 平均解决时长 <avg>。

| 维度 | 分布 |
|---|---|
| 严重度 | critical <a> · high <b> · medium <c> · low <d> |
| 类别 | <category>:<n> · ... |
| 项目 | <project>:<n> · ... |

## 本周解决（逐条）
### <P-id> <标题>
- 严重度/类别/耗时：<severity> / <category> / <耗时>
- 现象：<描述摘要>
- 根因：<root cause>
- 影响：<impact>
- 方案：<solution>
- 教训：<lessons>
- 关联：<repo>@<fix_commit>（如有）
## 仍未结案（carryover）
- <P-id> <标题>（<severity>，已 open <D> 天，<status>，下一步：<...>）
## 复发/反复
- <若有同 category 反复出现，点名提示>
## 下周看板
- <把 carryover 中 high/critical 列为重点>
```
