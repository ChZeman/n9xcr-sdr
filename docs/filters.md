# Filters

Two different filters do two different jobs: a **transmit harmonic low-pass** on each PA output
(transmit cleanliness), and a **receive bandpass preselector** on the receive side (front-end
protection + crossband isolation). They are kept separate so the preselector adds no insertion
loss to the 50 W transmit path and no noise figure penalty isn't forced onto a shared part.

> **Region note:** the cutoffs below are the US / IARU R2 worked example (2 m / 222 / 70 cm / 902). The synthesis method applies to any allocation - recompute fc and the L/C (or resonator) values for your slices. See [`regions.md`](regions.md).

> **Scope note (TX path):** the TX filters are spec'd **per band-select slice (the full port range), not per ham band.** The node hardware is built to transmit the whole 1.5:1 slice of each populated port; the legal band-limit is enforced only in the node's allowed-TX frequency table (see [`tx-chain.md`](tx-chain.md)). This lets a licensed operator with MARS/CAP authority work the assigned segments adjacent to a ham band (e.g. CAP VHF ~148-150, just above 2 m) without the hardware fencing them out. The **RX preselectors below stay narrow** (per band) - their job is front-end protection and crossband isolation, which a wide passband would defeat.

## TX low-pass bank

Per-slice sub-octave low-pass filters: **fc sits just above the top of the band-select slice**, so one
filter cleans the 2nd harmonic across the whole port. All are **9-element, 0.1 dB-ripple Chebyshev,
50 Ω, shunt-first, symmetric** (ladder C1-L2-C3-L4-C5-L6-C7-L8-C9). The 9-element order is the "build
once" choice — far past the −43 dBc requirement across the slice — vs the ~45 dB of a 7-element.

A 1.5:1 slice is sub-octave, so the 2nd harmonic of the slice *bottom* (2× bottom) lands above the
slice *top*: a single LPF cutoff just over the top therefore covers the entire slice. The rejection is
strongest at the top of the slice and weakens toward the bottom — the `2nd-harmonic rej` column below
is the **worst in-band case** (taken at the low edge of the ham band, the lowest frequency normally
keyed). A slice's literal bottom few MHz dips under −43 dBc on the LPF alone, but that sliver sits below
the lowest allocation in every slice and the slice-wide TX BPF's upper skirt covers it; see *Usable
windows*.

| Band / slice | fc | C1=C9 | L2=L8 | C3=C7 | L4=L6 | C5 (ctr) | 2nd-harmonic rej |
|------|----|------:|------:|------:|------:|---------:|-----------------:|
| 2 m   — slice 105–158  | 165 MHz | 23.1 pF | 69.6 nH | 41.2 pF | 78.0 nH | 42.5 pF | 68 dB @ 288 |
| 222   — slice 158–237  | 250 MHz | 15.2 pF | 45.9 nH | 27.2 pF | 51.5 nH | 28.1 pF | 70 dB @ 444 |
| 70 cm — slice 355–533  | 555 MHz |  6.9 pF | 20.7 nH | 12.2 pF | 23.2 nH | 12.6 pF | 54 dB @ 840 |
| 902   — slice 800–1200 |1250 MHz |  3.0 pF |  9.2 nH |  5.4 pF | 10.3 nH |  5.6 pF | 49 dB @ 1804 |

Synthesis: analytic Matthaei g-values, L = g·Z0/(2π·fc), C = g/(2π·fc·Z0), Z0 = 50 Ω.

Raising fc to the slice top (vs ~1.1× the ham-band top) trades a little harmonic margin **on the ham
band** — 70 cm drops from ~64 dB to ~54 dB at 840, 902 from ~70 dB to ~49 dB at 1804 — but stays
comfortably past −43 dBc. 2 m and 222 are unchanged: their old band-scoped cutoffs already cleared the
slice top.

**These L/C values are NanoVNA starting points, not a final BOM.** Board parasitics pull them,
increasingly so with frequency:

- 2 m and 222 land close to the computed values. 70 cm and especially 902 need real tuning — at
  ~1 GHz the 3 pF / 9–10 nH parts are small enough that board parasitics rival them, the edge of
  where lumped-LC behaves.
- Capacitors: high-Q RF/power-rated NP0/C0G or porcelain (ATC 100B-type); hit odd values by
  paralleling standards. Watch capacitor self-resonance — a part fine at HF can self-resonate in
  a VHF stopband and ruin rejection.
- Inductors: air-wound, few turns; tune by squeezing/spreading turns against the analyzer. At
  902 they are down to ~1 turn where geometry is everything.

### Usable windows

A harmonic low-pass is usable only in the ~1.5:1 span just below cutoff (below that, the second
harmonic falls back into the passband). For these 9-element filters: −3 dB at 1.041·fc, 43 dB at
1.370·fc, 60 dB at 1.608·fc.

| Filter | passband | usable ≥43 dBc | usable ≥60 dBc | slice |
|--------|---------:|---------------:|---------------:|------:|
| 2 m   | DC–165  | 113–165 MHz | 133–165 | 105–158 |
| 222   | DC–250  | 171–250 MHz | 201–250 | 158–237 |
| 70 cm | DC–555  | 380–555 MHz | 446–555 | 355–533 |
| 902   | DC–1250 | 856–1250 MHz |1005–1250 | 800–1200 |

The ≥43 dBc window covers the top ~97 % of each slice; the under-margin sliver at the slice
bottom (e.g. 105–113 on 2 m, 355–380 on 70 cm, 800–856 on 902) always sits **below the ham band**
(2 m starts at 144, 70 cm at 420, 902 at 902), so it is never keyed. The ham band itself sits high
in its window, inside or just under the 60 dBc region. The software allowed-TX table is what keeps
operation inside the clean part of the slice.

## TX cleanup bandpass (per slice)

The low-level bandpass sits at the band-select output (~+19 dBm / 80 mW), ahead of the driver. Its
job is to limit the AD9361's wideband noise and far spurs/images **before** the gain — not to sharpen
the band edges (near-carrier LO leakage and the I/Q image are an SDR-calibration problem, not a
filtering one). It is spec'd **slice-wide** (~1.5:1) so it never fences the band; software sets the
legal limit. Its real unique value is the **high-pass edge** (killing sub-slice noise/spurs feeding
the driver); the post-final LPF backstops the harmonics.

**Catalog vs fab — two Mini-Circuits routes evaluated:**

- **LTCC ceramic bandpass (BFCN series) — not a fit.** The family only runs ~950 MHz–13.2 GHz (nothing
  for 2 m / 222 / 70 cm), and the parts are narrow channel filters (single-digit-% to ~20 % BW), far
  narrower than a 1.5:1 slice.
- **Lumped-element SMT bandpass (BPF- series) — the right family.** Lumped-LC, shielded SMD, 50 Ω,
  rated 0.08–5 W (our 80 mW is trivial), and it reaches down to VHF. Most BPF-A/B/C parts are *also*
  narrow channel filters, but a few **wide** variants (BPF-AM / BPF-BY) span a useful fraction of a
  slice. They don't land on the 1.5:1 slices exactly, so a catalog part means accepting its passband
  (usually ham-band-plus — fine, since MARS/CAP sits within a few MHz of the band).

**Per-slice candidates (US/R2 worked bands):**

| Slice (band) | Catalog candidate | Fit |
|--------------|-------------------|-----|
| 2 m, 105–158 (144–148) | none wide | falls in a gap between narrow parts (BPF-A122 ~120, BPF-A175 ~175) — **fab LC** |
| 222, 158–237 (222–225) | **BPF-BY250+** (150–350 MHz) | covers the whole slice — drop-in ✓ |
| 70 cm, 355–533 (420–450) | **BPF-AM585+** (420–750 MHz) | covers the ham band + above; misses 355–420 — band-OK, not full-slice |
| 902, 800–1200 (902–928) | none wide (only narrow BPF-A800 795–805) | **fab LC** |

Practical path = **hybrid**: catalog SMT where a wide part lands on the band (222 cleanly, 70 cm for
the ham segment), fab a lumped-LC bandpass (or an HP+LP cascade) where it doesn't (2 m, 902). Fabbing
all four is the alternative if one design method and true slice-wide everywhere is worth more than
skipping two tuning jobs. Decision pending.

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
