---
id: m14-02-datasets
module: m14-deeper-ml-agents
slug: datasets-and-data-quality
title: "Datasets and Data Quality: The Part That Decides Everything"
order: 2
duration_min: 30
is_free: false
mission: false
volatility: low
last_reviewed: 2026-11-16
---

## Key learnings
- Write a labelling guide and measure whether two people applied it the same way
- Find the four defects that quietly cap your score: noise, duplicates, imbalance, contamination
- Split by time or by user when random splitting would leak the future
- Record provenance, licence and consent for every row before you train on it
- Catch a dataset whose quality nobody measured being blamed on the model

## Lesson

### The pain

M09 chapter 3 gave you splits, leakage and honest metrics. M10 chapter 9 gave you ingestion quality for documents you retrieve from. Neither covers the thing in front of you now: a file of labelled rows that you built, which decides the ceiling of everything in chapter 1 and chapter 4.

A team labels 1,800 support messages into four buckets over a weekend. Three people split the work. They train, and the score stalls at 0.68 whatever they do. Four weeks of model changes follow — a bigger model, more epochs, three learning rates, a different architecture — and the number moves by two points.

Then somebody takes 100 rows and relabels them independently. The three original labellers disagree with each other on 24 of them. "Complaint" and "refund request" were never defined, so each person drew the line somewhere else. On top of that, 140 rows are near-duplicates of each other because the product's template replies got exported too, and 61 of those duplicates have one copy in train and one in test — so the honest score was never 0.68 at all.

The ceiling was in the file. Four weeks of GPU time went into a model that was already as good as the labels allowed.

### The idea

**Your dataset is a measuring instrument, and an unmeasured instrument cannot be trusted no matter what it reads.**

A dataset is not "the data we have". It is a file with a version, a written definition of every label, a known rate of disagreement between labellers, a measured duplicate rate, a recorded split rule, and a note of where every row came from and whether you are allowed to use it. Building that takes a day. Not building it costs a month.

### The analogy

The canteen wants to know which dishes get complaints. Staff tick a box on a slip: good, average, bad.

Six weeks in the data says nothing. Investigate and you find the reasons. Nobody defined "average", so one server ticks it for anything not praised and another never ticks it at all. Slips from the 1:10 pm rush are missing because nobody had hands free, so the busiest, worst-tasting half-hour is simply absent. The same regular customer filled in forty slips, all for one dish. And the kitchen started printing a new slip design in week four, so the first three weeks and the last three are not comparable.

Each of those has a name in this chapter: **label ambiguity**, **selection bias**, **duplicate concentration**, and a **distribution shift** inside your own file.

**Where the analogy breaks.** A manager reading slips notices the handwriting is all the same and gets suspicious. Training notices nothing. It will fit the forty slips from one customer as confidently as forty from forty people, and it will learn that the 1:10 pm rush does not exist. There is no mechanism inside the loop that can object to its own input.

### Picture it

```
  RAW ROWS from your product
    │
    ├─▶ DEDUPE        exact + near-duplicate ──▶ 140 removed
    ├─▶ LABEL GUIDE   one paragraph per label, with 2 edge cases
    ├─▶ DOUBLE-LABEL  100 rows by two people ──▶ agreement 76%
    │                                            (fix guide, redo)
    ├─▶ PROVENANCE    source, date, consent, licence per row
    └─▶ SPLIT RULE    by time? by user? random? ──▶ decided ONCE
                                   │
                                   ▼
                   dataset-v3.csv  +  a dataset card
                   (rows, label counts, agreement,
                    duplicate rate, split rule, licence)
```

Raw rows become a dataset only after deduplication, a written label guide checked by double-labelling, recorded provenance and one decided split rule; the output is a versioned file plus a card stating all of it.

### The four defects that cap your score

**Label noise.** Two people applying your guide to the same row and disagreeing. Measure it: have two people label the same 100 rows and compute the percentage they agree on. Below roughly 85% agreement, no model can do better than your guide, and the fix is rewriting the guide with worked edge cases — not more data. Also beware a single labeller being internally inconsistent: slip 50 rows they have already done back into their queue and see if they repeat themselves.

**Duplicates and near-duplicates.** Exact copies inflate whatever they copy. Near-duplicates — the same message with a different name, the same template with one word changed — are worse, because they cross splits and turn memorisation into an apparently high test score. Deduplicate before splitting, always, and check near-duplicates by text similarity rather than only exact matching.

**Imbalance.** If 71% of rows are one label, any score must be read against that 71% baseline, and the rare labels may have twenty examples and no hope. Three honest responses: collect more of the rare class, merge classes that you cannot separate anyway, or accept a model that only decides the common case and escalates the rest to a human. Weighting the loss helps a little and is not a substitute for examples.

**Contamination.** Train and test sharing information. Duplicates are one route; the other two are time and identity. If your rows have dates and the thing you predict evolves, a random split lets the model see next month while being tested on last month. If one user appears 40 times, a random split puts some of their rows on both sides and you measure memory of that user rather than generalisation.

### Splitting by time and by user

The default random split is wrong more often than students expect, and the rule is simple. **Split by time** when you will run the model on data from after training — order volumes, prices, trends, anything seasonal. Train on January to August, validate on September, test on October, and the score you get is the one you will actually see. **Split by user** when rows are grouped: all of one user's rows go to exactly one split. **Split randomly** only when rows are independent and time does not matter.

A random split on time-series data is the most common way to produce a score that is excellent in the notebook and useless in production, and it is the default in nearly every script an AI will write you.

### Where it lives

This chapter sits under chapters 1, 4, 5 and 10 of this module. The file you build here is what chapter 1's loop reads and what chapter 4's fine-tune consumes in a different shape — instruction and response pairs rather than rows and labels, same quality rules. Chapter 5's preference pairs have an extra version of the agreement problem, since two people often disagree about which of two answers is better. And chapter 10's eval set is a dataset too: real cases, written before you see output, with provenance.

It also touches law. M07's data-protection material and India's DPDP Act apply to a training file exactly as they apply to a database. A student's phone number in `dataset-v3.csv` on a shared drive is personal data, and a model trained on it can repeat it.

### Words to own

| Term | Plain meaning | Example |
|---|---|---|
| Label guide | Written definition of each label, with edge cases | One paragraph each |
| Inter-annotator agreement | How often two labellers match | 76 of 100 rows |
| Label noise | Wrong or inconsistent labels | "Complaint" vs "refund" |
| Near-duplicate | Almost the same row, not byte-identical | Template with a new name |
| Class imbalance | One label dominates the file | 71% "general" |
| Contamination | Test data the model effectively saw | A duplicate in both splits |
| Temporal split | Train on the past, test on the later period | Jan–Aug / Sep / Oct |
| Grouped split | All rows of one user stay in one split | Prevents identity leakage |
| Provenance | Where a row came from and when | Exported 2026-10-02 |
| Dataset card | A one-page description of the file | Counts, rules, licence |
| Synthetic data | Rows generated by a model, not collected | Useful for rare classes |
| Golden rows | A small set you labelled yourself, carefully | 200 rows, never changed |

### Tradeoffs and gotchas

**More data is the wrong first instinct.** Doubling a noisy file doubles the noise. Two hundred rows you labelled carefully against a written guide beat two thousand labelled quickly by three people who never agreed on the definitions. Fix the guide, then scale.

**Derive labels from behaviour where you can.** M09 chapter 2 made this point and it is the highest-return habit in this chapter. Did the ticket get reopened within a week? Did the order get cancelled? Did the user click? Your product records behaviour already, nobody has to be paid to label it, and it cannot drift because somebody was tired on Sunday.

**Synthetic data helps with rare classes and lies about your real distribution.** Generating 200 examples of a rare label so the model has something to learn from is legitimate. Generating your test set is not, ever: you would be measuring whether the model agrees with another model. Keep synthetic rows flagged in a column so you can always train with them and evaluate without them.

**A dataset is a versioned artefact, not a living spreadsheet.** The moment two people edit the same file, no run is reproducible and no comparison between runs means anything. Freeze, name with a version, and make the next change a new file with a one-line changelog.

**Consent and licence are not formalities.** Rows scraped from a site whose terms forbid it, or user messages collected without a privacy notice that covers model training, are a problem that surfaces at the worst possible time — when the product works and someone asks where the data came from. Record source, date and basis for every batch at the moment you collect it, because reconstructing it later is impossible.

**Your label distribution will drift and your dataset will not.** The product changes, users change, and a file frozen in October describes October. Diarise a re-check: sample 100 fresh rows a month later, label them, and see whether the mix has moved. This is chapter 13's drift monitoring in its cheapest manual form.

### Verify it

Ask an AI to "prepare the dataset" and you will get a script that loads, shuffles, splits and reports. None of those steps measures anything. Make it measure.

1. **Demand the label guide before any labelling.** One paragraph per label plus two deliberately hard edge cases per label. If the AI writes labels without definitions, you have bought the 0.68 ceiling.
2. **Demand the agreement number.** Two people, same 100 rows, percentage matched, written down. No agreement figure means the dataset's ceiling is unknown.
3. **Check that deduplication happened before splitting, not after.** Ask for the near-duplicate count, not only the exact-duplicate count, and ask which similarity threshold was used.
4. **Ask which split rule was chosen and why.** The answer must name random, temporal or grouped and justify it from your data. "Random, for simplicity" on dated rows is a defect.
5. **Ask for the per-label counts in every split.** A rare label with four rows in the test split cannot be measured, and an aggregate score will hide that completely.
6. **Ask where every row came from.** If no column records the source, provenance is already lost.

Red flags: a dataset with no version in its filename; no written label definitions; no agreement measurement; deduplication after the split or not at all; a random split on dated data; an aggregate score with no per-label breakdown; synthetic rows mixed in with no flag column; and the diagnosis "we need a bigger model" offered before anyone has measured label noise.

## Walkthrough
1. Export a few thousand real rows from your own product into one file and immediately add a version to the filename. Add columns for source and export date. Never edit this file again.
2. Write a label guide: one paragraph per label, plus two genuinely hard edge cases per label with the correct answer stated. Keep it to one page.
3. Label 100 rows yourself against that guide. Then have a teammate label the same 100 rows independently, without seeing yours.
4. Compute the agreement percentage and list every row you disagreed on. For each, decide which of you was right and whether the guide was ambiguous.
5. Rewrite the guide to settle every disagreement, then repeat steps 3 and 4 on 100 fresh rows. Record both agreement figures. If the second is not meaningfully higher, your labels are the problem, not your model.
6. Deduplicate: find exact duplicates, then near-duplicates by text similarity. Record how many of each you removed and keep three examples of the near-duplicates you found.
7. Count rows per label. Write down the majority-class baseline. Decide what you will do about your rarest label: collect more, merge it, or escalate it to a human.
8. Choose your split rule deliberately. Check whether your rows have dates and whether any user or source appears many times. Write one sentence justifying random, temporal or grouped, then implement it and print per-label counts for each split.
9. Verify there is no contamination: confirm no near-duplicate pair spans two splits, and no user appears in two splits.
10. Write a dataset card — one page: row count, label counts, agreement rate, duplicate rate, split rule and reason, source, export date, licence or consent basis, and anything you know is wrong with it.
11. Retrain chapter 1's run on this cleaned dataset and put the new test score beside the old one. Record which of your cleaning steps you believe moved it, and why.

## Talk to AI
```prompt
CONTEXT:      I am a student on an AI-native engineering program. I direct AI
              and verify its output; I do not write code by hand. I already
              know about train/validation/test splits, overfitting, leakage
              and precision versus recall, so do not re-teach those. My team
              of three runs [product in two sentences] with real users in
              India. I am building a labelled dataset for [prediction task in
              one sentence]. I have roughly [number] rows exported from my own
              product. Each row contains [list the columns, honestly,
              including any dates, user ids, names or phone numbers]. I intend
              to label them into [list the labels]. We will be [number] people
              labelling.

GOAL:         A versioned dataset file plus a one-page dataset card, with the
              label noise, duplicate rate and contamination risk measured
              rather than assumed.

REQUIREMENTS: 1. Write me a label guide: one paragraph defining each label,
                 plus two genuinely hard edge cases per label with the right
                 answer stated and the reasoning
              2. Give me a procedure to measure inter-annotator agreement on
                 100 rows, tell me what percentage is good enough to proceed,
                 and tell me what to do if we fall short
              3. Give me a deduplication step that finds near-duplicates as
                 well as exact ones, state the similarity threshold and tell
                 me how to check it is not removing legitimate rows
              4. Tell me, from my columns above, whether I should split
                 randomly, by time or by group, and why. If any column makes a
                 random split wrong, name it
              5. Check my columns for contamination risks and for anything
                 that is personal data under India's DPDP Act, and tell me
                 what to remove, hash or keep
              6. Tell me my majority-class baseline once I give you label
                 counts, and what score would make this model worth shipping
              7. For my rarest label, give me three honest options and the
                 cost of each, including whether synthetic examples are
                 acceptable and how I must flag them
              8. Give me the dataset card template to fill in, including a
                 "known problems" section

CONSTRAINTS:  Free tier only. Plain English for a second-language reader.
              Label every step READ-ONLY or CHANGES-THINGS. We have about one
              working day of human labelling time in total, so tell me where
              to spend it. Do not propose a paid labelling service or an
              annotation platform subscription.

NON-GOALS:    Do not write the training script - that is already done. Do not
              re-explain overfitting or the three splits. Do not generate my
              test set. Do not propose collecting more data before we have
              measured what is wrong with what we have.

ACCEPTANCE:   - A one-page label guide exists with two edge cases per label
              - Two people labelled the same 100 rows and I have the
                agreement percentage written down
              - Near-duplicate and exact-duplicate counts are recorded, and
                deduplication happened before the split
              - My split rule is named and justified in one sentence
              - No near-duplicate pair and no single user spans two splits
              - Per-label counts are printed for all three splits
              - A dataset card exists, including a known-problems section
              - Every row has a source and an export date

ASK:          First, look at my column list and tell me which single defect is
              most likely to be capping my score, and what one measurement
              would confirm it. Do not write the label guide until you have
              answered that.
```

## Stuck? Ask AI
```prompt
Here are 20 rows from my dataset with the labels my team assigned: [paste them]. Do not fix them. Instead, for each row tell me whether the label is defensible, ambiguous, or wrong under a reasonable reading, and say exactly which word in my label definitions made it ambiguous. Then tell me the two label definitions most likely being confused with each other, and write the one sentence I should add to my guide to separate them.
```

```prompt
My rows each have a user id and a created-at date, and I predict whether a support ticket will be reopened within a week. Explain, with a concrete example from my setup, what goes wrong if I split these rows randomly. Then tell me what a temporal split and a grouped split would each look like here, which one I need, and whether I need both at once. Finally tell me what my test score would look like in each case and which one matches what I will see in production.
```

```prompt
My four labels have counts 1,280 / 310 / 150 / 22. Tell me what my majority-class baseline is, what accuracy would be meaningless here, and which metric I should report instead. Then give me three honest options for the 22-row label, with the cost and the risk of each. Ask me which one I pick before telling me how to implement it.
```

## Quiz

### Q1 · L3
Two teammates independently label the same 100 rows and agree on 74 of them. What does this tell you?
- (a) One of them is careless and should relabel everything
- (b) The label definitions are ambiguous, and no model can beat that ceiling ✅
- (c) The dataset is too small
- (d) The model needs more epochs
> Agreement measures the instrument, not the people. Around 74% means your guide does not separate the labels, so fix the definitions with worked edge cases before labelling another row.

### Q2 · L4
Your rows have a `created_at` column and you predict next month's cancellations. Your script splits randomly. What happens?
- (a) Nothing, as long as the split is seeded
- (b) The model is tested on earlier data than it trained on, so the score is optimistic ✅
- (c) The validation split becomes too small
- (d) Class imbalance gets worse
> A random split lets the model learn from the future and be graded on the past. Split by time — train on the earlier months, test on the latest — so the score matches what production will give you.

### Q3 · L4
You find 140 near-duplicate rows, 61 of which have one copy in train and one in test. What was wrong with your reported score?
- (a) Nothing, duplicates are harmless
- (b) It partly measured memorisation of rows the model had already seen ✅
- (c) It was too pessimistic
- (d) It only affects the validation split
> Those 61 test rows were effectively training rows, so part of the score measured recall of specific examples. Always deduplicate — including near-duplicates — before splitting.

### Q4 · L4
Your four labels have 1,280 / 310 / 150 / 22 rows and your model scores 0.81 accuracy. What is the problem with quoting that number?
- (a) Accuracy cannot be used on four classes
- (b) It hides that the 22-row label may never be predicted at all ✅
- (c) 0.81 is too low to report
- (d) Accuracy requires a balanced test split to compute
> An aggregate figure dominated by the largest class can be high while a rare and possibly important class is at zero. Report per-label counts and per-label scores, against the majority-class baseline.

### Q5 · L5
A team's score has been stuck at 0.68 for four weeks across many model changes. Which action is most likely to move it?
- (a) Train a larger model for more epochs
- (b) Double the dataset using the same labelling process
- (c) Measure inter-annotator agreement and rewrite the label guide ✅
- (d) Lower the learning rate and add warmup
> A hard plateau that ignores model changes usually means the ceiling is in the labels. Doubling a noisy file doubles the noise; measuring agreement tells you whether the ceiling is the guide.

## Go deeper
- [Datasheets for Datasets: the dataset-card paper](https://arxiv.org/abs/1803.09010)
- [Ministry of Electronics and IT: the Digital Personal Data Protection Act](https://www.meity.gov.in/data-protection-framework)
