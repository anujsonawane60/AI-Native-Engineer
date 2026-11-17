---
id: m11-05-checkpoint-3
module: m11-ship-your-mvp
slug: checkpoint-3-auth-data-ai
title: "Checkpoint 3 · Auth, Data and One AI Feature, With Tests"
order: 5
duration_min: 25
is_free: false
mission: true
volatility: low
last_reviewed: 2026-11-16
---

## Key learnings
- Pass the week-three gate: real accounts, real roles, real data, one AI feature
- Prove authorization is enforced on the server, not hidden in the interface
- Show your AI feature behaving when the model is slow, wrong or unavailable
- Put a cost ceiling and a rate limit on anything that calls a model
- Demonstrate tests that run in CI and that genuinely can fail

## Lesson

### Where this sits in the calendar

**Checkpoint 3 is due at the end of week three of the Sprint.** Checkpoint 1 was the problem, the PRD and the diagram. Checkpoint 2 was one path, deployed, with the data really saved. This week the product stops being a skeleton: it has people in it, it has a shape for their data, and it has the thing that makes it an AI product rather than a web page.

### The pain

A group demos on day twenty-one: a login, a staff-only menu, a "suggest a meal for ₹50" button, twenty-two passing tests. But the staff menu is hidden by an `if` in the page, so a student who types the URL gets it; the key sits in the frontend, so it is in git history and every visitor's browser; and when the model is slow the button spins forever. None of the tests touch any of it.

### The idea

**Week three is where a product acquires the three things that can hurt someone: other people's accounts, other people's data, and a bill that grows while you sleep. Each needs a server-side check and a behaviour when it fails.**

A permission enforced in the interface is not enforced, and a feature with no failure state is not a feature.

### The analogy

The stall has grown: a till with money, a storeroom with a lock, and a coffee machine with a sign telling staff what to do when it fails. Two people hold the key, and it is checked at the door, not by how someone looks.

**Where the analogy breaks:** a lock is physical; you can see whether it is locked. A permission check is invisible, and the interface looks identical whether it lives on the server or only in the page. You test it as the wrong user, deliberately.

### Picture it

```
  student's phone                 your server                database
  ┌─────────────┐   request   ┌─────────────────┐  query  ┌──────────┐
  │ taps a      │────────────▶│ 1 who is this?  │────────▶│ users    │
  │ staff-only  │             │   (session)     │         │ orders   │
  │ URL directly│             │ 2 may they?     │◀────────│ menu     │
  │             │◀────403─────│   (role check)  │         └──────────┘
  └─────────────┘             │ 3 recompute the │
                              │   price here    │
                              └────────┬────────┘
                                       │ prompt, with a timeout,
                                       │ a cost ceiling, a rate limit
                                       ▼
                                ┌──────────────┐
                                │ model API    │
                                │ (key lives   │
                                │  ONLY here)  │
                                └──────────────┘
```

Every request passes three gates on the server — who is this, may they, and what is the real price — and the model key exists only on the server, behind a timeout and a ceiling.

### What must exist by the end of week three

**1. Real accounts** — a login creating a durable session, not a name in a text box. M04.12.

**2. Two roles, enforced on the server** — in the endpoint or middleware, not the page. An ordinary user who asks for a privileged URL is refused with a status code. M04.14.

**3. A schema with one relationship, and one migration applied to existing rows**, with the result written down. M04.7, M04.11.

**4. One AI feature, live, called from your server**, its key in environment settings and never in git or a browser. M10.

**5. Its failure states, visible** — a timeout, a truthful message, a way to carry on. M10.14.

**6. A cost ceiling and a rate limit**, per user and overall, with the arithmetic for a hundred users. M09.14, M05.4.

**7. Three tests in CI** — one on authorization, one on a failure state, all going red when broken. M06.10, M06.12.

**8. A security pass, written down** — no secret in git history, inputs validated on the server, a paragraph on untrusted text reaching your prompt. M07.1, M10.7.

**9. Two more dated decision-log entries** — six in total, on the way to eight.

### Where it lives

This is where your product becomes something you are responsible for: a stranger's account and data are inside it, which is why M07.9's consent material stops being theoretical.

**Each of you submits your own evidence.** Whoever did not build the auth still demonstrates it failing for the wrong user, on their own device.

### Words to own

| Term | Plain meaning | Example |
|---|---|---|
| Authentication | Proving who someone is | Logging in |
| Authorization | Deciding what they may do | Only staff change prices |
| Session | The durable proof of a login | A signed cookie |
| Role | A label that grants permissions | `student`, `staff` |
| Migration | A recorded change to stored data shape | Adding a required column |
| Backfill | Filling the new column for existing rows | 400 old orders get a default |
| Cost ceiling | A hard cap on spend per user and overall | 20 calls a day per user |
| Rate limit | A cap on requests in a time window | 5 per minute, then HTTP 429 |
| Timeout | Giving up on a slow call deliberately | 10 seconds, then a message |
| Prompt injection | Untrusted text changing your instructions | A review that says "ignore the above" |

### Tradeoffs and gotchas

**Never let the model key reach the browser.** Any key shipped to a client is public, and your server is the only place a ceiling applies.

**Choose the cheap model first** — the cheapest that passes your own examples, with its cost per call measured (M09.13).

**Keep your prompt in git.** A prompt edited in a dashboard is a change nobody reviewed, and Checkpoint 4's eval set needs something stable.

### Verify it

Run these yourself.

1. **As an ordinary logged-in user, request a privileged URL, then the endpoint behind it.** If either succeeds, the permission lives in the interface and does not exist.
2. **Change an id to another account's** — M11.4's eleven-character bug, tested not read.
3. **Search your whole git history for your keys**, and open the network tab while the AI feature runs. A key that appears in either is public.
4. **Make the model fail three ways:** timeout, wrong key, unusable output.
5. **Hit the rate limit**, then do the arithmetic for a hundred users.
6. **Break each test's subject** and confirm it goes red.
7. **Apply the migration on a copy with real rows** and look at the old rows.

Red flags: permissions only in the interface; a key a browser can see; a spinner with no timeout; tests that pass with the feature removed. Every one demos perfectly.

## Walkthrough
1. Brief the auth work as a whole feature using M11.3: a plan of five to eight steps, with the role check in the first half, and a dedicated step for failure states.
2. Build accounts and sessions first, with no roles at all. Verify a login survives a reload and a logout really ends the session.
3. Add the role column as its own step, then the server-side check as its own step. Do not add any privileged page until the check exists.
4. Now add one privileged page. Test it as an ordinary logged-in user by typing the URL, then by calling the endpoint directly without the page.
5. Design the relationship your product needs and apply one migration to the rows your Checkpoint 2 users already created. Write down what happened to those rows.
6. Brief the AI feature separately, as its own feature with its own plan. The key goes in the host's environment settings before the first call is ever made.
7. Write the prompt yourself, in a file in the repository, and record in `DECISIONS.md` why it is shaped that way and which model you chose from M09.13.
8. Add the timeout, the per-user cap and the overall cap in the same step as the first working call, not afterwards. Then compute the cost at a hundred users.
9. Force all three model failures — timeout, error, unusable output — and give each a truthful message and a way to continue.
10. Write three tests: the wrong user is refused, one failure state behaves, and the core path works. Put them in CI, then break each subject in turn and confirm each goes red.
11. Run the security pass — search git history for keys, check the network tab, list every untrusted input that reaches a prompt — then run all nine verification checks, record pass or fail, and add two dated decision-log entries.

## Talk to AI
```prompt
I am at the end of week three of a startup sprint. My product now has real accounts, roles, a schema and one AI feature, and I have to prove it is genuinely finished rather than merely demonstrable. Interrogate it. Do not write code and do not tell me it looks fine.

WHAT EXISTS
Stack and hosting: [framework, host, database, all free tier]
Auth: [how login works, where the session lives, how long it lasts]
Roles: [name them, and say exactly which file or layer checks the role]
Privileged pages and endpoints: [list them]
Schema: [paste the tables, columns and relationships]
Migration applied this week: [what changed, and what happened to rows that already existed]
AI feature: [what it does, which model, where the key is stored, what the prompt is]
Limits: [timeout, per-user cap, overall cap, rate limit]
Tests in CI: [what each one asserts]
Untrusted text that reaches my prompt: [user input, retrieved documents, anything from outside]

Work through these in order:

1. AUTHORIZATION. Based on where I said the role is checked, tell me whether an ordinary logged-in user could reach each privileged thing by typing the URL, or by calling the endpoint directly without the page. Name every one you are unsure about and tell me the exact manual test for it.
2. OTHER PEOPLE'S DATA. Given my schema, list every place where changing an id in a request might return a different user's data. For each, tell me what the server must check.
3. SECRETS. Given where I said the key lives, could it reach a browser? What else in my setup is likely to be a secret that I have not named?
4. AI FAILURE STATES. List every way my AI feature can fail - slow, errored, rate-limited, unusable output, refusal, empty input, enormous input - and for each, tell me what my user currently sees and what they should see.
5. COST. With my caps, what is the worst a single user could spend in a day, and what does the feature cost at 100 users and at 1,000? Show the arithmetic. Then tell me which cap is missing.
6. PROMPT INJECTION. For each untrusted input I listed, describe the sentence an attacker would insert and what it would make my feature do. Then tell me the cheapest defence for each.
7. MIGRATION. Given what I changed, what should I check on the rows that existed before it ran? What would be irreversible?
8. TESTS. For each test I described, tell me whether it would still pass if the feature it covers were deleted. Name the one implementation line I should break by hand to prove each one can fail.

Then give me a numbered list of the nine checks I should run myself, in the order most likely to find a real problem, each written as something I can do by hand in under three minutes.

Finally: name the one thing about this product that would embarrass me in front of a real user next week.
```

## Mission

**Objective.** Real accounts with roles enforced on the server, a schema with a migration applied to existing rows, one AI feature with its failure states and its caps, and three tests in CI that genuinely can fail.

**Steps.**
1. **Plan the auth** — as a whole feature from M11.3, role check in the first half, a step for failure states.
2. **Accounts, then roles, then the check** — three separate steps, in that order, each verified before the next.
3. **One privileged page** — then attack it yourself as an ordinary user, by URL and by endpoint.
4. **Migrate** — one real change to stored data shape, applied to the rows your week-two users created.
5. **AI feature** — separate plan, key in environment settings before the first call, prompt in a file in git.
6. **Caps** — timeout, per-user cap, overall cap, rate limit, in the same step as the first working call.
7. **Break the model three ways** — timeout, error, unusable output. Each gets a truthful message.
8. **Three tests in CI** — authorization, a failure state, the core path. Break each subject; watch each go red.
9. **Security pass** — git history searched for keys, network tab checked, prompt-injection paragraph written.
10. **Verify** — all nine checks, pass or fail recorded. Two dated decision-log entries.

**Upload.**
- The live URL → `demo_url`; the repository → `repo_url`
- A screen capture of an ordinary logged-in user being refused a privileged URL, and a second of the endpoint being refused directly
- A screen capture of an attempt to read another account's data being refused
- Your schema, plus the migration and a note on what happened to the rows that already existed
- Your prompt file, and the decision-log entry naming the model you chose and why
- Screen captures of all three AI failure states as the user sees them
- Your rate limit refusing you, and your cost arithmetic at 100 users
- A screenshot of CI green, plus one of each test going red when you broke its subject
- Your written security pass, including the prompt-injection paragraph
- Your nine verification checks with pass or fail per item
- Reflection, 150 words max → which of the nine checks failed the first time you ran it, and what you fixed

**Self-check.** Tick every box before you submit.
- [ ] Login creates a session that survives a reload, and logout really ends it
- [ ] There are at least two roles, and the check runs on the server
- [ ] An ordinary logged-in user is refused a privileged URL
- [ ] That same user is refused when calling the endpoint directly, with a status code
- [ ] Changing an id to another account's does not return their data
- [ ] My schema has at least one real relationship
- [ ] One migration has been applied to rows that already existed, and I looked at them
- [ ] The model key is in environment settings and has never been in git or a browser
- [ ] I checked the network tab and no key or raw model endpoint appears
- [ ] The AI feature has a timeout, and I have seen it time out
- [ ] All three model failure states show a truthful message and a way to continue
- [ ] There is a per-user cap and an overall cap, and I hit one on purpose
- [ ] I have written the cost per call and the cost at 100 users
- [ ] My prompt is in a file in the repository, not only in a dashboard
- [ ] Three tests run in CI, and each went red when I broke its subject
- [ ] No secret appears anywhere in git history, including old commits
- [ ] I have written one paragraph on what untrusted text reaches my prompt
- [ ] `DECISIONS.md` has two new entries, dated this week, six in total
- [ ] I personally demonstrated the wrong-user refusal on my own device
- [ ] This submission is mine: my own evidence, my own reflection

**Stretch.** Write down what your AI feature costs per call, then try to halve it without making the output worse — a shorter prompt, a cheaper model, a cached result from M05.3, or fewer retrieved chunks. Measure both versions on the same five inputs and write 100 words on what you traded. Keep those five inputs: they are the beginning of Checkpoint 4's eval set.

## Stuck? Ask AI
```prompt
I think my permission check might only exist in my user interface rather than on my server. Ask me where the check happens, file by file, and then tell me exactly how to test it as an ordinary logged-in user without using my own interface at all - including how to call my endpoint directly from a terminal. Do not fix anything until I have run that test and told you the result.
```

```prompt
My AI feature works when the model answers quickly and does nothing sensible otherwise. List every way a model call can fail - slow, errored, rate-limited, refused, empty, unusable, too long - and for each one ask me what my user currently sees. Then help me decide, for each, what the honest message and the fallback should be. Do not write code yet.
```

```prompt
Explain the difference between authentication and authorization using the canteen storeroom as the example, then explain why a hidden button is not a permission. Finally, give me four ways a logged-in ordinary user could reach a page I thought was staff-only, in the order they would try them.
```

## Quiz

### Q1 · L4
A staff-only page is hidden by a check in the page code, and the endpoint behind it does not ask who is calling. What have you built?
- (a) Authorization that needs a second check for defence in depth
- (b) No authorization at all — the interface is only a convenience for honest users ✅
- (c) Authentication without a session
- (d) A rate-limiting problem
> Anyone can type the URL or call the endpoint with a tool. The server is the only place a decision is real, which is why this gate tests the endpoint directly rather than the page.

### Q2 · L4
Why must the model API key stay on your server rather than in the browser?
- (a) Browsers cannot make requests to model APIs
- (b) Anything shipped to a client is public, and the server is the only place you can cap usage ✅
- (c) Model providers require server-side calls by contract
- (d) It makes the feature respond faster
> A key in client code is visible in the network tab whatever wraps it, and a leaked key is somebody else spending your free tier. The server is also where the timeout, the rate limit and the ceiling live.

### Q3 · L4
Your AI button spins forever on slow college wifi. What is the minimum fix at this gate?
- (a) Choose a faster model
- (b) Add a loading animation so the wait feels shorter
- (c) A timeout, a truthful message, and a way to carry on without the feature ✅
- (d) Retry the call until it succeeds
> A feature without a failure state is not a feature. Retrying without a limit turns one slow service into your outage and your bill.

### Q4 · L3
Why does this gate insist the migration be applied to rows that already exist?
- (a) Empty tables cannot accept migrations
- (b) A new required column is harmless on an empty table and breaks every existing row ✅
- (c) CI will not run against an empty database
- (d) The host charges for migrations on empty tables
> This is the data-shape question from M01.8's review list, made real. It is cheap this week against the rows your week-two users created, and expensive once there are real users.

### Q5 · L5
Your group has twenty-two passing tests, none of which covers authorization or a failure state. What does the suite actually tell you?
- (a) That the product is well tested but under-documented
- (b) That the code runs under conditions the author chose ✅
- (c) That coverage is high enough for this gate
- (d) That the agent understood the requirements
> Happy-path tests measure that nothing crashed. Three tests that go red when you break their subject — one of them on the wrong user — are worth more than twenty that cannot fail.

## Go deeper
- [OWASP Authorization Cheat Sheet](https://cheatsheetseries.owasp.org/cheatsheets/Authorization_Cheat_Sheet.html)
- [OWASP Top 10 for LLM Applications](https://owasp.org/www-project-top-10-for-large-language-model-applications/)
