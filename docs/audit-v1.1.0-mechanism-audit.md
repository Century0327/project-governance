# project-governance v1.1.0 机制漏洞横向审计报告

- 审计日期：2026-09-10
- 审计对象：d:\Stable Diffusion\workspace\drafts\00顶层\社区贡献\project-governance（v1.1.0，17 个文件全部通读）
- 审计方法：把 CHANGELOG 案例抽象为审计模式（canonical location / format / owner / source / trigger 缺失），按 GPT 给出的 10 个维度逐项审查 + 反向审计（"能力很强但会合理自由解释规则的 AI"合同自由度测试 + "很忙不会主动维护文档的用户"模拟）
- 审计目标：判断"规范缺失导致 AI 自由解释"是孤立问题还是系统性特征

## 核心结论

**CHANGELOG 暴露的问题是系统性特征，不是孤立案例。**

对 17 个文件的逐条审查显示，同一机制缺陷以不同形态反复出现：**多数治理文件都留下了"多个都说得通的选择"，或把维护责任默认推给人类**。全仓共发现 A 类问题 11 项、B 类 6 项、C 类 5 项、D 类确认无问题 6 项。

最有力的证据链：

1. **变更记录存在三个事实源、三种结构**：`templates/CHANGELOG.md`（表格 + Judge 列）、`index.md` 内嵌 `## Change log`（表格、无 Judge 列）、作者自己维护的 `CHANGELOG.md`（Keep a Changelog 标题格式）——模板定义的格式作者本人都不用。
2. **index 命令会静默删除 root↔changelog 之间的人工内容**（`governance.py cmd_index` 直接拼接 `lines[:start] + section + lines[end:]`，无警告）——suhu 项目人工编辑的树与注释一旦重跑 index 即丢失，这是 suhu 停用 index 命令、check 永久报过期的直接原因。
3. **占位符有三种语义无人区分**：`{{...}}` 既表示"init 自动替换"（`{{PROJECT_NAME}}`）、"AI 应填充"（`{{ARCHITECTURE_OVERVIEW}}`）、"示例占位"（LESSONS 的 `{{N}}`/`{{TITLE}}`），且无任何门禁检测未替换占位符。

---

## A 类：已明确存在的问题（有直接证据）

### A-1 变更记录无 canonical location（现象源头）
- **现象**：AI 更新 CHANGELOG 时有时插头部有时追加尾部（suhu 观测）；作者自己的 CHANGELOG 用标题格式，模板却定义表格格式。
- **直接原因**：模板只给示例行 `| {{DATE}} | Initial record | human | ... |`，未定义新记录插入位置；`AGENTS.md` 规则 7 只说"更新 CHANGELOG.md"，无位置规定。
- **机制根因**：P1 canonical location 缺失——"更新 CHANGELOG"存在顶部/底部/示例行上/示例行下多个合法选择。
- **为什么现有设计允许**：模板、AGENTS.md、SKILL.md 三处均未定义唯一位置；check 只查文件存在，不查位置/格式。
- **如何验证**：对空白 CHANGELOG、含 1 条记录的 CHANGELOG、格式混乱的 CHANGELOG 分别让 AI 记录一条，观察位置是否漂移。
- **修复原则**：定义"新记录 MUST 插入表格第一行数据之前，禁止追加文末；现有文件结构不合规时先报告，不自行选择布局"。

### A-2 index 命令静默删除人工内容（suhu 实证根因）
- **现象**：suhu 的 index.md 树被人工加注释、调整 experiments4 位置、添加"正式实例"section；重跑 `index` 后这些内容全部消失（root 与 changelog 之间被整段替换）。
- **直接原因**：`cmd_index` 用 `"".join(lines[:start]) + section + "".join(lines[end:])` 重建，root→changelog 之间的**一切内容（含人工 section）无警告删除**。
- **机制根因**：P2 source of truth / P3 ownership 未定义——index.md 是"生成物"还是"人工知识库"？二者皆是的混合所有权失败。
- **为什么现有设计允许**：README FAQ Q4 还指引用户"check 说过期就跑 index 重建"，等于指引用户丢失人工内容；测试 A9 只验证幂等性，未覆盖"区块间有额外内容"场景。
- **如何验证**：在 index.md 的 root 与 changelog 之间插入人工 section，跑 `index`，观察其消失且无提示。
- **修复原则**：index 只管理自己生成的块；人工区块保留或移入独立文件；覆盖前检测区块间人工内容并告警。

### A-3 index 头部时间戳永不更新
- **现象**：suhu index.md 头部 `Record time: 2026-08-23` 永久停留，内容已更新到 09-04。
- **直接原因**：`cmd_index` 只替换 Root layout 区块，不碰头部；`Record time: {{DATE}}` 由 init 一次性写入。
- **机制根因**：P3 ownership / P4 trigger 未定义——timestamp 语义（"最后生成时间"？"最后变更时间"？）与负责方（谁更新、什么事件触发）均未声明。
- **为什么现有设计允许**：模板头部注释未定义语义；check 也不校验时间戳。
- **如何验证**：init 后跑 `index`，观察头部日期不变化。
- **修复原则**：定义 timestamp = "index 命令上次生成时间"，由 index 命令独占更新；头部同时记录文件树 mtime 摘要。

### A-4 index 与 check 的 max_depth 参数独立默认，互相不一致即误报
- **现象**：用 `--max-depth 6` 生成 index 后，`check`（默认 4）必然报"index 已过期"。
- **直接原因**：`cmd_index` 与 `cmd_check` 各自 `default=4`，无共享配置；`_is_index_fresh` 用 `args.max_depth` 重建树对比。
- **机制根因**：P5 可验证性——检查参数与生成参数不同源，验证必然失真。
- **为什么现有设计允许**：两命令的 parser 独立定义默认值；测试 C1/C5 均用默认深度，未覆盖深度不一致场景。
- **如何验证**：`index --max-depth 6` 后 `check`，观察误报。
- **修复原则**：index 生成时把生效的 max_depth/区块名写入 index.md 头部元数据，check 从头部读取同源参数。

### A-5 占位符生命周期缺失（三义混用、无门禁）
- **现象**：init 后 `ARCHITECTURE.md`/`PROJECT.md` 保留整篇 `{{ARCHITECTURE_OVERVIEW}}`/`{{PROJECT_GOAL}}` 等英文占位符；`LESSONS.md` 顶部保留 `## Error {{N}}: {{TITLE}}` 裸模板块（suhu 至今未删）；AI 首次运行读 PROJECT.md 时无法区分"模板"与"项目状态"。
- **直接原因**：`AUTO_PLACEHOLDERS` 只替换 4 个键，其余 `{{...}}` 一律保留且无标识；全仓无"未替换占位符"检测。
- **机制根因**：P4 lifecycle / P5 可验证性——占位符允许存在与否、谁负责替换、提交前是否必须消失，均未定义。
- **为什么现有设计允许**：模板自身用 `{{...}}` 表达三种不同含义而不加区分；check 不扫描占位符。
- **如何验证**：init 后 grep 生成的 11 个文件中的 `{{`，观察残留规模；让 AI 读 PROJECT.md 判断项目目标，观察其是否把占位符当真实内容。
- **修复原则**：统一占位符语法并分区（自动替换 / 待填 / 示例）；check 增加 placeholder 门禁；模板待填区改为"示例+删除标注"。

### A-6 superseded_by 是"幽灵字段"
- **现象**：`AGENTS.md` 参数规则 5 与 blacklist/whitelist 模板 note 均指导"用 `superseded_by` 标记取代"，但 validate 不校验该字段、README「注册表 Schema」不收录、`FIELD_HELP` 无说明。
- **直接原因**：模板与规则文档引入 schema 未定义的字段；validate 对未知字段采取"放行"策略（测试 A7 明确 forward compat）。
- **机制根因**：P1 canonical format——"标记取代"存在两种合法机制（`status: superseded` 与 `superseded_by` 字段），无优先级规定。
- **为什么现有设计允许**：注册表 status 枚举含 superseded，而 note 又推荐字段；两处文档不一致，validate 两边都放行。
- **如何验证**：分别用 `status: superseded` 和 `superseded_by` 记录同一替换，观察 validate 均通过、AI 无法判断哪种正确。
- **修复原则**：二选一定为唯一机制（建议 `superseded_by` 字段 + validate 校验其存在性与目标 id 存在性），README/FIELD_HELP 同步。

### A-7 变更记录三个事实源、三种结构
- **现象**：`CHANGELOG.md`（独立文件，表格+Judge 列）、`index.md` 内嵌 `## Change log`（表格，无 Judge 列）、作者实际维护的 `CHANGELOG.md`（标题格式）——同一"变更记录"概念三处实现、三套格式。
- **直接原因**：模板定义了 A 格式，作者自己用了 B 格式，index 模板又内嵌了 C 格式的 changelog；SKILL.md 说 CHANGELOG 是"决策与版本历史"，README 说是"项目为什么变成现在这样"，模板说是"decisions and outcomes + judge 区分"——用途定义也漂移。
- **机制根因**：P2 source of truth——同状态多文件记录，无 canonical source，且二级副本（index 内嵌 changelog）无失效规则。
- **为什么现有设计允许**：init 同时生成 index.md（含 changelog 块）与 CHANGELOG.md；AGENTS.md 规则 7 说"any decision updates CHANGELOG.md"未提 index 内嵌块，但 After Completing a Task 又让两者都更新。
- **如何验证**：记录一条决策，观察 AI 写哪个文件、index 内嵌块是否同步。
- **修复原则**：定义唯一事实源（独立 `CHANGELOG.md`），index 内嵌 changelog 块删除或改为只读镜像（check 校验一致性）。

### A-8 VERSIONS 定义 6 级权威等级，表格只支持 2 态
- **现象**：模板列出 AUTHORITATIVE→ARCHIVED 6 级，记录表格只有 `Stable?`（yes/no）一列，EXPERIMENTAL/HISTORICAL/DEPRECATED 无处记录（只能塞 Notes）。
- **直接原因**：表格 schema 未与等级模型对齐。
- **机制根因**：P1 canonical format——分类体系与记录格式脱节。
- **为什么现有设计允许**：模板作者未意识到 6 级模型需要对应记录列；validate 也不检查 VERSIONS 格式。
- **如何验证**：让 AI 记录一个 EXPERIMENTAL 版本，观察其只能写进 Notes 或干脆不记。
- **修复原则**：表格加 `Level` 列（枚举 6 级）或明确"仅记录 STABLE/AUTHORITATIVE，其余状态进 Notes 并标注"。

### A-9 check 是"看起来检查了"的假检查
- **现象**：check 名义上是"健康门禁"，实际只验证：8 个文件存在 + 注册表 schema + notes JSON 合法 + index 树新鲜。不检查：占位符残留、时间戳、CHANGELOG 位置/格式、版本一致性（SKILL version↔CHANGELOG）、notes 失效路径、VERSIONS 与 index 一致性、ARCHITECTURE/PROJECT 是否存在（不在 CORE_REQUIRED_FILES）。
- **直接原因**：check 只覆盖了机器可验证的注册表部分，治理核心承诺（版本权威、规则执行、记录质量）全部不可自动验证。
- **机制根因**：P5 可验证性——检测做了，诊断与修复引导没做；且核心约束未被验证。
- **为什么现有设计允许**：测试 76 例全测健壮性（编码/路径/崩溃），无一测治理语义；README 把 check 描述为"健康门禁"过度承诺。
- **如何验证**：构造"占位符残留 + 时间戳陈旧 + 版本号不一致"的坏工作区，check 仍通过。
- **修复原则**：check 增补可机器验证项（占位符、时间戳、版本一致性、notes 失效、CHANGELOG 格式），输出 Expected/Actual 差异；不可验证项列出人工检查清单。

### A-10 隐藏文件被静默跳过且未文档化
- **现象**：`_build_tree` 跳过所有 `.` 开头文件（含 .env 等），README Q7 只提 SKIP_DIRS 目录排除，未提隐藏文件。
- **直接原因**：代码 `not p.name.startswith(".")` 无条件排除隐藏文件。
- **机制根因**：P8 exception handling——排除规则未写入文档，AI 依赖 index 找文件时对隐藏文件一无所知。
- **为什么现有设计允许**：README 的排除说明不完整；测试 idx_hidden 还特意断言"dotfiles excluded"（把行为固化为正确）。
- **如何验证**：项目根放 `.env`，跑 index，观察其不在树中且无任何提示。
- **修复原则**：文档化排除规则；对 `.*` 文件提供显式纳入选项（如 `--include-hidden`）或至少告警。

### A-11 测试覆盖健壮性、不覆盖治理语义
- **现象**：76 用例全部围绕 CLI 边界（编码、路径、对抗输入、幂等），无一条验证治理规范（canonical location、时间戳、占位符、版本一致性、notes 失效、人工区块保留）。
- **直接原因**：测试是从"CLI 工具"视角编写，非"治理系统"视角。
- **机制根因**：P5 可验证性——没有把治理规范翻译成机器断言，规范就只是文字。
- **为什么现有设计允许**：无"规范→断言"的映射清单；A-2 的人工区块场景、A-4 的深度不一致场景均无测试。
- **如何验证**：对照本报告 A 类 11 项，逐一检查是否有对应测试用例（现状：仅 A-9 部分有）。
- **修复原则**：为每条可机器验证的治理规范补测试；引入"规范↔测试"对照表。

---

## B 类：高概率存在、需实验验证

### B-1 AGENTS.md 两处指令冲突
- **现象**：文件头 `Do not modify it without human approval` 与「Project Customization」`edit these for this project` 字面矛盾。
- **直接原因**：同一文件内"禁止修改"与"编辑这些"并存，未消解（合理解释是"改定制区需批准"，但未写明）。
- **机制根因**：P8 exception handling——规则冲突时无优先级。
- **验证**：让 AI 初始化后自主添加项目规则，观察其是否因"禁止修改"而不敢动。
- **修复原则**：明确"定制区可编辑，但每次修改需人确认；通用核心规则区禁止修改"。

### B-2 judge 概念三处不一致
- **现象**：AGENTS.md Judgment Levels 4 级（AUTOMATED/AI REVIEW/HUMAN REVIEW/AUTHORITATIVE）、注册表 `judge` 2 态（ai/human）、CHANGELOG 模板 Judge 列未定义取值。
- **机制根因**：P1 canonical format——同一概念多套枚举。
- **验证**：让 AI 在 CHANGELOG 记录"human review"，观察其写 human/human review/HUMAN REVIEW 哪种。
- **修复原则**：统一为小写 `human`/`ai`（机器可验证），4 级模型只用于 AGENTS 语义说明。

### B-3 VERSIONS"仅人工确认才 stable"与表格 judge 允许 ai 冲突
- **现象**：Rules 说 only human-confirmed stable，表格 `judge` 列写 `human/ai`。
- **机制根因**：P8——两条规则冲突无优先级（若 AI 写 `Stable?=yes, judge=ai` 算不算违规？）。
- **验证**：构造上述条目跑 validate（不检查 VERSIONS），观察通过。
- **修复原则**：judge 列改 human-only 或加"AI 标记必须人工复核"规则 + check 校验。

### B-4 index/check 参数不一致的更多场景
- **现象**：root_section/changelog_section/max_note_length 同样各自独立默认；自定义区块名后 check 若忘传同参数即误报。
- **机制根因**：P5——配置同源性缺失（同 A-4）。
- **验证**：用中文区块名跑 index 后不带参跑 check，观察误报（测试 C7 已显式传参，掩盖了该问题）。
- **修复原则**：同 A-4，参数随 index 生成时写入头部元数据。

### B-5 权限分区与自主权等级无默认值
- **现象**：AGENTS.md 定义 5 个权限分区、4 级自主权，但无"默认哪个目录属于哪个区"；模板把映射留给「Project Customization」人工填写（README Q2 明示"只需编辑 AGENTS.md"）。
- **机制根因**：P6 默认值缺失——人类不填时 AI 自由解释目录归属；同时是"把责任默认推给人类"（GPT 反面案例，同 architecture）。
- **验证**：init 后不编辑 AGENTS.md 直接让 AI 改 scripts/ 下文件，观察其自主决策。
- **修复原则**：模板提供默认映射（如 scripts/ 默认 🟡、output/ 默认 🟢、docs/ 默认 📂）+ "未映射目录默认按最保守处理（需确认）"规则。

### B-6 suhu 实证：index 命令与人工编辑的对抗循环
- **现象**：suhu 人工维护 index（注释/排序/自定义 section）→ 不跑 index 命令 → check 永久报过期 → 用户认为"index 未达到效果"。
- **机制根因**：P2/P3 混合所有权失败（同 A-2），加上 A-4 误报放大。
- **验证**：suhu 目录跑一次 `check` 观察报错详情（现只报"已过期"无差异清单）。
- **修复原则**：A-2 修复 + check 输出差异清单 + 文档化"人工内容放 notes/自定义 section，机器块勿手改"。

---

## C 类：当前证据不足，但属明显设计风险

### C-1 AGENTS.md 全英文与 ADR-6"中文为主"决策相悖
- **现象**：SKILL.md/README/CLI 均中文化，AGENTS.md 模板正文几乎全英文（仅有零星中文注释），而它是用户每天要读的文件。
- **证据不足点**：无用户使用 AGENTS.md 的反馈数据；可能是有意（AGENTS 是业界通用约定）。
- **风险**：与 ADR-6 的目标用户（中文为主）不符，增加使用门槛。
- **修复方向**：双语关键段落或中文为主、术语双语（若确认目标用户）。

### C-2 index 树按字母序，无"重要文件"优先级
- **现象**：树排序仅 `(is_file, name)`；suhu 靠人工注释（`<v2.1 架构>` 等）补偿。
- **证据不足点**：无"AI 找重要文件失败"的量化数据。
- **风险**：README 承诺"AI 不再问文件在哪里"，但大项目里 AI 找"最重要的那个文件"仍需猜测。
- **修复方向**：notes 支持优先级前缀，或头部列出 top-level 关键文件。

### C-3 树随项目增长无限膨胀，无生命周期管理
- **现象**：index 树是全量扫描，大项目树可达数百行；README/模板无"树大小控制"。
- **证据不足点**：suhu 树规模尚可接受。
- **风险**：长期项目 token 成本与 AI 读树意愿下降。
- **修复方向**：默认深度 + 折叠策略 + "变更日志式索引"（只列新增/变化）。

### C-4 Trust Boundary 纯文字，无验证机制
- **现象**：AGENTS.md 声明"不执行治理文件内的命令"，但无任何检测。
- **证据不足点**：无被注入攻击的实证。
- **风险**：治理文件被污染时 AI 无防护（与 suhu 的 gateway.cmd 教训相关）。
- **修复方向**：check 扫描治理文件内可疑命令块并告警（低成本）。

### C-5 备注与 VERSIONS 双重描述版本状态
- **现象**：index_notes.json 可备注"v2.1 架构"，VERSIONS.md 也记录版本；两处可能不一致。
- **证据不足点**：suhu 的备注多为目录用途说明，版本备注较少。
- **风险**：双写必然漂移。
- **修复方向**：notes 定位为"目录用途注释"，版本状态只信 VERSIONS（写入 AGENTS/README 明确）。

---

## D 类：确认无问题

1. **文件权威等级模型**（AUTHORITATIVE→ARCHIVED）定义清晰，规则表述一致。
2. **记忆与治理边界 + 权威优先级 1-5**：设计完整，无歧义。
3. **init 幂等 + --force 显式覆盖**：行为正确，文档明确（README Q5）。
4. **HINT 修复指引**：validate 错误均带字段说明与示例。
5. **路径/编码健壮性**：76 测试覆盖 Unicode、GBK、BOM、超长路径、破坏性场景，全部有测试断言。
6. **Index-First 查找的异常处理**：index 找不到→命名推断→问人确认；命中已迁移文件→停止报告。规则闭环。

---

## 反向审计：合同自由度测试（节选）

把 SKILL 当合同，逐条问"这里有哪些未定义的自由度"，每个场景都给出"两种都说得通的选择"：

| # | 合同条款 | 自由度 A | 自由度 B | 根因类 |
|---|---|---|---|---|
| R1 | "更新 CHANGELOG.md"（AGENTS 规则 7） | 插顶部 | 追加底部 | canonical location |
| R2 | "任何决策更新 CHANGELOG" vs "更新 index" | 只写 CHANGELOG.md | 写 index 内嵌 Change log | source of truth |
| R3 | "用 superseded_by 标记取代"（模板 note） | 加 superseded_by 字段 | 改 status: superseded | canonical format |
| R4 | "记录 judge"（CHANGELOG 模板） | 写 human | 写 HUMAN REVIEW / 人工评审 | canonical format |
| R5 | "更新 index.md"（文件变动时） | 手动编辑树块 | 运行 index 命令 | ownership |
| R6 | "读 ARCHITECTURE.md（任务相关时）" | 占位符=待我填 | 占位符=项目现状 | placeholder lifecycle |
| R7 | "记录稳定版本"（VERSIONS） | 只记 yes/no | 在 Notes 塞 EXPERIMENTAL | canonical format |
| R8 | "按权限分区存放产物" | 无映射时默认 🟢 自由写 | 无映射时默认 🔴 保守 | 默认值缺失 |
| R9 | "新错误记入 LESSONS" | 追加到末尾 | 插到模板示例之前 | canonical location |

结论：9 个抽查场景全部存在双解。**自由度不是个例，而是模板体系的默认状态。**

## 现实用户模拟（很忙、不主动维护）

假设用户 init 后只做最低限度定制（甚至零定制）：

- `ARCHITECTURE.md`/`PROJECT.md` 占位符**永久为空** → AI 把空模板当项目状态（读 PROJECT.md 以为项目没有目标）。
- 权限分区/自主权不填 → AI 按自身偏好自由行为，无保守默认兜底。
- `VERSIONS.md` 不维护 → 权威等级失效，"文件存在≠有效"的防护归零。
- `index_notes.json` 不维护 → 备注陈旧（suhu 实证：仍引用已归档的 docs/experiment-001-design.md）。
- `CHANGELOG.md` 不维护 → 决策历史丢失，交接靠 session_handoff 单点。

**结论**：系统把大量关键状态默认为"人类会维护"，与"人类不会主动维护"的现实相悖——GPT 的判断成立。v1.2.0 应引入"AI 生成草案→人确认→canonical"模式，并让 check 在关键状态缺失时给出可操作告警，而不是静默通过。

---

## 对 v1.2.0 范围的影响

本次审计将原计划从"10 个已知问题的补丁"升级为**按机制类别修复**，优先级排序：

1. **P2/P3（source of truth / ownership）**：index 人工区块保护（A-2）、变更记录唯一事实源（A-7）——先修数据丢失风险。
2. **P1（canonical location/format）**：CHANGELOG 位置（A-1）、superseded_by（A-6）、VERSIONS 等级列（A-8）、judge 统一（B-2）。
3. **P5（可验证性）**：check 差异输出 + 占位符门禁 + 版本一致性 + 参数同源（A-3/A-4/A-5/A-9）+ 治理语义测试（A-11）。
4. **P4/P6（生命周期/默认值）**：占位符生命周期（A-5）、权限分区默认映射（B-5）、handoff 纪律。
5. **P8（异常处理）**：AGENTS 指令冲突（B-1）、隐藏文件文档化（A-10）、规则冲突优先级（B-3）。

明确不纳入 v1.2.0（记入 DESIGN 方向）：C 类 5 项（需用户数据支撑后再定）、10 任务诊断法、工具层权限 gate。
