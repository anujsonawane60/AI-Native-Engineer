---
id: m10-11-hybrid-search
module: m10-building-with-ai
slug: hybrid-search-and-reranking
title: "Hybrid Search, Reranking and Citations"
order: 11
duration_min: 25
is_free: false
mission: false
volatility: medium
last_reviewed: 2026-11-16
---

## Key learnings
- Combine keyword and vector search because they fail in opposite ways
- Fuse two result lists by rank, not by incomparable scores
- Rerank a wide candidate set down to the few chunks you can afford
- Require a citation per claim and verify it in code
- Treat a fabricated citation as the worst output your system can produce

## Lesson

### The pain

The team from the last chapter has working vector search and three problems they cannot fix by tuning it.

"SPDT kitne ka hai" returns five chunks about the mess menu in general and nothing about SPDT, even though the term appears in a circular. Searching for order "ORD-20481" returns chunks about orders, plural, in the abstract. And the refund question does retrieve the right circular — at position four of five, where chapter five already warned you attention goes to die, so the model answers from position one instead.

Then the worst one. Their answer about hostel fees ends with "[circular-2026-04]". A mentor opens circular-2026-04. It is about library timings. The citation is real, the document exists, and the claim is not in it. The model cited the first id it had been handed.

The first three problems are search design. The fourth is a trust problem, and it is the more serious of the two.

### The idea

**Keyword search and vector search fail in opposite ways, so run both and fuse the results. Then rerank the combined list with a model that reads the question and the chunk together. Then make every claim carry a citation your code has verified.**

Three separate moves, each independently worth doing, in that order of effort.

### The analogy

The canteen's recipe cards from the last chapter are filed by taste. That is wonderful for "something spicy with gravy" and useless for "the card called SPDT", so the kitchen keeps a second index too: a plain alphabetical list of dish names on the inside of the box lid. Two indexes over one set of cards, each answering the questions the other cannot.

And when five candidate cards come out of the box, the head cook reads all five before deciding which to cook from. That reading step is the reranker, and it is the difference between pulling the right drawer and pulling the right card.

**Where the analogy breaks:** the head cook has actually tasted these dishes, so their judgement draws on something outside the words. A reranker has only the words. A confidently written, beautifully formatted, completely irrelevant card can still win, which is why a citation you can open yourself remains the last line of defence.

### Two kinds of search

**Keyword search** — the classic kind, usually scored with BM25, and available as full-text search in the Postgres you already run. It matches tokens. It is unbeatable on exact strings: codes, ids, surnames, part numbers, "SPDT". It is helpless on paraphrase — ask about "money back" when the document says "refund" and it returns nothing.

**Vector search** — chapter ten's kind. It matches meaning, so "money back" finds "refund" and a clumsily phrased question still lands in the right area. It is helpless on exact strings, because a code carries no meaning to embed.

Neither is better. They are complementary, and almost every real question is a bit of both: "SPDT ka refund milta hai?" has an exact token and a paraphrase in one sentence.

### Fusion: combine ranks, not scores

The naive move is to add the two scores together. Do not. A BM25 score of 14.2 and a cosine similarity of 0.78 are different units on different scales with no shared zero. Adding them produces a number that means nothing and is dominated by whichever happens to be larger.

Combine **positions** instead. Reciprocal rank fusion is the standard simple method: for each chunk, add up one divided by its rank in each list, and sort by the total. A chunk ranked first in both lists beats one ranked first in one and twelfth in the other. It needs no calibration, no weights and no tuning, which is exactly why it is the right thing to start with. Weighted score fusion can beat it, after you have an eval set from chapter twelve to tune against — not before.

### Reranking: read before you choose

Retrieval is fast and shallow. It compares a query vector against chunk vectors that were computed before your question existed, so nothing ever looked at the question and the chunk together.

A **reranker** does exactly that. It takes each question-and-chunk pair and scores how well that chunk answers that question, reading both at once. It is far more accurate and far too slow to run over your whole corpus, which gives the standard shape:

**Retrieve wide, rerank narrow.** Pull forty candidates with hybrid search, rerank those forty, keep the best five for the context block. You get accuracy close to reading everything at the cost of reading forty.

This is usually the biggest single accuracy gain available after fixing ingestion, and it is the move teams reach for last because it sounds advanced. Small rerankers run locally and free.

### Query rewriting

One cheap addition. Real questions are short, mixed-script and full of local shorthand: "aaj ka rate", "SPDT", "wahi jo kal liya tha". Before searching, have a cheap model expand the question into two or three search queries — one literal, one paraphrased, one with acronyms expanded from a glossary you maintain. Search all of them and fuse.

A glossary table mapping "SPDT" to "Special Plate Dal Tadka" is twenty rows of typing and fixes a whole class of failure that no amount of embedding work will touch.

### Picture it

```
  "SPDT ka refund milta hai?"
        │
        ├──▶ rewrite ──▶ 3 queries (literal, paraphrase, expanded)
        │
    ┌───┴────────────────┐
    ▼                    ▼
  KEYWORD             VECTOR         both pre-filtered
  top 20              top 20         on access + language
    │                    │
    └────▶ FUSE by rank ◀┘      1/rank1 + 1/rank2
              │
              ▼  40 candidates
         ┌─────────┐
         │ RERANK  │  reads question + chunk together
         └────┬────┘
              ▼  best 5, each with its chunk id
         context block ──▶ answer with [chunk-id] per claim
              │
              ▼
         your code checks every cited id exists
         and that the quoted span is really in it
```

Wide and cheap first, narrow and expensive last, with a verification step after the answer rather than trust.

### Citations, and verifying them

A citation has one job: let a human check a claim in five seconds. That requires three things your code must enforce, not request.

**Every chunk arrives in the context carrying its id.** `[chunk: fee-2026-04-p3-c2]`. If the model cannot see ids, it will invent plausible ones.

**The output format reserves a place for the citation per claim.** Use chapter three's schema: an array of claims, each with a `text` field and a `source_chunk_id` from an enum of the ids you actually supplied. Now an invented id is not representable.

**Your code verifies before display.** Check the cited id was in the set you sent. Then, for the strongest version, check that a short quoted span from the chunk really appears in that chunk — ask the model to include the sentence it relied on, and confirm it is there with a string comparison. If verification fails, show the answer as unverified or refuse it. This is the step that turns a citation from decoration into a guarantee.

A fabricated citation is worse than no citation at all, because it manufactures the appearance of having been checked. That is the output to be most afraid of in this whole module.

### Where it lives

Keyword search is a full-text index in the Postgres from M04 — the same database holding your vectors, which is why one store for both is so convenient. M04's search-and-ranking chapter is the general theory; this is its application to retrieval for a model.

Reranking is one more model call, so M05's rate limiting and M09's token economics apply, and it sits behind the same `search_docs` read tool from chapter four. Citation verification is plain code: a set membership test and a substring test. Chapter twelve is where you prove any of this helped.

### Words to own

| Term | Plain meaning | Example |
|---|---|---|
| Keyword search | Matching exact tokens | Finds "SPDT" |
| BM25 | The standard keyword scoring method | Postgres full-text |
| Hybrid search | Running keyword and vector, then fusing | Both lists, merged |
| Fusion | Merging two ranked lists | By rank, not score |
| Reciprocal rank fusion | Sum of one-over-rank per list | Needs no tuning |
| Reranker | A model scoring question-and-chunk pairs | 40 in, 5 out |
| Candidate set | The wide list before reranking | Top 40 |
| Query rewriting | Expanding a question into several queries | Acronym expanded |
| Citation | A pointer from a claim to a chunk id | `[fee-2026-04-p3-c2]` |
| Citation verification | Code proving the citation is real | Id in set, span present |

### Tradeoffs and gotchas

**Hybrid search cannot rescue bad ingestion.** If sixty of your documents extracted blank, two search methods will both find nothing in them. Chapter nine comes first for a reason, and teams who skip it spend weeks here.

**Do not add scores from different systems.** It feels like the obvious thing and it quietly destroys your ranking. Rank-based fusion first; weighted fusion only once you have numbers to tune against.

**Reranking costs latency on every query.** Forty pairs is forty comparisons. Measure it on a throttled connection and decide whether the accuracy is worth the wait for your product. For a chat reply, often yes. For a typeahead, no.

**A wider candidate set is not always better.** Reranking two hundred candidates costs five times as much and typically adds very little, because hybrid fusion already put the right chunk in the top forty. Measure where your curve flattens.

**Rerankers can be confidently wrong too.** They read words, not truth. A well-written irrelevant chunk can outrank a terse relevant one. The reranker improves the odds; it does not remove the need for citations.

**Verification must be code, not an instruction.** "Only cite documents that contain the claim" is a request. A set membership test is a guarantee. Same distinction as chapter seven.

**Scores are not comparable across queries.** A top score of 0.9 on one question and 0.4 on another says nothing about which answer is better. Never build a confidence display out of raw similarity.

### Verify it

**Ablate the four configurations on the same twenty questions.** Keyword only, vector only, hybrid, hybrid plus rerank. For each, record how many questions had the correct chunk anywhere in the top five. Four numbers on one line is the most useful measurement in this chapter and it takes an afternoon.

**Measure retrieval separately from the answer.** Chapter ten's habit, still the most important one. Recall at five — did the right chunk make it into the context at all — is the number to improve first, because nothing downstream can recover from a miss.

**Open three citations and read them.** Not skim: find the sentence. Do this on every demo, every time. It is the check that catches the fabricated-citation failure, and nothing automatic will convince you as fast as finding one yourself.

**Turn on verification and count the failures.** The proportion of answers whose citation does not verify is a quality metric. If it is zero, confirm your verification is actually running by deliberately corrupting a chunk id.

**Test one exact-match query and one paraphrase query at every stage.** The exact one proves keyword search is wired in. The paraphrase proves vector search survived the fusion. Many teams ship hybrid search where one of the two legs is silently broken and the other covers for it.

Red flags: a citation that points at a real document not containing the claim; citations never opened by anyone; two search scores added together; no keyword search at all in a product full of codes and names; reranking over the whole corpus or over hundreds of candidates; verification implemented as a sentence in the prompt; a confidence percentage derived from a similarity score; and no recall measurement, so improvements are a matter of opinion.

## Walkthrough
1. Take the twenty test questions from chapter ten, each with the chunk id that actually contains the answer. Without this ground truth, nothing in this chapter is measurable.
2. Add full-text keyword search over the same chunk table in the database you are already using. Run all twenty questions and record recall at five.
3. Run the same twenty through your existing vector search and record recall at five. Note which questions each method got and the other missed.
4. Implement reciprocal rank fusion: take the top twenty from each, add one over each rank, sort, keep the top forty. Record recall at five again.
5. Compare the three numbers. The questions that only hybrid solves are almost always a code, an id or a rare name on one side and a paraphrase on the other.
6. Build a glossary table of your local acronyms and shorthand, then add a cheap query-rewriting step producing a literal, a paraphrased and an expanded query. Re-measure.
7. Add a reranker over the forty candidates, keep the best five, and record recall at five one final time. Write all four numbers in your decision log.
8. Measure added latency for the rerank step on a throttled connection, and decide explicitly whether your product can afford it.
9. Make every chunk arrive in the context with its id visible, and change your output schema so each claim has a `source_chunk_id` restricted to the ids you supplied.
10. Write the verification code: the cited id must be in the set you sent, and the model's quoted sentence must appear in that chunk as a substring. Fail closed when either check fails.
11. Deliberately break it — corrupt one chunk id, or ask a question your corpus cannot answer — and confirm verification catches it rather than passing silently.

## Talk to AI
```prompt
Help me upgrade a working vector search into hybrid search with reranking and verified citations, and make me measure each step.

MY SETUP: [vector store and embedding model, and the database I already run]
MY CHUNK TABLE: [columns, including metadata fields and the chunk id format]
MY TWENTY TEST QUESTIONS, each with the chunk id that truly contains the answer:
1. [question - correct chunk id]
2. [...]
WHAT FAILS TODAY: [be specific - which questions, and how]
MY CONSTRAINTS: free tiers, slow 4G users, budget Android phones.

Deliver all of this:

1. Sort my twenty questions into exact-match questions, paraphrase questions, and mixed. Predict which search method handles each and which currently fails.
2. How to add keyword search over my existing chunk table without a new service, and what to index.
3. The fusion method, written out as the exact arithmetic over two ranked lists, and why rank-based fusion rather than adding scores.
4. The retrieve-wide-rerank-narrow design: how many candidates, how many survive, and the experiment to find where the benefit flattens for MY corpus.
5. A query-rewriting step: how many rewrites, what each is for, and the glossary table I should build from my own local shorthand. Give me the first ten rows based on what you can infer about my domain.
6. The citation contract: how chunk ids reach the context, the output schema with source ids as an enum, and the exact verification my code must perform. Write the verification as plain pseudocode.
7. What my code should do when verification fails, and why failing closed is right.
8. A measurement table I should fill in: recall at five for keyword only, vector only, hybrid, hybrid plus rerank - plus latency for each.
9. The one thing in this plan most likely to be silently broken in a way that still looks like it works, and the test that would expose it.
```

## Stuck? Ask AI
```prompt
Explain why keyword search and vector search fail in opposite ways, with two concrete example queries where each one wins and the other returns nothing useful. Then explain reciprocal rank fusion to me in plain arithmetic, and why adding a BM25 score to a cosine similarity is wrong.
```

```prompt
Explain what a reranker does differently from the retrieval step, and why it cannot simply replace retrieval. Use the idea that retrieval compares precomputed vectors while a reranker reads the question and the chunk together. Then tell me how many candidates I should rerank and how I would find the right number for my own data.
```

```prompt
My AI answer cited a real document that does not contain the claim. Explain how that happens mechanically. Then design the strongest citation guarantee I can build with code rather than instructions - what the context must carry, what the output schema must look like, and exactly what my code should check before showing anything.
```

## Quiz

### Q1 · L3
Why combine two search result lists by rank rather than by adding their scores?
- (a) Rank fusion is faster to compute
- (b) BM25 and cosine scores are different units with no shared scale ✅
- (c) Scores are not returned by most stores
- (d) Rank fusion gives keyword search more weight
> Adding 14.2 to 0.78 produces a meaningless number dominated by whichever scale is larger. Positions are comparable across systems and need no calibration.

### Q2 · L3
What does a reranker do that retrieval cannot?
- (a) Search a larger corpus
- (b) Read the question and the chunk together and score the pair ✅
- (c) Apply permission filters
- (d) Embed the query more accurately
> Retrieval compares vectors computed before the question existed. A reranker evaluates the actual pair, which is far more accurate and far too slow to run over everything.

### Q3 · L4
Your product is full of codes and order ids and you only have vector search. What is the cheapest high-value fix?
- (a) A larger embedding model
- (b) Smaller chunks with more overlap
- (c) Add keyword search and fuse the two result lists ✅
- (d) Increase top-k to twenty
> Codes carry no meaning to embed, so no embedding work finds them. Keyword search matches exact tokens, and fusion keeps the paraphrase strength you already had.

### Q4 · L5
An answer cites a real document that does not contain the claim. Which fix is a guarantee rather than a request?
- (a) Instructing the model to cite only documents containing the claim
- (b) Restricting the citation field to supplied ids and checking the quoted span appears in that chunk ✅
- (c) Showing the similarity score beside each citation
- (d) Using a reasoning model for the final answer
> A schema enum makes an invented id unrepresentable, and a substring check proves the sentence is really there. Instructions are requests; code is a guarantee.

### Q5 · L4
Which single number should you improve first when RAG answers are wrong?
- (a) The similarity score of the top chunk
- (b) Whether the correct chunk reaches the context at all ✅
- (c) The length of the generated answer
- (d) Average response latency
> Nothing downstream can recover from a retrieval miss. Recall at k measures the only thing generation cannot fix, which is why retrieval is measured separately.

## Go deeper
- [Microsoft: hybrid search and reciprocal rank fusion explained](https://learn.microsoft.com/en-us/azure/search/hybrid-search-ranking)
- [Anthropic: Contextual retrieval, including reranking results](https://www.anthropic.com/news/contextual-retrieval)
