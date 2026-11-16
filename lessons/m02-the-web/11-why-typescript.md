---
id: m02-11-why-typescript
module: m02-the-web
slug: why-typescript
title: "Types: Why Teams Add TypeScript"
order: 11
duration_min: 25
is_free: false
mission: false
volatility: medium
last_reviewed: 2026-11-16
---

## Key learnings
- Explain what a type is as a contract about shape, not as syntax
- Say exactly what types catch and, crucially, what they do not
- Tell compile time from run time, and why that distinction decides everything
- Recognise the two moves an AI uses to make type errors vanish without fixing them
- Say why a typed codebase is a better place for an AI agent to work

## Lesson

### The pain

Three students and an agent are working on the same project. On Tuesday, one of them asks the agent to tidy up the order API, and the agent renames `price_paise` to `price` because it reads better.

Nothing breaks. Nothing errors. The code runs. The build succeeds. Everything that reads `price_paise` now reads a field that does not exist, which gives back `undefined`, which added to a number gives `NaN`, which the price formatter turns into "₹NaN" on exactly one screen that nobody opens during testing.

It is found on Friday by a user. Tracing it back takes two hours, because by Friday there are forty commits and the rename is inside one labelled "tidy up order API".

Now the version with types. The same rename produces, immediately and before anything runs, eleven errors naming the eleven exact places that referred to the old field. The agent sees them, fixes them, and the whole episode costs ninety seconds.

### The idea

**A type is the shape of your data written down in a form a machine checks, before the code is ever run.**

Chapter 9 ended with a data contract written in a table. Types are that same table expressed so that a tool can enforce it. "A dish has an `id` number, a `name` string, a `price_paise` number and an optional `photo_url` string or null" stops being a document somebody might read and becomes a rule that fails loudly the moment anything disagrees with it.

### The analogy

The canteen's coloured tokens.

Blue tokens are for the dosa counter, yellow for chaat, green for beverages. Hand a blue token at the chaat counter and you are refused instantly — before anything is cooked, before anybody's time is wasted, and with the refusal naming the problem: wrong counter.

Without colours, every token is just a token. You hand it over, the chaat is made, and only when you come back for your dosa does anyone discover that two orders got mixed up and one of them has already been eaten.

**Where the analogy breaks:** and this is the most important paragraph in the chapter. The token colour is checked **at the counter, at the moment of service** — in programming terms, at run time. TypeScript's checks happen **before the kitchen opens**, when the code is compiled. Once your app is actually running, the types are gone; they do not exist in the shipped JavaScript at all.

So types cannot check anything that arrives while the app is running. Data from your API, data from a form, data from a URL — a type says "I expect this shape" and the data is under no obligation to comply. That is validation, it happens at run time, and it is M04's job. Confusing the two is the single most dangerous misconception about types, because it produces an app that feels safe and checks nothing.

### Picture it

```
  WITHOUT TYPES
  rename field ──▶ builds fine ──▶ ships ──▶ user sees "₹NaN"
                                              │
                                   2 hours of hunting ◀┘

  WITH TYPES
  rename field ──▶ 11 errors, each naming a file and line
                   │
                   └──▶ fixed before anything runs (90 seconds)

  WHAT TYPES DO NOT DO
  API sends price as a string ──▶ type says "number" ──▶ no error
                                   at run time, nothing checked
```

Types move the discovery of a shape mistake from a user on Friday to the moment the change is made, but they check only your own code — never data arriving from outside while the app runs.

### Where it lives

Types live in the client box and the server box both, which is most of their value: when the same type describes the dish your server sends and the dish your page renders, a mismatch between the two becomes impossible to introduce accidentally.

The checking itself happens in your tooling, before deployment. In M06 you will put that check into CI, so a pull request that breaks a type cannot be merged — which is the point at which types stop being a personal convenience and become a team guarantee. In M01's PR review, "the type check passes" is one of the few things about a diff you did not write that you can verify without reading it.

And M04 is where the boundary gets properly guarded: data arriving from a network or a form gets *validated* at run time, and only then is it allowed to be treated as the type it claims to be.

### Words to own

| Term | Plain meaning | Example |
|---|---|---|
| Type | A description of what shape a value has | `number`, `string` |
| Annotation | Writing the type down next to the thing | `price_paise: number` |
| Interface | A named shape for an object | `Dish` |
| Union type | "One of these" | `string \| null` |
| Optional field | May be absent entirely | `photo_url?: string` |
| Compile time | Before the code runs; when types are checked | During the build |
| Run time | While the app is actually running | A user tapping Pay |
| Inference | The tool working out the type without you saying it | From the value |
| `any` | "Stop checking this" | The escape hatch |
| Cast | Asserting a type the tool cannot verify | `as Dish` |
| Validation | Checking real incoming data at run time | On the server, M04 |
| Generated types | Types produced from a database or API schema | Kept in sync automatically |

### What types actually catch

It is worth being precise, because the honest list is narrower than the enthusiasm suggests and still extremely valuable.

They catch **renames and typos** — the Tuesday story, and `price_pasie` instead of `price_paise`. They catch **missing cases**: if a field can be `null`, every place that uses it must say what happens when it is. They catch **wrong units by name**, if you name your fields honestly. They catch **the forgotten consumer**: the eleventh file nobody remembered. And they give an AI agent something to read — the error message names the file, the line and the expected shape, which is far better direction than anything you would have typed.

They do **not** catch wrong logic. A total that adds instead of multiplying is perfectly well-typed. They do not catch bad data from outside. They do not catch a security hole. And they do not make the app faster — there is no type code in the shipped bundle at all.

### Tradeoffs and gotchas

**`any` turns the whole thing off.** It means "stop checking", and one `any` on a dish object removes the protection from every field and every consumer downstream. When an AI's type errors "all got fixed" very quickly, this is usually how.

**A cast is a promise you are making on the tool's behalf.** Writing `as Dish` tells the checker "trust me, this is a dish" and it stops asking. Applied to data that just arrived over the network, it is precisely the dangerous misconception above, written as code. It looks like typing and it is the opposite.

**Types slow down day one and pay back in week three.** On a two-day prototype they are overhead. On a six-month project with three people and an agent touching the same files, they are the thing that makes the files safe to touch at all. Your project is the second kind.

**Generated types beat hand-written ones.** If the database or the API can produce the types, do that, because hand-written ones drift the moment someone changes a column and forgets. M04 returns to this.

**Types are not documentation of intent.** `price_paise: number` says it is a number. It does not say it must be positive, under ₹10,000, or an integer. Those are run-time validations and they still need to exist.

**An over-typed codebase is its own problem.** Elaborate generic machinery to describe something simple makes a codebase nobody — including the AI — can change confidently. If an AI produces a type you cannot read aloud in one sentence, ask for a simpler one.

### Verify it

You are not going to read type definitions line by line. You are going to check for the escape hatches and the boundary.

1. **Search for `any`.** Every occurrence is a place where checking has been switched off. On new work the target is zero, and each one needs a reason stated out loud.
2. **Search for `as`.** Casts are assertions, not checks. Pay special attention to any cast applied to something that came from `fetch`, a form or a URL — those are the dangerous ones.
3. **Ask the AI: what does the type check at run time?** The correct answer is "nothing". If it tells you the types will protect you from a bad API response, it has the central misconception and everything it builds on top of that belief needs re-examining.
4. **Compare the type against the real response.** Open the Network panel, read the actual JSON, and check it field by field against the declared type. A type is a claim about reality, and this is how you find out whether reality agrees.
5. **Check the boundary is validated.** For every place data enters your app from outside, ask what checks it at run time. "It is typed" is not an answer to that question.
6. **Run the type check yourself and read the count.** Not the AI's summary — the actual output. Zero errors with four new `any` declarations is worse than eleven honest errors.
7. **Make a deliberate rename and watch.** Change one field name and confirm the checker lists every consumer. If it finds fewer places than you expect, something in between is `any`.

Red flags in AI output: new `any` declarations; `as` casts on network data; a type that disagrees with the actual API response; `@ts-ignore` or similar suppression comments; types hand-written when the schema could generate them; and the claim that adding types removed the need for validation. That last one is the one to watch for hardest, because it sounds like progress.

## Walkthrough
1. Take the data contract table you wrote in chapter 9 for your own product's main object. Have it in front of you.
2. Open an AI chat and ask it to express that exact table as a TypeScript interface, changing nothing about the fields. Read the result against your table, field by field, and confirm every optional field is marked optional.
3. Now ask it: "At run time, what does this interface check?" Read the answer carefully. The correct answer is "nothing — it is erased before the code runs". Note whether you got it.
4. Ask it to show you the same shape at the two boundaries: how a value arriving from `fetch` would be handled with a cast, and how it would be handled with real run-time validation. Have it explain the difference in one paragraph.
5. Ask it to deliberately produce three realistic type errors for this shape — a typo in a field name, a missing null check, and a wrong unit — and to show you the exact error message each would produce.
6. For each error message, read it as if you were the agent receiving it, and write down what instruction the message alone gives you. This is why typed codebases are easier for agents to work in.
7. Now ask it to "make all three errors go away as quickly as possible, without fixing the underlying problems". Look at what it does. You should see `any`, a cast, or a suppression comment. These are the three moves to recognise on sight.
8. Ask it to do it properly instead, and compare the two diffs side by side.
9. Open any API response from a real site in the Network panel, and ask the AI to write the interface for it from the actual JSON. Then check its interface against the response yourself, looking especially for fields it marked required that are actually missing from some items.
10. Write a five-line note for your own project: whether you will use types, whether they will be generated or hand-written, who checks them, where run-time validation will live, and your team's rule about `any`.

## Talk to AI
```prompt
CONTEXT:      I direct AI rather than writing code. I am a non-coder running
              a six-month project with two teammates and an AI agent, all
              editing the same codebase. The product is a canteen ordering
              app: a dish list, a cart, orders, and eventually UPI payments.
              Money is integer paise. Some dishes have no photo and some
              have no category. Data comes from an API I also control. I
              have a written data contract already, as a table.

GOAL:         A decision, with reasoning I can defend to my team, about
              whether to add TypeScript to this project — and if yes, the
              minimum set of rules that keeps it honest rather than
              decorative.

REQUIREMENTS: 1. A direct recommendation, not a list of considerations, and
                 the three strongest arguments against your own
                 recommendation
              2. A precise statement of what types will and will not catch
                 in MY situation, with one concrete example of each
              3. The boundary plan: exactly where data enters this app from
                 outside, and what checks it at run time rather than at
                 compile time
              4. A written team rule for `any`, for casts, and for
                 suppression comments, specific enough that I could reject
                 a pull request against it
              5. Whether my types should be hand-written or generated from
                 the API schema, and what breaks if we choose wrong
              6. What changes about how I review my AI agent's work once
                 types exist

CONSTRAINTS:  Three people, none of whom have written code before, plus an
              AI agent. Free tiers only. Six-month horizon with a shipped
              MVP in the first 30 to 45 days. Do not assume we will add a
              test suite before week six. Keep the rules to one page.

NON-GOALS:    Do not write any type definitions yet. Do not recommend any
              paid tooling. Do not teach me TypeScript syntax — I am not
              going to type it. Do not discuss framework choice.

ACCEPTANCE:   - Your recommendation is a yes or a no, with the three best
                counter-arguments stated
              - I can explain to a teammate, in two sentences, what types
                check and when
              - The boundary plan names every entry point for outside data
                in this product
              - The `any` rule is specific enough to fail a pull request
                against
              - Nothing in your answer implies that types remove the need
                for run-time validation

ASK:          First ask me up to five questions whose answers would change
              your recommendation. Then answer. Then, separately, list the
              two most common mistakes teams in exactly my position make
              when adopting types, and how I would notice each one in a
              diff.
```

## Stuck? Ask AI
```prompt
Explain the difference between compile time and run time as if I have never programmed, using an analogy from a kitchen or a factory. Then give me six mistakes a project might contain and ask me, for each, whether a type checker would catch it before the code runs or not. Do not tell me the answers until I have attempted all six.
```

```prompt
I keep hearing that types are not validation. Explain the difference concretely: show me the same dish object arriving from an API, once trusted because it was cast to a type and once actually validated, and walk me through exactly what happens in each version when the API returns a price as the string "45.00". Tell me what the user sees.
```

```prompt
Give me three realistic type errors from a small web project, with the exact error messages. For each, explain what the message is telling me, what the honest fix is, and what the lazy fix would be — including what `any`, a cast, and a suppression comment would each do. Then tell me how I would spot the lazy fix in a diff.
```

## Quiz

### Q1 · L2
When are TypeScript's checks performed?
- (a) Every time a user interacts with the page
- (b) Before the code runs, during the build — and the types are then erased ✅
- (c) On the server, as each request arrives
- (d) Continuously, at run time, inside the browser
> This is the whole distinction. Compile-time checking is why a rename is caught in ninety seconds, and it is also why types cannot inspect anything that arrives while the app is running.

### Q2 · L3
An agent renames `price_paise` to `price` in a typed codebase. What happens?
- (a) Nothing, until a user reports "₹NaN" on Friday
- (b) The check fails immediately, listing every file and line that referred to the old name ✅
- (c) The browser shows a runtime error on the affected screen
- (d) The field is automatically renamed everywhere
> The error list *is* the value. It names the eleventh consumer nobody remembered, before anything is shipped, and it gives the agent precise direction without you writing any.

### Q3 · L4
Your API returns `price_paise` as the string `"4500"` while the type declares it a number. What does TypeScript do?
- (a) Converts it to a number automatically
- (b) Throws an error when the response arrives
- (c) Nothing — the types are gone by then, so this needs run-time validation ✅
- (d) Refuses to compile the project
> Types describe what you expect of your own code. Incoming data is not bound by them, which is why the boundary needs real validation at run time — M04's job, not the type checker's.

### Q4 · L4
An AI reports that it "fixed all the type errors" in two minutes. What should you check first?
- (a) That the build output is smaller
- (b) Whether it added `any`, casts, or suppression comments rather than fixing the causes ✅
- (c) Whether the tests still pass
- (d) Whether the file count changed
> These are the three moves that make errors vanish without fixing anything. Zero errors with four new `any` declarations is strictly worse than eleven honest errors, because the protection is now silently absent.

### Q5 · L5
Which problem would types NOT have prevented?
- (a) A renamed field that eleven files still referred to
- (b) A missing `category` field treated as always present
- (c) A cart total that adds prices where it should multiply by quantity ✅
- (d) A `photo_url` that can be null being used without a null check
> Wrong logic is perfectly well-typed. Types enforce shape, never meaning — which is why a passing type check is not a substitute for actually verifying the behaviour.

## Go deeper
- [TypeScript: TS for JavaScript programmers](https://www.typescriptlang.org/docs/handbook/typescript-in-5-minutes.html)
- [MDN glossary: TypeScript](https://developer.mozilla.org/en-US/docs/Glossary/TypeScript)
