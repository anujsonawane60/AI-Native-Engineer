---
id: m04-13-oauth-sso-passkeys-2fa
module: m04-backend-data-auth-payments
slug: oauth-sso-passkeys-2fa
title: "OAuth, SSO, Passkeys and 2FA"
order: 13
duration_min: 30
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

The canteen team adds "Sign in with Google" in forty minutes and it feels like the best forty minutes of the project. No passwords, no reset emails, no forgotten-password support. Students tap once and they are in.

Three problems arrive over the next two weeks.

The first is chapter 12's, and it is the reason this chapter follows that one: the team believes they have added security. They have added *identity*. Google told them who the student is. Google said nothing about whether that student may open the admin page, and the admin page is still open.

The second is identity collision. A student signs in with Google on Monday using `priya@college.edu`. On Friday, on a friend's phone, she uses the email-and-OTP option with the same address. The app creates a second account. Her orders are split across two accounts and she is certain the app lost her data. Nobody decided what should happen when the same human arrives by two doors.

The third shows up at the demo. A judge asks how staff accounts are protected, because a staff account can mark any dish sold out and read every student's phone number. The honest answer is "a Google password that might be `priya2003`". Nothing protects it beyond that.

### The idea

**OAuth lets a user prove their identity to you using an account they already have somewhere else, without ever giving you their password — and that is all it does.**

The flow is a redirect dance. Your app sends the student to Google. Google authenticates them and asks whether they consent to sharing their name and email with you. Google redirects back to your app with a short-lived **code**. Your *server* exchanges that code, plus your client secret, for a token, and reads the identity from it. At no point do you see the password.

**SSO** (single sign-on) is the same machinery used across an organisation so one login works for many applications. **2FA** adds a second, different kind of proof — something you have, not just something you know. **Passkeys** replace the password entirely with a key held in your device's secure hardware, unlocked by your fingerprint or face.

### The analogy

A student wants a canteen tab without the canteen holding her documents.

She walks to the college registrar's office with the canteen's request slip. The registrar checks her thoroughly — the canteen never sees how — asks "may I confirm your name and roll number to the canteen?", and on her yes hands her a sealed one-time chit.

She brings the chit to the canteen. The canteen manager does not open it in front of her; he sends it to the registrar's office himself, with his own stamped letterhead, and the registrar confirms: this is Priya Sharma, roll 4471. The canteen opens a tab in her name and has never seen a document.

Notice what the chit does not say: nothing about whether Priya may open the cash box. The registrar does not know and does not care. Permissions are the canteen's business.

**Where the analogy breaks:** the chit can be stolen between offices. That is why the exchange happens server-to-server with the canteen's own letterhead — a secret the student's hands never touch — and why the chit expires in seconds. The physical version has no equivalent of a stolen redirect, which is the thing the protocol is mostly designed to prevent.

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

OAuth sits at the authentication gate from chapter 12 and replaces only that gate. Every interior door — the role check, the ownership check — is still yours, unchanged, and still the thing that actually protects data.

The box in the middle of that diagram is the whole lesson. When the token comes back, your server does three things Google has no opinion about: finds or creates a user row, assigns a role, and starts your own session. **The role is assigned by your code, from your database, never from anything in the token.** An identity provider that could grant admin rights in your app would be a strange and dangerous thing.

Forward: chapter 14 builds the roles and row-level policies this hands off to. Chapter 15's signed URLs lean on the session this creates. In M07 these flows meet OWASP, secret handling and the DPDP Act — because the moment you receive a student's name, email and phone from Google, you are processing personal data and you owe them a reason, a retention period and a way to delete it.

### Second factors, and which ones work in India

2FA means two different *kinds* of proof. Something you know (a password), something you have (a phone, a key), something you are (a fingerprint). Two passwords is not 2FA.

| Factor | How it works | Honest verdict for your users |
|---|---|---|
| SMS OTP | A code texted to the phone | Familiar everywhere in India, works on any handset, and the weakest option — SIM swap and delivery delays on bad networks |
| Authenticator app | A 6-digit code from a shared secret, rotating every 30 seconds | Strong, free, works offline, needs the user to install an app and keep a backup |
| Email OTP | A code sent to the inbox | Only as strong as the email account; fine as a fallback |
| Passkey | A key in the device's secure hardware, unlocked by fingerprint or face | Strongest and the best experience; needs a reasonably modern phone and good platform support |
| Push approval | A prompt in an app you already run | Excellent and requires an app you do not have |

For a student product the sensible split is: ordinary users get social login or OTP and no mandatory second factor, and **anyone with staff or admin powers gets a real second factor.** The asymmetry is deliberate — the account that can read 400 phone numbers deserves more protection than the one that can order a samosa.

Note the Indian texture. SMS OTP is culturally the default: UPI, banking, IRCTC and Aadhaar have trained everyone to expect it. It also costs money per message, arrives slowly on a weak network, and fails entirely in a hostel basement. Passkeys are genuinely better and some budget Android devices still handle them poorly. Offer a fallback, always, and never make the fallback weaker than the primary — that is where attackers go.

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

**Social login solves authentication and zero percent of authorization.** If you take one thing from this chapter: adding Google login does not protect a single endpoint. Chapter 12's six attacks must still all pass.

**Decide account linking before you ship.** The rule most teams want: match on verified email and link the new method to the existing user rather than creating a second account. Write it down, because the alternative is a student with two half-filled accounts and no way to merge them. Be careful with *unverified* email — linking on an address someone does not control is an account takeover.

**The client secret belongs on the server, always.** In frontend code it is public, and anyone can impersonate your application. This is M07's secrets chapter arriving early.

**Redirect URIs must be exact and allow-listed.** A loose or wildcard redirect lets an attacker have the code delivered to their own site — the most-exploited OAuth misconfiguration, prevented in the provider's console.

**An email from a provider is not automatically verified.** Check the `email_verified` claim.

**Social login alone excludes people.** A student without a smartphone, on a shared device, or with a college account that blocks third-party apps needs another route in.

**2FA needs recovery, and recovery is the attack surface.** Single-use codes shown once, and a human process for staff who lose a phone. A reset weaker than the login defeats the 2FA.

**Do not implement OAuth by hand.** The failure modes — state parameter, PKCE, nonce checks, token validation — are subtle, and hand-rolled implementations are reliably wrong in ways that look fine.

**Provider UIs change.** Button names, console layouts and free-tier limits in this area churn every few months. Follow current provider documentation rather than any screenshot, including any you find in an AI answer.

### Verify it

Six checks. The first three are chapter 12's attacks, repeated because social login does not remove them.

1. **Sign in with Google as an ordinary student, then type the admin URL.** The page must refuse. Then call the admin endpoint directly with that session. It must return `403`.
2. **Change an id in a URL to another user's record** while signed in via Google. You must get `403` or `404`.
3. **Call a protected endpoint with no session.** `401`, every time.
4. **Sign in with Google, log out, then sign in with the email OTP method using the same address.** Count the rows in your users table. Two rows is the collision bug, and it is much cheaper to find today than after the demo.
5. **Search your frontend bundle for your client secret.** Open dev tools, search the loaded JavaScript for the string. Finding it means rotating the secret today.
6. **Read the role assignment code.** Confirm the role comes from your own database, not from anything in the provider's token, and that a brand-new user cannot be created as an admin. Then confirm `email_verified` is actually checked.

Red flags: a flow where the token exchange happens in the browser; a redirect URI set to a wildcard or to `localhost` in production; a role read from the provider's response; no `email_verified` check; 2FA offered to students but not to admins; SMS OTP as the only second factor with no fallback; recovery codes that can be regenerated without re-authenticating; and any claim that "Google handles security for us".

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
