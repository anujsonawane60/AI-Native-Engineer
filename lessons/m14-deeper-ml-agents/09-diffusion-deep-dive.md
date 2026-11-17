---
id: m14-09-diffusion-deep-dive
module: m14-deeper-ml-agents
slug: diffusion-deep-dive
title: "Diffusion Deep Dive: Latents, Guidance, ControlNet, Style Adapters"
order: 9
duration_min: 15
is_free: false
mission: false
volatility: medium
last_reviewed: 2026-11-16
---

## Key learnings
- Explain the latent space concretely, and why it is what kills small text and faces
- Control composition with a structural condition instead of more adjectives
- Choose between an image reference and a trained style adapter for brand consistency
- Record a recipe complete enough that someone else reproduces your image exactly
- Catch a generated image whose settings nobody wrote down

## Lesson
### The pain

M09 chapter 15 taught diffusion as repeatedly removing noise and explained why hands and text go wrong. New here is running it yourself: controlling composition, holding a character consistent, reproducing the image that worked.

A team needs twelve images of the same delivery rider in six scenes. Rider one looks twenty, rider two forty, rider three wears a different uniform, and four signboards read "RESTURANT". They add adjectives, which changes everything except the thing they wanted. Then the image that finally looks right cannot be reproduced, because nobody recorded the seed, sampler, steps or guidance.

### The idea

**Diffusion gives you four separate control surfaces, and almost everyone tries to operate all four through the prompt.**

The **prompt** controls content and mood. A **structural condition** controls composition. A **reference image or trained adapter** controls style and identity. The **sampling settings** control how faithfully and expensively the denoising runs. Reaching for stronger wording when the problem belongs to another surface is the central mistake here.

### The analogy

The canteen wants photographs of its thali for a menu board. The prompt is the brief: "a full South Indian thali, warm light, steel plate". The structural condition is the chalk outline fixing where the plate, glass and papad go. The style reference is three old photographs with "match this look". The sampling settings are how many times the photographer re-shoots.

**Where it breaks.** A photographer who cannot read a smudged sign asks what it says. The model has no such channel: the sign is a few pixels wide in a compressed space, and it invents letters because inventing is its only operation.

### Picture it

```
  TEXT PROMPT ──▶ text encoder ──▶ ┌──────────────────┐
                                   │                  │
  STRUCTURAL MAP ────────────────▶ │  DENOISING LOOP  │
  (pose / depth / edges)           │                  │
                                   │  runs N steps    │
  STYLE REFERENCE or ADAPTER ────▶ │  in LATENT space │
                                   │  (image at 1/8   │
  random noise, from SEED ───────▶ │   scale, 64× less│
                                   │   work per step) │
                                   └────────┬─────────┘
                                            │ final latent
                                            ▼
                                     VAE DECODER
                                            │
                                            ▼
                                   full-size image
                                   (fine detail is
                                    reconstructed, not
                                    preserved — this is
                                    why small text fails)
```

Four inputs steer a denoising loop that works on an eight-times-smaller representation, and the decoder at the end reconstructs full-size detail rather than preserving it, which is the structural reason small text and small faces come out wrong.

### Steps, schedulers and the true cost of guidance

The **scheduler** is the timetable for removing noise. Classic schedulers want 30 to 50 steps, fast samplers 15 to 25, distilled variants 4 to 8. Quality rises steeply then flattens, so 80 steps pays for nothing.

**Guidance** needs a correction. Classifier-free guidance makes *two* predictions every step — one on your prompt, one unconditioned — and pushes away from the second, so guidance above 1 roughly **doubles compute per step**. The negative prompt is not separate: it replaces the unconditioned branch. Past a point, more guidance produces the burnt look everybody recognises as AI output.

### Tools today

A free notebook GPU runs the Stable Diffusion family comfortably, including structural conditioning and adapter training at modest resolution. The Diffusers library is the scriptable path and ComfyUI the visual one; both expose schedulers, guidance, ControlNet and adapters explicitly, which is what you want while learning. Model names, adapter formats and the best fast scheduler change every few months.

### Where it lives

- M15 and M03 — the content engine, and the design system your adapter should match
- Chapters 6 and 7 — a diffusion pipeline is a model being served
- M04 chapter 15 — object storage: generate once, reuse
- M09 chapter 15 — no likeness without consent, disclosure where a user could be misled

### Words to own

| Term | Plain meaning | Example |
|---|---|---|
| Latent | The compressed representation the loop works on | One-eighth scale |
| VAE | The encoder and decoder either side of the loop | Reconstructs detail |
| Scheduler / sampler | The timetable for removing noise | 20 steps, not 50 |
| Classifier-free guidance | Two predictions per step, pushed apart | Doubles compute |
| Guidance scale | How hard the prompt is enforced | Too high looks burnt |
| Denoise strength | How much of a source image is destroyed | 0.3 keeps most of it |
| ControlNet | A parallel network conditioned on a structural map | Pose, depth, edges |
| IP-Adapter | Style or identity taken from a reference image | No training needed |
| Style LoRA | A small adapter trained on 20–40 of your images | Brand consistency |
| Inpainting | Regenerating only a masked region | Fix one signboard |
| Recipe | Every setting needed to reproduce an image | Seed plus eight fields |

### Tradeoffs and gotchas

**A seed alone does not reproduce an image.** You need the seed, the model and its exact version, the scheduler, steps, guidance scale, resolution, every adapter with its weight, and often the library version. One field missing and you have a mystery PNG.

**Guidance is not a quality dial.** Raising it to fix composition gives a more saturated version of the wrong composition. Composition goes to structural conditioning, style to adapters.

**Latent compression decides what you can do about detail.** Generate larger, add real text afterwards as a layer, and inpaint one region rather than re-rolling the image.

**Consistency comes from a trained adapter, not words** — 20 to 40 images on a free notebook GPU in under an hour, under chapter 2's data rules, which include having the right to the images. An adapter trained on another illustrator's portfolio is a legal problem.

### Verify it

The red flag here is an image with no recipe: a result nobody can reproduce or defend.

1. **Demand the recipe card with the image.** Nine fields, written down.
2. **Make it name the control surface first:** content, composition, style, or sampling.
3. **Ask for the step comparison on your own prompt,** then hold the number.
4. **Ask what guidance above 1 costs.** No mention of two predictions per step means the estimate is wrong.
5. **For consistency, insist on an adapter or reference image.** Prompt-only claims should be rejected on sight.
6. **Ask where the training images came from** — source, licence, consent.

Red flags: no seed recorded; a recipe missing the scheduler or model version; guidance at 15 to fix composition; 80 steps by default; "consistent character" as a prompt phrase; an adapter trained on scraped artwork.

## Walkthrough
1. On a free notebook GPU, set up a diffusion pipeline you can script. Generate one image and print every setting the pipeline used. That printed block is your recipe template.
2. Run the identical prompt and seed at 10, 20, 30 and 50 steps. Place the four images side by side and mark where quality stops improving. Record your step number.
3. Run the same prompt and seed at guidance 3, 7, 12 and 20. Note where it starts looking burnt. Record the time per image at each setting and confirm for yourself that guidance costs real compute.
4. Generate a 1024-pixel image containing a signboard with your product's name. Zoom in on the text. Then generate the same scene at a larger resolution and compare the lettering. Write down what the latent compression did.
5. Fix the signboard two ways: once by inpainting only that region, and once by adding the text as a layer afterwards. Compare the effort and the result.
6. Add structural conditioning: take a photograph of a thali on a table, derive an edge or depth map from it, and generate three completely different dishes in that exact layout. Confirm the composition held across all three.
7. Take an image whose style you own and use it as a reference adapter. Generate four new scenes and judge how much of the style transferred without any training.
8. Now train a style adapter properly: collect 20 to 40 images you have the rights to, record their provenance, and train a LoRA on the diffusion model. Keep the run under an hour.
9. Generate the same six scenes three ways — prompt only, reference adapter, trained adapter — and lay out all eighteen images in a grid. This grid is the evidence for which tool consistency actually requires.
10. Measure cost: time per image on the free GPU, and what the same volume would cost on a rented GPU and on a hosted image API. Write the per-image figure in rupees.
11. Write recipe cards for your six final images and store them beside the files. Then hand one card to a teammate and have them reproduce the image on their own machine.

## Talk to AI
```prompt
CONTEXT:      I am a student on an AI-native engineering program. I direct AI
              and verify its output; I do not write code by hand. I already
              know diffusion as repeatedly removing noise steered by a prompt,
              and I know the words seed, steps, guidance, negative prompt,
              image-to-image and inpainting, so do not re-teach those. My team
              of three runs [product in two sentences] with real users in
              India. I need [number] images for [describe the use: app
              illustrations, menu photography, a recurring character]. They
              must look like they belong together. My constraints: a free
              notebook GPU, and the images will be shown [where]. I own
              [describe the images you have the rights to use for training].

GOAL:         A reproducible pipeline that produces consistent images, with
              composition controlled by a structural condition rather than by
              adjectives, and a recipe card per image.

REQUIREMENTS: 1. For each of my problems, tell me which control surface it
                 belongs to - content, composition, style and identity, or
                 sampling settings - and which tool handles it. Do this before
                 suggesting any prompt
              2. Give me the complete recipe card fields I must record to
                 reproduce an image exactly, and tell me what happens if each
                 one is missing
              3. Tell me how to run a step comparison and a guidance
                 comparison on my own prompt, and explain why guidance above 1
                 changes my cost per image
              4. Explain, for my resolution, how many latent positions my
                 signboard text will occupy and therefore whether legible text
                 is possible at all. Give me the two reliable workarounds
              5. Tell me which structural conditioning type - pose, depth,
                 edges or a region mask - fits my case, how to derive the map
                 from a photograph I take, and what it costs in memory and
                 time
              6. Compare an image-reference adapter against a trained style
                 adapter for MY consistency need: setup time, training time on
                 a free GPU, how many images I need, and how well each holds
                 across different scenes
              7. Tell me exactly what to check about my training images'
                 rights and provenance before I train anything
              8. Give me the cost per image in rupees three ways: free
                 notebook GPU, a rented hourly GPU, and a hosted image API,
                 and tell me where generated images should be stored

CONSTRAINTS:  Free tier only, one notebook GPU of about 15 GB. Plain English
              for a second-language reader. Label every cell READ-ONLY or
              CHANGES-THINGS. No likeness of any real person. Nothing
              generated may be presented as a photograph of real food we
              serve. Treat every model name, adapter format and scheduler
              recommendation as stale and tell me to check it today.

NON-GOALS:    Do not solve composition or consistency by adding adjectives to
              my prompt. Do not propose a paid image service before I have
              measured the free path. Do not train on images I do not own. Do
              not re-explain what diffusion is. Do not generate video.

ACCEPTANCE:   - Each of my problems is mapped to a control surface and a tool
              - A recipe card template exists with every field named
              - I have run a step comparison and fixed my step count
              - My composition held across three different generated contents
              - A side-by-side grid compares prompt-only, reference adapter
                and trained adapter consistency
              - Training-image provenance is recorded before any training
              - A per-image cost in rupees exists for all three paths
              - A teammate reproduced one of my images from its recipe alone

ASK:          First, take my list of problems and tell me which control
              surface each belongs to, and say which of them cannot be fixed
              by prompting at all. Do not write a prompt until you have.
```

## Stuck? Ask AI
```prompt
Explain, with numbers, why small text in a generated image comes out as letter-shaped nonsense. Use a 1024-pixel image with a signboard occupying 200 pixels, work out how many latent positions that is, and explain what the decoder does at the end. Then give me the two workarounds that actually work and tell me which one professional pipelines use. Do not suggest I change my prompt.
```

```prompt
I need the same character to appear in six different scenes and look like the same person. Compare three approaches for me: prompt wording with a fixed seed, an image-reference adapter, and a trained style adapter. For each, tell me the setup effort, how many images I need, how long training takes on a free notebook GPU, and how well it holds when the scene changes a lot. Then tell me which one you would choose for six scenes and why, and what will still be imperfect.
```

```prompt
Here is the recipe for an image I generated: [paste every setting you recorded]. Tell me whether this is enough for someone else to reproduce the exact image on a different machine. List every missing field, and for each one, tell me how much the image would differ if that field were wrong. Then give me the complete recipe card template I should be storing beside every image from now on.
```

## Quiz

### Q1 · L3
Why does small text in a generated image come out wrong?
- (a) The text encoder does not understand letters
- (b) The loop works on a compressed representation where the text occupies too few positions ✅
- (c) Guidance was set too low
- (d) The negative prompt was missing
> Denoising happens at roughly one-eighth scale, so a small sign is a handful of latent positions and the decoder reconstructs plausible rather than accurate detail. Generate larger, or add real text as a layer.

### Q2 · L4
What does raising the guidance scale above 1 cost you?
- (a) Nothing; it only changes prompt adherence
- (b) Roughly double the compute per step, because two predictions are made ✅
- (c) More memory for the key-value cache
- (d) A longer decoding pass at the end
> Classifier-free guidance runs a conditioned and an unconditioned prediction at every step and pushes them apart. That is where your negative prompt plugs in, and where the extra compute goes.

### Q3 · L4
You need six images with the same layout but different dishes. Which tool?
- (a) A longer, more detailed prompt
- (b) A higher guidance scale
- (c) Structural conditioning from a reference map ✅
- (d) A larger output resolution
> Layout is composition, and composition has its own control surface. A pose, depth or edge map fixes where things sit while the prompt supplies what they are — something no amount of wording achieves reliably.

### Q4 · L4
Your best image cannot be reproduced. Which single recorded field would most likely have been enough on its own?
- (a) The seed
- (b) None — reproduction needs the seed plus the model version, scheduler, steps, guidance, resolution and adapters ✅
- (c) The negative prompt
- (d) The resolution
> The seed only fixes the starting noise. Change the scheduler or the step count and the same seed produces a different image, which is why a recipe card has nine fields rather than one.

### Q5 · L5
A team trains a style adapter on 30 images downloaded from an illustrator's portfolio. The outputs look nothing like the originals. Is this acceptable?
- (a) Yes, because the generated images are new
- (b) Yes, because the adapter is only a few megabytes
- (c) No — the training data was used without rights, and output novelty does not change that ✅
- (d) No, but only if the illustrator is in India
> Chapter 2's provenance rule applies to image training exactly as to text: source, licence and consent recorded before training. Dissimilar output is not a defence for how the dataset was obtained.

## Go deeper
- [Hugging Face Diffusers: schedulers, guidance, ControlNet and adapters](https://huggingface.co/docs/diffusers/index)
- [Adding Conditional Control to Text-to-Image Diffusion Models (ControlNet)](https://arxiv.org/abs/2302.05543)
