# Prusa Core One / MK4 in an enclosure — printer pack (stub)

> **Stub.** Prusa printers on Buddy firmware (Marlin-derived), with a network API (PrusaLink). The Core One is enclosed; the MK4 runs in the Prusa enclosure or a third-party one. Every unknown is marked [U]; nothing here is verified on a unit unless tagged [V]. Not usable for heating until designed (doc 12 §6). Structure and contracts: [doc 12](../../docs/12_porting_guide.md). Paths to core docs are relative to this folder (`../../docs/`).

## 1. Summary

| Item | Value |
|---|---|
| Status | **stub** |
| Role | node-lite [U] |
| Link | `power_only` (Tier 0); `prusalink` (Tier 1) listed, not built; `marlin_serial` (Tier 2b) [U: whether the port accepts it] |
| Chamber cap | [U] |
| Heater supply | None until designed |
| Door | [U] |
| Port adapter | Shroud over the enclosure's fan or filter outlet, or a panel flange [U] |
| Profile | [`profile.yaml`](profile.yaml) |

## 2. Printer facts

| Fact | Value |
|---|---|
| Enclosure | Core One: built in [V]; MK4: Prusa enclosure or third-party [V] |
| Exhaust opening | [U]: depends on the enclosure and any filtration unit |
| Own chamber heater | No dedicated heater [U]; chamber temperature management with fans [U] |
| Firmware | Prusa Buddy (Marlin-derived) [V]; PrusaLink local API [V] |
| USB host ports | [U] |
| Main supply | [U] |
| Multi-material unit | MMU3 [U] |

## 3. Role and integration

[U] until the pack is designed. Likely shape: node-lite on Tier 0 first; Tier 1 once a `prusalink` link type exists.

## 4. Enclosure modifications

[U]. Start from the kind of port adapter in §1 (doc 12 §5) and the generic kit mods (design decisions T3).

## 5. Heater supply

None until designed. A stub never runs a node heater.

## 6. Printer link

Tier 0 from the metering plug. `prusalink` would add print state and job name; `marlin_serial` [U].

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
| PR-1 | Exhaust outlet on each enclosure option | Port adapter | PQ-1 | Inspect |
| PR-2 | Which PrusaLink fields give print state and material | Tier 1 link | Link type | Whoever builds the link type |
| PR-3 | Whether a node heater is ever wanted on this printer | Role | Design | Owner of the printer |

## 12. Qualification results

None.
