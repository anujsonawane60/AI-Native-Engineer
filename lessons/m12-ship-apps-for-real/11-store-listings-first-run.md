---
id: m12-11-listings-first-run
module: m12-ship-apps-for-real
slug: store-listings-first-run
title: "Store Listings, Screenshots and the First-Run Experience"
order: 11
duration_min: 15
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

Your app is live. Over a fortnight 1,100 people reach your Play Store page and 44 install it.

Look at the page as a stranger would. The title is your product name, which means nothing to anybody. The short description says "A React Native app for canteen ordering built with Supabase". The screenshots are raw captures: an empty list, the login screen, a settings page, and one useful screen at position five that nobody scrolled to.

Now the 44 who installed: 31 never complete an order. The app opens on a sign-up screen asking for a phone number, an OTP, a name and notification permission before showing anything, and the first screen after all that is an empty list saying "No orders yet". Two funnels, both leaking, and neither leak is a code bug.

### The idea

**Your store listing and your first thirty seconds are one continuous funnel, and both are read by someone standing in a queue with one hand free.**

She gives your listing about four seconds before scrolling on, and your first screen about ten more, answering one question — "will this do something for me?" Everything else is in the way.

**Lead with the user's outcome, in their words.** Not what the app is, not how it was built. "Order your canteen lunch, skip the queue" beats any sentence containing the word "platform".

**Give before you ask.** The user should see your product working, with real content, before you request an account, a permission or a payment.

### The analogy

The canteen's board answers one question for someone walking past: what can I eat, and for how much. Not "established 2019", not "a modern multi-cuisine dining solution" — a photo of a thali and "₹60, ready in 2 minutes". Then the counter: a good one puts a plate in your hand and takes your money after; a bad one asks for your student ID first.

**Where the analogy breaks:** the board is read by people already walking past, so they have context. Your store page is read by someone who has never heard of you, scrolling nine similar apps — less context and less patience than any signboard, so the first line has to do all the work.

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

### Where it lives

The listing lives in the store console, so chapter 5 is its neighbour: same account, and a listing change can be submitted independently of a build. Each slot has one job:

- **App name**, about 30 characters — your name plus the outcome: "Tiffin — Canteen Orders". Also the strongest store-search signal.
- **Short description**, about 80 characters on Play — the line shown before "read more", and the highest-value copy in this module. One outcome, no adjectives.
- **Screenshots**, two to eight, of which almost everyone sees the first two. A real screen plus a caption saying what the user achieves there. Never a login screen; screenshot one is your best moment.
- **Feature graphic** on Play — skipping it makes the page look abandoned.
- **Full description** — three lines of outcome, then what it does, then the practical details: slow networks, offline, Hindi and Marathi. Nobody else mentions those.
- **Localised listings**, the whole set per language: chapter 10 paying off.

Screenshots come from the accessible, localised app of chapters 9 and 10, on a real device. The first run is where the module meets: chapter 7's deferred permission, chapter 4's offline-tolerant first screen and chapter 3's notification ask all belong *after* the first value. Store conversion and activation are also M16's first funnel steps.

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

**Guest mode is usually right, and sometimes it is not.** Letting a user browse and reach value before signing up is the biggest first-run improvement most products have. If your product is personal from the first tap, ask for the minimum — one field, not four.

**Ask for one thing at a time, in order of value received.** Account when they need to save something, notification permission after the first order, location when they tap "find nearest".

**Empty states are a teaching opportunity you are wasting.** "No orders yet" tells the user nothing. "You have not ordered yet — today's thali is ₹60, tap below" makes an empty screen the first useful thing they saw.

**Never ask for a rating on first launch**, which reliably collects one-star ratings from people who have not used your product. Do reply to reviews, including angry ones: a reply naming the fix and its version is read by every future visitor.

**Listing policies and asset sizes change** — image dimensions, character limits, restrictions on words like "best" or "free". Read the console when you write the listing; hence the high-volatility tag.

### Verify it

**Red flag: a listing written for the developer.** Scan your copy for framework names and words a user would not say: "React Native", "platform", "solution", "seamless". Then show the page to someone outside your team for four seconds, take it away, and ask what the app does.

**Red flag: screenshots that are raw captures.** If either of the first two is a login screen or an empty state, replace it, and give every one a caption.

**Red flag: a first run that asks before it gives.** Count the taps, fields and permission dialogs between opening the app and the first useful moment. Anything asked before the first value moves after it.

**Red flag: an AI stating current listing requirements as fact.** Character limits, image dimensions and content rules are volatile — confirm each in the console, as in chapter 5.

**Red flag: no numbers.** Without a store conversion rate and an activation rate you are guessing about both halves of the funnel. The console gives page views and installs; your analytics must give installs-to-first-value.

**Verify with a stranger and a real phone.** Hand your budget phone to someone who has never seen the product, with the app not installed, and say nothing. Time how long until they achieve anything.

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
