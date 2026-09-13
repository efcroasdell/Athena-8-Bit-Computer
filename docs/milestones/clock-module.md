# Athena Milestone — Clock Module

Status: **in progress**

## Purpose

Produce and verify a complete clock subsystem for Athena that preserves the functional behaviour required by the Ben Eater 8-bit computer while making each timing, selection and gating stage observable and understandable.

## Current implementation

- RS 305-838 dual 556
  - Timer 1: variable-speed astable clock
  - Timer 2: manual monostable STEP pulse
- TI SN7400
  - RUN/MANUAL state latch
  - HLT clock gating
- TI SN74LS157N
  - manual/astable clock-source selection

## Prerequisites

- stable +5 V supply
- common ground reference
- verified IC orientation and pinout
- oscilloscope available for analogue timing and digital waveform checks
- known-good STEP, RUN/MANUAL and HLT controls

## Model

The clock module consists of four observable functional stages:

1. generation of a free-running clock by the 556 astable timer;
2. generation of one manual pulse by the 556 monostable timer;
3. persistent RUN/MANUAL state stored by the SN7400 latch and applied to the SN74LS157 selector;
4. HLT gating of the selected clock through the remaining SN7400 gates.

The final internal CPU clock is SN7400 pin 11.

## Expected behaviour

### Astable mode

The timing capacitor repeatedly charges and discharges between approximately one-third and two-thirds of VCC. 556 pin 5 produces a corresponding square-wave clock whose frequency varies with the speed potentiometer.

### Manual mode

556 pin 8 is HIGH at rest. Pressing STEP pulls pin 8 LOW, causing pin 9 to produce one HIGH monostable pulse before returning automatically to LOW.

### Selection

The SN7400 RUN/MANUAL latch retains the chosen mode. Its Q output drives SN74LS157 SELECT so that only the appropriate clock source reaches the selected-clock output.

### Halt

HLT inactive permits the selected clock to propagate to the final clock output. HLT active forces the final clock LOW.

## Observability

The following nodes must remain measurable during this milestone:

- Timer 1 discharge node
- Timer 1 timing capacitor / threshold / trigger node
- Timer 1 control reference
- Timer 1 output
- Timer 2 trigger
- Timer 2 output
- Timer 2 timing node
- RUN/MANUAL latch Q and complementary output
- LS157 SELECT
- LS157 selected-clock output
- HLT control
- final CPU clock

## Predicted results

- Timer 1 timing node: ramp between roughly one-third and two-thirds VCC
- Timer 1 output: clean TTL-compatible square wave
- Timer 2 trigger: HIGH at rest, LOW while STEP is pressed
- Timer 2 output: LOW at rest, one HIGH pulse per valid trigger
- RUN/MANUAL latch: stable complementary states until deliberately changed
- selected clock: only the chosen source appears at LS157 output
- HLT active: final clock remains LOW
- HLT inactive: final clock follows selected source with intended polarity

## Acceptance criteria

The clock milestone is complete when:

- all relevant 556 pins have been measured and documented;
- astable frequency changes predictably with the speed control;
- one STEP action produces one manual output pulse;
- RUN/MANUAL state is stable and repeatable;
- source selection is verified at the LS157 input and output pins;
- HLT reliably suppresses the final clock;
- final clock waveform is measured directly;
- no input required to hold a defined state is left floating;
- the current breadboard wiring record matches the physical build;
- evidence files are linked from the measurement record.

## Fault tests

Controlled tests should include:

- force Timer 2 trigger LOW with a jumper and confirm the expected response;
- change RUN/MANUAL state and confirm LS157 SELECT changes accordingly;
- activate HLT while the astable source continues running and confirm only the final clock is suppressed;
- verify that removing a control action returns the node to its defined pull-up or stable state;
- where a waveform is suspicious, distinguish source behaviour from probe/ground artefacts before assigning a component fault.

## Required documentation

- `hardware/breadboard/clock/README.md`
- `hardware/breadboard/clock/wiring.md`
- `docs/design-decisions/clock-architecture.md`
- `docs/measurements/clock-556-verification.md`
- `docs/faults/clock-fault-history.md`
- oscilloscope evidence under `evidence/oscilloscope/clock/`
- breadboard photographs under `evidence/photos/clock/`

## Current state

556 pins 1 through 12 have been systematically checked. Pins 8, 9 and 12 have been verified in both idle and triggered states. Pins 13 and 14 remain to be verified; the next pin is 13 (DISCHARGE2). Pin verification is currently paused while the migration records are checked.
