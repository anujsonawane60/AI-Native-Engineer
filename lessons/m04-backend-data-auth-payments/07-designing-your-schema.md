---
id: m04-07-designing-your-schema
module: m04-backend-data-auth-payments
slug: designing-your-schema
title: "Designing Your Schema"
order: 7
duration_min: 35
is_free: false
mission: true
volatility: low
last_reviewed: 2026-11-16
---

## Key learnings
- Derive tables from your PRD's user stories rather than from a prompt
- Turn product questions into schema decisions all three founders agree on
- Write the constraints that make your product's impossible states impossible
- Direct AI to a schema and review it against your own list, not its summary
- Verify a schema by attempting the writes it should refuse

## Lesson

### The pain

A team gives an AI one line: "design a database for a canteen ordering app". Eleven seconds later they have fourteen tables: `users`, `roles`, `permissions`, `restaurants`, `branches`, `cuisines`, `tags`, `coupons`, `loyalty_points`, `reviews`, `delivery_zones`, `riders`, `payment_methods`, `audit_logs`.

It looks impressive. It is a generic food-delivery schema from the training data, and it fits a company of 400 people. Their product is one canteen, no delivery, no riders, no branches, and a demo in eleven weeks.

Worse, the parts they actually need are wrong. There is no way to record that a dish ran out at 1:15 p.m. and came back at 1:40. There is nowhere to store that a student paid by UPI and the webhook has not confirmed it yet. The `price` column is a float. And the one question their whole product depends on — can a student order on behalf of a friend who is in class? — has no answer anywhere in fourteen tables.

They now spend three weeks building on a schema nobody designed, deleting tables they never used, and discovering in week nine that the thing they needed was never possible.

### The idea

**A schema is derived from your user stories, one at a time, and the hard part is the questions it forces you to answer about your product.**

You do not ask AI for a schema. You bring AI a list of the things your product must be able to record and the things it must make impossible, and have it propose tables that satisfy that list. Then you check its proposal against your list, not against its own summary of its work.

The method is four passes. Nouns become tables. Facts become columns. Relationships become keys. Rules become constraints.

### The analogy

The canteen manager does not buy a register from a shop that sells "restaurant registers". She sits down and asks what she needs to be able to answer at the end of the day.

How much did we take? Which dishes ran out, and when? Who has an unpaid tab? Which student complained about the dal, and on which day? Each question becomes a column somewhere, and the register gets exactly the lines it needs.

A register bought off the shelf has a column for "table number" and no column for "token number", because it was designed for a sit-down restaurant. She would spend six months writing token numbers in the margin.

**Where the analogy breaks:** the manager can add a column with a pen in three seconds. You cannot. Once a thousand rows exist, adding a required column means a migration, a default value for the old rows, and a decision about what that default means — which is chapter 11. The register analogy makes schema change sound free, and it is the most expensive kind of change you will make.

### Picture it

```
  PRD user stories                    schema
 ┌─────────────────────────┐
 │ "A student places an    │   nouns ──▶ students, orders,
 │  order for 2 samosas"   │             menu_items,
 │                         │             order_items
 │ "Staff mark a dish      │   facts ──▶ qty, unit_price_paise,
 │  sold out"              │             is_available, placed_at
 │                         │
 │ "A student sees what    │   links ──▶ orders.student_id
 │  they paid in September"│             order_items.order_id
 │                         │
 │ "Nobody can order a     │   rules ──▶ FK must exist,
 │  sold-out dish"         │             qty between 1 and 10,
 │                         │             CHECK is_available
 └─────────────────────────┘
        ▲                                      │
        └──── a story with nowhere to live ◀───┘
              means the schema is incomplete
```

Each user story contributes nouns, facts, links and rules; a story that cannot be recorded by the resulting tables is the signal that the design is not finished.

### Where it lives

This chapter closes a loop that opened in M01. There you wrote a PRD and a scope document, and you were told the schema is a product decision rather than Tech's call. Here is the proof: almost every interesting schema question is a product question.

Can a student order for a friend? If yes, `orders` needs both `placed_by_student_id` and `for_student_id`, and every screen that says "your orders" now has to decide which it means. That is not a database detail; it changes the product.

Can one dish have two prices — one for students, one for staff? Then price does not belong on `menu_items` at all; it belongs in a separate table keyed by dish and audience. One sentence from a founder, three tables of consequence.

Is an order editable after it is placed? If yes, you need history, because "what did this order look like when it was paid?" stops having an obvious answer.

All three founders should be in the room for these, and the answers belong in the decision log. Downstream, chapter 9 reads queries across these tables, chapter 11 changes them safely, chapter 14 restricts who can see which rows, and M11's checkpoints will ask you to defend the design.

### The four passes, concretely

**Pass one: nouns become tables.** Read your user stories and underline every noun that has its own identity and its own lifecycle. "Student", "order", "dish" survive. "Total", "name" and "status" do not — they are facts about something else. Resist the urge to create a table for something that only ever appears as a column.

**Pass two: facts become columns, with types and nullability decided.** For each column, three questions. What type? Can it genuinely be absent? Does it have a sensible default? Money is integer paise. Timestamps are UTC. Statuses are a fixed, named list, not free text — `placed`, `preparing`, `ready`, `collected`, `cancelled` — because free text becomes "Ready", "ready" and "redy" within a month.

**Pass three: relationships become keys.** One-to-many puts the foreign key on the many side. Many-to-many creates a join table, and you then ask whether the pairing has facts of its own.

**Pass four: rules become constraints.** This is the pass everyone skips and the one that pays. Write out the sentences that must always be true of your data, then make the database enforce each one. "A quantity is between 1 and 10." "An email appears at most once." "A completed order has a payment reference." "An order total is never negative." Every rule you cannot express as a constraint is a rule your application code must remember forever, across every path, including the ones AI writes next month.

Alongside those, write the list of **impossible states** — the rows your product must never be able to hold. An order with no items. A payment for an order that does not exist. Two students with the same account. A dish marked both available and deleted. That list is the most valuable artifact of this chapter, because it is what you test against.

### Words to own

| Term | Plain meaning | Example |
|---|---|---|
| Schema | The full definition of your tables and rules | your design |
| Entity | A noun with its own identity and lifecycle | a student |
| Attribute | One fact about an entity | `phone` |
| Cardinality | How many of one relate to how many of another | one-to-many |
| Constraint | A rule the database itself enforces | `qty BETWEEN 1 AND 10` |
| Enum / status list | A fixed set of allowed values | `placed`, `ready` |
| Default | The value used when none is given | `status = 'placed'` |
| Impossible state | A row your product must never hold | order with no items |
| Audit column | Who and when, kept on every row | `created_at` |
| Soft delete | Marking inactive instead of removing | `is_active = false` |

### Tradeoffs and gotchas

**Design for the product you are shipping in eleven weeks, not the one in your pitch deck.** Multi-branch, multi-city, multi-currency and loyalty tiers all cost you columns, joins and bugs today for a future that may not arrive. The cheap future-proofing is clean naming and a surrogate key on everything; the expensive kind is tables for features nobody has agreed to build.

**But do not design yourself into a corner on identity and money.** Those two are genuinely hard to change later. Surrogate ids, integer paise, UTC timestamps, and a status column with a fixed vocabulary. Four decisions, made once, that save you a migration each.

**AI will give you a generic schema for your category unless you stop it.** Food app gets riders. Marketplace gets sellers and commissions. Social app gets follows and feeds. The defence is to put your user stories in the brief and add a non-goal: "do not add any table that is not required by a story I listed."

**Status as free text is a slow disaster.** It arrives as `status: string` in almost every AI schema. Within a month you will have three spellings of "ready", a screen that filters on the wrong one, and no way to know which rows are affected.

**Every table wants `created_at` and `updated_at` in UTC.** It costs nothing on day one. Without them, your first "when did this break?" question in M07 is unanswerable.

**A table per user role is a trap.** `students`, `staff` and `admins` as three tables means three login paths, three sets of permissions, and no way to make one person both. One `users` table with a role column is almost always right, and chapter 14 builds on exactly that.

### Verify it

Six checks, and the first two are the ones that actually catch things.

1. **Walk every user story through the schema out loud.** For each story, name the rows that would be created or changed. A story you cannot trace is a missing table or column, and finding it now costs minutes rather than weeks.
2. **Attempt every impossible state.** Take your list and try to insert each one directly into the database. Every single one that succeeds is a missing constraint. This is the single highest-value hour in the module.
3. **Count the tables you cannot justify.** For each table, name the user story that needs it. Any table with no story gets deleted now, while deleting is free.
4. **Check the four hard-to-change decisions.** Surrogate primary keys, integer paise, UTC timestamps, fixed status vocabulary. All four present, everywhere.
5. **Ask what happens on delete, for every foreign key.** Refuse, cascade, or soft delete — chosen, not defaulted.
6. **Have the non-technical founder read the table and column names.** If they cannot tell you what `order_items.unit_price_paise` means, rename it. A schema your own team cannot read is a schema your own team will misuse.

Red flags in AI-generated schemas: tables for a business model you do not have; `status` as free text; any float in a money column; everything nullable; no `created_at`; a `users` table with twenty columns of unrelated profile data; separate tables per role; a `metadata` or `extra_data` column that will become a dumping ground for decisions nobody wanted to make; and a schema delivered with no constraints at all beyond primary keys, which is a drawing rather than a design.

## Walkthrough
1. Open your PRD from M01. Copy out six to ten user stories in the form "as a [role] I can [action] so that [reason]". If you do not have six, your PRD is the thing to fix first.
2. Underline every noun in those stories. Cross out the ones that are facts about something else rather than things in their own right. What remains is your first table list.
3. For each surviving noun, write one sentence saying what one row represents. Any sentence needing "and" is two tables.
4. For each table, list the facts you must store, and for each fact write the type, whether it can be absent, and its default. Mark every money column as integer paise and every timestamp as UTC.
5. Draw the relationships on paper. One-to-many gets an arrow; put the key on the many side. For every many-to-many, name the join table and ask what facts belong to the pairing itself.
6. Write your rules list: sentences that must always be true of your data. Aim for at least eight, phrased as plain English assertions.
7. Write your impossible-states list: rows your product must never hold. Aim for at least six. Be specific — "an order whose total does not match the sum of its lines" rather than "bad data".
8. Now — and only now — brief AI with the stories, the table list, the rules and the impossible states, and ask for the schema plus the constraint for each rule. Forbid any table not required by a story.
9. Review what comes back against your own lists, item by item, with a pen. Mark every rule it did not enforce and every table it invented.
10. Apply the schema to a scratch database and attempt all six impossible states as direct inserts. Record which succeeded; each success is a missing constraint and a brief to write.
11. Walk all your user stories through the final schema out loud with a teammate, naming the rows each one touches, then record in your decision log the three product questions the design forced you to answer and what the team decided.

## Talk to AI
```prompt
CONTEXT:      Three-person student team, [N] weeks to demo, free-tier
              managed PostgreSQL. We will build on this schema for months,
              so I care more about what it forbids than what it allows.
              Product: [one sentence]
              Our committed conventions: surrogate integer or UUID primary
              keys that carry no meaning; all money as integer paise in
              columns named *_paise; all timestamps UTC; every status is a
              fixed vocabulary, never free text; every table has created_at
              and updated_at.

              OUR USER STORIES — the schema must support exactly these and
              nothing more:
              [paste 6-10 stories in "as a X I can Y so that Z" form]

              RULES THAT MUST ALWAYS BE TRUE OF OUR DATA:
              [paste your 8+ plain-English assertions]

              STATES THAT MUST BE IMPOSSIBLE TO STORE:
              [paste your 6+ impossible states]

GOAL:         A schema where every one of my stories is recordable and
              every one of my impossible states is rejected by the database
              itself, not by application code.

REQUIREMENTS: 1. Table list first, with one sentence per table saying what
                 one row represents, and the story number that requires it
              2. Then columns: type, nullable or not with a justification
                 for every nullable one, and defaults
              3. Then relationships, with the foreign key on the correct
                 side and the delete behaviour chosen explicitly — refuse,
                 cascade or soft delete — with its effect on history
              4. Then, for each of my rules, the exact constraint that
                 enforces it. If a rule cannot be expressed as a database
                 constraint, say so plainly and tell me where it must live
                 instead
              5. Then a table mapping each of my impossible states to the
                 specific constraint that prevents it. Any state you cannot
                 prevent must be listed as an unguarded risk
              6. Finally, walk three of my stories through your schema and
                 name the rows each one creates or changes

CONSTRAINTS:  PostgreSQL on a free tier. Do not add any table that is not
              required by a story I listed. No multi-tenant, multi-branch,
              multi-currency or loyalty structures. No table per user role.
              No free-text status columns. No floats for money.

NON-GOALS:    No indexes yet — that is a later chapter. No migration
              scripts. No application code. No seed data. Do not design the
              authentication tables beyond a users table with a role.

ACCEPTANCE:   - Every table cites the story that requires it
              - Every rule I listed maps to a named constraint, or is
                explicitly declared unenforceable with a reason
              - Every impossible state maps to the constraint that blocks
                it, or is flagged as an unguarded risk
              - No nullable column exists without a written justification
              - Three of my stories are traced through the tables

ASK:          Give me the table list and the story mapping FIRST and stop
              there. I want to approve the table list before you write a
              single column definition.
```

## Mission

**Objective.** A schema for your own startup, derived from your own user stories, where every rule you wrote is enforced by the database and every impossible state you listed has been attempted and refused.

**Steps.**
1. **Frame** — copy six to ten user stories out of your PRD. All three of you in the room; this is a product session, not a coding session.
2. **Four passes by hand** — nouns to tables, facts to columns with types and nullability, relationships to keys, rules to constraints. On paper or in a plain file. Before AI.
3. **Decide the product questions** — find at least three questions the design forces, of the kind "can a student order for a friend?" or "can a price differ by audience?". Answer each as a team and write down what you decided and why.
4. **Write your two lists** — at least eight plain-English rules that must always be true, and at least six states that must be impossible to store.
5. **Brief** — give AI the stories, your table list, your rules and your impossible states. Demand the table list and story mapping first, and approve it before any columns.
6. **Review on paper** — check its output against your own lists item by item. Mark every rule it failed to enforce and every table it invented that no story needs.
7. **Verify by attack** — apply the schema to a scratch database and attempt every impossible state as a direct insert. Record which were refused and which succeeded.
8. **Close the gaps** — for each state that succeeded, write one precise iteration naming the constraint you want, and confirm afterwards that the insert is now refused.
9. **Trace the stories** — walk every user story through the final schema out loud with the teammate who wrote it, naming the rows each one touches. Fix anything that cannot be traced.
10. **Document** — commit the schema and a short `SCHEMA-DECISIONS.md` recording the three product questions, your answers, and the delete behaviour you chose for each key relationship.

**Upload.**
- Your hand-written four-pass work, before AI → photo or attached file
- Your rules list and impossible-states list → attached file or post caption
- The AI conversation, including the table list you approved first → share link in `ai_chat_url`
- The final schema → attached file, plus a screenshot of the tables in your database
- Proof of attack → screenshots of every impossible-state insert attempt with the error or the success it produced, before and after your fixes
- Your decision document naming the three product questions and your answers
- Reflection, 150 words max → the product question the schema forced you to answer that you had not thought about, and one thing the AI got wrong

**Self-check.** Tick every box before you submit.
- [ ] My tables came from my own user stories, not from a one-line prompt
- [ ] Every table in my final schema cites the story that requires it, and I deleted the ones that could not
- [ ] I wrote at least eight rules and at least six impossible states before AI saw anything
- [ ] I attempted every impossible state as a real insert and recorded the result
- [ ] Every state that succeeded is now refused, and I have the screenshot proving both
- [ ] Every money column is an integer in paise; no floats anywhere
- [ ] Every timestamp is UTC and every table has `created_at`
- [ ] No status column holds free text
- [ ] Every nullable column has a written justification
- [ ] Every foreign key has a deliberate delete behaviour recorded with its effect on history
- [ ] I traced every user story through the schema out loud with a teammate
- [ ] My decision document names three product questions the design forced

**Stretch.** Hand your schema, with no other context, to a fresh AI session and ask it to list the states this schema still allows that a sensible product would forbid. Treat anything it finds that is genuinely wrong as a constraint you missed, and add it.

## Stuck? Ask AI
```prompt
Here are my user stories: [paste them]. Do not design a database. Instead, ask me the product questions this design will force me to answer — the ones where two reasonable answers lead to different tables. Ask them one at a time and wait for my answer before the next.
```

```prompt
Here is my schema: [paste it]. Do not improve it. List every row this schema would currently allow me to insert that a sensible version of my product should forbid. For each one, show me the insert that would succeed and name the constraint that should have stopped it.
```

```prompt
Explain the difference between a rule I can enforce in the database and a rule that has to live in application code. Give me five example rules from a food-ordering product, tell me which category each falls into, and explain what makes the difference.
```

## Quiz

### Q1 · L3
You ask AI for "a database for a canteen app" and get fourteen tables including riders, delivery zones and loyalty points. What went wrong?
- (a) The AI used the wrong database engine
- (b) You supplied no user stories, so it produced a generic schema for your category ✅
- (c) Fourteen tables is always too many
- (d) The prompt should have specified PostgreSQL
> With no constraints it pattern-matches the category from its training data. The defence is your own stories in the brief plus a non-goal forbidding tables no story requires.

### Q2 · L2
Why should a status column hold a fixed vocabulary rather than free text?
- (a) Free text takes more disk space
- (b) Free text cannot be indexed
- (c) Within a month you will have "Ready", "ready" and "redy" and no way to know which rows are affected ✅
- (d) Databases do not support text columns in queries
> A fixed set of allowed values makes a whole class of filtering bug impossible, and it makes the valid state machine visible in the schema itself.

### Q3 · L4
Which of these questions is a product decision disguised as a schema detail?
- (a) Whether the primary key is an integer or a UUID
- (b) Whether a student can place an order on behalf of a friend ✅
- (c) Whether timestamps are stored in UTC
- (d) Whether `created_at` has a default value
> It changes the `orders` table, and it changes what "your orders" means on every screen. This is why M01 insists the schema is not Tech's call alone.

### Q4 · L4
What is the highest-value verification you can run on a new schema?
- (a) Read every table definition carefully with a teammate
- (b) Count the foreign keys and confirm each points somewhere real
- (c) Attempt to insert every state on your impossible-states list ✅
- (d) Ask the AI to review its own design for weaknesses
> A constraint only exists if it refuses. Every impossible state that inserts successfully is a missing constraint, and each one is a class of bug your application code will have to remember forever.

### Q5 · L5
Your team debates adding `branch_id` to every table "so we can expand to other colleges later". The demo is in eleven weeks and there is one canteen. What is the stronger argument?
- (a) Add it now — retrofitting a tenant column later is a large migration
- (b) Leave it out; the cheap future-proofing is surrogate keys and clean naming, and this costs joins and bugs today ✅
- (c) Add it but leave it nullable so nothing breaks
- (d) Create a separate database per branch now
> Multi-tenancy is a real cost on every query, every permission check and every test, paid immediately for a future nobody has committed to. Identity and money conventions are worth deciding early; speculative structure is not.

## Go deeper
- [PostgreSQL docs: data definition and constraints](https://www.postgresql.org/docs/current/ddl.html)
- [A plain-language guide to database design decisions](https://www.postgresql.org/docs/current/datatype.html)
