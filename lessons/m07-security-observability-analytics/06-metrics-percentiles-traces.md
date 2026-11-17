---
id: m07-06-metrics-percentiles-traces
module: m07-security-observability-analytics
slug: metrics-percentiles-traces
title: "Metrics, Percentiles and p95; Traces and Spans"
order: 6
duration_min: 15
is_free: false
mission: false
volatility: low
last_reviewed: 2026-11-16
---

## Key learnings
- Read a p50, p95 and p99 and say what each one claims about real users
- Explain why an average response time hides the users having the worst time
- Pick the right metric type for a thing you want to count or measure
- Read a trace waterfall and point at where the time actually went
- Name the four signals worth putting on a dashboard before anything else

## Lesson

### The pain

A team is told their app feels slow. Their dashboard reports an average response time of 480 milliseconds, so they conclude the complaint is wrong. Three weeks later they sit next to a user: her order takes nine seconds, twice, then is instant, then takes eleven and she gives up.

Both were true. The average was 480 milliseconds *and* one request in twenty took over four seconds. At fifteen taps a session, "one in twenty" means almost every student meets the slow version. The average was arithmetic about requests; the complaint was about people.

### The idea

**An average describes your requests. A percentile describes your users — specifically the ones having the worst time.**

Sort every request by duration. The p50, or median, sits in the middle; the p95 is 95% of the way along, so one request in twenty was slower; the p99 is the worst in a hundred. Teams live by p95, because that is where complaints come from and it is still stable enough to chart.

| Type | What it does | Example | Read it as |
|---|---|---|---|
| Counter | Only goes up | Orders placed; errors returned | A rate: per minute, per hour |
| Gauge | Goes up and down | Queue length; memory in use; open connections | The current value and its trend |
| Histogram | Buckets many values so percentiles can be computed | Request duration | p50, p95, p99 |

**You cannot get a p95 out of a counter or a gauge.** Percentiles require a histogram from the start, and "track response time" often produces one average and nothing else. Four charts are enough to begin with, from Google's site reliability work: **latency** as p50 and p95 per route, failed requests kept separate; **traffic** as requests per minute, so you can tell "no errors" from "no users"; **errors** as a percentage of traffic; **saturation**, how full the limiting thing is — usually database connections or a quota, not CPU.

### The analogy

The canteen's kitchen thermometer is a metric: one number read over and over, where the shape over time is the information. A single 180°C reading means nothing; 180°C and climbing when it should be 140°C is an incident. The queue's average wait of four minutes mixes 10:30am with 1pm, when the p95 wait is eighteen minutes and the student with a 1:15 lecture does not care about your average. The CCTV is a trace: fourteen of those eighteen minutes went at the payment counter, so hiring another server would have fixed nothing.

**Where the analogy breaks:** a thermometer measures one oven; a metric summarises thousands of events, so it has already thrown information away. You cannot ask a metric "which student?" — that is what logs and traces are for.

### Picture it

```
  100 requests to POST /orders, sorted fastest ──▶ slowest

  ├──────────────────────────────┬──────────┬────┬──┤
  0                            p50        p90  p95 p99
                             210 ms      1.9 s 4.2s 11 s

  mean = 480 ms  ◀── a number almost nobody experienced
```

Requests laid out fastest to slowest show the mean far to the left of the tail, which is why a 480-millisecond average and a four-second p95 describe the same morning.

A metric says checkout's p95 is 9.2 seconds but not why. A **trace** is the end-to-end record of one request, broken into timed **spans**.

```
  POST /orders                              total 9,240 ms
  ├─ auth check                    12 ms  ▌
  ├─ load menu prices              34 ms  ▌
  ├─ check stock  (×14 queries) 1,180 ms  ████
  ├─ call payment provider      7,860 ms  ██████████████████
  └─ write order row              140 ms  ▌
```

One request split into steps shows 85% of the nine seconds spent waiting for an outside provider, so database tuning would not have helped.

Two things jump out of a waterfall and nowhere else. Time waiting on somebody else's system changes the fix from "make our code faster" to "add a timeout and show a pending state". And `×14 queries` is fourteen round trips for a fourteen-item order — one query per row instead of one for all. That pattern is an N+1: invisible in logs and metrics, extremely common in generated code.

### Where it lives

These are pillars two and three from chapter four.

- M05 — timeouts, retries, backoff and error budgets, which are promises written in these numbers ("99% of orders under two seconds") and cannot exist until something measures them
- Chapter 7 — these numbers as alerts: alerting on latency and error rate works, alerting on CPU does not
- M12 — the same percentiles as performance budgets on real devices

### Words to own

| Term | Plain meaning | Example |
|---|---|---|
| Percentile | The value N% of measurements fall below | p95 = 4.2 s |
| p50 / median | The middle measurement | What normal feels like |
| p95 | One request in twenty is slower than this | What complaints are about |
| Tail latency | The slow end of the distribution | p99 and beyond |
| Counter | A number that only increases | Total errors |
| Gauge | A number that moves both ways | Open connections |
| Histogram | Bucketed measurements, so percentiles work | Request duration |
| Trace | One request's full journey | 9.24 s across five spans |
| Span | One timed step inside a trace | "payment provider: 7.86 s" |
| N+1 | One query per row instead of one for all rows | 14 stock checks |
| Saturation | How full the limiting resource is | 18 of 20 DB connections |

### Tradeoffs and gotchas

- **Measure where the user is, not where the server is.** Your server's p95 excludes the slow 4G to a budget phone in Nashik, so 200 milliseconds there can be a six-second experience.
- **A p95 is meaningless without saying of what, and when.** p95 across all routes mixes a static image with a payment; p95 across a day hides the 1pm spike.
- **Percentiles cannot be averaged.** Monday's p95 averaged with Tuesday's is not the p95 of those two days. AI makes this mistake in generated dashboards; a "mean p95" means something is wrong.
- **At low traffic, percentiles are noisy.** With 40 requests a day your p95 is the second-slowest one. Read it as a direction, not a fact.
- **Tracing everything is unaffordable, so you sample** — some normal requests plus everything that errored. The request a user complains about probably has no trace: you reproduce rather than retrieve.

### Verify it

Ask AI for "monitoring" and you often get a CPU chart and an average.

1. **Look for percentiles, not means.** **A dashboard with only averages** is the headline red flag here: it looks like information and describes nobody.
2. **Check there is an error rate, as a percentage of traffic.** Latency with no errors cannot tell a healthy app from one failing instantly.
3. **Check traffic is on the same screen.** Zero errors with zero users is the commonest false sense of health there is.
4. **Confirm durations are recorded as histograms.** Can this produce a p95, or only a mean? A mean never becomes a percentile afterwards.
5. **Find one real slow request and open its trace.** Without a span breakdown you are guessing.
6. **Compare the server number with your own phone on a throttled connection.** Server says 200 milliseconds, stopwatch says six seconds: now you know where to look.

Red flags: averages only; no error rate; **p95 measured only inside the server** but quoted as the user experience; latency charts including failed requests, making outages look fast; a metric labelled with a user id; CPU graphs offered as an answer to slowness.

## Walkthrough
1. Open your deployed app's existing dashboard and write down every number it shows you. Mark each as a counter, a gauge or a percentile.
2. Find out whether any of them is a percentile. If every latency number is an average, write that down as finding one.
3. List your product's four or five most important routes — the ones a user would notice. Everything that follows is per route, not overall.
4. For your busiest route, find or request p50 and p95 for the last 24 hours. Write both numbers, and note the time of day each peaks.
5. Compute the ratio of p95 to p50. If it is above about five, you have a tail problem rather than a general slowness problem, and the fix is different.
6. Now measure the same route by hand: open the app on a phone, throttle the network to slow 4G in developer tools, and time the action with a stopwatch three times. Compare with the server's p95.
7. Open one trace for a slow request, or brief AI to add tracing to that one route if you have none. Read the waterfall and write one sentence saying where the time went.
8. In that waterfall, look for a step that repeats — the same query many times. If you find one, you have found an N+1, and that is your highest-value finding of the day.
9. Build or trim your dashboard to exactly four charts: latency p50 and p95 by route, requests per minute, error rate as a percentage, and your nearest free-tier quota.
10. Write down, as a sentence, what number would make you stop building features and fix performance instead. That sentence is the start of M05's error budget.
11. Put the four numbers and the date in your decision log, so in a month you can tell whether you got faster or just got used to it.

## Talk to AI
```prompt
CONTEXT:      I am a student in a team of three. I direct AI and verify its
              output; I do not write code. Our product: [describe it in two
              sentences]. Stack: [your stack] on [platform], free tier, with
              [your database] and [any external providers you call, e.g. a
              payment provider and an AI provider]. Our users are on budget
              Android phones in India, frequently on slow 4G. Our most
              important user actions are: [list three or four].

GOAL:         A four-chart dashboard I will actually look at every morning,
              built on percentiles rather than averages, plus tracing on the
              single slowest user action.

REQUIREMENTS: 1. Explain p50, p95 and p99 in plain English, using my own
                 user actions as the example, and say what each one tells me
                 about a real student using my app.
              2. Tell me, for each of my user actions, what should be
                 measured and whether it needs a counter, a gauge or a
                 histogram. Be explicit about which measurements must be
                 histograms for p95 to be possible at all.
              3. Specify the four charts — latency, traffic, errors,
                 saturation — in terms of my product: what exactly is on
                 each axis, split by what, over what window. For
                 saturation, tell me what my real limiting resource is on a
                 free tier.
              4. Tell me what my server-side p95 will NOT include for a
                 user on slow 4G, and how I can measure the difference by
                 hand with a browser and a stopwatch.
              5. Explain traces and spans using one of my user actions, and
                 list the spans I should expect to see. Tell me what an N+1
                 would look like in that waterfall.
              6. Warn me about cardinality: name three fields in my product
                 that must never become metric labels, and say what would
                 happen on a free tier if they did.

CONSTRAINTS:  Free tier only, no paid vendors. Explain every term on first
              use. Assume I verify in a browser. No code until I approve
              the chart specification.

NON-GOALS:    Do not design alerting — that is the next chapter. Do not
              cover user analytics or funnels. Do not add charts beyond the
              four, however tempting. Do not modify any files yet.

ACCEPTANCE:   - I can say what each of the four charts would look like
                during an outage, and during a quiet night
              - Every latency measurement you specify can produce a p95
              - I know one measurement I can only get with a stopwatch
              - I can describe an N+1 in my own waterfall in one sentence
              - I can name three fields that must not be metric labels

ASK:          Ask me up to five questions about my routes, providers and
              database that would change the design. Then give me the
              chart specification only, and stop before any
              implementation plan.
```

## Stuck? Ask AI
```prompt
Give me a list of twenty request durations in milliseconds for one endpoint. Walk me through computing the mean, the p50 and the p95 by hand, step by step. Then change two of the twenty numbers to very large values and show me which of the three moves and which barely does, and explain what that means for a user complaining about slowness.
```

```prompt
Explain the difference between a counter, a gauge and a histogram using things in a college canteen. Then give me ten things I might want to measure in a food-ordering app and ask me which type each needs, one at a time, without telling me the answers first.
```

```prompt
Here is a trace waterfall from my app: [paste or describe the spans and their durations]. Explain, step by step, how to read it. Tell me where the time actually went, whether any step looks like an N+1, and what the fix would be for the largest span — including whether the fix is in my code at all.
```

## Quiz

### Q1 · L4
Your average response time is 480 ms and your p95 is 4.2 s. A user says the app is slow. Who is right?
- (a) The dashboard — half a second is acceptable
- (b) Both; the average describes requests, the p95 describes her experience ✅
- (c) The user, because averages are always wrong
- (d) Neither; you need the p99 before deciding
> One request in twenty exceeding four seconds means a student touching the app fifteen times a session meets the slow version repeatedly. The average is arithmetic about requests; the complaint is about people.

### Q2 · L4
You want a p95 for checkout, but the only thing recorded is a running average. What has to change?
- (a) Nothing; p95 can be derived from the average and the request count
- (b) The duration must be recorded as a histogram from the start ✅
- (c) You need a longer retention window on the metric
- (d) You must split the average by route first
> A mean discards the distribution, so no percentile can be recovered from it afterwards. Percentiles require bucketed measurements, which is why "track response time" must be briefed as "record a histogram".

### Q3 · L5
A trace of a nine-second checkout shows 7.9 s inside the call to the payment provider. What does this change about the fix?
- (a) Nothing; nine seconds is nine seconds
- (b) The work moves to timeouts, retries and a pending state, not your own speed ✅
- (c) You should add database indexes to compensate
- (d) You should cache the payment response
> You cannot make somebody else's system faster. The trace redirects effort to M05's reliability tools and to an interface that behaves honestly while waiting — and it stops you optimising code that was never the problem.

### Q4 · L3
Your error count is zero and the team declares the app healthy. What is missing from the dashboard?
- (a) A saturation chart
- (b) Traffic, since zero errors with zero users looks identical ✅
- (c) A p99 latency line
- (d) A log of successful requests
> Errors must be read as a rate against traffic. Without request volume on the same screen, an outage that stops all requests reads as perfect health.

### Q5 · L4
Which of these would most likely exhaust a free-tier metrics quota?
- (a) Recording request duration as a histogram per route
- (b) Labelling latency with the user id ✅
- (c) Charting requests per minute for 90 days
- (d) Adding a saturation chart for database connections
> Each distinct label value becomes its own time series, so a user-id label multiplies your series count by your user count. Identifiers belong in logs and traces; metrics take categories.

## Go deeper
- [Google SRE Book: the four golden signals](https://sre.google/sre-book/monitoring-distributed-systems/#xref_monitoring_golden-signals)
- [OpenTelemetry: traces](https://opentelemetry.io/docs/concepts/signals/traces/)
