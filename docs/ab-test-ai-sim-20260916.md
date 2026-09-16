# v1.2.1 AI 侧模拟测试结论（2026-09-16）

> 用户决策：不做人工 A/B（user memory 自动读取会污染对照），人工侧只"切 1.2.1 正常使用 + 粗略感知"；AI 侧由 Trae 做隔离模拟测试。本文件 = AI 模拟测试记录与结论。
> 方法：在隔离项目 `tests/ab-v120-vs-v121/sim-project/`（含全套治理文件 + 业务文件）上，分别注入 1.2.0（线上 main）与 1.2.1（本地补丁）SKILL.md 指令，由独立子代理对同一 6 用例做行为推演（只读，不碰用户记忆，不污染真实项目）。

---

## 一、判定标准（沿用手册唯一判据）

> 同一任务下，1.2.1 是否比 1.2.0 少做了不必要的治理动作，同时没有漏掉真正需要的治理信息？

## 二、6 用例 × 两版 行为对比

| 用例 | 1.2.0（线上）读取 | 1.2.1（补丁）读取 | 1.2.0 结束更新 | 1.2.1 结束更新 |
|---|---|---|---|---|
| A 修 bug | index+handoff+LESSONS+bug.py | index+bug.py | handoff+CHANGELOG | CHANGELOG+handoff |
| B 上次做到哪 | index+handoff+LESSONS+VERSIONS | **仅 handoff** | 不更新 | 不更新 |
| C 哪个稳定版 | index+handoff+LESSONS+VERSIONS | **仅 VERSIONS** | 不更新 | 不更新 |
| D 初始化 | 盘点全套治理文件 | **init 生成全套** | 不更新 | 生成全套（正确） |
| E 写 README | index+handoff+LESSONS+README | index+README | **index+CHANGELOG+handoff** | 不更新 |
| F 稳定版配置文件 | index+handoff+LESSONS+VERSIONS+v2.json | **index+VERSIONS+v2.json** | 不更新 | 不更新 |

## 三、结论

- **读取层面**：1.2.0 全部 6 例默认触发"会话开始读三件套"，B/C/F 明显超额（多读 2-3 个治理文件）；1.2.1 每例收敛到映射表对应文件（±任务目标文件本身，如 bug.py/v2.json 属业务文件非治理文件，合理）。
- **写入层面**：1.2.0 的 E（写 README）被升级成更新 3 个治理文件——正是补丁要修的"普通任务升级成完整治理流程"；1.2.1 无此行为。
- **核心场景 F**（明确治理任务但只需局部能力）：1.2.0 连带读 handoff/LESSONS，1.2.1 只取 index+VERSIONS ——补丁目标达成。
- **判定：通过**。唯一观察项：1.2.1 的 A 结束时仍记 CHANGELOG+handoff，属 P2"条件式更新"的裁量边界（修 bug 是否算项目级决策/是否跨会话继续），不构成失败，列为下一补丁候选观察点。

## 四、测试边界（诚实声明）

1. 本测试验证的是**指令遵循度**（给定 SKILL.md 内容，模型是否按"按任务需要"执行），不是宿主触发机制（description 扫描发生在加载前，会话内不可观测）。
2. 子代理为隔离会话，未读取任何用户记忆文件，未改动真实项目；模拟环境 schema 校验未全过（whitelist 顶层数组格式为我手工构造时的偏差），不影响行为推演。
3. 人工粗略感知结果（用户日常使用 1.2.1）待补充，回填后形成完整 AI+人工评审。

## 五、产物

- 模拟环境：`tests/ab-v120-vs-v121/sim-project/`（11 文件，可复用）
- 对照版本：`tests/ab-v120-vs-v121/SKILL-1.2.0-线上.md` / `SKILL-1.2.1-本地.md`
- 手册：`docs/ab-test-guide-20260916.md`

*生成：2026-09-16（会话 6aa1b00982a06e8a967273ec）。子代理 ID：31b3c4dd（1.2.1）、47932bae（1.2.0）。*
