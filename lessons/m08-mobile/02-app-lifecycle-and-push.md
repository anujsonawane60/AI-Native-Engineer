---
id: m08-02-lifecycle-and-push
module: m08-mobile
slug: app-lifecycle-and-push
title: "App Lifecycle, Permissions and Push Notifications"
order: 2
duration_min: 15
is_free: false
mission: false
volatility: medium
last_reviewed: 2026-11-16
---

## Key learnings
- Name the states a mobile app moves through, and what the OS throws away at each
- Ask for a permission at the moment it is needed, with a reason the user accepts
- Trace a push notification from your server to a locked phone
- Judge whether a notification earns its interruption before you send it
- Catch an AI that requests every permission on first launch or keeps state in memory

## Lesson

### The pain

A student opens the canteen app and is immediately asked to allow notifications, location, camera and her contacts. She taps Deny four times, and Android treats those denials as permanent — though contacts was never used by any feature, just copied from an AI's boilerplate. She then half-fills the order form, switches to WhatsApp, and returns to an empty form: her 2 GB phone reclaimed the app. The app promised something the OS never agreed to.

### The idea

**On a phone you do not own your app's life. The OS starts it, freezes it and kills it on behalf of the battery and the user — so any state you did not write down is lost.**

Two consequences. Assume termination at any moment, so persist anything the user typed. And anything the user must grant is a request you make once, so make it when they want the thing.

| Where you put it | Survives backgrounding | Survives termination | Survives reinstall |
|---|---|---|---|
| A variable in memory | usually | no | no |
| Local storage on the device | yes | yes | no |
| Your server | yes | yes | yes |

The rule: **what the user typed goes to device storage as they type it; what the business needs goes to the server.**

### The analogy

Your tray at the canteen counter. Step out for a ten-second call and it is where you left it — backgrounded, intact. Step out for five minutes in the lunch rush and it is gone: the counter needed the space. To keep your place you needed a token, something written down outside the tray. Permissions are the same — a regular asking "can I keep a tab?" after a month gets it; a stranger asking for a tab, your phone and your friend's number gets a no that sticks.

**Where the analogy breaks:** the canteen never pushes a notification into your pocket at 11 pm. Push is someone else's software waking a device your app is not even running on.

### Picture it

```
   NOT RUNNING
        │  user taps the icon  (cold start: slowest path)
        ▼
    FOREGROUND ◀──── user returns ────┐
        │                             │
        │  user switches app,         │
        │  a call arrives,            │
        │  screen locks               │
        ▼                             │
    BACKGROUND ───────────────────────┘
        │
        │  OS needs the RAM, or battery saver steps in
        ▼
    TERMINATED  ──▶ in-memory state is gone forever
```

An app moves between foreground and background many times a session; the move to terminated is made by the OS without asking, and everything held only in memory dies with it.

Push runs the other way, and your server cannot reach a phone directly.

```
  your server          Google FCM            the phone
 ┌────────────┐      ┌────────────┐      ┌──────────────┐
 │ "order 42  │─────▶│  routes by │─────▶│ OS draws the │
 │  is ready" │      │  device    │      │ notification │
 └────────────┘      │  token     │      └──────┬───────┘
                     └────────────┘      user taps it
                                                ▼
                                         your app opens
                                           on order 42
```

Your server sends a message plus a device token to FCM, the only party with a live connection to the phone; the OS draws the notification, and a tap deep-links into the right screen.

Web push works the same way through chapter 5's service worker, so an installable PWA receives push on Android with no store involved.

### Where it lives

Lifecycle and permissions live in M00's **client** box but decide what the others handle: termination means the server must tolerate a client that vanishes mid-flow and returns (chapter 3's queue), and changing tokens mean the server owns a users-to-tokens table (M04). A mobile session is a long-lived token in device storage, not a cookie dying with the tab.

**M08 is how to choose plus how to ship a PWA; M12 is the practicum** — the real FCM and APNs pipeline, categories, scheduling, delivery metrics, device APIs. No Firebase project needed here.

### Words to own

| Term | Plain meaning | Example |
|---|---|---|
| Background | Still in memory, not on screen | They switched to WhatsApp |
| Terminated | Killed by the OS; memory gone | Reclaimed in the lunch rush |
| Cold start | Launch from not-running, the slowest path | First tap of the day |
| Runtime permission | Granted by the user while the app runs | Camera, location, notifications |
| Rationale | Your own screen explaining why, shown first | "Only to attach this photo" |
| Device token | The address of one install on one phone | Changes on reinstall |
| FCM / APNs | Google's and Apple's push delivery services | Your server's only route in |
| Notification channel | A category the user can mute separately | "Order ready" vs "Offers" |
| Doze / battery saver | OS and vendor power saving that suspends apps | Xiaomi killing your app |

### Tradeoffs and gotchas

**Ask late, with a reason, and never on first launch.** Trigger (the user taps "Add a photo"), rationale (your own screen: "only to attach a photo to this order"), then the system dialog — which roughly doubles acceptance. On Android the second denial is effectively final, and since Android 13 notifications are a runtime permission too.

**Never ask for a permission no feature uses today.** Unused permissions are a reason to deny all of them.

**Do not trust background execution on cheap Android phones.** Xiaomi, Oppo, Vivo and Realme battery managers kill backgrounded apps within minutes; important work is driven by the server waking the device.

**A notification is a cost you impose on someone else.** Separate "your order is ready" from "today's offers", and ask whether you would want it on your own phone daily for a month.

### Verify it

**Where is the permission requested?** A call site in the startup path or a `useEffect` on mount is asking on first launch.

**Red flag: no rationale screen before the system dialog.** The AI skipped the half that decides the answer.

**Red flag: a permission list copied from boilerplate.** Name the feature behind each one; anything unnamed is deleted now.

**Red flag: the denied path is missing or loops.** Deny it on a real phone: a blank screen or a repeated dialog means that state was never designed.

**Red flag: a notification with no user value.** "Check out our new features!" is a retention anti-pattern, not a message.

**Red flag: state kept only in memory.** "React state" is the wrong answer to "where is a half-typed form when the OS kills the app".

## Walkthrough
1. Open your team's product on a real Android phone, start filling in your longest form, then switch to WhatsApp for a minute and come back. Write down exactly what was lost.
2. Repeat it harder: in Android developer options enable "don't keep activities", or on the web close and reopen the tab. Note what the user would have to retype.
3. List every permission your product asks for today. Beside each, write the one feature that uses it and the exact screen it is requested from. Anything with a blank is deleted.
4. For the permission that matters most to you, write the rationale text in your own words: what you access, what you do with it, and what you never do. Keep it under 25 words.
5. Draw the three screens of the sequence on paper — trigger, rationale, system dialog — then draw the fourth screen nobody draws: what the user sees after tapping Deny.
6. Deny that permission on your phone and use the product for two minutes. If anything is blank, broken or asks again, you have found work.
7. Write the three notifications your product would genuinely send, with the exact text and the time of day. Not categories — the actual sentences.
8. Show those three to five classmates who are not on your team and ask one question: would you want your phone to buzz for this? Count the yeses per message.
9. Cut every notification that scored below three out of five, and group the survivors into named channels a user could mute separately.
10. Write one decision-log entry: which permissions you ask for, at which moment, which notifications you will send, and which one you cut and why.

## Talk to AI
```prompt
CONTEXT:      I am a student directing AI rather than writing code. My
              team's product is [one sentence]. Stack: [your stack].
              Our users are on budget Android phones, 2-3 GB RAM, often
              Xiaomi or Realme with aggressive battery managers, on
              patchy 4G with metered data.

              Features that need something from the device:
              [list each feature and what it needs - camera, location,
              notifications, file picker, and so on]

              Today the app asks for: [list the permissions it asks for
              now, and where]. State we keep while the user works:
              [list it - form fields, a draft order, a cart].

GOAL:         A permission and notification plan that a real user accepts,
              and an app that loses nothing when the OS kills it.

REQUIREMENTS: 1. For every permission, tell me the exact user action it
                 should be requested from, and write the rationale screen
                 text in under 25 words
              2. For every permission, describe the screen the user sees
                 after denying, including what still works
              3. List every piece of state we hold and say where it must
                 be written so it survives backgrounding, termination and
                 a cold start - be specific about which storage
              4. Write the three notifications this product should send,
                 as the exact text, with the time of day and the channel
                 each belongs to
              5. Name the notifications a product like mine usually sends
                 that I should refuse to send, and why
              6. Tell me what will break specifically because of vendor
                 battery managers on cheap Android phones

CONSTRAINTS:  Free tier only. Assume notification permission must be
              requested at runtime. Assume the app can be terminated at
              any moment with no warning. Do not rely on any code of mine
              running while the app is closed. No paid push service.

NON-GOALS:    Do not set up a push provider or write credentials config -
              that is a later module. Do not add any permission that no
              feature in my list uses. Do not redesign my features.

ACCEPTANCE:   - Not one permission is requested during app startup
              - Every permission in your plan maps to a feature I listed
              - Every denied path leaves a usable app
              - Every piece of state I listed has a named storage
                location and a reason
              - Each of the three notifications would survive the test
                "would I want this on my own phone, daily, for a month"

ASK:          Before any plan, list back to me every permission I named
              that no feature justifies, and ask me to confirm deleting
              it. Then give me the plan.
```

## Stuck? Ask AI
```prompt
Explain to someone who has never built software what happens to a phone app when the user switches to another app, when the screen locks, and when the phone runs out of memory. Use one everyday analogy, keep it under 250 words, and end by telling me the single thing I must do differently because of it.
```

```prompt
Give me six short scenarios where a mobile app asks the user for a permission. Some are asked at a good moment with a good reason and some are not. Ask me to sort them and explain my reasoning before you tell me which is which, then show me how you would rewrite the bad ones.
```

```prompt
I want to understand push notifications from first principles. Explain, without code, how a message gets from my server onto a locked phone's screen: who holds the connection, what a device token is, why it changes, and what happens to the message if the phone is switched off for a day. Then tell me the three things that most commonly go wrong in that chain.
```

## Quiz

### Q1 · L3
A user half-fills your order form, switches to WhatsApp for two minutes, returns, and the form is empty. What actually happened?
- (a) A bug in the form validation code
- (b) The OS terminated the app to reclaim memory, and the state lived only in memory ✅
- (c) The network connection dropped and the draft failed to upload
- (d) The user's phone cleared the browser cache
> On a 2 GB phone, termination while backgrounded is normal OS behaviour, not a fault. Anything the user typed must be written to device storage as they type, which is the M00 lesson that RAM forgets.

### Q2 · L2
On Android, what is the practical consequence of a user denying a permission twice?
- (a) The app is removed from the Play Store
- (b) Your app can no longer prompt for it; only Settings can grant it ✅
- (c) The permission is granted automatically on the next launch
- (d) Nothing changes; you may keep asking each session
> The second denial is effectively final from inside the app, which is why the moment and the reason you ask matter far more than the wording of the system dialog.

### Q3 · L4
Which sequence gives you the best chance of a yes for camera access?
- (a) Ask on first launch, alongside notifications and location
- (b) Ask on the splash screen so the app is ready before use
- (c) User taps "Add a photo", your own screen explains why, then the OS dialog ✅
- (d) Never ask; use a file upload field instead and hope they allow it
> Trigger, then rationale, then request. The user already wants the outcome when the dialog appears, which roughly doubles acceptance compared with asking cold.

### Q4 · L4
Your server needs to tell one specific phone that order 42 is ready. What does it send and to whom?
- (a) A direct TCP connection to the phone's IP address
- (b) The message plus that install's device token, to FCM or APNs ✅
- (c) An SMS, because push cannot reach a locked phone
- (d) A message to the user's email address, which the OS converts to a notification
> Only the platform service holds a live connection to the device. Your server addresses one app installation by its device token, which changes on reinstall and must be refreshed.

### Q5 · L5
Your team plans a daily 11 am notification saying "Check out today's menu!". What is the strongest argument against shipping it?
- (a) Daily notifications are technically harder than event-driven ones
- (b) It spends attention without returning value, and the user's mute button is uninstall ✅
- (c) FCM rate-limits messages sent at the same time each day
- (d) Android 13 forbids promotional notifications
> A notification is a cost imposed on someone else. "Your order is ready, counter 2" earns the interruption; a generic nudge trains the user to silence the channel that carried the useful message.

## Go deeper
- [Android Developers: App permissions best practices](https://developer.android.com/training/permissions/usage-notes)
- [web.dev: Push notifications overview](https://web.dev/explore/notifications)
