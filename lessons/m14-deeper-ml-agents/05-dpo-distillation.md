---
id: m14-05-dpo-distillation
module: m14-deeper-ml-agents
slug: dpo-distillation
title: "DPO, Distillation and When Fine-Tuning Is the Wrong Answer"
order: 5
duration_min: 30
is_free: false
mission: false
volatility: low
last_reviewed: 2026-11-16
---

## Key learnings
- Tell apart the three jobs that all get called fine-tuning, and pick the right one
- Build preference pairs that encode judgement rather than length
- Distil an expensive model into a cheap one and measure the gap you accepted
- Recite the seven situations where no amount of tuning will help
- Catch a preference dataset where one person's taste became the product's rule

## Lesson

### The pain

M09 chapter 11 told you that preference training exists, that a reward model sits inside RLHF, and that sycophancy is one of its documented side effects. Chapter 4 of this module ran supervised fine-tuning. This chapter covers the rung above it, the sideways move that saves money, and — at greater length than either — the cases where the whole family of techniques is the wrong answer.

Start with the wrong answer, because it is where teams lose the most time. A team's assistant gives correct but unhelpful answers: technically right, three paragraphs long, never leading with the thing the student asked. They collect 800 pairs where one answer is "better" and run preference tuning. The model's answers get longer. Everyone is confused, until someone reads the pairs and finds that in 71% of them the chosen answer was simply the longer one, because the person labelling at 11 pm used length as a proxy for effort. The tune worked perfectly. It learned exactly what the data said.

A second team does something more expensive. They want to cut their ₹18,000-a-month inference bill, so they distil a large provider model into a small open-weight one by training on thousands of the large model's outputs. It works: the bill falls to ₹2,400 and quality drops by four points on their eval set, which they accept. Then someone reads the provider's terms of service and finds that training a competing model on its outputs is prohibited. The artefact is unusable and nobody checked before spending the fortnight.

### The idea

**Three different problems hide behind the word "fine-tuning", and each needs a different kind of data.**

**Supervised fine-tuning** needs examples of the right answer. Use it when you can write the output you want. **Preference tuning**, of which Direct Preference Optimization is the practical form, needs *pairs* — a better and a worse answer to the same input. Use it when every answer is defensible and the difference is judgement you can only express by choosing. **Distillation** needs a teacher: a strong model's outputs on your real inputs, which a small model is then trained to imitate. Use it when quality is already acceptable and the problem is cost or latency.

### The analogy

Three ways to improve the canteen's thali.

Show the cook forty photographs of a correctly assembled thali and have them copy the arrangement. That is supervised fine-tuning: you knew the right answer and demonstrated it.

Put two correctly assembled thalis side by side and tell the cook which one a student would prefer, four hundred times. Neither is wrong. You cannot write the rule down — it is about balance, and you only know it when you see both. That is preference tuning.

Have your most experienced cook plate two thousand thalis while a new junior watches and copies, until the junior produces something close at a third of the wage. That is distillation. The junior will never exceed the senior, and will faithfully reproduce the senior's one bad habit.

**Where the analogy breaks.** A cook shown two thalis understands *why* one is better and generalises. Preference tuning has no access to your reason. It only sees which string you picked, so any accidental regularity in your picks — longer, more polite, more bullet points — becomes the lesson. The canteen has an explanation channel; the training data does not.

### Picture it

```
  YOUR PROBLEM                 DATA YOU NEED            TYPICAL SIZE
  ───────────────────────────────────────────────────────────────
  "I can write the             prompt → response         500–5,000
   right answer"               pairs                     examples
        │                      (chapter 4: SFT)
        │
  "both answers are ok,        prompt → chosen,          500–2,000
   one is better"              rejected                  pairs
        │                      (DPO, this chapter)
        │
  "quality is fine,            your real inputs →        2,000–20,000
   the bill is not"            strong model's output     examples
        │                      (distillation)
        │
  "the facts are wrong"  ──▶   NOT A TUNING PROBLEM
  "needs a citation"     ──▶   go back to chapter 3,
  "changes weekly"       ──▶   rung 2 or rung 3
```

Each of the three tuning jobs needs a different shape of data and a different volume, and three common complaints at the bottom are not tuning problems at all.

### How DPO actually works, in words

Preference tuning used to require a reward model and a reinforcement-learning loop — the RLHF pipeline M09 described. Direct Preference Optimization removes both. It takes your pairs and adjusts the model directly so that, for each prompt, the chosen answer becomes more likely relative to the rejected one, while a strength setting keeps the model from drifting far from where it started.

Three practical consequences. It runs as an ordinary training loop, so it fits the same free notebook GPU and the same LoRA adapter machinery as chapter 4. It needs the model to already be decent at the task, so **DPO goes after SFT, not instead of it**. And the strength setting is the knob that matters: too weak and nothing changes, too strong and the model collapses into whatever superficial pattern your chosen answers share.

Where do pairs come from? Three honest sources. Your product's own thumbs-up and thumbs-down, if the same prompt produced both. Two models answering the same real input, with a human choosing. And deliberate pair construction: take a good answer, make a worse version of it that fails in the specific way you want trained out, and pair them. The third is the most controllable and the most work.

### Distillation, and its two hard limits

Distillation is the most practically useful idea in this chapter for an Indian student startup, because it attacks the number that decides survival: cost per request. Collect your real inputs, have a strong model produce outputs for them, check those outputs, and supervised-fine-tune a small model on the result. You now serve a model a fraction of the size, often on your own hardware, at a fraction of the cost per request.

Two limits are absolute. **The student cannot exceed the teacher**, so distilling a model that is wrong 12% of the time gives you a model that is wrong at least 12% of the time, usually more, and in the same places. And **the licence may forbid it**: several providers' terms prohibit using their outputs to train competing models, and open-weight model licences carry their own conditions. Read the terms before you spend the GPU time, not after.

The thing to measure is the gap. Score teacher and student on the same eval set, record both numbers and both costs per request, and state the trade in one sentence: four points of quality for a seven-times cost reduction, accepted or rejected.

### Where it lives

Below this chapter: chapter 4's SFT, which DPO requires as a foundation and distillation uses as its mechanism. Above it: chapter 8's reinforcement learning, which is what DPO replaced for this purpose and which you will almost certainly never run. Beside it: chapter 10's regression suite, which is the only thing that can tell you a preference tune did not make something else worse, and chapter 7's serving, where a distilled model's cost advantage is actually realised.

Outside the module: M09 chapter 14's cost arithmetic is what makes a distillation decision defensible, and M17 chapter 6 is where that number becomes margin.

### Words to own

| Term | Plain meaning | Example |
|---|---|---|
| Preference pair | One prompt, a chosen and a rejected answer | Both correct, one better |
| DPO | Preference tuning without a reward model or RL loop | Runs like ordinary SFT |
| Beta / strength | How far the tune may drift from the starting model | Too high collapses it |
| Reward model | A separate model that scores answers | What DPO avoids needing |
| Distillation | Training a small model on a strong model's outputs | Teacher and student |
| Teacher | The strong, expensive model producing the examples | Scored on your eval set |
| Student | The small model being trained to imitate it | Cheaper per request |
| Quality gap | Teacher score minus student score | 4 points, accepted |
| Length bias | Preferring longer answers by accident | 71% of pairs |
| Terms of service | What a provider allows you to do with outputs | Check before training |

### Tradeoffs and gotchas

**Check your pairs for the boring regularity before you train.** Compute the average length of your chosen answers and your rejected ones. If chosen is consistently longer, politer, or more heavily formatted, that is what will be learned. Fix it by constructing pairs where the better answer is *shorter*.

**One person's preferences are not your product's preferences.** Pairs labelled by one teammate at midnight encode that teammate. Have two people label the same 50 pairs and measure agreement exactly as chapter 2 does for labels; below about 80%, you do not yet have a shared idea of "better" and no tune can supply one.

**DPO on an unprepared model wastes the pairs.** If the model cannot produce a reasonable answer to your prompts at all, preference tuning has nothing to push between. Do SFT first.

**Distillation inherits the teacher's failures exactly.** Review the teacher's outputs before training on them, and throw away the bad ones. A distillation set is a dataset and chapter 2 applies: deduplicate, spread the categories, record provenance.

**Sycophancy is the predictable side effect.** M09 chapter 11 warned you. Preference tuning rewards answers humans liked, and humans like being agreed with. Include pairs where the better answer says "no", "you are mistaken", or "I cannot do that", or you will train a model that flatters.

**Here are the seven cases where no tuning helps.** The answer depends on a current fact. The output must carry a citation to a source. The requirement changes more often than monthly. A tool or a schema would make the error impossible. You have fewer than about two hundred examples. You have no eval set, so you cannot tell whether anything improved. And the real problem is that two stakeholders want different things, which is a product decision wearing a technical costume.

### Verify it

The specific failure here is a tune that worked perfectly on data that encoded the wrong lesson.

1. **Measure the length difference in your pairs before training.** Average chosen length versus average rejected length. A large gap means you are about to train length.
2. **Demand the agreement number on preference pairs.** Two labellers, same 50 pairs, percentage matched. Preference data without it is one person's taste.
3. **Check that refusals and disagreements are present among the chosen answers.** If every chosen answer is agreeable and helpful, you are training sycophancy.
4. **For distillation, demand the licence check in writing, before the GPU time.** Which provider, which clause, what it permits. Then demand the teacher and student scores side by side with costs per request.
5. **Ask what the student lost.** The quality gap, stated as a number on your eval set, plus any capability that fell off entirely.
6. **Run the seven-case list against your own problem out loud.** If any of them fits, the correct output of this chapter is "do not tune", and that is a successful outcome.

Red flags: preference pairs where the chosen answer is reliably longer; one labeller; DPO proposed without SFT first; distillation planned with no licence sentence; no teacher-versus-student table; no cost-per-request figure attached to a distillation decision; every chosen answer agreeable; and a second tuning attempt whose only change is "more data".

## Walkthrough
1. Write down your actual complaint about your product's AI output in one sentence. Then run it against the seven cases where tuning does not help, and record which ones fit.
2. If none fit, classify the problem: can you write the right answer (SFT), or can you only pick the better of two (DPO), or is quality already fine and cost is the issue (distillation)? Write one sentence of justification.
3. For a preference problem, collect 50 real inputs where your product produced two different answers — from two models, two prompts, or two moments. Keep both answers for each.
4. Have two teammates independently choose the better answer in all 50 pairs, without discussing. Compute the agreement percentage.
5. Compute the average character length of the chosen answers and of the rejected ones. Write both numbers down. If chosen is more than about 20% longer, your data has length bias.
6. Deliberately construct ten pairs where the better answer is shorter, and ten where the better answer declines to do something. Add them to the set.
7. For a cost problem instead, find your provider's terms and quote the exact clause about training on outputs. Write down whether distillation is permitted. If it is not, stop and record that.
8. If it is permitted, collect 500 real inputs, generate teacher outputs, and read 50 of them yourself. Record how many you rejected and why.
9. Score the teacher on your frozen eval set and record its cost per request from M09 chapter 14's method. This is the pair of numbers the student must be compared against.
10. Run the tune — DPO on top of your chapter 4 adapter, or SFT on the teacher outputs — and score the result on the same frozen eval set.
11. Write the trade in one sentence with two numbers in it: what you gave up and what you got. Then write the exit criterion that would make you undo it.

## Talk to AI
```prompt
CONTEXT:      I am a student on an AI-native engineering program. I direct AI
              and verify its output; I do not write code by hand. I have
              already run a supervised fine-tune with LoRA and I know what an
              adapter, a rank and an eval set are, so do not re-teach those.
              My team of three runs [product in two sentences] with real users
              in India. My complaint about our AI output is: [one sentence].
              Concrete example - input [paste], what we got [paste], what we
              wanted [paste]. Our frozen eval set has [number] cases and our
              current score is [number]. Our inference cost is about ₹[number]
              per month at [number] requests per day. We use [provider and
              model].

GOAL:         Decide which of supervised fine-tuning, preference tuning or
              distillation my complaint actually calls for - or establish that
              none of them does - and prepare the data for whichever it is.

REQUIREMENTS: 1. First run my complaint against the seven situations where no
                 tuning helps: a current fact, a required citation, a
                 requirement that changes weekly, something a tool or schema
                 would prevent, fewer than 200 examples, no eval set, and two
                 stakeholders wanting different things. Tell me which fit
              2. If none fit, classify my problem as SFT, preference tuning or
                 distillation, and justify it from my example
              3. For preference tuning: tell me where my pairs should come
                 from, how many I need, and give me a procedure to measure
                 agreement between two labellers with a pass threshold
              4. Tell me how to detect length bias in my pairs with a specific
                 calculation and a threshold, and give me five example pairs
                 where the better answer is SHORTER
              5. Give me five pairs where the better answer declines or
                 disagrees, so I do not train the model to flatter
              6. Explain what the DPO strength setting does, give me a
                 starting value, and tell me the symptom of it being too high
              7. For distillation: tell me exactly what to check in my
                 provider's terms before I train on its outputs, quote the
                 kind of clause to look for, and tell me what to do if it is
                 prohibited
              8. Give me the comparison table I must produce: teacher score,
                 student score, teacher cost per request, student cost per
                 request, and the one-sentence trade

CONSTRAINTS:  Free tier for training, on a notebook GPU of about 15 GB. Plain
              English for a second-language reader. I cannot afford a
              dedicated GPU running all month. My users write in English and
              [language]. Treat all provider terms and prices as stale and
              tell me to read them myself today.

NON-GOALS:    Do not propose reinforcement learning with a reward model. Do
              not propose collecting more data before the existing data has
              been checked for length bias and agreement. Do not write
              training code until I have confirmed the classification. Do not
              assume my provider permits training on its outputs.

ACCEPTANCE:   - The seven-case list has been run against my complaint
                explicitly, one line each
              - My problem is classified with a justification I agree with
              - A measured agreement percentage exists for my pairs
              - Average chosen length and average rejected length are both
                computed and compared
              - My pair set contains examples where better means shorter and
                examples where better means declining
              - For distillation, I have the exact licence clause quoted
              - A teacher-versus-student table exists with quality and cost
                per request on both rows

ASK:          Start with the seven-case list and nothing else. If any case
                fits my complaint, tell me to stop and say why. Only if none
              fits should you classify the problem.
```

## Stuck? Ask AI
```prompt
Here are 15 of my preference pairs: [paste them, chosen and rejected]. Do not train anything. Instead, compute the average length of my chosen answers and my rejected ones and tell me the ratio. Then tell me every other regularity you can find that distinguishes chosen from rejected — formatting, politeness, hedging, bullet points, opening phrases. For each regularity, tell me what the model will learn from it and whether that is what I actually want.
```

```prompt
Explain the difference between supervised fine-tuning, preference tuning and distillation using one single example product: a canteen support assistant. For each one, tell me what the data file looks like row by row, roughly how many rows I need, what it can fix, and what it definitely cannot fix. Then ask me to classify three of my own complaints before you tell me the answers.
```

```prompt
I want to distil an expensive model into a cheap one to cut my inference bill. Before any code: tell me what to look for in the provider's terms of service, what an open-weight model's licence might restrict, and what "the student cannot exceed the teacher" means in practice for my error rate. Then give me the exact table of numbers I must produce to prove the trade was worth it, and tell me the quality gap that should make me abandon it.
```

## Quiz

### Q1 · L3
Your 800 preference pairs have chosen answers averaging 740 characters and rejected answers averaging 310. What will the tune most likely learn?
- (a) The judgement you intended to encode
- (b) To produce longer answers ✅
- (c) To refuse more often
- (d) Nothing, because the pairs are balanced
> Preference tuning sees only which string you picked, never your reason. Any consistent surface difference becomes the lesson, and length is the easiest one to introduce by accident.

### Q2 · L3
What does Direct Preference Optimization remove compared with the RLHF pipeline?
- (a) The need for an eval set
- (b) The separate reward model and the reinforcement-learning loop ✅
- (c) The need for a base model
- (d) The need for preference data
> DPO adjusts the model directly from the pairs, so it runs as an ordinary training loop on the same hardware as a LoRA fine-tune. It still needs the pairs, and it still needs an eval set.

### Q3 · L4
A distilled student model is wrong on 16% of your eval cases and the teacher was wrong on 12%. What does this tell you?
- (a) The student needs more training epochs
- (b) The distillation failed and should be redone
- (c) This is the expected shape: a student inherits the teacher's errors and adds its own ✅
- (d) The teacher was the wrong choice
> A student trained to imitate a teacher cannot beat it, and compresses imperfectly on top. The real question is whether four points of quality is a fair trade for the cost reduction you measured.

### Q4 · L4
Which of these should send you back down the ladder rather than into a tune?
- (a) Answers are correct but in the wrong house style
- (b) Answers must include a citation to the source document ✅
- (c) Answers are all defensible but some read better than others
- (d) Quality is acceptable but cost per request is too high
> A citation requires the source to be present at request time, which is retrieval. Style is SFT, judgement between good answers is preference tuning, and cost is distillation.

### Q5 · L5
Your preference tune makes the model agree with users far more readily, including when they are wrong. What was missing from the pair set?
- (a) Enough pairs overall
- (b) Pairs where the better answer disagrees or declines ✅
- (c) A higher strength setting
- (d) A larger base model
> Humans pick answers that agree with them, so a set built only from natural preferences trains flattery. The fix is deliberately constructed pairs in which the chosen answer says no.

## Go deeper
- [Direct Preference Optimization: your language model is secretly a reward model](https://arxiv.org/abs/2305.18290)
- [Hugging Face TRL: DPO trainer documentation](https://huggingface.co/docs/trl/dpo_trainer)
