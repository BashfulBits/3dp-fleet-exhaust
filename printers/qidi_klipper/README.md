# Qidi enclosed Klipper printers — printer pack (stub)

> **Stub.** Qidi's enclosed printers ship with Klipper and Moonraker. Some models heat their own chamber (node-lite); others don't (node-heat candidates). One stub for the family until a model is designed [U per model]. Every unknown is marked [U]; nothing here is verified on a unit unless tagged [V]. Not usable for heating until designed (doc 12 §6). Structure and contracts: [doc 12](../../docs/12_porting_guide.md). Paths to core docs are relative to this folder (`../../docs/`).

## 1. Summary

| Item | Value |
|---|---|
| Status | **stub** |
| Role | node-lite on models with a chamber heater; node-heat [U] on the rest |
| Link | `usb_serial` [U] (backup `moonraker`) |
| Chamber cap | The vendor's rating, or the weakest material [U per model] |
| Heater supply | [U] |
| Door | [U] |
| Port adapter | Shroud over the exhaust fan [U] |
| Profile | [`profile.yaml`](profile.yaml) |

## 2. Printer facts

| Fact | Value |
|---|---|
| Enclosure | Built in [V] |
| Exhaust opening | Exhaust fan with filter on several models [U] |
| Own chamber heater | On some models [U per model: rating] |
| Firmware | Klipper with Moonraker and Fluidd, stock [V] |
| USB host ports | On the host board [U] |
| Main supply | [U] |
| Multi-material unit | [U] |

## 3. Role and integration

[U] until the pack is designed. Likely shape: per model: node-lite like the C5 Pro pack where the printer heats, node-heat like the AD5X pack where it doesn't.

## 4. Enclosure modifications

[U]. Start from the kind of port adapter in §1 (doc 12 §5) and the generic kit mods (design decisions T3).

## 5. Heater supply

None until designed. A stub never runs a node heater.

## 6. Printer link

[U]

## 7. Host integration

[U]

## 8. Printed parts

[U]. Look for existing models first (doc 08 policy).

## 9. Install variant

Stock Klipper with Moonraker (doc 12 §3.3) [U: extras path].

## 10. Pack decisions

None yet.

## 11. Open items

| # | Item | Impact | Closes at | Plan |
|---|---|---|---|---|
| QD-1 | Split into one pack per model once one is designed | Pack structure | Design | Doc 12 §7 |
| QD-2 | Exhaust fan position and circulation macros per model | Port adapter, `klipper.modes` | PQ-1 | Inspect `printer.cfg` |

## 12. Qualification results

None.
