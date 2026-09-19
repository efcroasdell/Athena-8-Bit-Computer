# Athena clock schematic

Open `Athena.kicad_pro` and enter the **Clock** sheet. `exports/Athena-clock.pdf` is the complete clock drawing on one A3 page.

The schematic now contains both halves of U1 (RS 305-838 / NE556), all five units of U2 (TI SN7400), U3 (TI SN74LS157N), U4 (TI CD40106BE), both timing networks, the RC/Schmitt STEP-debounce network, RUN/MANUAL controls, HLT gating, and only the two required LEDs. Signal paths are drawn with continuous wires. A dot at a crossing means connected; crossings without dots are not connected. Matching +5 V and ground symbols denote the common supply rails. PWR_FLAG is an electrical-check annotation, not a physical part.

## Defined connections

- U4 pin 1 is the debounce RC node: R3 = 47 kΩ to +5 V, C6 = 100 nF to GND, and R9 = 1 kΩ in series with SW1 to GND.
- U4 pin 2 feeds U4 pin 3; U4 pin 4 feeds U1 pin 8. This gives two Schmitt inversions and preserves the active-LOW STEP trigger polarity.
- U4 pin 14 is connected to +5 V and pin 7 to GND. Unused U4 inputs are tied to GND and unused outputs are marked unconnected.

- U1 pin 5 feeds U3 pin 3 (astable); U1 pin 9 feeds U3 pin 2 (manual).
- U2 pin 6 feeds U3 pin 1: LOW selects MANUAL, HIGH selects ASTABLE.
- U3 pin 4 feeds U2 pin 9. U2 pin 8 feeds U2 pins 12 and 13.
- U2 pin 11 is the external CPU_CLK output. No connector type or terminal numbering is assumed.
- U2 pin 10 is CLOCK_ENABLE. Closing SW3 pulls it LOW, lights D1 and holds CPU_CLK LOW.
- U3 pin 15 and unused data inputs are grounded; unused outputs are marked unconnected.
- U1A/B are one physical DIP-14 IC. U2A-E are one physical DIP-14 IC. U3 is DIP-16. U4A-G are one physical DIP-14 CD40106BE.

## Details still requiring confirmation

| Reference/detail | Meaning |
|---|---|
| R4, R5 | Fitted RUN/MANUAL pull-up resistances unknown: TBC |
| R7, R8 | Fitted HLT and astable LED series resistances unknown: TBC |
| D2 | Signal is confirmed as U1 pin 5; complete LED wiring/polarity to GND shown provisionally |
| RV1 unused end | Shown open; confirm whether strapped to the wiper |
| SW2 | Functional contacts shown; physical switch terminal numbering/type needs confirmation |
| C3, C4, C5 | Local supply bypass provisions for U1, U2, U3 respectively; fitted presence and values TBC |
| U4 local decoupling | Existing 100 nF rail decoupling is present close to the device on the breadboard; exact schematic reference/fitted placement remains to be reconciled if a dedicated capacitor reference is wanted |
| U3 unused data inputs | Grounded as required by the wiring record; verify actual breadboard connections |

U1 control pins 3 and 11 are externally unconnected in this drawing. Optional control bypass capacitors are not shown as fitted. The unknown details remain explicit rather than being guessed. Resolve them before using this as a definitive as-built record or final build specification. Footprints and PCB layout are outside this schematic update.

## Verification

The core connections follow `../../breadboard/clock/wiring.md`, `../../../docs/design-decisions/clock-architecture.md` and the user's confirmed values. All 44 IC pins were checked in the exported netlist, including power/reset, both timing networks, latch feedback, selector routing, HLT and LED connections. Actual pin membership was checked against expected sets to detect unintended joins.

KiCad **10.0.3** loaded the schematic in the existing Athena hierarchy. ERC reports **0 errors and 1 warning**: CPU_CLK is connected only to U2 pin 11 because the receiving CPU circuit is outside this module. This warning remains visible; no check was disabled to hide it. See `exports/Clock-erc.txt`.

The selector/output path was rechecked on the bench on 19 September 2026 and behaved correctly. The Clock schematic and Clock-logic sheet have now been revised to include the CD40106B STEP-debounce stage. The previously exported PDF/ERC files pre-date this revision and must be regenerated in KiCad 10.0.3 before they are treated as current verification artefacts.

The original top-level schematic and existing project settings were retained. Journal content was not edited as part of this work.

## Logic diagram

The **Clock logic** hierarchical sheet (`Clock-logic.kicad_sch`) now also identifies the CD40106B STEP-conditioning stage before the manual 556 path, alongside source selection, mode memory, both HLT NAND gates and the Boolean relationships. `exports/Clock-logic.pdf` is its A4 preview. This sheet contains documentation graphics only and is excluded from the BOM, board and simulation. Its U1/U2/U3 annotations refer to the existing physical ICs on the Clock circuit sheet; it adds no duplicate parts or electrical nets.


## 19 September 2026 schematic update

The Clock sheet has been revised to match the current physical STEP path:

- CD40106B pin 1 → 47 kΩ → +5 V;
- CD40106B pin 1 → 100 nF → GND;
- CD40106B pin 1 → 1 kΩ → STEP switch → GND;
- CD40106B pin 2 → pin 3;
- CD40106B pin 4 → U1 pin 8;
- CD40106B pin 14 → +5 V;
- CD40106B pin 7 → GND;
- unused CD40106B inputs are tied to GND; unused outputs are marked unconnected.

The former direct 10 kΩ pull-up / STEP-switch connection at U1 pin 8 is superseded. R3 is now 47 kΩ, C6 is 100 nF and R9 is 1 kΩ.

Bench verification on 19 September 2026 established that the conditioning stage produces a clean trigger at U1 pin 8 and one clean CPU clock pulse per deliberate STEP action. In RUN, STEP produced no observable disturbance at U3 pin 4, U2 pin 9 or U2 pin 11.

**Export status:** the editable KiCad schematic files are current, but the committed PDF preview and ERC report still represent the earlier revision until regenerated in KiCad 10.0.3.
