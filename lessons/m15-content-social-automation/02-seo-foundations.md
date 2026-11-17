---
id: m15-02-seo-foundations
module: m15-content-social-automation
slug: seo-foundations
title: "SEO Foundations: Crawling, Indexing and Search Intent"
order: 2
duration_min: 15
is_free: false
mission: false
volatility: medium
last_reviewed: 2026-11-16
---

## Key learnings
- Trace a page through discover, crawl, index and rank, and say which stage yours is stuck at
- Classify the intent behind a query before you write a single word for it
- Write for the Hinglish, transliterated and regional queries your users really type
- Tell a keyword with volume apart from a keyword with actual buyers
- Spot a page that was written for a query nobody searches

## Lesson

### The pain

A team in Nashik builds a tiffin-service finder and decides to "do SEO" by publishing twelve articles like *Why Tiffin Services Are the Next Big Thing*. Six weeks later Search Console reports 41 impressions and 2 clicks, one a team member. Meanwhile people in Nashik were typing `tiffin service nashik price per month`, `nashik mein tiffin ka rate` and `नाशिक टिफिन सर्विस` daily, and not one article answered any of them. Worse, three of the twelve were never in Google's index — not ranked low, *absent* — and nobody had checked, because nobody knew those were different states.

### The idea

**Search is not one machine but four stages in a row — discover, crawl, index, rank — each failing silently. Your page is stuck at exactly one of them, and the first job is finding out which.**

Then the rule above it: **ranking is downstream of intent.** A page that does not serve the reason behind a query will not move. Four reasons cover most queries.

| Intent | A query that carries it | What the page must be |
|---|---|---|
| Informational | `monthly tiffin cost nashik` | The number, before any story |
| Navigational | `annapurna tiffin nashik` | Their chosen place; not an SEO win |
| Commercial | `best veg tiffin service nashik` | A list with honest prices |
| Transactional | `tiffin service subscribe nashik` | One step from acting |

In India a fifth cuts across all four: **local intent** — `near me`, or no location at all because the phone knows. Those resolve in a map pack, where your Google Business Profile beats any page. And it is one page per intent, never per keyword.

### The analogy

A junior clerk photographs every notice on every campus board — the crawler. The photos go into a register organised by topic — the index. A student asks for the cheapest thali and he hands over the three most useful notices — ranking. Four different things can go wrong: your notice sits behind a door he never opens (discovery), it is in faded pencil (crawl), he decides it duplicates another and does not file it (indexing), or it is filed but two others answer better (ranking).

**Where the analogy breaks:** the register is rebuilt continuously, the clerk weighs how many other boards point at your notice, and he cannot read a notice that appears only once a projector warms up — what a JavaScript-rendered page looks like to a crawler.

### Picture it

```
                ┌───────────────────────────────┐
  your page ───▶│ 1 DISCOVER   a link or your   │
                │   sitemap told it you exist   │
                └───────────────┬───────────────┘
                                │ a URL
                ┌───────────────▼───────────────┐
                │ 2 CRAWL      a bot fetched it │
                │   and rendered the text       │
                └───────────────┬───────────────┘
                                │ HTML + text
                ┌───────────────▼───────────────┐
                │ 3 INDEX      stored, and      │
                │   judged worth keeping        │
                └───────────────┬───────────────┘
                                │ a candidate
                ┌───────────────▼───────────────┐
                │ 4 RANK       for ONE query,   │
                │   at ONE position             │
                └───────────────────────────────┘
```

A page moves top to bottom through four independent gates, and "not showing up in Google" means it failed at one of them.

### Where it lives

M04 taught you indexes and ranking functions from the inside; this is the same machinery at internet scale, from outside. Chapter 3 fixes discover-crawl-index, chapter 4 scales page-per-intent, chapter 10 turns impressions into a decision. The boundary holds: **M15 ends at the visit**, **M16 begins at it**.

### Words to own

| Term | Plain meaning | Example |
|---|---|---|
| Crawler | The bot that fetches your pages | Googlebot |
| Index | The stored, searchable copy of the web | Your page is "indexed" or not |
| Query | What a human actually typed | `nashik tiffin rate` |
| Keyword | Your shorthand for a group of queries | "tiffin nashik" |
| Search intent | The reason behind the query | Compare prices, not read an essay |
| SERP | The search results page | Ten results plus a map pack |
| Impression | Your result appeared on someone's SERP | 900 impressions, 12 clicks |
| Position | Where you appeared, averaged | Average position 14.3 |
| Long tail | Specific, low-volume, high-intent queries | "veg tiffin college road nashik" |
| Zero-click | The SERP answered it; nobody clicked | "what time does it open" |
| Transliteration | One language typed in another's script | `tution fees kitni hai` |
| Cannibalisation | Your own pages competing for one query | Six near-identical pages |
| Local pack | The map block above the results | Three businesses with ratings |

### Tradeoffs and gotchas

**High volume is usually a trap.** "Online tuition" is contested by companies with budgets you cannot see the top of. "Tuition for 10th standard maths near college road nashik" has no real competition and the person typing it is minutes from paying someone. Want twenty of the second kind.

**Transliterated and regional queries are where the free traffic is**, because most teams write only clean English. `tution fees kitni hai` is Hindi in Latin script — a different string from both `ट्यूशन फीस` and `tuition fees`. Spoken queries run longer still, and in Nashik people search in Marathi, where an English-only page cannot compete. Say the price in the query's own language, in ₹, as a number — and for many of these the winner is a video or Maps listing, not a page.

**SEO pays out in months.** Expect eight to twelve weeks before a new page settles.

**Brand queries and zero-click SERPs both flatter you.** Traffic that is all your own product name measures people who already knew you, and a SERP that answers directly never sends the visit. Own queries whose answer needs your page: a price, a form, a list.

### Verify it

**Red flag: invented numbers.** A language model has no volume data, so "2,400 searches/month" with no source was made up. Demand each number be labelled measured or estimated.

**Red flag: no SERP check.** Type the query yourself first. If no real results exist and autocomplete does not offer it, nobody is asking. If the top ten are comparison lists and you are writing about your founders, you will not rank — and should not.

**Red flag: one page targeting six keywords.** Cannibalisation dressed as efficiency. Name the single intent it serves; cut it if the answer has an "and" in it.

**Red flag: no indexed check.** "Published" is not "indexed". Search `site:` plus your URL, or use URL inspection, before theorising about rank.

**Red flag: English-only keywords for Hindi-speaking users.** If your interviews produced Devanagari or Hinglish queries and the plan has none, your audience was dropped somewhere.

## Walkthrough
1. Open the free Search Console for your site and connect it if you have not. Read the Performance report for the last 28 days and write down total impressions and total clicks.
2. Filter out every query containing your brand name. Whatever remains is your actual non-brand search presence, and for most teams it is close to zero.
3. List the ten highest-impression queries you do have. For each, write the intent in one word: informational, navigational, commercial or transactional.
4. Pick your three most important pages. For each, run the URL inspection tool and record whether Google says it is indexed. If it is not, stop and note that — chapter 3 is about exactly this.
5. Write down fifteen real queries from your chapter 1 user interviews, in the exact words and script the users used, including any Hinglish or Devanagari.
6. Type each of the fifteen into Google on a phone. Screenshot the first screen. Note whether the top result is a web page, a video, a map pack or a direct answer.
7. Classify each of the fifteen by intent, and mark the five with the clearest local or transactional intent. Those are your first targets.
8. For each of those five, write the one sentence of intent the page must serve. If the sentence contains "and", split it into two pages or drop one.
9. Check for cannibalisation: search `site:yourdomain.com` plus one of your target keywords and see whether more than one of your own pages comes back.
10. Record the five target queries, their intents, and one honest note per query on whether a page is even the right format for it. That list is the input to chapters 3 and 4.

## Talk to AI
```prompt
CONTEXT:      I am a student founder. I do not write code; I direct AI.
              My product: [one sentence]. My city and region: [e.g.
              Nashik, Maharashtra]. My users: [who they are, what phone,
              what languages they read and speak].

              Fifteen queries my real users told me they type, in their
              own words and script, including Hinglish and Devanagari:
              [paste all fifteen]

              What Search Console shows for the last 28 days, after
              removing brand queries:
              - impressions: [X]   clicks: [X]
              - top non-brand queries: [paste]
              - pages confirmed indexed: [X of Y]

GOAL:         Turn my fifteen real queries into a shortlist of five
              page-worthy search intents I can realistically win.

REQUIREMENTS: 1. For each of the fifteen, name the intent as exactly one
                 of informational, navigational, commercial or
                 transactional, and say if it also has local intent
              2. Group queries that share ONE intent into ONE target, so
                 I do not build pages that compete with each other
              3. For each group, say whether a web page is the right
                 format at all, or whether a video or a Maps listing
                 wins that SERP instead
              4. Rank the groups by how winnable they are for a brand
                 new site with no links, and explain each ranking
              5. Label every single number you give me as MEASURED (and
                 name the source) or GUESSED. Do not present a guess as
                 data.

CONSTRAINTS:  Free tools only. New domain, almost no backlinks, three
              people. We can publish in English and [other languages].
              Budget 0 rupees.

NON-GOALS:    Do not invent monthly search volumes. Do not write any
              content. Do not give me generic high-volume English
              keywords like "best tiffin service in India". Do not
              discuss technical fixes - that is the next chapter.

ACCEPTANCE:   - Exactly five target intents, each stated in one sentence
                with no "and" in it
              - Every one of my fifteen queries is either assigned to a
                group or explicitly rejected with a reason
              - Every number is labelled MEASURED or GUESSED
              - At least one target comes from a Hinglish or
                regional-language query
              - A section titled "Queries you should not chase" with the
                reason for each

ASK:          First, tell me which of my fifteen queries I will be most
              tempted to target, and why that would be a mistake. Then
              give me the shortlist.
```

## Stuck? Ask AI
```prompt
My page is live but does not appear in Google at all. Do not guess the cause. Instead, walk me through the four stages — discover, crawl, index, rank — one at a time, and for each stage tell me the single free check I can run to prove whether my page passed it. Ask me the result of each check before moving to the next stage.
```

```prompt
Explain search intent to me as if I have never used a search engine, using one query in four different versions. Then give me eight real-sounding queries, including some Hinglish and transliterated ones, and ask me to classify each by intent before you tell me the answers.
```

```prompt
Here are five keywords I plan to target: [paste them]. For each one, tell me honestly whether a brand new site with no backlinks can realistically rank for it, what kind of result currently wins that search, and what a smaller, more specific version of the same query would be. Do not give me search volume numbers.
```

## Quiz

### Q1 · L2
Your page is published and live, but Google's URL inspection says it is not indexed. What does that mean?
- (a) It is ranked very low for every query
- (b) Google has no stored copy of it, so it cannot appear in any result ✅
- (c) It needs more keywords in the heading
- (d) The page loads too slowly to be ranked
> Indexed and published are different states. A page that is not in the index is absent rather than low-ranked, and keyword or content changes cannot help until it gets in.

### Q2 · L3
Someone searches "best veg tiffin service nashik". What intent is that, and what should the page be?
- (a) Transactional — a one-click subscribe page
- (b) Commercial — a comparison with honest prices and differences ✅
- (c) Navigational — your brand's home page
- (d) Informational — an essay about the tiffin industry
> "Best" plus a category plus a city is a comparison query. The searcher is choosing between options, so a page that only sells you does not serve the reason behind the query.

### Q3 · L4
A team publishes six pages: "tiffin nashik", "tiffin service nashik", "nashik tiffin service", "best tiffin nashik", "tiffin in nashik", "nashik tiffin". What is the problem?
- (a) Nothing — more pages means more chances to rank
- (b) All six serve one intent, so they compete with each other and Google picks one almost at random ✅
- (c) The pages will each need their own sitemap entry
- (d) Six pages is too few to establish topical authority
> This is keyword cannibalisation. The rule is one page per intent, not one page per keyword variant — the variants belong on the same page.

### Q4 · L4
An AI gives you a keyword table with "1,900 searches/month" beside each term and no source. What should you do?
- (a) Use it, since models are trained on search data
- (b) Treat every figure as invented until you verify a sample in a real tool ✅
- (c) Divide each figure by ten to be safe
- (d) Ask it to recalculate with more precision
> A language model has no access to live volume data. Unsourced numbers are plausible-sounding fabrication, which is exactly the failure mode M00 warned about — confident, specific and wrong.

### Q5 · L5
Your users are in Nashik and told you they search `nashik mein tiffin ka rate` and `नाशिक टिफिन सर्विस`. Your AI-generated keyword plan contains only clean English terms. What has gone wrong, and what is the cost?
- (a) Nothing — Google translates queries automatically
- (b) The real audience was dropped between interview and brief, losing the least contested traffic you had ✅
- (c) Devanagari queries cannot be targeted by a web page
- (d) Hinglish queries have too little volume to matter
> Transliterated and regional queries are distinct strings with far less competition, and they are where a new site can actually win. A plan that silently drops them has ignored the evidence you paid for by interviewing users.

## Go deeper
- [Google Search Central: how Search works](https://developers.google.com/search/docs/fundamentals/how-search-works)
- [Google Search Console: performance report documentation](https://support.google.com/webmasters/answer/7576553)
