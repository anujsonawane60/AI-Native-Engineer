---
id: m09-15-diffusion
module: m09-ai-foundations
slug: diffusion-and-generative-media
title: "Diffusion and Generative Media: From Noise to Image"
order: 15
duration_min: 25
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

A team needs forty photographs for their canteen app: one per dish, consistent lighting, appetising. The real canteen's dishes photographed on a phone under tube lights look grey and unappealing. Stock photos are either watermarked, wrong — North Indian thalis for a South Indian menu — or cost more than the team has.

So they generate them. Within an hour they have forty beautiful images, and three problems they did not anticipate.

The signboard in one image reads "CANTEFN SPECAIL". A hand reaching for a dosa has six fingers. And the biryani image is gorgeous, photorealistic, and shows a dish the canteen does not sell, plated in a way it will never be served — which a student will notice at the counter when the ₹80 they paid produces something else.

The first two are mechanical consequences of how diffusion works. The third is not a technical problem at all; it is a question about whether the image is honest, and it is the one that could actually get the team in trouble.

### The idea

**A diffusion model is trained to remove a small amount of noise from a noisy image. Run it on pure noise, repeatedly, each step guided by your text, and an image appears.**

The training trick is clever and simple. Take millions of real images, add noise to each in measured amounts, and train a model to predict what the noise was so it can be subtracted. Do that enough times and you have something that can take any noisy image and make it slightly less noisy — slightly more like a real photograph.

Now start from an image that is *entirely* noise. Ask for one step of noise removal. The result is still noise, but very slightly structured. Repeat twenty or fifty times, and the structure compounds into an image. Your text prompt steers every step, nudging the removal towards "a plate of pongal on a steel thali under warm light".

This is a fundamentally different process from the next-token prediction of chapter 8. Text is generated left to right, one token committed at a time. An image is generated all at once, everywhere, getting progressively sharper — which is why you cannot stream half an image usefully and why an image has a fixed cost rather than a per-token one.

### The analogy

The canteen's wall has a photograph of a thali, taken in 1998, now almost completely faded and stained.

A skilled restorer looks at it, sees a faint grey smudge where the rice should be, and knows from having seen ten thousand thalis what belongs there. She paints in a little. Then she steps back, looks again, and paints in a little more, now guided by what she just added. Twenty passes later there is a crisp, convincing thali on the wall.

And here is the thing: she was not recovering the original. She was constructing a plausible thali, consistent with the smudges and with everything she knows about thalis. If you handed her a blank sheet of grey noise and said "thali", she would do the same thing and produce something equally convincing.

**Where the analogy breaks:** the restorer knows she is inventing, and would tell you which parts she made up. The model does not distinguish between recovering and inventing — it produces a plausible image either way, with no record of which parts correspond to anything real. And the restorer knows what rice is. The model knows what rice *looks like next to* other things, which is why it can render a perfect grain of rice and a hand with six fingers in the same picture.

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

### Why hands, text and counting go wrong

Three failure modes, all explained by the same fact: the model learned what things look like in relation to each other, not what they are.

**Text inside images.** Letters are shapes that happen to carry meaning. The model has learned that signboards contain letter-like shapes in rows, so it produces letter-like shapes in rows. There is no spelling check anywhere in the process. Newer models are much better at this than older ones and still not reliable, so never generate an image that must contain exact words — put real text over the image in your own layout instead.

**Hands and fingers.** Hands are small, appear in a vast number of configurations, and are frequently partly hidden in training photos. Getting "five, in a plausible arrangement" right requires counting, and nothing in the process counts.

**Quantities generally.** "Four samosas on a plate" may give three or five. There is no counter — only a sense of what a plate of samosas looks like.

The practical rule that follows: **generated images are good for mood, texture, background and illustration, and unreliable for anything that must be exactly right.** If accuracy matters, photograph the real thing.

Two settings you will meet. **Steps** is how many noise-removal passes to run: more steps means slower and usually slightly better, with diminishing returns quickly. **Guidance** is how strictly to follow your prompt: too low and it ignores you, too high and the image looks over-saturated and strange. Both are dials to try, not values to get right in theory.

### Where it lives

Three practical facts place this on your map.

**Cost and latency are completely different from text.** A paragraph of text is fractions of a paisa and arrives in a second or two. An image is typically a few rupees and several seconds. Video is dramatically more again — a few seconds of video can cost more than a thousand text requests and take minutes. So generated media is almost never something you do live in a user request. You generate it ahead of time, store the file in object storage as M04 taught, and serve it like any other image. Generating on demand per user is a cost mistake you will only make once.

**Free tiers are tight and the landscape moves fast.** Image and video model line-ups, prices and free allowances change constantly. Read the provider's own page, as chapters 13 and 14 insisted, and expect any number you were told to be stale.

**Doing this properly is M14.** Running diffusion models yourself, fine-tuning one on your own product's look, controlling composition precisely and generating consistent characters across images are M14 chapter 9. Here you need the mechanism, the costs and the judgement.

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

**Licensing and training data are genuinely unsettled.** Image models were trained on enormous scrapes of the web, much of it copyrighted, and the law differs by country and is actively being litigated. Different providers offer different commercial terms and some offer indemnities. Before you put a generated image on anything commercial, read your provider's terms yourself. "An AI said it was fine" is not a defence.

**Never generate a likeness of a real person without their consent.** Your classmate, a cricketer, a politician, a local shopkeeper. This is not a style question; it is impersonation, it is covered by India's IT rules and by basic decency, and M07 is where the consequences live. Deepfakes of real people are the clearest ethical line in this module.

**A generated image of your product is a claim about your product.** A beautiful AI thali on your menu page, presented as what the student will receive, is a misrepresentation whether or not you intended one. Use generated images for illustration, background and mood; photograph what you actually sell. The honest test is simple: would a user feel misled if they knew?

**Consistency across images is hard.** Forty dishes in forty separate generations will have forty slightly different lighting setups, plate styles and angles, and a menu page made of them looks subtly wrong. Fixing this properly is the fine-tuning and control work in M14.

**Disclose it.** Marking generated images as generated costs nothing, protects you, and is increasingly expected or required. Provenance standards such as C2PA exist for exactly this. M07's ethics work and chapter 16's governance section both land here.

### Verify it

**Red flag: a generated image presented as a photograph of something real.** The question to ask of any image in your own product is "does this depict something that exists?" If not, it must not be positioned as though it does.

**Red flag: text in the image.** Read every letter. "CANTEFN SPECAIL" is the default outcome, not an unlucky one. If words must be exact, overlay real text in your layout.

**Red flag: hands, fingers, counts and symmetry.** Count the fingers. Count the samosas. Check that both earrings match and that the fork has four tines. These are the mechanical weak points and they are quick to check.

**Red flag: a cost estimate that treats images like text.** Ask for cost per image and per second of video, from the provider's current page, and ask whether generation happens per user request or once ahead of time. If it is per request, that is a design error.

**Red flag: silence on licensing and consent.** Ask directly whether the provider's terms permit commercial use of the output, whether any real person's likeness appears, and whether the image will be labelled as generated. An AI that recommends generated imagery without raising any of this has skipped the part that matters.

And the check to run yourself on every image before it ships: look at it for ten full seconds at full size, on a phone screen, specifically hunting for letters, fingers, counts and repeated patterns. You will find something in roughly one image in three, and that ten seconds is the entire quality process.

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
