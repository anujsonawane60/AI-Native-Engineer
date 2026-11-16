---
id: m03-10-motion-and-3d
module: m03-design
slug: motion-and-3d
title: "Motion, Micro-Interactions and 3D on the Web"
order: 10
duration_min: 25
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

A team adds animation to their canteen app the week before demo day. Every card fades and slides up on scroll, staggered by 100 ms. The cart badge bounces. The page transitions with a 600 ms cross-fade. The hero has a rotating 3D plate of food, which is 2.4 MB.

On the mentor's iPhone it looks impressive. On a four-year-old budget Android on hostel 4G, three things happen. The page takes eleven seconds before anything is readable, because the 3D model loads before the menu. The scroll animations stutter at around fifteen frames per second, which reads as the app being broken rather than being animated. And a student with vestibular sensitivity gets a headache from the parallax and closes it.

Measured honestly, the animation added 2.9 MB, cost about nine seconds on first load, and made the product feel slower than the plain version did. It did not make anything easier to understand. That is the entire failure: motion was added as decoration, and decoration on a constrained device is a tax.

### The idea

**Motion earns its place only when it explains a change: where something came from, where it went, or that the system is working.**

Everything else is cost — in milliseconds, in battery, in bytes, and in the small irritation of being made to wait for a transition you have already seen four hundred times.

### The analogy

At the canteen counter, useful motion is everywhere and none of it is decorative. The server slides your plate forward: you see it arrive and you know it is yours. The token number flips on the board: you see which number changed and you look at the new one without hunting. Steam rises off the pot: you know it is hot without touching it. Each movement answers a question you actually had.

Now imagine a counter where the plate spun twice before reaching you, the board animated every number for half a second whether or not it changed, and the wall had a rotating sculpture of a samosa. The first time it is funny. The second time you are late for class.

**Where the analogy breaks:** motion at the counter is free and instant because it is physical. Yours costs real resources — frames the phone may not be able to render, bytes over a metered connection, battery on a device that is already at 14%. A physical gesture has no download size. Yours does, and on the devices your users actually own, that is the deciding constraint.

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

### Durations and easing, in numbers

Motion has a small set of working numbers, and they are much shorter than most people expect.

| Purpose | Duration | Easing |
|---|---|---|
| Tap feedback | 50–100 ms | Linear or ease-out |
| Something appearing | 150–250 ms | Ease-out — fast in, settle |
| Something leaving | 100–200 ms | Ease-in — gets out of the way |
| Page or screen change | 200–300 ms | Ease-in-out |
| Loading indicator | Loops, 1–1.5 s | Linear |
| Anything over 400 ms | Almost never | — |

Easing is the difference between a mechanical slide and something that feels physical. Ease-out for things arriving, because they should appear quickly and settle gently. Ease-in for things leaving, because once you have decided to dismiss it you do not want to watch it go. Linear only for loaders and spinners, where the whole point is a steady rhythm.

Two rules on top. **Animate only transform and opacity.** Moving, scaling and fading are cheap because the phone can hand them to the GPU; animating width, height, margin or colour forces the browser to recalculate layout every frame, and that is where the fifteen-frames-per-second stutter comes from. **Exits are faster than entrances.** The same duration in both directions makes dismissal feel sticky.

### Micro-interactions worth having

Only a handful of animations genuinely improve a product, and they all map to something from earlier in this module.

The pressed state on tap — chapter 6's active state, and chapter 3's heuristic 1. A spinner or progress bar inside a button during submission, which is the thing that prevents double orders. A skeleton while content loads, which shows the *shape* of what is arriving and makes the wait feel shorter than a centred spinner does. An item visibly moving into the cart, so the user knows where it went. A shake on a rejected input, paired with text, never alone. A toast sliding in from a fixed position so the eye knows where to look.

That is six. If your product has more than about eight animations, the extras are decoration.

### Reduced motion is not optional

Operating systems let people say "I do not want animation", and they say it for real reasons: vestibular disorders, where parallax and large movement cause genuine nausea and dizziness; migraine triggers; attention and focus conditions; and plenty of people who simply find it faster without. On Android it is under accessibility settings; iOS has Reduce Motion.

Honouring it means large movement, parallax and auto-playing video are off, while essential feedback — the pressed state, the progress indicator — stays, possibly as a simple cross-fade. "No animation" does not mean "no feedback". This is WCAG 2.3.3, and chapter 9 treated the same family of rules as hard requirements. AI will not add reduced-motion handling unless you ask for it, essentially ever.

### 3D: when it earns the download

3D on the web is genuinely accessible now, and it is almost always the wrong choice for a student product. The test is simple: **does the user need to understand a shape?**

Yes for a product you must inspect from multiple angles, a floor plan, a molecule, a map with elevation. No for a rotating logo, a hero decoration, or a 3D plate of food on a menu — a photograph conveys more in 80 KB than a model does in 2 MB.

If the answer is genuinely yes, the budget discipline is what matters. A 3D library is a large download before any model loads. It must never block the first readable content, so it loads only after the page is usable, and only when it scrolls into view. It needs a static image fallback for devices that cannot render it, and a stated fallback when the WebGL context fails rather than a blank rectangle. And it needs a non-visual equivalent, because a rotating model conveys nothing to a screen reader.

A rule of thumb that will serve you for the whole program: on a budget Android on slow 4G, total page weight under about 500 KB keeps you in acceptable territory. A single 3D scene typically spends all of that and more before showing a word. M02's Core Web Vitals chapter and M08's mobile work are where this gets measured properly.

### Where it lives

Motion is the last layer, applied after chapters 5 to 8 are settled — animating a screen with broken hierarchy only draws attention to the break. Chapter 6's eight states are what most of your motion attaches to. Chapter 9's principles constrain it absolutely. M02 covered the rendering cost and Core Web Vitals; M07's analytics is where you find out whether a transition is making people abandon a flow; M08 is where you measure load on real devices rather than a laptop.

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

**Scroll-triggered animation is the worst value in common use.** It delays content the user is actively trying to read, it stutters on cheap phones, and it has to replay every time they scroll back. For a product where people are in a hurry, it is pure cost.

**Motion cannot fix slowness, and usually advertises it.** A 400 ms transition hiding a 2-second load makes the total 2.4 seconds. Users read a slow animation as a slow app.

**Animating the wrong properties is how a smooth design becomes a stuttering one.** Transform and opacity are cheap; width, height, margin, padding and box-shadow are not. If AI gives you motion that stutters, this is the first thing to check.

**A loading spinner with no content shape is weaker than a skeleton.** The skeleton tells the user what is coming and makes the same wait feel shorter.

**Everything feels fine on your laptop.** Your development machine renders anything. Test motion on the cheapest Android phone your team owns, with the network throttled, or you have tested nothing.

**This chapter's tooling moves fast; the numbers do not.** Animation libraries and 3D frameworks churn constantly. Durations, easing, transform-and-opacity, and reduced motion have not changed in a decade.

### Verify it

1. **Ask "what change does this explain?" of every animation.** If you cannot answer in one sentence, delete it. Most teams delete half.
2. **Time them.** Ask AI to list every animation with its duration in milliseconds. Anything over 400 ms that is not a loader is too slow. A list is checkable; "subtle animations" is not.
3. **Test on the cheapest phone you have, with the network throttled.** Watch for stutter and measure how long until the first readable content. If motion delays reading, it is a defect.
4. **Turn reduced motion on in your phone's accessibility settings and reload.** Large movement and parallax should stop; pressed states and progress indicators should remain. If nothing changes, it was never implemented.
5. **Check which properties are animated.** Ask directly: "list every animated property". Anything other than transform and opacity needs a reason.
6. **Measure the page weight before and after.** If motion or 3D added more than about 100 KB, decide explicitly whether the change it explains is worth it.

Red flags: animation nobody asked for; staggered scroll reveals on content people need to read; any duration over 400 ms; width or margin being animated; no reduced-motion handling; 3D above the fold; auto-playing video; parallax on a mobile product; and AI calling motion "smooth and delightful" rather than listing durations.

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
