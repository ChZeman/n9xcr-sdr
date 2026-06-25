# Bill of materials

Consolidated part list for a VHF/UHF node. "Firm" = decided; "Open" = still needs a part picked.

## Firm

| Item | Part | Notes |
|------|------|-------|
| VHF/UHF SDR | OpenSourceSDRLab 7020-SDR (PlutoSky R2), AD9361 + onboard PA | Buy direct from opensourcesdrlab.com (PlutoSky R2 7020) and select the **"9361 with PA"** variant (~$220) — not the "9363" variants, and not the $120 AD9363 "Professional Edition". Aluminum case is a separate ~$18 item (match the shell to a 9361-with-PA board). AliExpress "7020-SDR AD9361 with case" (~$200.71) is the cheaper equivalent — confirm the listing says *with PA*. AntSDR E200 (Mouser `ANTSDR-AD9361-With-CASE-01`) is the turnkey alternative. AD9361 not AD9363; onboard PGA-102+ PA (~+15–19 dBm) covers the pre-driver; runs plutosdr-fw / libiio. |
| HF SDR | Hermes-Lite 2 | example; any HPSDR-class HF SDR |
| Digital-voice vocoder | DVMEGA DVstick 30 | AMBE-3000; ham dealer (e.g. GigaParts), not a component distributor |
| Final, 2 m + 222 | NXP MRF101AN | 100 W LDMOS, 1.8–250 MHz; run at 50 W |
| Final, 70 cm + 902 | Wolfspeed/MACOM CGH40120F | 120 W GaN, 28 V; one part both bands; run at 50 W |
| Pre-driver | onboard PGA-102+ on the SDR | ~+15–19 dBm out; no separate board (discrete PGA-103+ optional if the onboard PA is bypassed) |
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

## Band-select build (ADRF5040)

Parts for one band-select SP4T. The ADRF5040 needs **+3.3 V and −3.3 V** — the ADM8829 makes
the −3.3 V rail from +3.3 V. Schematic: `diagrams/band-select-schematic.svg`.

| Item | Part | Source (approx) | Notes |
|------|------|-----------------|-------|
| SP4T switch | **ADRF5040BCPZ** | Mouser / DigiKey ~$32 | order the BCPZ strip (MOQ 1); not the `-R7` reel, not the `-EVALZ` board |
| −3.3 V generator | **ADM8829ARTZ-REEL7** | DigiKey ~$3 (cut-tape qty 1) | SOT-23-6 charge-pump inverter; +3.3 V in → −3.3 V out to VSS |
| Charge-pump caps | 2 × 1 µF | with order | C1 flying (CAP+/CAP−), C2 reservoir (OUT→GND) |
| Supply decoupling | 100 nF + bulk (1–10 µF) | with order | on VDD and VSS to GND |
| RF DC-blocks | ~1 nF C0G × 5 | with order | RFC + RF1–RF4 |
| PCB | **ADI EVAL-ADRF5040 official gerbers** (4-layer RO4350) | PCBWay (Rogers-capable); turnkey PCBA to place the LFCSP | proven ADI layout — see "Board: fab ADI gerbers" below; own 2-layer FR-4 is the cheaper alternative |
| Connectors (standalone only) | SMA edge-launch × 5 | with order | omit if integrated onto the TX board |
| Control | 2 × GPIO (V1/V2) | from control MCU | 3.3 V CMOS; nothing to buy |

- The TX node uses **one** band-select; the RX front-end uses **two** more of the same ADRF5040
  (switch–filter–switch), so a full build is **3** switches — the two co-located RX switches can
  share a single ADM8829 −3.3 V rail.
- No-build alternative: the `ADRF5040-EVALZ` populated eval board (~$282) — turnkey, but still needs
  external +3.3 V and −3.3 V applied.


### Board: fab ADI's official gerbers (chosen)

Rather than design a board from scratch or order the PCBWay community clone, fab ADI's **own**
eval-board gerbers – the authoritative ADRF5040-EVALZ design.

- **Gerbers:** <https://www.analog.com/media/en/technical-documentation/evaluation-documentation/gerber-files/ADRF5040_Gerbers.zip> (linked from the EVAL-ADRF5040 page). Verified contents: a 4-layer package (METAL-01–04) with soldermask / silkscreen / paste, NC drill, a fabrication-drawing PDF (material + stackup + dimensions), an IPC-2581 file, and an X-Y pick-and-place file.
- **Fab:** PCBWay (handles Rogers). Upload the zip; it is **4-layer Rogers RO4350**, ENIG finish. The fab-drawing PDF in the package carries the stackup/material spec if they ask.
- **Assembly:** use the included X-Y pick-and-place file for turnkey PCBA (PCBWay places the LFCSP – this removes the hand-reflow problem), or hand-populate. Assembly BOM = ADRF5040BCPZ + RF DC-block caps + 5 × SMA + test points.
- **Tweak for our bands:** the eval's RF DC-blocks are 100 pF (fine up high, ~11 Ω at 144 MHz); populate ~1 nF C0G in those positions for clean low-band response.
- **Still needs ±3.3 V:** the eval expects external +3.3 V *and* –3.3 V at its test points – it has **no** onboard negative generator, so pair it with the ADM8829 rail above (a small adjacent board, or a bench –3.3 V supply for bring-up).
- Do **not** fab the PCBWay community clone of this board – its author flagged unresolved design issues and disabled ordering.
