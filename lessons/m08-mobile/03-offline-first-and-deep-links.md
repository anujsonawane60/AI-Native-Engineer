---
id: m08-03-offline-and-deep-links
module: m08-mobile
slug: offline-first-and-deep-links
title: "Offline-First, Sync, Local Storage and Deep Links"
order: 3
duration_min: 15
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

A student on the library's third floor, where the signal dies, types a long note — "no onions, extra chutney, paying by UPI at the counter" — and taps Place Order. The spinner turns for forty seconds, so she taps again, walks to the stairwell, and gets an error. She reloads, the note is gone, and the cook holds two identical orders. The app was built on campus Wi-Fi, then met the real Indian network: present, absent, and worst of all *present but unusable*.

### The idea

**Treat the network as something that might not be there, and the copy on the device as the one the user may see and change.**

That is offline-first: instead of "ask the server, show a spinner, show what comes back", you "show what is on the device, record the change, send it when sending is possible". The network becomes a background job, not a blocking step.

The mirror image: a **deep link** is a URL that opens one exact screen with one exact thing on it — `canteen.app/order/42`. You need them because WhatsApp is your distribution, because "order 42 is ready" must land on order 42, and because without per-screen URLs nothing can be shared.

### The analogy

The canteen at 1:10 pm runs on paper slips: you write the order on a slip, the cashier stamps a token number, and you sit down rather than wait at the counter. That is a write queue — your tap creates a slip on the device, the UI says "order placed, token 42" at once, and the slip reaches the server when the network allows. Two identical slips with the *same* token are one order, and that number is an idempotency key.

**Where the analogy breaks:** the canteen has one copy of the truth; an offline-first app has one per phone. Two hostel-mates claiming the last plate of biryani while offline produce two slips that both say yes. That is a conflict.

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

### Where it lives

On M00's four boxes, offline-first moves a copy of the **data** box onto the **client** and turns the **network** box from a prerequisite into a background worker. Your server changes in one way: two requests carrying the same idempotency key must produce one result. Local data does not all belong in one place.

| Where | Holds | Survives | Use it for |
|---|---|---|---|
| Memory | anything | nothing | this second's UI state |
| localStorage | MBs of text, synchronous | restart | a flag, a theme |
| IndexedDB | tens of MB, structured, async | restart | outbox, lists, drafts |
| Cache Storage | whole HTTP responses | restart | app shell and images |
| Your server | the truth | everything | money or shared data |

Anything the user typed goes to IndexedDB as they type it; anything another person depends on goes to the server. The browser may evict all of it, so it is never the only copy.

**M08 is how to choose plus how to ship a PWA; M12 is the practicum** — a real sync engine, background sync, local migrations, deferred deep links. Chapter 5 builds the service worker.

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

**Offline-first costs real complexity, so scope it.** Pick the three things that must work with no signal — read the menu, draft an order, see your last order. Everything else may say "you are offline".

**Never retry without an idempotency key, and never leave a call without a timeout.** A retry without a key is how one tap becomes three orders, and a request that never resolves is worse than a failure: the user cannot act.

**Choose a conflict rule per data type.** Server wins (the device's copy is a cache — right for menus and prices); last write wins (fine for a private note, destructive for shared edits); per-field merge (more work, fewer lost edits); append-only (two offline claims on the last plate become two claims and one business decision). If nobody chose, the AI chose last-write-wins for you.

**Never pretend a payment succeeded.** A UPI payment or the last plate needs the server's confirmation — say "we will take payment when you are back online". Optimistic UI must be visibly reversible.

### Verify it

Start with the network off, not slow: aeroplane mode, two minutes of use. Then throttle to high latency with packet loss.

**Red flag: the plan assumes a fast connection.** Ask what the user sees at second 10, 30 and 60 of a request that never answers.

**Red flag: retry logic with no idempotency key.** Check the server honours the key, not merely accepts the header.

**Red flag: something that must never be stale, cached.** Prices, stock, balances: ask which fields come from the local copy and for how long.

**Red flag: one URL for the whole app.** Open three screens and watch the address bar; if it never changes you cannot be shared or opened from a push.

Then the duplicate test: offline, tap submit three times, back online, count the rows. The answer must be one.

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
