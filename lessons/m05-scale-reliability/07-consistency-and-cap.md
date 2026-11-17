---
id: m05-07-consistency-cap
module: m05-scale-reliability
slug: consistency-and-cap
title: "Consistency, the CAP Theorem and What You Trade Away"
order: 7
duration_min: 15
is_free: false
mission: false
volatility: low
last_reviewed: 2026-11-16
---

## Key learnings
- Decide strong or eventual consistency per feature, never per app
- State what CAP actually forces you to choose, and only when
- Name the compensating action for every feature you let be eventual
- Explain why adding a cache makes a strongly consistent read eventual
- Catch an AI that calls a system consistent while a cache sits in front of it

## Lesson

### The pain

There is one paneer plate left. Two students tap "Order" in the same second, served by two of your three instances. Both read `quantity = 1`, both decide the order is fine, both write. Two students pay ₹60 for one plate. Your database never lied: both reads were correct when they happened. And had you demanded the strictest correctness here, your app would also have refused everybody during the two seconds your database was failing over last Tuesday.

### The idea

**You cannot have data that is simultaneously correct everywhere, instantly, and still answerable when the network between your machines breaks. So you choose — and you choose per feature.**

**Strong consistency** means every read sees the most recent write: the truth, or an error rather than a guess. **Eventual consistency** means copies converge after a while, usually milliseconds, so a read may be slightly old. The **CAP theorem** says that during a *partition*, when parts of your system cannot reach each other, you keep consistency or availability but not both. Partitions are rare; the daily choice is the softer cousin — the nearby replica is faster and sometimes stale, the distant primary slower and right.

### The analogy

Two serving points, one stock of paneer. The strongly consistent arrangement is a single token window: the last paneer token is issued exactly once — and when that window's power fails, nobody eats, though there is food on the counter. The eventually consistent arrangement is each counter keeping its own slips and reconciling at closing: everybody eats immediately, and twice a week the slips show the last paneer was sold at both counters. Real canteens run the second, because they have what distributed systems only have if you build it: a human who says "sorry, that's finished, take chole instead, no charge". That apology is a **compensating action**, and it is the price of availability.

**Where the analogy breaks:** the canteen improvises and your code cannot, so the apology must be written in advance. An eventual feature with no compensating action is a bug you have not met yet.

### Picture it

```
      the network between your two halves breaks
  ┌───────────────┐       ╳╳╳       ┌───────────────┐
  │   counter A   │ ── no link ───  │   counter B   │
  │  1 paneer     │                 │  1 paneer     │
  └───────┬───────┘                 └───────┬───────┘
          │                                 │
          ▼                                 ▼
   choose consistency:              choose availability:
   refuse to serve                  serve anyway
   (right, but closed)              (open, but may oversell)
```

When the two halves cannot talk, each must either refuse to answer until the link returns, which keeps the stock count correct, or answer from what it knows, which keeps the app open and risks selling the same plate twice.

This table is the chapter in practice. Fill in your own version.

| Feature | What it needs | What you accept |
|---|---|---|
| Like or view count | Eventual | Off by a few for a few seconds |
| Menu availability | Eventual, bounded at 60 s | An occasional wasted tap |
| Profile photo | Eventual | Old picture briefly |
| Your own order list | Read-your-writes | Read from the primary, slower |
| Last plate of paneer | Strong | Refuse to sell under partition |
| UPI payment status | Strong, plus idempotency | Slower confirmation screen |
| Wallet balance | Strong | No offline writes at all |

Anything **counted approximately** can be eventual, because nobody audits a like count. Anything where **two people can take the same thing**, or where **money moves**, needs the strict answer — a single-writer check inside a transaction (M04), or a reservation that expires, not a cleverer cache.

### Where it lives

This chapter names, formally, the bills the previous five ran up. Your 60-second menu TTL was a consistency decision: eventual, bounded at one minute. So was your read replica, and so is your browser cache. Every second copy opts you into eventual consistency, whether or not anybody said the words.

- M04 — transactions and idempotency keys implement the strong rows; queues hold compensating actions
- Chapter 8 — what to do when the strongly consistent thing is unreachable
- M07 — measuring how often you are actually wrong
- M13 — quorums, consensus and multi-region

### Words to own

| Term | Plain meaning | Example |
|---|---|---|
| Consistency | Whether all copies agree right now | Replica matches primary |
| Strong consistency | Every read sees the latest write | Wallet balance |
| Eventual consistency | Copies agree after a short delay | Like count |
| Stale read | A correct answer from slightly too long ago | 400 ms old |
| Partition | Parts of the system cannot reach each other | A network split |
| Availability | The system answers rather than erroring | Serving during a split |
| CAP theorem | Under a partition, pick consistency or availability | The diagram above |
| Read-your-writes | A user always sees their own change | Order list after ordering |
| Last-write-wins | Conflicts resolved by timestamp | Silently drops the other write |
| Idempotency | Doing it twice has the effect of once | One charge per payment key |
| Compensating action | The fix when you were wrong | Refund and apologise |
| Quorum | A majority of copies must agree | 2 of 3 before confirming |

### Tradeoffs and gotchas

**CAP is the most misquoted idea in this module.** It applies during a partition and says nothing about normal operation, where the honest trade is that freshness costs latency.

**Last-write-wins loses data silently, and it is the default.** Two instances increment a counter from 10 to 11 and the result is 11, not 12. Counters need an operation the store merges atomically.

**A cache in front of a strongly consistent database makes the read eventual.** Any claim of strong consistency must account for every layer in Chapter 2's diagram, including the browser.

**Put a number on everything eventual, and accept the cost of everything strong.** "Stale by at most 60 seconds, enforced by a TTL" is a design; "eventually consistent" alone is a hope. A strong feature should stop during a failover rather than answer wrongly.

**Every eventual feature needs its apology written first.** Overselling the last paneer is acceptable only if the refund is automatic; without that you did not choose availability, you scheduled an incident.

### Verify it

- **Make it produce the table, not an opinion:** per feature, strong or eventual, what a user sees when it is wrong, and the compensating action.
- **Red flag: a system called consistent while a cache sits in front of it.** Walk browser, CDN, shared cache, replica, primary, asking how old each can be.
- **Red flag: last-write-wins on a counter or concurrent edits.** Ask what happens to the other write, and whether anybody finds out.
- **Red flag: no compensating action for anything eventual touching stock, seats or money.** At 1:10 pm simultaneous taps are the normal case, not the edge case.
- **Red flag: CAP invoked with no partition in the story,** and its twin, "more memory" as a fix for a race condition.
- **Red flag: stronger consistency aimed at the part that was never at risk,** such as a transaction around a read-only menu query while the last-plate race stays unprotected.
- Test the race: set stock to one, have two teammates tap on the count of three ten times, then fire twenty simultaneous requests from a script. More than one success is the paneer bug, on video instead of in production.

## Walkthrough
1. List every feature in your app on one page. Aim for ten to twenty lines; include the boring ones like profile photos and counters.
2. Next to each, write "strong" or "eventual". Force yourself to choose. Most lines will be eventual and that is normal and correct.
3. For every line you marked eventual, write what a user sees when it is wrong. "Menu shows a dish that is finished." "Like count is two lower than reality for a second."
4. For every line you marked eventual that involves stock, seats, or money, write the compensating action. If you cannot name one, change the line to strong or design the apology now.
5. For every line you marked strong, write what happens to that feature when the database is failing over for thirty seconds. The acceptable answer is a clear error, not a wrong answer.
6. Now find your races. Look for any feature where two users can take the same thing: last item in stock, a seat, a slot, a unique username, a referral code.
7. Pick the most important race and test it. Set the stock to one, then have two teammates tap at the same instant, ten times. Record how many times both succeeded.
8. Repeat with a script that fires twenty simultaneous requests at the same endpoint. Record the number of successes. Anything above one is a bug with a rupee value attached.
9. Trace the read path of your most consistency-sensitive feature through every layer of Chapter 2's diagram, and write the maximum staleness each layer adds. Sum it. That total, not your database's guarantee, is what the user actually experiences.
10. Write one decision-log entry naming a feature, its consistency choice, the bound in seconds if eventual, and the compensating action. This is the entry mentors ask about most often in M11.

## Talk to AI
```prompt
CONTEXT:      I am a student directing AI on my team's app. Stack: [your
              stack]. Database: [your database], free tier. Caching: [what
              you cache, and the TTL in seconds]. Read replicas: [yes/no].
              App instances: [how many]. Users are Indian students on
              slow 4G; payments are by UPI.

              My features, with what I currently believe about each:
              1. [feature] - I think it needs [strong/eventual]
              2. [feature] - I think it needs [strong/eventual]
              3. [feature] - I think it needs [strong/eventual]
              4. [feature] - I think it needs [strong/eventual]
              5. [feature] - I think it needs [strong/eventual]

              Features where two users could take the same thing:
              - [e.g. the last plate of a dish]
              - [e.g. a unique username or referral code]

GOAL:         A per-feature consistency decision I can defend, with the
              compensating action written down for everything eventual.

REQUIREMENTS: 1. For each feature, agree or disagree with my guess and say
                 why, in one sentence
              2. For each eventual feature, state the staleness bound in
                 seconds and what a user sees when it is wrong
              3. For each eventual feature touching stock, seats or money,
                 name the compensating action
              4. For each strong feature, tell me what happens to it
                 during a 30-second database failover
              5. For each race I listed, tell me the specific mechanism
                 that prevents a double-take, and what it costs in latency
              6. Trace one read through every cache layer I described and
                 sum the maximum staleness it can accumulate

CONSTRAINTS:  Free tier only. Do not propose a new database or a
              distributed lock service. Any mechanism must work with the
              tools I already listed. Assume simultaneous taps are normal
              at lunch rush, not an edge case.

NON-GOALS:    Do not invoke the CAP theorem unless you are describing an
              actual network partition. Do not tell me a feature is fine
              without naming what a user sees when it is wrong. No code
              until I have the table.

ACCEPTANCE:   - Every feature has a verdict, a bound or a mechanism, and a
                user-visible consequence
              - Every eventual money or stock feature has a compensating
                action
              - The staleness sum for my traced read is a number
              - You have disagreed with at least one of my guesses, or
                explained why all of them are right

ASK:          First give me the table. Then tell me which single feature is
              most likely to produce a real incident at lunch rush, and
              what test would prove it. No code until I pick one.
```

## Stuck? Ask AI
```prompt
Explain strong and eventual consistency to someone who has never programmed, using an analogy that is not about banks or ATMs. Then give me eight things an app might store and ask me, one at a time, which kind each one needs. Do not tell me the answers first.
```

```prompt
Explain the CAP theorem precisely, including what it does NOT say. Then give me three examples of people misusing it in architecture arguments, and tell me what the correct statement would have been in each case.
```

```prompt
Two students tap "order the last item" at the same moment and both succeed. Explain, step by step with timestamps, exactly how this happens even though my database is correct. Then show me three mechanisms that prevent it, ranked by how much latency each adds, and tell me which one a small team should use.
```

## Quiz

### Q1 · L2
What does eventual consistency mean?
- (a) The data will be correct eventually, but reads may return slightly old values ✅
- (b) The data is sometimes lost permanently
- (c) Writes are applied in a random order
- (d) Reads always fail during a network problem
> Copies converge after a short delay, so a read can be stale. "Eventual" with no stated bound is a hope rather than a design — attach a number of seconds.

### Q2 · L3
The CAP theorem forces a choice between consistency and availability:
- (a) At all times, for every system
- (b) Only when parts of the system cannot reach each other ✅
- (c) Only for NoSQL databases
- (d) Only when you have more than three replicas
> CAP is about behaviour during a partition. In normal operation the real everyday trade is latency versus freshness, which is why "by CAP we chose availability" is usually decoration.

### Q3 · L4
Your database is strongly consistent, and you put a 60-second cache in front of a stock-availability read. What is that read now?
- (a) Still strongly consistent, because the database is
- (b) Eventually consistent, with a bound of 60 seconds ✅
- (c) Strongly consistent for the first read and eventual afterwards
- (d) Undefined; caches do not affect consistency
> The weakest layer sets your real guarantee, and it is almost never the database. Every copy you make — browser, CDN, shared cache, replica — opts you into eventual consistency.

### Q4 · L4
Two instances both read a like count of 10 and both write 11. Conflicts are resolved by last-write-wins. What is the outcome?
- (a) The count becomes 12 as intended
- (b) The count becomes 11 and one increment is silently lost ✅
- (c) One write returns an error the user sees
- (d) The database rejects both writes
> Read-modify-write under last-write-wins destroys increments with no error anywhere. Counters need an operation the store can merge atomically, not a read followed by a write.

### Q5 · L5
You decide the "last plate in stock" feature may be eventually consistent at lunch rush. What must exist before you ship it?
- (a) A larger database instance
- (b) An automatic compensating action: refund, clear message, and a record ✅
- (c) A shorter cache TTL on the menu
- (d) A read replica in the same region
> Choosing availability means accepting you will sometimes be wrong, which is only a decision if the apology is written in advance. Without it you have not chosen availability, you have scheduled an incident.

## Go deeper
- [Jepsen: consistency models explained](https://jepsen.io/consistency)
- [Martin Kleppmann: please stop calling databases CP or AP](https://martin.kleppmann.com/2015/05/11/please-stop-calling-databases-cp-or-ap.html)
