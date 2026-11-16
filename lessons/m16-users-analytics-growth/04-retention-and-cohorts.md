---
id: m16-04-retention-and-cohorts
module: m16-users-analytics-growth
slug: retention-and-cohorts
title: "Retention Curves, Cohorts and the Flattening Point"
order: 4
duration_min: 30
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

A team plots their first real retention curve in week three of this module. Day zero is signup. By week four, 4% of each cohort is still active. The curve does not flatten; it falls to the floor. They spend an evening seriously discussing a pivot.

The next morning somebody re-plots the same data with day zero set at each user's first completed order instead of their signup, counting only people who ever got that far. The curve flattens at 31% by week four and stays there.

Both numbers are correct. They answer different questions. The first says that most people who sign up never reach value, which is an onboarding problem. The second says that among people who did reach value, roughly a third built a habit, which means there is a product here. Measuring from signup told them to throw the product away when the correct action was to fix the first five minutes. One choice of day zero nearly ended the company.

### What M07 covered, and what is new here

M07's chapter eight defined a cohort, showed the shape of a retention curve, explained that the flattening point is the share of people for whom the product became a habit, insisted that you define "active" as the action that is the point of your product, and warned that the period must match your product's natural frequency. That vocabulary is assumed here.

New in this chapter: choosing day zero, which M07 did not raise; the two competing definitions of retention and when each lies; the cohort triangle, which is how you tell whether your product changes actually helped; the minimum cohort size for an honest percentage; resurrection; and closing the loop on the baseline you recorded in M01 before you built anything.

### The idea

**A retention curve means nothing until you say what day zero is, and day zero should be the moment the user first received value — not the moment they filled in a form.**

Signup-anchored retention measures your onboarding and your product mixed together, and when the two disagree it reports the worse of them. Value-anchored retention measures the product. You want both, labelled, side by side, because the gap between them is the size of your onboarding problem and chapter six's whole agenda.

### The analogy

Of the students who ate at the canteen during the first week of term, how many were still eating there in week eight? That is a cohort and a retention measurement, and the canteen can answer it from a register with names in it.

**Where the analogy breaks, twice.** First, the canteen has a captive audience: students who cannot easily leave campus at lunchtime will keep coming regardless of how good the food is, so a flat line there may measure convenience rather than preference. Your users can close your app forever at no cost, so your flat line is a stronger claim than the canteen's. Second, the canteen empties completely during exam break and everyone returns in July. On a chart that looks like total churn followed by a miracle. It is a holiday. Your product has the same seasons — exam weeks, vacations, placement season — and a curve drawn across one of them is a chart of the academic calendar.

### Two definitions, and saying which you used

**N-day retention** asks: was this user active *on* day N exactly? It is the right measure only for products that are genuinely daily, and it produces brutally low numbers for everything else. A weekly product measured as N-day retention looks dead.

**Bracketed retention** asks: was this user active at any point *during* week N? This is the right default for almost every student product, because it tolerates the natural rhythm of a user who orders on Tuesday one week and Friday the next.

Neither is more honest. What is dishonest is not saying which one a number came from, because the same product can be at 8% by one definition and 34% by the other. Write the definition beside the number, every time.

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

Reading across a row tells you about one group of people. Reading down a column tells you about your product, because every cell in a column is measured at the same age — so a column that rises from 44% to 63% is evidence that something you shipped in October worked. That is the single most useful thing a cohort triangle does, and it is why cohorts exist at all rather than one blended number.

### Resurrection, and the curve that smiles

Some users come back after weeks of absence. A student who stops ordering during exams and returns in the new term is **resurrected**, not churned, and a product with a real academic rhythm will have many of them. Counted properly, some cohorts stop falling and tick upward — the curve smiles.

Two warnings. A smile is only real if your bracket is long enough to contain the absence, so a weekly bracket will record an exam break as churn no matter what happens afterwards. And resurrection is easy to confuse with your own re-engagement effort: if you sent a WhatsApp reminder to 200 lapsed users, the uptick is your message, not your product, and it belongs in chapter ten's ledger rather than in this curve.

### Where it lives

M01's chapter three is the loop this chapter closes. There you wrote a baseline from five real conversations: skipped lunch twice last week, pays a senior ₹20 on Thursdays, asked a friend to bring food up four times this month. Those were measurements of behaviour before your product existed. Your retained cohort's behaviour now is the measurement after. If your flat line is 30% and those users have stopped skipping lunch, you changed something real. If your flat line is 30% and the baseline behaviour is unchanged, you have a habit that is not solving the problem you set out to solve — which is worth knowing, and is a sentence M18's panel will ask you for.

Chapter three's funnel is the first week of a cohort's life, in detail. Chapter six attacks the gap between the signup curve and the activation curve. Chapter ten is churn, the same measurement read from the other side. M10 matters more here than teams expect: if an AI feature in your product gives a bad answer on a user's second visit, that user is in this curve's steep section and your retention is partly a model-quality number. M13's warehouse is where these cohorts eventually get computed properly.

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

**A cohort under about fifty people is an anecdote.** At n = 20, one person is five percentage points and a cohort can "improve" by eleven points because two friends ordered on a Thursday. Write n beside every row and say out loud which rows you are not allowed to interpret.

**Your flattening point may not exist yet.** It takes several periods to appear, and a six-month track gives you perhaps eight to twelve weekly periods of real data. A curve still falling at week four has not told you anything final. Say "not yet flat" rather than inventing the number you would like.

**A generous definition of active destroys the measurement.** "Opened the app" is the classic, and it flatters every row of the triangle. Keep the definition from your taxonomy and do not change it because a number became embarrassing. If you must change it, every earlier row becomes incomparable and needs a dated note.

**Cohorts of very different sizes do not compare cleanly.** A 61-person cohort recruited by hand in a classroom and a 142-person cohort from a WhatsApp link are different populations as well as different weeks. Note the acquisition channel beside each cohort, or your column is measuring your recruiting rather than your product.

**Survivorship will shape everything you hear.** The users who talk to you are the ones who stayed. The 70% who left have the information you need and no reason to answer. Chapter ten is about going after them anyway.

### Verify it

AI will produce a retention chart that looks professional and is wrong in one of six specific ways.

1. **Ask what day zero is.** **Retention measured from signup rather than from first value** is the single most consequential error in this chapter and it produces a curve that recommends a pivot.
2. **Ask for the definition of active, in the words of your taxonomy.** If the answer is "opened the app" or "was logged in", the chart is decoration.
3. **Ask whether it is N-day or bracketed, and whether the period matches your natural frequency.** A weekly product measured daily looks like a collapse.
4. **Demand n for every cohort.** Any row under about fifty gets labelled as a lead, not a result.
5. **Check that cohorts are defined by a date range you can state**, not by a tool's default "last 30 days" that slides every time you reload. A sliding cohort cannot be compared with yesterday's chart.
6. **Check whether the curve was smoothed or interpolated.** A missing week drawn as a straight line between two real points is invented data, and tools do this quietly.

Red flags: a curve with no day-zero definition; "active users" with no stated action; a cohort triangle with no n column; a flat line declared after two periods; a curve that crosses an exam break without a note; and a number that improved on the same day somebody changed the definition of active.

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
