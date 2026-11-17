---
id: m02-14-https-cookies-cors
module: m02-the-web
slug: https-tls-cookies-cors
title: "HTTPS, TLS and Certificates; Cookies, Sessions and CORS"
order: 14
duration_min: 15
is_free: false
mission: false
volatility: low
last_reviewed: 2026-11-16
---

## Key learnings
- Say exactly what HTTPS protects and what it says nothing about
- Explain how a stateless protocol remembers a logged-in student
- Name the three cookie flags that decide whether a session can be stolen
- Reject the wildcard "fix" for a CORS error and say what it actually does
- Explain why CORS is a browser rule and never a server security control

## Lesson

### The pain

A team demos their canteen app on college Wi-Fi, login page on plain `http://` because HTTPS "was not set up yet". A mentor on the same network opens a free traffic tool and reads a student's password out of the air in front of the room — not decrypted, just read.

Two more surface that afternoon: a session cookie with no `HttpOnly` flag, readable by any script that reaches the page, and a CORS error "fixed" with `Access-Control-Allow-Origin: *` plus credentials. Three failures in the small print, none visible in a screenshot.

### The idea

**HTTPS makes the pipe unreadable and proves who is at the other end. Cookies are how a protocol with no memory remembers you. CORS is the browser's rule about which other websites may read your responses.**

Three mechanisms that beginners merge into one vague feeling of "security". Each protects against something specific and is silent about everything else.

### The analogy

**HTTPS is the sealed tiffin carrier**: nobody on the path can see inside or swap the contents without breaking a seal you would notice, and the stamp proves which canteen packed it. **A cookie is the ink stamp on your hand** — the counter cannot remember two thousand students, and whoever copies the stamp eats your lunch. **CORS is the rule that the hostel mess cannot collect food using your stamp.**

**Where the analogy breaks:** the seal protects the tiffin *on the journey*; once opened, the canteen can do anything with the contents. HTTPS says nothing about whether the server is trustworthy, and phishing sites hold valid certificates. The padlock means "this connection is private and goes to the name in the address bar", never "this site is safe".

### Picture it

```
  OVER http://  on shared college Wi-Fi
  phone ──"password=hunter2"──▶ [anyone listening reads it] ──▶ server

  OVER https://
  phone ──[░░░ unreadable ░░░]──▶ [listener sees only noise] ──▶ server
            │                                                      │
            └── certificate proves this really is canteen-nashik.in
```

Over plain HTTP anyone sharing the network can read the traffic as text; over HTTPS they see only noise, and the certificate proves the server really is the name you typed.

HTTP has no memory, so the server gave you a **cookie** on login; the browser attaches it to every later request and the server looks it up to find your **session**. That value *is* your identity — whoever holds it is you.

| Flag | What it does | What happens without it |
|---|---|---|
| `Secure` | Only sent over HTTPS | Sent in the clear on `http://`, readable on shared Wi-Fi |
| `HttpOnly` | Not readable by page scripts | Any injected script can read and exfiltrate it |
| `SameSite` | Not sent on requests from other sites | Another site can trigger authenticated actions as you |

All three, always, on any cookie representing a logged-in user.

### Where it lives

This chapter wraps M00's network box and reaches into client and server both. HTTPS is terminated at your hosting platform, which issues and renews the certificate once DNS is correct — hence chapter 12 first. Cookies are set by the server and stored by the browser; CORS is enforced entirely inside the browser. M04 builds the real session logic.

### Words to own

| Term | Plain meaning | Example |
|---|---|---|
| HTTPS | HTTP inside an encrypted connection | `https://` and a padlock |
| TLS | The encryption that does the work | Negotiated per connection |
| Certificate | A signed document binding a key to a name | Issued for your domain |
| Certificate authority | Who signed it, and whom browsers trust | A free issuer |
| Mixed content | HTTPS page loading some resources over HTTP | Browser blocks or warns |
| Cookie | A small value the browser returns on every request | The session cookie |
| Session | The server-side record a cookie points at | "This is student 412" |
| `Secure` | Cookie sent only over HTTPS | Always, on auth cookies |
| `HttpOnly` | Cookie unreadable by page scripts | Always, on auth cookies |
| `SameSite` | Cookie withheld on cross-site requests | `Lax` or `Strict` |
| Origin | Scheme plus host plus port, taken as one unit | `https://canteen.in:443` |
| Same-origin policy | Pages may not read other origins' responses | The browser's default |
| CORS | The server's opt-in to let a named origin read it | A response header |
| Preflight | A probing OPTIONS request before the real one | For non-simple requests |

### Tradeoffs and gotchas

**CORS is not a security control.** The browser's *same-origin policy* stops a page reading another origin's response — necessary because cookies travel automatically, so otherwise any site you visited could fetch your bank page. CORS is the server's opt-in, a header naming an origin that may read it, and it restricts browsers only: a command-line tool anywhere can call your API and read every byte. Only server-side authorisation protects it.

**The CORS error appears in your browser and the fix is on the server.** Non-trivial requests send a "preflight" OPTIONS first, so the failure can appear as a request your code never made. And `*` with credentials is forbidden by the spec, so an AI that "fixes" CORS that way either still fails or has silently disabled credentials.

**The padlock is not a safety rating,** and an expired certificate takes the site down behind a warning nobody clicks past. Automatic renewal is the only acceptable arrangement.

### Verify it

1. **Check the scheme on every page that takes input.** A login page on `http://` is a complete failure.
2. **Read the cookie flags** in Application → Cookies. Missing `HttpOnly` turns script injection into a stolen account.
3. **Click the padlock.** Certificate name matches; expiry in the team calendar.
4. **Look for mixed-content warnings** in the Console: a blocked resource means a hardcoded `http://`.
5. **Read the whole CORS error.** It names the origin and the missing header — that text is the fix.
6. **Test your API from outside a browser.** Data returned with no authorisation means it is open to the world.
7. **Try the wrong user.** A staff endpoint returning 200 to a student means server-side authorisation is missing.

Red flags: `Access-Control-Allow-Origin: *` · disabling CORS or proxying around it in production · cookies with no flags · tokens in `localStorage` with no discussion of the tradeoff · `http://` in your own URLs · an endpoint called safe because the UI does not link to it · a self-signed certificate for users.

And one in reasoning: if an AI treats a CORS error as a security feature protecting your API, it has the central misconception, and every authorisation decision it has made on that basis needs re-checking.

## Walkthrough
1. Open a site that has a login, with DevTools on the Network panel. Note the scheme in the address bar. Click the padlock, open the certificate details, and write down the issuer, the name it was issued for, and the expiry date.
2. Log in — use any account you own, or a throwaway one. Find the login request in the Network panel and look at the response headers for `Set-Cookie`.
3. Switch to the Application panel and open Cookies. Find the cookie that looks like a session. Record whether `Secure`, `HttpOnly` and `SameSite` are set, and what `SameSite` is set to.
4. Now navigate to any other page on the same site and look at a request's headers. Confirm that the cookie is being attached automatically, without your code doing anything. This is the mechanism that makes a stateless protocol remember you.
5. In the Application panel, delete that cookie and reload. Note what happens. You have just demonstrated that the cookie *is* your identity.
6. Open the Console on a few different pages and look for mixed-content or blocked-resource warnings. Note any resource being requested over `http://` from an `https://` page.
7. Find any request on the site that goes to a different domain than the page itself — an analytics endpoint, an image host, an API subdomain. Look at its response headers for `Access-Control-Allow-Origin` and note the value.
8. Ask an AI to explain that specific value to you: whether it names an origin or uses a wildcard, and what that permits.
9. Log out, then try to open a URL that only a logged-in user should see, by pasting it directly into the address bar. Record the status code and whether you got any data. Compare that against what the logged-out UI shows you.
10. Write up five findings: the certificate expiry, the three cookie flags, any mixed content, what the cross-origin headers permit, and whether a logged-out request to a protected URL is actually refused by the server rather than merely hidden by the UI.

## Talk to AI
```prompt
CONTEXT:      I direct AI rather than configuring servers. My product is a
              canteen ordering app for students in Nashik. Students log in
              and place orders; canteen staff use a separate dashboard on a
              different subdomain that calls the same API. Students use
              budget Android phones, very often on shared college Wi-Fi.
              UPI payments will be added later, which means a payment
              provider will redirect back to our site. We are on free-tier
              hosting with automatic certificates. I have been told a CORS
              error is a security feature and I am not sure that is right.

GOAL:         A written security baseline for transport, sessions and
              cross-origin access that I can check in DevTools in ten
              minutes, plus a correct explanation of what each mechanism
              does and does not protect.

REQUIREMENTS: 1. A plain statement of what HTTPS protects and what it says
                 nothing about, with one example of each
              2. The exact cookie configuration for the student session —
                 every flag, its value, and what specifically goes wrong
                 without it on shared Wi-Fi
              3. A recommendation between an HttpOnly cookie and a token in
                 localStorage, with the risk each one accepts, and a direct
                 pick for my situation
              4. The CORS configuration for the staff dashboard subdomain
                 calling the API subdomain: which origins, which methods,
                 whether credentials are allowed, and what a preflight is
              5. A correction or confirmation of my belief that CORS
                 protects my API, with a concrete demonstration of what
                 someone outside a browser can still do
              6. What must change about SameSite when the UPI redirect flow
                 is added, and how to avoid loosening it permanently
              7. A ten-minute DevTools checklist a teammate could run

CONSTRAINTS:  Free-tier hosting with managed certificates. No self-signed
              certificates anywhere. No wildcard Access-Control-Allow-
              Origin. No disabling of browser security features, no
              browser launch flags, and no development proxy presented as a
              production fix. Assume I cannot read code and must verify
              everything through DevTools or a single command you give me.

NON-GOALS:    Do not design the login flow or the password storage. Do not
              implement the payment integration. Do not cover XSS or SQL
              injection in depth. Do not recommend any paid security
              product.

ACCEPTANCE:   - I can explain in two sentences what the padlock does and
                does not tell a user
              - The session cookie configuration names all three flags with
                values and reasons
              - The CORS configuration names explicit origins and never
                uses a wildcard with credentials
              - Your answer states plainly whether CORS protects my API
                from a non-browser caller, and proves it
              - The checklist is runnable by a teammate in ten minutes
                without reading any code
              - Nothing in your answer treats hiding a URL in the UI as a
                protection

ASK:          Before answering, ask me up to five questions whose answers
              would change your recommendations. Then answer in the order
              of my requirements. Then, separately, list the three mistakes
              teams in exactly my position make most often, and for each
              one, how it would appear in DevTools so I could catch it.
```

## Stuck? Ask AI
```prompt
Explain what HTTPS actually protects, using an analogy that is not about envelopes, seals or tunnels. Then give me six statements about a site with a padlock — "my data cannot be read in transit", "this site is trustworthy", and four more — and ask me which are true and which are false, and why. Do not reveal the answers until I have attempted all six.
```

```prompt
I have a CORS error in my console: [paste the full error text]. Explain, in order: what the browser was trying to do, which rule stopped it, why that rule exists, where the fix has to be made, and what the fix should literally say. Then tell me the two "fixes" I might be tempted by that would make my app less safe, and exactly what each one would expose.
```

```prompt
Explain why CORS is not a security control for my API. Show me, step by step, what someone with a command-line tool can do to an API that has strict CORS configured, and what that means I must do instead. Then explain the one thing CORS genuinely does protect, and who it protects.
```

## Quiz

### Q1 · L3
A page shows a padlock and a valid certificate. What does that guarantee?
- (a) The website is trustworthy and will handle your data responsibly
- (b) The connection is encrypted and goes to the name in the address bar — nothing more ✅
- (c) The site has been security-audited by the certificate authority
- (d) No one can read your data after it reaches the server
> Phishing sites hold valid certificates routinely. HTTPS protects the journey and proves the name; it says nothing about what the server does with what arrives.

### Q2 · L3
Your session cookie is missing `HttpOnly`. What does that specifically enable?
- (a) The cookie can be sent over plain HTTP
- (b) Any script that runs on the page can read it and send it elsewhere ✅
- (c) The cookie will be attached to requests from other websites
- (d) The cookie never expires
> `HttpOnly` is what stops page scripts from reading the cookie. Without it, any injected script — including one arriving through an unescaped dish name — turns into a stolen session. `Secure` covers the HTTP case and `SameSite` the cross-site one.

### Q3 · L4
An AI resolves your CORS error by setting `Access-Control-Allow-Origin: *` alongside credentials. What is wrong?
- (a) Nothing; the wildcard is the standard fix
- (b) The combination is disallowed, so either it still fails or credentials were silently dropped — and naming the origins is the real fix ✅
- (c) Wildcards are slower for the browser to process
- (d) It only works on subdomains
> The specification refuses wildcard-plus-credentials precisely because it would let any site on the internet make authenticated requests. The honest fix names the staff dashboard's origin explicitly.

### Q4 · L5
Why is CORS not a security control for your API?
- (a) Because it can be disabled in the browser's settings
- (b) Because only browsers enforce it — a command-line tool or another server reads every byte regardless ✅
- (c) Because it only applies to GET requests
- (d) Because certificates override it
> CORS protects your users' browsers from other websites. Nothing stops a non-browser caller, which is why server-side authorisation is the only real protection — and why treating a CORS error as proof of safety is dangerous.

### Q5 · L4
A login form is served over `http://` on college Wi-Fi. What is the concrete consequence?
- (a) The browser will refuse to submit the form
- (b) Anyone sharing that network can read the submitted password as plain text ✅
- (c) The password is weakly encrypted and could be cracked later
- (d) Only the server's own logs are at risk
> There is no encryption to break — the credentials travel as readable text through equipment anyone on the network can listen to. This is why `Secure` cookies and HTTPS-only pages are not optional.

## Go deeper
- [MDN: Cross-Origin Resource Sharing (CORS)](https://developer.mozilla.org/en-US/docs/Web/HTTP/Guides/CORS)
- [MDN: Using HTTP cookies and their security flags](https://developer.mozilla.org/en-US/docs/Web/HTTP/Guides/Cookies)
