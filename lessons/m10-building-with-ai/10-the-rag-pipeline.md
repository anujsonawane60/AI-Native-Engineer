---
id: m10-10-rag-pipeline
module: m10-building-with-ai
slug: the-rag-pipeline
title: "The RAG Pipeline: Chunk, Embed, Store, Retrieve"
order: 10
duration_min: 15
is_free: false
mission: false
volatility: low
last_reviewed: 2026-11-16
---

## Key learnings
- Walk the four stages of RAG and name what each one can silently ruin
- Choose a chunk that a human could answer from on its own
- Use the same embedding model on both sides, forever
- Apply permission filters before the search, not after
- Inspect retrieved chunks before judging the generated answer

## Lesson

### The pain

Two teams build retrieval over the same two hundred circulars and fail in opposite directions. The first embeds each document whole, so retrieval returns five entire circulars — sixty pages — with the one useful sentence in the middle, where attention is weakest. The second splits everything into hundred-character pieces, so a chunk reads "shall be ₹42,000 per annum" with no indication of what, for whom, or from when.

Then both discover the same thing: they embedded documents with one model and questions with another, so the two sets of coordinates have nothing to do with each other and the system was returning near-random text all along.

### The idea

**RAG is four stages, and each can silently ruin the next: split documents into retrievable units, turn each into a vector, store the vectors with their metadata, and at question time fetch the nearest few.**

The word to hold onto is *silently*. A bad chunk is a valid chunk; a mismatched embedding is a valid vector. Nothing throws an error, so only measurement tells you.

### The analogy

The canteen's recipes used to live in six cookbooks, so finding anything meant knowing which book. They rewrote them as index cards: one per dish, each complete enough to cook from alone, filed not alphabetically but *by taste*, so "spicy, gravy, chicken" sits beside "spicy, gravy, mutton". That is RAG — chunking writes the cards, embedding is the taste coordinates, the store is the box, retrieval grabs the five nearest.

**Where the analogy breaks:** cards near each other by taste can be the wrong dish — mutton is a fine match for a chicken question and a useless answer. Nearness in meaning is not correctness. And whoever filed the cards understood the dishes; the embedding model only saw words, so mangled OCR gets filed somewhere confidently wrong.

### Picture it

```
  INGEST SIDE                      QUERY SIDE
  documents                        "hostel fee 2nd year?"
      │                                    │
      ▼ chunk                              ▼ embed
  self-contained units             query vector
      │                                    │
      ▼ embed (model X)                    ▼
  vectors + metadata ──▶ STORE ◀───── nearest k, with
                         (filter by        a pre-filter on
                          access level)    access + language
                                           │
                                           ▼
                                      5 chunks ──▶ context
                                                   block (ch 5)
```

The same embedding model must be used on both sides or the coordinates are not comparable. The permission filter is applied inside the store, before the nearest-neighbour search picks winners.

The test for a good chunk is one question: **could a human answer from this chunk alone?** Split on structure first — headings, clauses, FAQ pairs — because the author already marked where one idea ends. Cap the size, overlap ten to fifteen percent so an answer is not severed at a boundary, never split a table. Then prefix every chunk with its context: "Fee Circular 2026-04, effective 1 July 2026 · Section 3 — ₹42,000 per annum for second-year students." A chunk needing its neighbours is always retrieved without them.

### Where it lives

Retrieval is a read tool in chapter four's sense — `search_docs(query)` returning chunks with ids — and it fills the data block of chapter five's context. The store is M04's database: for a student project, the Postgres you already have plus a vector extension, chapter nine's metadata in columns beside each vector.

**Pre-filter on metadata, then search** — access level, language, effective date — because a post-filter searches everything and discards what the user may not see, leaving a student with nothing. And **keep a similarity floor**, so a distant best match returns nothing.

### Words to own

| Term | Plain meaning | Example |
|---|---|---|
| Chunk | One retrievable, self-contained unit | A clause plus its heading |
| Overlap | Text repeated between adjacent chunks | The last two sentences |
| Contextual prefix | Title and section prepended to a chunk | "Fee Circular 2026-04 · S3" |
| Embedding | Text turned into coordinates of meaning | A 768-number vector |
| Dimension | How many numbers per vector | 768 |
| Cosine similarity | How close two vectors point | 0.83 |
| Vector store | A table of vectors plus metadata | Postgres with pgvector |
| ANN index | Approximate nearest-neighbour search | Fast, not exhaustive |
| Top-k | How many chunks you fetch | k = 5 |
| Pre-filter | Restricting rows before the search runs | `access_level = 'student'` |

### Tradeoffs and gotchas

**Big chunks dilute, small chunks decapitate.** A long chunk buries the answer among distractors; a short one loses its subject. The right size is the smallest unit that still stands alone.

**Embeddings cannot do exact match.** "SPDT", an order id, a part number, a surname — vector search is about meaning and these have none. It returns things *about* the right area and misses the exact token. That is chapter eleven's subject.

**Changing the embedding model means re-embedding everything**, a data migration in M04's sense. Embedding is cheap next to generation, but thousands of chunks meet a free-tier daily limit, so batch, store, and consider a small local model that keeps your documents out of third-party logs.

**Similarity scores are not comparable across models or corpora.** 0.78 means nothing alone; only the ranking within one query is meaningful. And a near-empty chunk is a magnet, because chapter nine's blank extractions embed centrally and match many questions.

### Verify it

- **Look at the chunks before the answer.** Print the five retrieved chunks per test question; if the answer is in none, generation is innocent and no prompt change helps.
- **Read ten random chunks in isolation**, source hidden. Can you tell what each is about, who it applies to, and when? Anything that fails needs a contextual prefix.
- **Verify both sides use the same embedding model** in code, not from memory. Then embed one chunk's exact text as a query and confirm it comes back first.
- **Query as two different users and compare the chunk lists.** Identical lists mean your permission filter is not running.
- **Search for something exact — a code, an id, a rare name — and watch it fail.** Meeting that in a test rather than a demo is what makes chapter eleven land.

Red flags: retrieved chunks that lack the answer while the model answers anyway; chunks unintelligible out of context; a table split across chunks; different embedding models on the two sides; no metadata on chunks; a permission filter applied after search; top-k picked because it sounded safe; a similarity threshold copied from a blog post; chunking straight from PDFs.

## Walkthrough
1. Start from chapter nine's output: clean extracted text with metadata in a table. Confirm you are not chunking from raw PDFs.
2. Write five test questions with the answer you expect and the document the answer lives in. Keep this list; it becomes your eval set in chapter twelve.
3. Chunk one document by structure — headings, clauses, paragraphs — with a token cap and about ten percent overlap. Do not touch the rest of the corpus yet.
4. Print ten chunks from that document and read each one with the source hidden. Note every chunk you cannot understand alone.
5. Add a contextual prefix to every chunk: document title, effective date and section heading. Re-read the same ten and note how many now stand alone.
6. Embed those chunks, and make the embedding model a single named constant in your code so it cannot differ between the two sides by accident.
7. Store vectors plus all of chapter nine's metadata. Then embed one chunk's exact text as a query and confirm that same chunk is the top hit.
8. Run your five test questions and print the retrieved chunks only — no generated answer yet. Mark each question as found, partially found or missed.
9. Add a metadata pre-filter on access level and language, then run the same questions as two different user types and compare the two chunk lists.
10. Now generate answers, and for each one check whether the claim is actually in the chunks that were retrieved rather than merely plausible.
11. Finally, search for an exact code, id or rare name and record the failure. That failure is the brief for the next chapter.

## Talk to AI
```prompt
Design the chunking and retrieval strategy for my own documents, and tell me what to measure rather than what to believe.

MY DOCUMENTS: [what they are, how many, typical length, structure - headings? clauses? FAQ pairs? tables?]
MY METADATA, from ingestion: [list the fields you have per document and per chunk]
MY USER TYPES AND WHAT EACH MAY SEE: [be specific]
MY TEST QUESTIONS, with the expected answer and the document it lives in:
1. [question - answer - document]
2. [...]
MY CONSTRAINTS: free tiers only, Postgres already in use, and I would rather not add a new paid service.

Deliver all of this:

1. A chunking strategy specific to my document structure: where to split, the token cap, the overlap, and how to keep tables whole. Justify each number and tell me how I would test whether it is right.
2. The exact contextual prefix format to prepend to each chunk, with one real worked example using my metadata fields.
3. For each of my five test questions, predict which chunk SHOULD be retrieved under your scheme, and what would cause it to be missed.
4. Where to store this given that I already run Postgres, and whether exact or approximate search is right at my corpus size. Say what changes that answer.
5. The metadata pre-filter for each of my user types, written as a condition, and an explanation of why pre-filtering rather than filtering the results afterwards.
6. A starting top-k and a similarity floor, plus the experiment that would tell me the right values for MY corpus rather than in general.
7. The retrieval-only test harness: how to print the retrieved chunks for each question so I can judge retrieval separately from generation.
8. Which of my test questions will fail because embeddings cannot do exact matching, and what kind of query they are.
9. What I must never change casually once this is live, and what the migration looks like if I do.
```

## Stuck? Ask AI
```prompt
Explain the four stages of RAG - chunk, embed, store, retrieve - to someone who understands databases but has never used embeddings. For each stage, tell me what a silent failure looks like: something that produces no error and ruins the stage after it. Use a worked example of a college fee circular.
```

```prompt
Here are five of my chunks: [paste them]. For each, tell me whether a human could answer a question from it with nothing else available. Where the answer is no, say exactly what context is missing and write the contextual prefix that would fix it.
```

```prompt
Explain why vector search cannot find an exact code like "SPDT" or an order id, even though the text is right there in a chunk. Use the idea of meaning-as-coordinates. Then tell me what kind of search DOES find it, and what a system needs to have both.
```

## Quiz

### Q1 · L3
What is the best test of whether a chunk is the right size?
- (a) It is close to the configured token cap
- (b) A human could answer the question from that chunk alone ✅
- (c) It ends at a sentence boundary
- (d) It embeds to a vector with high magnitude
> A chunk is a retrieval unit, so it has to stand alone — the model receives it without its neighbours. If a person cannot use it in isolation, neither can the model.

### Q2 · L4
Documents were embedded with one model and questions with another. What does it look like from the outside?
- (a) An error at query time
- (b) Retrieval that returns near-random text, looking like a bad RAG system ✅
- (c) Slower searches but correct results
- (d) Correct results with wrong similarity scores
> Two models produce incompatible coordinate systems. Nothing errors, so it presents as "retrieval quality is poor" and teams tune chunk sizes for weeks.

### Q3 · L4
Why must the permission filter be applied before the nearest-neighbour search rather than after?
- (a) Post-filtering is slower on large corpora
- (b) Post-filtering can discard all five results, leaving nothing, and is easy to omit ✅
- (c) Vector stores cannot filter after a search
- (d) Pre-filtering improves similarity scores
> Searching everything and then removing what the user may not see wastes the slots, and the filter being a separate later step is exactly how it gets forgotten.

### Q4 · L3
Your model answers confidently but the retrieved chunks do not contain the answer. Where is the fault?
- (a) Generation; tighten the prompt
- (b) Retrieval, plus a missing refusal rule ✅
- (c) The embedding model's dimension
- (d) The context window is too small
> If the evidence was never retrieved, no prompt change fixes it. Printing the chunks before judging the answer is the habit that locates this in seconds.

### Q5 · L5
Which query will pure vector search most reliably fail on?
- (a) "What are the hostel timings?"
- (b) "Explain the refund rules in simple words"
- (c) "What does SPDT cost?" where SPDT is a local code ✅
- (d) "Summarise the mess policy"
> Embeddings encode meaning, and a local code has none to encode. Vector search returns things in the same subject area and misses the exact token, which is why hybrid search exists.

## Go deeper
- [pgvector: vector similarity search for Postgres](https://github.com/pgvector/pgvector)
- [Anthropic: Contextual retrieval](https://www.anthropic.com/news/contextual-retrieval)
