---
id: m13-12-data-engineering-for-products
module: m13-production-engineering
slug: data-engineering-for-products
title: "Data Engineering for Products: Pipelines, Warehouses, ELT and BI"
order: 12
duration_min: 15
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

Nothing in the Sprint teaches data engineering. M04 taught the database your product writes to, M05 its connection limits and replicas, M07 how to instrument users. None of that answers the question a founder asks.

A mentor asks: "how many students who ordered once came back within seven days?" Somebody runs a forty-line query against the production database at 1:05 p.m. It scans the whole orders table and the app slows through the lunch rush. The answer, 23%, cannot be defended, because "came back" was never defined. Worse and more confident: a nightly job runs twice after a retry, revenue doubles for that day, and nobody questions a slightly too good number.

**Document ingestion, PDF extraction and data quality for AI features are M10's subject, not this chapter.** This is your product's own operational data.

### The idea

**Operational databases and analytical questions want opposite designs, so you copy the data somewhere shaped for questions — raw first, transformed afterwards, with every metric defined exactly once.**

Your product's database is **OLTP**: many tiny reads and writes. An analytical question is **OLAP**: one enormous read across every row. Running the second against the first takes down your app.

**ELT** rather than ETL is the modern order, for a practical reason. Extract from the source, **Load** it raw and unchanged, then **Transform** with SQL inside the warehouse. Because the raw layer is kept, a changed definition — "active user now means two orders" — is re-derived over data you still have. With ETL the original is gone.

### The analogy

The canteen keeps two books. The **daily register** takes every order as it happens. It is the truth, and useless for "which dish sells best in the monsoon", because that means reading nine months of pages while the cashier is still writing in it.

The **month-end summary book** is copied over each night and shaped for questions: dishes down the side, days across the top. It is never the truth and always a few hours old, and that is fine. The clerk writes "copied up to Tuesday 11 p.m.": the **watermark**.

**Where the analogy breaks:** the clerk notices they already copied Tuesday. A pipeline has no memory unless you give it one, so a job that runs twice copies Tuesday twice and nothing looks wrong.

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

- The extract is a read, and it comes from M05 chapter 6's replica at a quiet hour, never the primary.
- Orchestration is M04 chapter 16's cron and queues, with chapter 2's idempotency applied to a table instead of a message.
- Transformations live in git, reviewed per M01 and tested per chapter 6; chapter 9's backups must include the raw layer.

Free paths are good: a local single-file analytical database, a cloud warehouse sandbox tier with a monthly allowance and no card, or a second small managed Postgres.

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

**Never point an analytics query at the production primary.** A replica is acceptable, a warehouse is correct.

**Make every load idempotent, which usually means delete-then-insert the window.** Appending "yesterday's rows" doubles them on a second run.

**Keep the raw layer forever and read-only** — that is what makes a changed definition cheap. Fix bad values in a transformation, never in a raw table.

**Define each metric exactly once, in git.** "Active user" computed in the dashboard, a spreadsheet and a notebook gives three numbers and an argument.

**Show freshness on every number.** A pipeline that failed at 2 a.m. leaves yesterday's figures looking like today's, so a break must show on the dashboard.

**Expect schema drift and fail loudly.** A renamed column makes a transform produce nulls, so assert row counts and non-null rates in your chapter 6 suite. Keep five tiles rather than forty, and no personal data unless a question needs it.

### Verify it

1. **Ask which database the extract reads from.** The production primary stops the conversation; a replica at a quiet hour is the minimum.
2. **Run the pipeline twice and compare a total.** If revenue changed, it is not idempotent.
3. **Ask for the watermark and where it is stored.** "It loads yesterday" is not a watermark; a position that survives a crash is.
4. **Demand the metric definition as SQL, by name,** then check nothing else computes it differently.
5. **Skip a run deliberately** and confirm the dashboard shows the staleness.

Red flags: analytics against the production primary; no watermark; appending rows rather than replacing a window; an edited raw layer; two definitions of "active user"; transformations living only in a BI tool.

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
