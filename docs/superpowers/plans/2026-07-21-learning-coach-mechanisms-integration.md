# Learning Coach Mechanisms Integration Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** 在不增加教学阶段、文件或复杂排期系统的前提下，把支架淡出、对比式穿插、重复错误根因修复和轻量跨课间隔融入英语播客口语教练。

**Architecture:** 四项机制分别进入现有职责边界：课堂行为写入 `SKILL.md`，跨课间隔写入 `references/persistence.md`，行为合同写入 `references/evaluation.md`，用户说明同步到 `README.md`。所有规则采用可观察条件和正向行动，不引入新的状态文件、日期算法、学习模式选择或独立教学阶段。

**Tech Stack:** Markdown skill 文档、Git、基于课堂轨迹的行为回归测试。

## Global Constraints

- 保留整课 8–15 个高价值候选与单块 3–5 个表达的现有覆盖策略。
- 保持 agent 自主判断学习用途与课堂节奏。
- 不增加专注、睡眠、flashcards、完整 SRS 排期或固定自评问卷。
- 不出现版本称谓、旧版迁移或发音排除说明。

---

### Task 1: 定义四项行为回归场景

**Files:**
- Modify: `skills/english-podcast-speaking-coach/references/evaluation.md`

**Interfaces:**
- Consumes: 当前核心表达循环、援助阶梯、纠错规则和表达状态。
- Produces: 可用于 RED/GREEN 复测的四类行为合同。

- [ ] **Step 1: 运行当前 skill 的基线轨迹**

  使用以下四个场景检查当前规则是否能给出唯一行为：辅助成功后的下一题、两个近义表达的整合、同一错误反复出现、`repaired`/`active` 的跨课复习时机。

- [ ] **Step 2: 确认基线失败原因**

  预期至少出现以下缺口：没有明确撤除支架；没有要求学习者自主选择近义表达；重复错误可能继续表层纠正；状态存在但没有轻量复习节奏。

- [ ] **Step 3: 将四类合同写入评估文件**

  每个场景包含输入、预期、失败和依据，并确保测试关注行为而非固定措辞。

### Task 2: 融合三项课内机制

**Files:**
- Modify: `skills/english-podcast-speaking-coach/SKILL.md`
- Test: `skills/english-podcast-speaking-coach/references/evaluation.md`

**Interfaces:**
- Consumes: 核心表达教学循环、难度与援助、纠错、整合输出。
- Produces: 支架淡出、对比式穿插和根因修复的运行时规则。

- [ ] **Step 1: 在援助阶梯后加入支架淡出**

  辅助成功后的下一次提取撤除上一层支架；完整示范后的即时简化使用仍是辅助练习，延迟提取恢复无提示。

- [ ] **Step 2: 在整合输出中加入对比式穿插**

  当学习块存在功能相近或容易混淆的表达时，安排一次不指定目标表达的新场景，让学习者自主选择，并只解释影响选择的一个语气或使用条件。

- [ ] **Step 3: 在纠错中加入根因修复**

  同一问题在本课或近期记录中再次出现时，定位一个底层缺口，完成一次简短专项修复，再回到原口语场景验证。

- [ ] **Step 4: 用原场景复测三项课内机制**

  预期 agent 能唯一说明何时撤支架、何时做对比提取、何时从表层纠错切换为根因修复。

### Task 3: 加入轻量跨课间隔

**Files:**
- Modify: `skills/english-podcast-speaking-coach/references/persistence.md`
- Test: `skills/english-podcast-speaking-coach/references/evaluation.md`

**Interfaces:**
- Consumes: `learning`、`repaired`、`active`、`stable` 状态和启动读取规则。
- Produces: 不依赖日期字段的跨课复习优先级。

- [ ] **Step 1: 定义状态对应的复习节奏**

  `repaired` 在下一次相关课程优先出现；`active` 间隔 2–3 次课程后在自然场景中检查；`stable` 在自然适配或学习者主动要求时复习。

- [ ] **Step 2: 保持上下文匹配优先**

  启动时从到期项目中选择与当前课程自然相容的少量表达，其他项目等待后续合适场景。

- [ ] **Step 3: 验证状态与节奏没有冲突**

  首次合格证据仍只支持 `active`，跨课间隔成功才支持 `stable`；复习节奏不自动改变状态。

### Task 4: 同步说明并完成验证

**Files:**
- Modify: `README.md`
- Verify: `skills/english-podcast-speaking-coach/SKILL.md`
- Verify: `skills/english-podcast-speaking-coach/references/persistence.md`
- Verify: `skills/english-podcast-speaking-coach/references/evaluation.md`

**Interfaces:**
- Consumes: 已通过复测的运行时与持久化规则。
- Produces: 与实际 skill 一致的用户说明和最终验证证据。

- [ ] **Step 1: 更新 README 的练习原则**

  用简短条目说明辅助会逐步撤除、相近表达会穿插选择、重复问题会做根因修复、复习按状态轻量间隔。

- [ ] **Step 2: 运行结构与文本检查**

  运行 frontmatter 校验、引用文件存在性检查、禁止旧术语扫描和 `git diff --check`。

- [ ] **Step 3: 运行最终轨迹复测**

  使用 Task 1 的相同四个场景，并确认既有默认场景、覆盖节奏、角色扮演、延迟提取和状态升级场景没有回归。
