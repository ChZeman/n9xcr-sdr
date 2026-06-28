# Bill of materials

Consolidated part list for a VHF/UHF node. "Firm" = decided; "Open" = still needs a part picked.

> **Region note:** the bands, filters, and antennas below are the US / IARU Region 2 worked example. The design is region-agnostic - see [`regions.md`](regions.md) to adapt bands, filters, finals, and antennas to Region 1 or 3.

## Firm

| Item | Part | Notes |
|------|------|-------|
| VHF/UHF SDR | OpenSourceSDRLab 7020-SDR (PlutoSky R2), AD9361 + onboard PA | Buy direct from opensourcesdrlab.com (PlutoSky R2 7020) and select the **"9361 with PA"** variant (~$220) — not the "9363" variants, and not the $120 AD9363 "Professional Edition". Aluminum case is a separate ~$18 item (match the shell to a 9361-with-PA board). AliExpress "7020-SDR AD9361 with case" (~$200.71) is the cheaper equivalent — confirm the listing says *with PA*. AntSDR E200 (Mouser `ANTSDR-AD9361-With-CASE-01`) is the turnkey alternative. AD9361 not AD9363; onboard PGA-102+ PA (~+15–19 dBm) covers the pre-driver; runs plutosdr-fw / libiio. |
| HF SDR | Hermes-Lite 2 | example; any HPSDR-class HF SDR |
| Digital-voice vocoder | DVMEGA DVstick 30 | AMBE-3000; ham dealer (e.g. GigaParts), not a component distributor |
| Final, VHF / low-UHF (ex: 4 m / 2 m / 222) | NXP MRF101AN | 100 W LDMOS, 1.8–250 MHz; run at 50 W |
| Final, UHF (ex: 70 cm / 902 / 23 cm) | Wolfspeed/MACOM CGH40120F | 120 W GaN, 28 V; one part both bands; run at 50 W |
| Pre-driver | onboard PGA-102+ on the SDR | ~+15–19 dBm out; no separate board (discrete PGA-103+ optional if the onboard PA is bypassed) |
| Driver | Wolfspeed/MACOM CGH40010 (F flange / P pill) | 10 W GaN (13 W PSAT), DC–6 GHz, 28 V; one part type, 4 builds (per band); runs ~30 % for linear |
| TX band-select | pSemi PE42512A (UltraCMOS SP12T) | silicon SP12T, absorptive, 9 kHz-8 GHz, **single 3.3 V supply** (internal -V gen; VSS_EXT->GND), 4-bit V1-V4 + LS control; low-level switch ahead of the per-band chains. Order code `PE42512A-X` (DigiKey, ~$15-20). Base PE42512 is EOL - buy ~4 spares. See "Band-select build (PE42512A)" below. |
| RX front-end switches | Analog Devices ADRF5040 (x2) | SP4T switch-filter-switch in the RX preselector carrier (unchanged); needs +3.3 V & -3.3 V via a shared ADM8829. See `rx-frontend.svg`. |
| RX preselector, 2 m | DCI-146-4H | 4-pole cavity, 144–148 |
| RX preselector, 222 | Temwell helical | 222–225 |
| RX preselector, 70 cm | Temwell helical | 420–450 |
| RX preselector, 902 | FreeWave EBF901 (or Fairview cavity) | 902–928 |
| RX quick-start | Nooelec Ham Filter Bank | one module, all four bands, RX-only; validate chain before cavities |
| Antenna, 2m/440/900 | Tram tri-band NMO wideband | via triplexer |
| Antenna, 222 | Comet SBB-224NMO | dedicated whip |
| TX low-pass bank | 4 filters (2 m / 222 / 70 cm / 902) | 9-element Chebyshev; build or adapt published boards; see `filters.md` |
| Reference | 10 MHz GPSDO | one per site; programmable type futureproofs 122.88 MHz |
| Control wrapper | amplifier control board + dual directional detector + relay sequencer + heatsink thermistor | see `tx-chain.md` |
| Node host | small PC (Pi 5 / x86 mini-PC) | runs SDRangel headless + digital apps + agent |

## Open

| Item | Candidate / status |
|------|--------------------|
| **PE42512A band-select PCB** | **custom board still to be designed** — lay out from the SnapEDA/SnapMagic `PE42512A-X` footprint against `diagrams/band-select-schematic.svg`; FR-4 2-layer through 928 MHz (4-layer only if >1 GHz ports are populated); no usable eval/breakout exists. **Not started** — deferred to a later layout pass. |
| GPSDO unit | e.g. Leo Bodnar Mini Precision GPSDO — confirm frequency range |
| T-R relays + sequencer | off-the-shelf control boards (W6PQL) — leading source |
| Triplexer 2m/440/902 | sourcing risk — verify a real 900 MHz transmit-rated port |
| Wideband RX antenna | discone-class to ~1.3 GHz — element TBD |
| FM trap (88–108 band-stop) | commodity, unselected |
| Bypassable LNA | commodity, unselected |
| Node-host model | Pi 5 vs x86 mini-PC — sizing pending |

## Notes

- The MRF101AN tops out at 250 MHz, so it serves 2 m and 222 only — 70 cm and 902 are the GaN
  (CGH40120F) builds.
- Avoid cheap eBay "100 W" LDMOS amps (counterfeit MRF9120) and constant-envelope FM modules
  (e.g. Mitsubishi RA-series) — both are wrong for an all-mode linear station.
- Whatever bundled filter ships on any amp board, verify harmonic suppression on an analyzer
  (−43 dBc required); do not assume.
- **DigiKey single-sourcing.** DigiKey-stocked: the active devices (CGH40010F driver, CGH40120F +
  MRF101AN finals — both finals confirmed in stock, MACOM/Wolfspeed; PE42512A switch) and all
  passives / connectors / enclosures (Hammond) / PSUs (Mean Well), including the **BPF parts** (high-Q
  C0G/NP0 caps — Johanson / Murata GJM-GQM / KYOCERA AVX / Knowles; Coilcraft chip inductors or magnet
  wire; edge-launch SMA). **Not on DigiKey — order direct / ham-dealer:** 7020-SDR (opensourcesdrlab /
  AliExpress), DVstick 30 (GigaParts), antennas (Tram / Comet), GPSDO (Leo Bodnar), W6PQL control /
  sequencer / SWR boards, RX cavities (DCI / Temwell / FreeWave). **Custom PCBs** (band-select, BPF
  boards) go to a fab house (JLCPCB / PCBWay), not DigiKey — or build the BPFs on copper-clad
  (copper-clad stock is on DigiKey).

## Band-select build (PE42512A - TX)

Parts for the one TX band-select SP12T. Unlike the ADRF5040, the PE42512A is **single-supply** -
tie VSS_EXT (pin 10) to GND and its internal negative-voltage generator does the rest. No ADM8829,
no -3.3 V rail, and **no RF DC-block caps** (every RF pin sits at 0 VDC). Schematic:
`diagrams/band-select-schematic.svg`.

| Item | Part | Source (approx) | Notes |
|------|------|-----------------|-------|
| SP12T switch | **PE42512A-X** | DigiKey (`PE42512A-X`, 500/T&R) ~$15-20 | active "A" revision; base PE42512 is discontinued. Buy ~4 spares - pSemi is phasing out its broadband-switch line. Confirm DigiKey "Part Status: Active". |
| Supply decoupling | 0.1 uF + 1 uF to GND | with order | on VDD (3.3 V, IDD ~200 uA) |
| VSS_EXT (pin 10) | tie -> GND | - | normal mode = internal -V gen ON; ~5 MHz spur is irrelevant at VHF/UHF. Bypass mode (-3.0 V) optional for spur-free, unused. |
| LS (pin 32) | tie -> GND | - | selects the LS=0 decode map; grounding LS also improves IL/isolation. Internal 1 MOhm pull-up makes a floating LS read high - tie it deliberately. |
| Control | 4 x GPIO (V1-V4) | from control MCU | 3.3 V CMOS; nothing to buy |
| RF DC-blocks | **none** | - | not required (RF ports at 0 VDC) |
| PCB | small custom 2-layer (or 4-layer) board | JLCPCB / PCBWay | lay out from the SnapEDA/SnapMagic **PE42512A-X** footprint+symbol (KiCad/Altium/PADS/Eagle). The pSemi eval Gerbers are a 12-SMA *characterization* board - reference only, not the node board. |
| Connectors (standalone only) | SMA/edge-launch as needed | with order | omit if integrated onto the TX board |

**Port map** - each port is a 1.5:1 sub-octave slice spanning 4 m (70 MHz) to 5 cm (5.8 GHz); the
amateur band that lands in each slice is tagged. Loss-tiered: RF1/RF12 (lowest loss) carry the
highest bands. Populate only the ports you build - each band is a full chain (BPF -> driver -> final
-> LPF); the switch only routes.

| Port | Loss rank | Slice (MHz) | Amateur band |
|------|-----------|-------------|--------------|
| RF1 | 1 (lowest) | 4050-6075 | 5 cm (5.65-5.925 GHz) |
| RF12 | 1 | 2700-4050 | 9 cm (3.3-3.5 GHz) |
| RF2 | 2 | 1800-2700 | 13 cm (2.3-2.45 GHz) |
| RF11 | 2 | 1200-1800 | 23 cm (1240-1300) |
| RF3 | 3 | 800-1200 | 33 cm (902-928) |
| RF10 | 3 | 533-800 | - none |
| RF4 | 4 | 355-533 | 70 cm (420-450) |
| RF9 | 4 | 237-355 | - none |
| RF5 | 5 | 158-237 | 1.25 m (222-225) |
| RF8 | 5 | 105-158 | 2 m (144-148) |
| RF6 | 6 | 70-105 | 4 m (70-70.5) |
| RF7 | 6 (highest) | dummy / spare | low-level 50 Ohm test / park |

- Spans 4 m (70 MHz) to 5 cm (5.8 GHz) - the AD9361 TX range; 6 m and below ride the HF SDR.
- RF9 (237-355) and RF10 (533-800) slices have no current amateur allocation - spare. RF7 = low-level
  dummy / park.
- **Decode (LS=0, bits V4 V3 V2 V1):** RF1 `0000`, RF2 `1000`, RF3 `0100`, RF4 `1100`, RF5 `0010`,
  RF6 `1010`, RF7 `0110`, RF8 `1110`, RF9 `0001`, RF10 `1001`, RF11 `0101`, RF12 `1101`;
  all-isolated **park** (TX-inhibit) = `0011`.
- Cold-switch only (no TX during a band change); hot-switch limit is 20 dBm above 100 MHz and the
  exciter is ~+19 dBm - another reason the band-select sits *before* the drivers.

### Why PE42512A over the ADRF5040 (TX path)

- **Single supply** - internal -V generator, so no ADM8829 and no -3.3 V rail.
- **No DC-block caps** (RF ports at 0 VDC).
- **8 GHz coverage** - never the band limit again; covers any future band the AD9361 could TX.
- Trade-offs: 4 control bits vs 2, a custom board (the ADI eval-gerber path is dropped), and EOL
  risk (mitigated by buying spares now). RF penalty at <1 GHz is negligible (<1 dB IL, 40-60 dB iso).

The RX front-end is **unchanged** - still 2 x ADRF5040 SP4T (switch-filter-switch) with a shared
ADM8829 -3.3 V rail; see `rx-frontend.svg`. The ADRF5040 + ADM8829 sourcing is retained below for it.

### Board: custom PE42512A layout (chosen)

The ADI ADRF5040 eval-gerber fab path is **dropped** for the TX band-select (PCBWay's CAM kept
rejecting the old `.pho`-format files, and it was a 12-SMA characterization board anyway). Instead,
lay out a small custom board:

- **Footprint/symbol:** SnapEDA / SnapMagic `PE42512A-X` (verified; exports to KiCad, Altium, PADS,
  Eagle, OrCAD): <https://www.snapeda.com/parts/PE42512A-X/pSemi/view-part/>
- **Reference only:** pSemi publishes eval-board Gerbers + Schematic/B.O.M. on the PE42512 product
  page, but that board is a 12-SMA de-embedding fixture - copy its RF ground/transition style, do
  not fab it as the node board.
- **Stackup:** FR-4 2-layer is fine to 928 MHz for the real bands; go 4-layer / controlled-impedance
  only if the speculative >1 GHz ports are ever populated.
- Generate clean RS-274X Gerbers straight from the EDA tool - no `.pho` / old-format headache.

### RX-only: ADRF5040 + ADM8829 sourcing (retained)

The RX front-end's two SP4Ts still use these parts (one shared -3.3 V rail for the co-located pair):

| Item | Part | Source (approx) | Notes |
|------|------|-----------------|-------|
| SP4T switch | **ADRF5040BCPZ** | Mouser / DigiKey ~$32 | strip (MOQ 1); not `-R7` reel, not `-EVALZ` |
| -3.3 V generator | **ADM8829ARTZ-REEL7** | DigiKey ~$3 | SOT-23-6 charge-pump inverter; +3.3 V in -> -3.3 V out to VSS |
| Charge-pump caps | 2 x 1 uF | with order | C1 flying, C2 reservoir |
| Supply decoupling | 100 nF + bulk (1-10 uF) | with order | on VDD and VSS to GND |
| RF DC-blocks | ~1 nF C0G x 5 (each switch) | with order | RFC + RF1-RF4 |

## BPF passives — DigiKey sourcing

**Build pattern: fixed Coilcraft chip inductors + trimmer caps.** The inductors sit at the nearest E24
value and stay put; each filter is brought to spec by adjusting the **capacitors only**. A trimmer on
the cap side pulls each LC corner onto frequency even with L at a standard step (the corner tracks the
LC *product*), and the value rounding is small (≤~5 %: 66→68, 44→43, 46→47) — well within trimmer
travel. You do not swap inductors during tuning.

- **Inductors → fixed Coilcraft `0603DC` chip**, nearest E24 value (PNs per board below). 0603DC is
  wirewound, the highest-Q 0603 series (up to ~40 % higher Q than the older `0603HP`, and lower DCR),
  and — unlike `0603HP`, which DigiKey factory-orders only — it is **DigiKey-stocked in cut tape
  (qty 1, ships today)**. Order code `0603DC-<val>XJRW` (±5 %, fine for a trimmed filter) or `…XGRW`
  (±2 %). At 902 use `0402DC` (or just wind air-cores, preferred there). SRF clears every band. *Avoid
  Abracon `AISC-0603HP-…` lookalikes* — same name, but not stocked and tape-&-reel of 3 000 only.
  *Insurance:* order one adjacent E24 value per position (e.g. a 62 nH and a 75 nH alongside each
  68 nH) so a stubborn section can be stepped rather than fought — rarely needed. Coilcraft also ships
  direct (same-day, free samples, small qty) if you'd rather not chase per-value DigiKey stock.
- **Tuning caps → Knowles Johanson Giga-Trim sapphire piston trimmers** (Q>3000 @ 250 MHz, sapphire
  dielectric, near-zero tuning noise, DigiKey-stocked, numeric PNs). Pick a range that centers the
  target: `5802` (0.35–3.5 pF), `5761` (0.6–6 pF), `5201` (0.8–10 pF), `5502` (1–20 pF), `5602`
  (1–30 pF). Above ~30 pF (only 2 m C4 ≈ 40 pF) use a fixed 33 pF C0G (Murata `GRM18`) in parallel
  with a `5201` trimmer.
- **Higher-Q option → air-wound inductors**, silver-plated Cu (~20 AWG; DigiKey stocks the wire),
  tuned by turn spacing. Chip Q (dozens) vs air-wound (150+) costs a little insertion loss and
  ultimate stopband at the top bands; negligible at 2 m/222. **902 is where air-wound earns its keep**
  — wind those coils if you want the last bit of loss/rejection; fixed chips are fine on 2 m/222/70 cm.
  (Air-wound is also the only good *tunable* inductor at UHF — ferrite slug-tuned cores are unusable
  above VHF from eddy-current loss — but with cap tuning you don't need a tunable inductor.)
- **Connectors:** Amphenol RF `901-10513-1` edge-launch SMA jack (0.062″ PCB), ×2 per board.
- **Board:** custom FR-4 (JLCPCB / PCBWay) or copper-clad.

Per board (inductor = fixed Coilcraft nearest E24; cap = trimmer range chosen to center the target):

| Board | Inductors (Coilcraft, fixed) | Trimmer caps (Johanson PN) |
|-------|------------------------------|----------------------------|
| **2 m** | L1,L3 `0603DC-68N`; L2 `0603DC-39N`; L4,L5 `0603DC-68N` | C1,C2,C3,C5 ~22 pF → `5602`; C4 ~40 pF → 33 pF `GRM18` + `5201` |
| **222** | L1,L3 `0603DC-43N`; L2 `0603DC-27N`; L4,L5 `0603DC-47N` | C1,C2,C3,C5 15 pF → `5602`; C4 27 pF → `5602` |
| **70 cm** | L1,L3,L4,L5 `0603DC-20N`; L2 `0603DC-11N` | C1,C2,C3,C5 ~6.6 pF → `5201`; C4 12 pF → `5502` |
| **902** | L1,L3 `0402DC` 9.1 nH; L2 `0402DC` 5.1 nH; L4,L5 `0402DC` 9.1 nH *(or air-wound)* | C1,C2,C3,C5 ~3.0 pF → `5761`; C4 ~5.2 pF → `5201` |

Five chip inductors + five trimmers per board. Tune the caps for best in-band return loss and the
harmonic rejection you need; the inductors don't move. (Computed pre-round L/C values: filters.md.)
