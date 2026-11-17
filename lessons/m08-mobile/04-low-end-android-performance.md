---
id: m08-04-low-end-android
module: m08-mobile
slug: low-end-android-performance
title: "Performance and Data on Low-End Android"
order: 4
duration_min: 15
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

Your team demos on a mentor's phone and it is fine. Hand the same URL to a classmate with a ₹8,000 Android — 2 GB of RAM, Android Go, 600 MB free — and the screen is white for six seconds. The list stutters, a tap does nothing for a second so she taps again, then Chrome discards the tab for memory and her form is empty. She also says the page "finished her data": the hero image was 2.4 MB and she is on a 1.5 GB daily pack. In India that phone is the median.

### The idea

**Performance on a budget phone is a budget in three currencies: CPU milliseconds, megabytes of RAM, and rupees of mobile data — and your users pay all three.**

A flagship hides every mistake; a ₹8,000 phone charges you for each one. Pick the cheapest phone in your persona's hands, write down numbers you promise not to exceed, and test on it rather than yours.

A 500 KB image and 500 KB of JavaScript are not the same 500 KB: the image is decoded by dedicated hardware and done, while the JavaScript is parsed, compiled and executed on the main thread — the one thread that also draws the screen and handles taps. Budget phones take **three to five times longer** on the same script, and anything over **50 milliseconds** in one task is felt as a tap being ignored.

### The analogy

The canteen has one gas burner and one cook. At 1:10 pm the limit is that one burner: a dish needing eight minutes of flame blocks every simple dish behind it, so the cook pre-chops in the morning and keeps slow dishes off the menu. The burner is the phone's CPU, and JavaScript that must be downloaded, parsed and executed before anything appears is the eight-minute dish. Images are different — sacks of vegetables from the market, where the cost is the trip, not the cooking.

**Where the analogy breaks** in two places. A cheap phone gets *slower* as it heats up and throttles itself, so minute ten is slower than minute one. And a 2 GB phone has no spare counter space: when memory runs out the system does not slow down, it throws your tab away.

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

Images sized for the screen they appear on, modern image formats and caching are where the bytes go.

### Where it lives

On M00's four boxes this is the **client** and the **network**, nothing else: no server upgrade fixes it, which is the reflex M05 already warned you about. M02 gave you Core Web Vitals and budgets as web concepts; this chapter says which numbers to use when the device is a ₹8,000 Android.

**M08 is how to choose plus how to ship a PWA; M12's "Performance Budgets on Real Devices" is the practicum** — budgets wired into CI so a bad build fails before it ships, and profiling on a physical handset.

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

**"It works on my phone" is the most expensive sentence in this module.** Your phone is faster, your Wi-Fi better, your copy already cached. Dev tools at 4× CPU with a slow network profile are a cheap proxy, but a real ₹8,000 handset adds thermal throttling and a memory ceiling no emulator reproduces.

**A budget without a number is a wish.** "JavaScript under 170 KB compressed, total first load under 1 MB, no image over 100 KB, usable within 3 seconds on a slow profile" can be failed; "keep it light" cannot.

**Memory is a correctness problem, not polish.** Rendering 2,000 rows for a list showing eight, or holding every image at full resolution, is how a 2 GB phone discards your tab — and with it everything the user typed. Render only what is on screen; never ship an image larger than it is displayed.

**Fonts block text, and pretty libraries cost CPU.** No visible text until the font arrives; 90 KB of JavaScript to fade in one card.

### Verify it

**Make it state the target device first.** Which phone, how much RAM, what network, what data pack. Code written before that answer is a guess.

**Red flag: adjectives instead of numbers.** Ask for first-load JavaScript in KB compressed, the largest single dependency, and the estimated parse-and-execute time on a chip three to five times slower than a laptop.

**Red flag: no mention of bytes or data cost.** A plan that never converts page weight into a fraction of a daily data pack was written for somebody else's country.

**Red flag: optimisation aimed at the wrong currency.** Minifying 4 KB of CSS while a 2.4 MB image loads above the fold changes nothing; demand the byte and CPU cost of the fix itself.

**Red flag: the memory ceiling ignored.** Ask what happens on a 2 GB phone with 600 MB free when a list has 2,000 rows each with an image.

Then measure it yourself: 4× CPU throttling, slow network, time until text is readable and total bytes, both against your budget.

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
