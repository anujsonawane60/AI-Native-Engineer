---
id: m07-04-monitoring-vs-observability
module: m07-security-observability-analytics
slug: monitoring-vs-observability
title: "Monitoring vs Observability: Logs, Metrics and Traces"
order: 4
duration_min: 25
is_free: false
mission: false
volatility: low
last_reviewed: 2026-11-16
---

## Key learnings
- State the difference between monitoring and observability in one sentence
- Choose the right one of logs, metrics and traces for a given question
- Explain why "it works on my machine" is not evidence about production
- Recognise a question your current setup cannot answer
- Describe what each pillar costs you in money and in storage

## Lesson

### The pain

A student gets a message at 9pm: "your app is broken". That is the entire report. They open the app on their own phone and it works perfectly.

So begins the evening. Is it broken for everyone, or for that one person? Was it broken ten minutes ago and fine now? Is it the app, the database, the payment provider, or that user's 4G? Was it slow or did it show an error? Which page? They cannot answer any of it, because the only instrument they have is their own phone, and their own phone says everything is fine.

They eventually ask the user for a screenshot, get one at 11pm, and discover the order page failed for anyone whose name contained an apostrophe. It had been failing for nine days. Thirty-one students had hit it. Nobody had told them, and nothing had recorded it.

The code was fine. The blindness was the problem.

### The idea

**Monitoring answers questions you knew to ask in advance. Observability is having enough recorded detail to answer questions you had not thought of.**

Monitoring is a fixed set of dials: is the site up, how many errors per minute, how slow is the database. You decided those in advance and you watch them. Observability is the property of a system where, when something strange happens, the data to explain it already exists — because you recorded enough context at the time.

You need both. Monitoring tells you something is wrong. Observability tells you what.

### The analogy

A well-run canteen has exactly three instruments, and they are the three pillars of observability.

**The daily sales register** is your **logs**. One line per thing that happened, written at the moment it happened, with details: time, item, quantity, who served it. Dense, specific, and you read it when you need to reconstruct a particular event. It is also enormous after a month.

**The kitchen thermometer** is your **metrics**. One number, sampled over and over, and what matters is the shape over time rather than any single reading. 180°C is meaningless alone; 180°C when it should be 140°C and climbing for twenty minutes is an incident. Cheap to keep, because a number per minute is tiny.

**The CCTV** is your **traces**. Follow one customer from the door, through the queue, to the counter, to the payment, to the table. Where did *this particular person's* twenty minutes actually go? The register tells you they were served; only the CCTV shows that eleven of those minutes were spent waiting at the payment counter.

**Where the analogy breaks:** CCTV records continuously and cheaply. Tracing every single request in a real app produces more data than the app itself, so you sample — one request in ten, or one in a hundred, plus every request that errored. Which means a trace for the specific request you care about may simply not exist, and you have to reproduce the problem to get one. Logs and metrics have no such gap; traces are the pillar you have to decide how much of to keep.

### Picture it

```
            A QUESTION YOU HAVE
                    │
      ┌─────────────┼──────────────┐
      ▼             ▼              ▼
 "did THIS      "is this       "where did
  one thing      getting        THIS request
  happen?"       worse?"        spend time?"
      │             │              │
      ▼             ▼              ▼
 ┌─────────┐   ┌──────────┐   ┌──────────┐
 │  LOGS   │   │ METRICS  │   │  TRACES  │
 │ register│   │thermometer│  │  CCTV    │
 └─────────┘   └──────────┘   └──────────┘
  one line      one number     one request,
  per event     over time      end to end
```

The three pillars are not three copies of the same data; each answers a different shape of question, and picking the wrong one is why people stare at dashboards and learn nothing.

### Which one answers which question

| Your question | The pillar | Why |
|---|---|---|
| Did student 4412's payment actually reach us? | Logs | You need one specific event with its details |
| Are errors rising since this morning's deploy? | Metrics | You need a shape over time, not an event |
| Why does checkout take nine seconds for some people? | Traces | You need the breakdown within one request |
| How many orders failed yesterday? | Metrics | A count over a window |
| What exactly did the database say when it failed? | Logs | The detail lives in the message |
| Is the slowness us or the payment provider? | Traces | Only a trace splits the time between them |

A useful habit: before adding instrumentation, write the question down. Teams that start from "what will we want to know at 9pm on a Friday?" end up with far less data and far more answers than teams that start from "let us log everything".

### Where it lives

All three run on the server, not in the user's browser — which is exactly why "it works on my machine" is not evidence. Your machine is one device, one network, one set of data, and one user: yours. M06 showed you logs on a server and in CI; this module is about what to put in them and what to do with them.

The next three chapters take one pillar each: logs in chapter five, metrics and traces in chapter six, and chapter seven is what happens when one of them crosses a line and a human needs to know. M05's work on p95, retries and error budgets is the same subject from the reliability side — those numbers only exist if something is measuring them. In M11's going-live checkpoint you are asked to show working monitoring before you call the product live, and in M12 the same pillars get a mobile-specific form as crash reporting.

One thing to be clear about now: this half of the module is about the health of your *system*. Chapters eight and nine are about the behaviour of your *users*. They look similar, they often use the same events, and they answer completely different questions.

### Words to own

| Term | Plain meaning | Example |
|---|---|---|
| Monitoring | Watching a fixed set of known indicators | Uptime, error rate, latency |
| Observability | Having enough recorded detail to ask new questions | Logs with a request id on every line |
| Log | One written record of one event | "order 8421 failed: timeout" |
| Metric | One number sampled repeatedly over time | Errors per minute |
| Trace | The end-to-end record of one request | Checkout took 9.2s, 7.1s in the database |
| Span | One step inside a trace | "database query: 7.1s" |
| Instrumentation | The code that emits the above | Added once, mostly by a library |
| Cardinality | How many distinct values a field can take | User id: millions. Status: five |
| Retention | How long you keep the data | Logs 7 days, metrics 90 days |
| Sampling | Keeping a fraction of the data | 1 trace in 20, plus all errors |

### Tradeoffs and gotchas

**Observability costs money per byte, and the bill is real.** Logs are the expensive pillar because they are text and there is one line per event. The free tiers this course runs on have limits in the low gigabytes, and a chatty debug log can burn a month's allowance in a day. Metrics are cheap because a number is small. Traces are in between, and that is why sampling exists.

**Dashboards nobody opens are worse than no dashboards.** They create a feeling of being instrumented. A single dashboard with five charts that your team genuinely looks at every morning beats forty charts in a tab nobody has opened since the day they were built.

**High cardinality is where the costs hide.** Putting a user id on a *log line* is useful. Putting a user id on a *metric* creates one separate time series per user, which is how a free tier becomes a bill. The rule of thumb: identifiers belong in logs and traces, categories belong in metrics.

**You cannot add observability during an incident.** The data has to have been collected before the thing happened. This is why the walkthrough below is a day-one task, not a day-ninety one, and why the question "what will I wish I had recorded?" is worth ten minutes now.

**Averages lie, and the next chapter but one is about why.** A dashboard showing average response time is almost worse than nothing, because it looks like information. Keep that discomfort until chapter six.

### Verify it

AI asked to "add logging and monitoring" will produce something that runs. Five checks to find out whether it produces anything you can use.

1. **Ask a real question and try to answer it.** "Did this specific order fail, and why?" Open whatever was built and try. If you cannot get to an answer in two minutes, you have instrumentation rather than observability.
2. **Check that it is recording from the server, not the browser.** Browser-only error tracking misses everything that happens after the request leaves the phone, which is where most failures live.
3. **Check there is one id tying a request's records together.** Without a request id on every line, you have a pile of unrelated sentences and no way to assemble one story. This is the single most valuable thing to insist on.
4. **Make something fail on purpose and go find it.** Break a database connection string, submit an apostrophe in a name, cut your network mid-request. If a failure you caused yourself does not appear anywhere, nothing a user causes will either.
5. **Read the retention and the quota.** How long is data kept, and how much before it stops recording or starts charging? A log store that silently dropped everything after day three is a common and invisible failure.

Red flags: a setup that only records errors the browser noticed; log lines with no timestamp, no request id and no user context; an "observability" setup that is one average-latency chart; **an error message that returns the raw database error** to the user rather than to the log, which is both a security flaw from chapter one and a sign that nobody has decided what belongs where; and a free-tier quota already exhausted before you had your first incident.

## Walkthrough
1. Write down five questions you would actually want answered at 9pm if a user said your app was broken. Be specific to your product — "did Priya's UPI payment reach us?" not "is it working?".
2. Beside each question, write which pillar answers it: logs, metrics or traces. If you cannot decide, the question is too vague — rewrite it.
3. Open your deployed app's hosting platform and find where server logs appear. Load a page, then find the record of your own request. Note how long it took you.
4. Search those logs for the word "error" over the last seven days. Read whatever comes back, including things you did not know were happening.
5. Find out the retention period and the free-tier quota for that log store. Write both numbers in your notes; most teams have never looked.
6. Now break something deliberately: submit an apostrophe in a text field, or point a configuration value at a wrong address. Go and find the resulting record in the logs.
7. Check whether that record tells you who it happened to and which request it belonged to. If not, note that as your first gap.
8. Find whatever metric your platform already gives you for free — request count, error rate, response time — and look at the last 24 hours. Note the shape, not the number.
9. Pick the one question from step 1 that you currently cannot answer at all, and write a one-paragraph brief for what would have to be recorded to answer it.
10. Put the five questions in your repository as a file. Chapters five to seven are you making them answerable, one at a time.

## Talk to AI
```prompt
CONTEXT:      I am a student in a team of three. We direct AI and verify its
              output; we do not write code. Our product: [describe it in two
              sentences]. Stack: [your stack], deployed on [platform], free
              tier, with [your database]. Our users are on budget Android
              phones in India, often on slow 4G. Right now our only way of
              knowing whether the app works is opening it ourselves.

GOAL:         I want to understand what I can already answer about my
              running app, what I cannot, and the smallest set of additions
              that would close the most important gaps.

REQUIREMENTS: 1. Explain monitoring and observability in plain English, and
                 give me one question from an app like mine that each one
                 answers but the other does not.
              2. Take these five questions I want answered: [paste your
                 five from the walkthrough]. For each, tell me which of
                 logs, metrics or traces answers it, and why the other two
                 cannot.
              3. Tell me which of the three pillars my platform and
                 database already give me for free, and where I would look.
              4. For each pillar, tell me what it costs: in storage, in
                 money, and in free-tier quota. Say which one will run out
                 first for a student project and roughly when.
              5. Explain cardinality in plain English, and give me three
                 fields from my product that belong in logs but must never
                 become metric labels.
              6. Give me the smallest possible first step — one change —
                 that would most improve my ability to answer questions at
                 9pm on a Friday, and say why that one before the others.

CONSTRAINTS:  Free tiers only, no paid vendors. Assume I will not run
              anything on my own machine. Explain every term the first time
              you use it. Do not give me code yet.

NON-GOALS:    Do not set anything up. Do not compare vendors or recommend a
              product. Do not cover user analytics, funnels or product
              metrics — this is about system health only.

ACCEPTANCE:   - For each of my five questions I know the pillar and the gap
              - I can say which pillar my free tier will exhaust first
              - I can name three fields that must not become metric labels
              - Your single recommended first step is justified against the
                alternatives you rejected

ASK:          First ask me up to five questions about my stack and users
              that would change your answer. Then give me the mapping table
              for my five questions and stop before the recommendation.
```

## Stuck? Ask AI
```prompt
Explain logs, metrics and traces using the analogy of a canteen's sales register, kitchen thermometer and CCTV. Then give me eight questions a canteen owner might ask and make me say which instrument answers each one, one question at a time, without telling me the answer first.
```

```prompt
I do not understand why "it works on my machine" is not evidence that my app works. List every way my laptop differs from a real user's situation — device, network, data, account, location, time — and for each one give me an example of a failure that only appears on their side.
```

```prompt
My app is deployed on [platform] with [database] on free tiers. Tell me exactly where to click to find server logs, error counts and response times, and what the free-tier limit and retention period are for each. If you are not certain about current limits, say so and tell me where in the platform's own settings to read the real number.
```

## Quiz

### Q1 · L3
A user says "the app is broken", and your app works on your phone. Which pillar first tells you whether the problem is widespread or individual?
- (a) Traces, because they follow one request end to end
- (b) Metrics, because a rate over time shows how many are affected ✅
- (c) Logs, because they contain the error message
- (d) None; you must reproduce it on your own device first
> Error rate over time answers "how many and since when" in one glance. You then go to logs for the message and to a trace for where the time or the failure went.

### Q2 · L4
Why is putting a user id on a metric label a costly mistake, when putting it on a log line is good practice?
- (a) Metrics cannot store text values
- (b) Each distinct label value creates its own time series, so cost explodes ✅
- (c) Log lines are compressed and metrics are not
- (d) It breaks the ability to average across users
> This is cardinality. Identifiers belong in logs and traces, where they cost one field on one line; as metric labels they multiply the number of series by your user count.

### Q3 · L4
Your team has one average-response-time chart and calls it observability. What is the strongest objection?
- (a) It should be a median rather than a mean
- (b) It answers one pre-decided question and no new ones ✅
- (c) Charts are less useful than logs for debugging
- (d) It needs a second chart for request count
> Observability is the ability to ask questions you had not planned. One aggregate number is monitoring at best, and chapter six shows that an average actively hides the users having the worst time.

### Q4 · L5
An order page has been failing for nine days for users whose name contains an apostrophe, and nobody noticed. Which single addition would most likely have caught it?
- (a) A daily manual check of the app by one team member
- (b) Server-side error recording with an error-rate chart ✅
- (c) A trace on every checkout request
- (d) A longer log retention period
> A steady trickle of server errors shows up immediately on a rate chart and in the logs. Manual checks miss anything that does not happen to you, and traces explain slowness rather than announce failure.

### Q5 · L3
Why can observability not be added during an incident?
- (a) Deploys are blocked while an incident is open
- (b) The data has to have been recorded before the event happened ✅
- (c) Free tiers rate-limit new instrumentation
- (d) Changing instrumentation resets the dashboards
> Instrumentation is a decision made in advance; there is no way to recover context that was never written down. Hence the day-one habit of asking what you will wish you had recorded.

## Go deeper
- [Google SRE Book: Monitoring Distributed Systems](https://sre.google/sre-book/monitoring-distributed-systems/)
- [OpenTelemetry: observability primer](https://opentelemetry.io/docs/concepts/observability-primer/)
