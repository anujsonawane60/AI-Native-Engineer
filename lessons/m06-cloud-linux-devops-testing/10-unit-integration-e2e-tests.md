---
id: m06-10-testing-levels
module: m06-cloud-linux-devops-testing
slug: unit-integration-e2e-tests
title: "Unit, Integration and End-to-End Tests: What Each Catches"
order: 10
duration_min: 35
is_free: false
mission: true
volatility: low
last_reviewed: 2026-11-16
---

## Key learnings
- Say what each of the three test levels catches that the others cannot
- Explain why a test suite is what lets an agent verify its own work
- Recognise a test that asserts nothing and passes forever
- Write one test case as a sentence, before any code exists
- Choose the handful of tests worth having when your time is the constraint

## Lesson

### The pain

A team's canteen app has worked for three weeks. An agent is asked to add a student discount. Twenty minutes later the discount works. Two days later a student pays ₹0 for a ₹240 order, because the discount was applied to the total *after* a quantity change, and the quantity path was never exercised again by anyone.

Nobody noticed, because the way they check their work is to open the app and click the thing they just changed. That checks one path. The product has forty.

Now the second half of the pain, which is why this chapter exists. The same team asks an agent to refactor their pricing code. The agent does it and reports "all changes complete and working as expected". It has no way to know that. It cannot open a browser, tap through a checkout and look. It changed text in files and inferred success. The team has no way to know either, short of forty manual clicks they will not do.

A test suite is what converts "I think it works" into "the machine checked, and here is the answer" — for you and for the agent.

### The idea

**A test is an executable sentence about what your product must do, and the three levels differ only in how much of the product each sentence covers.**

A **unit** test checks one small piece in isolation: given these inputs, this function returns this. An **integration** test checks that two or three real pieces work together: your code, your database, your query. An **end-to-end** test drives the real product the way a user does: open the page, tap, fill, submit, expect to see this.

They are not competing. Each catches a category of bug the others structurally cannot see.

### The analogy

The canteen checks food at three levels.

The cook tastes the dal from the pot. One item, fast, immediate — and it tells you nothing about whether the rice is cooked or whether the plate ever reaches the student. That is a unit test.

The thali is assembled and inspected: dal, rice, sabzi, roti, together on the tray, in the right amounts. Each item was fine alone; the check is whether the combination is right. That is an integration test.

Then a staff member actually queues, orders, pays, collects and eats, start to finish. Slow, awkward, done once a day — and the only check that would catch "the card machine is unplugged" or "the queue leads to the wrong counter". That is end-to-end.

No sensible kitchen does only one of the three. They taste constantly, inspect every tray, and walk the whole queue once a day — because the cost of each check and the class of problem it finds are both completely different.

**Where the analogy breaks:** a cook tasting dal uses judgement and notices things nobody specified, like a smell that is slightly wrong. A test checks exactly and only what it was told to check, forever. It will never notice that your page is unreadable in sunlight or that the Marathi text is mangled. Tests catch regressions in things you thought of; they do not catch the things you did not.

### Picture it

```
            how much it covers        speed     how many
  ┌──────────────────────────────┐
  │ E2E     browser → app → db   │   slow      a few
  │         ████████████████     │   10s+      3-5
  ├──────────────────────────────┤
  │ INTEGRATION   app → db       │   medium    some
  │               ██████         │   ~1s       10-30
  ├──────────────────────────────┤
  │ UNIT    one function         │   fast      many
  │         ██                   │   ~1ms      50+
  └──────────────────────────────┘
       ▲                                  ▲
       │ catches: wrong maths,            │ catches: the
       │ wrong rules, edge cases          │ button does
       │                                  │ nothing
```

The three levels trade coverage against speed: many fast unit tests check logic precisely, a few slow end-to-end tests check that the whole thing is actually usable, and integration tests sit between them.

### What each level catches that the others cannot

| Level | Catches | Cannot catch |
|---|---|---|
| Unit | Wrong arithmetic, wrong rules, unhandled edge cases, off-by-one | Anything about wiring, the database, or the UI |
| Integration | Wrong query, missing column, broken migration, bad API contract | Whether a human can complete the task |
| End-to-end | Button does nothing, broken route, login loop, blank page on mobile | Which of fifty functions is actually wrong |

Three concrete bugs make the distinction real.

**A discount that gives 110% off** is a unit bug. One function, wrong maths. A unit test with the input "₹240 order, 10% student discount, quantity changed to 3" would have caught it in a millisecond, and would still be catching it in March.

**A query that reads `price` from a column renamed to `price_paise` in a migration** is an integration bug. Every function is correct in isolation; the pairing with the real database is wrong. No unit test will see this, because a unit test does not use your database.

**A checkout button wired to a route that no longer exists** is an end-to-end bug. The maths is right, the query is right, and the product is unusable. Only a test that drives the real interface will find it.

### Why this is the chapter that makes agents trustworthy

This is the connection to make now and carry into M10.

An agent cannot see your product. It can read files, write files, and run commands. When it says "this works", it is predicting, not reporting — unless there is something it can *run* that answers the question. A test suite is exactly that: a command that returns a pass or fail, with the name of the failing case in plain text.

That changes the agent's whole loop. Instead of "change code, claim success", it becomes: change code, run the suite, read the failure, change again, run again, and stop when the suite is green. The agent verifies itself, and you review a result rather than a promise.

This is most of what "agent-ready repository" means in M10: one documented command that runs the tests, tests that fail honestly when the product is broken, and a clean git state from M01 so the diff is readable. It is also why chapter 12's CI matters — the same suite runs on every pull request, so nobody has to remember.

And it is why a test that asserts nothing is worse than no test. A suite that is always green teaches both you and the agent that everything is fine. You have replaced "I do not know" with a confident wrong answer, which is the one state worse than ignorance.

### Where it lives

Tests sit between your code and everything that depends on it being correct. M01's pull requests are where they run; chapter 12 makes that automatic. M04's migrations are the most common cause of an integration test failure, which is exactly what you want — the failure arrives in CI rather than in production. M05's load tests ask "how fast", not "is it right". M07's monitoring tells you something broke in production; tests tell you before it ships. M11's checkpoint asks for your suite, and M13 goes deep on test strategy.

### Words to own

| Term | Plain meaning | Example |
|---|---|---|
| Test case | One executable sentence about required behaviour | "₹0 order shows an error" |
| Assertion | The check that decides pass or fail | "expect total to be 21600" |
| Unit test | Tests one function in isolation | Discount maths |
| Integration test | Tests real pieces working together | Code plus real database |
| End-to-end test | Drives the real product like a user | Browser taps checkout |
| Test suite | All your tests, run by one command | `npm test` |
| Regression | Something that used to work and stopped | The discount bug |
| Fixture | Known starting data for a test | One user, three dishes |
| Mock | A fake stand-in for a real dependency | A fake payment gateway |
| Flaky test | Passes and fails without the code changing | Usually a timing bug |
| Coverage | Share of code some test touches | A hint, not a goal |
| Red, green | Failing, then passing | Write it red first |

### Tradeoffs and gotchas

**Write the test so it fails first.** A test you have never seen fail may be testing nothing. Thirty seconds of proof — break the code deliberately, watch the test go red, fix it, watch it go green — is the only thing that establishes the test is connected to reality. Do this every single time, and especially for tests an agent wrote.

**End-to-end tests are expensive and flaky; have few and choose them well.** They are slow, they break when the design changes, and they fail intermittently for timing reasons. Three to five, covering the paths that would end your demo — sign up, place an order, pay — are worth more than thirty that nobody trusts.

**Coverage percentage is a hint, not a goal.** It measures which lines ran, not whether anything was checked. A suite with 95% coverage and no meaningful assertions proves nothing. Chasing the number produces exactly those tests.

**Tests that duplicate the implementation are dead weight.** A test that re-does the same arithmetic as the function it tests will agree with any bug the function has. The useful test states the *expected answer* as a literal — `21600` — not a recalculation.

**Do not test the framework or the library.** You are testing your product's rules, not whether the HTTP library can make a request.

**Flaky tests destroy the suite's value fast.** Once a test fails randomly, people start ignoring red, and an ignored suite is worse than none. Fix it or delete it the same week.

**Your time is limited, so start absurdly small.** Five tests covering money, permissions and the main user path beat a plan for a hundred that never happens. Money, because errors there are not recoverable. Permissions, because M04's broken-authorisation bug is invisible from the outside. The main path, because without it you cannot demo.

### Verify it

Agents write tests readily, and a meaningful fraction of them check nothing. Five checks, and the first is non-negotiable.

1. **Break the code and confirm the test goes red.** Change a `+` to a `-`, change a returned number, delete a line. If the suite stays green, the test is decoration. This single habit catches the module's recurring red flag — tests that assert nothing — better than reading them does.
2. **Read every assertion and look for an expected value.** A test with no assertion, or one that asserts `true` is `true`, or that the result "is not null", is theatre. Real assertions contain a literal you can disagree with.
3. **Ask which level each test is, and what it catches that the others cannot.** If an agent produces eleven unit tests and nothing that touches the database or the interface, your two most dangerous categories are unguarded.
4. **Check the failure message.** Run the suite red on purpose and read what it prints. If a failure does not tell you which behaviour broke, it will not help you at 11 p.m. the night before a checkpoint.
5. **Make sure the test uses real inputs for your product.** Rupees in paise, as M04 insisted. A UPI reference. A dish name in Devanagari. Tests built on `foo` and `bar` pass while your product fails on real data.

Red flags: a test with no assertion; a test that mocks the exact thing it claims to test; `expect(result).toBeDefined()` as the only check on a money calculation; a suite that has never been seen to fail; a test that recalculates the expected value instead of stating it; and an agent reporting "all tests pass" without having shown you the command's output.

## Walkthrough
1. List the five things in your product that must never break. Be specific: "a student is never charged the wrong amount", not "payments work". Write them as sentences.
2. For each, write one test case as a plain English sentence in the form: given this situation, when this happens, the result must be this. Use real values from your product — rupees in paise, a real dish name.
3. Label each of the five with the cheapest level that could catch it: unit, integration or end-to-end. Most money and rules questions should land on unit.
4. Ask AI for the one command that runs the test suite in your stack, and put that command in your README. This is the command an agent will use in M10 and CI will use in chapter 12.
5. Brief AI to write just the first test — one test, from your sentence, with a literal expected value. Read it before running it.
6. Run it and watch it pass. Then deliberately break the code it tests, run it again, and watch it fail. If it does not fail, the test is worthless; have it rewritten and repeat.
7. Read the failure message out loud. If it does not name the broken behaviour, ask for a clearer assertion message.
8. Now add the integration test: one test that uses your real database with known starting data, and checks a query returns what you expect. Break the column name on purpose and watch it fail.
9. Add exactly one end-to-end test for the path that would end your demo — the main user journey, start to finish. Accept that it is slow.
10. Run the whole suite and record the total time. Under two minutes keeps it a habit; much longer and people stop running it.
11. Commit the tests and the README command together, and put the pass/fail output in your team notes as your baseline.

## Talk to AI
```prompt
CONTEXT:      I am a student on an AI-native engineering program. I direct AI
              rather than writing code, so I need to be able to READ and JUDGE
              tests, not write them from memory. My team of three is building
              [describe your product in two sentences]. Stack: [language,
              framework, database]. We have no tests at all today. Money is
              involved: [describe it - e.g. orders in rupees stored as paise,
              UPI payments]. We have roles: [e.g. student and canteen staff].
              Our users are in India on budget Android phones and slow 4G.
              An AI agent will be working in this repository, and I need a
              suite it can run to check its own work.

GOAL:         Get the smallest test suite that would genuinely protect this
              product, at the right levels, with assertions I can argue with.

REQUIREMENTS: 1. Explain unit, integration and end-to-end tests using MY
                 product as the example, and for each, name a specific bug in
                 my product that only that level could catch
              2. Propose at most eight test cases total, written first as
                 plain English sentences, prioritised, with the level each
                 belongs at and one line on why that level
              3. Include at least one test for money correctness and at least
                 one for permissions - a user of the wrong role being refused
              4. Write only the FIRST test as code, with every line
                 explained, and tell me exactly how to break the code so I can
                 watch the test fail
              5. Give me the single command that runs the whole suite, and
                 tell me where to document it so an agent and CI can find it
              6. Give me a five-point checklist for reviewing any test an AI
                 writes for me in future

CONSTRAINTS:  Free tier only, must run in a free browser dev environment.
              Plain English, second-language reader. Every assertion must
              contain a literal expected value I could disagree with - no
              "is defined" or "is not null" as the only check. Use real values
              from my product, not foo and bar. The whole suite must run in
              under two minutes. Do not install a paid or heavy tool.

NON-GOALS:    Do not aim for a coverage percentage. Do not write more than one
              end-to-end test. Do not test the framework or the libraries. Do
              not set up CI - that is a later chapter. Do not refactor my
              application code as part of this.

ACCEPTANCE:   - I can state, for each of my eight cases, which level it is and
                what it catches that the other levels cannot
              - Every assertion contains a literal value
              - I have broken the code on purpose and watched the first test
                go red, then fixed it and watched it go green
              - The failure message names the behaviour that broke
              - One documented command runs the suite, and it finishes in
                under two minutes
              - At least one test covers money and one covers a wrong-role
                user being refused

ASK:          First ask me what my product's five most important rules are and
              what my stack's standard test runner is, so nothing is guessed.
              Then propose the eight sentences and WAIT for me to approve or
              change them before writing any code.
```

## Mission

**Objective.** Your own startup has a running test suite of five to eight meaningful tests covering all three levels, invoked by one documented command, and you have proved every test fails when the thing it protects is broken.

**Steps.**
1. **Frame** — write the five things in your product that must never break, as sentences with real values. At least one about money and one about who is allowed to do what.
2. **Brief** — write a Director's Brief asking for a test plan as plain English sentences first, with the level for each and the specific bug each would catch. Require literal expected values. Demand no code until you approve the sentences.
3. **Direct** — have the suite built one test at a time. Unit tests first, then one integration test against your real database, then exactly one end-to-end test for the journey that would end your demo.
4. **Prove every test** — for each test, break the code it protects on purpose, run the suite, screenshot the red failure, restore the code, and screenshot green. A test you have not seen fail does not count towards your five.
5. **Document the command** — one command in your README that runs everything. Then have a teammate who did not build it run that command on their own machine from a fresh clone.
6. **Verify** — review the suite against this chapter's five checks. Record any test you deleted or rewrote because it asserted nothing.
7. **Connect to the agent loop** — give an agent a small change to make, instruct it to run the suite and fix its own failures before reporting back, and record what happened. This is the M10 habit, started now.
8. **Document** — one decision-log entry: which level you put most of your effort into, what you deliberately left untested, and why.

**Upload.**
- Your five "must never break" sentences, with real values → post caption or file
- Your test plan: each case, its level, and the bug it catches → file or screenshot
- The test code in your repository → the suite committed to your repo
- Red-then-green evidence for every test: paired screenshots of the deliberate break and the restore → screenshots
- Your README section with the single command → screenshot
- The AI conversation where you approved the sentences before any code → share link in `ai_chat_url`
- The agent run where it fixed its own failing test → screenshot of the transcript
- Reflection, 150 words max → one test that caught something you did not expect, and one test the AI wrote that asserted nothing

**Self-check.** Tick every box before you submit.
- [ ] I have between five and eight tests, and I can say the level of each
- [ ] At least one unit, one integration and one end-to-end test exist
- [ ] At least one test covers money, with the amount as a literal value
- [ ] At least one test proves a user of the wrong role is refused
- [ ] I have seen every single test fail, on purpose, and have the screenshot
- [ ] Every assertion contains a literal expected value, not "is defined"
- [ ] No test recalculates its own expected answer
- [ ] One documented command runs the suite, and it is in the README
- [ ] A teammate ran that command from a fresh clone without asking me anything
- [ ] The whole suite finishes in under two minutes
- [ ] An agent ran the suite, saw a failure, and fixed it before reporting to me
- [ ] My reflection names a specific test the AI wrote that checked nothing

**Stretch.** Add one test for a failure path your product will actually meet: a payment webhook that arrives twice, a submission on a dropped 4G connection, or an upload that fails halfway. The tests nobody writes are the ones for the bad day, and the bad day is the demo.

## Stuck? Ask AI
```prompt
Here is a test an AI wrote for my project: [paste it]. Do not improve it. Tell me exactly what it asserts, and whether it would still pass if the function it tests returned a completely wrong value. Then tell me the precise change I should make to the application code to watch this test fail, and what the failure message should say. If this test asserts nothing meaningful, say so plainly.
```

```prompt
I have limited time and no tests. My product is [describe it] and the things that must never break are [list three]. Tell me the five tests that would protect the most value, which level each belongs at, and - importantly - what you are deliberately leaving untested and what risk that accepts. Rank them so I can stop after three if I run out of time.
```

```prompt
Explain why a test suite is what lets an AI agent verify its own work, rather than just claiming success. Then tell me specifically what my repository needs so an agent can run my tests without asking me anything: what command, documented where, and what must be true about the environment and the test data. Ask me what is missing from my repo today.
```

## Quiz

### Q1 · L3
A migration renamed a column from `price` to `price_paise`, and a query still reads `price`. Which level of test catches this?
- (a) Unit, because the query is one function
- (b) Integration, because it needs your real code running against the real database ✅
- (c) End-to-end only, because the error appears in the browser
- (d) None — schema changes are not testable
> Every function is correct in isolation, so unit tests pass. The fault is in the pairing of code and schema, which is exactly what an integration test exercises. End-to-end would also catch it, far more slowly and less precisely.

### Q2 · L4
An agent writes a test whose only check is `expect(total).toBeDefined()` on a price calculation. Why is this worse than having no test?
- (a) It makes the suite slower to run
- (b) It will pass for any value, including a wrong one, and teaches you and the agent that the behaviour is verified ✅
- (c) It cannot run in CI
- (d) It duplicates an existing unit test
> An always-green test replaces honest uncertainty with a confident wrong answer. The agent then reports success on the strength of it, which is the one state worse than having no information.

### Q3 · L4
What is the single most reliable way to confirm a test is actually connected to the behaviour it claims to protect?
- (a) Read the assertion carefully
- (b) Check that coverage increased
- (c) Break the code on purpose and confirm the test turns red ✅
- (d) Ask the AI that wrote it to explain the assertion
> Reading helps and is not proof. A test you have never seen fail may be testing nothing at all. Thirty seconds of deliberate breakage is the only evidence that the test and the code are connected.

### Q4 · L3
Why does a test suite change what an AI agent can honestly claim about its own work?
- (a) It makes the agent write better code in the first place
- (b) It gives the agent a command it can run whose pass/fail result is evidence, instead of inferring success from the text it wrote ✅
- (c) It prevents the agent from editing files it should not touch
- (d) It replaces the need to read the diff
> An agent cannot open a browser and look. Without something runnable, "it works" is a prediction. With a suite, the agent changes code, runs it, reads a named failure and iterates — and you review a result.

### Q5 · L5
You have time for exactly five tests. Which selection is strongest for a student canteen ordering product?
- (a) Five unit tests covering every function in the pricing module
- (b) Five end-to-end tests covering the five main screens
- (c) Three unit tests on money and discount edge cases, one integration test on the order query, one end-to-end test on the order-and-pay journey ✅
- (d) Five integration tests covering every database query
> Concentrating on one level leaves whole categories unguarded: five unit tests cannot see a broken route, and five end-to-end tests are slow, flaky and cannot tell you which calculation is wrong. Money, the database pairing, and the demo path is the spread that protects the most value.

## Go deeper
- [Martin Fowler on the test pyramid, and its practical limits](https://martinfowler.com/articles/practical-test-pyramid.html)
- [Playwright: writing your first end-to-end test](https://playwright.dev/docs/writing-tests)
