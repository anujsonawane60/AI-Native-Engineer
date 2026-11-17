---
id: m02-12-ip-dns-ports
module: m02-the-web
slug: ip-dns-domains-ports
title: "IP Addresses, DNS, Domains and Ports"
order: 12
duration_min: 15
is_free: false
mission: false
volatility: low
last_reviewed: 2026-11-16
---

## Key learnings
- Trace a typed address to the machine that answers it
- Explain why a DNS change is not instant and cannot be forced to be
- Say what a port is and why `localhost:3000` is not on the internet
- Lower a record's TTL before you change it, and say why that order matters
- Name the five DNS and port red flags that cost teams a whole day

## Lesson

### The pain

A team buys `canteen-nashik.in` the night before their demo, points it at their deployment, gets nothing, and spends four hours convinced the deploy is broken.

The deploy was fine. Their record pointed at an address copied from an old screenshot. The old record's TTL was 24 hours, so even after the fix their laptops served the cached wrong answer — while one teammate's phone, on a different resolver, started working, which convinced everybody the site was "sometimes up". Meanwhile the app ran perfectly at `localhost:3000` on one laptop, which the team kept offering as proof. It was not on the internet at all.

### The idea

**Machines on the internet are found by number, people remember names, and DNS is the lookup that converts the second into the first. A port then says which program on that machine should answer.**

Three layers, three failure modes: the name resolves to the wrong number, the answer is cached somewhere you cannot reach, or nothing is listening on the port. Almost every "the site is down" that is not a code problem is one of those.

### The analogy

The **IP address** is the physical location: Building C, ground floor, west wing — precise and useless in conversation. The **domain name** is "the canteen". The **directory board** at the gate saying "Canteen → Building C" is **DNS**: a lookup table somebody maintains, which can be wrong or stale. The **ports** are the windows — coupons at window 1, food at window 2, staff entrance at window 3, kept shut.

**Where the analogy breaks:** repaint the board and it is repainted. DNS is thousands of copies — your phone, browser, college network, operator's resolver and layers above — each holding its own for as long as the TTL allowed. You cannot force them to expire. That is why "it works on my phone but not my laptop" is a normal mid-change state, and why "propagation" is really just caching running down.

### Picture it

```
  you type canteen-nashik.in
        │
        ▼
  ┌───────────┐   not cached?   ┌──────────────────────┐
  │ browser + │────────────────▶│ your network's or    │
  │ OS cache  │                 │ operator's resolver  │
  └───────────┘                 └──────────┬───────────┘
        ▲                                  │ asks up the chain
        │ "103.21.x.x"                     ▼
        │                        ┌──────────────────────┐
        └────────────────────────│ the domain's own     │
                                 │ nameservers answer   │
                                 └──────────────────────┘
                 then: open a connection to 103.21.x.x on port 443
```

Your device asks a resolver for the name, the resolver asks the domain's nameservers unless it already has a cached answer, and the returned number is then connected to on a specific port.

### Where it lives

This chapter is the **network** box from M00's four boxes. A request cannot begin until the name resolves, which is why a DNS problem looks exactly like a dead server. M06 is where you point a domain at a deployment and issue certificates — and a certificate needs DNS resolving first, which is why DNS mistakes delay deploys rather than merely breaking them.

A machine runs many programs, so each listens on a numbered **port**: 80 for HTTP, 443 for HTTPS, 5432 for Postgres, 3000 or 5173 for development servers. `localhost` and `127.0.0.1` mean **this machine only** — a server at `localhost:3000` is reachable from your laptop and nowhere else on earth, not even your own phone on the same Wi-Fi.

### Words to own

| Term | Plain meaning | Example |
|---|---|---|
| IP address | The number that identifies a machine on a network | `103.21.244.12` |
| IPv4 / IPv6 | The older short numbers, and the newer longer ones | `1.2.3.4` vs `2001:db8::1` |
| Domain name | The human-readable name people type | `canteen-nashik.in` |
| Subdomain | A name under your domain | `staff.canteen-nashik.in` |
| Apex / root | The domain with nothing in front of it | `canteen-nashik.in` |
| DNS | The system that turns names into addresses | The campus directory |
| Resolver | The service your device asks | Your operator's, or a public one |
| Nameserver | The authority that holds your records | Set at your registrar |
| A record | "This name points at this IPv4 address" | `@ → 103.21.244.12` |
| CNAME | "This name is an alias for that other name" | `www → app.platform.app` |
| TTL | How long others may cache this answer | 300 seconds |
| Port | Which program on the machine should answer | 443 for HTTPS |
| `localhost` | This machine only | `127.0.0.1` |

### Tradeoffs and gotchas

**Lower the TTL *before* you change the record, not after.** Drop it to 300 seconds, wait for the old long TTL to expire, then make the change — the world picks it up in five minutes instead of a day. In the other order the low TTL is itself stuck behind the old cache.

**You cannot force other people's caches to clear.** Your own device and browser are the end of your power, and during a change two users correctly see two different sites.

**A CNAME traditionally cannot sit on the apex.** `www.canteen-nashik.in` can be a CNAME; the bare domain historically cannot, which is why platforms offer an apex IP or their own flattening. An AI will hand you an apex CNAME without mentioning it.

**One typo costs a day:** a trailing dot, a hostname from memory, an A record holding a name. Paste from the platform's instructions. And set a renewal reminder — an expired domain takes your site, email and certificates with it.

**Do not expose your database's port.** A database on `0.0.0.0:5432` with a weak password is found by scanners within hours. Ports are doors; most should not open onto the street (M07).

### Verify it

1. **Ask what the name resolves to from the command line.** `nslookup` or `dig` gives the answer your machine has right now, with no browser cache in the way.
2. **Check it from a second network.** Two different answers means a change in flight; two matching wrong answers means the record is wrong.
3. **Read the TTL before changing anything.** It tells you how long you will wait afterwards.
4. **Check whether anything is listening on the port.** Name resolving but connection refused means DNS is fine and the server is not.
5. **Never accept `localhost` as evidence.** Ask for a URL you can open on your phone, on mobile data.
6. **Confirm the apex and the `www` version both work.** Teams set up one and forget the other; users type both.
7. **Check the registrar's nameservers match where you edit records.** Editing in a platform dashboard while the nameservers point at the registrar's default is a silent, common failure.

Red flags: an IP quoted from memory rather than the platform's instructions · an apex CNAME with no mention of the limitation · "wait for propagation" as a diagnosis rather than a description · flushing your DNS cache offered as a fix for other people's problem · a database or admin port exposed publicly · a claim of working evidenced only by `localhost`.

## Walkthrough
1. Open a terminal and run `nslookup` against a site you use daily — `nslookup flipkart.com` or similar. Read the answer: the resolver that replied, and the address or addresses returned.
2. Run it again against a smaller site. Note whether you get one address or several, and note that several is normal and is how load is spread.
3. Now run `nslookup` for the `www` version of the same domain. Compare the answer with the apex version. Note whether you got an address or an alias.
4. Run the same lookup from your phone on mobile data, using any DNS lookup app or site. Compare the answer with your laptop's. If they differ, write down what that tells you.
5. Find the TTL on a record if your tool shows it. Write it down and convert it to minutes. That is how long a change to that record would take to reach everyone.
6. In your browser, open DevTools, go to the Network panel and click the main document request. Look for the remote address in the Headers section. That is the actual machine you are talking to, and the port it answered on.
7. Start any local development server on your own laptop and open it at `localhost:3000`. Then try to open exactly that URL on your phone, on the same Wi-Fi. Note what happens and why.
8. Find out your laptop's address on the local network, and try reaching the same server from your phone using that address and port instead. Note whether it works, and what that tells you about the difference between `localhost` and a network address.
9. Deliberately try a port nothing is listening on — open `localhost:3999` in your browser. Read the exact error. Compare it with the error you get for a domain name that does not exist. Write down how the two errors differ, because telling them apart is the skill.
10. Write a short pre-flight checklist for your own team's eventual domain setup: what you will check before changing a record, what you will check after, and what evidence you will accept that the site is actually reachable by someone who is not you.

## Talk to AI
```prompt
CONTEXT:      I direct AI rather than configuring servers. My three-person
              team is about to point a .in domain at a free-tier hosting
              platform for our canteen ordering app, ahead of a demo in
              four days. We have never done this. Our users are students in
              Nashik, mostly on mobile data. We will need both the apex
              domain and the www version to work, and we will later need a
              staff subdomain. We also need HTTPS, which I understand
              depends on DNS being correct first.

GOAL:         A step-by-step DNS plan with a pre-flight checklist and a
              diagnosis guide, written so that a non-technical teammate
              could follow it and so that we do not lose a day to caching.

REQUIREMENTS: 1. The exact order of operations, including when to lower the
                 TTL relative to making the change, and why that order
                 matters
              2. What record type to use for the apex and what for www,
                 with the apex CNAME limitation explained plainly and the
                 two standard ways around it
              3. A pre-change checklist: everything to verify or write down
                 before touching a record
              4. A post-change verification procedure using command-line
                 lookups, including how to check from two different
                 networks and what disagreement between them means
              5. A diagnosis table: for each of "name does not resolve",
                 "name resolves to the wrong address", "resolves correctly
                 but connection refused", and "works on one device only" —
                 the likely cause and the next check
              6. A plain statement of what we cannot control, so we do not
                 waste hours trying

CONSTRAINTS:  Free tiers only. Assume we are editing records in the
              registrar's dashboard unless you tell us to move the
              nameservers, in which case say exactly why. Do not give us
              any IP address from memory — tell us where to read the
              current correct value from instead. Commands must work on
              Windows. Assume we have four days and cannot afford a
              24-hour cache wait.

NON-GOALS:    Do not configure the hosting platform itself. Do not set up
              email. Do not explain how DNS works internally beyond what we
              need to act. Do not recommend any paid DNS service.

ACCEPTANCE:   - The plan states when to lower the TTL and why that step
                comes before the change
              - Both the apex and the www version are covered explicitly
              - Every verification step is a command or an action with a
                stated expected result
              - The diagnosis table lets me tell a DNS problem from a
                server problem without guessing
              - Nowhere does the plan accept localhost as evidence that
                anything works
              - A teammate who has never done this could follow it alone

ASK:          Before writing the plan, ask me the three or four questions
              whose answers would change it. Then give the plan. Then list
              separately the two mistakes teams most commonly make in
              exactly this situation and how each one would look to us when
              it happened.
```

## Stuck? Ask AI
```prompt
Explain what DNS is using an analogy that is not a phone book or a directory. Then explain why a change to a DNS record is not instant, where the delay actually lives, and why nobody — not even the domain owner — can force it to be faster. Finish by telling me what I can and cannot clear myself.
```

```prompt
Our domain works on my phone and not on my laptop, and a teammate sees the old site. Give me an ordered diagnostic list, cheapest check first, with the exact command to run at each step on Windows and what the result means. Tell me at which point I should conclude that the only correct action is to wait, and roughly how long.
```

```prompt
Explain what a port is and why a server running at localhost:3000 cannot be opened by anyone else, including my teammate on the same Wi-Fi. Then explain the difference between the error I get for a port with nothing listening and the error I get for a domain that does not exist, and why telling those two apart is the most useful thing I can learn here.
```

## Quiz

### Q1 · L2
What does a DNS resolver do?
- (a) Stores your website's files and serves them
- (b) Turns a domain name into the IP address of a machine that can answer ✅
- (c) Encrypts the connection between the browser and the server
- (d) Decides which port the server listens on
> DNS is a lookup, nothing more. The name is for humans; the number is what a connection is actually opened to. Encryption is chapter 14's subject, not this one.

### Q2 · L3
You change an A record and the old site still appears on your laptop while your phone shows the new one. What is happening?
- (a) The hosting platform has not finished deploying
- (b) Cached answers are still in force on some resolvers and not others, which is normal mid-change ✅
- (c) The record was saved with the wrong type
- (d) Your laptop's network is blocking the new address
> Thousands of independent caches expire on their own schedules. Two devices correctly seeing two different sites is what a DNS change in flight looks like — not an intermittent server fault.

### Q3 · L4
You are about to move your domain to a new host and you want the switch to take minutes rather than a day. What do you do first?
- (a) Make the change, then flush your DNS cache
- (b) Lower the TTL and wait for the old TTL to expire, then make the change ✅
- (c) Change the nameservers at the registrar at the same time
- (d) Make the change and ask users to clear their browser cache
> The low TTL must itself be cached before it can help. Doing it in the other order leaves the new short TTL stuck behind the old long one, which is the whole day you were trying to avoid.

### Q4 · L3
A teammate says the app works and shows you `localhost:3000` on their laptop. What have you learned?
- (a) The app is deployed and reachable
- (b) That it runs on that one machine, and nothing about whether anyone else can reach it ✅
- (c) That DNS is configured correctly
- (d) That the app works on port 3000 for all users
> `localhost` means this machine only, by design. The evidence you need is a URL you can open on your own phone, on mobile data.

### Q5 · L5
Your domain resolves to the correct IP address, but opening it gives "connection refused". Where is the problem?
- (a) In DNS — the record must be wrong
- (b) In the caches, which have not expired yet
- (c) At the machine — nothing is listening on the port you connected to ✅
- (d) In the browser, which needs a hard reload
> Resolution succeeded, so the name is doing its job. A refused connection means you reached the machine and no program accepted you — the server is not running, or is listening on a different port.

## Go deeper
- [MDN: What is a domain name?](https://developer.mozilla.org/en-US/docs/Learn_web_development/Howto/Web_mechanics/What_is_a_domain_name)
- [Cloudflare Learning: DNS records explained](https://www.cloudflare.com/learning/dns/dns-records/)
