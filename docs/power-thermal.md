# Power, thermal, and maximum safe output

The 50 W baseline is the thermally-easy design point, but the chain is built so output can be raised
toward ~80 W without re-architecting. "Maximum safe output" is not a single number — it is per-band,
thermally bounded, and enforced in real time by the control wrapper rather than fixed by nameplate.

Device figures below are datasheet-verified: **MRF101AN** — 100 W CW, 1.8–250 MHz, 50 V, IDQ 100 mA,
~76 % drain efficiency and ~21 dB gain at VHF (saturated, in the NXP reference fixtures);
**CGH40120F** — 120 W PSAT, 28 V, IDQ 1.0 A, 70 % efficiency at PSAT, 20 dB small-signal gain at
1 GHz, rated for Class AB linear.

## The devices have the headroom

| Band | Device | Rating | at 50 W | at 80 W |
|------|--------|-------:|--------:|--------:|
| 2 m / 222 | MRF101AN | 100 W | 50 % | 80 % |
| 70 cm / 902 | CGH40120F | 120 W | 42 % | 67 % |

Near rating a linear amplifier's IMD and harmonics climb, so the LDMOS at 80 W (80 % of rating) sits
closer to its clean-linear edge, while the GaN at 80 W (67 %) stays comfortable. The GaN bands take
80 W more gracefully than the LDMOS bands.

## Thermal is the real limit

Dissipation, not the silicon, sets the ceiling in a sealed fanless enclosure. The datasheet
efficiencies above are **saturated** figures; running the finals backed off for all-mode linearity
derates them to roughly **~52 % (LDMOS)** and **~58 % (GaN)** at the operating point. At those:

| Output | LDMOS dissipated | GaN dissipated |
|-------:|-----------------:|---------------:|
| 50 W | ~46 W | ~36 W |
| 60 W | ~55 W | ~43 W |
| 70 W | ~65 W | ~51 W |
| 80 W | ~74 W | ~58 W |

A sealed, sun-exposed, fanless enclosure can shed only so much per PA — order ~60–75 W into a
heatsink wall with a sun shield. So **both device types can approach 80 W passively**, with the
LDMOS bands sitting right at the limit (~74 W dissipated at 80 W) and the GaN bands comfortably
inside it (~58 W). An earlier worst-case estimate put LDMOS dissipation near 100 W; that assumed a
lower efficiency than the datasheet supports. The real figure is better — but the LDMOS bands still
have the least thermal margin at 80 W.

## Continuous-duty rating

The 80 W figure above is a **peak / low-duty** number. Continuous duty — 100 % key-down, no
cool-down between overs — is a separate and lower rating, set purely by how fast heat leaves the
sealed enclosure in the steady state. With the same linear efficiencies and a realistic worst-case
passive budget (~45–55 W per PA in full sun, more when cooler or shaded):

| Band | Device | Continuous-duty output (fanless) | Dissipated |
|------|--------|---------------------------------:|-----------:|
| 2 m / 222 | MRF101AN | ~50 W | ~46 W |
| 70 cm / 902 | CGH40120F | ~60–65 W | ~43–47 W |

So the all-band continuous figure lands right around the 50 W baseline — a bit higher (~60–65 W) on
the GaN bands, which dissipate less per watt. This is not a coincidence: 50 W was chosen as the
thermally-easy point, and it turns out to be essentially the fanless 100 %-duty rating.

**Which modes care about the difference:**

- **SSB (~20–25 % average) and CW (~40–50 %):** average dissipation is well below key-down, so the
  full 80 W peak is fine — the heatsink never sees sustained 80 W of heat.
- **FM, RTTY, AM, ATV, beacon carrier:** true 100 % duty — the continuous numbers above govern.
- **FT8 / FT4:** each transmission is a multi-second full-power key-down, and back-to-back periods
  approach steady state, so digital behaves as continuous duty for thermal purposes. For an
  unattended, largely-digital station this is the rating that matters most.

**What moves it:** the continuous rating is a property of the heatsink and worst-case ambient, not
the device — a larger finned wall or a cooler/shaded mount pushes the LDMOS bands toward ~60–65 W
and the GaN bands toward ~80 W; harsh sun pulls them down. The temperature-controlled fan on the
LDMOS bands raises their steady-state budget enough to reach continuous 80 W. And thermal foldback
enforces whatever the real limit is on the day: configure an 80 W ceiling and, during a long
key-down, the wrapper trims output to the sustainable level as the sink heats — so the station
self-limits to its true continuous-duty power instead of overheating.

**Plan on ~50 W continuous as the safe all-band figure (~60–65 W on the GaN bands), treat 80 W as
headroom for SSB/CW peaks and short digital, and let foldback — plus the optional LDMOS fan — cover
the rest.**


## What it takes to support 80 W

| Item | At 50 W | At 80 W |
|------|---------|---------|
| Driver output (worst case, 902) | ~2.5 W | ~4 W — same 5–10 W driver, run harder |
| Onboard PA, filters, control wrapper | — | unchanged |
| 50 V LDMOS supply | ~2.5 A | ~5 A |
| 28 V GaN supply | ~2.5 A | ~6 A |
| LDMOS-band cooling | passive | passive, plus a temperature-controlled fan for sustained high duty |
| GaN-band cooling | passive | passive |

The CGH40120F's 20 dB small-signal gain (and the MRF101AN's ~21 dB) mean the finals actually need
*less* drive than the conservative gain budget assumes, so the driver keeps margin even at 80 W. The
only real changes for 80 W are the higher supply current and a fan on the LDMOS bands for long,
high-duty transmissions.

## Maximum safe output is enforced, not assumed

Rather than commit to a fixed wattage, build the chain with ~80 W of headroom and let the protection
wrapper hold the live safe limit:

- a per-band **configured power ceiling** — e.g. 80 W on the GaN bands; 60–80 W on the LDMOS bands
  depending on duty and whether the fan is fitted — and
- **thermal foldback** that trims drive as the heatsink warms.

The amplifier then delivers up to its ceiling when the enclosure is cool and automatically backs off
when sun load or a long high-duty transmit pushes the heatsink up. "Maximum safe output" becomes a
live, self-protecting value, shown in the UI (see `telemetry.md`) — 80 W is an opportunistic peak,
not a standing thermal liability.

## Practical recommendation

- **GaN bands (70 cm, 902):** specify 80 W. It is within the passive thermal budget and well inside
  the device's linear range — nearly free.
- **LDMOS bands (2 m, 222):** comfortable at 50–60 W fully passive; 80 W is reachable but sits at the
  passive limit, so fit a temperature-controlled fan that only spins up under sustained high power
  (FM, long digital transmissions). Normal SSB/CW duty is fine without it.
- Size the supplies and driver for the 80 W case from the start (cheap insurance) — 50 V at ~5 A
  (LDMOS), 28 V at ~6 A (GaN), driver to ~4 W — so raising a band's ceiling later is a configuration
  change, plus the fan on the LDMOS bands.
