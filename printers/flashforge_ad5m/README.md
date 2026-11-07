# Flashforge Adventurer 5M / 5M Pro — printer pack (stub)

> **Stub.** The Adventurer 5M family is a close relative of the AD5X: same vendor firmware family and the same firmware mod. The Pro has a built-in enclosure; the 5M needs a third-party or OEM enclosure [U]. Every unknown is marked [U]; nothing here is verified on a unit unless tagged [V]. Not usable for heating until designed (doc 12 §6). Structure and contracts: [doc 12](../../docs/12_porting_guide.md). Paths to core docs are relative to this folder (`../../docs/`).

## 1. Summary

| Item | Value |
|---|---|
| Status | **stub** |
| Role | node-heat [U], following the AD5X pack |
| Link | `usb_serial` (backup `moonraker`) through zmod [U] |
| Chamber cap | [U]: from the enclosure's weakest material |
| Heater supply | [U] (supply rating unknown) |
| Door | [U] |
| Port adapter | Kit part replacement or panel flange [U] |
| Profile | [`profile.yaml`](profile.yaml) |

## 2. Printer facts

| Fact | Value |
|---|---|
| Enclosure | 5M Pro: built in [V: vendor]; 5M: none stock [V], enclosure kits exist [U] |
| Exhaust opening | [U]; the 5M Pro has a rear filter/fan module [U: exhausts or recirculates] |
| Own chamber heater | No [U] |
| Firmware | Flashforge stock Klipper-based firmware; zmod supports this family [V: zmod] |
| USB host ports | External USB-A [U: role]; the `TVOC` air-sensor port is used by the 5M Pro's sensor (see the AD5X pack's ports review) |
| `cdc_acm` | [U] |
| Main supply | [U] |
| Multi-material unit | None stock [U] |

## 3. Role and integration

[U] until the pack is designed. Likely shape: as the AD5X pack: node-heat, heater from the printer's supply, USB link to the zmod plugin.

## 4. Enclosure modifications

[U]. Start from the kind of port adapter in §1 (doc 12 §5) and the generic kit mods (design decisions T3).

## 5. Heater supply

None until designed. A stub never runs a node heater.

## 6. Printer link

[U]. Review the ports as the AD5X pack does (§6.1 there).

## 7. Host integration

[U]

## 8. Printed parts

[U]. Look for existing models first (doc 08 policy).

## 9. Install variant

zmod variant (doc 12 §3.3) [U: extras path].

## 10. Pack decisions

None yet.

## 11. Open items

| # | Item | Impact | Plan |
|---|---|---|---|
| AD5M-1 | Does the 5M Pro's rear fan exhaust or recirculate? | Port adapter kind; recirculation duct | Inspect |
| AD5M-2 | Supply rating and whether the bed runs from it | Heater supply | Read the label |
| AD5M-3 | USB host port and `cdc_acm` | Tier 2 or Moonraker only | Plug a node in |

## 12. Commissioning results

None.
