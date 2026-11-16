---
id: m10-03-structured-outputs
module: m10-building-with-ai
slug: structured-outputs
title: "Structured Outputs and JSON Schemas"
order: 3
duration_min: 25
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

A team builds an order extractor. A student types "2 idli aur ek chai bhejo", and the model replies: "Sure! Here's the order: 2 idli and 1 chai, total ₹70."

Their code finds the numbers with string splitting. It works most of the time, and the failures are the worst kind — silent. "₹70" is read as 70, but the database stores paise, so a ₹70 order is saved as 70 paise. "ek chai" has no digit, so quantity becomes zero and the chai disappears. One evening the model wraps its answer in a code fence and the parser throws an exception on every order for forty minutes before anyone notices.

Not one of these is a model failure. The model answered the question. The team asked for a sentence and then treated it as a data structure.

### The idea

**Stop asking for text you have to parse. Declare the exact shape you need as a schema, and have the provider guarantee the reply matches it.**

Most APIs now accept a JSON Schema and constrain generation so the output is valid against it. Your code receives an object with the fields you named, of the types you named. The parsing problem disappears, and with it an entire class of bug.

### The analogy

The canteen has two ways to take an order. A helper can scribble a note, or they can fill in the printed slip: dish name, quantity box, table number, a tick for veg or non-veg, and a signature.

The slip cannot be filled in creatively. There is nowhere to write a greeting. The quantity box takes a number. The veg tick is one of two boxes. Anyone collecting slips at the end of the day can total them without interpreting anybody's handwriting.

**Where the analogy breaks:** a slip guarantees shape, never truth. A helper can tick "veg" on a chicken biryani and write 999 in the quantity box, and the slip is still perfectly valid. Schema validity means your code will not crash. It says nothing at all about whether the order is right. That distinction is the single most misunderstood thing about structured outputs, and it is what the Verify it section is for.

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

With a schema the model fills in named fields and your own code computes the total from your own prices. Without one, you are parsing a sentence and trusting a number the model invented.

### A small schema, read not written

You will not write JSON Schema by hand — you will brief AI to produce it and then read it critically. This is what you are reading for.

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

Four design choices are doing all the work here, and each one makes a specific wrong answer impossible to express.

**`dish_id` is an enum, not a string.** The model cannot invent "paneer tikka" because there is no slot for it. An enum built from your actual menu table is the cheapest hallucination defence in this entire module.

**`qty` is a bounded integer.** No decimals, no zero, no 999. The order that broke their database cannot be represented.

**There is no `total` field.** Money is not the model's job. Your code multiplies your prices by the quantities, in paise, as integers. Never ask a model for an amount you can compute yourself.

**`unresolved` and `needs_human` exist.** "Ek special dena" goes into `unresolved` and sets `needs_human` to true. Without these, the model has no legal way to be uncertain, so it will guess — and a guess inside a valid schema looks exactly like a fact.

### Where it lives

The schema sits exactly on the boundary between the model and your code, which is the same place M04 put API contracts. It *is* a contract: the model promises a shape, your code promises to validate it. Field names should match your database columns, so nothing needs translating on the way in.

Chapter four is this same mechanism pointed the other way: a tool definition is a schema describing the arguments a model may send to one of your functions. Chapter twelve's eval sets compare structured answers field by field, which is only possible because the answer has fields. And in chapter nineteen, your schema file is one of the things a linter can check automatically.

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

**A valid shape is not a true answer.** This bears repeating because teams genuinely conflate the two. Constrained decoding removes parse errors and invented field names. It does not remove wrong quantities, mislabelled items or confidently fabricated identifiers.

**Never let a model produce an identifier.** Not an order id, not a user id, not a payment reference. It will produce something that looks perfect and refers to nothing, and your foreign key will fail at the worst moment. The model may *select* an id from a list you supplied; it may never mint one.

**Money as a float is a bug waiting for a demo.** `34.50` cannot be stored exactly in binary floating point, and a three-way split of ₹340 will drift. Integers in paise, always, and the arithmetic in your code — which is where M04's payment chapter and chapter thirteen of this module meet.

**Deep nesting degrades quality.** A schema five levels deep with thirty fields gets filled in worse than two flat calls with eight fields each. If the model has to populate everything to populate anything, split the call.

**A free-text field is still free text.** `notes: string` inside a beautiful schema is an open door: whatever the user wrote arrives in your system and may be read later by an agent as an instruction. Chapter seven is about that, and the mitigation starts here — prefer enums, bound your strings, and never concatenate a free-text field into a later prompt without marking it as data.

**Enums drift.** You add a dish to the menu and forget the enum, so real orders land in `unresolved` and nobody notices because nobody is watching that field. Generate the enum from the database at request time rather than typing it into the schema by hand.

**Strict mode is not always available.** On some free tiers you get "please reply in JSON" rather than a hard guarantee, which means fenced code blocks, trailing commentary and occasional prose. Validate in your own code regardless, retry once with the validation error appended, and fail loudly on the second attempt rather than writing a half-parsed row.

### Verify it

**Validate every single response, in code, before it touches your database.** Not sometimes. Log every failure with the raw text. The failure rate is a number you should be able to quote, and if you cannot, you are not validating.

**Read the schema for fields that should not exist.** A `total`, a `price`, an `order_id`, a `created_at` — anything the model is being asked to invent that your code already knows. Each one is a future wrong number.

**Check there is an escape hatch, and that it fires.** Send three deliberately ambiguous inputs: "ek special dena", "wahi jo kal liya tha", "do plate". If `needs_human` stays false on all three, the model is guessing and your schema is hiding it.

**Count what lands in the catch-all.** If `unresolved` or an `other` enum value is absorbing a tenth of your traffic, your enum is stale or your categories are wrong. This is a weekly check, not a one-time one.

**Round-trip one real example.** Take the structured output, render it back into a human sentence, and compare it with what the student actually typed. Anything dropped on the way in shows up immediately.

Red flags: code that splits strings on model output; money in a float or rupees rather than paise integers; a model-generated identifier used as a key; an enum typed by hand instead of generated from your data; no `needs_human` equivalent anywhere in the schema; a schema validated only in testing; and a free-text field whose contents get concatenated into a later prompt.

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
