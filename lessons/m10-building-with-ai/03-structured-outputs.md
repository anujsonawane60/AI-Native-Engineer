---
id: m10-03-structured-outputs
module: m10-building-with-ai
slug: structured-outputs
title: "Structured Outputs and JSON Schemas"
order: 3
duration_min: 15
is_free: false
mission: false
volatility: low
last_reviewed: 2026-11-16
---

## Key learnings
- Replace prose you have to parse with a schema the provider must satisfy
- Design fields that make a wrong answer impossible to express
- Give the model a legal way to say "I do not know"
- Validate every response in code, because a valid shape is not a true answer
- Spot the schema design mistakes that corrupt money and identifiers

## Lesson

### The pain

A team builds an order extractor. A student types "2 idli aur ek chai bhejo" and the model replies "Sure! Here's the order: 2 idli and 1 chai, total ₹70." Their code finds the numbers by splitting strings, and the failures are silent: "₹70" becomes 70 paise in a database that stores paise, and "ek chai" has no digit, so the chai vanishes at quantity zero. Then a reply arrives wrapped in a code fence and the parser throws for forty minutes. They asked for a sentence and treated it as data.

### The idea

**Stop asking for text you have to parse. Declare the exact shape you need as a schema, and have the provider guarantee the reply matches it.**

Most APIs now accept a JSON Schema and constrain generation so the output is valid against it. Your code receives an object with the fields and types you named, and a whole class of bug disappears with the parsing.

### The analogy

The canteen has two ways to take an order: scribble a note, or fill in the printed slip — dish, quantity box, table number, a veg tick. The slip cannot be filled in creatively: nowhere to write a greeting, a number in the quantity box, and a day's slips total up without interpreting handwriting.

**Where the analogy breaks:** a slip guarantees shape, never truth. A helper can tick "veg" on a chicken biryani and write 999 in the quantity box, and the slip is still perfectly valid. Schema validity means your code will not crash; it says nothing about whether the order is right.

### Picture it

```
  WITHOUT A SCHEMA
  "2 idli aur ek chai"
        │
        ▼
  "Sure! Here's the order: 2 idli and 1 chai, ₹70"
        │
        ▼  string splitting, regex, hope
  { items: [...], total: 70 }     <-- 70 paise. silent.

  WITH A SCHEMA
  "2 idli aur ek chai"  +  schema
        │
        ▼
  { "items": [ {"dish_id": "idli",  "qty": 2},
               {"dish_id": "chai",  "qty": 1} ],
    "unresolved": [], "needs_human": false }
        │
        ▼  validate, then your code does the money
```

With a schema the model fills in named fields and your code computes the total from your own prices. You will not write JSON Schema by hand — you will brief AI and read the result critically.

```
{
  "type": "object",
  "properties": {
    "items": {
      "type": "array",
      "items": {
        "type": "object",
        "properties": {
          "dish_id": { "enum": ["idli", "poha", "chai"] },
          "qty":     { "type": "integer",
                       "minimum": 1, "maximum": 20 }
        },
        "required": ["dish_id", "qty"]
      }
    },
    "unresolved":  { "type": "array",
                     "items": { "type": "string" } },
    "needs_human": { "type": "boolean" }
  },
  "required": ["items", "unresolved", "needs_human"]
}
```

Four design choices do all the work here.

- **`dish_id` is an enum, not a string.** The model cannot invent "paneer tikka" — there is no slot for it. The cheapest hallucination defence in this module.
- **`qty` is a bounded integer.** No decimals, no zero, no 999.
- **No `total` field.** Money is not the model's job; your code multiplies your own prices in paise.
- **`unresolved` and `needs_human` exist.** Without a legal way to be uncertain the model guesses, and a guess inside a valid schema looks exactly like a fact.

### Where it lives

The schema sits on the boundary between the model and your code, where M04 put API contracts — and it *is* a contract: the model promises a shape, your code validates it, and field names match your database columns. Chapter four is this mechanism reversed; chapter twelve's evals compare answers field by field, which only works because the answer has fields.

### Words to own

| Term | Plain meaning | Example |
|---|---|---|
| JSON Schema | A machine-readable description of a shape | The block above |
| Structured output | A reply constrained to match a schema | `{"qty": 2}` not a sentence |
| Enum | A closed list of allowed values | `["idli","poha","chai"]` |
| Required | A field that must be present | `needs_human` |
| Nullable | May be present but empty | `table_no: null` |
| Constrained decoding | The provider blocking invalid tokens | Cannot emit a stray word |
| Validation | Your code re-checking the shape | Reject and retry once |
| Escape hatch | A legal way to express uncertainty | `needs_human: true` |
| Paise integer | Money as whole paise, never a float | `7000` means ₹70 |
| Round-trip | Feeding the output back as input | Checks nothing was lost |

### Tradeoffs and gotchas

**A valid shape is not a true answer.** Constrained decoding removes parse errors and invented field names, not wrong quantities, mislabelled items or fabricated identifiers.

**Never let a model produce an identifier or an amount.** It will produce something that looks perfect and refers to nothing. It may *select* an id from a list you supplied; never mint one. Money stays integer paise in your code.

**Deep nesting degrades quality.** Thirty fields five levels deep fill in worse than two flat calls; split it.

**A free-text field is still free text.** `notes: string` is an open door: what the user wrote may later be read by an agent as an instruction. Prefer enums, bound your strings, never concatenate it into a later prompt unmarked.

**Strict mode is not always available.** On some free tiers you get "please reply in JSON" rather than a guarantee. Validate anyway, retry once with the validation error appended, then fail loudly.

### Verify it

- **Validate every response in code before it touches your database**, and log failures with the raw text. You should be able to quote your failure rate.
- **Read the schema for fields that should not exist** — `total`, `price`, `order_id`. Anything the model invents that your code already knows is a future wrong number.
- **Check the escape hatch fires.** Send "ek special dena" and "do plate". If `needs_human` stays false, the model is guessing.
- **Count what lands in the catch-all, weekly.** If `unresolved` absorbs a tenth of your traffic your enum is stale — generate it from the database at request time.
- **Round-trip one example.** Render the output back into a sentence and compare it with what the student typed.

Red flags: code that splits strings on model output; money as a float or in rupees; a model-generated identifier used as a key; a hand-typed enum; no `needs_human` equivalent; a schema validated only in testing; a free-text field concatenated into a later prompt.

## Walkthrough
1. Pick one AI call in your product whose output your code consumes. Write down, as a list of field names and types, exactly what your code needs from it.
2. Write the three things your code must never receive: a total it should compute itself, an identifier the model would be inventing, and a value outside a known list.
3. Brief AI to produce a JSON Schema for that output, stating explicitly that money is excluded, identifiers may only be chosen from a supplied list, and every category must be an enum.
4. Read the schema it produced and delete every field your code does not actually use. Each extra field is something the model must fill in, and filling in more things makes everything slightly worse.
5. Replace every free-text category field with an enum. Generate the enum values from your own database rows rather than typing them in, so they cannot drift.
6. Add an escape hatch: an array for things it could not resolve and a boolean for needs-a-human. Make both required so they cannot be silently omitted.
7. Run the call with ten real inputs, including two in Hindi or Hinglish, one with no digits, one asking for something not on the menu, and one deliberately ambiguous.
8. Validate each response against the schema in your own code even if the provider claims to guarantee it. Record the pass rate and keep the raw text of every failure.
9. Add a single retry that appends the validation error to the request, and confirm that a second failure raises an error rather than writing a partial row.
10. Have your code compute the money from the structured items and your own prices, in paise. Check one order by hand against a calculator, including a case that needs rounding.

## Talk to AI
```prompt
Design the structured output schema for one AI call in my product, and then stress-test your own design.

WHAT THE CALL DOES: [one sentence]
WHAT MY CODE DOES WITH THE RESULT: [be specific - which table, which columns]
MY EXISTING DATA SHAPE: [paste your table columns and types, or describe them]
REAL INPUTS MY USERS SEND: [paste 5 real or realistic examples, including at least two in Hindi or Hinglish]

Rules you must follow:
- Money is never a model output. My code computes amounts from my own prices, in integer paise.
- The model may never invent an identifier. It may only choose one from a list I supply.
- Every category field must be an enum, and you must tell me where the enum values should be generated from at runtime.
- The schema must include a legal way for the model to express uncertainty rather than guessing.
- Keep it flat. If it needs more than two levels of nesting, tell me to split it into two calls instead.

Deliver, in this order:
1. The JSON Schema.
2. A table of every field: name, type, why it exists, and what wrong answer it makes impossible to express.
3. Every field you considered and deliberately left out, with the reason.
4. The exact structured output you would produce for each of my 5 example inputs.
5. Three inputs that would most likely defeat this schema, and what the output would be.
6. What my code must still validate even though the provider constrains the shape - and what a valid-but-wrong response would look like here.
```

## Stuck? Ask AI
```prompt
Explain the difference between a response that is VALID against a schema and a response that is CORRECT, using one concrete example of a canteen order where the shape is perfect and the content is wrong. Then tell me what I have to check in my own code, given that the provider already guarantees the shape.
```

```prompt
Here is a JSON schema an AI wrote for me: [paste it]. Review it as if you had to maintain the code that consumes it. Which fields is the model being asked to invent rather than report? Which free-text fields should be enums? Where can it not express uncertainty? What breaks the first time my menu changes?
```

```prompt
Explain why storing money as 34.50 in a floating point number causes real bugs, with a worked example of three students splitting a bill. Then show me what the same calculation looks like in integer paise, and tell me where the rounding decision has to be made and by whom.
```

## Quiz

### Q1 · L2
What does constrained decoding against a JSON Schema guarantee?
- (a) That the content of the answer is correct
- (b) That the reply has the fields and types you declared ✅
- (c) That the model will not hallucinate
- (d) That the response costs fewer tokens
> It guarantees shape, not truth. Your code will not crash parsing it, and it can still contain a wrong quantity or a mislabelled dish.

### Q2 · L3
Why should the extraction schema for an order have no `total` field?
- (a) Totals make the schema too deeply nested
- (b) Your code should compute money from your own prices in paise ✅
- (c) Providers do not allow numeric fields
- (d) Totals change too often to be cached
> Never ask a model for a number you already know how to compute. The prices are yours, the arithmetic is deterministic, and a model-supplied total is an unverified invention.

### Q3 · L4
Your schema has no field for uncertainty. A student types "ek special dena". What happens?
- (a) The call fails schema validation
- (b) The model returns an empty object
- (c) The model guesses, and the guess looks exactly like a fact ✅
- (d) The provider retries automatically
> With no legal way to be uncertain, the model must fill the required fields. A guess inside a valid schema is indistinguishable from a confident correct answer.

### Q4 · L4
Which schema choice most reduces hallucinated menu items?
- (a) Lowering the temperature
- (b) Adding "do not invent dishes" to the system prompt
- (c) Making `dish_id` an enum generated from your menu table ✅
- (d) Increasing the maximum output length
> If there is no slot for a dish you do not sell, it cannot be emitted. A generated enum is structural, so it also cannot drift away from your real menu.

### Q5 · L5
Your provider's free tier does not offer a hard schema guarantee — it only asks nicely for JSON. What is the right design?
- (a) Trust it; modern models reply in JSON reliably
- (b) Parse with regex as a fallback when JSON fails
- (c) Validate in your code, retry once with the error, then fail loudly ✅
- (d) Switch the field types to strings so anything parses
> Validation belongs in your code whether or not the provider constrains output. One retry with the validation error appended fixes most failures; a second failure must raise rather than write a partial row.

## Go deeper
- [OpenAI: Structured outputs](https://platform.openai.com/docs/guides/structured-outputs)
- [Understanding JSON Schema](https://json-schema.org/understanding-json-schema)
