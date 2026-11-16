---
id: m07-05-structured-logs
module: m07-security-observability-analytics
slug: structured-logs
title: "Structured Logs, Levels and What Is Worth Logging"
order: 5
duration_min: 25
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

A team's app starts failing at checkout. They open the logs, which is the right instinct, and find 46,000 lines from the last two hours. Most of them say `here`, `here 2`, `got to the function`, and `data: [object Object]`. Several hundred print an entire order object. Four print something that looks like a password.

They search for "error" and get 900 results, because an earlier debugging session logged every retry as an error. They cannot tell which of the 900 belong to the student who complained, because no line says which request it came from. They cannot tell which came first, because a third of the lines have no timestamp.

Twenty minutes later the log store hits its free-tier quota for the month and stops accepting new lines. The app is still broken. Now it is broken in the dark.

### The idea

**A log line is a data record, not a sentence. Write it as fields so a machine can filter it, because you will have too many lines to read.**

The instinct is to write for a human reading top to bottom. That works until you have a thousand lines. After that, the only thing that works is filtering — show me every error, for this one user, on this one endpoint, in this one hour — and filtering requires fields.

### The analogy

Back to the canteen's daily sales register. The useful register has columns: time, item, quantity, price, who served it, payment mode. At the end of the month you can answer "how many samosas before noon" by running your finger down one column.

The useless register is a diary. "Busy morning, sold a lot of samosas, Ravi came in late, the idli ran out around eleven, somebody paid by UPI and it did not go through." Every fact is in there. None of it can be counted, sorted or compared, and in a month there are forty pages of it.

Same information. One is data; one is prose. Structured logging is insisting on columns.

**Where the analogy breaks:** a register is kept forever and costs nothing but shelf space. Logs cost money per gigabyte, every day, and on a free tier they stop working when the quota runs out. A canteen never has to decide to throw away March; you do, deliberately, and you set that retention on day one.

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

The same event written as a sentence can only be grepped; written as fields it can be filtered, counted and grouped — which is the entire difference between 46,000 lines you cannot use and 46,000 lines you can.

### The levels, and what each one promises

Levels are not degrees of enthusiasm. Each one is a promise about who reads it and when.

| Level | The promise | Example |
|---|---|---|
| `debug` | Nobody reads this in production; it is off by default | Every step of a calculation |
| `info` | A thing happened that we would want to confirm later | Order placed, user signed up |
| `warn` | Something recovered, but it should not be common | Payment retried and succeeded |
| `error` | One user's request failed; somebody should look today | Order not saved |
| `fatal` | The app itself cannot continue | Cannot reach the database at startup |

Two rules make levels work. **First, `error` means a human should look.** The moment you log expected, routine events as errors, your error count stops meaning anything and chapter seven's alerts become noise. **Second, `debug` must be genuinely off in production.** Not "we leave it on, it is useful" — that is how 46,000 lines happen and how quotas die.

A practical test: if your error rate chart would not change shape when a real outage started, your levels are wrong.

### The field that matters most

Of every field you could add, one is worth more than the rest put together: **a request id, generated when a request arrives and attached to every log line it produces.**

Without it, logs are a shuffled pile of sentences from hundreds of concurrent users. With it, you type one id and get that one student's complete story in order: request arrived, user looked up, payment called, payment slow, database timed out, error returned. That is the difference between debugging and guessing, and it is the thing to insist on in the brief because AI will not add it unasked.

Add to every line, as fields: timestamp in UTC, level, a stable `event` name, request id, user id where there is one, route, and duration in milliseconds for anything that calls out to another system.

### What must never reach a log

This list is not advice. Under India's DPDP Act and under GDPR, logs are personal data like anything else, and "we only put it in the logs" is not a defence — logs are copied to third-party services, read by whoever has dashboard access, and kept for weeks.

Never log: passwords, even wrong ones. Full card numbers or CVVs. OTPs. Session tokens, cookies or authorization headers. API keys. Aadhaar or other government id numbers. Full request bodies from any form that collects personal data.

Log instead the *fact* and a *reference*: not the card number but its last four digits, not the phone number but a user id, not the OTP but `otp_verification_failed`, not the whole body but the field names that failed validation. If you genuinely need an identifier, log the internal id and look the person up in your own database — which also means the log store never holds the mapping.

The sneakiest leak is not deliberate. It is a tired line that prints the whole request, or an error handler that logs the exception's full detail including the query and its parameters. Which brings us to the one from chapter one: **returning the raw database error** to the user is a security flaw, and logging it verbatim can quietly put personal data into your log store at the same time.

### Where it lives

Logs are the first of the three pillars from chapter four — the sales register. They run on the server, and on your hosting platform they are already being collected whether you planned it or not, which is where M06's deploy chapter left you. Chapter six adds metrics and traces, and chapter seven is what turns a rising count of `error` lines into somebody's phone buzzing. The security half of this module uses the same lines as its audit trail: who changed a price, who issued a refund, who logged in from where.

In M10 the same discipline applies to AI calls, where you log the model, the token counts, the latency and a reference to the prompt — never the user's raw text, for exactly the reasons above.

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

**Logging everything is the same as logging nothing, and costs more.** Signal goes down as volume goes up, because you stop being able to find anything. Before adding a line, ask what question it answers. If there is no question, there is no line.

**Logs are the pillar that bankrupts free tiers.** They are text, one line per event, and they add up fast. Set retention short — a week is plenty for a student project — and keep `debug` off. If you are near a quota, the cheapest fix is almost always deleting a chatty line rather than paying for storage.

**A log line is not a metric.** Counting `error` lines to get an error rate works until volume grows, then it is slow and expensive. Chapter six's counters exist for this. Use logs for the detail of one event, metrics for the shape of many.

**Event names must be stable.** If the same event is logged as `order_failed` in one place and `orderFailed` or `failed_order` elsewhere, every filter you write is wrong by an amount you cannot see. Pick one spelling convention and write it down, exactly as chapter eight will ask you to do for analytics events.

**Timestamps in UTC, always.** Your users are in IST, your server may not be, and your log store may display a third zone. One format, one zone, converted when displayed. Comparing a 21:14 to an 15:44 and concluding nothing happened is a genuinely common wasted hour.

**Do not log inside a loop over user data.** One line per order is fine. One line per item in every order means your log volume scales with your biggest customer, which is the classic way a working setup dies at 10x.

### Verify it

Ask AI to "add logging" and you get `console.log` statements. Six checks that find the difference between that and something usable.

1. **Read ten consecutive lines from production.** Could you answer "what happened to this user" from them? If they are sentences rather than fields, stop and re-brief.
2. **Pick one request and filter by its id.** You should get its complete story in order. If there is no request id, that is the single most valuable thing to add, before anything else.
3. **Grep your own logs for the obvious secrets.** Search for `password`, `token`, `authorization`, `cvv`, `otp`, and the first six digits of a test card. **Logging a password or a full card number** is the red flag here, and the search takes a minute.
4. **Check that `debug` is off in production and `error` means error.** Count your `error` lines for a normal hour. If there are hundreds and nothing is wrong, the levels are decorative and chapter seven's alerts are already dead.
5. **Read the retention setting and the current volume.** Both numbers, from the platform, not from AI. Work out how many days from now you hit the quota.
6. **Deliberately break one thing and read the resulting line.** Does it name the event, the user, the request and the reason? Does it also, accidentally, contain the whole request body?

Red flags to say out loud: a log line with no timestamp or no request id; the same event spelled two ways; an `error` level used for expected retries; the raw database error logged verbatim, parameters included; and a log store already at its quota before your first real incident.

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
