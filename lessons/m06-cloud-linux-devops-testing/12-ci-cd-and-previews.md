---
id: m06-12-ci-cd-previews
module: m06-cloud-linux-devops-testing
slug: ci-cd-and-previews
title: "CI and CD: Checks on Every Push, Environments and Preview Deploys"
order: 12
duration_min: 15
is_free: false
mission: false
volatility: medium
last_reviewed: 2026-11-16
---

## Key learnings
- Say what continuous integration actually is, in one sentence with no jargon
- Explain why a preview URL per pull request changes how a team reviews work
- Name the three environments and what each one is allowed to touch
- Keep secrets out of a pipeline that is defined in a public repository
- Recognise a green pipeline that is checking nothing

## Lesson

### The pain

A team has the test suite from chapter 10. One teammate changes the discount logic, forgets to run it, and the suite stays red for nine days because nobody ran it.

The same week a teammate opens a pull request and the others cannot tell whether it works, because a diff shows what changed, not whether the page looks right on a phone. The review is "looks fine to me". Then someone runs a migration against the live database while testing and loses the day's orders. Nothing happens automatically, so everything depends on someone remembering.

### The idea

**Continuous integration means every push runs your checks automatically, and continuous delivery means a change that passes can be deployed without anyone doing it by hand.**

The machinery — pipelines, runners, workflow files — exists to make "the tests ran" a fact about your repository, not a claim. A CI run is a clean Linux machine living about ninety seconds: it clones your repository, installs dependencies, runs the command from chapter 10 and reports the exit code from chapter 2. Because it starts clean, CI is the honest answer to "works on my machine": it catches missing dependencies, uncommitted files and chapter 1's capitalisation trap.

### The analogy

The canteen has a rule: nothing leaves the kitchen without passing the counter check — weight, temperature, item count. It is a station every tray passes, not a mood. Before a new dish joins the menu, a portion is plated exactly as a student would get it and left for the cooks to try: not described, plated. That is a preview deploy. And there are three kitchens deliberately: the practice counter, the tasting table, and the serving counter students queue at — and the practice counter never serves anyone.

**Where the analogy breaks:** a human at the counter notices things nobody wrote down; a pipeline checks exactly the list you gave it. Green means "the things we thought of passed", which is narrower than "this is good".

### Picture it

```
  you push a branch
        │
        ▼
  ┌──────────────────────────────────┐
  │ CI: a fresh Linux machine        │
  │  1 checkout your code            │
  │  2 install dependencies          │
  │  3 run the test suite            │
  │  4 build it                      │
  └───────────┬──────────────────────┘
              │
       pass ──┴── fail ──▶ ✗ on the pull request
        │                   nobody merges
        ▼
  ┌─────────────────┐     ┌────────────────────┐
  │ PREVIEW deploy  │────▶│ a URL your team    │
  │ (this branch)   │     │ can actually open  │
  └─────────────────┘     └────────────────────┘
        │
     merge to main
        ▼
  ┌─────────────────┐
  │ CD: production  │
  └─────────────────┘
```

A push starts a fresh machine that runs your checks and, if they pass, publishes a preview URL for that branch; merging to the main branch is what reaches production.

### Where it lives

| Environment | Who uses it | Its data | May it touch real money? |
|---|---|---|---|
| Local / preview | You and your team | Fake, seeded | No — test keys only |
| Staging | The team, and your mentor | A copy, or realistic fake | No — test keys only |
| Production | Real users | Real, backed up | Yes |

Two rules: a test environment never holds real user data or a live payment key, and production migrations go through your pipeline, not a laptop.

A preview deploy — one live URL per pull request — is the most valuable automation here: your reviewer uses the change on their own phone, you test on a budget Android on 4G, you send the link to a real student instead of testing on production.

CI runs chapter 10's tests, builds chapter 8's image, spends chapter 7's build minutes, deploys to chapter 6's platform.

### Words to own

| Term | Plain meaning | Example |
|---|---|---|
| CI | Every push runs your checks automatically | Tests on each PR |
| CD | A passing change can deploy without hand steps | Merge deploys |
| Pipeline / workflow | The ordered list of steps CI runs | Install, test, build |
| Runner | The fresh machine CI runs on | Lives ~90 seconds |
| Job / step | One stage in the pipeline | "Run tests" |
| Required check | A check that must pass before merge | The guardrail |
| Preview deploy | A live URL per pull request | Review on a real phone |
| Environment | A separate copy of your app and its data | Staging, production |
| Pipeline secret | A credential the platform injects at run time | Never in the repo |
| Build minutes | Metered CI time, a free-tier limit | Chapter 7's meter |
| Artefact | A file a run produces and keeps | A build, a test report |
| Rollback | Returning production to the previous version | One click, ideally |

### Tradeoffs and gotchas

- **A green pipeline is only as good as its checks.** One that installs and builds with no tests is green on every broken change and looks just as reassuring.
- **CI must run on pull requests, and a red pull request must not be mergeable.** Without that rule you have a pipeline; with it, a guardrail.
- **Never put a secret in the workflow file,** which is in your repository and often public. Secrets live in the platform's secret store, injected at run time.
- **Keep the pipeline under a few minutes,** because a ten-minute one gets bypassed; a workflow that triggers itself can eat a month's build minutes in a weekend.
- **CD on every merge needs a rollback you have practised,** because a mistake reaches users in ninety seconds. Nor should the pipeline be the only place tests run.

### Verify it

1. **Break something and watch it go red.** Push a branch with a failing test and confirm you cannot merge. If you can, it is all advisory.
2. **Search the workflow file for secrets.** Any token written in it is published; check also that logs do not print environment variables.
3. **Ask which environment each step touches.** Does anything reach the production database or a live payment key? On a pull-request pipeline, no.
4. **Check the build-minute allowance on the provider's usage page today;** a remembered figure may be a year old.

Red flags: a pipeline with no test step; a credential in the workflow file; a preview environment pointed at the production database; a workflow triggering on every branch; "all checks passing" where the only check is that it compiles.

## Walkthrough
1. Confirm you have the one command from chapter 10 that runs your test suite, and that it works from a fresh clone. CI can only run what already works locally.
2. Ask AI for a minimal CI configuration for your stack that does exactly four things: checkout, install, run the suite, build. Nothing else yet.
3. Read the configuration before committing it and check two things: is any secret written in it, and does any step touch a real database or live payment key.
4. Commit it on a branch and open a pull request. Watch the run start, and read the log from the top — the first failure is almost always a missing dependency or chapter 1's capitalisation.
5. When it is green, prove it is real. Push a deliberately failing test to the same branch and confirm the red cross appears on the pull request.
6. In your repository settings, make the check required so a red pull request cannot be merged. Then try to merge the red one and confirm you are blocked.
7. Remove the failing test, confirm green, and merge. You now have a guardrail rather than a pipeline.
8. Turn on preview deploys on your platform if they are not already on, and find the preview URL on your next pull request.
9. Open that preview URL on your actual phone, on mobile data rather than campus wi-fi. Write down one thing you found that you would not have seen on a laptop.
10. Find your platform's build-minutes usage page and today's allowance. Write both numbers, with the date, in your `STACK.md` from chapter 6.
11. Finally, find the rollback button for your production deploy, and have the whole team watch while you deploy a trivial change and roll it back. Knowing where it is before you need it is the point.

## Talk to AI
```prompt
CONTEXT:      I am a student on an AI-native engineering program. I direct AI
              rather than writing code. My team of three is building
              [describe your product in two sentences]. Stack: [language,
              framework, database]. The repository is hosted on [your git
              host] and deployed on [your platform], both on free tiers. We
              have a test suite that runs with [your command]. Right now
              nothing runs automatically, we have one database that is also
              our live one, and our reviews are "looks fine to me" because we
              cannot see each other's changes working. An AI agent also works
              in this repository.

GOAL:         Make it impossible to merge a change that breaks our tests, and
              give every pull request a URL a teammate can open on a phone.

REQUIREMENTS: 1. Explain continuous integration and continuous delivery in
                 plain words, and tell me which parts my platform already
                 does for me
              2. Give me a minimal pipeline configuration for my stack that
                 does only four things - checkout, install, run the suite,
                 build - with a comment on every line
              3. Tell me exactly where to turn on the setting that stops a red
                 pull request from being merged, because the pipeline is
                 worthless without it
              4. Explain the three environments - preview, staging,
                 production - and tell me precisely which database and which
                 payment keys each of mine should use
              5. Show me how a secret reaches the pipeline at run time, and
                 explain why writing it in the workflow file publishes it
              6. Tell me how to deliberately make the pipeline go red so I can
                 prove it is actually checking something

CONSTRAINTS:  Free tier only. Plain English, second-language reader. The
              pipeline must finish in under four minutes. Do not state any
              build-minute allowance as fact - tell me which usage page to
              open. No secret may appear in any file that is committed. The
              preview environment must never connect to our production
              database or a live payment key. Label every setting change as
              reversible or not.

NON-GOALS:    Do not add linting, formatting, code coverage gates, security
              scanning, matrix builds across versions, caching strategies, or
              deployment to multiple regions. Do not containerise anything
              that is not already containerised. Four steps, nothing more.

ACCEPTANCE:   - A pull request with a failing test shows a red cross and
                cannot be merged
              - A pull request with passing tests gives me a preview URL I can
                open on my phone
              - No credential appears in any committed file
              - I can state which database and which payment keys each of my
                three environments uses
              - The pipeline finishes in under four minutes
              - I know where my build-minute usage page is and today's number

ASK:          First ask me what my git host and platform are, and what my test
              command is, so nothing is guessed. Then tell me which single
              setting gives the biggest improvement for the least work, and
              why, before giving me any configuration file.
```

## Stuck? Ask AI
```prompt
Here is my CI configuration file: [paste it]. Do not improve it. Tell me exactly what it checks and, more importantly, what it does NOT check - list the kinds of breakage that would still produce a green tick. Then tell me the smallest change I could make to my code that would correctly turn this pipeline red, so I can prove it is actually working.
```

```prompt
Explain the three environments - preview, staging and production - and why mixing them up is dangerous, using a product that takes payments as the example. Then ask me, one at a time, which database connection and which payment key each of my environments currently uses, and tell me after each answer whether it is safe and what the worst case would be.
```

```prompt
My CI run failed and I do not understand the log. Here it is: [paste the whole log, not a summary]. Tell me which step failed, quote the exact line that shows the failure, and explain what that error means. Then tell me the three most likely causes given that CI runs on a clean Linux machine and my laptop does not, in order of probability.
```

## Quiz

### Q1 · L2
What does continuous integration actually do?
- (a) Deploys your app to production on every push
- (b) Runs your checks automatically on every push, on a clean machine ✅
- (c) Merges branches together without conflicts
- (d) Monitors your app for errors after it is live
> CI removes the requirement to remember. A fresh machine clones your code, installs dependencies, runs your nominated command and reports pass or fail. Deploying is the CD half; monitoring is M07.

### Q2 · L3
Why is a preview deploy per pull request worth more to a three-person student team than any other piece of automation in this chapter?
- (a) It reduces build minutes by avoiding production deploys
- (b) A reviewer can use the change on a real budget phone on 4G instead of imagining it from a diff ✅
- (c) It automatically runs the test suite twice
- (d) It prevents merge conflicts between teammates
> Reading a diff tells you what changed, not whether it works on the device your users have. A live URL per branch also removes the habit of testing on production because it is the only place the app exists.

### Q3 · L4
Your pipeline is green on every pull request, including one that broke the checkout page. What is the most likely explanation?
- (a) The runner cached an old result
- (b) The pipeline installs and builds but has no test step, so it is green on anything that compiles ✅
- (c) The required-check setting is disabled
- (d) The preview deploy succeeded, which marks the run green
> A pipeline that only proves the code compiles looks exactly as reassuring as one that proves the product works. The fix is the same as chapter 10's: break something deliberately and confirm it goes red.

### Q4 · L4
A workflow file in your public repository contains the database password so the pipeline can run migrations. What is wrong?
- (a) Nothing, provided the repository has few collaborators
- (b) The file is committed, so the credential is published to anyone who can read the repo or its history ✅
- (c) Workflow files cannot read passwords, only tokens
- (d) It will be overwritten on the next push
> Exactly chapter 8's Dockerfile rule and M01's git rule. Secrets belong in the platform's secret store, injected at run time, and a credential committed once is in the history permanently.

### Q5 · L5
You have CI running and green, and the team can still merge pull requests while the checks are red. What do you actually have?
- (a) Continuous integration, as intended
- (b) An advisory signal that depends on someone choosing to care — not a guardrail ✅
- (c) Continuous delivery without continuous integration
- (d) A correctly configured pipeline missing only preview deploys
> The branch rule is the mechanism. Without "this check must pass before merge", the pipeline is information, and information gets ignored at 1 a.m. before a checkpoint. The setting is the whole value.

## Go deeper
- [GitHub Docs: understanding workflows and required status checks](https://docs.github.com/en/actions/using-workflows/about-workflows)
- [Martin Fowler on continuous integration, the original practice](https://martinfowler.com/articles/continuousIntegration.html)
