---
id: m02-07-tokens-dark-mode
module: m02-the-web
slug: tokens-dark-mode-utility-css
title: "Design Tokens, CSS Variables, Dark Mode and Utility-First CSS"
order: 7
duration_min: 15
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

Six weeks in, the team decides their brand green is too dark for Nashik sunlight: "change `#0B6E4F` to `#15925F` everywhere." The agent reports success. The header is right; the "Add to order" button is still the old green, because it was written as `rgb(11, 110, 79)` in another file. Two badges are a fourth green nobody approved, and the dark theme — built by inverting the light colours — now shows white ₹ prices on pale green.

One decision, forty-seven places, and three inconsistencies a user will notice before the team does.

### The idea

**A token is a named decision. Name it once, refer to the name everywhere, and the change becomes a one-line edit instead of a search-and-replace.**

In CSS the mechanism is a custom property: define `--color-brand: #15925F` once at the top of the document and write `color: var(--color-brand)` everywhere else. Change the one line and everything that referred to the name follows, including the things you forgot exist.

### The analogy

Early on the canteen used whatever bowl was to hand, so "half plate of sambar" depended on who served you. Then it bought two hundred identical katoris: one katori is one serving, and "increase the portion by 20%" becomes one decision about the standard. Better still, they named the servings, not the vessels, so a new supplier's bowls change nothing.

**Where the analogy breaks:** a steel katori is physical; you cannot serve a different amount. A CSS variable is only a name, and nothing stops your agent writing a raw hex next to it. Tokens are a convention held up by review, not by the browser — which is why Verify it is a search for what bypassed the system.

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

The middle layer earns its keep: with one layer, `--green` eventually holds a blue. With two, components express intent — a sold-out tag asks for `--color-danger`, not red, so the day sold-out becomes amber it is one line.

### Where it lives

Tokens live in the client box from M00, in one file loaded before everything else, but conceptually they belong to design: M03 decides what they *should be*, and this chapter is only the mechanism that makes those decisions hold.

They are also the shared vocabulary between you and your agent. "Use `--space-4` for that gap" is verifiable; "make the gap a bit bigger" is not. In M10 the token list goes into a rules file so you stop restating it every session.

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

### Tradeoffs and gotchas

**Dark mode is not inversion.** Flipping every colour renders, and is worse than no dark mode:

| Inverting breaks | What goes wrong |
|---|---|
| Contrast | 7:1 dark-on-light becomes 2.9:1 light-on-dark, and the ₹ price is a suggestion |
| Pure black and white | White on pure black smears (halation); use dark grey and off-white |
| Shadows | A dark shadow on a dark background is invisible, so cards lose separation |
| Images and logos | An inverted thali photo is a horror; a white-background logo glows |

The correct model is two designed, measured sets of semantic values: same role names, different numbers.

**Utility-first CSS trades stylesheet discipline for markup noise.** `flex gap-4 p-4 rounded-lg` in the markup means never inventing a class name or fighting the cascade, in exchange for dense markup where two cards drift apart. A real trade, not a free win.

**Tokens only work if nothing bypasses them,** and four layers of indirection is as bad as none. One raw hex makes "change it once" false; five files to reach a colour makes it unanswerable. Two layers is right.

**AI invents shades.** Asked for a lighter green it produces a new hex rather than the next step on your scale, and twenty requests later you own eleven greens. Say: "only use values from the token list; if none fits, propose a new token."

### Verify it

1. **Search for raw values.** `#`, `rgb(`, `hsl(` and `px` outside the token file; each hit is a decision that will not follow the next change.
2. **Change one token and check three screens.** Set `--color-brand` to hot pink: anything still green is a bypass.
3. **Measure the contrast, do not judge it.** The DevTools colour picker shows the ratio — body text, the ₹ price, text on the brand colour, in both themes. Body text needs 4.5:1.
4. **Toggle to dark mode and walk the whole app,** hunting for invisible shadows, lost contrast, white-background images and disabled states that look enabled.
5. **Count things.** Thirty-one colours including `green`, `green2` and `brandGreen` means the system failed; more than eight utility classes on one element means a component should have been extracted.

Red flags: literal hex codes in components · values not on your scale · a dark theme made by inversion or a CSS filter · no manual theme toggle, or one the media query overrides · tokens named for their value, like `--color-green-button` · a "theme" that leaves borders, shadows and focus rings on light-mode values.

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
