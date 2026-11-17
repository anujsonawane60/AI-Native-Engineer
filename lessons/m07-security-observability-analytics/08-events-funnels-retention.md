---
id: m07-08-events-funnels-retention
module: m07-security-observability-analytics
slug: events-funnels-retention
title: "Events, Funnels, Cohorts, Retention and Churn"
order: 8
duration_min: 15
is_free: false
mission: false
volatility: low
last_reviewed: 2026-11-16
---

## Key learnings
- Define an event so that every number you build on it stays trustworthy
- Read a funnel and name the step that is actually losing you people
- Explain what a retention curve flattening out tells you about your product
- Tell a vanity metric from one that would change a decision
- Say what belongs in a product event and what must never go into one

## Lesson

### The pain

A team presents in week nine. "Users love it." The evidence: 200 signups and a WhatsApp message from a friend saying it is cool.

Somebody asks how many people have ordered twice. Nobody knows, so they count rows in the database by hand: nine. Two hundred signed up; nine came back. The daily canteen habit they thought they had does not exist. It is not necessarily a bad product; it is an unknown one, because nobody decided in advance what to record about what people do.

### The idea

**An event is a named thing one person did, at a time, with a few properties. Funnels, cohorts, retention and churn are all just arithmetic on events.**

`order_placed`, with a user id, a timestamp, an item count and a total. Record the right dozen honestly and every question here becomes countable; record them carelessly and nothing built on them can be trusted. Naming is load-bearing:

- **Object then action, past tense, lower case with underscores:** `order_placed`, `menu_viewed`, `signup_completed`
- **Fire on success, not on intent:** `payment_succeeded` when the money is confirmed, not when Pay is tapped
- **Variation goes in properties, not the name:** `order_placed` with `payment_mode: "upi"`, never `order_placed_upi`
- **A dozen events beats a hundred,** and **never put personal data in one** — user id, not phone number

Write that list in a repository file, one line per event: when it fires, what it carries.

**A boundary, stated up front.** This chapter teaches the concepts. Doing it on real users — a full taxonomy, the real leak, growth loops, an experiment programme — is **M16, at the Company tier**. Here you learn to read the instruments; there you fly the plane.

### The analogy

The canteen's register says 180 plates went out today — which cannot tell you whether 180 students came once or 60 came three times. Those are two different businesses: a stall people pass, and a canteen people belong to. Record who collected each plate and you can ask the question that matters: of the students who came in the first week of term, how many are still coming in week eight?

**Where the analogy breaks:** a register is written by a person who understands what they are recording. Your events are fired by code, and if the code fires `order_placed` when the button is *tapped* rather than when the order is *saved*, your register faithfully records orders that never happened, with nobody to notice.

### Picture it

```
  SIGNUP ─▶ FIRST ORDER, 7 days, 412 students

  opened app       412  ████████████████████████  100%
  viewed menu      358  █████████████████████      87%
  added to cart    196  ███████████                48%  ◀ −39
  reached payment  171  ██████████                 41%
  order placed      94  █████                      23%  ◀ −18
```

A funnel is ordered steps with the count of people reaching each, and the two marked drops — 39 and 18 points — are where the product is losing students.

Three honest requirements that generated dashboards routinely break: steps genuinely **ordered**, so step three implies step two; counting **people, not events**, or one student adding four items inflates a step; and a **window** such as "within seven days of signup".

Resist the pull of the biggest drop. The 39 points between menu and cart may be healthy — people look at menus without being hungry. The 18 points between payment and ordering is smaller and far more alarming, because those people had decided to buy.

A **cohort** is a group defined by when they joined; one overall number mixes yesterday's joiners with July's and describes your marketing, not your product. **Retention** asks what share of a cohort is still active after N periods.

```
  % of the week-1 cohort still ordering, by week

  100 │ ●
      │
   50 │     ●
      │          ●    ●
   20 │                    ●    ●    ●    ●  ◀ flattens
    0 └────┴────┴────┴────┴────┴────┴────┴───
       W1   W2   W3   W4   W5   W6   W7   W8
```

The curve falls steeply and then, if you have a real product, **flattens**. That flat line — 18% here — is the share of people for whom the product became a habit, and the most informative number about a young product. A curve falling to zero means an activity, not a product; **churn** is the same fact from the other side.

### Where it lives

Chapters four to seven measured whether your *system* is healthy; this chapter measures whether your *product* works. These events are not chapter five's logs: logs debug one request, events count many people, and one stream doing both does neither well.

- M04 — where the data these events describe actually lives
- Chapter 9 — experiments and consent, which shapes these numbers directly
- M11 — the going-live checkpoint wants analytics before you go live
- M16, Company tier — the practicum: real users, leak-hunting, activation, growth loops, a real experiment programme

### Words to own

| Term | Plain meaning | Example |
|---|---|---|
| Event | A named thing one person did, with properties | `order_placed` |
| Taxonomy | Your written list of events and their meanings | A file in your repo |
| Funnel | Ordered steps with the count reaching each | Signup to first order |
| Cohort | A group defined by when they joined or what they did | Week-1 signups |
| Retention | Share of a cohort still active after N periods | 18% at week 8 |
| Flattening point | Where a retention curve stops falling | Your real product |
| Churn | Share of a cohort that stopped, in a window | 40% monthly |
| Activation | The first moment a user got real value | First order placed |
| Vanity metric | A number that changes no decision | Cumulative signups |
| Active | Your own definition of a meaningful action | "Placed an order" |

### Tradeoffs and gotchas

- **Define "active" as the action that is the point of your product,** measured over its natural frequency. "Opened the app" makes every chart look better and useless; measuring a weekly product daily makes healthy retention look like collapse.
- **Churn needs a window, and watch for seasons:** a canteen app loses everybody during exam break — a holiday, not churn.
- **A vanity metric goes up and changes no decision** — signups, cumulative views, installs. Ask what you would do if it halved.
- **Your analytics will never match your database, and that is normal.** Ad blockers, refused consent, lost 4G. Events for shapes; the database for anything exact.
- **Small numbers are stories, not statistics.** With 60 users, a step moving from 48% to 55% is three people. Go and talk to them.

### Verify it

AI will build you a funnel chart in minutes. These checks decide whether it means anything.

1. **Check when each event actually fires:** do the action and confirm it appears once. **Events fired on page load rather than on success** are the commonest fault.
2. **Count people, not events, at every funnel step.** One enthusiastic tester can double a step.
3. **Check the steps are really ordered and really windowed.** Otherwise it is a bar chart wearing a funnel's name.
4. **Check the definition of "active" behind any retention number.** "Opened the app" makes it decoration. **Counting signups as activation** is the same error: a signup is a form submission, not value received.
5. **Search your event properties for personal data** — phone numbers, emails, names, UPI ids. Anything found is a privacy and legal problem.

Red flags: the same event spelled two ways; a funnel with no window; percentages from fewer than a hundred people quoted as facts; cumulative totals presented as growth; "active users" with no stated definition; a retention curve over a period shorter than the natural frequency.

## Walkthrough
1. Write one sentence saying what a successful user of your product does, and how often. "Places an order on three days out of five" is a sentence; "engages with the platform" is not.
2. From that sentence, name the single action that counts as **active** for your product, and write it in your repository. Everything else in this walkthrough depends on it.
3. List the steps a brand-new user passes through from first opening your app to that active action. You should have four to six steps; if you have twelve, your onboarding is the finding.
4. Turn those steps into an event list: object, past-tense action, and the two or three properties each one carries. Keep it to a dozen events total.
5. Beside each event write exactly when it fires — on success, never on intent — and what would make it fire wrongly. Save this file; it is your taxonomy.
6. Check what your product already records, in the database or in logs, that could answer these questions today without any new instrumentation. Most teams find they can already compute first orders and second orders.
7. Build the funnel by hand for the last month, from whatever data you have, even if it means counting rows. Write the count and the percentage at each step.
8. Mark the two biggest drops in percentage points, then argue about which one is actually the problem. Write one sentence defending your choice.
9. Pick your earliest cohort of users and compute, by hand, how many were still active one week and four weeks later. Two numbers. Plot them with two more and look at the shape.
10. Write down one decision you would make differently if the flattening point were half what it is, and one you would make if it were double. If neither changes anything, you have chosen the wrong metric — go back to step 2.
11. Put the taxonomy file and the funnel numbers in your repository with today's date, so M16 starts from evidence rather than memory.

## Talk to AI
```prompt
CONTEXT:      I am a student in a team of three. I direct AI and verify its
              output; I do not write code. Our product: [describe it in two
              sentences]. Our users are students in India on budget Android
              phones. A successful user of our product does [the one action]
              about [how often]. We currently have [number] users and we
              record [what you already record, e.g. rows in a database,
              server logs]. We have no product analytics yet.

GOAL:         A written event taxonomy of at most twelve events, and a
              funnel and retention definition I can compute from it — both
              designed so the numbers would actually change a decision.

REQUIREMENTS: 1. Help me define "active" for my product in one sentence,
                 and tell me two tempting definitions I should reject and
                 why each one would flatter me.
              2. Propose at most twelve events using object_action past
                 tense, with the properties each should carry. For each, say
                 exactly when it must fire and name one way it could fire
                 wrongly.
              3. Define my signup-to-activation funnel: the ordered steps,
                 the time window, and whether each step counts people or
                 events. Explain why the window matters.
              4. Define my retention measurement: the cohort, the period
                 length matched to my product's natural frequency, and what
                 a healthy flattening point would look like for a product
                 of this kind.
              5. Tell me which of these numbers I could compute today from
                 what I already record, with no new instrumentation.
              6. List every field in my product that must never appear in
                 an event property, and what to send instead.
              7. Give me three numbers that would look impressive on a
                 slide and change no decision, and say what to replace each
                 one with.

CONSTRAINTS:  Free tier only. At most twelve events — argue me down rather
              than up. Explain every term on first use. Assume I verify by
              doing the action myself and watching what gets recorded. No
              code until I approve the taxonomy.

NON-GOALS:    Do not design A/B tests or consent flows; the next chapter
              covers those. Do not plan how to acquire users or build
              growth loops — that is a later module. Do not recommend an
              analytics vendor. Do not modify any files yet.

ACCEPTANCE:   - My definition of active is the action that is the point of
                my product, and I can defend rejecting the other two
              - Every event has a firing condition I could test by hand
              - My funnel has a stated window and counts people
              - My retention period matches how often a real user would
                return
              - For each metric I can name a decision it would change

ASK:          First ask me up to five questions about my product, my users
              and their natural frequency. Then give me the definition of
              active and the event list only, and stop before the funnel.
```

## Stuck? Ask AI
```prompt
Explain the difference between a vanity metric and an actionable metric using a college canteen as the example. Then give me ten numbers a student startup might put on a slide and ask me, one at a time, which kind each is and what decision it would or would not change. Do not tell me the answers first.
```

```prompt
Draw me, in plain text, three retention curves for the same product: one that flattens at a healthy level, one that keeps falling to zero, and one that looks bad only because the time period was chosen wrongly. Explain how I would tell the third apart from the second if I only had the chart.
```

```prompt
Here is my funnel: [paste your steps with counts]. Interrogate it before interpreting it. Are the steps genuinely ordered? Is it counting people or events? Is there a time window? Then tell me which drop-off is most likely to be a real problem rather than normal behaviour, and why the biggest drop might not be the answer.
```

## Quiz

### Q1 · L4
Your funnel shows a 39-point drop from menu to cart and an 18-point drop from reaching payment to placing an order. Which is more likely to be the real problem?
- (a) The 39-point drop, because it loses more people
- (b) The 18-point drop — those people had already decided to buy ✅
- (c) Neither; both are within normal ranges
- (d) Whichever happens earlier in the funnel
> Browsing a menu without ordering is ordinary behaviour. Abandoning at payment means intent existed and something broke it, which is far more likely to be a fixable defect than a change of mind.

### Q2 · L3
A retention curve flattens at 18% by week eight. What does the flat line tell you?
- (a) That 82% of users are dissatisfied and should be surveyed
- (b) That roughly 18% formed a habit — that share is your real product ✅
- (c) That the measurement period was too long
- (d) That churn is 18% per week
> The steep part is people trying and leaving; the flat part is people for whom the product became routine. A curve that never flattens means everyone eventually leaves, which is the distinction worth knowing early.

### Q3 · L4
AI builds you a funnel where the first step fires on page load. What is wrong with every number downstream?
- (a) Nothing; page load is the honest start of the funnel
- (b) Step one is inflated by people who never intended anything, so every rate below it is understated ✅
- (c) The events will be deduplicated and the step will be too low
- (d) Page-load events cannot carry properties
> Firing on load rather than on a deliberate action counts bots, accidental taps and bounced loads as participants. The conversion rates then look worse than reality, and the step you would optimise is the wrong one.

### Q4 · L5
Your analytics shows 1,180 orders last month; your database shows 1,274. What should you do?
- (a) Treat the database number as broken and investigate the gap
- (b) Expect the gap; use events for shapes and the database for exact counts ✅
- (c) Replace the analytics tool, since it is losing data
- (d) Fire the event from the server so both numbers always agree
> Blockers, refused consent and dropped 4G requests mean client events always undercount. Chasing an exact match wastes weeks; the discipline is knowing which number each question deserves.

### Q5 · L4
Which definition of "active" is most useful for a canteen pre-order app?
- (a) Opened the app at least once
- (b) Placed an order ✅
- (c) Viewed the menu
- (d) Has an account that has not been deleted
> Active should be the action that is the point of the product. The generous definitions make every chart look better and make none of them capable of changing a decision.

## Go deeper
- [PostHog docs: funnels](https://posthog.com/docs/product-analytics/funnels)
- [PostHog docs: retention](https://posthog.com/docs/product-analytics/retention)
