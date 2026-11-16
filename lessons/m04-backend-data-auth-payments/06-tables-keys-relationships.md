---
id: m04-06-tables-keys-relationships
module: m04-backend-data-auth-payments
slug: tables-keys-relationships
title: "Tables, Rows, Keys and Relationships"
order: 6
duration_min: 25
is_free: false
mission: false
volatility: low
last_reviewed: 2026-11-16
---

## Key learnings
- Read a table definition and say what each column is for and what it forbids
- Tell a primary key from a foreign key and explain what each one guarantees
- Recognise one-to-many and many-to-many, and where the join table comes from
- Spot the duplicated-data smell that causes two screens to disagree
- Check that AI's tables actually make the bad rows impossible

## Lesson

### The pain

The canteen app's first database has one table called `orders`. It has a column for the student's name, their phone number, the dish name, the dish price, and the quantity.

For two weeks it is wonderful. Then a student changes their phone number. It updates in their profile and stays old in all forty of their past orders, so the canteen calls a dead number about a missing order. Then the price of a samosa goes from ₹12 to ₹15 and somebody runs an update that rewrites the price on every historical order, destroying the record of what was actually charged in September. Then a student's name is entered three different ways — "Priya S", "priya s", "Priya Sharma" — and the "top customers" screen shows one person as three.

Nobody wrote a bug. The structure allowed all of it. A table where the same fact is written in forty places will eventually have forty different versions of that fact, and no amount of careful coding prevents it.

### The idea

**A good schema makes wrong data impossible to store, rather than relying on code to remember not to store it.**

A **table** holds one kind of thing: students, menu items, orders. A **row** is one of that thing. A **column** is one fact about it, with a type and rules.

A **primary key** is the column that uniquely identifies a row forever. A **foreign key** is a column in one table that points at another table's primary key, and the database *enforces* that the target exists — you cannot have an order belonging to student 7 if student 7 does not exist.

That enforcement is the whole point. Rules written in the schema are kept by the database, for every writer, forever. Rules written in your application code are kept only by the code paths that remember them.

### The analogy

The canteen keeps three registers, not one.

One register lists students: roll number, name, phone. Each student appears once. The roll number is the primary key — it never changes and no two students share it.

One register lists dishes: code, name, today's price. Each dish appears once.

One register lists orders, and here is the trick: an order line does not write out the student's name or the dish's price. It writes the roll number and the dish code — plus, crucially, the price *charged at that moment*, because that is a fact about the order and not about the dish.

Change a phone number and you change one line in one register. Raise the price of a samosa and yesterday's orders still say ₹12, because they recorded what was actually paid.

**Where the analogy breaks:** a paper register tolerates a roll number that does not exist — the clerk writes it and nobody notices. A database with a foreign key refuses the write outright. That refusal feels obstructive and is the single most valuable thing the schema does for you.

### Picture it

```
  students                     orders
┌──────────────────┐        ┌────────────────────────┐
│ id      PK       │◀───┐   │ id            PK       │
│ name             │    └───│ student_id    FK       │
│ phone            │        │ placed_at              │
│ created_at       │        │ status                 │
└──────────────────┘        │ total_paise            │
                            └───────────┬────────────┘
  menu_items                            │ 1 order has
┌──────────────────┐                    │ many lines
│ id      PK       │◀───┐               ▼
│ name             │    │   order_items
│ price_paise      │    │ ┌────────────────────────┐
│ is_available     │    └─│ order_id      FK       │
└──────────────────┘      │ menu_item_id  FK       │
                          │ qty                    │
                          │ unit_price_paise       │
                          └────────────────────────┘
```

Four tables: one row per student, one per dish, one per order, and one per line within an order — with foreign keys pointing at the owners and the price copied into the line because it is a fact about that sale.

### Where it lives

This is the "data" box from M00's four boxes. Your server reaches into it on every request, and nothing else should — in chapter 14 you will see why letting a browser talk to a database directly is a problem.

M01 made a point you should now take literally: the schema is a product decision, not just Tech's call. "Can a student order for a friend?" is answered in the `orders` table by whether there is a separate `ordered_for` column. "Can a dish have two prices on two days?" is answered by whether price lives on `menu_items` or in a separate price table. These are product questions wearing database clothes, and all three founders should be in the room.

Chapter 7 is where you design your own. Chapter 9 is how you read a query across these joins. Chapter 11 is how you change the schema without losing data.

### One-to-many, many-to-many, and where join tables come from

Almost every relationship is one of three kinds.

**One-to-one.** A student has one profile. Rare, and usually better folded into one table unless you have a good reason.

**One-to-many.** One student has many orders. This is the common case, and the rule is simple: **the foreign key goes on the "many" side.** `orders.student_id` points at `students.id`. There is no list of orders stored on the student — you find them by asking the orders table.

**Many-to-many.** A dish can be in many categories, and a category contains many dishes. Neither side can hold the key, so you create a third table — a **join table** — with one row per pairing: `menu_item_categories(menu_item_id, category_id)`. Students are often confused about where this table comes from. It comes from the fact that the relationship itself has no natural home, and frequently it earns extra columns of its own: who added it, when, in what display order.

`order_items` in the diagram is a many-to-many join between orders and menu items, and look at what it carries: `qty` and `unit_price_paise`. Those are facts about the pairing, not about either side. Recognising when a relationship deserves its own columns is most of what schema design is.

### Words to own

| Term | Plain meaning | Example |
|---|---|---|
| Table | One kind of thing | `students` |
| Row / record | One of that thing | one student |
| Column / field | One fact about it, with a type | `phone` |
| Primary key (PK) | The unique, permanent id of a row | `students.id` |
| Foreign key (FK) | A pointer to another table's PK, enforced | `orders.student_id` |
| Unique constraint | No two rows may share this value | one account per email |
| Check constraint | A rule the value must satisfy | `qty > 0` |
| Join table | A table that exists to pair two others | `order_items` |
| Normalisation | Storing each fact in exactly one place | phone only on `students` |
| Denormalisation | Copying a fact on purpose, for a reason | `unit_price_paise` |

### Tradeoffs and gotchas

**Store each fact once — except when the copy is a different fact.** The phone number on an order is a duplicate and a bug. The price on an order line looks like a duplicate and is not: `menu_items.price_paise` means "what it costs today", `order_items.unit_price_paise` means "what this student was actually charged". If you cannot say which of those two you mean, you will eventually rewrite history. Learning to tell these apart is the main skill of this chapter.

**Use a surrogate primary key, not something meaningful.** A roll number feels like a natural key until the college changes its format, or a student has two, or someone was entered wrong. An `id` that means nothing can never become wrong. Keep the roll number as a column with a unique constraint if you need it.

**Never store money as a float.** `price_paise` as an integer. `240.00` as a decimal is survivable; as a float it is a slow leak that nobody notices until a reconciliation in chapter 17 is off by a few paise and you cannot explain why.

**Nullable columns are decisions, not defaults.** AI output tends to make everything nullable because it never fails to insert. Then you have orders with no student, totals with no value, and code full of checks for states that should have been impossible. Ask for each column: can this genuinely be absent? If not, say so in the schema.

**A soft delete is usually right and must be designed.** If a dish is deleted, what happens to the three hundred historical order lines pointing at it? Either you keep the row and mark it `is_active = false`, or your history breaks. Decide before you need it, not during the demo.

**Timestamps in UTC, always.** `created_at` and `updated_at` on nearly every table, stored in UTC, displayed in IST. Mixed time zones in a database are an afternoon you will never get back.

### Verify it

When AI hands you a schema, five checks.

1. **Try to insert a bad row and see if the database stops you.** An order for a student id that does not exist. A quantity of `-5`. Two accounts with the same email. A good schema refuses all three at the database level. If your application code is the only thing stopping them, the next code path will not.
2. **Hunt for duplicated facts.** Read the column list of every table and look for the same real-world fact in two places. For each one, ask: is this a copy, or a different fact at a different moment? Copies without a reason get removed.
3. **Check every money column.** Integer, named `_paise`, no floats anywhere.
4. **Check every nullable column.** For each one, ask the AI to justify why that value may be absent. Most cannot be justified, and each unjustified one is a class of bug in your application code.
5. **Ask what happens on delete.** Pick your most referenced table and ask: if I delete this row, what happens to everything pointing at it? There are three honest answers — refuse, cascade, or mark inactive — and the schema must have chosen one deliberately.

Red flags: a single wide table with twenty columns covering three different kinds of thing; a column holding a comma-separated list of ids, which is a join table someone refused to create; `price` on an order line that is a foreign-key lookup rather than a stored amount; no `created_at` anywhere; a user's email as the primary key; any column named `data` or `info` with free-form content in it.

## Walkthrough
1. Open your project's database and list every table with its columns. If you are using a hosted database, its table editor shows this. Write the list on paper.
2. For each table, say in one sentence what single kind of thing one row represents. If a sentence needs the word "and", that table is probably two tables.
3. Mark the primary key of every table. Then mark every foreign key, and draw an arrow to the table it points at.
4. Hunt for duplicated facts: the same real-world information stored in two tables. List each one, and decide whether it is a genuine copy to remove or a point-in-time record to keep.
5. Find a relationship in your product that is many-to-many. Name the join table that holds it, and list the columns that belong to the pairing rather than to either side.
6. Try to break it. Using the database's query editor, attempt to insert a row referencing an id that does not exist. Record whether it was refused, and read the error message.
7. Attempt to insert a negative quantity or a negative price. Record whether anything stopped you. Most first-draft schemas let this through.
8. Attempt to create two rows with the same email, phone or any value that should be unique. Record the result.
9. For every check that was not refused, write one line describing the constraint that is missing. That list is your next brief to AI.
10. Pick your most referenced table and decide the delete behaviour: refuse, cascade, or soft delete. Write the decision and the reason in your decision log.
11. Check every money column is an integer in paise, and every timestamp is UTC. Fix any that are not before they spread.

## Talk to AI
```prompt
CONTEXT:      I am reviewing the database schema of a student project
              rather than writing one. Three-person team, free tiers,
              PostgreSQL-compatible managed database.
              Product: [one sentence on what it does]
              Conventions we have committed to: all money is an integer
              number of paise in columns named *_paise; all timestamps are
              UTC; ids are surrogate keys that mean nothing.
              Here is the current schema, table by table, with column names
              and types exactly as they exist:
              [paste it]

GOAL:         Find every place where this schema permits data that should
              be impossible, before we build any more on top of it.

REQUIREMENTS: 1. For each table, state in one sentence what one row
                 represents. Flag any table whose sentence needs the word
                 "and", because that is two tables
              2. List every duplicated real-world fact across tables. For
                 each, tell me whether it is a genuine duplicate to remove
                 or a deliberate point-in-time record to keep, and why
              3. List every column that is nullable, and for each one give
                 me the justification or tell me it has none
              4. List every bad row this schema currently allows — negative
                 quantities, orphaned references, duplicate accounts,
                 impossible states — and for each name the constraint that
                 would make it impossible
              5. For my three most referenced tables, tell me what should
                 happen when a referenced row is deleted, with the tradeoff
                 of each option against historical records
              6. Flag any money column that is not an integer and any
                 timestamp that is not UTC

CONSTRAINTS:  Do not redesign the whole schema. Do not rename tables.
              Assume existing data must survive any change you suggest.
              Everything must work on a free-tier managed PostgreSQL.

NON-GOALS:    No migration scripts yet. No indexes yet. No code. Do not add
              features I have not asked for, and do not add any table that
              is not required to fix a problem you have identified.

ACCEPTANCE:   - Your bad-row list has at least three entries, because
                AI-generated schemas always allow some
              - Every duplicated fact is classified as copy or record, with
                a reason
              - Every nullable column is either justified or flagged
              - You name the delete behaviour for my three key tables and
                its effect on history

ASK:          Start with the bad-row list, before anything else. I want to
              see what my schema currently permits before I see your
              opinions about its structure.
```

## Stuck? Ask AI
```prompt
Explain the difference between a primary key and a foreign key using the registers of a college canteen as the example. Then give me five short descriptions of columns and ask me which kind of key each one is, or neither, without telling me the answers first.
```

```prompt
I do not understand why a many-to-many relationship needs a third table. Walk me through what goes wrong if I try to store it as a list of ids in a column, using a real example of dishes and categories, and show me what queries become impossible.
```

```prompt
Explain why the price stored on an order line is not the same fact as the price stored on a menu item, even though the numbers are identical on the day of the order. Then give me three more examples of facts that look like duplicates and are not.
```

## Quiz

### Q1 · L2
In a one-to-many relationship between students and orders, where does the foreign key go?
- (a) On the students table, as a list of order ids
- (b) On the orders table, pointing at the student ✅
- (c) In a separate join table between the two
- (d) On both tables, pointing at each other
> The key goes on the "many" side. You find a student's orders by asking the orders table, not by storing a list on the student, which no relational database supports cleanly.

### Q2 · L3
A student changes their phone number and the canteen still calls their old one about a past order. What is the structural cause?
- (a) The update query had a bug in its filter
- (b) The phone number was copied onto every order row instead of living only on the student ✅
- (c) The orders table was missing an index
- (d) The phone number column should have been nullable
> A fact stored in forty places will eventually have forty versions. Phone belongs on `students` once; orders reference the student by id.

### Q3 · L4
`menu_items.price_paise` is ₹15 today. `order_items.unit_price_paise` on a September row says ₹12. Is this a data integrity problem?
- (a) Yes — the two values must always be kept in sync
- (b) Yes — the order line should look up the current price instead
- (c) No — they are different facts: today's price versus what was charged ✅
- (d) No, but only because the order is already completed
> This is deliberate, necessary denormalisation. Looking up the live price from a historical order would rewrite what a student actually paid every time the canteen changes a price.

### Q4 · L4
Which of these is the strongest sign that a schema was never designed?
- (a) Every table has a `created_at` column
- (b) A column holds a comma-separated list of menu item ids ✅
- (c) Money is stored as integer paise
- (d) Primary keys are meaningless surrogate ids
> That list is a join table somebody refused to create. The database cannot enforce that those ids exist, cannot index them usefully, and cannot join on them.

### Q5 · L5
You want to prove AI's schema actually protects your data. What is the single most informative thing to do?
- (a) Read every table definition carefully
- (b) Ask the AI to explain its design decisions
- (c) Try to insert bad rows — an orphaned reference, a negative quantity, a duplicate email — and see which the database refuses ✅
- (d) Count the number of foreign keys it created
> A constraint only exists if it refuses. Reading the schema tells you what it says; attempting the write tells you what it enforces, and the gap between the two is where your bugs live.

## Go deeper
- [PostgreSQL docs: constraints](https://www.postgresql.org/docs/current/ddl-constraints.html)
- [Database normalisation explained in plain language](https://www.postgresql.org/docs/current/tutorial-sql.html)
