---
id: m09-14-token-economics
module: m09-ai-foundations
slug: token-economics
title: "Token Economics: What Your AI Feature Actually Costs"
order: 14
duration_min: 35
is_free: false
mission: true
volatility: high
last_reviewed: 2026-11-16
---

## Key learnings
- Compute your own cost per request from measured token counts and today's published price
- Turn one request into a monthly figure, and compare it against revenue per user
- Name the six multipliers that make a cheap-looking feature expensive
- Apply the levers in order of return: trim, cap, cache, route, then redesign
- Treat every price quoted by anyone, including an AI, as stale until read on the provider's page

## Lesson

### The pain

A team ships an AI tutor for their canteen-ordering startup. It explains dishes, suggests meals within a budget, answers in Marathi. The demo is the best in the cohort.

They price it at ₹49 a month and sign up 1,500 students. Revenue is about ₹73,500 a month, which feels like a business.

Then the first real bill arrives. Their tutor uses a reasoning model, carries a 1,200-token system prompt, keeps the full conversation history, and retrieves menu context every turn. One turn consumes roughly 6,000 input tokens and about 1,800 output tokens once hidden thinking tokens are counted. Conversations run about eight turns. Students have about three a month.

That is roughly ₹65,000 a month in model costs against ₹73,500 of revenue, before hosting, before the database, before anyone's time. The product does not have a margin. It has a leak.

Nobody did anything stupid. Every decision was reasonable. None of them were ever multiplied out, and the arithmetic takes twenty minutes.

### The idea

**Cost per request is measured tokens multiplied by today's published price. Everything else is that number multiplied by how often it happens, and compared against what a user is worth to you.**

Two numbers decide whether an AI feature can exist: **cost per successful request** and **revenue per active user per month**. If the first, multiplied by how often a user triggers it, approaches the second, you do not have a product — and at Indian price points, where ₹49 or ₹99 a month is what a student will actually pay, this happens far more easily than teams expect.

And the warning that governs this entire chapter, stated once and meant absolutely. **Every price in this chapter is a placeholder for teaching the arithmetic. Every price an AI gives you is probably stale. The provider's own pricing page, read today, with the date written next to it, is the only authority for any number that goes into your budget.** Prices get cut, restructured, tiered and withdrawn; an AI's knowledge of them predates its cutoff, which chapter 11 explained is structural. This is M00's knowledge-cutoff failure mode landing exactly where it costs money.

### The analogy

The canteen prices a thali at ₹60 and the owner is pleased, because the rice, dal and vegetables come to about ₹22.

Then someone sits down with the ledger. The two extra chapatis nobody charges for. The curd refill a third of customers ask for. The 4% of plates sent back and remade. The gas. The ₹3 of pickle. The plate washed by someone who is paid. The real cost is ₹51, and on high-wastage days the thali loses money while the queue is at its longest.

Nothing was stolen and nobody made a bad decision. The owner simply never wrote down every line that attaches to one plate, and so never knew the number that mattered.

**Where the analogy breaks:** the canteen's input prices move slowly. Model prices move fast and in both directions — a model can get four times cheaper in a year, or be retired and replaced at a different price. So your cost model is not a spreadsheet you build once. It is a spreadsheet with a date on it and a reason to be rebuilt.

### Picture it

```
  WHAT ONE REQUEST IS MADE OF

  ┌─ INPUT TOKENS ──────────────────────────────────┐
  │  system prompt     re-sent EVERY turn           │
  │  retrieved context re-sent every turn           │
  │  conversation history  grows with every turn    │
  │  the user's message    usually the smallest part│
  └─────────────────────────────────────────────────┘
  ┌─ OUTPUT TOKENS ─────────────────────────────────┐
  │  thinking tokens   invisible, usually billed    │
  │  the visible answer                             │
  └─────────────────────────────────────────────────┘
          ×  retries  ×  turns  ×  agent steps
          ×  requests per user  ×  users
                        │
                        ▼
              ₹ per month, versus
              ₹ revenue per user
```

One request is built from several token buckets, and the monthly bill is that request multiplied by every repetition in your product.

### The arithmetic, done once, slowly

Here is the tutor from the top of the chapter, as a ledger. **The rates below are invented round numbers so the method is clear — do not reuse them.**

```
  EXPENSIVE VERSION  (reasoning model)
  rates used here:  input ₹120 per million · output ₹600 per million

  item                        tokens      cost
  system prompt                1,200
  retrieved menu context         800
  conversation history         4,000
  user's message                  40
  ----------------------------------------
  input subtotal               6,040      ₹0.72
  thinking tokens              1,500
  visible answer                 300
  ----------------------------------------
  output subtotal              1,800      ₹1.08
  ========================================
  cost per turn                           ₹1.80
  × 8 turns per conversation             ₹14.40
  × 3 conversations per user per month   ₹43.20
  × 1,500 active users               ₹64,800
```

The tutor's monthly model cost, built up from one turn to the whole user base.

Read the last three lines again. One turn costs under two rupees and nobody would blink at it. The same number, multiplied by turns, conversations and users, is ₹64,800 against ₹73,500 of revenue.

Now the same feature after the levers, with a cheap model at an invented ₹20 per million input and ₹80 per million output:

```
  TRIMMED VERSION  (small model, no reasoning)

  item                        tokens      cost
  system prompt (rewritten)      400
  retrieved context (2 chunks)   600
  history (last 2 turns only)  1,000
  user's message                  40
  ----------------------------------------
  input subtotal               2,040      ₹0.04
  visible answer (capped)        300      ₹0.024
  ========================================
  cost per turn                           ₹0.064
  × 8 turns                               ₹0.51
  × 3 conversations                       ₹1.54
  × 1,500 users                         ₹2,310
```

The same feature, rebuilt with four decisions changed, at roughly one twenty-eighth of the cost.

₹64,800 becomes ₹2,310, with nothing removed that a student would notice. Four decisions changed: a shorter system prompt, less history, fewer retrieved chunks, and a smaller model without reasoning — which chapter 13's comparison would have shown was good enough.

### The six multipliers

Cheap-looking features become expensive through these, in roughly this order of frequency.

**The system prompt, charged every turn.** A 1,200-token prompt across eight turns is 9,600 input tokens of pure repetition. M10's prompt caching is the right fix; shortening it is the free one.

**Conversation history.** Each turn re-sends all prior turns, so a ten-turn chat costs far more than ten single turns — closer to fifty, as chapter 8 showed.

**Retries.** Malformed JSON, a timeout, a rate limit. If one request in ten retries, your real cost is ten per cent higher, and uncapped retry loops have produced genuinely alarming bills.

**Thinking tokens.** A reasoning model can spend thousands of invisible billed tokens before a short answer.

**Agent steps.** An agent that reads, thinks, calls a tool, reads the result and answers has made four or five calls for one user action, each carrying the growing context. This is M10's territory and where costs surprise people most.

**Free users.** If the feature is open to anyone, cost scales with usage and revenue does not. One enthusiastic user can cost more than a hundred paying ones, which is why per-user rate limits are a cost control and not only an abuse control.

### Where it lives

Three places in the course meet here.

**M00's billing alerts are the safety net, and they are not optional.** Set a hard spend limit and an alert at a fraction of it today, before any feature ships. A runaway retry loop at 2 am is how students lose a semester's budget overnight.

**M05's caching is the mechanics.** Identical or near-identical requests should not be paid for twice. Of the levers, caching has the best return for a product where many users ask similar questions — exactly what a canteen assistant is.

**M07's metrics are where this lives permanently.** Log input tokens, output tokens, model name and estimated cost on every model call from day one. Without that, "why did the bill double" is unanswerable. With it, it is a query.

### Words to own

| Term | Plain meaning | Example |
|---|---|---|
| Input tokens | Everything you send | System, history, docs, message |
| Output tokens | Everything generated, including thinking | Priced higher than input |
| Price per million | How providers quote rates | Read it today, note the date |
| Cost per request | One complete call, both directions | ₹1.80 per turn |
| Cost per successful request | Including retries and failures | The honest unit |
| Cost per active user per month | What one user costs you monthly | Compare against revenue |
| Unit economics | Cost per user against revenue per user | Whether a business exists |
| Prompt caching | A discount on an unchanged prompt prefix | M10 chapter 6 |
| Semantic cache | Reusing an answer for a similar question | M05 and M10 |
| Rate limit | A cap per user or per minute | A cost control too |
| Billing alert | A notification before the damage | Set it on day one |
| Spend cap | A hard ceiling on the account | The real protection |

### Tradeoffs and gotchas

**Apply the levers in order of return, not of cleverness.** Trim the system prompt and the history. Cap max output tokens. Cache repeated questions. Route easy requests to a cheap model. Only then redesign. The first three are an afternoon and routinely cut most of the cost; teams reach for the last two first because they are more interesting.

**Sometimes the right answer is not to use a model at all.** "Which category is this ticket" can be a small classifier trained once, as chapter 4 argued, at effectively zero per request. "What is today's menu" is a database query. Before optimising a call, ask whether it needs to exist.

**Latency and cost move together.** Shorter prompts and capped outputs are both cheaper and faster, so this work pays twice — once on the bill and once in M05's latency numbers.

**A free tier is not a cost model.** Free credits tell you nothing about what the feature costs at a thousand users. Do the arithmetic at your target scale on day one, while the design is still cheap to change.

**Currency matters.** Most providers price and bill in dollars; your revenue is in rupees. An exchange-rate move raises your costs and not your prices, so keep a margin rather than budgeting at the day's rate.

**Every number here expires, including the structure.** Prices change, caching discounts change, and token accounting for reasoning models has changed more than once. Put a date on your spreadsheet and a reminder to rebuild it.

### Verify it

**Red flag, the headline one: a confidently quoted price.** Any figure per million tokens, from any AI, is a rumour. Open the pricing page, read it, write the date. An AI that gives you a price without telling you to verify it is itself the warning sign.

**Red flag: an estimate with no output-token line, or no thinking-token line.** Output is the dearer half and thinking tokens are invisible. An estimate missing either is not an estimate.

**Red flag: no multiplier for turns, retries or agent steps.** Ask: how many model calls does one user action make, and how many tokens does each carry? For an agent the answer is rarely one.

**Red flag: a cost per request with no revenue comparison.** "Five paise per request" sounds free until you multiply it by eight turns, three conversations and 1,500 users and set it beside ₹49 a month.

**Red flag: "we are on the free tier so it costs nothing."** Ask what it would cost at a thousand users. If nobody has computed it, nobody knows whether the product can exist.

**Red flag: no spend cap and no billing alert.** Not an estimation error — an unbounded liability. M00 covered it; this is where it becomes real money.

And the measurement no estimate substitutes for: make ten real calls through your feature, read the input and output token counts the provider returns, and average them. Those two numbers, times today's published prices, are your cost per request. Everything else is multiplication.

## Walkthrough
1. Pick the one AI feature in your product that will be used most. Write down who triggers it and what one use of it means.
2. Count the token buckets for a single call: system prompt, retrieved context, conversation history, user message, and expected output. Measure them with a real tokenizer rather than estimating.
3. Make ten real calls through your feature and read the input and output token counts the provider returns. Average them and compare against your estimate from step 2. Use the measured numbers from here on.
4. Open your provider's own pricing page. Write down the input and output price per million for your exact model, and write today's date next to them.
5. Compute cost per request as a ledger with one line per bucket, like the tables in this chapter. Show every line; do not collapse it into one number.
6. Now apply the multipliers in order: turns per conversation, retries, agent steps if any. Write each multiplication as its own line so you can see which one hurts.
7. Multiply out to a month: requests per active user per month, then your target number of active users. Write the monthly rupee figure.
8. Write down your revenue per active user per month next to it. React honestly to the two numbers side by side.
9. Apply the levers one at a time and recompute after each: shorten the system prompt, cut the history, cap max output tokens, cache repeated questions, route easy requests to the cheap model. Record the monthly figure after each step.
10. Set a hard spend cap and a billing alert on your provider account at a fraction of it. Screenshot both.
11. Add logging of input tokens, output tokens, model name and estimated cost to every model call in your code, so next month's question has an answer.

## Talk to AI
```prompt
I need an honest cost model for one AI feature in my startup, and I need you to follow my rule about prices exactly.

RULE: do NOT give me any price per million tokens, in any currency. For every price I need, tell me which provider page to open and what to look for. State plainly that any pricing you remember may be out of date and must not go into my budget.

My feature: [describe it - what it does, who triggers it, how often]
Model I plan to use: [exact model string]
Is it a reasoning model? [yes/no/unsure]
My context structure, with rough token counts if I have them: [system prompt / retrieved context / conversation history / user message]
Typical turns per conversation: [X]
Conversations per active user per month: [X]
Target active users: [X]
My revenue per active user per month, in rupees: [X, or 0 if free]
Is the feature available to non-paying users? [yes/no]

Do all of this:

1. Build me a cost ledger template with one line per token bucket, as a table with columns for item, tokens and cost. Leave the rates blank for me to fill in from the pricing page. Include a line for thinking tokens and tell me whether my model has them.

2. Tell me exactly how to measure my real token counts rather than estimate them - what the API response returns and where to find it.

3. List every multiplier that applies to MY feature specifically - turns, retries, agent steps, anything else - and tell me roughly what each one does to the total. Be explicit about any multiplier I seem to have forgotten.

4. Show me the full chain from one request to a monthly rupee figure as a sequence of multiplication lines, each labelled, so I can see which line hurts most.

5. Compare the monthly figure against my revenue and tell me bluntly whether this feature can exist at my price point. If it cannot, say so.

6. Give me the levers in order of return for MY feature, with an estimate of what each one saves. Tell me which ones change what a user would notice and which ones do not.

7. Tell me whether any part of this feature should not use a language model at all, and what should do it instead.

8. Finally, tell me what spend cap and billing alert I should set, and warn me about the specific runaway failure modes - retry loops, agent loops, abusive free users - that produce a shocking bill overnight.
```

## Mission

**Objective.** Produce a dated, line-by-line cost model for your own startup's main AI feature — cost per request and cost per month at your target scale, with the arithmetic shown — compare it against your revenue per user, and cut it with the levers until the feature can actually exist.

**Steps.**
1. **Frame** — name the one feature, who triggers it, and what one use of it means. One feature; the others get the same treatment later with the same spreadsheet.
2. **Measure, do not estimate** — make at least ten real calls through your feature and read the input and output token counts the provider returns. Average them. As a group of three, each person makes ten calls with different realistic inputs so your average is not one person's phrasing.
3. **Read the real prices** — open your provider's own pricing page, record input and output price per million for your exact model string, and write today's date beside them. No price in this mission may come from an AI.
4. **Build the ledger** — one line per token bucket, as a table, with a line for thinking tokens if your model has them. Show every line.
5. **Multiply out** — turns per conversation, retries, agent steps, requests per user per month, target users. One labelled line per multiplication, so the painful one is visible.
6. **Compare against revenue** — write your revenue per active user per month beside the cost per active user per month. Write one honest sentence about what the two numbers mean for your product.
7. **Apply the levers in order** — shorten the system prompt, trim the history, cap max output tokens, cache repeated questions, route easy requests to a cheap model. Recompute the monthly figure after each one and record all five numbers.
8. **Check what you broke** — rerun five real inputs through the trimmed version and confirm the output is still acceptable. Cheaper and worse is not a win; say so if that is what you got.
9. **Set the guardrails** — a hard spend cap and a billing alert on the provider account, plus a per-user rate limit if non-paying users can trigger the feature. Screenshot all of them.
10. **Instrument it** — add logging of input tokens, output tokens, model name and estimated cost to every model call, so next month's bill is explainable.
11. **Document** — one decision-log entry with the ledger, the date, the prices you read, and the trigger for rebuilding the model.

**Upload.**
- Your cost ledger, every line visible → spreadsheet or screenshot
- The provider pricing page you read, with the date visible → screenshot
- Your ten real calls with the token counts the API returned → file or screenshot
- Your multiplication chain from one request to a monthly rupee figure, labelled line by line → text or screenshot
- Cost per active user per month beside revenue per active user per month → one screenshot with both
- The monthly figure after each of the five levers, as five numbers → text
- Five real outputs from the trimmed version, so quality can be judged → file
- Your spend cap and billing alert → screenshots
- Your per-user rate limit, if applicable → screenshot or code reference
- Your decision-log entry → file in your repo
- Reflection, 150 words max → which multiplier surprised you most, and one price or limit an AI told you that turned out to be stale

**Self-check.** Tick every box before you submit.
- [ ] My token counts are measured from real API responses, not estimated
- [ ] Every price in my ledger came from the provider's own page, with the date I read it
- [ ] No price or limit in my mission came from an AI
- [ ] My ledger has a separate line for output tokens, priced higher than input
- [ ] If my model has thinking tokens, they are a line in my ledger
- [ ] My multiplication chain shows turns, retries and agent steps as separate labelled lines
- [ ] I have a cost per active user per month, not only a cost per request
- [ ] Revenue per active user per month appears next to it in the same place
- [ ] I recomputed the total after each of the five levers and kept all five numbers
- [ ] I re-ran real inputs after trimming and judged the output quality myself
- [ ] A hard spend cap and a billing alert are set on the account right now
- [ ] Non-paying users cannot trigger unlimited calls
- [ ] Every model call in my code logs input tokens, output tokens, model and cost
- [ ] My decision log has a date and a trigger for rebuilding the cost model

**Stretch.** Model the worst case rather than the average: a retry loop that fires five times, an agent that takes twelve steps instead of four, and your most talkative user making forty requests in an hour. Write 100 words on which of these your spend cap would actually stop, and which would simply exhaust it.

## Stuck? Ask AI
```prompt
I do not know how to turn token counts into a monthly cost. Walk me through the chain one multiplication at a time, using round placeholder rates that you clearly label as invented. Stop after each step and ask me for my own number before continuing. Do not give me any real price.
```

```prompt
My AI feature costs far more than I expected. Here is my prompt structure and my token counts: [paste them]. Do not rewrite my code. Instead, rank every contributor to the cost from largest to smallest, and for each one tell me the cheapest change that would reduce it and whether a user would notice.
```

```prompt
Explain the specific ways a student team ends up with a shocking AI bill overnight - retry loops, agent loops, abusive free usage, and anything else. For each one, tell me what guardrail stops it, and whether a spend cap alone would have been enough.
```

## Quiz

### Q1 · L2
Which part of a request is usually priced highest?
- (a) Input tokens, because there are more of them
- (b) Output tokens, including any hidden thinking tokens ✅
- (c) The system prompt specifically
- (d) Retrieved context
> Providers charge more per output token, and reasoning models bill thinking tokens you never see. An estimate that counts only visible output can be wrong by a large multiple.

### Q2 · L3
Your feature costs ₹1.80 per turn. Why is that number alone not enough to decide anything?
- (a) Because prices change frequently
- (b) Because it must be multiplied by turns, conversations and users, then set against revenue per user ✅
- (c) Because input and output are priced differently
- (d) Because free tiers do not charge per turn
> Under two rupees looks harmless until the multipliers run. The decision is made by cost per active user per month against revenue per active user per month.

### Q3 · L4
Which lever should you usually apply first to a costly chat feature?
- (a) Redesign the feature to remove the AI entirely
- (b) Trim the system prompt and the conversation history, and cap max output tokens ✅
- (c) Negotiate a discount with the provider
- (d) Switch to a reasoning model for better answers
> The cheapest, fastest levers come first and often cut most of the cost in an afternoon. Redesign is a real option but it is the last one, not the first.

### Q4 · L4
An AI gives you a specific price per million tokens for your model and a monthly estimate based on it. What is the correct response?
- (a) Use it; the arithmetic is what matters, not the source
- (b) Keep the arithmetic, discard the price, and read the real one on the provider's page with today's date ✅
- (c) Halve the price to be safe
- (d) Ask the AI to double-check its own number
> The method is sound and the number is a rumour from before its cutoff. Asking the same AI to verify itself produces another rumour, not a source.

### Q5 · L5
Your agent reads a document, thinks, calls a tool, reads the result and answers. One user action, four model calls, context growing each time. What does this mean for your cost model?
- (a) Nothing; it is still one request from the user's point of view
- (b) Multiply by the number of steps, and account for the context growing on each step ✅
- (c) It is cheaper, because each individual call is shorter
- (d) Agent steps are not billed separately
> Agent loops are where costs surprise teams most, because each step re-sends an ever-larger context. The honest unit is cost per completed user action, not per model call.

## Go deeper
- [Anthropic: pricing — read the live page, never a remembered number](https://www.anthropic.com/pricing)
- [OpenAI: API pricing and token usage in responses](https://platform.openai.com/docs/pricing)
