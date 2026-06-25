# Power, thermal, and maximum safe output

The 50 W baseline is the thermally-easy design point, but the chain is built so output can be raised
toward ~80 W without re-architecting. "Maximum safe output" is not a single number — it is per-band,
thermally bounded, and enforced in real time by the control wrapper rather than fixed by nameplate.

## The devices have the headroom

| Band | Device | Rating | at 50 W | at 80 W |
|------|--------|-------:|--------:|--------:|
| 2 m / 222 | MRF101AN | 100 W | 50 % | 80 % |
| 70 cm / 902 | CGH40120F | 120 W | 42 % | 67 % |

Near rating, a linear amplifier's IMD and harmonics climb, so the LDMOS at 80 W (80 % of rating) sits
close to its clean-linear edge, while the GaN at 80 W (67 %) is still comfortable. The GaN bands take
80 W far more gracefully than the LDMOS bands.

## Thermal is the real limit

In a sealed, fanless enclosure, dissipation — not the silicon — sets the ceiling. Assuming ~45 %
LDMOS and ~58 % GaN drain efficiency (realistic for Class AB linear):

| Output | LDMOS dissipated | GaN dissipated |
|-------:|-----------------:|---------------:|
| 50 W | ~61 W | ~36 W |
| 60 W | ~73 W | ~43 W |
| 70 W | ~85 W | ~51 W |
| 80 W | ~98 W | ~58 W |

A sealed, sun-exposed, fanless enclosure can shed only so much per PA (order ~60–75 W into a heatsink
wall with a sun shield). That puts the **passive ceiling at roughly 60 W for the LDMOS bands and
~85–90 W for the GaN bands.** Uniform 80 W therefore means the LDMOS bands (~98 W dissipated) need
**active cooling**, while the GaN bands reach 80 W within the passive budget.

## What it takes to support 80 W

| Item | At 50 W | At 80 W |
|------|---------|---------|
| Driver output (worst case, 902) | ~2.5 W | ~4 W — same 5–10 W driver, run harder |
| Pre-driver, filters, control wrapper | — | unchanged |
| 48 V LDMOS supply | ~2.5 A | ~5 A |
| 28 V GaN supply | ~2.5 A | ~6 A |
| LDMOS-band cooling | passive | active (temp-controlled fan or larger heatsink) |
| GaN-band cooling | passive | passive |

The only structural change for 80 W is the LDMOS bands' cooling; everything else is a component-value
bump.

## Maximum safe output is enforced, not assumed

Rather than commit to a fixed wattage, build the chain with ~80 W of headroom and let the protection
wrapper hold the live safe limit:

- a per-band **configured power ceiling** — e.g. 80 W on the GaN bands; 60 W passive or 80 W with a
  fan on the LDMOS bands — and
- **thermal foldback** that trims drive as the heatsink warms.

The amplifier then delivers up to its ceiling when the enclosure is cool, and automatically backs off
when sun load or a long high-duty transmit pushes the heatsink up. "Maximum safe output" becomes a
live, self-protecting value, shown in the UI (see `telemetry.md`) — 80 W is an opportunistic peak,
not a standing thermal liability.

## Practical recommendation

- **GaN bands (70 cm, 902):** specify 80 W. It is within the passive thermal budget and well inside
  the device's linear range — nearly free.
- **LDMOS bands (2 m, 222):** specify 50–60 W passive, with a temperature-controlled fan as the
  option that unlocks 80 W when wanted. This keeps the enclosure fanless in normal use and only spins
  up under sustained high power.
- Size the supplies and driver for the 80 W case from the start (cheap insurance), so raising a
  band's ceiling later is a configuration change — plus, on the LDMOS bands, adding the fan.
