---
id: m07-09-ab-tests-privacy-consent
module: m07-security-observability-analytics
slug: ab-tests-privacy-consent
title: "A/B Tests, Significance, Privacy and Consent"
order: 9
duration_min: 25
is_free: false
mission: false
volatility: low
last_reviewed: 2026-11-16
---

## Key learnings
- Decide, before running anything, whether you have the traffic for an A/B test
- Say what statistical significance claims and what it does not
- Name the three design choices that make an experiment result trustworthy
- Apply the DPDP Act's consent requirements to your own analytics
- Spot an analytics setup that collects personal data it has no right to

## Lesson

### The pain

A team changes their Order button from grey to orange. Over two days, 23 of 190 visitors order with orange against 18 of 182 with grey — 12.1% against 9.9%, a 22% relative improvement. They ship orange, put the number on a slide, and move on.

Nothing changes. The next month's orders look exactly like the last month's. The difference they measured was five people, which is what two coin flips of 190 tosses differ by on an ordinary Tuesday. They measured noise, named it a result, and built on it.

Meanwhile their analytics snippet had been sending the whole signup form to a vendor's server — phone numbers included — with no notice, no consent, and no idea which country the data sat in. That is the more expensive of the two mistakes, and it was the one nobody looked at.

### The idea

**An experiment is a comparison you designed before you looked at the data, and significance is a statement about how surprising your difference would be if nothing were really happening.**

Both halves matter. Decide the metric, the groups, the duration and the threshold in advance, and the result means something. Decide any of them after seeing the numbers and you have found a pattern in noise, which humans do effortlessly and reliably.

### The analogy

The canteen wants to know whether a new samosa recipe sells better.

The wrong experiment: new recipe on Monday, old recipe on Tuesday. Monday was a holiday, Tuesday had rain, and there was a cricket match. The difference measures the week, not the samosa.

The right experiment: both recipes, same day, two counters, students sent to one or the other by a coin flip — and the same student always goes to the same counter, so nobody compares the two and gets confused. Decide in advance that you are measuring plates per hundred students, run two full weeks so every weekday appears in both groups, and look once at the end.

**Where the analogy breaks:** in the canteen you can watch faces, and you can ask the student who put the samosa down why. In software you get a number and nothing else — no expression, no reason, nobody to ask. That absence is why the design has to be right before you start: there is no way to interrogate a disappointing result afterwards.

### Picture it

```
  1,000 students arrive        assignment is sticky:
            │                  the same student always
      ┌─────┴─────┐            gets the same variant
      ▼           ▼
  ┌─────────┐ ┌─────────┐
  │    A    │ │    B    │
  │   500   │ │   500   │
  │ control │ │ variant │
  └────┬────┘ └────┬────┘
       │           │
   52 orders   61 orders
    10.4%       12.2%
       └─────┬─────┘
             ▼
   is +1.8 points real, or luck?
   at this baseline you need
   roughly 3,800 per group
```

A thousand students split evenly produce a difference of nine orders, which looks like a result and is well inside what chance produces — detecting a move from 10% to 12% reliably needs something like 3,800 people in each group.

### What significance actually claims

The number you will be shown is a **p-value**, and almost everybody misreads it.

A p-value of 0.03 means: *if there were truly no difference between A and B, a gap this large or larger would turn up about 3% of the time by chance alone.* That is all. The usual threshold is 0.05, often displayed as "95% confidence".

It does **not** mean there is a 97% chance your change works. It does not mean the effect is large, or that it will persist. And a p-value above 0.05 does not mean the variants are the same — far more often it means your sample was too small to tell.

Three terms you need alongside it:

**Minimum detectable effect.** How small a difference you want to be able to notice. Smaller effects need dramatically more people — halving the effect roughly quadruples the sample. Decide this first, because it determines whether the experiment is possible at all.

**Sample size.** Computed from your current rate and that minimum effect, *before* you start. If the answer is more users than you will have this term, you cannot run this test, and knowing that in advance saves you from running it badly.

**Peeking.** Checking significance repeatedly and stopping when it crosses the line. This is how honest teams get false results: look twenty times and you cross 0.05 by chance eventually, then declare victory. Fix the duration in advance and look at the end.

**So most student teams should not run A/B tests.** That is the honest conclusion, and not a failure. With a few hundred users the alternative is better: watch chapter eight's funnel, sit next to five people using the product, change the thing that visibly confused all five, and see whether the drop-off moves. A/B testing becomes the right tool when traffic is large enough that you can no longer see individuals — M16's territory, with real users.

### Privacy and consent: the part with legal weight

An experiment runs on people, and so does analytics. Two frameworks govern this for an Indian student product.

**India's Digital Personal Data Protection Act, 2023** — the DPDP Act. You are a **data fiduciary**; your user is a **data principal**. The obligations that bite a small product:

- **Notice and consent.** Before processing personal data you must give a plain-language notice saying what you collect and why, and obtain consent that is free, specific, informed and given by a clear affirmative action. A pre-ticked box is not consent. Silence is not consent.
- **Purpose limitation.** Use the data for the stated purpose, not a new one you thought of later.
- **Data minimisation.** Collect what the purpose needs — not the whole form because the form was there.
- **Rights.** The data principal can ask what you hold, have it corrected, and have it erased. If you cannot delete one person's data, you cannot meet that obligation.
- **Children.** Processing data of anyone under 18 requires verifiable parental consent, and behavioural tracking aimed at children is prohibited. A college app will have first-year users under 18, so this is not theoretical.

**GDPR** applies the moment you have users in the EU, which a public URL may acquire accidentally. The shape is similar — a lawful basis, minimisation, a defined retention period, rights of access and erasure — and analytics cookies there require consent before they load.

What this means for your setup, in four lines. No personal data in events: user id, never phone number or email. Do not load an analytics script before consent, which is what a generated cookie banner almost always gets wrong. Anonymise or truncate IP addresses if your tool offers it. Set a retention period, because "forever" is not a period.

### Where it lives

Chapter eight gave you the events; this chapter is what you may do with them and who you must ask first. M04 is where the personal data lives and where an erasure request has to be answerable. Chapter five's never-log list is the same principle applied to logs — and logs are personal data too.

The practicum is M16 at the Company tier: a real experiment programme, and privacy, consent, DPDP and GDPR with a real user base. This chapter is the vocabulary and the judgement; that module is the practice. M11's going-live checkpoint is where a consent notice stops being optional, because that is when strangers start arriving.

### Words to own

| Term | Plain meaning | Example |
|---|---|---|
| Control / variant | The existing version, and the change | Grey button, orange button |
| Sticky assignment | The same person always sees the same variant | Hashed on user id |
| Primary metric | The one number decided in advance | Orders per visitor |
| Guardrail metric | A number that must not get worse | Error rate, refunds |
| p-value | How often chance alone would produce this gap | 0.03 |
| Minimum detectable effect | The smallest difference you could notice | 2 percentage points |
| Peeking | Checking repeatedly and stopping when it looks good | The commonest false result |
| Data principal | The person the data is about | Your user |
| Data fiduciary | Whoever decides how the data is used | You |
| Data minimisation | Collect only what the purpose needs | User id, not phone number |

### Tradeoffs and gotchas

**Do not A/B test what you can decide by judgement.** Button wording, spacing, which of two icons. The cost of a test — traffic, weeks, a fork in your product — is only worth paying for decisions that are expensive to reverse or genuinely contested. Taste is cheaper than statistics.

**Novelty fades.** A new thing gets attention because it is new. Run long enough that first-week curiosity has passed, and be suspicious of a lift that lives entirely in days one and two.

**Twenty metrics means one false winner.** Measure twenty numbers at a 5% threshold and one will look significant by chance. Choose one primary metric in advance, keep two or three guardrails, and treat the rest as questions for next time.

**Reassignment destroys an experiment silently.** If the variant is chosen freshly on every page load, a user sees orange, then grey, then orange. Their behaviour belongs to no group, and nothing warns you. Assignment must come from something stable, like a hash of the user id.

**Consent refusals shrink your sample and bias it.** People who refuse tracking are not a random slice. Accept it and read your numbers as directional, rather than looking for a way around asking.

**Experimenting on people has an ethical floor, not only a legal one.** Do not test things that could harm someone: different prices for different students, a variant that hides a safety warning, anything involving money the user cannot see. If you would not describe the test to the person in it, do not run it.

### Verify it

Six checks. The first three are about the experiment, the last three are about whether you were allowed to run it.

1. **Ask what the primary metric is and when it was chosen.** If the answer came after the data, the result is a story. Look for a written hypothesis with a date on it.
2. **Check that assignment is sticky, by testing it.** Reload the page ten times and confirm you see the same variant every time. **Assignment recomputed on every render** is the classic generated-code flaw and it quietly invalidates everything.
3. **Ask for the sample size calculation and the planned end date.** No calculation means nobody knows whether this test could have detected anything. No end date means peeking is inevitable.
4. **Open the network tab and watch what loads before you click anything on the consent banner.** If a tracker fires before consent, the banner is decoration. This takes thirty seconds and is the check almost nobody runs.
5. **Inspect the actual payload of one analytics event.** Read every field. **A phone number, email or full name inside an event property** is the red flag here, and it is usually there because a generated snippet sent the whole form object rather than the fields you chose.
6. **Try to answer an erasure request.** Pick a user id and work out, concretely, how you would delete that person from your database, your logs and your analytics tool. If you cannot, you have an obligation you cannot meet — and finding that out now is far cheaper than finding out when someone asks.

Red flags: significance "achieved" after daily checking; a result based on a difference of single-digit people; a test with no primary metric named in advance; variants reassigned per request; analytics loading before consent; personal data in event properties; and a retention policy of "we keep everything".

## Walkthrough
1. Take a change your team is currently arguing about and write it as a hypothesis in one sentence: "Changing X will increase Y because Z."
2. Name the single primary metric for that hypothesis, and two guardrail metrics that must not get worse. Write all three down before going any further.
3. Find your current rate for that primary metric from chapter eight's funnel. This is your baseline, and without it nothing else can be computed.
4. Decide the smallest difference that would actually change your decision. Not the smallest you could imagine — the smallest you would act on.
5. Have AI compute the sample size per group for that baseline and that effect, and ask it to show the inputs it used. Compare the answer with how many users you will realistically have this term.
6. If the answer is more users than you will have, stop and write one sentence saying so. Then design the alternative: which five people you will watch, and which funnel step you will check afterwards.
7. Either way, now audit your analytics. Open your deployed app in a fresh browser, open the network tab, and record every request that fires before you interact with any consent banner.
8. Click into one analytics event and read every field in its payload. Write down anything that identifies a person.
9. Find your consent notice. If there is none, draft one in plain English: what you collect, why, how long you keep it, and how someone asks for deletion. Four sentences is enough to start.
10. Pick one real user id and write the exact steps you would take to erase that person from your database, your logs and your analytics tool. Note every place where you cannot.
11. Put the hypothesis, the sample-size verdict and the erasure procedure in your repository, and note in your decision log what you chose not to test and why.

## Talk to AI
```prompt
CONTEXT:      I am a student in a team of three. I direct AI and verify its
              output; I do not write code. Our product: [describe it in two
              sentences]. We have [number] users, roughly [number] visits a
              week. Our users are students in India; some may be under 18,
              and our URL is public so we may get EU visitors. We collect
              these personal fields: [list them]. Our current conversion
              rate on [the action we care about] is [number]%. We use
              [analytics tool, or "none yet"]. India's DPDP Act applies to
              us; GDPR may apply to EU visitors.

GOAL:         An honest answer to whether we can run a meaningful A/B test
              at our size, and a privacy and consent setup for our
              analytics that would survive being read by a regulator.

REQUIREMENTS: 1. Given my baseline rate, compute the sample size per group
                 needed to detect a 2-point and a 5-point absolute change.
                 Show your inputs and assumptions. Then tell me plainly
                 whether my traffic supports either.
              2. If it does not, design the alternative: how many people I
                 should watch using the product, what I should look for, and
                 which single funnel number I should check afterwards.
              3. Explain a p-value in one paragraph, then give me three
                 sentences people commonly say about significance that are
                 wrong, and correct each one.
              4. Explain sticky assignment, how I can test it myself in a
                 browser in under a minute, and what exactly goes wrong if
                 the variant is recomputed on every page load.
              5. For my field list, tell me which fields may appear in an
                 analytics event, which must never, and what to send
                 instead.
              6. Draft a plain-English consent notice of at most six
                 sentences covering what I collect, why, how long I keep it,
                 and how someone requests deletion. Tell me where it must
                 appear and what must not load before consent is given.
              7. Tell me what the DPDP Act requires specifically because
                 some of my users may be under 18.
              8. Give me the step-by-step procedure for honouring one
                 erasure request across my database, my logs and my
                 analytics tool, and name the places where this is usually
                 impossible.

CONSTRAINTS:  Free tiers only. Explain every term on first use. Plain
              English, no legal jargon without a translation. Where the law
              is genuinely uncertain or has changed, say so and point me at
              the primary source rather than guessing. You are not my
              lawyer and I am not treating this as legal advice.

NON-GOALS:    Do not design the event taxonomy; I have one. Do not write
              code. Do not recommend a consent-management vendor. Do not
              tell me we are "compliant" — give me checks, not verdicts.

ACCEPTANCE:   - I know whether an A/B test is possible at my size, with the
                arithmetic shown
              - I can correct three wrong statements about significance
              - I can test sticky assignment myself in under a minute
              - My consent notice is six sentences or fewer and a
                first-year student could understand it
              - I can state what must not load before consent
              - I have a written erasure procedure, including its gaps

ASK:          First ask me up to five questions whose answers would change
              your advice, including anything about my users' ages and
              locations. Then answer requirement 1 only, with the
              arithmetic visible, and stop.
```

## Stuck? Ask AI
```prompt
Explain a p-value to me as if I have never studied statistics, using a canteen taste test with two samosa recipes. Then give me five sentences that people commonly say about significance, where some are correct and some are wrong, and ask me to sort them before you tell me the answers.
```

```prompt
Explain what "peeking" means in an A/B test and why it produces false results. Then walk me through a concrete example: show me what happens to the apparent significance if I check the numbers every day for twenty days on two groups that are genuinely identical.
```

```prompt
Explain India's DPDP Act to me in plain English as it applies to a student product with a few hundred users: what notice means, what counts as valid consent, what purpose limitation stops me doing, and what rights my users have. Then tell me the three things a student team most commonly gets wrong, and how I would check each one in my own app today.
```

## Quiz

### Q1 · L4
Your test shows a p-value of 0.03. Which statement is correct?
- (a) There is a 97% chance the change works
- (b) If there were truly no difference, a gap this large would occur about 3% of the time ✅
- (c) The change improves conversion by 3%
- (d) The result will hold for at least three months
> A p-value describes how surprising your data would be under the assumption of no effect. It says nothing about the size of the effect, nothing about whether it persists, and it is not the probability that you are right.

### Q2 · L5
With 190 visitors per group and a 10% baseline, your variant shows a 2-point lift. What should you conclude?
- (a) Ship it; a 20% relative improvement is substantial
- (b) Nothing — at this size the sample cannot distinguish a 2-point lift from chance ✅
- (c) That the control is broken and should be investigated
- (d) Run it another two days and ship if it holds
> Detecting a 2-point move at a 10% baseline needs thousands per group. Option (d) is worse than (a), because stopping as soon as it looks good is exactly the peeking that manufactures false winners.

### Q3 · L4
AI builds your A/B test so the variant is chosen randomly on each page load. What breaks?
- (a) Nothing, as long as the split stays near 50/50
- (b) Users move between groups, so neither group measures a consistent experience ✅
- (c) The p-value is computed on too small a sample
- (d) The control group will be systematically larger
> Assignment must be sticky — derived from a stable value like a hash of the user id. Reassignment blends the groups, invalidates the comparison, and produces no warning of any kind.

### Q4 · L4
Your consent banner appears on load, and the analytics script loads at the same time. Why is this worse than having no banner?
- (a) It slows the page down for no benefit
- (b) The data is collected anyway, plus a false claim of having asked ✅
- (c) Banners are not required under the DPDP Act
- (d) It reduces the sample size of your experiments
> Consent must precede processing and must be a clear affirmative action. A banner that fires trackers before the click collects the data without permission and documents that you knew permission was needed.

### Q5 · L3
Which field is acceptable inside an analytics event property?
- (a) The user's phone number, for matching across devices
- (b) An internal user id ✅
- (c) The user's email address, hashed for privacy
- (d) The full signup form, so nothing is lost
> Minimisation means sending the least that answers your question, and an internal id does that while keeping the mapping inside your own database. A hashed email is still personal data, and sending the whole form is how phone numbers end up at a vendor.

## Go deeper
- [Evan Miller: How Not To Run An A/B Test](https://www.evanmiller.org/how-not-to-run-an-ab-test.html)
- [PRS India: Digital Personal Data Protection Act, 2023](https://prsindia.org/billtrack/digital-personal-data-protection-bill-2023)
