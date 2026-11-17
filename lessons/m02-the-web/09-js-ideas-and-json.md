---
id: m02-09-js-ideas-json
module: m02-the-web
slug: js-ideas-and-json
title: "Functions, Objects, Arrays and JSON as Ideas"
order: 9
duration_min: 15
is_free: false
mission: false
volatility: low
last_reviewed: 2026-11-16
---

## Key learnings
- Recognise the four shapes that hold almost all data in any product
- Read a JSON response and say whether it matches what the page expects
- Insist on money as integer paise, and say what floats cost you
- Decide a data shape before anything is built, and write it into the brief
- Name the five data red flags that produce "it works for some users"

## Lesson

### The pain

A team's cart worked for two weeks, then a user was charged ₹0.45 instead of ₹45. One endpoint returned prices as `45.00` and another as `4500` paise, and the cart divided by a hundred in one place and not the other. No error, no crash — just a wrong number, and only for carts mixing both endpoints.

The same week another screen died with `Cannot read properties of undefined`, because one dish had no category, so the API omitted the field and the page read `dish.category.name`. Both are **shape** mistakes, catchable by looking at the actual data.

### The idea

**Almost all data in any product is held in four shapes, and most bugs that "only happen sometimes" are a disagreement about which shape something has.**

A **function** is a named recipe: inputs in, output out. An **object** is one thing with labelled fields. An **array** is an ordered list of things. **JSON** is objects and arrays written as text so they can travel. You will never write these, but you must be able to say "the page expects a `price_paise` integer and this is a `price` string".

### The analogy

A **recipe card** is a function: "Sambar — inputs dal, tamarind, vegetables, masala; output one pot." Same inputs, same output, whoever cooks. A **menu card** for one dish is an object: labelled fields for name, price, is-vegetarian and photo, so nobody guesses that ₹45 is the price. The **whole menu** is an array of those cards. The **menu you type out and post to the new branch in Pune** is JSON.

**Where the analogy breaks:** a recipe card tolerates "salt to taste" and a cook uses judgement. A function gets exactly what it is passed; hand it a dish with no `category` and it stops loudly. There is no judgement step anywhere, which is why the shape must be right rather than approximately right.

### Picture it

```
  ONE OBJECT                      AN ARRAY OF THEM
  ┌──────────────────────┐        ┌──────────────────────┐
  │ id: 7                │        │ [ {dish}, {dish},    │
  │ name: "Masala Dosa"  │        │   {dish}, {dish} ]   │
  │ price_paise: 4500    │        └──────────────────────┘
  │ is_veg: true         │                 │
  │ category: "south"    │                 │ written as text
  │ photo_url: "/d7.jpg" │                 ▼
  └──────────────────────┘        ┌──────────────────────┐
         labelled fields          │ JSON over the network│
                                  └──────────────────────┘
```

An object is one thing with labelled fields, an array is an ordered list of such things, and JSON is that structure written as text so it can travel.

Settle the shape **before** anything is built, in the brief: "a dish is an object with `id` (integer), `name` (string, max 60 characters), `price_paise` (integer, never a decimal), `is_veg` (true or false), `category` (string, always present, `"other"` if unknown) and `photo_url` (string or null)." That prevents the ₹0.45 bug, the `undefined` crash, and the week-four argument about what `price` means.

### Where it lives

These shapes are the only thing that crosses between M00's client box and server box. The server turns database rows (M04) into objects, serialises them to JSON and sends them; the browser parses them back and renders. The *same shape* is therefore agreed in two places at once, which is why a rename breaks things silently: one side was updated and the other was not. Chapter 11 is the direct answer — types are the shape written down in a form a machine checks — and chapter 13 covers the trip itself.

### Words to own

| Term | Plain meaning | Example |
|---|---|---|
| Function | A named recipe: inputs in, output out | `cartTotal(items)` |
| Parameter | What a function expects to be given | The list of cart items |
| Return value | What a function hands back | The total in paise |
| Object | One thing with labelled fields | A single dish |
| Key / property | One field's label | `price_paise` |
| Array | An ordered list of things | The menu |
| Index | A thing's position in an array, counted from 0 | `menu[0]` is the first |
| JSON | Objects and arrays written out as text | What an API returns |
| Parse | Turn JSON text back into usable objects | The browser does this |
| `null` | A field that exists and is deliberately empty | `photo_url: null` |
| `undefined` | A field that was never there at all | A missing `category` |
| Shape / schema | The agreed set of fields and their kinds | The six lines above |

### Tradeoffs and gotchas

**Money is never a decimal.** Pass rupees as integer paise: ₹45 is `4500`. Decimal arithmetic is approximate — three items at ₹0.10 famously do not make ₹0.30 — and UPI works in the smallest unit for exactly this reason.

**`null` and "missing" are different, and both crash the same screen.** `null` exists and is empty on purpose; `undefined` was never sent. Pick one for "no photo" and be consistent.

**JSON has no date type,** so every date is a string needing a convention. Without an offset, "2026-11-16T09:00" is 9am somewhere, and your opening hours are wrong for anyone in another zone. Insist on ISO 8601.

**Renaming a key is a breaking change with no error.** `price` becomes `price_paise` and the page silently reads `undefined`, displaying "₹NaN". The strongest argument for chapter 11.

**An empty array is a legitimate state almost nothing handles,** and uneven items are the same trap: if three dishes have `photo_url` and one does not, every piece of code must defend itself. Make the shape uniform at the source.

### Verify it

1. **Ask for the shape before the code,** as a table of every field, its kind and whether it can be absent. Approve that table.
2. **Compare the Network response against what the page reads.** One letter of difference in a key is a silent failure.
3. **Check every money field is an integer.** `4500` good; `45.00` needs a conversation about units.
4. **Scan the whole array, not the first element.** Real data has fields some items lack.
5. **Check how dates are written.** No offset is a bug waiting for a user in another timezone.
6. **Force the empty case.** A category with no dishes that renders blank is a finding.
7. **Force the missing-field case.** Drop one optional field and see whether the page copes.

Red flags: `price` as a decimal · a key differing from the agreed shape by a letter · dates with no offset · `any`-shaped data whose fields are never stated · code assuming the array has at least one item · errors returned inside a success body (chapter 13) · a key renamed as part of an unrelated change, the scope violation M01's diff review exists to catch.

## Walkthrough
1. Open any site that loads data — a food delivery app, a jobs board, your college portal. Open DevTools, go to the Network panel and filter to Fetch/XHR, then reload.
2. Pick the request that looks like it fetches the main list of things. Click it and open the Response or Preview tab. You are now looking at JSON.
3. Expand the top level. Decide: is the top level an array of things, or an object with an array inside it? Write down which.
4. Expand the first item in the array and write out its shape by hand: every key, and what kind of value it holds. This is your reconstruction of their schema.
5. Now expand the fifth, tenth and last items. Compare their keys against your list from step 4. Note any key that is present in some and absent in others, or that is `null` in some.
6. Find any field that represents money. Note whether it is an integer, a decimal, or a string, and what unit it must be in for the displayed price to make sense.
7. Find any field that represents a date or time. Note whether it carries a timezone offset. Work out what would be displayed to a user whose phone is set to a different timezone.
8. Pick one field from the JSON and find where it appears on screen. Then use the Elements panel to confirm the displayed text matches the raw value — look for formatting the page is doing, like adding a ₹ symbol or rounding.
9. Use the site's own filters or search to produce zero results, and record exactly what the screen shows. Judge whether it explains itself or just looks broken.
10. Write up the shape you reconstructed as a six-to-ten line schema in plain words, then list the three ways this API would most easily break the page if someone changed it carelessly.
11. Draft the same schema for the main object in your own product, with every field marked as always-present or optional, and money in paise.

## Talk to AI
```prompt
CONTEXT:      I direct AI rather than writing code. My product is a canteen
              ordering app for students in Nashik on budget Android phones
              on slow 4G. I am about to have the menu, the cart and the
              order built, and I want the data shapes agreed and written
              down before anything is built, because I have been burned by
              a renamed field before. Payments will eventually go through
              UPI. The canteen has dishes that sometimes have no photo and
              sometimes have no category.

GOAL:         A written data contract for this product — the shape of a
              dish, a cart line, a cart and an order — precise enough that
              I can check any future API response against it in the Network
              panel in under a minute.

REQUIREMENTS: 1. For each of the four shapes, a table with: field name,
                 what kind of value, always present or optional, what an
                 empty or unknown value looks like, and one example value
              2. All money as integer paise, with field names that make the
                 unit impossible to misread
              3. All dates and times as strings with an explicit timezone
                 offset, and tell me the exact format
              4. An explicit decision, stated once and applied everywhere,
                 about whether missing data is null or an absent field
              5. The shape of an error, so a failure never arrives looking
                 like a success
              6. A list of the states each screen must handle because of
                 this shape: empty list, missing photo, missing category,
                 a 60-character dish name, and a cart with one item

CONSTRAINTS:  Plain JSON shapes described in words and tables — no code, no
              framework, no database schema yet. No decimal money anywhere.
              No field whose name does not state its unit. No date without
              an offset. Keep the whole contract to one page I can print
              and pin up.

NON-GOALS:    Do not design the database. Do not write any endpoint. Do not
              introduce TypeScript yet. Do not add fields for features I
              have not asked for, and if you think one is missing, say so
              separately rather than adding it.

ACCEPTANCE:   - Every money field is an integer and its name contains the
                unit
              - Every field is marked always-present or optional, with no
                ambiguity
              - I can read any API response in the Network panel and decide
                in under a minute whether it conforms
              - The contract names what the page must show for an empty
                list and for a dish with no photo
              - An error response cannot be mistaken for a success response

ASK:          Before producing the contract, ask me up to five questions
              whose answers would change the shapes. Then give the four
              tables. Then, separately, list the three fields you think I
              will regret not including, and the two decisions in my
              requirements that you think are wrong, with your reasoning.
              Do not act on either list without my approval.
```

## Stuck? Ask AI
```prompt
Explain the difference between null, undefined and an empty string using a non-software analogy, then give me eight situations from a food-ordering app and ask me which of the three each one should be. Do not tell me the answers until I have attempted all eight.
```

```prompt
Explain, with actual arithmetic I can follow, why storing money as a decimal number goes wrong on a computer, and why every payment system uses the smallest unit instead. Then show me the three places in a cart-and-checkout flow where the error would first become visible to a user.
```

```prompt
Here is a JSON response from an API: [paste it]. Do not write any code. Instead, reconstruct its schema as a table — every field, its kind, whether it is always present — and then tell me the five ways this shape would most easily break a page that consumes it, ranked by how likely each is.
```

## Quiz

### Q1 · L2
What is JSON?
- (a) A programming language used only in browsers
- (b) Objects and arrays written out as plain text, so they can travel or be stored ✅
- (c) A database that stores objects
- (d) The format CSS uses to describe styles
> JSON is a text representation of the same object and array shapes your code uses. The browser parses it back into usable objects on arrival.

### Q2 · L3
A user is charged ₹0.45 instead of ₹45. One endpoint returned `45.00` and another returned `4500`. What is the underlying mistake?
- (a) The payment provider applied the wrong currency
- (b) The unit of the money field was never agreed and fixed, so two conventions coexist ✅
- (c) The browser rounded the number incorrectly
- (d) The cart array was sorted in the wrong order
> Nothing errored because both values are valid numbers. Fixing the unit in the contract — integer paise, in a field named so the unit cannot be misread — removes the whole class of bug.

### Q3 · L3
A page crashes with `Cannot read properties of undefined (reading 'name')` for exactly one dish out of 95. What is the most likely cause?
- (a) That dish's record is corrupt in the database
- (b) That dish has no `category` field at all, and the page assumed one was always there ✅
- (c) The array index was off by one
- (d) The network request for that dish failed
> "Works for 94, crashes on one" is the signature of an optional field treated as mandatory. It is why every field in a data contract must be marked always-present or optional.

### Q4 · L4
Why insist on the data shape being written down and approved *before* anything is built?
- (a) It makes the AI produce code faster
- (b) It gives you something concrete to check every later response against ✅
- (c) It is required by the JSON specification
- (d) It prevents the AI from using arrays
> An approved table turns "does this look right?" into "does this conform?". Without it there is nothing to verify against, so every shape bug is found by a user instead of by you.

### Q5 · L5
An AI delivers a menu screen that looks excellent with its six sample dishes. Which single test is most likely to expose a shape problem?
- (a) Reload the page twice
- (b) Change the brand colour
- (c) Force the list to zero results and to one dish with no photo ✅
- (d) Check the page loads under 2 seconds
> Sample data is uniform and non-empty. The empty case and the missing-optional-field case are the two states real data produces constantly and AI sample data never does.

## Go deeper
- [MDN: Working with JSON](https://developer.mozilla.org/en-US/docs/Learn_web_development/Core/Scripting/JSON)
- [MDN: JavaScript object basics](https://developer.mozilla.org/en-US/docs/Learn_web_development/Core/Scripting/Object_basics)
