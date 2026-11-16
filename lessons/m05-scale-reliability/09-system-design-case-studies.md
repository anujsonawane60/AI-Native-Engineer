---
id: m05-09-case-studies
module: m05-scale-reliability
slug: system-design-case-studies
title: "System Design Case Studies: Shortener, Chat, Delivery, UPI"
order: 9
duration_min: 30
is_free: false
mission: false
volatility: low
last_reviewed: 2026-11-16
---

## Key learnings
- Run the five steps of a design in order: requirements, estimates, sketch, deep dive, tradeoffs
- Do a back-of-envelope estimate and conclude honestly that one small box is enough
- Separate two workloads in one product that need opposite treatment
- Name the single bottleneck of a design and what you would do at ten times the load
- Catch an AI that answers a 1,000-user problem with a 100-million-user architecture

## Lesson

### The pain

Somebody says "design a URL shortener" and the student starts drawing. Within two minutes there is a message queue, a sharded key-value store, three microservices and a service mesh. Asked how many links a day, they have no idea. Asked the bottleneck: "scaling".

The reverse happens at M11's checkpoints. A team has built a real product, it works, and when a mentor asks "what happens at ten times the traffic?" nobody can answer, because the architecture was never designed — it accumulated. They cannot name their bottleneck, so they cannot defend any choice in it.

Both failures skip the boring part. Design is not drawing boxes. It is a few questions in a fixed order, and the boxes fall out of the answers almost mechanically.

### The idea

**Requirements, then estimates, then a sketch, then one deep dive, then the tradeoffs you accepted. In that order, every time.**

**Requirements** split in two: functional, what it does, and non-functional, how fast, how fresh, how reliable, for how many people. The non-functional half is where every decision in this module gets made, and it is the half people skip.

**Estimates** are deliberately crude: requests per second, read-to-write ratio, bytes per year, rounded hard. The usual conclusion for a student project — say it out loud — is "this fits comfortably on one small database", which is a successful design, not a failed one.

**The sketch** is boxes and arrows for the happy path only. **The deep dive** is one part, chosen because the estimates called it the bottleneck. **The tradeoffs** are what you gave up, in your own words, which is what M18's defence asks for.

### The analogy

Designing the canteen itself, rather than queuing in it.

You do not start with the kitchen layout. You start with: how many students, in what window, eating what. Twelve hundred students, 70% between 1:00 and 1:40, mostly three dishes. From that one sentence the rest is arithmetic — four hundred plates in forty minutes, ten a minute, so three counters at four a minute plus one spare for the rush. The layout is a consequence of the numbers.

And the deep dive is the samosa warmer, because the arithmetic said the counter is the constraint and pre-made food is what makes a counter fast. You do not deep-dive the cutlery.

**Where the analogy breaks:** a canteen's capacity is set in concrete for ten years, so over-building is rational. Software changes next week, which inverts the economics — the cheapest design is the smallest one meeting today's numbers.

### Picture it

```
  REQUIREMENTS ──▶ ESTIMATES ──▶ SKETCH ──▶ DEEP DIVE ──▶ TRADEOFFS
   what it does     how much      happy      the one       what you
   how fresh        per second    path       bottleneck    gave up
   how reliable     per year      only
        │                │                                    │
        └──── if the estimates change the requirements ────────┘
```

The five steps run strictly left to right, and the only loop that matters is back to requirements when your arithmetic shows one was unaffordable.

### Case study: a link shortener

**Requirements.** Create a short code for a long URL; follow a code to its target. Non-functional: redirects must feel instant, say under 100 ms at p95; a code once issued must never point elsewhere; reads vastly outnumber writes.

**Estimates.** Say a busy campus service: 1,000 new links and 100,000 redirects a day. That is 0.01 writes a second and a little over 1 read a second, peaking perhaps ten times the average — about 12 reads a second. Five years at 1,000 links a day is 1.8 million rows; at 500 bytes each, under a gigabyte.

Say the conclusion plainly: **twelve reads a second and under a gigabyte fits on the smallest database you can rent.** Any design involving sharding here is wrong, and knowing that is the point of the estimate.

**Sketch.** A write endpoint storing `code → long_url`. A read endpoint that looks up the code and returns a redirect. One table, one unique index on `code`.

**Deep dive: the code itself.** Seven characters from a 62-character alphabet gives over three trillion possibilities, so random generation almost never collides — and a unique index plus one retry handles it when it does. The tempting alternative, an incrementing counter in base 62, gives shorter codes and makes every link guessable, so anyone can enumerate your database.

**The caching here is unusually clean.** A code never changes target, so the mapping is *immutable*, so there is no invalidation problem at all — the hardest part of Chapter 3 does not exist. Long TTL in the shared cache, a hit rate above 95% because a few links take most traffic, and a redirect in about 2 ms.

**Tradeoffs.** Immutability made this easy; add editable links and you import the whole invalidation problem. And use 302, not 301: a 301 is cached by browsers semi-permanently, so you lose click counts and can never fix a mistake.

### Case study: food delivery order tracking

A Swiggy-shaped problem, and a better teacher than the shortener: it holds two workloads that want opposite things.

**Requirements.** A student orders; watches status move through accepted, preparing, picked up, delivered; sees the delivery partner on a map. Non-functional: status must never be wrong, because money moved; the map may be a few seconds behind; it must work on slow 4G on a cheap Android phone.

**Estimates.** One city, 5,000 orders a day, 30% in the 8 pm hour — 1,500 an hour, under half an order a second. Trivial. Now the other workload: 200 delivery partners each sending a location every 5 seconds is 40 writes a second, continuously, all evening. That is **a hundred times the order traffic**, and it shapes the design.

**Sketch and the split.** Orders go in the database: low volume, durable, strongly consistent, audited. Locations go in the shared cache: one key per partner, overwritten every ping, short TTL, never in the orders table. Only the last location matters, so keeping history by default is millions of rows a day nobody reads.

```
 rider app ──location every 5 s──▶ shared cache  (1 key per rider,
                                                  overwritten, TTL 30 s)
 student app ──poll every 5 s────▶ app ──▶ cache (position)
                                      └─▶ database (order status)
 restaurant ──status change──────▶ app ──▶ database ──▶ notify student
```

Location pings overwrite one disposable cache key per rider while order status goes to the durable database, and the student's app reads one of each.

**Deep dive: how the student's screen updates.** Polling every five seconds is more requests but survives flaky 4G, reconnects for free, and is a day's work. A WebSocket sends fewer bytes and feels better, and costs connection handling, reconnect logic and a scaling concern. On budget Android phones on patchy networks, polling wins for a first version — a decision-log entry with a reason, not a compromise.

**Everything in this module appears here.** Status changes are strongly consistent because money moved; positions are eventual, bounded at five seconds. The payment webhook is retried by the provider, so it needs an idempotency key or you double-confirm orders. The location endpoint needs a per-rider rate limit, because a buggy rider app in a loop is the likeliest load you will see. A restaurant tablet going offline needs a timeout and a fallback, not an order stuck in "preparing".

**The other two, in a line each.** Chat's hinge is fan-out: one message must reach N devices, and the per-user unread count is harder than the message store. UPI's hinge is that you are not the source of truth — the bank is — so everything collapses into idempotency, reconciliation, and never trusting your own copy of a payment status.

### Where it lives

This chapter is the module's assembly point, and the method is M00's Director's Loop applied to architecture: FRAME is requirements, ARCHITECT is estimates and the sketch. M01's decision log is where tradeoffs land. M04 supplied schemas, queues and idempotency; M06 containers and load tests; M07 the metrics that tell you which estimate was wrong. M11's checkpoints and M18's defence are where you present this out loud, and M13 gives load testing and multi-region their own chapters.

### Words to own

| Term | Plain meaning | Example |
|---|---|---|
| Functional requirement | What it does | "Follow a short code" |
| Non-functional requirement | How well it must do it | "Under 100 ms at p95" |
| Back-of-envelope estimate | Deliberately crude arithmetic | 12 reads a second |
| QPS | Requests per second | 40 location writes |
| Read-write ratio | Reads per write | 100 to 1 |
| Immutable data | Never changes once written | A short code's target |
| Fan-out | One write reaching many readers | A chat message to 50 people |
| Source of truth | The system whose answer wins | The bank, for UPI |
| Happy path | The flow when nothing fails | Order placed, food arrives |
| Bottleneck | The one thing that gives out first | The location write rate |

### Tradeoffs and gotchas

**Designing for scale you will never see is the most common failure here.** Kafka, Kubernetes and sharding for a thousand users is not caution, it is a product that never ships. The correct response to an estimate of twelve requests a second is relief.

**Estimates come second for a reason.** They give you permission to be simple. Without them every suggestion sounds plausible and the loudest voice wins.

**Deep-dive exactly one thing.** Covering everything shallowly reads as not knowing which part is hard. Pick what your arithmetic flagged and go three levels in.

**Two workloads in one product usually want opposite designs.** Low-volume durable writes and high-volume disposable writes should not share a table, a consistency model or a rate limit. Noticing the split is most of the value of estimating.

**Name your bottleneck before anyone asks.** "At ten times this traffic the location writes break first, and the fix is batching pings to fifteen seconds" is a complete answer. Not having one is what fails a defence.

### Verify it

**Give the AI the requirements and withhold the architecture.** Ask for requirements, estimates with arithmetic shown, a sketch, one deep dive and the tradeoffs — in that order, and refuse to let it skip ahead. A design that arrives without numbers cannot be interrogated.

**Red flag: a 100-million-user architecture for your problem.** Queues, shards, microservices and a message bus for 5,000 orders a day. Ask: "at my estimated load, how much of this could I delete and still meet the requirements?" The honest answer is usually most of it.

**Red flag: no estimates, or estimates with no arithmetic.** "It will handle high traffic" is not an estimate. Make it show requests per second and bytes per year, and check the multiplication yourself — those are the numbers most likely to be confidently wrong.

**Red flag: no stated bottleneck.** Every design has one. If the AI cannot name what gives out first and at roughly what load, it has drawn a picture rather than designed a system.

**Red flag — the recurring ones: caching added to the fastest part,** such as a cache on the write path of a service doing 0.01 writes a second while the 40-per-second location endpoint is unprotected; and "a bigger instance" in place of noticing two workloads sharing a table.

**Red flag: two workloads not separated** — location pings and orders sharing a table, a consistency model and a rate limit.

Then test the design against reality rather than your own satisfaction. Load-test it, which is M06's job, and compare real numbers to your estimates — the gap teaches more than the design did. Ask a teammate who did not build it to explain the bottleneck back to you. And write the tradeoffs into your decision log now, because M18 will ask you to defend each one and "I don't remember why" is the worst available answer.

## Walkthrough
1. Pick one of the two systems this chapter did not walk through — group chat, or a UPI payment flow. You will design it in twenty minutes, on paper, before touching any AI.
2. Write the functional requirements as five short sentences. What a user can do. Nothing about technology.
3. Write the non-functional requirements as numbers: how many users, how fresh the data must be in seconds, what must never be wrong, what device and network they are on.
4. Do the arithmetic out loud. Users, actions per user per day, divide by 86,400, multiply by a peak factor of ten. Then bytes per record times records per year. Write both totals down.
5. Say your conclusion in one sentence, even if it is "this fits on one small database". Especially if it is.
6. Sketch the happy path only, in boxes and arrows, on paper. Label each arrow with the thing that travels, not the verb.
7. Identify the single bottleneck your arithmetic exposed, and deep-dive only that. For chat it is probably fan-out and unread counts; for UPI it is idempotency and reconciliation.
8. Write down three tradeoffs you accepted, each as "I chose X over Y because Z", with a number in Z.
9. Now give an AI only your requirements from steps 2 and 3, and ask for the same five steps. Compare its estimates to yours, line by line, and check its multiplication.
10. Note every place it proposed infrastructure your numbers do not justify. Ask it how much it could delete and still meet the requirements. Record what it removed.
11. Finally, run the same five steps on your own startup's riskiest feature, and put the output in your repo as a design note. This is the document M11 and M18 will ask you to defend.

## Talk to AI
```prompt
CONTEXT:      I am a student learning system design by directing AI rather
              than writing code. I want to practise the method, not
              receive an answer. The system to design:

              [Describe the system in 3 sentences. For example: a group
              chat for a college club; or a UPI payment flow for a canteen
              ordering app.]

              What I already know about scale:
              - expected users: [X]
              - actions per user per day: [X]
              - peak window: [e.g. 30% of daily traffic in one hour]
              - what must never be wrong: [list it]
              - what may be a few seconds stale: [list it]
              - devices and network: budget Android phones, slow 4G, India

GOAL:         A design I can defend out loud in five minutes, built in
              five ordered steps, with the arithmetic visible.

REQUIREMENTS: 1. Step one: separate functional from non-functional
                 requirements, and challenge any of mine that are vague
              2. Step two: back-of-envelope estimates with the arithmetic
                 shown - requests per second at peak, read-to-write ratio,
                 bytes stored per year. Show every multiplication
              3. Step three: a high-level sketch of the happy path only,
                 as text boxes and arrows, each arrow labelled with the
                 data that travels
              4. Step four: a deep dive into exactly ONE component, chosen
                 because the estimates identified it as the bottleneck.
                 Say why you chose it
              5. Step five: the tradeoffs, each written as "chose X over Y
                 because Z" with a number in Z
              6. Then tell me what gives out first at ten times my load,
                 and what I would change at that point - not now
              7. Finally, tell me how much of your own design I could
                 delete today and still meet my requirements

CONSTRAINTS:  Free tiers only. Assume a team of three students with no
              operations experience. Anything you propose must be
              buildable this term. Do not assume a budget.

NON-GOALS:    Do not propose Kubernetes, Kafka, microservices, sharding or
              a service mesh unless my estimates justify it - and if you
              think they do, show me the number that does. Do not give me
              a design before the estimates. Do not write code.

ACCEPTANCE:   - Every estimate shows its arithmetic and I can check it
              - The design has exactly one named bottleneck
              - Exactly one component is deep-dived, and you justified the
                choice from the numbers
              - Each tradeoff contains a number
              - You have told me what to delete

ASK:          Do the five steps strictly in order and stop after step two.
              Show me the estimates and your one-sentence conclusion about
              whether this needs anything more than a single small
              database. Wait for me before continuing.
```

## Stuck? Ask AI
```prompt
Teach me back-of-envelope estimation with five worked examples, each at a different scale, going from a college club app to a national service. For each, show every multiplication and finish with one sentence about whether it needs more than a single database. Then give me a sixth to try myself, and check my arithmetic without doing it for me.
```

```prompt
I designed a system and I think I over-built it. Here is my design: [paste it]. Do not improve it. Instead, tell me which components my stated numbers do not justify, what each one costs me in complexity, and what the smallest version that still meets my requirements would look like.
```

```prompt
Explain the fan-out problem in group chat with a concrete example of one message to fifty members, including what happens to unread counts. Then explain why the unread count is harder than storing the message, and give me two approaches with the tradeoff of each.
```

## Quiz

### Q1 · L3
In the five-step method, why do estimates come before the architecture sketch?
- (a) To make the design document longer
- (b) Because the numbers are what give you permission to be simple ✅
- (c) Because interviewers always ask for them first
- (d) Because they determine which programming language to use
> Without estimates every proposal sounds equally plausible. With them, "twelve reads a second" rules out sharding in one sentence and the design shrinks to something a small team can actually ship.

### Q2 · L3
A link shortener's cache is unusually easy to get right. Why?
- (a) Short codes are too small to cache incorrectly
- (b) A code's target never changes, so there is no invalidation problem ✅
- (c) Redirects bypass the cache entirely
- (d) Browsers cache redirects, so your cache does not matter
> Immutable data removes the hardest part of Chapter 3. Add editable links and you import the whole invalidation problem, which is exactly the tradeoff worth naming out loud.

### Q3 · L4
In the delivery design, 5,000 orders a day coexist with 200 riders pinging location every 5 seconds. What does the arithmetic tell you?
- (a) Orders are the bottleneck because money is involved
- (b) Location writes are roughly a hundred times the order traffic and shape the design ✅
- (c) Both workloads are equal and belong in one table
- (d) The design needs sharding from day one
> Under half an order a second against 40 location writes a second. Noticing that two workloads in one product want opposite treatment is most of the value of doing estimates at all.

### Q4 · L4
You ask an AI to design order tracking for 5,000 orders a day and it returns microservices, Kafka and a sharded store. What is the best next question?
- (a) Which cloud provider should I use?
- (b) At my estimated load, how much of this could I delete and still meet the requirements? ✅
- (c) Can you add a service mesh for observability?
- (d) How many instances of each service do I need?
> Over-building is the default failure mode of generated designs. Forcing a subtraction pass against your own numbers is the fastest way to get back to something shippable this term.

### Q5 · L5
Which answer best completes "at ten times my traffic, the thing that breaks first is…" for the delivery design?
- (a) The database's storage limit, because orders accumulate
- (b) The location write rate, and the fix is batching pings to a longer interval ✅
- (c) The CDN, because map tiles are large
- (d) Nothing, because the design is horizontally scalable
> Naming the bottleneck and the specific response to it is a complete answer. Orders grow slowly, map tiles are somebody else's problem, and "nothing breaks" is never true.

## Go deeper
- [The System Design Primer](https://github.com/donnemartin/system-design-primer)
- [Designing Data-Intensive Applications: chapter summaries](https://dataintensive.net/)
