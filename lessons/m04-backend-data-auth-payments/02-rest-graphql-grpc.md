---
id: m04-02-rest-graphql-grpc
module: m04-backend-data-auth-payments
slug: rest-graphql-grpc
title: "REST, GraphQL and gRPC: Three Ways to Shape an API"
order: 2
duration_min: 25
is_free: false
mission: false
volatility: low
last_reviewed: 2026-11-16
---

## Key learnings
- Choose between REST, GraphQL and gRPC for a specific screen and say why
- Recognise over-fetching and the N+1 request problem on a slow mobile network
- Explain what GraphQL buys you and what it costs you in return
- Say why gRPC is for service-to-service traffic and not for your browser
- Check that AI has not quietly handed you three API styles in one project

## Lesson

### The pain

The canteen app's home screen needs six things: today's menu, the student's name, their wallet balance, their last order, whether the canteen is open, and three "order again" suggestions.

An AI builds it the obvious way: six endpoints, six requests. On a laptop it loads instantly. On a budget Android phone on hostel 4G, each round trip costs about 300 milliseconds before any data moves, and the requests fire one after another because the suggestions need the last order first. Nearly two seconds of waiting, mostly spent on the network doing nothing.

A second team hears "GraphQL fixes that", installs it, and now has one request — plus a query language, a schema, a caching problem they do not understand, and an endpoint where a single malicious query can ask for every order of every user and sit there for ninety seconds. They traded a slow screen for an outage.

Neither problem is about the tool. Both are about choosing a shape without knowing what each shape is for.

### The idea

**REST, GraphQL and gRPC are three answers to one question: who decides the shape of the response — the server, the client, or a compiled contract both sides share?**

In REST the server decides. Each endpoint returns a fixed shape, and the client takes what it is given.

In GraphQL the client decides. There is one endpoint, and the request itself describes exactly which fields it wants.

In gRPC a shared contract file decides. Both sides compile code from the same definition, speak a compact binary format, and cannot drift apart without the build failing.

### The analogy

Three canteens.

The first has a menu board with fixed thalis. Thali number 3 is rice, dal, two rotis and a sabzi. You cannot swap the sabzi. Ordering is instant, the kitchen is simple, and if you want only the dal you still carry the rice away. That is REST.

The second has a counter where you fill in a slip: "rice yes, dal yes, roti no, sabzi yes, pickle yes". You get precisely what you asked for and nothing else. The slip takes longer to write, the kitchen has to be smarter, and someone eventually hands in a slip asking for one of everything on the premises. That is GraphQL.

The third is the hatch between the kitchen and the storeroom. The two staff behind it use clipped codes nobody outside understands, because they work together fifty times an hour and speed matters more than friendliness. That is gRPC.

**Where the analogy breaks:** a canteen's slip system cannot bankrupt the canteen. A single GraphQL query can genuinely take down a database, because the client is now writing part of your data access plan. Giving the caller that much power is a security decision, not just an ergonomics one.

### Picture it

```
 REST              GraphQL                 gRPC
 ┌──────┐          ┌──────┐                ┌──────┐
 │client│          │client│                │svc A │
 └──┬───┘          └──┬───┘                └──┬───┘
    │ GET /menu       │ one POST /graphql      │ binary call
    │ GET /me         │ { menu{name price}     │ GetMenu(req)
    │ GET /wallet     │   me{name}             │ over HTTP/2
    │ GET /orders     │   wallet{paise} }      │
    ▼                 ▼                        ▼
 ┌──────────────┐  ┌──────────────┐        ┌──────────────┐
 │ fixed shapes │  │ one endpoint │        │ compiled     │
 │ server picks │  │ client picks │        │ contract     │
 └──────────────┘  └──────────────┘        └──────────────┘
   4 round trips     1 round trip            fastest, but
   simple to cache   hard to cache           not for browsers
```

The same home screen fetched three ways: four separate fixed responses, one tailored response, or one compact binary call between two servers.

### Where it lives

All three sit in the same slot — the seam between client and server from chapter 1. What changes is who holds the pen.

For your project in this program, the honest default is REST. It is what every free-tier platform gives you by default, what caching in M05 is designed around, what your browser's Network tab shows you plainly, and what AI produces most reliably because there is thirty years of it in the training data.

GraphQL earns its place when one screen genuinely needs data from five places and you control both sides. Swiggy-style apps with dozens of screens and a mobile release cycle use it precisely because the client can change what it asks for without a server deploy.

gRPC lives behind your own walls. Browsers cannot speak it directly without a proxy. Its home is one of your services calling another — which is chapter 5's territory, and M13 takes it further.

### The N+1 request problem, which is the real enemy

Over-fetching is asking for too much. Under-fetching is the opposite and it hurts more: you ask for the list, then make one more request per item in the list.

Fetch 20 orders, then fetch the student's name for each one. That is 1 + 20 = 21 requests. At 300 ms of latency each on 4G, that is six seconds of a spinner. The same thing happens one layer down inside the database, where it becomes 21 queries instead of one join — chapter 9's subject.

REST solves it by designing a purpose-built endpoint: `GET /api/orders?include=student`. GraphQL solves it by letting the client ask for nested fields in one query. gRPC solves it by making each call so cheap that twenty-one of them inside a data centre cost almost nothing. Three different answers to the same arithmetic.

### Words to own

| Term | Plain meaning | Example |
|---|---|---|
| REST | Fixed endpoints, server decides the shape | `GET /api/orders/91` |
| Resource | A noun your API exposes | an order, a menu item |
| GraphQL | One endpoint, client names the fields it wants | `{ menu { name price_paise } }` |
| Schema | The typed list of what can be asked for | GraphQL's contract |
| Resolver | Server code that fills in one field | fetches `order.student` |
| gRPC | Compact binary calls from a shared contract | service to service |
| Protobuf | The file format that contract is written in | `order.proto` |
| Over-fetching | Getting fields the screen never uses | 40 fields, 6 shown |
| Under-fetching | Needing another request per item | 1 + 20 calls |
| Round trip | One request out and its response back | ~300 ms on 4G |

### Tradeoffs and gotchas

**Caching is REST's superpower and GraphQL's weakness.** `GET /api/menu` is one URL. A CDN can hold it, your browser can hold it, and M05's cache layers all work on it for free. Every GraphQL request is a `POST` with a different body, so none of that machinery applies and you have to build caching yourself, per field, inside the server.

**An unprotected GraphQL endpoint is a loaded gun.** Depth limits, query cost limits and disabled introspection in production are not optional extras. A nested query asking for every order of every student, each with their orders again, is a valid query and a denial-of-service. If AI sets up GraphQL for you and never mentions depth or cost limits, it has given you a vulnerability, and M07 will make you fix it.

**gRPC does not work from a browser without extra machinery.** If AI suggests gRPC for your web frontend, it is wrong unless it also specified a proxy layer. Do not accept it.

**Three styles in one small project is a self-inflicted wound.** A REST endpoint here, a GraphQL endpoint there because one prompt mentioned it, and an RPC-shaped `POST /api/doThing` somewhere else. Each one has its own auth handling, its own error format and its own way of being wrong. On a team of three with four months, pick one and write it in your scope document.

**"GraphQL is modern, REST is old" is not an engineering argument.** The real question is whether your clients need to vary what they fetch more often than you can deploy the server. For a three-person team shipping daily, they do not.

### Verify it

When AI proposes an API style, check five things.

1. **Is one style used consistently across the project?** Grep your routes. If you find `/api/orders`, `/api/graphql` and `/api/getOrderDetails`, nobody is in charge of the design.
2. **Count the round trips your worst screen needs.** Open the Network tab, load the heaviest screen, and count `fetch` rows. More than six for one screen on a mobile-first product is a design problem, whatever the style.
3. **If it chose GraphQL, ask where the limits are.** Query depth limit, cost limit, introspection off in production, and a plan for the N+1 inside resolvers. No answer to all four means it has not actually done the job.
4. **If it chose gRPC for a browser client, reject it.** Ask it directly: "can a browser call this without a proxy?" The honest answer is no.
5. **Check that reads are cacheable.** A read that is a `POST` cannot be cached by anything, and in M05 you will wish it could.

Red flags: the recommendation arrives without being asked which screens exist; the words "flexible" and "modern" appear more often than any number; the AI switches style between two answers in the same conversation; a proposal to "expose the database directly through GraphQL so the frontend can query anything", which is chapter 14's nightmare wearing a tie.

## Walkthrough
1. Pick the heaviest screen in your own product — the one that needs data from the most places. List every distinct piece of information it displays. Aim for a list of at least six.
2. On paper, design it as REST: write the endpoints you would need and the fields each returns. Count the round trips a cold load would take.
3. Now mark every field in those responses that this screen does not display. That is your over-fetching, measured.
4. Find any place where you would need one request per item in a list. Write down the arithmetic: 1 + N, with your real N. That is under-fetching, measured.
5. Design the same screen as a single GraphQL query — just the field selection, as a sketch. Note how many round trips it becomes.
6. Now write down the three things you would have to build to make that GraphQL version safe: a depth limit, a cost limit, and something to stop resolvers making N queries.
7. Open the Network tab on a real Indian app you use, throttle it to Slow 4G, and reload. Count the requests and read the slowest one's timing breakdown. Note how much of it is waiting rather than transferring.
8. Decide, in one sentence with a reason, which style your project will use. Write it in your scope document as a decision, not a preference.
9. Brief AI with that decision as a constraint and ask it to design one purpose-built REST endpoint that collapses your worst 1 + N case into a single call.
10. Review what it returns against step 3: did it fix the round trips while quietly reintroducing over-fetching? Most first attempts do.
11. Record the decision in your log: what you chose, what you rejected, and the number that decided it.

## Talk to AI
```prompt
CONTEXT:      Three-person student team, four months, free tiers only. Our
              users are on budget Android phones on slow 4G in India, so
              round trips are expensive — assume ~300 ms each.
              Product: [one sentence]
              Our heaviest screen displays: [list every piece of data it
              shows, at least six items]
              Our clients: [web only / web + mobile app / also admin panel]
              We deploy: [how often — e.g. several times a week]

GOAL:         Decide whether this product should use REST, GraphQL or a
              mixture, based on my actual screens rather than on fashion.

REQUIREMENTS: 1. For my heaviest screen, design it as REST: list the
                 endpoints, their response fields, and count the round trips
              2. Design the same screen as a single GraphQL query sketch
              3. For each design, tell me the measured cost: number of round
                 trips, and which returned fields the screen never uses
              4. Identify every place I would hit a 1 + N request pattern
              5. If you recommend GraphQL, list exactly what I must add to
                 make it safe in production and estimate the work in hours
              6. Tell me what each option costs me in M05-style caching

CONSTRAINTS:  Free tier only. Team has never used GraphQL. We cannot afford
              to spend more than one day on API plumbing. Browsers must be
              able to call this directly, so no gRPC for client traffic.

NON-GOALS:    Do not write any implementation code. Do not design the
              database. Do not propose more than one API style for us to run
              at the same time.

ACCEPTANCE:   - Your recommendation cites a number from my screen list, not
                an adjective
              - You state the round-trip count for both designs
              - You name at least one thing your recommended option makes
                harder, not just what it makes easier
              - A teammate who has never heard of GraphQL can read your
                answer and understand the choice

ASK:          Give me both designs with tradeoffs first, then your
              recommendation and the single number that decided it. No code
              until I pick one.
```

## Stuck? Ask AI
```prompt
Explain the difference between over-fetching and under-fetching using a canteen as the example. Then give me five short descriptions of app screens and ask me which problem each one has, without telling me the answers first.
```

```prompt
I keep reading that GraphQL is better than REST. Argue the opposite case as strongly as you can, for a three-person team on free tiers shipping a mobile-first product in four months. Then tell me the one situation where that argument would be wrong.
```

```prompt
Explain what a round trip is and why 300 milliseconds of latency on 4G matters more than the size of the data. Use arithmetic with real numbers for a screen that needs six separate requests, and show me what happens if three of them must happen in sequence.
```

## Quiz

### Q1 · L2
In GraphQL, who decides which fields come back in a response?
- (a) The server, through a fixed endpoint shape
- (b) The client, by naming the fields in its query ✅
- (c) A compiled contract file shared by both sides
- (d) The CDN, based on what it has cached
> That is the defining difference. The server fixes the shape in REST; a compiled contract fixes it in gRPC; in GraphQL the request itself describes the shape.

### Q2 · L3
A list screen makes one request for 20 orders, then one request per order for the student's name. How many round trips, and what is this called?
- (a) 20 requests — over-fetching
- (b) 2 requests — batching
- (c) 21 requests — the N+1 problem ✅
- (d) 1 request — under-fetching
> One for the list plus one per item. At 300 ms each on 4G that is about six seconds of spinner, and the same pattern reappears as 21 database queries in chapter 9.

### Q3 · L4
AI sets up a GraphQL endpoint for your project and never mentions query depth or cost limits. What have you actually been given?
- (a) A performance improvement with no downside
- (b) A denial-of-service vulnerability a single valid query can trigger ✅
- (c) An endpoint that cannot be called from a browser
- (d) A caching layer you did not ask for
> A deeply nested query asking for every student's orders, each with their student, is legal GraphQL. Without depth and cost limits the caller can plan arbitrarily expensive work on your database.

### Q4 · L4
Your mobile-first product needs a read that a CDN and the browser can both cache. Which shape fits worst?
- (a) `GET /api/menu` in REST
- (b) A GraphQL query sent as a `POST` to `/graphql` ✅
- (c) `GET /api/menu?day=today` in REST
- (d) A REST endpoint returning a fixed JSON shape
> Caching keys on the URL and the method. Every GraphQL request is a POST with a different body, so browser, CDN and proxy caching from M05 simply do not apply.

### Q5 · L3
AI recommends gRPC for the browser frontend of your canteen app. What is the correct response?
- (a) Accept it — gRPC is the fastest option available
- (b) Accept it but add a REST endpoint alongside for compatibility
- (c) Reject it: browsers cannot call gRPC directly without a proxy layer ✅
- (d) Accept it only if the team already knows Protobuf
> gRPC's home is service-to-service traffic behind your own walls. Browser support requires an extra translation layer, which is cost you have no reason to pay for a student project.

## Go deeper
- [MDN: Choosing between REST and GraphQL style APIs](https://developer.mozilla.org/en-US/docs/Learn_web_development/Extensions/Server-side)
- [GraphQL: Best practices and security considerations](https://graphql.org/learn/best-practices/)
