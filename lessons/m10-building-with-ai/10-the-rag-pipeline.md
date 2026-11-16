---
id: m10-10-rag-pipeline
module: m10-building-with-ai
slug: the-rag-pipeline
title: "The RAG Pipeline: Chunk, Embed, Store, Retrieve"
order: 10
duration_min: 25
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

Two teams build retrieval over the same two hundred college circulars and fail in opposite directions.

The first embeds each document whole: one vector per twelve-page circular. Retrieval returns five entire circulars, sixty pages of text, which blows the context budget from chapter five, and the one sentence that answers the question sits somewhere in the middle where chapter five already told you attention is weakest.

The second splits everything into hundred-character pieces. Now a chunk reads "shall be ₹42,000 per annum" with no indication of what, for whom, or from when. Retrieval finds it, the model reads it, and the answer is a number with no subject. Several chunks end mid-word. One table is spread across nine chunks.

Then both teams discover the same thing: they embedded their documents with one model and their questions with another, so the two sets of coordinates have nothing to do with each other and the whole system was returning near-random text the entire time.

### The idea

**RAG is four stages, and each one can silently ruin the next: split documents into retrievable units, turn each unit into a vector, store the vectors with their metadata, and at question time fetch the nearest few.**

The word to hold onto is *silently*. None of these stages throws an error when it is done badly. A bad chunk is a valid chunk. A mismatched embedding is a valid vector. Only measurement tells you, which is why chapter twelve exists.

### The analogy

The canteen's recipe knowledge used to live in six cookbooks on a shelf. Finding anything meant knowing which book.

So they rewrote it as index cards: one card per dish, each card complete enough to cook from on its own — ingredients, quantities, method, and the dish name at the top. Then they filed the cards not alphabetically but *by taste*, so "spicy, gravy, chicken" sits next to "spicy, gravy, mutton", and a cook who wants something spicy and gravy-ish can pull the nearby cards without knowing any dish names.

That is the whole of RAG. Chunking is writing self-contained cards. Embedding is the taste coordinates. The store is the filing box. Retrieval is grabbing the five nearest cards.

**Where the analogy breaks:** cards that are near each other by taste can be entirely the wrong dish — a spicy gravy mutton card is a perfectly good match for a spicy gravy chicken question and a useless answer. Nearness in meaning is not correctness. And the person who filed the cards understood the dishes; the embedding model only saw the words, so a card full of mangled OCR gets filed somewhere confidently wrong.

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

### Chunking: write cards, not fragments

The test for a good chunk is one question: **could a human answer the question from this chunk alone, with nothing else?** If not, the chunk is too small or missing its context.

**Split on structure first.** Headings, numbered clauses, paragraph breaks, question-answer pairs in an FAQ. Documents already carry the author's own idea of where one idea ends, and it beats any character count.

**Then cap the size.** Something in the range of a few hundred to a thousand tokens suits most prose. Measure rather than believe; your right answer depends on your documents and chapter twelve is where you find it.

**Overlap a little.** Repeating the last sentence or two of the previous chunk stops an answer being severed exactly at a boundary. Ten to fifteen percent is a common starting point, and too much overlap just means retrieving the same text twice.

**Never split a table.** Keep it whole in one chunk or handle it as chapter nine described.

**Prefix every chunk with its context.** This is the cheapest large win in the chapter. Store the chunk as "Fee Circular 2026-04, effective 1 July 2026 · Section 3: Hostel charges — shall be ₹42,000 per annum for second-year students." Now the fragment has a subject, a date and a scope, and it answers on its own. A chunk that needs its neighbours to make sense will fail the moment it is retrieved without them, which is always.

### Embedding: one model, both sides, forever

M09 taught embeddings as meaning turned into coordinates. Three practical consequences.

**The same model embeds documents and questions.** Different models produce incompatible coordinate systems. This is the bug both teams in the opening story had, and it looks exactly like "retrieval is bad" rather than like a configuration error.

**Changing the embedding model means re-embedding everything.** It is a data migration in M04's sense — plan it, keep the old vectors until the new ones are verified, and do not do it casually.

**Free-tier reality.** Embedding is cheap compared with generation, but two hundred documents is thousands of chunks and provider free tiers have daily limits. Batch your requests, store the vectors so you never recompute them, and consider a small local embedding model: these run acceptably on a laptop, cost nothing, and keep your documents out of a third party's logs.

### Store: the database you already have

A vector store is a table that holds a vector per row plus whatever metadata you want to filter on. For a student project the right answer is almost always the Postgres from M04 with a vector extension, because it is already there, it is free, and your chunks can sit next to your other tables.

Two things matter beyond the vectors themselves.

**Metadata lives alongside.** Document id, title, page, effective date, language, access level — all of chapter nine's output. Without it you cannot cite, filter or detect staleness.

**Search is approximate.** Comparing a query against every vector is exact and slow, so stores build an index that finds *probably* the nearest. That is the right trade at scale and it means recall is never quite one hundred percent. At a few thousand chunks you can often afford exact search, and you should, because one fewer source of uncertainty while you are learning is worth a lot.

### Retrieve: filter, then rank

**Pre-filter on metadata, then search.** Access level, language, effective date. A pre-filter searches only the rows this user may see. A post-filter searches everything and throws away what the user may not see — which means a student's five best chunks might all be staff documents, discarded, leaving nothing. Worse, some stores make it easy to forget the filter entirely, which is chapter eight's incident.

**Choose top-k by measurement.** Five is a reasonable start. The needle test from chapter five tells you where your own threshold sits.

**Keep a similarity floor.** If the best match is far away, the honest result is nothing retrieved, which lets the model abstain instead of blending five unrelated chunks.

### Where it lives

Retrieval is a read tool in chapter four's sense — `search_docs(query)` returning chunks with their ids — and it fills the data block of chapter five's context. The store is M04's database. The embedding job is M04's queue-and-worker pattern, run after chapter nine's ingestion.

Chapter eleven fixes what pure vector search cannot do, chiefly exact matches on codes and names. Chapter twelve measures all of it. And chapter eight remains the gate: if you got here without needing retrieval, the prompt or a tool was the better answer.

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

**Big chunks dilute, small chunks decapitate.** A long chunk buries the answer among distractors and costs tokens on every call. A short one loses its subject. The right size is the smallest unit that still stands alone, and that is a property of your documents.

**Embeddings cannot do exact match.** "SPDT", an order id, a part number, a person's surname — vector search is about meaning and these have none. It will return things that are *about* the same area and miss the exact token. This is the single biggest weakness of pure vector search and it is precisely what chapter eleven fixes.

**Retrieval is not free latency.** Embed the query, search, then generate. That is two network round trips before the first token, and on 4G it is felt. Cache repeated queries and keep k small.

**A near-empty chunk is a magnet.** Chapter nine's blank extractions embed to an unhelpfully central position and match many questions. If you skipped the scoring stage, you will see it here.

**Re-chunking is cheap; re-ingesting is not.** Store extracted text separately from chunks so you can change your chunking strategy in an afternoon without touching OCR. Teams that chunk straight from PDFs pay for that choice repeatedly.

**Similarity scores are not comparable across models or corpora.** 0.78 means nothing on its own. Only the ranking within one query and one model is meaningful, so never hard-code a threshold you have not calibrated on your own data.

### Verify it

**Look at the chunks before you look at the answer.** This is the habit that separates people who can debug RAG from people who cannot. Print the five retrieved chunks for every test question. If the answer is not in any of them, the generation step is innocent and no prompt change will help.

**Read ten chunks in isolation, chosen at random.** Cover up the document they came from. Can you tell what each is about, who it applies to and when? Anything that fails this test needs a contextual prefix.

**Verify both sides use the same embedding model.** Check it in the code, not from memory. Then sanity-check it: embed one chunk's exact text as a query and confirm that chunk comes back first. If it does not, something is badly wrong and nothing downstream is worth testing.

**Query as two different users and compare the chunk lists.** A student and a staff member. If the lists are identical, your permission filter is not running.

**Search for something exact — a code, an id, a rare name.** Watch it fail. Experiencing this failure now is what makes chapter eleven make sense, and it is better to meet it in a test than in a demo.

Red flags: retrieved chunks that do not contain the answer while the model answers anyway; chunks that are unintelligible out of context; a table split across chunks; different embedding models on the two sides; no metadata on chunks; a permission filter applied after search; top-k picked because it sounded safe; a hard-coded similarity threshold copied from a blog post; and chunking done directly from PDFs with no stored intermediate text.

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
