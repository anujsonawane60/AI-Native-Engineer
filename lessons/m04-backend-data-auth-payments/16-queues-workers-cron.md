---
id: m04-16-queues-workers-cron
module: m04-backend-data-auth-payments
slug: queues-workers-cron
title: "Background Work: Queues, Workers, Cron and Event-Driven Patterns"
order: 16
duration_min: 15
is_free: false
mission: false
volatility: low
last_reviewed: 2026-11-16
---

## Key learnings
- Decide what must happen inside a request and what must be moved out of it
- Trace a job from enqueue to worker to retry to dead-letter queue
- Make a job safe to run twice, because it will be
- Choose between a queue, a scheduled job and an event for a real feature
- Verify a background system by making a job fail on purpose

## Lesson

### The pain

A student taps "Place order". The server saves the order, emails a confirmation, generates a PDF receipt, notifies the canteen's WhatsApp group, then replies — four seconds on a good day. On the day the WhatsApp provider has an outage the host kills the request at ten seconds, and she sees an error even though her order was saved in 80 milliseconds. She taps again; now there are two orders. The team's fix is a try-catch that ignores failures, so nobody finds out emails stopped sending three days ago.

### The idea

**A request handler should do the smallest amount of work that makes the user's action true, and hand everything else to a worker.**

A **queue** is a list of jobs waiting. The handler **enqueues** a tiny record — "send confirmation for order 91" — and returns immediately; a **worker** drains the queue, retrying on failure, with nobody waiting. **Cron** is work triggered by a clock, not a user. **Event-driven** is the generalisation, and harder to debug.

The test for what stays is one question: **is the user's action true without this?**

| Work | Inside the request? | Why |
|---|---|---|
| Save the order row | yes | the action itself |
| Recompute the price server-side | yes | correctness, and it is fast |
| Check stock and reserve it | yes | two students, one samosa |
| Send a confirmation email | no | informs, takes seconds, can retry |
| Generate a PDF receipt | no | slow, nobody is waiting for it |
| Resize an uploaded image | no | seconds to minutes |
| Notify a WhatsApp group | no | third party, can be down |
| Update a daily sales total | no | cron can recompute it |

### The analogy

The cashier takes your money, writes a token, and hands it over in eight seconds; she does not cook your food while you stand there. The slip goes on a spike by the kitchen — that is the queue — and the cooks work through it. A failed dish goes back on the spike; after three failures it goes in the manager's tray, the dead-letter queue.

**Where the analogy breaks:** a cook who drops a slip sees the gap, but a queue failing silently looks exactly like a quiet day. "Nothing appears to be wrong" is also what a stopped worker looks like.

### Picture it

```
 request handler                   queue            worker
 ┌────────────────────┐         ┌──────────┐      ┌──────────┐
 │ save order    80ms │         │ job 1    │      │ take job │
 │ enqueue 3 jobs 5ms │────────▶│ job 2    │─────▶│ do work  │
 │ respond 201        │         │ job 3    │      │ ack      │
 └────────────────────┘         └──────────┘      └────┬─────┘
       student waits                                   │ fails?
       85ms, not 4s                  ┌─────────────────┘
                                     ▼
                              retry after 1s, 2s, 4s
                                     │ still failing
                                     ▼
                              ┌──────────────────┐
                              │ DEAD LETTER      │
                              │ a human looks    │
                              └──────────────────┘
```

The handler does the one thing that must be true and enqueues the rest; a worker drains the queue, retrying with increasing delays, and gives up into a dead-letter queue a human reviews.

### Where it lives

The worker is a second place your code runs — usually a scheduled function draining a table every minute. **A Postgres table with a status column is a perfectly respectable queue**: rows marked `pending`, `processing`, `done`, `failed`, and a worker that picks the oldest pending one.

Chapter 5's timeout is why this exists; chapter 11's transaction is where the job insert belongs; the webhook, the resize and reconciliation are all jobs; M07 is where depth becomes an alert.

### Words to own

| Term | Plain meaning | Example |
|---|---|---|
| Queue | A list of jobs waiting to be done | pending rows |
| Enqueue | Adding a job to the queue | in the transaction |
| Worker | The process that does queued jobs | a scheduled function |
| Acknowledge / ack | Telling the queue a job completed | marks it done |
| Retry with backoff | Trying again after growing delays | 1s, 2s, 4s |
| Dead-letter queue | Where jobs go after repeated failure | a human reviews it |
| Idempotent | Safe to run twice | "set status to paid" |
| Cron | Work triggered by the clock | nightly at 10 p.m. |
| Event | An announcement others may subscribe to | `order.placed` |
| Queue depth | How many jobs are waiting | the number to watch |

### Tradeoffs and gotchas

**Every job will run twice, so every job must be idempotent.** A worker does the work, then crashes before acknowledging it, and the queue hands the job to another worker — normal behaviour, because acknowledging before working loses jobs entirely. Crediting a wallet twice is a loss. Give each job a unique key, or make it naturally idempotent: "set status to paid" is safe twice, "add ₹240 to the balance" is not.

**Enqueue inside the transaction,** or a crash between commit and enqueue loses the job silently.

**Never swallow job failures.** A try-catch that logs nothing hides a broken email system. Retry a network timeout; send a malformed job straight to the dead-letter queue.

**Free-tier cron is not guaranteed.** Write jobs that catch up — "process everything unprocessed", not "the last hour" — so a missed run self-heals.

**Watch the depth.** A queue growing faster than it drains is a slow-motion outage; "Processing" beats a blank space.

### Verify it

Five checks; the second tells you whether you have a real system.

1. **Time the request.** More than about 500 ms means something slow is still in the handler.
2. **Make a job fail on purpose.** Did it retry? How many times? Where is it now? If you cannot answer those three by looking at something, you will not notice real failures.
3. **Run a job twice, deliberately.** Two of anything that should be one is a missing idempotency key.
4. **Kill the worker mid-job.** On restart it must complete or retry, with nothing done twice that matters.
5. **Skip a cron run.** A well-designed job catches up; one written against "the last hour" loses it.

Red flags: a third-party call in a request handler; an enqueue after the commit; a try-catch around job execution that logs and continues; no dead-letter queue; no idempotency key on a job touching money; a cron job assuming it ran on time and exactly once; retries with no maximum; no number showing how many jobs wait.

## Walkthrough
1. Pick your most important write action — placing an order, submitting a form, completing a signup. List every single thing your code currently does inside that request, in order.
2. Beside each item, answer the one question: is the user's action true without this? Mark each as "must be in the request" or "can be a job".
3. Time the request as it stands today, using the Network tab's server timing. Write the number down.
4. Brief AI to move everything you marked as a job into a queue, with the explicit requirement that the enqueue happens inside the same transaction as the main write.
5. Re-time the request. Compare the two numbers. This is the user-visible value of the whole chapter.
6. Find where the worker runs and how often. Write it down — a scheduled function every minute, a managed queue consumer, whatever it is.
7. Look at your job records. Confirm there is a status, an attempt count, a last error, and a created timestamp. If any is missing, you cannot diagnose a failure.
8. Break a job on purpose. Give the email provider wrong credentials, or make the job throw an error. Watch what happens over the next few minutes: how many retries, with what delays, and where the job ends up.
9. Find the failed job. If you cannot find it by looking at a table or a dashboard, that is the gap to fix before anything else.
10. Re-run a completed job manually. Count what it produced. If you get two emails or two credits, write the brief for an idempotency key.
11. Create one cron job your product genuinely needs — a nightly summary, or a sweep for stuck records — written to process everything unprocessed rather than a fixed window, and test it by skipping a run. Then record in your decision log: the before and after request times, your retry policy and maximum, where failed jobs go, and who looks at them.

## Talk to AI
```prompt
CONTEXT:      Three-person student team, free tiers, serverless functions
              with roughly a 10-second timeout, managed PostgreSQL. Users
              on budget Android phones and slow 4G.
              Product: [one sentence]

              MY MOST IMPORTANT WRITE ACTION and everything the handler
              currently does, in order:
              [paste the handler code, or list the steps]

              Current server response time for it: [measured number]

              THIRD PARTIES WE CALL: [email provider, messaging, payment
              gateway — whichever apply]

              Scheduled work we need: [e.g. nightly sales summary, sweep
              for stuck payments, cleanup of orphaned files]

GOAL:         Get everything out of this request that does not make the
              user's action true, into a queue that retries, fails loudly
              and is safe to run twice.

REQUIREMENTS: 1. For each step in my handler, tell me whether it must stay
                 in the request, using the test "is the user's action true
                 without this?" — and justify each decision
              2. Design the simplest queue that works on my free tier.
                 Consider a Postgres table with a status column and say
                 honestly whether that is good enough for me
              3. The enqueue must happen inside the same transaction as the
                 main write. Explain exactly what goes wrong if it does not
              4. Specify the job record's fields so I can diagnose a
                 failure: status, attempts, last error, timestamps
              5. Give me the retry policy: delays, maximum attempts, which
                 errors are worth retrying and which should go straight to
                 the dead-letter queue
              6. For each of my jobs, tell me whether it is naturally
                 idempotent. For the ones that are not, specify the
                 idempotency key and where it is recorded
              7. Tell me what the user sees while a job is still pending,
                 for each job
              8. For my scheduled work, write each job to process
                 everything unprocessed rather than a fixed time window,
                 and tell me how I would test a skipped run
              9. Give me the one number I should watch to know the system
                 is healthy, and what a bad value looks like

CONSTRAINTS:  Free tier only. No paid queue service. No always-on worker
              process — assume a scheduled function. Do not exceed the
              10-second timeout in either the handler or the worker; tell
              me how a long job is split if needed.

NON-GOALS:    Do not build an event bus or a pub-sub system. Do not add
              monitoring dashboards — that is a later module. Do not change
              the schema beyond what the queue itself requires. Do not
              touch any other endpoint.

ACCEPTANCE:   - My request response time drops to under 500 ms
              - I can force a job failure and then find that job, see its
                attempt count and read its last error
              - Re-running a completed job produces no duplicate effects
              - A skipped cron run self-heals on the next run
              - Nothing slow or third-party remains in the request handler

ASK:          Start by classifying every step in my handler with your
              reasoning, and tell me which of my jobs are NOT safe to run
              twice. I want to see that list before any design.
```

## Stuck? Ask AI
```prompt
Explain why a queue will sometimes run the same job twice, and why that is normal rather than a bug. Use a canteen kitchen with order slips on a spike as the analogy. Then give me five jobs from a food-ordering app and ask me which ones are safe to run twice, without telling me the answers first.
```

```prompt
Here is my request handler: [paste it]. For each thing it does, ask me whether the user's action is true without it. Do not propose a design yet — just work through the list with me one step at a time.
```

```prompt
Explain what a dead-letter queue is and why swallowing job failures is worse than letting them fail. Then tell me the three questions I must be able to answer about any failed job, and what I need to record in order to answer them.
```

## Quiz

### Q1 · L3
Which of these belongs inside the request handler rather than in a queue?
- (a) Sending the confirmation email
- (b) Generating a PDF receipt
- (c) Checking stock and reserving the item ✅
- (d) Notifying the canteen's WhatsApp group
> The test is whether the user's action is true without it. Stock reservation determines correctness — two students must not get the same last samosa — so it stays in the request, inside the transaction.

### Q2 · L3
Why must the enqueue happen inside the same transaction as the main write?
- (a) It makes the request faster
- (b) Otherwise a crash between the commit and the enqueue loses the job silently ✅
- (c) Queues require transactional guarantees to accept jobs
- (d) It prevents the job from running twice
> An order committed with no confirmation job queued is a silent gap with no error anywhere. Using a table in the same database makes the insert part of the same transaction, which is a real argument for the simple approach.

### Q3 · L4
A worker does the work, then crashes before acknowledging the job. What happens, and what must you have designed for?
- (a) The job is lost; you need a backup queue
- (b) The queue hands it to another worker and the work happens twice, so jobs must be idempotent ✅
- (c) The queue detects the completed work and marks it done
- (d) Nothing; the acknowledgement is implicit once work begins
> Re-delivery is normal queue behaviour, because the alternative — acknowledging before working — loses jobs entirely. "Set status to paid" is safe twice; "add ₹240 to the balance" is not.

### Q4 · L4
Your cron job is written to "process the last hour of orders" and the platform skips a run. What is the consequence?
- (a) The next run processes two hours automatically
- (b) That hour is silently lost forever ✅
- (c) The platform retries the missed run later
- (d) The job fails loudly and alerts you
> Free-tier schedulers are not guaranteed. Jobs written as "process everything unprocessed" self-heal after a missed run; jobs written against a fixed window do not, and nothing fails to tell you.

### Q5 · L5
You ask whether a background system works. Which answer is actual evidence?
- (a) The jobs are visible in the queue table and the code looks correct
- (b) You broke a job on purpose and can show its retry count, its last error and where it ended up ✅
- (c) The request response time dropped below 500 ms
- (d) The AI confirmed retries are configured with exponential backoff
> A background system is only as good as your ability to see it fail. If you cannot answer "did it retry, how many times, where is it now?" by looking at something, you will not notice real failures either.

## Go deeper
- [PostgreSQL docs: SELECT ... FOR UPDATE SKIP LOCKED, the simple queue pattern](https://www.postgresql.org/docs/current/sql-select.html)
- [AWS docs: dead-letter queues and why jobs end up there](https://docs.aws.amazon.com/AWSSimpleQueueService/latest/SQSDeveloperGuide/sqs-dead-letter-queues.html)
