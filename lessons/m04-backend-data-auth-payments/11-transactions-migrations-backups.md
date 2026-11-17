---
id: m04-11-transactions-migrations-backups
module: m04-backend-data-auth-payments
slug: transactions-migrations-backups
title: "Transactions, ACID, Migrations and Backups"
order: 11
duration_min: 15
is_free: false
mission: false
volatility: low
last_reviewed: 2026-11-16
---

## Key learnings
- Wrap writes that must succeed or fail together in one transaction
- Say what each letter of ACID protects you from, with a real failure
- Change a live schema without losing data or breaking the running app
- Treat a backup as untested until you have restored from it
- Verify AI's migration by asking what happens to the rows that already exist

## Lesson

### The pain

A student pays ₹240 by UPI. The server does three writes: deduct ₹240 from the wallet, mark the order paid, reduce the stock of two samosas. The first two succeed; the third fails because the connection dropped. The student has been charged, the order says paid, and the stock still promises two sold samosas — and nobody is watching.

The same week has two more versions: a `NOT NULL` phone column that works on an empty dev database and fails on the deployed one with 180 phone-less users, and a cleanup script run against production where the never-tested backup turns out to be of a different database.

### The idea

**A transaction makes several writes behave as one: all of them happen, or none of them do.**

Wrapped in a transaction, the payment above cannot end up half-done.

**ACID** names the four guarantees. **Atomic**: all or nothing. **Consistent**: chapter 6's constraints hold at the end. **Isolated**: two transactions at once do not see each other's half-finished work. **Durable**: once committed, it survives a power cut. A **migration** is a recorded, reviewable schema change; a **backup** is a copy you have actually restored from.

### The analogy

The canteen manager records a sale in three places: the cash register, the order register and the stock register. A transaction is the rule that she makes all three entries or none — money recorded with stock not is her Sunday reconciliation. Isolation is why two staff cannot both sell the last plate of biryani. A migration is adding a column to a register that already has 180 filled rows.

**Where the analogy breaks:** she can spot a half-written line. A database with a half-finished write looks completely normal — no error anywhere, and you find out when a student complains two days later. That missing symptom is why the guarantee must be built into the write.

### Picture it

```
 WITHOUT A TRANSACTION          WITH A TRANSACTION
 ┌───────────────────┐          ┌───────────────────┐
 │ deduct ₹240  ✓    │          │ BEGIN             │
 │ mark paid    ✓    │          │  deduct ₹240  ✓   │
 │ reduce stock ✗    │          │  mark paid    ✓   │
 └───────────────────┘          │  reduce stock ✗   │
                                │ ROLLBACK          │
 charged, order paid,           └───────────────────┘
 stock wrong.
 Nobody is told.                nothing happened.
                                the student retries.
```

Three writes left to themselves can stop halfway and leave the data inconsistent with no error anywhere; the same three inside a transaction either all take effect or are all undone.

Migrations need the same care across three deploys, because old code keeps running while the new schema arrives.

| Step | What you do |
|---|---|
| Expand | Add the column nullable or with a default — old code ignores it |
| Migrate | Backfill existing rows, in batches if there are millions |
| Contract | Only now make it `NOT NULL` and drop whatever it replaced |

### Where it lives

Transactions live in your server code, around the database calls — the arrow between M00's server and data boxes. The guarantee comes from the database, but **any two writes that would embarrass you if only one happened belong in one transaction.** Chapter 17 makes this unavoidable, chapter 16 takes the slow work out, M06 runs migrations in your deploy, M07 treats lost data as an incident.

### Words to own

| Term | Plain meaning | Example |
|---|---|---|
| Transaction | A group of writes that all happen or none do | payment + status |
| Commit | Making a transaction's writes permanent | after all three succeed |
| Rollback | Undoing a transaction's writes | the stock update failed |
| Atomicity | All or nothing | the "A" in ACID |
| Isolation | Concurrent transactions do not see half-work | the last biryani |
| Race condition | Two writers interleaving badly | double-selling stock |
| Migration | A recorded, reviewable schema change | `add_phone_to_users` |
| Backfill | Filling a new column on existing rows | 180 users |
| Rollback plan | How you undo a deploy's schema change | written in advance |
| Retention | How far back your backups go | 7 days |

### Tradeoffs and gotchas

**Do not put slow or external work inside a transaction.** An email, a payment API call, an image resize — each holds locks open while it waits, and other writers queue behind it. Write the row, queue the job, commit.

**Isolation is not absolute.** The default level prevents the obvious disasters and still permits surprising interleavings. The guard for "two students bought the last samosa" is a constraint or a conditional update that fails the second writer.

**AI writes migrations against an empty database.** Say so in every brief: "the deployed table has N rows; tell me what happens to them." `NOT NULL` with no default fails outright, with a default it can lock a large table, and dropping a column is irreversible.

**Never point an agent at production.** Keep scratch and real visibly different. One careless cleanup script is the commonest way a student project loses a month.

### Verify it

Six checks; the fourth is the one almost nobody does.

1. **Confirm the writes that must go together are in one transaction.** List every endpoint writing to more than one table; money first.
2. **Force a failure in the middle.** Break the last write, call the endpoint, check the database — surviving earlier writes mean there is no transaction.
3. **Ask every migration what happens to existing rows.** Count them first. "It fails" and "old rows get NULL" are both answers you need beforehand.
4. **Restore a backup into a scratch database, once, this month.** Check your rows are there and time it — that is your recovery time. Daily backups and a 1 p.m. failure means eleven hours lost; chapter 15's uploaded files are usually not in the backup.
5. **Check nothing slow sits inside a transaction** — nothing external between a begin and a commit.
6. **Confirm scratch and real connection strings are visibly different,** and that the real one is nowhere an agent can read.

Red flags: three sequential writes in a payment handler with no transaction; `NOT NULL` with no default against an existing table; a migration file edited after it had already run; "the backup is automatic" as the answer to "have you restored one?"; a rename done in one step while the old code is still deployed; an HTTP call to a payment provider inside a transaction; a rollback plan dismissed because "we tested it locally".

## Walkthrough
1. List every endpoint in your project that writes to more than one table. If you have none yet, list the ones you will have — a signup, a payment, an order.
2. For each one, write the sentence "if only the first write happened, the user would see ___". That sentence tells you whether a transaction is required.
3. Read the code of your most important multi-write endpoint. Confirm with your own eyes that one transaction wraps all the writes. Note what you find.
4. Now break it on purpose. In a scratch environment, make the last write fail — misspell a column, violate a constraint — and call the endpoint.
5. Inspect the database. If the earlier writes survived, you have proved there is no working transaction. Write a precise iteration asking for one, scoped to that endpoint only.
6. Repeat step 4 after the fix and confirm the database is unchanged. That is your proof.
7. Separately, check for slow work inside transactions: search your code for an email, an HTTP request or a file operation sitting between a begin and a commit. List every one you find.
8. Find how your project's migrations are stored. Confirm they are files in git, in order, and that none has been edited since it ran.
9. Count the rows in your largest deployed table. Then take your next planned migration and ask AI directly what happens to those existing rows when it runs.
10. Find your database provider's backup settings. Write down the frequency and the retention. Then calculate your worst-case data loss in hours.
11. Restore a backup into a scratch database. Open it, count the rows in one table you care about, and time the whole exercise. Then check your connection strings: confirm scratch and real are visibly distinguishable and that the real one is not in any file an agent can read.

## Talk to AI
```prompt
CONTEXT:      Three-person student team, free-tier managed PostgreSQL, and
              a deployed app with real rows in it. We have been generating
              code with AI and have never checked our write safety.
              Money is stored as integer paise.
              Our deployed row counts: [table: count, for the top 3-4]

              THE ENDPOINT I AM WORRIED ABOUT — it writes to more than one
              table:
              [paste the handler code]

              THE MIGRATION I AM ABOUT TO RUN:
              [paste it, or describe the schema change in words]

GOAL:         Make this endpoint's writes all-or-nothing, and make this
              migration safe to run against a database that already has
              rows in it.

REQUIREMENTS: 1. Tell me, for the endpoint, every partial state that is
                 currently possible if a write fails halfway, and what the
                 user would see in each case
              2. Show me where the transaction boundary must go, and name
                 anything currently inside it that should be outside —
                 emails, HTTP calls, file work — and where it should go
                 instead
              3. Give me a way to PROVE the transaction works: a specific
                 failure I can force, and exactly what I should see in the
                 database afterwards
              4. For the migration, tell me what happens to my existing
                 rows when it runs, using my real row counts
              5. If it would fail or produce nulls, rewrite it as expand,
                 migrate, contract — as separate steps I deploy separately
                 — and tell me what the old code does between each step
              6. Give me the rollback plan for each step, and say clearly
                 which step becomes irreversible
              7. Tell me what my provider's automatic backups would and
                 would not cover if this migration destroyed data

CONSTRAINTS:  Free-tier PostgreSQL, so avoid anything that locks a table
              for a long time. The app must stay up while the migration
              runs. Do not change behaviour beyond write safety. Assume the
              old version of the code is still running during the deploy.

NON-GOALS:    Do not refactor the endpoint's logic. Do not add caching. Do
              not touch any other endpoint or table. Do not propose a
              different database.

ACCEPTANCE:   - I can force a mid-sequence failure and confirm the database
                is completely unchanged afterwards
              - Nothing slow or external remains inside the transaction
              - The migration runs successfully against my real row counts
              - Each migration step has a stated rollback, and the
                irreversible one is named

ASK:          Before any code, list every partial state my endpoint can
              currently reach and tell me which of them has already
              probably happened in production. I want to know the damage
              before the fix.
```

## Stuck? Ask AI
```prompt
Explain each letter of ACID using a canteen that records sales in a cash register, an order register and a stock register. For each letter, describe one specific thing that goes wrong if that guarantee is missing, with a number in it.
```

```prompt
I am about to add a required column to a table that already has 180 rows. Explain why a single migration will fail, then walk me through expand, migrate, contract as three separate deploys, telling me what the old code and the new code each do between the steps.
```

```prompt
My database provider says backups are automatic. Give me the list of questions I should be able to answer about those backups before I trust them, and tell me which single one I can only answer by actually doing something rather than reading a settings page.
```

## Quiz

### Q1 · L2
What does a transaction guarantee?
- (a) Each write is retried until it succeeds
- (b) All the writes in it happen, or none of them do ✅
- (c) Writes happen in the order they were issued
- (d) The database is backed up before the writes
> That is atomicity, the "A" in ACID. A payment that deducts money and then fails to mark the order paid is exactly the state a transaction makes impossible.

### Q2 · L3
AI writes a migration adding a `NOT NULL` phone column with no default. It works on the developer's machine and fails on deploy. Why?
- (a) The deployed database uses a different PostgreSQL version
- (b) The developer's database was empty; the deployed one has 180 rows with no phone value ✅
- (c) `NOT NULL` is not supported on free tiers
- (d) The migration ran out of its execution time limit
> This is the question to ask every AI-written migration: what happens to the rows that already exist? AI tests against empty databases almost by default.

### Q3 · L4
How do you actually prove an endpoint's writes are transactional?
- (a) Read the code and confirm a transaction wrapper is present
- (b) Ask the AI to confirm it used a transaction
- (c) Force the last write to fail, call the endpoint, and check the database is unchanged ✅
- (d) Run the endpoint many times and check for inconsistencies
> A guarantee you have not tested is a claim. Breaking the final write deliberately is a two-minute experiment that distinguishes a real transaction from the appearance of one.

### Q4 · L4
Which of these should NOT be inside a database transaction?
- (a) Inserting an order and its line items
- (b) Deducting a wallet balance and updating an order status
- (c) Calling a payment provider's API and waiting for its response ✅
- (d) Inserting a row into a background job queue
> External calls hold locks open across the network for however long the other side takes. Write the row, queue the job, commit — and let chapter 16's worker make the slow call.

### Q5 · L5
Your provider takes automatic daily backups with seven-day retention. What is the one thing you cannot learn from the settings page?
- (a) How far back you could recover
- (b) How often backups are taken
- (c) Whether a restore actually works and how long it takes ✅
- (d) Whether backups are encrypted
> An untested backup has an unknown success rate. Restoring into a scratch database once, and timing it, converts a hope into a recovery time you can quote.

## Go deeper
- [PostgreSQL docs: transactions and concurrency control](https://www.postgresql.org/docs/current/mvcc.html)
- [Strong Migrations: safe schema changes against live data](https://www.postgresql.org/docs/current/sql-altertable.html)
