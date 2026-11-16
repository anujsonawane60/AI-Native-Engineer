---
id: m11-07-checkpoint-4
module: m11-ship-your-mvp
slug: checkpoint-4-ten-users
title: "Checkpoint 4 · Ten Users, an Eval Set, a Decision Log and a Technical Demo"
order: 7
duration_min: 35
is_free: false
mission: true
volatility: low
last_reviewed: 2026-11-16
---

## Key learnings
- Pass the Sprint's final gate with ten real users from outside your group
- Write an eval set before you look at the output, and report the failures
- Show a decision log of eight entries dated across four weeks
- Present a ten-minute technical demo you can defend on your own
- Tell a product that works from a product that merely demos

## Lesson

### Where this sits in the calendar

**Checkpoint 4 is due in the final week of the Sprint, and it is the terminal gate.** Checkpoint 1 was the PRD and the diagram in week one, Checkpoint 2 the deployed skeleton in week two, Checkpoint 3 auth, data and the AI feature in week three. M11.6 put the product on a name people can type at the start of this week.

This gate is also the hinge into the Company tier. M18's demo day asks each student to present and defend their product individually, with live debugging and an operational handover. Everything you assemble here is what you walk into that room with — which is why this gate is individual in a way the others only hinted at.

### The pain

Four groups present on the last day.

The first demos beautifully and admits, when asked, that the only people who have ever used it are the three of them. Their product has never met someone who did not already know where to tap.

The second has an impressive eval set: forty cases, every one passing. They wrote it on the last evening by looking at what their feature already produced and writing down that that was correct. It is a photograph of current behaviour labelled "good".

The third has a decision log of twelve entries, all written that morning. Every decision in it turned out well, because the entries were written by people who already knew the ending.

The fourth has eleven users, nineteen eval cases of which five fail, eight dated log entries including two that record mistakes, and a demo slower and uglier than the first group's. They are the only one of the four with a product.

### The idea

**The final gate does not ask whether your product impresses. It asks whether it has survived contact with people who are not you, and whether you can prove what you learned.**

Four artefacts, each only convincing with a timestamp: users who used it, a rubric written before the output existed, decisions recorded while still uncertain, and a demo where the person speaking understands what they built.

### The analogy

The stall has been running a month. Judging day is not the owner tasting their own poha. It is two things at once.

Ten strangers paid real money this week, and some came back. That is the only evidence that matters about the product, and it cannot be produced on judging day — it accumulated or it did not.

Separately, a tasting panel works from a **scorecard written before anyone tasted anything**: is it hot, is it salted, is it the right portion for ₹20, is it ready in two minutes. It exists in advance so the owner cannot taste their own cooking and conclude, sincerely, that this is how poha should be.

**Where the analogy breaks:** a taster says "needs salt" in one second with no written rubric, because taste is immediate and shared. Model output is not. It is fluent and plausible whatever it says, and a fluent wrong answer reads better than a terse right one. Without a rubric you will rationalise whatever appeared — which is why the eval set must be written first.

### Picture it

```
  THE EVIDENCE PACK            WHO IT CONVINCES
  ┌──────────────────────────┐
  │ 10 real users            │ ──▶ the product is used
  │   who, when, what broke  │
  ├──────────────────────────┤
  │ eval set, 20 cases       │ ──▶ the AI feature is measured
  │   written BEFORE output  │     not just admired
  │   pass rate, failures    │
  ├──────────────────────────┤
  │ decision log, 8 entries  │ ──▶ the choices were yours
  │   dated across 4 weeks   │
  ├──────────────────────────┤
  │ 10-minute technical demo │ ──▶ you understand it
  │   given by YOU           │
  └──────────────────────────┘
```

Four artefacts, each answering a different question a sceptic asks, none of which can be convincingly produced on the last evening.

### What must exist in the final week

**1. Ten real users from outside your group.** Ten people who are not the three of you, who opened the product on their own device and completed the core action. Record who they are, when they used it, what they did, and what went wrong. Your own accounts do not count. Your mentor does not count. A group chat saying "looks nice" does not count — using it does.

Record the **drop-offs** too. If fourteen opened it and ten finished, the four who stopped are the most valuable thing you learned all month, and your funnel from M11.6 tells you where.

**2. An eval set for your AI feature: at least twenty cases, written before you look at the output.** The hard requirement of this gate, and the one most groups get wrong.

An eval case is three things: an input, what a good answer **must** contain, and what it **must not** contain. Write all twenty from your PRD and your users' real requests, then run them and report the pass rate honestly. **At least five should fail**; if none does, your rubric is too lenient — real quality at this stage sits between sixty and eighty-five per cent, not a hundred.

Include the awkward cases deliberately: empty input, Hinglish or Marathi if your users write that way, a request that is out of scope, an injection attempt, and a request with a number your feature must respect — "under ₹50" means under ₹50. M10.12 is the full treatment; this gate asks for the smallest honest version.

**3. A decision log with at least eight entries, dated across the four weeks.** Two per week, which is what Checkpoints 1, 2 and 3 asked for. Each entry: what you chose, what you rejected, why, and what you would need to see to change your mind.

At least one entry must record a decision that turned out wrong, and what you did about it. A log where everything worked is a log written with hindsight, and it is obvious from outside.

**4. A ten-minute technical demo, presented by you.** Not the group: you. Three students, three demos, each from their own laptop:

- The problem, with a number in it, in thirty seconds.
- The architecture diagram from Checkpoint 1, and what changed since week one.
- One request traced from a tap to the database and back, through your actual code, naming the files — the M00.2 exercise on your own product.
- One failure demonstrated live: the wrong user refused, or the model timing out. Shown, not described.
- Your north star number, and what it was last week.
- One thing you rejected in review, from M11.4, and why.
- One thing you would rebuild, and its cost.

From your own machine, on the live URL. If any part needs a specific laptop, it is not finished.

### Where it lives

This is the end of the Sprint and the entrance to everything after it. At the Build tier, M12 takes this product to app stores and M13 makes it survive load. At the Company tier, M16 turns your north star into retention cohorts, M17 puts a price on it, and **M18 is this demo again, longer, with live debugging and an operational handover, defended individually**. The eval set you write this week is the baseline every later improvement is measured against.

**Each of you submits your own evidence, and here that is not a formality.** The users are shared; the demo, the trace and the reflection are yours. Three identical submissions is the clearest signal that a group contained one engineer and two passengers.

### Words to own

| Term | Plain meaning | Example |
|---|---|---|
| Real user | Someone outside the team, on their own device | Not your own account |
| Drop-off | Someone who started and did not finish | 4 of 14 stopped at login |
| Eval set | Cases with a pass rule, written in advance | 20 inputs with must/must-not |
| Rubric | What counts as a good answer | "Names a dish under ₹50" |
| Pass rate | Fraction of eval cases that pass | 14 of 20, or 70% |
| Baseline | The first honest measurement | What later changes beat |
| Regression | Used to pass, now fails | Caught by rerunning evals |
| Trace | One request followed through your own code | Tap → route → query → render |
| Technical demo | The ten minutes you present and defend | From your own machine |

### Tradeoffs and gotchas

**An eval set written after seeing the output is not an eval set.** The order matters more than the size: twelve cases written first beat forty written afterwards. If you have already seen the output, write cases from your users' requests and your PRD rather than from your feature's answers, and say so in your submission.

**A hundred per cent pass rate is a red flag, not a result.** The rubric must be specific enough to fail: "mentions a dish on today's menu costing under ₹50" can fail; "gives a helpful suggestion" cannot.

**Ten users means ten humans, and recruiting them takes longer than you think.** Budget two days. People say yes and do not follow through; your link arrives during an exam. Over-recruit, and watch at least three of them use it in person without helping. You will want to tap for them. Do not — every second of their confusion is information you cannot get any other way, as in M03.4.

**Do not improve the product during the demo.** Note it, finish, fix it afterwards. The demo is a measurement, and changing the thing while measuring it loses both.

**If you cannot explain a part of your product, say so in the demo.** "This file was generated and I do not fully understand it" is far stronger than a confident invention, and it is the honest position M01.8 asked you to take in review. Inventing an explanation is the one thing an M18 panel reliably catches.

### Verify it

Nine checks on your own evidence. Be harsh — this is the last gate before someone else checks.

1. **Count your users by name.** Which ten? On whose device? On what day? If you cannot name them, you do not have ten.
2. **Check your analytics against your list.** Ten completions from ten distinct people. If the numbers disagree with your list, trust the tool.
3. **Check the date on your eval file against the date your feature first worked.** If the eval set is newer, say so honestly and add cases from user requests rather than from output.
4. **Count your eval failures.** Zero means the rubric is too lenient. Rewrite it until something fails.
5. **Re-run the whole set today.** The pass rate must be reproducible, and anything that used to pass is now a regression worth a log entry.
6. **Look at the commit dates on your decision log.** Eight entries, four weeks, one recording something that went wrong.
7. **Do the whole demo once from a machine that is not yours**, on the live URL, with nothing installed.
8. **Have someone outside the team question you afterwards** — "what happens if the model is down?" is a good one — and answer without notes.
9. **Trace one request out loud, naming the files.** If you cannot, that is this week's work, and it matters more than any remaining feature.

The red flags that end a Sprint badly: a product nobody outside the team has used; an eval set written after seeing the output; a decision log written in one sitting; a demo that only works on one laptop; a feature with no failure state; three identical submissions from one group; and a student who can describe their product but cannot trace a request through it. All are visible from outside, and all are fixable this week if you look today rather than on Friday.

## Walkthrough
1. Open `SCOPE.md` and write your north star metric at the top of a fresh page, with the number it currently stands at and the date. Everything below is aimed at moving it.
2. Write the eval set **today**, before you touch the feature again. Twenty cases from your PRD and from things your users have actually asked for, each with an input, a must-contain and a must-not-contain.
3. Include the awkward ones on purpose: empty input, Hinglish or Marathi if your users write that way, out of scope, a prompt-injection attempt, and a request with a number your feature must respect.
4. Run the whole set once and record the pass rate without adjusting anything. Write down every failure as it is. Expect five or more.
5. Pick the two cheapest failures and fix only those, then re-run the full set to check nothing that passed now fails.
6. Recruit users for two days, over-recruiting because people drop out. Nobody from your group, nobody using your phone.
7. Sit with at least three of them while they use it, say nothing, and write down where they hesitate and what they tapped that you did not expect.
8. Pull your funnel numbers from M11.6 and reconcile them with your named list. Record the drop-offs and where they stopped.
9. Fill the decision log to eight dated entries, adding any week where you are short, and make sure one entry records a decision that turned out wrong.
10. Build your own ten-minute demo in the seven-part shape from the lesson, then rehearse it once out loud, timed, from a machine that is not yours.
11. Have someone outside the team ask you two questions at the end and answer without notes, fix whichever answer you could not give, then submit individually: your own demo recording, your own trace, your own reflection.

## Talk to AI
```prompt
I am at the final checkpoint of a startup sprint and I have to prove my product works rather than merely demos. Help me build an eval set and then interrogate my evidence. Do not write code, and do not tell me my product sounds good.

MY PRODUCT
Problem and users: [paste the PROBLEM and USER sections of your PRD]
North star metric and its current value: [paste it, with the date]
My AI feature, in one sentence: [what it does for the user]
The prompt it uses: [paste it]
Constraints that matter: [money units, language, devices, free tier, anything the answer must respect]
Real requests my users have actually made: [paste 5 to 10 in their own words, including the badly worded ones]
What I have so far: [number of real users outside my team, number of decision-log entries and their dates]

PART 1 - EVAL SET. Write me 20 eval cases for my AI feature. I have NOT shown you my feature's output, and I do not want you to guess at it. For each case give exactly three things: the input, what a good answer MUST contain, and what it MUST NOT contain. Make the must-contain specific enough that a fluent but wrong answer fails it.
Cover all of these deliberately: typical requests; a request with a number my feature must respect; empty input; extremely long input; a request that is out of scope for my product; a request in Hinglish or in my users' own language; a request that tries to override my instructions through injected text; a request where the honest answer is "I cannot help with that".
Then tell me which five of these twenty my feature is most likely to fail, and why.

PART 2 - RUBRIC CRITICISM. Look back at the must-contain rules you just wrote. Which of them could a confident, fluent, wrong answer still satisfy? Rewrite those rules so they cannot.

PART 3 - EVIDENCE INTERROGATION. Act as a sceptical reviewer of my evidence, not my product:
1. Given my user count and how I described getting them, what would make a reviewer doubt these are real users on their own devices? What should I record for each one?
2. Given the dates on my decision-log entries, would a reviewer believe they were written as the decisions were made? What specifically gives it away if not?
3. If my eval pass rate came out at 100%, what would that tell a reviewer about my rubric rather than my feature?
4. Name the three questions I am least likely to be able to answer about my own product, given everything above.

PART 4 - DEMO. Give me a ten-minute technical demo outline for my product with a time budget per section, covering: the problem with a number, the architecture, one request traced from tap to database through my own files, one failure shown live, my north star number, one thing I rejected in review, one thing I would rebuild. Then list the five questions a panel would most likely ask me afterwards, hardest first.

Finally: name the one piece of evidence I cannot create this week because it needed to accumulate, and tell me the most honest thing to say about it.
```

## Mission

**Objective.** The Sprint's terminal evidence pack: ten real users from outside your group, an eval set of twenty cases written before you looked at the output, eight dated decision-log entries, and a ten-minute technical demo you present yourself.

**Steps.**
1. **Baseline** — write your north star's current number and today's date before anything else.
2. **Eval first** — twenty cases written today, from your PRD and real user requests, each with an input, a must-contain and a must-not-contain.
3. **Measure honestly** — run the whole set, record the pass rate, write down every failure unchanged. Expect five or more.
4. **Fix two, re-run all** — the two cheapest failures only, then the full set again to catch regressions.
5. **Recruit for two days** — ten people outside your group, on their own devices. Over-recruit.
6. **Watch three of them** — in person, in silence. Write down every hesitation.
7. **Reconcile** — your named list against your analytics funnel. Record the drop-offs and where they stopped.
8. **Log** — eight dated entries across four weeks, one of which records a decision that turned out wrong.
9. **Demo** — build the seven-part demo, rehearse it once out loud and timed, from a machine that is not yours.
10. **Defend** — two questions from someone outside the team, answered without notes.

**Upload.**
- The live URL → `demo_url`; the repository → `repo_url`
- Your list of ten users: who, when, which device, what they did, what broke for them
- Your funnel numbers, including everyone who started and did not finish, and where they stopped
- Your eval set file, with its commit date visible, and your pass rate before and after the two fixes
- The five or more failing cases, written out, with what the feature actually returned
- Your decision log with eight dated entries, and a pointer to the one that records a wrong call
- Your own ten-minute demo → a screen recording, or a link to the recorded live session
- The request trace you presented: the files it passes through, in order, in your own words
- The two questions you were asked and your answers
- Reflection, 150 words max → the most surprising thing a real user did, and one thing you would rebuild

**Self-check.** Tick every box before you submit.
- [ ] Ten users I can name, none of them in my group, each on their own device
- [ ] Each user's date, what they did and what broke for them is recorded
- [ ] My analytics funnel agrees with my named list
- [ ] I recorded the people who started and did not finish, and where they stopped
- [ ] I sat in silence while at least three people used it
- [ ] My eval set has at least twenty cases
- [ ] Every case has an input, a must-contain and a must-not-contain
- [ ] The eval set was written before I looked at the output, or I have said plainly that it was not
- [ ] The set includes empty input, out-of-scope input, an injection attempt and a number to respect
- [ ] At least five cases fail, and I have written down what the feature actually returned
- [ ] I re-ran the whole set after my fixes and checked for regressions
- [ ] `DECISIONS.md` has at least eight entries, dated across four weeks
- [ ] At least one entry records a decision that turned out to be wrong
- [ ] My demo runs from a machine that is not mine, on the live URL
- [ ] My demo shows one failure live rather than describing it
- [ ] I can trace one request through my own files, naming them, without notes
- [ ] I answered two outside questions without notes
- [ ] My north star number is written down, with its date
- [ ] This demo, trace and reflection are mine, and differ from my teammates'

**Stretch.** Change one thing about your AI feature — a shorter prompt, a cheaper model from M09.13, more retrieved context from M10.11 — and re-run the full eval set. Report both pass rates and the cost per call for each. Write 100 words on the tradeoff. This is the exact shape of the improvement loop M16 and M18 expect from you, and having run it once makes that room much easier.

## Stuck? Ask AI
```prompt
I have already seen my AI feature's output, so I cannot write an eval set blind. Help me do the next most honest thing. Ask me for my PRD, my users' actual requests in their own words, and my constraints - and then write cases from those alone, never from what my feature currently produces. Afterwards, tell me which of your cases I would have been tempted to leave out, and why leaving them out would have flattered my pass rate.
```

```prompt
I could only get four real users outside my team and I need ten this week. Ask me who I have already approached, what I said to them, and what stopped them. Then give me five specific approaches for an Indian college campus, with the exact sentence to open with, and tell me how to tell a real user apart from a polite friend who will never open my link.
```

```prompt
I have to present a ten-minute technical demo of my own product and then be questioned on it. Ask me to trace one request from a tap to the database and back, naming my own files, and interrupt me whenever my explanation skips a step or uses a word I have not defined. Keep going until I can do the whole trace without gaps. Do not explain my own code to me - make me do it.
```

## Quiz

### Q1 · L4
A group's eval set has forty cases and all forty pass. What is the most likely explanation?
- (a) The AI feature is unusually well built
- (b) The rubric is too lenient, or the cases were written from the output ✅
- (c) The eval set is too large to be meaningful
- (d) The model is being run at a low temperature
> Real feature quality at this stage sits between roughly sixty and eighty-five per cent. A perfect score means "gives a helpful suggestion" rather than "names a dish on today's menu under ₹50".

### Q2 · L5
Why must the eval set be written before you look at the feature's output?
- (a) It is faster to write cases without the output in front of you
- (b) Model output is fluent and confident whatever it says, so afterwards you will rationalise it ✅
- (c) The evaluation tool requires cases to be created first
- (d) Writing afterwards produces too many cases to run
> A fluent wrong answer reads better than a terse right one. Without a rubric written in advance you are photographing current behaviour and labelling it correct.

### Q3 · L4
Which of these counts towards the ten real users at this gate?
- (a) Your three group members' own accounts
- (b) A friend who replied "looks nice" in the group chat
- (c) A classmate who opened the product on her own phone and completed the core action ✅
- (d) Your mentor, who reviewed the code
> A real user is someone outside the team who used it on their own device. Praise is not usage, and your own accounts prove only that the path exists for its authors.

### Q4 · L4
A decision log has twelve entries, all written on the final morning, and every decision in it worked out. What does a reviewer conclude?
- (a) The group made unusually good decisions
- (b) The log was written with hindsight, since the bad calls have already been reversed ✅
- (c) The log has too many entries to be credible
- (d) The entries are too short
> Decisions recorded while still uncertain include the ones that turned out wrong. That is why every earlier checkpoint asked for two entries, dated in that week.

### Q5 · L5
Your demo works perfectly on your teammate's laptop but not on yours, because a tool is installed there. What does the gate conclude?
- (a) It is acceptable, since the product itself is deployed
- (b) Present from the teammate's laptop and note the dependency
- (c) It is not finished — a demo tied to one machine is the red flag this gate exists to catch ✅
- (d) It only matters if the missing tool is part of the product
> The whole demo must run from a machine with nothing installed, on the live URL. The same red flag appeared at Checkpoint 2 and it is individually fatal in M18's defence.

## Go deeper
- [Hamel Husain — your AI product needs evals](https://hamel.dev/blog/posts/evals/)
- [OpenAI Evals — a framework and registry of eval sets](https://github.com/openai/evals)
