---
id: m04-05-serverless-monolith-microservices
module: m04-backend-data-auth-payments
slug: serverless-monolith-microservices
title: "Serverless Functions, Monoliths and Microservices"
order: 5
duration_min: 25
is_free: false
mission: false
volatility: medium
last_reviewed: 2026-11-16
---

## Key learnings
- Choose a shape for your backend and defend it with your team size and your traffic
- Explain cold starts, timeouts and statelessness, and what each one breaks
- Say why microservices make a three-person team slower, not faster
- Name the things serverless free tiers genuinely cannot do
- Check that AI's architecture suggestion matches where your code will actually run

## Lesson

### The pain

A team builds the canteen app as one application. It works. In week nine someone reads an article about microservices and splits it into five services: orders, menu, payments, users, notifications.

Day one of the split: nothing can talk to anything. Day two: there are five deployments, five sets of environment variables, five free-tier accounts, and a bug where an order exists but the user does not, because the two writes happened in two services and the second one failed. Day four: a change that used to touch one file now touches three services and cannot be tested locally at all. Day six: they merge it all back and have lost a week they needed for the demo.

A different failure, same week, different team. They host everything as serverless functions on a free tier. A student uploads a photo of the canteen menu and the function dies at ten seconds because image processing takes fourteen. Another function opens a database connection on every single call, and at lunch peak the database refuses new connections because 300 functions are each holding one. Both problems are invisible on a laptop with one user.

### The idea

**Your backend's shape should be chosen from your team size and your traffic, not from what large companies do at a thousand times your scale.**

A **monolith** is one application, one deployment, one codebase. All the features live together.

**Microservices** split features into separate applications that deploy independently and talk over the network.

**Serverless functions** are a hosting model rather than a shape: your code sits idle, a request wakes a copy of it, it runs for a few seconds, and it disappears. You pay for execution, not for a machine that waits.

These are not three points on a line from bad to good. Monolith and microservices answer "how do we divide the code?" Serverless answers "what runs it?" You can have a serverless monolith, and for a small team that is often exactly the right answer.

### The analogy

One canteen with one kitchen is the monolith. The cook who makes dal also makes rice. Communication is a shout across two metres. When it gets busy, everyone is in one room and it is chaotic, but nothing is ever waiting on a form.

Five separate food stalls, each with its own kitchen, its own till and its own stock, are microservices. Each can be rebuilt without closing the others. Each can hire its own staff. But a combo meal now needs three stalls to coordinate, the student queues three times, and if the rice stall shuts the thali is impossible.

Serverless is a kitchen with no permanent staff. A cook is summoned when an order arrives, cooks it, and leaves. You pay per dish and nothing idles overnight. The catch: summoning takes a moment — the first student after a quiet hour waits while the cook walks in — and the cook remembers nothing from the last dish, so anything that must be remembered has to be written down somewhere outside the kitchen.

**Where the analogy breaks:** real stalls fail independently and a student can see which one is closed. In software, the failures interleave invisibly. An order that exists in one service with no matching user in another is a state no physical canteen can reach, and cleaning it up is the real cost of splitting things.

### Picture it

```
 MONOLITH               MICROSERVICES          SERVERLESS
 ┌──────────────┐    ┌──────┐ ┌──────┐      ┌─────────────┐
 │ orders       │    │orders│ │ menu │      │ /api/orders │
 │ menu         │    └───┬──┘ └───┬──┘      │ wakes, runs,│
 │ users        │        │ HTTP   │         │ dies        │
 │ payments     │    ┌───┴──┐ ┌───┴──┐      ├─────────────┤
 └──────┬───────┘    │users │ │ pay  │      │ /api/menu   │
        │            └───┬──┘ └───┬──┘      │ wakes, runs,│
        ▼                ▼        ▼         │ dies        │
   ┌─────────┐      ┌────┐ ┌────┐ ┌────┐    └──────┬──────┘
   │   one   │      │ db │ │ db │ │ db │           ▼
   │database │      └────┘ └────┘ └────┘      ┌─────────┐
   └─────────┘                                │database │
                                              │ (pooled)│
  1 deploy          4 deploys, 4 db,          └─────────┘
  1 place to look   network between all       0 idle cost,
                                              cold starts
```

One application with one database, four applications each with their own, and a set of functions that wake per request — three answers to who owns what, with different numbers of places a change can go wrong.

### Where it lives

This is the "server" box from M00's four boxes, zoomed in on how it is arranged and hosted.

For this program, the recommendation is specific and deliberate: **build a monolith, host it on a free serverless platform, and use one managed database.** One codebase your whole team can read, one deployment in M06, one place to look when something breaks, and no idle cost.

Everything that pushes against serverless limits — the fourteen-second image resize, the nightly report, the email that takes four seconds to send — moves out into a background job, which is chapter 16. That one pattern removes almost every reason a small team thinks it needs long-running servers.

M05 picks up what happens when one instance is not enough: stateless servers, load balancers, replicas and connection pooling. M13 is where microservices become a reasonable conversation, because by then you will have a reason.

### What serverless actually costs you

Three constraints, and they are not negotiable by prompting harder.

**Cold starts.** A function with no recent traffic has to be loaded before it runs. On free tiers that is commonly a few hundred milliseconds and occasionally a few seconds. Your canteen app at 11 a.m. is cold; the first student of the lunch rush waits. This is survivable and worth knowing before you call it a bug.

**Timeouts.** Free tiers typically stop a function after about ten seconds. Anything slower — resizing a photo, generating a PDF, calling a slow third-party API, looping over 5,000 rows — must not live in a request handler. It gets queued.

**No memory between calls.** Each invocation starts fresh. A counter in a variable resets. A file written to disk is gone. An in-memory session store loses every login the moment the platform scales to a second copy. Everything that must persist goes to the database or to object storage from chapter 15. This property is called being **stateless**, and in M05 it is also what makes scaling possible at all.

A fourth, less obvious one: **database connections**. Each warm function instance may hold its own connection, and 200 instances at lunch can exhaust a free-tier database's limit of, say, 60. The fix is a connection pooler, which most managed database providers offer as a separate connection string. If AI never mentions it, your app will work in testing and fail at peak.

### Words to own

| Term | Plain meaning | Example |
|---|---|---|
| Monolith | One application containing all features | your project |
| Microservice | One feature as its own deployable app | a payments service |
| Serverless function | Code that wakes per request and then exits | an API route |
| Cold start | The delay before idle code can run | 200 ms to 3 s |
| Timeout | The hard limit on one execution | ~10 s on free tiers |
| Stateless | Keeps nothing in memory between calls | required to scale |
| Connection pooling | Sharing database connections across callers | a pooler URL |
| Distributed transaction | One change spanning two services | hard; avoid |
| Service boundary | The line you split features along | orders vs payments |
| Background job | Slow work moved out of the request | chapter 16 |

### Tradeoffs and gotchas

**Microservices trade a code problem for a network problem, and the network problem is harder.** In a monolith, two features calling each other is a function call: instant, reliable, and either both happen or neither. Across services it is an HTTP request that can be slow, can fail halfway, and can succeed while the caller times out and retries. You need retries, timeouts, idempotency and a way to repair half-finished state. That is M05 and chapter 17 of this module, as a tax on every interaction, forever.

**The real cost of splitting is human, not technical.** Three people cannot own five services. You will end up with one person per service, nobody able to review anyone else's work, and a change that requires three deployments in the right order. Monoliths are not a compromise at this size; they are correct.

**"We will split it later" is fine, and splitting later is much easier than you fear** — provided you kept clean module boundaries inside the monolith. Organise folders by feature, keep each feature's data access in one place, and the eventual split is a refactor rather than a rewrite.

**Serverless plus WebSockets is usually a mistake on a free tier.** Chapter 3's point, restated here: a connection that must stay open for an hour is a poor fit for a host that bills per short invocation.

**Beware the architecture that matches the article rather than the project.** If AI proposes an event bus, a service mesh and separate databases per service for a three-person four-month student project, it is pattern-matching on blog posts. The correct response is to restate your team size and timeline as a hard constraint.

**Free tiers change.** Specific limits in this chapter are illustrative. Read your platform's current limits page before you design against a number.

### Verify it

Five checks on any architecture AI proposes.

1. **Count the deployments.** More than one for a three-person project needs a written justification that survives the question "what breaks if we keep it together?"
2. **Ask where the slow things go.** Name your slowest operation — image resize, report, bulk email — and ask directly whether it runs inside a request. If the answer is yes and your host times out at ten seconds, the design is already broken.
3. **Look for in-memory state.** Search for any variable that holds a count, a cache, a session or a queue across requests. On a stateless host it is a bug that appears only under load, when a second instance exists and the two disagree.
4. **Ask about connection pooling by name.** "At 200 concurrent invocations, how many database connections will exist, and what is my database's limit?" A design with no answer will fail at lunch peak and pass every test you run alone.
5. **Test a cold start yourself.** Leave the app alone for twenty minutes, then load it and time the first request. Know your number rather than discovering it during a demo.

Red flags: the word "scalable" without a number anywhere near it; separate databases per service proposed for a student project; a design that cannot be run end to end on one laptop; "we'll use an event bus for communication between services" in a product with no users yet; any proposal to keep sessions in server memory, which is both a scaling bug and chapter 12's problem.

## Walkthrough
1. Write down three numbers for your own project: team size, weeks remaining, and your honest peak concurrent users. These three numbers decide the architecture, and everything else is decoration.
2. List every feature in your product as a one-line bullet. This is your candidate list of services, and you are about to decide not to split it.
3. For the two features that look most separable, write what would have to happen if one of them were a separate service: how it would be called, what happens if that call fails halfway, and who on your team of three would own it.
4. Find your slowest operation. Time it, roughly. Anything over three seconds is a candidate for chapter 16's queue rather than a request handler.
5. Look up the current limits page for the platform you plan to deploy on. Write down three numbers: maximum execution time, free-tier invocations per month, and anything about concurrent executions.
6. Look up your database's connection limit on its free tier, and find out whether it offers a pooled connection string. Write both down.
7. Deploy whatever you have, or a trivial endpoint, and leave it untouched for twenty minutes. Then load it and measure the first response time. That is your cold start.
8. Load it again immediately and measure. The gap between the two numbers is what cold start costs your users.
9. Search your codebase for any module-level variable that holds state between requests. For each one found, decide where it should actually live: the database, object storage, or nowhere.
10. Write your architecture decision in the scope document as one paragraph: the shape you chose, the three numbers that chose it, and the specific event that would make you reconsider.

## Talk to AI
```prompt
CONTEXT:      Three-person student team, [N] weeks remaining, free tiers
              only. Nobody has run a production system before.
              Product: [one sentence]
              Features: [list them as bullets]
              Honest peak concurrent users: [your number]
              Planned hosting: [platform name, or "undecided"]
              Planned database: [name, or "undecided"]
              Our slowest operation is [name it] and it takes about
              [seconds] seconds.

GOAL:         Decide the shape of our backend — one application or several,
              and what hosts it — and know in advance which free-tier
              limits we will hit.

REQUIREMENTS: 1. Recommend a shape and justify it using my three numbers:
                 team size, weeks remaining, peak users
              2. List every operation in my feature list that will not fit
                 inside a 10-second request handler, and say where each
                 should go instead
              3. Tell me every piece of state that must not live in server
                 memory if my host scales to multiple instances
              4. Do the connection arithmetic: at my peak, how many
                 database connections could exist, what my free tier allows,
                 and whether I need a pooled connection string
              5. Tell me what a cold start will cost my first user after an
                 idle period, and whether that is acceptable for my product
              6. Describe how I would keep module boundaries clean inside
                 one application so that splitting later is a refactor
                 rather than a rewrite

CONSTRAINTS:  Free tier only. One deployment unless you can justify more
                 against my team size. No paid managed services. Must be
                 runnable end to end on a single laptop. Must work for
                 users on slow 4G.

NON-GOALS:    Do not propose microservices, an event bus, a service mesh or
              per-service databases. Do not write any code. Do not design
              the database schema. Do not set up CI.

ACCEPTANCE:   - Your recommendation cites my team size and my week count
              - Every operation over 3 seconds has a named home outside the
                request path
              - The connection arithmetic is shown, with both numbers
              - You name at least one thing your recommendation makes
                harder later, and the signal that would tell me to change

ASK:          First give me two options with tradeoffs — the simplest
              possible thing, and the next step up — then your
              recommendation and the single number that decided it. No code.
```

## Stuck? Ask AI
```prompt
Explain what "stateless" means for a server, using a canteen with no permanent staff as the example. Then give me six things a web app might want to remember and ask me, for each one, where it must actually be stored if the server keeps nothing in memory. Do not tell me the answers first.
```

```prompt
My function times out after ten seconds when a student uploads a large photo. Explain why this is a design problem rather than a limit I should try to raise, and describe in plain language what the request handler should do instead so the user still gets immediate feedback.
```

```prompt
An AI suggested I split my three-person student project into five microservices with separate databases. Argue against that as strongly as you can, naming the specific extra work each split creates. Then tell me the one condition under which the suggestion would have been right.
```

## Quiz

### Q1 · L2
What does "stateless" mean for a serverless function?
- (a) It cannot write to a database
- (b) It keeps nothing in memory between invocations ✅
- (c) It has no access to environment variables
- (d) It can only handle one user at a time
> Each call starts fresh. Counters in variables reset, files on disk vanish, and in-memory sessions disappear — so anything that must persist goes to a database or object storage.

### Q2 · L3
Your free-tier host times out a request after about ten seconds, and resizing an uploaded canteen photo takes fourteen. What is the right fix?
- (a) Ask the platform to raise the timeout
- (b) Resize the image in the browser instead and trust the result
- (c) Accept the upload, queue the resize as a background job, respond immediately ✅
- (d) Split the resize across two function calls that run in sequence
> Slow work does not belong in a request handler. Chapter 16's queue is the standard answer: acknowledge fast, do the work out of band, update the record when it finishes.

### Q3 · L4
A three-person team with eight weeks left splits their working app into five microservices. What is the most likely outcome?
- (a) Faster development, because each person owns a service
- (b) Better reliability, because services fail independently
- (c) A week lost to deployment, configuration and half-written state across services ✅
- (d) Lower free-tier cost, because each service is smaller
> The cost of splitting is human and operational: five deployments, five configs, no local end-to-end run, nobody able to review anyone else's work, and failures that leave one service's data out of step with another's.

### Q4 · L4
Your app works perfectly in testing and starts failing at lunch peak with database connection errors. What is the most likely cause on a serverless host?
- (a) The database ran out of disk space
- (b) Many warm function instances each hold a connection and exceed the free-tier limit ✅
- (c) The cold start delay is compounding
- (d) The requests are arriving out of order
> Concurrency multiplies connections. A pooled connection string is the standard fix, and an architecture proposal that never mentions pooling will pass every single-user test and fail under load.

### Q5 · L5
Which combination is the right default for a three-person team on free tiers, and why?
- (a) Microservices on serverless, so each feature scales independently
- (b) A monolith on a single always-on paid server, so nothing ever cold-starts
- (c) A monolith on serverless with one managed database, slow work queued ✅
- (d) Serverless functions with session state held in server memory for speed
> One codebase everyone can read, one deployment, one place to look when it breaks, no idle cost — and the only hard constraint, the ten-second timeout, handled by moving slow work to a background job.

## Go deeper
- [Martin Fowler: Monolith first](https://martinfowler.com/bliki/MonolithFirst.html)
- [PostgreSQL docs: connections and resource consumption](https://www.postgresql.org/docs/current/runtime-config-connection.html)
