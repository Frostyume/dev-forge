# 增值模块：stats / recall / open / lessons

四个独立动作，共享 `problems/` 数据源。执行前先读 `conventions.md`。`stats`/`recall`/`open` 默认把结果**打印到对话**（派生物，不落盘）；`lessons` 落盘到 `~/.claude/dev-forge/lessons.md`。

## stats —— 仪表盘 Dashboard

`/dev-forge stats [--project <p>]`

扫 `problems/`，输出度量（markdown 表格，遵守 conventions §7）：

- **总览**：总数、open/investigating/resolved/wontfix 计数、解决率（resolved / 非 wontfix）。
- **分布**：按 severity、category、project 各一张计数表。
- **时长**：`time_spent` 求和；平均与中位解决时长（resolved − created，仅对已解决）。
- **复发热点**：同 category 高频项、或标题/根因高度相似的反复问题，点名提示。
- **最该处理**：未结案中按 `severity` 降序、`created` 升序取 Top 5。

可加 `--save` 时另存 `reports/dashboard-<YYYY-MM-DD>.md`。

## recall —— 根因级旧坑回溯 Recall

`/dev-forge recall <关键词>`

在 `problems/` 中检索，**以「根因」为首要匹配维度**（不只是标题关键词命中）：检索 `title` / 问题描述 / **根因** / `tags` / `category`，对**根因**字段的命中给更高权重，按相关度排序返回：

- 每条：`<id>` · `<标题>` · `<status>` · **根因一句话**（命中重点） · 解决一句话（若已解决） · 距今 `<D>` 天。
- 当多条旧坑**根因同源**（如都归因于「PS5.1 缺某 .NET Core API」「混合精度前向溢出」），把它们归并成一组并点出共同根因——这正是该沉淀进 `lessons` 的复发模式。
- 命中 0 条时明说"无历史记录"，不要编造。

**与 log 的根因级防重联动**：`log` 录入时自动用新问题的标题 + 根因关键词跑一次 recall（见 SKILL.md log 步骤）。命中**同源根因**的旧坑时，显式提示「⚠️ 疑似重复踩坑：你 `<D>` 天前在 `<project>` 已踩过同款根因 `<id>`」，并让用户选择：补充到旧记录 / 确为新问题再新建。目标是在你**重新浪费一天去解同一个根因之前**就拦下你。

## open —— 未结案追踪 Tracker

`/dev-forge open [--project <p>] [--severity <s>]`

列出 `status ∈ {open, investigating}` 的全部问题，构成迷你 issue tracker：

- 排序：`severity` 降序 → `created` 升序（最老最严重置顶）。
- 每条：`<id>` · `<标题>` · `<severity>` · 已 open `<D>` 天 · `<project>` · 下一步（若正文有）。
- **过期高亮**：open 超过 14 天的标 ⚠️ 提醒。
- 结尾给一句话总结（共 N 项未结案，其中 high/critical M 项）。

## lessons —— 可复用避坑清单 Pre-flight Checklist

`/dev-forge lessons [--category <c>] [--project <p>]`

把所有已解决问题的「经验教训」**蒸馏成一份预防式避坑清单**（pre-flight checklist）——不是被动的经验摘录，而是**开工/改动前主动要过的检查项**，落盘 `~/.claude/dev-forge/lessons.md`。这是 dev-forge 区别于「成就记录」类工具结构上做不到的核心：把踩过的坑转化为**下次不再踩**的可执行检查。

生成规则：

- **每条写成祈使式检查项**，而非陈述式回忆。把「`-Encoding utf8` 在 PS5.1 会带 BOM」改写成「在 PS5.1 写 .md/.json 前，确认用无 BOM 的 UTF-8（Write 工具或 `UTF8Encoding($false)`）」。
- 用 `- [ ]` 复选框语法承载每条检查项，便于在编辑器/Obsidian 里实际勾选。
- **按 `category` 分组**（config / dependency / env / numerics …），组内可按「适用阶段」（写代码前 / 提交前 / 部署前）再细分，让用户在对应时机一眼扫过。
- 每条附**来源 `<id>`**（可多个）。同一根因被多条问题命中的，标注「⟳ 已踩 N 次」并在组内置顶——它最该被清单拦住。
- 合并语义重复的教训，保留信息量最大、最可执行的措辞；丢弃无法转成「下次怎么做」的纯感慨。
- 该文件每次运行**覆盖重算**（唯一真相源始终是 `problems/`），会随你解决的坑持续生长。
- 输出遵守 conventions §7/§8。

理想用法：开一个新项目、或动一段危险代码前，跑 `/dev-forge lessons --category <相关类>` 扫一遍对应清单，把过去的你流过的血变成开工前的预检表。
