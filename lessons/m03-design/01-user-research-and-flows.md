---
id: m03-01-research-flows
module: m03-design
slug: user-research-and-flows
title: "User Research, User Flows and Information Architecture"
order: 1
duration_min: 25
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

A team of three builds a canteen pre-ordering app. They are sure students want to browse. So they ship a menu with cuisine filters, a veg/non-veg toggle, photo cards, a search bar, and a ratings system. Fourteen taps from opening the app to food being ordered. It looks like Swiggy.

Nobody uses it. They stand near the counter for twenty minutes and finally see why: a student has twenty-two minutes between lectures, is walking, holding a bag, and wants the same vada pav they had yesterday. They do not want to browse. They want "the usual, again, now" — which the app cannot do at all, because reordering was never a feature.

The team did not build it badly. They built the wrong thing beautifully. Three weeks gone. The information that would have saved them was available for free, twenty metres away, in a ten-minute conversation.

### The idea

**Research tells you what people are actually trying to do; a flow tells you the shortest honest path to doing it; information architecture decides where things live and what they are called.**

All three happen before any pixel is designed. They are cheap to change now and brutally expensive to change after the AI has generated forty components around the wrong assumption. In this module you never write the interface yourself — you brief it. A flow is what you brief with.

### The analogy

Walk into the campus canteen at 1 pm. The layout is a design decision someone made. Chai and snacks are on the right, near the door, because that is 70% of visits and those people must not block the meal queue. Plates are stacked before the food, not after. The price board is above the counter at eye height, not on a laminated card you have to pick up. The person who arranged that watched a thousand lunch rushes and arranged the room around what people repeatedly do.

That arrangement is information architecture. The path from "I'm hungry" to "I'm eating" — enter, grab plate, queue, say order, pay, collect, find seat — is a user flow.

**Where the analogy breaks:** in the canteen there is a human behind the counter who repairs every confusion instantly. Point at the wrong dish and someone says "that one's finished, try this". Your app has nobody. Every ambiguity you leave in the flow becomes a student standing still, and then a student who closes the app. Software has to carry its own signage.

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

One intent enters, three steps carry it to an outcome, and three named failure branches catch the cases where it cannot — each with a decided response rather than a blank screen.

### Three research methods that fit a two-hour session

You are not running a study. You need enough evidence to stop guessing, by Thursday.

**Five interviews, ten minutes each.** Not "would you use an app that…" — that question only measures politeness. Ask about the last time instead: "Tell me about the last time you bought food on campus. What time was it? How long did you have? What did you order? What annoyed you?" Past behaviour is data. Future intention is fiction.

**Intercept counting.** Stand where the thing happens and count. How many people are in the queue at 1:10 pm? How many pay cash versus UPI? How many are alone versus in a group? Twenty minutes of counting gives you numbers you can put in a brief, and a number in a brief changes what AI builds.

**Artefact hunting.** Look at what people already use to solve this. The canteen's WhatsApp group. The handwritten sold-out note stuck on the board. The teacher's Google Form. These are evidence of real demand, already working, and they tell you the vocabulary people already use.

Five interviews is enough because the fifth one usually tells you nothing new. If the fifth is still surprising you, do three more. This connects directly to M01, where your team gathered persona evidence for the PRD — this chapter is how you turn that evidence into something designable.

### Writing a flow AI can build from

A flow is not a wish list. It has a specific shape, and the shape is what makes it brief-able.

| Part | What it states | Canteen example |
|---|---|---|
| Actor | Who, specifically | Student with 22 minutes, walking |
| Trigger | What starts it | Hungry, between lectures |
| Steps | Numbered, each one screen or action | 3 steps |
| Outcome | The observable end state | Order placed, ETA on screen |
| Failure branches | What happens when a step cannot complete | Sold out, payment failed |
| Step budget | The maximum taps you will accept | 3 taps, under 20 seconds |

The step budget is the part everyone forgets and the part that disciplines everything else. "Reorder in three taps" is a constraint AI can design against. "Make reordering easy" is not.

A second flow worth naming early: the staff flow. Someone behind the counter has to mark the vada pav sold out, on a shared Android phone, with oily fingers, in four seconds. Two actors, two flows, one product. Teams routinely design the customer flow and discover the staff flow a week before demo day.

### Information architecture: naming is the product

Information architecture is two questions. What are the top-level places in this app? What is each one called?

Both are harder than they look, and AI gets both wrong in the same predictable way: it mirrors your database. Tables become tabs. You get "Items", "Transactions", "Entities" — accurate, and meaningless to a hungry nineteen-year-old.

Good names come from the research, in the users' own words. If every student said "chai" and nobody said "beverages", the section is called chai. If staff call it "sold out" and never "unavailable", the toggle says sold out. In an Indian app this matters twice over, because a word that is fine in English may be read by someone who thinks in Marathi or Hindi — "Outlet" is a shop to a designer and an electrical socket to everyone else. M15 takes this further into brand voice; here, just use the words you heard.

A practical test: write your four top-level labels on paper and show them to someone outside your team. Ask where they would tap to do three specific things. If they hesitate on more than one, the names are wrong, not the person.

### Where it lives

Research and flows sit at the very front of the build. In M01 you wrote the PRD; a flow is the PRD's "how" made concrete enough to design. In the next chapter these flows become wireframes, one frame per step. In chapter 4 you test them with three friends, which is how you find out your step budget was fantasy. In M07 the same flow becomes a funnel you measure — the steps you named here become the events you count there, which only works if you named them properly now. M11's MVP checkpoints will ask which flow your product is for, and "all of them" is the answer that fails.

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

**Research theatre is worse than no research.** Five interviews where you described your app and people said "nice" is not evidence — it is a trap, because now you believe you checked. If nobody in your interviews said anything that surprised or annoyed you, you were pitching, not researching.

**You are not your user, even when you are.** You are a student on this campus, so you are tempted to skip research entirely. But you are a student who is building an app, which makes you the single most atypical user available. Your own experience is a hypothesis, not a finding.

**Flows that branch forever never get built.** If your flow has eleven conditional branches, you have written a specification nobody can hold in their head, including the AI. Name the three or four failures that will actually happen, decide what each one shows, and let the rest fall into one honest error state.

**IA by org chart.** If your three top-level tabs happen to match your three team members' areas of responsibility, that is a smell. Users do not know how your team is split.

**A flow with no step budget will grow.** Every review meeting adds a step and nobody ever removes one. The budget is what lets you say "that is a fourth tap, so what comes out?"

### Verify it

When you hand AI a flow and ask for screens, flows or an IA, check five things before you accept anything.

1. **Count the steps against your budget.** If you said three taps and the output has six, it ignored your constraint. This is the single most common failure and the easiest to catch.
2. **Look for the failure branches by name.** Ask explicitly: "show me what the sold-out state and the payment-failed state look like." If AI only produced the happy path, you have a demo, not a product.
3. **Check for invented evidence.** AI will cheerfully write you a persona called "Priya, 21, loves trying new cuisines" that came from nothing. Every persona attribute must trace to something a real person said to you. Strike the rest.
4. **Read the labels aloud.** If the tabs are "Dashboard", "Items", "Manage" and "Settings", you got the database back. Replace every label with a word from your interview notes.
5. **Hunt the second actor.** Ask "which flows does this design not cover?" A product with customers almost always has a staff or admin flow hiding behind it.

Red flags: a flow with no numbers in it, a persona with no quote behind it, an IA where every section name is a noun from your schema, and any output that describes the happy path as though failure were impossible.

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
