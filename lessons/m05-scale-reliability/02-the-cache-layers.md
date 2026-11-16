---
id: m05-02-cache-layers
module: m05-scale-reliability
slug: the-cache-layers
title: "The Cache Layers: Browser to CDN to App to Redis to Database"
order: 2
duration_min: 25
is_free: false
mission: false
volatility: low
last_reviewed: 2026-11-16
---

## Key learnings
- Name the five places a cached copy can live, in order of distance from the user
- Decide which layer a given piece of data belongs in, and why
- Explain why an in-process cache breaks the moment you run two servers
- Say what each layer can and cannot be told to forget
- Catch an AI that caches personal data in a shared or public layer

## Lesson

### The pain

Your team reads the last chapter, finds the bottleneck, and adds caching. The AI writes it in four minutes. The menu page is now fast on your laptop.

Then three things happen. A student who was logged in as Priya sees Rahul's name in the header, because the page got cached at the edge with Priya's greeting baked into it. The chef marks paneer sold out and the menu keeps showing it available for eleven minutes, with no way to force it. And when your free host restarts the server at 2 am, the first forty requests of the morning are slow again, because the cache was living in that server's memory and memory forgets on restart — exactly the point M00 made about why servers are stateless.

Each of those is the same mistake: the data was cached in the wrong layer. Caching is not one technique. It is five different techniques that share a name, and choosing between them is the entire skill.

### The idea

**A cache is a copy of an answer kept closer to whoever keeps asking — and the layer you put it in decides who can see it, how fast it is, and whether you can ever take it back.**

The five layers run from the user outward: the browser on their phone, the CDN at the edge of the network, your app server's own memory, a shared cache like Redis that all your servers talk to, and finally the database itself, which is not a cache but the truth everything else is a copy of.

Closer to the user means faster and cheaper. It also means less control, because a copy sitting on somebody's phone in Nashik is not something you can reach in and delete.

### The analogy

The canteen keeps copies of the menu in five places, and each one behaves differently.

A student photographs the menu board and keeps it in their phone gallery. That is the **browser cache**: instant, free, completely private — and utterly beyond the canteen's reach. If prices change, that student is wrong until they look again.

A copy of the board is pinned up at the hostel gate, so three hundred residents can read it without walking to the canteen. That is the **CDN**: near the users, shared by everyone in that area, and updated only when somebody walks over and replaces the sheet.

Each cashier keeps a scribbled note in their apron pocket of today's six specials. That is the **in-process cache**: the fastest thing in the building, and different in every apron. Ask two cashiers and you may get two answers.

Behind the counter is a whiteboard every cashier reads before answering. That is **Redis**, a shared cache: one version of the truth for all the staff, one short walk away, wiped and rewritten whenever the kitchen says so.

And the kitchen's stock ledger is the **database**. Slow to consult, but it is the only thing that is actually right.

**Where the analogy breaks:** a samosa taken from the warmer is gone, while a cached value can be served a million times without being used up — caches have no stock problem, only a truth problem. And food announces when it has gone off. Stale cached data looks absolutely fresh, which is why the entire next chapter is about expiry.

### Picture it

```
  Phone        Edge          Your app       Shared      Database
┌────────┐  ┌─────────┐   ┌───────────┐  ┌─────────┐ ┌─────────┐
│ browser│  │   CDN   │   │  in-proc  │  │  Redis  │ │  rows   │
│  cache │─▶│  cache  │──▶│  memory   │─▶│  cache  │▶│ (truth) │
└────────┘  └─────────┘   └───────────┘  └─────────┘ └─────────┘
   0 ms        20 ms         0.1 ms          1 ms     5 - 200 ms
  private    per region    per server     all servers  one copy
```

A request walks left to right only as far as it has to: whichever layer has a usable copy answers immediately, and only a complete miss all the way along reaches the database.

### What belongs in which layer

This table is the actual decision you will make, over and over, for the rest of the program.

| Layer | Good for | Never put here |
|---|---|---|
| Browser | Logos, fonts, CSS, JS, a user's own last-seen list | Anything that must change instantly |
| CDN | Images, menus, public product pages, static files | Anything personalised or private |
| In-process memory | Config, feature flags, tiny lookup lists | Anything that must agree across servers |
| Redis / shared | Sessions, rate-limit counters, query results, computed feeds | The only copy of anything you cannot recreate |
| Database | The truth | — |

Two rules do most of the work. **Personalised data never goes in a shared or public layer** — that is the Priya-and-Rahul bug, and it is a security incident, not a caching bug. And **anything that must be identical on every server does not go in process memory**, because you will run more than one server the first time you scale, and the chapter on load balancers explains why you cannot predict which one a student lands on.

### Where it lives

On M00's four-box map, caches sit in the gaps. The browser cache removes the network hop entirely. The CDN shortens it. Redis replaces a slow database hop with a fast one. The database is the one thing that is not a copy.

You have already met three of these layers without the word "cache" attached. M02's rendering chapter put your static files behind a CDN. M04 put uploaded images in object storage, which is served through a CDN. M07 will show you cache hit rate as a metric worth a dashboard. And M10 applies the identical idea to language-model calls: prompt caching reuses the expensive part of a long prompt, and semantic caching returns a stored answer when a new question means roughly the same thing as an old one. Same layer logic, different payload.

The practical arrangement for a student project on free tiers is small and boring: static files and images on the CDN your host already gives you, a single shared cache instance for the three or four expensive queries that actually hurt, and nothing at all in process memory. That is enough for every traffic level you will see this year.

### Words to own

| Term | Plain meaning | Example |
|---|---|---|
| Cache | A copy kept closer to the asker | Menu JSON held in Redis |
| Origin | The real source the copies came from | Your app server and database |
| Edge | A server physically near the user | A CDN node in Mumbai |
| CDN | A network of edge servers holding copies | Serving your dish photos |
| In-process cache | Memory inside one app server | A lookup table in that process |
| Shared cache | A separate cache server all app servers use | Redis or Valkey |
| Cache key | The label a stored answer is filed under | `menu:2026-11-16` |
| Hit | The cache had it | Answered in 1 ms |
| Miss | The cache did not have it | Fell through to the database |
| Warm / cold | Whether useful copies are present yet | Cold after a restart |

### Tradeoffs and gotchas

**Every cache is a second copy of the truth, and two copies disagree eventually.** That is not a flaw you can engineer away, only one you can bound. Adding a cache always trades correctness for speed; the question is how much staleness your feature can survive. A dish photo can be a day old. A "sold out" flag cannot be eleven minutes old. A UPI payment status cannot be stale at all.

**Caching the wrong thing costs more than not caching.** A cache lookup is not free: it is a network hop, a serialisation step, and code someone has to understand. If the thing you cached took 2 ms to compute, you have added complexity and latency. Cache what is slow and asked for often. Something slow but asked once a week is not a caching problem.

**In-process caches are a trap specifically because they work on your laptop.** One process, one cache, perfectly consistent, beautifully fast. Deploy two instances and half your users see one answer and half see another, intermittently, in a way that is miserable to debug. If you cannot say out loud why the data is safe to differ per server, do not put it there.

**A cache is not storage.** Redis free tiers evict data when they fill up and can be wiped on restart. Anything you cannot recompute from the database must not live only in a cache. Session tokens are the classic near-miss: losing them logs everybody out, which is survivable; losing an order is not.

**Public caching of a logged-in page is a real-world outage pattern.** It has happened to large retailers, who cached HTML at the CDN and served one customer's basket to another. The rule is absolute: if the response depends on who is asking, it does not go in a shared layer unless the cache key includes the user.

### Verify it

**Ask which layer, and why that one.** The first question to put to an AI is never "add caching" but "for each of these five pieces of data, tell me which of the five layers it belongs in, what could go wrong in each of the other four, and how stale it is allowed to be". If it answers with one layer for everything, it has not engaged with your problem.

**Red flag: a personalised response cached in the CDN or in Redis without the user in the key.** Read the cache key out loud. If it is `dashboard` rather than `dashboard:user:42`, you are one request away from showing somebody else's data. This is the most expensive caching mistake there is and AI makes it routinely, because the fast version and the correct version look almost identical.

**Red flag: caching in process memory with no mention of multiple servers.** Ask directly: "what happens to this cache when I run three instances behind a load balancer, and what happens when the host restarts one?" A good answer names both the inconsistency and the cold start. A bad answer says it will be fine.

**Red flag: caching the step that was never slow.** The recurring one from the last chapter. Make the AI state which measured step the cache removes and how many milliseconds that step cost. If it cannot, you are adding a layer for decoration.

**Red flag: a cache with no removal story.** Every cached thing needs an answer to "how does this get updated when the underlying data changes?" If the only answer is "it expires", that needs to be a deliberate choice with a number attached, not a shrug. That is the next chapter.

Then prove the hit rather than trust it. Load the page twice and compare the timings; check that the second one is dramatically faster. Look at the response headers for a cache status. And log hits and misses so you have a hit rate, not a feeling. Write the layer choice into your M01 decision log — "menu JSON in Redis, 60 s, because the chef's sold-out flag must land within a minute" is a textbook decision-log entry, and in M11 somebody will ask you why.

## Walkthrough
1. Open your deployed app, open developer tools, and reload with the Network tab open. Find a static file — a font, a logo, your JS bundle — and look at its size and time.
2. Reload the page again without clearing anything. That same file should now show as served from cache, with a time near zero. You have just watched the browser cache work.
3. Click the file and read its response headers. Find `cache-control`. Write down the `max-age` value in seconds and convert it to minutes in your notes.
4. Find a header that names a CDN or shows a cache status such as hit or miss. Reload twice more and see whether the status changes from miss to hit.
5. Now find your own API request — the one that returns data, not a file. Look at its `cache-control`. If it says `no-store` or has no header at all, nothing is caching it, and that is probably correct for now.
6. List the five slowest pieces of data your app serves. For each, write one line: how often it changes, and how stale it is allowed to be. Use seconds, not words like "quickly".
7. Sort that list. Anything allowed to be stale for hours and identical for all users is a CDN candidate. Anything that changes per minute and is shared is a Redis candidate. Anything personalised is neither, unless the key includes the user.
8. Pick exactly one item from your list — the slowest shared one — and write down the cache key you would use for it, including every piece of information the answer depends on.
9. Read that key back and ask whether two different users could ever get each other's data from it. If yes, add what is missing and read it again.
10. Finally, write down what would have to happen for that cached value to become wrong, and how a user would notice. You now have the problem the next chapter solves.

## Talk to AI
```prompt
CONTEXT:      I am a student directing AI to build a web app with a team of
              three. Stack: [your stack]. Hosting: [your host], free tier.
              Database: [your database], free tier. We may add one free-tier
              Redis instance with a small memory limit. Users are on budget
              Android phones on slow 4G in India.

              The five slowest or most-requested pieces of data in my app
              are, with how often each changes and how stale each is allowed
              to be, in seconds:
              1. [data] - changes [X] - may be stale [X] seconds
              2. [data] - changes [X] - may be stale [X] seconds
              3. [data] - changes [X] - may be stale [X] seconds
              4. [data] - changes [X] - may be stale [X] seconds
              5. [data] - changes [X] - may be stale [X] seconds

GOAL:         Decide which cache layer each of those five belongs in -
              browser, CDN, in-process memory, shared cache, or no cache at
              all - before any code is written.

REQUIREMENTS: 1. For each item, recommend exactly one layer and justify it
                 against the staleness number I gave you
              2. For each item, say explicitly what would break if I put it
                 in each of the other four layers
              3. Give me the exact cache key you would use, listing every
                 input the answer depends on, including the user where the
                 data is personalised
              4. Flag any item where caching is not worth it, and say so
                 plainly rather than finding a layer for everything
              5. Tell me what happens to each choice when I run more than
                 one app server, and when a server restarts

CONSTRAINTS:  Free tier only, so assume the shared cache is small and may be
              wiped without warning. Nothing may live only in a cache. We
              run multiple app instances and cannot control which one a user
              reaches.

NON-GOALS:    No code yet. Do not propose a new paid service. Do not propose
              caching anything I did not list.

ACCEPTANCE:   - Every item has one layer and one key
              - Any personalised item's key contains the user identifier
              - At least one item is recommended for no cache, or you
                explain why all five genuinely deserve one
              - Each recommendation references my staleness number

ASK:          Give me the table of decisions first. Then tell me which single
              one I should implement first and why. No code until I pick.
```

## Stuck? Ask AI
```prompt
Explain the difference between a cache that lives inside one app server's memory and a cache that lives in a separate shared server, using an analogy that is not about libraries or books. Then give me five situations and ask me which of the two I should use, without telling me the answers first.
```

```prompt
I do not understand why caching a logged-in page at a CDN is dangerous. Walk me through, request by request, exactly how one user ends up seeing another user's data. Then show me the two ways to make it safe and tell me which one you would pick for a small team.
```

```prompt
My app serves this piece of data: [describe it]. Ask me five questions about it, one at a time, that would let you decide whether it should be cached and where. Do not recommend anything until you have asked all five.
```

## Quiz

### Q1 · L2
Which cache layer is shared by every app server you run?
- (a) The browser cache
- (b) The in-process memory cache
- (c) A shared cache such as Redis ✅
- (d) The CPU cache
> Browser caches are per user, in-process caches are per server. A shared cache exists precisely so that all your instances see one version of the cached answer.

### Q2 · L3
You cache a logged-in dashboard at the CDN under the key `dashboard`. What is the most likely consequence?
- (a) The page loads slower because of the extra hop
- (b) One user is served another user's data ✅
- (c) The cache never produces a hit
- (d) The database gets more load, not less
> If the response depends on who is asking and the key does not, the first cached copy is served to everyone. This is a security incident, and it has happened to real retailers.

### Q3 · L4
Your team caches a lookup table in app-server memory. It works perfectly in testing. What happens in production behind a load balancer with three instances?
- (a) All three instances automatically share the cache
- (b) Users get inconsistent answers depending on which instance serves them ✅
- (c) The cache is disabled automatically
- (d) The load balancer merges the three caches
> Process memory is private to one process, and M00's point stands: memory also forgets on restart. Anything that must agree everywhere belongs in a shared cache.

### Q4 · L4
Which of these is the best candidate for a CDN cache?
- (a) A student's order history page
- (b) The current UPI payment status of an order
- (c) Dish photographs on the public menu ✅
- (d) The rate-limit counter for an API key
> CDNs suit large, public, identical-for-everyone files that change rarely. Payment status must never be stale, order history is personal, and counters must be shared and writable.

### Q5 · L5
An AI proposes adding Redis in front of a function that reads a six-row config table already held in memory and takes 0.3 ms. What is the strongest objection?
- (a) Redis cannot store configuration data
- (b) It replaces a 0.3 ms memory read with a 1 ms network hop and adds a layer to maintain ✅
- (c) Six rows is too much data for a cache
- (d) Configuration should always be cached in the browser instead
> This is the "cache the fastest part" red flag. A cache lookup is itself a hop and a dependency; if the original step was already sub-millisecond, the cache makes things slower and more fragile.

## Go deeper
- [MDN: HTTP caching](https://developer.mozilla.org/en-US/docs/Web/HTTP/Caching)
- [Redis: caching patterns and data types](https://redis.io/docs/latest/develop/use/patterns/)
