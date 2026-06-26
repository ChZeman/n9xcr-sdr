# Filters

Two different filters do two different jobs: a **transmit harmonic low-pass** on each PA output
(transmit cleanliness), and a **receive bandpass preselector** on the receive side (front-end
protection + crossband isolation). They are kept separate so the preselector adds no insertion
loss to the 50 W transmit path and no noise figure penalty isn't forced onto a shared part.

> **Region note:** the passbands and cutoffs below are the US / IARU R2 worked example (2 m / 222 / 70 cm / 902). The synthesis method applies to any allocation - recompute fc and the L/C (or resonator) values for your bands. See [`regions.md`](regions.md).

## TX low-pass bank

Per-band sub-octave low-pass filters: each ham band's second harmonic lands ~1.7–1.9× the
cutoff, where a simple Chebyshev crushes it (64–70 dB). All are **9-element, 0.1 dB-ripple
Chebyshev, 50 Ω, shunt-first, symmetric** (ladder C1-L2-C3-L4-C5-L6-C7-L8-C9). The 9-element
order is the "build once" choice — 64–70 dB second-harmonic rejection vs the ~45 dB of a
7-element, both clearing the −43 dBc requirement.

| Band | fc | C1=C9 | L2=L8 | C3=C7 | L4=L6 | C5 (ctr) | 2nd-harmonic rej |
|------|----|------:|------:|------:|------:|---------:|-----------------:|
| 2 m (144–148)  | 165 MHz | 23.1 pF | 69.6 nH | 41.2 pF | 78.0 nH | 42.5 pF | 68 dB @ 288 |
| 222 (222–225)  | 250 MHz | 15.2 pF | 45.9 nH | 27.2 pF | 51.5 nH | 28.1 pF | 70 dB @ 444 |
| 70 cm (420–450)| 500 MHz |  7.6 pF | 23.0 nH | 13.6 pF | 25.7 nH | 14.0 pF | 64 dB @ 840 |
| 902 (902–928)  |1010 MHz |  3.8 pF | 11.4 nH |  6.7 pF | 12.7 nH |  7.0 pF | 70 dB @ 1804 |

Synthesis: analytic Matthaei g-values, L = g·Z0/(2π·fc), C = g/(2π·fc·Z0), Z0 = 50 Ω.

**These L/C values are NanoVNA starting points, not a final BOM.** Board parasitics pull them,
increasingly so with frequency:

- 2 m and 222 land close to the computed values. 70 cm and especially 902 need real tuning — at
  ~1 GHz the 3.8 pF / 11 nH parts are small enough that board parasitics rival them, the edge of
  where lumped-LC behaves.
- Capacitors: high-Q RF/power-rated NP0/C0G or porcelain (ATC 100B-type); hit odd values by
  paralleling standards. Watch capacitor self-resonance — a part fine at HF can self-resonate in
  a VHF stopband and ruin rejection.
- Inductors: air-wound, few turns; tune by squeezing/spreading turns against the analyzer. At
  902 they are down to 1–2 turns where geometry is everything.

### Usable windows

A harmonic low-pass is usable only in the ~1.5:1 span just below cutoff (below that, the second
harmonic falls back into the passband). For these 9-element filters: −3 dB at 1.041·fc, 43 dB at
1.370·fc, 60 dB at 1.608·fc.

| Filter | passband | usable ≥43 dBc | usable ≥60 dBc |
|--------|---------:|---------------:|---------------:|
| 2 m   | DC–165  | 113–165 MHz | 133–165 |
| 222   | DC–250  | 171–250 MHz | 201–250 |
| 70 cm | DC–500  | 343–500 MHz | 402–500 |
| 902   | DC–1010 | 692–1010 MHz | 812–1010 |

Each ham band sits in the top of its window, comfortably inside the 60 dBc region.

## RX preselectors

A preselector is a bandpass with both skirts real, doing double duty: protecting the wide-open
AD9361 front end from a busy RF environment, and providing anti-desense isolation for crossband
full-duplex.

Key insight: the preselector is **loss-bound, not rejection-bound**. The ham bands are narrow, so
anything cross-band sits dozens of bandwidths away and is rejected for free (75–117 dB by
analysis); the whole challenge is passing the band with low loss, which adds straight to noise
figure. That demands high-Q resonators (helical / cavity), not lumped LC.

Topology (3-pole, 0.1 dB Chebyshev bandpass):

| Band | passband | est. insertion loss | resonator |
|------|---------:|--------------------:|-----------|
| 2 m   | 144–148 | ~0.5 dB | helical |
| 222   | 222–225 | ~0.9 dB | helical |
| 70 cm | 420–450 | ~0.3 dB | helical / interdigital |
| 902   | 902–928 | ~1.2 dB | interdigital / combline / cavity |

The real rejection challenge is **close-in**, not cross-band: pagers just above 2 m and cellular
just below 902 are the near-neighbors that test the skirts — add a notch or a 4th pole if strong
on site.

**Placement:** preselector first, then a switch-bypassable LNA. With one's own 50 W
nearby, overload protection beats the small NF penalty; ~0.5 dB preselector loss + a good LNA
gives ~1 dB system NF.

### Off-the-shelf

Narrow bands are often cheaper to buy than to build and tune:

- **2 m:** DCI-146-4H (4-pole cavity, full 144–148).
- **222 and 70 cm:** Temwell helical bandpass (one source for the less-common bands).
- **902:** FreeWave EBF901 or Fairview cavity (902–928).
- **Quick-start:** Nooelec Ham Filter Bank — one RX-only SAW module covering all four bands, to
  validate the receive chain before the cavities arrive (SAW = receive-only, 100 mW).

Real cavity/helical ultimate rejection is ~55–70 dB (not the ideal-Chebyshev 85–115 dB), still
far beyond the desense budget.

### Wideband receive antenna

The single wideband receive-only antenna gets a **switchable FM band-stop** (88–108 notch,
~20–30 dB, commodity), since FM broadcast is the one guaranteed overloader on that wide feed.
