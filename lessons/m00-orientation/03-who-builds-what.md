---
id: m00-03-who-builds-what
module: m00-orientation
slug: who-builds-what
title: "Frontend, Backend, Infrastructure: Who Builds What"
order: 3
duration_min: 20
is_free: true
mission: false
volatility: low
last_reviewed: 2026-11-16
---

## Key learnings
- Map the four boxes onto the roles real teams actually hire for
- Say what a frontend, backend, DevOps and data role each own
- Explain what "full stack" means and why it is now the default
- Place a task in the right role before you ask AI to do it
- Understand which of these roles AI has changed most

## Lesson

### The pain

You will sit in a room — a hackathon, an internship interview, your own group of three — and someone will say "I'll take the backend." If you do not know what that sentence promises, you cannot divide work, you cannot tell whether a task is done, and you cannot tell whether the person is overloaded.

Worse: when you brief an AI agent, you are effectively hiring all of these roles at once. If you do not know what each role is responsible for, you will not notice which responsibility the AI silently skipped.

### The idea

**The four boxes map onto four jobs: frontend owns the client, backend owns the server, data owns the database, and infrastructure owns everything the other three run on.**

These are not separate skills so much as separate *responsibilities*. One person can hold all four — on your startup, you will. But the responsibilities do not disappear just because one person holds them. They just become easier to drop.

### The analogy

Back to the canteen, now hiring.

**Frontend** is the person who designs the menu board, keeps it legible, puts today's specials at the top and makes sure the prices are readable from the back of the queue. They do not cook.

**Backend** is the kitchen staff. They decide whether an order is possible, in what sequence to cook, and what to do when the paneer runs out mid-order. They never speak to customers directly; they work from the chits.

**Data** is the person who runs the storeroom and the register — what is in stock, what was sold, what it cost, what the records say happened last Tuesday. When there is an argument about money, their records settle it.

**Infrastructure** is the person who makes sure there is gas in the cylinder, the counter has power, there are enough tables for the lunch rush, and the shutter opens at 11. Nobody notices them until the gas runs out, at which point nothing else matters.

**Where the analogy breaks:** a canteen's counter does not suddenly need to serve ten thousand students because someone posted about it. Software does. Infrastructure in software is less "keep the lights on" and more "survive an unpredictable crowd", which is why it became its own specialism.

### Picture it

```
         ┌──────────────────────────────────────────┐
         │              INFRASTRUCTURE              │
         │   servers, deploys, domains, scaling,    │
         │        monitoring, security, cost        │
         │  ┌──────────┐              ┌──────────┐  │
         │  │ FRONTEND │◀──network──▶ │ BACKEND  │  │
         │  │  screens │              │ decisions│  │
         │  │  input   │              │  rules   │  │
         │  └──────────┘              └────┬─────┘  │
         │                                 │        │
         │                            ┌────▼─────┐  │
         │                            │   DATA   │  │
         │                            │  schema  │  │
         │                            │ queries  │  │
         │                            └──────────┘  │
         └──────────────────────────────────────────┘
```

Frontend, backend and data sit inside infrastructure, because infrastructure is not a fourth peer — it is the ground all three stand on. This is why "it works on my machine" is an infrastructure problem, not a code problem.

### Where it lives

Here is the same split as a table you can actually divide work with:

| Role | Owns | Typical task | Done looks like |
|---|---|---|---|
| Frontend | The client box | "Add a quantity stepper to the cart" | It works on a 5-inch phone and with a keyboard |
| Backend | The server box | "Reject orders for out-of-stock items" | A request for 50 paneer rolls returns a clear error |
| Data | The data box | "Store orders so we can report daily sales" | The query for yesterday's revenue runs in under a second |
| Infrastructure | Everything underneath | "Make the site live on our domain with HTTPS" | A stranger can open it, and you get told when it breaks |

### Full stack, and what it means now

"Full stack" means one person takes responsibility across frontend, backend and usually data. For a decade it was a slightly suspicious claim — nobody is excellent at everything. Two things changed it:

1. **Platforms absorbed infrastructure.** Deploying used to mean renting a machine and configuring it. Now a push to a repository can produce a live, encrypted, globally distributed site. A large chunk of the old infrastructure job became someone else's product.
2. **AI absorbed the typing.** The bottleneck used to be knowing the syntax of each layer. That is now cheap. What remains expensive is knowing what *should* be built in each layer and whether what came back is correct.

That is why this program is structured the way it is. You are being trained as a full-stack **director**: someone who holds all four responsibilities and uses AI to execute each one. In your group of three you will still split product, tech and growth — but the technical responsibilities above do not get to vanish into "the AI handled it."

### Which role did AI change most?

Worth knowing, because it tells you where to put your attention.

- **Frontend changed most in output speed.** Screens from a description, a design converted to a working layout. This is where AI looks most magical, and where it most often produces something that looks right and behaves wrong — inaccessible, unusable on small screens, broken in error states.
- **Backend changed less than it appears.** AI writes endpoints fluently, but the decisions — what is allowed, what happens on failure, what must never be trusted — are exactly the decisions it cannot make for you, because they depend on your product and your risk.
- **Data changed least.** Schema design is a long-lived commitment. A bad schema is expensive to undo after you have real orders in it, and AI will cheerfully propose one that works today and hurts in six months.
- **Infrastructure split in two.** The routine half got absorbed by platforms. The remaining half — cost, security, what to do at 2 am when it is down — got *more* important, because you now have fewer people who understand the machine.

### Words to own

| Term | Plain meaning | Example |
|---|---|---|
| Frontend | Everything the user sees and touches | The cart screen |
| Backend | The server-side rules and decisions | "Reject this order" |
| Full stack | One person responsible across the layers | You, for the next six months |
| DevOps | Practices for shipping and running software reliably | Automatic deploy on every merge |
| SRE | Site Reliability Engineering; keeping things up | On-call, error budgets |
| Platform / PaaS | A service that absorbs infrastructure work | Push code, get a live site |
| Stack | The specific set of technologies chosen | "Next.js, Postgres, Vercel" |
| Schema | The shape your data is stored in | Orders have a student, items, total, status |

### Tradeoffs and gotchas

**Boundaries are where bugs live.** The hardest bugs are rarely inside one box; they are at the handoff. The frontend assumed the price came as rupees, the backend sent paise. Nobody is wrong inside their own box. When you divide work in your group of three, write down the contract between the boxes — what exactly gets sent, in what units, with what name — before anyone builds.

**"I'll take the backend" is not a plan.** It is a claim on a box. The plan is the list of decisions that box must make. Ask for those.

**A single owner of everything drops the invisible work.** Infrastructure and data are the two that get dropped, because nothing visibly breaks on day one. No backups, no monitoring, no idea what it costs — these are not noticed until they are expensive. Put them on the list explicitly.

**AI will not tell you which box it skipped.** Ask it for a feature and you will get a frontend and a backend. You will often not get the migration, the error state, the permission check, or the monitoring — and it will not mention that it left them out.

### Verify it

A four-question check you can run on any AI-built feature. It takes two minutes and catches an enormous proportion of real problems.

1. **Frontend:** what does this look like when it is loading, when it is empty, and when it fails? (AI defaults to building only the happy path.)
2. **Backend:** which of these decisions is enforced on the server, not just hidden in the UI? (Hiding a button is not a permission.)
3. **Data:** what changed in the database shape, and what happens to the rows that already exist? (A new required field breaks every old row.)
4. **Infrastructure:** how does this get deployed, what does it cost per month, and how will I know when it breaks?

If the AI's answer to any of these is silence, that is your next task — not a thing to discover in week four.

## Walkthrough
1. Pick an app you use daily. Open it and find one screen with a clear action — a cart, a transfer, a submit button.
2. List three things on that screen that are purely **frontend** decisions: a layout choice, a piece of wording, a visual state like a disabled button.
3. Name one rule that must be enforced on the **backend**, and say why putting it in the frontend would be unsafe. Money, identity and permissions are the usual candidates.
4. Describe, in one sentence each, two things that must be stored in **data** for this screen to work tomorrow rather than only right now.
5. Name two **infrastructure** facts you cannot see from the screen but that must be true for it to work at all — a domain, encryption, something that restarts it when it crashes.
6. Now assign each of your items to one of the four roles in a two-column list. Anything you cannot assign confidently is a boundary — note it, because that is where the handoff bugs will be.
7. Finally, pick the one item you would be least confident briefing an AI to build correctly. That is the chapter in this program you should pay the most attention to.

## Talk to AI
```prompt
I am planning a feature and I want to make sure I have not forgotten a layer. The feature is: [describe it in one or two sentences].

Break it into exactly four lists:
1. FRONTEND — what the user sees, including the loading state, the empty state and the error state
2. BACKEND — the rules and decisions that must be enforced on the server, and why each one cannot be trusted to the client
3. DATA — what must be stored, and what happens to data that already exists
4. INFRASTRUCTURE — what is needed to deploy it, roughly what it costs per month, and how I would find out it had broken

Then tell me which of the four I am most likely to forget, and give me the single question I should ask myself before calling this feature finished. Do not write any code.
```

## Stuck? Ask AI
```prompt
I keep mixing up backend, DevOps and infrastructure. Explain the difference between these three using one running example, and tell me which tasks belong to each. Then give me three tasks and ask me to sort them, without telling me the answers first.
```

```prompt
I am in a team of three building a product. Help me divide the technical responsibilities across frontend, backend, data and infrastructure so that nothing is left unowned. Ask me what each person is interested in before you suggest a split.
```

```prompt
Explain what "full stack" means, and then argue the case AGAINST calling yourself full stack as a beginner. I want to understand the honest limits of the term, not the marketing version.
```

## Quiz

### Q1 · L3
A team ships a cart feature. The frontend shows totals in rupees; the backend calculates in paise. Nothing inside either box is wrong, but the displayed total is 100× too large. What kind of problem is this?
- (a) A data problem — the database stored the wrong number
- (b) A boundary problem — the contract between two boxes was never agreed ✅
- (c) An infrastructure problem — the deploy was misconfigured
- (d) A frontend problem — the formatting code has a bug
> Each box behaved correctly by its own assumption. The failure is the undocumented handoff, which is exactly where the hardest bugs live.

### Q2 · L4
An AI agent builds a feature where an admin-only button is hidden from normal users in the interface, and nothing else is changed. What is wrong?
- (a) Nothing — hiding the button is sufficient
- (b) Hiding a control is not a permission; the server must still refuse the action ✅
- (c) The button should be disabled rather than hidden
- (d) The permission belongs in the infrastructure layer
> The client cannot be trusted. A user can call the underlying endpoint directly, so authorization has to be enforced on the server regardless of what the UI shows.

### Q3 · L2
Which responsibility is most often dropped when one person owns the entire stack?
- (a) Frontend, because there is a lot of visual work
- (b) Backend, because the rules are complex
- (c) Infrastructure and data, because nothing visibly breaks on day one ✅
- (d) All four are dropped equally
> Backups, monitoring and cost are invisible until they are expensive. Frontend problems are noticed immediately, which is exactly why they get fixed.

### Q4 · L3
Which of these is an infrastructure responsibility rather than a backend one?
- (a) Rejecting an order for an out-of-stock item
- (b) Recalculating the price rather than trusting the client
- (c) Making the site reachable on a custom domain over HTTPS ✅
- (d) Deciding which users may view the admin page
> Rejecting orders, recalculating prices and authorization are server-side decisions — backend. Domains and certificates are the ground the server runs on.

### Q5 · L4
You ask AI for a new feature and it returns a working screen and a working endpoint. Based on this chapter, what are you most likely still missing?
- (a) Nothing; a screen and an endpoint are a complete feature
- (b) The error and empty states, the data migration, and any monitoring ✅
- (c) A faster algorithm
- (d) A second frontend framework
> AI defaults to the happy path. The states that only occur when things go wrong, changes to existing data, and the ability to find out it broke are the usual silent omissions.

## Go deeper
- [MDN: What is a web server?](https://developer.mozilla.org/en-US/docs/Learn/Common_questions/Web_mechanics/What_is_a_web_server)
- [Google SRE Book — Introduction](https://sre.google/sre-book/introduction/)
