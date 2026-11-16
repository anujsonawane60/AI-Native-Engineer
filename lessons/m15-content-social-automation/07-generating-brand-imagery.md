---
id: m15-07-brand-imagery
module: m15-content-social-automation
slug: generating-brand-imagery
title: "Generating Brand Imagery and Video With AI"
order: 7
duration_min: 35
is_free: false
mission: true
volatility: high
last_reviewed: 2026-11-16
---

## Key learnings
- Lock one look across a whole set using a reference, a fixed style block and a seed
- Direct generative media with references and bans instead of adjectives
- Produce every channel's aspect ratio and safe area from one source asset
- Catch fabricated product detail, broken in-image text and stereotype defaults
- Keep rights, consent, disclosure and provenance straight before anything goes public

## Lesson

### The pain

A team needs twelve assets for a week of posts. They generate all twelve in forty minutes and feel extremely productive. Then they look at them together.

Twelve different illustration styles: two flat-vector, three photoreal, one watercolour, one 3D-render, the rest somewhere in between. Every image meant to show "an Indian college student" shows the same stock fantasy — a woman in a bright saree in a wood-panelled library — and one post about Nashik has the Taj Mahal in the background. Text generated inside two images reads `TIFFIN SERVICF` and `Mothly Plam`. One image shows a product screen with a map view and a loyalty-points widget; neither exists, and a user messages asking where to find them. One face is uncomfortably close to a recognisable actor. The logo, redrawn by the model, has the wrong spacing. And all twelve are square, so the vertical versions crop the price straight off the bottom.

Everything the team gained in forty minutes they spend over the next two days, and the account looks like twelve different companies — chapter 6's grid test, failed in a single week.

### The idea

**Generative media is directed, not requested. A set looks like one brand because you fixed the things that vary — a reference, a style block, a seed, a palette, a framing rule — not because you asked nicely twelve times.**

M03 chapter 8 taught this for interfaces and chapter 5 taught it for prose. Images are the same problem with higher variance: without constraints, every generation lands somewhere different on the average of everything the model has seen.

### The analogy

The canteen's photo board. Hire one person for one afternoon: one camera, one window for light, one steel tray, one angle, forty dishes. The board looks like a board. Now instead ask twelve students to each send one photo from their own phone — twelve lightings, twelve angles, twelve colour casts — and it looks like a group chat.

**Where the analogy breaks**, in the way that matters most in this chapter: a camera can only photograph food the kitchen actually made. A model will happily render a dish your kitchen cannot cook, a product screen you have not built, and a phone your users do not own. The photographer's constraint is physical; yours has to be a rule you enforce.

### Picture it

```
      ┌──────────────────────────┐
      │ SOURCE  2048 x 2048      │  one generation:
      │ subject centred,         │  reference image +
      │ price in the middle,     │  fixed style block +
      │ brand accent only        │  recorded seed
      └───┬────────┬────────┬────┘
          ▼        ▼        ▼
      ┌───────┐ ┌──────┐ ┌──────┐
      │ 1:1   │ │ 4:5  │ │ 9:16 │   text sits inside the
      │ feed  │ │ feed │ │ reel │   safe area in all three
      └───────┘ └──────┘ └──────┘
```

One generated source image, composed with everything important in the middle, is cropped to each channel's ratio afterwards — so the price survives the vertical crop instead of being cut off by it.

### Five levers that lock a set together

**1. A reference image, not a description.** "A young man in a kurta holding a tiffin" produces a different man every time. The same reference photo passed in as an image input produces something recognisably related. For product and person consistency, references beat adjectives every single time.

**2. A style block you paste verbatim, every generation.** Ten to fifteen words fixing medium, light, palette and background: *flat illustration, two colours only — steel grey and one hot orange, no gradient, plain background, thick even outlines.* Never retype it. Never improve it mid-set.

**3. The seed.** M09 taught you that a diffusion model starts from random noise and denoises toward an image. The seed is the number that chooses that starting noise. The same prompt with the same seed gives you the same picture; the same seed with a small prompt change gives you a *related* picture. Record the seed with every asset you keep — it is the only way to make a variation of something later instead of starting over.

**4. A palette and a ban list.** One accent colour from `DESIGN.md`, and a written list of what must never appear: no gradients, no glow, no lens flare, no 3D blobs, no text inside the image, no stock-office backgrounds, no national monuments.

**5. One framing rule.** Subject centred, breathing room on all four sides, nothing important in the outer tenth. This single rule is what makes one source asset survive three crops.

### Text, stereotypes and invented products

Three failure modes specific to this chapter.

**Do not let the model write text inside images.** It is unreliable at any length and will produce `TIFFIN SERVICF` at exactly the wrong moment. Generate a clean background, then place real text on top with your chapter 6 template, in your real typeface. Your text is then also correct in every language, which matters when the caption is in Marathi.

**The "generic India" default is real, and it is the content-side generic look.** Ask for "Indian students" and you will often get marigolds, a saree, a rickshaw, and a monument from 1,200 km away. Nashik is not a stock photograph. The fix is the same as M03 chapter 8's: your own reference photos. Photograph your actual street, your actual users' hands, your actual packaging. Ten photos you took are worth more than a hundred generations.

**Never generate your own product.** No generated screenshots, no generated UI, no generated features. A screen recording of the real thing is faster, correct, and does not invite a user to ask where the map view is. This is the single most damaging mistake in the chapter, because it is a promise you cannot keep.

### Video, briefly and honestly

For a 30-to-60-second vertical clip, three practical rules. Keep generated shots under about four seconds each, because models lose continuity fast and faces drift. Burn captions into the video, because most people in a feed are watching muted. And prefer real footage for anything about your product: a screen recording with a voiceover beats any generation, and takes less time.

Generated b-roll between real shots is a reasonable use. Generated product demonstrations are not.

### Rights, consent, disclosure, provenance

Four things, each of which has ended somebody's week.

**Rights.** Whether you own generated output varies by jurisdiction and by the terms of the model you used, and in India the position on purely machine-generated work is not settled. Read the terms of whatever you use, and check whether the free tier permits commercial use at all before you put an asset on a pricing page.

**Consent.** Do not generate recognisable real people, celebrities or look-alikes. For real customers, get written permission before their face appears anywhere, and do not use a model to "improve" a real person's photo.

**Disclosure.** Several platforms require synthetic media to be labelled, and several detect and label it themselves. Label it. A team caught passing a generated image off as a photograph of their own kitchen loses more than the post.

**Provenance.** Store every kept asset with its prompt, style block, seed, model, date and source references — a four-line note beside the file in your object storage from M04. Six weeks later, when you need one more image in the same style, this file is the difference between thirty seconds and an afternoon.

**Tools today:** model names, free quotas and feature sets in this area change every few months, which is why this chapter is tagged high-volatility. The five levers above do not change. Current tool names belong in Go deeper, not in your head.

### Where it lives

M09 is the theory under this chapter: latents, guidance strength, references and style adapters. M03's design system supplies the palette and type; chapter 6's kit supplies the ratios and safe areas; chapter 8 posts the results per channel; M04's object storage is where the assets and their provenance notes live; and M12's store screenshots are another surface that must match the same kit.

### Words to own

| Term | Plain meaning | Example |
|---|---|---|
| Reference image | An image fed in to steer the output | Your own street photo |
| Style block | Fixed wording pasted every generation | "Flat, two colours, no gradient" |
| Seed | The number choosing the starting noise | Seed 48211, recorded |
| Negative prompt | A list of what must not appear | "No text, no gradient" |
| Aspect ratio | Shape of the frame | 1:1, 4:5, 9:16 |
| Safe area | Margin platform controls may cover | Outer 10% kept empty |
| Upscale | Enlarging without losing detail | 1024 to 2048 |
| Consistency set | Assets that visibly belong together | Twelve posts, one look |
| Provenance | The record of how an asset was made | Prompt, seed, model, date |
| Synthetic media | Images or video made by a model | Labelled on publish |
| Likeness | A recognisable real person's appearance | Needs consent, always |
| B-roll | Supporting footage between main shots | Generated street scene |

### Tradeoffs and gotchas

**Your own photos beat generation more often than students expect.** A real photo of your actual packaging on an actual Nashik street is specific, free, instantly on-brand, and impossible for a competitor to copy.

**Variance is the enemy of a brand.** Models are built to be creative, which means different every time. Everything in this chapter is a way to suppress that.

**Free tiers are small and they move.** Plan the set before you start generating, or you will spend your month's quota on experiments and have nothing for the posts.

**Do not generate your logo.** Ever. Use the file.

**Generated faces age badly.** Hands, teeth, jewellery and text drift. Prefer hands, objects, food, places and backs of heads to close-up faces.

**One good image used four ways beats four mediocre ones.** The crop work from one strong source asset is twenty minutes and consistent by construction.

### Verify it

**Red flag: an image set with inconsistent brand.** Put all your assets in one grid at the same size. Any outlier in medium, light, palette or outline weight means the style block was not pasted or was edited mid-set. This is the fastest check here, and it is chapter 6's grid test again.

**Red flag: text inside a generated image.** Zoom in and read every word, including small background text. One misspelling published is a permanent screenshot.

**Red flag: product detail that does not exist.** Open your real product beside the asset and compare element by element. Any screen, feature, button or claim not in the live product must be removed, not "fixed later".

**Red flag: the stereotype default.** Ask yourself whether this image could have been generated by someone who has never been to your city. Monuments, marigolds and generic saree-and-library framing are the tells.

**Red flag: a recognisable face.** If anyone on your team says "that looks a bit like…", discard it.

**Red flag: no seed, no prompt, no provenance.** An asset you cannot regenerate or vary is a dead end, and you will need a variation within two weeks.

**Red flag: the crop test fails.** Crop every asset to 1:1, 4:5 and 9:16 and check that nothing important is lost. If the price or the subject disappears, the framing rule was ignored.

**Red flag: the model calling its own output "vibrant" or "eye-catching".** The same vocabulary problem as "modern and clean" in M03 — ask instead which of your constraints produced each visual decision.

## Walkthrough
1. Write down the exact asset list you need for one week of posting, with the channel and ratio for each. Plan the set before generating anything, because free quotas are small.
2. Go and photograph ten real things yourself: your packaging, your street, your users' hands, your actual workspace. These are your reference images.
3. Write a style block of ten to fifteen words fixing medium, light, palette and background, using your accent colour from `DESIGN.md`. Save it somewhere you can paste from.
4. Write a ban list for images: no text in the image, no gradients, no glow, no 3D blobs, no monuments, no stock offices, no generated faces close up, no generated product screens.
5. Generate ONE image with a reference, your style block, your bans and a recorded seed. Judge it on a phone, not a laptop.
6. Generate three variations by changing one small thing while keeping the seed. Note which lever moved the output most. That is the lever you will use for the rest of the set.
7. Produce the full set using the identical style block each time. Record prompt, seed, model and date beside every file you keep.
8. Crop every asset to 1:1, 4:5 and 9:16 and check the safe area. Add your real text on top with your chapter 6 template, never inside the generation.
9. Lay the whole set out in one grid at the same size and find the outliers. Regenerate the outliers with the original style block rather than editing them.
10. Compare every asset against your live product and remove anything it shows that you do not actually have. Then label the synthetic assets as required and file the provenance notes.

## Talk to AI
```prompt
CONTEXT:      I am a student founder. I do not write code; I direct AI.
              My product: [one sentence]. My city: [e.g. Nashik].
              My users: [who, what phone, what they care about].

              My brand from DESIGN.md and VOICE.md:
              - accent colour: [hex], used only for [what]
              - typeface: [name]
              - three voice traits: [list]
              My reference photos, described: [the ten photos I took
              myself - what is in each, light, colours]

              Assets I need this week, with channel and ratio:
              [list each one, e.g. "1 hero 1:1 for the site, 3 feed 4:5
              for Instagram, 1 vertical 9:16 for Reels"]

GOAL:         A directing plan for a set of images that visibly belong
              to one brand, plus the exact style block I will paste
              every time.

REQUIREMENTS: 1. A style block of 10-15 words fixing medium, light,
                 palette and background, using only my accent colour
              2. For each asset, which of my reference photos to use,
                 exactly what to take from it, and what to take
                 nothing of
              3. One framing rule that lets ONE source image survive
                 crops to 1:1, 4:5 and 9:16 without losing the subject
              4. A negative list of at least eight things that must
                 never appear in any image
              5. A provenance template: the four or five fields to
                 record beside every asset I keep

CONSTRAINTS:  Free tiers only, and small quotas - plan the set before
              generating. All text goes on top in my own typeface,
              never inside the generation. Every asset must read on a
              5-inch phone in sunlight.

NON-GOALS:    Do not describe my product's interface or invent any
              screen, feature or widget. Do not generate or redraw my
              logo. Do not propose close-up generated faces. Do not use
              national monuments, marigolds or generic "Indian college"
              imagery. Do not name specific tools or models.

ACCEPTANCE:   - One style block, under 15 words, pasteable verbatim
              - Every asset maps to one of MY reference photos
              - Zero instructions that would place text inside an image
              - Zero references to product features I did not list
              - A section titled "How this set drifts", naming the three
                most likely ways my twelve assets stop matching

ASK:          First, describe in detail what a careless version of this
              set would look like - the defaults you would reach for
              without my references. Then tell me which one constraint
              of mine most changes that. Only then give me the plan.
```

## Mission

**Objective.** A channel kit for your own startup — one hero image, three social images and one short vertical video — that passes the grid test, contains nothing your product does not have, and ships with a provenance record for every asset.

**Steps.**
1. **Frame** — list the five assets with their channel, ratio and the one job each has to do. One sentence each.
2. **Reference** — photograph ten real things yourself. Your street, your packaging, your users' hands, your workspace. No stock, no generation.
3. **Brief** — write the style block, the framing rule and the image ban list, using your accent colour and typeface from `DESIGN.md` and `VOICE.md`.
4. **Direct** — generate one asset first. Iterate it with a fixed seed until it is right, then produce the rest with the identical style block.
5. **Compose** — add all text on top in your own typeface, inside the safe area. Crop each asset to the three ratios.
6. **Video** — build one 30-to-60-second vertical clip. Use a real screen recording for anything showing the product. Burn in captions. Generated shots, if any, stay under four seconds each.
7. **Verify** — grid test for consistency, zoom test for in-image text, side-by-side against the live product for invented detail, crop test for safe areas.
8. **Record** — write prompt, style block, seed, model, date and references beside every kept asset, and store them in one place.
9. **Disclose** — label synthetic assets as the platform requires, and confirm the free tier you used permits this use.
10. **Document** — one decision-log entry: which lever gave you the most consistency, and what you chose to shoot rather than generate.

**Upload.**
- Your Director's Brief including the style block and ban list → attached file or post caption
- Your ten reference photographs → contact sheet screenshot
- The full asset set laid out as one grid at the same size → screenshot
- The same set cropped to 1:1, 4:5 and 9:16 showing the safe areas → screenshots
- Your vertical video → `demo_url` or an attached file
- Your provenance record for every asset → screenshot or table
- Your AI conversation → share link in `ai_chat_url`
- Your verification: the grid test, the zoom test and the product comparison → screenshots
- Reflection, 150 words max → one tradeoff you chose, one thing the AI got wrong

**Self-check.** Tick every box before you submit.
- [ ] All five assets in one grid look like one brand, with no outlier
- [ ] Exactly one accent colour appears across the whole set
- [ ] No text was generated inside any image; all text is in my own typeface
- [ ] I zoomed in and read every word in every asset, including background text
- [ ] No asset shows a screen, feature or claim my live product does not have
- [ ] No asset contains a close-up generated face or a recognisable real person
- [ ] Every asset survives crops to 1:1, 4:5 and 9:16 with the subject intact
- [ ] Every kept asset has its prompt, style block, seed, model and date recorded
- [ ] I checked the free tier's terms permit this use, and labelled synthetic media
- [ ] My reflection names a tradeoff in my own words

**Stretch.** Regenerate one asset six weeks later using only your provenance record, and see whether it matches. That is the real test of whether the record was good enough.

## Stuck? Ask AI
```prompt
I generated twelve images and they look like twelve different companies. Here is a detailed description of each one — medium, light, colours, outline weight, background, framing: [describe all twelve]. Do not generate anything. Tell me which are the outliers, which single variable differs most across the set, and what one fixed instruction would have prevented it.
```

```prompt
Explain what a seed is in image generation as if I have never heard of diffusion, and why the same seed with a slightly different prompt gives me a related image rather than a totally new one. Then tell me three situations where recording the seed saves me real time, and ask me which of them I am likely to hit first.
```

```prompt
Here is a reference photo I took myself, described in detail: [describe it]. List the eight visual qualities it actually has — light, palette, texture, framing, background, mood. Then ask me which three I want to borrow for my brand, and tell me what would happen if I borrowed all eight.
```

## Quiz

### Q1 · L3
You generate twelve assets and they look like twelve different companies. What is the most likely cause?
- (a) The model's quality varies between generations
- (b) The style block, reference and palette were not held fixed across the set ✅
- (c) Twelve is too many assets to generate in one session
- (d) The images needed upscaling
> Models are built for variety, so sameness has to be imposed. A pasted-verbatim style block, the same reference and one palette are what make a set hang together.

### Q2 · L4
Why should text never be generated inside an image?
- (a) Text in images cannot be indexed by search engines
- (b) Models render text unreliably, and a misspelling you publish is permanent ✅
- (c) It increases the file size too much
- (d) Platforms reject images containing text
> `TIFFIN SERVICF` is the classic outcome. Generating a clean background and placing real text on top in your own typeface is also what keeps a Marathi caption correct.

### Q3 · L4
An AI-generated marketing image shows your app with a map view and a loyalty widget. Neither exists. What should you do?
- (a) Publish it and build those features soon
- (b) Discard it — it promises something the product does not have, and users will ask ✅
- (c) Publish with a disclaimer in the caption
- (d) Keep it for internal use and publish a cropped version
> Never generate your own product. A screen recording of the real thing is faster and correct, and an invented feature in a public asset is a promise you cannot keep.

### Q4 · L2
What is the purpose of recording a seed alongside a generated asset?
- (a) It proves you own the copyright
- (b) It lets you regenerate or make a related variation later instead of starting over ✅
- (c) It improves the resolution on re-export
- (d) It is required for platform disclosure
> The seed chooses the starting noise, so the same seed with a small prompt change gives you a related image. Without it, "one more in that style" means starting from scratch.

### Q5 · L5
Your post about Nashik features marigolds, a saree-clad student in a wood-panelled library, and a monument 1,200 km away. The brief said "authentic Indian college vibe". What went wrong, and what is the fix?
- (a) The model lacks Indian training data
- (b) The adjective invited the model's average, so replace it with your own reference photographs ✅
- (c) The aspect ratio was wrong for the platform
- (d) Authenticity cannot be expressed in a prompt
> "Authentic Indian vibe" is M03 chapter 8's "modern and clean" wearing different clothes. Ten photos of your actual street beat any adjective, and nothing else gets you specificity.

## Go deeper
- [C2PA: content credentials and provenance for synthetic media](https://c2pa.org/)
- [Hugging Face: diffusion models course](https://huggingface.co/learn/diffusion-course)
