# Writeback Format

Read this file before writing a completed lesson file, preparing a `Writeback Summary`, applying lesson-end state updates, creating learning records, rewriting `state/CURRENT.md`, or appending `state/writeback-ledger.md`.

## Lesson-End Writeback

At the end of each completed lesson:

1. Prepare the complete lesson file content, including lesson evidence and `Writeback Summary`.
2. Write the lesson file under `english-coach/lessons/` before mutating any state file.
3. Verify the lesson file contains a mechanically complete `Writeback Summary`.
4. Run the lesson-end writeback self-review checklist below.
5. Snapshot state files into `archives/state-snapshots/` when snapshot criteria are met, before mutating state files.
6. Apply the writeback summary idempotently in this order:
   - update `phrase-bank/*.md`;
   - rebuild or update `state/phrase-bank-index.md`;
   - update `state/review-queue.md`;
   - update `state/repair-bank.md`;
   - create justified `learning-records/*.md`;
   - rewrite `state/CURRENT.md` with `last_writeback_lesson_id`;
   - append the lesson as `applied` in `state/writeback-ledger.md`.
7. Move review overflow from active to dormant before archiving retired history.

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

Before applying durable state, self-review the prepared writeback:

1. **Completeness:** `Writeback Summary` has every required field and final `writeback_complete: true`.
2. **Snapshot:** snapshot criteria were checked; if changing more than 5 durable items, a state snapshot is prepared before mutation.
3. **Mastery evidence:** every `active` phrase cites unaided learner production or later unaided near-transfer. Anything produced only after a hint, sentence frame, answer reveal, English menu, or prior coach model stays `repaired` or lower.
4. **Target leakage:** no target modeled by the coach before the learner's attempt is counted as unaided evidence.
5. **Review scheduling:** each active review item has `last_seen`, `next_due`, `attempt_count`, `last_outcome`, and concrete evidence.
6. **Learning records:** user corrections, corrected misconceptions, or ability changes that should affect future teaching are captured in `learning-records/*.md` or explicitly listed as `none` with a reason.
7. **Privacy:** durable examples anonymize personal names, companies, health details, family events, appointments, salary, legal matters, and sensitive workplace facts unless the user explicitly wants them preserved.
8. **Index and ledger:** phrase-bank index updates match phrase-bank files, and the ledger is appended only after all state mutations succeed.

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
- the learner corrects the coach or clarifies their intent in a way that should prevent the same coaching mistake later;
- the coach discovers a recurring scoring or prompting issue that changes future lesson handling;
- the mission changes.

Do not create a learning record for a phrase the coach merely explained, a one-off repair already captured in `repair-bank.md`, a normal lesson summary, or a list of covered material.
