---
id: m01-10-portfolio-and-decision-log
module: m01-team-startup-git
slug: portfolio-and-decision-log
title: "Your Daily Portfolio and Decision Log"
order: 10
duration_min: 25
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

Two students finish the same six-month program with similar products.

The first has a profile with thirty posts, each showing something that was built that week: the brief they wrote, the thing that got built, a screenshot of a failing test and then a passing one, and a note about what the AI got wrong. An interviewer can scroll it and watch a beginner become an engineer.

The second has a working product and nothing else. At interview they are asked why they chose polling over realtime updates. It was a reasonable decision made in week four, for a reason that was obvious at the time. They cannot remember it. They say "it was simpler", which is true and sounds like they never considered the alternative.

Both built the same thing. One can prove it.

### The idea

**Post the work as it happens, and write down each real decision with the option you rejected. Both are evidence; neither can be reconstructed later.**

A portfolio built at the end is a claim. A portfolio built daily is a record. The difference is visible to anyone who has seen both.

### The analogy

The canteen register once more, with a second column.

The first column is what you sold — the daily output, visible, countable. That is your portfolio.

The second column is why anything changed. "Raised samosa to ₹15 — oil up 20%, considered smaller samosa instead, rejected because students counted them." That is the decision log.

Six months later the first column proves the stall worked. The second column proves you were running it rather than watching it. When someone asks why samosas cost ₹15, the answer is a sentence rather than a shrug — and the fact that you considered a smaller samosa and rejected it for a specific reason is what makes you sound like an operator.

**Where the analogy breaks:** a register is private. Your portfolio is public and is read by people deciding whether to hire you, so it has a second audience and needs to be legible to someone who was not there.

### What makes a post evidence

The platform gives each post four slots — a caption, the AI conversation, the repository, and the live demo — plus images. Used properly, a post is self-proving.

| Slot | What goes in it | Why it matters |
|---|---|---|
| Caption | What you set out to do, and what you learned | The thinking, not the feature list |
| AI chat | The share link to the actual conversation | Proof you directed rather than copied |
| Repo | The branch or PR | Proof the code is real |
| Demo | The live URL | Proof it runs |
| Images | Before and after, a failing test, a diff | Proof you verified |

The weakest post is a screenshot of something working. It shows an outcome and no process, and an outcome can come from anywhere. The strongest post shows a decision: here is what I tried, here is what went wrong, here is what I changed and why.

Specifically worth posting, because most people do not:

**The thing that broke.** A screenshot of the error and then the fix is more convincing than the fix alone, because it shows diagnosis.
**What the AI got wrong.** This is the single most differentiating thing you can post. Anyone can show AI output. Showing that you caught AI being wrong demonstrates the judgement this whole program is about.
**The option you rejected.** Same reason as the decision log.

### The decision log entry

Five lines. Short enough to actually write at the end of a session.

```
DATE
DECISION    What we are doing, in one sentence
WHY         The reason, with a number or constraint in it
REJECTED    What we did not do, and why not
REVISIT IF  The condition that would change this
```

A real one:

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

**REJECTED is the line that does the work.** It proves you chose rather than defaulted. In M18's defence, "we used polling" invites "did you consider realtime?" and leaves you improvising. The entry above answers the follow-up question before it is asked, four months after you stopped remembering.

**REVISIT IF** is the line that makes the log a tool rather than a diary. It tells your future self the condition under which this decision expires — and when that condition arrives in month four, you will find the entry and know immediately what to change.

### What counts as a decision

Not everything. A log with forty entries is a log nobody reads.

Log it when the decision:
- is hard or expensive to reverse later (the data model, the auth approach)
- had a real alternative you seriously considered
- will be questioned by someone who was not there
- involves money, user data, or a tradeoff you accepted knowingly

Do not log naming a variable, or picking a colour, or anything where there was only one sensible option. Roughly one to three entries a week is healthy.

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

One day's work splits two ways. The visible part becomes a post with its evidence attached; anything that was genuinely a choice also gets an append-only log entry naming the option you turned down.

### Where it lives

The decision log lives in your repository as `DECISIONS.md`, appended to, never rewritten. Being under version control means each entry is dated by its commit as well as by its text, which makes it hard to fabricate later — and that is precisely what makes it credible.

Both feed forward:

- **M11 Checkpoint 4** asks for the decision log as a deliverable
- **M18** is a defence built on it: the architecture walkthrough, the ten what-if questions, and the operational handover all draw on entries you wrote months earlier
- **Your portfolio** is what the program's outcomes are actually measured by, and it is the artefact that outlives the program

There is also an individual dimension. In a group of three, the product is shared. Your posts are not — they are under your account, and they are the record of *your* contribution. Chapter 1 warned that the fastest person can quietly absorb all the work; a thin posting history is the earliest visible symptom of that, for you and for anyone looking.

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

**Daily beats good.** A two-minute post every day builds a record; a perfect post once a month does not. The habit is the deliverable, and the bar is "did I show one true thing today".

**Do not post only successes.** A profile of thirty wins reads as a highlight reel and is less convincing than twenty wins and ten honest problems. The failures are what demonstrate diagnosis, and diagnosis is the scarce skill.

**Write the decision at the end of the session, not later.** The reason is obvious while you are deciding and genuinely gone within a fortnight. The constraint that drove it — the free-tier limit, the number from your interviews — is the first thing to evaporate.

**Never rewrite a log entry.** If a decision changes, append a new entry that references the old one. An edited log is worthless as evidence, and the sequence of changing your mind is more impressive than appearing to have been right all along.

**Do not let the agent write the log for you.** It will produce plausible rationale, which is exactly the wrong thing — the point of the log is that the reasoning was yours. It is legitimately useful for the opposite task: give it an entry and ask what question an interviewer would ask next, or whether your REJECTED line is actually a reason or just a restatement.

**A caption that lists features is a weak caption.** "Added login, cart and checkout" is a changelog. "Tried storing the cart in memory, lost it on every redeploy, moved it to the database — here's the before and after" is a portfolio post.

### Verify it

Three checks on the log, three on the portfolio, once a week.

**Log:**
1. **Does every entry have a REJECTED line with a reason in it?** An entry without one does not prove you chose.
2. **Does every WHY contain a number or a named constraint?** "It was simpler" is not a reason; "3 changes a day, free tier has no realtime quota" is.
3. **Is it append-only, with commits spread across the weeks?** Ten entries committed on the same day in month five fool nobody.

**Portfolio:**
1. **Could someone who was not there follow what you did?** Read your last post as a stranger.
2. **Have you posted anything that went wrong this week?** If every post is a success, you are curating rather than recording.
3. **Is the AI conversation linked?** This is what separates your portfolio from someone who pasted output.

The real test, and the one M18 applies: **pick a decision from a month ago and explain it out loud in thirty seconds, including what you rejected.** If you cannot, the log either does not exist or is not specific enough.

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
