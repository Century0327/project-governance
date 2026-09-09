# Project Governance — 设计文档（DESIGN.md）

本文件记录 project-governance Skill 的关键架构决策（ADR）。每个决策说明：背景、选择、权衡。

## ADR-1：治理文件优先于平台记忆

- **背景**：AI 长期项目里，平台记忆（用户画像/项目记忆）只是上下文来源，不可靠；AI 会把记忆当权威导致用错版本、重复犯错。
- **决策**：建立文件化治理体系（AGENTS.md / index.md / VERSIONS.md / LESSONS.md 等），冲突时治理文件优先。
- **权衡**：多一层文件维护成本，换来 AI 换会话/换模型后仍能正确接续。

## ADR-2：JSON 台本作为唯一输入

- **背景**：AI 项目需要可复现、可校验的输入。
- **决策**：治理工作区以结构化 JSON（blacklist/whitelist 注册表）承载参数，CLI 负责校验 schema。
- **权衡**：JSON 比 Markdown 严格，但换来确定性校验与自动化门禁。

## ADR-3：CLI 仅依赖 Python 标准库

- **背景**：跨平台（Trae / Claude Code / OpenClaw）可用性要求无第三方依赖、无网络。
- **决策**：`governance.py` 只用 argparse/json/pathlib 等标准库。
- **权衡**：功能受限（无 fancy 输出），但零安装成本、零网络风险，安全扫描友好。

## ADR-4：幂等且确定性

- **背景**：重复运行同一命令不应破坏已有文件。
- **决策**：`init` 默认跳过已存在文件，仅 `--force` 覆盖；`index` 只替换固定区块。
- **权衡**：用户需显式 `--force` 才能覆盖，避免误操作。

## ADR-5：错误提示带 HINT 修复指引

- **背景**：评测反馈"错误提示生硬，只说不符合格式不说怎么改"。
- **决策**：所有校验错误附带 `HINT`，说明字段用途、示例与修复方法（FIELD_HELP 字典）。
- **权衡**：错误信息更长，但用户无需查文档即可修复。

## ADR-6：中文为主、关键术语双语

- **背景**：评测反馈"部分地方保留英文说明"；目标用户以中文为主，但需兼容英文平台。
- **决策**：SKILL.md / README / CLI 输出以中文为主，frontmatter 保留英文触发词与描述。
- **权衡**：中文用户上手快，英文平台仍可通过 triggers/description 触发。

## ADR-7：文件权威等级显式化

- **背景**：AI 会把历史文件当权威（"文件存在 ≠ 文件有效"）。
- **决策**：定义 AUTHORITATIVE → STABLE → EXPERIMENTAL → HISTORICAL → DEPRECATED → ARCHIVED 等级，写入模板与规则。
- **权衡**：需要人工维护版本状态，但杜绝 AI 用错版本。

## ADR-8：开放 Agent Skills 规范、Agent 中立

- **背景**：评测反馈"主要针对 Trae 优化，其他平台兼容性一般"。
- **决策**：遵循开放 Agent Skills 规范；治理文件用业界通用 `AGENTS.md` 约定；CLI 无平台绑定。
- **权衡**：失去部分平台专属能力（如自动 Skill 发现），换来跨平台可用。

## ADR-9：index 目录树区块由命令独占维护（人工内容保护）

- **背景**：suhu 用户曾人工编辑 index.md（注释/调整），随后运行 `index` 被静默覆盖；旧规范只说"更新 index"，未显式声明树块所有权，AI 与人工都可能手动编辑。
- **决策**：`## Root layout` 与 `## Change log` 之间的机器区块由 `governance.py index` 独占维护；检测到该区间存在人工内容时默认停止并提示，仅 `--force` 允许覆盖（只解除此保护，不改变其他生成逻辑）。人工补充内容写入 `index_notes.json`，或 `## Change log` 之后的自定义 section。
- **权衡**：用户需显式 `--force` 才能覆盖人工内容，换来"不静默丢数据"；`--force` 语义被严格限定，避免演变成"万能覆盖"。

## 变更记录

详见 `CHANGELOG.md`。
