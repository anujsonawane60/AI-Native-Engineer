---
id: m13-07-load-testing
module: m13-production-engineering
slug: load-testing
title: "Load Testing and Finding Your Breaking Point"
order: 7
duration_min: 25
is_free: false
mission: true
volatility: low
last_reviewed: 2026-11-16
---

## Key learnings
- Write a load test as a realistic user journey with varied data, not a URL hammer
- Find the knee: the point where throughput stops rising and latency starts climbing
- Read error rate and saturation alongside p95, and know which one ends the test
- Run a test that cannot produce a surprise cloud bill or hit a payment provider
- Catch a load test that only proved your cache works

## Lesson

### The pain

M05 chapter 9 taught you back-of-envelope estimates: 1,200 students, 70% arriving in a forty-minute window, three counters plus a spare. M05 chapter 1 taught you why things get slow, and M07 chapter 6 taught you p95 and histograms. Estimating is the Sprint. Measuring is this.

A team points a tool at their homepage, cranks it to a thousand virtual users, and records 12,000 requests per second at 40 ms p95. They measured their CDN: the homepage is a cached static page. Two weeks later a festival sends four hundred students at the order endpoint, which the test never touched, and the app dies. A load test is a controlled experiment; run as a vibe it teaches nothing.

### The idea

**A load test's purpose is to find the number of concurrent users at which your system stops getting faster and starts getting wrong, and then to explain why.**

That number is the **knee**. Below it, adding users adds throughput and barely moves latency. At it, throughput flattens while latency climbs, because queues are forming somewhere. Past it, errors appear and throughput *falls*, because capacity is going to work that will never complete.

### The analogy

The canteen holds a fire drill: they time it and the number goes on a form. But watch the drill they actually run — the same twenty students, queueing in the same order, on last month's route. Then festival night arrives: four hundred students, half unfamiliar with the building, the side door locked. A real drill varies what it measures and ends with a finding — "the side stairwell takes four minutes because it narrows at the landing" — not a number on a form.

**Where the analogy breaks:** the gas line has a stamped capacity anyone can read. Your bottleneck is invisible — a connection limit, a single-threaded image resize, your own wifi — and you find it only by plotting latency against concurrency.

### Picture it

```
 concurrent   throughput (req/s)       p95      errors
   users
    50        480  ████████            110 ms   0%
   100        940  ████████████████    140 ms   0%
   200       1810  ██████████████████  220 ms   0%
   400       1920  ███████████████████ 980 ms   0.2%  ◀ the knee
   800       1740  █████████████████   4200 ms  6%    ◀ past it
```

Throughput rises with concurrency until it flattens; the knee is where latency starts climbing steeply while throughput stops improving, and past it errors appear and throughput actually falls.

### Where it lives

This is the measuring instrument for most of M05:

- The pool limits from **M05 chapter 6** usually bend the line first, and **M05 chapter 4's** rate limiter will cut your test off.
- **M07 chapter 6** supplies the p95, and its **saturation** signal names which component is the bottleneck.
- Inside M13 it feeds **chapter 3's** resource limits, **chapter 5's** canary thresholds and **chapter 9's** failover plan.

The free path is good: the standard open-source tool runs locally with no account, and several hosted services give a small free monthly quota of cloud-run minutes — how you test from a machine that is not your hostel wifi. Run against staging.

### Words to own

| Term | Plain meaning | Example |
|---|---|---|
| Virtual user | A simulated user running a journey in a loop | 200 VUs |
| Scenario / journey | The sequence a virtual user performs | Browse, add, pay |
| Think time | A pause between a user's steps | 2–5 seconds |
| Closed model | Fixed number of users; slow responses slow arrivals | VU-based |
| Open model | Fixed arrival rate regardless of response time | Requests per second |
| Ramp | Increasing load gradually over time | 0→400 over 5 min |
| Baseline | The measurement at low load, taken first | 110 ms p95 at 50 |
| Knee | Where throughput flattens and latency climbs | 400 users |
| Saturation | How full the limiting resource is | Pool at 100% |
| Smoke test | One user, one pass, to prove the script works | Always first |
| Soak test | Moderate load for a long time | 2 hours |
| Spike test | Sudden jump to high load | 0→500 in 10 s |
| Stress test | Deliberately past the knee, to see how it fails | 800 users |
| Parameterised data | Different ids and accounts per iteration | 500 students |
| Threshold | A pass/fail condition on the test itself | p95 < 500 ms |

### Tradeoffs and gotchas

**Vary the data or you are testing your cache.** `/dish/42` five million times produces one cache key and a magnificent number.

**Include think time, and state the conversion.** A virtual user with no pause does perhaps thirty times the requests of a real student, so write down what N virtual users represent.

**Smoke test, then baseline, then assert on errors.** 200s containing error pages are the most confident wrong result you can produce, and past the knee fast failures make p95 *improve*. Fail on error rate too.

**Do not load test from your laptop, and set the budget alert first.** Your uplink becomes the bottleneck and you measure your wifi. Egress is metered per gigabyte: M06 chapter 7's figure was ₹3,800 for twenty minutes. Against production, go at the quietest hour with a stop condition.

**Never load test a third-party provider.** Payment sandboxes, maps, model endpoints: it breaks their terms, gets your key blocked and can fail real checkouts. Stub them at your boundary, which also exercises your M05 chapter 8 timeouts.

**Naming the bottleneck is the other half of the work.** "We flatten at 400 users because the pool of 20 connections is fully occupied" is what M18 will ask you for.

### Verify it

1. **Check the cache hit rate during the test.** A high rate on varied data means you measured the cache.
2. **Read one response body** and confirm it holds correct data for the right student.
3. **Confirm the load generator was not itself saturated.** If it was, every number is invalid.
4. **Ask where the threshold numbers came from.** A test asserting `p95 < 500ms` must say why 500, and your M05 chapter 8 SLO is the only source.
5. **Demand the bottleneck, named, with its saturation metric,** then check the actual bill and record the rupees per run.

Red flags: a test that hits a cache and proves nothing; no error-rate threshold; a run from a laptop; identical ids every iteration; a payment sandbox in the script; a knee with no named bottleneck.

## Walkthrough
1. Write down the one user journey that matters most in your product — the sequence a real student actually performs, with the pauses. For the canteen that is open the menu, view a dish, add it, pay. Then, using M05 chapter 9's method, estimate your realistic peak in requests per second and write it down before you measure anything.
2. Set a budget alert on your cloud account first, at a rupee figure you would be upset to see. Then confirm your test will run against staging, not production.
3. Install the free open-source load tool locally and write a one-user smoke test of your journey. Run it once and read the response bodies. Confirm the data is correct, not merely that the status was 200.
4. Parameterise the data: a pool of at least a hundred accounts and a hundred item ids, chosen at random per iteration. Add realistic think time between steps and write down the arithmetic converting virtual users to real students.
5. Stub every third-party call at your boundary — payments, maps, AI, email. Confirm by searching your script for external hostnames that none remain.
6. Take a baseline: fifty virtual users for three minutes. Record throughput, p50, p95 and error rate. Everything later is compared to this.
7. Now ramp: step up the virtual users — 100, 200, 400, 800 — three minutes at each level, and record the same four numbers at every step. Stop immediately if error rate passes a figure you decided in advance.
8. Plot throughput against concurrency and find the knee: the level where throughput stops improving and p95 starts climbing steeply. Write down the concurrency number.
9. Now name the bottleneck. While holding load at the knee, look at saturation: database connection pool usage, CPU, memory, queue depth. Also record your cache hit rate. Write one sentence of the form "we flatten at N users because X is at 100%".
10. Fix one thing — a connection pool size, a missing index from M04, a cache with varied keys — and re-run the same ramp. Record the new knee. The difference between the two numbers is the only honest measure of an optimisation.
11. Check your actual cloud bill the next morning and record the rupees per test run, then write the knee, the bottleneck and the cost into your repository as a capacity note.

## Talk to AI
```prompt
CONTEXT:      I am a student on an AI-native engineering program. I direct AI
              and verify its output; I do not write code by hand. My team of
              three runs [product in two sentences] with [number] real users
              in India on budget Android phones and slow 4G. Stack: [your
              stack]. Database: [your database] on a free tier with a
              connection limit of [number]. We have a cache: [describe it or
              say none]. We deploy on [platform] and our staging environment
              is [describe it]. We use these third-party services: [payments,
              maps, AI, email - list them]. My estimated realistic peak from
              earlier in the course is [number] requests per second. Our SLO
              is [state it, or say we have none]. Budget: free tiers only, and
              a surprise bill of a few thousand rupees is a serious problem.

GOAL:         A load test of ONE real user journey that finds our knee, names
              the bottleneck causing it, and cannot produce a surprise bill or
              touch a third-party provider.

REQUIREMENTS: 1. Turn my journey into a scenario with realistic think time,
                 and give me the arithmetic converting virtual users into real
                 concurrent students for MY think-time values
              2. Parameterise the data so I am not measuring my cache. Tell me
                 how many distinct accounts and item ids I need, and how to
                 confirm during the run that the cache is not answering
                 everything
              3. Give me a smoke test first: one user, one pass, asserting the
                 response BODY is correct and not just a 200
              4. Give me a ramp profile with explicit steps and durations, and
                 a stop condition on error rate with a number I decided
              5. Thresholds for the test itself - p95 and error rate - derived
                 from my SLO, and say explicitly where each number came from
              6. Tell me how to stub every third-party service at my boundary,
                 and why load testing a payment sandbox can block my real
                 checkouts
              7. Tell me where to RUN the generator from, why my laptop is the
                 wrong place, and what free options exist to run it from a
                 machine near my users
              8. Tell me exactly which saturation metrics to watch while at
                 the knee so I can NAME the bottleneck - connection pool, CPU,
                 memory, queue depth - and where each is visible
              9. Tell me what this test will cost in rupees, which meters it
                 runs, and the budget alert to set BEFORE the first run

CONSTRAINTS:  Free tier only, and say clearly where free is not possible.
              Against staging, never production. Plain English for a
              second-language reader. Label every command READ-ONLY or
              CHANGES-THINGS. Keep runs short - tell me the shortest run that
              still gives a valid answer. Rupees for every cost.

NON-GOALS:    Do not test my third-party providers. Do not run against
              production. Do not recommend a paid load-testing plan. Do not
              optimise my code in this conversation - I want a measurement
              first. Do not give me a test that only hits my homepage.

ACCEPTANCE:   - My smoke test asserts a correct value from the response body
              - During the ramp, my cache hit rate shows the data is varied
              - I have throughput, p50, p95 and error rate at every step
              - I can state my knee as a concurrency number
              - I can name the bottleneck with its saturation metric at 100%
              - No external hostname appears anywhere in my script
              - A budget alert existed before my first run, and I know the
                actual rupee cost of one run

ASK:          Before writing any script, tell me the five ways a beginner's
              load test produces a meaningless result, and how I would detect
              each one in my own run. Then give me the smoke test only, and
              wait for me to confirm it passes before you give me the ramp.
```

## Mission

**Objective.** Find your own startup's knee with a realistic, varied, third-party-free load test against staging, name the bottleneck that causes it, fix one thing, and prove the knee moved.

**Steps.**
1. **Frame** — write down the one journey that matters, with its real pauses, and your estimated peak requests per second from M05 chapter 9's method. Estimate before you measure.
2. **Guard the money first** — set a budget alert at a rupee figure you would hate to see, and confirm your target is staging. Screenshot the alert; it is part of your upload.
3. **Brief** — use the Director's Brief above. Demand the five-ways-to-get-a-meaningless-result list before any script.
4. **Smoke test** — one user, one pass, asserting a correct value from the response body. Do not proceed until this passes.
5. **Make it realistic** — parameterise accounts and item ids, add think time, and write the arithmetic converting virtual users to real students.
6. **Remove the third parties** — stub every external service and prove it by searching your script for external hostnames.
7. **Baseline** — fifty virtual users, three minutes. Record throughput, p50, p95 and error rate.
8. **Ramp** — step up with a pre-decided stop condition, recording all four numbers at each level, plus your cache hit rate.
9. **Find the knee** — plot it and state the concurrency number where throughput flattens and p95 climbs.
10. **Name the bottleneck** — while at the knee, find the saturation metric that is pinned, and write the one-sentence finding.
11. **Fix one thing and re-measure** — change a single thing and run the identical ramp. Report both knees.
12. **Document** — a capacity note in your repository: your knee, your bottleneck, your estimated peak, whether the first comfortably exceeds the second, and the rupee cost of one test run.

**Upload.**
- Your Director's Brief, in full → post caption or an attached file
- Your budget alert, set before the first run → screenshot
- Your estimated peak requests per second, written before measuring → text
- Your smoke test asserting a response body value, passing → screenshot
- Your virtual-user-to-real-student arithmetic → text
- Proof no external hostname appears in your script → screenshot of the search
- A table of throughput, p50, p95, error rate and cache hit rate at every load level → text or screenshot
- Your plotted knee, with the concurrency number marked → screenshot
- The saturation metric that was pinned at the knee → screenshot
- Your one-sentence bottleneck finding
- The identical ramp after your single fix, with the new knee → screenshot
- Your actual cloud bill for the test runs, in rupees → screenshot
- Your committed capacity note
- Reflection, 150 words max → one tradeoff you chose, and one thing the AI got wrong

**Self-check.** Tick every box before you submit.
- [ ] I estimated my peak before I measured anything
- [ ] A budget alert existed before my first run
- [ ] My smoke test checked a value in the response body, not just the status
- [ ] My test uses at least a hundred distinct accounts and item ids
- [ ] I recorded cache hit rate during the run and it shows varied data
- [ ] My test includes think time and I can convert virtual users to real ones
- [ ] No third-party service is called anywhere in my script
- [ ] I ran the generator from somewhere other than my own laptop
- [ ] My test has an error-rate threshold, with a number I chose and can justify
- [ ] I ran against staging, not production
- [ ] I can state my knee as a number and name the bottleneck with its metric
- [ ] I re-ran the identical ramp after one fix and have both knees
- [ ] I know the rupee cost of one test run from the actual bill
- [ ] My reflection names something specific the AI got wrong

**Stretch.** Run a two-hour soak at 60% of your knee and compare the p95 in the first ten minutes with the last ten. If it drifted upward, you have found a leak that no short test can reveal; write 80 words on what you would investigate first.

## Stuck? Ask AI
```prompt
Here are my load test results: [paste the table of concurrency, throughput, p50, p95 and error rate at each level]. Do not suggest optimisations yet. First tell me where my knee is and how you can tell. Then tell me which of my numbers look suspicious - in particular whether anything suggests I was measuring a cache, a rate limiter, or my own load generator rather than my application. Finish with the three saturation metrics I should check next to name the bottleneck.
```

```prompt
Explain the difference between a closed workload model, where I set a number of virtual users, and an open model, where I set an arrival rate. Give me one situation where each gives a dangerously misleading answer. Then ask me which one I used and what my think time was, and work out with me roughly how many real concurrent users my test actually represented.
```

```prompt
Explain why a load test that requests the same URL repeatedly can report an excellent result while the real system is fragile. Walk me through what each layer - CDN, application cache, database - sees in that test versus in real traffic. Then give me a checklist for making test data varied enough, and tell me the one measurement during the run that would have exposed the problem immediately.
```

## Quiz

### Q1 · L3
Your load test reports 12,000 requests per second with a 40 ms p95 against your homepage. What have you most likely measured?
- (a) Your application's true capacity
- (b) Your CDN or cache serving one identical response ✅
- (c) Your database's read throughput
- (d) Your load generator's maximum output
> A single cached URL produces one cache key and a magnificent number that describes the edge, not your code. The fix is parameterised data and watching your cache hit rate during the run.

### Q2 · L4
As you increase concurrency past a point, throughput falls and p95 improves. What is happening?
- (a) The system has auto-scaled and is now faster
- (b) You are past the knee and the system is returning fast failures, so a latency-only threshold hides it ✅
- (c) The load generator has started skipping iterations
- (d) The cache has warmed up and is absorbing traffic
> Errors are usually fast, so overload can make p95 look better. This is why a load test must fail on error rate and check a value in the response body, not only on latency.

### Q3 · L4
Why must you never include a third-party payment sandbox in a load test?
- (a) Sandbox responses are slower than production and distort the result
- (b) It breaks their terms, gets your key rate-limited or blocked, and can break real checkouts ✅
- (c) Sandboxes do not support concurrent requests at all
- (d) The test would measure their latency rather than yours, which is harmless
> Stub it at your boundary. A blocked key is a real outage for your real users, and stubbing has the useful side effect of exercising your own timeouts from M05 chapter 8.

### Q4 · L4
Your test ran from your laptop on hostel wifi and flattened at 60 requests per second. What is the correct conclusion?
- (a) Your application's capacity is 60 requests per second
- (b) Nothing about your application — the generator was the bottleneck, so every number is invalid ✅
- (c) Your database connection pool is the limiting factor
- (d) Your CDN is not caching the tested endpoints
> If the load generator saturates, you measured your own uplink. Checking the generator's own CPU and network is a mandatory step, and it is why a small cloud machine near your users is the right place to run from.

### Q5 · L5
You report "our knee is 400 concurrent users". Your reviewer is unimpressed. What is missing?
- (a) A longer test duration
- (b) The named bottleneck with its saturation metric, so the number can be acted on ✅
- (c) A comparison against a competitor's product
- (d) The test's total cost in rupees
> A knee with no explanation is trivia. "We flatten at 400 because our pool of 20 connections is fully occupied and requests queue" tells you what to change, what to re-measure, and what to say in M18's defence.

## Go deeper
- [k6: writing your first load test, thresholds and scenarios](https://grafana.com/docs/k6/latest/examples/get-started-with-k6/)
- [Google SRE: load testing and the meaning of saturation](https://sre.google/sre-book/monitoring-distributed-systems/)
