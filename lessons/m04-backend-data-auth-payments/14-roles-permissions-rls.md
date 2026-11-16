---
id: m04-14-roles-permissions-rls
module: m04-backend-data-auth-payments
slug: roles-permissions-rls
title: "Roles, Permissions and Row-Level Security"
order: 14
duration_min: 25
is_free: false
mission: false
volatility: low
last_reviewed: 2026-11-16
---

## Key learnings
- Write a permission matrix your whole team agrees on before any code
- Tell a role check from an ownership check and apply both on every endpoint
- Explain how row-level security catches the check your application forgot
- Default to deny, so a new endpoint is closed until you open it
- Verify permissions by testing every cell of the matrix as the wrong user

## Lesson

### The pain

The canteen app has three kinds of people: students, canteen staff, and the manager. Nobody has ever written down who may do what, so every endpoint was decided by whoever briefed the AI that afternoon.

The results, found during one afternoon of testing: staff can read every student's phone number, because the endpoint that lists orders returns the full student record. A student can cancel any order by id, not only their own, because the endpoint checks the role and not the owner. The manager cannot see yesterday's revenue, because that endpoint was built for an `admin` role that nobody ever assigned to her. And one endpoint added last week has no check at all, because it was "only for the dashboard" and the dashboard is the thing you reach by typing a URL.

Four different bugs with one cause. Permissions were never designed; they accumulated one prompt at a time.

Then the worse version. The team uses a hosted database with a client library, and a developer wires the browser directly to it to save a round trip. Now the browser holds a key that can read a table. Every row of it. The permission check the server used to do is simply not in the path anymore.

### The idea

**Write the permission matrix first, enforce it in two places, and default to deny.**

A **role** is a named group of people with the same powers: student, staff, manager. A **permission** is one specific thing that may be done: read own orders, read all orders, mark a dish sold out. The **matrix** is the grid of roles against actions with a yes or no in each cell. It takes twenty minutes and it is a product document, not a technical one.

Enforcement happens in two layers, and the second is what makes this chapter different from chapter 12. The application checks the role and the ownership on every request. And **row-level security** — policies inside the database itself — makes the data physically unreachable even if the application forgets. Belt and braces, where the braces survive the next AI-generated endpoint that forgets its check.

**Default deny** is the rule that makes the whole thing hold: a new endpoint, a new table, a new role is closed until someone explicitly opens it. The opposite — open until someone remembers to close it — is how the dashboard endpoint from last week happened.

### The analogy

The canteen has a laminated sheet in the office: rows for student, cook, cashier, manager; columns for enter the kitchen, open the stock room, take money from the till, see the month's accounts. Each cell is a tick or a cross. New staff read it on day one, and arguments end by pointing at it.

Row-level security is the stock room's lock. The sheet says cooks may enter; the lock enforces it. If a new cook is hired and nobody shows him the sheet, the lock still stops him. The sheet alone is a policy; the lock is what makes it true when somebody forgets.

**Where the analogy breaks:** a physical lock is one lock on one door. Row-level security is finer than that — it is a lock that lets the same person into the same room and permits them to touch only their own shelf. "Read orders" is not one permission; it is "read orders where the student is you", and that distinction is the single most commonly missed thing in this whole module.

### Picture it

```
            ┌──────────────────────────────────────┐
 request ──▶│ 1. APPLICATION LAYER                 │
            │    role check: is caller staff?      │
            │    ownership check: is this row      │
            │    theirs?                           │
            └──────────────┬───────────────────────┘
                      pass │ 403 if not
                           ▼
            ┌──────────────────────────────────────┐
            │ 2. DATABASE LAYER (row-level         │
            │    security)                         │
            │    policy: a student sees only rows  │
            │    where student_id = current user   │
            └──────────────┬───────────────────────┘
                           ▼
                   rows this user may see
                   — and no others, even if
                   layer 1 forgot to filter
```

Two independent gates: the application decides whether the action is allowed, and the database independently restricts which rows are visible at all, so a missing application check leaks nothing.

### Where it lives

Layer one lives in your server, exactly where chapter 12 put it. Layer two lives inside the database, which is new.

Row-level security works by attaching a policy to a table: a condition the database applies to every query against it, automatically, forever. "On the `orders` table, a caller with role student may select only rows where `student_id` equals the current user id." Write `SELECT * FROM orders` and the database silently returns only that student's rows. There is no way to forget it, because it is not in your code.

This is what makes browser-to-database architectures safe at all. If a client library holds a key that can read a table, row-level security is the only thing standing between one student and everyone's data. Hosted platforms that offer direct client access make these policies the centre of their security model, and a table with client access enabled and no policy is an open door.

Forward: chapter 15 applies the same thinking to files, where signed URLs are the equivalent of a per-row permission. M07 returns to all of it as OWASP broken access control and as the DPDP Act's expectation that you limit access to personal data. M11's checkpoints will ask for your matrix.

### Writing the matrix

Rows are roles. Columns are actions — specific, verb-shaped, and scoped. The scope is where the real thinking happens.

| Action | Student | Staff | Manager |
|---|---|---|---|
| Read own orders | yes | yes | yes |
| Read any student's orders | no | yes | yes |
| Cancel own order, within 2 min | yes | yes | yes |
| Cancel any order | no | yes | yes |
| Read a student's phone number | no | own canteen only | yes |
| Mark a dish sold out | no | yes | yes |
| Change a dish price | no | no | yes |
| See the day's revenue | no | no | yes |
| Create a staff account | no | no | yes |

Look at the two rows that matter. "Read own orders" and "read any student's orders" are different permissions, and collapsing them into "read orders" is the bug from the opening story. Every single action involving someone else's data needs that split written out.

Note also "read a student's phone number" as its own row. Personal data deserves its own column in the matrix rather than travelling along inside a record that somebody is allowed to read for another reason. Returning the full student object from an orders endpoint is how phone numbers leak without anyone deciding to share them.

Two practical rules. **Three or four roles is usually right** for a product this size; a permissions system with per-user grants is a project of its own. And **put the matrix in your repository**, so changing it is a pull request your non-technical founder can read and argue with.

### Words to own

| Term | Plain meaning | Example |
|---|---|---|
| Role | A named group with the same powers | staff |
| Permission | One specific allowed action | mark a dish sold out |
| Permission matrix | The grid of roles against actions | the table above |
| Role check | Is this caller the right kind of user? | role == staff |
| Ownership check | Does this row belong to this caller? | `student_id == me` |
| Row-level security | Database policies filtering rows per user | on `orders` |
| Policy | One rule attached to a table | students see own rows |
| Default deny | Closed unless explicitly opened | new endpoints |
| Least privilege | Give the minimum needed to do the job | staff cannot set prices |
| Privilege escalation | Gaining powers you were not granted | student becomes admin |

### Tradeoffs and gotchas

**Row-level security is not a substitute for application checks.** It controls which *rows* are visible; it does not stop a student calling an endpoint that should be staff-only, and it does not stop a staff member doing a manager action on rows they can legitimately see. You need both layers, and the application layer is still where most of the decisions live.

**Policies are easy to write permissively by accident.** A policy of `USING (true)` is a policy that permits everything, and it looks like a configured table in a dashboard. Read every policy's condition out loud as a sentence; if the sentence does not mention the current user, be suspicious.

**Turning row-level security on without writing policies locks everyone out.** The default once enabled is to deny, so the app breaks completely, which at least fails loudly. Enabling client access to a table *without* policies fails silently in the other direction, which is worse.

**Roles in a token go stale.** If the role is a claim in a JWT from chapter 12 and you demote someone, their existing token still says staff until it expires. For privileged actions, read the role from the database rather than from the token.

**Beware the "god" service key.** Hosted databases issue an administrative key that bypasses every policy by design, for server use. If it ever reaches frontend code or a public repository, every policy you wrote is irrelevant. This is M07's secrets lesson and it is worth stating twice.

**Do not let an endpoint take the role from the request.** It sounds absurd written down; it appears in AI output regularly as an `is_admin` flag in a request body.

**More roles is not more security.** Seven roles in a three-person project means nobody knows which one to assign, and someone will be given the powerful one "for now".

### Verify it

The method is mechanical and it is the whole job: **test every cell of your matrix as the wrong user.**

1. **Create one test account per role,** plus a second ordinary account so you can attempt cross-user access.
2. **For every "no" cell, attempt the action and confirm it is refused.** Through the API directly, not through the UI — the UI hides buttons, and chapter 12 established that hiding is not protecting. Record the status code for each. This grid of results is your evidence.
3. **Attempt cross-user access on every endpoint that takes an id.** Your own record, then a teammate's. `403` or `404`, never data.
4. **Prove row-level security independently.** If your platform supports it, query the table with a student's credentials and confirm you see only that student's rows. If you can see all of them, the policy is missing or permissive.
5. **Check the response bodies, not just the status codes.** An endpoint can return `200` with the right rows and still include the student's phone number inside each one. Read the actual JSON against your matrix.
6. **Add a new endpoint and confirm it is closed by default.** This is the test that proves your pattern rather than your endpoints. If a freshly generated endpoint is reachable by anyone, your next AI session will create a hole.

Red flags: an endpoint checking `if (user)` and nothing more; a single `isAdmin` boolean doing the work of a matrix; row-level security enabled with a policy of `USING (true)`; a service key in frontend code or committed to git; a role read from a request body; a response that includes a full user object where only a name was needed; and any endpoint whose protection is described as "it's only linked from the admin page".

## Walkthrough
1. Sit down with both teammates and list every role in your product. Three or four. For each, write the single most dangerous thing it can do.
2. List every action in your product as a verb phrase. Be specific and split anything involving other people's data into "own" and "any" versions.
3. Build the matrix as a table: roles across, actions down, yes or no in every cell. Fill every cell; an empty cell is a disagreement you have not noticed.
4. Mark every cell where personal data is involved — phone numbers, email addresses, payment details. These deserve their own actions rather than riding inside another record.
5. Commit the matrix to your repository as a markdown file. It is now a document a pull request can change.
6. Create test accounts, one per role, plus a second ordinary user. Note the credentials somewhere your team can reach.
7. Go through every "no" cell in the matrix and attempt that action through the API as that role. Record the status code you got in a copy of the matrix. This grid is your audit.
8. For every cell that succeeded when it should have failed, write one line in a work list. Expect several.
9. For every endpoint that takes an id, attempt it with a teammate's id. Record the result in the same grid.
10. Read the response bodies of three endpoints you are allowed to call and check that no field exceeds what your matrix permits. Phone numbers are the usual offender.
11. Brief AI with the matrix and your work list, and ask it to close the gaps endpoint by endpoint, with role and ownership as two separate checks. Add row-level security policies for your most sensitive table. Then re-run the entire grid, and finally generate one brand-new endpoint and confirm it is denied by default before you add any check to it.

## Talk to AI
```prompt
CONTEXT:      Three-person student team, free tiers, [database platform].
              Users are Indian college students; we hold names, emails and
              phone numbers, so DPDP matters to us.
              Product: [one sentence]
              Our login works and we have already run chapter 12's attacks.

              OUR PERMISSION MATRIX — roles across, actions down, decided
              by the whole team:
              [paste the full table, every cell filled]

              OUR ENDPOINTS, with what each does and what it returns:
              [paste the list]

              HOW WE DETERMINE THE CALLER'S ROLE TODAY:
              [describe — from a token claim, from the database, or not
              at all]

GOAL:         Enforce this matrix in the application and, for our most
              sensitive table, in the database itself — and know exactly
              which cells are currently violated.

REQUIREMENTS: 1. Map every endpoint to the matrix cells it implements. Flag
                 every endpoint that implements no cell, and every cell no
                 endpoint enforces
              2. For each endpoint, state the role check AND the ownership
                 check separately. Never merge them
              3. Give me the list of my endpoints you predict are currently
                 violated, before you write any fix
              4. Check my response bodies against the matrix: name every
                 field that leaks data a role should not see, especially
                 phone numbers and emails
              5. Write row-level security policies for my most sensitive
                 table, and express each policy as a plain English sentence
                 that mentions the current user
              6. Tell me what the policies do NOT protect, so I do not
                 mistake them for complete protection
              7. Tell me whether reading the role from a token is safe for
                 my privileged actions, and what happens when I demote
                 someone
              8. Give me the default-deny pattern so that a brand-new
                 endpoint is closed until I open it, and tell me how to
                 test that

CONSTRAINTS:  Free tier. Keep roles to the ones in my matrix; do not invent
              new ones. No per-user permission grants. Every check must run
              server-side or in the database, never in the client. The
              administrative service key must never appear in client code.

NON-GOALS:    Do not add features. Do not change the schema beyond what a
              policy requires. Do not touch authentication. Do not build an
              admin UI for managing roles.

ACCEPTANCE:   - Every "no" cell in my matrix, attempted through the API as
                that role, returns 403 or 404
              - No response body contains a field my matrix forbids for
                that role
              - Querying my sensitive table as an ordinary user returns
                only that user's rows
              - A brand-new endpoint with no checks added is denied by
                default, and I can demonstrate it

ASK:          Start with the violation list and the cell-to-endpoint
              mapping. I want to see which cells my app currently breaks
              before I see a single fix.
```

## Stuck? Ask AI
```prompt
Explain the difference between a role check and an ownership check using a canteen with students, cooks and a manager. Then give me six endpoint descriptions from a food-ordering app and ask me, for each one, which checks it needs — without telling me the answers first.
```

```prompt
Explain row-level security as if I have never used a database beyond simple queries. Use one table of orders as the example, show me in plain language what the policy sentence says, and then tell me three things row-level security does NOT protect me from.
```

```prompt
Here is my permission matrix: [paste it]. Do not write code. Instead, tell me which cells are ambiguous or contradictory, which actions I have probably merged that should be split into "own" and "any" versions, and which personal data fields deserve their own row.
```

## Quiz

### Q1 · L2
What does row-level security do?
- (a) Encrypts each row before storing it
- (b) Applies a per-user condition to every query against a table, inside the database ✅
- (c) Replaces the need for role checks in your server code
- (d) Limits how many rows an endpoint can return
> It is a policy attached to the table, so even `SELECT *` returns only the rows that caller may see. It is a second independent layer, not a replacement for the first.

### Q2 · L3
A student calls `DELETE /api/orders/91`, an order belonging to someone else, and it is cancelled. The role check passed because students may cancel orders. What was missing?
- (a) Row-level security on the orders table
- (b) An ownership check that order 91 belongs to the caller ✅
- (c) A confirmation dialog in the UI
- (d) A rate limit on the endpoint
> "Cancel own order" and "cancel any order" are two different permissions. Merging them into "cancel orders" is the most common way a matrix gets implemented wrongly.

### Q3 · L4
Your dashboard endpoint has no permission check because "it is only linked from the admin page". What is the right assessment?
- (a) Acceptable, since the link is not shown to students
- (b) The endpoint is public to any logged-in user who types or guesses the URL ✅
- (c) Acceptable if the page itself redirects non-admins
- (d) A performance problem rather than a security one
> Attackers do not use your navigation. Default deny exists precisely so a new endpoint is closed until someone opens it, rather than open until someone remembers.

### Q4 · L4
You enable client access to a table on a hosted database and write no row-level security policies. What is the likely result?
- (a) The table is unreachable until policies exist
- (b) Any client with the public key can read every row ✅
- (c) Only the table owner can read it
- (d) Reads work but writes are blocked
> Enabling client access without policies is the silent-failure direction: the app works perfectly and everyone's data is readable. Enabling the feature and forgetting the policies is the single most common misconfiguration on these platforms.

### Q5 · L5
Your staff endpoint returns `200` with exactly the right orders, but each order includes the student's full record with their phone number. Is the matrix satisfied?
- (a) Yes — the role was allowed to read those orders
- (b) No — reading a phone number is a separate permission and the response leaks it ✅
- (c) Yes, provided the UI does not display the phone number
- (d) No, but only if the phone numbers are unverified
> Status codes are not the whole check. Personal data deserves its own row in the matrix rather than travelling inside a record somebody may read for a different reason — which is how phone numbers leak with nobody deciding to share them.

## Go deeper
- [PostgreSQL docs: row security policies](https://www.postgresql.org/docs/current/ddl-rowsecurity.html)
- [OWASP Authorization Cheat Sheet](https://cheatsheetseries.owasp.org/cheatsheets/Authorization_Cheat_Sheet.html)
