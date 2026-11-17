---
id: m14-04-sft-lora-qlora
module: m14-deeper-ml-agents
slug: sft-lora-qlora
title: "SFT, LoRA and QLoRA in Practice"
order: 4
duration_min: 25
is_free: false
mission: true
volatility: medium
last_reviewed: 2026-11-16
---

## Key learnings
- Run a QLoRA fine-tune of a small instruct model on a free notebook GPU
- Build the example file in the right shape, with the loss applied to the answer only
- Set the four knobs that matter — rank, target modules, learning rate, epochs
- Prove the tune beats a well-prompted baseline on a fixed eval set, or delete it
- Catch a tune that learned your format and lost the ability it used to have

## Lesson
### The pain

M09 chapter 11 put LoRA in a glossary noting "practised in M14"; chapter 3 decided whether you belong on this rung. New here is the practice — and the hard part is the example file, not the training.

A team tunes 900 house-style support replies with QLoRA on a free notebook GPU; loss falls from 2.1 to 0.4. Then they use it: every reply is in house style, including the ones it should have refused, and "15% of ₹240" now says ₹42. Three defects, none visible in the loss curve — loss computed over prompts too, every example from one month, and no prompted baseline.

### The idea

**A fine-tune is chapter 1's loop with almost every weight frozen, and the thing you are really designing is the example file.**

Full fine-tuning updates every weight and needs several times the model's size in memory. **LoRA** freezes the base and trains two small matrices beside each targeted layer, producing an **adapter** of a few megabytes. **QLoRA** holds the frozen base in 4-bit, halving memory again — which is what fits a 1-to-3-billion-parameter instruct model inside a free notebook GPU.

### The analogy

The canteen hires a cook who knows three hundred dishes; you need every plate presented one way. Full fine-tuning sends the cook away for six months. LoRA is a laminated card clipped to the counter: skills untouched, one thin layer at plating time. QLoRA reprints the recipe book smaller to fit your shelf.

**Where it breaks.** The card is clipped outside; an adapter sits inside the cook's hands — train it on plating and tasting gets worse.

### Picture it

```
  FROZEN BASE MODEL (4-bit)               TRAINED (16-bit)
  ┌─────────────────────────┐            ┌──────────────┐
  │ attention layer         │──────┬────▶│  A  (r × d)  │
  │  2.0 GB of weights,     │      │     │  B  (d × r)  │
  │  never updated          │      │     │  ~8 MB total │
  └─────────────────────────┘      │     └──────┬───────┘
            │                      │            │
            └──── output ──────────┴──── + ◀────┘
                                        adapter
                                        contribution

  memory: base 2.0 GB + adapter 8 MB + optimizer state for 8 MB
  versus full fine-tuning: 4 GB weights + 8 GB optimizer + more
```

The base model's weights stay frozen in 4-bit while two small matrices beside each targeted layer are trained and added to that layer's output, which is why the whole run fits in a fraction of the memory a full fine-tune needs.

### The example file is the project

- **Shape** — instruction and wanted response, rendered through the exact chat template of the model you are tuning
- **Loss on the answer only** — mask the prompt tokens, or half the capacity learns to generate questions
- **Volume and diversity** — 400 rows across every category beat 4,000 from one month
- **Include deliberately** — refusals, very short and very long inputs, every user language, and the cases your baseline already fails

### The four knobs

- **Rank** — start at 8 or 16; raise only if the eval says underfitting
- **Target modules** — attention projections usually; moves results more than rank does
- **Learning rate** — around one ten-thousandth, higher than full tuning; worth a sweep
- **Epochs** — one to three; beyond that a small file is memorised

### Tools today

A free notebook GPU with roughly 15 GB handles QLoRA on an open-weight instruct model of 1 to 3 billion parameters comfortably, and 7 to 8 billion at short sequence lengths with patience. The ecosystem is Hugging Face Transformers with PEFT for adapters and bitsandbytes for 4-bit loading, plus a trainer wrapper that handles chat templates and answer-only loss masking. All those names and every free-tier limit change — check them this week.

### Where it lives

The artefact is an adapter loaded beside a base model at serving time — swappable per customer — or one merged file.

- Chapters 1 to 3 — loop, data, decision
- Chapter 5 — the preference rung above
- Chapters 6, 7 — quantizing and serving
- Chapters 10, 13 — regression suite, versioning

### Words to own

| Term | Plain meaning | Example |
|---|---|---|
| SFT | Supervised fine-tuning on instruction and response pairs | 900 support replies |
| Full fine-tune | Every weight updated | Needs a big GPU |
| LoRA | Freeze the base, train two small matrices beside layers | 8 MB adapter |
| QLoRA | LoRA with the frozen base held in 4-bit | Fits a free GPU |
| Adapter | The small trained file added to a base model | Swappable |
| Rank | How big those small matrices are | 8 or 16 to start |
| Target modules | Which layers get an adapter | Attention projections |
| Chat template | The exact turn markers the model expects | Must match at serving |
| Loss masking | Scoring only the answer tokens, not the prompt | Mask the instruction |
| Merging | Folding the adapter back into the base weights | One file to serve |
| Catastrophic forgetting | Losing skills you never tested | Arithmetic gets worse |
| Prompted baseline | The same task done well with prompting only | What you must beat |

### Tradeoffs and gotchas

**You are not allowed to skip the prompted baseline.** Run your best prompt over the same eval set first. A tune that does not beat it visibly is two weekends you should take back.

**A falling loss is not a result.** Loss measures agreement with your example file, mistakes and narrowness included. Only the eval score counts, and it often moves the other way.

**The chat template must be byte-identical at training and serving.** An adapter without its base version and data file cannot be reproduced.

### Verify it

AI will produce a working QLoRA script in one reply. Verifying it is six checks.

1. **Find the loss masking** and demand the line — prompts included means the tune learned the wrong half.
2. **Print one rendered training example** and compare it to the serving prompt.
3. **Demand the prompted baseline score before training.**
4. **Ask which modules were targeted and why** — a config that cannot justify its targets was copied.
5. **Ask for the untouched-capability check** — arithmetic, a refusal, another language, scored before and after.
6. **Check the file against chapter 2:** spread, duplicates, refusals, languages.

Red flags: a loss curve as the result; no prompted baseline; an eval set from the training file's slice; a chat template differing between notebook and app; rank 64 "for quality".

## Walkthrough
1. Open your chapter 3 decision-log entry and re-read the falsifiable test you wrote. The number in it is what this chapter has to beat.
2. Freeze your eval set first: 30 to 40 real cases with expected outputs, including refusals, long inputs, and at least two languages. Never edit it again during this chapter.
3. Score your best prompted setup against that eval set and write the number on the board. This is the baseline. Record the cost per request too.
4. Build the example file from chapter 2's rules: 500 or more rows, instruction and response, spread across every category, with refusals and edge cases deliberately included. Version the filename.
5. Render one example fully through the model's chat template and read it out loud. Confirm the turn markers, the system message and the response boundary are exactly what you will send at serving time.
6. In a free notebook with a GPU, load a 1-to-3-billion-parameter open-weight instruct model in 4-bit and attach a LoRA adapter at rank 8 on the attention projections. Print the number of trainable parameters and the memory used.
7. Confirm loss masking: print the label tensor for one example and check that the instruction positions are masked out. Fix it before training if they are not.
8. Train for one epoch with a learning rate around one ten-thousandth, logging training and validation loss. Save the adapter. Note the wall-clock time and whether the session held.
9. Run two more configurations, changing exactly one thing each: a ten-times-smaller learning rate, then two epochs. Keep all three adapters.
10. Score all three adapters and the prompted baseline against the frozen eval set, in one table. Add a second table for the untouched capabilities — arithmetic, a refusal, another language — scored before and after.
11. Decide: keep the best adapter, or delete all three and say so. Write the decision, the two tables, and the adapter's full configuration into your decision log.

## Talk to AI
```prompt
CONTEXT:      I am a student on an AI-native engineering program. I direct AI
              and verify its output; I do not write code by hand. I have
              already run a training loop, I know what loss and epochs are,
              and I have decided from a cost and maintenance comparison that
              my problem belongs on the fine-tuning rung, so do not re-teach
              the ladder or gradient descent. My team of three runs [product
              in two sentences] with real users in India. The behaviour I need
              is: [describe the format or style the model must hold, in two
              sentences]. I have [number] example pairs, spread across
              [describe the categories and the date range honestly]. My best
              prompted setup scores [number] on a frozen eval set of [number]
              real cases. Hardware: a free notebook GPU with about 15 GB.

GOAL:         One QLoRA adapter on a small open-weight instruct model that
              beats my prompted baseline on the frozen eval set, with proof
              that nothing unrelated got worse.

REQUIREMENTS: 1. Tell me what shape my example file must be in, and show me
                 one row rendered fully through the chat template of the model
                 you recommend, with every turn marker visible
              2. Show me the exact configuration line that makes the loss
                 apply to the response tokens only, and show me how to print
                 the label tensor to prove the instruction is masked
              3. Recommend a model size and name for 15 GB with 4-bit
                 loading, and tell me the longest sequence length that fits
              4. Give me starting values and a reason for each of: rank,
                 target modules, learning rate, epochs. Tell me which one to
                 sweep first and the three values to try
              5. Tell me how to print the trainable parameter count and the
                 GPU memory used, so I can prove the adapter is small
              6. Give me a before-and-after check for capabilities I am NOT
                 tuning: list six specific prompts covering arithmetic, a
                 refusal, another Indian language and a reasoning step
              7. Tell me how to save the adapter, what metadata I must record
                 with it, and the difference between serving it beside the
                 base and merging it in — including which to choose if I want
                 different behaviour per customer
              8. Tell me the three symptoms that would mean I should delete
                 this adapter rather than tune it further

CONSTRAINTS:  Free tier only, no paid GPU, and the notebook session can die
              without warning so checkpoint to mounted storage. Plain English
              for a second-language reader. Label every cell READ-ONLY or
              CHANGES-THINGS. My users write in English and [language], so
              both must survive the tune. Treat every library name, model name
              and free-tier limit as stale and tell me to check it.

NON-GOALS:    Do not do a full fine-tune. Do not propose preference tuning or
              reinforcement learning. Do not re-explain LoRA's mathematics
              beyond what I need to set the knobs. Do not touch my eval set.
              Do not suggest collecting more examples until the first run is
              scored against the baseline.

ACCEPTANCE:   - One rendered training example and one rendered serving prompt
                are character-identical in their template
              - I can see the line that masks the instruction from the loss,
                and the printed label tensor confirms it
              - Trainable parameters are under 1% of the model's total
              - Three runs differ in exactly one setting each
              - All three adapters and the prompted baseline appear in one
                table scored on the same frozen eval set
              - Six untouched-capability prompts are scored before and after
              - The adapter is saved with its base model version, rank,
                target modules and data file version recorded

ASK:          First, look at my example count and category spread and tell me
              whether this file can support a fine-tune at all, and what its
              most likely failure will be. Then tell me what score I need to
              beat. Only after that give me the configuration.
```

## Mission

**Objective.** Produce one QLoRA adapter for your own product's task, scored against a well-prompted baseline on a frozen eval set, with a written decision to keep it or delete it.

**Steps.**
1. **Frame** — state the behaviour you need in one sentence, and say why chapter 3's lower rungs could not hold it. Quote the scores.
2. **Freeze the eval set** — 30 to 40 real cases including refusals, long inputs and every language your users write in. Commit it and do not touch it again.
3. **Baseline** — score your best prompted setup on that set, and record its cost per request.
4. **Build the example file** — 500 or more pairs from chapter 2's rules, versioned, with the category spread written down.
5. **Brief** — write a Director's Brief covering template rendering, answer-only loss masking, the four knobs, and the untouched-capability check.
6. **Direct** — run three configurations differing in one setting each, on a free notebook GPU, checkpointing to durable storage.
7. **Verify** — score all three adapters plus the baseline on the frozen set. Then score the six untouched-capability prompts before and after.
8. **Decide** — keep the best adapter only if it beats the baseline by a margin you can defend. Otherwise delete all three and write why.
9. **Document** — record the adapter's base model version, rank, target modules, learning rate, epochs and data file version, plus your two score tables.

**Upload.**
- Your Director's Brief → post caption or an attached file
- One fully rendered training example and the serving prompt, side by side → screenshot
- Proof of answer-only loss masking: the printed label tensor → screenshot
- Trainable parameter count and GPU memory used → screenshot
- Your scores table: three adapters plus the prompted baseline on the frozen eval set
- Your untouched-capability table, before and after
- The adapter configuration and its recorded metadata
- The AI conversation → share link in `ai_chat_url`
- Reflection, 150 words max → one tradeoff you chose, and one thing the AI got wrong

**Self-check.** Tick every box before you submit.
- [ ] My eval set was frozen before I trained anything
- [ ] I scored a well-prompted baseline and wrote the number down first
- [ ] My example file has 500 or more rows with a recorded category spread
- [ ] My training template and serving template are character-identical
- [ ] I can point at the line that masks the instruction out of the loss
- [ ] Trainable parameters are under 1% of the model's total
- [ ] Three runs differ in exactly one setting each
- [ ] I scored six capabilities I did not tune, before and after
- [ ] My decision names a number, not an impression
- [ ] The adapter's base model version and data version are recorded
- [ ] I am willing to say out loud whether this was worth two weekends

**Stretch.** Train a second adapter on a deliberately narrow slice — one category, one month — and score it on the full eval set beside your good one. The gap is what dataset diversity bought you, in your own numbers.

## Stuck? Ask AI
```prompt
Here is my QLoRA training script and one row of my example file: [paste both]. Do not improve them. Instead: quote the line that decides which tokens the loss is computed over, and tell me whether my instruction tokens are included. Then render my example row fully through the chat template the script uses and show me the exact string, every turn marker visible. Finally, tell me what the serving code would have to send for that string to match, and name anything that could differ.
```

```prompt
My fine-tune's training loss fell from 2.1 to 0.4 and my eval score went down. Explain how both can be true at the same time, using my situation: [describe your example file and your eval set in three sentences]. Give me the four most likely causes in order of probability, and for each one the single check that confirms or eliminates it. Do not propose fixes until you have given me the checks.
```

```prompt
Explain catastrophic forgetting in a LoRA fine-tune as if to someone who has never trained a model, including why adapting the attention layers for my output format can make the model worse at arithmetic. Then give me a before-and-after test set of eight prompts covering abilities I am not trying to change, tell me what a degradation would look like in each, and tell me which three knobs reduce the risk and what each one costs me.
```

## Quiz

### Q1 · L3
What does QLoRA add to LoRA?
- (a) It trains more of the model's layers
- (b) It holds the frozen base model in 4-bit so the run fits in less memory ✅
- (c) It removes the need for an eval set
- (d) It merges the adapter automatically at the end
> LoRA already freezes the base and trains small matrices. QLoRA quantizes that frozen base to 4-bit, which is what brings a few-billion-parameter model inside a free notebook GPU.

### Q2 · L4
Your loss is computed over both the instruction and the response tokens. What is the consequence?
- (a) Training runs out of memory sooner
- (b) Part of the model's learning goes into generating user-style questions ✅
- (c) The adapter file becomes larger
- (d) The chat template stops matching
> Every token the loss covers is a token the model is being taught to produce. Including prompts spends capacity on imitating your users instead of answering them, and the loss curve looks perfectly healthy.

### Q3 · L4
After a successful tune, which check is most likely to reveal a problem nobody expected?
- (a) Re-reading the training loss curve
- (b) Checking the adapter file size
- (c) Scoring six tasks unrelated to what you tuned ✅
- (d) Counting the trainable parameters
> The adapted layers are the same layers that did everything else, so arithmetic, refusals and other languages can quietly degrade. Score them before and after or you will not know.

### Q4 · L4
Your tuned model scores 0.86 on the eval set. What single number decides whether the two weekends were worth it?
- (a) The final training loss
- (b) The score of your best prompted setup on the same eval set ✅
- (c) The number of training examples used
- (d) The rank you chose
> 0.86 against a 0.84 prompted baseline is a rounding error carrying a permanent maintenance burden. Against 0.61 it is a real result. The baseline has to be measured before you train.

### Q5 · L5
A team's tuned model works in the notebook and is noticeably worse in the app, with no code changes to the model. What is the first suspect?
- (a) The adapter was not merged into the base
- (b) The chat template rendered at serving differs from the one used in training ✅
- (c) The rank was too low
- (d) The GPU in production is slower
> A tune is conditioned on the exact turn markers it saw. A different system-message placement or a missing marker at serving looks like nothing and costs most of the gain, so compare the two rendered strings character by character.

## Go deeper
- [QLoRA: efficient finetuning of quantized LLMs, the original paper](https://arxiv.org/abs/2305.14314)
- [Hugging Face PEFT: LoRA configuration and adapter handling](https://huggingface.co/docs/peft/index)
