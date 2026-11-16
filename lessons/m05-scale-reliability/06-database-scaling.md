---
id: m05-06-db-scaling
module: m05-scale-reliability
slug: database-scaling
title: "Database Scaling: Replicas, Pooling, Sharding and Partitioning"
order: 6
duration_min: 25
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

You scaled to three instances. Capacity is up, students are happy for four days, and then at 1:10 pm the app starts returning errors — not slow pages, outright errors. The message is `too many connections`.

You look at the dashboard. CPU is at 4%. Memory is fine. Disk is almost empty. The database is doing nothing at all and refusing work, which makes no sense until you count: three instances opening twenty connections each is sixty, and your free plan allows thirty. You did not run out of database. You ran out of doors into it.

So you add a read replica, which is what everyone suggests. Now a student taps "Place order", sees the confirmation, lands on their orders page — and the order is not there. They place it again. Two orders, and the refund is your afternoon.

Both failures come from one gap: the app servers multiplied, and nobody thought about what that does to the one thing they all share.

### The idea

**Your app servers are disposable and your database is not, so the database has exactly four kinds of fix — and they go in a strict order of cost.**

First, do less work: an index (M04) and a cache (Chapters 2 and 3) remove queries entirely, and a query that never runs beats one you optimised. Second, share the doors: a pooler lets fifty app processes take turns through a handful of real connections. Third, add copies for reading: a replica takes `SELECT` traffic off the primary. Fourth, only when the first three are exhausted, split the data across separate stores.

The order is not a style preference. Each step is roughly ten times more disruptive than the last, and skipping to step four is how small teams build systems they cannot change.

### The analogy

Four counters, one kitchen. The counters are your app instances and the kitchen is your database, and no number of counters makes the kitchen cook faster.

**Pooling** is the serving hatch. Without one, every cashier walks into the kitchen whenever they like, and at 1:10 pm there are twelve people in a space built for three and nothing moves. With a hatch, cashiers queue briefly at a fixed opening and the kitchen stays workable. The hatch adds a tiny wait to every order and removes the chaos — which is exactly the trade a connection pooler makes.

**Read replicas** are photocopies of the stock board pinned up at each counter. Anyone may read a photocopy; only the kitchen writes the original. The copies are refreshed constantly but they are always a few seconds behind, so a dish that just ran out may still show as available on the copy.

**Partitioning** is separating today's tray shelf from the cupboard of last month's records. Same kitchen, same staff, but a search through today's food no longer wades through March.

**Sharding** is building a second kitchen in the hostel block and deciding by roll number which kitchen serves which student. Capacity genuinely doubles. And now nobody can easily answer "how much food did the whole campus eat today", because the answer lives in two buildings and somebody has to visit both and add up.

**Where the analogy breaks:** kitchens take months to build, while a replica appears in minutes. But the data has to be copied over the network, and that copying time is the replication lag — a real, measurable delay with no physical equivalent in the canteen, and the direct cause of the vanishing order above.

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

### The four fixes, in order

| Fix | What it does | When you need it | Cost of getting it wrong |
|---|---|---|---|
| Index and cache | Removes queries | Immediately | Stale data (Chapter 3) |
| Connection pooling | Shares a few real connections among many clients | The day you run two instances | Queries queue at the pooler |
| Read replica | Serves `SELECT` from a copy | Reads are 90% of traffic and the primary is busy | Stale reads, vanishing orders |
| Partition or shard | Splits the data | Tens of millions of rows, measured | Cross-shard queries become your code |

Two notes. **Pooling is not optional and not advanced** — serverless functions make it urgent, because each concurrent invocation may open its own connection and a hundred will flatten a free-tier limit instantly. Most managed databases offer a pooler you switch on with a different connection string, and that one change fixes the error above.

And **almost nobody here will need sharding.** A single well-indexed database handles tens of millions of rows and thousands of requests a second. If an AI proposes sharding your 10,000-row orders table, it has pattern-matched on the word "scale" and is about to cost you a month.

### Partitioning versus sharding

They sound alike and differ in one way that matters.

**Partitioning** splits one table into pieces inside one database, usually by date: November's orders in one piece, October's in another. Your queries do not change. The database skips pieces that cannot contain matching rows, so a query for today stops scanning last year. It helps only if your queries actually filter on the partition key — partition by month, then search by student id, and you have gained nothing.

**Sharding** splits the data across separate databases by a shard key such as a city or a user id range. Capacity genuinely multiplies, and three things get much harder: a join across shards becomes your job rather than the database's, an aggregate like "total orders today" means querying every shard and adding up, and changing the key later means moving data while serving traffic. Pick a bad key and you get a **hot shard** — shard by city with 60% of users in Mumbai and you have one overloaded database and three idle ones.

### Where it lives

The database is the last box in M00's four, and this chapter is about the one thing all your stateless servers share. It builds directly on M04: indexes and query plans are step one here, and transactions and backups are what make a primary trustworthy. Chapter 7 explains what replication lag means formally. Chapter 8 covers the timeouts that stop a pooler queue turning into an outage. M13 goes deep on failover and multi-region.

The realistic sequence this term is: read your query plan, add the missing index, cache the hottest read, switch to the pooled connection string, and stop. Everything past that needs a measurement, not an ambition.

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

**Replication lag breaks the one read users care about most.** Students forgive a menu that is two seconds stale. Nobody forgives their own order vanishing. The standard fix is a rule: reads immediately after a write by the same user go to the primary, everything else may go to a replica. Decide this per feature and write it down, because the default of "all reads go to the replica" produces exactly the double-order incident above.

**A replica does not help a write problem.** Replicas serve reads. If your primary is busy because students are placing orders, adding replicas changes nothing and adds a lagging copy to worry about. Check your read-write split before you add one.

**`max_connections` is not a dial to turn up.** Each connection costs the database real memory and scheduling, so raising the limit trades a clear error for a slow, unpredictable database. This is the same shape of mistake as "add more memory": making room for the symptom instead of removing it. A pooler is the fix.

**Pooling moves the queue rather than deleting it.** If your queries are slow, requests now wait at the pooler instead of failing at the database. Waiting beats erroring — but a pooler in front of a missing index just hides the index for another week.

**Partitioning the wrong column is work for nothing.** It helps only when your queries filter on the partition key. Check your real queries, not the ones you imagine having.

**Sharding is close to irreversible.** Cross-shard joins, global aggregates and the migration itself all become your problem permanently. Treat a proposal to shard the way you would treat a proposal to change language: possible, occasionally right, never casual.

### Verify it

**Make it show you the arithmetic.** Ask: "with N instances each holding a pool of M connections, how many connections will my database see, and what is my plan's limit?" If those numbers are not in the answer, nothing else in it is trustworthy. This one calculation prevents the most common free-tier database outage there is.

**Red flag: sharding, or a NoSQL migration, proposed for a small table.** Ask for the row count and the measured query time that justifies it. "Scale" is a word, not evidence. For anything under a few million rows, the honest answer is almost always an index.

**Red flag: a read replica with no mention of lag.** Ask directly: "which of my reads would be wrong if the data were two seconds old, and where do those reads go?" If the AI cannot name your post-write reads as the risk, it has not thought about your app.

**Red flag: "raise `max_connections`" or "upgrade the database plan" as the first suggestion.** Same family as "add more memory". Both may eventually be right; neither is right before you have pooled and indexed.

**Red flag: a replica added when the bottleneck is writes,** or an index that duplicates one already present. Ask what fraction of your queries are reads, with a number.

**Red flag — the familiar one: a cache or a replica placed in front of the part that was never slow,** while the unindexed 900 ms query stays as it is. Ask which measured step each proposal removes.

Then verify by hand. Watch your connection count under load and compare it to the limit. Run your slowest query's plan and confirm it uses an index (M04 taught you to read one). If you have a replica, write a row and immediately read it back through the replica path, repeatedly, and record how often it is missing — that is your lag risk, measured rather than assumed. And log the decision: "pooled connection string, 10 per instance, because three instances against a thirty-connection plan" is the kind of M01 entry that settles an argument in March.

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
