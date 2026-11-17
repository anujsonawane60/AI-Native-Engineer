---
id: m02-10-async-fetch
module: m02-the-web
slug: async-promises-fetch
title: "Async, Promises and fetch: Why the Page Does Not Freeze"
order: 10
duration_min: 15
is_free: false
mission: false
volatility: low
last_reviewed: 2026-11-16
---

## Key learnings
- Explain why the browser starts slow work and carries on instead of waiting
- Design every data-loading screen for four states, not two
- Insist on a timeout and a visible error path on every request
- Spot a race condition from its symptoms rather than from the code
- Name the five async red flags that break on slow 4G and never on a laptop

## Lesson

### The pain

Demo day, crowded venue 4G. A judge taps "Pay ₹240", nothing happens, so she taps twice more. Four seconds later the screen says "Order placed" and the dashboard shows three orders from the same student.

Earlier she had tapped "South Indian" then quickly "Chinese". Chinese appeared, then filled with South Indian dishes, because the first request took a slower route and arrived last: the page showed the most recently *arrived* data, not the most recently *requested*. Both bugs are invisible on a laptop and guaranteed on Nashik 4G at 1pm.

### The idea

**Fetching data over a network takes an unknown amount of time, and the browser refuses to sit still while it happens — so you start the work, carry on, and handle the answer when it turns up.**

That is all "asynchronous" means, and the consequence for a director is about screens, not code. The answer arrives later, so there is a period where you have asked and do not know — and a chance the answer is "no". Every data screen has **four** states, not two.

### The analogy

You order a dosa and pay. Rather than blocking the counter until it is cooked, you take a **token** and sit down; later your number is called, or the staff announce the dosa is finished and refund you. The token is a promise: something you hold now standing for an answer later, with three lives — waiting, fulfilled, rejected. There is no fourth outcome, and none where you never find out.

**Where the analogy breaks:** the canteen calls numbers roughly in order; the network does not. Two requests sent a second apart can return in either order, so "the newest answer wins" must be arranged deliberately. The filter bug is normal network behaviour.

### Picture it

```
  SYNCHRONOUS (if the browser waited)
  tap ──▶ [████████ fetch: 4 seconds ████████] ──▶ draw
          page frozen, taps ignored, scrolling dead

  ASYNCHRONOUS (what actually happens)
  tap ──▶ start fetch ──┐
          draw "Loading…"│  page stays alive, scrolls, responds
          disable button │
                         └──▶ answer arrives ──▶ draw dishes
                              or fails ────────▶ draw error
```

If the browser waited, the whole page would be dead for four seconds; instead it starts the request, immediately shows a loading state, and draws either the result or an error when the answer arrives.

The four states every data screen owes its users:

| State | What the user must see |
|---|---|
| Loading | A change within 100ms — skeleton, spinner, disabled button. This prevents the triple-tap |
| Success | The data, drawn |
| Empty | A sentence saying there are zero dishes, not a blank area that reads as a bug |
| Error | Plain language and a retry: "Could not load the menu. Try again." |

Empty and error are the two that get skipped, and the two your users on bad connections see most.

### Where it lives

`fetch` sits on the line between M00's client box and server box. The request itself — method, status, headers — is chapter 13; what the server does with it is M04; rate limiting is M05.

Carry this forward: **the network is the least reliable part of your product and the only part you do not control.** Your code, server and database can be fixed. A student's 4G in a basement cannot.

### Words to own

| Term | Plain meaning | Example |
|---|---|---|
| Synchronous | One thing at a time; nothing else happens meanwhile | Blocking the page |
| Asynchronous | Start it, carry on, handle the result later | Loading the menu |
| Promise | A token standing for an answer that has not arrived | What `fetch` gives back |
| Pending | The answer has not arrived yet | Show the loading state |
| Fulfilled | The answer arrived successfully | Draw the dishes |
| Rejected | It failed — network down, server refused | Draw the error state |
| `await` | "Pause this piece of work until the token resolves" | Reading the response |
| Timeout | Give up after a set time rather than waiting forever | 10 seconds |
| Retry | Try again after a failure, usually after a short wait | With backoff |
| Race condition | Two answers arriving in an order you did not plan for | The filter bug |
| Abort | Cancel a request you no longer care about | On filter change |
| Idempotent | Safe to repeat without extra effect | Setting, not adding |

### Tradeoffs and gotchas

**No timeout means waiting forever.** A `fetch` with no timeout on a dead connection sits in "loading" until the user reloads. Ten seconds is a reasonable ceiling for a list.

**Retry is only safe if the action is.** Auto-retrying "place order" is how you get three orders; retrying "load the menu" is free. The line is whether repeating the action repeats its effect; the server-side fix is M04's idempotency keys.

**The newest request must win, not the newest answer.** Cancel the previous request, or tag requests and ignore stale answers. An AI will not do this unless asked.

**A failed `fetch` and a failed *request* differ.** `fetch` only rejects when the network failed; a 404 or 500 is a *successful* fetch carrying bad news, and code that only handles rejection renders the 500 page as your menu. Chapter 13 covers the statuses.

**`await` in a loop turns forty 200ms requests into eight seconds.** Start independent requests together, and reserve the loading space so the layout does not jump.

### Verify it

Every check needs Slow 4G and 4x CPU throttling; on an unthrottled laptop these bugs are invisible.

1. **Confirm all four states exist.** Ask for a screenshot of each; what cannot be screenshotted does not exist.
2. **Go offline mid-request.** The app must say something a student understands, not hang on a spinner.
3. **Triple-tap every action that writes,** then check the data: one intention, one record.
4. **Switch filters fast, five times over.** A first result landing after the second is a race condition.
5. **Check the timeout exists.** The request must give up with a message rather than hang.
6. **Count requests on one page load.** Forty requests for one list is a loop that should have been one request.

Red flags: only a success state · `fetch` with no timeout · automatic retry on a write · no cancellation when a new request supersedes an old one · `await` inside a loop · error handling that ignores the status code · raw error text shown to the user. ACCEPTANCE: "I can screenshot all four states on Slow 4G."

## Walkthrough
1. Open a data-heavy site — a food delivery app or a job board — with DevTools open on the Network panel, filtered to Fetch/XHR.
2. Reload with no throttling and note how many requests fire and the total time. Then set throttling to Slow 4G and 4x CPU and reload again. Note the new total.
3. On the throttled reload, watch the screen closely for the first second. Screenshot what the page shows before the data arrives. Describe it: skeleton, spinner, blank, or old content.
4. Measure, roughly, how long after the reload the first real content appears, and whether the layout jumped when it did.
5. Find a filter or category control. Tap one option, then immediately tap a second. Repeat five times and watch whether the wrong results ever appear after the right ones.
6. Use the site's search or filters to produce zero results. Screenshot what you get, and judge whether a first-time user would understand that this is an answer rather than a fault.
7. Set DevTools to Offline, then trigger a data load. Screenshot the result. Note whether the message is in plain language and whether there is a way to retry.
8. Go back online, start a slow request, and then quickly navigate away or change the filter. Check in the Network panel whether the abandoned request was cancelled or allowed to complete.
9. Find a button that writes something — add to cart, save, favourite. On Slow 4G, tap it three times fast and then check the result. Record whether you created one thing or three.
10. Still on Slow 4G, time how long from your tap until anything at all changes on screen. Anything over about 100ms is what causes repeat taps.
11. Write up five findings with screenshots: which of the four states this app actually implements, whether it has a race condition, and whether its primary write action is safe to double-tap.

## Talk to AI
```prompt
CONTEXT:      I direct AI rather than writing code. My product is a canteen
              ordering app for students in Nashik on budget Android phones,
              very often on congested 4G where a request can take 4 to 8
              seconds or fail outright. Two flows matter: loading a list of
              dishes filtered by category, and placing an order. Prices are
              integer paise. There is no idempotency support on the server
              yet and I know that is a gap.

GOAL:         A loading and failure design for both flows that behaves
              correctly on a bad connection, cannot create duplicate orders
              from repeat taps, and never shows stale results after a newer
              request.

REQUIREMENTS: 1. For each flow, specify all four states — loading, empty,
                 error, success — including the exact words the user sees
                 in the empty and error cases, in plain English
              2. A timeout on every request, with the number stated and
                 justified separately for the list and for the order
              3. A rule for retry: which flow may retry automatically,
                 which may only retry on an explicit user tap, and why
              4. Explicit handling of the case where the newest request
                 must win: say how an older in-flight response is prevented
                 from overwriting a newer one
              5. Visible feedback within 100ms of every tap, and a disabled
                 state on the order button from the moment it is pressed
              6. A note on which of these protections are browser-side only
                 and still need a server-side counterpart

CONSTRAINTS:  Plain JavaScript, no framework and no libraries. Do not auto-
              retry anything that writes. Do not use setTimeout to wait for
              data to arrive. Loading states must reserve the same space as
              the content they replace, so the layout does not jump. Error
              messages must never show raw technical text to a student. Do
              not start more than six requests for one screen.

NON-GOALS:    No real payment integration. No caching layer. No offline
              support beyond a clear error message. Do not redesign the
              visual layout. Do not add a state-management library.

ACCEPTANCE:   - Throttled to Slow 4G, I can screenshot all four states for
                both flows
              - Tapping the order button three times in two seconds creates
                exactly one order
              - Switching category twice quickly never ends up showing the
                first category's dishes
              - With DevTools set to Offline, both flows show a plain
                message and a way to try again
              - On the slowest throttling setting, every request eventually
                gives up with a message rather than hanging
              - Something visible changes within 100ms of every tap
              - The layout does not shift when loading content is replaced

ASK:          Before any code, give me a table with one row per flow and
              one column per state, saying exactly what is on screen in
              each cell. Then list every protection you are adding and mark
              it "browser only" or "needs a server counterpart". Tell me
              which of my requirements cannot be fully satisfied without
              the server change I said is missing. Wait for my approval.
```

## Stuck? Ask AI
```prompt
Explain what a promise is using an analogy that is not about tokens, restaurants or deliveries. Then walk me through the three states a promise can be in and what the user should see on screen during each. Finally give me five situations and ask me which state the page is in and what it should be showing, without telling me the answers first.
```

```prompt
Explain a race condition as if I have never programmed. Use a concrete example of two requests sent one second apart arriving in the wrong order, and show me step by step what the user sees. Then explain the two standard ways to make the newest request win, and the tradeoffs of each.
```

```prompt
I do not understand why a 404 or a 500 from the server is not treated as a failure by fetch. Explain what fetch considers a failure and what it does not, why it was designed that way, and exactly what must be checked in addition. Then give me four responses and ask me whether each one is a success, a fetch failure, or a successful fetch carrying bad news.
```

## Quiz

### Q1 · L2
Why does the browser not simply wait for a network request to finish?
- (a) Because network requests always fail
- (b) Because the page would be frozen and unresponsive for the whole wait ✅
- (c) Because the server would reject a request that waits
- (d) Because waiting uses more mobile data
> On a 4-second request, waiting means 4 seconds of dead scrolling, ignored taps and no feedback. Starting the work and carrying on is what keeps the page alive.

### Q2 · L3
Which state do data-loading screens most often omit, and who sees it most?
- (a) Success, seen by everyone
- (b) Loading, seen by desktop users
- (c) Empty and error, seen most by users on poor connections ✅
- (d) Pending, which the browser handles automatically
> An AI asked for a list screen builds the success case. Users on congested 4G in a basement classroom live in the error state, and a filter with no matches produces the empty one every day.

### Q3 · L3
A student taps "Pay ₹240" three times on slow 4G and three orders appear. Which two defences were missing?
- (a) A faster server and a bigger timeout
- (b) An immediate disabled state on the button, and a server-side guarantee that repeating the request is safe ✅
- (c) A loading spinner and a retry button
- (d) Request cancellation and a shorter timeout
> The browser-side fix stops most of it in two lines. The server-side guarantee — M04's idempotency — is what makes it actually impossible, because a request can always be sent twice.

### Q4 · L4
A user taps "South Indian", then immediately "Chinese", and ends up looking at South Indian dishes. What happened?
- (a) The server returned the wrong category
- (b) The first request arrived last and overwrote the newer result ✅
- (c) The filter button had a double-click handler
- (d) The browser cached the first response
> Two requests can return in any order. The page was showing the most recently *arrived* data instead of the most recently *requested* data — which is why the old request must be cancelled or its answer ignored.

### Q5 · L5
An AI says it "added error handling" and the code only handles the case where `fetch` itself rejects. What is still broken?
- (a) Nothing — that is the only way a request can fail
- (b) A 404 or 500 is a successful fetch carrying bad news, so the error page's content gets rendered as if it were your data ✅
- (c) Timeouts will now be ignored
- (d) The retry logic will fire twice
> `fetch` rejects only on a network-level failure. The status code must be checked separately, or a server error is treated as a valid response — which is how students end up displaying an error page's text inside their menu.

## Go deeper
- [MDN: How to use promises](https://developer.mozilla.org/en-US/docs/Learn_web_development/Extensions/Async_JS/Promises)
- [MDN: Using the Fetch API, including status checking](https://developer.mozilla.org/en-US/docs/Web/API/Fetch_API/Using_Fetch)
