---
id: m14-07-inference-serving
module: m14-deeper-ml-agents
slug: inference-gpus-serving
title: "Inference, GPUs and Serving Providers"
order: 7
duration_min: 30
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

M09 chapter 7 explained why a GPU is faster for this work and distinguished a model being too big to fit from one being too slow to use. M09 chapter 13 made you measure latency on a phone. Chapter 6 of this module got a model running on your own laptop. What none of them covers is the part that decides your bill: what happens on the serving machine when forty people use your feature at 1:10 pm.

The failure has two symmetric forms. A team reads that a GPU serves "1,800 tokens per second" and rents one for ₹28,000 a month. Their actual load is 300 requests a day, which the GPU finishes in under four minutes of work, so they are paying for 24 hours of an idle machine and their cost per request is roughly sixty times what a per-token API would have charged.

The other team goes serverless to avoid exactly that, and discovers that every cold request spends 40 seconds loading 4 GB of weights before it does any work. Their p95 latency is a minute. They had measured the model's speed and never measured the system's.

Both teams measured something true and irrelevant. The number that matters is cost and latency per request at *your* concurrency, and it cannot be read off anybody's benchmark page.

### The idea

**Generating a response is two different jobs with different bottlenecks, and almost everything about serving follows from that.**

**Prefill** reads your whole prompt at once. Every token can be processed in parallel, so this is a compute-heavy burst, and its duration sets your **time to first token**. **Decode** then produces the answer one token at a time, each one needing the whole model's weights read from memory again, so it is limited by memory bandwidth rather than arithmetic. That is why output tokens are slower than input tokens, why they usually cost more, and why the single most valuable serving technique is sharing one pass over the weights across many users at once.

### The analogy

The canteen at 1:10 pm. Prefill is reading the whole order slip for a table of six: you can scan every line at a glance, and it takes one moment however long the list is.

Decode is cooking the dishes one at a time on a single tawa. Each dish needs the cook to walk to the shelf, fetch the masala, come back. The walk dominates, not the cooking. Which means the clever move is obvious: if six tables all want dosas, fetch the masala once and make six dosas on the same trip. The sixth table waits no longer than the first, and the kitchen's output multiplies.

That is **batching**, and the refinement that matters is that new orders should join the current trip rather than waiting for it to finish — **continuous batching**. A kitchen that waits until all six dosas are done before accepting a seventh order is wasting the tawa.

**Where the analogy breaks.** The cook's shelf has infinite room, but the serving machine's memory does not. Every conversation in progress occupies memory for its own context — the **KV cache** — so the number of simultaneous diners is capped by memory, not by the cook's hands. Run out and new requests queue, and M05's lesson arrives: a queue makes everybody's wait longer at once.

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

**A hosted API, priced per token.** Zero operations, instant, no idle cost, and you pay for every token forever. This is the right default for almost every student startup, and M09 chapter 14 already taught you to compute its cost per request.

**Serverless GPU, priced per second of compute.** You supply the model, the platform starts a worker on demand. No idle cost, but a cold start pays for loading weights — tens of seconds for a multi-gigabyte model — and that lands on a real user. Good for bursty, latency-tolerant work: batch jobs, image generation, overnight processing.

**A dedicated GPU, priced per hour.** You rent a machine and it is yours, idle or not. The cost per request falls as utilisation rises, so this only beats per-token pricing above a volume you must calculate. Also the only shape that gives you predictable latency, which matters if your feature is interactive.

**Your own hardware.** Chapter 6's path. No per-request cost at all, bounded by one machine's concurrency, and you own every operational problem.

The break-even is simple arithmetic you should do rather than guess: take your monthly dedicated-GPU cost, divide by your monthly requests, and compare that to your per-token cost per request. A machine at ₹28,000 a month serving 300 requests a day is about ₹3.10 a request; the same machine at 30,000 requests a day is about ₹0.03. Nothing changed except utilisation.

### Tools today

The open serving engines that implement continuous batching and paged key-value caches are vLLM and Hugging Face's Text Generation Inference; both will serve a quantized open-weight model with an OpenAI-shaped API, which keeps you portable. Serverless GPU platforms and hourly GPU rentals exist from several vendors at prices that move monthly, and the cheapest honest path for a student team is: use a per-token hosted API until a measured volume justifies otherwise, and prototype anything heavier on a free notebook GPU. Every price, product name and instance type in this paragraph will be wrong within six months — this chapter is marked high-volatility for that reason.

### Where it lives

Upstream: chapter 6 decides what you are serving and at what precision, since the quantized size determines how many concurrent conversations fit in memory. Downstream: chapter 14 puts a gateway in front of whatever you choose, which is what makes switching between these four shapes a configuration change instead of a rewrite.

Sideways: M05 chapters 4 and 8 are the queueing, rate-limiting and timeout discipline this needs, because an overloaded inference server fails in exactly the ways that module describes. M07 chapter 6 is where the p95 you must watch is recorded. M09 chapter 14 is the cost arithmetic. M17 chapter 6 is where it becomes margin.

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

**Throughput and latency are in opposition, and benchmarks quote the one that flatters.** A server tuned for maximum tokens per second batches aggressively, which makes any individual user wait longer. A published "1,800 tokens per second" is an aggregate across many simultaneous requests and tells you nothing about what one student on 4G will experience. Always ask: at what concurrency, and what was the p95 for a single request?

**Idle time is the expensive part of a rented GPU, and nobody notices it.** Your bill does not depend on requests, so a quiet week costs exactly as much as a busy one. Set a utilisation target, measure against it weekly, and treat a GPU under 20% utilised as a decision to revisit rather than a sunk cost.

**Cold starts belong in the p95, not in a footnote.** On serverless GPU, the honest latency figure includes the cold path at the frequency it actually occurs. Measure it by leaving the service idle for your real gap between requests and then timing one.

**Long prompts are a first-token problem, long answers are a throughput problem.** They need different fixes: trim the prompt (M10 chapters 5 and 6) for the first, cap the output length for the second. Teams frequently apply one to the other and see nothing improve.

**The KV cache is why your concurrency limit is lower than you expect.** At a long context, per-conversation cache memory can rival the weights themselves. A machine that holds one model comfortably may support only a handful of simultaneous long conversations, and the symptom is queueing rather than an error.

**Spot and preemptible instances are cheap because they can be taken away.** Fine for a training run that checkpoints. Not fine for the thing answering your users, unless you have a fallback path that is tested.

### Verify it

The red flag for this chapter is the plainest one in the module: **a per-request cost nobody has measured.**

1. **Demand cost per request, in rupees, for every option.** Not per token, not per hour, not per million. Per request, at your measured volume, including idle time for anything rented.
2. **Ask for the break-even volume as a number.** "Requests per day above which renting wins." If the AI cannot state it, it has not done the arithmetic.
3. **Separate the two latency numbers.** Time to first token and tokens per second, each measured at the concurrency you expect, not at concurrency one.
4. **Ask what happens at double your peak concurrency.** The answer should mention the KV cache, a queue and a timeout. If it mentions none of them, the plan has no failure behaviour.
5. **Ask for the cold-start figure and where it lands.** Then ask what fraction of your requests will hit it.
6. **Check for continuous batching by name** in any self-hosted plan. Naive batching wastes most of the advantage.
7. **Insist the API shape is portable.** If switching providers means rewriting your application, you have bought lock-in along with the GPU.

Red flags: a GPU rented before load was measured; a throughput benchmark presented as a latency promise; no utilisation figure; cold starts excluded from p95; no concurrency test; no timeout or queue in the design; spot instances serving users with no fallback; and a self-hosted plan that never mentions the key-value cache.

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
