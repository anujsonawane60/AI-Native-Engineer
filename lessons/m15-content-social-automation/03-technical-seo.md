---
id: m15-03-technical-seo
module: m15-content-social-automation
slug: technical-seo
title: "Technical SEO: Sitemaps, Schema, Canonicals and Core Web Vitals"
order: 3
duration_min: 35
is_free: false
mission: true
volatility: medium
last_reviewed: 2026-11-16
---

## Key learnings
- Fix a page stuck at discover, crawl or index using the mechanism that actually controls that stage
- Keep one canonical URL per page so duplicates stop splitting your ranking signals
- Ship a sitemap that matches what your site really serves, with no 404s and no noindexed URLs
- Read Core Web Vitals from real mobile field data instead of a laptop lab score
- Catch structured data that claims something the visible page does not show

## Lesson

### The pain

A team's site has 60 pages. Google has indexed 4. They spend two weeks rewriting headlines, which changes nothing, because not one of the problems was about words.

The audit, when they finally run it, finds five separate plumbing faults. Their `robots.txt` still carries `Disallow: /` from the week they put the site behind a staging password. Their sitemap lists 19 URLs, of which 7 return 404 and 5 carry a `noindex` tag — so the file is simultaneously incomplete and lying. Every page renders entirely in the browser, so the crawler's first fetch returns an empty shell with a spinner. The same product page is reachable at `/p`, `/P`, `/p/` and `/p?ref=wa`, which Google treats as four pages that each look thin. And on a real ₹9,000 Android phone on 4G, the largest element on screen takes 6.2 seconds to appear.

Every one of those five is a yes-or-no question with a free tool that answers it. None of them is a matter of opinion. That is what makes this chapter the most mechanical in the module and the highest-return hour in it.

### The idea

**Technical SEO is not a ranking trick. It is making sure a machine can reach your page, read your page, and keep exactly one copy of it — and every part of that is a binary check you can run yourself.**

Chapter 2 gave you four stages: discover, crawl, index, rank. This chapter is the control panel for the first three. Each stage has its own mechanism, and using the wrong mechanism is the most common mistake in the whole field.

### The analogy

Back to the campus clerk who files notices. `robots.txt` is the sign at the department gate: "staff rooms, do not enter". The sitemap is the typed list you hand him: "here are the 42 boards worth photographing". The canonical tag is the line at the bottom of each photocopy saying "the official version of this notice is on the main board" — so he files one notice, not four. A 301 redirect is a permanent card on the empty board saying "this notice now lives at the library entrance". And a `noindex` tag is a note saying "read this, but do not put it in the register".

**Where the analogy breaks** in two important ways. The gate sign is obeyed strictly — if you say do not enter, he does not enter, and therefore never sees the "do not file this" note inside, which is exactly why `Disallow` plus `noindex` is a contradiction that backfires. But the list and the canonical line are only *hints*: he may photograph boards that are not on your list, and he may decide your "official version" claim is wrong and pick a different one. Permissions are rules; the rest is advice.

### Picture it

```
  /p?ref=wa ──┐
  /P         ──┤                           ┌──────────────┐
  /p/        ──┼── rel=canonical ─────────▶│  /p          │
              │                            │  ONE page,   │
  /old-p ─────┴── 301 Moved Permanently ──▶│  ONE set of  │
                                           │  signals     │
                                           └──────────────┘
```

Four URL variants and one retired URL all point at a single canonical page, so the links, impressions and relevance that would otherwise be split five ways accumulate on one page instead.

### Crawl permission is not index permission

This single distinction causes more wasted weeks than anything else in technical SEO.

```
  Can the bot FETCH it?        Can the bot KEEP it?
  ─────────────────────        ─────────────────────
  robots.txt Disallow          <meta name="robots"
                                     content="noindex">

  blocked  ──▶ never read  ──▶ may STILL be listed
                               from other sites' links

  allowed  ──▶ read         ──▶ noindex obeyed, dropped
```

A URL blocked in `robots.txt` is never fetched, so a `noindex` tag inside it is never seen — and the URL can still appear in results from third-party links; to remove a page you must let the bot in and tell it `noindex`.

So: use `robots.txt` to save crawl effort on things like internal search results or filter permutations. Use `noindex` to keep a page out of the index. Never both on the same URL.

### The five mechanisms, and what each controls

**`sitemap.xml`** — discovery. A list of the canonical URLs you want crawled, with a last-modified date. It must contain only URLs that return 200, are indexable, and are canonical. A sitemap full of 404s and redirects trains the crawler to trust it less.

**`robots.txt`** — crawl permission and crawl effort. One line of carelessness here can delete your entire search presence, which is why it is the first thing to check after any deploy.

**`rel=canonical`** — consolidation. Every page declares its own true URL. Tracking parameters, uppercase variants, trailing slashes and printer versions all point home.

**Redirects** — moving. M02's status codes matter here and the distinction is not academic. `301` is permanent, and search engines transfer signals to the destination. `302` and `307` are temporary, and they do not. A permanent move served as a `302` keeps the old URL alive in the index for months.

**`noindex`** — index control, for pages that must exist but must not rank: thank-you pages, internal search results, staging routes, thin filter combinations.

One more for Indian sites: **`hreflang`** tells the engine that your English, Hindi and Marathi versions of a page are alternates of each other rather than duplicates. Get it wrong and your Marathi page competes with your English one.

### Structured data, honestly

Structured data is machine-readable facts added to a page — usually JSON-LD — describing what the page is: a product with a price, a local business with opening hours, a recipe, an FAQ, a review. It can earn **rich results**: star ratings, prices or a map card in the SERP, which lifts click-through even without a ranking change. For a local Indian business, `LocalBusiness` with the real address, phone and hours is the single highest-value piece you can add.

The rule that matters: **the markup must describe what a human actually sees on the page.** Rating markup with no visible reviews, a price in the markup different from the price on the page, or FAQ markup for questions nobody asked is spam, it is detected, and the penalty is losing rich results sitewide. Mark up what is true and visible. Nothing else.

### Core Web Vitals, on the phones your users own

M02 defined them: **LCP** is when the biggest thing on screen finishes appearing, **INP** is how long the page takes to respond to a tap, and **CLS** is how much the layout jumps while loading. The targets are roughly LCP under 2.5s, INP under 200ms, CLS under 0.1.

Two numbers exist for each, and students confuse them constantly. **Lab data** is a simulated run on a tool's machine — fast, repeatable, and not your users. **Field data** is collected from real Chrome users on real devices and networks, which for you means budget Androids on congested 4G in Maharashtra. Field data is the one search uses. A lab score of 98 and a field LCP of 6.2 seconds is a completely normal combination, and it means you are failing.

Core Web Vitals are a small ranking factor. They are a large *conversion* factor: a page that takes 6 seconds loses most of its visitors before chapter 10 can even count them.

### Where it lives

This chapter is M02 applied: rendering decides whether the crawler sees text, status codes decide whether a move is permanent, CDN and image work decides your LCP. M11 deployed the site this audit runs against. Chapter 4 generates hundreds of pages, every one of which needs a correct canonical and a sitemap entry, so getting the mechanism right here is what makes that chapter safe. M06's scheduled jobs are how you regenerate a sitemap automatically instead of by hand.

### Words to own

| Term | Plain meaning | Example |
|---|---|---|
| robots.txt | File saying which paths bots may fetch | `Disallow: /admin` |
| sitemap.xml | Your list of canonical URLs for crawling | 42 URLs, all 200 |
| Canonical URL | The one true address for a page | `/p` not `/p?ref=wa` |
| noindex | "Read it, do not list it" | On a thank-you page |
| 301 | Permanent redirect; signals transfer | Old URL to new URL |
| 302 / 307 | Temporary redirect; signals do not transfer | A maintenance page |
| Soft 404 | Says 200 but shows "not found" | An empty results page |
| Crawl budget | How much crawling your site gets | Wasted on filter URLs |
| Structured data | Machine-readable facts on the page | JSON-LD LocalBusiness |
| Rich result | A SERP listing with extra detail | Price and stars shown |
| hreflang | Declares language alternates | `hi-IN` and `en-IN` versions |
| LCP | When the biggest element appears | 2.1s on 4G |
| INP | Delay before the page reacts to a tap | 180ms |
| CLS | How much the layout jumps | 0.04 |
| Field data | Real users' measurements | Budget Android on 4G |
| Lab data | A simulated test run | A laptop score of 98 |

### Tradeoffs and gotchas

**`robots.txt` is the most dangerous file you own.** One stray `Disallow: /` removes you from search. Check it after every deploy, as a habit, forever.

**Do not block what you want removed.** Blocking plus noindex means the noindex is never read. Let it be crawled, then noindex it.

**Client-side rendering is a crawl risk, not a crawl ban.** Google does render JavaScript, but it costs crawl budget, it is delayed, and other crawlers — including several social preview bots from chapter 8 — do not do it at all. Server-rendered HTML for anything you want found is simply safer.

**A canonical is a hint.** If your canonical says `/p` but every internal link points at `/p?ref=wa`, the engine may pick the parameter version. Make your internal links agree with your canonicals.

**Sitemaps do not make pages rank.** They help discovery only. A page that is thin stays thin after you add it to the sitemap.

**Chasing a lab score of 100 is a waste of your term.** Get LCP under 2.5s in field data on a mid-range Android and stop. The next ten points cost more than they return.

**Schema that overstates is worse than no schema.** Losing rich results sitewide for faking reviews is a real and common outcome.

### Verify it

Technical SEO is where AI output is most confidently wrong, because it will happily generate a plausible file that does not match your actual site.

**Red flag: a sitemap listing pages that 404.** Fetch every URL in your sitemap and record its status code. Any non-200 is a defect, and so is any URL that redirects, is noindexed, or is not its own canonical. This is the fastest, highest-value check in the chapter.

**Red flag: generated config that was never checked against reality.** AI will produce a confident `robots.txt` referencing `/wp-admin/` on a site that is not WordPress. Every rule must point at a path you actually have.

**Red flag: `Disallow` and `noindex` on the same URL.** A contradiction that keeps the page in the index.

**Red flag: schema whose values do not appear on the page.** Open the rendered page beside the markup and compare every field. A phone number or price in markup only is a fabrication.

**Red flag: a lab score quoted as proof.** Ask for field data for mobile. If there is not enough traffic for field data yet, say so honestly rather than substituting the lab number.

**Red flag: 302 for a permanent move.** Ask for the status code by number, not the word "redirect".

**Red flag: "indexed" claimed without inspection.** Only the URL inspection tool decides that. Not the AI, and not you.

## Walkthrough
1. Open `yourdomain.com/robots.txt` in a browser. Read every line aloud and confirm each path exists and is meant to be blocked. Look especially for a leftover `Disallow: /`.
2. Open `yourdomain.com/sitemap.xml`. Count the URLs. Compare that count with the number of pages you believe you published, and write down the difference.
3. Take every URL in the sitemap and check its status code. Record each one. Mark every URL that is not a 200, is not canonical, or carries a noindex.
4. In Search Console, open the Pages report and write down how many URLs are indexed and the top three reasons given for the ones that are not.
5. Run the URL inspection tool on your three most important pages. Use the "view crawled page" option and check whether your real text is present in the HTML the bot received, or only a loading shell.
6. Visit one of your pages four ways: with a trailing slash, without, with an uppercase letter, and with `?ref=test` appended. Note which variants load. Then view source and read the canonical tag on each.
7. Pick one page that you have moved or renamed since launch. Check the old URL's status code and confirm it is 301 and not 302.
8. Add or correct one piece of structured data appropriate to your product, then run it through a free rich-results test and record every error and warning.
9. Open your page's field data for mobile in the free PageSpeed report. Write down LCP, INP and CLS from the field section, not the lab section. If there is no field data, say so and record the lab numbers clearly labelled as lab.
10. Write the five faults you found, ranked by how many pages each one affects. Fix the top one and nothing else, then re-run the exact check that found it.

## Talk to AI
```prompt
CONTEXT:      I am a student founder. I do not write code; I direct AI.
              My site: [URL]. Stack: [e.g. Next.js on Vercel free tier].
              Rendering: [server-rendered / client-rendered / mixed].
              Languages served: [e.g. English and Marathi].

              What I measured myself, not what I assume:
              - robots.txt contents: [paste the whole file]
              - sitemap URL count: [X]; of those, non-200: [X]
              - Search Console: [X] indexed, top exclusion reasons:
                [paste them]
              - URL variants that load: [/p, /p/, /P, /p?ref=test - say
                which return 200]
              - canonical tag found on /p: [paste it]
              - field data for mobile: LCP [X]s, INP [X]ms, CLS [X]
                (or "no field data yet")
              - pages I believe I published: [X]

GOAL:         A ranked list of the technical faults stopping my pages
              being discovered, crawled or indexed, with the right
              mechanism for each.

REQUIREMENTS: 1. For every fault, say which stage it breaks: discover,
                 crawl or index. One stage each.
              2. For every fault, name the ONE correct mechanism -
                 robots.txt, sitemap, canonical, redirect status code or
                 noindex - and say why the other four are wrong here
              3. Rank faults by how many of my pages each affects, using
                 my numbers
              4. Give exact redirect status codes as numbers, never the
                 word "redirect"
              5. For each fix, give the exact free check that proves it
                 worked, and what the passing output looks like

CONSTRAINTS:  Free tiers only. I cannot change hosting this week. Do not
              assume WordPress or any CMS I did not name. Only reference
              paths that appear in the robots.txt and URLs I pasted.

NON-GOALS:    Do not write content. Do not discuss keywords. Do not
              suggest paid SEO tools. Do not propose a redesign. Do not
              chase a lab score of 100.

ACCEPTANCE:   - Every fault cites one of MY pasted measurements
              - No recommendation mentions a file or path I did not give
                you
              - You explicitly check whether any URL is both disallowed
                and noindexed, and flag it if so
              - You tell me whether my field data or only lab data was
                used, and refuse to treat lab as field
              - A section titled "What I cannot tell from your data",
                with the exact extra measurement needed

ASK:          First, list any measurements I gave you that contradict
              each other - for example a sitemap count that cannot match
              my indexed count. Ask me to re-check those before you
              diagnose anything.
```

## Mission

**Objective.** Every page you meant to publish is confirmed indexed or has a named reason why not; every page has one canonical URL; your sitemap contains only live canonical URLs; one piece of structured data validates against what the page actually shows; and your mobile Core Web Vitals are recorded as numbers.

**Steps.**
1. **Frame** — write down how many pages you intended to publish and how many Search Console says are indexed. The gap is the mission.
2. **Brief** — collect the real measurements from the walkthrough: robots.txt, sitemap status codes, indexed count and exclusion reasons, URL variants, canonical tags, field vitals. Paste them into the Director's Brief. Do not let AI guess any of them.
3. **Direct** — ask for faults ranked by pages affected, each mapped to exactly one stage and one mechanism. Fix them one at a time, re-running the specific check after each.
4. **Canonicalise** — pick your single most important page and make all four URL variants resolve to one canonical form, with internal links agreeing.
5. **Sitemap** — regenerate it so every entry returns 200, is canonical, and is indexable. Re-fetch every URL to prove it.
6. **Schema** — add one structured-data type that genuinely matches your page, with every value also visible to a human. Validate it and fix all errors.
7. **Vitals** — record mobile LCP, INP and CLS. If the field data fails the thresholds, make exactly one fix and re-measure.
8. **Verify** — re-run URL inspection on three pages and record the before and after. Submit the corrected sitemap.
9. **Document** — one decision-log entry naming the fault that affected the most pages, and what you chose not to fix and why.

**Upload.**
- Your Director's Brief with the real measurements in it → attached file or post caption
- Your `robots.txt` before and after → screenshots
- A table of every sitemap URL with its status code, before and after → screenshot or text
- Search Console indexed count before and after, and the URL inspection results for three pages → screenshots
- The rich-results validation output for your structured data, with zero errors → screenshot
- Mobile LCP, INP and CLS, clearly labelled field or lab → screenshot
- Your AI conversation → share link in `ai_chat_url`
- Your deployed site → `demo_url`
- Reflection, 150 words max → one tradeoff you chose, one thing the AI got wrong

**Self-check.** Tick every box before you submit.
- [ ] I fetched every URL in my sitemap and recorded its status code myself
- [ ] Zero sitemap URLs return 404, redirect, or carry a noindex
- [ ] No URL on my site is both disallowed in robots.txt and noindexed
- [ ] All four variants of my main page resolve to one canonical URL
- [ ] My internal links point at the canonical form, not a parameter version
- [ ] Any permanent move returns 301, and I checked the number, not the word
- [ ] Every value in my structured data is visible to a human on the page
- [ ] My vitals are labelled field or lab, and I did not pass off lab as field
- [ ] I used URL inspection to confirm indexing rather than assuming it
- [ ] My reflection names a tradeoff in my own words

**Stretch.** Wire sitemap regeneration into a scheduled job using M06's cron work, so the file cannot drift from reality again.

## Stuck? Ask AI
```prompt
I do not understand the difference between robots.txt and a noindex tag. Explain both as if I have never built a website, using the idea of a person who photographs noticeboards and files them in a register. Then give me four scenarios and ask me which mechanism each one needs, one at a time, before you tell me the answer.
```

```prompt
Here is my sitemap: [paste the URLs]. Here is the status code I got for each one when I fetched it: [paste them]. Do not rewrite my sitemap. Instead, tell me exactly which entries are defects and what kind of defect each one is, and explain what a crawler concludes about a site whose sitemap is mostly wrong.
```

```prompt
My PageSpeed lab score is [X] but my site feels slow on a cheap Android phone on 4G. Explain the difference between lab data and field data, which one search actually uses, and why these two numbers disagree so often. Then tell me the three measurements I should record instead of the score.
```

## Quiz

### Q1 · L3
You want a thank-you page removed from Google. You add `Disallow: /thank-you` to robots.txt and a `noindex` tag to the page. What happens?
- (a) The page is removed twice as fast
- (b) The bot never fetches the page, so it never reads the noindex and the URL can stay listed ✅
- (c) robots.txt overrides the tag and the page is removed cleanly
- (d) The page returns a 404 to crawlers
> Crawl permission and index permission are different controls. To honour a noindex the bot must be allowed to fetch the page, so blocking it in robots.txt defeats the removal.

### Q2 · L2
What does a `rel=canonical` tag do?
- (a) Blocks crawlers from the page
- (b) Declares the one true URL for a page so duplicates consolidate onto it ✅
- (c) Guarantees the page will be indexed
- (d) Redirects visitors to the main version
> It is a consolidation hint. Four URL variants pointing at one canonical stop your links and impressions being split four ways — but it is a hint, not a command.

### Q3 · L4
Your sitemap lists 19 URLs; 7 return 404 and 5 carry a noindex. Why is this worse than having no sitemap?
- (a) Google penalises sites with broken sitemaps
- (b) The file is your own statement of what matters, so being mostly wrong wastes crawl effort and reduces trust in it ✅
- (c) A sitemap must contain at least 20 URLs to be valid
- (d) 404s in a sitemap cause the whole site to be deindexed
> A sitemap is a claim about your canonical, indexable pages. Filling it with dead and excluded URLs makes it unreliable as a discovery source, which is the only job it has.

### Q4 · L4
You moved `/old-pricing` to `/pricing` permanently and served a 302. What is the consequence?
- (a) Nothing; 301 and 302 behave identically
- (b) The move is treated as temporary, so signals stay with the old URL and it lingers in the index ✅
- (c) Visitors see an error page
- (d) The new URL is blocked from being crawled
> 301 is permanent and transfers signals; 302 and 307 are temporary and do not. This is why the brief should always demand the status code as a number rather than the word "redirect".

### Q5 · L5
An AI adds `AggregateRating` markup showing 4.8 stars to your product page, which displays no reviews at all. Your clicks rise. What is the real situation?
- (a) A clever win; the markup is technically valid
- (b) The markup claims something not visible on the page, which is spam and risks losing rich results sitewide ✅
- (c) Harmless, because ratings are not a ranking factor
- (d) Fine as long as you add reviews later
> Structured data must describe what a human actually sees. Invented ratings are detectable and the penalty applies across the whole site, which is a large downside for a temporary click bump.

## Go deeper
- [Google Search Central: sitemaps, robots.txt and canonicalization docs](https://developers.google.com/search/docs/crawling-indexing)
- [web.dev: Core Web Vitals and field data](https://web.dev/articles/vitals)
