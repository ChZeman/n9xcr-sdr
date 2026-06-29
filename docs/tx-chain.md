# Transmit chain

From the 7020-SDR's ~1 mW core output to 50 W is ~47 dB of gain, too much for one stage and subject to
a hard linearity requirement (all-mode ⇒ every stage Class AB). The chain is therefore three
amplifier stages with filtering placed where it belongs.

## Signal flow

```
7020-SDR TX with onboard PA (drive set in software, no ALC)
 → TX band-select (solid-state SP12T; routes to the active port/slice)
 → slice-wide bandpass       ← cleans SDR noise/spurs/images BEFORE the gain
 → driver (~5–10 W linear, to ~1 GHz)
 → final PA (Class AB, ~100 W-class device backed off to 50 W)
 → harmonic low-pass filter
 → directional coupler / T-R / antenna
```

The **low-level bandpass is mandatory**: the post-PA low-pass only cleans the high side (the
PA's own harmonics) and does nothing about the AD9361's near-carrier noise, mixer spurs, LO
leakage, and sideband images. Filter clean at low level, *then* amplify. It is cut **slice-wide**
(passing the whole band-select port, not just the ham band), so the hardware can reach any segment
the operator is authorized for; the legal band-limit lives in the software allowed-TX table, not the
filter. See [`filters.md`](filters.md) and [`regions.md`](regions.md).

Everything from the band-select down is **per-port (per-slice)** — the SDR with its onboard PA is
shared — so a band/slice can be added later without touching the SDR or upstream stages.

## Gain budget (50 W out = +47 dBm)

| Band | Final | Final gain | SDR out (onboard PA) | Driver out | Final out |
|------|-------|-----------:|---------------------:|-----------:|----------:|
| 2 m  | LDMOS | 20 dB | +12 dBm (15 mW) | +27 dBm (0.5 W) | 50 W |
| 222  | LDMOS | 18 dB | +14 dBm (24 mW) | +29 dBm (0.8 W) | 50 W |
| 70 cm| LDMOS-class | 15 dB | +17 dBm (48 mW) | +32 dBm (1.6 W) | 50 W |
| 902  | GaN   | 13 dB | +19 dBm (76 mW) | +34 dBm (2.5 W) | 50 W |

Read-offs: the SDR's onboard PA delivers +12 to +19 dBm (set in software per band, so it never compresses). One driver class covers all bands.

## Why 50 W (not 100 W)

100 W barely changes the gain budget (+3 dB everywhere) but roughly doubles the downstream
physics: dissipation/current roughly double (~90 W waste heat per PA in a sealed outdoor enclosure
→ forced-air territory). The benefit is only +3 dB — half an S-unit at the far end. 50 W is the
efficient, thermally-easy sweet spot: single-device finals, ~50–60 W dissipation per PA, ~100 W
supply. (On 2 m / 222 the MRF300AN final makes raising power later a cooling problem, not a device
change — see [`power-thermal.md`](power-thermal.md); the 50 W baseline is unchanged.)

## Devices

- **Pre-driver:** provided by the **7020-SDR's onboard PA** (Mini-Circuits PGA-102+, ~10 dB, up to ~+19 dBm out) — no separate board. The board's PA can be bypassed and a discrete PGA-103+ used instead if more drive headroom is ever wanted.
- **TX band-select:** pSemi **PE42512A** — silicon SP12T, nonreflective (9 kHz–8 GHz), low loss and
  high isolation across HF–UHF, single 3.3 V supply (internal −V generator, no charge pump / DC-blocks),
  LS tied low. It switches at low level right after the SDR; its twelve ports are fixed 1.5:1 sub-octave
  slices spanning 4 m–5.8 GHz, and each *populated* port has its own slice-wide bandpass, driver and
  final downstream. One final spans several slices (broadband amps), so the chain count tracks bands
  built, not ports. (This replaced an earlier ADRF5040 SP4T on the TX path; 2× ADRF5040 SP4T remain on
  the RX path. A Skyworks SKY13xxx-class part is an option where that isolation isn't needed.)
- **Driver:** Wolfspeed/MACOM **CGH40010** GaN HEMT — 10 W (13 W typ PSAT), DC–6 GHz, 28 V, ~18–20 dB gain at VHF/UHF, Class AB linear. Available solder-down pill (CGH40010P) or screw-down flange (CGH40010F); one part covers all bands. At the ~4 W worst-case drive (902 at 80 W) it runs at ~30 % of rating for clean IMD, and it is family-matched to the CGH40120F final. (The 8 W CGH60008D fits electrically but ships as bare die — not hand-solderable — so it is not used.) Full schematic, per-board BOM, bias setpoints and the GaN sequencing rule: [`driver.md`](driver.md).
- **Finals 2 m + 222:** NXP **MRF300AN** — ~300 W LDMOS, 1.8–250 MHz, 50 V, single-ended Class AB;
  run at the 50 W design point where it loafs hard (~17 % of rating) for excellent IMD and low
  junction temperature, with a deferred path to ~150–200 W on added cooling (no device change). Use a
  VHF-tuned board off the NXP reference; covers 2 m and 222 only (≤250 MHz). Avoid counterfeit eBay
  "100 W" (MRF9120) amps.
  *Alternatives considered:* **MRF101AN** (100 W, currently ~$44.45 vs the MRF300AN's ~$88.10) is the
  cheaper part and identical on the air at 50 W, but caps at ~80–100 W and would need a push-pull
  redesign for more. The MRF300AN's ~$44 premium buys headroom-on-tap and is the lower-regret choice:
  never use it and you're out ~$44; hard-cap with the MRF101AN and later want more and you redesign
  the final. Verify both prices live on the page.
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
   SDR drive or inhibits TX. Drive is already set in software, so the supervisory loop closes
   there; this part is not time-critical (the hardware already caught the fast fault).

The wrapper is buildable almost entirely from off-the-shelf boards: an amplifier control board
(bias keying + sequencing), a dual directional detector (SWR), a relay sequencer, a heatsink
thermistor, and the telemetry tie-back to the node host.
