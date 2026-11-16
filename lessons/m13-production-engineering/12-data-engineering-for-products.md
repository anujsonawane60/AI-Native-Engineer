---
id: m13-12-data-engineering-for-products
module: m13-production-engineering
slug: data-engineering-for-products
title: "Data Engineering for Products: Pipelines, Warehouses, ELT and BI"
order: 12
duration_min: 30
is_free: false
mission: false
volatility: medium
last_reviewed: 2026-11-16
---

## Key learnings
- Say why an analytics question must never be asked of your production database
- Load raw first and transform in the warehouse, so a changed definition is re-derivable
- Make a pipeline idempotent with a watermark, so a second run cannot double your revenue
- Define each metric once, in version control, and show freshness next to every number
- Catch a dashboard whose numbers nobody can trace back to a definition

## Lesson

### The pain

This chapter exists because a reviewer of this syllabus argued, correctly, that data engineering should not be cut. Nothing in the Sprint teaches it. M04 taught you the database your product writes to, M05 chapter 6 taught you its connection limit and its replicas, and M07's analytics chapters taught you to instrument what users do. None of that answers the question a founder actually asks, which is the first failure of this chapter.

A mentor asks: "how many students who ordered once came back within seven days?" Somebody writes a forty-line query and runs it against the production database at 1:05 p.m. It scans the whole orders table, holds three connections out of a free-tier limit of thirty, and for ninety seconds the app is slow for every student in the lunch rush. Then the answer comes back — 23% — and nobody can defend it, because "came back" was never defined. Does a cancelled order count? An order placed by the same phone number on a different account? The number goes in a slide deck anyway.

Second failure, which is worse because it is confident. The team builds a nightly job that loads "yesterday's orders" into a reporting table. One night the job fails halfway, retries, and runs twice. That day's revenue is now double. Nobody notices, because the only evidence is a number being slightly too good, and a slightly too good number is exactly the kind nobody questions. Six weeks later the monthly revenue chart goes to a potential investor with one inflated bar in it.

Note the boundary: **document ingestion, PDF extraction and data quality for AI features are M10's subject, not this chapter.** This chapter is about your product's own operational data and the questions your team needs answered from it.

### The idea

**Operational databases and analytical questions want opposite designs, so you copy the data somewhere shaped for questions — raw first, transformed afterwards, with every metric defined exactly once.**

Your product's database is **OLTP**: many tiny reads and writes, optimised for one order at a time. An analytical question is **OLAP**: one enormous read across every row, optimised for aggregation. Running the second against the first is how you take down your own app.

**ELT** rather than ETL is the modern order and the reason is practical. Extract from the source, **Load** it raw and unchanged into the warehouse, then **Transform** with SQL inside the warehouse. Because the raw layer is kept, a changed definition — "active user now means two orders, not one" — is re-derived by re-running a query over data you still have. With ETL you transform before loading, the original is gone, and a changed definition means a change you cannot apply backwards.

### The analogy

The canteen keeps two books.

The **daily register** is written in all day: every order, in order, as it happens. It is the truth. It is also almost useless for answering "which dish sells best in the monsoon", because that means reading nine months of pages while the cashier is still trying to write in it.

The **month-end summary book** is copied over each night from the register and shaped for questions: dishes down the side, days across the top, totals at the edges. The manager reads it in thirty seconds. It is never the truth, it is always a few hours old, and that is fine — nobody charges a student from the summary book.

Two details matter. The clerk copying each night writes "copied up to Tuesday 11 p.m." at the bottom of the page, so tomorrow they know where to resume. That is the **watermark**. And the register pages are kept, which is why when the manager changes their mind about what counts as a "repeat customer", the clerk can recompute it rather than shrug.

**Where the analogy breaks:** the clerk notices they already copied Tuesday. A pipeline has no memory unless you give it one, so a job that ran twice copies Tuesday twice, silently, and the summary is wrong in a way nobody can see by looking at it. All the discipline in this chapter exists because the clerk's common sense has to be written down as code.

### Picture it

```
  PRODUCTION (OLTP)         WAREHOUSE (OLAP)          BI LAYER
 ┌───────────────┐        ┌────────────────────┐    ┌─────────┐
 │ orders        │EXTRACT │ raw.orders         │    │ 5 tiles │
 │ students      │ since ▶│  append only,      │    │ each    │
 │ dishes        │ 23:00  │  never edited      │    │ with an │
 └───────────────┘ (water-└─────────┬──────────┘    │ "as of" │
  analytics reads   mark)           │ TRANSFORM     │ time    │
  here ✗ never                      │ (SQL, in git) └─────────┘
                                    ▼                    ▲
                           ┌────────────────────┐        │
                           │ mart.active_users  │────────┘
                           │ ONE definition     │
                           └────────────────────┘
```

Data is extracted from the product database since a recorded watermark, landed raw and never edited, transformed by version-controlled SQL into a small number of defined tables, and read by a dashboard that always shows how old it is.

### Where it lives

This sits beside your product rather than inside it. The extract is a read, and it should come from a replica from M05 chapter 6 rather than the primary — and even then at a quiet hour. The orchestration is M04 chapter 16's cron and queues, with the idempotency rule from chapter 2 of this module applied to a whole table rather than a single message. The transformations live in git and get reviewed like code, per M01, and tested like code, per chapter 6 — a transformation with no test is a number nobody can trust. Chapter 9's backup list must include the warehouse's raw layer, because that is the thing you cannot regenerate once the source has moved on.

Forward: M16 is where this data becomes growth work — funnels, retention curves and the experiments that change them — and that module assumes a defined metric exists. M18's handover asks what your product's numbers are and where they come from, and "a query someone ran once" is not an answer.

Free paths are genuinely good here. A local single-file analytical database runs on your laptop for nothing and will handle years of a student startup's data. One major cloud warehouse offers a free sandbox with a monthly query allowance and no card. A second small managed Postgres, separate from production, is a respectable warehouse at your scale. For the BI layer, free tiers exist, and a committed notebook is enough to start.

### Words to own

| Term | Plain meaning | Example |
|---|---|---|
| OLTP | Your product's database: many small reads and writes | Orders |
| OLAP | Shaped for big aggregate questions | The warehouse |
| Warehouse | A separate database for analysis | Free sandbox tier |
| Extract | Read rows out of the source | Since the watermark |
| Load | Write them in, unchanged | `raw.orders` |
| Transform | Reshape with SQL, inside the warehouse | `mart.active_users` |
| ELT | Load raw first, transform after | Re-derivable |
| Raw layer | Untouched copy, append only | Never edited |
| Mart | The defined, query-ready tables | What the dashboard reads |
| Watermark | Where the last run stopped | "Up to 23:00 Tuesday" |
| Incremental load | Only the new rows, not everything | Cheap and fast |
| Idempotent pipeline | Running twice gives the same result | Delete-then-insert the window |
| Schema drift | The source adds or renames a column | Breaks a transform |
| Semantic layer | One definition per metric, in one place | "Active user" |
| Freshness | How old the numbers on screen are | "As of 06:12 today" |

### Tradeoffs and gotchas

**Never point an analytics query at the production primary.** It is the single rule of this chapter. A read replica is acceptable, a warehouse is correct, and the primary is how you cause an outage during lunch. The giveaway is a query with no `limit` and a date range of months.

**Make every load idempotent, which usually means delete-then-insert the window.** A pipeline that appends "yesterday's rows" doubles them when it runs twice. A pipeline that deletes everything for yesterday and then inserts yesterday is safe to run five times. This is chapter 2's idempotency at table scale, and the failure is silent in both cases — which is why you test it by running the job twice on purpose and comparing a total.

**Keep the raw layer forever and treat it as read-only.** It is what makes a changed definition cheap. Never edit a raw table to fix a value; fix it in a transformation, where the fix is visible, reviewable and reversible.

**Define each metric exactly once and put the definition in git.** "Active user" computed in the dashboard, again in a spreadsheet, and again in someone's notebook gives you three different numbers and an argument. One SQL definition, one name, one owner. When someone disagrees with the number, you change the definition rather than the query — and everything downstream updates.

**Show freshness on every number, always.** A dashboard tile with no "as of" timestamp is dangerous, because a pipeline that failed at 2 a.m. leaves yesterday's figures looking exactly like today's. Make a broken pipeline visible on the dashboard itself rather than only in a log nobody reads.

**Expect schema drift and fail loudly.** Someone adds a column, renames one, or changes a type in the product database, and a transform silently starts producing nulls — or worse, a column of zeros that gets averaged. A transformation test that asserts row counts and non-null rates is the cheap defence, and it belongs in your chapter 6 suite.

**Five numbers, not forty.** A founder reads a five-tile dashboard and ignores a forty-tile one. Pick the five that would change a decision this month and delete the rest. A dashboard nobody opens is a pipeline maintained for nothing.

**Never put personal data in the warehouse unless a question needs it.** Phone numbers and names rarely belong in an analytics copy; a hashed identifier answers "how many distinct students" without storing anyone. This is chapter 11's data-minimisation argument applied to a second copy of your data that is easy to forget exists.

### Verify it

1. **Ask which database the extract reads from.** If the answer is the production primary, stop there. A replica at a quiet hour is the minimum.
2. **Run the pipeline twice and compare a total.** If revenue changed, it is not idempotent. This is a two-minute test that catches the most expensive bug in the chapter.
3. **Ask for the watermark and where it is stored.** "It loads yesterday" is not a watermark; a recorded position that survives a crash is.
4. **Demand the metric definition as SQL, by name.** Then check nothing else computes the same metric differently. Two definitions of one metric is a guaranteed future argument.
5. **Check the dashboard shows freshness,** and then deliberately skip a pipeline run and confirm the dashboard makes that visible rather than showing stale numbers confidently.
6. **Ask what happens when the source adds a column.** The correct answer is a loud failure or a recorded warning, never "the transform ignores it", which is how a column of zeros gets averaged into a slide.
7. **Look for personal data in the warehouse** and ask which question needs it. Most of it is there because the extract copied everything.

Red flags: analytics queries against the production primary; a pipeline with no watermark; appending rows rather than replacing a window; a raw layer that gets edited; two places defining "active user"; a dashboard tile with no "as of" time; transformations that live only in a BI tool rather than in git; a nightly job whose failure is visible only in a log; forty tiles nobody reads; and personal data copied into the warehouse because it was easier than selecting columns.

## Walkthrough
1. Write down the five questions your team actually needs answered this month — the ones that would change a decision. Not twenty. For each, write who would act on the answer.
2. For the hardest of the five, write the metric definition in plain English first, including the edge cases: does a cancelled order count, does a refunded one, what about two accounts with one phone number. This sentence is the real work.
3. Create a free warehouse: a local single-file analytical database, a free cloud sandbox tier, or a second small managed Postgres that is not production. Confirm it is separate from your product's database.
4. Build the extract as a read from a replica, or at a genuinely quiet hour, and load the rows unchanged into a `raw` schema. Change nothing on the way in.
5. Store a watermark — the timestamp or id your last run reached — somewhere that survives a crash, and make the next run resume from it.
6. Make the load idempotent: delete everything in the window you are about to load, then insert it. Then run the job twice on purpose and compare a revenue total before and after. The two numbers must match.
7. Write your metric as one SQL transformation in a git-tracked file, producing a small `mart` table. Give it a name and an owner in a comment.
8. Add two tests to that transformation: a row-count assertion and a non-null-rate assertion on the columns you care about. Wire them into your pipeline from chapter 6 so a broken transform fails rather than publishing nulls.
9. Build a BI layer with exactly five tiles, each showing its "as of" timestamp. Then skip a pipeline run deliberately and confirm the dashboard makes the staleness obvious.
10. Simulate schema drift: rename or add a column in a copy of your source and run the pipeline. Confirm it fails loudly rather than producing silent nulls, and fix it if it does not.
11. Finally, audit what personal data your extract copied. Remove any field no question needs, replace identifiers with a hash where you only need distinct counts, and add the warehouse's raw layer to chapter 9's backup list.

## Talk to AI
```prompt
CONTEXT:      I am a student on an AI-native engineering program. I direct AI
              and verify its output; I do not write code by hand. My team of
              three runs [product in two sentences] with [number] real users
              in India. Our product database is [name and plan] with a
              connection limit of [number], and it has [a read replica / no
              replica]. Our main tables are: [list them with roughly how many
              rows each]. We collect these personal fields: [list them]. We
              have no warehouse and no dashboard. Today, when someone wants a
              number, they run a query against production. Budget: free tiers
              only.

              The five questions we actually need answered are:
              1. [e.g. how many students who ordered once came back in 7 days]
              2. [...]
              3. [...]
              4. [...]
              5. [...]

GOAL:         A working ELT pipeline into a free warehouse, one metric defined
              exactly once in version control, and a five-tile dashboard that
              shows how old its numbers are.

REQUIREMENTS: 1. Explain why running my hardest question against my production
                 primary is dangerous, in terms of my actual connection limit
                 and row counts
              2. Recommend a FREE warehouse for my data volume, with the
                 honest limits of each option and what happens when I outgrow
                 it
              3. Design the extract: which database it reads from, at what
                 hour, and how the watermark is stored so a crash does not
                 lose or duplicate rows
              4. Make the load idempotent. Show me the delete-then-insert
                 window approach and give me the exact two-run test that
                 proves it - run the job twice and compare which total
              5. Explain ELT versus ETL using MY hardest question, and show me
                 concretely what changes if I later redefine the metric. Tell
                 me what I lose if I do not keep a raw layer
              6. Turn my plain-English metric definition into ONE SQL
                 transformation in a git-tracked file, handling the edge cases
                 I listed, with a name and an owner
              7. Give me two tests for that transformation - a row-count
                 assertion and a non-null-rate assertion - and tell me where
                 they run
              8. Tell me what happens when someone adds or renames a column in
                 my product database, and how to make the pipeline fail LOUDLY
                 instead of producing nulls
              9. Design a five-tile dashboard with a freshness timestamp on
                 each tile, and tell me how a failed overnight run becomes
                 visible on the dashboard itself
             10. Looking at my personal fields, tell me which must NOT be
                 copied into the warehouse, and where a hashed identifier
                 answers the question instead

CONSTRAINTS:  Free tiers only, and say clearly where free is not possible with
              a rupee figure. Plain English for a second-language reader. The
              extract must never read my production primary during the day.
              Label every command READ-ONLY or CHANGES-THINGS. Respect India's
              data protection rules and minimise what is copied. Do not change
              my product's database schema.

NON-GOALS:    Do not set up a streaming pipeline. Do not recommend a paid
              warehouse, a paid BI product, or a managed orchestration
              service. Do not build all five metrics - one, properly. Do not
              cover document ingestion or PDF extraction; that is a different
              module. Do not give me forty dashboard tiles.

ACCEPTANCE:   - My warehouse is separate from production and cost me nothing
              - My extract reads a replica or runs at a quiet hour
              - A watermark survives a crash, and I know where it is stored
              - Running the pipeline twice leaves my revenue total unchanged
              - My raw layer is append-only and I have never edited it
              - My metric has exactly one SQL definition, in git, with an owner
              - Renaming a source column makes the pipeline fail loudly
              - Every dashboard tile shows an "as of" timestamp
              - A skipped run is visible on the dashboard, not just in a log
              - No personal field is in the warehouse unless a question needs it

ASK:          Start with my hardest question and my plain-English definition.
              Argue with my definition - tell me which edge cases I have not
              decided - before you write any SQL.
```

## Stuck? Ask AI
```prompt
Explain OLTP and OLAP as two opposite designs, using one concrete query for each from a food-ordering app. Then explain exactly what happens, step by step, when I run the OLAP query against my production database with a connection limit of 30 during a lunch rush: what the database does, what my app's users experience, and which M05 concept it breaks. Finish by telling me the three cheapest places I could have run that query instead.
```

```prompt
Explain ELT versus ETL with a worked example: I define "active user" as someone who ordered in the last 7 days, and three months later I change it to someone who ordered twice. Walk me through exactly what I can and cannot recompute under each approach, and what data I would need to still have. Then tell me the one rule about my raw layer that makes the ELT version possible, and the one thing I must never do to it.
```

```prompt
My nightly pipeline appends yesterday's orders to a reporting table. Explain precisely what happens if it runs twice, and why I would probably not notice. Then show me the delete-then-insert window pattern that makes it idempotent, tell me what the watermark must store for it to be crash-safe, and give me the exact two-run test that would prove it works - including which single total I should compare before and after.
```

## Quiz

### Q1 · L3
Why must an analytics question not be asked of your production primary?
- (a) Production databases cannot run aggregate queries
- (b) One huge read holds connections and scans every row, slowing the app for real users ✅
- (c) The answer would be stale because of replication lag
- (d) Analytics queries require a different SQL dialect
> OLTP and OLAP want opposite designs. The giveaway is a query with no limit and a date range of months, running against a free tier with a thirty-connection ceiling during lunch.

### Q2 · L4
Your nightly job appends yesterday's orders and one night it runs twice. What happens, and what is the fix?
- (a) The second run is rejected as a duplicate key violation
- (b) Revenue for that day doubles silently; delete the window then insert it, so repeat runs are safe ✅
- (c) Only the new rows since the first run are added, so nothing breaks
- (d) The watermark prevents the second run from doing anything
> A slightly too good number is the kind nobody questions. Delete-then-insert makes the load idempotent, and the two-run comparison test takes two minutes and catches the most expensive bug here.

### Q3 · L4
Three months after defining "active user" as one order in seven days, you change it to two orders. What makes that cheap?
- (a) ETL, because the data was already transformed on the way in
- (b) ELT with a kept raw layer, so you re-derive the metric from data you still have ✅
- (c) A larger warehouse instance to recompute faster
- (d) A second dashboard showing both definitions side by side
> This is the practical argument for load-then-transform. If you transformed before loading, the originals are gone and a redefinition cannot be applied backwards — which is why the raw layer is append-only and never edited.

### Q4 · L3
Your overnight pipeline failed and the dashboard shows yesterday's numbers. Why is a missing freshness timestamp dangerous?
- (a) It makes the dashboard load more slowly
- (b) Stale numbers look identical to current ones, so a broken pipeline reads as a quiet day ✅
- (c) Without it, the BI tool cannot cache tiles correctly
- (d) It prevents the metric definition from being traced
> A failure visible only in a log nobody reads is not visible. Every tile carries an "as of" time so the dashboard reports its own health, which is the same argument as chapter 2's consumer lag alert.

### Q5 · L5
Two people produce different answers for "monthly active users" and argue for forty minutes. What was missing, structurally?
- (a) A faster warehouse so both queries return the same day
- (b) One SQL definition of the metric, in git, with a name and an owner ✅
- (c) A shared spreadsheet both people edit
- (d) A larger raw layer covering more history
> A metric computed in three places is three metrics. With one definition, disagreement becomes a reviewable change to that definition and everything downstream updates — which is also what M16's growth work and M18's handover assume exists.

## Go deeper
- [dbt: what is ELT, and why transformations belong in version control](https://docs.getdbt.com/docs/introduction)
- [DuckDB: a free single-file analytical database you can run locally](https://duckdb.org/docs/)
