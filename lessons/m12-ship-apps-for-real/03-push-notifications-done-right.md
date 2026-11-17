---
id: m12-03-push-done-right
module: m12-ship-apps-for-real
slug: push-notifications-done-right
title: "Push Notifications People Do Not Mute"
order: 3
duration_min: 15
is_free: false
mission: false
volatility: low
last_reviewed: 2026-11-16
---

## Key learnings
- Trace a push notification from your server to a specific phone
- Treat notification permission as a one-time asset you can only spend once
- Ask for permission at the moment of value rather than at launch
- Handle device tokens, token rotation and per-user delivery correctly
- Spot a notification plan that serves your team rather than the user

## Lesson

### The pain

Your team ships push on a Monday. The first one is genuinely useful: "Your thali is ready at counter 2." On Tuesday somebody adds "New items added today!", on Wednesday "Don't miss lunch!", on Thursday a 9:40 pm re-engagement push. By Friday your opt-out rate is 44%, two reviews say "too many notifications", and the Monday notification no longer reaches half your users.

**You cannot undo it.** A user who turns your notifications off in Android settings is gone from that channel permanently — no re-asking, no apology screen. Another team makes the opposite mistake and asks on the first screen, before the user knows what the app does: 70% tap "Don't allow", including everyone who wanted the thali notification.

### The idea

**Notification permission is a single-use, non-renewable asset. Spend it only on messages the user would have opened the app to find out.**

That test settles almost every argument your team will have. "Your order is ready" passes: she was waiting for exactly that. "New items added today" fails: nobody was waiting. If a notification exists because *you* want attention rather than because *she* wants information, it is a tax on having your app installed.

Two consequences. Ask late, at the moment the user hits the thing the notification is about. And build fewer types than you think, each with its own off switch inside your app, so a user who hates one does not kill them all at OS level.

### The analogy

The canteen tannoy saying "Table four, your thali is ready" is the perfect announcement: one person, information they are actively waiting for. Now have the tannoy also say "Try our new cold coffee!" every twenty minutes — identical mechanism, opposite value — and within a day everyone filters out the tannoy, including table four.

**Where the analogy breaks**, dangerously. A tannoy cannot be muted by one person, and tomorrow everyone's ears work again. Push is per-person, permanently muteable, and the mute button is two taps away. The canteen's worst case is annoyance; yours is permanent loss of a channel.

### Picture it

```
 Your server            Expo push         FCM (Android)        Phone
┌──────────┐          ┌───────────┐      ┌──────────────┐   ┌────────┐
│ order is │─token +─▶│ routes by │─────▶│ Google's     │──▶│ shows  │
│  ready   │ message  │ token type│      │ delivery     │   │ banner │
└──────────┘          └───────────┘      └──────────────┘   └────────┘
     ▲                                   ┌──────────────┐        │
     │                                   │ APNs (iOS)   │──▶ ... │
  token stored                           └──────────────┘        │
  against user_id ◀─────── token registered on first grant ──────┘
```

Your server never talks to a phone directly: it sends a message plus a device token to a push service, which hands it to Google's or Apple's delivery network, which wakes the phone.

### Where it lives

Four moving parts, in order.

**Permission.** iOS always required asking; since Android 13 you must ask at runtime too. Older Android granted it on install, so opt-in looks suspiciously high on old devices.

**The device token.** The OS gives your app an opaque string identifying *this install on this device*. Your app sends it to your server, which stores it against the user's id — no user id, no targeted push. Tokens rotate on reinstall and on data clear, so registration runs on every app start, your server upserts rather than inserts, and invalid tokens get deleted.

**The send.** Your server posts message plus token to Expo's push service, which forwards to FCM for Android and APNs for iOS. The credential stays on the server.

**The tap.** The notification must deep-link to the order or the result — never the home screen.

Token table: M04. Device token as personal data: M07, and again on chapter 5's data safety form. Opt-in and tap-through rates: M16 needs them, so record them from the first send.

### Words to own

| Term | Plain meaning | Example |
|---|---|---|
| Device token | Opaque id for one install on one device | Stored against `user_id` |
| FCM | Firebase Cloud Messaging, Android delivery | Google's pipe to the phone |
| APNs | Apple Push Notification service | Apple's equivalent |
| Notification channel | Android's per-category on/off switch | "Order updates" vs "Offers" |
| Deep link | A link that opens one screen in the app | Opens order #412, not home |
| Opt-in rate | Share of users who allowed notifications | 34% if you ask at launch |
| Transactional push | About something the user did or awaits | "Your order is ready" |
| Marketing push | About something you want them to do | "New items today!" |
| Quiet hours | Times you refuse to send | Nothing between 10 pm and 8 am |
| Best-effort delivery | Not guaranteed, not instant | Phone asleep, message waits |

### Tradeoffs and gotchas

**Delivery is best-effort.** The phone may be off, the OS may batch your message to save battery, the user may be in a dead zone. Push is a nudge, not a transport: anything that matters must also exist inside the app.

**Indian Android is harder than the docs admit.** Xiaomi, Oppo, Vivo and Realme battery managers suspend background activity, so your notification may arrive late or never unless the user enables autostart. Design so a missed notification is survivable.

**Android 13 changed the economics of asking.** Asking at first launch loses most users; asking right after the first order, with one sentence of your own explanation first, keeps most of them.

**Give every category its own switch, plus quiet hours.** Mirror Android's channels in your own settings screen, or a user who wants order updates and hates offers has one tool: turning everything off. And write quiet hours in before your first campaign — a 9:40 pm push to a student in a hostel is a cost you impose.

**Never put a push credential in the app.** If the app itself sends, anyone who downloads it can notify all your users.

### Verify it

**Red flag: a notification with no user value.** For every type AI proposes, ask: would the user have opened the app to find this out? If not, you are building an opt-out machine.

**Red flag: permission requested at launch.** The correct pattern: user does the thing → your own screen explains in one sentence what you will send and when → then the OS prompt. Ask to see the flow, not just the call.

**Red flag: a token stored without a user id.** No foreign key to your users means you can broadcast but never target, and cannot stop sending to a deleted account. Check there is a delete path.

**Red flag: a notification that opens the home screen.** Tap every type on a real phone and check where it lands.

**Measure the number, not the vibe.** After a week you should know your opt-in rate, how many were sent, how many tapped, and how many users turned notifications off. Without the last you cannot tell you are destroying your own channel until it is gone.

## Walkthrough
1. List every notification your product might send. For each, write the one sentence a user would say about it. Delete the ones where that sentence is negative.
2. For each survivor, write whether it is transactional or marketing, who it goes to, what screen it opens, and whether it is still useful if it arrives an hour late.
3. Design the moment of asking: pick the exact user action after which you request permission, and write the one sentence of your own explanation that appears first.
4. Have AI add notification support to your Expo app, register the device token on every app start, and send it to your API. Read the registration code yourself.
5. Design the server-side table: token, user id, platform, created at, last seen. Confirm there is a delete path for an invalid token and for a deleted account.
6. Build one sending endpoint on your server that takes a user id and a message. Confirm the push credential lives only on the server, never in the app.
7. Send a real notification to your own phone with the app closed. Then with the app open in the foreground. Note that the behaviour differs and decide what you want in each case.
8. Make the notification deep-link to the specific screen. Tap it from the lock screen and confirm where you land.
9. Test on the cheapest Android phone you can borrow, especially a Xiaomi, Oppo, Vivo or Realme. If it does not arrive, find the battery or autostart setting and record what the user would have had to do.
10. Add your own in-app settings screen with one switch per notification category, and confirm turning one off actually stops that category.
11. Write your quiet hours and your maximum sends per user per week into your decision log, before you send anything to real users.

## Talk to AI
```prompt
CONTEXT:      We have a React Native app built with Expo, shipping Android
              first to students in India on budget phones, many of them
              Xiaomi, Oppo, Vivo or Realme. Our product is [one sentence].
              Our API is [stack], and users log in, so we have a user id
              for every signed-in person.
              Candidate notifications we have brainstormed:
              1. [notification idea]
              2. [notification idea]
              3. [notification idea]

GOAL:         A notification design that users keep switched on for a year,
              plus the minimum plumbing to deliver it to one named user.

REQUIREMENTS: 1. For each candidate, judge it against one test: would the
                 user have opened the app to find this out? Give a verdict
                 of keep, change or drop, and say why in one sentence
              2. Design the permission moment: the exact user action after
                 which we ask, and the one sentence of our own copy that
                 appears before the OS prompt
              3. Design the token table, including rotation on reinstall,
                 cleanup of invalid tokens, and deletion when a user
                 deletes their account
              4. Describe where the push credential lives and prove the app
                 itself cannot send notifications
              5. Specify the deep link target screen for each surviving
                 notification
              6. Give me quiet hours and a per-user frequency cap as
                 concrete numbers, and explain each
              7. Tell me what the user on a Xiaomi phone may have to do for
                 delivery to work, and how our product stays usable if they
                 never do it

CONSTRAINTS:  Expo managed workflow. Free tiers only. Android 10 and newer,
              including Android 13 runtime permission. Assume delivery is
              best-effort and sometimes hours late. Our users pay for data.

NON-GOALS:    No marketing campaigns. No re-engagement pushes. Do not
              design an email or SMS channel. Do not add an analytics
              product; just tell me which four numbers to record.

ACCEPTANCE:   - At least one of my candidate notifications is dropped, with
                a reason
              - The permission request does not happen on first launch
              - No credential or server key appears in any client code
              - Every surviving notification names a specific screen it
                opens, not the home screen
              - Every notification is still useful, or harmlessly ignorable,
                if it arrives 90 minutes late
              - There is a per-category off switch inside our own app

ASK:          Start with the verdicts on my three candidates and the
              permission moment. Do not write any code until I have agreed
              which notifications we are actually building.
```

## Stuck? Ask AI
```prompt
Explain how a push notification physically reaches a specific phone, step by step, to someone who has never written server code. Name each party involved and what it knows. Then ask me to explain back where the device token comes from and where it is stored, and correct me if I get it wrong.
```

```prompt
I am about to add notifications to my app and I want to be talked out of the bad ones. Here are the notifications I plan to send: [list them]. For each one, argue the user's side as harshly as you can, then tell me which single one I should ship first and which I should never ship. Be blunt.
```

```prompt
My push notifications arrive on one Android phone and not on another. Both have permission granted. Explain the most likely causes in order of probability, including manufacturer battery restrictions common in India, and tell me exactly what to check on the failing phone. Do not suggest code changes until we have identified the cause.
```

## Quiz

### Q1 · L2
Why is notification permission described as a single-use, non-renewable asset?
- (a) The operating system only allows one prompt per app version
- (b) A user who disables notifications in settings cannot be re-prompted, so the channel is lost permanently ✅
- (c) Each device token can only receive one notification
- (d) Push services charge per permission request
> You can ask once and you can lose it forever. That asymmetry is the reason to ask late and to send only messages the user was already waiting for.

### Q2 · L3
When is the best moment to request notification permission?
- (a) In the first screen's setup code, to get it out of the way
- (b) Right after the user does the thing the notification is about, preceded by your own one-sentence explanation ✅
- (c) At the end of signup, bundled with the other permissions
- (d) Only when the user visits your settings screen
> Asking in context roughly triples opt-in compared with asking at launch, because the user can see what they are agreeing to. Same code, different placement, very different outcome.

### Q3 · L4
Your server stores device tokens but not the user id they belong to. What breaks?
- (a) Nothing; tokens are enough to send notifications
- (b) You can broadcast but never tell one person their order is ready, and you cannot stop sending to a deleted account ✅
- (c) Android will reject the tokens as invalid
- (d) Notifications arrive but cannot be deep-linked
> Targeting and deletion both depend on the link to a user. A token table with no foreign key is a broadcast list, which is almost never what a product needs.

### Q4 · L4
An AI gives you code where the mobile app itself calls the push service to send notifications. What is wrong?
- (a) Nothing, it reduces server load
- (b) It requires a server credential inside the app, so anyone who downloads the app can notify all your users ✅
- (c) Mobile apps are not allowed to make network requests to Google
- (d) It would double the notification delivery time
> App bundles are downloadable and unpackable. Sending credentials belong on your server, and the app should only ask your API to send — the same secrets rule as M06.

### Q5 · L5
Your product tells users when their order is ready. A teammate argues that push notification is sufficient and the information need not appear in the app. What is the strongest objection?
- (a) Push notifications cost money per message
- (b) Delivery is best effort — phones sleep, networks drop and Indian OEM battery managers suspend background work — so the only reliable place for the information is inside the app ✅
- (c) Notifications cannot contain order numbers
- (d) Users prefer SMS for order updates
> Push is a nudge, not a transport. On the budget Android phones your users carry, a missed or delayed notification is routine, so the app must always be able to answer the question on its own.

## Go deeper
- [Expo: push notifications overview](https://docs.expo.dev/push-notifications/overview/)
- [Android developers: notification runtime permission](https://developer.android.com/develop/ui/views/notifications/notification-permission)
