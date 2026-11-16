---
id: m06-11-debugging
module: m06-cloud-linux-devops-testing
slug: debugging-a-failure
title: "Debugging: Reading a Stack Trace, Reproducing, Bisecting a Failure"
order: 11
duration_min: 35
is_free: false
mission: true
volatility: low
last_reviewed: 2026-11-16
---

## Key learnings
- Read a stack trace from the bottom up and find your own code in it
- Reproduce a bug reliably before attempting any fix
- Bisect a failure to halve the search space each time
- Give an AI the exact error text instead of a description of it
- Write a blocker in the four-part format so someone can help in one message

## Lesson

### The pain

A student's canteen app breaks. They tell their AI assistant "my app is broken, the order page does not work". They get a confident answer, apply it, and the app is still broken — differently. Four more rounds, two more unrelated changes, and now they are not sure the original problem still exists or whether they have three problems.

Then they post in the group chat: "anyone know why my app is not working?" Nobody can help — not because nobody knows, but because the message contains no information. No error text, no expected result, no account of what they already tried.

Meanwhile the error was on the screen the entire time. Line four of the stack trace named their own file and line number, and the message said `Cannot read properties of undefined (reading 'price')`. Together those say: something arrived without a price, at that line. Twelve minutes of guessing would have been four of thinking.

### The idea

**Debugging is not guessing cleverly. It is three moves in a fixed order: get the exact error text, make the failure happen on demand, then halve the search space until only the cause is left.**

Skipping move one is why conversations go in circles. Skipping move two is why fixes that seemed to work come back. Move three is a method, not a talent, and it turns "I have no idea" into "it is in these four lines".

### The analogy

Every student in the canteen says the dal tastes wrong today.

The bad cook adds salt, then sugar, then more dal. Three changes, no knowledge, and the pot is now wrong in a new way.

The good cook tastes the dal themselves — that is the exact error text rather than a report of it. Then they get it to happen on demand: a fresh bowl from the same pot tastes wrong every time, so the problem is in the pot and not in one student's plate. Then they halve the search space: taste the dal before salt was added. Fine, so the fault is after that step. Taste the oil. The oil is rancid. One ingredient, found in three tastes.

**Where the analogy breaks:** the cook can taste the intermediate states because the dal is physically in front of them. Software's intermediate states are invisible unless you make them visible — which is what logging a value, or a test that isolates one function, is for. Most of the skill in debugging is creating the taste you need, and this is where an AI genuinely helps: it is very good at telling you what to look at next.

### Picture it

```
  1. CAPTURE        2. REPRODUCE        3. BISECT
  exact text        make it happen      halve the space
  ──────────        on demand           ───────────────
  copy the          ───────────         works at commit 40 ✓
  whole trace       smallest steps      fails at commit 50 ✗
  + the time        that always              │
  + what you        break it            try 45 → ✓
    expected             │                    │
       │                 │              try 48 → ✗
       └────────┬────────┘                    │
                ▼                        try 46, 47 ...
         now you can ask                      ▼
         for help, or test                 one change
         a hypothesis                      one cause
```

Debugging runs in one direction: capture what the machine said, make the failure repeatable, then halve the search space until one change remains.

### Move one: read the error, all of it

A stack trace lists the function calls that were in progress when the failure happened, newest first. It looks like a wall of text and holds exactly three things you need.

```
TypeError: Cannot read properties of undefined (reading 'price')
    at calculateTotal (/app/src/lib/cart.ts:42:18)      ← your code
    at OrderSummary (/app/src/components/Order.tsx:17)  ← your code
    at renderWithHooks (/app/node_modules/react-dom/...)
    at beginWork (/app/node_modules/react-dom/...)
    at performUnitOfWork (/app/node_modules/react-dom/...)
```

The first line gives the type and message, and the lines below are the chain of calls, with the two naming your own files being the ones that matter.

**The message.** `Cannot read properties of undefined (reading 'price')` is specific. Something you expected to be an object was not there, and you asked it for `price`. Not "the app is broken".

**The first line that is your code.** Nearly all trace lines come from libraries. Scan down for the first path that is yours — here, `cart.ts:42`. Beginners read the top line, see a library name, and conclude the library is broken. It is not; your code called it wrongly.

**The bottom of the trace.** Reading upward from the bottom gives you the story in order, and for nested or asynchronous errors the "caused by" section down there is usually the real cause.

Three error types worth recognising by sight: a **TypeError** on `undefined` means something was missing, almost always from data; a **network error** with a status code means the other side answered, and the code says whose fault it is — 4xx yours, 5xx theirs, as M02 covered; and a **timeout** means nothing answered, which is a different hunt entirely.

### Move two: reproduce it

A bug you cannot reproduce cannot be fixed, only disturbed. You want steps that make it happen every time.

Start from the full path and remove steps until it stops failing. The smallest sequence that still breaks is your reproduction, and shortening it *is* diagnosis: if the bug only happens with an empty cart, you have learned more than from any amount of reading code.

Four questions resolve most "it works for me" cases. **Which user?** A different role, from M04, is the most common answer. **Which data?** An empty list, a very long dish name, a ₹0 total, Devanagari text. **Which device or network?** Slow 4G, a budget Android browser, a cold start from chapter 5. **Which environment?** If it only breaks when deployed, chapter 1's case sensitivity and chapter 8's run-time environment variables are the first two suspects.

Then do the thing that makes the fix permanent: **turn the reproduction into a failing test.** You already have the suite from chapter 10. A test that reproduces this bug is red now, green when fixed, and red again forever if it returns — one evening's pain converted into a permanent guard.

### Move three: bisect

When you cannot see the cause, stop looking and start halving.

**Bisect over time.** It worked on Monday and fails today, across thirty commits. Do not read thirty diffs — check the middle one. If it works there, the cause is in the later fifteen; if it fails, the earlier fifteen. Five checks find it among thirty. Git has a built-in command for this, and it is the concrete reason M01 insisted on small commits: bisecting three enormous ones tells you almost nothing.

**Bisect over the input.** The import fails on a 500-row file. Try 250, then 125, then 60. In seven steps you have the one row, and that row usually explains everything.

**Bisect over the stack.** Is the bad value wrong in the browser, the request, the server handler, or the database? Check the middle — look at what the server received. One check eliminates half the system, and chapter 4's `curl` does it in four seconds.

**Bisect over the code path.** Log the value halfway through. The question is never "what is wrong" but "is it still correct here".

### Giving AI something it can work with

This is the same four-part blocker format M01 gave you for asking a teammate.

**Intent.** What you were trying to do. **Expected.** What should have happened, specifically. **Actual.** The exact error text, copied, complete, with the timestamp. **Already tried.** What you have ruled out.

Four lines. "My order page is broken" versus a message carrying `TypeError: Cannot read properties of undefined (reading 'price') at calculateTotal (cart.ts:42)`, "expected ₹240 for one item", and "happens only when the cart is empty" — that is the difference between four rounds of guessing and one correct answer.

And never paraphrase an error. "Something about undefined" has thrown away the file, the line and the property.

### Where it lives

Debugging is where every other chapter in this module gets used. Chapter 2's logs hold the error text, chapter 4's commands are how you look, chapter 10's tests lock a fix in place, and chapter 12's CI output is a stack trace you did not have to reproduce. M07's error tracking collects traces from real users. M01's blocker format is this chapter's output.

### Words to own

| Term | Plain meaning | Example |
|---|---|---|
| Stack trace | The chain of calls when it failed | Newest at the top |
| Exception | An error that stopped normal flow | `TypeError` |
| `undefined` / null | Something expected was not there | The missing `price` |
| Reproduction | Steps that cause the bug every time | "Submit with empty cart" |
| Minimal repro | The shortest steps that still break it | Diagnosis in itself |
| Bisect | Halve the search space each step | 30 commits in 5 checks |
| Regression test | A test that stops a fixed bug returning | Red now, green after |
| Root cause | The thing that, changed, makes it stop | Not the symptom |
| Blocker format | Intent, expected, actual, already tried | How you ask for help |
| Works-on-my-machine | An environment difference, not a mystery | Case, env vars, versions |

### Tradeoffs and gotchas

**Change one thing at a time.** Three simultaneous changes tell you nothing about which one mattered. This is also why you commit before letting an agent start, from M01.

**The first plausible cause is usually not the cause.** Especially the one the AI offers first, which is the most common cause of that message in general, not in your code. Hold it as a hypothesis.

**Fixing the symptom guarantees a repeat.** Wrapping the crash in a check so the page stops breaking is sometimes the right emergency action, and it is not a fix. Whatever produced a missing `price` is still producing it.

**Do not let an agent "fix" a bug you have not reproduced.** The symptom moves, and you can no longer tell whether the original problem exists.

**Deployed-only bugs have five usual causes, in this order:** a missing environment variable; file name capitalisation from chapter 1; a dependency on your laptop that is not in the dependency file; a build-versus-run-time configuration difference; and a region or cold-start effect from chapters 5 and 7.

**Intermittent is a category, not a mystery.** These failures are almost always timing, ordering or shared state. The useful question is "what else is running at the same time".

### Verify it

AI is genuinely excellent at debugging *if* you give it evidence and hold it to it.

1. **Did you give it the exact text, or a description?** This is on you, and it is the most common cause of a useless answer. Paste the whole trace, the timestamp, and your expected result.
2. **Does its diagnosis point at a specific line of your code?** "There may be an issue with state initialisation" is not a diagnosis. Demand a file, a line, and a claim you can check.
3. **Ask for the read-only check before the fix.** "What one command or log line would confirm this?" An agent that cannot name the evidence is guessing.
4. **Does the fix address the cause or the symptom?** Ask directly. Both answers are sometimes acceptable; you must know which one you accepted.
5. **Ask for the regression test.** A fix with no test that was red before it can return silently. Chapter 10's rule applies: watch it fail first.

Red flags: a confident diagnosis with no reference to your actual error text; a fix that changes several files for a one-line symptom; deleting and reinstalling dependencies as the first step rather than the last; a `try`/`catch` around the failure with nothing logged, which hides the evidence permanently; and an agent that says "this should now work" without running anything.

## Walkthrough
1. Find a real failure in your own project — or break something deliberately, such as removing a field from the data your page expects.
2. Capture it properly. Copy the complete error text, the timestamp, and the URL or action that triggered it. Do not summarise, do not retype.
3. Find the first line of the stack trace that names one of your own files and write down the file and line number — that is your starting point, not the top line. Then read the trace from the bottom up once and narrate the story in one sentence.
4. Build the reproduction. Write the steps that make it happen, then delete steps until it stops failing. Keep the shortest sequence that always breaks.
5. Answer the four reproduction questions in writing: which user, which data, which device or network, which environment. Note which one turned out to matter.
6. Bisect over the stack with `curl -i`, per chapter 4: does the server return the bad value, or does the browser? One command, half the system eliminated.
7. If you still cannot see it, bisect over time. Find a commit where it worked, and check the middle commit between then and now. Repeat. Record how many checks it took.
8. Write the failing test from chapter 10 that reproduces the bug. Watch it go red. Only now ask AI for a fix.
9. Brief AI using the four-part blocker format with the exact text, and demand a specific file and line plus the read-only check that would confirm it. Apply only the fix you understand.
10. Watch the test go green. Then run the whole suite, because a fix that breaks two other things is not a fix.
11. Write the blocker in M01's four-part format in your team notes, even though you solved it. That note is what makes you fast the second time.
## Talk to AI
```prompt
CONTEXT:      I am a student on an AI-native engineering program. I direct AI
              rather than writing code. My team of three is building
              [describe your product in two sentences]. Stack: [language,
              framework, database], deployed on [platform]. I have a test
              suite I can run with [your command]. Our users are in India on
              budget Android phones and slow 4G. I have a real bug right now.

              INTENT:   I was trying to [what you were doing].
              EXPECTED: [exactly what should have happened, with a value].
              ACTUAL:   Here is the complete, unedited error text and the
                        time it happened:
                        [paste the whole stack trace, not a summary]
              ALREADY TRIED: [what you have done and what it ruled out].
              REPRODUCTION: [the shortest steps that make it happen every
                        time, or say honestly that it is intermittent].

GOAL:         Find the actual cause of this bug and fix it in a way that
              cannot silently come back.

REQUIREMENTS: 1. First, read the error text back to me and tell me what it
                 literally says - the type, the message, and which line of MY
                 code appears first in the trace
              2. Give me your top three hypotheses, ranked, each naming a
                 specific file and line in my project
              3. For each hypothesis, give me ONE read-only check that would
                 confirm or eliminate it, and tell me what result means what
              4. Do not propose any fix until I tell you the result of the
                 checks
              5. When you do propose a fix, tell me explicitly whether it
                 addresses the cause or only stops the error appearing
              6. Give me a test that reproduces this bug - red now, green
                 after the fix - and tell me how to confirm it is genuinely
                 red first

CONSTRAINTS:  Plain English, second-language reader. Change one thing at a
              time. Do not touch any file outside the ones you name in your
              hypothesis. Label every command READ-ONLY or CHANGES-THINGS. Do
              not suggest deleting dependencies and reinstalling unless we
              have eliminated everything else. Do not wrap the failure in a
              try/catch that swallows it.

NON-GOALS:    Do not refactor anything. Do not upgrade any dependency. Do not
              improve code you were not asked about. Do not fix other bugs
              you happen to notice - list them for me instead.

ACCEPTANCE:   - I can state the cause in one sentence, naming a file and line
              - I have run a read-only check whose result supports that cause
              - A test reproduces the bug and I have watched it fail
              - After the fix the test passes and the whole suite still passes
              - I can say whether I fixed the cause or the symptom

ASK:          Before anything else, tell me whether my error text and
              reproduction are sufficient to diagnose this, and if not,
              exactly what else you need from me and how to get it.
```

## Mission

**Objective.** Find, reproduce, bisect and fix one real bug in your own startup, and leave behind a regression test and a written blocker note that would let a stranger understand the whole episode.

**Steps.**
1. **Find** — take a real bug from your product. If you genuinely have none, have a teammate break something in a branch without telling you what.
2. **Capture** — copy the complete error text, timestamp, and the triggering action. Nothing paraphrased. Screenshot the raw trace.
3. **Read** — identify the error type, the message, and the first line of the trace that is your own code. Write down the file and line before forming any theory.
4. **Reproduce** — produce the shortest sequence of steps that fails every time. Answer the four questions in writing: which user, which data, which device or network, which environment. Say which one mattered.
5. **Bisect** — narrow the cause by halving. Over the stack with `curl`, over the input, or over commits. Record every step and how many it took. State the search space you started with and the one you ended with.
6. **Test first** — write the failing test that reproduces the bug. Screenshot it red before any fix exists.
7. **Direct** — brief AI in the four-part blocker format with the exact text. Demand three ranked hypotheses with a file and line each, and a read-only check per hypothesis, before any fix. Run the checks yourself and report the results.
8. **Fix and verify** — apply one change. Screenshot the test going green. Run the whole suite and confirm nothing else broke.
9. **Classify** — state in one sentence whether you fixed the cause or the symptom, and if the symptom, what remains.
10. **Document** — write the episode in M01's four-part blocker format and commit it to your repository. This becomes evidence in M11 and M18.

**Upload.**
- The raw error text and stack trace, unedited → screenshot
- Your minimal reproduction steps, and which of the four questions mattered → file or post caption
- Your bisection log: each step, the result, and how many steps it took → file or screenshot
- The failing test, red, before the fix → screenshot
- The AI conversation in blocker format, showing hypotheses and read-only checks before any fix → share link in `ai_chat_url`
- The test passing and the full suite passing after the fix → screenshots
- Your committed blocker note in the four-part format → in your repo
- Reflection, 150 words max → the first hypothesis you believed and why it was wrong, and one thing the AI got wrong

**Self-check.** Tick every box before you submit.
- [ ] I captured the complete error text, not a summary, with a timestamp
- [ ] I identified the first line of the trace that is my own code
- [ ] My reproduction is the shortest set of steps that fails every time
- [ ] I answered all four reproduction questions in writing
- [ ] My bisection log shows each step and the search space it eliminated
- [ ] I wrote the failing test and saw it red BEFORE any fix was applied
- [ ] I gave AI the exact error text, not a description of it
- [ ] I got three ranked hypotheses with a file and line, before any fix
- [ ] I ran at least one read-only check and reported its result
- [ ] I changed one thing, not several
- [ ] I can state whether I fixed the cause or the symptom
- [ ] The full suite passes, not only the new test
- [ ] My blocker note is committed and a stranger could follow it

**Stretch.** Take a bug your team has already "fixed" in a previous week and write the regression test for it now. If the test passes immediately, you have confirmed the fix. If it fails, the bug never left — which is a more useful finding than any new feature you could have built in the same half hour.

## Stuck? Ask AI
```prompt
Here is a complete stack trace: [paste it, unedited]. Do not diagnose anything yet. First, teach me to read it: tell me what the error type and message literally mean, which lines come from libraries and which from my own code, which line I should look at first and why, and what the order of the lines tells me about what happened. Then ask me three questions about what I was doing when it appeared.
```

```prompt
I have a bug that only happens sometimes and I cannot make it happen on demand. Here is what I know: [describe it, with any error text you have]. Help me build a reproduction: ask me about the user, the data, the device and network, and the environment, one question at a time, and after each answer tell me what that eliminates. Do not propose a fix - I only want a reliable reproduction.
```

```prompt
My app worked last week and fails now, and I do not know which change caused it. I have [number] commits since then. Explain how bisecting works and why it takes about five checks instead of thirty, then walk me through doing it with git on my repository, one step at a time, waiting for me to tell you whether each commit worked or failed. Label every command READ-ONLY or CHANGES-THINGS.
```

## Quiz

### Q1 · L3
In a stack trace where the top four lines are library paths and the fifth names your own file, which line do you look at first?
- (a) The top line, because it is where the error was thrown
- (b) The first line that names your own code ✅
- (c) The bottom line, because it is the original call
- (d) Any line containing the word `Error`
> The library did not break. Your code called it with something it did not expect, and the first line of your own code in the chain is where that happened. The top line usually names a library frame.

### Q2 · L4
Why must you reproduce a bug before letting an agent attempt a fix?
- (a) Agents refuse to work on unverified bugs
- (b) Without a reproduction you cannot tell whether a change fixed it, moved it, or did nothing ✅
- (c) Reproduction is required to write a stack trace
- (d) It makes the agent's diagnosis faster
> A fix applied to an unreproducible bug produces an unverifiable result. The symptom may shift, and after two such changes you can no longer tell whether the original problem still exists.

### Q3 · L4
Your app worked at commit 40 and fails at commit 50. What is the fastest way to find the breaking change?
- (a) Read all ten diffs in order
- (b) Revert to commit 40 and reapply changes one at a time
- (c) Test commit 45, then halve again based on the result ✅
- (d) Ask AI to review the ten commits and identify the problem
> Halving finds one culprit among ten in about four checks and among thirty in about five. Reading diffs is linear, reverting-and-reapplying is slower still, and an AI reviewing diffs is guessing without running anything.

### Q4 · L4
Which message gives an AI or a teammate what they need to actually help?
- (a) "My order page is broken, any ideas?"
- (b) "Getting an undefined error somewhere in the cart logic"
- (c) Intent, expected result with a value, the complete unedited error text with timestamp, and what you already ruled out ✅
- (d) A screenshot of the broken page
> This is M01's blocker format. The exact text carries the file, the line and the property name; the expected value makes the goal checkable; "already tried" stops the helper repeating your work.

### Q5 · L5
A fix makes the error message stop appearing. What single question determines whether you are finished?
- (a) Does the page load faster now?
- (b) Did the AI say it was fixed?
- (c) Is this addressing the cause, or only preventing the symptom from displaying — and is there a test that was red before it? ✅
- (d) Does the full test suite still pass?
> The suite passing is necessary and not sufficient: wrapping a crash in a silent check also makes suites pass. A fix is complete when you can name the cause and a test that failed before it now passes.

## Go deeper
- [Git bisect: finding the commit that broke it, from the Pro Git book](https://git-scm.com/docs/git-bisect)
- [How to report bugs effectively, by Simon Tatham](https://www.chiark.greenend.org.uk/~sgtatham/bugs.html)
