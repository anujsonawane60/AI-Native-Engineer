---
id: m14-10-regression-evals
module: m14-deeper-ml-agents
slug: evals-that-catch-regressions
title: "Writing Evals That Actually Catch Regressions"
order: 10
duration_min: 35
is_free: false
mission: true
volatility: low
last_reviewed: 2026-11-16
---

## Key learnings
- Wire an eval suite into CI as a required check that blocks a merge when quality drops
- Build the suite in three tiers and put most of your cases in the cheap, deterministic ones
- Gate on a per-case ratchet, not an average, so a fix that breaks five things cannot pass
- Separate real regressions from sampling noise by running non-deterministic cases several times
- Catch an expected answer that was edited to make a failing case pass

## Lesson

### The pain

M10 chapter 12 built your eval set: 30 to 50 real questions written before you saw any output, scored for retrieval and answer separately, with a judge calibrated against twenty of your own labels, on a scoreboard. It said M14 takes it into production as regression testing. M13 chapter 6 did the same for code — every bug becomes a test that failed first, and a flaky test leaves the gate the same day. This chapter joins the two, and what is new is the gate.

Here is why the gate is the whole point. A team has a good eval set. On a Tuesday they change the system prompt to fix a complaint about tone. Nobody runs the evals, because running them is a notebook somebody has to remember. The aggregate score, had anyone measured it, went from 0.86 to 0.84 — which they would have accepted. Hidden inside that two-point drop: refusals fell from 9 of 10 to 3 of 10, because the friendlier tone made the model reluctant to say no. Three weeks later it tells a student their fee was waived.

The eval set existed and was correct. It was not *wired to anything*, and the one number it produced was an average that hid the only category that mattered.

### The idea

**An eval set tells you whether your feature is good. A regression suite tells you whether your change made it worse, and only the second can be a gate.**

Three differences follow. A regression suite runs on every push as a required check, so it must be fast and cheap. It gates on **per-case movement**, not an average, because an average is where a broken category hides. And it grows by one case every time production fails — M13 chapter 6's rule transplanted: every bug becomes a case that failed first and then stays forever.

### The analogy

M13 chapter 6 gave you the canteen's complaint book: every complaint ever received, walked each morning before service.

The gate is what the book lacks. Add a rule: the shutters do not open until the book has been walked and every line signed. Not "the manager usually checks" — the shutters are locked. That one change turns a document into a control.

Then add a second rule, because the first is not enough. Yesterday 48 of 50 lines passed; today 47 pass. The average is fine. But the line that flipped is "the dal is never served cold to the hostel batch", and it is not interchangeable with the other 49. So the rule is not "score at least 46" — it is "no line that passed yesterday may fail today", with the cold-dal line given its own floor.

**Where the analogy breaks.** A cook tasting the same sambar twice gets the same answer. Your model does not. Ask it the same question three times and you may get three answers, two of which pass. A complaint book has no concept of a line that passes two times in three, and that difference is why the next section exists.

### Picture it

```
  TIER 0  ASSERTIONS            free, instant, never flaky
  ┌──────────────────────────────────────────────────┐
  │ valid JSON · required fields · no PII in output  │  ~60% of
  │ citation id resolves · length < limit · p95 < 3s │  your cases
  └──────────────────────────────────────────────────┘
  TIER 1  DETERMINISTIC SCORERS  cheap, repeatable
  ┌──────────────────────────────────────────────────┐
  │ exact match · number within ±1 · set overlap     │  ~30%
  │ tool-call sequence · refused? yes/no             │
  └──────────────────────────────────────────────────┘
  TIER 2  LLM JUDGE              slow, costs money, needs
  ┌──────────────────────────────────────────────────┐  calibration
  │ helpfulness · tone · faithfulness to sources     │  ~10%
  └──────────────────────────────────────────────────┘
                          │
   on every push: tiers 0+1   ──▶ REQUIRED CHECK ──▶ merge blocked
   nightly: all three tiers   ──▶ scorecard + trend
```

Most of a regression suite should be cheap assertions and deterministic scorers running on every push as a blocking check, with expensive judged cases reserved for a nightly run.

### Non-determinism, and how to gate anyway

This is the part code testing never prepared you for. Temperature zero is not determinism: providers batch requests differently, hardware differs, and the model behind a name is updated. A case can pass, pass, fail, pass, with nothing changed. Three mechanics make a gate possible anyway.

**Run non-deterministic cases several times and score a pass rate.** Three runs is the practical minimum. A case that passes 3 of 3 is green; 0 of 3 is red; 1 or 2 of 3 is **unstable**, which is information, not an error.

**Define a noise band once, empirically.** Run the whole suite three times unchanged and record the spread. If the aggregate moves by a point and a half between identical runs, a one-point drop is not a regression. Skip this and you either gate on noise and stop trusting the check, or set a threshold so loose it never fires.

**Quarantine unstable cases exactly as M13 chapter 6 quarantines flaky tests.** Out of the gate the same day, an owner, a one-week deadline, then sharpened or deleted. A case permanently at 2 of 3 teaches your team that red is negotiable.

### The gate rule that actually works

Not "aggregate must exceed 0.85". Three rules together.

**The per-case ratchet.** No case that passed on the main branch may fail on this branch. This catches "fixed three, broke five", and it is the most valuable line in the chapter. A deliberate exception is allowed — recorded in the pull request, never silent.

**Category floors.** Gate per category, not only overall: refusals, money arithmetic, permissions, each language, long inputs. The refusals disaster above is invisible to an aggregate and obvious to a category floor.

**A budget ceiling.** The gated run must finish inside a time and a rupee figure you wrote down — three minutes and ₹5, say. A suite costing ₹300 and twenty minutes per push will be made optional within a fortnight, and then it is a notebook again.

### Evaluating agents, not just answers

For anything agentic from M10 chapters 13 to 15, the final answer is only half of what can regress. The other half is the **trajectory**: which tools were called, in what order, how many steps were used, whether a forbidden action was attempted. All four are deterministic to check and belong in tier 1.

Four trajectory assertions earn their place immediately. The expected tool was called. No tool outside the allowed list was called. The step count stayed inside its budget. No write or send happened in a read-only case. An agent that reaches the right answer in nineteen steps instead of four has regressed in cost, and only a trajectory check sees it.

### Where it lives

M06 chapter 12 is the home: a required check in your pipeline, beside your unit tests, with the same authority. M13 chapter 6 supplies the discipline — fail-first cases, quarantine, no production data in fixtures. M07 chapters 5 and 6 supply the traces and percentiles. M10 chapter 12 supplies the cases and the calibrated judge. Chapters 4, 6, 7 and 11 of this module are what this suite referees: a fine-tune, a quantization level, a serving change and a multi-agent design all need the same question answered.

Beyond the push, the same cases run twice more. **Nightly**, the full suite including judged cases, producing a trend. And **online**, where a sample of real production traffic is scored against tier 0 assertions — which is how you find the failures your 50 cases never imagined.

### Words to own

| Term | Plain meaning | Example |
|---|---|---|
| Regression suite | Cases proving nothing got worse | Runs on every push |
| Assertion | A pass/fail check needing no model | Valid JSON |
| Deterministic scorer | Repeatable scoring, no model involved | Number within ±1 |
| Judge pinning | Fixing the judge's model version | Else your ruler moves |
| Pass rate at n | Passes out of n repeated runs | 2 of 3 is unstable |
| Noise band | Spread across identical runs | ±1.5 points |
| Per-case ratchet | No passing case may start failing | The core gate rule |
| Category floor | A minimum score per category | Refusals ≥ 9 of 10 |
| Trajectory eval | Scoring the tool calls, not the answer | Step count, tool list |
| Case provenance | Where a case came from | Incident 2026-10-14 |
| Quarantine | Unstable case out of the gate, with an owner | One-week deadline |
| Eval budget | Time and rupees a gated run may cost | 3 min, ₹5 |
| Scorecard diff | Per-category change versus main | Posted on the PR |

### Tradeoffs and gotchas

**Every production failure becomes a case, and it must fail before the fix.** Reproduce the failure as a case, watch it go red, fix, watch it go green. A case added after the fix proves nothing, exactly as in M13 chapter 6.

**Never edit an expected answer in the same pull request as a fix.** This is the most dangerous diff in the chapter. Either the expected behaviour genuinely changed — which deserves its own commit and its own reasoning — or somebody bent the case to fit a bug. Only a person can tell those apart, and an agent told to "make the evals pass" will reliably choose the second.

**A 100% pass rate means the suite is too easy.** Real suites sit below full marks, with known-hard cases tracked. All-green is a signal to add the cases you have avoided.

**Pin the judge, and treat its prompt as code.** If the judge model updates silently, your measuring instrument changed and every historical number is incomparable. Pin the version, commit the rubric, and re-run M10 chapter 12's twenty-label calibration whenever either changes. And never judge with the model you are testing — it prefers its own style, inflating the thing you are measuring.

**Cost and p95 are scorecard columns, not separate concerns.** A change that gains one quality point and triples cost per request is a regression. Without money and latency columns, half your regressions are invisible.

**Deleting a failing case is the one unforgivable move.** It is the eval equivalent of re-running a red build until it goes green, and it converts your gate into decoration.

### Verify it

Everything here depends on cases written honestly, so the red flags are about provenance and timing.

1. **Ask when each case was written, relative to the output it checks.** An eval set written after seeing the model's answers measures agreement with what the model already does. Demand provenance per case: a real request, a real incident, a date.
2. **Count the tiers.** If most cases need a judge, the suite is too slow to gate with, and most of them could have been assertions.
3. **Ask for the noise band, measured.** Three identical runs, spread recorded. A threshold tighter than the noise fires randomly.
4. **Check the gate rule is per-case, not an average,** and that a pass-to-fail flip blocks, with any exception explicit in the pull request.
5. **Look for category floors**, specifically refusals, money and permissions.
6. **Check the judge's version is pinned** and is not the model under test.
7. **Read the diff for changed expected answers.** Any such change inside a fix commit needs an explanation before anything else is reviewed.
8. **Confirm it actually blocks.** Break something on purpose, push, watch the merge be refused. A check never seen to fail has not been tested.

Red flags: an eval set written after seeing output; cases with no provenance; a single aggregate as the gate; no per-category breakdown; no noise band; an unrecorded judge version; the model judging itself; expected answers edited in a fix PR; a failing case deleted; 100% pass; no cost or latency column; agents with no trajectory checks; and a suite never observed to block a merge.

## Walkthrough
1. Start from your M10 chapter 12 eval set. For every case, add a provenance column: where it came from and the date. Any case you cannot source, mark as invented — you will replace it.
2. Pull your last ten real production failures from your logs or support messages. Turn each into a case with its expected output. Confirm each one fails against your current system before you fix anything.
3. Sort every case into tiers. Rewrite as many as you can as tier 0 assertions: valid JSON, a required field present, a citation id that resolves, no phone number in the output, a length limit, a latency limit. Aim for most of your suite here.
4. Convert what remains into tier 1 deterministic scorers where possible — exact match, a number within a tolerance, set overlap, refused yes or no, the expected tool called. Leave only genuinely open-ended quality in tier 2.
5. Tag every case with a category: refusals, money, permissions, each language, long inputs, and anything else your product has failed at.
6. Measure your noise band: run the whole suite three times with no changes. Record the aggregate each time and the per-case pass rates. Quarantine every case that is not 3 of 3 or 0 of 3.
7. Build the scorecard: a per-category table plus total cost of the run and p95 latency, written to a file as well as printed.
8. Wire the gated run — tiers 0 and 1 only — into your pipeline from M06 chapter 12 as a required check. Enforce three rules: no case may move from pass to fail, every category must meet its floor, and the run must finish inside your time and rupee budget.
9. Prove it works. Deliberately regress something small — loosen a refusal instruction, drop a field from the schema — push it, and confirm the merge is blocked and the scorecard names the category that fell. Screenshot that.
10. Add the nightly job: all three tiers including the judge, with the judge's model version pinned and its rubric committed. Record the trend.
11. Add the online sample — score 20 real production responses a night against your tier 0 assertions, routing any failure into the suite as a new case — and commit your team's three rules: every production failure becomes a case that failed first, no expected answer is edited in the same commit as a fix, and an unstable case leaves the gate the same day.

## Talk to AI
```prompt
CONTEXT:      I am a student on an AI-native engineering program. I direct AI
              and verify its output; I do not write code by hand. I already
              have an eval set of [number] real cases with expected answers,
              scored for retrieval and answer separately, and a judge I
              calibrated against 20 of my own labels. I know what recall at k,
              a rubric and a scoreboard are, so do not re-teach them. My team
              of three runs [product in two sentences] with [number] real
              users in India. Our AI feature is [describe it, and say whether
              it is a single call, a workflow or an agent with tools]. Our
              pipeline already runs unit tests on every push. Today we run the
              evals by hand, in a notebook, when someone remembers.

GOAL:         A regression suite wired into CI as a required check that blocks
              a merge when quality drops, with a scorecard I can read on the
              pull request, running inside a budget of 3 minutes and ₹5.

REQUIREMENTS: 1. Sort my cases into three tiers and tell me, case by case,
                 which ones I can rewrite as pure assertions needing no model
                 at all. Give me the assertion for each
              2. Give me the deterministic scorers for the middle tier:
                 exact match, numeric tolerance, set overlap, refusal
                 detection, and tool-call checks. Tell me which of my cases
                 each one fits
              3. My feature is agentic, so give me four trajectory assertions:
                 expected tool called, no forbidden tool called, step budget
                 respected, and no write in a read-only case
              4. Give me a procedure to measure my noise band by running the
                 suite three times unchanged, and tell me how to set a gate
                 threshold that is wider than the noise
              5. Define the gate as three rules: a per-case ratchet, per
                 category floors, and a budget ceiling. Tell me which
                 categories deserve their own floor for MY product
              6. Tell me how to handle a case that passes 2 out of 3 runs,
                 including the quarantine policy, the owner and the deadline
              7. Give me the scorecard format to post on a pull request:
                 per-category pass rates versus main, plus total run cost in
                 rupees and p95 latency
              8. Tell me exactly how to wire this as a REQUIRED check in my
                 pipeline, and how to prove it blocks by regressing something
                 on purpose
              9. Give me the nightly job: full suite with the judge, with the
                 judge version pinned and the rubric committed. Tell me what
                 to do when the judge model is updated
             10. Give me an online eval: score 20 sampled production responses
                 a night against tier 0 assertions, and route failures into
                 the suite as new cases

CONSTRAINTS:  Free tier only. The gated run must finish in under 3 minutes and
              cost under ₹5, so tell me if any of my cases are too expensive
              to gate with. Plain English for a second-language reader. Label
              every command READ-ONLY or CHANGES-THINGS. No real user phone
              numbers, emails or payment references in any case. Do not use
              the model under test as the judge.

NON-GOALS:    Do not re-teach eval set construction or judge calibration. Do
              not gate on a single aggregate number. Do not propose a paid
              eval platform. Do not add cases by generating them - every case
              must come from a real request or a real incident. Do not change
              my application code.

ACCEPTANCE:   - Most of my cases run as assertions with no model call
              - Four trajectory assertions exist for my agent
              - A measured noise band exists from three identical runs
              - The gate blocks on any case moving from pass to fail
              - Refusals, money and permissions each have their own floor
              - A gated run finishes inside 3 minutes and ₹5
              - I have watched the check block a merge after a deliberate
                regression, and have the screenshot
              - The judge's model version is pinned and its rubric committed
              - Every case has a provenance field with a source and a date
              - A nightly full run and a nightly online sample both exist

ASK:          First, go through my cases and tell me which ones are currently
              judged that could be pure assertions instead, and what that does
              to my run time and cost. Then tell me which single category of my
              product would be most likely to regress invisibly under an
              average. Do not write any configuration until you have.
```

## Mission

**Objective.** A regression suite for your own product, wired into CI as a required check, which you have personally watched block a merge after a deliberate regression.

**Steps.**
1. **Frame** — write the one quality failure that would most damage your product if it shipped silently. That category gets its own floor.
2. **Source the cases** — add provenance to every existing case, and add one case for each of your last ten real production failures. Confirm each new case fails before you fix anything.
3. **Tier them** — rewrite as many as possible as assertions, then deterministic scorers, leaving only open-ended quality to the judge. Record how many ended in each tier.
4. **Tag categories** — refusals, money, permissions, every language your users write in, long inputs.
5. **Measure noise** — three identical runs. Record the aggregate spread and quarantine every unstable case.
6. **Brief** — write a Director's Brief for the suite, the gate rules and the scorecard, including the 3-minute and ₹5 budget.
7. **Direct** — build it, wire the gated tiers as a required check in your pipeline from M06 chapter 12, and add the nightly full run with a pinned judge.
8. **Verify** — regress something on purpose, push, and confirm the merge is blocked and the scorecard names the fallen category. Keep the screenshot. Then revert and confirm it goes green.
9. **Document** — commit the three team rules, and record a decision-log entry with your noise band, your gate thresholds and your per-run cost.

**Upload.**
- Your Director's Brief → post caption or an attached file
- Your case file with the provenance and tier columns visible → screenshot
- Your tier counts: how many assertions, scorers and judged cases
- The three identical runs with the measured noise band → screenshot
- Your quarantine list with owners and dates
- A screenshot of the required check **blocking a merge** after a deliberate regression
- A screenshot of the scorecard diff showing which category fell
- The gated run's duration and cost in rupees
- The AI conversation → share link in `ai_chat_url`
- Reflection, 150 words max → one tradeoff you chose, and one thing the AI got wrong

**Self-check.** Tick every box before you submit.
- [ ] Every case has a source and a date, and none were invented
- [ ] At least ten cases came from real production failures
- [ ] Each of those ten was seen failing before any fix
- [ ] Most of my cases run with no model call at all
- [ ] My agent's trajectory is checked, not only its final answer
- [ ] I measured my noise band from three identical runs
- [ ] My gate blocks on any case moving from pass to fail, not on an average
- [ ] Refusals, money and permissions each have their own floor
- [ ] The gated run finishes inside my stated time and rupee budget
- [ ] My judge's model version is pinned and its rubric is committed
- [ ] The judge is not the model being tested
- [ ] I personally watched the check block a merge, and I have the screenshot
- [ ] My scorecard includes cost per run and p95 latency
- [ ] No expected answer was edited in the same commit as a fix

**Stretch.** Take a change you have already shipped — a prompt edit, a model swap, a quantization level — and run your new suite against the version before and after. Report whether anything regressed that nobody noticed at the time.

## Stuck? Ask AI
```prompt
Here are 15 of my eval cases, each currently scored by an LLM judge: [paste them]. Do not improve the rubric. Instead, for each case tell me whether it could be rewritten as a pure assertion or a deterministic scorer needing no model call, and if so write that check for me exactly. Then tell me what my total run time and cost would become, and which cases genuinely still need a judge and why.
```

```prompt
My suite's aggregate score went from 0.86 to 0.84 after a change and I was about to accept it. Here is the per-case result before and after: [paste it]. Tell me which individual cases flipped from pass to fail, group them by what they have in common, and tell me whether the aggregate was hiding a category collapse. Then tell me what gate rule would have blocked this change, and write it precisely.
```

```prompt
Explain why temperature zero does not make my evals deterministic, and what else varies between identical runs. Then give me a procedure to measure my suite's noise band, tell me how to set a gate threshold wider than that noise, and explain what to do with a case that passes 2 out of 3 runs. Include the quarantine policy as a file I can commit, with an owner and a deadline.
```

## Quiz

### Q1 · L3
What is the main difference between an eval set and a regression suite?
- (a) A regression suite uses an LLM judge and an eval set does not
- (b) A regression suite runs automatically as a gate and blocks a change that makes anything worse ✅
- (c) An eval set is larger
- (d) A regression suite does not need expected answers
> An eval set answers "is this good?" A regression suite answers "did my change break something?" — which only works if it runs on every push and can refuse a merge.

### Q2 · L4
Aggregate score drops from 0.86 to 0.84 while refusals fall from 9 of 10 to 3 of 10. Which gate rule catches this?
- (a) An aggregate floor of 0.85
- (b) A per-case ratchet plus a category floor for refusals ✅
- (c) A longer nightly run
- (d) A stricter judge rubric
> An average of fifty cases absorbs a collapse in one category of ten. Per-case movement and a named floor for refusals both fire immediately; the aggregate does not.

### Q3 · L4
A case passes on two of three identical runs. What should happen?
- (a) Treat it as a pass, since the majority passed
- (b) Treat it as a failure and block the merge
- (c) Quarantine it with an owner and a one-week deadline ✅
- (d) Delete it, since it is unreliable
> An unstable case cannot gate anything, and leaving it in teaches the team that red is negotiable. Out of the gate the same day, then made sharper or removed — exactly as M13 chapter 6 treats a flaky test.

### Q4 · L4
A pull request fixes a bug and also changes two expected answers in the eval file. What is the first thing to check?
- (a) Whether the suite still runs inside the budget
- (b) Whether the judge version changed
- (c) Whether the expected behaviour genuinely changed, or a case was bent to fit the bug ✅
- (d) Whether the new code has unit tests
> Only a human reading the diff can tell those two apart, and an agent told to "make the evals pass" will choose bending. Expected answers change in their own commit, with reasoning.

### Q5 · L5
Your agentic feature returns the right answer but now takes nineteen tool calls instead of four. Your suite is all green. What is missing?
- (a) A larger case set
- (b) Trajectory assertions on tool calls and the step budget ✅
- (c) A stricter judge rubric
- (d) A higher category floor on correctness
> Answer-only scoring cannot see a cost regression. The step count, the tools called and the absence of forbidden actions are deterministic, cheap, and belong in the gated tier.

## Go deeper
- [Anthropic: creating empirical evaluations and strong success criteria](https://docs.anthropic.com/en/docs/test-and-evaluate/develop-tests)
- [GitHub Actions: required status checks and branch protection](https://docs.github.com/en/repositories/configuring-branches-and-merges-in-your-repository/managing-protected-branches/about-protected-branches)
