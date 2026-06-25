# RX front-end packaging

How to physically build the receive front end — the band-select, the four preselectors, and the
bypassable LNA. The electrical design is in `filters.md` and `rx-chain.svg`; this is the
board-and-packaging decision.

## Three options

**A — Loose modules + small switch/LNA boards.** Keep each preselector as the finished, shielded
module it already is (the DCI cavity, the Temwell helicals, the FreeWave 902), add a small ADRF5040
SP4T board and a small bypassable-LNA board, and join everything with short coax. The filters arrive
pre-tuned and pre-shielded, so the hard isolation work is done at the factory. Costs more connectors,
jumpers, loss and space, but there is almost nothing to design or tune in the filter domain.

**B — Carrier board (recommended for node one).** One small motherboard carries the SP4T(s) and the
LNA and provides mounting plus short edge-launch/coax to each purchased filter module. One control
line, one tidy assembly — but the filters still live in their own shielded cases, so the
compartment-and-tuning burden is avoided. This captures most of the integration tidiness without the
filter-shielding work.

**C — Fully integrated etched bank.** Switch + four filter sections + LNA on one PCB. Cleanest signal
path, lowest loss, cheapest in volume — but you own all the shielding and tuning, and a layout error
means a board respin. Justified only when building several nodes.

## Recommendation

For a single, unattended, hard-to-service node: **do not etch your own filter bank.** Use the finished
filter modules and integrate only the switch and LNA — option B. Revisit option C only if you ever
build several nodes and want to shrink cost and size.

Two fixed points regardless of option:

- The **2 m DCI is a cavity** — its high-Q skirts cannot be matched by anything mounted on a flat
  board, and 2 m is the band most likely to be hit hard by commercial/broadcast energy at a high
  site. Keep it external and bring its output onto the board.
- The **wideband discone RX** is a deliberately unfiltered monitoring path and stays off the
  preselector board entirely.

## Shielding

Any custom board here (the switch/LNA board, or the carrier) wants two nested shielding layers:

- **Outer enclosure (whole board).** A die-cast or tinplate box that stops ingress from everything
  else on the structure and carries the connectors. By itself it does little for filter rejection.
- **Internal compartments (per section).** Walls that put the LNA input and output — and each filter
  port — in their own shielded pockets, so signal goes *through* a filter, not *around* it, and the
  LNA cannot feed back on itself and oscillate.

With finished filter modules (options A/B) the per-filter compartmentalization is already handled by
the modules' own cases, so the custom board's remaining shielding job is mainly the **LNA (input vs
output separation)** and the **switch**, inside the outer box. Only the fully-integrated option C
makes you build the full compartment grid yourself.

How the compartments are built (option C, or any on-board walling):

- A solid, unbroken ground plane, with a **dense via fence** (vias every ~2–3 mm) in a straight line
  under every shield wall, and a continuous top-side ground pad over it for the wall to solder to. A
  sparse via fence is a slot antenna.
- Physical walls: soldered tinplate/brass fences for a one-off; board-level shield frames/cans
  (Laird, Würth, Harwin) for repeatability; or a die-cast box machined with internal ridges that
  press onto the ground pads. Every compartment needs a **lid**, not just walls.
- Keep each filter's input and output on **opposite sides of its wall**; the only path between them
  should be through the resonators.
- **Tune with the walls in place** — fences sit close to the resonators and shift tuning, so
  fences-first, then NanoVNA.

The mental model is a metal box with a grid of soldered-in rooms, each room tied to an unbroken
ground plane by a dense via fence, each room lidded — exactly how commercial switched preselector
banks are built.
