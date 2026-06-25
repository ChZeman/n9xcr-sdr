# Transmit chain

From the AntSDR's ~1 mW output to 50 W is ~47 dB of gain, too much for one stage and subject to
a hard linearity requirement (all-mode ⇒ every stage Class AB). The chain is therefore three
amplifier stages with filtering placed where it belongs.

## Signal flow

```
AntSDR TX (~0 dBm, drive set in software, no ALC)
 → fixed pad (clean level + good 50 Ω load for the AD9361)
 → pre-driver MMIC (broadband gain block, ~+22 dB)
 → TX band-select (solid-state SP4T; routes to the active band)
 → low-level bandpass        ← cleans SDR noise/spurs/images BEFORE the gain
 → driver (~5–10 W linear, to ~1 GHz)
 → final PA (Class AB, ~100 W-class device backed off to 50 W)
 → harmonic low-pass filter
 → directional coupler / T-R / antenna
```

The **low-level bandpass is mandatory**: the post-PA low-pass only cleans the high side (the
PA's own harmonics) and does nothing about the AD9361's near-carrier noise, mixer spurs, LO
leakage, and sideband images. Filter clean at low level, *then* amplify.

Everything from the band-select down is **per-band** (pre-driver and SDR output are
shared), so a band can be added later without touching the SDR or upstream stages.

## Gain budget (50 W out = +47 dBm)

| Band | Final | Final gain | AntSDR drive | Pre-driver out | Driver out | Final out |
|------|-------|-----------:|-------------:|---------------:|-----------:|----------:|
| 2 m  | LDMOS | 20 dB | −7 dBm | +12 dBm (15 mW) | +27 dBm (0.5 W) | 50 W |
| 222  | LDMOS | 18 dB | −5 dBm | +14 dBm (24 mW) | +29 dBm (0.8 W) | 50 W |
| 70 cm| LDMOS-class | 15 dB | −2 dBm | +17 dBm (48 mW) | +32 dBm (1.6 W) | 50 W |
| 902  | GaN   | 13 dB |  0 dBm | +19 dBm (76 mW) | +34 dBm (2.5 W) | 50 W |

Read-offs: the AntSDR only needs −7…0 dBm (well inside its clean range; software sets it per
band, so the SDR never compresses). One MMIC pre-driver and one driver class cover all bands.

## Why 50 W (not 100 W)

100 W barely changes the gain budget (+3 dB everywhere) but roughly doubles the downstream
physics: finals jump a device class, and dissipation/current roughly double (~90 W waste heat
per PA in a sealed outdoor enclosure → forced-air territory). The benefit is only +3 dB — half an
S-unit at the far end. 50 W is the efficient,
thermally-easy sweet spot: single-device finals, ~50–60 W dissipation per PA, ~100 W supply.

## Devices

- **Pre-driver:** broadband MMIC gain block, PGA-103+ class (~22 dB) — one part, all bands.
- **TX band-select:** Analog Devices **ADRF5040** — silicon SP4T, nonreflective (9 kHz–12 GHz), ~0.5 dB loss and >50 dB isolation at HF/UHF, 33 dBm handling, 3.3 V logic. It switches at low level right after the pre-driver, so each band has its own bandpass, driver and final downstream — four chains. (A Skyworks SKY13xxx-class SP4T is a cheaper option where that isolation isn't needed.)
- **Driver:** Wolfspeed/MACOM **CGH40010** GaN HEMT — 10 W (13 W typ PSAT), DC–6 GHz, 28 V, ~18–20 dB gain at VHF/UHF, Class AB linear. Available solder-down pill (CGH40010P) or screw-down flange (CGH40010F); one part covers all bands. At the ~4 W worst-case drive (902 at 80 W) it runs at ~30 % of rating for clean IMD, and it is family-matched to the CGH40120F final. (The 8 W CGH60008D fits electrically but ships as bare die — not hand-solderable — so it is not used.)
- **Finals 2 m + 222:** NXP **MRF101AN** — 100 W LDMOS, 1.8–250 MHz, linear to ~100 W at
  100 mA Idq; at 50 W it loafs for clean IMD. Use a VHF-tuned board (NXP 136–174 MHz reference
  circuit), not the HF-tuned eval deck. Avoid counterfeit eBay "100 W" (MRF9120) amps.
- **Finals 70 cm + 902:** Wolfspeed/MACOM **CGH40120F** — 120 W GaN HEMT, 28 V, broadband
  (DC–~1.5 GHz, spans 432 and 915). One part covers both bands (two builds). MACOM publishes a
  0.8–1.3 GHz reference circuit and DVB linear data, so it's an adapted vendor reference, not a
  scratch design. At 50 W it runs at ~40 % of rating (excellent IMD); run IDQ leaner than the
  1.0 A datasheet value to cut idle heat. GaN efficiency means only ~30–40 W dissipated at 50 W.

## Control / protection wrapper

A bare SDR provides none of a commercial rig's ALC, sequencing, or protection. The wrapper
supplies it, and the hard-real-time parts live in hardware at the PA:

1. **T-R sequencing** — on key-up, switch the coax relays first (cold switching), wait ~100 ms,
   then apply PA bias; reverse on key-down.
2. **Gate-bias keying** — bias applied only during transmit (set Idq), off in receive so the PA
   is cold and draws nothing.
3. **SWR / reflected-power foldback** — a dual directional detector samples forward/reflected;
   an SWR spike cuts drive or shuts down.
4. **Thermal foldback + fan** — a heatsink sensor ramps the fan and, past a limit, reduces drive.
5. **Overcurrent / overdrive trip** — watch drain current; cut bias and exciter fast on a fault.
6. **"Software ALC"** — the control board's alarms feed back to the node host, which reduces the
   AntSDR drive or inhibits TX. Drive is already set in software, so the supervisory loop closes
   there; this part is not time-critical (the hardware already caught the fast fault).

The wrapper is buildable almost entirely from off-the-shelf boards: an amplifier control board
(bias keying + sequencing), a dual directional detector (SWR), a relay sequencer, a heatsink
thermistor, and the telemetry tie-back to the node host.
