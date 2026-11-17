---
id: m01-05-scoping-an-mvp
module: m01-team-startup-git
slug: scoping-an-mvp
title: "Scoping an MVP You Can Ship, and Your North Star Metric"
order: 5
duration_min: 15
is_free: false
mission: false
volatility: low
last_reviewed: 2026-11-16
---

## Key learnings
- Cut an MVP along the user's journey rather than along the technology stack
- Explain why a walking skeleton beats a perfect single layer
- Choose a north star metric that cannot be gamed by doing nothing
- Tell vanity metrics from actionable ones
- Set a scope you can ship in two weeks and defend the cuts

## Lesson

### The pain

A team with a good PRD decides to build "the backend properly first", because that feels responsible. Three weeks later they have an excellent data model, seven endpoints, and nothing a human can look at — only half a product, perfectly. A second team spent four days on one ugly screen that puts one real order into a real database, and it has already told them three things about their users.

### The idea

**Slice the product along the user's journey, not along the layers of the stack. Build the thinnest complete path first.**

A thin slice through all four boxes — client, server, data, deployed — beats a perfect version of any one box, because only a complete path can be used by a human.

### The analogy

You are opening the canteen stall. **By layer:** build the entire kitchen. By the end of the week you can cook anything and have sold nothing, because there is no counter and no way to take money. **By slice:** make poha, write "Poha ₹20" on paper, take cash. By the end of day one you have revenue, you know whether anyone wants poha at that price, and you have found that your real problem is change for ₹500 notes.

**Where the analogy breaks:** a badly built kitchen can be rebuilt over a weekend. Software accumulates, so the tape must be *replaceable* — which is why the first slice still goes through git, still deploys properly, and still has a real database rather than fake data.

### Picture it

```
  SLICING BY LAYER — nothing usable until the end
  ┌────────────────────────────────────────────┐
  │ week 1  data model       ████████  perfect │
  │ week 2  API endpoints    ████████  perfect │
  │ week 3  screens          ░░░░░░░░  started │
  │ week 4  deploy           ░░░░░░░░          │  ← first user, week 4
  └────────────────────────────────────────────┘

  SLICING BY JOURNEY — usable on day four
  ┌───────────────────────────────────────────────┐
  │ slice 1  see menu → order → saved → confirm   │  ← first user, day 4
  │          ██ ugly but real, end to end         │
  │ slice 2  + log in as yourself                 │
  │ slice 3  + pay in test mode                   │
  │ slice 4  + staff marks sold out               │
  └───────────────────────────────────────────────┘
```

Each slice in the lower diagram touches every layer and ends in something a person can do. That first complete path is a **walking skeleton**: all the bones connected, no muscle yet. The test for slice one: **one real user completes the single most important action, and the result is really stored.** Everything else waits.

| Left out | Why it can wait |
|---|---|
| Login | Ask for a name in a text box. You learn nothing from auth in week one |
| Payments | Test mode in M04. Cash on collection works for slice one |
| Admin panel | Staff can look at the database table directly |
| Nice design | It must be legible, not beautiful. M03 is later |
| Error handling | Handle the one error you actually hit |
| Images | Text names. Photos are a slice of their own |

Three things are **not** optional: it goes in git, it deploys to a real URL, and the data is really saved.

### Where it lives

Checkpoint 2 in M11 is "walking skeleton, deployed". Slicing also shapes your briefs: "build the whole backend" is not checkable; "a student submits an order and I can see the row in the database" is.

The second output is your **north star metric** — one number that means the product works, because what you measure is what the team optimises. The test: could it rise while the product delivers no value?

| Vanity | Why it fails | Actionable instead |
|---|---|---|
| Signups | Twenty friends sign up once, out of politeness | Students who ordered twice |
| Page views | Rises when the page is confusing and people reload | Orders completed |
| Total orders ever | Only goes up; cannot get worse | Orders this week |
| App installs | Install then never open | Weekly active orderers |
| Time on site | Rises when people are lost | Time to complete an order, falling |

A good north star contains a **repeat**: one order proves curiosity, a second proves usefulness. M16 is where you instrument it.

### Words to own

| Term | Plain meaning | Example |
|---|---|---|
| MVP | Smallest version that delivers the core value | Not the smallest that compiles |
| Walking skeleton | A thin complete path through every layer | See menu, order, saved, confirmed |
| Vertical slice | A feature cut through all layers at once | One action, end to end |
| Horizontal slice | One whole layer at a time | "The backend first" |
| North star metric | The single number that means it is working | Weekly repeat orderers |
| Vanity metric | A number that rises without value | Signups, page views |
| Actionable metric | A number that tells you what to change | Drop-off at checkout |
| Scope | What is in this slice, and what is not | Slice one excludes login |
| Timebox | A fixed duration you will not extend | Two weeks |

### Tradeoffs and gotchas

**"Ugly but real" beats "beautiful but fake".** Hard-coded data teaches you about layout; a real database teaches you about people.

**Timebox the slice, then cut features to fit — never extend the box.** Scope is the only honest lever. Teams that extend a timebox extend it again and reach month three with nothing deployed.

**Beware "while we're in there".** The commonest way two weeks becomes five. Write the idea in your nice-to-haves; this is why NON-GOALS exists.

**Do not skip deployment in slice one.** Environment variables, build failures and the Linux case-sensitivity bug from M00 surface at deploy time. Week one is a bad afternoon; week eight is a bad week.

**One north star, not five.** A dashboard lets everyone pick the number that flatters them.

### Verify it

Five checks before you build.

1. **Does slice one touch all four boxes and end in something a person can do?** If it stops at an endpoint, it is a layer.
2. **Is the data really saved, in a real database, at a real URL?** Fake data makes a demo, not a skeleton.
3. **Can you list what you are deliberately leaving out?** If not, you have not scoped.
4. **Could your north star rise while the product is useless?** Then it is vanity. Add a repeat.
5. **Can you ship this in two weeks, around exams and three schedules?** If not, cut now.

Red flag: you cannot describe slice one as one action a person takes. Then it is still a list of technical tasks.

## Walkthrough
1. Open your PRD. Look only at MUST HAVE. Everything in this exercise comes from that list.
2. Write the single most important action a user takes — the one that, if it never happens, the product has failed. One verb. "Places an order."
3. Write the thinnest path to that action as numbered steps a person performs, not components you build. "Opens the page, sees four items, taps one, types their name, taps order, sees a confirmation."
4. For each step, note what has to exist in each of the four boxes. Keep it minimal — a text box is a login, a table is an admin panel.
5. Now write the exclusion list: everything from MUST HAVE that is *not* in slice one, with one reason each. This list should be longer than the slice.
6. Check the three non-negotiables: in git, deployed to a real URL, data really saved. If any is missing, add it to the slice.
7. Estimate honestly against two weeks, counting the days you will actually have after classes and the other modules. If it does not fit, remove a step from the path — not a day from the estimate.
8. Write your north star metric. Apply the test: could it rise while the product delivers nothing? If yes, add a repeat, a time window, or both.
9. Write two supporting numbers you will also watch, and label them clearly as supporting. This stops the north star from quietly becoming three.
10. Add a `SCOPE.md` to your repository: slice one as a user path, the exclusion list, the timebox with its end date, and the north star. Commit it.
11. Last, sanity-check the slice as a brief. Write the acceptance criteria for it. If you cannot write testable criteria, the slice is still too vague to hand to an agent.

## Talk to AI
```prompt
I need to cut a two-week first slice from my product, and I want you to argue for a smaller slice than I will propose.

My PRD MUST HAVE list: [paste it]
My SUCCESS criterion: [paste it]
The single most important user action: [one verb]
My proposed slice one: [list the steps]
Time available: two weeks, three people, alongside other coursework

Do this:
1. Tell me which steps of my proposed slice are not strictly needed for one real user to complete the core action. Be aggressive - assume I have over-scoped, because I have.
2. For each thing I am keeping, say which of the four boxes it touches (client, server, data, infrastructure). If any step touches only one box, tell me - that is a sign I am slicing by layer.
3. Name anything I have left out that is actually non-negotiable, specifically: is it in version control, is it deployed at a real URL, is the data really saved?
4. Write testable acceptance criteria for the slice you recommend - including at least one failure state.
5. Then review my north star metric: [paste it]. Could this number rise while my product delivers no value? If so, propose a better one and explain what makes yours harder to game.

Do not write code. I want the scope decided before anything gets built.
```

## Stuck? Ask AI
```prompt
Explain the difference between slicing vertically and horizontally using an example that is not software — building a house, opening a shop, anything physical. Then explain why horizontal slicing feels more responsible and is usually wrong.
```

```prompt
Here are five metrics I am considering as a north star: [list them]. For each one, tell me how I could make that number go up this week without making my product any more useful. Then rank them from most to least gameable.
```

```prompt
My team keeps adding things to our two-week slice because they seem quick. Give me three sentences I can actually say in the moment to stop this, and explain where the idea should go instead so the person feels heard.
```

## Quiz

### Q1 · L4
A team spends three weeks building a complete data model and seven endpoints before any screen exists. What is the main cost?
- (a) The endpoints will need rewriting when the design changes
- (b) They cannot test with a real user, so they learn nothing about whether the product is useful ✅
- (c) It uses more of the free tier than necessary
- (d) The data model will be wrong without the frontend to guide it
> Only a complete path can be used by a human. A perfect single layer produces no information about users, which is the thing a first slice exists to get.

### Q2 · L3
What is a walking skeleton?
- (a) A design mockup with no working code behind it
- (b) The backend finished before the frontend starts
- (c) A thin complete path through every layer, ending in something a person can do ✅
- (d) A prototype with hard-coded sample data
> All the bones connected, no muscle. Hard-coded data makes a demo instead — the data must really be saved for the skeleton to teach you anything.

### Q3 · L4
Which of these is the strongest north star metric for a canteen ordering product?
- (a) Total signups since launch
- (b) Page views per day
- (c) Students who placed a second order this week ✅
- (d) Total orders ever placed
> It contains a repeat and a time window, so it cannot be inflated by curiosity or politeness and it can go down. Totals only rise, and signups measure a one-time favour.

### Q4 · L4
You are behind on a two-week slice with three days left. What is the honest lever?
- (a) Extend the timebox by a week
- (b) Cut scope from the slice to fit the two weeks ✅
- (c) Reduce testing to save time
- (d) Add the remaining work to the next slice and skip deployment
> Scope is the only lever that does not lie. Teams that extend a timebox extend it again; skipping deployment defers the surprises to when they are far more expensive.

### Q5 · L4
Why must slice one be in version control and deployed to a real URL, even though it is "ugly but real"?
- (a) The platform requires a deployed URL for mission submission
- (b) Deployment is where the surprises live, and they are far cheaper to hit with fifty lines than with a real codebase ✅
- (c) It is impossible to save data without deploying
- (d) Agents cannot work on code that is not deployed
> Environment variables, build failures and the Linux case-sensitivity bug from M00 all surface at deploy time. Week one is a bad afternoon; week eight is a bad week.

## Go deeper
- [Alistair Cockburn — Walking Skeleton](http://wiki.c2.com/?WalkingSkeleton)
- [Lean Analytics — the One Metric That Matters](https://leananalyticsbook.com/one-metric-that-matters/)
