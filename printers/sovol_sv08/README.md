# Sovol SV08 with an enclosure — printer pack (stub)

> **Stub.** Open-frame CoreXY printer on mainline-style Klipper with Moonraker, run inside an enclosure kit [U: OEM or third-party]. Close to the generic Klipper + Moonraker stub. Every unknown is marked [U]; nothing here is verified on a unit unless tagged [V]. Not usable for heating until designed (doc 12 §6). Structure and contracts: [doc 12](../../docs/12_porting_guide.md). Paths to core docs are relative to this folder (`../../docs/`).

## 1. Summary

| Item | Value |
|---|---|
| Status | **stub** |
| Role | node-heat [U] |
| Link | `usb_serial` (backup `moonraker`) |
| Chamber cap | From the enclosure's weakest material [U] |
| Heater supply | [U] |
| Door | `node_reed` [U] |
| Port adapter | Kit part replacement or panel flange [U] |
| Profile | [`profile.yaml`](profile.yaml) |

## 2. Printer facts

| Fact | Value |
|---|---|
| Enclosure | Kit [U: which] |
| Exhaust opening | [U]: depends on the kit |
| Own chamber heater | No [V] |
| Firmware | Klipper + Moonraker on the printer's own host [V] |
| USB host ports | On the host board [U] |
| Main supply | [U] |
| Multi-material unit | [U] |

## 3. Role and integration

[U] until the pack is designed. Likely shape: node-heat like the AD5X pack: a kit-part port module, heater from the supply or a brick, USB to the host.

## 4. Enclosure modifications

[U]. Start from the kind of port adapter in §1 (doc 12 §5) and the generic kit mods (design decisions T3).

## 5. Heater supply

None until designed. A stub never runs a node heater.

## 6. Printer link

USB-serial to the printer's Klipper host (doc 04 §5) [U: free port].

## 7. Host integration

[U]

## 8. Printed parts

[U]. Look for existing models first (doc 08 policy).

## 9. Install variant

Stock Klipper with Moonraker (doc 12 §3.3).

## 10. Pack decisions

None yet.

## 11. Open items

| # | Item | Impact | Plan |
|---|---|---|---|
| SV-1 | Which enclosure kit, and its panel material | Port adapter, cap | Owner of the printer |
| SV-2 | Supply rating and headroom | Heater supply | Read the label |

## 12. Commissioning results

None.
