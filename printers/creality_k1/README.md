# Creality K1 series — printer pack (stub)

> **Stub.** Enclosed CoreXY printers (K1, K1C, K1 Max) on Creality's Klipper-based firmware. Moonraker and a host plugin need root access through community helper scripts [U]. Every unknown is marked [U]; nothing here is verified on a unit unless tagged [V]. Not usable for heating until designed (doc 12 §6). Structure and contracts: [doc 12](../../docs/12_porting_guide.md). Paths to core docs are relative to this folder (`../../docs/`).

## 1. Summary

| Item | Value |
|---|---|
| Status | **stub** |
| Role | node-heat [U] |
| Link | `moonraker` after rooting [U]; `usb_serial` [U: cdc_acm and a free port] |
| Chamber cap | [U]: from stock and printed plastics |
| Heater supply | [U] |
| Door | [U] |
| Port adapter | Shroud over the rear exhaust or filter [U] |
| Profile | [`profile.yaml`](profile.yaml) |

## 2. Printer facts

| Fact | Value |
|---|---|
| Enclosure | Built in [V] |
| Exhaust opening | Rear fan and filter [U: exhausts or recirculates, by model] |
| Own chamber heater | No [U] |
| Firmware | Creality Klipper fork [V]; root and Moonraker via community scripts [U] |
| USB host ports | [U] |
| Main supply | [U] |
| Multi-material unit | [U] |

## 3. Role and integration

[U] until the pack is designed. Likely shape: node-heat with a shroud over the rear outlet, if the stock plastics and supply allow.

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

Stock Klipper with Moonraker variant after rooting [U] (doc 12 §3.3).

## 10. Pack decisions

None yet.

## 11. Open items

| # | Item | Impact | Plan |
|---|---|---|---|
| K1-1 | Rear outlet: exhaust or recirculation, by model | Port adapter | Inspect |
| K1-2 | Stock chamber plastics | `chamber.cap_c` | Check markings |
| K1-3 | Supply rating and bed supply | Heater supply | Read the label |

## 12. Commissioning results

None.
