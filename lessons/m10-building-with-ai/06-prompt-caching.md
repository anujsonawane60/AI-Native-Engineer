---
id: m10-06-prompt-caching
module: m10-building-with-ai
slug: prompt-caching
title: "Prompt Caching and Why It Changes Your Costs"
order: 6
duration_min: 25
is_free: false
mission: false
volatility: medium
last_reviewed: 2026-11-16
---

## Key learnings
- Explain what a cached prompt prefix is and what makes it match
- Order your context so the stable part sits in front and stays identical
- Name the small mistakes that silently destroy every cache hit
- Tell prompt caching apart from caching the answer itself
- Read cache-read and cache-write counts to prove it is working

## Lesson

### The pain

A team's canteen assistant runs out of free-tier quota by two in the afternoon, every day. They look at their call and find 1,200 tokens of rules, tool definitions and examples, plus 2,500 tokens of retrieved menu and FAQ text, on top of a forty-token question. Three thousand conversations a day, several calls each, and the same 3,700 tokens re-sent and re-processed every single time.

Their fix is to cut the system prompt. They delete the Hinglish examples and two of the refusal rules, the quality drops noticeably, and the quota still runs out — now at three in the afternoon.

What they had not noticed is the line at the top of their prompt. Their template begins "Today is 16 November 2026, 14:32." That timestamp changes every minute, which means the very first tokens of every call differ from the last call, which means nothing after it can ever be reused. One helpful line was costing them the entire saving.

### The idea

**Providers can store the processed form of a prompt prefix. If the first tokens of this call are identical to the last one, you pay a small fraction for them and they process almost instantly — but only if they are identical, and only if they are at the front.**

Two consequences follow, and the second one surprises people more than the first. Cost drops a lot, often by most of the bill for a prompt-heavy product. And *latency* drops, because the model is no longer re-reading 3,700 tokens before it writes the first word. On a slow Indian 4G connection, time to first word is what a user actually experiences.

### The analogy

Every canteen that serves two hundred plates at lunch makes a base masala in the morning: onion, tomato, ginger, garlic, the standard spices, one big vessel. A specific order adds the vegetable at the end and it is on the plate in ninety seconds.

Making fresh base masala for each individual plate is what an uncached prompt does. Same ingredients, same work, repeated two hundred times, while the queue grows.

**Where the analogy breaks in two places.** The vessel of masala keeps all day whether anyone orders or not. A prompt cache expires after a short period of inactivity — minutes, typically — so if your next call comes an hour later you pay to build it again. And making the vessel costs *more* than making one plate's worth: writing to the cache is usually billed above a normal call, so a product with a handful of scattered conversations can genuinely end up worse off. Caching pays when calls are frequent and share a prefix.

### Picture it

```
  CALL 1 (cold)
  ┌──────────────────────────┬──────────────┐
  │ rules + tools + examples │ docs + Q     │
  │ 1,200 tok  CACHE WRITE   │ 2,540 normal │
  └──────────────────────────┴──────────────┘

  CALL 2..N (warm, same prefix)
  ┌──────────────────────────┬──────────────┐
  │ IDENTICAL bytes          │ docs + Q     │
  │ 1,200 tok  CACHE READ    │ 2,540 normal │
  │ pay a fraction, fast     │ full price   │
  └──────────────────────────┴──────────────┘

  CALL 2' (prefix changed by one token: a timestamp)
  ┌──────────────────────────┬──────────────┐
  │ CACHE MISS - full price and full wait   │
  └─────────────────────────────────────────┘
```

The first call pays to build the cache; later calls with a byte-identical prefix read it cheaply. One changed token anywhere in the prefix discards all of it.

### What makes a prefix match

Matching is by exact prefix, from the first token forward. The provider walks your call from the beginning and reuses as much as is identical. The moment something differs, everything from that point on is new work — including the parts after it that have not changed.

So this is a layout problem, and it is the same layout chapter five already argued for, now with a bill attached.

**Front:** the system prompt, tool definitions, few-shot examples. Identical for every user, every call, forever.

**Middle:** anything shared by many calls but not all. A product-wide FAQ, the day's menu, a policy document. Still cacheable, just with a shorter useful life.

**Back:** retrieved chunks for this question, conversation history, the question itself. Never the same twice, never cached, and that is fine.

### The small mistakes that kill it

Each of these has cost a real team its entire saving.

**A timestamp or a date at the top.** Put the date in the user turn at the end, never in the system prompt. If a rule genuinely needs today's date, the rule can refer to a date supplied later.

**A per-user system prompt.** "You are helping Priya, roll number 4121." That is a different prefix for every student, so your cache hit rate is zero across users. Put the student's name in the final turn and keep one canonical prefix for everybody.

**Tool definitions in a different order.** If your code builds the tool list from a dictionary or a set, the order can change between runs. Sort it. Deterministically.

**Shuffled retrieved chunks above the stable block.** If anything variable sits above your rules, nothing below it caches.

**Editing the rules file during a session.** Entirely correct behaviour that you should expect: every edit is a new prefix and a new cache write. Expect a cost blip after each change, not a mystery.

### Prompt caching is not answer caching

These get confused constantly and they are different techniques with different failure modes.

**Prompt caching** stores the *processed input*. The model still generates a fresh answer every time. There is no staleness risk, because nothing about the answer is reused.

**Answer caching** stores the *output*: "what are the canteen timings" asked twice returns the stored reply. That is far cheaper again — zero model calls — and it carries the classic staleness problem M05 taught you, which is why it needs a TTL and a way to invalidate when the underlying data changes. Caching "which dishes are sold out" for an hour is a bug, not an optimisation.

Both are worth having. Decide them separately, and never explain a staleness bug by pointing at the prompt cache.

### Where it lives

Caching is a provider feature you enable in the API call, usually by marking where the cacheable prefix ends. In your code it is a property of the single `build_context()` function from chapter five — which is the argument for having exactly one such function.

M05 covered caching, rate limiting and cost control as general systems ideas; this is the same idea applied one layer up, at the model boundary. M09's token economics is how you price the before and after. And chapter two's build-time trick combines well with this: a prompt and schema designed once by a reasoning model, then served as a cached prefix to a cheap model, is close to the cheapest reliable AI feature you can build on a free tier.

### Words to own

| Term | Plain meaning | Example |
|---|---|---|
| Prompt caching | Reusing the processed form of a prefix | Rules block on call 2 |
| Prefix | Tokens from the start, in order | Rules, tools, examples |
| Cache write | The first call, which stores the prefix | Usually priced above normal |
| Cache read | A later call reusing it | Priced well below normal |
| Cache hit | The prefix matched | Fast and cheap |
| Cache miss | It did not match | Full price, full wait |
| TTL | How long an unused entry survives | Minutes, not hours |
| Warm / cold | Whether the entry exists right now | First call of the day is cold |
| Time to first token | Wait before the first visible word | What users feel on 4G |
| Answer caching | Reusing the reply itself | Different thing, needs a TTL |

### Tradeoffs and gotchas

**The numbers move.** Discounts, minimum cacheable lengths and TTLs differ by provider and change between releases. Learn the mechanism here and read the current pricing page before you design around a specific figure. Anything you memorise today will be wrong within a year.

**There is usually a minimum prefix length.** Below it, nothing is cached at all and you get no warning. A 200-token system prompt may simply not qualify, which is one of the rare arguments for a *longer* stable block — if the extra content earns its place on quality grounds too.

**Low-traffic products lose.** Cache writes cost more than normal calls. If your demo gets twelve conversations a day spread over twelve hours, you will pay more with caching on. Measure before and after rather than assuming.

**Caching never improves correctness.** It changes cost and latency and nothing else. A wrong answer gets wrong faster and cheaper.

**Do not put one user's data in a shared prefix.** The mechanism is scoped to your account, not to a user, so a prefix containing Priya's order history is a prefix every other student's call will match against. Per-user data belongs at the end, with the question.

**Shortening your rules is the wrong first move.** The team in the opening story deleted quality to save money that caching would have saved for free. Fix the layout first, measure, and only then decide whether the content is too long.

### Verify it

**Read the token counts the provider returns.** Most responses report cache-read and cache-write token counts separately. Log them. On a warm product, cache-read should dominate within minutes of starting. If cache-write is high on every call, your prefix is not stable and the numbers will tell you so immediately.

**Diff two consecutive prompts byte for byte.** Print the exact prefix on two calls and compare them with a diff. What shows up is almost always a timestamp, a user name, a reordered tool list or a randomly ordered set.

**Measure time to first token, before and after.** Twenty calls each way, on a throttled connection. This is the number your users experience, and it is the most persuasive evidence that the work was worthwhile.

**Price it honestly over a full day, not one call.** Include the cache writes and the cold starts. A product with bursty traffic has more cold starts than you expect.

**Confirm your cache and your freshness are separate.** Ask the model something that changed five minutes ago — a dish just marked sold out. If the answer is stale, find out whether you also enabled answer caching somewhere, because prompt caching cannot cause that.

Red flags: a date, time, user name or session id anywhere in the system prompt; a tool list built from an unordered collection; no cache-read or cache-write numbers in your logs; a team that shortened its rules to cut costs without checking the layout; a staleness bug blamed on the prompt cache; and one user's private data inside a prefix shared across users.

## Walkthrough
1. Print the full prompt your product sends, for two consecutive calls, and save both to files.
2. Diff the two files. Note every difference. Anything differing above the question is a cache killer and your list of fixes.
3. Find the first byte at which they differ. Everything from that point on is being reprocessed on every call — that is your current effective prefix length.
4. Move every variable thing below the stable block: dates, user names, session ids, retrieved chunks. The date goes into the final user turn.
5. Sort your tool definitions into a fixed deterministic order and confirm the order is the same across two process restarts, not just two calls.
6. Enable prompt caching in the API call and mark where the cacheable prefix ends — the boundary between your stable block and your variable block.
7. Make twenty calls in quick succession. Log the cache-read and cache-write token counts the provider returns for each one.
8. Confirm call one is a write and calls two to twenty are reads. If not, diff the prompts again; something is still varying.
9. Measure time to first token for twenty cached and twenty uncached calls on a throttled connection, and write down both averages.
10. Price a realistic day — your expected call volume, including cold starts after idle gaps — before and after, in rupees, using M09's method. Record both numbers in your decision log.

## Talk to AI
```prompt
Help me restructure one AI call so its prompt prefix is cacheable, and tell me honestly whether caching is even worth it for my traffic.

MY PROVIDER AND MODEL: [name them]
MY CALL VOLUME: [calls per day, and how they are spread - bursty at lunch, or even?]
MY CURRENT PROMPT, in the exact order I send it, with rough token counts per section:
1. [section]
2. [section]
3. [...]

ANYTHING THAT VARIES BETWEEN CALLS: [list it - dates, user names, retrieved text, history]

Do all of the following:

1. Identify every element in my prompt that varies between calls, including ones I may not have listed, and say where each one must move to.
2. Give me the exact section order for a cacheable prefix: what goes in the stable block, what goes in a semi-stable middle, what must stay at the end.
3. Tell me where to place the cache boundary and why there rather than one section earlier or later.
4. Check my stable block against the current minimum cacheable prefix length for my provider, and tell me if it is too short to qualify.
5. Estimate my daily cost before and after, in rupees, including cache writes and cold starts after idle periods. Show the arithmetic. State the assumptions you had to make.
6. Tell me plainly whether caching is worth enabling at my traffic level, and at what call volume the answer would change.
7. Explain the difference between what you have just designed and caching the ANSWER itself, and tell me which of my calls - if any - are safe to answer-cache and with what TTL.
8. List the mistakes most likely to silently break this later, and what I should log to notice within an hour rather than at the end of the month.
```

## Stuck? Ask AI
```prompt
Explain prompt caching to someone who understands HTTP caching but has never used an LLM API. What exactly is stored, what has to match for a hit, who stores it, and how long does it live? Then explain why one changed token at the top of my prompt destroys the whole saving.
```

```prompt
Explain the difference between prompt caching and caching the model's answer. For each: what is stored, what it saves, what can go stale, and what invalidation I need. Then give me three examples of questions in a canteen app that are safe to answer-cache and three that absolutely are not, with reasons.
```

```prompt
My cache hit rate is zero and I cannot see why. Here are two consecutive prompts I sent: [paste both]. Find every difference, tell me which one is breaking the prefix match, and tell me exactly where each varying element should move to instead.
```

## Quiz

### Q1 · L2
What does prompt caching actually store?
- (a) The model's previous answers
- (b) The processed form of an identical prompt prefix ✅
- (c) Your retrieved documents, embedded
- (d) A summary of the conversation so far
> Prompt caching reuses input processing. A fresh answer is still generated every call, so there is no staleness risk from it.

### Q2 · L3
Your system prompt begins "Today is 16 November 2026, 14:32." What is the effect on caching?
- (a) Only that line is re-processed each call
- (b) The cache still hits because the rest is unchanged
- (c) Nothing after it can be reused, so every call is a miss ✅
- (d) The cache TTL is shortened to one minute
> Matching is by exact prefix from the first token. A difference at the top discards everything below it, including the parts that did not change.

### Q3 · L4
Why can enabling prompt caching make a low-traffic demo more expensive?
- (a) Cached tokens are billed twice
- (b) Cache writes cost more than ordinary calls, and scattered traffic means mostly cold starts ✅
- (c) The provider charges a monthly cache fee
- (d) Caching forces a larger model
> You pay a premium to build the entry and only recover it on subsequent warm hits. Twelve conversations spread across twelve hours mostly pays the premium.

### Q4 · L4
A dish was marked sold out five minutes ago and the bot still says it is available. Can prompt caching be the cause?
- (a) Yes, the cached prefix contains the old menu
- (b) No — prompt caching reuses input processing, not answers; look for answer caching or stale retrieval ✅
- (c) Yes, if the TTL is longer than five minutes
- (d) Only if the menu is in the system prompt
> A fresh answer is generated every call. Staleness comes from answer caching, or from retrieval and context assembly handing the model old data.

### Q5 · L5
Your system prompt says "You are helping Priya, roll number 4121." What are the two separate problems?
- (a) It is too long, and it wastes tokens
- (b) Zero cross-user cache hits, and one user's data sits in a prefix other users match against ✅
- (c) It breaks structured outputs, and it raises latency
- (d) It cannot be translated, and it reveals the prompt
> Per-user text in the prefix means a different prefix per student, so nothing is shared. It also places private data where the shared-prefix mechanism can reach it. Both are fixed by moving the name to the final turn.

## Go deeper
- [Anthropic: Prompt caching](https://docs.anthropic.com/en/docs/build-with-claude/prompt-caching)
- [OpenAI: Prompt caching](https://platform.openai.com/docs/guides/prompt-caching)
