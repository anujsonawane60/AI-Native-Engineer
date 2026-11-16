---
id: m09-11-pretraining-to-reasoning
module: m09-ai-foundations
slug: pretraining-to-reasoning
title: "Pretraining, Instruction Tuning, RLHF and Reasoning Models"
order: 11
duration_min: 25
is_free: false
mission: false
volatility: medium
last_reviewed: 2026-11-16
---

## Key learnings
- Name the four stages that turn raw text prediction into a usable assistant
- Explain what instruction tuning and preference training each added, and what neither fixed
- Say what a reasoning model does differently, and what it costs in latency and tokens
- Climb the fine-tuning ladder in the right order: prompt, examples, retrieval, then tuning
- Catch "fine-tune it" proposed where retrieval or a better prompt is the real answer

## Lesson

### The pain

Two problems, same root.

A team is told that "the model was trained on the internet" and concludes it must know the canteen's prices, because the canteen has a website. It does not — the site was crawled before the last price rise, if at all, and the model will produce last year's number in a confident sentence.

A second team wants their assistant to always reply in the house style: short, in Marathi, never apologising twice. They propose fine-tuning. Someone else wants today's menu in the answers and proposes fine-tuning for that too. One of those is a reasonable use of fine-tuning and the other is a guaranteed waste of a weekend, and nobody in the room can say which.

Both confusions dissolve once you know how a model got from "predicts the next token" to "behaves like an assistant". It happened in four stages, each adding something specific and leaving other things untouched.

### The idea

**A usable assistant is built in four stages: pretraining gives it the world's patterns, instruction tuning makes it follow instructions, preference training shapes how it answers, and reasoning training teaches it to work before it speaks.**

**Pretraining** is self-supervised learning from chapter 2, at enormous scale: predict the next token across trillions of tokens of text. Out comes a **base model**, which knows an extraordinary amount and is nearly useless to talk to. Ask a base model a question and it may well continue with more questions, because a list of questions is a plausible continuation of a question.

**Instruction tuning**, also called supervised fine-tuning, is further training on tens of thousands of example pairs — an instruction and a good response, mostly human-written. This is the stage that makes it answer rather than continue. It is the biggest behavioural change in the whole pipeline and it is comparatively small and cheap.

**Preference training**, usually called **RLHF** — reinforcement learning from human feedback — is where humans are shown two responses to the same prompt and pick the better one. Those choices train a scoring model, and the assistant is then tuned to produce responses that score well. This is the stage that produces the helpful, formatted, hedging, polite voice you recognise, and the refusals.

**Reasoning training** is the newest stage. The model is trained to produce a long internal working-out before its answer, rewarded on tasks where the answer can be checked automatically — maths, code that either runs or does not, puzzles with verifiable solutions. The result is a model that spends many more tokens thinking and is substantially better at multi-step problems.

### The analogy

One cook, four stages of becoming employable.

**Pretraining** is reading every cookbook, food blog, menu and argument about food ever written. Vast knowledge, zero training in a kitchen. Ask this person for lunch and they may recite a recipe, or write a review, or continue your sentence — because they have absorbed what text looks like, not what a request means.

**Instruction tuning** is a month of apprenticeship. "When someone asks for lunch, you make lunch." A few thousand corrected examples and the behaviour changes completely. They now respond to requests.

**Preference training** is a year of customers comparing two plates and saying which they preferred. Nobody specifies the ideal dish; they just keep choosing. The cook converges on a house style: well-presented, safe, generous with garnish, and unwilling to serve certain things at all.

**Reasoning training** is being taught to taste and adjust before plating, and being judged only on whether the final dish was right. They now take longer and get hard dishes right far more often.

**Where the analogy breaks:** the cook, after all that, knows what is in the fridge this morning. The model does not. Nothing in these four stages gives it access to today — its knowledge is frozen at its **cutoff**, which is M00's second failure mode and the reason the first team's price was wrong. And preference training optimises what humans *preferred reading*, which is not the same as what was true. A confident, well-structured wrong answer frequently beats a hesitant correct one in a side-by-side comparison, and the model learns that.

### Picture it

```
  RAW TEXT, trillions of tokens
        │  self-supervised: predict the next token
        │  months, thousands of GPUs, enormous cost
        ▼
  ┌──────────────┐
  │  BASE MODEL  │  knows a lot, cannot be talked to
  └──────┬───────┘
         │  instruction tuning: ~tens of thousands of
         │  instruction-and-good-answer pairs
         ▼
  ┌──────────────┐
  │  INSTRUCT    │  follows instructions, answers questions
  └──────┬───────┘
         │  preference training (RLHF): humans pick the
         │  better of two answers, many times
         ▼
  ┌──────────────┐
  │  ASSISTANT   │  helpful tone, formatting, refusals
  └──────┬───────┘
         │  reasoning training: rewarded on checkable
         │  tasks for working it out first
         ▼
  ┌──────────────┐
  │  REASONING   │  thinks in tokens, slower, pricier,
  │    MODEL     │  much better at multi-step problems
  └──────────────┘
```

Each stage adds one capability to the one before, and none of them adds knowledge of today.

### Reasoning models, and when they are worth it

A reasoning model generates **thinking tokens** before its visible answer — sometimes hundreds, sometimes thousands. You are usually billed for them, you wait for them, and often you cannot read them.

The trade is easy to apply. For a multi-step problem — debugging why three systems disagree, planning a schema, working through a calculation with conditions — a reasoning model is dramatically better and the extra seconds are worth it. For "reformat this list as JSON" or "classify this ticket into four buckets", it is slower, dearer and no better. Paying reasoning prices for formatting is a common way student teams burn credits, and M10 chapter 2 treats the decision properly.

### The fine-tuning ladder

This is the most immediately useful thing in the chapter. When a model's output is not what you want, there are four moves, and they go in this order, cheapest and fastest first.

```
  1. BETTER PROMPT        minutes      fixes most problems
     clearer instruction, constraints, output format

  2. EXAMPLES IN PROMPT   an hour      fixes style and shape
     two or three good input-output pairs ("few-shot")

  3. RETRIEVAL            days         fixes FACTS and freshness
     fetch the real data and put it in the prompt

  4. FINE-TUNING          weeks        fixes deep style, tone,
     further training on your own pairs    narrow format at scale
```

Four responses to bad output, ordered by how much they cost you and what each one actually fixes.

Read rung 3 and rung 4 together, because the distinction is the whole point. **Retrieval fixes facts. Fine-tuning fixes behaviour.** Today's menu is a fact, so it goes in the prompt — rung 3, which is M10's RAG work. House style is behaviour, so it is a legitimate rung-4 candidate, though a prompt with three examples usually gets most of the way there for a thousandth of the effort.

The honest guidance for a student team: you will almost never need rung 4 in this program, and when you think you do, check whether rung 1 was done properly. M14 chapters 3 to 5 are where fine-tuning is practised, including the cheap variants that adjust a small set of extra weights rather than the whole model.

### Where it lives

Nothing in this chapter happens in your product. All four stages happened before you got access, and what reaches you is a named model with a version, a cutoff date, a price and a behaviour profile.

Three of your own decisions follow from it. **Which model variant** — base, instruct or reasoning — chapter 13's work. **Where facts come from**, which is retrieval and belongs to M10. **What you do about the cutoff**: supply anything time-sensitive yourself, every request.

### Words to own

| Term | Plain meaning | Example |
|---|---|---|
| Pretraining | Huge self-supervised run over raw text | Months of GPUs |
| Base model | The pretrained model before any tuning | Continues text, cannot chat |
| Instruction tuning | Training on instruction-and-answer pairs | Makes it answer |
| SFT | Supervised fine-tuning, the same thing | Tens of thousands of pairs |
| RLHF | Tuning from human preferences between answers | Produces the assistant voice |
| Reward model | A model trained to score answers like humans did | Used to tune the assistant |
| Alignment | Making behaviour match intended values | Chapter 16 |
| Reasoning model | Works through a problem in tokens before answering | Slower, dearer, better |
| Thinking tokens | The hidden working-out, usually billed | Can be thousands |
| Knowledge cutoff | The date its training data ends | Why prices are stale |
| Few-shot | Putting examples in the prompt | Rung 2 of the ladder |
| LoRA | Cheap fine-tuning of a small set of extra weights | Practised in M14 |
| Distillation | Training a small model to imitate a big one | Why small models got good |

### Tradeoffs and gotchas

**Preference training made models agreeable, and agreeable is not accurate.** RLHF optimises for what raters preferred, and raters prefer confident, tidy, complete-sounding answers. The documented side effect is sycophancy: push back on a correct answer and the model may well fold. The defence is to ask for reasoning and alternatives rather than agreement — M00's Director's Loop ASK field.

**Refusals are a trained behaviour, not a rule engine.** Being statistical, they both over-fire — refusing a legitimate medical or security question — and under-fire, which is what jailbreaks exploit in chapter 16. Never treat a refusal as a security control.

**The cutoff is not a soft boundary.** Every library version, price, policy, free-tier limit and model name an AI gives you is from before its cutoff. That is why chapters 13 and 14 insist on the provider's own page for anything involving money.

**Fine-tuning can make a model worse at everything else.** Tune hard on your narrow format and general ability degrades. It also freezes you: the base model improves next quarter and your tuned version does not. Both are real costs that plans rarely mention.

**Thinking tokens are billed and invisible.** A reasoning model can quietly spend several thousand tokens before a two-line answer. If your cost estimate counts only the visible output, it can be wrong by a large multiple — a trap chapter 14 returns to.

### Verify it

**Red flag, the one worth memorising: "fine-tune it on your data" offered for a fresh-facts problem.** Apply the test from chapter 6 — if the menu changes tomorrow, does this plan still work? If the answer is "retrain", it is rung 4 doing rung 3's job. Ask for the retrieval version and compare the two in hours.

**Red flag: a plan that skips rungs 1 and 2.** Before anyone talks about tuning, ask what the best prompt achieved and what three examples in the prompt achieved, with the actual outputs shown. A surprising share of fine-tuning requests are badly-written prompts.

**Red flag: a current fact asserted with no source.** Prices, limits, version numbers, "the latest model is...". Ask where it came from. The right answer is a URL you can open.

**Red flag: a reasoning model proposed for a formatting or classification task.** Ask what it costs per request against the cheap model and whether the quality difference is measurable on your own examples. Usually it is not, and chapter 13's mission is how you find out.

**Red flag: RLHF described as making a model truthful.** It made it preferred. Those are different, and the gap is where confident wrong answers live.

And the check to run yourself, in fifteen minutes: ask a model for a fact about your own domain that changed recently. Note how confidently it answers. Then tell it, falsely, that it is wrong, and see whether it folds. Then ask it directly what its knowledge cutoff is. Three questions, and you have seen the cutoff, the confidence and the sycophancy on your own screen.

## Walkthrough
1. Ask a model a question about your own domain whose answer changed in the last year — a price, a fee, a rule. Record the answer and how confident it sounded.
2. Check the real answer at the source. Record the gap and the date of the real information.
3. Ask the model directly what its knowledge cutoff is. Write it down and compare against what you just discovered.
4. Now test for sycophancy: in a fresh conversation, get a correct answer, then insist firmly that it is wrong. Record whether it holds its position or folds, and what that tells you about pushing back on your own work.
5. Take one output from your product that is not in the style you want. Write down which of the four rungs the problem actually sits on, and justify it in one sentence.
6. Do rung 1 properly: rewrite the prompt with explicit constraints and output format. Record the result.
7. Do rung 2: add two or three good input-output examples to the prompt. Record the result and compare against rung 1.
8. If facts are still wrong, write down what would have to be fetched and from where. That sentence is your rung-3 design, and M10 is where you build it.
9. Take one genuinely multi-step task from your product and run it on a normal model and on a reasoning model. Record quality, latency and token counts for both.
10. Take one trivial formatting task and do the same comparison. Write one sentence on when the reasoning model is worth its price, using your own four numbers.
11. Write one decision-log entry: which rung you are on, what you tried on the rungs below, and the evidence that made you move up.

## Talk to AI
```prompt
I have an AI feature whose output is not good enough, and I want to fix it at the cheapest rung rather than jumping to fine-tuning. Challenge me.

My feature: [what it does, who uses it]
My current prompt: [paste it in full]
Three examples of output I am unhappy with: [paste them, and say what was wrong with each]
What good output would look like: [describe or paste an example]

Work through this in order and stop after step 3:

1. For each of my three bad outputs, classify the problem as one of: WRONG FACTS, WRONG STYLE OR FORMAT, WRONG REASONING, or PROMPT NOT ACTUALLY ASKING FOR WHAT I WANT. Quote the part of my prompt or output that justifies your classification.

2. Tell me which of these four moves would fix each problem, and which would not: (1) a better prompt, (2) examples inside the prompt, (3) retrieving real data at request time, (4) fine-tuning. Be explicit about which combinations would waste my time.

3. Rewrite my prompt properly - rung 1 - with clear constraints, an explicit output format, and whatever is missing. Explain each change you made and which of my bad outputs it targets.

4. Then show me rung 2: what two or three examples should I put in the prompt, and why those specific ones.

5. For anything that is a facts problem, tell me plainly that no amount of prompting or tuning fixes it, name what would have to be fetched and from where, and give me the test that proves it is a facts problem.

6. Tell me honestly whether fine-tuning would help me at all, what it would cost in hours for a team of three students, and what I would lose - including what happens when the base model improves next quarter.

7. Finally, tell me whether a reasoning model would help here. If it would not, say so plainly and tell me what it would cost me in latency and tokens for no benefit.
```

## Stuck? Ask AI
```prompt
Explain the four stages that turn a raw text predictor into a chat assistant, using one running analogy about training a new employee. For each stage, tell me what new ability appeared and what problem was still left unsolved afterwards.
```

```prompt
Explain why preference training can make a model agree with me when I am wrong. Then demonstrate it: give me a factual claim, let me insist you are mistaken, and afterwards explain what just happened and how I should phrase questions to avoid it.
```

```prompt
Give me eight tasks a student app might need. For each, tell me whether a normal model or a reasoning model is the right choice, and estimate how many extra tokens and seconds the reasoning model would cost for no benefit where it is the wrong choice. Then give me four more and ask me to decide first.
```

## Quiz

### Q1 · L2
Which stage makes a model answer a question instead of continuing your text?
- (a) Pretraining
- (b) Instruction tuning ✅
- (c) Preference training
- (d) Reasoning training
> A base model continues plausible text. Instruction tuning on instruction-and-answer pairs is the comparatively small, cheap stage that changes the behaviour completely.

### Q2 · L3
What did preference training mainly produce?
- (a) Factual accuracy on current events
- (b) The helpful tone, formatting and refusal behaviour you recognise ✅
- (c) The ability to use tools
- (d) The model's knowledge of the world
> Humans picked the better of two answers, so the model learned what humans prefer reading. That is a style and behaviour signal, not a truth signal.

### Q3 · L4
Your assistant must mention today's menu. Which rung of the ladder does this problem sit on?
- (a) Fine-tuning, because the model needs to learn the menu
- (b) Retrieval: fetch the current menu and put it in the prompt ✅
- (c) Examples in the prompt showing past menus
- (d) A reasoning model, which can work out the menu
> Facts that change belong in the prompt at request time. Training shifts behaviour, not knowledge, so a tuned model would still be wrong tomorrow.

### Q4 · L4
When is a reasoning model clearly worth its extra cost and latency?
- (a) For reformatting a list into JSON
- (b) For classifying a support ticket into four buckets
- (c) For a multi-step problem where intermediate steps must be worked out ✅
- (d) For any task, since it is a newer kind of model
> Thinking tokens pay off on genuinely multi-step work. On formatting and classification they add seconds and rupees for no measurable gain.

### Q5 · L5
You get a correct answer from a model, insist firmly that it is wrong, and it changes its answer. What does this reveal, and what should you change?
- (a) The model has a bug; switch providers
- (b) Preference training rewarded agreeableness, so ask for reasoning and alternatives instead of agreement ✅
- (c) The temperature is set too high
- (d) The knowledge cutoff has been exceeded
> Raters preferred confident, accommodating answers, so the model learned to accommodate. Demanding options and tradeoffs rather than confirmation is the Director's Loop defence from M00.

## Go deeper
- [Anthropic: Core views on AI safety and how models are trained](https://www.anthropic.com/news/core-views-on-ai-safety)
- [Hugging Face: illustrated reinforcement learning from human feedback](https://huggingface.co/blog/rlhf)
