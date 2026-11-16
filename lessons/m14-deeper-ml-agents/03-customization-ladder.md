---
id: m14-03-customization-ladder
module: m14-deeper-ml-agents
slug: customization-ladder
title: "The Customization Ladder: Prompt, RAG, Fine-Tune, Train"
order: 3
duration_min: 30
is_free: false
mission: false
volatility: low
last_reviewed: 2026-11-16
---

## Key learnings
- Place any AI quality problem on the seven-rung ladder before proposing a fix
- Judge every rung on four costs: build, per request, maintenance, reversibility
- Say in one sentence what fine-tuning can fix that prompting and retrieval cannot
- Write the falsifiable test that proves a lower rung was genuinely exhausted
- Catch a fine-tune proposed where the real problem was a fact, a tool or a prompt

## Lesson

### The pain

M09 chapter 11 gave you a four-rung ladder — better prompt, examples in the prompt, retrieval, then fine-tuning — and one load-bearing rule: retrieval fixes facts, fine-tuning fixes behaviour. It said you would almost never need the top rung, and that this module is where the climbing is practised. This chapter extends the ladder to the rungs a Build-tier team actually faces, and adds the axis M09 left out: what each rung costs you *forever*, not what it costs to build.

Here is why that axis matters. A team's assistant gives wrong fee amounts. Someone says "let us fine-tune it on our handbook." Two weekends later they have a model that sounds exactly like their handbook and still gives wrong fee amounts, because the fee changed in September and the training file was built in July. They now carry three permanent burdens: a model to retrain whenever a fee changes, a hosting bill, and a dependency on a base model the provider will deprecate within a year. The original problem is untouched.

The reverse failure is quieter. A team pushes prompts for six weeks because fine-tuning sounds frightening, when their actual problem — output that must match a rigid internal format on every call, after eleven prompt attempts — is exactly what the fourth rung is for.

### The idea

**Climb one rung at a time, and justify each rung by what the one below it provably could not do.**

Every rung is judged on four numbers, not one. **Build cost**: hours and rupees to get it working. **Per-request cost**: what every user interaction now costs, forever. **Maintenance cost**: what breaks it, and how often you must redo it. **Reversibility**: how long it takes to undo if it was wrong. Students almost always compare rungs on build cost alone, which is the one axis where the higher rungs look cheapest relative to their appeal.

### The analogy

The canteen's dal is too salty for the hostel students and fine for everyone else.

Rung one: tell the cook each morning, "less salt in the hostel batch". Free, instant, and it depends on somebody remembering. Rung two: pin a note above the pot with the exact quantity. Rung three: keep a card index of each group's preferences and have the cook look up today's groups. Rung four: retrain the cook's hands so less salt is their default — weeks of practice, and now every dish is slightly different, including the ones that were fine.

The note costs nothing to change. Retrained hands cannot be undone on Tuesday because students complained the dal is now bland, and they changed dishes nobody asked about.

**Where the analogy breaks.** A cook whose hands you retrained still remembers the recipe for everything else. A fine-tuned model can quietly get worse at tasks you never tested, because the new training pulls all its weights, not only the salty-dal ones. That is **catastrophic forgetting**, and the canteen has no equivalent of it.

### Picture it

```
  7  TRAIN FROM SCRATCH     never, for you
  6  CONTINUED PRETRAINING  a new domain's language itself
  5  PREFERENCE TUNING      which of two good answers is better
  4  SUPERVISED FINE-TUNE   a format or behaviour prompting cannot hold
  ───────────────────── the line almost nobody should cross ──────
  3  RETRIEVAL (RAG)        facts you own, current, permissioned
  2  TOOLS + STRUCTURE      anything computable, schemas, routing
  1  PROMPT + EXAMPLES      wording, role, few-shot, caching
  0  PICK A DIFFERENT MODEL often the whole answer

        ▲ build cost, per-request cost, maintenance, lock-in
        │ all rise together as you climb
```

The ladder runs from choosing a different model up to training one yourself, with a line between retrieval and fine-tuning most student teams should never cross.

### What each rung is actually for

**Rung 0 — a different model.** M09 chapter 13 made this a mission for a reason. A quality problem that disappears on a model swap was never a customization problem. Try it first; an afternoon, fully reversible.

**Rung 1 — prompt and examples.** System prompt, role, constraints, few-shot examples, and the stable-prefix layout from M10 chapter 6 so the cost stays low. This is M10 chapters 1 and 5. The honest test of whether rung 1 is exhausted: can you show eleven distinct prompt versions and the eval score of each?

**Rung 2 — tools and structure.** Anything that can be computed should be computed, not generated: totals, dates, availability, permissions. A JSON schema makes format failures impossible rather than unlikely. M10 chapters 3 and 4. An enormous share of "the model keeps getting it wrong" is a missing tool.

**Rung 3 — retrieval.** Facts that are yours, current, private or permissioned. M10 chapters 8 to 12. Changing a fact means editing one document, which is the cheapest maintenance story on the ladder, and it is why retrieval beats fine-tuning for anything that can change.

**Rung 4 — supervised fine-tuning.** Teaching behaviour: a house style, a rigid output shape, a domain's phrasing, a classification judgement you have thousands of examples of. Chapter 4 runs one. Fine-tuning does not teach facts reliably and cannot be updated by editing a document.

**Rung 5 — preference tuning.** When answers are all correct but some are better, and the difference is a matter of judgement you can express only by choosing between pairs. Chapter 5.

**Rung 6 — continued pretraining.** Feeding a large volume of unlabelled domain text to a base model so it learns a field's vocabulary. Real, occasionally correct, far outside a student budget. **Rung 7 — from scratch.** As M09 chapter 2 said: you will never pretrain a model, and you do not need to.

### The four costs, side by side

| Rung | Build | Per request | Maintenance | Undo |
|---|---|---|---|---|
| 0 model swap | hours | changes | provider deprecations | minutes |
| 1 prompt | hours | small, cacheable | re-test on model change | minutes |
| 2 tools | days | one extra call | the tool's own upkeep | hours |
| 3 retrieval | 1–2 weeks | embedding + storage | ingest new documents | days |
| 4 fine-tune | 1–2 weeks | hosting a custom model | redo per base version | weeks |
| 5 preferences | 2–3 weeks | same as rung 4 | redo per base version | weeks |
| 6 pretraining | months | dedicated serving | continuous | not really |

The maintenance column is the one that decides real projects. Rungs 4 and above create an artefact that must be rebuilt every time the base model you built on is retired — and M09 chapter 13's glossary already warned you that deprecation is routine.

### Where it lives

This chapter is the decision gate for chapters 4 to 7 of this module. It is also an M01 decision-log entry: which rung you are on, what you tried below it, the number that proved the lower rung insufficient, and what would make you climb down again.

Downwards it leans on M10: rungs 1 to 3 are that module's entire content, and chapter 12's eval set is what makes "rung 1 was exhausted" a measurement instead of a feeling. Upwards it leans on M17 chapter 6, because a rung-4 decision changes your cost per user per month and therefore your margin.

### Words to own

| Term | Plain meaning | Example |
|---|---|---|
| Customization ladder | Ordered ways to make a model fit your product | The seven rungs |
| Build cost | One-off effort to get a rung working | Two weekends |
| Maintenance cost | Recurring work to keep it working | Retrain per base version |
| Reversibility | How fast you can undo the decision | Prompt: minutes |
| Catastrophic forgetting | Getting worse at untested tasks after tuning | Code quality drops |
| Capability vs knowledge | Behaviour you can train vs facts you must supply | Style vs today's fee |
| Base model version | The exact model your tune was built on | Deprecated in a year |
| Lock-in | How hard it is to leave this choice behind | A custom hosted model |
| Falsifiable test | A check that could prove you wrong | "Score below 0.8 on 40 cases" |
| Exit criterion | What would make you climb back down | Retrieval reaches 0.9 |

### Tradeoffs and gotchas

**"Fine-tune it" is the most common wrong answer in this subject.** The reason is that it sounds like the serious option. Ask one question first: is the thing wrong a *fact* or a *behaviour*? Facts go to retrieval. Formats and style go to fine-tuning. If the answer is "both", you need rung 3 and possibly rung 4, in that order, measured separately.

**Fine-tuning and retrieval are not alternatives.** The strongest setup is often a small fine-tuned model that reliably produces your format, reading retrieved facts at request time. Teams present these as a choice because the ladder looks like a staircase; it is better read as layers.

**A fine-tune is a hostage to a base model's lifecycle.** When the provider retires the version you tuned, your artefact stops being servable and the work must be redone against a new base. Open-weight models you host yourself avoid this and swap the problem for an inference bill, which is chapter 7.

**You cannot evaluate a rung you have not instrumented.** Climbing without M10 chapter 12's eval set means never knowing whether rung 4 beat rung 3, and the usual outcome is a team that believes the expensive thing helped because it cost more.

**Write the exit criterion at the same time as the decision.** "We will drop the fine-tune if retrieval plus a prompt reaches within two points of it on our 40 cases" is a sentence that saves a future month. Without it, a rung-4 decision becomes permanent by inertia.

**Cost per request can invert the ranking.** Hosting your own small fine-tuned model is sometimes dramatically cheaper per request than a large general model with a long prompt, which makes rung 4 the economical choice at volume even when a lower rung works. That is a legitimate reason to climb — and it is a cost argument, so it needs M09 chapter 14's arithmetic, not a vibe.

### Verify it

The AI will tell you which rung to use, confidently, in its first reply. Make it earn that.

1. **Make it classify your problem before proposing anything.** Fact, behaviour, format, capability, or cost. Those five words map onto the ladder, and the mapping is the whole decision.
2. **Demand the evidence that the rung below failed.** Specifically: how many prompt versions, what the eval score was for each, and which failure cases survived. "Prompting did not work" without numbers means rung 1 was not attempted.
3. **Ask for the maintenance story, not the build plan.** What breaks this, who notices, and how often must it be redone? If the answer does not mention base-model deprecation for rungs 4 and above, the plan is incomplete.
4. **Ask for the exit criterion in a sentence.** No exit criterion means no decision, just a direction.
5. **Ask what gets worse.** Fine-tuning for your format often degrades general reasoning or multilingual handling. Demand a named list of capabilities to re-test after tuning.
6. **Check the per-request cost both ways.** A rung-4 answer that is cheaper per request is a real argument; a rung-4 answer that doubles cost needs a quality number to justify it.

Red flags: a fine-tune proposed for data that changes; a fine-tune proposed before any eval set exists; "we need more training data" when the problem was a missing tool call; a plan with no base-model version recorded; no list of capabilities to re-test; a model swap never tried; and a rung chosen because it is more impressive to describe at demo day.

## Walkthrough
1. Write down your single worst AI quality failure as a specific example: the input, what came out, and what should have come out. One case, not a category.
2. Classify it in one word — fact, behaviour, format, capability, or cost — and write the one sentence that justifies the word.
3. Try rung 0: run the same input through two other models of different sizes. Record all three outputs. If one is correct, your problem was model choice.
4. Try rung 1 properly: write five genuinely different prompt versions, including one with three few-shot examples. Score every version against your eval set from M10 chapter 12, not against your impression.
5. Try rung 2: ask whether any part of the correct answer could have been computed or constrained instead of generated. If yes, add the tool or the schema and re-score.
6. Try rung 3 if the failure involves any fact you own: check whether the fact is in your store at all, and whether retrieval returned it. M10 chapter 12's ladder tells you which layer failed.
7. Now build the four-cost table for your own case: for rungs 1, 3 and 4, write your own estimates of build hours, per-request cost, maintenance work and time to undo. Use real rupee figures from M09 chapter 14's arithmetic.
8. Write the falsifiable test that would prove you need rung 4: a specific score, on a specific set, that rungs 1 to 3 failed to reach after real attempts.
9. Write the exit criterion that would send you back down, with a number in it.
10. Record the whole thing as one decision-log entry: the classification, what you tried at each rung with scores, the four-cost table, the falsifiable test, and the exit criterion. This entry is what chapter 4 will act on.

## Talk to AI
```prompt
CONTEXT:      I am a student on an AI-native engineering program. I direct AI
              and verify its output; I do not write code by hand. My team of
              three runs [product in two sentences] with [number] real users in
              India. I already know the difference between prompting,
              retrieval and fine-tuning, and that retrieval fixes facts while
              fine-tuning fixes behaviour, so do not re-teach that. I have an
              eval set of [number] real cases with expected answers. My worst
              remaining failure is: input [paste it], output I got [paste it],
              output I needed [paste it]. So far I have tried: [list honestly
              what you tried, with eval scores if you have them]. My current
              cost per request is about ₹[number] and my revenue per paying
              user per month is about ₹[number].

GOAL:         Decide, with evidence, which rung of the customization ladder my
              problem belongs on, and write a decision-log entry I could
              defend to a reviewer in three months.

REQUIREMENTS: 1. First classify my failure as one of: a fact, a behaviour, a
                 format, a missing capability, or a cost problem. Justify the
                 word in one sentence using my example
              2. Tell me, for my classification, which rung is correct and
                 which rungs are definitely wrong and why
              3. For the rung below your recommendation, give me three
                 specific things I have not tried yet, and the eval score that
                 would prove that rung exhausted
              4. Build me a four-column cost table for rungs 1, 3 and 4
                 applied to MY case: build cost in hours, per-request cost in
                 rupees, maintenance work and how often, and time to undo
              5. For any rung at 4 or above, tell me what happens when the
                 base model version I tune is deprecated, and what the work
                 to recover looks like
              6. List the specific capabilities I must re-test after
                 fine-tuning that have nothing to do with my target task,
                 because tuning can quietly degrade them
              7. Write me a falsifiable test: a number, on a named set, that
                 would prove the lower rung insufficient
              8. Write me an exit criterion with a number in it that would
                 make me climb back down

CONSTRAINTS:  Free tier for experimentation; I can spend at most a few hundred
              rupees of GPU time. Plain English for a second-language reader.
              My facts change at least monthly. I cannot afford a dedicated
              GPU running all month. Treat all prices as stale and tell me to
              check them.

NON-GOALS:    Do not write any training code yet. Do not recommend a rung
              without naming the evidence that the rung below it failed. Do
              not propose continued pretraining or training from scratch. Do
              not re-explain what RAG is.

ACCEPTANCE:   - My failure is classified in one of the five words, with a
                justification I agree with
              - Three untried things at the rung below are named specifically
              - The cost table has rupee and hour figures for my case, not
                generic ones
              - The maintenance row for rung 4 mentions base-model
                deprecation explicitly
              - A falsifiable test exists with a number and a named set
              - An exit criterion exists with a number in it
              - A list of capabilities to re-test after tuning exists

ASK:          Start by telling me which of the five words my failure is, and
              say what evidence in my message made you choose it. Then tell me
              whether you believe I have genuinely exhausted the rung below.
              Do not recommend a rung until you have done both.
```

## Stuck? Ask AI
```prompt
Give me eight short scenarios where an AI feature is behaving wrongly in a product. For each one, do not tell me the answer. Ask me which rung of this ladder it belongs on — pick a different model, prompt and examples, tools and structure, retrieval, supervised fine-tuning, or preference tuning — and ask me to justify it in one sentence. Then tell me whether I was right and what evidence would have settled it faster. Include at least two scenarios where the obvious answer is wrong.
```

```prompt
Explain, with a worked example from a student product, why fine-tuning a model on a company handbook does not reliably make it answer questions about the handbook correctly. Cover what the training actually changes, why a changed fee in September is still wrong afterwards, and what the right architecture is instead. Then explain the one situation where fine-tuning plus retrieval together is better than retrieval alone.
```

```prompt
I am about to fine-tune a model to make it produce our rigid output format. Before I start, tell me everything this decision costs me that is not the training run itself: hosting, the base model being deprecated, capabilities that might quietly get worse, how long it takes to undo, and what happens to my cost per request. For each one, tell me the specific check or measurement I should take before and after so I can prove what changed.
```

## Quiz

### Q1 · L3
Your assistant quotes an out-of-date hostel fee. Which rung is correct?
- (a) Supervised fine-tuning on the handbook
- (b) Retrieval, because the fee is a fact that changes ✅
- (c) Preference tuning on good and bad answers
- (d) Continued pretraining on college documents
> Facts that change must be supplied at request time from a source you can edit. A fine-tune bakes July's fee into weights and must be redone whenever the fee moves.

### Q2 · L4
Which cost do student teams most often ignore when choosing to fine-tune?
- (a) The cost of the GPU hours for the training run
- (b) The maintenance cost of redoing the tune when the base model is retired ✅
- (c) The cost of writing the training script
- (d) The cost of the eval set
> Build cost is visible and one-off; maintenance is invisible and permanent. A tuned artefact is tied to one base-model version, and deprecation is routine rather than rare.

### Q3 · L4
After fine-tuning for your output format, which check matters most?
- (a) That the training loss went down
- (b) That the format is now correct on the training examples
- (c) That capabilities unrelated to your task did not get worse ✅
- (d) That the model file is smaller than before
> Tuning moves all the weights, so skills you never tested can degrade — that is catastrophic forgetting. Re-test reasoning, other languages and your general eval set, not only the format.

### Q4 · L4
Your team says "prompting did not work, so we are fine-tuning." What single piece of evidence makes that claim checkable?
- (a) The number of hours spent prompting
- (b) The eval scores of several distinct prompt versions on a fixed case set ✅
- (c) A screenshot of the best output so far
- (d) The size of the model being prompted
> Without a fixed set and per-version scores, "prompting did not work" is a feeling. With them, it is a measurement that justifies a two-week commitment.

### Q5 · L5
A team's small fine-tuned model matches their large prompted model on quality and costs a fifth as much per request. Is climbing the ladder justified?
- (a) No, because a lower rung produced the same quality
- (b) Yes, if the cost saving is measured and the maintenance burden is accepted ✅
- (c) Yes, because smaller models are always preferable
- (d) No, because fine-tuning should only ever fix quality problems
> Cost is a legitimate reason to climb, and at volume it is often the strongest one. It still has to be a measured rupee figure, with the deprecation and re-test burden written down and accepted.

## Go deeper
- [Anthropic: strengthen guardrails and keep a model on task](https://docs.anthropic.com/en/docs/test-and-evaluate/strengthen-guardrails/keep-claude-in-character)
- [Hugging Face: when to fine-tune, and alternatives to consider first](https://huggingface.co/docs/transformers/en/training)
