---
id: m01-10-portfolio-and-decision-log
module: m01-team-startup-git
slug: portfolio-and-decision-log
title: "Your Daily Portfolio and Decision Log"
order: 10
duration_min: 15
is_free: false
mission: false
volatility: low
last_reviewed: 2026-11-16
---

## Key learnings
- Post work daily in a form that is evidence rather than a claim
- Write a decision log entry that still makes sense in four months
- Explain why the rejected option is the most valuable part of an entry
- Tell the difference between a portfolio and a list of screenshots
- Build the record that your technical defence depends on

## Lesson

### The pain

Two students finish the same program with similar products. The first has thirty posts, each showing something built that week: the brief, what got built, a failing test and then a passing one, a note on what the AI got wrong. An interviewer can scroll it and watch a beginner become an engineer.

The second has a working product and nothing else. Asked why they chose polling over realtime, they cannot remember — a reasonable week-four decision, for a reason obvious at the time. "It was simpler" sounds like they never considered the alternative.

### The idea

**Post the work as it happens, and write down each real decision with the option you rejected. Both are evidence; neither can be reconstructed later.**

A portfolio built at the end is a claim. A portfolio built daily is a record, and the difference is visible to anyone who has seen both.

### The analogy

The canteen register, with a second column. The first is what you sold — daily, visible, countable: your portfolio. The second is why anything changed: "raised samosa to ₹15 — oil up 20%, considered a smaller samosa, rejected because students counted them." That is the decision log. Column one proves the stall worked; column two proves you ran it rather than watched it.

**Where the analogy breaks:** a register is private. Your portfolio is read by people deciding whether to hire you, so it must be legible to someone who was not there.

### Picture it

```
  every working day                    what it becomes
  ┌────────────────────┐
  │ you build, direct  │
  │ or verify something│
  └─────────┬──────────┘
            │
    ┌───────┴────────┐
    ▼                ▼
┌──────────┐   ┌───────────────┐
│  A POST  │   │ was it really │
│ caption  │   │  a decision?  │
│ ai_chat  │   └───────┬───────┘
│ repo     │           │ yes
│ demo     │           ▼
│ images   │   ┌───────────────┐
└────┬─────┘   │ DECISIONS.md  │
     │         │ incl. REJECTED│
     │         └───────┬───────┘
     ▼                 ▼
  a record a        the answer to
  stranger can      "why did you
  follow            do it that way?"
```

One day's work splits two ways: the visible part becomes a post with its evidence attached, and anything that was genuinely a choice also gets an append-only log entry naming the option you turned down. Five lines, short enough to write before you leave.

```
2026-11-18
DECISION    Students' menu refreshes by polling every 60 seconds.
WHY         Menu changes ~3 times a day; a 60s delay is acceptable.
            Free tier gives no realtime connection quota.
REJECTED    Realtime subscriptions — needs a paid plan, and adds a
            connection per open tab, which is our scaling risk at 1 pm.
REVISIT IF  We move to a paid tier, or staff start changing
            availability during the rush.
```

**REJECTED does the work**, because it proves you chose rather than defaulted: "we used polling" invites "did you consider realtime?" and leaves you improvising. **REVISIT IF** makes the log a tool rather than a diary, naming the condition under which the decision expires.

A log with forty entries is a log nobody reads. Log a decision when it is expensive to reverse, had a real alternative, will be questioned by someone who was not there, or involves money or user data. One to three a week is healthy.

### Where it lives

`DECISIONS.md` lives in your repository, appended to and never rewritten, so each entry is dated by its commit as well as its text — hard to fabricate, therefore credible. M11's Checkpoint 4 asks for it; M18's defence is built on it.

Your posts live on the platform, which gives each one four slots plus images. Used properly, a post is self-proving.

| Slot | What goes in it | Why it matters |
|---|---|---|
| Caption | What you set out to do, and what you learned | The thinking, not the feature list |
| AI chat | The share link to the actual conversation | Proof you directed rather than copied |
| Repo | The branch or PR | Proof the code is real |
| Demo | The live URL | Proof it runs |
| Images | Before and after, a failing test, a diff | Proof you verified |

In a group of three the product is shared but your posts are not — they record *your* contribution. Chapter 1 warned that the fastest person can quietly absorb all the work; a thin posting history is the earliest visible symptom.

### Words to own

| Term | Plain meaning | Example |
|---|---|---|
| Decision log | A dated record of choices and reasons | `DECISIONS.md` |
| ADR | Architecture Decision Record; the formal version | One entry per decision |
| Rationale | The reason behind a choice | "3 changes/day, 60s is fine" |
| Tradeoff | What you gave up to get something | Staleness for free hosting |
| Prompt journal | The briefs and iterations you kept | Part of your portfolio |
| Artefact | A concrete thing that proves work happened | A PR, a diff, a URL |
| Append-only | Added to, never edited | What makes a log credible |
| Defensible | You can explain and justify it | The goal of both practices |

### Tradeoffs and gotchas

**Daily beats good.** A two-minute post every day builds a record; a perfect post once a month does not. The bar is "did I show one true thing today".

**Do not post only successes.** Thirty wins reads as a highlight reel. Failures demonstrate diagnosis, and diagnosis is the scarce skill — so the two most differentiating posts are **the thing that broke**, error then fix, and **what the AI got wrong**.

**A caption that lists features is weak.** "Added login, cart and checkout" is a changelog. "Stored the cart in memory, lost it on every redeploy, moved it to the database — here's the before and after" is a portfolio post.

**Write the decision at the end of the session.** The constraint that drove it — a free-tier limit, a number from your interviews — evaporates first.

**Never rewrite a log entry; append a new one referencing the old.** An edited log is worthless as evidence, and the sequence of changing your mind beats appearing to have been right all along.

**Do not let the agent write the log.** Plausible rationale is exactly wrong — the reasoning has to be yours.

### Verify it

Three checks on the log, three on the portfolio, once a week.

1. **Does every entry have a REJECTED line with a reason in it?** Without one it does not prove you chose.
2. **Does every WHY contain a number or a named constraint?** "It was simpler" is not a reason.
3. **Is it append-only, with commits spread across weeks?** Ten entries committed on one day in month five fool nobody.
4. **Could a stranger follow your last post?**
5. **Have you posted anything that went wrong this week?** All successes means curating, not recording.
6. **Is the AI conversation linked?** That separates your portfolio from someone who pasted output.

Red flag: you cannot pick a decision from a month ago and explain it in thirty seconds, including what you rejected.

## Walkthrough
1. Create `DECISIONS.md` in your repository with the five-field template at the top as a reminder of the format.
2. Write your first three entries retroactively, for decisions you have already made in this module: the role split, the problem you chose, and your slice-one scope. Include the REJECTED line for each — you did consider alternatives, so write them down before you forget.
3. Check each WHY. If it does not contain a number or a named constraint, go back to your interview notes or your PRD and put one in.
4. Commit it. The commit date is part of the evidence.
5. Now make your first portfolio post. Pick anything you actually did this week — writing the PRD counts, cutting the slice counts.
6. Write the caption as a decision rather than a feature list: what you set out to do, what surprised you, what you changed as a result.
7. Fill the slots: the AI conversation share link, the repository or PR link, the live URL if anything is deployed yet.
8. Attach an image that shows process rather than outcome. A diff, a failing test, your before-and-after PRD, your interview notes.
9. Post it. Then set a recurring reminder for the end of each session — the habit needs a trigger, not willpower.
10. Agree in `TEAM.md`: each person posts daily under their own account, and whoever is in the session writes the day's decisions before leaving.
11. At the end of the week, open your own profile and read it as a stranger. Anything you cannot follow, fix in next week's posts.

## Talk to AI
```prompt
I am keeping a decision log for a six-month project and I want my entries to survive a technical interview four months from now.

Here is my entry:
[paste it - date, decision, why, rejected, revisit if]

Review it as an interviewer who was not there:
1. Is my WHY an actual reason, or a restatement of the decision? Quote it and say which.
2. Does my WHY contain a number or a named constraint? If not, what kind of evidence is missing?
3. Is my REJECTED line a real alternative I weighed, or something I never seriously considered? Can you tell the difference from what I wrote?
4. What is the first follow-up question you would ask me about this decision - the one my entry does NOT answer?
5. Is my REVISIT IF condition something I would actually notice happening, or is it vague?
6. In four months, reading only this entry, could I rebuild the reasoning? Name the one thing I would have forgotten.

Then rewrite ONLY the weakest line, and tell me what you changed and why. Do not rewrite the whole entry - the reasoning has to stay mine.
```

## Stuck? Ask AI
```prompt
I do not know what counts as worth logging. Here are eight things my team decided this week: [list them]. Sort them into "log this" and "do not log this", and explain the test you applied. Do not tell me the answers until I have guessed first.
```

```prompt
My portfolio captions all read like changelogs — "added X, fixed Y". Here are three of them: [paste them]. Rewrite each as a post about a decision or a problem, and tell me what makes the rewritten version stronger evidence.
```

```prompt
Explain why the REJECTED line in a decision log entry matters more than the DECISION line, using an example from outside software. Then explain what it signals to someone reading it months later.
```

## Quiz

### Q1 · L4
Two students build similar products. One has thirty daily posts; the other has only the finished product. In an interview, what is the practical difference?
- (a) The first student's product is more likely to be technically better
- (b) The first can prove how the work and the thinking developed; the second can only claim it ✅
- (c) The second student will be asked fewer questions
- (d) There is no real difference if both products work
> A portfolio built at the end is a claim; one built daily is a record. The second student cannot reconstruct a reason they have forgotten.

### Q2 · L4
Which line of a decision log entry does the most work in a technical defence?
- (a) DECISION, because it states what was built
- (b) WHY, because it gives the reason
- (c) REJECTED, because it proves you chose rather than defaulted ✅
- (d) DATE, because it establishes the timeline
> "We used polling" invites "did you consider realtime?" The rejected option answers the follow-up before it is asked, and distinguishes a choice from a default.

### Q3 · L4
Which WHY line is usable four months later?
- (a) It was simpler
- (b) It seemed like the best approach at the time
- (c) Menu changes about 3 times a day, so 60s staleness is acceptable, and the free tier has no realtime quota ✅
- (d) The AI recommended it
> A WHY must contain a number or a named constraint. The constraint that drove the decision is the first thing to evaporate from memory.

### Q4 · L3
A decision you logged in month two turns out to be wrong in month four. What should you do?
- (a) Edit the original entry so the log stays accurate
- (b) Delete the entry, since it no longer reflects reality
- (c) Append a new entry referencing the old one ✅
- (d) Leave it and explain verbally if asked
> An edited log is worthless as evidence. The sequence of changing your mind with reasons is more impressive than appearing to have been right throughout.

### Q5 · L4
What is the single most differentiating thing to post in a portfolio?
- (a) A screenshot of a finished feature working
- (b) A list of the technologies you used
- (c) A case where you caught the AI being wrong, and what you changed ✅
- (d) The total number of commits you made
> Anyone can show AI output. Demonstrating that you caught it being wrong shows the judgement the whole program trains, and it is what a working screenshot cannot prove.

## Go deeper
- [Architecture Decision Records — the original lightweight format](https://cognitect.com/blog/2011/11/15/documenting-architecture-decisions)
- [ADR templates and examples](https://github.com/joelparkerhenderson/architecture-decision-record)
