---
id: m04-09-joins-indexes-query-plans
module: m04-backend-data-auth-payments
slug: joins-indexes-query-plans
title: "Reading a Query: Joins, Indexes and Query Plans"
order: 9
duration_min: 15
is_free: false
mission: false
volatility: low
last_reviewed: 2026-11-16
---

## Key learnings
- Read a SQL query aloud and say which tables it touches and what it returns
- Explain what an index is, what it costs, and which columns deserve one
- Read a query plan well enough to spot a full table scan
- Recognise the N+1 query pattern inside your own server code
- Verify AI's "I optimised it" claim with a measurement instead of a sentence

## Lesson

### The pain

The canteen admin page lists today's orders with student names. With 40 orders it loads in 200 milliseconds; three weeks later, with 9,000, it takes eleven seconds — and nobody changed the code. The server runs one extra query per order for the student's name, so 1 query becomes 1 + 200, and each reads the whole `students` table from the top because no index exists on the column it filters by.

### The idea

**A join combines two tables in one query; an index lets the database find rows without reading all of them; a query plan is the database telling you, before you run anything, which it intends to use.**

You will read these queries rather than write them: say what a join returns, spot the scan in a plan, and spot the loop that should have been a join.

### The analogy

The canteen keeps a register of orders and one of students. A **join** is holding both open and matching roll numbers across them. An **index** is the alphabetical thumb-tab on the student register: without it, finding roll number 4471 means reading every line from page one, and the cost is writing every new student into the tab too. A **query plan** is her saying out loud what she is about to do.

**Where the analogy breaks:** she knows her registers; the database guesses from statistics that can be out of date, and sometimes scans a table because it believes it holds 40 rows when it holds 90,000.

### Picture it

```
 WITHOUT AN INDEX                WITH AN INDEX
 find orders for student 4471    find orders for student 4471

 orders table                    index on orders(student_id)
 ┌─────────────────┐             ┌──────────────────┐
 │ row 1   check   │             │ 4470 ──▶ row 812 │
 │ row 2   check   │             │ 4471 ──▶ row 203 │──┐
 │ row 3   check   │             │ 4471 ──▶ row 455 │──┤
 │   ...   9000    │             │ 4472 ──▶ row 119 │  │
 │ row 9000 check  │             └──────────────────┘  │
 └─────────────────┘                                   ▼
   SEQ SCAN                      INDEX SCAN     2 rows read
   9000 rows read                               directly
```

Without an index the database checks every row to find the matching ones; with an index it looks up the value and goes straight to the rows that match.

And the join that fixes the other half:

```
SELECT o.id, s.name, o.total_paise, o.placed_at
FROM orders o
JOIN students s ON s.id = o.student_id
WHERE o.placed_at >= '2026-11-16'
  AND o.status != 'cancelled'
ORDER BY o.placed_at DESC
LIMIT 50;
```

Read it as the database runs it: **FROM** and **JOIN** combine the rows, **WHERE** narrows them, **ORDER BY** and **LIMIT** take the newest fifty. That one `JOIN` replaces 50 queries for 50 names.

### Where it lives

A query travels down the arrow between M00's server and data boxes, so nothing in the client can fix a bad one. Chapter 6 gave you the tables; chapter 10 extends this to text search; M05 caches in front of all of it. **Fix the query first, then cache** — a cached bad query stays hidden until the cache misses during your demo.

### Words to own

| Term | Plain meaning | Example |
|---|---|---|
| Join | Combining rows from two tables in one query | orders + students |
| Inner join | Keep only rows matching on both sides | drops orphans silently |
| Left join | Keep all left rows; blanks where no match | order with no student |
| Index | A sorted shortcut for finding rows by value | on `orders.student_id` |
| Sequential scan | Reading every row in the table | the slow path |
| Query plan | The database's stated strategy | from `EXPLAIN` |
| N+1 query | One query for a list, then one per item | 1 + 200 |
| Pagination | Returning a page at a time | `LIMIT 50 OFFSET 0` |
| Composite index | One index across two or more columns | `(student_id, placed_at)` |
| Cardinality | How many distinct values a column has | `status` has 5 |

### Tradeoffs and gotchas

Every index is updated on every write, so add one only when a measurement showed a scan; a unique constraint is free.

| Column | Index it? |
|---|---|
| Foreign keys like `orders.student_id` — joins filter on these | Always |
| Columns a big table's `WHERE` narrows by | Usually |
| A filter plus a sort, as `(student_id, placed_at)` in that order | Composite |
| Booleans, three-value statuses, free text nobody filters | Rarely |

**N+1 queries are the most common performance bug in AI-generated backends, and they are invisible at small data sizes.** With 40 rows nobody notices; the tell is a database call inside a loop.

**Indexes do not fix a query that returns everything.** With no `LIMIT` you send 9,000 rows to a phone however fast the database found them.

### Verify it

`EXPLAIN` asks what the database plans to do; `EXPLAIN ANALYZE` runs it and reports what happened.

| What you see | What it means |
|---|---|
| `Seq Scan` on a big table | reading every row — usually a missing index |
| `Index Scan` | using an index, which is what you want |
| `rows=` | how many rows it expects at that step |
| `actual time=` | how long that step really took, in milliseconds |
| `Nested Loop` over many rows | a join done row by row; fine for few, bad for many |

Find the step with the largest `actual time`, then check whether it is a scan. Five checks on anything AI calls optimised.

1. **Ask for the plan, before and after, on realistic data** — the real `EXPLAIN ANALYZE` output, not an explanation.
2. **Hunt for loops containing queries.** A database call inside a `for` or `map` is an N+1.
3. **Check every foreign key has an index.** List the keys, list the indexes, compare.
4. **Check every list endpoint has a limit.** "Only today's orders" is fine until it is not.
5. **Re-measure with seeded data.** A fix that only works on an empty database is not a fix.

Red flags: "I added an index" with no named column; an index on a boolean or three-value status with no measurement; `SELECT *` in a list endpoint; an inner join where a left join was meant, so rows vanish instead of showing blanks; "this will scale" with no number; a fix never run beyond dev's handful of rows.

## Walkthrough
1. Find the slowest screen in your own product. Load it and time it with the browser's Network tab, noting the server time for the API call rather than the total page time.
2. Seed your scratch database with realistic volume — a few thousand rows in your busiest table. Ask AI for a seed script; this is one of the few times generated code needs no review beyond "does it insert plausible data".
3. Reload the screen and time it again. Write both numbers down. This gap is what you are about to diagnose.
4. Find the server code behind that screen and read it for one thing only: is there a database call inside a loop? If yes, write down the arithmetic — one query plus how many.
5. Run `EXPLAIN ANALYZE` on the main query in your database's query editor. Save the output. You now have a before measurement.
6. Read the plan for the four words from the lesson: `Seq Scan`, `Index Scan`, `rows=`, `actual time=`. Find the single step with the largest actual time and say out loud what it is doing.
7. List your tables' foreign keys and your existing indexes side by side. Mark every foreign key with no index.
8. Brief AI with the plan output, the loop you found, and your missing-index list. Ask for the fix as two separate changes — collapse the N+1 into a join, and add the specific indexes — so you can measure each one.
9. Apply one change. Re-run `EXPLAIN ANALYZE` and record the new actual time. Then apply the second and measure again. Two measurements, two attributions.
10. Confirm the screen still shows the same data. A join changed from left to inner will quietly drop rows, and a faster page with missing rows is a worse page.
11. Check every list endpoint for a limit and add pagination where it is missing. Then record in your decision log: the before and after times, which change caused which improvement, and the indexes you added with the query each one serves.

## Talk to AI
```prompt
CONTEXT:      I am diagnosing a slow page in a student project, and I want
              to learn to read the diagnosis rather than be handed a fix.
              Free-tier managed PostgreSQL. Users on slow 4G, so payload
              size matters too.
              The screen: [describe what it displays]
              Row counts in the relevant tables right now: [real numbers]
              Expected row counts by demo day: [honest numbers]

              THE SERVER CODE BEHIND IT:
              [paste the function that fetches the data]

              THE SCHEMA FOR THE TABLES INVOLVED, INCLUDING EXISTING
              INDEXES:
              [paste it]

              EXPLAIN ANALYZE OUTPUT FOR THE MAIN QUERY, RUN AGAINST
              REALISTIC SEEDED DATA:
              [paste it]

GOAL:         Understand exactly why this is slow, and fix it as separate
              measurable changes rather than one rewrite.

REQUIREMENTS: 1. Walk me through the EXPLAIN ANALYZE output line by line in
                 plain language: what each step does, how many rows it
                 touched, and how long it took
              2. Name the single step costing the most time and say why it
                 is expensive
              3. Tell me whether my server code contains an N+1 pattern,
                 and if so give me the arithmetic at my expected row counts
              4. Propose the fixes as a numbered list of INDEPENDENT changes
                 I can apply and measure one at a time — never as one
                 combined rewrite
              5. For each index you propose, name the exact columns in
                 order, the query it serves, and what it costs me on writes
              6. Tell me whether any of your changes could alter which rows
                 come back — especially any change between inner and left
                 join — and exactly how I would check

CONSTRAINTS:  Free-tier PostgreSQL. Do not add caching; that is a later
              module and I want the query itself correct first. Do not
              change my schema's columns. Keep the response payload small
              enough for slow 4G.

NON-GOALS:    No caching. No read replicas. No schema redesign. Do not
              combine the fixes. Do not touch any other endpoint.

ACCEPTANCE:   - I can re-run EXPLAIN ANALYZE after each change and see
                which one caused which improvement
              - Every proposed index names its columns in order and the
                query it serves
              - You state explicitly whether the result set can change
              - The endpoint returns a bounded number of rows

ASK:          Explain the plan output first and tell me what you think is
              wrong, before proposing any change. I want to understand the
              diagnosis before I see the prescription.
```

## Stuck? Ask AI
```prompt
Explain what an index is using the alphabetical thumb-tabs on a paper register as the analogy. Then explain the cost side honestly: what gets slower, and give me three columns in a food-ordering app where an index would be a waste, with the reason for each.
```

```prompt
Here is a function from my server: [paste it]. Tell me how many database queries this makes when it renders a page with 50 items, show me the arithmetic, and explain what this pattern is called. Do not rewrite the code yet.
```

```prompt
Here is EXPLAIN ANALYZE output I do not understand: [paste it]. Walk me through it line by line in plain English. Tell me which single step is costing the most time, how you can tell, and what the words Seq Scan and Nested Loop mean here.
```

## Quiz

### Q1 · L2
What does a `Seq Scan` on a large table in a query plan usually indicate?
- (a) The query is using an index efficiently
- (b) The database is reading every row, often because no useful index exists ✅
- (c) The table is locked by another query
- (d) The result set is being sorted in memory
> A sequential scan reads the whole table. On a small table that is fine and sometimes faster; on a large one filtered by a column, it is the signature of a missing index.

### Q2 · L3
Your server fetches 200 orders, then fetches the student name for each order separately. How many queries, and what is the fix?
- (a) 200 queries — add an index on students.id
- (b) 2 queries — nothing needs fixing
- (c) 201 queries — replace the loop with a join that fetches names in the same query ✅
- (d) 201 queries — add a cache in front of the students table
> This is the N+1 pattern. One join collapses it to a single query, and the tell in code review is a database call inside a loop.

### Q3 · L4
Which column is the best candidate for a new index on a large `orders` table?
- (a) `status`, which has five possible values
- (b) `is_paid`, a boolean
- (c) `student_id`, a foreign key you filter and join on constantly ✅
- (d) `note`, free text that is displayed but never filtered
> Foreign keys are the columns joins filter on, and most databases do not index them automatically. Low-cardinality columns like booleans rarely benefit from one on their own.

### Q4 · L4
AI tells you "I optimised the query, it should be much faster now." What is the correct next step?
- (a) Deploy it, since the explanation was convincing
- (b) Run EXPLAIN ANALYZE before and after on realistically seeded data and compare the actual times ✅
- (c) Add a cache so the query runs less often
- (d) Ask the AI to explain its reasoning in more detail
> "Should be" is reasoning, not evidence. Every query is fast on 40 rows, so the measurement only means something against realistic volume.

### Q5 · L5
A fix changes an inner join to a left join and the page gets faster. What must you check before accepting it?
- (a) Whether the index was used
- (b) Whether the same rows come back, since join type changes which rows appear ✅
- (c) Whether the query still has a LIMIT
- (d) Whether the database statistics are up to date
> An inner join silently drops rows with no match; a left join keeps them with blanks. A faster page showing different data is not an optimisation, it is a behaviour change.

## Go deeper
- [PostgreSQL docs: using EXPLAIN](https://www.postgresql.org/docs/current/using-explain.html)
- [Use The Index, Luke — a plain-language guide to SQL indexing](https://use-the-index-luke.com/)
