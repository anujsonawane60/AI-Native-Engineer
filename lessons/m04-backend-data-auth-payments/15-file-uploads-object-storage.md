---
id: m04-15-file-uploads-object-storage
module: m04-backend-data-auth-payments
slug: file-uploads-object-storage
title: "File Uploads: Object Storage, Buckets and Signed URLs"
order: 15
duration_min: 25
is_free: false
mission: false
volatility: low
last_reviewed: 2026-11-16
---

## Key learnings
- Say why files belong in object storage and never in your database or server disk
- Trace a direct upload with a signed URL and explain who holds what
- Apply chapter 14's permissions to files, where buckets have no row-level checks
- Resize and validate uploads without blowing a serverless timeout
- Verify an upload path by trying to break it with a wrong, huge or hostile file

## Lesson

### The pain

The canteen app lets staff photograph each dish. The first version works like this: the phone uploads the photo to the server, the server saves it to a folder on disk, and the database stores the filename.

Four things go wrong in two weeks.

The server is serverless, so the disk vanishes between invocations. Half the photos disappear and nobody can reproduce it, because on a laptop the disk persists.

The team moves the images into the database as binary columns. Now every query that touches the menu drags megabytes with it, backups take forty minutes, and the free-tier storage limit is reached on day three.

Then a staff member uploads a 12-megapixel photo straight from a phone camera: 6 MB. The upload itself times out at ten seconds on hostel 4G. When one does get through, the student menu loads 6 MB per dish, which on a budget phone on 4G means a blank screen and an eventual reload.

Finally, a student discovers that the image URLs are sequential and public. Photos uploaded for an internal stock-check — including one with a staff member's handwritten notes and phone number — are readable by anyone who guesses the number.

### The idea

**Files live in object storage, not in your database and not on your server's disk; your database stores only a key pointing at them, and access is granted one file at a time with a signed URL.**

**Object storage** is a service built for exactly this: a **bucket** holds **objects**, each identified by a **key** that looks like a path (`dishes/91/main.jpg`). It is cheap, effectively unlimited, and designed to serve a file to a browser directly without your server touching the bytes.

A **signed URL** is a temporary link, cryptographically signed, that grants one specific operation on one specific object for a few minutes. It is how you let a phone upload straight to the bucket without your server relaying the file, and how you let someone read a private file without making the bucket public.

### The analogy

The canteen does not keep crates of supplies behind the serving counter. It keeps them in a storeroom, and the register at the counter notes which shelf each item is on.

Object storage is the storeroom. The database is the register — one line per crate, with a shelf reference, never the crate itself. A register with sacks of rice stuffed between the pages is the "images in the database" mistake, and it is why nobody can carry the register anymore.

A signed URL is a chit the manager writes: "bearer may collect one sack from shelf D4, valid fifteen minutes". The storeroom keeper checks the chit, not the person. The manager never carries the sack himself — he just authorises the trip.

**Where the analogy breaks:** a storeroom has one door and a keeper who sees who walks in. A bucket is reachable from anywhere on the internet, and if you make it public, every object in it is public forever to anyone who guesses a key. There is no keeper. The chit system is not a convenience — it is the only door.

### Picture it

```
 PHONE                YOUR SERVER            BUCKET
   │                       │                    │
   │─ "I want to upload" ─▶│                    │
   │  (name, type, size)   │ checks: logged in? │
   │                       │ role allowed?      │
   │                       │ type allowed?      │
   │                       │ size under limit?  │
   │◀─ signed PUT URL ─────│                    │
   │   valid 5 minutes     │                    │
   │                       │                    │
   │────── the file itself ─────────────────────▶│
   │        (never touches your server)         │
   │                       │                    │
   │─ "done, key is X" ───▶│─ save key in DB ──▶│
   │                       │  queue a resize job│
```

Your server authorises the upload and stores the key, the file travels directly from the phone to the bucket, and the slow work of resizing happens afterwards in a background job.

### Where it lives

Object storage is a fourth box beside M00's client, server, network and data — or more precisely, it is a second kind of data store, one your client talks to directly under supervision.

That directness is the point and the risk. The bytes bypass your server, which is why a 6 MB upload does not hit chapter 5's ten-second timeout. But it also means chapter 14's row-level security does not apply: a bucket has no concept of "rows this student may see". **Permission for files is entirely the signed URL and the bucket's policy**, which is why this chapter sits right after the permissions chapters.

The resize belongs to chapter 16's background jobs. The private-versus-public decision belongs to M07, where leaked documents are an incident and the DPDP Act has opinions about a student's ID photo being publicly readable. In M10, documents you ingest for RAG are also objects in a bucket, with the same access questions.

### Public or private, and the decision you must make per bucket

Two kinds of file, and conflating them is the leak in the opening story.

**Public files** are meant for everyone: a dish photo on a menu that anyone can browse. Make the bucket readable, serve through a CDN, and let the browser cache it. Cheap and fast.

**Private files** are meant for one person or one role: a student's ID card, a payment receipt, an internal stock photo with handwritten notes. These live in a bucket that is **not** publicly readable, and every read goes through a signed URL your server issues after checking the permission matrix from chapter 14.

The mistake is one bucket for both, made public because the dish photos needed it. Then private files are one guessed key away. Use two buckets — or two prefixes with different policies — and decide which one a file goes into at the moment you issue the upload URL.

A related detail: **do not let the client choose the key.** If the phone sends `key: "dishes/91/main.jpg"`, a malicious client sends `key: "students/441/id-card.jpg"` and overwrites someone else's file. Your server generates the key, including a random component, and tells the client what it will be.

### Validating an upload, properly

Four checks, and the order matters because the first three happen before any bytes move.

**Size.** Enforce a maximum when you issue the signed URL, not after the upload. A limit checked afterwards means you have already paid for the bandwidth and the storage.

**Type.** Allow a specific list — `image/jpeg`, `image/png`, `image/webp` — rather than blocking a list. And know that the declared content type is a claim by the client; the real check is inspecting the file's actual bytes after it arrives, which is work for a background job.

**Permission.** Chapter 14's matrix. May this role upload this kind of file at all? Can staff upload dish photos but not replace a student's ID?

**Filename.** Never trust it. It can contain path traversal, Unicode tricks, or 400 characters. Generate your own key and keep the original name as a plain text column if you need to display it.

After the file lands, the real work starts and none of it belongs in a request handler. A 6 MB phone photo must become a sensible web image of about 100 KB — resizing, re-encoding and stripping EXIF metadata, which matters more than it sounds: phone photos carry GPS coordinates, and publishing one with the canteen's exact location embedded is a privacy leak nobody intended.

### Words to own

| Term | Plain meaning | Example |
|---|---|---|
| Object storage | A service for storing files by key | buckets |
| Bucket | A named container for objects | `canteen-public` |
| Key | The object's path-like identifier | `dishes/91/main.jpg` |
| Signed URL | A temporary, signed link for one operation | valid 5 minutes |
| Presigned upload | A signed URL that permits a PUT | direct from phone |
| Public bucket | Anyone with the key can read it | menu photos |
| CDN | A cache near the user serving files fast | M05's territory |
| Content type | The declared kind of file | `image/jpeg` |
| EXIF | Metadata inside a photo, including GPS | strip it |
| Orphan file | An object with no database row | cleanup job |

### Tradeoffs and gotchas

**Never store files in your database.** Binary columns bloat every backup, slow every query that selects them by accident, and burn a free tier in days. The database stores the key.

**Never rely on your server's disk.** On serverless it does not persist; on a scaled server the second instance cannot see what the first wrote. Both failures are invisible with one instance on a laptop.

**Signed URL lifetimes should be short.** Five or ten minutes for an upload, a few minutes for a private read. A link valid for a week will be pasted into a WhatsApp group, and then it is a public link you cannot revoke without moving the object.

**Deleting the row does not delete the file.** Your database and your bucket are separate systems with no transaction between them — chapter 11's point. You need a deliberate decision: delete the object when the row goes, or run a periodic cleanup for orphans. Pick one, or your free tier fills up with photos of dishes that no longer exist.

**The reverse orphan is worse.** A file uploaded and then the database write fails leaves an object nobody knows about. Issue the key, upload, then write the row — and have the cleanup job sweep objects with no row after a day.

**Strip EXIF from anything you publish.** Faces plus locations in a student app is the kind of thing the DPDP Act exists for.

**Resize on the way in, not on the way out.** Serving a 6 MB original and letting the browser scale it down spends the user's data allowance, which on a metered 4G plan is their money. Generate a thumbnail once, store it, serve that.

**Do not let AI build an upload that posts to your server.** It is the default answer and the one the ten-second timeout kills. Ask for a direct-to-bucket upload with a server-issued signed URL.

### Verify it

Six attacks on your own upload path. Every one takes two minutes.

1. **Upload something far too big.** A 50 MB file. It must be refused — ideally at the point of requesting the signed URL, before any bytes travel. If the refusal happens after the upload, you paid for it.
2. **Upload the wrong type.** A PDF, a `.zip`, a text file renamed to `.jpg`. The declared type check should stop the obvious ones; note whether anything inspects the actual bytes afterwards.
3. **Try to choose your own key.** Send a request asking to upload to another user's path. If the server honours a client-supplied key, you can overwrite other people's files.
4. **Guess a private URL.** Take a private file's URL and open it in a browser with no session, in a private window. If it loads, your bucket is public and chapter 14's matrix means nothing for files.
5. **Reuse an expired signed URL.** Get one, wait past its lifetime, use it. It must fail. Then check how long you actually set — if it is hours or days, shorten it.
6. **Check what the student actually downloads.** Open the menu on a throttled connection and look at the image sizes in the Network tab. If you are serving the 6 MB original, the resize is not happening, whatever the code says.

Red flags: an upload endpoint that receives the file through your server; files in a database column; one bucket holding both public photos and private documents; a client-supplied key used directly; signed URLs lasting days; resizing inside the request handler; no plan for objects when rows are deleted; images served at original resolution.

## Walkthrough
1. List every kind of file your product will store, and mark each one public or private. Be strict: anything containing a person's identity, a document or a receipt is private.
2. Decide your buckets or prefixes from that list — at least one public and one private — and write down which kinds of file go where.
3. For each file type, write the limits: maximum size, allowed content types, and which role may upload it. Four lines each, from chapter 14's matrix.
4. Brief AI for a direct-to-bucket upload with a server-issued signed URL, explicitly forbidding any design that routes the file through your server.
5. Check that the key is generated by your server with a random component, and that nothing in the request lets the client choose it.
6. Upload a real photo straight from a phone camera, unresized. Note its size in megabytes. This is what your users will actually send.
7. Confirm the resize happens outside the request: find where the job is queued, and check the record is updated when it finishes. Note what the UI shows while it is still processing.
8. Open the menu on a throttled Slow 4G connection and read the image sizes in the Network tab. Compare them against the original from step 6.
9. Run all six attacks from the Verify section and record the result of each: oversized file, wrong type, client-chosen key, guessed private URL, expired signed URL, and the delivered image size.
10. Check EXIF data on a published image using any metadata viewer. If GPS coordinates are present, that is a leak to fix before the demo.
11. Delete a record that has a file attached, then look in the bucket. Record in your decision log: your buckets and their visibility, your size and type limits per file kind, your signed URL lifetimes, and your orphan cleanup decision.

## Talk to AI
```prompt
CONTEXT:      Three-person student team, free tiers only. Server runs as
              serverless functions with roughly a 10-second timeout and no
              persistent disk. Users are Indian college students on budget
              Android phones and slow 4G; their data allowance is their own
              money.
              Product: [one sentence]
              Object storage we have available: [platform, or "undecided"]

              FILES WE NEED TO STORE — kind, who uploads it, who may read
              it, and public or private:
              [list each one, e.g. "dish photo, staff uploads, anyone
              reads, public" / "student ID card, student uploads, only that
              student and admin read, private"]

              OUR PERMISSION MATRIX for these actions:
              [paste the relevant rows]

GOAL:         An upload and delivery path that never routes file bytes
              through my server, keeps private files genuinely private, and
              serves images small enough for a 4G phone.

REQUIREMENTS: 1. Design a direct-to-bucket upload using a server-issued
                 signed URL. Describe the full sequence and say, at each
                 step, who holds what
              2. My server must generate the object key with a random
                 component; tell me exactly what breaks if the client
                 supplies it instead
              3. Specify per file kind: maximum size, allowed content
                 types, which role may upload, and which bucket or prefix
                 it lands in
              4. Enforce size and type BEFORE any bytes move, when the
                 signed URL is issued. Tell me what is still only
                 checkable after arrival and where that check runs
              5. Separate public and private storage and tell me exactly
                 how a private read is authorised, tying it to my matrix
              6. Choose signed URL lifetimes and justify each one; assume a
                 link will be pasted into a WhatsApp group
              7. Put resizing and EXIF stripping in a background job, not
                 the request. Tell me what the UI shows while a file is
                 still processing
              8. Give me the orphan problem both ways — object without a
                 row, row without an object — and a policy for each
              9. Give me a six-attack test plan I run myself: oversized
                 file, wrong type, client-chosen key, guessed private URL,
                 expired signed URL, and checking the delivered image size

CONSTRAINTS:  Free tier only. Nothing may rely on server disk. No file
              bytes through my server. Serve images under 150 KB for list
              views. No paid image service.

NON-GOALS:    Do not build an image editor. No video. No virus scanning
              service. Do not change my database schema beyond the columns
              a file reference needs.

ACCEPTANCE:   - A 50 MB upload is refused before any bytes travel
              - A private file's URL opened with no session fails
              - A client-supplied key is ignored or rejected
              - An expired signed URL fails
              - The menu list view serves images under 150 KB
              - Published images carry no GPS metadata

ASK:          First tell me which of my listed files you think I have
              misclassified as public, and why. Then give me the design.
              No code until I confirm the public-private split.
```

## Stuck? Ask AI
```prompt
Explain what a signed URL is using a canteen storeroom and a handwritten chit as the analogy. Then tell me what an attacker can do with a signed URL that is valid for seven days, and what they can do with one valid for five minutes.
```

```prompt
Explain why storing uploaded images in my database is a bad idea, with specific consequences for backups, query speed and my free-tier limits. Then explain why saving them to my server's disk fails on serverless hosting, and why I would not notice either problem while developing on my laptop.
```

```prompt
A staff member uploads a 6 MB photo from a phone camera and my function times out. Walk me through, step by step, what a correct upload path does instead — including where the resize happens and what the user sees while it is still processing. Do not write code.
```

## Quiz

### Q1 · L2
Where should an uploaded image be stored?
- (a) As a binary column in the database, so backups include it
- (b) In object storage, with only the key saved in the database ✅
- (c) On the server's disk, in a folder served statically
- (d) In the user's browser storage until they need it
> Binary columns bloat backups and burn free tiers; server disk does not persist on serverless and is invisible to a second instance. The database stores a key, nothing more.

### Q2 · L3
Why does a direct-to-bucket upload with a signed URL avoid the serverless timeout?
- (a) Signed URLs compress the file before it is sent
- (b) The bytes travel from the phone to the bucket without passing through your server ✅
- (c) The upload runs in a background job
- (d) Object storage has no timeout limits
> Your server does two short things — authorise and record — while the slow transfer happens between the phone and the bucket, which is why a 6 MB photo on 4G never touches your ten-second limit.

### Q3 · L4
Your app lets the client specify the object key for its upload. What is the risk?
- (a) The key may contain characters the bucket rejects
- (b) A malicious client can write to another user's path and overwrite their file ✅
- (c) The file will be stored in the wrong bucket region
- (d) Keys will collide between users with the same filename
> The key is a permission decision. Your server generates it, with a random component, exactly as identity comes from the session and never from the request.

### Q4 · L4
You put public menu photos and private student ID cards in one bucket, made public so the menu works. What follows?
- (a) Nothing, as long as the ID card keys are long and random
- (b) Every private file is readable by anyone who obtains or guesses its key ✅
- (c) The private files are protected by row-level security on the bucket
- (d) Reads of private files will be refused because they have no signed URL
> Buckets have no row-level security. Public means public, and chapter 14's matrix has no effect on files — two buckets with different policies, and the decision made when the upload URL is issued.

### Q5 · L5
The menu loads slowly on 4G. The Network tab shows images of 6 MB each. What does this tell you, regardless of what the code says?
- (a) The CDN cache is cold
- (b) The resize job is not running, or the original is being served instead of the thumbnail ✅
- (c) The signed URLs have expired and are being reissued
- (d) The bucket is in a distant region
> Checking what the user actually downloads is the only honest verification. A resize that exists in code but is not in the delivery path is spending your users' data allowance on their own money.

## Go deeper
- [MDN: File API and handling uploads in the browser](https://developer.mozilla.org/en-US/docs/Web/API/File_API)
- [AWS S3 docs: presigned URLs](https://docs.aws.amazon.com/AmazonS3/latest/userguide/using-presigned-url.html)
