---
id: m04-14-roles-permissions-rls
module: m04-backend-data-auth-payments
slug: roles-permissions-rls
title: "Roles, Permissions and Row-Level Security"
order: 14
duration_min: 15
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

The canteen app has students, staff and a manager, and nobody wrote down who may do what. An afternoon of testing found three bugs: staff read every student's phone number, because the orders endpoint returns the whole student record; a student can cancel anyone's order, because the endpoint checks role and not owner; and last week's endpoint has no check at all, because it was "only for the dashboard" — which is a URL you can type. Nobody designed permissions; they accumulated.

### The idea

**Write the permission matrix first, enforce it in two places, and default to deny.**

A **role** is a named group with the same powers; a **permission** is one specific action; the **matrix** is the grid of the two — a product document, not a technical one.

Enforce it twice: your application checks role and ownership on every request, and **row-level security** — policies inside the database — catches what it forgets. **Default deny** closes every new endpoint and table until someone opens it.

### The analogy

The canteen office has a laminated sheet: rows for student, cook, cashier, manager; columns for the kitchen, the stock room, the till, the accounts, each cell a tick or a cross. Row-level security is the stock room's lock: the sheet says cooks may enter, and the lock still stops a new cook nobody briefed.

**Where the analogy breaks:** a physical lock guards one door; row-level security is finer — it lets you into the room but only to your own shelf.

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

The matrix those gates enforce:

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

The two order rows are the lesson: "read own" and "read any" are different permissions, and collapsing them is the opening bug. Personal data earns its own row, because returning a full student object is how phone numbers leak.

### Where it lives

Layer one is your server, where chapter 12 put it. Layer two lives inside the database: you attach a **policy** to a table and the database applies it to every query, forever. "On `orders`, a student may select only rows where `student_id` equals the current user." `SELECT * FROM orders` then returns only that student's rows — you cannot forget it, because it is not in your code. It is also what makes browser-to-database architectures safe: client access with no policy is an open door. Next: 15 does this for files.

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

**Row-level security is not a substitute for application checks.** It controls which *rows* are visible, not whether a student may call a staff-only endpoint.

**Policies are easy to write permissively by accident.** `USING (true)` permits everything and looks configured in a dashboard; if a policy never mentions the current user, be suspicious.

**Roles in a token go stale.** Demote someone and their JWT still says staff until it expires, so read the role from the database — never from the request, where it appears as an `is_admin` flag.

**The service key bypasses every policy by design.** In frontend code or a committed file, every policy you wrote is irrelevant.

### Verify it

Mechanical, and the whole job: **test every cell of your matrix as the wrong user.**

1. **Create one test account per role,** plus a second ordinary account.
2. **For every "no" cell, attempt the action through the API, not the UI, and confirm it is refused.** Record every status code; that grid is your evidence.
3. **Attempt cross-user access on every endpoint that takes an id.** Your own record, then a teammate's: `403` or `404`, never data.
4. **Prove row-level security independently.** Query the table with a student's credentials; seeing everyone's rows means the policy is missing or permissive.
5. **Check response bodies, not just status codes.** A `200` with the right rows can still carry the student's phone number inside each one.
6. **Add a brand-new endpoint and confirm it is denied before you add any check.** This tests your pattern, not your endpoints.

Red flags: an endpoint checking `if (user)` and nothing more; one `isAdmin` boolean doing a matrix's job; a policy of `USING (true)`; a service key in frontend code or git; a role read from a request body; a full user object where a name was needed; "it's only linked from the admin page".

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
