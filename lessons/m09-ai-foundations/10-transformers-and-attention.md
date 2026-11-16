---
id: m09-10-transformers
module: m09-ai-foundations
slug: transformers-and-attention
title: "Transformers and Attention, Visually"
order: 10
duration_min: 25
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

A team builds a canteen support assistant. At the top of the conversation they set the rules: always reply in Marathi, never promise a refund, always ask for the order number.

For six turns it is perfect. By turn fifteen it is answering in English. By turn twenty it has promised a refund. They add "IMPORTANT" in capitals. It helps for a while and then stops helping. Someone suggests the model is "getting tired".

Separately, another team notices that pasting a long document into the prompt makes the assistant much slower and much more expensive than the length alone would suggest. Doubling the document more than doubles the cost, which nobody predicted.

Both effects come from the same mechanism. Understanding attention is what turns these from mysteries into things you can design around.

### The idea

**Attention lets every token look at every other token in the input, score how relevant each one is to it, and build its own understanding from the ones that matter. A transformer is this operation, repeated in dozens of layers.**

Take "the sambar was too sour so the cook added more of it". For the model to handle "it", something has to connect "it" back to "sambar" rather than to "cook" or "sour". Attention is that connection, computed rather than hand-written: when processing "it", the model scores every earlier token for relevance, finds "sambar" scores highest, and pulls in that meaning.

Two things make this a revolution rather than a refinement. The distance between "it" and "sambar" costs nothing — eight tokens away or eight hundred, it is one direct look, so nothing fades. And every token does this simultaneously, which means the whole operation is exactly the thousands-of-independent-things shape that a GPU devours, as chapter 7 described.

### The analogy

Picture a long table in the canteen with twelve people arguing about whether to order one more plate of biryani.

An RNN is a whispered chain: person one whispers to person two, who whispers to person three, and by person twelve the message has been mangled and the original point is lost. It is also slow, because nobody can speak until the person before them has finished.

Attention is different. Every one of the twelve, at the same moment, looks around the table and decides who is worth listening to. The person who knows the budget listens hardest to whoever mentioned money. The person who is hungry listens hardest to whoever mentioned biryani. Each builds their own view from a weighted blend of everyone else's, all in one round, in parallel. Then they do it again, now informed by the first round — and again, dozens of times.

**Where the analogy breaks:** the twelve people remember yesterday's argument. The model does not. When the round ends, everything is discarded. The next question arrives at a table of twelve people with no memory whatsoever, and the only reason a chat feels continuous is that the whole conversation is re-sent, every single time. There is nothing else there. That is the most practically important fact in this chapter.

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

### Heads, layers, and positions

Three more pieces, each of which explains something you will notice in practice.

**Heads.** One round of attention can only track one kind of relationship well. So each layer runs many attention operations side by side — commonly dozens — called **heads**. One may end up tracking grammatical subjects, another tracking which noun a pronoun refers to, another tracking tone. Nobody assigns these roles; they emerge in training, and most heads do nothing a human could name. This is why the model can keep several kinds of relationship straight at once.

**Layers.** The output of one layer becomes the input to the next, and a large model stacks dozens. Early layers do local, shallow work; later layers do abstract work. This stacking is where "understanding" accumulates, and it is also why a bigger model costs more per token: each token passes through every layer.

**Positions.** Attention by itself is blind to order — "cook added sambar" and "sambar added cook" would look identical. So position information is added to each token before the layers begin. This is a bolted-on fix rather than something inherent, and it is the reason models are oddly weak at very precise positional questions, like "what was the fourth word" or "exactly how many items are in this list".

### Why long prompts cost more than you expect

Here is the practical consequence that will show up on your bill.

If every token attends to every other token, then doubling the number of tokens roughly quadruples the number of pairwise comparisons. Ten times the tokens is roughly a hundred times the attention work. The cost does not rise in step with length; it rises much faster.

Providers have engineered around a great deal of this, and the price you are charged per input token is usually flat, so you will not see a squared term on your invoice. What you will see is that very long prompts get slower, that the effective cost of stuffing the whole conversation in grows unpleasantly, and that there is a hard ceiling — the context window of chapter 12.

There is also a memory consequence with a name you will meet in M10. To avoid recomputing everything for each new token, the model caches its intermediate attention state — the **KV cache**. That cache grows with the length of the conversation and lives in GPU memory, which is why long conversations are expensive to serve and why providers offer **prompt caching**, a billing discount for the part of your prompt that has not changed. M10 chapter 6 is entirely about turning that into a cost saving, and the reason it works is this paragraph.

### Where it lives

On your architecture map, the transformer is inside the box you call across the network. You will never touch it. What leaks out of it into your design decisions is a short list, and that list is why the chapter exists.

**The model is stateless.** Every request is complete and independent. Conversation memory is your application's job — you store the history and re-send it, which is M04's data problem and chapter 14's cost problem. When a chat product appears to remember you across sessions, something outside the model saved that text and pushed it back in.

**Instructions at the top of a long context get diluted.** Attention spreads over everything present, so a rule set twenty turns ago is competing with twenty turns of conversation. The fix is not shouting; it is restating the constraint near the end of the prompt, keeping the context short, or enforcing the rule in your own code instead of hoping. M10's context engineering chapter is this problem treated properly.

**Nothing inside is readable.** You cannot ask which head fired. A model can produce a fluent account of its own reasoning, and that account is more generated text — a plausible story, not a log. This matters for chapter 16 and for any product where you have to justify a decision.

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

**"It forgot my instruction" is a context problem, not a model defect.** Switching to a bigger model rarely fixes it. Shortening the context, restating the rule late, or validating in your own code does. This is M00's "model or harness?" question in a new form, and the answer here is usually neither — it is your prompt design.

**Longer context is not free capability.** Models are measurably better at the beginning and end of a long input than at its middle, an effect usually called lost-in-the-middle. Burying the one important paragraph at position 40 of 80 is a reliable way to have it ignored. Retrieving three relevant paragraphs beats pasting eighty, which is the entire argument for RAG in M10.

**Repetition beats emphasis.** Capital letters and "IMPORTANT" do very little. Stating the constraint twice, once near the end, does more, because position affects attention and shouting does not.

**You are paying for history you may not need.** If a support conversation does not actually need turn two to answer turn twenty, trim it. Teams re-send everything by default and are surprised by the bill, which is the mechanism from chapter 8 and the arithmetic of chapter 14.

**Do not design a feature around precise counting or positions.** "How many items are in this list", "what is the fourth bullet", "is this exactly 200 words" — positional precision is a known weak spot, and ordinary code does all of it perfectly for free.

### Verify it

**Red flag: "the model remembers our earlier conversation".** It does not. Ask where that history is stored and who re-sends it. If nobody can answer, the feature is relying on something that does not exist.

**Red flag: a confident account of what happened inside the model.** "The attention mechanism identified your intent and routed it to..." is storytelling. The honest form is "the output suggests", and anyone claiming to read a large model's internal reasoning from the outside is overclaiming.

**Red flag: "use a bigger model" offered for an ignored instruction.** Ask instead how long the context was, where in the prompt the instruction sat, and whether it is restated near the end. Then ask what would enforce the rule in code if the model ignored it anyway — because for anything that matters, something must.

**Red flag: a plan that pastes an entire document in when three paragraphs would do.** Ask what it costs per request at full length versus retrieved length, and point at lost-in-the-middle. This is the single highest-value habit from this chapter.

**Red flag: invented architecture details.** Head counts, layer counts and context limits for named models are exactly the facts an AI produces fluently and wrongly. The provider's model page is the authority.

And the check to run yourself, in twenty minutes: take one real conversation, set a strict rule at the top, and run it out to twenty turns, testing the rule at turns two, ten and twenty. Record where it starts failing. Then run the same conversation again with the rule restated in the last message each turn, and record the difference. That one experiment teaches more about prompt position than any explanation, and it is the measurement you will reuse all through M10.

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
