# Athena Clock Module — Wiring Record

Status: **current working breadboard wiring**

This file records the wiring of the present Athena clock module. It is intended to describe the circuit as actually built and verified, not an idealised future design.

## Supply convention

- Logic supply: +5 V
- Ground: common 0 V reference
- All IC supply pins require local decoupling appropriate to the final build.

## RS 305-838 dual 556

Verified pinout:

| Pin | Signal | Current connection |
|---:|---|---|
| 1 | DISCHARGE1 | Astable timing network; fixed resistor from +5 V and 1 MΩ speed potentiometer to pins 2/6 node |
| 2 | THRESHOLD1 | Joined to pin 6 and 1 µF timing capacitor to GND |
| 3 | CONTROL1 | Internal control-voltage reference; no functional external control connection required for current build |
| 4 | RESET1 | +5 V |
| 5 | OUTPUT1 | Astable clock output; feeds SN74LS157 channel-1 B input |
| 6 | TRIGGER1 | Joined to pin 2 and 1 µF timing capacitor to GND |
| 7 | GND | GND |
| 8 | TRIGGER2 | Driven by CD40106B pin 4; HIGH at rest and driven LOW by the debounced STEP path |
| 9 | OUTPUT2 | Manual monostable output; feeds SN74LS157 channel-1 A input |
| 10 | RESET2 | +5 V |
| 11 | CONTROL2 | Normally unconnected in current build |
| 12 | THRESHOLD2 | Joined to pin 13; 1 MΩ to +5 V and 1 µF to GND |
| 13 | DISCHARGE2 | Joined to pin 12 timing node |
| 14 | VCC | +5 V |

### Timer 1 astable timing network

- +5 V → fixed timing resistor → pin 1
- pin 1 → 1 MΩ speed potentiometer → pins 2/6
- pins 2/6 → 1 µF → GND
- pin 5 = free-running clock output

The 1 MΩ potentiometer is used as the variable timing resistance between pin 1 and the joined pins 2/6. The wiper is the active connection to the timing node; the unused outer terminal may be tied to the wiper as a fail-safe against an open wiper.

### Timer 2 monostable timing and STEP network

The STEP trigger is now conditioned by a CD40106B Schmitt-trigger stage before reaching pin 8.

- CD40106B pin 4 → 556 pin 8
- pins 12/13 → 1 MΩ → +5 V
- pins 12/13 → 1 µF → GND
- pin 9 = manual STEP pulse output

The former direct 10 kΩ pull-up and push-button connection at 556 pin 8 has been removed. Pin 8 is now actively driven by CD40106B pin 4.

## TI CD40106BE hex Schmitt-trigger inverter

Two inverter stages are used so that the debounced signal presented to the 556 retains the original active-LOW STEP polarity.

| Pin | Current connection |
|---:|---|
| 1 | Debounce RC node |
| 2 | First Schmitt inverter output; connected to pin 3 |
| 3 | Second Schmitt inverter input; connected from pin 2 |
| 4 | Debounced STEP output → 556 pin 8 |
| 5 | Unused input tied to GND |
| 6 | Unused output; unconnected |
| 7 | VSS → GND |
| 8 | Unused output; unconnected |
| 9 | Unused input tied to GND |
| 10 | Unused output; unconnected |
| 11 | Unused input tied to GND |
| 12 | Unused output; unconnected |
| 13 | Unused input tied to GND |
| 14 | VDD → +5 V |

### STEP debounce network

- CD40106B pin 1 → 47 kΩ → +5 V
- CD40106B pin 1 → 100 nF → GND
- CD40106B pin 1 → 1 kΩ → normally-open STEP switch → GND
- CD40106B pin 2 → pin 3
- CD40106B pin 4 → 556 pin 8

Existing 100 nF rail decoupling is present close to the logic devices. The 100 nF capacitor on pin 1 is not a supply-decoupling capacitor; it is part of the RC debounce network.

At rest, the 47 kΩ resistor charges the pin-1 node HIGH. Pressing STEP discharges the node through 1 kΩ. Schmitt hysteresis prevents contact bounce and the slow RC transition from producing multiple logic transitions. The two inversions restore the required HIGH-at-rest, LOW-on-STEP polarity at 556 pin 8.

## TI SN7400 standard TTL quad NAND

| Pin | Function in current clock module |
|---:|---|
| 1 | Gate 1 input A; switch-driven latch input, normally pulled HIGH and switched LOW |
| 2 | Gate 1 input B; feedback from pin 6 |
| 3 | Gate 1 output; complementary latch output, feeds pin 5 |
| 4 | Gate 2 input A; switch-driven latch input, normally pulled HIGH and switched LOW |
| 5 | Gate 2 input B; feedback from pin 3 |
| 6 | Gate 2 output Q; drives SN74LS157 pin 1 SELECT |
| 7 | GND |
| 8 | Gate 3 output; feeds pins 12 and 13 |
| 9 | Gate 3 clock input; receives selected clock from SN74LS157 pin 4 |
| 10 | HLT/CLOCK_ENABLE input; 10 kΩ pull-up to +5 V, HLT switch pulls LOW |
| 11 | Gate 4 output; final internal CPU clock |
| 12 | Gate 4 input A; tied to pin 8 |
| 13 | Gate 4 input B; tied to pin 8 |
| 14 | +5 V |

### HLT indicator

Current indicator connection:

- +5 V → current-limiting resistor → LED anode
- LED cathode → SN7400 pin 10 HLT node

The LED therefore illuminates when HLT is active and the pin-10 node is LOW.

## TI SN74LS157N selector

Channel 1 performs RUN/MANUAL clock selection.

| Pin | Current connection |
|---:|---|
| 1 | SELECT ← SN7400 pin 6 Q |
| 2 | 1A ← 556 pin 9 manual STEP output |
| 3 | 1B ← 556 pin 5 astable output |
| 4 | 1Y → SN7400 pin 9 selected-clock input |
| 8 | GND |
| 15 | active-LOW strobe tied to GND |
| 16 | +5 V |

Unused LS157 data inputs are to be tied to a defined logic level, normally GND. Unused outputs may remain unconnected.

## Current signal chain

`STEP switch` → `CD40106B RC + Schmitt conditioning` → `556 pin 8 trigger` → `556 pin 9 manual pulse`, while `556 pin 5` supplies the astable clock; the two sources then feed `SN74LS157 selector` → `SN7400 HLT gating` → `SN7400 pin 11 final CPU clock`

The RUN/MANUAL latch output on SN7400 pin 6 controls which 556 output the SN74LS157 selects.

## Verification state

Verified on the current breadboard:

- Timer 1 astable activity
- CD40106B debounce RC node transitions cleanly between HIGH and LOW
- CD40106B output provides a clean active-LOW trigger at 556 pin 8
- Timer 2 trigger HIGH at rest and LOW on STEP press
- Timer 2 output LOW at rest and HIGH for the monostable pulse
- one manual STEP action produces one clean CPU clock pulse in the post-debounce DSLogic capture
- RUN/MANUAL latch state change
- HLT gating behaviour
- LS157 strobe LOW and SELECT input changing with latch Q
- in RUN mode, STEP produces no observable disturbance at LS157 pin 4, SN7400 pin 9 or SN7400 pin 11

The selector and final clock propagation path were rechecked end-to-end on 19 September 2026 and behaved as designed.
