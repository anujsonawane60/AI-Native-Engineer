---
id: m16-09-running-experiments
module: m16-users-analytics-growth
slug: running-experiments
title: "Running a Real Experiment Program"
order: 9
duration_min: 25
is_free: false
mission: true
volatility: low
last_reviewed: 2026-11-16
---

## Key learnings
- Write and date a hypothesis, a primary metric and a decision rule before any test
- Pre-declare the sample size and the end date, then look once
- Compute whether your traffic supports a test at all, and accept the answer
- Run the qualitative programme with the same rigour when it does not
- Keep a log of every experiment, including the ones that failed

## Lesson

### The pain

A team runs their first A/B test. On day two the variant is ahead and the tool calls it significant, so they ship it and announce a 22% lift. On day nine it is back where it started: they had checked the dashboard eleven times in forty-eight hours and stopped when it looked best.

Six weeks later someone asks what the primary metric was and when it was chosen. The answer: on day two, from four numbers, because it was the one that moved.

### What M07 covered, and what is new here

M07 taught experiment design properly — p-values, minimum detectable effect, sample size, sticky assignment, guardrails, peeking, the ethical floor. New here is the **programme**: the dated artefacts that make a series of experiments trustworthy.

### The idea

**An experiment programme is a set of written artefacts, not a set of tests. The hypothesis, the primary metric, the sample size, the end date and the decision rule all exist on paper, dated, before anybody looks at a number.**

Everything that goes wrong here goes wrong because a decision was made after seeing the data. The significance threshold accepts a one-in-twenty chance of being fooled — for **one** look, at the end. Look twenty times and you have twenty chances, and on two identical variants you will eventually see the line crossed. "We will check daily and stop when it is clear" is the mechanism that manufactures winners. Monitor guardrails daily if you must, never the primary metric.

Two more artefacts: a **log** with one row per experiment ever, failures included, and a **backlog** ranked by reach times believable effect divided by cost.

### The analogy

The canteen manager keeps a logbook: the date, the one thing she changed, what she predicted, and what happened — including the eleven changes that did nothing. She changes one dish at a time, because three changes on Thursday tell her nothing.

**Where the analogy breaks:** she serves a hundred and eighty people a day, her whole population, and can ask the student who left the samosa why. Your four hundred users are too small a sample for anything subtle, and cannot be asked.

### Picture it

```
  EXPERIMENT 07                      written 2026-11-16

  HYPOTHESIS   Moving phone verification to after the
               first order will raise 24-hour activation,
               because 5 of 6 replays showed users
               abandoning at an OTP that never arrived.

  PRIMARY      activation rate within 24h of first
  METRIC       arrival  (today: 29%)

  GUARDRAILS   order error rate (now 1.1%), fake-order
               share (now 0%), support messages per 100
               users (now 4)

  DETECT       +8 points or more. Smaller moves will not
               change our decision.

  SAMPLE       1,050 per group at this baseline
  DURATION     start 18 Nov, end 09 Dec. One look, on
               09 Dec. No interim decisions.

  DECISION     ship if primary +8 or more and no
  RULE         guardrail worsens; kill if primary falls;
               otherwise INCONCLUSIVE and we keep the
               control.
```

Five fields, each blocking a specific failure: the hypothesis blocks metric-shopping, the primary metric blocks measuring twenty things, the detect threshold blocks celebrating noise, the duration blocks peeking, and the decision rule blocks arguing about what the result meant after you have seen it.

### Where it lives

Chapter three supplies the metric and the leak, chapter five the mechanism, chapter six is where most early experiments aim. M13's feature flags serve the variant; M18 reads the log aloud.

### Words to own

| Term | Plain meaning | Example |
|---|---|---|
| Hypothesis | A dated prediction with a reason | "Because 5 of 6 replays showed…" |
| Pre-registration | Writing the plan before the data | The document above |
| Primary metric | The one number decided in advance | 24-hour activation rate |
| Guardrail | A number that must not get worse | Order error rate |
| Detect threshold | The smallest move you would act on | +8 points |
| Peeking | Checking repeatedly and stopping on a good day | Eleven looks in two days |
| Decision rule | Ship, kill or inconclusive, written first | Written 16 Nov |
| Inconclusive | A valid outcome that is not a failure | Keep the control |
| Holdout | A group deliberately left unchanged | Last week's cohort |
| Experiment log | Every test, including the failures | One row each |

### Tradeoffs and gotchas

**Most of you cannot A/B test, and that is fine.** Do the sample-size arithmetic before designing anything: detecting a two-point move at a ten per cent baseline needs thousands per group, and halving the effect roughly quadruples the need. With four hundred users, a test needing two thousand per group takes years — finding that out takes ten minutes.

**The alternative is not "trust your gut"; it has the same artefacts.** Locate the step with chapter three's funnel, count the pattern across six to ten replays, sit with five users. Write the prediction as a number and a date — "activation within 24 hours rises from 29% to at least 40% for cohorts starting after 18 November" — ship one change, then re-measure the identical funnel on the next cohort only. Name the confounds beforehand. A pre-registered before-and-after with named confounds beats an underpowered A/B test.

**Weight-match your variant or you are testing load time.** 200 KB more JavaScript on a slow prepaid connection tests page weight, not design. Novelty and the calendar will fool you in the same way: run long enough to pass both.

**Shared devices break sticky assignment.** One shared account puts three people in one variant; note it as a limitation.

**Inconclusive is the commonest honest outcome, and the hardest to accept.**

**The ethical floor still applies, and students are the subjects.** If you would not describe the test to the person inside it, do not run it.

### Verify it

AI will build an experiment, declare significance, and be wrong in checkable ways.

1. **Ask for the hypothesis document and look at its date.** A primary metric chosen after the data arrived is this chapter's central failure.
2. **Ask for the sample size calculation and the end date before anything runs.** Without it nobody knows whether the test could detect anything.
3. **Test sticky assignment yourself.** Reload ten times and confirm the same variant every time.
4. **Ask what outcome would have kept the control.** A rule with only a ship branch means the test was a formality.

Red flags: significance after daily checking; a lift based on single-digit people; a log of successes only.

## Walkthrough
1. Write your experiment backlog: at least six candidate changes, each with the users it would reach, the effect you would honestly believe, and the build cost in days.
2. Rank the backlog by reach times believable effect, divided by cost. Write the ranking down before discussing it, then take the top item.
3. For that item, write the hypothesis in one sentence with its reason, citing the chapter three or chapter five evidence that produced it. Date it.
4. Name the single primary metric and its current value, plus two or three guardrails with their current values.
5. Write the detect threshold: the smallest move that would change your decision. Not the smallest you can imagine.
6. Have AI compute the sample size per group for your baseline and that threshold, with its inputs shown, and compare it against your realistic traffic for the next three weeks.
7. If the arithmetic says you cannot run it, write one sentence saying so with the numbers, and switch to the qualitative programme: which five users you will watch, which replays, and which single funnel number you will check afterwards.
8. Either way, write the duration and the one date on which you will look, plus the decision rule covering ship, kill and inconclusive.
9. If you are running an A/B test, verify sticky assignment by reloading ten times, and compare the two variants' page weight before launching.
10. Launch, then look only at the guardrails until the end date. Record each guardrail check in the log without touching the primary metric.
11. On the stated date, look once, apply the written decision rule without amending it, and record the outcome — including inconclusive — in the experiment log.

## Talk to AI
```prompt
CONTEXT:      I am a student in a team of three in India. I direct AI and
              verify its output; I do not write code. Our product is live:
              [describe it in two sentences]. We have [number] active users
              and roughly [number] sessions a week, and we expect [number]
              new users over the next three weeks. Our current rate on the
              metric I care about is [number]%. The change I want to test
              is: [describe it]. The evidence that made me want to test it
              is: [paste the funnel number, replay count or user quote].
              Our users are students on budget Android phones, often on
              slow prepaid connections, sometimes sharing a phone with one
              account. Upcoming calendar events in the next month:
              [exam weeks, festivals, holidays, planned releases].

GOAL:         A pre-registered experiment document I can date and commit
              today, or an honest statement that my traffic cannot support
              a test plus the qualitative programme instead.

REQUIREMENTS: 1. Compute the sample size per group for my baseline at a
                 2-point, a 5-point and a 10-point detect threshold. Show
                 your inputs. Then tell me plainly, using my traffic, how
                 many weeks each would take.
              2. State clearly whether I can run an A/B test at all. If I
                 cannot, say so before designing anything else.
              3. Help me write the hypothesis in one sentence with its
                 reason and today's date.
              4. Name my single primary metric, and two or three guardrails
                 that must not get worse. Tell me which guardrail I am most
                 likely to forget for a product like mine.
              5. Write the decision rule with all three branches - ship,
                 kill, inconclusive - and tell me which branch teams most
                 often leave out.
              6. Explain, in words and with a worked example, what happens
                 to my chance of a false result if I check the primary
                 metric daily for twenty days on two identical variants.
              7. If I cannot run a test, design the qualitative programme
                 with the same rigour: how many users I watch, what I look
                 for, the single funnel number I check afterwards, the
                 written prediction with a date, and the confounds I must
                 name in advance from my calendar.
              8. Tell me how to check that my two variants are
                 weight-matched, and what it means for my result if they
                 are not on a slow connection.
              9. Give me the columns of an experiment log that will still
                 be useful to me in four months, including for the tests
                 that fail.

CONSTRAINTS:  Free tier only. No formulas or statistics notation - explain
              everything in words and small numbers. Do not tell me a
              result is significant; give me the design and the checks. Do
              not propose testing anything involving different prices for
              different users. Plain English, every term explained on first
              use.

NON-GOALS:    Do not implement the variant. Do not propose more than one
              change in a single test. Do not design my event taxonomy. Do
              not write code and do not modify any files.

ACCEPTANCE:   - The sample-size arithmetic is shown with its inputs
              - You said plainly whether a test is possible for me
              - My document has a dated hypothesis, one primary metric,
                guardrails, a detect threshold, a duration with one look
                date, and a three-branch decision rule
              - I can explain in my own words why peeking manufactures
                winners
              - If a test is impossible, I have an equally rigorous
                qualitative plan with named confounds

ASK:          First ask me up to five questions about my traffic, my
              baseline and my calendar. Then answer requirements 1 and 2
              only, and stop before writing any document.
```

## Mission

**Objective.** A dated, pre-registered experiment document and its result — either one properly powered A/B test or the honest qualitative programme with the sample-size arithmetic that ruled a test out — plus an experiment log containing every attempt.

**Steps.**
1. **Frame** — build and rank a backlog of at least six candidate changes by reach times believable effect divided by cost. Record the ranking before discussing it.
2. **Compute first** — have AI calculate the sample size per group at three detect thresholds, with inputs shown, and compare against your realistic traffic. Do this before designing anything.
3. **Decide the method** — A/B test if the arithmetic supports it; otherwise write one sentence with the numbers saying it does not, and switch to the qualitative programme.
4. **Pre-register** — write the one-page document: dated hypothesis with its evidence, one primary metric with its current value, two or three guardrails, the detect threshold, the duration with a single look date, and a three-branch decision rule. Commit it before launch.
5. **Check the mechanism** — if testing, verify sticky assignment by reloading ten times and confirm the two variants are weight-matched.
6. **Run** — look only at guardrails until the stated date. Record each guardrail check.
7. **Look once** — on the stated date, apply the written decision rule without amending it.
8. **Log it** — one row in the experiment log, including inconclusive outcomes and anything you abandoned.
9. **Verify** — run the six checks from "Verify it" against your own experiment and write down which ones you nearly failed.

**Upload.**
- Your ranked backlog of at least six candidates, with reach, believable effect and cost → table or screenshot
- The sample-size arithmetic with its inputs, and your verdict on whether a test was possible → screenshot or text
- Your pre-registered experiment document, with its date visible and committed before launch
- Evidence of the sticky-assignment reload test, or of the five user sessions if you ran the qualitative programme
- The two variants' page weights, if you ran a test
- Your guardrail checks during the run, dated
- The single final measurement and the decision you took under the written rule
- Your experiment log with every attempt in it, including failures → file or screenshot
- Your AI conversation, including where it declared a result you did not accept → share link in `ai_chat_url`
- Reflection, 150 words max → the moment you most wanted to peek, and one confound you could not rule out

**Self-check.** Tick every box before you submit.
- [ ] My hypothesis was written and dated before any data was collected
- [ ] I computed the sample size before designing the test
- [ ] If I ran a test, my traffic genuinely supported the detect threshold
- [ ] If I did not, I wrote the arithmetic that ruled it out and ran the qualitative programme instead
- [ ] I named exactly one primary metric and two or three guardrails
- [ ] My decision rule has a ship branch, a kill branch and an inconclusive branch
- [ ] I declared the end date in advance and looked at the primary metric once
- [ ] I checked only guardrails during the run
- [ ] I verified sticky assignment by reloading, or documented why it did not apply
- [ ] My two variants were comparable in page weight
- [ ] I named the calendar confounds before starting
- [ ] My experiment log includes attempts that failed or were inconclusive
- [ ] Nothing I tested involved different prices or hidden costs for different users

**Stretch.** Take a change your team already shipped on instinct and reconstruct the experiment document it should have had, honestly, including whether your traffic could ever have detected the effect you claimed. Write 100 words on what you would now say about that change in M18.

## Stuck? Ask AI
```prompt
Explain, with a worked day-by-day example using two genuinely identical variants, what happens to my chance of seeing a false winner if I check the primary metric every day for twenty days and stop when it looks significant. Then tell me the smallest change to my process that removes this problem, and why monitoring guardrails daily is different.
```

```prompt
Here is my experiment plan: [paste it]. Do not improve it. Audit it instead. Which decision in it was made after seeing data? Which metric could I swap for a better-looking one? What is missing that would let me argue my way to a win afterwards? Then tell me the single field whose absence would make this result indefensible.
```

```prompt
I have [number] users and [number] sessions a week, and my current rate on the metric I care about is [number]%. Tell me plainly whether I can run an A/B test, showing the arithmetic. If I cannot, design the qualitative alternative with the same rigour: how many people I watch, the single number I check afterwards, the written prediction, and the confounds I must name in advance.
```

## Quiz

### Q1 · L4
Your test looks significant on day two of a planned three-week run. What does the written decision rule require?
- (a) Ship immediately, since the effect is clear
- (b) Continue to the pre-declared end date and look once ✅
- (c) Extend the test by a week to be certain
- (d) Switch the primary metric to the one moving fastest
> The threshold protects a single look, not a series of them. Stopping on the first favourable day is exactly how two identical variants produce a winner, and the pre-declared end date is the only thing that prevents it.

### Q2 · L5
Your variant ships 200 KB more JavaScript than the control, and it converts worse on budget Android phones. What have you actually measured?
- (a) That the new design is worse for low-end users
- (b) A confound between your design change and page weight on slow connections ✅
- (c) That the sample was too small for mobile users
- (d) That sticky assignment failed on those devices
> With two variables changed at once you cannot attribute the result to either. On slow prepaid connections, page weight is often the stronger effect, which makes weight-matching a precondition rather than a refinement.

### Q3 · L4
You have 400 users and the sample-size calculation says you need 2,000 per group. What is the correct response?
- (a) Run the test anyway and interpret the direction of the result
- (b) Lower the significance threshold so the test can conclude
- (c) Write down the arithmetic and run a pre-registered qualitative programme instead ✅
- (d) Test a bigger change so a smaller sample suffices
> An underpowered test cannot distinguish an effect from noise however you read it. The honest substitute keeps the same discipline — a written prediction, one change, a re-measured funnel — with its confounds named.

### Q4 · L3
Which outcome should the decision rule include that teams most often leave out?
- (a) Ship the variant
- (b) Kill the variant
- (c) Inconclusive, and keep the control ✅
- (d) Extend the test
> Inconclusive is the most common honest result. Writing it in advance turns "we could not tell" into a planned outcome rather than something to be argued away once the data disappoints.

### Q5 · L4
Why must the experiment log contain failed and inconclusive experiments?
- (a) To satisfy the analytics tool's audit requirements
- (b) So the same idea is not re-run later, and so the process is visible rather than a highlights reel ✅
- (c) Because failed tests can be re-analysed to find wins
- (d) To increase the total number of experiments reported
> A log of successes only is a selected sample of your own work. The failures prevent repetition and are the evidence that decisions followed a process, which is what gets questioned in a defence.

## Go deeper
- [Evan Miller: How Not To Run An A/B Test](https://www.evanmiller.org/how-not-to-run-an-ab-test.html)
- [Microsoft: trustworthy online controlled experiments, practical guide](https://www.microsoft.com/en-us/research/group/experimentation-platform-exp/articles/)
