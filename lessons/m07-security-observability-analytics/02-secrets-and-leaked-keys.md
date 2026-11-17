---
id: m07-02-secrets-and-leaked-keys
module: m07-security-observability-analytics
slug: secrets-and-leaked-keys
title: "Secrets, Environment Variables and Leaked API Keys"
order: 2
duration_min: 15
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

A student pushes an AI feature and goes to sleep. They wake to two emails: the code host detected a secret in a public repository, and the AI provider reports usage up 400 times. A bot found the key eleven minutes after the push and generated content with it all night, billed to the student's father's card.

A second team put their key in an environment variable exactly as instructed — but the variable name began with a prefix meaning "send this to the browser", so the key was compiled into the JavaScript every visitor downloads. It never entered the repository at all, and it is just as leaked.

### The idea

**A secret is anything that proves you are allowed to act. Protect the credential, not the code around it.**

Keys leak because a secret has to travel — from your laptop, into a deploy platform, into a running server — and every stop on that journey is a place it can be left behind. Knowing which stops are safe starts with knowing that not every value is a secret.

| Category | Who may see it | Example |
|---|---|---|
| **Secret** | Your server only | Payment provider secret key, database password, AI provider API key |
| **Public identifier** | Anyone, by design | Payment provider *publishable* key, your app's URL, an analytics project id |
| **Configuration** | Anyone, harmless | Page size, feature flag, currency code, time zone |

Payment providers hand you two keys precisely to make this line visible: the publishable one is meant for the browser and can only start a payment, the secret one lives on your server and can move money.

### The analogy

The canteen till key lives on the manager's keyring, handed out for a shift and given back — never left in the lock, never photocopied "just for convenience". If it goes missing the canteen changes the lock first and looks afterwards, because a missing key and a copied key are the same thing until proven otherwise. And the till key is not the store room key: when one goes, one thing is exposed.

**Where the analogy breaks:** a physical key is in one place at a time, and you notice when it is gone. A copied API key works from Mumbai and Moldova simultaneously, the original still works perfectly, and nothing tells you — except, eventually, the bill. There is no "missing", only "also somewhere else", so detection here is a billing alert rather than an empty hook.

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

The bundle is marked three times for a reason. Anything the browser needs in order to run, the browser downloads; minifying, obfuscating, base64-encoding or a `PUBLIC_` prefix change nothing. **If the browser can use it, the user has it.** The fix is structural, not better hiding: the browser calls *your* server, and your server holds the key, the rate limit and the spend cap.

### Where it lives

- M00 — accounts and billing alerts, and the fact that an agent reads anything in the folder
- M01 — `.gitignore` on the first commit, and the permanence of history
- M06 — deploy platform and CI, where a build step can inline a secret into the bundle
- M04 — payments, where the two-key split is most visible
- Chapter 1's "cryptographic failures" is the Top Ten entry this sits under; chapter 3 is where AI puts a key in a page unasked

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

- **A leak is an incident with a fixed order:** revoke at the provider, reissue into the deploy store, check what the old key did, set a spend cap, and *only then* clean git history. Almost everyone does the last step first, force-pushing while the key is still valid. Rotate first, tidy later.
- **A gitignored `.env` is protected from git, not from anything else.** An agent in the folder reads it, a screen-share shows it, a demo screenshot immortalises it. So does the team WhatsApp group, and so does a branch nobody merged.
- **Use test-mode keys throughout this program.** A leaked test key is an inconvenience; a leaked live key is a phone call home.
- **Scope beats caution.** A read-only key, one table, ₹500 cap, is a leak you survive. Security people call this blast radius — caution fails occasionally, scope fails never.
- **Logs are a leak channel.** A debugging line that prints request headers prints the authorization header too.

### Verify it

AI will put a key wherever is shortest. The first two checks find almost everything.

1. **Search the whole project for the key's first eight characters**, then search git history too. Anything outside `.env.local` and the deploy platform is a finding.
2. **Search the deployed page's downloaded JavaScript for those same eight characters.** **A key in frontend code is the single most common AI-generated security flaw in this course,** and developer tools catch it in thirty seconds.
3. **Check what the browser is calling.** Any network-tab request going straight to a provider's domain means the browser is holding a credential.
4. **Check the prefix on every environment variable.** Any name containing `PUBLIC` is, by the framework's own definition, shipped to the browser.
5. **Confirm a billing alert and a spend cap on every paid provider.** Open the account and read the number.

Red flags, in order of how badly they end: a key visible in the browser bundle; a key in any commit in history; a key pasted in a chat or a screenshot; a log line that prints request headers; one all-powerful key shared by laptop, CI and production.

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
