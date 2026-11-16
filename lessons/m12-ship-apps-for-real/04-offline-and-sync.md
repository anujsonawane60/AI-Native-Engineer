---
id: m12-04-offline-and-sync
module: m12-ship-apps-for-real
slug: offline-and-sync
title: "Offline and Sync When the Network Is Bad"
order: 4
duration_min: 35
is_free: false
mission: true
volatility: low
last_reviewed: 2026-11-16
---

## Key learnings
- Design reads from a local store so the app opens useful with no network
- Put writes in a durable queue that drains when the network returns
- Use an idempotency key so a retry never creates a duplicate
- Decide, per feature, which writes must never be queued
- Spot an AI design that assumes the network is always there

## Lesson

### The pain

A student boards the Nashik–Mumbai bus and opens your app. Mobile data flickers between one bar and none for the next forty minutes.

Here is what she sees. The app opens to a blank screen with a spinner, because every screen fetches on mount and nothing is stored locally. After eleven seconds it shows "Something went wrong". She pulls to refresh. It spins again. She fills in an order anyway — the form is local, after all — taps submit, and the button spins for thirty seconds before the request times out. The form clears. Her typing is gone.

Twenty minutes later the signal comes back. Nothing happens, because your app only tries once and only when the user taps. She has to notice the signal herself, reopen the app and start again.

Now the failure that costs you money. Another user taps submit on a weak connection. The request actually reached your server and created the order, but the response never got back, so the app showed an error. He taps again. You now have two orders, two kitchen tickets, and one angry customer. That bug exists in a startling number of student apps and it is entirely preventable with one extra field.

### The idea

**Treat the network as an occasional visitor rather than a permanent fixture. Reads come from a local copy you refresh when you can; writes go into a durable queue that drains by itself.**

Three consequences. Opening the app never shows a spinner on an empty screen, because yesterday's data is on the device. Submitting never loses the user's typing, because the write is saved locally before any request is attempted. And retrying is safe, because every queued write carries a key that lets your server recognise the second attempt as the same attempt.

This is not an exotic architecture. It is the pattern every app you respect already uses, and WhatsApp is the reference your users already understand: a message typed with no signal shows a clock, then one tick when it reaches the server. The user is never lied to and never loses their words.

### The analogy

The canteen loses power at 1:05 pm, right at the rush. The till is dead.

The staff do not send everyone home. They take orders on paper chits, hand over food, and keep the chits in a clip. When power returns at 1:40, somebody enters all forty chits into the till. Service continued; the record caught up.

That is offline-first. The chit is the queued write. The clip is the durable queue. The person entering them later is your sync process.

**Where the analogy breaks**, and this is the interesting part. Two customers wrote a chit for the last plate of biryani. On paper, the cook notices at the counter and resolves it with a human conversation. In software, both chits arrive at the server forty minutes later and something must decide. There is no obvious right answer — and pretending there is one is the most common mistake in offline design. Your server has to be the authority for anything scarce, and your app has to be able to tell the user, politely and later, that their chit could not be honoured.

### Picture it

```
  READS                                 WRITES
┌──────────┐                       ┌────────────────────┐
│  screen  │◀── instant ───┐       │ user taps "Submit" │
└──────────┘               │       └─────────┬──────────┘
      ▲                    │                 │ saved first
      │ refresh when       │                 ▼
      │ network returns ┌──┴──────┐   ┌──────────────┐
      │                 │ local   │   │ write queue  │
┌─────┴────┐  JSON      │ store   │   │ + idem. key  │
│  server  │───────────▶│ (SQLite)│   └──────┬───────┘
└──────────┘            └─────────┘          │ drains on
      ▲                                      │ reconnect
      └──────────── one request per chit ◀───┘
```

Screens read instantly from a local store that the network refreshes in the background, while every write is saved locally with an idempotency key before a single request is attempted.

### Where it lives

M02 gave you service workers and the web version of this idea; M05 gave you caching, stale data and offline tolerance as concepts. This chapter is both of those on a device where the network is genuinely worse and the storage is genuinely better — a phone has a real SQLite database, which a browser only approximates.

On the server side, the idempotency key is a column with a unique constraint, which is M04's territory. The retry schedule is the exponential backoff from M05's rate-limiting chapter, pointed the other way: your client backing off rather than your server pushing back.

Three mechanisms, in the order you build them:

**A local store.** Key-value storage is fine for small things like the last-seen cursor; a device SQLite database is the right answer for lists of records. Write every successful fetch into it, and read screens from it.

**Optimistic UI.** When the user submits, update the screen immediately as if it succeeded, with a visible pending marker. If the write later fails permanently, revert and tell them clearly. The pending marker is not decoration — without it you have lied.

**A drain loop.** On app start, on network reconnect, and on a timer, attempt the oldest queued write. On success, remove it. On a network failure, leave it and back off. On a definitive rejection from the server — the biryani is gone — remove it and surface it to the user.

### Words to own

| Term | Plain meaning | Example |
|---|---|---|
| Offline-first | The local copy is what screens read | App opens with yesterday's list |
| Write queue | Durable list of pending changes | Three unsent orders |
| Idempotency key | Unique id so a retry is not a new write | One UUID per submit tap |
| Optimistic UI | Show success before the server agrees | Order appears with a clock icon |
| Exponential backoff | Wait longer after each failed retry | 2s, 4s, 8s, 16s |
| Conflict | Two changes to the same thing | Both users claim the last plate |
| Last-write-wins | Newest timestamp overwrites | Fine for a profile name |
| Server authority | The server decides, client obeys | Stock, money, bookings |
| Sync cursor | Marker of how far you have caught up | "Changes since 14:02" |
| Tombstone | A record that marks a deletion | So deletes sync too |
| Stale-while-revalidate | Show old data, fetch new quietly | Menu from last night, then today's |

### Tradeoffs and gotchas

**Not everything should work offline, and payments least of all.** Never queue a payment. A UPI transaction is an action in the real world with a real bank behind it, and a queued payment that fires forty minutes later — possibly twice, possibly after the user gave up and paid in cash — is worse than a clear error. The correct offline behaviour for a payment is to refuse politely and say what to do: "You need a connection to pay. Your cart is saved." Same for anything that books a scarce resource or sends a message to a third party on the user's behalf.

**Idempotency has to be generated on the device, not on the server.** The key must be created when the user taps submit and reused on every retry of that same tap. If the client generates a fresh key per retry, you have achieved nothing. A unique constraint on the server, plus returning the original result when the key repeats, is what makes the double-tap safe.

**Last-write-wins is fine until it is catastrophic.** Overwriting a profile nickname by timestamp is sensible. Overwriting an inventory count by timestamp loses sales silently. The rule of thumb: last-write-wins for things only one person owns, server authority for anything counted, booked or paid for.

**Storage is not free on a ₹8,000 phone.** A device with 32 GB and 3 GB of photos has less headroom than you think. Cache what the user actually looks at, cap how much you keep, and clean up. Do not mirror your whole database onto every phone.

**Your users pay for the data your sync uses.** A drain loop that polls every ten seconds while the signal is weak burns battery and data for nothing. Back off properly, sync on reconnect rather than on a tight timer, and prefer fetching changes since a cursor over refetching whole lists.

**Clocks on phones are wrong.** Device time can be minutes or days off, and users change it. Never trust a device timestamp for ordering, conflict resolution or anything security-related. The server stamps the time.

**Pending must be visible.** The worst version of optimistic UI is the one that looks identical to success. Users will close the app believing the order is placed. A small clock icon costs nothing and prevents a support conversation.

### Verify it

**Red flag: a design that assumes a fast connection.** The giveaway is that every screen fetches on mount and renders a spinner until it resolves, with no local store anywhere in the plan. Ask the direct question: "with the network switched off, what exactly does each of my screens show?" If the answer is a spinner or an error, the design is not finished.

**Red flag: no idempotency key.** Ask explicitly what happens if the server receives the request but the response is lost and the user taps again. A correct answer names a client-generated key, a unique constraint, and returning the original result on the second attempt. An answer that says "the user can delete the duplicate" is not an answer.

**Red flag: a queued payment.** Read the feature list the AI produced and check which writes it decided to queue. If a payment, a booking or anything with real-world scarcity is in the queue, that is a bug with money in it. Insist on an explicit list of writes that must *not* be queued.

**Red flag: retry with no backoff and no cap.** A loop that retries every second forever will flatten the battery and may get you rate-limited by your own API. Look for increasing delays and a maximum attempt count, plus a decision about what happens to a write that never succeeds.

**Red flag: optimistic UI with no pending state and no revert path.** Check both halves. Does the screen show that this item is not yet saved? And if it fails permanently, what does the user see and what happens to their data?

**Test it by breaking the network for real, not by mocking it.** Turn on airplane mode, do the thing, close the app completely, turn the network back on, reopen, and confirm the write arrived exactly once. Then do the harder test: enable airplane mode *in the middle* of a submit. Then the hardest: on a real weak signal, tap submit twice.

## Walkthrough
1. List every screen in your app and every write action. For each screen write what it should show with no network; for each write, whether it may be queued.
2. Mark the writes that must never be queued — payments, bookings, anything claiming a scarce thing — and write one sentence of refusal copy for each.
3. Pick the single most important read screen. Have AI add a local store for it and make the screen read from the store, with the network refreshing in the background.
4. Open that screen on a real phone in airplane mode with data already cached. Confirm it renders instantly and says clearly when the data was last updated.
5. Pick the single most important queueable write. Have AI add a durable queue that saves the write before any request is attempted, with a device-generated idempotency key.
6. Add the server side: a unique constraint on the idempotency key, and a handler that returns the original result instead of creating a second record.
7. Add the drain loop: on app start, on reconnect, and with exponential backoff. Cap the attempts and decide what happens to a write that never succeeds.
8. Show pending state in the UI for queued writes, visibly different from saved. Have someone outside your team look at the screen and tell you which items are saved.
9. Test the sequence properly on a real phone: airplane mode, submit, force-quit the app, turn the network on, reopen. Confirm exactly one record exists on the server.
10. Test the double-submit: on a genuinely weak signal, tap submit twice quickly. Confirm the server has one record, not two.
11. Record in your decision log which writes you queue, which you refuse, how you resolve conflicts, and the data cost of one sync cycle.

## Talk to AI
```prompt
CONTEXT:      React Native app built with Expo, Android first, for students
              in India on budget phones and patchy expensive mobile data,
              including long stretches with no signal at all. Product:
              [one sentence]. API: [stack]. Database: [yours].
              Our screens today: [list them]
              Our write actions today: [list them, including any payment]
              Today every screen fetches on mount and shows a spinner, and
              every write is a single request with no retry.

GOAL:         The app opens useful with no network, never loses a user's
              input, and a retried write can never create a duplicate.

REQUIREMENTS: 1. A table of my screens with what each shows offline, and a
                 table of my writes marked queueable or must-refuse
              2. For every must-refuse write, the exact copy the user sees
                 and what happens to their input
              3. A local store design: what goes in key-value storage, what
                 goes in device SQLite, and a cap on how much we keep
              4. A durable write queue with a client-generated idempotency
                 key created at the moment of the tap and reused on retries
              5. The server-side change that makes the retry safe, stated
                 precisely, including what the second request receives
              6. A drain schedule with exponential backoff, an attempt cap,
                 and a defined outcome for a write that never succeeds
              7. A conflict rule per data type: say where last-write-wins
                 is acceptable and where the server must be the authority
              8. An estimate of bytes transferred per sync cycle

CONSTRAINTS:  Free tiers only. Device may have under 2 GB of free storage.
              Device clocks cannot be trusted. Users pay per megabyte.
              Assume 300 ms round trips and frequent total loss of signal.

NON-GOALS:    Do not introduce a sync framework or a backend-as-a-service.
              Do not redesign my API. Do not queue anything involving
              money. Do not add background sync beyond app start and
              reconnect.

ACCEPTANCE:   - Every screen in my table has a defined offline appearance
                that is not a spinner
              - At least one of my writes is classified as must-refuse
              - The idempotency key is generated on the device at tap time
              - You have described the exact test sequence that proves no
                duplicate is created, including force-quitting the app
              - Pending writes are visually distinct from saved ones
              - No device timestamp is used to resolve a conflict

ASK:          Give me the two classification tables first, and the
              must-refuse list. Do not write any code until I have agreed
              which writes are queueable.
```

## Mission

**Objective.** One real write path in your own product survives airplane mode end to end, and a double tap on a weak network produces exactly one record on your server.

**Steps.**
1. **Frame** — name the one write in your product where losing the user's input hurts most, and the one write that must never be queued. One sentence each.
2. **Classify** — produce the two tables: every screen with its offline appearance, every write marked queueable or must-refuse. This is the artefact your teammates review.
3. **Brief** — write a Director's Brief for making your chosen write survive the network being gone. CONSTRAINTS must include the untrusted device clock and metered data; NON-GOALS must exclude queuing anything with money in it.
4. **Direct** — have AI build the local save, the queue, the idempotency key and the drain loop, one step at a time with a check after each.
5. **Harden the server** — add the unique constraint and make the repeat request return the original result rather than creating a second record.
6. **Make pending visible** — show queued items differently from saved ones, and have someone outside your team point at the screen and say which is which.
7. **Verify, on a real phone** — airplane mode, submit, force-quit, reconnect, reopen: exactly one record. Mid-submit airplane mode: no lost input. Weak signal, double tap: one record.
8. **Measure** — record the bytes your sync cycle transfers and the time from reconnect to the queue being empty.
9. **Document** — one decision-log entry naming your conflict rule, your must-refuse list, and the tradeoff you accepted.

**Upload.**
- Your Director's Brief → post caption or an attached file
- Your two classification tables → screenshot or text
- The AI conversation → share link in `ai_chat_url`
- Proof it works → a screen recording on a real phone showing airplane mode, submit, force-quit, reconnect and the item syncing
- Proof of no duplicate → a screenshot of your server data after the double-tap test
- Your pending-state screen, plus what your outside tester said about it
- Reflection, 150 words max → one tradeoff you chose, one thing the AI got wrong

**Self-check.** Tick every box before you submit.
- [ ] Every screen in my table has an offline appearance that is not a spinner
- [ ] At least one write is classified must-refuse, with refusal copy written
- [ ] No payment or booking is in my queue
- [ ] The idempotency key is generated on the device when the user taps
- [ ] My server has a unique constraint, and the repeat request returns the original result
- [ ] I force-quit the app mid-queue and the write still arrived
- [ ] I double-tapped on a real weak signal and there is exactly one record
- [ ] Pending items are visibly different from saved items to a stranger
- [ ] My retries back off and have a cap, and I know what happens at the cap
- [ ] No device timestamp decides any conflict in my design

**Stretch.** Add a sync cursor so refreshing fetches only what changed since your last successful sync, and measure how many bytes that saves on one refresh.

## Stuck? Ask AI
```prompt
Explain idempotency to someone who has never written server code, using a real-world example that is not about bank transfers. Then describe exactly what happens, step by step, when a phone sends a request, the server processes it, the response is lost, and the user taps the button again — once without an idempotency key and once with one. Ask me to predict each outcome before you reveal it.
```

```prompt
Give me six features from a typical student app and ask me, one at a time, whether that feature's writes should be queued when offline or refused with a clear message. Do not tell me your answers first. After I answer each one, tell me if you disagree and why, and name the specific real-world harm of getting it wrong.
```

```prompt
My app shows cached data offline but the user cannot tell whether what they are looking at is current. Give me three different ways to communicate freshness and pending state in a mobile UI, with the tradeoffs of each for users who are not technical, and tell me which you would choose for a budget Android phone with a small screen.
```

## Quiz

### Q1 · L2
In an offline-first mobile app, where do screens read their data from?
- (a) Directly from the server on every mount
- (b) From a local store, which the network refreshes in the background ✅
- (c) From an in-memory variable that resets on app close
- (d) From the push notification payload
> Reading locally is what makes the app open useful with no network. The fetch becomes a background refresh rather than a blocking requirement.

### Q2 · L4
A user taps submit on a weak signal. The server creates the order but the response is lost, so the app shows an error and the user taps again. What prevents a duplicate?
- (a) The server rejecting requests that arrive too close together
- (b) A device-generated idempotency key, reused on the retry, with a unique constraint on the server ✅
- (c) Disabling the submit button for five seconds
- (d) Checking whether an identical order already exists before inserting
> The key must be created at the moment of the tap and reused, so the server recognises the second request as the same request and returns the original result rather than inserting again.

### Q3 · L4
Which write should never go into an offline queue?
- (a) Saving a draft note
- (b) Marking a task complete
- (c) A UPI payment ✅
- (d) Updating your display name
> A payment is an action in the real world with a bank behind it. Firing it forty minutes late, possibly after the user paid in cash instead, is worse than a clear refusal with the cart saved.

### Q4 · L4
Your team resolves all sync conflicts by last-write-wins using the device's timestamp. Where does this fail worst?
- (a) A user's display name
- (b) A personal draft only that user edits
- (c) A count of remaining stock ✅
- (d) A read-only list of categories
> Last-write-wins is acceptable for things one person owns. Anything counted, booked or paid for needs server authority — and device clocks are unreliable enough that they must never decide anything.

### Q5 · L5
An AI proposes an offline design where submitted items appear in the list immediately and look exactly like saved items. What is the strongest objection?
- (a) Optimistic UI is always wrong
- (b) It tells the user their write succeeded when it has not, so they close the app believing the job is done ✅
- (c) It makes the list render more slowly
- (d) It prevents the use of exponential backoff
> Optimistic UI without a visible pending state is a lie with no cost to implement away. A small clock icon is the difference between informed and misled, and it also gives you somewhere to show a permanent failure.

## Go deeper
- [Expo: SQLite for local storage](https://docs.expo.dev/versions/latest/sdk/sqlite/)
- [web.dev: offline UX considerations](https://web.dev/articles/offline-cookbook)
