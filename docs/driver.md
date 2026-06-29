# Driver stage

Between the slice-wide cleanup bandpass and the per-band final sits one gain stage: the **driver**.
It takes the exciter level handed off by the band-select (+12…+19 dBm, set in software per band) and
lifts it to the few watts the final needs to reach 50 W. One device type covers every band; the board
is built per band because the matching network retunes. See [`tx-chain.md`](tx-chain.md) for where this
sits in the chain and [`boards.md`](boards.md) for the buy-vs-fab split. Schematic + per-board BOM:
[`../diagrams/driver-schematic.svg`](../diagrams/driver-schematic.svg).

## Device

**Wolfspeed/MACOM CGH40010** — 10 W GaN HEMT (13 W typ PSAT), DC–6 GHz, 28 V, ~18–20 dB gain at
VHF/UHF, rated Class AB linear. One part covers 2 m through 902 (and on up past 1.3 GHz if those
slices are ever built), and it is family-matched to the **CGH40120F** final on the GaN bands. Two
packages, same die:

- **CGH40010F** — screw-down flange. Easiest to heatsink; preferred at the PA.
- **CGH40010P** — solder-down pill. Lower-profile, fine where reflow/thermal is handled on board.

The 8 W **CGH60008D** fits electrically but ships as bare die (not hand-solderable), so it is not used.

At the worst case — 902, driven for an 80 W final — the driver puts out ~4 W, about **30 % of its
10 W rating**, so it runs deep in its linear region for clean IMD. At the 50 W baseline it is loafing
(≤2.5 W out). This headroom is deliberate: a clean driver keeps the whole chain's IMD set by the
final, not by the small stage ahead of it.

## Why a discrete driver at all

The SDR's onboard PA (PGA-102+) gives the pre-driver gain, and the final gives the output gain, but
there is a ~15–20 dB hole between them that one broadband Class AB stage fills cleanly. Putting it in
its own stage (rather than leaning harder on the onboard PA or the final) keeps every stage backed off
and linear — the all-mode requirement. The driver is also the natural place to set per-band drive: its
input/output match is retuned per band, so each path is flat where it needs to be.

## Topology (reference-amp derived)

The board is an adaptation of the **Wolfspeed/MACOM CGH40010F-AMP** reference amplifier (published
schematic, BOM, and Rogers RO4350B layout) — a single-ended common-source Class AB amp:

```
RF IN → Cin (DC block) → input match (L1/C1) → GATE
                                                 │
                          U1 CGH40010 GaN HEMT  ─┤  (source to ground)
                                                 │
        drain ── output match (L2/C2) → Cout (DC block) → RF OUT
```

- **Input/output match** — lumped L/C networks (with short microstrip) transforming the device's low
  gate/drain impedance to 50 Ω. These are the per-band parts; everything else is common.
- **Drain bias tee** — RF choke `Ld` feeding +28 V into the drain node, with a bypass bank
  (`Cd1–Cd3`: an RF cap, a mid cap, and a bulk cap) spanning RF down through the modulation/“video”
  bandwidth so the supply looks like a short across the envelope. Linearity depends on this bank.
- **Gate bias tee** — series stability resistor `Rg` then choke `Lg` feeding the **negative** gate
  bias, with bypass `Cg1/Cg2`.

“Reference-derived” means the matching, stability, and bias networks come from a known-good vendor
design and are **retuned** for the amateur band, not synthesized from scratch — the same standard the
finals are held to in [`boards.md`](boards.md).

## Bias and keying

GaN HEMTs are **depletion-mode**: at VGS = 0 the channel is open and the device conducts hard. That
makes bias setpoint and sequencing a safety matter, not a tuning preference.

| Parameter | Value | Note |
|-----------|-------|------|
| VDD (drain) | +28 V | keyed (TX only) |
| IDQ (quiescent) | **~100–150 mA** | leaner than the 200 mA reference, to cut idle heat at the PA |
| VGS at IDQ | ≈ −2.7…−3.0 V | **set per device** — every part differs; trim while watching drain current |
| Pinch-off | ≈ −3.0 V (typ) | device essentially off |
| RX / cutoff | **≤ −8 V** | gate pulled well past pinch-off so the driver is cold in receive |

**Sequencing (hard rule):**

1. **Power-up:** apply VGG (negative, at or below pinch-off so U1 is OFF) **first**, then VDD +28 V.
   With the gate already negative, the drain can come up safely.
2. **Set IDQ:** raise VGG toward the operate point while monitoring drain current; stop at ~100–150 mA.
3. **Power-down:** remove **VDD first**, then VGG.
4. **Never** apply VDD with the gate at 0 V or floating — the device would draw destructive current.

Because the gate bias is what turns the device on, it doubles as the **keying** point: in receive the
gate sits at ≤ −8 V (cold, drawing nothing); on key-up it is switched to the operate point **after** the
T-R relays have cold-switched. This is the same gate-bias keying and sequencing the control wrapper
already does for the finals — see [`tx-chain.md`](tx-chain.md) “Control / protection wrapper.” The
driver is keyed in lockstep with the final on its band.

> **The negative gate supply and its keying live on the control board, not on the driver board.** The
> driver board carries only the bias-injection point, the choke, `Rg`, and the bypass caps. **Open item:**
> confirm the chosen amplifier-control board generates and keys a **negative** gate rail — the driver is
> GaN on *every* band (and the 70 cm / 902 finals are GaN too), so negative keyed bias is needed on all
> driver builds, whereas the common LDMOS-oriented control boards default to a *positive* gate rail for
> the MRF101AN finals. The station needs both polarities: positive-keyed for the LDMOS finals,
> negative-keyed for the GaN driver (all bands) and the GaN finals. See `bom.md` control-wrapper note.

## Per-band build

One CGH40010 per band → **4 driver boards** (2 m / 222 / 70 cm / 902). The bias/stability/bypass
network is identical; only the input/output match and the DC-block values change with frequency. Match
values come from retuning the reference / the Wolfspeed large-signal model (ADS / Microwave Office) and
are confirmed on a VNA — they are not asserted here as fixed numbers.

| Band | Band-select port | Final it drives | Driver gain (typ) | DC-block start | Match notes |
|------|------------------|-----------------|-------------------|----------------|-------------|
| 2 m   | RF8 (105–158) | MRF101AN | ≈20 dB | ~1 nF   | easiest; largest lumped elements |
| 222   | RF5 (158–237) | MRF101AN | ≈19 dB | ~680 pF | — |
| 70 cm | RF4 (355–533) | CGH40120F | ≈18 dB | ~100 pF | — |
| 902   | RF3 (800–1200) | CGH40120F | ≈16 dB | ~33–47 pF | touchiest — tight layout, shortest stubs, edge of lumped matching |

DC-block values are starting points (near-short at the band); the actual input/output match elements
are set by the retune. As frequency rises the elements shrink and layout parasitics dominate — 902 is
the fiddliest, same as it is for that band's BPF and final.

## Bill of materials (one board)

| Ref | Function | Value / setpoint | Type / vendor | Qty |
|-----|----------|------------------|---------------|----:|
| U1 | GaN HEMT | 28 V, IDQ ~100–150 mA | Wolfspeed/MACOM **CGH40010F** (flange) or **CGH40010P** (pill) | 1 |
| Cin, Cout | DC blocks | near-short at band | ATC 100B porcelain (value per band) | 2 |
| L1, C1 | input match | per band (retune) | C0G chip cap / Coilcraft chip or air-wound L | 2 |
| L2, C2 | output match | per band (retune) | C0G chip cap / Coilcraft chip or air-wound L | 2 |
| Rg | gate stability | ~10–47 Ω | thin-film chip 0402/0603 | 1 |
| Lg | gate bias choke | high-Z across band | wideband conical (Coilcraft BCR) or λ/4 line | 1 |
| Ld | drain bias choke | high-Z across band | wideband conical (Coilcraft BCR) or λ/4 line | 1 |
| Cg1, Cg2 | gate bias bypass | 0.1 µF + 0.01 µF | X7R chip + bulk to GND | 2 |
| Cd1–Cd3 | drain bias bank | 100 pF / 0.1 µF / 10–100 µF | C0G + X7R + tantalum/elec (video-bandwidth) | 3+ |
| J1, J2 | connectors | SMA | Amphenol 901-10513-1 edge-launch | 2 |
| PCB | substrate | — | Rogers **RO4350B 20 mil** | 1 |
| HW | thermal | — | F-flange screw-down to heatsink + thermal interface | 1 |

For all 4 bands: 4× CGH40010F, plus the common support set ×4 and the per-band match/DC-block parts.
Buy **+1–2 spare CGH40010** (GaN finals/drivers are the long-lead, can't-improvise parts).

## Sourcing

- **CGH40010F / CGH40010P** — DigiKey-stocked (Wolfspeed/MACOM); confirm Active status and live stock
  at order time (do not assume from a search snippet).
- **ATC 100B porcelain DC blocks, RO4350B board** — DigiKey / a fab house (RO4350B is a JLCPCB/PCBWay
  material option); small-signal sections tolerate good FR-4, but the gain stage benefits from RO4350B.
- **Conical chokes** (Coilcraft BCR / Piconics) or printed λ/4 bias lines — DigiKey / Coilcraft.
- **Negative-bias control board** — see the open item above; this is the one piece not yet pinned.

## Open items

- **Negative keyed gate bias** on the control board for the GaN driver (all bands) + GaN finals —
  confirm the chosen board (or add a small negative-bias/keying board) alongside the positive rail for
  the LDMOS finals. *This is the gating open item for the driver stage.*
- **Per-band match values** — extract from the Wolfspeed LSM / reference retune and confirm on a VNA;
  fold the final numbers back into this doc and the schematic once measured.
- **Microwave drivers (13/9/5 cm)** — the CGH40010 covers DC–6 GHz, so the same driver part likely
  carries those slices if they are ever built; the finals are the open question there, not the driver.
