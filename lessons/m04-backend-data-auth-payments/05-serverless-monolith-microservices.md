---
id: m04-05-serverless-monolith-microservices
module: m04-backend-data-auth-payments
slug: serverless-monolith-microservices
title: "Serverless Functions, Monoliths and Microservices"
order: 5
duration_min: 15
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

In week nine a three-person team reads an article about microservices and splits their working canteen app into five services: orders, menu, payments, users, notifications.

By day four there are five deployments, five sets of environment variables, an order with no matching user because the second write failed in another service, and a change that used to touch one file now touching three. They merge it back on day six, having lost the week they needed for the demo.

### The idea

**Your backend's shape should be chosen from your team size and your traffic, not from what large companies do at a thousand times your scale.**

A **monolith** keeps all features in one deployable application; **microservices** split them into separate applications that talk over the network; **serverless** is a hosting model where idle code wakes per request and then exits.

The first two answer "how do we divide the code?"; serverless answers "what runs it?" So a serverless monolith is a real option, usually the right one.

### The analogy

One kitchen is the monolith: the cook who makes dal also makes rice, and communication is a shout across two metres. Five stalls with their own kitchens, tills and stock are microservices — each rebuildable without closing the others, but a combo meal needs three to coordinate. Serverless is a kitchen with no permanent staff: a cook is summoned per order and leaves, so nothing idles overnight, though summoning takes a moment and the cook remembers nothing.

**Where the analogy breaks:** real stalls fail visibly; in software failures interleave invisibly. An order in one service with no matching user in another is a state no canteen can reach, and cleaning it up is the real cost of splitting.

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

This is M00's "server" box, zoomed in on arrangement and hosting. The recommendation here is deliberate: **build a monolith, host it on a free serverless platform, use one managed database** — one codebase your team can read, one place to look when it breaks, no idle cost. Anything that pushes against the serverless limits moves into a background job.

Downstream: chapter 15 holds files · chapter 16 holds slow work · M05 adds load balancers and pooling · M13 is where microservices become reasonable.

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

Serverless has three hard constraints plus one that appears only under load.

| Constraint | What it means | What it breaks |
|---|---|---|
| Cold start | Idle code loads before it runs: 200 ms to 3 s | the first student of the lunch rush waits |
| Timeout | One execution is killed, ~10 s on free tiers | image resize, PDF, 5,000-row loop |
| Stateless | Nothing survives between invocations | counters, disk files, in-memory sessions |
| Connections | Each warm instance holds its own: 200 vs 60 | database refuses new connections at peak |

**Microservices trade a code problem for a network problem, and the network problem is harder.** In a monolith, two features calling each other is a function call: instant, all-or-nothing. Across services it is an HTTP request that can fail halfway — so you owe retries, timeouts, idempotency and state repair forever.

**The real cost of splitting is human.** Three people cannot own five services: nobody reviews anyone else's work, and a change needs three deployments in the right order. Keep each feature's folders and data access in one place and splitting later stays cheap.

**AI proposes the architecture from the article, not for your project.** An event bus and a database per service, for a three-person four-month project, is pattern-matching on blog posts. Restate team size and timeline as hard constraints.

### Verify it

Five checks on any proposed architecture.

1. **Count the deployments.** More than one for a three-person project needs a written answer to "what breaks if we keep it together?"
2. **Ask where slow work goes.** A slow operation inside a request, on a host that times out at ten seconds, is already broken.
3. **Look for in-memory state.** A variable holding a count, cache, session or queue across requests breaks once a second instance exists.
4. **Ask about pooling by name.** "At 200 concurrent invocations, how many connections exist, and what is my limit?" No answer means solo tests pass, peak fails.
5. **Test a cold start.** Leave the app idle twenty minutes, load it, time the first request.

Red flags: "scalable" with no number near it · a database per service for a student project · a design that cannot run on one laptop · an event bus with no users · sessions in server memory, a scaling bug and chapter 12's.

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
