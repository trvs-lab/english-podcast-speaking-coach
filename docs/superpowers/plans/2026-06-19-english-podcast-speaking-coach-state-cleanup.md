# English Podcast Speaking Coach State Cleanup Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Make local learner-state writeback purposeful, less noisy, and reliably reused in future lessons without adding a new human-facing dashboard file.

**Architecture:** Keep the existing state machine and core files: `lessons/` as replayable lesson evidence, `phrase-bank/` as mastered expressions, `state/review-queue.md` as unmastered expressions, and `state/repair-bank.md` as recurring error patterns. Improve the current files' writing rules, make context-fit review opportunities more reliable, and demote snapshots to high-risk recovery insurance instead of ordinary lesson output. Preserve `GLOSSARY.md`; do not create a new `DASHBOARD.md` or rewrite learner workspace data.

**Tech Stack:** Markdown skill files, `rg`, `sed`, `find`, `git diff --check`, manual regression review.

---

## Source Decisions

Use these decisions from the design discussion:

- Do not add `DASHBOARD.md` or another human-facing entry file.
- Keep `GLOSSARY.md`.
- Keep existing bottom-layer structure: `lessons/`, `phrase-bank/`, `state/review-queue.md`, `state/repair-bank.md`, `state/CURRENT.md`, `state/writeback-ledger.md`, and `state/phrase-bank-index.md`.
- Make `phrase-bank/`, `review-queue.md`, and `repair-bank.md` easier for humans to scan while retaining IDs, statuses, and evidence for replay.
- Require every new lesson to check old review or repair items for context fit. Practice old items only when they naturally fit the current lesson; do not force unrelated warm-ups.
- When review or repair items are fixed, update durable files: promote to phrase bank or stable repair status when evidence supports it; keep or downgrade when the learner forgets again.
- Reduce snapshot noise: ordinary lesson-end writeback should not create snapshots; snapshots are for replay, reconciliation, migration, malformed state repair, or large manual reorganization.
- Standardize snapshot layout as one directory per snapshot.

## Scope

### In Scope

- Update skill rules so local files have explicit learning purposes.
- Strengthen startup/new-lesson review opportunity checks.
- Clarify promotion, demotion, and removal behavior across phrase bank, review queue, and repair bank.
- Make current core state files more human-readable without adding new files.
- Change snapshot policy, layout, and retention guidance.
- Add regression scenarios for snapshots, context-fit review, and durable state transitions.
- Keep replay and idempotent writeback intact.

### Out Of Scope

- Do not modify any learner workspace under `english-coach/`.
- Do not migrate existing snapshots or delete existing learner files.
- Treat existing flat/per-lesson snapshots as legacy recovery material. Do not normalize, migrate, or delete them in this change; ignore them during ordinary startup unless lesson replay and directory snapshots are insufficient during explicit recovery.
- Do not add `DASHBOARD.md`, `MASTERED-EXPRESSIONS.md`, `REVIEW.md`, or other new learner-facing files.
- Do not remove `GLOSSARY.md`.
- Do not replace stable IDs (`PB-*`, `RQ-*`, `RB-*`) with prose-only records.
- Do not change the mastery lifecycle names.
- Do not add a database, scripts, or automatic test harness.

## File Structure

Modify:

- `skills/english-podcast-speaking-coach/SKILL.md`
  - Add high-level reminders that startup must check old review or repair items for context fit.
  - Clarify that durable files serve three learning purposes: mastered expressions, unmastered expressions, and recurring error patterns.
  - Add a compact startup integrity check for interrupted writeback before review target selection.
  - Mention snapshots only as high-risk recovery insurance.

- `skills/english-podcast-speaking-coach/WORKSPACE-FORMAT.md`
  - Clarify core file responsibilities from the learner-state perspective.
  - Keep `GLOSSARY.md`.
  - Mark archive subdirectories as optional/high-risk storage, not ordinary learning records.
  - State that future workspaces should initialize only `archives/state-snapshots/`; retired learning records stay in active files' dormant/retired sections unless an explicit recovery or migration operation says otherwise.

- `skills/english-podcast-speaking-coach/INTERACTION-RULES.md`
  - Change lesson setup from passive "mix when naturally fits" to an explicit context-fit review check.
  - Avoid forced unrelated warm-ups when no old item fits the new lesson context.
  - Require lesson-end writeback to record attempted review/repair outcomes when old items are practiced.
  - Preserve target hiding for review items and repair targets.

- `skills/english-podcast-speaking-coach/STATE-SCHEMAS.md`
  - Clarify human-readable ordering inside items: expression/pattern first, learner-safe prompt/reason next, status/evidence after.
  - Clarify promotion and demotion across `review-queue.md`, `repair-bank.md`, and `phrase-bank/*.md`.
  - Keep ID/evidence fields required for replay.

- `skills/english-podcast-speaking-coach/WRITEBACK-FORMAT.md`
  - Remove ordinary lesson-end snapshot expectation.
  - Require writebacks to include review/repair attempt outcomes, including promotion, demotion, or retention when old items are practiced.
  - Keep full replayable payloads for add/update/promote/demote/retire actions.
  - Add required replay payload fields for state transitions.

- `skills/english-podcast-speaking-coach/RECOVERY-RULES.md`
  - Rewrite snapshot criteria as high-risk only.
  - Add standardized snapshot directory layout.
  - Add retention and pruning guidance.
  - Clarify that replay from lesson files is the primary recovery mechanism.
  - Move malformed-file preservation and migration notes into standardized snapshot directories.

- `skills/english-podcast-speaking-coach/REGRESSION-SCENARIOS.md`
  - Add maintenance-only scenarios for context-fit review handling, promotion/demotion, snapshot noise control, and interrupted writeback replay.

Do not modify:

- `docs/superpowers/specs/*.md`
- Existing plan files other than this plan.
- Any `english-coach/` learner workspace.

## Acceptance Criteria

- No new learner-facing dashboard or summary file is introduced.
- `GLOSSARY.md` remains in the workspace structure.
- Ordinary lesson-end writeback no longer requires snapshot creation.
- Snapshot rules are limited to replay/reconciliation, migration, malformed state repair, and large reorganization.
- Snapshot layout is directory-only and documented.
- New workspaces initialize only `archives/state-snapshots/` as archive structure; retired learning records are not hidden in archive directories during normal operation.
- Startup includes a compact integrity check for interrupted writeback before selecting review targets.
- Startup/new lesson rules require checking old review/repair items for context fit, without forcing unrelated review into every new lesson.
- Review/repair outcomes must update durable state after practice through canonical replayable writeback actions.
- Promotion, retention, demotion, and retirement payloads contain enough fields to replay without guessing from prose.
- Human readability improves inside existing files without removing IDs, statuses, evidence, or replay fields.
- Regression scenarios cover snapshot policy, context-fit review, promotion/demotion, and no-dashboard design.

## Implementation Tasks

### Task 1: Preflight Baseline

**Files:**

- Read: `skills/english-podcast-speaking-coach/SKILL.md`
- Read: `skills/english-podcast-speaking-coach/WORKSPACE-FORMAT.md`
- Read: `skills/english-podcast-speaking-coach/INTERACTION-RULES.md`
- Read: `skills/english-podcast-speaking-coach/STATE-SCHEMAS.md`
- Read: `skills/english-podcast-speaking-coach/WRITEBACK-FORMAT.md`
- Read: `skills/english-podcast-speaking-coach/RECOVERY-RULES.md`
- Read: `skills/english-podcast-speaking-coach/REGRESSION-SCENARIOS.md`

- [ ] **Step 1: Check working tree state**

Run:

```bash
git status --short
```

Expected: only intentional plan or skill documentation changes appear. If unrelated user changes exist, inspect them and do not revert them.

- [ ] **Step 2: Confirm the current rule anchors**

Run:

```bash
rg -n "Use existing state to mix|Snapshot state files|Snapshot criteria|Keep active review|Review Queue Item|Lesson-End Expression Areas|state-snapshots|GLOSSARY.md" skills/english-podcast-speaking-coach
```

Expected matches include:

```text
skills/english-podcast-speaking-coach/SKILL.md:Use existing state to mix
skills/english-podcast-speaking-coach/WORKSPACE-FORMAT.md:GLOSSARY.md
skills/english-podcast-speaking-coach/WRITEBACK-FORMAT.md:Snapshot state files
skills/english-podcast-speaking-coach/RECOVERY-RULES.md:Snapshot criteria
skills/english-podcast-speaking-coach/STATE-SCHEMAS.md:Keep active review
```

- [ ] **Step 3: Confirm no dashboard file exists**

Run:

```bash
rg -n "DASHBOARD|MASTERED-EXPRESSIONS|ERROR-PATTERNS|REVIEW.md" skills docs/superpowers/plans
```

Expected: the command may match this plan only. It must not show an existing skill requirement to create a dashboard file.

- [ ] **Step 4: Commit**

No commit is required for read-only preflight.

### Task 2: Clarify Workspace File Purposes Without Adding Files

**Files:**

- Modify: `skills/english-podcast-speaking-coach/WORKSPACE-FORMAT.md`
- Modify: `skills/english-podcast-speaking-coach/SKILL.md`

- [ ] **Step 1: Update the entrypoint durable-state purpose**

In `SKILL.md`, after the existing core principle `Completed lesson files are the source of truth.`, insert:

```markdown
**Durable files have learner-state jobs.** `phrase-bank/*.md` stores expressions the learner can actively produce, `state/review-queue.md` stores unmastered expressions that should return, and `state/repair-bank.md` stores recurring learner-specific error patterns. Keep these files useful for both startup and human inspection: write the expression or pattern first, then the status and evidence.
```

Expected: the entrypoint names the three core learning-state files and their purpose.

- [ ] **Step 2: Update workspace directory structure**

In `WORKSPACE-FORMAT.md`, replace the `archives/` block in the first-startup tree:

```text
  archives/
    phrase-bank/
    review-queue/
    state-snapshots/
```

with:

```text
  archives/
    state-snapshots/
```

Expected: new workspaces no longer pre-create empty `archives/phrase-bank/` and `archives/review-queue/` directories.

- [ ] **Step 3: Replace workspace file responsibility bullets**

In `WORKSPACE-FORMAT.md`, replace these bullets:

```markdown
- `state/review-queue.md`: review items grouped into active, dormant, and retired sections.
- `state/repair-bank.md`: repeated learner-specific repair patterns grouped into active, dormant, and retired sections.
- `state/writeback-ledger.md`: compact derived ledger of completed applied lesson writebacks, keyed by `lesson_id`.
- `phrase-bank/*.md`: source of truth for active, stable, and retired phrase-bank items.
- `lessons/*.md`: source of truth for completed lesson evidence and intended writebacks.
- `learning-records/*.md`: decision-grade learning insights only.
- `archives/`: snapshots and old state that should not stay in startup files.
```

with:

```markdown
- `state/review-queue.md`: unmastered or recently repaired expressions that should return in future lessons, grouped into active, dormant, and retired sections.
- `state/repair-bank.md`: recurring learner-specific error patterns that should shape future prompts and corrections, grouped into active, dormant, and retired sections.
- `state/writeback-ledger.md`: compact derived ledger of completed applied lesson writebacks, keyed by `lesson_id`; this is agent bookkeeping, not a learning note.
- `phrase-bank/*.md`: expressions and reusable patterns the learner can actively produce, grouped by topic for scanning and review.
- `lessons/*.md`: source of truth for completed lesson evidence and intended writebacks.
- `learning-records/*.md`: decision-grade learning insights only.
- `archives/`: high-risk recovery material only. Ordinary completed lessons should not create archive files.
```

Expected: the workspace description distinguishes learning records from agent bookkeeping.

- [ ] **Step 4: Add archive subdirectory creation rule**

In `WORKSPACE-FORMAT.md`, after the file responsibility bullets, insert:

```markdown
Do not create empty archive subdirectories other than `archives/state-snapshots/` during normal workspace initialization. Do not create `archives/phrase-bank/` or `archives/review-queue/` for routine retired learning records; keep those records in the active files' dormant or retired sections unless an explicit recovery or migration operation requires a snapshot directory.
```

Expected: empty archive directories are no longer part of ordinary setup, and archive policy remains recovery-only.

- [ ] **Step 5: Verify workspace wording**

Run:

```bash
rg -n "Durable files have learner-state jobs|state-snapshots|Do not create empty archive subdirectories|agent bookkeeping|human inspection" skills/english-podcast-speaking-coach/SKILL.md skills/english-podcast-speaking-coach/WORKSPACE-FORMAT.md
```

Expected:

```text
skills/english-podcast-speaking-coach/SKILL.md:Durable files have learner-state jobs.
skills/english-podcast-speaking-coach/WORKSPACE-FORMAT.md:state-snapshots
skills/english-podcast-speaking-coach/WORKSPACE-FORMAT.md:agent bookkeeping
skills/english-podcast-speaking-coach/WORKSPACE-FORMAT.md:Do not create empty archive subdirectories
```

Run:

```bash
rg -n 'Create `archives/phrase-bank/`|Create `archives/review-queue/`' skills/english-podcast-speaking-coach/WORKSPACE-FORMAT.md
```

Expected: no matches; `rg` exit code `1` means success for this negative check.

- [ ] **Step 6: Commit**

Run:

```bash
git add skills/english-podcast-speaking-coach/SKILL.md skills/english-podcast-speaking-coach/WORKSPACE-FORMAT.md
git commit -m "docs: clarify coach workspace file purposes"
```

Expected: commit succeeds with two modified skill files.

### Task 3: Demote Snapshots To High-Risk Recovery Only

**Files:**

- Modify: `skills/english-podcast-speaking-coach/WRITEBACK-FORMAT.md`
- Modify: `skills/english-podcast-speaking-coach/RECOVERY-RULES.md`
- Modify: `skills/english-podcast-speaking-coach/SKILL.md`

- [ ] **Step 1: Remove ordinary writeback snapshot requirement**

In `WRITEBACK-FORMAT.md`, replace the lesson-end writeback steps:

```markdown
4. Run the lesson-end writeback self-review checklist below.
5. Snapshot state files into `archives/state-snapshots/` when snapshot criteria are met, before mutating state files.
6. Apply the writeback summary idempotently in this order:
```

with:

```markdown
4. Run the lesson-end writeback self-review checklist below.
5. Apply the writeback summary idempotently in this order:
```

Then remove the separate `Move review overflow...` step and make overflow movement part of the state mutation order before rewriting `state/CURRENT.md` and before appending `state/writeback-ledger.md`. Add the sentence: "The ledger must remain the final mutation; no review, repair, phrase-bank, index, or archive movement should happen after the ledger append."

Expected: normal lesson-end writeback no longer instructs snapshot creation.

- [ ] **Step 2: Replace the writeback snapshot self-review item**

In `WRITEBACK-FORMAT.md`, replace:

```markdown
2. **Snapshot:** snapshot criteria were checked; if changing more than 5 durable items, a state snapshot is prepared before mutation.
```

with:

```markdown
2. **Snapshot scope:** ordinary lesson writeback does not create snapshots. A snapshot is prepared only for replay/reconciliation, migration, malformed state repair, or large manual reorganization as defined in `RECOVERY-RULES.md`.
```

Expected: self-review now points to high-risk snapshot criteria.

- [ ] **Step 3: Replace recovery snapshot criteria**

In `RECOVERY-RULES.md`, replace the current `Snapshot criteria` list:

```markdown
Snapshot criteria:

- before replaying reconciliation;
- before changing more than 5 durable items in one lesson;
- before moving more than 5 review items between active, dormant, and retired;
- at least every 5 completed lessons if state has changed.
```

with:

```markdown
Snapshot criteria:

- before replaying or reconciling multiple completed lesson writebacks;
- before migrating a workspace to a newer `skill_version`;
- before repairing or replacing a malformed state, phrase-bank, ledger, or index file;
- before moving 20 or more review, repair, or phrase-bank items between active, dormant, stable, and retired sections in one maintenance pass.

Do not create state snapshots for ordinary lesson-end writeback, even when a lesson adds many phrase-bank items. Completed lesson files with mechanically complete `Writeback Summary` sections are the primary recovery mechanism.
```

Expected: the "more than 5 durable items" and "every 5 completed lessons" rules are gone.

- [ ] **Step 4: Add standard snapshot layout**

In `RECOVERY-RULES.md`, after the new snapshot criteria block, insert:

````markdown
Use one directory per snapshot:

```text
archives/state-snapshots/<reason-prefix>-YYYYMMDD-HHMMSS/
  CURRENT.md
  review-queue.md
  repair-bank.md
  phrase-bank-index.md
  writeback-ledger.md
  phrase-bank-topic-files-that-will-change.md
```

Do not mix flat snapshot filenames with directory snapshots. If only one file needs protection, still create a snapshot directory containing that file.
````

Expected: future snapshots use one directory per snapshot.

- [ ] **Step 5: Add snapshot retention guidance**

In `RECOVERY-RULES.md`, after the standard snapshot layout block, insert:

```markdown
Snapshot retention:

- Name snapshot directories with a reason prefix: `routine-`, `migration-`, `repair-`, or `manual-reorg-`.
- After a successful recovery or mutation, prune routine snapshots beyond the most recent 5.
- Keep migration snapshots until the migrated workspace has completed at least 3 later lessons without state-format issues; after that, they are eligible for pruning.
- Keep snapshots explicitly tied to user-requested repair until the user confirms the repaired state is correct.
- Never delete lesson files as part of snapshot retention.
```

Expected: long-term snapshot growth is bounded.

- [ ] **Step 6: Propagate snapshot directories to malformed-state and migration recovery**

In `RECOVERY-RULES.md`, replace:

```markdown
- If a state file is malformed, preserve the original in `archives/` and create a clean replacement.
```

with:

```markdown
- If a state file is malformed, preserve the original inside a timestamped `archives/state-snapshots/repair-.../` directory and create a clean replacement.
```

Also replace:

```markdown
6. Record migration notes in `archives/state-snapshots/` when migration affects teaching behavior or state interpretation.
```

with:

```markdown
6. Record migration notes inside the timestamped `archives/state-snapshots/migration-.../` directory when migration affects teaching behavior or state interpretation.
```

Expected: recovery and migration paths use the same one-directory-per-snapshot policy.

- [ ] **Step 7: Add startup interrupted-writeback integrity check**

In `SKILL.md`, after the startup checklist, insert:

```markdown
Before selecting review targets, run a compact writeback integrity check: compare `state/CURRENT.md` `last_writeback_lesson_id` with the latest `applied` entry in `state/writeback-ledger.md`. If the ledger is missing, stale, suspect, or mismatched with `CURRENT.md`, read `RECOVERY-RULES.md` and replay missing mechanically complete lesson writebacks before continuing. For large histories, scan `lessons/` only when this compact check is suspect.
```

Expected: interrupted writeback is repaired before future review selection trusts materialized state.

- [ ] **Step 8: Add entrypoint reminder**

In `SKILL.md`, after the lesson-end writeback sentence:

```markdown
Write the completed lesson file under `english-coach/lessons/` before mutating state files. Apply writeback summaries idempotently. Read `RECOVERY-RULES.md` for replay, conflict handling, malformed state, privacy questions, or migration.
```

append:

```markdown
Ordinary lesson writeback should rely on replayable lesson files rather than creating state snapshots.
```

Expected: the entrypoint discourages per-lesson snapshots.

- [ ] **Step 9: Verify snapshot policy**

Run:

```bash
rg -n "ordinary lesson writeback|one directory per snapshot|routine-|primary recovery mechanism|compact writeback integrity check|ledger must remain the final mutation" skills/english-podcast-speaking-coach/WRITEBACK-FORMAT.md skills/english-podcast-speaking-coach/RECOVERY-RULES.md skills/english-podcast-speaking-coach/SKILL.md
```

Expected:

```text
skills/english-podcast-speaking-coach/WRITEBACK-FORMAT.md:ordinary lesson writeback does not create snapshots
skills/english-podcast-speaking-coach/RECOVERY-RULES.md:Do not create state snapshots for ordinary lesson-end writeback
skills/english-podcast-speaking-coach/RECOVERY-RULES.md:Use one directory per snapshot
skills/english-podcast-speaking-coach/RECOVERY-RULES.md:routine-
skills/english-podcast-speaking-coach/SKILL.md:compact writeback integrity check
```

Run:

```bash
rg -n 'changing more than 5 durable|every 5 completed|preserve the original in `archives/`' skills/english-podcast-speaking-coach/WRITEBACK-FORMAT.md skills/english-podcast-speaking-coach/RECOVERY-RULES.md skills/english-podcast-speaking-coach/SKILL.md
```

Expected: no matches; `rg` exit code `1` means success for this negative check.

- [ ] **Step 10: Commit**

Run:

```bash
git add skills/english-podcast-speaking-coach/WRITEBACK-FORMAT.md skills/english-podcast-speaking-coach/RECOVERY-RULES.md skills/english-podcast-speaking-coach/SKILL.md
git commit -m "docs: reduce coach state snapshot noise"
```

Expected: commit succeeds with three modified skill files.

### Task 4: Strengthen Context-Fit Review Checks During New Lessons

**Files:**

- Modify: `skills/english-podcast-speaking-coach/SKILL.md`
- Modify: `skills/english-podcast-speaking-coach/INTERACTION-RULES.md`
- Modify: `skills/english-podcast-speaking-coach/WRITEBACK-FORMAT.md`

- [ ] **Step 1: Replace startup review wording in `SKILL.md`**

In `SKILL.md`, replace:

```markdown
6. Use existing state to mix a small number of natural review or repair targets into the session when they fit the lesson context.
```

with:

```markdown
6. Check existing state for review or repair targets that naturally fit this session. Mix a small number into the lesson only when they fit the lesson context; if none fit, continue the new lesson normally and keep the items in review for a future suitable context.
```

Expected: startup requires a review opportunity check, but does not force unrelated review into the lesson.

- [ ] **Step 2: Replace lesson setup review rules**

In `INTERACTION-RULES.md`, replace lesson setup items 5 and 6:

```markdown
5. Mix in a small number of relevant active review items from `state/review-queue.md` only when they fit naturally.
6. Use active repair patterns from `state/repair-bank.md` to create near-transfer prompts when the lesson context makes them useful.
```

with:

```markdown
5. Review `state/CURRENT.md`, `state/review-queue.md`, and `state/repair-bank.md` for up to 3 candidate review or repair targets that may fit the lesson.
6. Practice 0-2 old targets only when they naturally fit the current role-play, retelling, free expression, or Speed round. Keep target hiding intact.
7. Do not force unrelated review warm-ups into every new lesson. If no old target fits, continue the new lesson normally.
8. If a high-priority item remains due across 3 completed lessons without a natural fit, keep it visible in `current_summary` or suggest a dedicated review session.
9. Track lesson vocabulary separately from expression mastery: vocabulary the learner actively used, and vocabulary the coach explained but the learner has not used.
10. Track coach-supplied natural alternatives that the learner has not practiced as extension expressions.
```

Expected: new lessons check for context-fit review opportunities and practice old items only when useful.

- [ ] **Step 3: Add context-fit target hiding rule**

In `INTERACTION-RULES.md`, after the paragraph ending with:

```markdown
Review and repair targets from prior sessions must still follow target hiding. Do not reveal their English chunks in prompts before the learner attempts, even when they are already known or marked `active`.
```

insert:

```markdown
Context-fit review is still active recall. Prompt with Chinese meaning, situation, and explained scene vocabulary only. Do not show `target_hidden` before the learner attempts. If the learner saw the exact target in the prompt, the attempt cannot count as unaided evidence.
```

Expected: review opportunities do not weaken target hiding or active evidence.

- [ ] **Step 4: Add writeback requirement for selected review targets**

In `WRITEBACK-FORMAT.md`, after the `current_summary` completeness paragraph, insert:

```markdown
When a session practices old review or repair targets, the `Writeback Summary` must record the outcome for each practiced target:

- Record practiced outcomes inside the normal `phrase_bank_updates`, `review_queue_updates`, or `repair_bank_updates` bucket. The canonical replay action for practiced old targets remains `promote`, `retain`, `demote`, or `retire`; use `update` only for non-transition metadata, scheduling, or explicit review-overflow movement.
- Add `practice_outcome: practiced_promoted` when the learner produced the old target unaided in a new related context and durable status changed upward.
- Add `practice_outcome: practiced_retained` when the learner practiced the target but evidence was not enough to promote.
- Add `practice_outcome: practiced_downgraded` when the learner forgot or failed an item that had previously been active or stable.
- Include `previous_status`, `status`, concrete evidence, and scheduling fields for every practiced target update.
- Use `not_practiced_still_due` only as an optional `current_summary` note for high-priority items that remain due because no suitable context appeared across 3 completed lessons.
```

Expected: practiced old items create durable follow-through without requiring skip records for every unrelated lesson.

- [ ] **Step 5: Verify review trigger rules**

Run:

```bash
rg -n "naturally fit this session|Practice 0-2 old targets|Do not force unrelated review warm-ups|Context-fit review|practiced_promoted|not_practiced_still_due|practice_outcome" skills/english-podcast-speaking-coach/SKILL.md skills/english-podcast-speaking-coach/INTERACTION-RULES.md skills/english-podcast-speaking-coach/WRITEBACK-FORMAT.md
```

Expected:

```text
skills/english-podcast-speaking-coach/SKILL.md:naturally fit this session
skills/english-podcast-speaking-coach/INTERACTION-RULES.md:Practice 0-2 old targets
skills/english-podcast-speaking-coach/INTERACTION-RULES.md:Context-fit review is still active recall.
skills/english-podcast-speaking-coach/WRITEBACK-FORMAT.md:practiced_promoted
skills/english-podcast-speaking-coach/WRITEBACK-FORMAT.md:not_practiced_still_due
```

Run:

```bash
rg -n "Use existing state to mix|Select 1-2 priority" skills/english-podcast-speaking-coach/SKILL.md skills/english-podcast-speaking-coach/INTERACTION-RULES.md skills/english-podcast-speaking-coach/WRITEBACK-FORMAT.md
```

Expected: no matches; `rg` exit code `1` means success for this negative check.

- [ ] **Step 6: Commit**

Run:

```bash
git add skills/english-podcast-speaking-coach/SKILL.md skills/english-podcast-speaking-coach/INTERACTION-RULES.md skills/english-podcast-speaking-coach/WRITEBACK-FORMAT.md
git commit -m "docs: clarify context-fit speaking review"
```

Expected: commit succeeds with three modified skill files.

### Task 5: Clarify Promotion, Demotion, And Human-Readable Item Shape

**Files:**

- Modify: `skills/english-podcast-speaking-coach/STATE-SCHEMAS.md`
- Modify: `skills/english-podcast-speaking-coach/WRITEBACK-FORMAT.md`
- Modify: `skills/english-podcast-speaking-coach/INTERACTION-RULES.md`

- [ ] **Step 1: Add human-readable item rule**

In `STATE-SCHEMAS.md`, after:

```markdown
Use predictable Markdown fields for durable items so writebacks can be merged, replayed, and audited.
```

insert:

```markdown
Keep durable files scan-friendly for humans. Put the expression, pattern, learner-safe prompt, or corrected shape near the top of each item. Keep IDs, statuses, source fields, and evidence because replay depends on them, but do not make the item read like a database row before it explains the learning point.
```

Expected: state schema explicitly balances human readability and replay.

- [ ] **Step 2: Add promotion and demotion rules**

In `STATE-SCHEMAS.md`, after the `Mastery Lifecycle` evidence rules, insert:

```markdown
### Promotion And Demotion

When a review-queue expression reaches `active`, add or update the corresponding phrase-bank item, then move the review item out of `Active Review Items`. Keep the review item only as dormant or retired history unless it still needs scheduled review.

When a repair-bank pattern becomes reliable, update the repair pattern to `active` or `stable` with evidence. If it no longer needs near-term attention, move it to dormant or retired instead of keeping it in active repair.

When a previously active or stable phrase fails in a later lesson, do not write `needs_review` or `repaired` into `phrase-bank/*.md`; phrase-bank statuses remain `active`, `stable`, or `retired`. Keep the phrase-bank evidence history, add `review_reference: RQ-*` when useful, and add a new review-queue item or reactivate the existing one as the current unmastered record. Downgrade using the rules in `RECOVERY-RULES.md` and cite the new failure evidence.

Promotion, retention, demotion, and retirement must be written as explicit writeback actions. Do not rely on prose in the lesson summary to imply a state move.
```

Expected: the review loop can move both upward and downward.

- [ ] **Step 3: Update review queue item shape**

In `STATE-SCHEMAS.md`, within the `Review Queue Item` example, ensure the item fields appear in this order:

```markdown
### RQ-0007

- learner_safe_prompt: "用更委婉的方式预约一个时间"
- learning_point: "soft scheduling request without sounding abrupt"
- target_hidden: "I was wondering if..."
- status: needs_review | repaired | active | stable | retired
- priority: high | medium | low
- type: phrase | repair | core_target | extension
- source_lesson: lessons/20260611-002-making-appointments.md
- source_phrase_id: PB-0012
- source_phrase_file: phrase-bank/workplace.md
- evidence:
  - 20260611-001:RQ-0007: failed without hint in appointment role-play
  - 20260611-002:RQ-0007: repaired after Chinese cue in near-transfer scenario
- last_seen: 2026-06-11
- next_due: 2026-06-14
- attempt_count: 2
- last_outcome: repaired after Chinese cue
- next_review_hint: next scheduling or workplace free-expression session
```

Expected: the prompt and learning point appear before bookkeeping fields.

- [ ] **Step 4: Update repair bank item shape**

In `STATE-SCHEMAS.md`, within the `Repair Bank Item` example, ensure the item fields appear in this order:

```markdown
### RB-0004: Direct Requests Sound Too Blunt

- pattern: uses direct commands where a softer request is expected
- corrected_shape: "I was wondering if..." / "Would it be possible to..."
- learner_safe_prompt: "换一个场景，用更委婉的方式提出请求"
- status: needs_review | repaired | active | stable | retired
- evidence:
  - 20260611-001:RB-0004: "Move it to Friday" in scheduling role-play
  - 20260611-002:RB-0004: repaired with Chinese cue
- next_near_transfer: ask a colleague to change a deadline
```

Expected: repair pattern reads like a learning note before evidence.

- [ ] **Step 5: Add writeback action vocabulary**

In `WRITEBACK-FORMAT.md`, before the example `Use this shape:`, insert:

```markdown
Use these writeback actions:

- `add`: create a new durable item;
- `update`: update fields or evidence on an existing durable item;
- `promote`: move an item upward after sufficient unaided evidence;
- `retain`: keep an item in its current status after practice did not meet the next threshold;
- `demote`: move an item downward after later failure or renewed assistance;
- `retire`: remove an item from active rotation while preserving evidence.
```

Expected: writeback summaries can represent the whole learning loop.

- [ ] **Step 6: Add replay payload requirements and example fields**

In `WRITEBACK-FORMAT.md`, after the writeback action vocabulary, insert:

```markdown
Replay payload requirements:

- Every practiced old target must appear in a normal `phrase_bank_updates`, `review_queue_updates`, or `repair_bank_updates` entry. Do not use prose-only outcomes as the durable update.
- For `promote`, `retain`, `demote`, and `retire`, include `source_id`, `source_file`, `destination`, `previous_status`, `status`, `practice_outcome`, concrete `evidence`, and any scheduling fields affected by the action.
- If a transition touches `phrase-bank/*.md`, include `phrase_bank_index_impact: add | update | remove | unchanged`.
- Include the full durable item fields needed to reconstruct the resulting item without guessing from surrounding lesson prose.
- Include schema-required scan-friendly fields such as `learning_point` for review items and `learner_safe_prompt` for repair items whenever they apply.
```

Then update the example `Writeback Summary` so `review_queue_updates` include `learning_point`, `repair_bank_updates` include `learner_safe_prompt`, and at least one old practiced target uses a canonical transition action:

```yaml
  - id: RQ-0007
    action: promote
    practice_outcome: practiced_promoted
    source_id: RQ-0007
    source_file: state/review-queue.md
    destination: phrase-bank/workplace.md
    previous_status: repaired
    status: retired
    promoted_phrase_id: PB-0012
    phrase_bank_index_impact: add
    learning_point: soft scheduling request without sounding abrupt
    evidence:
      - 20260611-002:RQ-0007: unaided near-transfer in a new scheduling role-play
```

Expected: transition writebacks are replayable and examples contain the new human-readable fields.

- [ ] **Step 7: Add lesson-end transition requirement**

In `INTERACTION-RULES.md`, under `Lesson-End Expression Areas`, after the four-section mapping, insert:

```markdown
For old review or repair targets practiced in this session, decide the durable transition before writeback: promote, retain, demote, or retire. Learner-facing summaries can stay simple, but the writeback must preserve the explicit transition and evidence.
```

Expected: lesson-end summary and durable state update stay connected.

- [ ] **Step 8: Verify transition rules**

Run:

```bash
rg -n "scan-friendly|Promotion And Demotion|learning_point|learner_safe_prompt|promote|retain|demote|retire|Replay payload requirements|practice_outcome|phrase_bank_index_impact|decide the durable transition" skills/english-podcast-speaking-coach/STATE-SCHEMAS.md skills/english-podcast-speaking-coach/WRITEBACK-FORMAT.md skills/english-podcast-speaking-coach/INTERACTION-RULES.md
```

Expected matches appear in all three files.

- [ ] **Step 9: Commit**

Run:

```bash
git add skills/english-podcast-speaking-coach/STATE-SCHEMAS.md skills/english-podcast-speaking-coach/WRITEBACK-FORMAT.md skills/english-podcast-speaking-coach/INTERACTION-RULES.md
git commit -m "docs: clarify coach state transitions"
```

Expected: commit succeeds with three modified skill files.

### Task 6: Add Regression Scenarios For The New State Policy

**Files:**

- Modify: `skills/english-podcast-speaking-coach/REGRESSION-SCENARIOS.md`

- [ ] **Step 1: Append context-fit review scenario**

Append this section to `REGRESSION-SCENARIOS.md`:

```markdown
## 14. Context-Fit Old Review Handling

Setup:

- `state/CURRENT.md` lists `RQ-0015`, `RQ-0016`, and `RB-0002` as priority review.
- The user starts a new unrelated lesson about small talk.

Expected coach behavior:

- Coach checks old review and repair targets for context fit before or during lesson setup.
- If a target naturally fits the lesson, it is mixed into role-play, retelling, free expression, or Speed round.
- If no old target fits, coach continues the new lesson normally and keeps the items in review for a future suitable context.
- Any old target that is practiced still hides target English before the learner attempts.

Failure signs:

- Coach never checks existing review or repair state during lesson setup.
- Coach forces an unrelated warm-up that disrupts the new lesson.
- Coach reveals `target_hidden` in a review prompt.
- Coach practices an old item but fails to update its durable outcome.
```

Expected: regression file covers context-fit review without forced warm-ups.

- [ ] **Step 2: Append promotion/demotion scenario**

Append:

```markdown
## 15. Promotion And Demotion Across Durable Files

Setup:

- `RQ-0015` asks the learner to say a computer might have a virus.
- Learner produces the target unaided in a new app-troubleshooting scenario.
- Two lessons later, learner says `it might be virus` again.

Expected coach behavior:

- First success promotes or updates the review item with concrete evidence and adds or updates the phrase-bank item when active-threshold evidence is met.
- Later failure reactivates or adds a review item and cites the new failure evidence.
- The phrase-bank history is not deleted; the review-queue status and scheduling reflect the renewed weakness.

Failure signs:

- Review item remains active and unresolved after clear promotion evidence.
- Later failure is ignored because the phrase already exists in phrase bank.
- State changes are implied only in prose and not represented as writeback actions.
```

Expected: regression file covers the full learning loop.

- [ ] **Step 3: Append snapshot policy scenario**

Append:

```markdown
## 16. Snapshot Noise Control

Setup:

- A normal lesson adds 10 phrase-bank entries and 2 review items.
- The lesson file has a complete `Writeback Summary`.

Expected coach behavior:

- Coach writes the lesson file first and applies the writeback.
- Coach does not create `archives/state-snapshots/` entries for ordinary lesson writeback.
- Coach relies on the replayable lesson file for recovery.

Failure signs:

- Ordinary lesson completion creates several snapshot files.
- Snapshot filenames are flat files mixed directly under `state-snapshots/`.
- Coach treats snapshots as the primary recovery mechanism instead of lesson replay.
```

Expected: regression file covers snapshot reduction.

- [ ] **Step 4: Append no-dashboard scenario**

Append:

```markdown
## 17. No Extra Human Dashboard

Setup:

- Workspace already has `phrase-bank/`, `state/review-queue.md`, `state/repair-bank.md`, `state/CURRENT.md`, and `GLOSSARY.md`.
- User wants local files to have clearer purpose but does not want a dashboard.

Expected coach behavior:

- Coach does not create `DASHBOARD.md`, `MASTERED-EXPRESSIONS.md`, `ERROR-PATTERNS.md`, or `REVIEW.md`.
- Existing core files become clearer through item ordering and concise learning-point fields.
- `GLOSSARY.md` remains available as a reference file.

Failure signs:

- Coach creates a new human-facing summary file during normal lesson writeback.
- Existing state files lose IDs or evidence in the name of readability.
- `GLOSSARY.md` is removed from the workspace structure.
```

Expected: regression file protects the chosen design boundary.

- [ ] **Step 5: Append interrupted writeback replay scenario**

Append:

```markdown
## 18. Interrupted Writeback Replay

Setup:

- A completed lesson file has `writeback_status: ready` and `writeback_complete: true`.
- `phrase-bank/*.md` and `state/phrase-bank-index.md` were updated.
- `state/review-queue.md`, `state/CURRENT.md`, or `state/writeback-ledger.md` was not updated because the previous session stopped mid-writeback.

Expected coach behavior:

- On next startup, coach compares `CURRENT.last_writeback_lesson_id` with `state/writeback-ledger.md` before selecting review targets.
- If the ledger is missing, stale, or mismatched, coach reads `RECOVERY-RULES.md` and replays the mechanically complete writeback idempotently.
- Duplicate evidence is not added, already-applied phrase-bank updates are treated as no-ops, and the missing review/current/ledger updates are completed.

Failure signs:

- Coach trusts materialized state without checking the ledger.
- Coach scans all historical lesson files on every normal startup even when ledger and `CURRENT.md` agree.
- Replay creates duplicate phrase-bank items or duplicate evidence lines.
```

Expected: regression file covers the mid-writeback recovery path.

- [ ] **Step 6: Verify regression additions**

Run:

```bash
rg -n "Context-Fit Old Review Handling|Promotion And Demotion Across Durable Files|Snapshot Noise Control|No Extra Human Dashboard|Interrupted Writeback Replay" skills/english-podcast-speaking-coach/REGRESSION-SCENARIOS.md
```

Expected: five new headings match.

- [ ] **Step 7: Commit**

Run:

```bash
git add skills/english-podcast-speaking-coach/REGRESSION-SCENARIOS.md
git commit -m "docs: add coach state policy regressions"
```

Expected: commit succeeds with one modified regression file.

### Task 7: Final Verification

**Files:**

- Check: `skills/english-podcast-speaking-coach/*.md`

- [ ] **Step 1: Run static policy checks**

Run:

```bash
rg -n --glob '!REGRESSION-SCENARIOS.md' "DASHBOARD\.md|MASTERED-EXPRESSIONS\.md|ERROR-PATTERNS\.md|REVIEW\.md" skills/english-podcast-speaking-coach
```

Expected: no matches; `rg` exit code `1` means success for this negative check.

Run:

```bash
rg -n "changing more than 5 durable|every 5 completed lessons|Use existing state to mix|Select 1-2 priority" skills/english-podcast-speaking-coach
```

Expected: no matches; `rg` exit code `1` means success for this negative check.

- [ ] **Step 2: Run positive policy checks**

Run:

```bash
rg -n "naturally fit this session|Practice 0-2 old targets|Context-fit review|ordinary lesson writeback does not create snapshots|Do not create state snapshots for ordinary lesson-end writeback|Promotion And Demotion|scan-friendly|Use one directory per snapshot|Replay payload requirements|compact writeback integrity check|Interrupted Writeback Replay|GLOSSARY.md" skills/english-podcast-speaking-coach
```

Expected: matches cover `SKILL.md`, `INTERACTION-RULES.md`, `STATE-SCHEMAS.md`, `WRITEBACK-FORMAT.md`, `RECOVERY-RULES.md`, `WORKSPACE-FORMAT.md`, and `REGRESSION-SCENARIOS.md`.

- [ ] **Step 3: Check Markdown diff health**

Run:

```bash
git diff --check
```

Expected: no trailing whitespace or whitespace error output.

- [ ] **Step 4: Review no learner workspace edits**

Run:

```bash
git status --short
```

Expected: no paths under `english-coach/` appear. The current repository should only show committed skill changes, or a clean working tree if each task was committed.

- [ ] **Step 5: Manual scenario review**

Read these scenarios in `REGRESSION-SCENARIOS.md`:

```text
14. Context-Fit Old Review Handling
15. Promotion And Demotion Across Durable Files
16. Snapshot Noise Control
17. No Extra Human Dashboard
18. Interrupted Writeback Replay
```

Expected:

- Scenario 14 preserves target hiding while avoiding forced unrelated review.
- Scenario 15 allows both promotion and later demotion.
- Scenario 16 prevents ordinary lesson snapshots.
- Scenario 17 blocks new dashboard files and preserves `GLOSSARY.md`.
- Scenario 18 recovers from interrupted writeback without scanning all lessons on healthy startup.

- [ ] **Step 6: Final commit if verification changed files**

If verification edits were made, run:

```bash
git add skills/english-podcast-speaking-coach
git commit -m "docs: verify coach state cleanup rules"
```

Expected: commit succeeds only if verification required edits. If no edits were made, do not create an empty commit.

## Self-Review

Spec coverage:

- Purposeful local files: Task 2 and Task 5.
- No new human dashboard: Task 1, Task 6, and Task 7.
- Keep `GLOSSARY.md`: Task 2 and Task 6.
- Context-fit review checks: Task 4 and Task 6.
- Promotion/demotion loop: Task 5 and Task 6.
- Snapshot reduction for 1000-lesson scale: Task 3 and Task 6.
- Replay safety: Task 3 keeps lesson replay as primary recovery; Task 5 preserves IDs and evidence.

Placeholder scan:

- This plan contains no `TBD`, `TODO`, `implement later`, or unspecified "add validation" steps.
- Each planned edit gives exact target files and concrete text to insert or replace.

Risk notes:

- The plan intentionally does not edit existing learner records. Existing noisy snapshots can be cleaned separately after the skill rules are updated.
- Context-fit review must not become forced warm-up. If old items do not fit the lesson, keep them in review for a future suitable context.
- Human readability must not remove `target_hidden`; target hiding is a runtime prompting rule, not a filesystem secrecy rule.
- Replay safety now depends on a startup integrity check plus ledger-last writeback order. The plan must not leave any state mutation after appending `state/writeback-ledger.md`.
