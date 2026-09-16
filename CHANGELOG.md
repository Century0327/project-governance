# Project Governance — 变更记录（CHANGELOG.md）

本文件记录 project-governance Skill 的版本变更。格式：日期 + 版本 + 变更内容。新条目加在顶部。

## [1.2.1] — 2026-09-16

外部 review（DeepSeek/Kimi 提议三层架构 vs GPT 主张最小补丁）后的语义审校补丁：把"作者项目习惯"从通用规则中剥离，修正触发后的默认工作流。只改 SKILL.md，不动 CLI / 模板 / tests。

### 改进

- **第 3 步改为「按任务需要执行治理」**：原"日常维护（每个会话）"会使 Skill 触发后默认读全套治理文件并强制更新三件套；现改为"先判断本次任务需要哪些治理信息，只读取相关文件，不默认读全套"，附任务→治理文件映射表。
- **索引优先而非索引绝对**："禁止盲目搜索"改为"索引优先定位；索引缺失/过期/无法定位时搜索文件，并视情况更新索引"，避免把"索引优先"变成"索引绝对权威"。
- **具体机制下沉**：`score > 0.85` 阈值、`permanent_ban` 字段名等项目具体细节从通用层移除，改为"优先继承已验证条目 / 不使用已标记禁用的条目"，具体字段与阈值以模板和项目规范为准。
- **条件式更新**：任务结束由"必更新三件套"改为"只更新发生了变化且有持久价值的治理文件；没有对应变化时，不创建或更新治理记录"。
- **description 精简**：708 字符（中英近乎重复）→ 约 310 字符；末句由功能宣传改为触发场景表述；触发判别信息密度优先于词覆盖率。
- **何时使用/何时不用压缩**：触发判断由 frontmatter 负责，正文只保留边界（4 条何时使用 / 2 条何时不用）。

### 说明

- `triggers` 保留 10 条，但定性为**社区约定/辅助元数据**：trae-skills 仓库公开标准 frontmatter 仅 `name + description`（description 决定何时加载），本地抓取的 Trae 官方文档亦未提及 triggers，故不将其表述为官方触发机制。
- 审校方法：P1/P2 语义审校（通用 Skill 规则 vs 项目具体规则）。佐证实例——作者本人项目规则"找文件时绝对禁止直接搜索"强于通用层应有强度，说明具体规则不能平移到通用层。

### 测试

- `tests/test_governance.py` 92 个用例全部通过（stdlib-only），SKILL.md 补丁不影响 CLI 逻辑。
- 验证设计：6 用例梯度（A 普通 bug 修复 / B 跨会话进度 / C 版本选择 / D 初始化 / E README / F 明确治理任务但只需局部能力 index+VERSIONS），核心判据 = 触发后是否只执行与当前任务相关的治理动作。
- **AI 模拟评审（2026-09-16，见 docs/ab-test-ai-sim-20260916.md）**：隔离项目上 1.2.0 vs 1.2.1 成对推演 6 用例——1.2.0 全部默认"会话开始读三件套"，B/C/F 超额读 2-3 个治理文件，E 被升级成更新 3 个治理文件；1.2.1 每例收敛到映射表对应文件，F 只取 index+VERSIONS。**判定通过**；观察项 1：A 结束时仍记 CHANGELOG+handoff（P2 裁量边界）。
- **人工粗略感知（用户日常使用 1.2.1）**：待补充。

## [1.2.0] — 2026-09-10

基于三轮审计（机制漏洞横向审计、Anti-Overfitting 变更预算、动态行为审计与校准）的最小充分修复，共 14 项。

### 新增
- **A-2 index 人工内容保护**：`## Root layout` 与 `## Change log` 之间检测到人工内容时，`index` 默认停止并提示；确认覆盖需加 `--force`（仅解除人工内容保护，不改变其他生成逻辑）。
- **A-3 Record time 独占更新**：index.md 头部 `Record time` 由 `index` 命令每次更新（UTC），不再依赖 init 时的静态日期。
- **A-4 index/check 参数同源**：`index` 把生效的 `max_depth` / `max_note_length` / 区块标题写入 index.md 头部元数据注释；`check` 优先读取（显式 CLI 参数 > 元数据 > 默认值），旧项目无元数据时回退默认、不误报。
- **A-9a Expected/Actual 差异输出**：`check` 判定索引过期时，输出差异清单（当前 vs 期望，最多 20 行），判定逻辑不变。
- **A-9b 三类防噪 warning**：未替换的 AUTO 占位符残留、可解析版本不一致、notes 失效路径——均不阻塞通过；合法待填占位符、旧项目缺元数据、未启用的备注功能一律不报警。

### 改进
- **A-1 CHANGELOG 规范**：模板明确唯一事实源（index 内嵌 Change log 仅镜像）、新条目在顶部、记录"决策/状态变化"而非流水账；"记录前自查去重"保留为操作建议，不增加硬性检查。
- **A-5a 模板示例化 + 草案标注**：LESSONS / ARCHITECTURE / PROJECT 模板的占位符替换为示例条目并标注 **DRAFT — 未经验证**，避免未填写占位符被当作已确认事实。
- **B-1 / B-5a / M-1 AGENTS 规则收敛**：索引未命中时"不盲搜、报告缺口"（消除与禁止盲搜的措辞冲突）；未映射目录默认只读保守处理；新增"最小更新原则"（不顺手重构、不为统一而统一、不超出裁定范围）。
- **H-1 handoff 进行中任务登记**：模板新增 In-progress work 区块（当前任务 / 进度 / 已确认项）。
- **A-7a / A-10a 文档化**：README/SKILL 明确 index 区块所有权边界、`--force` 语义、隐藏文件排除。

### 测试
- `tests/test_governance.py` 由 76 增至 92 个用例，全部通过（stdlib-only，无需 pytest）。
- A-2 三场景回归：纯机器块正常更新 / 机器块间有人工内容默认停止不覆盖 / `--force` 才覆盖且不改变其他逻辑。

## [1.1.0] — 2026-08-20

SkillHub 适配 + 评测反馈优化。

### 新增
- 按 SkillHub 规范适配 frontmatter：新增 `license: MIT-0`、`template`、`triggers`（5 中英触发词）、`token_budget`。
- 新增 `DESIGN.md`（8 条 ADR 架构决策记录）。
- 新增 `CHANGELOG.md`（本文件）。
- README 新增「适合谁」章节与「常见问题（FAQ）」8 条。

### 改进
- **统一中文说明**：SKILL.md 正文由英文为主改为中文为主；governance.py 全部错误提示、警告、帮助文本中文化。
- **错误提示改进**：所有校验错误附带 `HINT` 修复指引；新增 `FIELD_HELP` 字典说明各字段用途与示例。
- **跨平台兼容**：SKILL.md / README 明确开放 Agent Skills 规范，一套 SKILL.md 可用于 Trae / Claude Code / OpenClaw；CLI 仅依赖 Python 标准库。
- **简化初始化**：README 明确 `init` 只需 `--project-dir`，`--project-name` 可选。

### 测试
- `tests/test_governance.py` 76 个用例全部通过（stdlib-only，无需 pytest）。

## [1.0.0] — 2026-08-17

首个发布版。

- 核心功能：init / validate / index / check 四个 CLI 子命令。
- 模板：AGENTS.md / index.md / VERSIONS.md / LESSONS.md / session_handoff.md / CHANGELOG.md / blacklist.json / whitelist.json 等 11 个治理文件模板。
- 设计理念：治理文件优先于平台记忆；文件权威等级；先查索引再找文件。
