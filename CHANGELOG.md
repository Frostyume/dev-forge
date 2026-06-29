# Changelog

本项目遵循语义化版本（SemVer）。

## [0.5.0] - 2026-06-28

新增自动捕获（auto-capture），把「记得要记」这一步也省掉。

### 新增

- **`auto` 动作（自动捕获，opt-in，默认关闭）**：开启后 dev-forge 在你**经 Claude Code 开发**的过程中**静默**记录遇到并解决的非平凡问题，免去每次手动 `log`。含 `auto on / off / status / scope` 四个子命令；范围（`scope`）、严重度闸（`min_severity`，默认 medium）、根因去重等均可在 `config.json` 的 `auto_capture` 调。
- **受管触发块默认装在全局 `~/.claude/CLAUDE.md`**（而非项目 CLAUDE.md），**不写入任何代码仓库**，延续 dev-forge「不污染仓库」原则；`auto off` 幂等精确移除。
- **`reference/auto-capture.md`**：自动捕获完整契约——config 结构、受管块模板、静默捕获规则（非平凡阈值 / 根因去重 / resolved-open 判定 / 极简回执）、与手动 `log` 的关系、L1 局限与 L2/L3 hook 升级路径。
- **插件封装（可作为 Claude Code 插件安装）**：新增 `.claude-plugin/plugin.json`，使 dev-forge 既能作为裸 skill 克隆，也能作为插件一键安装（`/plugin marketplace add Frostyume/dev-forge` → `/plugin install dev-forge`），并够资格投递官方社区插件市场；仓库结构不变，原 skill 安装方式照常可用。

### 说明

- 自动捕获为 **L1（实时静默）**：高可靠但非 100%，依赖 Claude 在会话中遵循受管块，且仅覆盖经 Claude Code 的开发；确定性兜底（hook）列入路线图。
- 默认关闭、静默、本地——开启后所有记录仍 100% 留在 `~/.claude/dev-forge/`，绝不外传。

## [0.4.0] - 2026-06-25

聚焦面向全球用户的命令一致性与英文文档纯净度。

### 变更

- **report 档位参数英文化**：详略档位由 `简洁 / 标准 / 详细` 改为 `brief / standard / detailed`（默认 `standard`），命令参数对全球用户统一英文；报告 frontmatter 的 `level` 字段与派生文件名（`<YYYY>-W<ww>-<level>.md`）随之英文化。
- **英文 README 去中文**：清除 `README.md` 中残留的中文（示例 `title`、正文五节名、`report` 档位示例），使英文文档纯英文且与命令实际行为一致；中文 README 散文保持中文，仅命令参数同步为英文。

## [0.3.0] - 2026-06-25

聚焦开源社区可用性与协作入门。

### 新增

- **README 中英可切换**：拆成 `README.md`（英文）与 `README.zh-CN.md`（中文）双文件，顶部互链切换；各自补 **FAQ**（数据位置 / 隐私 / 触发排查 / 备份 / Windows 无 BOM / 无需 Git）。
- **`help` 动作**：`/dev-forge` 无参数时打印命令菜单，点出 `log → recall → lessons` 主线，纯展示不读写文件。
- **协作入门件**：`CONTRIBUTING.md`（含如何提议扩展 taxonomy）、`CODE_OF_CONDUCT.md`（Contributor Covenant 2.1）、issue 模板（bug 模板复用 dev-forge 自身的描述/根因/影响 schema；feature 模板带"是否契合根因知识库定位"自检）。

## [0.2.0] - 2026-06-25

聚焦差异化定位：从「一录多用的产出物套件」收敛到「**本地根因知识库 + 永不重复踩坑**」这一核心，把结构上难被复制的能力做深。

### 变更

- **重定位叙事**：README 头条与 SKILL 自述从「自动出周报/简历」改为「本地踩坑根因知识库 / 永不重复 debug 同一个坑」，把周报与简历明确降格为下游副产物；新增「How is this different」诚实对标 brag-doc / git→简历 类工具（根因 vs 成就、本地 vs 云、skill vs app）。
- **recall 升级为根因级**：以「根因」为首要匹配维度并加权，归并同源根因为复发组。
- **lessons 升级为可复用避坑清单**：从被动经验摘录改为祈使式、按 category 分组、带复发计数与来源 id、可勾选的 pre-flight checklist。

### 新增

- **log 根因级防重闸**：录入前自动以根因比对历史，命中同源根因即提示「疑似重复踩坑」并让用户选择补充旧记录或新建。
- **`reference/interop.md`**：把 YAML schema 文档成开放可编程契约，给出 ripgrep / yq / Obsidian Dataview 配方与字段稳定性承诺，吃住「本地 + 可编程」生态位。
- **`examples/lessons.md`**：由两条示例问题蒸馏出的避坑清单样例。

## [0.1.0] - 2026-06-24

首个公开版本。

### 新增

- **log / resolve**：结构化录入与结案（描述 / 根因 / 影响 / 解决方案 / 经验教训），自动生成 `P-YYYYMMDD-NN` ID，尽力抓取 git 上下文（repo/branch/commit/fix_commit）。
- **report**：周 / 月 / 季战报，brief / standard / detailed 三档；含 Windows 任务计划与 cron 无人值守调度方案。
- **resume**：把已解决问题锻造成 STAR 简历项目，中英双语活文档，支持按 JD 裁剪与脱敏，设真实性红线。
- **stats**：按严重度 / 类别 / 项目的度量仪表盘，含解决时长与复发热点。
- **recall**：旧坑回溯检索，并接入 log 录入时自动查重。
- **open**：未结案迷你 issue tracker，过期高亮。
- **lessons**：跨问题经验教训萃取成可复用清单。
- 全部数据本地存于 `~/.claude/dev-forge/`，不写入任何代码仓库。
- 内置 ID 跳号不复用、YAML 转义、UTF-8 无 BOM、git 缺失优雅跳过等健壮性约定。
