---
id: m19-10-gpio-sensors
module: m19-frontier-electives
slug: gpio-sensors-actuators
title: "GPIO, Sensors, Actuators and Power"
order: 10
duration_min: 25
is_free: false
mission: false
volatility: low
last_reviewed: 2026-11-16
---

## Key learnings
- Explain what a pin is doing when it reads versus when it drives
- Say why a motor or relay is never driven straight from a logic pin
- Treat sensor readings as claims that need calibration and plausibility checks
- Build a power budget from wake interval, current and battery capacity
- Spot an AI hardware design with no power budget and no common ground

## Lesson

### The pain

The fridge sensor from last chapter is built, and it lies. On the bench it read 6 degrees; in the fridge room it reports 6, then 41, then minus 12, then 6 again. The soil probe in the college's demonstration plot reported perfect moisture for three weeks and then nothing, because its electrodes corroded in wet soil. And when the team added a relay so the node could switch on a small exhaust fan, the board rebooted the instant the fan started, every single time.

Three separate failures, one shared cause: the physical layer was treated as if it were software. A pin was assumed to deliver a clean number. A sensor was assumed to tell the truth. A supply was assumed to be infinite. None of those assumptions survive contact with a fridge room.

This chapter is the layer where your data is born. Everything downstream — the schema in M04, the dashboards in M07, the model in M09 — inherits whatever quality this layer produced, and no amount of cleverness later recovers a reading that was wrong at the pin.

### The idea

**A pin either reads a voltage or drives one; a sensor converts something physical into a voltage or a digital message; an actuator converts current into motion; and power is the budget every one of those spends from.**

Reading is the input direction. The pin reports whether the voltage on it is near the supply or near zero — high or low — or, if the pin can measure, an analog-to-digital converter turns the voltage into a number with a fixed number of steps. Driving is the output direction: the pin connects itself to the supply or to zero, and it can supply only a tiny current, on the order of tens of milliamps.

That current limit is why actuators need a driver. A relay coil, a motor or a pump wants hundreds of milliamps to amperes. The pin's job is to tell a transistor or a relay module what to do; the heavy current comes from its own supply, and the two supplies must share a common ground or nothing works.

### The analogy

The canteen's electrical board. The switches on the wall are inputs: somebody's hand decides their state. The mixer and the exhaust fan are outputs that draw serious current. Everything shares one 15 A line, and when the mixer starts the tube lights visibly dim for a moment — the supply sagged because a large load started suddenly.

That dimming is a brownout, and on a microcontroller it is a reboot. The fan is the mixer, and the board is the tube light.

**Where the analogy breaks** in the way that matters most: the canteen is on mains. Current is effectively unlimited and free, so nobody computes a budget. A field node has a 2,000 mAh cell and must last a season, which turns every milliamp into an engineering decision — the radio's on-time, the sensor's warm-up, how often you wake. On mains you design for function; on a battery you design for sleep, and that inverts the whole shape of the firmware.

### Picture it

```
            ┌─────────────────────────┐
  button ──▶│ input pin   (reads V)   │
            │                         │   3.3 V logic
  sensor ──▶│ bus pins    (I2C/SPI)   │   supply
            │                         │
            │ output pin  (drives V)  │──▶ transistor ─┐
            └─────────────────────────┘                │
                      │                                ▼
                 common ground ───────────────── motor / relay
                                                 own 12 V supply
```

A pin reads or drives a small voltage only; anything that moves gets its current from a separate supply through a driver, and the two supplies must share a ground for the control signal to mean anything.

```
  a reading is a claim, not a fact

  physical thing ──▶ sensor ──▶ voltage ──▶ ADC ──▶ number
                       │           │         │        │
                   drifts,     noise,    limited   still needs
                   self-heats, wiring    steps     a plausibility
                   corrodes    length               check
```

Every stage between the real world and your database adds its own error, which is why a reading must be calibrated against a known reference and then sanity-checked before anyone trusts it.

```
  power budget, worked in words

  awake 2 s per minute   = about 3% of the time
  awake current  80 mA   -> contributes ~2.6 mA on average
  sleep current  0.05 mA -> contributes ~0.05 mA on average
  total average  ~2.7 mA
  2000 mAh battery / 2.7 mA = roughly 740 hours = ~30 days
```

A power budget is four numbers and one division, and running it before building is what separates a node that lasts a month from one that lasts a day.

### Where it lives

M00's four boxes get a fifth thing in front of them: the physical world. This is the only layer in the whole course where your data does not arrive from a user, and that changes your defaults. In M04 a bad value usually came from a user typing wrongly, and validation caught it. Here a bad value comes from a wire that fell off, and it looks exactly like a real value — which is why range checks, rate-of-change checks and a "last heard from" timestamp belong in the schema from day one.

M05's reliability thinking applies directly. A node that reports once a minute will miss readings; the server must tolerate gaps, duplicates and out-of-order arrivals, which is chapter 11's territory. M09's data-quality lesson is the same lesson here: a model trained on a drifting sensor learns the drift.

Everything in this chapter can be done free in a browser. Simulators give you buttons, temperature and humidity sensors, potentiometers, LEDs, servos, relays and a serial monitor, and they will happily show you a floating input reading garbage. What a simulator cannot show you is brownout, electrical noise, corrosion and heat — so where this chapter says a design fails in the field, take that on trust and design for it anyway. A student who owns a board can see all four.

### Words to own

| Term | Plain meaning | Example |
|---|---|---|
| GPIO | A pin you can configure to read or to drive | The pins along a board's edge |
| Digital | Only two states, high or low | A button |
| Analog | A continuous voltage | A light sensor's output |
| ADC | Converts a voltage into a number of fixed steps | 12-bit means 4,096 steps |
| Pull-up / pull-down | A resistor giving a floating input a default state | Why a button reads cleanly |
| Floating input | A pin connected to nothing, reading noise | Random ones and zeros |
| PWM | Switching on and off fast to average a level | Dimming an LED, speeding a motor |
| Driver | The part that switches heavy current for a pin | A transistor or relay module |
| Common ground | Two supplies sharing a zero reference | Without it, signals are meaningless |
| Brownout | Supply sagging below the chip's minimum | A reboot when the motor starts |
| Inrush current | The large spike when a motor first starts | Several times its running current |
| Debounce | Ignoring the contact chatter of a switch | One press reading as one press |
| Calibration | Correcting a sensor against a known reference | Ice water and boiling water |
| I2C / SPI / UART | Standard wiring schemes for talking to sensors | Two wires, four wires, two wires |

### Tradeoffs and gotchas

**Never drive a load from a logic pin.** A pin supplies tens of milliamps. A relay coil, pump or motor wants far more, and the pin or the chip dies — sometimes immediately, sometimes three weeks later in the field, which is worse.

**Floating inputs read garbage.** An unconnected pin picks up noise and reports it as data. Every input needs a defined idle state, from a pull-up or pull-down resistor.

**Mixing 5 V and 3.3 V damages things.** Many sensors are 5 V parts and most modern boards are 3.3 V. Feeding 5 V into a 3.3 V pin is a slow way to kill a board, and it needs a level shifter rather than optimism.

**Sensors drift, self-heat and corrode.** A temperature sensor near its own power regulator reads high. A cheap humidity sensor drifts over months. Bare-electrode soil probes corrode in weeks; capacitive ones last. Plan recalibration into the product, not into the hope.

**Deep sleep is the only real battery strategy.** Not a smaller processor, not a bigger battery. Sleep, wake, read, send, sleep. And remember the radio: a sleeping processor with a listening radio is not asleep.

**Solar is not free power.** A panel in Nashik through a cloudy monsoon week delivers a fraction of its rating, so a solar node needs a battery sized for the worst week and a design that degrades gracefully when it runs short.

### Verify it

**Red flag: a relay or motor connected straight to a pin.** The first thing to check in any AI-produced wiring description. Look for a transistor, a driver module or an explicit "relay module with its own supply". If the load's current is never mentioned, the design has not been thought about.

**Red flag: no common ground.** Two supplies and no statement that their grounds are joined. Ask directly: where do the two grounds meet? A design that cannot answer will behave randomly.

**Red flag: no power budget.** Same as last chapter, now at the pin level. Demand wake interval, awake duration, current awake, current asleep, battery capacity, and the resulting life. Six numbers, one answer.

**Red flag: 5 V assumed on a 3.3 V board.** Ask what the logic voltage of the board is and what the sensor expects, and whether a level shifter is needed. An AI will often answer in generic 5 V Arduino terms for a 3.3 V board.

**Red flag: readings used with no calibration or plausibility check.** This is the one that silently ruins a dataset. Ask what the valid range is, what rate of change is physically possible, and what the system does when a reading falls outside it. A fridge in Nashik reporting minus forty degrees is a disconnected wire, and your code should say so rather than storing it.

## Walkthrough
1. In a free in-browser simulator, place a board and a push button wired to an input pin with no resistor at all. Print the pin's value continuously and watch it when nothing is pressed.
2. Record what you saw. Then enable the pin's internal pull-up and watch again. Write down the difference in one sentence.
3. Press and release the button several times while printing every change. Count how many changes you get per press, and write down what that behaviour is called.
4. Add a temperature and humidity sensor on a two-wire bus. Print readings once a second and record five of them, noting how much they vary when nothing physical has changed.
5. Write down a valid range and a maximum believable rate of change for a canteen fridge. Then make the simulator report a value outside that range and confirm you could detect it.
6. Add an LED on an output pin and drive it with PWM. Sweep the duty cycle from low to high and describe in one line what duty cycle is doing to the brightness.
7. Add a relay module and a motor in the simulator. Note in the wiring where the motor's current comes from, where the pin's signal goes, and where the two grounds join.
8. Look up the datasheet current for a small relay coil and for your board's maximum current per pin. Write both numbers down and state plainly why the pin cannot drive the coil.
9. Build a full power budget for a node that wakes for two seconds a minute: awake current, sleep current, average current, and days of life on a 2,000 mAh cell. Show each step in words.
10. Repeat step 9 for a node that stays awake continuously, and write one sentence comparing the two results and what that means for the fridge fleet.

## Talk to AI
```prompt
CONTEXT:      I am a student in India with no hardware. I design and test in
              free in-browser simulators. I do not write code; I need to
              judge a hardware design and understand where its data quality
              comes from.

GOAL:         A reviewable design for a battery-powered node that reads a
              fridge temperature once a minute and can also switch a small
              exhaust fan.

REQUIREMENTS: 1. Describe in words, not code, which pins are inputs, which
                 are outputs, and what each one is connected to
              2. State explicitly how the fan is switched: the driver used,
                 where its current comes from, and where the two grounds
                 join
              3. A power budget with six numbers: wake interval, awake
                 duration, awake current, sleep current, battery capacity,
                 and resulting days of life - and say which numbers are
                 assumptions
              4. A data-quality section: the sensor's valid range, the
                 maximum believable rate of change, what a disconnected
                 sensor would look like, and what the node should do in
                 each bad case
              5. A calibration plan: what reference you check against and
                 how often
              6. The three ways you expect this design to fail in a real
                 fridge room and what you would do about each

CONSTRAINTS:  No code and no pin numbers. The board runs on 3.3 V logic;
                say so wherever it matters. Battery powered, no mains at
                the fridge. Must be testable in a free simulator first.

NON-GOALS:    Do not design the cloud side or the dashboard. Do not choose
              a brand of sensor. Do not assume I can buy anything.

ACCEPTANCE:   - You never connect the fan directly to a pin
              - You state where the two grounds join, in one sentence
              - Every number in the power budget has a unit and is marked
                as measured or assumed
              - Your data-quality section says what a minus forty reading
                means and what the node does about it

ASK:          Give me the power budget first, then the data-quality section,
              then the wiring description. If any of my requirements are
              physically impossible together, say so before designing.
```

## Stuck? Ask AI
```prompt
Explain to me, as if I have never seen a computer, why a small control wire cannot also carry the power for a motor. Use one everyday comparison, no formulas, under 200 words, and end with the one thing I should always look for in a wiring description.
```

```prompt
I do not understand floating inputs. Explain in plain words what a pin connected to nothing actually reads and why, then tell me what a pull-up resistor changes. Then ask me what would happen to a button with no pull-up on a windy day near a motor, and discuss my answer.
```

```prompt
Give me five sensor readings from a canteen fridge over one hour: some plausible, some impossible, some suspicious but possible. Ask me to classify each one and say what check would have caught it. Do not tell me your answers until I have done all five with reasons.
```

## Quiz

### Q1 · L2
What does a pull-up resistor do for an input pin?
- (a) Increases the voltage the pin can tolerate
- (b) Gives the pin a defined idle state so it does not read noise ✅
- (c) Converts an analog reading into a digital one
- (d) Protects the pin from excess current when driving a load
> An unconnected pin picks up electrical noise and reports it as data. A pull-up or pull-down holds it at a known level until something actively changes it.

### Q2 · L3
Why must a relay or motor never be driven directly from a logic pin?
- (a) Pins cannot switch on and off fast enough
- (b) The load needs far more current than a pin can supply, so the pin or chip fails ✅
- (c) Relays require an analog signal
- (d) Motors must be driven by PWM only
> A pin supplies tens of milliamps and a coil or motor wants hundreds or thousands. The pin signals a driver, and the heavy current comes from a separate supply.

### Q3 · L4
Your board reboots every time the exhaust fan starts. What is the most likely cause?
- (a) The firmware has a bug in its startup sequence
- (b) The fan's inrush current sags the shared supply below the chip's minimum ✅
- (c) The output pin is configured as an input
- (d) The sensor is returning an invalid reading
> That is a brownout. A motor's starting current is several times its running current, and a shared supply with no separation or buffering drags the chip below its operating voltage.

### Q4 · L4
Your fridge node reports minus forty degrees. What should the system do?
- (a) Store it, since the sensor is the source of truth
- (b) Average it with earlier readings to smooth it out
- (c) Flag it as implausible, raise a fault, and not treat it as a temperature ✅
- (d) Lower the alert threshold to accommodate it
> An impossible reading is a hardware fault report, usually a disconnected wire. Storing or averaging it corrupts the dataset and hides the real problem, which is a broken node.

### Q5 · L4
An AI gives you a design with two power supplies and never mentions grounds. What should you ask?
- (a) Which supply has the higher voltage
- (b) Where the two grounds are joined ✅
- (c) Whether PWM is needed
- (d) What the ADC resolution is
> A control signal is a voltage measured against a reference. Without a shared zero reference between the two supplies, the signal has no defined meaning and the circuit behaves randomly.

## Go deeper
- [SparkFun: Analog to digital conversion](https://learn.sparkfun.com/tutorials/analog-to-digital-conversion)
- [Espressif: ESP32 sleep modes and power consumption](https://docs.espressif.com/projects/esp-idf/en/latest/esp32/api-reference/system/sleep_modes.html)
