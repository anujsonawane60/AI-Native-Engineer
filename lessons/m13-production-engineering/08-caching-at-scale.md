---
id: m13-08-caching-at-scale
module: m13-production-engineering
slug: caching-at-scale
title: "Caching Strategy at Scale: Layers, Stampedes, Invalidation Graphs"
order: 8
duration_min: 30
is_free: false
mission: false
volatility: low
last_reviewed: 2026-11-16
---

## Key learnings
- Draw the invalidation graph: every derived key one write makes wrong
- Purge a CDN by tag instead of by path, and know which layer you cannot purge
- Set a TTL ladder per layer, deciding in advance for the layers you cannot reach
- Stop a stampede with single-flight locking rather than only jitter
- Catch a cache invalidation that handles the obvious key and nothing else

## Lesson

### The pain

M05 chapter 2 taught you the five layers — browser, CDN, in-process, shared cache, database — and the rule that personalised data never goes in a shared layer. M05 chapter 3 taught TTL, eviction, invalidation, hit rate, the stampede, jitter and stale-while-revalidate, and it taught the paneer bug: the chef marks a dish sold out, the cache keeps saying available, eleven students pay for food that does not exist. You learned to delete the key in every write path. That was one cache and one key.

Now the same bug, two layers out. The chef marks paneer sold out. Your write path deletes `dish:42` from the shared cache exactly as you were taught, and it works perfectly. The menu *page* is still wrong for eleven minutes, because the HTML is cached at the CDN with a ten-minute lifetime and then in each student's browser for five more. You did the invalidation you learned and it did not reach the layer the student actually reads from. This is not the same bug; it is a layer problem that the single-cache fix cannot touch.

And the harder one. A dish's price changes. You delete `dish:42`. But five other things were derived from that price: the `menu:today` payload, the snacks-category summary with its "from ₹40" line, the search index entry, the homepage HTML, and the cached count of dishes under ₹50. All five are now wrong, none has the string `42` in its key, and nobody can list them from memory. A student sees ₹40 on the category page and ₹55 at checkout, and stops trusting you.

### The idea

**At scale, caching stops being a per-key decision and becomes two artefacts: a written invalidation graph, and a TTL ladder with one decision per layer.**

The **invalidation graph** answers: when this thing changes, which cached entries become wrong? Including derived ones, including ones at other layers, including the search index. It is a diagram you draw once per entity and keep in the repository.

The **TTL ladder** answers: for each layer, how stale may this be, and can I purge it? Layers you can purge get long TTLs and event invalidation. Layers you cannot purge — the student's browser, an intermediate proxy — get a short TTL chosen before launch, because after launch you have no control over them at all.

### The analogy

M05 gave you the canteen menu copied in five places: the student's phone photo, the sheet at the hostel gate, each cashier's apron note, the whiteboard behind the counter, and the kitchen's stock ledger.

Watch what happens when the chef raises the paneer price. The whiteboard changes instantly. The cashiers must each be told — five people, and you will forget one. Someone has to physically walk to the hostel gate with a new sheet. And the student's photo cannot be reached at all; the only control you ever had over it was the "valid until 1 p.m." you printed on the sheet before they photographed it.

Then the part nobody thinks about. The "today's specials" poster by the entrance was *copied from* the price list last night. It is not a copy of the paneer price; it is a thing derived from it. Changing the board does not change the poster, and nobody remembers the poster exists until a student points at it.

**Where the analogy breaks:** you can phone the hostel gate, and a sheet left up too long eventually looks obviously old — curled, dusty, last week's date. A stale cached response looks immaculate and has a plausible timestamp. And you cannot phone the student's browser, ever, which is why the only honest lever at that layer is a number you chose in advance and must now live with.

### Picture it

```
  WRITE: dish 42 price changes
    │
    ├─▶ shared cache  dish:42            purge now
    ├─▶ shared cache  menu:today         purge now (derived)
    ├─▶ shared cache  cat:snacks:summary purge now (derived)
    ├─▶ search index  doc 42             reindex (derived)
    ├─▶ CDN  /menu, /dish/42, /          purge by tag dish-42
    └─▶ browser copies                   CANNOT purge
                                         → TTL 60 s, chosen
                                           before launch
```

One write makes six cached things wrong, four of them derived rather than named after the entity, and exactly one layer cannot be purged at all — which is why its lifetime has to be a decision made before anyone depends on it.

### Where it lives

This is M05 chapters 2 and 3 taken from one cache to a system of caches. The stampede defences extend M05 chapter 3's jitter and stale-while-revalidate with **single-flight** locking: on a miss, one request takes a short lock and refills while the others wait briefly or get the stale copy, so forty misses become one database query rather than forty. The deploy interaction is chapter 5's: restarting every instance at once empties every in-process cache simultaneously, which is a stampede you caused yourself, and a staggered rollout plus a warm-up request is the fix.

Chapter 7 is the honest check on all of it: a load test with varied data reveals your real hit rate per layer, and a load test with one id is how you fool yourself. And M05 chapter 2's security rule gets sharper here — a personalised response that reaches a shared CDN entry is not a performance bug, it is one student seeing another's name, which M07 chapter 1 would class as broken access control.

### Words to own

| Term | Plain meaning | Example |
|---|---|---|
| Invalidation graph | Map of which keys one write makes wrong | Drawn per entity |
| Derived key | A cached thing computed from another | `cat:snacks:summary` |
| Surrogate key / tag | A label on a cached response, purgeable as a group | `dish-42` |
| Purge by tag | Clear every response carrying a label | One call, many URLs |
| TTL ladder | A chosen lifetime per layer | 300 s CDN, 60 s browser |
| Unpurgeable layer | A cache you cannot reach after the fact | The browser |
| Single-flight | One request refills; the rest wait or serve stale | Lock on the key |
| Request coalescing | The edge merges identical in-flight misses | CDN feature |
| Early recomputation | Refresh before expiry, at random, to avoid lockstep | Probabilistic |
| Cold cache | Nothing cached yet, so everything hits the origin | After a full restart |
| Cache warming | Requesting hot keys before traffic arrives | Post-deploy step |
| Hit-rate budget | The hit rate each layer is expected to deliver | Edge 80%, app 95% |
| Key cardinality | How many distinct keys a pattern can create | One per user, bounded |
| Coherence | All layers agreeing about the same fact | The whole problem |

### Tradeoffs and gotchas

**Draw the graph before you write the invalidation.** Pick an entity, list every cached thing that reads it directly, then everything computed from *those*, then the search index, then the pages at the edge. Fifteen minutes per entity, and it is the only way derived keys stop being surprises. Commit the diagram.

**Tag your edge responses so you can purge by meaning.** Purging by path requires you to know every URL a dish appears on, which you do not. Tagging a response with the ids it contains lets one purge call clear every page that mentions dish 42, including ones created after you wrote the invalidation code. Without tags, the only honest CDN strategy is a short TTL.

**Never purge everything to fix one thing.** M05 chapter 3 said this about `flushall`; a full CDN purge is the same mistake one layer bigger, because every page becomes a miss at once and your origin takes traffic that had been 90% absorbed.

**The layers you cannot purge must be short, and that decision is permanent in practice.** A browser cache lifetime of a day is a day of potential wrongness you cannot shorten later, because the copies already handed out carry the old number — exactly the DNS TTL lesson from M06 chapter 13. Sixty seconds on anything that can change, long lifetimes only on content whose URL changes when the content does.

**Jitter and stale-while-revalidate are not enough on their own at real volume.** They spread expiry and hide refresh latency. They do not stop a genuinely cold key from being requested by four hundred users in the same second — a new dish going viral, or a cache that was just emptied. Single-flight locking is what bounds that to one origin request. Add a short lock timeout so a crashed holder cannot block the key forever.

**A deploy is a cache event.** Restarting all instances together empties in-process caches together. On a small system this shows up as a thirty-second latency spike after every deploy that everyone has learned to ignore; on a bigger one it is an outage. Stagger the restart, and consider a warm-up that requests your top keys before an instance joins the load balancer.

**Measure hit rate per layer, with a denominator, and watch key count beside it.** "95%" means nothing without knowing which layer and how many lookups. Each layer has a different job: the edge absorbs volume, the shared cache absorbs expensive computation, the in-process cache absorbs chatter, and a layer sitting at 20% is costing you a lookup and buying nothing. Rising key count with a falling hit rate is the signature of unbounded cardinality — a key containing a timestamp or a search string invents a new entry per request and fills your free-tier memory until eviction starts discarding the keys that were working.

### Verify it

The question from M05 chapter 3 — "list every code path that writes this data and show me the key deletion in each" — still applies and is no longer sufficient. The new question is about the graph and the layers.

1. **Ask for the full invalidation set, derived keys included.** "When this price changes, list every cached entry that becomes wrong, at every layer, including anything computed from it and the search index." An answer naming only the entity's own key has not understood the problem.
2. **Ask which layers the plan actually reaches.** If the answer covers the shared cache and never mentions the CDN or the browser, the paneer bug is still live two layers out.
3. **Measure staleness per layer with a stopwatch.** Change the data, then time how long until it appears: in a direct API call, in the rendered page, in a hard-reloaded browser, and in a normal reload. Four different numbers, each of which must be at or under the TTL you chose for that layer.
4. **Check the unpurgeable layer's number specifically.** Read the response headers and find the browser's lifetime. If it is longer than you can defend, change it today, because tomorrow's copies are already being handed out.
5. **Force a cold cache and watch the origin.** Empty the cache deliberately while chapter 7's load runs. If the origin falls over, you have no single-flight protection and your cache has become a dependency rather than an optimisation.
6. **Look for personalisation in a shared layer.** Request a page as one logged-in user and then as another, through the same edge. If you ever see the first user's name, stop everything — that is a security incident, not a cache bug.

Red flags: an invalidation that handles only the obvious key; no mention of the CDN or browser layer; a full purge as the invalidation strategy; a browser cache lifetime over a minute on changeable data; stampede protection described as "we added jitter" at high request rates; a deploy process that restarts everything at once; a cache the app cannot run without; a key containing a timestamp or a free-text search string; and a hit rate quoted with no layer and no denominator.

## Walkthrough
1. Pick the single entity in your product whose change matters most — a dish, a price, an availability flag. Write its name at the top of a page.
2. Draw its invalidation graph by hand. List every cached entry that reads it directly, then everything computed from those, then your search index if you have one, then every page at the edge that displays it. Keep going until your teammates stop adding items.
3. For every item on that graph, write down which layer it lives in and whether you can purge it. Mark the unpurgeable ones clearly; they are the ones whose TTL you must decide rather than control.
4. Write your TTL ladder as a four-row table: browser, CDN, shared cache, in-process. One lifetime each, with one sentence of justification. The browser row is the one to argue about.
5. Read your actual response headers for the page that shows this entity. Find the real browser lifetime being sent today and compare it with the row you just wrote. Fix the mismatch.
6. Measure staleness per layer with a stopwatch. Change the entity, then time its appearance in a direct API call, in the rendered page from the edge, in a hard reload, and in a normal reload. Record all four numbers against your ladder.
7. Add tags to your edge responses naming the entity ids they contain, then purge by tag and confirm every page that mentions the entity updates — including one you did not list in step 2.
8. Defeat your own cache in a load test: run chapter 7's ramp with one single id, record the hit rate, then run it with two hundred varied ids and record it again. The gap between those two numbers is how much your earlier tests were flattering you.
9. Force a cold cache during that varied-data run. Empty the shared cache and watch the origin's latency and error rate. Record what happened; this is your stampede exposure with real numbers.
10. Add single-flight locking on your hottest key, with a short lock timeout, and repeat step 9. Record the difference in origin requests.
11. Check two safety properties: request the same personalised page as two different logged-in users through the edge and confirm no leakage, then plot key count over an hour and confirm it stabilises rather than climbing forever.

## Talk to AI
```prompt
CONTEXT:      I am a student on an AI-native engineering program. I direct AI
              and verify its output; I do not write code by hand. My team of
              three runs [product in two sentences] with [number] real users
              in India on budget Android phones and slow 4G. Stack: [your
              stack]. Our cache layers today are: [list them honestly -
              browser headers, CDN, in-process memory, a free-tier shared
              cache, and say which you are not sure about]. We run [number]
              app instances. We already have TTLs and write-path invalidation
              from earlier in the course for ONE key. We also have [a search
              index / no search index].

              The entity whose change matters most is [name it], and the
              cached things I know about today are: [list them].

GOAL:         A written invalidation graph for that entity and a TTL ladder
              per layer, so a change reaches every layer within a staleness I
              chose, and a cold cache cannot take down my origin.

REQUIREMENTS: 1. Build the full invalidation graph: every cached entry that
                 becomes wrong when this entity changes, including DERIVED
                 entries computed from it, the search index, and every page
                 cached at the edge. Tell me which ones I probably missed
              2. For each item, name its layer and whether I can purge it.
                 List the unpurgeable layers separately
              3. Give me a TTL ladder as a table - browser, CDN, shared cache,
                 in-process - with one sentence of justification per row, and
                 explain why the browser row cannot be shortened after the
                 fact
              4. Show me how to TAG edge responses with the entity ids they
                 contain so one purge clears every page mentioning it, and
                 tell me what my only honest option is if tagging is not
                 available on my plan
              5. Explain why a full purge to fix one key is dangerous, with
                 the arithmetic for my request rate
              6. Give me single-flight locking for my hottest key: how one
                 request refills while others wait or get stale, the lock
                 timeout, and what happens if the lock holder crashes. Explain
                 why jitter and stale-while-revalidate alone do not cover a
                 genuinely cold key
              7. Tell me what my deploy does to my in-process caches across
                 all instances, and how to stagger restarts and warm up
              8. Give me the per-layer hit-rate measurements I should collect,
                 each with a denominator, and tell me what a 20% hit rate at
                 one layer means I should do
              9. Tell me how to prove no personalised response can reach a
                 shared edge entry, and why that would be a security incident
                 rather than a performance bug

CONSTRAINTS:  Free tier only - our shared cache is roughly 25-30 MB with a low
              connection limit, so assume eviction under pressure and nothing
              may live only in the cache. Plain English for a second-language
              reader. Do not change my database schema. Do not add a second
              caching product. Label every command READ-ONLY or
              CHANGES-THINGS.

NON-GOALS:    Do not cache anything new in this conversation - this is about
              making the existing caches coherent. Do not use a full flush or
              a full CDN purge anywhere. Do not add a CDN if I do not have
              one. Do not design for a million users.

ACCEPTANCE:   - I have a committed invalidation graph with derived keys on it
              - I have a four-row TTL ladder with a justification per row
              - My browser cache lifetime matches the row I wrote, verified in
                response headers
              - I measured staleness per layer with a stopwatch and every
                number is at or under its chosen TTL
              - One purge by tag updates a page I had not listed
              - Emptying the shared cache under load does not take down the
                origin
              - Two different logged-in users never see each other's data
                through the edge
              - My key count stabilises over an hour instead of climbing

ASK:          Before any code, build the invalidation graph and tell me which
              derived entries you think I have forgotten and why. Then give me
              the TTL ladder and wait for me to argue about the browser row.
```

## Stuck? Ask AI
```prompt
I invalidate the cache key for a dish when its price changes, and users still see the old price. Here are all my cache layers: [list them, with the TTL of each if you know it]. Do not rewrite my code. Instead, for each layer in turn, tell me whether my invalidation reaches it, how long it could keep the old value, and the exact read-only check that would prove which layer is serving the stale copy. Rank the layers by how likely each one is to be the culprit.
```

```prompt
Explain a derived cache key with three concrete examples from an app that sells food: things cached that are computed from a dish's price rather than being the price itself. Then ask me to list the derived entries in my own product, one at a time, and after each answer tell me what category of derived entry I still have not thought of. Do not give me a complete list up front.
```

```prompt
Explain the difference between jitter, stale-while-revalidate, and single-flight locking as three stampede defences. For each, give me one scenario it handles and one it does not. Then walk me through what the origin sees when 400 requests arrive in one second for a key that has never been cached, under each of the three defences, with numbers.
```

## Quiz

### Q1 · L3
You correctly delete the shared-cache key when a dish sells out, and students still see it available for eleven minutes. What is the most likely cause?
- (a) The TTL on the shared-cache key was too long
- (b) The rendered page is cached at the CDN and in browsers, and your invalidation never reached those layers ✅
- (c) The database write failed silently
- (d) The cache ran out of memory and refused the delete
> This is the paneer bug two layers out. A single-key fix cannot reach the layers that cache the whole response, which is why the TTL ladder and purge-by-tag exist.

### Q2 · L4
A dish's price changes. Which cached entry is most likely to be forgotten?
- (a) The dish's own key
- (b) The category summary showing "from ₹40", which was computed from the price ✅
- (c) The database row itself
- (d) The user's session record
> Derived entries do not contain the entity's id in their key, so nobody finds them by searching. They are exactly why the invalidation graph is drawn once, in writing, rather than recalled from memory.

### Q3 · L4
Your browser cache lifetime on the menu page is one day, and you decide today it should be sixty seconds. What have you fixed?
- (a) Everything; the next request will use the new lifetime
- (b) Only future copies — the day-long copies already handed out cannot be shortened or reached ✅
- (c) Nothing, because browser caches ignore lifetime headers
- (d) Everything, once you purge the CDN
> Same lesson as the DNS TTL in M06 chapter 13: a cache you cannot reach holds the clock it was given. This is why the unpurgeable layer's number has to be decided before launch.

### Q4 · L4
Four hundred requests arrive in one second for a key that has never been cached. Which defence bounds the origin to one query?
- (a) Jitter on the TTL
- (b) Stale-while-revalidate
- (c) Single-flight locking, so one request refills while the others wait or serve stale ✅
- (d) Increasing the cache's memory limit
> Jitter prevents lockstep *expiry* and stale-while-revalidate needs a stale copy to serve. Neither helps a genuinely cold key, which is the case a new dish going viral or a freshly emptied cache produces.

### Q5 · L5
After every deploy, your p95 spikes for about thirty seconds and the team has learned to ignore it. What is happening, and what does it predict?
- (a) The new version is slower until the runtime warms up; nothing to do
- (b) All instances restarted together, emptying every in-process cache at once — a self-inflicted stampede that will become an outage as traffic grows ✅
- (c) The CDN is revalidating every cached page against the new build
- (d) The load balancer is redistributing connections
> A deploy is a cache event. The fix is a staggered rollout plus a warm-up before an instance takes traffic, which makes chapter 5's release mechanism and this chapter the same conversation.

## Go deeper
- [Fastly: surrogate keys and purging by tag](https://www.fastly.com/documentation/guides/concepts/purging/)
- [AWS Builders' Library: caching challenges and strategies](https://aws.amazon.com/builders-library/caching-challenges-and-strategies/)
