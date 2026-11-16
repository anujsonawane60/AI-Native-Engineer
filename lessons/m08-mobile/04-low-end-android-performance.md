---
id: m08-04-low-end-android
module: m08-mobile
slug: low-end-android-performance
title: "Performance and Data on Low-End Android"
order: 4
duration_min: 25
is_free: false
mission: false
volatility: low
last_reviewed: 2026-11-16
---

## Key learnings
- Treat the cheapest phone your persona owns as the specification, not the exception
- Explain why JavaScript costs CPU time, and why cheap chips pay it five times over
- Convert page weight into rupees of your user's mobile data
- Set a performance budget with numbers your team can fail against
- Catch an AI that optimised for a flagship and never mentioned the device

## Lesson

### The pain

Your team demos on a mentor's phone and it is fine. Then you hand the same URL to a classmate with a ₹8,000 Android — 2 GB of RAM, an entry-level chip, Android Go, 600 MB of free storage — and watch.

The screen is white for six seconds. Then the layout appears without text, because a web font is still downloading. She scrolls and the list stutters; she taps a button, nothing happens for nearly a second, so she taps again. Halfway through, Chrome discards the tab because the phone is out of memory, and she is back at a blank page with her form empty. She also tells you, unprompted, that the page "finished her data" — your hero image was 2.4 MB and she is on a 1.5 GB daily pack.

The code is identical to what ran fine on the mentor's phone. Nothing is broken. The product was never specified for the device most of its users hold, and in India that device is not an edge case — it is the median.

### The idea

**Performance on a budget phone is a budget in three currencies: CPU milliseconds, megabytes of RAM, and rupees of mobile data — and your users pay all three.**

A flagship hides every mistake. A ₹8,000 phone charges you for each one, several times over on CPU-bound work, because the gap between a cheap chip and an expensive one is far wider than the gap in their price. The discipline is unglamorous: pick the cheapest phone in your persona's hands, write down numbers you promise not to exceed, and test on that phone rather than yours.

### The analogy

The canteen has one gas burner and one cook.

At 1:10 pm the limit is not the menu, the recipes or the number of students. It is that one burner. A dish needing eight minutes of flame blocks every dish behind it, however simple. So the experienced cook pre-chops in the morning, keeps common items ready, and refuses to put a slow dish on the lunch menu at all.

The burner is the phone's CPU. JavaScript that must be downloaded, parsed and executed before anything appears is the eight-minute dish, and on a cheap chip the flame is weak. Images are different: they are sacks of vegetables carried in from the market, and the cost is the trip, not the cooking. Weak flame and expensive deliveries are two problems with two fixes, and confusing them wastes a week.

**Where the analogy breaks** in two places. The canteen's burner does not get *slower* as the day gets hotter; a cheap phone does, because it heats up and throttles itself, so minute ten of your app is slower than minute one. And the canteen has unlimited counter space, while a 2 GB phone does not — when memory runs out the system does not slow down, it throws your whole tab away.

### Picture it

```
  cost of 500 KB of JavaScript, download excluded
  ────────────────────────────────────────────────────────
  flagship phone      ██                        ~0.3 s
  mid-range phone     ██████                    ~1.0 s
  ₹8,000 Android      ████████████████████      ~3.5 s
  ────────────────────────────────────────────────────────
  cost of a 2.4 MB image on a 1.5 GB daily pack
  ────────────────────────────────────────────────────────
  one page load       ██                        ~0.16% of the pack
  twenty loads a day  ██████████████████████    ~3.2% of the pack
```

The same bundle costs a budget phone several seconds of pure CPU work that a flagship absorbs in a fraction of a second, and bytes cost the user money separately from costing them time.

### Why JavaScript is the expensive part

A 500 KB image and 500 KB of JavaScript are not the same 500 KB. The image is decoded by dedicated hardware and then it is done. The JavaScript must be parsed, compiled and executed on the main thread — the same single thread that draws the screen and handles taps. Until it finishes, your app can look complete and respond to nothing: the "I tapped twice" bug from the opening story.

Three numbers to carry. Budget phones take roughly **three to five times longer** than flagships on the same script. The main thread does **one thing at a time**, so a long task makes taps feel ignored. And anything over about **50 milliseconds** in one task is something the user can feel.

So the highest-value question you can ask about a page is "what are we downloading and executing before the user can read anything?" — and the usual answer in an AI-generated first draft is an animation library, a charting library, a whole icon set, a date library and three fonts, none of which the feature needed.

### The data bill, in rupees

Mobile data in India is cheap per gigabyte and strictly rationed per day. A typical prepaid pack gives 1.5 GB a day, and students count it — they turn data off between uses, they ask whether an app "eats data", and they uninstall things that do.

Do the arithmetic once and you will never forget it. A 3 MB page loaded twenty times a day is 60 MB: 4% of a 1.5 GB daily pack, for one app. Trim the page to 600 KB and it is 12 MB, under 1%. That is not a micro-optimisation; it is the difference between a product a hostel student keeps and one they delete during exam week.

The biggest wins are the same three, in order: **images sized for the screen they appear on** (a 1,200-pixel photo on a 360-pixel phone wastes three quarters of its bytes), **modern image formats**, and **caching so a repeat visit downloads almost nothing** — M02's HTTP caching applied to a metered connection.

### Memory, and the tab that vanishes

A 2 GB phone running Android, Chrome, WhatsApp and your app has very little headroom, and Android Go devices are tighter still. When memory runs out the system does not degrade gracefully; it kills something. Usually the background tab or app — yours.

Two things cause this in student projects, every time. **Rendering a list of everything**: 2,000 rows of DOM for a list showing eight. And **holding every image at full resolution** because none was resized. Render only what is on screen; never ship an image larger than it is displayed. Both are one brief each to an AI, provided you know to ask.

This rejoins chapter 2: if the tab is discarded, the user's typed input is gone unless you wrote it to device storage. Low memory is the most common *cause* of termination on the phones your users own.

### Where it lives

On M00's four boxes this is the **client** box and the **network** box, nothing else. No server upgrade fixes it, which is worth saying because "our app is slow, let us get a bigger instance" is the reflex M05 already warned you about. A fast server cannot make a weak CPU parse your bundle faster.

M02 gave you Core Web Vitals and performance budgets as web concepts; this chapter says which numbers to use when the device is a ₹8,000 Android. M03's touch-target and reduced-motion rules are performance rules in disguise here, because an animation that drops frames reads as a broken app. M05's latency chapter supplies the network half, and chapter 3's offline work keeps a bad network from becoming a blank screen.

This chapter is the **overview**: the device constraints and how to set a budget. **M12's "Performance Budgets on Real Devices" is the practicum** — budgets wired into CI so a bad build fails before it ships, profiling on a physical low-end handset, and native rendering performance. Here you write the budget down; there it becomes a gate.

### Words to own

| Term | Plain meaning | Example |
|---|---|---|
| Main thread | The one thread that draws and handles taps | Blocked by a long script |
| Parse and execute | The CPU work JavaScript costs after download | 3.5 s for 500 KB |
| Long task | One job over ~50 ms; taps feel ignored | A heavy render pass |
| Thermal throttling | A hot chip slowing itself down | Minute ten is slower |
| Android Go | A low-memory edition of Android | Common under ₹8,000 |
| Tab discard | The OS reclaiming your tab's memory | Form contents lost |
| Performance budget | Numbers you promise not to exceed | "JS under 170 KB gzipped" |
| Virtualised list | Rendering only the rows on screen | 8 rows, not 2,000 |
| Data pack | The user's daily metered allowance | 1.5 GB a day |

### Tradeoffs and gotchas

**"It works on my phone" is the most expensive sentence in this module.** Your phone is faster, your Wi-Fi is better, and your copy of the site is already cached. Nothing you observe on it is evidence about your users.

**Throttled dev tools are a good proxy, not the real thing.** CPU at 4× slower with a slow network profile catches most problems and costs nothing. But a real ₹8,000 handset adds thermal throttling, a worse GPU and a memory ceiling no emulator reproduces. Your team of three should share one cheap test phone.

**A budget without a number is a wish.** "Keep it light" fails nothing. "JavaScript under 170 KB compressed, total first load under 1 MB, no image over 100 KB, usable within 3 seconds on a slow profile" is a budget, because a build can be measured against it and found wanting.

**Beware the second-order cost of a pretty library.** 90 KB of JavaScript to make one card fade in is a bad trade here. M03's rule that motion must explain a change rather than decorate one is also the cheapest performance decision you will make.

**Fonts block text.** A custom web font usually means no visible text until it arrives — the blank-layout bug from the opening story. A system font or an immediately visible fallback costs nothing and is almost never in a first draft.

**Measure before and after, and write both down.** In six weeks the only way to prove a change helped is to have recorded what it was before.

### Verify it

Every AI will happily produce "optimisations". Here is how to tell whether it understood the device.

**Make it state the target device first.** The right opening move is a question or a stated assumption: which phone, how much RAM, what network, what data pack. An answer that begins with code has not asked the only question that matters.

**Red flag: a bundle that is fine on a flagship and unusable on a budget phone.** Ask for numbers, not adjectives: JavaScript on first load in KB compressed, the largest single dependency, and the estimated parse-and-execute time on a chip three to five times slower than a laptop. If it cannot produce those, it has not thought about CPU.

**Red flag: no mention of bytes or data cost.** A plan for Indian users that never converts page weight into a fraction of a daily data pack was written for somebody else's country.

**Red flag: optimisation aimed at the wrong currency.** Minifying 4 KB of CSS while a 2.4 MB image loads above the fold changes nothing. Ask which measured number the change moves, and by how much. A virtualised-list library may pay for itself; an animation library never does, so demand the byte and CPU cost of the fix itself.

**Red flag: the memory ceiling ignored.** Ask what happens on a 2 GB phone with 600 MB free when the list has 2,000 rows each with an image. If the answer omits "the tab may be discarded", keep pushing.

Then measure it yourself in four minutes: CPU throttled 4× on a slow network profile, record time until text is readable and total bytes transferred, and compare both against your budget. A number that fails your own budget is the only proof that counts.

## Walkthrough
1. Find the cheapest Android phone among your team and your friends. Note its price, RAM, free storage and Android version. That phone is now your specification.
2. Open your deployed product on it, over mobile data rather than Wi-Fi, with a stopwatch. Record the time until you can read text, and the time until a tap responds.
3. On a laptop, open the same page in dev tools, set CPU throttling to 4× slower and the network to a slow profile, and reload. Record the same two numbers for comparison.
4. In the Network tab, sort by size and write down the three largest things the page downloads, with their sizes. Then record the total bytes transferred on first load.
5. Convert that total into money: multiply by how many times a day a real user opens the page, then express it as a percentage of a 1.5 GB daily pack.
6. In the Performance or Coverage panel, find how much of your JavaScript is actually used on first load. Unused JavaScript is pure CPU cost for your users.
7. Name every third-party library in your bundle and the feature that needs it. Anything you cannot justify in one sentence goes on a removal list.
8. Find your longest list screen and count the rows it renders. If it renders more than roughly fifty while showing eight, log it as a memory and CPU problem.
9. Check images: for the largest one, compare its pixel width with the width it is displayed at. Anything more than twice the display width is wasting your user's rupees.
10. Write your performance budget as four numbers: KB of JavaScript, total KB on first load, seconds to readable text on a slow profile, and KB for the largest single image.
11. Brief an AI to fix the single biggest number only, re-measure on the same cheap phone, and record before and after in your decision log.

## Talk to AI
```prompt
CONTEXT:      I am a student directing AI rather than writing code. Our
              product is [one sentence]. Stack: [your stack], free tier.

              My target device, which is the specification and not an
              edge case: an Android phone costing about Rs 8,000, 2-3 GB
              RAM, entry-level chip, 16 GB storage with under 1 GB free,
              on patchy 4G with a 1.5 GB daily metered data pack. Users
              open the product roughly [X] times a day.

              Measurements I have actually taken, on that phone and with
              dev tools at 4x CPU throttling and a slow network profile:
              - time until text is readable: [X] s
              - time until a tap responds: [X] s
              - total bytes on first load: [X] KB
              - the three largest downloads: [name and size each]
              - JavaScript shipped on first load: [X] KB compressed
              - fraction of that JavaScript actually used: [X] %
              - rows rendered on my longest list: [X]
              - third-party libraries in the bundle: [list them]

GOAL:         A ranked plan to make this usable on that phone, and a
              written performance budget my team can fail a build against.

REQUIREMENTS: 1. Split my measured load into CPU cost, bytes cost and
                 network wait, with estimated numbers that sum to what I
                 measured, and say which of the three is my real problem
              2. Estimate the parse-and-execute cost of my JavaScript on
                 a chip three to five times slower than a laptop
              3. Convert my page weight into a percentage of a 1.5 GB
                 daily data pack at my usage frequency
              4. Rank fixes by the number each one moves, largest first,
                 and state for every fix how many KB or milliseconds it
                 saves and what it costs me to do
              5. For each third-party library I listed, say whether it
                 earns its bytes on this device, and name a cheaper way
                 to get the same outcome
              6. Tell me what will happen on a 2 GB phone with under
                 1 GB free, including whether my tab may be discarded
              7. Write my performance budget as four hard numbers

CONSTRAINTS:  Free tier only. I cannot change hosting this week. Do not
              assume a fast device or Wi-Fi anywhere in your reasoning.
              Any fix that adds a dependency must show its own byte and
              CPU cost. Assume the user pays for every megabyte.

NON-GOALS:    Do not rewrite my app or change framework. Do not suggest a
              bigger server - this is a client problem. Do not propose
              anything I cannot measure on a phone I already own. Do not
              plan a CI performance gate; that is a later module.

ACCEPTANCE:   - Your breakdown names one dominant cost, not three
              - Every fix is quantified in KB or ms, never in adjectives
              - At least one row of your table tells me to delete
                something rather than add something
              - The data-cost section is expressed in percentage of a
                daily pack
              - The budget is four numbers a build could fail against

ASK:          Give me the three-way cost breakdown and the one dominant
              cost first, then stop. No fixes until I reply.
```

## Stuck? Ask AI
```prompt
Explain to someone who has never written code why 500 KB of JavaScript is far more expensive for a cheap phone than a 500 KB photograph, even though both are 500 KB. Use one everyday analogy, keep it under 250 words, and end with the one question I should ask about any page.
```

```prompt
I have a web page that loads 3 MB and a user who opens it twenty times a day on a 1.5 GB daily data pack in India. Do the arithmetic with me step by step: what fraction of their pack my app consumes, what that would be if I got the page to 600 KB, and what the three largest sources of those bytes usually are in a page like mine. Then tell me which single change usually gives the biggest reduction.
```

```prompt
Give me five scenarios where an app feels slow on a budget Android phone: a blank screen for six seconds, a tap that does nothing for a second, a stuttering scroll, a tab that reloads itself, and text that appears late. For each one, tell me which of CPU, memory, bytes or network latency is the likely cause and the single measurement I should take to confirm it. Ask me to guess before you answer.
```

## Quiz

### Q1 · L3
Your page ships 500 KB of JavaScript. Roughly how does the CPU cost of that compare between a flagship and a ₹8,000 Android?
- (a) About the same, since both run the same code
- (b) Three to five times longer on the budget phone ✅
- (c) Slightly slower, around 20% worse
- (d) Faster on the budget phone, because it runs a lighter OS
> The gap between cheap and expensive chips on parse-and-execute work is much larger than the price gap suggests. This is why a bundle that is invisible on a flagship becomes seconds of blank screen on the median Indian device.

### Q2 · L2
Why does a long JavaScript task make taps feel ignored?
- (a) Touch events are disabled while scripts download
- (b) The main thread does one thing at a time, and it also handles taps ✅
- (c) The phone throttles the touchscreen to save battery
- (d) The browser queues taps until the next image finishes decoding
> Drawing the screen and handling input share one thread with your script. Anything over roughly 50 ms in a single task is something the user can feel, which produces the double-tap bug.

### Q3 · L4
A user opens your 3 MB page twenty times a day on a 1.5 GB daily pack. What share of their pack does your app consume?
- (a) About 0.4%
- (b) About 4% ✅
- (c) About 40%
- (d) Too little to be worth calculating
> 3 MB × 20 is 60 MB, which is roughly 4% of 1.5 GB — for one app. Getting the page to 600 KB takes it under 1%, which is the difference between a product a hostel student keeps and one they delete.

### Q4 · L4
An AI proposes minifying your CSS to fix a six-second blank screen, while a 2.4 MB hero image loads above the fold. What is wrong?
- (a) CSS minification breaks older Android browsers
- (b) It optimises a step worth a few kilobytes while the dominant cost is untouched ✅
- (c) Minified CSS costs more CPU to parse
- (d) Nothing — every saving helps equally
> This is M05's "caching the fastest part" failure wearing mobile clothes. Always ask which measured number the change moves, and by how much, before accepting a fix.

### Q5 · L5
A list screen renders 2,000 rows, each with a full-resolution image, on a 2 GB phone with 600 MB free. What is the most likely user-visible outcome?
- (a) Scrolling is slightly less smooth than usual
- (b) The OS discards the tab and the user's typed input is lost ✅
- (c) The images load in a lower resolution automatically
- (d) The page refuses to render and shows an error
> When memory runs out Android kills something rather than degrading. That rejoins the lifecycle chapter: anything not written to device storage dies with the tab, which is why virtualised lists and sized images are correctness issues here, not polish.

## Go deeper
- [web.dev: Performance budgets 101](https://web.dev/articles/performance-budgets-101)
- [web.dev: The cost of JavaScript](https://web.dev/articles/the-cost-of-javascript-in-2019)
