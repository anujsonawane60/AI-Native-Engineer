---
id: m08-05-installable-pwa
module: m08-mobile
slug: installable-pwa
title: "Turn Your Product into an Installable PWA"
order: 5
duration_min: 35
is_free: false
mission: true
volatility: low
last_reviewed: 2026-11-16
---

## Key learnings
- Name the three ingredients a browser requires before it will offer to install
- Write a manifest that produces a real home-screen icon, not a blurry bookmark
- Choose a caching strategy per request type, and never cache-first your API
- Ship an update that reaches an already-installed user within one visit
- Catch the service worker that serves a stale app forever and cannot be fixed remotely

## Lesson

### The pain

Your product works. The link is in a WhatsApp group, the page loads, the flow is complete. And yet three weeks later usage is ten opens a week, all from the same two people.

The answers are mundane. One student lost the link when the group hit a thousand messages. Another bookmarked it and has never opened the bookmark. A third opened it in the library, had no signal, saw Chrome's dinosaur, and concluded your product was down. Nobody chose against you. Your product simply had no address on their phone — nothing to tap next to WhatsApp.

Then your team ships a fix and discovers the second half: there is no way to tell the two people who do use it. The engineering was fine. The *presence* was missing, and presence is a three-file problem.

### The idea

**Three things turn a website into something a phone treats as an app: HTTPS, a web app manifest, and a service worker. No store, no account, no review.**

HTTPS you already have from any modern host. The **manifest** is a small JSON file telling the browser your app's name, its icons and how it should open — that is what makes "Install app" appear. The **service worker** is a script the browser keeps even when your page is closed, sitting between your app and the network, able to answer from its own cache when the network cannot.

That is the whole contract. The hard part is not getting installed; it is deciding what the service worker may cache, which is where almost all the real bugs live.

### The analogy

The canteen has been serving from a window with no signboard. Regulars find it; nobody else does.

Three changes fix that. It gets a **signboard with its name and logo**, so people can point at it and remember it. It gets a **fixed address**, so a friend can say "third building, ground floor" instead of forwarding directions. And it installs a **shutter with a small counter built into it**, so when the gas fails the window is not dark: it still serves the three cold items kept ready, with a sign saying what is unavailable.

Signboard is the manifest. Fixed address is the start URL. The shutter with a counter is the service worker — limited service instead of a blank screen.

**Where the analogy breaks:** a signboard is repainted when you decide to repaint it. A service worker, once installed on a student's phone, is *already there* and will keep serving yesterday's canteen until you have shipped a correct update strategy. A badly cached PWA is the one thing in this course that can break a user's copy of your product in a way you cannot reach from your side.

### Picture it

```
  your app ─── fetch ───▶ ┌──────────────────┐
                          │  service worker  │
                          │ sees every fetch │
                          └────────┬─────────┘
       network first?              │           cache first?
      ┌─────────────────────────────┴────────────────────────┐
      ▼                                                      ▼
 ┌─────────────────┐                        ┌───────────────────┐
 │ server, else    │                        │ Cache Storage,    │
 │ the cached copy │                        │ refresh afterwards│
 └─────────────────┘                        └───────────────────┘
    for API data                               for the app shell
```

The service worker intercepts every request and chooses where to answer it from; the choice differs per request type, and getting it wrong is the whole of PWA debugging.

### The manifest, field by field

A handful of fields do all the work. These are the ones that go wrong.

| Field | What it decides | Get it wrong and |
|---|---|---|
| `name`, `short_name` | Install dialog and icon label | The label truncates mid-word |
| `icons` | The home-screen icon | A blurry or cropped icon |
| `start_url` | Which screen opens on tap | It opens on a random page |
| `display` | Whether browser chrome shows | It looks like a bookmark |
| `theme_color` | The status-bar colour | A white bar over a dark app |
| `background_color` | The launch screen before paint | A white flash on every open |

Two specifics worth knowing by name. You need at least a **192×192 and a 512×512 icon**, plus a **maskable** variant — Android crops icons into whatever shape the launcher uses, and an icon without safe padding has its edges sliced off. And `display: standalone` removes the address bar; without it the installed app is visibly a web page in a frame, which users read as "not really an app". Make `start_url` the screen a returning user wants, usually today's list rather than your marketing home page.

### Caching strategy, which is the part that matters

A service worker that caches everything the same way is a bug. Choose per request type.

**Cache first, for the app shell.** HTML skeleton, CSS, JavaScript, icons. These change only when you deploy, so serve them from the cache instantly and update in the background. That is what makes an installed PWA open in under a second on a budget phone.

**Network first, falling back to cache, for API data.** The menu, the order list, anything from your server. Try the network; if it fails, serve the last known copy *and label its age*. This is chapter 3's offline-first, implemented.

**Network only, for anything that must never be stale or shared.** Payments, authentication, anything personal. Do not put one student's data into a cache another student's session could read — M05's warning about caching personal data in a shared layer, now on a shared hostel phone.

**An offline fallback page, for navigation.** With no signal and a cached shell, the user should see your page explaining what is available offline, never the browser's dinosaur. That file is the highest-value thing in this chapter per minute spent.

### Updates, and the trap

Here is the failure that bites every team once. You deploy a fix. Already-installed users open the app and still see the old version — because the service worker served the cached shell, exactly as you told it to.

Two facts explain it. A new service worker **installs but waits** by default, taking over only once every tab of your app is closed, which on a phone may be days. And if you let it **cache your HTML cache-first with no version check and no update path**, installed users can be stuck on that build permanently.

The safe pattern is small. Version your cache name, so a new deploy creates a new cache and deletes the old ones. Check for an update when the app regains focus. When one is found, show a small "Update available — reload" button rather than reloading underneath the user's typing.

Test it deliberately: install, deploy a visible change, reopen, confirm you see it within one visit. Without that test you do not know whether you can ship a fix at all.

### Where it lives

A PWA is M00's **client** box upgraded: it keeps a copy of itself on the device and can answer its own requests. Your server, network and data boxes are untouched — it is additive, which is why every Sprint student can ship it in one session.

It also stitches the module together. Chapter 1 chose this route. Chapter 2's push, on Android, arrives through this service worker. Chapter 3's offline rules are what the caching strategy implements. Chapter 4's budget is what the cached shell helps you meet, since a repeat open downloads almost nothing. M02's service worker and caching chapters are the mechanism, M03 supplies the offline and error screens, M06 deploys it over HTTPS, and M07 is where you check nothing personal landed in a cache.

**Store distribution stays in M12** — a Play Store listing for this same PWA via a Trusted Web Activity, plus the native route. You are not creating a developer account this week.

### Words to own

| Term | Plain meaning | Example |
|---|---|---|
| Manifest | JSON naming your app, icons and start URL | `manifest.webmanifest` |
| Service worker | A script that answers requests when offline | Your shutter counter |
| App shell | The HTML, CSS and JS skeleton | Cached, opens instantly |
| Cache Storage | Where the service worker keeps responses | Versioned per deploy |
| Cache first | Serve stored copy, refresh later | Right for the shell |
| Network first | Try the server, fall back to the cache | Right for API data |
| Offline fallback | Your own page shown with no signal | Never the dinosaur |
| Maskable icon | An icon with padding for launcher cropping | 512×512, safe area |
| `display: standalone` | Opens with no address bar | Looks like an app |

### Tradeoffs and gotchas

**Do not ask to install on the first paint.** It is chapter 2's permission mistake in a new costume. Let the user complete something useful once, then offer installation with a reason: "add to home screen so you can check the menu without opening the browser".

**A service worker is the one thing you can break remotely and cannot always fix remotely.** Keep it small, version the caches, and test the update path before you need it. An unregister-and-clear path written in advance is what saves you.

**Never cache-first anything personal or financial.** On shared hostel phones this is not theoretical. One student's cached order list must not be served to the next person who opens the app.

**Offline does not mean "the whole app works".** Pick chapter 3's three features, make those work, and have your fallback page say plainly what is unavailable. An honest limit beats a half-broken pretence.

**Installed does not mean retained.** The icon is an invitation, not a habit. If the product is not worth opening twice a week, installation changes nothing.

**iPhone is a footnote with a plan attached.** On iOS a PWA installs only through Share → Add to Home Screen, with no prompt, and most users do not know the gesture exists. Web push needs that install first and the rules have shifted across versions. Ship the PWA, add one "how to install on iPhone" line, and do not build a core feature on iOS web push.

### Verify it

AI will produce a manifest and a service worker in one pass, and the result often looks perfect and is wrong in the ways below.

**Check the caching strategy per request type, not in general.** Ask for a table: every kind of request, the strategy, the reason. If every row says the same thing it was not a decision, and if your API rows say cache-first you have shipped stale prices.

**Red flag: the HTML cached cache-first with no versioning.** The one that can strand users on a broken build. Look for a cache name containing a version and old caches deleted on activation. If neither is there, fix it before deploying.

**Red flag: no offline fallback page.** Ask: "show me the exact screen a user sees opening the installed app in aeroplane mode". If the answer is the browser's error page, the most valuable file is missing.

**Red flag: anything personal in the cache.** Read the cached paths out loud. If an authenticated response or a user-specific page is there, remove it and re-ask with that as a constraint.

**Red flag: an install prompt fired on page load.** Find where it is triggered. If it is in the startup path, move it behind a button the user reaches after the product has done something for them.

**Red flag: icons from one source file at one size.** Confirm 192, 512 and a maskable variant. A sliced-off icon is the easiest thing to miss, because it looks fine in the manifest.

Then run the three tests no summary can fake. Install on a real Android phone and confirm there is no address bar. In aeroplane mode, confirm you see your page rather than the dinosaur. Deploy a visible change, reopen, and confirm it arrives within one visit.

## Walkthrough
1. Confirm your deployed product is served over HTTPS with no mixed-content warnings, then write down your three offline features from chapter 3. Without that list you will cache by instinct.
2. Write the caching strategy as a table before any code: one row per request type — shell, public API reads, authenticated API, images, writes — with the strategy and the reason. Mark everything personal or financial as network only.
3. Write the manifest — name, short name, `start_url` pointing at the screen a returning user wants, `display: standalone`, theme and background colours matching your design — then produce icons at 192×192 and 512×512 plus a maskable variant with safe padding, and reference the manifest from your HTML.
4. Register a service worker and precache only the app shell: HTML skeleton, CSS, JS, icons, and fonts if you have any. Name the cache with a version string.
5. Add your own offline fallback page, designed like the rest of your product, listing exactly what works without a signal and what does not.
6. Implement your table in the service worker's fetch handler, and delete old versioned caches on activation so a deploy cannot leave two generations behind.
7. Run a Lighthouse or dev-tools audit for installability and fix whatever it names, in its own words, until it passes.
8. Install it on a real Android phone. Confirm the icon is not cropped, the label is not truncated, there is no address bar, and there is no white flash on launch.
9. Switch that phone to aeroplane mode, open the installed app, and confirm your own offline page appears and your three offline features work. Then turn the network back on and confirm it recovers without a manual reload.
10. Deploy a visible change — a word in the header is enough — reopen the installed app, and confirm you see the change within one visit. Record what you had to do to make that work.
11. Add the install invitation last: a small prompt shown only after the user has completed something useful once, with a one-line reason.

## Talk to AI
```prompt
CONTEXT:      I am a student directing AI rather than writing code. I want
              to make my team's existing deployed web product installable
              and usable with no signal. Stack: [your stack]. Hosting:
              [your host], free tier, already HTTPS.

              Users: budget Android phones, 2-3 GB RAM, patchy 4G,
              metered data packs, roughly 90% Android and 10% iPhone.
              Some phones are shared between students.

              My screens and routes: [list them with their URLs]
              My API endpoints, marked public or authenticated:
              [list them]
              The three things that MUST work with no signal:
              [list exactly three]
              Things that must never be stale or cached:
              [prices, payments, anything personal - list them]
              My design tokens for theme and background colour:
              [values]

GOAL:         An installable PWA that opens from the home screen with no
              address bar, shows my own offline page with no signal, and
              can still receive an update I deploy tomorrow.

REQUIREMENTS: 1. A web app manifest with every field explained in one
                 line each, including which screen start_url should open
                 and why, and the exact icon sizes I must produce
              2. A caching strategy TABLE before any code: one row per
                 request type from my lists, the strategy, and the reason.
                 Everything authenticated or financial must be network
                 only
              3. A service worker plan that precaches only the app shell,
                 names the cache with a version, and deletes old versions
                 on activation
              4. The content of my offline fallback page, written as the
                 actual words a user reads, naming what works and what
                 does not
              5. An update path: how an already-installed user receives
                 tomorrow's deploy within one visit, what they see while
                 it happens, and why a reload must not be forced while
                 they are typing
              6. The exact steps I will take to verify this on a real
                 Android phone, as checks with a pass or fail each
              7. One honest paragraph on what iPhone users get, what they
                 must do manually, and what I should not rely on

CONSTRAINTS:  Free tier only. No store submission, no developer account,
              no native shell - those belong to a later module. Must work
              on a 2-3 GB RAM Android on slow 4G. Phones may be shared,
              so no personal data in any cache. Total first load must stay
              under 1 MB. Do not add a heavy framework or build tool.

NON-GOALS:    Do not redesign my UI. Do not add push notifications yet.
              Do not implement background sync. Do not cache my
              authenticated endpoints under any circumstances. Do not
              touch my auth flow.

ACCEPTANCE:   - Every request type I listed appears in the strategy table
                with a reason, and not all rows use the same strategy
              - No authenticated or payment route is cached anywhere
              - The cache name contains a version and old caches are
                deleted
              - The offline page is written as real user-facing words
              - The update path explains what happens to an installed
                user who is mid-typing
              - Your verification steps can each be marked pass or fail
                by someone who is not me

ASK:          Give me the caching strategy table and the update path
              first, and stop there. Explain the two riskiest decisions
              in it. No code until I approve the table.
```

## Mission

**Objective.** Your team's own product is installable on a real Android phone, opens from the home screen with no address bar, shows your own offline page with no signal, and can receive a change you deploy afterwards within one visit.

**Steps.**
1. **Frame** — write one sentence on who installs this and why they would want an icon rather than a link, and name the three things that must work with no signal.
2. **Brief** — write a full Director's Brief for the PWA work. Include your route list, your authenticated endpoints, the three offline features, and the explicit non-goal that nothing personal or financial is cached. Write ACCEPTANCE first.
3. **Decide the table** — before any code, get the caching strategy table: one row per request type, the strategy, the reason. Reject it if every row is the same strategy, or if an authenticated route is cached.
4. **Direct** — have the manifest, icons, service worker and offline page built in that order, one step at a time, committing between steps so each change is reviewable as its own diff.
5. **Verify on a device** — install it on the cheapest Android phone your team can find. Check the icon is not cropped, the label is not truncated, there is no address bar, and there is no white flash on launch.
6. **Verify offline** — aeroplane mode, open the installed app, confirm your own offline page appears and that your three offline features work. Then confirm the app recovers when the network returns without a manual reload.
7. **Verify the update path** — deploy a visible change, reopen the installed app, and confirm you see it within one visit. If you do not, your caching is a trap and fixing it is the mission.
8. **Check for leaks** — list every cached path and confirm no authenticated response and no payment route is among them. On a shared phone, log out and confirm the previous user's data is not served.
9. **Measure** — record first-load bytes and time-to-readable-text for a fresh install and for a repeat open, on that same cheap phone.
10. **Document** — one decision-log entry: your caching table in one line per row, the three offline features, what you deliberately left online-only, and your update strategy.

**Upload.**
- Your Director's Brief, in full → post caption or an attached file
- Your caching strategy table → screenshot or text, one row per request type
- Proof it installs → deployed URL in `demo_url`, plus a home-screen screenshot showing your icon
- Proof it works offline → screenshot of your own offline page in aeroplane mode on a real phone
- Proof the update path works → before and after screenshots of the installed app after a deploy
- The AI conversation showing the strategy table and your corrections → share link in `ai_chat_url`
- Your measurements → first-load bytes and seconds, fresh install versus repeat open
- Reflection, 150 words max → one tradeoff you chose, and one thing the AI got wrong

**Self-check.** Tick every box before you submit.
- [ ] My brief named the three offline features and the things that must never be cached
- [ ] My caching table has more than one strategy in it, with a reason per row
- [ ] No authenticated or payment route appears in any cache
- [ ] My cache name has a version, and old caches are deleted on activation
- [ ] It installed on a real Android phone, not an emulator
- [ ] The icon is not cropped and the app has no address bar
- [ ] Aeroplane mode shows my own offline page, never the browser's dinosaur
- [ ] I deployed a visible change afterwards and saw it within one visit
- [ ] The install invitation appears after the user has done something useful, not on first paint
- [ ] I recorded bytes and seconds for both a fresh install and a repeat open
- [ ] My reflection names something specific the AI got wrong

**Stretch.** Add a one-line "how to install on iPhone" hint shown only to iOS visitors, then have a classmate with an iPhone follow it without help and record where they got stuck.

## Stuck? Ask AI
```prompt
My installed PWA will not update. Users who installed it last week still see the old version after I deploy. Explain to me, without code, exactly why this happens: what the service worker lifecycle does by default, what "waiting" means, and why closing one tab is not enough on a phone. Then give me the three safest changes in order, and tell me which one I should never make.
```

```prompt
Here is my service worker fetch handler: [paste it]. Do not rewrite it. Instead, list every kind of request my app makes and tell me which strategy this code applies to each one. Then tell me which of those choices is wrong and what the user will experience because of it. Pay special attention to anything authenticated, anything with a price in it, and what happens when the network is present but unusable.
```

```prompt
I installed my PWA on an Android phone and something looks wrong: [describe it - cropped icon, white flash on launch, address bar still visible, wrong screen on open, truncated label]. For each symptom, tell me which manifest field or icon requirement causes it, how to confirm that is the cause in browser dev tools, and what the corrected value looks like. Ask me for the current value before you guess.
```

## Quiz

### Q1 · L2
What three things must be true before a browser will offer to install your web app?
- (a) A Play Store listing, an APK and a signing key
- (b) HTTPS, a web app manifest and a registered service worker ✅
- (c) A custom domain, push notifications and an offline page
- (d) A native shell, a maskable icon and a developer account
> That is the entire installability contract, and no store or paid account is part of it. Everything hard about PWAs comes after installation, in deciding what the service worker caches.

### Q2 · L3
Which caching strategy belongs on your `/api/menu` endpoint, where prices change during the day?
- (a) Cache first, so the menu opens instantly
- (b) Network first, falling back to the cached copy with its age shown ✅
- (c) Cache only, since the menu rarely changes
- (d) The same strategy as the app shell, for consistency
> Cache-first on API data ships stale prices. Try the network, fall back to the last known copy and label it — which is chapter 3's offline-first rule implemented in the service worker.

### Q3 · L4
You deploy a bug fix. Already-installed users still see the old build. What is the most likely cause?
- (a) Your host did not purge its CDN cache
- (b) The service worker served the cached shell and the new worker is waiting ✅
- (c) Android caches installed PWAs for seven days
- (d) The manifest `start_url` is pointing at an old route
> A new service worker installs but waits until every tab of the app is closed. Without a versioned cache and an update check on focus, installed users can sit on an old build for days.

### Q4 · L4
On a hostel phone shared between three students, which of these must never be in Cache Storage?
- (a) Your CSS and JavaScript bundle
- (b) Your 192 and 512 pixel icons
- (c) An authenticated response containing one student's order history ✅
- (d) Your offline fallback page
> The cache now lives on a device more than one person uses. Personal and financial responses are network only, which is M05's rule about caching personal data in a shared layer with a sharper edge.

### Q5 · L5
Your team adds a service worker that caches the HTML cache-first, with no version in the cache name and no update check. Why is this worse than having no service worker at all?
- (a) It increases first-load bytes beyond the performance budget
- (b) It can strand installed users on a broken build with nothing you can do from your server ✅
- (c) It prevents the browser from offering the install prompt
- (d) It disables the offline fallback page automatically
> This is the one failure in the course you can cause remotely and cannot always fix remotely. Versioned caches, deletion of old ones and an update check on focus are what keep a deploy reachable.

## Go deeper
- [web.dev: Add a web app manifest](https://web.dev/articles/add-manifest)
- [MDN: Using Service Workers](https://developer.mozilla.org/en-US/docs/Web/API/Service_Worker_API/Using_Service_Workers)
