---
id: m18-03-live-debugging
module: m18-demo-day
slug: live-debugging-under-questioning
title: "Live Debugging Under Questioning"
order: 3
duration_min: 35
is_free: false
mission: true
volatility: low
last_reviewed: 2026-11-16
---

## Key learnings
- Run one debugging method out loud: reproduce, read, isolate, hypothesise, test, confirm
- Read the exact error before you change anything, and change one thing at a time
- Isolate the failing layer by splitting the problem in half instead of guessing
- Say "I do not know, and here is how I would find out" rather than bluffing
- Direct AI in front of a panel without handing it the problem

## Lesson

### What happens in this block

**You are given a defect in your own product and ten minutes, with your screen shared and people watching.** Sometimes it is a real issue from your own known-issues list. Sometimes a teammate has injected one into a branch you have not seen. You may use AI. You must talk the whole time.

This is the hardest chapter in the module and the one that decides the most, because it is the only part of a defence that cannot be prepared in advance. A story can be rehearsed. A diagram can be drawn the night before. Ten minutes inside a system you did not fully build, with a problem nobody warned you about, is where it becomes undeniable whether you understood what was built. Fixing it is not the pass condition. **Visible, sound reasoning is.**

### The pain

Three students, ten minutes each.

The first starts typing immediately. Within four minutes they have raised a timeout, added a null check, and bumped a library version, and they have redeployed twice. Something is now worse. Asked which change caused it, they cannot say, because three changes went out together. They never read the error text; it was on screen the entire time.

The second pastes their whole repository into an AI and types *my app is broken, fix it*. Ninety seconds later they have a change across nineteen files and they accept it. The panel asks what the error message said. They scroll back and read it for the first time: it was a 403 on one endpoint. Nineteen files were changed to address one refused request, and the student cannot say what any of it did. This is M11.4's lesson in reverse — accepting an unreviewable diff, under time pressure, on stage.

The third reproduces the problem once on the live URL, reads the error out loud — *403 Forbidden on POST /api/orders* — and says: *403, not 401, so the session exists and something refused the role. That is authorization, not authentication, so I will not look at the login page at all. The cheapest test is to do the same action as a staff account; if that succeeds, the bug is in the role, not the endpoint.* Four minutes later it is fixed, in one file, and everybody in the room watched the reasoning happen.

### The idea

**Debugging under questioning is one method run out loud: reproduce, read the exact error, isolate the layer, form one hypothesis, test it cheaply, confirm. The narration is not commentary on the work. It is the work being assessed.**

Nobody can see your reasoning. If you type in silence for ninety seconds, a competent diagnosis and a lucky guess look exactly the same from the outside, and a panel has to assume the second.

### The analogy

The canteen's main burner will not light at 1 pm. One cook swaps the cylinder, adjusts the regulator and cleans the jet all at once. The flame comes back. Nobody knows why, and it happens again on Thursday. Another cook turns on the second burner on the same line first. It lights. In ten seconds that one cheap test has cut the problem in half: the gas line is fine, so the fault is in this burner.

**Where the analogy breaks:** the cook has senses. Smell, hiss, flame colour — free, instant, continuous feedback from a system with about six parts. You have none of that. The error text, the log line and whatever you deliberately measure are your only inputs, across a dozen layers you cannot see. So reading replaces sensing, and narration replaces the fact that a cook working silently still visibly looks competent while you do not. A silent cook is obviously fine. A silent student is indistinguishable from a guesser.

### Picture it

```
  1 REPRODUCE   make it happen on demand, on the live URL
       │
  2 READ        the exact text: status code, message, file, line
       │
  3 ISOLATE     split in half: browser | server | data | outside
       │
  4 HYPOTHESIS  exactly one, stated as "X, because Y"
       │
  5 CHEAPEST    one reversible change or one query - narrated
       │
       ├── wrong ──▶ back to 3, with one layer ruled out
       ▼
  6 CONFIRM     re-run step 1's reproduction and watch it pass
```

Six moves in order, with the only loop going back to isolation rather than forward into another change.

Step 1 carries a hidden pass condition: **if you cannot reproduce it, say so and stop changing things.** "I cannot reproduce this; here is what I would collect — the request id from the error tracker, the user's role, and the time, and then I would search the logs for that window" is a complete answer. Guessing at a bug you cannot trigger is how the first student's ten minutes went.

Step 3 is the move students skip and the one that saves the most time. Splitting in half means one test that eliminates a whole layer: does the request even reach the server, does the row exist in the table, does the same call work from a terminal without the page. M06.11 called this bisecting a failure, and it is the same instinct as the second cook's second burner.

### Narrating, concretely

Keep cycling three sentences. *What I am looking at. What I think it means. What I will do next.* That is all narration is.

Three habits make it sound like engineering rather than talking:

**Say the exact text.** Read the status code and the message aloud. "403" and "500" lead to completely different next moves, and saying it out loud forces you to actually look.

**Say what you have ruled out.** "The row is in the table, so this is not the save" is worth more to a listener than any amount of describing what you are about to try. It shows the search space shrinking.

**Say when you are guessing.** "This is a guess, and the cheapest way to find out is…" is a strong sentence. It is honest, and it tells the panel you know the difference between a hypothesis and a fact.

### Using AI on stage

You are expected to. This whole program is about directing AI, and a defence where you refuse to use it proves nothing. The difference between the second student and a good one is entirely in the brief.

*Fix my app* hands over the problem and the judgement. What works instead is M00.8's brief, compressed: the exact error, the one hypothesis, the single file or function in scope, and an explicit non-goal. *Here is the 403 from POST /api/orders. My hypothesis is that the role on the session is undefined for accounts created before the October migration. Look only at the middleware and tell me whether that is possible. Do not change any file.*

Three rules under time pressure. **Paste the error, not the repository** — the whole repository invites a whole-repository answer. **Ask it to explain before you ask it to change** — an explanation you can check beats a diff you cannot. And **never accept a multi-file change in a ten-minute window**, because you cannot review it, which is exactly the failure M11.4 exists to prevent.

### Where it lives

The method is M06.11's: reproducing, reading a stack trace, bisecting a failure. The one-change rule is step 6 of M00.8's Director's Loop. Reverting rather than patching forward is M11.3. Not accepting unreviewable diffs is M11.4. Your inputs — structured logs, error tracking, traces — are M07.4 to M07.7, and the discipline of working an incident with people watching is M13.10. The route you travel is the trace you wrote in M18.1 section 4.

Forward, this is the longest single block of M18.6, and the only one you cannot rehearse your way through.

### Words to own

| Term | Plain meaning | Example |
|---|---|---|
| Reproduce | Make the failure happen on demand | "Order as a student, every time" |
| Exact error | The literal text, not your summary of it | `403 Forbidden on POST /api/orders` |
| Status code | The number that says which kind of refusal | 401 is login; 403 is permission |
| Stack trace | The list of calls that led to the error | Read it bottom-up for your own files |
| Bisect | One test that eliminates half the system | Call the endpoint without the page |
| Hypothesis | One checkable claim, "X because Y" | "Role is null for pre-migration rows" |
| One-change rule | Change one thing, then look | Not a timeout and a version together |
| Red herring | A real-looking signal that is not the cause | A console warning four months old |
| Narration | Saying what you see, think and will do | The thing being assessed |
| Honest unknown | Not knowing, plus how you would find out | "I would search logs for that request id" |

### Tradeoffs and gotchas

**Never put real users' data on the projector.** Your logs and your table views contain phone numbers, emails and order histories. On a shared screen in front of thirty people that is a disclosure, it is irreversible, and under India's DPDP framework it is your responsibility. Seed a test account before the session, filter your log view, and rehearse opening both — M07.5 said keep personal data out of logs for exactly this reason.

**Do not refactor on stage.** Rename nothing. Tidy nothing. The smallest change that answers the hypothesis, and nothing else.

**Do not deploy to production during a defence unless you are asked to.** A scratch branch and a preview URL. The one thing worse than an unfixed bug is a second bug you shipped live while being watched.

**"It worked yesterday" is not a hypothesis.** It is the beginning of one. What changed since yesterday is a question you can actually answer: your deploy history, your merged changes, a provider's status page.

**Time-box out loud.** At eight minutes, stop and say where you are: what you ruled out, what you still suspect, what you would do with another hour. Running out of time with a clear account of the search is a good outcome. Silent flailing at minute nine is not.

**Bluffing is the one unrecoverable move.** Say an invented thing confidently, have it collapse on one follow-up, and every answer you gave earlier becomes suspect — the panel has no way to tell which others were invented. "I do not know" costs one question. A bluff costs the whole defence.

### Verify it

You cannot practise this by reading. Five drills, each recorded.

1. **The blind bug.** A teammate breaks one thing on a branch you have not seen. Ten minutes, narrating, recorded. Do this at least three times with three different injected defects.
2. **Listen back and count the silence.** Any stretch of more than about fifteen seconds of silent typing is a failure, however good the fix was.
3. **Count your changes.** More than one change in flight at a time means you have lost the thread and cannot attribute the result.
4. **Check whether you read the error aloud before touching anything.** If not, you were pattern-matching rather than diagnosing.
5. **Do one drill you cannot win.** Have someone inject something genuinely beyond ten minutes, and practise ending with the honest unknown: what you ruled out, what you suspect, what you would collect next, how long it would take.

The red flags, all visible from the back of a room: silent typing; three changes at once; a redeploy before a diagnosis; pasting the repository into an AI; accepting a multi-file diff under time pressure; real user data on screen; "it worked yesterday" said twice; and refusing to say "I do not know".

## Walkthrough
1. Write your own reproduction template and keep it on screen during drills: what I did, what I expected, what happened, the exact text, and whether it happens every time.
2. Before any drill, prepare the room: a seeded test account, a log view filtered so no personal data appears, your error tracker open, and a scratch branch you are allowed to break.
3. Ask a teammate to inject exactly one defect into a branch you have not seen, and to write down what they broke in a sealed note. Good candidates: a role read from the request instead of the session, a timeout removed, an environment variable renamed, a required column added without a backfill.
4. Start a recording, open the live preview of that branch, and reproduce the failure once before touching anything. Read the exact error out loud, including the status code.
5. Isolate with one cheap test that eliminates a whole layer: call the endpoint without the page, look for the row in the table, or try the same action as a different role. Say out loud which layer you have just ruled out.
6. State one hypothesis in the form "X, because Y" and write it down. Then make the single cheapest reversible change or query that would prove it wrong.
7. If you are wrong, go back to isolation with one more layer ruled out. Do not add a second change on top of the first; revert it, as in M11.3.
8. Use AI exactly once in the drill, with a compressed brief: the exact error, your hypothesis, one file in scope, and "do not change any file". Read what it says and say out loud whether you believe it and why.
9. At the eight-minute mark, stop and summarise aloud: ruled out, still suspected, next step, and the time it would take. Do this even if you are close.
10. Confirm by re-running your original reproduction, then compare your diagnosis against your teammate's sealed note and write down where your reasoning went wrong.
11. Repeat the whole drill twice more with different defects, and once with a defect designed to be unfixable in ten minutes, so you practise the honest-unknown ending.

## Talk to AI
```prompt
I have to debug a problem in my own product live, in front of a panel, in ten minutes, while narrating my reasoning out loud. I want you to drill me, not to debug for me. Do not suggest a fix and do not write any code unless I explicitly ask in a later message.

MY SYSTEM
As-built architecture, boxes and labelled arrows: [paste your map]
My request trace for the core action, naming my own files in order: [paste it]
Tables and what each holds: [paste them]
Where my logs, error tracker and metrics live, and what each shows me: [describe them]
Known issues I already have on my list: [paste them]
The layers I can test independently: [for example - the page, the endpoint called directly, a database query, an external service]

RUN THIS AS A DRILL, one turn at a time. Do not reveal later parts early.

ROUND 1. Invent a realistic defect in MY system, drawn from my architecture above - something an agent plausibly wrote. Tell me ONLY what a user reports and the exact error text they would see. Nothing else.

Then wait for me. I will reply with my reasoning. After each of my replies, do exactly four things:
1. Tell me whether my next step would actually eliminate a layer, or whether it only looks like progress. Name the layer it eliminates, if any.
2. Tell me if I changed more than one thing, or proposed a change before stating a hypothesis.
3. If I asserted something I could not know from the evidence so far, quote it and say "that is a guess" - and tell me the cheapest test that would settle it.
4. Reply with only the output of the step I actually took. Do not volunteer the cause, do not hint, and do not reveal anything I did not test for.

When I either find it or run out of moves, score me on five things, quoting me: did I reproduce before changing anything; did I read the exact error; did I isolate by halving rather than guessing; did I keep to one hypothesis and one change; did I say what I had ruled out.

ROUND 2. Do it again with a different defect in a different layer - this time one that cannot be found in ten minutes with the tools I described. When I run out of time, judge my honest-unknown ending: did I name what I ruled out, what I still suspect, exactly what I would collect next, and how long it would take? Rewrite my ending as the strongest honest version of itself.

Finally: from my architecture, name the three defects I would be slowest to diagnose, and tell me what I would need to have in place beforehand to be fast at each.
```

## Mission

**Objective.** Three recorded ten-minute debugging drills on your own product — two with defects injected by a teammate into branches you have not seen, one deliberately unwinnable — each narrated out loud, with your hypothesis sequence written down.

**Steps.**
1. **Prepare the room** — seeded test account, log view with no personal data, error tracker open, a scratch branch.
2. **Get a blind defect** — a teammate breaks one thing and seals a note saying what.
3. **Reproduce first** — on the live preview, before touching anything, and read the exact error aloud.
4. **Isolate** — one cheap test that eliminates a layer; say which layer out loud.
5. **One hypothesis, one change** — written as "X, because Y", then the cheapest reversible test.
6. **Use AI once, properly** — exact error, hypothesis, one file in scope, no file changes.
7. **Time-box at eight minutes** — say ruled out, still suspected, next step, time needed.
8. **Confirm and compare** — re-run the reproduction, then open the sealed note.
9. **Repeat twice** — a second injected defect, and one designed to be unwinnable in ten minutes.

**Upload.**
- The three recordings, with your narration audible → links or attachments
- Your reproduction notes for each: what you did, expected, actual, exact text, every time or not
- Your hypothesis sequence per drill, in order, with the layer each test ruled out
- Your teammate's sealed notes, and where your diagnosis differed from the real cause
- The AI brief you used in each drill, and what you decided to believe or reject from its answer
- A screenshot showing your log view with personal data filtered out
- Your honest-unknown ending from the unwinnable drill, written out in full
- The diff of each fix, small enough to read on one screen
- Reflection, 150 words max → the longest silence in your recordings and what you were doing during it

**Self-check.** Tick every box before you submit.
- [ ] I reproduced each defect before changing anything
- [ ] I read the exact error, including the status code, out loud
- [ ] Every drill has a written hypothesis in the form "X, because Y"
- [ ] I never had more than one change in flight at a time
- [ ] Each of my isolation tests eliminated a whole layer, and I said which
- [ ] I said out loud, at least once per drill, what I had ruled out
- [ ] I said "this is a guess" whenever it was one
- [ ] My AI brief contained the exact error and a scope limit, never the whole repository
- [ ] I accepted no multi-file change during any drill
- [ ] No real user's personal data appeared on my screen at any point
- [ ] I did not deploy to production during any drill
- [ ] I time-boxed at eight minutes and summarised out loud
- [ ] My unwinnable drill ends with a complete honest unknown: ruled out, suspected, next step, time
- [ ] No stretch of silent typing in my recordings is longer than about fifteen seconds
- [ ] I compared my diagnosis against the sealed note and wrote down where I was wrong
- [ ] This submission is my own work; these are my drills on parts of the system I personally directed, and I have named which parts those are

**Stretch.** Have someone inject a defect into a part of the system you did **not** direct — a teammate's feature. Run the same drill and record it. Write 100 words on what was different, and on which single piece of knowledge you were missing. That gap is exactly what the panel will find if they pick that part of the product, and you have a few days to close it.

## Stuck? Ask AI
```prompt
Teach me to read an error properly instead of skipping to a fix. Here is an error from my own product: [paste the exact text, including the status code and any stack trace]. Do not tell me the cause. Instead, ask me what each part of that text means - the status code, the method and path, the file and line, the order of the calls - and correct me when I get one wrong. Then ask me which layer it rules out, and make me justify it.
```

```prompt
I keep debugging in silence and I need to narrate. Give me the three sentences to cycle through, then run a practice: describe a symptom in a web product, and every time I reply, tell me which of the three sentences I left out - what I am looking at, what I think it means, or what I will do next. Keep going for ten exchanges. Do not solve the problem for me.
```

```prompt
Explain the difference between bisecting a problem and guessing at it, using a non-software example such as finding which bulb in a string of lights has failed. Then ask me, for my own product, three tests that would each eliminate a whole layer - the page, the endpoint, the database, an outside service - and tell me which of my three actually eliminates anything.
```

## Quiz

### Q1 · L4
A student is given ten minutes to fix a defect, immediately changes a timeout, adds a null check and bumps a library version, and the product gets worse. What is the core mistake?
- (a) They should have used AI to make the changes faster
- (b) Three changes in flight means no result can be attributed to any of them ✅
- (c) They picked the wrong three changes
- (d) They should have deployed between each change
> The one-change rule exists so that the outcome carries information. With three changes out together, a fix and a new break are indistinguishable, and reverting cleanly is no longer possible.

### Q2 · L4
You reproduce the bug and see `403 Forbidden` rather than `401 Unauthorized`. What does that immediately tell you, and what should you not look at?
- (a) The session is missing, so examine the login flow
- (b) The session exists and something refused the role, so the login page is not where to look ✅
- (c) The server is down, so check the host's status page
- (d) The request never arrived, so check the network tab only
> 401 is "I do not know who you are"; 403 is "I know, and no". Reading the exact status code eliminates a whole layer in one second, which is the cheapest isolation available.

### Q3 · L5
Nine minutes in, you have not found the cause. What is the strongest way to end?
- (a) Make one more quick change in case it works
- (b) Apologise and say the system is too complex to explain
- (c) State what you ruled out, what you still suspect, exactly what you would collect next, and how long it would take ✅
- (d) Say it worked yesterday and offer to show a recording instead
> The block assesses visible reasoning, not the fix. A complete honest unknown is a strong result; a hopeful change at minute nine tells the panel the search was never structured.

### Q4 · L4
Which use of AI during a live debugging block is defensible?
- (a) Pasting the whole repository with "my app is broken, fix it"
- (b) Pasting the exact error and your one hypothesis, asking it to explain whether that is possible, and forbidding file changes ✅
- (c) Asking it to rewrite the failing feature from scratch
- (d) Accepting its nineteen-file change because tests pass
> A whole repository invites a whole-repository answer you cannot review in ten minutes. The error plus a hypothesis plus a scope limit is M00.8's brief compressed, and it keeps the judgement yours.

### Q5 · L3
Why does this block require you to talk continuously while you work?
- (a) Because silence makes the recording unusable
- (b) Because reasoning is invisible, so silent competence and lucky guessing look identical ✅
- (c) Because explaining aloud makes people type more accurately
- (d) Because the panel cannot read code quickly enough to follow
> The narration is what is being assessed. From the back of a room, ninety seconds of silent typing carries no evidence that anything was diagnosed rather than tried.

## Go deeper
- [Julia Evans — debugging zine and the practice of forming hypotheses](https://jvns.ca/blog/2019/06/23/a-few-debugging-resources/)
- [Google SRE Workbook — generic mitigations and working an incident under pressure](https://sre.google/workbook/generic-mitigations/)
