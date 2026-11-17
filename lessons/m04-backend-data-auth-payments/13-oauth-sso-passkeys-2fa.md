---
id: m04-13-oauth-sso-passkeys-2fa
module: m04-backend-data-auth-payments
slug: oauth-sso-passkeys-2fa
title: "OAuth, SSO, Passkeys and 2FA"
order: 13
duration_min: 25
is_free: false
mission: true
volatility: medium
last_reviewed: 2026-11-16
---

## Key learnings
- Trace an OAuth login and say which party trusts which, and why
- Explain why "Sign in with Google" gives you identity and no permissions
- Choose a second factor that works on a budget Android phone on slow 4G
- Decide what happens when the same person signs in two different ways
- Verify a social login by attacking it as a logged-in ordinary user

## Lesson

### The pain

The canteen team adds "Sign in with Google" in forty minutes. Then the admin page turns out to be still open, because Google said who the student is and nothing about what she may do. A student who signs in two ways with the same address gets a second account. And a judge asks what protects a staff account that reads 400 phone numbers: a Google password that might be `priya2003`.

### The idea

**OAuth lets a user prove their identity to you using an account they already have somewhere else, without ever giving you their password — and that is all it does.**

A redirect dance: your app sends the student to Google, which authenticates them, asks consent, and redirects back with a short-lived **code**. Your *server* exchanges that code plus your client secret for a token and reads the identity from it — you never see the password.

### The analogy

A student wants a canteen tab without handing over her documents. She takes its slip to the registrar, who checks her, asks consent, and gives her a sealed one-time chit. The manager never opens the chit — he sends it to the registrar himself, who confirms "Priya Sharma, roll 4471".

**Where the analogy breaks:** the chit can be stolen between offices, so the real exchange is server-to-server with a secret the student never touches, and expires in seconds.

### Picture it

```
 student      your app          Google         your server
   │            │                 │                │
   │─ tap ─────▶│                 │                │
   │            │─ redirect ─────▶│                │
   │◀────── Google login page ────│                │
   │─ password + consent ────────▶│                │
   │            │◀── redirect with short-lived ────│
   │            │    code ────────────────────────▶│
   │            │                 │◀─ code +       │
   │            │                 │   client secret│
   │            │                 │── id token ───▶│
   │            │                 │                │
   │            │   ┌─────────────────────────┐    │
   │            │   │ NOW DO YOUR OWN WORK:   │    │
   │            │   │ find or create the user │    │
   │            │   │ assign a role YOURSELF  │    │
   │            │   │ start your own session  │    │
   │            │   └─────────────────────────┘    │
   │◀─ logged in, role assigned by you ────────────│
```

Google proves who the student is and hands back a short-lived code; your server exchanges it using a secret the browser never sees, and then does the part Google cannot do — deciding what this person is allowed to do in your app.

### Where it lives

OAuth replaces chapter 12's authentication gate and nothing else; every interior door is still yours. The box in that diagram is the lesson — your server finds or creates the user row, assigns a role, and starts its own session, and **the role comes from your database, never from the token.** Next: 14 builds those roles; M07 adds the DPDP Act, since you now hold personal data.

### Words to own

| Term | Plain meaning | Example |
|---|---|---|
| OAuth | Protocol for logging in with another account | Sign in with Google |
| Identity provider | The service that proves who someone is | Google, GitHub |
| Authorization code | A short-lived chit exchanged server-side | expires in seconds |
| Client secret | Your app's secret, never in the browser | server-only |
| ID token | Signed proof of identity and its claims | name, email |
| Redirect URI | The exact address the provider sends back to | must be allow-listed |
| SSO | One login across many applications | college accounts |
| 2FA / MFA | A second, different kind of proof | rotating 6-digit code |
| Passkey | Device-held key unlocked by fingerprint or face | no password at all |
| Account linking | Joining two login methods to one person | Google plus OTP |

### Tradeoffs and gotchas

Second factors, judged against real phones:

| Factor | How it works | Honest verdict for your users |
|---|---|---|
| SMS OTP | A code texted to the phone | Familiar everywhere in India, works on any handset, and the weakest option — SIM swap and delivery delays on bad networks |
| Authenticator app | A 6-digit code from a shared secret, rotating every 30 seconds | Strong, free, works offline, needs the user to install an app and keep a backup |
| Email OTP | A code sent to the inbox | Only as strong as the email account; fine as a fallback |
| Passkey | A key in the device's secure hardware, unlocked by fingerprint or face | Strongest and the best experience; needs a reasonably modern phone and good platform support |
| Push approval | A prompt in an app you already run | Excellent and requires an app you do not have |

**Social login solves authentication and zero percent of authorization.** Chapter 12's attacks must all still pass.

**The client secret stays server-side; redirect URIs must be exact.** In the browser the secret is public, and a wildcard redirect sends the code to the attacker.

**Decide account linking before you ship.** Match on *verified* email and link to the existing user; linking an unverified address is account takeover.

**Match protection to power, and keep a fallback.** Staff need a second factor, students do not; recovery must never be weaker than the login, and someone without a smartphone still needs a route in.

### Verify it

Six checks; the first three are chapter 12's attacks, because social login does not remove them.

1. **Sign in with Google as an ordinary student, then type the admin URL.** The page must refuse; then call the admin endpoint directly with that session — `403`.
2. **Change an id in a URL to another user's record** while signed in via Google: `403` or `404`.
3. **Call a protected endpoint with no session.** `401`, every time.
4. **Sign in with Google, log out, sign in by email OTP with the same address.** Two rows in your users table is the collision bug.
5. **Search your frontend bundle for your client secret.** Finding it means rotating the secret today.
6. **Read the role assignment code.** The role must come from your database, a new user must not be creatable as admin, and `email_verified` must be checked.

Red flags: a token exchange in the browser; a redirect URI set to a wildcard or `localhost` in production; a role read from the provider's response; no `email_verified` check; 2FA for students but not admins; SMS OTP as the only factor, no fallback; recovery codes regenerated without re-authenticating; "Google handles security for us".

## Walkthrough
1. List the login methods your product will offer, and who uses each. Name at least one route in for someone who cannot use social login.
2. Decide your account-linking rule in writing, as a team: what happens when the same verified email arrives by a second method. Decide also what happens for an unverified one.
3. Decide which roles require a second factor. Write the rule as a sentence; the expected answer is that staff and admin do and ordinary users do not.
4. Set up one social provider using your platform's authentication service rather than hand-rolled code. Keep the client secret in an environment variable from the first minute.
5. Sign in as yourself and watch the redirects in the browser's Network tab. Find the redirect to the provider, the redirect back, and confirm the code exchange does not appear as a browser request — because it should be happening on your server.
6. Open your users table and look at the row that was created. Note which fields came from the provider, and find where your code assigned the role.
7. Read that role assignment line out loud. Confirm it comes from your database or a hard-coded default, never from the token.
8. Run chapter 12's attacks again with your new social session: admin page by URL, admin endpoint directly, another user's id, and no session at all. Record four status codes.
9. Test the collision: sign in with the social provider, log out, then sign in with a second method using the same email. Count the user rows and record the result.
10. Search your frontend bundle for the client secret string. Record that it is absent, or rotate it today.
11. Enable a second factor on one staff account, test the recovery path as if the phone were lost, and record your decisions: login methods, linking rule, which roles need 2FA, whether recovery is weaker than the login, and what a staff member does when they lose their device.

## Talk to AI
```prompt
CONTEXT:      Three-person student team, free tiers only. Users are Indian
              college students on budget Android phones and slow 4G; some
              share devices and some have no reliable SMS in hostels.
              Product: [one sentence]
              Roles: [e.g. student, canteen staff, admin] and what each can
              do: [one line each — be specific about the most dangerous
              power each role has]
              Our auth today: [platform auth service / custom / none]
              Our hosting: [platform]
              We handle this personal data: [name, email, phone, etc.]

GOAL:         A login design using an existing identity provider, with the
              right second factor for the right roles, and an explicit rule
              for what happens when one person arrives by two doors.

REQUIREMENTS: 1. Describe the OAuth flow for my setup step by step, naming
                 which steps happen in the browser and which must happen on
                 my server, and where the client secret lives
              2. State explicitly what the identity provider does and does
                 NOT tell me, and where in my code I assign the role
              3. Give me an account-linking rule covering: same verified
                 email by a second method, an unverified email, and two
                 different emails that are the same human
              4. Recommend a second factor per role, judged against my
                 users' devices and network — not against what is
                 theoretically strongest — and give me a fallback for each
              5. Design the recovery path for a staff member who loses
                 their phone, and tell me whether your recovery is weaker
                 than the login it protects
              6. Give me a manual attack plan I run myself after
                 implementing this: the exact requests to send as an
                 ordinary logged-in user, as a logged-out visitor, and as a
                 user asking for someone else's row, with expected status
                 codes
              7. List what personal data I will now hold because of this
                 login, and what the DPDP Act expects me to tell users

CONSTRAINTS:  Free tier only. Use my platform's authentication service; do
              not hand-roll OAuth. Client secret in an environment
              variable, never in frontend code. Must work for a student
              with no reliable SMS. Roles come from my database only.

NON-GOALS:    Do not build row-level security policies — that is the next
              chapter. Do not add more than two login methods. Do not
              design the admin UI. Do not write password handling.

ACCEPTANCE:   - As an ordinary user signed in via the provider, every admin
                endpoint returns 403 and the admin page refuses
              - Signing in by two methods with the same verified email
                produces exactly one user row
              - The client secret does not appear anywhere in the frontend
                bundle
              - A brand-new user cannot be created with an admin role
              - Every role with dangerous powers has a second factor and a
                recovery path that is not weaker than the login

ASK:          Before any implementation, tell me exactly what adding social
              login does NOT protect in my app, and give me the attack plan
              so I can see my current failures first.
```

## Mission

**Objective.** A working login for your own startup through an external identity provider, with roles assigned by your own database, a second factor on every privileged role, and a written record of your own attacks against it passing.

**Steps.**
1. **Frame** — list your roles and, for each, the single most dangerous thing it can do. That list is what decides who needs a second factor.
2. **Decide, as a team** — your login methods including one route in for someone who cannot use social login; your account-linking rule for verified and unverified emails; and which roles require 2FA. Write all three down before building.
3. **Baseline attack** — before you change anything, run chapter 12's four attacks against your current app and record the status codes. You need the before picture.
4. **Brief** — give AI your roles, your users' device and network reality, and your decisions. Demand the attack plan before any implementation.
5. **Direct** — implement one social provider using your platform's authentication service, with the client secret in an environment variable from the first minute.
6. **Assign roles yourself** — make the role come from your database with a safe default. Prove a brand-new user cannot be created as an admin.
7. **Verify by attacking** — as an ordinary signed-in user: admin page by URL, admin endpoint directly, another user's row by id, and every protected endpoint logged out. Four results, recorded with screenshots.
8. **Test the collision** — sign in by both methods with the same email and count the user rows. Fix it if there are two, then re-test.
9. **Check the secret** — search your frontend bundle for the client secret string and record the result.
10. **Second factor** — enable one on a privileged account, then test the recovery path as if the phone were lost, and judge honestly whether recovery is weaker than the login.
11. **Document** — your decisions, your before-and-after attack results, and the personal data you now hold because of this login.

**Upload.**
- Your roles table with the most dangerous power of each → post caption or attached file
- Your three written decisions: login methods, linking rule, 2FA by role
- The AI conversation including the attack plan → share link in `ai_chat_url`
- Working login → deployed URL in `demo_url`, or a screen recording of the full flow
- Attack evidence → screenshots of all four attacks with their status codes, before and after
- The user-row count from your collision test, before and after any fix
- Evidence the client secret is absent from the frontend bundle → screenshot of the search
- Your 2FA and recovery notes, plus your list of personal data held
- Reflection, 150 words max → what social login did not protect, and one thing the AI got wrong

**Self-check.** Tick every box before you submit.
- [ ] I ran the four attacks BEFORE implementing and recorded the failures
- [ ] As an ordinary signed-in user, the admin page refuses me
- [ ] As an ordinary signed-in user, calling the admin endpoint directly returns 403
- [ ] Requesting another user's row by id returns 403 or 404, never their data
- [ ] Every protected endpoint returns 401 with no session
- [ ] The role is assigned by my own database code, not read from the provider's token
- [ ] A brand-new user cannot be created with a privileged role, and I tested this
- [ ] `email_verified` is checked before I trust an email address
- [ ] Signing in two ways with the same verified email produces exactly one user row
- [ ] The client secret is in an environment variable and absent from the frontend bundle
- [ ] Every privileged role has a second factor, with a fallback that works without reliable SMS
- [ ] I tested the recovery path and wrote down whether it is weaker than the login
- [ ] I have a login route for a student who cannot use social login

**Stretch.** Have a teammate who did not build the login try to break it for fifteen minutes with only an ordinary account, dev tools, and the address bar. Write down everything they reached that they should not have.

## Stuck? Ask AI
```prompt
Walk me through the OAuth flow one step at a time, as a conversation between a student, my app, my server and Google. After each step, tell me what each party now knows and what they still do not know. Then tell me which single step must never happen in the browser, and what an attacker could do if it did.
```

```prompt
I just added Sign in with Google. List everything in my app this does NOT protect, and give me the exact sequence of actions an ordinary logged-in student would take from their own phone to reach data they should not see.
```

```prompt
I need to pick a second factor for staff accounts. My users are Indian college students on budget Android phones, sometimes in hostels with poor signal. Compare SMS OTP, an authenticator app and passkeys for exactly these people, then ask me three questions about my users before you make a recommendation.
```

## Quiz

### Q1 · L3
After adding "Sign in with Google", what has been solved and what has not?
- (a) Both authentication and authorization are now handled by Google
- (b) Authentication is solved; authorization is still entirely yours ✅
- (c) Authorization is solved because Google returns verified roles
- (d) Neither, until you also add two-factor authentication
> Google proves who someone is. It has no opinion on whether they may open your admin page, so chapter 12's role and ownership checks remain exactly as necessary as before.

### Q2 · L3
In the OAuth flow, which step must happen on your server rather than in the browser?
- (a) Redirecting the user to the provider's login page
- (b) Exchanging the authorization code for a token using your client secret ✅
- (c) Receiving the redirect back from the provider
- (d) Showing the consent screen
> The client secret must never reach the browser. A code exchange done in frontend code publishes the secret and lets anyone impersonate your application.

### Q3 · L4
A student signs in with Google using `priya@college.edu`, then later with email OTP using the same address, and ends up with two accounts. What was missing?
- (a) Email verification on the second method
- (b) A written account-linking rule applied at sign-in ✅
- (c) A unique constraint on the users table
- (d) A second factor on the student role
> Nobody decided what happens when one human arrives by two doors. The usual rule is to match on verified email and link the method to the existing user, and it must be decided before launch because merging afterwards is painful.

### Q4 · L4
Your users are students on budget Android phones, some in hostels with poor signal. Which second-factor plan fits best?
- (a) Mandatory SMS OTP for every user, with no alternative
- (b) Passkeys only, since they are the strongest option
- (c) Social login or OTP for students, a real second factor plus a non-SMS fallback for staff and admin ✅
- (d) Two passwords, one of which rotates monthly
> Protection should match the power of the account. Students ordering samosas do not need a second factor; the account that can read 400 phone numbers does — and it needs one that works without reliable SMS.

### Q5 · L5
Your signup code sets a new user's role from the `role` field in the identity provider's token. Why is this wrong?
- (a) Providers do not include a role field, so it will always be null
- (b) Roles are permissions, which are yours to decide — an external provider must never be able to grant powers in your app ✅
- (c) It will break if the provider changes its token format
- (d) It prevents account linking from working
> Identity comes from outside; permission is always yours. A provider-supplied role means a party outside your system can influence what someone may do in it, and any claim an attacker can shape becomes an admin promotion.

## Go deeper
- [OAuth 2.0 Simplified — the flows explained plainly](https://www.oauth.com/)
- [OWASP Multifactor Authentication Cheat Sheet](https://cheatsheetseries.owasp.org/cheatsheets/Multifactor_Authentication_Cheat_Sheet.html)
