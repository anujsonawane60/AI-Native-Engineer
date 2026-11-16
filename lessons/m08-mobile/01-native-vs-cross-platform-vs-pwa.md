---
id: m08-01-choose-a-route
module: m08-mobile
slug: native-vs-cross-platform-vs-pwa
title: "Native vs Cross-Platform vs PWA vs Hybrid, and How to Choose"
order: 1
duration_min: 25
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

Week two of the Sprint. A team of three has a working idea — a page where hostel students see what the canteen has left before they walk over — and they spend eleven days arguing about whether it should be "an app".

They settle on a real mobile app, because that is what products are. One member spends four days on a cross-platform toolchain. Another discovers that the App Store needs a Mac they do not have and a developer account costing about ₹8,500 a year. The third gets bored of waiting and builds a web page in an afternoon.

At demo day the "app" is a 38 MB download that four people installed and one still has. The web page has been used 200 times, because the link fits in a WhatsApp message. Nobody made a bad engineering decision. They made a bad *framing* decision: they chose the route before they knew what the product needed from the device, then spent their only scarce resource — eleven days of three people — defending it.

### The idea

**There are four ways to put software on a phone, and the right one is decided by two questions: what does this product need from the device, and how hard is it to get installed?**

Everything else — language, framework, which YouTube tutorial is nicest — follows from those two answers. The four routes are a web page, an installable web app (a PWA), a cross-platform app, and a native app written once per operating system. A fifth, hybrid, is a web page wearing an app's clothes.

The honest default for a student product in India is the web, made installable. You move off it when a specific capability forces you to, and you should be able to name that capability out loud.

### The analogy

The canteen can reach a hungry student in four ways.

It can serve at the **walk-up counter**: anyone can arrive, nothing is installed, and the menu changes the moment the cook changes it. That is the web.

It can give regulars a **membership card with the counter's exact location on it**, so they come straight back without searching. Nothing new is built; the counter is the same counter. That is a PWA.

It can open **one small outlet per hostel block, from the same central kitchen**, with one delivery boy per block who knows that block's rules. That is cross-platform: one codebase, two platforms, a thin adapter each side.

Or it can build **two separate kitchens**, each designed to that block's gas line, staff and inspector. Best possible food, twice the cost, two of everything forever. That is native.

**Where the analogy breaks:** no landlord reviews the canteen's menu before students may read it. App stores do. Every native, cross-platform and hybrid release passes through a review queue owned by someone else, which turns a ten-minute fix into a two-day wait. The web has no gatekeeper, and for a team shipping weekly that is not a small detail — it is often the deciding one.

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

### What each route actually is

**Web page.** HTML, CSS and JavaScript over HTTPS, opened in Chrome. No install, no store, no review. Shareable as a link — which in India means shareable on WhatsApp, the single most effective distribution channel a student team has.

**PWA (Progressive Web App).** The same web page plus three additions: HTTPS, a manifest file describing its name and icons, and a service worker — a small script the browser keeps, which can answer requests when the network does not. Chrome on Android then offers "Install app", and afterwards it has a home-screen icon, opens without browser chrome, and can work offline. That is chapter 5.

**Cross-platform.** One codebase, usually React Native with Expo or Flutter, compiled into a real Android app and a real iOS app. You get the full device API surface and a store listing, plus two build pipelines, store accounts, review queues and a native debugging problem every few weeks.

**Native.** Kotlin for Android, Swift for iOS, written twice. Chosen when the product *is* the device integration — a camera app, a maps app, something that runs for hours in the background.

**Hybrid / WebView.** A thin native shell whose whole screen is a browser view pointed at your web app. On Android, a Trusted Web Activity is the modern form: your PWA, listed on the Play Store, with no separate UI code. It buys store presence without a second codebase, and very little extra device access.

### The three capabilities that genuinely force you off the web

Most "we need an app" arguments collapse under one question: which capability, specifically? Only a short list survives.

**Background work.** Code that must run when your app is closed — a fitness tracker counting steps all day, an alarm, a location trail. The web gets almost nothing here.

**Hardware the browser does not expose.** Bluetooth peripherals, NFC for tap payments, raw telephony, fingerprint-gated secure storage. Some of this is arriving on the web; most of it is not dependable on a two-year-old Android.

**Reliable push on iPhones.** Web push exists on iOS, but only for a PWA the user has added to their home screen, and the behaviour has moved repeatedly across versions. If notifications are the core of your product and iPhone users matter, that is a real reason to ship a native app.

Camera, photos, location, file upload, payments and offline use are all available to a modern web app. UPI works from a web page: the payment app opens via a link. None of these is a reason to leave.

### Where it lives

On M00's four boxes, this chapter is a decision about the shape of the **client** box. The server, the network and the data boxes do not change — the same API serves a web page, a PWA and a React Native app. That is worth internalising, because it makes the route a reversible decision and your backend work is never wasted.

Your choice is also already written down. In M01 your PRD took its constraints from your persona's device: if that persona has a ₹8,000 Android with 2 GB of RAM, storage already 90% full, and a prepaid data pack they count in megabytes, then a 40 MB install is not a neutral ask. The PRD decides the route; the route does not decide the PRD.

This module is the **overview**. You learn to choose, and in chapter 5 you make your own product installable. The practicum lives at the Build tier: **M12 is where you actually ship to the stores** — Expo and React Native, Play Store and App Store submission start to finish, over-the-air updates, native device APIs, crash reporting and performance budgets on real devices. Nothing in M08 asks you to create a developer account or submit anything for review.

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

**Play Store dominance changes the maths in India.** Roughly nine in ten Indian phones run Android. A team that builds Android-only is reaching most of its market; a team that insists on iOS parity from day one has doubled its work for the minority. This is the opposite of the advice written for an American audience, and it is written for your users.

**Install friction is a tax you pay on every single user, forever.** A link costs a tap. An install costs a store search, a 38 MB download on a metered data pack, 90 MB of storage the phone may not have, and a permission screen. Each step loses people. A product used twice a week rarely survives that tax; a product used daily sometimes earns it.

**App Store access costs money and a Mac.** The Apple developer programme is roughly ₹8,500 a year and building for iOS realistically wants macOS; Google Play is a one-time fee of around ₹2,000. On a free-tier course that is a genuine constraint, and another reason the web is the default here.

**"We will add a native app later" is usually true and usually fine.** The API you build in M04 does not care which client calls it. Choosing the web now is not a dead end; choosing native now and discovering nobody installs it is much closer to one.

**Do not pick a route to impress a jury.** In M11's checkpoints and M18's defence you are asked *why* you chose what you chose. "A PWA, because our persona's phone has 400 MB free and our feature list needs nothing beyond the camera" is a strong answer. "A React Native app, because apps are more professional" is not.

### Verify it

Ask any AI "should I build a mobile app?" and you get a confident recommendation in seconds. Here is how to tell whether it reasoned or pattern-matched.

**Demand the capability list before the recommendation.** The correct first response is a question or a table: which device features this product needs, and which of them the web cannot do. If it names a framework before it names a capability, it is guessing.

**Red flag: a native recommendation for a product that is a form, a list and a payment.** That is a web app. If the AI justifies native with "better performance" or "native feel", ask which measured number improves and by how much on a 2 GB phone. The honest answer is usually that a 38 MB install made things worse.

**Red flag: silence about install size and storage.** Any store route should come with an estimated download size and a sentence about the user's free storage. An answer that never mentions megabytes has not thought about your persona.

**Red flag: "PWAs work everywhere, including push on iOS" with no caveat.** iOS web push requires a home-screen install first, and the rules have changed more than once. Unqualified claims about iOS PWA support are the commonest factual error in this topic.

**Red flag: a plan that assumes a Mac or a paid developer account.** Your constraint line says free tier. If the plan quietly spends ₹8,500, it ignored the brief — M00's missing-context failure in a new costume.

Then do the cheapest check of all: write "we are choosing X because our product needs Y, which the web cannot do". If you cannot fill in Y, you have your answer.

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
