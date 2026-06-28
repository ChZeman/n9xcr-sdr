# BPF consolidated parts list (DigiKey)

Covers **one each of the four TX cleanup BPF boards** (2 m, 222, 70 cm, 902). Build = fixed Coilcraft
chip inductors + **two** Knowles Johanson Giga-Trim sapphire trimmers per board (C2 sets the low corner,
C4 the high corner); C1/C3/C5 fixed C0G. Multiply quantities if you build spares. Inductor order codes
are ±5 % (`…XJRW`) — choose **Cut Tape** at DigiKey; use `…XGRW` for ±2 %.

## Inductors — Coilcraft (cut tape, qty 1)

| DigiKey PN | Value | Qty | Positions |
|------------|-------|----:|-----------|
| `0603DC-68NXJRW` | 68 nH | 4 | 2 m L1,L3,L4,L5 |
| `0603DC-39NXJRW` | 39 nH | 1 | 2 m L2 |
| `0603DC-43NXJRW` | 43 nH | 2 | 222 L1,L3 |
| `0603DC-27NXJRW` | 27 nH | 1 | 222 L2 |
| `0603DC-47NXJRW` | 47 nH | 2 | 222 L4,L5 |
| `0603DC-20NXJRW` | 20 nH | 4 | 70 cm L1,L3,L4,L5 |
| `0603DC-11NXJRW` | 11 nH | 1 | 70 cm L2 |
| `0402DC-9N1XJRW` | 9.1 nH | 4 | 902 L1,L3,L4,L5 |
| `0402DC-4N7XJRW` | 4.7 nH | 1 | 902 L2 (5.0 nH target) |

20 inductors. DigiKey 0402DC ±5 % stock is spotty per value: `9N1` is stocked, but **neither `5N1` nor `5N0` is**, so L2 (5.0 nH target) uses the nearest in-stock value — `4N7` (4.7 nH) shown, or `5N6` (5.6 nH); the C2 trimmer absorbs the few-percent difference. For exact 5.0 nH, order `0402DC-5N0` from **Coilcraft direct** (same-day, free samples), or wind L2 air-core (preferred at 902 anyway). Confirm live stock per value at order time — it shifts.

## Trim caps — Knowles Johanson Mfg, sapphire Giga-Trim (qty 1)

| DigiKey PN | Range | Qty | Positions |
|------------|-------|----:|-----------|
| `5602` | 1–30 pF | 3 | 2 m C2; 222 C2,C4 |
| `5201` | 0.8–10 pF | 3 | 2 m C4; 70 cm C2; 902 C4 |
| `5761` | 0.6–6 pF | 1 | 902 C2 |
| `5502` | 1–20 pF | 1 | 70 cm C4 |

8 trimmers (2 per board). ≈ $400 total — the bulk of the build cost. Pick the surface-mount variant of
a range to match the board (1–30 pF: `5601` side-SMT / `5641` top-SMT vs panel-mount `5602`).

## Fixed caps — Murata GRM18 C0G, 0603, 50 V, ±5 % (qty 1)

| DigiKey PN | Value | Qty | Positions |
|------------|-------|----:|-----------|
| `GRM1885C1H220JA01D` | 22 pF | 3 | 2 m C1,C3,C5 |
| `GRM1885C1H330JA01D` | 33 pF | 1 | 2 m C4 (bulk, parallel w/ 5201) |
| `GRM1885C1H150JA01D` | 15 pF | 3 | 222 C1,C3,C5 |
| `GRM1885C1H6R8DA01D` | 6.8 pF | 3 | 70 cm C1,C3,C5 |
| `GRM1885C1H3R0CA01D` | 3.0 pF | 3 | 902 C1,C3,C5 |

13 fixed caps. At 902 a high-Q part (Murata GQM18 0603 or GJM15 0402) is the upgrade for the three
fixed caps; standard C0G works. Confirm small-value (<10 pF) tolerance suffix at checkout.

## Connectors & board

| PN | Description | Qty |
|----|-------------|----:|
| `901-10513-1` | Amphenol RF edge-launch SMA jack, 0.062″ | 8 |
| — | FR-4 PCB, custom (JLCPCB / PCBWay) | 4 |

## Totals

- **DigiKey line items:** 9 inductor + 4 trimmer + 5 fixed cap + 1 connector = **19**
- **DigiKey parts:** 20 inductors + 8 trimmers + 13 fixed caps + 8 SMA = **49** (+ 4 PCBs, fabbed)
- **Cost is dominated by the 8 sapphire trimmers (~$400);** everything else totals under ~$100.

## Bench insurance (optional)

Order one adjacent E24 inductor value per position (e.g. a 62 nH and a 75 nH beside each 68 nH) and a
couple of spare trimmers. If a board's return loss comes out poor on two-knob tuning, adding a trimmer
to one more element (C1 or C3) is the fallback. Per-band schematics: `../diagrams/bpf-<band>-schematic.svg`.
