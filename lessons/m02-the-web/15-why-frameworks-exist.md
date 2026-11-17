---
id: m02-15-frameworks
module: m02-the-web
slug: why-frameworks-exist
title: "Why Frameworks Exist: Components, Props, State and the Landscape"
order: 15
duration_min: 15
is_free: false
mission: false
volatility: medium
last_reviewed: 2026-11-16
---

## Key learnings
- Explain the two problems frameworks exist to solve, in terms of real failures
- Tell props from state, and say who owns each piece of data
- Decide where a piece of state should live, and name the cost of getting it wrong
- Judge a framework choice by your team and your product, not by popularity
- Name the five component red flags that make a codebase hard to direct

## Lesson

### The pain

A team's canteen app has nine pages, each with its own copy of the header. They change the logo and ask the agent to "update the header everywhere": it edits four files, misses three, and creates a fifth variant in a page it tidied up. Nobody notices the three logos for a week.

The same app has chapter 8's cart problem grown up: the count appears in the header, the cart page and the checkout summary, each reading from somewhere different, so after fast taps the three disagree. One root cause — no single definition of anything, and copies drift.

### The idea

**A component is a piece of UI defined once, used many times, with holes for the parts that vary — and the framework rebuilds the screen for you whenever the data changes, so the picture can never fall out of step with the data.**

**Define once, use everywhere** kills the three-logos problem; **automatic re-render from state** kills chapter 8's two-sources-of-truth problem. What you pass *into* a component is **props**; what it remembers for itself is **state**.

### The analogy

The **thali tray** is the component: one standard layout, defined once by whoever bought the trays. What goes in the compartments today — rice, bhindi, dal, pickle — is **props**, passed in from outside and different for every tray. Which compartments the student has emptied is **state**: it belongs to this tray, now.

**Where the analogy breaks:** a tray is stamped out once and never thinks again. A component is re-evaluated every time its props or state change, which makes "where does this data live" consequential — should the cart count belong to the header, or above it where checkout can see it too? That question is most of the skill here.

### Picture it

```
            ┌──────────────────────┐
            │       App            │  owns: cart (the state)
            └──┬────────────┬──────┘
      props ▼  │            │  ▼ props
         ┌─────────────┐  ┌──────────────────┐
         │   Header    │  │   DishList       │
         │ count = 2   │  │ dishes = [...]   │
         └─────────────┘  └────┬────────┬────┘
                        props ▼         ▼ props
                       ┌──────────┐ ┌──────────┐
                       │ DishCard │ │ DishCard │
                       │ dish = 7 │ │ dish = 8 │
                       └────┬─────┘ └──────────┘
                            │ "add dish 7" travels back up
                            └──────────────▶ App updates cart
```

Data flows down as props from whoever owns it, and events travel back up to that owner, which updates the state once and lets every reader re-render from it.

The rule: **state belongs to the lowest component that contains everyone who needs it.** A card's "is the description expanded" flag belongs to the card; the cart is read by header, list and checkout, so it belongs above all three — **lifting state up**, which fixes the three-disagreeing-counts bug.

### Where it lives

Components live in the client box from M00, but a framework blurs that line: the same component can be rendered on the server into plain markup, or shipped as code that runs in the browser. One concept, two places, very different cost on a budget Android (chapter 16). Data the server owns should not be copied into component state at all (M04).

Framework names change faster than this course can be revised, so judge candidates on five questions:

| Judge a framework by | Why |
|---|---|
| Components and state done the usual way | Almost all do, so this chapter transfers |
| Renders on the server as well as the client | Decides what a slow-4G student sees first |
| A free tier that deploys in one step | A real constraint here, not a preference |
| Your AI agent works well in it | Stable documentation beats last quarter's release: the model saw more correct examples |
| Three beginners can read it in week two | A framework you cannot read is one you cannot direct |

### Words to own

| Term | Plain meaning | Example |
|---|---|---|
| Component | A piece of UI defined once and reused | `DishCard` |
| Props | Values passed into a component from outside | The dish it should show |
| State | Data a component keeps and can change | Is the description open |
| Composition | Building big components out of small ones | `DishList` of `DishCard` |
| Re-render | Redrawing a component because its data changed | After adding to the cart |
| Lifting state | Moving state up to a shared parent | The cart |
| Prop drilling | Passing props through layers that do not use them | A smell, not a sin |
| Server component | A piece rendered on the server, not shipped as script | Chapter 16 |
| Client component | A piece that runs in the browser, needed for interaction | A button |
| Framework | The library that does components and rendering | The thing you pick |
| Meta-framework | A framework plus routing, data and build conventions | What you deploy |

### Tradeoffs and gotchas

**A framework is not free:** extra bytes, an extra build step, new concepts and new error messages. For a genuinely static page, plain HTML and CSS are faster to build and easier to direct. Reach for a framework when there is real interaction and real reuse.

**Copying a component is the mistake frameworks exist to prevent, and AI does it constantly.** Asked for a slightly different card, an agent produces `DishCard2` instead of adding a prop. Say: "extend the existing component with a prop unless you can justify a second one."

**Prop drilling through five layers is a signal, not a crime.** It usually means state is owned too high, or a layer should not exist. Fix ownership before reaching for a state library.

**Everything-is-a-client-component is the common AI default,** because it always works — and it ships script for parts of the page that never change, which chapter 16 measures in seconds on 4G.

### Verify it

1. **For every value on screen, ask who owns it.** One answer, and it must be one component or the server.
2. **Count the files a one-line visual change touches.** One is correct; four means the header was copied.
3. **Search for numbered component names.** `Card2`, `ButtonNew`, `HeaderCopy` — each is a prop that was not added.
4. **Ask which components are interactive,** then check the others ship no script.
5. **Look for data fetched in two places.** Two requests mean two copies and two chances to disagree.
6. **Type into a text field with 4x CPU throttling.** Lag means state owned too high: a keystroke at the top re-renders everything below.
7. **Read the props a component takes.** More than about seven means it is doing several jobs.

Red flags: a second component where a prop belonged · the same header defined twice · server data copied into local state · everything marked interactive · props passed through four layers that do not use them · a state library brought in to solve wrong ownership · a "refresh" button, usually a stale-copy confession.

## Walkthrough
1. Open any modern web app and look at its header. Navigate between three different pages and watch whether the header flickers, reloads or stays steady. Note what that suggests about whether it is one component or several.
2. Find a repeated card or row on the page — a product, a job, a dish. Inspect two of them in the Elements panel and compare their structure, class names and nesting. Identical structure is evidence of one component used twice.
3. Now find two cards that look *slightly* different — a featured one, or one with a badge. Compare their markup. Decide whether this looks like one component with a prop, or two separate components.
4. Find a value that appears in two places at once — a cart count in the header and on the cart page, or an unread badge. Change it and watch whether both update together or whether one lags.
5. If one lags, you have evidence of two copies. Write down which places disagreed and in what order they updated.
6. Open the Network panel, filter to Fetch/XHR, and reload. Look for the same data being fetched more than once. Note any duplicate request and what it suggests about ownership.
7. Set CPU throttling to 4x, find a text input, and type a long sentence quickly. Note whether the characters appear smoothly or in bursts. Lag here is a re-render cost you would never see on a laptop.
8. Still throttled, look at the Network panel's total JavaScript transferred for a page that is mostly text. Write the number down — chapter 16 will use it.
9. Pick one repeated component on this page and write out the props you think it must take, as a list of names. Then write what state, if any, you think it keeps for itself.
10. Finally, sketch your own product's component tree on paper: the five or six components you will need, and for each piece of data, which component owns it. Mark anything owned by the server rather than by a component.
11. Take one piece of data in your sketch and deliberately argue the other side — why it might belong one level higher or lower — then write one sentence on why you chose where you did.

## Talk to AI
```prompt
CONTEXT:      I direct AI rather than writing code. Three teammates, none
              of us have coded before, plus an AI agent, all working in one
              codebase for six months. The product is a canteen ordering app
              for students in Nashik on budget Android phones on slow 4G.
              Screens: a dish list, a dish detail, a cart, a checkout, a
              student's past orders, and a staff dashboard for marking
              dishes sold out. Shared UI: a header with a cart count, and a
              dish card used in several places. Money is integer paise.
              Free tiers only. We must ship a working MVP in 30 to 45 days.

GOAL:         A component and state plan for this product, plus a framework
              recommendation I can defend, such that a one-line visual
              change touches one file and no value on screen has two owners.

REQUIREMENTS: 1. A component tree: every component, what it contains, and
                 the props it takes — with a note on any component you
                 expect to be reused in more than one screen
              2. A state ownership table: every piece of data in the
                 product, who owns it (a named component, or the server),
                 and the one-sentence reason
              3. An explicit list of which data the server owns and must
                 not be copied into component state, and what we do instead
              4. Which components need to be interactive in the browser and
                 which can be rendered once and shipped as plain markup
              5. A framework recommendation judged against five criteria:
                 standard component model, server rendering, one-step free
                 deployment, how well an AI agent works in it, and whether
                 three beginners can read it in week two
              6. A written team rule that prevents the AI from creating a
                 second component where a prop belonged

CONSTRAINTS:  Free tiers only. Must deploy in one step. No state-management
              library in the first version. No component may take more than
              seven props without you flagging it. No server data duplicated
              into local state without a stated reason. Assume none of us
              can debug a build system, so prefer boring and
              well-documented over new and clever.

NON-GOALS:    Do not design the database or the API. Do not write any
              component code yet. Do not choose a CSS approach. Do not plan
              the payment integration.

ACCEPTANCE:   - Every piece of data in the table has exactly one owner
              - Changing the logo would touch exactly one file
              - The header's cart count and the cart page read the same
                single source
              - The plan names which components ship no JavaScript
              - Your framework recommendation is a single pick with the
                three strongest arguments against it stated
              - The anti-duplication rule is specific enough to reject a
                pull request against

ASK:          Before producing anything, ask me up to five questions whose
              answers would change the plan. Then give the component tree
              and the state table. Then, separately, name the two state
              ownership decisions in your own plan that you think are the
              most arguable, and what you would choose instead. Do not act
              on that list without my approval.
```

## Stuck? Ask AI
```prompt
Explain the difference between props and state using an analogy that is not about trays, forms or containers. Then give me ten pieces of data from a food-ordering app and ask me whether each should be a prop, local state, state lifted to a shared parent, or owned by the server. Do not tell me the answers until I have attempted all ten.
```

```prompt
My cart count shows a different number in the header than on the cart page, but only sometimes. Explain what structural mistake causes this, why it is intermittent, and why moving state to a shared parent fixes it rather than merely hiding it. Then give me three other symptoms that have the same root cause.
```

```prompt
I asked for a dish card with a "featured" variant and the AI created a second component called DishCardFeatured. Explain exactly what this costs me over six months, with three concrete future changes that become harder. Then write the instruction I should have put in my brief to prevent it, and tell me when creating a second component genuinely would be the right call.
```

## Quiz

### Q1 · L2
What is the difference between props and state?
- (a) Props are numbers and state is text
- (b) Props are passed in from outside; state is data the component keeps and can change ✅
- (c) Props live on the server and state lives in the browser
- (d) Props are read by the user; state is read by the developer
> Props come from whoever used the component; state belongs to the component itself. Knowing which is which is how you answer "who owns this value", which is the question behind most UI bugs.

### Q2 · L3
A cart count appears in the header, the cart page and the checkout summary, and the three sometimes disagree. What is the fix?
- (a) Add a refresh button so users can resynchronise them
- (b) Lift the cart to a shared parent so all three read one source ✅
- (c) Make each place fetch the count from the server on every render
- (d) Introduce a state-management library
> Three copies drift. One owner above all three readers removes the possibility, which is the structural version of chapter 8's single-source-of-truth rule.

### Q3 · L3
A dish card's "description expanded" flag is stored at the top of the app. What is the cost?
- (a) Nothing; state is cheap wherever it lives
- (b) Every expansion re-renders a large part of the app, which lags on a budget phone ✅
- (c) The flag will be shared between all cards accidentally
- (d) The flag cannot be read by the card itself
> State belongs to the lowest component containing everyone who needs it. Nothing outside the card cares about this flag, and 4x CPU throttling is where the needless re-render becomes visible.

### Q4 · L4
Asked for a "featured" variant of a dish card, an AI creates `DishCardFeatured`. Why push back?
- (a) Two components are slower to render than one
- (b) You now have two definitions that will drift, which is the exact problem components exist to prevent ✅
- (c) Component names cannot contain more than one word
- (d) The new component will not receive props correctly
> Every future change to the card has to be made twice, and one day it will be made once. A prop on the existing component keeps the definition single.

### Q5 · L5
Which criterion for choosing a framework is specific to working with an AI agent, and why does it matter?
- (a) Whether it has the newest features, because the AI prefers modern code
- (b) How much stable public documentation and history it has, because that is what the model has seen correct examples of ✅
- (c) Whether it is written in the same language as the agent
- (d) How many stars its repository has, because popularity means quality
> A framework released last quarter produces worse AI output than a boring one with years of correct examples behind it. It is an uncomfortable criterion and a real one for a team that types nothing by hand.

## Go deeper
- [React: Thinking in React, and choosing the state structure](https://react.dev/learn/thinking-in-react)
- [MDN: Framework main features](https://developer.mozilla.org/en-US/docs/Learn_web_development/Core/Frameworks_libraries/Main_features)
