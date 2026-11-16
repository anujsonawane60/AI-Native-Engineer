---
id: m03-08-generic-ai-look
module: m03-design
slug: avoiding-generic-ai-look
title: 'Avoiding the Generic "AI Look"'
order: 8
duration_min: 25
is_free: false
mission: false
volatility: medium
last_reviewed: 2026-11-16
---

## Key learnings
- Name the specific visual tells that mark a screen as AI-generated
- Direct style with references, constraints and bans instead of adjectives like "modern"
- Derive a look from your own content, users and region rather than from a default
- Run the swap test: if your screen would suit any other startup, it is generic
- Tell the difference between distinctive and merely unusual

## Lesson

### The pain

Demo day. Eleven teams present. Nine of the eleven products share a visual identity nobody chose.

A purple-to-blue gradient across the hero. The same geometric sans-serif. Three feature cards in a row, each with a thin line icon, each with a two-word heading and a one-line description. Frosted-glass panels over a blurred background. Everything centred. Rounded corners at exactly the same generous radius. A headline of the form "Transform your X with AI". A dark mode that is the light mode with the colours flipped. Emoji used as bullet points.

The tenth team's product is a canteen app that looks like the canteen: steel-tray greys, a single hot orange for the one action that matters, dish names at a size you can read while walking, a hand-set price board on the home screen, and prices in a font large enough to see from the queue. It is not more beautiful than the other nine. It is *theirs*, and three mentors remember it a week later.

The nine teams did not have bad taste. They asked for "a modern, clean design" and received the statistical average of everything the model has ever seen.

### The idea

**Generic output is what you get when your brief contains no information. Style comes from constraints, references and bans — never from adjectives.**

"Modern", "clean", "sleek", "professional" and "beautiful" are all words that describe the centre of the distribution. They carry no instruction. Every word of real direction you add narrows the output away from the average, and narrowing is the whole job.

### The analogy

Walk past five new cafés near any Indian campus and they are interchangeable: chalkboard menu, exposed brick, Edison bulbs, a wall of succulents, a neon sign saying something in cursive. Each owner chose that look because it signals "nice café". The signal has been copied so thoroughly that it now signals nothing except "we copied".

The old canteen next door is unmistakable, and nobody designed it. Steel plates because they survive ten thousand washes. A tiled counter because it wipes down. Prices in chalk because they change. Fluorescent light because it is cheap and bright. Every visual decision came from a real constraint, and the result is a look you could identify from a single photograph.

**Where the analogy breaks:** the canteen's look accumulated over thirty years of use, and you have six weeks. You cannot wait for your constraints to express themselves. You have to go and find them deliberately — in your users, your region, your content and your actual physical conditions — and then write them into the brief as instructions. The specificity that a canteen earns by existing, you have to import on purpose.

### Picture it

```
  VAGUE BRIEF                      CONSTRAINED BRIEF
  "modern, clean,                  "steel grey surfaces, ONE hot
   professional"                     orange accent on the single
        │                            primary action, no gradients,
        ▼                            system font only, price is the
  ┌────────────┐                     biggest text on every card"
  │ the average │                           │
  │ of all UI   │                           ▼
  │ ever seen   │                    ┌──────────────┐
  └────────────┘                     │ one specific │
        │                            │ product      │
        ▼                            └──────────────┘
  purple gradient, glass                    │
  cards, centred hero,                      ▼
  3 icon columns                     nobody else's
```

A brief with no constraints collapses to the centre of the training distribution; every specific constraint you add pushes the output away from that centre toward something only your product would look like.

### The tells, by name

You cannot avoid what you cannot name. These are the current signatures, and they shift every few months — so treat the list as a method, not scripture.

| Tell | What it looks like | What to do instead |
|---|---|---|
| Default gradient | Purple-to-blue, violet-to-pink | One flat accent colour |
| Glass everything | Frosted blur panels over blur | Flat surfaces, real contrast |
| The three-column feature row | Icon, two-word title, one line | Content in the order users need |
| Centred everything | Hero text, cards, footer, all centred | Left-aligned text; centre sparingly |
| Uniform radius | Every corner at the same generous curve | Pick one radius, and vary it by role |
| Decorative line icons | A thin icon beside every heading | Icons only where they aid recognition |
| Emoji as structure | 🚀 ✨ as bullet points and headings | Words |
| Stock abstract art | Blobs, 3D spheres, floating cubes | Your own photos, or nothing |
| "Transform your X" copy | Verb-your-noun-with-AI headlines | What your product literally does |
| Flipped dark mode | Light values inverted | A second deliberate colour set |

One honest caveat: some of these are also just conventions, and conventions exist because they work. The problem is not that a card has rounded corners. The problem is a screen where *every* decision is the default one, because that is a screen where nothing was decided.

### Four levers that actually work

**1. References, specified in three respects.** "Make it like Zomato" is useless — the model will average its idea of a food app. Useful direction names the artefact and exactly what to take from it: "Take the price-forward layout of a railway station departure board: large numerals, left-aligned labels, high contrast, no imagery. Take nothing else." Three respects, and a "take nothing else".

Your best references are rarely other apps. A bus ticket. A Maharashtra State Transport timetable. A hand-painted shop board in Nashik. A ration card. A cricket scoreboard. These have strong visual logic and are nowhere near the centre of the training distribution.

**2. Constraints that force choices.** Each of these removes a default: exactly one accent colour; no gradients anywhere; one font family; a maximum of four icons in the entire product, and name them; no shadows, use borders; the price must be the largest text on any card; no decorative imagery at all. A brief with six constraints like these cannot produce the average screen, because the average screen violates all six.

**3. Content-derived aesthetics.** Let the real content dictate the form. If your users are reading dish names while walking, type size is driven by that, not by taste. If your product is used in sunlight, you are pushed toward flat high-contrast surfaces and away from soft gradients — a real constraint producing a real look. If prices are the thing people came for, prices get the hierarchy. This is the most durable lever, because it cannot be copied by a team with different content.

**4. A written ban list.** The strongest single sentence in a style brief is a list of things never to do. "Never: gradients, glassmorphism, centred body text, emoji, stock illustration, more than one accent colour, more than two font sizes above body." Bans work because they are checkable, and the AI cannot drift back to the default without visibly violating one.

Put together, this becomes a compact style direction: three adjectives you can defend, three references with what to take from each, and six bans. That fits in fifteen lines and goes at the top of every build session, alongside the `DESIGN.md` from chapter 7.

### Where it lives

This chapter sets the *values* that chapter 7 built the structure for. It depends entirely on chapters 5 and 6 — a distinctive screen with broken hierarchy or random spacing just looks eccentric. Chapter 9 constrains it: a bold look that fails contrast is not a style, it is a defect. M15 extends the same direction to your landing page, deck and social presence, where consistency is what makes a three-person team look like a company. And at M11's checkpoint, "why does your product look like this?" is a question with a right answer: because of these constraints, from these users, in these conditions.

### Words to own

| Term | Plain meaning | Example |
|---|---|---|
| Generic output | The statistical average of the training data | Purple gradient hero |
| Style direction | Written instructions that constrain the look | Three references, six bans |
| Reference artefact | A specific thing to borrow a specific quality from | A station departure board |
| Ban list | Things the design must never do | "No gradients, no emoji" |
| Accent colour | The one colour reserved for the main action | One hot orange |
| Visual convention | A default that exists because it works | Rounded corners on buttons |
| Swap test | Would this screen suit a different startup? | If yes, it is generic |
| Distinctive | Memorable and still usable | Price-board home screen |
| Novelty cost | What users pay to learn your invention | A custom back gesture |
| Content-derived | A look driven by the real content | Prices largest on the card |

### Tradeoffs and gotchas

**Distinctive is not the same as unusual.** Inventing your own navigation pattern is unusual and expensive — every user pays to learn it, and chapter 3's consistency heuristic is not suspended because you had an idea. Be conventional in *interaction* and specific in *surface*. Buttons where buttons go; your own typography, colour, density and hierarchy.

**Do not fight the platform for looks.** On Android, the system back must work. Tap targets stay thumb-sized. M08 and M12 are not negotiable in service of an aesthetic.

**Taste is built from references, not from opinions.** If you cannot say what you want, collect fifteen screenshots of things you find good and write one sentence under each saying *what* is good about it. That becomes the only real vocabulary you have, and it is also exactly what makes your briefs specific.

**Constraints must be few and real.** Twenty constraints produce paralysis and a muddle. Six that you can defend produce a look.

**A distinctive look can still be bad.** Orange on grey is distinctive; orange on grey at 2.1:1 is unreadable. Run chapter 5's checks on anything this chapter produces, every time.

**The tells list expires.** The specific signatures shift as models and fashions move. What does not expire is the method: name the current defaults, then ban them.

### Verify it

1. **The swap test.** Cover your logo and product name. Would this screen work, unchanged, for a laundry startup, a tuition app or a gym? If yes, nothing in it came from your product. This is the fastest and most brutal check in the module.
2. **Run the tells checklist.** Go down the table above and mark each tell present or absent. Three or more present means the brief lost.
3. **Count the accent colours.** More than one colour competing for "this is the action" means the constraint was ignored.
4. **Ask it to justify three decisions.** "Why this type size, this accent colour and this density?" The answer should cite your users and conditions. If it cites "modern design trends", you got the average and a confident sentence about it.
5. **Line it up against three real competitors.** Screenshot yours beside three products in your space, at the same size. If a stranger cannot pick yours out, go back to the references.
6. **Re-run the contrast and sunlight checks from chapter 5.** Every time. A new look is the most common way accessibility silently regresses.

Red flags: a gradient you did not ask for; any emoji in the interface; "Transform your…" copy; a hero section on an app screen; stock abstract illustration; identical corner radius on every single element; and AI describing its own output as "modern and clean", which is the exact vocabulary that produced the problem.

## Walkthrough
1. Screenshot one screen of your product and one screen each from three other teams in your cohort. Put all four side by side at the same size.
2. Run the swap test on your own screen with the logo covered. Write down honestly whether it could belong to any of the other three products.
3. Go down the tells table and mark each one present or absent in your screen. Count them.
4. Collect fifteen reference images that are not apps: tickets, boards, signage, packaging, timetables, scoreboards, anything from your own city. Photograph them yourself where you can.
5. Pick the three strongest and write one sentence under each naming exactly what you are taking — and add "take nothing else" to each.
6. Write three adjectives you can defend with a reason from your users or conditions, not from taste. "Legible in sunlight" beats "modern".
7. Write six bans. Each must be checkable by looking at a screenshot. "No gradients" is checkable; "not too busy" is not.
8. Combine the adjectives, references and bans into a fifteen-line style direction and paste it with your `DESIGN.md` into a fresh session. Ask for one screen, restyled, with no layout or copy changes.
9. Re-run the tells checklist and the swap test on the result. Then ask AI to justify three of its decisions and check whether the answers cite your users.
10. Finally re-run chapter 5's contrast and sunlight checks. Record any accessibility ground you lost while gaining a look, and fix that before anything else.

## Talk to AI
```prompt
CONTEXT:      I am a student founder directing AI to build my product's
              interface. I do not write code. My product: [one sentence].
              My users: [who they are, where they are, what phone, what
              light, what network, what language]. The content that
              matters most on screen: [e.g. dish name and price].

              My three defensible adjectives, each with a reason from my
              users rather than my taste:
              1. [adjective] because [user or condition]
              2. [adjective] because [user or condition]
              3. [adjective] because [user or condition]

              My three reference artefacts and exactly what to take from
              each - and nothing else:
              1. [artefact]: take [three specific qualities]. Take
                 nothing else.
              2. [artefact]: take [three specific qualities]. Take
                 nothing else.
              3. [artefact]: take [three specific qualities]. Take
                 nothing else.

              My existing design tokens: [paste DESIGN.md]

GOAL:         A visual style for ONE screen of my product that could not
              be mistaken for any other team's product.

REQUIREMENTS: 1. Exactly ONE accent colour, reserved for the single
                 primary action
              2. Type, colour, density and hierarchy driven by my content
                 and my users' conditions, with a stated reason for each
              3. For every choice you make, one sentence citing which of
                 my adjectives, references or user conditions caused it
              4. A statement of what you took from each reference and
                 what you deliberately did not take
              5. The computed contrast ratio, as a number, for every text
                 and background pairing you propose

BANS - never produce any of these:
              - Gradients of any kind
              - Frosted glass, blur or translucency effects
              - Emoji anywhere in the interface
              - Stock or abstract illustration: blobs, spheres, cubes
              - Centred body text
              - More than one accent colour
              - More than two text sizes above body size
              - A hero section on an app screen
              - Identical corner radius on every element
              - Copy of the form "Transform your X with AI"

CONSTRAINTS:  Must stay legible in direct sunlight on a cheap LCD phone.
              Body text 16px minimum, 4.5:1 contrast minimum. System
              fonts only - no downloaded font files. Interaction patterns
              stay conventional: the Android system back works, tap
              targets are 48dp, buttons look like buttons. Do not change
              my layout order or my copy.

NON-GOALS:    Do not invent new navigation or gestures. Do not add
              components. Do not redesign other screens. Do not use the
              words "modern", "clean", "sleek" or "professional" anywhere
              in your reasoning - they describe the average, which is
              what I am trying to avoid.

ACCEPTANCE:   - With my logo and product name removed, this screen would
                look wrong on a laundry app, a tuition app or a gym app,
                and you can say why
              - Zero bans violated
              - Exactly one accent colour, used only on the primary
                action
              - Every contrast figure is a real ratio, verifiable in a
                checker
              - A final section titled "Defaults I avoided", naming each
                default you would normally have reached for and what you
                did instead

ASK:          First, before designing anything, tell me what you would
              have produced from a vague brief that said only "modern and
              clean" for this product - describe it in detail. Then tell
              me which of my constraints most changes that outcome. Only
              then produce the style.
```

## Stuck? Ask AI
```prompt
Here is a screenshot description of one screen of my product: [describe every element, colour, size and spacing]. Cover my branding mentally and tell me honestly: could this screen belong to a laundry service, a tuition app or a gym? For every element that is generic, say which default it came from. Do not redesign anything — just diagnose.
```

```prompt
I want to give you a reference without you copying the whole thing. Here is my reference: [name an object — a bus ticket, a station board, a shop sign]. List the six visual qualities it actually has, then ask me which three I want to borrow. Do not design anything until I answer.
```

```prompt
Here are three adjectives I want my product to feel like: [paste them]. For each one, tell me whether it is defensible from a user condition or whether it is just taste, and rewrite the taste ones as a constraint I could check on a screenshot. Then tell me which of the three is closest to the generic default and should be replaced.
```

## Quiz

### Q1 · L3
Why does "make it modern and clean" reliably produce a generic result?
- (a) The model does not understand design adjectives
- (b) Those words describe the centre of the training distribution, so they carry no instruction ✅
- (c) The prompt is too short to be processed properly
- (d) Modern and clean are contradictory goals
> Adjectives like these add no information that narrows the output. Specificity — references, constraints and bans — is what pushes a design away from the average.

### Q2 · L4
Which is a usable reference instruction?
- (a) Make it look like Zomato
- (b) Make it feel premium and trustworthy
- (c) Take the price-forward layout of a station departure board: large numerals, left-aligned labels, high contrast, no imagery. Take nothing else. ✅
- (d) Use a modern food-delivery aesthetic
> A reference works when it names the artefact, names exactly which qualities to borrow, and bans everything else. "Like Zomato" just invites the model's average food app.

### Q3 · L3
You cover your logo and your screen would work unchanged for a laundry startup. What does this tell you?
- (a) Your design is versatile, which is a strength
- (b) Nothing in the screen came from your own product, users or content ✅
- (c) Your colour palette needs to be more saturated
- (d) You should add your logo more prominently
> The swap test measures whether any constraint of yours actually shaped the output. A screen that fits anything was shaped by nothing.

### Q4 · L4
Why is a ban list stronger than a list of positive style goals?
- (a) Negative instructions are processed more reliably by models
- (b) Bans are checkable on a screenshot, so drift back to the default becomes visible ✅
- (c) Bans produce shorter briefs
- (d) Positive goals cannot be written specifically
> "No gradients" can be verified by looking. "Feels premium" cannot. Checkability is what lets you hold the line across many sessions.

### Q5 · L5
A team replaces their generic look with a bold orange-on-grey scheme. It is distinctive and memorable. What must they check before keeping it, and why?
- (a) Whether competitors already use orange, to avoid confusion
- (b) Contrast ratios and sunlight legibility — a new look is the most common way accessibility silently regresses ✅
- (c) Whether orange tests well with their persona
- (d) Whether the orange matches their logo exactly
> Distinctive and unreadable is a defect, not a style. Restyling touches every colour pair at once, so chapter 5's checks have to be re-run every time the surface changes.

## Go deeper
- [Refactoring UI: visual design tips for developers](https://www.refactoringui.com/)
- [WebAIM: Contrast checker](https://webaim.org/resources/contrastchecker/)
