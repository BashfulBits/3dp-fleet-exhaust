# Flashforge AD5X — printer pack

> Pack for the Flashforge AD5X with the official enclosure kit, on zmod over stock firmware. Structure and contracts: [doc 12](../../docs/12_porting_guide.md). Paths to core docs are relative to this folder (`../../docs/`).

## 1. Summary

| Item | Value |
|---|---|
| Role | node-heat (150 W PTC heater, recirculation blower on B1) |
| Link | `usb_serial` (bottom internal USB header), backup `moonraker` |
| Chamber cap | **60 °C** operating, 65 °C firmware hard limit (D26). Assumes a sealed, insulated chamber (gaskets or RTV on the seams) and every printed part in the chamber in ASA/ABS (HDT ≈ 95 °C, so ≥ 30 °C margin). Check the stock toolhead and cable-chain plastics at unboxing [U: O11] |
| Heater supply | Printer supply tap, reused or upgraded (O1), or an external DIN-4 brick (D22) |
| Door | Node reed switch (C6) |
| Port adapter | C1, high at the rear, replacing one kit printed part (D19) |
| Profile | [`profile.yaml`](profile.yaml) |

## 2. Enclosure modifications

| Mod | Specification |
|---|---|
| Kit printed parts | Printed in **ASA/ABS on the C5 Pro**. PETG-CF only for parts outside the chamber below ~60 °C |
| IFS/Bowden pass-through | Plate with 4 PTFE couplers and grommets replacing the open IFS window. Start from the MakerWorld "AD5X Bowden Window Remix" (licence and fit check, doc 08); on the panel facing the IFS, which sits beside the enclosure (O2) |
| Gaskets | EPDM D-profile on the door and lid; foam tape on panel seams |
| Insulation | Foil-faced foam or reflective bubble inside the rear, side and top panels; front door left clear |
| Exhaust port | **High at the rear**, in a printed port module (C1) that **replaces one of the kit's printed parts** (D19), so the polycarbonate isn't cut. ModulAir L bayonet to the branch damper (D34) |
| Recirculation duct | **Required (O3):** printed duct + 24 V 5015 blower that pulls chamber air through the AD5X's rear filter and returns it inside (doc 08 E7). Switched by node output B1 |
| Makeup inlet | Low on the front or side, a slotted grille with a light gravity flap, sized at commissioning so purge flow doesn't pull hard through the IFS pass-through. Mostly closed during heated prints, when the chamber is only −2 to −5 Pa |
| Heater module | Low at the rear (§3) |
| Door switch | Reed switch + magnet in printed mounts |
| Cable pass-through | Grommeted, in the C1 port module beside the node housing: heater feed from the supply tap (absent with an external brick), heater lead, sensor-head cable, NTC leads, USB lead, ΔP tube |
| Node mounting | **Printed housing attached to the branch damper** at the exhaust port (D19), outside the chamber, with a thermal break from the duct wall; cables go straight through the pass-through |

## 3. Heater supply (O1)

The core rules are in doc 02 §3.1 (sizing rule, tap, bed interlock, rail throttle). For the AD5X, at unboxing: read the supply label (watts, 24 V current), measure its footprint and mounting, and check whether the bed heater runs from it (24 V bed) or from mains. Then apply the sizing rule: reuse, upgrade inside the base (Mean Well LRS-class with the same connections and cooling), or fit the DIN-4 jack and use an external brick.

## 4. USB link

Generic link rules: doc 04 §5.

| Item | Specification |
|---|---|
| Printer end | **Bottom internal USB header** on the AD5X mainboard (hub downstream port 3) [V: zmod #207]. A user measured **5.05 V on VBUS**, GND continuity and idle D+/D− on the bottom-left header [V: #207]. Footprint type and pin order are unreported (R7): confirm with a meter at unboxing before wiring. |
| Header | Solder a matching pin header into the footprint (pins into holes, not wires onto pads). If it turns out to be SMD pads only, use a pre-made adapter cable rather than flying wires. |
| Enumeration | The node appears as a USB CDC-ACM device; the AD5X kernel has `cdc_acm` [V: zmod #348]. The plugin opens it as `/dev/serial/by-id/usb-fleet_exhaust_node_<serial>-if00` [U: check that `/dev/serial/by-id` exists on the AD5X; otherwise match the VID/PID through `/sys`]. |
| Route | Inside the AD5X base → out through a grommet in the rear/base panel → node enclosure on the rear exterior. |
| Fallbacks | (1) `TVOC` 4-pin UART (3.3 V; probably `ttyS3`, **untested** [#207]). (2) UDP over Wi-Fi with the same plugin code. **Never use `J29`, `DEBUG` or `J13`** (§4.1). |

### 4.1 Ports reviewed

Checked against zmod Discussion #207 on 2026-10-25. **[207-T]** = tested and reported in the thread; **[207-S]** = stated or traced there, not tested.

| Port | What it is | Verdict |
|---|---|---|
| **External USB-A port** (the USB-stick port) | Host USB; `cdc_acm` enumerates USB serial devices [V: zmod #348] | **Preferred if you don't print from USB sticks.** No modification and fully reversible. The cable leaves through the sealed grommet. |
| **Bottom internal USB footprint** (hub downstream port 3) [207-S] | Host USB on the onboard hub. A user measured **5.05 V VBUS**, GND continuity and idle D+/D− [207-T] | **Preferred if you want the front port free.** VBUS is live, so the cable must not connect it to the node (data + GND only). Needs a header in the footprint [U]. |
| Top internal USB footprint [207-S] | SoC USB controller, possibly OTG/device role | Avoid: role unknown and untested. |
| `TVOC` 4-pin UART | Possibly a CPU UART (`ttyS3`), untested [207-S] | **Fallback only.** It's the AD5M Pro air-sensor port, so Flashforge's `firmwareExe` may probe or claim it. |
| `J29` 6-pin `GND GND RX TX 24V 24V` | ~24 V on the 24 V pins at idle [207-T]; suspected RS-232 levels [207-S] | **Do not use.** |
| `DEBUG` (3- or 4-pin) | Boot console at 115200, possibly 1.8 V [207-S] | Do not use. |
| `J13` 3-pin GND/TX/RX | No activity at boot [207-T] | Do not use. |

Some listings mention v1 boards with a USB-C port; if yours has one, identify its role before using it [U].

## 5. Klipper integration

| Item | AD5X |
|---|---|
| Node link | USB-serial (primary) + Moonraker WebSocket (backup) |
| `[temperature_sensor chamber]` | From `fleet_node` |
| `M141`/`M191` | Plugin |
| `FLEET_EXHAUST` also calls | — |
| `FLEET_PURGE` end of print | — |
| Door state | Node reed switch |

## 6. Printed parts

| # | Part | Candidate source | Material | Verify |
|---|---|---|---|---|
| E1 | AD5X enclosure kit printed parts | Flashforge kit files (the official kit's printable parts) | ASA | Print as supplied; no PETG |
| E2 | IFS/Bowden pass-through plate | MakerWorld "AD5X Bowden Window Remix" (search by name; URL to record) | ASA + TPU grommets | Seals the window fully with 4 PTFE couplers; goes on the panel facing the IFS (beside the enclosure, O2). Modify to add grommets if needed |
| E4 | AD5X exhaust flange (base for remix) | Printables [#1176762](https://www.printables.com/model/1176762), [#462199](https://www.printables.com/model/462199) | ASA | Reference for the panel mounting of custom C1; its duct end is the ModulAir L bayonet (D34) |
| E7 | **AD5X recirculation duct + 24 V 5015 blower (required, O3)** | Printables [#1473971](https://www.printables.com/model/1473971) (Dzung Vu; 4-wire 24 V 5015, pulls chamber air through the AD5X's rear filter and returns it; clears the stock mainboard fan); alternative [#1472826](https://www.printables.com/model/1472826) (Joel Whitington, a remix of Jwidess's AD5M Pro duct) | ASA (both authors suggest PETG, which creeps at 55 °C, §1) | Fit with the kit installed and with the node's cables routed; blower on node output B1 instead of the printer's fan port; licence CC BY-NC-SA (link, don't redistribute the STLs) |

| # | Part | Material | Key requirements |
|---|---|---|---|
| C1 | **AD5X exhaust port module** | ASA | **Replaces one of the kit's printed parts** at a high rear position (part chosen at kit assembly; D19), so the polycarbonate isn't cut; ModulAir L bayonet port to the E3 damper (D34); holds the C8 grommet beside the port; inside lip keeps the sensor head out of the direct jet |
| C6 | **Door reed + magnet mounts** | ASA | Glue-free clips onto the kit's door frame |

Core parts used here: E3 damper, C2 makeup inlet, C3 heater mount, C4 sensor-head cover, C5 node housing, C8 grommet (doc 08).

## 7. Install variant

From the `fleet-exhaust-klipper` repo (doc 07 §3.1). The pack also ships a Moonraker `[sensor fleet_exhaust]` block for the node readings and job history (doc 07 §6.3). zmod plugin (doc 07 §3.1).

## 8. Porting checklist results

To be filled in during Phase 2 commissioning (doc 12 §6 item 7, §7).
