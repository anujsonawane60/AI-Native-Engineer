---
id: m03-01-research-flows
module: m03-design
slug: user-research-and-flows
title: "User Research, User Flows and Information Architecture"
order: 1
duration_min: 15
is_free: false
mission: false
volatility: low
last_reviewed: 2026-11-16
---

## Key learnings
- Turn five short conversations into evidence you can design against
- Write a user flow as a numbered path from intent to outcome, with the failure branches named
- Lay out information architecture by what users look for, not by how your team is organised
- Spot the difference between a research finding and your own opinion wearing a lab coat
- Brief AI with a flow instead of a feature list, and check what it quietly skipped

## Lesson

### The pain

A team builds a canteen pre-ordering app: cuisine filters, a veg toggle, photo cards, search, ratings. Fourteen taps from opening to ordered. It looks like Swiggy, and nobody uses it. Twenty minutes near the counter shows why — a student has twenty-two minutes between lectures and wants "the usual, again, now", which the app cannot do at all because reordering was never a feature.

### The idea

**Research tells you what people are actually trying to do; a flow tells you the shortest honest path to doing it; information architecture decides where things live and what they are called.**

All three are cheap now and brutally expensive after AI has generated forty components around the wrong assumption. You brief the interface in this module rather than draw it, and a flow is what you brief with.

Three methods fit a two-hour session:

- **Five interviews, ten minutes each.** Ask about the last time — what time, how long did you have, what annoyed you. Never "would you use an app that…": past behaviour is data, future intention is fiction.
- **Intercept counting.** Queue length at 13:10, cash versus UPI, group size. A number in a brief changes what AI builds.
- **Artefact hunting.** The canteen's WhatsApp group, the sold-out note on the board — proof of demand, and the words people use.

### The analogy

Walk into the campus canteen at 1 pm. Chai sits near the door because that is 70% of visits and those people must not block the meal queue. Someone arranged the room around what people repeatedly do — that arrangement is information architecture, and the path from "I'm hungry" to "I'm eating" is a user flow.

**Where the analogy breaks:** the canteen has a human who repairs confusion instantly — "that one's finished, try this". Your app has nobody, so every ambiguity in the flow becomes a student standing still, then a student closing the app. Software carries its own signage.

### Picture it

```
  INTENT                    FLOW                        OUTCOME
┌──────────┐    ┌────────────────────────────┐    ┌─────────────┐
│ "the     │───▶│ 1 open ──▶ 2 tap "again"   │───▶│ order       │
│  usual,  │    │             │               │    │ placed,     │
│  again"  │    │             ▼               │    │ ETA shown   │
└──────────┘    │        3 confirm & pay      │    └─────────────┘
                └────────────┬───────────────┘
                             │
          ┌──────────────────┼──────────────────┐
          ▼                  ▼                  ▼
    no order yet       dish sold out        payment failed
    (show menu)        (suggest nearest)    (retry, keep cart)
```

One intent enters, three steps carry it to an outcome, and three named failure branches each get a decided response rather than a blank screen.

That fixed shape is what makes a flow brief-able:

| Part | What it states | Canteen example |
|---|---|---|
| Actor | Who, specifically | Student with 22 minutes, walking |
| Trigger | What starts it | Hungry, between lectures |
| Steps | Numbered, each one screen or action | 3 steps |
| Outcome | The observable end state | Order placed, ETA on screen |
| Failure branches | What happens when a step cannot complete | Sold out, payment failed |
| Step budget | The maximum taps you will accept | 3 taps, under 20 seconds |

The step budget disciplines everything else: "reorder in three taps" is a constraint AI can design against, "make reordering easy" is not.

### Where it lives

Research and flows sit at the very front of the build.

- **M01's PRD** — a flow is its "how", concrete enough to design
- **Next chapter** — each flow step becomes one wireframe
- **Chapter 4** — testing shows your step budget was fantasy
- **M07** — the steps you name here become the funnel events you count
- **M11** — MVP checkpoints ask which flow this is for; "all of them" fails

### Words to own

| Term | Plain meaning | Example |
|---|---|---|
| User research | Finding out what people actually do, with evidence | Five ten-minute interviews |
| Persona | A named, evidence-backed user type | "Rushed student, 22 minutes" |
| Job to be done | The outcome the user wants, not the feature | "Eat before the next class" |
| User flow | A numbered path from intent to outcome | Open, reorder, pay |
| Happy path | The flow when everything works | Dish available, payment clears |
| Edge case | A real situation outside the happy path | Dish sold out mid-payment |
| Information architecture | Where things live and what they are called | Four tabs: Today, Again, Orders, Me |
| Step budget | The maximum taps you will accept | Reorder in 3 taps |
| Card sort | Asking users to group and name things themselves | Students sorting 20 dish names |
| Leading question | A question that smuggles in its own answer | "Wouldn't an app be better?" |

### Tradeoffs and gotchas

**Research theatre is worse than no research.** Five interviews where you pitched and people said "nice" is not evidence, and now you think you checked.

**You are not your user, even when you are.** A student building an app is the most atypical student available. Your experience is a hypothesis.

**Flows that branch forever never get built.** Name three or four real failures; let the rest fall into one honest error state.

**The second actor is a whole flow.** Someone must mark the vada pav sold out in four seconds. Teams find that flow a week before demo day.

**AI gives you your database back.** Asked for an IA, it mirrors your schema: "Items", "Transactions", "Manage" — accurate, meaningless to a hungry nineteen-year-old. Names come from research: if students said "chai", the section is chai. Tabs matching your three team members' areas are the same smell.

### Verify it

Run five checks on any flow, screen set or IA the AI hands you.

1. **Count the steps against your budget** — three asked, six delivered means the constraint was ignored.
2. **Ask for failure branches by name**: "show me the sold-out state and the payment-failed state."
3. **Strike invented evidence** — every persona line must trace to something a real person said.
4. **Read the labels aloud**, then ask an outsider where they would tap to do three tasks. Hesitation means the name is wrong, not the person.
5. **Hunt the second actor**: "which flows does this design not cover?"

Red flags: a flow with no numbers in it; a persona with no quote behind it; section names that are nouns from your schema; output that treats failure as impossible.

## Walkthrough
1. Pick one job your product does and write it as a sentence in the user's voice, starting with "I want to". Not a feature — an outcome. "I want to eat before my 1:40 class" rather than "I want a menu filter".
2. Go where that job actually happens and talk to five people for ten minutes each. Ask only about the last time they did it. Write down exact phrases, not summaries.
3. Spend twenty minutes counting something real: queue length at peak, cash versus UPI, group size, time from arrival to leaving. Write the numbers down with the time of day.
4. Collect two artefacts people already use for this job — a WhatsApp group, a paper notice, a spreadsheet. Photograph them.
5. From that evidence, write one persona of four lines, where every line is backed by something you heard or counted. Put the quote next to the line.
6. Write the primary flow: actor, trigger, numbered steps, outcome, step budget. Keep it under six steps. If it will not fit, you have two flows.
7. Name three failure branches that will genuinely happen and decide what the user sees in each. Write them as "when X, show Y".
8. Write the second actor's flow — staff, admin, whoever has to keep the data true. One actor, trigger, steps, outcome.
9. Draft four top-level section names using only words from your notes. Show them to someone outside your team and ask where they would tap to do three tasks. Record every hesitation.
10. Give AI your flow and ask it to list what it would need to invent in order to build it. Everything on that list is a gap in your research — go and close the two biggest.

## Talk to AI
```prompt
CONTEXT:      I am a student founder in a team of three. We are designing,
              not yet building. Our users are on budget Android phones in
              India, often on slow 4G, often outdoors in bright sunlight.
              Many of them think in Hindi or Marathi and read English
              slowly.

              Here is the raw evidence I actually collected:
              - 5 interviews, key quotes: [paste 5-8 real quotes verbatim]
              - Things I counted: [e.g. 23 people in queue at 13:10;
                17 of 20 paid by UPI; 12 of 20 were in groups]
              - Artefacts already in use: [e.g. a WhatsApp group of 180
                students where the canteen posts the day's menu]

GOAL:         Turn this evidence into one primary user flow, one secondary
              actor flow, and a four-item information architecture I can
              wireframe next.

REQUIREMENTS: 1. One persona, max 4 lines, and every line must cite which
                 quote or number it came from
              2. The primary flow as: actor, trigger, numbered steps,
                 outcome, step budget
              3. Exactly three named failure branches, each written as
                 "when X, show Y"
              4. A second flow for whoever keeps the data accurate
              5. Four top-level section labels using ONLY words that
                 appear in my evidence above

CONSTRAINTS:  Primary flow must fit in 3 taps and 20 seconds. Labels must
              be words a first-year student would use out loud. No feature
              I did not ask for. Free tiers only, so assume no paid
              service anywhere in the flow.

NON-GOALS:    No wireframes. No colours, fonts or visual design. No tech
              stack choices. Do not invent any user attribute, quote or
              statistic that is not in my evidence.

ACCEPTANCE:   - Every persona line names its source
              - The primary flow has a number of steps I can count, and
                it is 3 or fewer
              - Each failure branch says exactly what the user sees
              - No section label is a database-style word like "Items",
                "Entities", "Manage" or "Dashboard"
              - A separate list at the end titled "What I had to assume",
                naming every gap you filled in yourself

ASK:          First give me TWO different information architectures from
              the same evidence, with the tradeoff of each, and say which
              you would pick and why. Only after I choose, write the flows.
```

## Stuck? Ask AI
```prompt
Here are five questions I plan to ask students about buying food on campus: [paste your questions]. For each one, tell me whether it measures real past behaviour or just politeness, and rewrite the weak ones so they ask about the last actual time it happened. Do not add new questions — only judge and rewrite mine.
```

```prompt
Here is my user flow: [paste it]. Act as a hostile reviewer who has watched real users fail. List every point where a user could get stuck, confused or stop, and for each one tell me whether it is a real risk or a rare one. Do not redesign the flow. Just find the holes and rank them.
```

```prompt
I have these four top-level section names for my app: [paste them]. Give me eight specific tasks a first-time user might want to do, and for each tell me which section they would probably tap and how confident you are. Where two sections are equally plausible, say so — that is the name I need to fix.
```

## Quiz

### Q1 · L3
During interviews for a campus food app, which question produces usable evidence?
- (a) Would you use an app that lets you pre-order food?
- (b) Tell me about the last time you bought food on campus — what time, how long did you have, what did you order? ✅
- (c) Do you think students need a faster way to order?
- (d) Which of these three designs do you prefer?
> Past behaviour is data; stated future intention mostly measures politeness. The last-time question produces times, constraints and annoyances you can design against.

### Q2 · L4
Your flow says "reorder in 3 taps". The AI returns a design with a menu screen, a filter, a dish page, a cart and a checkout. What have you learned?
- (a) The AI improved on your flow by adding useful features
- (b) Your step budget was ignored, so the constraint needs restating and the output rejected ✅
- (c) Reordering is technically impossible in three taps
- (d) Nothing — step counts are not a real design concern
> A step budget is a testable constraint. Output that silently exceeds it is a scope failure, and counting the steps catches it in seconds without reading anything else.

### Q3 · L2
What is information architecture?
- (a) The database tables behind the app
- (b) Where things live in the app and what each place is called ✅
- (c) The colour and typography system
- (d) The order in which pages load
> IA answers two questions: what are the top-level places, and what is each one named. Mirroring your database tables is the classic way to get it wrong.

### Q4 · L4
AI gives you a persona: "Priya, 21, adventurous eater who loves discovering new cuisines." Your interview notes never mention variety. What should you do?
- (a) Keep it — personas are meant to be aspirational
- (b) Strike the unsupported attributes, because an invented persona will justify features nobody asked for ✅
- (c) Rename her and keep the attributes
- (d) Run five more interviews until someone mentions variety
> An invented attribute becomes an invented requirement: "loves discovering" quietly justifies browse-and-filter, which is exactly the feature that failed. Every line must trace to evidence.

### Q5 · L5
A team designs a canteen ordering flow that works perfectly in testing, then discovers a week before demo day that nobody can mark a dish sold out. Which step of this chapter would have prevented it?
- (a) Writing a tighter step budget for the customer flow
- (b) Naming the second actor and writing their flow alongside the customer's ✅
- (c) Running more customer interviews
- (d) Choosing better section labels
> Products with customers almost always have a staff or admin actor keeping the data true. Asking "which flows does this design not cover?" surfaces them at design time rather than at demo time.

## Go deeper
- [Nielsen Norman Group: Why you only need to test with 5 users](https://www.nngroup.com/articles/why-you-only-need-to-test-with-5-users/)
- [Nielsen Norman Group: Information architecture — study guide](https://www.nngroup.com/articles/ia-study-guide/)
