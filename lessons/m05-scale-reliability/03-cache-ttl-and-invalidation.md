---
id: m05-03-ttl-invalidation
module: m05-scale-reliability
slug: cache-ttl-and-invalidation
title: "Hits, Misses, TTL, Eviction and Invalidation"
order: 3
duration_min: 35
is_free: false
mission: true
volatility: low
last_reviewed: 2026-11-16
---

## Key learnings
- Choose a TTL from how stale the data is allowed to be, in seconds
- Delete the cache key on every write path, not just the obvious one
- Tell the three ways a cached entry dies apart: expiry, eviction and invalidation
- Measure hit rate and prove a cache is actually being hit
- Catch an AI that caches a value and never invalidates it

## Lesson

### The pain

At 1:06 pm the chef runs out of paneer and marks it sold out. He watches the admin screen confirm it. He is satisfied.

At 1:07 pm a student orders paneer. At 1:09 pm another one does. By 1:17 pm eleven students have paid for a dish that does not exist, there is a queue of angry people at the counter, and your team is processing refunds by hand through UPI while the rush is still happening.

Nothing was broken. The database row said `is_available = false` the entire time. Your cache had a copy that said `true`, with a one-hour lifetime, and you never wrote the three lines that throw that copy away when the chef changes something. The cache did exactly what you told it to. You simply never told it when to stop believing itself.

This is the part of caching that is actually hard. Making a cache fast takes ten minutes. Making a cache correct is a design decision you have to make per piece of data, and it is the decision AI skips most often.

### The idea

**Every cached entry needs an answer to one question: when do you stop believing this? There are only two good answers — after N seconds, or when the underlying thing changes — and most data needs both.**

Time is the safety net. A TTL, or time to live, is a number of seconds after which the cache throws the copy away whether anybody asked it to or not. It guarantees your staleness never exceeds that number, which is the only guarantee you get for free.

Events are the precision. When the chef writes to the database, you delete the cache key in the same breath. The next read misses, refills from the truth, and the staleness window drops from an hour to roughly zero.

TTL alone is survivable. Invalidation alone is not, because you will eventually miss a write path. Both together is the answer for almost everything you will build.

### The analogy

Watch the samosa warmer at the counter through a whole lunch rush.

Samosas go in at noon and are pulled at 12:30 whether anyone bought them or not. That is a **TTL**: a fixed lifetime, applied blindly, that bounds how wrong the counter can be.

The warmer holds forty plates. When a fresh batch arrives and there is no room, the staff clear out whatever has been sitting longest to make space. That is **eviction**: not a decision about freshness at all, but about running out of room, and it is the cache deciding, not you.

And when the chef shouts "paneer is finished, pull it off the board", the board changes in three seconds regardless of what any clock says. That is **invalidation**: an event-driven removal, triggered by the write that made the old answer wrong.

**Where the analogy breaks:** a samosa past its TTL looks and smells wrong, so somebody notices. Stale data looks immaculate. It has the right shape, the right fields and a plausible timestamp, and the only way anybody finds out is a student paying ₹60 for a dish that ran out eleven minutes ago. The whole discipline exists because staleness is invisible.

### Picture it

```
READ PATH (cache-aside)
  request ──▶ cache ──hit──▶ answer in 1 ms
                │
               miss
                ▼
            database ──▶ store under key, with TTL ──▶ answer in 50 ms

WRITE PATH
  chef saves ──▶ database (truth) ──▶ DELETE the key
                                          │
                                          ▼
                                  next read misses and refills
```

A read checks the cache first and falls through to the database only on a miss; a write updates the database and then deletes the key, so the very next read rebuilds a correct copy.

### Three ways an entry dies

Keep these separate in your head, because people use the word "expired" for all three and then cannot debug any of them.

| How it dies | Who decides | What it protects you from |
|---|---|---|
| Expiry (TTL) | The clock | Unbounded staleness |
| Eviction | The cache, when memory is full | Running out of memory |
| Invalidation | Your write path | Being wrong after a known change |

Eviction is the one students meet first on a free tier, because free Redis plans are small — often around 25 to 30 MB with a cap on simultaneous connections. When that fills, the cache starts throwing things out under a policy. The usual default is **LRU**, least recently used: the entry nobody has asked for in the longest time goes first, which is a good guess most of the time. Some setups instead refuse new writes when full, which is worse, because your cache silently stops caching and nothing errors.

Hit rate is how you know any of this is working. If the cache answered 940 of the last 1,000 requests, your hit rate is 94%. Below about 70% on data you expected to be hot, something is wrong with your key — usually it is too specific, so every request invents a brand new key and nothing is ever reused.

### The stampede

One more failure mode, because it will happen to you at 1:10 pm specifically.

A popular key expires. Forty requests arrive in the same second. All forty miss. All forty query the database at once for the identical answer, and the database — which was coping fine — now has forty copies of its most expensive query running together. That is a **cache stampede**, or thundering herd, and it is how a cache designed to protect your database ends up being the thing that takes it down.

Two cheap defences. Add **jitter**: instead of a flat 300-second TTL on everything, use 300 seconds plus a random 0 to 60, so a thousand keys do not expire in lockstep. And allow **stale-while-revalidate** where the data tolerates it: serve the slightly old copy immediately while one single request goes off to refresh it. The student gets a 1 ms answer, the database gets one query instead of forty, and nobody notices the two seconds of staleness.

### Where it lives

This chapter is the rules for the Redis layer from Chapter 2, and the same rules apply one layer out: a CDN entry has a TTL, and purging a CDN path is invalidation.

It connects outward in three directions. M04's queues are the clean place to put invalidation work that must not slow down the write. M07's metrics are where hit rate and miss rate belong on a dashboard, next to p95. And M10 reuses this exact vocabulary for language-model calls — a semantic cache has a TTL and an invalidation problem just like a menu does, and the cost of a stale AI answer is measured in rupees per call rather than milliseconds.

### Words to own

| Term | Plain meaning | Example |
|---|---|---|
| Hit | The cache had the answer | 940 of 1,000 requests |
| Miss | It did not, so you went to the truth | The remaining 60 |
| Hit rate | Hits divided by total lookups | 94% |
| TTL | Seconds before the copy is discarded | 60 for a sold-out flag |
| Eviction | The cache drops entries to free memory | LRU on a full free tier |
| LRU | Least recently used goes first | The default eviction policy |
| Invalidation | Deleting a key because the data changed | Chef saves, key is deleted |
| Cache-aside | Read cache, miss, read database, store | The pattern above |
| Write-through | Write to cache and database together | Keeps both in step, costs a hop |
| Stampede | Many simultaneous misses on one key | Forty requests at 1:10 pm |
| Jitter | Random spread added to a TTL | 300 s + 0-60 s |
| Negative cache | Storing "not found", briefly | 10 s on a missing dish |

### Tradeoffs and gotchas

**A short TTL is almost always the right first answer.** Sixty seconds on a shared read removes 98% of your database load at lunch rush and bounds your wrongness at one minute. Teams reach for clever invalidation and end up with neither correctness nor simplicity. Start with a number of seconds you can defend out loud, and add event invalidation where one minute is genuinely too long.

**You will forget a write path.** The admin page deletes the key. The bulk-import script does not. The scheduled job that marks yesterday's specials unavailable does not. The teammate's new endpoint does not. This is why the TTL stays even after you add invalidation: it is the thing that limits the damage from the write path nobody remembered.

**Never let `flushall` be your invalidation strategy.** Wiping the entire cache to fix one stale key makes every other key cold at once, which is a self-inflicted stampede across your whole app. Delete the key you meant.

**Do not cache errors for long.** If the database is briefly down and you cache the failure for an hour, you have extended a thirty-second outage into an hour-long one. Negative caching is useful — storing "this dish does not exist" stops a hammering lookup — but at five or ten seconds, never minutes.

**Hit rate without a denominator is meaningless.** "We have a 99% hit rate" on a key looked up four times a day tells you nothing. Ask how many lookups, over what window.

**Caching hides a bad query rather than fixing it.** If your uncached query takes 900 ms for want of an index, a cache turns that into 900 ms for one unlucky student in sixty rather than all of them. A real improvement, and still a missing index. M04 taught you to read the query plan; do that first, then cache.

### Verify it

**Demand the invalidation story before the code.** The question is: "list every code path that writes this data, and show me the key deletion in each one." If the AI shows you a read path with a TTL and no write path, you have the paneer bug already written down.

**Red flag: a TTL with no reasoning attached.** 3600 appears constantly in AI-generated caching code because it is a round number of seconds, not because anyone decided an hour of staleness was acceptable. Make it state the number and the sentence that justifies it: "60 seconds, because the chef expects a sold-out flag to reach students within a minute."

**Red flag: caching with no invalidation at all, described as complete.** Ask what happens if the data changes one second after it is cached. The honest answer is a number of seconds of wrongness. If you get "the cache will update automatically", push back — that phrase almost always means "it expires eventually" and sometimes means nothing at all.

**Red flag: a key that is missing one of its inputs.** Read the key aloud and list everything the answer depends on. A menu key that omits the date serves yesterday's specials. A dashboard key that omits the user serves a stranger's data.

**Red flag: the same fix again — a cache added in front of something that was never the bottleneck,** or "increase the Redis memory" proposed for what is actually an unbounded key pattern writing a new entry per request and never reusing one. More memory does not fix a leak; it postpones it.

Then prove it by hand rather than reading a summary. Load the page twice and check the second is dramatically faster. Change the underlying row and time, by stopwatch, how long until it appears — that number must be at or under your stated TTL. Watch hit and miss counters over five real minutes. And check key count after an hour; if it climbs forever, something has no TTL.

## Walkthrough
1. In your own app, pick the single read that is slowest and most repeated. Write it down with its measured time from Chapter 1 and roughly how many times an hour it runs.
2. Write one sentence, with a number of seconds in it, saying how stale that data is allowed to be. "Up to 60 seconds old is fine because the chef expects changes to land within a minute." This sentence is your TTL.
3. Write down the cache key, listing every input the answer depends on: the entity, the date if it matters, the user if the data is personal. Read it aloud and check nothing is missing.
4. Now list every place in your codebase that writes the underlying data. Include admin pages, scripts, scheduled jobs and anything a teammate added this week. Expect to find more than you thought.
5. Brief an AI to implement cache-aside for that one read, with your TTL, plus a key deletion in every single write path you listed. Give it the list; do not let it find the paths itself.
6. Read the diff and count the deletions. There must be one per write path on your list. A missing one is the paneer bug, pre-installed.
7. Load the page twice and record both timings. The second must be dramatically faster. If it is not, your cache is not being hit and nothing else in this walkthrough matters yet.
8. Now test staleness deliberately. Change the underlying row directly in the database — not through the admin page — then reload and time how long until the page reflects it. That duration must be at or under your TTL.
9. Test invalidation. Change the same row through your admin page instead and reload immediately. The change must appear at once, because the key was deleted.
10. Add a hit and miss counter, use the app normally for five minutes, and compute your hit rate. Under 70% on data you expected to be hot means your key is too specific.
11. Finally, check memory and key count in your cache dashboard. If key count grows without limit, something is being cached with no TTL — find it now, before a free-tier eviction surprises you.

## Talk to AI
```prompt
CONTEXT:      I am a student directing AI to build my team's app. Stack:
              [your stack]. Database: [your database], free tier. Cache:
              one free-tier Redis-compatible instance, roughly 25-30 MB
              and a low connection limit. We run more than one app
              instance and cannot control which one serves a request.

              The read I want to cache:
              - what it returns: [describe it]
              - measured time without a cache: [X] ms
              - roughly how often it is requested: [X] per hour
              - is it the same for every user: [yes/no]
              - how stale it may be, in seconds: [X]

              Every code path that writes this data:
              1. [e.g. admin availability toggle]
              2. [e.g. nightly specials job]
              3. [e.g. bulk CSV import script]
              4. [any others - list them all]

GOAL:         A correct cache-aside implementation for this one read, with
              a defended TTL and an invalidation in every write path above.

REQUIREMENTS: 1. State the exact cache key, listing every input the answer
                 depends on, and justify each input
              2. State the TTL in seconds and the one sentence that
                 justifies it against my staleness number
              3. Delete the key in every one of the write paths I listed -
                 one deletion per path, and tell me if you think I missed a
                 path
              4. Handle a cache outage: if the cache is unreachable, the app
                 must still work by reading the database
              5. Add hit and miss counters I can read, and tell me where
              6. Tell me whether this key needs stampede protection and
                 why, given my request rate

CONSTRAINTS:  Free tier only, so assume the cache may be wiped or may evict
              under memory pressure. Nothing may live only in the cache.
              Do not add a second cache layer. Do not change my database
              schema.

NON-GOALS:    Do not cache anything other than this one read. Do not use
              flushall anywhere. Do not refactor unrelated files.

ACCEPTANCE:   - The key contains every input, including the user if the data
                is personal
              - The TTL has a sentence of justification, not a round number
              - There is one key deletion per write path I listed
              - With the cache unreachable, the page still loads from the
                database
              - I can state my hit rate after five minutes of use
              - Changing the row directly in the database is reflected
                within my stated TTL

ASK:          First give me the key, the TTL with its justification, and a
              numbered list of the places you will delete the key. Tell me
              which of my write paths worries you most. No code until I
              confirm the list is complete.
```

## Mission

**Objective.** Cache the single hottest read in your own startup's app with a defended TTL and working invalidation, and prove with your own measurements that it is faster, that its staleness is bounded, and that a write clears it immediately.

**Steps.**
1. **Frame** — name the one read you are caching, its measured latency from Chapter 1, and its rough request rate. One read only. Resist the urge to cache three things.
2. **Decide the staleness budget** — write one sentence containing a number of seconds, and whose expectation that number comes from. A chef, a student, a payment. This is the hardest thinking in the mission and it takes four minutes.
3. **Enumerate the write paths** — as a group of three, each person independently lists every place the underlying data is written. Combine the lists. The union is almost always longer than any individual's, which is the whole point of doing it three times.
4. **Brief** — write a Director's Brief using the prompt above, with your real numbers and your real write-path list. Demand the key, the TTL justification and the deletion list before any code.
5. **Direct** — have it implement cache-aside for that one read. Read the diff yourself and count the key deletions against your list.
6. **Verify speed** — record the timing of the first and second load. Both numbers, screenshotted.
7. **Verify the staleness bound** — change the row directly in the database, then time by stopwatch how long until the app shows it. Record the number. It must be at or under your TTL.
8. **Verify invalidation** — change the same row through your app's own write path and reload immediately. It must be correct at once. Record it.
9. **Verify resilience** — stop or disconnect the cache and load the page. It must still work, slowly. If it 500s, your cache is a dependency rather than an optimisation, and that must be fixed before you submit.
10. **Measure** — use the app for five minutes, then report hits, misses and hit rate with the raw counts, not just the percentage.
11. **Document** — one decision-log entry in your repo: what you cached, the key, the TTL, the sentence justifying it, and what you chose not to cache.

**Upload.**
- Your Director's Brief, in full → post caption or an attached file
- Your write-path list, including the ones only one teammate thought of → text
- The AI conversation showing the key, TTL justification and deletion list → share link in `ai_chat_url`
- The working app → deployed URL in `demo_url`
- First-load and second-load timings → screenshots
- Your stopwatch measurement of the staleness bound, with your TTL written next to it → screenshot or text
- The page still loading with the cache disconnected → screenshot
- Hit, miss and hit-rate counts after five minutes of real use → screenshot
- Your decision-log entry
- Reflection, 150 words max → one tradeoff you chose, and one thing the AI got wrong or left out

**Self-check.** Tick every box before you submit.
- [ ] My TTL is a number of seconds with a sentence justifying it, not 3600
- [ ] My cache key lists every input the answer depends on
- [ ] If the data is per-user, the user identifier is in the key
- [ ] I listed the write paths myself before the AI saw them
- [ ] There is one key deletion per write path, and I counted them in the diff
- [ ] I measured the staleness bound with a stopwatch, not by reasoning
- [ ] The measured staleness is at or under my stated TTL
- [ ] The page still works with the cache disconnected
- [ ] I report hit and miss counts, not just a percentage
- [ ] Nothing in my app exists only in the cache
- [ ] My reflection names something specific the AI got wrong

**Stretch.** Add jitter to your TTL and explain in 80 words what stampede it protects you from at your actual request rate — including the honest answer if your traffic is too low for it to matter yet.

## Stuck? Ask AI
```prompt
Explain the difference between expiry, eviction and invalidation using one analogy that is not about food or libraries. Then give me six short situations and ask me which of the three is happening in each, one at a time, without telling me the answers first.
```

```prompt
I added a cache and now users sometimes see old data. Here is my read path and my write path: [paste them]. Do not rewrite my code. Instead, list every way this could produce stale data, in order of how likely each is, and tell me the single measurement that would distinguish between them.
```

```prompt
Explain a cache stampede to me step by step, with forty simultaneous requests and one expired key, and tell me what the database sees at each moment. Then give me three defences, ranked by how much complexity each adds, and tell me honestly at what traffic level each one starts to be worth it.
```

## Quiz

### Q1 · L2
What does a TTL on a cache entry control?
- (a) How much memory the entry may use
- (b) How many seconds the copy is believed before being discarded ✅
- (c) How many users may read the entry
- (d) Which eviction policy applies to it
> TTL is a clock, not a size limit. It is the only guarantee of bounded staleness you get without writing any invalidation code.

### Q2 · L3
The chef marks a dish sold out and students keep ordering it for eleven minutes. What is the defect?
- (a) The database write failed
- (b) The TTL was too short
- (c) The write path does not delete the cache key, and the TTL is long ✅
- (d) The cache ran out of memory
> The truth was correct the whole time. A cached copy with a long lifetime and no event invalidation is the exact shape of this bug.

### Q3 · L4
You add key deletion to your admin toggle. Why keep the TTL as well?
- (a) Redis requires a TTL on every key
- (b) It bounds the damage from the write path you forgot ✅
- (c) It increases the hit rate
- (d) TTLs are needed for eviction to work
> Bulk imports, scheduled jobs and a teammate's new endpoint are the paths that get missed. TTL is the safety net that turns "wrong forever" into "wrong for sixty seconds".

### Q4 · L4
A popular key expires and forty requests arrive in the same second. What happens, and what is the cheapest defence?
- (a) The cache returns stale data; no defence needed
- (b) All forty miss and hit the database together; add jitter or serve stale while one refreshes ✅
- (c) Thirty-nine requests queue behind the first automatically
- (d) The cache evicts other keys to make room
> This is a stampede. Random jitter on TTLs stops keys expiring in lockstep, and stale-while-revalidate lets one request refresh while the rest get a fast slightly-old answer.

### Q5 · L5
An AI implements caching and reports it is done. Which single question most reliably exposes whether it is correct?
- (a) What is the cache's memory limit?
- (b) List every code path that writes this data and show me the key deletion in each ✅
- (c) Which eviction policy did you choose?
- (d) What is the expected hit rate?
> Speed is easy to verify and easy to get right. Correctness fails at the write paths, which is exactly where generated caching code is thinnest.

## Go deeper
- [Redis: key expiration and eviction policies](https://redis.io/docs/latest/develop/reference/eviction/)
- [Caching strategies and the stampede problem](https://aws.amazon.com/builders-library/caching-challenges-and-strategies/)
