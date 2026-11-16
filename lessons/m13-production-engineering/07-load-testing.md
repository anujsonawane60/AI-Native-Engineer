---
id: m13-07-load-testing
module: m13-production-engineering
slug: load-testing
title: "Load Testing and Finding Your Breaking Point"
order: 7
duration_min: 35
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

M05 chapter 9 taught you back-of-envelope estimates: 1,200 students, 70% arriving in a forty-minute window, three counters plus a spare. It taught you to compute requests per second before designing anything, and it promised M13 would give load testing its own chapter. M05 chapter 1 taught you why things get slow, and M07 chapter 6 taught you p95 and histograms. Estimating is the Sprint. Measuring is this.

So a team runs their first load test. They point a tool at their homepage, crank it to a thousand virtual users, and watch it serve 12,000 requests per second with a p95 of 40 ms. They screenshot it and feel excellent. What they measured was their CDN, because the homepage is a cached static page, and they never got within a hundred metres of their application. Two weeks later a college festival sends four hundred real students into the app at once and it dies at the order endpoint, which the test never touched.

Second failure, which costs money. Another team does test the real endpoints, from their laptop, for twenty minutes, against their production environment. Three things happen. Their laptop's uplink saturates at about 60 requests per second, so every number they recorded is a measurement of their own hostel wifi. Their cloud egress meter runs for twenty minutes and produces a bill of around ₹3,800 — the exact trap M06 chapter 7 warned about. And their payment provider's sandbox rate-limits them and briefly blocks the key, so real checkouts fail for twenty minutes.

A load test is a controlled experiment. Run as a vibe, it costs money and teaches nothing.

### The idea

**A load test's purpose is to find the number of concurrent users at which your system stops getting faster and starts getting wrong, and then to explain why.**

That number is the **knee**. Below it, adding users adds throughput and latency barely moves. At it, throughput flattens while latency climbs — queues are forming somewhere. Past it, errors appear and throughput *falls*, because the system is spending its capacity on work it will never complete. The test is only useful if the journey it drives is realistic and the data it uses is varied, because a test that requests the same URL with the same id measures your cache and nothing else.

### The analogy

The canteen holds a fire drill. Everyone leaves, they time it, the number is written on a form.

But watch the drill they actually run: the same twenty students, queueing in the same order, walking the same route they walked last month. It takes ninety seconds, as it did last time, and the form says the canteen is safe. Then a festival night arrives — four hundred students, half of them unfamiliar with the building, two carrying trays, one wheelchair, the side door locked because the caretaker went home. The ninety-second figure describes an event that will never happen again.

A real drill varies the thing being measured: different numbers, different entrances, a blocked route, people who do not know where to go. And it ends with a specific finding — "the side stairwell takes four minutes because it narrows at the landing" — not with a number on a form.

**Where the analogy breaks:** the canteen's gas line has a stamped capacity you can read off the pipe and a physical limit everyone can see. Your bottleneck is invisible. It might be a database connection limit, a single-threaded image resize, a free-tier request quota, or your own laptop's wifi. You cannot see it; you can only find it by plotting latency against concurrency and watching where the line bends — and then proving which component bent it.

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

This is the measuring instrument for most of M05. The replica and pool limits from M05 chapter 6 are usually what bends the line first. The cache hit rate from M05 chapter 3 is what you must deliberately defeat with varied data, or you measure nothing. The rate limiter from M05 chapter 4 will cut your test off, and you need to know whether that is your limiter working correctly or your test being wrong. M07 chapter 6 is where the p95 comes from, and chapter 6's golden signal of **saturation** is the one that tells you *which* component is the bottleneck.

Inside M13, this chapter supplies numbers to three others. Chapter 3's resource requests and limits should come from a load test, not a guess. Chapter 5's canary thresholds are only defensible if you know your normal p95 under load. And chapter 9's failover plan needs to know whether one region alone can carry your traffic.

The free path is good here. The standard open-source load-testing tool runs locally at no cost with no account, and several hosted services give a small monthly quota of cloud-run test minutes free, which is how you test from a machine that is not your hostel wifi. Run against staging, not production.

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

**Vary the data or you are testing your cache.** This is the single most common wasted load test. Requesting `/dish/42` five million times produces one cache key and a magnificent number. Parameterise: a pool of a few hundred student accounts, a pool of dish ids, randomly chosen per iteration. Then record your cache hit rate *during* the test — if it is 99%, your real users' experience is not what you measured.

**Include think time, or your closed-model test lies in both directions.** Real students read the menu for four seconds. A virtual user with no pause does maybe thirty times the requests of a real one, so "200 virtual users" might represent six thousand real students — or, if your bottleneck is connection count rather than request rate, far fewer. State your conversion explicitly: these N virtual users with this think time represent roughly M concurrent students, and here is the arithmetic.

**Take a baseline first, and smoke test before that.** One virtual user, one full journey, checking the response bodies are actually correct. A test that gets 200s containing error pages is the most confident wrong result you can produce. Then measure at low load so you have something to compare against.

**Assert on error rate and correctness, not only on latency.** A system past its knee often returns fast failures, which makes p95 *improve*. If your test's only threshold is latency, your worst run can look like your best. Make the test fail on an error rate above a stated fraction, and check a value in the response body.

**Do not load test from your laptop, and never against production without a plan.** Your home uplink will be the bottleneck and you will be measuring your wifi. Use a free cloud quota or a small VM in a region near your users. Against production, you risk real users and you definitely risk the bill — and if you must, do it at the quietest hour, with a stop condition, having warned your team.

**Never load test a third-party provider.** Payment sandboxes, mapping APIs, AI model endpoints, email services. It violates their terms, it gets your key rate-limited or blocked, and it can break real checkouts. Stub them at your boundary and load test your own code. This also means your test exercises your timeouts from M05 chapter 8, which is worth knowing separately.

**Set a budget alert before the first run, not after.** Egress is metered per gigabyte and a load test is an egress machine. M06 chapter 7's figure was ₹3,800 for twenty minutes. Set the alert, keep runs short, and prefer a staging environment with small payloads.

**Finding the knee is half the work; naming the bottleneck is the other half.** A knee with no explanation is a number. Go look at saturation: connection pool usage, CPU, memory, queue depth, lock waits. "We flatten at 400 users because the pool of 20 connections is fully occupied and requests queue" is a finding you can act on, and it is what M18 will ask you for.

### Verify it

1. **Check the cache hit rate during the test.** High hit rate on data that should be varied means your test is measuring the cache. Fix the data, not the conclusion.
2. **Read one response body from the run.** Confirm it contains real, correct data for the right student. 200s full of empty results are the classic false pass.
3. **Confirm the load generator was not the bottleneck.** Check its own CPU and network. If the generator was saturated, every number is invalid; this is why you do not run from a laptop.
4. **Ask where the threshold numbers came from.** A test with `p95 < 500ms` must be able to say why 500. Your SLO from M05 chapter 8 is the only legitimate source.
5. **Demand the bottleneck, named, with its saturation metric.** If the report is only a knee value, the test is unfinished.
6. **Confirm no third-party service was called.** Look at the script for any external host. One payment sandbox call per iteration at 400 users is how you get a provider key blocked.
7. **Check the bill the next morning.** Not the estimate — the actual meter. Record the rupees per test run so your team knows what an experiment costs.

Red flags: a load test that hits a cache and proves nothing; a test with no error-rate threshold; a test run from a laptop; identical ids on every iteration; no think time and no stated conversion to real users; a test against production with no stop condition; a payment sandbox in the script; a knee reported with no named bottleneck; and a result quoted with no baseline to compare it against.

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
