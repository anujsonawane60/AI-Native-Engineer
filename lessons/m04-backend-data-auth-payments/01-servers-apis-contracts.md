---
id: m04-01-servers-apis-contracts
module: m04-backend-data-auth-payments
slug: servers-apis-contracts
title: "What a Server Does; APIs, Endpoints and Request/Response Contracts"
order: 1
duration_min: 25
is_free: false
mission: false
volatility: low
last_reviewed: 2026-11-16
---

## Key learnings
- Say what a server does that a browser can never be trusted to do
- Read an endpoint as a contract: method, path, request body, status, response body
- Agree the contract between two teammates before either of them builds anything
- Spot the three ways AI silently breaks a contract it wrote ten minutes ago
- Verify an endpoint by calling it yourself instead of trusting the summary

## Lesson

### The pain

Two members of a three-person team split the canteen app on a Tuesday. Priya takes the screen where a student taps "Place order". Rohit takes the part that saves the order.

Priya's AI builds a screen that sends `POST /api/orders` with `{ "items": [...], "total": 240 }` and expects back `{ "order_id": 91, "eta_minutes": 12 }`.

Rohit's AI builds a server that listens on `POST /api/order` — singular — reads a field called `cart`, ignores anything called `total`, and replies `{ "id": 91, "status": "PENDING" }`.

Both of them tested their own half. Both halves work. On Thursday they connect the two and get a 404, then a crash, then an order with no total. They spend forty minutes arguing about whose AI was wrong. Neither AI was wrong. Nobody had written down what the two halves promised each other.

### The idea

**A server is the one place in your app you actually control, and an API is the written promise of exactly what it will accept and exactly what it will send back.**

Everything on the phone — the app, the browser, the JavaScript — runs on a device a stranger owns. They can edit it, replay it, or lie to it. The server is yours. So every decision that matters (what a thing costs, who is allowed to see it, what gets saved) has to happen there.

The API is the seam between the two. If the seam is written down, two people can build either side at the same time. If it is not, you get Thursday.

### The analogy

The canteen counter is an API.

There is a menu board listing exactly what you can ask for, and the prices. You write your order on a slip in a fixed format: dish, quantity. You hand the slip through the window and get a token with a number on it. You cannot walk into the kitchen, you cannot touch the gas, and you cannot decide the price of a samosa by writing ₹2 on your own slip.

The menu board is the API. The slip is the request. The token is the response. The kitchen is the server's internals, and the fact that you are kept out of it is the entire point.

**Where the analogy breaks:** a canteen worker will improvise. Ask for "the usual, less spicy" and a human fills in the gaps. A server does not improvise — it either matches the contract exactly or it refuses with an error. That rigidity feels unfriendly and is the reason two machines can trust each other at all.

### Picture it

```
  CLIENT (phone)                            SERVER (yours)
┌────────────────────┐                 ┌──────────────────────┐
│  tap "Place order" │                 │  1. is the request   │
│                    │   REQUEST       │     well-formed?     │
│  POST /api/orders  │ ──────────────▶ │  2. who is asking?   │
│  Authorization: …  │  method + path  │  3. are they allowed?│
│  {items, note}     │  headers + body │  4. what is the real │
│                    │                 │     price?           │
│                    │   RESPONSE      │  5. save it          │
│  201 Created       │ ◀────────────── │                      │
│  {order_id, paise} │  status + body  │                      │
└────────────────────┘                 └──────────┬───────────┘
                                                  │ query
                                                  ▼
                                            ┌───────────┐
                                            │ database  │
                                            └───────────┘
```

A tap becomes a request with four parts — method, path, headers, body — and the server answers with two: a status code and a body. The five numbered checks inside the server are the work the phone is not trusted to do.

### Where it lives

In M00 you learned the four boxes: client, server, network, data. This chapter is the handshake between box one and box two, carried over box three.

Notice what the server does that the client cannot. The client sent `items` and a `note`. It did **not** get to send the price. Step four of that diagram — "what is the real price?" — is the server looking up today's prices itself. This is the single most important habit in the whole module, and it comes straight from M00's rule that the client cannot be trusted. A student who edits the request to say the total is ₹2 gets charged the real amount anyway, because the server never read their number.

You will meet this seam again everywhere. In M05 the cache sits in front of it. In M06 your tests call it. In M07 the OWASP checks are mostly about what the server failed to re-check. In M13 you will put a gateway in front of several of them.

### Reading one endpoint as a contract

Here is a contract written out in full. Not code — a promise.

```
POST /api/orders
  Auth:     required, student
  Body:     { items: [{ menu_item_id, qty }], note?: string }
  Returns:  201 { order_id, total_paise, eta_minutes }
  Errors:   400 bad body · 401 not logged in
            409 a listed item is sold out · 422 qty exceeds 10
```

Five lines, and both halves of the team can now work in parallel for two days without talking. Every field name is fixed. Every failure has a number. `total_paise` tells you the server computes money in paise, so nobody writes `240.00` as a float and loses a rupee to rounding — the ₹ and paise rule you will use for the rest of this module.

The method says what kind of action it is. `GET` reads and changes nothing. `POST` creates. `PUT` and `PATCH` change an existing thing. `DELETE` removes it. These are conventions, not laws — but breaking them means every tool, cache and proxy on the internet guesses wrong about your API.

The status code is the headline of the response and the part beginners ignore. `2xx` worked. `4xx` means the caller made a mistake — wrong body, not logged in, not allowed. `5xx` means the server broke. The difference between `401` and `403` is the whole of chapter 12: `401` is "I do not know who you are", `403` is "I know exactly who you are and you still cannot".

### Words to own

| Term | Plain meaning | Example |
|---|---|---|
| Server | A program always running, waiting for requests | The thing that saves an order |
| Endpoint | One address plus one method that does one job | `POST /api/orders` |
| Method | What kind of action the request is | `GET`, `POST`, `DELETE` |
| Path | The address part of the endpoint | `/api/orders/91` |
| Request body | The data you send, usually JSON | `{ items: [...] }` |
| Status code | The three-digit verdict on the response | `201`, `404`, `500` |
| Header | Extra information about the request | `Authorization: Bearer …` |
| Query parameter | Options bolted onto the path for reads | `?status=pending&limit=20` |
| Payload | Any body of data in transit | The JSON you POST |
| Contract | The agreed shape of request and response | The five lines above |

### Tradeoffs and gotchas

**The client cannot be trusted, and AI forgets this constantly.** Ask for a checkout endpoint and you will very often get one that accepts `total` from the phone and saves it. That is not a style issue; it is a free-food bug. Any number that affects money, stock or permissions must be recomputed server-side from your own data.

**A `200` with an error inside it is a lie.** AI output frequently returns `200 OK` with `{ "success": false, "message": "sold out" }`. Now every caller has to parse the body to find out whether it worked, retries behave wrongly, and your monitoring in M07 reports a perfectly healthy service while students cannot order. Failures belong in the status code.

**Verbs in paths are a smell.** `/api/getOrders`, `/api/createOrder` and `/api/deleteOrder2` are three endpoints doing what `GET`, `POST` and `DELETE` on `/api/orders` already express. It is not fatal, but it usually means nobody designed the API; it accreted one prompt at a time.

**Over-fetching costs real money on 4G.** An endpoint that returns every field of every order, including the full menu for each item, is fine on your laptop and painful on a budget Android phone in a hostel with two bars of signal. Decide what the screen actually needs and return that.

**Renaming a field is a breaking change.** The moment a second person, a mobile app, or a deployed site depends on `total_paise`, changing it to `amount` breaks them. That is what chapter 4 on versioning exists for.

### Verify it

When AI hands you an endpoint, four checks, in this order.

1. **Does the client's call match the server's route, character for character?** Singular versus plural, `/api/orders` versus `/orders`, `menu_item_id` versus `menuItemId`. Read both sides side by side. This is Thursday, prevented.
2. **Does the server recompute anything that matters?** Search the server code for the word `total` or `price`. If it reads a price out of the request body instead of out of the database, reject the work and say so plainly.
3. **Do the failures have the right status codes?** Try it logged out — you want `401`, not `200`. Send a garbage body — you want `400`, not `500`. Ask for an order that does not exist — `404`, not an empty `200`.
4. **Is money in paise, and is it an integer?** `24000` not `240.00`. Floats on currency lose paise and the losses do not cancel out.

Red flags that mean stop and re-brief: the response has no status code discussed at all; the AI says "the frontend will validate that"; the endpoint accepts a `user_id` from the request body to decide who the order belongs to (it must come from the session, never from the caller); the explanation includes the phrase "for simplicity" next to anything about authentication.

## Walkthrough
1. Open any real site you use on your phone's browser on a laptop — a college portal, a news site — and open the browser's developer tools, Network tab. Reload the page.
2. Pick one row in the list whose Type is `fetch` or `xhr`. That is an API call, not an image. Click it.
3. Read the method and the full path. Write them down on paper exactly as they appear, including any `?` query parameters, and say out loud what you think that endpoint's one job is.
4. Find the status code and the response headers. Note the `content-type`. If it says `application/json`, the body is data meant for a program, not for a human.
5. Look at the response body. Count the fields the page actually seems to display versus the fields that came back. That gap is over-fetching, and you are now looking at it in production software.
6. Now do the same on your own team's project if you have one running, or on a tiny endpoint you ask AI to stand up. Trigger one action in the UI and find its request.
7. Change something in the request before it is sent — use the "Edit and resend" feature your browser's dev tools offer — and send a value the UI would never produce: a quantity of `-3`, or a price field you invented.
8. Read what came back. A good server replies `400` or `422` and saves nothing. A bad one saves it. Record which you got; this is your first real experience of the client not being trustworthy.
9. Write the five-line contract for one endpoint in your own project, in the format shown in the lesson: method, path, auth, body, returns, errors. Put it in a file in your repo.
10. Hand that contract to a teammate and have them describe, without looking at any code, what a successful call and a failed call look like. If they cannot, the contract is not finished.

## Talk to AI
```prompt
CONTEXT:      I am designing the API seam for a student project before any
              code exists. Three of us are building it; two will work on
              opposite sides of this seam at the same time.
              Product: [one sentence on what your startup does]
              Users: [who they are, on budget Android phones, slow 4G]
              Data we already store: [list your tables or "none yet"]
              Money, if any, is stored in paise as integers.

GOAL:         A written request/response contract for the three most
              important endpoints in my product. No implementation.

REQUIREMENTS: 1. For each endpoint give me: method, path, whether auth is
                 required and for which role, the request body fields with
                 types, the success status code and response body, and every
                 error status code with the condition that triggers it
              2. For every field in a request body, say explicitly whether
                 the server trusts it or recomputes it, and why
              3. Flag any field I have asked for that the server must NOT
                 accept from the client
              4. Use snake_case field names consistently and store all money
                 as integer paise with names ending in _paise
              5. Keep responses small enough to be reasonable on slow 4G,
                 and tell me which fields you left out on purpose

CONSTRAINTS:  Free tier only. JSON over HTTP. No GraphQL and no gRPC for
              now. Do not invent tables I did not list.

NON-GOALS:    No code. No database migrations. No authentication
              implementation. Do not design an admin API yet.

ACCEPTANCE:   - A teammate can read the contract and build the client side
                without asking me a single question
              - Every endpoint has at least two documented error cases
              - No price, total or user id is accepted from the client
              - Reading a thing uses GET and never changes data

ASK:          First propose two different ways to split these endpoints —
              one fine-grained, one coarse-grained — with the tradeoffs of
              each on a slow mobile network. Say which you would choose and
              why. Write no code until I pick one.
```

## Stuck? Ask AI
```prompt
Explain the difference between a 400, a 401, a 403, a 404, a 409 and a 500 status code using a canteen counter as the example, one sentence of story each. Then give me six situations from a food-ordering app and ask me which code each should return, without telling me the answers first.
```

```prompt
Here is an API endpoint someone built for me: [paste the route or its description]. Do not improve it. Instead, tell me which values it accepts from the client that it should be recomputing on the server, and for each one describe exactly how a student could abuse it from their phone.
```

```prompt
I do not understand what a server does that my browser cannot. Explain it to me as if I have never written a program, using exactly one example that involves money. Then explain why that example would be unsafe if the same work happened in the browser.
```

## Quiz

### Q1 · L2
What are the four parts of an HTTP request?
- (a) Status code, headers, body, cookies
- (b) Method, path, headers, body ✅
- (c) Endpoint, database, cache, response
- (d) Username, password, path, payload
> Method and path say what action on what resource; headers carry extra information such as auth; the body carries the data. The status code belongs to the response, not the request.

### Q2 · L3
An AI builds a checkout endpoint that accepts `{ items, total_paise }` from the phone and saves `total_paise` as the amount owed. What is wrong?
- (a) Nothing — sending the total saves the server a database query
- (b) The field name should be camelCase
- (c) A student can edit the request and pay ₹1 for a ₹240 order ✅
- (d) The total should be a float so paise are not lost
> Anything that affects money, stock or permissions must be recomputed on the server from your own data. The client runs on a device a stranger controls.

### Q3 · L3
Your server replies `200 OK` with `{ "success": false, "error": "sold out" }`. Why is this a problem beyond style?
- (a) JSON cannot contain a boolean field
- (b) Callers, retries and monitoring all read the status code and will believe it worked ✅
- (c) It uses more bandwidth than a 409
- (d) `200` is reserved for GET requests only
> The status code is the machine-readable verdict. A permanent `200` means your dashboards in M07 show a healthy service while students cannot order anything.

### Q4 · L4
Priya's client calls `POST /api/orders` and Rohit's server listens on `POST /api/order`. Both were tested and both work alone. What does this tell you about the method, not the people?
- (a) They should have used the same AI tool
- (b) The contract was never written down before either side was built ✅
- (c) Plural paths are always correct and singular ones are wrong
- (d) The client should have retried until it found the right path
> Each AI invented a reasonable contract because nobody supplied one. Writing the seam down first is what lets two people build opposite sides in parallel.

### Q5 · L4
Which of these should the server refuse to take from the request body?
- (a) The free-text note "less spicy"
- (b) The quantity of each item ordered
- (c) The `user_id` that says whose order this is ✅
- (d) The id of the menu item being ordered
> Identity comes from the session or token, never from a field the caller fills in. Accepting `user_id` from the body lets anyone place or read orders as anyone else.

## Go deeper
- [MDN: An overview of HTTP](https://developer.mozilla.org/en-US/docs/Web/HTTP/Overview)
- [MDN: HTTP response status codes](https://developer.mozilla.org/en-US/docs/Web/HTTP/Status)
