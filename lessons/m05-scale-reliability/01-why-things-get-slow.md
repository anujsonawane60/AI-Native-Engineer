---
id: m05-01-why-slow
module: m05-scale-reliability
slug: why-things-get-slow
title: "Why Things Get Slow: Latency Numbers Worth Knowing"
order: 1
duration_min: 15
is_free: false
mission: false
volatility: low
last_reviewed: 2026-11-16
---

## Key learnings
- Recall the four latency numbers that explain most slowness
- Split a slow request into network time, server time and database time
- Explain why memory is roughly a thousand times faster than disk
- Read p95 instead of the average when judging speed
- Spot an AI "fix" aimed at the part of the system that was never slow

## Lesson

### The pain

Your app is instant on your laptop with eleven rows in the database. Then forty students open it at 1:10 pm on budget Android phones on college 4G, the page takes nine seconds, and three walk to the counter instead. Your team's instinct is "a bigger server" — and nothing in those nine seconds was CPU.

### The idea

**A request's time is a budget, and you can only spend a budget sensibly if you know the price of things.**

Memory is roughly a thousand times faster than disk. One network hop inside a datacentre is roughly a thousand times slower again. Crossing a continent is a hundred times worse than that. And in a normal web app, the place the request waited is the database: memory is near, disk is far, the network is further, distance is worst.

### The analogy

Behind the canteen counter the distances are physical. The samosas at your elbow are memory: one second. The fresh batch six steps away in the kitchen is disk: half a minute. The gas cylinder across the courtyard is one network hop: three minutes. The supplier in the next town is Mumbai to Virginia: a two-hour round trip. Nobody walks to the next town to serve one plate, yet apps do the equivalent constantly, because in code all four look like one innocent line.

**Where the analogy breaks:** the courtyard does not get longer when more students queue, while a shared database does get slower under load, because requests wait behind each other — the queueing effect the rest of this module is about.

### Picture it

```
  1 ns  █                      CPU reads its own cache
100 ns  █                      read 1 KB from RAM
100 µs  ██                     read 4 KB from an SSD
500 µs  ███                    one hop to another box in the same
                               datacentre
  5 ms  █████                  one indexed query, including the hop
 60 ms  ████████               Mumbai ──▶ Singapore ──▶ Mumbai
220 ms  ███████████            Mumbai ──▶ Virginia ──▶ Mumbai
```

Each step down the ladder is not slightly slower than the one above it but hundreds or thousands of times slower, which is why the two bottom rungs — anything that leaves the building — dominate a request's total time.

To the nearest factor of ten, as you know a samosa costs ₹15 and not ₹1,500:

| What happens | Roughly | Canteen equivalent |
|---|---|---|
| Read 1 KB from RAM | 100 nanoseconds | samosa already on the counter |
| Read 4 KB from an SSD | 100 microseconds | walk to the kitchen |
| One hop to a nearby machine | 0.5 ms | cross the courtyard |
| One simple indexed query | 1–10 ms | courtyard, plus a short wait |
| One unindexed query on 100k rows | 200 ms+ | search the whole storeroom |
| Mumbai to Singapore, there and back | ~60 ms | next town |
| Mumbai to Virginia, there and back | ~220 ms | next state |
| A phone on slow 4G, before any work | 100–300 ms | the student is still walking in |

For the human: under 100 ms feels instant, under 1 second keeps attention, past 10 seconds they leave. Your nine-second page sat on that cliff edge.

### Where it lives

Latency lives in the gaps between M00's four boxes, not in the boxes. Trace the nine seconds and it is usually 300 ms to open a connection on 4G, 1,500 ms of cold start on a sleeping free-tier server, 41 queries because the code asked for the dish list and then each dish separately, and 4 MB of uncompressed photos still arriving. Three of those four are "we crossed a gap more times than we needed to".

### Words to own

| Term | Plain meaning | Example |
|---|---|---|
| Latency | How long one thing takes, start to finish | 220 ms for a round trip to Virginia |
| Throughput | How many things finish per second | 300 menu loads a minute at lunch |
| Round trip | Out across the network and back once | Phone asks, server answers |
| Hop | One leg between two machines | App server to database |
| p50 / median | The middle request | Half of all requests are faster |
| p95 | The slow tail — 1 request in 20 is worse | The number your angriest user feels |
| N+1 query | One query, then one more per row it returned | 1 + 40 queries for 40 dishes |
| Cold start | The first request pays the setup cost | A sleeping free-tier function |
| Payload | How many bytes actually travel | A 2 MB uncompressed hero image |
| Time to first byte | The wait before anything at all arrives | Mostly network plus server thinking |

### Tradeoffs and gotchas

**The average is a liar.** Nineteen requests at 80 ms and one at 6 seconds averages 376 ms, while your p95 is 6 seconds and one student in twenty thinks your app is broken.

**Bandwidth is not latency.** College 4G may download 5 MB a second and still take 200 ms to get the first byte moving, so shrinking an image does nothing for 41 queries.

**N+1 is the commonest cause of a slow page.** Forty-one queries at 3 ms each is 123 ms; the same code with your database in Virginia is nine seconds.

**Free tiers sleep.** Fifteen quiet minutes can cost 1–3 seconds on the first request, and 1:10 pm is the worst moment to find out.

**Every number above is an order of magnitude, not a measurement of *your* app.** Fixing without measuring is gambling with extra steps.

### Verify it

- **Demand a breakdown before a fix:** milliseconds per step — connection, cold start, server, each query, payload. Jumping to a solution means it pattern-matched on "slow".
- **Red flag: caching added to the fastest part.** Ask which step the fix removes and what that step cost.
- **Red flag: "a bigger instance" or "more memory" first.** More RAM does not reduce round trips, and climbing memory is a leak.
- **Red flag: averages in the report.** Ask for p50, p95 and p99.
- **Red flag: optimising a loop that runs once.** Ask what share of total time it accounted for; under 5% is a distraction.
- Record your measurement in your M01 decision log; in six weeks it is the only proof a change helped.

## Walkthrough
1. Open your team's deployed app on a phone, not a laptop, and turn Wi-Fi off so you are on real mobile data. Time how long the main page takes, by stopwatch, three times.
2. Reload once more and time it again. Note whether the second load was dramatically faster — if it was, you have just seen a cold start, not a slow app.
3. Open the same page on a laptop, open the browser developer tools, and go to the Network tab. Reload with the tab open.
4. Sort the request list by time and write down the slowest three requests, with their durations and sizes. One of them is probably an image; one is probably your own API.
5. Find the "waiting" or "TTFB" number on your own API request. That is network plus server plus database. Everything in this chapter is about shrinking that one figure.
6. In the same dev tools, switch network throttling to "Slow 4G" and reload. Write down the new total. This is closer to what your users actually experience than anything you have measured so far.
7. Count the requests your page makes. If it is over about thirty, note the count — on a high-latency connection each one costs you.
8. Now look at your server or database dashboard and find the query count for that single page load. If loading a list of ten things ran eleven or more queries, you have found an N+1.
9. Write a one-paragraph latency budget in your notes: of the total time, roughly how many milliseconds went to network, to cold start, to queries, and to images. Guess where you must, but assign every millisecond.
10. Pick the single largest number in that budget and write one sentence naming it as your bottleneck. Everything you do in the rest of this module should be aimed at that number and nothing else.

## Talk to AI
```prompt
CONTEXT:      I am a student building a web app with a small team. Our main
              page feels slow on budget Android phones on 4G in India, and
              fine on a laptop on Wi-Fi. Stack: [your stack]. Hosting:
              [your host], free tier. Database: [your database], free tier.
              Here are the measurements I have actually taken:
              - total page load on slow 4G: [X] seconds
              - time to first byte on my own API call: [X] ms
              - number of requests the page makes: [X]
              - total bytes transferred: [X] MB
              - number of database queries for one page load: [X]
              - second load immediately after the first: [X] seconds

GOAL:         Understand where the time is actually going, in milliseconds,
              before changing anything.

REQUIREMENTS: 1. Turn my numbers into a latency budget: a table of steps
                 with estimated milliseconds each, summing to my measured
                 total
              2. Name the single largest contributor and say how confident
                 you are, given what I measured and what I did not
              3. Tell me exactly which further measurement would raise that
                 confidence, and how to take it
              4. Only then propose fixes, ranked by milliseconds saved per
                 hour of work
              5. For each fix, state which step in the table it removes

CONSTRAINTS:  Free tiers only, no paid upgrades. Assume the users are on
              slow 4G with 150-300 ms round trip times. I cannot change
              hosting regions this week.

NON-GOALS:    Do not rewrite my code. Do not suggest a caching layer yet.
              Do not suggest a bigger server or more memory unless you can
              point at a measurement that justifies it.

ACCEPTANCE:   - Every row in your table has a number, not an adjective
              - The rows sum to roughly my measured total
              - You have told me at least one thing my measurements cannot
                yet distinguish between
              - Your top-ranked fix names the step it eliminates

ASK:          First give me the budget table and the bottleneck. Do not give
              me any fix until you have shown me the table.
```

## Stuck? Ask AI
```prompt
Explain, as if to someone who has never seen a computer, why reading from memory is about a thousand times faster than reading from a disk, and why sending a request to another country is far worse than both. Use one everyday analogy and keep it to 200 words. Then give me the three numbers you would want me to memorise and nothing else.
```

```prompt
I have a list page that loads ten items and I am told it makes eleven database queries. Explain what an N+1 query is using this example, show me what the query log looks like in the bad case and the good case, and tell me three ways I could notice this problem without anyone telling me it exists.
```

```prompt
Give me five short scenarios where a web page is slow. For each one, tell me the single measurement I should take first to tell the difference between a network problem, a server problem, a database problem and a payload-size problem. Ask me to answer before you reveal your reasoning.
```

## Quiz

### Q1 · L2
Roughly how much slower is reading from an SSD than reading the same data from RAM?
- (a) About 10 times slower
- (b) About 1,000 times slower ✅
- (c) About the same, since both are solid state
- (d) About 100,000 times slower
> Memory is around 100 nanoseconds, an SSD read around 100 microseconds. That thousand-fold gap is the reason caches exist at all.

### Q2 · L3
A page takes 9 seconds on slow 4G. The team's first action should be to:
- (a) Upgrade to a larger server instance
- (b) Add a caching layer in front of everything
- (c) Break the 9 seconds down into milliseconds per step ✅
- (d) Rewrite the page in a faster framework
> You cannot fix a bottleneck you have not located. Every other option is a guess that may well target a step that was only costing 20 ms.

### Q3 · L4
Nineteen requests take 80 ms and one takes 6 seconds. Which statement is the most useful summary?
- (a) Average response time is 376 ms, which is acceptable
- (b) p95 is around 6 seconds, so one user in twenty sees a broken page ✅
- (c) The system is fast because the median is 80 ms
- (d) Throughput is the metric that matters here
> The median and the average both hide the tail. p95 is the number that corresponds to a real person's real experience, which is why M07 builds alerting on it.

### Q4 · L4
Your app server is in Mumbai and your database is in Virginia. The dish list page runs 1 query for the list plus 1 per dish for 40 dishes. What is the dominant cost?
- (a) The size of the JSON response
- (b) 41 round trips at roughly 220 ms each ✅
- (c) The CPU time spent formatting the rows
- (d) The database's disk reads
> 41 cross-continent round trips is about nine seconds of pure waiting. The same code against a nearby database would cost around 120 ms — identical logic, eighty times the latency.

### Q5 · L4
An AI reviews your slow checkout page and proposes caching a total that is recalculated in memory on each request. What is wrong with this?
- (a) Nothing — caching is always an improvement
- (b) It caches a microsecond-scale step while the real waiting happens elsewhere ✅
- (c) In-memory values can never be cached
- (d) Caching totals is unsafe because money is involved
> This is the classic failure mode: caching is added to the fastest part of the system. Always ask which measured step the fix removes and how many milliseconds that step cost.

## Go deeper
- [Latency numbers every programmer should know](https://samwho.dev/numbers/)
- [MDN: Understanding latency](https://developer.mozilla.org/en-US/docs/Web/Performance/Understanding_latency)
