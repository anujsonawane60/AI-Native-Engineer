---
id: m06-11-debugging
module: m06-cloud-linux-devops-testing
slug: debugging-a-failure
title: "Debugging: Reading a Stack Trace, Reproducing, Bisecting a Failure"
order: 11
duration_min: 25
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

A student's canteen app breaks. They tell their AI "my app is broken, the order page does not work", get a confident answer, apply it, and the app is still broken — differently. Four rounds later they cannot tell whether the original problem still exists or whether they now have three. They post "anyone know why my app is not working?" in the group chat and nobody can help, because the message contains no information. The error was on the screen the whole time: `Cannot read properties of undefined (reading 'price')`, at their own file and line.

### The idea

**Debugging is not guessing cleverly. It is three moves in a fixed order: get the exact error text, make the failure happen on demand, then halve the search space until only the cause is left.**

**1. Capture.** Copy the error complete and unparaphrased — type, message, timestamp, and the first trace line naming one of *your* files. Read upward from the bottom for the story in order. A `TypeError` on `undefined` means data was missing, a status code means the other side answered (4xx yours, 5xx theirs), a timeout means nothing answered.

**2. Reproduce.** A bug you cannot reproduce cannot be fixed, only disturbed. Remove steps until it stops failing; the shortest sequence that still breaks *is* diagnosis. Four questions settle most "works for me" cases: which user, which data, which device, which environment. Then turn the reproduction into a failing test from chapter 10 — red now, green when fixed.

**3. Bisect.** When you cannot see the cause, start halving. Over time: thirty commits between working and broken, so check the middle one; five checks find it, which is why M01 insisted on small commits. Over the input: a 500-row import fails, so try 250, then 125. Over the stack: is the value wrong already in the request, the handler or the database? `curl` eliminates half the system in seconds.

### The analogy

Every student says the dal tastes wrong today. The bad cook adds salt, then sugar, then more dal: three changes, no knowledge, a pot wrong in a new way. The good cook tastes it themselves — the exact error text, not a report — then gets it to happen on demand: a fresh bowl from the same pot tastes wrong every time, so the fault is in the pot. Then they halve the space: the dal before the salt is fine; the oil is rancid.

**Where the analogy breaks:** the cook can taste intermediate states; the dal is in front of them. Software's intermediate states are invisible unless you make them visible, which is what logging a value is for. Most of the skill is creating the taste you need.

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

```
TypeError: Cannot read properties of undefined (reading 'price')
    at calculateTotal (/app/src/lib/cart.ts:42:18)      ← your code
    at OrderSummary (/app/src/components/Order.tsx:17)  ← your code
    at renderWithHooks (/app/node_modules/react-dom/...)
    at beginWork (/app/node_modules/react-dom/...)
    at performUnitOfWork (/app/node_modules/react-dom/...)
```

The first line gives the error type and message; the lines below are the chain of calls, and the two naming your own files are the ones that matter.

### Where it lives

Debugging is where the rest of the module gets used: chapter 2's logs hold the error text, chapter 4's commands are how you look, chapter 10's tests lock the fix in place, chapter 12's CI output is a trace you did not have to reproduce.

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

- **Change one thing at a time,** and commit before letting an agent start (M01).
- **The first plausible cause is usually not the cause** — especially the AI's first offer, which is the commonest cause of that message in general, not in yours.
- **Fixing the symptom guarantees a repeat,** and never let an agent fix a bug you have not reproduced: the symptom moves and the original is lost.
- **Deployed-only bugs have five usual causes, in order:** a missing environment variable; file name capitalisation from chapter 1; a dependency on your laptop but not in the dependency file; a build-versus-run-time difference; a cold start.

### Verify it

AI is excellent at debugging *if* you give it evidence and hold it to it.

1. **Exact text, not a description,** in M01's blocker format: intent, expected, actual, already tried. Paste the whole trace and the timestamp; never paraphrase.
2. **Does the diagnosis point at a specific line of your code?** "There may be an issue with state initialisation" is not one.
3. **Ask for the read-only check before the fix.** An agent that cannot name the evidence is guessing.
4. **Cause or symptom?** Both are sometimes acceptable; know which you accepted.
5. **Ask for the regression test,** and watch it fail first.

Red flags: a diagnosis that never quotes your error text; a fix touching several files for a one-line symptom; reinstalling dependencies first; a `try`/`catch` with nothing logged; "this should now work" without running anything.

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
