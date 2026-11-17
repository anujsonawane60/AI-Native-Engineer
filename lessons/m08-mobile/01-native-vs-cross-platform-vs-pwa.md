---
id: m08-01-choose-a-route
module: m08-mobile
slug: native-vs-cross-platform-vs-pwa
title: "Native vs Cross-Platform vs PWA vs Hybrid, and How to Choose"
order: 1
duration_min: 15
is_free: false
mission: false
volatility: medium
last_reviewed: 2026-11-16
---

## Key learnings
- Tell the four delivery routes apart by what each one actually costs you
- Choose a route from your persona's phone and your team's size, not from fashion
- Explain why install friction decides adoption more often than features do
- Name the three device capabilities that genuinely force you off the web
- Catch an AI that recommends a native app for a problem a URL solves

## Lesson

### The pain

A team of three spends eleven days arguing about whether their canteen page "should be an app". They settle on cross-platform, lose four days to a toolchain, then learn the App Store wants a Mac they do not have and ₹8,500 a year. At demo day the 38 MB "app" has one surviving install; the web page a teammate built in an afternoon has been used 200 times, because the link fits in a WhatsApp message. They chose the route before knowing what the product needed from the device.

### The idea

**There are four ways to put software on a phone, and the right one is decided by two questions: what does this product need from the device, and how hard is it to get installed?**

The routes: a web page, an installable web app (a PWA), a cross-platform app, and a native app written twice. The honest default for a student product in India is the web, made installable; you move off it only when a named capability forces you. Only three qualify.

- **Background work** — code that must run while your app is closed: a step counter, an alarm, a location trail.
- **Hardware the browser does not expose** — Bluetooth, NFC tap payments, raw telephony, fingerprint-gated secure storage.
- **Reliable push on iPhones** — iOS web push works only for a PWA already added to the home screen, and the rules have moved across versions.

Camera, photos, location, file upload, offline use and UPI payments all work from a modern mobile browser.

### The analogy

The canteen can reach a hungry student four ways: a **walk-up counter** where anyone arrives, nothing is installed, and the menu changes the moment the cook changes it (the web); a **membership card with the counter's exact location on it**, so regulars come straight back to the same counter (a PWA); **one small outlet per hostel block from one central kitchen** (cross-platform); or **two separate kitchens**, each built to its block's gas line and inspector, twice the cost forever (native).

**Where the analogy breaks:** no landlord reviews the canteen's menu before students read it. App stores do. Every native, cross-platform and hybrid release passes someone else's review queue, turning a ten-minute fix into a two-day wait.

### Picture it

```
  route           install      device      release     team
                  friction     access      speed       cost
  ──────────────────────────────────────────────────────────
  web page        none         low         minutes     1x
  PWA             one tap      medium      minutes     1.1x
  hybrid shell    store        high        days        1.5x
  cross-platform  store        high        days        2x
  native, twice   store        highest     days        4x
```

The four routes trade install friction and release speed against how deeply you can reach into the phone, and the cost column is roughly how much of your three-person team the route consumes.

### Where it lives

On M00's four boxes this chapter decides the shape of the **client** box only. The server, network and data boxes do not change — the same API serves a web page, a PWA and a React Native app — so the route is reversible and your backend work is never wasted. Your M01 PRD already constrains the choice: a persona holding a ₹8,000 Android with 2 GB of RAM, storage 90% full and a prepaid data pack counted in megabytes makes a 40 MB install anything but neutral.

**M08 is how to choose, plus how to ship a PWA** in chapter 5. **M12 is where you actually ship to the stores** — Expo and React Native, store submission, over-the-air updates, native device APIs. No developer account here.

### Words to own

| Term | Plain meaning | Example |
|---|---|---|
| Native | Written in the platform's own language, once per platform | Kotlin for Android |
| Cross-platform | One codebase compiled into two real apps | React Native, Flutter |
| PWA | A web app that can be installed and work offline | Chapter 5's mission |
| Hybrid / WebView | A native shell showing your web app | A Trusted Web Activity |
| Install friction | Everything between "interested" and "using it" | A 38 MB download |
| Store review | Someone else approving your release | Typically 1–3 days |
| OTA update | Shipping a change without a store review | M12 covers this |
| APK / AAB | The Android install file and the format you upload | 38 MB on a full phone |
| TWA | Trusted Web Activity: your PWA, listed on Play | Store presence, one codebase |

### Tradeoffs and gotchas

**Play Store dominance changes the maths in India.** Roughly nine in ten Indian phones run Android, so insisting on iOS parity from day one doubles your work for the minority.

**Install friction is a tax you pay on every user, forever.** A link costs a tap; an install costs a store search, a 38 MB download on a metered pack, 90 MB of storage the phone may not have, and a permission screen — each step loses people.

**App Store access costs money and a Mac.** Apple's programme is roughly ₹8,500 a year and iOS builds realistically want macOS; Google Play is a one-time ₹2,000.

**Do not pick a route to impress a jury.** In M11 and M18 you are asked *why*. "A PWA, because our persona's phone has 400 MB free and nothing we ship needs more than the camera" is a strong answer; "React Native, because apps are more professional" is not.

### Verify it

**Demand the capability list before the recommendation.** A framework named before a capability is a guess; the right first answer is a table of device features needed and which the web cannot do.

**Red flag: native for a product that is a form, a list and a payment.** That is a web app — ask which measured number improves on a 2 GB phone.

**Red flag: silence about install size and storage.** A store route needs a download size set against your persona's free storage.

**Red flag: "PWAs work everywhere, including push on iOS" with no caveat.** iOS web push needs a home-screen install first; this is the commonest factual error in the topic.

**Red flag: a plan assuming a Mac or a paid developer account.** Your constraint line says free tier — M00's missing-context failure in a new costume.

Then write "we are choosing X because our product needs Y, which the web cannot do". If you cannot fill in Y, you have your answer.

## Walkthrough
1. Open your team's PRD from M01 and copy out the persona's device line: phone price, RAM, free storage, and how they pay for data. If that line does not exist, you have found the real gap — write it today.
2. List every feature in your current scope as one line each. Beside each, write the device capability it needs: camera, location, file picker, payment handoff, notifications, background work, Bluetooth, none.
3. Mark each capability as web-capable or not. Camera, photos, location, file upload and UPI handoff are web-capable. Background work, Bluetooth, NFC and dependable iOS push are not.
4. Count the "not" rows. If the count is zero, your route is the web, and chapter 5 makes it installable. Write that sentence down before you continue.
5. If the count is not zero, write the single feature driving it and ask whether it is in your MVP or in your someday list. Features in the someday list do not get to choose your architecture.
6. Estimate install friction honestly. For a store route, write down the expected download size and your persona's free storage. For the web route, write down how a new user receives the link.
7. Price both routes in team-days, not in money: setup, build pipeline, store accounts, review waits, and who on your team of three owns it. Use real days you have, not ideal days.
8. Check your platform split. If your users are 90% Android, say what iOS gets in version one — nothing, a web page, or parity — and why.
9. Write a five-line decision entry in your decision log: the route, the capability that decided it, what you gave up, what would make you change your mind, and the date.
10. Share the entry with your two teammates and have each of them try to break it in one sentence. If neither can, you are done. If one can, the entry was incomplete, not wrong — fix it.

## Talk to AI
```prompt
CONTEXT:      I am a student on a three-person team building our first real
              product. I need to choose how it reaches a phone, and I have
              not chosen a framework yet and do not want to.

              Our product: [one sentence on what it does and for whom]
              Our features in scope now: [list them, 5-8 lines]
              Our persona's phone: [price, RAM, free storage, Android/iOS]
              How they pay for data: [prepaid pack, daily limit, Wi-Fi]
              Our team: three students, no Mac, no paid accounts,
              free tiers only, about [X] hours a week each.
              Our backend: [your stack], already serving a JSON API.

GOAL:         Decide between a plain web page, an installable PWA, a
              cross-platform app and a native app, with a reason I can
              defend in a viva.

REQUIREMENTS: 1. First produce a table: one row per feature in my scope,
                 the device capability it needs, and whether a modern
                 mobile browser can do it. No recommendation yet.
              2. Then list only the capabilities the web genuinely cannot
                 deliver on a two-year-old Android, and say for each how
                 confident you are and why.
              3. Then give me two candidate routes with tradeoffs, each
                 costed in three-person team-days including store accounts
                 and review waits, not just coding time.
              4. For each route, state the expected install size in MB and
                 compare it to my persona's free storage.
              5. State what I would have to give up on each route, and what
                 single piece of evidence would make you switch.

CONSTRAINTS:  Free tier only. No Mac, so no local iOS builds. No paid
              developer accounts this term. Our users are about 90%
              Android. Changes must be shippable the same day we write
              them. Assume patchy 4G and metered mobile data.

NON-GOALS:    Do not write any code. Do not recommend a framework by name
              until after the capability table. Do not plan a store
              submission - that is a later module.

ACCEPTANCE:   - Every feature in my list appears in the capability table
              - Each "web cannot do this" row names the specific API or
                platform limitation, not a vague "performance"
              - Both routes are costed in days with the store overhead
                shown separately from the coding
              - The recommendation ends with one sentence of the form
                "choose X because the product needs Y, which the web
                cannot do" - or says plainly that nothing forces me off
                the web

ASK:          Give me the capability table first and stop. Do not
              recommend anything until I have replied to it.
```

## Stuck? Ask AI
```prompt
Explain the difference between a native app, a cross-platform app, a hybrid WebView app and an installable PWA to someone who has only ever used a phone, never built anything. Use one everyday analogy, keep it under 300 words, and end with the single question I should ask myself to pick between them.
```

```prompt
I keep hearing that "real products are apps". Give me five real, widely used products that deliberately reach most of their users through a mobile web page rather than an installed app, and for each one say what they gave up and what they gained. Then give me three products where an installed app was genuinely unavoidable, and name the exact capability that forced it.
```

```prompt
I am on a three-person student team in India. Our users have budget Android phones with very little free storage and metered data. List every hidden cost of asking such a user to install a 40 MB app, in the order they are paid, from first hearing about the product to using it a second time. For each cost, tell me roughly what fraction of interested people it loses, and say how confident you are in that number.
```

## Quiz

### Q1 · L3
Your product is a form, a list of items, a photo upload and a UPI payment handoff. Your users are 90% on budget Android phones with little free storage. Which route is the sensible default?
- (a) Native Android and native iOS, for the best possible performance
- (b) An installable PWA on the web, since nothing in that list needs a store ✅
- (c) Cross-platform, because one codebase is always cheaper
- (d) A hybrid WebView app, so it appears in the Play Store
> Camera, file upload and a UPI handoff all work from a modern mobile browser. Nothing in the feature list forces a store route, so the route that costs no install friction wins.

### Q2 · L2
What three things does a web app need before a browser will offer to install it?
- (a) A Play Store listing, an APK and a developer account
- (b) HTTPS, a web app manifest and a registered service worker ✅
- (c) A native shell, a signing key and an icon
- (d) A paid domain, a privacy policy and push notifications
> Those three are the whole installability contract, and they are what chapter 5 has you add to your own product. No store or account is involved.

### Q3 · L4
Which of these is a genuine reason the web cannot serve your product?
- (a) You need the camera to capture a photo of a dish
- (b) You need to accept a UPI payment
- (c) You need code to keep counting steps while the app is closed ✅
- (d) You need the app to work when the network drops
> Camera, UPI handoff and offline use are all available to a modern web app. Sustained background execution is the capability the web genuinely does not give you.

### Q4 · L4
A teammate argues for React Native because "apps feel more professional than websites". What is the strongest objection?
- (a) React Native is harder to learn than HTML
- (b) It names no capability, and pays install friction on every user forever ✅
- (c) Flutter would be a better cross-platform choice
- (d) Professional feel is impossible to achieve outside native code
> A route must be justified by something the product needs from the device. Without that, you have added a store queue, a download and a storage cost in exchange for a feeling.

### Q5 · L5
Why is choosing the web route early a largely reversible decision?
- (a) Because web code can be automatically converted into a native app
- (b) Because the server, network and data boxes are unchanged by the client's shape ✅
- (c) Because app stores accept websites as submissions
- (d) Because PWAs become native apps once enough users install them
> Only the client box changes. The API you build in M04 serves a web page, a PWA or a React Native app equally, so backend work carries over if you later ship to stores in M12.

## Go deeper
- [web.dev: What makes a good Progressive Web App](https://web.dev/articles/pwa-checklist)
- [Chrome for Developers: Trusted Web Activity](https://developer.chrome.com/docs/android/trusted-web-activity)
