# Athena Measurement Record — Z80 RUN-Mode Regression Check

Date: **19 September 2026**

Status: **verified for the captured configuration**

## Purpose

Confirm that the Z80 system remained operational after substantial rebuilding, STEP-switch diagnosis, addition of the CD40106B debounce stage, and repeated movement of temporary test leads.

## Test configuration

- Z80 in the established forced-NOP bring-up configuration
- A0–A7 observed on DSLogic Plus D0–D7
- M1̄ on D8
- MREQ̄ on D9
- RD̄ on D10
- CPU CLK on D11
- RUN mode selected
- final CPU clock measured at Z80 pin 6

## Clock-frequency cross-check

The Long Wei TFC-2700L frequency counter was connected to the final CPU clock node at Z80 pin 6, with the counter reference connected to Athena circuit ground.

Observed frequency:

**approximately 270.407 kHz**

This agreed with the recent oscilloscope and logic-analyser observations of the Athena clock in the approximately 270–280 kHz region.

## Logic-analyser observations

At the normal RUN-mode clock rate:

- A0–A7 behaved as independent address lines rather than moving together;
- A0 changed most rapidly, with each successively higher address bit changing less frequently;
- the resulting pattern showed the expected binary hierarchy;
- M1̄, MREQ̄ and RD̄ were active and regular;
- the capture was consistent with continued forced-NOP execution.

## Interpretation

### Observed fact

The clock, address bus and captured control lines all showed coherent activity after the clock-module rebuilding and STEP-debounce work.

### Deduction

The current RUN-mode baseline is therefore restored: the rebuilding did not introduce a persistent fault into the normal forced-NOP execution path.

## Transient selector anomaly

During debugging, a manual STEP action appeared briefly to influence the clock while RUN was selected. Because many temporary leads had been moved during the investigation, the propagation path was checked directly rather than assuming a selector fault.

Measured:

- SN74LS157 pin 1: 0 V in MANUAL and approximately 3.6 V in RUN;
- SN74LS157 pin 4: no STEP-correlated change in RUN;
- SN7400 pin 9: no STEP-correlated change in RUN;
- SN7400 pin 11: no STEP-correlated change in RUN.

The anomaly could not be reproduced after the wiring was stabilised. It is retained in the record as a debugging/probing artefact rather than promoted into a circuit fault.

## Evidence

The corresponding healthy RUN-mode DSView capture from 19 September 2026 was also recreated as a print-friendly annotated figure using the same white-background annotation style as the earlier CPU bring-up figure.

The figure explains:

- the Parallel decoder output;
- A0–A7 address-line behaviour;
- M1̄, MREQ̄ and RD̄;
- the CPU CLK reference;
- why the observed pattern is consistent with forced-NOP address progression.

Raw analyser captures and the annotated print figure should be preserved under `evidence/logic-analyser/` when binary evidence files are committed.