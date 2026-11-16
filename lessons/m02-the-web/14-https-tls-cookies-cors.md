---
id: m02-14-https-cookies-cors
module: m02-the-web
slug: https-tls-cookies-cors
title: "HTTPS, TLS and Certificates; Cookies, Sessions and CORS"
order: 14
duration_min: 25
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

A team demos their canteen app on the college Wi-Fi. The login page is served over plain `http://`, because HTTPS "was not set up yet" and it worked fine locally.

A mentor on the same network opens a free traffic-inspection tool and reads a student's password out of the air in front of the room. Not decrypted — just read, because it was never encrypted. Shared Wi-Fi means everybody's traffic passes through equipment that anybody on that network can listen to.

Two more problems surface the same afternoon. The session cookie has no `HttpOnly` flag, so any script that ends up on the page — including one injected through an unescaped dish name — can read it and send it elsewhere. And the staff dashboard, hosted on a different subdomain, had thrown a CORS error during development; the agent had "fixed" it by setting `Access-Control-Allow-Origin: *` together with credentials, which means any website on the internet could, in principle, make authenticated requests on a logged-in student's behalf.

Three failures, all of them in the small print, none of them visible in any screenshot.

### The idea

**HTTPS makes the pipe unreadable and proves who is at the other end. Cookies are how a protocol with no memory remembers you. CORS is the browser's rule about which other websites may read your responses.**

Three separate mechanisms that beginners routinely merge into one vague feeling of "security". Keeping them distinct is the whole chapter, because each one protects against something specific and is silent about everything else.

### The analogy

Three things at the canteen.

**HTTPS is the sealed tiffin carrier.** The canteen packs your order, seals it with their own stamp, and sends it across campus. Nobody on the path can see what is inside, and nobody can swap the contents without breaking a seal you would notice. The stamp also proves it came from that canteen and not from someone who set up a table outside pretending to be them.

**A cookie is the ink stamp on the back of your hand.** The counter cannot possibly remember two thousand students, so they stamp your hand when you pay. You show the stamp; you get served. The stamp *is* your proof of payment, which means anyone who can copy it can eat your lunch.

**CORS is the rule that the hostel mess cannot send its own staff to collect food using your stamp.** Your stamp works at this canteen, with you presenting it. It does not authorise a different establishment to use it on your behalf.

**Where the analogy breaks:** the seal protects the tiffin *on the journey*. Once it is opened at the counter, the canteen can do anything at all with the contents — cook them badly, write down what you ordered, sell that list. HTTPS is exactly the same: it says nothing whatsoever about whether the server at the other end is trustworthy. A phishing site can hold a perfectly valid certificate, and most do. The padlock means "this connection is private and goes to the name in the address bar", never "this website is safe".

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

### How a stateless protocol remembers you

Chapter 13 ended on a hard fact: HTTP has no memory. Every request arrives as a complete stranger. So how does the server know, on request number forty, that you logged in on request number three?

You send the proof back every time. The server gives you a small value on login — a **cookie** — and the browser attaches it automatically to every subsequent request to that site. The server looks it up, finds your **session**, and knows who you are.

Which means that value is your identity. Anyone who obtains it is you, until it expires. That is the entire reason the three flags below exist, and why "it's just a cookie" is the wrong instinct.

| Flag | What it does | What happens without it |
|---|---|---|
| `Secure` | Only sent over HTTPS | Sent in the clear on `http://`, readable on shared Wi-Fi |
| `HttpOnly` | Not readable by page scripts | Any injected script can read and exfiltrate it |
| `SameSite` | Not sent on requests from other sites | Another site can trigger authenticated actions as you |

All three, always, on any cookie that represents a logged-in user. There is no interesting debate here.

### Where it lives

This chapter wraps the network box from M00 and reaches into both the client and the server. HTTPS is terminated at your hosting platform, which on every free tier worth using issues and renews the certificate for you once DNS is correct — which is why chapter 12 comes first. Cookies are set by the server and stored by the browser. CORS is enforced entirely inside the browser.

M04 builds the real session and authorisation logic; M07 is where the security thinking becomes systematic. The job of this chapter is to make sure you can read these three mechanisms in the Network and Application panels and recognise when AI output has quietly weakened one of them.

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

### CORS, explained once, properly

A page loaded from one origin is not allowed by the browser to *read* the response from another origin. That is the **same-origin policy**, and it is the default.

It exists for a narrow but vital reason: your browser attaches your cookies to requests automatically. Without the policy, any random site you visited could quietly request your bank's account page, receive it complete with your logged-in session, and read your balance. The policy is what stops that.

**CORS** is how a server says "it is fine, this particular origin may read my responses" — a response header naming the allowed origin. So when your staff dashboard at one subdomain calls your API at another and the browser blocks it, the correct fix is for the API to name that origin.

Three things follow, and the third is the one that matters most.

The error appears in *your* browser but the fix is on the *server*, which is why it feels so confusing. A "preflight" OPTIONS request is sent first for anything non-trivial, so a CORS problem can show up as a failed request your code never made. And — the important one — **CORS is not a security control.** It restricts browsers. A script, a command-line tool or a server anywhere in the world can call your API and read every byte, because nothing is enforcing CORS on their side. CORS protects your *users' browsers* from other websites. It does not protect your API from anyone. Authorisation on the server is the only thing that does.

### Tradeoffs and gotchas

**`Access-Control-Allow-Origin: *` with credentials is forbidden, and for good reason.** The specification refuses the combination, so an AI that "fixes" CORS this way either produces something that still does not work or has disabled credentials without telling you. Either way, naming the exact origins is the answer.

**The padlock is not a safety rating.** Phishing sites hold valid certificates routinely, so "look for the padlock" teaches users something false.

**An expired certificate takes your whole site down** with a full-page warning most users will not click past. Automatic renewal is the only acceptable arrangement, and it depends on DNS staying correct.

**Mixed content breaks pages silently.** An HTTPS page loading one image over HTTP gets that resource blocked, and the symptom is a missing image with a console warning nobody read.

**A token in `localStorage` is readable by any script on the page**, where an `HttpOnly` cookie is not readable at all. For a student project the cookie is the safer default, and M04 revisits the tradeoff.

**`SameSite` interacts with third-party flows.** A payment provider redirecting back to your site can lose a `Strict` cookie, so people loosen it and forget to tighten it again. Decide deliberately and write down why.

### Verify it

Everything here is checkable in the Network and Application panels in a few minutes.

1. **Check the scheme on every page that takes input.** Any login, payment or personal-data page on `http://` is a complete failure, and on shared college Wi-Fi it is demonstrably exploitable.
2. **Read the cookie flags.** DevTools, Application panel, Cookies. For your session cookie, confirm `Secure`, `HttpOnly` and a `SameSite` value are all present. Absent `HttpOnly` is the one that turns any script injection into a stolen account.
3. **Click the padlock and read the certificate.** Check the name matches and note the expiry date. Then put that date in your team's calendar even if renewal is automatic.
4. **Look for mixed-content warnings.** Console, on every page. One blocked resource is a visible bug and a signal that something still has a hardcoded `http://` address.
5. **Read the actual CORS error, all of it.** It names the origin, the header that was missing and whether a preflight failed. That text is the fix, and it is routinely skipped in favour of guessing.
6. **Test your API from outside a browser.** Ask the AI to show you the equivalent command-line request. If it returns data without any authorisation, your API is open to the world and CORS was never protecting it. This is the single most valuable check in the chapter.
7. **Try the wrong user.** Log in as a student and request a staff-only endpoint directly. A 200 means authorisation is missing on the server, regardless of what the UI hides.

Red flags in AI output: `Access-Control-Allow-Origin: *`; any suggestion to disable CORS, run the browser with security flags off, or use a proxy to "get around" it in production; cookies set without flags; tokens in `localStorage` with no discussion of the tradeoff; `http://` anywhere in a URL in your own code; a claim that an endpoint is safe because the UI does not link to it; and advice to use a self-signed certificate for anything users will open.

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
