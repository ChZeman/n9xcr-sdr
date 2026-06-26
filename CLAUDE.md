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
- **BOM + PCB:** finish the consolidated Mouser / DigiKey BOM; order the custom PE42512A band-select
  board (DigiKey `1046-PE42512A-XCT-ND`, Cut Tape).
- **Propagate the per-slice / software-limit decision into the deep docs.** `filters.md` LPF table +
  usable-window table re-derived for slice-top fc (4 m/70 cm/33 cm/23 cm/13 cm/9 cm cutoffs rise);
  `tx-chain.svg` BPF boxes relabelled band->slice and the "band-tuned BPF restricts each path" caption
  corrected (band-limit now lives in software); `regions.md` BPF/LPF guidance reworded. Currently only
  the decision log (this file) reflects it.
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
