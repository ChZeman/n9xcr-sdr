# Deployment considerations

Generic engineering notes for deploying a node where the transceiver enclosure sits **at the
antennas** (mast-mounted, outdoors) rather than in an equipment room. Mounting the radio at the
antenna eliminates the long VHF/UHF coax run (worst at 902 MHz) — the RF path becomes inches, and
what runs back to the equipment room is data and power, not RF.

## Power

- AC mains at the enclosure is the simplest feed. A 50 W PA draws roughly 1 A on transmit.
- With mains (rather than PoE), PA size is **thermally** limited, not power-limited — the cooling
  budget sets the ceiling, not the supply.

## Thermal

- A sealed, sun-exposed outdoor enclosure with no airflow is the binding constraint. 50 W per band
  keeps this tractable; 100 W+ pushes into forced-air territory.
- PA devices heatsink to the enclosure wall, with a sun shield; target fanless if the dissipation
  budget allows. GaN efficiency at the top band helps (only ~30–40 W dissipated at 50 W out).
- Enclosure: sealed and weatherproof (NEMA 4X / IP66+), with a vent/desiccant plug; choose
  materials for the local environment (humidity, salt, UV).

## Lightning and grounding

- An outdoor mast is a strike path. Use coax arrestors on every antenna feed, a surge protector
  (SPD) on the AC feed, and single-point bonding of every enclosure to one ground tied to the
  structure's down-conductor.
- A **fiber** data path is galvanically immune to surge and isolates the mast-top electronics from
  the run back to the equipment room — the preferred data path for lightning reasons. The node
  simply joins the local network at the enclosure.

## Node-host placement

The node-host PC does **not** need to sit in the outdoor enclosure — it belongs somewhere
climate-controlled, serviceable, and UPS-backed, reaching the AntSDR over the local network (the
AntSDR is a network device; SDRangel connects over IIO/Ethernet).

What stays at the enclosure is anything hard-real-time, and that is all in hardware: the PA control
wrapper does T-R sequencing, bias keying, and SWR/thermal/overcurrent foldback autonomously in
microseconds, and PTT / band-select are keyed by the AntSDR's own GPIO. The host's role is
supervisory only (read telemetry, reduce drive or inhibit TX on a sustained alarm) and tolerates
network latency. A remote host needs a small local IO bridge at the enclosure to put the
control-board alarm lines and PTT on the local network. The GPSDO stays at the enclosure (it
disciplines the SDR locally).

Outdoor-enclosure essentials, then: SDR + PA/control-wrapper + GPSDO + the local IO bridge.
Everything else lives indoors.

## RF exposure

If people can be near the antennas while the station could key, provide a maintenance **TX
lockout** (disable transmit when someone is working near the antennas) and signage. The near-field
worker is the case to design for; follow the applicable RF-exposure limits for the power and
frequencies in use.
