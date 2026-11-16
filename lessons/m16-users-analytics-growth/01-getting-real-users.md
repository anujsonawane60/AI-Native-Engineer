---
id: m16-01-getting-real-users
module: m16-users-analytics-growth
slug: getting-real-users
title: "Getting Real Users: The Unscalable First Hundred"
order: 1
duration_min: 35
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

A team finishes M11 with a working, deployed product. The architecture is sound, the p95 is fine, the error rate is zero. They open their analytics dashboard in week one of this module and it shows eleven users. Nine are the three team members on three devices each. Two are flatmates who signed up at 11 p.m. because they were asked to and never opened it again.

Every chapter that follows this one — funnels, retention, cohorts, experiments — is arithmetic on user behaviour. With eleven users there is no behaviour. A funnel with eleven people at the top is a sentence about four of your friends. A retention curve on eleven people is a straight line with no information in it. The entire rest of the module is blocked, and no amount of better instrumentation unblocks it.

This is where most student products quietly die. Not from a bug and not from a bad idea, but because nobody did the uncomfortable, manual, un-engineerable work of putting the product in front of strangers. The team keeps building features instead, because building features is familiar and asking people is not.

### What M07 covered, and what is new here

M07's chapter eight taught you to read the instruments: what an event is, what a funnel measures, how a retention curve is shaped, what churn means. Chapter nine taught experiment design and consent. Both assumed users existed.

Nothing in M07 tells you how to get the people. That is this chapter, and it is first in the module because everything after it is useless without it. M01 taught you to talk to five users before building. You now need a hundred who actually use it.

### The idea

**The first hundred users are recruited one at a time, by hand, by you — and nothing about the process scales, which is exactly why it works.**

Growth channels that scale — search, content, paid ads, referral loops — all need something to amplify. At eleven users there is nothing to amplify. What works instead is the individual ask: one named person, one message written for them, often one conversation where you sit beside them while they sign up. It is slow, it feels like begging, and it is the only thing that works at this size.

### The analogy

A new stall opens inside the campus canteen. Nobody knows it exists.

The person running it does not buy a hoarding. They stand at the front with a tray of free samples and hand them to individual students walking past, one plate at a time, and they watch each face as the student tastes it. Thirty plates on day one, most of which produce nothing, three of which produce a student who comes back on Thursday. By week three the stall has forty regulars, every one of whom was handed a sample by a human being.

**Where the analogy breaks:** the canteen stall has a doorway and a captive stream of students who pass it four times a day. Your product has no doorway. Nobody walks past a URL. So the equivalent of standing at the front is going to where the stream already is — the class WhatsApp group, the hostel floor, the 3 p.m. lab queue, the society meeting — and the equivalent of handing over a plate is a message written to one named person, not a broadcast to two hundred.

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

Three hundred individual asks produce roughly seventeen people who did the thing your product is for, and six who came back — which means a hundred users who actually use the product costs something like fifteen hundred asks.

That arithmetic is the whole chapter. The first reaction of every team is that their rate will be better; in week one it will not be. It improves when the asks get more targeted, which only happens after you have logged enough of them to see which channel worked.

### Where the users actually come from, in India, at this size

**One-to-one WhatsApp beats everything.** A message addressed to one person, naming why you thought of them, gets read and sometimes acted on. The same text pasted into a group of two hundred gets read by nobody, because everyone in a large group assumes the message is for someone else. Broadcast blindness is real and it is severe.

**Groups work only with the admin's permission and only once.** Ask the class or society group admin first, post once, never twice. A second post gets you removed and costs you the channel permanently. One well-timed post in a hostel group of ninety, with the admin's blessing, is worth more than thirty uninvited posts.

**Go to a physical room.** For a Nashik-offline cohort this is the strongest channel available and the one teams skip because it is frightening. Walk into a classroom before a lecture, ask the teacher for ninety seconds, show the product on your own phone, and offer to set it up for anyone who wants it now. Ten installs you watched happen are worth a hundred link clicks.

**Install it with them, on their phone.** Sit beside the person and watch where they hesitate. This is concierge onboarding, and it does two jobs: it produces a user who actually reached value, and it shows you the exact step that confuses a stranger. Chapter five's replays and chapter six's activation work both start here, with your own eyes.

**Do the work manually if you must.** If the product cannot yet do something, do it yourself behind the scenes for the first twenty users. A canteen pre-order app with no counter integration can have a team member walk each order over. It does not scale and it is not cheating; it buys you real behaviour to measure while the automation gets built.

**Shared devices and prepaid data shape all of this.** A hostel phone passed between three students produces three people's behaviour under one account, which breaks per-user identity for every number in this module. A student on a prepaid pack will not download a 40 MB app on mobile data to try something a classmate mentioned.

### The recruitment log

One row per ask: who, which channel, which date, what you said, whether they opened it, whether they signed up, whether they did the thing your product is for. A spreadsheet is correct here.

The log does three jobs. It converts a vague feeling of effort into a number you can divide. It tells you which channel produced users who stayed rather than users who signed up, which are very different lists. And it is the only honest record of whether your hundred users are strangers or your friend group wearing different names.

### Where it lives

M01's chapter three is directly upstream. There you ran five conversations, wrote a persona from evidence, and recorded a baseline — "skipped lunch twice last week", "pays a senior ₹20 on Thursdays". The people you recruit here must match that persona, or every number in this module describes somebody you were never building for. That baseline is what you will finally measure against in chapter four.

Downstream, this chapter feeds M17's unit economics: the cost per acquired user starts as your own hours, which is still a cost, and the channel column of your log becomes the first version of an acquisition channel mix.

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

**Friends poison your numbers more than they help them.** A friend who signs up to be supportive and never returns adds one to your signup count and drags your retention curve down, and you will spend chapter four wondering why. Keep friends in a separate marked cohort in your log from day one. It is far cheaper than untangling it in month three.

**Do not spend money on ads at this size.** A thousand rupees of ads at eleven users buys traffic you cannot interpret, from people who do not match your persona. Paid acquisition is chapter eight's topic and M17's arithmetic, and it needs a working retention curve first.

**Counting installs as users is the oldest self-deception in this subject.** An install is a file on a phone. A signup is a form submission. Neither is a user. Only "did the thing" counts, which is why that row sits at the bottom of the ask ladder.

**Hundred is a target, not a magic number.** Below roughly a hundred actives, percentages are stories about individuals rather than statistics — M07 said this and it is the reason this chapter exists. Three hundred actives is where funnels start behaving; a thousand is where chapter nine's experiments become possible at all.

**The social cost is real and it is temporary.** Asking classmates to try your thing is genuinely uncomfortable, you will feel like a nuisance, and roughly four in five will say no or say nothing. The teams that get through this module are not the ones who found it easy. They are the ones who kept a log and did it anyway.

### Verify it

AI is good at drafting your messages and terrible at telling you where your users are. Four checks on anything it gives you.

1. **Does the plan assume traffic you do not have?** Ask for an acquisition plan and you will get SEO, content, launch-site posts and referral loops. Every one of those amplifies existing traffic. At eleven users there is nothing to amplify, and a plan full of them is a plan to do nothing for six weeks. Red flag: any step whose first requirement is "drive traffic to".
2. **Does the plan assume a budget?** "Run a small paid campaign" is a sentence that ends your free tier. Say the budget is zero rupees in the brief and check that every step survives it.
3. **Is the message written for one person or for everyone?** A draft that opens "Hi! We've built an exciting new app" is a broadcast and will be ignored. Make it name the person and name the specific problem you heard them describe in M01.
4. **Would your persona recognise themselves?** Ask AI who it thinks the message is aimed at, without telling it. If the answer is not your M01 persona, you are recruiting the wrong hundred people.

Red flags in your own log: a column for signups but none for reached value; every row the same channel; no date column; every name a friend; and a plan with fewer than fifty rows against it, which means it was never actually run.

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
