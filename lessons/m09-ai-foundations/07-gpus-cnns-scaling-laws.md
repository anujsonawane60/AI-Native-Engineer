---
id: m09-07-gpus-and-scaling
module: m09-ai-foundations
slug: gpus-cnns-scaling-laws
title: "Why GPUs Matter; CNNs, RNNs, Parameters and Scaling Laws"
order: 7
duration_min: 25
is_free: false
mission: false
volatility: low
last_reviewed: 2026-11-16
---

## Key learnings
- Explain why a GPU is dramatically faster than a CPU for this one kind of work
- Say what a CNN and an RNN each were good at, and why transformers replaced RNNs
- Read a scaling law as "predictably better, with diminishing returns per rupee"
- Tell the difference between a model being too big to fit and too slow to use
- Catch an AI quoting hardware requirements or free-tier GPU limits as settled fact

## Lesson

### The pain

Two students run the same small image classifier on the same canteen photos. One leaves the Colab runtime on its default CPU setting and waits forty minutes for an epoch. The other changes one dropdown to a free GPU and finishes the same epoch in fifty seconds. Same code, same data, same notebook, same free account.

The first student concludes that machine learning is impractical for a team with no money and drops the feature. The second runs twelve experiments that afternoon.

A third student picks the largest model on the list and gets an out-of-memory crash. A fourth gets a 7-billion model running on the free GPU, is delighted, and then finds it emits about three words per second in their chat interface, which no user will tolerate.

All four facts — the CPU-versus-GPU gap, the memory wall, the throughput wall, and the fact that none of these numbers stay true for long — decide what your team can actually build this semester.

### The idea

**Almost all the work in a neural network is the same tiny multiply-and-add, repeated billions of times, and every one of those is independent of the others. A CPU does a handful of things very fast; a GPU does thousands of things at once. That is the entire reason deep learning became possible.**

The second half of the chapter is what happens as you make these models bigger: performance improves in a predictable, boring, measurable way as you add data, parameters and compute — and the improvement per rupee shrinks the whole time. Those two facts together explain both why the frontier models are enormous and why the right model for your canteen app is small.

### The analogy

The canteen has to produce five hundred dosas for a function.

One option is the head cook: twenty years of skill, works alone, superb at anything you hand him including the complicated orders. He makes a dosa every twelve seconds and never stops. That is a **CPU** — a few extremely capable cores that handle any kind of instruction, one sequence at a time, very fast.

The other option is two hundred first-year volunteers at two hundred tawas. Each is slow, clumsy, and can do exactly one thing: pour batter, wait, flip, remove. If you need anything unusual they are useless. But for five hundred identical dosas they finish in a fraction of the head cook's time. That is a **GPU** — thousands of simple cores doing the identical operation on different data simultaneously.

Dosas are the right job for the volunteers because every dosa is independent. Neural network arithmetic is the same shape, which is why the industry runs on graphics hardware that was originally designed for drawing triangles in games.

**Where the analogy breaks:** the volunteers can each do their own thing if you let them, and they can wait for each other. GPU cores are far more rigid — they want to execute the same instruction in lockstep, and work that must happen in strict order gets almost no benefit. This is exactly why the RNN, coming up next, was such a problem: it insisted on going one word at a time.

### Picture it

```
  CPU                              GPU
  ┌─────────────────┐              ┌────────────────────────┐
  │  ┌───┐ ┌───┐    │              │ ░░░░░░░░░░░░░░░░░░░░░░ │
  │  │ 1 │ │ 2 │    │              │ ░░░░░░░░░░░░░░░░░░░░░░ │
  │  └───┘ └───┘    │              │ ░░░░░░░░░░░░░░░░░░░░░░ │
  │  ┌───┐ ┌───┐    │              │ ░░░░░░░░░░░░░░░░░░░░░░ │
  │  │ 3 │ │ 4 │    │              │ ░░░░░░░░░░░░░░░░░░░░░░ │
  │  └───┘ └───┘    │              │ ░░░ thousands of  ░░░░ │
  │                 │              │ ░░░ simple cores  ░░░░ │
  │ a few fast cores│              │ ░░░░░░░░░░░░░░░░░░░░░░ │
  │ anything, in    │              │ one operation, on      │
  │ sequence        │              │ everything, at once    │
  └─────────────────┘              └────────────────────────┘
      head cook                      200 volunteers, 200 tawas
```

A CPU has a few powerful general-purpose cores working in sequence; a GPU has thousands of simple cores applying the same operation to huge amounts of data simultaneously.

### CNNs and RNNs, and why one of them is history

Before transformers, two architectures mattered, and knowing what each was for explains why today's models look the way they do.

A **CNN**, or convolutional neural network, slides a small window across an image looking for one pattern at a time — an edge here, a corner there — and stacks those findings into bigger ones: edges become textures, textures become wheels, wheels become cars. It rests on the observation that in a photograph what matters is local and repeats everywhere, so one small detector can be reused across the whole image. CNNs are not obsolete. For classifying canteen dish photos on a small dataset, a CNN is still an excellent, cheap, well-understood answer.

An **RNN**, or recurrent neural network, reads a sequence one item at a time, carrying a running summary forward. It was the standard approach to language for years and had two crippling problems. The summary **forgets**: by word two hundred, word three has faded to nothing. And because word two cannot be processed until word one is done, it is **strictly sequential** — the one shape of work that gets no benefit from two hundred volunteers at two hundred tawas.

The transformer, which is chapter 10, removed both problems at once: every token can look directly at every other token, and all of it computes in parallel. That is why RNNs were abandoned for language within a few years, and it is less a story about cleverness than about hardware. The architecture that won was the one the GPU could run.

### Scaling laws

Researchers measured something that turned out to be remarkably regular. If you increase three things together — the amount of training data, the number of parameters, and the compute spent — the model's loss falls along a smooth, predictable path. Not in jumps. Smoothly enough that you can forecast roughly how good a model will be before you train it.

Two consequences, and both matter to you.

**It justified spending enormous money.** If a hundred times the compute reliably produces a meaningfully better model, a company with a hundred times the budget gets a better model, and the race to build large models follows directly. This is why the frontier models exist and why you rent them rather than build them.

**The returns diminish, steeply.** Each doubling of quality costs far more than the previous one. Going from weak to decent is cheap; going from very good to slightly better costs a fortune. For you this is the useful half: a cheap small model is often most of the way there on your actual task, and the gap you pay a premium to close may not be one your users can perceive. Chapter 13 makes you measure exactly that.

There is also a balance point. Early large models were trained with far too little data for their size; the correction, associated with the Chinchilla results, was that for a given compute budget you should train a somewhat smaller model on substantially more data. The echo for you is that a newer small model routinely beats an older larger one, so parameter count ranks models badly.

### Where it lives

For your team, all of this collapses into three questions you ask about any model before you commit.

**Does it fit?** Chapter 5's arithmetic: parameters, bytes per number, plus working space, against the GPU's memory. A free Colab or Kaggle GPU is in the region of fifteen gigabytes, but check it in the notebook rather than trusting any quoted figure — this is precisely the kind of free-tier detail M06 warns you goes stale.

**Is it fast enough?** Tokens per second for text, images per second for vision. Fitting and being usable are separate gates, and the second kills chat features.

**Can you avoid owning the hardware entirely?** Usually yes, and usually you should. Calling a hosted model means no GPU and no memory arithmetic, and a price per request instead — which moves the problem to chapter 14. A rented GPU costs money while idle; an API costs money only when used, which for spiky student traffic is decisive.

Quantization, serving, batching and squeezing real throughput out of one GPU are M14.

### Words to own

| Term | Plain meaning | Example |
|---|---|---|
| CPU | A few fast general-purpose cores, in sequence | The head cook |
| GPU | Thousands of simple cores, all at once | 200 volunteers |
| VRAM | The GPU's own memory | ~15 GB on free Colab |
| Parallel | Many independent operations at the same time | Network arithmetic |
| Sequential | Each step must wait for the previous one | An RNN reading words |
| CNN | Slides a small pattern detector over an image | Dish photo classifier |
| RNN | Reads a sequence one item at a time | Superseded for language |
| Throughput | How much work per second | Tokens per second |
| Scaling law | Quality improves predictably with data, size and compute | Smooth, not jumpy |
| Diminishing returns | Each doubling costs more than the last | Why small models win |
| Quantization | Fewer bytes per number, smaller and faster | Covered in M14 |

### Tradeoffs and gotchas

**Switch the runtime to a GPU before you conclude anything is too slow.** The most common wasted afternoon in this module is a student training on a CPU by accident. Print the device your code is using, every time, as the first line of output.

**A GPU is useless for most of what your app does.** Serving a page, reading a database, running your API — all sequential, branching work, which is the head cook's job. Never let anyone tell you your web app needs a GPU.

**Parameter count is not a quality ranking.** A well-trained small model from this year routinely beats a larger model from two years ago. Rank by measured performance on your own task, which is chapter 13.

**Fitting and running are different walls.** "It loaded" is not "it works". Measure tokens per second before you design an interface around it.

**Every number in this chapter is perishable except the ideas.** Free-tier GPU allowances, session limits, which GPU you are assigned, model line-ups and memory requirements all change without notice. The CPU-versus-GPU reasoning will still be true in five years; "about fifteen gigabytes" will not. Verify in the notebook, on the day.

### Verify it

**Red flag: a hardware claim with no measurement behind it.** "This will run on a free Colab GPU" must be backed by the GPU memory you printed yourself. Make the AI show its arithmetic, then check the inputs.

**Red flag: an invented model name or size.** This recurs from chapter 5 because it is the most frequent fabrication in this subject. Model families are real; specific size-and-version combinations are often hallucinated. Find the exact string on the provider's own page.

**Red flag: free-tier limits quoted as current.** Anything an AI tells you about free GPU hours, session length or memory predates its knowledge cutoff — M00's failure mode landing where it wastes your afternoon. The platform's own documentation is the only authority.

**Red flag: "bigger will be better" with no task-level evidence.** Ask what measurable improvement the larger model gives on your examples, and what it costs per request. Scaling laws speak about averages across everything; they say nothing about your twenty canteen tickets.

**Red flag: a GPU proposed for your web server.** Ask which operation needs parallel arithmetic. If the answer is vague, the GPU is a cost with no benefit.

And the check to run yourself, in four minutes: print the device, print the GPU's name and memory, run one small batch on CPU and the same batch on GPU, and write down both timings. One measured ratio teaches more than any explanation.

## Walkthrough
1. Open a free Colab or Kaggle notebook and print which device your code is running on. Write it down. Do this before anything else, always.
2. Switch the runtime to a GPU, print the device again, and print the GPU's name and total memory. Record both real numbers in your notes with today's date.
3. Run any small training example on the CPU and time one epoch. Then run the identical code on the GPU and time one epoch. Write down the ratio.
4. Write one sentence explaining, in terms of independent operations, why the ratio came out the way it did.
5. Now take the memory arithmetic from chapter 5 and list three model sizes against the GPU memory you actually measured. Mark each fits, borderline or no.
6. Load the largest model you marked "fits" and generate a short paragraph. Time it and convert to tokens per second. Decide whether a user would wait that long.
7. Ask an AI what the current free Colab GPU allowance and session limit are. Then find the same facts in the platform's own documentation. Record both answers and the gap.
8. Ask it to recommend the best model for your task and to state the parameter count and memory requirement. Verify every number on the provider's own page and mark each one confirmed or invented.
9. Finally, write one decision-log entry: your team's hardware ceiling, the measured numbers that prove it, the date you measured, and whether you will use a hosted API instead of owning a GPU.

## Talk to AI
```prompt
I am a student with no money and only free Colab or Kaggle GPUs, and I need to work out what my team can realistically run. Do not reassure me. Flag every number you are not certain about.

My task: [describe what the model needs to do, and whether a user waits for the answer]
My data: [type and rough size - images, text, rows]
My deadline: [how long I have]

Work through this in order:

1. Explain to me, in terms of independent operations, why this particular workload is or is not a good fit for a GPU. If part of my pipeline would get no benefit from a GPU, name that part.

2. Give me the memory arithmetic for three candidate model sizes: parameters, bytes per number, working space for input, total gigabytes. Show every step so I can redo it myself.

3. For each candidate, tell me the SECOND question too: roughly how many tokens or images per second I should expect, and whether that is acceptable for a user who is waiting. Be explicit that this is an estimate.

4. Tell me what I should measure MYSELF in the notebook before trusting any of your numbers, and give me the exact things to print.

5. For any claim you make about free-tier GPU memory, session limits or hours, say plainly that it may be out of date and tell me which official page to check.

6. Then argue the other side: make the case that I should not own or rent a GPU at all and should call a hosted model instead. Compare the two on cost when idle, cost per request, latency, and how much of my time each one eats.

7. Finish with the three mistakes you think a student team is most likely to make here, and how I would notice each one within an hour rather than a week.
```

## Stuck? Ask AI
```prompt
Explain the difference between a CPU and a GPU using an analogy that is not about cooks, cars or construction. Then describe five computing tasks, one at a time, and ask me which hardware suits each and why, before telling me the answer.
```

```prompt
Explain why recurrent neural networks were abandoned for language work. Cover both problems - forgetting across long sequences and being strictly sequential - and explain specifically why the second one mattered so much given how GPUs work.
```

```prompt
Explain scaling laws to me in plain words with no formulas, and then explain diminishing returns with concrete rupee amounts for a student team. Finish by telling me what scaling laws do NOT tell me about which model to pick for my own specific task.
```

## Quiz

### Q1 · L2
Why is a GPU so much faster than a CPU for neural network work?
- (a) Its individual cores are much faster than CPU cores
- (b) The arithmetic is thousands of independent operations it can do at once ✅
- (c) It has more memory than a CPU
- (d) It compresses the model before running it
> GPU cores are individually simpler and slower. The win comes from applying the same operation to enormous amounts of data simultaneously, which is exactly the shape of network arithmetic.

### Q2 · L3
Which property of RNNs made them a poor match for GPU hardware?
- (a) They used too many parameters
- (b) They must process one item at a time, so the work cannot be parallelised ✅
- (c) They could only handle images
- (d) They required labelled data
> Strictly sequential work gets almost no benefit from thousands of parallel cores. Transformers process the whole sequence at once, which is a large part of why they won.

### Q3 · L3
A model loads successfully on your free GPU and then produces about three words per second. What kind of problem do you have?
- (a) A memory problem; use a smaller model
- (b) A throughput problem; it fits but is too slow for a waiting user ✅
- (c) A training problem; run more epochs
- (d) No problem; three words per second is normal for chat
> Fitting and being fast enough are separate gates. The feature may still work as a background job, but not in an interface where somebody is waiting.

### Q4 · L4
What do scaling laws practically tell a student team choosing a model?
- (a) Always pick the largest model you can afford
- (b) Quality rises predictably with size, but returns per rupee shrink, so test whether a small model is already enough ✅
- (c) Parameter count is the best ranking of model quality
- (d) Small models cannot be useful for real products
> Scaling laws are about averages across everything. The diminishing-returns half is the part that matters to you, because the premium you pay may close a gap your users cannot perceive.

### Q5 · L5
An AI tells you the current free Colab GPU allowance and session limit, with specific numbers. How should you treat this?
- (a) As fact, since it is a technical detail
- (b) As probably stale: check the platform's own documentation and measure in the notebook ✅
- (c) As irrelevant, since limits do not affect training
- (d) As a reason to avoid Colab entirely
> Free-tier limits change without notice and sit squarely behind any model's knowledge cutoff. The provider's own page is the only authority, and the notebook gives you the real memory figure.

## Go deeper
- [Google Colab: frequently asked questions on GPUs and limits](https://research.google.com/colaboratory/faq.html)
- [Hugging Face: methods and tools for efficient training on one GPU](https://huggingface.co/docs/transformers/en/perf_train_gpu_one)
