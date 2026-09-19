# Athena Clock Module

## Purpose

The Athena clock module provides the timing and manual stepping functions required by the 8-bit computer while preserving the user-facing behaviour of the original Ben Eater clock module:

- variable-speed free-running clock
- manual single-step clock
- RUN/MANUAL mode selection
- HLT-controlled clock gating
- final CPU clock output
- observable intermediate signals for verification and learning

The current implementation is intentionally compact but remains built from discrete TTL-era functions so that each stage can be observed and understood directly.

## Current architecture

The current clock uses four main ICs:

| Device | Function in Athena |
|---|---|
| RS 305-838 dual 556 timer | Timer 1: astable free-running clock. Timer 2: monostable manual STEP pulse. |
| TI CD40106BE hex Schmitt-trigger inverter | RC + hysteretic conditioning of the mechanical STEP input. Two inverter stages preserve active-LOW trigger polarity. |
| TI SN7400 standard TTL quad NAND | RUN/MANUAL latch and HLT clock gating. |
| TI SN74LS157N quad 2-to-1 multiplexer | Selects manual or astable clock source. |

Functional signal path:

1. 556 Timer 1 generates the variable-speed astable clock.
2. The STEP push-button is filtered by a 47 kΩ / 100 nF RC network and two CD40106B Schmitt inverters.
3. 556 Timer 2 generates one monostable pulse for each clean active-LOW STEP trigger.
4. The SN7400 cross-coupled NAND latch stores the RUN/MANUAL state.
5. The SN74LS157 selects either the manual pulse or the astable clock according to the stored mode.
6. The remaining SN7400 gates apply HLT control to the selected clock.
7. SN7400 pin 11 is the final internal CPU clock output.

## 556 pinout used in this build

The RS 305-838 dual 556 pinout has been verified as:

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

Timer 1 is configured as an astable oscillator. Timer 2 is configured as a monostable single-step generator.

## Timer 1: astable clock

Current functional connections:

- pin 14 → +5 V
- pin 7 → GND
- pin 4 → +5 V
- pins 2 and 6 joined as the timing-capacitor node
- fixed timing resistor from +5 V to pin 1
- 1 MΩ speed-control potentiometer between pin 1 and the joined pins 2/6
- 1 µF timing capacitor from pins 2/6 to GND
- pin 5 = astable clock output

The timing node repeatedly charges and discharges between approximately one-third and two-thirds of VCC. The internal threshold and trigger comparators change the state of the timer latch at those points, while the discharge transistor on pin 1 controls the capacitor discharge interval.

## Timer 2: manual STEP monostable

Current functional connections:

- pin 8 = active-LOW STEP trigger driven by CD40106B pin 4
- CD40106B pin 1 is the debounce node: 47 kΩ to +5 V, 100 nF to GND, and 1 kΩ in series with the normally-open STEP switch to GND
- CD40106B pin 2 → pin 3; pin 4 → 556 pin 8
- pin 9 = monostable output
- pin 10 → +5 V
- pin 11 normally unconnected; optional small control-voltage bypass may be fitted
- pins 12 and 13 joined as the monostable timing node
- 1 MΩ timing resistor from +5 V to pins 12/13
- 1 µF timing capacitor from pins 12/13 to GND

At rest, the CD40106B holds pin 8 HIGH and pin 9 is LOW. Pressing STEP discharges the RC node; the Schmitt thresholds convert the slow, potentially bouncing mechanical transition into one clean LOW-going trigger at pin 8. Pulling pin 8 below the internal trigger threshold sets the internal latch, drives pin 9 HIGH, turns the discharge transistor off, and allows the timing capacitor to charge. When the timing node reaches the upper threshold, the threshold comparator resets the latch, pin 9 returns LOW, and the discharge transistor restores the timing node to its idle state.

## SN7400 functions

The SN7400 uses all four NAND gates.

| Gate | Pins | Function |
|---|---|---|
| Gate 1 | 1,2 → 3 | First half of cross-coupled RUN/MANUAL latch |
| Gate 2 | 4,5 → 6 | Second half of cross-coupled RUN/MANUAL latch; pin 6 is Q |
| Gate 3 | 9,10 → 8 | First stage of HLT clock gating |
| Gate 4 | 12,13 → 11 | Inverts gate-3 result to restore final clock polarity |

Pin 6, Q, drives the SELECT input of the SN74LS157.

HLT control is applied at SN7400 pin 10. The current behaviour has been verified:

- pin 10 HIGH → selected clock passes through to pin 11 with the intended polarity
- pin 10 LOW → final clock pin 11 is held LOW

An HLT indicator LED is connected so that it illuminates when the HLT control node is LOW.

## SN74LS157 clock-source selector

Channel 1 is used for clock selection:

| Pin | Function in Athena |
|---:|---|
| 1 | SELECT ← SN7400 pin 6 (Q) |
| 2 | 1A ← 556 pin 9 manual STEP output |
| 3 | 1B ← 556 pin 5 astable output |
| 4 | 1Y → SN7400 pin 9 selected-clock input |
| 8 | GND |
| 15 | active-LOW strobe tied to GND |
| 16 | +5 V |

The selector function is:

Y = A·Q̄ + B·Q

so one clock source is selected according to the stored RUN/MANUAL state.

Unused LS157 data inputs are intended to be tied to a defined logic level rather than left floating. Unused outputs may remain open.

## Current verification status

Confirmed so far:

- 556 Timer 1 astable operation and timing-node behaviour
- 556 Timer 2 pin 8 active-LOW trigger behaviour
- 556 Timer 2 pin 9 idle LOW state and triggered HIGH pulse
- 556 Timer 2 pin 10 steady HIGH reset input
- 556 Timer 2 pin 11 steady control/reference level
- 556 Timer 2 pin 12 idle timing-node level and triggered exponential charge followed by rapid discharge
- CD40106B STEP debounce operation
- one STEP press producing one clean CPU clock pulse in the post-debounce DSLogic capture
- SN7400 RUN/MANUAL latch operation
- SN7400 HLT gating behaviour
- SN74LS157 enable state and SELECT input operation
- RUN-mode rejection of the manual STEP path verified at SN74LS157 pin 4, SN7400 pin 9 and SN7400 pin 11
- final CPU clock independently measured at approximately 270.407 kHz during the 19 September 2026 RUN-mode regression check

Systematic pin-by-pin 556 verification has reached pin 12, with pins 8, 9 and 12 checked in both idle and triggered states. Pins 13 and 14 remain to be documented. The manual STEP debounce and selector propagation path are now independently verified.

## Evidence and measurement record

Detailed oscilloscope measurements, expected behaviour, observed values, and explanations of the internal 556 mechanisms belong in the corresponding measurement journal under `docs/measurements/`.

Photographs and oscilloscope captures should be stored under `evidence/` and referenced from the journal by filename so that the written conclusion remains tied to the original evidence.

## Status

**Current architectural decision:** 556 + SN7400 + SN74LS157 implementation.

**Current verification state:** the clock architecture, RUN/MANUAL selection, HLT gating and debounced manual STEP path are functionally verified; remaining work is completion of the formal pin-by-pin 556 documentation and evidence indexing.

**Project boundary:** this clock module belongs to Athena. Phoenix FPGA/multi-processor architecture material is not part of this repository.
