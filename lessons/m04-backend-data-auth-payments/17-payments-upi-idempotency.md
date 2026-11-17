---
id: m04-17-payments-upi-idempotency
module: m04-backend-data-auth-payments
slug: payments-upi-idempotency
title: "Payments: Gateway, UPI, Checkout, Webhooks and Idempotency"
order: 17
duration_min: 25
is_free: false
mission: true
volatility: medium
last_reviewed: 2026-11-16
---

## Key learnings
- Trace a UPI payment from tap to confirmed order and name who holds the truth
- Make a webhook handler verify, deduplicate and queue, in that order
- Prevent the duplicate-webhook double credit with an idempotency key
- Decide the amount server-side, in paise, and never from the client
- Verify a payment flow by replaying webhooks and killing the app mid-payment

## Lesson

### The pain

In test mode the canteen app takes UPI payments. The provider sends a webhook — `payment.captured`, ₹240, order 91 — and the server credits ₹240 to the student's wallet and takes eleven seconds to reply, because the handler also emails a PDF receipt. The provider waited eight seconds, treated it as failed, and retried; the server had no memory of the first and credited another ₹240. ₹480 for a ₹240 payment.

By week's end forty students had worked out that paying twice produced free credit, the canteen was ₹7,000 short, and the only record was a log of bodies with no event ids. The handler assumed a webhook arrives exactly once, and **no webhook in the world arrives exactly once.**

### The idea

**The payment provider holds the truth about money, your webhook handler must be safe to run many times, and the amount is always decided by your server.**

Your database records what the provider told you; a phone saying "payment successful" is a hint, not a fact. **Idempotency** means an operation can repeat without changing the result, so every money operation must be built so the second occurrence does nothing. The client sends what the student wants to buy, never the price.

### The analogy

A student pays at the counter and gets token 41. The bank's confirmation slip — "token 41, ₹240, cleared" — goes in the register; if the same slip arrives twice the clerk files the duplicate without writing anything, because token 41 is already recorded. That is idempotency; the register's token column is the idempotency key.

**Where the analogy breaks:** a clerk can see two slips are identical and use judgement. Your handler sees two unrelated HTTP requests, possibly in parallel on two instances. The check has to be a database constraint — a unique index on the provider's event id — because two copies of your code will both pass an "if not already recorded" test and both insert.

### Picture it

```
 student   your app    your server   provider      your worker
    │         │             │           │               │
    │─ tap ──▶│             │           │               │
    │         │─ create ───▶│           │               │
    │         │  order      │─ create ─▶│               │
    │         │             │  order,   │               │
    │         │             │  amount   │               │
    │         │             │  IN PAISE │               │
    │         │◀─ order id ─│◀──────────│               │
    │◀─ UPI app opens ──────│           │               │
    │─ approves in UPI app ─────────────▶               │
    │                                   │               │
    │◀─ "success" (a HINT, not truth) ──│               │
    │         │             │◀─ webhook─│               │
    │         │             │  signed   │               │
    │         │        ┌────┴──────────────┐            │
    │         │        │ 1. verify sig     │            │
    │         │        │ 2. event id seen? │            │
    │         │        │    yes ─▶ 200, do │            │
    │         │        │    nothing        │            │
    │         │        │ 3. record + queue │            │
    │         │        │ 4. reply 200 fast │            │
    │         │        └────┬──────────────┘            │
    │         │             │──────── job ─────────────▶│
```

Your server creates the order and tells the provider the amount in paise; the student's success screen is only a hint, and the verified, deduplicated, recorded webhook is what makes the payment real.

### Where it lives

This chapter is the whole module arriving at once: chapter 1's untrusted client, chapter 3's webhook, chapters 7 and 11's rows in one transaction, and chapter 12's public endpoint whose signature is its only authentication. Everything stays in **test mode**, where every rule is identical to production.

### Words to own

| Term | Plain meaning | Example |
|---|---|---|
| Gateway | The service that moves the money | a payment provider |
| Checkout | The flow where the user pays | UPI app handoff |
| Capture | The moment the money is actually taken | `payment.captured` |
| Webhook event id | The provider's unique id for this event | your dedup key |
| Idempotency key | A value that makes a repeat a no-op | your order id |
| Pending | Approved but not yet confirmed | normal in UPI |
| Reconciliation | Comparing your records to the provider's | a daily cron job |
| Refund | Money returned, with its own webhook | asynchronous |
| Paise | The integer unit you store money in | ₹240 is 24000 |
| Test mode | Simulated money, identical rules | where you stay |

### Tradeoffs and gotchas

**Idempotency at both ends, enforced by the schema.** Outbound, your order id is the idempotency key, so a retry returns the same payment instead of charging twice. Inbound, insert the provider's event id into a table with a **unique constraint** before doing any work; if that insert fails, reply `200` and do nothing. "Check, then insert" is two statements, and two copies of your handler both pass the check and both insert — so verify this in the schema, not the handler's `if`.

**Never make the credit an addition.** `balance = balance + 24000` run twice gives ₹480. A payment row carrying the unique event id, with the balance summed from rows, cannot double.

**Verify the signature, deduplicate, record, queue, return `200` — under a second.** Every slow thing there is a retry you invited, and the client's callback must never mark an order paid.

**UPI leaves your app, so the webhook is the primary path** — the student approves in GPay and may never come back. Amounts are integer paise, and pending is a normal state rather than an error.

### Verify it

1. **Replay the same webhook twice:** one payment row, one credit, one status change.
2. **Replay it five times, quickly** — that tests the unique constraint under concurrency, not your `if`.
3. **Send a wrong signature.** It must be rejected, changing nothing.
4. **Send a webhook claiming a different amount.** ₹1 for a ₹240 order must not be fulfilled.
5. **Tamper with the checkout amount.** Set the total to ₹1; the real price must still be charged.
6. **Kill the app mid-payment.** Approve, close the browser; the order must still become paid via the webhook.
7. **Leave a payment pending.** Use the provider's pending and failed flows; confirm an honest state and that cron resolves it.
8. **Read the schema for the unique constraint on the event id.** Not the handler — the schema.

Red flags: an order marked paid by the client's callback; crediting before verifying the signature; no unique constraint on the event id; a balance updated by addition; floats or rupees anywhere; email or PDFs inline; no reconciliation job; no pending state; a key in git; "duplicates are unlikely".

## Walkthrough
1. Read your provider's test-mode UPI documentation and find three things: how to create an order, the webhook events you will receive, and how signature verification works. Write down the signature header's name.
2. Set up test credentials with every key in an environment variable, then confirm nothing is in git by searching your history for the key prefix.
3. Design the status flow on paper as a list of allowed transitions: created, awaiting payment, pending confirmation, paid, failed, refunded. Write which event causes each move, and which transitions are forbidden.
4. Brief AI for the checkout creation, with the server computing the amount in paise from your own prices and sending your order id as the idempotency key.
5. Tamper with the request. Change the amount your client sends to ₹1 and confirm the order is still created at the real price.
6. Brief AI for the webhook handler, in this explicit order: verify the signature, insert the event id into a table with a unique constraint, record the payment, queue the slow work, return `200`. Then read the schema and screenshot the unique constraint.
7. Run a full test payment and watch it land. Confirm the order moves to paid only because of the webhook.
8. Replay the webhook from the provider's dashboard. Count the payment rows and the credits. Then replay it five times quickly and count again.
9. Send a webhook with a broken signature and confirm nothing changed. Then send one with a mismatched amount and confirm it is refused.
10. Start a payment, approve it, and close the browser before returning; confirm the order still reaches paid. Then use the provider's test flows for a pending and a failed payment and confirm each shows an honest state.
11. Write and run the reconciliation job — list the provider's payments for the day, compare against your rows, report any difference — then record in your decision log your status transitions, your idempotency strategy at both ends, and what reconciliation found.

## Talk to AI
```prompt
CONTEXT:      Three-person student team, free tiers, TEST MODE ONLY — we
              will never move real money in this project. Serverless
              functions with roughly a 10-second timeout, managed
              PostgreSQL. Users are Indian college students paying by UPI
              on budget Android phones and slow 4G; payments frequently get
              interrupted because the UPI app takes over the screen.
              All money is stored and sent as INTEGER PAISE.
              Product: [one sentence]
              What is being paid for: [describe, and where the real price
              lives in your database]
              Payment provider: [name] in test mode.
              We already have: [transactions / a background job queue /
              neither — be honest]

              OUR ORDER STATUS TRANSITIONS, as we designed them:
              [paste your list of allowed transitions and what causes each]

GOAL:         A UPI checkout and webhook flow that cannot double-credit,
              cannot be told the wrong amount, and still reaches the
              correct state when the student never returns to our app.

REQUIREMENTS: 1. Describe the full flow step by step, naming at each step
                 who holds the truth about whether money moved
              2. The server must compute the amount from my own prices in
                 paise. Tell me exactly what the client is allowed to send
              3. Specify the outbound idempotency key for creating the
                 payment and what happens when I retry with it
              4. Write the webhook handler as an explicit ordered list:
                 verify signature, deduplicate on the provider's event id,
                 record, queue slow work, return 200. Tell me the maximum
                 time the whole handler may take and why
              5. The deduplication must rely on a UNIQUE CONSTRAINT in the
                 schema, not an if-statement. Show me the constraint and
                 explain what goes wrong with the if-statement version when
                 two copies of my handler run at the same moment
              6. Tell me how the credit or fulfilment is recorded so that
                 it cannot double even if everything else fails — and why
                 adding to a balance is the wrong shape
              7. Handle the student never returning: what marks the order
                 paid, and what the student sees if they reopen the app
              8. Handle pending and failed payments, out-of-order webhook
                 delivery, and a late event for an already-final order
              9. Specify a daily reconciliation job comparing my records to
                 the provider's, and what it should report
             10. Give me a test plan I run myself in test mode: replay the
                 same webhook twice and then five times quickly, send a
                 wrong signature, send a mismatched amount, tamper with the
                 checkout amount, and close the browser mid-payment — with
                 the exact expected result for each

CONSTRAINTS:  Test mode only. Free tier only. Integer paise everywhere, no
              floats and no rupee amounts. All keys in environment
              variables. The webhook endpoint is public, so the signature
              is its only authentication. Nothing slow inside the handler.
              Marking an order paid and recording the payment must happen
              in one transaction.

NON-GOALS:    No refund flow yet. No subscriptions. No wallet top-ups
              beyond what this order needs. No admin payment dashboard. Do
              not go live. Do not add a second payment provider.

ACCEPTANCE:   - Replaying the same webhook five times produces exactly one
                payment row and one fulfilment
              - A webhook with a wrong signature changes nothing
              - A webhook claiming a different amount is refused
              - A tampered checkout amount is ignored; the real price is
                charged
              - Closing the browser mid-payment still results in a paid
                order once the webhook arrives
              - The deduplication is enforced by a schema constraint I can
                point at

ASK:          Before any code, tell me every way this flow can lose or
              duplicate money, and for each one name the single mechanism
              that prevents it. I want the failure list first.
```

## Mission

**Objective.** A working test-mode UPI payment in your own product that survives five replayed webhooks, a forged signature, a tampered amount, and a student who closes the app mid-payment — with your own evidence for each.

**Steps.**
1. **Frame** — write down what is being paid for in your product, where the real price lives, and in one sentence why the client must not be allowed to send it.
2. **Design the states** — on paper, list every order status and every allowed transition, naming the event that causes each. Mark the transitions that must be forbidden.
3. **List the failure modes** — before briefing AI, write down every way this flow could lose or duplicate money. Aim for at least six.
4. **Brief** — give AI your states, your failure list and your constraints. Demand the failure-prevention mapping before any code.
5. **Direct checkout** — build the creation flow with the server computing the amount in paise and sending your order id as the outbound idempotency key.
6. **Direct the webhook** — verify, deduplicate on the event id with a schema-level unique constraint, record, queue, return `200`. Screenshot the constraint in your schema.
7. **Run a real test payment** — in test mode, end to end, and confirm the order becomes paid because of the webhook rather than the client callback.
8. **Attack it** — replay the webhook twice, then five times quickly; send a wrong signature; send a mismatched amount; tamper with the checkout amount; close the browser mid-payment; and drive a pending and a failed payment. Record the result of every one.
9. **Count** — after the replays, count your payment rows and your credits or fulfilments. One of each, or you have the bug from the lesson.
10. **Reconcile** — write and run a job comparing your records against the provider's test-mode list, and record what it found.
11. **Secrets** — confirm every key is in an environment variable and search your git history for the key prefix.
12. **Document** — your status transitions, your idempotency strategy at both ends, your attack results, and what reconciliation found.

**Upload.**
- Your status transition design and your failure-mode list, written before AI → attached file or post caption
- The AI conversation including the failure-prevention mapping → share link in `ai_chat_url`
- Working test payment → `demo_url`, or a screen recording of the complete flow
- A screenshot of the unique constraint in your schema, not your handler code
- Attack evidence → screenshots for each of: webhook replayed five times with the resulting row count, wrong signature rejected, mismatched amount refused, tampered checkout amount ignored, browser closed mid-payment still reaching paid, and a pending and a failed payment showing honest states
- Your reconciliation job's output
- Evidence that no key is in your git history → screenshot of the search
- Reflection, 150 words max → the failure mode you had not thought of before writing the list, and one thing the AI got wrong

**Self-check.** Tick every box before you submit.
- [ ] Every amount in my system is an integer number of paise; no floats, no rupees
- [ ] The server computes the amount from my own prices, and I proved a tampered client amount is ignored
- [ ] The order becomes paid because of the webhook, never because of the client's success callback
- [ ] My webhook handler verifies the signature before reading anything else
- [ ] Deduplication is enforced by a unique constraint in my schema, and I have the screenshot
- [ ] I replayed the same webhook five times and counted exactly one payment row and one fulfilment
- [ ] A webhook with a wrong signature was rejected and changed nothing
- [ ] A webhook claiming a different amount was refused
- [ ] Marking the order paid and recording the payment happen in one transaction
- [ ] Nothing slow — email, PDF, third-party call — runs inside the webhook handler
- [ ] Closing the browser mid-payment still results in a paid order
- [ ] Pending and failed payments show an honest state to the student
- [ ] A reconciliation job exists, has been run, and I recorded its output
- [ ] Every key is in an environment variable and absent from my git history
- [ ] I never left test mode

**Stretch.** Write the refund path on paper only — the event, its own idempotency key, the status transition, and what happens if the refund webhook arrives twice. Do not implement it; hand the design to a teammate and have them find the hole.

## Stuck? Ask AI
```prompt
Tell me the story of a duplicate webhook causing a double credit, step by step, with the ₹ amounts at each stage. Then tell me the three separate things that each would have prevented it on their own, and which of the three is the most reliable and why.
```

```prompt
Explain why checking "have I already seen this event id?" with an if-statement is not enough, and why a unique constraint in the database is. Walk me through what happens when two copies of my handler run at exactly the same moment with the same webhook.
```

```prompt
My student pays by UPI, approves it in their UPI app, and then closes the browser without returning. Walk me through what my system must do so that their order still becomes paid, and tell me what they see when they reopen the app two minutes later.
```

## Quiz

### Q1 · L3
A provider waits eight seconds for your webhook response, your handler takes eleven because it generates a PDF, and the provider retries. What is the root cause of the resulting double credit?
- (a) The provider's retry policy is too aggressive
- (b) Slow work inside the handler invited a retry, and the handler was not safe to run twice ✅
- (c) The PDF generation failed and corrupted the payment record
- (d) The webhook arrived out of order
> Two defects compound. Verify, deduplicate, record, queue, return 200 in under a second avoids the retry; idempotency on the event id makes the retry harmless when it happens anyway.

### Q2 · L4
Why must webhook deduplication rest on a unique constraint rather than an "if already seen" check in the handler?
- (a) Database constraints are faster than application checks
- (b) Two copies of the handler running at once both pass the check and both insert ✅
- (c) The handler cannot read from the database before verifying the signature
- (d) Unique constraints also validate the signature
> Check-then-insert is two statements and nothing stops them interleaving. Only a database-level unique index makes the second insert fail, which is why you verify this by reading the schema, not the handler.

### Q3 · L4
A student approves a UPI payment and closes the browser before returning to your app. What must happen?
- (a) The payment is abandoned and the student must pay again
- (b) The client's success callback is retried when they reopen the app
- (c) The webhook arrives independently and promotes the order to paid ✅
- (d) The order stays pending until an admin resolves it manually
> UPI hands the screen to another app, so the client journey is unreliable by nature. The webhook is the primary path, not a backup, and this is the most common real-world journey that first implementations fail.

### Q4 · L4
Which way of recording a credit cannot double, even under duplicate delivery?
- (a) `balance = balance + 24000` inside a transaction
- (b) A payment row carrying the provider's event id under a unique constraint, with balance computed from the sum of rows ✅
- (c) `balance = balance + 24000` with a log line recording the event id
- (d) Setting the balance to the amount of the latest payment
> Addition applied twice gives ₹480. A row that physically cannot exist twice gives one credit however many times the webhook arrives, which turns correctness into a schema property rather than a code property.

### Q5 · L5
A webhook arrives with a valid signature saying ₹1 was captured for a ₹240 order. What should your handler do?
- (a) Credit ₹1 and mark the order partially paid
- (b) Credit ₹240, since the order's price is the authority
- (c) Refuse to fulfil, record the mismatch, and reply 200 so the provider stops retrying ✅
- (d) Reject with a 400 so the provider retries until the amount matches
> A valid signature proves who sent it, not that it matches your order. Fulfilling on a mismatched amount is a loss; retrying forever is noise. Record it, do not fulfil, and have a human look at it.

## Go deeper
- [Razorpay docs: payment gateway, webhooks and test mode](https://razorpay.com/docs/payments/)
- [Stripe docs: idempotent requests](https://docs.stripe.com/api/idempotent_requests)
