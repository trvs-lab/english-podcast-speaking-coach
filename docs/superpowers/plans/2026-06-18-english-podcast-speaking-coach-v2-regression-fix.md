# English Podcast Speaking Coach V2 Regression Fix Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Restore the useful teaching density of `SKILL-v2.md` while keeping the current split-file architecture.

**Architecture:** Keep `SKILL.md` as the runtime entrypoint and hard-rule summary. Put live coaching behavior in `INTERACTION-RULES.md`, durable state thresholds in `STATE-SCHEMAS.md` and `WRITEBACK-FORMAT.md`, and manual checks in `REGRESSION-SCENARIOS.md`. Do not change learner workspace data or durable file schemas beyond rule wording.

**Tech Stack:** Markdown skill files, `rg`, `sed`, `git diff --check`, manual regression review against session exports.

---

## Source Evidence

Use these files as the basis for implementation:

- Current entrypoint: `skills/english-podcast-speaking-coach/SKILL.md`
- Current live teaching rules: `skills/english-podcast-speaking-coach/INTERACTION-RULES.md`
- Current durable state rules: `skills/english-podcast-speaking-coach/STATE-SCHEMAS.md`
- Current writeback rules: `skills/english-podcast-speaking-coach/WRITEBACK-FORMAT.md`
- Current recovery rules: `skills/english-podcast-speaking-coach/RECOVERY-RULES.md`
- Current manual checks: `skills/english-podcast-speaking-coach/REGRESSION-SCENARIOS.md`
- Baseline backup: `/Users/wxg/Desktop/Projects/englishpod exercise/session exports/SKILL-v2.md`
- Degraded latest session: `/Users/wxg/Desktop/Projects/englishpod exercise/session exports/new/008.md`
- Better old sessions: `/Users/wxg/Desktop/Projects/englishpod exercise/session exports/old/003.md`, `/Users/wxg/Desktop/Projects/englishpod exercise/session exports/old/004.md`, `/Users/wxg/Desktop/Projects/englishpod exercise/session exports/old/005.md`

Observed regressions:

- Corrected answers introduce useful chunks such as `ran into` without explaining why they are better.
- Required details from prompts, such as `last month`, may be silently added to the natural version without pointing out the omission.
- Feedback openings repeat generic phrases such as `意思很清楚`, `意思到了`, and `整体很好`.
- Already-produced patterns are repeated in Speed round, such as practicing `I'm terrible with + noun` after multiple successful near-transfer attempts.
- Repeated spelling issues such as `convasation` and `cornor` are only mentioned in passing.
- `听过但还没练熟的表达` can become a dead archive bucket with no review action.
- The conservative writeback rule lacks a quantified threshold for when a phrase enters the active phrase bank.

## Scope

In scope:

- Restore correction completeness without making every turn verbose.
- Add required-detail omission as a first-class correction type.
- Add a spelling repair rule for repeated, high-value, or phrase-bank-bound spelling issues.
- Replace generic praise openers with contextual coach response moves.
- Add pattern-level mastery and Speed round eligibility rules to avoid repetition.
- Add review routing for unmastered but valuable expressions.
- Quantify active phrase admission thresholds.
- Add manual regression scenarios covering the `new/008.md` failures.
- Make writeback examples replayable enough to avoid guessing, without changing existing lifecycle semantics.

Out of scope:

- No rollback to monolithic `SKILL-v2.md`.
- No new lesson mode, quick/deep practice branch, or UI.
- No automatic test harness.
- No learner workspace edits under `english-coach/`.
- No broad durable schema migration, existing state rewrite, or new state file unless a later design explicitly approves it.
- No automatic promotion of every unpracticed extension expression into review.

## File Structure

Modify:

- `skills/english-podcast-speaking-coach/SKILL.md`
  - Add entrypoint reminders for required-detail omissions, spelling repair, pattern-level mastery, and active-entry evidence thresholds.

- `skills/english-podcast-speaking-coach/INTERACTION-RULES.md`
  - Add a correction completeness checklist.
  - Add required-detail omission and spelling repair handling.
  - Add contextual coach response moves.
  - Add pattern-level mastery and Speed round eligibility filtering.
  - Clarify routing from `听过但还没练熟的表达` to review when needed.

- `skills/english-podcast-speaking-coach/STATE-SCHEMAS.md`
  - Clarify active-entry thresholds and pattern-level evidence without requiring migration.
  - Clarify how review queue items represent attempted-but-unmastered expressions and spelling repair targets.

- `skills/english-podcast-speaking-coach/WRITEBACK-FORMAT.md`
  - Add writeback self-review checks for active-entry threshold, unmastered expression routing, and spelling repair evidence.
  - Replace terse update examples with replayable payload examples that include the fields needed to reconstruct phrase/review/repair items.

- `skills/english-podcast-speaking-coach/REGRESSION-SCENARIOS.md`
  - Add manual regression cases for `ran into`, omitted `last month`, repeated feedback openings, repeated pattern practice, spelling repair, review routing, and active-entry threshold.

Do not modify:

- `skills/english-podcast-speaking-coach/WORKSPACE-FORMAT.md`
- `skills/english-podcast-speaking-coach/RECOVERY-RULES.md` unless implementation discovers a direct recovery conflict.
- Any `english-coach/` learner workspace files.

## Delta Map And Acceptance Criteria

| Finding | Current failure mode | Target rule owner | Acceptance criterion |
| --- | --- | --- | --- |
| `ran into` introduced without explanation | Natural version adds a useful spoken chunk silently. | `INTERACTION-RULES.md`, `REGRESSION-SCENARIOS.md` | Every coach-added useful chunk is explained or explicitly marked optional. |
| `last month` omitted but not called out | Required prompt details are silently restored. | `INTERACTION-RULES.md`, `SKILL.md` | Missing time/place/person/object/quantity/cause/contrast is a correction bucket. |
| Robotic feedback openings | Rules say vary openings but seed generic praise stems. | `INTERACTION-RULES.md` | Feedback starts from communicative job, successful chunk, or main repair point; repeated praise stems are disallowed in close succession. |
| Repeated `I'm terrible with + noun` practice | Exact target and reusable frame are conflated. | `INTERACTION-RULES.md`, `STATE-SCHEMAS.md` | Current-session active items and covered patterns are filtered out of Speed round unless a new error appears. |
| Spelling issues treated as throwaway polish | Repeated/high-value spelling has no lane. | `INTERACTION-RULES.md`, `STATE-SCHEMAS.md` | One-off spelling stays lesson-local; repeated or expression-bound spelling can become a repair target. |
| `听过但还没练熟` becomes a dead bucket | Extension items have no routing decision. | `INTERACTION-RULES.md`, `WRITEBACK-FORMAT.md` | Attempted-but-unmastered items route to review; optional unpracticed items stay extension-only; high-value next-session candidates are recorded without fake failure status. |
| Active phrase admission is vague | `Aim for 8-12` can behave like a quota. | `STATE-SCHEMAS.md`, `WRITEBACK-FORMAT.md` | Active entries require concrete evidence; no active quota; large active lists require per-entry audit. |
| Writeback summaries are terse | Update buckets require guessing during replay. | `WRITEBACK-FORMAT.md` | Examples show full replayable payload fields for new/updated phrase, review, and repair items. |

## Implementation Tasks

### Task 1: Preflight Baseline

**Files:**

- Read: `skills/english-podcast-speaking-coach/SKILL.md`
- Read: `skills/english-podcast-speaking-coach/INTERACTION-RULES.md`
- Read: `skills/english-podcast-speaking-coach/STATE-SCHEMAS.md`
- Read: `skills/english-podcast-speaking-coach/WRITEBACK-FORMAT.md`
- Read: `skills/english-podcast-speaking-coach/REGRESSION-SCENARIOS.md`
- Read: `/Users/wxg/Desktop/Projects/englishpod exercise/session exports/SKILL-v2.md`
- Read: `/Users/wxg/Desktop/Projects/englishpod exercise/session exports/new/008.md`

- [ ] **Step 1: Check the working tree**

Run:

```bash
git status --short
```

Expected: only intentional plan or skill documentation changes appear. If unrelated user changes exist, inspect them and do not revert them.

- [ ] **Step 2: Confirm the relevant headings exist**

Run:

```bash
rg -n "^## |^### " skills/english-podcast-speaking-coach/SKILL.md skills/english-podcast-speaking-coach/INTERACTION-RULES.md skills/english-podcast-speaking-coach/STATE-SCHEMAS.md skills/english-podcast-speaking-coach/WRITEBACK-FORMAT.md skills/english-podcast-speaking-coach/REGRESSION-SCENARIOS.md
```

Expected headings include:

```text
skills/english-podcast-speaking-coach/SKILL.md:## Core Principles
skills/english-podcast-speaking-coach/SKILL.md:## Lesson Flow
skills/english-podcast-speaking-coach/INTERACTION-RULES.md:## Learner-Facing Templates
skills/english-podcast-speaking-coach/INTERACTION-RULES.md:## Free Retelling
skills/english-podcast-speaking-coach/INTERACTION-RULES.md:## Speed Round
skills/english-podcast-speaking-coach/INTERACTION-RULES.md:## Correction
skills/english-podcast-speaking-coach/INTERACTION-RULES.md:## Lesson-End Expression Areas
skills/english-podcast-speaking-coach/STATE-SCHEMAS.md:### Mastery Lifecycle
skills/english-podcast-speaking-coach/WRITEBACK-FORMAT.md:## Writeback Self-Review
```

- [ ] **Step 3: Locate the `new/008.md` failure examples**

Run:

```bash
rg -n "ran into|last month|convasation|cornor|I'm terrible with|听过但还没练熟|business card|Don't worry about it" "/Users/wxg/Desktop/Projects/englishpod exercise/session exports/new/008.md"
```

Expected: matches include the retelling correction around `ran into`, the near-transfer prompt with `last month`, and repeated `I'm terrible with...` practice.

- [ ] **Step 4: Commit only if Task 1 required no edits**

No commit is required for read-only preflight. If this task discovers stale plan assumptions, update this plan before continuing.

- [ ] **Step 5: Confirm the delta map still matches the source evidence**

Read the `Delta Map And Acceptance Criteria` section above and confirm each row maps to a concrete source observation from `SKILL-v2.md`, `new/008.md`, old sessions, or the screenshot feedback. If any row is unsupported, remove or rewrite it before implementation.

### Task 2: Restore Correction Completeness

**Files:**

- Modify: `skills/english-podcast-speaking-coach/INTERACTION-RULES.md`
- Modify: `skills/english-podcast-speaking-coach/SKILL.md`
- Modify: `skills/english-podcast-speaking-coach/REGRESSION-SCENARIOS.md`

- [ ] **Step 1: Add required-detail omission to the entrypoint**

In `SKILL.md`, update the `Lesson Flow` bullet that starts with `Use problem-triggered correction` so it explicitly includes missing required details:

```markdown
- Use problem-triggered correction: brief feedback for natural answers, full correction for transferable problems. Full correction has priority when the learner makes Chinglish, lesson-chunk, tense, article, collocation, linkage, information-order errors, or omits required prompt details such as time, place, person, object, quantity, cause, or contrast.
```

- [ ] **Step 2: Add a correction completeness checklist**

In `INTERACTION-RULES.md`, inside `## Correction`, add a short checklist after the introductory paragraph:

```markdown
Before sending a natural version, compare it with the learner's answer and the prompt. Explain every meaningful difference in one of these buckets:

- required prompt detail omitted or changed: time, place, person, object, quantity, cause, contrast, or relationship;
- lesson chunk or spoken upgrade introduced by the coach;
- Chinglish replacement or information-order repair;
- tense, article, countability, preposition, word order, collocation, linkage, or punctuation that affects understanding or transfer;
- spelling repair when repeated, high-frequency, or attached to a phrase-bank candidate.

If the natural version adds a useful expression such as `ran into`, explain why that expression is being introduced. If the natural version restores a prompt detail such as `last month`, explicitly say the learner left that detail out.
```

- [ ] **Step 3: Add spelling repair handling**

In `INTERACTION-RULES.md`, inside `## Correction`, add:

```markdown
Spelling is not the main goal of speaking practice, but repeated or high-value spelling errors deserve a separate, brief note. Do not bury them inside grammar feedback. If the same useful word is misspelled more than once in a lesson, or the misspelled word appears in an active/review expression candidate, create a learner-specific spelling repair target. Keep the spoken practice moving: explain the spelling once, then return to the expression task.
```

- [ ] **Step 4: Add Common Mistakes reminders**

In `SKILL.md` and `INTERACTION-RULES.md`, add concise common mistakes:

```markdown
- Silently adding required prompt details to the natural version without telling the learner what was missing.
- Introducing a useful coach-added phrase such as `ran into` without explaining its meaning, use case, or why it improves the learner's wording.
- Treating repeated high-value spelling errors as throwaway polish instead of a brief spelling repair target.
```

- [ ] **Step 5: Add regression scenario for the `ran into` and `last month` case**

In `REGRESSION-SCENARIOS.md`, add a scenario with this setup:

```markdown
## 10. Retelling Correction Completeness

Setup:

- Prompt asks the learner to say they met someone at a workshop last month.
- Learner says: `I met Anna near the coffee shop, we had met at a workshop, but his name had just slipped my mind.`
- Coach natural version uses `I ran into someone at a coffee shop. We had met at a workshop last month...`

Expected coach behavior:

- Explain `ran into` as a natural spoken chunk for unexpectedly meeting someone.
- Point out that `last month` was omitted from the learner answer.
- Correct `near` versus `at`, pronoun mismatch, and comma splice if present.
- Repair only the highest-value 1-2 issues through near-transfer.

Failure signs:

- Natural version adds `last month` silently.
- Natural version introduces `ran into` without explanation.
- Coach only says the sentence is more natural without identifying the changed learning points.
```

- [ ] **Step 6: Verify correction rules are present**

Run:

```bash
rg -n "required prompt detail|last month|ran into|Spelling is not the main goal|spelling repair target|Silently adding required prompt details|Retelling Correction Completeness" skills/english-podcast-speaking-coach
```

Expected: matches in `SKILL.md`, `INTERACTION-RULES.md`, and `REGRESSION-SCENARIOS.md`.

- [ ] **Step 7: Check Markdown formatting**

Run:

```bash
git diff --check
```

Expected: no output.

- [ ] **Step 8: Commit**

Run:

```bash
git add skills/english-podcast-speaking-coach/SKILL.md skills/english-podcast-speaking-coach/INTERACTION-RULES.md skills/english-podcast-speaking-coach/REGRESSION-SCENARIOS.md
git commit -m "docs: restore speaking correction completeness"
```

Expected: commit succeeds and only the listed files are included.

### Task 3: Make Feedback Sound Like Coaching

**Files:**

- Modify: `skills/english-podcast-speaking-coach/INTERACTION-RULES.md`
- Modify: `skills/english-podcast-speaking-coach/REGRESSION-SCENARIOS.md`

- [ ] **Step 1: Replace generic opener guidance with coach response moves**

In `INTERACTION-RULES.md`, under `## Learner-Facing Voice`, add:

```markdown
Start feedback by responding to what the learner was trying to do in the scene, not by grading first. Prefer one concrete coach move:

- name the communicative job: `这句是在缓和尴尬，语气已经对了。`
- name the successful chunk: `你把确认语气抓住了，后面只要去掉 Do you。`
- name the one repair priority: `这里最大的问题不是语法，是少了时间信息。`
- continue the scene when the answer is natural: `这句可以直接对 Anna 说。我接着演她。`

Avoid using generic praise openers such as `意思很清楚`, `意思到了`, or `整体很好` in consecutive feedback turns. If the answer is mostly natural, skip broad praise and move directly to the one useful adjustment or the next role line.
```

- [ ] **Step 2: Update learner-facing templates**

In `## Learner-Facing Templates`, revise the mostly-natural and meaningful-error templates so their first placeholder is contextual:

```markdown
[contextual coach opening tied to the learner's communicative job or main repair]

[natural version]

[one useful explanation or natural transition into the next role-play line]
```

For full correction, require the opening to name the main issue instead of using a fixed praise phrase:

```markdown
[contextual opening that names the main issue: missing detail, Chinglish, lesson chunk, tense, linkage, or tone]
```

- [ ] **Step 3: Add an anti-repetition common mistake**

In `INTERACTION-RULES.md`, add:

```markdown
- Reusing generic feedback openers for several turns in a row instead of responding to the learner's actual communicative move.
```

- [ ] **Step 4: Add regression scenario**

In `REGRESSION-SCENARIOS.md`, add:

```markdown
## 11. Coach Voice And Feedback Openings

Setup:

- Three consecutive learner replies are mostly correct but each has a different teaching point: one missing time detail, one mixed question tag, one spelling issue.

Expected coach behavior:

- Each feedback opening names the actual job or repair point.
- The coach does not repeat `意思很清楚`, `意思到了`, or `整体很好` across consecutive turns.
- The answer still stays concise when the issue is minor.
- Positive feedback is allowed when it is specific to the learner's actual sentence.

Failure signs:

- Feedback starts with the same generic praise pattern several times.
- The opening sounds like a grading report instead of a coach responding inside the scene.
```

- [ ] **Step 5: Verify voice rules are present**

Run:

```bash
rg -n "communicative job|Avoid using generic praise|contextual coach opening|Coach Voice And Feedback Openings|generic feedback openers" skills/english-podcast-speaking-coach/INTERACTION-RULES.md skills/english-podcast-speaking-coach/REGRESSION-SCENARIOS.md
```

Expected: all phrases match.

- [ ] **Step 6: Check Markdown formatting**

Run:

```bash
git diff --check
```

Expected: no output.

- [ ] **Step 7: Commit**

Run:

```bash
git add skills/english-podcast-speaking-coach/INTERACTION-RULES.md skills/english-podcast-speaking-coach/REGRESSION-SCENARIOS.md
git commit -m "docs: improve speaking coach feedback voice"
```

Expected: commit succeeds and only the listed files are included.

### Task 4: Add Pattern-Level Mastery And Speed Round Eligibility

**Files:**

- Modify: `skills/english-podcast-speaking-coach/SKILL.md`
- Modify: `skills/english-podcast-speaking-coach/INTERACTION-RULES.md`
- Modify: `skills/english-podcast-speaking-coach/STATE-SCHEMAS.md`
- Modify: `skills/english-podcast-speaking-coach/REGRESSION-SCENARIOS.md`

- [ ] **Step 1: Add pattern-level mastery principle**

In `SKILL.md`, under `## Core Principles`, add:

```markdown
**Mastery can belong to a reusable pattern, not only an exact sentence.** When the learner has produced the same sentence frame with different meaningful slots in unaided near-transfer, treat the frame as covered. Do not keep testing the same frame by swapping only one noun unless the new slot exposes a real grammar, collocation, or meaning problem.
```

- [ ] **Step 2: Add pattern evidence to state schema wording**

In `STATE-SCHEMAS.md`, after the active status definition, add:

```markdown
Pattern-level active evidence is valid when the learner produces the same reusable frame in at least two meaningfully different unaided contexts, or repairs it after correction and later produces it unaided in a new context. Slot-only repetition, such as changing `names` to `faces` inside an already-mastered frame, does not by itself create a new review need.
```

- [ ] **Step 3: Add Speed round eligibility gate**

In `INTERACTION-RULES.md`, inside `## Speed Round`, add this before the list of items to test:

```markdown
Before choosing Speed round prompts, filter out items that are already covered at pattern level in the current session. An item is covered when the learner has produced the reusable frame unaided in a new context and later shown the same frame with a different meaningful slot, unless the latest attempt exposed a new error. Do not retest by changing only a noun, person, or place if the target frame is already active.
```

- [ ] **Step 4: Add live target queue priority**

In `INTERACTION-RULES.md`, inside `## Speed Round`, add:

```markdown
Choose Speed round items in this order: unproduced primary targets, attempted-but-unmastered targets, repaired targets needing unaided proof, and learner-specific repair targets. Do not retest current-session active exact chunks or active pattern families while higher-priority unresolved items remain. If an already-active item is intentionally spaced later, use a genuinely new communicative situation and do not crowd out unresolved targets.
```

- [ ] **Step 5: Add coverage tracking reminder**

In `INTERACTION-RULES.md`, inside `## Coverage Tracking`, add:

```markdown
Track both exact chunks and reusable frames. If the exact phrase is `I'm terrible with faces`, the reusable frame may be `I'm terrible with + noun`. Once the frame is active, related noun swaps should usually be treated as vocabulary exposure or light review, not a fresh Speed round target.
```

- [ ] **Step 6: Add regression scenario**

In `REGRESSION-SCENARIOS.md`, add:

```markdown
## 12. Pattern-Level Mastery And No Repetition

Setup:

- Learner first says `I'm terrible with names too` after correction.
- Learner then produces `I'm terrible with directions too`.
- Learner then produces `I'm terrible with numbers` unaided.
- The lesson also contains `I'm terrible with faces`.

Expected coach behavior:

- Mark the frame `I'm terrible with + noun` as covered at pattern level.
- Do not add a final Speed round item that only swaps in `faces`.
- If `faces` is useful, list it as vocabulary or a light example, not as an unresolved target.

Failure signs:

- Coach tests `I'm terrible with faces` after the frame has already been shown across meaningful slots.
- Coach treats every noun swap as a separate unmastered expression.
```

- [ ] **Step 7: Verify pattern-level rules are present**

Run:

```bash
rg -n "pattern-level|reusable frame|terrible with \\+ noun|Slot-only repetition|No Repetition|Do not retest by changing only|current-session active|unproduced primary targets" skills/english-podcast-speaking-coach
```

Expected: matches in `SKILL.md`, `INTERACTION-RULES.md`, `STATE-SCHEMAS.md`, and `REGRESSION-SCENARIOS.md`.

- [ ] **Step 8: Check Markdown formatting**

Run:

```bash
git diff --check
```

Expected: no output.

- [ ] **Step 9: Commit**

Run:

```bash
git add skills/english-podcast-speaking-coach/SKILL.md skills/english-podcast-speaking-coach/INTERACTION-RULES.md skills/english-podcast-speaking-coach/STATE-SCHEMAS.md skills/english-podcast-speaking-coach/REGRESSION-SCENARIOS.md
git commit -m "docs: add speaking pattern mastery rules"
```

Expected: commit succeeds and only the listed files are included.

### Task 5: Close Review Queue And Writeback Routing Gaps

**Files:**

- Modify: `skills/english-podcast-speaking-coach/INTERACTION-RULES.md`
- Modify: `skills/english-podcast-speaking-coach/STATE-SCHEMAS.md`
- Modify: `skills/english-podcast-speaking-coach/WRITEBACK-FORMAT.md`
- Modify: `skills/english-podcast-speaking-coach/REGRESSION-SCENARIOS.md`

- [ ] **Step 1: Quantify active phrase admission**

In `STATE-SCHEMAS.md`, after the existing rule that active requires unaided production, add:

```markdown
Active phrase bank admission requires one of these evidence paths:

- exact chunk: clean target-hidden learner production in the communicative task, or later unaided near-transfer after correction, reveal, or sentence-frame help;
- pattern family: at least two unaided productions with different meaningful slot values or contexts, with no unresolved grammar, collocation, or meaning error;
- repaired item: correction or hint-assisted repair followed by later unaided production in a new related context.

Do not mark an item active from copied correction, sentence-frame completion, answer reveal, English menu, coach-modeled exact chunk, ambiguous evidence, or slot-only repetition. The learner can own an exact chunk without yet owning the broader pattern family.
```

- [ ] **Step 2: Add learner-facing routing for unmastered expressions**

In `INTERACTION-RULES.md`, under `### 听过但还没练熟的表达`, add:

```markdown
This section is not automatically a dead archive. Route items this way:

- high-value core target selected for the lesson but not practiced because time ran out -> record as a next-session candidate in the lesson-end writeback `current_summary`, or ask the learner whether to keep it; do not label it `needs_review` unless the learner attempted it or it was intentionally assigned as review;
- coach-supplied natural alternative that is useful but optional -> keep only in `听过但还没练熟的表达`;
- learner attempted but did not master -> place in `下次还要再练的表达` and review queue;
- repeated high-value spelling issue -> add as a spelling repair target when it affects an expression candidate.

If unsure whether an exposed expression should return next time, ask the learner briefly at lesson end or keep it as an optional next-session candidate. Do not turn every exposed expression into review queue debt.
```

- [ ] **Step 3: Add review queue schema clarification**

In `STATE-SCHEMAS.md`, inside `### Review Queue Item`, add:

```markdown
Review queue items may represent unmastered core targets, practiced extension expressions, learner-specific repair targets, or high-value spelling repair targets. Do not add every coach-shown expression. Add only items that should actively return because they are central, attempted-but-unmastered, repeatedly misspelled, or explicitly requested by the learner.
```

- [ ] **Step 4: Make writeback update examples replayable**

In `WRITEBACK-FORMAT.md`, replace the terse example bucket shape with examples that include enough information to recreate the durable item without guessing. Use existing schema fields; do not invent a new file format.

For phrase bank updates, the example must include:

```markdown
- id: PB-0012
  action: add
  destination: phrase-bank/workplace.md
  chunk: I was wondering if...
  meaning_zh: 我想知道是否...
  use_case: polite scheduling request
  learner_example: I was wondering if we could move the meeting.
  recall_prompt_zh: 礼貌询问能不能改会议时间
  status: active
  evidence:
    - 20260611-002: unaided near-transfer in scheduling scenario
```

For review queue updates, the example must include:

```markdown
- id: RQ-0007
  action: add
  chunk: How does next Monday work for you?
  meaning_zh: 下周一你方便吗？
  reason: needed sentence frame before producing it
  status: needs_review
  next_review_hint: scheduling or appointment lesson
  evidence:
    - 20260611-002: sentence-frame-assisted production
```

For repair bank updates, the example must include:

```markdown
- id: RB-0004
  action: add
  pattern: repeated spelling of conversation
  issue_type: spelling
  status: needs_review
  next_near_transfer: use conversation in a new meeting or networking sentence
  evidence:
    - 20260617-008: misspelled conversation as convasation
```

- [ ] **Step 5: Add writeback self-review checks**

In `WRITEBACK-FORMAT.md`, inside `## Writeback Self-Review`, add checks:

```markdown
9. **Active threshold:** every phrase-bank addition satisfies one active evidence path: exact target-hidden production, repair plus later unaided production, or valid pattern-level evidence.
10. **Unmastered routing:** attempted-but-unmastered targets are in review queue updates; optional unpracticed expressions are explicitly left as extension-only or next-session candidates with a reason.
11. **Spelling repairs:** repeated high-value spelling errors are either captured as repair/review items or explicitly treated as minor one-off polish.
12. **Replayable payload:** new phrase, review, and repair updates include the fields needed to reconstruct the durable item without guessing from surrounding prose.
```

- [ ] **Step 6: Add regression scenario**

In `REGRESSION-SCENARIOS.md`, add:

```markdown
## 13. Unmastered Expression Routing And Active Threshold

Setup:

- Coach shows `business card` as a better version of `contact card`, but the learner does not practice it again.
- Coach shows `Don't worry about it`, but learner only copied or saw it once.
- Learner produces one target unaided in the original scene but never in a second context.

Expected coach behavior:

- High-value unmastered core or attempted expressions enter `下次还要再练的表达` and review queue.
- Optional coach alternatives can stay in `听过但还没练熟的表达` with no review state.
- Unpracticed but useful next-session candidates are recorded without falsely marking learner failure.
- Active phrase bank additions meet the quantified evidence threshold.
- Writeback update buckets include replayable payloads.

Failure signs:

- `听过但还没练熟的表达` receives no routing decision.
- One original-scene production is treated as active without later unaided evidence.
- Coach-shown alternatives are all pushed into review regardless of importance.
- Writeback updates are pointer-only and require guessing fields from prose.
```

- [ ] **Step 7: Verify routing and threshold rules are present**

Run:

```bash
rg -n "Active phrase bank admission|pattern family|dead archive|next-session candidate|Unmastered Expression Routing|Active threshold|Spelling repairs|Replayable payload|replayable payloads" skills/english-podcast-speaking-coach
```

Expected: matches in `INTERACTION-RULES.md`, `STATE-SCHEMAS.md`, `WRITEBACK-FORMAT.md`, and `REGRESSION-SCENARIOS.md`.

- [ ] **Step 8: Check Markdown formatting**

Run:

```bash
git diff --check
```

Expected: no output.

- [ ] **Step 9: Commit**

Run:

```bash
git add skills/english-podcast-speaking-coach/INTERACTION-RULES.md skills/english-podcast-speaking-coach/STATE-SCHEMAS.md skills/english-podcast-speaking-coach/WRITEBACK-FORMAT.md skills/english-podcast-speaking-coach/REGRESSION-SCENARIOS.md
git commit -m "docs: clarify speaking coach review routing"
```

Expected: commit succeeds and only the listed files are included.

### Task 6: Final Manual Regression Review

**Files:**

- Read: all modified skill files.
- Read: `/Users/wxg/Desktop/Projects/englishpod exercise/session exports/new/008.md`
- Read: `/Users/wxg/Desktop/Projects/englishpod exercise/session exports/SKILL-v2.md`

- [ ] **Step 1: Run static checks**

Run:

```bash
git diff --check
```

Expected: no output.

Run:

```bash
rg -n "T[B]D|TO[D]O|implement late[r]|fill in detail[s]" skills/english-podcast-speaking-coach docs/superpowers/plans/2026-06-18-english-podcast-speaking-coach-v2-regression-fix.md
```

Expected: no output.

- [ ] **Step 2: Verify no learner workspace files changed**

Run:

```bash
git status --short
```

Expected: no paths under `english-coach/`. Only intended skill docs or plan files should appear if work has not yet been committed.

- [ ] **Step 3: Run `new/008.md` failure checklist manually**

Inspect the final rules and confirm a future coach would be required to:

```text
- explain `ran into` when the natural version introduces it;
- point out omitted `last month`;
- explain repeated spelling repairs such as `convasation` and `cornor` separately when they matter;
- avoid repeated generic feedback openings;
- avoid retesting `I'm terrible with + noun` by only swapping `faces`;
- route `business card` and `Don't worry about it` according to importance and practice evidence;
- keep active phrase bank additions within the quantified evidence threshold.
```

- [ ] **Step 4: Run v2 preservation checklist manually**

Compare final current rules against `SKILL-v2.md` and confirm the following v2 strengths remain:

```text
- active recall before answer reveal;
- full correction for transferable errors;
- near-transfer instead of copy-the-answer practice;
- conservative mastery evidence;
- no active-entry quota: `Aim for 8-12` cannot override evidence quality;
- lesson-end areas with meaning, use case, learner example, and recall prompt;
- writeback before state mutation;
- no mixing of phrase bank, review queue, extension expressions, and vocabulary.
```

- [ ] **Step 5: Commit any final regression scenario or wording adjustments**

Run:

```bash
git add skills/english-podcast-speaking-coach docs/superpowers/plans/2026-06-18-english-podcast-speaking-coach-v2-regression-fix.md
git commit -m "docs: plan speaking coach v2 regression fixes"
```

Expected: commit succeeds if there are final uncommitted documentation changes. If all prior tasks already committed their changes and only the plan remains, the commit contains only this plan.

## Self-Review Checklist

Before executing this plan, verify:

- Every user-reported regression maps to at least one task.
- No task modifies learner workspace files.
- Active-entry thresholds do not require schema migration.
- Review routing distinguishes attempted-but-unmastered targets, optional coach alternatives, and next-session candidates without fake learner failure.
- Spelling repair remains lightweight and does not turn the skill into a writing tutor.
- Pattern-level mastery avoids repetition without hiding genuine unresolved errors.
- Writeback examples are replayable enough without changing the durable lifecycle.
- Regression scenarios include the concrete `new/008.md` failures.
