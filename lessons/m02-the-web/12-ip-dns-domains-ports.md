---
id: m02-12-ip-dns-ports
module: m02-the-web
slug: ip-dns-domains-ports
title: "IP Addresses, DNS, Domains and Ports"
order: 12
duration_min: 25
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

A team buys `canteen-nashik.in` the night before their first checkpoint demo. They point it at their deployment, open it, and get nothing. They spend four hours convinced the deploy is broken.

The deploy was fine. Three separate things were happening. The record they created pointed at an address that had changed two days earlier, because they had copied it from an old screenshot. The old record's TTL was 24 hours, so even after they corrected it, their own laptops kept using the cached wrong answer. And one teammate's phone, on mobile data with a different resolver, started working forty minutes in — which convinced everybody that the site was "sometimes up", and sent them hunting for a server problem that did not exist.

Meanwhile the app had been running perfectly the whole time at `localhost:3000` on one laptop, which the team kept pointing at as proof it worked. It was not on the internet at all.

### The idea

**Machines on the internet are found by number, people remember names, and DNS is the lookup that converts the second into the first. A port then says which program on that machine should answer.**

Three layers, three failure modes, and almost every "the site is down" that is not actually a code problem is one of them: the name resolves to the wrong number, the answer is cached somewhere you cannot reach, or nothing is listening on the port.

### The analogy

The canteen on a large campus.

The **IP address** is the physical location: Building C, ground floor, west wing. Precise, unambiguous, and useless for conversation — nobody says "meet me at Building C ground floor west wing", they say "meet me at the canteen".

The **domain name** is "the canteen". Human, memorable, and not actually a location at all.

The **directory board** at the campus gate, which says "Canteen → Building C", is **DNS**. It is a lookup table maintained by somebody, and it can be wrong, and it can be out of date.

The **ports** are the windows. Window 1 sells coupons, window 2 serves food, window 3 is the staff entrance and is kept shut. The building is one address; what you get depends entirely on which window you go to.

**Where the analogy breaks:** the campus directory board is one board, and when you repaint it, it is repainted. DNS is thousands of copies, cached in your phone, your browser, your college's network, your mobile operator's resolver and several layers above that, each holding its own copy for as long as the TTL said it could. When you change a record you are not updating a board — you are waiting for thousands of independent copies to expire. You cannot force them. That is why "it works on my phone but not on my laptop" is a completely normal mid-change state, and why what people call "propagation" is really just caching running down.

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

### Ports, and why localhost is not the internet

A single machine runs many programs that all want to talk to the network, so each one listens on a numbered **port**. Port 80 is where plain HTTP traffic is expected, 443 is HTTPS, 5432 is where a Postgres database usually listens. Development servers typically use 3000, 5173 or 8080 — numbers chosen because nothing important uses them.

`localhost` and `127.0.0.1` both mean **this machine, and only this machine**. A server at `localhost:3000` is reachable from your laptop and from nowhere else on earth. Not from your phone on the same Wi-Fi, not from your teammate sitting next to you, not from a mentor. This is a deliberate safety default, and it is also the cause of an entire genre of "but it works on my machine" conversations in the live session.

The related error you will meet in week one is "port already in use", which means a previous run of your app is still alive and holding port 3000. The honest fix is to stop the old process, not to move to port 3001 and forget.

### Where it lives

This chapter is the **network** box from M00's four boxes, and it sits between every client and every server. A request cannot even begin until the name has been resolved, which is why a DNS problem looks exactly like a dead server from the browser's point of view.

M06 is where you actually point a domain at a deployment and where HTTPS certificates get issued — and a certificate can only be issued once DNS resolves correctly, which is why DNS mistakes delay deploys rather than just breaking them. M05's CDN discussion relies on DNS to send users to a nearby server. And chapter 14's certificates are issued for a *name*, which is why the name and the machine have to agree.

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

**Lower the TTL *before* you change the record, not after.** This is the one operational trick in this chapter and it is worth memorising. Drop the TTL to 300 seconds, wait for the old long TTL to expire, then make the actual change — now the world picks it up in five minutes instead of a day. Doing it in the other order means the low TTL is itself stuck behind the old cache.

**You cannot force other people's caches to clear.** You can flush your own device's cache and your browser's, and that is the end of your power. Anyone telling you to "just wait for propagation" is describing caching honestly, even if the word is wrong.

**During a change, two users will correctly see two different sites.** This is not a bug and it is not intermittent failure. It is what a mid-change DNS state looks like, and knowing that saves the four hours from the pain story.

**A CNAME traditionally cannot sit on the apex.** `www.canteen-nashik.in` can be a CNAME; `canteen-nashik.in` historically cannot, which is why hosting platforms offer either a specific IP for the apex or their own flattening feature. An AI will often hand you a CNAME for the apex without mentioning this.

**One typo costs a day.** A trailing dot, a missing dot, the platform's hostname spelled from memory, the record type set to A when the value is a hostname. Copy and paste the value from the platform's own instructions, and read it back character by character before saving.

**`.in` and `.co.in` domains are cheap and renew.** Set a calendar reminder for renewal. An expired domain takes your site, your email and your certificates with it, and recovering one is slow.

**Do not expose your database's port.** A database listening on `0.0.0.0:5432` with a weak password is found by automated scanners within hours. M07's security chapter covers this properly; the habit to form now is that ports are doors and most should not open onto the street.

### Verify it

DNS verification is the one place in this module where you leave the browser.

1. **Ask what the name resolves to, from the command line.** `nslookup canteen-nashik.in` on Windows, or `dig canteen-nashik.in` elsewhere, tells you the answer your machine is getting right now, with no browser cache in the way. Compare it against what you intended.
2. **Check it from a second network.** Your laptop on college Wi-Fi and your phone on mobile data use different resolvers. Two different answers means a change is in flight; two matching wrong answers means the record itself is wrong.
3. **Read the TTL on the current record before changing anything.** It tells you how long you will be waiting afterwards. This is a thirty-second check that sets your expectations correctly.
4. **Check whether anything is listening on the port.** If the name resolves correctly and the connection is refused, DNS is fine and the server is not running or not listening where you think. These are two completely different problems and distinguishing them is most of the diagnosis.
5. **Never accept `localhost` as evidence.** If a teammate or an AI shows you something working at `localhost:3000`, that is a statement about one machine. Ask for a URL you can open on your phone, on mobile data.
6. **Confirm the apex and the `www` version both work.** Teams routinely set up one and forget the other, and users type both.
7. **Check the registrar's nameservers match where you are editing records.** Editing records in a platform's dashboard while the domain's nameservers still point at the registrar's default is a surprisingly common and completely silent failure.

Red flags in AI output and advice: an IP address quoted from memory rather than from the platform's current instructions; a CNAME proposed for the apex with no mention of the limitation; "wait for propagation" offered as a diagnosis rather than as a description; advice to flush your DNS cache as a fix for other people's problem; a database or admin port exposed publicly; and any claim that something works, evidenced only by `localhost`.

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
