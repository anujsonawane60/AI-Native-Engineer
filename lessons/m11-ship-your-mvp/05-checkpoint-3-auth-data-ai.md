---
id: m11-05-checkpoint-3
module: m11-ship-your-mvp
slug: checkpoint-3-auth-data-ai
title: "Checkpoint 3 · Auth, Data and One AI Feature, With Tests"
order: 5
duration_min: 35
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

Everything here was taught elsewhere. Accounts, sessions, roles and row-level rules are M04.12–M04.14. Schema and migrations are M04.7 and M04.11. Rate limiting is M05.4. Tests and CI are M06.10 and M06.12. Secrets, OWASP and prompt injection are M07.1–M07.3 and M10.7. Model choice and token cost are M09.13 and M09.14. The AI feature is M10. This chapter is the gate.

### The pain

A group demos at the end of week three and it looks strong. There is a login. Staff see a staff menu; students do not. There is a "suggest a meal for ₹50" button that calls a model and writes a sensible paragraph. There are twenty-two passing tests.

Three things are true that the demo does not show. The staff menu is hidden by an `if` in the page, so a student who types the URL directly gets it, and the endpoint behind it never asks who is calling. The model's API key was put in the frontend so the button would work, and it is now in the repository's history and in every visitor's browser. And when the model is slow — which on college wifi it often is — the button spins forever, because nobody asked what should happen if it does not answer. None of the twenty-two tests touch any of that; they test that the page renders and that a valid order is accepted.

The demo was honest. The product is not finished. This gate exists to find the gap between those two sentences while there is still a week left.

### The idea

**Week three is where a product acquires the three things that can genuinely hurt someone: other people's accounts, other people's data, and a bill that grows while you sleep. Each needs a check on the server and a behaviour when it fails.**

A feature without a failure state is not a feature. A permission enforced in the interface is not enforced. A model call with no ceiling is an open tap.

### The analogy

The stall has grown. There is a till with money in it, a storeroom with a lock, a ledger of what was sold, and a new coffee machine that is wonderful and fails about one time in twenty.

The rules change with the growth. Only two people hold the storeroom key, and the key is checked at the door rather than by asking whether someone looks like staff. The ledger is in pen, so last week's entries still mean something. And a handwritten sign is taped to the coffee machine: *if it does not start in thirty seconds, switch it off, refund the customer, tell the owner.* That sign is what makes the machine usable by someone who is not its designer.

**Where the analogy breaks:** a lock on a storeroom door is physical, so you can see whether it is locked. A permission check is invisible, and the interface will look identical whether the check exists on the server or only in the page. You cannot see this one. You have to test it as the wrong kind of user, deliberately, which is why that test is a line item at this gate.

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

**1. Real accounts.** A sign-up and a login that create a durable session. Not a name in a text box. From M04.12.

**2. At least two roles, enforced on the server.** One ordinary user, one elevated. The check lives in the endpoint or the middleware, not in the page. An ordinary logged-in user who requests a privileged URL or calls the endpoint directly is refused with a status code. From M04.14.

**3. A schema with at least one relationship, and one migration applied to existing rows.** Users to orders, orders to items — something joined. Plus one real change to the shape of stored data, applied to rows already there, with what happened to them written down. From M04.7 and M04.11.

**4. One AI feature, live, called from your server.** The key is in the host's environment settings and has never been in the repository or the browser. You wrote the prompt and can explain its shape. From M10.

**5. That AI feature's failure states, visible.** What the user sees when the model is slow, returns something unusable, or is unavailable. At minimum a timeout, a truthful message, and a way to carry on without it. From M10.14.

**6. A cost ceiling and a rate limit on the model call.** A cap per user and a cap overall, plus the arithmetic for what the feature costs at a hundred users. From M09.14 and M05.4.

**7. At least three tests in CI, green on every push.** One must cover authorization — the wrong user is refused. One must cover a failure state. All three must go red when you break what they test. From M06.10 and M06.12.

**8. A security pass, written down.** No secret anywhere in git history. Every input validated on the server. One paragraph on prompt injection: what untrusted text reaches your prompt, and what you did about it. From M07.1–M07.3 and M10.7.

**9. Two more decision-log entries, dated this week** — six in total, on the way to eight by Checkpoint 4.

### Where it lives

This is the gate where your product becomes something you are responsible for. Three weeks ago nobody could lose anything by using it; now a stranger's account and a stranger's data are inside it, which is what DPDP is about and why M07.9's consent material stops being theoretical. Next week M11.6 puts it on a real domain with monitoring, and M11.7 puts ten real people through it.

**Each of you submits your own evidence.** The three roles from M01.1 excuse nobody here: whoever did not build the auth still has to demonstrate it failing for the wrong user, on their own device, in their own submission. M18's defence is individual, and "my teammate did that part" is the one answer that cannot pass it.

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

**A hidden button is not a permission.** The interface is a convenience for honest users. Anyone can type a URL or call your endpoint with a tool, and the server is the only place a decision is real. This is M01.8's highest-value review question and the most common defect at this gate.

**Never let the model key reach the browser.** Any key shipped to a client is public — it is in the network tab, whatever wraps it. Model calls go through your server, which is also the only place a limit or a ceiling can be applied.

**Choose the cheap model first and measure.** M09.13's point in practice: start with the cheapest model that passes your own examples, measure what it costs per call, then decide whether a better one is worth it. Starting with the most capable model and economising later means you never learn what the feature actually needs.

**Apply the migration to real rows, not an empty table.** A new required column is harmless on an empty database and breaks every existing row in production. Do it this week, on the rows your Checkpoint 2 users created, and write down what happened to them.

**Your prompt is part of your product, so put it in git.** A prompt edited live in a dashboard is a change nobody reviewed, cannot revert and cannot explain. Keep it in a file, change it through a pull request, and Checkpoint 4's eval set will have something stable to measure.

### Verify it

Nine checks. Run them yourself; do not accept a summary.

1. **Log in as an ordinary user and request a privileged URL directly.** You must be refused. Then call the endpoint behind that page directly, bypassing the page entirely. You must be refused again, with a status code. If the second one succeeds, your permission lives in the interface and does not exist.
2. **Log out and request the same URL.** You must be sent to the login, not shown a blank version of the page.
3. **Read someone else's data on purpose.** Change an id in a URL or a request to one that belongs to another account. You must be refused. This is the eleven-character bug from M11.4, tested rather than read.
4. **Search your whole git history for your keys**, not just the current files. The first six characters of each is enough to search for. A key that was ever committed must be rotated.
5. **Open the browser's network tab while using the AI feature.** If the model key or the raw model endpoint appears there, the key is public.
6. **Make the model fail three ways.** Set the timeout to one second so it times out. Point it at a wrong key so it errors. Feed it input that produces something unusable. Each must produce a truthful message and a way to carry on.
7. **Hit the rate limit on purpose** and confirm you get a refusal rather than a bill. Then do the arithmetic for a hundred users and write the number down.
8. **Break each of your three tests' subject in turn** and confirm each goes red. Then restore and confirm green.
9. **Apply the migration on a copy with real rows in it** and look at those rows afterwards. Not the new ones — the old ones.

The red flags here: a feature with no failure state; permissions enforced only in the interface; a model key anywhere a browser can see it; a spinner with no timeout; tests that pass with the feature removed; a prompt that exists only in a dashboard; and a migration only ever run on an empty table. Every one of them demos perfectly.

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
