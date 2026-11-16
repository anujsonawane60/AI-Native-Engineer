---
id: m10-08-grounding-ai
module: m10-building-with-ai
slug: grounding-ai
title: "Grounding AI in Private, Current and Domain-Specific Data"
order: 8
duration_min: 25
is_free: false
mission: false
volatility: low
last_reviewed: 2026-11-16
---

## Key learnings
- Name the three gaps between what a model knows and what your product needs
- Choose between prompt, tool, retrieval and fine-tuning for each gap
- Explain why fine-tuning teaches shape and does not add facts
- Require a citation and allow a refusal, or it is not grounding
- Scope retrieval to who is asking, not just to what was asked

## Lesson

### The pain

A team demos their canteen assistant. A mentor asks it the refund policy for a cancelled order.

It answers immediately and beautifully: cancellations within fifteen minutes are refunded in full, after that a fifty percent charge applies, and refunds reach the UPI account in three to five working days.

There is no refund policy. The team has never written one. Nothing in the product implements any of that. The model produced the most plausible refund policy for an Indian food business, because that is the shape of the thing it was asked for, and it had nothing specific to go on.

Then two more questions land. "Aaj ka rate kya hai" gets last semester's prices, which were in the demo data someone pasted weeks ago. And "SPDT kitne ka hai" gets a confused guess, because at this particular college SPDT is the Monday special and that fact exists nowhere outside the campus.

One failure, three different causes. They need three different fixes.

### The idea

**A model knows what was in its training data: public, general and frozen at a cutoff date. Your product needs answers that are private, current and specific to your domain. Grounding means supplying those facts at request time and requiring the answer to come from them.**

The second half of that sentence is the part teams skip. Putting the facts nearby is not grounding. Grounding is when the model must point at which fact it used, and must say "I do not know" when the fact is not there.

### The three gaps

**Private.** Your prices, your menu, your policies, your students' orders. Never in any training set, and you would be alarmed if it were.

**Current.** Today's sold-out dishes, this morning's price change, the complaint filed an hour ago. The model's knowledge stops at its cutoff and your product's most valuable facts are from this afternoon.

**Domain-specific.** SPDT. The hostel mess timings. The rule that first-years cannot use the staff counter. Your supplier's minimum order. Real, written down somewhere, and far too local to have ever been on the public internet.

### The analogy

You hire a skilled cook from another city. They genuinely know cooking — technique, timing, proportions, far better than you do. That is the training data.

They do not know your prices. They do not know the dosa batter ran out twenty minutes ago. And they have no idea that SPDT is a thing.

You fix those three with three different objects. The **menu board on the wall** is small, stable and always visible, so prices need no asking. The **storekeeper** is who you ask about stock, because stock changes hourly and only one person has the live number. And the **shelf of printed circulars** holds the college's rules, the supplier terms and the mess policy — too much to put on a board, consulted only when the question comes up.

**Where the analogy breaks:** after three weeks the cook has learnt the menu and stops glancing at the board. A model never does. Every single conversation starts from zero, so the board, the storekeeper and the shelf are permanent fixtures of the system rather than a training period that ends.

### Picture it

```
  question
     │
     ▼
  Is the fact small and stable?        ──yes──▶ PUT IT IN
  (a 40-row menu, your 6 policies)              THE PROMPT
     │ no
     ▼
  Is it structured and live?           ──yes──▶ GIVE IT
  (stock, order status, balance)                A TOOL
     │ no
     ▼
  Is it large unstructured text?       ──yes──▶ RETRIEVE IT
  (circulars, FAQs, 200 PDFs)                   (RAG)
     │ no
     ▼
  Is it about voice, format or style?  ──yes──▶ FINE-TUNE
  (not facts - never facts)                     (rarely)
```

Four routes, chosen by the nature of the fact rather than by what sounds impressive. Most product questions stop at the first or second box.

### Choosing, honestly

**Put it in the prompt** when the data is small and stable. A forty-row menu is about six hundred tokens. It is exact, it is cheap, it caches beautifully from chapter six, and there is no retrieval step to get wrong. Teams routinely build a vector database for data that would have fitted in the system prompt, and the retrieval version is less accurate.

**Give it a tool** when the data is structured and changes. Stock, order status, today's price, a student's own last order. Chapter four's `get_stock` is grounding — the freshest, most exact kind there is, because it reads the row at the moment of asking. For anything with a primary key, a tool beats embeddings.

**Retrieve it** when the data is large, unstructured and you cannot know in advance which part you will need. Two hundred college circulars. A year of WhatsApp complaints. The supplier contracts. That is what chapters ten and eleven build.

**Fine-tune** almost never, and never for facts. This is the single most expensive misconception in applied AI.

### Why fine-tuning does not add facts

Fine-tuning adjusts the model's weights on your examples. What it reliably teaches is *shape*: a house tone of voice, a rigid output format, a classification taste that is hard to describe in words, the knack of writing menu descriptions the way your brand writes them.

What it does not do is install knowledge you can rely on. Facts learnt this way come out blurred and confidently wrong, there is no way to cite a weight, you cannot delete one student's data from a trained model, and the day your prices change your model is stale and retraining is the only fix. Compare that with editing one row in a table.

The test is simple. If the answer changes when the data changes, it is retrieval or a tool. If the answer changes only when your taste changes, it might be fine-tuning.

### Citations and the right to refuse

Grounding is not finished when the facts are in the context. Two more requirements make it real.

**Every factual claim carries a source.** "Cancellations are refunded within fifteen minutes [circular-2026-04]." A citation is how you check the answer in five seconds instead of re-deriving it. Chapter eleven is about making citations trustworthy.

**"I do not know" is a legal answer.** If the retrieved text does not contain the refund policy, the correct output is that there is no refund policy on record — not a plausible one. This has to be stated as a rule, shown in a few-shot example from chapter one, and represented in the schema from chapter three, or the model will do the helpful thing and invent.

### Where it lives

Grounding sits in the context-assembly function from chapter five: it is what fills the data block. Where the facts come *from* is M04 — your tables, your object storage, your search index. Freshness is M05's cache-invalidation problem.

This chapter is the hinge of the module. Chapter nine gets messy real documents into a usable state, chapter ten builds the retrieval pipeline, chapter eleven makes the ranking and the citations trustworthy, and chapter twelve measures whether any of it works. Everything after chapter eight assumes you chose retrieval deliberately rather than by default.

### Words to own

| Term | Plain meaning | Example |
|---|---|---|
| Grounding | Answering from supplied facts, with a source | Menu rows in the prompt |
| Parametric knowledge | What the model learnt in training | General cooking technique |
| Knowledge cutoff | The date training data stops | Nothing about this term |
| Retrieval | Fetching relevant text at request time | Pulling 5 circular chunks |
| Fine-tuning | Adjusting weights on your examples | Teaching house tone |
| Citation | A pointer to the source of a claim | `[circular-2026-04]` |
| Abstention | Refusing when the facts are absent | "No policy on record" |
| Provenance | Where a fact came from and when | Doc id plus updated date |
| Freshness | How recent the grounded data is | Stock as of 14:02 |
| Permission-aware retrieval | Only searching what the asker may see | Students cannot see staff docs |

### Tradeoffs and gotchas

**Retrieval is a system; a prompt is a string.** Choosing RAG means chunking, embeddings, a store, a ranker, an eval set and a refresh job. That is the right cost for two hundred PDFs and an absurd cost for forty menu rows. Start with the cheapest route that could work and move up only when it demonstrably fails.

**Grounding without a refusal rule just produces better-dressed hallucinations.** Supply five chunks that do not contain the answer and a model with no permission to abstain will blend them into something confident. You have added cost and removed nothing.

**Retrieval must know who is asking.** This is the gap that turns a nice feature into an incident. If your search index holds staff-only circulars and supplier pricing alongside the student FAQ, a student's question can retrieve them and the model will cheerfully quote. Filter by the asker's permissions *before* search, not after — the same row-level thinking M04 taught, applied to your index.

**Stale is worse than absent.** An answer from last semester's prices looks exactly as confident as a correct one. Carry an "as of" timestamp with every grounded fact and show it, so a human can notice.

**Private data in a prompt is still private data leaving your building.** Grounding sends your menu, your circulars and sometimes your users' rows to a provider. Know what you are sending, strip what you do not need, and if something genuinely cannot leave, that is the argument for a local model — which is why this program keeps one in reach.

**Do not grind on prompt wording when the fact is missing.** Half the "the model keeps getting it wrong" problems in student projects are a fact that was never supplied. Check the context first, every time, before touching the prompt.

### Verify it

**Ask a question whose answer is not in your data, and insist on a refusal.** Your refund policy, if you have not written one. The hostel timings, if they are not in the index. An answer of any kind is a bug, and it is the single most diagnostic test in this chapter.

**Make it cite, then check the citation.** Open the cited document and confirm the sentence is actually there. A real citation to a real document that does not contain the claim is the most dangerous output in the whole module, because it looks verified.

**Change a fact and re-ask.** Edit one price in your source, then ask again. If the answer does not change, the model is answering from training or from a cache, not from your data, and you have been testing nothing.

**Ask the same question as two different users.** A student and a staff member. If the student's answer quotes a staff-only document, your retrieval is not permission-aware.

**Say "as of when?" out loud.** Every grounded answer should be able to tell you the age of the fact it used. If it cannot, you cannot detect staleness and neither can your users.

Red flags: a confident policy nobody in the team ever wrote; any factual answer with no citation; a citation pointing at a document that does not contain the claim; "we will fine-tune on our own data" offered as the fix for freshness; a vector store built for data that would fit in the prompt; retrieval that ignores who is asking; and no timestamp anywhere on grounded facts.

## Walkthrough
1. Write down ten real questions your users will ask your product. Use their actual words, including at least two in Hindi or a regional language.
2. For each, write where the true answer lives today: a database row, a document, somebody's head, or nowhere at all. Be honest about "nowhere" — those need writing before they need engineering.
3. Tag each question with the gap it exposes: private, current, domain-specific, or none of the three.
4. Route each one using the decision tree: small and stable goes in the prompt, structured and live gets a tool, large and unstructured gets retrieval. Write the route next to the question.
5. Count how many ended up in each bucket. If almost everything landed in retrieval, re-examine — that usually means structured data was mislabelled as documents.
6. Take your prompt-bucket facts and put them in the system prompt now, in the stable block from chapter six. Test those questions. Several will already be solved.
7. Take your tool-bucket facts and write the tool definitions, using chapter four's rules. Test those questions. Note the freshness: "as of" what time.
8. Add a refusal rule and one few-shot example showing an abstention, then ask three questions whose answers are genuinely not available anywhere. Record whether it refused or invented.
9. Add a citation requirement to the output format, then check one citation by opening the source and finding the sentence. Record whether it was really there.
10. Write down which questions still have no good route. That list is the scope of your retrieval work in the next four chapters, and it should be shorter than you expected.

## Talk to AI
```prompt
Help me decide how to ground each answer my product needs, and talk me out of over-engineering.

MY PRODUCT: [one sentence]
MY USERS AND THEIR PERMISSIONS: [who asks, and what each kind of user may and may not see]

REAL QUESTIONS USERS ASK - use their wording, include regional language ones:
1. [question]
2. [question]
3. [...]

WHERE MY DATA LIVES TODAY:
- Database tables: [names, row counts, how often each changes]
- Documents: [what kind, how many, how long, how often they change]
- Things that exist only in someone's head: [list them]

For each question, do all of this:

1. Name the gap: private, current, domain-specific, or none of the three.
2. Route it: put-it-in-the-prompt, give-it-a-tool, retrieve-it, or fine-tune. Justify in one line and estimate the token cost of your choice.
3. If you route it to retrieval, tell me what would have to be true for the prompt or a tool to be enough instead - I would rather not build a retrieval system I do not need.
4. State what the correct REFUSAL looks like when the fact is unavailable, in the user's words.
5. State what the citation should look like, and what a human would open to check it.
6. State what permission filter must apply before any search runs, given my user types.

Then across the whole product:
A. Which of my questions have no answer anywhere, and therefore need a human to write something before any AI work helps?
B. How much of my data would fit in a cached system prompt, with a token estimate?
C. What would I be fine-tuning for, if anything - and say plainly if the answer is nothing.
D. What private data would leave my building under your plan, and what should I strip before it does?
```

## Stuck? Ask AI
```prompt
Explain why fine-tuning a model on my company documents does not reliably teach it the facts in those documents. Use a concrete example of a price that changes. Then tell me what fine-tuning IS good for, and how I would know if my problem is one of those.
```

```prompt
I have a model, a database and a pile of PDFs. Explain the difference between putting facts in the prompt, giving the model a tool, and retrieving documents - what each costs, what each gets right, and what each gets wrong. Use one canteen example that needs all three.
```

```prompt
My AI feature gives a confident answer to a question my company has never written a policy about. Explain exactly why this happens in terms of next-token prediction, then give me the three changes - a rule, an example and a schema field - that would make it say "I do not know" instead.
```

## Quiz

### Q1 · L2
Your product needs today's sold-out dishes. Which route fits best?
- (a) Fine-tune nightly on the day's menu
- (b) Give the model a tool that reads the live stock row ✅
- (c) Embed the menu and retrieve it
- (d) Rely on the model's training data
> Structured data that changes belongs behind a tool, which reads the current row at the moment of asking. Embeddings add staleness and imprecision for no gain.

### Q2 · L3
A team wants their bot to know their forty-row menu. What is the right first move?
- (a) Build a vector store and retrieve the relevant rows
- (b) Put the menu in the system prompt, in the cacheable block ✅
- (c) Fine-tune on the menu
- (d) Add a search tool over the menu table
> Forty rows is a few hundred tokens: exact, cheap, cacheable and with no retrieval step to get wrong. Retrieval for data this small is usually less accurate, not more.

### Q3 · L4
Why is fine-tuning a poor way to teach a model your prices?
- (a) Fine-tuning requires too much data to be practical
- (b) Facts come out blurred, cannot be cited, and go stale until you retrain ✅
- (c) Providers do not allow numeric data in fine-tuning sets
- (d) Fine-tuned models cannot use tools
> Fine-tuning teaches shape, not retrievable fact. You cannot cite a weight or edit one, whereas changing a price should be one row in a table.

### Q4 · L4
You supply five retrieved chunks and none contains the answer. The model gives a confident answer anyway. What is missing?
- (a) A larger context window
- (b) More chunks
- (c) A refusal rule, an example of abstaining, and a schema field for it ✅
- (d) A reasoning model
> Grounding without permission to abstain produces better-dressed hallucinations. "I do not know" has to be stated as a rule, demonstrated, and representable in the output.

### Q5 · L5
A student asks about the refund policy and gets a correct answer citing a staff-only supplier circular. What has gone wrong?
- (a) Nothing; the answer was accurate
- (b) The citation format is wrong
- (c) Retrieval is not permission-aware, so it searched documents this user may not see ✅
- (d) The chunk size is too large
> Correctness is not the only requirement. The permission filter must be applied before the search runs, exactly as row-level security works for ordinary queries.

## Go deeper
- [Anthropic: Reduce hallucinations](https://docs.anthropic.com/en/docs/test-and-evaluate/strengthen-guardrails/reduce-hallucinations)
- [OpenAI: Fine-tuning versus retrieval, in the fine-tuning guide](https://platform.openai.com/docs/guides/fine-tuning)
