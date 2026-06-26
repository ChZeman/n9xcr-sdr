# Diagrams

Block and reference diagrams for the station design. Each SVG is self-contained (inline styles) and
renders on its own in a browser; GitHub shows them in the file view.

> Most diagrams below use the US / IARU R2 example bands (2 m / 222 / 70 cm / 902). **`tx-chain.svg`** and **`band-coverage.svg`** instead show every band / path across all IARU regions. See [`../docs/regions.md`](../docs/regions.md) to adapt.

| File | What it shows |
|------|---------------|
| `station-overview.svg` | Whole-station architecture: a node site (antennas → RF front end → SDRs → node host) dialing home to a shared head-end and web UI, with raw IQ staying on site and other nodes federating. |
| `tx-chain.svg` | The full transmit signal chain, **every band path**: 7020-SDR exciter → PE42512A SP12T band-select (low level) → all 12 ports, each populated port a module (BPF → CGH40010 driver → final by frequency: MRF101AN / CGH40120F / µwave TBD → LPF) → 50 W RF out. Spare slices and the dummy/park port are shown; the chain ends at the LPF (node boundary), routing station-defined. All IARU regions; each device box also shows its technical working frequency range. The blue-outlined BPF boxes (built bands 2 m / 222 / 70 cm / 902) hyperlink to their per-band BPF schematics below (clickable when the SVG is opened raw or in a viewer; use the links below on GitHub). |
| `rx-chain.svg` | Receive chain with part numbers: antennas (Tram tri-band, Comet 222 whip, wideband discone) → triplexer / FM-trap → band-select → preselectors (DCI-146-4H, Temwell helical, FreeWave EBF901) → bypassable LNA → 7020-SDR. |
| `rx-frontend.svg` | Receive front-end packaging: a shielded carrier board (two ADRF5040 SP4Ts + bypassable LNA) hosting the 222 / 70 cm / 902 filter modules, with the 2 m cavity and the wideband path external. |
| `gain-budget.svg` | Level diagram of the transmit gain budget (dBm at each stage) for the worst-case 902 MHz / 80 W path. |
| `control-wrapper.svg` | The PA control / protection subsystem: T/R sequencing, gate-bias keying, SWR and thermal foldback, overcurrent trip, the sensors and fan, and the software-ALC tie-back to the node host. |
| `telemetry.svg` | The telemetry path from PA sensing through the local IO bridge and LAN to the web-UI tiles — with the reminder that hard-real-time protection is autonomous and not on this path. |
| `band-coverage.svg` | Every amateur band the node can transmit on (4 m–5.8 GHz) on a log-frequency axis, colour-tagged by IARU region availability (all regions / R2-only / R1-only / varies); plus receive to 6 GHz. |
| `band-select-schematic.svg` | Functional schematic of the TX band selector: PE42512A SP12T on a single 3.3 V supply (VSS_EXT->GND, internal -V gen), no DC-blocks, V1-V4 + LS control, the loss-tiered 12-port band map, and the all-isolated park (TX-inhibit) state. |
| `bpf-2m-schematic.svg` | TX cleanup BPF for **2 m** (RF8, slice 105–158): highpass+lowpass cascade ladder with designators + values, ports, and a complete per-board BOM (ref / value / type / qty / pkg). |
| `bpf-222-schematic.svg` | TX cleanup BPF for **222** (RF5, slice 158–237): same, with its BOM. |
| `bpf-70cm-schematic.svg` | TX cleanup BPF for **70 cm** (RF4, slice 355–533): same, with its BOM. |
| `bpf-902-schematic.svg` | TX cleanup BPF for **902** (RF3, slice 800–1200): same, with its BOM (high-Q porcelain caps, 0402). |

**Per-band BPF schematics + BOMs** (the `tx-chain.svg` BPF boxes link to these; these markdown links are clickable on GitHub):
[2 m](bpf-2m-schematic.svg) · [222](bpf-222-schematic.svg) · [70 cm](bpf-70cm-schematic.svg) · [902](bpf-902-schematic.svg). Filter synthesis and the value tables live in [`../docs/filters.md`](../docs/filters.md).

**Color conventions:** blue = SDR / digital, teal = LDMOS, purple = GaN (and head-end / UI), gray =
passive / structural, amber dashed = control-wrapper boundary and sense/control lines.
