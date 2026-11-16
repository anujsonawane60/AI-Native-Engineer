---
id: m05-04-rate-limiting
module: m05-scale-reliability
slug: rate-limiting
title: "Rate Limiting: Token Bucket, Sliding Window and HTTP 429"
order: 4
duration_min: 35
is_free: false
mission: true
volatility: low
last_reviewed: 2026-11-16
---

## Key learnings
- Pick the identity a limit is counted against, and say why IP alone fails in India
- Choose between fixed window, sliding window and token bucket for a given endpoint
- Return 429 with Retry-After instead of a 500 or a silent failure
- Put rate-limit counters in a shared cache, never in one server's memory
- Catch an AI that limits the cheap endpoint and leaves the expensive one open

## Lesson

### The pain

Your app sends an OTP by SMS so students can log in. Each SMS costs you a few paise, and your free tier includes a hundred a day.

At 2 am somebody discovers that your `/api/send-otp` endpoint will send an SMS to any number, as many times as asked, with no limit. It might be a classmate being funny. It might be a bug in your own retry code that loops when the network drops. By 6 am the endpoint has been called nine thousand times, your SMS quota died at 2:04 am, your account is suspended, and no student can log in all day during the lunch rush you built the whole thing for.

Now the same shape with a bigger number. M10 has you calling a language model per request, and each call costs real rupees. One unlimited loop overnight and the month's budget is gone before anyone wakes up — and this time no quota stops it, only a bill.

Nothing was hacked. Your app did precisely what it was asked, nine thousand times, because nobody ever told it how often it was willing to say yes.

### The idea

**A rate limit is a promise you make to yourself about the maximum rate at which you will do expensive work for one identity, and a 429 is how you say no politely.**

The two words that matter are *rate* and *identity*. Rate, because the answer is never "no" but "not that fast" — five OTPs a minute is generous for a real student and useless to a script. Identity, because a limit counted across all users means one abuser degrades everybody, and a limit counted per user means a signup script just makes more users.

Rate limiting is not primarily a security feature. It is a cost and fairness feature that stops a lot of abuse as a side effect. You are protecting a finite resource — SMS credits, model tokens, database connections, your free-tier quota — from being spent faster than you meant.

### The analogy

The canteen sign reads **max 3 plates per person**.

It is not there because three plates is the limit of what a human can eat. It is there because at 1:10 pm there are forty people and a finite number of samosas, and without the sign the first four in the queue leave with everything. The sign keeps the counter usable for the fortieth person.

Notice how the staff enforce it. They keep no ledger of anyone's lifetime consumption. They look at the person in front of them, now, and count the plates on their tray. And when you ask for a fourth they do not throw you out of the canteen — they say "not now, come back in a bit", which is exactly what HTTP 429 with a `Retry-After` header means.

The rule has a burst allowance too. A student collecting food for three hostel-mates genuinely needs six plates at once, so the staff let an occasional burst through while refusing a steady stream. That is a **token bucket**, and it is the shape most real limits take.

**Where the analogy breaks:** the canteen recognises faces. Your server does not. Everything hard about rate limiting is the identity question — the canteen's staff know that this is the same student who came back twice, and your API has to work that out from an IP address, a login, or a key, each of which can be faked, shared or thrown away.

### Picture it

```
  refill: 1 token every 12 s  (5 per minute)
              │
              ▼
        ┌───────────┐
        │ ● ● ● ○ ○ │  bucket, capacity 5, 3 tokens left
        └───────────┘
              │ each request takes 1 token
              ▼
      token available? ──yes──▶ do the work, return 200
              │
              no
              ▼
      429 Too Many Requests + Retry-After: 12
```

Tokens drip into a bucket at a fixed rate up to a maximum, each request spends one, and a request arriving at an empty bucket is refused with a header telling it how many seconds to wait.

### Four algorithms, and which to actually use

| Algorithm | How it counts | Why you would pick it |
|---|---|---|
| Fixed window | Requests per calendar minute, reset at :00 | Simplest; allows a double burst at the boundary |
| Sliding window | Requests in the last 60 seconds, always | Smooth and fair; costs a little more to store |
| Token bucket | Tokens refill over time, request spends one | Allows honest bursts, caps the average |
| Leaky bucket | Requests queue and drain at a fixed rate | When you want to smooth, not refuse |

The fixed window's flaw bites in practice. With a limit of 100 per minute, a client can send 100 at 12:00:59 and 100 more at 12:01:00 — two hundred requests in one second, entirely within the rules. For something expensive, that is not a rounding error.

For almost everything you build this year: **token bucket**, counters in a shared cache, keyed per identity, with different limits per endpoint. Sliding window where exact fairness beats bursts. Fixed window only when the thing you are protecting is cheap.

### What to count against

This is the decision AI gets wrong most often, and in India it is wrong in a specific way.

**By IP address** is the obvious choice and a poor one here. A whole hostel or campus Wi-Fi network frequently shares one public address through carrier-grade NAT, and mobile networks do the same at scale. Limit by IP at five a minute and you have throttled three hundred students as one person — while the actual abuser switches to mobile data.

**By user id** is fair and precise, and useless before login. Signup, login and OTP are exactly the expensive endpoints, and exactly the ones where no user id exists yet.

**By API key** is the right answer for anything machine-to-machine, and it is how every API you consume limits you.

So the working pattern is layered: a generous per-IP limit as a blunt instrument against floods, a tight per-user limit where you have one, and a tight per-phone or per-email limit on the unauthenticated endpoints that cost money. Three cheap limits beat one clever one.

### Where it lives

A rate limiter sits in front of your handler, and its counters live in the shared cache from Chapter 2 — never in process memory, because with three instances a per-process counter gives every abuser three times the promised limit, and a restart forgives everything.

Outward: M04's payment and webhook endpoints need limits and idempotency keys together, since a limit stops the flood and idempotency stops the duplicate charge. M07's alerting turns a spike in 429s into a signal. M10's model calls are the most expensive thing you will ever put a limit in front of. M06 is where you generate the load to test it, and Chapter 8 is the other side of the same coin — what *your* code should do when somebody else returns 429 to you.

### Words to own

| Term | Plain meaning | Example |
|---|---|---|
| Rate limit | Maximum requests per unit of time | 5 OTPs per minute per phone |
| Quota | A total allowance over a long period | 100 SMS per day |
| Burst | A short spike above the average rate | 6 plates for six hostel-mates |
| Token bucket | Tokens refill over time; a request spends one | Capacity 5, refill 1 per 12 s |
| Sliding window | Count over the trailing N seconds | Requests in the last 60 s |
| 429 | HTTP "too many requests" | The polite refusal |
| Retry-After | Header saying how long to wait | `Retry-After: 12` |
| Throttle | Slow down rather than refuse | Queue and drain |
| CGNAT | Many users sharing one public IP | A whole hostel on one address |
| Fail open | Allow requests if the limiter is broken | Availability over protection |
| Fail closed | Refuse requests if the limiter is broken | Protection over availability |

### Tradeoffs and gotchas

**Too strict is an outage you built yourself.** Ten requests a minute per IP will block an entire shared network during your demo. Work out what one enthusiastic real user does in a minute, multiply generously, then check what happens if two hundred of them share an address.

**A 500 is not a rate limit.** If your limiter crashes the request or returns a generic error, every client retries immediately and you have built an amplifier. Return 429, include `Retry-After`, and say something human in the body. Chapter 8 explains why the right status code is what lets a well-behaved client back off instead of pile on.

**Decide fail-open or fail-closed on purpose.** If the cache is down, do you allow everything or refuse everything? For a menu page, fail open — availability beats protecting a cheap read. For the endpoint that sends SMS or calls a paid model, fail closed, because the downside is a bill you cannot undo. Write the choice in your decision log; it will be questioned.

**Limits belong per endpoint, not per app.** One global "100 per minute" either throttles your menu pointlessly or leaves your OTP endpoint wide open. Make the limit proportional to what one call costs: generous on a cached read, tight on anything that sends a message, moves money or calls a model.

**Your own frontend is a plausible attacker.** A retry loop firing on every network blip, an effect that runs twice, a submit button with no disabled state. Most student rate-limit incidents are self-inflicted, which is reason enough to watch your 429 rate when nobody is attacking you.

**Never rate limit a health check or a webhook receiver carelessly.** Blocking your host's health probe takes your app out of rotation. Blocking a payment provider's webhook retries can lose an order confirmation; M04's idempotency keys plus a generous limit on that path is the combination you want.

### Verify it

**Ask what one call costs before asking for a limit.** The first question is: "for each endpoint, what does one request cost me in money, in third-party quota and in database work?" The limits fall out of that table, and an AI proposing numbers without it is inventing them.

**Red flag: counters in process memory.** Look for a plain dictionary or map holding counts inside the app. It tests perfectly and fails in production, giving each instance its own allowance. Ask: "where does this counter live, and what happens with three instances and a restart?"

**Red flag: keyed on IP only.** For Indian student users this is close to a guaranteed incident. Insist on a per-user or per-phone key on the endpoints that cost money.

**Red flag: a 500, a 403, or a silent empty response instead of 429.** Check the status code with your own eyes, and check `Retry-After` is present. A limiter the client cannot understand produces retries, not restraint.

**Red flag — the recurring one: the limit is on the cheap endpoint.** AI will cheerfully limit your cached menu read, which costs 1 ms, while leaving `/api/send-otp` and the model-calling endpoint open. Ask it to rank endpoints by cost per call and confirm the tightest limit sits on the dearest one.

**Red flag: "add more memory" or "upgrade the plan" offered as the fix for abuse.** Scaling up to absorb an unlimited loop is paying for the attack. The fix is a limit.

Then test it rather than reading about it. Loop requests and watch for the exact request number where 429 begins; it should match your limit. Read `Retry-After`, wait that long, confirm the next request succeeds. Use the app as a human and confirm you are never limited. Restart, and check the counter did not reset.

## Walkthrough
1. List every endpoint in your app. Next to each, write what one call costs you: rupees, third-party quota, emails or SMS sent, model tokens, database writes. Be concrete.
2. Sort that list by cost, most expensive first. The top three are the only endpoints that matter for this exercise. Everything else can wait.
3. For your most expensive endpoint, write down what a single enthusiastic real user plausibly does in one minute. Then multiply by three. That is your starting limit.
4. Decide the identity you will count against. If the endpoint needs login, use the user id. If it does not, use whatever natural identity exists — phone number, email — plus a looser IP limit behind it.
5. Write down your fail-open or fail-closed choice for this endpoint, with the reason. Expensive and irreversible means fail closed.
6. Brief an AI to add a token-bucket limiter on that one endpoint, with counters in your shared cache, keyed as you decided, returning 429 with `Retry-After`.
7. Read the diff. Confirm the counter is in the shared cache and not a variable in the process, and confirm the status code is literally 429.
8. Now attack yourself. Send requests in a tight loop and record the number of the first request that is refused. It must match your limit.
9. Read the `Retry-After` value from that response, wait exactly that long, and send one more. It must succeed. If it does not, your refill is wrong.
10. Use the app normally, as a human, for two minutes. You must never see a 429. If you do, your limit is too tight for a real user and your demo will break.
11. Restart your app and immediately replay the loop. If the limit forgives everything after a restart, your counter is in the wrong place — go back to step 7.

## Talk to AI
```prompt
CONTEXT:      I am a student directing AI to build my team's app. Stack:
              [your stack]. Hosting: [your host], free tier, running more
              than one instance. Shared cache: one free-tier Redis-
              compatible instance, small memory, low connection limit.
              Users are Indian students, many on one shared campus Wi-Fi
              network behind CGNAT, and many on mobile data.

              My endpoints and what one call actually costs me:
              1. [endpoint] - costs [e.g. 1 SMS, about 20 paise]
              2. [endpoint] - costs [e.g. one model call, about 2 rupees]
              3. [endpoint] - costs [e.g. one cached read, under 1 ms]
              4. [endpoint] - costs [e.g. one payment attempt]

              For the most expensive one, a single real enthusiastic user
              plausibly makes [X] requests per minute.

GOAL:         Rate limit the expensive endpoints without ever blocking a
              normal student, including when hundreds of them share one
              public IP address.

REQUIREMENTS: 1. Recommend one algorithm per endpoint - fixed window,
                 sliding window or token bucket - and justify each choice
                 against what that endpoint costs
              2. Give exact numbers: capacity and refill rate, or requests
                 per window, per endpoint
              3. State the identity each limit is counted against, and
                 explain what happens on the unauthenticated endpoints
                 where no user id exists yet
              4. Explain explicitly what happens to 300 students sharing
                 one IP through CGNAT under your scheme
              5. Store counters in the shared cache, and tell me what
                 happens with three app instances and after a restart
              6. Return 429 with a Retry-After header and a human-readable
                 body, and show me the exact response
              7. Tell me fail-open or fail-closed for each endpoint, and why

CONSTRAINTS:  Free tier only. The shared cache may be unavailable without
              warning, so the app must behave predictably when it is. Do
              not add a paid API gateway or WAF. Do not limit health
              checks or incoming payment webhooks in a way that could
              lose a confirmation.

NON-GOALS:    No code until I pick the numbers. Do not limit my cheapest
              endpoint. Do not change my authentication.

ACCEPTANCE:   - Every expensive endpoint has a number, an algorithm and an
                identity, each with a one-line justification
              - The tightest limit is on the highest-cost endpoint
              - Counters survive a restart and are shared across instances
              - 300 users on one IP are not blocked as if they were one
              - A refused request returns status 429 with Retry-After
              - A normal user doing normal things is never refused

ASK:          First give me the table of endpoint, cost, algorithm, numbers,
              identity and fail mode. Tell me which number you are least
              confident about and what I should measure to settle it. No
              code until I confirm the table.
```

## Mission

**Objective.** Rate limit the single most expensive endpoint in your own startup's app, and prove by attacking yourself that the limit triggers at the right request, returns 429 with a usable `Retry-After`, survives a restart, and never fires for a normal user.

**Steps.**
1. **Frame** — build the cost table for your own endpoints: one row per endpoint, with what a single call costs in rupees, quota or messages sent. Pick the most expensive one. Name it in one sentence.
2. **Budget** — write down what one real enthusiastic user does in a minute on that endpoint, and how you arrived at the figure. Then write your limit and the multiplier you applied.
3. **Choose the identity** — user id, phone number, email or API key, plus a looser IP limit behind it. Write one sentence on why IP alone is not enough for your users.
4. **Decide the fail mode** — fail open or fail closed if the cache is unavailable, with the reason. Expensive and irreversible means closed.
5. **Brief** — write a Director's Brief using the prompt above with your real cost table and your real numbers. Demand the decision table before any code.
6. **Direct** — have it implement the limiter on that one endpoint. Read the diff and confirm the counter is in the shared cache and the status code is 429.
7. **Attack yourself** — run a loop against the endpoint and record the number of the first refused request. Screenshot the 429 response including its headers.
8. **Verify the refill** — wait exactly the `Retry-After` number of seconds, send one request, and confirm it succeeds. Record both timestamps.
9. **Verify the human path** — use the app normally for two minutes and confirm you are never limited. Note what you did.
10. **Verify persistence** — restart your app, replay the loop immediately, and confirm the limit still applies. If it forgives everything, fix it before you submit.
11. **Document** — one decision-log entry: the endpoint, the limit, the identity, the fail mode, and the cost per call that justified all of it.

**Upload.**
- Your Director's Brief, in full → post caption or an attached file
- Your endpoint cost table, with real rupee or quota figures → text or screenshot
- The AI conversation showing the decision table before any code → share link in `ai_chat_url`
- The working app → deployed URL in `demo_url`
- The 429 response with its full headers, including `Retry-After` → screenshot
- Your loop output showing the exact request number where refusal began → screenshot
- Evidence that waiting `Retry-After` seconds let the next request through → screenshots with timestamps
- Evidence the limit survived an app restart → screenshot
- Your decision-log entry, including the fail-open or fail-closed choice
- Reflection, 150 words max → one tradeoff you chose, and one thing the AI got wrong or left out

**Self-check.** Tick every box before you submit.
- [ ] My cost table has real numbers, not adjectives
- [ ] The endpoint I limited is the most expensive one on that table
- [ ] My limit number is derived from real user behaviour, not picked round
- [ ] The counter lives in the shared cache, and I checked that in the diff
- [ ] My limit is not keyed on IP alone, and I can say why in one sentence
- [ ] A refused request returns status 429, verified with my own eyes
- [ ] `Retry-After` is present, and waiting that long actually works
- [ ] I used the app as a human for two minutes and was never refused
- [ ] The limit still applies after a restart
- [ ] I wrote down fail-open or fail-closed and the reason
- [ ] My reflection names something specific the AI got wrong

**Stretch.** Add a second, looser limit keyed on IP behind your per-identity limit, then explain in 100 words what each of the two limits stops that the other one cannot.

## Stuck? Ask AI
```prompt
Explain a token bucket to me as if I have never programmed, using an analogy that is not about buckets or water. Then walk me through ten requests arriving at irregular times against a bucket of capacity 5 refilling 1 token every 12 seconds, and tell me which ones are refused and why.
```

```prompt
I want to rate limit an endpoint but many of my users share one public IP address through CGNAT on a campus network. Explain, with a worked example of 300 students, exactly what goes wrong if I limit by IP. Then give me three alternative identities to count against, with what each one fails to stop.
```

```prompt
Explain the difference between HTTP 429, 403 and 503, and what a well-behaved client should do differently for each. Then give me five short scenarios and ask me which status code I should return, one at a time, without telling me the answers first.
```

## Quiz

### Q1 · L2
What does HTTP 429 with a `Retry-After` header tell a client?
- (a) The request was forbidden and should not be repeated
- (b) The server is broken and the client should give up
- (c) The client exceeded its rate and may retry after the stated seconds ✅
- (d) The resource has moved elsewhere
> 429 means "not that fast", not "no". `Retry-After` is what lets a well-behaved client wait instead of hammering, which is the whole point of using the right status code.

### Q2 · L3
Your limiter keeps counters in a plain variable inside the app process. You run three instances. What is the effect?
- (a) The limit works exactly as configured
- (b) Each abuser effectively gets three times the limit, and a restart forgives everything ✅
- (c) The load balancer synchronises the counters
- (d) The limiter refuses all requests after a restart
> Process memory is per instance and forgets on restart, which is M00's point again. Rate-limit counters belong in the shared cache for exactly this reason.

### Q3 · L4
Three hundred students on one campus Wi-Fi network share a single public IP. Your limit is 10 requests per minute per IP. What happens at lunch rush?
- (a) Each student gets 10 requests per minute
- (b) The whole campus is throttled as if it were one user ✅
- (c) The limiter automatically detects CGNAT and adjusts
- (d) Only the abusive student is blocked
> This is why IP-only limiting fails in India specifically. The fix is a per-user or per-phone limit on the expensive paths, with a looser IP limit behind it.

### Q4 · L4
You have a cached menu read costing under 1 ms and an OTP endpoint costing an SMS per call. An AI adds a tight rate limit to the menu read and none to the OTP endpoint. What is wrong?
- (a) Nothing — reads are the highest-volume endpoint
- (b) The limit protects the cheapest call and leaves the one that costs money open ✅
- (c) Menu reads cannot be rate limited
- (d) OTP endpoints are limited by the SMS provider automatically
> The same recurring failure as caching the fastest part of the system. Limits should be proportional to cost per call, so the tightest one belongs on the endpoint that spends real money.

### Q5 · L5
Your rate limiter depends on the shared cache. The cache goes down. For the endpoint that sends paid SMS, which behaviour should you choose and why?
- (a) Fail open, because availability always matters more
- (b) Fail closed, because allowing unlimited paid calls creates an unrecoverable bill ✅
- (c) Return 500 so clients retry immediately
- (d) Switch the counter to process memory automatically
> Fail-open suits a cheap read where blocking users is the bigger harm. For irreversible spend the asymmetry flips: a few minutes of refusals is recoverable, a drained SMS or model budget is not.

## Go deeper
- [MDN: HTTP 429 Too Many Requests](https://developer.mozilla.org/en-US/docs/Web/HTTP/Status/429)
- [Cloudflare: rate limiting algorithms compared](https://blog.cloudflare.com/counting-things-a-lot-of-different-things/)
