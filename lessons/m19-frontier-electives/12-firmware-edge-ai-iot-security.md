---
id: m19-12-firmware-edge-security
module: m19-frontier-electives
slug: firmware-edge-ai-iot-security
title: "Firmware, Flashing, Edge AI and IoT Security"
order: 12
duration_min: 25
is_free: false
mission: true
volatility: medium
last_reviewed: 2026-11-16
---

## Key learnings
- Treat the ability to update a deployed device as the core design requirement
- Name the four IoT security failures that cause almost all real incidents
- Explain signed images, A and B partitions and rollback in plain words
- Decide what inference belongs on the device and what belongs on the server
- Spot a device design with no update path, a shared credential or no end-of-life plan

## Lesson

### The pain

Fifty fridge and cold-store nodes are deployed across a district. Then, in one week, four things go wrong.

A unit-conversion bug makes every node report Fahrenheit as if it were Celsius, so a fridge at 4 degrees reads as safe at 39 — unfixable, because the firmware was flashed over a cable in a lab and the devices are now in fifty buildings. A researcher finds the fleet's MQTT broker on the public internet with one password shared by every node, sitting in a config file anyone can read off a flash chip with a ₹200 clip. Since one node is wired to a cold-store fan, that is a safety problem. And eighteen months later the startup shuts down and fifty devices keep calling a server that is gone.

None of those four is exotic. They are the ordinary way IoT fails, they map onto the published top-ten list of IoT weaknesses, and each is decided at design time by somebody not thinking about year three.

### The idea

**Firmware is the program that *is* the device, and the whole of device engineering is the ability to change it safely, prove it is yours, and survive your own company's death.**

Four properties make that true. You need an update path, or every bug you ship is permanent. Updates must be signed, or your update path is an attacker's installation path. You need two firmware slots and automatic rollback, or a bad update bricks the fleet. And you need per-device credentials, or one compromised device is all of them.

Edge AI sits on top and inherits all four. Running a small model on the device saves bandwidth and keeps raw data local — and it makes your model firmware, so it drifts and can improve only through the same update path.

### The analogy

Two things in the canteen. The staff rota on the whiteboard can be changed by anyone walking past with a marker. The vending machine's control card was welded in at the factory: changing its behaviour needs somebody with a key and a replacement card to visit it. Firmware is the welded card, and an over-the-air update path is what turns it, carefully, into something closer to the whiteboard.

**Where the analogy breaks**: you can walk to the vending machine. It is forty feet away. You cannot walk to fifty nodes in fifty buildings across a district. Physical access as a fallback holds for exactly as long as your deployment is one room, and every student project crosses that line without noticing.

### Picture it

```
  a safe over-the-air update

  server ──signed image──▶ device
                             │ 1. check signature against a key
                             │    burned in at manufacture
                             │ 2. write to the INACTIVE slot
                             ▼
     ┌───────────────┬───────────────┐
     │ slot A (live) │ slot B (new)  │
     └───────────────┴───────────────┘
                             │ 3. boot slot B once
                             │ 4. B must report "I am healthy"
                             │    before the switch is made
                             │    permanent
                             ▼
              no report ──▶ roll back to slot A
```

Signature check, inactive slot, trial boot and automatic rollback are the four steps that make remote updates survivable, and skipping any one of them turns the update path into the fleet's biggest risk.

```
  the four failures behind almost every IoT incident

  1. default or shared credentials  ──▶ one password, whole fleet
  2. unencrypted transport          ──▶ plain MQTT, readable keys
  3. no update path                 ──▶ every bug is permanent
  4. no end of life                 ──▶ device outlives the company
```

These four are the ordinary causes of real incidents rather than exotic attacks, and all four are design decisions made long before deployment.

```
  where does the model run?

  on device (edge)              on server
  ─────────────────             ─────────────────
  no bandwidth needed           retrain any time
  raw data stays local          big models possible
  works offline                 one place to fix
  but: model is firmware        but: data leaves the site
       drifts in place               needs connectivity
       tiny memory budget            costs per inference
```

Choosing where inference runs is mostly a choice between keeping data local and being able to improve the model, and a device that cannot be updated has effectively frozen its model forever.

### Where it lives

M06 is the direct parent: flashing a board is the build-and-deploy pipeline you know, except the target has no shell and no human to roll it back. M07 is its hardest case, because flash can be read physically — so credentials must be provisioned per device, ideally into a secure element the firmware can use but not export. And a device that can switch a fan or open a gate is an M10 agent on a physical system, so the permission question becomes "what is the worst this device can do to a person" — bounded in hardware, not only software.

Regulation moves here, which is why this chapter is marked volatile. India's DPDP Act governs the personal data a device collects, and several jurisdictions now require a stated support period and ban default passwords on consumer products. Verify the current position rather than this paragraph. Everything here can be built in a free simulator.

### Words to own

| Term | Plain meaning | Example |
|---|---|---|
| Firmware | The program that is the device's behaviour | What you flash |
| Bootloader | The small program that starts the firmware | Decides which slot boots |
| Flashing | Writing firmware onto the device | Over a cable, in a lab |
| OTA | Over-the-air; updating remotely | The thing you must build in |
| A/B partitions | Two firmware slots so one stays known-good | Survive a bad update |
| Rollback | Returning to the previous working firmware | Automatic, on failed health check |
| Secure boot | Refusing to run firmware that is not signed | Blocks attacker images |
| Secure element | A chip holding a key it will never reveal | Per-device identity |
| Provisioning | Giving each device its own identity at manufacture | One credential per node |
| TinyML | Machine learning small enough for a microcontroller | Keyword spotting, anomaly detection |
| Quantization | Shrinking a model by using smaller numbers | M09's technique, applied here |
| Model drift | The world changing out from under a frozen model | A sensor ageing |
| End of life | The stated date after which you stop supporting it | Must be decided before launch |

### Tradeoffs and gotchas

**Your update path is an attack path.** Unsigned updates are worse than no updates, because anyone who can reach the device then owns it permanently. Signature verification against a key the firmware cannot overwrite is not optional.

**The signing key is the fleet.** Lose it and you can never update again; leak it and anyone can. Treat it like a production database credential, with one named owner.

**Per-device credentials cost real effort.** Someone must generate, record and install a unique identity for every unit. It is tedious, and it is the highest-value security decision you will make.

**LoRa cannot carry an update.** A LoRa-only fleet has no over-the-air path at all, which must be a conscious decision rather than a discovery in year two.

**Devices outlive companies.** Decide now what happens when you stop operating: standalone firmware, a documented way to repoint the device, or at minimum a published statement.

### Verify it

**Red flag: no update path.** The first question for any device design. How does version two reach a deployed unit, who authorises it, and what happens if it crashes on boot? A design with no answer has decided every future bug is permanent.

**Red flag: default or shared credentials.** Ask whether each device has its own identity and where it comes from. One password for the fleet is the commonest cause of real IoT compromise, and it makes revoking one node impossible.

**Red flag: unencrypted transport.** Ask which port, whether TLS, and how the server's identity is verified. Plain MQTT means credentials and readings are readable on the path.

**Red flag: secrets in the firmware image.** Ask what an attacker learns from reading one device's flash chip. A Wi-Fi password, an API key or the fleet credential breaks the whole fleet, not one unit.

**Red flag: no end-of-life statement.** Ask what the device does in year five if your servers are gone. Silence is the failure that outlasts the company.

## Walkthrough
1. In a free in-browser simulator, build a node that reads a sensor and prints a version string at startup. Note the version.
2. Change the version string and re-run. Write down how the new program reached the simulated device, and then write down how it would reach a device in a cold store 40 km away.
3. List every secret your node would need in the real world: Wi-Fi password, broker address, broker credential, server certificate. Write down where each one would live.
4. For each secret, write one line on what an attacker learns by reading the flash chip off one unit, assuming they have physical access and a cheap clip.
5. Redesign the credential scheme so that compromising one device does not compromise the other forty-nine. Write it in three lines.
6. Sketch the over-the-air update flow in four steps: signature check, which slot, trial boot, health report. Write what happens at each step if it fails.
7. Write down who in your team holds the signing key, where it is stored, and what happens if that person leaves. If you cannot answer, that is the finding.
8. Decide what inference, if any, runs on the device. Write the memory budget in kilobytes, what the model decides, and how a better model would reach the fleet.
9. Write an end-of-life statement in three sentences: the supported-until date, what the device does afterwards, and how an owner could point it at a different server.
10. Finish by scoring your design against the four failures: shared credentials, unencrypted transport, no update path, no end of life. Mark each as pass or fail honestly.

## Talk to AI
```prompt
CONTEXT:      I am a student founder in India. I do not write code. I am
              reviewing a device design, not building it. The device: a
              battery-powered node that reports a cold-store temperature
              over Wi-Fi to an MQTT broker, and can switch a small exhaust
              fan. Fifty units, fifty buildings, across one district. No
              physical access after installation without a day's travel.

GOAL:         A security and lifecycle review I could defend to a reviewer,
              with every finding stated as pass or fail.

REQUIREMENTS: 1. Score the design against the four common IoT failures:
                 default or shared credentials, unencrypted transport, no
                 update path, no end-of-life plan. Pass or fail each, with
                 one line of reasoning
              2. Describe a safe over-the-air update flow in plain words:
                 signing, which slot is written, trial boot, health check,
                 automatic rollback - and what happens when each step fails
              3. List every secret the device holds, where it lives, and
                 what an attacker learns from physically reading one
                 device's flash
              4. A per-device credential scheme: where identities come
                 from, who records them, how one node is revoked
              5. The extra consideration because this device can actuate a
                 fan: what is the worst physical outcome, and how is it
                 bounded outside software
              6. An end-of-life plan in three sentences
              7. What changes if the fleet is on LoRa instead of Wi-Fi

CONSTRAINTS:  No code. Assume no physical access after installation. Assume
              I have no budget for custom hardware. Treat Indian data
              protection rules and any default-password regulations as
              things I must verify against current law.

NON-GOALS:    Do not design the dashboard. Do not pick a cloud vendor. Do
              not propose custom cryptography.

ACCEPTANCE:   - Every one of the four failures has an explicit pass or fail
              - Your update flow says what happens when the health check
                never arrives
              - You state plainly what an attacker gets from one device's
                flash chip
              - Your actuation answer names a limit that holds even if the
                firmware is fully compromised

ASK:          Give me the four-failure scorecard first, unhedged. Then the
              update flow. Then everything else.
```

## Mission

**Objective.** Produce a device security and lifecycle review for your own product — or, if your product has no device, for the longest-lived artefact you ship — scored against the four failures, with a written plan for each failure you found.

If your startup has or could plausibly have a physical device, review that. If it is purely software, run the identical review against the closest thing you ship that outlives your attention and cannot easily be fixed in place: a mobile app build with no forced-update path, a kiosk or shared tablet, a browser extension, or an on-premise install at a customer. The failure shape is the same — something you shipped is still running, you cannot reach it, and it holds a credential.

**Steps.**
1. **Frame** — name the artefact you are reviewing in one sentence, and state how long it will plausibly still be running after you stop paying attention to it.
2. **Inventory the secrets** — list every credential, key, endpoint and certificate the artefact holds, and where each one lives.
3. **Attacker's view** — for each secret, write what someone with full physical or file access to one copy learns, and whether that compromises the rest of the fleet.
4. **Update path** — describe exactly how version two reaches a deployed copy: who authorises it, how authenticity is checked, what happens if it fails to start, and whether anyone can be forced to take it.
5. **Brief** — use the Talk to AI prompt with your real artefact. Record its four-failure scorecard.
6. **Score honestly** — mark your own design pass or fail on shared credentials, unencrypted transport, update path, and end of life. A fail you have written down is worth more than a pass you assumed.
7. **Actuation check** — if your artefact can cause any physical or financial action, write the worst outcome and the limit that holds even if the software is fully compromised. If it cannot, say so explicitly.
8. **Document** — write the end-of-life statement and the decision-log entry: the one failure you will fix first, who owns the signing or release key, and the date you will revisit.

**Upload.**
- Your secrets inventory with the attacker's-view column → attached file or post caption
- Your four-failure scorecard, with at least one honest fail if one exists
- Your update-path description including the failure behaviour at each step
- The AI conversation including its scorecard → share link in `ai_chat_url`
- Your end-of-life statement, three sentences
- Your decision-log entry, 150 words maximum
- Reflection, 150 words max → the failure you did not expect to find, and one thing the AI got wrong

**Self-check.** Tick every box before you submit.
- [ ] I named the artefact and how long it will still be running after I stop maintaining it
- [ ] Every secret in my inventory has a location and an attacker's-view line
- [ ] I said explicitly whether one compromised copy compromises all of them
- [ ] My update path says who authorises a release and how authenticity is checked
- [ ] My update path says what happens when the new version fails to start
- [ ] I scored all four failures as pass or fail, not as "partly"
- [ ] I named one person who holds the signing or release key
- [ ] My end-of-life statement says what happens when my servers are gone
- [ ] If my artefact can act physically or financially, I named a limit outside software
- [ ] My reflection names something specific the AI got wrong

**Stretch.** Take the same artefact and write 150 words on what would change if it ran a model on the device rather than calling a server: what you gain in privacy and bandwidth, and what you lose in your ability to fix a mistake.

## Stuck? Ask AI
```prompt
Explain to me, as if I have never seen a computer, why being able to change the program inside a device after it has been installed is both the most useful thing and the most dangerous thing about it. Use one everyday comparison, no code, under 200 words, and end with the single safeguard I should always look for.
```

```prompt
I do not understand why two firmware slots are needed. Walk me through what happens when an update is installed into the only slot and then crashes on boot, and then the same situation with two slots and an automatic rollback. Then ask me what the device must send for the new version to become permanent, and wait for my answer.
```

```prompt
Give me five device designs with exactly one serious security problem each - one with a shared password, one with unencrypted transport, one with a secret in the firmware image, one with no update path, one with no end-of-life plan. Ask me to find the problem in each. Give hints rather than answers if I get stuck.
```

## Quiz

### Q1 · L3
Why are unsigned over-the-air updates worse than having no update path at all?
- (a) They use more bandwidth than signed updates
- (b) They let anyone who can reach the device install their own firmware permanently ✅
- (c) They cannot be rolled back
- (d) They require two partitions
> An update path without signature verification is an installation path for an attacker. Secure boot checking against a key the firmware cannot overwrite is what makes remote updates safe.

### Q2 · L3
What does an attacker gain from physically reading the flash chip of one device in a fifty-unit fleet that shares one credential?
- (a) Access to that one device only
- (b) Access to the whole fleet and the broker ✅
- (c) Nothing, since flash contents are compiled
- (d) The ability to read the device's sensor but not to publish
> A shared credential means one device's secrets are every device's secrets, and it also means you cannot revoke a single compromised node. Per-device identities are the fix.

### Q3 · L4
Your fleet runs on LoRa only. What is the most important consequence for firmware?
- (a) Updates will be slow but possible
- (b) There is effectively no over-the-air update path, so that must be a conscious decision with a plan ✅
- (c) Firmware must be smaller than on Wi-Fi
- (d) Signatures cannot be verified over LoRa
> Tens of bytes a few times an hour cannot carry a firmware image. A LoRa-only fleet needs either a second radio for updates or an explicit, documented acceptance that bugs are permanent.

### Q4 · L4
An update boots on a device and then crashes repeatedly. With a correct design, what happens?
- (a) The device stays crashed until someone visits it
- (b) The bootloader returns to the previous slot because no health report arrived ✅
- (c) The server re-sends the same image automatically
- (d) The device erases both slots and waits
> The trial boot plus health check is what makes the rollback automatic. Without it, one bad release bricks every device at the same moment.

### Q5 · L5
Your startup shuts down in two years and fifty deployed nodes keep calling a server that no longer exists. What should have been decided before launch?
- (a) A longer warranty period
- (b) An end-of-life plan: a supported-until date, standalone behaviour, and a way to repoint the device ✅
- (c) A larger flash chip for future updates
- (d) A faster radio so updates finish sooner
> Devices routinely outlive the companies that made them. A stated support period, useful standalone behaviour and a documented way to point at another server are the honest minimum.

## Go deeper
- [OWASP Internet of Things project: top ten weaknesses](https://owasp.org/www-project-internet-of-things/)
- [Espressif: over-the-air updates, partitions and rollback](https://docs.espressif.com/projects/esp-idf/en/latest/esp32/api-reference/system/ota.html)
