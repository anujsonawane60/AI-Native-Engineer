---
id: m01-01-group-of-three
module: m01-team-startup-git
slug: group-of-three
title: "A Group of Three: Splitting Product, Tech and Growth"
order: 1
duration_min: 25
is_free: false
mission: false
volatility: low
last_reviewed: 2026-11-16
---

## Key learnings
- Split a three-person team by decision domain rather than by task
- Name the one person accountable for each kind of decision
- Keep every member directing AI and shipping their own evidence
- Spot the two ways a group of three fails, early enough to fix it
- Work the same way if you are temporarily on your own

## Lesson

### The pain

Three students form a group. All three are interested in the technology, because that is what the program is about. So all three build.

By week two they have three half-finished versions of the same login screen, no idea who their user is, and nobody has spoken to a single person outside the group. By week four one of them — the fastest — has quietly taken over everything, and the other two are watching. At demo day one person can explain the product and two cannot.

Both of those failures are structural, not personal. They come from dividing work by *task* instead of by *decision*, and they are entirely avoidable with one conversation in week one.

### The idea

**Each of the three owns a domain of decisions, not a pile of tasks. Everyone still builds.**

The split is about who has the final word when there is disagreement and no time to debate. It is not a split of the work, and it is certainly not a split of the learning — all three of you walk the same 200-plus concept map and all three direct AI daily.

### The analogy

A three-person canteen stall.

**Product** decides what is on the menu. Which dishes, at what price, for whom. They are the one who notices that students want a ₹30 option and that nobody is buying the ₹90 thali. They do not own the kitchen, but when there is an argument about whether to add a new dish, they decide.

**Tech** decides how it gets made. The equipment, the process, what can be prepped in advance, what breaks at 1:10 pm. When there is an argument about whether something is feasible by Friday, they decide.

**Growth** decides how anyone finds out it exists. The board outside, the WhatsApp group, the deal with the hostel. When there is an argument about where to spend the week's attention, they decide.

All three cook during lunch rush. The division is of judgement, not of labour.

**Where the analogy breaks:** a canteen's menu is fairly stable, so Product is a part-time job. In a startup the product changes weekly because you keep learning your users were different from what you assumed. Product is the busiest of the three in week one and stays busy.

### Picture it

```
            ┌──────────── PRODUCT ────────────┐
            │  who is this for               │
            │  what are we building          │
            │  what are we NOT building      │
            └───────────────┬────────────────┘
                            │
         ┌──────────────────┴──────────────────┐
         ▼                                     ▼
┌─────────────────┐                  ┌──────────────────┐
│      TECH       │                  │      GROWTH      │
│  how it works   │◀── feasibility ──│  how people      │
│  what it costs  │     vs demand    │  find it         │
│  when it ships  │                  │  what they do    │
└─────────────────┘                  └──────────────────┘
         │                                     │
         └──────────── all three ──────────────┘
                 direct AI · write briefs
                 review diffs · post daily
```

Three decision domains at the top, one shared practice at the bottom. The arrows between Tech and Growth matter: most real arguments are "is this possible in time" against "do users need it now", and Product breaks the tie.

### Where it lives

In M00 you learned the four technical responsibilities — client, server, data, infrastructure. Those do not map onto these three people, and trying to force it is a mistake worth naming.

Tech owns the *decisions* about all four boxes. But a schema decision with pricing implications is a Product decision wearing a technical costume, and "should we store the user's phone number" is a Growth and privacy decision before it is a database one. The boxes describe the system; these roles describe who decides.

Practically: **Tech does not get to pick the data model alone**, because the data model encodes what the product believes about its users. You will feel this in M04 when you design your schema.

### Everyone still directs AI

This is the part that makes the program work, and the part most likely to quietly decay.

Each of you writes your own Director's Briefs. Each of you reviews diffs. Each of you submits your own mission evidence — and the platform enforces this, because a mission is completed by *your* submission under *your* account, not your group's. Three people on one product still means three sets of artefacts.

If Growth has not written a brief in two weeks, something has gone wrong even if the product is going well. They will reach demo day unable to answer a question about their own system, and in M18 the defence is individual.

### Words to own

| Term | Plain meaning | Example |
|---|---|---|
| Decision domain | The kinds of calls one person has final say on | Product decides scope |
| DRI | Directly Responsible Individual; the single accountable person | "Who is the DRI for payments?" |
| Working agreement | What the team has agreed about how it works | "We review PRs within a day" |
| Standup | A short daily sync on progress and blockers | The first 10 minutes of the live session |
| Blocker | Something stopping someone from progressing | "Waiting on the API contract" |
| Pairing | Two people working on one thing together | Debugging a failing deploy |
| Bus factor | How many people must vanish before the project stops | A bus factor of 1 is fragile |
| Scope creep | Work expanding beyond what was agreed | "While we're here, let's also…" |

### Tradeoffs and gotchas

**A role is not a silo.** "That's Tech's problem" is how a group of three becomes three people working alone in the same room. The owner decides; everybody contributes and everybody must understand. If only one of you can explain how authentication works, your bus factor is one and your demo day is going to be uncomfortable.

**The fastest person is the biggest risk.** It feels efficient to let them do everything, and it produces exactly the failure described at the top. If you are that person, your job is to hand work over and review it, which is slower and is the actual skill being trained. If you are not, you have to ask for work rather than wait for it.

**Rotate once, deliberately.** Around the halfway point, swap one role. It is briefly inefficient and it is the cheapest way to discover what each role actually involves — and whether your documentation is good enough for someone else to pick up.

**Decide the tie-break rule before you need it.** Not "we'll discuss it" — you will be discussing it at 11 pm with a deadline. Write down: Product decides scope, Tech decides feasibility and sequencing, Growth decides channel priority, and anything touching money or user data needs all three.

**Write the agreement down in week one.** Six lines is enough: who owns what, how fast we review each other's work, when we meet, where decisions get recorded, what we do when someone disappears for exams, how we handle a disagreement. Unwritten agreements are the ones that get violated.

**If you are temporarily solo**, nothing changes except that you hold all three domains and must schedule them. The real risk is that Product and Growth silently lose to Tech, because building is more fun than talking to users. Block time for them explicitly. Everything in this program is completable alone; it is just easier to fool yourself.

### Verify it

Four checks you can run on your own group at the end of any week. They take five minutes and they catch decay early.

1. **Can each of you name your own domain and give one decision you made in it this week?** If someone cannot, they are doing tasks, not owning a domain.
2. **Has every member written a brief and reviewed someone else's work this week?** This is the individual-evidence check, and it is the one that fails first.
3. **Can the two non-Tech members each explain one part of the architecture without help?** Pick a different part each week. If the answer is consistently no, you have a bus factor problem that demo day will expose.
4. **Is there anything nobody owns?** Infrastructure and data are the usual orphans, exactly as in M00. Unowned work does not get done; it gets discovered.

## Walkthrough
1. Get the three of you in one room, or one call, with something to write on that everyone can see afterwards.
2. Each person writes down, privately, which of the three roles they want and which they would be worst at. Compare. Overlaps are normal — resolve them by who is more willing to do the parts they dislike, not by who is keenest.
3. Assign the three domains. Write the names down. If you are two people, one takes Product and Growth; if you are alone, you hold all three and schedule them.
4. For each domain, list three concrete decisions that domain will own in the next two weeks. "Product decides whether we support UPI at launch", not "Product does product things."
5. Write the tie-break rule in one sentence per domain, plus the rule for decisions that need all three.
6. Agree four numbers: how often you meet, how fast you review each other's work, how long a blocker may sit before being raised, and what the minimum is that everyone ships each week.
7. Agree where decisions are recorded. One shared document. Nominate who writes in it — recording it is a job, and if it is nobody's job it stops happening by week three.
8. Agree what happens during exam weeks, because this will happen and the answer "we'll figure it out" means one person carries it.
9. Put the whole thing in a single file in your repository, called `TEAM.md`. It is now versioned, which means changes to it are visible — and it is the first thing you will commit in chapter 6.
10. Read it back out loud. Anything that makes someone wince is the thing to fix now rather than in week five.

## Talk to AI
```prompt
I am forming a three-person team for a six-month internship where we build a startup together. We will split into Product, Tech and Growth.

About us:
- Person A: [interests, strengths, what they want to learn]
- Person B: [same]
- Person C: [same]
- Our rough product idea: [one or two sentences, or "undecided"]

Help me think this through rather than just assigning roles:
1. Suggest a role split, and explain the reasoning for each person
2. For each role, list five concrete decisions that role should own in our first month
3. Name the three decisions that should require all three of us to agree, and why
4. Based on what I told you, predict the most likely way our specific team fails, and the earliest warning sign I would see
5. Give me six lines for a working agreement, written as rules we could actually violate and notice

Then ask me the three questions you would most want answered to improve this split. Do not write any code.
```

## Stuck? Ask AI
```prompt
I am in a team of three and one person is doing most of the building while the other two watch. Explain why this happens even when nobody intends it, and give me three concrete things to change this week. Then ask me which of the three I am least likely to actually do, and why.
```

```prompt
Our team keeps arguing about whether a feature is worth building. Explain the difference between a product decision, a technical decision and a growth decision using one example feature, then help me work out who should have the final say on the thing we are currently arguing about: [describe it].
```

```prompt
I am doing this program alone for now. Help me build a weekly schedule that forces me to spend real time on product and growth rather than only building, and tell me what I will most likely skip and how to notice when I have.
```

## Quiz

### Q1 · L2
In a group of three, what is actually being divided between Product, Tech and Growth?
- (a) The tasks, so each person builds a different part
- (b) The decisions each person has final say on ✅
- (c) The modules of the course each person studies
- (d) The AI tools each person is allowed to use
> All three build, all three direct AI, and all three walk the whole concept map. The split exists so that disagreements resolve quickly without debate.

### Q2 · L4
By week four, one member has taken over almost all the building and the other two mostly watch. Why is this a serious problem rather than efficient?
- (a) It is against the program rules
- (b) It slows the product down compared to three people building
- (c) Two members reach an individual technical defence unable to explain their own system ✅
- (d) The AI produces worse output for a single user
> It often does ship faster in the short term. The cost is a bus factor of one and two people who cannot answer for the product — and in M18 the defence is individual.

### Q3 · L4
Your team is deciding how to store users' phone numbers. Who should decide?
- (a) Tech alone, because it is a database question
- (b) Product alone, because it concerns users
- (c) All three, because it touches user data ✅
- (d) Growth alone, because phone numbers are used for marketing
> It looks like a schema decision and is really a privacy, product and growth decision at once. Anything touching money or user data is the standard all-three case.

### Q4 · L3
Which pair of responsibilities is most often left unowned in a three-person team?
- (a) Frontend and design
- (b) Infrastructure and data ✅
- (c) Product and growth
- (d) Testing and code review
> The same orphans as in M00: nothing visibly breaks on day one, so backups, monitoring and cost get discovered rather than decided.

### Q5 · L3
Why should the team's working agreement be written into a file in the repository rather than agreed verbally?
- (a) It is required before the repository can be created
- (b) Version control makes changes to it visible, and unwritten agreements are the ones that get violated ✅
- (c) AI agents read it automatically and enforce it
- (d) It counts as a mission submission
> Putting it under version control means a quiet change is a visible change. The point is accountability, not ceremony.

## Go deeper
- [Shape Up — chapter on teams and scopes](https://basecamp.com/shapeup)
- [Google re:Work — what makes a team effective](https://rework.withgoogle.com/print/guides/5721312655835136/)
