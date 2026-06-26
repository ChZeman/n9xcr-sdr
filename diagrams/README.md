# Diagrams

Block and reference diagrams for the station design. Each SVG is self-contained (inline styles) and
renders on its own in a browser; GitHub shows them in the file view.

> Bands shown in these diagrams are the US / IARU R2 example (2 m / 222 / 70 cm / 902); see [`../docs/regions.md`](../docs/regions.md) to adapt.

| File | What it shows |
|------|---------------|
| `station-overview.svg` | Whole-station architecture: a node site (antennas → RF front end → SDRs → node host) dialing home to a shared head-end and web UI, with raw IQ staying on site and other nodes federating. |
| `tx-chain.svg` | Transmit signal chain with part numbers: 7020-SDR (onboard PA) -> PE42512A SP12T band-select (low level) -> four per-band chains (bandpass -> CGH40010 driver -> MRF101AN / CGH40120F final, in the control wrapper) -> per-band LPFs. The chain ends at four per-band 50 W RF outputs; routing (antenna, triplexer, switch, or dummy load + RF-exposure lockout) is station-defined, not part of the node design. |
| `rx-chain.svg` | Receive chain with part numbers: antennas (Tram tri-band, Comet 222 whip, wideband discone) → triplexer / FM-trap → band-select → preselectors (DCI-146-4H, Temwell helical, FreeWave EBF901) → bypassable LNA → 7020-SDR. |
| `rx-frontend.svg` | Receive front-end packaging: a shielded carrier board (two ADRF5040 SP4Ts + bypassable LNA) hosting the 222 / 70 cm / 902 filter modules, with the 2 m cavity and the wideband path external. |
| `gain-budget.svg` | Level diagram of the transmit gain budget (dBm at each stage) for the worst-case 902 MHz / 80 W path. |
| `control-wrapper.svg` | The PA control / protection subsystem: T/R sequencing, gate-bias keying, SWR and thermal foldback, overcurrent trip, the sensors and fan, and the software-ALC tie-back to the node host. |
| `telemetry.svg` | The telemetry path from PA sensing through the local IO bridge and LAN to the web-UI tiles — with the reminder that hard-real-time protection is autonomous and not on this path. |
| `band-coverage.svg` | Conceptual coverage map: the four TX bands, receive to 6 GHz, the HF exciter, and the wideband RX antenna. |
| `band-select-schematic.svg` | Functional schematic of the TX band selector: PE42512A SP12T on a single 3.3 V supply (VSS_EXT->GND, internal -V gen), no DC-blocks, V1-V4 + LS control, the loss-tiered 12-port band map, and the all-isolated park (TX-inhibit) state. |

**Color conventions:** blue = SDR / digital, teal = LDMOS, purple = GaN (and head-end / UI), gray =
passive / structural, amber dashed = control-wrapper boundary and sense/control lines.
