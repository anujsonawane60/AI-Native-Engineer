---
id: m13-02-event-driven-at-scale
module: m13-production-engineering
slug: event-driven-at-scale
title: "Event-Driven Architecture at Scale: Kafka, Streams, Idempotent Consumers"
order: 2
duration_min: 15
is_free: false
mission: false
volatility: low
last_reviewed: 2026-11-16
---

## Key learnings
- Tell a queue and a log apart, and say which problem each one solves
- Choose a partition key and predict the ordering you do and do not get
- Make a consumer idempotent with a dedupe key that survives a restart
- Use consumer lag as the one number that tells you a pipeline is healthy
- Catch a consumer that commits its offset before it finishes the work

## Lesson

### The pain

M04 chapter 16 taught you queues properly: enqueue, worker, acknowledge, retry, dead-letter queue, and idempotent jobs. It said M13 is where event-driven architecture becomes a real conversation. That scale is now.

Here is what a queue does not solve. Your app publishes "order paid" and three things need to know: the kitchen display, the loyalty counter, the nightly revenue report. One worker takes the job and destroys it, so you add three queues and three enqueues, and the payment handler now knows all three consumers. Worse: a deploy restarts the loyalty worker after it credited forty students but before it acknowledged, so the broker redelivers and forty students silently hold double points.

### The idea

**A queue delivers each message to one worker and then forgets it. A log keeps every message in order and lets any number of independent readers move through it at their own pace.**

The publisher stops knowing who consumes, and a new consumer can be added without touching it, starting from the beginning rather than only from now. The cost is three problems you now own: ordering holds only within a partition, delivery is at-least-once so every consumer must be idempotent, and lag can grow without anything looking broken.

### The analogy

M04 gave you the spike where order slips are impaled and a finished slip is thrown away. A log is the canteen's bound register instead: every order goes on the next line and no line is erased. Each reader keeps a paper strip at the last line they handled — the **offset**. The register is split into books by hostel block so three clerks work at once: within a book lines are ordered, across books not at all. That is **partitioning**, and which book an order goes in is the **partition key**.

**Where the analogy breaks:** a register is kept forever. A real log has **retention** — the oldest lines are deleted after some days, so a consumer that falls behind gets a gap it can never fill, and no error fires when the data is lost.

### Picture it

```
PRODUCER            TOPIC  "order.paid"            CONSUMER GROUPS
                 ┌──────────────────────────┐
 order paid ────▶│ P0: 11 12 13 14 15 16 17 │◀── kitchen   @16
  key = shop_id  │ P1: 44 45 46 47 48       │◀── loyalty   @45
                 │ P2: 90 91 92             │◀── reporting @90
                 └──────────────────────────┘      (lag 2, 3, 2)
                        │
                 retention: 7 days ── older lines deleted

  ordered within P0 ✓      ordered across P0,P1,P2 ✗
```

One topic is split into partitions; each consumer group keeps its own offset per partition, so three consumers read the same events independently and the distance between the newest message and a group's offset is its lag.

### Where it lives

This extends M04 chapter 16 and leans on three chapters.

- The dedupe store is M05 chapter 2's shared cache, with a TTL from M05 chapter 3.
- An event's schema is chapter 1's contract problem: a published event is an API.
- Consumer lag is an M07 metric, and belongs beside p95.

**Honest scoping: your startup almost certainly does not need Kafka.** A Postgres table with a status column is a respectable queue. Take the vocabulary and the failure modes: two consumers of one event inherit all of them whatever the technology. Run a single-broker container locally, or a free serverless tier — never a managed multi-broker cluster.

### Words to own

| Term | Plain meaning | Example |
|---|---|---|
| Event | A statement that something happened | `order.paid` |
| Topic | A named log | `order.paid` |
| Partition | One ordered slice of a topic | `P0`, `P1`, `P2` |
| Partition key | What decides a message's partition | `shop_id` |
| Offset | A reader's position in a partition | `@45` |
| Consumer group | A set of consumers sharing one offset set | `loyalty` |
| Consumer lag | Newest offset minus the group's offset | `3` |
| Retention | How long messages are kept before deletion | 7 days |
| At-least-once | Messages may arrive twice, never zero times | The normal case |
| Idempotent consumer | Processing twice has the same effect as once | Dedupe key |
| Dedupe store | Record of event ids already processed | With a TTL |
| Transactional outbox | Write event to your DB, publish from there | Avoids lost events |
| Dead-letter topic | Where unprocessable messages are parked | A human looks |
| Replay | Reset an offset and re-read old events | Rebuild a projection |
| Projection | A derived table built by consuming events | Loyalty balances |

### Tradeoffs and gotchas

**Commit the offset after the work, never before.** Commit first and a crash loses the message silently; commit after and it is redelivered, which is safe only if the consumer is idempotent. "Exactly-once" across a database and a broker does not exist.

**Idempotency needs durable storage.** Store the event id with a TTL longer than your longest retry window, written with the business change.

**Ordering is per partition, and the key has consequences.** Key by order id and one order's events stay in order; key by a city and one **hot partition** takes all the traffic — the cost of three, the throughput of one.

**The outbox exists because a broker cannot join your database transaction.** Write the row and the event in one transaction into an `outbox` table, and publish from there.

**Lag is the health metric and nothing errors while it grows.** Alert on it as a duration, not a count: lag approaching retention is data loss approaching.

**Replay is the superpower and the foot-gun.** Rebuilding a table from offset zero is wonderful; replaying a consumer that sends WhatsApp messages is nine thousand notifications about March.

### Verify it

1. **Find the offset commit and read what comes before it.** A commit before the work completes is silent message loss.
2. **Ask for the dedupe key by name and where it is stored.** A variable is not idempotency. Then deliver the same event twice and check the result is identical.
3. **Ask what happens when the publish succeeds but the database write fails, and the reverse.** Without an outbox, one case loses data.
4. **Ask which consumers are safe to replay,** and confirm lag is alerted as a duration, not a count.

Red flags: an offset committed before processing; no dead-letter topic, so one bad message blocks a partition forever; retention never mentioned; a partition key with a handful of values.

## Walkthrough
1. Write down one event your product already produces, in past tense, and list every thing that needs to know about it today. If the list has one item, write down the second one you expect within three months.
2. Run a single-broker log locally in a container on your laptop — free, no account. Create one topic with three partitions.
3. Publish ten events by hand with a partition key of your choosing, then read them back and record which partition each landed in. Confirm for yourself that order holds inside a partition and not across them. Then choose your real partition key deliberately, with one sentence on what ordering it guarantees and one on what hot partition it could create.
4. Brief AI for one consumer that updates a derived table, explicitly requiring the offset commit to happen after the work and the dedupe key to be stored durably. Read the code and find the commit line.
5. Deliver the same event twice on purpose. Confirm the derived table is identical after the second delivery. If it is not, your idempotency is theatre.
6. Now kill the consumer mid-batch, restart it, and confirm no message was lost and no effect was doubled. Record what you observed.
7. Add a second consumer group for a different purpose and start it from the beginning of the topic. Watch it catch up while the first group keeps its own position. This is the thing a queue cannot do.
8. Publish a deliberately malformed event. Confirm it goes to a dead-letter topic rather than blocking the partition. Check whether the following valid messages still processed.
9. Create lag on purpose: pause a consumer, publish a few hundred events, and watch lag grow. Resume and watch it drain. Record the lag in seconds, not messages.
10. Implement the outbox pattern for one write: event and row committed in the same transaction, published by a separate reader. Then simulate a crash between the write and the publish and confirm the event still goes out.
11. Finally, write down which of your consumers are safe to replay and which are not, and why. That list is what makes replay a tool instead of an accident.

## Talk to AI
```prompt
CONTEXT:      I am a student on an AI-native engineering program. I direct AI
              and verify its output; I do not write code by hand. My team of
              three runs [product in two sentences] with [number] real users
              in India. Stack: [your stack], [your database] on a free tier.
              We already have background work on a queue from earlier in the
              course: [describe what it does]. The event I want to move to a
              log is [describe it], and the things that need to know about it
              are: [list them all, including ones you expect within 3 months].
              We have no budget - I will run a single-broker log in a
              container on a laptop, or a free serverless tier with a small
              monthly message allowance. A managed multi-broker cluster is out
              of the question.

GOAL:         One correctly built event flow: a published event, two
              independent consumers, idempotent processing, a dead-letter
              path, and lag I can see - plus an honest answer about whether I
              should be doing this at all yet.

REQUIREMENTS: 1. First, argue honestly whether my situation justifies a log
                 rather than the queue I already have. Use my number of
                 consumers, not a hypothetical future
              2. Define the event payload as a contract: fields, types, and
                 the rule for changing it later without breaking a consumer I
                 forgot about
              3. Recommend a partition key for my case, and state exactly
                 what ordering it guarantees and what ordering it does NOT.
                 Name the hot-partition risk of the key you chose
              4. A consumer where the offset commit happens AFTER the work
                 completes. Point at the exact line and tell me what a crash
                 one line earlier and one line later each produce
              5. Idempotency with a durable dedupe key: where it is stored,
                 what its TTL is and why that number, and how the dedupe
                 write and the business change avoid a crash in between
              6. The transactional outbox pattern for my write path, and what
                 specifically it prevents that a plain publish does not
              7. A dead-letter topic, plus what happens to the messages
                 behind a bad one while it is being retried
              8. Consumer lag as a monitored metric in SECONDS with an alert
                 threshold, and what my retention is and why lag approaching
                 retention is data loss rather than slowness
              9. Tell me which of my consumers would be safe to replay from
                 offset zero and which would spam my users

CONSTRAINTS:  Free only - local container or a free serverless allowance; say
              clearly where free is not possible and give the cheapest honest
              path in rupees. Plain English for a second-language reader.
              Label every command READ-ONLY or CHANGES-THINGS. Do not promise
              exactly-once delivery. Do not change my database schema beyond
              adding an outbox table and a dedupe table.

NON-GOALS:    Do not provision a managed cluster. Do not add a stream
              processing framework, a schema registry product, or a service
              mesh. Do not migrate my existing queue work. Do not design for
              a million events a day.

ACCEPTANCE:   - I can state in one sentence why a log, not a queue, for this
              - Delivering the same event twice leaves my derived table
                identical, and I have tested it
              - Killing a consumer mid-batch loses nothing and doubles nothing
              - A second consumer group reads from the beginning while the
                first keeps its own position
              - A malformed event lands in a dead-letter topic and the
                following valid messages still process
              - I can read my consumer lag in seconds and I have an alert
              - A crash between my database write and my publish still
                results in the event being published
              - I have a written list of which consumers are replay-safe

ASK:          Start with the honest case for NOT doing this, given my actual
              consumer count. Then give me the event contract and the
              partition key with its ordering guarantee, and wait for me to
              confirm before writing any consumer.
```

## Stuck? Ask AI
```prompt
Explain the difference between a queue and a log using an analogy that is not about registers, notebooks, conveyor belts or post offices. Then give me six situations - a new consumer needs last month's data, one consumer is slow, a message is unprocessable, two consumers need the same event, a consumer must reprocess from the start, a message must be handled exactly once by exactly one worker - and ask me which of the two fits each, one at a time, without telling me the answers first.
```

```prompt
Here is a consumer: [paste the code]. Do not rewrite it. Find the line where the offset is committed and tell me whether it is before or after the work completes. Then describe, step by step, what happens if the process is killed one line before that commit and one line after it. Tell me which of the two produces silent data loss and which produces a double effect, and what the user would see in each case.
```

```prompt
Explain consumer lag and retention together. Why does a pipeline with growing lag raise no errors? Give me the arithmetic for a consumer processing 50 messages a second while 80 arrive per second, with 7-day retention - tell me when it falls behind irrecoverably and what exactly is lost at that moment. Then tell me whether I should alert on lag in messages or in seconds, and why.
```

## Quiz

### Q1 · L3
Three different features need to know about every paid order. What does a log give you that a queue does not?
- (a) Guaranteed delivery, which queues lack
- (b) Three independent readers of the same events, each with its own position, added without changing the publisher ✅
- (c) Ordering across the whole topic
- (d) Exactly-once processing for all three
> A queue delivers each message to one worker and forgets it. A log keeps messages and lets consumer groups move through independently — which is why the publisher stops needing to know who consumes.

### Q2 · L4
A consumer commits its offset before doing the work. What is the failure mode?
- (a) Messages are processed twice after a crash
- (b) Messages are lost silently after a crash, because the broker believes they were handled ✅
- (c) The consumer group rebalances continuously
- (d) Lag is reported incorrectly but no data is affected
> Committing early converts at-least-once into at-most-once. Committing after the work gives you redelivery instead, which is survivable because you made the consumer idempotent — so commit last, always.

### Q3 · L4
A worker credits loyalty points and keeps a set of processed event ids in memory to avoid duplicates. Why is this not idempotency?
- (a) Sets are too slow at high message rates
- (b) The set is lost on restart, which is exactly when redelivery happens ✅
- (c) In-memory sets cannot hold enough ids
- (d) Idempotency must be enforced by the broker, not the consumer
> Redelivery is triggered by the restart that wipes the set. The dedupe record must be durable, with a TTL longer than the maximum retry window, and written together with the business change.

### Q4 · L4
You partition a topic by `city` and 85% of your orders are from one city. What have you built?
- (a) Correct ordering across all orders
- (b) A hot partition: the cost of several partitions and the throughput of one ✅
- (c) A dead-letter loop
- (d) A consumer group that cannot rebalance
> Low-cardinality or skewed keys concentrate traffic on one partition. Partition key choice is a design decision about both ordering and distribution, and a key with few possible values fails the second test.

### Q5 · L5
Your loyalty consumer is forty minutes behind, nothing is erroring, and retention is seven days. Your reporting consumer is three days behind. Which is the emergency and why?
- (a) The loyalty consumer, because user-visible data is stale
- (b) Neither; nothing is erroring
- (c) The reporting consumer, because lag approaching retention is permanent data loss rather than delay ✅
- (d) Both equally, since lag is lag
> Lag inside retention is slowness you can recover from. Lag approaching retention is data that will be deleted before it is ever read, with no error at the moment it happens — which is why lag is alerted as a duration measured against retention, not as a message count.

## Go deeper
- [Kafka: topics, partitions, offsets and consumer groups, from the official docs](https://kafka.apache.org/documentation/#intro_concepts_and_terms)
- [Microsoft: the transactional outbox pattern](https://learn.microsoft.com/en-us/azure/architecture/best-practices/transactional-outbox-cosmos)
