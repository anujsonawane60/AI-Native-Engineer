---
id: m12-08-crash-reporting-mobile
module: m12-ship-apps-for-real
slug: crash-reporting-mobile
title: "Crash Reporting and Mobile Observability"
order: 8
duration_min: 25
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

Your Play Store rating drops from 4.4 to 2.8 over nine days. The reviews are not detailed: "keeps closing", "not opening", "waste of time". One says "crashes when I press pay", which is the only useful sentence you have.

You try it on your own phone. It works. You try it on your teammate's phone. It works. You cannot reproduce anything, because the crash happens on Android 10 on a specific manufacturer's build with 3 GB of RAM when the photo picker returns something your code did not expect, and you own none of those conditions.

On your server you would have logs. On a phone you have nothing, because the phone is not your computer and nobody sends you its logs.

So you install a crash reporter, and the reports start arriving. They look like this: a list of memory addresses and a frame called `anonymous`. No file name, no line number, nothing recognisable, because the JavaScript that shipped was minified and you never uploaded the map that translates it back. Eleven days of crash data that tells you nothing except that you are crashing.

And then you fix it, you ship, and you cannot tell whether the fix worked — because nothing in your dashboard records which build each report came from.

### The idea

**On mobile you cannot read the logs, so the app has to report its own failures — and a report is only useful if it names your code, names the build, and tells you how many people it happened to.**

Those three properties are the whole chapter. Symbols turn addresses into file names and line numbers. A release tag ties each report to a specific version. And a rate — not a count — tells you whether this matters.

The number to live by is **crash-free sessions**: the percentage of app opens that ended without a crash. Counts lie, because 200 crashes is terrible for 500 users and irrelevant for 500,000. A rate is comparable week to week and across releases.

### The analogy

A canteen that only learns about problems when someone shouts at the counter learns about very few problems. Most people who get a bad plate simply do not come back and tell their friends instead.

A canteen with a complaints register placed where people leave gets the information. But a register entry saying "food bad" is as useless as a stack trace with no symbols. What you need is the date, the counter, the dish and the batch — which is a release tag, a screen name and a stack trace.

And one more thing the register must do: count how many plates were served that day. Eleven complaints out of 2,000 plates is a different fact from eleven out of 30.

**Where the analogy breaks:** a complaints register relies on a human choosing to write in it. A crash reporter fires automatically, from the user who would never have told you, which is exactly why it is worth more than all your reviews put together. The flip side is that automatic collection means you are collecting data about people who did not volunteer it — which is why the privacy half of this chapter is not optional.

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

### What you are actually watching

Three different kinds of failure, and they do not all look the same.

**A native crash** kills the app process. The user sees it close. These appear in the crash dashboard and in Google Play's own Android vitals.

**A JavaScript error** in React Native may crash the app or may just break one screen, leaving a white rectangle. These need the reporter's JavaScript integration and source maps, and they are the majority of what a React Native app produces.

**An ANR** — Application Not Responding — is Android's complaint that your app stopped answering the main thread for several seconds. The user sees a frozen app and a system dialog. ANRs are not crashes and are easy to miss entirely, and on budget phones they are common.

Google Play Console gives you **Android vitals**, which is field data from real users: crash rate, ANR rate, slow starts, excessive wakeups. This matters commercially as well as technically — exceed Google's bad-behaviour thresholds and your app can be demoted in Play Store search and recommendations. Your own crash reporter tells you *what* broke; Android vitals tells you how Google judges you for it.

### Where it lives

The reporter is a library in your app plus a step in your build pipeline. The library part is five minutes. The pipeline part — uploading the symbol files on every release, automatically — is the part teams skip and then regret, and it belongs in the CI from M06 so that nobody has to remember.

This is M07's error-tracking and observability chapter on a device instead of a server. The percentile thinking is the same; the difference is that you are measuring sessions on hardware you do not own. It also connects straight back to chapter 5: a crash reporter collects device identifiers and diagnostic data, so it must appear on your data safety declaration. And forward to chapter 6: the number you watch during a staged rollout, before widening from 5% to 20%, is the crash-free sessions rate you set up here.

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

**A crash report with no symbols is close to worthless.** This is the single most common failure in this chapter. The upload of source maps and symbol files must happen automatically in your release pipeline, tied to the exact build, or it will be skipped on the release where it mattered. Verify it by deliberately crashing a release build and checking that the dashboard shows your file name.

**No release tag means no learning.** Without a version on each report you cannot tell whether a crash is new, whether yesterday's fix worked, or whether the spike is from the 5% rollout. Tag the release and the build number, every time.

**Breadcrumbs leak personal data by default.** Breadcrumbs often capture network requests, screen names and sometimes form content. If they record an email address, a phone number, a token or a UPI id, your crash dashboard has become a store of personal data you did not plan for, and a third party now holds it. Scrub them deliberately — and remember M07's consent rules and your chapter 5 declaration.

**Free tiers have event quotas, so sample.** A crash loop can send tens of thousands of events in an hour and exhaust a free plan before you notice. Set a sampling rate for non-crash events, keep crashes at 100%, and check what your plan does when you hit the limit.

**Reports arrive late, usually on next launch.** A crash cannot be reported at the instant it happens because the app is dying. Expect a delay, especially from users who do not reopen quickly. Your dashboard is behind reality by hours.

**Development noise will drown the signal.** Crashes from your own development builds, from simulators and from your own testing will outnumber real ones at first. Separate environments so the production number stays honest.

**ANRs need their own attention.** They do not appear as crashes and they happen most on the cheap phones your users carry. Watch the ANR rate in Android vitals specifically; it is usually a sign that something heavy is running on the main thread, which is chapter 9's subject.

### Verify it

**Red flag: a crash report with no symbols.** The test is simple and you should run it on purpose. Add a deliberate crash behind a hidden gesture in a release build, trigger it on a real phone, and look at the dashboard. If you see your own file name and line number, the pipeline works. If you see hex and `anonymous`, your symbol upload is broken, no matter what the setup guide said.

**Red flag: no release tag on reports.** Open any report and look for the version. If it is missing, blank, or the same string for every build, fix that before you ship anything else, because every conclusion you draw will be unattributable.

**Red flag: counts instead of rates.** If AI reports progress as "crashes went from 400 to 150", ask for crash-free sessions. A drop in count can simply mean fewer people opened your app. The rate is the number that corresponds to a user's experience, exactly as with p95 in M05.

**Red flag: personal data in breadcrumbs.** Trigger a crash while logged in, then read your own report line by line in the dashboard. Look for an email, a phone number, an auth token or anything identifying in the request URLs. Whatever you can see, your crash provider stores.

**Red flag: a reporter installed but never tested.** A reporter you have never seen a report from is a reporter you cannot trust. The instrumentation itself must be verified on a real device before you depend on it, which is the same lesson as never trusting an AI's summary over running the check.

**Red flag: AI telling you what to do about a crash before it has the stack trace.** Paste the symbolised trace and the breadcrumbs. If the suggestion arrives without them, you are getting pattern-matching on the word "crash", not diagnosis.

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
