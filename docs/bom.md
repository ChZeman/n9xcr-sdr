# Bill of materials

Consolidated part list for a VHF/UHF node. "Firm" = decided; "Open" = still needs a part picked.

## Firm

| Item | Part | Notes |
|------|------|-------|
| VHF/UHF SDR | AntSDR E200, AD9361 | Mouser `ANTSDR-AD9361-With-CASE-01` (board + AD9361 + 32 GB SD + case). Not the AD9363 SKU. |
| HF SDR | Hermes-Lite 2 | example; any HPSDR-class HF SDR |
| Digital-voice vocoder | DVMEGA DVstick 30 | AMBE-3000; ham dealer (e.g. GigaParts), not a component distributor |
| Final, 2 m + 222 | NXP MRF101AN | 100 W LDMOS, 1.8–250 MHz; run at 50 W |
| Final, 70 cm + 902 | Wolfspeed/MACOM CGH40120F | 120 W GaN, 28 V; one part both bands; run at 50 W |
| Pre-driver | broadband MMIC, PGA-103+ class | ~22 dB, one part all bands |
| Driver | Wolfspeed/MACOM CGH40010 (F flange / P pill) | 10 W GaN (13 W PSAT), DC–6 GHz, 28 V; one part type, 4 builds (per band); runs ~30 % for linear |
| TX band-select | Analog Devices ADRF5040 | silicon SP4T, nonreflective, 9 kHz–12 GHz, 3.3 V control; low-level switch ahead of the per-band chains |
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
