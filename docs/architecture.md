# Architecture

## Layers

- **Radio (per node):** the SDR(s) plus the per-band PA / filter / sequencer / band-router hardware.
- **Node host (per node):** a small computer running the DSP engine, the digital-mode
  applications, the band-router / sequencer logic, and a dial-home agent. Disciplined to
  GPS time for FT8.
- **Transport:** each node connects **outbound** to a shared head-end. A hard rule governs
  what crosses the link.
- **Head-end + web UI:** aggregates the nodes and presents one control surface (waterfall,
  full-duplex audio, PTT, band/mode/frequency, decode panels).

### Hard rule: raw IQ never leaves the site

Raw IQ stays on the node's local network. Only **demodulated audio**, **FFT frames** for the
waterfall, and **decoded text** travel to the head-end. This is the WebSDR / KiwiSDR pattern:
the browser is a thin client, all heavy DSP stays at the node. A 2 Msps IQ stream is ~64 Mbps
and would melt a remote link; Opus audio is ~24 kbps.

## DSP engine

Do not write a DSP stack from scratch — wrap **SDRangel headless** as each node's DSP brain.
It already drives the Hermes-Lite 2 and the 7020-SDR under one API, has every mode native
(SSB/AM/FM/CW, DMR/D-STAR/YSF/NXDN/M17/FreeDV, built-in FT8), uses an AMBE dongle for the
proprietary vocoders and Codec2 for the open ones, and exposes a REST + WebSocket API the
custom control surface can drive. The engine is a node implementation detail; the web stack
is the product.

## CW and digital modes run at the node

These are timing-critical and are never streamed as audio across the link:

- **CW:** transmit = envelope-shaped keying of the exciter at the node; receive = a CW modem
  at the node. Only text and keying control cross the link.
- **FT8 / JS8 / fldigi / WSJT-X:** the actual applications run on the node host. SDRangel
  demodulates to a **virtual audio sink** (PipeWire null-sink or ALSA loopback); the app
  decodes; CAT/PTT goes through rigctld. Only decodes and spots cross the link. WSJT-X (a Qt
  GUI with no daemon) runs under Xvfb and is consumed via its UDP decode protocol, or surfaced
  over a remote desktop; fldigi exposes XML-RPC.

## Full-duplex — two senses, both delivered

1. **Full-duplex audio UX** — hear receive while talking, phone-style, over the media transport.
2. **True RF full-duplex** — the AD9361 is 2T2R with independent TX/RX local oscillators, so a
   single SDR node can receive on one band while transmitting on another at the same time.

## Crossband and satellite capability

Crossband full-duplex and amateur-satellite work are the same problem: transmit one band while
receiving your own downlink on another.

- Independent **TX band-select** and **RX band-select** make crossband natural. It works between
  bands in **different filter segments** (bands sharing one TX chain cannot crossband):
  2 m ↔ 70 cm, 2 m ↔ 902, 70 cm ↔ 902 are fine; **2 m ↔ 222 is not** (shared VHF segment).
- The real limiter is isolation: the TX low-pass scrubs harmonics/noise and the RX preselector
  rejects the opposite band at the front end. Separate antennas add free spatial isolation; a
  shared antenna puts all of it on the triplexer + filters.
- Satellite: FM birds and most linear transponders live in the 2 m (145.8–146.0) and 70 cm
  (435–438) sub-bands — both covered, different segments, so the simultaneous-operation rule is
  satisfied. Doppler is handled in software (e.g. gpredict driving Hamlib). A clear
  horizon gives horizon-to-horizon LEO coverage with an omni vertical.

Build switched RX-preselector slots and keep antennas per-band-separable so crossband/satellite
can be enabled later without disturbing the basic build.
