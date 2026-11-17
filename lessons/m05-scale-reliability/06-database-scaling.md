---
id: m05-06-db-scaling
module: m05-scale-reliability
slug: database-scaling
title: "Database Scaling: Replicas, Pooling, Sharding and Partitioning"
order: 6
duration_min: 15
is_free: false
mission: false
volatility: low
last_reviewed: 2026-11-16
---

## Key learnings
- Apply the four database fixes in the right order, cheapest first
- Explain why three app instances can exhaust a free-tier connection limit
- Describe replication lag and the read-your-own-writes problem it causes
- Tell partitioning and sharding apart, and say why sharding is a last resort
- Catch an AI that proposes sharding for a table with ten thousand rows

## Lesson

### The pain

Four days after scaling to three instances, the app returns outright errors at 1:10 pm: `too many connections`. CPU is at 4%, memory is fine, disk is almost empty. The database is refusing work while doing nothing, until you count: three instances at twenty connections each is sixty, and your free plan allows thirty. You ran out of doors into the database, not out of database.

So you add a read replica. Now a student taps "Place order", sees the confirmation, lands on their orders page, and the order is not there. They order again, and the refund is your afternoon.

### The idea

**Your app servers are disposable and your database is not, so the database has exactly four kinds of fix — and they go in a strict order of cost.**

First, do less work: an index (M04) and a cache (Chapters 2 and 3) remove queries entirely, and a query that never runs beats one you optimised. Second, share the doors with a pooler. Third, add copies for reading. Fourth, and only then, split the data. Each step is roughly ten times more disruptive than the last, and skipping to the fourth is how teams build systems they cannot change.

### The analogy

Four counters, one kitchen: no number of counters makes the kitchen cook faster. **Pooling** is the serving hatch — without one, every cashier walks in whenever they like, and at 1:10 pm twelve people are in a space built for three. **Read replicas** are photocopies of the stock board at each counter: anyone may read a copy, only the kitchen writes the original, and the copies are a few seconds behind. **Partitioning** separates today's tray shelf from last month's records. **Sharding** builds a second kitchen and splits students by roll number: capacity doubles, and "how much did the whole campus eat today" becomes hard.

**Where the analogy breaks:** a replica appears in minutes rather than months, but its data travels over the network, and that copying time is replication lag — the direct cause of the vanishing order.

### Picture it

```
     writes ──────▶ ┌──────────────┐
                    │   PRIMARY    │  the only writable copy
                    └──────┬───────┘
                           │ replication: 10 ms to 2 s behind
                 ┌─────────┴─────────┐
                 ▼                   ▼
          ┌─────────────┐     ┌─────────────┐
          │  replica 1  │     │  replica 2  │   reads only
          └─────────────┘     └─────────────┘
                 ▲                   ▲
                 └──── SELECT ───────┘
```

All writes go to the single primary, which streams its changes to read-only replicas that serve `SELECT` traffic slightly behind the truth.

| Fix | What it does | When you need it | Cost of getting it wrong |
|---|---|---|---|
| Index and cache | Removes queries | Immediately | Stale data (Chapter 3) |
| Connection pooling | Shares a few real connections among many clients | The day you run two instances | Queries queue at the pooler |
| Read replica | Serves `SELECT` from a copy | Reads are 90% of traffic and the primary is busy | Stale reads, vanishing orders |
| Partition or shard | Splits the data | Tens of millions of rows, measured | Cross-shard queries become your code |

**Pooling is not optional and not advanced.** Serverless makes it urgent, because each concurrent invocation may open its own connection. Most managed databases offer a pooler behind a different connection string, and that one change fixes the error above. **Almost nobody here needs sharding:** one well-indexed database handles tens of millions of rows and thousands of requests a second.

### Where it lives

The database is the last of M00's four boxes and the one thing all your stateless servers share. M04 supplies step one, indexes and query plans; Chapter 8 covers the timeouts that stop a pooler queue becoming an outage; M13 goes deep on failover.

**Partitioning** splits one table inside one database, usually by date, and your queries do not change — the database skips pieces that cannot match, but only if your queries filter on the partition key. **Sharding** splits data across separate databases by a shard key, and a cross-shard join, a global aggregate and any later change of key all become your problem. Pick badly and you get a **hot shard**: shard by city with 60% of users in Mumbai and you have one overloaded database and three idle ones.

### Words to own

| Term | Plain meaning | Example |
|---|---|---|
| Connection | An open channel from app to database | One of your thirty |
| Connection pool | A set of reused connections | 20 per instance |
| Pooler | A middleman sharing real connections | A pooled connection string |
| Primary | The one writable copy | Takes all writes |
| Replica | A read-only copy kept in step | Serves `SELECT` |
| Replication lag | How far behind a replica is | 10 ms to 2 s |
| Read-your-writes | Seeing your own change immediately | Your order after placing it |
| Partitioning | One table split inside one database | Orders by month |
| Sharding | Data split across separate databases | Users A–M and N–Z |
| Shard key | What decides which shard holds a row | City, or user id |
| Hot shard | One shard holding most of the traffic | 60% of users in Mumbai |
| Failover | Promoting a replica when the primary dies | Usually 10–60 seconds |

### Tradeoffs and gotchas

**Replication lag breaks the one read users care about most.** Nobody forgives their own order vanishing, so the rule is: reads immediately after a write by the same user go to the primary, everything else may go to a replica.

**A replica does not help a write problem.** If the primary is busy because students are placing orders, replicas change nothing and add a lagging copy to reason about.

**`max_connections` is not a dial to turn up.** Each connection costs real memory, so raising it trades a clear error for a slow, unpredictable database. A pooler is the fix, though it moves the queue rather than deleting it.

**Partitioning the wrong column is work for nothing.** It helps only when your real queries filter on the key.

**Sharding is close to irreversible.** Treat a proposal to shard like a proposal to change language: possible, occasionally right, never casual.

### Verify it

- **Make it show the arithmetic:** "with N instances each holding a pool of M connections, how many will my database see, and what is my limit?" That one calculation prevents the commonest free-tier outage.
- **Red flag: sharding or a NoSQL migration for a small table.** Ask for the row count and measured query time; under a few million rows, the answer is an index.
- **Red flag: a read replica with no mention of lag.** Ask which reads would be wrong at two seconds old.
- **Red flag: "raise `max_connections`" or "upgrade the plan" first.** Neither is right before pooling and indexing.
- **Red flag: a replica added when the bottleneck is writes.** Ask what share of queries are reads, with a number.
- **Red flag: a cache or replica in front of the part that was never slow.**
- Verify by hand: watch the connection count under load against the limit, confirm your slowest query's plan uses an index, and with a replica, write a row and read it straight back repeatedly, recording how often it is missing.

## Walkthrough
1. Open your database dashboard and find the connection limit for your plan. Write the number down. On free tiers it is often between 20 and 60.
2. Find the current connection count. Then multiply your instance count by your app's pool size and compare. If the product exceeds the limit, you have a scheduled outage waiting for your next traffic spike.
3. Check whether your provider offers a pooled connection string — usually a different host or port. If it does and you are not using it, that is your first change.
4. Switch to the pooled string in your environment variables only, nothing else, and deploy. Re-check the connection count under load.
5. Now find your read-write split. Count, over an hour of real use, roughly how many queries are `SELECT` versus writes. Write both numbers down.
6. If reads are not the overwhelming majority, stop thinking about replicas. Go back to M04 and read the query plan of your slowest query instead.
7. Take your three slowest queries and get their plans. Note any that scan a whole table. Each one is an index you are missing, and an index is cheaper than every other fix in this chapter.
8. List every read in your app that happens immediately after the same user wrote something: order placed then order list, profile saved then profile shown, payment made then status checked.
9. That list is your read-your-writes set. Write next to each one that it must go to the primary. Keep this list; it is the thing that makes a replica safe to add later.
10. Finally, write one line in your decision log recording your connection arithmetic, your read-write split, and the fix you chose. If the honest answer was "an index, not a replica", that is a better entry than any architecture diagram.

## Talk to AI
```prompt
CONTEXT:      I am a student directing AI on my team's app. Database:
              [your database and plan], free tier, connection limit [X].
              Hosting: [your host], currently running [N] app instances,
              each with a connection pool size of [M]. Are we using a
              pooled connection string: [yes/no].

              Measured facts about my data and traffic:
              - largest table and its row count: [table, X rows]
              - roughly what share of queries are reads: [X%]
              - my three slowest queries and their measured times:
                1. [query description] - [X] ms
                2. [query description] - [X] ms
                3. [query description] - [X] ms
              - do those queries use an index: [yes/no/unknown]
              - errors I have actually seen: [e.g. too many connections]

GOAL:         Decide which database fix I actually need right now, in the
              correct order, with the arithmetic shown.

REQUIREMENTS: 1. Calculate the total connections my instances will open
                 and compare it against my limit, with the numbers visible
              2. Tell me whether I need pooling, and exactly what changes
                 if so
              3. Given my row counts and read share, tell me plainly
                 whether I need a read replica - including saying no
              4. Tell me explicitly whether I need partitioning or
                 sharding, and if not, at roughly what row count or
                 request rate that would change
              5. List which of my reads must never be served by a replica,
                 because the same user just wrote the data
              6. Rank everything by milliseconds saved per hour of work

CONSTRAINTS:  Free tier only, no plan upgrades this term. I cannot change
              my database engine. Any change must be shippable on its own,
              not as part of a rewrite.

NON-GOALS:    Do not propose sharding unless my row counts justify it, and
              if you do, state the row count threshold you are using. Do
              not propose raising max_connections. Do not propose a new
              database. Do not refactor my schema.

ACCEPTANCE:   - The connection arithmetic appears as numbers
              - Every recommendation cites one of my measurements
              - At least one of pooling, replicas, partitioning and
                sharding is explicitly ruled out, with a reason
              - My read-your-own-writes list is complete
              - The top-ranked fix is one I could ship today

ASK:          First show me the arithmetic and the ranked list. Tell me
              which of my measurements you trust least and what I should
              measure to settle it. No code until I pick one fix.
```

## Stuck? Ask AI
```prompt
Explain what a database connection actually is and why there is a limit on how many can be open at once, as if I have never programmed. Then walk me through the arithmetic for three app servers with a pool of twenty each against a limit of thirty, and explain what a connection pooler changes about that picture.
```

```prompt
Explain replication lag with a timeline: a student places an order at 12:00:00.000 and reads their order list at 12:00:00.200, with a replica that is 400 ms behind. Show me what each moment looks like from the database's point of view, then give me two ways to fix it and the cost of each.
```

```prompt
Explain the difference between partitioning and sharding with one concrete example each, then tell me the three things that become permanently harder after sharding. Finish by telling me honestly, with a row count, when a small team should even start thinking about it.
```

## Quiz

### Q1 · L3
Three app instances each hold a pool of twenty connections. Your free plan allows thirty. What happens at the next traffic spike, and what is the fix?
- (a) Nothing; pools are shared automatically between instances
- (b) `too many connections` errors; use a connection pooler ✅
- (c) The database slows down but keeps working; add an index
- (d) The load balancer reduces the pool sizes for you
> Sixty requested against thirty allowed is a refusal, not a slowdown — and the database may be at 4% CPU while it happens. A pooler lets many clients share a few real connections.

### Q2 · L4
A student places an order, is redirected to their orders page, and the order is missing. Reads are served by a replica. What happened?
- (a) The write failed silently
- (b) The replica had not yet received the new row — replication lag ✅
- (c) The cache TTL had not expired
- (d) The order was written to the wrong partition
> This is the read-your-own-writes problem. The standard fix is to route reads that immediately follow the same user's write to the primary, and everything else to replicas.

### Q3 · L4
Your orders table has 10,000 rows and one query takes 900 ms. An AI proposes sharding by city. What is the better first step?
- (a) Shard as proposed, since it will scale further
- (b) Read the query plan and add the missing index ✅
- (c) Add two read replicas
- (d) Raise `max_connections`
> 10,000 rows is tiny; 900 ms means a full table scan. Sharding is near-irreversible and makes joins and aggregates your problem. Indexes come first, always.

### Q4 · L3
Your primary database is saturated because students are placing orders at lunch. Will adding read replicas help?
- (a) Yes, replicas share all database work
- (b) No — replicas serve reads, and the load here is writes ✅
- (c) Yes, because writes are forwarded to replicas too
- (d) Only if the replicas are in the same region
> All writes go to the single primary. Replicas help a read-heavy workload and add a lagging copy to reason about; they do nothing for write pressure.

### Q5 · L5
You shard by city and 60% of your users are in Mumbai. What have you built?
- (a) A balanced cluster, since cities are natural boundaries
- (b) A hot shard: one overloaded database and several idle ones ✅
- (c) A partitioned table that skips irrelevant data
- (d) A read replica topology
> A shard key must spread load, not just divide data. Skewed keys concentrate traffic on one shard, and changing the key later means moving data while serving live traffic.

## Go deeper
- [PostgreSQL: connection pooling explained](https://www.pgbouncer.org/features.html)
- [Use the index, Luke — a guide to database indexing](https://use-the-index-luke.com/)
