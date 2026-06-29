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

## Device selection — alternatives considered

The CGH40010F is the **device of record** for the driver on all four bands. This was reconfirmed against
a full survey of alternatives (2026-06) after the part was found flagged **NRND** ("not recommended for
new design"). Conclusion: **stay with the CGH40010F** — it is still the best fit despite NRND, because it
is the one part that covers every band, the design around it is already complete, and it is still buyable
from distributor stock for a one-off node.

| Candidate | Why not (for this build) |
|-----------|--------------------------|
| **CG2H40010F** (MACOM Gen-2 successor) | The natural drop-in — same 440166 flange, 28 V, now DC–8 GHz — but available only on pre-order at **MOQ 120**, unusable for a ~6-device node. The intended replacement *if* a rebuild is ever done at volume. |
| **Qorvo QPA2237** (GaN MMIC, 0.03–2.5 GHz, 10 W, 50 Ω matched) | Active and internally matched (no tuning), but **~$134 ea** (dearer than the CGH40010F), 10 W is overkill for a ≤4 W driver, and its 2.5 GHz ceiling drops the future microwave slices. |
| **MACOM CMPA0060002F1** (internally-matched MMIC, 20 MHz–6 GHz, 4.8 W) | Technically ideal — one part, all bands, 50 Ω in/out, zero match — but **>$300 ea**, too expensive per board. |
| **Qorvo QPD1010** (bare GaN, DC–4 GHz, 10 W, 3×3 QFN) | Viable (Mouser / sample stock, Modelithics model), but needs a QFN relayout, has a 4 GHz ceiling, and offers no decisive edge over the CGH40010F already designed around. |
| **Qorvo TGF2936** | Discontinued. |
| **Band-optimized LDMOS** (e.g. NXP AFT05MS-class for VHF, positive bias) | Cheaper per part and simpler (positive) bias, but band-specific → multiple device types, multiple reference designs, and mixed bias schemes = more total complexity for a 4-band node. The coherent version, if ever pursued: an LDMOS driver on the LDMOS-final bands (2 m/222) + a GaN driver on the GaN-final bands (70 cm/902). |

**Why the CGH40010F wins anyway:** one device covers 2 m–902 (and to 6 GHz for future slices); the
schematic, BOM, and bench bring-up are already done; it is family-matched to the GaN finals and shares
the CGH40010F-AMP reference ecosystem (ADS/MWO large-signal model for the match); and at ~$90 in stock it
is *cheaper* than the QPA2237 it was being weighed against.

**Accepted trade-offs:** (1) **NRND** — buy from remaining distributor stock (DigiKey/Mouser still list
it) and treat it as a final buy; fine for a one-off node, not a part to base a production run on. If stock
dries up, the fallbacks are the CG2H40010F (Gen-2, MOQ 120) or the LDMOS-low/GaN-high split above.
(2) **Per-band match tuning** — there is no internal 50 Ω match, so each band is tuned on the bench per
*Bench bring-up*. Both costs were judged acceptable for a personal, hand-built node.

> The **finals face the same NRND situation**: the CGH40120F (70 cm/902) has a Gen-2 successor
> (CG2H40120F). That is a separate decision (see open items), but the same "buy-from-stock for a one-off,
> Gen-2 for a rebuild" logic applies.

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

## Build BOM — four boards (2 m through 902)

One VHF/UHF node = **four driver boards** (2 m / 222 / 70 cm / 902), one CGH40010 each. The parts
split into three buckets, and that split is the whole point:

- **Buy now (fixed value known):** device, DC blocks, bias/stability network, connectors, PCB.
- **Buy after the bench (value measured, not guessed):** the four match parts per board (L1/C1/L2/C2).
- **Bench tuning kit (one shared set, not shipped on any board):** trimmers, magnet wire, assortments.

You can order everything in the first and third buckets today; the second bucket is a short list you
place once tuning converges (see *Bench bring-up*).

### A. Active device + thermal (the long-lead items)

| Ref | Part | Per board | Node total | Notes |
|-----|------|----------:|-----------:|-------|
| U1 | Wolfspeed/MACOM **CGH40010F** (flange) | 1 | **6** (4 + 2 spare) | GaN, long lead; buy spares |
| HW | flange screws + thermal interface (thermal pad or grease) | 1 set | 4 sets | torque to spec; never run bare |

### B. DC blocks — fixed, value per band (ATC 100B porcelain)

| Band | Cin / Cout | Qty |
|------|-----------|----:|
| 2 m | 1 nF | 2 |
| 222 | 680 pF | 2 |
| 70 cm | 100 pF | 2 |
| 902 | 47 pF | 2 |

8 total. Near-shorts at band, not tuning elements. C0G 0603 substitutes fine — confirmed PNs in **G**.

### C. Bias & stability network — common to all four boards

| Ref | Value | Per board | Node total | Type |
|-----|-------|----------:|-----------:|------|
| Rg | 22 Ω | 1 | 4 | thick-film 0603 fine — Yageo RC0603FR-0722RL (stock 10/22/33 Ω to pick on bench) |
| Lg, Ld | high-Z choke | 2 | 8 | wideband conical (Coilcraft conical / Piconics) **or** per-band chip choke, high SRF |
| Cg1 | 0.1 µF | 1 | 4 | X7R 0603 50 V |
| Cg2 | 0.01 µF | 1 | 4 | X7R 0603 50 V |
| Cd1 | 100 pF | 1 | 4 | C0G 0603 50 V |
| Cd2 | 0.1 µF | 1 | 4 | X7R 0603 50 V |
| Cd3 | 10 µF + 100 µF | 2 | 8 | aluminum electrolytic 50 V — ECA-1HM100/101 (not tantalum; drain video-bandwidth bank) |

### D. Connectors + PCB

| Ref | Part | Per board | Node total | Notes |
|-----|------|----------:|-----------:|-------|
| J1, J2 | Amphenol **901-10513-1** edge-launch SMA | 2 | 8 | repo-standard internal SMA |
| PCB | Rogers **RO4350B 20 mil**, custom fab | 1 | 4 (panel 5–6 w/ spares) | JLCPCB / PCBWay |

### E. Match parts — found on the bench, installed FIXED (order post-tune)

Do **not** pre-order these at a value — the bench gives the value. Stuff/tune with the kit in (F),
read the result, then order the fixed parts below at the measured values.

| Position | Per board | Node total | What you order after tuning |
|----------|----------:|-----------:|------------------------------|
| C1, C2 | 2 | 8 | fixed high-Q C0G / ATC 100B porcelain at measured value |
| L1, L2 | 2 | 8 | air-wound (no PN) **or** nearest Coilcraft `0603DC` (`0402DC` at 902) at measured value |

On 2 m / 222 the **input** inductance runs large (it's resonating the gate cap) and often past the
chip series' top — leave the input coil **air-wound** on the low bands. Where a match cap lands above a
trimmer's top range (the 2 m input can need ~80 pF), park a fixed C0G in parallel and trim the
remainder — the same fixed-plus-trim trick used on the 2 m BPF C4.

### F. Bench tuning kit — ONE shared set, reused on all four boards (never shipped)

| Item | Qty | Use |
|------|----:|-----|
| Sapphire Giga-Trim trimmers `5602` (1–30 pF), `5202` (0.8–10 pF), `5502` (1–20 pF), `27273` (0.6–4.5 pF) | 2–3 each | temporarily set C1/C2 to find the value (same PNs as the BPF build) |
| Magnet wire, 18 AWG + 22 AWG enameled | 1 spool each | wind + squeeze-tune the L1/L2 air coils |
| Fixed C0G / porcelain cap assortment, ~1–100 pF | 1 kit | drop-in fixed caps after tuning; parallel-fix for the large 2 m input cap |
| Coilcraft `0603DC` + `0402DC` inductor assortment (adjacent E24 values) | 1 set | only if going chip instead of air-wound |

The kit is a set of **instruments**: trimmers, wire, and assortments tune all four boards (and future
spares). They cost a fraction of the per-board parts and are reused indefinitely.

### G. DigiKey order list — verified part numbers (as ordered)

Every line below is decoded to its real value and confirmed against a live DigiKey part number
(verified 2026-06). The Six Flags PR descriptions read "VARIOUS VALUES"; the value lives in the DigiKey
number, so this table is the key for reconciling a PR against the design. Commodity caps/resistors
substitute freely at the same value/voltage — don't chase a specific MLCC PN if it goes NRND. Confirm
live stock at order time.

**Active device**

| Ref | Value | Mfr PN | DigiKey # | Node qty | Notes |
|-----|-------|--------|-----------|---------:|-------|
| U1 | CGH40010F GaN HEMT (flange) | CGH40010F | 1465-CGH40010F-ND | 4 (+1–2 spare) | NRND — final buy from stock; see *Device selection* |

**DC blocks (Cin/Cout) — C0G 0603 50 V, value per band** (C0G 0603 is the sourced equivalent of the §B ATC 100B porcelain — fine for a near-short at band)

| Band | Value | Mfr PN | DigiKey # | Node qty |
|------|-------|--------|-----------|---------:|
| 2 m | 1 nF | GRM1885C1H102JA01D | 490-1451-1-ND | 2 |
| 222 | 680 pF | GRM1885C1H681JA01D | 490-1447-1-ND | 2 |
| 70 cm | 100 pF | GRM1885C1H101JA01D | 490-1427-1-ND | 2 |
| 902 | 47 pF | GRM1885C1H470JA01D | 490-1419-1-ND | 2 |

**Bias & stability network (common, ×4 boards)**

| Ref | Value | Mfr PN | DigiKey # | Node qty | Notes |
|-----|-------|--------|-----------|---------:|-------|
| Rg | 22 Ω 0603 | Yageo RC0603FR-0722RL | 311-22.0HRCT-ND | 4 | thick-film fine (bias line, ~no current) |
| Cg1, Cd2 | 0.1 µF X7R 0603 50 V | Yageo CC0603KRX7R9BB104 | 311-1344-1-ND | 8 | |
| Cg2 | 0.01 µF X7R 0603 50 V | Yageo CC0603KRX7R9BB103 | 311-1085-1-ND | 4 | |
| Cd1 | 100 pF C0G 0603 50 V | GRM1885C1H101JA01D | 490-1427-1-ND | 4 | same PN as 70 cm DC block |
| Cd3a | 10 µF 50 V alum. electrolytic | Panasonic ECA-1HM100 | P5178-ND | 4 | not tantalum (derating + short-fail) |
| Cd3b | 100 µF 50 V alum. electrolytic | Panasonic ECA-1HM101 | P5182-ND | 4 | |
| Lg, Ld | wideband bias choke | Coilcraft 4310LC series | (value per bias-tee) | 8 | **not on PR1102611408 — order separately** |

**Connectors**

| Ref | Value | Mfr PN | DigiKey # | Node qty |
|-----|-------|--------|-----------|---------:|
| J1, J2 | edge-launch SMA jack | Amphenol 901-10513-1 | ARF2504-ND | 8 |

> **Consolidate the 100 pF.** `490-1427-1-ND` (100 pF) serves both the 70 cm DC blocks (2) and Cd1 (4)
> → order as **one line, qty 6**. On PR1102611408 it is split across line 25 (qty 2) and line 29
> (qty 4); combine them.

> **DigiKey number decoders** (so a dead PN never stalls a reorder):
> Murata 0603 C0G `GRM1885C1H` value series → DigiKey `490-14xx-1-ND`, E24-sequential
> (490-1413 = 27 pF, 1419 = 47 pF, 1427 = 100 pF, 1447 = 680 pF, 1451 = 1 nF).
> Yageo MLCC `CC0603[K=±10%]RX7R9BB(=50 V)[104 = 0.1 µF / 103 = 10 nF]`.
> Yageo resistor `RC0603FR-07<value>L` (`22R` = 22 Ω). Panasonic elec `ECA-1H(=50 V)M[100 = 10 µF / 101 = 100 µF]`.

### Order-now vs order-after

- **Order now:** A, B, C, D, F — the device, fixed support parts, boards, and the whole tuning kit.
- **Order after the bench:** E only — eight caps and (if chip) eight inductors, at measured values.

## Bench bring-up

Tuning one board, in the order you actually do it. The rule underneath every step: **bias before RF,
small-signal before large-signal, tune before you fix values.** "Tune" means temporary bench
instruments (trimmer + air coil); the shipped board is all-fixed — see the table at the end.

**Gear:** device bolted to a heatsink (thermal compound) *before any power*; two metered supplies — a
current-limited **+28 V** drain and a separate **adjustable −V** gate (0 to −10 V), each with a current
readout; a VNA with **output pads** (the amp has gain — protect the receiver port); a large-signal
chain of source → rated **attenuator → 50 Ω dummy load** → power meter / spectrum analyzer; and an LCR
meter to read final component values.

### Phase 0 — DC sanity (no RF, no drain)

Torque the flange, inspect for shorts. Gate to **−8 V** (past pinch-off). Drain **off**, current-limited
to ~0.5 A. Gate negative **first**, always.

### Phase 1 — set IDQ

1. Gate at −8 V (OFF). Bring up +28 V. Drain current should read **~0 mA** — if it jumps, stop and fix.
2. Walk the gate less-negative (−8 → −3 → −2.9 …); current starts to climb around **−2.7 to −3.0 V**
   (per-device — every part differs).
3. Set **IDQ ≈ 100–150 mA**, let it warm a couple minutes, re-trim (GaN drifts warm). **Record the VGG**
   that gives target IDQ — that is this device's number.
4. Rehearse shutdown once: **drain off first, then gate.**

### Phase 2 — small-signal match (find L1/C1/L2/C2)

Biased at IDQ, VNA-level drive (~−10 dBm), amp **standalone** (BPF not in line yet). Every match
position is made **temporarily adjustable**: fit a **sapphire trimmer** in the C1/C2 spots (a bench
instrument only) and an **air-wound coil** in the L1/L2 spots (tune by squeezing/spreading turns) — two
real knobs per L-match.

1. **Stability sweep first, wide.** No S-parameter > 1 out of band, no low-frequency gain spike. GaN
   oscillates low if the Rg / bias decoupling is off — fix that **before** tuning anything.
2. Tune the **output** (L2 + C2) for best S22 and a gain peak centered on band.
3. Tune the **input** (L1 + C1) for deepest S11 (target < −15 to −20 dB).
4. **Iterate input ↔ output** a few passes — they interact. Confirm S21 flat at ~18–20 dB across band.

### Phase 3 — large-signal: power + linearity

Drive for real into the attenuator/load, stepping up **gradually** from low.

1. Confirm it makes the budget output (e.g. 2.5 W at 902) from the SDR's actual +19 dBm drive, with
   gain holding before compression.
2. **Two-tone IMD** at operating output — the all-mode acceptance test. Aim IMD3 ≈ −30 dBc or better.
   Ugly = driven too hard or IDQ too low.
3. Watch drain current rise (normal Class AB), harmonics, and **device temperature** — no runaway.
4. Re-peak the match slightly if it shifted (it won't move much at ~30 % of rating — the upside of
   running backed off).

### Phase 4 — lock to fixed parts

1. **Read C1, C2** off the trimmers with the LCR meter → remove the trimmers → install **fixed high-Q
   C0G / porcelain** at the measured values. **Trimmers do not ship on this board.**
2. **L1, L2:** either immobilize the air coil as the final part, or read its inductance and swap to the
   nearest Coilcraft `0603DC` (`0402DC` at 902). The PN is chosen **now**, from the measured value.
3. **Re-sweep** to confirm nothing moved when you went fixed. That is the tuned, shippable board.

| Position | Phase 2 (bench, to find the value) | Phase 4 (shipped board) |
|----------|------------------------------------|--------------------------|
| C1, C2 | sapphire trimmer (instrument) | **fixed C0G / porcelain** at measured value |
| L1, L2 | air-wound, squeeze to tune | lock the coil, **or** nearest Coilcraft `0603DC` / `0402DC` |

This is the deliberate contrast with the BPF: there the sapphire trimmer **stays** (low level,
drift-tolerant, useful touch-up); here it's a bench tool only, because the driver output is a power /
efficiency node you want fixed in a set-and-forget node.

### Standing safety rules (every power cycle)

- **Gate −V before drain; drain off before gate.** No exceptions — depletion-mode GaN conducts at VGS=0.
- **Always on a heatsink**; set IDQ warm, not cold (it drifts up as it heats).
- **Pad the VNA output** — a biased amp will damage the receiver port otherwise.
- **Dummy load only** — never key into an open or short.
- **Check stability wideband**, not just in-band — GaN's favorite failure is a low-frequency oscillation.
- **Do 902 last** — shortest stubs, tightest layout, the match moves fastest; get your rhythm on 2 m.

## Sourcing

- **CGH40010F / CGH40010P** — DigiKey/Mouser still list it in stock, but it is **NRND** (not
  recommended for new design): buy from remaining stock as a final buy, and confirm live stock at order
  time (don't assume from a snippet). See *Device selection* for the lifecycle rationale and fallbacks.
- **ATC 100B porcelain DC blocks, RO4350B board** — DigiKey / a fab house (RO4350B is a JLCPCB/PCBWay
  material option); small-signal sections tolerate good FR-4, but the gain stage benefits from RO4350B.
- **Conical chokes** (Coilcraft BCR / Piconics) or printed λ/4 bias lines — DigiKey / Coilcraft.
- **Negative-bias control board** — see the open item above; this is the one piece not yet pinned.

## Open items

- **Negative keyed gate bias** on the control board for the GaN driver (all bands) + GaN finals —
  confirm the chosen board (or add a small negative-bias/keying board) alongside the positive rail for
  the LDMOS finals. *This is the gating open item for the driver stage.*
- **Per-band match values** — measured on the bench per *Bench bring-up* (not pre-orderable); fold the
  final L1/C1/L2/C2 numbers back into this doc and the schematic once a board is tuned.
- **Microwave drivers (13/9/5 cm)** — the CGH40010 covers DC–6 GHz, so the same driver part likely
  carries those slices if they are ever built; the finals are the open question there, not the driver.
- **Finals lifecycle (NRND).** The CGH40120F final (70 cm/902) is in the same NRND family as the
  driver; its Gen-2 successor is the CG2H40120F. Decide buy-from-stock (one-off) vs CG2H (rebuild)
  for the finals — same logic the driver settled on. See *Device selection*.
