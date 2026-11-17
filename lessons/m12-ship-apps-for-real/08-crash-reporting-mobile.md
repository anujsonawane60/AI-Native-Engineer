---
id: m12-08-crash-reporting-mobile
module: m12-ship-apps-for-real
slug: crash-reporting-mobile
title: "Crash Reporting and Mobile Observability"
order: 8
duration_min: 15
is_free: false
mission: false
volatility: medium
last_reviewed: 2026-11-16
---

## Key learnings
- Learn about a crash from your tooling instead of from a one-star review
- Upload source maps so a stack trace names your code instead of hex
- Tag every report with a release so you know which build is broken
- Watch crash-free sessions and ANR rate as your two mobile health numbers
- Keep personal data out of crash reports while keeping them useful

## Lesson

### The pain

Your Play Store rating drops from 4.4 to 2.8 over nine days. The reviews say "keeps closing", "not opening", "waste of time". One says "crashes when I press pay" — the only useful sentence you have. It works on your phone and your teammate's, because the crash happens on Android 10 on a particular manufacturer's build with 3 GB of RAM when the photo picker returns something your code did not expect. On a server you would have logs; on a phone you have nothing.

So you install a crash reporter, and the reports arrive as memory addresses and a frame called `anonymous`, because the JavaScript that shipped was minified and you never uploaded the map that translates it back. Then you fix it, ship, and cannot tell whether the fix worked, because nothing records which build each report came from.

### The idea

**On mobile you cannot read the logs, so the app has to report its own failures — and a report is only useful if it names your code, names the build, and tells you how many people it happened to.**

Symbols turn addresses into file names and line numbers. A release tag ties each report to a version. And a rate, not a count, tells you whether it matters.

The number to live by is **crash-free sessions**: the percentage of app opens that ended without a crash. Counts lie — 200 crashes is terrible for 500 users and irrelevant for 500,000 — while a rate is comparable week to week and across releases.

### The analogy

A canteen that only hears about problems when someone shouts at the counter hears about very few; most people with a bad plate just never come back. A complaints register by the exit gets the information — but an entry saying "food bad" is as useless as a stack trace with no symbols. You need the date, the counter, the dish and the batch. And the register must count plates served, because eleven complaints out of 2,000 is a different fact from eleven out of 30.

**Where the analogy breaks:** a register relies on a human choosing to write in it. A crash reporter fires automatically, from the user who would never have told you — which is why it beats all your reviews put together, and why the privacy half of this chapter is not optional.

### Picture it

```
  app crashes on a user's phone
            │
            ▼
 ┌────────────────────────┐   stack trace is minified:
 │ crash reporter SDK     │   0x4a21, anonymous, 0x8f03
 │ captures + breadcrumbs │
 └───────────┬────────────┘
             │ sends on next launch
             ▼
 ┌────────────────────────┐        ┌──────────────────────┐
 │ crash dashboard        │◀─maps──│ build pipeline       │
 │                        │  dSYM  │ uploads symbols for  │
 │ CheckoutScreen.tsx:84  │ mapping│ every release        │
 │ release 1.4.2, 340 hits│        └──────────────────────┘
 └────────────────────────┘
```

The phone sends an unreadable trace; the dashboard only turns it into a file name and a line number because your build pipeline uploaded the matching symbol files for that exact release.

### Where it lives

Three kinds of failure, which do not look the same:

- **A native crash** kills the app process; the user sees it close.
- **A JavaScript error** may crash the app or just break one screen, leaving a white rectangle. These need source maps and are the majority of what a React Native app produces.
- **An ANR** (Application Not Responding) is Android's complaint that your app stopped answering the main thread for seconds. Not a crash, easy to miss, common on budget phones.

Google Play Console also gives you **Android vitals**: field data from real users — crash rate, ANR rate, slow starts, excessive wakeups. Exceed Google's bad-behaviour thresholds and your app can be demoted in Play Store search. Your reporter tells you *what* broke; Android vitals tells you how Google judges you for it.

The reporter is a library plus a step in your build pipeline. The library takes five minutes; uploading symbol files automatically on every release is what teams skip and then regret, so it belongs in M06's CI. A crash reporter collects device identifiers, so it must appear on chapter 5's data safety declaration, and crash-free sessions is the number you watch during chapter 6's staged rollout.

### Words to own

| Term | Plain meaning | Example |
|---|---|---|
| Crash | The app process dies | App closes on its own |
| JavaScript error | Error in your JS, may break one screen | A white screen on checkout |
| ANR | App stopped answering for seconds | Frozen list, system dialog |
| Crash-free sessions | Share of app opens with no crash | 99.3% this week |
| Source map | Translates minified JS to your files | `CheckoutScreen.tsx:84` |
| Mapping file | Android's equivalent for native code | ProGuard/R8 output |
| dSYM | Apple's symbol file for native crashes | Uploaded per iOS build |
| Release tag | Which build a report came from | `1.4.2 (build 23)` |
| Breadcrumb | Recent actions before the crash | "opened cart", "tapped pay" |
| Android vitals | Google's field data on your app | Crash rate, ANR rate |
| Sampling | Recording only a share of events | 20% of sessions, free tier |

### Tradeoffs and gotchas

**A crash report with no symbols is close to worthless.** The upload of source maps and symbol files must happen automatically in your release pipeline, tied to the exact build, or it will be skipped on the release where it mattered.

**No release tag means no learning.** Without a version on each report you cannot tell whether a crash is new, whether yesterday's fix worked, or whether the spike came from the 5% rollout.

**Breadcrumbs leak personal data by default.** They often capture network requests, screen names and sometimes form content. An email, a phone number, a token or a UPI id in there makes your crash dashboard a store of personal data held by a third party. Scrub them deliberately.

**Free tiers have event quotas.** A crash loop can send tens of thousands of events in an hour: sample non-crash events, keep crashes at 100%.

**Reports arrive late, usually on next launch**, so your dashboard is behind reality by hours. Development and simulator crashes will also outnumber real ones at first: separate the environments.

**ANRs need their own attention**, because they never appear as crashes. An ANR usually means something heavy is on the main thread — chapter 9's subject.

### Verify it

**Red flag: a crash report with no symbols.** Add a deliberate crash behind a hidden gesture in a release build, trigger it on a real phone, and look at the dashboard. Your own file name and line number means the pipeline works; hex and `anonymous` means the symbol upload is broken, whatever the setup guide said.

**Red flag: no release tag on reports.** If the version is missing, blank or identical for every build, fix that first — every conclusion you draw will be unattributable.

**Red flag: counts instead of rates.** If AI reports "crashes went from 400 to 150", ask for crash-free sessions. A drop in count can just mean fewer people opened your app. The rate is what corresponds to a user's experience, as with p95 in M05.

**Red flag: personal data in breadcrumbs.** Trigger a crash while logged in, then read your own report line by line. Whatever you can see, your provider stores.

**Red flag: AI telling you what to do about a crash before it has the stack trace.** Paste the symbolised trace and the breadcrumbs. Without them you are getting pattern-matching on the word "crash", not diagnosis.

## Walkthrough
1. Add a crash reporting library to your Expo app and configure separate environments so your development crashes never mix with production.
2. Set the release tag and build number on every report, taken from your app's own version fields, not typed in by hand.
3. Add the symbol upload to your release pipeline in CI, so source maps and the Android mapping file upload automatically for every build.
4. Hide a deliberate crash behind an unlikely gesture — five taps on the version label in your settings screen is a good one — in a release build only.
5. Install that release build on a real budget phone, trigger the crash, reopen the app so the report sends, and open your dashboard.
6. Confirm the report shows your actual file name and line number. If it shows hex addresses, stop and fix the symbol upload before doing anything else.
7. Confirm the report shows the correct release and build number. Then make a new build with a bumped version and confirm the new report is tagged differently.
8. Read one full report line by line, including breadcrumbs and network entries. Write down every piece of personal data you can see, then configure scrubbing until none remain.
9. Find your crash-free sessions number and write it in your decision log as today's baseline. Set the threshold at which you would halt a rollout.
10. Open Android vitals in the Play Console and record your crash rate and ANR rate from real users. Note anything flagged against Google's thresholds.
11. Add your crash reporter and the data it collects to your privacy policy and your data safety notes from chapter 5.

## Talk to AI
```prompt
CONTEXT:      React Native app built with Expo, Android first, live in
              [production / closed testing] with about [N] users on budget
              Android phones in India. Product: [one sentence].
              Our build pipeline: [describe, e.g. EAS Build triggered from
              CI on a tag].
              Crash reporting today: [none / which library].
              Our users log in, and our app handles [list any sensitive
              data: phone numbers, addresses, payment references].
              Store reviews mentioning crashes: [paste any]

GOAL:         Learn about every crash from our dashboard before any user
              tells us, with reports that name our own code and our own
              release, and without collecting personal data we should not
              hold.

REQUIREMENTS: 1. Set up crash reporting for both native crashes and
                 JavaScript errors, with development and production kept
                 separate
              2. Automate the upload of source maps and the Android mapping
                 file in our release pipeline, and tell me exactly how to
                 prove it worked rather than assume it
              3. Tag every report with release and build number taken from
                 our app config automatically
              4. Give me a safe deliberate-crash mechanism for a release
                 build that a real user could never trigger by accident
              5. List every field the reporter captures by default, mark
                 which could contain personal data, and give me the exact
                 scrubbing configuration
              6. Tell me which numbers to watch, where to find them, and
                 the threshold at which we halt a staged rollout
              7. Explain what ANRs are, why they will not appear as
                 crashes, and where I read our ANR rate
              8. Tell me the free tier event quota and what happens when a
                 crash loop exhausts it

CONSTRAINTS:  Free tier only. Expo managed workflow. Our users are on cheap
              Android phones, so assume ANRs and low-memory kills are
              common. We must be able to declare whatever this collects on
              a store data safety form.

NON-GOALS:    Do not add a full analytics product. Do not add session
              replay. Do not change our build pipeline beyond adding the
              symbol upload step.

ACCEPTANCE:   - Your instructions include a test that proves symbols work,
                performed on a real device
              - Every report carries a release and build number
              - You have listed every captured field and flagged the ones
                that can contain personal data
              - The metric you tell me to watch is a rate, not a count
              - You have named the exact threshold for halting a rollout
              - Every category of data collected is listed in a form I
                could copy onto a data safety declaration

ASK:          First tell me the complete list of data the reporter would
              collect by default and which parts I should scrub. Do not
              write any configuration until I have approved that list.
```

## Stuck? Ask AI
```prompt
Explain why a crash report from a released mobile app shows memory addresses instead of my file names, and what a source map and a symbol file actually do. Use a plain analogy, then ask me to explain back what would happen if I uploaded the symbols from the wrong build.
```

```prompt
Here is a symbolised crash report from my app, including the breadcrumbs: [paste it]. Tell me what the user was doing, which line is actually failing, and the three most likely causes in order of probability. For each cause, tell me the one thing I should check first. Do not change any code yet.
```

```prompt
Explain the difference between a crash, a JavaScript error and an ANR on Android, with one concrete example of each from a typical student app. Then tell me, for each, where I would see it and what number measures it. Ask me which of the three I am least likely to notice and why.
```

## Quiz

### Q1 · L3
Why is "crash-free sessions" a better health number than a count of crashes?
- (a) It is easier to calculate
- (b) A count depends on how many people opened the app, so it is not comparable across weeks or releases ✅
- (c) Counts exclude JavaScript errors
- (d) Google Play only accepts rates
> 200 crashes is catastrophic for 500 users and irrelevant for 500,000. A rate corresponds to a user's actual chance of a bad experience, which is the same reason M05 insists on percentiles over averages.

### Q2 · L4
Your crash dashboard shows memory addresses and frames called `anonymous`. What is missing?
- (a) The crash reporter library
- (b) The source maps and symbol files for that exact release ✅
- (c) Permission from the user to collect crashes
- (d) An internet connection on the user's device
> The shipped JavaScript is minified and native code is stripped. Without the matching symbols uploaded for that build, the trace cannot be translated back into your file names and line numbers.

### Q3 · L3
Reports arrive but none of them carries a version string. What can you no longer do?
- (a) Nothing important; the stack trace is enough
- (b) Tell whether a crash is new, whether your fix worked, or whether the spike came from the staged rollout ✅
- (c) See the user's device model
- (d) Receive JavaScript errors
> Attribution to a build is what turns a stream of reports into learning. It is also what makes a staged rollout decision possible, since you need to know which cohort is crashing.

### Q4 · L4
You trigger a crash while logged in and read the report. The breadcrumbs include the user's email and an auth token in a request URL. What should you do?
- (a) Nothing; crash data is private to your team
- (b) Configure scrubbing so those fields never leave the device, and declare whatever remains ✅
- (c) Delete that single report
- (d) Stop using breadcrumbs entirely
> Whatever appears in your dashboard is held by a third party. Scrubbing deliberately is the fix; the data that legitimately remains still has to appear on your privacy policy and data safety declaration.

### Q5 · L5
Your app's ANR rate is high but your crash-free sessions number looks healthy. What does this most likely mean?
- (a) Your crash reporter is broken
- (b) Something heavy is blocking the main thread, so users see a frozen app without the process ever dying ✅
- (c) Users are force-quitting the app deliberately
- (d) Android is miscounting sessions on budget devices
> ANRs are not crashes and will not appear in crash-free sessions at all. They are most common on the cheap phones your users carry, and the usual cause is work on the main thread — which is chapter 9's territory.

## Go deeper
- [Android developers: Android vitals overview](https://developer.android.com/topic/performance/vitals)
- [Expo: using Sentry for error reporting](https://docs.expo.dev/guides/using-sentry/)
