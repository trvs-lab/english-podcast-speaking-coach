# Regression Scenarios

This file is maintenance-only. Read it when changing `english-podcast-speaking-coach` rules or reviewing a change to the skill. Do not read it during ordinary coaching startup, live lessons, review sessions, free retelling, or free-expression practice.

Use these scenarios as manual regression checks. Each scenario lists the setup, expected coach behavior, and failure signs. They are not complete transcripts.

## 1. Target Leakage

Setup:

- Lesson target chunk: `with a side of`.
- Scene vocabulary: `beef tacos`, `salsa`.
- Learner should order food and ask for salsa on the side.

Expected coach behavior:

- The prompt may show `beef tacos（牛肉塔可）` and `salsa（莎莎酱）`.
- The prompt must express `with a side of` only in Chinese, such as `并且想配一份`.
- The learner gets an active-recall chance before seeing the English target chunk.

Failure signs:

- Prompt says `with a side of salsa` before the learner attempts.
- Coach treats `with a side of` as ordinary vocabulary instead of a target chunk.
- Learner repetition after the leaked prompt is counted as unaided production.

## 2. Help Ladder

Setup:

- Target chunk: `How does next Monday work for you?`
- Learner replies: `不会说`.

Expected coach behavior:

- First help response gives only a keyword hint such as `work for you`.
- Second help response may give a sentence frame such as `How does ___ work for you?`.
- Complete expression appears only after another help request or an explicit answer request.
- After each help level, the coach asks the learner to try again.

Failure signs:

- Coach gives the complete expression on the first help request.
- Coach repeats the exact help cue in every prompt.
- Assisted production is marked `active`.

## 3. Role-Play Contract

Setup:

- Lesson target chunk: `all set`.
- Coach plays a travel partner.
- Learner should say they are almost ready and the car is packed.

Expected coach behavior:

- Coach role line is English by default, such as `Travel partner: Ready to go?`.
- Coach does not say `Are we all set?` before the learner attempts `all set`.
- If natural English role speech would leak the target, the coach paraphrases, switches roles, uses Chinese only when needed, or demotes the target.

Failure signs:

- Coach replaces the whole role-play with Chinese narration when no leakage requires it.
- Coach models `all set` in the role line and later counts learner repetition as unaided.
- Prompt uses `我问你：「...」` when the coach should be another speaker in the scene.

## 4. Layered Correction

Setup:

- Learner says: `I forget bought water and we all prepared, but the map not in car.`
- Intended meaning: forgot to buy water, almost everything is ready, but the map is not in the car.
- Useful lesson chunks include `forgot to`, `all set`, and `in the car`.

Expected coach behavior:

- Coach gives a natural version first.
- Coach explains every key change that affects meaning or naturalness.
- Coach groups explanations by priority, such as action pattern, lesson chunk, grammar, and article/preposition.
- Coach repairs only the highest-value 1-2 points through near-transfer.

Failure signs:

- Coach silently changes several learner choices without explanation.
- Coach explains only a small subset of meaningful errors.
- Coach asks the learner to copy the polished full sentence.

## 5. Lesson-Language Anchoring

Setup:

- Learner says a natural non-lesson version: `everything covered`.
- Lesson offers the stronger chunk: `all our bases covered`.

Expected coach behavior:

- Coach first accepts the learner's meaning if it is natural.
- Coach offers the lesson chunk as a useful upgrade.
- Coach explains why the lesson chunk is more useful or more idiomatic.
- Coach tests it once in a new but related context before counting it as active.

Failure signs:

- Coach rejects a natural learner answer only because it differs from the transcript.
- Coach adds the lesson chunk to active state without learner production.
- Coach exposes internal category names such as `lesson chunk`, `repair target`, or `optional extension` to the learner.

## 6. Free Retelling Correction

Setup:

- Learner gives a 4-5 sentence first-person retelling with tense errors, missing articles, Chinglish ordering, and a missed lesson chunk.

Expected coach behavior:

- Coach lets the learner finish before polishing.
- Coach keeps first-person retelling consistent.
- Coach groups correction by story order/linkage, tense/time, articles/countability, lesson chunks, and Chinglish replacements when those issues appear.
- Coach repairs only the highest-value 1-2 sentences through near-transfer.

Failure signs:

- Coach interrupts before the retelling is complete.
- Coach rewrites the paragraph but explains only a few of the changes.
- Coach asks the learner to copy the polished paragraph verbatim.

## 7. Coverage Checkpoint

Setup:

- A role-play block has ended after 2-3 turns.
- Some chunks are active, and one chunk needs another attempt.

Expected coach behavior:

- Coach gives one compact classroom checkpoint.
- Active items appear under a learner-facing label such as `已经能自己说出来`.
- Items needing review appear under a learner-facing label such as `还要再练一下`.
- Only smallest reusable chunks or fixed questions appear.

Failure signs:

- Coach shows raw labels such as `coverage`, `needs_review`, or `repaired` as the main learner-facing display.
- Coach shows full learner replies, long retellings, or paragraph rewrites as coverage items.
- Coach shows a coverage table after every reply.

## 8. Lesson-End Evidence

Setup:

- One expression was produced unaided in near-transfer.
- One expression was produced only after a sentence frame.
- One useful expression was shown by the coach but not practiced by the learner.
- Several scene words were explained.

Expected coach behavior:

- Unaided production goes under `这节课真正说出来的表达`.
- Sentence-frame-assisted production goes under `下次还要再练的表达` with a short reason.
- Coach-shown but unpracticed expression goes under `听过但还没练熟的表达`.
- Scene words go under `这节课碰到的词`.

Failure signs:

- Assisted production is placed in the active expression section.
- Extension expressions and vocabulary are mixed into the active phrase bank.
- Lesson-end summary is a thin list without meaning, use case, learner example, or recall prompt for important active expressions.

## 9. Writeback Evidence

Setup:

- A completed lesson file is being prepared.
- The lesson has one active phrase, one review item, and one repair pattern.

Expected coach behavior:

- The lesson file is written before state files mutate.
- `Writeback Summary` has `writeback_status: ready`, `lesson_id`, all update buckets, `current_summary`, and final `writeback_complete: true`.
- Active phrase evidence cites concrete learner production or unaided near-transfer.
- Assisted, copied, or leaked production remains `repaired` or lower.

Failure signs:

- State files mutate before the lesson file is complete.
- `current_summary` is missing or set to `none`.
- Evidence says only `practiced in lesson` without concrete learner behavior.
- The same evidence line is duplicated after replay.
