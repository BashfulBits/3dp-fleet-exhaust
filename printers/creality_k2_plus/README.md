# Creality K2 Plus — printer pack (stub)

> **Stub.** Enclosed CoreXY printer on Creality's Klipper-based firmware that heats its own chamber [U: rating]. Every unknown is marked [U]; nothing here is verified on a unit unless tagged [V]. Not usable for heating until designed (doc 12 §6). Structure and contracts: [doc 12](../../docs/12_porting_guide.md). Paths to core docs are relative to this folder (`../../docs/`).

## 1. Summary

| Item | Value |
|---|---|
| Status | **stub** |
| Role | node-lite |
| Link | `moonraker` [U: access] |
| Chamber cap | The vendor's rating [U] |
| Heater supply | None |
| Door | [U] |
| Port adapter | Shroud [U] |
| Profile | [`profile.yaml`](profile.yaml) |

## 2. Printer facts

| Fact | Value |
|---|---|
| Enclosure | Built in [V] |
| Exhaust opening | [U] |
| Own chamber heater | Yes [U: rating] |
| Firmware | Creality Klipper fork [V]; Moonraker access [U] |
| USB host ports | [U] |
| Main supply | [U] |
| Multi-material unit | CFS [U: material status] |

## 3. Role and integration

[U] until the pack is designed. Likely shape: node-lite like the C5 Pro pack: the printer heats, the node adds exhaust, purge and fire detection.

## 4. Enclosure modifications

[U]. Start from the kind of port adapter in §1 (doc 12 §5) and the generic kit mods (design decisions T3).

## 5. Heater supply

None.

## 6. Printer link

[U]

## 7. Host integration

[U]

## 8. Printed parts

[U]. Look for existing models first (doc 08 policy).

## 9. Install variant

[U]

## 10. Pack decisions

None yet.

## 11. Open items

| # | Item | Impact | Closes at | Plan |
|---|---|---|---|---|
| K2-1 | Exhaust outlet and circulation macros | Port adapter, `klipper.modes` | PQ-1 | Inspect `printer.cfg` |
| K2-2 | Chamber heater rating and Klipper object names | `chamber.cap_c`, fire signals (D39) | Design (rating), PQ-9 (names) | Vendor spec, `printer.cfg` |

## 12. Qualification results

None.
