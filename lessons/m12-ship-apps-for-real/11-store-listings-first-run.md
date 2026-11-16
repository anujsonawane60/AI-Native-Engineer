---
id: m12-11-listings-first-run
module: m12-ship-apps-for-real
slug: store-listings-first-run
title: "Store Listings, Screenshots and the First-Run Experience"
order: 11
duration_min: 25
is_free: false
mission: false
volatility: high
last_reviewed: 2026-11-16
---

## Key learnings
- Write a listing for the user standing in a queue, not for the developer
- Build screenshots that read as a captioned story rather than raw captures
- Remove the login wall between a new user and your product's first value
- Measure store conversion and first-run completion as two separate numbers
- Spot a listing that names frameworks and a first run that asks before it gives

## Lesson

### The pain

Your app is live. Over a fortnight, 1,100 people reach your Play Store page. 44 install it.

You look at the page for the first time as a stranger would. The title is your product name, which means nothing to anybody. The short description says "A React Native app for canteen ordering built with Supabase". The five screenshots are raw screen captures, in order: an empty list, the login screen, a settings page, a form with placeholder text, and one actual useful screen at position five, which nobody scrolled to. There is no feature graphic. The full description begins "In today's fast-paced world".

Nothing on that page tells a hungry student that she can order her lunch without standing in the 1:10 pm queue.

Then look at the 44 who did install. 31 of them never complete a single order. The app opens on a sign-up screen. Before showing anything at all it asks for a phone number, an OTP, a name, and notification permission. The first screen a new user sees after all that is an empty list saying "No orders yet", which is technically true and completely useless. They have given you four pieces of information and received nothing.

Two separate funnels, both leaking, and neither leak is a code bug.

### The idea

**Your store listing and your first thirty seconds are one continuous funnel, and both are read by someone standing in a queue with one hand free.**

That person gives your listing about four seconds before deciding to scroll on. She gives your first screen about ten more. In that time she is answering exactly one question — "will this do something for me?" — and every element that is not answering it is in the way.

Two rules cover nearly everything:

**Lead with the user's outcome, in their words.** Not what the app is, not how it was built. "Order your canteen lunch, skip the queue" beats any sentence containing the word "platform".

**Give before you ask.** The user should see your product working, with real content, before you request an account, a permission or a payment. Every ask moved after the first value raises completion dramatically.

### The analogy

The canteen's board outside answers one question for someone walking past: what can I eat, and for how much. Not "established 2019", not "a modern multi-cuisine dining solution". A photo of a thali and "₹60, ready in 2 minutes".

Then the first thirty seconds at the counter. A good counter puts a plate in your hand and takes your money after. A bad counter asks for your student ID, your registration number and your phone number before telling you what is available today.

**Where the analogy breaks:** the board is seen by people already walking past the canteen, so they have some context. Your store page is seen by someone who has never heard of you, scrolling a list of nine similar apps, with no idea what category your product is in. You get less context and less patience than any physical signboard, which is why the first line has to do all the work.

### Picture it

```
 1,100 see the listing
        │
        │  listing: title, short description, first 2 screenshots
        ▼
    44 install            ◀── store conversion: 4%
        │
        │  first run: what happens before any value
        ▼
    13 complete one order ◀── activation: 30% of installs
        │
        ▼
     ? come back tomorrow  ◀── retention, measured in M16

  biggest leaks: screenshot 1 is a login screen
                 first screen after signup is empty
```

Two measurable drop-offs — install rate from the listing and activation rate inside the app — sit back to back, and each has a specific cause you can see by looking at the page and the first screen as a stranger.

### What a listing is made of

Both stores give you a small set of slots, and each has one job.

**App name**, roughly 30 characters. Your name plus the outcome, if it fits: "Tiffin — Canteen Orders". This is also the strongest signal for store search, so the words a user would actually type belong here.

**Short description**, about 80 characters on Play. This is the one line shown before anyone taps "read more", and it is the single highest-value piece of copy you will write in this module. One outcome, no adjectives.

**Screenshots**, two to eight, and the first two are what almost everyone sees. Treat them as a comic strip: each one is a real screen with a short caption added above or below saying what the user achieves there. Never a raw capture, never the login screen, never an empty state. Screenshot one should be your product's single best moment.

**Feature graphic** on Play, a wide banner. Many student teams skip it, and skipping it makes the page look abandoned.

**Full description.** Read by a minority. Lead with three lines of outcome, then a short list of what it does, then the practical details — works on slow networks, works offline, available in Hindi and Marathi. Those last three are genuine selling points to your users and nobody else mentions them.

**Localised listings.** You can supply the whole set per language. A Marathi-speaking user searching in Marathi will not find an English-only listing, which is a direct consequence of chapter 10's work.

### Where it lives

The listing lives in the store console, which means chapter 5 is its neighbour: the same console, the same account, and a listing change can be submitted independently of a build. The screenshots must come from the accessible, localised, performant app you built in chapters 9 and 10, taken on a real device.

The first-run experience lives in your app, and it is where everything in this module meets. The deferred permission from chapter 7, the offline-tolerant first screen from chapter 4, and the notification ask from chapter 3 all belong *after* the first value, not before it.

And the numbers you create here are the input to M16. Store conversion and activation rate are the first two steps of the funnel that module teaches you to analyse properly; instrumenting them now means that in M16 you have real data instead of a blank dashboard.

### Words to own

| Term | Plain meaning | Example |
|---|---|---|
| Short description | The one line shown before "read more" | 80 characters, one outcome |
| Feature graphic | Wide banner image on the Play listing | 1024 × 500 pixels |
| ASO | Choosing words users actually search | "canteen order" in the title |
| Store conversion | Installs divided by page views | 44 of 1,100 is 4% |
| Activation | New users who reach first value | Completed one order |
| Time to first value | Seconds from open to something useful | Target: under 15 |
| Login wall | Signup demanded before any value | The biggest first-run leak |
| Empty state | What a screen shows with no data | Should teach, not apologise |
| Skeleton screen | Placeholder shapes while loading | Better than a blank screen |
| Aspirational screenshot | Shows more than the app delivers | Causes one-star reviews |

### Tradeoffs and gotchas

**Do not oversell in screenshots.** A screenshot showing features you do not have earns an install and then a one-star review, and your rating is far harder to repair than your install rate. Show your best real moment, not an imagined one.

**Guest mode is usually the right answer, and sometimes it is not.** Letting a user browse and reach value before signing up is the single biggest first-run improvement available to most products. But if your product is inherently personal from the first tap, the honest fix is different: ask for the minimum — one field, not four — and explain in one line what they get.

**Ask for one thing at a time, in order of value received.** Account when they need to save something. Notification permission after their first order. Location when they tap "find nearest". Chapter 3 and chapter 7 both said this; the first-run screen is where it is most often violated.

**Empty states are a teaching opportunity you are wasting.** "No orders yet" tells the user nothing. "You have not ordered yet — today's thali is ₹60, tap below" turns an empty screen into the first useful thing they saw.

**Never ask for a rating on first launch.** It is the most reliable way to collect one-star ratings from people who have not used your product. Ask after a successful outcome, at most once, and never during.

**Reply to reviews, including the angry ones.** Both stores let you reply publicly, and a specific reply naming the fix and the version it shipped in is read by every future visitor to your page. It is also, frequently, how you learn about the bug that chapter 8 should have caught.

**Store policies on listing content change, and so do asset sizes.** Image dimensions, character limits, rules about what may appear in a screenshot, and restrictions on words like "best" or "free" all move. Read the console when you write the listing, which is why this chapter carries a high-volatility tag.

### Verify it

**Red flag: a listing written for the developer.** Scan your own copy for framework names, architecture words and anything a user would not say: "React Native", "platform", "solution", "seamless", "leveraging". Every one of those is a slot that should have held an outcome. Then run the real test: show the page to someone outside your team for four seconds, take it away, and ask what the app does. If they cannot say, the listing has failed regardless of how it reads to you.

**Red flag: screenshots that are raw captures.** Check what your first two screenshots actually are. If either is a login screen, a settings page or an empty state, replace it. If none has a caption, add one to each — a short line saying what the user achieves there.

**Red flag: a first run that asks before it gives.** Count the number of taps, fields and permission dialogs between opening the app and the first moment the user gets something useful. Write that count down. If anything is being asked for before the first value, ask AI to move it after and explain what breaks.

**Red flag: an AI stating current listing requirements as fact.** Character limits, image dimensions and content rules are volatile. Ask where in the console to confirm each one, exactly as in chapter 5.

**Red flag: no numbers.** If you cannot say what your store conversion rate and your activation rate are, you are guessing about both halves of this funnel. The console gives you page views and installs; your own analytics must give you installs-to-first-value. Instrument that before you start changing copy, or you will never know whether a change helped.

**Verify with a stranger and a real phone.** Hand your budget phone to someone who has never seen the product, with the app not installed, and say nothing. Watch them find it, install it and use it. Time how long until they achieve anything, and write down every moment they hesitated. Twenty minutes of this is worth more than any amount of discussion.

## Walkthrough
1. Open your own store listing on a phone, as a stranger would. Screenshot exactly what is visible before any scrolling or tapping.
2. Write down the one sentence a hungry, hurried user would need to read to install your app. That sentence is the draft of your short description.
3. Rewrite your app name, short description and the first three lines of your full description to lead with the user's outcome. Remove every framework name and every adjective you cannot defend.
4. Take fresh screenshots on your real budget phone, with real content, from the accessible and localised app. Pick your single best moment for screenshot one.
5. Add a short caption to every screenshot saying what the user achieves on that screen. No raw captures, no login screens, no empty states.
6. Create the feature graphic. Keep the text short and large enough to read at thumbnail size, and put no text inside it that you would need to translate.
7. Produce a localised listing in Hindi or Marathi using the strings a cohort member reviewed in chapter 10.
8. Now the app: count the taps, fields and permission dialogs between opening and first value. Write the count down as your baseline.
9. Remove or defer everything you can from before that first value — guest browsing, one field instead of four, permissions moved later.
10. Rewrite every empty state in the app so it teaches the next action instead of apologising for having no data.
11. Hand the phone to someone outside your team, with the app uninstalled, and watch silently as they find, install and use it. Record their time to first value and every hesitation.

## Talk to AI
```prompt
CONTEXT:      Our app is live on the Play Store. Product: [one sentence in
              your own words]. Users: [who they are and what they are
              doing when they would use it].
              Numbers from the console and our analytics:
              - store listing page views, last 14 days: [X]
              - installs: [X]
              - new users who completed [your core action]: [X]
              Our current listing copy:
              - app name: [paste]
              - short description: [paste]
              - first three lines of full description: [paste]
              Our current screenshots, in order: [describe each one]
              Our first-run sequence, step by step, from opening the app to
              the first useful thing: [list every tap, field and permission
              dialog in order]

GOAL:         More of the people who see our listing install it, and more
              of the people who install it reach our core action on day
              one.

REQUIREMENTS: 1. Calculate my store conversion rate and my activation rate
                 and tell me which is the bigger problem
              2. Rewrite my app name and short description to lead with the
                 user's outcome, giving me three options for each with the
                 reasoning, and name every developer-facing word you removed
              3. Tell me, for each of my current screenshots, keep or
                 replace, and what the replacement should show - plus a
                 caption for each in under eight words
              4. Audit my first-run sequence: count what is asked before
                 the first value, and give me a reordered sequence with
                 everything possible moved after it
              5. Rewrite my empty states to teach the next action
              6. Tell me the two events I must instrument to measure
                 activation, and where they fire
              7. For every character limit, image dimension or content rule
                 you mention, tell me where in the console to confirm it
                 rather than asserting it

CONSTRAINTS:  Free tiers only. Screenshots must be of the real app on a
              budget Android phone, with real content, nothing we cannot
              deliver. Our users may read Hindi or Marathi better than
              English. Assume four seconds of attention on the listing.

NON-GOALS:    Do not propose paid acquisition or ads. Do not invent
              features we do not have. Do not use the words seamless,
              leveraging, platform or solution. Do not ask users to rate
              the app on first launch.

ACCEPTANCE:   - You have stated both rates as percentages and said which to
                fix first
              - Every copy option is a user outcome, with no framework or
                architecture words
              - Nothing is asked of the user before the first value, or you
                explain precisely why it is unavoidable
              - Every screenshot has a caption of eight words or fewer
              - Every empty state tells the user what to do next
              - You have named the two events I must instrument

ASK:          Start with the two rates and which one is the bigger leak.
              Do not rewrite any copy until I have agreed where the problem
              actually is.
```

## Stuck? Ask AI
```prompt
Here is my app's store listing copy and a description of my screenshots: [paste them]. Act as a hurried student with one hand free who has never heard of this app and four seconds of attention. Tell me, honestly and unkindly, what you think the app does and whether you would install it. Then tell me which single element lost you.
```

```prompt
Explain the difference between store conversion rate and activation rate, why a team can have a good one and a terrible other, and what each one tells you to fix. Then give me four short scenarios with numbers and ask me which problem each team has, without telling me the answers first.
```

```prompt
My app's first screen after signup says "No orders yet". Give me five different rewrites of this empty state that teach the user what to do next, ranked by how useful they are to someone who has just installed the app and knows nothing. For each one, say what information the screen would need in order to show it.
```

## Quiz

### Q1 · L3
Your listing gets 1,100 views and 44 installs, and 13 of those 44 complete your core action. Which number do you fix first?
- (a) Installs, because 4% conversion is the larger leak in absolute people ✅
- (b) Activation, because 30% is the lower percentage
- (c) Retention, because it predicts revenue
- (d) Neither; both are normal
> 1,056 people left at the listing and 31 left inside the app. The listing is losing an order of magnitude more people, so it is where the first hour of work belongs.

### Q2 · L2
What is the job of the short description on a store listing?
- (a) Summarise the technical architecture
- (b) Deliver one user outcome in the single line shown before anyone taps "read more" ✅
- (c) List every feature for store search
- (d) Explain the pricing model
> It is the highest-value copy you write, because almost everyone reads it and almost nobody expands the full description. One outcome, no adjectives.

### Q3 · L4
Which first screenshot is the right choice?
- (a) The login screen, so users know an account is needed
- (b) Your product's single best real moment, with a short caption ✅
- (c) An empty state, so expectations are realistic
- (d) The settings screen, showing configurability
> Screenshots are read as a captioned comic strip and almost everyone only sees the first two. Position one is your best real moment — not raw, not empty, and never overselling.

### Q4 · L4
31 of 44 new users never complete your core action. The app opens with a phone number, an OTP, a name field and a notification prompt. What is the fix?
- (a) Make the signup form prettier
- (b) Let them reach first value before asking for anything, then ask one thing at a time in order of value received ✅
- (c) Add an onboarding tour before the signup
- (d) Send a push notification reminding them to finish signup
> Give before you ask. Four asks before any value is the largest first-run leak there is, and every ask you move after the first value raises completion.

### Q5 · L5
An AI rewrites your listing and the new short description reads "A seamless cross-platform ordering solution built on a modern stack". What is wrong, beyond style?
- (a) It is too short for the character limit
- (b) Every word describes the app to a developer and none of it answers "will this do something for me?" ✅
- (c) It will be rejected by store review
- (d) It cannot be localised into Hindi
> The listing is read in four seconds by someone with no context. Framework and architecture words occupy the slots that should hold the user's outcome, which is the single most common listing failure.

## Go deeper
- [Google Play: create an effective store listing](https://support.google.com/googleplay/android-developer/answer/9866151)
- [Apple: product page optimization](https://developer.apple.com/app-store/product-page/)
