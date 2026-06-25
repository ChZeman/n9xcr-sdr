# Telemetry and protection

Two related things share the same sensors: the **protection** that keeps the PA alive (autonomous,
in hardware, at the amplifier) and the **monitoring** that surfaces what the station is doing
(forwarded to the UI). The protection loop runs whether or not anything is watching; the UI reads
the same measurements for display.

## SWR protection (already in the design)

Reflected-power / SWR protection is part of the amplifier control wrapper, not an add-on:

- A **dual directional detector** at each PA output continuously samples forward and reflected power.
- The **amplifier control board** folds back drive — and shuts the PA down on a hard fault — when
  reflected power (SWR) exceeds threshold.
- This loop is **local and autonomous**: it acts in hardware in microseconds and does not depend on
  the node host, the network, or the UI. An antenna or feedline fault is caught even if the link is
  down. (It matters more as output rises — see `power-thermal.md` — because there is less linear
  margin to give back.)

The convenient consequence: the detector is **already measuring forward and reflected power**, so
displaying output power and SWR in the UI costs no new sensing hardware — it taps a signal the
protection circuit already produces.

## What's available to display

| Quantity | Source | Cost |
|----------|--------|------|
| Forward RF power (W) | directional detector, forward port | free (already sensed) |
| Reflected power / SWR | directional detector, reflected port; SWR computed | free |
| PA heatsink temperature | thermal-foldback thermistor | free |
| TX/RX state, band, PTT | control-wrapper logic | free |
| Fault / foldback flags | control board (SWR, thermal, overcurrent) | free |
| Drive level | AntSDR software setting | free (already in software) |
| PA drain current | overcurrent-trip current sense | one value to surface |
| PA supply voltage (50 V / 28 V rail) | ADC divider | one cheap sensor |
| Enclosure / ambient temperature | second thermistor | one cheap sensor |
| GPSDO lock status | GPSDO lock output | one digital line |
| Node / link health | node host | software, non-RF |

The first block is effectively free because the protection circuit already senses it. The second
block is one inexpensive sensor each and turns the dashboard into a real health monitor — drain
current creeping up, or supply sag under load, are early warnings of trouble.

## Signal path to the UI

The control board's analog outputs (forward, reflected, temperature, current, voltage) land on the
small **local IO bridge** at the enclosure — the ESP32 / Pico-class board already in the design. It
digitizes them with its ADC and publishes them over the LAN (MQTT or plain TCP) to the node host,
which feeds the web UI alongside the SDR's own waterfall and audio. The hard-real-time protection
never goes through this path — the IO bridge carries read-only telemetry plus the non-time-critical
"software ALC" supervisory feedback.

```
PA control board  (analog: forward, reflected, temp, drain current, rail voltage)
   -> local IO bridge (ADC, at the enclosure)
   -> LAN  (MQTT / TCP)
   -> node host
   -> web UI tiles    (+ waterfall / audio from the SDR)
```

## Accuracy caveat

A diode directional detector gives a good **relative** power/SWR reading — plenty for monitoring and
for protection — but it is not a calibrated laboratory wattmeter, and its response varies with
frequency. For a trustworthy on-air wattage number, calibrate the detector against a known reference
once per band. Good for a dashboard and for "is the antenna okay," not for +/-2 % absolute.

## Suggested UI tiles

Per active band: **power out** (with the configured ceiling shown), **SWR**, **heatsink temp** (with
the foldback threshold marked), and a **state** line (band / TX-RX / PTT / fault). Station-wide:
**drive**, **rail voltage**, **drain current**, **enclosure temp**, **GPSDO lock**, **link health**.
Color the power and temperature tiles against their limits so headroom — and any active foldback — is
visible at a glance.
