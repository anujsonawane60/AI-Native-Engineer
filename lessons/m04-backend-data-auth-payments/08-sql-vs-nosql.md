---
id: m04-08-sql-vs-nosql
module: m04-backend-data-auth-payments
slug: sql-vs-nosql
title: "SQL vs NoSQL, and When to Use Each"
order: 8
duration_min: 15
is_free: false
mission: false
volatility: low
last_reviewed: 2026-11-16
---

## Key learnings
- Choose a database type from your query patterns, not from what sounds modern
- Explain what a relational database enforces that a document store will not
- Name the four NoSQL families and the one job each is genuinely good at
- Spot the "schemaless means no schema" mistake and what it costs later
- Check that AI's database recommendation matches the questions you must answer

## Lesson

### The pain

A team picks a document database because a blog post called it faster and more flexible, and for three weeks it is lovely: no tables, no migrations, just save an object. Then the product manager asks which dishes made the most money last month. In a relational database that is one query with two joins; here it means fetching every order of the month and looping in code — nine seconds at 30,000 orders. Worse, a dish renamed in week two now appears under two names, its old name copied into 4,000 documents.

### The idea

**Pick your database from the questions you need to answer and the rules you need enforced, not from the word on the front of it.**

A **relational (SQL) database** stores rows in tables with a declared structure, enforces constraints, and combines tables in one query using joins. **NoSQL** is four families that share only "not relational":

| Family | Genuinely good at | Where it fits |
|---|---|---|
| Document store | One document = one screen's data | an order, read whole |
| Key-value store | Dictionary lookup by key, very fast | sessions, counters |
| Wide-column | Enormous write volumes across machines | event streams |
| Graph | "Friends of friends of friends" cheaply | recommendations |

For a product with users, orders and money, the honest default is relational — Postgres — and this module assumes one.

### The analogy

The canteen's ruled register is a relational database: every line has the same columns, a student id that does not exist is refused by the clerk, and the manager can total any column at month end. A document store is a shoebox of order slips — each slip complete, so reading one order is instant, but "which dish earned the most in September" means counting 4,000 slips by hand.

**Where the analogy breaks:** real shoeboxes do not silently accept a slip with the quantity written in the price box. Document stores do, which is how three shapes of order accumulate by week six, unnoticed until a screen renders blank.

### Picture it

```
 RELATIONAL                   DOCUMENT STORE
 students    orders           orders collection
 ┌────────┐ ┌──────────┐      ┌──────────────────────┐
 │ id     │◀│student_id│      │ { _id: 91,           │
 │ name   │ │ total    │      │   student: {name,ph},│
 │ phone  │ │ status   │      │   items: [ {name,    │
 └────────┘ └────┬─────┘      │     price, qty} ],   │
                 │            │   total: 240 }       │
            order_items       └──────────────────────┘
            ┌──────────┐
            │order_id  │      one read = one screen
            │item_id   │      no joins available
            │qty, price│      name copied 4000 times
            └──────────┘

 "revenue per dish" = 1 join   "revenue per dish" = fetch
                               everything, count in code
```

The same orders stored as linked rows versus self-contained documents: the first makes cross-cutting questions cheap, the second makes single-screen reads cheap and cross-cutting questions expensive.

### Where it lives

Still the "data" box from M00, and the choice colours everything after it.

- Chapters 9, 11, 14 and 17 — joins, transactions, migrations, row-level security, payments — all assume relational: a product with money and permissions needs enforcement, and enforcement is what relational databases sell.
- Where NoSQL belongs, it sits *alongside* Postgres: a key-value cache in M05, vector storage for RAG in M10 (Postgres does that too), wide-column stores in M13.

### Words to own

| Term | Plain meaning | Example |
|---|---|---|
| Relational / SQL | Tables with declared structure and joins | Postgres |
| Join | Combining rows from two tables in one query | orders + students |
| Document store | JSON-like documents, no joins | one order per document |
| Key-value store | Fast dictionary lookup by key | sessions, counters |
| Wide-column | Huge write volumes across machines | event logs |
| Graph database | Relationships as first-class data | friend-of-friend |
| Schemaless | No structure declared up front | shape drifts over time |
| Embedding vs referencing | Copying data in versus pointing at it | items inside an order |
| Vector store | Storage searchable by meaning | M10's RAG |
| Eventual consistency | A read may briefly return stale data | replicas in M05 |

### Tradeoffs and gotchas

Three questions settle the choice. A fourth often decides it: **AI generates relational code far more reliably.**

| Ask | If yes |
|---|---|
| Do my queries cut across many records? | Relational, without further debate |
| Does one record equal one screen, always? | A document store fits — one dashboard ends that |
| Must the database enforce rules — keys, uniqueness, paired writes? | Relational |

**"Schemaless" means the schema moved into your code, where nobody can see it.** Every screen still depends on a structure, but it is now implicit, unenforced, and different in documents written in different weeks.

**Copying data into a document is a decision about history.** Embedding the dish name and price is right if you mean "what was sold, at what price" (chapter 6's `unit_price_paise` reasoning) and wrong if you mean "its current name", because 4,000 copies cannot be fixed atomically.

**Postgres does JSON too, which resolves most arguments.** A `jsonb` column holds genuinely variable data — a webhook payload, free-form preferences — inside a relational table with real constraints.

**Two databases triple your operational work.** Two sets of credentials, two backups, two failure modes, and no transaction spanning both — a write that must land in both can half-fail.

### Verify it

Five checks on a database recommendation.

1. **Did it ask what questions you need to answer?** Any recommendation made before someone listed the reports and screens is a guess.
2. **Name your hardest cross-cutting query and ask how it is served.** If the answer loops over fetched records in application code, you have found the cost.
3. **Ask which rules the database will enforce.** "The application handles that" means you own enforcement forever, including on the paths AI writes next month.
4. **Ask what a schema change looks like.** In Postgres, a reviewable migration; in a document store, defensive code in every reader for documents lacking the field.
5. **Count the databases.** More than one needs a written justification naming what the first cannot do.

Red flags: "scalable" and "flexible" with no query from your product; the same mutable fact embedded in thousands of documents; a key-value store as the system of record for orders; two databases with no users yet; "we can always migrate later", true and a month expensive; silence on money when two writes must succeed together.

## Walkthrough
1. Write down the five most important questions your product must be able to answer about its own data. Real sentences, like "how much did we take yesterday" or "which items has this user saved".
2. Mark each one as single-record ("this one order, complete") or cross-cutting ("across all orders, grouped by something").
3. Count. If most are cross-cutting, your decision is already made, and the rest of this walkthrough is about confirming you understand why.
4. Write down the rules you need enforced: which references must exist, which values must be unique, which pairs of writes must succeed or fail together. This is your enforcement list.
5. For each item on the enforcement list, ask whether you are prepared to guarantee it yourself in application code forever, on every path. Be honest.
6. Take your hardest cross-cutting question and brief AI to show you — in plain language, not code — how it would be answered in a relational database and in a document store. Ask for the number of round trips and where the computation happens in each.
7. Now ask what each design does when a dish is renamed, assuming the name is embedded in historical records. Note which design makes that a one-row update.
8. Identify any genuinely variable data in your product — a third-party payload, a free-form preferences blob. Ask how a `jsonb` column inside a relational table would handle it, and what constraints you would keep around it.
9. Look at where a key-value store would help you later: sessions, caches, rate-limit counters. Note those as M05's territory rather than today's decision.
10. Write the decision in your scope document in one paragraph: the database you chose, the two query shapes that decided it, and the one condition that would make you add a second store.

## Talk to AI
```prompt
CONTEXT:      Three-person student team, four months, free tiers only. No
              production experience. Users on budget Android phones in
              India; money is involved and stored as integer paise.
              Product: [one sentence]

              THE FIVE QUESTIONS OUR PRODUCT MUST ANSWER ABOUT ITS DATA:
              [list five real sentences, e.g. "how much did we take
              yesterday", "which dishes sold out most often this month"]

              RULES WE NEED ENFORCED:
              [list them, e.g. "an order must belong to a real user",
              "one account per phone number", "a payment and an order
              status must update together or not at all"]

              Expected data volume at demo day: [honest numbers — users,
              orders per day]

GOAL:         Choose the database type for this product and understand
              exactly what the choice costs me.

REQUIREMENTS: 1. Classify each of my five questions as single-record or
                 cross-cutting, and say which database type serves it well
              2. Take my hardest cross-cutting question and describe, in
                 plain language, how it is answered in a relational
                 database and in a document store — including how many
                 round trips and where the computation happens
              3. For each rule on my enforcement list, say whether the
                 database can enforce it or whether my application code
                 must, for each option
              4. Describe what a schema change looks like in each option,
                 six months in, with existing data
              5. Tell me whether any part of my data is variable enough to
                 justify a JSON column inside a relational table, and what
                 constraints should surround it
              6. Tell me honestly which option AI tools generate more
                 reliable code for, and why that matters for my team

CONSTRAINTS:  Free tier only. One database unless you can justify a second
              by naming something the first genuinely cannot do. Must
              support transactions across the writes in my rules list.

NON-GOALS:    Do not design my schema. Do not write code. Do not recommend
              a paid managed service. Do not optimise for a scale I have
              not described.

ACCEPTANCE:   - Your recommendation quotes at least two of my five
                questions
              - You state what my chosen option makes harder, not only
                what it makes easier
              - Every rule on my enforcement list is marked as
                database-enforced or application-enforced
              - You name the single condition that would make me add a
                second store later

ASK:          Give me the two options side by side with the costs of each
              against my actual questions, then your recommendation and the
              one question of mine that decided it. No code.
```

## Stuck? Ask AI
```prompt
Explain the difference between embedding data inside a document and referencing it by id, using a canteen order as the example. Then give me five pieces of information from a food-ordering app and ask me which should be embedded and which referenced, without telling me the answers first.
```

```prompt
People say document databases are schemaless. Explain where the schema actually went, and give me three concrete problems that appear six months later because of it. Use an example where two documents written in different months have different shapes.
```

```prompt
I need to store a webhook payload from a payment provider that has fields I cannot predict. Explain how I would do that inside a relational database using a JSON column, what I should pull out into real columns, and what constraints I should keep around it.
```

## Quiz

### Q1 · L2
Which NoSQL family is the right tool for sessions, caches and rate-limit counters?
- (a) Document store
- (b) Key-value store ✅
- (c) Wide-column store
- (d) Graph database
> Those are all "give me a key, get a value, very fast" problems with no cross-cutting queries. This is the role a key-value store plays alongside Postgres in M05, not instead of it.

### Q2 · L3
A team stores orders as documents with the dish name copied inside each one. A dish is renamed. What happens?
- (a) All documents update automatically when the dish document changes
- (b) Thousands of documents keep the old name, so the dish appears under two names in reports ✅
- (c) The rename is refused because documents reference it
- (d) Nothing — document stores resolve references at read time
> Embedding is copying. It is correct when you mean "what was sold at the time" and wrong when you mean "the dish's current name", and there is no atomic way to fix 4,000 copies.

### Q3 · L4
"Schemaless" is often described as avoiding migrations. What is the honest version of that claim?
- (a) It is true — no structure means nothing to migrate
- (b) The schema moved into application code, where it is implicit and unenforced ✅
- (c) It only applies to key-value stores
- (d) Migrations still happen but run automatically in the background
> Every screen still depends on a structure. You have swapped a reviewable migration for defensive checks in every reader and documents of three different shapes written in three different months.

### Q4 · L4
Your product must guarantee that marking an order paid and recording the payment either both happen or neither does. What does that requirement tell you?
- (a) Nothing about the database choice; it is an application concern
- (b) You need a database with transactions across those writes — a relational one ✅
- (c) You need two databases so each write is isolated
- (d) You need a message queue instead of a database
> A pair of writes that must succeed or fail together is exactly what a transaction provides, and it is chapter 11's subject. Anything with money needs it.

### Q5 · L5
A design for a three-person student project proposes Postgres for users and orders, a document store for menu content, and a key-value store for sessions. What is the strongest objection?
- (a) Key-value stores cannot store sessions reliably
- (b) Three stores means three sets of credentials, backups and failure modes, with no transaction spanning them ✅
- (c) Menu content is too structured for a document store
- (d) Postgres cannot handle users and orders at the same time
> Operational work multiplies per store and a write that must happen in two of them can half-fail. A `jsonb` column plus a cache added later in M05 gets the same benefits for a fraction of the cost.

## Go deeper
- [PostgreSQL docs: JSON types](https://www.postgresql.org/docs/current/datatype-json.html)
- [MongoDB manual: data modelling, embedding versus referencing](https://www.mongodb.com/docs/manual/data-modeling/)
