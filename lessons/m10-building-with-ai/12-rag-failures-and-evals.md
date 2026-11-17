---
id: m10-12-rag-evals
module: m10-building-with-ai
slug: rag-failures-and-evals
title: "RAG Failure Modes, Test Sets and Evals"
order: 12
duration_min: 25
is_free: false
mission: true
volatility: low
last_reviewed: 2026-11-16
---

## Key learnings
- Locate a RAG failure on the ladder before changing anything
- Build an eval set from real questions, written before you see the output
- Score retrieval and the answer separately, because they fail separately
- Keep a scoreboard so a fix that breaks five things cannot hide
- Use an AI judge only after checking it against your own labels

## Lesson

### The pain

A team demos their student-services assistant. Three answers are excellent; the fourth is confident, fluent and completely wrong about the hostel fee. "Edge case," says the team.

The next week better chunking fixes the hostel fee. Nobody re-asks the other three, and two have quietly broken. Two weeks later reranking fixes one and breaks the refund question. After six weeks of real work they cannot say whether the system is better than on day three, and their whole evidence is "it feels better now".

### The idea

**An eval set is a fixed list of real questions, each with the answer you expect and the source it should come from, scored the same way every time. Without it you have opinions; with it you have a number that moves when you change something.**

Thirty to fifty questions is enough, and less work than one week of guessing. The rule that makes it worth anything: **write the eval set before you look at the output.** Build it afterwards and you will write questions the system already passes — not dishonestly, but because those are the ones that come to mind. The set must be able to fail.

### The analogy

A cook dips the tasting spoon in the sambar, nods, and sends it out. Usually fine. A canteen serving two hundred plates checks the same five things daily: salt, sourness, consistency, chilli, temperature. Because it is the same five in the same order, "today's sambar is thinner" becomes a fact rather than a mood.

**Where the analogy breaks, twice.** A palate adapts: after three weeks of thin sambar, thin tastes normal. An eval set does not, which is better. But a checklist only finds what is on it, so real user complaints keep their place beside the scoreboard forever.

### Picture it

```
  a wrong answer
        │
        ▼
  Is the fact in the store at all?
        │ no ──▶ INGESTION       (ch 9)
        ▼ yes
  Does search return it in the top 40?
        │ no ──▶ RETRIEVAL       (ch 10, 11)
        ▼ yes
  Is it in the 5 chunks the model saw?
        │ no ──▶ RANKING         (ch 11)
        ▼ yes
  Did the answer use it, and cite it truly?
        │ no ──▶ PROMPT, SCHEMA, REFUSAL RULE
        ▼ yes
  Then the chunk itself is wrong or stale.
```

Four questions, asked in order, each with a different fix. Running down this ladder takes two minutes and saves days of changing the wrong layer.

| Failure mode | Fixed in |
|---|---|
| Not stored at all | Chapter nine |
| Stored, search missed it | Chapters ten and eleven |
| Right chunk, buried among distractors | Reranking, smaller top-k |
| Chunk holds half the answer | Chunking, contextual prefix |
| Answered from a superseded document | Dates, `superseded_by` |
| Two sources disagree silently | Prefer newest, name the conflict |
| Answered when nothing contained it | Refusal rule and schema field |
| Refused when the answer was there | Loosen that fix; keep both rows |
| Cited a document lacking the claim | Chapter eleven's verification |
| Right answer, forbidden document | Pre-filtering — an incident |

### Where it lives

The eval set is a file in your repository, versioned and reviewed in a pull request like any other change. Each row records four things: the question, the expected answer, the chunk id that should be retrieved, and the kind of question it is. That chunk id makes retrieval scoring possible with no human in the loop.

Cover nine kinds deliberately: plain lookup, paraphrase, exact code, two-document, unanswerable, stale trap, permission trap, regional-language, and messy typos. M06's CI runs the set on any pull request touching retrieval.

### Words to own

| Term | Plain meaning | Example |
|---|---|---|
| Eval set | Fixed questions with expected answers | 40 rows in a CSV |
| Ground truth | The answer you know is right | ₹42,000, from fee-2026-04 |
| Recall at k | Did the right chunk reach the top k | 0.78 at k=5 |
| Regression | A change that breaks something that worked | Fixed 3, broke 5 |
| Abstention accuracy | Refusing exactly when it should | 8 of 10 unanswerables |
| Unanswerable question | One whose correct answer is "I do not know" | Refund policy, unwritten |
| Stale trap | A question an obsolete document also matches | Last year's fee |
| LLM-as-judge | A model scoring answers against a rubric | Needs calibrating |
| Rubric | Written criteria the judge applies | Supported? Answered? |
| Scoreboard | Run history, one row per config | Catches regressions |

### Tradeoffs and gotchas

**A set written after seeing the output scores ninety percent on day one and never moves.** A first run above about eighty percent should make you suspect the set.

**Forty questions will not cover your product, and that is fine.** It is a regression net, not proof of correctness. Promote real user failures in; otherwise freeze the set, because editing a failed question is cheating with extra steps.

**Unanswerable rows are the most valuable and most often missing.** Without refusal cases you cannot detect the failure that embarrasses you in front of a mentor.

**An uncalibrated AI judge measures fluency**: judges prefer longer, more confident answers.

**Keep retrieval and answer scores separate.** If the right chunk reaches the top five sixty percent of the time, sixty percent is your ceiling. A blended percentage hides which half broke.

### Verify it

- **Check the provenance of every question**: "I made it up after the demo" is a weak row.
- **Count your unanswerable rows.** None means the set cannot measure abstention.
- **Confirm the set can fail.** Remove a document or cripple keyword search; the score must drop.
- **Calibrate your judge once, in writing:** twenty hand labels against twenty judge labels; fix the rubric below about eighty percent agreement.
- **Keep one scoreboard and re-read it before every demo** — date, what changed, recall at five, answer accuracy, abstention accuracy, citation verification rate, p95 latency. If the newest row is not the best, say so out loud.

Red flags: an eval set created after seeing the output; every question invented by the team; no unanswerable rows; no expected chunk id, so retrieval cannot be scored; a single blended accuracy number; an AI judge never compared with human labels; a question edited because the system failed it; no run history; "it feels better" as evidence.

## Walkthrough
1. Collect real questions first. Scroll your WhatsApp group, your counter conversations, your first users' messages. Copy thirty of them verbatim, including typos and mixed script.
2. Write the expected answer for each one yourself, from the source document, before running anything through your system.
3. Record the chunk id that should be retrieved for each. Where you cannot find one, the honest expected answer is a refusal — mark it unanswerable.
4. Add the missing kinds deliberately: at least three unanswerable, two stale traps, two permission traps, three regional-language, two exact-code and two multi-document questions.
5. Freeze the file in your repository and open it as a pull request so a teammate reviews it as a change, per M01.
6. Write the runner: for each row, retrieve, generate, and record the retrieved chunk ids, the answer, the citation and whether it refused.
7. Score retrieval first and only: recall at five across the whole set. Write the number down. This is your ceiling.
8. Hand-label twenty answers into correct, partial and wrong. Then run an AI judge with a written rubric over the same twenty and record the agreement rate.
9. Run the diagnostic ladder on your five worst failures and write the layer each one belongs to. Count how many are ingestion or retrieval rather than prompt problems.
10. Fix exactly one thing — the layer with the most failures — and re-run the whole set. Record both rows in your scoreboard, including anything that got worse.
11. Promote one new failure found by a real user into the set, and leave the original rows untouched.

## Talk to AI
```prompt
Help me build an eval set for my retrieval feature, and be strict with me about where my questions came from.

MY PRODUCT: [one sentence]
MY USER TYPES AND WHAT EACH MAY SEE: [list them]
MY DOCUMENTS: [what they cover, and which ones supersede others]

REAL QUESTIONS MY USERS HAVE ACTUALLY ASKED - verbatim, with typos, in whatever language they used:
1. [question]
2. [question]
3. [...]

THINGS MY DOCUMENTS DO NOT COVER: [list them honestly - these become refusal cases]
DOCUMENTS THAT REPLACED OLDER ONES: [list the pairs]

Deliver all of this:

1. Tell me which of my questions are too easy or were obviously written by someone who already knows the answer, and why.
2. Build the eval set as a table with four columns: question, expected answer, expected chunk id, question kind. Use my real questions as the base.
3. Fill the gaps in coverage, labelling each addition: paraphrase, exact code, multi-document, unanswerable, stale trap, permission trap, regional language, messy input. Tell me how many of each I should have in a 40-row set.
4. For every unanswerable row, write the exact refusal wording that would count as correct.
5. For every permission trap, state which user type must ask it and what a leak would look like.
6. Design the scoring: what I compute for retrieval, what I compute for factual answers, what needs a human label, and what an AI judge can do.
7. Write the judge rubric in full, as I would paste it, and tell me exactly how to calibrate it against my own labels and what agreement rate is acceptable.
8. Design my scoreboard: the exact columns, and the rule for when a change is a regression rather than an improvement.
9. Tell me how to deliberately break my system to prove this set actually detects failures.
10. Finally: name the failure mode this set still cannot detect, and what else I need besides an eval set to catch it.
```

## Mission

**Objective.** A frozen eval set of at least thirty real questions over your own startup's data, a runner that scores retrieval and answers separately, a scoreboard with at least two rows, and one fix you can prove helped.

**Steps.**
1. **Collect** — gather thirty real questions from real people, verbatim. Not invented. Record where each came from.
2. **Ground** — write the expected answer and the expected chunk id for each, from the source, before you run anything.
3. **Cover** — add the missing kinds: at least three unanswerable, two stale traps, two permission traps, three regional-language, two exact-code, two multi-document.
4. **Freeze** — commit the set and open it as a pull request for a teammate to review, with the provenance of each question visible.
5. **Direct** — brief AI to build the runner. Acceptance criteria must include recall at five reported separately from answer accuracy.
6. **Baseline** — run the whole set and record every column of your scoreboard, including the numbers you are embarrassed by.
7. **Diagnose** — run the ladder on your five worst failures and record which layer each belongs to.
8. **Fix one thing** — the layer with the most failures. One change only.
9. **Re-run** — record the second scoreboard row, and name anything that got worse as well as better.
10. **Calibrate** — hand-label twenty answers, compare with your AI judge, and write down the agreement rate.

**Upload.**
- Your eval set file, all rows visible, with the provenance column → attached file or screenshot
- The pull request where a teammate reviewed the set → `repo_url`
- Your scoreboard with at least two rows and all columns → screenshot
- Your diagnostic ladder results for five failures, with the layer named for each
- Evidence of the one fix: what you changed, and the before and after numbers
- Your judge calibration: twenty hand labels, twenty judge labels, and the agreement rate
- One regression you caused and caught → the row that got worse
- The AI conversation → share link in `ai_chat_url`
- Reflection, 150 words max → what the eval set told you that the demo did not, and one thing the AI got wrong

**Self-check.** Tick every box before you submit.
- [ ] At least thirty questions, and I can say where each one came from
- [ ] The questions were written before I looked at my system's output
- [ ] At least three questions are genuinely unanswerable, with a refusal as the expected answer
- [ ] At least two permission traps, each asked as a user who may not see the source
- [ ] At least three questions in a regional language, in my users' real wording
- [ ] Every row has an expected chunk id, so retrieval is scored without a human
- [ ] Recall at five is reported separately from answer accuracy
- [ ] I broke something deliberately and confirmed the score dropped
- [ ] I hand-labelled twenty answers before trusting any AI judge
- [ ] My scoreboard has at least two rows and shows what got worse as well as better
- [ ] I changed exactly one thing between the two runs
- [ ] No question was edited because my system failed it

**Stretch.** Wire the runner into CI so it runs on any pull request touching retrieval, per M06, and make the pull request fail if recall at five drops. Then write 100 words on what changes about your team's behaviour when the number is checked by a machine instead of by whoever remembers.

## Stuck? Ask AI
```prompt
My RAG answer is wrong and I do not know which part to fix. Walk me down a diagnostic ladder: is the fact in my store, did search return it, was it in the chunks the model saw, did the answer use it. For each step, tell me exactly what to print or query to find out, and what the fix is at that layer.
```

```prompt
Explain why an eval set written after watching my system's output is almost worthless, even though every question in it is real. Then tell me how to tell whether my existing set has this problem, and what score on the first run should make me suspicious.
```

```prompt
Explain LLM-as-judge for scoring answers: what it is good at, what biases it has, and why I must calibrate it against my own labels first. Then write me a rubric for judging whether a retrieval answer is supported by its cited chunk, and tell me what agreement rate with my own labels I should insist on.
```

## Quiz

### Q1 · L4
Why must an eval set be written before you see the system's output?
- (a) It saves time later
- (b) Otherwise you unconsciously write questions it already passes ✅
- (c) Providers cache questions after the first run
- (d) Expected answers change once you see real output
> Watching the demo shapes which questions come to mind. A set built afterwards scores high on day one and never moves, because it selected for what already worked.

### Q2 · L3
Recall at five is sixty percent. What does that tell you about prompt improvements?
- (a) They could raise accuracy to about ninety percent
- (b) Sixty percent is the ceiling, because the evidence is absent in the rest ✅
- (c) Nothing; recall and accuracy are unrelated
- (d) The prompt is the cause of the sixty percent
> If the correct chunk never reaches the context, no wording can recover it. Retrieval is measured separately precisely so this ceiling is visible.

### Q3 · L4
A change fixes three questions and breaks five. What catches this?
- (a) A demo to a mentor
- (b) A scoreboard with one row per configuration ✅
- (c) A larger top-k
- (d) An AI judge with a stricter rubric
> In a demo, three fixes look like progress. A run history over a frozen set makes the net loss visible immediately and is the whole reason to keep one.

### Q4 · L5
Your AI judge agrees with your own labels on twelve of twenty answers. What should you do?
- (a) Accept it; eighty percent agreement is unrealistic
- (b) Fix the rubric and recalibrate before trusting any judge number ✅
- (c) Use the judge for retrieval scoring instead
- (d) Increase the judge model's temperature for variety
> Sixty percent agreement means the judge is scoring its own taste, most likely rewarding fluency and length. Its numbers are not evidence until the rubric agrees with a human.

### Q5 · L4
An answer is correct but was taken from a document this user may not see. Where does this failure belong?
- (a) The prompt; add a rule about confidentiality
- (b) Retrieval; the permission pre-filter was missing ✅
- (c) Ingestion; the document should not have been ingested
- (d) The judge; it should have marked the answer wrong
> Correctness is not the test. The filter must restrict the searchable rows before ranking, and this row belongs in the eval set as a permission trap.

## Go deeper
- [Ragas: evaluation metrics for retrieval-augmented generation](https://docs.ragas.io/en/stable/concepts/metrics/)
- [OpenAI: Evals design guide](https://platform.openai.com/docs/guides/evals)
