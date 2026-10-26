# Vendor Model — printer pack

> Copy this folder to `printers/<vendor>_<model>/` and fill it in. Structure and contracts: [doc 12](../../docs/12_porting_guide.md). Worked examples: [AD5X](../flashforge_ad5x/README.md) (node-heat, USB link) and [C5 Pro](../flashforge_c5pro/README.md) (node-lite, native chamber heater).

## 1. Summary

| Item | Value |
|---|---|
| Role | node-heat or node-lite |
| Link | Tier and link type (doc 12 §3.2), plus the backup |
| Chamber cap | °C, and the material that sets it |
| Heater supply | None, printer tap, upgraded internal supply or external brick |
| Door | Node reed, link objects or none |
| Port adapter | Part ID and where it sits |
| Profile | [`profile.yaml`](profile.yaml) |

## 2. Enclosure modifications

Port adapter to the standard interface (doc 12 §5), pass-throughs, seals, insulation, makeup inlet, door switch, node mounting.

## 3. Heater supply

Node-heat only: the sizing check (doc 02 §3.1) with the printer's supply rating, or the external brick used. Otherwise "None".

## 4. Printer link

Tier 2: the USB connection point (external port preferred; headers, never wires on pads), `cdc_acm` check, route. Tier 1: host and API. Tier 0: the metering plug.

## 5. Host integration

What the profile's `klipper:` keys produce on this printer, or the equivalent for other firmware.

## 6. Printed parts

| # | Part | Candidate source | Material | Verify |
|---|---|---|---|---|

| # | Part | Material | Key requirements |
|---|---|---|---|

Request catalogue IDs in doc 08 so cross-references stay stable.

## 7. Install variant

zmod, stock Klipper with Moonraker `update_manager`, or none (doc 12 §3.3).

## 8. Porting checklist results

Measured `power.idle_w` / `power.printing_w`, and the commissioning tests from doc 12 §6 item 7 with dates and results.
