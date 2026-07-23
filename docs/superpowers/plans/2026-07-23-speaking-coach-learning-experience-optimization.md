# Speaking Coach Learning Experience Optimization Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** 保留当前 skill 的快速开课、广泛覆盖、自然表达接纳和轻量记录，同时让每节课拥有清楚的学习块边界、连贯场景、可靠的掌握证据和不过载的结束体验。

**Architecture:** 只调整两个现有职责边界：`SKILL.md` 用简短的正向行为合同控制课次范围、学习块连贯性、整合降阶和证据措辞；`references/evaluation.md` 把 EnglishPod 第 25 课暴露的问题固定成可重复的轨迹回归。完整课文仍保留 8–15 个候选表达，并在同一次课程的 1–3 个学习块中按核心深练、次要轻练和场景词汇整理完成覆盖。

**Tech Stack:** Markdown skill 文档、Git、课堂轨迹回归、fresh-context agent 测试、`rg`、`wc`、`git diff --check`。

## Global Constraints

- 优化前基线已经提交为 `3b4a14b`（`docs: 保存口语教练优化基线`）；后续提交保持独立，不 amend 该提交。
- 保持 skill 名称和目录 `english-podcast-speaking-coach` 不变。
- 保留整课约 8–15 个高价值候选、核心表达通常 6–10 个、单块 3–5 个表达的覆盖策略。
- “3–5 个”始终表示一个学习块，不表示整篇课文的总学习数量。
- 同一篇材料默认在一次课程内完成 1–3 个学习块；核心表达执行完整教学循环，次要表达通过一次主动尝试、对比或整合任务覆盖。
- 学习用途和迁移场景继续由 agent 根据当前请求、档案、历史和课文判断。
- 普通开放请求以本次覆盖整篇高价值内容为目标；明确希望尽可能掌握整篇时，提高次要表达的主动练习深度。
- 新增规则使用正向输出合同和可观察条件，不增加版本称谓、模式选择菜单或大段负向约束。
- 保留现有 `PROFILE.md`、`PHRASES.md`、`REVIEW.md` 和 `lessons/` 持久化结构，本次不修改 `references/persistence.md`。
- `history/test/1.md`、`history/good-case/1.md` 和 `history/bad-case/` 作为只读回归证据。
- 维护文件保持简洁：`SKILL.md` 目标不超过 180 行，`evaluation.md` 目标不超过 180 行。

## Acceptance Criteria

- 普通“学这一课”请求默认在本次完成 1–3 个学习块，覆盖整篇 8–15 个候选；核心表达完整训练，次要表达至少经过一次主动尝试、对比选择或整合使用。
- “尽可能掌握整篇课文”请求使用相同的 1–3 块结构，在没有负担信号时为更多次要表达安排主动提取和迁移。
- 一个学习块围绕一个可以持续发展的现实任务，迁移改变人物、约束、信息或语气，而不是每个表达重置一个无关世界。
- 学习者在整合任务中回答“不知道”或无法开始时，教练把任务缩为一个有支架句子；成功后进入本块小结。
- 固定场景组合成功后，才进入修改真实细节；自由个人表达是状态良好时的延伸，不是课末必须通过的关卡。
- 即时支架或紧邻纠错后的成功称为“本次修复成功”；只有无提示新情境证据才进入“已经能自己说出来”。
- 功能相近的表达在块内整合时至少经过一次不指定答案的语境选择。
- 课末小结为主动表达提供中文用途和具体回忆提示，并区分主动掌握、本课覆盖、继续练习和场景词汇；剩余内容只在中途结束时出现。
- 第一题启动速度、自然替代表达接纳、延迟提取和四文件轻量记录不回归。

---

### Task 1: 固化 EnglishPod 25 的 RED 回归合同

**Files:**
- Modify: `skills/english-podcast-speaking-coach/references/evaluation.md`
- Read: `history/test/1.md`
- Read: `history/good-case/1.md`

**Interfaces:**
- Consumes: 已提交的真实测试轨迹 `history/test/1.md`。
- Produces: 后续修改必须通过的块边界、负担、证据和小结行为合同。

- [ ] **Step 1: 重新确认基线失败证据**

  按以下位置检查当前轨迹，并把它们视为已经观察到的 RED：

  - `history/test/1.md:238`：第一次检查点前已经引入超过 5 个主要表达。
  - `history/test/1.md:266-535`：第二段连续切换多个无关任务世界，没有新的块检查点。
  - `history/test/1.md:581`：自由整合得到“我不知道”。
  - `history/test/1.md:583-660`：降阶成功后继续扩展多轮练习，没有收束。
  - `history/test/1.md:687`：紧邻提示后的成功被称为“已经掌握”。
  - `history/test/1.md:689-700`：课末清单缺少中文用途和回忆提示。
  - 整条轨迹没有让学习者在 `backup plan / contingency plan`、`important / critical` 等相近表达中自主选择。

- [ ] **Step 2: 在评估方法后加入固定回归样本**

  加入以下 Markdown：

  ```markdown
  ## 固定回归样本

  `history/test/1.md` 是 EnglishPod 第 25 课的体验回归样本，重点检查：

  - 单块是否在第 3–5 个主要新表达后收束；
  - 块内表达是否共同服务于一个持续发展的现实任务；
  - 普通开放请求是否在一次课程的 1–3 个学习块中覆盖整篇候选；
  - 整合任务无法启动时，是否在一次降阶成功后结束本块；
  - 辅助成功、课内主动掌握和跨课稳定掌握的反馈措辞是否一致；
  - 相近表达是否经过一次不指定答案的语境选择；
  - 课末是否提供中文用途、回忆提示和整篇候选状态，并只在中途结束时列出剩余内容。

  `history/good-case/1.md` 只用于参考块级主线、聚焦修复和回忆提示，不作为启动开销、文件数量或整课长度的金标准。
  ```

- [ ] **Step 3: 收紧现有场景 1、5、6、11 和轨迹审查清单**

  对现有行为合同做以下精确调整：

  - 场景 1 的失败条件加入：“开始第 6 个主要新表达前仍没有块检查点；或所谓同一学习块只是多个无关日常场景的集合。”
  - 场景 5 的预期加入：“反馈语言与证据状态一致，辅助完成使用‘本次完成/修复成功’，不使用‘已经掌握’。”
  - 场景 6 的输入加入：“学习者在整合任务中回答‘不知道’或无法开始。”
  - 场景 6 的预期加入：“缩为一个有支架句子，成功后进入本块小结。”
  - 场景 11 的失败条件加入：“把紧邻答案且仍含提示词的正确组合称为主动掌握。”
  - 审查清单加入四个问题：普通请求完成了几个块、块内是否维持同一任务主线、整合无法启动后何时收束、掌握措辞是否与证据一致。

- [ ] **Step 4: 验证评估文件仍然聚焦行为**

  Run:

  ```bash
  wc -l skills/english-podcast-speaking-coach/references/evaluation.md
  rg -n "固定回归样本|第 6 个|不知道|修复成功|任务主线" skills/english-podcast-speaking-coach/references/evaluation.md
  git diff --check
  ```

  Expected:

  - `evaluation.md` 不超过 180 行。
  - 五类新增判断都能被搜索到。
  - `git diff --check` 无输出。

- [ ] **Step 5: 提交回归合同**

  ```bash
  git add skills/english-podcast-speaking-coach/references/evaluation.md
  git commit -m "test: 固化口语课堂体验回归"
  ```

---

### Task 2: 建立单次覆盖整篇的学习块结构

**Files:**
- Modify: `skills/english-podcast-speaking-coach/SKILL.md`
- Test: `skills/english-podcast-speaking-coach/references/evaluation.md`

**Interfaces:**
- Consumes: 整课候选清单、3–5 项学习块和主要学习用途。
- Produces: agent 可直接执行的 1–3 块单次覆盖结构与块级任务主线。

- [ ] **Step 1: 把“连贯场景”改成可观察的任务合同**

  在“选择和分组表达”与“每个学习块”中合并重复描述，保留以下核心文字：

  ```markdown
  - 每个学习块包含 3–5 个语义相关、能共同完成一个现实任务的表达。
  - 一个学习块维持同一任务主线；迁移主要改变人物、关系、约束、信息或语气，并让新表达继续服务于当前任务。

  用一个可以持续发展的现实任务练习 3–5 个表达，每轮推进一个主要新表达。块内近迁移保留任务主线，块末用一个短整合任务完成这一小段经历。
  ```

- [ ] **Step 2: 用分层深度在一次课程中覆盖整篇**

  把普通开放请求的自动续块规则替换为：

  ```markdown
  学习者只提出“学这一课”等开放请求时，以本次覆盖整篇高价值内容为目标，根据候选数量组织 1–3 个学习块。核心表达完成主动提取和新场景迁移；次要表达通过一次主动尝试、对比选择或块末整合完成覆盖；场景词汇在理解和小结中整理。

  学习者明确希望尽可能掌握整篇课文时，使用相同的 1–3 块结构；没有出现学习负担信号时，为更多次要表达安排主动提取和迁移，而不是增加无边界的连续练习。
  ```

  该规则让 agent 根据材料密度和学习证据决定使用 1、2 或 3 个块，不向学习者展示课程模式选择。

- [ ] **Step 3: 保持整篇覆盖与单次负担分离**

  确认“选择和分组表达”仍保留：

  - 整篇材料约 8–15 个高价值候选；
  - 核心表达通常 6–10 个；
  - 次要表达通常 3–6 个；
  - 每个表达候选在本次小结中归入“已经能自己说出来”“本课已经覆盖”或“还要再练一下”；
  - 场景词汇在表达候选之外单独整理；
  - 只有明确暂停或出现负担信号时，未覆盖候选才进入“下次从这里继续”。

- [ ] **Step 4: 运行两个课次边界微测试**

  使用 fresh context agent 各运行 5 次：

  **场景 A**

  ```text
  学习者说：“帮我学这一课，我想在日常生活中灵活表达。”
  课文可提取 15 个高价值表达。请只给出内部课程分块、第一节课何时收束，以及判断依据。
  ```

  Expected:

  - 每块 3–5 项；
  - 第一节课根据密度完成 1–3 块；
  - 15 个表达候选全部获得核心深练或次要轻练，并在小结中拥有明确状态；
  - 场景词汇不冒充主动表达候选；
  - 不要求学习者选择日常、职场、旅行或考试模式。

  **场景 B**

  ```text
  学习者说：“我希望尽可能掌握整篇课文。”
  课文可提取 15 个高价值表达，前两块回答稳定，仍有 4 个次要表达没有覆盖。请说明第二块结束后的下一步。
  ```

  Expected:

  - 第二块检查点后进入第三个学习块；
  - 不把整课缩减成 3–5 项；
  - 第三块优先让剩余次要表达获得主动尝试、对比或整合使用；
  - 继续时仍一次只引入一个主要新表达。

- [ ] **Step 5: 提交课次与场景调整**

  ```bash
  git add skills/english-podcast-speaking-coach/SKILL.md
  git commit -m "refactor: 优化口语课程节奏"
  ```

---

### Task 3: 优化整合降阶、证据措辞和课末小结

**Files:**
- Modify: `skills/english-podcast-speaking-coach/SKILL.md`
- Test: `skills/english-podcast-speaking-coach/references/evaluation.md`

**Interfaces:**
- Consumes: 援助阶梯、整合输出、`learning`/`repaired`/`active`/`stable` 证据状态。
- Produces: 可预测的困难收束流程和对学习者有复习价值的小结。

- [ ] **Step 1: 把无法启动整合定义为可观察负担信号**

  在“每个学习块”的负担处理后加入：

  ```markdown
  学习者在整合任务中明确表示“不知道”或无法开始时，把任务缩为一个有关键词或短句框架的句子；完成这次短成功后进入本块小结，剩余内容留到下次。
  ```

- [ ] **Step 2: 为整合输出加入轻量递进**

  将整合任务的入口写成：

  ```markdown
  完成至少一个学习块后，先用同一任务主线中的短角色对话、真实计划或简短复述组合两个以上已学表达。学习者轻松完成后，可以让其修改一个真实细节；自由选择个人场景用于状态良好时的延伸。
  ```

  这形成“固定场景组合 → 修改一个真实细节 → 自由个人表达”的递进，同时允许课程在第一层完成后正常结束。

- [ ] **Step 3: 让反馈措辞直接反映证据状态**

  在“纠错”或“小结”中加入以下正向合同：

  ```markdown
  反馈措辞与证据状态一致：关键词、句框或紧邻答案后的成功称为“这次已经完成”或“本次修复成功”；无提示新情境提取后称为“已经能自己说出来”；跨课间隔成功后才称为“已经稳定掌握”。
  ```

- [ ] **Step 4: 把课末小结改成可直接复习的固定形状**

  将课末小结写成以下固定形状：

  ```markdown
  - `已经能自己说出来`：使用“`表达` — 中文用途；回忆提示：一个具体中文场景”的紧凑形状。
  - `本课已经覆盖`：列出经过一次主动尝试、对比选择或整合使用，但尚未形成无提示新情境证据的次要表达。
  - `还要再练一下`：只列出实际得到帮助、出现反复问题或等待延迟提取的表达，并附中文回忆提示。
  - `这节课碰到的词`：整理帮助理解场景的词汇，不与主动表达候选混合。
  ```

  只有明确暂停或出现负担信号、确实留下未覆盖候选时，才显示“下次从这里继续”。

- [ ] **Step 5: 运行三个证据与收束微测试**

  每个场景使用 fresh context agent 运行 5 次：

  **场景 C：整合无法启动**

  ```text
  学习者已经完成一个学习块。面对三句自由个人表达时回答“我不知道”。教练下一步怎么做？
  ```

  Expected: 一个有支架句子；学习者成功后小结；不重新展开多个延迟提取任务。

  **场景 D：紧邻修复**

  ```text
  学习者刚说 “in case the internet offline”。教练给出 goes offline；下一题又提供场景词 “the power goes out”，学习者正确组合完整句。请给反馈并更新状态。
  ```

  Expected: 使用“修复成功”措辞；状态仍是辅助完成或 `repaired`；不称为已经掌握。

  **场景 E：相近表达选择**

  ```text
  学习者已经练过正式的 contingency plan 和日常的 backup plan。现在朋友间讨论下雨后的聚会安排。请设计块末检查。
  ```

  Expected: 提供关系和目的但不指定目标英文；接受自然选择；只解释一个语气差异。

- [ ] **Step 6: 提交证据与小结调整**

  ```bash
  git add skills/english-podcast-speaking-coach/SKILL.md
  git commit -m "refactor: 校准口语掌握证据"
  ```

---

### Task 4: 运行 EnglishPod 25 的 GREEN 端到端回归

**Files:**
- Verify: `skills/english-podcast-speaking-coach/SKILL.md`
- Verify: `skills/english-podcast-speaking-coach/references/evaluation.md`
- Compare: `history/test/1.md`
- Compare: `history/good-case/1.md`

**Interfaces:**
- Consumes: Task 1–3 的行为合同。
- Produces: 新 skill 在真实密集课文上的通过证据。

- [ ] **Step 1: 用相同请求生成 fresh-context 课堂轨迹**

  使用原始材料 `/Users/wxg/Desktop/Resources/播客/EnglishPod/主持人对话 AI 转 txt/englishpod_0025.txt` 和相同请求：

  ```text
  帮我学习这个内容，我想要学会，并在日常生活中灵活表达。
  ```

  用基线中的候选清单核对完整读取结果：`come up with a clear plan`、`backup/contingency plan`、`figure out`、`have an impact on`、`leave ... to someone`、`head up`、`forecast`、`critical`、`flu outbreak`、`avoid getting infected`、`It’s come to my attention that`、`outline specific actions`、`draft`、`move on to`、`maintain essential services`。测试要求在同一次课程中完成分层覆盖，但不要求每个候选都执行完整的双场景迁移。

  测试 agent 只加载修改后的 `english-podcast-speaking-coach`，不加载 `learning-coach`。原始输出临时保存到 `/tmp/englishpod-0025-speaking-coach-green.md`，不覆盖现有历史记录。

- [ ] **Step 2: 按固定回归样本逐项评分**

  必须同时满足：

  - 第一条实际教学消息是一道单一口语题；
  - 单块包含 3–5 个主要表达；
  - 块内有一条持续发展的现实任务；
  - 每块结束都有检查点；
  - 普通请求在 1–3 块中完成整篇分层覆盖；
  - “不知道”只触发一次降阶和本块小结；
  - 辅助成功没有被称为掌握；
  - 至少一个相近表达对经过无指定答案选择；
  - 小结包含中文用途、回忆提示，并为整篇表达候选给出明确状态；
  - 学习记录仍在课末后台按四种职责写回。

- [ ] **Step 3: 对比 good-case 中值得保留的体验**

  只比较以下三项：

  - 是否像 `history/good-case/1.md:315-328` 一样有清楚的块边界；
  - 是否像 `history/good-case/1.md:544-617` 一样由整合输出进入聚焦修复；
  - 是否像 `history/good-case/1.md:646-679` 一样提供可复习的小结。

  同时确认没有恢复 good-case 的课前大量文件操作和额外“继续”轮次。

- [ ] **Step 4: 只针对仍失败的合同做最小措辞调整**

  若某项失败，先判断是：

  - 规则缺少可观察条件；
  - 输出模板缺少必填槽位；
  - 评估条件表达不清。

  只修改对应位置，然后用同一场景重新运行 5 次 fresh-context 测试。已经稳定通过的章节不扩写。

- [ ] **Step 5: 提交 GREEN 后的最小修正**

  如果 Task 4 产生修改：

  ```bash
  git add skills/english-podcast-speaking-coach/SKILL.md skills/english-podcast-speaking-coach/references/evaluation.md
  git commit -m "fix: 修复口语课堂体验回归"
  ```

  如果没有修改，跳过该提交。

---

### Task 5: 完成结构与维护性验证

**Files:**
- Verify: `skills/english-podcast-speaking-coach/SKILL.md`
- Verify: `skills/english-podcast-speaking-coach/references/evaluation.md`
- Verify: `skills/english-podcast-speaking-coach/references/persistence.md`
- Verify: `README.md`

**Interfaces:**
- Consumes: 已通过课堂轨迹回归的最终文档。
- Produces: 可交付、可维护且没有无关改动的优化结果。

- [ ] **Step 1: 检查 frontmatter、引用和行数**

  Run:

  ```bash
  sed -n '1,5p' skills/english-podcast-speaking-coach/SKILL.md
  rg -n "references/(persistence|evaluation)\\.md" skills/english-podcast-speaking-coach/SKILL.md
  wc -l skills/english-podcast-speaking-coach/SKILL.md skills/english-podcast-speaking-coach/references/evaluation.md
  ```

  Expected:

  - frontmatter 仍有正确的 `name` 和触发型 `description`；
  - 两个 reference 路由仍然存在；
  - 两个修改文件各自不超过 180 行。

- [ ] **Step 2: 检查文案和职责边界**

  Run:

  ```bash
  rg -n "v1|v2|旧版|新版" skills/english-podcast-speaking-coach
  rg -n "PROFILE\\.md|PHRASES\\.md|REVIEW\\.md|lessons/" skills/english-podcast-speaking-coach/references/persistence.md
  git diff 3b4a14b -- skills/english-podcast-speaking-coach/references/persistence.md README.md
  ```

  Expected:

  - skill 中没有版本式描述；
  - 四种持久化职责仍然存在；
  - `persistence.md` 和 `README.md` 相对基线没有变化。

- [ ] **Step 3: 检查 diff 质量和工作区**

  Run:

  ```bash
  git diff --check 3b4a14b
  git status --short
  git log --oneline 3b4a14b..HEAD
  ```

  Expected:

  - `git diff --check` 无输出；
  - 工作区只包含计划执行产生且尚未提交的预期文件，最终应为空；
  - 提交历史依次体现回归合同、课程节奏、掌握证据，以及必要时的回归修复。

- [ ] **Step 4: 最终人工审阅**

  逐条回答：

  - 新规则是否减少了本次课堂的认知切换，而不是减少整篇覆盖计划？
  - agent 是否可以根据材料密度和学习证据自行决定使用 1、2 或 3 个块，而不把判断交给学习者？
  - 所有新增规则是否都来自 `history/test/1.md` 的真实失败？
  - 是否可以删除任一新增句子而不损失可观察行为？可以则删除。
  - 普通课堂是否仍只需读取 `SKILL.md`，维护评估是否仍留在 `evaluation.md`？
