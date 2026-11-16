---
id: m13-09-multi-region-and-dr
module: m13-production-engineering
slug: multi-region-and-dr
title: "Multi-Region, Failover, Backups and Disaster Recovery"
order: 9
duration_min: 30
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

M05 chapter 6 taught replicas, replication lag and failover at overview level, and promised M13 would go deep on failover and multi-region. M05 chapter 7 taught CAP and said M13 would take it into quorums and consensus. M06 chapter 7 taught regions, the latency floor between Nashik and Virginia, and the bills that come with both. This chapter is the operational half: what you actually do at 10:04 a.m. when a region or a database is gone.

A team says, honestly, "we have backups". They do: a daily snapshot, fourteen days of retention, visible in the dashboard. At 10:04 on a Tuesday a migration written in a hurry drops a column and rewrites order totals in paise as rupees. They decide to restore last night's snapshot. Four things then happen that nobody had thought about. The snapshot list contains two databases with similar names and they are not sure which is production. Nobody has ever performed a restore, so the first attempt goes to the wrong target. The restore itself takes forty-one minutes, a number they learn for the first time during the incident. And the dish photos in object storage are in no backup at all, because what they set up was the database's built-in one.

Second failure, cheaper but sharper. A team adds a read replica in Singapore to speed up reads. A student pays through UPI, the confirmation page reads from the replica, the replica is 1.4 seconds behind, and the page says "you have no orders". The student pays again. Now there are two charges, a support conversation, and a refund — caused by an optimisation, in a system where nothing was broken.

### The idea

**Disaster recovery is two numbers you choose and one drill you run. Everything else is a diagram.**

**RPO**, recovery point objective, is how much data you accept losing — measured in time. A daily snapshot is an RPO of up to twenty-four hours. **RTO**, recovery time objective, is how long you accept being down. These are business decisions, not technical ones: for a canteen app, losing an hour of orders is survivable and being down for a day is not. For a payments feature, neither is acceptable, which is why you were told in M05 chapter 7 not to be the source of truth for money.

And the drill: a backup you have never restored is a hypothesis. The restore drill converts it into a capability and, incidentally, gives you the RTO number you would otherwise learn during an incident.

### The analogy

M05 gave you photocopies of the canteen's stock board as replicas. This chapter is about two different objects.

The first is the recipe binder's photocopy, kept in a locked drawer. Everyone knows it exists. Nobody has opened it in a year. It might be last year's binder, it might be missing the new supplier's pages, the key might be with the manager who left. Opening it — reading a page, cooking from it — is the only thing that converts it from a comfort into a capability.

The second is a whole second canteen in another hostel block, stocked, staffed, cooking nothing. It will serve students the day Block A floods. It also costs two kitchens' worth of staff and stock every day until then, and its menu must be kept in step with Block A or it will serve last month's prices on the day it matters.

**Where the analogy breaks:** opening the drawer proves the photocopy is readable in one glance. A backup can be silently unusable for months — pointed at the wrong database, encrypted with a key that was rotated, truncated by a quota, or perfectly valid and twelve hours older than you believed. There is no equivalent of "looking at it". The only test is a full restore into a scratch environment, which is why it is a drill with a date on it rather than an inspection.

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

A second region is only sane as a chapter 4 module you instantiate twice, because a console session repeated from memory under pressure is how regions drift apart. Failover is driven by chapter 3's health checks deciding the primary is gone. And M06 chapter 13's DNS TTL applies directly: if your failover changes a DNS record and your TTL is an hour, your RTO is at least an hour no matter how fast the database promotes. Chapter 10 is the human process that wraps all of this.

The honest verdict for your startup: **you should not be multi-region, and you must have a tested restore.** Multi-region doubles your bill and your deployment surface, and adds replication lag as a correctness problem, for a reliability gain you cannot measure at two hundred users. A tested restore costs one afternoon and protects against what actually happens to student teams: a bad migration, a careless delete, or a destroy plan nobody read.

**Quorum and consensus**, promised by M05 chapter 7, belong here in one paragraph. When several copies must agree, a write is accepted once a majority confirms it: three copies tolerate one failure, five tolerate two. Consensus protocols are how managed databases pick a new primary without two copies both believing they are — the split-brain case, where two primaries accept conflicting writes that someone must reconcile by hand. You will not implement this. You need to know it is why managed failover is worth paying for and a hand-rolled failover script is dangerous.

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

**Back up more than the database.** What student teams lose is rarely the database. It is the object storage holding dish photos, the environment variables nobody wrote down, the DNS record set, and chapter 4's state file. List everything that cannot be regenerated from git, and give each item its own backup method.

**Point-in-time recovery is worth far more than snapshots for the failure you will actually have.** A nightly snapshot answers "restore yesterday". PITR answers "restore to 10:03, one minute before the migration", which is the question a bad migration asks. Check whether your database's free tier includes it, because many do for a short window, and if it does not, your RPO is the snapshot interval and you should say that out loud.

**Replication lag is a correctness problem, not a performance one.** A replica serving a user's own confirmation page is the double-payment bug above. The rule from M05 chapter 7: route a user's reads to the primary for a short window after their own write, and route everything else wherever is fastest. Decide this per feature, not per app.

**The failover runbook's missing steps are always the same ones.** The database promotes in two minutes and the outage lasts forty, because the connection string still points at the old primary, the DNS TTL is an hour, the cache in the dead region is unreachable and treated as a hard dependency, the workers still consume from a queue that is gone, and cron now runs in two places. Write all five into the runbook as numbered steps, because nobody remembers them at 10:04.

**Decide in advance who declares a failover.** It is destructive and hard to reverse — failing back is often harder than failing over. One named person, one written trigger. This is chapter 10's incident commander, and failover is the decision that most needs one.

**Active-active is a different product, not a configuration.** Two regions accepting writes means conflicting writes, which you must resolve somehow — M05 chapter 7's last-write-wins, which silently loses increments, or a rule you design per entity. It is a correctness project, not a reliability improvement.

**Put a real number on the cost first.** A second region is a second set of every meter from M06 chapter 7 — compute, storage, and cross-region egress, which surprises people because replication traffic is egress and it runs continuously. Price it per month in rupees before the architecture discussion, not after.

### Verify it

The single question that exposes a fake disaster-recovery plan: **when did you last restore, and how long did it take?** If the answer is "we have backups enabled", there is no plan.

1. **Restore into a scratch environment and query the data.** Not "the restore completed" — open it, count the orders, check a total. A restore that completes and contains the staging database is a successful restore of the wrong thing.
2. **Time it and write the number down.** That is your real RTO, and it is almost always several times what anyone guessed.
3. **Ask what is *not* backed up.** Make the list explicitly: object storage, environment variables, DNS records, infrastructure state, third-party configuration. Every item without a named backup method is a gap.
4. **Ask for the RPO as a number and check it against the backup schedule.** An RPO of five minutes with a nightly snapshot is a contradiction, and it is a very common one in generated plans.
5. **Read the failover runbook for the five forgotten steps**: connection strings, DNS TTL, cache, queue consumers, cron. Any runbook that stops at "promote the replica" is a third of a runbook.
6. **Ask who declares it and on what trigger.** A plan with no named person is a plan that will be debated for twenty minutes during the thing it was written for.
7. **Check read-your-writes per feature.** Find every page a user sees immediately after their own write and confirm which database it reads from.

Red flags: a backup nobody has ever restored; an RTO nobody has measured; object storage absent from the backup list; a nightly snapshot paired with a five-minute RPO; a failover runbook that ends at promotion; multi-region proposed for a product with two hundred users; active-active described as a reliability improvement; a DNS-based failover with an hour-long TTL; and a hand-written failover script with no protection against two primaries.

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
