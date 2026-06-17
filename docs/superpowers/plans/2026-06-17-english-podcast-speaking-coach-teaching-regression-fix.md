# English Podcast Speaking Coach Teaching Regression Fix Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** 修复 `english-podcast-speaking-coach` 拆分后出现的教学体验回退：角色扮演英文台词不稳定、纠错解释变薄、课内表达回拉不足、课末复习材料变瘦。

**Architecture:** 只收紧 live coaching 规则，不改变 workspace、state schema、writeback、recovery 或 mastery lifecycle。主要修改 `INTERACTION-RULES.md`，必要时在 `SKILL.md` 的 Lesson Flow 和 Common Mistakes 中增加短规则，保证入口文件能提醒 agent 读取并执行这些口径。

**Tech Stack:** Markdown skill files, `rg`, `sed`, Git, manual transcript regression checks.

---

## Source Evidence

本计划来自一次 old/new 会话记录对比。关键观察如下：

- old 会话中，对方角色台词通常使用英文直接话语，例如 `Receptionist: Good afternoon. What can I do for you?`。
- new 会话中，同类角色扮演有时退化为中文描述，例如「我问你：都准备好了吗？」。
- old 会话对复述和自由表达中的错误解释更完整，会逐项解释拼写、搭配、课内表达、连接词、间接转述和中式表达。
- new 会话仍能做近场景修复，但常只选 2-3 个点讲，容易漏掉其他可迁移错误。
- new 规则中 `Keep feedback short first` 容易压过 `Explain every key modification`，导致「短反馈」被误解成「少讲错因」。
- new 课末总结更轻，但复习价值下降，缺少 old 版本中常见的中文含义、用法、用户例句、回忆提示和状态说明。

## Scope

### In Scope

- 收紧角色扮演语言协议。
- 明确「短反馈」和「完整纠错」的优先级。
- 强化中式表达识别和解释。
- 强化课内表达回拉。
- 强化 free retelling 和 free expression 的纠错覆盖。
- 恢复课末总结的可复习信息密度。
- 增加手工回归检查清单。

### Out Of Scope

- 不重写课程流程。
- 不修改 target hiding。
- 不修改 mastery lifecycle。
- 不修改 durable state 文件格式。
- 不新增自动化测试框架。
- 不移动 reference files。
- 不修改 learner workspace 下的 `english-coach/` 数据。

## File Structure

Modify:

- `skills/english-podcast-speaking-coach/INTERACTION-RULES.md`
  - 增加 `Role-Play Language Contract`。
  - 调整 `Learner-Facing Voice` 中的短反馈规则。
  - 调整 `Learner-Facing Templates` 中的完整纠错要求。
  - 增加课内表达回拉规则。
  - 增强 `Free Retelling`、`Free Expression`、`Correction`、`Lesson-End Expression Areas` 和 `Common Mistakes`。

- `skills/english-podcast-speaking-coach/SKILL.md`
  - 在 Lesson Flow 中增加一句入口级提醒：角色对手台词默认英文，纠错完整性优先于短反馈。
  - 在 Common Mistakes 中增加 2-3 条高层反例，避免入口文件读者只看到轻量措辞优化。

Do not modify:

- `skills/english-podcast-speaking-coach/STATE-SCHEMAS.md`
- `skills/english-podcast-speaking-coach/WORKSPACE-FORMAT.md`
- `skills/english-podcast-speaking-coach/WRITEBACK-FORMAT.md`
- `skills/english-podcast-speaking-coach/RECOVERY-RULES.md`
- existing plan/spec documents

## Implementation Tasks

### Task 1: Preflight And Baseline

**Files:**

- Read: `skills/english-podcast-speaking-coach/SKILL.md`
- Read: `skills/english-podcast-speaking-coach/INTERACTION-RULES.md`

- [ ] **Step 1: Check working tree state**

Run:

```bash
git status --short
```

Expected:

```text
?? .gitignore
```

If additional files appear, inspect them before editing. Do not modify unrelated files.

- [ ] **Step 2: Confirm current headings**

Run:

```bash
rg -n "^## |^### " skills/english-podcast-speaking-coach/SKILL.md skills/english-podcast-speaking-coach/INTERACTION-RULES.md
```

Expected headings in `INTERACTION-RULES.md` include:

```text
## Lesson Setup
## Guided Preview
## Learner-Facing Voice
## Active-Recall Blocks
## Learner-Facing Templates
## Help Requests
## Comprehensible Input Guard
## Prompt Leakage Guard
## Phrase Substitution
## Near-Transfer Vocabulary
## Free Retelling
## Free Expression
## Coverage Tracking
## Speed Round
## Correction
## Lesson-End Expression Areas
## Common Mistakes
```

- [ ] **Step 3: Confirm no state schema files need changes**

Run:

```bash
rg -n "target_hidden|Writeback Summary|Review Queue Item|Phrase Bank Item" skills/english-podcast-speaking-coach
```

Expected:

```text
skills/english-podcast-speaking-coach/STATE-SCHEMAS.md
skills/english-podcast-speaking-coach/WRITEBACK-FORMAT.md
```

Only proceed with `INTERACTION-RULES.md` and `SKILL.md` edits.

### Task 2: Add Role-Play Language Contract

**Files:**

- Modify: `skills/english-podcast-speaking-coach/INTERACTION-RULES.md`

- [ ] **Step 1: Add a new subsection after `Learner-Facing Voice`**

Insert this section immediately before `## Active-Recall Blocks`:

```markdown
## Role-Play Language Contract

During direct role-play, the coach's in-character lines should be English by default. Chinese is for setup, intent prompts, vocabulary explanation, and feedback. This keeps the learner inside the spoken scene.

Use this split:

- Coach-as-teacher: Chinese setup and instructions.
- Coach-as-role: English direct speech.
- Learner prompt: Chinese communicative intent plus explained vocabulary.

If an English role line would reveal a target chunk the learner should produce later, do not replace the whole role-play with Chinese. Instead, choose one of these options:

1. Paraphrase the coach line in English without the target chunk.
2. Switch roles so the learner produces the target first.
3. Give the role line in Chinese only when no natural English paraphrase is possible.
4. Demote that target to exposure-only if the coach already modeled it in English.

Avoid prompts shaped like `我问你：「...」` when the coach is supposed to be another speaker in the scene. Prefer an English line plus a Chinese intent prompt:

```text
Travel partner: Are we all set?

你想回答：差不多，车已经装好了，零食、音乐和地图也都在车里。请用英文说。
```
```

- [ ] **Step 2: Update `Active-Recall Blocks` to reference the new contract**

Change item 1 from:

```markdown
1. Start a new role-play scenario and play the other speaker.
```

To:

```markdown
1. Start a new role-play scenario and play the other speaker. Follow `Role-Play Language Contract`: in-character coach lines are English by default; Chinese explains the learner's communicative intent.
```

- [ ] **Step 3: Verify the section exists**

Run:

```bash
rg -n "Role-Play Language Contract|Coach-as-role|Travel partner: Are we all set" skills/english-podcast-speaking-coach/INTERACTION-RULES.md
```

Expected:

```text
skills/english-podcast-speaking-coach/INTERACTION-RULES.md:<line>:## Role-Play Language Contract
skills/english-podcast-speaking-coach/INTERACTION-RULES.md:<line>:- Coach-as-role: English direct speech.
skills/english-podcast-speaking-coach/INTERACTION-RULES.md:<line>:Travel partner: Are we all set?
```

### Task 3: Fix Feedback Priority

**Files:**

- Modify: `skills/english-podcast-speaking-coach/INTERACTION-RULES.md`
- Modify: `skills/english-podcast-speaking-coach/SKILL.md`

- [ ] **Step 1: Replace the ambiguous short-feedback rule**

In `INTERACTION-RULES.md`, replace:

```markdown
- Keep feedback short first. Expand only when there is a real transferable problem, repeated issue, or useful contrast to explain.
```

With:

```markdown
- Keep feedback brief for mostly natural answers. When there is a transferable error, lesson-chunk mismatch, Chinglish wording, or information-order problem, full correction wins over brevity.
```

- [ ] **Step 2: Strengthen the full correction template**

In `Learner-Facing Templates`, replace:

```markdown
For a meaningful spelling, grammar, collocation, Chinglish, lesson-chunk, or information-organization issue, use the full correction shape. Use a feedback label only when there is something transferable to explain.
```

With:

```markdown
For a meaningful spelling, grammar, collocation, Chinglish, lesson-chunk, or information-organization issue, use the full correction shape. If the natural version changes several learner choices, explain every changed choice or remove the change. Brevity must not hide a teachable reason.
```

- [ ] **Step 3: Add a short entrypoint reminder**

In `SKILL.md`, under `During the lesson:`, replace:

```markdown
- Use problem-triggered correction: short feedback for natural answers, full correction for transferable problems.
```

With:

```markdown
- Use problem-triggered correction: brief feedback for natural answers, full correction for transferable problems. Full correction has priority when the learner makes Chinglish, lesson-chunk, tense, article, collocation, linkage, or information-order errors.
```

- [ ] **Step 4: Verify wording**

Run:

```bash
rg -n "full correction wins|Brevity must not hide|Full correction has priority" skills/english-podcast-speaking-coach
```

Expected three matches across `INTERACTION-RULES.md` and `SKILL.md`.

### Task 4: Add Lesson-Language Anchoring

**Files:**

- Modify: `skills/english-podcast-speaking-coach/INTERACTION-RULES.md`

- [ ] **Step 1: Add a subsection after `Near-Transfer Vocabulary`**

Insert:

```markdown
## Lesson-Language Anchoring

The coach should not merely polish the learner's sentence. When the lesson provides a useful spoken chunk, actively pull the learner toward that chunk after the learner attempts the meaning.

After a correction, identify which parts are:

- learner wording that is already natural;
- lesson chunk or natural spoken upgrade;
- repair target that needs another attempt;
- optional extension that should not be graded yet.

When a learner uses a natural but non-lesson wording, accept it first, then offer the lesson chunk as an upgrade if it is useful:

```text
`everything covered` 没错；这课里更地道的块是 `all our bases covered`，更像「该想到的点都想到了」。
```

When a learner uses a Chinglish expression, explicitly name the mismatch in Chinese and give the natural English shape:

```text
`protect us from new employee risks` 听起来像中文直译。这里英语更自然地说「比正式招新人风险小」：`less risky than taking on new staff`。
```

Do not force every sentence to match the transcript. Prioritize reusable chunks that fit realistic speech.
```

- [ ] **Step 2: Reference anchoring in `Correction`**

Add this bullet under `## Correction` after the existing bullet that compares learner wording with lesson chunks:

```markdown
- When a lesson chunk is a better target than the learner's wording, say what the learner's wording means, why the lesson chunk is more natural or more useful, and then create a near-transfer prompt that hides the chunk again.
```

- [ ] **Step 3: Verify wording**

Run:

```bash
rg -n "Lesson-Language Anchoring|learner wording that is already natural|Chinglish expression|near-transfer prompt that hides the chunk" skills/english-podcast-speaking-coach/INTERACTION-RULES.md
```

Expected four matches.

### Task 5: Strengthen Free Retelling Repair

**Files:**

- Modify: `skills/english-podcast-speaking-coach/INTERACTION-RULES.md`

- [ ] **Step 1: Replace the final paragraph of `Free Retelling`**

Replace:

```markdown
Let the learner finish before polishing. Focus on first-person consistency, tense, sentence linkage, and natural lesson language. If there is a transferable key error, use the same problem-triggered correction style and then ask for a near-transfer repair of only the affected 1-2 sentences. Do not ask the learner to copy the polished sentence verbatim. A successful near-transfer repair counts as learner production.
```

With:

```markdown
Let the learner finish before polishing. Free retelling corrections should cover all transferable issues introduced by the polished version: first-person consistency, tense, article choice, singular/plural forms, sentence linkage, information order, Chinglish wording, and natural lesson language.

If the retelling has many issues, group them by type instead of dropping them:

- story order and linkage;
- tense and time reference;
- article and countability;
- lesson chunks and Chinglish replacements;
- reporting verbs and direct/indirect speech.

After the explanation, repair only the highest-value 1-2 sentences through near-transfer. Do not ask the learner to copy the polished paragraph verbatim. A successful near-transfer repair counts as learner production.
```

- [ ] **Step 2: Add prompt-quality guard**

Add this paragraph at the end of `Free Retelling`:

```markdown
Avoid Chinese prompts that induce unnatural English, such as vague nouns like 「忘了一件事」 when the intended English is `forgot to + verb`. Prefer action-based prompts such as 「忘了上厕所」「忘了买水」「忘了带文件」.
```

- [ ] **Step 3: Verify wording**

Run:

```bash
rg -n "Free retelling corrections should cover|story order and linkage|忘了一件事" skills/english-podcast-speaking-coach/INTERACTION-RULES.md
```

Expected three matches.

### Task 6: Strengthen Free Expression Repair

**Files:**

- Modify: `skills/english-podcast-speaking-coach/INTERACTION-RULES.md`

- [ ] **Step 1: Expand `Free Expression`**

Append this paragraph to `## Free Expression`:

```markdown
Free expression is not only a fluency check. When the learner chooses a real-life situation, correct transferable problems with the same standard as role-play and retelling. Accept natural alternatives, but if the learner misses a high-value lesson chunk that fits the chosen situation, offer it as an upgrade after the attempt and test it once in a near-transfer or final check.
```

- [ ] **Step 2: Verify wording**

Run:

```bash
rg -n "Free expression is not only a fluency check|offer it as an upgrade" skills/english-podcast-speaking-coach/INTERACTION-RULES.md
```

Expected two matches.

### Task 7: Restore Richer Lesson-End Summary

**Files:**

- Modify: `skills/english-podcast-speaking-coach/INTERACTION-RULES.md`

- [ ] **Step 1: Update `这节课真正说出来的表达`**

Replace:

```markdown
Include only expressions the learner actively produced or successfully repaired through a near-transfer scenario and then produced unaided in a new but related context. Aim for 8-12 when enough qualify; never pad with unpracticed expressions. For durable/writeback records, each entry includes:

`chunk`, Chinese meaning, use case, personal sentence or corrected personal sentence, Chinese recall prompt, and status `active`.
```

With:

```markdown
Include only expressions the learner actively produced or successfully repaired through a near-transfer scenario and then produced unaided in a new but related context. Aim for 8-12 when enough qualify; never pad with unpracticed expressions.

Learner-facing summaries should be compact but usable for later recall. For each important active expression, include:

- expression chunk;
- Chinese meaning;
- use case;
- learner sentence or corrected learner sentence;
- Chinese recall prompt.

For durable/writeback records, also include status `active`.
```

- [ ] **Step 2: Update `下次还要再练的表达`**

Append:

```markdown
For each review item, include the reason it remains in review when useful, such as `不是 be virus`, `to 漏了`, `具体物品前的 the`, or `中式直译`. Keep the reason short and actionable.
```

- [ ] **Step 3: Verify wording**

Run:

```bash
rg -n "usable for later recall|Chinese recall prompt|reason it remains in review" skills/english-podcast-speaking-coach/INTERACTION-RULES.md
```

Expected three matches.

### Task 8: Add Anti-Regression Common Mistakes

**Files:**

- Modify: `skills/english-podcast-speaking-coach/INTERACTION-RULES.md`
- Modify: `skills/english-podcast-speaking-coach/SKILL.md`

- [ ] **Step 1: Add detailed common mistakes to `INTERACTION-RULES.md`**

Append these bullets under `## Common Mistakes`:

```markdown
- Replacing in-character English role-play lines with Chinese narration when no target leakage requires it.
- Treating `Keep feedback brief` as permission to skip important tense, article, collocation, Chinglish, lesson-chunk, linkage, or information-order explanations.
- Polishing a retelling paragraph but explaining only 2-3 of the changes while silently changing other learner choices.
- Accepting a natural answer without offering a high-value lesson chunk that the learner was supposed to acquire.
- Ending with a thin expression list that lacks meaning, use case, learner example, or recall prompt for important active expressions.
```

- [ ] **Step 2: Add high-level common mistakes to `SKILL.md`**

Append these bullets under `## Common Mistakes`:

```markdown
- Letting role-play collapse into Chinese narration instead of English in-character speech plus Chinese intent prompts.
- Letting brevity override full correction for transferable learner errors.
- Ending with summaries that are easy to skim but too thin to support later recall.
```

- [ ] **Step 3: Verify wording**

Run:

```bash
rg -n "Chinese narration|brevity override|too thin to support later recall|silently changing other learner choices" skills/english-podcast-speaking-coach
```

Expected matches in both `INTERACTION-RULES.md` and `SKILL.md`.

### Task 9: Manual Regression Checks

**Files:**

- Read: `skills/english-podcast-speaking-coach/INTERACTION-RULES.md`
- Read: `skills/english-podcast-speaking-coach/SKILL.md`

- [ ] **Step 1: Static rule check**

Run:

```bash
rg -n "Role-Play Language Contract|full correction wins|Lesson-Language Anchoring|Free retelling corrections should cover|Free expression is not only|usable for later recall" skills/english-podcast-speaking-coach/INTERACTION-RULES.md
```

Expected: each phrase appears once.

- [ ] **Step 2: Entry-point reminder check**

Run:

```bash
rg -n "Full correction has priority|role-play collapse|brevity override" skills/english-podcast-speaking-coach/SKILL.md
```

Expected: all three phrases appear.

- [ ] **Step 3: Manual dry-run prompt for role-play English**

Use this manual prompt in a fresh session or local review:

```text
Use english-podcast-speaking-coach. Lesson scenario: road trip. Start a role-play where the coach is my travel partner asking whether everything is ready. Do not reveal target answers before I attempt.
```

Expected learner-facing shape:

```text
Travel partner: Are we all set?

你想回答：差不多，车已经装好了，零食、音乐和地图也都在车里。请用英文说。
```

Failure shape:

```text
我问你：「都准备好了吗？」
```

- [ ] **Step 4: Manual dry-run prompt for full correction**

Use this learner answer:

```text
My mate and I were ready for a road trip. I made sure the car, the munchies, the music, the map, the camera and the tank are all set. I had got all our bases covered. But after a while, I wanted to make a pit stop, because I forgot a thing before we left.
```

Expected correction covers at least:

```text
mate -> partner
are all set -> were all set
I had got all our bases covered -> We had all our bases covered
after a while -> after only ten minutes on the road
I forgot a thing -> I forgot to go to the bathroom
comma before because
```

- [ ] **Step 5: Manual dry-run prompt for Chinglish**

Use this learner answer:

```text
It would protect us from new employee risks.
```

Expected correction explicitly says this is Chinglish or Chinese-like wording and suggests:

```text
It would be less risky than taking on new staff.
```

- [ ] **Step 6: Manual dry-run prompt for lesson-end summary**

Finish a short lesson and inspect the summary. Expected active expression entries include:

```text
- expression chunk
- 中文含义
- 用法
- learner sentence or corrected learner sentence
- 回忆提示
```

The summary may be concise, but it must not be only a bare bullet list of chunks.

### Task 10: Review And Commit

**Files:**

- Modify: `skills/english-podcast-speaking-coach/INTERACTION-RULES.md`
- Modify: `skills/english-podcast-speaking-coach/SKILL.md`

- [ ] **Step 1: Review diff**

Run:

```bash
git diff -- skills/english-podcast-speaking-coach/INTERACTION-RULES.md skills/english-podcast-speaking-coach/SKILL.md
```

Expected:

- Only the two expected files are modified.
- No durable state schema wording changed.
- No writeback or recovery transaction wording changed.

- [ ] **Step 2: Check status**

Run:

```bash
git status --short
```

Expected:

```text
 M skills/english-podcast-speaking-coach/INTERACTION-RULES.md
 M skills/english-podcast-speaking-coach/SKILL.md
?? .gitignore
?? docs/superpowers/plans/2026-06-17-english-podcast-speaking-coach-teaching-regression-fix.md
```

If `.gitignore` was already unrelated before this task, leave it unstaged.

- [ ] **Step 3: Commit only relevant files**

Run:

```bash
git add skills/english-podcast-speaking-coach/INTERACTION-RULES.md skills/english-podcast-speaking-coach/SKILL.md docs/superpowers/plans/2026-06-17-english-podcast-speaking-coach-teaching-regression-fix.md
git commit -m "fix: restore speaking coach teaching depth"
```

Expected:

```text
[branch <sha>] fix: restore speaking coach teaching depth
 3 files changed
```

## Self-Review

- Spec coverage: role-play English, full correction priority, Chinglish detection, lesson-chunk anchoring, retelling repair, free-expression repair, lesson-end summary richness, and manual validation are all mapped to tasks.
- Placeholder scan: no placeholder markers or unspecified implementation steps remain.
- Scope check: the plan only touches live interaction rules and entrypoint reminders. Durable state and writeback remain unchanged.
- Risk check: the main risk is overcorrecting and making replies too long. The plan preserves brief feedback for mostly natural answers and expands only for transferable errors.

## Execution Choice

Plan complete and saved to `docs/superpowers/plans/2026-06-17-english-podcast-speaking-coach-teaching-regression-fix.md`. Two execution options:

1. Subagent-Driven (recommended): dispatch a fresh subagent per task, review between tasks, fast iteration.
2. Inline Execution: execute tasks in this session using executing-plans, with checkpoints for review.
