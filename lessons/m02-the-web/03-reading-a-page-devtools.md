---
id: m02-03-devtools
module: m02-the-web
slug: reading-a-page-devtools
title: "Reading a Page You Did Not Write: View Source and DevTools"
order: 3
duration_min: 25
is_free: false
mission: true
volatility: low
last_reviewed: 2026-11-16
---

## Key learnings
- Verify any claim an AI makes about a page in under a minute, using the browser alone
- Say why view-source and the Elements panel often disagree, and which one to trust
- Read a failed request's method, status and payload instead of guessing
- Turn "it does not work" into a bug report with evidence attached
- Simulate a budget Android on slow 4G before your users do it for you

## Lesson

### The pain

A student asks an agent to fix a button that does nothing. "Fixed — the click handler was not attached." They reload, click, nothing happens, type "still broken", and the agent rewrites a different file. Forty minutes later two things are broken and nobody knows what changed.

Everything they needed was on screen all along: one red Console line naming the problem, a Network panel showing no request was ever sent, an Elements panel showing the button covered by an invisible box. Three facts, sixty seconds.

### The idea

**Every browser ships with a complete inspection kit, and it turns M00's VERIFY step from an opinion into a measurement.**

You are not going to debug code. You are going to answer small factual questions: is the element there, is it visible, did the request fire, what came back, how long did it take. Those answers make your next instruction precise instead of "still broken".

### The analogy

The canteen kitchen has a serving window. You are not going to cook, but you can see the counter, hear the shouting, watch the delivery van at the back door, and look into the storeroom. That is the layout of DevTools: Elements is the counter, Console is the shouting, Network is the back door with every delivery's time and weight, Application is the storeroom of what was kept from earlier visits.

**Where the analogy breaks:** through a real window you can only look. In DevTools you can reach in — change a colour, delete an element, shrink the screen to a phone — and none of it is permanent. Reload and every experiment vanishes. That is what makes it a thinking tool, not just a report.

### Picture it

```
  DevTools panel      question it answers
  ┌──────────────┐
  │ Elements     │  Is it there? Is it visible? Which CSS won?
  ├──────────────┤
  │ Console      │  What did the page complain about, and where?
  ├──────────────┤
  │ Network      │  Did a request fire? Status, payload, size, time
  ├──────────────┤
  │ Application  │  What is stored: cookies, storage, cached files
  ├──────────────┤
  │ Lighthouse   │  How slow is this on a cheap phone on 4G?
  └──────────────┘
```

Each panel answers a different kind of factual question, and knowing which panel to open is most of the skill.

### Where it lives

DevTools lives in the client box from M00 — the browser's side of everything, none of your server's internals. A 500 says the server failed; *why* is in server logs (M07). This chapter is the Director's Loop step 5, VERIFY, made concrete: "I checked" now means "I opened the Network panel".

The distinction that confuses everybody — neither view is lying:

| View | Shows | Trust it for |
|---|---|---|
| View source (`Ctrl+U`) | The HTML text the server sent, before any JavaScript ran | "Why does Google not see my menu?", "why is the first paint empty?" |
| Elements panel | The live DOM right now, after JavaScript added and rearranged things | "Why is this button not clickable?" |

On a framework app (chapter 15) view source may be an empty `<div id="root"></div>` while Elements shows a thousand nodes.

### Words to own

| Term | Plain meaning | Example |
|---|---|---|
| View source | The HTML text the server sent, before JavaScript | `Ctrl+U` |
| Rendered DOM | The live tree as it is now | The Elements panel |
| Console error | A red line naming what the page choked on | "undefined is not a function" |
| Request | One trip to a server for one thing | Fetching the menu |
| Status code | The server's verdict on that request | 200, 404, 500 |
| Payload | The data a request carried or returned | The order JSON |
| Waterfall | The chart of what loaded when | Network panel timeline |
| Computed styles | The CSS values that actually won | Elements → Computed |
| Device toolbar | Pretends the browser is a phone | 360 x 800 |
| Throttling | Pretends the network and CPU are slow | "Slow 4G" |
| Hard reload | Reload ignoring all cached files | Rules out stale caching |

### Tradeoffs and gotchas

**DevTools edits are temporary, and that is a feature.** Changing a colour in Elements changes no file — the cheapest way to test an idea before briefing the AI with the value. Never report a DevTools edit as a fix.

**Half the red lines in the Console are not yours.** Extensions, ad blockers and analytics shout into the same place. Check the file name on the right before chasing an error.

**Throttling is an approximation.** "Slow 4G" is a reasonable fake; a real ₹8,000 Android on Nashik 4G at 1pm is worse. Throttling says whether you have a problem, a real device says how bad.

**Tick "Preserve log" before you submit anything,** or a form that redirects wipes the evidence of its own request.

**"The element is there but nothing happens"** is almost always one of four things: covered by another element, `pointer-events: none`, the handler on a different element, or the handler threw on first click. Console plus Elements separates them fast.

### Verify it

1. **Make the claim testable.** If no panel could prove "fixed", it was never a claim about the page.
2. **Did the request fire at all?** Network, filtered to Fetch/XHR. No request means nothing about your server is relevant yet.
3. **What is the status, and what is the body?** A 200 carrying `{"error": "..."}` is a failure the page may treat as success (chapter 13).
4. **Is the element in the DOM, and is it visible?** Present-but-invisible, present-but-covered and absent are three different bugs.
5. **Which CSS rule won?** Elements → Computed; the rule you expected may be struck through (chapter 4).
6. **Does it survive a hard reload and a private window?** That separates real bugs from stale caches and extensions.

Red flags in AI behaviour: "fixed" with no way to check it · a server-side fix when no request was sent · explaining a console error it was never shown, which means guessing · telling you to clear your cache, which hides the problem from you alone.

The habit that matters most: **paste the actual error text, status code and payload into your next message.** The gap between "it does not work" and "Submit fires no network request; Console shows `TypeError: cannot read properties of null` at `order.js:42`" is bigger than the gap between any two AI models.

## Walkthrough
1. Open your team's current product page — whatever exists today, even one rough screen. If nothing is deployed yet, open a competitor or a site your users already use.
2. Press `Ctrl+U` (or `Cmd+Option+U`) to view source. Note roughly how many lines it is, and whether your actual content is in there as text or whether you see mostly empty container elements.
3. Now open DevTools and look at the Elements panel for the same page. Compare: is the content present in both, or only here? Write down which, and what that tells you about how the page is rendered.
4. Go to the Console. Read every line. For each one, note the file it came from on the right-hand side, and mark it "ours" or "extension/third party". Re-check in a private window with extensions disabled to confirm.
5. Open the Network panel, tick "Preserve log" and "Disable cache", then reload. Sort by size. Write down the three largest things the page downloads and what each one is.
6. Still in Network, read the top-level document request: its method, its status code, and how long it took. Then filter to Fetch/XHR and count how many data requests the page makes.
7. Pick one Fetch/XHR request, open it, and read its payload and its response. Write down, in plain words, what it asked for and what came back.
8. Open the device toolbar and set the viewport to 360 x 800. Find one thing that is broken or awkward at that size and screenshot it.
9. Keep the phone viewport and set throttling to "Slow 4G" with CPU throttled 4x. Reload and time how long until you can read the main content. Screenshot the mid-load state — that half-built page is what many of your users actually see.
10. Use the Elements panel to try one fix live: change a colour, a font size or a padding value until it looks right. Note the exact value you landed on. Reload and watch it disappear.
11. Write your findings as five numbered facts, each with the panel it came from and a screenshot. Facts only — no theories about causes yet.

## Talk to AI
```prompt
CONTEXT:      I am a non-coding director of AI, learning to verify work
              instead of trusting it. I have browser DevTools open on my own
              product's page. My users are students in Nashik on budget
              Android phones on slow 4G.

GOAL:         A reusable, ordered inspection checklist I can run on any page
              in under ten minutes, that turns vague complaints into
              evidence an AI can act on.

REQUIREMENTS: 1. For each check: which panel, exactly what to click, what a
                 healthy result looks like, and what an unhealthy one looks
                 like
              2. Checks ordered so the cheapest, most-information checks
                 come first
              3. One section specifically on distinguishing "the browser
                 never asked" from "the server answered badly"
              4. One section on what to capture as evidence, and in what
                 form, so my report is usable without the page in front of
                 the reader
              5. Plain-language explanations of the five console error
                 messages a beginner is most likely to see

CONSTRAINTS:  Browser DevTools only — no installed extensions, no paid
              tools, no command line. Must work on a mid-range Windows
              laptop. Assume I cannot read code, only read output.
              Keep it to one page I could print.

NON-GOALS:    Do not teach me to fix anything. Do not teach me JavaScript
              debugging with breakpoints yet. Do not recommend any product
              or service.

ACCEPTANCE:   - Every check names a panel and a specific thing to click
              - Every check states a pass condition I could not argue with
              - I can tell, from the checklist alone, whether a failure is
                in the browser, in the network, or on the server
              - The evidence section tells me exactly what to paste into my
                next AI message

ASK:          First ask me up to five questions about my page and my setup
              that would change your answer. Then give the checklist. Then
              list the three checks beginners most often skip and what each
              one costs them.
```

## Mission

**Objective.** Produce an evidence-backed inspection report on your own startup's current web page — or, if nothing is deployed yet, on the closest existing product your users already use — and turn its worst finding into a Director's Brief.

**Steps.**
1. **Frame** — write one sentence naming the page you are inspecting, who its users are, and what the page is supposed to achieve. If your team has nothing deployed, say so and name the stand-in product and why it is a fair comparison.
2. **Survey** — run the eleven-step walkthrough on that page. Capture a screenshot for every panel you look at.
3. **Compare** — put view source and the Elements panel side by side. State in two sentences whether the page's content is in the sent HTML, and what that means for a student arriving on slow 4G.
4. **Measure** — with the device toolbar at 360 x 800, throttling at Slow 4G and CPU at 4x, record: total bytes transferred, the three largest files, and the seconds until the main content is readable. These are numbers, not impressions.
5. **Interrogate** — pick the one most important action on the page (submit, order, log in, search). Perform it with "Preserve log" on and record the request's method, status code, payload and response.
6. **Report** — write five numbered findings. Each needs: the panel, what you observed, a screenshot, and one sentence on who it hurts. Facts only.
7. **Rank** — order your five findings by cost to the business, and justify the top one in two sentences.
8. **Direct** — write a full Director's Brief for fixing finding number one. Paste your actual evidence into CONTEXT: the real error text, the real status code, the real numbers. Include at least one NON-GOAL and acceptance criteria that are verifiable in DevTools.
9. **Verify** — have the AI produce the fix, then re-run exactly the checks named in your ACCEPTANCE section and record pass or fail per criterion with fresh screenshots.

**Upload.**
- Your five numbered findings with panel, observation and screenshot for each → post caption or attached PDF
- Your before and after measurements at 360px on Slow 4G → screenshots of the Network panel summary
- Your full Director's Brief for finding one → post caption or attachment
- The AI conversation → share link in `ai_chat_url`
- Proof it works → deployed URL in `demo_url`, or before/after screenshots
- Your acceptance-criteria checklist with pass/fail marked per line → screenshot or text
- Reflection, 150 words max → one thing you believed about your page that the evidence disproved, and one thing the AI got wrong

**Self-check.** Tick every box before you submit.
- [ ] Every finding names the panel it came from and has a screenshot
- [ ] My findings are observations, not theories about causes
- [ ] I recorded total bytes and seconds-to-readable as actual numbers
- [ ] I compared view source against the Elements panel and said what the difference means
- [ ] I captured the real request for the page's most important action, including its status code
- [ ] I ticked "Preserve log" before submitting anything
- [ ] I checked the page once in a private window to rule out extensions
- [ ] My brief pastes real error text and real numbers, not paraphrases
- [ ] Every acceptance criterion in my brief is checkable in a named DevTools panel
- [ ] I re-ran the checks myself after the fix rather than trusting the AI's summary

**Stretch.** Borrow the cheapest Android phone in your group, open the page on real mobile data, and compare the seconds-to-readable against your DevTools throttled number. Write 100 words on the gap and what it means for how you test from now on.

## Stuck? Ask AI
```prompt
I see this in my browser console and I do not understand it: [paste the full error text including the file name and line number]. Explain in plain language what the browser was trying to do, what it found instead, and the three most likely causes in order of likelihood. Then tell me exactly which DevTools panel to open next to tell those causes apart, and what I should look for there. Do not propose a code fix yet.
```

```prompt
I clicked the main button on my page and nothing happened. Give me a decision tree, as a numbered list of yes/no checks in DevTools, that narrows this down. Start with the check that eliminates the most possibilities. For each branch, tell me what a yes and a no each mean, and stop when I would know whether the problem is in the browser, the network or the server.
```

```prompt
Explain why "view page source" and the Elements panel can show completely different things for the same page. Then give me four real-world questions about a page and ask me which of the two I should look at for each one, and why. Do not reveal the answers until I have tried all four.
```

## Quiz

### Q1 · L3
View source on an app shows almost nothing but an empty container element, while the Elements panel shows hundreds of nodes with all your content. What does this tell you?
- (a) The page is broken and failed to send its content
- (b) The content is built in the browser by JavaScript after the empty document arrives ✅
- (c) Someone has obfuscated the HTML to hide it
- (d) The Elements panel is showing a cached older version
> Both views are honest: view source is the sent document, Elements is the live DOM after scripts ran. This is normal for client-rendered apps, and chapter 16 covers the costs of it.

### Q2 · L3
You click a button and nothing happens. Which single check eliminates the most possibilities first?
- (a) Hard reload the page and try again
- (b) Read the CSS applied to the button
- (c) Check the Network panel for whether any request fired at all ✅
- (d) Ask the AI to rewrite the button's handler
> If no request fired, everything about your server, database and API is irrelevant and the problem is entirely in the browser. One look halves the search space.

### Q3 · L4
An agent says "fixed — the handler is now attached". What is the most useful next thing you do?
- (a) Reload and say "still broken" if it is
- (b) Ask it to explain its reasoning in more detail
- (c) Decide which panel would prove the claim, open it, and report what you actually see ✅
- (d) Ask a different AI the same question
> A claim about a page is only worth as much as the panel that can check it. "Still broken" gives the agent nothing; the error text, the status code and the payload give it everything.

### Q4 · L4
You change a padding value in the Elements panel until the layout looks right. What have you achieved?
- (a) Fixed the layout — the change is saved to your file
- (b) Nothing, because DevTools edits cannot show real results
- (c) Found the exact value to put in your next brief, with no change to any file ✅
- (d) Broken the page for other users until you reload
> DevTools edits are local and temporary. That is precisely what makes them the cheapest way to settle a question before you ask for a change — then you brief the specific value.

### Q5 · L5
Your page loads in 1.2 seconds on your laptop and 11 seconds for a student in Nashik. You have DevTools throttling and you have a cheap Android. What is the right order and why?
- (a) Real phone only — throttling is a fake and tells you nothing
- (b) Throttle first to find the problem cheaply, then confirm severity on the real device ✅
- (c) Laptop only, then scale the number up by four
- (d) Real phone first, and skip throttling entirely once it is fast
> Throttling is repeatable, free and available on every reload, so it is where you find and iterate on problems. The real device on real 4G tells you how bad it actually is, which throttling consistently understates.

## Go deeper
- [Chrome DevTools documentation](https://developer.chrome.com/docs/devtools)
- [web.dev: Inspect network activity](https://developer.chrome.com/docs/devtools/network)
