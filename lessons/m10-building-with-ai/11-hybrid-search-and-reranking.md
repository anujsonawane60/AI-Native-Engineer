---
id: m10-11-hybrid-search
module: m10-building-with-ai
slug: hybrid-search-and-reranking
title: "Hybrid Search, Reranking and Citations"
order: 11
duration_min: 15
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

The team has working vector search and three problems tuning cannot fix. "SPDT kitne ka hai" returns chunks about the mess menu in general, though the term is in a circular. Order "ORD-20481" returns chunks about orders in the abstract. And the refund question retrieves the right circular at position four of five, where attention goes to die.

Then the worst one. Their answer on hostel fees ends with "[circular-2026-04]". A mentor opens it: library timings. The citation is real, the document exists, the claim is not in it — the model cited the first id it was handed. The first three are search design; the fourth is trust.

### The idea

**Keyword search and vector search fail in opposite ways, so run both and fuse the results. Then rerank the combined list with a model that reads the question and the chunk together. Then make every claim carry a citation your code has verified.**

Three separate moves, each worth doing alone, in that order.

### The analogy

The recipe cards are filed by taste, wonderful for "something spicy with gravy" and useless for "the card called SPDT" — so the kitchen keeps a second index inside the box lid: an alphabetical list of dish names. Two indexes over one set of cards, each answering what the other cannot. And when five candidates come out, the head cook reads all five: that is the reranker.

**Where the analogy breaks:** the head cook has tasted these dishes, so their judgement draws on something outside the words. A reranker has only words, so a beautifully formatted irrelevant card can still win — which is why a citation you can open yourself is the last line of defence.

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

Wide and cheap first, narrow and expensive last, with verification after the answer rather than trust.

**Keyword search** matches tokens: unbeatable on exact strings — codes, ids, surnames, "SPDT" — helpless on paraphrase. **Vector search** matches meaning, so "money back" finds "refund", and is helpless on exact strings. Every real question is both: "SPDT ka refund milta hai?"

**Fuse by combining ranks, not scores.** A BM25 score of 14.2 and a cosine similarity of 0.78 are different units with no shared zero, so adding them yields a number dominated by whichever is larger. Reciprocal rank fusion adds one divided by each chunk's rank in each list, needing no weights. Then **retrieve wide, rerank narrow**: forty candidates in, best five out. Add query rewriting and a glossary mapping "SPDT" to "Special Plate Dal Tadka" — twenty rows that fix a failure no embedding work can touch.

### Where it lives

Keyword search is a full-text index in the same Postgres that holds your vectors. Reranking is one more model call behind the same `search_docs` read tool. Citation verification is plain code, in three parts:

- Every chunk arrives carrying its id, or the model invents plausible ones
- The schema reserves a `source_chunk_id` per claim, from an enum of the ids you supplied, so an invented id is not representable
- Your code checks the cited id was in the set you sent and that the quoted sentence appears in that chunk, marking the answer unverified if not

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

**Do not add scores from different systems.** It feels obvious and quietly destroys your ranking. Weighted fusion only once chapter twelve gives you numbers to tune against. And hybrid search cannot rescue bad ingestion: if sixty documents extracted blank, both methods find nothing.

**Reranking costs latency on every query** — worth the wait for a chat reply, not a typeahead. A wider set is not better: two hundred candidates cost five times as much and add little, because fusion already put the right chunk in the top forty. And rerankers read words, not truth, so a well-written irrelevant chunk can outrank a terse relevant one.

**Verification must be code, not an instruction.** "Only cite documents that contain the claim" is a request; a set membership test is a guarantee. A fabricated citation is worse than none, because it fakes the appearance of a check.

### Verify it

- **Ablate four configurations on the same twenty questions** — keyword, vector, hybrid, hybrid plus rerank — counting how many had the correct chunk in the top five.
- **Measure retrieval separately from the answer.** Recall at five is the number to improve first, because nothing downstream recovers from a miss.
- **Open three citations and read them** — not skim, find the sentence. It is what catches a fabricated citation.
- **Turn on verification and count the failures.** If the rate is zero, corrupt a chunk id to confirm the check runs.
- **Test one exact-match query and one paraphrase at every stage.** Many teams ship hybrid search with one leg silently broken and the other covering for it.

Red flags: a citation pointing at a document that lacks the claim; citations nobody opens; two search scores added; no keyword search in a product full of codes; reranking over the whole corpus; verification written as a prompt sentence; a confidence percentage from a similarity score; no recall measurement.

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
