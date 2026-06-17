# English Podcast Speaking Coach Review Followups Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** 修复 `english-podcast-speaking-coach` 教学回归修复后的遗留问题，避免示例泄露目标表达、内部分类外露，以及入口文件缺少角色扮演正向规则。

**Architecture:** 只调整 live interaction 文档和入口文件提醒，不改 durable state、writeback、recovery 和 workspace 规则。修改集中在 `INTERACTION-RULES.md` 与 `SKILL.md`，并把当前未提交的 Common Mistakes 英文化改动纳入最终 diff 一起审阅。

**Tech Stack:** Markdown skill files, `rg`, `sed`, Git, manual rule checks.

---

## Source Findings

本计划来自对 `fix: restore speaking coach teaching depth` 后的 skill review。主要发现：

- `Role-Play Language Contract` 示例使用 `Travel partner: Are we all set?`，但 `all set` 可能正是 road trip 课中的目标表达，会破坏 active recall。
- `Lesson-Language Anchoring` 要求 correction 后识别 `learner wording`、`lesson chunk`、`repair target`、`optional extension`，容易被执行为用户可见标签，和「少暴露内部流程名」规则有冲突。
- `SKILL.md` 的 Lesson Flow 只有 target hiding 和纠错提醒，缺少「角色扮演中 coach 角色台词默认英文」的正向规则。
- 当前工作区还有未提交的 Common Mistakes 英文化改动，应在修复完成前明确保留或回退。

## Scope

### In Scope

- 替换会泄露目标表达的 role-play 示例。
- 明确 `Lesson-Language Anchoring` 的分类只供内部跟踪。
- 在 `SKILL.md` 的 Lesson Flow 增加角色扮演英文台词正向规则。
- 保留或整理当前未提交的 Common Mistakes 英文化改动。
- 增加静态回归检查命令。

### Out Of Scope

- 不改 lesson flow 顺序。
- 不改 target hiding、mastery lifecycle 或 writeback schema。
- 不新增自动化测试框架。
- 不修改 `english-coach/` 学习状态数据。
- 不修改历史 plan/spec 文档，除非只更新本计划文件。

## File Structure

Modify:

- `skills/english-podcast-speaking-coach/INTERACTION-RULES.md`
  - 替换 role-play 示例中的 `Are we all set?`。
  - 给 `Lesson-Language Anchoring` 增加「internal only」约束。
  - 视情况保留当前 Common Mistakes 英文化改动。

- `skills/english-podcast-speaking-coach/SKILL.md`
  - 在 Lesson Flow 中增加角色扮演英文台词正向提醒。

Do not modify:

- `skills/english-podcast-speaking-coach/STATE-SCHEMAS.md`
- `skills/english-podcast-speaking-coach/WORKSPACE-FORMAT.md`
- `skills/english-podcast-speaking-coach/WRITEBACK-FORMAT.md`
- `skills/english-podcast-speaking-coach/RECOVERY-RULES.md`
- `english-coach/` learner workspace files

## Implementation Tasks

### Task 1: Preflight Baseline

**Files:**

- Read: `skills/english-podcast-speaking-coach/INTERACTION-RULES.md`
- Read: `skills/english-podcast-speaking-coach/SKILL.md`

- [ ] **Step 1: Check working tree state**

Run:

```bash
git status --short
```

Expected before edits:

```text
 M skills/english-podcast-speaking-coach/INTERACTION-RULES.md
?? .gitignore
?? docs/superpowers/plans/2026-06-17-english-podcast-speaking-coach-review-followups.md
```

The existing `INTERACTION-RULES.md` change is the Common Mistakes English wording change. Do not revert it unless the reviewer explicitly chooses to keep Common Mistakes bilingual.

- [ ] **Step 2: Inspect current uncommitted diff**

Run:

```bash
git diff -- skills/english-podcast-speaking-coach/INTERACTION-RULES.md
```

Expected: only Common Mistakes wording changes appear before this plan's edits.

- [ ] **Step 3: Confirm previous regression fix is already in `HEAD`**

Run:

```bash
git log --oneline --decorate -3
```

Expected includes:

```text
fix: restore speaking coach teaching depth
```

### Task 2: Remove Target Leakage From Role-Play Example

**Files:**

- Modify: `skills/english-podcast-speaking-coach/INTERACTION-RULES.md`

- [ ] **Step 1: Replace the leaking example**

In `Role-Play Language Contract`, replace:

```text
Travel partner: Are we all set?

你想回答：差不多，车已经装好了，零食、音乐和地图也都在车里。请用英文说。
```

With:

```text
Travel partner: Ready to go?

学习者想回答：差不多，车已经装好了，零食、音乐和地图也都在车里。请用英文说。
```

Rationale: `Ready to go?` preserves English in-character speech without modeling `all set` before recall.

- [ ] **Step 2: Add explicit example-safety rule**

Immediately after the example, add:

```markdown
Even examples in this document must avoid modeling a target chunk before the learner attempts it. Use neutral English role lines such as `Ready to go?`, `Can you check that?`, or `What happened?` when the target chunk is meant for the learner.
```

- [ ] **Step 3: Verify the old leaking phrase is gone**

Run:

```bash
rg -n "Travel partner: Are we all set|Ready to go\\?|Even examples in this document" skills/english-podcast-speaking-coach/INTERACTION-RULES.md
```

Expected:

```text
skills/english-podcast-speaking-coach/INTERACTION-RULES.md:<line>:Travel partner: Ready to go?
skills/english-podcast-speaking-coach/INTERACTION-RULES.md:<line>:Even examples in this document must avoid modeling a target chunk before the learner attempts it.
```

No `Travel partner: Are we all set?` match should remain.

### Task 3: Make Lesson-Language Anchoring Categories Internal

**Files:**

- Modify: `skills/english-podcast-speaking-coach/INTERACTION-RULES.md`

- [ ] **Step 1: Replace user-visible classification wording**

In `Lesson-Language Anchoring`, replace:

```markdown
After a correction, identify which parts are:
```

With:

```markdown
After a correction, internally track which parts are:
```

- [ ] **Step 2: Add learner-facing wording guard**

After the four classification bullets, add:

```markdown
Do not expose these category names to the learner. In learner-facing replies, translate the classification into ordinary coaching language such as `这句已经自然`, `这课里更常用的说法是...`, `这个点还要再换场景练一次`, or `这个只是拓展，先听过即可`.
```

- [ ] **Step 3: Verify internal-only language exists**

Run:

```bash
rg -n "internally track which parts|Do not expose these category names|ordinary coaching language" skills/english-podcast-speaking-coach/INTERACTION-RULES.md
```

Expected three matches.

### Task 4: Add Positive Role-Play Rule To Entrypoint

**Files:**

- Modify: `skills/english-podcast-speaking-coach/SKILL.md`

- [ ] **Step 1: Add Lesson Flow bullet**

In `SKILL.md`, under `During the lesson:`, insert this bullet after `Do not reveal target English in prompts.`:

```markdown
- In direct role-play, use English for the coach's in-character lines by default; use Chinese for setup, communicative intent, vocabulary explanation, and feedback.
```

- [ ] **Step 2: Verify the entrypoint reminder**

Run:

```bash
rg -n "coach's in-character lines by default|role-play collapse" skills/english-podcast-speaking-coach/SKILL.md
```

Expected:

```text
skills/english-podcast-speaking-coach/SKILL.md:<line>:- In direct role-play, use English for the coach's in-character lines by default; use Chinese for setup, communicative intent, vocabulary explanation, and feedback.
skills/english-podcast-speaking-coach/SKILL.md:<line>:- Letting role-play collapse into Chinese narration instead of English in-character speech plus Chinese intent prompts.
```

### Task 5: Decide Common Mistakes Language Consistency

**Files:**

- Modify: `skills/english-podcast-speaking-coach/INTERACTION-RULES.md`

- [ ] **Step 1: Review current Common Mistakes language**

Run:

```bash
sed -n '/^## Common Mistakes/,$p' skills/english-podcast-speaking-coach/INTERACTION-RULES.md
```

Expected: Common Mistakes mostly use English bullets, with Chinese examples only where they are literal learner-facing labels or phrases.

- [ ] **Step 2: Keep the English wording change unless bilingual comments are preferred**

Recommended final state:

```markdown
- Starting every round with the same praise line, making feedback sound templated.
- Showing internal process names, status names, or stage names directly to the learner.
- Over-explaining minor issues and crowding out the expression that matters most for the round.
- Replacing natural classroom checkpoints with status tables, or letting status symbols overpower the expressions themselves.
- Making the lesson-end summary read like a database export instead of reviewable classroom material.
```

Do not change these lines again if they already match this text.

- [ ] **Step 3: Verify only intended diff remains**

Run:

```bash
git diff -- skills/english-podcast-speaking-coach/INTERACTION-RULES.md skills/english-podcast-speaking-coach/SKILL.md
```

Expected diff contains only:

- role-play example replacement;
- example target-leakage guard;
- `internally track` anchoring wording;
- learner-facing category-name guard;
- `SKILL.md` role-play English bullet;
- current Common Mistakes English wording change.

### Task 6: Static Regression Checks

**Files:**

- Read: `skills/english-podcast-speaking-coach/INTERACTION-RULES.md`
- Read: `skills/english-podcast-speaking-coach/SKILL.md`

- [ ] **Step 1: Check for removed leaking example**

Run:

```bash
rg -n "Travel partner: Are we all set\\?" skills/english-podcast-speaking-coach/INTERACTION-RULES.md
```

Expected: no matches.

- [ ] **Step 2: Check for replacement safeguards**

Run:

```bash
rg -n "Travel partner: Ready to go\\?|Even examples in this document|internally track which parts|Do not expose these category names|coach's in-character lines by default" skills/english-podcast-speaking-coach
```

Expected matches in `INTERACTION-RULES.md` and `SKILL.md`.

- [ ] **Step 3: Check target hiding remains intact**

Run:

```bash
rg -n "Only give the Chinese meaning of target chunks|avoid speaking that target in English first|If the coach has already modeled the exact target" skills/english-podcast-speaking-coach/INTERACTION-RULES.md
```

Expected all three matches remain.

- [ ] **Step 4: Check Markdown whitespace**

Run:

```bash
git diff --check
```

Expected: no output.

### Task 7: Manual Dry-Run Review

**Files:**

- Read: `skills/english-podcast-speaking-coach/INTERACTION-RULES.md`
- Read: `skills/english-podcast-speaking-coach/SKILL.md`

- [ ] **Step 1: Role-play example review**

Inspect the `Role-Play Language Contract` example. Expected shape:

```text
Travel partner: Ready to go?

学习者想回答：差不多，车已经装好了，零食、音乐和地图也都在车里。请用英文说。
```

The example must not include `all set`.

- [ ] **Step 2: Anchoring label review**

Inspect `Lesson-Language Anchoring`. Expected behavior:

- internal categories are allowed in the skill file;
- learner-facing replies must not show `learner wording`, `lesson chunk`, `repair target`, or `optional extension` as labels;
- examples use natural Chinese coaching text.

- [ ] **Step 3: Entrypoint review**

Inspect `SKILL.md` Lesson Flow. Expected behavior:

- target hiding remains prominent;
- coach in-character English line rule appears before correction rules;
- Common Mistakes include the role-play collapse warning.

### Task 8: Commit

**Files:**

- Modify: `skills/english-podcast-speaking-coach/INTERACTION-RULES.md`
- Modify: `skills/english-podcast-speaking-coach/SKILL.md`
- Create: `docs/superpowers/plans/2026-06-17-english-podcast-speaking-coach-review-followups.md`

- [ ] **Step 1: Check final status**

Run:

```bash
git status --short
```

Expected:

```text
 M skills/english-podcast-speaking-coach/INTERACTION-RULES.md
 M skills/english-podcast-speaking-coach/SKILL.md
?? .gitignore
?? docs/superpowers/plans/2026-06-17-english-podcast-speaking-coach-review-followups.md
```

Leave `.gitignore` unstaged if it was unrelated before this plan.

- [ ] **Step 2: Stage relevant files**

Run:

```bash
git add skills/english-podcast-speaking-coach/INTERACTION-RULES.md skills/english-podcast-speaking-coach/SKILL.md docs/superpowers/plans/2026-06-17-english-podcast-speaking-coach-review-followups.md
```

- [ ] **Step 3: Commit**

Run:

```bash
git commit -m "fix: tighten speaking coach review followups"
```

Expected:

```text
[branch <sha>] fix: tighten speaking coach review followups
 3 files changed
```

## Self-Review

- Spec coverage: all review findings map to tasks: target leakage example, internal-only anchoring labels, entrypoint role-play reminder, and current uncommitted Common Mistakes diff.
- Placeholder scan: no placeholder markers or unspecified implementation steps remain.
- Scope check: only live interaction docs and entrypoint reminders are in scope; durable state and writeback files remain untouched.
- Risk check: replacing `Are we all set?` with `Ready to go?` preserves the goal of English in-character speech while protecting active recall.

## Execution Choice

Plan complete and saved to `docs/superpowers/plans/2026-06-17-english-podcast-speaking-coach-review-followups.md`. Two execution options:

1. Subagent-Driven (recommended): dispatch a fresh subagent per task, review between tasks, fast iteration.
2. Inline Execution: execute tasks in this session using executing-plans, with checkpoints for review.
