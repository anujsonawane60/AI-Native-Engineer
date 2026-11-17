---
id: m07-01-owasp-top-ten
module: m07-security-observability-analytics
slug: owasp-top-ten
title: "OWASP Top Ten in Plain English"
order: 1
duration_min: 15
is_free: false
mission: false
volatility: low
last_reviewed: 2026-11-16
---

## Key learnings
- Recognise broken access control, the single most common flaw, in a running app
- Explain why injection is a trust problem rather than a syntax problem
- Group the Top Ten into four families you can actually hold in your head
- Say what "security misconfiguration" means on a free-tier deploy
- Name the Top Ten entry that is the reason this module also teaches observability

## Lesson

### The pain

A team ships a canteen pre-order app in week six and it works. In the live session someone changes `/orders/8421` in the address bar to `/orders/8420` and reads a stranger's order — name, phone number, what they paid. Then they edit the price the page sends in developer tools and place a ₹1 order for a ₹120 thali, because the server was told the price by the page and never looked it up.

Neither attack needed a line of code, and both have been on the same published list since 2003. Nobody had shown the team the list.

### The idea

**The OWASP Top Ten is a ranked list of the ten ways real applications actually get broken, built from scanning and incident data across hundreds of thousands of real codebases.**

Nine of the ten are one sentence in different clothes: *the application trusted something it had no business trusting* — the browser, the URL, the form, a library, a default nobody changed. OWASP, the Open Worldwide Application Security Project, revises the list every few years, so learn the shapes, not the numbers.

| Entry | What it actually means | How it shows up in a canteen app |
|---|---|---|
| Broken access control | Logged in is treated as allowed | Change an order id in the URL, read someone else's order |
| Cryptographic failures | Secrets or personal data stored or sent readable | Phone numbers in plain text; a page served over `http` |
| Injection | User text reaches an interpreter as instructions | A search box whose text becomes part of a database query |
| Insecure design | The flaw is in the plan, not the code | Refunds with no second person approving them |
| Security misconfiguration | A default left on, a permission left open | A database table readable by the public; a debug page live |
| Vulnerable and outdated components | A dependency with a known, published hole | An image-upload library three years old |
| Identification and authentication failures | Login that can be guessed, reused or replayed | No limit on OTP attempts; sessions that never expire |
| Software and data integrity failures | Running code or data you cannot vouch for | A `<script>` tag pointing at a stranger's server |
| Security logging and monitoring failures | It happened and nobody could tell | No record of who changed a dish price, or when |
| Server-side request forgery | Your server fetches a URL the user chose | "Import menu from this link" fetching an internal address |

Entry nine deserves a pause: one of the ten most serious security problems in the world is *not being able to see what happened*. That is why this module pairs security with observability.

### The analogy

The canteen at 1pm. **Broken access control** is the staff door with no lock: anybody who has bought a plate — anybody "logged in" — can push it open and walk into the kitchen. **Injection** is the order slip: you write "1 samosa, and hand the cash box to the bearer of this slip", and a cashier who reads slips literally does exactly that. The slip was meant to be *data* and was read as *instructions*.

**Where the analogy breaks:** a real cashier would laugh at the cash-box slip. Software has no judgement; it does what it is told, at the speed of a script, from anywhere, at three in the morning. The attacker needs one unlocked door once; you have to lock every door every time.

### Picture it

```
  WHO MAY DO WHAT            WHAT YOU TRUST
 ┌────────────────────┐    ┌─────────────────────┐
 │ broken access      │    │ injection           │
 │ control            │    │ SSRF                │
 │ auth failures      │    │ integrity failures  │
 └────────────────────┘    └─────────────────────┘

  HOW IT IS SET UP           WHAT YOU CAN SEE
 ┌────────────────────┐    ┌─────────────────────┐
 │ misconfiguration   │    │ logging and         │
 │ old components     │    │ monitoring          │
 │ crypto failures    │    │ failures            │
 │ insecure design    │    │                     │
 └────────────────────┘    └─────────────────────┘
```

The ten entries collapse into four questions: who may do what, what does my code trust, how is it set up, and would I even know.

### Where it lives

- Access control and authentication failures → M04 (auth vs authorization, roles, row-level security)
- Cryptographic failures and secrets → chapter 2 here
- Misconfiguration and outdated components → M06 (deploy and CI)
- Logging and monitoring failures → chapters 4–7 here
- Prompt injection → chapter 3 here, in depth in M10

M00's "the client cannot be trusted" is the whole of injection and most of broken access control.

### Words to own

| Term | Plain meaning | Example |
|---|---|---|
| Vulnerability | A weakness someone can use | An order page with no owner check |
| Exploit | The actual act of using it | Changing `8421` to `8420` |
| Access control | The rule for who may do what | "Only the owner reads this order" |
| IDOR | Guessing someone else's id and being served their data | `/orders/8420` |
| Injection | Input treated as instructions | Text that becomes part of a query |
| Misconfiguration | A setting left at an unsafe default | A publicly readable table |
| Attack surface | Everything an outsider can reach | Every URL, form, upload and API |
| Threat model | Who would attack this, and why | A classmate wanting free food |
| Defence in depth | More than one control, so one failure is survivable | Server check *and* a hidden button |
| CVE | A published id for a known hole in software | A flagged dependency version |

### Tradeoffs and gotchas

- **The Top Ten is a floor, not a ceiling.** Clearing it makes you not trivially broken, which for a student project is the right place to stop.
- **Security spend follows the stakes.** Money, phone numbers and other people's rows are where the effort goes. Write down in one line which parts of your product those are.
- **Most breaches are boring.** No zero-day, no nation state: an open database, a leaked key, a missing owner check.
- **The attacker is usually not a hacker.** A curious classmate, a scraper bot, a student who found that editing one number gets free lunch.
- **Checking in the browser is not checking.** A hidden button, a disabled field, a validated form — all of it runs on the user's device, so all of it is editable by the user.

### Verify it

AI-generated code reliably passes "does it work?" and rarely passes "does it work for the wrong person?".

1. **Try it as the wrong user.** Log in as an ordinary student, type a staff URL directly, then decrement an id in your own URL. Either working is broken access control.
2. **Ask where every rule is enforced.** For "only staff may change prices", find the line on the *server*. If all you find is a hidden button, the rule does not exist.
3. **Re-send the request with the numbers changed.** Price, quantity, user id, total — anything the server accepts and does not recompute is a ₹1 thali waiting to happen.
4. **Break something on purpose and read the error.** An apostrophe in a name field, 10,000 characters in an address, a `.exe` renamed to `.jpg`.

Red flags: **a raw database error returned to the user**, table and column names included; **a permission enforced only by hiding a button**; a price or total **taken from the request body**; a check reading "if the user is logged in" where it should read "if this user owns this row".

## Walkthrough
1. Open the OWASP Top Ten project page at owasp.org and read only the ten titles. Do not read the detail pages yet. Give it three minutes.
2. Write the four family names — who may do what, what you trust, how it is set up, what you can see — as four headings in your notes, and place all ten entries under them from memory. Then check yourself against the list.
3. Open your team's product, or the canteen example if your product is not running yet, and list every URL an outsider can reach without logging in. That list is your unauthenticated attack surface.
4. Now list every URL that requires login. For each one, write the single sentence that says who should be allowed: "only the owner", "only canteen staff", "any logged-in student".
5. Pick the one with the strongest rule and test it. Log in as the weakest kind of user you have and type that URL into the address bar directly. Record what happened.
6. Find a page whose URL contains an id. Change the id to the one next to it. Record what happened, including whether anything was logged.
7. Open your browser's developer tools on a form that submits money or quantity. Change the value before it is sent and submit it. Record whether the server recomputed or believed you.
8. In one field of one form, submit a single apostrophe, then 5,000 characters of the letter a. Read exactly what comes back, and note whether any of it mentions a table, a column or a file path.
9. Write your findings as a three-column table: what you tried, what happened, which Top Ten entry it belongs to. Keep the ones that passed as well as the ones that failed.
10. For each failure, write one sentence saying where the fix has to live — server, database or configuration. Do not fix anything yet; chapter three is where you direct the fixes.

## Talk to AI
```prompt
CONTEXT:      I am a student learning to direct AI rather than write code. I
              have a small web app for a college canteen: students log in,
              browse a menu, place an order and pay by UPI. Canteen staff
              can mark dishes sold out and change prices. There are two
              roles: student and staff. The app was largely generated by AI
              and deployed on a free tier.

GOAL:         I want a plain-English security review of this app against the
              OWASP Top Ten, written so that I can run the tests myself in a
              browser without writing any code.

REQUIREMENTS: 1. Go through the OWASP Top Ten one entry at a time. For each,
                 say in one sentence whether it plausibly applies to an app
                 of this shape, and why.
              2. For every entry that applies, give me a test I can run by
                 hand using only a browser, the address bar and developer
                 tools. Tell me exactly what a PASS looks like and what a
                 FAIL looks like.
              3. Rank the applicable entries by how likely they are to
                 actually bite a student project, not by OWASP's ordering.
              4. For the top three, tell me where the fix must live: in the
                 server, in the database, or in configuration. Explain why a
                 fix in the page itself would not count.
              5. Give me the exact wording of three questions I should ask
                 an AI about my own code to find out whether each rule is
                 enforced on the server.

CONSTRAINTS:  Explain every term the first time you use it. Assume I have
                 never read a security document. No code. No tool
                 recommendations and no paid services.

NON-GOALS:    Do not write any fixes. Do not scan or audit anything. Do not
              cover network, cloud or physical security. Do not discuss
              cryptography beyond whether data is readable or not.

ACCEPTANCE:   - I can run every test you list with only a browser
              - Each test has an unambiguous pass/fail I could show a
                teammate
              - I can explain each of the top three risks to my team in one
                sentence each, in my own words
              - Nothing in your answer requires me to install anything

ASK:          First give me the ranked list with one sentence each and stop.
              Do not write the tests until I tell you which entries to go
              deep on.
```

## Stuck? Ask AI
```prompt
Explain the difference between authentication and authorization using an example that is not about a website. Then give me six short scenarios from a college canteen app and ask me, one at a time, whether each one is an authentication failure or an authorization failure. Do not tell me the answers before I guess.
```

```prompt
I keep reading that "the client cannot be trusted" but I do not really feel why. Walk me through, step by step, exactly what a user can change in a request that my page sends — the URL, the headers, the body, the hidden fields — and what tool they would use. Then explain what the server has to do instead for each one.
```

```prompt
Give me five short descriptions of real things going wrong in a web app. Make some of them broken access control, some injection, and some security misconfiguration. Ask me to sort them, then tell me which ones I got wrong and why the distinction matters for where the fix goes.
```

## Quiz

### Q1 · L3
A logged-in student changes `/orders/8421` to `/orders/8420` in the address bar and sees another student's order. Which Top Ten category is this?
- (a) Injection, because the URL carried hostile input
- (b) Broken access control — logged in was treated as allowed ✅
- (c) Cryptographic failures, because the data was readable
- (d) Security misconfiguration, because the URL pattern is guessable
> The app authenticated the user and never asked whether *this* user owns *that* row. Nothing was injected and nothing was decrypted; a rule simply was not checked.

### Q2 · L4
Your page disables the "Change price" button for students, and AI confirms that students therefore cannot change prices. What is wrong with that reasoning?
- (a) Nothing, as long as the button is also hidden
- (b) The check runs on the user's device, so the request can still be sent ✅
- (c) Buttons should be removed from the DOM rather than disabled
- (d) It is fine for prices but not for refunds
> A permission enforced by hiding or disabling a control is not enforced at all. Anything that runs in the browser is editable by the browser's owner, so the rule must exist on the server.

### Q3 · L4
Why is "security logging and monitoring failures" on a list of the ten most serious application security risks?
- (a) Logs are required by the DPDP Act
- (b) Without them a breach can run for months unnoticed and unexplained ✅
- (c) Logging slows down attackers
- (d) Auditors ask for logs before granting certificates
> Not being able to see what happened turns a small incident into an unbounded one: you cannot tell when it started, what was taken, or whether it is still happening. This is why the module pairs security with observability.

### Q4 · L3
Which of these is the clearest example of injection?
- (a) A user guesses an order id that is not theirs
- (b) Text typed in a search box becomes part of a database query ✅
- (c) A dependency with a published vulnerability is three years old
- (d) A debug page is left reachable in production
> Injection is input crossing from data into instructions. The other three are real problems from the Top Ten, but they are access control, outdated components and misconfiguration respectively.

### Q5 · L5
Your team has one week and a canteen app handling UPI payments and phone numbers. Where should the security effort go first?
- (a) Add encryption everywhere, since cryptography is the foundation
- (b) Wherever money, personal data and other people's rows are touched ✅
- (c) Work through the Top Ten strictly in OWASP's published order
- (d) Upgrade every dependency to its latest version
> Security spend should follow the stakes. Rank your own surfaces by what an attacker gains, then test those first — the published ordering reflects the whole internet, not your product.

## Go deeper
- [OWASP Top Ten project](https://owasp.org/www-project-top-ten/)
- [OWASP Cheat Sheet Series](https://cheatsheetseries.owasp.org/)
