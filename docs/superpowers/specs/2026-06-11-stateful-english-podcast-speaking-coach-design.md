# Stateful English Podcast Speaking Coach Design

## Context

The current `english-podcast-speaking-coach` skill is strong at guiding one lesson. It defines active recall, hidden target chunks, learner repair, coverage states, speed rounds, and lesson-end expression areas. Its main limitation is that state is only described as internal tracking. After the conversation ends, the next session has no reliable learner memory unless the user repeats it.

This design makes the skill stateful by creating or reusing a learner workspace for the current coaching project. The design borrows from Matt Pocock's `teach` skill where it fits: a mission file, resources, notes, learning records, and a distinction between current teaching state and long-term learning evidence. It does not copy the HTML lesson model because this skill's primary experience is interactive speaking practice.

Pronunciation coaching is out of scope. The existing `发音提示` behavior should be removed completely from the lesson flow, state files, examples, and common-mistake guidance.

## Goals

- Automatically initialize or reuse a learner workspace for the current coaching project.
- Preserve learner state across sessions without requiring a database.
- Keep active state files short enough for the agent to read at the start of a session.
- Store full lesson history in per-lesson files so Markdown growth is bounded by file type.
- Track only evidence-based mastery: active production or successful near-transfer repair.
- Support cross-lesson review of expressions and learner-specific repair targets.
- Remove pronunciation coaching from the skill.

## Non-Goals

- Do not ask the user for a mission before the first lesson.
- Do not build a browser lesson system or printable HTML lessons.
- Do not track pronunciation, stress, linking, weak forms, pauses, or accent coaching.
- Do not write every interaction into long-running state files.
- Do not treat lesson coverage, correction display, or copied answers as mastery.

## Recommended Approach

Use a visible `english-coach/` directory under the discovered learner project root:

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

This keeps learner state portable and visible while avoiding pollution of the user's transcript, audio, or course-material directory. A hidden `.english-coach/` directory would be tidier but would undermine the main benefit of Markdown state: easy inspection and editing.

## Workspace Discovery

The skill should initialize automatically, but only when it is actually running a coaching session. Discussing, reviewing, or editing the skill itself should not create a learner workspace.

Discovery rules:

1. Starting from the current directory, look upward through parent directories for `english-coach/WORKSPACE.md`.
2. If found, reuse the nearest workspace.
3. If the user explicitly provides a project or course directory, create or reuse `english-coach/` inside that directory.
4. Otherwise, create `english-coach/` in the current directory when the user begins a speaking-coach session.
5. Do not ask the user to choose a workspace path during normal startup.

Discovery stops after 5 parent levels, at the filesystem root, or before the user's home directory unless the current directory itself is the home directory. This prevents a workspace in a broad parent directory from accidentally capturing unrelated coaching projects.

`WORKSPACE.md` identifies the workspace and records initialization metadata:

```md
# English Coach Workspace

- workspace_id: english-coach-YYYYMMDD-HHMMSS
- created_at: YYYY-MM-DD
- skill_name: english-podcast-speaking-coach
- skill_version: stateful-v1
- root_policy: nearest english-coach/WORKSPACE.md, otherwise current directory on coaching start
- discovery_limit: 5 parent levels, stop before home directory
```

## Default Mission

The skill owns the default mission. If `english-coach/MISSION.md` is missing, empty, or clearly damaged, the skill should restore the default content and continue. It should not stop to ask the user for learning goals.

Default content:

```md
# Mission

Help a Chinese-speaking learner turn English podcast and dialogue lessons into active spoken English.

The coach should prioritize active recall, reusable spoken chunks, natural sentence repair, near-transfer practice, free retelling, free expression, and cross-lesson review.

The goal is not to memorize transcripts. The goal is to build expressions the learner can actively produce in realistic situations.

Pronunciation coaching is out of scope.
```

If the user later states a new long-term goal, the skill may update `MISSION.md` and add a learning record explaining why the mission changed. The user should not have to answer a mission question before starting normal practice.

## File Responsibilities

### `WORKSPACE.md`

Stores workspace identity and initialization metadata. It is used for workspace discovery and version-aware migration. The skill should create it once and rarely edit it.

### `MISSION.md`

Stores the stable purpose of the learner workspace. It should be short and skill-owned by default. It may be updated only when the user's long-term goal changes.

### `NOTES.md`

Stores durable preferences and coaching constraints. Examples: the user prefers Chinese explanations, wants compact corrections, or often practices workplace scenarios. It is not a session log.

### `RESOURCES.md`

Stores source materials and trusted recurring resources: podcast series, transcript folders, course links, or user-provided lesson collections. It should help the skill decide what material is available without rescanning the full directory every time.

### `GLOSSARY.md`

Defines stable terms used by the coach, such as active recall, core target, near-transfer, repair target, extension expression, active phrase bank, and review queue. It should not contain pronunciation terminology.

It is a reference document, not default startup context. Read it when terminology is ambiguous, when updating the skill, or when writing documentation that must use the workspace's canonical terms.

### `state/CURRENT.md`

The short startup summary. Keep this file under one screen when possible. It should include:

- current learning focus;
- recent lesson summary;
- next-session recommendation;
- highest-priority review or repair themes;
- any temporary constraints that matter for the next lesson.

This file should be rewritten at the end of each lesson rather than appended forever.

### `state/phrase-bank-index.md`

Stores a compact derived index of active and stable phrase-bank entries. It should not become the full bank and is not source of truth. Use it to find which topic file contains a phrase and to avoid duplicates.

If this index is missing, stale, or contradictory, rebuild it from `phrase-bank/*.md`.

### `phrase-bank/*.md`

Stores expressions the learner has produced unaided in a new but related context. Split entries by scenario or speaking function, such as workplace, daily life, opinions, storytelling, travel, restaurants, or scheduling. Each item should include the smallest reusable chunk, Chinese meaning, use case, corrected personal example, current status, and evidence. Do not include expressions the coach merely explained.

The minimum status for entering phrase bank is `active`. Items in `attempted`, `needs_review`, or `repaired` stay in `state/review-queue.md` until the learner produces them unaided in a new but related context.

For phrase targets, `phrase-bank/*.md` becomes the source of truth once the item reaches `active`. `state/review-queue.md` may still reference the phrase for spaced review scheduling, but it should not become the source of truth for the phrase text, meaning, example, or mastery evidence.

If one phrase-bank file grows too large to scan comfortably, split it into narrower topic files and rebuild `state/phrase-bank-index.md`.

### `state/review-queue.md`

Stores review items by lifecycle section. It should hold core targets, practiced extension expressions, and repair targets that still need review.

Use three sections:

- `Active Review Items`: current review set, kept within the startup reading budget.
- `Dormant Review Items`: unresolved but temporarily paused items.
- `Retired Review Items`: stable, no longer relevant, or intentionally removed items.

The active section should stay around 1,500-2,500 English tokens or the Chinese equivalent. As a rough secondary limit, this usually means 12-20 compact items, not 30 verbose ones. When the active queue is too long, move lower-priority unresolved items to dormant rather than losing them in a generic archive. Archive only old queue snapshots or retired historical records.

### `state/repair-bank.md`

Stores repeated learner-specific issues such as tense, articles, prepositions, word order, collocation, information organization, and common Chinglish patterns. Each item should describe the pattern, corrected shape, evidence, and next near-transfer opportunity.

Use three sections:

- `Active Repair Patterns`: current repair patterns to consider during startup and lesson planning.
- `Dormant Repair Patterns`: unresolved but lower-priority patterns that should not be loaded every session.
- `Retired Repair Patterns`: stable, no longer relevant, or intentionally removed patterns.

Keep active repair content around 1,000-1,500 English tokens or the Chinese equivalent. If it grows beyond that, keep the most recent, repeated, or mission-relevant repair patterns active, compress older patterns, and move low-priority or stable patterns into dormant or retired sections.

### `state/writeback-ledger.md`

Stores a compact derived ledger of lesson writebacks that have been applied to materialized state. It is not source of truth. Completed lesson files remain source of truth; the ledger is an optimization so startup does not need to inspect every lesson file.

The ledger key is `lesson_id`, not filename alone. The path is a locator that may change if the user renames a file; the lesson id inside the lesson's `Writeback Summary` is the stable identity.

The ledger records completed `applied` writebacks only. It should not record `pending` or in-progress writeback state; interruption recovery comes from completed lesson files and idempotent replay.

If the ledger is missing, stale, or contradictory, replay completed lesson `Writeback Summary` sections idempotently and rebuild the ledger.

Example:

```md
# Writeback Ledger

- 20260611-001-restaurant-ordering | lessons/20260611-001-restaurant-ordering.md | applied | 2026-06-11
- 20260611-002-making-appointments | lessons/20260611-002-making-appointments.md | applied | 2026-06-11
```

### `lessons/*.md`

One lesson file per completed session. Name files with a date prefix and a short per-day sequence number:

```text
20260611-001-restaurant-ordering.md
20260611-002-making-appointments.md
```

If two sessions finish on the same date, increment the per-day sequence. If a number is skipped because a session did not complete, do not reuse it. Lesson files are allowed to be longer than state files. They preserve lesson evidence, selected core targets, learner attempts, final statuses, vocabulary, and what was written back to state.

### `learning-records/*.md`

Write learning records sparingly. They are decision-grade insights, not activity logs. Use them when:

- the learner demonstrates a non-trivial ability that should change future difficulty;
- the learner reveals prior knowledge that should prevent reteaching;
- a misconception is corrected;
- the mission changes.

Use date-prefixed numbering and short files:

```md
# Can Repair Soft Requests With Near Transfer

The learner successfully repaired a direct request into a softer spoken request in a new appointment scenario. Future lessons can assume basic awareness of softening with "could" but should continue testing it in higher-pressure contexts.
```

Example filename: `20260611-001-can-repair-soft-requests.md`.

Good learning-record candidates:

- The learner repeatedly used soft-request forms unaided across two different scheduling scenarios, so future lessons can raise difficulty.
- The learner disclosed that they already use English at work for status updates but struggle with spontaneous disagreement.
- A repeated misconception was corrected, such as using `at last` for neutral sequence where `finally` is expected.

Do not create learning records for:

- a phrase the coach merely explained;
- a one-off grammar repair that is already captured in `repair-bank.md`;
- a normal lesson summary, activity log, or list of covered material.

### `archives/`

Stores old review queues, old summaries, or retired state snapshots. Archives prevent active state files from becoming long-term dumps.

## State Schemas

The state files are Markdown, but each durable item should use a predictable field shape so the skill can merge, deduplicate, and reconcile entries across sessions.

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

The skill may mention unpracticed extension expressions in lesson files, but they should not enter durable review or phrase-bank state until the learner actively practices them.

### Conservative State Updates

The model is not a perfect state machine. Durable state updates should therefore be conservative and evidence-led:

- Every status upgrade must cite a concrete evidence line from the current lesson.
- If evidence is ambiguous, keep the lower status and leave the item in review.
- Do not mark `stable` from one success. `stable` requires spaced success or success in a meaningfully different context.
- Do not infer mastery from fluency immediately after the answer was revealed.
- User correction overrides model inference.
- When a status changes, add a short evidence note explaining why.

### ID Allocation

Durable items use stable ids within their namespace:

- `PB-0001` for phrase-bank items;
- `RQ-0001` for review-queue items;
- `RB-0001` for repair-bank items.

Rules:

- New item ids use the next highest numeric id within their namespace.
- Before creating a new id, scan the relevant active state file and derived index.
- If `state/phrase-bank-index.md` is stale, missing, or suspect, rebuild it from `phrase-bank/*.md` before allocating a new `PB` id.
- Replayed writebacks must reuse ids recorded in the lesson's `Writeback Summary`.
- If an id collision is found, preserve the older item, assign a new id to the newer item, and update references in the current writeback.
- Do not allocate a new id when an existing item has the same target chunk, safe prompt, and source lesson evidence.

### Evidence Identity

Evidence lines should include a lesson id prefix so replay can avoid duplicates:

```md
- 20260611-001:RQ-0007: failed without hint in appointment role-play
- 20260611-002:PB-0012: unaided near-transfer production in workplace scheduling
```

The lesson id plus item id plus short event description is the deduplication key. Replaying the same `Writeback Summary` must not append the same evidence line twice.

### Downgrade Rules

Mastery can move down as well as up:

- `stable` -> `active`: the learner hesitates or needs minor support, but can still produce the target in a related context.
- `stable` -> `needs_review`: the learner fails unaided or makes the same reusable error again.
- `active` -> `needs_review`: the learner cannot produce the item without a hint in a related context.
- `active` -> `repaired`: the learner fails first but repairs successfully after a cue.
- `repaired` -> `needs_review`: the learner fails again or copies a revealed answer.
- any status -> `retired`: the user says the item is no longer relevant, the item is intentionally removed, or a `stable` item has remained stable after spaced review and no longer needs rotation.

The phrase "new but related context" means the scenario changes at least one practical variable, such as speaker relationship, setting, goal, object, time pressure, or emotional tone, while still testing the same reusable chunk. When unsure whether a context is different enough, do not upgrade beyond `repaired`.

### Privacy and Anonymization

Durable state should preserve the language pattern, not private facts. Personal examples should be lightly anonymized by default when they mention real names, companies, addresses, health issues, relationship details, salary, legal matters, or sensitive workplace information.

These privacy and anonymization rules apply to all durable files, including lesson files, `learning-records/*.md`, `NOTES.md`, state files, phrase-bank files, and `Writeback Summary` sections.

Examples:

```text
Too specific: I need to tell Alice from Company X that my manager rejected the proposal.
Durable state: I need to tell a colleague that my manager rejected the proposal.
```

If the user explicitly says not to record something, do not store it in durable state. If the language pattern is useful, store a generic replacement.

### Review Queue Item

Use one stable item id per target:

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

`target_hidden` is for internal tracking only and must not be shown inside learner prompts. `learner_safe_prompt` is the Chinese recall prompt or scenario cue that can safely be shown to the learner.

Because the workspace is visible and editable Markdown, target hiding applies to the live coaching prompt flow, not to the file system. State files may contain `target_hidden` for auditability and replay, but the coach must not surface those targets in recall prompts before the learner attempts.

`next_review_hint` is advisory, not source of truth. Create it with the item, update it when status or priority changes, and ignore it when the current lesson context suggests a better review opportunity.

For phrase targets that have reached `active`, `source_phrase_id` is the stable reference. `source_phrase_file` is only a locator. If phrase-bank topic files are split, renamed, or moved, update `source_phrase_file` in affected review items or resolve it through `state/phrase-bank-index.md`.

### Phrase Bank Item

Use this compact shape inside `phrase-bank/*.md`:

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

The derived `state/phrase-bank-index.md` maps ids and chunks to topic files:

```md
- PB-0012 | I was wondering if... | phrase-bank/workplace.md | active
```

### Repair Bank Item

Use this shape for repeated learner-specific repair patterns:

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

### Current Summary

`state/CURRENT.md` should include:

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

`last_writeback_lesson_id` is the stable key. `last_writeback_lesson_path` is an optional locator and may be repaired from the ledger or lesson files if the user renames a lesson file.

Use `temporary_constraints` only for short-lived next-session constraints. Durable preferences and coaching constraints belong in `NOTES.md`; if the two conflict, `NOTES.md` is authoritative for long-term preferences.

## Session Startup Flow

1. Run workspace discovery.
2. If no workspace exists and the user is starting a coaching session, create `english-coach/` in the selected root.
3. Create `WORKSPACE.md` during initialization if missing.
4. If `MISSION.md` is missing or empty, restore the default mission without asking the user.
5. Read only the startup set by default:
   - `english-coach/MISSION.md`
   - `english-coach/NOTES.md`
   - `english-coach/state/CURRENT.md`
   - `english-coach/state/review-queue.md`
   - `english-coach/state/repair-bank.md`
6. Read `state/writeback-ledger.md` only when checking whether completed lesson writebacks are unapplied.
7. Read `state/phrase-bank-index.md` when selecting review items, checking whether a phrase is already active, or preparing lesson-end output.
8. Read the relevant `phrase-bank/*.md` topic file only after the index points to it.
9. Read `RESOURCES.md` only when the user asks for a lesson without providing material, asks to continue a course or podcast series, or when the current lesson source is ambiguous.
10. Read older lesson files only when a specific conflict, replay, or user request requires evidence.
11. If the user provides a lesson, transcript, or file path, build the lesson around that material and mix in a small number of review items.
12. If the user provides no new material, start a review, free-retelling, or free-expression session based on `CURRENT.md`, `review-queue.md`, and `RESOURCES.md` when relevant.

## Lesson Flow Changes

The existing lesson sequence remains:

```text
guided preview -> active-recall blocks -> free retelling and repair -> free expression -> precise Speed round -> lesson-end expression areas
```

The stateful version adds:

- a short startup review of previous needs when useful;
- review-queue items mixed into role-play only when they fit naturally;
- repair-bank patterns used to create near-transfer prompts;
- lesson-end writeback to the workspace.

Target leakage rules remain unchanged: target chunks must stay hidden until the learner attempts, asks for help, or gets stuck.

## Pronunciation Removal

Remove all `发音提示` behavior from the skill:

- no `发音提示` label;
- no stress, pause, linking, weak-form, or pronunciation-marker rules;
- no optional pronunciation hints inside active-recall blocks;
- no pronunciation content in lesson-end areas;
- no pronunciation state file or queue;
- no common-mistake guidance about pronunciation hints;
- no pronunciation hints counted toward mastery.

If the user explicitly asks how to pronounce a word or phrase, the coach may answer briefly in ordinary prose. That answer must not enter the lesson flow, state files, review queue, or mastery logic.

## Lesson-End Writeback

Completed lesson files are the source of truth for learning evidence and intended state changes. `state/*.md` and `phrase-bank/*.md` are materialized views optimized for startup and review. They should be recoverable from completed lesson files plus user corrections.

Only a lesson file with a mechanically complete `Writeback Summary` is replayable. A partial lesson file is evidence for the human reader but must not be used for automatic reconciliation until the missing writeback summary is completed.

A `Writeback Summary` is complete only when all of these are true:

- it contains `writeback_status: ready`;
- it contains `lesson_id`;
- it contains the required update sections, even when the value is `none`;
- its final field is `writeback_complete: true`.

Required update sections:

- `phrase_bank_updates`;
- `review_queue_updates`;
- `repair_bank_updates`;
- `learning_records`;
- `current_summary`.

`current_summary` must always contain the complete intended `CURRENT.md` values after the lesson, even if nothing changed. Do not write `current_summary: none`; replay should be able to rewrite `CURRENT.md` without guessing previous state.

At the end of each lesson:

1. Prepare the complete lesson file content, including lesson evidence and `Writeback Summary`.
2. Write the lesson file before mutating any state file.
3. Verify the lesson file contains a mechanically complete `Writeback Summary`.
4. Snapshot state files into `archives/state-snapshots/` when snapshot criteria are met.
5. Apply the writeback summary idempotently in this order:
   - update `phrase-bank/*.md`;
   - rebuild or update `state/phrase-bank-index.md`;
   - update `state/review-queue.md`;
   - update `state/repair-bank.md`;
   - create any justified `learning-records/*.md`;
   - rewrite `state/CURRENT.md` with `last_writeback_lesson_id`;
   - append the lesson as `applied` in `state/writeback-ledger.md`.
6. Move review overflow from active to dormant before archiving retired history.

`Writeback Summary` should be explicit enough to replay:

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

On startup, use `state/writeback-ledger.md` to detect completed lesson files whose writeback has not been applied. If the ledger is missing or suspect, inspect completed lesson files in filename order and replay any missing mechanically complete `Writeback Summary` sections. Replaying a writeback must be idempotent: updating an existing item with the same evidence should not duplicate it. If the ledger is missing an entry but the materialized state already contains the lesson's updates, replay should be a no-op and then append the missing ledger entry.

Snapshot criteria:

- before replaying reconciliation;
- before changing more than 5 durable items in one lesson;
- before moving more than 5 review items between active, dormant, and retired;
- at least every 5 completed lessons if state has changed.

## Conflict Handling

- If an expression appears as both active and needs-review, resolve by evidence quality rather than by file order.
- Prefer the most recent unaided production over older evidence.
- Prefer repeated failures across lessons over a single success.
- Prefer successful near-transfer over exact repetition.
- Prefer explicit user correction over model inference.
- If a state file is missing, recreate the minimal template and continue.
- If a state file is malformed, preserve the original in `archives/` and create a clean replacement.
- If the review queue exceeds active capacity, keep recent, repeated, and mission-relevant items active; move other unresolved items to dormant.
- If the user corrects the stored state, trust the user and update the relevant file.
- If old lesson evidence conflicts with `CURRENT.md`, reconcile using evidence quality rules and then rewrite `CURRENT.md`.
- If the user says not to record a phrase, personal example, or lesson detail, omit it from durable state. Use a generic example if needed for practice.

## Migration Rules

`WORKSPACE.md` includes `skill_version` so future schemas can migrate conservatively.

If `skill_version` is older than the current schema:

1. Do not rewrite all files immediately.
2. Create a state snapshot before migration when changing durable state.
3. Migrate only files needed for the current session.
4. Preserve unknown fields unless they conflict with required schema.
5. Prefer additive changes over destructive rewrites.
6. Record migration notes in `archives/state-snapshots/` when the migration affects teaching behavior or state interpretation.
7. Update `WORKSPACE.md` only after the needed migration succeeds.

## Implementation Phases

### Phase 1: Minimal Stateful Coach

- Discover or create the learner workspace.
- Create `WORKSPACE.md` and default files.
- Restore the default mission without asking the user for goals.
- Read the startup set.
- Remove pronunciation guidance from the skill.
- Use the conservative mastery lifecycle for current-session evidence.
- Write completed lesson files with `Writeback Summary` and `writeback_status: ready`.
- Add the final `writeback_complete: true` marker to completed lesson files.
- Maintain a lightweight `state/writeback-ledger.md` after successful state updates.
- Rewrite `CURRENT.md`.
- Append or update `review-queue.md` and `repair-bank.md` conservatively.
- Write simple `active` phrase-bank entries when the evidence is clear, without topic splitting or index optimization.
- Apply privacy anonymization before writing durable examples.

### Phase 2: Durable Recovery

- Replay completed `Writeback Summary` sections idempotently.
- Rebuild stale `state/phrase-bank-index.md`.
- Add state snapshots using the snapshot criteria.
- Apply downgrade rules across older evidence during reconciliation.
- Add conservative migration handling for older `skill_version` values.

### Phase 3: Scale Management

- Split phrase-bank topic files when they become too large.
- Tune active, dormant, and retired review-queue rotation.
- Add more migration rules for future schema versions.
- Consider helper scripts only if manual Markdown updates become too fragile.

## Validation Scenarios

Validate the redesign with manual scenarios:

### Workspace

1. First coaching run in a clean directory creates `english-coach/WORKSPACE.md` and starts without asking for a mission.
2. Startup from a subdirectory finds and reuses the nearest parent `english-coach/WORKSPACE.md`.
3. Upward workspace discovery stops at the configured boundary and does not accidentally reuse a broad home-directory workspace.
4. Discussing or editing the skill itself does not create a learner workspace.

### Startup

5. Existing workspace startup reads the short startup set instead of scanning all lessons.
6. `RESOURCES.md` is read when the user asks to continue without providing new material.

### Lesson Flow and Target Hiding

7. A transcript-based lesson selects core targets and mixes in relevant review items.
8. Target chunks are not leaked in prompts, including review items that are already known.
9. `target_hidden` and `learner_safe_prompt` remain separate in review queue entries.
10. Active review queue stays within the startup reading budget rather than a fixed item count.

### Writeback and Recovery

11. Lesson-end output writes a completed lesson file with `writeback_status: ready` before mutating state.
12. A replayable lesson has required writeback sections, complete `current_summary`, and final `writeback_complete: true`.
13. A partial lesson file without a mechanically complete `Writeback Summary` is not used for automatic reconciliation.
14. Startup detects a completed lesson writeback summary that was not fully reflected in state and replays it idempotently.
15. Ledger lag after successful state updates causes a no-op replay and then repairs the `lesson_id`-keyed ledger with an `applied` entry.
16. Duplicate review items are merged rather than copied.
17. A stale `state/phrase-bank-index.md` is rebuilt before allocating a new `PB` id.
18. Replayed writeback does not duplicate existing evidence lines.

### State Quality

19. A phrase appearing in phrase bank and review queue with conflicting statuses is resolved by evidence quality.
20. A `stable` item can downgrade when the learner fails unaided in a related context.
21. Phrase-bank rejects `attempted`, `needs_review`, and `repaired` items until they become `active`.
22. Repair bank uses active, dormant, and retired sections and stays within its startup reading budget by compressing or retiring low-priority patterns.
23. `next_review_hint` updates when status or priority changes but does not override lesson context.
24. Learning records are created for decision-grade evidence and skipped for normal repairs or activity summaries.
25. After several lessons, `CURRENT.md` remains short, uses `last_writeback_lesson_id` as the stable key, and keeps durable preferences in `NOTES.md`.
26. Phrase-bank growth is handled through topic files, the derived `state/phrase-bank-index.md`, and repaired `source_phrase_file` locators.
27. New `PB`, `RQ`, and `RB` ids do not collide with existing ids.
28. Visible `target_hidden` fields in Markdown are not shown in live recall prompts.

### Privacy and Pronunciation

29. If the user says not to record a personal example, durable state omits it or uses a generic replacement.
30. A user pronunciation question receives only a brief ordinary answer and does not create any pronunciation state.
31. The skill text no longer contains `发音提示`, stress/linking/weak-form workflows, or pronunciation examples.
32. Personal examples containing sensitive details are anonymized before entering durable state.

### Migration

33. A workspace created under an older `skill_version` is migrated conservatively.

## Open Implementation Notes

- The current project directory is not a git repository, so design docs can be written here but not committed unless the project is initialized or moved into a repository.
- The implementation can be done by editing the existing `skills/english-podcast-speaking-coach/SKILL.md`; scripts are optional and should only be added if manual Markdown update rules become too fragile.
- Keep the first implementation small: discover or initialize workspace, create `WORKSPACE.md`, write completed lesson files with replayable writeback summaries, update materialized state conservatively, use stable item schemas, and remove pronunciation guidance.
