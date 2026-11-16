---
id: m12-01-past-the-mvp
module: m12-ship-apps-for-real
slug: past-the-mvp
title: "Past the MVP: What Production Actually Demands"
order: 1
duration_min: 25
is_free: false
mission: false
volatility: low
last_reviewed: 2026-11-16
---

## Key learnings
- Name the five promises a production app makes that an MVP does not
- Tell the difference between a demo that works and a product strangers can rely on
- Say what this module adds on top of the mobile overview you already did in M08
- Budget the real money and real calendar days that app stores cost a student team
- Spot an AI plan that assumes a fast network, a new phone and an instant release

## Lesson

### The pain

Your MVP works. You shipped it in M11, you demoed it, your mentor nodded, and the installable PWA you built at the end of M08 sits on your own home screen. Twelve people you know have used it.

Then your team puts the link in three WhatsApp groups and 180 strangers arrive over a weekend.

A student in Nashik on a ₹8,000 Android phone with 3 GB of RAM opens it on 4G at the bus stand. It takes eleven seconds. She switches to WhatsApp to answer a message, comes back, and the app has been killed by the operating system and restarts from the login screen, losing the order she had half-filled. She never tells you. She leaves a one-line review saying "not working" and uninstalls.

Meanwhile a different user hits a crash on a code path you never ran. You find out nine days later, from a 2.9-star rating. You push a fix — and discover it has to go through a store review queue you have not set up, from an account nobody created, signed with a key that does not exist yet. Your two-day bug becomes a two-week bug.

Nothing in that paragraph is a coding problem. Every one is a production problem, and production problems are what this module is about.

### The idea

**An MVP proves the idea can work once. Production promises it will keep working for someone you have never met, on a device you do not own, on a network you cannot control — and that you can fix it when it does not.**

Those promises are specific. There are five of them, and every chapter in this module is one of them made real.

**It runs on their device, not yours.** Budget Android, small screen, 200% font scale, Hindi, no headroom.
**It survives a bad network.** Patchy 4G, a train tunnel, 300 ms round trips, data the user is paying for by the megabyte.
**It is findable and installable.** A store listing, a review that can reject you, an icon and a name a stranger will judge in four seconds.
**You can see what is going wrong without being told.** Crash reports with real stack traces, a crash-free sessions number, performance measured in the field.
**You can change it quickly and safely.** A signed build, a release channel, over-the-air updates for the parts that allow it, and a way back when a release is bad.

### The boundary with M08

M08 was the overview. It taught you how to *choose* between native, cross-platform, PWA and hybrid; it explained app lifecycle, permissions and push at the level of what the words mean; and its mission turned your product into an installable PWA. A PWA is still a legitimate answer for many products.

M12 is the practicum. Here you build a real React Native app with Expo, sign it, submit it to both stores, measure it on a cheap phone, instrument its crashes, and write the listing that decides whether anyone installs it. Where M08 said "Android requires a runtime permission for notifications since Android 13", M12 has you request it at the right moment and measure your opt-in rate.

If a sentence here feels familiar from M08, that is the design. The difference is that this time it ends with something in a store.

### The analogy

In M00 the canteen was one person cooking in a hostel room for three friends. That is your MVP. You know the three friends, you know they will forgive a burnt dosa, and if you run out of oil you go and buy some.

Production is a licensed stall in the main canteen block. You need an FSSAI licence before you can serve anyone. Your prices must be on a board. You declare what is in the food. You open at a fixed time whether or not you feel like it, you keep a complaints register, and when the 1:10 pm rush arrives with forty people you cannot say "come back later". One bad batch produces a complaint that stays on the record.

**Where the analogy breaks**, in two ways that matter. First, a canteen that has a bad day can shut and reopen tomorrow with a clean slate; an app store rating is cumulative and effectively permanent, so your first fifty users set a number that takes hundreds of later users to move. Second, a food inspector visits occasionally; app review happens on *every single update* you ever ship, against rules that change without telling you. That is why chapters 5 and 11 are tagged as high-volatility and get re-checked every six months.

### Picture it

```
  MVP (M11)                                    PRODUCTION (M12)
┌──────────────────┐                      ┌────────────────────────┐
│ works on my      │── real device ──────▶│ ₹8,000 Android, 3 GB   │
│ laptop + my      │── bad network ──────▶│ offline queue, retries │
│ phone            │── store review ─────▶│ signed build, listing  │
│                  │── field errors ─────▶│ crash reports, symbols │
│ 12 known users   │── fast fixes ───────▶│ OTA + staged rollout   │
└──────────────────┘                      └────────────────────────┘
        │                                             │
        └──── same product, five new promises ────────┘
```

The same product moves left to right by taking on five promises; each arrow is a chapter in this module, and none of them changes what the product does.

### Where it lives

This module sits on top of almost everything you already have. The API your app talks to is the one from M04, with its auth sessions and object storage. Caching and offline tolerance come from M05. The pipeline that builds and signs your app is CI/CD from M06, and M06's warning about secrets in build artefacts becomes literal here, because an app bundle is a file strangers can download and open. Crash reporting and consent are M07 made mobile. The product itself is the MVP from M11.

Two modules sit downstream. M13 takes release strategy further — canaries, feature flags, rollback as a practised drill rather than a panic. M16 uses the analytics you wire up here to work out why 1,000 people saw your listing and 40 installed.

And the money matters, because this is the first module in the program where free tiers run out. A Google Play Console developer account is a **one-time** registration fee of roughly US$25 — about ₹2,100 at the time of writing. An Apple Developer Program membership is an **annual** fee of roughly US$99 — about ₹8,400 every year. For a team of three students that is one shared Play account for the price of two weeks of lunch, and an Apple account that costs more than this entire course tier. The honest recommendation for an Indian student team: ship Android first, share one Play account per team, and treat iOS as something you do only if you have a user who genuinely needs it or a budget that genuinely exists. Everything else in this module — Expo, EAS free build quota, crash reporting, push — has a usable free tier.

### Words to own

| Term | Plain meaning | Example |
|---|---|---|
| Production | Running for strangers you cannot contact | 180 users from a WhatsApp group |
| Budget device | The cheapest phone you promise to support | ₹8,000 Android, 3 GB RAM |
| Store review | A human or automated check before release | 1–7 days on either store |
| Signing key | The secret that proves a build is yours | Lose it and you cannot update |
| Crash-free sessions | Share of app opens with no crash | 99.2% of sessions this week |
| OTA update | Shipping new JavaScript without a review | Fixing a typo in 10 minutes |
| Staged rollout | Releasing to a small share of users first | 5%, then 20%, then 100% |
| Cold start | First open with nothing warm in memory | 11 seconds on a cheap phone |
| Data safety form | Your written declaration of what you collect | Required on the Play Store |
| Time to first value | How long until the app is useful | Four seconds, not four screens |

### Tradeoffs and gotchas

**Going to the stores is a real cost, and sometimes the wrong one.** If your users reach you through a shared link, your product is text and forms, and you need no camera, no push and no offline writes, the installable PWA from M08 may genuinely be the better product. The store buys you three things: discoverability, push notifications that work reliably on Android, and the credibility of an install. If you cannot say which of those three you need, you are not ready to pay for the account.

**Production work is unglamorous and it is most of the remaining work.** Your feature list will barely grow in this module. What grows is the share of users for whom the existing features actually work.

**"It works on my phone" is the single most expensive sentence in this module.** Your phone is newer, your network is campus Wi-Fi, your account already has data in it, and your app is a development build that behaves differently from a release build. Four separate lies in one sentence.

**Do not start the store account on the last week.** Chapter 5 explains why in detail, but the headline is that account verification, a mandatory testing period for new accounts, and a review queue can easily add two weeks that no amount of coding speed removes. Open the account the day you start this module.

**Your rating is set by your worst users, not your average ones.** The people who had a good time mostly say nothing. Everything in chapters 8 and 9 exists so that you learn about the bad experience from your tooling instead of from a public review.

### Verify it

Ask an AI to plan your move from MVP to production and you will get a confident checklist in fifteen seconds. Here is how to tell whether it understood *your* situation.

**Demand that the plan name your constraints back to you.** A usable plan mentions the budget device, the network, the Play-first decision, the one-time versus annual fee difference, and the review calendar. If none of those appear, you have been handed a generic article.

**Red flag: a plan that assumes a fast connection.** If nothing in it mentions what happens on 4G at 1.2 Mbps with 300 ms latency, or what the app does with no network at all, it was written for a user in San Francisco.

**Red flag: a plan that assumes a current flagship phone.** "Users expect 60 fps and instant startup" is not a plan. Ask for the numbers it expects on a 3 GB RAM device.

**Red flag: "then just publish to the stores".** The word "just" is doing an enormous amount of hiding: account creation, identity verification, signing keys, a privacy policy at a live URL, a data safety declaration, a content rating, a testing track, and a review that can say no. If the plan is one bullet, it is wrong by about two weeks.

**Red flag: a release plan with no way back.** Ask the explicit question: "if the build I ship tomorrow crashes on launch for 30% of users, what exactly do I do, and how long does it take?" A plan that has no answer is not a production plan.

**Red flag: secrets in the app.** If any step says to put an API key, a service account or a database credential into the app's config so the app can call a service directly, stop. A shipped app bundle is a file anyone can download and unpack. This is the M06 secrets-in-build-artefacts warning, and on mobile it is not theoretical.

Write the five promises into your decision log from M01, each with a number. "Cold start under 3 seconds on a 3 GB Android phone" is a promise. "Fast" is a feeling.

## Walkthrough
1. Open your MVP on the cheapest Android phone anyone in your team or your hostel owns. Not an emulator, not your own phone. Note the make, the RAM and the price.
2. Turn Wi-Fi off so you are on real mobile data, force-quit the app, and time the first open with a stopwatch. Write the number down. This is your baseline cold start.
3. With the app open and something half-filled in a form, switch to WhatsApp, scroll for two minutes, then come back. Record exactly what you lost.
4. Turn on airplane mode and try the single most important action in your product. Describe what the user sees — a spinner, an error, or nothing.
5. Set the phone's font size to the largest setting and the display size to large. Screenshot any screen where text clips or buttons overlap.
6. Write down the five production promises from this lesson and, next to each, one sentence on how your product does today. Be unkind.
7. Pick the single promise your product fails hardest. That becomes your team's priority for this module, and the chapter that covers it is where you spend extra time.
8. As a team, decide now: Android only, or Android and iOS. Write the decision and the money behind it in your decision log, including who pays and from where.
9. Today, not later: have one team member start the Google Play Console account creation and identity verification. Note the date you started it.
10. Write a one-paragraph production definition for your product: the device you support, the network you support, the cold start you promise, and the crash-free percentage you aim for. Four numbers, no adjectives.

## Talk to AI
```prompt
CONTEXT:      I am part of a three-person student team in India. We have
              already shipped an MVP: [one sentence on what the product
              does]. Stack: [your stack]. It is deployed at [URL] and we
              also made it an installable PWA. We have about [N] real
              users. Our users are students on Android phones costing
              around Rs 8,000-12,000 with 3 GB of RAM, on patchy and
              expensive mobile data, many of whom prefer Hindi or Marathi.

              Measurements I have actually taken on the cheapest phone I
              could find, on mobile data:
              - cold start: [X] seconds
              - what happens in airplane mode on our main action: [describe]
              - what we lose when the OS kills the app in the background:
                [describe]
              - crash reports we currently receive: [none / describe]

GOAL:         A realistic plan to take this from "works for people we know"
              to "works for strangers on cheap phones", ordered by how much
              user pain each step removes per day of work.

REQUIREMENTS: 1. Audit us against five promises: runs on their device,
                 survives a bad network, is findable and installable, is
                 observable when it breaks, can be changed quickly and
                 safely. One honest verdict per promise, using my numbers
              2. Say which single promise we fail hardest and why
              3. Give a calendar, not a checklist: mark which items have
                 waiting time outside our control (store accounts, identity
                 verification, review queues, mandatory testing periods)
              4. Separate what costs money from what is free, and tell me
                 the difference between a one-time fee and an annual fee
              5. Tell me, explicitly, what you would NOT bother doing yet
                 for a product with our user count

CONSTRAINTS:  Free tiers wherever possible. Total spend under Rs 3,000 for
              the whole team. No Mac hardware available. Android is the
              priority market; iOS only if you can justify it. Three
              students with about two hours a day each.

NON-GOALS:    Do not propose new product features. Do not propose a rewrite
              in a different framework. Do not tell me to hire anyone.

ACCEPTANCE:   - Every verdict cites one of my measurements, not a guess
              - The calendar marks which days are waiting, not working
              - Costs are split into one-time and recurring, in rupees
              - You have named at least one thing we should skip
              - Nothing in your plan puts an API key inside the app bundle

ASK:          Give me the audit and the calendar first. Do not propose any
              implementation work until I have agreed which promise we are
              fixing first.
```

## Stuck? Ask AI
```prompt
Explain the difference between an MVP and a production app to someone who has never shipped software, using one everyday analogy that is not about cars or houses. Then give me six short descriptions of apps and ask me, one at a time, whether each one is an MVP or production-ready, without telling me the answers first.
```

```prompt
I have a working web app that is also an installable PWA, and users on cheap Android phones in India. Argue both sides properly: give me the three strongest reasons to publish a real app to the Play Store, and the three strongest reasons to stay a PWA. Then ask me four questions about my product whose answers would settle it, and wait for my answers before recommending anything.
```

```prompt
My team thinks production work is boring compared to building features. Give me five short true-to-life stories of student products that failed after launch for production reasons rather than feature reasons, and for each one name the single cheap thing that would have prevented it. Keep each story under 80 words.
```

## Quiz

### Q1 · L2
Which of these is a production promise rather than an MVP property?
- (a) The core idea can be demonstrated end to end
- (b) You can tell that it broke for a stranger without being told ✅
- (c) It runs on the developer's laptop
- (d) It has at least one working feature
> An MVP proves the idea once. Production means you can observe, support and fix the product for people you cannot contact, which is why crash reporting and field performance exist.

### Q2 · L3
Your team has two weeks until demo day and no store account. What is the real risk?
- (a) The code will not be finished in time
- (b) Account verification, a mandatory testing period and review queues add waiting days you cannot code away ✅
- (c) The app will be too large to upload
- (d) The store will ask for a company registration
> Store friction is calendar time outside your control. This is why chapter 5 tells you to open the account on day one of the module, not in the final week.

### Q3 · L3
A student's app is killed by Android while she is answering a WhatsApp message, and she loses a half-filled form. Which production promise failed?
- (a) Findable and installable
- (b) Runs on their device, including surviving the OS reclaiming memory ✅
- (c) Can be changed quickly and safely
- (d) None — that is normal behaviour and not your problem
> Low-memory kills are routine on 3 GB devices. Saving in-progress state locally is part of running on their device rather than yours, and it is covered in the offline and performance chapters.

### Q4 · L4
An AI gives you a migration plan whose store step reads "then just publish to the Play Store and App Store". What is the strongest objection?
- (a) It should have recommended the App Store first
- (b) The word "just" hides accounts, identity checks, signing keys, a privacy policy URL, a data safety declaration and a review that can reject you ✅
- (c) Publishing is not possible without a Mac
- (d) Store publishing is always a bad idea for students
> A one-bullet store step is wrong by roughly two weeks. Demanding that the plan separate waiting time from working time is how you catch this before it costs you the deadline.

### Q5 · L4
Why does this module warn so hard against the sentence "it works on my phone"?
- (a) Because developers are bad at testing
- (b) Because it hides four different differences at once: newer device, better network, pre-populated account, and a development rather than release build ✅
- (c) Because phones are unreliable test devices in general
- (d) Because only emulators give accurate results
> Each difference alone can hide a serious problem; together they guarantee it. Chapter 9 therefore insists that your performance numbers come from a real budget device running a real release build.

## Go deeper
- [Google Play: launch checklist](https://developer.android.com/distribute/best-practices/launch/launch-checklist)
- [web.dev: what makes a good mobile experience](https://web.dev/explore/fast)
