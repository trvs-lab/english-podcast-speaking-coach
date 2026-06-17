# English Podcast Speaking Coach Execution Enhancement Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Implement the approved execution enhancements for `english-podcast-speaking-coach`: hot-path decision table, target selection rubric, layered complete correction, and maintenance-only regression scenarios.

**Architecture:** Keep the current medium-split skill architecture. Add a compact execution index and local rule refinements to `INTERACTION-RULES.md`, and create a maintenance-only `REGRESSION-SCENARIOS.md` file that is not part of ordinary coaching startup.

**Tech Stack:** Markdown skill files, `rg`, `sed`, `git diff --check`, manual regression review.

---

## Source Design

Read before implementation:

- `docs/superpowers/specs/2026-06-17-english-podcast-speaking-coach-execution-enhancement-design.md`
- `skills/english-podcast-speaking-coach/SKILL.md`
- `skills/english-podcast-speaking-coach/INTERACTION-RULES.md`

Confirmed scope:

- Add target selection rubric.
- Add layered complete correction wording.
- Add hot-path decision table.
- Add maintenance-only regression scenarios.

Confirmed exclusions:

- Do not add lesson modes, quick/standard/deep practice branches, or time-based session variants.
- Do not add pause, early-stop, or partial-lesson protocol.
- Do not change lesson flow order.
- Do not change target hiding, mastery lifecycle, writeback order, replay behavior, or durable state schema.
- Do not modify learner workspace files under `english-coach/`.

## File Structure

Modify:

- `skills/english-podcast-speaking-coach/INTERACTION-RULES.md`
  - Add `## Hot-Path Decision Table` near the top, after the opening read instruction and before `## Lesson Setup`.
  - Add `### Target Selection Rubric` inside `## Lesson Setup`, immediately after the three-way target split.
  - Strengthen `## Learner-Facing Templates` and `## Correction` so complete correction is layered by priority.

Create:

- `skills/english-podcast-speaking-coach/REGRESSION-SCENARIOS.md`
  - Maintenance-only manual regression scenarios.
  - Not read during ordinary coaching startup.

Do not modify:

- `skills/english-podcast-speaking-coach/STATE-SCHEMAS.md`
- `skills/english-podcast-speaking-coach/WORKSPACE-FORMAT.md`
- `skills/english-podcast-speaking-coach/WRITEBACK-FORMAT.md`
- `skills/english-podcast-speaking-coach/RECOVERY-RULES.md`
- `english-coach/`

## Implementation Tasks

### Task 1: Preflight Baseline

**Files:**

- Read: `docs/superpowers/specs/2026-06-17-english-podcast-speaking-coach-execution-enhancement-design.md`
- Read: `skills/english-podcast-speaking-coach/INTERACTION-RULES.md`
- Read: `skills/english-podcast-speaking-coach/SKILL.md`

- [ ] **Step 1: Check working tree state**

Run:

```bash
git status --short
```

Expected: clean output before edits. If there are unrelated user changes, inspect them and do not revert them.

- [ ] **Step 2: Verify target files and current headings**

Run:

```bash
rg -n "^## " skills/english-podcast-speaking-coach/INTERACTION-RULES.md skills/english-podcast-speaking-coach/SKILL.md
```

Expected includes:

```text
skills/english-podcast-speaking-coach/INTERACTION-RULES.md:5:## Lesson Setup
skills/english-podcast-speaking-coach/INTERACTION-RULES.md:104:## Learner-Facing Templates
skills/english-podcast-speaking-coach/INTERACTION-RULES.md:352:## Correction
skills/english-podcast-speaking-coach/SKILL.md:30:## Reference Files
skills/english-podcast-speaking-coach/SKILL.md:40:## Session Startup
```

Line numbers may differ after previous edits; headings must exist.

- [ ] **Step 3: Verify regression scenarios file does not already exist**

Run:

```bash
test ! -e skills/english-podcast-speaking-coach/REGRESSION-SCENARIOS.md
```

Expected: exit code 0. If the file already exists, read it and merge the scenario requirements instead of overwriting user content.

### Task 2: Add Hot-Path Decision Table

**Files:**

- Modify: `skills/english-podcast-speaking-coach/INTERACTION-RULES.md`

- [ ] **Step 1: Insert the decision table**

Use `apply_patch` to insert this section after the opening paragraph and before `## Lesson Setup`:

```markdown
## Hot-Path Decision Table

Use this table as a compact execution index during live coaching. It does not replace the detailed rules below.

| Situation | Coach action | Guardrail |
| --- | --- | --- |
| Learner answers naturally | Give brief concrete feedback and continue the role-play. | Do not over-explain minor polish or expose internal status names. |
| Learner is stuck or asks for help | Reveal one help level at a time: keyword hint, then sentence frame, then complete expression. | Do not jump to the complete answer unless the learner explicitly asks for it. |
| Learner makes Chinglish, lesson-chunk mismatch, information-order, tense, article, collocation, or linkage errors | Give layered complete correction, then use near-transfer for the highest-value repair. | Do not ask the learner to copy a just-revealed answer. |
| Coach role line may reveal a target chunk | Paraphrase the role line, switch roles, use Chinese only when needed, or demote the target to exposure-only. | Do not model the target chunk before the learner attempts it. |
| A block ends | Show one compact classroom checkpoint with the smallest reusable chunks. | Do not show raw coverage tables, long learner replies, or status-heavy progress reports. |
| Before lesson end | Test unresolved, repaired, or unproduced practiced targets with new Chinese intent prompts. | Do not test stable active targets or unpracticed extension expressions. |
| Lesson-end summary | Use the four learner-facing areas: active expressions, review expressions, exposure expressions, and vocabulary. | Do not mix active phrases, review items, extension expressions, and vocabulary into one bucket. |
```

- [ ] **Step 2: Verify the section appears in the hot path**

Run:

```bash
sed -n '1,70p' skills/english-podcast-speaking-coach/INTERACTION-RULES.md
```

Expected: `## Hot-Path Decision Table` appears before `## Lesson Setup`.

- [ ] **Step 3: Verify all key decision rows are present**

Run:

```bash
rg -n "Hot-Path Decision Table|Learner answers naturally|Learner is stuck|Coach role line may reveal|Before lesson end|Lesson-end summary" skills/english-podcast-speaking-coach/INTERACTION-RULES.md
```

Expected: one match for the heading and one match for each listed decision row.

- [ ] **Step 4: Check Markdown formatting**

Run:

```bash
git diff --check
```

Expected: no output.

- [ ] **Step 5: Commit the decision table**

Run:

```bash
git add skills/english-podcast-speaking-coach/INTERACTION-RULES.md
git commit -m "docs: add speaking coach hot path table"
```

Expected: commit succeeds with only `INTERACTION-RULES.md` included.

### Task 3: Add Target Selection Rubric

**Files:**

- Modify: `skills/english-podcast-speaking-coach/INTERACTION-RULES.md`

- [ ] **Step 1: Insert the rubric after the target split**

Use `apply_patch` to insert this section immediately after the three bullets for `primary core targets`, `secondary core targets`, and `exposure-only useful expressions`:

```markdown
### Target Selection Rubric

Use this rubric to choose which useful lesson expressions deserve active recall. It is a selection aid, not a scorecard; do not expose it to the learner.

- **Spoken chunk value:** prefer phrases, question shapes, response patterns, and reusable sentence frames over isolated vocabulary.
- **Communicative function:** prioritize expressions that perform a clear speaking job, such as requesting, confirming, refusing, adding information, softening, changing direction, telling a story, or expressing an opinion.
- **Transfer potential:** prefer expressions the learner can reuse in daily life, workplace, travel, opinions, storytelling, or future podcast lessons.
- **Lesson anchoring:** prefer natural lesson language that improves on likely learner wording, repairs Chinglish, or gives a more spoken version of the same intent.
- **Review and repair fit:** promote expressions that naturally combine with active review items or repair patterns from prior sessions.
- **Cognitive load:** demote expressions that require too much new vocabulary or scene knowledge before the learner can practice the chunk itself.

Classify targets this way:

- `primary core targets`: expressions that satisfy several rubric points and deserve active recall plus near-transfer in this lesson.
- `secondary core targets`: useful expressions that deserve at least one active-recall opportunity when time allows.
- `exposure-only useful expressions`: useful expressions to show or mention, but not enough practiced evidence for mastery state.

Ordinary vocabulary stays in Lesson Vocabulary. Do not promote a word or phrase to expression mastery unless it functions as a reusable spoken chunk.
```

- [ ] **Step 2: Verify rubric placement**

Run:

```bash
sed -n '15,70p' skills/english-podcast-speaking-coach/INTERACTION-RULES.md
```

Expected: `### Target Selection Rubric` appears after the target split and before `Learn broadly but grade conservatively`.

- [ ] **Step 3: Verify rubric coverage**

Run:

```bash
rg -n "Target Selection Rubric|Spoken chunk value|Communicative function|Transfer potential|Lesson anchoring|Review and repair fit|Cognitive load|Ordinary vocabulary stays in Lesson Vocabulary" skills/english-podcast-speaking-coach/INTERACTION-RULES.md
```

Expected: one match for each rubric label and the vocabulary guard.

- [ ] **Step 4: Verify no lesson mode language was added**

Run:

```bash
rg -n "quick practice|standard practice|deep practice|课时模式|快速练习|标准练习|深练" skills/english-podcast-speaking-coach/INTERACTION-RULES.md
```

Expected: no output.

- [ ] **Step 5: Check Markdown formatting**

Run:

```bash
git diff --check
```

Expected: no output.

- [ ] **Step 6: Commit the rubric**

Run:

```bash
git add skills/english-podcast-speaking-coach/INTERACTION-RULES.md
git commit -m "docs: add speaking coach target rubric"
```

Expected: commit succeeds with only `INTERACTION-RULES.md` included.

### Task 4: Strengthen Layered Complete Correction

**Files:**

- Modify: `skills/english-podcast-speaking-coach/INTERACTION-RULES.md`

- [ ] **Step 1: Replace the full-correction lead paragraph**

In `## Learner-Facing Templates`, replace the paragraph beginning with `For a meaningful spelling, grammar, collocation` with this text:

```markdown
For a meaningful spelling, grammar, collocation, Chinglish, lesson-chunk, or information-organization issue, use layered complete correction. Start with the natural version so the learner can see the intended spoken shape. If the natural version changes several learner choices, explain every change that affects meaning, naturalness, transferability, or the lesson chunk; remove changes that are only cosmetic. Brevity must not hide a teachable reason. Match the label to the correction, for example:
```

- [ ] **Step 2: Replace the post-template correction rule**

In `## Learner-Facing Templates`, replace the paragraph beginning with `Use the full template only when it teaches transferable points.` with this text:

```markdown
Use the full template only when it teaches transferable points. Do not omit important learner errors or coach-added expressions just to keep the note short. When there are many key modifications, group them by priority:

- meaning and communicative intent;
- lesson chunk or reusable spoken upgrade;
- Chinglish and information order;
- tense, article, countability, preposition, word order, and collocation;
- minor style polish.

After explaining the grouped changes, repair only the highest-value 1-2 items through near-transfer. Do not ask the learner to copy a just-revealed answer or a polished paragraph.
```

- [ ] **Step 3: Add the same priority rule to `## Correction`**

In `## Correction`, insert this paragraph after the opening sentence `Use problem-triggered correction. The depth of feedback depends on the current answer, not on a fixed guess about learner level.`:

```markdown
Complete correction should still be layered. Explain every key change that affects meaning, naturalness, transferability, or lesson-language acquisition, but group multiple issues by priority so the learner can act on the feedback. After a dense correction, choose only the highest-value 1-2 repair points for near-transfer.
```

- [ ] **Step 4: Verify layered correction wording**

Run:

```bash
rg -n "layered complete correction|meaning and communicative intent|lesson chunk or reusable spoken upgrade|highest-value 1-2|Complete correction should still be layered" skills/english-podcast-speaking-coach/INTERACTION-RULES.md
```

Expected: matches in `Learner-Facing Templates` and `Correction`.

- [ ] **Step 5: Verify the old thin-feedback failure guard remains**

Run:

```bash
rg -n "Polishing a retelling paragraph but explaining only 2-3|Treating `Keep feedback brief` as permission" skills/english-podcast-speaking-coach/INTERACTION-RULES.md
```

Expected: both existing common-mistake guards remain.

- [ ] **Step 6: Check Markdown formatting**

Run:

```bash
git diff --check
```

Expected: no output.

- [ ] **Step 7: Commit layered correction changes**

Run:

```bash
git add skills/english-podcast-speaking-coach/INTERACTION-RULES.md
git commit -m "docs: clarify layered speaking corrections"
```

Expected: commit succeeds with only `INTERACTION-RULES.md` included.

### Task 5: Add Maintenance-Only Regression Scenarios

**Files:**

- Create: `skills/english-podcast-speaking-coach/REGRESSION-SCENARIOS.md`

- [ ] **Step 1: Create the regression scenarios file**

Use `apply_patch` to create `skills/english-podcast-speaking-coach/REGRESSION-SCENARIOS.md` with this complete content:

```markdown
# Regression Scenarios

This file is maintenance-only. Read it when changing `english-podcast-speaking-coach` rules or reviewing a change to the skill. Do not read it during ordinary coaching startup, live lessons, review sessions, free retelling, or free-expression practice.

Use these scenarios as manual regression checks. Each scenario lists the setup, expected coach behavior, and failure signs. They are not complete transcripts.

## 1. Target Leakage

Setup:

- Lesson target chunk: `with a side of`.
- Scene vocabulary: `beef tacos`, `salsa`.
- Learner should order food and ask for salsa on the side.

Expected coach behavior:

- The prompt may show `beef tacos（牛肉塔可）` and `salsa（莎莎酱）`.
- The prompt must express `with a side of` only in Chinese, such as `并且想配一份`.
- The learner gets an active-recall chance before seeing the English target chunk.

Failure signs:

- Prompt says `with a side of salsa` before the learner attempts.
- Coach treats `with a side of` as ordinary vocabulary instead of a target chunk.
- Learner repetition after the leaked prompt is counted as unaided production.

## 2. Help Ladder

Setup:

- Target chunk: `How does next Monday work for you?`
- Learner replies: `不会说`.

Expected coach behavior:

- First help response gives only a keyword hint such as `work for you`.
- Second help response may give a sentence frame such as `How does ___ work for you?`.
- Complete expression appears only after another help request or an explicit answer request.
- After each help level, the coach asks the learner to try again.

Failure signs:

- Coach gives the complete expression on the first help request.
- Coach repeats the exact help cue in every prompt.
- Assisted production is marked `active`.

## 3. Role-Play Contract

Setup:

- Lesson target chunk: `all set`.
- Coach plays a travel partner.
- Learner should say they are almost ready and the car is packed.

Expected coach behavior:

- Coach role line is English by default, such as `Travel partner: Ready to go?`.
- Coach does not say `Are we all set?` before the learner attempts `all set`.
- If natural English role speech would leak the target, the coach paraphrases, switches roles, uses Chinese only when needed, or demotes the target.

Failure signs:

- Coach replaces the whole role-play with Chinese narration when no leakage requires it.
- Coach models `all set` in the role line and later counts learner repetition as unaided.
- Prompt uses `我问你：「...」` when the coach should be another speaker in the scene.

## 4. Layered Correction

Setup:

- Learner says: `I forget bought water and we all prepared, but the map not in car.`
- Intended meaning: forgot to buy water, almost everything is ready, but the map is not in the car.
- Useful lesson chunks include `forgot to`, `all set`, and `in the car`.

Expected coach behavior:

- Coach gives a natural version first.
- Coach explains every key change that affects meaning or naturalness.
- Coach groups explanations by priority, such as action pattern, lesson chunk, grammar, and article/preposition.
- Coach repairs only the highest-value 1-2 points through near-transfer.

Failure signs:

- Coach silently changes several learner choices without explanation.
- Coach explains only a small subset of meaningful errors.
- Coach asks the learner to copy the polished full sentence.

## 5. Lesson-Language Anchoring

Setup:

- Learner says a natural non-lesson version: `everything covered`.
- Lesson offers the stronger chunk: `all our bases covered`.

Expected coach behavior:

- Coach first accepts the learner's meaning if it is natural.
- Coach offers the lesson chunk as a useful upgrade.
- Coach explains why the lesson chunk is more useful or more idiomatic.
- Coach tests it once in a new but related context before counting it as active.

Failure signs:

- Coach rejects a natural learner answer only because it differs from the transcript.
- Coach adds the lesson chunk to active state without learner production.
- Coach exposes internal category names such as `lesson chunk`, `repair target`, or `optional extension` to the learner.

## 6. Free Retelling Correction

Setup:

- Learner gives a 4-5 sentence first-person retelling with tense errors, missing articles, Chinglish ordering, and a missed lesson chunk.

Expected coach behavior:

- Coach lets the learner finish before polishing.
- Coach keeps first-person retelling consistent.
- Coach groups correction by story order/linkage, tense/time, articles/countability, lesson chunks, and Chinglish replacements when those issues appear.
- Coach repairs only the highest-value 1-2 sentences through near-transfer.

Failure signs:

- Coach interrupts before the retelling is complete.
- Coach rewrites the paragraph but explains only a few of the changes.
- Coach asks the learner to copy the polished paragraph verbatim.

## 7. Coverage Checkpoint

Setup:

- A role-play block has ended after 2-3 turns.
- Some chunks are active, and one chunk needs another attempt.

Expected coach behavior:

- Coach gives one compact classroom checkpoint.
- Active items appear under a learner-facing label such as `已经能自己说出来`.
- Items needing review appear under a learner-facing label such as `还要再练一下`.
- Only smallest reusable chunks or fixed questions appear.

Failure signs:

- Coach shows raw labels such as `coverage`, `needs_review`, or `repaired` as the main learner-facing display.
- Coach shows full learner replies, long retellings, or paragraph rewrites as coverage items.
- Coach shows a coverage table after every reply.

## 8. Lesson-End Evidence

Setup:

- One expression was produced unaided in near-transfer.
- One expression was produced only after a sentence frame.
- One useful expression was shown by the coach but not practiced by the learner.
- Several scene words were explained.

Expected coach behavior:

- Unaided production goes under `这节课真正说出来的表达`.
- Sentence-frame-assisted production goes under `下次还要再练的表达` with a short reason.
- Coach-shown but unpracticed expression goes under `听过但还没练熟的表达`.
- Scene words go under `这节课碰到的词`.

Failure signs:

- Assisted production is placed in the active expression section.
- Extension expressions and vocabulary are mixed into the active phrase bank.
- Lesson-end summary is a thin list without meaning, use case, learner example, or recall prompt for important active expressions.

## 9. Writeback Evidence

Setup:

- A completed lesson file is being prepared.
- The lesson has one active phrase, one review item, and one repair pattern.

Expected coach behavior:

- The lesson file is written before state files mutate.
- `Writeback Summary` has `writeback_status: ready`, `lesson_id`, all update buckets, `current_summary`, and final `writeback_complete: true`.
- Active phrase evidence cites concrete learner production or unaided near-transfer.
- Assisted, copied, or leaked production remains `repaired` or lower.

Failure signs:

- State files mutate before the lesson file is complete.
- `current_summary` is missing or set to `none`.
- Evidence says only `practiced in lesson` without concrete learner behavior.
- The same evidence line is duplicated after replay.
```

- [ ] **Step 2: Verify maintenance-only declaration and scenario count**

Run:

```bash
rg -n "^## [0-9]+\\.|maintenance-only|Do not read it during ordinary coaching startup" skills/english-podcast-speaking-coach/REGRESSION-SCENARIOS.md
```

Expected: one maintenance-only declaration and nine numbered scenario headings.

- [ ] **Step 3: Verify startup read-set does not mention regression scenarios**

Run:

```bash
rg -n "REGRESSION-SCENARIOS|Regression Scenarios" skills/english-podcast-speaking-coach/SKILL.md skills/english-podcast-speaking-coach/WORKSPACE-FORMAT.md
```

Expected: no output.

- [ ] **Step 4: Check Markdown formatting**

Run:

```bash
git diff --check
```

Expected: no output.

- [ ] **Step 5: Commit regression scenarios**

Run:

```bash
git add skills/english-podcast-speaking-coach/REGRESSION-SCENARIOS.md
git commit -m "docs: add speaking coach regression scenarios"
```

Expected: commit succeeds with only `REGRESSION-SCENARIOS.md` included.

### Task 6: Final Scope And Regression Verification

**Files:**

- Verify: `skills/english-podcast-speaking-coach/INTERACTION-RULES.md`
- Verify: `skills/english-podcast-speaking-coach/REGRESSION-SCENARIOS.md`
- Verify: `skills/english-podcast-speaking-coach/SKILL.md`
- Verify: `skills/english-podcast-speaking-coach/WORKSPACE-FORMAT.md`

- [ ] **Step 1: Verify approved features are present**

Run:

```bash
rg -n "Hot-Path Decision Table|Target Selection Rubric|layered complete correction|Complete correction should still be layered|highest-value 1-2|Regression Scenarios|maintenance-only" skills/english-podcast-speaking-coach/INTERACTION-RULES.md skills/english-podcast-speaking-coach/REGRESSION-SCENARIOS.md
```

Expected: matches for the decision table, rubric, layered correction wording, scenario title, and maintenance-only declaration.

- [ ] **Step 2: Verify excluded features were not introduced**

Run:

```bash
rg -n "quick practice|standard practice|deep practice|课时模式|快速练习|标准练习|深练|partial lesson|early-stop protocol|pause protocol|提前结束协议|暂停协议" skills/english-podcast-speaking-coach/INTERACTION-RULES.md skills/english-podcast-speaking-coach/REGRESSION-SCENARIOS.md
```

Expected: no output.

- [ ] **Step 3: Verify schema and writeback files are untouched**

Run:

```bash
git diff --name-only HEAD~4..HEAD
```

Expected recent implementation commits only changed:

```text
skills/english-podcast-speaking-coach/INTERACTION-RULES.md
skills/english-podcast-speaking-coach/REGRESSION-SCENARIOS.md
```

If the number of implementation commits differs, run `git diff --name-only <base-before-task-2>..HEAD` using the commit before Task 2.

- [ ] **Step 4: Verify ordinary startup read-set remains unchanged**

Run:

```bash
rg -n "Read these files by need|REGRESSION-SCENARIOS|Startup Reading Set|MISSION.md|NOTES.md|state/CURRENT.md|state/review-queue.md|state/repair-bank.md" skills/english-podcast-speaking-coach/SKILL.md skills/english-podcast-speaking-coach/WORKSPACE-FORMAT.md
```

Expected:

- `SKILL.md` still lists only the runtime reference files already used by the skill.
- `WORKSPACE-FORMAT.md` startup set still lists `MISSION.md`, `NOTES.md`, `state/CURRENT.md`, `state/review-queue.md`, and `state/repair-bank.md`.
- No match for `REGRESSION-SCENARIOS`.

- [ ] **Step 5: Run final Markdown whitespace check**

Run:

```bash
git diff --check HEAD~4..HEAD
```

Expected: no output. If the number of implementation commits differs, use the same base commit chosen in Step 3.

- [ ] **Step 6: Manual review the regression scenarios against the design**

Read:

```bash
sed -n '1,260p' skills/english-podcast-speaking-coach/REGRESSION-SCENARIOS.md
```

Expected:

- It is clearly maintenance-only.
- It covers target leakage, help ladder, role-play contract, layered correction, lesson-language anchoring, free retelling correction, coverage checkpoint, lesson-end evidence, and writeback evidence.
- It does not ask ordinary coaching sessions to load the file.

- [ ] **Step 7: Commit final verification notes only if a verification fix was needed**

If Task 6 required no edits, do not create a commit.

If Task 6 required a small fix, run:

```bash
git add skills/english-podcast-speaking-coach/INTERACTION-RULES.md skills/english-podcast-speaking-coach/REGRESSION-SCENARIOS.md
git commit -m "docs: verify speaking coach execution enhancements"
```

Expected: commit succeeds only when Task 6 made a real file change.

## Implementation Completion Checklist

- `INTERACTION-RULES.md` has a compact hot-path decision table near the top.
- `Lesson Setup` has a target selection rubric and still supports broad-but-conservative learning.
- `Learner-Facing Templates` and `Correction` describe layered complete correction.
- `REGRESSION-SCENARIOS.md` exists and is maintenance-only.
- Ordinary startup read-set does not include `REGRESSION-SCENARIOS.md`.
- No lesson modes or early-stop protocol were added.
- State, workspace, writeback, and recovery reference files were not changed.
- `git diff --check` passes for the implementation commits.

## Self-Review

- Spec coverage: Tasks 2-5 implement all four approved enhancements from the design spec.
- Scope check: Task 6 verifies excluded lesson modes and early-stop protocol were not introduced.
- Consistency check: the plan keeps existing lesson flow, target hiding, mastery lifecycle, writeback order, and recovery semantics unchanged.
- Placeholder scan: the plan contains concrete file paths, exact Markdown snippets, exact commands, and expected results for every task.
