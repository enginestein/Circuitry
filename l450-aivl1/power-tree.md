# Power Tree — Lenovo ThinkPad L450 (AIVL1 NM-A351)

Goal: identify every power rail (name, source, voltage, consumers) by measurement, building a power tree diagram.

Equipment: multimeter (DC V, continuity, resistance) + charger.

---

## Rail inventory (fill as measured)

| Rail | Nominal V | Source IC / path | Measured V | Continuity links (consumers) | Notes |
| ---- | --------- | ---------------- | ---------- | ---------------------------- | ----- |
| GND | 0 | ground plane | — | everywhere (continuity to plane) | black-probe reference |
| VIN | ~20V | DC jack → BQ24780S DCIN | | | unplugged: continuity through input fuse |
| VBAT | 11.1V (3S) | battery connector ↔ charger | | | cell cluster |
| +3VS5 | 3.3V | standby buck (EC rail) | | | must be live with charger alone, no power-on |
| +5VS5 | 5.0V | standby buck (USB sleep) | | | only some USB ports / dock on S5 |
| +3V (RTC) | 3.3V | coin cell / VBAT diode | | | RTC + chipset suspend |
| +5V (S0) | 5.0V | main 5V rail | | | only after power-on |
| VCore | ~0.8–1.0V | NCP81101 (CPU VR) | | | inductor output, CPU core |
| VCCSA | ~0.9V | system agent rail | | | |
| VCCIO / VCC3 .3PCH | 1.05/3.3 | PCH rails | | | |
| DDR rail | 1.35V | RAM VR | | | near DIMM slots |

---

## Procedure

### 1. Safety / unpowered check
- Before applying power, DMM on resistance: measure VIN-to-GND. Expect capacitor charging (climbs) not a locked short. Step 1 gives you a no-short sanity check.

### 2. Find the DC-in path (unpowered)
- Probe the barrel-jack center pin → continuity to one side of the input fuse (near jack, often `F…`).
- Trace fuse → BQ24780S `DCIN` pin (the "80 24780 I3" chip already identified).
- Record any series protection MOSFETs between jack and charger IC.

### 3. Standby rails — power (charger only, no battery)
- Plug charger. Black probe on board ground.
- Verify VIN ≈ 20V at the fuse and at BQ24780S DCIN.
- Find **always-on rails that live without pressing power**: sweep the little inductors; whichever show ~3.3V and ~5.0V unpressed = +3VS5 / +5VS5.
- Also catch +3V RTC: probe coin-cell pins / its diode (powered from same standby 3.3V).

### 4. Main rails (press power button on motherboard edge if possible)
- +5V S0, VCore (NCP81101 inductor), VCCSA, DDR: every one of these appears only after the rail sequence triggers.
- Record voltage at each inductor's output pad (the rail's source) and a nearby bulk cap.

### 5. Consolidate loads per rail
- For each rail, continuity from its inductor output to the bypass caps / IC power pins in its neighborhood → that's the "consumers" column + the skeleton of the power tree.

---

## Watch-outs
- Never short GND to a live rail while measuring.
- Inductors read ~0Ω across; it's the *voltage* at their output pad that identifies the rail.
- If a rail refuses to come up when powered, log it — that's a fault-finding lead, not a dead end.

## Status

- [ ] VIN path traced (jack → fuse → BQ24780S)
- [ ] Standby rails identified (+3VS5, +5VS5, +3V RTC)
- [ ] Main rails identified (VCore, VCCSA, DDR, +5V S0)
- [ ] Loads mapped per rail
- [ ] Power tree diagram produced