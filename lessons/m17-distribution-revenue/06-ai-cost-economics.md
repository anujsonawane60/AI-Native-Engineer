---
id: m17-06-ai-cost-economics
module: m17-distribution-revenue
slug: ai-cost-economics
title: "AI Cost Economics: When Inference Eats Your Margin"
order: 6
duration_min: 35
is_free: false
mission: true
volatility: high
last_reviewed: 2026-11-16
---

## Key learnings
- Compute cost per active user as a distribution, never as an average
- Show how five per cent of users can turn a 67% margin into a loss
- Pull the two levers that actually work: cost per request, and requests per user
- Price from the heavy user when there is no cap, and cap when you price from the mean
- Refuse any model price that did not come from the provider's own page today

## Lesson

### The pain

A team sells an AI study assistant at ₹99 a month. Each use is an agentic run — it reads the student's notes, retrieves the syllabus, thinks, drafts, checks itself — and costs about ₹4 once every step is counted with M09's method. They sanity-check the economics against their own usage: twelve runs a month.

```
  WHAT THE TEAM BELIEVED

  revenue per subscriber per month          ₹99
  requests per user per month, estimated     12
  cost per request                           ₹4
  inference cost per user                   ₹48
  ----------------------------------------------
  gross profit per user                     ₹51
  gross margin                              51%
```

A 51% margin on a ₹99 plan, from an estimated average of twelve requests per user per month.

Fifty-one per cent is a real business. They launch, reach a hundred subscribers, and the first bill arrives.

```
  WHAT 100 SUBSCRIBERS ACTUALLY DID, IN ONE MONTH

  group     users  req/user   at ₹4/req      cost
  light        70         4                ₹1,120
  medium       25        20                ₹2,000
  heavy         5       400                ₹8,000
  ----------------------------------------------
  all         100   27.8 avg              ₹11,120

  revenue, 100 x ₹99                       ₹9,900
  gross profit                            -₹1,220
```

The same hundred subscribers in three usage groups: ₹11,120 of cost against ₹9,900 of revenue.

Two things went wrong and only one is obvious. The estimated average of twelve was wrong; the real average was 27.8. But the deeper error is that **an average was used at all.**

### The idea

**Cost per active user is a distribution with a long tail, and a margin computed from its average is not a margin — it is the average of a profit and a disaster.**

```
  THE SAME MONTH WITH AND WITHOUT THE TOP 5 USERS

                      all 100    the 95   the top 5
  revenue              ₹9,900    ₹9,405        ₹495
  inference cost      ₹11,120    ₹3,120      ₹8,000
  gross profit        -₹1,220    ₹6,285     -₹7,505
  gross margin           -12%       67%      -1516%
```

Ninety-five of the hundred subscribers were a 67%-margin business; five of them destroyed it. Those five did nothing wrong — they are your best users, the ones who tell their friends, the ones M16 would call your most activated cohort. The product worked exactly as designed for them, and that is what cost ₹8,000.

And the warning that governs every number here. **Every rupee figure in this chapter is a placeholder for teaching the arithmetic. Every price an AI quotes you is probably stale. The provider's own pricing page, read today, with the date beside it, is the only authority for anything that goes into your budget.** This is M00's knowledge-cutoff failure mode landing where it costs money.

### The analogy

The canteen offers an unlimited thali at ₹60: rice, dal, sabzi, as many refills as you like.

Most students eat one and a half plates, which costs the owner about ₹40, and he is pleased. Then the wrestling team starts coming. Six of them, five plates each, every day. One wrestler costs him ₹130 against ₹60 of revenue, and the forty students in the queue behind are paying for it.

He has three moves: cap the refills, charge the wrestlers more, or make the plate cheaper. He does all three.

**Where the analogy breaks:** the owner can *see* the wrestler at the counter on the first day and improvise. Your heavy user is invisible until the bill arrives — and invisible in your product analytics too, because M16's dashboards count events, not rupees. The query that ranks users by cost does not exist until you write it, and it is the prerequisite for every decision in this chapter.

### Picture it

```
  REQUESTS PER USER IN ONE MONTH, 100 SUBSCRIBERS

  4 requests    ████████████████████████████████   70 users
  20 requests   ████████████                       25 users
  400 requests  ██                                  5 users

  SHARE OF ALL REQUESTS CONSUMED

  light,  70 users  ███                            10%
  medium, 25 users  █████                          18%
  heavy,   5 users  ████████████████████████       72%
```

A tiny group makes most of the requests: five per cent of subscribers consumed 72% of them. This is the normal shape of usage in almost every product, so plan for it rather than discovering it.

### The two levers

There are only two, and for a product like this you need both.

**Lever one: cost per request.** M09 chapter 14's territory, in order of return: trim the system prompt and the history, cap max output tokens, cache repeated and near-repeated work (M05's caching, M10's prompt caching), route easy requests to a smaller model, and only then redesign. M14's work on smaller and distilled models is what makes routing real rather than theoretical.

**Lever two: requests per user.** An allowance in the plan, a visible meter, and a hard cap — chapter 3's hybrid of base fee plus allowance plus overage, which is why it exists.

```
  TWO LEVERS, APPLIED TO THE SAME 100 USERS

  what changed                     cost    profit  margin
  nothing                       ₹11,120   -₹1,220    -12%
  cap at 60 requests a month     ₹4,320    ₹5,580     56%
  cost per request ₹4 to ₹0.40   ₹1,112    ₹8,788     89%
  both                             ₹432    ₹9,468     96%
```

Either lever alone makes the product solvent; both together make it comfortable.

But read the second row carefully. The cap fixes your margin by taking the product away from your five best users at request sixty-one, and you cannot solve that by selling them a bigger plan, because at ₹4 a request their 400 runs cost ₹1,600 and no Indian student pays ₹1,600 a month. **The cap makes you solvent; only the cost reduction makes a higher plan sellable.** At ₹0.40 a request, a ₹499 plan with a 400-request allowance costs you ₹160 and earns ₹339 — a 68% margin, and your best users stay.

That is the whole lesson: pricing cannot rescue a cost per request that is too high for what your market will pay, because the price ceiling is set by your market and not by you.

### Pricing when your variable cost is real

Classic software pricing assumes serving one more user costs nothing. Yours does not, so one rule changes.

**If a plan has no cap, price it from the heavy user, not the average one.** Find the user at the 95th percentile of your cost distribution and treat their cost as your floor. Pricing an uncapped plan from the mean guarantees the tail is loss-making, and the tail is where your most enthusiastic users live.

**If you want to price from the mean, you must cap.** The allowance converts an unbounded promise into a bounded cost, and it is what lets you quote a number a student will pay. These are the same rule from both ends, and one of them must be true of every plan you publish.

A cap only works if it does not feel like a punishment. Four things make it bearable: an allowance covering roughly 95% of users; a meter in the product so nobody is surprised; a warning at 80% with a clear next step; and degradation rather than a stop at the cap — fall back to the smaller model instead of a locked door. A hard stop at request sixty with no warning turns your best user into the screenshot in chapter 9.

### The free tier is an unbounded liability

One free user behaving like your heavy user costs ₹1,600 a month at ₹4 a request. Ten of them cost more than your entire paying base brings in, and they can arrive in an afternoon if a post does well.

So: no free tier without a cap, ever, for anything with a per-request cost. A free tier is a marketing budget, so give it a number — "at most ₹3,000 a month on free usage" — and enforce it with per-user limits and an account spend cap. M00's billing alerts are the backstop, and the backstop is not the plan.

### Where it lives

M09 chapter 14 gives you cost per request and the six multipliers; this chapter turns that number into a commercial decision. M05's caching and M14's smaller models are the mechanics of lever one, and M10's prompt caching is often the cheapest win. Chapter 2 set the price ceiling, chapter 3 chose the structure carrying the allowance, chapter 4 enforces the overage, chapter 5 consumes this as its inference line, and chapter 11 keeps it on the scorecard because it moves. M16 tells you who your heavy users are; this chapter tells you what they cost.

### Words to own

| Term | Plain meaning | Example |
|---|---|---|
| Cost per request | One complete user action, all steps | ₹4 |
| Cost per active user | What one user costs you in a month | ₹111, not ₹48 |
| Distribution | The whole spread, not the average | 4, 20 and 400 requests |
| Heavy-user tail | The small group consuming most usage | 5% using 72% |
| p95 user | The user at the 95th percentile of usage | 400 requests |
| Allowance | Usage included in the plan | 60 requests a month |
| Overage | What you charge beyond the allowance | ₹1 per extra run |
| Hard cap | Where usage stops or degrades | 60, with a fallback |
| Cache hit rate | Share served without inference | Measure, do not assume |
| Model routing | Easy work to a cheaper model | M14's territory |
| Spend cap | A hard ceiling on the provider account | The backstop |

### Tradeoffs and gotchas

**Caching is the best lever and the most over-claimed.** "Caching will fix it" means nothing without a measured hit rate. A 40% hit rate cuts cost by 40%, not 95%, and a semantic cache that returns the wrong answer to a slightly different question is cheaper and worse.

**Cheaper and worse is not a win.** Rerun five realistic requests after each lever and judge the output yourself.

**Your cost moves without you doing anything.** Prices get cut, models get retired, token accounting changes, and the exchange rate moves against you because providers bill in dollars and you earn rupees.

**An agent step is not a request.** One user action in an agentic product is four or five model calls with a growing context, which is why ₹4 rather than ₹0.40 is realistic here. Price per *user action*.

**Do not cap your way out of a bad product.** If almost everyone hits the cap, the allowance is wrong, not the users. And a runaway retry loop is not a margin problem but an unbounded liability.

### Verify it

**Red flag, the headline one: a margin computed from an average cost per user.** Ask for the distribution: the median, the 90th percentile, the 95th, and your single most expensive user. One number is an incomplete answer.

**Red flag: a gross margin table with no inference line.** Chapter 5's commonest omission starts here.

**Red flag: no query that ranks users by cost.** You cannot manage a tail you cannot see. If nobody can run "top twenty users by cost last month" today, build it first.

**Red flag: a quoted model price.** Any figure per million tokens, from any AI, is a rumour from before its cutoff. Open the provider's page, read it, write the date. An AI that gives you a price *without* telling you to verify it is itself the warning.

**Red flag: a cap with no in-product meter and no warning.** You have built a trap rather than a plan. **A free tier with no per-user cap and no account spend cap** is not a pricing error but an open liability.

**Red flag: "caching will fix it" with no hit rate**, or **a plan priced from the mean user with no allowance.** Ask what share of requests were served from cache last week, measured; and check that one of the two pricing rules holds, because if neither does, the heavy users are loss-making by construction.

And the measurement nothing substitutes for: query your own logs for cost per user last month, sort descending, and look at the top five rows. That list is your business.

## Walkthrough
1. Confirm you have the per-call logging from M09 chapter 14 in place: input tokens, output tokens, model name and estimated cost on every model call, tagged with the user id. If it is missing, add it first; nothing below is possible without it.
2. Open your provider's own pricing page, write down the input and output price per million for your exact model string, and write today's date beside them. Use no price from any other source.
3. Compute cost per *user action*, not per model call: count how many model calls one complete action makes, including retries and agent steps, and sum them.
4. Write the query that reports, for last month: user id, requests, and estimated cost, sorted by cost descending. Run it.
5. From that output, record five numbers: the mean, the median, the 90th percentile, the 95th percentile, and the single most expensive user's cost.
6. Build the three-group table like the one in this lesson — light, medium and heavy — with user counts, requests each and total cost. If you have fewer than twenty users, use your five percentile numbers and label every projected figure as an assumption.
7. Compute gross profit three ways: all users, all users except your top five per cent, and the top five per cent alone. Write all three margins.
8. Set an allowance that covers about 95% of your users, and recompute total cost with that cap applied. Record the new margin.
9. Pull one cost-per-request lever — trim the prompt, cap output tokens, add caching, or route to a smaller model — measure the new cost per action, and recompute. Then rerun five realistic requests and judge the quality yourself.
10. Decide which rule your plan obeys: priced from the p95 user with no cap, or priced from the mean with a cap. Write it as one sentence.
11. Set or confirm a per-user rate limit, an account-level spend cap and a billing alert, and screenshot all three.

## Talk to AI
```prompt
I need to decide whether my AI product can have a margin, and I need you to follow my rule about prices exactly.

RULE: do NOT give me any price per million tokens, in any currency, and do not estimate my cost per request. For every price I need, tell me which provider page to open and what to look for, and state plainly that any pricing you remember may be stale and must not go into my budget.

My plan price and billing period: [from chapter 2]
My measured cost per complete USER ACTION, in rupees: [from M09's method - say how many model calls one action makes]
My subscriber count: [number]
My per-user usage distribution for last month, from my own logs:
  mean requests per user: [X]
  median: [X]
  90th percentile: [X]
  95th percentile: [X]
  single most expensive user: [requests and rupees]
Is the feature available to non-paying users? [yes/no, and is there a cap]
My other per-user costs per month: [gateway fee, hosting, etc.]
What my market will realistically pay at the top end: [the highest price a real user of mine would accept]

Do all of this, in order:

1. Build me a three-group table - light, medium, heavy - from MY percentile numbers, with users, requests each, and total cost. Do not invent a distribution; derive it from what I gave you and say which parts are interpolated.

2. Compute gross profit and gross margin three ways: all users, all users except the top five per cent, and the top five per cent alone. Show the arithmetic.

3. Tell me bluntly whether my top five per cent are profitable, and what share of my total inference cost they are.

4. Apply lever two: propose an allowance that covers about 95% of my users, recompute the margin with that cap, and tell me how many of my current users would hit it.

5. Apply lever one: list the cost-per-request levers in order of return for MY structure, and for each one tell me roughly what it saves and whether a user would notice. Tell me which ones I can do this week.

6. Now tell me whether a higher-priced plan for my heavy users is sellable at my current cost per request. Show the arithmetic: their usage times my cost, against the highest price my market would accept. If it is not sellable, say so and tell me what cost per request would make it sellable.

7. Tell me which of the two pricing rules my plan should obey - priced from the p95 user with no cap, or priced from the mean with a cap - and write the one-sentence version I can put in my decision log.

8. Design the cap so it does not feel like a punishment: where the meter lives, when the warning fires, and what degradation rather than a hard stop would look like in my product.

9. If non-paying users can trigger the feature, tell me what my maximum monthly exposure is if ten free users behave like my heaviest paying user, and what per-user and account-level limits I need.

10. Finally, tell me what I should put on a weekly scorecard so this never surprises me again, and what would make me need to rebuild this whole model.
```

## Mission

**Objective.** The cost-per-active-user distribution for your own product, the margin computed three ways, one cost lever pulled and measured, a cap chosen with a visible meter, and one sentence saying which pricing rule your plan obeys.

**Steps.**
1. **Frame** — name the one AI feature this mission is about, and what one complete user action means in your product.
2. **Check the instrumentation** — confirm every model call logs tokens, model, estimated cost and user id. If not, fix that first; it is the prerequisite.
3. **Read the real prices** — your provider's own page, your exact model string, with today's date. No price in this mission may come from an AI.
4. **Cost per user action** — count the model calls in one action, including retries and agent steps, and compute the rupee figure.
5. **Query the distribution** — users ranked by estimated cost last month. Record the mean, median, p90, p95 and your single most expensive user. If you have fewer than twenty users, project the distribution and label every assumption.
6. **Margin three ways** — all users, all users minus the top five per cent, and the top five per cent alone. Write all three.
7. **Choose the cap** — an allowance covering roughly 95% of your users. Recompute the margin with it applied and count how many current users would hit it.
8. **Pull one cost lever** — trim, cap output, cache or route. Measure the new cost per action, recompute the margin, and record the before and after.
9. **Check what you broke** — rerun five realistic requests through the cheaper version and judge the output yourself. Say so plainly if it got worse.
10. **Build the meter** — the user can see their usage, is warned at 80%, and at the cap degrades rather than stops.
11. **Decide the rule** — write the one sentence: priced from the p95 user with no cap, or priced from the mean with a cap.
12. **Guardrails** — per-user rate limit, account spend cap, billing alert. Screenshot all three.
13. **Document** — the whole model with today's date, the prices you read, and the trigger to rebuild it.

**Upload.**
- Your provider's pricing page with the date visible → screenshot
- Your cost per complete user action, with the model-call count shown → text
- Your ranked cost-per-user query and its output → screenshot
- Your five distribution numbers: mean, median, p90, p95, most expensive user → text
- Your three-group table → spreadsheet or screenshot
- Gross margin computed three ways, with the arithmetic visible → screenshot
- Your chosen allowance, the recomputed margin, and the count of users who would hit it → text
- The cost lever you pulled, with cost per action before and after → text
- Five real outputs from the cheaper version, so quality can be judged → file
- Your in-product meter and the 80% warning → screenshots
- Your one-sentence pricing rule → text
- Your per-user rate limit, account spend cap and billing alert → screenshots
- The AI conversation → share link in `ai_chat_url`
- Reflection, 150 words max → what your single most expensive user cost you, and one price or limit an AI gave you that turned out to be stale

**Self-check.** Tick every box before you submit.
- [ ] Every model call in my product logs tokens, model, cost and user id
- [ ] Every price in my model came from the provider's own page, with the date
- [ ] No price or limit in this mission came from an AI
- [ ] My cost is per complete user action, counting every model call in it
- [ ] I have a query that ranks my users by cost, and I ran it
- [ ] I report mean, median, p90, p95 and my most expensive user — not just an average
- [ ] My margin is computed three ways, including the top five per cent alone
- [ ] My allowance was chosen from my distribution, not picked round
- [ ] I pulled a cost lever and measured the result rather than assuming it
- [ ] I reran real requests after the lever and judged the quality myself
- [ ] My cap has a visible meter, a warning before it, and degradation rather than a locked door
- [ ] My plan obeys one of the two pricing rules, written as one sentence
- [ ] Non-paying users cannot trigger unlimited requests
- [ ] A per-user rate limit, an account spend cap and a billing alert are live right now
- [ ] My decision log has today's date and a trigger to rebuild the model

**Stretch.** Model the week a post about you does well: four hundred signups in two days, ten of them behaving like your heaviest user, and your cache cold. Compute the bill. Then say which of your three guardrails would actually stop it and which would simply be exhausted.

## Stuck? Ask AI
```prompt
Explain why a margin computed from the average cost per user can be badly wrong, using a worked example of 100 users split into three usage groups. Show the total both ways. Then ask me what my own distribution looks like before you say anything about my product. Do not quote me any model prices.
```

```prompt
My five heaviest users cost me more than they pay. Here are my numbers: [price, cost per request, their usage]. Do not tell me to remove them. Instead, lay out every option - cap, overage, a higher plan, cutting cost per request, degrading to a cheaper model - and for each one tell me what it does to my margin and what the user experiences.
```

```prompt
Explain the difference between a usage cap that feels fair and one that feels like a trap, with concrete examples of what the user sees in each case. Then tell me the four things my cap needs before I ship it, and what happens to my support queue if any one of them is missing.
```

## Quiz

### Q1 · L3
Your product has 100 subscribers at ₹99. Five of them make 400 requests a month at ₹4 each; the other 95 make few. What does your average cost per user hide?
- (a) Nothing; the average is the correct basis for a margin
- (b) That those five cost ₹8,000 against ₹495 of revenue, and they alone turn a 67% margin into a loss ✅
- (c) That the other 95 users are unprofitable
- (d) That the cost per request estimate is too low
> Usage is a long-tailed distribution, so the mean sits between a profitable body and a loss-making tail. The margin has to be computed on the groups, not on the average.

### Q2 · L4
You cap usage at 60 requests a month and your margin recovers to 56%. Why is a higher-priced plan for your heavy users still not sellable at ₹4 per request?
- (a) Heavy users never upgrade
- (b) Their 400 requests cost ₹1,600, which is far above anything your market will pay ✅
- (c) Caps and higher plans cannot coexist in one catalogue
- (d) The gateway cannot charge two different amounts
> The price ceiling is set by your market, not by you. Only reducing cost per request makes a bigger plan possible, which is why you need both levers rather than either.

### Q3 · L4
Your plan has no usage cap. From which user should you set the price floor?
- (a) The median user, since half are below them
- (b) The mean user, since that is your average cost
- (c) The user at the 95th percentile of usage ✅
- (d) The single most expensive user ever recorded
> Without a cap you have promised unlimited use, so the floor must cover the heavy end. Pricing an uncapped plan from the mean makes the tail loss-making by construction.

### Q4 · L4
An AI gives you a specific price per million tokens and a complete monthly margin model built on it. What should you do?
- (a) Use it; the arithmetic is what matters
- (b) Keep the method, discard the price, and read the real one on the provider's page with today's date ✅
- (c) Add 20% as a safety buffer
- (d) Ask the same AI to double-check its figure
> The method is sound and the number is a rumour from before its cutoff. Asking the model to verify itself produces a second rumour, not a source.

### Q5 · L5
You offer a free tier with no per-user cap on an AI feature that costs ₹4 a request. A post about you does well overnight. What have you actually built?
- (a) A growth loop, since free users drive word of mouth
- (b) An unbounded liability, because cost scales with usage and revenue does not ✅
- (c) A pricing problem that a paid tier will fix
- (d) A caching problem
> Free converts cost into an obligation with no revenue attached. One enthusiastic free user can cost more than a hundred paying ones, so a free tier needs a stated budget, per-user limits and an account spend cap.

## Go deeper
- [Anthropic: pricing — read the live page, never a remembered number](https://www.anthropic.com/pricing)
- [OpenAI: API pricing and the usage figures returned in responses](https://platform.openai.com/docs/pricing)
