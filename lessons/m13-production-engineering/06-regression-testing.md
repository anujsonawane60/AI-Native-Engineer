---
id: m13-06-regression-testing
module: m13-production-engineering
slug: regression-testing
title: "Regression Testing and Test-Data Management"
order: 6
duration_min: 30
is_free: false
mission: false
volatility: low
last_reviewed: 2026-11-16
---

## Key learnings
- Turn every bug you fix into a test that fails first and then stays forever
- Make a test deterministic by controlling time, randomness, ids and ordering
- Build test data from factories and anonymised shapes, never from real user rows
- Run a quarantine policy so one flaky test does not destroy trust in all of them
- Catch a test that was changed to match broken behaviour instead of fixing it

## Lesson

### The pain

M06 chapter 10 taught you that a test is an executable sentence, that unit, integration and end-to-end differ only in how much they cover, and — its strongest argument — that a test suite is what makes an agent trustworthy, because an agent cannot open a browser and look, so "it works" is a prediction unless something runnable returns pass or fail. It taught fixtures, mocks, flaky tests, coverage and the red-green habit. It promised M13 would go deep on test strategy. This is that chapter, and what is new is time: what happens to a suite after four months, two hundred tests, and a team that has started ignoring it.

Here is the shape of that failure. A team has 340 tests. Eleven of them fail roughly one run in four, for reasons nobody has investigated. So the habit becomes: see red, re-run it, and if it goes green, merge. Three weeks of that and the suite has become decoration. Then a real regression ships — the discount calculation from chapter 5 — and a test *did* catch it, in that same red run that somebody re-ran and merged anyway. The suite worked. The team had already stopped listening.

Second failure, more serious. To make the end-to-end tests realistic, someone copies a slice of the production database into the test environment. It contains two thousand real student phone numbers and their order history. The test environment has no authentication, because it is only tests. Under India's data protection rules that is a reportable problem, and the fact that it was well-intentioned changes nothing. M06 chapter 12 told you preview and staging environments never hold real user data; this is the chapter that tells you what to put there instead.

### The idea

**A regression suite is the written memory of every way your product has already been wrong, and it is only worth having if the team believes it.**

Belief is a technical property, not a cultural one. It comes from three things: every test is deterministic, so red always means something; every test is isolated, so order does not matter; and every flaky test is quarantined within days, so the failure rate of the suite itself stays at zero. The feature-adding tests from M06 prove the product works today. The regression tests prove it has not quietly gone back to being wrong.

### The analogy

M06 gave you the canteen's three levels of tasting: the cook tasting dal from the pot, the assembled thali inspected on the tray, and a staff member queuing and eating a whole meal.

Add the complaint book. Every complaint the canteen has ever received is written in it, and every morning before service, somebody walks the counters with that book and checks each line: the sambar was cold on the 14th — check the warmer; the raita was sour last month — smell it. The book only grows. Nothing is ever removed, because a complaint that has happened once will happen again.

Two things go wrong, and both will happen to you. The book grows until walking it takes longer than cooking, so people start skipping lines. And some lines are unreliable — "the chapati was hard" depends on the hour, so the checker marks it as a problem some days and not others, and after a fortnight everyone ignores that line and then starts ignoring the neighbouring ones.

**Where the analogy breaks:** a human checker notices things not in the book — an unlisted smell, a wrong colour. A regression suite catches only what someone encoded, forever, exactly. This is M06's point and it matters more at this scale: a suite of 400 tests can give you total confidence about 400 things and none at all about the 401st, which is why your suite's coverage of *money, permissions and data loss* matters far more than its percentage.

### Picture it

```
  BUG REPORTED              ┌───────────────────────────────┐
  "discount wrong" ───────▶ │ 1 reproduce it as a TEST      │
                            │    it must fail first (RED)   │
                            │ 2 fix the code                │
                            │    it passes (GREEN)          │
                            │ 3 the test stays forever      │
                            └──────────────┬────────────────┘
                                           ▼
                                REGRESSION SUITE (n+1)
                                           │
                            runs on every push ──▶ CI gate
                                           │
                        fails intermittently?
                                           ▼
                                  QUARANTINE LIST
                            (fix or delete within 1 week;
                             never just re-run and merge)
```

Every fixed bug becomes a test that failed before the fix, the suite grows by one, and any test that becomes unreliable leaves the gate immediately rather than being re-run until it passes.

### Where it lives

This builds on M06 chapter 10 and feeds three places. The CI gate is M06 chapter 12: a regression suite that is not a required check is a suggestion. Chapter 5's expand-contract migrations need regression tests at each step, because "the previous code still works against the new schema" is a claim that should be executable. And M10's agent-ready repository depends on this entirely: at this scale an agent cannot be trusted without a suite it can run and a suite the team believes, so your quarantine discipline is what makes autonomous work possible rather than reckless.

Test data touches M07 chapter 2's rules about secrets and M06 chapter 12's rule about environments. The short version: production data does not leave production.

### Words to own

| Term | Plain meaning | Example |
|---|---|---|
| Regression | Something that worked and stopped working | The discount bug returning |
| Regression test | A test written from a real bug | Added after every fix |
| Deterministic | Same input, same result, every time | No `now()`, no random |
| Isolated | Does not depend on other tests or their order | Fresh data per test |
| Fixture | A fixed starting state loaded before a test | Three dishes |
| Factory | Code that builds a test object on demand | `makeOrder(total: 100)` |
| Seed | A fixed number that makes randomness repeatable | `seed = 42` |
| Frozen clock | Time pinned to a fixed value during a test | `2026-01-15 13:05` |
| Flaky test | Passes and fails without the code changing | Quarantine it |
| Quarantine | Removed from the gate, with an owner and a deadline | One week |
| Golden / snapshot test | Compares output against a stored copy | An invoice PDF |
| Anonymised data | Production-shaped data with real values replaced | Fake phone numbers |
| Suite runtime budget | The longest you will let CI take | Under 10 minutes |
| Mutation testing | Breaks code on purpose to see if tests notice | Finds empty tests |

### Tradeoffs and gotchas

**Every bug gets a test, and the test must fail before the fix.** This is the whole chapter in one rule. A test written after the fix, which passes immediately, proves nothing — it may be asserting the wrong thing entirely. Watch it go red, then fix, then watch it go green. M06 said break the code and watch it fail; this is that habit applied to every bug for the rest of the product's life.

**The four sources of flakiness are time, randomness, order and the network, and all four are fixable.** Time: freeze the clock, because a test that uses today's date will fail on the first of a month or at midnight in a different timezone. Randomness: seed it. Order: never let one test depend on data another created — wrap each test in a transaction that rolls back, or rebuild the schema per run. Network: no test in the gate may call a real external service, because that makes someone else's outage your red build.

**Quarantine is a policy, not a feeling.** When a test flakes, it leaves the required-check list the same day, gets a named owner and a one-week deadline, and is either fixed or deleted. A deleted flaky test is better than a quarantined one that lives for six months, because the second one teaches the team that red is negotiable.

**Never copy production data into a test environment.** Build data three ways instead. Factories for most tests: small functions that construct exactly the object the test needs, so the test reads as its own documentation. A small shared fixture for the handful of things every test needs. And for realistic volume, an *anonymised* snapshot where names, phone numbers, emails and payment references are replaced with generated values — generated, not scrambled, because scrambled real data is often reversible. Under India's data protection rules, a student's phone number in your test database is personal data wherever it sits.

**Coverage is a liar in both directions.** 90% coverage with no assertion on the order total tells you nothing, and a test suite that merely executes code without checking the result is worse than no suite because it produces a green tick. Ask a different question: is there a test for every path that moves money, grants permission, or deletes data? Mutation testing is the honest check — break a line on purpose and see whether anything goes red. A line nothing complains about is a line nobody is testing.

**Snapshot tests rot.** A stored copy of an output is easy to write and very easy to update wrongly: when it fails, the fastest fix is to accept the new output, which silently accepts the bug too. Use them for things that genuinely should not change, read the diff every single time, and never let an agent regenerate snapshots as part of a fix.

**Keep a runtime budget and defend it.** A suite that takes forty minutes will be skipped. Run the fast tests on every push and the full suite before merge, run them in parallel, and treat a slow test as a defect with an owner. Ten minutes is a budget a small team can hold.

### Verify it

The specific thing AI gets wrong here is writing tests that assert *how* the code works rather than *what* it should do. Those tests pass, feel thorough, and break on every harmless refactor while catching no real bug.

1. **Make it show you the test failing first.** For any bug fix, demand the red run before the fix. If you are shown only a green run, you have no evidence the test tests anything.
2. **Read the assertion and ask what it would catch.** A test that checks a function was called is asserting implementation. A test that checks the total is 11,800 paise is asserting behaviour. Prefer the second always.
3. **Run the suite twice in a row, and then in reverse order.** Any test that changes result has a hidden dependency on time, order or shared data. This single check finds most flakiness before it reaches CI.
4. **Grep your test data for anything real.** Real phone numbers, real emails, real payment ids. One hit is a problem to fix today.
5. **Break one line of production code on purpose and run the suite.** If nothing goes red, the coverage number is fiction for that area. Do this for the code that calculates money.
6. **When a test changes in a diff, ask why.** The most dangerous pull request in this chapter is one where the test was edited to match the new behaviour. Behaviour changed deliberately, or the test was bent to fit a bug — and only you can tell which.

Red flags: a test suite where red is routinely re-run; a test written after the fix that was never seen failing; `new Date()` or an unseeded random inside a test; tests that only pass in one order; real user data in a test fixture; a coverage percentage quoted with no assertion on money; a snapshot regenerated as part of a bug fix; and an agent that "fixed the failing test" without changing any production code.

## Walkthrough
1. Count your tests, run the full suite, and write down the runtime in seconds. Then run it four more times and write down how many of those five runs were green. That ratio is your suite's real credibility.
2. List every test that failed in any of those runs. These are your flaky tests. For each one, guess which of the four causes — time, randomness, order, network — is responsible, then confirm.
3. Create a quarantine list in your repository. Move every flaky test out of the required-check list, give each a named owner and a date one week out, and commit it.
4. Fix the easiest flaky test by controlling its cause: freeze the clock, seed the randomness, isolate the data, or mock the network call. Run the suite five more times and confirm five greens.
5. Now run the suite in reverse order. Anything that newly fails has a hidden dependency on another test's data. Fix the isolation rather than the assertion.
6. Pick a real bug from your own product's history — one you actually shipped and fixed. Write the test that reproduces it, confirm it fails against the old code, then confirm it passes against the fixed code. That is your first deliberate regression test.
7. Audit your test data. Search every fixture and seed file for real phone numbers, real emails and real payment references. Replace each one with generated values and record how many you found.
8. Replace one large shared fixture with a factory: a small function that builds exactly the object one test needs, with the relevant value passed in. Re-read that test and note whether it is now self-explanatory.
9. Break one line of your money-handling code on purpose — change a plus to a minus, or drop a rounding step — and run the suite. If nothing goes red, write the missing test now. Repeat for your permission check.
10. Set a runtime budget, write it in the repository, and split your suite: fast tests on every push, full suite required before merge. Confirm both are wired as checks in your pipeline from M06 chapter 12.
11. Finally, write your team's three rules into the repository: every bug gets a test that failed first; no production data in any test environment; a flaky test is quarantined the same day. These three sentences are what an agent will read in M10 before it is trusted to work alone.

## Talk to AI
```prompt
CONTEXT:      I am a student on an AI-native engineering program. I direct AI
              and verify its output; I do not write code by hand. My team of
              three runs [product in two sentences] with [number] real users
              in India, handling [yes/no] real payments. Stack: [your stack],
              [your database]. Our test suite today: [number] tests, [number]
              seconds to run, and roughly [number] out of 5 runs are green
              without re-running anything. Our test data comes from [describe
              it honestly - fixtures, a copied database, whatever is true].
              Our CI from earlier in the course runs on every push. We are
              about to let an AI agent make changes with less supervision, so
              the suite has to be something we actually believe.

GOAL:         Turn our tests into a regression suite the team trusts: every
              test deterministic and isolated, flaky tests on a quarantine
              policy, test data with no real user information in it, and a
              runtime we will not start skipping.

REQUIREMENTS: 1. Give me a procedure to identify flaky tests from repeated
                 runs, and for each of the four causes - time, randomness,
                 test order, network - the specific fix and how I verify it
              2. Show me how to freeze the clock and seed randomness in my
                 stack, and tell me what a test using today's real date will
                 do on the 1st of a month and at midnight in another timezone
              3. Show me how to isolate tests so order never matters, and tell
                 me the tradeoff between a transaction rolled back per test
                 and rebuilding the schema per run, with timings
              4. Give me a test-data strategy with three layers: factories for
                 most tests, a small shared fixture, and an ANONYMISED
                 production-shaped dataset for volume. For the third, tell me
                 why generated values are safer than scrambled real ones
              5. Tell me exactly what counts as personal data in a test
                 database under India's data protection rules, and what I must
                 remove from my current test data
              6. Show me how to write a regression test from a real bug so
                 that it FAILS before the fix. Give me the order of steps and
                 tell me what I must see with my own eyes at each one
              7. Explain why a test that asserts a function was called is
                 weaker than one asserting an output value, with an example of
                 each for my code
              8. Give me a quarantine policy I can commit as a file: what
                 happens the day a test flakes, who owns it, and the deadline
              9. Tell me how to split my suite to hold a 10-minute budget, and
                 which tests belong in the fast set

CONSTRAINTS:  Free tier only. Plain English for a second-language reader. No
                test in the required-check set may call a real external
              service. Do not have me copy any production data anywhere. Label
              every command READ-ONLY or CHANGES-THINGS. My database free tier
              has a small connection limit, so tell me if a strategy needs
              many parallel connections.

NON-GOALS:    Do not rewrite my application code. Do not introduce a paid
              testing service or a test-management product. Do not aim for a
              coverage percentage - aim at specific risky paths. Do not add
              end-to-end tests for things a cheaper test can cover.

ACCEPTANCE:   - Five consecutive full runs are green with no re-runs
              - The suite passes in reverse order as well as forward
              - No real phone number, email or payment reference exists in any
                test data, and I have searched for them
              - I have one regression test I personally watched fail before
                the fix and pass after
              - Breaking one line of my money code makes something go red
              - A quarantine policy file is committed with owners and dates
              - The required suite runs inside my stated budget

ASK:          First, ask me to run my suite five times and report the results,
              and ask me to paste one existing test you think is weak. Then
              tell me which of my problems is most urgent and why, before
              changing anything.
```

## Stuck? Ask AI
```prompt
Here is a test from my suite: [paste it]. Do not rewrite it yet. Tell me what this test would actually catch if the code broke, and what it would miss. Then tell me whether it asserts behaviour or implementation, and show me the version that asserts behaviour instead. Finally, list everything in it that could make it flaky - anything to do with time, randomness, test order or the network - and how I would prove each one.
```

```prompt
Explain the four causes of flaky tests - time, randomness, test order, and network calls - and for each one give me a short example of a test that would fail intermittently because of it. Then ask me to diagnose six failing-sometimes tests, one at a time, without telling me the answers first. Finish by telling me why re-running a red build until it goes green is the most expensive habit in this chapter.
```

```prompt
I need realistic test data without using any real user information. Explain the difference between factories, fixtures and an anonymised production-shaped dataset, and when each is the right choice. Then tell me why replacing real phone numbers by scrambling them is unsafe compared to generating new ones, and give me a checklist of every field type in a typical order-and-payment database that counts as personal data and must be generated rather than copied.
```

## Quiz

### Q1 · L3
Why must a regression test fail before you apply the fix?
- (a) Failing tests are required by most CI systems before a merge
- (b) Otherwise you have no evidence the test actually detects the bug ✅
- (c) It establishes the correct coverage percentage
- (d) It proves the bug was reproducible in production
> A test written after the fix passes immediately and may be asserting something entirely unrelated. Watching red, then green, is the only proof the test is connected to the behaviour you care about.

### Q2 · L4
Eleven of your 340 tests fail about one run in four, and the team's habit is to re-run and merge. What has actually been lost?
- (a) Eleven tests' worth of coverage
- (b) Trust in all 340, because red no longer means anything ✅
- (c) The ability to run the suite in parallel
- (d) Nothing, as long as the eleven are known
> Flakiness is contagious to attention, not to code. Once red is negotiable, a real regression in a different test merges in the same re-run — which is why quarantine has a one-week deadline and a deletion option.

### Q3 · L4
A teammate copies a slice of the production database into the test environment to make end-to-end tests realistic. What is the problem?
- (a) The test suite will be too slow with production-sized data
- (b) Real personal data now sits in an environment with no authentication, which is a data-protection problem ✅
- (c) Production ids will collide with fixture ids
- (d) Nothing, provided the copy is deleted after each run
> Preview and staging environments never hold real user data, as M06 chapter 12 said. The fix is an anonymised production-*shaped* dataset with generated values — and generated, not scrambled, because scrambling is often reversible.

### Q4 · L4
A pull request fixes a bug, and the diff also changes an existing test's expected value. What should you check?
- (a) Whether the coverage percentage went up
- (b) Whether the test now runs faster
- (c) Whether the behaviour changed deliberately, or the test was bent to match a bug ✅
- (d) Whether the test was moved to the correct file
> This is the most dangerous diff in the chapter and only a human reading it can tell the two cases apart. An agent that "fixed the failing test" with no production-code change has almost certainly done the second one.

### Q5 · L5
Your coverage is 92% and a rounding bug in the order total ships anyway. What does that tell you to measure instead?
- (a) Branch coverage rather than line coverage
- (b) Whether every path that moves money, grants permission or deletes data has a test with a real assertion ✅
- (c) The ratio of unit tests to end-to-end tests
- (d) The suite's total runtime
> Coverage counts lines executed, not results checked, so a suite can execute the money code without asserting anything about the amount. Mutation testing is the honest version: break the line and see whether anything goes red.

## Go deeper
- [Google Testing Blog: flaky tests at scale and how to handle them](https://testing.googleblog.com/2016/05/flaky-tests-at-google-and-how-we.html)
- [Martin Fowler: test data builders, fixtures and the obscure test problem](https://martinfowler.com/bliki/ObscureTest.html)
