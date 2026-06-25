# n9xcr-sdr

A reference design for a distributed, internet-operated, all-mode SDR transceiver station —
maintained by N9XCR and written to be general enough for others to follow and adapt. It holds
**only** the radio / RF / signal-path design — no infrastructure, server, or
network-operations material.

## Concept

A federation of independent, self-contained SDR nodes that each dial home over the
internet to a shared head-end, presented through a single web control surface with a
waterfall and full-duplex audio. Each node is a standalone radio site; nodes can be
replicated and placed at different locations.

- **Transmit span:** 550 kHz – 928 MHz (HF through 33 cm).
- **Receive span:** to 6 GHz (the AntSDR ceiling) — a wide monitoring dividend on top of the ham bands.
- **Modes:** SSB / CW / AM / FM, the FT8/JS8/fldigi digital suite (run as the actual
  applications at the node), and digital voice (DMR / D-STAR / Fusion via an AMBE
  vocoder; M17 / FreeDV in software).

## Coverage

| Span | Radio | Notes |
|------|-------|-------|
| 0.5 – 30 MHz | Hermes-Lite 2 (example HF radio) | HF, 5 W, PureSignal-capable |
| 70 – 928 MHz (TX) / → 6 GHz (RX) | AntSDR E200 (AD9361) | 2 m, 222, 70 cm, 902; full-duplex |
| 6 m (50–54) | future SDRlab 122-16 swap | deferred; direct-sampling, real displayed freq |

The only amateur band in the 30–70 MHz gap is 6 m, which the future HF-board swap
restores. The TX build is capped at **902 / 33 cm**; receive keeps reaching to 6 GHz.

## Hardware at a glance

- **HF exciter:** Hermes-Lite 2 (example; any HPSDR-class HF SDR works).
- **VHF/UHF radio:** AntSDR E200, AD9361 version.
- **Digital voice:** DVMEGA DVstick 30 (AMBE-3000).
- **Reference:** single 10 MHz GPSDO per site (GPS-disciplined OCXO).
- **TX finals:** NXP MRF101AN (2 m, 222) and Wolfspeed/MACOM CGH40120F GaN (70 cm, 902), 50 W per band.

See [`docs/bom.md`](docs/bom.md) for the consolidated part list.

## Status

RF design complete on paper end-to-end: radios, gain budget, all four finals, both
filter banks, antennas, and the PA control/protection wrapper are specified. Remaining
work is non-RF: node-host sizing and the head-end / web-UI software stack.

## Documents

- [`docs/architecture.md`](docs/architecture.md) — node and federation concept, full-duplex, crossband, satellite.
- [`docs/radios.md`](docs/radios.md) — the radios, clocking, and the 6 m path.
- [`docs/tx-chain.md`](docs/tx-chain.md) — transmit gain budget, stages, and control wrapper.
- [`docs/filters.md`](docs/filters.md) — TX low-pass bank and RX preselectors.
- [`docs/antennas.md`](docs/antennas.md) — antenna plan and trade-offs.
- [`docs/bom.md`](docs/bom.md) — consolidated part numbers.
- [`docs/site-install.md`](docs/site-install.md) — outdoor deployment considerations.
- [`docs/telemetry.md`](docs/telemetry.md) — SWR protection, telemetry, and the path to the UI.
- [`docs/power-thermal.md`](docs/power-thermal.md) — power headroom, thermal limits, and maximum safe output.
- [`docs/boards.md`](docs/boards.md) — board source for every stage (fab vs buy).
- [`diagrams/`](diagrams/) — block diagrams, architecture, gain budget, control & telemetry, coverage ([`diagrams/README.md`](diagrams/README.md)).
