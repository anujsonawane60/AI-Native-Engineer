---
id: m00-02-follow-one-tap
module: m00-orientation
slug: follow-one-tap
title: "Follow One Tap: An Order from Phone to Database and Back"
order: 2
duration_min: 15
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

Knowing the four boxes is not enough to debug anything, because the interesting question is never "what are the parts" — it is **"what happened, in what order, and where did it stop?"** A senior engineer joining a new company does not read all the code. They pick one user action and follow it all the way down and back, and two hours later they understand the system better than someone who read the docs for a week.

### The idea

**Any action in any app can be understood by following one request through every hop it makes, and asking at each hop: what arrived, what decided, what changed, what left.**

Four questions at each stop. That is the whole technique, and it will still work in eight years on whatever replaces today's stack.

### The analogy

A student hands a chit through the canteen window. Follow the chit, not the canteen. Whoever is at the window checks it is legible and that she has a meal pass, then shouts the order to the cook. The cook checks the ingredients. Someone marks the register. A plate comes out with a token, the token is called, she collects. One action, six steps in sequence, each of which could fail — and most of the elapsed time was not work, it was queueing between steps.

**Where the analogy breaks:** in the canteen you can watch the whole process. In software each hop is invisible, so engineers add logs and traces to make the chit visible as it moves.

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

One thing jumps out of those numbers: **the two network crossings and the database dominate everything else.** The server's logic — the part that feels like "the app" — is a rounding error. That explains most of the decisions you will meet later:

- **Caching** (M05) skips steps 8–10 when the answer has not changed.
- **CDNs** (M02, M05) shorten steps 4 and 12 with a copy closer to the student.
- **Indexes** (M04) make step 9 take 5 ms instead of 500 ms.
- **Queues** (M04) move slow work out of this path, so the response arrives now.
- **Rate limiting** (M05) exists because each run costs real money.

Every one answers "which hop is slow?" — unanswerable until you see the hops.

One tap is also rarely one request. The canteen home screen might fire:

| Request | Why |
|---|---|
| `GET /menu` | the items and prices |
| `GET /me` | who is logged in, so it can greet you |
| `GET /orders?status=active` | "your order is being prepared" banner |
| `GET /banner` | today's announcement |
| 14 image requests | one per dish photo |

Eighteen round trips for one screen. Sequential at 100 ms each, the screen takes 1.8 seconds; in parallel, about 150 ms. "Does it run in parallel?" is one of the highest-value questions about a slow page, and AI often gets it wrong unless you ask.

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

**Averages lie.** "200 ms on average" hides one student in twenty waiting four seconds. Ask for p95 — the time the slowest 5% experience. At 1:10 pm, the average student is not the one complaining.

**The first request costs far more than the rest.** DNS and TLS happen once per connection, so your first action can take three times longer than the identical action thirty seconds later. Students often "fix" a cold start.

**A hop can succeed and still lose.** Step 9 writes the order, step 12 vanishes: the server logged success, the student saw failure. Anything that charges money must be safe to retry — that property is called idempotency.

**More hops than you think are not yours.** Payment gateways, SMS providers, map tiles and AI model APIs are round trips to someone else's server, with their own failures and their own bills. An AI feature adds a hop that costs money per call.

### Verify it

Before accepting any AI-built feature, make it list the hops. Then check three things.

1. **Count the round trips.** If one screen needs nine sequential requests, ask why they are not parallel, or why one request does not return everything.
2. **Find the slowest hop before optimising.** AI will happily cache the fastest part of your system. Ask: "which step is slowest, and how do you know?"
3. **Ask what happens if a hop fails halfway.** "If the database write succeeds but the response is lost, what does the user see, and what if they retry?" If the answer is "charged twice", you found a real bug before shipping.

You need no code reading for any of this. You need to know the hops exist.

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
