---
id: m12-09-performance-budgets
module: m12-ship-apps-for-real
slug: performance-budgets
title: "Performance Budgets on Real Devices"
order: 9
duration_min: 25
is_free: false
mission: true
volatility: low
last_reviewed: 2026-11-16
---

## Key learnings
- Set five performance budgets as numbers, on a named budget device
- Measure cold start, app size, scroll smoothness and data per session for real
- Measure a release build on real hardware instead of a dev build on a laptop
- Find the one cause that owns most of your budget before changing anything
- Spot a performance claim backed by an emulator, a flagship or no before-number

## Lesson

### The pain

Your team's app feels instant, and every phone on the team cost at least ₹25,000. Then you hand your release build to a classmate with a ₹8,000 Android phone and 3 GB of RAM. The download is 38 MB and she asks whether it will finish before her data pack runs out. The first open takes nine seconds, the screen white, and she taps twice more. The list scrolls in visible jerks. She switches to WhatsApp for a minute; when she returns the app restarts from scratch, because Android reclaimed its memory. None of that appeared on your phone, which is four times faster, had the app warm, and was running a development build anyway.

Then the team "optimises": someone asks an AI to make the app faster, applies six suggestions, and reports it is "much snappier". No before number, no after number, and the 24 MB of uncompressed images that caused most of the problem is still there.

### The idea

**A performance budget is a number you agreed before you built anything, measured on the cheapest device you promise to support, using a release build.**

Every word is load-bearing. A *number*, so you can fail. Agreed *before*, so it is a constraint and not an excuse. On the *cheapest device you support*, because that is where your users are. A *release build*, because a dev build is always slow.

Five budgets are enough. Reasonable starting values for a ₹8,000 Android phone on patchy mobile data:

**Cold start under 3 seconds**, from tap to a usable screen.
**Download size under 25 MB** for the Play Store listing's figure.
**First meaningful list rendered under 1 second** after the screen opens.
**No frozen frames while scrolling** your main list.
**Under 1 MB of data** for a typical session.

Measure them honestly and you will usually find one cause owns most of the gap. It is almost always images.

### The analogy

The canteen promises to serve a thali within 90 seconds. That promise only means something measured at 1:10 pm with forty people queuing on the day the second gas cylinder is empty. At 4 pm with nobody waiting, every canteen in India serves within 90 seconds — true and useless.

**Where the analogy breaks:** the canteen can respond to a bad measurement by hiring a second server or buying a bigger pan. You cannot upgrade your users' phones. Your only lever is to do less — fewer bytes, fewer requests, less work on the main thread. That asymmetry is why mobile performance work is mostly subtraction.

### Picture it

```
 tap icon ──▶ process starts ──▶ JS loads ──▶ data arrives ──▶ usable
     │             │                 │              │            │
     │  0.4 s      │   1.1 s         │   2.9 s      │   4.6 s    │
     └─────────────┴─────────────────┴──────────────┴────────────┘
                         total: 9.0 s on a 3 GB phone

  where it went:          budget: 3.0 s
  ████████████████████████  images 4.6 s   ◀── owns the gap
  ███████████               JS bundle 2.9 s
  ████                      process start 1.1 s
  ██                        native init 0.4 s
```

A nine-second cold start broken into its four stages, with one stage — unoptimised images — accounting for more than the entire budget on its own.

### Where it lives

M05 taught you that slowness is a sum of measurable delays and that averages lie. This is that lesson on a device whose CPU is slow, whose memory is scarce, and whose network the user pays for. You measure in four places:

- **A stopwatch and your eyes** on a real phone — the measurement that matters most, and it costs nothing
- **The Play Console listing** — the download size real users see
- **Android vitals** — startup time and frozen frames from the field
- **A development profiler** — for finding the cause once you know there is one, never for the headline number

The release build matters because the gap is large: production JavaScript is minified, the development bundle is served over the network from your laptop, and development mode runs extra checks on every render. A dev build can be three times slower than the release build of identical code.

### Words to own

| Term | Plain meaning | Example |
|---|---|---|
| Budget device | The cheapest phone you promise to support | ₹8,000 Android, 3 GB RAM |
| Cold start | First open with nothing in memory | Tap icon to usable screen |
| Warm start | App was in memory already | Much faster, not the number |
| Time to interactive | When the user can actually do something | Not when pixels appear |
| Frozen frame | A frame that took far too long | The visible jerk in a scroll |
| Main thread | Where UI work must happen | Block it and the app freezes |
| Bundle size | How much JavaScript ships | Often the second biggest cost |
| Download size | What the store says users will download | The number in your listing |
| Low-memory kill | OS reclaims your app in the background | User loses their place |
| Release build | Optimised, signed, production JavaScript | The only honest thing to measure |
| Field data | Measurements from real users' devices | Android vitals |

### Tradeoffs and gotchas

**Images are almost always the answer.** Before restructuring any code, count the bytes of images your first screen downloads and the pixel dimensions of each. A 1600-pixel photo displayed in a 120-pixel thumbnail is 99% waste. Teams routinely find half their cold start here.

**Measure one change at a time.** Applying six suggestions at once and declaring victory teaches you nothing. One change, measure, keep or revert, record — the Director's Loop from M00 applied to numbers.

**Five runs, and report the worst.** Cold start on a cheap phone varies with whatever else the phone is doing, so one measurement is an anecdote. Quote the slowest: that is a user's experience on a bad day.

**Animations cost more than you think on 3 GB phones**, and a low-memory kill is a design problem, not only a performance one. Keep decorative animation off your heaviest screens, and save the user's place locally — chapter 4's local store doing double duty.

**Do not optimise what is not in the budget.** A startup function that takes 4 milliseconds is not your problem, however satisfying it is to improve. Work only on the stage that owns the largest share, as in M05. And five achievable numbers your team checks beat twelve aspirational ones nobody measures.

### Verify it

**Red flag: a number from an emulator or a flagship phone.** Ask four questions of any performance claim: which physical device, what price, how much RAM, and was it a release build? If one is missing, the number is not a measurement.

**Red flag: no before number.** "This will make your app faster" is unfalsifiable. Insist on the pattern: measure, change one thing, measure again, state the difference in milliseconds or megabytes.

**Red flag: a dev build in the measurement.** The build you measure must be the build you ship.

**Red flag: optimisation aimed at a stage that was never the problem.** Ask which startup stage the change removes time from, and how many milliseconds that stage was. Memoising a component is not a fix for 24 MB of images.

**Red flag: an average instead of the worst case, or no field confirmation.** "Average startup improved to 2.1 seconds" hides the user whose phone takes six, and your one phone is one data point. A week after shipping, check Android vitals.

## Walkthrough
1. Agree your five budgets as a team, as numbers, and name the exact budget device: make, model, price, RAM. Write all of it in your decision log.
2. Get hold of that device or the closest thing available — a classmate's phone, a family member's, a second-hand handset in the cohort. An emulator is not acceptable for this.
3. Build a release build and install it on that phone. Note the download size the store or the file itself reports.
4. Force-stop the app, then time the cold start with a stopwatch from tap to a screen you could actually use. Do this five times and write down all five numbers.
5. Record the screen while scrolling your main list for ten seconds. Watch the recording back at reduced speed and count the visible jerks.
6. Reset the phone's data usage counter, use the app for a typical two-minute session, and read how many kilobytes your app consumed.
7. Open the app, switch to WhatsApp, scroll for three minutes, then return. Record whether your app restarted and what the user lost.
8. Now break the cold start into stages: native init, JavaScript load, first data arriving, first render. Use the profiler and your own logging to attribute milliseconds to each, summing to your measured total.
9. Count the images on your first screen: how many bytes each, and what pixel size they are displayed at. Total the waste.
10. Fix exactly one thing — the stage with the largest number, which is probably the images. Change nothing else.
11. Re-measure all five budgets the same way on the same phone, and record the before and after side by side with the difference stated in milliseconds, megabytes or kilobytes.

## Talk to AI
```prompt
CONTEXT:      React Native app built with Expo, Android first, real users in
              India. Product: [one sentence].
              Our budget device: [make and model], Rs [price], [X] GB RAM,
              Android [version].
              All of the following are measured on that device with a
              RELEASE build, on mobile data:
              - download size: [X] MB
              - cold start, five runs: [a, b, c, d, e] seconds
              - time from screen open to first list rendered: [X] s
              - visible jerks counted in a 10-second scroll: [X]
              - data used in a 2-minute session: [X] KB
              - app restarted after 3 minutes in the background: [yes/no]
              - images on our first screen: [count] totalling [X] MB,
                displayed at [sizes]
              Our budgets: cold start under 3 s, download under 25 MB,
              first list under 1 s, zero frozen frames, under 1 MB per
              session.

GOAL:         Find the single cause that owns the largest share of our
              cold-start gap, and fix only that, with a measured before and
              after.

REQUIREMENTS: 1. Turn my numbers into a stage-by-stage breakdown of the
                 cold start - native init, JavaScript load, first data,
                 first render - in milliseconds, summing to my worst run
              2. Name the single largest contributor and say how confident
                 you are given what I measured and what I did not
              3. Tell me the one further measurement that would raise that
                 confidence, and exactly how to take it on this phone
              4. Only then propose fixes, ranked by milliseconds or
                 megabytes saved per hour of work
              5. For each fix, state which stage it removes time from and
                 roughly how much
              6. Tell me how to verify each fix on the same device, with
                 the same method, so the before and after are comparable
              7. Tell me which of my five budgets I will still miss after
                 the top fix, and what it would take to meet it

CONSTRAINTS:  Free tiers only. We cannot upgrade our users' phones. Assume
              metered data at Indian prices. Assume the OS will kill our
              app in the background. Measurements must come from a release
              build on the physical device named above, never an emulator.

NON-GOALS:    Do not propose a rewrite or a different framework. Do not
              suggest removing features. Do not propose more than one
              change at a time. Do not report averages.

ACCEPTANCE:   - Every row of your breakdown has a number, not an adjective
              - The rows sum to roughly my slowest measured run
              - You have named one measurement I am missing
              - Your top-ranked fix names the stage it removes
              - You have told me how to prove the fix worked on the same
                phone, and what field data would confirm it later

ASK:          Give me the stage breakdown and the single bottleneck first.
              Do not propose any fix until I have confirmed the breakdown
              matches what I observed.
```

## Mission

**Objective.** A real before-and-after performance measurement for your own product, taken on a named budget phone with a release build, with one change made and its effect proved in numbers.

**Steps.**
1. **Set the budgets** — agree five numbers as a team and name the budget device by make, model, price and RAM. Record it all.
2. **Get the device** — borrow the cheapest Android phone you can find in your cohort or family. Record whose it is and what it cost. An emulator disqualifies the mission.
3. **Measure the baseline** — release build on that phone: download size, five cold-start runs, time to first list, jerks counted from a screen recording, data per session, and whether the OS killed your app after three minutes in the background.
4. **Attribute the time** — break the slowest cold start into four stages with a millisecond figure for each, summing to your measured total. Guess where you must, but assign every millisecond.
5. **Brief** — write a Director's Brief for fixing the single largest stage. NON-GOALS must include "no rewrite" and "one change only".
6. **Change one thing** — make only the top-ranked fix. Nothing else in the same commit.
7. **Re-measure** — repeat step 3 identically on the same phone and state every difference in milliseconds, megabytes or kilobytes.
8. **Report the worst, not the average** — quote your slowest of five runs before and after.
9. **Confirm in the field** — check Android vitals a few days after shipping and record whether real users' startup numbers moved in the same direction.
10. **Document** — one decision-log entry: your five budgets, which you now meet, which you miss, and the tradeoff you accepted.

**Upload.**
- Your five budgets and your named budget device, with price and RAM → screenshot or text
- A photo of the actual phone running your app, with the handset visible
- Your baseline table: all five measurements, including all five cold-start runs
- Your four-stage cold-start breakdown with milliseconds per stage
- Your Director's Brief → post caption or an attached file
- The AI conversation showing the breakdown and the bottleneck → share link in `ai_chat_url`
- Your after table, measured identically, with the difference stated per budget
- The screen recordings before and after, or the jerk counts from them
- Reflection, 150 words max → one tradeoff you chose, one thing the AI got wrong

**Self-check.** Tick every box before you submit.
- [ ] Every number came from a physical phone, never an emulator
- [ ] Every number came from a release build, not a development build
- [ ] I named the device make, model, price and RAM
- [ ] I took five cold-start runs and I quote the slowest, not the average
- [ ] My stage breakdown sums to roughly my measured total
- [ ] I changed exactly one thing between the before and after
- [ ] My before and after were measured the same way on the same phone
- [ ] I counted the bytes and the displayed pixel size of my first screen's images
- [ ] I recorded what happened after three minutes in the background
- [ ] I checked field data to see whether real users improved too
- [ ] My reflection names a budget I still miss and why

**Stretch.** Give the same brief and the same numbers to a second AI tool and compare the bottlenecks they identify. Where they disagree, decide which measurement would settle it and take it.

## Stuck? Ask AI
```prompt
Explain why a development build of a React Native app is much slower than the release build of the exact same code. Name each difference and what it costs. Then tell me what would go wrong in my reasoning if I measured startup time on a development build and used that number to decide what to optimise.
```

```prompt
I have an image on my app's first screen that is 1600 pixels wide, 2.4 MB, and displayed in a 120-pixel-wide thumbnail on a budget Android phone. Walk me through exactly how much of that is waste, in bytes and in seconds on a slow connection. Then give me the specific dimensions, format and compression I should serve instead, with the reasoning for each number.
```

```prompt
Give me five short scenarios of a mobile app feeling slow on a cheap Android phone. For each one, tell me the single measurement I should take first to tell apart a bundle-size problem, an image-weight problem, a main-thread problem and a network problem. Ask me to answer before you explain your reasoning.
```

## Quiz

### Q1 · L3
What makes a performance budget useful rather than decorative?
- (a) It is reviewed by your mentor
- (b) It is a number, agreed before building, measured on the cheapest device you support with a release build ✅
- (c) It covers every metric the profiler reports
- (d) It is based on what competitors achieve
> Each part does work: a number means you can fail it, agreeing beforehand makes it a constraint, and the device and build type are what make the measurement about your actual users.

### Q2 · L4
Your team reports that the app is "much snappier" after applying six AI suggestions at once. Why is this worthless as evidence?
- (a) Six changes is too few to matter
- (b) There is no before number and no way to tell which change did anything, or whether anything did ✅
- (c) Snappiness cannot be measured on Android
- (d) AI suggestions are never effective
> One change, measure, keep or revert, record. Without a baseline and isolated changes you have a feeling, and the real cause — often the images — is usually still there.

### Q3 · L4
Which measurement setup gives a number you can trust?
- (a) A development build on an emulator on your laptop
- (b) A release build on a ₹8,000 Android phone with 3 GB RAM, slowest of five runs ✅
- (c) A release build on the team's ₹30,000 phone, averaged over ten runs
- (d) A development build on your own phone over campus Wi-Fi
> The emulator uses your laptop's hardware, the dev build can be three times slower than release, and a flagship phone is not where your users are. Quoting the slowest run reflects a real bad day.

### Q4 · L4
Your cold start is 9 seconds and your images account for 4.6 of them. An AI proposes memoising a component that renders in 4 milliseconds. What is the right response?
- (a) Apply it; every optimisation helps
- (b) Reject it and ask which startup stage it removes time from, since 4 ms cannot touch a 4.6-second problem ✅
- (c) Apply it and then fix the images
- (d) Ask for ten more micro-optimisations to add up
> This is M05's "optimising a loop that runs once" on mobile. Always ask which measured stage the fix removes and how large that stage was.

### Q5 · L5
After shipping your top fix, your own phone shows startup down from 9 seconds to 4. Why check Android vitals a week later?
- (a) To confirm the store listing updated
- (b) Because one phone is one data point, and field data across all your users is the only proof the improvement is real for them ✅
- (c) Because Android vitals is the only place crash data appears
- (d) To reset your performance budget automatically
> Your reasoning was done on one device. Field data tells you whether the users on devices you never touched, on networks you never used, actually got faster.

## Go deeper
- [Android developers: app startup time](https://developer.android.com/topic/performance/vitals/launch-time)
- [web.dev: performance budgets 101](https://web.dev/articles/performance-budgets-101)
