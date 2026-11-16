---
id: m18-01-architecture
module: m18-demo-day
slug: architecture-walkthrough
title: "The Architecture Walkthrough Document"
order: 1
duration_min: 35
is_free: false
mission: true
volatility: low
last_reviewed: 2026-11-16
---

## Key learnings
- Write an as-built architecture document that matches what is deployed today
- Label every arrow with the payload that travels and every boundary with what it trusts
- Annotate which boxes you personally directed and which you only reviewed
- Catch diagram drift by reading your hosting console beside your own picture
- Hand a stranger a document they can follow with you not in the room

## Lesson

### Where this sits in the module

**This is the first of six chapters that make up your individual defence, and it is the document everything else is read against.** Chapter 2's what-if questions are asked about boxes on this map. Chapter 3's live debugging follows the trace inside it. Chapter 5's handover is this document plus money, keys and known issues. Chapter 6 is the room where all of it is questioned.

Three things about this module before you start. It is **individual**: the product is built by three of you, the defence is not shared, and each of you submits your own evidence against your own account. It is **terminal**: there is nothing after it. And it builds directly on **M11.7**, the Sprint's Checkpoint 4 — your ten users, your eval set, your decision log and your ten-minute technical demo. That pack is your starting material, not your deliverable. Four months of M12 to M17 have happened since, and almost everything in it is now out of date.

### The pain

Three students present a system diagram in month six.

The first opens the diagram committed at M11.1, in week one. It shows three boxes and a model API. Since then the product acquired a payment webhook, a cache, a scheduled job that recomputes a leaderboard at 2 am, a second database for analytics events, and a queue that was briefed, half-built and abandoned. The panel asks where a UPI webhook lands. There is no box for it. The diagram is not wrong because the student is careless; it is wrong because it was never updated, and nobody noticed for four months.

The second asked an AI to draw their architecture from a paragraph of description. What came back is beautiful, symmetrical, and contains a load balancer, a Redis cache and a message broker — none of which exist. Asked which box holds the model key, the student points at the broker.

The third has a document with seven tables listed by name, an arrow labelled `signed payment event`, and a line that says: *the leaderboard job is the one thing I did not direct; my teammate built it, I reviewed the diff and I can explain what it reads but not why it runs at 2 am.* That last sentence is the strongest line in all three submissions.

### The idea

**An architecture walkthrough is a written document, not a slide, whose only virtue is that it is true of what is deployed right now — and which says plainly who directed each part of it.**

A diagram drawn in week one was a decision, and M11.1 was right to ask for it. A diagram drawn in month six is a **claim about reality**, and reality can be checked. The document is what makes the next five chapters answerable instead of improvised.

### The analogy

The canteen has a laminated floor plan taped beside the till: where the gas cylinder sits, which burner runs off which regulator, where the water line enters, which exit stays clear, and the supplier's number. An inspector reads it and then walks the kitchen to confirm it. Both halves matter — the plan, and the walk.

**Where the analogy breaks:** a kitchen can be verified by standing in it. Your system has no room to stand in. Every box on your map is invisible, which means a plan can be wrong for four months without a single symptom, and nobody — including you — will find out by looking. The walk has to be replaced by something deliberate: reading your hosting console, your environment variable list and your database table list beside your own picture, one line at a time.

### Picture it

```
  ┌──────────────┐   tap + session cookie   ┌──────────────────┐
  │ phone        │─────────────────────────▶│ app server       │
  │ browser      │◀─────────────────────────│ role + price are │
  └──────────────┘   html + order id        │ decided here     │
                                            └──┬────────────┬──┘
  ┌──────────────┐   signed payment event      │            │
  │ UPI gateway  │────────────────────────────▶│            │
  └──────────────┘                             │            │
                                  order rows   │            │ prompt +
                            ┌──────────────┐◀──┘            │ 3 chunks
                            │ Postgres     │                ▼
                            │ 7 tables     │        ┌────────────────┐
                            └──────────────┘        │ model API      │
                                                    │ key lives only │
                                                    │ on the server  │
                                                    └────────────────┘
```

Everything to the left of the app server is untrusted input — a phone and a payment gateway — and the server is the only box that decides a role or a price, holds the model key, or speaks to the database.

### What the document contains

Seven sections. Short ones.

```
  ARCHITECTURE.md
  1 ONE PARAGRAPH  what the product does, for whom, in plain words
  2 THE MAP        boxes and labelled arrows, exactly as deployed
  3 THE DATA       every table, what it holds, who may read it
  4 ONE TRACE      a single tap followed through named files
  5 BOUNDARIES     what is trusted; where money and personal data sit
  6 DEPENDENCIES   every external service, and what breaks if it stops
  7 ATTRIBUTION    who directed each box, and what I cannot explain yet
```

The seven sections of the walkthrough document, in the order a stranger reads them.

Section 4 is the one that separates students. One tap, named files, in order: the page the user touched, the route that received it, the function that checked the role, the query that ran, and the response. This is M00.1's four-boxes exercise and M11.7's trace, now on a system with four more months of growth in it.

Section 7 is why this module exists. For every box, one of three labels: **I directed this** — I wrote the brief, reviewed the diff, verified it. **I reviewed this** — a teammate directed it, I read the change and tested it. **I inherited this** — it exists, I can say what it does, I did not verify it. Three honest labels beat one dishonest claim, and an empty section 7 is the clearest possible signal that one person did the engineering.

### Where it lives

The underlying material is all behind you. M00.1 gave you the four boxes and M00.3 the first diagram. M11.1 asked for the week-one version. M04.7 is your schema, M05.3 your cache, M07.4 to M07.7 your monitoring, M12 your mobile surface, M13 your production architecture. M16 and M17 added analytics tables and billing.

Forward, it feeds every remaining chapter of this module: the ten what-if questions in M18.2 are asked about the boxes on this map; the live debugging in M18.3 travels along section 4's trace; M18.5 extends sections 5 and 6 into costs, access and known issues; and M18.6 is the twenty minutes in which you present it.

### Words to own

| Term | Plain meaning | Example |
|---|---|---|
| As-built | The system as actually deployed, not as planned | Seven tables, not the five you designed |
| Drift | The gap between the document and reality | A queue on the diagram that was never built |
| Trust boundary | The line where input stops being trusted | Browser to server; gateway to server |
| System of record | The one place a fact officially lives | `orders` table, not the page state |
| Payload | What travels along an arrow | `signed payment event`, not "sends data" |
| Dependency inventory | Every outside service you rely on | Host, database, model API, email sender |
| Blast radius | What else fails when one box fails | Model API down: suggestions only, not orders |
| Attribution | Who directed which part, honestly labelled | "I reviewed this, I did not direct it" |
| Hot path | The request every user makes | Open menu, place order |

### Tradeoffs and gotchas

**Do not let AI draw the map from your description.** It will produce the average of ten thousand tutorials: a load balancer you do not have, a Redis you never provisioned, a broker nobody briefed. This is M00.7's generic-output failure, and on a diagram it is worse than useless because it is persuasive. Draw the boxes yourself from the console, then ask AI to attack what you drew.

**A box per service is not a map of your system.** The value is in the arrows. "App server to Postgres" tells a reader nothing; "order rows, written inside one transaction with the payment row" tells them where your money lives.

**Detail is not accuracy.** A map with thirty boxes that nobody can hold in their head is worse than nine boxes that are all true. If something can be deleted without the reader misunderstanding the system, delete it.

**Things your host provides still belong on the map, marked as such.** TLS termination, the CDN in front of your pages, the connection pooler your database platform runs. You did not build them; you depend on them; they are in your blast radius. Saying "we have a load balancer" when your host has one and you have never configured it is the exact overclaim a panel probes.

**Section 7 is uncomfortable and that is its purpose.** Writing "I inherited this" next to three boxes feels like admitting weakness. It is the opposite: it is the only version of the document a reviewer can trust, and it tells you precisely what to study in the days before the defence.

### Verify it

Six checks, and five of them are done with a second window open.

1. **Open your hosting dashboard beside your map.** Every deployed service, cron job and environment appears on the map or is deliberately absent. Count them.
2. **Open your environment variable list.** Every variable implies a dependency. Any key whose service is not on the map is a missing box — and quite possibly a service nobody remembers signing up for.
3. **Open your database table list.** Every table appears in section 3 by name, with one line on what it holds. A table you cannot describe is a finding.
4. **Point at every arrow and say the payload out loud.** An arrow you can only describe with a verb is an undecided design, exactly as it was in M11.1.
5. **Read section 4 aloud while clicking through the live product.** The file names must come in the order the request actually travels. If you have to say "and then something happens", that is this week's study.
6. **Hand the document to someone outside your group and leave the room.** Ask them afterwards: what does the server decide, where does money live, what breaks if the model API is down. Three wrong answers mean the document, not the reader, is wrong.

The red flags a panel finds in seconds: a diagram that does not match what is deployed; a box the student cannot name a file for; an arrow labelled "data"; a cache or queue on the map that was abandoned half-built; no trust boundary at all; personal data in a box nobody has thought about; and an attribution section that claims all nine boxes for one student in a group of three.

## Walkthrough
1. Create `ARCHITECTURE.md` with the seven section headings and nothing under them. Commit it empty, so the file's history shows when the work started.
2. Open your hosting dashboard, your environment variable list and your database table list in three tabs. Write down every service, job, environment and table you find, as a flat list, before drawing anything.
3. Draw the map by hand from that list, as ASCII, with no AI involved. One box per thing that actually runs. Keep it under seventy-two characters wide so it reads on a phone.
4. Label every arrow with the payload that travels along it. Where you cannot name the payload, mark the arrow with a question mark and keep going; that list of question marks is your study list.
5. Draw the trust boundary. Mark which boxes accept input from outside, which box decides roles and prices, and which boxes hold personal data or money.
6. Write section 3 by walking your table list, one line each: what it holds, who may read it, and which table is the system of record for an order.
7. Write section 4 by clicking the single most important action in the live product and following it through your own files, naming each one in order. Do this on the deployed URL, not locally.
8. Write section 6 from your environment variables: every external service, what it does for you, and one sentence on what the user experiences if it stops answering.
9. Write section 7 honestly. Label every box "directed", "reviewed" or "inherited", in your own copy, under your own name. Do not coordinate these labels with your teammates before writing them.
10. Run the AI critique prompt below against the finished document. Fix what you agree with and record one thing you rejected and why, as a decision-log entry.
11. Hand the document to someone outside your group, leave the room, and ask them the three questions from the verification list. Fix whatever they got wrong, then compare your section 7 with your two teammates' and note where the three of you claim the same box.

## Talk to AI
```prompt
I am preparing an individual technical defence of a product my team of three built over six months. Below is my as-built architecture document. Interrogate it as a hostile reviewer who will question me on it tomorrow. Do not rewrite it, do not improve my diagram, and do not write any code.

MY DOCUMENT
One-paragraph description: [paste it]
The map, as ASCII, exactly as I drew it: [paste it]
Every table and what it holds: [paste section 3]
My trace of one request, naming my own files in order: [paste section 4]
Trust boundaries, and where money and personal data sit: [paste section 5]
Every external service I depend on: [paste section 6]
My attribution labels per box - directed, reviewed, or inherited: [paste section 7]
What my hosting dashboard lists as deployed: [paste the list of services, jobs and environments]
My environment variable names, with the values removed: [paste the names only]

Work through these in order, quoting my own words back to me:

1. DRIFT. Compare my map against the deployed services and the environment variable names I pasted. Name every box that appears on one list and not the other, in both directions. Which environment variable implies a dependency that is missing from my map?
2. ARROWS. Quote every arrow label that is a verb or a vague noun rather than a payload. For each, tell me what question I will be asked about it.
3. OVERCLAIM. Which parts of my map am I probably describing as mine when they are provided by my host or my database platform? How should each be labelled instead?
4. TRACE. Read my request trace as someone who has never seen my code. Where does it skip a step? Where do I use a word I never defined? Name the first point at which you stop being able to follow it.
5. DATA. Given my tables, where does personal data sit, which table is the system of record for the thing users care about most, and which table would be the most damaging to lose?
6. BLAST RADIUS. For each external service, tell me what my users would experience if it stopped answering for an hour, and whether my document says so.
7. ATTRIBUTION. Given my labels, which three boxes am I least able to defend? Write the hardest question a reviewer would ask me about each.

Then describe my system back to me in four sentences, using only what I pasted. If that description is not my system, tell me exactly which line of mine misled you.

Finally: name the one thing about this architecture a reviewer would ask that my document gives no way to answer.
```

## Mission

**Objective.** An `ARCHITECTURE.md` in your repository that is true of what is deployed today, verified line by line against your hosting console, and annotated under your own name with which parts of the system you personally directed.

**Steps.**
1. **Inventory** — list every deployed service, job, environment, table and environment variable before you draw anything.
2. **Draw** — the map by hand, as ASCII, under seventy-two characters wide, no AI.
3. **Label** — every arrow with its payload; mark the trust boundary and where money and personal data sit.
4. **Tabulate** — section 3, one line per table, naming the system of record.
5. **Trace** — one real request through your own named files, on the deployed URL.
6. **Depend** — section 6, every external service and what a user feels when it stops.
7. **Attribute** — section 7, every box labelled directed, reviewed or inherited, written alone.
8. **Critique** — run the prompt above; fix what you agree with, record one rejection.
9. **Test on a stranger** — hand it over, leave the room, ask the three questions, fix what they got wrong.

**Upload.**
- `ARCHITECTURE.md` → attached file, plus the repository in `repo_url`
- The live URL → `demo_url`
- Your ASCII map as a code block in the post caption
- A screenshot of your hosting dashboard's service list beside your map, showing they agree
- A screenshot of your database table list, matching section 3
- Your request trace, written out, naming the files in order
- Your attribution section, with the count of boxes you labelled directed, reviewed and inherited
- The drift you found: every box, job or service that was missing from the old M11.1 diagram
- The AI critique conversation → share link in `ai_chat_url`
- Reflection, 150 words max → the box you were least able to explain, and what you did about it

**Self-check.** Tick every box before you submit.
- [ ] Every service, cron job and environment in my hosting dashboard appears on my map
- [ ] Every environment variable maps to a box, or I have explained why it does not
- [ ] Every table in my database appears in section 3 with one line on what it holds
- [ ] I have named which table is the system of record for my core action
- [ ] Every arrow carries a payload label, not a verb
- [ ] My trust boundary is drawn, and the server is the only box deciding roles and prices
- [ ] Section 5 names every place personal data is stored
- [ ] My trace names real files in the real order, checked against the deployed URL
- [ ] Section 6 lists every external service and what a user feels when it stops
- [ ] Anything my host provides is marked as provided, not claimed as built
- [ ] My map is at most seventy-two characters wide and readable on a phone
- [ ] I listed the drift between this document and the week-one diagram
- [ ] An outside reader answered all three verification questions correctly
- [ ] Section 7 labels every box directed, reviewed or inherited, and I wrote it alone
- [ ] This submission is my own work; section 7 names the parts of this system I personally directed, and it differs from my teammates'

**Stretch.** Draw a second map of the same system as it would be at a thousand daily users, marking in a different style every box that would have to change and every box that would break first. Write 100 words on which single change you would make first and what it would cost per month. Keep this diagram — M18.2's scaling question and M18.5's growth ceilings both ask for it again.

## Stuck? Ask AI
```prompt
I have to draw my own system's architecture from what is actually deployed, and I keep wanting to draw the diagram I imagined rather than the one that exists. Do not draw anything for me. Instead, interview me: ask me one question at a time about what I found in my hosting dashboard, my environment variables and my database table list, and after each answer tell me which box or arrow that implies and what I still have not accounted for. Stop when I have named every box. Then tell me which three answers sounded like guesses.
```

```prompt
Explain what a trust boundary is to someone who has never heard the term, using a non-software example such as a shop counter and a storeroom. Then ask me, for my own product, which inputs arrive from outside, which single place re-decides anything about money or permissions, and what would go wrong if I trusted a value that crossed the boundary without being checked.
```

```prompt
I can describe my product but I cannot follow one request through my own files. Ask me what happens when a user taps the single most important button in my product, and interrupt me every time I skip a layer or use a word I have not defined - page, route, handler, query, response. Keep pushing until I can say the whole path with real file names in the real order. Do not explain my code to me; make me do it.
```

## Quiz

### Q1 · L4
Your week-one diagram from M11.1 still shows three boxes, but the product now has a payment webhook, a cache and a nightly job. What is the problem called, and why is it hard to notice?
- (a) Scope creep, because the product grew beyond the PRD
- (b) Drift, because a diagram can be wrong for months with no visible symptom ✅
- (c) Technical debt, because the diagram was never refactored
- (d) Overclaim, because the diagram describes more than exists
> Nothing breaks when a document becomes untrue, so nothing tells you. That is why the map is verified against the hosting console and the table list rather than from memory.

### Q2 · L3
Which arrow label is doing its job?
- (a) The app server sends data to the database
- (b) Communicates with the payment provider
- (c) `signed payment event` ✅
- (d) API call
> Label the arrow with the thing that travels, not the verb. A payload name is checkable and immediately raises the next useful question: who signed it, and where is the signature verified?

### Q3 · L4
You ask an AI to draw your architecture from a paragraph describing your product. What is the most likely result?
- (a) An accurate diagram, since you described the product correctly
- (b) A plausible generic diagram containing services you do not have ✅
- (c) A refusal, because it cannot see your code
- (d) A diagram missing your database
> This is the generic-output failure from M00.7, and on a diagram it is worse than useless because it looks authoritative. Draw from the console, then ask AI to attack what you drew.

### Q4 · L5
In a group of three, why does the architecture document carry a per-box attribution section rather than one shared author line?
- (a) Because the platform requires a different author on each file
- (b) Because the defence is individual, and an unattributed map hides who understood what ✅
- (c) Because it makes the document longer and more thorough
- (d) Because teammates should not review each other's boxes
> One strong developer can otherwise carry the engineering while three identical documents are submitted. Labelling each box directed, reviewed or inherited is what makes the claim checkable.

### Q5 · L4
Your host terminates TLS and runs a CDN in front of your pages. How should these appear in your document?
- (a) Leave them out, since you did not build them
- (b) On the map, marked as provided by the host ✅
- (c) On the map, described as your load balancer
- (d) In the dependency list only, not the map
> They are in your blast radius whether or not you configured them, so they belong on the map. Claiming them as built is the overclaim a panel probes first.

## Go deeper
- [The C4 model — levels of architecture diagram and what each is for](https://c4model.com/)
- [arc42 — a template for documenting a system as built](https://arc42.org/overview)
