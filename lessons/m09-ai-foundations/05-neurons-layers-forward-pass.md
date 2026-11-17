---
id: m09-05-neurons-layers
module: m09-ai-foundations
slug: neurons-layers-forward-pass
title: "Neurons, Layers, Weights and the Forward Pass"
order: 5
duration_min: 15
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

A group reads that a model has "7B parameters" and tries to run it on a free Colab GPU. The notebook dies while loading, so they try the 70B version, reasoning that bigger must be better; it dies faster. Two days disappear into forum posts and an AI suggesting fixes that cannot help, because the real problem is arithmetic: a parameter is a number, numbers take space, and a free Colab GPU holds about 15 GB.

### The idea

**A neuron is a weighted vote. A layer is thousands of those votes computed at once. A model is the whole collection of weights, stored in a file. Running input through all the layers to get an output is the forward pass.**

One neuron takes several numbers in, multiplies each by its own importance — its **weight** — adds them with a small offset called a **bias**, and passes the total through a gate. Nothing in a neuron is clever; what is extraordinary is that stacking millions of them produces something that writes Marathi poetry.

### The analogy

The head cook polls his team on extra biryani. The storekeeper reports rice left, the cashier yesterday's till, the server the queue length — and he has learned the cashier's view counts double and the trainee's barely counts. He adds up the weighted opinions, applies his standing lean ("extra on Fridays regardless"), and if the total crosses a threshold, extra biryani happens. That is one neuron: opinions are inputs, the importance he assigns is the weights, his lean is the bias, the threshold is the gate. Run a thousand such polls in parallel and feed their answers into a second thousand, ninety times over: a deep network.

**Where the analogy breaks:** every person in the poll answers a question a human named. In a real network only the first layer's inputs and the last layer's output mean anything to us — the middle layers invent their own questions during training, with no names and no readable meaning.

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

The gate — the **activation function** — is why depth works: without it a hundred layers of weighted sums collapse into one, no better than chapter 4's linear regression. ReLU turns negative totals into zero, and that one bend, repeated, represents curved, conditional relationships.

Every weight and bias is one **parameter**, and those numbers must be in memory to run. At two bytes each, the gigabytes needed are a little over twice the parameter count in billions.

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

Two corrections: working space grows with prompt length, so a model that fits on paper often does not in practice; and **quantization** at one byte per number drops a 7B model to about 7 GB, at some cost in quality.

### Where it lives

- **A trained model is a file**, its size roughly the arithmetic above, living in object storage and loading at server startup.
- **The forward pass is the thing you pay for** — one per token generated, which is why chapter 14's cost arithmetic is arithmetic about forward passes.
- **The backward pass, which gave the weights their values, is chapter 6.** Real training loops and quantization are M14.

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

- **Bigger is not automatically better, and is always more expensive** in memory, latency and price per request. For narrow jobs a small model is as good for a fraction of the cost.
- **Parameter count tells you about size, not quality.** Treat it as a cost signal, never a ranking.
- **A model that fits in memory may still be unusably slow.** Memory is the first gate, throughput the second: a few tokens per second is fine for a batch job, intolerable in chat.
- **Your users' phones are not your laptop.** Budget Android devices share a few gigabytes with everything else, so on-device models there run in hundreds of millions of parameters.
- **Nothing in the middle layers is inspectable.** A model's explanation of its own output is generated text, not a recording of what happened inside.

### Verify it

- **An invented parameter count.** Line-ups change monthly, so any count, context size or hardware claim from a chat window must be checked on the model's own page.
- **A model name you cannot find.** Names get invented by mixing a real family with a size that never shipped.
- **"This runs fine on free Colab" with no arithmetic.** Make it state four numbers: parameters, bytes per number, gigabytes, and the GPU's actual memory.
- **"Just run it on the user's phone."** Ask device RAM, size after quantization, and tokens per second on an old Android. Two will be guesses.
- **A confident account of a model's internal reasoning.** The honest version is "here is what the output suggests".
- **Check yourself:** print the GPU's free memory, load the model, print again. The difference rarely matches the estimate.

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
