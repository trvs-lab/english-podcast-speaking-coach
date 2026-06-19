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

**Mastery can belong to a reusable pattern, not only an exact sentence.** When the learner has produced the same sentence frame with different meaningful slots in unaided near-transfer, treat the frame as covered. Do not keep testing the same frame by swapping only one noun unless the new slot exposes a real grammar, collocation, or meaning problem.

**Assistance caps the current attempt.** If the learner needed a keyword hint, sentence frame, complete answer, English menu, or prior coach model of the exact target, the current attempt can be at most `repaired`. Upgrade to `active` only after later unaided production in a new but related context.

**Completed lesson files are the source of truth.** `state/*.md` and `phrase-bank/*.md` are materialized views optimized for startup and review. They must be recoverable from mechanically complete lesson `Writeback Summary` sections plus explicit user corrections.

**Durable files have learner-state jobs.** `phrase-bank/*.md` stores expressions the learner can actively produce, `state/review-queue.md` stores unmastered expressions that should return, and `state/repair-bank.md` stores recurring learner-specific error patterns. Keep these files useful for both startup and human inspection: write the expression or pattern first, then the status and evidence.

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
- In direct role-play, use English for the coach's in-character lines by default; use Chinese for setup, communicative intent, vocabulary explanation, and feedback.
- Give help step by step: `keyword hint -> sentence frame -> complete natural expression`.
- Use problem-triggered correction: brief feedback for natural answers, full correction for transferable problems. Full correction has priority when the learner makes Chinglish, lesson-chunk, tense, article, collocation, linkage, information-order errors, or omits required prompt details.
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
- Letting role-play collapse into Chinese narration instead of English in-character speech plus Chinese intent prompts.
- Letting brevity override full correction for transferable learner errors.
- Silently adding required prompt details to the natural version without telling the learner what was missing.
- Introducing a useful coach-added expression without explaining its meaning, use case, or why it improves the learner's wording.
- Treating repeated spelling errors in important expressions as throwaway polish.
- Ending with summaries that are easy to skim but too thin to support later recall.
