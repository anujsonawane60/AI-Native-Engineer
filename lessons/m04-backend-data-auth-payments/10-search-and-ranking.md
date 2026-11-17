---
id: m04-10-search-and-ranking
module: m04-backend-data-auth-payments
slug: search-and-ranking
title: "Search and Ranking: Full-Text, Fuzzy and Relevance"
order: 10
duration_min: 15
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

A student opens the canteen app at 1:10 p.m., hungry, on one bar of 4G, and types `panir`. Zero results — the dish is spelled "Paneer Butter Masala" and the search box matches only the start of the name. The student concludes the canteen does not sell paneer and closes the app.

A second attempt matches anywhere in the name. Now `rice` returns eleven dishes in whatever order the database found them, with the canteen's most popular dish ninth. Nobody ever wrote down what a good result for `panir` looks like — that, not the technology, is why both attempts failed.

### The idea

**Search is two separate problems — finding candidates, and ordering them — and the ordering is a product decision your team must make explicitly.**

Finding has three levels. **Exact and partial matching** looks for your letters inside a column. **Full-text search** understands words: it tokenises and lowercases the text, stems each word to a root so `fries` matches `fried`, and drops stop words like `and` and `the`. The query runs through the same pipeline, which is why `butter masala paneer` finds the dish in any word order. **Fuzzy search** tolerates misspelling by measuring edit distance, the number of single-character changes between two strings, so `panir` finds `paneer` one edit away.

Ordering is **ranking**, and the database cannot guess it. Does availability beat relevance? Does popularity count? If nobody on the team decides, the answer becomes "whatever order the rows came out in".

### The analogy

A new student asks the cashier for "something with paneer". She works out what you meant, forgiving the pronunciation; she lists what fits; then she orders that list by what she judges useful — available now first, then popular. The third step is the valuable one, and it is entirely judgement.

**Where the analogy breaks:** she knows a student asking at 1:10 p.m. wants something fast, and that "paneer" from a hostel student usually means the cheap roll rather than the ₹160 platter. That context your search box cannot infer, and everything she knows implicitly you must write down as a rule and maintain.

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

Search is a query, so it lives where chapter 9 lived: between your server and your database. It takes chapter 9's ordering too — **make the database do it before you add a service.** Postgres ships full-text search and fuzzy matching on a free tier, both indexable and both consistent with your data because they *are* your data. M05 caches popular queries; M10's retrieval for RAG is this same two-step with embeddings instead of words.

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

**Indian search queries are not English search queries.** Users type Hinglish, transliterate inconsistently (`chole`, `chhole`, `cholay`), abbreviate (`vdp` for vada pav), and type while walking. You will not catch this in testing, because you know the right spellings.

**Fuzzy matching is a dial, not a switch.** Loose thresholds return nonsense — `rice` starts matching `ice` and `nice` — and tight ones miss real typos. Only a judged query list can set it.

**Hiding sold-out items is usually the wrong call.** A student searching `paneer` wants to know the roll is finished, not to conclude it does not exist. Rank it low and label it.

**A separate search service is a sync problem in disguise.** Every write to your database must reach it, and one missed write serves dishes you deleted yesterday.

### Verify it

The one practice that separates working search from a demo: **a judged query list.** Write twenty queries your users would really type — three misspellings, two Hinglish words, one single letter, one with a trailing space, one for something you do not sell — and beside each, the result that should rank first and why. That list is your test suite, your brief to AI, and your only honest answer to "is our search good?"

1. **Run the twenty and count passes,** before and after every change. A change that fixes one query and breaks three is visible only this way.
2. **Test the empty and absurd cases:** empty string, one letter, a 200-character paste, only stop words, something you do not sell.
3. **Ask what the ranking rule is, in words.** If AI cannot state the ordering as a sentence, there is no rule.
4. **Check it against realistic volume.** Seed a few thousand rows and confirm an index is used.
5. **Check whether unavailable items are hidden or ranked low,** and that it matches what your team decided.

Red flags: nothing returned for a one-character typo; results that reorder between two identical requests; a search service proposed before the database's own full-text search was tried; "relevance" with no explanation of what feeds it; a `LIKE '%query%'` on a large table with no index, which is chapter 9's sequential scan wearing a search box; any search feature called done without a list of test queries.

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
