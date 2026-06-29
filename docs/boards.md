# Boards: fab vs buy

A per-stage map of where each board comes from. The pattern across the transmit chain: the active
gain stages (finals, driver) are vendor **reference / evaluation** boards — buy the eval
board or fab the published reference and retune — while the protection and filtering stages are
off-the-shelf ham-catalog products.

"Reference / demo board" means the device maker publishes a known-good schematic, BOM, and layout.
Fabbing it removes the guesswork in matching and stability; it is not a from-scratch design.

## Transmit chain

| Stage | Device | Board / source | Buy or fab | Count |
|-------|--------|----------------|-----------|------:|
| Pre-driver | onboard (PGA-102+) | on the 7020-SDR — no board to fab or buy | — | 0 |
| TX band-select | PE42512A | **custom board to be laid out** from the SnapEDA/SnapMagic `PE42512A-X` footprint — no usable eval/breakout exists (the pSemi eval is a 12-SMA characterization fixture). See `bom.md` "Band-select build". | fab (custom, **not started**) | 1 (shared) |
| Driver | CGH40010 | MACOM **CGH40010-AMP** reference amp (schematic + BOM, Rogers RO4350B 20 mil); the **CGH40010F-AMP** test board ships with the device installed. Schematic + BOM + bias/keying: [`driver.md`](driver.md), [`../diagrams/driver-schematic.svg`](../diagrams/driver-schematic.svg). | buy test board, or fab + retune | 4 (per band) |
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
- **Substrate:** the GaN reference boards specify Rogers RO4350B (low-loss). Small-signal sections are
  fine on good FR-4 at these frequencies; the finals and driver benefit from RO4350B.
- The split happens at the band-select: the SDR (with its onboard PA) and the band-select are shared and sit ahead of it, while the bandpass, driver, final and low-pass are per-band (built four times). Switching at low level keeps the band-select a small signal-level part and lets each band's bandpass clean the signal before it is amplified.
- Verify harmonic suppression on any bundled or fabbed filter with an analyzer (the design target is
  −43 dBc or better); do not assume a board meets it.

## Connectors

Split the connector families by role, not by convenience:

- **Internal, low-level interconnects (switch → BPF, and the inter-stage hops):** **SMA**. The
  band-select runs at the ~+19 dBm / 80 mW exciter level, so power handling is irrelevant here — what
  matters is a true 50 Ω match, shielding (spurs are cleaned *before* the gain, so inter-path
  isolation counts), and frequency headroom past the highest band. SMA gives all three, is threaded/
  secure for a permanent node, and is compact enough to fan out 12 ports. Use **edge-launch SMA jacks**
  with short **RG402 / .141 semi-rigid** jumpers (lower leakage and repeatable geometry vs flexible
  RG316; loss is a non-issue at these lengths/levels, so optimize for shielding).
- **50 W output / antenna feed:** **N** — robust, weatherproof, good past 11 GHz. Keeping it a
  different family from the internal SMA also physically prevents cross-patching a low-level port into
  the antenna.
- **Avoid:** UHF / PL-259 (not constant-impedance; useless above ~300 MHz, so wrong for 2 m and up)
  and BNC (marginal past ~1 GHz). TNC is acceptable where threaded robustness is wanted, but bulkier
  than SMA for a dense switch board.
- **Integration endgame:** if the per-band modules later move onto a common motherboard instead of
  separate boxes with jumpers, switch the internal hops to **SMP / SMPM blind-mate push-on** — they
  mate without torque (a real win across 12 ports), stay compact, and run well past the bands. SMA
  jumpers are the right starting point; SMP/SMPM is the upgrade path.
