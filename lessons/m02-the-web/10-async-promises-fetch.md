---
id: m02-10-async-fetch
module: m02-the-web
slug: async-promises-fetch
title: "Async, Promises and fetch: Why the Page Does Not Freeze"
order: 10
duration_min: 25
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

Demo day. A judge opens the canteen app on her phone, on the venue's crowded 4G.

She taps "Pay ₹240". Nothing happens. She taps again. Then a third time. Four seconds later the screen shows "Order placed" — and the canteen's dashboard shows three orders for ₹240 from the same student.

On the menu screen earlier, something else had gone wrong and nobody noticed. She had tapped "South Indian", then quickly "Chinese". The screen showed Chinese for a moment and then filled with South Indian dishes, because the first request — issued earlier, over a slower route — arrived last and overwrote the newer answer. The page was not broken. It was faithfully displaying the most recently *arrived* data rather than the most recently *requested* data.

Both of these are invisible on a fast laptop. Both are guaranteed on Nashik 4G at 1pm.

### The idea

**Fetching data over a network takes an unknown amount of time, and the browser refuses to sit still while it happens — so you start the work, carry on, and handle the answer when it turns up.**

That is all "asynchronous" means. The consequence for you as a director is not about code; it is about screens. Because the answer arrives later, there is a period where you have asked and do not yet know, and a possibility that the answer is "no". Every screen that loads data therefore has **four** states, not two: loading, empty, error, and success. An AI asked for a menu screen will build you one.

### The analogy

The canteen token system.

You order a dosa and pay. You do not stand at the counter blocking everybody behind you until it is cooked — you get a **token** and sit down. The counter keeps serving. Later your number is called and you collect your dosa, or the staff announce that the dosa is finished and refund you.

The token is a promise: a thing you hold now that stands for an answer later. It has exactly three possible lives. It is still waiting. It is fulfilled and you have your food. Or it is rejected and you have bad news. There is no fourth outcome, and crucially there is no outcome where you never find out.

**Where the analogy breaks:** the canteen calls numbers roughly in order. The network does not. Two requests sent one second apart can come back in either order, because they may take different routes and hit different servers. That is why "the newest answer wins" has to be arranged deliberately, and why the filter bug above is not a rare freak event but a normal consequence of how networks behave.

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

### The four states, and the one everybody forgets

**Loading.** Something changed within 100 milliseconds of the tap. A skeleton, a spinner, a disabled button — anything that says "received, working". This is what prevents the triple-tap.

**Success.** The data arrived and is drawn.

**Empty.** The data arrived and there is none. Zero dishes in this category. Chapter 9's point again: this is a legitimate answer and needs a sentence explaining it, not a blank area that looks like a bug.

**Error.** The request failed, or timed out, or the server said no. This needs a plain-language message and, where it makes sense, a retry button. "Could not load the menu. Check your connection and try again."

Empty and error are the two that get skipped, and they are the two your users on bad connections see most.

### Where it lives

`fetch` sits exactly on the line between M00's client box and server box: it is the browser asking the server for something. Everything about the request itself — the method, the status code, the headers — is chapter 13. What the server does with it is M04. How to stop a user hammering it is M05's rate limiting.

The important thing to carry forward is that **the network is the least reliable part of your product and the only part you do not control.** Your code, your server and your database are all things you can fix. A student's 4G in a basement classroom is not. So every assumption that a request will succeed quickly is an assumption that will be false for some of your users, all of the time.

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

**No timeout means waiting forever.** A `fetch` with no timeout on a connection that has silently died will sit in "loading" until the user gives up and reloads. Ten seconds is a reasonable ceiling for a list; a payment may need longer, but it still needs a number. "It will eventually fail" is not a design.

**Retry is only safe if the action is.** Automatically retrying a "place order" request is how you get three orders. Retrying "load the menu" is free. The dividing line is whether repeating the action repeats its effect, and the server-side fix for actions that are not safe is M04's idempotency keys. Until that exists, do not auto-retry anything that writes.

**The newest request must win, not the newest answer.** Either cancel the previous request when a new one starts, or tag each request and ignore answers from a tag that is no longer current. An AI will almost never do this unless asked, because it never shows up in testing.

**Awaiting in a loop makes requests serial.** Fetching forty dish photos one after another, each waiting for the last, turns forty 200-millisecond requests into eight seconds. Requests that do not depend on each other should be started together.

**A failed `fetch` and a failed *request* are different things.** `fetch` only rejects when the network itself failed. A 404 or a 500 from the server is a *successful* fetch carrying bad news, and code that only handles rejection will cheerfully render a 500 error page's text as if it were your menu. This trips up almost everybody once, and chapter 13 explains the status codes you must check.

**Loading states that cause layout jumps are worse than none.** A spinner that is replaced by content of a different height shifts the page just as the user reaches for a button. Reserve the space — a skeleton the same shape as the content is the standard answer, and chapter 16 measures why.

**Not every wait needs a spinner.** For something under about 200 milliseconds a spinner flashes and reads as a glitch. For a known, small, fast action, a pressed state on the button is better.

### Verify it

All of this chapter's verification happens with the network throttled. On an unthrottled laptop, every one of these bugs is invisible.

1. **Throttle to Slow 4G and add 4x CPU throttling, then use the app normally.** This is the baseline condition for the rest of the programme. Anything you cannot find here, your users will find for you.
2. **Confirm all four states exist.** Ask the AI directly: show me loading, show me empty, show me error. If it cannot produce a screenshot of each, they do not exist. Then force each one yourself: throttle for loading, filter to nothing for empty, and go offline mid-request for error.
3. **Go offline in the middle of a request.** DevTools has an Offline mode. The app must say something a student would understand. A blank screen, a spinner that never stops, or a raw error object are three different failures.
4. **Triple-tap every action that writes.** Add, submit, pay. Then check the data. One intention must produce one record.
5. **Switch filters fast, twice.** Tap one category then immediately another. If the first result ever appears after the second, you have found a race condition. Do it five times on Slow 4G — it is intermittent by nature.
6. **Check the timeout exists.** Use DevTools to throttle to the slowest setting and watch whether the request eventually gives up with a message, or hangs indefinitely.
7. **Count requests on a single page load.** Network panel, filter Fetch/XHR. Forty requests to draw one list usually means a loop that should have been one request — a problem M04 names and M05 caches around.

Red flags in AI output: a screen with only a success state; `fetch` with no timeout; automatic retry on a write action; no cancellation when a new request supersedes an old one; `await` inside a loop over a list; error handling that only catches network rejection and ignores the status code; a `setTimeout` used to "wait until the data is there"; and an error message that shows the raw technical text to the user.

The brief line that covers most of it, under ACCEPTANCE: "I can see and screenshot all four states — loading, empty, error, success — on Slow 4G, and triple-tapping the primary action creates exactly one record."

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
