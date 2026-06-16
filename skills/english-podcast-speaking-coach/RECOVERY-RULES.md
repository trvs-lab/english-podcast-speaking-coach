# Recovery Rules

Read this file when replaying writebacks, resolving conflicts, handling malformed state, anonymizing durable examples, or migrating old workspace versions.

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

Lightly anonymize personal examples by default when they mention real names, companies, addresses, health issues, appointments, family events, relationship details, salary, legal matters, or sensitive workplace information.

Example:

```text
Too specific: I need to tell Alice from Company X that my manager rejected the proposal.
Durable state: I need to tell a colleague that my manager rejected the proposal.
```

Prefer generic durable examples such as `a personal appointment`, `a family event`, `a health issue`, `a colleague`, `a client`, or `a company project` when the specific fact is not needed to preserve the language pattern.

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
