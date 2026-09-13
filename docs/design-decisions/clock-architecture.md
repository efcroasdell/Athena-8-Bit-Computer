# Clock Architecture Decision Record

Status: **current architectural decision**

## Decision

Athena's clock module will retain the functional behaviour of the Ben Eater 8-bit computer clock while using a more compact implementation based on:

- one RS 305-838 dual 556 timer
- one TI SN7400 standard TTL quad NAND gate
- one TI SN74LS157N quad 2-to-1 multiplexer

This is an Athena decision. It is not a Phoenix architectural rule.

## Required external behaviour

The clock module must provide:

- variable-speed free-running clock
- manual single-step operation
- persistent RUN/MANUAL mode selection
- clean clock-source selection
- HLT-controlled clock gating
- final CPU clock output
- observable intermediate nodes for measurement and diagnosis

## Functional allocation

### 556 Timer 1 — astable source

Timer 1 produces the free-running variable-speed clock. The timing capacitor repeatedly traverses the internal trigger and threshold reference levels, approximately one-third and two-thirds of VCC. The internal latch and discharge transistor create the repeating charge/discharge cycle.

### 556 Timer 2 — manual monostable

Timer 2 produces one manual clock pulse for each active-LOW STEP trigger. Its timing interval is determined by the external 1 MΩ / 1 µF timing network.

### SN7400 gates 1 and 2 — RUN/MANUAL memory

The first two NAND gates are cross-coupled to form an SR latch. The latch stores the selected operating mode rather than requiring the selector switch itself to carry the clock signal.

The chosen convention is:

- SN7400 pin 6 = Q
- SN7400 pin 3 = complementary output

Q drives the SELECT input of the SN74LS157.

### SN74LS157 — clock-source selection

One multiplexer channel selects between:

- A = manual pulse from 556 pin 9
- B = astable clock from 556 pin 5

with Q from the SN7400 latch controlling SELECT.

The intended selector relationship is:

`Y = A·Q̄ + B·Q`

The active-LOW strobe is tied LOW so the selector remains enabled.

### SN7400 gates 3 and 4 — HLT gating

The remaining two NAND gates provide the final HLT gating function. The two-gate arrangement restores the selected clock to the intended polarity at the final output.

Verified behaviour:

- HLT control HIGH → selected clock propagates to final clock output
- HLT control LOW → final clock output is held LOW

SN7400 pin 11 is the current final internal CPU clock output.

## Why this architecture was chosen

The design keeps the full educational value of the original clock while reducing package count and making the signal path explicit.

The implementation exposes several distinct concepts in one small subsystem:

- analogue RC timing
- internal comparator thresholds
- SR-latch state
- TTL NAND logic
- multiplexing
- active-LOW control
- clock gating
- propagation through multiple logic stages
- oscilloscope verification of analogue and digital behaviour

The 556 is particularly useful because it places the astable and monostable functions in one package while preserving the same internal timer mechanisms that can be observed and understood individually.

## Rejected or superseded approaches

### Multiple 555 timers

The original learning path used separate 555 timers. This remains historically important and is retained in the archive, but it is not the current Athena clock architecture.

Reason for supersession: the dual 556 implements the two required timer functions in one package without hiding the timing mechanism.

### Treating the RUN/MANUAL control as direct clock switching

Rejected. Mode state is retained in the NAND latch and then applied to the multiplexer. This makes the state explicit and independently observable.

### Moving the clock function into FPGA logic

Not part of the current Athena clock milestone. The purpose of this module is to understand and measure the physical timer, TTL, switching and gating behaviour directly.

## Observability requirement

The following points should remain practically observable during development:

- 556 astable timing node
- 556 astable output
- 556 monostable trigger
- 556 monostable output
- RUN/MANUAL latch outputs
- LS157 SELECT
- LS157 selected clock output
- HLT control
- final CPU clock

Observability is a design objective, not an incidental debugging convenience.

## Verification requirement

The architecture is not considered fully verified merely because the final clock appears to work. Each stage must be checked against its expected behaviour and, where useful, against the internal mechanism that produces that behaviour.

Current systematic verification is proceeding pin-by-pin through the dual 556 before the complete selector and final clock path is rechecked end-to-end.
