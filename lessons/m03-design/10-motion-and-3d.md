---
id: m03-10-motion-and-3d
module: m03-design
slug: motion-and-3d
title: "Motion, Micro-Interactions and 3D on the Web"
order: 10
duration_min: 15
is_free: false
mission: false
volatility: medium
last_reviewed: 2026-11-16
---

## Key learnings
- Use motion to explain what changed, never to decorate
- Pick durations and easing that feel instant rather than impressive
- Respect the reduced-motion setting, and know who it protects
- Judge whether 3D earns its download on a budget phone and slow 4G
- Catch the motion failures AI adds by default and remove them

## Lesson

### The pain

A team adds animation the week before demo day: cards fade and slide on scroll, pages cross-fade for 600 ms, and the hero holds a rotating 3D plate of food weighing 2.4 MB. On a budget Android on hostel 4G the page needs eleven seconds to become readable, the animations stutter at fifteen frames per second — which reads as broken, not animated — and a student with vestibular sensitivity closes it with a headache. Honestly measured: 2.9 MB added, nine seconds of load, nothing clearer.

### The idea

**Motion earns its place only when it explains a change: where something came from, where it went, or that the system is working.**

Everything else is cost: milliseconds, battery, bytes, and the irritation of waiting for a transition you have seen four hundred times.

### The analogy

At the canteen counter, useful motion is everywhere and none of it decorative. The server slides your plate forward, so you know it is yours. The token number flips, so you see which number changed without hunting. Now imagine the plate spinning twice first: funny once, then you are late.

**Where the analogy breaks:** counter motion is physical, so it is free. Yours costs frames the phone may not render, bytes over a metered connection, and battery at 14%.

### Picture it

```
            DOES THIS MOTION EARN ITS PLACE?
┌──────────────────────────────────────────────────────────┐
│ Does it explain a CHANGE?                                │
│   yes -> where it came from / went to / that it's working │
│   no  -> delete it                                       │
├──────────────────────────────────────────────────────────┤
│ How long?                                                │
│   feedback on tap .... 100ms      enter / exit .... 200ms │
│   page transition .... 300ms      anything more .. cut it │
├──────────────────────────────────────────────────────────┤
│ What does it cost?                                       │
│   extra bytes? frames dropped on a cheap phone?          │
│   battery? a reason for someone to feel unwell?          │
├──────────────────────────────────────────────────────────┤
│ Honoured prefers-reduced-motion?   no -> not shippable   │
└──────────────────────────────────────────────────────────┘
```

Four questions in order: whether the motion explains a change, how long it is allowed to take, what it costs on a real device, and whether it respects the user's own setting — and a no at the last question is disqualifying regardless of the first three.

The working numbers are much shorter than most people expect.

| Purpose | Duration | Easing |
|---|---|---|
| Tap feedback | 50–100 ms | Linear or ease-out |
| Something appearing | 150–250 ms | Ease-out — fast in, settle |
| Something leaving | 100–200 ms | Ease-in — gets out of the way |
| Page or screen change | 200–300 ms | Ease-in-out |
| Loading indicator | Loops, 1–1.5 s | Linear |
| Anything over 400 ms | Almost never | — |

Ease-out for arrivals, ease-in for exits, linear for loaders. Two rules on top: **animate only transform and opacity**, because the GPU handles those while width, height and margin force a layout recalculation every frame — the fifteen-fps stutter — and **exits are faster than entrances**.

Few animations genuinely improve a product: the pressed state; a progress indicator inside a submitting button, which prevents double orders; a skeleton showing the *shape* of what is loading; an item moving into the cart; a toast from a fixed position. Past eight, the extras are decoration.

Reduced motion is not optional: people turn it on for vestibular disorders, where parallax causes real nausea. Honouring it switches off large movement and auto-play while keeping essential feedback — "no animation" is not "no feedback". This is WCAG 2.3.3, and AI never adds it unless asked.

3D is almost always wrong here. The test is **does the user need to understand a shape?** Yes for a floor plan; no for a rotating logo or a plate of food, where a photograph says more in 80 KB than a model in 2 MB. If it is genuinely yes, the library loads only after the page is readable and only in view, with a static fallback and a non-visual equivalent. Keep page weight under 500 KB on slow 4G.

### Where it lives

Motion is the last layer, after chapters 5 to 8 are settled.

- **Chapter 6** — the eight states are what most motion attaches to
- **Chapter 9** — its principles constrain motion absolutely
- **M02** — rendering cost and Core Web Vitals
- **M07** — whether a transition makes people abandon a flow
- **M08** — load measured on real devices

### Words to own

| Term | Plain meaning | Example |
|---|---|---|
| Micro-interaction | A small animation tied to one action | Button press feedback |
| Duration | How long the motion takes | 200 ms |
| Easing | How speed changes during the motion | Ease-out for arrivals |
| Transform | Moving, scaling or rotating — cheap to animate | Slide and scale |
| Layout thrash | Animating size, forcing recalculation every frame | Animating height |
| Skeleton | A grey outline of content while it loads | Menu rows as grey bars |
| Reduced motion | The user's system setting to limit animation | WCAG 2.3.3 |
| Parallax | Layers moving at different speeds on scroll | A common nausea trigger |
| Frame rate | Frames drawn per second; 60 is smooth | 15 fps reads as broken |
| WebGL | The browser's 3D rendering interface | What 3D libraries use |
| Progressive loading | Showing usable content before extras arrive | Menu first, model later |

### Tradeoffs and gotchas

**Scroll-triggered animation is the worst value in common use.** It delays content people are reading, stutters on cheap phones, and replays on every scroll back.

**Motion cannot fix slowness; it advertises it.** A 400 ms transition over a 2-second load makes 2.4 seconds.

**A spinner with no content shape is weaker than a skeleton.**

**Everything feels fine on your laptop.** Test on the cheapest Android your team owns, throttled.

**The tooling moves fast; the numbers do not.** Durations, easing, transform-and-opacity and reduced motion have not changed in a decade.

### Verify it

1. **Ask "what change does this explain?" of every animation.** No one-sentence answer means delete it.
2. **Time them.** Over 400 ms and not a loader is too slow. A duration list is checkable; "subtle animations" is not.
3. **Test on the cheapest phone you have, throttled.** Motion that delays reading is a defect.
4. **Turn reduced motion on and reload.** No change means it was never implemented.
5. **Ask "list every animated property".** Anything but transform and opacity needs a reason.
6. **Measure page weight before and after.** Over 100 KB needs a decision.

Red flags: animation nobody asked for; staggered scroll reveals on content people need to read; any duration over 400 ms; width or margin animated; no reduced-motion handling; 3D above the fold; parallax on mobile; AI calling motion "delightful" instead of listing durations.

## Walkthrough
1. List every animation currently in your product. Include ones you did not ask for — AI adds them by default, so look rather than remember.
2. For each one, write the change it explains in a single sentence. Mark the ones where you cannot.
3. Delete everything you marked. Reload and check whether anything is harder to understand. Usually nothing is.
4. For the survivors, ask AI for a table of each animation with its duration in milliseconds, its easing and the properties it animates.
5. Flag every duration over 400 ms and every animated property that is not transform or opacity. These are your two fix lists.
6. Install your product on the cheapest Android phone your team owns, throttle the network to slow 4G, and reload. Time how long until you can read the main content.
7. Watch for stutter during scrolling and transitions. Anything visibly jerky is worse than no animation at all, and should be removed rather than tuned.
8. Turn on reduced motion in that phone's accessibility settings and reload. List what changed and what did not. Confirm that pressed states and progress indicators survived.
9. If you have any 3D or heavy visual element, measure page weight with and without it, and write down the byte cost and the seconds cost.
10. Decide, in writing, which animations stay. For each, record the change it explains and its duration. That list is your motion specification from now on.

## Talk to AI
```prompt
CONTEXT:      I am a student founder directing AI to build my product. I
              do not write code; I verify output. My users are students
              and canteen staff in India on budget Android phones,
              typically 3-4 years old, on slow 4G, often at low battery,
              and usually in a hurry. Some have vestibular sensitivity
              and use the reduced-motion setting.

              Every animation currently in my product, and what I
              believe each one does: [list them]
              The states my main interactive elements have: [paste from
              your component states work]
              Current total page weight and time to first readable
              content on a cheap phone: [paste your measurements]

GOAL:         A motion specification that keeps only the animations that
              explain a change, with exact durations, and that fully
              honours reduced motion.

REQUIREMENTS: 1. For EACH animation I listed: the change it explains in
                 one sentence, or a recommendation to delete it
              2. A table of the survivors: trigger, duration in
                 milliseconds, easing, and the exact properties animated
              3. Only transform and opacity animated - flag anything
                 that cannot be done that way and say why
              4. Exits faster than entrances, stated as numbers
              5. A reduced-motion specification: exactly what stops,
                 exactly what continues, and why each feedback cue must
                 survive
              6. A loading treatment that shows the SHAPE of the content
                 arriving rather than a centred spinner
              7. An estimated byte cost for anything you add

CONSTRAINTS:  No animation over 400ms except looping loaders. No
              scroll-triggered reveals on content the user came to read.
              No parallax. No auto-playing video. No 3D. No animation
              library unless you can justify its download against the
              change it explains. Total added weight under 50KB. Must hold
              a readable frame rate on a 4-year-old Android. Do not change
              my layout, my copy or my colours.

NON-GOALS:    Do not add animations I did not list unless they prevent a
              real error, and if you do, say which error. Do not redesign
              any screen. Do not propose "delightful" or "premium"
              motion - every item must map to a change it explains.

ACCEPTANCE:   - Every surviving animation has a one-sentence change it
                explains, and I could disagree with it
              - Every duration is a number in milliseconds, not a word
                like "quick" or "subtle"
              - No animated property other than transform and opacity
                without a stated reason
              - The reduced-motion spec keeps tap feedback and progress
                indicators while removing all large movement
              - A final section titled "What I would cut first if the
                phone is struggling", ranked

ASK:          First tell me which of my listed animations you would
              delete outright and why, before proposing anything new.
              Then write the specification for what remains.
```

## Stuck? Ask AI
```prompt
Here is a list of every animation in my app: [paste it]. For each one, tell me in a single sentence what change it explains, or say "explains nothing - delete". Be blunt. Then rank the deletions by how much time they are costing a user who is in a hurry. Do not suggest any new animations.
```

```prompt
Explain why animating width or height stutters on a cheap phone while animating transform and opacity does not. Use an explanation a non-programmer can follow. Then give me six animation descriptions and ask me which would be cheap and which expensive, without telling me first.
```

```prompt
I want to add a 3D element to my landing page. Interrogate me before agreeing: ask what shape the user actually needs to understand, what the download cost would be on slow 4G, what a screen reader would get from it, and what the static fallback is. If my answers do not justify it, tell me plainly and suggest what to do instead.
```

## Quiz

### Q1 · L3
Which animation earns its place in a canteen ordering app?
- (a) Cards fading and sliding up as the user scrolls the menu
- (b) A progress indicator inside the pay button while the order submits ✅
- (c) A rotating 3D plate of food in the header
- (d) A 600 ms cross-fade between every screen
> It explains a change — the system is working — and it prevents double orders. The other three explain nothing and delay content the user came for.

### Q2 · L2
Why animate only transform and opacity?
- (a) They are the only properties browsers can animate
- (b) They avoid recalculating layout every frame, which is what causes stutter on cheap phones ✅
- (c) They use less memory than colour animations
- (d) They are the only properties that respect reduced motion
> Moving, scaling and fading can be handed to the GPU. Animating width, height or margin forces layout work on every frame, which is where 15 fps stutter comes from.

### Q3 · L4
A user has reduced motion enabled. What should happen?
- (a) All animation stops, including the button's pending state
- (b) Large movement and parallax stop; tap feedback and progress indicators remain ✅
- (c) Nothing changes — the setting applies only to the operating system
- (d) Animations run at half speed
> Reduced motion is not "no feedback". Removing the pending state would reintroduce the double-order problem for the users who asked for less movement.

### Q4 · L4
Your team wants a 3D rotating logo on the landing page. What is the deciding question?
- (a) Whether the team can build it in time
- (b) Whether the user needs to understand a shape — and a logo does not ✅
- (c) Whether competitors have 3D on their pages
- (d) Whether it can be made to run at 60 fps
> 3D earns its download when the shape itself carries information. For decoration, a photograph conveys more in 80 KB than a model does in 2 MB.

### Q5 · L5
Motion was added the week before demo day. Load time went from 2 s to 11 s on a budget Android, and the animations stutter. What is the correct first move?
- (a) Optimise the animations to hit 60 fps on that device
- (b) Delete every animation that does not explain a change, then re-measure ✅
- (c) Show a loading spinner during the 11 seconds
- (d) Ship it, since it runs well on newer phones
> Tuning decoration is work spent on something that should not exist. Deleting first usually recovers most of the time and reveals how few animations were ever load-bearing.

## Go deeper
- [web.dev: Animations guide — what is cheap to animate and why](https://web.dev/animations-guide/)
- [MDN: prefers-reduced-motion](https://developer.mozilla.org/en-US/docs/Web/CSS/@media/prefers-reduced-motion)
