# Radios and clocking

## HF — Hermes-Lite 2 (example HF radio)

Direct-sampling HPSDR transceiver, 0.5–30 MHz, ~5 W, 12-bit / 76.8 MS/s. PureSignal-capable
(adaptive predistortion) which bare USB SDRs cannot do. Network-native, headless, all-mode —
fits the dial-home node pattern. At ~5 W it nearly direct-drives a 100 W LDMOS pallet, so the
HF transmit chain needs **no milliwatt driver buildout**. Any HPSDR-class HF SDR fills this role.

## VHF/UHF — OpenSourceSDRLab 7020-SDR (AD9361)

The VHF/UHF node radio — a Pluto/AntSDR-class board; the AntSDR E200 (Mouser `ANTSDR-AD9361-With-CASE-01`) is the drop-in equivalent.

- AD9361, 70 MHz – 6 GHz, 2T2R **full-duplex** (independent TX/RX LOs).
- Zynq-7020 SoC (dual Cortex-A9 + Artix-7 fabric): a standalone networked SDR-computer — it can
  host the dial-home agent itself, and the fabric can run a predistortion loop to linearize the
  LDMOS.
- **Onboard PA** (Mini-Circuits PGA-102+, ~10 dB) lifts TX output to ~+15–19 dBm across the bands, covering the pre-driver stage of the transmit chain (see `tx-chain.md`); it can be bypassed for ~0 dBm if a discrete pre-driver is preferred.
- Runs a **PlutoSDR firmware fork** (plutosdr-fw): standard **libiio / PlutoSDR** interface over USB and Gigabit Ethernet, so SDRangel connects via its PlutoSDR plugin — the same stack as the AntSDR. OpenSourceSDRLab publishes the firmware, hardware and FPGA project on GitHub.
- External 10 MHz / 1 PPS reference input and a 0.5 ppm VCTCXO on board.
- **Order the AD9361 version**, not AD9363 (which floors at 325 MHz).

The 70 MHz floor is why there is no 6 m on this board.

## Digital voice — DVMEGA DVstick 30

Genuine DVSI AMBE-3000 hardware vocoder (DMR + D-STAR + C4FM Fusion). Only needed for the
proprietary vocoder modes; **M17 and FreeDV are open Codec2 and need no dongle** (software on
the node). Request the genuine DVSI part; clones fail. The FTDI/FT234 USB-serial variant is
smoothest on a headless Linux node and is the one required for XLX reflector transcoding.

## Clocking — single 10 MHz GPSDO per site

A GPSDO is a GPS-disciplined oscillator: an internal GPS receiver steers a local OCXO, giving
atomic-clock long-term accuracy with clean short-term stability. Do **not** use a bare GPS
module's raw frequency output — poor short-term stability and phase noise.

- The HL2 and the 7020-SDR both discipline to **10 MHz**, so one 10 MHz GPSDO locks the whole
  site. Needs a small GPS puck with sky view.
- **Multi-site coherence for free:** every site's GPSDO locks to the same GPS constellation, so
  separate sites share one frequency reference with no cable between them.
- A programmable GPSDO (e.g. Leo Bodnar type) futureproofs the 122.88 MHz requirement of the
  HF-board swap below.

## 6 m and the future HF-board swap

6 m (50–54 MHz) is deferred. The upgrade path is to swap the HF radio to a **Red Pitaya
SDRlab 122-16** — a direct-sampling board (122.88 MS/s, 16-bit) whose first Nyquist zone
(~61 MHz) reaches 6 m on the **real displayed frequency**, no transverter. Its 16-bit ADC also
gives ~12 dB more HF dynamic range, and its Zynq-7020 matches the 7020-SDR for architectural
symmetry. OpenSourceSDRLab offers a Red Pitaya-class board for this swap, the **TRX-duo** (dual 16-bit ADC, 10 kHz–60 MHz direct-sampling RX reaching 6 m, dual 14-bit DAC TX, Zynq-7010 + Gigabit Ethernet, network-native and headless). The catch is it wants a 122.88 MHz reference (vs the 7020-SDR's 10 MHz), so the swap
adds a 122.88 MHz GPSDO output. Everything downstream of the exciter is unaffected — the swap
is contained to: 6 m appears, add an HF milliwatt driver stage, and add the 122.88 MHz clock.

The 30–70 MHz span otherwise holds no US amateur allocation (vacated low-VHF TV, FM broadcast,
aero nav) — nothing lost by deferring it.
