---
id: m16-09-running-experiments
module: m16-users-analytics-growth
slug: running-experiments
title: "Running a Real Experiment Program"
order: 9
duration_min: 35
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

A team runs their first A/B test. On day two the variant is ahead and the tool says the result is significant, so they ship it and announce a 22% lift in the group chat.

On day nine the number is back where it started. There was no lift. They had checked the dashboard eleven times in forty-eight hours and stopped at the moment it looked best, which is the single most reliable way to manufacture a false result.

Six weeks later in M18 rehearsal someone asks what the primary metric was and when it was chosen. The honest answer is that it was chosen on day two, from among four numbers they were watching, because it was the one that moved. Nobody wrote anything down beforehand, and there is no record of the two earlier changes they shipped on the same reasoning.

### What M07 covered, and what is new here

M07's chapter nine taught experiment design and it taught it properly: what a p-value claims and does not claim, minimum detectable effect, sample size, sticky assignment, guardrail metrics, peeking, the ethical floor, and the honest conclusion that most student teams should not run A/B tests at all. Go back to it for any of those terms; this chapter does not redefine them.

What is new is the **programme**: the artefacts and the cadence that make a series of experiments trustworthy rather than a sequence of anecdotes. A dated hypothesis document. A pre-declared end date. A decision rule written before the data. A backlog that is triaged rather than argued about. A log that contains the failures. And the version of all of that which works when you have four hundred users, which is most of you.

### The idea

**An experiment programme is a set of written artefacts, not a set of tests. The hypothesis, the primary metric, the sample size, the end date and the decision rule all exist on paper, dated, before anybody looks at a number.**

Everything that goes wrong in this chapter goes wrong because a decision was made after seeing the data that should have been made before. Pre-writing is not bureaucracy; it is the only mechanism that makes the result mean anything.

### The analogy

The canteen manager keeps a logbook. Each entry: the date, the one thing she changed, what she predicted would happen, how long she would run it for, and what actually happened — including the eleven changes that did nothing and the two that made things worse. She changes one dish at a time, because if she changes three on Thursday then Thursday tells her nothing.

**Where the analogy breaks, in the way that matters most to you.** She serves a hundred and eighty people a day, which is her entire population, and she can walk over and ask the student who left the samosa why. Her sample is complete and her feedback is conversational. Your four hundred users are a sample too small to detect anything subtle, and you cannot ask a single one of them. That combination is why the honest answer for most student products is not a smaller A/B test but a different method — and why the logbook discipline matters more than the statistics.

### Picture it

The experiment document: one page, written before the test, with a date on it. This is the chapter's artefact.

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

### Why peeking manufactures winners

The usual threshold accepts a one-in-twenty chance of being fooled by randomness — for **one** look at the data, at the end.

Look twenty times and you have twenty chances to be fooled, and on two genuinely identical variants you will eventually see the line crossed. Stop there and you have a winner that does not exist. The threshold was never a promise about repeated looking.

This is why the end date is pre-declared, and why "we will check daily and stop when it is clear" is not caution but the exact mechanism that produces false results. If you must monitor daily, monitor the guardrails for harm — not the primary metric for victory.

### Most of you cannot A/B test, and that is fine

Do the sample-size arithmetic first, every time, before designing anything. M07 gave the shape of it: detecting a two-point move at a ten per cent baseline needs thousands of people per group, and halving the effect you want to detect roughly quadruples what you need.

With four hundred users and twelve hundred sessions a month, a test needing two thousand per group would take years. Discovering that in advance takes ten minutes and saves three weeks of running something that could never have concluded.

**The alternative programme is not "trust your gut".** It has the same artefacts and the same discipline. Locate the step with chapter three's funnel. Watch six to ten replays and count the pattern, as in chapter five. Sit with five real users as M03 taught. Write the prediction as a number and a date — "activation within 24 hours rises from 29% to at least 40% for cohorts starting after 18 November". Ship exactly one change. Re-measure the identical funnel on the next cohort only.

That method has one known weakness: you cannot rule out that something else changed in the same period. So you name the confounds in the document — the exam week, the festival, the other release — and report the result with that caveat attached. A pre-registered before-and-after with named confounds is far more honest than an underpowered A/B test, and in M18 you present it as a deliberate choice with the arithmetic behind it.

### The log and the backlog

The **log** is one row per experiment, ever: id, date written, hypothesis in one line, primary metric, outcome, decision taken. Inconclusive and failed entries matter most — they stop you re-running the same idea in March, and they are the evidence that your programme is a process rather than a highlights reel.

The **backlog** is how you choose what to test next, and it stops the loudest person deciding. For each idea, note how many users it would affect, how large an effect you would believe, and how long it would take to build. Rank by reach times believable effect, divided by cost, then take the top one.

Cadence: one change at a time, one look at the end, one row in the log, every week or two.

### Where it lives

Chapter three supplies the metric and the leak; chapter five supplies the mechanism; chapter six is where most of your early experiments will aim, and chapter seven's loop steps are next. M13's feature flags are how a variant is served at all, and M06's CI is what stops one shipping to everyone by accident. M09 matters when a variant changes model usage: a variant that triples cost per user can win on conversion and still be wrong. M18 is where the log is read aloud.

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

**Weight-match your variant or you are testing load time.** If variant B ships 200 KB more JavaScript than control, then on a slow prepaid connection you have tested page weight and called it design. This is the most common invalid experiment in an Indian product and almost nobody checks for it.

**Shared devices break sticky assignment in a way you cannot fix.** Assignment hashed on user id is correct, and a phone shared by three students with one account puts three people in one variant. Note it as a limitation rather than pretending the groups are clean.

**One primary metric, two or three guardrails, and nothing else.** Watch twenty numbers at a one-in-twenty threshold and one will look significant by chance. The rest are questions for next time.

**Inconclusive is the most common honest outcome and the hardest to accept.** The decision rule exists so that "we could not tell" is a result you wrote down in advance rather than a disappointment you argue your way out of.

**Novelty and the calendar will both fool you.** A new thing gets attention for being new. A campus festival, exam week or a long weekend changes behaviour more than most variants do. Run long enough to pass the novelty, and write the calendar confounds into the document before you start.

**The ethical floor from M07 still applies, and students are the subjects.** No price differences between students, nothing hidden that involves their money, no variant that removes a safety or cost warning. If you would not describe the test to the person inside it, do not run it.

### Verify it

AI will build you an experiment, declare significance, and be wrong in six checkable ways.

1. **Ask for the hypothesis document and look at its date.** **A primary metric chosen after the data arrived** is the central failure of this chapter, and a dated page is the only defence.
2. **Ask for the sample size calculation and the planned end date before anything runs.** No calculation means nobody knows whether the test could have detected anything.
3. **Check whether the test was stopped early.** **An experiment stopped on the day it first looked significant** is the named red flag here, and the reported lift is usually noise.
4. **Test sticky assignment yourself.** Reload ten times and confirm the same variant every time. Assignment recomputed per render invalidates everything silently.
5. **Compare the payload of the two variants.** If one is materially heavier, the experiment is confounded with load time on exactly the users you care about.
6. **Ask what outcome would have caused you to keep the control**, and check it was written down. If the decision rule only has a ship branch, the test was a formality.

Red flags: significance reached after daily checking; a lift based on a difference of single-digit people; four metrics reported and one highlighted; no end date; a log containing only successes; a variant that also changed three other things; and any experiment run on price or money without disclosure.

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
