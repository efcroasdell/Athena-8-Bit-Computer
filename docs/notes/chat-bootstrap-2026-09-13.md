# Athena Chat Bootstrap — 2026-09-13

Use this as the compact first-message context for a fresh Athena conversation.

Athena is my practical 8-bit computer build and electronics learning/documentation project. It is strictly separate from Phoenix, my FPGA/Verilog modular multi-processor project.

Repository: `efcroasdell/Athena-8-Bit-Computer`

GitHub is the authoritative engineering record. Chat is only the working conversation.

Local clone: `~/Google Drive/My Drive/Personal Projects/Github/Athena-8-Bit-Computer`

## Working rules

- One diagnostic measurement or connection per step.
- Wait for my result before moving on.
- Every photograph is fresh evidence.
- Do not infer wiring from expected topology.
- Distinguish observation, documented fact, deduction and hypothesis.
- Do not reinterpret measurements to fit the model.
- Explain the internal IC mechanism, not merely HIGH/LOW behaviour.
- Use exact pin numbers and measured values.
- Use British English.
- Boolean negation uses overbar notation, e.g. `Q̄` or `\(\overline{Q}\)`.
- Preserve Unix case exactly.
- Do not guess KiCad UI commands.

Do not treat my electronics interests as scattered. I naturally see valves, TTL, processors, FPGA, RF, instrumentation and restoration as connected through theory, function, history and technological lineage.

## Current clock architecture

- U1: RS 305-838 / NE556 dual timer
- U2: vintage TI SN7400 standard TTL quad NAND
- U3: TI SN74LS157N

Functions:

- 556 Timer 1 = variable-speed astable
- 556 Timer 2 = manual monostable STEP
- 7400 A/B = RUN/MANUAL latch
- 74LS157 = selects manual or astable clock
- 7400 C/D = HLT gating
- 7400 pin 11 = final internal CPU clock

## 556 pinout

1 DISCHARGE1
2 THRESHOLD1
3 CONTROL1
4 RESET1
5 OUTPUT1
6 TRIGGER1
7 GND
8 TRIGGER2
9 OUTPUT2
10 RESET2
11 CONTROL2
12 THRESHOLD2
13 DISCHARGE2
14 VCC

## Timer 2 wiring

- pin 8 → +5 V through 10 kΩ pull-up
- STEP push-button pulls pin 8 to GND
- pin 9 = monostable output
- pin 10 → +5 V
- pin 11 CONTROL2 externally unconnected; optional 10 nF (`103`) bypass to GND
- pins 12 and 13 tied together
- +5 V → 1 MΩ → pins 12/13
- pins 12/13 → 1 µF → GND

Nominal monostable interval ≈ 1.1 s.

## Verification state

Pins 1–12 have been tested.

Recent measurements:

- pin 8 idle: ~4.72–5.04 V
- pin 8 pressed: ~0 V
- pin 9 idle: ~0 V
- pin 9 triggered: ~4.32 V HIGH pulse; small transient edge overshoot noted
- pin 10: ~4.72–5.04 V steady HIGH
- pin 11: ~3.0–3.3 V steady control/reference level
- pin 12 idle: Vmin ~−160 mV, Vmax ~+80 mV, Vpp ~240 mV
- pin 12 triggered: Vmin ~−160 mV, Vmax ~3.20 V, Vpp ~3.36 V, clear exponential charge followed by abrupt discharge

Important fault history: a ~1 kΩ resistor had accidentally been fitted where the Timer 2 timing resistor should have been 1 MΩ. That made the timing interval roughly 1000× too short and made pin 12 appear square at the previous timebase. After fitting 1 MΩ, the expected exponential timing ramp appeared. Keep this in the fault history, not in the clean pin-verification entry.

## Journal format

For every measured pin:

### Pin N — FUNCTION

**What it does:** internal IC mechanism

**What is connected to it:** actual present circuit

**Expected:** predicted waveform/voltage

**Why:** internal mechanism producing it

**Observed:** actual scope values and waveform

Then explain what the measurement proves internally and finish with:

**Pin N is behaving correctly.**

For multiple states, document each state separately. Do not include earlier faults in the clean verification entry.

## Immediate continuation point

Next pin: **13 — DISCHARGE2**.

Pins 12 and 13 are externally the same timing node, so the external waveform should be similar. The pin-13 explanation must focus on its internal role as the connection to the discharge transistor:

- idle: transistor ON, timing node held near ground;
- trigger: transistor OFF, capacitor charges;
- threshold reached: latch resets;
- transistor ON again, timing node rapidly returns to ground.

Do not write observed values until I provide a fresh pin-13 scope trace.

## KiCad

Version: KiCad 10.0.3 on macOS.

Project: `hardware/kicad/Athena/Athena.kicad_pro`

Top-level schematic: `Athena.kicad_sch`

Hierarchical sheet: `Clock.kicad_sch`

Clock sheet currently contains NE556 A/B, 7400 A/B/C/D/E, 74LS157, +5 V, GND, `R1 = 1k`, `RV1 = 1M`, and `C1 = 1uF` polarised.

Do not trust old remembered KiCad menu paths. Verify KiCad 10.0.3 macOS commands before instructing me.