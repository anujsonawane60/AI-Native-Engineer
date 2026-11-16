---
id: m08-03-offline-and-deep-links
module: m08-mobile
slug: offline-first-and-deep-links
title: "Offline-First, Sync, Local Storage and Deep Links"
order: 3
duration_min: 25
is_free: false
mission: false
volatility: low
last_reviewed: 2026-11-16
---

## Key learnings
- Design for a network that disappears mid-action instead of one that is simply slow
- Choose the right local storage for a given piece of data, and know its limits
- Queue a write, retry it safely, and explain why an idempotency key is required
- Make every screen reachable by a URL that survives a WhatsApp forward
- Catch an AI that caches data which must never be stale, or retries without a key

## Lesson

### The pain

A student is on the third floor of the library, where the signal dies. She opens the canteen app, types a long note — "no onions, extra chutney, paying by UPI at the counter" — and taps Place Order.

The spinner turns for forty seconds. She taps again. She walks to the stairwell, finds one bar, and the app shows an error with no detail. She reloads, and the note is gone. At the counter the cook holds two identical orders, because both taps eventually arrived — and neither carries the note, because the request holding it timed out at the phone while the server was still working.

Nothing here is exotic. The app was built on campus Wi-Fi, where the network is solved, and then met the real Indian network: present, absent, and — worst — *present but unusable*. The endless spinner is not a bug in the spinner. It is the absence of a decision about what happens when the network does not answer.

### The idea

**Treat the network as something that might not be there, and the copy on the device as the one the user may see and change.**

That is offline-first, and it inverts the usual order. Instead of "ask the server, show a spinner, show what comes back", you do "show what is on the device now, record the change on the device, send it when sending is possible". The network becomes a background job, not a blocking step.

The chapter's second half is the mirror image: you still need a way to *point at* a screen. That is a deep link, and it is how your product travels through WhatsApp.

### The analogy

The canteen at 1:10 pm runs on paper slips.

You do not wait at the counter for the cook to confirm before you may walk away. You write the order on a slip, the cashier stamps a token number, and you sit down. The slip is the record. The kitchen works through the spike at its own pace, and if the gas fails the slips wait — nobody's order is lost, nobody stood frozen at the counter. That is a write queue: your tap creates a slip on the device, the UI shows "order placed, token 42" at once, and the slip reaches the server when the network allows.

The token is the other half. Two identical slips with the *same* token are one order, and the kitchen makes one plate. Two with no token are two plates and one annoyed cook. That number is an idempotency key.

**Where the analogy breaks:** the canteen has one copy of the truth, the kitchen. An offline-first app has one per phone, and they can diverge. If two hostel-mates both claim the last plate of biryani while offline, both slips say yes and the server must choose. Paper cannot create that problem; distributed copies can, and it is called a conflict.

### Picture it

```
  tap "Place order"
        │
        ▼
 ┌───────────────┐  UI shows "placed, token 42" at once
 │ local store   │
 │ order + key   │
 └───────┬───────┘
         ▼
 ┌───────────────┐   offline ──▶ wait, retry later
 │ outbox queue  │
 └───────┬───────┘   online
         ▼
   POST /orders  with Idempotency-Key: 42
         │
         ▼
 ┌───────────────┐  same key ──▶ the SAME order
 │ your server   │  new key  ──▶ one new order
 └───────────────┘
```

The tap is recorded on the device and acknowledged at once; a background queue delivers it with a key that lets the server recognise a retry rather than create a second order.

### The storage ladder

Not all local data belongs in the same place, and picking wrong is how teams put a 6 MB JSON string in a store built for a few MB of text.

| Where | Holds | Survives | Use it for |
|---|---|---|---|
| Memory | anything | nothing | this second's UI state |
| localStorage | MBs of text, synchronous | restart | a flag, a theme |
| IndexedDB | tens of MB, structured, async | restart | outbox, lists, drafts |
| Cache Storage | whole HTTP responses | restart | app shell and images |
| Your server | the truth | everything | money or shared data |

Two rules make this easy. **Anything the user typed goes to IndexedDB as they type it** — asynchronous, so it never blocks the screen, and it holds real amounts of data. **Anything another person depends on goes to the server**, because a copy on one phone is not a shared fact.

One warning: the browser may evict all of it. On a phone whose 16 GB is 95% full, cached data can vanish without notice. Local storage is never the only copy of anything that matters.

### Sync, conflicts and what to do about them

The moment two copies exist you need a rule for disagreement. There are four, in increasing order of effort.

**Server wins.** The device's copy is a cache; on reconnect you discard it and refetch. Correct for data the user only reads — menu, prices, hours — and the right default for most of a student product.

**Last write wins.** Whichever update arrives last is kept. Fine for a personal note. Quietly destructive for anything two people edit, because the loser's change vanishes without trace.

**Per-field merge.** Accept both updates when they touched different fields. More work, far fewer lost edits, right for a shared document.

**Append-only.** Never update a row; record events and compute state from them. Two offline claims on the last plate become two recorded claims and one business decision — the honest model for stock.

The decision is product, not technical. "What should happen if two people do this at once, offline?" is a question your team answers in a sentence. If nobody has answered it, whatever the AI generated answered it for you, usually with last-write-wins.

And some things must never be optimistic. **Do not pretend a payment succeeded.** A UPI payment, a seat booking, the last plate of biryani need the server's confirmation first, because an optimistic lie costs real money and real trust. Say "we will take payment when you are back online".

### Deep links: a URL for every screen

A deep link is a URL that opens one specific screen with one specific thing on it — `canteen.app/order/42`, not "open the app and navigate". Three reasons it matters more here than the textbooks suggest.

**WhatsApp is your distribution.** Products spread by someone forwarding a link to a hostel group. A link that opens on the right dish converts; a generic home screen does not.

**Notifications need somewhere to land.** The push "order 42 is ready" must open order 42.

**It is what makes you shareable at all.** On the web this is nearly free — unless your team built every screen as a state change behind one URL, a mistake common in AI-generated single-page apps that quietly removes your ability to be shared.

Two terms for the app world. An **App Link** on Android (a **Universal Link** on iOS) is an ordinary `https://` URL your app claims, verified by a file you host on your own domain, so one link opens the app if installed and the website if not. A **custom scheme** like `canteen://order/42` fails badly: it does nothing if the app is absent, and anyone can register it. A **deferred deep link** — tap, no app, install, still land on order 42 — is M12's problem.

### Where it lives

On M00's four boxes, offline-first moves a copy of the **data** box onto the **client** and turns the **network** box from a prerequisite into a background worker. Your server changes in one way: it must accept an idempotency key and promise that two requests with the same key produce one result.

M02 gave you service workers and HTTP caching; the service worker is what serves your shell when the network is gone, and chapter 5 makes you build one. M05's caching chapters gave you TTLs and invalidation, and the same thinking applies to the copy on the phone under a harsher constraint — a phone can be offline for an hour, not seconds.

This chapter is the **overview**: the model, the storage choices, the conflict rules, the deep link. **M12's "Offline and Sync When the Network Is Bad" is the practicum** — a real sync engine, background sync, local database migrations and the deferred deep-link case. Here you decide the rules; there you implement them.

### Words to own

| Term | Plain meaning | Example |
|---|---|---|
| Offline-first | The device's copy is what the user sees and edits | Menu loads with no signal |
| Optimistic UI | Show success before the server confirms | "Placed, token 42" |
| Outbox | Pending changes waiting to be sent | The library order |
| Idempotency key | A unique id so a retry is not a new order | `Idempotency-Key: 42` |
| Conflict | Two copies changed the same thing | Both claimed the last plate |
| IndexedDB | The browser's real local database | Outbox and drafts |
| Deep link | A URL that opens one exact screen | `/order/42` |
| App Link | A verified https link your app claims | Opens app, else the site |

### Tradeoffs and gotchas

**Offline-first costs real complexity, so scope it.** You do not need a sync engine. Pick the three things that must work with no signal — reading the menu, drafting an order, seeing your last order — and make only those work. Everything else may say "you are offline".

**A spinner with no timeout is the worst possible state.** A request that never resolves is worse than a failure, because the user cannot act. Every call needs a timeout and an outcome.

**Never retry without an idempotency key.** Retries are how you survive a bad network; a retry without a key is how one tap becomes three orders. Write the server contract first.

**Optimistic UI must be reversible, visibly.** If the sync later fails, the item comes back with an explanation rather than silently disappearing. Show pending, then confirm or revert.

**Hash-routed single-page apps break sharing.** If every screen is the same URL you have no deep links, nothing is forwardable on WhatsApp, and push has nowhere to land. Retrofitting routes is tedious, so check early.

### Verify it

The network is the easiest thing to test, and the thing AI output most consistently assumes is fine. Start with the network off rather than slow — aeroplane mode, two minutes of use — then the harder case, dev tools throttled to high latency with packet loss, which is closer to campus 4G than "offline" is.

**Red flag: the plan assumes a fast connection.** No timeout, no retry, no offline branch and no pending state means it was written for Wi-Fi. Ask: "what does the user see at second 10, 30 and 60 of a request that never answers?"

**Red flag: retry logic with no idempotency key.** Search the diff for the retry, then the key. If the retry exists and the key does not, duplicate orders are a schedule rather than a risk. Check the server honours the key rather than merely accepting the header.

**Red flag: something that must never be stale, cached.** Prices, stock, balances, anything a payment depends on. Ask which fields come from the local copy and for how long, then what happens if the price changed ten minutes ago. A tick shown before money moved is a defect.

**Red flag: one URL for the whole app.** Open three screens and watch the address bar. If it never changes you cannot be shared, bookmarked or opened from a notification.

Then run the duplicate test by hand: offline, tap submit three times, back online, count the rows. The answer must be one. That test catches the worst bug in the chapter.

## Walkthrough
1. Write down the three things in your product that must work with no signal at all. Three, not ten. Everything else is allowed to say "you are offline".
2. Put your phone in aeroplane mode, open your deployed product, and write down what the user sees. Most first drafts show a blank screen or a browser error page.
3. Turn the network back on and watch what happens. If the app does not recover without a manual reload, note that as a defect.
4. In browser dev tools, throttle to a slow profile and submit your main form. Time it. Then throttle to offline mid-submit and record exactly what the user sees at 10, 30 and 60 seconds.
5. Open three different screens of your product and copy the address bar each time. If the URL never changes, you have no deep links and no shareable screens — log it.
6. Take the URL of one inner screen, send it to a teammate on WhatsApp, and have them open it on a phone that has never used your product. Note every step they had to take.
7. Draw your outbox on paper: where a pending write is stored, what the user sees while it is pending, how often it retries, and what the key is.
8. Decide your conflict rule for each kind of data — server wins, last write wins, per-field merge, or append-only — and write one line per kind. Say explicitly which data is never allowed to be optimistic.
9. Now run the duplicate test. Go offline, tap submit three times, go back online, and count the rows created. If the answer is not one, your key is missing or your server ignores it.
10. Check your storage choices against the ladder: list each thing you store locally and name where it lives and why. Move anything the user typed into IndexedDB.
11. Write one decision-log entry: the three offline features, the conflict rule per data type, and the one thing you deliberately left online-only.

## Talk to AI
```prompt
CONTEXT:      I am a student directing AI rather than writing code. Our
              product is [one sentence]. Stack: [your stack], free tier.
              Users are on budget Android phones on patchy Indian 4G -
              often present but unusable, sometimes fully offline for
              several minutes, with metered data.

              Screens we have: [list them]
              Data the user reads: [list it]
              Actions the user takes that write to the server: [list them]
              Anything involving money or shared stock: [list it, or none]
              Where we currently store anything on the device: [list it]

GOAL:         A specific offline plan: what works with no signal, what
              honestly refuses, and how a queued write reaches the server
              exactly once.

REQUIREMENTS: 1. Pick the three features that must work fully offline and
                 justify each from what my users do. Say plainly which
                 features should NOT be offline, especially payments
              2. For each piece of data I listed, name the storage layer
                 it belongs in - memory, localStorage, IndexedDB or Cache
                 Storage - and say why, including the size limits
              3. Design the write queue: where a pending action is stored,
                 what the user sees while pending, the retry schedule with
                 a timeout, and what happens after repeated failure
              4. Specify the idempotency contract: what the key is, where
                 it is generated, and exactly what my server must do when
                 it sees the same key twice
              5. Give me a conflict rule per data type, chosen from server
                 wins, last write wins, per-field merge, append-only, and
                 say what the user loses under each
              6. Tell me which of my screens need their own URL so a
                 notification or a WhatsApp forward can open them directly
              7. List the user-visible states I must design: offline,
                 pending, synced, failed, and conflict

CONSTRAINTS:  Free tier only. Assume local storage can be evicted at any
              time, so it is never the only copy of anything important.
              Assume a request may hang forever rather than fail.
              Every network call must have a timeout. No paid sync
              service or realtime add-on.

NON-GOALS:    Do not build a general sync engine. Do not write code yet.
              Do not make payments optimistic. Do not touch my auth.
              Do not handle the install-then-deep-link case.

ACCEPTANCE:   - Exactly three fully-offline features, each justified
              - Every data item has one named storage layer and a reason
              - The retry plan includes a timeout and a give-up state
              - The idempotency section says what the SERVER does, not
                just what the client sends
              - Nothing involving money shows success before confirmation
              - Every state in requirement 7 has a described screen

ASK:          Before any plan, tell me which of my listed actions would be
              dangerous to make optimistic, and why. Then give me the plan
              as a table, not prose. No code until I pick the approach.
```

## Stuck? Ask AI
```prompt
Explain what an idempotency key is to someone who has never written code, using an everyday example with paper and a counter rather than anything technical. Then give me three real situations where leaving it out causes a visible, expensive bug, and tell me what the user sees in each.
```

```prompt
Give me five pieces of data a small ordering app might store: a login token, a theme preference, a 20 MB cached image set, a half-typed order note, and the current price of every dish. For each, ask me which local storage layer it belongs in and why, and make me answer before you reveal yours. Then tell me which one should not be stored on the device at all.
```

```prompt
Two users of my app are both offline. Both claim the last available item. Both see "confirmed". Walk me through what should happen when they reconnect, under four different conflict rules, and tell me what each user experiences in each case. Then tell me which rule you would choose for stock and why, and what I should have shown them instead of "confirmed".
```

## Quiz

### Q1 · L3
A user taps Place Order three times while offline. When the network returns, how many orders should exist, and what makes that true?
- (a) Three, because the user tapped three times
- (b) One, because each retry carries the same idempotency key the server honours ✅
- (c) One, because the browser automatically discards duplicate requests
- (d) Zero, because offline requests are never queued
> The key is what lets the server recognise a retry instead of creating a second row. Retries without a key turn a bad network into duplicate orders, which is the classic failure of this chapter.

### Q2 · L3
You need to store a half-typed order note as the user types, on a phone with patchy 4G. Where does it belong?
- (a) A React state variable, since it is temporary
- (b) localStorage, because it is synchronous and simple
- (c) IndexedDB on the device, written as they type ✅
- (d) Your server, on every keystroke
> Memory dies when the OS terminates the app, and a synchronous store blocks the screen. IndexedDB is asynchronous and holds real amounts of structured data, which is exactly the draft case.

### Q3 · L4
Which of these should NOT use optimistic UI?
- (a) Marking a dish as a favourite
- (b) Saving a draft note on an order
- (c) Changing the app's theme to dark
- (d) Confirming that a UPI payment succeeded ✅
> An optimistic lie about money costs real trust and real rupees. The honest offline behaviour is to say payment will be taken when back online, never to show a tick before the server confirms.

### Q4 · L4
Your app shows a spinner for 60 seconds on a present-but-unusable connection and then nothing. Why is this worse than a plain failure?
- (a) It uses more mobile data than a failure would
- (b) The user cannot act, because no outcome was ever communicated ✅
- (c) It keeps the service worker from installing
- (d) It causes the OS to terminate the app
> Every network call needs a timeout and a user-visible outcome. A state that never resolves removes the user's ability to retry, queue, or walk to the counter instead.

### Q5 · L5
Every screen in your app is the same URL. Name the two things this costs you, beyond tidiness.
- (a) Slower first paint and larger bundles
- (b) Shareability on WhatsApp, and a target for push notifications to open ✅
- (c) HTTPS support and installability
- (d) Local storage quota and cache eviction behaviour
> Deep links are how a product spreads through a hostel group and how "order 42 is ready" opens order 42. One URL for everything removes both, and retrofitting routes later is tedious.

## Go deeper
- [MDN: Using IndexedDB](https://developer.mozilla.org/en-US/docs/Web/API/IndexedDB_API/Using_IndexedDB)
- [Android Developers: Verify Android App Links](https://developer.android.com/training/app-links/verify-android-applinks)
