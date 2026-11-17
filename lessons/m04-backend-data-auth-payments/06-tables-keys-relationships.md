---
id: m04-06-tables-keys-relationships
module: m04-backend-data-auth-payments
slug: tables-keys-relationships
title: "Tables, Rows, Keys and Relationships"
order: 6
duration_min: 15
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

The canteen app's first database has one table, `orders`, with columns for the student's name and phone, the dish name and price, and the quantity. For two weeks it is wonderful.

Then a student changes their phone number and it stays stale in all forty past orders, so the canteen calls a dead number. Then a samosa goes from ₹12 to ₹15, someone updates every historical order, and the record of what September charged is gone. Nobody wrote a bug — the structure allowed it.

### The idea

**A good schema makes wrong data impossible to store, rather than relying on code to remember not to store it.**

A **table** holds one kind of thing, a **row** is one of them, a **column** is one fact about it. A **primary key** identifies a row forever; a **foreign key** points at another table's primary key, and the database *enforces* that the target exists.

That enforcement is the point. Rules in the schema are kept for every writer, forever; rules in application code are kept only by the paths that remember them.

### The analogy

The canteen keeps three registers, not one: students, dishes and orders. Students lists roll number, name and phone, the roll number being the primary key that never changes. Orders holds the trick: a line writes the roll number and the dish code, plus the price *charged at that moment*, because that is a fact about the order rather than the dish. So changing a phone number changes one line, and raising the samosa price leaves yesterday's orders at ₹12.

**Where the analogy breaks:** a paper register tolerates a roll number that does not exist; a foreign key refuses the write outright. That refusal feels obstructive and is the most valuable thing the schema does for you.

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

Every relationship there is one of three kinds.

| Kind | Example | Where the key goes |
|---|---|---|
| One-to-one | a student has one profile | either side; usually fold into one table |
| One-to-many | one student has many orders | on the "many" side: `orders.student_id` |
| Many-to-many | a dish sits in many categories | a third **join table**, one row per pairing |

A join table exists because the relationship has no natural home, and it often earns columns of its own, like `order_items.qty`.

### Where it lives

This is M00's "data" box, and only your server should reach into it — chapter 14 shows why letting a browser talk to a database is a problem.

M01's point now becomes literal: the schema is a product decision, not Tech's call. "Can a student order for a friend?" is answered by whether `orders` carries an `ordered_for` column. That is a product question wearing database clothes, so all three founders should be in the room.

Next: chapter 7 designs your own · chapter 9 reads across these joins · chapter 11 changes them safely.

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

**Store each fact once — unless the copy is a different fact.** A phone number on an order is a duplicate and a bug. A price on an order line is not: `menu_items.price_paise` is "what it costs today", `order_items.unit_price_paise` is "what this student was charged". Confuse them and you rewrite history.

**Use a surrogate primary key, not something meaningful.** A roll number feels natural until the college changes its format. An `id` that means nothing can never become wrong.

**Nullable columns are decisions, not defaults.** AI makes everything nullable because that never fails to insert; then you have orders with no student and code checking for impossible states. Delete behaviour is the same kind of decision: if a dish is removed, either you mark it `is_active = false` or the order lines pointing at it break your history. Money is always integer paise, timestamps always UTC.

### Verify it

Five checks on any schema AI hands you.

1. **Insert a bad row and see if the database stops you.** An order for a student id that does not exist, a quantity of `-5`, two accounts with one email. A good schema refuses all three.
2. **Hunt for duplicated facts.** For each, ask: a copy, or a different fact at a different moment? Copies without a reason get removed.
3. **Check money columns.** Integer, named `_paise`, no floats.
4. **Check every nullable column.** Make the AI justify each one; every unjustified one is a class of bug.
5. **Ask what happens on delete.** For your most referenced table: refuse, cascade, or mark inactive — deliberately.

Red flags: one wide table covering three kinds of thing · a comma-separated list of ids, which is a join table someone refused to create · `price` on an order line that is a lookup rather than a stored amount · no `created_at` anywhere · an email as primary key · a column named `data` or `info`.

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
