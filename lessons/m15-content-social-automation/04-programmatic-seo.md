---
id: m15-04-programmatic-seo
module: m15-content-social-automation
slug: programmatic-seo
title: "Programmatic and Landing-Page SEO at Scale"
order: 4
duration_min: 25
is_free: false
mission: true
volatility: medium
last_reviewed: 2026-11-16
---

## Key learnings
- Treat programmatic SEO as a data problem, where the dataset and the eligibility rule matter more than the template
- Write an eligibility gate that decides which generated pages are even allowed to exist
- Test a generated page for unique value before publishing four hundred of its siblings
- Keep hundreds of pages discoverable with hub pages, internal links and sitemap sections
- Spot thin pages, index bloat and fabricated data in AI-generated page sets

## Lesson

### The pain

A team covering two localities in Nashik generates 1,800 pages: `best {diet} tiffin service in {city}` across 300 cities and six diets. The 1,780 pages with no data behind them all say "We couldn't find any veg tiffin services in Bhopal yet. Browse our top picks!" — and three invent providers, because the AI filled the template with plausible names, addresses and ₹ prices that do not exist. Google indexes 90, drops 60 over three weeks, and total search traffic falls below where it started. The two genuinely good pages now sit inside a site the crawler distrusts. The template worked perfectly; publishing 1,780 empty pages was the failure.

### The idea

**Programmatic SEO is a data problem wearing a content costume. The template is ten percent of the work; the dataset and the rule deciding which pages are allowed to exist are the other ninety.**

One dataset, one template, one page per row. What separates a set that earns traffic for years from one that gets your site demoted is one question per row — *does this page have something that exists nowhere else?* You answer it before generating anything, with an **eligibility gate**.

- At least three real records in this combination, not one and not zero
- Every record has a real, current ₹ price or a stated "price on request"
- The locality is one a human on the team has actually verified
- The page will contain one fact a competitor's page does not have

A row that fails gets **no page at all** — not a "coming soon" page, which is a soft 404 and a doorway page. If the URL must keep working, serve it `noindex`. The gate is also your growth plan: every real provider you add moves rows closer to eligibility.

### The analogy

The canteen prints a price board per hostel block, because block A really gets a 7 pm dinner at ₹45 while block D gets 8:30 pm at ₹55. The printing is mechanical; the usefulness comes from the numbers differing. Now print 300 boards, one per hostel in Maharashtra, most reading "no menu yet". Students stop reading boards and the twelve good ones are buried.

**Where the analogy breaks** in exactly the way that causes the problem: paper costs money, so nobody would print 300 useless boards. Generating 1,800 pages costs nothing and takes one prompt. The discipline physical things impose for free must be imposed deliberately here.

### Picture it

```
 ┌─────────────┐   ┌──────────────┐   ┌───────────────┐
 │ DATASET     │   │ ELIGIBILITY  │   │ TEMPLATE      │
 │ 140 real    │   │ GATE         │   │ one page per  │
 │ providers,  │──▶│ 3+ rows AND  │──▶│ allowed combo │
 │ 11 areas,   │   │ real price   │   │ + its own     │
 │ 4 diets     │   │ AND verified │   │ unique data   │
 └─────────────┘   └──────┬───────┘   └───────┬───────┘
  44 combinations         │                   │
                    25 rejected          19 live pages,
                    (no page at all)     in the sitemap
```

The dataset proposes 44 pages, the gate rejects 25 for having too little real data, and only the surviving 19 are generated and submitted for indexing.

### Where it lives

M04 is the foundation: a schema, a query and a join, where the gate is a `WHERE` clause and the page set is a view over your real data. M06's scheduled jobs regenerate the pages and sitemap when the data changes; chapter 3's mechanisms keep ineligible rows out of the index.

Generated pages are orphans by default, so discovery fails before ranking is relevant. Build a **hub page** listing every eligible page, link it from your navigation, give each page a link back plus three sibling links, and add the URLs to your sitemap. Then click from your home page to three random generated pages: if only the URL reaches them, the crawler is stuck too.

### Words to own

| Term | Plain meaning | Example |
|---|---|---|
| Programmatic SEO | Pages generated from data plus a template | One page per locality |
| Template | The fixed page shape every row fills | Title, table, map, paragraph |
| Row | One record that becomes one page | Gangapur Road, veg |
| Modifier | The varying part of the query pattern | `{area}`, `{diet}` |
| Eligibility gate | The rule a row must pass to get a page | 3+ verified providers |
| Thin content | A page with nothing on it worth reading | "No results yet" |
| Index bloat | Thousands of low-value URLs indexed | 1,800 pages, 2 useful |
| Doorway page | A page existing only to catch a query | 300 city pages, no data |
| Hub page | A page linking to every generated page | "All areas we cover" |
| Orphan page | No internal link points to it | A page nobody links |

### Tradeoffs and gotchas

**Thirty good pages beat three thousand empty ones.** Start with the smallest set that passes your gate: 3,900 thin URLs eat the crawl allowance your good pages needed. Deleting is normal — zero impressions in eight weeks and no unique data, and removing it helps everything else.

**Never let AI invent the data.** An AI asked to fill 400 templates will produce names, addresses, phone numbers and ₹ prices that look right and are fabricated. A user who calls an invented number is a trust loss you cannot undo, and in some categories a legal problem. Facts come from your database; the AI gets structure and wording only.

**Unique value has four honest sources**, in descending durability: your own data (prices you collected, timings you verified — a competitor cannot copy a number they do not have), aggregation nobody else bothered with, street-level local specificity, and one human-written sentence per page. Thirty pages is thirty sentences, one evening's work, and the strongest signal the set was not mass-produced. A reworded intro, a rotating adjective or filler padded to a word count is not unique value. Shared structure is fine — every category page shares one; structure with nothing different inside is not.

### Verify it

**Red flag: a page with no unique value.** The delete test: if this page vanished tonight, would any human notice? Put three generated pages side by side and highlight every piece of text that differs. If the only differences are the place name and an adjective, the set is thin.

**Red flag: data you cannot trace to a database row.** Pick five facts at random from five pages — a price, a phone number, an opening time — and find each in your real data. Any fact you cannot trace was invented.

**Red flag: "coming soon" as a template branch.** A branch for having no data means the gate is missing. For no data, the right page count is zero.

**Red flag: indexation unchecked.** Three weeks after publishing, compare pages published against pages indexed. If 40 of 400 are indexed, the set is judged low-value and more pages make it worse.

## Walkthrough
1. Write the query pattern you believe your users search, using braces for the varying parts, and justify it with one real query from your chapter 2 list.
2. Export the real data you actually have into a spreadsheet: one row per combination of your modifiers, with a count of real records and the fields you can honestly show.
3. Count how many combinations the pattern allows in theory. Write that number down. It will be much larger than the next one.
4. Write your eligibility gate as three or four conditions, each checkable in the spreadsheet. Apply it and count the surviving rows.
5. For the surviving rows, list exactly which fields differ between pages. If the only differing field is the place name, go back and find more real data before continuing.
6. Design the template by hand for ONE row first: title, the unique data block, a human paragraph, internal links, and nothing decorative. Publish that single page and look at it on a phone.
7. Write the human paragraph for each surviving row yourself. One true sentence per page is enough; this is the step teams skip and the one that separates the set from spam.
8. Generate the rest, then pick five random pages and trace five facts from them back to rows in your database. Record any fact you cannot find.
9. Build the hub page, link it from your navigation, and add sibling links on each page. Click from your home page to three random generated pages to prove they are not orphans.
10. Add the pages to your sitemap, confirm every URL returns 200 and is canonical, and submit it. Diary a date three weeks out to record how many of them got indexed.

## Talk to AI
```prompt
CONTEXT:      I am a student founder. I do not write code; I direct AI.
              My product: [one sentence]. My city and region: [e.g.
              Nashik, Maharashtra].

              The REAL data I actually have, and nothing more:
              - table and fields: [paste the column names]
              - total records: [X]
              - distinct values per modifier: [e.g. 11 areas, 4 diets]
              - fields I can show publicly and can prove are current:
                [list them]
              - fields that are missing or stale for many records:
                [list them]

              The query pattern I think my users search, with one real
              user query that supports it: [pattern] / [real query]

GOAL:         Decide which programmatic pages are allowed to exist, and
              design one template for them.

REQUIREMENTS: 1. Write an eligibility gate as 3-4 conditions I can check
                 in a spreadsheet, using only the fields I listed
              2. Tell me how many combinations the pattern allows, and
                 how many survive the gate, using my numbers
              3. For surviving rows, list exactly which fields differ
                 between pages, and judge whether that is enough
                 unique value to justify separate pages
              4. Design ONE template: the shortest page that fully
                 serves the intent, with every block labelled as from
                 my data or written by a human
              5. Specify the internal linking: hub page, sibling links
                 and where each generated page links back to

CONSTRAINTS:  Free tiers only. I have three people. Every fact on every
              page must come from a row in my database. You may write
              structure and wording; you may NOT write facts.

NON-GOALS:    Do not invent provider names, addresses, phone numbers,
              prices or ratings - not even as examples. Do not write a
              "coming soon" branch for rows with no data. Do not set a
              word count. Do not propose pages for places I never
              listed.

ACCEPTANCE:   - A specific number of allowed pages, derived from my data
              - Every template block is tagged DATA or HUMAN-WRITTEN,
                and no block is tagged AI-WRITTEN-FACTS
              - You say plainly if my data is too thin to justify
                programmatic pages at all
              - A section titled "Rows I would reject and why"
              - A section titled "How this set becomes spam" naming the
                three most likely ways my team drifts into thin pages

ASK:          Before designing anything, tell me the number of pages a
              careless team would generate from my pattern, and the
              number my data actually supports. Then explain the gap.
```

## Mission

**Objective.** A live set of programmatic pages — as many as your real data honestly supports and no more — each with traceable facts, a human-written sentence, working internal links, and a recorded indexation count three weeks later.

**Steps.**
1. **Frame** — write the query pattern and the one real user query that justifies it. One sentence on the intent the page set serves.
2. **Data** — export your real rows into a spreadsheet. Count combinations in theory and records per combination in practice.
3. **Gate** — write three or four eligibility conditions and apply them. Record how many rows you rejected and why. The rejected count is part of the deliverable.
4. **Template** — direct AI to design one template for one row. Tag every block as coming from your data or written by a human. Build and check that single page on a phone first.
5. **Human pass** — write one true sentence per surviving page yourself, by hand.
6. **Generate** — produce the rest from the dataset. No "coming soon" branch exists.
7. **Verify** — trace five random facts from five random pages back to database rows. Click from your home page to three generated pages. Confirm each URL returns 200, is canonical and is in the sitemap.
8. **Submit** — submit the sitemap and record the date.
9. **Measure** — three weeks later, record pages published versus pages indexed, and total impressions. Decide what to delete.

**Upload.**
- Your Director's Brief → attached file or post caption
- The spreadsheet showing combinations allowed, rows rejected and rows surviving → screenshot
- Your eligibility gate, written as explicit conditions → text
- Three generated pages side by side with the differing fields highlighted → screenshot
- The traceability check: five facts, five database rows → screenshot or table
- Proof the pages are not orphans: the click path from your home page → screenshots
- Your live pages → `demo_url`
- Your AI conversation → share link in `ai_chat_url`
- Indexation count after three weeks → screenshot of Search Console
- Reflection, 150 words max → one tradeoff you chose, one thing the AI got wrong

**Self-check.** Tick every box before you submit.
- [ ] My eligibility gate is written down and every published page passes it
- [ ] Zero published pages say "no results" or "coming soon"
- [ ] I traced five random facts back to real database rows and found all five
- [ ] No page contains a name, price, address or phone number the AI produced
- [ ] Each page has at least one sentence a human on my team wrote
- [ ] Three generated pages placed side by side differ in more than the place name
- [ ] Every generated page is reachable by clicking from my home page
- [ ] Every generated URL returns 200, is canonical and is in my sitemap
- [ ] I recorded how many rows I rejected, not just how many I published
- [ ] My reflection names a tradeoff in my own words

**Stretch.** Wire regeneration into M06's scheduled jobs so that a row becoming eligible creates its page automatically, and a row falling below the gate removes it.

## Stuck? Ask AI
```prompt
Explain programmatic SEO to me as if I have never built a website, using a canteen printing one price board per hostel block. Then explain, with the same example, what makes 300 boards worse than 12. Finally ask me three questions about my own data before suggesting anything.
```

```prompt
Here are three of my generated pages, pasted in full: [paste them]. Do not improve them. Instead, highlight every piece of text that is identical across all three, and every piece that genuinely differs. Then tell me honestly whether a human would notice if any one of these pages were deleted tonight.
```

```prompt
I have [X] records covering [Y] areas and [Z] categories. Help me write an eligibility gate: the conditions a combination must meet before it deserves its own page. Give me three candidate gates of increasing strictness, tell me how many pages each would allow, and ask me which I can defend in front of a user before you recommend one.
```

## Quiz

### Q1 · L3
A team generates 1,800 city pages but has data for two localities. What is the most likely result?
- (a) Traffic rises because there are 1,800 chances to rank
- (b) Index bloat and thin content, with the site's existing good pages losing visibility ✅
- (c) Google ignores the thin pages and ranks the two good ones higher
- (d) The pages rank for a while and then stay flat
> Thousands of empty URLs consume crawl budget and signal low quality sitewide. The two genuinely useful pages end up inside a site the crawler has learned to distrust.

### Q2 · L4
Which of these is the correct behaviour for a row with no real data behind it?
- (a) Publish a page saying "coming soon" to capture the query early
- (b) Publish no page at all, or serve the URL noindex until it qualifies ✅
- (c) Publish the page and fill it with AI-written general advice
- (d) Publish the page and add it to the sitemap so Google learns it exists
> A "coming soon" page is a soft 404 and a doorway page. The eligibility gate exists precisely so that rows without data produce nothing.

### Q3 · L4
You ask AI to fill 400 location templates. It returns pages with provider names, phone numbers and ₹ prices. What is the problem?
- (a) Nothing, as long as the formatting is consistent
- (b) The facts are fabricated, which misleads users and can be a legal problem ✅
- (c) The prices will be out of date within a month
- (d) 400 pages is too many for one sitemap file
> Facts come from your database; the model may write structure and wording only. A user who calls an invented number loses trust permanently, and in regulated categories this is worse than a trust problem.

### Q4 · L2
What does a hub page do for a programmatic page set?
- (a) Improves page speed by caching the set
- (b) Gives every generated page an internal link so it can be discovered ✅
- (c) Replaces the need for a sitemap
- (d) Prevents duplicate content between pages
> Generated pages are orphans by default. A hub linked from your navigation, plus sibling links, is how discovery — chapter 2's first stage — actually happens.

### Q5 · L5
Three weeks after publishing 400 pages, Search Console shows 40 indexed. Your teammate suggests generating 400 more. What should you argue?
- (a) Agree — more pages increase the chance of indexation
- (b) A 10% indexation rate means the set is being judged low-value, so adding more makes it worse ✅
- (c) Indexation always takes six months, so wait
- (d) Resubmit the sitemap daily until the rest are indexed
> Selective indexing is Google telling you the pages are thin. The fix is stronger unique value and fewer pages, plus deleting the ones with no impressions — not more of the same.

## Go deeper
- [Google Search Central: spam policies on scaled content abuse and doorway pages](https://developers.google.com/search/docs/essentials/spam-policies)
- [Google Search Central: Search Console page indexing report](https://support.google.com/webmasters/answer/7440203)
