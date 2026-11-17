---
id: m02-16-rendering-vitals
module: m02-the-web
slug: rendering-bundlers-web-vitals
title: "Rendering Strategies, Bundlers, CDNs and Core Web Vitals"
order: 16
duration_min: 25
is_free: false
mission: true
volatility: medium
last_reviewed: 2026-11-16
---

## Key learnings
- Choose where a page is built — build time, request time or in the browser
- Measure speed as three named numbers instead of describing it as a feeling
- Find the single largest thing your page downloads and decide whether it earns its place
- Set a performance budget your team can fail against
- Name the five performance red flags that only appear on a budget phone

## Lesson

### The pain

Checkpoint demo. The app opens in 1.4 seconds on the presenter's laptop and 11 seconds on the mentor's three-year-old Android on venue 4G. The Network panel explains it: 3.2 MB transferred, 2.4 MB of it JavaScript, to show six dish names and six prices. A charting library rides along because one admin screen has a graph and the bundle is not split, and the photos are 900 KB camera images displayed at 160 pixels wide.

The worst part is not the wait. At second nine the logo loads, the page reflows, and the mentor — reaching for "Masala Dosa" — taps "Sold out: Idli".

### The idea

**Where and when your page is built decides what the first second feels like, and "fast" is three specific measurable numbers rather than an impression.**

Three places a page can be built. At **build time**, once in advance — a plain file, so serving it is as fast as the internet gets. At **request time** on the server — always fresh, and the user waits. In the **browser**, after an almost-empty document arrives — your server's job is tiny and the work lands on a ₹8,000 Android.

### The analogy

**Pre-cooked in the warmer** is build-time rendering: served in four seconds, but made at 10am, so a noon price change leaves the board wrong. **Cooked when you order** is server rendering: exactly current, and you stand there while it is made. **Handed raw ingredients to assemble at your table** is client rendering: the counter frees up instantly, which looks efficient from behind the counter, until you notice you are doing the work with whatever tools you have.

**Where the analogy breaks:** the canteen serves one person at a time. A CDN copies your pre-cooked page to servers worldwide and serves thousands at once from the nearest — which is why build time plus a CDN is fast *and* cheap, and why a CDN cannot help the cooked-to-order case, where the wait is the kitchen rather than the distance.

### Picture it

```
  BUILD TIME (static)
  build ──▶ file on a CDN ──▶ user gets bytes immediately
                              fresh? only as of the last build

  REQUEST TIME (server rendered)
  tap ──▶ server builds the page ──▶ bytes arrive, already useful
          always fresh, user waits for the server

  IN THE BROWSER (client rendered)
  tap ──▶ empty shell arrives ──▶ 2.4 MB of script downloads
          ──▶ script runs on the phone ──▶ fetches data ──▶ draws
          cheap for your server, slow on a budget Android
```

Build time serves finished bytes from a nearby CDN, request time builds fresh bytes while the user waits, and client rendering sends an empty shell plus the code and does the work on the user's own phone.

Real apps mix all three, per page: **static unless there is a reason, server-rendered when the content is per-user or changes constantly, client-rendered only for the genuinely interactive parts.** Landing page static; menu server-rendered or static with a short revalidation window; cart client work. Asking "which is this page?" in the brief stops everything defaulting to the slowest option.

### Where it lives

This chapter touches all four of M00's boxes: what the client does, what the server does, how much crosses the network, where the data comes from. Chapter 15's rendering choice becomes concrete here, M06 enforces a performance budget in CI, and M07's analytics collects these numbers from real users rather than your machine.

The three measurements map onto what a user feels: LCP is "when can I read it", INP is "did my tap do anything" (chapter 8's silence, measured), CLS is the mentor tapping "Sold out: Idli". Targets are in the table below, and the only number worth quoting is measured **on a mid-range phone on a throttled network**.

### Words to own

| Term | Plain meaning | Example |
|---|---|---|
| Static generation | Built once in advance, served as a file | The landing page |
| Server rendering | Built on the server per request | A per-user dashboard |
| Client rendering | Built in the browser by JavaScript | The cart |
| Revalidation | Rebuilding a static page every N minutes | A menu refreshed hourly |
| Hydration | Script attaching behaviour to server-sent markup | The cost of interactivity |
| Bundle | All your JavaScript packed into files to ship | The 2.4 MB |
| Bundler | The tool that packs and optimises it | Part of the framework |
| Code splitting | Shipping only the code this page needs | Chart code off the menu |
| Tree shaking | Dropping code nothing actually uses | Unused library parts |
| CDN | Copies of your files on servers worldwide | Serves the nearest one |
| LCP | Time until the biggest useful thing is visible | Under 2.5s |
| INP | Time from a tap to a visible response | Under 200ms |
| CLS | How much the layout jumps while loading | Under 0.1 |
| Performance budget | A limit your team agrees not to exceed | 300 KB of JavaScript |

### Tradeoffs and gotchas

**"Client-render everything" is the AI default,** because it always works and needs the least thought. It also moves the cost onto the slowest device in the chain.

**A library for one number is the most common waste** — a charting library for "42 orders today", a date library to print a time, an icon package for four icons. Ask of every dependency: what does this cost in kilobytes, and what would we do instead?

**Images are the bigger problem and the easier win.** A 4000px photo displayed at 160px wastes 99% of its bytes. Resizing and correct `width`/`height` are the highest-return performance work in a student project, and the sizing fixes CLS too — unsized images and late fonts are what CLS actually is.

**Static means stale, and stale sometimes costs money.** A menu built at 10am will sell a dish that sold out at 11, so revalidate often, fetch availability on the client, or server-render that page — and write down what staleness you accepted. The reverse trap: **a CDN cannot fix a slow database**, because a page rebuilt per request from a slow query has nothing cacheable.

**Your own repeat visit is the fastest anyone will ever experience your site:** cached, resolved, fonts local. Measure in a private window with the cache disabled, in Lighthouse's mobile preset, or you are measuring something no first-time user sees.

### Verify it

1. **Read the total transferred with cache disabled,** then the JavaScript subtotal separately. For six dishes and six prices, anything over a few hundred kilobytes needs justifying.
2. **Sort by size and name the three largest files.** For each: what is it for, and does every user on every page need it?
3. **Run Lighthouse in the mobile preset** and record LCP, INP and CLS. Three runs, take the middle one.
4. **Check CLS by watching, not by score.** Reload on Slow 4G: anything that moves after you could have reached for it is the bug.
5. **Compare one image's intrinsic size against its displayed size.** A 4000px image in a 160px slot is a finding you can fix today.
6. **Check `width` and `height` are on every image.** The most common CLS cause, and the check is a search.
7. **Measure on a real budget Android on mobile data.** Stopwatch, tap to readable content. This is the number you quote, and it will be worse than every throttled one.
8. **Ask which rendering strategy each page uses, and why.** The same answer for every page means nobody chose.

Red flags: every page client-rendered · a library added for one small job · images with no `width`/`height` · full-resolution images · a performance claim measured on a laptop · `100vh` sections and late fonts causing shift · no code splitting, so one admin screen's dependencies ride along everywhere · a Lighthouse score quoted with no throttling stated, which is comparable to nothing.

## Walkthrough
1. Open your team's product page, or the closest existing product your users use, in a private window with DevTools on the Network panel and "Disable cache" ticked.
2. Reload with no throttling and record two numbers: total transferred, and the JavaScript subtotal shown in the summary bar. Write both down.
3. Sort the requests by size and list the five largest with their types. For each, write one sentence on what it is for.
4. For the single largest JavaScript file, ask an AI what it most likely contains given the framework, and whether it could be split so that only the pages needing it download it.
5. Inspect the largest image on the page. DevTools shows its intrinsic size and its rendered size when you hover the `src`. Record both and calculate roughly what fraction of the bytes were wasted.
6. Search the rendered markup for `<img` and check how many have both `width` and `height` attributes. Record the count of those that do not.
7. Set throttling to Slow 4G with 4x CPU and reload while watching the top of the page closely. Screenshot the mid-load state, and note every element that moves position after first appearing.
8. Run Lighthouse in the mobile preset, three times. Record LCP, INP and CLS from the middle run, and note the throttling settings the report states.
9. Compare that LCP against what you would get with no throttling, by running it once more in the desktop preset. Write down both numbers side by side — the gap is the thing most teams never see.
10. On the cheapest Android phone in your group, on mobile data, open the same page and time with a stopwatch from tap to readable content. Record the number.
11. Write a one-page summary: the four measured numbers, the three biggest things downloaded, the worst layout shift, and the single change you believe would buy the most improvement.

## Talk to AI
```prompt
CONTEXT:      I direct AI rather than writing code. My product is a canteen
              ordering app for students in Nashik. 85% of traffic is budget
              Android phones on congested 4G; some users are on 2G-like
              speeds at lunchtime. Pages: a landing page that rarely
              changes, a menu that changes when staff mark dishes sold out,
              a dish detail page, a per-user cart, a checkout, a per-user
              order history, and a staff dashboard with one chart. Dish
              photos are uploaded by canteen staff from their phones, so
              they arrive as full-resolution camera images. Free tiers only.
              I have measured my current page: [paste your numbers from the
              walkthrough, or write "not measured yet"].

GOAL:         A rendering strategy per page and a performance budget my
              team can fail against, aimed at a readable menu in under 2.5
              seconds on a budget Android on slow 4G.

REQUIREMENTS: 1. A table with one row per page: rendering strategy (build
                 time, request time, or browser), the one-sentence reason,
                 and what staleness that choice accepts
              2. For the menu specifically, address the conflict between
                 serving it fast and never showing a dish that just sold
                 out — propose two options with tradeoffs
              3. A performance budget as numbers: total transferred,
                 JavaScript transferred, largest image, LCP, INP and CLS —
                 each with the measurement conditions stated
              4. A plan for the staff-only chart so that students never
                 download any of its code
              5. A pipeline for staff-uploaded photos: what sizes, what
                 format, and how width and height get onto every image tag
              6. The three changes you expect to buy the most improvement,
                 ranked, with an estimate of the saving for each
              7. How I verify each budget line in DevTools or Lighthouse,
                 including the exact throttling settings to use

CONSTRAINTS:  Free tiers only. No paid image service unless there is a free
              tier, and say which. Do not client-render any page that does
              not need interaction. Every image must carry width and
              height. No library added for a single small job — if you
              propose one, state its transferred size. The budget must be
              expressed as numbers I could put in a CI check later.

NON-GOALS:    Do not redesign the visual layout. Do not change the data
              model or the API. Do not add a caching layer or a service
              worker yet. Do not discuss framework choice — assume it is
              already made.

ACCEPTANCE:   - Every page has a stated strategy and a stated staleness
                consequence
              - The budget is numbers with measurement conditions, not
                adjectives
              - Students downloading the menu receive none of the chart code
              - Every image in the plan has width and height, and a
                maximum dimension
              - LCP under 2.5s, INP under 200ms and CLS under 0.1, measured
                in the mobile preset with Slow 4G and 4x CPU throttling
              - Each of your three ranked changes has an estimated saving
                in kilobytes or seconds

ASK:          First ask me up to five questions whose answers would change
              the plan — including anything you need to know about how
              often the menu changes. Then give the table and the budget.
              Then, separately, tell me which of my requirements you think
              is the wrong priority for a team with 45 days, and why. Do
              not act on that opinion without my approval.
```

## Mission

**Objective.** Produce a measured performance report and an enforced performance budget for your own startup's most important page, with before-and-after numbers taken under the same stated conditions.

**Steps.**
1. **Frame** — name the one page that matters most to your product's success, say who opens it and on what, and state in one sentence what "fast enough" means for that person.
2. **Baseline** — run the eleven-step walkthrough on that page. Record total transferred, JavaScript transferred, the five largest files, LCP, INP and CLS, and your stopwatch number from a real budget Android on mobile data. State the throttling settings for every number.
3. **Diagnose** — for each of the five largest files, write what it is for and whether every user needs it on this page. Name the single biggest waste.
4. **Shift** — throttle to Slow 4G, reload while watching, and screenshot the mid-load state. List every element that moves after first appearing, and identify the cause of each.
5. **Decide** — write the rendering strategy for this page: build time, request time or browser, with the reason and the staleness you are accepting. If it changes from what exists now, say what you are trading.
6. **Budget** — write a performance budget as numbers with measurement conditions: total transferred, JavaScript transferred, largest image dimension, LCP, INP, CLS. These must be numbers your team could fail against.
7. **Direct** — write a full Director's Brief for the three highest-value changes, with your real baseline numbers pasted into CONTEXT, an explicit NON-GOALS list, and acceptance criteria stated as the budget numbers.
8. **Verify** — have the changes made, then re-run every measurement under identical conditions and record pass or fail per budget line, with fresh screenshots.
9. **Re-test on the phone** — repeat the stopwatch test on the same cheap Android on mobile data and compare against your baseline.
10. **Document** — one decision-log entry: the rendering strategy you chose, what you rejected, and the staleness or complexity you accepted in exchange.

**Upload.**
- Your baseline table: all measured numbers with their measurement conditions → post caption or attached PDF
- The mid-load screenshot with every shifting element marked → screenshot
- Your performance budget as numbers → post caption or attachment
- Your full Director's Brief for the three changes → post caption or attachment
- The AI conversation → share link in `ai_chat_url`
- Proof it works → deployed URL in `demo_url`
- Your after table, under identical conditions, with pass/fail marked per budget line → screenshot or text
- Your stopwatch numbers from a real budget Android, before and after
- Your one decision-log entry
- Reflection, 150 words max → the gap between your laptop number and your phone number, and one thing the AI got wrong

**Self-check.** Tick every box before you submit.
- [ ] Every number I report states its throttling settings and device
- [ ] My before and after measurements were taken under identical conditions
- [ ] I measured in a private window with the cache disabled
- [ ] I took three Lighthouse runs and used the middle one
- [ ] I have a stopwatch number from a real budget Android on mobile data, not Wi-Fi
- [ ] I named the five largest downloads and said what each is for
- [ ] My budget is numbers, not adjectives, and my team could fail against it
- [ ] I stated the rendering strategy for this page and the staleness I accepted
- [ ] Every image on the page has `width` and `height`
- [ ] I identified the cause of every element that moved during load
- [ ] My reflection names the laptop-versus-phone gap with both numbers

**Stretch.** Pick one competitor or one product your users already use, measure it under the same conditions, and write 100 words comparing it with yours. If theirs is faster, name the one thing they do that you do not.

## Stuck? Ask AI
```prompt
Explain the difference between static generation, server rendering and client rendering using an analogy that is not about cooking or restaurants. Then give me eight pages from a real product and ask me which strategy each should use and what staleness that accepts. Do not tell me the answers until I have attempted all eight.
```

```prompt
Here are my measured numbers for a page: [paste total transferred, JavaScript transferred, the five largest files, LCP, INP, CLS, and the throttling settings]. Do not propose a rewrite. Instead, rank the three changes that would buy the most improvement, with an estimated saving in kilobytes or seconds for each, and tell me exactly how I would verify each one in DevTools. Then tell me which of my numbers is the least trustworthy and why.
```

```prompt
My page has a Cumulative Layout Shift of 0.34 and I can see things jumping as it loads. Explain what each of the usual causes looks like on screen — unsized images, late fonts, injected banners, content loading above existing content — and give me a diagnostic order for finding which ones apply to me. Tell me what to look at in DevTools for each.
```

## Quiz

### Q1 · L3
Which rendering strategy moves the most work onto the user's device?
- (a) Static generation at build time
- (b) Server rendering per request
- (c) Client rendering in the browser ✅
- (d) All three are equivalent for the user
> Client rendering sends an almost-empty document plus the code, then the phone downloads, parses, runs and fetches. It is cheapest for your server and most expensive for a ₹8,000 Android on 4G.

### Q2 · L3
Your page transfers 2.4 MB of JavaScript to show six dishes and six prices, and a charting library is in the bundle for one admin screen. What is the specific fix?
- (a) Minify the JavaScript further
- (b) Code splitting, so only the admin screen downloads the chart code ✅
- (c) Move the chart to a server-rendered page
- (d) Compress the dish images
> The waste is that every student downloads code they will never run. Splitting the bundle per route means the cost lands only on the page that needs it.

### Q3 · L4
A mentor reaches for "Masala Dosa", the logo loads, the page reflows, and she taps "Sold out: Idli". Which metric does this describe and what usually causes it?
- (a) LCP, caused by a slow server
- (b) INP, caused by heavy JavaScript
- (c) CLS, caused by images and fonts loading without reserved space ✅
- (d) LCP, caused by the image being too large
> Cumulative Layout Shift measures exactly this. `width` and `height` on every image, plus reserved space for anything that arrives late, is most of the fix — and it is the same skeleton idea from chapter 10.

### Q4 · L4
An AI reports a Lighthouse performance score of 96. What must you ask before the number means anything?
- (a) Which browser version was used
- (b) Which preset and throttling settings — a laptop run with no throttling is not comparable to anything ✅
- (c) Whether the page was minified
- (d) How many requests the page made
> A number without its conditions is not a measurement. Mobile preset, Slow 4G, 4x CPU, cache disabled, three runs, middle result — and then a real cheap phone, which will still be worse.

### Q5 · L5
Your menu is statically generated at build time and a dish sells out at 11am. What have you traded, and what are the two honest ways out?
- (a) Nothing — static pages update automatically
- (b) You traded freshness; revalidate frequently, or fetch availability in the browser on top of the static page ✅
- (c) You traded speed; switch the whole app to client rendering
- (d) You traded reliability; add a CDN in front of the page
> Static means stale, and here stale means selling food that is gone. Short revalidation and a client-side availability check are the two standard answers, and the choice should be written down as an accepted tradeoff.

## Go deeper
- [web.dev: Core Web Vitals](https://web.dev/articles/vitals)
- [web.dev: Performance budgets 101](https://web.dev/articles/performance-budgets-101)
