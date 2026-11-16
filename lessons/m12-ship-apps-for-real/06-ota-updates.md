---
id: m12-06-ota-updates
module: m12-ship-apps-for-real
slug: ota-updates
title: "OTA Updates and Shipping Without a Review Queue"
order: 6
duration_min: 25
is_free: false
mission: false
volatility: medium
last_reviewed: 2026-11-16
---

## Key learnings
- Draw the line between what you can update over the air and what needs a store build
- Ship a JavaScript fix to installed users in minutes instead of days
- Keep your API working for users who are still on an older bundle
- Roll an update back when it turns out to be worse than the bug
- Spot an AI claim that something can be shipped over the air when it cannot

## Lesson

### The pain

Friday evening. A user messages you: tapping "Pay" on the checkout screen crashes the app for everyone on Android. Your crash dashboard confirms it — 340 crashes in two hours, all on the same line.

You find the bug in four minutes. It is one incorrect property name in your JavaScript. You fix it, you build, you upload to the Play Store, and then you wait. Review takes a day and a half, and after it is approved the update still has to actually reach devices, which depends on each user's auto-update setting and whether they are on Wi-Fi. By Monday afternoon some users are fixed and some are not. Your weekend was spent watching a graph.

Now the other failure. A different team discovers over-the-air updates, loves them, and starts pushing changes straight to users several times a day with no review of their own. One Tuesday they push an update that calls a new API endpoint which has not been deployed yet. Every user who receives the update is broken, and the update mechanism itself is now the thing that broke them. They have no rollback plan, so they spend an hour panicking before anyone thinks to re-publish the previous bundle.

### The idea

**Your app is two things in one file: native code that only a store build can change, and JavaScript that you can replace over the air in minutes.**

The whole of this chapter is learning exactly where that line sits, because everything useful and everything dangerous about over-the-air updates comes from it.

Your screens, your logic, your text, your styling, your API calls — JavaScript. Updatable in minutes, no review queue. The camera module you installed, a new permission in the manifest, the app icon, the app name, the Expo SDK version, anything that changed a native file — native. Store build, review queue, days.

With Expo this mechanism is EAS Update: your app checks a URL on launch, and if a newer compatible JavaScript bundle exists it downloads it and runs it on the next start.

### The analogy

The canteen can change its menu card in thirty seconds. New prices, a new dish name, the samosa marked sold out — print it, clip it to the board, done. Nobody inspects a menu card.

Rebuilding the kitchen to add a tandoor is a different matter entirely. That needs a licence amendment, an inspection, and a shutdown. You cannot slip a tandoor in by changing the menu card, and a menu card that *advertises* tandoori food you cannot cook is worse than no card at all.

**Where the analogy breaks** in a way you must plan around. A menu card change is instant and universal: everyone standing at the board sees the new card. An over-the-air update lands on each phone whenever that phone next opens the app — which for some users is in ten seconds and for others is in three weeks. For a period of time your users are running two or three different versions of your JavaScript against one API. The canteen never has customers ordering from last month's card. You will.

### Picture it

```
  STORE BUILD (days)             OTA UPDATE (minutes)
┌──────────────────────┐       ┌──────────────────────┐
│ native modules       │       │ your screens         │
│ permissions          │       │ your logic           │
│ app icon and name    │       │ your text and styles │
│ SDK version          │       │ API call shapes      │
│ app signing          │       │ bug fixes in JS      │
└──────────┬───────────┘       └──────────┬───────────┘
           │ review queue                 │ publish
           ▼                              ▼
   ┌───────────────┐              ┌────────────────┐
   │ Play / Apple  │              │ update server  │
   └───────┬───────┘              └───────┬────────┘
           │ auto-update, days            │ next app launch
           └──────────────▶ PHONE ◀───────┘
```

Anything that changes a native file goes the slow way through a store; anything that is only JavaScript goes the fast way straight to the phone on its next launch.

### Where it lives

An over-the-air update is a second deploy pipeline sitting beside the one you built in M06. It has its own channels, usually one per release lane: a `preview` channel your team runs, and a `production` channel real users receive. The channel a build listens to is baked in at build time, which is why the build and the update are linked rather than independent.

The compatibility link is the **runtime version**. Every build declares one, and an update is only delivered to builds with a matching runtime version. This is the safety mechanism that stops you from shipping JavaScript that calls a native module the installed app does not contain. When you add a native module, you change the runtime version, you make a new store build, and your old installs simply stop receiving updates — correctly.

Downstream, M13 takes this much further: feature flags, canaries and rollback as a rehearsed drill. For now you need the beginner version of all three.

### Words to own

| Term | Plain meaning | Example |
|---|---|---|
| OTA update | New JavaScript without a store release | A one-line fix in 10 minutes |
| JS/native boundary | What can and cannot be updated OTA | Styles yes, new permission no |
| Runtime version | Compatibility stamp on a build | Update only reaches matching builds |
| Channel | A named lane updates are published to | `preview`, `production` |
| Embedded bundle | The JavaScript shipped inside the build | What a fresh install runs first |
| Rollback | Re-publishing the previous good bundle | Your undo button |
| Version skew | Users on several bundles at once | Three JS versions, one API |
| Staged rollout | Giving the update to some users first | Watch crashes, then widen |
| Auto-update setting | Whether the store updates for the user | Many users have it off |

### Tradeoffs and gotchas

**You cannot push a permission, a native module or an icon over the air.** This is the single most common mistake. If an AI tells you a new camera feature can go out as an OTA update, it is wrong — the camera module is native. The runtime version mechanism will usually protect you by refusing to deliver the update, which is good, but the plan was still wrong and you have lost a day.

**Your API must keep working for old bundles.** Because updates arrive whenever each user next opens the app, you always have users on older JavaScript. So your API changes must be additive: add fields, do not rename them; add endpoints, do not delete them; keep old shapes working for a few weeks. Shipping a JavaScript update and an API change simultaneously, with the API change going first, is how teams break every user who has not updated yet.

**Users pay for the bundle download.** An update is a real download on a real metered connection. Keep bundles small, do not publish eleven times a day, and understand that each publish costs your users data. This matters far more in Nashik than in Nebraska.

**Store policies allow JavaScript updates but not a change of purpose.** Both stores permit code updates to interpreted code within an app. What they do not permit is using that mechanism to turn a reviewed app into a different app — adding a store you did not declare, enabling features review rejected, or changing what the app is for. Fix bugs and improve what you declared. Do not use OTA to go around a reviewer.

**An OTA update needs the same review as a store build — your own.** The whole danger of a fast mechanism is that it removes the friction that was accidentally protecting you. Keep the discipline you built in M01: a pull request, a teammate reading the diff, and a check on a real device before publishing. The speed should be there for emergencies, not for skipping review.

**Rollback is not automatic.** Know, before you need it, exactly how you re-publish the previous bundle, and how long it takes to reach a user. Write the command and the expected time in your runbook. The day you need it, you will not be in a state to work it out.

**Updates apply on next launch, not instantly.** The user who is in the app right now when you publish is still on the old bundle. If you need an update applied urgently, you have to decide whether to prompt the user to restart — and doing that mid-task is its own rudeness.

### Verify it

**Red flag: an AI claiming a native change can go out over the air.** The test question is blunt: "does this change touch any native module, permission, app config field or the Expo SDK version?" If yes, it is a store build. If the answer is vague, assume store build. A second check: ask whether the change requires the runtime version to change, and why.

**Red flag: an API change shipped in the same breath as a JavaScript update.** Ask explicitly: "what happens to a user still running the previous bundle after this API change deploys?" A correct answer describes additive changes and a window during which both shapes work. An answer of "they will get the update soon" is not an answer.

**Red flag: no rollback step.** An update plan without a documented way back is incomplete. Demand the exact steps, and ask how long it takes for the rollback to reach a user who has the app open right now.

**Red flag: no staged rollout and no crash check.** The point of a fast mechanism is a fast correction, which only works if you are watching. The update plan must say which number you watch — your crash-free sessions rate from chapter 8 — and for how long before you widen the rollout.

**Red flag: OTA used to get around a reviewer.** If any plan involves shipping a feature over the air because review rejected it or would reject it, stop. That is a policy violation with account termination at the end of it.

**Verify on a real device, twice.** Install the previous build, publish the update, close the app completely, reopen it, and confirm the new behaviour appears on the second launch rather than the first. Then roll it back and confirm you can get the old behaviour returned the same way. A rollback you have never performed is a hope.

## Walkthrough
1. Write two lists for your app: changes you can ship over the air, and changes that need a store build. Put your next five planned changes into one of the two lists.
2. Configure update channels: one for your team's preview builds and one for production. Confirm which channel your current store build listens to.
3. Find and record your build's runtime version, and write one sentence explaining what would make it change.
4. Install your current production build on a real phone from the store or from your testing track, so you are testing the real thing.
5. Make a small, visible, JavaScript-only change — a label, a colour, a piece of copy. Open a pull request for it and have a teammate read the diff, exactly as in M01.
6. Publish the update to your preview channel first. On the phone, close the app completely, reopen it, and note which launch the change actually appeared on.
7. Publish to production with a staged rollout if your tooling supports it. Watch your crash-free sessions number for a defined period before widening.
8. Now practise the rollback: re-publish the previous bundle and confirm on the phone that the old behaviour returns. Time how long it took.
9. Write the rollback steps and the measured time into your team runbook, where somebody who did not do it can follow them at 11 pm.
10. Take one planned API change and write down what an old bundle would do after it deploys. If the answer is "break", rewrite the change to be additive.
11. Record in your decision log your rule for when an OTA is allowed without review and when it is not, and who may publish to production.

## Talk to AI
```prompt
CONTEXT:      React Native app built with Expo, Android first, real users
              in India on metered mobile data. Product: [one sentence].
              We have a store build live in [production / closed testing].
              Our API is [stack] and we deploy it [how].
              Our next five planned changes:
              1. [change]
              2. [change]
              3. [change]
              4. [change]
              5. [change]
              Our current crash-free sessions rate: [X]%

GOAL:         A safe over-the-air update process we can use in an emergency
              at 11 pm and trust not to make things worse.

REQUIREMENTS: 1. Classify each of my five planned changes as OTA-eligible
                 or store-build-required, and say which native file, config
                 field, permission or SDK dependency forces the decision
              2. Explain what my runtime version is for and exactly which
                 kinds of change must bump it
              3. Design two update channels and tell me which channel my
                 current build listens to and how I confirm that
              4. Give me an API compatibility rule: for each of my changes
                 that touches the API, say what a user on the previous
                 bundle experiences after the API deploys, and how to make
                 the change additive instead
              5. Write a rollback runbook: the exact steps, who may run
                 them, and how long until a user sees the rollback
              6. Tell me which single number I watch after publishing, for
                 how long, and the threshold at which I roll back
              7. Estimate the download size of one update bundle and what
                 that costs a user paying per megabyte

CONSTRAINTS:  Expo managed workflow, free tier. Users on expensive metered
              data. Assume some users will not open the app for three
              weeks, so several bundle versions are live at once. Assume
              we cannot force anyone to restart the app.

NON-GOALS:    Do not propose feature flags or a canary infrastructure yet.
              Do not suggest shipping anything over the air that a store
              reviewer has declined. Do not change our API deployment
              process.

ACCEPTANCE:   - Every one of my five changes has a verdict with the
                specific reason attached
              - At least one change is correctly identified as needing a
                store build
              - The API rule states what happens to an old bundle, per
                change
              - The rollback runbook is followable by a teammate who has
                never done it
              - You have named the exact metric and threshold for rolling
                back

ASK:          Start with the five classifications and the API compatibility
              rule. Do not give me any commands until I have confirmed the
              classifications look right.
```

## Stuck? Ask AI
```prompt
Explain the difference between the JavaScript part and the native part of a React Native app to someone who has only built websites. Then give me ten specific changes a developer might want to make and ask me, one at a time, whether each needs a new store build or can go out over the air. Do not reveal your answers until I have guessed each one.
```

```prompt
I published an over-the-air update and now some users are broken while others are fine. Explain the most likely reasons different users are on different versions of my code, how I find out who is on what, and the safest order of actions to get everyone to a working state. Do not change any of my code yet.
```

```prompt
Explain version skew to me with a concrete example: an app where the JavaScript asks for a field the server has renamed. Walk through what each user sees depending on when they last opened the app. Then give me three rules for changing an API so that old app versions keep working, and one example of breaking each rule.
```

## Quiz

### Q1 · L2
Which change can be delivered to installed users over the air?
- (a) Adding the camera module to your app
- (b) Fixing a wrong property name in a checkout screen's JavaScript ✅
- (c) Changing the app icon
- (d) Adding a new runtime permission
> Only interpreted JavaScript can be replaced over the air. Native modules, permissions, icons and SDK versions all live in native files and require a store build and review.

### Q2 · L4
You publish an over-the-air update and simultaneously deploy an API change that renames a field. What happens?
- (a) Nothing; the update and the API change arrive together
- (b) Users still on the previous bundle break, because updates arrive only when each user next opens the app ✅
- (c) The store blocks the update until all users are current
- (d) The runtime version prevents the API change from taking effect
> Version skew is unavoidable: some users open the app in ten seconds and some in three weeks. API changes must be additive, with both shapes working for a window.

### Q3 · L3
What is the purpose of the runtime version on a build?
- (a) To order your releases chronologically
- (b) To ensure an update is only delivered to builds that can actually run it ✅
- (c) To tell the store which Android versions you support
- (d) To control how often the app checks for updates
> It is the compatibility stamp. When you add a native module the runtime version changes, so old installs correctly stop receiving updates that would reference code they do not contain.

### Q4 · L4
A reviewer rejects a feature. A teammate suggests shipping it as an over-the-air update afterwards, since review will not see it. What is wrong?
- (a) Nothing; code updates are explicitly permitted
- (b) Using the update mechanism to change what the reviewed app does is a policy violation that can end in account termination ✅
- (c) The feature would be too large to ship as a bundle
- (d) The runtime version would block it
> Stores permit updating interpreted code to fix and improve what you declared. They do not permit using it to become a different app than the one reviewed.

### Q5 · L5
Your team can now publish to all users in ten minutes with no external review. What is the biggest new risk?
- (a) Bundle downloads will exhaust your free tier
- (b) The friction that was accidentally enforcing your own review is gone, so unreviewed changes reach everyone fast ✅
- (c) The store will revoke your signing key
- (d) Users will receive updates they did not ask for
> Speed is valuable for emergencies and dangerous as a habit. The M01 discipline — a pull request, a teammate reading the diff, a check on a real device — has to be kept deliberately once the queue stops enforcing it.

## Go deeper
- [Expo: EAS Update introduction](https://docs.expo.dev/eas-update/introduction/)
- [Apple: App Store Review Guidelines, section 2 on performance](https://developer.apple.com/app-store/review/guidelines/#performance)
