# 24 V 120 × 38 mm 4-wire PWM, high pressure (Jentech JC12038B24UC2 class) — fan pack

> Structure and the profile schema: [doc 14](../../docs/14_fan_packs.md). Requirements FN-1…FN-10: [doc 02 §6.1](../../docs/02_hardware_specification.md#61-requirements). Paths to core docs are relative to this folder.

## 1. Summary

| Item | Value |
|---|---|
| Status | **designed** (doc 14 §5) |
| Family | `pwm_4wire` |
| Duct | 4″ |
| Connection | B2 XH 6 pins 1–4; supply from the fan tap (pin 2) |
| Fleet sizes | 1–15; a second one on fan A from ≈ 9 printers (FN-10) |
| Profile | [`fan.yaml`](fan.yaml) |

## 2. Maker data

| Requirement | Data |
|---|---|
| FN-2 operating point | ≈ 380 Pa at 50 CFM [V: maker datasheet and P–Q curve]: meets 250 Pa and the 330 Pa long-run budget with margin (≈ 55–85 CFM delivered, D88 [E]) |
| FN-3 shut-off | 1.97 inH₂O (≈ 490 Pa) [V] |
| FN-4 duct | Printed 120 mm → 4″ or 6″ adapter |
| FN-5 control | 4-wire PWM, 21–28 kHz [V: Intel 4-wire specification] |
| FN-6 no signal | Full speed with PWM open, by the 4-wire standard [V] |
| FN-7 feedback | Tach, open collector, 2 pulses/rev [V] |
| FN-8a power | 24 V, 1.63 A typical, 1.80 A maximum [V]; starting current [U] |
| FN-9 noise | Runs well below full speed in normal use, so it is quieter than the budget class there [E] |
| Other | 270 CFM free air [V]; sourcing [U] |

## 3. Connection and cable

Pins 1–4 of B2 (GND, +24 V from the fan tap, tach, PWM) through a 4-pin fan lead to the XH 6 plug. Jumpers: SWAP and DC not used.

## 4. Open items

| # | Item | Impact | Closes at | Plan |
|---|---|---|---|---|
| JC-1 | Starting and running current on samples (D87) | Must stay inside the tap eFuse's ≈ 2.7 A at start and ≤ ≈ 2.2 A running | FQ-4 | Twenty cold starts; meter the running current |
| JC-2 | Sourcing: a distributor that sells single units | BOM | Design | Find one before the example fleet's BOM is final |

## 5. Qualification results

None yet (doc 14 §4).
