---
id: m05-02-cache-layers
module: m05-scale-reliability
slug: the-cache-layers
title: "The Cache Layers: Browser to CDN to App to Redis to Database"
order: 2
duration_min: 15
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

Your team adds caching and the AI writes it in four minutes. Then Priya, logged in, sees Rahul's name in the header, because the page was cached at the edge with Priya's greeting baked in. The chef marks paneer sold out and the menu keeps offering it for eleven minutes. A 2 am restart makes the first forty requests slow again, because the cache lived in that server's memory. Three bugs, one mistake: the wrong layer.

### The idea

**A cache is a copy of an answer kept closer to whoever keeps asking — and the layer you put it in decides who can see it, how fast it is, and whether you can ever take it back.**

Five layers run from the user outward: the browser, the CDN at the edge, your app server's own memory, a shared cache like Redis that all your servers use, and the database, the truth everything else copies. Closer to the user is faster and cheaper, and gives you less control: a copy on a phone in Nashik is not something you can delete.

### The analogy

A student photographs the menu board: the **browser cache**, instant, private, beyond the canteen's reach. A copy pinned at the hostel gate: the **CDN**, shared by one area, updated only when somebody replaces the sheet. A cashier's apron note of today's specials: the **in-process cache**, fastest in the building and different in every apron. The whiteboard every cashier reads: **Redis**, one version for all the staff, wiped when the kitchen says so. The kitchen's stock ledger: the **database**, slow, and the only thing actually right.

**Where the analogy breaks:** food announces when it has gone off, and stale cached data looks absolutely fresh — which is why the next chapter is about expiry.

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

Which layer a thing belongs in is the decision you will keep making:

| Layer | Good for | Never put here |
|---|---|---|
| Browser | Logos, fonts, CSS, JS, a user's own last-seen list | Anything that must change instantly |
| CDN | Images, menus, public product pages, static files | Anything personalised or private |
| In-process memory | Config, feature flags, tiny lookup lists | Anything that must agree across servers |
| Redis / shared | Sessions, rate-limit counters, query results, computed feeds | The only copy of anything you cannot recreate |
| Database | The truth | — |

One rule does most of the work: **personalised data never goes in a shared or public layer.** That is the Priya bug, and it is a security incident rather than a caching one.

### Where it lives

Caches sit in the gaps of M00's four boxes: the browser removes a network hop, the CDN shortens it, Redis replaces a slow database hop with a fast one. You have met three already: M02's static files on a CDN, M04's uploads in object storage, M07's hit rate on a dashboard. For a student project: static files and images on the CDN your host gives you, one shared cache for the few queries that hurt, nothing in process memory.

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

**Every cache is a second copy of the truth, and two copies disagree eventually.** You can only bound it: a dish photo can be a day old, a sold-out flag cannot be eleven minutes old, a UPI status cannot be stale at all.

**Caching the wrong thing costs more than not caching.** A lookup is a hop, a serialisation step and code to maintain: cache what is slow *and* asked for often.

**In-process caches are a trap because they work on your laptop.** With two instances, half your users see one answer and half another, intermittently.

**A cache is not storage.** Free tiers evict when full and can be wiped on restart, so anything you cannot recompute must not live only in a cache. And public caching of a logged-in page is a real outage pattern: large retailers have served one customer's basket to another.

### Verify it

- **Ask which layer, and why that one:** per piece of data, which of the five, what breaks in the other four, how stale it may be. One layer for everything means it has not engaged.
- **Red flag: a personalised response cached without the user in the key.** `dashboard` rather than `dashboard:user:42` leaks somebody's data, and AI does this routinely.
- **Red flag: process memory with no mention of multiple servers.** A good answer names the inconsistency and the cold start.
- **Red flag: caching the step that was never slow.** Make it name the step removed and its cost in milliseconds.
- **Red flag: a cache with no removal story.** "It expires" has to be a deliberate number, not a shrug.
- Prove the hit: load twice and compare timings, read the headers for a cache status, log hits and misses. Then log the choice: "menu JSON in Redis, 60 s, because the chef's sold-out flag must land within a minute".

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
