---
id: m04-10-search-and-ranking
module: m04-backend-data-auth-payments
slug: search-and-ranking
title: "Search and Ranking: Full-Text, Fuzzy and Relevance"
order: 10
duration_min: 25
is_free: false
mission: false
volatility: low
last_reviewed: 2026-11-16
---

## Key learnings
- Tell the difference between matching, full-text search and fuzzy search
- Say why ranking is a product decision and who should make it
- Handle the Indian reality of transliteration, Hinglish and typed-in-a-hurry queries
- Start with what your database already does before adding a search service
- Verify search quality with a judged query list instead of a vibe check

## Lesson

### The pain

A student opens the canteen app at 1:10 p.m., hungry, on one bar of 4G, and types `panir`.

Zero results. The dish is spelled "Paneer Butter Masala". The search box does an exact match on the start of the name, so `panir` matches nothing, `butter` matches nothing, and `paneer butter` matches because it happens to be a prefix. The student concludes the canteen does not sell paneer and closes the app.

The team's second attempt searches anywhere in the name, case-insensitively. Now `rice` returns eleven dishes with fried rice at the bottom and "Curd Rice (small)" at the top, because the results come back in whatever order the database found them. The most popular dish in the canteen is the ninth result.

The third attempt installs a dedicated search engine on a free tier, which takes two days, adds a service that must be kept in sync, and returns results in an order nobody chose either — just a different arbitrary order, now with a syncing bug where dishes deleted yesterday still appear.

Nobody ever wrote down what a good result for `panir` looks like. That, not the technology, is why all three attempts failed.

### The idea

**Search is two separate problems — finding candidates, and ordering them — and the ordering is a product decision your team must make explicitly.**

Finding has three levels of sophistication. **Exact and partial matching** looks for your letters inside a column. **Full-text search** understands words: it splits text into terms, reduces them to stems so "fried" matches "fries", ignores noise words, and can search several columns at once. **Fuzzy search** tolerates misspelling by measuring how different two strings are, so `panir` finds `paneer`.

Ordering is **ranking**, and the database cannot guess it. How much does a match in the name matter versus the description? Does availability beat relevance — should a sold-out exact match rank below an available near-match? Does popularity count? Does price? Those are questions for the whole founding team, and if nobody answers them the answer becomes "whatever order the rows came out in".

### The analogy

A new student asks the canteen cashier for "something with paneer".

The cashier does three things in sequence without thinking about it. She works out what you meant, forgiving the pronunciation. She lists what exists that fits. Then she orders the list by what she judges useful: what is actually available right now first, then what is popular, then the rest — and she volunteers "the paneer roll is finished, but the butter masala is fresh".

The third step is the valuable one, and it is entirely judgement. A new cashier who lists dishes in the order they appear on the laminated menu is technically correct and unhelpful.

**Where the analogy breaks:** the cashier knows that a student asking at 1:10 p.m. wants something fast, and that "paneer" from a hostel student usually means the cheap roll rather than the ₹160 platter. She has context your search box does not have and cannot infer. Everything she knows implicitly, you have to write down as a rule and maintain.

### Picture it

```
  query: "panir"
      │
      ▼
 ┌──────────────────┐
 │ 1. NORMALISE     │  lowercase, trim, strip accents
 └────────┬─────────┘
          ▼
 ┌──────────────────┐
 │ 2. FIND          │  exact ──▶ 0 hits
 │                  │  full-text ──▶ 0 hits ("panir" is
 │                  │                not a word stem)
 │                  │  fuzzy ──▶ "Paneer Butter Masala"
 │                  │            "Paneer Roll"
 └────────┬─────────┘
          ▼
 ┌──────────────────┐
 │ 3. RANK          │  available first, then name match
 │                  │  over description, then popularity
 └────────┬─────────┘
          ▼
   Paneer Roll (available, popular)
   Paneer Butter Masala (available)
   Paneer Platter (sold out, shown greyed)
```

A query is normalised, then matched by one or more finding strategies, and only then ordered by rules your team chose — with sold-out items shown rather than hidden.

### Where it lives

Search is a query, so it lives exactly where chapter 9 lived: between your server and your database. And the ordering is the same as chapter 9's ordering — **make the database do it before you add a service.**

Modern Postgres ships full-text search and fuzzy matching, both usable from a free tier, both indexable, both transactionally consistent with your data because they *are* your data. A separate search service means a second system, a sync process, a sync bug, and results that can disagree with your database. For a three-person team, that is the wrong trade until you have a measured reason.

Two forward links matter. In M05 you will cache popular queries, and ranking is exactly the kind of computation worth caching. In M10, retrieval for RAG is this chapter's problem again with embeddings instead of words — "find candidates, then rank them" is the same two-step, which is why vector search and full-text search are increasingly used together.

### What full-text search actually does

Four steps happen to your text before it is stored, and the same four happen to the query.

**Tokenise.** "Paneer Butter Masala (small)" becomes `paneer`, `butter`, `masala`, `small`.

**Normalise.** Lowercase, strip punctuation and accents.

**Stem.** Reduce words to a root so `fries`, `fried` and `frying` all match. This is why full-text search finds "fried rice" when you type "fries" and exact matching never will.

**Remove stop words.** `and`, `the`, `with` carry almost no signal and are dropped.

The result is stored in a searchable index, and a query runs through the same pipeline so the two can be compared. This is what makes `butter masala paneer` find the dish despite the wrong word order — something no `LIKE '%...%'` match can do.

What it will not do is fix spelling. `panir` is not a stem of `paneer`; it is a different word. That needs fuzzy matching, which measures the edit distance between strings — how many single-character changes turn one into the other. `panir` to `paneer` is one insertion, so a threshold of two edits finds it. Set the threshold too loose and `rice` starts matching `ice` and `nice`.

The practical answer for an Indian product is usually both: full-text search for real words, fuzzy matching as a fallback when full-text returns nothing.

### Words to own

| Term | Plain meaning | Example |
|---|---|---|
| Exact / partial match | Looking for your letters inside a column | `LIKE '%rice%'` |
| Full-text search | Word-aware search with stemming | finds "fried" for "fries" |
| Token | One word after text is split up | `paneer` |
| Stemming | Reducing a word to its root | `frying` to `fri` |
| Stop word | A common word carrying no signal | `the`, `with` |
| Fuzzy match | Tolerating misspelling by edit distance | `panir` finds `paneer` |
| Edit distance | How many character changes between strings | 1 |
| Relevance score | A number saying how well a row matched | 0.84 |
| Ranking | The rule that orders the results | available first |
| Transliteration | Writing one language in another's script | `chawal` for चावल |

### Tradeoffs and gotchas

**Indian search queries are not English search queries.** Your users type Hinglish, transliterate inconsistently (`chole`, `chhole`, `cholay`), abbreviate (`vdp` for vada pav), use regional names for the same dish, and type on a phone keyboard while walking. A search that only handles correctly spelled English names will fail for a large share of real queries, and you will not see this in testing because you know the right spellings.

**Fuzzy matching is a dial, not a switch.** Loose thresholds return nonsense and users stop trusting the box; tight thresholds miss real typos. The only way to set it is against a list of real queries with judged answers, which is the next section.

**Hiding sold-out items is usually the wrong call.** A student searching `paneer` wants to know the paneer roll is finished, not to conclude it does not exist. Rank it low, show it greyed, say "finished today". This is a product decision and it belongs in your ranking rules.

**Ranking without a written rule is still ranking.** It is ranking by whatever order the database returned, which is arbitrary and will change. Write your rules down, in priority order, even if there are only three of them.

**A separate search service is a sync problem in disguise.** Every write to your database must reach it. Miss one and you serve deleted dishes, as the team in the opening story did. Postgres full-text search cannot drift from your data because it is not a copy.

**Searching with no limit and no debounce will hurt.** A search-as-you-type box that fires a query per keystroke, each returning every match, is four requests per word on a 4G connection. Debounce the input and bound the results.

### Verify it

The one practice that separates working search from a demo: **a judged query list.**

Write down twenty real queries your users would actually type — including three misspellings, two Hinglish words, one single letter, one with a trailing space, and one for something that does not exist. Beside each, write what the top result should be and why. Then run all twenty and mark pass or fail. That list is your test suite, your brief to AI, and the only honest answer to "is our search good?"

Then five checks on anything AI builds.

1. **Run the twenty queries and count passes.** Before and after any change. A change that improves one query and breaks three is visible only this way.
2. **Test the empty and absurd cases.** Empty string, one letter, a 200-character paste, a query with only stop words, a query for something you do not sell. Each must return something sensible rather than an error or everything.
3. **Ask what the ranking rule is, in words.** If AI cannot state the ordering as a sentence, there is no rule and you are seeing arbitrary order.
4. **Check it against realistic volume.** Search on 40 dishes is always fast. Seed a few thousand rows and confirm an index is being used, using chapter 9's plan reading.
5. **Check whether unavailable items are hidden or ranked low,** and confirm that matches the decision your team actually made.

Red flags: a search that returns nothing for a one-character typo; results that change order between two identical requests; a dedicated search service proposed before the database's own full-text search was tried; "relevance" with no explanation of what contributes to it; a `LIKE '%query%'` on a large table with no index, which is chapter 9's sequential scan wearing a search box; and any search feature declared done without a list of test queries.

## Walkthrough
1. Sit with a teammate and write twenty queries a real user of your product would type. Make three of them misspelled, two Hinglish or transliterated, one a single letter, one with a trailing space, and one for something you do not offer.
2. Beside each query, write the result that should be first, and one line on why. Disagreements here are product decisions; settle them now and write down what you decided.
3. Run all twenty against whatever search you have today. Mark pass or fail. Record the count. This is your baseline.
4. Write your ranking rules in priority order as plain sentences. Three to five rules. For example: available before unavailable, name match before description match, more popular before less popular.
5. Look up what your database offers natively for full-text and fuzzy search before considering any separate service. Note the feature names you find.
6. Brief AI with your twenty queries, your expected top results and your ranking rules, and ask it to implement search using only your database's built-in capabilities.
7. Re-run the twenty queries. Mark pass or fail again and compare to your baseline. Note any query that got worse — there is usually one.
8. Test the ugly inputs: empty string, one letter, a long paste, only stop words, punctuation only. Record what each returns.
9. Seed a few thousand rows and re-run your slowest query with `EXPLAIN ANALYZE` from chapter 9. Confirm an index is used rather than a sequential scan.
10. Check on a real phone on a throttled connection. Count the requests your search box fires while you type one word, and fix it if it is one per keystroke.
11. Record in your decision log: your ranking rules, your pass count before and after, and the one query you deliberately chose not to support.

## Talk to AI
```prompt
CONTEXT:      Three-person student team, free-tier managed PostgreSQL.
              Users are in India on budget Android phones and slow 4G; they
              type Hinglish, transliterate inconsistently, misspell, and
              often type two or three letters only.
              Product: [one sentence]
              What is searchable: [table and columns, e.g. menu_items.name,
              menu_items.description, with row counts now and expected]
              Availability is tracked by: [column name, e.g. is_available]

              MY JUDGED QUERY LIST — query, then the result that should
              rank first, and why:
              [paste all twenty]

              MY RANKING RULES, IN PRIORITY ORDER:
              [paste your 3-5 plain sentences]

GOAL:         Search that passes as many of my twenty judged queries as
              possible, using only what my database already provides.

REQUIREMENTS: 1. Use PostgreSQL's own full-text search and fuzzy matching.
                 Do not introduce any separate search service
              2. Describe the pipeline in plain language first: how a query
                 is normalised, how candidates are found, and how they are
                 ordered according to MY ranking rules
              3. Handle the case where full-text returns nothing by falling
                 back to fuzzy matching, and tell me the edit-distance
                 threshold you chose and why
              4. Tell me which of my twenty queries your design will still
                 fail, and why, before you write anything
              5. Specify the behaviour for empty input, a single character,
                 a very long paste, and a query that matches nothing
              6. Name the index required and confirm with chapter-9-style
                 reasoning that it will be used at my expected row count
              7. Say explicitly how unavailable items are treated, matching
                 my ranking rules rather than your own preference

CONSTRAINTS:  Free-tier PostgreSQL only. Results must be bounded — tell me
              the limit you chose. The payload must stay small enough for
              slow 4G. No external search service, no paid API, no
              background sync process.

NON-GOALS:    No autocomplete yet. No search analytics. No personalisation.
              No semantic or vector search. Do not change my schema beyond
              adding what search itself requires.

ACCEPTANCE:   - Running my twenty queries, I can mark pass or fail on each
              - You predicted which ones would fail before I ran them
              - The ranking order matches my written rules, not an
                arbitrary database order
              - Unavailable items are handled the way my rules state
              - Empty and single-character queries return something
                sensible rather than an error or everything

ASK:          Before any implementation, tell me which of my twenty queries
              are hard and why, and which of my ranking rules conflict with
              each other. I want to resolve the product questions first.
```

## Stuck? Ask AI
```prompt
Explain the difference between exact matching, full-text search and fuzzy matching using a canteen menu as the example. For each one, give me a query it handles well and a query it fails on, and say which of the three would find "Paneer Butter Masala" when I type "panir".
```

```prompt
I want to set a fuzzy matching threshold for an Indian food app. Explain edit distance in plain language, then give me five pairs of words — some that should match and some that should not — and ask me what threshold I would pick, without telling me the answer first.
```

```prompt
My search returns eleven results in an order nobody chose. Help me write ranking rules as plain sentences, in priority order, by asking me questions about my product. Ask them one at a time and do not write any code.
```

## Quiz

### Q1 · L2
What does stemming do in full-text search?
- (a) Removes misspellings from the query
- (b) Reduces words to a root so "fried", "fries" and "frying" all match ✅
- (c) Sorts results by relevance score
- (d) Splits text into separate columns
> Stemming is why full-text search finds "fried rice" when you type "fries". It is word-aware, which partial matching is not — and it still cannot fix a typo.

### Q2 · L3
A student types `panir` and gets zero results for "Paneer Butter Masala". Which technique fixes this specific failure?
- (a) Full-text search with stemming
- (b) Removing stop words from the query
- (c) Fuzzy matching on edit distance ✅
- (d) An index on the name column
> `panir` is not a stem of `paneer`, it is a misspelling one character away. Edit-distance matching handles it; stemming and indexing do not.

### Q3 · L4
Your search returns eleven dishes for "rice" with the most popular one ninth. Whose problem is this?
- (a) The database's, for returning rows in the wrong order
- (b) The founding team's — nobody wrote down the ranking rules ✅
- (c) The index's, which needs rebuilding
- (d) The client's, which should sort the results itself
> With no written ordering rule, results come back in whatever order the database found them. Ranking is a product decision, and "arbitrary" is what you get by default.

### Q4 · L4
Why is a dedicated search service a poor first choice for a three-person team?
- (a) Search services cannot do fuzzy matching
- (b) It adds a second system that must be kept in sync, and sync bugs serve deleted data ✅
- (c) PostgreSQL full-text search is faster at every scale
- (d) Free tiers do not offer search services
> Postgres full-text search cannot drift from your data because it is your data. A separate index is a copy, and every missed write shows students dishes that no longer exist.

### Q5 · L5
What is the single most useful thing to produce before building search?
- (a) An index on every text column
- (b) A list of twenty real queries with the result that should rank first for each ✅
- (c) A choice between full-text and fuzzy matching
- (d) A relevance scoring formula
> It is simultaneously your brief, your test suite and your only honest answer to "is our search good?". Without it, every change is judged by vibe and a fix for one query silently breaks three.

## Go deeper
- [PostgreSQL docs: full text search](https://www.postgresql.org/docs/current/textsearch.html)
- [PostgreSQL docs: pg_trgm for fuzzy string matching](https://www.postgresql.org/docs/current/pgtrgm.html)
