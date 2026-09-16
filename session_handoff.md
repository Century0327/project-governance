# Session Handoff — project-governance Skill 开发仓库

Updated: 2026-09-10

## 上次会话（2026-09-10）
- 查询 PR #22（trae-community/trae-skills）回复：维护者 MasamiYui 2026-09-07 回复 "Hi, please resolve the conflict"；PR open，`mergeable_state: dirty`（README 两张表格与 #23 gbr-pair / #24 cycle-delivery / #27 docx-diff-comment 冲突）。其他 9 个 PR 均已合并。
- 用户决策：先更新 skill（依据 E:\苏狐\suhu 实际使用经验），再解决 PR 冲突。
- 用户已确认：版本 v1.2.0；本地 v1.1.0 未提交内容合并为一次 v1.2.0 提交；不同步 suhu 副本；PR 推送后回复维护者；**硬约束：commit/push/PR 评论等外发动作必须逐次经用户确认，绝不未经确认直接 PR**（执行到模块 6、7 前各设确认点）。

## 本轮进展（2026-09-10）：机制漏洞横向审计已完成
- 用户采纳 GPT 建议，暂停 v1.2.0 执行，先做横向机制审计（纯只读，未修改任何 skill 代码/模板）。
- 通读全部 17 个文件（SKILL/README/governance.py/11 模板/tests/DESIGN/CHANGELOG），审计报告已落盘：
  **docs/audit-v1.1.0-mechanism-audit.md**（本文件与 docs/ 已加入 .skillignore，不进发布包）。
- **核心结论**：CHANGELOG 暴露的"规范缺失→AI 自由解释"是**系统性特征**，非孤立案例。发现 A 类 11、B 类 6、C 类 5、D 类 6；合同自由度测试 9/9 场景存在双解；现实用户模拟证明系统大量依赖"人类会主动维护"的错误假设。
- 最有力证据：①变更记录三处事实源三种结构（模板表格 vs 作者 Keep a Changelog 标题 vs index 内嵌 Change log）；②`cmd_index` 静默删除 root↔changelog 之间人工内容（suhu 停用 index 命令的直接原因）；③`{{...}}` 占位符三义混用无门禁。



## 本轮进展 2（2026-09-10）：Anti-Overfitting / Change-Budget 审查已完成
- 用户采纳 GPT 二次建议：防止"发现问题→把系统设计得更复杂"，要求输出建议修改范围（不执行修改）。
- 裁定文档：**docs/change-budget-v1.2.0.md**。8 问逐项审查 A/B/C 全部 22 项 + 8 个"修 A 坏 B"反例全部可规避。
- **v1.2.0 最终边界（用户待批准）**：
  - **必须修 7**：A-1 CHANGELOG 位置规则（模板+文档）、A-2 index 人工块保护（默认停止+告警+--force，唯一行为变更）、A-3 头部时间戳更新、A-4 index/check 参数同源（元数据+回退）、A-9a check 差异输出（不改判定）、A-10a 隐藏文件文档化、A-11a 为本轮补测试。
  - **建议修 5**：A-5a 占位符模板示例化、A-7a 变更记录唯一事实源（仅文档）、B-1 AGENTS 指令冲突措辞、B-5a 未映射目录保守处理一句规范、A-9b check 新增告警项（warning 级，不判红）。
  - **暂不修 10**（留观察）：A-6 superseded_by、A-8 VERSIONS Level 列、A-5b 门禁 error 级、A-7b 删 index 内嵌 changelog、A-10b --include-hidden、B-2 judge 统一、B-3 VERSIONS judge 冲突、C 类全部。
- 原则：规范问题优先规范修复；不为自动化而自动化；无真实歧义证据不改 schema；所有修改不改变现有行为判定（A-2 例外：防数据丢失，suhu 真实遭遇）。

## 本轮进展 3（2026-09-10）：动态行为审计已完成
- 用户采纳 GPT 三次建议：前两轮审静态规范，本轮审 **AI 实际执行时的动态行为**。
- 报告：**docs/audit-v2-dynamic-behavior.md**。5 个任务模拟（正常/重复执行/部分失败/中断恢复/错误推断）+ 8 个新维度（authority hierarchy/状态转换/语义幂等/部分失败/中断/认知状态/self-modifying/规则膨胀）。
- **核心结论**：系统最薄弱的不是"规则缺失"而是**执行轨迹不可恢复**——四条防线全缺：语义幂等（CHANGELOG 重复写无去重）、checkpoint（部分失败欠账不可恢复）、中断恢复（handoff 无"进行中任务"字段）、认知状态（AI 推断写入即获事实地位，无草案标记）。
- **对 v1.2.0 的影响（克制）**：12 项裁定不变；仅建议新增 2 项极小候选（纯模板/文档，低风险）：
  - **H-1** handoff 模板加"当前进行中任务/进度/已确认项"登记行（解决中断失明）
  - **H-2** ARCHITECTURE 模板占位符区加"草案（AI 推断未经确认）"固定标注 + AGENTS 补"推断性内容必须带草案标记"一句（解决错误自强化）
- 明确不进入：语义幂等、checkpoint、authority 跨文件文档化、self-modifying 完整机制、规则膨胀治理（观察项，等真实事故证据）。
- 待用户批准事项：**v1.2.0 范围 = 原 12 项 + H-1/H-2（共 14 项）**，批准后执行阶段 A（纯本地）。

## 本轮进展 4（2026-09-10）：审计结果校准已完成（收网）
- 用户采纳 GPT 四轮建议：不扩大问题清单，复核过度归因/过度设计。报告：**docs/audit-v3-calibration.md**。
- 5 个复核点结论：
  1. 任务 1"多路径"修正：规范**从未允许手动编辑 index**，只是未显式声明"树块由命令独占维护"→ 归因为维护方式未显式化（非两路合法）；A-2 保留并附显式化一句。
  2. 语义幂等：CHANGELOG 语义 = 决策/状态变化（非事件）→ 仅加"记录前自查去重"一句，**不做变更指纹机制**。
  3. 原子性 vs 可恢复性：真实需求是可恢复性 → **事务系统不做**，H-1 即最小方案。
  4. H-2 收窄：仅 ARCHITECTURE/PROJECT 模板占位符区加草案标注，**并入 A-5a**；通用 epistemic schema 不做。
  5. 新维度 M-1：最小变更原则——AGENTS 补"治理文件只更新任务需要+规则明确要求的，其余非必要不动"一句（识别的连锁修改风险）。
- **校准后最终边界 = 14 项**：必须修 7（A-1+A-2 各附语义句）、建议修 6（A-5a 含 H-2、A-7a、A-9b、B-1、B-5a、M-1 新）、H-1 独立项。全部模板/文档/告警级，唯一行为变更 A-2。

## 本轮进展 5（2026-09-10）：v1.2.0 实施完成，待确认点 1 放行
- 三点实施前卡位已逐项核对（代码 + 测试 + 模板）：
  1. **A-1** 去重仅保留为"操作建议"一句（templates/CHANGELOG.md 第 10 行），无新增硬性 check / schema；canonical location 与"决策/状态变化"语义已入规范。
  2. **A-2** 三场景回归位于 test_v120_features（纯机器块正常更新 / 人工内容默认停止不覆盖 / `--force` 覆盖且再次运行 no-op-safe）；`--force` 在 cmd_index 中仅出现在人工内容保护一处判断，不改变其他生成逻辑（冒烟验证 exit=1 → --force exit=0）。
  3. **A-9b** 三 warning 判定范围精确：仅检测 AUTO 占位符（非用户待填 `{{...}}`）、仅可解析版本且 CHANGELOG 用 `## [x.y.z]` 标题、仅 index_notes.json 存在且可解析时检测失效路径；旧项目缺元数据 / 未启用备注功能 / 表格格式 CHANGELOG 均不报警。
- tests **92/92 全部通过**（stdlib-only）；冒烟验证通过：init 11 文件 → index 更新 → check PASSED → 人工内容拦截 exit=1 → `--force` 放行 exit=0。
- 补记 **ADR-9**（DESIGN.md）：index 目录树区块由命令独占维护 + 人工内容保护（对应 A-2，handoff 配套项；信息生命周期/批准边界未入，属留观察）。
- **状态：已到确认点 1**。提交 / push / 重打 zip 等外发动作待用户逐次确认后执行。

## 本轮进展 6（2026-09-10）：发布前验收 5 项全部通过，确认点 1 放行
- 验收 1 git diff 范围核对：19 文件逐条映射 14 项，无顺手重构/额外 schema/额外 CLI/无关文件。
- 验收 2 A-2 安全：拦截均发生在写入前，正常路径单次全量写入；测试断言拦截后人工内容仍在（数据不丢，非仅 exit code）。
- 验收 3 防噪：v1.0.0/v1.1.0 真实旧项目跑 check 均 **0 warning** + 1 个真实红灯（index 过期），`index` 一次升级后 PASSED；无狼来了。
- 验收 4 一致性：--force 语义 / Record time UTC / CHANGELOG 唯一事实源 / A-9a 差异输出 / 测试数 92 / B-1 措辞 / 版本号 1.2.0，6 组跨文档全部一致。
- 验收 5 Claude 对照：必需项全符合；5 项参考差异（README 在包内/顶层 version/无 references/无主体 Examples/无 assets）只记录不整改。
- 报告：**docs/release-acceptance-v1.2.0.md**。scope 冻结。
- **状态：确认点 1 已放行，待确认点 2**：push 主仓库 → 重打 v1.2.0 zip。

## 本轮进展 7（2026-09-10）：确认点 2 完成——发布、冲突解决、回复维护者
- 提交：验收报告 + handoff 进展 6（8571baf）→ push 个人仓库 `Century0327/project-governance`（走代理 127.0.0.1:7897，git 直连 443 被重置）。
- 重打 **project-governance-v1.2.0-skillhub.zip**（50,319 B；排除 .skillignore 8 项 + tests/.gitignore/旧 zip；内容 7 项与 v1.1.0 一致）；删除 v1.1.0 zip。zip 不进 git（产物）。
- PR #22（trae-community/trae-skills）：维护者 MasamiYui 曾评论 "please resolve the conflict"。已在 fork `Century0327/trae-skills` 的 `add/project-governance-skill` 分支：更新 skill 至 v1.2.0（4b0d438）→ merge upstream/main 解决 README.md / README.zh-CN.md 表格冲突（c0fad60，保留 cycle-delivery/gbr-pair/docx-diff-comment 行）→ push → **mergeable: MERGEABLE**（BLOCKED 因组织分支保护需维护者 review）。
- 已回复 MasamiYui（comment-5608447805）：冲突已解决 + v1.2.0 更新说明 + 请求 re-review。
- 遗留：PR 合并需维护者批准；**下次会话先查 PR #22 是否已合并/有无新评论**。

## 本轮进展 8（2026-09-10）：PR #22 已合并，v1.2.0 正式发布
- 维护者 MasamiYui **APPROVED（LGTM）** 于 2026-09-10 02:47 UTC，随即合并 PR #22（mergedAt 02:47:38Z）。
- 已验证主仓库 trae-community/trae-skills main：skills/project-governance/SKILL.md version="1.2.0"、governance.py 604 行（与本地一致）、README 表格含 project-governance 行。
- 发布闭环完成：本地提交 → 个人仓库 push → fork 更新 → 合并 → 主仓库 v1.2.0。
- 待办：本地 handoff 进展 7+8 未提交到个人仓库（8571baf 之后）；zip 产物不进 git。

## 新任务登记（2026-09-12）：社区发言运营（用户社恐，委托代起草+代发）
- 需求：用户希望我维护论坛/GitHub/SkillHub/HuggingFace 等社区的发言与回复（社恐，降低社交负担）。
- 调研完成：**docs/community-map.md**（社区地图与优先级）。要点：S 级 forum.trae.cn + GitHub trae-community；A 级 ClawHub（clawhub.ai）+ 腾讯云 SkillHub（frontmatter 已对齐）；B 级 HuggingFace/掘金/知乎/CSDN；C 级 Reddit/V2EX/OpenClaw 中文（待验证）。
- 工作流：我起草 → 用户确认 → 发布。**可代发**：GitHub（有 gh 认证）、ClawHub/SkillHub（CLI，待授权）；**仅草稿**：需用户账号的平台（论坛/掘金/知乎等）。外发必先确认。

## 本轮进展（2026-09-12）：全盘 changelog 扫描完成，事故案例库就绪
- 用户纠正"只看部分 changelog 不够"，要求全 D 盘 + E 盘扫描。已按索引（D 根 index.md / E 苏狐 index.md）完成：
  - 读全 5 份 CHANGELOG：D 根（167 行）/ workspace/tests（2192 行，提取全部标题）/ E 苏狐（109 行）/ cdu-freshman-guide v1.4.0 / douyinwenan_upgrade v2.0.0（含 douyin-edu-copywriting）。
  - 读全 3 份错误档案：D 根 LESSONS（5 条）/ workspace LESSONS（32 条，含"历史复发"标注）/ E 苏狐 LESSONS（11 条）。
  - 确认 E 盘其余 CHANGELOG 均为游戏 mod（无关）；D 盘 auto_changelog.md 为机器维护（13 行）。
- 产出：**docs/changelog-scan-20260912.md**（问题全景 7 类 + 治理映射表 + 5 个故事性素材 + 数据源索引）。
- 增补：**fact-pack-forum-post.md 新增第 11 部分"真实事故案例库"**（案例 A-H，脱敏版，引用前需确认）。
- 核心发现：发生过的问题 7 类中 6 类是治理体系设计靶子（版本参数用错 / AI 失忆找不到文件 / 重复犯错 / 静默破坏 / 编码环境坑 / 过程纪律缺失），另有社区发布流程类 1 类。

## 本轮进展（2026-09-12）②：发帖素材勘探完成（按 GPT 六范围，未文章化）
- 产出：**docs/fact-mining-forum-post-20260912.md**（素材勘探报告）。
  - 1.项目真实痛点：14 条有"前因→事故→修正"闭环的案例池（来源=workspace 32 条 LESSONS 全文 + D 根 5 条 + 苏狐 11 条 + 社区项目 changelog）。
  - 2.治理机制映射表：10 组"事故→机制→前后变化"。
  - 3.故事性排序 5 案例（五要素齐全）：A=index 吞人工内容（治理工具自身事故，★★★★★）＞ B=对抗性测试漏 11 崩溃（"用户是邪恶的"）＞ C=blacklist 一句话不精确（治理规则源头）＞ D=handoff 膨胀 101KB ＞ E=凭文件名猜结论。备选：误覆盖长文档 / 上下文污染（留第二篇）/ AI 读图判错。
  - 4.证据盘点：文本/代码证据齐（3 审计文档/692 行核心/92 用例/11 模板/PR#22），**全盘无截图**，已标注可补位置。
  - 5.版本演化：v1.0.0（漫画管线事故沉淀）→ v1.1.0（SkillHub+中文化+HINT）→ v1.2.0（三轮审计 14 项，A-2 为真实事故驱动）。
  - 6.社区验证：PR #22 实查（2026-08-16 创建 → 09-07 维护者要求解冲突 → 09-09 作者升级 v1.2.0 → 09-10 LGTM + MERGED，commit 0931b9ad）。
- 数字验证：governance.py 恰 692 行 ✅；test_governance.py 834 行、record() 逐条计 PASS/FAIL，92 用例与 CHANGELOG/PR 一致 ✅。
- 隐私标注：涉苏狐/QQ/记忆的 4 处已标【需用户确认后公开】；漫画管线细节建议脱敏。
- 严格遵守 GPT 边界：未设计标题、未写开场、未润色、未拔高。

## 本轮进展（2026-09-12）③：诞生前史勘探完成（按 GPT A-E 结构，证据先行、未反推）
- 产出：**docs/prehistory-timeline-20260912.md**（Project Governance 诞生前史 + 时间线）。
- 突破性证据（文件时间戳）：治理体系**不是渐进生长，是 08-12 凌晨 02:15~07:20 集中建立**（PROJECT→session_handoff→AGENTS→CHANGELOG→LESSONS×2，5 小时内 6 文件）。
- 治理前（07-30~08-11）非空白：08-02 指导手册-AI阅读.md（首次"给 AI 立规矩"，含"Step5 Quality Check：不是 AI 自己觉得不错，而是检查"——check 命令思想源头）；08-09 22:14 架构.md；**08-09 23:27 测试稳定版本索引.md（whitelist/blacklist 与 check 的直接前身，"人工判断>AI判断"分离）**；08-09"项目记忆更新"条目机制。
- 治理前事故 8 案（A2 参考图用错/B5 AI 误判融合/DreamShaper 挂画失败/B8 复用旧图三重否定/B21 FAILED/架构五版并存/结论散落/C01-1 当天 8 错），全按 6 问作答。
- "第一次真正意识到需要项目治理"三候选：萌芽=08-02 指导手册；首个治理产物=08-09 稳定版本索引；**系统性起点=08-12 02:15 PROJECT.md**（推荐标注口径已写好）。
- 【时间无法确认】4 处：08-12 凌晨直接触发事件、非结构化 blacklist 首次出现、A2/B21 原始发生日——均已标注，未推测。
- git 铁证：本地 Initial commit 08-17 21:25；PR #22 北京 08-17 03:17 创建。

## 本轮进展（2026-09-12）④：skill 成型前后效果对比完成（数据版）
- 产出：**docs/skill-before-after-comparison-20260912.md**。按三期（治理前 08-02~11 / 治理体系建立 08-12 / skill 成型 08-17 起）逐文件实测计数。
- 核实修正：**"tests CHANGELOG 149 条目"原归类有误**——该文件记录 08-05~08-24 全期测试（治理前 28 条 / 治理后 121 条），skill 成型后仅占约 40%，不能当"成型后成绩单"。
- 核实数据：旧架构 CHANGELOG 67 条（08-03~14）；workspace LESSONS 30 条错误（#1-32 缺 9/10，2 条"历史复发"#14/#17）；tests CHANGELOG 149 条（28/14/107）；skill 用例 43→60→76→92；对抗性 17 用例暴露 11 缺陷→0；session_handoff 现 19,028 B（101KB 事故后）。
- 关键新发现：**cdu-freshman-guide / douyinwenan_upgrade 目录内无任何治理文件 → skill 外部采用证据 = 0**（复用层归因需悬置）。
- 结论分层：A 档工具层"明显变好"（缺陷 11→0、用例 +114%、PR LGTM）证据硬；B 档治理层机制落地为实但效果被"08-18 后测试骤停"混淆；C 档复用/体验层无证据。归因存疑 4 项已列。

## 本轮进展（2026-09-16）⑤：外部 review（DeepSeek/Kimi 提三层架构 vs GPT 提最小补丁）→ 审计 + 补丁方案完成
- 触发：GPT 反驳 DeepSeek 三层架构方案，主张"SKILL.md 148 行不超限（<500 行），真问题是'触发后默认全面治理'"，给出 P0 实测 + P1-P4 最小补丁路线。
- 核查（全部本地实测 + 官方原文）：SKILL.md 192 行/148 有效/9,185 B/v1.2.0 ✅；「日常维护（每个会话）」措辞属实 ✅；Trae 官方「动态按需加载，先扫描 description」原话 ✅；OpenAI skill-creator 三级披露/不建无意义 router 原文 ✅；Trae best-practice（误区二/三、description≤1024、评测驱动失败优先）✅。
- 补充量化：description 708 字符未超 1024 上限，但中英近乎重复（中文 360 + 英文 348）→ 触发扫描面一半浪费。
- 独立判断：**SKILL.md 的默认流比用户自己实际实践更重**（用户规则是任务触发式：找文件才查索引、任务结束才更新 changelog）——"文档比实践过度"的硬证据。
- 产出：**docs/skill-patch-plan-trigger-scope-20260916.md**（核查表 + P1-P4 补丁全文草案 + P0 触发实测 5 用例 + A/B 验证设计 + 不做清单）。
- 补丁范围：P1 第3步改"按任务需要"+"任务→文件映射表"；P2 结束更新改条件式；P3 description 去重（708→~360，保触发关键词）；P4 何时使用/不用压缩为边界。全不动 governance.py/templates/tests；版本拟 v1.2.1。

## 本轮进展（2026-09-16）⑥：GPT 二轮收紧 5 处 + P1/P2 语义审校 → 方案升为 v2 最终改稿
- GPT 批准方向，要求改前先做 P1/P2 语义审校（通用 Skill 规则 vs 项目具体规则），并提 5 处收紧：①"禁止盲目搜索"过强→"索引优先；缺失/过期可搜索并视情况更新"；②score>0.85 阈值不上浮通用层→"优先继承已验证条目"；③P3 目标=触发判别信息密度而非词覆盖率；④P0 中"不触发"不写死，"是否触发"降辅助指标、核心=触发后是否过度介入；⑤总原则="不要求少触发，要求触发后只执行与任务相关的治理动作"。
- 审校结论：第 3 步有 4 处项目具体机制上浮到通用层（固定三件套流程、注册表文件名、0.85 阈值、permanent_ban 字段名）→ SKILL.md 只留通用原则+映射表，数值/字段名权威下沉模板与 schema。佐证实例：用户自己规则"绝对禁止直接搜索文件"比通用层该有的强，正说明具体规则不该平移到通用层。
- 产出更新：**docs/skill-patch-plan-trigger-scope-20260916.md** 已升 v2 最终改稿（P1+P2 合并重写第 3 步全文、P3 新 description ~330 字符、P4 边界压缩、P0 修订表）。
- 待执行：SKILL.md 应用补丁（version→1.2.1）→ 跑 92 用例确认无影响 → CHANGELOG 记一条 → A/B 对照实测。

## 本轮进展（2026-09-16）⑦：v1.2.1 补丁已落地（SKILL.md 已改 + 92/92 通过 + CHANGELOG 已记）
- GPT 三轮收紧（P3 description 末句改触发场景；P0 加用例 F"明确治理任务只需局部能力"形成 6 用例梯度；triggers 定性为社区约定/辅助元数据——已核实 trae-skills 公开标准 frontmatter 仅 name+description）。
- **已执行**：SKILL.md 4 处补丁（version→1.2.1；description 708→279 字符；何时使用/不用压缩；第 3 步重写为"按任务需要"+映射表+条件式更新）→ tests 92/92 PASS（stdlib-only）→ CHANGELOG 记 [1.2.1]（含审校说明与 6 用例验证设计）。
- 验证后 SKILL.md：214 行，frontmatter 15 键完整（name/version/description/triggers 均在）。
- 方案文档：docs/skill-patch-plan-trigger-scope-20260916.md（v2 + 三轮修订，最终改稿）。

## 本轮进展（2026-09-16）⑧：A/B 实测就绪（对照材料 + 手册）
- GPT 批准进入 A/B 实测，不再改 SKILL.md；留下关键观察点："模型主动读了什么" vs "Skill 文本自带提及但未实际读取"（token 成本只看前者）；triggers 建议保持现状不再动。
- **已执行**：下载线上 main 1.2.0（9185 B/192 行）到 `D:\Stable Diffusion\workspace\tests\ab-v120-vs-v121\SKILL-1.2.0-线上.md`；本地 1.2.1（9308 B/214 行）另存对照 → 产出 **docs/ab-test-guide-20260916.md**（唯一判据 + 两版行为差异速览 + 6 用例 prompt + 跑法 A/B + 记录表含"主动读取 vs 上下文已有"区分 + 判定标准 + 结果去向）。
- 待用户执行：跑 6 用例（推荐方案 A：日常项目新会话顺带做，不烧积分）；结果回填 changelog（区分 AI/人工评审）。

## 本轮进展（2026-09-16）⑨：AI 侧模拟测试完成（判定通过）；人工侧改为"粗略感知"
- 用户决策：不做人工 A/B（user memory 自动读取会污染对照）；人工侧只切 1.2.1 正常使用 + 粗略感知；AI 侧自行测试。
- **AI 模拟测试**：建隔离项目 `tests/ab-v120-vs-v121/sim-project/`（11 文件），两个独立子代理分别注入 1.2.0（线上）/1.2.1（本地）SKILL.md，对 6 用例做只读行为推演（不碰用户记忆）。结果：1.2.0 全部 6 例默认"会话开始读三件套"，B/C/F 超额读 2-3 个治理文件，E 被升级成更新 3 个治理文件；1.2.1 每例收敛到映射表对应文件，F 只取 index+VERSIONS。**判定通过**；观察项 1：A 结束时仍记 CHANGELOG+handoff（P2 裁量边界）。
- 产出：**docs/ab-test-ai-sim-20260916.md**；CHANGELOG v1.2.1 测试段补 AI 模拟评审（区分 AI/人工，人工粗略感知待补）。
- 版本一致性确认：SKILL.md=1.2.1 = CHANGELOG 最新 1.2.1 ✅。
- 测试边界（诚实声明）：模拟验证的是指令遵循度，非宿主触发机制（触发=辅助指标，会话内不可观测）；人工粗略感知结果回填后形成完整评审。

## 本轮进展（2026-09-16）⑩：发布节奏定案——v1.2.1 已发自己仓库，社区主仓冻结
- GPT 定案：v1.2.1 先发自己仓库（Century0327/project-governance），**不**提 trae-community/trae-skills；等 1-2 个真实可复现问题（如"局部治理判断仍过宽""某类任务错误更新 CHANGELOG/handoff""跨平台加载行为差异"）累积成 v1.2.2/v1.3.0 再提 PR；例外=修复了影响社区用户的明确错误（非作者工作流独有）才紧急发社区。
- **已执行**：git commit `106b49c`（SKILL.md+CHANGELOG+session_handoff+3 个 patch 相关 docs，6 文件 410+/15-）→ push origin master 成功（8571baf..106b49c）。项目记忆已建 project_memory.md（发布节奏 + 关键约束）。
- 未纳入本次提交（留给后续）：6 个论坛素材 docs（prehistory/fact-mining/changelog-scan/community-map/fact-pack/skill-before-after）+ `project-governance-v1.2.0-skillhub.zip` 打包产物。

## 本轮进展（2026-09-17）⑪：README 一致性修复（用户怀疑成立，全仓库排查确认仅此一处）
- 用户怀疑"只改了个别文件"→ 成立：GPT 抓已发布 v1.2.1 源码发现 README「执行纪律」仍写旧行为（"每次会话结束写 session_handoff"与 1.2.1 冲突）。
- **全仓库关键词扫描**（盲目搜索/每次会话/会话开始/会话结束/score 0.85/permanent_ban/先查/先找/黑名单/白名单/绝对）确认：**仅 README.md 执行纪律段（75-79 行）为旧语义**；其余命中（templates/AGENTS.md 字段名、templates/LESSONS.md 教训示例、examples 示例数据、README 注册表 Schema 段）均属实现层或历史记录，按"下沉而非删除"原则保留。
- **已改**：README 执行纪律 4 条 → GPT 最小修法（索引优先含缺失可搜索 / 参数从注册表取 / 新错误入档 / 按需更新）；CHANGELOG [1.2.1] 补"修复（2026-09-17 follow-up）"小节。
- 待确认：README+CHANGELOG 变更 commit/push（自己仓库）；是否重打 v1.2.1 zip。

## 下一步（待用户拍板）
1. 用户将前史勘探报告（docs/prehistory-timeline-20260912.md）+ 素材勘探报告（docs/fact-mining-forum-post-20260912.md）+ 效果对比（docs/skill-before-after-comparison-20260912.md）转交 GPT 出最终论坛稿（叙事基线="一开始根本没想做治理"→事故→立规矩→规矩成系统→回头发觉已是 Skill）。
2. 发布前需用户确认：①隐私标注 4 处；②是否补拍证据截图；③对比稿 Q3 口径——论坛稿只讲 A 档可量化、"skill 变好用"不归因于 skill 本身防错、C 档复用证据不主动提。
3. **人工粗略感知**（进行中）：用户日常使用 1.2.1 后把感受告诉我，回填 CHANGELOG 人工评审栏；出现真实问题即记录，攒 1-2 个后规划 v1.2.2。
4. **社区主仓**：冻结，等真实使用证据后再评估 PR。
5. **README follow-up 提交**（新）：确认后 commit+push（自己仓库）+ 可选重打 v1.2.1 zip。

## 修订后的 v1.2.0 执行范围（以校准后最终边界为准）
- 最终边界见 **docs/audit-v3-calibration.md**（校准后 14 项：必须修 7 + 建议修 6 + H-1；H-2 已并入 A-5a；新增 M-1）。前两轮裁定文档（change-budget 12 项）仅作背景，凡冲突处以校准报告为准。
- 进入项要点：
  1. A-1 CHANGELOG 模板位置规则（顶部、禁文末）+ **语义定义一句**（决策/状态变化，记录前自查去重）
  2. A-2 index 人工块保护：检测 root↔changelog 间人工内容→默认报错提示，`--force` 才覆盖 + **维护方式显式化一句**（树块由命令独占维护，人工补充进 notes/自定义 section）
  3. A-3 index 头部 Record time 由命令独占更新（UTC）
  4. A-4 index/check 参数同源：index 写元数据到头部注释，check 优先读、缺失回退默认
  5. A-9a check 输出 Expected/Actual 差异清单（判定逻辑不变）
  6. A-9b check 新增 3 类 warning（未替换占位符/版本不一致/notes 失效路径），不计入失败
  7. A-5a LESSONS/ARCHITECTURE/PROJECT 模板示例化 + 删除标注 + **草案标注（含原 H-2）**
  8. A-7a 文档化"CHANGELOG.md 唯一事实源，index 内嵌块仅镜像"（不动结构）
  9. A-10a README 文档化隐藏文件排除（不加 CLI 参数）
  10. B-1 AGENTS 指令冲突措辞统一
  11. B-5a AGENTS 补"未映射目录默认保守处理"一句
  12. **M-1** AGENTS 补"最小更新原则"一句（治理文件只更新任务需要+规则明确要求的，其余非必要不动）
  13. **H-1** handoff 模板加"当前进行中任务/进度/已确认项"登记行
  14. A-11a 为上述修改补测试
- 配套：SKILL version→1.2.0；CHANGELOG 增 [1.2.0]（按新位置规则）；README 微调；DESIGN 增 ADR-9（index 所有权边界；信息生命周期/批准边界未入，留观察）。

## 执行计划（确认点不变）
- 阶段 A：✅ 代码/模板/文档/测试已全部完成（tests 92/92 通过，冒烟验证通过）→ **确认点 1 待放行**：一次 v1.2.0 提交 → push 主仓库 → 重打 v1.2.0 zip（排除 .skillignore 内容）。
- 阶段 B：**确认点 2**：fork 分支合并 main → 解决 README 表格冲突（保留全部 skill 行）→ 同步 v1.2.0 → push → PR 评论回复维护者。
- 下次会话从 docs/audit-v1.1.0-mechanism-audit.md 开始复核，按本范围执行阶段 A。

## 用户决策（2026-09-10 已确认）
1. 版本号：**v1.2.0**。
2. 提交方式：本地 v1.1.0 未提交内容与 v1.2.0 改动**合并为一次 v1.2.0 提交**。
3. suhu 副本：**不同步**（只更新主仓库源 + 发布包）。
4. PR 冲突解决并推送后：**回复维护者**（英文"已解决冲突，请重新审阅"）。
5. **⚠️ 硬约束**：提交 / push / PR 评论等所有外发动作，**必须先经用户逐次确认后执行，绝不可未经确认直接 PR**。执行到阶段 A 提交推送与阶段 B PR 更新前各设一个确认点。

## 下次会话读什么
- `docs/audit-v1.1.0-mechanism-audit.md`（审计全文）→ 本文件 → `templates/` + `scripts/governance.py` + `tests/test_governance.py`
