# Bambu Lab X1 / P1 series — printer pack (stub)

> **Stub.** Enclosed CoreXY printers on closed firmware. No Klipper, so no host plugin; print state comes from plug power until a network link type exists. Every unknown is marked [U]; nothing here is verified on a unit unless tagged [V]. Not usable for heating until designed (doc 12 §6). Structure and contracts: [doc 12](../../docs/12_porting_guide.md). Paths to core docs are relative to this folder (`../../docs/`).

## 1. Summary

| Item | Value |
|---|---|
| Status | **stub** |
| Role | node-lite |
| Link | `power_only` (Tier 0); `bambu_mqtt` (Tier 1, LAN mode) is listed but not built |
| Chamber cap | [U]; no node heater on closed firmware |
| Heater supply | None |
| Door | `node_reed` [U] or `none` |
| Port adapter | Shroud over the rear exhaust; ModulAir publishes an X1/P1 adapter (MA-L-BL-X1-P1, [MakerWorld #2523279](https://makerworld.com/models/2523279)) |
| Profile | [`profile.yaml`](profile.yaml) |

## 2. Printer facts

| Fact | Value |
|---|---|
| Enclosure | Built in (X1C, X1E, P1S) [V]; P1P is open-frame [V] |
| Exhaust opening | Rear chamber exhaust fan [V]; filter fitted on some models [U] |
| Own chamber heater | No active heater on X1C/P1S [V]; X1E [U] |
| Firmware | Bambu closed firmware; LAN-mode MQTT reports print state [V]; access rules for third-party clients change between firmware releases [U] |
| USB host ports | No user-accessible USB host for a serial link [U] |
| Main supply | [U] |
| Multi-material unit | AMS; material per slot over MQTT [U] |

## 3. Role and integration

[U] until the pack is designed. Likely shape: node-lite on Tier 0, with the ModulAir adapter carrying the damper and node housing.

## 4. Enclosure modifications

[U]. Start from the kind of port adapter in §1 (doc 12 §5) and the generic kit mods (design decisions T3).

## 5. Heater supply

None: no node heater on a Tier 0 or closed-firmware printer.

## 6. Printer link

Tier 0 from the metering plug (doc 07 §6.5). A `bambu_mqtt` link (Tier 1) would add job name and material; not built.

## 7. Host integration

[U]

## 8. Printed parts

[U]. Look for existing models first (doc 08 policy).

Candidate: ModulAir MA-L-BL-X1-P1 ([MakerWorld #2523279](https://makerworld.com/models/2523279)) as the port adapter.

## 9. Install variant

None (doc 12 §3.3).

## 10. Pack decisions

None yet.

## 11. Open items

| # | Item | Impact | Plan |
|---|---|---|---|
| BX-1 | Does the ModulAir adapter seal over the rear fan without starving it? | Port adapter | Print and test |
| BX-2 | `bambu_mqtt` link: which fields are available in LAN mode on current firmware | Tier 1 link | Whoever builds the link type |
| BX-3 | Does the chamber fan's speed need to follow fleet modes? | Bleed control | Commissioning |

## 12. Commissioning results

None.
