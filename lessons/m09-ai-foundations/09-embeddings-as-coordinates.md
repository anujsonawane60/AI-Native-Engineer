---
id: m09-09-embeddings
module: m09-ai-foundations
slug: embeddings-as-coordinates
title: "Embeddings: Meaning as Coordinates"
order: 9
duration_min: 15
is_free: false
mission: false
volatility: low
last_reviewed: 2026-11-16
---

## Key learnings
- Describe an embedding as a position in space where nearby means similar in meaning
- Say why semantic search finds things that keyword search misses, and where it fails instead
- Explain why the query and the documents must be embedded by the same model
- Place embeddings on your architecture: who generates them, where they are stored, what they cost
- Catch an AI treating similarity as truth or ignoring the cost of re-embedding

## Lesson

### The pain

A student types "cheap veg lunch" into the canteen app's search box and nothing comes back. The canteen sells pongal at ₹40 — cheap, vegetarian, served at lunch — but none of those words appear in its name, and the search box compares letters. A synonym list survives until a student types "something light after gym". That is a meaning problem, and letters do not carry meaning.

### The idea

**An embedding turns a piece of text into a long list of numbers that works as a position in space, arranged so that things with similar meanings end up near each other.**

Feed "cheap veg lunch" to an embedding model and you get a few hundred to a few thousand numbers; feed "pongal, ₹40, rice and lentils" and you get another list. The two land close together despite sharing no words. Searching by distance instead of letters is **semantic search**, and it works on anything embeddable: descriptions, tickets, images, audio.

### The analogy

Imagine the canteen rearranged its seating so that where you sit encodes what you came for: far-left for people in a hurry, front rows cheap, back rows expensive, spicy upstairs, mild downstairs. Nobody wrote these rules — the manager learned over years where each kind of customer belongs. A student says "something light after gym" and he matches no keywords; he walks them to front-left-lower, where light, quick, mild and inexpensive lives, and whoever sits nearby is eating roughly what this student wants. That position is an **embedding**, and "look at who is sitting nearby" is **nearest-neighbour search**.

**Where the analogy breaks:** the canteen has two dimensions and you can name both axes. A real embedding has thousands and **not one of them has a name**, so you cannot debug an embedding by looking at it, only test it with queries. And the seating plan came from the model's training data, not your menu.

### Picture it

```
  a 2-D sketch of something that really has 1,500 dimensions

   expensive
       ▲
       │        · paneer butter masala
       │                    · biryani
       │   · masala dosa
       │                              · "heavy dinner"
  ─────┼──────────────────────────────────────────▶ spicy
       │  · pongal
       │     · curd rice      · idli
       │   ★ "cheap veg lunch"
       │   ★ "something light after gym"
     cheap

  the two queries (★) land near pongal, curd rice and idli
  sharing no words with any of them
```

Text becomes a position, similar meanings land near each other, and a query is answered by returning whatever sits closest to it.

Distance is usually the angle between two embeddings — **cosine similarity**, near 1 for "pointing the same way" and near 0 for unrelated. One habit matters: **a similarity score is a ranking signal, not a truth value.** 0.83 does not mean 83% correct, only "closer than the thing scoring 0.79". Nearest-neighbour search always returns something, so with no light vegetarian dish on the menu it hands you the five least-wrong items at respectable-looking scores. The threshold below which you say "nothing matched" must be chosen by testing your own queries.

### Where it lives

- **Who makes it.** An embedding model — far smaller and cheaper than a chat model, and fast.
- **When.** Documents are embedded once, when added or changed; the query is embedded on every search, so one search costs one embedding call plus a database lookup.
- **Where it is stored.** A vector column or vector database — pgvector in Postgres is the standard free-tier answer. Store the original text and an id beside each vector, because the vector itself is unreadable.
- **What it enables.** Retrieval-augmented generation in M10, whose chunking, hybrid search and reranking all assume this geometry.

### Words to own

| Term | Plain meaning | Example |
|---|---|---|
| Embedding | A list of numbers acting as a position for meaning | 1,536 numbers |
| Vector | The same thing, said mathematically | One row in your table |
| Dimension | How many numbers each embedding has | 384, 768, 1,536, 3,072 |
| Embedding model | The model that produces embeddings | Small, cheap, fast |
| Cosine similarity | Closeness measured as an angle | 0.83 |
| Nearest neighbour | The closest stored items to a query | Top 5 results |
| Semantic search | Searching by meaning rather than letters | "light after gym" |
| Keyword search | Matching the actual characters | Finds "pongal" exactly |
| Hybrid search | Both together, results combined | M10's usual answer |
| Vector database | Storage that can search by closeness | pgvector in Postgres |
| Index | The structure that makes the search fast | Built over the vectors |
| Re-embedding | Regenerating every vector | Needed if you switch models |

### Tradeoffs and gotchas

- **The query and the documents must be embedded by the same model** — the rule that breaks the most projects. Two models place meaning in incompatible spaces, so mixing them gives plausible nonsense, and switching means re-embedding everything you have.
- **Semantic search is worse than keyword search at things your users do**: order ids, product codes, prices. "Order 4471" is a letters problem, so the production answer is usually hybrid.
- **Dimensions cost storage and speed.** A 3,072-dimension embedding is four times the storage of a 768 one, which on a free-tier database decides how many documents you can hold.
- **Embeddings inherit every bias in their training data.** Associations between names, places, castes, genders and professions sit in the space whether you want them or not.
- **Chunk size changes everything downstream.** One vector for a 40-page document averages its meaning into mush; one per sentence loses the surrounding context.

### Verify it

- **A similarity score treated as a confidence or a percentage.** Ask what your known-irrelevant queries score; if nonsense scores 0.74, then 0.74 means nothing.
- **No threshold and no "nothing found" path.** Ask what this returns when the answer is genuinely not in your data. "The top five anyway" means a system that cannot say "I do not have that".
- **Different models for query and documents.** Ask by exact name which model embeds each; they must match.
- **An invented embedding model name or dimension count.** Check both on the provider's own page.
- **No mention of keyword search anywhere.** Ask how "order 4471" is handled.
- **Check yourself:** run twenty real queries — three in your users' language, three misspelled, three with no answer in your data, two with an exact code — and record whether the right item came back in the top three.

## Walkthrough
1. Write down five real search queries your users would type in their own words. Then write the exact text of the items in your data that should match them.
2. Count how many of those five would be found by letter matching alone. That number is the gap semantic search is for.
3. Open any free embedding playground or notebook and embed two sentences that mean the same thing in different words. Note the dimension count — how many numbers came back.
4. Embed two sentences that share many words but mean different things. Compare the two similarity scores and write down what surprised you.
5. Embed ten items from your own data plus one query, and rank the ten by similarity to the query. Read the ranking and judge it yourself.
6. Now embed a deliberately irrelevant query — something your data has no answer for — and record the top score it gets. This number is your noise floor and it is the most useful thing in this walkthrough.
7. Using that noise floor, pick a threshold below which your app should say "nothing matched", and write it down with the evidence behind it.
8. Write a query containing an exact code or order number and see where it ranks. Record the failure, because this is the case that needs keyword search.
9. Work out the storage: your number of documents times your dimension count, and check it against your free-tier database limit. Write the number down.
10. Finally, write one decision-log entry: the embedding model by exact name, its dimensions, your threshold with its evidence, and what re-embedding everything would cost if you switched.

## Talk to AI
```prompt
I am adding search by meaning to my product and I want to understand the geometry and the failure modes before any code exists.

My situation:
- What users are searching: [describe the items, and roughly how many there are]
- Example queries real users would type: [list five, including at least one in my users' own language and one containing an exact code or number]
- Where my data is stored today: [e.g. Postgres free tier]
- Rough total size of the text I would embed: [e.g. 400 items, 2 paragraphs each]

Please work through this and stop after step 4:

1. Explain in plain language what embedding my items would do, and why one of my example queries that shares no words with its target would still find it.

2. Go through my five example queries one at a time and predict, with a reason, whether semantic search would handle each one WELL, BADLY, or NEEDS KEYWORD SEARCH INSTEAD. Be blunt about the ones that will fail.

3. Tell me what a similarity score is and is not. Specifically: explain why 0.8 is not "80% correct", and tell me exactly how I should determine a cutoff below which my app says "nothing matched". I want a procedure I can run, not a number to copy.

4. Tell me what my system should do when the answer genuinely is not in my data, and what it will do by default if I do not handle that.

5. Then: what embedding model would you suggest, what is its dimension count, and how much storage would my data need? Mark any number you are not certain about, and tell me which official page to check.

6. Explain what happens if I later switch embedding models, including whether I have to re-embed everything and what that would cost me.

7. Finally, give me a test plan of twenty queries I should run to measure whether this actually works - tell me what categories the twenty should cover, and what result would tell me to abandon the approach.
```

## Stuck? Ask AI
```prompt
Explain embeddings using an analogy about a map of a city where similar shops end up in the same neighbourhood. Then tell me exactly where the analogy breaks down compared to real embeddings, and ask me to guess two of the breakdowns before you tell me.
```

```prompt
I do not understand why two embedding models cannot be mixed. Explain it concretely, then describe what my search results would actually look like if I made that mistake - would it error, or would it silently return bad results? Then tell me how I would detect it.
```

```prompt
Give me ten search queries for a food app. For each, tell me whether keyword search, semantic search, or both together would serve it best, and why. Then give me five more and ask me to classify them myself before you answer.
```

## Quiz

### Q1 · L2
What is an embedding?
- (a) A compressed copy of the original text
- (b) A list of numbers acting as a position, where nearby means similar in meaning ✅
- (c) A keyword index built over your documents
- (d) A smaller version of a language model
> It is a position in a high-dimensional space learned so that similar meanings land near each other, which is what lets a computer compare meaning by distance.

### Q2 · L3
"Cheap veg lunch" finds a dish called "pongal ₹40" despite sharing no words. Why?
- (a) The system expanded the query using a synonym list
- (b) Both texts embed to nearby positions because their meanings are similar ✅
- (c) The dish name was translated before matching
- (d) Keyword search matched the rupee symbol
> Embeddings place meaning geometrically, so a query and a document can be neighbours without sharing any characters. That is exactly what keyword search cannot do.

### Q3 · L4
Your documents were embedded with one model and you embed queries with a different one. What happens?
- (a) An error is raised because the dimensions differ
- (b) Results come back looking plausible but are effectively meaningless ✅
- (c) Quality drops by roughly ten percent
- (d) The system falls back to keyword search automatically
> Each model has its own incompatible space. Nothing errors if the dimensions happen to match, which is what makes this failure so dangerous — the output looks normal.

### Q4 · L4
A nonsense query against your data returns a top result with a similarity of 0.74. What have you learned?
- (a) The system is 74% confident the result is correct
- (b) Your scores have a high noise floor, so 0.74 cannot be used as a relevance threshold ✅
- (c) The embedding model is broken and should be replaced
- (d) You need a larger number of dimensions
> Nearest-neighbour search always returns something. Measuring what irrelevant queries score is how you find a defensible cutoff instead of copying one from a tutorial.

### Q5 · L5
Users of your canteen app frequently search by exact order number. What should the design be?
- (a) Pure semantic search, with more dimensions
- (b) Pure keyword search for everything
- (c) Hybrid: keyword search for exact identifiers, semantic for meaning-based queries ✅
- (d) Semantic search with a lower similarity threshold
> Embeddings are vague about exact strings and numbers, which is precisely what an order id is. Production retrieval almost always combines both, which M10 develops in depth.

## Go deeper
- [Google: embeddings in the machine learning crash course](https://developers.google.com/machine-learning/crash-course/embeddings)
- [pgvector: vector similarity search in Postgres](https://github.com/pgvector/pgvector)
