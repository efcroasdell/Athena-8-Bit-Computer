# Athena Chat Migration Handover — 2026-09-13

Status: **current handover for starting a fresh ChatGPT conversation**

This document captures the current Athena project state after the previous conversation became unreliable. Treat this file together with the repository itself as the authoritative starting context for a new conversation.

## Project identity

Athena is the practical 8-bit computer build and associated electronics learning, measurement, verification and documentation project. It includes breadboard construction, TTL logic, timing circuits, oscilloscope and logic-analyser work, KiCad schematics, photographs, experiments, fault records, datasheets, firmware and programs.

Athena is strictly separate from **Phoenix**, the separate modular FPGA/Verilog multi-processor architecture project. Do not merge the two projects or their documentation.

GitHub repository: `efcroasdell/Athena-8-Bit-Computer`

GitHub is the authoritative engineering record. Chat is a working conversation, not the permanent source of truth.

Local clone:

`~/Google Drive/My Drive/Personal Projects/Github/Athena-8-Bit-Computer`

## Working method

The project prioritises understanding and verification rather than merely obtaining a working circuit.

For live diagnosis:

- take exactly one measurement or make exactly one connection per step;
- wait for the result before moving on;
- treat every new photograph as fresh evidence;
- trace visible wiring rather than filling in expected topology;
- distinguish observed fact, documented fact, deduction and hypothesis;
- do not reinterpret measurements to force them to fit the model;
- explain what is happening inside the IC, not merely external HIGH/LOW behaviour.

Use exact pin numbers and measured values. Boolean negation should use overbar notation, for example `Q̄` or `\(\overline{Q}\)`.

The working desktop is macOS/Unix. Preserve exact case in paths, filenames, commands and identifiers.

The user's electronics interests span valves, analogue electronics, TTL, processors, buses, FPGA, RF, SDR, instrumentation and restoration. Treat that breadth as coherent rather than scattered.

## Current clock architecture

The clock retains the Ben Eater clock's user-facing functions:

- variable-speed free-running clock;
- manual single-step pulse;
- RUN/MANUAL selection;
- clean clock-source selection;
- HLT clock gating;
- final CPU clock;
- observable intermediate signals.

Current implementation:

- `U1`: vintage RS 305-838 / NE556 dual timer;
- `U2`: vintage TI SN7400 standard TTL quad NAND;
- `U3`: TI SN74LS157N quad 2-to-1 multiplexer.

Functional allocation:

- 556 Timer 1 = variable-speed astable;
- 556 Timer 2 = manual monostable STEP pulse;
- SN7400 A/B = cross-coupled RUN/MANUAL latch;
- SN74LS157 = clock-source selector;
- SN7400 C/D = HLT gating;
- SN7400 pin 11 = final internal CPU clock.

## RS 305-838 / NE556 pinout

| Pin | Signal |
|---:|---|
| 1 | DISCHARGE1 |
| 2 | THRESHOLD1 |
| 3 | CONTROL1 |
| 4 | RESET1 |
| 5 | OUTPUT1 |
| 6 | TRIGGER1 |
| 7 | GND |
| 8 | TRIGGER2 |
| 9 | OUTPUT2 |
| 10 | RESET2 |
| 11 | CONTROL2 |
| 12 | THRESHOLD2 |
| 13 | DISCHARGE2 |
| 14 | VCC |

Only pins 7 and 14 are shared supply pins.

## Timer 1 — astable

Current configuration:

- pin 14 → +5 V;
- pin 7 → GND;
- pin 4 → +5 V;
- pins 2 and 6 tied together;
- fixed resistor from +5 V to pin 1;
- 1 MΩ speed-control potentiometer between pin 1 and the joined pins 2/6 timing node;
- 1 µF timing capacitor from pins 2/6 to GND;
- pin 5 = astable output.

Verified measurements:

- pin 1 DISCHARGE1: Vmin ≈ −80 mV, Vmax ≈ 5.12 V, Vpp ≈ 5.20 V;
- pin 2 THRESHOLD1: Vmin ≈ 1.750 V, Vmax ≈ 3.370 V, Vpp ≈ 1.620 V, timing ramp;
- pin 3 CONTROL1: Vmin ≈ 3.290 V, Vmax ≈ 3.370 V, Vpp ≈ 80 mV;
- pin 4 RESET1: firmly HIGH;
- pin 5 OUTPUT1: Vmin ≈ −80 mV, Vmax ≈ 3.640 V, Vpp ≈ 3.720 V, square wave;
- pin 6 TRIGGER1: Vmin ≈ 1.760 V, Vmax ≈ 3.320 V, Vpp ≈ 1.560 V, same timing-node ramp;
- pin 7 GND: near ground.

## Timer 2 — monostable

Current configuration:

- pin 8 → +5 V through 10 kΩ pull-up;
- STEP push-button momentarily connects pin 8 to GND;
- pin 9 = monostable output;
- pin 10 → +5 V;
- pin 11 CONTROL2 externally unconnected at present; optional 10 nF (`103`) bypass to GND;
- pins 12 and 13 tied together;
- +5 V → 1 MΩ → pins 12/13 timing node;
- pins 12/13 → 1 µF → GND.

Nominal monostable interval is about `1.1 × R × C`, approximately 1.1 s for 1 MΩ and 1 µF.

### Pin 8 — TRIGGER2

Released:

- Vmin ≈ 4.72 V;
- Vmax ≈ 5.04 V;
- Vpp ≈ 320 mV;
- stable HIGH.

Pressed:

- approximately +5 V → 0 V → +5 V;
- correct active-LOW trigger.

Internally, falling below approximately one-third VCC causes the trigger comparator to set the Timer 2 SR latch, driving pin 9 HIGH and turning the discharge transistor OFF.

### Pin 9 — OUTPUT2

Idle:

- Vmin ≈ −240 mV;
- Vmax ≈ +160 mV;
- Vpp ≈ 400 mV;
- effectively 0 V.

Triggered:

- LOW ≈ 0 V;
- HIGH ≈ 4.32 V;
- Vpp ≈ 4.64 V;
- clean monostable pulse;
- small transient overshoot visible at switching edges.

The overshoot is an observation, not a diagnosed fault. Its source has not yet been isolated between the 556 output transition and the probing arrangement.

### Pin 10 — RESET2

- tied to +5 V;
- Vmin ≈ 4.72 V;
- Vmax ≈ 5.04 V;
- Vpp ≈ 320 mV;
- stable HIGH.

RESET is active LOW. Holding it HIGH removes the reset override and allows the trigger/threshold comparators and latch to operate normally.

### Pin 11 — CONTROL2

Pin 11 exposes Timer 2's upper control/reference node associated with approximately two-thirds VCC. It is externally unconnected but is not electrically floating internally because the 556's internal divider biases it.

Observed approximately 3.0–3.3 V, essentially flat.

An optional 10 nF ceramic capacitor to GND may be used as a noise bypass. Capacitor marking: `103`.

### Pin 12 — THRESHOLD2

Pin 12 is tied externally to pin 13 at the timing node.

Idle:

- Vmin ≈ −160 mV;
- Vmax ≈ +80 mV;
- Vpp ≈ 240 mV;
- essentially 0 V.

Triggered:

- Vmin ≈ −160 mV;
- Vmax ≈ 3.20 V;
- Vpp ≈ 3.36 V;
- clear exponential charging curve;
- abrupt return to approximately 0 V.

This waveform directly shows the timing capacitor charging while the discharge transistor is OFF, the threshold comparator tripping near two-thirds VCC, the SR latch resetting, and the discharge transistor returning the node rapidly to ground.

## Important current fault history

During pin-12 investigation the timing node initially appeared square rather than showing the expected capacitor ramp. Replacing the 1 µF capacitor with 10 µF, and even removing the capacitor, did not initially produce the expected behaviour.

Continuity checks confirmed:

- capacitor positive node → pin 12 = continuity;
- capacitor positive node → pin 13 = continuity.

Resistance from the pins-12/13 timing node to +5 V was then measured.

Expected: approximately 1 MΩ.

Measured: approximately `0.9943 kΩ`, about 994 Ω.

A roughly 1 kΩ resistor had been installed instead of 1 MΩ. That made the timing interval roughly 1000 times too short. With 1 kΩ and 1 µF, the interval is only around 1.1 ms, so at the previous 100 ms/div timebase the capacitor charge appeared effectively instantaneous.

After fitting the correct 1 MΩ resistor, the expected exponential pin-12 timing ramp appeared immediately.

This belongs in the fault history, not in the clean pin-verification journal.

## Next measurement

The next systematic measurement is:

**Pin 13 — DISCHARGE2**

Pins 12 and 13 are externally the same timing node, so the external voltage waveform should be similar. The explanation for pin 13 must focus on its distinct internal role: connection to the internal discharge transistor.

Expected internal sequence:

- idle: discharge transistor ON, timing node held near ground;
- trigger: discharge transistor turns OFF, capacitor charges;
- threshold reached: latch resets;
- discharge transistor turns ON again, rapidly returning the timing node to ground.

Do not write observed values until a fresh pin-13 scope capture has been supplied.

## SN7400 current functions

Correct DIP14 gate allocation:

- gate 1: pins 1,2 → 3;
- gate 2: pins 4,5 → 6;
- gate 3: pins 9,10 → 8;
- gate 4: pins 12,13 → 11;
- pin 7 = GND;
- pin 14 = VCC.

Gates 1 and 2 form the cross-coupled RUN/MANUAL latch. User-defined `Q` is pin 6 and `Q̄` is pin 3. Pin 6 drives the SN74LS157 SELECT input.

HLT gating uses gates 3 and 4. Pin 10 is the HLT/CLOCK_ENABLE input with a 10 kΩ pull-up; the HLT switch pulls it LOW. Verified behaviour:

- pin 10 HIGH → selected clock passes to pin 11 with intended polarity;
- pin 10 LOW → pin 11 held LOW.

An HLT LED indicates the LOW/active HLT state.

## SN74LS157 current use

Channel 1:

- pin 1 SELECT ← SN7400 pin 6 (`Q`);
- pin 2 1A ← 556 pin 9 manual STEP output;
- pin 3 1B ← 556 pin 5 astable output;
- pin 4 1Y → SN7400 pin 9 selected-clock input;
- pin 8 GND;
- pin 15 active-LOW enable → GND;
- pin 16 +5 V.

Selection equation:

`Y = A Q̄ + B Q`

or `\(Y=A\overline{Q}+BQ\)`.

Unused LS157 data inputs should be tied to a defined logic level; unused outputs may remain open.

## KiCad state

Version: **KiCad 10.0.3 on macOS**.

One KiCad project is intended to hold all Athena schematics:

`hardware/kicad/Athena/Athena.kicad_pro`

Top-level schematic:

`Athena.kicad_sch`

Hierarchical clock sheet:

`Clock.kicad_sch`

The Clock sheet currently contains:

- NE556 units A and B;
- SN7400 units A, B, C, D and power unit E;
- SN74LS157;
- +5 V and GND symbols;
- `R1 = 1k`;
- `RV1 = 1M`;
- `C1 = 1uF`, polarised.

Planned IC references:

- U1 = NE556;
- U2 = SN7400;
- U3 = SN74LS157.

The KiCad work was interrupted because the previous conversation repeatedly gave incorrect or unverified UI instructions. In a new conversation, do **not** trust remembered menu paths from the old thread. Verify every KiCad 10.0.3 macOS command or menu path against current documentation or the user's screenshot before instructing.

The schematic should be deliberately arranged before extensive wiring. Avoid long tangled wires; use short local wiring and meaningful net labels where appropriate.

## Journal format

For every measured pin, use the formal learning-journal structure:

### Pin N — FUNCTION

**What it does:** explain the internal IC mechanism.

**What is connected to it:** state the actual present external circuit.

**Expected:** state the predicted waveform or level before measurement.

**Why:** explain the internal mechanism producing it.

**Observed:** record actual scope values and waveform.

Then explain what the observation demonstrates internally and finish with either:

**Pin N is behaving correctly.**

or a clear unresolved discrepancy.

For pins with multiple meaningful states, document each state separately under the same pin entry.

Do not contaminate clean verification entries with fault-history narrative. The 1 kΩ/1 MΩ mistake belongs in `docs/faults/clock-fault-history.md`, not in the clean pin-12 verification entry.

## Evidence and documentation

Scope photographs and breadboard photographs are engineering evidence and should eventually be stored under the repository's `evidence/` hierarchy and linked from the relevant journal entries.

The user is also maintaining a detailed LibreOffice learning journal. Scope images should participate in document flow rather than float over text; image layout/page-flow details should be verified against the actual LibreOffice UI before giving instructions.

## Trust cautions from the old conversation

Do not repeat these failures:

- inventing or misreading visible breadboard connections;
- replacing the user's stated physical action with the wiring the circuit was expected to have;
- guessing IC pinouts;
- giving KiCad menu paths from memory without verification;
- bundling several diagnostic actions when one was requested;
- treating broad electronics interests as disconnected.

The next technical continuation point is pin 13 of the 556.