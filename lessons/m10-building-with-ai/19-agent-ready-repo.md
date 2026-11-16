---
id: m10-19-agent-ready-repo
module: m10-building-with-ai
slug: agent-ready-repo
title: "Making a Repo Agent-Ready: Linters, Test Harnesses, Self-Verifying Diffs"
order: 19
duration_min: 35
is_free: false
mission: true
volatility: low
last_reviewed: 2026-11-16
---

## Key learnings
- Build a repository where an agent can discover it is wrong without you
- Give every check one short, fast, deterministic command the agent can run
- Demand three artefacts for done: the change, a test, and the passing output
- Prove a test is real by reverting the change and watching it fail
- Make verification unskippable with hooks and a CI gate, not with reminders

## Lesson

### The pain

A student asks an agent to add the sold-out toggle. Twenty minutes later it presents a four-hundred-line diff across nine files and the sentence: "Done. I have implemented the feature and everything works."

Nothing works. The dev server does not even start.

Ask why it said that and the answer is uncomfortable: nothing in that repository could have told it otherwise. There were no tests. There was no linter. There was no type checker. There was no command it could have run to find out it was wrong. Its only source of truth about whether the code worked was its own belief that it had written good code.

So the student becomes the entire verification mechanism, and the student cannot read four hundred lines.

The same agent, same model, on a different repository: one setup command, a type checker, a linter, forty tests and a seed script. It produces a hundred-and-twenty-line diff, having run the tests three times, fixed two failures itself, and the diff contains the test it wrote. The difference is not the model. It is the repository.

### The idea

**An agent-ready repository is one where the agent can find out it is wrong without you. Every check you can run, it can run — by one short command, quickly, deterministically. The measure of an agent-ready repository is how fast a wrong change becomes a failing command.**

This is the chapter that closes the loop on the whole programme. M00 taught you to direct. M01 made edits reversible and reviewable. M06 built the test suite. This chapter connects them so the machine checks the mechanical things and you spend your attention on what only a human can judge.

### The analogy

A kitchen with a weighing scale, a thermometer, a tasting spoon and a written checklist lets a new cook catch their own mistakes before the plate leaves the counter. A kitchen with none of those makes the customer the quality check, and you find out about the salt from a complaint.

**Where the analogy breaks:** a cook tastes and *judges*. A test suite only knows what somebody told it to check. A green suite means "none of the things we thought of are broken" — never "it works". So the suite replaces your attention on the mechanical, and it does not replace your review. M01's six questions still get asked by you.

### Picture it

```
  agent edits ──▶ runs: lint · typecheck · test · build
                          │
                  ┌───────┴────────┐
                  │                │
               FAILING          PASSING
                  │                │
            reads the error        │
            fixes it ──────────────┘
            (loop, max N)          │
                                   ▼
                  presents THREE artefacts:
                  1. the diff
                  2. the test that proves it
                  3. the command output, pasted

  you then check: revert the change, run the test,
  and confirm the test FAILS. now the test is real.
```

The loop on the left is the agent catching its own mistakes. The three artefacts on the right are what "done" means. The check at the bottom is yours and it takes thirty seconds.

### The six properties

**One command to set up.** `make setup`, or one script. If an agent cannot get the project running, every session begins with twenty minutes of flailing and guessing at package managers. Write it down as a command, not as prose in a README.

**One command each for lint, typecheck, test, build and run.** Short, memorable, identical for every teammate, and listed in your rules file from chapter eighteen. The agent should never have to work out how to run your tests.

**Fast, deterministic tests.** An agent will run the suite ten times in a session. A nine-minute suite does not get run; a thirty-second suite gets run constantly. And **flaky tests are worse than no tests**, because they teach the agent — correctly — that a red result can be ignored. Fix or delete every flaky test before adding anything.

**A type checker and a linter.** These catch the mechanical mistakes without any human or any test: the wrong type, the unused import, the missing case. They are the cheapest feedback in the whole loop and they run in seconds.

**Seed data and a way to observe behaviour.** A command that fills the database with a realistic menu, a sold-out dish, a long dish name, an order with a remainder. Compiling is not working. The agent needs to be able to look at the thing running.

**Verification that cannot be skipped.** Chapter eighteen's hooks: lint and format on every edit, the test suite before the agent may report completion. Plus M06's CI as the gate on the pull request, so a human who forgets is also covered.

### The self-verifying diff

This is the deliverable the chapter is named for, and it is a change in what you accept rather than a tool you install.

**Done means three artefacts, together:** the change, a test that proves the change does what was asked, and the pasted output of the command showing that test passing. A diff on its own is a claim. A diff with a passing test the agent also wrote is evidence.

With one caveat that matters enormously. An agent can write a test that passes for the wrong reason — asserting what the new code happens to do, or asserting nothing at all. So there is one check you perform, and it is the most valuable thirty seconds in this module:

**Revert the change. Run the test. Confirm it fails.** If the test passes without the fix, the test is decoration and you have learnt nothing. If it fails, the test genuinely encodes the behaviour you asked for, and you can trust the green.

Ask for this explicitly in your brief: "include a test that fails before your change and passes after, and show me both runs." A well-configured agent will do both runs for you, and you spot-check.

### Keeping secrets out, and the diff small

Two things from earlier chapters that belong in this setup.

The working folder must contain no real credentials — chapter seven's first ingredient. Placeholders locally, real values in your deploy platform, and `.env` in `.gitignore` with a committed `.env.example` so the agent knows which variables exist without knowing their values.

And the diff must stay small. M01's finding was that small pull requests get real reviews and large ones get rubber stamps, and that reviewability is the author's responsibility. With an agent you are the author. One concern per change, under roughly four hundred lines, with the non-goals in the brief.

### Where it lives

Everything here is files in your repository: a setup script, a task runner, a test directory, a linter config, a type-checker config, a seed script, a rules file, hooks, and a CI workflow. All reviewable in pull requests, all shared by the team, all surviving a change of harness.

M06 is the direct dependency — the test suite built there is the thing that makes a success predicate real in chapter fourteen, and CI is the gate. M00's permission split is what lets the agent run tests silently while never pushing. M01's pull-request review is where you still apply judgement. And M11's checkpoints will ask you to demonstrate exactly this.

### Words to own

| Term | Plain meaning | Example |
|---|---|---|
| Agent-ready | The agent can discover it is wrong unaided | Every check is one command |
| One-command setup | A single script that makes it runnable | `make setup` |
| Deterministic test | Same result every run | No clocks, no network, no order |
| Flaky test | Passes sometimes | Worse than no test |
| Type checker | Catches wrong types without running | Seconds of feedback |
| Seed data | Realistic rows to work against | Sold-out dish, long name |
| Self-verifying diff | Change plus test plus passing output | Three artefacts |
| Revert check | Undo the fix, confirm the test fails | Proves the test is real |
| Pre-commit hook | Checks run before a commit lands | Lint and format |
| CI gate | Pull request blocked until checks pass | M06 |

### Tradeoffs and gotchas

**Flaky tests are actively harmful.** One test that fails one run in five teaches both you and the agent to re-run until green. That habit then hides a real failure. Quarantine or delete them the day you notice.

**A slow suite is an unrun suite.** Put the fast checks — lint, types, unit tests — on the inner loop, and the slow ones on CI. Thirty seconds is a target worth engineering for.

**Tests the agent wrote can be vacuous.** `expect(true).toBe(true)`, or asserting the current output whatever it is. The revert check is the defence and it has to be a habit, not an intention.

**Do not let the agent delete or weaken tests to go green.** Chapter fourteen's story. State it as a non-goal, and make it structural: a hook or a CI rule that fails a pull request which deletes test files without explanation.

**Coverage is not correctness.** Ninety percent coverage with assertions that check nothing is worse than forty percent with real assertions, because it looks finished.

**Setup instructions in a README are not setup.** If the steps are prose, every agent and every new teammate will get them subtly wrong. A command either works or it does not.

**This has a one-time cost and it is the best-value day in the module.** Half a day of setup changes every agent session afterwards, for everyone on the team, for the rest of the project.

### Verify it

**Run the fresh-clone test, which is the real exam.** Clone your repository into an empty folder. Start an agent there. Give it nothing but the repository and one small task. Watch whether it can set up, run the checks, make the change, discover its own mistake and fix it. Everything it needed that was not in the repository is your gap list.

**Time your checks.** Write down the seconds for lint, typecheck and test. If the suite is over about two minutes, the agent will stop running it and so will you.

**Run the suite five times without changing anything.** Five identical results, or you have flakiness to fix before anything else in this chapter matters.

**Do the revert check on the agent's own test.** Undo the change, run the test, confirm it fails. Do this on every change until it is reflex.

**Ask for the output, not the summary.** "I ran the tests and they pass" is generated text. The pasted terminal output with the test count and the exit status is evidence. Insist on the second one.

**Check that a broken change is actually caught.** Deliberately introduce a bug — divide a price by a hundred — and confirm something goes red. If nothing does, your checks do not cover the thing you most care about.

Red flags: "Done, everything works" with no command output; a repository with no tests, no linter and no type checker; a test suite over a few minutes; any flaky test; setup that only exists as prose; a test that passes with the change reverted; test files deleted inside a feature diff; a real `.env` in the working folder; and a four-hundred-line diff for a one-line fix.

## Walkthrough
1. Clone your own repository into an empty folder and try to get it running using only what is committed. Write down every step you had to improvise — that list is the work.
2. Create one setup command that takes a fresh clone to a running application. Test it by deleting the folder and starting again.
3. Create one short command each for lint, typecheck, test, build and run. Use the same names you would use on any project so they are guessable.
4. Time each command and write the seconds down — anything over two minutes goes to CI rather than the inner loop — then run your test suite five times with no changes and fix or delete anything that is not identical every time.
5. Add a type checker and a linter if you do not have them, and get the existing code to pass. This is boring and it is the cheapest feedback you will ever buy.
6. Write a seed script that creates realistic awkward data: a sold-out dish, a very long name, a price needing rounding, an order with a remainder.
7. Add all the commands, plus the definition of done, to your rules file from chapter eighteen: the change, a test, and the pasted passing output.
8. Add hooks: format and lint after every edit, and the test suite before the agent may report completion. Confirm both fire by making a trivial edit.
9. Now direct one real change with a brief that demands all three artefacts and forbids modifying existing tests. Read the file list first, then the diff.
10. Do the revert check: undo the change, run the agent's new test, confirm it fails, then restore. Record what you found.
11. Finally, break something on purpose — divide a price by a hundred — and confirm a command goes red. Then wire the same commands into CI per M06 so a pull request cannot merge without them.

## Talk to AI
```prompt
Audit my repository for whether an agent can verify its own work in it, then give me the ordered plan to fix it.

MY STACK: [languages, framework, package manager, database]
HOW I CURRENTLY SET IT UP: [the real steps, including anything I do by hand]
MY EXISTING COMMANDS: [whatever exists for test, lint, typecheck, build, run - say "none" where there is none]
MY TESTS: [how many, how long they take, whether any are flaky]
MY LINTER AND TYPE CHECKER: [what exists, or none]
MY SEED DATA: [what exists, or none]
WHAT IS IN MY WORKING FOLDER: [list it, including any .env or credential files]
MY RULES FILE: [paste it, or say none]

Deliver all of this:

1. Score my repository against the six properties - one-command setup, one command per check, fast deterministic tests, type checker and linter, seed data, unskippable verification - and tell me which gap costs an agent the most time today.
2. Write the one-command setup for my stack, and tell me how to test that it works from a fresh clone.
3. Give me the exact short command for each check, named conventionally, plus the target time for each.
4. Tell me how to detect flaky tests in my suite and what to do with them, and explain why a flaky test is worse than no test for an agent specifically.
5. Design the seed data for my domain: the awkward rows that catch real bugs, not the happy path.
6. Write the rules-file section that defines DONE as three artefacts - the change, a test, the pasted passing output - including the exact wording I should use in briefs.
7. Write the hooks: what runs after every edit, what runs before the agent may claim completion, and what should NOT be a hook because it is too slow.
8. Write the CI gate per M06: which commands run on a pull request, and a rule that fails the build if test files were deleted.
9. Explain the revert check to me as a habit: exactly what I do, in what order, and what each outcome tells me.
10. Finally: name one thing an agent could do in my repository today that nothing would catch, and the single cheapest check that would catch it.
```

## Mission

**Objective.** Your own repository made agent-ready, proven by a fresh-clone test, plus one real change delivered as a self-verifying diff that you confirmed with a revert check.

**Steps.**
1. **Fresh clone** — clone into an empty folder, try to run it from what is committed only, and list everything you had to improvise.
2. **Setup** — write one command that takes a fresh clone to a running application. Prove it by deleting and redoing it.
3. **Commands** — one short command each for lint, typecheck, test, build, run. Record the seconds for each.
4. **Determinism** — run the suite five times unchanged. Fix or delete everything that is not identical.
5. **Mechanical checks** — add a linter and a type checker and make the existing code pass.
6. **Seed data** — a script creating the awkward rows: sold out, very long name, a price needing rounding, an order with a remainder.
7. **Rules and hooks** — put the commands and the three-artefact definition of done in your rules file. Add a format-and-lint hook on edit and a test hook before completion. Confirm both fire.
8. **Direct one change** — a real feature or fix, briefed to require the change, a test, and the pasted passing output, with modifying existing tests as a non-goal.
9. **Revert check** — undo the change, run the new test, confirm it fails, restore. Record it.
10. **Break it on purpose** — introduce a wrong price calculation and confirm a command goes red.
11. **Gate it** — wire the commands into CI per M06 so a pull request cannot merge red.

**Upload.**
- Your improvisation list from the first fresh-clone attempt
- Your setup command, and terminal output of it working from an empty folder
- A table of your five commands with measured seconds for each
- Proof of determinism: five identical suite runs → screenshot
- Your seed script, and a screenshot of the awkward data it creates
- Your rules-file section defining done as three artefacts
- Your hooks, plus evidence that each one fired
- The self-verifying diff: the change, the test, and the pasted passing output → `repo_url` for the pull request
- Your revert check: the test failing with the change undone → screenshot
- Your deliberate bug and the command that caught it → screenshot
- The CI run on your pull request, green → screenshot
- The AI conversation → share link in `ai_chat_url`
- Reflection, 150 words max → what the agent could discover for itself afterwards that it could not before, and one thing the AI got wrong

**Self-check.** Tick every box before you submit.
- [ ] A fresh clone becomes a running application with one command, and I proved it from an empty folder
- [ ] There is one short command each for lint, typecheck, test, build and run
- [ ] I measured and recorded the seconds for each command
- [ ] My test suite gave five identical results in five runs
- [ ] There are no known flaky tests, because I fixed or deleted them
- [ ] A linter and a type checker both run and both pass
- [ ] My seed data includes at least three awkward cases, not just the happy path
- [ ] My rules file lists every command and defines done as three artefacts
- [ ] A hook formats and lints after every edit, and I saw it fire
- [ ] My change came with a test and the pasted passing output, not a summary
- [ ] I reverted the change and confirmed the test failed
- [ ] No existing test was modified or deleted in my diff
- [ ] I introduced a deliberate bug and a command caught it
- [ ] CI blocks the pull request when the checks are red
- [ ] There is no real credential anywhere in the working folder

**Stretch.** Run the fresh-clone test again from scratch, giving an agent nothing but the repository and one sentence of task. Do not help it. Write 150 words on everything it still needed that is not in the repository, and add the top two to your setup.

## Stuck? Ask AI
```prompt
Explain why an agent says "done, everything works" when nothing works. Walk through what feedback it actually had available in a repository with no tests, no linter and no type checker. Then list, in order of value per hour of my effort, what I should add so it can find out it is wrong by itself.
```

```prompt
Explain the revert check: what I do, why it matters, and what each outcome tells me. Then show me two examples of a test that passes for the wrong reason - one that asserts nothing meaningful and one that just asserts whatever the new code happens to do - and tell me how to spot each by reading the test.
```

```prompt
Explain why a flaky test is worse than no test when an agent is working in my repository. Then tell me how to find flaky tests in a suite I did not write, the usual causes - clocks, network, shared state, test order - and what to do with each one.
```

## Quiz

### Q1 · L3
Why did the agent say "everything works" when nothing worked?
- (a) It was optimising for user satisfaction
- (b) Nothing in the repository could tell it otherwise ✅
- (c) The context window was exhausted
- (d) It needed a stronger system prompt
> With no tests, no linter and no type checker, its only source of truth was its own belief. Agent-readiness is about giving it a way to discover it is wrong.

### Q2 · L4
Why is a flaky test worse than no test at all, for an agent especially?
- (a) It slows the suite down
- (b) It teaches that a red result can be ignored and re-run ✅
- (c) It cannot be committed to version control
- (d) It confuses the type checker
> Once failures are sometimes meaningless, the correct response becomes re-running until green — and that habit then conceals a real failure from both of you.

### Q3 · L4
What are the three artefacts that make a diff self-verifying?
- (a) The diff, a summary, and a screenshot
- (b) The change, a test that proves it, and the pasted passing output ✅
- (c) The change, the rules file, and a pull request description
- (d) A plan, the change, and a code review
> A diff alone is a claim. A test the agent also wrote, plus the real command output rather than a summary of it, is evidence you can check in seconds.

### Q4 · L5
An agent gives you a change and a passing test. What single check proves the test is real?
- (a) Read the test and confirm it looks reasonable
- (b) Check the coverage percentage went up
- (c) Revert the change, run the test, and confirm it fails ✅
- (d) Ask the agent to explain the test
> A test that passes without the change tests nothing. The revert check is thirty seconds and it is the difference between evidence and decoration.

### Q5 · L4
Which single investment most improves every future agent session in a repository?
- (a) A longer rules file
- (b) One setup command plus fast deterministic checks the agent can run ✅
- (c) A more capable model
- (d) More MCP servers connected
> It is a one-time cost that changes every session for every teammate: the agent gets running immediately and can catch its own mechanical mistakes before you see them.

## Go deeper
- [Google Testing Blog: avoiding flaky tests](https://testing.googleblog.com/2016/05/flaky-tests-at-google-and-how-we.html)
- [Agent rules files: the AGENTS.md convention](https://agents.md/)
