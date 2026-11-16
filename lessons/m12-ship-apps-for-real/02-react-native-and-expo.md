---
id: m12-02-react-native-expo
module: m12-ship-apps-for-real
slug: react-native-and-expo
title: "React Native and Expo: One Codebase, Two Stores"
order: 2
duration_min: 35
is_free: false
mission: true
volatility: medium
last_reviewed: 2026-11-16
---

## Key learnings
- Explain how one JavaScript codebase renders real native screens on Android and iOS
- Say what Expo adds on top of React Native, and why cloud builds matter with no Mac
- Choose between a WebView wrapper, a cross-platform app and two native rewrites
- Produce a signed Android build and install it on a real phone
- Spot an AI answer written for bare React Native when you asked for Expo

## Lesson

### The pain

Your team has a working web app. You want it in the Play Store, and you have four hours before the live session.

The first thing anyone suggests is wrapping the website in a WebView — a native app whose only screen is a browser showing your site. It takes twenty minutes. It also fails in every way that matters: it cannot work offline, it cannot use the camera properly, push notifications do not arrive, the back button behaves strangely, it feels like a website in a box because it is one, and both stores have rules against apps that are only a repackaged site. Teams get rejected for this every week.

The second suggestion is to write the app twice — Kotlin for Android, Swift for iOS. Two codebases, two sets of bugs, two sets of screens to keep in sync, and nobody on your team has a Mac, which iOS development traditionally requires.

The third option is the one this module uses, and it is the reason you can ship to both stores in a week instead of a term.

### The idea

**React Native lets you describe your screens in JavaScript, and renders them using the platform's own native views — so one codebase produces a genuine Android app and a genuine iOS app.**

When you write a `<Text>` in React Native, Android gets a real `TextView` and iOS gets a real `UILabel`. It is not a webpage pretending. Scrolling uses the platform's scroll physics, text uses the system font and respects the user's font-size setting, and the operating system treats your app as an app.

Expo sits on top of React Native and removes almost all of the setup. It gives you a ready project structure, a large library of device modules (camera, location, notifications, file system) that already work on both platforms, and — the part that changes your life as an Indian student team — **EAS Build**, a cloud service that compiles your app on Google's and Apple's toolchains for you. You do not install Xcode. You do not own a Mac. You run one command and a signed build appears.

### The analogy

Picture the canteen with one kitchen and two serving counters: a veg window and a non-veg window.

One kitchen, one set of cooks, one stock of rice and oil and gas. That is your JavaScript codebase. The two windows each have their own queue, their own signage, their own rules about what can be served and in what kind of plate. That is Android and iOS. Most dishes come out of the same kitchen and simply get plated differently at each window.

But occasionally a window genuinely needs its own dish. The non-veg window is closed on Tuesdays by college rule. The veg window has a subsidised plate that the other does not. You cannot pretend the windows are identical, so you handle those few cases per window — and you keep the kitchen shared.

**Where the analogy breaks:** a cook can look over at the other window and adapt instantly. Your code cannot. Platform differences appear at build time and at runtime in ways you will only discover by testing on both — the Android hardware back button has no iOS equivalent, iOS safe areas and the notch have no Android equivalent, and permission dialogs behave differently. "One codebase" means one place to write the logic, not one place to stop thinking.

### Picture it

```
        YOUR CODE                      EAS BUILD (cloud)
  ┌────────────────────┐        ┌───────────────────────────┐
  │ JavaScript / TSX   │──src──▶│  Android toolchain        │──▶ .aab
  │ screens, logic     │        │  (Gradle, your keystore)  │
  │ expo-camera etc.   │        ├───────────────────────────┤
  └────────────────────┘        │  macOS + Xcode toolchain  │──▶ .ipa
            │                   │  (Apple certificates)     │
            │                   └───────────────────────────┘
            ▼                                │
  ┌────────────────────┐                     ▼
  │ Metro bundles JS   │          installs on a real phone,
  │ Hermes runs it     │          then uploads to the stores
  └────────────────────┘
```

Your JavaScript goes to a cloud build service that runs both platforms' native toolchains and returns one Android bundle and one iOS archive, neither of which required you to own the matching computer.

### Where it lives

The mobile app is a new client, not a new product. It talks to the same API from M04, with the same sessions and the same object storage. Your web app stays exactly where it is — many teams keep the web app as the admin surface and ship mobile only for the student-facing flows.

Practically, most teams put the app in the same git repository as the web app, in a folder like `apps/mobile`, so one pull request can change the API and both clients together. That is the M01 repository discipline applied to a second client. CI from M06 is what runs the build.

Three Expo words you will meet immediately, and must not confuse:

**Expo Go** is a pre-built app from the store that can run your JavaScript during development. Convenient, and limited: it only contains the native modules Expo chose to include.

**A development build** (sometimes called a dev client) is *your own* app, built once by EAS, containing *your* native modules, with your JavaScript loaded live. This is what you will actually use, because the moment you add anything Expo Go does not bundle, Expo Go stops being able to run your app.

**A production build** is the signed artefact you upload to a store. On Android that is an `.aab` — an Android App Bundle, from which Google generates the right-sized download for each device. On iOS it is an `.ipa`.

### Words to own

| Term | Plain meaning | Example |
|---|---|---|
| React Native | JavaScript that renders real native views | `<Text>` becomes a `TextView` |
| Expo | Tooling and modules layered on React Native | `expo-camera`, EAS Build |
| EAS Build | Cloud service that compiles your app | Builds iOS without a Mac |
| WebView wrapper | A native shell showing your website | Usually rejected, always feels wrong |
| Development build | Your own app with live JavaScript | What you test on daily |
| `app.json` | Your app's configuration file | Name, icon, permissions, version |
| Package name / bundle id | The app's permanent global identity | `com.yourteam.tiffin` |
| AAB / IPA | The signed upload artefacts | `.aab` for Play, `.ipa` for Apple |
| Metro | The bundler that packs your JavaScript | Rebuilds on every save |
| Hermes | The JavaScript engine inside the app | Faster start, smaller memory |
| Config plugin | A recipe that edits native project files | Adds a permission string for you |

### Tradeoffs and gotchas

**Pick your package name once and never again.** `com.yourteam.yourapp` is permanent on both stores. You cannot rename it after publishing; you can only publish a new app and lose every install. Decide it as a team, write it in your decision log, and check that nobody has typed `com.example.myapp` and shipped it.

**App size is bigger than you expect.** A trivial Expo app downloads at roughly 25–40 MB on Android. On a ₹8,000 phone with 12 GB of free storage and metered data, that is a real cost your users pay before seeing anything. Chapter 9 makes this a budget with a number.

**Not every npm package works.** Anything that touches the device needs a native side. If a library has no Expo support and no config plugin, adding it means a new build at minimum and sometimes is not possible at all. Check this *before* you design a feature around a library.

**Expo Go will mislead you.** It works beautifully until it silently cannot, and then the error messages are confusing. Move to a development build on day one and you skip a whole category of lost evening.

**iOS still costs money even with EAS.** EAS runs the Mac for you, but Apple will not let anyone install your iOS build — even on your own phone, even for testing — without an Apple Developer Program membership at roughly ₹8,400 a year. Android has no such barrier for local testing: a `.apk` from EAS installs on any phone for free. This is the single strongest practical reason to ship Android first.

**There is a point where React Native is the wrong answer.** A product whose core is heavy real-time graphics, serious audio processing, or deep integration with one platform's hardware will fight you. You made that decision in M08's first chapter; this chapter assumes you chose cross-platform and are now executing it.

### Verify it

AI is very good at producing React Native code and very bad at noticing which React Native you meant. Four checks.

**Check it is Expo, not bare React Native.** The two have different setup, different commands and incompatible instructions. If the answer tells you to run `react-native link`, edit `Podfile`, open `android/app/build.gradle` by hand, or install CocoaPods, it has answered for bare React Native and following it will break your Expo project. Ask explicitly: "is every step in this valid for a managed Expo project?"

**Red flag: it assumes you have a Mac.** Any instruction involving opening Xcode, running a simulator, or `pod install` assumes hardware you do not have. The correct Expo answer routes that work through EAS Build. If AI never mentions EAS, it has not registered your constraint.

**Red flag: secrets in `app.json`.** The `extra` block in your config is bundled into the app and shipped to users. An app bundle can be downloaded and unzipped by anyone. If AI puts a service key, a database URL with credentials, or a third-party API secret in there, that secret is public the moment you publish. Only values you would print on a poster go in the app; everything else lives behind your own API.

**Red flag: a WebView in the proposal.** If the "cross-platform app" it designs is a `WebView` pointed at your site, you have been handed the rejected option with extra steps. Ask it to state, in one sentence, what the app does when there is no network. A WebView answers "nothing".

**Verify on hardware, not on an emulator.** The emulator runs on your laptop's CPU and lies about startup time, memory and scroll smoothness. Install the real build on the cheapest phone you can find and open it. That is the only verification that counts.

## Walkthrough
1. As a team, agree your app name and your package name in the form `com.<yourteam>.<yourapp>`. Write both in your decision log. This is permanent.
2. Create a new Expo app in your existing repository under a folder like `apps/mobile`, so the web app and the mobile app share one history and one API contract.
3. Open the generated `app.json` and read every line. Set the name, slug, icon, splash screen, Android package name and iOS bundle identifier yourself rather than leaving the defaults.
4. Install the Expo CLI tooling and create an Expo account. Log in from your terminal so EAS can build on your behalf.
5. Configure EAS for two build profiles: a development build you install on your own phone, and a production build you will later upload to a store.
6. Run the development build for Android. It compiles in the cloud, which takes several minutes on the free queue. Download the resulting file and install it on a real phone.
7. Start the dev server and connect the phone to it. Change one word of text in a screen and watch it appear on the phone without rebuilding. This is the loop you will work in all module.
8. Replace the starter screen with one real screen from your product — the list or feed your users see first — talking to your actual API from M04.
9. Test the Android hardware back button on that screen and fix what it does. Then check the screen with the phone's font size set to maximum and note anything that clips.
10. Produce a production Android build as an `.aab`, and separately an `.apk` you can install directly. Note the download size of the `.apk` and write it in your notes as your starting app-size number.
11. Commit everything except secrets, open a pull request as in M01, and have a teammate review the `app.json` specifically for anything that should not ship to users.

## Talk to AI
```prompt
CONTEXT:      Three-person student team in India. We have a working web app
              and an API we control. Product: [one sentence]. Stack:
              [your web stack]. API base URL: [yours]. Auth: [how sessions
              work today].
              We are now building a mobile app with React Native using the
              MANAGED Expo workflow. Nobody on the team owns a Mac. We will
              ship Android first. Target device: Android phones around
              Rs 8,000 with 3 GB RAM, on patchy mobile data.
              Agreed package name: com.[team].[app]

GOAL:         A scaffolded Expo app that shows one real screen from our
              product, built in the cloud and installed on a real Android
              phone, with no secrets inside the app bundle.

REQUIREMENTS: 1. A project structure for an Expo app living inside our
                 existing repository alongside the web app
              2. A complete app.json / app config, with every field
                 explained in one line, including package name, bundle
                 identifier, icon, splash and version fields
              3. Two EAS build profiles: a development build for our own
                 phones, and a production build that produces an .aab
              4. One screen that fetches a list from our existing API and
                 renders it, handling loading, empty and error states
              5. Correct handling of the Android hardware back button on
                 that screen
              6. A written list of exactly which values are safe to put in
                 the app config and which must stay server-side, and why

CONSTRAINTS:  Managed Expo workflow only. No Mac, no Xcode, no CocoaPods,
              no manual edits to android/ or ios/ folders. Free tiers only,
              including the EAS free build queue. Must work on Android 10
              and newer. Assume slow networks with 300 ms round trips.

NON-GOALS:    Do not use a WebView to display our website. Do not add
              navigation libraries, state libraries or UI kits beyond what
              this one screen needs. Do not touch our web app. Do not set
              up push notifications or offline storage yet.

ACCEPTANCE:   - Every command you give works on Windows with no Mac
              - No step tells me to run pod install or open Xcode
              - No secret, key or credential appears in any app config
              - The screen renders something useful with an empty list and
                with the network switched off
              - You have told me the expected .apk download size and what
                drives it

ASK:          First compare two options for where the mobile app lives in
              our repository and how we share types with the web app, with
              tradeoffs, and say which you would choose for a three-person
              team. No code until I pick one.
```

## Mission

**Objective.** Your own startup's product has a real, installed Android app on a real phone: one genuine screen, talking to your real API, built in the cloud and signed.

**Steps.**
1. **Frame** — write one sentence naming the single screen of your product that must exist first, and why that one. Say which of your real users will see it.
2. **Decide** — as a team, agree and record the app name, the package name, the minimum Android version you support and the target device. Four lines in your decision log.
3. **Brief** — write a Director's Brief from the M00 template for scaffolding the Expo app. Your CONSTRAINTS must include "managed Expo, no Mac" and your NON-GOALS must include "no WebView".
4. **Direct** — have AI scaffold the project, the config and the one screen. Keep scope to that. Read `app.json` line by line yourself before accepting it.
5. **Build** — produce a development build through EAS and install it on a real phone. Then produce a production `.aab` and an installable `.apk`.
6. **Verify** — on the real phone: open the screen on mobile data, with the network off, with an empty list, with the font size at maximum, and with the hardware back button. Record what happened in each case.
7. **Measure** — note the `.apk` download size and the time from tapping the icon to the screen being usable. These are your baseline numbers for chapter 9.
8. **Review** — open a pull request. A teammate must review the app config specifically for anything that should not be shipped to users, and say so in the PR.
9. **Document** — one decision-log entry explaining why cross-platform rather than a WebView or two native apps, in your own words.

**Upload.**
- Your Director's Brief → post caption or an attached file
- The AI conversation showing the two repository options and your choice → share link in `ai_chat_url`
- Proof it runs → a photo or screen recording of the app open on a real phone, with the phone's make visible
- Your five verification results, one line each → screenshots or text
- Your two baseline numbers: `.apk` size and time to usable screen
- The pull request link and your teammate's config review comment
- Reflection, 150 words max → one tradeoff you chose, one thing the AI got wrong

**Self-check.** Tick every box before you submit.
- [ ] Our package name is ours, recorded, and contains no `example` or `myapp`
- [ ] No secret, key or credential appears anywhere in the app config
- [ ] Not one step I followed required a Mac, Xcode or CocoaPods
- [ ] The app is installed on a real phone, not only an emulator
- [ ] The screen shows something sensible with the network switched off
- [ ] The screen shows something sensible when the list is empty
- [ ] The Android hardware back button does something I intended
- [ ] I wrote down the `.apk` download size in megabytes
- [ ] A teammate reviewed the config and said what they checked
- [ ] My reflection names something specific the AI got wrong

**Stretch.** Add a second screen and real navigation between them, and check that the hardware back button walks the stack correctly rather than closing the app.

## Stuck? Ask AI
```prompt
Explain to someone who has only built websites what actually happens when React Native renders a screen: what the JavaScript does, what the native side does, and why this is different from showing a website inside an app. Use one analogy that is not about translation or bridges, keep it under 300 words, then ask me three questions to check I understood.
```

```prompt
Here is an instruction list someone gave me for a React Native setup: [paste it]. Tell me, step by step, whether each instruction is for a managed Expo project or for bare React Native, and which ones would break my Expo project if I followed them. Do not rewrite the list until you have classified every step.
```

```prompt
My Expo cloud build failed. Here is the error output: [paste it]. Explain in plain language what stage of the build failed and what the message actually means. Then give me the three most likely causes in order of probability, and for each one the single thing I should check first. Do not change my code yet.
```

## Quiz

### Q1 · L2
What does React Native actually produce on screen?
- (a) A web page rendered inside a native shell
- (b) Real platform views, so `<Text>` becomes an Android `TextView` or an iOS `UILabel` ✅
- (c) An image of your interface drawn by JavaScript
- (d) A Flutter widget tree
> This is the whole point of React Native rather than a WebView: the operating system renders your interface with its own components, so scrolling, fonts and font-size settings behave natively.

### Q2 · L3
Your team has no Mac and needs an iOS build. What does Expo's EAS Build change about that problem?
- (a) Nothing — iOS builds always require local Mac hardware
- (b) It runs the macOS and Xcode toolchain in the cloud and returns a signed `.ipa` ✅
- (c) It converts the iOS app into an Android app automatically
- (d) It removes the need for an Apple Developer account
> EAS removes the hardware requirement, not the account requirement. You still need the paid Apple Developer Program membership before anyone, including you, can install the build.

### Q3 · L4
An AI gives you setup steps that include running `pod install` and editing `android/app/build.gradle` by hand. What has gone wrong?
- (a) Nothing; those are normal Expo steps
- (b) It answered for bare React Native, and those steps assume a Mac and will break a managed Expo project ✅
- (c) It is using an old version of Android Studio
- (d) It confused React Native with React
> Bare and managed workflows have incompatible instructions. Asking "is every step valid for a managed Expo project?" catches this before you have broken your setup.

### Q4 · L4
Which value is safe to put in the `extra` block of your app config?
- (a) Your database connection string
- (b) Your payment provider's secret key
- (c) Your own API's public base URL ✅
- (d) The service account JSON your server uses
> An app bundle is a file strangers can download and unpack, so everything in it is public. Only values you would print on a poster belong there; the rest stays behind your own API, as in M06.

### Q5 · L5
A teammate proposes shipping a native app that is a single WebView pointing at your existing site, to save time. What is the strongest technical objection, as opposed to a policy one?
- (a) WebViews cannot display modern CSS
- (b) It cannot work offline, cannot use device APIs properly, and inherits none of the platform's input behaviour — so it fails exactly the production promises this module exists for ✅
- (c) WebView apps are always slower than the same site in a browser
- (d) WebViews are not supported on Android 10 and above
> Store rules against repackaged sites are real, but the deeper problem is that a WebView gives you none of the offline, device-API or platform-behaviour benefits that were the reason to build an app at all.

## Go deeper
- [Expo: create your first app](https://docs.expo.dev/tutorial/introduction/)
- [Android developers: about Android App Bundles](https://developer.android.com/guide/app-bundle)
