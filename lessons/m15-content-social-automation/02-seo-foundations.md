---
id: m15-02-seo-foundations
module: m15-content-social-automation
slug: seo-foundations
title: "SEO Foundations: Crawling, Indexing and Search Intent"
order: 2
duration_min: 25
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

A team in Nashik builds a tiffin-service finder. It works. They decide to "do SEO" and publish twelve articles over four weeks: *The Future of Food Tech in India*, *Why Tiffin Services Are the Next Big Thing*, *10 Productivity Tips for Students*. Six weeks later, Search Console reports 41 total impressions and 2 clicks, one of which was a team member.

Meanwhile people in Nashik were typing, every day: `tiffin service near me`, `tiffin service nashik price per month`, `nashik mein tiffin ka rate`, `नाशिक टिफिन सर्विस`, and `veg tiffin college road nashik`. Not one of the twelve articles answered any of them. The team had written for an imaginary reader who enjoys essays about industries, instead of a real one standing outside a hostel gate at 8:40 pm wanting a monthly price.

Worse, three of the twelve were never in Google's index at all — not ranked low, *absent*. Nobody had checked, because nobody knew there was a difference between "published" and "indexed".

### The idea

**Search is not one machine, it is four stages in a row — discover, crawl, index, rank — and each one fails silently. Your page is stuck at exactly one of them, and the first job is to find out which.**

Then one more rule that decides everything above it: **ranking is downstream of intent.** A query carries a reason. If your page does not serve that reason, no amount of writing moves it, because the engine is matching the reason, not the words.

### The analogy

The campus noticeboard system. Every week a junior clerk walks the whole campus photographing every notice on every board — that is the crawler. The photos go into a register in the office, organised by topic — that is the index. When a student walks up and asks "where is the cheapest thali today?", the clerk hands over the three most useful notices — that is ranking.

Four things can go wrong, and they are completely different problems. Your notice is pinned behind a door the clerk never opens: a discovery failure. The clerk walks past but your notice is written in faded pencil he cannot read: a crawl failure. He reads it, decides it is a duplicate of another notice, and does not file it: an indexing failure. It is filed, but two other notices answer the thali question better: a ranking failure.

**Where the analogy breaks** in three ways. The register is rebuilt continuously, not weekly. The clerk weighs hundreds of signals, including how many other boards point at your notice. And most importantly, the clerk cannot read a notice that only appears after a projector warms up — which is exactly what a page that renders entirely in JavaScript looks like to a crawler. M02's rendering chapter is why that matters here.

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

A page moves top to bottom through four independent gates, and "not showing up in Google" means it failed at one specific gate — so the fix for a discovery failure has nothing in common with the fix for a ranking failure.

### Search intent, and why it decides the page

Every query has a reason behind it. There are four common ones, and they want four different pages.

**Informational** — "how much does a monthly tiffin cost in nashik". They want an answer. Give them the number, high on the page, before any story.

**Navigational** — "annapurna tiffin nashik". They already decided; they want a specific place. If that is your brand, this is not an SEO win — they were coming anyway.

**Commercial** — "best veg tiffin service nashik". They are comparing. They want a list with prices and honest differences, not a sales page about you.

**Transactional** — "tiffin service subscribe nashik". They want to act now. The page should be one step from doing it.

And in India, a fifth cuts across all of them: **local intent**. `near me`, a locality name, or no location at all because the phone already knows. Those queries often resolve in a map pack, which means your Google Business Profile can matter more than any page you write.

The practical rule: one page per intent, not one page per keyword. Six near-identical pages chasing six variants of the same intent compete with each other, and Google picks one almost at random. That is keyword cannibalisation, and it is self-inflicted.

### The Indian query, taken seriously

This is where the free traffic is, because most teams write only clean English.

Your users type `tution fees kitni hai` — Hindi words in Latin script. That is **transliteration**, and it is a different string from both `ट्यूशन फीस` and `tuition fees`. They type **Hinglish**: `best tiffin service nashik under 2000 rupees mahina`. They *speak* queries, which makes them longer and conversational: "nashik mein sabse sasta veg tiffin kaun deta hai". They search in Marathi in Nashik, and a page that exists only in English simply does not compete.

Three things follow. Say the price in the query's own language, in ₹, as a number. Keep a natural Hinglish phrasing on the page where your users genuinely use it — forcing it where they do not is worse than leaving it out. And remember that for many of these queries the winning result is a YouTube video or a Maps listing, not a web page, so chapter 9's repurposing is part of your SEO, not separate from it.

### Where it lives

M04 taught you what an index and a ranking function are from the inside — inverted indexes, relevance scoring, why a query becomes a set of candidates and then an order. This chapter is the same machinery at internet scale, operated from outside.

Chapter 3 fixes the discover-crawl-index stages mechanically. Chapter 4 scales the page-per-intent idea to hundreds of pages. Chapter 10 is where impressions and positions become a decision. And the M16 boundary holds: search traffic arriving is this module; what those arrivals do next is not.

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

**High volume is usually a trap.** "Online tuition" has enormous volume and is contested by companies with budgets you cannot see the top of. "Tuition for 10th standard maths near college road nashik" has tiny volume, almost no competition, and the person typing it is three minutes from paying someone. Early on, you want the second kind — twenty of them.

**SEO pays out in months, not days.** Expect eight to twelve weeks before a new page settles. This is the clearest reason chapter 1 insists you start in week four, not week eleven.

**Brand queries are not an SEO result.** If your Search Console traffic is all your own product name, you are measuring people who already knew you. Filter brand queries out before you judge anything.

**Zero-click is real.** For many informational queries the SERP answers it directly and your impression never becomes a visit. Your defence is to also own queries where the answer requires your page — a price, a form, a live list.

**Do not write for crawlers.** Keyword-stuffed text reads badly to humans and is detected trivially. The machine is now good enough that writing well for the person is also writing well for the engine.

**Not every query should be a page.** Some belong in a video, some in your Maps listing, some in nothing at all. Deciding "we will not chase this" is a real answer.

### Verify it

AI will cheerfully hand you a keyword plan with a table of monthly search volumes. Treat that table as fiction until proven otherwise.

**Red flag: invented numbers.** A language model does not have Google's volume data. If it produces "2,400 searches/month" without naming where the figure came from, it made it up. Ask it to label every number as measured or estimated, and go and check a sample in a real free tool.

**Red flag: a page for a query nobody searches.** The clearest signal is Search Console showing a handful of impressions after eight weeks. Before you write, type the query yourself and look at whether real results exist and whether autocomplete offers it. If nothing competes, often nobody is asking.

**Red flag: an intent mismatch.** Open the actual SERP for your target query. If the top ten are all comparison lists and you are writing a story about your founders, you will not rank — and should not.

**Red flag: one page targeting six keywords.** That is cannibalisation dressed as efficiency. Ask which single intent the page serves, in one sentence, and cut it if the answer has an "and" in it.

**Red flag: no indexed check.** "Published" is not "indexed". Search `site:` plus your URL, or use the URL inspection tool, before you theorise about rankings.

**Red flag: English-only keywords for Hindi-speaking users.** If your interviews from chapter 1 produced Devanagari or Hinglish queries and the plan has none, your real audience was dropped somewhere between the interview and the brief.

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
