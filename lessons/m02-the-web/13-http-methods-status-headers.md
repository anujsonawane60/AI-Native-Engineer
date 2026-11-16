---
id: m02-13-http-basics
module: m02-the-web
slug: http-methods-status-headers
title: "HTTP: Methods, Status Codes and Headers"
order: 13
duration_min: 25
is_free: false
mission: false
volatility: low
last_reviewed: 2026-11-16
---

## Key learnings
- Read a request's method, status and headers and say what happened
- Explain why a GET must never change anything, with a real consequence
- Tell the four status families apart and pick the right one for a failure
- Reject a 200 response that carries an error inside it
- Name the five HTTP red flags that let failures look like successes

## Lesson

### The pain

A team builds a "remove from cart" link. It is a link, so it is a GET request, and visiting it deletes the item. It works.

Then two things happen. A student's browser pre-fetches links on the page to make navigation feel faster, and quietly empties their cart before they tap anything. And a search crawler finds the page, follows every link on it as crawlers do, and deletes forty rows.

The same week, the order endpoint develops a different problem. When the database is unavailable it returns HTTP 200 with a body of `{"success": false, "error": "db timeout"}`. The page checks the status code, sees 200, shows "Order placed", and clears the cart. Thirty-one students believe they have ordered lunch. The canteen has no record of any of them.

Nothing in either story is a hard bug. Both are HTTP used dishonestly, and both are visible in the Network panel in ten seconds.

### The idea

**Every interaction on the web is one request and one response. The request carries a method saying what kind of thing it is; the response carries a status code saying how it went. Both carry headers, which are the small print.**

The method is a promise about intent. The status code is the verdict. When either one lies — a GET that deletes, a 200 that failed — every piece of software in the chain that trusted it does the wrong thing, including browsers, crawlers, caches and your own page.

### The analogy

The canteen order slip, and what the counter writes back on it.

The **method** is what kind of slip you are handing over. "Show me the menu" is a different kind of request from "place this order", which is different again from "cancel order 412". The counter treats them differently and so does everybody watching: nobody minds you asking to see the menu twice, and everybody minds you placing the same order twice.

The **status** is the stamp the counter puts on the slip when it hands it back. SERVED. NO SUCH DISH. NOT YOUR ORDER. COUNTER CLOSED. The stamp is how you know what happened without reading the whole reply.

The **headers** are the margin notes on both: who you are, what language you read, whether this reply may be pinned up for others to read, how long it stays valid.

**Where the analogy breaks:** the counter remembers you. If you place an order and come back a minute later, the same staff member knows who you are. HTTP does not — every request is a complete stranger arriving for the first time, with no memory of anything before it. That is what "stateless" means, and it is the reason cookies exist, which is chapter 14.

### Picture it

```
  REQUEST                              RESPONSE
  ┌──────────────────────────┐         ┌──────────────────────────┐
  │ POST /api/orders         │         │ 201 Created              │
  │                          │         │                          │
  │ Content-Type: app/json   │         │ Content-Type: app/json   │
  │ Authorization: Bearer …  │  ────▶  │ Cache-Control: no-store  │
  │                          │  ◀────  │                          │
  │ { "dish_id": 7,          │         │ { "order_id": 412,       │
  │   "qty": 2 }             │         │   "total_paise": 9000 }  │
  └──────────────────────────┘         └──────────────────────────┘
    method + path, headers,              status, headers, body
    then the body
```

A request is a method and a path plus headers and an optional body; a response is a status code plus headers and a body, and the status is what tells you how to read the rest.

### The methods, as promises

| Method | Promise | Safe to repeat? |
|---|---|---|
| GET | Just give me this; change nothing | Yes |
| POST | Create something, or do something | **No** |
| PUT | Make it be exactly this | Yes |
| PATCH | Change these fields of it | Usually |
| DELETE | Remove it | Yes |

Two words in that table do the real work. **Safe** means it changes nothing — GET is safe, which is precisely why browsers and crawlers feel free to fetch GETs unasked. **Idempotent** means doing it twice has the same result as doing it once: PUT ("set quantity to 3") is idempotent, POST ("add one") is not. That distinction is the whole reason chapter 10 said never to auto-retry a write, and it is M04's idempotency chapter in one line.

### The status families

The first digit is the category, and knowing the five families is most of what you need.

**2xx — it worked.** 200 OK, 201 Created for something new, 204 No Content when there is nothing to return.

**3xx — look elsewhere.** 301 permanent redirect, 302 temporary. Caches and search engines treat these two very differently, so using 301 for something temporary is hard to undo.

**4xx — you got it wrong.** 400 malformed request, 401 "I do not know who you are", 403 "I know who you are and you may not", 404 not found, 409 conflict, 422 understood but invalid, 429 too many requests.

**5xx — we got it wrong.** 500 something broke, 502 and 503 the service is unavailable or overloaded.

The 4xx/5xx line is the one that matters operationally: 4xx means the caller must change something, 5xx means the caller should possibly try again later and somebody on your team needs to look at a log. Reporting your own crash as a 400 sends your users hunting for a mistake they did not make.

### Where it lives

HTTP is the language spoken across the network box from M00, between every client and every server, for everything: the HTML document, each image, each API call. Chapter 10's `fetch` is how your page speaks it; chapter 3's Network panel is where you read it.

M04 is where you design these contracts properly — which method, which status, what shape the error takes. M05 uses headers for caching and uses 429 for rate limiting. M07's observability is built on status codes: "the proportion of 5xx responses" is one of the first things you will ever put on a dashboard.

### Words to own

| Term | Plain meaning | Example |
|---|---|---|
| Request | One trip from client to server | `GET /api/menu` |
| Response | What comes back | `200` plus the JSON |
| Method | What kind of request this is | GET, POST, DELETE |
| Path | Which thing on the server | `/api/orders/412` |
| Status code | The three-digit verdict | 201, 404, 503 |
| Safe | Changes nothing on the server | GET |
| Idempotent | Repeating it has no extra effect | PUT, DELETE |
| Header | A labelled piece of small print | `Content-Type` |
| Body | The payload carried with it | The order JSON |
| `Content-Type` | What format the body is in | `application/json` |
| `Authorization` | Who is asking | A token |
| `Cache-Control` | Who may keep a copy, and for how long | `no-store` |
| Stateless | Each request arrives with no memory of the last | Why cookies exist |

### Tradeoffs and gotchas

**A GET that changes something will eventually be called by a machine.** Browser prefetch, link previews in WhatsApp, antivirus scanners, crawlers and your own monitoring will all fetch your GETs without a human deciding to. Anything that writes must be a POST, PUT, PATCH or DELETE, and this is not a style preference.

**A 200 with an error inside it is the most harmful pattern in this chapter.** It is also extremely common in AI-generated code, because it is easier to write and because it makes the happy path shorter. The cost is that every layer above — your page, a cache, a monitoring tool, your own dashboards — is told everything is fine. A failure must be a 4xx or a 5xx.

**401 and 403 are not interchangeable.** 401 means you are not identified; the fix is to log in. 403 means you are identified and not allowed; logging in again changes nothing. Returning 401 for an authorisation failure sends users into a login loop, and it is exactly M00's broken-Google-login scenario wearing a different hat.

**Choose between 404 and 403 deliberately for things that exist but are not yours.** Returning 404 hides whether the record exists at all, which is sometimes the right privacy choice and sometimes an unhelpful lie. M07 covers this; the point here is that it is a decision.

**429 needs a retry delay and a backoff.** Hammering a rate-limited endpoint harder is how a small problem becomes an outage. The `Retry-After` header exists for this, and M05 goes into it.

**Headers are not secret.** They travel with every request, they are visible in the Network panel, and the user can read and change them. A token in a header is only as safe as HTTPS makes the connection — which is chapter 14 — and never as safe as a server-side check.

**Status codes you invent are a trap.** 299 and 418 may be fun; no cache, proxy, browser or monitoring tool knows what to do with them.

### Verify it

The Network panel is the whole verification surface, and the discipline is to read the status rather than the page.

1. **For every action, read the method.** Network panel, click the request, read the method column. Anything that creates, changes or deletes must not be a GET. This single check prevents the crawler story.
2. **Read the status code, not the screen.** The page saying "Order placed" is the page's opinion. The status code is the server's. Where they disagree, the status code is the fact.
3. **Open the response body on every "success".** Look for `"success": false`, `"error"`, or an empty result inside a 200. If you find one, the endpoint is reporting failures as successes and every consumer downstream is being misled.
4. **Force each failure and check its code.** Submit a bad value and confirm you get a 400 or 422, not a 500. Log out and retry and confirm 401. Try someone else's order and confirm 403 or 404, never 200. Chapter 3's techniques make all of these a two-minute exercise.
5. **Ask, for every write: "is this safe to repeat?"** If no, it must be a POST and it needs M04's protection. If yes, say why — usually because it sets a value rather than adjusting one.
6. **Check `Content-Type` matches the body.** JSON announced as HTML, or HTML announced as JSON, produces parse errors that look like network errors and waste a lot of time.
7. **Look at what is sent, not only what comes back.** Expand the request headers and payload. An `Authorization` header that is absent explains a 401 instantly.

Red flags in AI output: any GET with side effects; 200 responses carrying errors; 500 used for user mistakes or 400 used for server crashes; 401 where 403 belongs; invented status codes; no handling of 429; a response with no `Content-Type`; and error bodies that leak stack traces or database messages to the user, which is both a bad experience and an M07 security finding.

## Walkthrough
1. Open a site you use often with DevTools on the Network panel. Make sure the Method and Status columns are visible — right-click the column headers if they are not.
2. Reload and look at the very first request, the document itself. Note its method, its status, and how long it took.
3. Filter to Fetch/XHR and perform a read action on the site — a search, a filter, opening a detail page. Note the method of each resulting request. They should all be GET.
4. Now perform a write action — add something to a cart, save a preference, submit a form. Note the method. Write down whether it was a POST, PUT, PATCH or DELETE, and whether that matches what the action actually does.
5. Click that write request and open its Headers section. Read the request headers: find `Content-Type` and, if present, `Authorization`. Then read the response headers and find `Cache-Control`.
6. Open the response body of that successful write. Read it fully. Check whether it is genuinely a success or whether there is an `error` or `success: false` field inside a 2xx status.
7. Now deliberately cause a failure. Submit a form with an obviously invalid value, or request a URL for an item id that cannot exist, like `/orders/99999999`. Record the status code you get back.
8. Judge that status code: was it the right family? A user mistake answered with 500, or a server crash answered with 400, is a finding worth writing down.
9. Log out, or open a private window, and try a URL that requires being logged in. Record whether you get 401, 403, a redirect, or a 200 with an empty page. Each of those is a different design decision.
10. Pick any three write actions on the site and, for each, write one sentence answering "would doing this twice be harmful?" Then say whether the method used is appropriate given your answer.
11. Write up five findings with the method, the status and the body for each, and name the one place where this site's HTTP is dishonest.

## Talk to AI
```prompt
CONTEXT:      I direct AI rather than writing backend code. My product is a
              canteen ordering app for students in Nashik on budget Android
              phones on congested 4G. The API needs these operations: list
              today's dishes, get one dish, place an order, change an
              order's quantity, cancel an order, and list my own past
              orders. There are two roles: student and canteen staff. Money
              is integer paise. There is no rate limiting yet. I have been
              burned before by a 200 response that was actually a failure.

GOAL:         An HTTP contract for these six operations that cannot let a
              failure look like a success, and that no browser, crawler or
              retry can damage by calling it twice.

REQUIREMENTS: 1. A table with one row per operation: method, path, what
                 the body contains, the success status code, and whether
                 the operation is safe and whether it is idempotent
              2. For each operation, the complete list of failure statuses
                 it can return and exactly what each one means to the
                 caller — including the difference between 401 and 403
              3. One single error body shape used everywhere, which cannot
                 be mistaken for a success, plus a rule that errors always
                 carry a 4xx or 5xx status
              4. An explicit decision, with reasoning, about whether a
                 student requesting another student's order gets 403 or 404
              5. Which operations a browser prefetch or a crawler could
                 safely trigger, and proof that none of them write
              6. What a 429 response must include so my page can back off
                 correctly, even though rate limiting does not exist yet

CONSTRAINTS:  REST over HTTPS. No operation that writes may be a GET. No
              2xx status may ever accompany an error. No stack traces,
              database messages or internal identifiers in any error body
              shown to a user. No invented status codes. Keep the whole
              contract to one page I can print and check responses against
              in the Network panel.

NON-GOALS:    Do not design the database. Do not write any implementation.
              Do not add authentication flows — assume a token exists. Do
              not add operations I did not list; if you think one is
              missing, tell me separately.

ACCEPTANCE:   - Every write operation uses a non-GET method
              - Every row of the table states safe and idempotent
                explicitly
              - I can read any response in the Network panel and tell
                success from failure using the status code alone
              - The 401 versus 403 distinction is stated in a sentence I
                could explain to a teammate
              - Cancelling the same order twice produces a sensible,
                non-harmful result, and the contract says what it is
              - No error body contains anything I would not show a student

ASK:          Before writing the contract, ask me up to five questions
              whose answers would change it. Then produce the table. Then,
              separately, list the three places where you think my
              requirements will cause a problem later and what you would
              have chosen instead. Do not change the contract based on that
              list without my approval.
```

## Stuck? Ask AI
```prompt
Explain the difference between "safe" and "idempotent" as if I have never programmed, with a non-software example of each. Then give me eight operations from a food-ordering app and ask me whether each is safe, idempotent, both or neither, and which HTTP method it should use. Do not tell me the answers until I have attempted all eight.
```

```prompt
I have a response with status 200 and a body that says the operation failed. Explain, concretely, every piece of software in the chain that is now being misled and what each one does wrong as a result — the browser, any cache, my own page, and a monitoring dashboard. Then tell me what the correct status should have been and how I would spot this pattern in the Network panel.
```

```prompt
Give me ten realistic failure situations from a web app — wrong password, expired token, someone else's order, malformed JSON, a database outage, too many requests, and four more. For each, ask me which status code I would return and why. After I answer all ten, tell me which ones I got wrong, what the correct code is, and what goes wrong for the user when the wrong one is used.
```

## Quiz

### Q1 · L2
What does the first digit of a status code tell you?
- (a) How long the request took
- (b) The category of outcome: 2xx worked, 3xx look elsewhere, 4xx caller's fault, 5xx server's fault ✅
- (c) Which HTTP method was used
- (d) Whether the response was cached
> The family is most of the information. The 4xx/5xx line in particular decides whether the caller must change something or whether somebody on your team needs to read a log.

### Q2 · L3
A "remove from cart" action is implemented as a link, so a GET. A crawler finds the page. What happens?
- (a) Nothing — crawlers do not follow links
- (b) The crawler follows every link and deletes rows, because a GET is assumed to change nothing ✅
- (c) The server rejects the crawler's request automatically
- (d) The cart is emptied only for the crawler's own session
> Browsers prefetch, link previews fetch, scanners fetch and crawlers crawl — all without a human deciding to. That is only safe because GET promises to change nothing, so a writing GET gets called by machines.

### Q3 · L4
An endpoint returns `200` with `{"success": false, "error": "db timeout"}`. Your page shows "Order placed". What is the root fault?
- (a) The page should have read the body before trusting the status
- (b) The endpoint reported a failure as a success, misleading every layer above it ✅
- (c) The database timeout should have been retried automatically
- (d) The error message was too technical for users
> Patching the page would fix one consumer. Caches, monitoring and dashboards are all still being told it worked. A failure must carry a 4xx or 5xx status.

### Q4 · L4
A logged-in student opens a page they are not permitted to see. Which status, and why does the choice matter?
- (a) 401, because they need to authenticate again
- (b) 403, because they are identified and simply not allowed ✅
- (c) 500, because the request should not have been possible
- (d) 200 with an empty page, to avoid confusing them
> 401 means "I do not know who you are", so it sends an already-logged-in user into a login loop that can never succeed. 403 states the real problem.

### Q5 · L5
Your page retries a failed "place order" request automatically. Which single change makes this defensible?
- (a) Reducing the retry delay so the user waits less
- (b) Switching the method from POST to GET so it can be cached
- (c) Making the operation idempotent so repeating it cannot create a second order ✅
- (d) Returning 200 on both attempts so the page does not show an error
> POST is not idempotent by nature, so retrying it is retrying the effect. Making repetition harmless — M04's idempotency keys — is what turns an unsafe retry into a safe one.

## Go deeper
- [MDN: HTTP response status codes](https://developer.mozilla.org/en-US/docs/Web/HTTP/Reference/Status)
- [MDN: HTTP request methods](https://developer.mozilla.org/en-US/docs/Web/HTTP/Reference/Methods)
