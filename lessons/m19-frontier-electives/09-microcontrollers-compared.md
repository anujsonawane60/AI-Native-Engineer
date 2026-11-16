---
id: m19-09-microcontrollers
module: m19-frontier-electives
slug: microcontrollers-compared
title: "Microcontrollers: Arduino vs ESP32 vs Raspberry Pi"
order: 9
duration_min: 25
is_free: false
mission: false
volatility: low
last_reviewed: 2026-11-16
---

## Key learnings
- Separate a microcontroller from a single-board computer by what it lacks
- Choose a board from power, connectivity and job rather than from popularity
- Explain why a battery sensor and a camera node are different machines
- Name the Indian price and failure realities: cost, power cuts, SD card death
- Spot an AI board recommendation that ignores the power budget

## Lesson

### The pain

The canteen loses forty litres of milk. The deep fridge failed at some point after 9 pm, the staff found out at 6 am, and nobody can say whether the milk sat at 18 degrees for one hour or eight. The manager wants a sensor that reports the temperature and shouts when it drifts.

A student team takes the brief and buys a Raspberry Pi 5 for about ₹8,000, plus a power adapter, plus a memory card. Three weeks later: it works beautifully on the bench, and in the fridge room it has corrupted its memory card twice because the campus power cuts out without warning and the Pi was in the middle of writing. It cannot run on a battery for more than a couple of hours. It takes forty seconds to boot. And the team has spent most of their budget on one node for one fridge, when the canteen has nine fridges and a cold store.

The job needed a ₹350 board the size of a matchbox that wakes up once a minute, reads a number, sends it, and sleeps. The team did not choose the wrong brand. They chose the wrong *class* of machine, and the three classes are genuinely different animals.

### The idea

**There are three classes of small computer, and the dividing lines are an operating system, a radio and a power budget.**

A microcontroller is a chip with a processor, a small amount of memory and a set of pins, running exactly one program with no operating system underneath it. It boots in milliseconds, it never needs patching because there is nothing to patch, it does nothing you did not write, and it can idle on microamps. An Arduino Uno is the teaching example.

A microcontroller with a radio built in is the same thing plus Wi-Fi and Bluetooth. The ESP32 family is the one that reshaped hobby and small-commercial IoT in India, because at a few hundred rupees it gives you a connected sensor node that can genuinely sleep.

A single-board computer is a small Linux machine. A Raspberry Pi has an operating system, a file system, package management, several cores, a camera interface and the ability to run Python, a database and a web server at once. It also wants steady mains power, a graceful shutdown, and a memory card it will eventually wear out.

### The analogy

Three members of canteen staff. The kettle timer does one thing forever, cannot be distracted, costs nothing to keep, and will never learn a second task — that is a microcontroller. The cashier with a phone does one main job and can also talk to the outside world, at the cost of needing to be charged — that is the ESP32. The manager with a laptop can do twenty things, keeps records, runs reports, and also takes lunch breaks, crashes, and needs the electricity to stay on — that is the Raspberry Pi.

**Where the analogy breaks**: every member of staff can improvise. If the fridge makes a strange noise, a human investigates. A microcontroller executes the loop you wrote and nothing else, forever, including when the sensor has been reading minus forty for a week because its wire fell off. There is no judgement anywhere in the device, which is why chapters 10 and 12 spend so much time on how it fails.

### Picture it

```
  Arduino-class MCU      ESP32-class MCU        Raspberry Pi
┌──────────────────┐  ┌──────────────────┐  ┌────────────────┐
│ no OS, 1 program │  │ no OS, 1 program │  │ Linux, many    │
│ no radio         │  │ Wi-Fi + BLE      │  │ programs       │
│ kilobytes of RAM │  │ ~0.5 MB RAM      │  │ gigabytes RAM  │
│ boots instantly  │  │ boots instantly  │  │ boots ~40 s    │
│ months on a cell │  │ months if asleep │  │ hours on a bank│
│ ~₹400            │  │ ~₹300-500        │  │ ₹2,000-8,000+  │
└──────────────────┘  └──────────────────┘  └────────────────┘
   timing, teaching      connected sensors     camera, local
                                               storage, edge AI
```

The three classes differ most in what they lack, and the absence of an operating system is what buys a microcontroller its instant boot and its months of battery life.

```
  pick by the job, not the brand

  reads one number, battery, remote  ──▶ ESP32-class
  precise timing, no network, cheap  ──▶ Arduino-class
  camera, video, local database, ML  ──▶ Raspberry Pi
  mains power unreliable, no UPS     ──▶ avoid the Pi
  needs to sleep for days            ──▶ never the Pi
```

Matching the job to the class settles the choice before any brand comparison starts, and two of these rules rule the Pi out for the canteen's fridge.

### Where it lives

M00 gave you four boxes: client, server, network, data. A device is a new kind of client — one with no user, no screen, and no ability to retry by having someone tap again. Everything you know about unreliable clients from M05 applies harder here.

M06's Linux is directly relevant to one of the three boxes and irrelevant to the other two. A Pi is a Linux machine: you will use a shell, a service manager and package updates, exactly as in M06. A microcontroller has none of those, which removes a whole class of problems and adds a different one — there is no shell to log into when something is wrong, so your only window is what the firmware chooses to report. M09's quantization and small-model work is why a Pi or a modern ESP32 can run inference at all, which chapter 12 covers as edge AI.

Indian cost reality, because it decides what a student team can actually attempt: an ESP32 development board is roughly ₹300 to ₹500, an Arduino Uno clone around ₹400, a Raspberry Pi Zero 2 W around ₹2,000, and a Pi 5 with accessories well past ₹8,000. Sensors are ₹50 to ₹500. Nothing in this module requires buying any of it — free in-browser simulators build the circuit, run the firmware and show you the serial output. A student who already owns a board will get more out of chapters 10 to 12, and a student who owns nothing will still learn every concept.

### Words to own

| Term | Plain meaning | Example |
|---|---|---|
| Microcontroller | A chip running one program with no OS | ESP32, Arduino's AVR |
| Single-board computer | A small complete Linux computer | Raspberry Pi |
| Bare metal | Code running with no operating system under it | Standard microcontroller firmware |
| RTOS | A tiny scheduler for several firmware tasks | What ESP32 firmware often uses |
| Flash | Non-volatile memory holding the program | 4 MB on a typical ESP32 board |
| GPIO | A general-purpose pin, readable or drivable | Chapter 10 |
| Deep sleep | Near-zero power with most of the chip off | Microamps instead of milliamps |
| Duty cycle | The fraction of time the device is awake | Awake 2 s a minute is about 3% |
| Watchdog | A timer that reboots a hung device | Essential in the field |
| Brownout | Supply voltage sagging below the minimum | A reboot mid-write |
| SD card wear | Flash cards failing after many writes | The classic Pi field failure |

### Tradeoffs and gotchas

**A Pi needs a graceful shutdown and India has power cuts.** Pulling power from a running Linux box mid-write corrupts the card. In the field you need a battery backup, a read-only root file system, or a microcontroller instead. This one failure accounts for most dead student IoT projects.

**Battery life is a design parameter, not a result.** A board awake all the time lasts hours. The same board sleeping and waking once a minute lasts months. If a plan does not say how often the device wakes and for how long, it has no battery life.

**There is no shell on a microcontroller.** No logging in, no reading a log file after the fact, no running one more command. Whatever the firmware does not report, you will never know, and chapter 12 is where that becomes a security problem too.

**More capability is more attack surface and more maintenance.** A Pi runs a full operating system that must be patched, which is M06 and M07 arriving in a fridge room. A microcontroller has nothing to patch and also no easy way to be patched.

**Pin numbering lies.** Board labels, chip pin numbers and library numbering disagree, and they differ between board revisions of the same family. This is the single most common source of wasted hours, and it is why you check against the specific board's own documentation rather than a generic answer.

### Verify it

**Red flag: a Raspberry Pi recommended for a battery-powered sensor.** The most common AI error in this area. Ask for the current draw while idle and the expected battery life with the stated battery. If those two numbers are absent, the recommendation is decoration.

**Red flag: no power budget at all.** A correct answer states how often the device wakes, how long it stays awake, how much current it draws awake and asleep, and the resulting life on a named battery. Insist on all four.

**Red flag: no mention of what happens when power is cut.** Ask directly. For a Pi the answer must include card corruption and a mitigation. For a microcontroller it must include what state is lost and whether the next boot recovers.

**Red flag: pin numbers given without naming the exact board.** "Connect the sensor to pin 4" is unusable. Ask which board revision, and whether the number refers to the silkscreen label or the chip's own numbering.

**Red flag: a single node costed and a fleet ignored.** You have nine fridges. Ask for the cost, the provisioning effort and the failure rate across the whole fleet, which is how a ₹8,000 choice becomes a ₹72,000 choice.

## Walkthrough
1. Open a free in-browser microcontroller simulator that supports both ESP32 and Arduino boards. Create a new ESP32 project with nothing but a blinking light.
2. Run it. Note how quickly it starts compared with any computer you own, and write that observation down.
3. Add a simulated temperature and humidity sensor to the ESP32 and print its readings to the serial monitor. Record three readings.
4. Change the delay so the board reports once every two seconds instead of continuously. Write down what fraction of each minute the board is now doing work.
5. Look up the datasheet figures for your chosen board's current draw while active and while in deep sleep. Write both numbers down with their units.
6. Using those two numbers, estimate how long the board would last on a 2,000 mAh battery if it were awake all the time, and then if it woke for two seconds once a minute. Write both answers side by side.
7. Build the same temperature-reading project on a simulated Arduino board. Note what you cannot do on it that you could do on the ESP32, and why.
8. Write a three-row comparison table for the canteen fridge job — Arduino-class, ESP32-class, Raspberry Pi — with columns for cost, battery life, connectivity and what kills it in the field.
9. Price a nine-fridge fleet for each of the three options, including one spare, and write the three totals in rupees.
10. Finish with one sentence choosing a class for the fridge job, naming the single deciding factor, and one sentence saying what would make you change your mind.

## Talk to AI
```prompt
CONTEXT:      I am a student in India with no hardware budget and no board
              yet. I use free in-browser simulators. I do not write code; I
              need to make the right choice and understand why.

GOAL:         Choose the right class of device for a specific job and be
              able to defend the choice.

              The job: report the temperature inside a canteen deep fridge
              once a minute, raise an alert when it goes above a threshold,
              survive campus power cuts, and be repeated across nine
              fridges. There is no reliable mains socket near every fridge.

REQUIREMENTS: 1. Compare an Arduino-class microcontroller, an ESP32-class
                 microcontroller and a Raspberry Pi in a table with rows
                 for operating system, connectivity, typical active and
                 sleep current, realistic battery life, Indian street
                 price, boot time, and the main field failure mode
              2. Recommend one class for this job and name the single
                 deciding factor
              3. Give me a power budget: how often awake, how long awake,
                 current in each state, and estimated life on a named
                 battery capacity
              4. Tell me what happens to each option when power is cut
                 without warning
              5. Cost the full nine-fridge fleet plus one spare for each
                 option, in rupees
              6. Tell me what would have to change for your recommendation
                 to flip to a different class

CONSTRAINTS:  No code. Prices in rupees and clearly marked as approximate.
              Assume I will build and test everything in a free in-browser
              simulator first. Do not assume any mains power at the fridges.

NON-GOALS:    Do not give me any pin numbers. Do not recommend a specific
              shop or brand of board. Do not design the firmware.

ACCEPTANCE:   - Your power budget has all four numbers, with units
              - Your table's field failure column mentions memory card
                corruption for the Pi
              - Your fleet cost has a rupee total for each option
              - You name one deciding factor in a single sentence

ASK:          Give me the table first, then the power budget, then your
              one-sentence recommendation. Do not hedge the recommendation.
```

## Stuck? Ask AI
```prompt
Explain to me, as if I have never seen a computer, the difference between a machine that runs one program forever with no operating system and a small computer that runs Linux. Use one everyday comparison, keep it under 200 words, and end with the one question I should ask before choosing between them.
```

```prompt
I do not understand why sleeping makes such a difference to battery life. Walk me through a device that is awake all the time and the same device awake for two seconds a minute, with made-up but realistic current numbers, and show me the arithmetic in words. Then ask me what would happen if the radio stayed on during sleep.
```

```prompt
Give me five small projects - a soil moisture logger in a field, a doorbell camera, a parking-space counter, a classroom attendance scanner, a cold-storage temperature monitor. For each one, ask me which class of board I would choose and what the deciding factor is. Do not tell me your answers until I have done all five with reasons.
```

## Quiz

### Q1 · L2
What is the defining difference between a microcontroller and a Raspberry Pi?
- (a) The microcontroller is slower
- (b) The microcontroller has no operating system and runs one program ✅
- (c) The Raspberry Pi has no general-purpose pins
- (d) The microcontroller cannot connect to Wi-Fi
> The absent operating system is what gives a microcontroller instant boot, predictable behaviour and sleep currents in microamps. Some microcontrollers, like the ESP32, do have Wi-Fi.

### Q2 · L3
Why is a Raspberry Pi a poor choice for a battery-powered field sensor?
- (a) It cannot read sensors
- (b) It draws far too much current and needs a graceful shutdown ✅
- (c) It has no network connectivity
- (d) Its pins are not reliable
> A Pi's idle draw is measured in hundreds of milliamps, and pulling its power mid-write corrupts the card. Both facts are fatal for an unattended battery node.

### Q3 · L4
An AI recommends a board but gives no current draw, no wake interval and no battery life. What is the correct response?
- (a) Accept it, since board choice is mostly preference
- (b) Ask for the power budget: wake interval, awake duration, current in each state, battery capacity ✅
- (c) Ask for the price in rupees instead
- (d) Ask it to pick a different brand
> Without a power budget there is no design. Those four numbers decide whether the device lasts a day or a season, and they usually decide the board class too.

### Q4 · L4
Your campus loses power without warning several times a week. Which failure should you plan for first on a Pi-based node?
- (a) The Wi-Fi password being forgotten
- (b) The processor overheating
- (c) The memory card corrupting during a write ✅
- (d) The sensor losing calibration
> Abrupt power loss during a file-system write is the classic Pi field death. Mitigations are a battery backup, a read-only root, or choosing a microcontroller instead.

### Q5 · L4
Why is "connect the sensor to pin 4" an unusable instruction?
- (a) Pin 4 is reserved on every board
- (b) Board labels, chip numbering and library numbering disagree and vary by revision ✅
- (c) Sensors must always use pin 1
- (d) Pins cannot be chosen in firmware
> Pin numbering is the most common source of wasted hours in embedded work. Any instruction must name the exact board and say which numbering scheme it uses.

## Go deeper
- [Wokwi: free in-browser ESP32, Arduino and Pi Pico simulator](https://wokwi.com/)
- [Raspberry Pi documentation](https://www.raspberrypi.com/documentation/)
