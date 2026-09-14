# Athena Chat Bootstrap — 2026-09-13

Use this as the compact first-message context for a fresh Athena conversation.

Updated: **14 September 2026**. Original filename retained for existing links.

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

The learning journal, `evidence/test-results/Athena Clock Module — Learning and Verification Journal.docx`, documents **556 pins 1–14** as behaving correctly. Older Markdown measurement/milestone records still stop at pin 12 and need reconciliation; do not repeat completed pin checks based on those stale statements.

Recent measurements:

- pin 8 idle: ~4.72–5.04 V
- pin 8 pressed: ~0 V
- pin 9 idle: ~0 V
- pin 9 triggered: ~4.32 V HIGH pulse; small transient edge overshoot noted
- pin 10: ~4.72–5.04 V steady HIGH
- pin 11: ~3.0–3.3 V steady control/reference level
- pin 12 idle: Vmin ~−160 mV, Vmax ~+80 mV, Vpp ~240 mV
- pin 12 triggered: Vmin ~−160 mV, Vmax ~3.20 V, Vpp ~3.36 V, clear exponential charge followed by abrupt discharge

Pin 13 is documented with a timing waveform consistent with the shared pin-12 node. Pin 14: Vmin ≈ 4.720 V, Vmax ≈ 5.040 V, Vpp ≈ 320 mV, essentially steady near +5 V. Full selector/output-path bench rechecking remains outstanding.

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

The latest companion discussion concerned schematic presentation: U3 has the component value `74LS157`, but lacks a separate large section heading matching TIMER 1, TIMER 2 and SN7400 A/B or C/D. The previous answer confused the component value with that heading.

Proposed heading: **SN74LS157 — CLOCK SOURCE SELECTOR**, using the same size and style as the other section headings. This remains an outstanding drawing change, not a completed edit.

The next bench phase is the planned recheck of the full latch/selector/HLT/output path, one measurement or connection at a time. Do not restart pin-13 verification merely because an older handover says it is next. Do not treat schematic connectivity or ERC as proof of physical wiring, loading, timing or glitch-free switching.

## KiCad

Version: KiCad 10.0.3 on macOS.

Project: `hardware/kicad/Athena/Athena.kicad_pro`

Top-level schematic: `Athena.kicad_sch`

Hierarchical sheet: `Clock.kicad_sch`

The Clock sheet now contains both halves of U1 (RS 305-838 / NE556), all five units of U2 (TI SN7400), U3 (TI SN74LS157N), both timing networks, STEP and RUN/MANUAL controls, HLT gating and the two required LEDs. Signal paths are continuously wired; explicit +5 V and ground symbols denote common rails. PWR_FLAG is an ERC annotation, not a physical component.

Exports:

- `hardware/kicad/Athena/exports/Athena-clock.pdf` — complete clock drawing on one A3 page;
- `hardware/kicad/Athena/exports/Clock-erc.txt` — recorded ERC result: **0 errors, 1 warning**, for the CPU_CLK label connected only to the module output pin.

The schematic README records a check of all 44 IC pins in the exported netlist. These are drawing checks, not proof of physical loading, timing or glitch-free operation.

Details still requiring confirmation:

- R4/R5: fitted RUN/MANUAL pull-up resistances;
- R7/R8: fitted HLT and astable LED series resistances;
- D2: complete wiring/polarity; U1 pin 5 is the confirmed signal, return to GND is provisional;
- RV1: whether the unused end is open or strapped to the wiper;
- SW2: physical contact numbering and switch type;
- C3/C4/C5: fitted presence and values of local supply bypass capacitors;
- U3 unused data inputs: grounded in the schematic, actual breadboard connections still to be checked.

U1 control pins 3 and 11 remain externally unconnected in the drawing. Optional control bypass capacitors are not shown as fitted. Footprints and PCB layout remain undecided. The schematic is not yet a definitive as-built record or final build specification.

See `hardware/kicad/Athena/README.md` for current connection and verification details. Verify KiCad 10.0.3 macOS commands against current documentation or the actual UI before giving instructions.
