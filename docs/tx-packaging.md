# TX chain packaging

The transmit counterpart to `rx-frontend.md`: how to physically build and shield the transmit chain
(band-select and the four per-band bandpass → driver → final → low-pass chains). The
signal design is in `tx-chain.md`; the thermal design is in `power-thermal.md`.

## The TX side can't dodge custom boards

On RX you can buy finished, shielded filter modules and skip most of the hard work. The TX power
stages have no equivalent off-the-shelf shielded module at the 50 W linear target — the driver and
finals are fabbed from vendor reference designs (see `boards.md`). So the TX chain is committed to
custom boards either way; the only real question is how to package and shield them.

## Modular vs integrated

**Per-band PA modules (recommended for node one).** Build each band as its own module —
bandpass → CGH40010 driver → final → LPF — on its own board in its own shielded, heatsinked box, fed
by a shared ADRF5040 band-select board. Easy to build and bench-test one band at a time,
easy to service (swap one band's module), and because the band-select means **only one PA is ever
keyed at a time**, the modules share thermal and isolation budgets cleanly. This also matches how the
reference boards come — one fabbed board per device.

**Single integrated TX board.** Smaller and cheaper in volume, but it concentrates the full
dissipation and a high-gain chain on one board, with the most failure-prone outcome and the hardest
service. Defer it the same way as the integrated RX bank.

## Shielding — required, and stricter than RX in places

The same two-layer model as `rx-frontend.md` (outer enclosure for ingress and connectors; internal
compartments for stage isolation) applies, with three TX-specific drivers that make it mandatory
rather than optional:

- **PA stability.** The SDR's onboard PA + driver + final is ~50 dB of gain on one band; if the output couples
  back to the input it *will* oscillate. Each stage goes in its own shielded compartment with input
  and output physically separated — this is the classic reason power-amp stages are
  compartmentalized.
- **Harmonic containment.** The low-pass filter only helps if energy is forced *through* it. A
  shielded compartment around the LPF stops harmonics coupling around it and keeps the emitted
  spectrum clean (the design target is −43 dBc or better).
- **TX ↔ RX isolation.** Transmit energy leaking into the receive front end desenses the receiver,
  which matters for crossband full-duplex. Keep TX and RX in separate, well-shielded enclosures.

## Shielding and thermal are one mechanical design

Unlike RX, the TX enclosure is also the **heatsink**: each final bolts to the box wall / heatsink
that is simultaneously its compartment shield and its thermal path (`power-thermal.md`). Packaging,
shielding and cooling are therefore designed together — a finned die-cast wall behind each PA serves
all three. The temperature-controlled fan on the LDMOS bands and the per-band power ceilings live in
this same enclosure design.

Construction details (unbroken ground plane, dense via fences, soldered or can-style fences or
machined box ridges, lids, and fences-first NanoVNA tuning) are identical to `rx-frontend.md`.

## Recommendation

Build the first node as **per-band PA modules + a shared band-select board**, each PA in
its own shielded, heatsinked box, with TX kept in its own enclosure away from RX. Treat a single
integrated TX board as a later optimization once the design is proven and multiple nodes justify it.
