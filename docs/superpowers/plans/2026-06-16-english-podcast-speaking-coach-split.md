# English Podcast Speaking Coach Split Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Split the oversized `english-podcast-speaking-coach` skill body into a small entrypoint plus five focused reference files without changing coaching behavior.

**Architecture:** `SKILL.md` becomes the runtime entrypoint, principle summary, and reference-file router. Live coaching rules move into `INTERACTION-RULES.md`; workspace, durable state schemas, writeback format, and recovery governance move into separate reference files read only when needed.

**Tech Stack:** Markdown skill files, `rg`, `sed`, `awk`, `wc`, Git.

---

## Source Spec

Implement the accepted design:

- `docs/superpowers/specs/2026-06-16-english-podcast-speaking-coach-split-design.md`

## File Structure

Create:

- `skills/english-podcast-speaking-coach/WORKSPACE-FORMAT.md`: workspace discovery, workspace file tree, default mission, and startup reading set.
- `skills/english-podcast-speaking-coach/INTERACTION-RULES.md`: live lesson, review, retelling, free-expression, target hiding, correction, and learner-facing voice rules.
- `skills/english-podcast-speaking-coach/STATE-SCHEMAS.md`: durable item schemas, mastery lifecycle, ID allocation, evidence identity, review queue, phrase bank, and repair bank formats.
- `skills/english-podcast-speaking-coach/WRITEBACK-FORMAT.md`: lesson-end writeback transaction, writeback summary format, current summary, and learning records.
- `skills/english-podcast-speaking-coach/RECOVERY-RULES.md`: replay, conflict handling, downgrade rules, privacy, anonymization, and migration.

Modify:

- `skills/english-podcast-speaking-coach/SKILL.md`: keep frontmatter, overview, hard principles, reference-file routing, short startup flow, short lesson flow, durable state summary, lesson-end summary, and concise common mistakes.

Do not modify:

- `docs/superpowers/specs/2026-06-16-english-podcast-speaking-coach-split-design.md`
- `docs/superpowers/specs/2026-06-12-english-podcast-speaking-coach-light-split-design.md`
- existing untracked `.gitignore`

## Current Source Heading Map

Use this map from the current `SKILL.md` as the migration baseline:

```text
8-19     ## Overview
20-33    ## Session Startup
34-58    ## Workspace Discovery
59-105   ## Workspace Files
106-123  ## Default Mission
124-143  ## Startup Reading Set
144-172  ## Lesson Setup
173-310  ## Durable State Schemas
311-318  ## Guided Preview
319-332  ## Learner-Facing Voice
333-353  ## Active-Recall Blocks
354-398  ## Learner-Facing Templates
399-408  ## Help Requests
409-434  ## Comprehensible Input Guard
435-460  ## Prompt Leakage Guard
461-483  ## Phrase Substitution
484-493  ## Near-Transfer Vocabulary
494-505  ## Free Retelling
506-509  ## Free Expression
510-544  ## Coverage Tracking
545-559  ## Speed Round
560-575  ## Correction
576-609  ## Lesson-End Expression Areas
610-657  ## Lesson-End Writeback
658-679  ## Writeback Summary
680-698  ## Current Summary Shape
699-713  ## Learning Records
714-733  ## Replay And Recovery
734-757  ## Conflict Handling
758-776  ## Privacy And Anonymization
777-790  ## Migration Rules
791-819  ## Common Mistakes
```

## Implementation Tasks

### Task 1: Preflight Baseline

**Files:**

- Read: `skills/english-podcast-speaking-coach/SKILL.md`
- Read: `docs/superpowers/specs/2026-06-16-english-podcast-speaking-coach-split-design.md`

- [ ] **Step 1: Confirm only expected existing untracked files are present**

Run:

```bash
git status --short
```

Expected:

```text
?? .gitignore
```

If additional files appear, inspect them before editing. Do not stage or alter unrelated files.

- [ ] **Step 2: Rebuild the heading map**

Run:

```bash
awk '/^## /{if(title){printf "%4d-%-4d %4d  %s\n", start, NR-1, NR-start, title} title=$0; start=NR} END{if(title){printf "%4d-%-4d %4d  %s\n", start, NR, NR-start+1, title}}' skills/english-podcast-speaking-coach/SKILL.md
```

Expected headings:

```text
## Overview
## Session Startup
## Workspace Discovery
## Workspace Files
## Default Mission
## Startup Reading Set
## Lesson Setup
## Durable State Schemas
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
## Lesson-End Writeback
## Writeback Summary
## Current Summary Shape
## Learning Records
## Replay And Recovery
## Conflict Handling
## Privacy And Anonymization
## Migration Rules
## Common Mistakes
```

- [ ] **Step 3: Confirm no reference files already exist**

Run:

```bash
rg --files skills/english-podcast-speaking-coach
```

Expected before implementation:

```text
skills/english-podcast-speaking-coach/SKILL.md
```

Do not commit in this task.

### Task 2: Create Workspace And State Reference Files

**Files:**

- Create: `skills/english-podcast-speaking-coach/WORKSPACE-FORMAT.md`
- Create: `skills/english-podcast-speaking-coach/STATE-SCHEMAS.md`
- Read: `skills/english-podcast-speaking-coach/SKILL.md`

- [ ] **Step 1: Extract the workspace source sections for review**

Run:

```bash
sed -n '34,143p' skills/english-podcast-speaking-coach/SKILL.md
```

Expected extracted headings:

```text
## Workspace Discovery
## Workspace Files
## Default Mission
## Startup Reading Set
```

- [ ] **Step 2: Create `WORKSPACE-FORMAT.md`**

Use `apply_patch` to create `skills/english-podcast-speaking-coach/WORKSPACE-FORMAT.md` with this structure:

```md
# Workspace Format

Read this file during workspace discovery, workspace initialization, default mission restoration, startup read-set decisions, or workspace schema version questions.
```

After the two-line preface above, insert the complete output from Step 1, preserving every heading, bullet, fenced code block, and sentence exactly.

The pasted content must include these exact strings:

```text
Use a visible `english-coach/` directory under the discovered learner project root.
Create this directory structure on first coaching startup:
If `english-coach/MISSION.md` is missing, empty, or clearly damaged, write this exact default and continue:
Read only this startup set by default:
```

- [ ] **Step 3: Extract the durable-state source section for review**

Run:

```bash
sed -n '173,310p' skills/english-podcast-speaking-coach/SKILL.md
```

Expected extracted heading:

```text
## Durable State Schemas
```

Expected schema example headings inside the output:

```text
### Mastery Lifecycle
### ID Allocation
### Evidence Identity
### Review Queue Item
### RQ-0007
### Phrase Bank Item
### PB-0012: I was wondering if...
### Repair Bank Item
### RB-0004: Direct Requests Sound Too Blunt
```

- [ ] **Step 4: Create `STATE-SCHEMAS.md`**

Use `apply_patch` to create `skills/english-podcast-speaking-coach/STATE-SCHEMAS.md` with this structure:

```md
# State Schemas

Read this file when creating, updating, deduplicating, reconciling, grading, or merging durable review queue, phrase bank, or repair bank items.
```

After the two-line preface above, insert the complete output from Step 3, preserving every heading, bullet, fenced code block, and sentence exactly.

The pasted content must include these exact strings:

```text
attempted -> needs_review -> repaired -> active -> stable -> retired
Use stable ids within each namespace:
Evidence lines must include a lesson id prefix:
Use three sections in `state/review-queue.md`:
Only `active`, `stable`, and `retired` items belong in phrase-bank files.
Use three sections in `state/repair-bank.md`:
```

- [ ] **Step 5: Validate both new files**

Run:

```bash
rg -n "Workspace Discovery|Workspace Files|Default Mission|Startup Reading Set" skills/english-podcast-speaking-coach/WORKSPACE-FORMAT.md
```

Expected: four matches.

Run:

```bash
rg -n "Mastery Lifecycle|ID Allocation|Evidence Identity|Review Queue Item|Phrase Bank Item|Repair Bank Item|RQ-0007|PB-0012|RB-0004" skills/english-podcast-speaking-coach/STATE-SCHEMAS.md
```

Expected: nine matches or more, including the three example IDs.

- [ ] **Step 6: Commit workspace and state reference files**

Run:

```bash
git add skills/english-podcast-speaking-coach/WORKSPACE-FORMAT.md skills/english-podcast-speaking-coach/STATE-SCHEMAS.md
git commit -m "docs: 拆出口语教练工作区与状态格式"
```

Expected: commit succeeds and includes only the two new files.

### Task 3: Create Interaction Reference File

**Files:**

- Create: `skills/english-podcast-speaking-coach/INTERACTION-RULES.md`
- Read: `skills/english-podcast-speaking-coach/SKILL.md`

- [ ] **Step 1: Extract interaction source sections for review**

Run:

```bash
sed -n '144,172p' skills/english-podcast-speaking-coach/SKILL.md
sed -n '311,609p' skills/english-podcast-speaking-coach/SKILL.md
sed -n '791,819p' skills/english-podcast-speaking-coach/SKILL.md
```

Expected extracted headings:

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

- [ ] **Step 2: Create `INTERACTION-RULES.md`**

Use `apply_patch` to create `skills/english-podcast-speaking-coach/INTERACTION-RULES.md` with this structure:

```md
# Interaction Rules

Read this file before any live lesson, review, free retelling, or free-expression session. These rules control learner-facing teaching behavior and target hiding.
```

After the two-line preface above, insert the complete output from Step 1 in this order:

1. `## Lesson Setup`
2. `## Guided Preview` through `## Lesson-End Expression Areas`
3. `## Common Mistakes`

Preserve every heading, bullet, fenced code block, and sentence exactly.

The pasted content must include these exact strings:

```text
Review and repair targets from prior sessions must still follow target hiding.
Start with one short Chinese teacher-style paragraph, not field-style bullets.
Keep internal control and learner-facing expression separate.
Each block selects 2-3 internal targets
On each help request, reveal only the next hint level:
Vocabulary may appear in English with Chinese explanation, but target chunks must not appear as English answers in prompts.
Substitution is an auxiliary exercise, not the main flow.
Near-transfer practice may introduce new words to keep scenarios realistic
After the core targets are mostly covered, choose one coherent practiced scenario.
After retelling, first ask the learner to choose a real-life situation
Use these internal lesson states for current-session coverage:
Keep `Speed round` as the internal control name
Use problem-triggered correction.
End with four lesson-summary areas before writing durable state.
```

- [ ] **Step 3: Validate interaction headings**

Run:

```bash
rg -n "Lesson Setup|Guided Preview|Learner-Facing Voice|Active-Recall Blocks|Learner-Facing Templates|Help Requests|Comprehensible Input Guard|Prompt Leakage Guard|Phrase Substitution|Near-Transfer Vocabulary|Free Retelling|Free Expression|Coverage Tracking|Speed Round|Correction|Lesson-End Expression Areas|Common Mistakes" skills/english-podcast-speaking-coach/INTERACTION-RULES.md
```

Expected: one or more matches for every heading listed in the command.

- [ ] **Step 4: Validate live-coaching hard rules are present**

Run:

```bash
rg -n "target chunks must not appear as English answers|must still follow target hiding|Do not treat copying a just-revealed correction as `active`|Do not mark it `active` until the learner later produces|Do not ask the learner to copy a just-revealed answer|Do not test stable active targets" skills/english-podcast-speaking-coach/INTERACTION-RULES.md
```

Expected: at least six matches.

- [ ] **Step 5: Commit interaction reference file**

Run:

```bash
git add skills/english-podcast-speaking-coach/INTERACTION-RULES.md
git commit -m "docs: 拆出口语教练互动规则"
```

Expected: commit succeeds and includes only `INTERACTION-RULES.md`.

### Task 4: Create Writeback And Recovery Reference Files

**Files:**

- Create: `skills/english-podcast-speaking-coach/WRITEBACK-FORMAT.md`
- Create: `skills/english-podcast-speaking-coach/RECOVERY-RULES.md`
- Read: `skills/english-podcast-speaking-coach/SKILL.md`

- [ ] **Step 1: Extract writeback source sections for review**

Run:

```bash
sed -n '610,713p' skills/english-podcast-speaking-coach/SKILL.md
```

Expected extracted headings:

```text
## Lesson-End Writeback
## Writeback Summary
## Current Summary Shape
## Learning Records
```

- [ ] **Step 2: Create `WRITEBACK-FORMAT.md`**

Use `apply_patch` to create `skills/english-podcast-speaking-coach/WRITEBACK-FORMAT.md` with this structure:

```md
# Writeback Format

Read this file before writing a completed lesson file, preparing a `Writeback Summary`, applying lesson-end state updates, creating learning records, rewriting `state/CURRENT.md`, or appending `state/writeback-ledger.md`.
```

After the two-line preface above, insert the complete output from Step 1, preserving every heading, bullet, fenced code block, and sentence exactly.

The pasted content must include these exact strings:

```text
Write the lesson file under `english-coach/lessons/` before mutating any state file.
- `writeback_status: ready`
- `writeback_complete: true`
Keep `CURRENT.md` under one screen when possible.
Create `learning-records/*.md` sparingly.
```

- [ ] **Step 3: Extract recovery source sections for review**

Run:

```bash
sed -n '714,790p' skills/english-podcast-speaking-coach/SKILL.md
```

Expected extracted headings:

```text
## Replay And Recovery
## Conflict Handling
## Privacy And Anonymization
## Migration Rules
```

- [ ] **Step 4: Create `RECOVERY-RULES.md`**

Use `apply_patch` to create `skills/english-podcast-speaking-coach/RECOVERY-RULES.md` with this structure:

```md
# Recovery Rules

Read this file when replaying writebacks, resolving conflicts, handling malformed state, anonymizing durable examples, or migrating old workspace versions.
```

After the two-line preface above, insert the complete output from Step 3, preserving every heading, bullet, fenced code block, and sentence exactly.

The pasted content must include these exact strings:

```text
Replaying a writeback must be idempotent:
Use evidence quality rather than file order:
Downgrade mastery when evidence supports it:
Durable state should preserve the language pattern, not private facts.
If `skill_version` is older than the current schema:
```

- [ ] **Step 5: Validate writeback and recovery files**

Run:

```bash
rg -n "Lesson-End Writeback|Writeback Summary|Current Summary Shape|Learning Records|writeback_status: ready|writeback_complete: true" skills/english-podcast-speaking-coach/WRITEBACK-FORMAT.md
```

Expected: six matches or more.

Run:

```bash
rg -n "Replay And Recovery|Conflict Handling|Privacy And Anonymization|Migration Rules|Downgrade mastery|idempotent" skills/english-podcast-speaking-coach/RECOVERY-RULES.md
```

Expected: six matches or more.

- [ ] **Step 6: Commit writeback and recovery reference files**

Run:

```bash
git add skills/english-podcast-speaking-coach/WRITEBACK-FORMAT.md skills/english-podcast-speaking-coach/RECOVERY-RULES.md
git commit -m "docs: 拆出口语教练写回与恢复规则"
```

Expected: commit succeeds and includes only the two new files.

### Task 5: Rewrite SKILL.md As Entrypoint

**Files:**

- Modify: `skills/english-podcast-speaking-coach/SKILL.md`
- Read: `skills/english-podcast-speaking-coach/INTERACTION-RULES.md`
- Read: `skills/english-podcast-speaking-coach/WORKSPACE-FORMAT.md`
- Read: `skills/english-podcast-speaking-coach/STATE-SCHEMAS.md`
- Read: `skills/english-podcast-speaking-coach/WRITEBACK-FORMAT.md`
- Read: `skills/english-podcast-speaking-coach/RECOVERY-RULES.md`

- [ ] **Step 1: Replace `SKILL.md` with the entrypoint content**

Use `apply_patch` to replace the body of `skills/english-podcast-speaking-coach/SKILL.md` with this content:

```md
---
name: english-podcast-speaking-coach
description: Use when the user wants to practice spoken English from ESLPod, EnglishPod, podcast transcripts, dialogue lessons, "this lesson", "practice speaking", "role-play", "口语练习", or audio-derived text.
---

# English Podcast Speaking Coach

## Overview

Turn passive listening material into active spoken English, with durable learner memory stored in a visible Markdown workspace.

This file is the runtime entrypoint. It keeps the hard rules, session route, and reference-file reading policy. Detailed coaching, workspace, state, writeback, and recovery rules live in the reference files listed below.

Pronunciation coaching is out of scope. If the user explicitly asks how to pronounce a word or phrase, answer briefly in ordinary prose, then return to the speaking task. Do not create pronunciation state, review items, lesson-end areas, or mastery evidence.

## Core Principles

**Active recall comes before answer reveal.** Keep target English internal during role-play. Reveal it only when the learner asks for help, gets stuck, or has already attempted an answer.

**Target hiding applies to old and new targets.** Review items, repair targets, active chunks intentionally reused for recall, and lesson core targets must stay hidden before the learner attempts.

**Mastery evidence comes from learner production.** Only active learner production or successful near-transfer repair followed by unaided production in a new but related context supports durable mastery status and the Active Phrase Bank.

**Assistance caps the current attempt.** If the learner needed a keyword hint, sentence frame, complete answer, English menu, or prior coach model of the exact target, the current attempt can be at most `repaired`. Upgrade to `active` only after later unaided production in a new but related context.

**Completed lesson files are the source of truth.** `state/*.md` and `phrase-bank/*.md` are materialized views optimized for startup and review. They must be recoverable from mechanically complete lesson `Writeback Summary` sections plus explicit user corrections.

**Durable state protects private facts.** Preserve the language pattern, not sensitive personal details. Anonymize durable examples before writing lesson files, state files, phrase-bank files, learning records, or writeback summaries.

## Reference Files

Read these files by need:

- `INTERACTION-RULES.md`: required before any live lesson, review, free retelling, or free-expression session.
- `WORKSPACE-FORMAT.md`: read during workspace discovery, initialization, default mission restoration, or startup read-set decisions.
- `STATE-SCHEMAS.md`: read when creating, updating, deduplicating, reconciling, or grading durable review queue, phrase bank, or repair bank items.
- `WRITEBACK-FORMAT.md`: read before writing a completed lesson file, preparing a `Writeback Summary`, applying lesson-end state updates, or creating learning records.
- `RECOVERY-RULES.md`: read when replaying writebacks, resolving conflicts, handling malformed state, anonymizing durable examples, or migrating old workspace versions.

## Session Startup

Only initialize or reuse a learner workspace when actually running a coaching session. Discussing, reviewing, or editing this skill must not create or mutate an `english-coach/` workspace.

Before starting a lesson, review, free retelling, or free-expression session:

1. Read `INTERACTION-RULES.md`.
2. Read `WORKSPACE-FORMAT.md` and discover the workspace.
3. Create the workspace only if no existing workspace is found and the user is starting a coaching session.
4. Restore the default mission if `MISSION.md` is missing, empty, or clearly damaged.
5. Read the startup set defined in `WORKSPACE-FORMAT.md`.
6. Use existing state to mix a small number of natural review or repair targets into the session when they fit the lesson context.

Never ask the user to choose a workspace path during normal startup. If the user explicitly provides a project or course directory, create or reuse `english-coach/` inside that directory.

## Lesson Flow

For any live coaching session, follow the detailed rules in `INTERACTION-RULES.md`.

Default sequence:

`guided preview -> active-recall blocks -> free retelling and repair -> free expression -> precise Speed round -> lesson-end expression areas`

During the lesson:

- Build a generous internal target inventory from the lesson, but grade conservatively.
- Prompt with Chinese intent and explained vocabulary before the learner speaks.
- Do not reveal target English in prompts.
- Give help step by step: `keyword hint -> sentence frame -> complete natural expression`.
- Use problem-triggered correction: short feedback for natural answers, full correction for transferable problems.
- Use near-transfer after meaningful correction instead of asking the learner to copy a revealed answer.
- Track lesson vocabulary separately from expression mastery.
- Keep internal process labels mostly out of learner-facing replies.

## Durable State Summary

Use this lifecycle for tracked expressions and repair targets:

```text
attempted -> needs_review -> repaired -> active -> stable -> retired
```

Read `STATE-SCHEMAS.md` before creating, updating, deduplicating, or reconciling durable items.

Stable ID namespaces:

- `PB-0001` for phrase-bank items.
- `RQ-0001` for review-queue items.
- `RB-0001` for repair-bank items.

Evidence lines must cite lesson-specific learner behavior. If evidence is ambiguous, keep the lower status and leave the item in review. User correction overrides model inference.

## Lesson-End Summary

End with the learner-facing expression areas defined in `INTERACTION-RULES.md` before mutating durable state:

- `这节课真正说出来的表达`
- `下次还要再练的表达`
- `听过但还没练熟的表达`
- `这节课碰到的词`

Before any durable writeback, read `WRITEBACK-FORMAT.md`.

Write the completed lesson file under `english-coach/lessons/` before mutating state files. Apply writeback summaries idempotently. Read `RECOVERY-RULES.md` for replay, conflict handling, malformed state, privacy questions, or migration.

## Common Mistakes

- Revealing target English before active recall, including review or repair targets.
- Treating recognition, correction display, copied answers, keyword hints, sentence frames, answer reveals, or English menus as `active` evidence.
- Skipping `INTERACTION-RULES.md` before live coaching.
- Skipping `WORKSPACE-FORMAT.md` before workspace discovery or initialization.
- Writing durable state without reading `STATE-SCHEMAS.md` and `WRITEBACK-FORMAT.md`.
- Mutating state files before writing the completed lesson file.
- Mixing active phrases, unmastered core targets, extension expressions, and lesson vocabulary into one bucket.
- Letting learner-facing replies become process reports full of internal labels.
- Leaving English vocabulary unexplained in role-play, retelling, free-expression, or near-transfer prompts.
- Storing sensitive personal facts in durable examples when a generic language-pattern example would preserve the teaching value.
```

- [ ] **Step 2: Validate entrypoint routing**

Run:

```bash
rg -n "Reference Files|INTERACTION-RULES.md|WORKSPACE-FORMAT.md|STATE-SCHEMAS.md|WRITEBACK-FORMAT.md|RECOVERY-RULES.md" skills/english-podcast-speaking-coach/SKILL.md
```

Expected: all five reference file names appear.

- [ ] **Step 3: Validate full examples were removed from entrypoint**

Run:

```bash
rg -n "### RQ-0007|### PB-0012|### RB-0004|writeback_status: ready|writeback_complete: true|# English Coach Workspace|# Mission" skills/english-podcast-speaking-coach/SKILL.md
```

Expected: no matches.

- [ ] **Step 4: Validate hard principles remain in entrypoint**

Run:

```bash
rg -n "Active recall comes before answer reveal|Target hiding applies to old and new targets|Mastery evidence comes from learner production|Assistance caps the current attempt|Completed lesson files are the source of truth|Durable state protects private facts" skills/english-podcast-speaking-coach/SKILL.md
```

Expected: six matches.

- [ ] **Step 5: Commit entrypoint rewrite**

Run:

```bash
git add skills/english-podcast-speaking-coach/SKILL.md
git commit -m "docs: 精简口语教练入口文件"
```

Expected: commit succeeds and includes only `SKILL.md`.

### Task 6: Static Validation Sweep

**Files:**

- Read: `skills/english-podcast-speaking-coach/SKILL.md`
- Read: `skills/english-podcast-speaking-coach/WORKSPACE-FORMAT.md`
- Read: `skills/english-podcast-speaking-coach/INTERACTION-RULES.md`
- Read: `skills/english-podcast-speaking-coach/STATE-SCHEMAS.md`
- Read: `skills/english-podcast-speaking-coach/WRITEBACK-FORMAT.md`
- Read: `skills/english-podcast-speaking-coach/RECOVERY-RULES.md`

- [ ] **Step 1: Confirm final file set**

Run:

```bash
rg --files skills/english-podcast-speaking-coach
```

Expected:

```text
skills/english-podcast-speaking-coach/INTERACTION-RULES.md
skills/english-podcast-speaking-coach/RECOVERY-RULES.md
skills/english-podcast-speaking-coach/SKILL.md
skills/english-podcast-speaking-coach/STATE-SCHEMAS.md
skills/english-podcast-speaking-coach/WORKSPACE-FORMAT.md
skills/english-podcast-speaking-coach/WRITEBACK-FORMAT.md
```

Order may differ.

- [ ] **Step 2: Check line counts**

Run:

```bash
wc -l skills/english-podcast-speaking-coach/SKILL.md skills/english-podcast-speaking-coach/WORKSPACE-FORMAT.md skills/english-podcast-speaking-coach/INTERACTION-RULES.md skills/english-podcast-speaking-coach/STATE-SCHEMAS.md skills/english-podcast-speaking-coach/WRITEBACK-FORMAT.md skills/english-podcast-speaking-coach/RECOVERY-RULES.md
```

Expected:

- `SKILL.md` is shorter than 260 lines.
- `INTERACTION-RULES.md` is the longest reference file.
- total line count may exceed the original file because each reference file has a routing preface.

- [ ] **Step 3: Validate required headings by file**

Run:

```bash
rg -n "^## (Workspace Discovery|Workspace Files|Default Mission|Startup Reading Set)" skills/english-podcast-speaking-coach/WORKSPACE-FORMAT.md
rg -n "^## (Lesson Setup|Guided Preview|Learner-Facing Voice|Active-Recall Blocks|Learner-Facing Templates|Help Requests|Comprehensible Input Guard|Prompt Leakage Guard|Phrase Substitution|Near-Transfer Vocabulary|Free Retelling|Free Expression|Coverage Tracking|Speed Round|Correction|Lesson-End Expression Areas|Common Mistakes)" skills/english-podcast-speaking-coach/INTERACTION-RULES.md
rg -n "^## Durable State Schemas|^### (Mastery Lifecycle|ID Allocation|Evidence Identity|Review Queue Item|Phrase Bank Item|Repair Bank Item)" skills/english-podcast-speaking-coach/STATE-SCHEMAS.md
rg -n "^## (Lesson-End Writeback|Writeback Summary|Current Summary Shape|Learning Records)" skills/english-podcast-speaking-coach/WRITEBACK-FORMAT.md
rg -n "^## (Replay And Recovery|Conflict Handling|Privacy And Anonymization|Migration Rules)" skills/english-podcast-speaking-coach/RECOVERY-RULES.md
```

Expected: every command prints the headings named in its pattern.

- [ ] **Step 4: Validate core behavior strings are still present**

Run:

```bash
rg -n "target chunks must not appear as English answers|keyword hint -> sentence frame -> complete natural expression|Do not ask the learner to copy a just-revealed answer|attempted -> needs_review -> repaired -> active -> stable -> retired|Write the lesson file under `english-coach/lessons/` before mutating any state file|Replaying a writeback must be idempotent|Durable state should preserve the language pattern" skills/english-podcast-speaking-coach
```

Expected: at least seven matches across the split files.

- [ ] **Step 5: Validate pronunciation workflow stays absent**

Run:

```bash
rg -n "发音提示|重读|连读|跟读" skills/english-podcast-speaking-coach
```

Expected: no matches.

- [ ] **Step 6: Confirm no unrelated file is staged or modified**

Run:

```bash
git status --short
```

Expected after Tasks 2 through 5 commits:

```text
?? .gitignore
```

If `SKILL.md` or reference files appear, inspect the diff and either commit intended changes or fix the missed step before continuing.

### Task 7: Manual Scenario Validation

**Files:**

- Read: `skills/english-podcast-speaking-coach/SKILL.md`
- Read: `skills/english-podcast-speaking-coach/INTERACTION-RULES.md`
- Read: `skills/english-podcast-speaking-coach/WRITEBACK-FORMAT.md`

- [ ] **Step 1: Validate new lesson active recall path**

Use this sample lesson material in a scratch note or conversation, not in committed files:

```text
Restaurant dialogue:
A: Are you ready to order?
B: I think so. I will have the chicken enchiladas with a side of guacamole.
A: Would you like anything to drink?
B: Could I get an iced tea, please?
```

Expected coaching behavior:

- Reads `INTERACTION-RULES.md` before practice.
- Opens with a short Chinese teacher-style scenario.
- Explains `enchiladas（墨西哥玉米卷）` and `guacamole（牛油果酱）` if used in the prompt.
- Hides target chunks such as `with a side of` and `Could I get...`.
- Lets the learner attempt before revealing target English.

- [ ] **Step 2: Validate help request path**

Use this learner reply:

```text
给我提示
```

Expected coaching behavior:

- Reveals only the next hint level.
- Uses the order `keyword hint -> sentence frame -> complete natural expression`.
- Asks the learner to try again after each hint.
- Does not jump to the complete answer unless explicitly requested.

- [ ] **Step 3: Validate correction and near-transfer path**

Use this learner reply:

```text
I want chicken enchiladas and guacamole side.
```

Expected coaching behavior:

- Gives a natural corrected version after the attempt.
- Explains the key change around `with a side of`.
- Does not mark copied correction as `active`.
- Uses a nearby new scenario before counting durable evidence.

- [ ] **Step 4: Validate lesson-end writeback route**

Expected writeback behavior:

- Learner-facing summary uses the four Chinese section titles from `SKILL.md`.
- The agent reads `WRITEBACK-FORMAT.md` before preparing durable writeback.
- Completed lesson file is written before state files are mutated.
- Active phrase bank entries require learner production evidence.
- Unpracticed extension expressions stay outside durable review and phrase-bank state.

- [ ] **Step 5: Record validation result in the final implementation response**

The final response should state whether the static checks passed and whether manual scenario validation found any behavior drift.

### Task 8: Final Review And Optional Consolidation Commit

**Files:**

- Read: all files under `skills/english-podcast-speaking-coach/`

- [ ] **Step 1: Review final diff from the design baseline**

Run:

```bash
git log --oneline -n 6
git status --short
```

Expected:

- Recent commits include the four implementation commits from Tasks 2, 3, 4, and 5.
- Status shows only the pre-existing untracked `.gitignore`.

- [ ] **Step 2: Confirm each spec requirement maps to a file**

Checklist:

- `SKILL.md` is an entrypoint and router.
- `WORKSPACE-FORMAT.md` owns workspace startup details.
- `INTERACTION-RULES.md` owns live coaching behavior.
- `STATE-SCHEMAS.md` owns durable state formats.
- `WRITEBACK-FORMAT.md` owns writeback transaction and summary formats.
- `RECOVERY-RULES.md` owns replay, conflict, privacy, and migration.
- No coaching behavior is intentionally changed.

- [ ] **Step 3: Leave `.gitignore` untouched**

Run:

```bash
git status --short .gitignore
```

Expected:

```text
?? .gitignore
```

Do not stage `.gitignore`.

- [ ] **Step 4: Final response content**

Include:

- Files created.
- `SKILL.md` was reduced to an entrypoint.
- Static validation results.
- Manual scenario validation results.
- Commit hashes created during execution.
- Note that `.gitignore` remains untouched.
