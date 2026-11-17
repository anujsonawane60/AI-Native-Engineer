---
id: m14-14-gateways-cost
module: m14-deeper-ml-agents
slug: ai-gateways-and-cost-control
title: "AI Gateways, Semantic Caching and Cost Control"
order: 14
duration_min: 15
is_free: false
mission: false
volatility: high
last_reviewed: 2026-11-16
---

## Key learnings
- Put one gateway in front of every model call and move keys, routing and budgets into it
- Tell apart the provider's prompt cache, your exact-match cache and a semantic cache
- Set a semantic-cache threshold from a measured false-hit rate instead of guessing
- Enforce per-user and per-feature budgets that return a refusal rather than a warning
- Catch a cache key missing the permission scope, which is a leak rather than a bug

## Lesson

### The pain

M09 chapter 14 taught cost per request from measured tokens, M10 chapter 6 the provider's prompt cache, M05 cache layers and rate limiting. New here is where any of it is *enforced*.

Count what calls a model in your product by now: a hosted API, a cheap classifier, a fine-tuned endpoint, a local build, four agents, an eval suite. Six callers, four credentials, keys in the app's environment. So one loop spends ₹4,100 overnight with no per-user ceiling, a provider's bad hour takes you down with no fallback, and the bill cannot be attributed.

### The idea

**One gateway: a single internal service every model call passes through, where keys, routing, budgets, caching, retries and accounting all live.**

Six jobs: **one API shape** over many providers, so a switch is configuration; **key custody**, so app code holds no credential; **routing and fallback**; **budgets** per user, tenant and feature, enforced by refusing; **caching**, in the two forms you own; and **accounting**, every request tagged with feature, user, release id, tokens and rupees.

### The analogy

Six counters, one store room, each with its own key. Nobody knows what the dosa counter consumed this month, and a helper with a key can empty a shelf at 2 am. Put one issue window on the store room: one key, every request written against the counter that asked, a counter past its allowance told no.

**Where it breaks.** The window can refuse forty kilos on judgement. Your gateway sees only tokens and a price, so it cannot tell a heavy user from a runaway loop.

### Picture it

```
  app code · orchestrator · eval suite · background jobs
        │        │              │             │
        └────────┴──────┬───────┴─────────────┘
                        ▼
        ┌───────────────────────────────────────────┐
        │              GATEWAY                      │
        │  1 keys (app holds none)                  │
        │  2 exact-match cache ──▶ hit? return      │
        │  3 semantic cache (guarded) ──▶ hit?      │
        │  4 budget: user · tenant · feature        │
        │       over? ──▶ refuse (429)              │
        │  5 route: cheap ──▶ expensive; on error   │
        │       or 429 ──▶ fallback provider        │
        │  6 log: feature · user · release id ·     │
        │       tokens · ₹ · cache · latency        │
        └───┬───────────┬───────────┬───────────────┘
            ▼           ▼           ▼
       hosted API   fine-tuned   local quantized
                    endpoint     model
```

Every caller reaches models only through one gateway, which holds the keys, checks its two caches, enforces budgets by refusing, routes and falls back between providers, and writes one accounting line.

### Three caches, and only two are yours

**The provider's prompt cache** is M10 chapter 6's discount on an unchanged prefix. You do not operate it; the gateway just keeps that prefix byte-identical.

**Your exact-match cache** stores an answer against a hash of the request, so it can never answer a different question. The difficulty is the key: model and prompt versions, document versions, locale, and **the permission scope**. Omit the last and user B gets an answer built from user A's documents — an incident, not a stale cache.

**A semantic cache** returns a stored answer above a similarity threshold: higher hit rate, categorical risk.

### Making a semantic cache safe

"Cancel order 4821?" and "cancel order 4812?" have near-identical embeddings and different correct answers. That is every semantic-cache incident.

Four rules. **Never semantic-cache text containing an identifier, quantity, date or name.** **Set the threshold by measurement:** a hundred labelled query pairs, fifty wanting the same answer and fifty that only look similar, then the threshold holding the false-hit rate at zero on the negatives. **Scope the key by user and permission.** **Log every hit with both queries.** A hit saves one request; a false hit is a confident wrong answer, so it sets the threshold.

### Cost control that actually controls

Four enforcements, all at the gateway, all refusing rather than logging.

- **A per-user daily budget** returning a 429 with a message the product can show — what would have prevented the ₹4,100 night.
- **A per-feature monthly budget** with a kill switch, so one experiment cannot eat the month.
- **A maximum output cap** per feature; unbounded output is the commonest surprise bill.
- **A cascade:** cheap model first, expensive only for hard requests — also where a rate limit becomes a fallback.

Plus one alert that pages a person mid-spend.

### Tools today

Managed AI gateway products exist and churn quickly, as does every provider's price page. Two things are stable enough to build on: the OpenAI-shaped request format has become the common language most providers and open serving engines accept, which makes it the right internal shape; and your own thin gateway is often better for a student team, because this is the one component whose behaviour you must fully understand.

### Where it lives

- M07 chapter 2, M05 chapters 2 to 4 — secret custody, caches, rate limits
- M09 chapter 14, M10 chapter 6 — the arithmetic enforced here, the cache it must not break
- Chapters 7, 13 — what is routed to, and the release id on every line
- M17 chapter 6 — per-feature cost as margin, answerable only with a feature tag

### Words to own

| Term | Plain meaning | Example |
|---|---|---|
| Gateway | One internal service all model calls pass through | Six jobs, one place |
| Key custody | Credentials held only by the gateway | App holds none |
| Fallback | A second provider used when the first fails | On error or 429 |
| Cascade | Cheap model first, expensive only when needed | Classifier, then model |
| Exact-match cache | Stored answer against a hash of the request | Safe, modest hits |
| Semantic cache | Stored answer reused for a similar question | High hits, real risk |
| Similarity threshold | How close counts as the same question | Set by measurement |
| False hit | A cached answer returned for a different question | 4821 vs 4812 |
| Cache key scope | Everything that changes the correct answer | Includes permissions |
| Per-user budget | A daily ceiling enforced by refusing | Returns 429 |
| Output cap | Maximum tokens a response may use | Per feature |
| Feature tag | The label attributing cost to a product area | Most valuable field |

### Tradeoffs and gotchas

**A gateway is a single point of failure, so keep it thin.** Timeouts and a circuit breaker belong here; anything that could slow it does not.

**Caching the wrong thing is worse than not caching.** Nothing personalised, permissioned or carrying an identifier — the boring repeated questions are both safe and most of your traffic.

**Measure the false-hit rate before the savings.** Teams ship at a guessed threshold, celebrate a 40% hit rate, and learn the wrong-answer rate from a complaint.

**Do not let the gateway rewrite prompts.** A timestamp above the stable prefix destroys the provider's cache for every feature at once.

### Verify it

The red flag is the module's bluntest: a cost nobody measured.

1. **Ask for last month's cost by feature.** If it cannot be produced, the feature tag is missing.
2. **Ask what a thousand requests in an hour from one user gets:** a refusal at a stated number.
3. **Ask where the provider keys are.**
4. **For a semantic cache, demand the labelled pairs and the measured false-hit rate.**
5. **Read the cache key field by field.** A missing permission scope is a data leak.
6. **Ask when the fallback was last exercised.**

Red flags: no per-feature attribution; a budget that only warns; keys outside the gateway; a guessed threshold; no permission scope in the key; an untested fallback.

## Walkthrough
1. List every place in your product that calls a model, including background jobs, the eval suite and each agent. Write down which credential each one uses. The length of this list is the argument for the chapter.
2. Build the thinnest gateway that works: one internal endpoint, one request shape, the keys held only there. Point one caller at it and confirm that caller's own key can be deleted.
3. Add the accounting line: feature, user id, release id, model and version, input and output tokens, rupees, cache status, latency. Then produce yesterday's spend grouped by feature.
4. Move every remaining caller behind the gateway and delete every provider key from application environments. Confirm by searching your codebase and your deployment settings.
5. Add a per-user daily budget and prove it: send requests from one test user until you are refused. Check the refusal is a 429 with a message your product can display.
6. Add a per-feature monthly budget and a kill switch. Turn one feature off through configuration alone, with no deploy, and confirm it stops calling the model.
7. Add an output cap per feature and measure what it does to your p95 cost per request.
8. Add the exact-match cache. Write the key out field by field, including the permission scope, then test the leak case: two users with different permissions asking the identical question must not share an answer.
9. Build the semantic-cache safety set: a hundred labelled query pairs, fifty that want the same answer and fifty that look similar but do not — include at least ten pairs differing only by an order number or a date. Sweep the threshold and record the hit rate and the false-hit rate at each value.
10. Choose the threshold with a zero false-hit rate on your fifty negatives, enable the semantic cache only for queries containing no identifiers, numbers or dates, and log both queries on every hit.
11. Add routing — a cheap model or classifier first, the expensive model only for hard requests, and a fallback provider on error or rate limit — then run chapter 10's regression suite against the fallback path and record whether any category dropped. Finally, audit twenty semantic-cache hits by hand and write down how many were wrong. That number, not the hit rate, decides whether the cache stays.

## Talk to AI
```prompt
CONTEXT:      I am a student on an AI-native engineering program. I direct AI
              and verify its output; I do not write code by hand. I already
              know how to compute cost per request and cost per active user
              from token counts, how the provider's prompt cache works and
              what breaks it, and how rate limiting and cache layers work. Do
              not re-teach any of that. My team of three runs [product in two
              sentences] with [number] real users in India. Things that call a
              model today: [list every one, including background jobs, each
              agent and the eval suite]. Providers and models in use: [list].
              Where the keys live today: [answer honestly]. Last month's AI
              bill: ₹[number], and I [can / cannot] say which feature caused
              what. Revenue per paying user per month: ₹[number].

GOAL:         One thin gateway in front of every model call, with key custody,
              per-feature cost attribution, enforced per-user and per-feature
              budgets, a tested fallback, and caching that cannot return an
              answer to a different question.

REQUIREMENTS: 1. Design the thinnest gateway that does the six jobs: keys,
                 caching, budget checks, routing and fallback, accounting, and
                 one request shape. Tell me what must NOT be in it
              2. Give me the exact accounting line to log per request, and
                 show me the query that produces last month's cost grouped by
                 feature
              3. Give me a per-user daily budget design that refuses with a
                 429 and a displayable message, and tell me what number to set
                 given my revenue per user
              4. Give me a per-feature monthly budget with a kill switch that
                 works without a deploy
              5. Write out my exact-match cache key field by field for MY
                 product, including the permission scope, and give me the test
                 that proves two users with different permissions cannot share
                 an answer
              6. For a semantic cache: tell me which of my query types must
                 never be semantically cached and why, then give me a
                 procedure to build 100 labelled query pairs including at
                 least ten that differ only by an identifier or a date
              7. Tell me how to sweep the similarity threshold and record hit
                 rate against false-hit rate, and tell me which number decides
                 the threshold
              8. Design the cascade and the fallback: which model decides
                 first, when the expensive one is used, and what happens on a
                 provider error or rate limit. Tell me how to test the
                 fallback and what to re-run against it
              9. Tell me every way this gateway could accidentally destroy my
                 provider's prompt cache, and how I verify it has not
             10. Tell me which alert should page a person during the night and
                 at what threshold

CONSTRAINTS:  Free tier only. Plain English for a second-language reader.
                Label every command READ-ONLY or CHANGES-THINGS. The gateway
              must add only a few milliseconds and must not become a single
              slow point. No provider key may remain in application code or
              any frontend. Treat every provider, price and gateway product
              name as stale and tell me to check it today.

NON-GOALS:    Do not re-teach cost-per-request arithmetic or the provider's
              prompt cache. Do not recommend a paid gateway product before
              describing the thin version I could own. Do not cache anything
              personalised, permissioned or containing an identifier. Do not
              put retry logic in my application code.

ACCEPTANCE:   - Every caller goes through the gateway and no provider key
                remains in application code
              - Last month's cost can be grouped by feature
              - A test user is refused at a stated daily number with a 429
              - One feature can be turned off without a deploy
              - My cache key includes the permission scope and I proved two
                differently-permissioned users cannot share an answer
              - A 100-pair labelled set exists and I have a false-hit rate
                measured at my chosen threshold
              - Queries containing identifiers, numbers or dates bypass the
                semantic cache entirely
              - The fallback path has been exercised and my regression suite
                run against it
              - Prompt-cache hit rate is unchanged or better after the gateway

ASK:          First, look at my list of callers and my key locations and tell
              me which single thing here is an incident waiting to happen.
              Then tell me whether my last month's bill is answerable per
              feature with what I log today. Answer both before designing.
```

## Stuck? Ask AI
```prompt
Explain the difference between the provider's prompt cache, my own exact-match answer cache and a semantic cache, using one concrete request from [describe your AI feature]. For each, tell me who operates it, what it keys on, what the hit rate looks like, and what the worst thing that can go wrong is. Then tell me which of the three I should build first and which I should probably never build for my product.
```

```prompt
I want a semantic cache. Here are ten real queries from my product: [paste them]. For each, tell me whether it is safe to semantically cache and why. Then build me twenty labelled query pairs from these — ten that should share an answer and ten that look similar but must not, including some differing only by an order number or a date. Tell me exactly how to use them to pick a similarity threshold, and which measurement decides it.
```

```prompt
Here is my cache key: [paste every field it includes]. Tell me, field by field, whether it is sufficient. Specifically: could two users with different permissions receive each other's answers? Could a stale retrieved document version be served after the document changed? Could a prompt change go unnoticed? For each gap, tell me the field to add and what incident it prevents.
```

## Quiz

### Q1 · L3
Which cache do you operate yourself and key on a hash of the whole request?
- (a) The provider's prompt cache
- (b) An exact-match answer cache ✅
- (c) A semantic cache
- (d) The browser cache
> The provider's prompt cache discounts an unchanged prefix and is not yours to operate. An exact-match answer cache stores a full response against a request hash, which is why it can never answer a different question.

### Q2 · L4
"Can I cancel order 4821?" returns the cached answer for "can I cancel order 4812?". What rule was broken?
- (a) The cache TTL was too long
- (b) Queries containing an identifier must never be semantically cached ✅
- (c) The embedding model was the wrong one
- (d) The cache was not scoped per user
> Two order numbers produce near-identical embeddings and different correct answers. Identifiers, quantities, dates and names must bypass the semantic cache entirely.

### Q3 · L4
How should a semantic cache's similarity threshold be chosen?
- (a) By the default the library recommends
- (b) By raising it until the hit rate stops improving
- (c) From a labelled pair set, picking the threshold with no false hits on the negatives ✅
- (d) By asking the model to judge similarity
> A false hit is a confidently wrong answer, which costs more than any token saving. So the threshold is decided by the false-hit measurement on deliberately-similar negative pairs, not by the hit rate.

### Q4 · L4
Your cache key omits the permission scope. What is the consequence?
- (a) A slightly lower hit rate
- (b) Stale answers after a document changes
- (c) One user can receive an answer computed from documents another user was allowed to see ✅
- (d) The provider's prompt cache stops working
> This is a permission leak, which is an incident rather than a bug. The key must contain everything that changes the correct answer, and who is asking changes it.

### Q5 · L5
One user spends ₹4,100 overnight. Which gateway control would have prevented it?
- (a) A semantic cache with a high hit rate
- (b) A per-user daily budget that refuses with a 429 when exhausted ✅
- (c) A cheaper model in the cascade
- (d) An alert on the monthly spend
> Only a refusal stops money being spent. A cascade reduces the per-request cost of a runaway, a cache may not be hit at all, and a monthly alert arrives after the night is over.

## Go deeper
- [Anthropic: rate limits, usage tiers and spend controls](https://docs.anthropic.com/en/api/rate-limits)
- [Cloudflare: what an AI gateway does — caching, routing and observability](https://developers.cloudflare.com/ai-gateway/)
