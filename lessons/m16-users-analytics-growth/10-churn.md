---
id: m16-10-churn
module: m16-users-analytics-growth
slug: churn
title: "Churn: Measuring It, Predicting It, Reducing It"
order: 10
duration_min: 30
is_free: false
mission: false
volatility: low
last_reviewed: 2026-11-16
---

## Key learnings
- Report churn as a pair of numbers, never as one, and state the denominator
- Define "lapsed" as a rule, because in a free product stopping is silent
- Use three readable rules instead of a prediction model at your size
- Interview people who left, using past-behaviour questions rather than opinions
- Keep win-back results in their own cohort, out of your retention curve

## Lesson

### The pain

A team reports monthly churn of 12% and everyone nods, because 12% sounds survivable.

Then somebody asks what the denominator was. The number was computed over users who had placed at least two orders — people who had already proved they liked it. Of six hundred signups, a hundred and seventy ever reached value, and twenty-one of those stopped last month. Twelve per cent is arithmetically correct and it describes twenty-eight per cent of the people who arrived.

The four hundred and thirty who never got anywhere do not appear in it at all. They are the largest group in the business and the number is built so that it cannot see them. In M18 a panel found this with one question, and the team had no answer, because the number had never been wrong — only incomplete.

### What M07 covered, and what is new here

M07's chapter eight gave churn one paragraph: the share of a cohort that stopped, meaningless without a window attached, and be careful in a product with seasons because a canteen app loses everybody during exam break and that is a holiday rather than churn. All of that is correct and assumed here.

New in this chapter: the denominator problem above, which is where almost every student churn number goes wrong; defining lapsed when there is no cancel button; leading indicators you can compute by hand; why a prediction model is not available to you at this size; exit interviews with people who have already left; and win-back, measured honestly.

### The idea

**Churn is a pair of numbers, not one: the share who never reached value at all, and the share who reached it and then stopped. Report both, with the denominator and the window stated, or you are describing a fraction of your users and calling it the business.**

### The analogy

The canteen manager notices that the boy who came every single day for a month has not been in for two weeks. There was no cancellation form and no announcement. She noticed an absence.

**Where the analogy breaks:** she knows faces, so absence is visible to her without any effort. Your product has no faces. Nobody on your team will ever notice that a particular user stopped, because there is nothing to notice — a user who leaves simply does not appear. So absence has to be defined as a rule and computed deliberately, and if you do not build that list, silent churn is invisible until the totals start falling months later.

### Picture it

```
  THE TWO CHURN NUMBERS, ONE MONTH, WINDOW = 28 DAYS

  signed up            600  ████████████████████  100%
  ever activated       170  ██████                 28%
                            └─ never activated 430 = 72%  ◀ (1)
  active this month    149
  lapsed this month     21  ▌                      12% of 170  ◀ (2)

  (1) never-activated share   72%   the bigger problem
  (2) churn among activated   12%   the number usually quoted
```

Number two is what teams report and number one is usually the larger failure, which is why both belong on the same slide with their denominators written beside them.

### Defining lapsed, because nobody cancels

In a paid product with subscriptions, churn has a moment: the cancellation. In a free product there is no such event, so a user who has lost interest and a user who is busy this week look identical. You have to choose a rule.

The usual rule is a multiple of your natural frequency from chapter four. A daily product might define lapsed as no active event for fourteen days; a weekly product as four weeks. What matters more than the multiple is writing it down in the taxonomy and not changing it later.

Two consequences. A lapsed user is not necessarily gone — chapter four's resurrection is this person returning — so "lapsed" is a state, not a verdict. And because the rule has a lag built in, your churn number always describes a period that ended some time ago, which is the price of measuring silence.

### Leading indicators, and why you do not get a model

You want to know who is about to leave while you can still do something. Three signals are computable from events you already have.

**Falling frequency.** Someone who ordered three times a week and now orders once is on their way out. The slope matters more than the level.

**A failed last session.** If a user's most recent session contained an error, a payment failure or a dead-end, there is a reasonable chance that session is why they stopped. This is the most actionable of the three because it names the cause and a person at the same time.

**A long gap relative to their own pattern.** A weekly user absent for three weeks is a stronger signal than a monthly user absent for three weeks. Compare each user against their own rhythm, not against an average.

Now the blunt part. At a few hundred users, a churn prediction model has almost nothing to learn and every reason to fit noise — twenty-one lapsed users in a month is not a training set, and the model's confident output cannot be checked by anybody. A list produced by three readable rules is better on every dimension that matters: you can explain it, you can act on each row, and you can tell when it is wrong. Ask for the model and you will get one; it will be theatre.

### Talking to the people who left

Chapter four warned about survivorship: the users who talk to you are the ones who stayed. The thirty who left have the information and no reason to answer. Get it anyway, and WhatsApp is why this is possible for an Indian student product in a way it would not be elsewhere.

Message twenty lapsed users individually. Expect three to five replies, and treat them as the most valuable data in the module. The question matters, and M01's method applies unchanged: do not ask "why did you stop using it?", which invites politeness or silence. Ask about the last time instead — "the last time you needed lunch between labs, what did you actually do?" — because that produces a fact rather than a verdict.

What you are listening for is the workaround they went back to. That tells you whether you lost to a competitor, to the old habit, or to a bug on their last session.

### Reducing it, and measuring the reduction honestly

Three levers, in order of return at your size.

**Fix the thing that broke on their last session.** Cheapest and most direct. The failed-last-session list from above is a list of specific repairs.

**Attack the never-activated pool instead.** Four hundred and thirty people who never reached value is a larger prize than twenty-one who left, and chapter six is the whole method. Most teams spend their effort on the smaller pool because the churned feel like a loss and the never-activated feel like strangers.

**Win-back, once, measured separately.** One personal message to lapsed users, with a reason to return that is not a discount. Keep the responders in their own cohort, because — as chapter four warned — an uptick caused by your own message is your message, not your product, and letting it into your retention curve corrupts the one number you most need to trust.

If you have payments, there is a fourth: **involuntary churn**, where a user intended to stay and a failed UPI mandate or an expired card ended it. That is M17's dunning work, and it is usually the cheapest churn to recover because the intent never went away.

### Where it lives

Chapter four is the same measurement from the other side, and the two must agree: if retention flattens at 30%, churn among activated users cannot be 60% a month. Chapter six owns the never-activated pool. Chapter three finds the bug on the last session and chapter five shows it to you. M17 turns churn into lifetime value, and M18 is where the denominator question gets asked.

### Words to own

| Term | Plain meaning | Example |
|---|---|---|
| Churn | Share of a group that stopped, in a window | 21 of 170 in 28 days |
| Denominator | Who the share is computed over | Activated users, not signups |
| Never-activated | Signed up, never reached value | 430 of 600 |
| Silent churn | Stopping with no cancellation event | A free product |
| Lapsed | No active event for your stated rule | 28 days for a weekly product |
| Voluntary churn | The user chose to stop | Went back to carrying a tiffin |
| Involuntary churn | Stopped without intending to | A failed UPI mandate |
| Leading indicator | A signal before the user leaves | Frequency falling |
| Risk list | Named users matching your rules | Twelve rows, readable |
| Win-back | One attempt to bring a lapsed user back | Its own cohort |

### Tradeoffs and gotchas

**Churn with no window is not a number.** "Our churn is 12%" says nothing until you add "in twenty-eight days, among users who had activated". Both halves, every time.

**A blended churn rate hides the segment that is failing.** Split it by chapter one's acquisition channel and by device class. Users recruited in a classroom and users who tapped a link at midnight do not churn at the same rate, and the blend tells you about neither.

**Seasons are not churn.** Exam weeks, vacations and placement season empty a campus product. Mark those windows and say so, rather than reporting a holiday as a collapse.

**Do not change the lapsed rule because a number got ugly.** Moving from fourteen days to twenty-eight halves your churn rate overnight and changes nothing real. If you must change it, every earlier figure needs a dated note.

**Small numbers still bite here.** Twenty-one of a hundred and seventy is readable; three of twelve is not. State n beside the churn rate exactly as you do for retention.

**Chasing the churned is usually the wrong priority.** The never-activated pool is normally several times larger and far more tractable. Work out both sizes before deciding where the week goes.

### Verify it

Ask AI for a churn analysis and you will get a confident percentage with an invisible denominator.

1. **Ask what the denominator is, explicitly.** **A churn number that excludes the users who never activated** is the central error of this chapter, and it makes a product look healthy while most arrivals go nowhere.
2. **Ask for the window and the lapsed rule in one sentence each.** No window means no number; no lapsed rule means somebody chose one silently.
3. **Check it against chapter four's retention.** If churn and the flattening point tell different stories, one of them has a different definition of active, and finding out which is the task.
4. **Refuse any churn prediction model at your size.** **A model trained on a few hundred users and twenty lapsed examples** cannot be validated and cannot be explained; ask for three rules and a named list instead.
5. **Check whether a win-back campaign is inside the retention curve.** An uptick produced by your own message presented as product retention is a serious misreport, and it is usually accidental.
6. **Read the exit-interview questions before they are sent.** If they ask for opinions about your product rather than what the person did the last time they had the problem, they will produce politeness.

Red flags: a churn rate with no denominator; the lapsed rule changed in the week the number improved; churn blended across channels; a churn model with a confidence score; a win-back uptick called retention; exit interviews run only with people who still like you; and a churn number that counts signups from the last three days as retained.

## Walkthrough
1. Write your lapsed rule in one sentence, derived from the natural frequency you recorded in chapter four, and put it in the taxonomy file.
2. Count four numbers for the last complete window: total signups, ever activated, active in the window, and lapsed in the window.
3. Compute both churn numbers from them: the never-activated share, and churn among activated users. Write the denominator and the window beside each.
4. Compare the second number against chapter four's flattening point and say in one sentence whether they agree. If they do not, find out which definition of active differs.
5. Split churn by acquisition channel from chapter one's log, and by device class. State n for every split and mark any split too small to read.
6. Mark every window overlapping an exam period, vacation or festival, and label those figures as calendar-affected.
7. Build the risk list using three rules: falling frequency against the user's own pattern, an error or failure in their most recent session, and a gap longer than their personal rhythm.
8. Look at the failed-last-session rows and find out what actually broke, using chapter five's replays. These are specific repairs, not a trend.
9. Message twenty lapsed users individually on WhatsApp with one past-behaviour question. Record every reply verbatim.
10. Compare the size of the never-activated pool against the lapsed pool, and write one sentence deciding where this week's effort goes, with the numbers behind it.
11. If you run a win-back, define its cohort before sending, and write down how you will keep its results out of the retention curve.

## Talk to AI
```prompt
CONTEXT:      I am a student in a team of three in India. I direct AI and
              verify its output; I do not write code. Our product is live
              and free: [describe it in two sentences]. From my taxonomy,
              ACTIVE means exactly: [paste the event and firing condition].
              Our natural frequency is [daily/weekly/other]. For the last
              complete window I have: [total signups], [ever activated],
              [active in window], [lapsed in window]. Retention from first
              value flattens at [number]% at week [number]. Acquisition
              channels available as a dimension: [list them]. Academic
              calendar in this period: [exam weeks, vacations, festivals].
              We have [number] total users, so my numbers are small.

GOAL:         An honest churn picture with both denominators stated, a
              readable risk list built from rules rather than a model, and
              exit-interview questions that produce facts.

REQUIREMENTS: 1. Help me write the lapsed rule for my natural frequency,
                 and tell me what changes if I pick two times rather than
                 three times the frequency.
              2. Compute both churn numbers from my figures, stating the
                 denominator and window beside each, and tell me which one
                 a panel would ask about first.
              3. Check my churn number against my retention flattening
                 point. If they disagree, tell me the most likely reason
                 and how I would confirm it.
              4. Tell me which of my splits - channel, device, calendar -
                 are large enough to read, and which I must label as leads.
              5. Give me exactly three computable leading-indicator rules
                 for my product, each expressed so I could produce a list
                 of named users from my own events today.
              6. Tell me plainly whether a churn prediction model is
                 appropriate at my size, with the reasoning, and what I
                 should use instead.
              7. Write five exit-interview questions in the style of asking
                 about the last time something happened rather than asking
                 for opinions about my product. Tell me which single one to
                 send if I only get one message.
              8. Compare the size of my never-activated pool against my
                 lapsed pool and tell me where a week of effort is better
                 spent, with the arithmetic.
              9. If I run a win-back message, tell me how to keep its
                 effect out of my retention curve and what cohort naming I
                 should use.

CONSTRAINTS:  Free tier only. Never quote a churn rate without its
              denominator and window. State n beside every percentage. Do
              not propose a machine-learning model. Do not propose a
              discount as the win-back reason. Plain English, every term
              explained on first use.

NON-GOALS:    Do not redesign onboarding; chapter six does that. Do not
              design experiments. Do not compute lifetime value; that is a
              later module. Do not write code and do not modify files.

ACCEPTANCE:   - Both churn numbers are stated with denominator and window
              - My churn number and my retention number are reconciled
              - My three rules produce a list of named users from events I
                already have
              - You refused a prediction model and explained why
              - My exit-interview questions ask about past behaviour
              - I know which pool deserves this week's effort, with numbers

ASK:          First ask me up to five questions about my active definition,
              my frequency and my calendar. Then answer requirements 1 and
              2 only, and stop.
```

## Stuck? Ask AI
```prompt
Here is my churn calculation: [paste the numbers and how you computed them]. Interrogate the denominator before anything else. Who is excluded from it, how many people is that, and how would the number look if they were included? Then tell me the one sentence a panel would use to take this number apart.
```

```prompt
Explain silent churn in a free product using a college canteen where a daily regular simply stops appearing. Then help me choose a lapsed rule for a product with [daily/weekly] natural frequency, and show me how the same users would be classified under a 14-day rule and a 28-day rule.
```

```prompt
I want to message twenty lapsed users and I will probably get four replies. Write me one question that produces a fact rather than a polite answer, in the style of asking about the last time they had the problem. Then tell me three questions I must not send, and what each of them would produce instead.
```

## Quiz

### Q1 · L4
A team reports 12% monthly churn, computed over users who had placed at least two orders. Of 600 signups, 170 ever activated. What is wrong?
- (a) Nothing; churn should only count engaged users
- (b) The denominator excludes the 430 who never reached value, which is the larger failure ✅
- (c) The window should be calendar months rather than 28 days
- (d) Two orders is too low a bar for counting someone as retained
> The number is arithmetically right and describes 28% of arrivals. Churn is a pair of numbers — the never-activated share and churn among activated users — and quoting only the second hides the bigger problem.

### Q2 · L3
Your product is a free weekly habit with no cancellation step. How is churn made measurable?
- (a) By asking users to confirm when they stop using it
- (b) By defining a lapsed rule, such as no active event for four weeks ✅
- (c) By counting uninstalls reported by the app store
- (d) By treating anyone inactive for seven days as churned
> Silence has to be turned into a rule. Two or three times the natural frequency is the usual choice, and the rule matters less than writing it down and not changing it when a number becomes inconvenient.

### Q3 · L5
You have 400 users and 21 lapsed last month. AI offers a churn prediction model with per-user risk scores. What should you do?
- (a) Use it, but only for users with high confidence scores
- (b) Refuse it and ask for three readable rules producing a named list ✅
- (c) Use it after collecting another month of data
- (d) Use it to rank users and verify the top ten by hand
> Twenty-one examples cannot train or validate anything, so the scores would be confident noise nobody can check. Three explainable rules give you a list you can act on row by row and correct when it is wrong.

### Q4 · L4
You message 200 lapsed users and 30 return, producing an uptick in your week-eight retention curve. How should this be handled?
- (a) Report it as evidence the product retains users well
- (b) Keep the responders in their own cohort, out of the retention curve ✅
- (c) Exclude those users from all future analysis
- (d) Treat it as resurrection, which belongs in the curve
> The uptick was produced by your message, not your product. Letting campaign effects into the retention curve corrupts the one number the module depends on, which is why win-back gets its own cohort.

### Q5 · L4
Which exit-interview question is most likely to produce usable information from someone who stopped using your app?
- (a) Why did you stop using our app?
- (b) What features would make you come back?
- (c) The last time you needed lunch between labs, what did you actually do? ✅
- (d) How would you rate your experience out of ten?
> A question about a specific past event produces a fact, and the fact you most want is the workaround they returned to. The others invite politeness, feature design by the user, or a number with no mechanism behind it.

## Go deeper
- [Lenny's Newsletter: churn benchmarks and how to read them](https://www.lennysnewsletter.com/p/churn-benchmarks)
- [Amplitude: lifecycle, lapsed users and resurrection analysis](https://amplitude.com/docs/analytics/charts/lifecycle)
