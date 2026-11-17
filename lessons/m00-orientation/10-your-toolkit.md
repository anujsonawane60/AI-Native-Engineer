---
id: m00-10-your-toolkit
module: m00-orientation
slug: your-toolkit
title: "Your Toolkit: Accounts, API Keys, Free Tiers and Billing Alerts"
order: 10
duration_min: 25
is_free: false
mission: true
volatility: high
last_reviewed: 2026-11-16
---

## Key learnings
- Set up every account this program needs, on free tiers only
- Explain what an API key is and why it must never reach the browser
- Read a free tier's limits and know what happens when you cross one
- Put billing alerts in place before you can be charged
- Secure your own accounts well enough to be trusted with a real product

## Lesson

### The pain

A student pushes a project to a public repository with their AI provider's API key in a config file. Scrapers find keys on GitHub within minutes; by morning someone else has run ₹40,000 of requests on their card.

A second student demos something genuinely good, and three weeks later the link is dead — their free database paused for inactivity and was deleted, with no backup. Neither story is about skill. Both are about twenty minutes of setup nobody insisted on.

### The idea

**Your accounts are infrastructure. Set them up deliberately once, with limits and alerts, before you build anything on top of them.**

Everything here runs on free tiers. That is a design constraint, not a compromise — but free tiers have edges, and crossing one either stops your product or starts a bill.

### The analogy

Opening a canteen stall. Before you cook you need a licence, a gas connection and a bank account — unexciting, and every one blocking. The gas connection has a meter: a monthly allowance, then you pay per cylinder, and a careful owner gets a warning before crossing the line. An API key is the key to that connection: leave copies around the market and someone else uses your gas on your bill.

**Where the analogy breaks:** steal a gas cylinder and it is obviously gone. A stolen API key leaves everything working perfectly while someone runs up charges in another country — no symptom until the bill, which is why the key rules are absolute rather than advisory.

### Picture it

```
       ┌─────────────────────────────────────────┐
       │   PUBLIC — anyone can read this         │
       │                                         │
       │   browser / phone app                   │
       │   · page source      · network tab      │
       │   · bundled JS       · app package      │
       │                                         │
       │   ✗ NEVER put a secret key here         │
       └──────────────────┬──────────────────────┘
                          │ request (no key)
                          ▼
       ┌─────────────────────────────────────────┐
       │   PRIVATE — only your server sees this  │
       │                                         │
       │   server  ·  .env  ·  platform secrets  │
       │                                         │
       │   ✓ the key lives here, and only here   │
       └──────────────────┬──────────────────────┘
                          │ request + key
                          ▼
                   AI / payment / database
                        provider
```

There is exactly one correct place for a secret key: the private side. "But it is minified" and "but it is in a frontend environment variable" are both still the public side — a frontend variable is compiled into the file the browser downloads.

Two kinds, and confusing them is the expensive mistake:

| Kind | Example | Safe in the browser? |
|---|---|---|
| **Public / publishable** | Stripe publishable key, Supabase anon key | Yes — designed for it, and limited in what it can do |
| **Secret** | AI provider key, Stripe secret key, Supabase service-role key | **Never.** Full account power |

If a key's name contains `secret`, `service_role`, `private` or `sk_`, it belongs on the server. If unsure, assume secret.

### Where it lives

Accounts, keys and quotas sit in the infrastructure box from chapter 3 — which is why a forgotten billing alert or a paused database takes the whole product down.

You will also hold credentials for a product other people use, so the bar is higher than for personal email: 2FA on GitHub and your email today (email is the recovery path for everything else), a password manager, recovery codes stored off the phone with the authenticator, and a careful read of what "Sign in with GitHub" grants.

### Words to own

| Term | Plain meaning | Example |
|---|---|---|
| API key | A string proving a request is yours | How the provider bills you |
| Secret / publishable | Server-only versus browser-safe | `sk_` versus `pk_` |
| Rotate | Replace a key and revoke the old one | What to do after a leak |
| Environment variable | A value supplied outside the code | `OPENAI_API_KEY` |
| `.env` | A local file holding those values | Never committed |
| Free tier | Usage a provider gives at no cost | 500 MB database |
| Quota | The ceiling on usage | 10,000 requests/month |
| Rate limit | A ceiling on speed | 20 requests/minute |
| Overage | Usage past the free limit | Where bills appear |
| Spending cap | A hard ceiling on charges | ₹100 maximum |
| 2FA | A second proof of identity | Authenticator app |

### Tradeoffs and gotchas

**A public repository is a public key.** Scrapers test new commits within minutes, and deleting the file does not help because history stays fetchable. After a leak: new key, revoke the old one, then clean up — every minute the old key lives is billable.

**Rate limits and quotas are different things.** A quota is the monthly total; a rate limit is how fast you may go. Hitting the rate limit looks like your app breaking under load while the quota is nowhere near exhausted.

**Free tiers shrink.** Every limit quoted in this program may be out of date by the time you read it. Check the provider's current pricing page, not a lesson or an AI answer — pricing sits squarely in the knowledge-cutoff failure mode.

**Test mode is a real feature, not a toy.** Payment providers give you keys that move no money and simulate success, failure and duplicate webhooks. All of M04 happens there.

### Verify it

**Before you add any service,** answer four questions:

1. **What is the limit, and in what unit?** Requests per month, rows, megabytes, build minutes.
2. **What happens when I cross it?** It stops, slows, or charges your card — find the sentence in the docs saying which.
3. **Does it expire or pause?** Free databases commonly pause after a week idle, then delete after a month.
4. **Can I cap it?** A hard limit, even ₹100, turns an unbounded risk into a known one — set it when you create the account, not later.

**Before you call the setup done,** prove four things:

1. **Search your repositories for secrets** — `sk_`, `service_role`, `API_KEY`, any long random string.
2. **Confirm `.env` is gitignored and was never committed.** `git log --all -- .env` should return nothing; anything returned is a key to rotate.
3. **View your deployed site's source and network tab and search for your key.** If it appears, it is public.
4. **Confirm a billing alert exists** on every service holding a payment method.

Two red flags when AI sets up a service: **a key written into a code file** rather than read from an environment variable, and **a frontend calling the provider directly.**

## Walkthrough
1. **GitHub.** Create the account if you do not have one. Turn on 2FA. Save the recovery codes somewhere that is not your phone. Apply for the Student Developer Pack with your college email.
2. **AI assistants.** Set up at least two of Claude, ChatGPT, Gemini or Google AI Studio. Two, deliberately — you will compare them in M09, and having a second one running is the fastest way to tell whether a bad answer is the model or your brief.
3. **An AI coding agent.** Install one in your editor or terminal. Open its permission settings and set the allowlist you decided on in the previous chapter.
4. **Deploy platform.** Create an account on a free static/app host and connect it to your GitHub. Do not add a card.
5. **Database / backend.** Create a free Postgres or backend-as-a-service project. Find and write down two numbers: the storage limit, and the inactivity period after which it pauses. Put a reminder in your calendar before that date.
6. **Design tools.** Set up an account on a diagramming tool and a design tool. You will draw architecture diagrams from M01 onward.
7. **Billing alerts.** On every account above that can accept a payment method, set the lowest alert threshold available. Where a hard spending cap exists, set it. Where neither exists and a card is required, note that and prefer prepaid credits.
8. **A secrets habit.** Create a practice folder with a `.env` file, add `.env` to `.gitignore`, commit, and confirm with `git status` that `.env` is not staged. Do this once deliberately so it is muscle memory.
9. **Prove the negative.** Run `git log --all -- .env` in each project you already have. Anything returned is a key to rotate today.
10. **Write the inventory.** One table: service, what it is for, free limit, what happens at the limit, whether a card is attached, alert set yes/no.

## Talk to AI
```prompt
I am setting up accounts for a six-month student project and I must stay on free tiers. I need to understand the risk of each service before I attach anything to it.

The services I am planning to use are: [list them]

For each one, give me a row with:
1. What the free tier includes, in its own units
2. What happens when I exceed it — stops, throttles, or charges
3. Whether it pauses or deletes after inactivity, and after how long
4. Whether a hard spending cap is available
5. Whether it issues both publishable and secret keys, and which is which
6. The single most common way a student gets an unexpected bill from it

Then tell me which of these services I should be most careful with and why, and which numbers you are least confident about so I can check the pricing page myself.

Important: your training data has a cutoff and these limits change often. Flag every figure you are not certain is current rather than stating it confidently.
```

## Mission

**Objective.** Have every account this program needs, on free tiers, with billing alerts set and no secret reachable from a browser or a repository — and an inventory that proves it.

**Steps.**
1. **Frame** — list the services you actually need for the next six months. Resist adding more.
2. **Create** — work through the walkthrough above. Free tiers only. No cards where avoidable.
3. **Limit** — set a billing alert, and a hard spending cap where one exists, on everything that can charge you.
4. **Secure** — 2FA on GitHub and your email. Recovery codes stored off-device. A password manager in use.
5. **Verify** — run the four checks from "Verify it". Specifically run `git log --all -- .env` on every repository you already have.
6. **Document** — build the inventory table.

**Upload.**
- Your inventory table: service, purpose, free limit, behaviour at the limit, card attached, alert set → text or screenshot
- Screenshots of billing alerts or spending caps actually configured, on at least two services
- A screenshot of 2FA enabled on GitHub
- A screenshot of `git status` showing `.env` present but not staged, with `.gitignore` visible
- The output of `git log --all -- .env` on your main project, showing no results
- Your AI conversation about free-tier limits → share link in `ai_chat_url`
- Reflection, 150 words max → which limit would bite you first and why, and one figure the AI got wrong or was unsure about

**Self-check.** Tick every box before you submit.
- [ ] Every account is on a free tier and I know its limit in its own units
- [ ] For each service I can say what happens when I cross the limit: stop, throttle, or charge
- [ ] I know which of my services pause or delete after inactivity, and when
- [ ] A billing alert or hard cap exists on everything that can charge me
- [ ] 2FA is on for GitHub and my email, and recovery codes are stored off-device
- [ ] `.env` is gitignored, and `git log --all -- .env` returns nothing
- [ ] No secret key appears in any repository or any deployed page's source
- [ ] I can state which of my keys are publishable and which are secret
- [ ] I checked at least two limits against the provider's own pricing page, not an AI answer
- [ ] My reflection names a specific figure the AI was wrong or uncertain about

**Stretch.** Deliberately leak a *throwaway* key: generate a new key on a free service with a hard spending cap, commit it to a public test repository, and see how long until the provider emails you about an exposed secret. Then revoke it. Write 100 words on what you observed. Only do this with a key you create for this purpose and revoke immediately — never with a real one.

## Stuck? Ask AI
```prompt
Explain the difference between a publishable key and a secret key, with examples from three different services. Then give me five key names and ask me to sort them into "safe in the browser" and "server only" without telling me the answers first.
```

```prompt
I think I committed an API key to a public GitHub repository. Walk me through exactly what to do, in order, and explain why deleting the file is not sufficient. Tell me what to do first if I only have two minutes.
```

```prompt
Explain the difference between a quota and a rate limit using a canteen analogy. Then describe how each one would look to a user if my app hit it, and why hitting a rate limit is often worse than hitting a quota.
```

## Quiz

### Q1 · L3
Your frontend code reads the AI provider key from an environment variable rather than hard-coding it, then calls the provider directly from the browser. Is the key safe?
- (a) Yes — environment variables are not visible to users
- (b) No — a frontend environment variable is compiled into the file the browser downloads ✅
- (c) Yes, as long as the site uses HTTPS
- (d) Only unsafe if the repository is public
> Frontend environment variables are build-time substitutions baked into the bundle. HTTPS protects the data in transit, not from the user who received it. The call must go through your server.

### Q2 · L4
You discover a secret key was committed to a public repository last week. What is the first thing to do?
- (a) Delete the file and commit the deletion
- (b) Rewrite git history to remove the commit
- (c) Rotate the key — generate a new one and revoke the old ✅
- (d) Make the repository private
> Scrapers found it within minutes of the push, so the key must be assumed compromised. Every minute it stays valid is billable. Cleaning history is worth doing afterwards, not first.

### Q3 · L4
Which question about a free tier do people most often skip, at the greatest cost?
- (a) What the limit is
- (b) What happens when the limit is crossed ✅
- (c) Which region it runs in
- (d) Whether it supports Postgres
> "Stops until next month" and "charges your card" are both normal answers, and the difference between them is the difference between a broken demo and an unexpected bill.

### Q4 · L3
A student's working demo is dead three weeks after a successful presentation, with no backup. What most likely happened?
- (a) The free tier's monthly request quota was exhausted
- (b) The free database paused for inactivity and was then deleted ✅
- (c) The deploy platform revoked the account
- (d) The API key expired
> Free databases commonly pause after about a week idle and are deleted after about a month. This is the inactivity question, and it is why the mission asks you to write the date in a calendar.

### Q5 · L4
Why is AI especially unreliable about free-tier limits and pricing?
- (a) It is not allowed to discuss commercial terms
- (b) These change frequently and sit squarely in the knowledge-cutoff failure mode ✅
- (c) Pricing pages block AI from reading them
- (d) It confuses quotas with rate limits
> Limits and prices move constantly, so a confidently quoted figure may be a year old. Anything versioned or priced gets checked against the provider's own current page.

## Go deeper
- [GitHub Student Developer Pack](https://education.github.com/pack)
- [GitHub: About secret scanning](https://docs.github.com/en/code-security/secret-scanning/about-secret-scanning)
