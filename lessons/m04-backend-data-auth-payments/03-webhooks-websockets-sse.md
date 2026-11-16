---
id: m04-03-webhooks-websockets-sse
module: m04-backend-data-auth-payments
slug: webhooks-websockets-sse
title: "Webhooks, WebSockets and SSE: When the Server Talks Back"
order: 3
duration_min: 25
is_free: false
mission: false
volatility: low
last_reviewed: 2026-11-16
---

## Key learnings
- Choose between polling, SSE, WebSockets and webhooks for one real feature
- Explain why a webhook is another service calling your server, not the reverse
- Say what makes a webhook endpoint dangerous and what must guard it
- Pick polling on purpose, with a number, instead of by accident
- Check that AI's realtime proposal survives a dropped 4G connection

## Lesson

### The pain

The canteen app ships with a live order tracker. The student taps "Order", and the screen is supposed to move from Placed to Cooking to Ready without them touching anything.

The first version asks the server "any news?" every two seconds. It works. Then 400 students use it during the lunch rush, the free tier's request quota is gone by Wednesday, and phones get warm because the radio never sleeps.

The second version uses WebSockets. It works beautifully on the laptop. On a hostel 4G connection the socket drops every few minutes when the phone switches towers, and because nobody wrote reconnection logic the tracker silently freezes on "Cooking" for an order that was collected twenty minutes ago. No error, no spinner, just a lie on the screen.

The third problem is worse and arrives with payments in chapter 17. The payment provider needs to tell your server that money arrived. It cannot ask your phone. It calls a URL on your server that you published — and if that URL trusts whatever arrives, anyone on the internet can tell your server that they paid.

### The idea

**Normally the client asks and the server answers. These four patterns are the ways something other than that happens — and each one trades effort against freshness.**

Polling: the client keeps asking on a timer. Simple, cacheable, wasteful.

Server-Sent Events (SSE): the client opens one long-lived connection and the server pushes text down it whenever it likes. One direction only.

WebSockets: a two-way pipe stays open, and either side can speak at any moment.

Webhooks: a different company's server calls *your* server when something happens in their system. Not a browser thing at all.

### The analogy

Four ways to find out your canteen order is ready.

**Polling** is walking to the counter every thirty seconds to ask. You will find out, eventually, and you will do a lot of walking. Half your trips get the same answer as the last one.

**SSE** is standing at the counter with the server calling out token numbers. You listen; you do not speak. One direction, and it works well because that is genuinely all you need.

**WebSockets** is a staff member with a walkie-talkie who can both tell you your food is ready and ask whether you want extra chutney, instantly, in both directions.

**Webhooks** are the bank's representative coming to the canteen's office, unannounced, to say "payment for token 41 has cleared". The canteen did not ask. The canteen does, however, check the representative's ID card before updating the register — because anyone can walk in and claim a payment cleared.

**Where the analogy breaks:** a person standing at the counter does not silently stop hearing. A phone's connection does. The hardest part of every realtime feature is not the pushing; it is noticing that the pipe broke and recovering — a problem the physical canteen simply does not have.

### Picture it

```
 POLLING              SSE / WEBSOCKET        WEBHOOK
 client  server       client  server         payment   your
   │       │            │       │            provider  server
   │─ask──▶│            │─open─▶│               │        │
   │◀─no───│            │       │               │        │
   │─ask──▶│            │◀─"cooking"─           │─POST──▶│
   │◀─no───│            │       │               │ signed │
   │─ask──▶│            │◀─"ready"───           │        │
   │◀─yes──│            │       │               │◀─200───│
   │       │            │ (drops silently       │        │
   │       │            │  on tower switch)     │        │
```

Polling repeats a question; SSE and WebSockets hold one connection open and push down it; a webhook is an entirely separate server making a request to yours, which is why it must be authenticated.

### Where it lives

Polling, SSE and WebSockets all live on the client-server seam from chapter 1 — they are different ways of using it.

A webhook is somewhere else entirely, and this is the part that confuses everyone the first time. Your server stops being only a server and becomes a client's target. Razorpay's servers, or GitHub's, or a WhatsApp Business provider's, make an HTTP request to a path you own. That path is open to the internet, has no logged-in user behind it, and is therefore one of the most security-sensitive routes you will ever write. Chapter 17 is where it meets money; M07 is where it meets OWASP.

Background jobs in chapter 16 are the other half of this picture: a webhook should do almost nothing except verify the signature, record the fact, and queue the real work.

### Choosing, with a number

The honest question is: **how stale can this screen be before a user is annoyed?**

| Feature | Tolerable staleness | Right answer |
|---|---|---|
| Canteen menu sold-out flag | 60 seconds | polling |
| Order status tracker | 5–10 seconds | polling or SSE |
| Live chat between two users | under 1 second | WebSockets |
| Admin dashboard counters | 30 seconds | polling |
| "Payment received" | must be exact | webhook |
| Typing indicator | instant | WebSockets |

Four of six are polling. That is not laziness; it is the correct answer most of the time for a small team. A poll every ten seconds on one screen is about six requests a minute per active user, it works through flaky networks because each request is independent, it is cacheable, it is trivially debuggable in the Network tab, and it survives your server restarting.

The number matters more than the technology. "We poll every 10 seconds because a 10-second delay on order status is invisible to a student standing in a queue" is an engineering decision you can defend in M11. "We used WebSockets because it's realtime" is not.

### Words to own

| Term | Plain meaning | Example |
|---|---|---|
| Polling | Client asks repeatedly on a timer | every 10 seconds |
| Long polling | Server holds the request open until news | older chat apps |
| SSE | One-way stream of text pushed from server | order status updates |
| WebSocket | Two-way connection, both sides can speak | live chat, typing dots |
| Webhook | Another service calls an endpoint you own | payment captured |
| Callback URL | The address you give them to call | `/api/webhooks/payments` |
| Signature | Proof the webhook really came from them | HMAC header |
| Replay | The same webhook delivered twice | must be handled |
| Heartbeat | Periodic ping that proves a pipe is alive | every 30 seconds |
| Reconnect / backoff | Retrying a dropped connection, slower each time | 1s, 2s, 4s |

### Tradeoffs and gotchas

**Every realtime connection costs you a slot on a free tier.** Free-tier limits are usually expressed in concurrent connections or execution hours, and a WebSocket holds one open for as long as the tab is open. Two hundred students with the app open is two hundred held connections. Polling uses a connection for 80 milliseconds and lets go.

**Serverless and WebSockets fight each other.** Most free serverless platforms (chapter 5's subject) bill and time out per invocation and are a poor host for a connection that must stay open for an hour. If AI proposes WebSockets on a serverless free tier without mentioning this, it has not thought about where the code runs.

**A dropped socket does not look like an error.** It looks like nothing happening, which is indistinguishable from "nothing has happened". Any WebSocket or SSE feature needs a visible connection state, automatic reconnection with backoff, and a full refetch after reconnect to catch up on what was missed. That is three pieces of work most first drafts omit entirely.

**An unverified webhook endpoint is a public write to your database.** The URL is guessable, there is no session, and the body is JSON someone else wrote. Without signature verification, anybody can post `{"status":"paid","order_id":91}` and get free food. This is the specific failure chapter 17 is built around.

**Webhooks arrive twice, out of order, or late.** Providers retry when they do not get a prompt `200`, which means your handler will see duplicates. Design for that from the first line. Also: do the slow work in a background job, because a provider that waits eight seconds for your response will time out and retry, giving you a second copy.

**Do not poll every second to look impressive.** At one second, 400 users is 24,000 requests a minute. You will exhaust a free tier before lunch ends, and nobody can tell the difference from ten seconds.

### Verify it

Five checks on any realtime work AI hands you.

1. **Ask for the staleness number first.** "How out of date can this be?" If the AI never asks and jumps straight to WebSockets, re-brief it. The number chooses the technology.
2. **Kill the network and watch the screen.** Turn off Wi-Fi mid-session. A correct implementation shows a disconnected state and recovers when you turn it back on. A typical first draft freezes while looking perfectly healthy, which is worse than an error.
3. **Check for a refetch after reconnect.** Reconnecting is not enough — the updates that happened while you were offline are gone forever. The client must re-read current state once the pipe is back.
4. **For webhooks, demand signature verification before anything else.** Read the handler. The first thing it does must be verifying the provider's signature header against your secret, and rejecting with `400` if it fails. If the handler reads the body first and verifies later, or not at all, reject the work.
5. **For webhooks, check it is safe to receive twice.** Send the same payload twice yourself. The second one must change nothing. If the order is marked paid twice or the student is credited twice, you have the bug chapter 17 is named after.

Red flags: the phrase "we can add reconnection later"; a webhook handler that does database writes, sends an email and generates a PDF before replying; WebSockets proposed for a feature where the data changes twice an hour; any realtime design that has not been tested with the network throttled to Slow 4G; a webhook secret pasted into the code instead of read from an environment variable, which M07 will fail you for.

## Walkthrough
1. Pick one feature in your own product where a screen must change without the user doing anything. Write it down in one sentence.
2. Answer one question in writing: how many seconds out of date can this screen be before a real user is annoyed? Commit to a number, not "as fast as possible".
3. Using that number, choose polling, SSE or WebSockets. Write the one-sentence justification that names the number.
4. Brief AI to implement the simplest version that meets your number. If your number is five seconds or more, explicitly forbid WebSockets in the non-goals.
5. Open the Network tab and watch the traffic for one minute. Count the requests or find the single held connection. Multiply by your expected number of concurrent users and compare that against your free tier's documented limit.
6. Now break it. Turn Wi-Fi off for thirty seconds while the screen is open, then back on. Record exactly what the screen showed during and after the outage.
7. If it froze without saying anything, that is your bug. Write a precise iteration: symptom, expected versus actual, and the scope limited to the connection handling.
8. Throttle the network to Slow 4G in dev tools and repeat step 6. Many implementations survive a clean disconnect and fail on a slow one.
9. Separately, find the webhook documentation for one provider you will use later — a payment gateway or a messaging provider — and locate the page about verifying signatures. Read it and write down the header name they use.
10. Ask AI to describe, without writing code, what your webhook handler must do in order, from the first line to the `200` response. Check that verification is step one and that the slow work is queued rather than done inline.

## Talk to AI
```prompt
CONTEXT:      Three-person student team, free tiers only, four months.
              Users are on budget Android phones on hostel 4G in India,
              where connections drop when the phone switches towers.
              Product: [one sentence]
              The feature: [describe the one screen that must update
              without the user acting]
              Expected concurrent users at peak: [your honest number]
              Where our server code runs: [serverless functions / a single
              always-on server / don't know yet]

GOAL:         Decide how this screen gets its updates, and build the
              simplest thing that is honest about failure.

REQUIREMENTS: 1. Before recommending anything, ask me how many seconds out
                 of date this screen may be, and use my answer
              2. Compare polling, SSE and WebSockets for THIS feature, with
                 the free-tier cost of each at my peak user count
              3. Say explicitly whether my hosting choice can even hold a
                 long-lived connection open
              4. Whatever you recommend, specify the behaviour when the
                 connection drops: what the user sees, how reconnection is
                 attempted, and how missed updates are caught up
              5. Give me the arithmetic: requests per minute per user, times
                 my peak users, against the documented free-tier limit

CONSTRAINTS:  Free tier only, and I must not exceed its request or
              connection limits during a one-hour lunch peak. The team has
              never built a realtime feature. Must work on an intermittent
              4G connection, not just on Wi-Fi.

NON-GOALS:    No chat feature. Do not introduce a paid realtime service. Do
              not change any other screen. No optimistic UI yet.

ACCEPTANCE:   - Your recommendation cites my staleness number
              - With the network turned off, the screen shows a clear
                disconnected state rather than freezing
              - When the network returns, the screen shows current data,
                including changes that happened while offline
              - Peak traffic stays inside the free tier, shown with
                arithmetic

ASK:          First give me two options with their tradeoffs and their
              free-tier arithmetic, then your recommendation and the one
              number that decided it. No code until I pick one.
```

## Stuck? Ask AI
```prompt
Explain the difference between a webhook and a WebSocket to someone who keeps mixing them up. Use one concrete example of each from an Indian food delivery app, and say clearly which one involves another company's server calling mine.
```

```prompt
I have a screen that polls my server every two seconds and I am burning through my free tier. Ask me questions about what the screen shows and who looks at it, then help me work out the slowest polling interval that no user would notice. Do not give me code.
```

```prompt
My realtime screen freezes silently when the network drops instead of showing an error. Walk me through the three separate things a correct implementation must do — show connection state, reconnect with backoff, and catch up on missed updates — and for each one tell me how I would test that it actually works from my own phone.
```

## Quiz

### Q1 · L2
What is a webhook?
- (a) A connection your browser holds open to receive updates
- (b) Another service making an HTTP request to an endpoint you own ✅
- (c) A timer in your client that refetches data
- (d) A two-way channel between a phone and a server
> The direction is the whole point: their server calls yours. That is why the endpoint has no logged-in user and must be protected by signature verification instead.

### Q2 · L3
Your order-status screen may be up to ten seconds out of date. Peak is 400 concurrent students on a free tier. What should you choose?
- (a) WebSockets, because order status is a realtime feature
- (b) Polling every ten seconds, because each request is independent and cheap ✅
- (c) Polling every second, so it feels instant
- (d) SSE with no reconnection logic, because the stream is one-way
> The staleness number chooses the technology. Ten seconds of tolerance means six cheap requests a minute per user, which survives flaky 4G and does not hold 400 connections open.

### Q3 · L4
A WebSocket-based tracker is tested on Wi-Fi and works. A student on 4G reports it "stops updating". What is the most likely cause?
- (a) The server is rejecting their authentication token
- (b) The socket dropped on a tower switch and nothing reconnects or refetches ✅
- (c) The database is too slow to push updates
- (d) Their phone is too old to support WebSockets
> A dead pipe looks exactly like "nothing has happened". Visible connection state, reconnection with backoff, and a full refetch after reconnect are three separate pieces of work that first drafts skip.

### Q4 · L4
Reviewing a webhook handler, you find it parses the JSON, marks the order paid, sends a confirmation email, then verifies the provider's signature. What is the problem?
- (a) Nothing, as long as verification happens before the response
- (b) It should also log the payload
- (c) Verification must come first — until then the body is untrusted input ✅
- (d) The email should be sent before the database write
> Anything done before verification is work done on behalf of whoever sent the request. The endpoint is public and guessable, so unverified writes mean anyone can mark an order paid.

### Q5 · L5
A payment provider delivers the same `payment.captured` webhook twice because your first response was slow. Your handler credits the student's wallet each time. Which two fixes address this properly?
- (a) Reply faster and hope the retry never happens
- (b) Record the provider's event id and ignore it if already seen, and reply 200 immediately while queuing the real work ✅
- (c) Reject the second delivery with a 500 so the provider stops retrying
- (d) Ask the provider to disable retries for your account
> Duplicates are normal, not exceptional. Deduplicating on the provider's event id makes a repeat harmless, and queuing the slow work means the provider gets its prompt 200 and does not retry in the first place.

## Go deeper
- [MDN: Server-sent events](https://developer.mozilla.org/en-US/docs/Web/API/Server-sent_events/Using_server-sent_events)
- [Razorpay docs: Webhooks and payload validation](https://razorpay.com/docs/webhooks/validate-test/)
