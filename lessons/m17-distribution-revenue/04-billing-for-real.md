---
id: m17-04-billing-for-real
module: m17-distribution-revenue
slug: billing-for-real
title: "Billing for Real: Plans, Upgrades, Dunning and Invoices"
order: 4
duration_min: 35
is_free: false
mission: true
volatility: medium
last_reviewed: 2026-11-16
---

## Key learnings
- Draw the subscription state machine, including past due, grace, suspended and cancelled
- Derive every feature a user can reach from one source of truth about their plan
- Build a dunning sequence so a failed debit does not silently become churn
- Handle a UPI autopay mandate's amount limit, notification and failure cases
- Catch a billing design with no invoice, no dunning path, or two sources of truth

## Lesson

### The pain

A team ships a ₹199 monthly plan with UPI autopay in test mode. Mandate created, first debit succeeds, everyone celebrates. Then month two arrives and about thirty per cent of debits fail: insufficient balance on the 1st because pocket money lands later, mandates created with a maximum amount of exactly ₹199 so adding GST pushed the charge over the limit, and some ordinary bank downtime.

None of that is the disaster. The disaster is what the system did about it: nothing. No retry, no email, no banner. The subscription row quietly moved to `expired`, premium features disappeared mid-week with no explanation, and the user concluded the product was broken and uninstalled it. The dashboard recorded thirty per cent churn and the team spent a fortnight redesigning onboarding.

Separately, a user who upgraded from ₹199 to ₹499 mid-month was charged both amounts in full, asked for an invoice, and was told "we don't have invoices yet."

Every one of these failures is in the billing layer, not the payment layer. The payments from M04 chapter 17 worked perfectly throughout.

### The idea

**A subscription is a state machine with money attached, and the states that decide your revenue are the unhappy ones.**

`active` is easy. What separates a billing system from a payment integration is `trialing`, `past_due`, `suspended`, `cancel_at_period_end`, `cancelled` and `reactivated` — and the rule that **exactly one place in your system decides what a user can do, derived from that state.**

M04 chapter 17 gave you the mechanics: the gateway holds the truth about money, webhooks are the primary path, every money operation is idempotent, amounts are integer paise. All of that still holds and none of it is repeated here. This chapter is the commercial layer on top: what the user bought, what they may use, what happens when the money does not arrive, and what piece of paper they get.

### The analogy

The canteen's monthly mess pass.

A student pays ₹1,800 on the 1st, the clerk stamps the register, and she eats. On the 5th, another student has not paid. The clerk does not throw her out; he reminds her at the counter, and again on the 8th. On the 12th the pass is marked unpaid and she pays per plate until she settles — but the register still has her name, her history and her seat, so when she pays on the 15th she is back in one stroke of a pen. And when her father asks for proof, he writes a receipt.

That is a billing system: a plan, a register, a reminder sequence, a grace period, a reversible suspension, and a receipt.

**Where the analogy breaks:** the clerk recognises the student and uses judgement. Your system has none. The grace period must be a number in a config file, the reminders must be scheduled jobs that run whether or not anyone is awake, and the suspension must be a state transition an automated check enforces. The one thing to copy exactly: he never lets an unpaid pass keep working silently, and he never destroys the register.

### Picture it

```
  SUBSCRIPTION STATE MACHINE

  trialing ──── trial ends, debit ok ────▶ active
     └───────── trial ends, debit fails ─▶ past_due

  active ────── user cancels ────────────▶ cancel_at_period_end
     └───────── debit fails on renewal ──▶ past_due

  past_due ──── a retry succeeds ────────▶ active
     └───────── retries exhausted ───────▶ suspended

  cancel_at_period_end ── period ends ───▶ suspended

  suspended ─── user pays ───────────────▶ active
     └───────── 30 days unpaid ──────────▶ cancelled

  cancelled ─── history kept, nothing deleted
```

Seven states, and every unhappy path has a named destination rather than silently disappearing.

### Entitlements: one source of truth

The bug that costs the most support time is having two answers to "is this user paid".

One place must answer it, and everything else asks that place. The shape that works: the subscription row carries a plan id and a state, and a single function takes a user and a feature name and returns yes or no. The feature code never reads the payments table, never reads a boolean somebody set during a webhook, and never trusts the client.

Two sources of truth produce both classic bugs at once: a cancelled user who keeps premium because an `is_premium` flag was never cleared, and a payer locked out because a different code path cleared it. And **entitlement checks live on the server, always** — hiding a button is a courtesy, not a control. M04 chapter 12's authentication-versus-authorization lesson is this same lesson wearing a price tag.

### UPI autopay and mandates: what is different

Recurring payment in India usually means a UPI autopay mandate, which behaves unlike a stored card in four ways. Gateway features change often, so **read your gateway's current recurring-payments documentation rather than this list or an AI's memory of it.**

**A mandate has a maximum amount.** The user approves "up to ₹X per month", and if your charge ever exceeds ₹X — a price rise, GST, an overage from chapter 3 — the debit fails. Create the mandate with headroom and write down why.

**The user is notified before the debit.** A pre-debit notification goes out ahead of the charge, so the user has a window in which to be surprised and cancel. Your own renewal message must not contradict it.

**Mandates expire and can be revoked in the user's own UPI app**, without touching your product. You learn about it from a webhook, not from the user, so handle the revocation event explicitly or you will show someone as active forever.

**Debit failures are routine, not exceptional.** Insufficient balance on the 1st is the commonest in a student market, so design the retry schedule around when money actually arrives.

### Dunning: the sequence that recovers money

**Dunning** is the structured sequence of retries and reminders after a failed payment. Involuntary churn — people who wanted to stay and lost access to a failed debit — is usually a large slice of total churn, and the cheapest to fix, because the user has already chosen your product.

```
  DUNNING SEQUENCE, ₹199 PLAN

  day  state       what happens                  user sees
  --------------------------------------------------------------
   0   active      debit fails                   nothing yet
   0   past_due    in-app banner + email         "payment failed,
                                                  retry on day 3"
   3   past_due    retry 1                       email if it fails
   7   past_due    retry 2                       email + banner
  10   past_due    final notice                  "access pauses
                                                  on day 12"
  12   suspended   features off, data kept       page explaining
                                                  how to fix it
  42   cancelled   subscription closed           final email
```

A ten-day sequence of retries and warnings before anything is switched off, with the user told at every step what happens next and when.

Three design rules inside it: retry when money is likely to be there rather than on a fixed interval that keeps hitting the same empty account; never switch anything off without having given a date in advance; and make recovery one tap.

### Invoices, GST and upgrades

**Invoices are not optional for anyone who might be reimbursed** — a student claiming from a parent, a society claiming from a college, any business buyer. An invoice needs your legal name and address, the buyer's name, a number in an unbroken sequence, the date, line items, the amount, the tax treatment and a payment reference. A GST-registered buyer wants your GSTIN for input credit; if you have none, say so rather than leaving a blank.

**Decide now whether your displayed price includes tax.** "₹199" and "₹199 + GST" are different products to a buyer, and changing your mind later means every mandate has the wrong maximum amount.

**Upgrades need a proration rule you can explain in one sentence**, and the simplest is to credit the unused portion of the old plan against the new. Do not invent the arithmetic; let the gateway do it and verify what it actually did. What you must never do is charge both plans in full. **Downgrades and cancellations take effect at the end of the paid period**, so the user keeps what they paid for.

### Where it lives

Directly on top of M04 chapter 17: the gateway is the source of truth, webhooks drive state changes, handlers are idempotent against the provider's event id, amounts are integer paise, and you stay in test mode.

Chapter 3 chose the structure this implements. Chapter 5 reads MRR and churn out of these tables, chapter 6 is why plans need allowances, chapter 9 is what happens when billing breaks on a Sunday, and chapter 11 reviews the numbers weekly. M07's structured logs answer "why did this user lose access"; M12's store rules override some of this for in-app purchases.

### Words to own

| Term | Plain meaning | Example |
|---|---|---|
| Plan | A named bundle with a price and limits | ₹199, 100 orders |
| Entitlement | What a plan actually unlocks | Access to exports |
| Mandate | The user's standing approval to debit | Autopay, up to ₹X |
| Pre-debit notification | The warning sent before a charge | Required for autopay |
| Past due | Payment failed, access not yet removed | Day 0 to day 12 |
| Dunning | The retry and reminder sequence | The table above |
| Involuntary churn | Lost to a failed debit, not a decision | The cheapest to fix |
| Suspended | Access off, data and history kept | Reversible in one tap |
| Proration | Adjusting for a mid-period change | Credit the unused days |
| Invoice | The document a buyer can claim with | Numbered, sequential |

### Tradeoffs and gotchas

**Do not build your own subscription engine.** Use the gateway's subscriptions, plans and mandates. Your job is the state machine *in your own database* that mirrors it, the entitlement function, and the dunning communication — the gateway will not write your emails or know what your features are.

**Never delete data on cancellation.** Suspend, keep, and tell the user how long you keep it. Reactivation in one tap is worth real money; chapter 10 covers the erasure request, which is a different thing.

**Test the unhappy paths or you have tested nothing.** Every gateway offers test-mode failure simulation, and a billing system verified only on the success path is the opening story.

**Refunds need a written policy before your first refund request**, which most gateways require anyway. Decide the rule, publish it, apply it the same way every time.

### Verify it

**Red flag, the headline one: a subscription with no dunning path.** If the design goes from "payment failed" straight to "no access", you are converting recoverable failures into permanent churn. Ask explicitly: what happens on day 0, day 3, day 7 and day 12, and what does the user see at each?

**Red flag: two sources of truth for whether a user is paid.** Search the code for every place that decides access. If a boolean and a subscription state can disagree, they eventually will.

**Red flag: an entitlement check only in the UI.** Call the paid endpoint directly as a free user; if it works, the hidden button changed nothing.

**Red flag: proration arithmetic invented by the AI.** Read the gateway's own record of what it actually charged and compare. Do not accept a formula; accept a reconciliation.

**Red flag: a mandate created with a maximum amount equal to today's price.** Ask what happens when you add GST or raise the price by ₹50. If the answer is "the debit fails", fix it now while you have four subscribers.

**Red flag: cancellation that deletes data immediately, or an invoice with a non-sequential number.** The first destroys the easiest revenue you will ever recover; the second gets asked about.

**Red flag: no record of why access was revoked.** When a user says "I was paying and you cut me off", you need a logged, timestamped, caused state transition. M07's structured logs are where it lives.

## Walkthrough
1. Write your plan catalogue on paper: each plan's name, price in paise, billing period, what is included, and the allowance and cap from chapter 3.
2. Draw your subscription state machine with every state and every transition, naming the event that causes each one. Mark which transitions are forbidden.
3. Write the entitlement function's signature in one line — user plus feature name returns yes or no — and list every feature it governs. One list, one function.
4. Read your gateway's current recurring-payments and subscriptions documentation. Write down three things: how a mandate's maximum amount is set, which webhook events signal a failed debit and a revoked mandate, and how it prorates an upgrade.
5. Decide and write down four numbers: the mandate headroom above your price, the retry days, the suspension day, and the cancellation day.
6. Brief AI for the plan, subscription and entitlement layer on top of your existing M04 payment handling, giving it your state machine and insisting the entitlement check is server-side with one source of truth.
7. Create a test-mode subscription, confirm the mandate exists with the headroom you chose, and screenshot the mandate's maximum amount.
8. Force a failed debit using your gateway's test-mode failure simulation. Confirm the state becomes past due, the banner and email appear, and nothing is switched off.
9. Run the clock forward through your dunning schedule. Confirm suspension happens on the day you chose, the user sees an explanation, and the data still exists.
10. Pay from the suspended state and confirm reactivation is one tap and restores exactly the right entitlements. Then call one paid endpoint directly as a free user and confirm it is refused.
11. Perform an upgrade, read what the gateway actually charged, compare it against your own proration expectation, and generate an invoice with a sequential number and the right tax treatment.

## Talk to AI
```prompt
CONTEXT:      Three-person student team, free tiers, TEST MODE ONLY - we will
              never move real money. We already have working payments from an
              earlier module: the gateway is the source of truth, webhooks are
              verified and deduplicated on the provider's event id with a
              database unique constraint, and all amounts are integer paise.
              Do not redesign any of that.
              Users are Indian college students paying by UPI autopay on
              budget Android phones. Money often arrives in their accounts
              after the 1st of the month.
              Product: [one sentence]
              Gateway: [name] in test mode.
              My plan catalogue: [name, price in paise, period, what is
              included, allowance and cap for each plan]
              My displayed price includes tax: [yes/no]

              MY SUBSCRIPTION STATE MACHINE, as I drew it:
              [paste every state and every transition with its cause]

GOAL:         A subscription and entitlement layer where a failed debit never
              silently becomes churn, access is decided in exactly one place
              on the server, and every buyer can get an invoice.

REQUIREMENTS: 1. Review my state machine. Name every state and transition I
                 am missing, especially around trials, revoked mandates,
                 cancellation at period end and reactivation
              2. Specify the single entitlement function - its inputs, its
                 output, and where it reads from. Then tell me every place in
                 a typical app that is tempted to decide access some other
                 way, so I can forbid each one
              3. Tell me what to do about UPI autopay mandate maximum amount,
                 and how much headroom to leave above my current price and why
              4. Point me at the exact current documentation pages for my
                 gateway covering mandates, failed debits, revoked mandates
                 and upgrade proration. Do NOT quote limits, fees or feature
                 behaviour from memory - tell me what to read and what to
                 look for
              5. Design my dunning sequence as a day-by-day table: day,
                 subscription state, what my system does, and the exact words
                 the user sees. Tune the retry days for users whose money
                 arrives after the 1st
              6. Specify the suspension behaviour: what is switched off, what
                 is kept, what the suspended user sees, and how reactivation
                 works in one tap
              7. Tell me how to handle an upgrade and a downgrade, including
                 what I must verify against the gateway's own record rather
                 than compute myself
              8. Specify the invoice: every field it needs, how the numbering
                 stays sequential, how tax is shown, and what changes if my
                 buyer is GST-registered and wants input credit
              9. Give me a test plan I run myself in test mode covering a
                 failed debit, exhausted retries, suspension, reactivation, a
                 revoked mandate, an upgrade and a cancellation - with the
                 exact expected result for each

CONSTRAINTS:  Test mode only. Integer paise everywhere. Do not build a
              subscription engine - use the gateway's subscriptions and
              mandates and mirror the state in my own database. Entitlement
              checks are server-side only. Never delete data on cancellation.
              No slow work inside any webhook handler.

NON-GOALS:    Do not redesign my payment or webhook handling. No marketplace
              payouts. No multi-currency. No usage-based overage billing yet -
              allowances only. Do not go live.

ACCEPTANCE:   - A failed debit produces past_due, a banner, an email, and no
                loss of access
              - Nothing is switched off without a prior warning naming a date
              - A suspended user who pays is restored in one tap with the
                correct entitlements
              - Calling a paid endpoint directly as a free user is refused
              - Exactly one place in my system answers "is this user paid"
              - An upgrade's charge matches the gateway's own record
              - An invoice exists with a sequential number and correct tax
                treatment

ASK:          Before any code, list every way this design can either give
              away paid features or take access from someone who is paying.
              For each one, name the single mechanism that prevents it. I want
              that failure list first.
```

## Mission

**Objective.** A working test-mode subscription in your own product that survives a failed debit, a full dunning sequence, a suspension, a one-tap reactivation, an upgrade and a cancellation — with one server-side source of truth for access and a real invoice at the end.

**Steps.**
1. **Frame** — write your plan catalogue: each plan's name, price in paise, period, inclusions, allowance and cap.
2. **Design the states** — draw the state machine with every transition and its cause, and mark the forbidden transitions. Do this before briefing AI.
3. **List the failure modes** — write down every way your design could give paid features to a non-payer or take access from a payer. Aim for at least six.
4. **Decide the four numbers** — mandate headroom, retry days, suspension day, cancellation day. Write why each one is what it is for your users.
5. **Read the gateway's own docs** — mandates, failed debits, revoked mandates and proration. Record the page and the date. No gateway behaviour in this mission may come from an AI's memory.
6. **Brief** — give AI your state machine, your four numbers and your failure list, and demand the failure-prevention mapping before any code.
7. **Build the entitlement function** — one function, server-side, reading one source of truth. List every feature it governs.
8. **Attack it** — force a failed debit, exhaust the retries, reach suspension, reactivate in one tap, revoke the mandate from the payer's side, upgrade, downgrade and cancel. Record the result of every one.
9. **Prove the single source of truth** — call a paid endpoint directly as a free user, and as a suspended user, and record the refusals. Then search your own code for any second place that decides access.
10. **Invoice it** — generate an invoice with a sequential number, the correct tax treatment, and the fields a buyer claiming reimbursement would need.
11. **Reconcile** — compare your subscription states against the gateway's own list for every test subscriber and report any difference.
12. **Document** — your state machine, your four numbers, your attack results, and your refund and cancellation policy in writing.

**Upload.**
- Your plan catalogue and state machine, drawn before AI → attached file
- Your failure-mode list, written before AI → file or caption
- The gateway documentation pages you read, with the date → screenshots
- The AI conversation including the failure-prevention mapping → share link in `ai_chat_url`
- Working test-mode subscription → `demo_url` or a screen recording
- A screenshot of the mandate showing its maximum amount and your headroom
- Attack evidence → screenshots for each of: failed debit producing past due with no loss of access, the dunning emails or banners, suspension with data intact, one-tap reactivation, a revoked mandate handled, an upgrade with the gateway's own charge record beside your expectation, and a cancellation taking effect at period end
- Proof of refusal when calling a paid endpoint as a free user and as a suspended user → screenshots
- Your generated invoice → PDF or screenshot
- Your reconciliation output → text
- Your written refund and cancellation policy → file in your repo
- Reflection, 150 words max → the state you had not thought of before drawing the machine, and one thing the AI got wrong about your gateway

**Self-check.** Tick every box before you submit.
- [ ] My state machine includes trialing, past due, suspended, cancel-at-period-end, cancelled and reactivated
- [ ] Exactly one function in my system answers "can this user use this feature"
- [ ] That function runs on the server, and I proved a direct call from a free user is refused
- [ ] I searched my own code and there is no second source of truth for paid status
- [ ] A failed debit produces past due and changes no entitlements
- [ ] Nothing is ever switched off without a prior message naming a date
- [ ] My retry days are chosen for when my users actually have money, and I wrote down why
- [ ] My mandate has headroom above my current price, and I have the screenshot
- [ ] A revoked mandate is handled by a webhook, not discovered by a user complaint
- [ ] Suspension keeps all data and reactivation takes one tap
- [ ] My upgrade charge matches the gateway's own record, which I read rather than computed
- [ ] My invoice has a sequential number and states the tax treatment
- [ ] No gateway limit, fee or behaviour in my submission came from an AI
- [ ] A refund and cancellation policy is written and committed
- [ ] Every amount is integer paise, and I never left test mode

**Stretch.** Write the plan-migration memo on paper: you are raising the price by ₹50 and every existing mandate has a maximum amount that is now too low. Describe exactly what each existing subscriber experiences, in what order, and how many you expect to lose. Hand it to another team and have them find the step you forgot.

## Stuck? Ask AI
```prompt
Explain the difference between voluntary and involuntary churn with ₹ amounts in a worked example of 100 subscribers at ₹199. Then tell me, for each kind, what mechanism reduces it, and which kind is cheaper to fix and why.
```

```prompt
I have a boolean called is_premium on my user row and I also have a subscription table with a state. Explain, with a step-by-step story, the two bugs this causes: a cancelled user who keeps access, and a paying user who loses it. Then tell me what to do instead, and what I should search my own code for.
```

```prompt
Walk me through everything that is different about UPI autopay mandates compared with a stored card, as questions I must answer about my own design rather than facts about the gateway. Then tell me which of my gateway's documentation pages I need to read, and warn me that any limit or behaviour you remember may be out of date.
```

## Quiz

### Q1 · L3
A debit fails on renewal and your system immediately moves the subscription to expired and removes access. What is the main cost of this design?
- (a) The gateway will stop sending webhooks for that subscriber
- (b) Recoverable payment failures become permanent churn from users who wanted to stay ✅
- (c) The invoice sequence breaks
- (d) The mandate is revoked automatically
> Involuntary churn is the cheapest churn to fix, because the user has already chosen your product. A dunning sequence with retries and warnings recovers a large share of it.

### Q2 · L4
Your subscription state lives in a table and you also set `is_premium` on the user row from a webhook. Why is this a defect even if it currently works?
- (a) Booleans cannot represent trials
- (b) Two sources of truth will eventually disagree, giving access to a cancelled user or removing it from a payer ✅
- (c) Webhooks cannot update two tables in one transaction
- (d) It duplicates data and wastes storage
> One place must decide entitlement and everything else must ask it. Both classic billing bugs are the same defect seen from two sides, and they appear as soon as a second code path exists.

### Q3 · L4
You created your UPI autopay mandates with a maximum amount exactly equal to your ₹199 price. You then add GST to the charge. What happens?
- (a) The gateway absorbs the difference and charges ₹199
- (b) The debit exceeds the mandate limit and fails, for every subscriber ✅
- (c) The user is asked to approve the extra amount at debit time
- (d) The mandate is automatically raised to cover the new amount
> The mandate is an approval for up to a stated amount. No headroom means any price change, tax addition or overage breaks every debit, and re-collecting mandates loses subscribers.

### Q4 · L4
A user upgrades mid-month. Which approach is correct?
- (a) Charge the new plan in full and keep the old charge, since both were used
- (b) Let the gateway's subscription product prorate it, then read and verify what it actually charged ✅
- (c) Compute the proration yourself from days remaining and trust the result
- (d) Start the new plan at the beginning of the next period with no charge now
> Proration is easy to get subtly wrong and the gateway already does it. Your job is to reconcile your expectation against the gateway's own record, not to invent arithmetic.

### Q5 · L5
A user says "I was paying and you cut me off." Which single thing makes this answerable rather than an argument?
- (a) A screenshot of their last successful payment
- (b) A structured log of state transitions with timestamps and causes, reconciled against the gateway ✅
- (c) The gateway's dashboard alone
- (d) The dunning emails you sent them
> The gateway knows about money and your system knows about access. Only a logged, timestamped, caused transition history tied to the gateway's record explains why a particular user lost a particular feature on a particular day.

## Go deeper
- [Razorpay: subscriptions, UPI autopay and e-mandates](https://razorpay.com/docs/payments/subscriptions/)
- [Stripe: dunning, smart retries and subscription lifecycle](https://docs.stripe.com/billing/subscriptions/overview)
