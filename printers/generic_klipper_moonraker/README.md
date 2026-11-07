# Generic Klipper + Moonraker — printer pack (stub)

> **Stub.** Starting point for any enclosed printer that runs Klipper with Moonraker and gives root access (mainline Klipper, a vendor fork opened up by community scripts, or a self-built machine). Every unknown is marked [U]; nothing here is verified on a unit unless tagged [V]. Not usable for heating until designed (doc 12 §6). Structure and contracts: [doc 12](../../docs/12_porting_guide.md). Paths to core docs are relative to this folder (`../../docs/`).

## 1. Summary

| Item | Value |
|---|---|
| Status | **stub** |
| Role | node-heat or node-lite, per printer |
| Link | `usb_serial` (Tier 2), backup `moonraker`; `moonraker` alone if the host has no `cdc_acm` |
| Chamber cap | From the weakest material in the chamber; ≤ 60 °C with a node heater |
| Heater supply | Per printer (doc 02 §3.1) |
| Door | `link` if the printer has door objects, else `node_reed` or `none` |
| Port adapter | Any of the three kinds (doc 12 §5) |
| Profile | [`profile.yaml`](profile.yaml) |

## 2. Printer facts

| Fact | Value |
|---|---|
| Enclosure | Built in, kit or self-built [U per printer] |
| Firmware | Klipper with Moonraker; host on the printer's own SBC or a separate one |
| USB host ports | The Klipper host's USB ports; most SBCs have several, so no internal header is needed [U per host] |
| `cdc_acm` | Present in most SBC kernels [U per host] |
| Main supply | [U per printer] |
| Multi-material unit | [U per printer] |

## 3. Role and integration

[U] until the pack is designed. Likely shape: the node on a damper at the printer's exhaust port, USB to the Klipper host, Moonraker as backup.

## 4. Enclosure modifications

[U]. Start from the kind of port adapter in §1 (doc 12 §5) and the generic kit mods (design decisions T3).

## 5. Heater supply

Per printer: the sizing rule for the printer's own supply (doc 02 §3.1), or a brick on the printer's plug.

## 6. Printer link

USB-serial to the Klipper host (doc 04 §5), lead without VBUS. Moonraker WebSocket as backup.

## 7. Host integration

[U]

## 8. Printed parts

[U]. Look for existing models first (doc 08 policy).

## 9. Install variant

Stock Klipper with Moonraker (doc 12 §3.3): `install.sh` symlinks the extra into `klippy/extras`, `[update_manager fleet_exhaust]`, Moonraker `[notifier]`.

## 10. Pack decisions

None yet.

## 11. Open items

| # | Item | Impact | Plan |
|---|---|---|---|
| G-1 | Copy this stub for a specific model and fill in its facts | — | Doc 12 §7 |

## 12. Commissioning results

None.
