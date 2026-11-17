---
id: m09-15-diffusion
module: m09-ai-foundations
slug: diffusion-and-generative-media
title: "Diffusion and Generative Media: From Noise to Image"
order: 15
duration_min: 15
is_free: false
mission: false
volatility: medium
last_reviewed: 2026-11-16
---

## Key learnings
- Explain diffusion as repeatedly removing a little noise, guided by your text
- Say why generated images get hands, text and counts wrong
- Compare the cost and latency of an image against a paragraph, and video against both
- Decide when a generated image is honest to use and when it misrepresents your product
- Catch a generated photo presented as a real photograph of a real thing

## Lesson

### The pain

A team needs forty appetising photographs for their canteen app. Real dishes shot under tube lights look grey, and stock photos are watermarked, wrong, or unaffordable — so they generate them, and within an hour have forty beautiful images and three problems. One signboard reads "CANTEFN SPECAIL". A hand has six fingers. And the gorgeous biryani shows a dish the canteen does not sell. The first two are mechanical consequences of how diffusion works; the third is a question about whether the image is honest, and it is the one that gets teams in trouble.

### The idea

**A diffusion model is trained to remove a small amount of noise from a noisy image. Run it on pure noise, repeatedly, each step guided by your text, and an image appears.**

The training trick is simple: take millions of real images, add measured amounts of noise, and train a model to predict the noise so it can be subtracted. Now start from pure noise and ask for one step of removal: the result is still noise, but slightly structured, and repeating fifty times compounds that structure into an image, with your prompt steering every step.

Unlike next-token prediction, an image is produced everywhere at once and gets progressively sharper — which is why it has a fixed cost rather than a per-token one.

### The analogy

The canteen's wall holds a faded photograph of a thali from 1998. A restorer sees a grey smudge where the rice should be and knows, from ten thousand thalis, what belongs there. She paints in a little, steps back, and paints in more, guided by what she just added; twenty passes later there is a convincing thali. She was not recovering the original but constructing a plausible thali consistent with the smudges — and handed blank noise, she would do the same thing.

**Where the analogy breaks:** the restorer knows she is inventing and would tell you which parts she made up. The model does not distinguish recovering from inventing. She also knows what rice is; the model knows what rice *looks like next to* other things, which is why it renders a perfect grain of rice and a six-fingered hand in one picture.

### Picture it

```
  TRAINING: learn to remove a little noise

   real photo ──add noise──▶ noisy photo ──▶ model guesses
                                             the noise it
                                             should remove

  GENERATING: start from pure noise, repeat

   ░░░░░░░    ░▒▒░▓░░    ▒▓▓▒▓▒▒    ▓███▓▓▒    ┌──────┐
   ░░░░░░░ ─▶ ░▓░▒░▓▒ ─▶ ▓▒███▒▓ ─▶ █████▓█ ─▶ │ dosa │
   ░░░░░░░    ▒░▓░░▒░    ▒▓███▓▒    ▓████▓▓    └──────┘
   step 0      step 10    step 25    step 40    step 50

   your text prompt steers EVERY step
```

Training teaches the model to subtract noise; generating starts from pure noise and subtracts repeatedly, with your prompt steering each step.

Three failure modes follow from one fact: the model learned what things look like next to each other, not what they are. **Text** comes out as letter-like shapes in rows, because nothing spells — overlay real text in your layout instead. **Hands** need five fingers in a plausible arrangement, which requires counting, and nothing counts. **Quantities** likewise: "four samosas" may give three. So **generated images are good for mood, texture and illustration, and unreliable for anything that must be exactly right.**

Two settings you will meet: **steps**, how many removal passes run, with fast diminishing returns; and **guidance**, how strictly the prompt is followed — too low and it ignores you, too high and the image looks over-saturated.

### Where it lives

- **Cost and latency are nothing like text.** A paragraph is fractions of a paisa; an image is a few rupees and several seconds; a few seconds of video can cost more than a thousand text requests.
- **So generate ahead of time**, store the file in object storage, and serve it like any other image. Generating per user request is a cost mistake you make once.
- **Free tiers are tight and the landscape moves fast**, so read the provider's own page and expect any number you were told to be stale.
- **Doing this properly is M14** — running diffusion yourself, fine-tuning for your product's look, controlling composition.

### Words to own

| Term | Plain meaning | Example |
|---|---|---|
| Diffusion | Generating by repeatedly removing noise | 50 steps from noise |
| Noise | Random pixel values with no structure | Step 0 |
| Steps | How many removal passes are run | 20 to 50 typical |
| Guidance | How strictly the prompt is followed | Too high looks strange |
| Latent space | A compressed space where the work happens | Faster than full pixels |
| Prompt | The text steering generation | "pongal on a steel thali" |
| Negative prompt | What you want kept out | "no text, no hands" |
| Image-to-image | Starting from your image instead of noise | Restyling a real photo |
| Inpainting | Regenerating only a masked region | Removing one object |
| Seed | The starting randomness, so results repeat | Same seed, same image |
| Watermark / C2PA | Markers recording that content was generated | Provenance standards |
| Deepfake | Generated media imitating a real person | A legal and ethical line |

### Tradeoffs and gotchas

- **Licensing and training data are genuinely unsettled.** Image models were trained on web scrapes, much of it copyrighted, and the law differs by country. Read your provider's terms yourself.
- **Never generate a likeness of a real person without their consent.** That is impersonation, covered by India's IT rules, and the clearest ethical line in this module.
- **A generated image of your product is a claim about your product.** An AI thali presented as what the student will receive is a misrepresentation whether you intended one or not. Would a user feel misled if they knew?
- **Consistency across images is hard.** Forty dishes in forty generations means forty lighting setups, and a menu page made of them looks subtly wrong.
- **Disclose it.** Marking generated images as generated costs nothing and is increasingly expected; C2PA exists for this.

### Verify it

- **A generated image presented as a photograph of something real.** Ask of every image: does this depict something that exists?
- **Text in the image.** Read every letter. "CANTEFN SPECAIL" is the default outcome, not an unlucky one.
- **Hands, fingers, counts and symmetry.** Count the fingers and the samosas; check both earrings match.
- **A cost estimate that treats images like text.** Ask cost per image and per second of video, and whether generation happens per request or ahead of time.
- **Silence on licensing and consent.** Ask whether the terms permit commercial use, whether a real likeness appears, and whether the image will be labelled.
- **Check yourself:** study each image for ten seconds at full size on a phone, hunting for letters, fingers and counts. One in three has something.

## Walkthrough
1. Open any free image generation playground and generate one image of a dish from your own product, with a plain prompt. Keep it.
2. Generate the same prompt again without changing anything. Compare the two images and write one sentence on what that tells you about repeatability.
3. Now add a signboard with specific words to your prompt and generate. Read every letter in the result and write down exactly what it says.
4. Generate an image with a specific count in the prompt — four samosas, three glasses. Count what you got. Record the result.
5. Generate an image containing a hand reaching for food. Count the fingers. Record the result.
6. Raise the step count substantially and regenerate. Note the change in quality and the change in time taken. Decide whether the extra wait was worth it.
7. Push the guidance setting high and regenerate. Describe what went wrong with the image in one sentence.
8. Find the cost per image on the provider's own pricing page, with today's date. Then find the cost per second of video if they offer it. Compare both against your text cost per request from chapter 14.
9. Decide, for your own product, which images will be generated and which must be real photographs. Write the rule in one sentence that someone else on your team could apply.
10. Read your provider's terms on commercial use of generated output and write down what they actually permit, in your own words, with a link.
11. Write one decision-log entry: where generated media is used, where it is forbidden, how it will be labelled, and the licensing terms you read with the date.

## Talk to AI
```prompt
I am deciding how to use generated images in my student startup, and I want the mechanics, the costs and the honesty question treated seriously. Do not write code.

My product: [what it is, who uses it]
Where I want images: [e.g. one photo per dish on a menu page, background art, illustrations in a blog]
Number of images I need: [X]
Who will see them: [e.g. paying students deciding what to order]

Please do all of this:

1. Explain diffusion to me in plain language: what the model was trained to do, and what happens step by step when it generates from noise. Make clear why this is a different process from how text is generated.

2. Explain specifically why text inside generated images comes out misspelled, why hands get extra fingers, and why counts are unreliable. Tie all three to the same underlying reason.

3. Go through my list of intended uses one at a time and classify each as HONEST TO GENERATE, RISKY, or MUST BE A REAL PHOTOGRAPH. For anything you mark risky or must-be-real, explain what a user would feel misled about.

4. Tell me what settings I should experiment with - steps, guidance, negative prompts, seeds - and what each one actually changes. Tell me which ones have diminishing returns quickly.

5. On cost: do NOT quote me prices. Tell me the shape of the cost - how an image compares to a paragraph of text, and how video compares to an image - and tell me which pricing pages to open. Then tell me whether I should generate per user request or ahead of time, and why.

6. On licensing: tell me the exact questions I must answer from my provider's terms before using any of this commercially, and tell me plainly that you cannot give me legal advice and that the terms differ by provider and country.

7. On consent and likeness: tell me what I must never generate, and why that line is not negotiable.

8. Finally, give me a ten-second checklist I can run on every generated image before it goes into my product.
```

## Stuck? Ask AI
```prompt
Explain diffusion using an analogy about sculpture or carving rather than photo restoration. Then tell me two ways your analogy misleads me about what is really happening, and ask me to guess one of them first.
```

```prompt
Explain why generated images get text, hands and counts wrong, all three from the same underlying cause. Then show me five things I should never ask an image model to produce, and five it is genuinely good at.
```

```prompt
I want to use generated images for my product's menu page. Argue forcefully that this is dishonest, then argue forcefully that it is acceptable, then tell me which specific uses fall on each side of the line and what single question decides it.
```

## Quiz

### Q1 · L2
How does a diffusion model generate an image?
- (a) It predicts the image one pixel at a time, left to right
- (b) It starts from pure noise and repeatedly removes a little, steered by your prompt ✅
- (c) It retrieves the closest matching image and edits it
- (d) It assembles the image from a library of parts
> It was trained to subtract noise from noisy images. Running that repeatedly from pure noise produces structure that compounds into a picture.

### Q2 · L3
Why is text inside generated images so often misspelled?
- (a) The model was trained only on non-English text
- (b) Letters are learned as shapes in plausible arrangements, with no spelling step anywhere ✅
- (c) The prompt was not specific enough about the words
- (d) Compression during generation corrupts fine detail
> The same cause explains six-fingered hands and wrong counts: the model learned appearances in relation to each other, and nothing in the process spells or counts.

### Q3 · L4
Your app needs forty dish images. Where should generation happen?
- (a) Live, inside each user's request, so images are always fresh
- (b) Ahead of time, stored as files and served like any other image ✅
- (c) On the user's device, to save server cost
- (d) Once per user session, cached in the browser
> An image costs orders of magnitude more than a text request and takes seconds. Generating per request is a cost and latency mistake; generate once and store it.

### Q4 · L4
You generate a photorealistic biryani for your menu page and the real dish looks quite different. What is the problem?
- (a) The guidance setting was too high
- (b) It misrepresents what the student will actually receive ✅
- (c) Generated images cannot be used commercially anywhere
- (d) The image needs more generation steps
> This is an honesty question rather than a technical one. Generated media is fine for mood and illustration; a depiction of what a customer will be served must be real.

### Q5 · L5
An AI proposes generated imagery for your commercial product and says nothing about licensing, likeness or labelling. What has it missed?
- (a) Nothing; those are legal matters, not technical ones
- (b) The three things that actually create risk: commercial terms, consent for real likenesses, and disclosure ✅
- (c) The optimal number of generation steps
- (d) That generated images are always lower quality than photographs
> Training data and commercial rights are unsettled and vary by provider and country, generating a real person's likeness without consent is impersonation, and labelling generated content is cheap protection that is increasingly expected.

## Go deeper
- [Google: how diffusion models work, in the ML crash course](https://developers.google.com/machine-learning/gan/gan_structure)
- [C2PA: content provenance and authenticity standard](https://c2pa.org/)
