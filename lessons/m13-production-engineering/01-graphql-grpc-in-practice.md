---
id: m13-01-graphql-grpc-in-practice
module: m13-production-engineering
slug: graphql-grpc-in-practice
title: "GraphQL and gRPC in Practice: Schemas, Resolvers, Contracts"
order: 1
duration_min: 35
is_free: false
mission: true
volatility: low
last_reviewed: 2026-11-16
---

## Key learnings
- Read a GraphQL schema and trace which resolver answers each field
- Spot the N+1 inside a resolver tree and say where batching belongs
- Configure the three limits that stop one query taking your database down
- Evolve a schema and a protobuf contract without breaking a shipped client
- Catch a generated API that is correct for one caller and unsafe for everyone

## Lesson

### The pain

M04 chapter 2 taught you the three API styles and the one question that separates them: who decides the response shape — the server with REST, the client with GraphQL, or a compiled contract with gRPC. It taught over-fetching and under-fetching, it taught the N+1 problem as 1 + 20 = 21 requests and about six seconds on a Nashik 4G connection, and it told you unprotected GraphQL is a denial-of-service vulnerability. All of that was the overview. This chapter is the practice: writing the schema, reading the resolvers, installing the limits, and evolving a contract without breaking a mobile app you already shipped to the Play Store.

So here is what week one of real GraphQL looks like. A team replaces four REST endpoints with one GraphQL endpoint. The mobile app gets faster, everyone is pleased. Then a teammate writes a query on the dashboard that asks for every order, and for each order the student, and for each student their last ten orders, and for each of those the dishes. One HTTP request. Four thousand database queries. The database's connection pool — the one from M05 chapter 6 — is exhausted in nine seconds, and the entire product is down, including the parts that have nothing to do with the dashboard.

Second failure, three weeks later. They rename a field from `price` to `priceInPaise` because the old name was ambiguous. The web app is updated the same day. The Android app on two hundred phones is not, because an app store update takes days and users do not install it. Every one of those phones now shows a blank price. In REST you would have versioned the endpoint. In GraphQL there are no versions, so the only tool you have is deprecation — and you did not use it.

### The idea

**In GraphQL you publish a type system and give up control of the query, so your safety comes entirely from the resolvers and the limits. In gRPC you publish a numbered contract, so your safety comes entirely from never reusing a number.**

A schema declares the types and fields that exist. A **resolver** is the function that produces one field's value. The client sends a tree of fields; the server walks that tree calling resolvers. Every performance problem and every security problem in GraphQL is a property of that walk.

gRPC is the opposite trade. The shape is fixed in a `.proto` file both sides compile, so the client cannot ask for anything new. Its fragility is in the field *numbers*, which are the actual wire identity of each field — the name is only for humans.

### The analogy

M04 gave you three canteens. Go behind the counter of the second one.

The tick-sheet a student fills in is the **schema**: it defines what can be asked for at all, and nothing off the sheet is orderable. Each ticked item is fetched by a particular person — one goes to the warmer, one to the fridge, one to the sweet counter. Those are the **resolvers**. Now watch the mistake: twenty students tick "dish of the day", and the runner walks to the kitchen twenty separate times to ask which dish it is today. One trip with a list of twenty would have taken the same time as one trip. That is **batching**, and the twenty trips are your N+1.

The third canteen's hatch has pre-printed carbon forms with numbered boxes. Box 7 always means "paneer trays". The storeroom never reads the labels, only the box positions, because that is faster. That is a **protobuf contract**. Reprint the form with the boxes in a new order and the storeroom will keep filling box 7 with paneer while the kitchen now reads box 7 as rice — silently, with no error anywhere.

**Where the analogy breaks:** a student cannot tick an item on the sheet a thousand times, nested inside itself. A GraphQL client can send a query nested fifteen levels deep that asks for a million rows, and the server will attempt it with total enthusiasm. The counter staff have a physical limit on how absurd an order can get; your resolvers have none unless you install one.

### Picture it

```
QUERY                      RESOLVER WALK
{                      ┌─────────────────────────────────┐
  orders(last: 20) {   │ orders        → 1 query         │
    id                 │                                 │
    student {          │ WITHOUT batching:               │
      name             │   student  → 20 queries         │
    }                  │   dishes   → 20 queries         │
    dishes {           │   TOTAL 41 ◀── N+1, twice       │
      name             │                                 │
    }                  │ WITH batching (one per level):  │
  }                    │   student  → 1 query (20 ids)   │
}                      │   dishes   → 1 query (20 ids)   │
                       │   TOTAL 3                       │
                       └─────────────────────────────────┘
```

One client query becomes a tree of resolver calls, and the difference between 41 database queries and 3 is whether each level collects its ids and asks once.

### Where it lives

This chapter sits on top of M04's API layer and reaches into three other places. The batching fix is a request-scoped cache, which is M05 chapter 2's in-process cache with a lifetime of one request — and that short lifetime is exactly why it has no invalidation problem. The query limits are M05 chapter 4's rate limiting applied to cost rather than to count. The connection pool a runaway query exhausts is M05 chapter 6. And gRPC's home, as M04 said, is one of your services calling another, which is where chapter 2's event-driven work and chapter 3's cluster put it.

Caching is the honest cost of GraphQL and it points at chapter 8. A REST `GET` has a URL, so a CDN can cache it. A GraphQL query is a `POST` with a body, so the edge cannot cache it at all unless you adopt persisted queries, which turn a known query into a short id that *can* appear in a URL.

### Words to own

| Term | Plain meaning | Example |
|---|---|---|
| Schema | The types and fields that exist | `type Dish { name: String! }` |
| Resolver | The function producing one field's value | `Order.student` |
| Resolver tree | The walk of resolvers one query triggers | 41 calls, or 3 |
| Batching | Collect ids at a level, fetch once | 20 lookups → 1 |
| Request-scoped cache | A cache living for one request only | No invalidation needed |
| Depth limit | Maximum nesting a query may have | Reject beyond 8 |
| Complexity / cost limit | A score per query, with a budget | Reject beyond 1000 |
| Introspection | The schema describing itself on request | Turn off in production |
| Persisted query | A pre-registered query sent as an id | Makes edge caching possible |
| Deprecation | Marking a field old without removing it | `@deprecated(reason:)` |
| Protobuf | gRPC's contract format | A `.proto` file |
| Field number | A field's real wire identity in protobuf | `price = 7;` |
| Reserved | Marking a retired number unusable | `reserved 7;` |
| Codegen | Generating client and server types from a contract | From schema or `.proto` |
| Contract test | A test asserting two services still agree | Runs in CI |

### Tradeoffs and gotchas

**Batching is not optional and it is the thing AI omits most.** A resolver that queries the database per parent row is the default shape of generated code, because it is the obvious shape. It works perfectly with three test rows and falls over at twenty. Ask for the query count, not the correctness.

**Install all three limits before you expose the endpoint, not after.** Depth, complexity and introspection. Depth stops infinite nesting. Complexity gives every field a cost and every query a budget, which is the only limit that catches a shallow but enormous query. Turning introspection off in production is not real security, but it does stop a stranger reading your entire data model in one request.

**GraphQL returns 200 when things fail.** An error arrives as an `errors` array inside a successful HTTP response, often alongside partial data. This breaks every monitoring assumption from M07: your error rate looks like zero while users see blanks. You must count GraphQL errors explicitly or you are blind.

**There are no versions, so deprecate rather than remove.** Add the new field, mark the old one deprecated with a reason and a date, watch field-level usage until it reaches zero, and only then delete. The shipped mobile app from M12 is why this matters: you do not control when users update.

**In protobuf, never reuse a field number, and never change a field's type.** Rename freely — names are for humans. But a retired number must be marked reserved forever, because an old client still on the wire will put its data in that box. This is the single rule that makes gRPC safe and the single rule that silently corrupts data when broken.

**Adding a third API style to a small product is a self-inflicted wound,** exactly as M04 said. The honest default for your startup is still REST, with GraphQL where one screen genuinely needs six resources at once, and gRPC only between your own services. Three styles means three sets of auth, three sets of monitoring and three sets of limits.

**Authorization lives in the resolvers, per field.** This is the trap M07 chapter 1's broken-access-control entry predicts. With REST you check permissions per endpoint. With GraphQL a student can reach `order { student { phone } }` through a path nobody thought about, and if the check is only on the top-level query, they get another student's phone number.

### Verify it

1. **Ask for the query count, with a number.** "How many database queries does this query produce for 20 orders?" If the answer is not stated and then demonstrated by a log of actual queries, you do not know. Turn on query logging and count them yourself.
2. **Try the abusive query yourself.** Write one nested fifteen levels deep and send it. If it returns data instead of a rejection, your limits are missing. Then write a shallow one asking for ten thousand rows; depth alone will not stop it, which is what complexity limits are for.
3. **Ask as the wrong user.** Pick a field holding someone else's personal data and reach it through the deepest path you can construct. This is the check that finds the resolver where authorization was forgotten.
4. **Confirm errors are visible.** Force a resolver to fail and look at your monitoring from M07. If your error rate stayed at zero, instrument GraphQL errors before you ship anything.
5. **For a schema change, demand the deprecation path.** Any answer that removes or renames a field without a deprecation period and a usage check is wrong, no matter how much tidier the result is.
6. **For a `.proto` change, read the field numbers.** Confirm nothing was renumbered, nothing retired was reused, and retired numbers are reserved.

Red flags: a resolver that queries inside a loop; a GraphQL endpoint with no depth or complexity limit; introspection enabled in production; a field removed rather than deprecated; a protobuf field renumbered "to tidy it up"; authorization checked only at the query root; a claim that GraphQL "reduces load on the server", which is backwards — it reduces round trips for the client and increases how much one request can cost you.

## Walkthrough
1. Pick one real screen in your product that currently needs three or more API calls. Write down the calls, what each returns, and the measured total time on a throttled connection.
2. Write the schema for exactly that screen's data by hand first — types, fields, which are non-null. Doing this before AI sees it is what stops you accepting a shape you did not choose.
3. Brief AI to implement the schema and resolvers against your existing database, and ask it to state, as a number, how many database queries one query for twenty rows will produce.
4. Turn on database query logging, send the query for twenty rows, and count the queries yourself. Compare with the number it claimed.
5. If the count is high, have it add request-scoped batching at every nested level. Re-run and count again. Record both numbers; the difference is this chapter's whole lesson.
6. Send a deliberately abusive query: nest it fifteen levels deep. Note whether it is rejected or served, and how long the server takes.
7. Add a depth limit and a complexity limit with explicit numbers, and turn introspection off for production. Re-send the abusive query and confirm a rejection that names the reason. Then send a *shallow* but enormous query asking for thousands of rows, and confirm the complexity limit catches what the depth limit cannot.
8. Break authorization on purpose: log in as one student and try to reach another student's phone number through the deepest nested path the schema allows. Fix the resolver, then re-test the same path.
9. Force one resolver to throw, and check your monitoring. If the error rate stayed at zero, add GraphQL error counting before you continue.
10. Evolve the schema properly: add a replacement field, mark the old one deprecated with a reason, and leave both working. Write down how you would know when the old field's usage reaches zero.
11. Finally, read a `.proto` file for one internal service call, identify every field number, and write the one-line rule about reuse in your decision log.

## Talk to AI
```prompt
CONTEXT:      I am a student on an AI-native engineering program. I direct AI
              and verify its output; I do not write code by hand. My team of
              three runs [product in two sentences] with [number] real users
              on budget Android phones and slow 4G in India. Stack: [your
              stack], [your database] on a free tier with a small connection
              limit. We currently expose REST endpoints. One screen -
              [describe it] - needs [number] separate calls today and takes
              [measured] ms on a throttled connection. We have a mobile app
              already in the Play Store, so users may be running an old
              version for weeks after we change anything.

GOAL:         Replace that ONE screen's data fetching with a GraphQL query
              that is correct, cannot be abused, and can be changed later
              without breaking the shipped app.

REQUIREMENTS: 1. A schema for exactly that screen - no extra types - and for
                 every field tell me which resolver answers it
              2. State as a NUMBER how many database queries one query for 20
                 rows will produce, before and after batching, and tell me how
                 to count them myself from query logs
              3. Add request-scoped batching at every nested level, and
                 explain why a request-scoped cache has no invalidation
                 problem
              4. Add a depth limit, a complexity limit with a per-field cost,
                 and disable introspection in production. Give me the actual
                 numbers you chose and one sentence justifying each
              5. Write me the two abusive queries I should send to test those
                 limits: one deeply nested, one shallow but enormous. Explain
                 which limit catches which, and why depth alone is not enough
              6. Tell me exactly where authorization must be checked, per
                 field, and give me the nested path a student could use to
                 reach another student's personal data if I get it wrong
              7. Tell me how GraphQL reports errors, why that makes my
                 existing error-rate monitoring blind, and what to count
                 instead
              8. Give me the deprecation procedure for changing a field later,
                 given that I cannot force a mobile app update

CONSTRAINTS:  Free tier only. Our database connection limit is [number], so
              tell me what a runaway query does to it. Plain English for a
              second-language reader. Do not change our database schema. Keep
              every existing REST endpoint working - this is additive.

NON-GOALS:    Do not convert our whole API to GraphQL. Do not add gRPC. Do not
              add a GraphQL gateway, federation, or subscriptions. Do not add
              a second database or a caching product.

ACCEPTANCE:   - I counted the database queries myself, before and after
                batching, and have both numbers
              - A query nested 15 levels deep is rejected with a named reason
              - A shallow query for 10,000 rows is also rejected
              - Introspection is off in production
              - Logged in as one student, I cannot reach another student's
                phone number through any nested path I can construct
              - A failing resolver shows up in my error monitoring
              - I can state the deprecation steps for changing a field

ASK:          Before any code, tell me honestly whether this screen justifies
              GraphQL at all, and make the strongest case for just adding one
              combined REST endpoint instead. Then give me the schema and the
              query-count numbers, and wait for me to confirm before you write
              resolvers.
```

## Mission

**Objective.** Put one screen of your own startup behind a GraphQL query with measured batching, working limits and per-field authorization, and prove each of the three with your own observation rather than the AI's summary.

**Steps.**
1. **Frame** — name the single screen, the REST calls it makes today, and its measured load time on a throttled connection. One screen only.
2. **Design the schema yourself** — write the types and fields by hand before AI sees the task. Mark which fields contain personal data; those are the ones that will need per-field checks.
3. **Brief** — use the Director's Brief above. Demand the honest argument against GraphQL first, and record whether you agreed with it.
4. **Direct** — have it build the schema and resolvers. Read the resolver code and mark every place it touches the database.
5. **Measure the N+1** — turn on query logging, request twenty rows, and count the database queries by hand. Record the number.
6. **Add batching and measure again** — record the second number. Both numbers go in your upload.
7. **Attack your own endpoint** — send a fifteen-level nested query and a shallow ten-thousand-row query. Record what happened before limits, then add depth and complexity limits and record what happens after.
8. **Attack your own authorization** — as one logged-in user, try to reach another user's personal data through the deepest nested path your schema allows. If you succeed, fix it and re-test.
9. **Check your blindness** — force a resolver to fail and look at your M07 monitoring. If the error rate stayed flat, add GraphQL error counting.
10. **Deprecate something** — add a replacement for one field, mark the original deprecated with a reason, and confirm both still work.
11. **Compare honestly** — measure the screen's load time again on the same throttled connection. State whether the improvement justified the new attack surface.
12. **Document** — one decision-log entry: why this screen, the two query counts, your three limit numbers, and what you would need to see before using GraphQL anywhere else.

**Upload.**
- Your Director's Brief, in full → post caption or an attached file
- The schema you wrote by hand, before the AI's version → text or photo
- Database query counts for twenty rows, before and after batching → two screenshots of query logs
- Your deeply nested query and your enormous shallow query, with the server's response before and after limits → four screenshots
- Your three limit values with one sentence of justification each → text
- Your authorization attempt as the wrong user, with the result → screenshot
- Monitoring before and after you added GraphQL error counting, with a resolver failing → screenshots
- The deprecated field and its replacement both working → screenshot
- Measured screen load time on the same throttled connection, before and after → text
- The AI conversation including its argument against GraphQL → share link in `ai_chat_url`
- Working endpoint → deployed URL in `demo_url`
- Your decision-log entry
- Reflection, 150 words max → one tradeoff you chose, and one thing the AI got wrong

**Self-check.** Tick every box before you submit.
- [ ] I wrote the schema myself before the AI saw the task
- [ ] I counted database queries from logs, not from the AI's claim
- [ ] My query count for twenty rows did not grow with the number of rows
- [ ] A fifteen-level nested query is rejected with a named reason
- [ ] A shallow query for thousands of rows is also rejected
- [ ] Introspection is disabled in production
- [ ] I tried to reach another user's personal data and could not
- [ ] A failing resolver appears in my error monitoring
- [ ] I deprecated a field rather than removing it
- [ ] I can state whether the measured improvement justified the change
- [ ] My reflection names something specific the AI got wrong

**Stretch.** Register your screen's query as a persisted query sent by id, and write 80 words on what that makes possible at the CDN layer that a `POST` body never could — and what it costs you in flexibility.

## Stuck? Ask AI
```prompt
Here is a GraphQL resolver file: [paste it]. Do not rewrite it. Instead, trace the resolver walk for a query that asks for 20 orders, each with its student and its dishes, and tell me exactly how many database queries that produces and why. Then tell me which resolvers need request-scoped batching, and what the query count becomes afterwards. Give me the read-only way to confirm both numbers myself from query logs.
```

```prompt
Explain depth limits and complexity limits as two different defences. Give me one query that a depth limit catches and a complexity limit does not, and one query that a complexity limit catches and a depth limit does not. Then ask me to guess sensible numbers for both for a small app with a 30-connection database, and tell me afterwards whether my numbers were too tight or too loose and how I would find out in production.
```

```prompt
Explain protobuf field numbers to someone who has only used JSON APIs. Then walk me through exactly what happens, byte by byte in concept, when a server renumbers a field from 7 to 9 while an old client is still sending 7. Tell me what the server sees, whether any error is raised, and what the user experiences. Finish with the two rules I must never break and the one keyword that enforces them.
```

## Quiz

### Q1 · L3
A GraphQL query asks for 20 orders and each order's student. Your resolver looks up a student per order. How many database queries run, and what is the fix?
- (a) 1 query; GraphQL combines them automatically
- (b) 21 queries; add request-scoped batching so the level collects ids and asks once ✅
- (c) 20 queries; increase the connection pool size
- (d) 2 queries; the fix is to add a depth limit
> This is M04's N+1 inside a resolver tree: one for the orders, one per order. Batching collects the 20 ids at that level and issues a single lookup. A bigger pool would hide it for a while and cost you more.

### Q2 · L4
You have a depth limit of 8. A client sends a two-level query that asks for 50,000 rows. What happens?
- (a) The depth limit rejects it, because row count counts toward depth
- (b) It is served, and a complexity limit is what would have stopped it ✅
- (c) GraphQL automatically paginates any field returning more than 100 rows
- (d) The query fails with an HTTP 500, which your monitoring will catch
> Depth and breadth are different attacks. A complexity limit scores each field and enforces a budget per query, which is the only one of the three limits that catches a shallow but enormous request.

### Q3 · L3
Why does a failing GraphQL resolver often leave your error-rate dashboard flat?
- (a) GraphQL retries failed resolvers silently before responding
- (b) Errors arrive inside a 200 response as an `errors` array, so HTTP-status monitoring sees success ✅
- (c) Resolver failures are logged at debug level by default
- (d) The error rate only counts errors on `GET` requests
> This breaks the assumption M07's monitoring is built on. You must count GraphQL errors explicitly, or your users see blank fields while every graph you own looks healthy.

### Q4 · L4
You want to rename a GraphQL field, and your Android app is already in the Play Store. What do you do?
- (a) Rename it and ship an app update the same day
- (b) Add a `v2` endpoint alongside the existing one
- (c) Add the new field, mark the old one deprecated with a reason, and remove it only after usage reaches zero ✅
- (d) Keep both names pointing at the same resolver forever
> GraphQL has no versions, so deprecation is the only tool. You do not control when users update a mobile app, which makes field-level usage data the thing that tells you when removal is safe.

### Q5 · L5
A teammate renumbers a protobuf field from 7 to 9 "to tidy up the file", keeping the name identical. An older service is still deployed. What happens?
- (a) The older service raises a type error and the call fails loudly
- (b) gRPC falls back to matching by field name, so nothing breaks
- (c) The old service keeps writing number 7, the new one reads something else there, and data is silently wrong ✅
- (d) The contract test fails at compile time, preventing deployment
> Numbers are the wire identity; names are for humans. This is the failure mode with no error message, which is why a retired number must be marked reserved forever and a contract test in CI is the only thing that catches a renumber before it ships.

## Go deeper
- [GraphQL: best practices, including versioning and deprecation](https://graphql.org/learn/best-practices/)
- [Protocol Buffers: updating a message type without breaking clients](https://protobuf.dev/programming-guides/proto3/#updating)
