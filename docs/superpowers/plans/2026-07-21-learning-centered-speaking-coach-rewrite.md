# Learning-Centered English Podcast Speaking Coach Rewrite Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Rewrite `english-podcast-speaking-coach` in Chinese so it can cover as much valuable language from a podcast lesson as practical while teaching in small, adaptive, low-friction learning loops inspired by `learning-coach`.

**Architecture:** Keep one concise runtime entrypoint for teaching behavior, one conditionally loaded reference for persistent learning records, and one maintenance-only evaluation reference. Absorb the useful `learning-coach` principles into the speaking skill instead of creating a runtime dependency; separate flexible teaching heuristics from deterministic storage rules. Replace the current seven-file rule system only after the new entrypoint and references cover its essential behavior.

**Tech Stack:** Markdown skill files, local session-history fixtures, `rg`, `wc`, `git diff --check`, the `skill-creator` validator, and fresh-context manual/agentic forward tests.

## Global Constraints

- Write the new skill instructions in Chinese, using concise imperative wording.
- Keep the folder name and frontmatter name `english-podcast-speaking-coach` unchanged.
- Do not modify `skills/learning-coach/`; distill its teaching principles into the rewritten speaking skill.
- Do not create a hard runtime dependency on `learning-coach` being installed.
- Preserve broad lesson coverage: extract roughly 8–15 valuable expressions from a normal dense lesson when the material supports them.
- Treat 3–5 expressions as one learning block, not as the total lesson limit.
- Default to 6–10 core expressions for active recall and 3–6 secondary expressions for lighter practice or later review; never pad to a quota.
- Keep each learner prompt focused on one communicative job and one main new expression.
- Preserve attempt-before-reveal, target hiding, useful correction, near transfer, delayed retrieval, agent-selected goal-aligned transfer, and privacy protection.
- Require the agent to infer the learner's main use case from the current request, learner profile, recent learning history, and lesson material; do not show a daily/work/travel/exam mode menu.
- Ask at most one short calibration question only when two plausible goals would materially change the lesson and the available evidence cannot resolve the choice.
- Treat `history/good-case/1.md` as evidence for better prompt granularity, transfer, and repair loops, not as a complete gold standard for startup cost, session length, or persistence design.
- Do not require every lesson to execute retelling, free expression, speed checks, and every secondary target in one uninterrupted sitting.
- Do not mutate any real `english-coach/` learner workspace while implementing or testing the skill.
- Do not modify `history/bad-case/` or `history/good-case/`; use them only as evidence and regression fixtures.
- Preserve unrelated and untracked user files. In particular, do not stage the existing untracked `history/` or `skills/learning-coach/` trees unless the user explicitly requests it.
- Keep the runtime `SKILL.md` at or below 350 lines unless a concrete validation failure proves that additional hot-path guidance is necessary.
- Keep all references one level below `SKILL.md` under `references/`.
- Use Git history, rather than copied backup documents inside the skill folder, to preserve the deleted v1 rule files.

---

## Scope

### In scope

- Rewrite the skill frontmatter description and runtime body in Chinese.
- Replace transcript-coverage-first behavior with learning-block orchestration.
- Preserve full-lesson target discovery while reducing per-prompt cognitive load.
- Add explicit difficulty adaptation and concise correction budgets.
- Simplify persistent learner state to four visible responsibilities.
- Add a non-destructive path from legacy stateful-v1 workspaces to the lightweight v2 layout.
- Replace 18 overlapping regression scenarios with a smaller behavior-focused evaluation suite.
- Validate the skill structurally and against the recorded good/bad sessions.
- Forward-test the rewritten skill on the same EnglishPod lesson and one unseen lesson when raw transcripts are available.

### Out of scope

- Do not rewrite `learning-coach`.
- Do not add pronunciation coaching.
- Do not build an application, database, dashboard, or spaced-repetition engine.
- Do not migrate or edit an actual learner workspace during this repository change.
- Do not promise that all 8–15 expressions become durable mastery in one sitting.
- Do not preserve the v1 ledger/index/snapshot protocol merely for backward compatibility.
- Do not add a migration script in this pass. If real legacy workspaces prove too irregular for the documented non-destructive import, write a separate migration plan based on raw workspace samples.

---

## Target File Structure

```text
skills/english-podcast-speaking-coach/
  SKILL.md
  references/
    persistence.md
    evaluation.md
```

### File responsibilities

- `skills/english-podcast-speaking-coach/SKILL.md`
  - Own triggering, lesson target selection, learning blocks, adaptive difficulty, correction, transfer, retrieval, session boundaries, and concise routing to persistence.
  - Be the only file required before an ordinary live lesson.

- `skills/english-podcast-speaking-coach/references/persistence.md`
  - Own workspace discovery, the lightweight learner-record format, lesson-end writeback, privacy, and one-time non-destructive import from stateful-v1.
  - Be read only when existing cross-session state must be loaded, a workspace must be initialized/imported, or a lesson must be written back.

- `skills/english-podcast-speaking-coach/references/evaluation.md`
  - Own maintenance-only acceptance scenarios and trace-review rubrics.
  - Never be read during ordinary coaching.

### Files removed after replacement coverage is verified

- `skills/english-podcast-speaking-coach/INTERACTION-RULES.md`
- `skills/english-podcast-speaking-coach/RECOVERY-RULES.md`
- `skills/english-podcast-speaking-coach/REGRESSION-SCENARIOS.md`
- `skills/english-podcast-speaking-coach/STATE-SCHEMAS.md`
- `skills/english-podcast-speaking-coach/WORKSPACE-FORMAT.md`
- `skills/english-podcast-speaking-coach/WRITEBACK-FORMAT.md`

---

## Acceptance Criteria

### Learning experience

- A dense lesson may still produce an internal inventory of 8–15 valuable expressions.
- The inventory is divided into learning blocks of 3–5 related expressions.
- Before choosing transfer scenarios, the coach infers the learner's primary use case without asking the learner to configure a course mode.
- Explicit current-request evidence outranks profile/history evidence; profile/history outranks the transcript's default domain.
- If no use case is explicit, the coach selects the most common, low-load, transferable scenario and allows later learner behavior to override the inference.
- The first learner prompt asks for one sentence serving one communicative job.
- A prompt never asks the learner to produce three new target expressions at once.
- After one attempt, feedback focuses on the target expression and at most one additional transferable issue.
- After two unsuccessful attempts on the same expression, the coach reveals the complete natural form and gives at most one immediately simplified transfer; if that also fails, the item moves to review and returns through delayed retrieval instead of an unlimited repair loop.
- A learned expression is transferred into a realistic context aligned with the inferred learner goal inside the same block.
- A repaired expression is retrieved again several exchanges later or in the final integration.
- Every block ends with a compact checkpoint showing what the learner can already say and what still needs practice.
- The coach continues to the next block by default when the learner wants to master the whole lesson; it does not ask for permission after every expression.
- If the learner stops early, untouched high-value targets are preserved as next-session candidates rather than treated as failures.

### Coverage and mastery

- `core` means selected for active recall and transfer; it does not mean guaranteed mastery in one session.
- `secondary` means useful and worth at least a light attempt when time and energy allow.
- `vocabulary` remains separate from reusable spoken expressions.
- An expression enters the active phrase record only after unaided production in a new related context.
- Assisted or copied production remains in review.
- Pattern-level mastery avoids repeated tests that only swap a noun or name.

### Maintainability

- Ordinary coaching requires only `SKILL.md` plus short learner-state files, not a 500-line interaction reference.
- No teaching rule is duplicated across the entrypoint and references.
- Storage mechanics do not appear in learner-facing responses.
- The first learner-facing lesson message is a real speaking prompt, not setup narration, workspace initialization, or a mode-selection questionnaire.
- Classroom-lightness and startup/persistence-lightness are evaluated separately; passing one does not excuse failure in the other.
- New workspaces use `PROFILE.md`, `PHRASES.md`, `REVIEW.md`, and `lessons/` only.
- Legacy state is never deleted automatically.
- The old six rule files have no remaining inbound references before deletion.
- `SKILL.md` passes skill frontmatter validation and stays within the agreed line budget.

---

## Task 1: Establish the Baseline and Protect the Dirty Worktree

**Files:**

- Read: `skills/english-podcast-speaking-coach/*.md`
- Read: `skills/learning-coach/SKILL.md`
- Read: `skills/learning-coach/references/playbook.md`
- Read: `history/bad-case/1.md`
- Read: `history/bad-case/2.md`
- Read: `history/good-case/1.md`
- Read: `docs/superpowers/plans/*.md`
- Create: `/tmp/english-podcast-speaking-coach-v1-metrics.txt`

**Interfaces:**

- Consumes: the current skill and recorded session evidence.
- Produces: a read-only baseline used by every later acceptance check.

- [ ] **Step 1: Record the working tree without changing it**

Run:

```bash
git status --short
```

Expected: existing untracked `history/` and `skills/learning-coach/` may appear. Do not stage, delete, or rewrite them.

- [ ] **Step 2: Record the v1 size and file count**

Run:

```bash
wc -l -w skills/english-podcast-speaking-coach/*.md
```

Expected baseline: seven Markdown files totaling about 1,613 lines and 13,936 whitespace-delimited words.

- [ ] **Step 3: Record duplicated rule anchors**

Run:

```bash
rg -n "target hiding|target chunk|near-transfer|active recall|assistance|writeback|ledger|snapshot|needs_review|repaired|active" skills/english-podcast-speaking-coach
```

Expected: the same concepts appear in several files. Save the command output to the temporary baseline only if needed during execution; do not add generated metrics to the repository.

- [ ] **Step 4: Confirm the controlled comparison**

Run:

```bash
rg -n "10-14|next order of business|come up with|head up|这一轮先收住|这节课真正说出来的表达" history/bad-case/2.md history/good-case/1.md
```

Expected: `bad-case/2.md` shows compound prompts for the same EnglishPod lesson, while `good-case/1.md` shows one-expression prompts, immediate transfer, and block checkpoints.

- [ ] **Step 5: Commit**

No commit is required for the read-only baseline.

---

## Task 2: Write the Evaluation Contract Before Rewriting Runtime Rules

**Files:**

- Create: `skills/english-podcast-speaking-coach/references/evaluation.md`
- Read: `history/bad-case/2.md`
- Read: `history/good-case/1.md`

**Interfaces:**

- Consumes: the observed experience failures and successful combined-skill behavior.
- Produces: acceptance scenarios that constrain Tasks 3–7.

- [ ] **Step 1: Create the references directory**

Run:

```bash
mkdir -p skills/english-podcast-speaking-coach/references
```

Expected: the directory exists and contains no unrelated files.

- [ ] **Step 2: Add the evaluation reference with these mandatory sections**

Create `references/evaluation.md` in Chinese with this exact heading structure:

```markdown
# 维护评估

本文件只用于修改或评审 skill。普通课堂不要读取。

## 评估方法
## 场景 1：完整课文覆盖但单题不过载
## 场景 2：一次只修最重要的问题
## 场景 3：连续困难后有限递归修复
## 场景 4：自主推断目标并选择迁移场景
## 场景 5：延迟提取而不是机械重复
## 场景 6：中途结束仍保留后续目标
## 场景 7：启动和学习记录不阻塞开课
## 场景 8：自然答案不被课文原句否定
## 轨迹审查清单
```

For every scenario, include all four fields:

```markdown
- 输入：原始课文条件和学习者表现
- 预期：教练应该采取的行为
- 失败：可观察到的回归信号
- 依据：对应的设计原则
```

Encode these concrete expectations:

- Scenario 1: extract 8–15 expressions, group them into 3–5-expression blocks, and ask only one communicative job in the first prompt.
- Scenario 2: show one natural version, explain one main learning point, and add no more than one secondary issue unless meaning is lost.
- Scenario 3: after two failed attempts, reveal the natural form and allow at most one immediately simplified transfer; if that fails, schedule delayed retrieval instead of continuing an unlimited immediate loop.
- Scenario 4: infer the learner's use case from current request, profile, history, and transcript evidence; choose a goal-aligned transfer inside the first block without presenting a mode menu. Ask one calibration question only when unresolved ambiguity would materially change the lesson.
- Scenario 5: retrieve a repaired expression after several exchanges and skip noun-only repetition once the pattern is already active.
- Scenario 6: record unpracticed core candidates without marking them `needs_review`.
- Scenario 7: make the first learner-facing lesson message an actual speaking prompt; do not require a `继续` turn, mode selection, or visible workspace initialization. Score classroom flow and startup/persistence overhead as separate dimensions.
- Scenario 8: accept natural non-transcript wording first, then offer a useful lesson expression as an optional upgrade.

- [ ] **Step 3: Verify the evaluation file is maintenance-only and complete**

Run:

```bash
rg -n "普通课堂不要读取|场景 [1-8]|输入：|预期：|失败：|依据：|8–15|3–5|有限递归|自主推断|迁移场景|不阻塞开课|延迟提取" skills/english-podcast-speaking-coach/references/evaluation.md
```

Expected: all eight scenarios and all required fields match.

- [ ] **Step 4: Check formatting**

Run:

```bash
git diff --check
```

Expected: no output.

- [ ] **Step 5: Commit the evaluation contract**

Run:

```bash
git add skills/english-podcast-speaking-coach/references/evaluation.md
git commit -m "test: define speaking coach experience scenarios"
```

Expected: only `references/evaluation.md` is committed.

---

## Task 3: Rewrite the Runtime Entrypoint in Chinese

**Files:**

- Replace: `skills/english-podcast-speaking-coach/SKILL.md`
- Read: `skills/learning-coach/SKILL.md`
- Read: `skills/learning-coach/references/playbook.md`
- Read: `skills/english-podcast-speaking-coach/references/evaluation.md`

**Interfaces:**

- Consumes: learning-coach principles and the evaluation contract.
- Produces: the only hot-path instruction file required before ordinary coaching.

- [ ] **Step 1: Replace the YAML frontmatter**

Use only the two supported frontmatter fields:

```yaml
---
name: english-podcast-speaking-coach
description: 使用英语播客、EnglishPod、ESLPod、对话课文或音频转写稿进行口语学习。适用于用户说“学这一课”“练口语”“角色扮演”“复习这篇课文”，或希望把课文表达迁移到日常、职场、旅行、考试等真实用途时。先从整篇材料提取高价值表达，自主推断学习用途，再按小块进行主动回忆、纠错、真实迁移和延迟提取；不要用于单独的发音训练。
---
```

Do not add `version`, `metadata`, or other frontmatter fields.

- [ ] **Step 2: Replace the body with this exact section order**

```markdown
# 英语播客口语教练

## 目标
## 教学原则
## 推断学习目标与迁移场景
## 开始一节课
## 选择和分组表达
## 每个学习块
## 每轮教学循环
## 自适应难度
## 提示与答案揭示
## 纠错
## 目标对齐的迁移与延迟提取
## 整合输出
## 小结与继续学习
## 学习记录
## 常见错误
```

- [ ] **Step 3: Encode autonomous goal inference and scenario selection**

The `推断学习目标与迁移场景` section must state all of the following in imperative Chinese:

```text
不要要求学习者在“日常、职场、旅行、考试”等模式中做选择。
按以下优先级自行推断主要学习用途：当前请求中的明确目标；PROFILE.md 中的长期目标和偏好；最近课程中的真实使用场景；课文本身最自然的交际用途；最常见、认知负荷最低且可迁移的真实场景。
当前请求中的明确目标覆盖旧 profile 和历史偏好。
根据推断结果直接选择本课迁移场景并开始练习，不展示课程配置菜单。
只有两种合理方向会显著改变课程内容、且现有证据无法判断时，才问一个简短校准问题。
学习者后续表达出新的用途时，立即调整后续迁移场景，不要求重新开始课程。
```

Add one compact example showing inference, not a mode menu:

```text
同一篇商务会议课文：用户说“日常灵活表达”时迁移到周末计划、聚会分工或搬家预算；用户历史明确用于跨国会议时迁移到项目风险和任务分配；用户只说“学这一课”时采用原场景理解加常见真实场景迁移。
```

- [ ] **Step 4: Encode the target-selection contract**

The `选择和分组表达` section must state all of the following in imperative Chinese:

```text
先通读整篇材料，提取有明确交际用途、可迁移、口语自然的表达。
普通密集课文可保留约 8–15 个高价值表达；材料较短时按实际数量，不为达到数字而填充。
把表达分为：核心表达、次要表达、场景词汇。
核心表达通常 6–10 个，安排主动回忆和至少一次新场景使用。
次要表达通常 3–6 个，时间和精力允许时至少尝试一次；未练到的保留为下次候选，不算学习者失败。
每个学习块只放 3–5 个语义相关的表达。
每个提示只考一个交际功能和一个主要新表达；不要把同一块的多个表达塞进一道长翻译题。
```

- [ ] **Step 5: Encode the learning-coach loop without naming it as a dependency**

The `每轮教学循环` section must define this runtime loop:

```text
1. 关联：用一句话说明当前表达在什么真实情境中有用。
2. 铺垫：只解释完成尝试所需的场景词，不先给目标英文。
3. 尝试：让学习者先说一句。
4. 纠正：根据实际缺口给自然版本和最关键解释。
5. 迁移：根据已推断的学习用途，换一个最贴近目标的真实情境再次使用。
6. 延迟提取：几轮后或整合输出时再次检查。
```

Add the explicit rule: do not execute all six as visible process labels, and do not turn the reply into a methodology report.

- [ ] **Step 6: Encode adaptive difficulty and cap immediate recursive repair**

The `自适应难度` section must include these observable branches:

```text
- 连续两次轻松成功：组合两个已经学过的表达，或增加一条真实信息。
- 第一次明显困难：缩短句子，只保留目标表达和必要场景词。
- 同一表达连续两次困难：给完整自然表达，解释一个关键点，再提供一次更简单的新场景提取；不要要求第三次请求提示。
- 简化后的新场景仍失败：停止当前即时追问，把表达放入待复习，并在几轮后或下次课程做延迟提取；不要原地反复练到“说稳”为止。
- 学习者回答越来越短、明确说累或想结束：立即做当前块小结，保留未练目标供下次继续。
- 学习者主动要求完整覆盖：继续下一学习块，但仍保持一题一个交际功能。
```

- [ ] **Step 7: Encode the correction budget**

The `纠错` section must distinguish three cases:

```text
- 表达自然：指出具体成功点，最多做一个轻微优化，然后继续场景。
- 有可迁移问题：先给自然版本，再解释一个主问题；最多补充一个会影响理解或复用的次要问题。
- 错误很多：先保住交际意图和本轮目标表达。其他冠词、拼写、标点或风格问题只有在反复出现、影响理解或阻碍目标表达时才展开。
```

Require the coach to explain any useful phrase it introduces, but prohibit exhaustive diff reports for every cosmetic change.

- [ ] **Step 8: Encode help and target hiding with at most two assistance stages**

The `提示与答案揭示` section must state:

```text
学习者尝试前隐藏目标英文。
第一次求助时给一个关键词或短句框架，并立即让学习者再试。
仍然困难时直接给完整自然表达，不要求学习者反复发送“给我提示”。
完整答案出现后的复述只算辅助练习；稍后在新情境中独立说出，才算真正会用。
```

- [ ] **Step 9: Make full-lesson progression explicit**

The `每个学习块` and `小结与继续学习` sections must require:

- one coherent scenario per block;
- a compact checkpoint after 3–5 expressions;
- `已经能自己说出来` and `还要再练一下` as learner-facing labels;
- automatic continuation to the next block when the learner asked to master the whole lesson and has not asked to stop;
- no repeated “继续还是结束” question after every item;
- a clear remaining-target note when the session stops before full coverage.

- [ ] **Step 10: Route persistence conditionally**

The `学习记录` section must say:

```text
普通教学只维护简短的课内状态，不加载存储格式细节。
需要读取跨课复习、首次建立工作区、导入旧工作区或课末写回时，读取 `references/persistence.md`。
修改或评审本 skill 时读取 `references/evaluation.md`；普通课堂不要读取。
```

- [ ] **Step 11: Verify the runtime entrypoint**

Run:

```bash
wc -l skills/english-podcast-speaking-coach/SKILL.md
rg -n "8–15|3–5|一个交际功能|推断学习目标|不要要求学习者.*模式|显著改变课程|连续两次|简化后的新场景|延迟提取|references/persistence.md|references/evaluation.md" skills/english-podcast-speaking-coach/SKILL.md
rg -n "INTERACTION-RULES|STATE-SCHEMAS|WRITEBACK-FORMAT|RECOVERY-RULES|WORKSPACE-FORMAT|REGRESSION-SCENARIOS" skills/english-podcast-speaking-coach/SKILL.md
```

Expected:

- line count is 350 or fewer;
- every new behavioral anchor is present;
- the final negative search returns no matches and exit code `1`.

- [ ] **Step 12: Check formatting and commit**

Run:

```bash
git diff --check
git add skills/english-podcast-speaking-coach/SKILL.md
git commit -m "refactor: rewrite speaking coach learning flow"
```

Expected: the commit contains only `SKILL.md`.

---

## Task 4: Replace the Stateful Workspace Protocol with Lightweight Persistence

**Files:**

- Create: `skills/english-podcast-speaking-coach/references/persistence.md`
- Read: `skills/english-podcast-speaking-coach/WORKSPACE-FORMAT.md`
- Read: `skills/english-podcast-speaking-coach/STATE-SCHEMAS.md`
- Read: `skills/english-podcast-speaking-coach/WRITEBACK-FORMAT.md`
- Read: `skills/english-podcast-speaking-coach/RECOVERY-RULES.md`

**Interfaces:**

- Consumes: conditional route from the new `SKILL.md`.
- Produces: a four-responsibility learner workspace and non-destructive legacy import behavior.

- [ ] **Step 1: Create the persistence reference using this section order**

```markdown
# 学习记录

仅在读取跨课状态、初始化或导入工作区、课末写回时读取。

## 工作区发现
## v2 文件结构
## 启动读取
## 表达状态
## 课末记录
## 从 stateful-v1 非破坏导入
## 隐私
## 异常处理
```

- [ ] **Step 2: Define exactly four learner-state responsibilities**

Use this v2 layout:

```text
english-coach/
  PROFILE.md
  PHRASES.md
  REVIEW.md
  lessons/
```

Define responsibilities:

- `PROFILE.md`: learner goal, preferences, approximate ability, source series, `schema_version: learning-centered-v2`, and any one-time migration marker.
- `PHRASES.md`: only expressions with unaided new-context evidence; status is `active` or `stable`.
- `REVIEW.md`: expressions still being learned plus recurring grammar/collocation/spelling patterns; status is `learning` or `repaired`.
- `lessons/*.md`: concise human-readable lesson summaries and the source used to recover the latest session manually.

Do not recreate IDs, phrase-bank indexes, writeback ledgers, snapshots, dormant/retired sections, or materialized views.

- [ ] **Step 3: Define the minimal startup read set**

Require normal startup to read only:

```text
PROFILE.md
REVIEW.md
```

Read `PHRASES.md` only when checking duplication, selecting spaced review, or writing back. Read older lesson files only for a user-requested review or obvious state conflict.

- [ ] **Step 4: Define simple evidence rules**

Use these state transitions:

```text
first/assisted attempt -> REVIEW: learning
successful repair after help -> REVIEW: repaired
later unaided new-context production -> PHRASES: active
spaced unaided production in a later lesson -> PHRASES: stable
```

If an active phrase fails later, keep its evidence in `PHRASES.md` and add it back to `REVIEW.md`; do not introduce a six-state lifecycle.

- [ ] **Step 5: Define the concise lesson file**

Every completed lesson file must use this shape:

```markdown
# <日期> <课题>

## 本课场景
<一段话>

## 已经能主动说
- `<表达>`：<中文用途>；证据：<新场景中的学习者句子>

## 下次继续练
- `<表达或问题>`：<简短原因和中文回忆提示>

## 词汇
- `<词>`：<中文义>

## 下一步
<剩余学习块或下次优先复习内容>
```

Write the lesson file first, then update `PHRASES.md` and `REVIEW.md`, then update `PROFILE.md` only when a durable preference or ability estimate changed. Do not expose this order in learner-facing chat.

- [ ] **Step 6: Define non-destructive legacy import**

When `english-coach/WORKSPACE.md` exists but `PROFILE.md` does not:

1. Do not delay the first speaking prompt to perform migration.
2. Read the existing `MISSION.md`, `NOTES.md`, `state/CURRENT.md`, active portions of `state/review-queue.md` and `state/repair-bank.md`, and relevant `phrase-bank/*.md` files only when persistence is needed.
3. At lesson end, create the v2 files alongside the legacy files.
4. Copy goals/preferences into `PROFILE.md`, active/stable expressions into `PHRASES.md`, and active review/repair items into `REVIEW.md`.
5. Record `migrated_from: stateful-v1` and the date in `PROFILE.md`.
6. Never delete or rewrite the legacy files automatically.
7. On later sessions, prefer the v2 files when `schema_version: learning-centered-v2` is present.

If legacy data is malformed or contradictory, preserve it and import only unambiguous items; mention the skipped items in the new lesson file rather than rebuilding a transaction ledger.

- [ ] **Step 7: Verify persistence simplification**

Run:

```bash
rg -n "PROFILE.md|PHRASES.md|REVIEW.md|lessons/|learning-centered-v2|migrated_from|不要删除|先写 lesson" skills/english-podcast-speaking-coach/references/persistence.md
rg -n "writeback-ledger|phrase-bank-index|state-snapshots|PB-[0-9]|RQ-[0-9]|RB-[0-9]|attempted -> needs_review" skills/english-podcast-speaking-coach/references/persistence.md
```

Expected:

- all lightweight-state anchors are present;
- the negative search returns no matches and exit code `1`.

- [ ] **Step 8: Check formatting and commit**

Run:

```bash
git diff --check
git add skills/english-podcast-speaking-coach/references/persistence.md
git commit -m "refactor: simplify speaking coach persistence"
```

Expected: only `references/persistence.md` is committed.

---

## Task 5: Remove Superseded Rule Files and Eliminate Stale References

**Files:**

- Delete: `skills/english-podcast-speaking-coach/INTERACTION-RULES.md`
- Delete: `skills/english-podcast-speaking-coach/RECOVERY-RULES.md`
- Delete: `skills/english-podcast-speaking-coach/REGRESSION-SCENARIOS.md`
- Delete: `skills/english-podcast-speaking-coach/STATE-SCHEMAS.md`
- Delete: `skills/english-podcast-speaking-coach/WORKSPACE-FORMAT.md`
- Delete: `skills/english-podcast-speaking-coach/WRITEBACK-FORMAT.md`
- Verify: `skills/english-podcast-speaking-coach/SKILL.md`
- Verify: `skills/english-podcast-speaking-coach/references/*.md`

**Interfaces:**

- Consumes: replacement behavior from Tasks 2–4.
- Produces: the final three-file skill structure.

- [ ] **Step 1: Run a coverage audit before deletion**

Confirm the new files contain owners for these essential behaviors:

```bash
rg -n "目标英文|主动|新情境|生活迁移|延迟提取|隐私|发音|工作区|旧工作区|异常|评估" skills/english-podcast-speaking-coach/SKILL.md skills/english-podcast-speaking-coach/references/*.md
```

Expected: every concept has one clear owner. If any is missing, fix the new owner before deleting old files.

- [ ] **Step 2: Delete the six superseded files with `apply_patch`**

Use patch-based file deletion so the diff is reviewable. Do not use recursive deletion and do not remove the skill directory.

Expected: `git status --short` shows six deleted files and no changes under `history/` or `skills/learning-coach/`.

- [ ] **Step 3: Confirm no stale inbound references remain**

Run:

```bash
rg -n "INTERACTION-RULES|RECOVERY-RULES|REGRESSION-SCENARIOS|STATE-SCHEMAS|WORKSPACE-FORMAT|WRITEBACK-FORMAT" skills/english-podcast-speaking-coach
```

Expected: no matches; exit code `1`.

- [ ] **Step 4: Confirm the final file tree**

Run:

```bash
find skills/english-podcast-speaking-coach -maxdepth 2 -type f | sort
```

Expected exactly:

```text
skills/english-podcast-speaking-coach/SKILL.md
skills/english-podcast-speaking-coach/references/evaluation.md
skills/english-podcast-speaking-coach/references/persistence.md
```

- [ ] **Step 5: Check formatting and commit**

Run:

```bash
git diff --check
git add skills/english-podcast-speaking-coach
git commit -m "refactor: remove superseded speaking coach rules"
```

Expected: the commit contains only the six deletions; replacement files were committed in earlier tasks.

---

## Task 6: Run Static Skill Validation and Maintainability Checks

**Files:**

- Verify: `skills/english-podcast-speaking-coach/SKILL.md`
- Verify: `skills/english-podcast-speaking-coach/references/persistence.md`
- Verify: `skills/english-podcast-speaking-coach/references/evaluation.md`

**Interfaces:**

- Consumes: the final rewritten skill tree.
- Produces: structural and maintainability evidence before behavioral testing.

- [ ] **Step 1: Run the official skill validator**

Run:

```bash
python3 /Users/wxg/.codex/skills/.system/skill-creator/scripts/quick_validate.py skills/english-podcast-speaking-coach
```

Expected: validation succeeds with no frontmatter or naming errors.

- [ ] **Step 2: Verify size budgets**

Run:

```bash
wc -l -w skills/english-podcast-speaking-coach/SKILL.md skills/english-podcast-speaking-coach/references/*.md
```

Expected:

- `SKILL.md` is 350 lines or fewer;
- total runtime plus references is materially smaller than the v1 baseline of 1,613 lines;
- `evaluation.md` is not required during ordinary runtime.

- [ ] **Step 3: Verify Chinese-first instructions**

Run:

```bash
rg -n "## Overview|## Core Principles|## Lesson Flow|## Common Mistakes|Use this|Read this file" skills/english-podcast-speaking-coach
```

Expected: no English instruction headings or legacy English prose remain; English examples and schema values are allowed.

- [ ] **Step 4: Verify prohibited complexity is absent**

Run:

```bash
rg -n "ledger|snapshot|materialized view|idempotent|PB-[0-9]|RQ-[0-9]|RB-[0-9]|12-18 or more|Aim for 8-12" skills/english-podcast-speaking-coach
```

Expected: no matches; exit code `1`.

- [ ] **Step 5: Verify required experience anchors remain**

Run:

```bash
rg -n "8–15|3–5|核心表达|次要表达|一个交际功能|推断学习目标|模式中做选择|连续两次|简化后的新场景|新情境|目标.*迁移|延迟提取|已经能自己说出来|还要再练一下" skills/english-podcast-speaking-coach/SKILL.md
```

Expected: every anchor appears exactly where the runtime coach can use it.

- [ ] **Step 6: Run final diff checks**

Run:

```bash
git diff --check
git status --short
```

Expected: no uncommitted skill changes. Existing user-owned untracked files may remain.

- [ ] **Step 7: Commit fixes only if validation required edits**

If and only if validation fixes were needed:

```bash
git add skills/english-podcast-speaking-coach
git commit -m "fix: validate rewritten speaking coach"
```

Expected: the commit contains only validation-driven fixes.

---

## Task 7: Review the Rewrite Against the Existing Good and Bad Traces

**Files:**

- Read: `history/bad-case/1.md`
- Read: `history/bad-case/2.md`
- Read: `history/good-case/1.md`
- Read: `skills/english-podcast-speaking-coach/SKILL.md`
- Read: `skills/english-podcast-speaking-coach/references/evaluation.md`
- Create: `/tmp/english-podcast-speaking-coach-trace-review.md`

**Interfaces:**

- Consumes: recorded user experience and the final rules.
- Produces: a temporary evidence matrix; no repository artifact.

- [ ] **Step 1: Build a trace comparison matrix in `/tmp`**

Use these rows:

```text
first-prompt target count
announced total workload
goal inference evidence and selected scenario pool
mode-selection questions before practice
main correction points per turn
time to first successful transfer
checkpoint frequency
goal-aligned transfer timing
immediate repair attempts before delayed retrieval
delayed retrieval present
unnecessary pattern repetition
workspace/setup chatter before practice
lesson-end active/review distinction
```

For each row, record:

- bad-case evidence;
- good-case evidence;
- new rule that prevents the bad behavior;
- evaluation scenario that detects regression.

- [ ] **Step 2: Confirm every observed improvement has a runtime owner**

Run:

```bash
rg -n "一个交际功能|推断学习目标|模式中做选择|自然版本|主问题|第一.*学习块|检查点|目标.*迁移|简化后的新场景|延迟提取|工作区" skills/english-podcast-speaking-coach/SKILL.md skills/english-podcast-speaking-coach/references/*.md
```

Expected: every row in the temporary matrix maps to one runtime rule and one evaluation scenario.

- [ ] **Step 3: Confirm good-case coverage is not accidentally reduced to 3–5 total**

Run:

```bash
rg -n "8–15|6–10|3–6|每个学习块.*3–5" skills/english-podcast-speaking-coach/SKILL.md
```

Expected: all four coverage anchors appear. Any rule saying a whole lesson should end after only 3–5 expressions is a failure.

- [ ] **Step 4: Keep the trace review temporary**

Do not add `/tmp/english-podcast-speaking-coach-trace-review.md` to Git. Summarize its findings in the implementation handoff instead.

- [ ] **Step 5: Commit**

No commit is required for trace review unless it exposes a rule gap. If a gap is fixed, commit only the affected skill file with message:

```bash
git commit -m "fix: close speaking coach trace regression"
```

---

## Task 8: Forward-Test the Skill on Real Lesson Material

**Files:**

- Use: `skills/english-podcast-speaking-coach/`
- Prefer raw input: the original `englishpod_0025.txt` referenced by the histories.
- Use a second raw transcript not represented in `history/` for generalization.
- Do not write test learner state into the repository.

**Interfaces:**

- Consumes: the validated rewritten skill and raw podcast transcripts.
- Produces: realistic session traces and the final go/no-go decision.

- [ ] **Step 1: Resolve raw test material**

Check whether the original transcript paths referenced in the history are readable. If they are not available, request the user to attach the original `englishpod_0025.txt` and one other representative lesson before claiming forward-test completion.

Expected: two raw transcripts, one controlled comparison and one unseen lesson.

- [ ] **Step 2: Prepare isolated test locations**

Use fresh temporary directories created with `mktemp -d`. Do not let either test see the repository histories, the expected conclusions, or the other test's output.

Expected: each test starts without prior learner state and writes only under its temporary directory.

- [ ] **Step 3: Run the controlled EnglishPod 0025 session**

Use the natural learner request:

```text
使用 english-podcast-speaking-coach 学习这篇课文。我想尽可能掌握整篇课文。
```

Do not mention `bad-case`, `good-case`, expected expression counts, or suspected defects in the test prompt.

Expected first-block behavior:

- builds a broad internal inventory without dumping it as a workload report;
- infers a sensible mixed use case from the terse request and lesson material without asking the learner to select daily/work/travel/exam mode;
- first prompt asks for one sentence and one communicative function;
- practices 3–5 expressions before a checkpoint;
- includes a goal-aligned real-life transfer inside the first block;
- continues toward full-lesson coverage without combining several new targets in one prompt.

- [ ] **Step 4: Run the unseen-lesson session**

Use an explicit but non-configurational request with the second transcript:

```text
使用 english-podcast-speaking-coach 学习这篇课文。我主要想把里面的表达用于跨国项目会议，也希望尽可能掌握整篇课文。
```

Expected: the same block structure generalizes, current-request workplace evidence overrides any generic daily-life default, and transfer scenarios fit project meetings without copying the bird-flu examples.

- [ ] **Step 5: Score both traces using `references/evaluation.md`**

Pass conditions:

- all eight evaluation scenarios pass;
- no learner-facing reply exposes storage mechanics;
- no opening mode menu or unnecessary calibration question appears;
- no whole-lesson limit of 3–5 expressions appears;
- no prompt combines more than one main new expression unless it is an explicit later integration task;
- no failed expression receives more than one immediately simplified transfer after the complete answer is revealed;
- correction remains concise enough that the next speaking opportunity is clear;
- the learner can identify remaining lesson coverage at each block boundary.

- [ ] **Step 6: Iterate once for concrete failures only**

For every failure, change the smallest owning rule. Do not reintroduce duplicated rules in multiple files. Rerun only the failed scenario and one previously passing scenario that shares the same rule owner.

- [ ] **Step 7: Re-run validation after any edit**

Run:

```bash
python3 /Users/wxg/.codex/skills/.system/skill-creator/scripts/quick_validate.py skills/english-podcast-speaking-coach
git diff --check
```

Expected: both commands succeed.

- [ ] **Step 8: Commit forward-test fixes**

If changes were required:

```bash
git add skills/english-podcast-speaking-coach
git commit -m "fix: tune speaking coach from forward tests"
```

Expected: only evidence-driven skill changes are committed; test traces and temporary workspaces remain untracked outside the repository.

---

## Task 9: Final Self-Review and Handoff

**Files:**

- Verify: `skills/english-podcast-speaking-coach/SKILL.md`
- Verify: `skills/english-podcast-speaking-coach/references/persistence.md`
- Verify: `skills/english-podcast-speaking-coach/references/evaluation.md`
- Verify: Git history for implementation commits.

**Interfaces:**

- Consumes: all implementation and validation evidence.
- Produces: a concise user handoff with remaining limitations.

- [ ] **Step 1: Check specification coverage**

Confirm each global constraint and acceptance criterion maps to one implemented rule or validation result. Specifically verify:

- full lesson coverage remains 8–15 when material supports it;
- 3–5 is only a block size;
- the agent infers the learning use case without presenting a mode menu, and explicit current-request evidence has highest priority;
- teaching is Chinese-first and one-function-per-prompt;
- adaptive difficulty has an explicit two-failure branch and a one-simplified-transfer cap before delayed retrieval;
- persistence is lightweight and conditionally loaded;
- legacy data is preserved non-destructively;
- `learning-coach` remains unchanged and optional at runtime.

- [ ] **Step 2: Check the final tree and worktree**

Run:

```bash
find skills/english-podcast-speaking-coach -maxdepth 2 -type f | sort
git status --short
git log --oneline -8
```

Expected: the final skill has exactly three files; no unintended user files are staged or modified.

- [ ] **Step 3: Report measured simplification**

Run:

```bash
wc -l -w skills/english-podcast-speaking-coach/SKILL.md skills/english-podcast-speaking-coach/references/*.md
```

Compare against the v1 baseline. Report both the line-count reduction and the fact that only `SKILL.md` is hot-path runtime context.

- [ ] **Step 4: Report behavioral validation**

Summarize:

- which good-case behaviors were preserved;
- which bad-case behaviors were prevented;
- how the agent inferred the target use case and selected transfer scenarios without asking the learner to configure the course;
- how many expressions the controlled lesson selected and how they were divided into blocks;
- any remaining uncertainty about real legacy workspace import.

- [ ] **Step 5: Do not claim legacy migration reliability without real samples**

If no real stateful-v1 workspace was tested, explicitly say that the import rules are designed but not proven against live learner data. Recommend a separate migration-script plan only if actual samples expose ambiguity.

---

## Self-Review Checklist

- [ ] Every requested behavior is owned by exactly one file.
- [ ] No step changes `skills/learning-coach/`.
- [ ] No step edits recorded history fixtures.
- [ ] No task treats 3–5 as the full lesson limit.
- [ ] No task requires the learner to choose a daily/work/travel/exam mode before practice.
- [ ] Goal inference priority is consistent across runtime rules, evaluation scenarios, trace review, and forward tests.
- [ ] Immediate recursive repair is capped at one simplified transfer after answer reveal.
- [ ] Good-case is used as evidence for interaction improvements, not as a gold standard for startup, duration, or persistence.
- [ ] Evaluation precedes runtime implementation.
- [ ] File deletion occurs only after replacement coverage is verified.
- [ ] Commands include expected outcomes.
- [ ] No step uses recursive destructive deletion.
- [ ] No placeholders such as `TODO`, `TBD`, or “implement similarly” remain.
- [ ] Forward tests use raw lesson artifacts without leaking expected conclusions.
- [ ] Final claims distinguish validated behavior from untested legacy migration behavior.
