---
id: m17-10-legal-compliance-india
module: m17-distribution-revenue
slug: legal-and-compliance-india
title: "Legal and Compliance Basics for an Indian Startup"
order: 10
duration_min: 15
is_free: false
mission: false
volatility: medium
last_reviewed: 2026-11-16
---

## Key learnings
- Separate what you need to KNOW now from what you need to DO now
- Name the four triggers that turn a student project into something needing an entity
- Treat the DPDP Act's consent and purpose limits as a design input, not a document
- Recognise when to stop reading and pay a CA or a lawyer
- Catch AI that states a threshold, fee or deadline as fact instead of telling you to check

## Lesson

**This chapter is not legal advice, and nobody writing it is your lawyer or your chartered accountant.** It is a map of the decisions that exist, so you know which apply to you, which arrive the day you take real money, and which need a professional. Thresholds, fees, forms and deadlines change, vary by state and by what you sell, and are deliberately not stated here; where a number matters, the official source is named instead.

### The pain

A team takes a ₹5,000 advance from a college society to run their event-registration product for a fest. The money lands in a founder's personal UPI account. The fest is cancelled, the society asks for it back, and two of the three have already spent part of it. Nothing criminal happened — but there is no entity, no separate account, no written refund terms, and no agreement about who owns what. The argument that follows ends the team, over ₹5,000 that a one-page document in week two would have prevented.

A second team has forty paying users and wants to leave test mode. The gateway asks for entity proof, a PAN, a bank account in the business's name, and published refund and privacy policies. None of it exists, and Friday's feature is blocked for three weeks. Neither team hit a legal problem; they hit the wall where other people's money begins.

### The idea

**Compliance is not a checklist you complete; it is a set of decision points, each triggered by something you do.** The useful question is never "are we compliant?" but "which decisions have been triggered for us, today?"

Four triggers matter at your stage. **You take real money from strangers** — not test mode, actual rupees — which triggers the entity question, the bank account, gateway onboarding, a refund policy and tax treatment. **You hold other people's personal data**, which triggers the DPDP Act whether or not you have incorporated. **You sign something with an institution**: who signs, on behalf of what, with what liability. And **you split ownership between founders**, which triggers the one document every student team should have and almost none do. Most of you trigger only the last two.

### The analogy

Three food operations at the same college carry three different obligations. Brownies sold from a hostel room need no licence and no entity. A one-day stall at the fest needs the organisers' permission, a fee, and somebody recording who is responsible if a customer falls ill. The permanent canteen needs a food licence, a registration, bills for everything it buys, and a person whose name is on it all. Same food, driven by **scale, permanence, and whose money and safety are involved** — not by how good the food is.

**Where the analogy breaks:** food rules are local and visible — the licence is on the wall. Software reaches users in states you have never visited, and the obligation often follows the user rather than your laptop, which is why "we are just students in Nashik" answers none of the questions.

### Picture it

```
  KNOW versus DO NOW

  DO NOW, during the program       KNOW, for the day you
  ───────────────────────────      take real money
  founder agreement, one page      ──────────────────────────
  who owns repo, domain, accounts  entity: proprietorship,
  a privacy notice that is TRUE      LLP or private limited
  consent at point of collection   gateway KYC and onboarding
  refund policy in writing         GST for your kind of supply
  delete what you do not need      invoices a buyer can claim
  stay in TEST MODE                contracts with institutions
```

The short list to do during a student program, and the longer list to understand before real money arrives.

| Entity option | What it is, and what it costs you |
|---|---|
| Sole proprietorship | Not a separate legal person — you, trading under a name. Simplest and cheapest, liability personal, fine for small revenue and one owner |
| Partnership | Two or more people doing that, with shared personal liability; rarely right today |
| LLP | Limited liability with lighter ongoing compliance than a company, but awkward for outside investment and cannot issue ESOPs |
| Private limited | What you need to raise money, grant equity or sign substantial contracts — real recurring annual compliance, which continues whether or not the company does anything |
| The decision points | Outside investment; ownership that must survive a disagreement; a bank account in a business name; a counterparty refusing to sign with an individual; real personal liability |

If every one is a no, you probably do not need an entity this month. If unsure, pay a CA for an hour.

### Where it lives

Chapter 4 is where most of this becomes code: the refund policy, the invoice fields, the tax-inclusive price. Chapter 7's capture mechanism needs the consent wording from here, and chapter 9's support channel is where an erasure request arrives. M07's consent and logging and M16's privacy chapter are the implementation. M18 will ask what you would need before taking real money, which is this chapter's second column.

### Words to own

| Term | Plain meaning | Example |
|---|---|---|
| Sole proprietorship | You, trading under a name | No separate entity |
| LLP | Limited liability, lighter compliance | Awkward for investors |
| Private limited | Needed to raise or grant equity | Real annual filings |
| GSTIN | Your GST registration number | Buyers want it |
| KYC | The gateway verifying who you are | The wall before live |
| Data fiduciary | Who decides why data is processed | You |
| Data principal | The person the data is about | Your user |
| Consent notice | Clear statement at collection | Not buried |
| Purpose limitation | Use it only for what you stated | No new uses later |
| Founder agreement | What the three of you agreed | One page, week two |

### Tradeoffs and gotchas

| Gotcha | What to do about it |
|---|---|
| The gateway is the real wall between test mode and live | Gateways run their own KYC and change their requirements — read your gateway's current onboarding documentation rather than any list, including this one |
| GST is where invented facts do the most damage | Whether you must register depends on thresholds, your state and what you supply: check the official GST portal and ask a CA |
| Tax-inclusive or not is yours to decide now | Changing your mind later breaks every autopay mandate you have already collected |
| A copied privacy policy is worse than a short honest one | It states untrue things about you; publish chapter 4's refund policy as well, which gateways generally require |
| Incorporating too early, or too late | Too early is an annual compliance bill with no revenue; too late blocks a gateway or a contract at the worst moment. The trigger list, not the calendar, decides |

**The DPDP Act is a design input, not a document.** India's Digital Personal Data Protection Act, 2023 most likely applies to you *today*, because it attaches to holding personal data rather than to having a company. Its rules were notified in stages, so **check the current status and the official text rather than any summary, including this one.**

| DPDP concept | What it means in your product |
|---|---|
| Data fiduciary, data principal | You decide why and how personal data is processed; your user is the person it is about |
| Notice and consent | Clear language on what you collect and why, specific and freely given, at the point of collection rather than buried |
| Purpose limitation | Use data for the stated purpose, not for a new idea you had later |
| Minimisation | Collect the minimum and delete what you no longer need |
| Correction and erasure | A mechanism users can use, which cannot be "edit the database by hand" |
| Breach notification | You must be able to say who was affected — M07's logging and M04's reconciliation, again |
| Children's data | Stricter treatment, and some of your users may be under eighteen |
| Third-party processing | An AI feature sends user data to your model provider, which your notice must disclose |

**The founder agreement is the highest-value document here and the one nobody writes.** One page in week two: who owns the code and the accounts, how decisions get made when you disagree, what happens if someone leaves. It costs an hour while everyone is friendly and cannot be written once they are not. Keep project money out of personal accounts too.

**Pay a professional at these moments, not "eventually"** — a CA for entity formation and filings, a lawyer for contracts and liability.

| Pay for an hour before | Why |
|---|---|
| You take real money from strangers | Entity, bank account, tax treatment and policies all land at once |
| You sign with an institution | Who signs, on behalf of what, with what liability |
| You split equity | The most expensive mistake on this list to undo |
| You raise outside money | The structure has to exist before the money does |
| You build on minors' data at scale | Stricter duties, and the ones most easily got wrong |
| Any tax question with a number in it | Exactly what an AI answers confidently and wrongly |

### Verify it

AI is useful here for mapping the landscape, and dangerous in one way: it states numbers, deadlines and fees with complete confidence, from a point in time you cannot see, for a jurisdiction it may have guessed.

**Red flag, the headline one: a specific threshold, fee, deadline or form number stated as fact.** Any figure about GST registration, filing dates, incorporation cost or penalties must come from the official source, read today, with the date written down. Ask which page to read, not what the number is. A model volunteering a number without that caveat is itself the warning.

**Red flag: advice that assumes a different country, or DPDP treated as GDPR with the names changed.** Delaware, a "DBA", an LLC, CCPA, a 1099 — American answers to Indian questions, from what the training data contains.

**Red flag: a recommendation to incorporate without asking why, or generated terms presented as ready to publish.** If nothing in the answer depends on whether you are raising money or signing contracts, it is a default, not advice.

**Red flag: a privacy notice that does not mention your model provider, or "we will figure out compliance later" where other people's money is involved.** The first is your most likely real DPDP exposure; the second is the opening story, two weeks before the fest was cancelled.

## Walkthrough
1. Run the four triggers against your own project today and write yes or no for each: real money from strangers, personal data held, something signed with an institution, ownership split between founders. Be honest about the second one.
2. Write your one-page founder agreement this week: who owns the repository, the domain and the accounts; how disagreements are decided; what happens if someone leaves; who owns the work if you stop. All three sign it, even informally.
3. Open a separate account or start a simple ledger for anything money-shaped, so project money and personal money never mix — even in test mode, where the only money is a domain and a hosting bill.
4. List every piece of personal data your product collects, field by field, and for each one write why you need it. Delete the fields with no answer.
5. Write a privacy notice from that list — short, specific and true — and explicitly name any third party you send user data to, including your model provider.
6. Move consent to the point of collection: the screen where the data is entered, in plain language, not a checkbox linked to a long document.
7. Build the mechanism for a correction and an erasure request, and test it on a real account of your own. If the answer is "edit the database by hand", write down that this is the gap.
8. Publish your refund and cancellation policy from chapter 4, and check what your payment gateway currently requires for live activation — from their own documentation, with the date.
9. Check the current status of the DPDP rules and the GST position for your kind of supply, from official sources only, and write down the date you checked and the page you read.
10. Write a list of the questions you would pay a CA or a lawyer one hour for, in priority order. Keep it in your repository; it is the agenda for the day you need it.

## Talk to AI
```prompt
I need a map of the legal and compliance decisions that apply to my student startup in India, and I need you to follow one rule strictly.

RULE: do NOT state any threshold, fee, filing deadline, penalty amount or form number as a fact. For every number I would need, tell me which official Indian source to read and what to look for there, and say plainly that anything you remember may be out of date. Do not give me answers based on United States or European law unless I ask.

My situation:
Product: [one sentence]
Payments: TEST MODE ONLY - we have taken no real money [or describe what real money has changed hands, from whom]
Personal data we collect, field by field: [list every field]
Third parties we send user data to: [model providers, analytics, email, anything]
Do any of our users appear to be under 18? [yes/no/unsure]
Have we signed anything with an institution? [yes/no, describe]
Founders: three students. Written agreement between us: [yes/no]
Are we planning to raise money or issue equity? [yes/no/unsure]
Our state: [state in India]

Do all of this, in order:

1. Run the four triggers against my situation - real money, personal data, institutional contracts, ownership split - and tell me which are live for me TODAY and which are not yet.

2. For each live trigger, list the decisions it opens as questions I have to answer, not as recommendations.

3. Entity: lay out sole proprietorship, partnership, LLP and private limited as a comparison, and then tell me which of my answers above would have to change before incorporating becomes necessary. Do not recommend an entity if nothing in my answers requires one.

4. For my data fields, tell me which ones I probably do not need, and for each remaining one what my notice has to say about why I hold it.

5. Draft a short, true privacy notice from my actual field list, naming every third party I told you about. Keep it under 400 words and do not include any practice I did not describe.

6. Tell me what the DPDP Act means for my design specifically - consent at collection, purpose limitation, minimisation, erasure, breach notification - and flag anything that depends on rules whose current status I must check myself. Point me at the official source.

7. Tell me what my payment gateway will likely require before live activation, and then tell me which of their own documentation pages to read rather than trusting your list.

8. On GST: tell me the questions I need answered about my kind of supply, which official source to check, and which of these questions genuinely needs a CA rather than a web page.

9. Draft the one-page founder agreement structure for three students: ownership, decisions, departure and dissolution. Leave the answers blank for us to fill in.

10. Finally, give me the prioritised list of questions I should pay a CA or a lawyer one hour for, and tell me which ones I should NOT attempt to answer with your help at all.
```

## Stuck? Ask AI
```prompt
Explain the difference between a sole proprietorship, an LLP and a private limited company in India in plain language, as a comparison of liability, cost, ongoing compliance and ability to raise money. Then ask me five questions about my own situation and tell me which answers would make each option necessary. Do not quote any fees or filing deadlines; tell me where to check them.
```

```prompt
I have a privacy notice that I copied from another product. Here it is: [paste it]. Go through it and mark every statement that is probably untrue about my product, which collects these fields: [list them] and sends data to these third parties: [list them]. Do not rewrite it yet - just show me how much of it is false.
```

```prompt
My team took ₹5,000 from a college society and the event was cancelled. We have no entity, no separate account, and nothing written down. Do not tell me what the law says. Tell me what to do in the next 48 hours, in order, and what document we should have written in week two that would have prevented this.
```

## Quiz

### Q1 · L3
Which trigger applies to almost every student team in this program, even though payments stay in test mode?
- (a) GST registration, because digital services are always registrable
- (b) Holding personal data, which brings DPDP obligations regardless of whether you have incorporated ✅
- (c) Payment gateway KYC, which applies in test mode too
- (d) Annual company filings
> Data obligations attach to processing personal data, not to having an entity. Test mode removes the money triggers and removes none of the data ones.

### Q2 · L4
An AI tells you the exact GST registration threshold for your service, the incorporation fee, and the filing deadline. What is the correct response?
- (a) Use the figures; tax rules are stable enough to rely on
- (b) Discard all three numbers and read the official source today, writing down the date ✅
- (c) Average them against a second AI's answer
- (d) Use them for planning and verify before filing
> Thresholds, fees and deadlines change, vary by state and by the nature of supply, and sit behind the model's cutoff. A confident number with no instruction to verify is itself the warning sign.

### Q3 · L4
Which document should a three-person student team write in week two, and almost never does?
- (a) Terms of service
- (b) A one-page founder agreement covering ownership, decisions and departure ✅
- (c) A privacy policy
- (d) A memorandum of association
> It costs an hour while everyone is friendly and is impossible to write once they are not. It is the document that prevents the argument about ₹5,000 and about who owns the repository.

### Q4 · L4
Your product sends user text to a model provider and your privacy notice does not mention it. Why does this matter?
- (a) It breaches the model provider's terms of service
- (b) The notice is inaccurate about where personal data goes, which is your most likely real exposure during this program ✅
- (c) It prevents you from registering for GST
- (d) It only matters if users are under eighteen
> A notice exists to be true. Sending personal data to a third party is exactly the kind of processing a data principal is entitled to be told about, and omitting it makes the whole notice unreliable.

### Q5 · L5
An AI recommends you incorporate a private limited company, mentions Delaware as an alternative, and cites CCPA for your privacy obligations. What is happening?
- (a) It is giving you globally applicable best practice
- (b) It is answering an Indian question with American defaults from its training data ✅
- (c) It has misunderstood your product
- (d) It is anticipating future international expansion
> This is the characteristic failure mode on this topic. Delaware, LLCs, DBAs, CCPA and 1099s are American answers, and they appear because of what the training data contains rather than because they apply to you.

## Go deeper
- [The Digital Personal Data Protection Act, 2023 — official text](https://www.meity.gov.in/data-protection-framework)
- [GST portal — registration, returns and the current rules](https://www.gst.gov.in/)
