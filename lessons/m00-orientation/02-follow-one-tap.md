---
id: m00-02-follow-one-tap
module: m00-orientation
slug: follow-one-tap
title: "Follow One Tap: An Order from Phone to Database and Back"
order: 2
duration_min: 25
is_free: true
mission: false
volatility: low
last_reviewed: 2026-11-16
---

## Key learnings
- Trace a single user action through every hop it makes, in order
- Say roughly how long each hop takes, and which hop dominates
- Explain why one tap can produce many requests
- Identify where a request can fail silently
- Use "follow one request" as your default way to understand any system

## Lesson

### The pain

You now know there are four boxes. That is not yet enough to debug anything, because the interesting question is never "what are the parts" — it is **"what happened, in what order, and where did it stop?"**

A senior engineer joining a new company does not read all the code. They pick one action a user can take, and they follow it all the way down and all the way back. Two hours later they understand the system better than someone who read documentation for a week. This chapter teaches you that habit, on the smallest possible example: one tap.

### The idea

**Any action in any app can be understood by following one request through every hop it makes, and asking at each hop: what arrived, what decided, what changed, what left.**

Four questions, repeated at each stop. That is the whole technique. It works on the canteen app, it works on UPI, and it will work in eight years on whatever has replaced today's stack.

### The analogy

A student hands a chit through the canteen window. Follow the chit, not the canteen.

The chit is taken by whoever is at the window. They check it is legible. They check the student has a meal pass. They shout the order to the cook. The cook checks the ingredients are there. Someone marks it in the register. A plate comes out with a token number. The token is called. The student collects.

Notice that the student performed **one** action — handing over a chit — and at least six different things happened in sequence, each of which could fail. Notice also that most of the elapsed time was not work; it was waiting in queues between the steps.

**Where the analogy breaks:** in the canteen you can see the whole process. In software each hop is invisible, so engineers add logs, metrics and traces to make the chit visible as it moves. That is what M07 is about, and it exists purely because you cannot watch the chit.

### Picture it

```
 tap "Order"
     │
     │  1. client builds request          ~1 ms
     ▼
┌─────────────┐
│   browser   │
│   / app     │
└──────┬──────┘
       │  2. DNS: what is the address?    ~20-100 ms  (first time only)
       │  3. TLS: agree on encryption     ~50-200 ms  (first time only)
       │  4. request crosses network      ~30-300 ms
       ▼
┌─────────────┐
│   server    │  5. authenticate user       ~2 ms
│             │  6. validate the order      ~1 ms
│             │  7. recalculate the price   ~1 ms
└──────┬──────┘
       │  8. query / write
       ▼
┌─────────────┐
│  database   │  9. save order row          ~5-50 ms
└──────┬──────┘
       │ 10. return the new order id
       ▼
┌─────────────┐
│   server    │ 11. build response          ~1 ms
└──────┬──────┘
       │ 12. response crosses network     ~30-300 ms
       ▼
   screen updates                          ~16 ms
```

One tap becomes roughly twelve steps. The server's own thinking takes about five milliseconds of it; almost all the elapsed time is network hops and the database.

### Where it lives

Look at the numbers in that diagram and one thing should jump out: **the two network crossings and the database dominate everything else.** The server's logic — the part that feels like "the app" — is a rounding error.

This single observation explains most of the decisions you will meet later in the program:

- **Caching** (M05) exists to skip step 8–10 entirely when the answer has not changed.
- **CDNs** (M02, M05) exist to shorten steps 4 and 12 by putting a copy physically closer to the student.
- **Indexes** (M04) exist to make step 9 take 5 ms instead of 500 ms.
- **Queues** (M04) exist to move slow work out of this path so the student gets a response now and the slow part happens later.
- **Rate limiting** (M05) exists because this whole sequence costs real money, and someone will run it ten thousand times.

Every one of those is an answer to "which hop is slow?" You cannot evaluate any of them until you can see the hops.

### One tap, many requests

A beginner's mental model is one tap, one request. Reality is messier. Opening the canteen app's home screen might fire:

| Request | Why |
|---|---|
| `GET /menu` | the items and prices |
| `GET /me` | who is logged in, so it can greet you |
| `GET /orders?status=active` | "your order is being prepared" banner |
| `GET /banner` | today's announcement |
| 14 image requests | one per dish photo |

That is eighteen round trips for one screen. If each takes 100 ms and they happen one after another, the screen takes 1.8 seconds. If they happen at the same time, it takes about 150 ms. This is why "does it run in parallel?" is one of the highest-value questions you can ask about a slow page — and it is a question AI frequently gets wrong unless you ask.

### Words to own

| Term | Plain meaning | Example |
|---|---|---|
| Hop | One leg of the journey | Client to server is one hop |
| Round trip | Request out plus response back | "Each round trip costs 80 ms" |
| DNS lookup | Translating a name into an address | `canteen.app` → `34.120.5.7` |
| TLS handshake | Agreeing on encryption before sending anything | The padlock appearing |
| Payload | The actual content of a request or response | The order JSON |
| Query | A question asked of the database | "orders where student = 412" |
| Trace | A recorded timeline of one request's hops | Used to find the slow step |
| Blocking | Waiting for one thing before starting the next | 18 sequential requests |
| Parallel | Several things happening at once | 18 simultaneous requests |

### Tradeoffs and gotchas

**Averages lie.** "The menu loads in 200 ms on average" can hide that one student in twenty waits four seconds. You will learn to ask for p95 — the time the slowest 5% experience — in M07. At 1:10 pm, the average student is not the one complaining.

**The first request is much more expensive than the rest.** DNS and TLS only happen once per connection, so your very first action in an app can take three times longer than the identical action thirty seconds later. Students often "fix" a performance problem that was just a cold start.

**A hop can succeed and still lose.** Step 9 can write the order and step 12 can vanish. The server logged success; the student saw failure. Any operation that charges money or creates something must therefore be safe to retry — the concept is called idempotency and it is the whole reason M04's payments chapter exists. You met the symptom in chapter 1; now you can see exactly which hop caused it.

**More hops than you think are not yours.** Payment gateways, SMS providers, map tiles and AI model APIs are all extra round trips to someone else's server, with their own failures and their own bills. When you add an AI feature in M10, you are adding a hop that costs money per call and can take two full seconds.

### Verify it

The habit to build: before accepting any AI-built feature, make it show you the trace.

Ask it to list the hops. Then check three things:

1. **Count the round trips.** If a single screen needs nine sequential requests, ask why they are not parallel, or why there is not one request that returns everything.
2. **Find the slowest hop before optimising anything.** AI is very willing to add caching to the fastest part of your system. Ask: "Which of these steps is slowest, and how do you know?"
3. **Ask what happens if a hop fails halfway.** Specifically: "If the database write succeeds but the response is lost, what does the user see, and what happens if they retry?" If the answer is "they get charged twice", you have found a real bug before shipping it.

You do not need to read code to do any of this. You need to know the hops exist.

## Walkthrough
1. Open any website you use in a desktop browser. Press F12 to open DevTools, then click the **Network** tab. Reload the page.
2. Watch the list fill up. Each row is one hop — one request your single "reload" action produced. Count them. Most sites produce between twenty and a hundred.
3. Sort by the **Time** column. The slowest request is almost never the HTML page itself; it is usually an image, a font, or a third-party script you did not know was there.
4. Click any row and look at **Headers**. The Request URL is where it went; the Status Code is how it turned out; `200` means fine and `404` means the server had no such thing.
5. Look at the **Waterfall** column on the right. Bars that are stacked diagonally happened one after another; bars that line up vertically happened in parallel. Diagonal staircases are where time is being wasted.
6. Find the total at the bottom of the panel: number of requests, data transferred, and load time. This is the honest cost of one tap on that site.
7. Now throttle it. In the Network tab, change "No throttling" to "Slow 4G" and reload. This is what a student on campus data actually experiences. Note how much worse the number gets.
8. Pick the single slowest request and write one sentence explaining what it was and why it was slow. That sentence is your first performance diagnosis.

## Talk to AI
```prompt
I want to understand what really happens when someone performs one action in an app. The action is: [describe it, e.g. "a student taps Order in a canteen app" or "someone sends money on a UPI app"].

List every hop that action makes, in order, from the moment of the tap to the moment the screen changes. For each hop give me:
1. What arrives at this hop
2. What decision is made here
3. What gets changed or stored, if anything
4. A realistic time range in milliseconds
5. What the user would see if this specific hop failed

Then tell me which single hop usually dominates the total time, and which hop is most dangerous to fail halfway. Use plain language and no abbreviations you have not spelled out.
```

## Stuck? Ask AI
```prompt
I opened my browser's Network tab and I do not understand what I am looking at. I am going to paste a few rows. Explain in plain language what each column means and which numbers I should actually care about as a beginner.
```

```prompt
Give me five short scenarios where a web app is slow. For each one, ask me which hop I would investigate first — client, DNS, network, server logic, or database. Do not reveal the answer until I guess, then tell me whether my reasoning was sound, not just whether I was right.
```

```prompt
Explain idempotency to me using the double-order problem: a student taps Order once, the order is created, but the response is lost and she taps again. Explain what idempotency means, then describe the simplest way to prevent the double order. Assume I know nothing about code.
```

## Quiz

### Q1 · L3
In a typical "tap Order" sequence, which parts account for most of the elapsed time?
- (a) The server's validation logic
- (b) The network crossings and the database ✅
- (c) Building the response object
- (d) Redrawing the screen
> The server's own logic is usually a few milliseconds. The two network crossings and the database query dominate, which is why caching and CDNs exist.

### Q2 · L3
A home screen makes 18 requests, each taking about 100 ms. The screen takes roughly 1.8 seconds to appear. What is the most likely cause?
- (a) The requests are running one after another instead of in parallel ✅
- (b) The database has no indexes
- (c) The TLS handshake is repeated 18 times
- (d) The images are too large
> 18 × 100 ms ≈ 1.8 s is the signature of sequential (blocking) requests. Run in parallel, the same 18 requests would finish in roughly the time of the slowest one.

### Q3 · L4
The database write succeeds but the response is lost on the way back. The user retries. Which concept prevents a duplicate order being created?
- (a) Caching
- (b) Rate limiting
- (c) Idempotency ✅
- (d) Load balancing
> Idempotency makes a repeated identical request produce the same single result. Caching, rate limiting and load balancing solve speed, abuse and distribution — not duplication.

### Q4 · L2
Why is the very first request to an app often much slower than the same request made again a few seconds later?
- (a) The server is asleep until the first request
- (b) DNS lookup and the TLS handshake happen once per connection ✅
- (c) Databases always run slowly on their first query
- (d) The client has to download the whole app again
> DNS resolution and TLS negotiation are one-time setup costs for a connection. Subsequent requests reuse it, which is why cold starts mislead beginners into "fixing" a non-problem.

### Q5 · L4
You are told "the menu loads in 200 ms on average" but students complain it is slow at lunchtime. What should you ask for?
- (a) The median load time
- (b) The p95 load time, and the time range it was measured over ✅
- (c) The total number of requests
- (d) The size of the database
> An average hides the tail. p95 tells you what the unluckiest 5% experience, and the lunchtime complaints are coming from exactly that group.

## Go deeper
- [Chrome DevTools: Inspect network activity](https://developer.chrome.com/docs/devtools/network/)
- [High Performance Browser Networking — Primer on Latency and Bandwidth](https://hpbn.co/primer-on-latency-and-bandwidth/)
