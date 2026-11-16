---
id: m04-12-authn-vs-authz
module: m04-backend-data-auth-payments
slug: authn-vs-authz
title: "Authentication vs Authorization; Passwords, Hashing, Sessions and JWTs"
order: 12
duration_min: 25
is_free: false
mission: false
volatility: low
last_reviewed: 2026-11-16
---

## Key learnings
- Separate "who are you" from "what may you do", and enforce the second on the server
- Catch the signature failure: login works and any student can open the admin page
- Say why a password is never stored and what hashing with a salt actually does
- Choose between a session and a token, and know what each cannot do
- Verify authorization by calling the endpoint directly as the wrong user

## Lesson

### The pain

This is the failure from M00, and it is the most common serious bug in AI-built applications. It deserves the whole chapter.

The canteen team adds Google login. It works beautifully: tap the button, pick an account, land on the home screen with your name at the top. Everyone congratulates themselves.

Two weeks later a student types `/admin` into the address bar out of curiosity. The admin dashboard loads. Every order, every student's phone number, the day's revenue, and a button to mark any dish sold out. She is logged in, so the app lets her in.

The team's defence is that the admin link is only shown to admins. That is true. The link is hidden; the page is not protected. Hiding a door is not locking it, and the address bar does not care what your navigation menu renders.

Then it gets worse. Even with the page protected, the *endpoints* are not. `GET /api/admin/orders` returns every order to any logged-in caller, because the only check anywhere is "is there a valid session?". A student with dev tools open — or just a guessed URL — reads the whole canteen's data from her phone.

**Login worked. Nothing was authorized. These are two different features and AI builds the first one by default.**

### The idea

**Authentication is proving who you are. Authorization is deciding what you may do. They are separate systems, and only the second one protects your data.**

Authentication answers: is this really Priya? Its tools are passwords, Google login, OTPs, passkeys.

Authorization answers: may Priya read this particular order, change this dish, open this page? Its tools are roles, permissions and ownership checks — and every one of them must run **on the server, on every request, for every resource.**

The shorthand to carry for the rest of your career: **authentication is the front door, authorization is every door inside.** AI reliably builds the front door. It frequently leaves every interior door open, and the app looks completely normal from the outside.

### The analogy

The canteen has a gate where a guard checks your college ID. That is authentication. Once inside, you are a verified student.

Authorization is everything after the gate. The kitchen is staff only. The cash box is the manager's. The stock room needs a key. A student who got through the gate has not thereby earned the right to open the cash box.

A canteen where the only check is at the gate — and anyone who entered can walk into the kitchen and take money from the till — is the broken Google login. The ID check worked perfectly. It was never the thing protecting the till.

**Where the analogy breaks:** a physical kitchen has a door you must walk through, and a cook who will shout at you. An API endpoint has no walls and no witnesses. There is no equivalent of "looking suspicious while walking into the kitchen" — a request from a student's phone to an admin endpoint looks exactly like a request from the manager's. Only your own code, on every single request, can tell them apart.

### Picture it

```
 request ──▶ ┌──────────────────┐
             │ AUTHENTICATION   │  who is this?
             │ valid session or │  reads cookie / token
             │ token?           │  ──▶ user 441, role student
             └────────┬─────────┘
                 fail │ 401 "I do not know you"
                      ▼ pass
             ┌──────────────────┐
             │ AUTHORIZATION    │  may THIS user do
             │ 1. role check    │  THIS, to THIS row?
             │ 2. ownership     │  role == admin?
             │    check         │  order.student_id == 441?
             └────────┬─────────┘
                 fail │ 403 "I know you, and no"
                      ▼ pass
             ┌──────────────────┐
             │ do the work      │
             └──────────────────┘
```

Every request passes two independent gates: the first establishes identity and returns `401` if it cannot, the second decides permission for this specific action on this specific row and returns `403` if the answer is no.

### Where it lives

Both gates live on the server, in front of every endpoint. That placement is the entire lesson, and it is M00's "the client cannot be trusted" applied to permissions.

Anything on the phone is decoration. Hiding a menu item, disabling a button, redirecting away from a route — all improve the experience and protect nothing, because the attacker is not using your UI. They are calling your endpoint.

Chapter 13 extends authentication: OAuth, SSO, passkeys, two-factor. Chapter 14 extends authorization, pushing the ownership check into the database itself so even a missed application check cannot leak rows. Chapter 15's signed URLs are authorization for files. M07 revisits all of it as OWASP's broken access control, consistently the most common serious vulnerability in real applications.

### Passwords: never stored, always hashed

If you store passwords, you will one day leak them, and your users reuse those passwords on their bank.

So you never store the password. You store a **hash** — the output of a one-way function. Given the password you can compute the hash; given the hash you cannot recover the password. At login you hash what was typed and compare the two hashes.

Three things make a hash safe for passwords, and all three are commonly missing from AI output.

**A salt.** A random value stored with each hash and mixed in before hashing. Without it, two users with the same password get the same hash, so an attacker can precompute hashes of common passwords once and look them all up. A per-user salt forces that work to be redone for every user.

**A slow algorithm, chosen for passwords.** `bcrypt`, `scrypt` or `argon2`. Deliberately slow — a few hundred milliseconds — so an attacker with your database tests a few thousand guesses a second instead of billions. `MD5` and `SHA-256` are fast general-purpose hashes and are *wrong* for passwords precisely because they are fast.

**A login response that does not reveal whether the email exists.** "Invalid email or password" for both cases, always.

The practical answer for your project: **do not build this.** Use your platform's authentication service. It gets salting, algorithm, rate limiting, reset and verification right, for free. You learn the mechanism so you can tell when something has got it wrong.

### Sessions or tokens

Once authenticated, the server must remember you across requests, because HTTP forgets everything between them. Two approaches.

A **session** stores the state on the server — a row saying "session abc123 belongs to user 441" — and gives the browser an opaque cookie. Every request sends the cookie and the server looks it up. Logging out deletes the row, so revocation is instant. The cost is a lookup per request and somewhere to keep sessions that is not server memory, which chapter 5 explained will not survive scaling.

A **JWT** is a signed token containing the claims themselves — user id, role, expiry — which the server verifies by checking the signature. No lookup required, which suits stateless hosting. The cost is that you cannot un-issue it: ban a user and their token works until it expires, because the server checks no list.

Which leads to the one thing you must know: **a JWT is signed, not secret.** Anyone holding it can read its contents — the payload is encoded, not encrypted — so never put anything private in it. And the signature must be *verified*, with your secret, on every request. Decoding without verifying means trusting whatever the caller wrote, including `"role": "admin"`.

Storage matters too. A token in `localStorage` is readable by any script on your page, so one bad dependency steals every login. An `httpOnly` cookie cannot be read by JavaScript at all, which makes it the better default for a web app.

### Words to own

| Term | Plain meaning | Example |
|---|---|---|
| Authentication | Proving who you are | logging in |
| Authorization | Deciding what you may do | admin only |
| 401 vs 403 | "I do not know you" vs "I know you, and no" | two different bugs |
| Hash | One-way transformation of a password | bcrypt output |
| Salt | Per-user random value mixed into the hash | stops precomputation |
| Session | Server-side record of who you are | row plus a cookie |
| JWT | Signed token carrying its own claims | readable by anyone |
| Claim | One fact inside a token | `role: student` |
| Ownership check | Confirming the row belongs to this user | `order.student_id` |
| httpOnly cookie | A cookie JavaScript cannot read | safest token store |

### Tradeoffs and gotchas

**The signature failure, stated plainly: a check on the page is not a check on the endpoint.** Your React route guard runs on the attacker's machine. Your endpoint runs on yours. Only one of those is a security control. Every protected endpoint needs its own check, written on the server, with no exceptions for "it's only reachable from the admin page".

**Role check and ownership check are two different checks.** `GET /api/orders/91` from a logged-in student is authenticated and role-appropriate — students may read orders — and still wrong if order 91 belongs to someone else. That bug has a name, Insecure Direct Object Reference, and it is found by changing a number in a URL. Every id-taking endpoint must verify the row belongs to the caller, or that the caller's role permits reading anyone's.

**Identity comes from the session, never from the request.** An endpoint that accepts `user_id` to decide whose data to return lets anyone pass any id.

**Never trust a decoded JWT.** Decoding is not verifying. A library call that reads claims without checking the signature against your secret will happily accept a token the attacker wrote in a text editor.

**Do not keep sessions in server memory.** Chapter 5's statelessness: when the platform runs a second instance, half your users are randomly logged out and nobody can reproduce it.

**Do not build your own password handling.** The list of things to get right — salting, algorithm, single-use expiring reset tokens, rate limiting, lockout, enumeration — is longer than your timeline. And rate limit the login endpoint, or an attacker simply tries ten thousand passwords. M05 covers how; this chapter is why.

### Verify it

This is the most important Verify section in the module. Do all six, by hand, on your own project.

1. **Log in as a normal user. Type the admin URL into the address bar.** If the admin page renders, you have the signature failure. Do this before anything else.
2. **Call the admin API endpoint directly as a normal user.** Not through the UI — use the browser dev tools or a request tool, with your student session. If it returns data instead of `403`, your page guard was the only protection and it protected nothing.
3. **Change the id in the URL.** Open one of your own records, then change the id to a teammate's. You must get `403` or `404`, never their data. Repeat for every endpoint that takes an id.
4. **Call every protected endpoint with no session at all.** Every one must answer `401`. Any that returns data is public, whatever your documentation claims.
5. **Send a forged token.** Change a claim in your own JWT — flip `role` to `admin` — and send it. If it works, the signature is not verified and nothing in your app is protected.
6. **Read the login handler's password path.** If it stores or compares a plain password, or uses `MD5` or `SHA-256`, replace it. Confirm a per-user salt and bcrypt, scrypt or argon2.

Red flags: a frontend route guard offered as the answer to "is this protected?"; an endpoint accepting `user_id` from the caller; `jwt.decode` where `jwt.verify` belongs; a session in a module-level variable; `401` and `403` used interchangeably; a login response that says "no such email"; "we can add role checks later"; a `users` table with a column named `password`.

## Walkthrough
1. In your own project, list every page that should not be visible to an ordinary user, and every endpoint that should not be callable by one. Write both lists down; the second is the one that matters.
2. Log in as an ordinary user. Type each restricted page's URL directly into the address bar and record what happens. Screenshot anything that loads.
3. Now attack the endpoints. With your ordinary-user session active, open dev tools and call each restricted endpoint directly. Record the status code each returns.
4. For every endpoint that returned data instead of `403`, write the line "endpoint X is unprotected" in a file. That file is your work list.
5. Open one of your own records through the UI and note its id. Change the id to a teammate's record and call the endpoint again. Record what comes back.
6. Log out completely. Call every protected endpoint with no session and record the status codes. Anything that is not `401` is public.
7. Find where identity is determined in your server code. Confirm it comes from the session or a verified token, and search the codebase for any endpoint reading `user_id` from a request body or query string.
8. If you use tokens, decode yours at a token inspection site and read the claims. Confirm nothing private is in there. Then modify a claim, send it, and confirm the server rejects it.
9. Read your login handler. Confirm you are using a platform authentication service, or that a per-user salt and bcrypt, scrypt or argon2 are present. Note the algorithm by name.
10. Brief AI to fix the endpoints on your work list, one at a time, with the explicit requirement that the check runs server-side and that both role and ownership are verified.
11. Re-run steps 3, 5 and 6 after each fix — a fix you have not re-tested is a claim — then write a decision-log entry listing every protected endpoint, its required role, and whether it needs an ownership check.

## Talk to AI
```prompt
CONTEXT:      Three-person student team. Our app has login working — users
              can sign in and see their name. We have never checked whether
              anything is actually protected, and we have learned that
              AI-built apps usually authenticate without authorizing.
              Product: [one sentence]
              Roles that exist or should exist: [e.g. student, staff, admin]
              How login currently works: [platform auth service / custom /
              social login — be specific]
              How identity is carried between requests: [cookie session /
              JWT / don't know]

              EVERY ENDPOINT WE HAVE, with what it does:
              [paste the list]

              THE SERVER CODE FOR OUR TWO MOST SENSITIVE ENDPOINTS:
              [paste them]

GOAL:         Find every endpoint an ordinary logged-in user can call that
              they should not be able to, and close each one on the server.

REQUIREMENTS: 1. For each endpoint, state the required role AND whether it
                 needs an ownership check on a specific row. Treat these as
                 two separate questions for every endpoint
              2. Tell me which of my endpoints are currently protected only
                 by the frontend hiding a link, and explain why that is not
                 protection
              3. Give me a numbered manual test plan I run myself: for each
                 endpoint, the request to send as a logged-out visitor, as
                 an ordinary user, and as an ordinary user asking for
                 someone else's row — with the exact status code I should
                 see in each case
              4. Tell me where identity must come from, and flag every
                 endpoint of mine that currently takes a user id from the
                 request instead
              5. If I use JWTs, tell me whether my code verifies the
                 signature or merely decodes the token, and show me how I
                 would forge a token to test it
              6. Review my password or session handling and name the exact
                 algorithm in use. If it is not bcrypt, scrypt or argon2, or
                 if there is no per-user salt, say so plainly
              7. Tell me which of 401 and 403 each failure should return,
                 and why mixing them up hides bugs

CONSTRAINTS:  Free tier only. Prefer my platform's authentication service
              over custom password code. Every check must run on the
              server. Do not store sessions in server memory, because our
              host scales to multiple instances.

NON-GOALS:    Do not add new features. Do not redesign the UI. Do not add
              OAuth providers, 2FA or passkeys yet. Do not write row-level
              security policies yet — that is the next chapter.

ACCEPTANCE:   - Logged out, every protected endpoint returns 401
              - As an ordinary user, every admin endpoint returns 403
              - As an ordinary user, requesting another user's row by id
                returns 403 or 404, never their data
              - A token with a modified role claim is rejected
              - No endpoint takes the caller's identity from the request

ASK:          Before proposing any fix, give me the manual test plan and
              tell me which of my endpoints you predict will fail it. I
              want to run the attack myself and see the failures before I
              see the patches.
```

## Stuck? Ask AI
```prompt
Explain the difference between authentication and authorization using a college canteen with a gate, a kitchen and a cash box. Then give me six situations from a food-ordering app and ask me which one each is, and whether it should return 401 or 403, without telling me the answers first.
```

```prompt
Explain why a route guard in my frontend code does not protect my data. Walk me through, step by step, exactly what an ordinary logged-in student would do from their own phone to read the admin data anyway, assuming the only check is in the UI.
```

```prompt
Explain what a salt does when hashing passwords, and why bcrypt is appropriate but SHA-256 is not, even though SHA-256 is considered a strong hash. Use numbers for how many guesses per second an attacker gets in each case.
```

## Quiz

### Q1 · L3
Google login works, the admin link is hidden from students, and a student who types `/admin` sees the admin dashboard. What is the defect?
- (a) The Google login integration is misconfigured
- (b) Authentication was built; authorization was not ✅
- (c) The admin link should be removed from the codebase entirely
- (d) The session cookie is missing the httpOnly flag
> Hiding a door is not locking it. The identity check passed, and no check ever asked whether this identity may open this page — the signature failure of AI-built apps.

### Q2 · L4
You fix the admin page so it redirects students away. Is the admin data now protected?
- (a) Yes — students can no longer reach the page
- (b) No — the endpoints are still callable directly by any logged-in user ✅
- (c) Yes, provided the redirect happens before the page renders
- (d) No, but only if the endpoints lack HTTPS
> The redirect runs on the attacker's machine. The endpoint runs on yours. Dev tools or a guessed URL reaches `GET /api/admin/orders` without touching your page at all.

### Q3 · L4
A logged-in student calls `GET /api/orders/91`, an order belonging to another student, and gets the data back. The role check passed because students may read orders. What is missing?
- (a) An index on the orders table
- (b) An ownership check that the row belongs to the caller ✅
- (c) A 401 response instead of a 200
- (d) Rate limiting on the endpoint
> Role and ownership are two separate checks. This is Insecure Direct Object Reference, found by changing a number in a URL, and every id-taking endpoint needs the second check.

### Q4 · L4
Why is SHA-256 the wrong choice for storing passwords even though it is a strong hash?
- (a) It produces hashes that are too short
- (b) It cannot be salted
- (c) It is fast, so an attacker with your database can test billions of guesses per second ✅
- (d) It is reversible with the right key
> Password hashing wants deliberate slowness. bcrypt, scrypt and argon2 take hundreds of milliseconds per attempt by design, cutting an attacker from billions of guesses a second to a few thousand.

### Q5 · L5
You change the `role` claim in your own JWT from `student` to `admin`, send it, and the admin endpoint returns data. What has this proved?
- (a) The token expired and was reissued with new claims
- (b) The server decodes the token without verifying its signature, so nothing in the app is protected ✅
- (c) JWTs should never contain a role claim
- (d) The token needed to be stored in an httpOnly cookie
> Decoding is not verifying. Without a signature check against your secret, the server trusts text the caller wrote, and every authorization decision in the app is made by the attacker.

## Go deeper
- [OWASP: Broken Access Control](https://owasp.org/Top10/A01_2021-Broken_Access_Control/)
- [OWASP Password Storage Cheat Sheet](https://cheatsheetseries.owasp.org/cheatsheets/Password_Storage_Cheat_Sheet.html)
