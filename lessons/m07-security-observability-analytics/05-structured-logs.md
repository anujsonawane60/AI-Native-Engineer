---
id: m07-05-structured-logs
module: m07-security-observability-analytics
slug: structured-logs
title: "Structured Logs, Levels and What Is Worth Logging"
order: 5
duration_min: 15
is_free: false
mission: false
volatility: low
last_reviewed: 2026-11-16
---

## Key learnings
- Tell a structured log line from a sentence, and say why the difference matters
- Choose the right level for an event, and know what each level promises
- Insist on a request id that ties one user's story together
- List what must never reach a log, and why that list is legal as well as ethical
- Spot a log that is expensive, noisy and still unable to answer your question

## Lesson

### The pain

A team's app starts failing at checkout. They open the logs and find 46,000 lines from the last two hours. Most say `here`, `here 2`, `data: [object Object]`. Several hundred print a whole order object. Four print something that looks like a password.

Searching for "error" returns 900 results, because an earlier session logged every retry as an error — and no line says which request it came from, so none can be tied to the student who complained. Twenty minutes later the log store hits its monthly quota and stops accepting lines. The app is still broken, in the dark.

### The idea

**A log line is a data record, not a sentence. Write it as fields so a machine can filter it, because you will have too many lines to read.**

Writing for a human reading top to bottom works until you have a thousand lines. After that only filtering works — every error, for this user, on this endpoint, in this hour — and filtering requires fields. One field is worth more than the rest together: **a request id, generated when a request arrives and attached to every line it produces.** Without it, logs are a shuffled pile from hundreds of concurrent users; with it you type one id and read one student's story in order. AI will not add it unasked.

### The analogy

The canteen's useful sales register has columns: time, item, quantity, price, who served it. You can answer "how many samosas before noon" by running a finger down one column. The useless register is a diary: "busy morning, sold a lot of samosas, the idli ran out around eleven". Every fact is in there and none can be counted. Same information — one is data, one is prose.

**Where the analogy breaks:** a register costs nothing but shelf space. Logs cost money per gigabyte every day, and on a free tier they stop working when the quota runs out. A canteen never has to decide to throw away March; you do, on day one.

### Picture it

```
  UNSTRUCTURED  — a sentence
  "Error placing order for Priya at 9:14pm - timeout"
        │
        └─▶ to find every timeout you search text and hope

  STRUCTURED  — a record
  { "ts":         "2026-11-16T21:14:03Z",
    "level":      "error",
    "event":      "order_failed",
    "request_id": "r-8f21c4",
    "user_id":    4412,
    "route":      "POST /orders",
    "reason":     "db_timeout",
    "duration_ms": 9140 }
        │
        └─▶ filter: level=error AND reason=db_timeout
```

The same event written as a sentence can only be searched; written as fields it can be filtered, counted and grouped — the difference between 46,000 lines you cannot use and 46,000 you can use.

Require those fields on every line: UTC timestamp, level, a stable `event` name, request id, user id, route, and duration in milliseconds for anything calling another system. Levels are not degrees of enthusiasm; each is a promise about who reads the line and when.

| Level | The promise | Example |
|---|---|---|
| `debug` | Nobody reads this in production; it is off by default | Every step of a calculation |
| `info` | A thing happened that we would want to confirm later | Order placed, user signed up |
| `warn` | Something recovered, but it should not be common | Payment retried and succeeded |
| `error` | One user's request failed; somebody should look today | Order not saved |
| `fatal` | The app itself cannot continue | Cannot reach the database at startup |

Two rules make levels work. **`error` means a human should look** — spend it on routine events and your error count stops meaning anything, killing chapter seven's alerts before you write them. **`debug` must be genuinely off in production**, because leaving it on is how 46,000 lines happen.

### Where it lives

Logs are the first of chapter four's three pillars, and your platform is already collecting them whether you planned it or not.

- Chapter 6 — metrics and traces; chapter 7 — rising `error` counts becoming a buzzing phone
- The security half of this module — the same lines as an audit trail: who changed a price, who issued a refund
- M10 — the same discipline for AI calls: model, token counts, latency, a reference to the prompt

### Words to own

| Term | Plain meaning | Example |
|---|---|---|
| Structured log | A log line written as fields, usually JSON | The record in the diagram |
| Level | How urgent the line is, and who reads it | `info`, `warn`, `error` |
| Event name | A stable name for the kind of thing that happened | `order_failed` |
| Request id | An id generated per request, on every line it causes | `r-8f21c4` |
| Correlation | Tying many lines to one story | Filtering by request id |
| Redaction | Removing or masking sensitive values | `card: ••••4242` |
| PII | Personal information that identifies someone | Phone number, email, name |
| Retention | How long lines are kept before deletion | 7 days on a free tier |
| Log volume | How much you are writing, usually per day | 200 MB/day |
| Audit log | A deliberate, kept record of who did what | "staff 12 changed price of idli" |

### Tradeoffs and gotchas

- **Never log** a password (even a wrong one), a full card number or CVV, an OTP, a session token, cookie or authorization header, an API key, Aadhaar or any government id, or a full request body from a personal-data form. Log the fact and a reference: the last four digits, a user id, `otp_verification_failed`, the fields that failed validation. Under the DPDP Act and GDPR, logs are personal data like anything else, and "it is only in the logs" is not a defence. The sneakiest leak is an error handler logging the exception's full detail, query parameters included.
- **Logging everything is the same as logging nothing, and costs more.** Before adding a line, ask what question it answers.
- **Logs are the pillar that bankrupts free tiers.** Keep retention short and `debug` off. Near a quota, delete a chatty line rather than paying for storage.
- **A log line is not a metric.** Logs carry the detail of one event; metrics carry the shape of many.
- **Event names stable, timestamps in UTC.** `order_failed` here and `orderFailed` there makes every filter wrong by an amount you cannot see.

### Verify it

Ask AI to "add logging" and you get print statements. These checks find the difference.

1. **Read ten consecutive lines from production.** Could you answer "what happened to this user"? If they are sentences rather than fields, re-brief.
2. **Pick one request and filter by its id.** You should get its complete story in order. No request id means that is the first thing to add.
3. **Search your logs for the obvious secrets:** `password`, `token`, `authorization`, `cvv`, `otp`, the first six digits of a test card. **Logging a password or a full card number** is the red flag.
4. **Check `debug` is off in production and `error` means error.** Hundreds of `error` lines in a normal hour means the levels are decorative.
5. **Read the retention setting and the current volume** from the platform, and work out how many days until the quota.
6. **Break one thing on purpose and read the line.** Does it name the event, the user, the request and the reason? Does it also contain the request body?

Red flags: a log line with no timestamp or no request id; the same event spelled two ways; `error` used for expected retries; **a raw database error logged verbatim**, parameters included; a log store already at quota before your first incident.

## Walkthrough
1. Open your deployed app's logs and read the most recent twenty lines. Write a one-sentence verdict: are these records or are these sentences?
2. Count how many of those twenty lines have a timestamp, a level, and something identifying the user or the request. Write the three counts down; this is your baseline.
3. Search your logs for `password`, `token`, `authorization`, `otp` and `cvv`. Record every hit. If there are hits, this becomes the first thing you fix and, if real credentials are involved, chapter two's rotation runbook applies.
4. Filter for level `error` over the last 24 hours and read everything. Decide, for each, whether a human genuinely should have looked. The ones where the answer is no are your mislabelled levels.
5. Pick one real user action in your app — placing an order, signing up — and try to reconstruct it from the logs alone. Note every piece of information you wished was there.
6. Write your team's log contract as a short file: the required fields on every line, the meaning of each level in your product, and the never-log list. One page.
7. Brief AI to convert your existing logging to structured lines matching that contract, with a request id generated per request and attached everywhere. Make it one change with an explicit non-goal about touching anything else.
8. Verify by repeating step 5. Filter by one request id and confirm you get a complete, ordered story.
9. Find the retention setting and the monthly quota in your platform, set retention to seven days, and confirm `debug` is disabled in production.
10. Break one thing on purpose — a wrong configuration value, an apostrophe in a name — and read the resulting error line against your own contract. Fix the contract if the line taught you a field you had not thought of.
11. Commit the contract file to your repository and note in your decision log what you chose not to log, and why.

## Talk to AI
```prompt
CONTEXT:      I am a student in a team of three. I direct AI and verify its
              output; I do not write code. Our product: [describe it in two
              sentences]. Stack: [your stack] on [platform], free tier, with
              [your database]. We collect these personal fields from users:
              [list them, e.g. name, phone, email, UPI id]. We take payments
              in test mode. Our users are in India, so the DPDP Act applies
              to us as well as GDPR for any EU visitors. Right now our
              logging is whatever AI generated, mostly console output.

GOAL:         A written logging contract for my product, and a plan to move
              our existing logging onto it as one scoped change.

REQUIREMENTS: 1. Propose the exact set of fields that every log line in my
                 product should carry, with a one-line reason for each.
              2. Define each log level in terms of my product, with two
                 real examples of events from my app per level. Be strict
                 about what counts as an error.
              3. Give me the complete never-log list for my specific field
                 list above, and for each item tell me what to log instead
                 so the line is still useful.
              4. Explain what a request id is, where it has to be created,
                 and what breaks if it is only present on some lines.
              5. Tell me ten events in my product that are worth logging at
                 `info`, and five that teams commonly log and should not.
              6. Estimate my log volume per day at 50 users and at 500, and
                 tell me what retention I should set on a free tier.
              7. Tell me which of my log lines would double as an audit
                 trail for security — who changed what — and what extra
                 field those need.

CONSTRAINTS:  Free tier only. Explain every term on first use. Assume I
              verify by reading the platform's log viewer in a browser. Do
              not give me code until I have approved the contract.

NON-GOALS:    Do not add metrics, traces, dashboards or alerting — later
              chapters cover those. Do not recommend a paid log vendor. Do
              not modify any files yet.

ACCEPTANCE:   - The contract fits on one page and I can explain every field
              - The never-log list covers every personal field I listed
              - Every level has a test I could apply to a new line
              - I know what retention to set and why
              - I can tell which lines are my security audit trail

ASK:          Ask me up to five questions about my product and users that
              would change the contract. Then give me the contract only, and
              wait for my approval before planning the change.
```

## Stuck? Ask AI
```prompt
Show me the same single event — a failed order in a canteen app — written four ways: as a bad unstructured sentence, as a good unstructured sentence, as a structured record, and as a structured record that accidentally leaks personal data. Explain what is wrong with each of the three bad ones, and what filter the good one makes possible.
```

```prompt
Give me twelve events from a food-ordering app and ask me, one at a time, which log level each one deserves: debug, info, warn, error or fatal. After each answer, tell me what would go wrong downstream if I chose the level I chose — especially what it does to alerting.
```

```prompt
I have these personal fields in my app: [list them]. For each one, tell me whether it may appear in a log at all, and if not, what I should log instead so the line is still useful for debugging. Explain which of these are restricted by India's DPDP Act and which by GDPR, in plain language.
```

## Quiz

### Q1 · L3
Why is a structured log line more useful than a well-written sentence?
- (a) It takes less storage space
- (b) Fields can be filtered, counted and grouped; sentences can only be searched ✅
- (c) It is easier for a human to read top to bottom
- (d) Structured logs are required for GDPR compliance
> Once there are more lines than you can read, filtering is the only thing that works, and filtering needs fields. A sentence is readable and not queryable, which is the wrong trade at volume.

### Q2 · L4
A team logs every payment retry at `error` level. Retries are normal and succeed. What has this broken?
- (a) Nothing, as long as retries are rare
- (b) The error count no longer signals anything, so alerting on it is dead ✅
- (c) Log retention, because errors are kept longer
- (d) The ability to filter by request id
> `error` is a promise that a human should look. Spending it on expected events means the error-rate chart and every alert built on it stop distinguishing a real outage from a Tuesday.

### Q3 · L4
Of everything you could add to your log lines, which single field buys the most?
- (a) The server's hostname
- (b) A request id attached to every line from the same request ✅
- (c) The level, written in uppercase
- (d) The full request body for later inspection
> Without correlation, logs are a shuffled pile from hundreds of concurrent users. With a request id you type one value and read one person's complete story in order.

### Q4 · L4
Your error handler logs the full exception, including the failed query and its parameters. Name both problems.
- (a) It is slow, and it uses too much disk
- (b) It can leak personal data into the log store, and it signals nobody decided what belongs where ✅
- (c) It breaks the request id, and it duplicates the metric
- (d) It is fine; full detail is always better in logs
> Query parameters routinely contain phone numbers, emails and tokens, and logs are copied to third parties and read by anyone with dashboard access. Log the event, the reason and a reference — not the payload.

### Q5 · L5
Your log store hits its free-tier quota mid-incident and stops accepting lines. What is the best first response?
- (a) Upgrade to a paid plan immediately
- (b) Turn off `debug` in production and shorten retention ✅
- (c) Switch to logging only errors
- (d) Start writing logs to a file on the server instead
> The volume is nearly always `debug` left on plus long retention. Dropping to errors only would also blind you to context, and a file on one server disappears with that server.

## Go deeper
- [OpenTelemetry: logs](https://opentelemetry.io/docs/concepts/signals/logs/)
- [OWASP Logging Cheat Sheet](https://cheatsheetseries.owasp.org/cheatsheets/Logging_Cheat_Sheet.html)
