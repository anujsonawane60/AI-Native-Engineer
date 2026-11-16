---
id: m10-02-reasoning-models
module: m10-building-with-ai
slug: reasoning-models
title: "Reasoning Models: Decomposition, Verification, Cost and Latency"
order: 2
duration_min: 25
is_free: false
mission: false
volatility: medium
last_reviewed: 2026-11-16
---

## Key learnings
- Say what a reasoning model actually buys you, and what it does not
- Match task type to model class instead of using one model for everything
- Measure the latency and cost a reasoning step adds, on a real connection
- Use an expensive model at design time and a cheap one at runtime
- Check a reasoning answer rather than trusting its visible working

## Lesson

### The pain

A team wires the strongest available reasoning model into every AI call in their canteen app. One of those calls classifies an incoming WhatsApp message as `order`, `complaint` or `other`. It is a three-way choice with a fifteen-word input.

Three things happen. The classification takes nine seconds, so on slow 4G the student sees a spinner long enough to close the app. The free-tier quota is gone by eleven in the morning, because classification is the highest-volume call in the product. And the one task that genuinely needed careful thought — working out whether four students splitting a ₹340 bill by UPI can each pay a whole-rupee amount, and who absorbs the remainder — was done by a cheap model, which produced a tidy, confident, wrong answer.

They had exactly the right tools and had pointed each one at the other one's job.

### The idea

**A reasoning model spends extra tokens thinking before it answers. You are buying decomposition and self-checking, and you pay for it in money and in seconds.**

This is not a smarter model in some general sense. It is the same kind of model given room and training to break a problem into steps, try one, notice it is wrong, and try again — all before the first word of the answer reaches you. On a problem with several dependent steps that is transformative. On "is this message an order", it is nine seconds of thinking about nothing.

### The analogy

The canteen has a counter boy and a manager. Ask the counter boy the rate of poha and you get "twenty" in half a second. Ask the manager and you get twenty as well, after they have finished the thing they were doing.

Now ask whether switching to a different rice supplier saves money, given a delivery charge, a bulk discount above forty kilos, and the fact that the cheaper rice has more wastage. The counter boy will give you a confident number. The manager will take out a calculator, work through it, catch their own slip in the middle, and give you a number you can act on.

Both are useful. Choosing between them is the whole skill.

**Where the analogy breaks:** the manager's written working really is how they got the answer, so you can check it line by line. A reasoning model's visible thinking is itself generated text, and it is not guaranteed to be the actual cause of the final answer. It can produce sound-looking working and a wrong answer, or messy working and a right one. The trace is a strong hint, not an audit trail — which changes how you verify, at the bottom of this lesson.

### Picture it

```
  CHEAP MODEL
  prompt ──▶ [answer tokens] ──▶ reply        ~0.4 s

  REASONING MODEL
  prompt ──▶ [thinking tokens: try, check, retry]
                      │
                      ▼
             [answer tokens] ──▶ reply        ~9 s
                      ▲
            you pay for these too,
            and usually never see them
```

The reasoning model emits a block of thinking tokens before any answer tokens. You are billed for them, you wait for them, and in most products they are hidden from the user.

### Which class for which task

A rough map you can apply today, and tighten with your own measurements.

**Cheap and fast wins for:** classification, extraction, reformatting, translation, short summaries, tone rewriting, choosing one of a known list, and anything where a schema already constrains the output. These are the high-volume calls in almost every product, so this is where your bill lives.

**Reasoning earns its cost for:** multi-step arithmetic and money logic, planning a change across several files, debugging from a symptom, comparing options against several competing constraints, writing a schema or an eval set, and any question where "show me where I am wrong" is the real request.

Notice the shape of the second list. Most of those tasks are things *you* do a handful of times while designing, not things your product does ten thousand times a day. That observation is the most valuable one in this chapter.

### The build-time trick

Use the expensive reasoning model to produce an *artefact*, then let a cheap model serve users against that artefact.

Ask the reasoning model to design your JSON schema, write your system prompt, enumerate the thirty hostile inputs your feature must survive, build your eval set, or work out the paise-rounding rule for a split UPI payment. Save the result in your repository. At runtime, the cheap model follows the schema, obeys the prompt, and applies the rule.

You paid for reasoning once, at design time, where nine seconds costs nothing and nobody is on 4G. Your users get the fast path. This pattern recurs through the whole module: chapter three's schemas, chapter twelve's eval sets and chapter nineteen's rules files are all reasoning-model output serving cheap-model runtime.

### Where it lives

M09 introduced reasoning models as a training story — what RLHF and reasoning post-training do to a base model — and gave you the token economics to price any of this. This chapter is the engineering decision that follows: which class of model sits behind which call in your product.

Mechanically, that decision lives in one place in your code, a router, and it should. One function that picks the model by task type is auditable and changeable. Model names scattered through forty call sites are neither. M05's work on rate limiting and cost control is the other half of this: a reasoning call is exactly the kind of expensive operation that needs a per-user limit and a timeout.

### Words to own

| Term | Plain meaning | Example |
|---|---|---|
| Reasoning model | One trained to think in tokens before answering | Used for multi-step money logic |
| Thinking tokens | Hidden tokens spent working, which you pay for | 2,000 before a 40-token answer |
| Reasoning effort | A dial for how much thinking to allow | Low for triage, high for planning |
| Decomposition | Splitting a problem into ordered steps | Bill split, then rounding, then UPI |
| Self-verification | The model checking its own intermediate work | Catching its own arithmetic slip |
| Latency | Wall-clock time to a usable answer | 0.4 s versus 9 s |
| p95 latency | The slow end most users actually notice | Your real number, not the average |
| Router | Code that picks the model per task | Classify to cheap, plan to reasoning |
| Escalation | Retrying a failed cheap call on a stronger model | Only when the cheap one is unsure |
| Thinking budget | A cap on thinking tokens per call | Prevents a runaway bill |

### Tradeoffs and gotchas

**Latency is a product decision, not a technical detail.** Nine seconds is unacceptable in a message reply and completely fine in a nightly report. Decide which one you are building before you choose a model, and measure p95 on a throttled connection rather than on campus wifi.

**Reasoning does not fix missing context.** If the menu is not in the prompt, no amount of thinking will produce today's prices. It will produce a *better-argued* wrong answer, which is more dangerous than an obviously wrong one. Grounding is chapter eight's job and it is not substitutable.

**Thinking tokens are real money and they are invisible.** A forty-token answer can hide three thousand tokens of thinking. If your cost model only counts what you can see, it will be wrong by an order of magnitude. Price a reasoning call by its total token count, the way M09 taught.

**Overthinking is a failure mode.** Hand a reasoning model a trivial question and it will sometimes reason itself out of the obvious answer, inventing edge cases that do not exist. If a task is genuinely simple, the simple model is not merely cheaper — it is often more accurate.

**The visible trace is not an audit trail.** Do not show a user the thinking as justification, and do not treat it as an explanation for a decision that matters, such as refusing a refund. If you need a defensible reason, ask for the reason as part of the structured answer, which is chapter three.

**Free tiers punish you for this fastest.** Reasoning calls consume quota several times faster per request than cheap ones. A single unguarded reasoning call inside a loop will end your day's allowance before lunch, which is why chapter fourteen insists on a step cap.

### Verify it

The temptation is to read the lovely working and feel reassured. Resist it.

**Check the answer against ground truth, not against the trace.** Keep five problems where you independently know the right answer — a bill split with an awkward remainder, a date calculation across a month boundary, a discount applied in the wrong order. Run them. Right answers with ugly working are fine. Pretty working with a wrong answer is the dangerous case, and only the ground truth catches it.

**Ask the same question twice.** These models are not deterministic. If two runs disagree, you have found a question the model is guessing at, and no amount of prompt polish will fix it — you need a deterministic calculation, which is chapter thirteen.

**Time it on a bad connection.** Throttle to slow 4G and record time to the first visible word. If nothing appears for more than about two seconds, you need a different model for that call or a different interface.

**Count the total tokens, including thinking.** Then multiply by your expected call volume. If the number frightens you, the task is probably on the wrong model class.

Red flags: a confident numeric answer with no working anywhere; a thinking trace whose conclusion does not match the answer given; a reasoning model behind a classification, extraction or formatting call; no measured p95 on a throttled connection; no cap on thinking tokens; and a user-facing explanation that is really the model's thinking trace copied out.

## Walkthrough
1. List every AI call in your product on one page. For each, write the input size, the output size, how many times a day you expect it, and whether a human is waiting for it.
2. Tag each call as structural — classify, extract, reformat, translate — or as judgement: plan, compute, debug, compare. Be strict; most calls are structural.
3. Pick your highest-volume structural call. Run it twenty times on a cheap fast model and record the wall-clock time for each run.
4. Run the identical call twenty times on the strongest reasoning model you have free access to. Record times and, if the provider exposes it, the thinking-token count.
5. Compare accuracy across the forty runs, not just speed. Count actual mistakes. If the cheap model made none, your decision is already made.
6. Multiply each model's per-call token count by your expected daily volume, using M09's pricing method. Write both daily figures down in rupees.
7. Now take your hardest judgement task — a money calculation with rounding, or a rule with three interacting conditions — and give it to the cheap model. Check the answer yourself against a calculator.
8. Give the same task to the reasoning model. Read its working, then ignore the working and check only the final answer against your own calculation.
9. Throttle your browser or phone to slow 4G and run both again. Record time to the first visible word, not time to completion.
10. Write a four-line routing rule for your product: which calls use which class, what the thinking budget is, and what happens on timeout. Put it in your repository next to the function that calls the model.

## Talk to AI
```prompt
Help me decide which class of model each AI call in my product should use, and price the decision.

MY PRODUCT: [one sentence]
MY USERS: [devices, network conditions, patience]
MY BUDGET: free tiers only, so quota per day matters as much as rupees

MY AI CALLS - for each: what it does, input size, output size, calls per day, and whether a user is waiting:
1. [call]
2. [call]
3. [call]

For each call, do all of this:

A. Say whether it is structural work (classify, extract, reformat, translate, choose from a list) or judgement work (plan, compute, debug, compare competing constraints). Justify it in one line.
B. Recommend cheap-and-fast or reasoning, and say what specifically would go wrong with the other choice.
C. Estimate total tokens per call INCLUDING hidden thinking tokens if reasoning is used, then estimate the daily token total.
D. State the maximum acceptable latency for that call given that a user may be waiting on slow 4G, and say whether your recommendation meets it.

Then, across the whole product:
1. Which of my judgement tasks could I move to DESIGN TIME instead - have you reason about it once now, and have me save the result as a schema, a prompt, a rule or an eval set that a cheap model follows at runtime? Be specific about the artefact.
2. Write me a routing rule: which call uses which class, what the thinking budget is, and what happens on timeout.
3. Name the one call where I am most likely to be wrong about this, and what measurement would settle it.
```

## Stuck? Ask AI
```prompt
Explain what a reasoning model actually does differently from a normal one, in plain English, to someone who knows that an LLM predicts the next token. Use one concrete example of a multi-step money calculation. Then explain why the visible thinking is not a guarantee that the answer is right, and what I should check instead.
```

```prompt
Give me six tasks a student app might ask an AI to do. For each, ask me whether I would use a cheap fast model or a reasoning model, and make me say why before you answer. Then tell me where my reasoning was weak rather than just marking me right or wrong.
```

```prompt
I have an AI call that takes nine seconds and my users are on slow 4G. Give me every option for making this acceptable, in order of how much work each one is - including options that change the interface rather than the model, and options that move the work to a time when nobody is waiting.
```

## Quiz

### Q1 · L2
What are you paying for when you use a reasoning model?
- (a) A larger context window
- (b) Hidden thinking tokens generated before the answer ✅
- (c) A guarantee the answer is correct
- (d) Access to more recent training data
> A reasoning model emits thinking tokens before answer tokens. You are billed for them and you wait for them, even though they are usually hidden from the user.

### Q2 · L3
Which of these calls is the worst fit for a reasoning model?
- (a) Working out a four-way bill split with a paise remainder
- (b) Classifying a fifteen-word message into one of three buckets ✅
- (c) Planning a change that touches five files
- (d) Comparing two database designs against three constraints
> Classification is high-volume, latency-sensitive and structurally simple. Reasoning adds seconds and quota to every call while adding nothing to the answer.

### Q3 · L4
A reasoning model produces an elegant step-by-step trace and a wrong final number. What does this tell you?
- (a) The trace was truncated
- (b) The visible working is generated text, not a proof of the answer ✅
- (c) The temperature was set too high
- (d) The model needed a larger thinking budget
> The trace is a strong hint and not an audit trail. Verification has to compare the answer against ground truth you established independently.

### Q4 · L4
Your money-rounding logic needs careful multi-step thought, but it runs on every checkout. What is the best design?
- (a) Use the reasoning model on every checkout
- (b) Use the cheap model and accept occasional errors
- (c) Have the reasoning model derive the rule once, then apply it deterministically ✅
- (d) Raise the reasoning effort and cache the response
> Reasoning at design time, cheap or deterministic execution at runtime. The expensive model produces an artefact — a rule, a schema, an eval set — that costs nothing to apply.

### Q5 · L4
Your reasoning feature gives today's prices wrongly. Which change is least likely to help?
- (a) Putting today's menu rows into the prompt
- (b) Increasing the thinking budget ✅
- (c) Grounding answers in retrieved data with citations
- (d) Refusing to answer when the menu block is missing
> Thinking cannot recover information that was never supplied. More reasoning on absent context produces a better-argued wrong answer, which is harder to spot than an obvious one.

## Go deeper
- [Anthropic: Extended thinking](https://docs.anthropic.com/en/docs/build-with-claude/extended-thinking)
- [OpenAI: Reasoning models guide](https://platform.openai.com/docs/guides/reasoning)
