---
id: m13-09-multi-region-and-dr
module: m13-production-engineering
slug: multi-region-and-dr
title: "Multi-Region, Failover, Backups and Disaster Recovery"
order: 9
duration_min: 15
is_free: false
mission: false
volatility: low
last_reviewed: 2026-11-16
---

## Key learnings
- Choose an RPO and an RTO as numbers, and say what each one costs you
- Run a restore drill, because an unrestored backup is not a backup
- Write a failover runbook including the parts everyone forgets
- Tell active-passive, active-active and read-replica setups apart by what they cost
- Catch a disaster-recovery plan that has never been executed once

## Lesson

### The pain

M05 chapter 6 taught replicas, replication lag and failover at overview level and promised M13 would go deep. M05 chapter 7 taught CAP and promised quorums and consensus. M06 chapter 7 taught regions, the latency floor and the bills. This chapter is the operational half: what you do when a region or a database is gone.

A team says, honestly, "we have backups" — a daily snapshot, fourteen days. Then a hurried migration rewrites order totals in paise as rupees, and three things nobody considered: nobody has ever restored, so the first attempt goes to the wrong target, the restore takes forty-one minutes, and the dish photos in object storage are in no backup at all.

### The idea

**Disaster recovery is two numbers you choose and one drill you run. Everything else is a diagram.**

**RPO** is how much data you accept losing, in time: a daily snapshot means up to twenty-four hours. **RTO** is how long you accept being down. Both are business decisions. And a backup you have never restored is a hypothesis — restoring it makes it a capability and hands you the RTO you would otherwise learn mid-incident.

### The analogy

M05 gave you photocopies of the stock board as replicas. This chapter is about two other objects. The first is the recipe binder's photocopy in a locked drawer: everyone knows it exists, nobody has opened it in a year, and the key might be with the manager who left. The second is a whole second canteen in another block, stocked and staffed and cooking nothing, which costs two kitchens every day until Block A floods.

**Where the analogy breaks:** opening the drawer proves the photocopy readable. A backup can be silently unusable for months — wrong database, rotated key, truncated by a quota, or twelve hours older than you believed.

### Picture it

```
  NORMAL                         FAILURE AT 10:04
 ┌─────────────┐                ┌─────────────┐
 │ Mumbai      │                │ Mumbai   ╳  │
 │ PRIMARY     │── replicate ─▶ │             │
 └─────────────┘   lag 1.2 s    └─────────────┘
        │                              │
        ▼                              ▼
 ┌─────────────┐                ┌─────────────┐
 │ Singapore   │                │ Singapore   │
 │ replica     │                │ PROMOTED    │
 │ (reads)     │                │ (writes)    │
 └─────────────┘                └─────────────┘

  RPO = the 1.2 s of writes that never replicated
  RTO = 14 min: promote + repoint app + DNS + cache + queues
```

Failover promotes a replica to primary, so the data you lose is whatever had not replicated at the moment of failure, and the time you are down is everything that has to be changed after the promotion — not just the database.

### Where it lives

- A second region is only sane as a **chapter 4** module instantiated twice; failover is triggered by **chapter 3's** health checks, with **chapter 10** around it.
- M06 chapter 13's **DNS TTL** applies directly: a failover changing a record with an hour TTL has an RTO of at least an hour.

**The honest verdict: you should not be multi-region, and you must have a tested restore.** Multi-region doubles your bill and makes replication lag a correctness problem, for a gain you cannot measure at two hundred users. A tested restore costs one afternoon.

**Quorum and consensus**, promised by M05 chapter 7: a write is accepted once a majority of copies confirms it, and consensus is how a managed database picks a new primary without **split-brain** — two primaries accepting conflicting writes. Hence managed failover is worth paying for.

### Words to own

| Term | Plain meaning | Example |
|---|---|---|
| RPO | How much data you accept losing, in time | Up to 5 minutes |
| RTO | How long you accept being down | Under 30 minutes |
| Snapshot | A full copy at one moment | Nightly |
| PITR | Point-in-time recovery: restore to any second | "10:03, before the migration" |
| Logical dump | An exported, portable copy of the data | Restorable anywhere |
| Restore drill | Actually restoring, into a scratch environment | Quarterly, dated |
| Failover | Promoting a standby to primary | Mumbai → Singapore |
| Active-passive | Standby exists, serves nothing until needed | Pays, waits |
| Active-active | Both regions serve traffic | Hardest and costliest |
| Replication lag | How far behind a replica is | 1.2 seconds |
| Read-your-writes | A user always sees their own last write | Read the primary |
| Split-brain | Two copies both believe they are primary | Manual reconciliation |
| Quorum | A majority of copies must confirm a write | 2 of 3 |
| Runbook | Numbered steps for a specific failure | Followed at 2 a.m. |
| Blast radius | How much one failure can reach | One region, or all |

### Tradeoffs and gotchas

**Back up more than the database:** object storage, environment variables, DNS records, chapter 4's state file — everything that cannot be regenerated from git.

**Point-in-time recovery beats snapshots.** A snapshot answers "restore yesterday"; PITR answers "restore to 10:03, before the migration". Without it your RPO is the snapshot interval.

**Replication lag is a correctness problem.** A confirmation page reading a replica 1.4 seconds behind tells a student "you have no orders", so they pay twice. Route a user's reads to the primary briefly after their own write.

**The runbook's missing steps are always the same five:** connection strings pointing at the old primary, an hour-long DNS TTL, the dead region's cache as a hard dependency, workers on a dead queue, cron running twice.

**Decide in advance who declares a failover.** It is destructive, and failing back is harder than failing over: one named person, one trigger.

**Active-active is a different product, not a configuration:** two regions accepting writes means conflicting writes to resolve. Price it first, including cross-region egress.

### Verify it

One question exposes a fake plan: **when did you last restore, and how long did it take?**

1. **Restore into a scratch environment and query it.** Count the orders and check a total; a successful restore of staging is the wrong thing.
2. **Time it — that is your real RTO.** Then check the stated RPO against the backup schedule.
3. **Ask what is *not* backed up:** object storage, environment variables, DNS records, infrastructure state.
4. **Read the runbook for the five forgotten steps,** and check read-your-writes on every page shown right after a write.

Red flags: a backup nobody has restored; an unmeasured RTO; a nightly snapshot paired with a five-minute RPO; a runbook that ends at promotion.

## Walkthrough
1. Write down your RPO and your RTO as two numbers, and next to each one write the sentence that justifies it from your product rather than from your tooling. For a canteen app: "we can lose up to one hour of orders; we cannot be down for more than two hours at lunch."
2. List everything you own that cannot be regenerated from your git repository. Database, object storage, environment variables, DNS records, infrastructure state, third-party configuration. This list is the actual scope of disaster recovery, and most teams find it is five items, not one.
3. For each item, find and write down its current backup method, schedule and retention. Mark the items with no backup. Fix the easiest gap today.
4. Check whether your database's free tier includes point-in-time recovery and for how long a window. Write down the answer, because it determines whether your real RPO is "any second in the last N days" or "last night".
5. Now run the drill. Restore last night's backup into a brand-new scratch database. Do not restore over anything.
6. Prove the restore by querying it: count the orders, check one known total, confirm the most recent timestamp. Write down how old the newest row is — that is your measured RPO, as opposed to your intended one.
7. Time the whole drill with a stopwatch, from decision to queryable data. Write the number in your runbook. That is your RTO and it is now a fact rather than a hope.
8. Write the failover runbook as numbered steps for one specific failure — "our primary database is unreachable" — and include the five forgotten items: connection strings, DNS TTL, cache, queue consumers, cron jobs. Name the single person who may declare it.
9. Lower the TTL on any DNS record that your failover would change, and record both the old and new value. Your RTO cannot be shorter than that number.
10. Find every page in your product that a user sees immediately after their own write, and confirm it reads from the primary. If you have a replica, deliberately introduce lag and watch the confirmation page lie to you.
11. Finally, price a second region honestly: compute, storage and cross-region egress per month in rupees, with the pricing page for each. Then write one paragraph deciding against it, with your reasons, in your decision log. That paragraph is a strong answer in M18 and "we built multi-region" usually is not.

## Talk to AI
```prompt
CONTEXT:      I am a student on an AI-native engineering program. I direct AI
              and verify its output; I do not write code by hand. My team of
              three runs [product in two sentences] with [number] real users
              in India, mostly around [city]. Stack: [your stack]. Database:
              [name and plan] in region [name]. We also store [photos / files
              / other] in [object storage]. Our backups today are: [describe
              honestly, including "we are not sure"]. Nobody has ever restored
              one. We deploy on [platform] and our DNS is at [provider]. A
              surprise bill of a few thousand rupees is a serious problem.

GOAL:         A disaster-recovery plan with an RPO and RTO I chose, a backup
              covering everything that cannot be regenerated from git, a
              restore I have actually performed and timed, and a failover
              runbook that does not stop at "promote the replica".

REQUIREMENTS: 1. Help me choose an RPO and an RTO from my product's needs,
                 and tell me what each number costs to achieve. Then tell me
                 whether my CURRENT backup schedule can deliver them
              2. Give me the complete list of things I must back up that are
                 NOT the database - object storage, environment variables, DNS
                 records, infrastructure state, third-party configuration -
                 and a backup method for each
              3. Tell me whether my database plan includes point-in-time
                 recovery and for what window, where I verify that myself, and
                 why PITR matters more than snapshots for a bad migration
              4. Give me the exact steps to restore into a brand-new scratch
                 database without touching production, plus the queries I run
                 to PROVE the restore contains the right data
              5. A failover runbook as numbered steps for "our primary
                 database is unreachable", explicitly including connection
                 strings, DNS TTL, the cache, queue consumers and cron jobs.
                 Tell me which step people forget most
              6. Explain read-your-writes: for each page a user sees right
                 after their own write, which database it must read from and
                 what the user experiences if it reads a lagging replica
              7. Explain quorum, consensus and split-brain in plain words, and
                 why I should never hand-roll a failover script
              8. Price a second region per month in rupees as separate meters
                 including CROSS-REGION EGRESS, with a pricing page per meter.
                 Then make the honest case that my team should NOT be
                 multi-region

CONSTRAINTS:  Free tier only where possible; say clearly where it is not and
              give the cheapest honest path in rupees. Plain English for a
              second-language reader. Label every command READ-ONLY or
              CHANGES-THINGS and name the point of no return. No step may risk
              our production data - restores go to a new target only. Assume
              nobody on the team is on call.

NON-GOALS:    Do not design an active-active setup. Do not add a second region
              in this conversation. Do not recommend a paid backup product
              before telling me what my existing plan already includes. Do not
              write a custom failover script.

ACCEPTANCE:   - My RPO and RTO are numbers with product reasons, and I know
                whether my current backups can deliver them
              - Every item that cannot be regenerated from git has a named
                backup method
              - I have restored a backup into a scratch environment and
                queried it to prove it holds the right data
              - I have a measured RTO from a stopwatch, written in the runbook
              - My runbook includes connection strings, DNS TTL, cache, queue
                consumers and cron
              - One named person may declare a failover, on a written trigger
              - Every post-write page reads from the primary
              - I have a rupee figure for a second region and a written
                decision against it

ASK:          Start by asking me what my newest backup actually contains and
              whether anyone has opened it. Then tell me which single gap in
              my current setup would hurt most, and fix that one first before
              anything else.
```

## Stuck? Ask AI
```prompt
I have backups enabled on my database and I want to know whether they are real. Give me a checklist of read-only checks that would tell me, without restoring anything: what exactly is included, what is excluded, the schedule, the retention, whether point-in-time recovery is available and for how long, and where the backup is stored. Then tell me the three ways a backup can look healthy in a dashboard and be unusable in practice, and which single test is the only one that settles it.
```

```prompt
Explain RPO and RTO with three worked examples for a small food-ordering app: a nightly snapshot only, snapshots plus point-in-time recovery, and a hot standby in a second region. For each, give me the RPO and RTO as numbers and the monthly cost in rough rupees. Then ask me what my product can actually tolerate losing and help me pick, pushing back if my numbers are stricter than my budget.
```

```prompt
Write me a failover runbook for "our primary database is unreachable" and then critique your own runbook: which steps would be forgotten at 2 a.m., which step is the point of no return, which step depends on a DNS TTL I should have lowered in advance, and what breaks if my background workers and cron jobs are still pointed at the dead region. Number every step and mark each one as reversible or not.
```

## Quiz

### Q1 · L3
Your team has nightly snapshots with fourteen days of retention. What is your RPO?
- (a) Fourteen days
- (b) Up to twenty-four hours, because the newest backup may be almost a day old ✅
- (c) Zero, because snapshots are continuous
- (d) However long a restore takes
> RPO is how much data you accept losing, measured in time, and it is set by the gap between backups. Retention tells you how far back you can go; the interval tells you how much you lose.

### Q2 · L4
A student pays, and the confirmation page reads from a replica 1.4 seconds behind. What do they see, and what is the fix?
- (a) A slow page; add a cache in front of the replica
- (b) "You have no orders", so they pay twice — route a user's reads to the primary briefly after their own write ✅
- (c) A stale total; increase the replica's memory
- (d) An error page; add a retry with backoff
> Replication lag is a correctness problem, not a performance one. Read-your-writes is decided per feature: a user's own confirmation reads the primary, everything else reads wherever is fastest.

### Q3 · L4
Your database promotes a standby in two minutes, but the outage lasts forty. What most likely consumed the other thirty-eight?
- (a) The database took longer than reported to become consistent
- (b) Connection strings, a long DNS TTL, an unreachable cache, queue consumers and cron still pointed at the dead region ✅
- (c) The load balancer's health checks had a long interval
- (d) The restore had to complete before the standby could accept writes
> Promotion is one step of a runbook. These five are the steps every team forgets, which is why they are written down as numbered items before the incident rather than recalled during it.

### Q4 · L4
A team insists their backups are fine because the dashboard shows green for ninety days. What is the only thing that settles it?
- (a) Checking the backup file size is consistent
- (b) Verifying retention is long enough
- (c) Restoring into a scratch environment and querying the data ✅
- (d) Enabling a second backup schedule as redundancy
> A backup nobody has restored is a hypothesis. It can be the wrong database, encrypted with a rotated key, truncated by a quota, or simply older than believed — and none of those is visible from a dashboard.

### Q5 · L5
Your team has two hundred users and a mentor suggests multi-region for reliability. What is the strongest honest response?
- (a) Agree; multi-region is the standard production architecture
- (b) Decline: it doubles the bill and adds lag as a correctness problem, while the failure that actually happens to us is a bad migration — so a tested restore buys far more ✅
- (c) Agree, but only in active-active mode so both regions are useful
- (d) Decline because our free tier does not support a second region
> The present-tense question from M06 chapter 9 applies: what breaks today without it? For a student team the realistic disasters are a careless delete, a bad migration and an unread destroy plan — all answered by PITR and a drilled restore, none by a second region.

## Go deeper
- [AWS: disaster recovery strategies, RPO and RTO explained](https://docs.aws.amazon.com/whitepapers/latest/disaster-recovery-workloads-on-aws/disaster-recovery-options-in-the-cloud.html)
- [PostgreSQL: continuous archiving and point-in-time recovery](https://www.postgresql.org/docs/current/continuous-archiving.html)
