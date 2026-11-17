---
id: m10-06-prompt-caching
module: m10-building-with-ai
slug: prompt-caching
title: "Prompt Caching and Why It Changes Your Costs"
order: 6
duration_min: 15
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

A team's canteen assistant runs out of free-tier quota by two every afternoon: 3,700 tokens of rules, tools, examples, menu and FAQ, re-sent and re-processed on every one of three thousand conversations a day.

So they cut the system prompt. Quality drops and the quota still runs out, now at three. What they had not noticed is the first line of their template — "Today is 16 November 2026, 14:32" — which changes every minute, so the first tokens differ every call and nothing after them can be reused. One helpful line cost them the entire saving.

### The idea

**Providers can store the processed form of a prompt prefix. If the first tokens of this call are identical to the last one, you pay a fraction for them and they process almost instantly — but only if they are identical, and only at the front.**

Cost drops a lot, often most of the bill for a prompt-heavy product. Latency drops too, because the model no longer re-reads 3,700 tokens before the first word, and on slow 4G time to first word is what the user experiences.

### The analogy

Every canteen serving two hundred plates makes a base masala in the morning: onion, tomato, ginger, garlic, spices, one big vessel. An order adds the vegetable at the end and is on the plate in ninety seconds. Making fresh masala per plate is what an uncached prompt does.

**Where the analogy breaks, twice.** The vessel keeps all day; a prompt cache expires after minutes of inactivity, so a call an hour later pays to build it again. And the vessel costs *more* than one plate's worth — a cache write is billed above a normal call, so a product with scattered conversations ends up worse off.

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

Matching runs by exact prefix from the first token forward, so the moment something differs everything after it is new work — including parts that did not change. This is chapter five's layout with a bill attached: rules, tools and examples at the front, things shared by many calls in the middle, this question's chunks and history at the back.

### Where it lives

Caching is a provider feature you switch on by marking where the cacheable prefix ends, so in your code it is a property of chapter five's single `build_context()`. M05 covered caching as a general systems idea; this is the same idea at the model boundary. Five small mistakes destroy it:

- A timestamp or date at the top — move it to the final user turn
- A per-user system prompt: "helping Priya, roll 4121" is a new prefix per student
- A tool list built from a set, so its order changes between runs
- Shuffled chunks sitting *above* the stable block
- Editing the rules file mid-session — correct behaviour, new prefix each time

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

**Prompt caching is not answer caching.** Prompt caching stores the processed *input* and the model still generates fresh output, so there is no staleness risk. Answer caching stores the *output*, is cheaper again, and carries M05's staleness problem — caching "which dishes are sold out" for an hour is a bug.

**The numbers move.** Discounts, TTLs and the minimum prefix length differ by provider and change between releases — below that minimum nothing caches and you get no warning. Learn the mechanism here and read the current pricing page.

**Low-traffic products lose.** Cache writes cost more than normal calls, so twelve conversations over twelve hours cost more with caching on.

**Never put one user's data in a shared prefix.** The cache is scoped to your account, not to a user, so a prefix holding Priya's order history is one every other student's call matches against. And caching changes cost and latency only — a wrong answer just gets wrong faster.

### Verify it

- **Read the token counts the provider returns.** Most report cache-read and cache-write separately. Log them; on a warm product cache-read should dominate.
- **Diff two consecutive prompts byte for byte.** What shows up is almost always a timestamp, a user name or a reordered tool list.
- **Measure time to first token before and after**, twenty calls each way on a throttled connection.
- **Price it over a full day, not one call**, including cache writes and cold starts.
- **Confirm cache and freshness are separate.** Ask about a dish marked sold out five minutes ago; a stale answer means you have answer caching somewhere, because prompt caching cannot cause it.

Red flags: a date, time, user name or session id in the system prompt; a tool list built from an unordered collection; no cache-read or cache-write numbers in your logs; rules shortened to cut costs without checking the layout; a staleness bug blamed on the prompt cache; one user's data inside a shared prefix.

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
