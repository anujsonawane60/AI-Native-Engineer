---
id: m12-07-native-device-apis
module: m12-ship-apps-for-real
slug: native-device-apis
title: "Native Device APIs: Camera, Location, Files"
order: 7
duration_min: 25
is_free: false
mission: false
volatility: low
last_reviewed: 2026-11-16
---

## Key learnings
- Request each permission at the moment of use, with a reason the user can read
- Pick the weakest permission that does the job, including asking for none
- Resize and compress a photo on the device before it ever leaves it
- Recover gracefully when a permission is denied, permanently or temporarily
- Spot an AI design that asks for more access than the feature needs

## Lesson

### The pain

A team ships a profile screen. On first launch the app asks, one dialog after another, for camera, photos, location, contacts and notifications. Five dialogs before the user has seen a single screen of the product.

She taps "Don't allow" five times, because a stranger asking for your camera before showing you anything is exactly what a scam app does. Now none of the app's features work, and on Android the two refusals she gave the camera prompt mean the system will not show that dialog again at all — your only remaining option is a message telling her to go into Settings herself, which almost nobody does.

The same team's photo upload works perfectly on their own phones. It fails for real users because a 12-megapixel photo from a modern camera is about 4 MB, they upload it at full size, and on a weak connection with metered data that takes ninety seconds, costs the user money, and frequently times out. Their object storage bill from M04 climbs for images that are displayed at 120 pixels wide.

And one more: the app asks for background location — the kind that tracks you when the app is closed — because the AI's example code used it. Background location triggers an extra declaration and an extra review step on the Play Store, and a reviewer will ask what you need it for. The feature only ever needed the user's position once, while the app was open.

### The idea

**Every permission is a question you are asking a stranger, so ask at the moment they understand why, ask for the weakest version that works, and design for "no".**

Three rules follow, and they handle almost every case.

**Ask in context.** The permission request belongs immediately after the user taps the thing that needs it, preceded by one sentence in your own interface explaining what you will do with it. Never in app startup code.

**Prefer no permission at all.** This is the rule nobody expects. On both platforms, launching the *system* image picker or the *system* document picker requires no permission from you, because the user is choosing the file themselves in the operating system's own interface. If your feature is "let the user attach a photo", you may need no camera and no photo-library permission whatsoever. Full camera permission is only for when you need a live preview inside your app.

**Design the denial path.** A denied permission is a normal state, not an error. The feature should degrade to something useful, and your message should tell the user what they lose and how to change their mind.

### The analogy

The canteen's subsidised thali requires proof that you are a registered student. The counter asks for your ID *when you claim the subsidy* — not when you walk through the door, and not for the full-price plate.

And notice the weakest-version rule in action. The counter does not need to photocopy your ID, keep it, or know your home address. It needs to glance at it once. A counter that demanded to keep your ID card for the whole term would be asking for permanent background access to something it only needed for one second.

**Where the analogy breaks:** a student refused at the counter can argue with a human, and the human can make an exception. The operating system cannot be argued with. On Android, two refusals of the same prompt and the dialog is gone for good; your app can only point at Settings. There is no appeal, which is why the first ask has to be the good one.

### Picture it

```
 user taps "Add photo"
          │
          ▼
 ┌──────────────────────┐   no permission needed
 │ system image picker  │──────────────────────────┐
 └──────────────────────┘                          │
          │ needs live preview?                    │
          ▼ yes                                    │
 ┌──────────────────────┐                          │
 │ your screen explains │                          │
 │ why, in one sentence │                          │
 └──────────┬───────────┘                          │
            ▼                                      ▼
    ┌───────────────┐  denied   ┌──────────────────────────┐
    │ OS permission │──────────▶│ fall back to the picker, │
    │ dialog        │           │ say what was lost        │
    └───────┬───────┘           └──────────────────────────┘
            ▼ granted
   resize to 1280px, compress, then upload
```

The cheapest path needs no permission at all; a permission is only requested when the feature genuinely needs one, after your own explanation, with a working fallback when it is refused.

### Where it lives

On the device side these are Expo modules — an image picker, a camera, a location module, a document picker, a file system. Each has a matching declaration in your app config, which Expo turns into the right native entries at build time: an Android manifest permission and, on iOS, a **purpose string** explaining the use in plain language. Apple rejects apps whose purpose strings are missing or meaningless, and the string is shown to the user inside the system dialog, so it is user-facing copy, not boilerplate.

On the server side, what comes back from these APIs is an upload, which is M04's file-upload and object-storage territory, and a privacy obligation, which is M07's. A photograph is personal data. A location is more sensitive personal data. Both have to appear in the privacy policy and the data safety declaration from chapter 5 — and a photo carries EXIF metadata that may include the exact GPS coordinates where it was taken, which you probably did not mean to store.

Adding any of these modules is a native change, so remember chapter 6: a new permission cannot be shipped over the air.

### Words to own

| Term | Plain meaning | Example |
|---|---|---|
| Runtime permission | Asked while the app runs, not at install | The camera dialog |
| Purpose string | iOS text explaining why you need access | "To attach a photo to your order" |
| Permanently denied | Android stops showing the dialog | After two refusals |
| Foreground location | Position only while the app is open | Finding the nearest counter |
| Background location | Position when the app is closed | Needs extra store review |
| System picker | OS-provided chooser, no permission | Attach an existing photo |
| Scoped storage | Apps see their own files, not the disk | No general file browsing |
| EXIF | Metadata inside a photo file | Camera model, date, GPS |
| Compression quality | How much image detail you discard | 0.7 is usually invisible |
| Graceful degradation | The feature still does something useful | Type the address instead |

### Tradeoffs and gotchas

**Upload a resized copy, never the original.** A 4 MB photo displayed at 120 pixels wide is a waste the user pays for twice: once in data and once in waiting. Resize on the device to the largest size you actually display — 1280 pixels on the long edge is plenty for most app use — and compress to about 70% quality. You will usually go from 4 MB to under 200 KB with no visible difference on a phone screen. On a metered Indian connection that is the difference between a feature working and a feature being abandoned.

**Strip EXIF unless you need it.** Photos taken on a phone frequently embed GPS coordinates. If you store the original, you are storing the user's home location without telling them. Strip the metadata during the resize, or declare it.

**Foreground location is almost always enough.** Background location means a separate Play Store declaration, a prominent-disclosure requirement, and a reviewer asking you to justify it. Ask yourself whether you need the position *while the app is open* — the answer for a student project is almost always yes, and the correct permission is the weaker one.

**Location accuracy is a dial, and the coarse setting is cheaper.** Approximate location is faster, uses less battery, and users grant it more readily. "Which counter is nearest" does not need three-metre precision.

**Storage is scoped, so the device "files" are not a filesystem you can roam.** Your app has its own directory. Reaching anything else goes through a picker the user drives. Plan features around that rather than around a desktop mental model.

**Permissions can be revoked at any time, including while your app is backgrounded.** Check the status when you use the feature, not once at startup and never again. Modern Android can also auto-revoke permissions for apps the user has not opened in months.

**Test the denial path as a first-class case.** The most common untested path in any mobile app is "user said no". Run through every permission feature with the permission denied and write down what the user sees. If it is a blank screen or a crash, that is the bug.

### Verify it

**Red flag: a permission requested at launch with no explanation.** Find the actual line in the code and look at what surrounds it. If it sits in a startup effect, in a layout file, or anywhere that runs before the user has tapped the relevant feature, move it. Ask AI to show the user flow from tap to dialog, not just the function call.

**Red flag: more permission than the feature needs.** For each permission in the plan, ask the question "what is the weakest thing that would still make this feature work?" Three specific swaps catch most cases: a system picker instead of camera permission, foreground instead of background location, and approximate instead of precise location. An AI that proposes the strongest option by default is copying an example, not designing for you.

**Red flag: a full-size image upload.** Look for an explicit resize and compress step before the upload call, with actual numbers in it. If the code sends what the picker returned, your users are paying for megabytes you will throw away on display. Ask for the expected file size before and after.

**Red flag: no denial path.** For every permission, ask: "what does the screen show if the user refuses, and what if they refused twice and the dialog will never appear again?" A plan whose answer is "show an error" has not been designed. On Android the correct second answer involves explaining what is lost and offering a route to Settings, plus a fallback that still works.

**Red flag: a missing or meaningless iOS purpose string.** "This app needs camera access" is a rejection waiting to happen and tells the user nothing. The string should name the user benefit: "To take a photo of your order so the kitchen can check it."

**Verify on a real phone, with a real refusal.** Install on a budget device, deny each permission, and use the app. Then grant it and check the file size that actually leaves the phone, using your server logs or your storage dashboard. Then check one stored photo for EXIF GPS data. Those three checks catch nearly everything in this chapter.

## Walkthrough
1. List every device capability your app uses or plans to use. Next to each, write the user-facing feature that needs it.
2. For each one, write down the weakest permission that would still work — and mark the ones where a system picker means you need no permission at all.
3. Remove every permission request from your app's startup path. Every single one. If something breaks, that is the finding.
4. For the most important capability, place the request immediately after the user taps the feature, with one sentence of your own explanation shown first.
5. Write the iOS purpose string for each permission as user-facing copy that names the benefit, not the mechanism. Read them aloud; if they sound like boilerplate, rewrite them.
6. Implement the denial path: what the screen shows on refusal, what it shows when Android will no longer display the dialog, and what still works regardless.
7. For any image upload, add a resize to 1280 pixels on the long edge and compression to roughly 70% quality, before the upload. Strip EXIF metadata in the same step.
8. Measure the result: pick a photo from a real phone camera and record the file size before and after your resize step. Write both numbers down.
9. If you use location, check whether foreground and approximate accuracy are enough. If you are using background location, write a sentence justifying it to a reviewer — or remove it.
10. On a real budget phone, run every capability twice: once refusing the permission, once granting it. Record what the user saw both times.
11. Add every category of data these features collect to your privacy policy and your data safety notes from chapter 5, including photos and location.

## Talk to AI
```prompt
CONTEXT:      React Native app built with Expo, Android first, users in
              India on budget phones with expensive metered data. Product:
              [one sentence].
              Device features we need, with the user-facing purpose of each:
              1. [feature] - because [user benefit]
              2. [feature] - because [user benefit]
              3. [feature] - because [user benefit]
              Where photos go today: [your object storage from M04]
              Typical photo size from our users' cameras: [X] MB

GOAL:         Each feature works with the weakest possible permission, asked
              at the right moment, and still does something useful when the
              user says no.

REQUIREMENTS: 1. For each feature, list every option from weakest to
                 strongest access, including any option that needs no
                 permission at all, and recommend one with the reason
              2. For each recommendation, describe the exact user flow:
                 what the user taps, what our own screen says, then the OS
                 dialog - and prove nothing is requested at app startup
              3. Write the iOS purpose string for each permission as
                 user-facing copy naming the benefit
              4. Define the denial path for each: first refusal, permanent
                 refusal on Android, and a fallback that still works
              5. For any image upload, give me a resize and compress step
                 with specific target dimensions and quality, plus the
                 expected before and after file sizes in kilobytes
              6. Tell me how to strip EXIF metadata and why it matters
              7. List every category of personal data these features cause
                 us to collect, for our privacy policy and data safety form

CONSTRAINTS:  Expo managed workflow, free tiers. Users pay per megabyte.
              Assume permissions can be revoked at any time. Assume no
              background behaviour unless you can justify it to a store
              reviewer in one sentence.

NON-GOALS:    Do not use background location. Do not request contacts. Do
              not add a permission for a feature we have not built yet. Do
              not change our storage provider.

ACCEPTANCE:   - At least one feature is achieved with no permission at all,
                or you explain clearly why that is impossible
              - No permission is requested during app startup
              - Every permission has a written denial path including the
                permanently-denied Android case
              - Every image upload has numeric target dimensions, quality
                and expected file size
              - Every purpose string names a user benefit, not a mechanism

ASK:          Start with the weakest-to-strongest options table for all
              three features and your recommendations. No code until I have
              chosen the permission level for each.
```

## Stuck? Ask AI
```prompt
Explain mobile permissions to someone who has never written an app: what the operating system is actually protecting, why the app cannot grant itself access, and what changes when a user refuses twice on Android. Then ask me to explain back the difference between foreground and background location, and correct me.
```

```prompt
I want users to attach a photo in my app. Walk me through every option from least to most invasive, including options that require no permission at all. For each option, say what the user experience is, what permission is needed, and what breaks if they refuse. Then ask me what my feature actually requires before recommending one.
```

```prompt
A user's photo upload is 4 MB and my app displays it at 120 pixels wide. Explain what is wrong in terms the user would care about, then give me a specific resize and compression target with the reasoning for each number. Tell me how I verify on a real phone that the smaller file is what actually leaves the device.
```

## Quiz

### Q1 · L3
A user needs to attach an existing photo to a form. What permission does your app need?
- (a) Full photo library access
- (b) None, if you launch the system image picker and let the OS handle the choice ✅
- (c) Camera permission
- (d) Scoped storage permission
> The system picker runs in the operating system's own interface, so the user is choosing the file directly. Full camera or library access is only needed when you require a live preview inside your app.

### Q2 · L2
Where should a permission request be placed in your app?
- (a) In the startup code, so it is done early
- (b) Immediately after the user taps the feature, preceded by your own one-sentence explanation ✅
- (c) On the signup screen, grouped with the other permissions
- (d) Only in the settings screen
> Asking in context is what makes the request make sense. On Android a refusal is close to permanent, so the first ask has to be the one the user understands.

### Q3 · L4
Your feature shows the nearest canteen counter while the user is looking at the app. Which location permission is correct?
- (a) Background location, so the app always knows where they are
- (b) Foreground location, and approximate accuracy is likely enough ✅
- (c) Precise background location with a prominent disclosure
- (d) No location permission; ask the user to type their position
> Background location triggers a separate Play Store declaration and reviewer justification, and this feature only needs a position while the app is open. Weakest permission that works is the rule.

### Q4 · L4
An AI gives you photo-upload code that sends exactly what the image picker returned. What is the problem?
- (a) The picker returns a format your server cannot read
- (b) It uploads a multi-megabyte original that your users pay for and you then display at 120 pixels wide ✅
- (c) Pickers do not return file data, only a name
- (d) Nothing; the server should do the resizing
> Resizing on the server still means the user already paid to upload the original. Resize and compress on the device before the upload, and strip EXIF while you are there.

### Q5 · L5
On Android, a user has refused your camera prompt twice. What is the correct design response?
- (a) Keep calling the permission request until it succeeds
- (b) Explain what the feature loses, offer a route to Settings, and provide a fallback that still works ✅
- (c) Show a blocking screen until they grant permission
- (d) Remove the feature from the app for that user
> The system will not show the dialog again, so there is nothing to retry. Denial is a normal state your design has to handle, and the fallback is usually the system picker or manual input.

## Go deeper
- [Android developers: app permissions best practices](https://developer.android.com/training/permissions/usage-notes)
- [Expo: ImagePicker and permissions](https://docs.expo.dev/versions/latest/sdk/imagepicker/)
