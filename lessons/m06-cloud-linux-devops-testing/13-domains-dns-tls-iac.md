---
id: m06-13-domains-dns-tls-iac
module: m06-cloud-linux-devops-testing
slug: domains-dns-tls-iac
title: "Domains, DNS Records, TLS Certificates and Infrastructure as Code"
order: 13
duration_min: 15
is_free: false
mission: false
volatility: low
last_reviewed: 2026-11-16
---

## Key learnings
- Read a DNS record and say what it points at and how long it is cached
- Explain why a domain change can take hours to reach some of your users
- Say what a TLS certificate proves, and what it does not
- Describe what infrastructure as code gives you, and when a text file beats it
- Recognise a DNS or certificate answer that is guessing rather than checking

## Lesson

### The pain

A team buys `canteenorder.in`, points it at their app, announces it. One teammate sees the old placeholder page for six hours: their phone's network is holding the previous answer in a cache nobody controls.

Then the browser says "Not secure" in red: the certificate was issued for the platform's subdomain, not the new domain. Three weeks later a teammate deletes the one cloud resource nobody could identify — the bucket serving dish photos, created by clicking a console in week two.

### The idea

**A domain name is a lookup table the whole internet caches, a certificate is a time-limited proof that you control that name, and infrastructure as code is the same idea applied to everything else you created by clicking.**

All three are one idea: your product's important configuration belongs written down, with an owner and an expiry, not in a console's memory and yours.

A certificate encrypts traffic and proves its holder controls the name. The padlock means private and verified, not safe or honest: a phishing site has a valid padlock.

### The analogy

The canteen moves from Block A to Block C. The new location is a fact, but the campus noticeboard, the hostel map and last year's WhatsApp group all still say Block A. The board updates instantly; the copies people hold expire on their own clock, and anyone with an old one finds nothing. That is DNS; the delay is propagation. The health licence on the wall is the certificate: verified, dated, expiring. The recipe binder is infrastructure as code.

**Where the analogy breaks:** a noticeboard has one copy. DNS is cached by your phone, your browser, your provider and intermediate resolvers, each with its own clock, and *none* answers to you. You can shorten the clock in advance; you cannot clear someone else's cache.

### Picture it

```
  student types canteenorder.in
        │
        ▼
  ┌──────────────┐  cached?  ┌──────────────────┐
  │ phone / OS   │──────────▶│ answer, for TTL  │
  └──────┬───────┘   yes     │ seconds          │
         │ no               └──────────────────┘
         ▼
  ┌──────────────┐    ┌──────────────┐    ┌─────────────┐
  │ network      │───▶│ authoritative│───▶│ A / CNAME   │
  │ resolver     │    │ nameserver   │    │ record      │
  └──────────────┘    └──────────────┘    └──────┬──────┘
                                                 │
                                                 ▼
                                         your app's address
                                         + TLS handshake
```

A name is resolved by asking a chain of caches before reaching the nameserver that holds your records, and every cache in that chain keeps its answer for the record's TTL.

### Where it lives

| Record | What it does | Typical use |
|---|---|---|
| A | Name → IPv4 address | `canteenorder.in` → an IP |
| AAAA | Same, for IPv6 | Alongside A |
| CNAME | Name → another name | `www` → platform host |
| TXT | Arbitrary text | Proving you own the domain |
| MX | Where email for this domain goes | Your email provider |
| NS | Which nameservers are authoritative | Set at the registrar |

Infrastructure as code means cloud resources described in text files, in git, applied by a tool. You do not need the tool yet; you need the discipline now, and chapter 6's `STACK.md` — every resource, its owner, purpose, region and non-default settings — gives most of the value for a tenth of the effort.

### Words to own

| Term | Plain meaning | Example |
|---|---|---|
| Domain / registrar | The name, and who you bought it from | `canteenorder.in` |
| Nameserver | The server that answers for your domain | Set at the registrar |
| A record | Name to IPv4 address | `canteenorder.in` → IP |
| CNAME | Name to another name | `www` → platform host |
| TXT record | Arbitrary text, used for ownership proof | Domain verification |
| TTL | Seconds a cache may keep the answer | Lower it before changes |
| Propagation | Waiting for old cached answers to expire | Up to the old TTL |
| TLS certificate | Proof you control the name, plus encryption | Free, auto-renewed |
| Certificate authority | The body that issues and vouches | Let's Encrypt |
| Apex / root domain | The name with no subdomain | `canteenorder.in` |
| Infrastructure as code | Resources described in text, in git | Terraform, in M13 |
| Drift | Reality no longer matches the written description | Someone clicked |

### Tradeoffs and gotchas

- **Lower the TTL before the change, not after.** TTL is the seconds a cache may keep an answer: 3,600 means an hour of old answers. Lower it to a few minutes a day before a planned change, then raise it after. Lowering it afterwards does nothing, because caches already took the long value.
- **Use a CNAME to your platform's hostname, not an A record to its IP,** because platform IPs change without notice and an A record on a reassigned IP is a silent outage. A plain CNAME is also banned on the apex, hence platforms' special records.
- **Never announce a domain change until you have checked it from a network you do not control:** mobile data, or a teammate elsewhere.
- **Certificates are free, automatic and per name;** paying for a basic one buys nothing. They renew every 90 days, so an expired certificate is an automation failure; a new domain needs a new one, so re-check the padlock after a domain change.
- **Email records are easy to destroy by accident,** so copy the record set first; and a tool that creates forty resources can delete forty.

### Verify it

DNS and certificates are where AI's guesses are hardest to tell from facts.

1. **Make it tell you what to run, and run it yourself.** A lookup shows the record and its TTL; `curl -i` shows what the name serves. An answer not preceded by a lookup is a guess.
2. **Check the record at the right place.** Your registrar and your DNS host can be different, and a record at the registrar does nothing if the NS records point elsewhere.
3. **Verify the certificate covers the exact name you use,** with and without `www`: a platform-subdomain certificate does not cover a custom domain.
4. **Before any destructive infrastructure step, make it show what will be deleted** — "this will remove these three things" is a sentence you must see.

Red flags: an A record at a platform's IP instead of a CNAME; "wait for propagation" with no mention of TTL; paying for a certificate; a DNS change that wipes MX records; an infrastructure command without the plan.

## Walkthrough
1. Find who your registrar is and where your DNS records are actually hosted. They may be the same or different. Write both down — this is the fact that saves you an hour later.
2. List every DNS record currently on your domain, including MX and TXT. Copy the whole list into a file in your repository before you change anything.
3. Ask AI for the read-only lookup command for your operating system, and run it against your domain. Read the record type, the value and the TTL in the output.
4. Note the TTL. If it is an hour or more and you plan a change this week, lower it now to a few minutes and note the time — you must wait out the old value before the new TTL helps.
5. Add your custom domain in your platform's dashboard and read the exact record it asks for. Note whether it wants a CNAME, an apex-specific record type, or a TXT record to prove ownership.
6. Add the record at the correct place — the DNS host, not necessarily the registrar — and then run your lookup command again until you see the new value.
7. Open the domain in a browser and check the padlock. If it warns, find the certificate status in your platform's domain settings and wait for issuance rather than changing anything.
8. Check both `canteenorder.in` and `www.canteenorder.in`, or your equivalents. Confirm one redirects to the other and that both show a valid certificate.
9. Verify from a network you do not control: open it on your phone on mobile data, and ask a teammate on a different connection. Only now is it safe to share the link.
10. Find your certificate's expiry date in the browser's certificate details and write it down. Then confirm your platform renews automatically, and note where you would see a failure.
11. Finally, update `STACK.md`: every resource you own, who created it, its region, and the non-default settings. Then export your environment variable *names* with the values removed and commit that list. That is your free, honest infrastructure as code for now.

## Talk to AI
```prompt
CONTEXT:      I am a student on an AI-native engineering program. I direct AI
              rather than writing code. My team of three is building
              [describe your product in two sentences], deployed on [your
              platform] on a free tier. We have bought the domain [your
              domain] from [your registrar]. Our users are students in India
              on budget Android phones and slow 4G. We have a demo in [number]
              days and the domain must work and show a valid padlock. We also
              have cloud resources that were created by clicking in consoles
              and nobody has written down.

GOAL:         Get our custom domain serving our app over HTTPS with no browser
              warning, without a propagation surprise, and write down our
              infrastructure so a deleted resource can be rebuilt.

REQUIREMENTS: 1. Explain A, CNAME, TXT and MX records and TTL in plain words,
                 and tell me which ones my setup needs
              2. Tell me how to find out whether my registrar or someone else
                 is hosting my DNS, and why adding a record in the wrong place
                 silently does nothing
              3. Give me the exact ordered steps to point my domain at my
                 platform, including the apex versus www question, and tell me
                 what to do about TTL BEFORE I make the change
              4. Explain what a TLS certificate proves and what it does not,
                 and tell me why adding a custom domain needs a new one
              5. Give me the read-only commands to verify the record and the
                 certificate myself, and tell me what a correct result looks
                 like
              6. Tell me what to write down about our existing cloud resources
                 so we could rebuild them, without introducing a new tool

CONSTRAINTS:  Free tier only - do not suggest buying a certificate. Plain
              English, second-language reader. Label every command READ-ONLY
              or CHANGES-THINGS. Preserve our existing email records. Do not
              have me replace my whole record set. We cannot afford downtime
              in the [number] days before our demo, so tell me the risk of
              each step and the point of no return.

NON-GOALS:    Do not introduce Terraform or any infrastructure-as-code tool -
              a text file in our repository is what we want for now. Do not
              set up a CDN, email sending, or subdomains for staging. Do not
              change our hosting platform.

ACCEPTANCE:   - My domain and its www form both load my app with a valid
                padlock, checked on mobile data and on a teammate's network
              - I can state the TTL of my records and what that means for my
                next change
              - My existing MX records still work
              - I know my certificate's expiry date and where a renewal
                failure would show
              - Every cloud resource we own is in a committed file with its
                purpose, region and owner

ASK:          First ask me who my registrar is, where my DNS is hosted, which
                platform I deploy on, and how many days until the demo. Then
              tell me the order of operations and what the riskiest step is,
              before giving me any instruction to carry out.
```

## Stuck? Ask AI
```prompt
Here is the output of a DNS lookup for my domain: [paste the complete output]. Explain every part of it: the record type, the value it points at, the TTL and what that TTL means for me in practice. Then tell me whether this record is pointing where I think it is, and give me the read-only command that would show me what that address actually serves.
```

```prompt
My browser shows a certificate warning on my custom domain, but my platform's own subdomain works fine over HTTPS. Explain why adding a custom domain requires a new certificate, what has to be true for it to be issued, and the three most common reasons issuance gets stuck. For each, tell me the read-only check that would confirm it. Do not tell me to buy a certificate.
```

```prompt
Explain infrastructure as code and then argue honestly against using it for a three-person student team with four cloud resources. Tell me what I would gain, what it would cost me in time and risk, and what a plain text file in my repository should contain to get most of the benefit. Then ask me what resources we actually have so you can tell me what to write down about each one.
```

## Quiz

### Q1 · L3
You change your domain's A record, and one teammate still sees the old site six hours later. Why?
- (a) The new record was saved incorrectly
- (b) Their network's resolver cached the old answer for the record's previous TTL, and you cannot clear it ✅
- (c) Their browser needs a hard refresh
- (d) The certificate has not been issued yet
> Every cache in the chain holds its answer for the TTL it was given. Lowering the TTL after the change does nothing; it has to be lowered a day before, so caches take the short value.

### Q2 · L4
Your platform offers both a CNAME to its hostname and an A record to its current IP address. Which should you choose, and why?
- (a) The A record, because it removes one lookup and is faster
- (b) The CNAME, because platform IP addresses change without notice while the hostname does not ✅
- (c) Either — they are functionally identical
- (d) The A record, because CNAMEs are not supported by most resolvers
> An A record freezes an address the platform may reassign, producing a silent outage with no change on your side. The hostname is the stable thing, so point at the name.

### Q3 · L3
What does a valid TLS certificate actually prove to your user?
- (a) That the site is safe and trustworthy
- (b) That the connection is encrypted and the domain name is verified ✅
- (c) That the site's owner has been identified by name
- (d) That the site does not store personal data
> Encryption plus name verification, nothing more. A phishing site can hold a perfectly valid certificate, which is why the padlock is not a safety judgement.

### Q4 · L4
Your custom domain shows a certificate warning while your platform's subdomain works over HTTPS. What is the most likely cause?
- (a) The platform's certificate expired
- (b) Certificates are issued per name, and one has not yet been issued for your new domain ✅
- (c) You need to buy a certificate for a custom domain
- (d) The A record points at the wrong IP
> The existing certificate covers the platform subdomain only. Adding a domain triggers issuance for that name, which requires the DNS record to be resolving first — and certificates on these platforms are free.

### Q5 · L5
Why does this course tell a three-person team to keep a committed text file describing their cloud resources rather than adopt an infrastructure-as-code tool?
- (a) Infrastructure as code does not work on free tiers
- (b) The file captures most of the value — nothing unidentifiable, rebuildable from a record — without the setup cost or the risk that one careless apply deletes a database ✅
- (c) Text files are automatically applied by most platforms
- (d) The tools are only useful once you run a cluster
> The discipline is needed now; the tool is not. A tool that can create forty resources from a file can also destroy them, which is a real reason to learn it properly in M13 rather than in a hurry before a demo.

## Go deeper
- [Cloudflare Learning: DNS records explained](https://www.cloudflare.com/learning/dns/dns-records/)
- [Let's Encrypt: how certificate issuance and renewal work](https://letsencrypt.org/how-it-works/)
