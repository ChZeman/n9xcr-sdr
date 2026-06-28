# Band-select switch — parts list (DigiKey)

The one TX band-select board: a **pSemi PE42512A** absorptive SP12T (9 kHz–8 GHz), switched at exciter
level (~+19 dBm / 80 mW, cold-switch only) ahead of the per-band chains. **All thirteen RF ports get
SMA** so the board is fully VNA-testable and re-routable. Baseline is **normal mode** (VSS_EXT→GND,
internal −V generator); the spur-free **bypass-mode** option is at the end. The control MCU is
**off-board** (see note). Port map, decode table, and board-layout notes live in `bom.md`
("Band-select build"); schematic: `../diagrams/band-select-schematic.svg`.

## Switch IC

| Ref | Part | DigiKey PN | Qty | Note |
|-----|------|-----------|----:|------|
| U1 | pSemi PE42512A SP12T, 32-QFN 5×5 | `PE42512A-X` | 1 (+2–4 spare) | Exposed pad → GND. ~$5–12 (verify on the DigiKey page — listings disagree). "A" rev is current; base PE42512 is EOL and the QFN is hard to rework, so keep spares. |

## Supply decoupling — normal mode (VSS_EXT → GND)

| Ref | Value | DigiKey PN | Qty | Note |
|-----|-------|-----------|----:|------|
| C1 | 0.1 µF X7R 0402 50 V | `GRM155R71H104KE14J` | 1 | At the VDD pin. |
| C2 | 1 µF X5R 0603 25 V | `CC0603KRX5R8BB105` | 1 | ∥ C1. Yageo, DigiKey-direct cut-tape. |
| C3 | 10 µF X5R 0805 25 V | `CC0805KKX5R8BB106` | 1 | Optional rail bulk. Yageo, DigiKey-direct cut-tape (25 V; 16 V `CC0805KRX5R7BB106` is a fine fallback). |

## Control interface — V1–V4 from an off-board 3.3 V MCU (no level shifter)

| Ref | Value | DigiKey PN | Qty | Note |
|-----|-------|-----------|----:|------|
| R1–R4 | 10 kΩ 0402 | `RC0402FR-0710KL` | 4 | Pull-downs on V1–V4 → defined power-up state. |
| R5 | 0 Ω 0402 | `RC0402JR-070RL` | 1 | LS → GND. Ground straight to plane (a good RF ground on LS improves IL/isolation). 0 Ω leaves the option to lift LS to a GPIO for the all-isolated park. |
| J_ctrl | 1×6 100-mil header | (generic) | 1 | V1·V2·V3·V4·3V3·GND to the control board (JST-SH 6-pin is a compact alt). |

**MCU is off-board.** It's driven by the enclosure **control/monitor board** — the same board doing
telemetry, T-R sequencing tie-in, and PA bias keying — not a chip on this PCB. The band decision must
stay in sync with the allowed-TX table and the sequencing wrapper (all in the host), and an on-board
ESP32 would put a 2.4 GHz radio right next to the 13 cm / 9 cm ports. If a self-contained module is ever
wanted, use a **radio-less** MCU (RP2040 / STM32) on a daughter-board off the RF copper — never an ESP32
on this board.

## RF ports — all 13 populated

| Ref | Part | DigiKey PN | Qty | Note |
|-----|------|-----------|----:|------|
| RFC, RF1–RF12 | Amphenol RF edge-launch SMA jack, 0.062″ | `901-10513-1` | 13 | RFC = exciter in; RF1–RF12 = the twelve slices. Same SMA as the BPF boards. |

The PE42512A is **absorptive** — every OFF port is internally 50 Ω-terminated, so unused/unbuilt ports
need **no external load**. Optional failsafe: thread a screw-on 50 Ω SMA load (e.g. Mini-Circuits
`ANNE-50+`, DC–6 GHz) onto unbuilt or park ports so a mis-selected port dumps into 50 Ω instead of an
open. These are screw-on accessories, **not board parts** — buy only if wanted.

## PCB

Custom, laid out from the SnapMagic `PE42512A-X` footprint. **2-layer FR-4 through 928 MHz**; go 4-layer
or RO4350B only if the >1 GHz ports (23 cm at 1.3 GHz, or the µwave slices) are populated. Fab
JLCPCB/PCBWay. Qty 1 (+1 spare bare board).

## Optional — bypass mode (spur-free TX)

Normal mode runs the internal −V charge pump, which can drop a small spur onto the routed signal. To
kill it, disable the pump by feeding **VSS_EXT a clean −3.3 V** instead of grounding it:

- **Easiest:** borrow the −3.3 V already generated on the RX side (the ADM8829 feeding the ADRF5040s);
  route it over and add a local 1 µF (`CC0603KRX5R8BB105`) reservoir at the VSS_EXT pin.
- **Standalone:** add an Analog Devices `ADM8829ARTZ-REEL7` switched-cap inverter + 2× 1 µF (flying +
  reservoir) — the same inverter already used on RX.

Only VSS_EXT changes (GND → −3.3 V); R5 still grounds LS, everything else is unchanged.

## Totals (normal mode, all ports populated)

- **1** PE42512A-X (+2–4 spare) · **2–3** caps · **5** resistors · **1** header · **13** SMA · **1** PCB.
- Switch ~$5–12; passives + connectors well under $20. Optional 50 Ω SMA loads priced separately.
