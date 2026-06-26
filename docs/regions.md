# Adapting this design to your region

This is a **region-agnostic** node design. The architecture, the transmit chain
(band-select -> per-band BPF -> driver -> final -> LPF), the receive front end, and the
federation model do not depend on any country's band plan. What changes between countries is
**which bands you build** and the **exact passbands, finals, and antennas** for them.

Everything else in this repo uses the **US / IARU Region 2** bands (2 m, 222, 70 cm, 902) as a
complete worked example. Treat it as a template and substitute your own bands.

## IARU regions

Amateur allocations are coordinated across three IARU regions, and national regulators differ
within them - always confirm against **your own national band plan**, which is the legal authority.

- **Region 1** - Europe, Africa, the Middle East, northern Asia.
- **Region 2** - the Americas (this repo's worked example is US / R2).
- **Region 3** - most of Asia and the Pacific.

## Bands in range of this node

The VHF/UHF SDR + PE42512A band-select reach **4 m (70 MHz) through 5 cm (5.8 GHz)** - the AD9361
transmit range. Below is where each band generally exists; build only the ones your country
allocates. (6 m and below ride the HF SDR, not this node.)

| Band | Typical range | Where it generally exists |
|------|---------------|---------------------------|
| 4 m | 70.0-70.5 MHz | R1 only (UK and parts of Europe); not R2/R3 |
| 2 m | 144-146 (R1) / 144-148 (R2, R3) | all regions |
| 1.25 m (222) | 222-225 MHz | R2 only (the Americas) |
| 70 cm | 430-440 (R1) / 420-450 (R2) | all regions; edges vary |
| 33 cm (902) | 902-928 MHz | R2 only, and not all of it |
| 23 cm | 1240-1300 MHz | all regions; edges vary |
| 13 cm | 2300-2450 MHz | varies widely by country |
| 9 cm | 3300-3500 MHz | varies; shrinking in places |
| 5 cm | 5650-5925 MHz | most regions; edges vary |

General references only, not legal allocations - bandplans change and differ nationally.

## How to adapt each stage

- **Band-select (PE42512A SP12T).** No change. Its 12 ports are fixed 1.5:1 sub-octave slices
  spanning 4 m-5.8 GHz; whatever band you build lands in the slice that contains it (the port map
  in [`bom.md`](bom.md) shows which). Populate only the ports you build.
- **BPF / LPF.** Region-specific. [`filters.md`](filters.md) gives the synthesis method plus a US/R2
  worked set; recompute fc and the L/C (or resonator) values for your passbands.
- **Driver.** No change - the CGH40010 is flat DC-6 GHz.
- **Finals.** Chosen by **frequency range, not country**:
  - MRF101AN (LDMOS, ~1.8-250 MHz) covers 4 m / 2 m / 222 and the R1 equivalents.
  - CGH40120F (GaN, UHF to ~1.3 GHz) covers 70 cm / 902 / 23 cm.
  - 13 cm / 9 cm / 5 cm (2.3-5.8 GHz) need different finals - not yet specified here.
- **Antennas + output routing.** Entirely site/region-specific. The published chain ends at the
  LPF (see [`tx-chain.svg`](../diagrams/tx-chain.svg)); how you route to antennas, a combiner, a
  switch, or a dummy load is your call.

## Power limits and RF exposure

The 50 W example level and the RF-exposure maintenance lockout are **engineering guidance, not legal
limits**. Comply with your national regulator's power ceiling and RF-exposure rules (e.g. FCC OET-65
in the US; ICNIRP-based national rules elsewhere). The exposure-lockout hook in the output stage is
there so you can meet whatever your rules require.
