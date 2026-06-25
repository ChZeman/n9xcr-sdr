# Boards: fab vs buy

A per-stage map of where each board comes from. The pattern across the transmit chain: the active
gain stages (finals, driver, pre-driver) are vendor **reference / evaluation** boards — buy the eval
board or fab the published reference and retune — while the protection and filtering stages are
off-the-shelf ham-catalog products.

"Reference / demo board" means the device maker publishes a known-good schematic, BOM, and layout.
Fabbing it removes the guesswork in matching and stability; it is not a from-scratch design.

## Transmit chain

| Stage | Device | Board / source | Buy or fab | Count |
|-------|--------|----------------|-----------|------:|
| Pre-driver | PGA-103+ | Mini-Circuits **TB-678-103+** eval board, or the datasheet "Recommended Application Circuit / Suggested PCB Layout" | buy eval, or fab | 1 (shared) |
| Driver | CGH40010 | MACOM **CGH40010-AMP** reference amp (schematic + BOM, Rogers RO4350B 20 mil); the **CGH40010F-AMP** test board ships with the device installed | buy test board, or fab + retune | 1 shared *or* 4 per-band |
| Cleanup bandpass | — | small low-level bandpass ahead of each driver | fab / buy | per band |
| Final, 2 m / 222 | MRF101AN | NXP **136–174 MHz reference circuit** (datasheet), retuned to band | fab + retune | 2 |
| Final, 70 cm / 902 | CGH40120F | MACOM **0.8–1.3 GHz reference (03-000255)** for 902; broadband reference for 70 cm | fab + retune | 2 |
| TX low-pass | — | 9-element Chebyshev (see `filters.md`) — fab, or adapt a published LPF board | fab / buy | 4 |

## Control / protection wrapper

The only stages bought as finished ham-catalog boards.

| Function | Board |
|----------|-------|
| Bias keying + sequencing | W6PQL Amplifier Control Board (or Upgraded Controller) |
| SWR / forward-reflected sampling | W6PQL Dual Directional Detector |
| T-R sequencing | W6PQL Relay Sequencer |
| Telemetry bridge | small ESP32 / Pico-class IO board (local, at the enclosure) |

## Receive chain

| Stage | Board / source | Buy or fab |
|-------|----------------|-----------|
| Preselectors | off-the-shelf cavity / helical (DCI-146-4H; Temwell helical for 222 and 70 cm; FreeWave EBF901 for 902), or the Nooelec Ham Filter Bank to start | buy |
| LNA | commodity bypassable LNA board | buy |

## Notes on the reference-board approach

- The active boards are characterized at the vendor's test frequency (often a GHz or more). For the
  amateur bands you retune the matching network — straightforward at VHF/UHF on these broadband
  parts, and increasingly fiddly with frequency (902 is the touchiest, the edge of lumped-element
  matching).
- **Substrate:** the GaN reference boards specify Rogers RO4350B (low-loss). The MMIC pre-driver is
  fine on good FR-4 at these frequencies; the finals and driver benefit from RO4350B.
- Where a board is "1 shared or 4 per-band" (the driver), the trade-off is in `tx-chain.md`: a
  per-band bandpass really wants to sit ahead of the driver, which nudges toward per-band drivers,
  while one shared broadband driver before the band-select saves three boards.
- Verify harmonic suppression on any bundled or fabbed filter with an analyzer (the design target is
  −43 dBc or better); do not assume a board meets it.
