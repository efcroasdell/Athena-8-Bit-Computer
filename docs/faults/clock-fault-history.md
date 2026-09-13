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

No unresolved fault currently prevents continued systematic verification of the 556. The next measurement work begins at pin 10.
