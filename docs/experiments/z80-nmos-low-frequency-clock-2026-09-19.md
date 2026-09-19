# Athena Experiment — NMOS Z80 Low-Frequency Clock Behaviour

Date: **19 September 2026**

Status: **observed experiment; exact failure threshold not yet characterised**

## Purpose

Observe directly what happens when the 1986 NMOS Z80A is clocked progressively more slowly, rather than treating the published clock limits only as datasheet abstractions.

The experiment was motivated by two objectives:

- verify the actual CPU clock frequency independently with bench instrumentation;
- observe the transition from coherent Z80 execution to invalid behaviour as the clock is slowed below the region in which the NMOS device is expected to operate reliably.

The frequency counter also provides useful visible bench feedback, but the measurement purpose is primary.

## Test configuration

The Z80 remains in the forced-NOP bring-up configuration:

- data bus held LOW through 1 kΩ pulldowns so memory reads present 00h NOP;
- DSLogic Plus connected to A0–A7, M1̄, MREQ̄, RD̄ and CLK;
- CPU clock observed at Z80 pin 6;
- clock source is the Athena 556 astable through the SN74LS157 selector and SN7400 HLT-gating path.

The Long Wei TFC-2700L frequency counter was connected to the final CPU clock node at Z80 pin 6, with the counter ground connected to Athena circuit ground.

## Normal-speed regression check

With RUN selected, the frequency counter displayed approximately:

**270.407 kHz**

This is consistent with the recent oscilloscope and logic-analyser observations of the Athena clock in the approximately 270–280 kHz region.

At this operating speed the analyser showed coherent execution:

- A0–A7 displayed the expected binary hierarchy;
- A0 changed fastest, with progressively slower activity on higher address bits;
- M1̄, MREQ̄ and RD̄ were active and regular;
- the pattern was consistent with continued forced-NOP execution.

This established that the substantial rebuilding and STEP-debounce work had not broken the normal RUN-mode CPU baseline.

## Very-low-frequency observation

The astable clock was then reduced far below its normal operating region.

At the very low clock rate the analyser capture changed qualitatively:

- A0–A7 no longer behaved as a coherent binary count;
- several address lines moved together rather than independently;
- M1̄ remained inactive/stuck in the captured interval;
- MREQ̄ and RD̄ behaviour no longer resembled the regular forced-NOP machine-cycle pattern;
- a valid clock signal was still present, but valid Z80 execution was not.

Restoring a substantially higher clock rate restored coherent address progression and regular control activity.

## Interpretation

### Observed fact

The same physical system behaved coherently at the higher clock rate and lost coherent CPU bus behaviour when the clock was reduced to the very-low-frequency region.

### Deduction

Clock frequency is therefore the material experimental variable separating the two captures. The observation is consistent with the known dynamic behaviour of early NMOS Z80 devices: a clock may still be electrically present while the internal processor state is no longer being maintained correctly at excessively long clock phases.

### Not yet established

This experiment does **not** yet establish the exact minimum operating frequency of this individual Z80A specimen.

To establish that boundary properly, a future experiment should:

1. begin from a verified normal operating frequency;
2. reduce frequency in controlled steps;
3. record the final CPU clock frequency and HIGH/LOW pulse widths at each step;
4. use the same logic-analyser acceptance criterion at each frequency;
5. identify the highest frequency at which failure first appears and the lowest frequency at which correct operation is reproducible;
6. repeat the boundary measurements to distinguish a stable threshold region from a one-off failure.

## Engineering significance

This experiment demonstrates an important distinction:

**clock signal present** does not necessarily mean **processor executing correctly**.

It also demonstrates why Athena retains direct access to the CPU clock, address bus and control signals. Without the frequency counter, oscilloscope and logic analyser, the low-frequency limit would be a datasheet statement rather than an observable property of the physical processor.

## Evidence

Relevant captures from 19 September 2026 include:

- a very-low-frequency capture showing loss of coherent bus behaviour;
- a higher-frequency comparison capture showing restored binary address progression and regular control activity;
- the annotated print-friendly DSView figure generated from the higher-frequency comparison capture.

The raw .dsl captures should be preserved under evidence/logic-analyser/ when committed to the repository.