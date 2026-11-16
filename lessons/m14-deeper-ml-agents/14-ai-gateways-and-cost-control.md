---
id: m14-14-gateways-cost
module: m14-deeper-ml-agents
slug: ai-gateways-and-cost-control
title: "AI Gateways, Semantic Caching and Cost Control"
order: 14
duration_min: 30
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

M09 chapter 14 taught you to compute cost per request and per active user from measured token counts, and gave the five levers in order. M10 chapter 6 taught the provider's prompt cache. M05 taught cache layers and rate limiting. None says where any of it is *enforced*, and by now you have a problem of shape rather than arithmetic.

Count what calls a model in your product after this module: a hosted API for the assistant, a cheap model for classification, a fine-tuned endpoint, a local build, an orchestrator with four agents, and an eval suite. Six callers, four credentials, three SDKs, keys in the application's environment.

Then three failures arrive together. One user, or one loop, spends ₹4,100 in a night, because nothing enforced a per-user ceiling. A provider has a bad hour and your product is down, because every call goes to one place with no fallback. And when the bill arrives nobody can say which feature caused it. Switching providers means editing fourteen files.

### The idea

**One gateway: a single internal service every model call passes through, where keys, routing, budgets, caching, retries and accounting all live.**

It does six jobs. **One API shape** over many providers, so a switch is configuration. **Key custody**, so application code never holds a credential. **Routing and fallback**, including a cheap-to-expensive cascade. **Budgets** per user, tenant and feature, enforced by refusing. **Caching**, in the two forms you own. And **accounting**: every request tagged with feature, user, release id, tokens and rupees.

None of the six is hard. The point is that they are in one place, which is the only way any of them is reliably true.

### The analogy

The canteen has six counters and one store room. Each counter holds its own key, sends its own helper in, and takes what it needs.

Nobody knows what the dosa counter consumed this month. When the rice runs out, every counter discovers it separately. A helper with a key can empty a shelf at 2 am unnoticed. And when the supplier changes, six counters must be told.

Put a single issue window on the store room instead. One person holds the key. Every request is written down against the counter that asked. A counter that has used its day's allowance is told no. If the usual rice is out, the window substitutes. One place to change the supplier, one ledger explaining the bill.

**Where the analogy breaks.** The issue window can see a counter asking for forty kilos and refuse on judgement. Your gateway sees a token count and a price and nothing else — it cannot tell a heavy user from a runaway loop. That is why the budget must be a hard number set in advance, and why the alert must fire during the night rather than in a report.

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

**The provider's prompt cache** is M10 chapter 6's: a discount on an unchanged prefix. You do not operate it. Your gateway's job is to keep the stable prefix byte-identical, never injecting a timestamp or a per-user string above the rules.

**Your exact-match answer cache** stores a full answer against a hash of the request. It can never answer a different question, so it is safe, and its hit rate is modest — good for popular identical questions, menus and policy lookups. The difficulty is the key: it must include everything that changes the correct answer — the model and version, the prompt version, the retrieved document versions, the locale, and **the permission scope**. Omit the last and user B receives an answer computed from documents user A was allowed to see, which is an incident rather than a stale cache.

**A semantic cache** embeds the incoming query, searches previous ones, and returns a stored answer if similarity passes a threshold. The hit rate is far higher, because people ask the same thing in different words. The risk is categorical rather than occasional.

### Making a semantic cache safe

Consider "can I cancel order 4821?" and "can I cancel order 4812?" Their embeddings are almost identical and their correct answers differ. That is the shape of every semantic-cache incident.

Four rules follow. **Never semantic-cache anything containing an identifier, a quantity, a date or a name** — route those to the exact-match cache or straight through. **Set the threshold from measurement**: build a hundred labelled query pairs, fifty that genuinely want the same answer and fifty that look similar and do not, then pick the threshold keeping the false-hit rate at zero on the second fifty and accept whatever hit rate remains. **Scope the key by user or tenant and by permission.** And **log every hit** with both queries, because a false hit is otherwise invisible.

The economics are then honest. A hit saves one request's cost. A false hit produces a confidently wrong answer, which costs more than any number of tokens. So the threshold is set by the second number.

### Cost control that actually controls

Four enforcements, all at the gateway, all returning a refusal rather than writing a log line.

**A per-user daily budget** in rupees or tokens, returning a 429 with a clear message, using M05 chapter 4's machinery. This is the control that would have prevented the ₹4,100 night.

**A per-feature monthly budget** with a kill switch, so one experiment cannot consume the month.

**A maximum output cap** per feature. Unbounded output length is the commonest source of a surprise bill.

**A cascade**: a cheap model or classifier decides first, and only hard requests reach the expensive model. M09 chapter 13 called this the professional answer; the gateway is where it is implemented, and also where a rate limit becomes a fallback rather than an error.

Beside all four, one alert: spend crossing a fraction of the monthly budget, or any request exceeding a multiple of your median cost, paging a person while it happens. M07 chapter 7 is where that lives.

### Tools today

Managed AI gateway products exist and churn quickly, as does every provider's price page. Two things are stable enough to build on: the OpenAI-shaped request format has become the common language most providers and open serving engines accept, which makes it the right internal shape; and your own thin gateway is often better for a student team, because this is the one component whose behaviour you must fully understand.

### Where it lives

Below: M07 chapter 2 for secret custody, M05 chapters 2 to 4 for cache layers and rate limiting, M09 chapter 14 for the arithmetic this enforces, and M10 chapter 6 for the provider cache it must not break. Chapter 7 supplies the things being routed to; chapter 13 supplies the release id stamped on every line.

Above: M17 chapter 6 is where per-feature cost attribution becomes margin, and it is only answerable if the gateway tagged every request with a feature. That tag is the most valuable field in this chapter.

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

**A gateway is a single point of failure, so keep it thin.** A few milliseconds, no clever logic. Timeouts and a circuit breaker from M05 chapter 8 belong here; anything that could make the gateway itself slow does not.

**Caching the wrong thing is worse than not caching.** Anything personalised, permissioned, time-dependent or containing an identifier should not be answer-cached. What is worth caching is the boring repeated questions — also most of your traffic.

**Measure the false-hit rate before the savings.** Teams ship a semantic cache at a guessed threshold, celebrate a 40% hit rate, and learn the wrong-answer rate months later from a complaint. The labelled pair set is an afternoon and it is the safety argument.

**A budget that logs is not a budget.** The only acceptable behaviour on exhaustion is refusing the call with a message the product can show. Anything else records money already spent.

**Fallback must be tested, and feature tags must exist from day one.** A fallback to a different model changes quality, so run chapter 10's suite against that path too. And retrofitting feature attribution across six callers is tedious enough that nobody does it — without it, M17's margin question has no answer.

**Do not let the gateway rewrite prompts.** Injecting a timestamp or request id above the stable prefix destroys the provider's prompt cache on every request — M10 chapter 6's mistake, committed centrally for every feature at once.

### Verify it

The red flag here is the module's bluntest: **a per-request cost nobody has measured** — and its cousin, a monthly bill nobody can attribute to a feature.

1. **Ask for last month's cost broken down by feature.** If it cannot be produced, the feature tag is missing and that is the first thing to build.
2. **Ask what happens when one user sends a thousand requests in an hour.** The answer must be a refusal at a stated number, not an alert.
3. **Ask where the provider keys are.** Any in application code or a frontend is a stop-everything finding.
4. **For a semantic cache, demand the labelled pair set and the measured false-hit rate.** No measurement, no semantic cache.
5. **Read the cache key out loud, field by field.** Model, prompt version, document versions, locale, permission scope. A missing permission scope is a data leak.
6. **Ask what the fallback is, when it was last exercised,** and whether the regression suite was run against it.
7. **Check the gateway does not modify the prompt prefix,** then compare prompt-cache hit rates before and after it appeared.

Red flags: no per-feature cost attribution; a per-user budget that only warns; keys outside the gateway; a threshold chosen by intuition; a cache key without a permission scope; no fallback or an untested one; no output cap; a gateway injecting anything above the stable prefix; and cache hits not logged with both queries.

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
