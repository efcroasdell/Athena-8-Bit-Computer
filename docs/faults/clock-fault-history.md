# Athena Clock Module — Fault and Diagnostic History

Status: **historical engineering record**

This file records faults, diagnostic dead ends, damaged parts, wiring errors and corrective reasoning associated with the Athena clock work. The purpose is to preserve what was learned rather than merely documenting the final working state.

## PSU over-voltage incident

A major earlier failure occurred when a bench PSU memory button adjacent to the power control was pressed unintentionally. The stored setting applied approximately **30 V at a 5 A current limit** to a TTL breadboard intended for roughly 5 V operation.

Observed consequence: multiple devices failed or were suspected damaged, including logic used in the earlier clock implementation.

Engineering significance:

- TTL parts must never be exposed to supply voltages remotely close to 30 V.
- Current limiting does not protect logic from destructive over-voltage if the limit is still far above the device's normal operating current.
- Bench-supply stored presets are an operational hazard when their controls can be confused with power controls.
- After such an event, subsequent faults must not be diagnosed under the assumption that every previously fitted IC remains healthy.

This incident is part of the clock's history but is not representative of the current verified 556/SN7400/SN74LS157 build.

## Floating RESET on earlier timer build

An earlier 555-based build had RESET left floating.

Diagnosis: a floating asynchronous control input can produce unpredictable operation because its logic level is not defined.

Correction: RESET was tied to the required inactive HIGH state.

Lesson retained in the current 556 implementation: both reset inputs used for normal operation are deliberately held HIGH.

## Earlier monostable wiring problems

During the earlier 555/556 learning work, the monostable output showed incorrect behaviour when the trigger and timing network were miswired. Symptoms included stuck indicator states and unexpected output polarity/levels.

The diagnostic method developed from this was to separate the trigger node from the rest of the switch wiring and verify the electrical requirements directly:

1. establish a defined idle level;
2. force the trigger state manually;
3. measure the trigger pin itself;
4. only then reintroduce the mechanical switch.

This method was reused successfully in the current pin-8 verification.

## STEP switch diagnosis — current 556 build

### Symptom

The STEP trigger initially behaved with the wrong polarity when the push-button wiring was introduced. Pin 8 could appear LOW at rest and HIGH on button action, contrary to the intended active-LOW trigger arrangement.

### Isolation

The switch was removed from the reasoning path and pin 8 was reduced to the simplest possible circuit:

- pin 8 connected to VCC through 10 kΩ;
- nothing else connected to pin 8.

Measured result: pin 8 was HIGH.

A jumper was then used to connect pin 8 directly to GND.

Measured result: pin 8 went LOW.

This verified the pull-up and trigger node independently of the switch.

### Switch correction

The STEP control was then wired so that:

- released = pin 8 HIGH;
- pressed = pin 8 connected to GND and therefore LOW.

Scope verification confirmed the intended HIGH → LOW → HIGH trigger waveform.

### Lesson

The important diagnostic result was not merely the final wiring. The trigger input was proved independently before the switch was treated as part of the circuit. This prevented a mechanical-contact assumption from being mistaken for an IC fault.

## Timer 2 timing resistor error — pin 12 investigation

### Symptom

During systematic verification of Timer 2, pin 12 (THRESHOLD2) did not show the expected monostable timing-capacitor ramp. The trace appeared square-like rather than showing an exponential rise towards approximately two-thirds of VCC followed by rapid discharge.

Changing the timing capacitor from **1 µF to 10 µF** did not initially produce the expected visible ramp. Removing the capacitor also failed to produce behaviour consistent with the assumed 1 MΩ timing network.

### Isolation

With power removed, continuity checks established that the physical timing node was correctly connected:

- capacitor positive node → pin 12: continuity confirmed;
- capacitor positive node → pin 13: continuity confirmed.

The resistance from the joined pins-12/13 timing node to +5 V was then measured directly.

Expected: approximately **1 MΩ**.

Measured: approximately **0.9943 kΩ**, about **994 Ω**.

A roughly **1 kΩ** resistor had been fitted where the monostable required **1 MΩ**.

### Causal chain

With 1 kΩ and 1 µF:

`RC ≈ 1 ms`

and the nominal monostable interval is only about:

`1.1 × RC ≈ 1.1 ms`.

At the scope timebase previously in use, approximately **100 ms/div**, that charging interval was effectively instantaneous on screen and therefore appeared as a square transition rather than a visible capacitor ramp.

The incorrect resistor was replaced with **1 MΩ**.

The expected pin-12 waveform then appeared immediately:

- timing node near 0 V at rest;
- exponential rise after trigger;
- peak around **3.20 V**, consistent with the upper threshold region;
- rapid return to approximately 0 V when the threshold comparator reset the latch and the discharge transistor turned on.

### Lesson

When a measured waveform contradicts a well-established model, verify the physical component values and node connectivity before inventing a new explanation for the behaviour.

Informal reminder retained from the session: **“pay attention, Eddie!”**

## Breadboard-image interpretation caution

During diagnosis, visual inspection of photographs repeatedly risked introducing false assumptions about which rail or switch terminal was electrically connected.

Project rule established from this work:

**Every photograph is fresh evidence.**

A new photograph must be traced from visible endpoint to visible endpoint. Expected topology must not be substituted for what is actually visible. Where continuity cannot be established visually, it must be measured rather than guessed.

## Pin 9 switching overshoot

During triggered Timer 2 operation, the pin-9 output showed a small transient overshoot at switching edges.

Current status: **observed, not yet isolated**.

The steady output levels remain valid:

- LOW ≈ 0 V
- HIGH ≈ 4.32 V

Possible sources include the real 556 output edge, breadboard/interconnect parasitics, and oscilloscope probe-ground inductance. No fault has been assigned because the source has not yet been distinguished experimentally.

## Historical 555 implementation

The clock began as a Ben Eater-style implementation using individual 555 timers for astable, monostable and mode-control functions.

That work remains educationally relevant because it established:

- the astable timing-capacitor waveform;
- approximate one-third/two-thirds VCC thresholds;
- discharge-pin switching;
- manual monostable timing;
- the distinction between astable, monostable and bistable behaviour.

The individual-555 implementation has been superseded by the current dual-556 plus TTL/multiplexer architecture, but it should be archived rather than discarded.

## Status

No unresolved fault currently prevents continued systematic verification of the 556. Verification is complete through pin 12; the next measurement is pin 13 (DISCHARGE2).


## STEP contact chatter and Schmitt-trigger debounce — 19 September 2026

### Symptom

Logic-analyser captures made during manual stepping showed that a single intended STEP action could produce multiple CPU-clock transitions. This made the assumption "one button press = one Z80 T-state" unsafe.

### Isolation by measurement

The fault was traced backwards through the clock path rather than treated as a generic CPU problem.

Observed sequence:

1. DSLogic showed extra short CPU-clock events during manual stepping.
2. Oscilloscope probing confirmed that the disturbance was real rather than only a logic-analyser threshold artefact.
3. The disturbance was visible at 556 pin 9, the manual monostable output.
4. Probing 556 pin 8 showed several rapid HIGH/LOW transitions around a STEP action.
5. This established that the unwanted events were already present at the trigger input, upstream of the 556 timing function.

A hand-held jumper was considered as a bypass test, but hand movement itself introduced contact chatter and therefore was not a sufficiently controlled discriminator.

### Mechanical-switch comparison

The original STEP switch was replaced temporarily with a small 6 mm tactile momentary switch.

Result: the new switch was dramatically cleaner, but a retained DSLogic capture still contained at least one additional short clock event. The mechanical replacement therefore reduced the fault but did not provide the reliability required for CPU single-stepping.

### Deduction

The evidence supported the following causal model:

mechanical contact bounce → repeated/slow trigger transitions → repeated 556 trigger activity → additional manual clock events → unreliable Z80 T-state stepping.

The solution was therefore derived from the measured failure mechanism rather than copied from an existing computer design.

### Alternatives investigated

Before choosing the final implementation, the following hardware approaches were considered:

- CD40106BE Schmitt-trigger inverter with RC input filtering;
- CD4093BE Schmitt NAND logic;
- comparator-based hysteresis using LM393, HA17393 or AN1319;
- NAND SR-latch debouncing using an SPDT momentary switch;
- 74LS123 monostable conditioning;
- flip-flop-based conditioning;
- modification of the existing 556 trigger network.

The alternatives remain useful experiments, particularly the comparator approach because it would allow hysteresis thresholds to be calculated and measured directly.

### Implemented correction

A CD40106BE was added ahead of 556 pin 8.

Current network:

- CD40106B pin 1 → 47 kΩ → +5 V;
- CD40106B pin 1 → 100 nF → GND;
- CD40106B pin 1 → 1 kΩ → STEP switch → GND;
- CD40106B pin 2 → pin 3;
- CD40106B pin 4 → 556 pin 8;
- pin 14 → +5 V;
- pin 7 → GND;
- unused CMOS inputs tied to GND.

The previous direct 10 kΩ pull-up and mechanical switch connection at 556 pin 8 was removed.

### Verification

Oscilloscope observation showed:

- the CD40106B pin-1 RC node making a controlled HIGH-to-LOW transition;
- 556 pin 8 receiving a clean abrupt HIGH-to-LOW trigger with no visible chatter at the test timebase.

A subsequent retained DSLogic capture showed one clean CPU clock pulse per deliberate STEP action and no short chatter bursts.

### Selector-path recheck

An apparent later observation suggested that STEP might still be reaching the CPU while RUN was selected. Because extensive probing and temporary wiring had been taking place, the complete propagation path was rechecked.

Measured:

- SN74LS157 pin 1: 0 V in MANUAL, approximately 3.6 V in RUN;
- SN74LS157 pin 4: no STEP-correlated change in RUN;
- SN7400 pin 9: no STEP-correlated change in RUN;
- SN7400 pin 11: no STEP-correlated change in RUN.

The anomaly could not be reproduced. Current evidence supports correct source selection. The earlier observation is retained as a transient debugging/probing artefact rather than silently erased.

### Learning result

This fault is a useful Athena example of evidence-led design. The circuit was not "fixed" by replacing parts at random. The unwanted CPU behaviour was traced to its source, alternative explanations were eliminated, a conditioning mechanism appropriate to the observed analogue behaviour was selected, and the correction was verified both locally and at the final CPU clock.
