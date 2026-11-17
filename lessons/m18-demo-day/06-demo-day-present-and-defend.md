---
id: m18-06-demo-day
module: m18-demo-day
slug: demo-day-present-and-defend
title: "Demo Day: Present and Defend"
order: 6
duration_min: 25
is_free: false
mission: true
volatility: low
last_reviewed: 2026-11-16
---

## Key learnings
- Run the thirty-minute defence in its five blocks, to time
- Decide what is shown live and what is recorded, and say which is which
- Recover from a failure on stage by naming it and carrying on
- Answer a question about a part you did not build without faking it
- Set up the machine, the network and the data before anyone is watching

## Lesson

### The pain

Three defences, same cohort, same afternoon.

The first spends nine of their first twelve minutes on slides, never opens the product, and starts the debugging block eight minutes late.

The second opens the product on college wifi at 2 pm. The free-tier database is cold and the first load takes forty seconds. They refresh four times, apologise for three minutes, blame the wifi. The product works perfectly; nobody saw it work, and there was no backup recording.

The third hits the same cold start and says: *that is a cold start on the free tier — twenty seconds the first time, under two after. I will let it load and meanwhile show the recorded version of the same path.* They lose nine seconds, and the room learns something true about the hosting.

### The idea

**Demo day is an operations problem, not a performance problem. Decide in advance what is live, what is recorded, what you will say when something fails, and what you will say when a question lands on a part you did not build.**

**Thirty minutes each, alone, from a machine that is not yours, on the live URL.** Three students from one group means three separate defences, ninety minutes in total, before mentors and at least one outsider. The product is shared; nothing else about the half hour is. The content was settled over six months — these thirty minutes decide whether any of it becomes visible.

### The analogy

The canteen on inspection morning. The food was decided months ago; what decides the morning is whether the cylinder is full, the price list readable and the licence on the wall. And when the mixer trips the fuse, the owner who says *that is the known fault, the workaround is to run it before the fridge* comes out ahead of the one who pretends otherwise.

**Where the analogy breaks:** the inspector can taste the food. Nobody in your room can taste your product — they can only watch what you show and hear how you answer, so for this half hour the showing and answering **are** the product.

### Picture it

```
  THE THIRTY MINUTES
  ┌──────────────────────────────────────────────────────┐
  │  0:00  STORY + PRODUCT   four beats, shown live  6m  │
  │  6:00  ARCHITECTURE      your map and one trace  4m  │
  │ 10:00  LIVE DEBUGGING    one defect, narrated   10m  │
  │ 20:00  WHAT-IF           questions from the ten  7m  │
  │ 27:00  HANDOVER          runbook, costs, access  3m  │
  └──────────────────────────────────────────────────────┘
```

Five blocks with a clock on each, where the longest single block is the one you cannot rehearse.

One rule decides what the room sees: **show live whatever you control; have recorded whatever depends on something you do not.** Live is the live URL, M18.4's single path, one failure state, your dashboard, your documents. Recorded is anything needing a third party — a payment confirming, a notification arriving — or anything slower than thirty seconds. The rule that cannot be bent: **never show a recording while letting the room believe it is live.** A recording announced is professional; one discovered ends your credibility for the remaining twenty minutes. Practically: a tested phone hotspot, recordings **downloaded**, the product warmed before you begin.

Something will fail. Four moves, in order. **Name it** — "that is a cold start", "that is a 500, not what I expected". **Say what you expected**, the difference between a surprise and a bug you know about. **Offer the fallback** — the recording, the second account, the M18.5 workaround. **Keep moving**; the clock is not paused for your apology. A genuine defect is not trouble, it is M18.3, and a panel would rather watch a real diagnosis than a flawless demo.

### Where it lives

Every block is a chapter you have done: M18.1 the architecture, M18.2 the questions, M18.3 the debugging, M18.4 the story, M18.5 the handover. The ancestor is M11.7's ten-minute Checkpoint 4 demo — this is three times the length, individual, with live debugging and a handover added. After this there is no gate: a recording of this half hour is the strongest item in your M01.10 portfolio.

### Words to own

| Term | Plain meaning | Example |
|---|---|---|
| Block | One timed segment of the defence | The ten-minute debugging block |
| Time-box | A fixed length you stop at, finished or not | Six minutes for the story |
| Cold start | The slow first request after idling | 20 seconds on a free tier |
| Fallback | What you switch to when the live thing fails | The downloaded recording |
| Hotspot | Your phone sharing mobile data to the laptop | Tested that morning |
| Seeded account | A test account with safe, realistic data | No real user's phone number |
| Clean profile | A browser with no logins or extensions | So nothing is pre-filled |
| Honest attribution | Saying who directed what, and what you verified | "I reviewed it; I did not build it" |
| Dress rehearsal | The whole thing, timed, on borrowed hardware | Done at least twice |

### Tradeoffs and gotchas

**Questions come in three kinds.** *About a number* — open the dashboard and show it. *About a limit* — use M18.2's five moves. *About a part you did not build* — the question this module exists for, answered in three parts: what the part does, what **you personally verified**, who directed it. *My teammate directed the push notifications. I reviewed that change and tested that a user who mutes them stops receiving them. I cannot tell you how the token refresh works.* That is strong. "I built all of it" is not, and a panel establishes otherwise within two follow-ups. Honest partial knowledge distinguishes a team member from a passenger, and is scored so.

**Change nothing in the twenty-four hours before.** No deploys, no updates, no "one small fix". The commonest cause of a broken demo is a change meant to improve things.

**Do not present from your own laptop.** A defence that needs your machine defends something only you can run. And slides are optional where the product is not — four blocks of five happen in a browser.

**Rehearse the handover block, because everyone loses it.** It is last, three minutes, and an overrun earlier eats it.

**Guard the debugging block above all.** If the story runs long, cut the story. Ten minutes of visible reasoning beats any traction number.

### Verify it

Two full dress rehearsals, minimum, and five checks.

1. **The whole thirty minutes, timed, from a machine that is not yours**, clean browser profile, live URL, recorded.
2. **Do one rehearsal with the wifi turned off at minute three.** Switch to the hotspot or the recording, out loud, without stopping. Unpractised, it will not happen.
3. **Have your stand-in panel ask one question of each kind** — a number, a limit, a part you did not build — answered without notes.
4. **Watch the recording and check for personal data.** A real user's name, phone number or order is an irreversible disclosure on a projector.
5. **Time each block from the recording.** Whichever overran is the one you cut, and it is rarely the one you expect.

The red flags: your own laptop; no recorded backup; a bookmark pointing at localhost; real users' data on screen; a product changed the night before; a student who can describe only what they built; and three indistinguishable defences from one group.

## Walkthrough
1. Book a thirty-minute slot and write the five block times on a card you keep beside the laptop. The card is allowed; notes for your answers are not.
2. Borrow a machine that is not yours and set it up from scratch: a clean browser profile, no extensions, notifications off, the live URL typed in rather than bookmarked, and the font large enough to read from the back of the room.
3. Seed a test account with realistic but fake data, and check that no screen you plan to open shows a real user's name, phone number, email or order.
4. Record a ninety-second capture of the single path from M18.4's third beat, plus any step that depends on a third party, and download both to the laptop so they play without a network.
5. Open the tabs you will need and no others: the product, your dashboard, your error tracker, `ARCHITECTURE.md`, `WHAT-IF.md` and `HANDOVER.md`.
6. Test your phone hotspot on that machine, confirm it has data, and practise switching to it mid-sentence twice.
7. Run a full dress rehearsal, timed, recorded, with a stand-in panel. Warm the product two minutes before you begin so the first load is not a cold start.
8. Run a second rehearsal in which your stand-in turns off the wifi at minute three, injects a defect for the debugging block, and asks one question from each of the three kinds.
9. Watch both recordings with the block times written down, and cut whichever block overran — protecting the debugging block first and the handover block second.
10. Freeze the product. No deploys, no updates, no fixes, for the twenty-four hours before your slot.
11. After your defence, write down every question you were asked and your answer, mark the one you could not answer, and spend the next day closing exactly that gap.

## Talk to AI
```prompt
I have an individual thirty-minute technical defence of a product my team of three built, and I want a rehearsal, not a pep talk. Be a hostile but fair panel. Do not compliment me, do not write code, and do not let a vague answer pass.

MY MATERIAL
My four-beat story, with every number, its denominator and its window: [paste it]
My as-built architecture map and my one-request trace: [paste them]
My ten what-if answers in five moves: [paste them]
My handover: runbook procedures, cost table from the real bill, access inventory, known issues, growth ceilings: [paste them]
What I personally directed, what I only reviewed, and what I inherited: [paste your attribution labels per box]
My setup: [the machine, the network, what is live, what is recorded and why]

RUN THIS AS THE REAL THING, in blocks, one at a time. Do not move on until I have answered.

BLOCK 1, six minutes. Tell me to deliver the story. Then judge only this: did beat one carry a number about people I observed; is my insight an insight or a feature request; did I show one path or tour features; does every traction number have a denominator and a window. Quote me.

BLOCK 2, four minutes. Ask me to walk the architecture and trace one request. Interrupt me at the first skipped layer or undefined word. Then ask which box I am least able to defend, and ask me about that one.

BLOCK 3, ten minutes. Pick a plausible defect in MY system from my own map. Give me only the user report and the exact error. Then respond to each of my moves with the output of that move alone - no hints, no cause. Score me afterwards on reproducing first, reading the exact error, isolating by halving, one hypothesis and one change, and saying what I ruled out.

BLOCK 4, seven minutes. Ask me four of the ten what-if questions, choosing the four my own answers are weakest on. After each, ask one follow-up: how do I know that number, what would it cost, or why not the other option.

BLOCK 5, three minutes. Ask me for costs, access and known issues in one hundred and eighty seconds. Cut me off at the limit and tell me what I failed to cover.

THEN, the questions that actually decide this. Ask me three questions about parts of the system my attribution labels say I did NOT direct. Judge each answer on whether it says what the part does, what I personally verified, and who directed it - and tell me plainly if any of my answers sounded like a claim I cannot support.

Finally: name the single question in this whole rehearsal I answered worst, tell me what a complete answer to it would contain, and tell me what I would have to do in the next twenty-four hours to be able to give it.
```

## Mission

**Objective.** Your individual thirty-minute defence, delivered to time from a machine that is not yours, recorded, with every question you were asked written up and the one you could not answer closed within a day.

**Steps.**
1. **Set up a borrowed machine** — clean profile, typed URL, notifications off, readable font.
2. **Seed and check data** — a test account, and no real user's personal data on any screen you will open.
3. **Record the fallbacks** — the single path, plus anything depending on a third party, downloaded locally.
4. **Prepare the network** — hotspot tested, switching practised twice.
5. **Dress rehearsal one** — all thirty minutes, timed, recorded, with a stand-in panel.
6. **Dress rehearsal two** — with the wifi cut at minute three, a defect injected, and one question of each kind.
7. **Cut to time** — from the recordings, protecting the debugging block first.
8. **Freeze** — no changes to the product for twenty-four hours.
9. **Defend** — the real thirty minutes, recorded.
10. **Close the gap** — write up every question and answer, and fix the one you could not answer within a day.

**Upload.**
- The recording of your real defence → link or attachment
- The live URL → `demo_url`; the repository → `repo_url`
- Your block times, planned against actual, from the recording
- Both dress-rehearsal recordings, including the one with the network cut
- Your setup evidence: a screenshot of the borrowed machine's clean browser, and your seeded test account
- Your recorded fallbacks, and a note of which parts of your defence were recorded and why
- Every question the panel asked, with your answer, written out
- The question you could not answer, and what you did in the following day
- Your three attribution answers: parts you did not direct, what you verified, who directed them
- Reflection, 150 words max → what failed on stage and what you said in the first ten seconds after

**Self-check.** Tick every box before you submit.
- [ ] I presented from a machine that is not mine, with nothing installed
- [ ] I used the live URL, typed, with no bookmark and no localhost anywhere
- [ ] My browser profile was clean and notifications were off
- [ ] No real user's name, phone number, email or order appeared on screen
- [ ] I had a downloaded recording of the single path, and of anything needing a third party
- [ ] I said out loud, every time, when something was a recording and why
- [ ] My hotspot was tested that morning and I had practised switching to it
- [ ] I warmed the product before starting so the first load was not a cold start
- [ ] I completed all five blocks, and the debugging block got its full ten minutes
- [ ] The handover block covered costs, access and known issues inside three minutes
- [ ] I changed nothing about the product in the twenty-four hours before
- [ ] When something failed I named it, said what I expected, offered the fallback and carried on
- [ ] I answered at least one question about a part I did not direct, honestly and in three parts
- [ ] I did not claim any part of the system I cannot explain
- [ ] I recorded the whole defence and watched it back with block times written down
- [ ] I wrote up every question I was asked, including the one I could not answer
- [ ] This submission is my own work; it names the parts of the system I personally directed, and my defence differs from my teammates'

**Stretch.** Within a week, re-record the single block you performed worst — most often the handover or the architecture trace — and post both versions side by side with 100 words on what changed. The program ends here; the habit of watching your own recording and fixing one block does not, and it is the single most transferable thing you take out of this module.

## Stuck? Ask AI
```prompt
I keep running out of time in my thirty-minute defence. Here are my five blocks and what I planned for each, plus the actual times from my rehearsal recording: [paste both]. Do not rewrite my content. Tell me which block is stealing the time, which sentences in it are doing no work, and what I should cut - given that the ten-minute debugging block and the three-minute handover block must survive intact. Then ask me to say the shortened version and time it again.
```

```prompt
Something will break while I am presenting and I will panic. Drill me. Describe one realistic failure at a time - a cold start, a 500, a blank dashboard, the wifi dropping, a payment that never confirms - and make me reply with exactly four things in order: name it, say what I expected, offer my fallback, and carry on. Tell me which of the four I left out each time. Do ten of these.
```

```prompt
I am going to be asked about a part of my product that a teammate directed. Explain why "I built all of it" fails faster than an honest partial answer, then drill me: pick a part of a typical three-person student product, and make me answer in three parts - what the part does, what I personally verified about it, and who directed it. Tell me when my answer drifts into claiming something I did not say I verified.
```

## Quiz

### Q1 · L4
Your product's first load on a free tier takes forty seconds in front of the panel. What is the best response?
- (a) Refresh until it loads, then continue without comment
- (b) Name it as a cold start, state the normal timing, and show the recorded path while it loads ✅
- (c) Apologise and explain that the college wifi is unreliable
- (d) Skip the product and move to the architecture block
> Name it, say what you expected, offer the fallback, keep moving. The room learns something true about your hosting and you lose seconds rather than minutes.

### Q2 · L5
You show a recorded clip of a payment confirming, without saying it is recorded, and a panel member later realises. What is the actual cost?
- (a) A small deduction for not demonstrating it live
- (b) Everything you say for the rest of the session becomes unverifiable ✅
- (c) Nothing, since the recording shows real behaviour
- (d) You must repeat the block live
> A recording announced is professional; a recording discovered is a credibility failure, and a panel cannot then tell which of your other claims were shown rather than demonstrated.

### Q3 · L4
A panel asks how the push-notification token refresh works. Your teammate directed that feature. What is the strongest answer?
- (a) Explain your best guess at how it probably works
- (b) Say the feature is out of scope for your defence
- (c) Say what the feature does, what you personally verified about it, and that a teammate directed it ✅
- (d) Say you built all of it and move on quickly
> Honest partial knowledge distinguishes a team member from a passenger, and it is scored that way. A guess collapses on one follow-up and makes every earlier answer suspect.

### Q4 · L3
Your story block overruns by four minutes. Which block should absorb the loss?
- (a) The live debugging block, since it is the longest
- (b) The story block itself, by cutting content before you start ✅
- (c) The handover block, since it is last
- (d) The what-if block, since questions can be answered briefly
> The blocks are time-boxed in advance precisely so the answer is to cut the story rather than the debugging. Ten minutes of visible reasoning is worth more than any extra traction number.

### Q5 · L4
Why must the defence be delivered from a machine that is not yours?
- (a) To make sure the presentation software is compatible
- (b) Because a product that needs your machine is a product only you can run ✅
- (c) Because the panel needs to control the screen
- (d) Because recordings play more reliably on borrowed hardware
> This has been checked since M11.2 and it is individually fatal here. A dependency on one laptop means the deployment, the credentials or the environment are not actually finished.

## Go deeper
- [Y Combinator — demo day advice and what a good live demo looks like](https://www.ycombinator.com/library/6j-how-to-prepare-for-demo-day)
- [Atlassian — running a blameless postmortem on what went wrong](https://www.atlassian.com/incident-management/postmortem/blameless)
