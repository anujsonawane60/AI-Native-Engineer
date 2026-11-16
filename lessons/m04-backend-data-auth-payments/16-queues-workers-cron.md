---
id: m04-16-queues-workers-cron
module: m04-backend-data-auth-payments
slug: queues-workers-cron
title: "Background Work: Queues, Workers, Cron and Event-Driven Patterns"
order: 16
duration_min: 25
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

A student taps "Place order". The server saves the order, sends a confirmation email, generates a PDF receipt, resizes the dish photo for the order history, and notifies the canteen's WhatsApp group. Then it replies.

Four seconds on a good day. Nine when the email provider is slow. On the day the WhatsApp provider has an outage, the request hangs for thirty seconds, the serverless host kills it at ten, and the student sees an error — even though the order was saved perfectly in the first 80 milliseconds. She taps again. Now there are two orders.

The team's fix is to wrap the whole thing in a try-catch and ignore failures. Now the order succeeds and nobody ever finds out that emails stopped sending three days ago.

Meanwhile the canteen wants a daily 10 p.m. summary of sales. Somebody implements it as an endpoint and sets a phone alarm to open it in a browser every night. It works until the day they forget, and nobody notices for a week because nothing fails loudly.

### The idea

**A request handler should do the smallest amount of work that makes the user's action true, and hand everything else to a worker.**

A **queue** is a list of jobs waiting to be done. The request handler **enqueues** a job — a tiny record saying "send confirmation for order 91" — and returns immediately. A **worker** is a separate process that takes jobs off the queue and does them, slowly if necessary, retrying on failure, with nobody waiting.

**Cron** is the other shape: work triggered by a clock rather than by a user. The 10 p.m. summary, the nightly cleanup of orphaned files from chapter 15, the hourly check for payments stuck in pending.

**Event-driven** is the generalisation. Instead of the order handler knowing about emails, PDFs and WhatsApp, it announces "order placed" and whatever cares subscribes. Powerful, and genuinely harder to debug, which is why it belongs later than you think.

### The analogy

At the canteen counter, the cashier takes your money, writes a token, and hands it to you. Eight seconds. She does not cook your food while you stand there.

The order slip goes on a spike by the kitchen — that is the queue. The cooks work through the spike in order — they are the workers. If a dish fails because the gas ran out, the slip goes back on the spike to be retried. If it fails three times, it goes in a separate tray the manager looks at — the dead-letter queue.

The nightly count of the till is cron: nobody requests it, the clock does.

**Where the analogy breaks:** a cook who drops a slip can see the gap. A queue failing silently looks exactly like a quiet day — no slips, no errors, no symptom. You need to watch the queue's depth and its failure tray deliberately, which is M07's observability, because "nothing appears to be wrong" is also what a stopped worker looks like.

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

The queue sits beside your server rather than in front of it, and the worker is a second place your code runs. On free tiers this is usually a scheduled function that runs every minute and drains a database table used as a queue, or a managed queue your platform provides. For a team of three, **a table in Postgres with a status column is a perfectly respectable queue** — rows with `pending`, `processing`, `done`, `failed`, and a worker that picks the oldest pending row.

It connects backwards to almost every chapter in this module. Chapter 5's ten-second timeout is why this exists. Chapter 11's transactions matter because the job insert must be part of the same transaction as the order — otherwise you can commit an order whose confirmation job was never queued. Chapter 15's resize is a job. Chapter 3's webhook handler verifies, records and queues, because the provider is waiting. And chapter 17's payment reconciliation is the most important cron job you will write.

Forward: M05 covers retries, backoff and circuit breakers properly. M07 is where you watch queue depth and alert on the dead-letter queue. M13 is where event-driven architecture and Kafka become a real conversation at a scale where they earn their complexity.

### What belongs in a request, and what does not

The test is one question: **is the user's action true without this?**

A student placed an order. The order row makes that true. The confirmation email does not — it informs her of something already true. So: save the row, queue the email, respond.

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

The pattern that falls out: anything involving a third party, anything slow, and anything the user is not literally staring at goes in the queue. Anything that determines correctness stays in the request, inside the transaction.

### Jobs run twice. Design for it.

This is the part that catches everyone, and it is the same lesson chapter 17 builds on.

A worker takes a job, does the work, and then crashes before it can mark the job done. The queue, seeing no acknowledgement, hands the job to another worker. The work happens twice. This is not a rare edge case — it is the normal behaviour of every queue, because the alternative (marking done before doing the work) loses jobs entirely, which is worse.

So every job must be **idempotent**: running it twice produces the same result as running it once.

Sending an email twice is annoying. Crediting a wallet twice is a loss. The techniques are simple once you know to apply them: give each job a unique key and record that key when it completes, so the second run sees it and stops. Or make the operation naturally idempotent — "set status to paid" is safe twice, "add ₹240 to the balance" is not.

The same discipline applies to cron. A nightly job that runs twice because the scheduler double-fired must not send two summaries, and a job that takes eleven minutes when it runs every ten minutes must not have two copies racing each other.

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

**Enqueue inside the transaction.** If you commit the order and then enqueue, a crash between the two loses the job silently. If the queue is a table in the same database, the insert is part of the same transaction and the problem disappears — which is a real argument for the simple approach.

**Never swallow job failures.** A try-catch that logs nothing turns a broken email system into a system nobody knows is broken. Failures go to a dead-letter queue and somebody looks at it.

**Retry only what is worth retrying.** A network timeout is worth retrying. A malformed job that will fail identically forever is not — it belongs in the dead-letter queue immediately. Infinite retries on a permanent failure burn your free tier and bury the real failures.

**The user needs to see the in-between state.** If the receipt is generating, say so. "Processing" with a spinner is honest; a blank space looks broken and generates support messages.

**Cron on free tiers is often minute-granularity at best, and not guaranteed.** A job scheduled for exactly 10 p.m. may run at 10:02, or be skipped if the platform is busy. Design jobs that catch up — "process everything unprocessed" rather than "process the last hour" — so a missed run self-heals.

**Do not go event-driven early.** Announcing `order.placed` and having four subscribers is elegant until you need to know why one of them did not fire. With three people and no users, a direct queue with named jobs is easier to reason about and to debug.

**Watch the depth.** A queue that is growing faster than it drains is a slow-motion outage. One number on a dashboard; M07 makes it an alert.

### Verify it

Five checks, and the second is the one that tells you whether you have a real system.

1. **Time the request.** Place an order and read the server's response time. If the user's action takes more than about 500 milliseconds, something slow is still inside the handler. Find it.
2. **Make a job fail on purpose.** Break the email provider's credentials, or make the job throw. Then look: did it retry? How many times? Where is it now? If you cannot answer those three questions by looking at something, you have a queue with no visibility and you will not notice real failures either.
3. **Run a job twice, deliberately.** Take a completed job and re-run it. Count whatever it produces — emails, credits, rows. Two of anything that should be one is a missing idempotency key.
4. **Kill the worker mid-job,** then let it restart. Confirm the job either completes or retries, and that nothing was done twice in a way that matters.
5. **Skip a cron run.** Disable the schedule for an hour, then re-enable it. A well-designed job catches up on everything it missed. A job that processes "the last hour" has silently lost that hour forever.

Red flags: an email or third-party call inside a request handler; an enqueue after the commit rather than inside the transaction; a try-catch around job execution that logs and continues; no dead-letter queue, so failures vanish; no idempotency key on a job that touches money; a cron job that assumes it ran on time and exactly once; retries with no maximum; and a background system with no number anywhere showing how many jobs are waiting.

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
