---
id: m05-08-reliability
module: m05-scale-reliability
slug: slos-retries-circuit-breakers
title: "Reliability: SLOs, Retries, Backoff, Timeouts and Circuit Breakers"
order: 8
duration_min: 25
is_free: false
mission: false
volatility: low
last_reviewed: 2026-11-16
---

## Key learnings
- Put a timeout on every outbound call, and say why the default is dangerous
- Retry only idempotent operations, with exponential backoff and jitter
- Explain what a circuit breaker does in each of its three states
- Turn an availability target into minutes of downtime per month
- Catch an AI that retries a payment without an idempotency key

## Lesson

### The pain

Your SMS provider does not go down. It goes *slow*. Requests that normally take 300 milliseconds start taking thirty seconds.

Your code calls it and waits, because nobody set a timeout and the default is to wait as long as it takes. Each waiting request holds an app worker and a database connection. Within ninety seconds every worker in all three instances is parked on a dead SMS call, and your entire app — menu, orders, admin page, everything with nothing to do with SMS — stops answering.

A teammate adds retries. Three attempts per request, immediately, no delay. The provider was struggling with your normal traffic and is now receiving three times as much. The thing that was recovering stops recovering, and your retries are the reason.

One third party got slow and you were down for an hour. That is the shape of most real outages: not a component failing, but a slow component nothing was protecting you from, plus well-meaning retries turning a dip into a disaster.

### The idea

**Reliability is not the absence of failure. It is deciding in advance how much failure is acceptable, and making sure one slow dependency cannot take everything else with it.**

Two halves. The first is a target you write down: an **SLO**, a service level objective, such as "99.9% of menu requests succeed in under 500 ms, monthly". That number tells you when to stop optimising and when to drop everything, and the gap between it and perfection is your **error budget**.

The second is four defences, applied in order to every call that leaves your process: a **timeout** so you stop waiting, a **retry with backoff** so a blip is not a failure, a **circuit breaker** so you stop calling something clearly broken, and a **fallback** so the user still gets something useful.

Of those, the timeout is worth more than the other three combined. An unbounded wait is how a small failure becomes total.

### The analogy

It is 1:10 pm and the juice supplier has not arrived.

Without a rule, the counter stops. Staff wait for juice, the queue backs up behind people who only wanted samosas, and the whole canteen is blocked on one missing item. That is no timeout.

The rule a real canteen has is: **if juice is not here by 1:15, take it off the board and keep serving.** Thirty people get fed, three are disappointed. That is a timeout plus a fallback — the difference between a bad day for juice and a bad day for everybody.

Now the supplier's phone. Calling once more is sensible — maybe the first call dropped. Calling every ten seconds for an hour is a **retry storm**: the line is permanently engaged, and if the supplier was about to call back, they cannot get through. The sane version waits a minute, then two, then four, then stops after the sixth failure and tries once at 2 pm. That last part is a **circuit breaker**.

**Where the analogy breaks:** canteen staff notice and improvise. They get bored of waiting, they look at the queue, they make a call. Code is infinitely patient. It will wait thirty minutes for a reply without complaint and without telling anybody, which is precisely why these rules have to be written down rather than left to good judgement at the time.

### Picture it

```
   ┌──────────┐   5 failures in a row   ┌───────────┐
   │  CLOSED  │ ──────────────────────▶ │   OPEN    │
   │ calls go │                         │ fail fast │
   │ through  │ ◀────────────────────── │  for 30 s │
   └──────────┘     trial succeeded     └─────┬─────┘
                                              │ after 30 s
                        ┌──────────────┐      │
                        │  HALF-OPEN   │ ◀────┘
                        │  one trial   │
                        └──────┬───────┘
                               │ trial failed
                               └────▶ back to OPEN
```

A breaker starts closed and passes calls through, trips open after repeated failures and fails instantly without calling, then after a cooling period allows one trial call that either closes it again or sends it straight back to open.

### Availability arithmetic

Percentages feel abstract until you convert them to minutes. Do that once and you will never promise 99.99% casually again.

| Target | Downtime per month | What that feels like |
|---|---|---|
| 99% | about 7 hours | one bad afternoon |
| 99.9% | about 43 minutes | one lunch rush |
| 99.95% | about 22 minutes | a slow deploy |
| 99.99% | about 4 minutes | essentially no manual recovery |

Then the part people miss: availability **multiplies** across dependencies. If your app needs your host, your database, your cache and your payment provider, each independently 99.9% available, your ceiling is roughly 99.6% — about three hours a month — before you write a single bug. Every dependency spends reliability you did not know you were spending, which is a good reason to work without the cache and a bad reason to add a fifth service for one nice feature.

A target of 100% is not ambitious, it is wrong. A zero error budget means never deploying, never experimenting and never taking a risk, which is how teams stop shipping. M07 builds this properly.

### The four defences, in order

**Timeout.** Every call that leaves your process — database, cache, payment gateway, SMS, model API — gets an explicit deadline. Many client libraries default to none, which means "wait forever". A good first pass: 2 seconds for a cache, 5 for a database query, 10 for a third-party API, and a total request budget smaller than wherever your host cuts you off.

**Retry with exponential backoff and jitter.** Retry once after a delay, then double the delay. Add randomness so a thousand clients do not retry in unison — the same jitter idea as Chapter 3's TTLs. Cap attempts at two or three. And retry only what is safe to repeat: a timeout, a 429, a 503. Never a 400, 401 or 422, because the answer will not change.

**Circuit breaker.** Count failures to one dependency. Past a threshold, stop calling it and return an error immediately — the opposite of patience, and what keeps your workers free. After a cooling period, let exactly one request through to test the water.

**Fallback.** Decide what the user sees when all that has happened. Yesterday's cached menu beats an error page. "Payment confirmation is delayed, we will notify you" beats a spinner. This is **graceful degradation**, and it is the only one of the four your users ever see.

### Where it lives

These defences wrap every arrow in Chapter 2's diagram and every arrow out to a third party. They are the other side of Chapter 4: there you returned 429, here you receive one and must respond well.

Outward: M04's idempotency keys are what make retrying a payment safe at all, and its queues are where work goes when a dependency is down. M06's tests and deploy pipeline are where you prove a fallback works before you need it. M07 is the home of SLOs, error budgets and the alerting that tells you a breaker has tripped. M18's runbook is where "what to do when the breaker opens" gets written for a human.

### Words to own

| Term | Plain meaning | Example |
|---|---|---|
| SLI | The thing you measure | Share of requests under 500 ms |
| SLO | Your own target for that measure | 99.9% monthly |
| SLA | A contract with money attached | Your provider's promise to you |
| Error budget | The failure your SLO permits | 43 minutes a month |
| Timeout | A deadline on one call | 10 s on a third-party API |
| Retry | Trying the same call again | Attempt 2 of 3 |
| Exponential backoff | Doubling the wait each attempt | 1 s, 2 s, 4 s |
| Jitter | Randomness added to the wait | 1 s plus 0–400 ms |
| Retry storm | Everyone retrying at once | Three times the load |
| Circuit breaker | Stop calling a failing dependency | Open for 30 s |
| Graceful degradation | Reduced service instead of none | Yesterday's cached menu |
| Blast radius | How much breaks when one thing does | One feature, or everything |

### Tradeoffs and gotchas

**Retrying a non-idempotent write charges twice.** The request timed out; you do not know whether the payment went through. Retry without an idempotency key and you may have taken ₹120 for a ₹60 plate. M04 taught you the key; this is where you find out what it was for. Retry reads freely, writes only with a key.

**Retries are load amplification at the worst possible moment.** Three attempts per client means the struggling service receives triple traffic exactly when it can least cope. Backoff, jitter, a low attempt cap and a breaker are what make retries safe rather than harmful.

**A timeout that is too short fails requests that would have succeeded.** On slow 4G a legitimate upload may take fifteen seconds. Base the number on your measured p95 from Chapter 1 with headroom, not on a round figure that felt right.

**Timeout budgets must nest.** If your host cuts requests off at 30 seconds and you make three sequential calls with 15-second timeouts each, your own timeouts never fire — the platform kills you first and your fallback never runs. Make the worst case fit inside the outer limit.

**A breaker with no half-open state never recovers,** and one that trips on a single failure trips during every deploy. Thresholds and cooling periods are numbers to choose deliberately.

**Do not put a breaker in front of your own database.** Fail fast on dependencies you can live without; your primary is not one of those, and a tripped breaker there is a self-inflicted outage.

### Verify it

**Ask for the inventory first.** "List every call my code makes that leaves the process, and for each one give me the current timeout, the retry policy, and what the user sees if it fails." The gaps in that table are your outage. An AI that cannot produce it has not read your code.

**Red flag: no timeout, or a default one.** The most common and most expensive omission in generated code. Check each client call in the diff has an explicit deadline. "The library handles it" is worth verifying, because the library's handling is frequently "wait forever".

**Red flag: retries with a fixed delay and no jitter,** or an attempt count above three. Ask what happens if a thousand clients do this simultaneously.

**Red flag: retrying a 400, 401 or 422.** The answer will not change. Retrying a client error wastes quota and hides the real bug.

**Red flag: a retry on a payment, an order creation or any write, with no idempotency key.** Ask straight out: "if this times out after the server already processed it, what stops the user being charged twice?" An answer without the word idempotency is not an answer.

**Red flag: a breaker with no half-open state, or a threshold of one failure.** Ask how it recovers, and what happens during your next deploy.

**Red flag — the recurring ones: "scale up" or "add more memory" as the fix for a slow third party,** which cannot work because the waiting is not happening in your memory; and a timeout added to the fast internal call while the thirty-second external one stays unbounded.

Then prove the behaviour instead of reading about it. Point one dependency at a deliberately wrong address: that feature should fail quickly while everything else keeps serving. Then make the dependency slow rather than dead, with a sleep or throttling, because slow is the case that actually takes systems down. Force the breaker open, confirm later calls return instantly, wait out the cooling period and confirm recovery. And read your logs: a well-behaved failure leaves a record, not silence.

## Walkthrough
1. List every call your app makes that leaves the process. Database, cache, object storage, payment gateway, SMS or email provider, model API, anything else. Expect six to ten lines.
2. For each line, find the timeout actually in effect. Not the one you assume — open the code or the client configuration. Write "none" where there is none, and expect several.
3. For each line, write what the user sees today if that call fails. If the answer is "a spinner forever" or "a 500 page", mark it.
4. Add a column for idempotency: can this call safely be made twice? Any write that cannot is a call you must never retry without a key.
5. Pick the dependency whose failure currently causes the widest damage — usually whichever one has no timeout and is called on your busiest page. That is your blast radius.
6. Brief an AI to add an explicit timeout to that one call, plus a user-visible fallback. One call, one change, nothing else in scope.
7. Test it by breaking the dependency on purpose: point it at a wrong address or an unroutable port. The feature must fail within your timeout and the rest of the app must keep working.
8. Now test slow instead of dead, because slow is what actually causes outages. Throttle the network or add a delay, and confirm your timeout fires rather than your patience.
9. Add retry with exponential backoff and jitter to that same call, capped at two retries, and only for timeouts and 5xx responses. Confirm a 400 is not retried.
10. Write your first SLO for one user-visible thing: "99.5% of menu loads succeed in under 1 second, monthly". Convert it to minutes of allowed failure and write that number next to it.
11. Finally, write the decision-log entry: the timeout values you chose, the reason for each, and what the user sees when each dependency is down.

## Talk to AI
```prompt
CONTEXT:      I am a student directing AI on my team's app. Stack: [your
              stack]. Hosting: [your host], free tier, which cuts requests
              off after about [X] seconds. Measured p95 for my main page:
              [X] ms. Users are on slow 4G in India; payments via UPI.

              Every call my app makes that leaves the process, with the
              timeout currently in effect and what the user sees on
              failure:
              1. [call] - timeout: [none/X s] - user sees: [...]
              2. [call] - timeout: [none/X s] - user sees: [...]
              3. [call] - timeout: [none/X s] - user sees: [...]
              4. [call] - timeout: [none/X s] - user sees: [...]
              5. [call] - timeout: [none/X s] - user sees: [...]

              Which of those calls are writes that cannot safely be
              repeated: [list them]

GOAL:         Make one slow dependency unable to take down the whole app,
              and give every outbound call a deliberate deadline.

REQUIREMENTS: 1. Recommend an explicit timeout for each call, justified
                 against my measured p95 and my host's request limit
              2. Show the arithmetic that my timeouts nest inside my
                 host's cut-off even in the worst case
              3. Say for each call whether to retry, how many times, with
                 what backoff and jitter, and which status codes qualify
              4. For every write that cannot be repeated safely, state
                 exactly what makes a retry safe, or say do not retry
              5. Tell me which single dependency deserves a circuit
                 breaker first, with threshold and cooling period
              6. Define the user-visible fallback for each call, in words
                 a student would see on screen
              7. Rank everything by how much blast radius it removes

CONSTRAINTS:  Free tier only, no new paid services. Do not put a circuit
              breaker in front of my primary database. Every change must
              be shippable on its own. Assume a dependency is more likely
              to be slow than to be completely down.

NON-GOALS:    Do not add a retry to anything that cannot be safely
              repeated. Do not propose scaling up or adding memory. Do not
              refactor files unrelated to these calls.

ACCEPTANCE:   - Every call has an explicit timeout with a justification
              - The worst-case sum of my timeouts is under my host's limit,
                shown as arithmetic
              - No non-idempotent write is retried without a stated
                mechanism that makes it safe
              - Every call has a fallback written as user-facing text
              - You have named the one change with the largest blast-radius
                reduction

ASK:          First give me the table and the ranking. Then tell me how I
              would test the top-ranked change by deliberately breaking
              something. No code until I pick one.
```

## Stuck? Ask AI
```prompt
Explain why a slow dependency is more dangerous than a completely dead one, step by step, with what happens to my app's workers at each moment. Then explain what a timeout changes about that story, and why library defaults are often the worst possible choice.
```

```prompt
Explain exponential backoff with jitter using real numbers across five retry attempts, and show me what a thousand clients retrying in unison does to a struggling service versus the same thousand clients with jitter. Then tell me which HTTP status codes I should and should not retry, and why for each.
```

```prompt
Walk me through a circuit breaker's three states with a concrete timeline of twenty calls to a failing payment provider. Say what happens to each call, when the state changes and why. Then tell me two places I should never put a breaker.
```

## Quiz

### Q1 · L3
A third-party API becomes slow rather than failing, and your code has no timeout on that call. What is the likely outcome?
- (a) Only the feature using that API breaks
- (b) Every worker ends up waiting on that call and the whole app stops answering ✅
- (c) The load balancer routes around the problem
- (d) The requests fail immediately with an error
> Waiting requests hold workers and connections. This is why an unbounded wait turns one slow dependency into a total outage, and why the timeout is worth more than the other three defences combined.

### Q2 · L2
An availability target of 99.9% measured monthly allows roughly how much downtime?
- (a) About 4 minutes
- (b) About 43 minutes ✅
- (c) About 7 hours
- (d) None, by definition
> Roughly 43 minutes a month — about one lunch rush. Converting percentages to minutes is what stops teams promising 99.99% without understanding it.

### Q3 · L4
Your payment call times out. You do not know whether the charge went through. What makes a retry safe?
- (a) Waiting ten seconds before retrying
- (b) Sending the same idempotency key so a repeat has the effect of one charge ✅
- (c) Retrying at most three times
- (d) Adding jitter to the retry delay
> Backoff and caps control load, not correctness. Only an idempotency key makes a repeated write safe, which is exactly what M04 built it for.

### Q4 · L4
Which response should your code retry?
- (a) 400 Bad Request
- (b) 401 Unauthorized
- (c) 503 Service Unavailable ✅
- (d) 422 Unprocessable Entity
> Retry transient failures: timeouts, 429 and 5xx. Client errors will return the same answer however many times you ask, so retrying them wastes quota and hides the real bug.

### Q5 · L5
Your app depends on a host, a database, a cache and a payment provider, each independently 99.9% available. What is your realistic ceiling?
- (a) 99.9%, since the weakest link sets the limit
- (b) About 99.6%, because availability multiplies across dependencies ✅
- (c) 100%, because the dependencies are independent
- (d) 99.99%, because redundancy improves with more services
> Four dependencies at 99.9% multiply to roughly 99.6% — about three hours a month before you write a bug. It is why being able to work without the cache is worth more than it looks.

## Go deeper
- [Google SRE Workbook: implementing SLOs](https://sre.google/workbook/implementing-slos/)
- [AWS: timeouts, retries and backoff with jitter](https://aws.amazon.com/builders-library/timeouts-retries-and-backoff-with-jitter/)
