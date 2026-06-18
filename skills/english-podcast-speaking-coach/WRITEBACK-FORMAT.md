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

## Writeback Self-Review

Before applying durable state, self-review the prepared writeback:

1. **Completeness:** `Writeback Summary` has every required field and final `writeback_complete: true`.
2. **Snapshot:** snapshot criteria were checked; if changing more than 5 durable items, a state snapshot is prepared before mutation.
3. **Mastery evidence:** every `active` phrase cites unaided learner production or later unaided near-transfer. Anything produced only after a hint, sentence frame, answer reveal, English menu, or prior coach model stays `repaired` or lower.
4. **Target leakage:** no target modeled by the coach before the learner's attempt is counted as unaided evidence.
5. **Review scheduling:** each active review item has `last_seen`, `next_due`, `attempt_count`, `last_outcome`, and concrete evidence.
6. **Learning records:** user corrections, corrected misconceptions, or ability changes that should affect future teaching are captured in `learning-records/*.md` or explicitly listed as `none` with a reason.
7. **Privacy:** durable examples anonymize personal names, companies, health details, family events, appointments, salary, legal matters, and sensitive workplace facts unless the user explicitly wants them preserved.
8. **Index and ledger:** phrase-bank index updates match phrase-bank files, and the ledger is appended only after all state mutations succeed.
9. **Active threshold:** every phrase-bank addition satisfies one active evidence path: target-hidden production in a new related context, repair plus later unaided production, or valid pattern-level evidence. Original-scene-only production is not enough.
10. **Unmastered routing:** attempted-but-unmastered targets are in review queue updates; optional unpracticed expressions are explicitly left as extension-only or next-session candidates with a reason.
11. **Spelling repairs:** repeated high-value spelling errors are either captured as repair/review items or explicitly treated as minor one-off polish.
12. **Replayable payload:** new phrase, review, and repair updates include the fields needed to reconstruct the durable item without guessing from surrounding prose.

Use this shape:

```md
## Writeback Summary

- writeback_status: ready
- lesson_id: 20260611-002-making-appointments
- phrase_bank_updates:
  - id: PB-0012
    action: add
    destination: phrase-bank/workplace.md
    chunk: I was wondering if...
    meaning_zh: 我想知道是否...
    use_case: polite scheduling request
    source_lesson: lessons/20260611-002-making-appointments.md
    personal_example: I was wondering if we could move the meeting.
    recall_prompt_zh: 礼貌询问能不能改会议时间
    status: active
    evidence:
      - 20260611-002:PB-0012: unaided near-transfer in scheduling scenario
- review_queue_updates:
  - id: RQ-0007
    action: add
    type: core_target
    target_hidden: How does next Monday work for you?
    learner_safe_prompt: 用英文问“下周一你方便吗？”
    reason: needed sentence frame before producing it
    status: needs_review
    priority: high
    source_lesson: lessons/20260611-002-making-appointments.md
    last_seen: 2026-06-11
    next_due: 2026-06-14
    attempt_count: 1
    last_outcome: sentence-frame-assisted production
    next_review_hint: scheduling or appointment lesson
    evidence:
      - 20260611-002:RQ-0007: sentence-frame-assisted production
- repair_bank_updates:
  - id: RB-0004
    action: add
    pattern: repeated spelling of conversation
    corrected_shape: conversation
    status: needs_review
    next_near_transfer: use conversation in a new meeting or networking sentence
    evidence:
      - 20260611-002:RB-0004: misspelled conversation as convasation
- learning_records:
  - none
- current_summary:
  - last_writeback_lesson_id: 20260611-002-making-appointments
  - last_writeback_lesson_path: lessons/20260611-002-making-appointments.md
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
