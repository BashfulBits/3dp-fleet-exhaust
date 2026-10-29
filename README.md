# Fleet Exhaust & Chamber Thermal System

An open-source exhaust, filtration and chamber-heating system for a small fleet of enclosed 3D printers: a shared duct and fan, one node per printer, and a hub.

**Reference build:** 2 × Flashforge AD5X (official enclosure kit, modified) + 1 × Flashforge Creator 5 Pro, all on **zmod over stock Flashforge firmware** (the Flashforge app, FlashPrint and the iOS app keep working). Other printers are added through printer packs ([doc 12](docs/12_porting_guide.md)).

**What it does:**
- holds every printing chamber at a slight negative pressure and vents fumes outdoors through one shared, filtered duct
- heats the AD5X chambers for ABS, ASA, PA-CF and PC
- runs cooling exhaust for PLA, PETG and TPU
- after every print, runs a slicer-controlled purge with a slow cooldown ramp and signals "safe to open"
- sends private push alerts only when someone's action could help

None of it can shut down Klipper or kill a print.

**Status:** Phase 1 design baseline, decisions D1–D64 as of 2026-10-28; open items are in [doc 01 §7](docs/01_system_architecture.md#7-risk-and-open-item-register). Phase 2 (implementation) follows the staged bring-up in [doc 01 §8](docs/01_system_architecture.md#8-phase-2-bring-up-stages-d41), starting with reverse-engineering the Vivosun G4 fan interface (risk R1).

---

## Architecture at a glance

```mermaid
flowchart TB
  HUB["Hub"]
  HUB --> SVC["G4 fan · smart plugs<br>Pushover · dashboard<br>MQTT → HA (optional)"]
  HUB ===|"CAT6"| N1["Node 1<br>node-heat"]
  N1 ===|"CAT6"| N2["Node 2<br>node-heat"]
  N2 ===|"CAT6"| N3["Node 3<br>node-lite"]
  N1 <-->|"USB + heater"| A1["AD5X #1"]
  N2 <-->|"USB + heater"| A2["AD5X #2"]
  N3 <-->|"USB"| C5["C5 Pro"]
```

The CAT6 daisy chain carries 24 V logic power and CAN, with ESP-NOW radio as a backup. Every printer talks to its node over USB-serial (Moonraker on Wi-Fi is the backup), and each AD5X also feeds its heater with 24 V. The smart plugs cut a printer's mains and meter its power.

- **Printer nodes:** one universal carrier PCB with a plug-in ESP32-S3 board. Each node runs its own heater and damper control and its own safety. The AD5X nodes add a 150 W PTC heater.
- **zmod plugin `fleet_exhaust`:** a klippy extra makes the chamber a normal Klipper sensor, so `M191` waits for chamber temperature. It also provides `FLEET_EXHAUST` / `FLEET_PURGE` for Orca G-code.
- **Hub:** the same carrier PCB. It holds a constant trunk vacuum, queues purges one at a time, monitors filter loading and room air, cuts a printer's mains on a corroborated fire or a verified heater runaway, and handles notifications.

## Documents

| Doc | Content |
|---|---|
| [01 System architecture](docs/01_system_architecture.md) | Goals, priorities, topology, key decisions and why, **risk & open-item register** |
| [02 Hardware specification](docs/02_hardware_specification.md) | Carrier board roles, heater, sensors, dampers, hub, fan (enclosure mods are in the printer packs) |
| [03 Bill of materials](docs/03_bill_of_materials.md) | Line-item BOM and cost for the reference build (≈ $632), and how it scales |
| [04 Wiring and pinouts](docs/04_wiring_and_pinouts.md) | Cables, RJ45 bus pinout, GPIO map, printer USB link, grounding |
| [05 Safety and failsafes](docs/05_safety_and_failsafes.md) | Safety layers, fault-response matrix, fire logic, commissioning tests |
| [06 Pneumatics and ducting](docs/06_pneumatics_and_ducting.md) | Layout, control loops, purge, material policy, heat budget, filter |
| [07 Firmware and comms](docs/07_firmware_and_comms.md) | Firmware, zmod plugin, G-code API, protocols, notifications, dashboard |
| [08 Printable parts](docs/08_printable_catalog.md) | Existing models to verify and custom parts to design |
| [09 Sensor bus and I/O controller](docs/09_sensor_bus_and_io_controller.md) | Up to 8 sensor headers, one I²C segment each, per-header power and interrupts, programming every MCU through its ROM bootloader, and the supported sensors (base set and bought-board catalogue) |
| [12 Porting guide](docs/12_porting_guide.md) | Adding other printers: printer packs, link tiers, profile schema, mechanical interface, safety contract, licensing |
| [13 Build your own](docs/13_build_your_own.md) | Quick start: does it fit, plan your fleet, staged build, reading order |
| [Design decisions](docs/design_decisions.md) | Decision record D1–D64, the cost-reduction pass and the trade studies behind them |

## Repository layout (Phase 2)

```text
├── LICENSES/                # Licence texts (REUSE); source files carry SPDX headers
├── docs/                    # Design documents (this baseline)
├── printers/                # Printer packs (doc 12): _template, flashforge_ad5x, flashforge_c5pro
├── firmware/
│   ├── fleet_node/          # One ESP-IDF project: node-heat, node-lite and hub roles
│   └── components/fleet_common/ # CAN + USB protocols, control loops, sensor drivers
├── slicer_profiles/orca/    # Filament and printer G-code snippets
└── hardware/
    ├── pcb/carrier/         # KiCad project, JLCPCB BOM/CPL, gerbers
    ├── pcb/sensor/          # Universal sensor board: S, P and E variants (D42)
    ├── cad/                 # CadQuery source + STEP + STL per custom part
    └── cables/              # Cable drawings and harness lists
```

The Klipper side (klippy extra, macros, installers) is published from its own small repo, `fleet-exhaust-klipper`, so Moonraker's update manager doesn't clone this whole repo onto each printer (D27).

## Using this for your own fleet

Keep your own build (which printers, where the duct runs, which plugs and supplies, your BOM and commissioning records) in a repository of your own, and add this one to it as a git submodule. Fleet-specific needs that this framework can't express yet are best raised here as a config option or a new printer pack, so they don't turn into a private fork.

## Licence

By area (details in [doc 12 §8](docs/12_porting_guide.md#8-licensing-and-contributions)):

| Area | Licence |
|---|---|
| Firmware, klippy extra, scripts | MIT ([LICENSE](LICENSE)) |
| Hardware designs (PCBs) | CERN-OHL-P-2.0 |
| Docs, printed-part CAD | CC BY 4.0 |

Licence texts are in [`LICENSES/`](LICENSES/) (the CERN-OHL-P-2.0 text arrives with the first hardware files) and files are labelled following the [REUSE](https://reuse.software/) spec ([`REUSE.toml`](REUSE.toml) covers the docs and printer packs; source files added in Phase 2 carry their own SPDX headers). Contributions: see [CONTRIBUTING.md](CONTRIBUTING.md).
