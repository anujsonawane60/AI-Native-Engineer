---
id: m16-04-retention-and-cohorts
module: m16-users-analytics-growth
slug: retention-and-cohorts
title: "Retention Curves, Cohorts and the Flattening Point"
order: 4
duration_min: 15
is_free: false
mission: false
volatility: low
last_reviewed: 2026-11-16
---

## Key learnings
- Set day zero at first value rather than at signup, and know what changes
- Read a cohort triangle down the column and across the row
- Choose between N-day and bracketed retention, and say which you used
- Refuse to read a cohort smaller than about fifty people as a statistic
- Compare measured retention against the baseline behaviour you recorded in M01

## Lesson

### The pain

A team plots their first real retention curve with day zero at signup. By week four 4% of each cohort is still active: the curve does not flatten, it falls to the floor. They spend an evening discussing a pivot.

Next morning somebody re-plots it with day zero at each user's first completed order. It flattens at 31% and stays there. Both numbers are correct and answer different questions: most signups never reach value, an onboarding problem, and a third of those who do build a habit, which means there is a product here. One choice of day zero nearly ended the company.

### What M07 covered, and what is new here

M07 defined cohorts, the curve, the flattening point and the rule that the period must match your natural frequency. New here is day zero, the two definitions, the triangle, minimum cohort size, resurrection, and closing the loop on your M01 baseline.

### The idea

**A retention curve means nothing until you say what day zero is, and day zero should be the moment the user first received value — not the moment they filled in a form.**

Signup-anchored retention mixes onboarding with product and reports the worse of the two. Value-anchored retention measures the product. Keep both, labelled: the gap is the size of your onboarding problem and chapter six's agenda.

Then say how you measured. **N-day retention** asks whether a user was active on day N exactly; it suits genuinely daily products and makes everything else look dead. **Bracketed retention** asks whether they were active at any point during week N, the right default for almost every student product. Neither is more honest; what is dishonest is not saying which, because the same product can be at 8% by one definition and 34% by the other.

### The analogy

Of the students who ate at the canteen in week one of term, how many still ate there in week eight? That is a cohort, answerable from a register with names.

**Where the analogy breaks, twice.** The canteen has a captive audience, so a flat line there may measure convenience rather than preference, while your users can close your app forever at no cost. And the canteen empties during exam break and refills in July, which on a chart looks like churn followed by a miracle: a curve drawn across one of your own seasons is a chart of the academic calendar.

### Picture it

```
  COHORT TRIANGLE - % still active, bracketed, day 0 = first order

  cohort      n     W0    W1    W2    W3    W4
  ────────────────────────────────────────────────
  Sep wk1    61    100%   44%   31%   28%   27%
  Sep wk3    78    100%   47%   35%   31%   30%
  Oct wk1    96    100%   58%   44%   41%    -
  Oct wk3   124    100%   61%   48%     -    -
  Nov wk1   142    100%   63%     -    -     -
                          ▲
                   read DOWN this column:
                   44 → 63 means the product
                   got better, not the week
```

Each row is one cohort's story over time; each column compares different cohorts at the same age, which is the only way to tell whether your changes helped.

### Where it lives

M01's chapter three is the loop this chapter closes: there you wrote a baseline from five conversations — skipped lunch twice last week, pays a senior ₹20 on Thursdays. That was behaviour before your product existed; your retained cohort's behaviour now is the after. A 30% flat line whose users have stopped skipping lunch means you changed something real; an unchanged baseline means a habit that is not solving the problem you set out to solve.

Around it: chapter three's funnel is a cohort's first week; chapter six attacks the gap between the two curves; chapter ten is the same measurement from the other side.

### Words to own

| Term | Plain meaning | Example |
|---|---|---|
| Day zero | The moment a user's clock starts | Their first completed order |
| Cohort | A group sharing when they joined or activated | Oct week 1, n = 96 |
| N-day retention | Active on day N exactly | Daily products only |
| Bracketed retention | Active at any point during week N | The sensible default |
| Cohort triangle | Cohorts down, age across | The diagram above |
| Flattening point | Where the curve stops falling | 30% at week 4 |
| Natural frequency | How often a real user would return | Lunch is daily |
| Resurrection | A lapsed user who became active again | Returns after exams |
| Survivorship | Only hearing from people who stayed | Your loudest users |
| Retained user | Active in the period being measured | Ordered in week 3 |

### Tradeoffs and gotchas

**A cohort under about fifty people is an anecdote.** At n = 20 one person is five percentage points, so a cohort can "improve" by eleven points because two friends ordered on a Thursday. Write n beside every row.

**Your flattening point may not exist yet.** A curve still falling at week four has told you nothing final; say "not yet flat" rather than inventing a number.

**A generous definition of active destroys the measurement.** "Opened the app" flatters every row. Keep your taxonomy's definition even when a number becomes embarrassing.

**Resurrection is not churn, and not always yours.** A student who stops during exams and returns next term is resurrected, and counted properly some cohorts tick upward — the curve smiles. But a smile is only real if your bracket contains the absence, and if you messaged 200 lapsed users the uptick is your message.

**Survivorship shapes everything you hear.** The users who talk to you are the ones who stayed; the 70% who left hold the information.

### Verify it

AI will produce a retention chart that looks professional and is wrong in one of four specific ways.

1. **Ask what day zero is.** Signup rather than first value is the most consequential error here, and it produces a curve that recommends a pivot.
2. **Ask for the definition of active, in your taxonomy's words.** If the answer is "opened the app", the chart is decoration.
3. **Demand n for every cohort**, and label any row under about fifty a lead rather than a result.
4. **Check whether the curve was smoothed.** A missing week drawn as a straight line between two real points is invented data, and tools do this quietly.

Red flags: no day-zero definition; "active users" with no stated action; a sliding "last 30 days" cohort you cannot restate.

## Walkthrough
1. Write down, from your taxonomy, the exact event that counts as active. Copy the wording rather than paraphrasing it.
2. Write down your product's natural frequency in one sentence, and choose weekly or daily brackets to match. Record the choice.
3. Define day zero twice: once as signup, once as the user's first occurrence of the active event. You will build both curves.
4. Build the signup-anchored curve for your earliest cohort, with n stated, and record the value at each period.
5. Build the value-anchored curve for the same cohort, counting only users who ever reached the active event, and record it beside the first.
6. Write one sentence on the gap between the two curves. That gap is the size of your activation problem and chapter six's brief.
7. Split your users into at least four cohorts by the week they first reached value, and build the cohort triangle with n on every row.
8. Read down one column and write one sentence: did the product get better between the earliest and latest cohort at the same age, and by how many points?
9. Beside each cohort row, note the dominant acquisition channel from chapter one's log, and say whether any column improvement could be explained by recruiting rather than by the product.
10. Mark any period that overlaps an exam week, a vacation or a festival, and note which cells must not be compared with which.
11. Open your M01 persona and baseline, and write two sentences: what the baseline behaviour was before you built anything, and what your retained cohort's behaviour is now. Say plainly if you cannot measure it.

## Talk to AI
```prompt
CONTEXT:      I am a student in a team of three in India. I direct AI and
              verify its output; I do not write code. Our product is live:
              [describe it in two sentences]. From my event taxonomy, the
              action that counts as ACTIVE is exactly: [paste the event and
              its firing condition]. Our product's natural frequency is
              [daily / weekly / other, and why]. I have [number] users who
              have ever been active, first activating between [date] and
              [date]. My signup counts by week are: [paste]. My
              activations by week are: [paste]. Academic calendar events in
              this period: [exam weeks, vacations, festivals]. Before we
              built anything, the baseline behaviour we recorded from five
              user conversations was: [paste it].

GOAL:         An honest retention picture: two curves with different day
              zeroes, a cohort triangle I can read down a column, and a
              clear statement of which numbers I am not allowed to
              interpret yet.

REQUIREMENTS: 1. Tell me whether to use N-day or bracketed retention for
                 my natural frequency, and show me roughly how different
                 the same product would look under each.
              2. Build both curves: day zero at signup, and day zero at
                 first active event. State n for each. Then tell me what
                 the gap between them means and which one I should quote
                 when someone asks about retention.
              3. Build the cohort triangle with n on every row. Tell me
                 which rows are too small to interpret and what the cutoff
                 should be.
              4. Read down one column for me and say whether my product
                 improved between my earliest and latest cohort, in
                 percentage points - then tell me what else could explain
                 that change besides the product.
              5. Mark every cell affected by my academic calendar and say
                 which comparisons are now invalid.
              6. Tell me whether my curve has flattened yet. If it has not,
                 say so plainly and tell me how many more periods I need.
              7. Compare my retained users' current behaviour against the
                 M01 baseline I pasted. Tell me exactly what I would have
                 to measure to make that comparison properly, and whether
                 my current events can support it.
              8. Give me three ways this chart could flatter me, and the
                 check for each.

CONSTRAINTS:  Free tier only. State n beside every percentage. Never
              change my definition of active - if you think it is wrong,
              say so separately rather than quietly substituting a better
              one. Do not interpolate or smooth across missing periods;
              show gaps as gaps. Plain English, every term explained on
              first use.

NON-GOALS:    Do not design onboarding changes; chapter six does that. Do
              not compute churn or predict it; chapter ten does that. Do
              not propose experiments. Do not write code or modify files.

ACCEPTANCE:   - Both curves are labelled with their day zero
              - Every row of the triangle carries its n
              - You told me which rows I may not interpret
              - Calendar-affected cells are marked
              - You said plainly whether my curve has flattened
              - I know what I would need to measure to close the loop on
                my pre-build baseline

ASK:          First ask me up to five questions about my active event, my
              natural frequency and my calendar. Then answer requirement 1
              only, and stop before building any curve.
```

## Stuck? Ask AI
```prompt
Draw me, in plain text, the same product's retention curve four times: anchored at signup, anchored at first value, measured as N-day, and measured as bracketed weekly. Use plausible numbers for a student product in India. Then explain which pair of these differences is about my onboarding and which is about my measurement choice.
```

```prompt
Here is my cohort triangle: [paste it with n on every row]. Do not congratulate me. Tell me which cells I am not allowed to interpret and why, which column comparison is the only one worth making, and at least three explanations other than "the product improved" for any change you see down that column.
```

```prompt
Explain resurrection and the smile-shaped retention curve using an Indian college canteen that empties during exams and refills in the new term. Then tell me how I would distinguish genuine resurrection from an uptick caused by my own WhatsApp reminder to lapsed users, using only data I would already have.
```

## Quiz

### Q1 · L4
Anchored at signup, your week-four retention is 4%. Anchored at each user's first completed order, it is 31%. What do the two numbers together tell you?
- (a) The 4% is wrong and should be discarded
- (b) Most signups never reach value, but those who do form a habit ✅
- (c) The product needs a pivot, since 4% is unsustainable
- (d) The 31% is inflated by survivorship and cannot be used
> The gap between the two curves is the size of your activation problem. Reading only the signup curve blames the product for an onboarding failure, which is how a team decides to pivot when it should fix the first five minutes.

### Q2 · L4
In a cohort triangle, what does reading down a single column tell you?
- (a) How one group of users behaved over successive weeks
- (b) Whether different cohorts, compared at the same age, are retaining better ✅
- (c) The total number of active users each week
- (d) The churn rate for the most recent cohort
> A column holds cohorts at equal age, so it is the only fair comparison across time and the only way to see whether what you shipped actually helped. Rows tell one group's story; columns tell the product's.

### Q3 · L3
Your product is a weekly habit. Someone reports retention using N-day retention and the numbers look catastrophic. What is the most likely explanation?
- (a) The product genuinely has no retention
- (b) N-day asks whether a user was active on that exact day, which a weekly product fails by design ✅
- (c) The cohorts are too small
- (d) Day zero was set at first value instead of signup
> N-day is only appropriate for genuinely daily products. Measuring a weekly rhythm with it reports absence on six days out of seven as churn, which is a measurement artefact rather than a finding.

### Q4 · L5
A cohort of 22 users moves from 18% to 32% retention between two weeks, and a teammate wants to present it as proof the new onboarding worked. What is the strongest objection?
- (a) Onboarding changes take longer than two weeks to show up
- (b) At n = 22 the whole move is about three people, so it cannot distinguish a real effect from noise ✅
- (c) Retention should never be compared across weeks
- (d) The cohort should have been anchored at signup
> Each person is roughly four and a half percentage points at that size. The number is a story about three individuals, which is worth investigating by talking to them and worth nothing on a slide.

### Q5 · L4
Your weekly retention curve drops to almost zero across four weeks and then returns to previous levels. Those four weeks were end-of-semester exams. How should this be reported?
- (a) As four weeks of severe churn followed by strong reacquisition
- (b) As a calendar effect, with the affected cells marked and excluded from comparisons ✅
- (c) As proof that the product has a resurrection loop
- (d) As evidence the active definition is too strict
> A canteen app loses everybody during exam break and that is a holiday, not churn. Marking calendar-affected cells is what stops the next cohort comparison from measuring the academic timetable.

## Go deeper
- [Lenny's Newsletter: what good retention looks like by product type](https://www.lennysnewsletter.com/p/what-is-good-retention-issue-29)
- [Amplitude: retention analysis and the N-day versus unbounded choice](https://amplitude.com/docs/analytics/charts/retention-analysis)
