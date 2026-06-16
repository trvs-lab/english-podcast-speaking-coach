# Workspace Format

Read this file during workspace discovery, workspace initialization, default mission restoration, startup read-set decisions, or workspace schema version questions.

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
