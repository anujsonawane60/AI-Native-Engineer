---
id: m10-04-tool-calling
module: m10-building-with-ai
slug: tool-and-function-calling
title: "Tool and Function Calling"
order: 4
duration_min: 15
is_free: false
mission: false
volatility: low
last_reviewed: 2026-11-16
---

## Key learnings
- Explain why the model never executes a tool — it only asks your code to
- Write a tool definition whose description decides when it gets used
- Keep tools narrow, and refuse to ship a generic escape-hatch tool
- Treat tool arguments as untrusted input and validate every one
- Read a list of tool definitions as the permission list it really is

## Lesson

### The pain

A team's canteen bot gets asked "kitne idli bache hain". Stock lives in a database the model has never seen, so the team pastes the inventory table into every prompt: four thousand tokens a call, stale when sent.

Then a student types "mera order cancel kar do" and the bot warmly replies that it has cancelled the order. Nothing was cancelled; there was never any mechanism by which anything could be. The model produced the most plausible next sentence, and the student went to class expecting a refund.

### The idea

**A tool is a function in your code that you describe to the model. The model does not run it. It asks you to run it, by name, with arguments — and your code decides whether to.**

That sentence carries the rest of the module. The model has no hands: it emits a request saying `get_stock(dish_id: "idli")`, and your code checks it, runs it or refuses, and returns the result. Power never leaves your side of the wire.

### The analogy

The canteen helper cannot walk into the storeroom; it is locked and the storekeeper has the key. What the helper can do is fill in a requisition chit — item, quantity, signed. The storekeeper reads it, decides whether it is reasonable, and brings the rice or says no.

**Where the analogy breaks:** the storekeeper has been there eleven years and will laugh at a chit for four hundred kilos. Your code has no such instinct. Every refusal it will ever make must be written down in advance as a bound, an allowlist or a gate — assuming otherwise is how `delete_order` ships with no checks.

### Picture it

```
  student: "kitne idli bache hain"
        │
        ▼
  ┌───────────┐  "call get_stock(dish_id='idli')"
  │   MODEL   │────────────────────────────────┐
  └───────────┘                                ▼
        ▲                            ┌──────────────────┐
        │   {"qty": 14}              │   YOUR CODE      │
        └────────────────────────────│ validate args    │
        │                            │ check permission │
        ▼                            │ run the query    │
  "14 idli bache hain."              └──────────────────┘
                                              │
                                              ▼
                                         your database
```

The model asks, your code decides and acts, the result goes back, and the model turns it into a sentence. Every arrow between the model and the database passes through code you wrote.

```
name:        get_stock
description: Returns remaining quantity for ONE dish on
             today's menu. Use when the user asks whether
             something is available or how much is left.
             Do NOT use for prices or for past days.
parameters:  { "dish_id": { "enum": [...from menu...] } }
returns:     { "qty": integer, "as_of": string }
```

The **description is a prompt** — the only thing the model reads when choosing between tools, and the sentence beginning "Do NOT use for" prevents more wrong calls than any system-prompt tightening. The parameters are chapter three's schema reversed: an enum means the model cannot ask about a dish you do not sell.

### Where it lives

Each tool is an endpoint in miniature, so M04's rules apply: validate input, authorise the caller, be idempotent on writes. The rest of the module builds on it.

- Retrieval in chapter ten is a read tool, `search_docs`
- An agent in chapter fourteen is this loop run until a stopping condition
- Your coding agent in chapter seventeen is a tool list: read, write, run command

### Words to own

| Term | Plain meaning | Example |
|---|---|---|
| Tool | A function of yours the model may request | `get_stock` |
| Tool definition | Name, description and argument schema | The block above |
| Tool call | The model's structured request to run one | `get_stock("idli")` |
| Tool result | What your code sends back | `{"qty": 14}` |
| Read tool | Changes nothing, safe to repeat | Fetch today's menu |
| Write tool | Changes the world | Cancel an order |
| Idempotency key | A token making a repeat safe | Same key, one refund |
| Allowlist | Tools permitted without asking | All read tools |
| Generic tool | One tool that can do anything | `run_sql` — do not ship it |
| Parallel tool calls | Several requested in one turn | Stock for three dishes |

### Tradeoffs and gotchas

**Never build a generic tool.** `run_sql(query)`, `http_get(url)` and `run_command(cmd)` each hand the model your whole database or machine, and cannot be bounded or allowlisted. Narrow instead — `get_stock(dish_id)`, `list_todays_menu()` — boring, bounded, reviewable.

**Split read from write, never both in one tool.** Reads are reversible and can be auto-approved, which makes a product feel fast. Writes need an idempotency key so a retry does not double-charge — M04's UPI discipline — and a human gate when irreversible. Return errors as data, not exceptions that end the conversation.

**Tool results are untrusted content.** A tool that fetches a web page or returns a free-text `notes` column injects someone else's writing into the model's context, where it may be read as an instruction. The tool is the attack surface — chapter seven.

**A read tool can still be an exfiltration channel.** `list_all_users()` sounds safe, and combined with any outward action it is a data breach. Scope read tools to the current user.

**Past roughly ten tools, choice degrades** — the model picks neighbours, `get_price` for a stock question. Batch related calls rather than adding more tools.

### Verify it

- **Read your tool list as a permission list.** If a hostile user could choose any sequence of these, what is the worst outcome? That is your blast radius, defined by this list and not your system prompt.
- **Find the generic tool.** Search for `sql`, `query`, `exec`, `command`, `url`, `eval`. If AI wrote your tool layer, one probably exists.
- **Check every write tool for three things:** an idempotency key, a bound on what it can affect, an approval gate if it is irreversible.
- **Test the wrong-argument path.** Call each tool with a nonexistent id, a negative number and an empty string; it should refuse cleanly and return the error as data.
- **Test the wrong-user path.** As student A, ask the bot to cancel student B's order. If the tool does not re-check ownership server-side, you have shipped M07's missing authorization with a chat interface on it.

Red flags: a tool named `run_sql`, `exec` or `http_get`; a write tool with no idempotency key; a description that never says when *not* to use it; unvalidated arguments; a read tool returning all users' rows; a tool result with no marker saying it is data; a claimed action with no matching tool.

## Walkthrough
1. List the three questions users most ask your product that the model cannot answer from the prompt alone. These are your first read tools.
2. For each, write a tool definition by hand: name, a description that includes one "do NOT use this for" sentence, and parameters as a schema with enums wherever the values come from your own data.
3. Write down, for each tool, exactly what your code returns — field names and types — and what it returns when the thing is not found.
4. Now list every action a user might ask for that changes something: cancel, update, send, pay. Mark each as reversible or irreversible.
5. Split your list into a read set and a write set in two separate columns. No tool may appear in both; if one does, split it into two tools.
6. Implement argument validation for every tool before implementing any tool body. Return errors as data in the same shape every time.
7. Direct AI to build the loop: send the tool definitions, receive a tool call, validate, execute, return the result, let the model answer. Run it with one read tool only.
8. Add a second read tool and confirm the model picks correctly between them. If it picks wrong, fix the descriptions rather than the system prompt.
9. Add one write tool with an idempotency key and a confirmation step that requires the user to say yes before your code executes it. Test the double-submit case.
10. Finally, try to break your own tool layer: a nonexistent id, a negative quantity, another user's order id, and an empty argument. Record what happened for each and fix anything that threw or succeeded when it should not have.

## Talk to AI
```prompt
Design the tool layer for one AI feature in my product, then audit your own design as if you were attacking it.

MY FEATURE: [one sentence - what the user can ask for]
MY DATA: [tables and columns the feature would need, or a description]
MY USERS: [who they are, and whether their data is private per user]
ACTIONS USERS MIGHT WANT: [list anything that changes state - cancel, update, send, pay]

Rules you must follow:
- No generic tools. Nothing named run_sql, exec, run_command, http_get or anything equivalent. If you think you need one, explain why narrow tools cannot do the job instead.
- Every tool is either read-only or write. Never both.
- Every parameter that comes from my own data must be an enum or a validated id, and you must say where the allowed values are generated from at runtime.
- Every tool description must include a sentence saying when NOT to use it.

Deliver, in this order:
1. The read tools: name, description, parameters, return shape, and the not-found response.
2. The write tools: the same, plus the idempotency key and whether it needs a human approval gate, with your reasoning.
3. A table of what my code must validate on arrival for every parameter.
4. THE AUDIT: assume a hostile user can make me call any sequence of these tools with any arguments that pass validation. Name the worst three outcomes and which tool makes each possible.
5. Which of my tools returns text written by someone other than me - a user note, a fetched page, a free-text column - and is therefore a prompt-injection path.
6. The one tool in your own design you would most expect to be misused, and the specific change that would fix it.
```

## Stuck? Ask AI
```prompt
Explain tool calling to me as if I thought the model runs the function itself. Walk through one full round trip for "how many idli are left" - what the model sends, what my code receives, what my code decides, what goes back, and what the user finally sees. Be explicit about which step has the power to say no.
```

```prompt
I want a tool that answers any question about my database, so I was going to write one tool that takes a SQL query. Explain concretely what goes wrong with that, with examples of what a user could get out of it. Then show me the five narrow tools I should build instead for a canteen ordering app.
```

```prompt
Here are my tool definitions: [paste them]. Read them as a permission list rather than as code. What is the worst thing achievable by calling these in some order with valid arguments? Which ones should require human approval? Which returns text that someone other than me wrote?
```

## Quiz

### Q1 · L2
When a model makes a tool call, what has actually happened?
- (a) The model executed your function and returned the result
- (b) The model emitted a structured request for your code to run ✅
- (c) The provider ran the function on their servers
- (d) The model read your database directly
> The model has no hands. It produces a named request with arguments; your code validates, decides and executes. That separation is the basis of every agent safety control in this module.

### Q2 · L3
Which part of a tool definition most determines whether the model uses it correctly?
- (a) The tool's name
- (b) The description, including when not to use it ✅
- (c) The return type
- (d) The order of the tools in the list
> The description is the prompt the model reads when choosing. A "do NOT use this for" sentence prevents more wrong calls than any system-prompt tightening.

### Q3 · L4
Why is a single `run_sql(query)` tool a bad design even though it is powerful?
- (a) SQL strings use too many tokens
- (b) It is slower than several narrow tools
- (c) It cannot be bounded or allowlisted, so the blast radius is the whole database ✅
- (d) Models write invalid SQL too often
> Narrow tools can be validated, bounded and individually gated. A generic tool makes every later safety question unanswerable because any query is a legal argument.

### Q4 · L4
Your bot has a `cancel_order(order_id)` tool. Student A asks it to cancel student B's order and it works. What failed?
- (a) The system prompt did not forbid it
- (b) The model should have refused
- (c) The tool did not re-check ownership server-side ✅
- (d) The order id should have been encrypted
> This is authentication without authorization wearing a chat interface. Ownership must be enforced in the tool, because the model's cooperation is not a permission check.

### Q5 · L5
A tool fetches a web page and returns its text to the model. What new risk has this introduced?
- (a) Higher token costs only
- (b) Text written by a stranger now enters the context and may be read as an instruction ✅
- (c) The model may cache the page incorrectly
- (d) None; read-only tools are safe by definition
> Tool results are untrusted content. A read-only tool that pulls in outside text is a prompt-injection path, which is why the defence has to be structural rather than a stronger instruction.

## Go deeper
- [Anthropic: Tool use with Claude](https://docs.anthropic.com/en/docs/build-with-claude/tool-use)
- [OpenAI: Function calling](https://platform.openai.com/docs/guides/function-calling)
