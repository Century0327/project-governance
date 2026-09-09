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
