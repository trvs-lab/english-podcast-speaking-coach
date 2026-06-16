# English Podcast Speaking Coach 互动措辞优化计划

> 本计划只覆盖 `english-podcast-speaking-coach` 的用户可见措辞和普通会话节奏。状态语义、active recall、target hiding、writeback、replay、privacy 等行为规则保持不变。

## 背景

当前 `english-podcast-speaking-coach` 已经具备可用的口语训练流程：能从播客或对话材料中抽取核心表达，隐藏目标英文，要求学习者先主动表达，再按证据写入学习状态。

现阶段主要问题不在教学机制，而在用户可见层。部分回复容易呈现固定模板感，例如反复出现「很好，意思清楚」「更自然一点可以说」「小点：」「我继续：」「Block 覆盖情况」等句式。流程控制强、标签密度高，会让学习者感觉像在被一个自动化系统处理，而不是被口语教练带练。

## 目标

- 降低普通练习中的 AI 味和流程感。
- 保留当前 active recall 与证据化写回规则。
- 让反馈更像真人口语教练：短、具体、顺手接话。
- 减少用户可见标签，把内部流程名留在内部。
- 保留必要结构，避免回复变成大段散文。
- 为后续轻拆 `SKILL.md` 留出清晰边界。

## 非目标

- 不重新设计课程流程。
- 不修改 mastery lifecycle。
- 不放宽 target hiding。
- 不修改 lesson-end writeback 语义。
- 不新增脚本或自动化测试框架。
- 不把所有结构化反馈删掉。

## 诊断结论

### 固定模板过强

`Learner-Facing Templates` 当前提供了较完整的固定形状。它能保证稳定教学，但也会诱导 agent 在多数轮次使用同一语气。

需要从「唯一模板」改为「反馈模式 + 句式变体」。保留教学动作，但允许自然改写。

### 流程名外露过多

`guided preview`、`active-recall blocks`、`Speed Round`、`Block 2 覆盖情况`、`Active Phrase Bank` 等名称适合内部控制，不适合频繁出现在学习者界面。

用户侧应使用更自然的转场，例如「这一轮先收住」「最后补两句」「还差两个表达没真正说出来」。

### 肯定语不够具体

当前肯定语偏泛，如「很好」「意思清楚」。更自然的反馈应指出有效点或主要问题，例如「这句能用，主要改搭配」「意思到了，语气可以软一点」。

### `teacher-style` 约束不够明确

`Guided Preview` 要求使用 teacher-style paragraph，但没有明确用户可见语气的边界。需要新增 `Learner-Facing Voice`，说明哪些表达应该减少，哪些表达应该优先。

### 主文件过重

当前 `SKILL.md` 同时包含实时教学、状态格式、写回、恢复和迁移规则。普通会话容易被状态治理语言影响，输出偏行政化。此前轻拆设计仍然适用，但建议在措辞优化之后单独执行。

## 推荐方案

采用两步改造。

第一步只优化用户可见措辞。新增 `Learner-Facing Voice`，改写反馈模板、覆盖展示、转场和课末总结说明。该步骤风险低，不触碰状态语义。

第二步再执行轻拆。把 durable state schema、writeback、replay、conflict、migration 等低频规则移动到 reference files。主文件保留实时教学规则、用户可见语气、target hiding 和按需读取入口。

不建议一次性重写课程流程。当前教学机制已经稳定，主要收益来自表达层收敛和主文件减负。

## 文件范围

- 修改：`skills/english-podcast-speaking-coach/SKILL.md`
  - 新增 `Learner-Facing Voice`。
  - 调整 `Guided Preview`。
  - 调整 `Learner-Facing Templates`。
  - 调整 `Coverage Tracking` 的用户可见展示。
  - 调整 `Speed Round` 的用户可见名称和转场。
  - 调整 `Lesson-End Expression Areas` 的用户可见标题和说明。
  - 补充 `Common Mistakes` 中关于 AI 味的反例。
- 可选后续修改：新增 reference files
  - `WORKSPACE-FORMAT.md`
  - `STATE-SCHEMAS.md`
  - `WRITEBACK-FORMAT.md`
  - `RECOVERY-RULES.md`

## 实施任务

### 任务 1：新增用户可见语气规则

在 `SKILL.md` 的实时教学部分新增 `Learner-Facing Voice`。

规则应覆盖：

- 不自称 AI。
- 不解释内部流程名。
- 不每轮都使用相同开头。
- 少用泛夸奖，多指出具体有效点。
- 反馈先短后深，只有真实问题才展开。
- 允许中文自然接话，但保持目标英文隐藏。
- 用户侧少展示状态名，内部仍正常维护状态。

验收：

- `SKILL.md` 中存在 `## Learner-Facing Voice`。
- 该节明确区分「内部控制」和「用户可见表达」。
- 不改变 target hiding 或 mastery lifecycle 规则。

### 任务 2：把固定模板改成反馈模式

重写 `Learner-Facing Templates`。

从固定文本：

```text
很好，意思清楚。更自然一点可以说：
```

改为变体规则，例如：

```text
这句能用。口语里可以顺一点：
意思到了，主要改一个搭配：
这次问题不大，换个更自然的说法：
```

从固定标签：

```text
小点：
```

改为按需使用：

```text
这里主要改两处：
注意一个搭配：
语气上软一点：
```

验收：

- 模板不再只有唯一推荐开头。
- 仍要求解释关键修改。
- 仍禁止要求学习者照抄刚揭示的答案。

### 任务 3：降低流程名外露

调整 guided preview、coverage、Speed round 和课末总结的用户可见文案。

内部仍可使用：

- `guided preview`
- `active-recall blocks`
- `coverage`
- `Speed round`
- `Active Phrase Bank`

用户侧优先使用：

- 「先进入一个真实情境」
- 「这一轮先收住」
- 「还差两个表达」
- 「最后补两句」
- 「这节课真正说出来的表达」

验收：

- 普通练习中不频繁暴露 `Block`、`coverage`、`Speed Round` 等流程词。
- 内部覆盖状态仍能显示，但展示更像课堂小结。

### 任务 4：重写覆盖展示

保留覆盖信息，但减少表格感。

当前风格：

```text
Block 2 覆盖情况

let me double-check                         ✅ active
be booked solid                             ✅ active
How does next Monday work for you?          🔁 needs_review
```

推荐风格：

```text
这一轮先收住。

已经能自己说出来：
- let me double-check
- be booked solid

还要再练一下：
- How does next Monday work for you?（换时间时用）
```

验收：

- 用户能看懂进展。
- 状态名不压过学习内容。
- 仍只展示可复用 chunk，不展示长句或整段复述。

### 任务 5：调整课末四区标题

保留四类信息，但标题更贴近学习者。

建议映射：

- `主动表达库 (Active Phrase Bank)` -> `这节课真正说出来的表达`
- `待复习核心表达、已练拓展表达与修复目标` -> `下次还要再练的表达`
- `拓展表达 (Extension Expressions)` -> `听过但还没练熟的表达`
- `本课词汇 (Lesson Vocabulary)` -> `这节课碰到的词`

验收：

- 用户可见标题更自然。
- 内部写回仍能区分 phrase bank、review queue、extension expressions、lesson vocabulary。
- 未练过的表达不会进入 active 状态。

### 任务 6：补充反例清单

在 `Common Mistakes` 增加 AI 味相关反例。

建议增加：

- 每轮都用同一句夸奖开头。
- 把内部流程名直接展示给学习者。
- 在轻微问题上展开过长解释。
- 用状态表代替自然课堂小结。
- 课末总结像数据库导出，而不是复习材料。

验收：

- `Common Mistakes` 能提醒 agent 避免模板化输出。
- 反例不改变状态规则。

### 任务 7：轻拆主文件

该任务作为第二阶段执行。先完成用户可见措辞，再处理结构拆分。

建议拆分：

- `WORKSPACE-FORMAT.md`：workspace discovery、workspace files、startup reading set。
- `STATE-SCHEMAS.md`：review queue、phrase bank、repair bank、mastery lifecycle。
- `WRITEBACK-FORMAT.md`：lesson-end writeback、writeback summary、current summary、learning records。
- `RECOVERY-RULES.md`：replay、conflict、privacy、migration。

验收：

- 普通 coaching session 主要读取 `SKILL.md`。
- 写回、恢复、迁移、状态冲突时再读取对应 reference file。
- 课中教学规则仍保留在主文件。

## 验证方式

### 文本检查

运行：

```bash
rg -n "Learner-Facing Voice|这节课真正说出来的表达|下次还要再练的表达|听过但还没练熟的表达" skills/english-podcast-speaking-coach/SKILL.md
```

预期：

- 能找到新增语气规则。
- 能找到新的用户可见课末标题。

运行：

```bash
rg -n "Block [0-9]+ 覆盖情况|Speed Round" skills/english-podcast-speaking-coach/SKILL.md
```

预期：

- 如果仍出现，应说明是内部术语或反例，不应作为默认用户可见文案。

### 场景检查

使用一小段对话课材料，手动检查以下路径：

- 学习者答得基本自然。
- 学习者出现语法或搭配错误。
- 学习者请求提示。
- 学习者在 near-transfer 中修复成功。
- 课末输出四类复习材料。

检查重点：

- 回复是否有重复模板感。
- 目标英文是否仍然隐藏到尝试之后。
- 纠错是否解释关键修改。
- 覆盖展示是否自然、不像状态表。
- 课末总结是否保留状态区分。

## 风险与处理

### 风险：结构减少后反馈变散

处理：保留「轻反馈」「问题触发反馈」「near-transfer」三类模式，只减少固定话术。

### 风险：用户侧少展示状态后，agent 追踪变弱

处理：只改变可见文本，不改变内部 coverage tracking。

### 风险：改写模板时误放宽 target hiding

处理：所有示例必须继续隐藏目标英文。凡是 learner attempt 前出现目标英文的例子都不合格。

### 风险：轻拆后漏读 reference file

处理：第二阶段再拆，并在主文件的相关流程处保留明确读取条件。

## 建议执行顺序

1. 新增 `Learner-Facing Voice`。
2. 重写 `Learner-Facing Templates`。
3. 调整 guided preview、coverage 和 Speed round 用户可见文案。
4. 调整课末四区标题。
5. 补充 AI 味反例。
6. 做一轮手动场景检查。
7. 另起一轮执行轻拆。

## 决策

先执行用户可见措辞优化，再执行主文件轻拆。

这个顺序能把风险控制在最低：第一阶段只改变说法，不改变学习状态；第二阶段再降低主文件上下文负担，减少状态规则对普通会话语气的干扰。
