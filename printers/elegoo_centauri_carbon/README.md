# Elegoo Centauri Carbon — printer pack (stub)

> **Stub.** Enclosed CoreXY printer on Elegoo's own firmware [U: Klipper base and access]. Tier 0 until its network protocol is supported. Every unknown is marked [U]; nothing here is verified on a unit unless tagged [V]. Not usable for heating until designed (doc 12 §6). Structure and contracts: [doc 12](../../docs/12_porting_guide.md). Paths to core docs are relative to this folder (`../../docs/`).

## 1. Summary

| Item | Value |
|---|---|
| Status | **stub** |
| Role | node-lite [U] |
| Link | `power_only` (Tier 0) [U: a network API] |
| Chamber cap | [U] |
| Heater supply | None |
| Door | [U] |
| Port adapter | Shroud or panel [U] |
| Profile | [`profile.yaml`](profile.yaml) |

## 2. Printer facts

| Fact | Value |
|---|---|
| Enclosure | Built in [V] |
| Exhaust opening | [U] |
| Own chamber heater | No [U] |
| Firmware | Elegoo firmware [U: architecture, root, API] |
| USB host ports | [U] |
| Main supply | [U] |
| Multi-material unit | [U] |

## 3. Role and integration

[U] until the pack is designed. Likely shape: node-lite on Tier 0.

## 4. Enclosure modifications

[U]. Start from the kind of port adapter in §1 (doc 12 §5) and the generic kit mods (design decisions T3).

## 5. Heater supply

None until designed. A stub never runs a node heater.

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

| # | Item | Impact | Closes at | Plan |
|---|---|---|---|---|
| CC-1 | Network protocol and print-state fields | Tier 1 link | Link type | Whoever builds the link type |
| CC-2 | Exhaust outlet | Port adapter | PQ-1 | Inspect |

## 12. Qualification results

None.
