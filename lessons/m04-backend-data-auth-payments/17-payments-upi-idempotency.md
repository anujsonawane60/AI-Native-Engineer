---
id: m04-17-payments-upi-idempotency
module: m04-backend-data-auth-payments
slug: payments-upi-idempotency
title: "Payments: Gateway, UPI, Checkout, Webhooks and Idempotency"
order: 17
duration_min: 35
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

The canteen app takes UPI payments in test mode. A student pays ₹240 for her order, the flow works, everyone is delighted. Then the real story, the one this chapter exists for.

The provider sends a webhook to the canteen's server: `payment.captured`, ₹240, order 91. The server credits ₹240 to the student's canteen wallet, marks the order paid, and then — because the handler also generates a PDF receipt and emails it — takes eleven seconds to reply.

The provider waited eight seconds, got nothing, and treated the delivery as failed. So it retries. The same webhook arrives again. The server, with no memory of the first, credits another ₹240. The student now has ₹480 of credit for a ₹240 payment.

By the end of the week forty students have discovered that paying slowly, or twice, or just getting lucky with a flaky network, produces free credit. The canteen is short ₹7,000 and the team cannot reconstruct what happened, because the only record is an unbounded log of webhook bodies with no event ids.

The bug is not in the payment. The payment was perfect. The bug is that the handler assumed a webhook arrives exactly once, and **no webhook in the world arrives exactly once.**

### The idea

**The payment provider holds the truth about money, your webhook handler must be safe to run many times, and the amount is always decided by your server.**

Three rules, each preventing a specific loss.

**The provider is the source of truth.** Your database records what the provider told you. A phone saying "payment successful" is a hint, not a fact — it can lie, the network can drop the confirmation, and the user can close the app at the wrong moment.

**Idempotency** means an operation can repeat without changing the result. Webhooks arrive twice, users double-tap, networks retry. Every money operation must be designed so the second occurrence does nothing.

**The server computes the amount.** Chapter 1's rule at the highest stakes. The client sends what the student wants to buy; the server looks up today's prices, totals them in paise, and tells the provider what to charge.

### The analogy

The canteen's token system, with the bank involved.

A student pays at the counter and the cashier gives her token 41. The kitchen cooks against the token, not against her word. If she says "I paid" with no token, the answer is no.

The bank sends a confirmation slip to the office: "token 41, ₹240, cleared". The clerk writes it in the register. If the same slip arrives twice — the courier was unsure the first was delivered — the clerk sees token 41 already recorded and files the duplicate without writing anything. That check is idempotency, and the register's token column is the idempotency key.

And the price: the student does not write ₹2 on her own slip. The cashier reads today's board.

**Where the analogy breaks:** a clerk can see two slips are identical and use judgement. Your handler sees two unrelated HTTP requests, possibly in parallel on two server instances, with no intuition at all. The check has to be a database constraint — a unique index on the provider's event id — because two copies of your code running simultaneously will both pass an "if not already recorded" test and both insert.

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

This chapter is the whole module arriving at once, which is why it is last.

The amount comes from chapter 1's rule that the client cannot be trusted. The webhook is chapter 3, with signature verification now protecting money. The order and payment rows are chapters 6 and 7. Marking the order paid and recording the payment together is chapter 11's transaction. The webhook endpoint is public, so chapter 12's lesson — a route with no session needs its own authentication, here the signature — applies directly. And verify, record, queue, return is chapter 16.

Forward: M07 treats a payment discrepancy as an incident with a response procedure. M11's checkpoints will ask you to demonstrate a payment flow and explain its failure cases. M17 is the business end — pricing, revenue, reconciliation as a monthly discipline.

Everything here stays in **test mode**: test credentials, test UPI handles, simulated webhook deliveries. You will never move real money in this course, and every rule is identical in test mode and production — which is exactly why you learn it here.

### UPI specifically

UPI is the default in India and behaves differently from a card in ways that matter to your design.

**The payment leaves your app.** The student is handed to their UPI app — GPay, PhonePe, Paytm — approves there, and comes back. Your app loses the screen mid-transaction, and they may not come back at all: they close the app, lose signal, or get a call. **The payment can succeed while your app never hears about it from the client.** That is why the webhook is the primary path rather than an optimisation.

**Amounts are integer paise.** ₹240 is `24000`. Providers' APIs take paise; your database stores paise. A float anywhere produces a reconciliation you cannot explain, and the amounts do not round in your favour.

**Pending is a real, common state, not an error.** A UPI payment can sit pending for minutes. Your order needs a status for it, your UI must say something honest, and chapter 16's cron job must ask the provider about anything pending after ten minutes.

**Refunds are asynchronous too** — their own request, webhook and idempotency problem. So are collect requests and static QR codes: same three rules, different journey.

### Idempotency, concretely

Two places need it, for two different reasons.

**Outbound: creating the payment.** You send an idempotency key — usually your own order id. If your network drops and you retry, the provider sees the same key and returns the same payment rather than creating a second. Without it, a student on a flaky connection who taps twice is charged twice.

**Inbound: receiving the webhook.** The provider's event has a unique id. You insert it into a table with a **unique constraint** before doing any work. If the insert fails because the id exists, this is a duplicate: reply `200` and do nothing else.

The unique constraint is load-bearing. "Check whether we have seen this id, then insert it" is two statements, and two copies of your handler running at the same moment both pass the check and both insert. A database-level unique index is the only thing that makes the second fail — which is why you verify this by reading the schema, not the handler's `if`.

And one more: **never make the credit an addition.** `balance = balance + 24000` run twice gives ₹480. A payment row carrying a unique event id, with the balance computed from the sum of rows, cannot double because the second row cannot exist.

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

**Never trust the client's success callback alone.** Mark the order `pending_confirmation` and let the webhook promote it. Otherwise a student who edits the callback gets free food, and a student whose connection drops after a real payment gets nothing.

**Reply to the webhook fast, then work.** Verify, deduplicate, record, queue, return `200` — under a second. Every slow thing in that handler is a retry you invited, and a duplicate you must survive.

**Verify the signature first.** Until it checks out, the body is a string from a stranger.

**Store the provider's raw event, once, with its id.** When you are ₹7,000 short, that is what lets you reconstruct the week. A text log with no ids is what the team in the opening story had.

**Webhooks arrive out of order.** `payment.captured` can arrive before `order.created` commits, and a failure after a success. Your transitions must be explicit about what may follow what, and a late event for an already-final order is ignored, not applied.

**Reconcile on a schedule.** A daily job comparing your payment rows against the provider's list finds the webhooks you never received — and you will have some.

**Test mode and production differ in configuration, not in rules.** Separate keys, secrets and dashboards, all in environment variables. A payment key in git is a rotate-today incident even in test mode.

### Verify it

Eight checks, and payments is the one place where all of them are mandatory.

1. **Replay the same webhook twice,** using the provider's resend button. Count: one payment row, one credit, one status change. Two of anything is the opening story, live in your project.
2. **Replay it five times, quickly.** A different test — whether the unique constraint holds under concurrency, rather than whether your `if` works.
3. **Send a wrong signature.** It must be rejected and change nothing, or anyone on the internet can mark any order paid.
4. **Send a webhook claiming a different amount.** ₹1 for a ₹240 order. Your handler must notice and refuse to fulfil.
5. **Tamper with the amount on the way in.** Change your checkout request's total to ₹1. The order must still be created at the real price.
6. **Kill the app mid-payment.** Approve a test payment, then close the browser. Does the order become paid when the webhook arrives? This is the commonest real UPI journey and most first implementations fail it.
7. **Leave a payment pending.** Use the provider's pending and failed test flows. Confirm an honest state and that your cron job resolves it.
8. **Read the schema for the unique constraint on the event id.** Not the handler — the schema.

Red flags: an order marked paid by the client's callback; crediting before verifying the signature; no unique constraint on the event id; a balance updated by addition; floats or rupees anywhere; email or PDFs generated inline; no reconciliation job; no pending state; a key in git; "duplicates are unlikely".

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
