# Bambu Lab H2D — printer pack (stub)

> **Stub.** Enclosed printer on closed firmware that heats its own chamber [U: rating]. Node-lite on Tier 0 until a network link type exists. Every unknown is marked [U]; nothing here is verified on a unit unless tagged [V]. Not usable for heating until designed (doc 12 §6). Structure and contracts: [doc 12](../../docs/12_porting_guide.md). Paths to core docs are relative to this folder (`../../docs/`).

## 1. Summary

| Item | Value |
|---|---|
| Status | **stub** |
| Role | node-lite |
| Link | `power_only` (Tier 0); `bambu_mqtt` not built |
| Chamber cap | The vendor's rating for its own chamber heating [U] |
| Heater supply | None |
| Door | [U] |
| Port adapter | Shroud over its exhaust [U] |
| Profile | [`profile.yaml`](profile.yaml) |

## 2. Printer facts

| Fact | Value |
|---|---|
| Enclosure | Built in [V] |
| Exhaust opening | [U] |
| Own chamber heater | Yes, active chamber heating [U: rating] |
| Firmware | Bambu closed firmware; LAN-mode MQTT [U] |
| USB host ports | [U] |
| Main supply | [U] |
| Multi-material unit | AMS family [U] |

## 3. Role and integration

[U] until the pack is designed. Likely shape: node-lite like the C5 Pro pack, but on Tier 0: fire signals from the sensor head only, no circulation-mode control.

## 4. Enclosure modifications

[U]. Start from the kind of port adapter in §1 (doc 12 §5) and the generic kit mods (design decisions T3).

## 5. Heater supply

None.

## 6. Printer link

Tier 0 from the metering plug.

## 7. Host integration

[U]

## 8. Printed parts

[U]. Look for existing models first (doc 08 policy).

## 9. Install variant

None.

## 10. Pack decisions

None yet.

## 11. Open items

| # | Item | Impact | Plan |
|---|---|---|---|
| H2D-1 | Exhaust outlet position and size | Port adapter | Inspect |
| H2D-2 | Chamber heating rating | `chamber.cap_c` | Vendor spec |
| H2D-3 | How fire signal F-a works without chamber data over a link (D39) | Fire detection | Phase 2 firmware: sensor-head SHT40 path |

## 12. Commissioning results

None.
