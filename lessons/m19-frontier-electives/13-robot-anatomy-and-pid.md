---
id: m19-13-robot-anatomy-pid
module: m19-frontier-electives
slug: robot-anatomy-and-pid
title: "Robot Anatomy, Control Loops and PID"
order: 13
duration_min: 15
is_free: false
mission: false
volatility: low
last_reviewed: 2026-11-16
---

## Key learnings
- Describe a robot as sense, decide and act repeated at a fixed rate
- Tell an open loop from a closed loop by what happens when reality disagrees
- Explain the three parts of PID in words, and what each one fixes and breaks
- State why tuning, sample rate and saturation decide whether a loop is stable
- Spot a PID explanation with no mention of tuning, noise or instability

## Lesson

### The pain

The canteen gets an automatic gate for the service entrance, so trolleys can be pushed through without anyone letting go. It behaves in four distinct ways over four weeks.

Week one it slams into the end stop at full speed and cracks a bracket. Week two, at reduced speed, it creeps: twenty seconds to open, and the trolley pusher leans on it. Week three someone turns the response up and it oscillates, hunting past the mark and never settling. Week four it almost works — except that after a heavy trolley holds it half-open for a minute, the moment the trolley is removed the gate flings itself wide, as if it had been saving up.

Those four are the classic behaviours of a feedback controller, and the fourth has a name — integral windup — which is why "just use PID" is not an answer.

### The idea

**A robot is sense, decide, act at a fixed rate; a closed loop measures how wrong it is and corrects; and PID is three reactions to that one error number — how wrong now, how long it has been wrong, how fast it is changing.**

The error is the difference between where you want to be, the setpoint, and where the sensor says you are, the process variable. A closed loop reads that error every cycle and decides again; an open loop ignores the sensor, running the motor three seconds and hoping.

The three reactions. Proportional: push harder the further you are from the target — which alone leaves you short, because near the target the push is too weak to finish. Integral: if you have been short a while, keep adding push — which clears the gap and causes the week-four lunge, because the adding does not stop while the gate is blocked. Derivative: closing in fast, ease off early — which tames overshoot and amplifies sensor noise, because jitter looks like rapid movement.

### The analogy

A canteen worker fills a tall glass from a tap, aiming at a line on the side: open the tap wide while the glass is empty and narrow it as the water rises (proportional), open a little more when the level has sat just below the line because the tap drips slowly (integral), and start closing early when the water is rushing up (derivative).

**Where the analogy breaks**, in three ways that are the engineering content. The worker sees the whole glass and anticipates; a controller sees one number and only reacts. The worker notices the glass is cracked and leaking; a controller pushes against a fault forever, which is windup. And the worker's eyes update continuously, while a controller samples at a fixed rate.

### Picture it

```
  setpoint ──▶( − )──error──▶ controller ──▶ actuator ──▶ plant
                 ▲                            (motor)     (gate)
                 │                                          │
                 └────────── sensor ◀───────────────────────┘
                              (position)

  open loop = the same picture with the bottom arrow deleted
```

A closed loop subtracts the measured position from the desired position and drives the actuator from that difference, and deleting the feedback arrow is exactly what makes a controller open loop.

```
  four responses to the same step command

  slamming     ╱▔▔▔▔▔   hits the stop at speed, no easing off
  creeping     ╱‾‾‾‾‾   arrives far too slowly, or never
  oscillating  ╱╲╱╲╱╲   overshoots, returns, overshoots again
  good         ╱─────   rises briskly, small overshoot, settles
```

The same mechanism produces all four behaviours depending only on how the three reactions are weighted, which is why tuning is the real work rather than a detail.

```
  windup, in one picture

  gate blocked by a trolley for 60 s
  error stays large ──▶ "how long it has been wrong" keeps
                        growing ──▶ accumulated push becomes
                        huge ──▶ trolley removed ──▶ gate
                        lunges
  fix: stop accumulating while the actuator is already maxed
```

Integral windup happens when the controller keeps accumulating a push it cannot deliver, and the cure is to stop accumulating whenever the actuator is already at its limit.

### Where it lives

You have met control loops without the vocabulary. M05's autoscaling is one: setpoint is target utilisation, process variable is measured load, actuator is the instance count, and a badly tuned one oscillates exactly like the week-three gate. So are M13's alerts that fire and clear repeatedly.

In a robot the sensors are encoders, limit switches and inertial units; the actuators are motors and servos; the plant is the thing being moved, with its mass, friction and backlash. Motor loops run at hundreds or thousands of cycles a second, while chapter 14's navigation runs at ten or twenty. Everything here is free in a browser: online PID simulators let you drag the three weights and watch the response curve, and free robot simulators give you a line-follower to detune into oscillation and back.

### Words to own

| Term | Plain meaning | Example |
|---|---|---|
| Setpoint | Where you want to be | Gate fully open |
| Process variable | Where the sensor says you are | Measured gate angle |
| Error | Setpoint minus process variable | 12 degrees short |
| Open loop | Acts without checking the result | Run the motor three seconds |
| Closed loop | Measures the error and corrects | Reads the encoder every cycle |
| Plant | The physical thing being controlled | The gate and its mass |
| Actuator | The part that applies force | The motor |
| Proportional | Push in step with how wrong you are now | The main response |
| Integral | Push more the longer you have been wrong | Removes the last small gap |
| Derivative | Ease off when closing in quickly | Reduces overshoot, adds noise |
| Steady-state error | A gap that never closes | Gate stops 2 degrees short |
| Overshoot | Going past the target before settling | Slamming the end stop |
| Integral windup | Accumulated push that could not be delivered | The week-four lunge |
| Deadband | A zone where no correction is applied | Stops hunting around the target |
| Sample rate | How often the loop runs | 10 times a second, or 1,000 |

### Tradeoffs and gotchas

**Integral is the one that surprises you.** It fixes the last small gap and introduces windup, slow oscillation and a delayed lunge. Any serious implementation limits the accumulation.

**Sample rate and delay destabilise a perfectly tuned loop.** Gains tuned at 200 cycles a second may oscillate violently at 20. Any latency — a slow sensor, a network hop, a model inference — behaves like reacting to the past.

**Tuning is empirical, and it has methods.** The classical procedure raises the proportional response until the system just oscillates, backs it off, then adds derivative and finally integral, recording each step. Named recipes such as Ziegler-Nichols are starting points for a tuning log, not formulas.

**Safety is not part of PID.** A tuned controller will still crush a hand. Limit switches, torque limits, a mechanical clutch and a physical emergency stop are separate protections that must hold when the software has crashed.

### Verify it

**Red flag: a PID explanation with no mention of tuning or instability.** The diagnostic test of this chapter. An answer that presents the three reactions as a recipe, without saying the weights must be found experimentally and that bad weights oscillate, has described half the subject.

**Red flag: gains offered as universal constants.** Three numbers are meaningless without the mass, friction, actuator and sample rate they were found for. Ask what system they came from.

**Red flag: no sample rate.** Ask how often the loop runs and how much delay the sensor adds; a controller specification without a rate is incomplete.

**Red flag: no windup protection.** Ask what happens when the actuator is at maximum and the error persists. No mention of limiting the accumulation means the week-four lunge.

**Red flag: the sensor assumed perfect.** Ask what the measurement noise is, and what the controller does if the sensor fails and reports a constant value.

**Red flag: safety discussed only in software.** Ask which protection still works when the controller has crashed mid-motion.

## Walkthrough
1. Open a free in-browser PID simulator or a simulated line-following robot. Set the integral and derivative weights to zero so only the proportional response is active.
2. Give it a step command and record the response. Note whether it reaches the target, and if not, how far short it stops.
3. Raise the proportional weight gradually until the response just begins to oscillate without settling. Record that value as your oscillation point.
4. Reduce the proportional weight to roughly half that value. Record the response shape, and write down the remaining steady-state gap.
5. Add derivative weight in small steps. Record what happens to the overshoot, and keep increasing until you can see the response becoming jittery.
6. If the simulator lets you add measurement noise, turn it on and note what the derivative term does with it. Write one sentence on the trade-off.
7. Add integral weight in small steps until the steady-state gap closes. Record how long it takes to settle compared with step four.
8. Now hold the system away from its target for a long time — block the simulated gate, or hold the line sensor off the line — then release it. Record the behaviour and name it.
9. Reduce the loop's sample rate by a factor of ten, keeping your tuned weights. Record what happens, and write one sentence explaining why the same weights are no longer safe.
10. Finish with a tuning log of five lines: your oscillation point, your final three weights, the sample rate they belong to, the one behaviour you could not remove, and the one safety protection you would add outside software.

## Talk to AI
```prompt
CONTEXT:      I am a student learning engineering judgement, not writing
              code and not building hardware. I test in free in-browser
              simulators. I want to understand control loops well enough to
              review a design and spot a bad explanation.

GOAL:         Understand PID well enough to diagnose a misbehaving system
              from its symptoms.

REQUIREMENTS: 1. Explain the three parts in plain words - how wrong now,
                 how long it has been wrong, how fast it is changing - and
                 for each one say what it fixes and what it breaks
              2. A symptom table: slamming, stopping short, oscillating,
                 lunging after a long obstruction, and jittering. For each,
                 the likely cause and the first thing to change
              3. Explain integral windup with a concrete physical example
                 and state the standard cure
              4. Explain why sample rate and sensor delay can destabilise a
                 loop that was tuned correctly at a higher rate
              5. A tuning procedure as an ordered list of steps, with what
                 to record at each step
              6. Say plainly which safety protections must exist outside
                 the controller and keep working if the software crashes

CONSTRAINTS:  No formulas, no mathematical notation, no transfer functions,
              no code. Plain English for a second-year student reading in a
              second language. Under 900 words. Do not give me specific gain
              values; explain why specific values would be meaningless
              without my system.

NON-GOALS:    Do not explain control theory mathematics. Do not recommend a
              library or a motor driver. Do not design a robot.

ACCEPTANCE:   - Every row of your symptom table names the first thing to
                change
              - You state explicitly that gains must be found
                experimentally and that bad gains oscillate
              - Your windup explanation names the cure
              - Your safety list contains at least one protection that is
                mechanical rather than software

ASK:          Give me the symptom table first. Then ask me to describe a
              misbehaving system in my own words and diagnose it with me
              rather than for me.
```

## Stuck? Ask AI
```prompt
Explain to me, as if I have never seen a computer, the difference between running a motor for three seconds and hoping, and measuring how wrong you are and correcting every moment. Use one everyday example, no formulas, under 200 words, and end with the one question I should ask about any automatic system.
```

```prompt
I do not understand the integral part. Explain it using someone filling a glass from a dripping tap, then explain what happens if the glass is blocked for a minute, and then ask me what the controller should have done while it was blocked. Wait for my answer before correcting it.
```

```prompt
Give me five misbehaving automatic systems described only by their symptoms - one slamming, one stopping short, one oscillating, one lunging after a delay, one jittering. Ask me which part of the controller is responsible in each and what I would change first. Do not reveal your answers until I have done all five.
```

## Quiz

### Q1 · L2
What is the difference between an open loop and a closed loop?
- (a) A closed loop runs faster
- (b) A closed loop measures the result and corrects from the error ✅
- (c) An open loop has no actuator
- (d) An open loop cannot be tuned
> Feedback is the whole distinction. An open-loop controller commands and hopes; a closed loop reads a sensor each cycle and decides again from how wrong it is.

### Q2 · L3
Your gate consistently stops two degrees short of fully open and stays there. Which part of the controller addresses this?
- (a) Proportional, increased
- (b) Derivative, increased
- (c) Integral, which accumulates while the error persists ✅
- (d) A faster sample rate
> A persistent small gap is steady-state error: near the target the proportional push is too weak to finish. The integral term accumulates until the remaining gap is cleared.

### Q3 · L4
A gate is blocked by a trolley for a minute; when the trolley is removed, it flings itself wide open violently. What happened?
- (a) The proportional gain is too high
- (b) Integral windup: the controller kept accumulating a push it could not deliver ✅
- (c) The derivative term amplified sensor noise
- (d) The sample rate dropped while blocked
> While the actuator is saturated and the error persists, the accumulation grows without limit. The cure is to stop accumulating whenever the actuator is already at its maximum.

### Q4 · L4
An AI gives you three specific gain values for your robot and nothing else. What is the problem?
- (a) Gains should always be set to one
- (b) Gains are meaningless without the mass, friction, actuator and sample rate they were found for ✅
- (c) PID requires four values, not three
- (d) Gains must be computed from a transfer function
> Tuning is empirical and system-specific. Numbers without the system they came from, and without a sample rate, cannot be transferred to your hardware.

### Q5 · L4
You tune a loop at 200 cycles a second and then run the same gains at 20. What should you expect?
- (a) Identical behaviour, since the gains are unchanged
- (b) Smoother motion because there are fewer corrections
- (c) Instability or oscillation, because the controller is reacting to older information ✅
- (d) A larger steady-state error only
> Reducing the rate adds effective delay, and delay destabilises feedback. Gains belong to the sample rate they were found at, which is why a rate must always be stated.

## Go deeper
- [Brett Beauregard: improving the beginner's PID](http://brettbeauregard.com/blog/2011/04/improving-the-beginners-pid-introduction/)
- [MathWorks Tech Talks: understanding PID control](https://www.mathworks.com/videos/series/understanding-pid-control.html)
