---
id: m02-08-javascript-events
module: m02-the-web
slug: what-javascript-does
title: "What JavaScript Does: Events, State and Changing the Page"
order: 8
duration_min: 15
is_free: false
mission: false
volatility: low
last_reviewed: 2026-11-16
---

## Key learnings
- Describe every interaction as event, then state change, then re-render
- Identify the one source of truth for any value on screen
- Say why reading a number off the page instead of from state produces wrong totals
- Explain what happens to state when the user reloads, and when that matters
- Name the four interaction red flags that cost money on slow 4G

## Lesson

### The pain

A canteen app's cart shows "2 items · ₹40". The user taps "Add" on a third dish, the number stays at 2, then jumps to 4. The tap added the dish to a list in memory and, separately, incremented the number on screen — and on slow 4G the first tap felt dead, so the student tapped again. The screen says 4, the list holds 3, and the payment total comes from a third place: the prices visible in the DOM.

Three numbers that should be one thing, derived three ways. That is a state problem, and the commonest cause of "sometimes the total is wrong".

### The idea

**Every interactive thing on a page is the same three-step loop: something happens, you change the data you are keeping, and the page is redrawn from that data.**

Event. State. Render — in that order, with the data as the only authority. The screen is a *picture of* the state, never a second copy. If the picture and the data can disagree, you have built a bug that shows up under load and never in testing.

### The analogy

A student says "one samosa" — the **event**. The staff member marks the tally sheet — the **state**. They rub out the total on the board and write the new one — the **render**. It works because the sheet is the authority and the board is only a copy. Update the board without marking the sheet and the next order is wrong; add up the board at closing time and nothing reconciles.

**Where the analogy breaks:** the staff member remembers the conversation. A web page remembers nothing — reload and the tally sheet is blank, unless the state was deliberately written somewhere that survives: browser storage, the URL, or a server. Chapter 10 and M04 are about where it should live.

### Picture it

```
   user taps "Add"
        │
        ▼
  ┌───────────┐     ┌──────────────┐     ┌──────────────┐
  │  EVENT    │────▶│    STATE     │────▶│    RENDER    │
  │  click    │     │ cart = [a,b] │     │ "2 items"    │
  └───────────┘     └──────────────┘     └──────────────┘
                          ▲                      │
                          │                      │ the screen
                    the only                     │ is only a
                    authority                    ▼ picture
                                            what the user sees
```

An event changes the state, the state is the single authority, and the screen is redrawn from it — so the picture can never disagree with the data.

The failure in the pain story has a name: **two sources of truth**. It is right most of the time, breaks under slow networks and double taps, and never reproduces on a developer's laptop. The cure is one line to hold an AI to: **the screen is derived from the state and nothing is read back off the screen.**

### Where it lives

JavaScript runs in the client box from M00 — in the browser, on the user's device, on their CPU. Three consequences:

- It spends the user's battery and processor, a real constraint on a ₹8,000 Android (chapter 16)
- The user can read and modify it, so nothing in it is a security control (M04, M07)
- It downloads before it runs, so buttons needing a 400 KB script are dead for four seconds on slow 4G

Chapter 15's frameworks exist largely to make this loop automatic: change the state and the re-render happens for you.

### Words to own

| Term | Plain meaning | Example |
|---|---|---|
| Event | Something that happened | A click, a keypress, a submit |
| Event listener | Code waiting for a particular event | "When this button is clicked…" |
| Handler | The code that runs when the event fires | The add-to-cart function |
| State | The data your page keeps in memory | The cart's list of items |
| Source of truth | The one place a value is authoritative | The cart list, not the label |
| Render | Drawing the screen from the state | Writing "2 items" |
| Derived value | Something computed from state, never stored | The total price |
| Debounce | Ignore repeat events within a short window | Search-as-you-type |
| Disabled state | A control that cannot be activated right now | Pay button while saving |
| Idempotent | Doing it twice has the same effect as once | Setting a quantity to 3 |
| Optimistic update | Showing success before the server confirms | Risky on payments |

### Tradeoffs and gotchas

**Double taps are not an edge case in India, they are the default.** On slow 4G a tap with no feedback feels broken, so users tap again. Anything that costs money needs two defences: disable the control the instant it is pressed, and make the action safe to repeat (M04's idempotency).

**State disappears on reload, and that is sometimes correct.** A vanishing half-filled form is infuriating; a vanishing cart loses an order; a "payment in progress" flag that survives can leave a user stuck forever. Decide per piece of state: memory, URL, browser storage, or server.

**Optimistic updates lie convincingly.** Showing the item as added before the server confirms feels instant, but when the request fails you must undo the lie visibly. No rollback is worse than a spinner, and never for a payment.

**The back button is part of your app, and the URL is underrated state.** A filter kept in the URL survives reload, can be shared and works with Android's back gesture. Change the screen without changing the URL and back throws the user out.

### Verify it

1. **Ask "what is the source of truth for this number?"** One answer per value, and it must be data, not a DOM element.
2. **Tap twice, fast, throttled to Slow 4G,** then check the data. Two records from one intention is the bug.
3. **Reload mid-form, mid-cart and mid-payment,** and decide whether what survives is what you want.
4. **Press back after every interaction.** Leaving the app where the user expected the previous step is a failure no screenshot shows.
5. **Cross-check the screen against the data** in DevTools: count and total must match the stored list.
6. **Throttle, tap, watch for feedback** within about 100ms. Silence is what causes the second tap.
7. **Interact twenty times without leaving the page.** A handler firing twice is a listener never removed.

Red flags: a value read out of the DOM, a parsed price being the classic · a counter incremented rather than recomputed · a submit button with no disabled state · an optimistic update with no rollback · `setTimeout` used to "wait for the data" · a screen change that never touches the URL.

## Walkthrough
1. Open any shopping or food-ordering site and add an item to the cart. Watch the cart counter in the header closely as you do it.
2. Open DevTools, go to the Application panel, and look under Local Storage and Session Storage for anything that looks like a cart. Note what is stored and in what shape.
3. Reload the page. Did the cart survive? Note the answer, then decide whether that is the behaviour you would want for your own product.
4. Set network throttling to Slow 4G. Add another item and time, roughly, how long it takes for the counter to change. Note whether anything at all happened in the first moment after your tap.
5. Still throttled, tap "Add" twice in quick succession on the same item. Then open the cart and check the quantity. Record whether you got one item or two.
6. Find a filter, a sort order or a tab on the page and change it. Watch the URL in the address bar. Did it change? Now reload and see whether your choice survived.
7. Press the browser back button after changing that filter. Note whether you go back one step within the app, or leave the page entirely.
8. In the Elements panel, find the cart counter element and edit its text to "99" by hand. Then add another item and watch what happens to the number — this tells you whether the counter is recomputed from data or incremented from what was there.
9. Stay on the page and interact with the same control fifteen to twenty times. Watch for any behaviour that starts repeating itself, which suggests listeners accumulating.
10. Write up three findings: where this app's source of truth for the cart appears to live, what happens to it on reload, and whether a double tap on slow 4G creates a duplicate.

## Talk to AI
```prompt
CONTEXT:      I direct AI rather than writing JavaScript. My product is a
              canteen ordering app for students in Nashik on budget Android
              phones, very often on slow 4G. The flow I am working on: a
              student sees six dishes, adds items to a cart, adjusts
              quantities, sees a running total in rupees, and then taps a
              Pay button that will eventually trigger a UPI payment. Prices
              are stored as whole paise, never as rupee decimals. There is
              no backend yet; assume the cart lives in the browser.

GOAL:         An interaction design for this cart with exactly one source
              of truth, which cannot show a total that disagrees with its
              contents, and which does not create duplicate items when a
              student on slow 4G taps twice.

REQUIREMENTS: 1. State the single source of truth for the cart, and say
                 what shape it has
              2. Every number on screen — item count, line totals, grand
                 total — must be derived from that state on each render,
                 never incremented and never read back from the page
              3. Every control that adds, changes or pays must enter a
                 visibly disabled or pending state within 100ms of the tap
              4. Decide, per piece of state, where it lives: memory, the
                 URL, or browser storage — and justify each choice against
                 what should survive a reload
              5. The Android back gesture must move the user one step back
                 inside the flow, not out of the app
              6. Define what happens on a failed action, including how an
                 optimistic update is rolled back visibly

CONSTRAINTS:  Plain JavaScript, no framework yet. Money is integer paise
              throughout; never floating point rupees. Do not read any
              value out of the DOM to compute anything. Do not use
              setTimeout to wait for data. Do not use an optimistic update
              anywhere near payment. Must work on a 360px screen one-handed.

NON-GOALS:    No real payment integration. No login. No server. No styling
              work. Do not introduce a state-management library.

ACCEPTANCE:   - After any sequence of taps, the displayed count and total
                exactly match the stored cart
              - Two fast taps on Add produce one item, not two
              - Throttled to Slow 4G, something visible changes within
                100ms of every tap
              - Reloading mid-cart produces the behaviour I chose in
                requirement 4, and I can state what that is
              - The back gesture after changing a filter returns to the
                previous filter, not to the previous website
              - Searching the code for code that reads text out of the page
                to compute a number returns nothing

ASK:          Before any code, give me a table of every piece of state in
              this flow: its name, its shape, where it lives, what happens
              to it on reload, and what is derived from it rather than
              stored. Flag anything I have asked for that is contradictory
              or that you think is the wrong call, and say why. Wait for my
              approval.
```

## Stuck? Ask AI
```prompt
Explain "two sources of truth" using a non-software example, then show me the same cart counter bug written two ways: once with the count incremented directly and once with the count derived from the cart. Walk me through exactly what the user sees in each version when they tap twice on a slow connection.
```

```prompt
I do not understand where state should live. Give me eight pieces of state from a food-ordering app — a selected filter, a half-typed search, a cart, a payment-in-progress flag, and four more — and ask me, for each, whether it should live in memory, in the URL, in browser storage or on the server. Make me justify each one before you tell me what you would choose and why.
```

```prompt
Explain what an optimistic update is, why it feels good on slow 4G, and exactly what must exist for it to be safe. Then give me three features where it is a good idea and three where it is dangerous, and explain what the user experiences when an optimistic update fails with no rollback.
```

## Quiz

### Q1 · L2
What is the correct order of the interaction loop?
- (a) Render, then event, then state
- (b) Event, then state, then render ✅
- (c) State, then render, then event
- (d) Event, then render, then state
> Something happens, you update the data, then you redraw the screen from that data. Updating the screen before or instead of the data is what creates a second source of truth.

### Q2 · L3
A cart counter is sometimes right and sometimes wrong, only on slow connections, and never on the developer's laptop. What is the most likely cause?
- (a) The server is returning the wrong number
- (b) The counter is updated independently of the cart, so the two can disagree ✅
- (c) The browser is caching the old counter value
- (d) The CSS is hiding part of the number
> Two sources of truth produce exactly this signature: intermittent, load-dependent, unreproducible locally. Deriving the counter from the cart on every render removes the possibility entirely.

### Q3 · L3
On slow 4G, a student taps "Pay ₹240" and nothing visibly changes for three seconds. What will they do, and what must the page have done?
- (a) Wait patiently; nothing was needed
- (b) Tap again — so the control must disable immediately and the action must be safe to repeat ✅
- (c) Reload the page; the page should have auto-refreshed
- (d) Nothing, because the browser blocks repeat taps automatically
> Silence after a tap reads as failure. Disabling the control is the browser-side defence and takes two lines; making the action safe to repeat is the server-side one, covered in M04.

### Q4 · L4
An AI computes the cart total by reading the price text out of each visible row on the page. Why reject this?
- (a) Reading the DOM is slower than reading a variable
- (b) The page is a picture of the state, so items not currently rendered are silently excluded from the total ✅
- (c) Price text cannot be converted to a number
- (d) It will break if the user changes the font size
> It makes the screen the source of truth. Anything off-screen, filtered out or not yet rendered simply does not count — and the total is wrong in a way that depends on scroll position.

### Q5 · L5
You are designing a filter on a dish list. Which choice best serves an Android user who relies on the back gesture?
- (a) Keep the filter in memory only, so reload gives a clean slate
- (b) Store the filter in browser storage so it persists forever
- (c) Put the filter in the URL, so it survives reload, is shareable, and back returns to the previous filter ✅
- (d) Re-request the filtered list from the server on every render
> URL state gets reload-survival, sharing and correct back-button behaviour for free, with no storage code. It is the option an AI least often proposes unless you ask for it.

## Go deeper
- [MDN: Introduction to events](https://developer.mozilla.org/en-US/docs/Learn_web_development/Core/Scripting/Events)
- [web.dev: Interaction to Next Paint, and why feedback latency matters](https://web.dev/articles/inp)
