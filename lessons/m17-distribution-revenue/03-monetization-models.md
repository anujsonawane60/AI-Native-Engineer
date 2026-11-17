---
id: m17-03-monetization-models
module: m17-distribution-revenue
slug: monetization-models
title: "Monetization: Subscriptions, Usage, Freemium, Marketplace"
order: 3
duration_min: 15
is_free: false
mission: false
volatility: low
last_reviewed: 2026-11-16
---

## Key learnings
- Match the monetization model to how value arrives, not to what other startups do
- Name the three conditions under which freemium works, and refuse it otherwise
- Choose between subscription, usage, prepaid credits, one-time and commission with a reason
- See what each model costs you in billing, support and refund work before you pick it
- Catch a model that cannot be metered honestly or predicted by the buyer

## Lesson

### The pain

A team builds a genuinely useful tool for assembling internship applications and monetizes it as a ₹149 monthly subscription, because every product they admire has one. But a student uses it hard for eleven days in August and has no reason to open it again until next August. Month two churn is 94%. They conclude their retention is broken and spend three weeks building nudges to drag people back into a product they are finished with. The retention was never broken: value arrived in a lump and they charged for it as if it arrived continuously.

A second team sells a document-summarising tool by the page. Fair, scalable, and nobody starts, because a user cannot answer "what will this cost me this month?" before they begin — no meter, no estimate at checkout, no cap. Neither team mispriced. Both chose a structure that fought the shape of their own product.

### The idea

**Charge in the same rhythm that value arrives: continuously for continuous value, per lump for lumpy value, and up front for value you deliver once.**

Chapter 2 gave you the number and the value metric; this chapter is the container it is collected through. Every structure you add is a real engineering and support cost, which is the tradeoff nobody budgets for.

Most real products end up as a base plus a meter: "₹199 a month including 100 orders, then ₹1 per order". That gives predictable revenue, a cap on your own cost exposure, and a natural upgrade conversation when someone keeps exceeding the allowance — the structure that survives chapter 6 best.

**Freemium** needs its own warning. A permanently free tier works under exactly three conditions, all of which must hold: the marginal cost of a free user is near zero, free users create value for you anyway by inviting others or creating content others see, and there is a clear moment where the free user hits a wall they want to pay to pass. If any one fails, freemium is a cost centre with a growth chart attached. For any product with a per-request AI cost the first condition usually fails outright.

### The analogy

The canteen runs five models at once, all visible from the counter. The **monthly mess pass** is a subscription: predictable for both sides, and he knows his revenue on the first. **Pay per plate** is usage-based: fair, and the student decides daily. The **coupon book**, ₹500 for twelve meals, is prepaid credits, which he loves because the cash arrives before the food does and some coupons are never redeemed. The **free chutney** is freemium: almost no marginal cost, genuinely liked, and the reason people come in and buy the thali. The **juice stall at the gate** pays a share of its takings to be there — a marketplace commission, paid because the canteen owns the footfall.

**Where the analogy breaks:** he runs all five with a register, a rubber stamp and a human memory. Every model you add in software is a billing integration, a state machine, a refund policy, a support category and a set of edge cases where models meet. His sixth model costs a new notebook. Yours costs a fortnight.

### Picture it

```
  HOW VALUE ARRIVES  ──▶  HOW TO CHARGE

  continuous, every week        ──▶  subscription
  "I use this most days"             monthly or annual

  bursty, unpredictable         ──▶  usage, or prepaid credits
  "some weeks 50, some 0"            with an in-product meter

  one big lump, then done       ──▶  one-time purchase
  "I needed it for admissions"       or a short paid pass

  value is OTHER PEOPLE         ──▶  commission on the
  "I came for the sellers"            transaction, a take rate

  near-zero marginal cost       ──▶  free tier that LEADS to
  and it sells the paid thing         a paid thing, i.e. freemium
```

Each shape of value arrival maps to a charging structure, and a mismatch between the two shows up later as churn or as a buyer who never starts.

| Structure | Fits when | What it costs you to run |
|---|---|---|
| Subscription | Value arrives most weeks | Plan catalogue, subscription state machine, dunning, invoices, cancellation |
| Usage-based | Your spend tracks their use | Trustworthy metering, a visible meter, an estimate at the point of use, a hard cap |
| Prepaid credits | The recharge habit fits; cash arrives first | Expiry policy, refunds for unused credits, revenue taken but not earned |
| Free trial | Value takes a few days to feel | A trial state, and whether a mandate is required up front |
| Freemium | All three conditions hold | A free user's cost, forever, plus a designed paywall |
| One-time | Value arrives in one lump | Nothing compounds, so acquisition must keep working forever |
| Commission | The value is other people | Both sides at once, liquidity, settlement, payouts, disputes |
| Advertising | Volumes you will not have for years | Ruled out, with a reason |

The right-hand column is billing work that must exist before the first rupee arrives, and it is why two structures is usually the limit for a team of three.

### Where it lives

Chapter 4 builds the container you chose: plans, entitlements, upgrades, dunning and invoices, on top of M04's payments mechanics. Chapter 5 is where the model's consequences show up — a one-time product has no LTV curve, a credit product has deferred revenue, a marketplace's ARPU is the take rate and not the basket. Chapter 6 is the constraint that overrides preference: with an inference cost per request, your model must contain a cap somewhere. M12 applies if you sell inside a mobile app, where the store's cut changes which models are permitted. M16 measures whether the free tier is doing its job — "free users who never convert and never invite anyone" is a measurable fact, not an opinion.

### Words to own

| Term | Plain meaning | Example |
|---|---|---|
| ARPU | Average revenue per user, per period | ₹99 a month |
| MRR | Monthly recurring revenue | 100 x ₹99 = ₹9,900 |
| Metered billing | Charging for measured consumption | ₹1 per order |
| Allowance | Usage included in a flat fee | 100 orders included |
| Overage | What you charge past the allowance | ₹1 per extra order |
| Credits | Prepaid units spent inside the product | ₹500 for 12 coupons |
| Deferred revenue | Money taken, value not yet delivered | Unredeemed credits |
| Freemium | A permanently free tier | Free chutney |
| Free trial | Time-boxed full access | 7 days, then decide |
| Take rate | Your percentage of a transaction | 8% of each order |
| Liquidity | Enough supply that a visit succeeds | Sellers on both sides |
| Paywall | The moment the free user is asked to pay | The 11th order |

### Tradeoffs and gotchas

| Gotcha | What it means for you |
|---|---|
| Do not copy the model of a product unlike yours | The subscription you admire belongs to a product people open daily; if yours is opened eleven days a year you are buying its billing complexity and none of its revenue |
| A usage model with no in-product meter | A trust problem, not a pricing one — users must see what they have consumed at any moment without asking you. Add the meter before the billing |
| Credits with no expiry decision | Never expiring is simplest and kindest, expiring is better for your cash and worse for your reputation; either is defensible, deciding after a complaint is not |
| Changing model is harder than changing price | Moving from usage to subscription migrates every customer's expectations, mandate and invoices — pick deliberately now |
| A marketplace with one side is a directory | If nothing transacts your revenue is a percentage of zero, and your take rate must stay low enough that sellers do not route around you, which they will the moment they can |

### Verify it

**Red flag: the model does not match how value arrives.** Make the AI state in one sentence how often a typical user gets value, then check its recommended model against that sentence. A subscription proposed for a once-a-year product is the commonest failure, and it is visible in the AI's own answer.

**Red flag: freemium recommended without the three conditions tested.** Ask what a free user costs per month, what they do for you if they never pay, and where exactly the wall is. A missing answer means the recommendation is a habit, not a decision.

**Red flag: usage pricing with no meter, no estimate and no cap.** Three separate things; the absence of any one shows up as users who sign up and never start.

**Red flag: a recommendation that ignores your per-request cost.** Monetization advice given without asking what one use costs you is advice about a zero-marginal-cost product, and yours probably is not one.

The honest check: write one sentence for a typical user that begins "You will be charged..." and ends with something they could predict before being charged it. If you cannot write that sentence, your users cannot either.

## Walkthrough
1. Write one sentence describing how often a typical user gets value from your product: most days, a few times a month, a burst then done, or once.
2. Write down what one use of your product costs you in variable cost — inference, storage, a third-party call, a delivery. If it is effectively zero, say so explicitly, because it changes which models are safe.
3. Shortlist two monetization structures from the list in this chapter that match your answer to step 1, and write one line on why each fits.
4. For each shortlisted structure, write down what it will cost you to build and run: the billing states, the support questions it creates, and the refund policy it needs.
5. If freemium is on your shortlist, test it against all three conditions in writing. Strike it if any condition fails.
6. Design the hybrid version: a base fee with an allowance, plus a price above the allowance. Write the three numbers — fee, allowance, overage price.
7. Write the "You will be charged..." sentence for a typical user, and a second one for your heaviest plausible user. If the second sentence alarms you, your cap is missing.
8. Sketch the in-product meter: where the user sees consumption, what it says when they are near the allowance, and what happens at the cap.
9. Show both sentences and the meter sketch to three real users. Ask each one what they think it would cost them this month, and write down their answer against the real one.
10. Record the decision in your decision log: the model you chose, the one you rejected, the condition that decided it, and what would make you change.

## Talk to AI
```prompt
I need to choose a monetization structure for my startup. Do not recommend a model until you have done the analysis below, and do not assume my marginal cost per use is zero.

My product and position: [the four lines from chapter 1]
How often a typical user gets value: [most days / a few times a month / a burst then done / once]
My value metric and price from chapter 2: [both]
What ONE use of my product costs me in variable cost, in rupees: [number, or "unknown" - include inference, third-party calls, delivery, storage]
Who pays, and is it the same person as the user: [answer both]
Is there a second side to my product - sellers, providers, institutions? [yes/no, describe]
Target users in six months: [number]
Our engineering capacity: three students, no paid tooling, payments in test mode only.

Do all of this, in order:

1. Restate in one sentence how often my user gets value. Then tell me which monetization structures are ruled OUT by that sentence alone, and why.

2. For each of subscription, usage-based, prepaid credits, free trial, freemium, one-time and commission, give me one line on whether it fits my product and one line on what it would cost me to build and support. Be concrete about the billing states each one needs.

3. Test freemium against all three conditions explicitly: what a free user costs me per month, what a free user does for me if they never pay, and where the wall is. If any condition fails for me, say freemium is not available and explain why.

4. Design a hybrid of a base fee plus an allowance plus an overage price, using my numbers. Give me the three figures and show how you derived the allowance from my variable cost.

5. Write the sentence "You will be charged..." for a typical user, and again for a user who uses my product twenty times as much as the typical one. Tell me what the second sentence reveals about my exposure.

6. Specify the in-product meter: what the user sees, when they are warned, and what happens at the cap. Tell me what goes wrong if I ship the billing before the meter.

7. If my product has a second side, tell me which side I must get first and what a realistic take rate looks like, plus what happens when sellers try to route around me.

8. Finally, recommend one structure, name the strongest argument against your own recommendation, and tell me what I should measure in month one to find out if you were wrong.
```

## Stuck? Ask AI
```prompt
I do not understand the difference between a free trial and freemium, or when each one is the right choice. Explain both with two Indian consumer examples each, then give me five product descriptions and ask me which of the two suits each one, without telling me the answers first.
```

```prompt
Explain prepaid credits as a monetization model: why the cash timing helps me, what deferred revenue means for my own honesty about how much money I have, and every policy decision I must make before launch. List the policy decisions as questions I have to answer, not as recommendations.
```

```prompt
My team wants to build a marketplace and take a commission. Here is the product: [describe both sides]. Do not encourage me. Tell me the chicken-and-egg problem in my specific case, which side is harder to get and why, what liquidity would have to look like before the model works at all, and what happens to my revenue when the two sides start dealing with each other directly.
```

## Quiz

### Q1 · L3
A product is used intensely for eleven days in August and then not again until the next August. Which monetization structure fits?
- (a) A monthly subscription with notification nudges to improve retention
- (b) A one-time purchase or a short paid pass ✅
- (c) Freemium with a paid tier unlocked after thirty days
- (d) Usage-based billing per page processed
> Value arrives in one lump, so charge for the lump. A monthly subscription on a once-a-year product produces 94% month-two churn that no onboarding work can fix.

### Q2 · L4
Which single condition most often rules freemium out for a student AI product?
- (a) Free users do not generate word of mouth
- (b) The marginal cost of a free user is not near zero, because every request costs inference ✅
- (c) The paywall is hard to design
- (d) Indian users rarely convert from free to paid
> Freemium needs near-zero marginal cost. A free user making AI requests costs real money every time, so growth in free users is growth in loss. Chapter 6 is about this.

### Q3 · L4
You ship usage-based pricing with no in-product meter. What is the most likely symptom?
- (a) Users are billed incorrectly and demand refunds
- (b) Users sign up and never start, because they cannot predict the bill ✅
- (c) Heavy users churn after their first invoice
- (d) Your metering costs more than the revenue it collects
> Uncertainty suppresses the first action. The meter, the estimate at the point of use and the hard cap are three separate requirements, and the meter is the one that unblocks adoption.

### Q4 · L3
Why is "a base fee with an allowance, plus a price above the allowance" the hybrid that usually survives a real per-request cost?
- (a) It is the structure most competitors use
- (b) It gives predictable revenue, caps your cost exposure, and creates a natural upgrade conversation ✅
- (c) It avoids the need for a metering system
- (d) It removes the need for a cancellation flow
> The allowance is where you convert your variable cost into a bounded commitment, and the overage price is what makes a heavy user pay for the cost they create rather than someone else absorbing it.

### Q5 · L5
An AI recommends a monetization model for your product without ever asking what one use costs you. What is the flaw in its advice?
- (a) It cannot know your competitors' prices
- (b) It is reasoning about a zero-marginal-cost product, which yours may not be ✅
- (c) It will always recommend subscriptions
- (d) Its pricing knowledge is out of date
> Classic software advice assumes serving one more user costs nothing. With inference in the loop that assumption is false, and every model it suggests inherits the error silently.

## Go deeper
- [Lenny's Newsletter: choosing a business model and pricing structure](https://www.lennysnewsletter.com/p/how-to-price-your-product)
- [a16z: the marketplace liquidity problem and take rates](https://a16z.com/marketplace-100/)
