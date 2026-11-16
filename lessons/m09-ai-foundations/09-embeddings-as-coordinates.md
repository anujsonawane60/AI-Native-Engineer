---
id: m09-09-embeddings
module: m09-ai-foundations
slug: embeddings-as-coordinates
title: "Embeddings: Meaning as Coordinates"
order: 9
duration_min: 25
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

A student types "cheap veg lunch" into the canteen app's search box. Nothing comes back.

The canteen sells pongal at ₹40, which is cheap, vegetarian, and served at lunch. But the word "cheap" is not in the dish's name, "veg" is not in its description, and "lunch" is not written anywhere. The search box compares letters, and no letters match. The dish the student wanted is item three on the menu.

The team's first fix is a synonym list. "Veg" matches vegetarian, vegan, no-onion-no-garlic. Then "cheap" needs to mean under fifty rupees. Then a student types "something light after gym" and the whole approach collapses, because that is not a synonym problem. It is a meaning problem, and letters do not carry meaning.

### The idea

**An embedding turns a piece of text into a long list of numbers that works as a position in space, arranged so that things with similar meanings end up near each other.**

Feed "cheap veg lunch" to an embedding model and you get back a few hundred to a few thousand numbers. Feed "pongal, ₹40, rice and lentils" and you get another list. The two lists are close together, even though they share not a single word. Searching by distance instead of by letters is called **semantic search**, and it is one of the highest-value, lowest-effort features a student product can add.

The same trick works on anything that can be embedded: product descriptions, support tickets, images, audio. Similar things sit near each other, and "near" is something a computer can compute in microseconds.

### The analogy

Imagine the canteen rearranged its seating so that where you sit encodes what you came for.

The far-left tables are for people in a hurry; the far-right for people who will sit for an hour. The front rows are cheap food, the back rows expensive. Spicy on the upper tables, mild on the lower. Nobody is told these rules — the manager just learned, over years, where each kind of customer belongs, and seats them accordingly.

Now a new student walks in and says "something light after gym". The manager does not match keywords. He walks them to a spot: front-left-lower, because that is where light, quick, mild, inexpensive lives. And whoever is already sitting nearby is eating roughly what this student wants. That seating position is an **embedding**, and "look at who is sitting nearby" is **nearest-neighbour search**.

**Where the analogy breaks:** the canteen has two dimensions and you can name both axes. A real embedding has hundreds or thousands of dimensions and **not one of them has a name**. There is no "spiciness" axis to inspect. Meaning is spread across all of them in a way no human can read, which is why you cannot debug an embedding by looking at it — you can only test it with queries. And the seating plan was learned from the model's training data, not from your menu, so it reflects the world's general sense of similarity rather than your canteen's.

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

### How "near" is measured, and what it does not mean

Distance between two embeddings is usually measured as the angle between them rather than the straight-line gap — **cosine similarity**, which comes out as a number near 1 for "pointing the same way" and near 0 for unrelated. You will see these numbers constantly and you need one habit about them: **a similarity score is a ranking signal, not a truth value.**

A score of 0.83 does not mean 83% correct. It means "closer than the thing scoring 0.79". Nearest-neighbour search always returns something — ask for the top five and you get five results whether or not any of them are relevant. If your canteen has no light vegetarian dish at all, the search will confidently hand you the five least-wrong things on the menu, with respectable-looking scores.

This is the single most common failure in retrieval systems built by beginners, and it is the reason M10 spends a whole chapter on retrieval failure modes. A threshold below which you say "nothing matched" has to be chosen by testing on your own queries, not copied from a tutorial.

### Where it lives

This is the chapter where several earlier modules join up, so place it carefully.

**Who makes the embedding.** An embedding model, which is a different and much smaller model than a chat model. You call it over the network, or run a small one locally, and you get numbers back. It is cheap — typically far cheaper per token than a chat model — and fast.

**When you make it.** Your documents are embedded once, when they are added or changed. The user's query is embedded at request time, on every search, which means a semantic search costs you one embedding call plus a database lookup.

**Where it is stored.** In a vector column or a vector database, which is M04's territory — a Postgres table with a vector column using pgvector is the standard free-tier answer, and a dedicated vector service is the alternative. The thing stored alongside each vector is the text it came from and an id, because the vector itself is unreadable.

**What it enables.** Retrieval-augmented generation, which is M10 chapters 8 to 12. Embeddings are the "how do I find the right three paragraphs" half of RAG, and this chapter is the foundation under it. Everything in M10 about chunking, hybrid search and reranking assumes you understand that similarity is geometric and approximate.

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

**The query and the documents must be embedded by the same model.** This is the rule that breaks the most projects. Two embedding models place meaning in completely different, incompatible spaces, so a query embedded by model A and documents embedded by model B produce results that look plausible and are nonsense. If you change embedding models, you must re-embed every document you have — which costs money and time, and is why the choice is worth five minutes of thought rather than none.

**Semantic search is worse than keyword search at some things, and they are things your users do.** Exact product codes, order ids, specific names, phone numbers, prices. "Order 4471" is a letters problem, and embeddings are vague about it. The honest answer in production is almost always hybrid: run both and combine, which is M10.

**Dimensions cost storage and speed.** A 3,072-dimension embedding is four times the storage of a 768 one and slower to search. On a free-tier Postgres with a few hundred megabytes, that difference decides how many documents you can hold. Bigger is not automatically better here either.

**Embeddings inherit every bias in their training data.** The geometry of meaning was learned from text written by people, with all of that text's assumptions baked in. Associations between names, places, castes, genders and professions sit in the space whether you want them or not, and a recommendation built on nearness will reproduce them quietly. This is M07's concern and chapter 16's, and it starts here rather than in the chat model.

**Chunk size changes everything downstream.** Embedding a whole 40-page document into one vector averages its meaning into mush. Embedding each sentence separately loses the context around it. Getting this right is most of what makes a retrieval system good, and it is M10's work — but the reason it matters is this chapter's geometry.

### Verify it

**Red flag: a similarity score treated as a confidence or a correctness percentage.** Ask what score your known-irrelevant queries get. If a nonsense query scores 0.74 against your best document, then 0.74 means nothing and your threshold is fiction.

**Red flag: no threshold and no "nothing found" path.** Ask the direct question: what does this return when the answer genuinely is not in my data? If the answer is "the top five anyway", you have a system that cannot say "I do not have that", and it will mislead users in exactly the situation where being honest mattered most.

**Red flag: different models for query and documents.** Ask explicitly which embedding model embeds the documents and which embeds the query, by exact name. They must match. Also ask what happens if you later switch — the honest answer involves re-embedding everything.

**Red flag: an invented embedding model name or dimension count.** Same red flag as chapters 5 and 7, in a new place. Check the exact name and its dimension count on the provider's own page.

**Red flag: no mention of keyword search anywhere.** If a plan proposes pure semantic search for a product where users type order numbers and dish names, it has an obvious gap. Ask how "order 4471" is handled.

And the test you run yourself, which takes fifteen minutes and is worth more than any amount of reading: write down twenty real queries your users would actually type, including three in your users' own language, three with spelling mistakes, three asking for something you genuinely do not have, and two containing an exact code or number. Run all twenty. Record, for each, whether the right thing came back in the top three. That table is your honest measurement — chapter 3's discipline applied to retrieval — and it is the only thing that tells you whether this feature works.

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
