# Athena clock schematic

Open `Athena.kicad_pro` and enter the **Clock** sheet. `exports/Athena-clock.pdf` is the complete clock drawing on one A3 page.

The schematic now contains both halves of U1 (RS 305-838 / NE556), all five units of U2 (TI SN7400), U3 (TI SN74LS157N), both timing networks, STEP and RUN/MANUAL controls, HLT gating, and only the two required LEDs. Signal paths are drawn with continuous wires. A dot at a crossing means connected; crossings without dots are not connected. Matching +5 V and ground symbols denote the common supply rails. PWR_FLAG is an electrical-check annotation, not a physical part.

## Defined connections

- U1 pin 5 feeds U3 pin 3 (astable); U1 pin 9 feeds U3 pin 2 (manual).
- U2 pin 6 feeds U3 pin 1: LOW selects MANUAL, HIGH selects ASTABLE.
- U3 pin 4 feeds U2 pin 9. U2 pin 8 feeds U2 pins 12 and 13.
- U2 pin 11 is the external CPU_CLK output. No connector type or terminal numbering is assumed.
- U2 pin 10 is CLOCK_ENABLE. Closing SW3 pulls it LOW, lights D1 and holds CPU_CLK LOW.
- U3 pin 15 and unused data inputs are grounded; unused outputs are marked unconnected.
- U1A/B are one physical DIP-14 IC. U2A-E are one physical DIP-14 IC. U3 is DIP-16.

## Details still requiring confirmation

| Reference/detail | Meaning |
|---|---|
| R4, R5 | Fitted RUN/MANUAL pull-up resistances unknown: TBC |
| R7, R8 | Fitted HLT and astable LED series resistances unknown: TBC |
| D2 | Signal is confirmed as U1 pin 5; complete LED wiring/polarity to GND shown provisionally |
| RV1 unused end | Shown open; confirm whether strapped to the wiper |
| SW2 | Functional contacts shown; physical switch terminal numbering/type needs confirmation |
| C3, C4, C5 | Local supply bypass provisions for U1, U2, U3 respectively; fitted presence and values TBC |
| U3 unused data inputs | Grounded as required by the wiring record; verify actual breadboard connections |

U1 control pins 3 and 11 are externally unconnected in this drawing. Optional control bypass capacitors are not shown as fitted. The unknown details remain explicit rather than being guessed. Resolve them before using this as a definitive as-built record or final build specification. Footprints and PCB layout are outside this schematic update.

## Verification

The core connections follow `../../breadboard/clock/wiring.md`, `../../../docs/design-decisions/clock-architecture.md` and the user's confirmed values. All 44 IC pins were checked in the exported netlist, including power/reset, both timing networks, latch feedback, selector routing, HLT and LED connections. Actual pin membership was checked against expected sets to detect unintended joins.

KiCad **10.0.3** loaded the schematic in the existing Athena hierarchy. ERC reports **0 errors and 1 warning**: CPU_CLK is connected only to U2 pin 11 because the receiving CPU circuit is outside this module. This warning remains visible; no check was disabled to hide it. See `exports/Clock-erc.txt`.

The final PDF was visually inspected. These checks establish the drawing's connectivity and readability, not physical loading, timing or glitch-free operation. The journal contains 556 pin measurements through pin 14; the full selector/output path still requires the planned bench recheck.

The original top-level schematic and existing project settings were retained. Journal content was not edited as part of this work.

## Logic diagram

The **Clock logic** hierarchical sheet (`Clock-logic.kicad_sch`) shows source selection, mode memory, both HLT NAND gates and the Boolean relationships. `exports/Clock-logic.pdf` is its A4 preview. This sheet contains documentation graphics only and is excluded from the BOM, board and simulation. Its U1/U2/U3 annotations refer to the existing physical ICs on the Clock circuit sheet; it adds no duplicate parts or electrical nets.
