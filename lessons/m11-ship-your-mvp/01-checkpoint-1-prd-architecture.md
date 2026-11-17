---
id: m11-01-checkpoint-1
module: m11-ship-your-mvp
slug: checkpoint-1-prd-architecture
title: "Checkpoint 1 · Problem, PRD and Architecture Diagram"
order: 1
duration_min: 25
is_free: false
mission: true
volatility: low
last_reviewed: 2026-11-16
---

## Key learnings
- Pass the week-one gate with evidence rather than intentions
- Draw an architecture diagram of your own product that a stranger can follow
- Submit your own evidence even though the product is shared by three people
- Tell a checkpoint artefact from a conversation your team had
- Fail yourself against a self-check list before anyone else does

## Lesson

### Where this sits in the calendar

**Checkpoint 1 is due at the end of week one of the Sprint.** Not week three, not at the end. This module is read across the whole 30–45 days: Checkpoint 1 in week one, Checkpoint 2 in week two, Checkpoint 3 in week three, Checkpoint 4 in the final week. The chapter list puts M11 last because the platform orders modules in one line, but the live-session calendar sequences it from day one, and your group runs its own company from the first week.

### The pain

Two groups reach day seven. The first has had excellent conversations and written nothing down; by week three their agent has been briefed three ways by three people and they are arguing about what they agreed, with nothing to point at. The second has four committed files — `PROBLEM.md`, `PRD.md`, `SCOPE.md`, `DECISIONS.md` — and a diagram of their own system. They had worse ideas, and they will finish.

### The idea

**A checkpoint is three things: a date, a list of artefacts that must exist, and a check you run on yourself.**

There is no grader here, so the self-check list is the only thing holding the bar. "We have a PRD" is not checkable; "our PRD has a SUCCESS line with a number and a date" is.

### The analogy

A canteen stall cannot open on a good idea about food. Before a plate is sold there is an inspection: a licence on the wall, a price list the customer can read, a layout the inspector walks through. Your PRD is the price list, your diagram the layout, your repository the licence.

**Where the analogy breaks:** nobody will shut your project down on day seven, so the gate only works if you run it on yourself honestly.

### Picture it

```
  WEEK 1        WEEK 2         WEEK 3        FINAL WEEK
┌──────────┐  ┌──────────┐  ┌──────────┐  ┌──────────────┐
│ CP1      │  │ CP2      │  │ CP3      │  │ CP4          │
│ problem  │─▶│ walking  │─▶│ auth +   │─▶│ 10 users,    │
│ PRD      │  │ skeleton │  │ data +   │  │ eval set,    │
│ diagram  │  │ deployed │  │ AI + CI  │  │ decisions,   │
│ repo     │  │          │  │          │  │ tech demo    │
└──────────┘  └──────────┘  └──────────┘  └──────────────┘
   M01          M01–M06       M04–M10      M07, M10, M16
```

The four gates sit on the Sprint calendar one week apart, and the row underneath names the modules whose missions actually build the thing each gate checks.

```
  ┌──────────┐   ┌───────────────┐   ┌────────────────┐
  │ browser  │──▶│ your app      │──▶│ database       │
  │ on a     │   │ (server code) │   │ (your tables)  │
  │ phone    │◀──│               │◀──│                │
  └──────────┘   └──────┬────────┘   └────────────────┘
                        │
                        ▼
                 ┌──────────────┐
                 │ model API    │
                 │ (AI feature) │
                 └──────────────┘
```

A phone talks to your server, the server is the only thing that talks to your database and to the model API, and nothing on the phone reaches either one directly.

### What must exist by the end of week one

Six artefacts, committed, so they carry a date.

**1. `PROBLEM.md`** — the problem in one or two sentences with no solution in it, plus notes from five real conversations. M01.2–M01.3.

**2. `PRD.md`** — the one-page PRD, all seven sections: PROBLEM, USER, GOAL, MUST HAVE, CONSTRAINTS, NON-GOALS, SUCCESS. M01.4.

**3. `SCOPE.md`** — slice one as a user path, the exclusion list, the timebox with its end date, the north star metric. M01.5.

**4. An architecture diagram** — committed ASCII or a whiteboard photo, every arrow labelled with what travels. M00.1, M00.3.

**5. `TEAM.md`** — who holds product, tech and growth, plus your working agreements. M01.1, M01.9.

**6. `DECISIONS.md`, two dated entries** — what you chose, what you rejected, why. By Checkpoint 4 it needs eight entries spread across the weeks.

### Where it lives

Checkpoint 1 gates M01 from everything after it:

- M03 turns your USER section into design constraints
- M04 turns MUST HAVE into a schema
- M05 takes your SUCCESS numbers as the load to plan for
- M10 turns your PRD and constraints into a rules file the agent reads

**Each of you submits your own evidence.** The product is shared; the completion record is not — three people on one repository means three submissions, each with that person's own reflection.

### Words to own

| Term | Plain meaning | Example |
|---|---|---|
| Checkpoint | A date, an artefact list, and a self-check | Checkpoint 1, end of week one |
| Artefact | A file that exists, not a discussion | `PRD.md`, committed |
| Evidence | Proof a stranger can check without you | A screenshot, a URL, a commit date |
| Architecture diagram | Every box and labelled arrow in your system | The picture above, with your names |
| Decision log | Dated record of choices and reasons | `DECISIONS.md` |
| Gate | A point you do not pass without the artefacts | Not a grade; a checklist |
| Self-check | The list you fail yourself against | Ticked before submitting |
| Working agreement | How the three of you operate | "PRs reviewed within one day" |

### Tradeoffs and gotchas

**A diagram drawn after the code is documentation; drawn before it, it is a decision.** Week one is the only time it is cheap — it is how you discover two of you assumed different databases.

**Do not let AI write your PRD from your idea.** It will produce a plausible average of every similar product, built for nobody you interviewed — the generic-output failure from M00.7.

**Two entries, on two different days.** Eight written in one sitting in the final week are visible from a distance: all the same length, all explained with hindsight.

### Verify it

Be unkind.

1. **Hand `PRD.md` to someone outside your team and ask what you are building and who for.** If they get it wrong, your AI will too.
2. **Search your PRD for fast, simple, easy, secure, intuitive and seamless.** Each is an adjective doing a fact's job; replace it with a number.
3. **Point at every arrow and say what travels along it.** One you cannot label is an undecided design question.
4. **Check SUCCESS has a number and a date.**
5. **Open your commit history.** Several days, or all at 11 pm on day seven?

The red flag here: **a product that exists only in one person's head.** If one member is the only one who can explain the architecture, two students will not pass M18's individual defence.

## Walkthrough
1. Open a shared document and put the end of week one on it as a real calendar date. Everything below is due then, not "soon".
2. Collect what already exists. Most groups have fragments of all six artefacts scattered across chat messages and notebooks. List what is written down and what is only remembered.
3. Fix `PROBLEM.md` first. One or two sentences, no solution inside them, plus the notes from your five real conversations with exact quotes where you have them.
4. Finish `PRD.md` from M01.4, writing SUCCESS before MUST HAVE. Count your must-haves; more than six means you have not scoped yet.
5. Finish `SCOPE.md` from M01.5. Slice one as a user path in plain verbs, the exclusion list longer than the slice, the timebox with its end date, and one north star metric.
6. Draw the architecture diagram as ASCII, by hand, before you ask any AI about it. Draw only the boxes you are confident about, then list the ones you are unsure of underneath as open questions.
7. Use the prompt below to have AI critique the diagram and the PRD together. Fix what you agree with; write down what you rejected and why. That disagreement is a decision-log entry.
8. Write `TEAM.md`: the three roles from M01.1, your daily session time, and who reviews whose pull requests.
9. Start `DECISIONS.md` with two dated entries. Good first candidates: why this problem and not the other three, and which approach you rejected from step seven.
10. Each of you, separately and from memory, redraw the architecture diagram on paper. Compare the three. Every difference is something your team has not actually agreed.
11. Run the self-check list below individually. Each member then submits their own post with their own reflection.

## Talk to AI
```prompt
I am at the end of week one of a startup sprint and I have to pass a checkpoint. I want you to inspect my artefacts as a hostile reviewer, not improve them for me. Do not write code and do not rewrite my documents.

Here is my problem statement: [paste PROBLEM.md]
Here is my one-page PRD: [paste PRD.md]
Here is my slice-one scope and north star metric: [paste SCOPE.md]
Here is my architecture diagram, in ASCII: [paste it]
Here are my first two decision-log entries: [paste them]

Work through these in order, quoting my own words back to me:

1. PROBLEM. Is there a solution hiding inside my problem statement? Quote it. Could this statement describe twenty other products?
2. EVIDENCE. Which lines of my USER section sound like imagination rather than something a real person said? Name each one.
3. ADJECTIVES. Quote every instance of fast, simple, easy, secure, intuitive, mobile-friendly or seamless, and tell me what number or testable statement should replace each.
4. SCOPE. Does my slice one touch the client, the server, the data and a deployed URL? If any step touches only one of those, say so - that means I am slicing by layer rather than by user journey.
5. DIAGRAM. Which arrows on my diagram are unlabelled or labelled with a verb instead of the thing that travels? Which boxes in my MUST HAVE list are missing from the diagram entirely?
6. CONTRADICTIONS. Anything in MUST HAVE that conflicts with CONSTRAINTS, NON-GOALS or my slice-one scope.
7. SUCCESS. Does it have a number and a date? Could that number rise while my product delivers no value?

Then describe my product back to me in three sentences using only what I pasted. If your description is not what I meant, tell me exactly which sentence of mine misled you.

Finally: name the one thing a reviewer would most likely catch that I have not asked you about.
```

## Mission

**Objective.** Pass the week-one gate: six committed artefacts, an architecture diagram of your own product that a stranger can follow, and your own individual evidence pack submitted.

**Steps.**
1. **Date it** — put the real end-of-week-one date on the checkpoint. Everything else is scheduled backwards from it.
2. **Audit** — list which of the six artefacts exist as files and which exist only as conversations.
3. **Write** — finish `PROBLEM.md`, `PRD.md` and `SCOPE.md` using M01.2–M01.5. SUCCESS before MUST HAVE.
4. **Draw** — the architecture diagram, by hand first, with every arrow labelled with what travels.
5. **Critique** — run the prompt above. Fix what you agree with; record one disagreement and why.
6. **Agree** — `TEAM.md` with roles and working agreements. `DECISIONS.md` with two dated entries.
7. **Cross-check** — each member redraws the diagram from memory alone; compare the three and resolve every difference.
8. **Outside reader** — one person who is not in your group describes your product back to you. Fix whatever they got wrong.
9. **Submit individually** — each of the three of you posts your own evidence with your own reflection.

**Upload.**
- All six artefacts → attached files, or the repository URL in `repo_url`
- Your architecture diagram → the ASCII block or a photo of the whiteboard
- A screenshot of your commit history showing these files were committed across more than one day
- The AI critique conversation → share link in `ai_chat_url`
- One thing the AI flagged that you disagreed with, and why, in two sentences
- What your outside reader said the product was, before you corrected anything
- The three from-memory diagrams and a note on what differed between them
- Reflection, 150 words max → what you personally wrote or decided, and one thing your group discovered it had not agreed

**Self-check.** Tick every box before you submit.
- [ ] My `PROBLEM.md` states a problem with no solution inside it
- [ ] At least five real conversations are recorded, with quotes
- [ ] `PRD.md` has all seven sections and fits on one page
- [ ] No adjective is doing a fact's job anywhere in the PRD
- [ ] Six or fewer must-haves, and removing any one breaks the product
- [ ] NON-GOALS has more lines than MUST HAVE, including one "do not touch"
- [ ] SUCCESS has a number and a date and is not a signup count
- [ ] `SCOPE.md` states slice one as a user path, with an exclusion list
- [ ] Every arrow on my architecture diagram is labelled with what travels
- [ ] Every box in MUST HAVE appears somewhere on the diagram
- [ ] `TEAM.md` names who holds product, tech and growth
- [ ] `DECISIONS.md` has two entries, dated on two different days
- [ ] An outside reader described our product correctly
- [ ] I can redraw the architecture from memory without help
- [ ] This submission is mine: my own evidence, my own reflection, not my teammate's

**Stretch.** Draw a second architecture diagram showing the product as it will be at Checkpoint 4, with auth, the AI feature and monitoring on it. Mark in a different style every box that does not exist yet. Write 100 words on which of those boxes you are least confident about.

## Stuck? Ask AI
```prompt
I have an architecture diagram of my product as ASCII, pasted below. Act as someone who has never seen my product. Tell me, box by box, what you think each one does and what you think travels along each arrow. Where you have to guess, say "I am guessing" and tell me which label is missing. Do not improve my diagram - I want to know what it fails to communicate. [paste the diagram]
```

```prompt
My group has spent a week discussing our idea and we have almost nothing written down. Ask me six questions, one at a time, waiting for my answer each time, that would extract a one-page PRD from what is already in my head. Do not invent any facts about my users; if I give you a vague answer, push back and ask for a number or a quote.
```

```prompt
Explain the difference between an artefact and a discussion, using a non-software example such as opening a food stall. Then give me five things a student team typically believes it has agreed in week one but has not actually written down anywhere, and for each, tell me what the written version would look like.
```

## Quiz

### Q1 · L3
Why is Checkpoint 1 scheduled at the end of week one rather than at the end of the Sprint?
- (a) Because the platform lists M11 last in the module order
- (b) Because the product is built by the missions in M01–M10, so the gates have to run alongside them ✅
- (c) Because the PRD cannot be changed after week one
- (d) Because architecture diagrams become impossible to draw later
> The module is read across the 30–45 days. `order_index` puts M11 last for platform reasons, but a gate that arrives after the work is finished cannot change anything.

### Q2 · L4
Your group has a shared repository with everything committed. How many evidence packs are submitted for this checkpoint?
- (a) One, from the member who holds the tech role
- (b) One per group, since the product is shared
- (c) Three — one per member, each with their own reflection ✅
- (d) Two: one for the product and one for the architecture
> The product is shared; the completion record is not. Each submission belongs to one student, and copying a teammate's reflection is how students arrive at the individual defence unable to explain their own product.

### Q3 · L4
Which of these counts as evidence that Checkpoint 1 passed?
- (a) The team agrees it understands the problem well
- (b) A committed `PRD.md` whose SUCCESS line has a number and a date ✅
- (c) A detailed plan for writing the PRD next week
- (d) A long group chat where the scope was argued out
> A checkpoint asks what exists, not what you know. A file with a commit date is checkable by a stranger; a shared understanding is not.

### Q4 · L4
Why does this chapter insist you draw the architecture diagram before writing any code?
- (a) Agents refuse to work without a diagram
- (b) A diagram drawn before the code is a decision; drawn afterwards it is only documentation ✅
- (c) It is required for the deployed URL to work
- (d) Diagrams are easier to draw when the system is larger
> Week one is the only time the diagram is cheap, because nothing exists that it could contradict. It is also how you find out that two of you assumed different databases.

### Q5 · L5
A group submits eight decision-log entries in the final week, all written that evening. What does a reviewer notice first?
- (a) That the entries are too short to be useful
- (b) That no entry records a choice that later turned out to be wrong ✅
- (c) That the file is not committed to the repository
- (d) That the entries use the wrong template
> A log written with hindsight only contains decisions that worked, because the bad ones have already been quietly reversed. Dated entries across weeks are the only version that shows learning — which is why this gate asks for two in week one.

## Go deeper
- [The C4 model for visualising software architecture](https://c4model.com/)
- [ADR — templates and tooling for decision records](https://adr.github.io/)
