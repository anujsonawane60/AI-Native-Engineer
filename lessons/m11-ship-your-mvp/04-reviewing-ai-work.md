---
id: m11-04-reviewing-ai-work
module: m11-ship-your-mvp
slug: reviewing-ai-work
title: "Reviewing AI's Work: Diffs, Tests and Red Flags"
order: 4
duration_min: 35
is_free: false
mission: true
volatility: low
last_reviewed: 2026-11-16
---

## Key learnings
- Triage a large diff by risk class and decide what you will not read
- Spend most of your attention on the small fraction of files that can hurt you
- Judge a test by whether it fails when you break the feature
- Recognise the red flags that recur in agent-written code
- Write a review that names where you did not look, and why

## Lesson

### Where this sits in the calendar

**Read this in the middle of week three, immediately before Checkpoint 3.** By now your product has authentication, a schema and an AI feature arriving, and the changes coming out of your agent are no longer eighty lines in three files. Checkpoint 3 asks you to show tests and a security pass on exactly this kind of work, so this is the craft chapter that makes that gate passable.

M01.8 taught you to review a small diff: the four passes, the six questions, blocking versus non-blocking comments. All of that still holds and is assumed here. **What is new is scale.** M01.8's answer to a 1,400-line diff was "split it" — correct, and sometimes not available. A migration plus an endpoint plus a permission change plus their tests is genuinely one change, and it is 1,100 lines. The skill this week is deciding what *not* to read, and knowing where the risk has collected.

### The pain

A student receives a 1,900-line change across 31 files. It is well organised, with a clear description and fourteen passing tests. They review it properly by their own standards: three evenings, nearly every line read, six comments on naming and structure. They approve it.

Two weeks later a student account can read every other student's order history. The cause is eleven characters in one file: a permission check that compares the requested user id to a value taken from the request body instead of from the session. Our student read that line, on the second evening, among four hundred others, and it looked like a permission check, because it is one.

The failure was not inattention. It was **uniform attention** — spreading the same care over 1,900 lines, which guarantees that the eleven characters that matter get the same tiny slice as a CSS class rename. Three evenings of reading produced less safety than forty minutes in the right four files.

### The idea

**Risk in a large diff is not spread evenly, and your attention must not be either. Decide what you will not read, then spend almost everything you have on the files where a mistake is unrecoverable.**

Line count is not a risk signal. A 600-line new page is usually safe; a one-line change to an authorization check can expose every user's data. Review by **what the code can do**, not how much of it there is.

### The analogy

The canteen does a monthly stock audit. Nobody counts every item — there are four hundred things in the storeroom and two hours to do it in. So you count the gas cylinders, the oil tins and the paneer exactly, because that is where the money and the spoilage are. You weigh the rice roughly, glance at the salt, and move on. If the oil is short by three litres you find out why; if the salt is short you do not care. An unreviewable diff gets the same treatment.

**Where the analogy breaks:** in a storeroom, value is obvious from the price tag, and the expensive items are also the big ones. In a diff the most dangerous change is often the smallest and looks exactly like the safe version of itself. One character — `>=` instead of `>`, the request body instead of the session — carries more risk than a thousand lines of markup. So you cannot triage by looking; you triage by asking what each file is *allowed to do*.

### Picture it

```
  ATTENTION BUDGET FOR A 1,900-LINE DIFF   (90 minutes)
  ┌────────────────────────────────────────────────────┐
  │ TIER A  ~4 files   read every line   55 min        │
  │   migrations · permission checks · money           │
  │   external API calls · config and secrets          │
  ├────────────────────────────────────────────────────┤
  │ TIER B  ~8 files   read with intent  25 min        │
  │   new dependencies · error handling · failure paths│
  │   the tests, and whether they can fail             │
  ├────────────────────────────────────────────────────┤
  │ TIER C  ~12 files  skim headings     10 min        │
  │   pages, components, copy, styling                 │
  ├────────────────────────────────────────────────────┤
  │ TIER D  ~7 files   do not read       0 min         │
  │   lockfiles, generated output, formatting only     │
  │   — but confirm that is all they are               │
  └────────────────────────────────────────────────────┘
```

Four files out of thirty-one get sixty per cent of the time, and seven get none after a glance to confirm they are what they claim.

### Sorting the files, before you read any code

Ten minutes with the file list, no code — from `git diff main --stat`.

**Tier A — read every line.** Money and prices. Anything deciding who may do what: sessions, roles, middleware, row-level rules. Anything changing stored data: migrations, schema files, new required columns. Anything calling an external service, including a model API. Configuration, environment and deployment settings. Anything that deletes.

**Tier B — read with intent.** Dependency manifests. Error handling and retries. The failure paths of the new feature. Caching and rate limits. And the tests, which get their own treatment below.

**Tier C — skim.** New pages, components, copy, styling. Read the top of each file and the names of what it exports, looking for a decision that should not be here: a price recalculated, a role inferred, a user id read from somewhere odd.

**Tier D — do not read.** Lockfiles, generated clients, snapshot files, machine-formatted whitespace. One check only: **is that genuinely all they contain?** A real change hidden in a 3,000-line lockfile is how a swapped dependency gets through.

If Tier A is more than about six files, stop reviewing and ask for the change to be split. That is no longer a scale problem; it is an unscoped brief, and M01.8's answer applies.

### Reviewing the tests

Fourteen passing tests is a claim, not evidence. Three questions settle it, faster than reading them line by line.

**1. Does it fail when I break the feature?** Change one meaningful line in the implementation — invert the permission check, drop the division by a hundred — run the tests, and see if anything goes red. If everything still passes, the tests are decorative. It takes two minutes and is the highest-value thing you can do with a suite you did not write.

**2. Does it assert behaviour or implementation?** "Calling this endpoint as a student returns 403" is behaviour and survives a rewrite. "The `checkRole` function is called once" is implementation: it passes forever, including after someone removes the thing it was protecting.

**3. Does anything test the failure states?** Agents write happy-path tests by default. If every test has valid input and a user of the right role, the suite measures that the code runs, not that it is correct.

And count the skips. A test marked skipped or pending is a failing test with better manners.

### The red flags that recur

These recur in agent-written code across tools and models. Learn them as a list; you will recognise them faster than you can reason about them.

| Red flag | Why it matters |
|---|---|
| A decision made only on the client | Prices, roles and totals must be re-decided on the server |
| An id taken from the request instead of the session | The caller chooses whose data they read |
| A caught error that logs nothing and returns success | The failure becomes invisible, and users see a false confirm |
| A permissive default | `role = 'admin'` when unset opens the door quietly |
| A new dependency you did not ask for | Unreviewed code, new licence, new supply chain |
| A new required column with no backfill | Every existing row becomes invalid |
| A disabled lint rule or a loosened type | The check that would have caught this, switched off |
| A retry loop with no limit or backoff | One slow service becomes your outage, and your bill |
| A model call with no timeout and no cost ceiling | M09.14's token economics, unbounded |

### Where it lives

This sits between M01.8 and M18. M01.8 was the small diff; this is the large one; M07.1 and M07.3 are the security content behind half the red-flag table; M06.10 is where the tests came from; M10.19's agent-ready repository exists so that linters and type checks catch the mechanical problems before a human spends attention on them. It feeds M11.7 directly: the technical demo at Checkpoint 4 asks you to trace one request through your own code and explain one thing you rejected in review.

**Each of you submits your own evidence.** In a group of three, the review you personally did is the clearest proof that you engaged with code you did not write.

### Words to own

| Term | Plain meaning | Example |
|---|---|---|
| Triage | Sorting by risk before reading | The four tiers |
| Risk class | What a file may affect | Money, permissions, data |
| Attention budget | Review time you have, spent unevenly | 90 minutes |
| Mutation check | Break the code, see if a test fails | Invert the role check |
| Behavioural test | Asserts the outcome a user sees | "A student gets 403" |
| Implementation test | Asserts how the code is written | "`checkRole` was called once" |
| Permissive default | A fallback that grants access | `role` defaults to admin |
| Supply chain | Every dependency your code pulls in | A new package nobody reviewed |

### Tradeoffs and gotchas

**Deciding not to read something is a decision you must record.** "I did not read the lockfile or the generated client" belongs in your review, not in your head. It names which risk was accepted and by whom, and it is honest in a way silent skimming is not.

**The agent is a biased reviewer of its own work.** Ask it to review what it just wrote and it will largely agree with itself. Two things do work: ask it to *explain* a change you do not understand, and ask a **fresh conversation**, given the brief and the diff but not the reasoning, to attack it. M11.3's handoff discipline applies — you want a reviewer without the author's memory.

**Big diffs make reviewers polite.** The larger the change, the more a reviewer comments on names and formatting instead of behaviour, because surface detail is all a tired reader can still see. If your comments are all about naming, you have stopped reviewing.

**Not every red flag is a defect.** A new dependency may be the right call; a permissive default may be unreachable. The flag means *ask*, not *reject*. "Why is this id coming from the body?" beats "this is wrong", and sometimes the answer is good.

### Verify it

Your review is finished when you can answer all five of these.

1. **Which files did I decide not to read, and why?** Write the list. If it includes anything in Tier A, you have not triaged; you have given up.
2. **Did I break the feature and watch a test go red?** If not, you counted the tests instead of checking them.
3. **For every Tier A file, can I say in one sentence what it now decides, and where that decision is enforced?** "The role is checked in the server middleware, from the session" is an answer. "There is a role check somewhere" is not.
4. **Did I run it, including one failure state?** Pass four of M01.8 still catches what reading cannot.
5. **Did I check the data-shape change against rows that already exist?** The one defect class that is cheap before merge and very expensive after.

The red flags in your own reviewing: comments only about naming; an approval given the same evening a 2,000-line diff arrived; "all tests pass" used as the reason; and a review where you cannot name a single thing you deliberately skipped. That last one means you read uniformly.

## Walkthrough
1. Produce a genuinely large change on a branch: brief the agent for a whole feature from M11.3, but this time let it run the full plan without stopping. You want something you did not watch being written.
2. Run `git diff main --stat` and save the output. Do not open a single file yet.
3. Sort every file in that list into Tier A, B, C or D on paper. Write the tier next to each filename and the reason in three words.
4. Count Tier A. If it is more than six files, stop here, record that finding, and ask for the change to be split — that result is worth more than the review would have been.
5. Set a ninety-minute timer and divide it the way the diagram divides it. Start with Tier A, every line, no skipping.
6. For each Tier A file, write one sentence saying what it now decides and where that decision is enforced. Anything you cannot write becomes a review comment.
7. Move to Tier B. Read the dependency manifest, every caught error, and the failure paths. Note anything from the red-flag table by name.
8. Do the mutation check: break one meaningful line in the implementation, run the tests, record whether anything went red, then restore the line.
9. Skim Tier C for misplaced decisions — a price recalculated in a component, a role inferred in the page, a user id read from a query string. Then glance at Tier D only to confirm it holds nothing but generated or formatted content, and write down that you did not read it.
10. Run the product on a preview URL, try one failure state, then write your review: comments labelled blocking or non-blocking, plus an explicit list of what you did not read and why.
11. Add one decision-log entry: the most serious thing the triage found, and whether you would have found it reading front to back.

## Talk to AI
```prompt
I have to review a large change that an AI agent wrote across many files, and I do not have time to read all of it. I want help triaging it, not approving it. Do not tell me the change looks fine, and do not rewrite any code.

The brief the agent was given, including its non-goals: [paste it]
The output of git diff main --stat: [paste the full file list with line counts]
The diff for the files I am about to name: [paste them, or the most important parts]
What the agent says it did: [paste its own summary]

Work through these in order:

1. TRIAGE. Sort every file in my stat output into four tiers and say why in one line each.
   Tier A, read every line: money or prices, anything deciding who may do what, anything changing the shape of stored data, anything calling an external service, configuration, secrets, deployment settings, anything that deletes.
   Tier B, read with intent: dependency manifests, error handling, retries, failure paths, caching, rate limits, tests.
   Tier C, skim: pages, components, copy, styling.
   Tier D, do not read: lockfiles, generated output, formatting-only changes.
   If Tier A has more than six files, say so plainly and tell me this change should be split.

2. CONCENTRATION. Name the three single lines in this diff where a mistake would be hardest to recover from, and say what the mistake would look like.

3. RED FLAGS. Go through this list and quote any instance you find: a decision about price, role or identity made only on the client; an id taken from the request instead of the session; a caught error that logs nothing or returns success; a permissive default; a dependency I did not ask for; a migration bundled into a feature; a new required column with no backfill; a disabled lint rule or loosened type; a retry with no limit; a model call with no timeout or cost ceiling; a hard-coded value that should be configuration; a leftover TODO or commented-out block.

4. DATA SHAPE. Did the shape of stored data change? What happens to rows that already exist? Is the change revertable on its own?

5. TESTS. For each test, tell me whether it asserts behaviour a user would notice or the way the code is written. Then name the one line of implementation I should break by hand to prove the suite can fail, and what should go red when I do.

6. SCOPE. Name everything in this diff that my brief did not ask for.

7. WHAT YOU SKIPPED. Tell me which parts of this diff you did not actually examine, and what I would have to check myself.

Then write me four review comments in the form symptom, expected versus actual, file and line, each labelled blocking or non-blocking.

Finally: what is the most likely defect in this change that none of the above would catch?
```

## Mission

**Objective.** One large agent-written change, triaged by risk, reviewed in a fixed time budget, with a mutation check run on its tests and an explicit record of what you chose not to read.

**Steps.**
1. **Generate it** — let the agent run a whole feature plan without stopping, so the diff is big and you did not watch it being written.
2. **Stat first** — `git diff main --stat`, saved, before you open any file.
3. **Triage** — every file into Tier A, B, C or D, with a three-word reason. Count Tier A.
4. **Budget** — ninety minutes, split as the diagram splits it. Tier A first, every line.
5. **Sentence per Tier A file** — what it decides, and where that decision is enforced.
6. **Mutate** — break one meaningful line, run the tests, record what went red, restore it.
7. **Run it** — on the preview URL, including one failure state.
8. **Review** — comments labelled blocking or non-blocking, plus a written list of what you did not read and why.
9. **Fix and merge** — address the blocking comments, reply to each, confirm main still works.

**Upload.**
- The brief the agent was given, including its non-goals
- A screenshot of `git diff main --stat` with your tier labels written on it
- Your Tier A sentences: what each file decides and where it is enforced
- The mutation check → a screenshot of tests red after you broke the feature, and green after restoring
- Your review comments, with blocking and non-blocking labels visible
- Your explicit "what I did not read, and why" list
- At least one real problem the triage found, and the fix that followed
- Your AI triage conversation → share link in `ai_chat_url`
- Reflection, 150 words max → whether you would have found that problem reading front to back, and one risk you consciously accepted

**Self-check.** Tick every box before you submit.
- [ ] I sorted the file list into tiers before opening any file
- [ ] My Tier A has six files or fewer, or I asked for the change to be split
- [ ] I read every line of every Tier A file
- [ ] I can say, per Tier A file, what it decides and where that is enforced
- [ ] I broke one implementation line and watched a test go red
- [ ] I counted the skipped and pending tests
- [ ] I checked whether any test asserts implementation rather than behaviour
- [ ] I checked the data-shape change against rows that already exist
- [ ] I searched the diff for every item in the red-flag table
- [ ] I confirmed the Tier D files contain only generated or formatted content
- [ ] I ran the product myself, including one failure state
- [ ] My written review names what I did not read and why
- [ ] My comments name symptoms with expected versus actual, not opinions
- [ ] This review is mine, done on code I did not write

**Stretch.** Open a fresh conversation with a different AI tool, give it only the brief and the diff — not the agent's reasoning and not your review — and ask it to attack the change. Write 100 words on what the outsider found that both you and the author missed, and why the author's own review did not find it.

## Stuck? Ask AI
```prompt
I have a diff with thirty-one changed files and ninety minutes. I do not know where to start. Ask me what each file is named and what it appears to do, then sort them by what they are allowed to affect: money, permissions, stored data, external calls, configuration, presentation, generated output. Tell me which ones I should read every line of and which I should not open at all, and give me a minute budget for each group.
```

```prompt
I have a test suite an agent wrote and all of it passes. Teach me the mutation check: tell me how to pick one line of implementation to break, what should go red when I do, and what it means if nothing does. Then look at these tests and tell me which of them assert behaviour a user would notice and which assert the way the code happens to be written. [paste the tests]
```

```prompt
Explain why spreading equal attention across a 2,000-line diff is less safe than reading four files closely, using a non-software example such as auditing a storeroom. Then explain why the most dangerous change in a diff is usually the smallest one, and give me three examples of one-line changes that would be catastrophic.
```

## Quiz

### Q1 · L4
A student reads nearly every line of a 1,900-line diff over three evenings and still misses an eleven-character authorization bug. What went wrong?
- (a) They should have read the diff in one sitting while it was fresh
- (b) They spread equal attention over everything, so the lines that mattered got a tiny share ✅
- (c) The agent should have written better tests
- (d) The diff should have been reviewed by two people
> Uniform attention guarantees that the one dangerous line gets the same slice as a class rename. Forty minutes in the right four files is safer than three evenings spread evenly.

### Q2 · L4
Which file belongs in Tier A, the read-every-line group?
- (a) A 600-line new page component
- (b) A lockfile with 3,000 changed lines
- (c) A four-line change to the server middleware that reads the session role ✅
- (d) A file where only indentation changed
> Tier is decided by what the code is allowed to do, not by how much of it changed. A permission decision is unrecoverable when wrong; a new page usually is not.

### Q3 · L4
Fourteen tests accompany the change and all pass. What is the fastest way to find out whether they are worth anything?
- (a) Read each test and check it is well written
- (b) Check the coverage percentage
- (c) Break one meaningful implementation line and see whether anything goes red ✅
- (d) Ask the agent whether its tests are thorough
> Passing tests only prove the code runs under conditions the author chose. The mutation check takes two minutes and is what turns a count of tests into evidence.

### Q4 · L3
Why should "what I did not read" be written into the review rather than left unsaid?
- (a) The platform requires it on mission submissions
- (b) It records which risk was accepted and by whom ✅
- (c) It shortens the review
- (d) It transfers responsibility to the author
> Silent skimming and deliberate triage look identical from outside. Naming the skipped files tells a teammate exactly what was and was not checked.

### Q5 · L5
You ask the agent that wrote a 1,900-line change to review it, and it reports no serious issues. What is the best next step?
- (a) Accept it — the agent has the most context on its own work
- (b) Ask the same conversation to try harder
- (c) Give the brief and the diff to a fresh conversation with no memory of the reasoning ✅
- (d) Skip review and rely on the passing tests
> An author is a biased reviewer, and an agent agrees with itself. A reviewer without the author's memory is the closest thing to a second pair of eyes you can get on short notice.

## Go deeper
- [Google's code review guide — the speed of reviews and why size matters](https://google.github.io/eng-practices/review/reviewer/speed.html)
- [Martin Fowler — the Test Pyramid](https://martinfowler.com/articles/practical-test-pyramid.html)
