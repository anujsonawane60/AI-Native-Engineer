---
id: m13-11-security-hardening
module: m13-production-engineering
slug: security-hardening
title: "Security Hardening: Threat Modelling, Pen-Test Basics, Dependency Scanning"
order: 11
duration_min: 30
is_free: false
mission: false
volatility: low
last_reviewed: 2026-11-16
---

## Key learnings
- Run a threat model by drawing trust boundaries and asking four questions at each
- Triage dependency findings by reachability, so the list stays short enough to act on
- Test your own staging with a free scanner, and know the legal line you must not cross
- Build an authorization matrix and test every forbidden cell
- Catch a security report whose findings have all been ignored for three months

## Lesson

### The pain

M07 chapter 1 taught you the OWASP Top Ten and the observation that nine of the ten are an application trusting something it had no business trusting. It gave you `/orders/8421` becoming `/orders/8420` and a ₹1 thali from an edited price. M07 chapter 2 taught you secrets: the three categories, the forbidden destinations, and the revoke-first leak sequence. That is the knowledge. This chapter is the *practice* that keeps it true over months, which is a different problem, because your attack surface grows every week whether you work on it or not.

Here is what that looks like. A team enables a free dependency scanner in month two. It reports 180 findings, 23 of them marked high. Nobody has time, so nobody looks. By month five the badge is red, the number is 340, and the team has learned to see it as decoration — exactly the pattern from chapter 6's flaky tests, applied to security. One of those 340 is a real, reachable, remotely exploitable flaw in an image-processing library their upload endpoint calls directly. It had been sitting in the list for eleven weeks. The scanner did its job perfectly and bought them nothing, because an unread list is not a control.

Second failure, newer and specific to how you work. An agent, asked to add a CSV export, installs a package with forty weekly downloads and one maintainer. It works. It also runs a script at install time that reads environment variables. Nobody reviewed the dependency, because the diff everyone read was the application code and the dependency arrived in a lockfile line. M07's secrets chapter told you a key in the browser is public; this is the newer version — a key in your build environment is readable by every package you install.

### The idea

**Hardening is three repeatable practices, not a one-off audit: draw your trust boundaries and ask four questions at each, triage dependency findings by whether your code can actually reach them, and test your own system the way an attacker would.**

The four questions, asked at every boundary where data crosses from something you do not control into something you do: *what are we building here, what can go wrong, what are we doing about it, and did we do a good enough job?* That is threat modelling, and it is a forty-minute conversation with a diagram, not a document.

### The analogy

M07 gave you the canteen at 1 p.m. with three doors: the staff door with no lock, the order slip the cashier reads literally, and the store room whose lock was never fitted.

Now the hygiene inspection arrives, and it has two halves. The first is the kitchen itself: surfaces, temperatures, who may stand where. That is the part you control and the part M07 taught you. The second half is the **supplier audit**, and it is the half everyone forgets. The inspector asks where the paneer came from, whether that dairy has a licence, and what happened the week their cold chain failed. You can cook perfectly and still poison forty students with an ingredient you did not check.

Then the triage problem. The inspector leaves a sheet with sixty items. Four of them matter — the fridge at the wrong temperature, the unlocked store room. Fifty-six are about paint and signage. A kitchen that treats all sixty as equal fixes none of them, and a kitchen that ignores the sheet because it is long misses the fridge.

**Where the analogy breaks:** the inspector visits once a term, and between visits the kitchen stays as inspected. Your dependencies change under you. A package that was clean on Tuesday has a published exploit on Friday, posted publicly with instructions, and the attacker scanning the internet for it has automation you do not. Security is not a state you reach; it is a rate you have to keep up with.

### Picture it

```
        TRUST BOUNDARIES — a check must live at each ╎
  ┌───────────┐     ╎     ┌──────────────┐    ╎    ┌──────────┐
  │ student's │────▶╎────▶│ your server  │───▶╎───▶│ payment  │
  │ phone     │ req ╎     │ authn +authz │key ╎    │ provider │
  └───────────┘     ╎     └──────┬───────┘    ╎    └──────────┘
   untrusted        ╎            │            ╎   trusted but
   input            ╎            ▼            ╎   not owned
                    ╎     ┌──────────────┐    ╎
                    ╎     │ database     │    ╎
                    ╎     └──────────────┘    ╎
                    ╎                         ╎
      dependencies ─╎──▶ run INSIDE your server with its
      (code nobody  ╎    environment variables and its network
       on your team ╎
       has read)    ╎
```

Each dotted line is a trust boundary where data or code crosses from something you do not control into something you do, and the four questions get asked at every one — including the boundary your dependencies cross, which is inside your server rather than in front of it.

### Where it lives

This extends M07 chapters 1 and 2 across time, and it touches four chapters of this module. Chapter 4 is where least-privilege permissions become reviewable: an IAM policy in code is a diff you can read, and "this service can write to one bucket" is far easier to verify than a console checkbox. Chapter 5's kill switch is a security control, because the fastest response to an abused feature is turning it off. Chapter 6 is where a security fix becomes permanent: a test that reproduces the IDOR is worth more than the fix alone. Chapter 10 is where a suspected breach becomes an incident, and a security incident has one extra step — preserve evidence before you mitigate, because you may need to know what was taken.

The India-specific part: under the DPDP rules, the strongest control available to a student team is collecting less. A phone number you never stored cannot leak. Before hardening a field, ask whether you need it.

### Words to own

| Term | Plain meaning | Example |
|---|---|---|
| Threat model | A diagram plus the four questions | 40 minutes, per feature |
| Trust boundary | Where data crosses into something you control | The request handler |
| Attack surface | Every way input reaches your system | Endpoints, uploads, webhooks |
| Authorization matrix | A grid of roles against actions | Student may not refund |
| IDOR | Changing an id in a URL to read another's data | `/orders/8420` |
| Dependency scanning | Automated checks for known flaws in packages | Runs on every push |
| CVE | A published identifier for one known flaw | With a severity score |
| Reachability | Whether your code actually calls the flawed path | The triage question |
| SBOM | A list of everything your build includes | One file, generated |
| Lockfile | Exact pinned versions of every dependency | Committed |
| Supply chain attack | Malicious code arriving through a package | Install-time script |
| Rotation | Replacing a credential on a schedule | Every 90 days |
| Least privilege | The smallest permission that works | One bucket, write only |
| Pen test | Deliberately attacking a system you own | Staging, with permission |
| Data minimisation | Not collecting what you do not need | The best control |

### Tradeoffs and gotchas

**Triage by reachability or the list will beat you.** A critical flaw in a library your code never calls is less urgent than a medium one in the path your upload endpoint runs on every request. Sort findings into three buckets: reachable and exploitable in your deployment, present but not reachable, and development-only. Fix the first bucket on a deadline, record a decision for the second, and ignore the third openly in writing. A short list you act on beats a complete list you do not. **A hundred unread findings is worse than ten read ones**, because it teaches the team to look away.

**Pin your dependencies and read the lockfile diff.** The lockfile is where a supply-chain attack arrives, and it is the part of a pull request nobody reads. Make a rule: any new dependency gets a human look at its age, maintainer count, weekly downloads and whether it runs anything at install time. This is the newest and most AI-specific risk in the course, because an agent adding a package is a two-second action with a permanent consequence.

**Your build environment holds secrets, and every package you install runs inside it.** M07 said a key in the browser bundle is public. The extension: a key in your CI environment is available to every install script in your dependency tree. Scope your CI credentials to the minimum and never give a build pipeline production database access.

**Only ever attack what you own, and only where you are allowed.** Running a free scanner against your own staging environment is education. Running it against a classmate's site, a college portal, or a payment provider's sandbox is a crime in India under the IT Act and will end your internship. Write the target in your notes before you start, and never test a third party — which is also chapter 7's rule for the same reason.

**Test the authorization matrix, not the login.** M07's broken-access-control entry is the top of the list for a reason: almost everyone checks that login works and nobody checks that a logged-in student cannot reach the staff page, edit another student's order, or call the refund endpoint directly with a crafted request. Build the grid of roles against actions, then test every cell that should say no. Most of them have never been tried.

**Rotate credentials on a schedule, and prove it by doing it once.** An unexecuted rotation procedure is chapter 9's untested backup in a different costume. Rotate one key on a quiet afternoon and find out where it was hard-coded.

**A scanner's green tick is not a security result.** Scanners find known patterns. They do not find business-logic flaws: a discount applied twice, a refund endpoint missing an ownership check, a price trusted from the client. Those come only from the threat model and from a human asking "what would I try if I wanted a free thali?"

### Verify it

1. **Count your reachable findings, not your total findings.** If nobody can tell you how many of the 340 are reachable from your own code, triage has not happened and the scanner is decoration.
2. **Check the age of the oldest unactioned high-severity finding.** Anything older than a month tells you the process is not real, regardless of what the dashboard says.
3. **Read the lockfile diff in your last five pull requests.** Note any package you cannot explain. For each, check its maintainer count and whether it runs at install time.
4. **Pick three cells from your authorization matrix that should say no, and try them yourself** — with a crafted request, not through the UI, because the UI hides the button and the endpoint does not.
5. **Ask AI for the threat model's *missing* half.** Generated threat models cover the login page and the database and almost never cover webhooks, file uploads, background jobs, admin tooling, or the dependency boundary. Name those explicitly and ask again.
6. **Ask what data you could stop collecting.** An answer that only hardens existing fields has skipped the strongest control available to you.
7. **Confirm your CI credentials cannot reach production data.** This is a single check and it closes the newest hole in the chapter.

Red flags: a dependency scanner whose findings are all ignored; a threat model that lists only the login page; a new dependency with one maintainer and forty weekly downloads added by an agent; secrets available to the build pipeline with production scope; a scanner run against something you do not own; an authorization fix with no regression test; a severity list with no reachability column; and "we are secure because we use HTTPS", which M06 chapter 13 already told you proves only encryption and a name.

## Walkthrough
1. Draw your attack surface on one page: every endpoint, every file upload, every webhook, every background job, every admin tool, and the dependency boundary. Count the items. Most teams find twice what they expected.
2. Mark the trust boundaries on that drawing — every place data or code crosses from something you do not control into something you do. Put a mark where the check lives today, and an empty mark where no check exists.
3. Run the four questions on the single riskiest boundary: what are we building, what can go wrong, what are we doing about it, and did we do a good enough job. Write the answers down. Forty minutes, three people, one page.
4. Build your authorization matrix: roles down the side, actions across the top, yes or no in every cell. Count the no cells. That count is the number of tests you are about to need.
5. Test five of those no cells by hand with a crafted request rather than through the interface. Record which ones were actually enforced. Expect at least one surprise.
6. Turn every failure from step 5 into a chapter 6 regression test, so the fix cannot silently come undone.
7. Enable or open your dependency scanner and export the full list. Then triage it into three buckets — reachable, present but unreachable, development-only — and record the count in each. The reachable bucket is your real work list.
8. Fix the reachable bucket on a stated deadline, and write a one-line decision for every item you are deliberately not fixing. An openly ignored finding is a decision; a silently ignored one is a hazard.
9. Review your lockfile: for every dependency added in the last month, check its maintainer count, its age, and whether it runs anything at install time. Remove anything you cannot justify.
10. Run a free web scanner against your own staging environment only, with the target written down first. Read every finding and classify it as real, a false positive, or not applicable — with a reason for each.
11. Rotate one real credential end to end: revoke, reissue into the platform secret store, confirm the app still works, and note every place you discovered it had been hard-coded. Then check your CI credentials cannot reach production data, and write down which fields you could stop collecting altogether.

## Talk to AI
```prompt
CONTEXT:      I am a student on an AI-native engineering program. I direct AI
              and verify its output; I do not write code by hand. My team of
              three runs [product in two sentences] with [number] real users
              in India, handling [yes/no] real payments. Stack: [your stack],
              [your database]. We collect these personal fields from users:
              [list them all]. Our roles are: [list them, e.g. student,
              staff, admin]. We have file uploads: [yes/no, describe]. We have
              webhooks: [yes/no, from whom]. Our dependency scanner reports
              [number] findings, of which [number] are high, and we have
              actioned [number]. An AI agent adds dependencies to this project.

GOAL:         A repeatable hardening practice: a threat model for our riskiest
              boundary, an authorization matrix we have actually tested, a
              dependency list triaged by reachability, and one credential
              rotated end to end.

REQUIREMENTS: 1. Help me list my complete attack surface - endpoints, uploads,
                 webhooks, background jobs, admin tooling, AND the dependency
                 boundary. Tell me which categories teams always forget
              2. Mark the trust boundaries and run the four threat-modelling
                 questions on the riskiest one. Give me the answers as a page
                 I can commit
              3. Build my authorization matrix: my roles against my actions,
                 with yes or no in every cell. Then give me the crafted
                 requests - not UI steps - to test five cells that should say
                 no, and tell me which is most likely to be unenforced
              4. Give me a triage method for my dependency findings with three
                 buckets: reachable from my own code, present but unreachable,
                 and development-only. Explain why a reachable medium beats an
                 unreachable critical
              5. Tell me exactly what to check before accepting any new
                 dependency, especially one an AI agent added: age, maintainer
                 count, downloads, and install-time scripts. Explain what an
                 install-time script can read in my build environment
              6. Tell me how to confirm my CI credentials cannot reach
                 production data, and what the correct scope for them is
              7. Give me a safe plan to scan my OWN staging environment with a
                 free tool, and state plainly what I must never scan and why
                 that is a criminal matter in India
              8. Give me a credential rotation procedure I can execute today on
                 one real key, and tell me how to find everywhere it is
                 hard-coded
              9. Looking at my list of personal fields, tell me which ones I
                 could stop collecting entirely, and why that is a stronger
                 control than protecting them

CONSTRAINTS:  Free tools only. Plain English for a second-language reader.
              Label every command READ-ONLY or CHANGES-THINGS. Nothing may be
              run against any system I do not own. Do not have me store any
              new personal data. Respect India's data protection rules and say
              where they apply. No step may risk our live payments.

NON-GOALS:    Do not recommend a paid security product or a penetration-testing
              service. Do not aim for a compliance certification. Do not try to
              fix all my scanner findings - help me decide which matter. Do not
              scan or test any third party, including a payment sandbox.

ACCEPTANCE:   - My attack surface is on one page and includes uploads,
                webhooks, background jobs and dependencies
              - I have a committed threat model for my riskiest boundary
              - My authorization matrix exists and I have tested five no cells
                with crafted requests
              - Every authorization failure I found has a regression test
              - My findings are triaged into three buckets with counts
              - Every finding I am not fixing has a written one-line decision
              - I checked every dependency added in the last month
              - My CI credentials cannot reach production data
              - I rotated one real credential end to end
              - I can name at least one field we will stop collecting

ASK:          Start by asking me how many of my scanner's findings are
              reachable from my own code. If I do not know, say plainly that
              the scanner is currently decoration, and make triage the first
              thing we do.
```

## Stuck? Ask AI
```prompt
Explain reachability in dependency scanning. Give me three concrete examples: a critical flaw in a package my code never imports, a medium flaw in a library my upload endpoint calls on every request, and a high flaw in a development-only tool. Rank them by real urgency and explain why that ranking disagrees with the severity scores. Then ask me about three of my own findings, one at a time, and help me place each in a bucket.
```

```prompt
An AI agent added this dependency to my project: [paste the package name and the lockfile lines]. Do not install or remove anything. Tell me what read-only checks I should run to judge it: how old it is, how many maintainers, weekly downloads, whether it runs any script at install time, and what that script could read from my build environment. Then tell me what a safer alternative would be and what I would lose by switching.
```

```prompt
Build me an authorization matrix for a food-ordering app with the roles student, canteen staff and admin, covering actions like view own order, view another's order, change a price, mark sold out, issue a refund, and export user data. Then, for each cell that should say no, give me the crafted HTTP request that would test it directly rather than through the UI. Finally tell me which three cells are most commonly left unenforced in real apps and why.
```

## Quiz

### Q1 · L3
Your scanner shows 340 findings, 23 high, none actioned in eleven weeks. What is the actual problem?
- (a) The scanner is too sensitive and should be reconfigured
- (b) There is no triage, so the list is decoration and a real reachable flaw can sit in it unseen ✅
- (c) The team needs a paid scanner with better accuracy
- (d) The findings should be fixed in severity order from the top
> This is chapter 6's flaky-test pattern applied to security: once the list is ignorable, everything in it is ignored. Ten findings you read and act on beat 340 you do not.

### Q2 · L4
Which finding is most urgent?
- (a) A critical flaw in a package your code never imports
- (b) A high flaw in a tool used only in development
- (c) A medium flaw in an image library your upload endpoint calls on every request ✅
- (d) A critical flaw in a transitive dependency of your test framework
> Reachability beats severity. The severity score describes the flaw in the abstract; what matters is whether an attacker can get to it through your deployed code, which only you can determine.

### Q3 · L4
An agent adds a package with one maintainer and forty weekly downloads, and it runs a script at install time. What is the specific risk?
- (a) The package may be slow or poorly documented
- (b) Install-time code runs inside your build environment and can read the secrets available there ✅
- (c) It may conflict with your other dependencies' versions
- (d) It will increase your deployed bundle size
> This is M07 chapter 2's lesson one layer deeper: a key in your CI environment is readable by everything you install. Scope build credentials to the minimum and never give a pipeline production database access.

### Q4 · L4
You want to practise penetration testing. Which target is acceptable?
- (a) Your college's student portal, since you are a student there
- (b) Your own staging environment, with the target written down first ✅
- (c) Your payment provider's sandbox, since it is not production
- (d) A classmate's project, with their verbal agreement in a group chat
> Only what you own, and only where you are allowed. Unauthorised testing is a criminal matter under India's IT Act, and a payment sandbox is a third party you must never test — the same rule as chapter 7's load testing.

### Q5 · L5
Your scanner is green, your dependencies are pinned, and a student still gets a free thali by editing a price in a request. Why did none of your tooling catch it?
- (a) The scanner needed a deeper configuration profile
- (b) Scanners find known patterns, not business-logic flaws — that requires the threat model and a human asking what they would try ✅
- (c) The flaw was in a transitive dependency not covered by the scan
- (d) Price validation is a frontend concern and out of scope for scanners
> A green tick means no known pattern matched. Trusting a price from the client is M07's insecure-design entry, and the only tools that find it are the four questions at a trust boundary and the authorization matrix.

## Go deeper
- [OWASP: threat modelling and the four questions](https://owasp.org/www-community/Threat_Modeling)
- [OWASP ZAP: getting started scanning an application you own](https://www.zaproxy.org/getting-started/)
