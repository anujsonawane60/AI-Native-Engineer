---
id: m12-05-store-submission
module: m12-ship-apps-for-real
slug: store-submission
title: "App Store and Play Store Submission, Start to Finish"
order: 5
duration_min: 35
is_free: false
mission: true
volatility: high
last_reviewed: 2026-11-16
---

## Key learnings
- Sequence store submission so the waiting happens in parallel with your building
- Protect your signing keys, and understand why losing one can be unrecoverable
- Fill the data safety and privacy declarations truthfully and survive review
- Budget the real money and the real calendar days for a student team
- Read a rejection, fix the actual cause, and resubmit without starting over

## Lesson

### The pain

A team finishes their app on a Wednesday. Demo day is Friday. They open the Play Console for the first time, pay the fee, and discover the following over the next eleven days.

Identity verification needs a government ID and a matching address, and takes days. Their package name has a typo they cannot change. Google will not accept their `.apk`, only an `.aab`. The listing needs a privacy policy at a live public URL, which does not exist. The data safety form asks eighteen questions about data they have never catalogued. A content rating questionnaire must be completed. And because it is a brand-new personal developer account, Google requires a **closed test with at least 12 testers for 14 continuous days** before they can even apply for production access.

They demo on a laptop browser. The app reaches the Play Store five weeks later.

A second team makes a worse mistake. They ship, they are live, and three months later they lose the laptop with the keystore file on it. There is no backup. With self-managed signing, that is the end: no update can ever be published for that app again, because Google will only accept uploads signed with the same key. Their only option is a new listing with zero installs, zero reviews and zero ranking.

Both failures are calendar and process failures. Neither is a coding failure.

### The idea

**Store submission is mostly waiting, and the waiting is outside your control — so start it on day one and do your building inside it.**

The second half: **your signing key is the one artefact in this course that you cannot regenerate.** Code can be rewritten, databases restored, servers rebuilt. A lost signing key, in the worst configuration, permanently severs you from your own published app and every user who installed it.

Everything else is a sequence. In the right order, submission is an afternoon of forms spread across two weeks of waiting. In the wrong order, five weeks.

### The analogy

The canteen stall needs an FSSAI food licence before it serves one plate. The licence needs an application, identity proof, an inspection and a waiting period. None of that is cooking, and no amount of cooking skill shortens it.

The stall also needs a price board and a declaration of what is in the food. If the board says "no onion" and the inspector finds onion, the problem is not the onion — it is the declaration. Being wrong on your own label is a far more serious offence than the onion.

**Where the analogy breaks**, in the way that catches teams out. The inspector visits occasionally and the licence lasts a year. App review happens on **every single update you ever ship**, by a different reviewer each time, against rules that change without anyone telling you. A build that passed last month can be rejected this month for a policy that did not exist then. That is why this chapter carries a high-volatility tag: the shape of the process is stable, the specifics are not, and you must read the console rather than trust this page or an AI's memory.

### Picture it

```
 WEEK 1 (start on day one, in parallel with building)
 ┌──────────────┐   ┌──────────────┐   ┌───────────────────┐
 │ create acct  │──▶│ identity     │──▶│ waiting: a few    │
 │ pay the fee  │   │ verification │   │ days, not in your │
 └──────────────┘   └──────────────┘   │ control           │
                                       └───────────────────┘
 WEEK 1-2 (your work)
 ┌──────────────┐   ┌──────────────┐   ┌───────────────────┐
 │ package name │──▶│ signed .aab  │──▶│ listing, rating,  │
 │ (PERMANENT)  │   │ keys backed  │   │ data safety,      │
 └──────────────┘   │ up offsite   │   │ privacy policy URL│
                    └──────────────┘   └───────────────────┘
 WEEK 2+ (waiting again)
 ┌──────────────┐   ┌──────────────┐   ┌───────────────────┐
 │ internal /   │──▶│ review:      │──▶│ staged rollout:   │
 │ closed test  │   │ days, may be │   │ 5% → 20% → 100%   │
 └──────────────┘   │ rejected     │   └───────────────────┘
                    └──────────────┘
```

Submission alternates between short bursts of your own work and long stretches of waiting on someone else, which is why the account and the identity check must start before you write the first screen.

### Where it lives

Submission sits at the end of your build pipeline and touches four earlier modules. The artefact you upload comes out of M06's CI/CD and secrets discipline. The privacy policy and the data declarations are M07's consent and privacy chapter turned into a form with legal weight. The keystore is a secret, so M01's rule about what never enters git applies absolutely. The staged rollout you finish with is the beginner version of M13's release engineering.

Inside your team this is one person's standing job, because the console has one account and the waiting needs chasing.

### The sequence, in order

**1. Create the developer account today.** Google Play Console: a **one-time** fee of about US$25, roughly ₹2,100. Apple Developer Program: an **annual** membership of about US$99, roughly ₹8,400 *every year*. For three students the Play fee is a shared pizza; the Apple fee is more than a fifth of this course tier's price, and you pay it again next year. One shared Play account per team, Android first, iOS only if a real user needs it. Check current fees in the console — they move.

**2. Complete identity verification immediately.** Both stores verify who you are, and this is pure waiting. Do it while still building screens.

**3. Fix the package name and bundle identifier.** `com.yourteam.yourapp`. **Permanent.** You cannot change it after publishing — only publish a different app. Check for `example`, `myapp` and typos now.

**4. Set up signing, and back the keys up.** On Android, enrol in **Play App Signing**: Google holds the key that signs what users download, and you hold an *upload key* for what you send Google. A lost upload key can be reset; if you manage signing yourself, a lost keystore is unrecoverable. Expo's build service can hold your credentials — but export the keystore and its passwords anyway, into a password manager the whole team can reach plus an offline copy.

**5. Produce the right artefact.** Google requires an **AAB** (Android App Bundle), not an `.apk`. Apple requires an `.ipa`. Your Android **version code** must increase on every upload and can never be reused; same for the iOS build number.

**6. Write the privacy policy and host it.** Both stores require a live public URL before you can submit, saying what you collect, why, who you share it with and how a user deletes their data.

**7. Fill the data declarations honestly.** Play calls it the **Data safety** form; Apple calls it **App Privacy** details. Both ask what you collect, whether it is linked to identity, whether it is shared, and whether it is used for tracking. The single most dangerous thing you can do here is **declare that you collect nothing while your app contains an analytics or crash-reporting library.** That is a false declaration, and the consequence escalates from rejection to app removal to account termination. Your crash reporter collects device identifiers. Your analytics collects usage. Declare them.

**8. Complete the content rating and audience declarations.** Both stores ask whether the app targets children, contains ads, has user-generated content or offers financial features. The easiest rejection to avoid.

**9. Give the reviewer a way in.** If your app requires login, supply working demo credentials. A reviewer who cannot get past your login screen rejects you.

**10. Ship to a testing track first, not production.** Play has internal, closed and open tracks; Apple has TestFlight. Internal testing reaches up to a hundred people you nominate, fast, and is where you discover your release build differs from your development build. And the rule that ambushes student teams: a **new personal Play developer account must run a closed test with at least 12 testers opted in for 14 continuous days** before it can apply for production access.

**11. Release with a staged rollout.** Play lets you release to 5%, watch your crash rate, then 20%, then 100%. Cheapest insurance in the module, and chapter 8 is what you watch during it.

### Words to own

| Term | Plain meaning | Example |
|---|---|---|
| Package name / bundle id | The app's permanent global identity | `com.yourteam.tiffin` |
| AAB | Android App Bundle, the required upload | Play builds per-device downloads |
| Keystore | File holding your Android signing key | Lose it and you may be locked out |
| Upload key | The key you sign uploads with | Resettable under Play App Signing |
| App signing key | The key users' devices trust | Held by Google if you enrol |
| Version code | A number that must always increase | 7, then 8, never 7 again |
| Data safety form | Play's declaration of what you collect | Must include your analytics |
| Testing track | Internal, closed or open release lane | Closed test, 12 testers, 14 days |
| Staged rollout | Release to a percentage of users | 5% today, 100% next week |

### Tradeoffs and gotchas

**Android first is almost always right for an Indian student team.** Play reaches the overwhelming majority of your users, the fee is one-time, and you can install and test freely. iOS costs ₹8,400 a year before anyone — including you — can install your build.

**Rejections are normal and are not personal.** Read the guideline they cite, fix that specific thing, and explain the change in the resubmission note. Do not rebuild the app, do not change three unrelated things, and do not argue.

**Both stores demand account deletion if you have accounts.** If a user can sign up in your app, they must be able to delete the account inside the app, not by emailing you. A one-afternoon feature and a guaranteed rejection if missing.

**Target API level requirements move every year.** Play refuses uploads built against an Android version that is too old. Keeping the Expo SDK current is what handles this.

**Your release build is not your development build.** Different optimisation, different engine settings, real signing, no dev server. Test the release artefact on a real phone before every upload.

**Do not let an AI tell you the current rules.** Fees, forms, timelines and policy names here were accurate when written and will drift. The console is the source of truth, and a confident AI answer about store policy is exactly the plausible-but-wrong failure from M00.

### Verify it

**Red flag: an AI stating current fees, review times or policy rules from memory.** These are the highest-volatility facts in the course. Ask AI *where in the console to look* and *what the answer will determine*, then read it yourself. Any number it gives you is a hypothesis.

**Red flag: a data safety form filled optimistically.** Cross-check the draft against the actual libraries in your app. Does your crash reporter send a device identifier? Does your analytics record screens viewed? Does your push setup store a token? Each is data. The honest form is longer than the convenient one and it is the only one that keeps your account.

**Red flag: a signing setup with no backup instruction.** If the plan does not say where the keystore and its passwords live and who else can reach them, it is incomplete. Ask: "if my laptop is stolen tonight, can my team publish an update tomorrow?" Then test it by having a teammate retrieve the credentials.

**Red flag: a secret inside the app bundle.** Before you upload, unzip your own `.aab` and search it for anything resembling a key. Whatever you find, strangers can find.

**Red flag: a submission plan with no demo account.** A login wall plus no reviewer credentials is a planned rejection.

**Verify the artefact, not the source.** Install the exact signed build you are about to upload onto a real budget phone and complete your main flow as a new user.

## Walkthrough
1. Today, before anything else: create the Google Play Console account, pay the one-time fee, and start identity verification. Write the date in your decision log.
2. Agree the final package name as a team and check it is free of `example`, `myapp` and typos. Record it. It is permanent.
3. Set up signing with Play App Signing enabled. Export the upload keystore and its passwords, store them in a password manager the whole team can reach, and keep a second copy offline.
4. Have a teammate who did not do step 3 retrieve the credentials and confirm they can reach them. If they cannot, the backup does not exist.
5. Write your privacy policy and host it at a live public URL. It must say what you collect, why, who you share it with, and how a user deletes their data.
6. Catalogue every third-party library in your app that touches user data — analytics, crash reporting, push, payments — and what each one sends. This list is the input to your data declarations.
7. Fill the data safety form from that catalogue, not from memory. Then complete the content rating questionnaire and the ads, audience and user-generated-content declarations.
8. Build a release `.aab`, install the matching release build on a real budget phone, and complete your main user flow as a brand-new user.
9. Unzip the build and search it for anything resembling a key or a credential. Fix whatever you find before uploading.
10. Create a permanent demo account with realistic data for reviewers, and add an in-app account deletion path if users can sign up.
11. Upload to a closed testing track and recruit at least twelve testers from your cohort. Note the date: the fourteen-day clock starts now.

## Talk to AI
```prompt
CONTEXT:      Three-person student team in India. React Native app built
              with Expo. Product: [one sentence]. Android first; iOS only
              if justified. Users log in with [auth method]. We have never
              published an app.
              Libraries in our app that touch user data: [list them all,
              including analytics, crash reporting, push, payments]
              What our app collects, to the best of our knowledge:
              [list it honestly]
              Our planned package name: com.[team].[app]
              Our target live date: [date]

GOAL:         A dated, ordered submission plan that gets us into a Play
              Store testing track with no avoidable rejection, and keys we
              cannot lose.

REQUIREMENTS: 1. A plan ordered by date, clearly separating days we are
                 working from days we are waiting on someone else
              2. For every store form or declaration, tell me which of my
                 facts above answers it, and flag anything I have not told
                 you that I will need
              3. A signing plan that states exactly where the upload key
                 and its passwords live, who can reach them, and what
                 happens if one laptop is lost
              4. A draft data safety declaration derived from my library
                 list, with a line per library saying what it sends
              5. The complete list of things that will get us rejected
                 given what I have described, each with the fix
              6. A rollout plan with percentages and what we watch at each
                 stage before increasing
              7. For every fee, policy rule or review timeline you mention,
                 tell me where in the console to confirm it rather than
                 asserting the value

CONSTRAINTS:  Total budget under Rs 3,000. No Mac. Free tiers elsewhere.
              Assume our developer account is brand new and personal, so
              any mandatory testing period applies to us. We have about two
              hours a day each.

NON-GOALS:    Do not write our privacy policy text for us to paste
              unreviewed. Do not advise us to declare less than we collect.
              Do not propose new features. Do not plan the iOS submission
              yet.

ACCEPTANCE:   - Every waiting day in the plan is labelled as waiting
              - The data safety draft has one line per library in my list
              - The signing plan survives the question "my laptop was
                stolen tonight"
              - You have named at least three rejection causes specific to
                what I described
              - You have not asserted a single current fee or review time
                as fact without telling me to verify it in the console

ASK:          Give me the dated plan and the rejection list first. Do not
              draft any form content until I confirm the dates are
              compatible with our target live date.
```

## Mission

**Objective.** Your own product is live in a Play Store closed testing track, installed by at least twelve real testers, with signing keys your whole team can recover.

**Steps.**
1. **Start the clock** — create the developer account and begin identity verification before you do anything else in this mission. Record the date.
2. **Lock the identity** — agree and record the final package name. Confirm as a team that it is what you want forever.
3. **Secure the keys** — enable Play App Signing, export the upload keystore and passwords, store them where the whole team can reach them, and have a teammate prove they can.
4. **Catalogue the data** — list every library that touches user data and exactly what each sends. This is your declaration evidence.
5. **Brief** — write a Director's Brief for the submission. CONSTRAINTS must include the brand-new-account testing requirement; NON-GOALS must include "do not declare less than we collect".
6. **Prepare** — privacy policy at a live URL, data safety form, content rating, audience and ads declarations, demo account for reviewers, in-app account deletion.
7. **Build and check** — produce the release `.aab`, install the matching build on a real budget phone, complete the main flow as a new user, and search the bundle for secrets.
8. **Submit** — upload to a closed testing track and recruit twelve testers from your cohort. Send them the opt-in link and confirm each one has installed.
9. **Verify** — have a tester who is not in your team install from the store link and complete your main flow on their own phone, and report back in writing.
10. **Document** — one decision-log entry: Android-first and why, what you declared and why, where the keys live, and what the fourteen-day clock means for your timeline.

**Upload.**
- Your Director's Brief → post caption or an attached file
- Your dated submission plan with waiting days marked → screenshot or text
- Your library-to-data catalogue and the data safety answers derived from it
- Proof of signing backup → a screenshot of a teammate confirming they retrieved the credentials, with secrets redacted
- The closed testing track → screenshot of the track with the tester count
- Proof it works for someone else → written report from a tester outside your team, with their phone model
- The live privacy policy URL
- Reflection, 150 words max → one tradeoff you chose, one thing the AI got wrong about store policy

**Self-check.** Tick every box before you submit.
- [ ] Our developer account exists and identity verification is submitted
- [ ] Our package name is final, recorded, and contains no placeholder text
- [ ] Play App Signing is enabled, not self-managed signing
- [ ] A teammate other than me has retrieved the keystore credentials successfully
- [ ] Our data safety declaration lists every library that sends anything
- [ ] Our privacy policy is at a live public URL and describes data deletion
- [ ] A reviewer can log in with permanent demo credentials we created
- [ ] Users can delete their account inside the app
- [ ] I unzipped the build and found no keys or credentials in it
- [ ] I installed the exact signed release build on a real budget phone and completed the main flow
- [ ] At least twelve testers have opted in and I know the date the clock started
- [ ] My reflection names a store rule I had to verify in the console rather than trust

**Stretch.** Write your own rejection playbook: for the five most likely rejection reasons for your app, the exact fix and the exact sentence you would put in the resubmission note.

## Stuck? Ask AI
```prompt
Explain app signing to someone who has never published an app. Cover what an upload key is, what an app signing key is, why there are two, what Play App Signing changes, and precisely what happens if each one is lost. Use a non-technical analogy, then ask me to explain back which key I must never lose and why.
```

```prompt
My app submission was rejected. Here is the exact rejection text: [paste it]. Tell me which specific guideline this is about, what the reviewer actually saw, the smallest possible change that fixes it, and what I should write in the resubmission note. Do not suggest any change unrelated to this rejection.
```

```prompt
Here is the list of third-party libraries in my mobile app: [paste it]. For each one, tell me what categories of user data it is likely to collect or transmit and whether that data is typically linked to a user's identity. Then tell me which of these I must declare on a store data-safety form, and warn me about any I might assume is harmless. Tell me where to confirm each answer rather than trusting you.
```

## Quiz

### Q1 · L3
Why should a student team open its Play Console account on day one of this module rather than in the final week?
- (a) The registration fee increases over time
- (b) Account creation, identity verification, a mandatory new-account testing period and review are all waiting time you cannot code away ✅
- (c) The package name is reserved on a first-come basis
- (d) Early accounts get faster review
> The process alternates between short bursts of your work and long stretches of waiting on someone else. Starting the waiting early is the only lever you have on the total calendar time.

### Q2 · L4
Which signing configuration protects you best against losing a laptop?
- (a) Self-managed signing with the keystore on one developer's machine
- (b) Play App Signing, where Google holds the app signing key and your upload key can be reset ✅
- (c) A new keystore generated for every release
- (d) No signing until the app reaches production
> With Play App Signing, a lost upload key can be reset. With self-managed signing, a lost keystore means Google will never accept another update for that listing, and your only option is a new app with zero installs.

### Q3 · L4
Your app includes a crash reporter and an analytics library. On the data safety form you declare that you collect no data. What is the real risk?
- (a) Slightly slower review
- (b) A false declaration, which escalates from rejection to app removal to account termination ✅
- (c) Your crash reports will stop arriving
- (d) Nothing, since crash data is anonymous
> Crash reporters send device identifiers and analytics records usage; both are collection. Being wrong on your own label is treated far more seriously than collecting the data in the first place.

### Q4 · L3
Your app requires login and the reviewer rejects it without testing the main flow. What was most likely missing?
- (a) A content rating
- (b) Permanent working demo credentials for the reviewer ✅
- (c) An Android App Bundle instead of an APK
- (d) A staged rollout plan
> A reviewer who cannot get past the login screen rejects the submission. It is the most common avoidable rejection, and the fix is a permanent demo account with realistic data.

### Q5 · L5
An AI confidently tells you the current Play Store registration fee, the exact review turnaround and the precise wording of a policy. How should you treat this?
- (a) As fact, since store policies are well documented
- (b) As a hypothesis to verify in the console, because these are the most volatile facts in the course ✅
- (c) As fact for Android but not for iOS
- (d) As irrelevant, since fees do not affect the submission sequence
> Fees, timelines and policy names drift constantly, and a confident wrong answer is exactly the plausible-but-wrong failure mode from M00. Ask AI where to look; read the number yourself.

## Go deeper
- [Google Play Console Help: publish your app](https://support.google.com/googleplay/android-developer/answer/9859152)
- [Apple: App Review Guidelines](https://developer.apple.com/app-store/review/guidelines/)
