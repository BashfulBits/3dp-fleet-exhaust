# Flashforge Creator 5 Pro — printer pack

> Pack for the Flashforge Creator 5 Pro on zmod (alpha) over stock firmware. Structure and contracts: [doc 12](../../docs/12_porting_guide.md). Paths to core docs are relative to this folder (`../../docs/`).

## 1. Summary

| Item | Value |
|---|---|
| Role | node-lite (the C5 heats its own chamber) |
| Link | `usb_serial` via the external USB port (D24, pending O10), backup `moonraker` |
| Chamber cap | **65 °C**, Flashforge's rating for the C5's own chamber heater. The node never heats, and its node-heat limits (doc 05 §3) don't apply |
| Door | Stock `topDoor`/`frontDoor` buttons over the link |
| Port adapter | C7 shroud over the side exhaust fan (O4) |
| Profile | [`profile.yaml`](profile.yaml) |

## 2. Enclosure modifications

No internal changes. A printed **shroud docks over its side exhaust fan** (the 7025 fan behind the right side cover, O4) and carries the branch damper, with the **node housing attached to the damper** (D19). The sensor-head cable enters the chamber through a grommet in the shroud, via the existing exhaust port. Its stock `AIR_CIRCULATION_EXTERNAL`/`INTERNAL` modes are selected by the plugin (doc 07).

## 3. Heater supply

None: no node heater on the C5.

## 4. USB link

Generic link rules: doc 04 §5.

Same protocol, same node end and the same **VBUS-less** lead construction as the AD5X; only the printer end differs.

| Item | Specification |
|---|---|
| Printer end, first choice | The C5's **external USB port** (on the right side per a hands-on review [U: confirm at unboxing, O10]). Nothing goes inside the printer. This is the thumb-drive port: either give it to the node (jobs arrive over the network anyway) or put a small USB 2.0 hub on it. |
| Printer end, fallback | **J34** (`U_disk`, 5-pin, `5V \| D− \| D+ \| G \| G`) inside the C5, through a CH334F-class hub board so the stock USB port keeps working. Use a **5-pin-to-4-pin crossover** pigtail (D+ and D− swap between J34 and the hub's `5V \| D+ \| D− \| GND`; ARG-03), the hub's VBUS to our lead left unconnected, and the printed hub bracket (doc 08). Only if the external port proves unusable. |
| Lead | USB-C plug → USB-A plug (or → hub), **D+, D−, GND and shield only, VBUS unconnected and insulated at both ends**, as doc 04 §5. |
| Enumeration | Needs `cdc_acm` in the C5's kernel [U: **O10**]. If missing, the C5 stays on Moonraker over Wi-Fi with no other change. |
| Ground | The node-lite has no heater, so no heater return shares this lead's ground (R11 doesn't apply). Shield connects at the C5 end only. |

## 5. Klipper integration

| Item | C5 Pro |
|---|---|
| Node link | **USB-serial (primary, D24)** + Moonraker WebSocket (backup); Moonraker only if O10 fails |
| `[temperature_sensor chamber]` | Not added (stock `chamber_heater` sensor exists). The node's exhaust-port air temperature registers as `[temperature_sensor fleet_exhaust_air]`; RH, VOC, ΔP and damper are in `printer.fleet_node` |
| `M141`/`M191` | Stock |
| `FLEET_EXHAUST` also calls | `AIR_CIRCULATION_INTERNAL` for heated modes, `AIR_CIRCULATION_EXTERNAL` for cooling (O4) |
| `FLEET_PURGE` end of print | `AIR_CIRCULATION_EXTERNAL`, then `AIR_CIRCULATION_STOP` when the node reports done |
| Door state | Stock `topDoor`/`frontDoor` buttons, read by the node through Moonraker |

## 6. Printed parts

| # | Part | Candidate source | Material | Verify |
|---|---|---|---|---|
| E8 | C5 Pro exhaust-port adaptor (base for C7) | Printables [#1802285](https://www.printables.com/model/1802285) (adaptor over the side exhaust fan with a side-cover gasket, ⌀80 mm hose fixture, blank fixture); MakerWorld [#2934469](https://makerworld.com/en/models/2934469) (80 mm duct held by the filament-holder pins) | ASA | Port dimensions (O4); remix to a ModulAir L bayonet port (D34) that carries the E3 damper and C5 housing; licence (#1802285 is CC BY-NC) |

| # | Part | Material | Key requirements |
|---|---|---|---|
| C7 | **C5 Pro side-port shroud** | ASA | Remix of E8: docks over the side exhaust fan (O4) without blocking stock airflow; ModulAir L bayonet port carrying the E3 damper and C5 housing (D34); grommet for the sensor-head cable; side-cover gasket; tool-free removal |

Core parts used here: E3 damper, C4 sensor-head cover, C5 node housing (doc 08).

## 7. Install variant

From the `fleet-exhaust-klipper` repo (doc 07 §3.1). The pack also ships a Moonraker `[sensor fleet_exhaust]` block for the node readings and job history (doc 07 §6.3). zmod plugin (doc 07 §3.1); zmod's C5 support is alpha, and the extras path is still to confirm (R5).

## 8. Porting checklist results

To be filled in during Phase 2 commissioning (doc 12 §6 item 7, §7).
