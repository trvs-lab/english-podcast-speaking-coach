# Stateful English Podcast Speaking Coach Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Make `english-podcast-speaking-coach` create and reuse a visible Markdown learner workspace, preserve conservative evidence-based state across sessions, and remove pronunciation coaching from the skill.

**Architecture:** Keep the first implementation inside the existing Markdown skill file because this project currently has no runtime code, package manifest, or test harness. The skill becomes the state machine: it defines workspace discovery, startup reading, durable schemas, lesson writeback, replay, conflict handling, privacy rules, and validation behavior that an agent follows during coaching sessions.

**Tech Stack:** Markdown skill instructions, `rg` shell validation, manual scenario validation, Git commits when executed inside a Git repository.

---

## Scope

Implement the design by editing `skills/english-podcast-speaking-coach/SKILL.md`. Do not add helper scripts in this pass; the spec says scripts are optional and the current project is instruction-only. Keep Phase 2 and Phase 3 behavior as explicit skill instructions for replay, index rebuild, snapshots, migration, and scale management, but do not create executable automation.

The current planning directory is not a Git repository. At execution time, run these tasks inside the real skills repository or initialize this folder before the first commit:

```bash
git status --short
```

Expected now in this planning folder:

```text
fatal: not a git repository (or any of the parent directories): .git
```

Expected before implementation commits:

```text

```

## File Structure

- Modify: `skills/english-podcast-speaking-coach/SKILL.md`
  - Owns the entire coach behavior.
  - Add stateful workspace discovery and initialization.
  - Add startup reading budget.
  - Add durable Markdown state schemas.
  - Add lesson-end writeback and replay rules.
  - Add conflict handling, migration, privacy, and scale-management instructions.
  - Remove all pronunciation workflow text.
- Create: `docs/superpowers/plans/2026-06-11-stateful-english-podcast-speaking-coach.md`
  - This implementation plan.

## Implementation Tasks

### Task 1: Add Stateful Overview And Session Gate

**Files:**
- Modify: `skills/english-podcast-speaking-coach/SKILL.md:8-16`
- Test: shell checks with `rg`

- [ ] **Step 1: Run the failing stateful-overview check**

Run:

```bash
rg -n "## Session Startup|english-coach/WORKSPACE.md|skill_version: stateful-v1|Pronunciation coaching is out of scope" skills/english-podcast-speaking-coach/SKILL.md
```

Expected: command exits `1` with no matches.

- [ ] **Step 2: Replace the Overview section opening**

In `skills/english-podcast-speaking-coach/SKILL.md`, replace the content from `## Overview` through the line before `## Lesson Setup` with this exact Markdown:

````markdown
## Overview

Turn passive listening material into active spoken English, with durable learner memory stored in a visible Markdown workspace.

**Core principle: Active recall comes before answer reveal.** Keep target English internal during role-play. Reveal it only when the learner asks for help, gets stuck, or has already attempted an answer.

**Mastery principle: What the coach shows is not what the learner owns.** Only active learner production or successful near-transfer repair supports durable mastery status and the Active Phrase Bank.

**State principle: Completed lesson files are the source of truth.** `state/*.md` and `phrase-bank/*.md` are materialized views optimized for startup and review. They must be recoverable from mechanically complete lesson `Writeback Summary` sections plus explicit user corrections.

Pronunciation coaching is out of scope. If the user explicitly asks how to pronounce a word or phrase, answer briefly in ordinary prose, then return to the speaking task. Do not create pronunciation state, review items, lesson-end areas, or mastery evidence.

## Session Startup

Only initialize or reuse a learner workspace when actually running a coaching session. Discussing, reviewing, or editing this skill must not create or mutate an `english-coach/` workspace.

Before starting a lesson, review, free retelling, or free-expression session:

1. Discover the workspace using the rules in `Workspace Discovery`.
2. Create the workspace only if no existing workspace is found and the user is starting a coaching session.
3. Restore the default mission if `MISSION.md` is missing, empty, or clearly damaged.
4. Read the startup set defined in `Startup Reading Set`.
5. Use existing state to mix a small number of natural review or repair targets into the session when they fit the lesson context.

Never ask the user to choose a workspace path during normal startup. If the user explicitly provides a project or course directory, create or reuse `english-coach/` inside that directory.

## Workspace Discovery

Use a visible `english-coach/` directory under the discovered learner project root.

Discovery rules:

1. Starting from the current directory, look upward through parent directories for `english-coach/WORKSPACE.md`.
2. Reuse the nearest workspace found.
3. If the user explicitly provides a project or course directory, create or reuse `english-coach/` inside that directory.
4. Otherwise, create `english-coach/` in the current directory when the user begins a speaking-coach session.
5. Stop discovery after 5 parent levels, at the filesystem root, or before the user's home directory unless the current directory itself is the home directory.

`WORKSPACE.md` identifies the workspace:

```md
# English Coach Workspace

- workspace_id: english-coach-YYYYMMDD-HHMMSS
- created_at: YYYY-MM-DD
- skill_name: english-podcast-speaking-coach
- skill_version: stateful-v1
- root_policy: nearest english-coach/WORKSPACE.md, otherwise current directory on coaching start
- discovery_limit: 5 parent levels, stop before home directory
```
````

- [ ] **Step 3: Run the stateful-overview check**

Run:

```bash
rg -n "## Session Startup|english-coach/WORKSPACE.md|skill_version: stateful-v1|Pronunciation coaching is out of scope" skills/english-podcast-speaking-coach/SKILL.md
```

Expected: output includes one match for `## Session Startup`, one match for `english-coach/WORKSPACE.md`, one match for `skill_version: stateful-v1`, and one match for `Pronunciation coaching is out of scope`.

- [ ] **Step 4: Commit**

Run:

```bash
git add skills/english-podcast-speaking-coach/SKILL.md
git commit -m "feat: add stateful coach startup"
```

Expected: commit succeeds with one modified file.

### Task 2: Add Workspace Files And Startup Reading Budget

**Files:**
- Modify: `skills/english-podcast-speaking-coach/SKILL.md`, insert after the new `Workspace Discovery` section
- Test: shell checks with `rg`

- [ ] **Step 1: Run the failing workspace-file check**

Run:

```bash
rg -n "## Workspace Files|## Default Mission|## Startup Reading Set|state/writeback-ledger.md|phrase-bank-index.md" skills/english-podcast-speaking-coach/SKILL.md
```

Expected: command exits `1` with no matches.

- [ ] **Step 2: Insert workspace file instructions**

In `skills/english-podcast-speaking-coach/SKILL.md`, insert this exact Markdown after the closing fence of the `WORKSPACE.md` example:

~~~markdown

## Workspace Files

Create this directory structure on first coaching startup:

```text
english-coach/
  WORKSPACE.md
  MISSION.md
  NOTES.md
  RESOURCES.md
  GLOSSARY.md
  state/
    CURRENT.md
    phrase-bank-index.md
    review-queue.md
    repair-bank.md
    writeback-ledger.md
  phrase-bank/
    daily-life.md
    opinions.md
    storytelling.md
    workplace.md
  lessons/
  learning-records/
  archives/
    phrase-bank/
    review-queue/
    state-snapshots/
```

File responsibilities:

- `WORKSPACE.md`: workspace identity, initialization metadata, and schema version.
- `MISSION.md`: short stable mission. Restore the default content without asking when missing, empty, or clearly damaged.
- `NOTES.md`: durable preferences and coaching constraints. This is not a session log.
- `RESOURCES.md`: recurring podcast series, transcript folders, course links, or lesson collections.
- `GLOSSARY.md`: canonical terms such as active recall, core target, near-transfer, repair target, extension expression, active phrase bank, and review queue. Do not include pronunciation terms.
- `state/CURRENT.md`: short startup summary. Rewrite it at lesson end instead of appending forever.
- `state/phrase-bank-index.md`: derived index of active and stable phrase-bank entries. Rebuild from `phrase-bank/*.md` if missing, stale, or contradictory.
- `state/review-queue.md`: review items grouped into active, dormant, and retired sections.
- `state/repair-bank.md`: repeated learner-specific repair patterns grouped into active, dormant, and retired sections.
- `state/writeback-ledger.md`: compact derived ledger of completed applied lesson writebacks, keyed by `lesson_id`.
- `phrase-bank/*.md`: source of truth for active, stable, and retired phrase-bank items.
- `lessons/*.md`: source of truth for completed lesson evidence and intended writebacks.
- `learning-records/*.md`: decision-grade learning insights only.
- `archives/`: snapshots and old state that should not stay in startup files.

## Default Mission

If `english-coach/MISSION.md` is missing, empty, or clearly damaged, write this exact default and continue:

```md
# Mission

Help a Chinese-speaking learner turn English podcast and dialogue lessons into active spoken English.

The coach should prioritize active recall, reusable spoken chunks, natural sentence repair, near-transfer practice, free retelling, free expression, and cross-lesson review.

The goal is not to memorize transcripts. The goal is to build expressions the learner can actively produce in realistic situations.

Pronunciation coaching is out of scope.
```

If the user later states a new long-term goal, update `MISSION.md` and create a learning record explaining why the mission changed. Do not ask for a mission before normal practice.

## Startup Reading Set

Read only this startup set by default:

- `english-coach/MISSION.md`
- `english-coach/NOTES.md`
- `english-coach/state/CURRENT.md`
- `english-coach/state/review-queue.md`
- `english-coach/state/repair-bank.md`

Read `state/writeback-ledger.md` only when checking whether completed lesson writebacks are unapplied.

Read `state/phrase-bank-index.md` when selecting review items, checking whether a phrase is already active, or preparing lesson-end writeback.

Read a relevant `phrase-bank/*.md` topic file only after the index points to it or when rebuilding a missing, stale, or contradictory index.

Read `RESOURCES.md` only when the user asks for a lesson without providing material, asks to continue a course or podcast series, or when the current lesson source is ambiguous.

Read older lesson files only when a specific conflict, replay, migration, or user request requires evidence.
~~~

- [ ] **Step 3: Run the workspace-file check**

Run:

```bash
rg -n "## Workspace Files|## Default Mission|## Startup Reading Set|state/writeback-ledger.md|phrase-bank-index.md" skills/english-podcast-speaking-coach/SKILL.md
```

Expected: output includes matches for all five search alternatives.

- [ ] **Step 4: Commit**

Run:

```bash
git add skills/english-podcast-speaking-coach/SKILL.md
git commit -m "feat: define coach workspace files"
```

Expected: commit succeeds with one modified file.

### Task 3: Replace Lesson Setup With Stateful Lesson Setup

**Files:**
- Modify: `skills/english-podcast-speaking-coach/SKILL.md:16-30` by section anchor
- Test: shell checks with `rg`

- [ ] **Step 1: Run the failing lesson-setup check**

Run:

```bash
rg -n "state/CURRENT.md|review-queue.md|repair-bank.md|RESOURCES.md|Do not quote long copyrighted transcript sections" skills/english-podcast-speaking-coach/SKILL.md
```

Expected before this task: output includes `Do not quote long copyrighted transcript sections`; it may include startup-file matches from Task 2, but it does not include state-aware lesson setup text under `## Lesson Setup`.

- [ ] **Step 2: Replace the Lesson Setup section**

Replace the content from `## Lesson Setup` through the line before `## Guided Preview` with this exact Markdown:

```markdown
## Lesson Setup

When the user provides a lesson, transcript, or file path:

1. Build the lesson around that material.
2. Identify the scenario, speaker goals, communicative functions, and useful phrases.
3. Select 6-10 reusable core targets and group them by speaking function.
4. Treat the material as unfamiliar unless the learner says otherwise.
5. Mix in a small number of relevant active review items from `state/review-queue.md` only when they fit naturally.
6. Use active repair patterns from `state/repair-bank.md` to create near-transfer prompts when the lesson context makes them useful.
7. Track lesson vocabulary separately from expression mastery: vocabulary the learner actively used, and vocabulary the coach explained but the learner has not used.
8. Track coach-supplied natural alternatives that the learner has not practiced as extension expressions.

When the user provides no new material, start a review, free-retelling, or free-expression session based on `state/CURRENT.md`, `state/review-queue.md`, and `RESOURCES.md` when relevant.

Prefer reusable spoken chunks over isolated vocabulary. Ignore host banter unless it teaches a useful expression or cultural point. Do not quote long copyrighted transcript sections.

Core targets participate in coverage tracking and Speed round. Every core target must receive at least one active-recall opportunity before the lesson ends. Learner-specific repair targets participate in coverage once they appear. Extension expressions remain untracked until the learner actively practices them.

Review and repair targets from prior sessions must still follow target hiding. Do not reveal their English chunks in prompts before the learner attempts, even when they are already known or marked `active`.
```

- [ ] **Step 3: Run the lesson-setup check**

Run:

```bash
rg -n "Mix in a small number of relevant active review items|Use active repair patterns|When the user provides no new material|Review and repair targets from prior sessions" skills/english-podcast-speaking-coach/SKILL.md
```

Expected: output includes one match for each phrase.

- [ ] **Step 4: Commit**

Run:

```bash
git add skills/english-podcast-speaking-coach/SKILL.md
git commit -m "feat: make lesson setup state aware"
```

Expected: commit succeeds with one modified file.

### Task 4: Add Durable State Schemas And Mastery Lifecycle

**Files:**
- Modify: `skills/english-podcast-speaking-coach/SKILL.md`, insert before `## Guided Preview`
- Test: shell checks with `rg`

- [ ] **Step 1: Run the failing schema check**

Run:

```bash
rg -n "## Durable State Schemas|attempted -> needs_review -> repaired -> active -> stable -> retired|### RQ-0007|### PB-0012|### RB-0004" skills/english-podcast-speaking-coach/SKILL.md
```

Expected: command exits `1` with no matches.

- [ ] **Step 2: Insert durable schema instructions**

Insert this exact Markdown before `## Guided Preview`:

~~~markdown

## Durable State Schemas

Use predictable Markdown fields for durable items so writebacks can be merged, replayed, and audited.

### Mastery Lifecycle

Use this lifecycle for tracked expressions and repair targets:

```text
attempted -> needs_review -> repaired -> active -> stable -> retired
```

Meanings:

- `attempted`: the learner tried to express the intended meaning.
- `needs_review`: the learner needed substantial help, copied a revealed answer, or made a reusable error.
- `repaired`: the learner fixed the issue after a cue or correction, but not yet in a new unaided context.
- `active`: the learner produced the target without seeing the target chunk, in a new but related context.
- `stable`: the learner produced the target again after spacing or in a meaningfully different context.
- `retired`: the item is no longer in active rotation because it is stable, irrelevant, or intentionally removed.

Do not mark an item `active` unless the learner produces it without seeing the target chunk in a new but related context. Copying a just-revealed correction cannot upgrade an item beyond `needs_review`; a successful prompted fix can upgrade it to `repaired`.

Do not mark `stable` from one success. `stable` requires spaced success or success in a meaningfully different context.

Every durable status upgrade must cite concrete evidence from the current lesson. If evidence is ambiguous, keep the lower status and leave the item in review. User correction overrides model inference.

### ID Allocation

Use stable ids within each namespace:

- `PB-0001` for phrase-bank items.
- `RQ-0001` for review-queue items.
- `RB-0001` for repair-bank items.

Before creating a new id, scan the relevant active state file and derived index. If `state/phrase-bank-index.md` is stale, missing, or suspect, rebuild it from `phrase-bank/*.md` before allocating a new `PB` id.

Do not allocate a new id when an existing item has the same target chunk, safe prompt, and source lesson evidence. If an id collision is found, preserve the older item, assign a new id to the newer item, and update references in the current writeback.

### Evidence Identity

Evidence lines must include a lesson id prefix:

```md
- 20260611-001:RQ-0007: failed without hint in appointment role-play
- 20260611-002:PB-0012: unaided near-transfer production in workplace scheduling
```

The lesson id plus item id plus short event description is the deduplication key. Replaying the same `Writeback Summary` must not append the same evidence line twice.

### Review Queue Item

Use this shape in `state/review-queue.md`:

```md
### RQ-0007

- type: phrase | repair | core_target | extension
- target_hidden: "I was wondering if..."
- learner_safe_prompt: "用更委婉的方式预约一个时间"
- status: needs_review | repaired | active | stable | retired
- priority: high | medium | low
- source_lesson: lessons/20260611-002-making-appointments.md
- source_phrase_id: PB-0012
- source_phrase_file: phrase-bank/workplace.md
- evidence:
  - 20260611-001:RQ-0007: failed without hint in appointment role-play
  - 20260611-002:RQ-0007: repaired after Chinese cue in near-transfer scenario
- next_review_hint: next scheduling or workplace free-expression session
```

Use three sections in `state/review-queue.md`:

- `Active Review Items`
- `Dormant Review Items`
- `Retired Review Items`

Keep active review around 1,500-2,500 English tokens or the Chinese equivalent. When active review grows too long, move lower-priority unresolved items to dormant instead of hiding them in an archive.

`target_hidden` is for internal tracking and writeback audit. The live coaching prompt must show `learner_safe_prompt` or a Chinese scenario cue, not `target_hidden`, before the learner attempts.

### Phrase Bank Item

Use this shape inside `phrase-bank/*.md`:

```md
### PB-0012: I was wondering if...

- meaning_zh: "我想委婉地问能不能..."
- use_case: polite scheduling or requests
- status: active | stable | retired
- source_lesson: lessons/20260611-002-making-appointments.md
- personal_example: "I was wondering if we could move our meeting to Friday."
- recall_prompt_zh: "委婉询问能不能把会议挪到周五"
- evidence:
  - 20260611-002:PB-0012: unaided near-transfer production in workplace scheduling
```

Only `active`, `stable`, and `retired` items belong in phrase-bank files. Items in `attempted`, `needs_review`, or `repaired` stay in `state/review-queue.md` until the learner produces them unaided in a new but related context.

Map phrase-bank items in `state/phrase-bank-index.md`:

```md
- PB-0012 | I was wondering if... | phrase-bank/workplace.md | active
```

### Repair Bank Item

Use this shape in `state/repair-bank.md`:

```md
### RB-0004: Direct Requests Sound Too Blunt

- pattern: uses direct commands where a softer request is expected
- corrected_shape: "I was wondering if..." / "Would it be possible to..."
- status: needs_review | repaired | active | stable | retired
- evidence:
  - 20260611-001:RB-0004: "Move it to Friday" in scheduling role-play
  - 20260611-002:RB-0004: repaired with Chinese cue
- next_near_transfer: ask a colleague to change a deadline
```

Use three sections in `state/repair-bank.md`:

- `Active Repair Patterns`
- `Dormant Repair Patterns`
- `Retired Repair Patterns`

Keep active repair content around 1,000-1,500 English tokens or the Chinese equivalent. Keep the most recent, repeated, or mission-relevant repair patterns active; compress older patterns and move low-priority or stable patterns into dormant or retired sections.
~~~

- [ ] **Step 3: Run the schema check**

Run:

```bash
rg -n "## Durable State Schemas|attempted -> needs_review -> repaired -> active -> stable -> retired|### RQ-0007|### PB-0012|### RB-0004" skills/english-podcast-speaking-coach/SKILL.md
```

Expected: output includes matches for all five search alternatives.

- [ ] **Step 4: Commit**

Run:

```bash
git add skills/english-podcast-speaking-coach/SKILL.md
git commit -m "feat: document durable coach state schemas"
```

Expected: commit succeeds with one modified file.

### Task 5: Update Live Lesson Flow For Stateful Review

**Files:**
- Modify: `skills/english-podcast-speaking-coach/SKILL.md:40-58`, `skills/english-podcast-speaking-coach/SKILL.md:204-237`
- Test: shell checks with `rg`

- [ ] **Step 1: Run the failing lesson-flow check**

Run:

```bash
rg -n "repaired -> active|Use review-queue items only when they fit naturally|A successful prompted fix can upgrade it to `repaired`|stable active targets" skills/english-podcast-speaking-coach/SKILL.md
```

Expected before this task: output may include lifecycle text from Task 4, but active-recall and coverage sections still use the old `unused -> attempted -> needs review -> active` wording.

- [ ] **Step 2: Replace the Active-Recall Blocks section**

Replace the content from `## Active-Recall Blocks` through the line before `## Learner-Facing Templates` with this exact Markdown:

```markdown
## Active-Recall Blocks

Each block selects 2-3 internal targets in `unused`, `attempted`, `needs_review`, or `repaired` state. Use review-queue items only when they fit naturally with the current role-play.

1. Start a new role-play scenario and play the other speaker.
2. Prompt the learner with Chinese intent and explained English vocabulary when needed; never include unexplained English words or target chunks as English answers.
3. Let the learner attempt an English reply before revealing target English.
4. If the learner is stuck, asks for help, or uses a natural help signal, advance one step at a time:

`Chinese intent -> keyword hint -> sentence frame -> complete natural expression`

5. Correct and upgrade the reply while preserving the learner's intent.
6. Use short feedback for natural answers and full problem-triggered correction for meaningful errors.
7. Optionally practice one high-value chunk with substitution after it has appeared.
8. When the learner made a reusable error, use a near-transfer scenario instead of asking them to copy the corrected sentence. The scenario may introduce new vocabulary, but it must explain new words and hide target chunks in English.
9. Continue for 2-3 turns, then show one compact coverage update.

Do not say "尽量用上" followed by the complete target sentence before the learner attempts. Do not show a keyword hint or sentence frame unless the learner needs help. Do not treat copying a just-revealed correction as `active`.

A successful prompted fix can upgrade an item to `repaired`. Upgrade to `active` only when the learner later produces the target without seeing it in a new but related context.
```

- [ ] **Step 3: Replace the Coverage Tracking section**

Replace the content from `## Coverage Tracking` through the line before `## Speed Round` with this exact Markdown:

~~~markdown
## Coverage Tracking

Use these internal lesson states for current-session coverage:

```text
unused -> attempted -> needs_review -> repaired -> active
```

- `unused`: selected for the lesson but not tested yet.
- `attempted`: the learner tried to express the intended meaning.
- `needs_review`: the learner needed substantial help, copied a revealed answer, or made a reusable error.
- `repaired`: the learner fixed the issue after a cue or correction, but not yet in a new unaided context.
- `active`: the learner produced the target without seeing the target chunk, in a new but related context.

Core targets always receive these states. Learner-specific repair targets enter coverage when the learner exposes a high-value reusable error. An extension expression receives a state only after the learner actively practices it.

Once practiced, an extension follows normal tracked-expression handling: `active` goes to the Active Phrase Bank and `needs_review` or `repaired` goes to the review area as a practiced extension expression. Merely seeing a correction or extension expression cannot make a chunk active.

Update states after each reply, but display coverage only when a block ends. Display the smallest reusable chunk or fixed question, not a whole retelling or long learner reply. Put the status emoji on the right:

```text
Block 2 覆盖情况

let me double-check                         ✅ active
be booked solid                             ✅ active
How does next Monday work for you?          🔁 needs_review
Can she do Tuesday?                         ✅ active
May I ask where you're calling from?        ✅ active
```
~~~

- [ ] **Step 4: Replace the Speed Round section**

Replace the content from `## Speed Round` through the line before `## Correction and Pronunciation` with this exact Markdown:

```markdown
## Speed Round

Before ending the lesson, test only:

- core targets still marked `needs_review` or `repaired`;
- core targets that have not yet received learner production;
- learner-specific repair targets created from high-value reusable errors;
- practiced extension expressions marked `needs_review` or `repaired`.

Use a new Chinese intent or scenario for each target. Natural unaided production in a new but related context may upgrade it to `active`; substantial help or failure leaves it at `needs_review`; a successful prompted fix may leave it at `repaired`.

Do not test stable active targets or unpracticed extension expressions. Ensure every core target receives an active-recall opportunity.
```

- [ ] **Step 5: Run the lesson-flow check**

Run:

```bash
rg -n "unused -> attempted -> needs_review -> repaired -> active|Use review-queue items only when they fit naturally|A successful prompted fix can upgrade an item to `repaired`|Do not test stable active targets" skills/english-podcast-speaking-coach/SKILL.md
```

Expected: output includes one match for each phrase.

- [ ] **Step 6: Commit**

Run:

```bash
git add skills/english-podcast-speaking-coach/SKILL.md
git commit -m "feat: update lesson flow for stateful review"
```

Expected: commit succeeds with one modified file.

### Task 6: Remove Pronunciation Workflow

**Files:**
- Modify: `skills/english-podcast-speaking-coach/SKILL.md:53`, `skills/english-podcast-speaking-coach/SKILL.md:62-75`, `skills/english-podcast-speaking-coach/SKILL.md:239-273`, `skills/english-podcast-speaking-coach/SKILL.md:302-327`
- Test: shell checks with `rg`

- [ ] **Step 1: Run the failing pronunciation-removal check**

Run:

```bash
rg -n "发音提示|pronunciation|stress|linking|weak|pause|重读|连读|停顿|跟读|accent" skills/english-podcast-speaking-coach/SKILL.md
```

Expected before this task: output includes pronunciation workflow matches from the old skill text and the new out-of-scope sentence.

- [ ] **Step 2: Update learner-facing template intro**

Replace this sentence:

```markdown
Keep learner-facing replies visually structured and easy to scan. Prefer short Chinese explanations, English chunks in code blocks when helpful, and clear labels such as `小点：`, `发音提示：`, and `短语替换：`.
```

with this exact sentence:

```markdown
Keep learner-facing replies visually structured and easy to scan. Prefer short Chinese explanations, English chunks in code blocks when helpful, and clear labels such as `小点：` and `短语替换：`.
```

Delete this sentence:

```markdown
Add `发音提示：` only when pronunciation is genuinely useful for the current line.
```

- [ ] **Step 3: Replace correction section**

Replace the content from `## Correction and Pronunciation` through the line before `## Lesson-End Expression Areas` with this exact Markdown:

```markdown
## Correction

Use problem-triggered correction. The depth of feedback depends on the current answer, not on a fixed guess about learner level.

- If the reply is basically natural, acknowledge it briefly, optionally polish one line, and keep moving.
- If the reply has spelling, grammar, collocation, word order, tone, lesson-chunk, information-organization, or Chinglish problems, use the full learner-facing correction template.
- Explicitly point out common low-level issues when they matter: spelling, tense, prepositions, articles, singular/plural forms, word order, and collocation.
- Compare the learner's wording with the lesson chunk or natural spoken version. Explain why the natural version works better; do not only say "more natural."
- Explain every key modification in the natural version, including corrected errors and coach-added expressions such as `instead`, `finally`, `first`, and `I learned that`.
- Explain replaced or deleted wording when the change affects naturalness, meaning, sequence, or tone.
- Correct repeated errors explicitly and add high-value repeated errors to coverage as learner-specific repair targets.
- Explain in Chinese for Chinese-context practice.
- Do not correct undefined information such as gender, identity, or background; preserve the name or use neutral wording.

If the user asks how to say or pronounce a word, answer briefly in normal prose and do not add that answer to coverage, review, phrase bank, repair bank, lesson-end areas, or writeback evidence.
```

- [ ] **Step 4: Replace pronunciation-related common mistakes**

In `## Common Mistakes`, delete every bullet that mentions pronunciation, `发音提示`, stress, linking, weak forms, pauses, or old pronunciation subheadings.

Make sure these exact bullets remain in the section:

```markdown
- Revealing target English before active recall, or requiring an exact help command.
- Using field-style preview bullets instead of a teacher-style introduction.
- Showing coverage after every reply.
- Forcing `小点：` without a transferable distinction, or correcting undefined details such as gender.
- Showing an English substitution menu before the attempt, or overusing substitution.
- Skipping free retelling, active repair, or learner choice in free expression.
- Mixing retelling scenarios or switching between narration and direct dialogue without marking it.
- Using Speed round on active targets while leaving unproduced core targets untested.
- Mixing active, unmastered core, and extension expressions in one phrase bank.
- Treating recognition, correction display, or mere repetition after answer reveal as proof that a chunk is active.
- Letting learner-facing replies collapse into dense paragraphs instead of structured teaching blocks.
- Showing whole sentences or long retellings as coverage items instead of reusable chunks.
- Putting coverage status before the chunk rather than on the right side.
- Treating a copied correction as `active` instead of requiring near-transfer production.
- Missing learner-specific repair targets created by repeated spelling, grammar, collocation, or Chinglish errors.
- Leaving English vocabulary unexplained in role-play, retelling, free-expression, or near-transfer prompts.
- Using `English（中文）` for target chunks and accidentally revealing the answer.
- Treating `active` target chunks as safe to show in prompts instead of requiring recall.
- Mixing ordinary vocabulary into coverage progress or the Active Phrase Bank.
- Forgetting to add lesson vocabulary to the separate `本课词汇` area.
- Letting practiced extension expressions fall between the Active Phrase Bank and Extension Expressions.
- Writing durable state without concrete evidence from learner production or near-transfer repair.
- Letting active review or repair files grow beyond the startup reading budget.
```

- [ ] **Step 5: Run the pronunciation-removal check**

Run:

```bash
rg -n "发音提示|stress|linking|weak|重读|连读|停顿|跟读|accent" skills/english-podcast-speaking-coach/SKILL.md
```

Expected: command exits `1` with no matches.

Then run:

```bash
rg -n "pronunciation|pronounce|pause" skills/english-podcast-speaking-coach/SKILL.md
```

Expected: output is limited to out-of-scope governance, the default mission, the glossary exclusion, and the brief user-question exception. There must be no workflow, marker, state, lesson-end, example, or mastery text for pronunciation.

- [ ] **Step 6: Commit**

Run:

```bash
git add skills/english-podcast-speaking-coach/SKILL.md
git commit -m "feat: remove pronunciation coaching workflow"
```

Expected: commit succeeds with one modified file.

### Task 7: Add Lesson-End Writeback And Materialized State Rules

**Files:**
- Modify: `skills/english-podcast-speaking-coach/SKILL.md:275-300`
- Test: shell checks with `rg`

- [ ] **Step 1: Run the failing writeback check**

Run:

```bash
rg -n "## Lesson-End Writeback|writeback_status: ready|writeback_complete: true|state/writeback-ledger.md|current_summary" skills/english-podcast-speaking-coach/SKILL.md
```

Expected before this task: command may match `state/writeback-ledger.md` from Task 2, but it does not match `## Lesson-End Writeback`, `writeback_status: ready`, or `writeback_complete: true`.

- [ ] **Step 2: Replace lesson-end expression section**

Replace the content from `## Lesson-End Expression Areas` through the line before `## Common Mistakes` with this exact Markdown:

~~~markdown
## Lesson-End Expression Areas

End with four learner-facing areas before writing durable state.

### 主动表达库 (Active Phrase Bank)

Include only expressions the learner actively produced or successfully repaired through a near-transfer scenario and then produced unaided in a new but related context. Aim for 8-12 when enough qualify; never pad with unpracticed expressions. Each entry includes:

`chunk`, Chinese meaning, use case, personal sentence or corrected personal sentence, Chinese recall prompt, and status `active`.

### 待复习核心表达、已练拓展表达与修复目标 (Core Targets and Repair Targets to Review)

List core targets, practiced extension expressions, and learner-specific repair targets that are not active after Speed round. Keep `needs_review` or `repaired` and include Chinese meaning plus a Chinese recall prompt. Do not place them in the Active Phrase Bank.

### 拓展表达 (Extension Expressions)

List useful non-core expressions the coach showed but the learner has not practiced. Do not assign mastery status and do not place them in durable review state.

### 本课词汇 (Lesson Vocabulary)

List lesson vocabulary separately from expression mastery.

- `主动用过`: vocabulary the learner used in an answer.
- `见过但未主动用`: vocabulary the coach explained but the learner did not use.

Do not assign expression mastery status to vocabulary. Do not put lesson vocabulary in coverage progress, the Active Phrase Bank, or Extension Expressions. If a vocabulary-looking phrase is also a target chunk, such as `with a side of`, track it as a target chunk instead of ordinary vocabulary.

## Lesson-End Writeback

At the end of each completed lesson:

1. Prepare the complete lesson file content, including lesson evidence and `Writeback Summary`.
2. Write the lesson file under `english-coach/lessons/` before mutating any state file.
3. Verify the lesson file contains a mechanically complete `Writeback Summary`.
4. Snapshot state files into `archives/state-snapshots/` when snapshot criteria are met.
5. Apply the writeback summary idempotently in this order:
   - update `phrase-bank/*.md`;
   - rebuild or update `state/phrase-bank-index.md`;
   - update `state/review-queue.md`;
   - update `state/repair-bank.md`;
   - create justified `learning-records/*.md`;
   - rewrite `state/CURRENT.md` with `last_writeback_lesson_id`;
   - append the lesson as `applied` in `state/writeback-ledger.md`.
6. Move review overflow from active to dormant before archiving retired history.

A replayable lesson file must have a mechanically complete `Writeback Summary`. A partial lesson file is evidence for a human reader but must not be used for automatic reconciliation until the writeback summary is complete.

A complete `Writeback Summary` contains:

- `writeback_status: ready`
- `lesson_id`
- `phrase_bank_updates`
- `review_queue_updates`
- `repair_bank_updates`
- `learning_records`
- `current_summary`
- final field `writeback_complete: true`

`current_summary` must always contain the complete intended `CURRENT.md` values after the lesson, even if nothing changed. Do not write `current_summary: none`.

Use this shape:

```md
## Writeback Summary

- writeback_status: ready
- lesson_id: 20260611-001-restaurant-ordering
- phrase_bank_updates:
  - PB-0012 -> phrase-bank/workplace.md -> active
- review_queue_updates:
  - RQ-0007 -> needs_review -> priority high
- repair_bank_updates:
  - RB-0004 -> needs_review
- learning_records:
  - none
- current_summary:
  - last_writeback_lesson_id: 20260611-001-restaurant-ordering
  - last_writeback_lesson_path: lessons/20260611-001-restaurant-ordering.md
  - current_focus: polite scheduling and soft requests
  - next_session: mix one new dialogue lesson with 3 active review items
  - priority_review: RQ-0007, RB-0004
  - temporary_constraints: hide target chunks during recall prompts
- writeback_complete: true
```

## Current Summary Shape

Write `state/CURRENT.md` with this shape:

```md
# Current State

- last_writeback_lesson_id: 20260611-002-making-appointments
- last_writeback_lesson_path: lessons/20260611-002-making-appointments.md
- current_focus: polite scheduling and soft requests
- next_session: mix one new dialogue lesson with 3 active review items
- priority_review:
  - RQ-0007
  - RB-0004
- temporary_constraints: hide target chunks during recall prompts
```

Keep `CURRENT.md` under one screen when possible. Durable preferences and coaching constraints belong in `NOTES.md`; if `CURRENT.md` and `NOTES.md` conflict, `NOTES.md` is authoritative for long-term preferences.

## Learning Records

Create `learning-records/*.md` sparingly. They are decision-grade insights, not activity logs.

Create a learning record when:

- the learner demonstrates a non-trivial ability that should change future difficulty;
- the learner reveals prior knowledge that should prevent reteaching;
- a misconception is corrected;
- the mission changes.

Do not create a learning record for a phrase the coach merely explained, a one-off repair already captured in `repair-bank.md`, a normal lesson summary, or a list of covered material.
~~~

- [ ] **Step 3: Run the writeback check**

Run:

```bash
rg -n "## Lesson-End Writeback|writeback_status: ready|writeback_complete: true|state/writeback-ledger.md|current_summary|## Current Summary Shape|## Learning Records" skills/english-podcast-speaking-coach/SKILL.md
```

Expected: output includes matches for all seven search alternatives.

- [ ] **Step 4: Commit**

Run:

```bash
git add skills/english-podcast-speaking-coach/SKILL.md
git commit -m "feat: add lesson writeback rules"
```

Expected: commit succeeds with one modified file.

### Task 8: Add Recovery, Conflict, Privacy, And Migration Rules

**Files:**
- Modify: `skills/english-podcast-speaking-coach/SKILL.md`, insert before `## Common Mistakes`
- Test: shell checks with `rg`

- [ ] **Step 1: Run the failing recovery check**

Run:

```bash
rg -n "## Replay And Recovery|## Conflict Handling|## Privacy And Anonymization|## Migration Rules|Snapshot criteria" skills/english-podcast-speaking-coach/SKILL.md
```

Expected: command exits `1` with no matches.

- [ ] **Step 2: Insert recovery and governance rules**

Insert this exact Markdown before `## Common Mistakes`:

~~~markdown

## Replay And Recovery

On startup, use `state/writeback-ledger.md` to detect completed lesson files whose writeback has not been applied. If the ledger is missing, stale, or suspect, inspect completed lesson files in filename order and replay any missing mechanically complete `Writeback Summary` sections.

Replaying a writeback must be idempotent:

- Reuse ids recorded in the lesson's `Writeback Summary`.
- Update existing items with the same id.
- Do not duplicate an evidence line whose lesson id, item id, and event description already exist.
- If the ledger is missing an entry but materialized state already contains the lesson's updates, replay is a no-op and then appends the missing `applied` ledger entry.

The ledger key is `lesson_id`, not filename alone. The path is a locator that may change if the user renames a file.

Snapshot criteria:

- before replaying reconciliation;
- before changing more than 5 durable items in one lesson;
- before moving more than 5 review items between active, dormant, and retired;
- at least every 5 completed lessons if state has changed.

## Conflict Handling

Use evidence quality rather than file order:

- If an expression appears as both active and needs-review, prefer the most recent high-quality learner evidence.
- Prefer recent unaided production over older evidence.
- Prefer repeated failures across lessons over a single success.
- Prefer successful near-transfer over exact repetition.
- Prefer explicit user correction over model inference.
- If a state file is missing, recreate the minimal template and continue.
- If a state file is malformed, preserve the original in `archives/` and create a clean replacement.
- If the active review queue exceeds the startup budget, keep recent, repeated, and mission-relevant items active; move other unresolved items to dormant.
- If the user corrects stored state, trust the user and update the relevant file.
- If old lesson evidence conflicts with `CURRENT.md`, reconcile using evidence quality rules and rewrite `CURRENT.md`.

Downgrade mastery when evidence supports it:

- `stable` -> `active`: the learner hesitates or needs minor support, but can still produce the target in a related context.
- `stable` -> `needs_review`: the learner fails unaided or makes the same reusable error again.
- `active` -> `needs_review`: the learner cannot produce the item without a hint in a related context.
- `active` -> `repaired`: the learner fails first but repairs successfully after a cue.
- `repaired` -> `needs_review`: the learner fails again or copies a revealed answer.
- any status -> `retired`: the user says the item is no longer relevant, the item is intentionally removed, or a stable item has remained stable after spaced review and no longer needs rotation.

## Privacy And Anonymization

Durable state should preserve the language pattern, not private facts.

Lightly anonymize personal examples by default when they mention real names, companies, addresses, health issues, relationship details, salary, legal matters, or sensitive workplace information.

Example:

```text
Too specific: I need to tell Alice from Company X that my manager rejected the proposal.
Durable state: I need to tell a colleague that my manager rejected the proposal.
```

If the user explicitly says not to record something, do not store it in durable state. If the language pattern is useful, store a generic replacement.

These rules apply to lesson files, `learning-records/*.md`, `NOTES.md`, state files, phrase-bank files, and `Writeback Summary` sections.

## Migration Rules

`WORKSPACE.md` includes `skill_version` so future schemas can migrate conservatively.

If `skill_version` is older than the current schema:

1. Do not rewrite all files immediately.
2. Create a state snapshot before migration when changing durable state.
3. Migrate only files needed for the current session.
4. Preserve unknown fields unless they conflict with required schema.
5. Prefer additive changes over destructive rewrites.
6. Record migration notes in `archives/state-snapshots/` when migration affects teaching behavior or state interpretation.
7. Update `WORKSPACE.md` only after the needed migration succeeds.
~~~

- [ ] **Step 3: Run the recovery check**

Run:

```bash
rg -n "## Replay And Recovery|## Conflict Handling|## Privacy And Anonymization|## Migration Rules|Snapshot criteria|Downgrade mastery" skills/english-podcast-speaking-coach/SKILL.md
```

Expected: output includes matches for all six search alternatives.

- [ ] **Step 4: Commit**

Run:

```bash
git add skills/english-podcast-speaking-coach/SKILL.md
git commit -m "feat: add coach state recovery rules"
```

Expected: commit succeeds with one modified file.

### Task 9: Final Validation Pass

**Files:**
- Validate: `skills/english-podcast-speaking-coach/SKILL.md`
- Test: shell checks and manual scenario checklist

- [ ] **Step 1: Verify all required state sections exist**

Run:

```bash
rg -n "## Session Startup|## Workspace Discovery|## Workspace Files|## Startup Reading Set|## Durable State Schemas|## Lesson-End Writeback|## Replay And Recovery|## Conflict Handling|## Privacy And Anonymization|## Migration Rules" skills/english-podcast-speaking-coach/SKILL.md
```

Expected: output includes one match for each section title.

- [ ] **Step 2: Verify removed pronunciation workflow has not returned**

Run:

```bash
rg -n "发音提示|stress|linking|weak|重读|连读|停顿|跟读|accent" skills/english-podcast-speaking-coach/SKILL.md
```

Expected: command exits `1` with no matches.

Run:

```bash
rg -n "pronunciation|pronounce|pause" skills/english-podcast-speaking-coach/SKILL.md
```

Expected: output is limited to out-of-scope governance, the default mission, the glossary exclusion, and the brief user-question exception. If any match describes a coaching workflow, marker syntax, durable state, lesson-end area, example, or mastery evidence, remove that text and rerun this step.

- [ ] **Step 3: Verify target hiding, writeback, and durable evidence rules**

Run:

```bash
rg -n "target_hidden|learner_safe_prompt|writeback_status: ready|writeback_complete: true|Every durable status upgrade must cite concrete evidence|The lesson id plus item id plus short event description is the deduplication key" skills/english-podcast-speaking-coach/SKILL.md
```

Expected: output includes matches for all six search alternatives.

- [ ] **Step 4: Verify manual workspace scenarios by reading the skill text**

Read the relevant sections and confirm each statement is true:

```text
1. First coaching run in a clean directory creates english-coach/WORKSPACE.md and starts without asking for a mission.
2. Startup from a subdirectory finds and reuses the nearest parent english-coach/WORKSPACE.md.
3. Discovery stops at the configured boundary before a broad home-directory workspace.
4. Discussing or editing the skill does not create a learner workspace.
5. Existing workspace startup reads the short startup set rather than scanning all lessons.
6. RESOURCES.md is read when the user asks to continue without providing new material.
```

Expected: each statement maps to `Session Startup`, `Workspace Discovery`, or `Startup Reading Set`.

- [ ] **Step 5: Verify manual lesson-flow scenarios by reading the skill text**

Read the relevant sections and confirm each statement is true:

```text
1. A transcript-based lesson selects core targets and can mix in relevant review items.
2. Target chunks are hidden in prompts, including known review items.
3. target_hidden and learner_safe_prompt remain separate in review queue entries.
4. Phrase-bank rejects attempted, needs_review, and repaired items until active.
5. User-sensitive examples are anonymized before durable writeback.
6. A user question about how to say a word receives only a brief answer and no durable state.
```

Expected: each statement maps to `Lesson Setup`, `Prompt Leakage Guard`, `Durable State Schemas`, `Privacy And Anonymization`, or `Correction`.

- [ ] **Step 6: Verify manual writeback scenarios by reading the skill text**

Read the relevant sections and confirm each statement is true:

```text
1. Lesson-end output writes the lesson file before mutating materialized state.
2. A replayable lesson has required writeback sections, complete current_summary, and final writeback_complete: true.
3. A partial lesson file is not used for automatic reconciliation.
4. Replay does not duplicate existing evidence lines.
5. Ledger lag after state update can be repaired with an applied ledger entry.
6. State snapshots are required before reconciliation and larger state moves.
```

Expected: each statement maps to `Lesson-End Writeback`, `Replay And Recovery`, or `Current Summary Shape`.

- [ ] **Step 7: Commit final validation fixes**

If Step 1 through Step 6 required any text fixes, run:

```bash
git add skills/english-podcast-speaking-coach/SKILL.md
git commit -m "test: validate stateful coach instructions"
```

Expected when fixes were needed: commit succeeds with one modified file.

Expected when no fixes were needed:

```text
No commit is needed because the validation pass did not change files.
```

## Self-Review

Spec coverage:

- Workspace initialization and reuse: Task 1 and Task 2.
- Default mission: Task 2.
- Startup reading budget: Task 2.
- State schemas, ids, evidence identity, lifecycle, and conservative upgrades: Task 4.
- Stateful lesson setup, target hiding, review mixing, repair mixing, and speed round behavior: Task 3 and Task 5.
- Pronunciation removal: Task 6 and Task 9.
- Lesson-end writeback, complete `Writeback Summary`, ledger update, and `CURRENT.md`: Task 7.
- Replay, snapshots, conflict handling, downgrade rules, privacy, and migration: Task 8.
- Validation scenarios from the design: Task 9.

Placeholder scan:

- No deferred-work markers.
- No incomplete implementation markers.
- Every section edit includes exact Markdown content.

Type consistency:

- Status names are consistently `attempted`, `needs_review`, `repaired`, `active`, `stable`, and `retired`.
- Durable ids are consistently `PB-0001`, `RQ-0001`, and `RB-0001`.
- Lesson replay identity is consistently `lesson_id`.
- Review prompt fields are consistently `target_hidden` and `learner_safe_prompt`.
- Current summary keys are consistently `last_writeback_lesson_id`, `last_writeback_lesson_path`, `current_focus`, `next_session`, `priority_review`, and `temporary_constraints`.
