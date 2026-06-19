# State Schemas

Read this file when creating, updating, deduplicating, reconciling, grading, or merging durable review queue, phrase bank, or repair bank items.

## Durable State Schemas

Use predictable Markdown fields for durable items so writebacks can be merged, replayed, and audited.

Keep durable files scan-friendly for humans. Put the expression, pattern, learner-safe prompt, or corrected shape near the top of each item. Keep IDs, statuses, source fields, and evidence because replay depends on them, but do not make the item read like a database row before it explains the learning point.

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

Pattern-level active evidence is valid when the learner produces the same reusable frame in at least two meaningfully different unaided contexts, or repairs it after correction and later produces it unaided in a new context. Slot-only repetition, such as changing `names` to `faces` inside an already-mastered frame, does not by itself create a new review need.

Active phrase bank admission requires one of these evidence paths:

- exact chunk: target-hidden learner production in a new related context, either as clean unaided near-transfer or as later unaided production after correction, reveal, or sentence-frame help;
- pattern family: at least two unaided productions with different meaningful slot values or contexts, with no unresolved grammar, collocation, or meaning error;
- repaired item: correction or hint-assisted repair followed by later unaided production in a new related context.

Do not mark an item active from original-scene-only production, copied correction, sentence-frame completion, answer reveal, English menu, coach-modeled exact chunk, ambiguous evidence, or slot-only repetition. The learner can own an exact chunk without yet owning the broader pattern family.

Do not mark an item `active` unless the learner produces it without seeing the target chunk in a new but related context. Copying a just-revealed correction cannot upgrade an item beyond `needs_review`; a successful prompted fix can upgrade it to `repaired`.

Hint-gated production is assisted production. If the learner needed a keyword hint, sentence frame, complete answer, or coach-provided English menu for the target chunk, the target can be at most `repaired` in that attempt. Upgrade it to `active` only after a later unaided attempt in a new but related context.

Do not mark `stable` from one success. `stable` requires spaced success or success in a meaningfully different context.

Every durable status upgrade must cite concrete evidence from the current lesson. If evidence is ambiguous, keep the lower status and leave the item in review. User correction overrides model inference.

### Promotion And Demotion

When a review-queue expression reaches `active`, add or update the corresponding phrase-bank item, then move the review item out of `Active Review Items`. Keep the review item only as dormant or retired history unless it still needs scheduled review.

When a repair-bank pattern becomes reliable, update the repair pattern to `active` or `stable` with evidence. If it no longer needs near-term attention, move it to dormant or retired instead of keeping it in active repair.

When a previously active or stable phrase fails in a later lesson, do not write `needs_review` or `repaired` into `phrase-bank/*.md`; phrase-bank statuses remain `active`, `stable`, or `retired`. Keep the phrase-bank evidence history, add `review_reference: RQ-*` when useful, and add a new review-queue item or reactivate the existing one as the current unmastered record. Downgrade using the rules in `RECOVERY-RULES.md` and cite the new failure evidence.

Promotion, retention, demotion, and retirement must be written as explicit writeback actions. Do not rely on prose in the lesson summary to imply a state move.

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

Use three sections in `state/review-queue.md`:

- `Active Review Items`
- `Dormant Review Items`
- `Retired Review Items`

Keep active review around 1,500-2,500 English tokens or the Chinese equivalent. When active review grows too long, move lower-priority unresolved items to dormant instead of hiding them in an archive.

Use `last_seen`, `next_due`, `attempt_count`, and `last_outcome` to keep review scheduling stateful instead of list-shaped. `next_due` can be approximate; choose a concrete date when the item should next be mixed naturally into a lesson or review session. After each review attempt, update all four fields and add evidence.

`target_hidden` is for internal tracking and writeback audit. The live coaching prompt must show `learner_safe_prompt` or a Chinese scenario cue, not `target_hidden`, before the learner attempts.

Review queue items may represent unmastered core targets, practiced extension expressions, learner-specific repair targets, or high-value spelling repair targets. Do not add every coach-shown expression. Add only items that should actively return because they are central, attempted-but-unmastered, repeatedly misspelled, or explicitly requested by the learner.

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
- learner_safe_prompt: "换一个场景，用更委婉的方式提出请求"
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
