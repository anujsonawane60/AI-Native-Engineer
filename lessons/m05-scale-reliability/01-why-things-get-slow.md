---
id: m05-01-why-slow
module: m05-scale-reliability
slug: why-things-get-slow
title: "Why Things Get Slow: Latency Numbers Worth Knowing"
order: 1
duration_min: 25
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

Your team's app works. On your laptop, on campus Wi-Fi, with eleven rows in the database, the menu page appears the instant you click. You demo it to your mentor and nobody has a complaint.

Then forty people open it at 1:10 pm on budget Android phones on college 4G. The page takes nine seconds. Three of them give up and walk to the counter instead. One keeps refreshing, which makes it worse for everybody. Your group's first instinct is "we need a bigger server", and you are about to spend money on exactly the wrong thing — because nothing in those nine seconds was the server running out of CPU.

Slowness is not a mood or a vibe. It is a sum of specific, measurable delays, and almost every one of them is either a trip across a network or a trip to the database. Until you know roughly what each step costs, every performance decision you make — and every performance decision you let an AI make on your behalf — is a guess dressed up as engineering.

### The idea

**A request's time is a budget, and you can only spend a budget sensibly if you know the price of things.**

Four facts carry most of this module. Reading from memory is roughly a thousand times faster than reading from disk. Crossing a network inside one datacentre is roughly a thousand times slower again than reading from memory. Crossing a continent is a hundred times worse than that. And in a normal web app, the place the request actually spent its time waiting is the database.

Memory is near. Disk is far. The network is further. Distance is worst. Hold just those four and you can reason about nine slowness problems out of ten before you open any tool.

### The analogy

Stand behind the canteen counter at 1:10 pm and the distances are physical.

The tray of pre-made samosas is at your elbow. Handing one over costs a second: that is memory. The fresh batch is in the kitchen six steps behind you, so fetching it costs half a minute: that is disk. The spare gas cylinder is in the storeroom across the courtyard, a three-minute walk: that is one network hop to another machine. And the wholesale supplier is in the next town, which is a two-hour round trip: that is a request crossing from Mumbai to Virginia and back.

Nobody running a canteen would walk to the next town to serve one plate. Yet apps do the equivalent constantly, because in code every one of those four actions looks like a single innocent line.

**Where the analogy breaks** in two ways worth naming. First, the real gaps are far more violent than any physical distance: if RAM is one step, disk is a kilometre away and a cross-continent hop is most of the way around the planet. Second, the courtyard does not get longer when more students queue — but a shared database absolutely does get slower under load, because requests start waiting behind each other. That queueing effect is the thing the canteen hides and the next eight chapters are mostly about.

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

### The numbers worth knowing

You do not need these precisely. You need them to the nearest factor of ten, the way you know a samosa costs about ₹15 and not ₹1,500.

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

And three numbers about the human on the other end: under 100 ms feels instant, under 1 second keeps their attention, and past about 10 seconds they assume it is broken and leave. Your nine-second menu page was sitting on the edge of that cliff.

### Where it lives

M00 gave you four boxes — client, network, server, data. Latency does not live in the boxes. It lives in the gaps between them.

Trace the nine seconds honestly and it usually looks like this: 300 ms for the phone to open a connection on 4G, 1,500 ms of cold start because a free-tier server was asleep, 41 separate database queries because the code asked for the dish list and then asked for each dish's details one at a time, and 4 MB of uncompressed photos still downloading after the HTML arrived. Not one of those is a CPU problem. Three of the four are "we crossed a gap more times than we needed to", which is the single most common shape of slowness in student projects and in production systems alike.

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

**The average is a liar.** If nineteen requests take 80 ms and one takes 6 seconds, the average is 376 ms and sounds acceptable. Your p95 is 6 seconds and one student in twenty thinks your app is broken. M07 teaches percentiles properly; for now, never accept an average as evidence that something is fast.

**Bandwidth is not latency.** College 4G may download 5 MB a second and still take 200 ms to get the first byte moving. Big files are a bandwidth problem; many small requests are a latency problem, and they need opposite fixes. Shrinking an image does nothing for 41 queries.

**N+1 is the single most common cause of a slow page.** Forty-one queries at 3 ms each is 123 ms and survivable. The same forty-one queries when your server is in Mumbai and your database is in Virginia is nine seconds, and the code looks identical. M04's query chapter is where you learn to spot it in a query plan.

**Free tiers sleep.** A function that has had no traffic for fifteen minutes may take 1–3 seconds to answer the first request. Your 1:10 pm rush is the worst possible moment to discover this. M06 covers what to do about it.

**Fixing without measuring is gambling with extra steps.** Every number above is an order of magnitude, not a measurement of *your* app. The first job is always to find out where the time went.

### Verify it

Ask an AI to make your page faster and you will get something plausible immediately. Here is how to tell whether it understood the problem.

**Demand a breakdown before a fix.** The right first answer is a table of milliseconds per step: connection, cold start, server processing, each query, payload transfer. If it jumps straight to a solution without asking what your timings are, it is pattern-matching on the word "slow", not reasoning about your system.

**Red flag: caching added to the fastest part.** AI very often proposes caching a computed value that already lives in memory and takes microseconds, while leaving the 41 database calls untouched. Caching the cheap thing adds complexity and saves nothing. Always ask: which of my steps does this remove, and how many milliseconds was that step?

**Red flag: "use a bigger instance" or "add more memory" as the first suggestion.** More RAM does not reduce the number of round trips, and if memory use is climbing steadily it is a leak that a bigger box only postpones. Treat "scale up" as an answer that needs a measured bottleneck behind it.

**Red flag: averages in the report.** If the AI says "average response time improved to 180 ms", ask for p50, p95 and p99. The fix that lowers the average while leaving the tail alone has helped nobody who was actually suffering.

**Red flag: optimising a loop that runs once.** Micro-optimising code is attractive to language models because it looks like real work. Ask what fraction of the total time that code accounted for. If the answer is under 5%, you have been handed a distraction.

Record whichever number you measured in your decision log from M01, because in six weeks the only way to prove a change helped is to have written down what it was before.

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
