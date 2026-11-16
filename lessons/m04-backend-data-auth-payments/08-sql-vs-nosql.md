---
id: m04-08-sql-vs-nosql
module: m04-backend-data-auth-payments
slug: sql-vs-nosql
title: "SQL vs NoSQL, and When to Use Each"
order: 8
duration_min: 25
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

A team picks a document database for the canteen app because a blog post said it is faster and more flexible. For three weeks it is lovely: no tables, no migrations, just save an object and move on.

Then the product manager asks a reasonable question: "which dishes made the most money last month, and which students ordered them more than twice?"

In a relational database that is one query with two joins. Here, orders are documents with the dish details copied inside each one. So they fetch every order of the month into the server, loop over them in code, group by dish, count per student, and sort. It works on 300 orders and takes nine seconds on 30,000. Then they notice that a dish renamed in week two appears under two names, because the name was copied into 4,000 documents and only new ones got the new spelling.

The mirror-image failure also happens. Another team puts their chat messages in a relational table, one row per message, and at 2 million rows their "last 50 messages in this room" query is slow because the shape of the data fights the shape of the question.

Neither database was wrong. Both teams chose before knowing what they would ask.

### The idea

**Pick your database from the questions you need to answer and the rules you need enforced, not from the word on the front of it.**

A **relational (SQL) database** stores rows in tables with a fixed, declared structure. It enforces relationships and constraints, and it can combine data from several tables in one query using joins. Postgres and MySQL are the names you will meet.

**NoSQL** is not one thing. It is four different families that happen to share "not relational":

- **Document stores** keep JSON-like documents. Good when one document is exactly one screen's worth of data.
- **Key-value stores** are a giant dictionary: give a key, get a value, extremely fast. Good for caches, sessions and rate-limit counters.
- **Wide-column stores** handle enormous write volumes across many machines. Good for event streams and time-series.
- **Graph databases** make "friends of friends of friends" cheap. Good for networks and recommendations.

For a product with users, orders, money and relationships, the honest default is relational — and the rest of this module assumes one.

### The analogy

The canteen's ruled register is a relational database. Every line has the same columns in the same order. A student id that does not exist is refused by the clerk. At the end of the month the manager can total any column, cross-reference two registers, and get an answer.

A document store is a shoebox of order slips. Each slip is complete in itself — student name, dishes, prices, note — so finding one slip and reading everything about that order is instant. But "which dish earned the most in September" means pulling out 4,000 slips and counting by hand, and the dish names on them are whatever was written that day.

A key-value store is the token board: number 41 maps to "ready". Lightning fast at that one question, useless for any other.

**Where the analogy breaks:** real shoeboxes do not silently accept a slip with the quantity written in the price box. Document stores do. The flexibility that makes them pleasant in week one is the same property that lets three different shapes of order accumulate by week six, and you will not notice until a screen renders blank for one of them.

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

Still the "data" box from M00, and the choice colours everything after it. Chapter 9's joins and query plans assume relational. Chapter 11's transactions and migrations assume relational. Chapter 14's row-level security is a relational feature. Chapter 17's payment reconciliation needs the guarantees of a relational transaction.

That is not an accident of the syllabus — it is because a product with money and permissions needs enforcement, and enforcement is what relational databases sell.

Where NoSQL legitimately appears in your project, it appears *alongside* Postgres rather than instead of it. In M05 you will put a key-value store in front of the database as a cache and use it for rate-limit counters. In M10, RAG needs vector storage, which is a specialised store for embeddings — and modern Postgres can do that too, which is often the right call for a team of three. In M13, event streams at scale meet wide-column stores and Kafka.

### Choosing, by question shape

Ask three questions about your product.

**Do my queries cut across many records?** "Revenue per dish", "students who ordered twice", "orders awaiting payment older than ten minutes". Cross-cutting questions are what joins and indexes are for. If your product's value is in aggregates and reports, relational, without further debate.

**Does one record equal one screen?** If the thing you read is always "this one order, complete" and you never ask across them, a document store fits. Note how rare this is in practice: the moment anyone asks for a dashboard, it stops being true.

**Do I need the database to enforce rules?** Foreign keys that refuse orphans, unique constraints that refuse duplicate accounts, transactions that make two writes succeed or fail together. If the answer is yes — and for anything with money it is — relational.

There is a fourth consideration that matters more than any of the three for a student team: **what will AI generate correctly?** There is far more good relational material in the training data than anything else, and every tutorial, every hosted free tier, and every tool in this program assumes it. That is a legitimate engineering reason, not a cop-out.

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

**"Schemaless" means the schema moved into your code, where nobody can see it.** The structure still exists — every screen depends on it — but it is now implicit, unenforced, and different in documents written in different weeks. You have not avoided migrations; you have replaced them with defensive checks scattered through the application and with rows nobody can explain.

**Copying data into documents is a decision about history.** Embedding the dish name and price inside an order is correct if you mean "what was sold and at what price" — the same reasoning as chapter 6's `unit_price_paise`. It is wrong if you mean "the dish's current name", because you now have 4,000 copies to update and no way to do it atomically.

**Flexibility is not the same as speed.** Document stores are fast at the read they are shaped for and often slower than Postgres at everything else, because the work that a join would do moves into your application and over the network.

**Postgres does JSON too, and this resolves most arguments.** A `jsonb` column holds genuinely variable data — a third-party webhook payload, a free-form set of preferences — inside a relational table with real constraints around it. You very rarely need a second database to store some flexible fields.

**Mixing two databases triples your operational work.** Two sets of credentials, two backups, two failure modes, and no transaction spanning both — so a write that must happen in both can half-fail. On a three-person team this is not a trade worth making before you have users.

**Beware the benchmark argument.** "Document stores handle a million writes a second" is true and irrelevant at 400 students. Choose for your actual scale; M05 and M13 are where scale becomes the question.

### Verify it

Five checks on a database recommendation.

1. **Did it ask what questions you need to answer?** A recommendation made before anyone listed the reports and screens is a guess. Restate your top five queries and ask again.
2. **Name your hardest cross-cutting query and ask how it is served.** "Revenue per dish last month, by student, excluding cancelled orders." If the answer involves fetching lots of records and looping in application code, you have found the cost.
3. **Ask which rules the database will enforce.** Foreign keys, uniqueness, and two writes succeeding or failing together. If the answer is "the application handles that", understand that you have taken on the enforcement yourself, forever, on every path including the ones AI writes next month.
4. **Ask what a schema change looks like.** In Postgres it is a migration you can review. In a document store it is "new documents have the field, old ones do not", which means defensive code in every reader until the end of time.
5. **Count the databases in the design.** More than one needs a written justification per database, naming what cannot be done in the first one.

Red flags: a recommendation containing the words "scalable" and "flexible" and no query from your product; a document design that embeds the same mutable fact in thousands of documents; a proposal to use a key-value store as the system of record for orders; two databases in a design with no users yet; "we can always migrate later", which is true and costs a month; and any answer that never mentions what happens to your money data when two writes must succeed together.

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
