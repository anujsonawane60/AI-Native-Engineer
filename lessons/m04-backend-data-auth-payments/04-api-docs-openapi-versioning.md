---
id: m04-04-api-docs-openapi-versioning
module: m04-backend-data-auth-payments
slug: api-docs-openapi-versioning
title: "API Docs, OpenAPI and Versioning"
order: 4
duration_min: 30
is_free: false
mission: true
volatility: low
last_reviewed: 2026-11-16
---

## Key learnings
- Write an API spec your teammate can build against without asking you anything
- Tell a breaking change from a safe one before you ship it
- Use a written spec as the brief that keeps two AI sessions in agreement
- Pick a versioning approach and know when you genuinely need one
- Verify that the spec and the running server actually say the same thing

## Lesson

### The pain

Three weeks into the canteen project, the API has eleven endpoints. Nobody can list them.

Priya asks what `GET /api/orders` returns. Rohit opens the file, reads for two minutes, and says "an array of orders". She asks whether each order includes the items. He is not sure, so he calls it, looks at the output, and says yes — but only for orders placed after Tuesday, because something changed on Tuesday and he does not remember what.

Meanwhile Priya's AI session, which has not seen the server code, has been confidently generating client calls against the API it imagined in week one. Half of them are wrong in small ways: a field renamed, a status code different, a date format that used to be a timestamp and is now a string.

The project has no bug. It has eleven undocumented contracts and three people guessing. On Friday Rohit renames `total` to `total_paise` to follow the money rule — a genuinely good change — and the deployed app shows ₹0 for every order, because nobody knew who was reading that field.

### The idea

**The spec is the single source of truth for your API, and it is a document, not a side-effect of the code.**

An API specification lists every endpoint, every field, every type, every status code. OpenAPI is the standard format for writing that down — a structured file, machine-readable, which tools turn into browsable documentation, test clients and type definitions.

Two properties make it worth the hour it costs. It lets two people build opposite sides of a seam at the same time. And in this program it is something more valuable still: it is a brief AI cannot drift away from, because you paste it into every session.

### The analogy

The canteen's menu board is the spec.

Written on the wall, visible to everyone, it says exactly what exists and what it costs. A new student can walk in and order correctly on day one without talking to the staff. A new cook can read it and know what they must be able to produce.

Now imagine the canteen raises the price of a samosa and does not change the board. Students bring ₹12 and the cashier wants ₹15. Nobody did anything wrong at the till; the board drifted from reality, and the board is what everyone trusts. The fix is not "tell people to ask the cashier" — it is to make changing the board part of changing the price.

**Where the analogy breaks:** a menu board is read by humans who can cope with ambiguity. "Rice plate — ₹40" is enough for a person and nowhere near enough for a program, which needs to know whether `price` is rupees or paise, an integer or a string, and whether it can ever be null. API specs are tediously precise because the reader cannot infer anything.

### Picture it

```
            ┌──────────────────┐
            │   openapi.yaml   │  one file, in git
            │  the single      │
            │  source of truth │
            └────┬───┬───┬─────┘
                 │   │   │
      ┌──────────┘   │   └──────────┐
      ▼              ▼              ▼
┌───────────┐  ┌───────────┐  ┌───────────┐
│ browsable │  │ pasted in │  │ server    │
│ docs page │  │ every AI  │  │ built to  │
│ for team  │  │ brief     │  │ match it  │
└───────────┘  └───────────┘  └───────────┘
      │                              │
      └────── do these two agree? ───┘
                (the only check
                 that matters)
```

One spec file in the repo feeds the team's docs, every AI brief, and the server itself — and the one check worth running is whether the spec and the running server still agree.

### Where it lives

The spec sits beside your PRD from M01, in the repository, under version control, and it changes through pull requests like anything else. A reviewer reading a PR that changes the spec is reviewing a product decision, not a formatting change.

Downstream it feeds a lot. In M06 your tests assert the shapes the spec promises, so a drift becomes a failing build rather than a surprise on Friday. In M07 the spec is the list of routes a security review has to cover — an endpoint absent from the spec is an endpoint nobody audited. In M13 an API gateway is configured from one.

And in this program specifically, it is the most reliable way to stop two AI sessions inventing two different APIs. "Here is our OpenAPI spec. Implement the `POST /api/orders` operation exactly as specified, and change nothing else" is a brief with no room to drift.

### Breaking versus safe changes

This is the distinction that matters most, and it is learnable in five minutes.

| Change | Breaking? | Why |
|---|---|---|
| Add a new optional field to a response | No | Old clients ignore it |
| Add a new endpoint | No | Nothing was reading it |
| Add an optional request field | No | Old callers omit it |
| Rename a response field | **Yes** | Old clients read the old name |
| Remove a field | **Yes** | Something was using it |
| Make an optional request field required | **Yes** | Old callers do not send it |
| Change a type: integer paise to "₹240.00" | **Yes** | Parsing breaks, money breaks |
| Change a success code from 200 to 204 | **Yes** | Clients branch on the code |
| Tighten validation, e.g. max qty 10 | **Yes** | Requests that worked now fail |

The rule behind the table: **adding is safe, changing and removing are not.** Anything a client could already be depending on is frozen the moment a second person depends on it.

Which brings up versioning. The common approach is a prefix in the path: `/api/v1/orders`, and when a breaking change is unavoidable you publish `/api/v2/orders` and run both for a while. Alternatives exist — a version header, a date-based version like the one Stripe uses — but a path prefix is the one your whole team can see, and that visibility is worth more than elegance.

Here is the part most courses skip: **for a three-person project with a single web client you deploy yourself, you do not need versioning yet.** You can make a breaking change and update both sides in the same pull request. Versioning becomes necessary the moment you have a client you cannot update in lockstep — a published mobile app on students' phones from M08, or another team consuming your API. Put `/api/v1/` in the path on day one because it costs nothing, and do not build version negotiation machinery until something forces you to.

### Words to own

| Term | Plain meaning | Example |
|---|---|---|
| Specification / spec | The written definition of your API | `openapi.yaml` |
| OpenAPI | The standard format for writing that spec | YAML or JSON |
| Schema | The defined shape of one object | an `Order` |
| Operation | One method on one path | `POST /api/v1/orders` |
| Required field | One that must be present | `menu_item_id` |
| Nullable | Allowed to be empty, deliberately | `note: string \| null` |
| Breaking change | One that breaks existing callers | renaming a field |
| Deprecated | Still works, but will be removed | announced with a date |
| Versioning | Running two shapes at once on purpose | `/v1/` and `/v2/` |
| Contract drift | Spec and server no longer agree | the Friday ₹0 bug |

### Tradeoffs and gotchas

**A spec that nobody updates is worse than no spec.** No spec makes people ask. A stale spec makes them confidently wrong, which is how the ₹12 samosa happens. The rule that actually holds: the pull request that changes behaviour changes the spec, or it does not merge. M01's PR review is where you enforce it.

**Generated-from-code docs describe what is, not what was agreed.** Deriving the spec automatically from the running server guarantees they agree, which sounds ideal. It also means a sloppy change silently rewrites the contract, and the spec stops being a decision anyone made. For a learning team, writing the spec first — as a design step — is worth more than the automation.

**Do not version endpoints individually.** `/api/v1/orders` with `/api/v3/menu` beside it is a maze. Version the whole surface or none of it.

**AI will happily write a 400-line spec for four endpoints.** Every field gets a description, examples, an `operationId`, and three error schemas nobody will use. Ask for the minimum that makes the contract unambiguous. If nobody reads it, it will not be maintained, and see gotcha one.

**Dates and money are where types quietly break.** A timestamp that was a Unix integer and is now an ISO string looks identical in a screenshot and crashes a client. Pin them in the spec: money is integer paise, timestamps are ISO 8601 in UTC. Write it once, repeat it in every brief.

**Removing a field is not a cleanup task.** It is a release with notice. Mark it deprecated, say when it goes, and check who reads it first.

### Verify it

Four checks, and the third is the one people skip.

1. **Can a teammate build a client against the spec with zero questions?** Hand it over and watch. Every question they ask is a hole in the spec; write the answer into the file rather than into the chat.
2. **Does every operation list its error responses, not just its success?** A spec with only `200` responses is half a contract and guarantees the client has no error handling.
3. **Do the spec and the server actually agree, right now?** Call the endpoint. Compare the live JSON to the spec field by field: names, types, whether nulls appear, and the exact status code. This is the only check that catches drift, and it is the reason the ₹0 bug lived until Friday.
4. **Is the money integer paise and are the timestamps ISO 8601 UTC everywhere?** Mixed conventions across endpoints will cost you an afternoon in M11.

Red flags in AI-written specs: a response schema where every field is optional, which promises nothing; `type: object` with no properties, which is a shrug in YAML; examples that contradict the schema, such as `"price_paise": 240.5`; a `PUT` operation documented without saying whether omitted fields are cleared or left alone; and a spec that mentions authentication nowhere, which means chapter 12's failure is already in your design.

## Walkthrough
1. List every endpoint your project has today, on paper, from memory. Then open the code and list them again. The difference between the two lists is what nobody on the team knows.
2. Pick the three most important endpoints. For each, call it against your running server and save the actual JSON response to a file.
3. Write the five-line contract for each from chapter 1: method, path, auth and role, request body, success status and body, every error status with its condition.
4. Now brief AI to turn those three contracts into a minimal OpenAPI file, with the constraint that money is integer paise and timestamps are ISO 8601 UTC, and that no field may be marked optional unless it genuinely can be absent.
5. Read the generated spec line by line against the real JSON from step 2. Mark every disagreement. There will be some.
6. For each disagreement, decide which one is wrong — the spec or the server. That decision is a product decision, so write down why.
7. Render the spec as a browsable docs page using any free OpenAPI viewer and have a teammate click through it. Note every question they still have to ask you.
8. Pick one field in one response and write down what it would take to rename it safely: who reads it, what breaks, and whether you can update both sides in one pull request.
9. Add the `/api/v1/` prefix to your paths if it is not there. Note in your decision log that you are not building version negotiation yet, and what would force you to.
10. Commit the spec. In the commit message, say what the spec now guarantees that was previously folklore.
11. Open a pull request that changes one endpoint's behaviour and its spec entry together. Have a teammate review both halves and confirm they match.

## Talk to AI
```prompt
CONTEXT:      Three-person student team building [one sentence on your
              product]. Free tiers only. Users on budget Android phones.
              We have [N] endpoints already running, written by AI across
              several separate sessions, and no written spec. Two of us are
              now building against the same API and keep disagreeing about
              what it returns.
              Our conventions: all money is integer paise in fields named
              *_paise; all timestamps are ISO 8601 in UTC; field names are
              snake_case.
              Here are the three endpoints that matter most, with the real
              JSON each currently returns:
              [paste method, path and the actual response for each]
              Here is what each one currently does about authentication:
              [describe, honestly, including "nothing"]

GOAL:         A minimal OpenAPI 3.1 specification for these three
              endpoints that my teammate can build a client against without
              asking me a single question.

REQUIREMENTS: 1. One operation per endpoint, with request body schema,
                 success response schema and EVERY error response with the
                 condition that triggers it
              2. Mark a field optional only if it can genuinely be absent;
                 say which fields are nullable and why
              3. State auth requirements per operation, including the role
              4. Keep it as short as possible while leaving nothing
                 ambiguous — no decorative descriptions
              5. After the spec, give me a table of every place where the
                 real JSON I pasted disagrees with a sensible contract, and
                 for each one tell me whether I should fix the server or
                 write the reality into the spec
              6. Then list which of my fields could not be renamed later
                 without breaking a client, and why

CONSTRAINTS:  OpenAPI 3.1, YAML, one file. No code generation. No new
              endpoints. Do not invent fields that are not in the JSON I
              pasted. Assume a single web client we deploy ourselves.

NON-GOALS:    Do not implement anything. Do not design versioning machinery
              beyond putting /api/v1 in the paths. Do not add pagination,
              filtering or sorting that does not already exist.

ACCEPTANCE:   - Every operation documents at least two error responses
              - No money field is a float; none is named without _paise
              - A teammate reading only the spec can construct a valid
                request and handle a failed one
              - Your disagreement table has at least one row, because
                AI-written endpoints always drift

ASK:          First show me the disagreement table, before the spec. I want
              to see what is actually inconsistent in my API before you
              write down a tidy version of it.
```

## Mission

**Objective.** A committed OpenAPI spec for your own product's three most important endpoints, proven to match the running server, with every disagreement between spec and reality written down and resolved.

**Steps.**
1. **Frame** — name the three endpoints that matter most in your product and, in one sentence each, who calls them and why.
2. **Capture reality** — call each one against your running server and save the actual response JSON. If an endpoint does not exist yet, write the contract you intend before you build it.
3. **Contract** — write the five-line contract for each by hand: method, path, auth and role, request body, success status and body, every error status with its trigger. By hand, before AI sees it.
4. **Brief** — give AI your contracts, your real JSON, and your conventions (integer paise, ISO 8601 UTC, snake_case). Demand the disagreement table before the spec.
5. **Resolve** — for every disagreement, decide whether the server is wrong or the spec should record reality. Write one line of reasoning for each decision.
6. **Verify** — call all three endpoints again and check the live response field by field against the final spec: names, types, nulls, status codes. Also call each one while logged out and record what happened.
7. **Review** — render the spec as a docs page, hand it to the teammate who did not write it, and have them build or describe a client call using only the spec. Log every question they had to ask you, then close each hole in the file.
8. **Freeze** — list the fields that can no longer be renamed without a breaking change, and note what your plan would be if you had to rename one.
9. **Document** — commit the spec with a message that says what it now guarantees, and add a decision-log entry on versioning: what you chose and what would force you to change.

**Upload.**
- Your three hand-written five-line contracts, before AI → post caption or attached file
- Your committed `openapi.yaml` → attached file, or a path plus a screenshot of the rendered docs page
- The AI conversation including the disagreement table → share link in `ai_chat_url`
- Proof of verification → screenshots of the live responses beside the spec entries, including one logged-out attempt
- The list of questions your teammate still had to ask, and how you closed each one
- Your frozen-fields list and your versioning decision-log entry
- Reflection, 150 words max → the most surprising disagreement you found between your spec and your server, and one thing the AI documented wrongly

**Self-check.** Tick every box before you submit.
- [ ] I wrote the contracts by hand before briefing the AI
- [ ] Every operation in my spec documents at least two error responses
- [ ] Every money field is an integer named with a `_paise` suffix
- [ ] Every timestamp field is ISO 8601 in UTC, stated in the spec
- [ ] I called each endpoint and compared the live JSON field by field, rather than trusting the AI's summary
- [ ] My disagreement table has at least one real row, with my decision and reason for each
- [ ] I called at least one endpoint logged out and recorded the status code I got
- [ ] A teammate built or described a client call from the spec alone, and I closed every hole their questions exposed
- [ ] My paths carry `/api/v1/` and my decision log says why I am not building more versioning yet
- [ ] My frozen-fields list names at least one field I can no longer rename freely

**Stretch.** Paste your finished spec into a brand-new AI session with no other context and ask it to implement one endpoint exactly as specified. Compare what it produces to your existing server. Where they differ, the spec was ambiguous — fix the spec, not the code.

## Stuck? Ask AI
```prompt
Here is my OpenAPI spec: [paste it]. Do not improve it. Instead, act as a teammate who has to build a client using only this document and nothing else. List every question you would have to ask me, in priority order, and for each one say what could go wrong if I answered it wrongly.
```

```prompt
Give me ten proposed changes to an API — some breaking, some safe — for a food-ordering app. Ask me to sort them into breaking and non-breaking and explain my reasoning for each, without telling me the answers first.
```

```prompt
I want to rename a response field from `total` to `total_paise`. Walk me through how to find out who is currently reading that field, and give me two different ways to make the change — one where I control every client, and one where a published mobile app I cannot update is reading it.
```

## Quiz

### Q1 · L2
Which of these is a breaking change to your API?
- (a) Adding a new optional field to a response
- (b) Adding a brand new endpoint
- (c) Making an existing optional request field required ✅
- (d) Adding an optional query parameter with a default
> Adding is safe because nothing was depending on it. Making a field required means every existing caller that omits it starts failing immediately.

### Q2 · L3
Your three-person team has one web client you deploy yourselves. Why is full API versioning machinery not yet worth building?
- (a) Versioning is only for GraphQL APIs
- (b) You can change the server and the client in the same pull request ✅
- (c) Free tiers do not support multiple versions
- (d) Versioning is only needed once you have paying customers
> Versioning exists for clients you cannot update in lockstep. Put `/api/v1/` in the path because it is free, and build negotiation when a published mobile app or an outside consumer forces it.

### Q3 · L4
A spec and a server disagree: the spec says `total_paise` is an integer, the live response returns `"240.00"`. Which check would have caught this, and when?
- (a) A teammate reading the spec carefully
- (b) Calling the endpoint and comparing the live JSON to the spec field by field ✅
- (c) Rendering the spec as a browsable docs page
- (d) Counting the endpoints in the code against the spec
> Drift is only visible when you compare the document to reality. Reading either one alone looks fine, which is exactly why the ₹0 bug survived until Friday.

### Q4 · L4
AI hands you a spec where every response field is marked optional. Why is that a problem?
- (a) It makes the YAML file longer than necessary
- (b) The contract now promises nothing, so clients must defend against every field being absent ✅
- (c) Optional fields cannot be cached
- (d) OpenAPI 3.1 does not allow optional response fields
> A contract is only useful where it commits. "Everything might be missing" forces the client to handle every combination, which is the same as having no spec at all.

### Q5 · L5
Why is a stale spec worse than having no spec?
- (a) It takes up space in the repository
- (b) Tools will refuse to render it
- (c) It makes people confidently wrong instead of making them ask ✅
- (d) It prevents the server from starting
> No board means students ask the price. A wrong board means they bring ₹12 for a ₹15 samosa. The fix is procedural: the pull request that changes behaviour changes the spec, or it does not merge.

## Go deeper
- [OpenAPI Specification, current version](https://spec.openapis.org/oas/latest.html)
- [Stripe API reference — a well-maintained spec in practice](https://docs.stripe.com/api)
