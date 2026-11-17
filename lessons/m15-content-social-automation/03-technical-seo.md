---
id: m15-03-technical-seo
module: m15-content-social-automation
slug: technical-seo
title: "Technical SEO: Sitemaps, Schema, Canonicals and Core Web Vitals"
order: 3
duration_min: 25
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

A team's site has 60 pages. Google has indexed 4. They spend two weeks rewriting headlines, which changes nothing, because none of the problems was about words. The audit finds five plumbing faults: `robots.txt` still carries `Disallow: /` from the staging week; the sitemap lists 19 URLs of which 7 return 404 and 5 carry a `noindex`; every page renders in the browser, so the crawler's first fetch is an empty shell; one product page is reachable at `/p`, `/P`, `/p/` and `/p?ref=wa`, which Google treats as four thin pages; and on a ₹9,000 Android the largest element takes 6.2 seconds. Every one is a yes-or-no question with a free tool that answers it.

### The idea

**Technical SEO is not a ranking trick. It is making sure a machine can reach your page, read it, and keep exactly one copy of it — and every part is a binary check you can run yourself.**

Chapter 2's four stages have a control panel, and this chapter is the first three of it: `sitemap.xml` for discovery, `robots.txt` for crawl permission, and `rel=canonical`, redirects and `noindex` for the index. Using the wrong control for a stage is the commonest mistake in the field.

### The analogy

Back to the campus clerk who files notices. `robots.txt` is the sign at the department gate: "staff rooms, do not enter". The sitemap is the list you hand him: "these 42 boards are worth photographing". The canonical line on each photocopy says "the official version is on the main board", so he files one notice instead of four.

**Where the analogy breaks:** the gate sign is obeyed strictly, so if you say do not enter he never sees the "do not file this" note inside — exactly why `Disallow` plus `noindex` backfires. But the list and the canonical line are only *hints*: he may photograph boards not on your list, and may reject your "official version" claim. Permissions are rules; the rest is advice.

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

Four URL variants and one retired URL point at a single canonical page, so signals that would be split five ways accumulate on one.

```
  Can the bot FETCH it?        Can the bot KEEP it?
  ─────────────────────        ─────────────────────
  robots.txt Disallow          <meta name="robots"
                                     content="noindex">

  blocked  ──▶ never read  ──▶ may STILL be listed
                               from other sites' links

  allowed  ──▶ read         ──▶ noindex obeyed, dropped
```

Crawl permission is not index permission: a blocked URL is never fetched, so its `noindex` is never seen and it can still be listed from others' links. To remove a page, let the bot in and tell it `noindex` — never both.

### Where it lives

This chapter is M02 applied: rendering decides whether the crawler sees text, status codes decide whether a move is permanent, image work decides your LCP. Chapter 4 generates hundreds of pages that each need a canonical and a sitemap entry, and M06's scheduled jobs regenerate that sitemap for you.

### Words to own

| Term | Plain meaning | Example |
|---|---|---|
| robots.txt | File saying which paths bots may fetch | `Disallow: /admin` |
| sitemap.xml | Your list of canonical URLs for crawling | 42 URLs, all 200 |
| Canonical URL | The one true address for a page | `/p` not `/p?ref=wa` |
| noindex | "Read it, do not list it" | On a thank-you page |
| 301 | Permanent redirect; signals transfer | Old URL to new URL |
| 302 / 307 | Temporary redirect; signals do not transfer | A maintenance page |
| Crawl budget | How much crawling your site gets | Wasted on filter URLs |
| Structured data | Machine-readable facts on the page | JSON-LD LocalBusiness |
| hreflang | Declares language alternates | `mr-IN` and `en-IN` versions |
| LCP · INP · CLS | Biggest element appears · tap response · layout jump | 2.1s · 180ms · 0.04 |
| Field data | Real users' measurements | Budget Android on 4G |
| Lab data | A simulated test run | A laptop score of 98 |

### Tradeoffs and gotchas

**`robots.txt` is the most dangerous file you own.** One stray `Disallow: /` removes you from search, so check it after every deploy.

**A permanent move must return 301.** `302` and `307` are temporary and transfer no signals, keeping the old URL indexed for months. Ask for the number, never the word "redirect".

**A canonical is a hint, and a sitemap is only discovery.** If every internal link points at `/p?ref=wa`, the engine may pick that over your canonical. A sitemap must carry only URLs that are 200, indexable and canonical, and a thin page stays thin after you add it. `hreflang` stops your Marathi page competing with your English one.

**Client-side rendering is a crawl risk, not a ban.** Google renders JavaScript, but at a cost and a delay, and several social preview bots from chapter 8 do not render at all.

**Field data is the only vitals number that counts.** Lab data is a simulated run on a tool's machine; field data comes from real Chrome users on budget Androids on congested 4G. A lab score of 98 beside a field LCP of 6.2s is normal — and means you are failing. Target field LCP under 2.5s, INP under 200ms, CLS under 0.1.

**Schema that overstates is worse than no schema.** `LocalBusiness` with your real address, phone and hours is the highest-value markup an Indian local business can add, but it must describe what a human sees. Invented ratings lose you rich results sitewide.

### Verify it

**Red flag: a sitemap listing pages that 404.** Fetch every sitemap URL and record its status code. Any non-200 is a defect, as is any URL that redirects, is noindexed, or is not its own canonical. Highest-value check here.

**Red flag: generated config never checked against reality.** AI will produce a confident `robots.txt` referencing `/wp-admin/` on a site that is not WordPress. Every rule must name a path you have.

**Red flag: `Disallow` and `noindex` on the same URL.** The contradiction above, which keeps the page listed.

**Red flag: schema whose values do not appear on the page.** Compare every field against the rendered page. A price in markup only is a fabrication.

**Red flag: a lab score quoted as proof.** Demand mobile field data. "No field data yet" is honest; substituting the lab number is not.

**Red flag: "indexed" claimed without inspection.** Only the URL inspection tool decides that — not the AI, and not you.

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
