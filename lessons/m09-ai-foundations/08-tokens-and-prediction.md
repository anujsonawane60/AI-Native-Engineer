---
id: m09-08-tokens
module: m09-ai-foundations
slug: tokens-and-prediction
title: "Tokens, Tokenization and Next-Token Prediction"
order: 8
duration_min: 25
is_free: false
mission: false
volatility: low
last_reviewed: 2026-11-16
---

## Key learnings
- Explain what a token is and why models count them instead of words
- Estimate the token count of a piece of text well enough to estimate a bill
- Describe next-token prediction as the one thing the model does
- Say why Marathi or Hindi text can cost two or three times more than the same meaning in English
- Catch an AI quoting token counts, prices or context sizes without checking a tokenizer

## Lesson

### The pain

A team ships an "ask our canteen" feature. It works beautifully in testing. Two weeks later their free credits are gone and nobody can explain where.

They investigate. Three things are happening. The feature was built for Marathi-speaking students, and the same question in Marathi consumes roughly two to three times as many tokens as in English, so every request costs more than the English tests suggested. The system prompt is 700 tokens of careful instructions and it is re-sent on every single turn of every conversation. And the chat keeps the full history, so by turn ten each message is re-paying for all nine earlier ones.

None of this is a bug. All of it is a direct consequence of how these models consume text, and every one of those three facts is invisible unless you know what a token is. The team was billed in a currency they had never looked at.

### The idea

**A model does not see letters or words. Text is chopped into tokens, each token becomes a number, and the model's one and only ability is to predict which token comes next. It then appends that token and does it again.**

That is genuinely all of it. "Write me a poem" produces a poem one token at a time, each token chosen with the whole preceding text in view. There is no plan, no outline held in reserve, no sentence assembled before it starts. The fluency is a side effect of doing this extremely well, at enormous scale.

A token is roughly a short word or a chunk of one. In English, a useful rule of thumb is about four characters per token, or about three tokens for every four words. "canteen" is probably one token. "unavailability" is probably three. A space usually travels attached to the front of the following word.

### The analogy

The canteen runs on order slips, and the slips use codes rather than sentences.

The kitchen has a fixed printed codebook: `TH` for thali, `BIR` for biryani, `XC` for extra curd. A cashier taking an order does not write "one thali with extra curd" — they write `TH` `XC`, because the kitchen only understands codes from the book. That codebook is the **vocabulary**, the act of translating an order into codes is **tokenization**, and the codes themselves are **tokens**.

The codebook was printed years ago by someone who studied the commonest orders. Popular items got their own short code. Rare items did not, so "Kashmiri dum aloo" has to be written as four separate codes cobbled together from fragments. The slip gets longer, and since the kitchen charges per code, the rare dish costs more to order even though it is one dish.

**Where the analogy breaks:** the cashier knows what `TH` means. The model has no codebook meanings at all — it only ever saw which codes follow which other codes, across trillions of slips. And unlike a canteen, you cannot ask for a new code to be added; the vocabulary was fixed when the model was trained, and whoever fixed it was probably optimising for English.

### Picture it

```
  "Today's thali is ready"

  TOKENIZE
  ┌────────┬────────┬────┬───────┬────┬────────┐
  │ Today  │ 's     │ th │ ali   │ is │ ready  │
  └────────┴────────┴────┴───────┴────┴────────┘
  each becomes a number:  1842  596  339  8810  318  5527

  NEXT-TOKEN PREDICTION, one step
  all tokens so far ──▶ model ──▶ a score for EVERY token
                                   in the vocabulary
                                       │
                                       ▼
                                  pick one ("for")
                                       │
                                       ▼
                            append it and run again
```

Text is split into tokens and turned into numbers; the model scores every possible next token, one is picked, appended, and the whole thing runs again.

### What this explains

Once you hold "it predicts the next token", a long list of confusing behaviour becomes obvious rather than mysterious.

**Why it cannot count the letters in a word.** It never saw letters. "strawberry" may arrive as two or three tokens, and counting the r's inside a token is not an operation it has. Asking a language model to do character-level work is asking it to inspect something it cannot see.

**Why arithmetic is shaky.** "1,247" might be three tokens. There is no calculator underneath — it is producing the token sequence that most plausibly follows, which for common sums is reliably correct and for unusual ones is confidently wrong. Chapter 4's point, from the other direction.

**Why it is confidently wrong rather than silent.** At every step some token is the most plausible continuation. There is no path through the machinery that produces "I do not know" unless that phrase was itself the plausible continuation. This is M00's first failure mode, explained at the mechanism level.

**Why output arrives word by word.** Streaming is not a user-interface flourish. Each token genuinely has to be computed after the one before it, so generating 500 tokens takes roughly 500 times as long as generating one. Input is different: the whole prompt is read in parallel, which is why long prompts are cheap to read and long answers are slow to write.

**Why non-English text costs more.** The vocabulary was built mostly from English text, so English words tend to be single tokens while Devanagari words get fragmented into several. The same question in Marathi or Hindi can cost two to three times the tokens of its English equivalent. For an Indian student product this is not a curiosity — it is a line item, and it is the clearest case in this course of a technical default having an economic consequence for your users.

### Where it lives

Tokens are the unit of three different things, which is why this short chapter matters so much.

**They are the unit of billing.** Providers price per million input tokens and per million output tokens, with output usually several times dearer. That is chapter 14, and it is where the team from the top of this chapter needed to be before they shipped.

**They are the unit of the context window.** The limit on how much a model can consider at once is a token count, not a word count or a character count. Chapter 12.

**They are the unit of latency.** Time to first token depends mostly on your input length; total time depends mostly on your output length. That distinction is the one that tells you whether to shorten your prompt or cap your answer, and it connects straight to M05's latency work.

Every provider publishes a tokenizer or a token-counting endpoint. Using it on your own real text, rather than estimating, is the habit this chapter is trying to install.

### Words to own

| Term | Plain meaning | Example |
|---|---|---|
| Token | A chunk of text, roughly a short word | "th" + "ali" |
| Tokenizer | The thing that chops text into tokens | Fixed at training time |
| Vocabulary | The complete fixed list of possible tokens | Often ~100k entries |
| Token id | The number a token maps to | 8810 |
| Subword | A token that is part of a word | "ali" |
| Input tokens | What you send | Prompt plus history |
| Output tokens | What it generates | Usually priced higher |
| Autoregressive | Each token predicted from all previous ones | One at a time |
| Sampling | Choosing which scored token to actually use | Chapter 12 |
| Streaming | Sending tokens to the user as they appear | Why text trickles in |
| Time to first token | How long before anything appears | Driven by input length |

### Tradeoffs and gotchas

**Estimate with the rule of thumb, bill with the tokenizer.** Four characters per token is good enough for a rough plan and wrong enough to matter for a budget. Before you commit to a cost model, run your actual prompts — including your real Marathi or Hindi examples — through the provider's own counter.

**Your system prompt is charged on every turn.** A 700-token instruction block in a ten-turn conversation is 7,000 input tokens of pure overhead. This is the most common avoidable cost in a student AI feature, and M10's prompt caching is the proper answer to it.

**Conversation history grows faster than you expect.** Each turn re-sends everything before it, so the cost of a long chat grows much faster than the number of turns. Ten turns is not ten times one turn; it is closer to fifty.

**Output tokens are the expensive ones.** They are priced higher and they are slow, because each must wait for the one before. Capping maximum output length is simultaneously a cost control, a latency control and a quality control, since an uncapped model will pad.

**Token counts differ between providers.** Each family has its own tokenizer, so the same text is a different number of tokens on different models. You cannot compare two providers' prices without converting through each one's own count — which chapter 13 will make you do properly.

### Verify it

**Red flag: a token count asserted rather than measured.** If an AI tells you your prompt is "about 300 tokens", treat that as a guess. Run it through the provider's tokenizer. For non-English text the guess will usually be badly low, which is exactly where it costs you money.

**Red flag: a price quoted confidently.** Model prices change, get cut, get restructured, and sit well behind any model's knowledge cutoff. A price from a chat window is a rumour. The provider's own pricing page is the only authority, and chapters 13 and 14 return to this because it is the one that costs real rupees.

**Red flag: "one word is one token".** A plausible simplification that breaks on exactly the cases you care about: Indian languages, code, long technical words, numbers, and emoji.

**Red flag: a language model proposed for character-level or arithmetic work.** Counting letters, reversing strings, checking digits, summing a column — ask for ordinary code instead. It will be correct, free and instant.

**Red flag: a cost estimate with no output-token line.** If the plan prices only what you send, it has missed the dearer half.

The check you can run in ten minutes, and should run before any AI feature ships: take five real user messages, including at least two in the language your users actually type in, count their tokens with the provider's tool, add your system prompt's token count, make one real call and record the input and output token counts the response reports back. You now have a measured number per request instead of a hope, and chapter 14 turns it into a monthly figure.

## Walkthrough
1. Open any free tokenizer playground and paste one English sentence. Record the token count and look at where the splits fell. Note any word that broke into pieces.
2. Paste the same sentence translated into Marathi or Hindi. Record the token count. Write down the ratio between the two.
3. Now paste a paragraph of your own product's real text — a support message, a menu description. Record characters, words and tokens, and work out your own characters-per-token figure.
4. Paste a block of code and a string of digits. Note how differently they tokenize from prose, and write one sentence on what that means for any feature that handles numbers.
5. Count the tokens in your system prompt, if you have one. Multiply by ten turns and write the total down. React to that number.
6. Make one real API call with a short prompt and read the token counts the provider returns in the response. Compare them against your estimate from the playground.
7. Ask a model to count the letter "r" in "strawberry" and then to count a longer word of your choosing. Record whether it is right, and explain why using the word "token".
8. Ask the same model for the current price per million input and output tokens for the model you just called. Then open the provider's pricing page and compare. Record both and note the date.
9. Finally, write down your measured tokens per request for one real feature — input and output separately — and keep it. Chapter 14 starts from this number.

## Talk to AI
```prompt
I want to understand how my text becomes tokens and what that costs me, and I want measurements rather than reassurance.

Here is a real piece of text from my product: [paste 3-4 sentences in English]
Here is the same thing in the language my users actually type: [paste the Marathi/Hindi/other version]
Here is my system prompt: [paste it, or say "none yet"]

Please do all of this:

1. Explain what a token is, using my English text as the example, and show me roughly where you would expect the splits to fall - including any word you expect to break into pieces.

2. Estimate the token count for both versions of my text and tell me the ratio between them. Then tell me explicitly that this is an estimate, and name the exact tool I should use to get the real number for the specific model I am using.

3. Explain why the non-English version costs more, in terms of how the vocabulary was built. Tell me what, if anything, I can actually do about it.

4. Work out my real cost per request structure in words, not just numbers: system prompt tokens, user message tokens, retrieved context tokens, conversation history tokens, output tokens. Tell me which of these are re-sent on every turn.

5. Tell me what a ten-turn conversation costs relative to a one-turn conversation, and explain why it is not ten times.

6. IMPORTANT: do not quote me any price per million tokens. Instead, tell me exactly which page on which provider's site I must open to get the current price, and warn me about why any price you know could be out of date.

7. Finally, name three things in my feature that I should ask a language model NOT to do because tokens make it unreliable, and say what to use instead.
```

## Stuck? Ask AI
```prompt
Explain tokenization using an analogy about a shorthand notebook, with no technical vocabulary. Then give me six short strings - an English word, a Hindi word, a long number, an email address, a code snippet and an emoji - and ask me to guess how many tokens each becomes before you tell me.
```

```prompt
Explain why a language model cannot reliably count the letters in a word or add two large numbers. Be specific about what it actually sees instead of characters. Then give me five tasks and ask me which ones I should give to ordinary code rather than to a model.
```

```prompt
Walk me through next-token prediction one step at a time for the sentence "the canteen closes at". Show me what the model is scoring at each step, how one token gets chosen, and what happens immediately afterwards. Then explain why this means the model can never answer "I do not know" unless that was itself a plausible continuation.
```

## Quiz

### Q1 · L2
What is a token?
- (a) One word of text
- (b) A chunk of text, often part of a word, drawn from a fixed vocabulary ✅
- (c) One character of text
- (d) One sentence sent to the model
> Tokens are subword chunks from a vocabulary fixed when the model was trained. "thali" may well arrive as two tokens, and a space usually travels with the following word.

### Q2 · L3
The same question in Marathi costs noticeably more than in English. Why?
- (a) Marathi sentences are always longer
- (b) The vocabulary was built mostly from English, so Devanagari fragments into more tokens ✅
- (c) Providers charge a surcharge for non-English requests
- (d) The model translates internally, doubling the cost
> Tokenizers are fit to their training text. Words common in that text get single tokens; scripts under-represented in it get chopped into several pieces per word.

### Q3 · L3
Why is generating a 500-token answer much slower than reading a 500-token prompt?
- (a) Output tokens are larger than input tokens
- (b) Each output token must be computed after the previous one, while input is read in parallel ✅
- (c) The provider throttles output deliberately
- (d) Output is encrypted and input is not
> Generation is autoregressive and therefore sequential. This is also why time to first token tracks your input length while total time tracks your output length.

### Q4 · L4
An AI tells you your prompt is "about 250 tokens" and that the model costs a specific amount per million tokens. How should you treat these two claims?
- (a) Both are reliable technical facts
- (b) Both are unverified: measure the count with a tokenizer and read the price on the provider's page ✅
- (c) The token count is reliable but the price is not
- (d) The price is reliable but the token count is not
> Token counts must be measured per model, and prices sit behind every knowledge cutoff. Neither belongs in a budget until you have checked it at the source.

### Q5 · L5
Your chat feature sends a 700-token system prompt plus the full conversation history on every turn. What is the most accurate description of the cost at turn ten?
- (a) Ten times the cost of turn one
- (b) The same as turn one, since the model remembers the earlier turns
- (c) Far more than ten times, because every turn re-pays for all previous turns plus the system prompt ✅
- (d) Lower than turn one, because later answers are shorter
> Models are stateless, so each turn re-sends everything. History grows with each turn and the system prompt is charged every time, which is why prompt caching and history trimming exist.

## Go deeper
- [OpenAI: what are tokens and how to count them](https://platform.openai.com/tokenizer)
- [Hugging Face: summary of tokenizers](https://huggingface.co/docs/transformers/en/tokenizer_summary)
