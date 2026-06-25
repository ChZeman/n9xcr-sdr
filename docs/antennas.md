# Antennas and band combining

A multiband station can give each band its own antenna, or share one antenna across several
bands. Separate antennas are simplest electrically and gain free isolation from physical spacing;
sharing reduces the number of antennas at the cost of frequency-domain combiners and more careful
filtering. Small, low-profile antennas are preferred where practical — unobtrusive, with low wind
and ice loading — which tends to favor sharing a compact multiband whip over hanging several
elements.

## Combiners: diplexers, triplexers, and duplexers

When two or more bands share one antenna and feedline, a passive combiner splits and joins them by
frequency so each band reaches its own radio chain:

- **Diplexer** — two ports combined onto one common port, split by frequency (for example a VHF
  band and a UHF band on one feed). Passive, low loss, bidirectional.
- **Triplexer** — the same idea for three bands (for example 2 m / 70 cm / 33 cm). Each band port
  connects to its own T-R switch, PA / low-pass filter, and receive preselector.
- **Duplexer** — splits **transmit from receive on the same band**, so a station can transmit and
  receive simultaneously through one antenna (the classic repeater part). This is a harder job
  than a di/triplexer because the two ports are close in frequency: a duplexer uses sharp cavity
  resonators to get high transmit-to-receive isolation. It is needed only for **same-band
  full-duplex**.

Rule of thumb: **di/triplexers separate different bands; a duplexer separates transmit from
receive within one band.** Reach for a di/triplexer to share an antenna across bands; reach for a
duplexer only when one band must transmit and receive at the same time on one antenna. Cross-band
full-duplex (transmit one band, receive another) does **not** need a duplexer — a di/triplexer plus
per-band filtering handles it.

### What sharing costs

With separate antennas, physical spacing provides 20–40 dB of isolation between bands for free. A
shared antenna gives that up — all isolation then rides on the combiner's port-to-port isolation
(typically ~50–60 dB) plus the per-band transmit low-pass and receive preselector. That is fine
for FM and FM-satellite work; it is tight for weak-signal SSB, where separate antennas win.

Transmit filtering matters more on a shared feed, because a harmonic that lands in another shared
band has a direct desense path — for example a 2 m third harmonic at 432 MHz sitting in the 70 cm
receive band, or a 70 cm second harmonic near 900 MHz. Keep antennas per-band-separable in the
plan so the choice stays open.

## Worked example: one tri-band whip via a triplexer

A single wideband tri-band NMO whip (covering roughly 2 m / 440 / 900) plus a triplexer puts three
bands on one unobtrusive antenna; each band port still gets its own T-R, PA/LPF, and preselector.
Verify SWR per band on the actual whip — wideband whips trade efficiency for coverage — especially
at 902. Any band not on the shared whip gets a dedicated small whip; 222 MHz is the notable gap, as
it falls in the dead zone of essentially every wideband tri-bander.

**Sourcing note:** most stock triplexers are 144 / 440 / 1200, not 144 / 440 / 902. A
transmit-rated triplexer with a real 900 MHz port is less common — verify before committing.

## Receive-only wideband antenna

A separate receive-only wideband element (discone-class, ~50 MHz – 1.3 GHz) covers everything the
transmit antennas do not: 4 m, FM broadcast, the inter-band gaps, and ADS-B at 1090 MHz. Because it
never transmits it needs no T-R or PA — just a switchable FM band-stop (FM broadcast is the one
guaranteed overloader) ahead of a bypassable LNA. Targets above ~1.3 GHz (L-band satellite,
weather-sat imagery, the 1420 MHz hydrogen line) want their own dish or horn.

## Note on 6 m

An efficient 6 m antenna cannot be made small, so it does not join a compact multiband-whip set —
it would be a separate, larger element.
