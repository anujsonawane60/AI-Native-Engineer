---
id: m09-07-gpus-and-scaling
module: m09-ai-foundations
slug: gpus-cnns-scaling-laws
title: "Why GPUs Matter; CNNs, RNNs, Parameters and Scaling Laws"
order: 7
duration_min: 15
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

Two students run the same image classifier on the same photos. One leaves the Colab runtime on CPU, waits forty minutes for an epoch, and drops the feature as impractical without money. The other switches one dropdown to a free GPU, finishes in fifty seconds, and runs twelve experiments that afternoon. A third picks the largest model and crashes out of memory; a fourth gets a 7B model running at three words per second, which no user tolerates.

### The idea

**Almost all the work in a neural network is the same tiny multiply-and-add, repeated billions of times, and every one of those is independent of the others. A CPU does a handful of things very fast; a GPU does thousands of things at once. That is the entire reason deep learning became possible.**

Then there are the **scaling laws**. Increase data, parameters and compute together and the loss falls along a smooth, predictable path. Two consequences matter: it justified enormous spending, because a hundred times the compute reliably buys a better model, which is why frontier models exist and why you rent them rather than build them; and the returns diminish steeply, so a cheap small model is often most of the way there on your own task.

### The analogy

Five hundred dosas are needed for a function. One option is the head cook: twenty years of skill, one dosa every twelve seconds. That is a **CPU** — a few capable cores handling any instruction in sequence. The other is two hundred first-years at two hundred tawas, each slow and able to do one thing, finishing in a fraction of the time. That is a **GPU**, and neural network arithmetic has the same shape, which is why the industry runs on hardware built for drawing triangles in games.

**Where the analogy breaks:** the volunteers could each do their own thing and wait for one another. GPU cores are rigid, executing the same instruction in lockstep, so strictly ordered work gets almost no benefit — which is exactly why the RNN was a problem.

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

Two architectures mattered before transformers. A **CNN** slides a small window across an image looking for one pattern and stacks the findings: edges become textures, textures become wheels — still a cheap, excellent answer for dish photos. An **RNN** reads a sequence one item at a time carrying a running summary, and had two crippling problems: the summary forgets, and it is strictly sequential, the one shape of work two hundred tawas cannot speed up. The transformer removed both at once — less a story about cleverness than about hardware, because the architecture that won was the one the GPU could run.

### Where it lives

Three questions decide any model before you commit.

- **Does it fit?** Chapter 5's arithmetic — parameters, bytes per number, working space — against the GPU's memory, checked in the notebook rather than quoted.
- **Is it fast enough?** Tokens per second for text, images per second for vision. Fitting and being usable are separate gates.
- **Can you avoid owning hardware at all?** Usually yes: a hosted model trades memory arithmetic for a price per request. A rented GPU costs money while idle; an API costs only when used.
- **Quantization, serving, batching and real throughput are M14.**

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

- **Switch the runtime to a GPU before concluding anything is too slow.** The commonest wasted afternoon here is training on a CPU by accident; print the device as the first line of output.
- **A GPU is useless for most of what your app does.** Serving pages and reading databases is sequential work — the head cook's job.
- **Parameter count is not a quality ranking.** A well-trained small model from this year beats a larger one from two years ago, so rank by measured performance on your task.
- **Fitting and running are different walls.** "It loaded" is not "it works": measure tokens per second first.
- **Every number here is perishable except the ideas.** Allowances, session limits and memory requirements change without notice, so verify in the notebook, on the day.

### Verify it

- **A hardware claim with no measurement behind it.** "This runs on a free Colab GPU" must be backed by memory you printed yourself.
- **An invented model name or size.** Families are real; size-and-version combinations are often fabricated. Find the exact string on the provider's page.
- **Free-tier limits quoted as current.** Anything about GPU hours, session length or memory predates the model's cutoff; the platform's own docs are the only authority.
- **"Bigger will be better" with no task-level evidence.** Scaling laws describe averages, not your twenty canteen tickets. Ask for the measured gain and the cost per request.
- **A GPU proposed for your web server.** Ask which operation needs parallel arithmetic; a vague answer means cost with no benefit.
- **Check yourself:** print the device, GPU name and memory, then time one batch on CPU and the same on GPU.

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
