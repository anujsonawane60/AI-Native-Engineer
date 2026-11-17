---
id: m14-07-inference-serving
module: m14-deeper-ml-agents
slug: inference-gpus-serving
title: "Inference, GPUs and Serving Providers"
order: 7
duration_min: 15
is_free: false
mission: false
volatility: high
last_reviewed: 2026-11-16
---

## Key learnings
- Split a response into prefill and decode, and predict which part your feature pays for
- Explain why the key-value cache, not the weights, limits how many users you can serve at once
- Compare four serving shapes and find your own break-even in requests per day
- Read a throughput benchmark without mistaking it for a latency promise
- Catch a GPU being rented for a load nobody has measured

## Lesson
### The pain

M09 chapter 7 explained why a GPU is faster and separated "too big to fit" from "too slow to use"; chapter 6 ran a model on your laptop. New here is what decides your bill: the serving machine at 1:10 pm.

One team reads that a GPU serves "1,800 tokens per second" and rents one for ₹28,000 a month. Their real load is 300 requests a day, so they pay for an idle machine at sixty times the per-token price. Another goes serverless, where cold requests spend 40 seconds loading weights and p95 is a minute. Both numbers were true and irrelevant: what matters is cost and latency per request at *your* concurrency.

### The idea

**Generating a response is two different jobs with different bottlenecks, and almost everything about serving follows from that.**

**Prefill** reads your whole prompt at once, in parallel, so it is compute-heavy and sets your **time to first token**. **Decode** produces one token at a time, each needing the model's weights read again, so it is limited by memory bandwidth. That is why output tokens are slower and dearer, and why the key technique is sharing one pass over the weights.

### The analogy

The canteen at 1:10 pm. Prefill is reading a table's order slip: one glance, however long the list. Decode is cooking one dish at a time on a single tawa, where the walk to the masala shelf dominates. If six tables want dosas, fetch the masala once and make six — **batching** — and let new orders join the trip in progress: **continuous batching**.

**Where it breaks.** The cook's shelf has infinite room; the machine's memory does not. Every conversation holds its own context — the **KV cache** — so concurrency is capped by memory, and requests queue once it fills.

### Picture it

```
  ONE REQUEST
  ├─ PREFILL   whole prompt at once, parallel, compute-bound
  │            ████                       ──▶ time to first token
  │
  └─ DECODE    one token at a time, memory-bandwidth-bound
               ░ ░ ░ ░ ░ ░ ░ ░ ░ ░ ░ ░   ──▶ tokens per second

  FOUR REQUESTS, NAIVE          FOUR REQUESTS, CONTINUOUS BATCH
  ████░░░░░░                    ████░░░░░░
       ████░░░░░░               ████░░░░░░   same weight reads
            ████░░░░░░          ████░░░░░░   shared across all
                 ████░░░░░░     ████░░░░░░
  ─────────────────────────▶    ──────────▶
  4× the time                   roughly 1× the time,
                                limited by KV-cache memory
```

Prefill is a parallel burst that sets the first-token wait while decode is a slow sequential trickle, and batching many requests into the same pass over the weights is what turns a single GPU into a server — until the key-value cache fills the memory.

### The four shapes, and where each wins

- **Hosted API, per token.** No operations, no idle cost, paid per token forever. The default for most student startups.
- **Serverless GPU, per second.** No idle cost, but a cold start of tens of seconds hits a real user.
- **Dedicated GPU, per hour.** Yours idle or not, so cost per request falls as utilisation rises; the only shape with predictable latency.
- **Your own hardware.** Chapter 6's path: no per-request cost, one machine's concurrency, every operational problem yours.


The break-even is arithmetic: monthly machine cost divided by monthly requests, against per-token cost per request. A ₹28,000 machine at 300 requests a day is ₹3.10 each; at 30,000 a day, ₹0.03. Only utilisation changed.

### Tools today

The open serving engines with continuous batching and paged key-value caches are vLLM and Hugging Face's Text Generation Inference; both serve a quantized open-weight model behind an OpenAI-shaped API, which keeps you portable. Serverless platforms and hourly rentals exist from several vendors at prices that move monthly. The cheapest honest path is a per-token hosted API until measured volume says otherwise, prototyping heavier work on a free notebook GPU. Every price and name here will be wrong within six months.

### Where it lives

- Chapter 6 — what you serve and at what precision, which sets how many conversations fit
- Chapter 14 — the gateway that makes switching shapes configuration, not a rewrite
- M05 chapters 4, 8 — queueing, rate limits
- M09 chapter 14, M17 chapter 6 — cost arithmetic, margin

### Words to own

| Term | Plain meaning | Example |
|---|---|---|
| Prefill | Processing the whole prompt in one parallel pass | Sets first-token time |
| Decode | Generating the answer one token at a time | Sets tokens per second |
| Memory bandwidth | How fast weights can be read from GPU memory | Decode's real limit |
| Batching | Serving several requests in one pass over the weights | Multiplies throughput |
| Continuous batching | New requests join the pass in progress | The one to insist on |
| KV cache | Per-conversation memory holding the context so far | Caps concurrency |
| Concurrency | How many requests are in flight at once | 12 at lunch |
| Throughput | Total tokens per second across all users | A server metric |
| Latency | What one user waits | A user metric |
| Cold start | Loading weights before the first request can run | 40 seconds |
| Utilisation | What fraction of a rented machine does work | 0.3% at 300/day |
| Break-even volume | Requests per day where renting beats per-token | Compute it |

### Tradeoffs and gotchas

**Throughput and latency are in opposition; benchmarks quote the flattering one.** "1,800 tokens per second" is an aggregate across simultaneous requests and says nothing about one student on 4G. Ask at what concurrency, and the p95 for a single request.

**Idle time is the expensive part of a rented GPU.** A quiet week costs what a busy one costs; under 20% utilisation is a decision to revisit. Cold starts belong in the p95, not a footnote.

### Verify it

The red flag here is the plainest in the module: a cost per request nobody measured.

1. **Demand cost per request in rupees for every option,** including idle time.
2. **Ask for the break-even volume as a number.** If the AI cannot state it, it did no arithmetic.
3. **Separate the two latency numbers,** at the concurrency you expect.
4. **Ask what happens at double your peak concurrency** — the answer should mention the KV cache, a queue and a timeout.
5. **Check for continuous batching by name,** and keep the API shape portable.

Red flags: a GPU rented before load was measured; a throughput benchmark sold as a latency promise; no utilisation figure; cold starts excluded from p95; spot instances with no fallback.

## Walkthrough
1. Measure your real load from your own logs: requests per day, requests in the busiest hour, and the maximum number in flight at once. Write all three down. Everything else depends on these.
2. Measure your real token shape: median and p95 input tokens, median and p95 output tokens. M09 chapter 14 taught you to read these from the API response.
3. Compute your current cost per request on your hosted API, in rupees, from those numbers and today's published price.
4. Find hourly prices for one small GPU from two providers. Compute the monthly cost of each and divide by your monthly requests. Put the result beside the figure from step 3.
5. Compute your break-even volume: the requests per day at which the rented machine's cost per request drops below the hosted API's. Write it down as a single number and compare it to your actual volume.
6. Now measure latency properly. Send one request and record time to first token and tokens per second. Then send ten simultaneously and record the same two numbers for the slowest of them.
7. Deploy a small quantized open-weight model on a free notebook GPU behind a serving engine that does continuous batching, and repeat step 6 against it. Record both sets side by side.
8. Test the limit: increase concurrency until latency degrades sharply, and record the number where it happens. Then watch GPU memory and confirm the key-value cache is what filled up.
9. Measure a cold start: leave your serverless or notebook deployment idle for your real gap between requests, then time one request end to end. Record where that figure would sit in your p95.
10. Write the decision as a table with four rows — hosted API, serverless GPU, dedicated GPU, own hardware — each with cost per request, time to first token, tokens per second at your concurrency, and one line of operational burden.
11. Record the choice, the break-even number, and the single measurement that would make you change your mind, in your decision log.

## Talk to AI
```prompt
CONTEXT:      I am a student on an AI-native engineering program. I direct AI
              and verify its output; I do not write code by hand. I already
              know why GPUs suit this work, how to compute cost per request
              from token counts, and how to measure latency on a phone, so do
              not re-teach those. My team of three runs [product in two
              sentences] with real users in India. Measured from our own logs:
              [number] requests per day, [number] in the busiest hour, up to
              [number] in flight at once. Token shape: input median [number]
              and p95 [number]; output median [number] and p95 [number]. We
              currently use [provider and model] at a cost per request of
              about ₹[number]. Our users are on budget Android phones on 4G.

GOAL:         Choose between a hosted per-token API, serverless GPU, a
              dedicated hourly GPU and our own hardware, using cost per
              request and latency at OUR concurrency rather than anybody's
              benchmark.

REQUIREMENTS: 1. Explain, for MY token shape, which part of my cost and my
                 latency is prefill and which is decode, and what that means
                 for where I should optimise first
              2. Compute cost per request in rupees for all four shapes at my
                 measured volume. For anything rented, include idle hours.
                 Show the arithmetic
              3. Give me my break-even volume as one number: requests per day
                 above which a dedicated GPU beats per-token pricing
              4. Tell me how much key-value cache memory one of my
                 conversations needs at my p95 input length, and therefore how
                 many simultaneous conversations fit on a small GPU
              5. Tell me what happens at double my peak concurrency, and what
                 the queue, timeout and rejection behaviour should be
              6. For serverless, tell me the cold-start cost for a model of
                 the size I would use, how often my traffic pattern would hit
                 it, and where it lands in my p95
              7. Tell me which open serving engine to use if I self-host, why
                 continuous batching matters for my load, and how to confirm
                 it is actually switched on
              8. Give me the four-row decision table to fill in, with cost per
                 request, time to first token, tokens per second at my
                 concurrency, and operational burden

CONSTRAINTS:  Free tier for experiments; a free notebook GPU is available. I
              can spend at most a few hundred rupees testing. Plain English
              for a second-language reader. Label every command READ-ONLY or
              CHANGES-THINGS. Keep my API shape portable so I can switch
              providers without rewriting my app. Treat every price, instance
              type and product name as stale and tell me to check it today.

NON-GOALS:    Do not quote a vendor's throughput benchmark as my latency. Do
              not recommend renting a GPU before the break-even number is
              computed. Do not propose Kubernetes or an autoscaling cluster.
              Do not re-explain what a GPU is. Do not propose spot instances
              for user-facing traffic without a tested fallback.

ACCEPTANCE:   - Cost per request in rupees exists for all four shapes,
                including idle time for rented options
              - A break-even requests-per-day number is stated
              - Time to first token and tokens per second are given
                separately, at my concurrency and not at one
              - A key-value cache estimate per conversation exists, and a
                resulting concurrency limit
              - Cold-start cost is quantified and placed in my p95
              - Queue, timeout and rejection behaviour are specified
              - Continuous batching is named, with a way to verify it is on

ASK:          First tell me, from my numbers alone, whether a dedicated GPU
              could possibly be cheaper than per-token pricing at my volume,
              and show the single division that settles it. Do not discuss any
              other option until you have.
```

## Stuck? Ask AI
```prompt
Explain prefill and decode using one concrete request: a 2,000-token prompt producing a 300-token answer. Tell me roughly where the time goes, why the output tokens are slower than the input tokens, and which of the two parts gets cheaper when many users are served at once. Then tell me which fix helps a long prompt and which fix helps a long answer, and why applying the wrong one changes nothing.
```

```prompt
A provider's page says their setup does 1,800 tokens per second on one GPU. Explain exactly what that number does and does not tell me about what one of my users will experience. Then give me the three questions I must ask to turn it into a latency prediction for my case, and tell me what measurement I should take myself instead.
```

```prompt
I want to self-host a quantized open-weight model on one small GPU. Walk me through how much memory the weights take, how much the key-value cache takes per simultaneous conversation at a 4,000-token context, and therefore how many users I can serve at once. Then tell me what should happen to request number N plus one, and what the queue, the timeout and the error response should look like.
```

## Quiz

### Q1 · L3
Why are output tokens usually slower and more expensive than input tokens?
- (a) Output tokens are longer on average
- (b) Decode generates one token at a time, re-reading the weights for each ✅
- (c) Input tokens are compressed before processing
- (d) Providers charge more for output as a business choice only
> Prefill processes the whole prompt in parallel in one burst. Decode is sequential and limited by memory bandwidth, so each output token costs a full pass over the weights.

### Q2 · L4
A team rents a ₹28,000-per-month GPU for a feature used 300 times a day. What is their cost per request?
- (a) Around ₹0.03
- (b) Around ₹0.30
- (c) Around ₹3.10 ✅
- (d) It cannot be computed without token counts
> ₹28,000 divided by roughly 9,000 requests a month is about ₹3.10. The bill is for the machine, not the work, so low utilisation is the entire cost problem.

### Q3 · L4
Your self-hosted model handles one user quickly and degrades sharply at twelve simultaneous users, with no error. What filled up?
- (a) The disk holding the model file
- (b) The key-value cache memory for the concurrent conversations ✅
- (c) The token budget on the provider's account
- (d) The CPU's instruction cache
> Weights are a fixed cost; each in-flight conversation adds its own context memory. When that fills, requests queue instead of failing, which looks like everything getting slower at once.

### Q4 · L4
Which serving shape makes a 40-second cold start a user-facing problem?
- (a) A hosted per-token API
- (b) Serverless GPU with bursty traffic ✅
- (c) A dedicated hourly GPU kept warm
- (d) A model running on your own laptop
> Serverless scales to zero, so the first request after an idle gap pays for loading the weights. With bursty traffic a meaningful share of real users hit that path, which is why it belongs in the p95.

### Q5 · L5
Your feature has long prompts and short answers, and users complain about the wait before text appears. Which fix is targeted?
- (a) Cap the maximum output length
- (b) Trim the prompt and cache its stable prefix ✅
- (c) Increase the batch size on the server
- (d) Switch to a larger GPU with more memory
> The complaint is time to first token, which is prefill, which is prompt length. Capping output and raising batch size address decode and throughput — real levers aimed at the wrong half of the request.

## Go deeper
- [vLLM: paged attention and continuous batching explained](https://docs.vllm.ai/en/latest/)
- [Hugging Face: LLM inference optimisation concepts](https://huggingface.co/docs/transformers/en/llm_optims)
