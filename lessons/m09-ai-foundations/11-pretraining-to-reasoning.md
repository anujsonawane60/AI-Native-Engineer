---
id: m09-11-pretraining-to-reasoning
module: m09-ai-foundations
slug: pretraining-to-reasoning
title: "Pretraining, Instruction Tuning, RLHF and Reasoning Models"
order: 11
duration_min: 15
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

One team hears "the model was trained on the internet" and concludes it knows the canteen's prices, because the canteen has a website. It does not, and it will give last year's number in a confident sentence. Another team proposes fine-tuning for the house style; someone else proposes fine-tuning to get today's menu into the answers. One of those is reasonable and the other wastes a weekend, and nobody can say which.

### The idea

**A usable assistant is built in four stages: pretraining gives it the world's patterns, instruction tuning makes it follow instructions, preference training shapes how it answers, and reasoning training teaches it to work before it speaks.**

**Pretraining** is chapter 2's self-supervised learning at enormous scale: predict the next token across trillions of them. Out comes a **base model** that knows an extraordinary amount and cannot be talked to.

**Instruction tuning**, or supervised fine-tuning, trains on tens of thousands of instruction-and-answer pairs. It makes the model answer rather than continue: the biggest behavioural change in the pipeline, and cheap.

**Preference training**, or **RLHF**, shows humans two responses and asks which is better; those choices train a scoring model and the assistant is tuned to score well. This produces the polite voice you recognise, and the refusals.

**Reasoning training** rewards working the answer out first, on tasks that can be checked automatically. It spends many more tokens thinking and is much better at multi-step problems.

### The analogy

One cook, four stages of becoming employable. **Pretraining** is reading every cookbook ever written: vast knowledge, no kitchen training. **Instruction tuning** is a month of apprenticeship — "when someone asks for lunch, you make lunch". **Preference training** is a year of customers comparing two plates, until the cook settles into a house style. **Reasoning training** is learning to taste and adjust before plating, judged only on the final dish.

**Where the analogy breaks:** the cook knows what is in the fridge this morning. The model does not — its knowledge is frozen at its **cutoff**, which is why the first team's price was wrong. And preference training optimised what humans *preferred reading*: a confident wrong answer beats a hesitant correct one, and the model learned that.

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

A reasoning model generates **thinking tokens** before its visible answer — sometimes thousands, billed and usually unreadable. For a multi-step problem they are worth it; for "reformat this list as JSON" they are slower and no better.

When the output is not what you want, there are four moves, cheapest first.

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

Rungs 3 and 4 carry the whole point: **retrieval fixes facts, fine-tuning fixes behaviour.** Today's menu is a fact, so it goes in the prompt; house style is behaviour, a legitimate rung-4 candidate, though three examples usually get most of the way there.

### Where it lives

- **Nothing here happens in your product.** All four stages ran before you got access; what reaches you is a named model with a version, a cutoff date, a price and a behaviour profile.
- **Which variant — base, instruct or reasoning — is chapter 13's work**, and where facts come from is M10's retrieval. Anything time-sensitive you supply yourself, every request.
- **Fine-tuning as a practice is M14.**

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

- **Preference training made models agreeable, and agreeable is not accurate.** The side effect is sycophancy: push back on a correct answer and the model may fold.
- **Refusals are a trained behaviour, not a rule engine.** They over-fire on legitimate questions and under-fire where jailbreaks exploit them, so never treat one as a security control.
- **The cutoff is not a soft boundary.** Every version, price, policy and model name an AI gives you predates it, which is why chapters 13 and 14 insist on the provider's own page.
- **Fine-tuning can make a model worse at everything else**, and it freezes you: the base model improves next quarter, your tuned copy does not.
- **Thinking tokens are billed and invisible**, so an estimate of visible output alone is wrong by a multiple.

### Verify it

- **"Fine-tune it on your data" for a fresh-facts problem** — the one worth memorising. If the menu changes tomorrow, does the plan still work? "We would retrain" means rung 4 is doing rung 3's job.
- **A plan that skips rungs 1 and 2.** Ask what the best prompt achieved, and what three examples achieved.
- **A current fact asserted with no source.** The right answer is a URL you can open.
- **A reasoning model proposed for formatting or classification.** Ask its cost against the cheap model.
- **RLHF described as making a model truthful.** It made it preferred, and the gap is where confident wrong answers live.
- **Check yourself:** ask for a fact that changed recently, tell it falsely that it is wrong, then ask its knowledge cutoff.

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
