---
id: m16-01-getting-real-users
module: m16-users-analytics-growth
slug: getting-real-users
title: "Getting Real Users: The Unscalable First Hundred"
order: 1
duration_min: 25
is_free: false
mission: true
volatility: low
last_reviewed: 2026-11-16
---

## Key learnings
- Do the arithmetic that tells you how many asks a hundred real users costs
- Recruit users one at a time through WhatsApp, classrooms and hostel floors
- Keep a recruitment log that tells you which channel produced users who stayed
- Separate friends who signed up to be kind from strangers who signed up to use it
- Close the loop on M01: compare measured behaviour against the baseline you recorded

## Lesson

### The pain

A team finishes M11 with a deployed product — sound architecture, fine p95, zero errors — and a dashboard showing eleven users. Nine are the three team members on three devices each; two are flatmates who signed up at 11 p.m. as a favour and never came back.

Every chapter after this one is arithmetic on user behaviour, and eleven users have no behaviour. Student products die here not from bugs but because nobody did the manual work of putting the thing in front of strangers.

### What M07 covered, and what is new here

M07 taught you to read the instruments — events, funnels, retention, churn, experiment design — and assumed users existed. Nothing in it tells you how to get the people.

### The idea

**The first hundred users are recruited one at a time, by hand, by you — and nothing about the process scales, which is exactly why it works.**

Search, content, ads and referral loops all amplify something, and there is nothing yet to amplify. What works is the individual ask. In India, at this size:

- **One-to-one WhatsApp beats everything.** The same text in a group of two hundred is read by nobody: everyone assumes it was meant for someone else.
- **Groups work only with the admin's permission, and only once.** A second post gets you removed and costs the channel permanently.
- **Go to a physical room.** Ninety seconds before a lecture, with an offer to set it up for anyone now, beats a hundred link clicks.
- **Install it with them, on their phone.** Concierge onboarding produces a user who reached value and shows you the step that confuses a stranger.
- **Do the work manually if you must.** A team member walking each pre-order to the counter buys real behaviour while the automation gets built.
- **Shared devices and prepaid data shape all of it.** A hostel phone passed between three students puts three people under one account, breaking per-user identity for every number in this module; nobody on a prepaid pack downloads 40 MB on a classmate's word.

### The analogy

A new stall opens inside the campus canteen and nobody knows it exists. The owner does not buy a hoarding; they stand at the front handing samples to individual students. Thirty plates on day one, three of which produce a student who returns on Thursday.

**Where the analogy breaks:** the stall has a doorway and a stream of students passing four times a day. Nobody walks past a URL. Standing at the front means going where the stream already is — the class group, the hostel floor, the lab queue.

### Picture it

```
  THE ASK LADDER, one week, one team of three

  asks sent         300  ████████████████████████  100%
  messages read     214  █████████████████         71%
  link opened        96  ████████                  32%
  signed up          41  ███                       14%  ◀ −18
  did the thing      17  █                          6%  ◀ −8
  came back once      6  ▌                          2%
```

Three hundred individual asks produce roughly seventeen people who did the thing your product is for and six who came back — so a hundred users who actually use the product costs something like fifteen hundred asks. Every team assumes its own rate will be better. It will not be.

Those rows exist only if you keep a recruitment log: one row per ask, with who, channel, date, opened, signed up, reached value. It separates channels that produced users who stayed from channels that only produced signups.

### Where it lives

M01's chapter three is upstream: the persona you wrote from evidence and the baseline you recorded — "skipped lunch twice last week". Recruit people who do not match it and every number here describes somebody you were never building for. Downstream, your channel column becomes M17's acquisition channel mix.

### Words to own

| Term | Plain meaning | Example |
|---|---|---|
| Ask | One individual request to one named person | A WhatsApp message to Rahul |
| Unscalable acquisition | Getting users by hand, one at a time | Classroom visits |
| Channel | A route through which users arrive | Class group, hostel floor, DM |
| Warm ask | Someone who knows you or your team | A classmate |
| Cold ask | A stranger with no connection to you | A student in another department |
| Broadcast blindness | Nobody acts on a message sent to everyone | A post in a 200-person group |
| Concierge onboarding | Setting the product up with the user, beside them | Installing on their phone |
| Recruitment log | One row per ask, with the outcome | A spreadsheet of 300 rows |
| Friend bias | Signups given as a favour, not from need | Flatmate who never returned |
| Reached value | Did the thing your product exists to do | Placed a first order |

### Tradeoffs and gotchas

**Friends poison your numbers more than they help them.** A favour signup never intended to return still drags your retention curve down. Mark friends as a separate cohort from day one.

**Do not spend money on ads at this size.** A thousand rupees buys traffic you cannot interpret from people who do not match your persona.

**An install is not a user.** An install is a file; a signup is a form submission. Only "did the thing" counts.

**Hundred is a target, not a magic number.** Below a hundred actives, percentages are stories about individuals. Three hundred is where funnels behave; a thousand is where experiments become possible.

**The social cost is real and temporary.** Four in five will say nothing. The teams that get through this module kept a log anyway.

### Verify it

AI is good at drafting your messages and terrible at telling you where your users are.

1. **Does the plan assume traffic you do not have?** Red flag: any step whose first requirement is "drive traffic to".
2. **Does the plan assume a budget?** Say zero rupees in the brief and check every step survives it.
3. **Is the message for one person or for everyone?** "Hi! We've built an exciting new app" is a broadcast; name the person and the problem you heard in M01.
4. **Would your persona recognise themselves?** Ask AI who it thinks the message is aimed at, without telling it.

Red flags in your own log: signups but no reached-value column; every row the same channel; no dates; every name a friend; fewer than fifty rows, meaning it was never run.

## Walkthrough
1. Open your analytics or your database and write down three numbers for today: total signups, how many did the thing your product is for at least once, and how many of those are not on your team.
2. Re-read your M01 persona and the baseline behaviour you recorded. Write one sentence naming the person you are about to go and find, in their words, not yours.
3. List every channel you can reach this week by name — not "social media" but "our class group of 62", "Mechanical second-year group, admin is Sana", "the 3 p.m. lab queue", "my hostel floor, 30 rooms".
4. Beside each channel write how many people it contains and whether you have permission to post. Where you do not, write who you must ask first.
5. Build the recruitment log: one row per planned ask, with columns for name, channel, date, warm or cold, opened, signed up, reached value, and a notes field for what they said.
6. Draft one message per channel, each addressed to a single person and each naming the specific problem from your persona rather than your product's features. Have AI critique them for broadcast language, then rewrite.
7. Send twenty-five asks on day one, by hand, individually. Fill in the log as replies arrive. Do not batch-send and do not use a group for this step.
8. Pick five people and do concierge onboarding: sit beside them, install it on their phone, say nothing while they use it, and write down every place they hesitated.
9. Walk into one physical room and ask for ninety seconds before a lecture or a meeting. Count installs that happened in the room.
10. At the end of the week, compute the ask ladder from your log: asks, opens, signups, reached value, returned once. Then compute the same ladder per channel and name the single best channel with the number that proves it.
11. Write one sentence comparing your friend cohort's return rate against your stranger cohort's. If they differ, every number later in this module must keep the two apart.

## Talk to AI
```prompt
CONTEXT:      I am a student in a team of three in India. I direct AI and
              verify its output; I do not write code. Our product is live
              at a public URL: [describe it in two sentences]. We have
              [number] total signups, of which [number] are our own team
              and friends, and [number] people have ever done the main
              action the product exists for. Our users are students on
              budget Android phones, often on prepaid data, sometimes
              sharing a phone with a sibling or roommate. WhatsApp is how
              they communicate. Our budget for acquiring users is zero
              rupees. Our persona from earlier research is: [paste it].
              The baseline behaviour we recorded before building was:
              [paste it, e.g. "skipped lunch twice last week"].

GOAL:         A week-one plan to get 30 real users who are not friends of
              the team and who match that persona, using only manual,
              unscalable, zero-rupee channels.

REQUIREMENTS: 1. Challenge my persona first: based on what I described,
                 tell me where I am most likely to find 200 people who
                 match it, as specific places and groups, not categories.
              2. Give me a ranked list of channels I can reach by hand
                 this week, with the realistic number of people in each
                 and the permission I need before using it.
              3. For my single best channel, draft three versions of a
                 one-to-one message: one too long, one too salesy, and one
                 you would actually send. Explain what is wrong with the
                 first two.
              4. Give me the columns my recruitment log needs so that in
                 four weeks I can tell which channel produced users who
                 stayed rather than users who signed up.
              5. Estimate, using plausible rates, how many individual asks
                 I need to send to reach 30 users who complete the main
                 action. Show the rate at each step and say which rate you
                 are least confident about.
              6. Tell me what concierge onboarding should look like for my
                 specific product: what I do, what I say, and what I must
                 not say while sitting beside the user.
              7. Name three things about Indian student users - shared
                 phones, prepaid data cost, patchy connectivity - that
                 should change my plan, and how.

CONSTRAINTS:  Zero rupees. No paid ads, no influencers, no tools with a
              subscription. Nothing that requires existing traffic, an
              email list, search rankings or an audience. Assume I will
              personally send every message by hand. Plain English.

NON-GOALS:    Do not design my event taxonomy, funnels or experiments -
              later chapters cover those. Do not propose SEO, content
              marketing, launch-site posts, Product Hunt, Reddit or
              referral programmes; I have no traffic to amplify yet. Do
              not write code and do not modify any files.

ACCEPTANCE:   - Every channel you name is a specific group, room or list I
                could reach today, with a number of people attached
              - No step in the plan requires money or existing traffic
              - The message you would actually send names one person and
                one problem, and does not list features
              - I can state how many asks 30 real users will cost me
              - My log has a column that distinguishes signed up from
                reached value, and friends from strangers

ASK:          First ask me up to five questions about my product, my
              persona and the groups I can actually reach. Then give me
              requirements 1 and 2 only, and stop before drafting any
              messages.
```

## Mission

**Objective.** Thirty real users who are not friends of the team, recruited by hand in one week, with a recruitment log that shows the ask ladder per channel and separates friends from strangers.

**Steps.**
1. **Frame** — write today's three honest numbers: signups, people who did the main action, and how many of those are outside your team. This is your starting line and it goes in the submission.
2. **Target** — restate your M01 persona in one sentence and list the specific groups and rooms where two hundred such people are reachable this week.
3. **Brief** — have AI critique your channel list and your draft messages for broadcast language and for hidden assumptions about budget or existing traffic. Rewrite after the critique.
4. **Log** — build the recruitment log with a row per planned ask, including a warm-or-cold column and a reached-value column.
5. **Ask** — send at least a hundred individual asks across the week, by hand. Post in at most two groups, each with the admin's permission, once each.
6. **Sit beside people** — do concierge onboarding with at least five users on their own phones. Write down every hesitation without helping.
7. **Go to a room** — make one in-person ask to a physical group of at least fifteen people. Record installs that happened in the room.
8. **Count** — compute the ask ladder overall and per channel. Name your best channel with the number that proves it.
9. **Verify** — run the four checks from "Verify it" against your plan and your log, and compare your friend cohort's return rate against your stranger cohort's.

**Upload.**
- Your recruitment log, at least a hundred rows, names initialised for privacy → file or screenshot
- Your ask ladder, overall and per channel, with counts and percentages → screenshot or text
- The three message drafts and the one you sent → text
- Notes from five concierge onboardings: every place a user hesitated → file or screenshot
- Your AI conversation critiquing the channel list and the messages → share link in `ai_chat_url`
- Proof the product is live and in use → deployed URL in `demo_url`
- Reflection, 150 words max → the channel that surprised you, and one thing the AI suggested that was impossible at your size

**Self-check.** Tick every box before you submit.
- [ ] At least thirty users outside my team and friend group have done the main action
- [ ] My log has at least a hundred rows, each one an individual ask
- [ ] My log distinguishes signed up from reached value, and warm from cold
- [ ] I can state my ask-to-reached-value rate as a number
- [ ] I computed the ask ladder separately for at least three channels
- [ ] I sat beside at least five users while they used it and wrote down their hesitations
- [ ] I made one in-person ask to a room of fifteen or more people
- [ ] I posted in no more than two groups, each with the admin's permission, once each
- [ ] Every recruited user plausibly matches my M01 persona, and I can say why
- [ ] I spent zero rupees
- [ ] My reflection names something the AI proposed that my traffic could not support

**Stretch.** Find ten users from outside your college entirely — a different campus, a coaching centre, a workplace. Compare their ask ladder against your campus numbers and write 100 words on what the gap tells you about how much of your current growth is your own social network rather than your product.

## Stuck? Ask AI
```prompt
I asked 120 people to try my product and 4 signed up. Here is the exact message I sent: [paste it]. Here is who I sent it to: [describe them]. Do not reassure me. Tell me whether the problem is most likely the message, the audience, the ask being too large, or the product itself - and tell me one specific test I could run this week that would distinguish between those four causes.
```

```prompt
Explain why one-to-one messages outperform group broadcasts when a product has almost no users, using an example from outside software. Then give me eight ways of asking someone to try a product, where some are individual asks and some are disguised broadcasts, and ask me to sort them before telling me the answers.
```

```prompt
My team keeps building features instead of going out and asking people to use our product, because asking feels humiliating and building feels productive. Explain what this behaviour costs us in measurable terms over four weeks, then give me the smallest possible daily action - something under fifteen minutes - that would break the pattern, and tell me how I would know in a week whether it worked.
```

## Quiz

### Q1 · L3
Your product has eleven users, nine of whom are your own team on multiple devices. AI gives you a growth plan whose first three steps are SEO, a content calendar and a referral programme. What is wrong with it?
- (a) Nothing; those are the standard growth channels
- (b) All three amplify existing traffic, and you have none to amplify ✅
- (c) They are too expensive for a student team
- (d) They work only for products outside India
> Search, content and referral are multipliers. A multiplier applied to almost zero is almost zero. At this size the only thing that works is the individual ask, which is why the chapter's arithmetic is in asks rather than impressions.

### Q2 · L4
You send the same text to one person individually and to a class group of two hundred. The individual message gets acted on far more often. What is the mechanism?
- (a) Group messages are delivered less reliably
- (b) In a large group everyone assumes the message is meant for someone else ✅
- (c) Individual messages arrive sooner
- (d) Group admins usually delete product messages
> This is broadcast blindness. Responsibility dissolves across a large audience, so nobody acts. The fix is not a better broadcast but an ask addressed to one named person with a reason you thought of them.

### Q3 · L4
Your signup count is 60 and your retention curve looks terrible. Twenty-eight of the sixty are friends who signed up as a favour. What should you do?
- (a) Remove the friends from the database so the numbers look right
- (b) Keep friends in a separately marked cohort and read the two curves apart ✅
- (c) Ignore it; sixty is sixty and the curve is the curve
- (d) Treat the friends as your most engaged users since they signed up fastest
> Favour signups never intended to return, so mixing them in mismeasures your product. Marking the cohort from day one is cheap; deleting real rows destroys evidence, and reading one blended curve tells you nothing about either group.

### Q4 · L3
Which of these counts as a user for the purposes of every later chapter in this module?
- (a) Someone who installed the app
- (b) Someone who submitted the signup form
- (c) Someone who did the action your product exists to do ✅
- (d) Someone who clicked the link you sent
> An install is a file and a signup is a form submission. Funnels, retention and churn only mean something when measured against value received, which M07 called activation and this module measures for real.

### Q5 · L5
Your ask ladder shows 300 asks, 41 signups and 17 who reached value. A teammate argues the next step is to send three thousand asks. What is the stronger objection?
- (a) Three thousand asks is more work than a week allows
- (b) The per-channel ladder should direct the next batch, since channels differ widely ✅
- (c) The rates will improve on their own as the product matures
- (d) Volume cannot help; only paid acquisition scales from here
> The overall ladder hides the useful information. One channel may convert at four times another, and the log exists precisely so the next hundred asks go where the last hundred worked rather than being spread evenly again.

## Go deeper
- [Paul Graham: Do Things That Don't Scale](https://paulgraham.com/ds.html)
- [First Round Review: how early-stage teams find their first users](https://review.firstround.com/)
