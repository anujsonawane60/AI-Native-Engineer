---
id: m14-06-quantization
module: m14-deeper-ml-agents
slug: quantization
title: "Quantization and Running Models on Cheap Hardware"
order: 6
duration_min: 25
is_free: false
mission: true
volatility: medium
last_reviewed: 2026-11-16
---

## Key learnings
- Quantize one model to three precisions and measure memory, speed and quality for each
- Turn parameters and context length into a memory figure before downloading anything
- Name what degrades first when precision drops, and test exactly those things
- Run a usable model on a laptop with no GPU, and say honestly what will not run
- Catch a quantized model that was never compared against the original

## Lesson
### The pain

M09 chapter 5 gave you the arithmetic — parameters times bytes per number — and glossed quantization as "fewer bytes, smaller and faster". New here is doing it on hardware you own and finding out what it cost you.

A team runs their assistant on the office laptop to avoid an inference bill. They download a 4-bit build, it answers, they ship it. Two weeks later it totals four items wrong by ₹40 and answers a Marathi question with fluent nonsense. The original was never scored, so they cannot tell "this model is not good enough" from "we compressed it too hard".

### The idea

**Quantization stores each weight in fewer bits, which buys memory and speed and costs accuracy — and the only honest way to choose a level is to score the same eval set at every level including the original.**

Thirty-two bits is training precision. Sixteen is the serving standard: half the memory, essentially no loss. Eight loses a little. Four loses something real but often acceptable. Below four, not worth serving. "Acceptable" is a property of your task, which makes this a decision and not a setting.

### The analogy

The canteen's recipe book lists 237 grams of rice and 18.4 grams of salt. Rounded to the nearest 10 grams it fills a quarter of the pages and every dish tastes the same. Round to 50 and the sweets, where sugar matters to the gram, come out wrong. The only way to know which rounding went too far is to cook the same ten dishes from each book.

**Where it breaks.** A cook reading "100 grams" knows it is approximate and compensates. A model does not know its numbers were rounded, so degradation arrives as a confident wrong total, not as hesitation.

### Picture it

```
  bits  bytes/param  a 3B model   what it costs you
  ────────────────────────────────────────────────────────────
   32      4.0          12.0 GB   training precision; pointless
                                  for serving
   16      2.0           6.0 GB   the serving default; no real
                                  quality loss
    8      1.0           3.0 GB   small loss; fits an 8 GB
                                  laptop with room
    4      0.5           1.5 GB   real but often acceptable
                                  loss; runs on a phone
    3      0.4           1.1 GB   usually not worth serving
  ────────────────────────────────────────────────────────────
  plus the KV cache, which grows with context length and
  can exceed the weights on a long conversation
```

Each halving of bits per weight halves the memory the weights occupy, and the key-value cache for the conversation sits on top of that and grows with how much context you send.

### What degrades first

Precision loss is not uniform. Your eval set must cover these four.

- **Arithmetic and exact numbers** go first. If money passes through your feature, this alone may decide the level.
- **Long-context behaviour** goes next: the middle of a long input gets ignored.
- **Low-resource languages** degrade faster than English: fine in Marathi at 16 bits, unreliable at 4, and an English-only eval set will not notice.
- **Instruction following at the margins** — exact JSON, refusals — gets looser, undoing part of a chapter 4 fine-tune.

### Tools today

On a laptop with no GPU, llama.cpp is the engine and GGUF the file format; Ollama and LM Studio are friendlier wrappers that will download a quantized build for you. On a GPU, bitsandbytes gives quick 8-bit and 4-bit loading, while AWQ and GPTQ builds are what you serve. A ₹35,000 laptop with 8 GB of RAM and no GPU runs a 1-to-3-billion-parameter model at 4-bit at usable speed; 16 GB runs a 7-to-8-billion model slowly but genuinely. A 70-billion model does not run on your laptop, and no quantization level changes that. Every name and number here moves — check it this week.

### Where it lives

Downstream of chapter 4, upstream of chapter 7: merge, quantize, serve. A model on hardware you own has no per-request price — only latency and quality costs.

- Chapter 10 — the regression suite you run at every precision level
- M05 — latency: a 200-token answer at 6 tokens a second is 33 seconds

### Words to own

| Term | Plain meaning | Example |
|---|---|---|
| Precision | How many bits store each weight | 16, 8 or 4 |
| Quantization | Storing weights in fewer bits | 4-bit build |
| Post-training quantization | Compressing an already-trained model | What you will do |
| Weight-only | Weights compressed, activations left alone | Most common choice |
| GGUF | A quantized file format for CPU-first serving | Laptop and phone |
| K-quant | Mixed levels, sensitive parts kept higher | Q4_K_M and similar |
| KV cache | Memory holding the conversation so far | Grows with context |
| Tokens per second | Output speed once generation starts | 6 on a cheap laptop |
| Time to first token | The wait before anything appears | Prefill time |
| Perplexity | A generic language-fit score | A poor proxy for your task |
| Quality gap | Original score minus quantized score | 3 points, accepted |

### Tradeoffs and gotchas

**"It fits" and "it works" are different claims.** A model occupying 7.6 GB of an 8 GB laptop will load, answer, then stall once a long conversation grows the KV cache. Test your longest realistic input.

**Quantize after merging, not before.** Merge a chapter 4 adapter into the base at full precision, then quantize. The other order produces something subtly wrong — a tune that mysteriously stopped working.

**A small model at 8-bit usually beats a bigger one at 3-bit.**

**Streaming changes the economics of slow hardware:** six tokens a second is intolerable as a wait and tolerable as text appearing.

**Heat is a real constraint:** sustained inference throttles.

### Verify it

The named failure here is a quantized model never compared against the original.

1. **Demand the comparison table, original included** — each level with memory, latency, tokens per second and your eval score.
2. **Refuse perplexity as the quality column.** It barely moves while arithmetic accuracy falls off a cliff.
3. **Check the eval set covers what degrades first:** money, a long input, every user language, a strict format.
4. **Ask for the memory arithmetic in advance,** including KV cache at your longest context.
5. **Ask for the quality gap as a number, and whether you accept it** — the output here is a sentence with two numbers, not a model file.

Red flags: no original-precision baseline; perplexity quoted as quality; an English-only eval set for Marathi users; warm-run speed only; no KV cache allowance.

## Walkthrough
1. Inventory the cheapest hardware your team genuinely owns: RAM, whether there is a usable GPU, and free disk space. Write the numbers down. This is your budget.
2. Before downloading anything, compute what fits: parameters times bytes per weight at 16, 8 and 4 bits, plus a KV-cache allowance for your longest realistic conversation, plus 2 GB for the operating system. List the levels that are actually available to you.
3. Freeze an eval set of 25 to 30 real cases that deliberately includes four things: a money calculation, a very long input, at least two cases in each language your users write in, and one strict-format output.
4. On a free notebook GPU, load your chosen model at 16-bit and score the eval set. Record the score, the memory used, the time to first token and the tokens per second. This row is your baseline.
5. Download or build the same model at 8-bit and at 4-bit. If you have a chapter 4 adapter, merge it into the base at full precision first, then quantize.
6. Score both quantized versions on the identical eval set and fill in their rows. Do not change a single case between runs.
7. Now move to the cheap laptop with no GPU. Load the 4-bit build and score the same eval set there. Record cold and warm time to first token separately, and tokens per second.
8. Run the long-input case on the laptop and watch memory while it runs. Record what happens as the KV cache grows. If it stalls or crashes, that is your real context limit.
9. Run the four degradation categories side by side across all rows — money, long input, each language, strict format — and mark exactly which cells broke at which precision.
10. Keep it running for ten minutes of continuous use and record whether tokens per second fell. Note the device getting hot if it does.
11. Pick one configuration and write the decision in one sentence containing the quality gap and the speed, plus the exit criterion that would make you change level.

## Talk to AI
```prompt
CONTEXT:      I am a student on an AI-native engineering program. I direct AI
              and verify its output; I do not write code by hand. I know that
              fewer bits per weight means a smaller and faster model and
              eventually a worse one, and I know how to multiply parameters by
              bytes, so do not re-teach those. My team of three runs [product
              in two sentences] with real users in India. I want to run a
              model on hardware we already own instead of paying per request.
              The cheapest machine we have: [RAM] GB of RAM, [GPU or "no
              usable GPU"], [disk] GB free. Our users write in English and
              [language]. Our longest realistic input is about [number] words.
              Our feature involves [money calculations: yes/no] and [a strict
              output format: yes/no].

GOAL:         Choose one model and one precision level, justified by a table
              that includes the original precision, and know the quality gap
              I accepted as a number.

REQUIREMENTS: 1. Before recommending anything, compute what fits on my
                 machine: weights at 16, 8 and 4 bits for the model sizes I
                 could consider, plus a KV-cache allowance at my longest
                 input, plus operating-system headroom. Show the arithmetic
              2. Recommend a model size and a quantization format for a
                 machine with no usable GPU, and a different one if I use a
                 free notebook GPU. Explain why the formats differ
              3. Tell me the order in which capabilities degrade as precision
                 drops, and give me 25 eval cases that specifically test that
                 order - money arithmetic, a long input, both my languages,
                 and a strict format
              4. Give me the exact comparison table to fill in: one row per
                 precision INCLUDING the original 16-bit, with columns for
                 memory used, time to first token cold, time to first token
                 warm, tokens per second, and my eval score
              5. Tell me why perplexity is a poor quality measure for my
                 decision, and what it would fail to notice
              6. If I have a LoRA adapter, tell me the correct order of
                 merging and quantizing, and what goes wrong in the other
                 order
              7. Tell me what tokens-per-second figure makes my feature usable
                 with streaming and what makes it unusable even with it,
                 using my longest expected answer length
              8. Tell me the three measurements I should take after ten
                 minutes of continuous use on a cheap laptop, and why

CONSTRAINTS:  Free tier and hardware I already own; no paid GPU and no cloud
              bill. Plain English for a second-language reader. Label every
              command READ-ONLY or CHANGES-THINGS. My internet is slow, so
              tell me the download size of anything before I start. Treat all
              tool names, model names and file formats as stale and tell me to
              check them today.

NON-GOALS:    Do not propose a paid inference provider. Do not propose a model
              that cannot fit on my stated hardware. Do not use perplexity as
              the quality metric. Do not recommend the lowest precision simply
              because it is smallest. Do not re-explain what a weight is.

ACCEPTANCE:   - The memory arithmetic is shown before any download
              - My comparison table has a 16-bit original row
              - My eval set contains money arithmetic, a long input, both
                languages and a strict-format case
              - Cold and warm time to first token are recorded separately
              - I know my real context limit from watching memory during a
                long input
              - I have a quality gap expressed as a number and a sentence
                saying whether I accept it
              - An exit criterion exists that would make me change level

ASK:          First do the memory arithmetic for my machine and tell me which
              precision levels are even available to me, and which model sizes
              are out of reach entirely. Do not recommend a model until you
              have shown me that.
```

## Mission

**Objective.** Produce a four-row comparison table for one model at several precisions — original included — measured on hardware your team actually owns, and a one-sentence decision naming the quality gap you accepted.

**Steps.**
1. **Frame** — write which AI feature of your product you want to run on your own hardware, and what it currently costs per month to run on a provider.
2. **Budget** — inventory your cheapest machine and compute, on paper, which precisions and model sizes fit including a KV-cache allowance.
3. **Freeze the eval set** — 25 to 30 real cases covering money arithmetic, a long input, every language your users write in, and one strict-format output.
4. **Baseline** — score the model at 16-bit on a free notebook GPU. Record score, memory, time to first token, tokens per second.
5. **Direct** — produce 8-bit and 4-bit versions. If you have a chapter 4 adapter, merge before quantizing.
6. **Measure** — score every version on the identical eval set, then repeat the 4-bit run on the cheap laptop and record cold and warm speeds.
7. **Verify** — mark exactly which eval categories broke at which precision. Run the long input while watching memory and find your real context limit. Measure speed again after ten minutes of use.
8. **Decide** — choose one configuration and write the trade in one sentence with the quality gap and the speed in it.
9. **Document** — a decision-log entry with the table, the decision, the exit criterion, and what this changes about your cost per request.

**Upload.**
- Your Director's Brief → post caption or an attached file
- Your memory arithmetic, done before downloading → photo or text
- Your frozen eval set → file or screenshot
- The full comparison table with the 16-bit row → screenshot
- A screenshot of the model running on the cheap laptop, with memory visible
- Your per-category breakdown of what broke at which precision
- Cold and warm time-to-first-token numbers, and tokens per second after ten minutes
- The AI conversation → share link in `ai_chat_url`
- Reflection, 150 words max → one tradeoff you chose, and one thing the AI got wrong

**Self-check.** Tick every box before you submit.
- [ ] My table has a row for the original precision
- [ ] The same eval set, unchanged, was used for every row
- [ ] My eval set includes a money calculation and a long input
- [ ] My eval set includes cases in every language my users write in
- [ ] I did the memory arithmetic before downloading anything
- [ ] I allowed for the KV cache at my longest realistic input
- [ ] I recorded cold and warm time to first token separately
- [ ] I found my real context limit by watching memory, not by guessing
- [ ] I merged any adapter before quantizing, not after
- [ ] My decision sentence contains the quality gap as a number
- [ ] I did not use perplexity as my quality measure

**Stretch.** Compare a smaller model at 8-bit against a larger model at 4-bit, where both occupy roughly the same memory. Report which wins on your eval set, and whether that surprised you.

## Stuck? Ask AI
```prompt
I have [RAM] GB of RAM and [GPU or no GPU]. I want to run a model of about [number] billion parameters. Do the memory arithmetic with me step by step: weights at 16, 8 and 4 bits; then the KV cache for an input of [number] words and an answer of [number] words; then operating-system headroom. Tell me which precisions fit, which do not, and at what input length the one I choose will run out. Show every multiplication.
```

```prompt
My 4-bit model gets a four-item total wrong by ₹40 and answers in fluent nonsense when a user writes in [language]. Explain which of these is a quantization symptom and which might be the model itself, and give me the exact experiment that distinguishes them. Then tell me what my eval set must contain to catch each failure in future, with five example cases for each.
```

```prompt
Explain why perplexity can stay almost unchanged while a quantized model's arithmetic accuracy collapses. Use a concrete example. Then tell me what measurements I should use instead for a product that handles money and two languages, and give me the exact table of columns I should be filling in for each precision level.
```

## Quiz

### Q1 · L3
A 3-billion-parameter model at 4-bit occupies roughly how much for its weights?
- (a) About 12 GB
- (b) About 6 GB
- (c) About 1.5 GB ✅
- (d) About 0.4 GB
> Four bits is half a byte per parameter, so 3 billion parameters is about 1.5 GB — before the key-value cache, which grows with context and is the part people forget.

### Q2 · L4
Which quality measure should decide your precision level?
- (a) Perplexity on a standard text corpus
- (b) Your own eval set, including money arithmetic and every language you serve ✅
- (c) The model's published benchmark scores
- (d) Tokens per second on your laptop
> Perplexity can barely move while exactly the capabilities you depend on collapse. Precision damage is task-specific, so the measurement has to be your tasks.

### Q3 · L4
Your 4-bit model loads in 7.6 GB on an 8 GB laptop and works in testing, then crashes in use. What is the most likely cause?
- (a) The model file was corrupted during download
- (b) The key-value cache grew with a long conversation and exhausted memory ✅
- (c) The quantization level was too high
- (d) The laptop's CPU is too slow
> Weights are a fixed cost; the KV cache is not. Short test prompts hide it. Budget for the cache at your longest realistic input and test with that input.

### Q4 · L4
You have a LoRA adapter from chapter 4 and want a quantized model. What is the correct order?
- (a) Quantize the base, then merge the adapter into it
- (b) Merge the adapter at full precision, then quantize the result ✅
- (c) Quantize the adapter and the base separately, then merge
- (d) Order does not matter
> Merging into already-quantized weights produces something subtly wrong, and the symptom is a tune that appears to have stopped working. Merge first, at full precision, then compress.

### Q5 · L5
A team ships a 4-bit local model with no comparison against the original. Two weeks later quality complaints arrive. What specific thing can they no longer determine?
- (a) How fast the model runs
- (b) How much memory it uses
- (c) Whether the model is simply not good enough, or whether they compressed it too far ✅
- (d) Which quantization format they used
> Those two have opposite fixes — a bigger model versus a higher precision — and only a baseline at original precision separates them. Without it, every later debugging step is guesswork.

## Go deeper
- [llama.cpp: quantization formats and what each level trades away](https://github.com/ggml-org/llama.cpp/blob/master/examples/quantize/README.md)
- [Hugging Face: quantization concepts and supported schemes](https://huggingface.co/docs/transformers/en/quantization/overview)
