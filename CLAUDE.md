# n9xcr-sdr - project notes

Working notes and design rationale for this repo. The formal design lives in `README.md`,
`docs/`, and `diagrams/`; this file is the orientation + decision log. Deployment / site
specifics and all credentials live outside this repo by design - keep them out.

## What this is

A distributed, internet-operated, all-mode SDR transceiver station: a federation of independent
SDR nodes that each dial home to a shared head-end presented through one web UI, with raw IQ kept
local and SDRangel (headless) doing DSP per node. One 10 MHz GPSDO per site disciplines everything.
The first node is a VHF/UHF transmit node; the design is written to be reproduced and adapted by
others (see `docs/regions.md`).

## Transmit signal chain

7020-SDR exciter (onboard PA, ~+19 dBm) -> PE42512A SP12T band-select, switched at low level ->
per-band module [BPF -> CGH40010 driver -> final -> LPF] -> 50 W out. The published chain ends at
the LPF (the node boundary); antenna routing is defined per station, not by the node design.
See `diagrams/tx-chain.svg`.

## Key decisions (and why)

- **Federated nodes, not one big radio.** Each node is standalone and dials home; raw IQ stays
  local; the head-end only presents. Keeps backhaul sane and nodes replaceable.
- **VHF/UHF SDR = OpenSourceSDRLab 7020-SDR (PlutoSky R2), AD9361 + onboard PA.** AD9361 (not
  AD9363) is required - the network-native libiio/PlutoSDR model over Gigabit Ethernet is the
  point; USB-3 / UHD devices (e.g. B210) and AD9363 "Pro" boards were rejected. The onboard PA
  absorbs the pre-driver stage.
- **HF = Hermes-Lite 2 (example).** 6 m and below ride the HF SDR; the 6 m gap is closed by a
  direct-sampling Red Pitaya-class board (SDRlab 122-16 / TRX-duo) as a future swap - a transverter
  was rejected.
- **Band-select = pSemi PE42512A SP12T** (replaced an earlier ADRF5040 SP4T on the TX path).
  Single 3.3 V supply (VSS_EXT->GND, internal -V generator) so no charge pump and no DC-blocks;
  9 kHz-8 GHz so it never limits the band; 12 ports; LS tied low (logic-0 decode map). Custom board
  from the SnapEDA `PE42512A-X` footprint. The 12 ports are fixed 1.5:1 sub-octave slices spanning
  4 m-5.8 GHz; populate the slices your region allocates. (The RX path keeps 2x ADRF5040 SP4T +
  shared ADM8829 -3.3 V.)
- **Filters spec'd per slice (full port range), band-limiting moved to software.** The hardware is
  built to transmit the FULL 1.5:1 slice of each populated port; the legal band-limit is enforced only
  in the node's allowed-TX frequency table. So the per-port BPF is **slice-wide** (cleans the AD9361's
  out-of-slice images / LO leakage / harmonics, no longer pins the band) and the per-port LPF cutoff
  sits just above the **slice top**, not ~1.1x the ham-band top. Motivation: a licensed op with
  MARS/CAP authority can work the assigned segments adjacent to each ham band (e.g. CAP VHF ~148-150
  just above 2 m) without the hardware fencing them out. One sub-octave LPF holds >=43 dBc 2nd-harmonic
  over the top ~97% of a 1.5:1 slice; the bottom-few-MHz under-margin sliver sits below the lowest
  allocation in every slice (and the slice-wide BPF's upper skirt covers it), so it is clean everywhere
  an op is actually authorized to key. Raising LPF fc to the slice top trades a little ham-band harmonic
  margin (70 cm ~64->~58 dB at 432) but stays well past -43 dBc. NOTE: with hardware no longer fencing
  bands, the software allowed-TX table is now SAFETY-CRITICAL -> must be FAIL-CLOSED (deny by default,
  permit only enumerated authorized segments; RF7 cold-switch park = safe state on fault/out-of-table).
  RX preselectors stay narrow (separate job: front-end protection + crossband isolation).
- **BPF cleanup filters: two-trimmer tune, top-adjust SMD Giga-Trims.** Each slice-wide BPF trims only
  C2 (low corner) + C4 (high corner); C1/C3/C5 fixed C0G. All trimmers standardized on **top-adjust
  surface-mount** sapphire Giga-Trims (lowest parasitics, tune the board flat): `5602` 1–30 pF,
  `5202` 0.8–10 pF, `27273` 0.6–4.5 pF (902 C2), `5502` 1–20 pF — DigiKey 1956-1000/1001/1032/1008-ND.
  DigiKey's "Top Panel Mount" tag on some is a mislabel (Giga-Trims have no bushing). See `docs/parts-list.md`.
- **Driver stage documented.** The CGH40010 GaN driver now has its own schematic + per-board BOM
  (`diagrams/driver-schematic.svg`) and a consolidated doc (`docs/driver.md`): reference-amp
  topology, ~100–150 mA IDQ, GaN bias-sequencing rule, per-band match/retune, a **4-board build
  BOM** (2 m–902, split buy-now / buy-after-bench / shared tuning kit), and a **bench bring-up**
  procedure (Phases 0–4, tune-then-fix). One schematic (not 4) since the device + bias network are
  common and only the match retunes per band. Match L/C are bench-measured, not pre-orderable;
  trimmers + air coils are bench instruments, the shipped board is all-fixed.
- **Driver device confirmed = CGH40010F (stay).** The part is **NRND**; a full alternatives survey
  (CG2H40010F = MOQ 120; QPA2237 = $134/10 W/2.5 GHz; CMPA0060002F1 = >$300; QPD1010 = QFN relayout;
  TGF2936 = discontinued; band-optimized LDMOS = multiple device types) concluded the CGH40010F is
  still best: one part all bands, design complete, ~$90 in stock (cheaper than QPA2237). Buy from
  remaining stock as a final buy; per-band match tuned on the bench. Rationale recorded in
  `docs/driver.md` (Device selection). Same NRND applies to the CGH40120F final → CG2H40120F is its
  Gen-2; finals lifecycle is a separate open decision.
- **Verified DigiKey order list committed to `docs/driver.md` §G.** Every driver line decoded to a
  confirmed PN (CGH40010F 1465-CGH40010F-ND; DC blocks 490-1451/1447/1427/1419-1-ND = 1n/680p/100p/47p;
  Rg RC0603FR-0722RL = 311-22.0HRCT-ND; 0.1u 311-1344-1-ND; 0.01u 311-1085-1-ND; Cd1 490-1427-1-ND;
  10u/100u elec ECA-1HM100/101 = P5178/P5182-ND; SMA 901-10513-1 = ARF2504-ND). PR audit (PR1102611408):
  100 pF (490-1427-1-ND) split across two lines — condense to qty 6; Lg/Ld chokes not on that PR.
  Rule reinforced: ignore Six Flags item descriptions, trust the DigiKey supplier #.
- **Driver bias choke settled = Coilcraft 4310LC-132KEC** (1.3 µH wideband bias choke, one part all
  bands; Coilcraft-direct, not DigiKey — small separate order, qty 8 +spares). Driver scope now fully
  specified up to the 10 W stage. PR1102611408 100 pF consolidation (lines 25+29 → qty 6) applied.
- **Driver PCB/mechanical captured in `docs/driver.md`.** One common layout all bands (per-band diff =
  bench-tuned match only; 2 m/222 input may go air-wound). Working board size ≈ 50×75 mm on RO4350B.
  Mounting: flange through a board cutout to a backplane-mounted spreader plate (aluminium for driver,
  copper reserved for final); two thermal interfaces; sealed-enclosure heat-escape (vents/fan/wall) since
  driver+final share the box. Layout is the remaining work; electrical design complete.
- **Finals by frequency, not by band:** MRF101AN LDMOS (1.8-250 MHz -> 4 m / 2 m / 1.25 m);
  CGH40120F GaN (DC-1.5 GHz rated, ~1.3 GHz practical -> 70 cm / 33 cm / 23 cm). Driver is one
  CGH40010 (GaN, DC-6 GHz) per path. One final serves several bands because the amps are broadband;
  the band-tuned BPF is what pins a path to its band, not the wider band-select slice.
- **Filters:** per-band 9-element 0.1 dB Chebyshev LPF, sub-octave so the 2nd harmonic lands above
  cutoff. RX preselectors are a separate high-Q (helical / cavity) job, kept off the TX path. See
  `docs/filters.md`.
- **50 W baseline** (100 W was explored and rejected on thermal / headroom grounds).
- **Region-agnostic.** Anchored on IARU R1/R2/R3; the US/R2 bands (2 m / 222 / 70 cm / 902) are the
  worked example; the band-select 4 m-5.8 GHz plan is the shared backbone. See `docs/regions.md`.
- **PureSignal N/A** - SDRangel + device back-off linearity, no DPD.
- **Digital voice = DVMEGA DVstick 30** (AMBE-3000).

## Open items

- **Microwave finals (13 / 9 / 5 cm, 2.3-5.8 GHz) - TBD.** Those paths are drawn but no power device
  is chosen; the current finals stop ~1.3-1.5 GHz.
- **CGH40120F wording:** the datasheet rates DC-1.5 GHz; practical ceiling ~1.3 GHz (23 cm / 1296 sits
  near the top). Reconcile `docs/regions.md` vs `diagrams/tx-chain.svg` phrasing.
- **Per-doc region tags** still pending on the deep docs (`tx-chain.md`, `power-thermal.md`,
  `boards.md`, `antennas.md`) and the RX diagrams (`rx-chain.svg`, `rx-frontend.svg`), which remain
  the US/R2 worked example.
- **Negative keyed gate bias (control board).** The CGH40010 driver is GaN on *every* band and the
  70 cm / 902 finals are GaN too → all need a **negative**, keyed gate rail; the LDMOS finals
  (MRF101AN) need a **positive** one. Confirm the chosen amp-control board does negative bias, or add
  a small negative-bias/keying board. Gating open item for the driver stage (see `docs/driver.md`).
- **BOM + PCB:** finish the consolidated Mouser / DigiKey BOM; order the custom PE42512A band-select
  board (DigiKey `1046-PE42512A-XCT-ND`, Cut Tape).
- **Deferred (non-RF):** node-host sizing; head-end / web-UI software stack.

## Repo layout

- `README.md` - overview + coverage.
- `docs/` - architecture, radios, tx-chain, filters, antennas, bom, regions, site-install, telemetry,
  power-thermal, boards, rx-frontend, tx-packaging.
- `diagrams/` - block / reference SVGs (see `diagrams/README.md`).

## Diagram conventions

Self-contained SVGs, inline styles. Palette: blue = SDR / digital, teal = LDMOS, purple = GaN /
head-end, gray = passive / structural, amber-dashed = control wrapper. On the all-region diagrams
(`band-coverage.svg`, `tx-chain.svg`): band-box colour = IARU region availability (R1 blue /
all-region teal / R2 purple / varies grey); final colour = device type (LDMOS teal / GaN purple /
TBD amber).
