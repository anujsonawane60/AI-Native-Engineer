---
id: m09-05-neurons-layers
module: m09-ai-foundations
slug: neurons-layers-forward-pass
title: "Neurons, Layers, Weights and the Forward Pass"
order: 5
duration_min: 25
is_free: false
mission: false
volatility: low
last_reviewed: 2026-11-16
---

## Key learnings
- Describe a neuron as a weighted vote and a layer as many votes at once
- Explain what "the weights" are and why a model is just a large file of numbers
- Turn a parameter count into a rough memory requirement, so you know what fits on a free GPU
- Say what the forward pass is and why it is the thing you pay for on every request
- Catch an AI inventing parameter counts or claiming a huge model runs on a small device

## Lesson

### The pain

A group reads that a model has "7B parameters" and decides to run it on a free Colab GPU. The notebook dies partway through loading with an out-of-memory error. They try the 70B version, reasoning that bigger must be better; it dies faster. Two days go into forum posts and random settings, and into an AI that confidently suggests fixes which do not help, because none of them address the actual problem.

The actual problem is arithmetic they could have done in ten seconds, if anybody had told them that a parameter is a number, that numbers take up space, and that a free Colab GPU has about 15 to 16 gigabytes of its own memory.

That single piece of intuition — parameters are numbers, numbers take space, space is what you have — decides which models you can touch for free, what your inference costs, and why a model that runs beautifully on a server will not run on a budget Android phone.

### The idea

**A neuron is a weighted vote. A layer is thousands of those votes computed at once. A model is the entire collection of weights, stored in a file. Running input through all the layers to get an output is the forward pass.**

One neuron takes several numbers in, multiplies each by its own importance — its **weight** — adds them up with a small offset called a **bias**, and passes the total through a simple gate that decides how much of it continues. That is all. Nothing in a neuron is clever. What is extraordinary is that stacking millions of them and tuning all the weights produces something that writes Marathi poetry.

### The analogy

The canteen is deciding whether to cook extra biryani tomorrow, and the head cook polls his team.

Each person has an opinion based on what they watch. The storekeeper weighs how much rice is left. The cashier weighs yesterday's till. The server weighs how long the queue was. The cook does not treat them equally: he has learned over years that the cashier's view counts double and the new trainee's counts for very little. He adds up the weighted opinions, applies his own standing mood — "I lean towards cooking extra on Fridays regardless" — and if the total crosses a threshold, extra biryani happens.

That is one neuron. The opinions are inputs, the importance he assigns each person is the weights, his standing lean is the bias, the threshold is the activation gate.

Now imagine the kitchen has a thousand such polls running in parallel, each asking a different sub-question, and the answers of the first thousand become the inputs to a second thousand, and so on for ninety rounds. Nobody in round forty could tell you what their question means. That is a deep network.

**Where the analogy breaks:** every person in the canteen poll is answering a question a human named. In a real network, only the first layer's inputs and the last layer's output mean anything to us. The middle layers invent their own intermediate questions during training, and those questions have no names and often no human-readable meaning at all. This is why you cannot open a large model and read its reasoning — there is no sentence in there to read.

### Picture it

```
  ONE NEURON
    rice left ──(weight 0.2)──┐
    till total ──(weight 0.9)─┤
    queue size ──(weight 0.5)─┼──▶ add up ──▶ gate ──▶ output
    bias: +0.1 ───────────────┘               (keeps
                                            only what
                                            is strong)

  LAYERS STACKED (the forward pass)
   input          hidden 1        hidden 2        output
  ┌──────┐       ┌────────┐      ┌────────┐      ┌───────┐
  │ 5 raw│──────▶│ 1,000  │─────▶│ 1,000  │─────▶│  1    │
  │ facts│       │ votes  │      │ votes  │      │ answer│
  └──────┘       └────────┘      └────────┘      └───────┘
                  no human         no human
                  meaning          meaning
```

A neuron multiplies each input by a learned weight, sums them with a bias and passes the total through a gate; a forward pass is doing that layer after layer from input to answer.

### Why the gate matters

The gate — properly, the **activation function** — looks like a detail and is the reason deep networks work at all.

Without it, stacking layers achieves nothing. A hundred layers of pure weighted sums collapses mathematically into one weighted sum, so a hundred-layer network would be exactly as capable as a single layer: able to represent only straight-line relationships, like chapter 4's linear regression. The gate bends things. The commonest one, called ReLU, simply passes positive totals through unchanged and turns negative totals into zero — "ignore weak or contrary signals entirely". That one bend, repeated across layers, is what lets a network represent curved, conditional, interacting relationships.

You never choose this yourself. You need it only to answer the question "why are many layers better than one", whose answer is: because of the gate between them.

### Parameters, and the arithmetic that decides what you can run

Every weight and every bias is one **parameter**. A 7-billion-parameter model has seven billion numbers in it, and those numbers have to be somewhere for the model to run.

Each number is stored in a few bytes. Two bytes each is the common choice for running models today. So the rough memory requirement in gigabytes is a little more than twice the parameter count in billions:

```
  model size        numbers    at 2 bytes each   fits on a free
                                                 Colab GPU (~15 GB)?
  ──────────────────────────────────────────────────────────────────
  0.5 billion       0.5 B          ~1 GB          yes, easily
  3 billion           3 B          ~6 GB          yes
  7 billion           7 B         ~14 GB          barely, often not
  13 billion         13 B         ~26 GB          no
  70 billion         70 B        ~140 GB          no, not close
```

Parameter count translated into rough memory, against what a free Colab GPU can hold.

Two corrections to keep this honest. **Running a model needs more than the weights** — there is working space for the input being processed, which grows with how long your prompt is, so a model that theoretically fits at 14 GB often does not fit in practice. And **the bytes per number can be reduced**, which is called quantization: squeeze each number into one byte or less and a 7-billion model drops to around 7 GB or lower, at some cost in quality. Quantization in practice — actually doing it, measuring what it cost you — is M14. Here you need only the shape of the trade: fewer bytes per number means smaller and faster, and eventually worse.

**Training needs several times more memory than running**, because it also has to hold the blame signals and the optimiser's own bookkeeping for every parameter. The rule of thumb is that a model you can just barely run cannot be trained on the same hardware.

### Where it lives

A trained model is a file. You can see it on disk and its size is roughly the arithmetic above. For your own small models that file lives in object storage (M04) and gets loaded by your server at startup.

The **forward pass is the thing you pay for.** Every request to a model — yours or a provider's — is one forward pass per token generated, and the price on a provider's bill is a proxy for exactly this work. That is why a bigger model costs more per request, and why chapter 14's cost arithmetic is ultimately arithmetic about forward passes.

The backward pass, which is how the weights got their values, is chapter 6. Running real training loops is M14.

### Words to own

| Term | Plain meaning | Example |
|---|---|---|
| Neuron | One weighted vote plus a gate | The cook's weighted poll |
| Weight | How much one input counts | "The cashier counts double" |
| Bias | A standing lean, independent of input | "I lean towards extra on Fridays" |
| Activation function | The gate that bends the output | ReLU: negatives become zero |
| Layer | Many neurons computed together | 1,000 votes at once |
| Hidden layer | A layer between input and output | Has no human meaning |
| Deep | Having many layers | Ninety layers |
| Parameter | One weight or one bias | 7 billion of them |
| Forward pass | Input travelling through to the answer | What you pay for per request |
| VRAM | The GPU's own memory | ~15 GB on free Colab |
| Quantization | Fewer bytes per number, so smaller | 2 bytes down to 1 |
| Architecture | The arrangement of layers | How it is wired |

### Tradeoffs and gotchas

**Bigger is not automatically better, and it is always more expensive.** More parameters means more memory, more latency per token, and a higher price per request. For many narrow jobs — classifying a support ticket, extracting a price from a line of text — a small model is as good and costs a fraction. Chapter 13 makes you prove this on your own task rather than assume it.

**Parameter count tells you about size, not about quality.** Two models with the same count can differ enormously depending on what they were trained on and for how long. Treat the number as a cost and memory signal, never as a quality ranking.

**A model that fits in memory may still be unusably slow.** Memory is the first gate; throughput is the second. A 7-billion model squeezed onto a free GPU may generate a few tokens per second — fine for a batch job, intolerable in a chat interface.

**Your users' phones are not your laptop.** M08 made this point about apps and it applies doubly here: budget Android devices in India have a few gigabytes of total RAM shared with everything else. On-device models for those phones are measured in hundreds of millions of parameters, not billions. "Just run it locally on the phone" is a sentence to check with arithmetic.

**Nothing in the middle layers is inspectable.** When someone promises you an explanation of why a large model produced a specific answer, be sceptical. A model can produce a plausible-sounding explanation of its own output, and that explanation is also generated text, not a recording of what happened inside. This matters for chapter 16 and for M07.

### Verify it

**Red flag: an invented parameter count.** Numbers about specific named models are exactly the kind of fact an AI produces confidently and wrongly, because model line-ups change monthly. Any parameter count, context size or hardware requirement quoted in a chat window must be checked against the model's own page before you plan around it.

**Red flag: a model name you cannot find.** Plausible model names get invented constantly, usually by mixing a real family with a size that never shipped. If you cannot find the exact string on the provider's or Hugging Face's own page, assume it does not exist.

**Red flag: "this will run fine on free Colab" with no arithmetic shown.** Make it state the parameter count, the bytes per number, the resulting gigabytes, and the GPU's actual memory, as four separate numbers. Then check the fourth yourself, in the notebook, rather than trusting it.

**Red flag: "just run it on the user's phone."** Ask for the device's RAM, the model's size after quantization, and how many tokens per second it would generate on a four-year-old mid-range Android. Two of those three will be guesses.

**Red flag: a confident explanation of a large model's internal reasoning.** The honest version is "here is what the output suggests", not "here is what the network computed".

The check you can do yourself takes two minutes: in a free notebook, print the GPU's total and free memory before loading anything, then load the model and print it again. The difference is the real answer, and it almost always differs from the estimate. That habit — measure the thing rather than accept the summary — is the whole method of this course applied to hardware.

## Walkthrough
1. Draw one neuron on paper with three inputs. Label each arrow with a weight you invent, add a bias, and write the gate at the end.
2. Pick three input numbers and compute the output by hand, including the gate. Do it a second time with one weight set to zero and note what changed. This is the only arithmetic in the chapter and it takes four minutes.
3. Write down, in one sentence, what the gate did in your calculation and what would happen if it were not there.
4. Now take three model sizes you have heard of and convert each to rough memory at two bytes per parameter. Write the three numbers in gigabytes.
5. Open a free Colab or Kaggle notebook, select a GPU runtime, and print the GPU's total memory. Write down the real number rather than the one you assumed.
6. Compare your three estimates against that real number and mark each one yes, borderline or no. You now know which models you can touch for free.
7. Load the smallest of them, print the GPU memory again, and compare the actual consumption against your estimate. Note the gap and write one sentence on where the extra went.
8. Ask an AI for the parameter count and memory requirement of a specific named model. Then find the same two facts on the model's own official page. Record whether they match.
9. Finally, write one sentence for your decision log: the largest model your team can run on free hardware, the number that proves it, and the date you checked.

## Talk to AI
```prompt
I want to understand what a neural network physically is, and what size of model I can actually run on free hardware. I am not going to write code and I do not want mathematical notation.

Please do all of this in plain language:

1. Describe one neuron as a decision made by weighted opinions. Use a shop or a kitchen, not a brain. Name the weights, the bias and the gate, and tell me what each one does.

2. Explain why stacking a hundred layers of pure weighted sums with NO gate between them would be no better than one layer. Then explain what the gate changes, in words, with no formulas.

3. Teach me the memory arithmetic as a repeatable habit: parameters, bytes per parameter, resulting gigabytes. Work it for 0.5 billion, 3 billion, 7 billion and 70 billion parameters at two bytes each, and show each step.

4. Tell me what ELSE consumes GPU memory when running a model beyond the weights themselves, and roughly how much, so my estimate is not optimistic.

5. Explain why training the same model needs several times more memory than running it.

6. Then, and only then, tell me what I should expect a free Colab or Kaggle GPU to have. Flag this number explicitly as something that changes and that I must verify in the notebook myself rather than trust from you.

7. Finally, give me three red flags that would tell me a claim about model size or hardware requirements is invented rather than checked.
```

## Stuck? Ask AI
```prompt
Explain what a weight and a bias are using an analogy about a panel of judges scoring a dance competition. Then give me four small changes to the weights and ask me to predict what happens to the final score before you tell me.
```

```prompt
I do not understand why "7B parameters" means anything practical. Walk me from the number 7,000,000,000 to a number of gigabytes, one step at a time, telling me what each step assumes. Then ask me to do the same for a different model size without help.
```

```prompt
Explain the difference between a model fitting in memory and a model being fast enough to use. Give me a concrete example of a model that fits but is too slow for a chat interface, and tell me what measurement I should take to find out which problem I have.
```

## Quiz

### Q1 · L2
What is a weight in a neural network?
- (a) The size of the model file in megabytes
- (b) A learned number saying how much one input counts ✅
- (c) The speed at which the model generates tokens
- (d) The threshold an output must cross to be reported
> Weights are the learned numbers that scale each input before they are summed. Collect every weight and bias and you have the model.

### Q2 · L3
Why does a deep network need an activation gate between layers?
- (a) To prevent the model from overfitting
- (b) Without it, stacked weighted sums collapse into a single weighted sum ✅
- (c) To reduce the number of parameters needed
- (d) To make the output a probability
> The gate is what introduces the bend. Without it, a hundred layers would represent nothing more than one straight-line relationship.

### Q3 · L4
A 13-billion-parameter model is stored at two bytes per number. Roughly how much GPU memory do the weights alone need?
- (a) About 6 GB
- (b) About 13 GB
- (c) About 26 GB ✅
- (d) About 130 GB
> Two bytes for each of thirteen billion numbers is around twenty-six gigabytes, before any working space for the input. That is well beyond a free Colab GPU.

### Q4 · L4
An AI tells you a specific named model has 8 billion parameters and runs comfortably on free Colab. What should you do?
- (a) Proceed; parameter counts are stable facts
- (b) Check the model's own official page, then measure GPU memory in the notebook ✅
- (c) Choose the 70-billion version instead for better quality
- (d) Assume it is wrong and pick a different family
> Model specifications change faster than any training cutoff, and "fits comfortably" hides the working space the input needs. Verify on the provider's page and in the notebook.

### Q5 · L5
Your team wants to run a language model directly on users' budget Android phones. What is the first thing to establish?
- (a) Which app store policies apply
- (b) The phone's available RAM against the quantized model size and the tokens per second ✅
- (c) Whether the model was trained with an activation gate
- (d) The model's score on a public benchmark
> On-device limits are a memory-and-throughput question first. Budget phones share a few gigabytes with everything else, which puts practical on-device models in the hundreds of millions of parameters.

## Go deeper
- [3Blue1Brown: But what is a neural network?](https://www.3blue1brown.com/lessons/neural-networks)
- [Google: Neural networks in the ML crash course](https://developers.google.com/machine-learning/crash-course/neural-networks)
