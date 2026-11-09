# Vendor Model — fan pack

> Copy this folder to `fans/<vendor>_<model>/` (or a class name) and fill it in. Structure, profile schema and qualification: [doc 14](../../docs/14_fan_packs.md). Requirements FN-1…FN-10: [doc 02 §6.1](../../docs/02_hardware_specification.md#61-requirements). Worked examples: [high-pressure 120 mm PWM](../jentech_jc12038b24uc2/README.md) and [6″ mains EC](../fantech_6in_ec/README.md).

## 1. Summary

| Item | Value |
|---|---|
| Status | stub, designed or qualified (doc 14 §5) |
| Family | `pwm_4wire`, `ec_0_10v`, `ec_usbc` or `ec_trrs` |
| Duct | 4″ or 6″ |
| Connection | Jack or pins on B2 (or fan A for a second fan), adapter, supply |
| Fleet sizes | From FN-2 and the airflow budget (doc 06 §4.5) |
| Profile | [`fan.yaml`](fan.yaml) |

## 2. Maker data

Each requirement against the maker's data, tagged [V] with the source, [E] or [U].

| Requirement | Data |
|---|---|
| FN-2 operating point | |
| FN-3 shut-off | |
| FN-4 duct | |
| FN-5 control | |
| FN-6 no signal | |
| FN-7 feedback | |
| FN-8 power | |
| FN-9 noise | |
| Isolation (mains fans) | |

## 3. Connection and cable

Pins or jack, the cable or adapter pin by pin, jumper settings (SWAP, DC), the printed duct adapter.

## 4. Open items

| # | Item | Impact | Closes at | Plan |
|---|---|---|---|---|

## 5. Qualification results

FQ-1…FQ-8 (doc 14 §4), each with its date, unit date code, jumper settings, the build that measured it and the numbers.
