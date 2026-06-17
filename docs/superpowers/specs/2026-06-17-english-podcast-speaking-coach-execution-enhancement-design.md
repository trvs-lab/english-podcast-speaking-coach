# English Podcast Speaking Coach 执行增强设计

## 背景

`english-podcast-speaking-coach` 已经完成中拆：

- `SKILL.md` 作为入口、硬规则和 reference-file 路由。
- `INTERACTION-RULES.md` 承担实时教学热路径。
- `WORKSPACE-FORMAT.md`、`STATE-SCHEMAS.md`、`WRITEBACK-FORMAT.md`、`RECOVERY-RULES.md` 分别承担工作区、状态、写回和恢复规则。

最近的修复已经补回角色扮演英文台词、纠错深度、课内表达回拉和课末复习材料密度。当前主要问题不再是规则缺失，而是规则执行成本偏高：`INTERACTION-RULES.md` 已经很完整，但 agent 在真实口语课中需要同时处理 target hiding、提示阶梯、课内表达回拉、分层纠错、coverage、Speed round 和 lesson-end summary，容易在局部细节上回归。

本设计只做轻量执行增强，不重写课程流程，不改变 durable state schema。

## 已确认范围

本轮做四项增强：

1. 增加 target selection rubric，帮助 agent 稳定区分 primary core targets、secondary core targets 和 exposure-only expressions。
2. 把纠错规则明确成「完整但分层」，避免反馈重新变薄，也避免一次性解释过载。
3. 在 `INTERACTION-RULES.md` 热路径顶部增加 decision table，作为课中执行索引。
4. 增加行为回归样例，保护 target hiding、help ladder、角色扮演、纠错和写回证据等关键行为。

## 非目标

- 不增加课时模式、快速/标准/深练等节奏分支。
- 不增加暂停、提前结束或 partial lesson 协议。
- 不改变 lesson flow 顺序。
- 不改变 target hiding、mastery lifecycle、writeback transaction order 或 replay 语义。
- 不新增数据库、脚本运行时或自动化测试框架。
- 不继续拆分 `INTERACTION-RULES.md`。
- 不修改 learner workspace 下的 `english-coach/` 数据。

## 方案比较

### 方案 A：继续补规则段落

在现有相关小节中直接增加更多文字，例如在 `Lesson Setup` 增加 target selection，在 `Correction` 增加纠错细节。

优点：

- 改动小。
- 不引入新文件。

缺点：

- 热路径已经较长，继续只加段落会让规则更完整但不一定更可执行。
- agent 仍需要在 400 多行互动规则中临场寻找决策。

### 方案 B：执行索引 + 局部规则增强

在 `INTERACTION-RULES.md` 顶部增加短 decision table，并只在相关小节补最必要的规则。另增一份 maintenance-only 的 regression scenarios 文档，用于人工回归检查。

优点：

- 不改变现有架构。
- 增强可执行性，而不是简单增加规则体积。
- 回归样例能保护后续维护质量。

缺点：

- 会多一个非运行时 reference file。
- regression scenarios 仍是人工检查，不是自动测试。

### 方案 C：引入自动化测试或脚本检查

为 skill 行为创建脚本化检查或 golden transcript 评估。

优点：

- 长期可自动验证。

缺点：

- 当前项目是 Markdown skill，没有稳定执行器。
- 会把本轮轻量设计扩大成测试框架建设。
- 很多行为质量需要人工读例子判断，脚本收益有限。

## 决策

采用方案 B：执行索引 + 局部规则增强。

本轮只改 skill 文档和维护用回归样例，不改变状态文件格式，也不加入新的运行时机制。

## 设计细节

### 1. Target Selection Rubric

在 `INTERACTION-RULES.md` 的 `Lesson Setup` 中加入 rubric，辅助现有三类目标：

- `primary core targets`
- `secondary core targets`
- `exposure-only useful expressions`

Rubric 不改变「dense lessons may have 12-18 or more useful targets」这条原则，但帮助 agent 判断哪些表达值得优先主动召回。

建议标准：

- **Spoken chunk value:** 优先选择可直接说出口的 phrase、question shape、response pattern，而不是孤立词汇。
- **Communicative function:** 表达是否承担明确交际功能，例如请求、确认、拒绝、补充、转折、讲故事、表达态度。
- **Transfer potential:** 是否容易迁移到用户真实生活、工作、旅行、观点表达或故事复述。
- **Lesson anchoring:** 是否是本课材料中值得吸收的自然说法，能替代用户常见 Chinglish 或不自然表达。
- **Review and repair fit:** 是否能自然融合已有 review item 或 repair pattern。
- **Cognitive load:** 如果表达需要大量新词才能说清楚，优先降为 secondary 或 exposure-only。

分类规则：

- `primary core targets`: 命中多个标准，并且值得本课 active recall + near-transfer。
- `secondary core targets`: 有价值，但本课只需要至少一次 active-recall opportunity。
- `exposure-only useful expressions`: 有用但不适合本课练熟，或只是 coach 展示过的自然替代说法。

普通 vocabulary 仍按 Lesson Vocabulary 处理，不进入 expression mastery。

### 2. Layered Complete Correction

现有规则已经要求 full correction 不能被 brevity 覆盖。本轮把它明确为「完整但分层」：

1. 先给 natural version，让用户知道目标答案长什么样。
2. 所有被 coach 改动过、会影响意义/自然度/迁移能力的点都要解释。
3. 如果问题很多，按优先级分组，而不是列一长串散点。
4. 修复练习只挑最高价值的 1-2 个点做 near-transfer，避免让用户复制完整改写段落。

优先级建议：

- meaning and communicative intent;
- lesson chunk or reusable spoken upgrade;
- Chinglish and information order;
- tense, article, countability, preposition, word order, collocation;
- minor style polish.

这条设计服务两个目标：

- 不回到「只解释 2-3 个点，其他修改静默发生」的薄反馈。
- 不把每个小问题都展开成同等重量，压垮口语练习节奏。

### 3. Hot-Path Decision Table

在 `INTERACTION-RULES.md` 开头、`Lesson Setup` 前后增加一张短 decision table。它是执行索引，不是新流程。

建议列：

- Situation
- Coach action
- Guardrail

建议覆盖：

- learner answers naturally -> brief concrete feedback, continue role-play -> do not over-explain minor polish;
- learner is stuck or asks for help -> keyword hint, then sentence frame, then complete expression -> do not jump to answer first;
- learner has Chinglish, lesson-chunk mismatch, or information-order issue -> full layered correction + near-transfer -> do not ask for copying;
- coach role line may reveal a target -> paraphrase, switch role, use Chinese only if needed, or demote target -> do not model target before attempt;
- block ends -> compact checkpoint -> do not show raw status table;
- before lesson end -> test unresolved/unproduced targets -> do not test stable active targets;
- lesson-end summary -> four learner-facing areas -> do not mix active, review, extension, and vocabulary.

该 table 应保持短，不复制完整规则。详细规则仍留在原小节。

### 4. Regression Scenarios

新增维护用文档：

```text
skills/english-podcast-speaking-coach/REGRESSION-SCENARIOS.md
```

它不是普通 coaching session 的启动读取文件，不加入 `Session Startup` 读集。它只用于修改 skill 后的人工回归检查。

建议场景：

1. **Target leakage:** 目标表达同时像 vocabulary 和 phrase 时，prompt 只能给中文意义，不能给英文答案。
2. **Help ladder:** 用户说「不会说」时，只给 keyword hint；第二次再 sentence frame；明确要答案时才给完整表达。
3. **Role-play contract:** coach 的角色台词默认英文，但不能泄露 learner 稍后要说的 target chunk。
4. **Layered correction:** 用户一句话里有 Chinglish、时态、冠词和课内表达 mismatch；自然版本的每个关键改动都要解释，并分组。
5. **Lesson-language anchoring:** 用户表达自然但没有用本课高价值 chunk；先接受，再把本课 chunk 作为升级，并做一次 near-transfer。
6. **Free retelling correction:** 用户复述有多类问题；coach 分组解释，随后只挑 1-2 个最高价值句子修复。
7. **Coverage checkpoint:** block 结束只展示最小 reusable chunks，不展示整段 learner reply 或 raw status names。
8. **Lesson-end evidence:** 只有 unaided production 或后续 unaided near-transfer 能进入 `这节课真正说出来的表达`。
9. **Writeback evidence:** `Writeback Summary` 中 active phrase 的 evidence 必须指向具体 learner production。

每个场景只需要包含：

- setup;
- expected coach behavior;
- failure signs.

不需要提供完整 gold transcript，避免维护成本过高。

## 文件影响

预计修改：

- `skills/english-podcast-speaking-coach/INTERACTION-RULES.md`
  - 新增 hot-path decision table。
  - 增加 target selection rubric。
  - 强化 layered complete correction wording。

- `skills/english-podcast-speaking-coach/REGRESSION-SCENARIOS.md`
  - 新增维护用人工回归样例。

可选修改：

- `skills/english-podcast-speaking-coach/SKILL.md`
  - 只在 `Reference Files` 或 `Common Mistakes` 中加入极短维护提示时修改。
  - 不把 `REGRESSION-SCENARIOS.md` 加入 ordinary session startup read-set。

不修改：

- `skills/english-podcast-speaking-coach/STATE-SCHEMAS.md`
- `skills/english-podcast-speaking-coach/WORKSPACE-FORMAT.md`
- `skills/english-podcast-speaking-coach/WRITEBACK-FORMAT.md`
- `skills/english-podcast-speaking-coach/RECOVERY-RULES.md`
- `english-coach/` learner workspace files

## 验证标准

静态检查：

- `INTERACTION-RULES.md` 包含 hot-path decision table。
- `INTERACTION-RULES.md` 包含 target selection rubric。
- `INTERACTION-RULES.md` 包含 layered complete correction 的优先级或分组规则。
- `REGRESSION-SCENARIOS.md` 存在，并明确是 maintenance-only。
- `REGRESSION-SCENARIOS.md` 不被加入 ordinary coaching startup read-set。

人工检查：

- 新规则不引入课时模式或暂停协议。
- 新规则不改变 target hiding。
- 新规则不改变 mastery lifecycle。
- 新规则不改变 writeback transaction order。
- regression scenarios 能覆盖最近出现过的教学回归类型：角色扮演退化成中文叙述、纠错变薄、课内表达回拉不足、课末总结变瘦。

## 风险与缓解

### 风险：新增 table 变成又一套重复规则

缓解：table 只写触发条件、动作和 guardrail，不复制完整解释。

### 风险：rubric 让 lesson setup 变慢

缓解：rubric 是选择辅助，不要求逐项打分；agent 只需用它判断 primary、secondary 和 exposure-only。

### 风险：纠错规则重新变长

缓解：明确「解释完整」和「修复练习只选 1-2 个最高价值点」同时成立。

### 风险：regression scenarios 被误读为普通课程输入

缓解：文件顶部写明 maintenance-only，并且不加入 startup read-set。

## 自检

- Placeholder scan: no unresolved placeholder markers remain.
- Scope check: the design excludes lesson modes and early-stop/partial-lesson protocol as requested.
- Consistency check: the design keeps existing lesson flow, state schema, writeback order, and recovery rules unchanged.
- Ambiguity check: regression scenarios are maintenance-only and must not be loaded during ordinary coaching startup.
