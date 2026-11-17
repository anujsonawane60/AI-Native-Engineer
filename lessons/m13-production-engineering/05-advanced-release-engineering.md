---
id: m13-05-advanced-release-engineering
module: m13-production-engineering
slug: advanced-release-engineering
title: "Advanced Release Engineering: Blue-Green, Canary, Feature Flags, Rollback"
order: 5
duration_min: 15
is_free: false
mission: false
volatility: low
last_reviewed: 2026-11-16
---

## Key learnings
- Choose between blue-green and canary from what your change actually risks
- Attach an automated rollback trigger to a measured threshold, not to a human
- Separate deploying code from releasing a feature, using a flag with a default
- Order a schema change so a rollback is still possible while it is half-done
- Catch a canary that nobody is watching and a rollback that has never been run

## Lesson

### The pain

M06 chapter 12 taught you CI and CD: every push runs your checks on a clean machine, a red pull request cannot merge, and a merge to main deploys. It mentioned rollback, then promised M13 would go deep into canaries, blue-green and feature flags. This chapter covers the gap between "we can deploy" and "we can deploy safely on a Friday night".

A team ships a change to the order total. It passes CI and deploys to everyone at once. The change has a rounding bug that only shows with a discount — one order in nine — and students are charged wrongly for thirty-eight minutes, until a user messages them. Behind a badly built canary the dashboard shows it in ninety seconds, to nobody.

### The idea

**A release strategy is worth exactly as much as its automated rollback trigger. Everything else is ceremony.**

Three mechanisms solve different problems. **Blue-green** runs two complete environments and moves all traffic at once, so rollback is moving it back. **Canary** sends a small share of real traffic to the new version, watches a measured signal, and promotes or reverts automatically. **Feature flags** separate deploying code from turning behaviour on, so a release needs no deploy.

### The analogy

M06 gave you the canteen's tasting table, where a new dish is plated for staff rather than described. Now it goes on the board, three ways. Build a second counter behind a curtain and move the sign across in one motion — **blue-green**: two counters at once, rollback in seconds. Or serve the new recipe to every twentieth student, stopping after two grimaces: a **canary**. Or cook the dish, keep it in the warmer, and never write it on the board — a **feature flag**, deployed but not released.

**Where the analogy breaks:** a grimace is instant, obvious and free. Your canary has no face. It has a p95 and an error rate, and someone must decide in advance which number means stop, then automate the stopping.

### Picture it

```
            ┌──────────────── 95% ───────▶ v7  (stable)
  traffic ──┤
            └──────────────── 5%  ───────▶ v8  (canary)
                                            │
                        every 30s, for 10 min (bake time):
                        ┌───────────────────┴──────────────┐
                        │ error rate  > 1%   ──▶ ROLLBACK  │
                        │ p95         > 800ms──▶ ROLLBACK  │
                        │ both inside budget ──▶ 5→25→100% │
                        └──────────────────────────────────┘
```

A canary splits traffic, compares the new version against written thresholds over a fixed bake time, and either promotes it in steps or reverts it without waiting for a human.

### Where it lives

This sits on M06 chapter 12's pipeline: CI decides whether a change *may* ship, this chapter decides *how*.

- **Thresholds** come from M05 chapter 8 — an SLO and an error budget make "too many errors" a number, not an opinion.
- **Signals** come from M07 chapter 6, which is why p95 and not the mean.
- **Chapter 3** splits traffic, **chapter 4** keeps that split reviewed rather than clicked, **chapter 10** starts where the rollback ends.

Free paths exist for all three: your platform's instant revert is blue-green you already own, traffic splitting is free on several platforms, and a flag is one database row.

### Words to own

| Term | Plain meaning | Example |
|---|---|---|
| Deploy | The new code is running somewhere | v8 is live |
| Release | Users can actually reach the new behaviour | Flag turned on |
| Blue-green | Two full environments; switch all traffic at once | Seconds to revert |
| Canary | A small share of real traffic on the new version | 5% |
| Bake time | How long you watch before promoting | 10 minutes |
| Promotion | Increasing the canary's traffic share | 5 → 25 → 100% |
| Rollback trigger | The measured condition that reverts automatically | Error rate > 1% |
| Feature flag | A switch that turns behaviour on without a deploy | Default off |
| Kill switch | A flag whose only job is to disable something fast | For a paid feature |
| Progressive delivery | Releasing gradually by percentage or cohort | Staff first, then all |
| Expand-contract | Add the new column, migrate, then remove the old | Safe schema change |
| Backward-compatible | Old code still works against the new database | Required for rollback |
| Flag debt | Flags left in the code long after the decision | Delete them |
| Forward fix | Fixing by deploying again rather than reverting | Slower; sometimes the only way |

### Tradeoffs and gotchas

**The hardest part is not the deploy, it is the database.** A rollback reverts your code, not your schema: if v8's migration dropped a column v7 reads, you can only fix forward under pressure. **Expand-contract** is the discipline: add the column, write to both, backfill, read the new, drop the old a release later.

**A canary needs enough traffic to mean anything.** At 5% of two hundred daily users it sees ten requests in ten minutes, which cannot tell a 1% error rate from zero. With small traffic, blue-green and an instant revert is the honest choice, though it costs double during the switch.

**Every flag needs a safe default, and it is off.** A flag defaulting to on fails open: when the config store is unreachable, the feature you were nervous about turns itself on for everybody. Each flag also doubles the code paths chapter 6's suite must cover.

**Decide in advance whether you revert or fix forward.** Revert is fast and known; fix forward is right when the problem is data already written. "We can roll back" is a claim until you have timed one in production.

### Verify it

1. **Ask for the rollback trigger as a number, a window and an action.** "We monitor error rates" is not a trigger; "error rate above 1% over 60 seconds reverts automatically" is. A human in the trigger means no trigger.
2. **Ask what happens if the canary is healthy but nobody promotes it.** No automatic promotion and no timeout leaves two versions in production forever.
3. **Ask for every flag's default and what happens when the flag store is unreachable.** A flag that fails on is worse than no flag.
4. **Run the rollback yourself and time it,** then confirm the old version is serving by a visible difference, not a green tick.

Red flags: a canary with no automated trigger; thresholds on the mean, not p95; a rollback plan that never mentions the database; a flag defaulting to on.

## Walkthrough
1. Take the last five changes your team shipped and classify each: pure frontend, backend behaviour, schema change, or payment-related. Write next to each one what it would have cost you if it had been wrong for forty minutes, in rupees or in users.
2. Find your platform's instant-revert feature and read how it works. Confirm whether it keeps the previous build and how far back it keeps them. This is the blue-green you already own and most teams never check.
3. Do a rollback on purpose, now, on a quiet afternoon. Deploy a visibly different version, revert it, and time the whole thing with a stopwatch. Confirm by looking at the visible difference in a browser, not at the pipeline.
4. Write your rollback trigger as one sentence with a number, a window and an action, using your SLO from M05 chapter 8 and your p95 from M07. If you do not have a p95 yet, get it before continuing — there is nothing to trigger on.
5. Do the canary arithmetic: your real requests per minute, times five per cent, times your bake time. Write the number of requests down and decide honestly whether a canary can detect anything at your traffic. Record that decision either way.
6. Build one feature flag with no product: a single row read at request time, default off, with a named owner and a removal date. Turn it on for your own account only, confirm the behaviour differs, and turn it off again.
7. Add a kill switch for the riskiest external dependency in your product — the payment provider, or an AI call. Confirm that with the switch off, the app degrades politely rather than erroring, which is M05 chapter 8's graceful degradation made operable.
8. Plan a real schema change the expand-contract way. Write the three releases out as separate steps and, for each one, state whether the previous code version still works against the new schema. If any step answers no, redesign it.
9. Execute the first step — the additive migration only — and confirm the currently deployed code still works untouched.
10. Write your release checklist into the repository: who may release, what must be green, the rollback trigger, the revert command, the stopwatch time from step 3, and the revert-versus-fix-forward rule. Chapter 10 will read this file during an incident.
11. Finally, pick an old flag — yours or one in an example codebase — and delete it, including both branches of the code it guarded. Note how much simpler the area becomes, and write down your team's rule for flag removal.

## Talk to AI
```prompt
CONTEXT:      I am a student on an AI-native engineering program. I direct AI
              and verify its output; I do not write code by hand. My team of
              three runs [product in two sentences] with [number] real users
              in India, handling [yes/no] real payments. Current deployment:
              [platform], merge to main deploys to all users at once. Our
              measured traffic is about [number] requests per minute at peak.
              Our p95 from monitoring is [number] ms and our error rate is
              [number]%. We have an SLO of [state it, or say we do not]. The
              change I want to release is [describe it], and it [does / does
              not] require a database schema change. We are three students
              with exams; nobody can sit watching a dashboard.

GOAL:         A release plan for this specific change with an AUTOMATED
              rollback trigger, a rollback I have tested, and a schema change
              ordered so that rolling back is still possible halfway through.

REQUIREMENTS: 1. Given my change and my traffic, recommend blue-green OR
                 canary OR a feature flag, and make the case against the other
                 two. Do the canary arithmetic for me: 5% of my request rate
                 over a bake time, as a number of requests, and tell me
                 honestly whether that can detect a 1% error rate
              2. Write my rollback trigger as ONE sentence containing a
                 metric, a threshold, a time window and an automatic action.
                 Use p95 rather than the mean and tell me why
              3. Tell me what happens if the canary is healthy and nobody
                 promotes it, and give me the timeout that prevents that
              4. For my schema change, give me the expand-contract sequence as
                 separate releases, and for EACH step state whether the
                 previously deployed code still works against the new schema
              5. A feature flag with no third-party product: how to store it,
                 how it is read at request time, its default, what happens if
                 the store is unreachable, its owner and its removal date
              6. A kill switch for my riskiest external dependency, and what
                 the user sees when it is off
              7. The exact revert command or steps, labelled, plus what I must
                 check in a browser to confirm the old version is really
                 serving
              8. A written rule for when we revert versus when we fix forward

CONSTRAINTS:  Free tier only - tell me clearly if blue-green doubles any cost
              and give the rupee figure. Plain English for a second-language
              reader. Label every command READ-ONLY or CHANGES-THINGS and name
              the point of no return. Assume nobody is watching a dashboard.
              Assume we have live users and real payments, so no step may risk
              a wrong charge.

NON-GOALS:    Do not recommend a feature-flag SaaS product or a service mesh.
              Do not redesign our pipeline. Do not propose a strategy that
              requires someone to be on call. Do not suggest we gather more
              traffic first - plan for the traffic I have.

ACCEPTANCE:   - My strategy choice follows from my measured traffic, and I can
                state the canary arithmetic
              - My rollback trigger is one sentence with a number, a window
                and an automatic action
              - I have run a rollback in production and timed it
              - Every step of my schema change leaves the previous code
                version working
              - My flag has a written default of off and a removal date
              - With the kill switch off, the app degrades politely
              - My release checklist is committed to the repository

ASK:          Before recommending anything, ask me what this change actually
              risks if it is wrong for forty minutes. Then make the honest
              case that my traffic is too small for a canary, if that is true.
              No configuration until I confirm the strategy.
```

## Stuck? Ask AI
```prompt
Explain blue-green, canary and feature flags as three answers to three different questions. For each, tell me what it protects against, what it does NOT protect against, and what it costs. Then give me six changes - a CSS tweak, a pricing calculation, a new database column, a third-party payment switch, an AI prompt change, and a background job rewrite - and ask me which strategy fits each, one at a time, without telling me the answers first.
```

```prompt
Explain the expand-contract pattern for database changes. Then walk me through a concrete case: I want to rename a column from `price` to `price_paise`. Give me the sequence of separate releases, and for every single step tell me explicitly whether the PREVIOUS code version still works against the new schema. Then tell me what goes wrong if I do it in one release and have to roll back halfway.
```

```prompt
I have a canary release at 5% traffic and I am about to say we are safe. Ask me the questions that would prove I am not: what exactly is measured, over what window, what threshold, what happens automatically when it is breached, who promotes it and when, what happens if nobody does, and whether my traffic volume makes any of those numbers meaningful. Do not accept vague answers - push back on each one.
```

## Quiz

### Q1 · L3
Your canary sends 5% of traffic to v8 and the dashboard clearly shows a doubled p95 within ninety seconds. Nobody notices for forty minutes. What was the defect in the release process?
- (a) The canary percentage was too low to be meaningful
- (b) There was no automated rollback trigger, so the canary only slowed the failure down ✅
- (c) p95 is the wrong metric for a canary
- (d) The bake time was too short to be conclusive
> A canary's value is entirely in the automatic action. With a human in the loop it is a rota, and three students with exams are not a rota. The trigger must be a metric, a threshold, a window and an automatic revert.

### Q2 · L4
Release v8 includes a migration that drops a column v7 still reads. Deploy succeeds, then errors appear. What is your situation?
- (a) Roll back the code; the schema is unaffected by code rollbacks
- (b) Roll back the code and the migration together, which most tools do atomically
- (c) You cannot roll back — the old code no longer works against the new schema, so you must fix forward under pressure ✅
- (d) Restore last night's backup, which is the standard remedy
> A code rollback does not revert a schema. This is why expand-contract exists: add first, dual-write, backfill, switch reads, and remove the old column in a much later release, so every single step is individually reversible.

### Q3 · L3
A feature flag's value is read from a config store at request time. The store becomes unreachable. What must happen?
- (a) The request fails, so nobody sees an inconsistent state
- (b) The flag falls back to its written default, which must be off ✅
- (c) The flag keeps its last known value indefinitely
- (d) The flag turns on, since the feature is already deployed
> A flag that fails on turns the feature you were nervous about on for everybody at the worst possible moment. The default belongs in the code next to the flag, and for a new feature it is off.

### Q4 · L4
Your product serves 40 requests per minute at peak. Someone proposes a 5% canary with a ten-minute bake time. What should you say?
- (a) Reduce the bake time so the release is faster
- (b) Twenty requests cannot distinguish a 1% error rate from zero, so use blue-green with an instant revert instead ✅
- (c) Raise the canary to 50%, which is still safer than a full release
- (d) Proceed; any canary is better than none
> 40 × 0.05 × 10 = 20 requests. A canary that cannot detect the thing it is watching for is theatre with extra moving parts. Recognising this is judgement, and saying it out loud is what M18 will ask you to do.

### Q5 · L5
Your team argues for thirty minutes during an incident about whether to revert or fix forward. What should have existed beforehand?
- (a) A larger canary percentage
- (b) A second staging environment
- (c) A written rule in the release checklist naming which situations call for each ✅
- (d) A faster CI pipeline
> Revert is the default because it is fast and known; fix forward is correct when data has already been written wrongly or the previous version is also broken. Deciding that at speed, under pressure, is how five minutes of recovery becomes thirty-five — which is exactly what chapter 10's runbook is for.

## Go deeper
- [Martin Fowler: blue-green deployment, canary release and feature toggles](https://martinfowler.com/bliki/CanaryRelease.html)
- [Google SRE workbook: canarying releases](https://sre.google/workbook/canarying-releases/)
