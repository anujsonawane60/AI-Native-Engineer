---
id: m16-10-churn
module: m16-users-analytics-growth
slug: churn
title: "Churn: Measuring It, Predicting It, Reducing It"
order: 10
duration_min: 15
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

A team reports monthly churn of 12% and everyone nods, because 12% sounds survivable. Then somebody asks for the denominator: users who had placed at least two orders, people who had already proved they liked it.

Of six hundred signups, a hundred and seventy ever reached value, and twenty-one stopped last month. Twelve per cent is arithmetically correct and describes twenty-eight per cent of the people who arrived. The four hundred and thirty who never got anywhere are the largest group in the business, and the number cannot see them.

### What M07 covered, and what is new here

M07 gave churn one paragraph: the share of a cohort that stopped, meaningless without a window. New here is the denominator problem, defining lapsed when there is no cancel button, rules instead of a model, and talking to people who left.

### The idea

**Churn is a pair of numbers, not one: the share who never reached value at all, and the share who reached it and then stopped. Report both, with the denominator and the window stated, or you are describing a fraction of your users and calling it the business.**

In a paid product churn has a moment: the cancellation. In a free product there is none, so a user who lost interest and a user who is busy this week look identical, and you must choose a rule — a multiple of chapter four's natural frequency, such as no active event for fourteen days for a daily product or four weeks for a weekly one. What matters more than the multiple is writing it in the taxonomy and never changing it. A lapsed user is not necessarily gone: resurrection is this person returning.

### The analogy

The canteen manager notices that the boy who came every day for a month has not been in for two weeks. No cancellation form, no announcement. She noticed an absence.

**Where the analogy breaks:** she knows faces, so absence is visible to her for free. Your product has no faces, and nobody will notice that a particular user stopped, because a user who leaves simply does not appear. Absence must be defined as a rule and computed deliberately, or silent churn stays invisible until the totals fall.

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

Three signals of leaving are computable from events you already have: **falling frequency** against the user's own pattern, where the slope matters more than the level; **a failed last session**, whose error names a cause and a person at once; and **a gap long for that user's own rhythm**.

### Where it lives

Chapter four is the same measurement from the other side, and the two must agree: if retention flattens at 30%, churn among activated users cannot be 60% a month. The levers sit on other chapters' ground: fix what broke on their last session, which chapter three finds and chapter five shows you; attack the never-activated pool, chapter six's method and usually several times larger; run win-back once, with a reason that is not a discount, keeping responders in their own cohort. With payments there is a fourth, **involuntary churn**, where a failed UPI mandate ends something the user meant to keep — M17's dunning work.

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

**You do not get a prediction model at this size.** Twenty-one lapsed users is not a training set and nobody can check the output. Three readable rules give you a list you can explain, act on, and correct. Ask for a model and you will get one; it will be theatre.

**The people who left have the information and no reason to answer.** Message twenty lapsed users individually on WhatsApp; expect three to five replies. Do not ask "why did you stop?", which invites politeness — ask "the last time you needed lunch between labs, what did you actually do?", because the fact you want is the workaround they went back to.

**A blended rate hides the failing segment.** Split by acquisition channel and device class, state n beside each rate, and mark windows that overlap exam weeks, because a holiday is not a collapse.

**Do not change the lapsed rule because a number got ugly.** Fourteen days to twenty-eight halves your churn rate overnight and changes nothing.

### Verify it

Ask AI for a churn analysis and you will get a confident percentage with an invisible denominator.

1. **Ask what the denominator is, explicitly.** A churn number that excludes the never-activated is this chapter's central error.
2. **Ask for the window and the lapsed rule in one sentence each.** No window means no number.
3. **Refuse any churn prediction model at your size** and ask for three rules and a named list.
4. **Check whether a win-back campaign is inside the retention curve.** An uptick produced by your own message is not product retention.
5. **Read the exit-interview questions before they are sent.**

Red flags: a churn rate with no denominator; a lapsed rule changed in the week the number improved; a churn model with a confidence score.

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
