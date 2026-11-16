---
id: m02-07-tokens-dark-mode
module: m02-the-web
slug: tokens-dark-mode-utility-css
title: "Design Tokens, CSS Variables, Dark Mode and Utility-First CSS"
order: 7
duration_min: 25
is_free: false
mission: false
volatility: medium
last_reviewed: 2026-11-16
---

## Key learnings
- Name a decision once so that changing it once changes the whole product
- Tell primitive tokens from semantic tokens, and say why the second kind matters
- Explain why dark mode is not inversion, and what it breaks if you treat it as one
- Say what utility-first CSS buys you and what it costs you
- Verify that a theme change actually propagated, instead of trusting the screenshot

## Lesson

### The pain

Six weeks into the build, the team decides their brand green is too dark to read in Nashik sunlight and picks a lighter one.

They tell the agent: "change the brand green from `#0B6E4F` to `#15925F` everywhere." The agent reports success. The header is right. Three buttons are right. The "Add to order" button on the dish card is still the old green, because it was written as `rgb(11, 110, 79)` in a different file. Two badges are a fourth green nobody remembers approving. The focus ring is the old colour. And the dark-mode variant, which was built by taking the light colours and inverting them, now has white ₹ prices on a pale green card that nobody can read at all.

One decision, forty-seven places. The change took an afternoon and shipped three inconsistencies that a user will notice before the team does.

### The idea

**A token is a named decision. Name it once, refer to the name everywhere, and the change becomes a one-line edit instead of a search-and-replace.**

In CSS the mechanism is a custom property — a variable you define once and refer to by name. `--color-brand: #15925F` defined at the top of the document, and `color: var(--color-brand)` everywhere else. Change the one line, and everything that referred to the name follows, including things you have forgotten exist.

### The analogy

The canteen's standard katori.

Early on, every counter used whatever bowl was to hand, so a "half plate of sambar" was a different amount depending on who served you. Then the canteen bought two hundred identical katoris and wrote a rule: one katori is one serving. Now "increase the sambar portion by 20%" is a single decision about the standard, not a negotiation with eleven staff members.

Better still, they named the servings rather than the vessels: "student portion", "worker portion". A new supplier changes the actual bowls and the names still hold.

**Where the analogy breaks:** a steel katori is a physical object, so you physically cannot serve a different amount. A CSS variable is only a name, and nothing stops anyone — including your AI agent — from writing a raw hex code right next to it. Tokens are a convention held up by review and verification, not a constraint enforced by the browser. That is precisely why the "verify it" section of this chapter is about searching for the things that bypassed the system.

### Picture it

```
  PRIMITIVE tokens        SEMANTIC tokens          used in components
  (raw values)            (roles)
  ┌──────────────┐        ┌────────────────┐       ┌───────────────┐
  │ green-600    │───────▶│ color-brand    │──────▶│ primary button│
  │ #15925F      │   ┌───▶│ color-success  │──────▶│ "in stock" tag│
  ├──────────────┤   │    ├────────────────┤       ├───────────────┤
  │ red-600      │───┼───▶│ color-danger   │──────▶│ "sold out" tag│
  │ #C62828      │   │    ├────────────────┤       │ delete button │
  ├──────────────┤   │    │ color-text     │──────▶│ all body text │
  │ grey-900     │───┘    └────────────────┘       └───────────────┘
  └──────────────┘          swapped per theme
```

Raw values get names, those names get roles, and components only ever refer to roles — so switching a theme means repointing the roles, and no component changes at all.

### Why the middle layer earns its keep

A beginner's first attempt at tokens is usually one layer: `--green: #15925F`, used directly in components. That already solves the search-and-replace problem, and it is a real improvement.

It fails the first time the brand changes to blue, because now you either have a variable called `--green` holding a blue, or you are back to editing every component. It also fails at dark mode, because "the green" is a different green in dark mode while "the brand colour" is still conceptually the brand colour.

The two-layer version — primitives named by what they *are*, semantics named by what they are *for* — means components express intent. A "sold out" tag asks for `--color-danger`, not for red. The day somebody decides sold-out should be amber rather than red, that is one line, and nothing in any component was wrong.

### Where it lives

Tokens live in the client box from M00, usually in one file loaded before everything else. But conceptually they belong to design, not to code: M03's design-systems chapter is where you decide what the tokens *should be*, and this chapter is only the mechanism that makes those decisions hold.

They also become the shared vocabulary between you and your AI agent, which is the part that matters most for this programme. "Use `--space-4` for that gap" is a verifiable instruction. "Make the gap a bit bigger" is not. In M10 you will put your token list into a rules file so you stop restating it every session.

### Words to own

| Term | Plain meaning | Example |
|---|---|---|
| Design token | A named design decision | `--color-brand` |
| CSS custom property | The CSS feature that stores a named value | `--space-4: 16px` |
| `var()` | How you refer to a custom property | `gap: var(--space-4)` |
| Primitive token | A name for a raw value | `--green-600` |
| Semantic token | A name for a role | `--color-danger` |
| Scale | A fixed set of allowed steps | 4, 8, 12, 16, 24, 32 |
| Theme | One complete set of semantic values | Light, dark |
| `prefers-color-scheme` | A media query reading the OS setting | `(prefers-color-scheme: dark)` |
| Contrast ratio | A measured number for text legibility | 4.5:1 minimum for body text |
| Utility class | A tiny single-purpose class | `p-4`, `text-sm` |
| Class soup | Markup where the class list dwarfs the content | Twelve classes on one `<div>` |

### Dark mode is not inversion

Flipping every colour to its opposite produces something that technically renders and is genuinely worse than no dark mode at all. Four specific things break.

**Contrast inverts unpredictably.** A dark grey text on a light card becomes light grey text on a dark card — which sounds fine, until you measure it and find the ratio dropped from 7:1 to 2.9:1 and the ₹ price is now a suggestion.

**Pure black and pure white are both wrong.** A white-on-pure-black interface produces halation, where text appears to smear, especially for people with astigmatism. Real dark themes use a very dark grey and an off-white.

**Shadows stop working.** In light mode a shadow means "this is raised". On a dark background a dark shadow is invisible, so cards lose their separation and the layout flattens into a single sheet.

**Images and logos do not invert.** A photo of a thali inverted is a horror. A logo with a white background becomes a glowing rectangle. These need per-asset decisions.

The correct model is two sets of semantic values, each designed and each measured. Same role names, different numbers behind them.

### Tradeoffs and gotchas

**Utility-first CSS trades stylesheet discipline for markup noise.** Writing the styling as small classes directly in the markup — `flex gap-4 p-4 rounded-lg` — means you never invent a class name, never hunt for which file a rule lives in, and never fight the cascade from chapter 4. In exchange, your markup becomes dense and hard to skim, and it becomes trivially easy for two cards to drift apart because nothing forces them to share. It is a real and reasonable trade; it is not a free win, and an AI will happily produce eighteen utility classes on a single element.

**Over-abstraction is as bad as none.** Four layers of indirection to reach a colour means nobody can answer "what colour is this button" without opening five files. Two layers is almost always right.

**Tokens only work if nothing bypasses them.** One raw hex in one component is the whole value proposition broken, because now the "change it once" promise is false and you will not know until a user notices.

**AI invents shades.** Asked for a slightly lighter green, a model will frequently produce a brand-new hex code rather than reaching for the next step on your existing scale. Over twenty such requests you accumulate a palette of eleven greens. The instruction that prevents it: "only use values from the token list; if none fits, tell me and propose a new token rather than a literal value."

**Honour the OS setting, then let people override it.** `prefers-color-scheme` reads what the user already chose system-wide, which is the right default. But some users want a dark app and a light OS, so a manual toggle that overrides it is worth having — and it must beat the media query, which is a cascade problem from chapter 4.

### Verify it

Tokens are a convention, so verification is a search for what escaped it.

1. **Search the codebase for raw colour values.** Look for `#`, `rgb(` and `hsl(` outside your token definition file. Every hit is a decision that will not follow the next change. Zero is the target, and on a young project zero is achievable.
2. **Search for raw spacing values.** `px` outside the token file, in padding, margin and gap. A handful is survivable; dozens means the scale is not being used.
3. **Change one token and look at three screens.** Set `--color-brand` to a violently obvious colour like hot pink and reload. Everything that should be brand-coloured turns pink. Anything that stays green is a bypass, and this finds them faster than reading any file.
4. **Measure the contrast, do not judge it.** Use the DevTools colour picker, which shows the contrast ratio for any text colour against its background and whether it passes. Do it for body text, for the ₹ price, for placeholder text and for text on your brand colour, in both themes. Body text needs 4.5:1.
5. **Toggle to dark mode and walk the whole app.** Not one screen. Look specifically for: invisible shadows, text that has lost contrast, images with white backgrounds, disabled states that now look enabled, and your own logo.
6. **Count the tokens.** If your colour list has thirty-one entries including four greens called `green`, `green2`, `brandGreen` and `primary`, the system has already failed and it is cheaper to consolidate now than in M11.
7. **Count the classes on one element.** In utility-first markup, more than about eight on a single element usually means a component should have been extracted.

Red flags in AI output: literal hex codes in components; new colour values that are not on your scale; a dark theme produced by inversion or by a single CSS filter; `prefers-color-scheme` handled but no manual toggle, or a toggle that the media query overrides; tokens named after their value rather than their role, like `--color-green-button`; and a "theme" that only covers backgrounds and text, leaving borders, shadows and focus rings on their light-mode values.

## Walkthrough
1. Open a product you admire that offers both themes — any major app will do — and inspect its body text in DevTools. In the Styles panel, look at whether `color` resolves to a literal value or to a `var(--something)`.
2. If it uses a variable, click through to where it is defined. Note whether the name describes a value (`grey-900`) or a role (`text-primary`). Write down which layer you are looking at.
3. In the Elements panel, select the `:root` or `html` element and read its list of custom properties in the Styles panel. Count the colour tokens. Count how many are obviously duplicates of each other.
4. Use the DevTools rendering options to emulate `prefers-color-scheme: dark` and watch the same page switch. Then re-read the same custom properties and note which values changed and which did not.
5. In dark mode, walk through at least four different screens or states of that product. Specifically hunt for: a shadow that has disappeared, text whose contrast has dropped, and an image or logo that looks wrong.
6. Click the small colour swatch next to any text colour in the Styles panel. DevTools shows the contrast ratio against the background and whether it passes. Record the ratio for body text, for any secondary grey text, and for text sitting on the brand colour — in both themes.
7. Now edit one custom property live in the Styles panel: set the main brand or accent colour to hot pink. Reload is not needed. Look at the whole page and list everything that did *not* turn pink. Those are the bypasses.
8. Do the same for a spacing token if the product has one, and note whether the layout moved consistently or in patches.
9. Pick one component on the page and count the classes on its outermost element. Decide whether this product uses utility-first CSS or named component classes, and write one sentence on how you can tell.
10. Write a short report: how many colour tokens this product has, whether it uses semantic naming, the lowest contrast ratio you measured and where, and the one thing its dark theme gets wrong.
11. Finally, draft the starting token list for your own product: no more than eight colour roles and one spacing scale of six steps. Name every one by role, not by value.

## Talk to AI
```prompt
CONTEXT:      I direct AI rather than writing CSS. My product is a canteen
              ordering app for students in Nashik. 85% of use is budget
              Android phones at 360px, often outdoors in bright sunlight.
              Prices are shown in rupees and are the most important text on
              the page. States I need to express: normal, success, warning,
              danger/sold-out, disabled, and focus. Three teammates and an
              AI agent all touch the styling. Nothing is tokenised yet and
              there are already several hardcoded colours in the code.

GOAL:         A small, two-layer design-token system plus a real dark theme,
              such that changing any one decision is a one-line edit and I
              can verify that it propagated.

REQUIREMENTS: 1. A primitive layer named by value, and a semantic layer
                 named by role. No component ever refers to a primitive
              2. At most eight semantic colour roles, each with a
                 one-sentence description of when to use it
              3. One spacing scale of exactly six steps, and a rule for
                 what to do when a designer wants a value between steps
              4. A dark theme designed independently, not inverted, with
                 the measured contrast ratio for every text role stated
              5. Explicit token values for borders, shadows and focus
                 rings in both themes, not just background and text
              6. Respect the OS setting by default, plus a manual override
                 that reliably beats it

CONSTRAINTS:  Plain CSS custom properties, no framework and no build step.
              No literal hex, rgb or hsl values anywhere outside the token
              file. No pure black and no pure white as a theme background.
              Body text and price text must measure at least 4.5:1 contrast
              in both themes, and you must state the ratios. No dark theme
              produced by a filter or by inverting the light theme.

NON-GOALS:    Do not pick my brand colour for me — assume #15925F and tell
              me if it fails contrast. Do not introduce a utility-first
              framework. Do not restyle existing components yet. Do not add
              animation or transitions.

ACCEPTANCE:   - Searching the project for "#" outside the token file
                returns nothing
              - Setting --color-brand to hot pink turns every brand-
                coloured element pink, with no exceptions
              - Toggling to dark mode changes borders, shadows and focus
                rings, not only background and text
              - Every text role measures at least 4.5:1 in both themes, and
                the ratios are written down
              - A manual dark toggle overrides an OS setting of light, and
                vice versa
              - There are no more than eight semantic colour roles

ASK:          Before writing anything, propose the semantic role list as a
              table — role name, when to use it, light value, dark value,
              measured contrast against its intended background — and flag
              any role you think I do not need or any I am missing. Tell me
              which of my requirements are in tension with each other. Wait
              for my approval before producing any CSS.
```

## Stuck? Ask AI
```prompt
Explain the difference between a primitive token and a semantic token using an example that is not about colour. Then give me a list of ten token names and ask me to sort them into primitives, good semantic tokens and badly named tokens, and to say what is wrong with each bad one. Do not tell me the answers first.
```

```prompt
I built a dark theme by inverting my light theme and something feels wrong but I cannot say what. Explain the four specific things that break when you invert a theme rather than designing one, with a concrete example of each. Then give me a checklist of what to look at on every screen when reviewing a dark theme.
```

```prompt
Explain what utility-first CSS actually is, what problem it solves that named component classes do not, and what it costs me in return. Be honest about the downsides. Then tell me which of the two I should choose for a three-person team where an AI agent writes most of the markup, and why.
```

## Quiz

### Q1 · L2
What is a design token?
- (a) A login credential used by a design tool
- (b) A named design decision that everything else refers to by name ✅
- (c) A unit of measurement specific to CSS
- (d) A licence that permits use of a font
> Naming the decision once means changing it once. `--color-brand` is a token; `#15925F` written in forty files is the problem tokens exist to remove.

### Q2 · L3
Why is a two-layer token system better than a single layer of names like `--green`?
- (a) It loads faster in the browser
- (b) Components express intent, so a role can be repointed to a different value without touching any component ✅
- (c) It allows more colours to be defined
- (d) Single-layer tokens are not supported in dark mode
> A "sold out" tag asking for `--color-danger` stays correct when danger becomes amber. A tag asking for `--green` is wrong the moment the brand stops being green.

### Q3 · L4
Your dark theme was made by inverting the light theme. Which problem is this guaranteed to have produced?
- (a) The CSS file is twice as large
- (b) Shadows become invisible, so cards lose their separation ✅
- (c) Media queries stop firing
- (d) Font sizes shrink
> A dark shadow on a dark background cannot be seen, so the raised-card effect disappears and the layout flattens. Contrast drift, pure-black halation and un-invertible images are the other three.

### Q4 · L4
You want to prove that a token change actually propagated everywhere. What is the fastest reliable check?
- (a) Read the AI's summary of which files it changed
- (b) Set the token to an obviously wrong colour like hot pink and list everything that did not change ✅
- (c) Reload the page with the cache disabled
- (d) Count the number of `var()` references in the stylesheet
> A deliberately absurd value makes every bypass visible at a glance, across every screen, in seconds. Reading a summary tells you what the AI believes it did.

### Q5 · L5
An AI is asked for a slightly lighter brand green and returns a brand-new hex code. What is the right correction?
- (a) Accept it — one more colour is harmless
- (b) Ask it to apply the new colour with `!important` so it wins
- (c) Tell it to use only values from the token list, and to propose a new token if none fits ✅
- (d) Ask it to invert the existing green instead
> Each invented value is a decision outside the system. Over twenty requests you get eleven greens and the "change it once" promise quietly becomes false — which you discover from a user, not from a diff.

## Go deeper
- [MDN: Using CSS custom properties](https://developer.mozilla.org/en-US/docs/Web/CSS/CSS_cascading_variables/Using_CSS_custom_properties)
- [web.dev: prefers-color-scheme and building for dark mode](https://web.dev/articles/prefers-color-scheme)
