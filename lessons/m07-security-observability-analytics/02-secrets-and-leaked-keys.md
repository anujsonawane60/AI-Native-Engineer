---
id: m07-02-secrets-and-leaked-keys
module: m07-security-observability-analytics
slug: secrets-and-leaked-keys
title: "Secrets, Environment Variables and Leaked API Keys"
order: 2
duration_min: 25
is_free: false
mission: false
volatility: low
last_reviewed: 2026-11-16
---

## Key learnings
- Tell the difference between a secret, a configuration value and a public identifier
- Explain why a key in frontend code is public no matter how it was stored
- Run the rotation sequence in the right order when a key leaks
- Place each of your own secrets in the one location that is correct for it
- Scope and limit a key so that a leak costs you less

## Lesson

### The pain

A student builds an AI feature over a weekend. It works beautifully. They push the project, go to sleep, and wake up to two emails: one from the code host saying a secret was detected in a public repository, and one from the AI provider saying usage is up 400 times. Somebody's bot found the key within eleven minutes of the push and had been generating content with it all night, billed to a card belonging to the student's father.

A second team avoids that mistake entirely. They put the key in an environment variable like they were told. But the variable name begins with a prefix that means "send this to the browser", so the key was compiled into the JavaScript that every visitor downloads. It is in the public bundle, visible in developer tools, on a page anyone can load. It was never in the repository at all, and it is just as leaked.

Both teams did one thing right and one thing wrong. The gap between them is this chapter.

### The idea

**A secret is anything that proves you are allowed to act. Protect the credential, not the code around it.**

Keys do not leak because people are careless with code. They leak because a secret has to travel — from your laptop, into a deploy platform, into a running server — and every stop on that journey is a place it can be left behind. The job is to know the journey, and to know which stops are safe.

### The analogy

The canteen has a till, and the till has a key.

The key lives on the manager's keyring. Staff who need to open the till are handed it for a shift and give it back. It is never left in the lock, never taped under the counter, never photocopied "just for convenience". And if it goes missing, the canteen does not look for it first — it changes the lock first and looks afterwards, because a missing key and a copied key are the same thing until proven otherwise.

The second part matters more than the first. Different keys for different things: the till key is not the store room key is not the bank login. When one goes, one thing is exposed.

**Where the analogy breaks:** a physical key has to be in one place at a time, and you notice when it is gone. A copied API key works from Mumbai and Moldova simultaneously, the original still works perfectly, and nothing tells you — except, eventually, the bill. There is no "missing"; there is only "also somewhere else". Which is why detection, in this world, is a billing alert rather than an empty hook.

### Not every value is a secret

Three categories, and confusing them is where most of the cost comes from.

| Category | Who may see it | Example |
|---|---|---|
| **Secret** | Your server only | Payment provider secret key, database password, AI provider API key |
| **Public identifier** | Anyone, by design | Payment provider *publishable* key, your app's URL, an analytics project id |
| **Configuration** | Anyone, harmless | Page size, feature flag, currency code, time zone |

Payment providers hand you two keys precisely to make this line visible: one is meant for the browser and can only start a payment, the other lives on your server and can move money. If you cannot say which of your own values are in which row, start there — write the three columns out and sort them.

### Picture it

```
  YOUR LAPTOP            DEPLOY PLATFORM          RUNNING SERVER
 ┌──────────────┐       ┌────────────────┐       ┌──────────────┐
 │ .env.local   │──┐    │ env var store  │──────▶│ process env  │
 │ (gitignored) │  │    │ (encrypted)    │       │ reads key    │
 └──────────────┘  │    └────────────────┘       └──────┬───────┘
                   │             ▲                      │ key in
         NEVER     │             │ you paste it         │ header
         committed │             │ once                 ▼
                   ▼             │                 ┌──────────┐
            ┌────────────┐       │                 │ provider │
            │ git history│───────┘                 └──────────┘
            │   ✗ ✗ ✗    │
            └────────────┘        BROWSER BUNDLE ✗ ✗ ✗
```

A secret travels from a gitignored file on your laptop to the deploy platform's encrypted store, and from there into the running server's memory, where it is attached to outbound requests. It must never enter git history and must never enter the browser bundle.

### Why a key in the browser is always public

This is the part that surprises people, so it is worth being blunt.

Anything the browser needs in order to run, the browser downloads. Your JavaScript is a file on the user's machine. Minifying it, obfuscating it, loading it late, putting it in an environment variable with a `PUBLIC` prefix, base64-encoding it — none of these change the fact that the file arrives on the user's computer and can be read with two clicks in developer tools.

There is no way to give the browser a secret and keep it secret. If the browser can use it, the user has it.

So when AI generates a page that calls an AI provider or a payment API directly from the browser with a key, the fix is never "hide the key better". The fix is structural: the browser calls *your* server, your server holds the key and calls the provider. That server step is also where you put the rate limit, the spend cap and the log line — and the reason M00 says "the client cannot be trusted" is that there is no version of this where the client gets to hold the credential.

### When it leaks: the order matters

A leaked key is not a cleanup task, it is a live incident, and the sequence is fixed.

1. **Revoke the key at the provider.** Now, before anything else. This is the only step that actually stops the bleeding; everything else is paperwork. A revoked key cannot be used by anyone, including whoever copied it.
2. **Issue a new key and put it in the deploy platform's store.** Not in the code. Not in a message to your teammate.
3. **Check what was done with the old one.** Usage logs, spend, any rows created or deleted. Write down what you find; you will be asked.
4. **Set or tighten a billing alert and a spend cap**, so the next one costs you ₹200 instead of ₹40,000. This is M00's billing-alert chapter becoming real.
5. **Only then clean history.** As M01 said, a secret committed once is in that commit forever, and deleting the file in a later commit does not remove it from the earlier one. Rewriting history is worth doing; it is not urgent, because the key is already dead.

The mistake almost everyone makes is doing step five first — deleting the file, force-pushing, feeling relieved — while the key is still valid. Rotate first. Tidy later.

### Where it lives

A secret touches four places, and a different module owns each. M00 is where you created these accounts, set billing alerts, and learned that an agent reads anything in the folder. M01 is `.gitignore` on the first commit and the permanence of history. M06 is the deploy platform and CI, where the real values live and where a build step can inline one into the bundle. M04 is payments, where the two-key split is most visible and confusing them involves money.

Chapter one's "cryptographic failures" is the Top Ten entry this sits under, and the next chapter is where AI puts a key in a page unasked.

### Words to own

| Term | Plain meaning | Example |
|---|---|---|
| Secret | Anything that proves you may act | Database password, provider secret key |
| Environment variable | A value handed to a program when it starts | `DATABASE_URL` |
| `.env` | A local file of environment variables, gitignored | Your laptop only |
| Rotate | Replace a credential with a new one | New key issued, old one revoked |
| Revoke | Make a credential stop working, immediately | The first step of any leak |
| Scope | What a credential is allowed to do | Read-only; one table; test mode |
| Secret scanning | Automated search for keys in code | Your host emails you within minutes |
| Spend cap | A hard ceiling on what an account can bill | ₹500 a month on an AI provider |
| Test-mode key | A credential that cannot touch real money | What you use all through this program |
| Bundle | The JavaScript file the browser downloads | Where a `PUBLIC_` key ends up |

### Tradeoffs and gotchas

**A gitignored `.env` is protected from git, not from anything else.** It is an ordinary file in your folder. An agent working there reads it. A screen-share shows it. A screenshot in your demo deck immortalises it.

**Convenience leaks are the common ones.** Pasting the key into the team WhatsApp group so a teammate can run the project. Hard-coding it "just for five minutes" to test something. Committing it in a branch nobody will merge — which is still history, still pushed, still scanned.

**Use test-mode keys for everything in this program.** Payment providers give you keys that behave identically and move no money. The whole course is designed around free tiers and test modes precisely so that your worst day costs you nothing. A leaked test key is an inconvenience; a leaked live key is a phone call home.

**Scope before you worry about secrecy.** A key that can only read, only one table, only from your server's region, with a ₹500 cap, is a key whose leak is survivable. Security people call this blast radius, and it is more useful than extra caution — because caution fails occasionally and scope fails never.

**Logs are a leak channel.** A tired debugging line that prints request headers prints the authorization header. Chapter five covers redaction; for now, "it is only in the logs" is not safety, especially when logs go to a third party.

**Rotating on a schedule beats rotating in a panic.** Once a term, every key, with a written five-line runbook. Then the day you need it, you follow a procedure rather than inventing one at midnight.

### Verify it

AI will cheerfully put a key wherever is shortest. Five checks, the first two of which find almost everything.

1. **Search the whole project for the key's first eight characters.** Keys have recognisable prefixes for exactly this reason. Anything found outside `.env.local` and the deploy platform is a finding. Then search git history, not just the current files.
2. **Load the deployed page and search the downloaded JavaScript for the same eight characters.** Open developer tools, look at the network tab and the sources, and use the find box. **A key in frontend code is the single most common AI-generated security flaw in this course,** and this is how you catch it in thirty seconds.
3. **Check what the browser is calling.** In the network tab, does any request go directly to a provider's domain? If yes, the browser is holding a credential. It must call your own server instead.
4. **Check the prefix on every environment variable.** Any variable name containing `PUBLIC` is, by the framework's own definition, shipped to the browser. If a secret is in one, it is public, regardless of where you typed it.
5. **Confirm there is a billing alert and a spend cap on every paid provider.** Not a plan to add one. Open the account and read the number.

Red flags, in order of how badly they end: a key visible in the browser bundle; a key in any commit in history; a key pasted in a chat or a screenshot; a log line that prints request headers; and a single all-powerful key used by the laptop, the CI pipeline and production alike.

## Walkthrough
1. Make a list of every credential your project uses: database, auth provider, payment provider, AI provider, email, storage. Write them in one column.
2. Beside each, write Secret, Public identifier or Configuration. Where you are unsure, open the provider's docs and find out — guessing here is the mistake that costs money.
3. For each Secret, write where it currently lives. Be honest and specific: "in `.env.local`", "in the deploy platform", "in our group chat", "hard-coded in a file I cannot remember".
4. Confirm `.env` and `.env.local` are in `.gitignore`, and then confirm it worked by running `git log --all -- .env` and checking that it returns nothing. Anything it returns is a key you rotate today.
5. Pick one secret and search your whole project for its first eight characters, including every file the search tool wants to skip. Record every place it appears.
6. Deploy the app, open it in a browser, open developer tools, and search the downloaded JavaScript for those same eight characters. Write down the result even if it is clean.
7. In the network tab, reload the page and list every domain the browser talks to. Flag any that is a provider you hold a secret for.
8. Open each paid provider's billing page. Set a spend cap if one exists, and set an alert at a number you would be comfortable paying twice. Screenshot it.
9. Write a five-line rotation runbook for your team: revoke, reissue, redeploy, review usage, clean history. Put it in your repository as a file, not in somebody's memory.
10. Rotate one real key end to end, following your own runbook, and time it. The first time takes twenty minutes; knowing that it takes twenty minutes is the point of the exercise.
11. Note in your decision log which keys are test-mode and which are live, and who on the team may touch the live ones.

## Talk to AI
```prompt
CONTEXT:      I am a student building a web app with a team of three. We use
              a hosted database, an email service, a payment provider in
              test mode, and an AI provider. The app is deployed on a free
              tier. Most of the code was generated by AI, and I cannot write
              code myself — I direct AI and verify its output.

GOAL:         I want a complete inventory of where every credential in a
              project like mine should live, and a set of checks I can run
              by hand to prove none of them have leaked.

REQUIREMENTS: 1. Explain the difference between a secret, a public
                 identifier and ordinary configuration, with two examples of
                 each from the services I listed.
              2. For each service, tell me which of its credentials are
                 meant for the browser and which must stay on the server,
                 and how I can tell from the provider's own naming.
              3. Give me a step-by-step manual check, using only a browser
                 and my project folder, that proves no secret is in my
                 frontend bundle and none is in git history.
              4. Give me the exact order of steps for when a key leaks, and
                 explain why cleaning git history is not the first step.
              5. Tell me what "scoping" a key means for each of these
                 services and what the narrowest useful scope would be.
              6. Tell me what a billing alert and a spend cap are, and what
                 number a student team should set them to.

CONSTRAINTS:  Free tiers and test modes only. Assume I will not install any
              security tooling. Explain every term on first use. Give me no
              code; give me instructions I can follow in a browser.

NON-GOALS:    Do not recommend a paid secrets manager. Do not cover
              enterprise key rotation, hardware tokens or compliance
              frameworks. Do not write or modify any files.

ACCEPTANCE:   - I can sort all my own credentials into the three categories
              - I can state, for each secret, the one place it belongs
              - I can run every check with a browser and a file search
              - I can recite the leak sequence in the right order
              - I know what to do in the first sixty seconds of a leak

ASK:          Before answering, ask me up to five questions about my setup
              that would change your advice. Then give me the inventory
              table first and wait before writing the checks.
```

## Stuck? Ask AI
```prompt
Explain, as if I have never used a browser's developer tools, exactly how someone can read an API key that my web page uses. Walk me through what they click, in order, and what they see. Then explain why minifying, obfuscating or base64-encoding the key does not help.
```

```prompt
I think I committed a secret to git last week and I have already deleted the file and pushed. Explain what is still exposed and why. Then give me the correct sequence of actions in priority order, and tell me which single thing I should do in the next sixty seconds if I can only do one.
```

```prompt
Give me eight values that might appear in a web app's configuration — some genuinely secret, some public by design, some just settings. Ask me to sort them into those three groups one at a time, and after each answer tell me what would actually happen if that value became public.
```

## Quiz

### Q1 · L4
AI puts your AI provider key in an environment variable whose name starts with `NEXT_PUBLIC_` and the app works. What is the status of that key?
- (a) Safe, because environment variables are not in the source code
- (b) Public — the prefix means it is compiled into the browser bundle ✅
- (c) Safe, because it is only readable by the deploy platform
- (d) Safe as long as the repository is private
> Framework prefixes like `PUBLIC` are an explicit instruction to ship the value to the browser. It never touched git and is leaked anyway, which is why the bundle check matters as much as the history check.

### Q2 · L3
Your key appears in a public repository. What is the first thing you do?
- (a) Delete the file and force-push to rewrite history
- (b) Revoke the key at the provider ✅
- (c) Make the repository private
- (d) Email your teammates to stop pushing
> Only revocation stops the key being used. History rewriting, visibility changes and messages all leave a working credential in someone else's hands while you tidy up.

### Q3 · L4
A page calls an AI provider directly from the browser. AI suggests obfuscating the key so it is harder to find. Why is this not a fix?
- (a) Obfuscation breaks on some browsers
- (b) If the browser can use the credential, the user has the credential ✅
- (c) Obfuscation is against the provider's terms of service
- (d) It would still appear in git history
> The browser must be able to read a value in order to send it, so no amount of hiding changes who holds it. The structural fix is a server step that holds the key and that you can rate-limit and log.

### Q4 · L3
Why do payment providers issue two different keys?
- (a) One is for testing and one is for production
- (b) To make the line between browser-safe and server-only visible ✅
- (c) To let two developers work at the same time
- (d) To support both web and mobile clients
> The publishable key can only start a payment and is meant for the browser; the secret key can move money and must stay on your server. Test and live modes are a separate axis entirely.

### Q5 · L5
Two teams leak a key. Team A's key is read-only, scoped to one table, capped at ₹500 a month. Team B's key is a single all-powerful key shared by laptops, CI and production. What does this tell you about where to spend effort?
- (a) Nothing; both leaked, so both failed equally
- (b) Scoping and caps decide what a leak costs, so they beat extra caution ✅
- (c) Team B should rotate more often to compensate
- (d) Team A was lucky rather than better prepared
> Leaks are a question of when. Narrow scope, separate keys per environment and a hard spend cap convert an incident into an inconvenience, and they keep working on the day somebody is careless.

## Go deeper
- [GitHub Docs: about secret scanning](https://docs.github.com/en/code-security/secret-scanning/about-secret-scanning)
- [OWASP Secrets Management Cheat Sheet](https://cheatsheetseries.owasp.org/cheatsheets/Secrets_Management_Cheat_Sheet.html)
