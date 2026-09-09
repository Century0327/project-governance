# v1.2.0 发布前验收报告

- 日期：2026-09-10
- 依据：gpt 发布前验收 5 项（不做新审计、不扩展 scope）
- 结论：**5 项验收全部通过，确认点 1 已放行**，可进入确认点 2（push / 重打 zip）

---

## 1. git diff 范围核对（对照 14 项逐条映射）

提交 `1cdfd43`（HEAD^ 为 v1.0.0 initial commit），19 文件 +1631/-278。说明：v1.2.0 提交按用户决策合并了 v1.1.0 未提交内容（README/SKILL/governance.py 中文化、DESIGN/CHANGELOG/handoff/.skillignore 新增），映射时予以区分。

| 文件 | 改动归属 | 越界？ |
|---|---|---|
| scripts/governance.py | v1.1.0 中文化 + A-2/A-3/A-4/A-9a/A-9b（新增 11 个函数全部映射） | 无 |
| templates/AGENTS.md | B-1 / B-5a / M-1（9 行） | 无 |
| templates/ARCHITECTURE.md | A-5a 示例化 + DRAFT + 中文化（23 行） | 无 |
| templates/CHANGELOG.md | A-1 规范 + 去重降为操作建议（10 行） | 无 |
| templates/LESSONS.md | A-5a 示例化 + DRAFT（含 H-2 并入）（19 行） | 无 |
| templates/PROJECT.md | A-5a 示例化 + DRAFT（15 行） | 无 |
| templates/index.md | A-2/A-3/A-7a 区块所有权说明（10 行） | 无 |
| templates/session_handoff.md | H-1 In-progress work（5 行） | 无 |
| tests/test_governance.py | A-11a：A-2 三场景 + v1.2.0 功能测试（139 行） | 无 |
| README.md / SKILL.md | v1.1.0 中文化 + A-7a/A-10a 文档化 + version→1.2.0 | 无 |
| DESIGN.md | v1.1.0 新增 + ADR-9（A-2 配套） | 无 |
| CHANGELOG.md / session_handoff.md | v1.1.0 新增 + v1.2.0 条目 | 无 |
| docs/audit-*（4 个） | 三轮审计记录（用户规则要求留存） | 无 |
| .skillignore | v1.1.0 发布包排除清单 | 无 |

CLI 核查：init/validate 无逻辑改动（仅 help 文本中文化）；index/check 参数 default 常量→None 属 A-4 参数同源，新增 `--force` 属 A-2。无新增 schema、无新增 CLI 参数类型、无无关文件。

**结论：无顺手重构、无额外 schema、无额外 CLI、无无关文件修改。**

## 2. A-2 行为变更安全性（重点：数据不丢，非仅 exit code）

`cmd_index`（governance.py:531）写入时序验证：

1. 所有拦截路径（人工内容 / UTF-8 解码失败 / 标题缺失）均在 `write_text` **之前** `return 1`，此时文件未被触碰 → 人工内容 100% 保留。
2. 正常路径在内存中完整构造 `new_text`（保留 `start` 之前头部与 `end` 之后尾部）后**单次全量写入**，无流式写、无临时拼接 → 不存在部分改写。
3. 测试（test_governance.py test_v120_features）scenario 2 断言拦截后 `"人工补充说明" in after`——直接验证数据仍在，而非仅 exit code；scenario 3 验证 `--force` 后内容被覆盖且 meta 写入。

三场景实测（冒烟）：纯机器块 exit=0 更新成功；人工内容 exit=1 停止且内容保留；`--force` exit=0 覆盖。

**结论：拦截发生在写入前，写入原子，数据不丢。**

## 3. A-9b 防噪验证（狼来了测试）

用 git 历史 v1.0.0 真实模板 + v1.1.0 发布 zip 分别构造旧项目，跑 v1.2.0 `check`：

| 项目 | warning（黄灯） | 红灯 | 说明 |
|---|---|---|---|
| v1.0.0 模板生成 | **0** | 1（index 过期） | 静态分区树非真实扫描，属真实状态问题 |
| v1.1.0 zip 生成 | **0** | 1（index 过期） | 同上 |
| 升级路径 | — | 运行 `index` 一次后 CHECK PASSED | 一次性升级动作，非持续噪声 |

三类 warning 均未误报：无 SKILL.md 的项目不触发版本一致性；旧表格格式 CHANGELOG 不触发；无 index_notes.json 不触发失效路径。无"全是黄灯"现象。

**结论：warning 判定范围精确，旧项目升级只报真实问题。**

## 4. 文档一致性核对（防漂移）

| 主题 | 涉及文件 | 结论 |
|---|---|---|
| index `--force` 语义（仅解除人工内容保护） | SKILL.md:141-143 / README:89,136,139 / CHANGELOG:10,21,25 / templates/index.md | 一致 |
| init 与 index 的 `--force` 语义区分 | README:139 明确"index 的 --force 只解除人工内容保护" | 一致 |
| Record time 由 index 每次更新（UTC） | SKILL.md:140 / CHANGELOG:11 / templates/index.md:30 | 一致 |
| CHANGELOG.md 唯一事实源，index 内嵌仅镜像 | CHANGELOG:17 / templates/CHANGELOG.md:3-4 / templates/index.md:34 | 一致 |
| A-9a Expected/Actual 差异输出 | SKILL.md:143 / README:136 / CHANGELOG A-9a / 实测输出格式 | 一致 |
| 测试数量 92 | SKILL.md:175 / CHANGELOG:24 / 实测 92/92 | 一致 |
| B-1 索引未命中（报告缺口，不盲搜） | AGENTS.md 模板 + SKILL.md:129 | 一致 |
| 版本号 1.2.0 | SKILL.md frontmatter / CHANGELOG [1.2.0] | 一致 |

**结论：v1.2.0 未制造新的表述漂移，6 组关键语义跨文档一致。**

## 5. Claude 对照检查（参考差异，只记录不整改）

对照 Anthropic《The Complete Guide to Building Skills for Claude》（33 页）核心规范：

- **必需项全部符合**：SKILL.md 精确拼写、name kebab-case、description WHAT+WHEN（708 字符 <1024）、无 XML 标签、无 claude/anthropic 保留前缀。
- **参考差异（不整改，避免开 v1.3.0 工程）**：
  1. README.md 在发布包内（Anthropic 建议不放 skill 文件夹；SkillHub 平台惯例相反，且官方认可 GitHub 分发配 repo-level README）。
  2. version/category/tags 用顶层字段而非 `metadata:` 嵌套（SkillHub 规范 vs Anthropic 建议写法，YAML 允许）。
  3. 无 `references/` 目录（以 templates/ + README 承载，语义近似）。
  4. 主体无 Examples 章节（frontmatter 有 2 例）。
  5. 无 `assets/`（可选）。

## 总结

已找到足够多的问题，且只修了值得修的那些（14 项最小充分修复）。5 项发布前验收全部通过，**scope 冻结**。下一步（确认点 2）：push 主仓库 → 重打 v1.2.0 zip（排除 .skillignore 内容），均需用户逐次确认。
