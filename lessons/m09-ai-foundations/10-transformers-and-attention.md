---
id: m09-10-transformers
module: m09-ai-foundations
slug: transformers-and-attention
title: "Transformers and Attention, Visually"
order: 10
duration_min: 15
is_free: false
mission: false
volatility: low
last_reviewed: 2026-11-16
---

## Key learnings
- Explain attention as every token deciding which other tokens matter to it
- Say why transformers beat RNNs: no forgetting across distance, and full parallelism
- Describe why a long prompt costs more than linearly and gets slower
- Explain why a model has no memory between requests, and what the chat window is really doing
- Catch an AI claiming the model remembers previous conversations or can explain its internals

## Lesson

### The pain

A team sets three rules at the top of their support assistant's conversation: reply in Marathi, never promise a refund, always ask for the order number. For six turns it is perfect; by turn twenty it answers in English and has promised a refund. They add "IMPORTANT" in capitals, which helps briefly. Another team finds that doubling the pasted document more than doubles the cost. Both effects come from the same mechanism.

### The idea

**Attention lets every token look at every other token in the input, score how relevant each one is to it, and build its own understanding from the ones that matter. A transformer is this operation, repeated in dozens of layers.**

Take "the sambar was too sour so the cook added more of it". Something must connect "it" back to "sambar" rather than "cook". Attention is that connection, computed rather than hand-written: processing "it", the model scores every earlier token and pulls in the meaning of the highest scorer.

Two things make this a revolution. Distance costs nothing — eight tokens away or eight hundred is one direct look, so nothing fades. And every token does it simultaneously, exactly the shape a GPU devours.

### The analogy

Twelve people at a long table argue about ordering one more plate of biryani. An RNN is a whispered chain: by person twelve the message is mangled, and it is slow, because nobody speaks until the person before has finished. Attention is different: all twelve at once look around and decide who is worth listening to — the one who knows the budget listens hardest to whoever mentioned money. Each builds a view from a weighted blend of the others in one parallel round, then does it again, dozens of times.

**Where the analogy breaks:** the twelve people remember yesterday's argument. The model does not. When the round ends everything is discarded, and the only reason a chat feels continuous is that the whole conversation is re-sent every time. That is the most practically important fact in this chapter.

### Picture it

```
  processing "it" in:
  "the sambar was too sour so the cook added more of it"

   the  sambar  was  too  sour  so  the  cook  added  more  of  [it]
    │     │      │    │     │    │    │    │     │      │    │    │
    └─────┼──────┴────┴─────┼────┴────┴────┼─────┴──────┴────┴────┘
          │                 │              │        all looked at,
       ████████          ███            ██          each scored
       strongest         some           a little
       attention

  "it" builds its meaning mostly from "sambar"
```

When the model processes "it", it scores every other token for relevance and draws most of its meaning from the highest-scoring one.

Three more pieces explain what you will notice. **Heads**: one round of attention tracks one kind of relationship, so each layer runs dozens side by side, and their roles emerge in training rather than being assigned. **Layers**: each layer's output feeds the next, local work early and abstract work later. **Positions**: attention alone is blind to order, so position is bolted on before the layers begin, which is why models are weak at "what was the fourth word".

### Where it lives

The transformer sits inside the box you call across the network. Three things leak out into your design.

- **The model is stateless.** Every request is complete and independent, so conversation memory is your application's job: you store the history and re-send it. When a chat product seems to remember you, something outside the model pushed that text back in.
- **Instructions at the top of a long context get diluted**, because attention spreads over everything present. The fix is restating the rule near the end, keeping the context short, or enforcing it in your own code.
- **Nothing inside is readable.** You cannot ask which head fired, and a fluent account of the model's own reasoning is more generated text, not a log.

### Words to own

| Term | Plain meaning | Example |
|---|---|---|
| Attention | Each token scoring how relevant every other one is | "it" finds "sambar" |
| Self-attention | Attention within one sequence | The usual kind |
| Head | One attention operation among many in a layer | Dozens per layer |
| Layer | One full round of attention plus processing | Dozens per model |
| Position encoding | Information telling the model the token order | Bolted on, not inherent |
| Transformer | The architecture built from stacked attention layers | Every modern LLM |
| Parallel | All tokens processed at once | Why GPUs suit it |
| Stateless | Nothing is remembered between requests | The key practical fact |
| KV cache | Saved attention state, so tokens are not recomputed | Grows with length |
| Prompt caching | A billing discount on an unchanged prompt prefix | M10 chapter 6 |
| Context | Everything the model can see in this one request | Chapter 12 |

### Tradeoffs and gotchas

- **"It forgot my instruction" is a context problem, not a model defect.** A bigger model rarely fixes it; shortening the context, restating the rule late, or validating in code does.
- **Long prompts cost more than their length suggests.** Every token attends to every other, so doubling the tokens roughly quadruples the comparisons: long prompts get slower and hit chapter 12's hard ceiling.
- **The attention state is cached, and the cache grows.** The **KV cache** avoids recomputing tokens, lives in GPU memory, and grows with the conversation — which is why providers offer **prompt caching**, a discount on an unchanged prompt prefix.
- **Longer context is not free capability.** Models are better at the beginning and end of a long input than the middle. Retrieving three paragraphs beats pasting eighty, which is the whole argument for RAG.
- **Repetition beats emphasis.** Capitals do little; stating the constraint twice, once near the end, does more, because position affects attention and shouting does not.
- **Do not design a feature around precise counting or positions.** Ordinary code does all of that perfectly, for free.

### Verify it

- **"The model remembers our earlier conversation."** It does not. Ask where that history is stored and who re-sends it.
- **A confident account of what happened inside the model.** The honest form is "the output suggests", not "the attention mechanism identified your intent".
- **"Use a bigger model" for an ignored instruction.** Ask how long the context was, where the instruction sat, and what would enforce the rule in code.
- **A plan that pastes a whole document in when three paragraphs would do.** Ask the cost at full length versus retrieved length.
- **Invented architecture details.** Head counts, layer counts and context limits for named models come out fluently and wrongly; the provider's model page is the authority.
- **Check yourself:** set a strict rule at the top of a real conversation, test it at turns two, ten and twenty, then rerun with the rule restated last.

## Walkthrough
1. Write one sentence containing a pronoun whose meaning depends on an earlier word — the sambar sentence or your own. Mark by hand which word the pronoun refers to.
2. Ask a model what the pronoun refers to and why. Then rewrite the sentence so the reference becomes genuinely ambiguous and ask again. Note how its answer changes.
3. Now set up a strict rule experiment. Start a conversation with three clear constraints — language, tone, and something it must always ask for.
4. Test all three constraints at turn two and record pass or fail for each.
5. Continue the conversation to roughly turn twenty with ordinary questions, then test the same three constraints again. Record exactly which failed and at roughly which turn.
6. Start a fresh conversation with the identical rules, but this time restate the constraints in your own final message on every turn. Run to twenty turns and test again. Compare the two results.
7. Paste a long document into a prompt and ask a question whose answer is in the middle of it. Then ask a question whose answer is in the last paragraph. Compare how well it did on each.
8. Measure the cost side: record the input token count for the full-document version, then for a version containing only the three relevant paragraphs. Write both numbers down.
9. Ask the model to explain which part of your prompt it based its answer on. Then judge honestly whether that explanation could be verified, and write one sentence on what it actually is.
10. Write one decision-log entry: where in your prompts your constraints will sit, what gets trimmed from history, and which rule you will enforce in code rather than trust the model with.

## Talk to AI
```prompt
I want to understand attention well enough to design prompts and estimate costs. Do not use mathematical notation and do not show me code.

Please work through all of this in plain language:

1. Take the sentence "the sambar was too sour so the cook added more of it". Walk me through what happens when the model processes the word "it": what it is comparing against, what it scores, and what it ends up using. Then do the same for a sentence where the reference is ambiguous and tell me what changes.

2. Explain why distance between two words costs attention nothing, and contrast that with how a recurrent model carrying a running summary would handle the same sentence.

3. Explain attention heads and layers using a concrete picture of many relationships being tracked at once. Be honest about how much of what heads do is human-interpretable.

4. Explain why attention is blind to word order on its own, and what is added to fix it. Then tell me which kinds of question this makes the model unreliable at, and what I should use instead for those.

5. Explain, in terms of pairwise comparisons, why doubling my prompt length more than doubles the work. Then tell me honestly which parts of that cost I actually see on a bill and which the provider has engineered away.

6. Explain why the model has no memory between requests, what the chat interface is really doing to create the illusion of memory, and what that means for what I am charged on turn twenty.

7. Finally: my assistant ignores an instruction I set at the top of a long conversation. Give me four possible fixes ranked by how well they actually work, and tell me which one I should use for a rule that must never be broken.
```

## Stuck? Ask AI
```prompt
Explain attention using an analogy about a group of people in a meeting deciding who to listen to. Then tell me three specific ways this analogy misleads me about how the real mechanism works, and ask me to guess one of them before you tell me.
```

```prompt
I do not understand why a language model has no memory between requests but a chat app feels like it remembers me. Explain exactly what is stored, by whom, and what is re-sent. Then tell me what I am billed for on turn twenty of a conversation and why.
```

```prompt
Explain the "lost in the middle" effect with a concrete example of a long document and a question whose answer is buried in the middle. Then tell me three design changes that handle it, and ask me which one fits a situation I will describe.
```

## Quiz

### Q1 · L2
What does attention let a token do?
- (a) Remember information from previous conversations
- (b) Look at every other token and score how relevant each one is to it ✅
- (c) Search the internet for related documents
- (d) Store itself in the model's weights for later
> Attention is a relevance computation inside one request. It is what connects "it" back to "sambar" without anyone writing a grammar rule.

### Q2 · L3
Which two problems of recurrent models did transformers solve at once?
- (a) Memory cost and training data volume
- (b) Forgetting across long distances, and being unable to run in parallel ✅
- (c) Tokenization and arithmetic accuracy
- (d) Overfitting and leakage
> Every token can look directly at every other, so distance costs nothing, and all tokens compute simultaneously — which is exactly the shape GPUs accelerate.

### Q3 · L4
Your assistant ignores a rule you set twenty turns ago. Which fix is most likely to actually work?
- (a) Writing the rule in capital letters with "IMPORTANT"
- (b) Switching to a model with more parameters
- (c) Restating the rule near the end of the prompt, shortening the context, and enforcing it in code ✅
- (d) Increasing the temperature so it pays more attention
> Attention spreads over everything present, so position and context length matter far more than emphasis. Anything that must never be broken belongs in your own code.

### Q4 · L4
Why does pasting an entire eighty-paragraph document perform worse than retrieving the three relevant paragraphs?
- (a) Long inputs are rejected by most providers
- (b) Models attend less reliably to the middle of a long input, and you pay for every token ✅
- (c) Documents must be embedded before they can be read
- (d) Attention cannot handle more than a few paragraphs at all
> This is the lost-in-the-middle effect, plus a straightforward token cost. Together they are the entire argument for retrieval over context-stuffing.

### Q5 · L5
A teammate says the model "remembers" that the user prefers Marathi from a conversation last week. What is actually required for that to be true?
- (a) Nothing; models retain user preferences automatically
- (b) The preference must be stored by your application and re-sent inside this request's context ✅
- (c) The model must be fine-tuned on that user's history
- (d) The KV cache must be kept alive between sessions
> Models are stateless. Any continuity is text your own system saved and pushed back in, which also means you pay for it in input tokens on every request.

## Go deeper
- [Jay Alammar: The Illustrated Transformer](https://jalammar.github.io/illustrated-transformer/)
- [Distill: Attention and augmented recurrent neural networks](https://distill.pub/2016/augmented-rnns/)
