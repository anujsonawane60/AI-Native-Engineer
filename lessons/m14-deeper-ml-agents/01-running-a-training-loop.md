---
id: m14-01-training-loop
module: m14-deeper-ml-agents
slug: running-a-training-loop
title: "Running a Real Training Loop"
order: 1
duration_min: 25
is_free: false
mission: true
volatility: low
last_reviewed: 2026-11-16
---

## Key learnings
- Run a complete training loop on a free notebook GPU and read its output honestly
- Diagnose a run from its two loss curves: not learning, learning, or memorising
- Make a run repeatable by fixing the seed, the split, the data version and the environment
- Fit a run onto a 15 GB GPU using batch size, gradient accumulation and mixed precision
- Catch a score that was measured on the data the model trained on

## Lesson
### The pain

M09 chapter 6 taught the loop as an idea: a batch goes forward, a loss comes out, backpropagation assigns blame, weights are nudged downhill. New here is what the idea leaves out — running one on borrowed hardware with a time limit.

A team trains a support-message classifier. Attempt one fails: model on the GPU, data not. Attempt two dies out of memory. Attempt three disconnects at minute thirty-eight, saving nothing. Attempt four prints `accuracy: 0.991`, computed on the rows it trained on; honestly it is 0.62.

### The idea

**A training run is an experiment, and an experiment you cannot repeat and cannot audit is not a result.**

Three things make it an experiment rather than an event. **Repeatable**: same seed, same data version, same numbers. **Checkpointed**: weights hit disk often enough that a disconnect costs minutes. **Honestly scored**: your quoted number comes from data the model never saw, opened once.

### The analogy

You teach a kitchen helper to sort orders into four trays from a box of three hundred the senior cook already judged. Thirty-two at a time, then a correction — a **batch** and a **step**. Box exhausted, shuffle, repeat — an **epoch**. A smaller box checks progress but never coaches — the **validation split**. A sealed box waits in the manager's drawer — the **test split**.

**Where it breaks.** A human asks "why was that wrong?" and gets a reason; the model gets only a direction and a size per weight. It cannot tell "refund request" from "arrived at 2 am", so if those correlate it learns the wrong one.

### Picture it

```
  DATA (one versioned file)
    │
    ├── train 70%  ─────────────────┐
    ├── val   15%  ──────┐          │
    └── test  15%  ─┐    │          │
                    │    │          ▼
                    │    │   ┌──────────────┐
                    │    │   │ for epoch:   │
                    │    │   │  for batch:  │
                    │    │   │   forward    │
                    │    │   │   loss       │
                    │    │   │   backward   │
                    │    │   │   step       │
                    │    │   └──────┬───────┘
                    │    │          │ every epoch
                    │    └──────────▶ val loss  ──▶ CHECKPOINT
                    │                                (keep the best)
                    │
                    └──── opened ONCE, at the very end ──▶ the number
                                                            you quote
```

One versioned file splits three ways: the loop touches only the training split, validation decides which checkpoint to keep, and the test split is opened once to produce the only score you may quote.

### The eight things in every loop

Read any training script by finding these eight.

1. **Seed everything** — shuffle, split, initialisation
2. **Load and split** — one versioned file, split in code from that seed
3. **Same device** — weights and batch both inside GPU memory
4. **Optimizer** — in practice AdamW: more memory per weight, less tuning
5. **Schedule** — rate climbs from near zero (**warmup**), then decays
6. **Inner loop** — forward, loss, backward, step, zero the gradients, log
7. **Validate each epoch, gradients off, evaluation mode on** — miss that switch and the number is wrong unpredictably
8. **Checkpoint** — on every validation improvement, to storage that outlives the session

### Tools today

Free notebook GPUs exist on Google Colab and Kaggle Notebooks: a usable session at no cost, roughly 15 GB of video memory, limits that have changed more than once. Use a small open-weight encoder of 20 to 70 million parameters and a public dataset of a few thousand labelled rows. Check every quota and model name this week rather than believing a lesson.

### Where it lives

It runs in a notebook beside your product, never inside it.

- Chapter 2 — the dataset
- Chapter 3 — whether to train at all
- Chapter 4 — this loop with few weights unfrozen
- Chapter 10 — your test score as a regression suite
- Chapter 13 — your results table as experiment tracking

### Words to own

| Term | Plain meaning | Example |
|---|---|---|
| Run | One complete execution of a training script | Run 7, seed 42 |
| Device | Where the numbers physically live | `cuda` or `cpu` |
| Optimizer | The rule that turns blame into a weight change | AdamW |
| Warmup | Learning rate starts near zero and climbs | First 300 steps |
| Mixed precision | 16-bit numbers for most of the maths | Half the memory |
| Gradient accumulation | Add blame over several batches, step once | 4 × 8 acts like 32 |
| Out of memory (OOM) | The GPU ran out of room and the run died | Cut the batch size |
| Evaluation mode | Switching off training-only layer behaviour | Before every val pass |
| Seed | The fixed number that makes a run repeatable | `seed = 42` |
| Run log | The recorded settings and numbers of one run | One row per run |

### Tradeoffs and gotchas

**Fitting 15 GB costs you something every time.** Cutting batch size is the first out-of-memory fix, but halving it usually means halving the learning rate. Gradient accumulation buys a big batch on a small GPU — four batches of 8, one nudge — without buying speed. Mixed precision halves memory. Past 15 GB only a smaller model saves you: chapter 4.

**The first number is wrong in the flattering direction.** A score on the training split, a validation split re-drawn after a bad result, a test split peeked at twice — all three inflate, all three by accident.

**A converged run can still be useless, and longer training is worse.** Falling loss means a pattern, not your pattern — check it against the baseline. Past the turn in validation loss, more epochs make the product worse. Early stopping and best-checkpoint are free, and AI-written scripts omit both.

### Verify it

AI will write you a training script that runs. Running is not the bar.

1. **Find the line computing the reported metric and name its split.** If it is training data, every number is fiction — the commonest defect in AI-generated training code.
2. **Demand the stupid baseline first.** Without it there is no experiment.
3. **Find the seed, then run twice.** Same seed, same losses — otherwise every comparison is noise.
4. **Check three one-line omissions:** evaluation mode and gradients off during validation, and a checkpoint path surviving the session.
5. **Demand both curves and the run log.** Training loss alone cannot tell learning from memorising.

Red flags: accuracy on the training set; a split re-drawn after a bad result; no seed; no baseline; the last checkpoint kept.

## Walkthrough
1. Open a free notebook environment, select the GPU runtime, and run one cell that prints the GPU name and its total memory. Write both down. Everything you plan must fit inside that number.
2. Mount durable storage and create a folder for this run's checkpoints and logs. Confirm by writing a file, restarting the runtime, and reading it back. Do this before you train anything.
3. Pick a small public labelled text dataset of a few thousand rows and a small encoder model. Load the data and print the number of rows per label. If one label is more than half the rows, write that down — it is your baseline.
4. Compute and record the stupid baseline: the score you get by always predicting the most common label. Every later number is compared against this one.
5. Split the data 70/15/15 in code, from a fixed seed. Print the row counts and the label distribution of each split. Then check for duplicate rows appearing in more than one split and record how many you found.
6. Write the training script with an AI, then read it and locate the eight things from this chapter by name. Mark in a comment which line is each one. Anything you cannot find is probably missing.
7. Deliberately break it once: run with batch size large enough to run out of memory. Read the whole error. Then fix it with a smaller batch plus gradient accumulation and mixed precision, and record the memory used before and after.
8. Train with a fixed seed for enough epochs that validation loss turns upward. Log training and validation loss every epoch to a file in durable storage, and save a checkpoint whenever validation improves.
9. Plot both curves and mark the epoch where validation turned. Confirm that the checkpoint you kept is from that epoch and not the last one.
10. Run the same script again with the same seed and confirm the losses match. Then run it twice more at learning rates ten times larger and ten times smaller, and add all four rows to a results table with the settings and the validation score.
11. Only now, once, load the best checkpoint and score the test split. Put that number beside your baseline from step 4 and write one sentence on whether two hours of GPU time bought anything.

## Talk to AI
```prompt
CONTEXT:      I am a student on an AI-native engineering program. I direct AI
              and verify its output; I do not write code by hand. I have
              already learned what a loss, a gradient, a learning rate, a
              batch and an epoch are, and how to read two loss curves, so do
              not re-explain those. I am running my first real training loop
              in a free notebook GPU environment with roughly 15 GB of video
              memory and a session that can disconnect without warning. Task:
              [classification task in one sentence]. Data: [number] labelled
              rows in one file, label counts [paste them]. Model: a small
              open-weight encoder of roughly [number] million parameters.

GOAL:         One training run that is repeatable, checkpointed and honestly
              scored, plus a results table with four rows I can defend.

REQUIREMENTS: 1. A single script with the eight parts labelled in comments:
                 seeding, load and split, device placement, optimizer,
                 learning-rate schedule with warmup, inner loop, validation
                 with gradients off and evaluation mode on, checkpointing
              2. Before any training, compute and print the majority-class
                 baseline score, and tell me what number my model must beat
                 to have been worth running
              3. Make the 70/15/15 split in code from one seed, print the
                 row count and label distribution per split, and report how
                 many duplicate rows appear in more than one split
              4. Checkpoint to a path on mounted durable storage, keeping the
                 best validation epoch and not the last one, and log losses
                 to a file as well as to the cell output
              5. Fit the run in memory with mixed precision and gradient
                 accumulation. Tell me the batch size to start at, how to
                 read the out-of-memory error, and what to halve first
              6. Tell me exactly which line computes my reported metric and
                 which split that data came from
              7. Give me a results table format with one row per run: seed,
                 learning rate, batch size, epochs, best validation score,
                 test score if opened, and what I changed from the run above
              8. Tell me three specific symptoms that would mean my run is
                 learning the wrong pattern rather than failing to learn

CONSTRAINTS:  Free tier only, no paid GPU. The session may die at any time.
              Plain English for a second-language reader. Label every cell
              READ-ONLY or CHANGES-THINGS. Assume slow and patchy internet, so
              anything downloaded must be cached. Do not use a model that
              cannot fit in 15 GB with training memory included.

NON-GOALS:    Do not re-teach gradient descent, backpropagation or what a loss
              is. Do not suggest a paid GPU, a distributed setup, or a model
              larger than my stated size. Do not tune the architecture. Do not
              touch the test split anywhere except one final cell.

ACCEPTANCE:   - Two runs with the same seed produce the same losses
              - I can point at the line where my metric is computed and name
                the split it used
              - Validation loss turns upward during the run and the kept
                checkpoint is from before the turn
              - The checkpoint survives a runtime restart
              - My results table has four rows differing in one setting each
              - The test split was scored exactly once, at the end
              - My test score is stated beside the majority-class baseline

ASK:          First tell me, given my label counts, what the majority-class
              baseline is and what score would make this run worth doing. Then
              list the three most likely ways this run fails on a free
              notebook GPU. Only after that write the script.
```

## Mission

**Objective.** Produce one repeatable, checkpointed, honestly scored training run on a dataset drawn from your own startup, together with a four-row results table and a test score stated beside a baseline.

**Steps.**
1. **Frame** — name one small prediction your product could actually use: routing an incoming message to a team, flagging an order as likely to be cancelled, tagging a photo as usable or not. Write who would see the prediction and what happens when it is wrong.
2. **Build the labelled file** — export a few thousand real rows from your product and label them, or derive the label from behaviour you already record. One file, a version in its name, a column of labels. Print the label counts.
3. **Baseline** — compute the majority-class score and write it down. This is the number your run must beat.
4. **Brief** — write a Director's Brief for the training script, with the eight parts, a seeded split, durable checkpointing and the metric line identified.
5. **Direct** — have the script written, then read it and label the eight parts yourself in comments. Fix anything missing before running.
6. **Run** — train with a fixed seed until validation loss turns up. Then run three more times changing exactly one setting each time.
7. **Verify** — repeat one run with the same seed and confirm the losses match. Confirm your checkpoint survives a runtime restart. Confirm the metric is not computed on the training split.
8. **Score once** — load the best checkpoint, score the test split one time, and record it beside the baseline.
9. **Document** — one decision-log entry: whether this prediction is worth putting in front of a user, and the number that decided it.

**Upload.**
- Your Director's Brief → post caption or an attached file
- The notebook → exported file or screenshots of every cell's output
- Your label counts and the majority-class baseline → screenshot or text
- Both loss curves with the turning epoch marked → screenshot
- Your four-row results table → screenshot or text
- Proof of repeatability: two same-seed runs side by side → screenshot
- Your single test score beside the baseline
- The AI conversation → share link in `ai_chat_url`
- Reflection, 150 words max → one tradeoff you chose, and one thing the AI got wrong

**Self-check.** Tick every box before you submit.
- [ ] My data is one versioned file and my split is made in code from one seed
- [ ] I printed label counts for every split and checked for rows in two splits
- [ ] I wrote down the majority-class baseline before training anything
- [ ] I can point at the exact line that computes my reported metric
- [ ] That line uses a split the model never trained on
- [ ] Two runs with the same seed gave me the same losses
- [ ] My checkpoint survived a runtime restart
- [ ] I kept the best-validation checkpoint, not the last one
- [ ] My results table has four rows and each differs from the one above it in one setting
- [ ] I opened the test split exactly once
- [ ] I can say in one sentence whether my model beat the baseline enough to use

**Stretch.** Train one more run with the labels deliberately shuffled at random. A model that still scores well above the baseline on shuffled labels has a leakage problem in your pipeline, not a learning ability. Report what you found.

## Stuck? Ask AI
```prompt
Here is my training script: [paste it]. Do not improve it yet. Instead, tell me for each of these eight things whether it is present, and quote the line: seeding, the data split, device placement, the optimizer, the learning-rate schedule, the inner loop, validation with gradients off and evaluation mode on, and checkpointing. Then tell me which split my reported metric is computed on, quoting the line. Finally, list everything in this script that would make two runs with the same seed produce different numbers.
```

```prompt
My run died with an out-of-memory error. Here is the full error: [paste it]. Explain what ran out, why training needs several times more memory than running the same model, and give me the fixes in the order I should try them, with what each one costs me. For each fix, tell me how I prove it worked by measuring memory rather than by the run not crashing.
```

```prompt
Here are my training and validation losses per epoch: [paste the numbers]. Do not tell me what to change yet. First tell me which of these three the run is doing: not learning, learning, or memorising. Say which epoch I should have stopped at and why. Then ask me what my majority-class baseline score is, and only after I answer, tell me whether this run was worth the GPU time.
```

## Quiz

### Q1 · L3
Your script reports `accuracy: 0.991` after training. What is the first thing to check?
- (a) Whether the learning rate was too high
- (b) Which data split that number was computed on ✅
- (c) Whether mixed precision was enabled
- (d) Whether the optimizer was AdamW
> A near-perfect score on a real task is far more often a metric computed on the training split than a genuinely excellent model. Find the line, find the variable, name the split.

### Q2 · L3
A run dies with an out-of-memory error at batch size 64. Which fix keeps the gradient behaviour of a large batch while fitting in memory?
- (a) Reduce the number of epochs
- (b) Batch size 16 with gradient accumulation over 4 batches ✅
- (c) Switch the optimizer from AdamW to plain gradient descent
- (d) Move the model to the CPU
> Accumulating blame over four batches of 16 and stepping once is arithmetically close to one batch of 64. It does not make the run faster, but it makes it finish.

### Q3 · L4
Two runs of the same script with no changes give validation scores of 0.71 and 0.78. What does this tell you?
- (a) The second run is better and should be used
- (b) Something is unseeded, so you cannot yet compare any two configurations ✅
- (c) The learning rate is too low
- (d) The validation split is too small to matter
> Until a repeated run reproduces its own numbers, every comparison you make between settings is measuring randomness. Seed the shuffle, the split and the initialisation first.

### Q4 · L4
Your trained four-class classifier scores 0.74 on the test split. What single extra number decides whether the run was worth doing?
- (a) The final training loss
- (b) The total GPU minutes used
- (c) The score of always predicting the most common class ✅
- (d) The number of parameters in the model
> 0.74 is excellent against a 0.25 baseline and worthless against a 0.73 one. The majority-class score is free to compute and must be recorded before training starts.

### Q5 · L5
You shuffle your labels at random, retrain, and the model still scores far above the majority-class baseline. What have you found?
- (a) The model is unusually powerful
- (b) The learning rate schedule is working well
- (c) Leakage in the pipeline, so the score comes from something other than the labels ✅
- (d) Nothing — this is expected behaviour
> With meaningless labels, nothing learnable remains, so a high score means the evaluation is reading information it should not have — duplicate rows across splits, an index column, or the metric computed on training data.

## Go deeper
- [PyTorch: training a classifier, the official tutorial](https://pytorch.org/tutorials/beginner/blitz/cifar10_tutorial.html)
- [Google: the Deep Learning Tuning Playbook](https://github.com/google-research/tuning_playbook)
