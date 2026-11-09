# 24 V 120 × 38 mm 4-wire PWM, industrial (Delta PFB1224SE / PFB1224HE class) — fan pack

> Structure and the profile schema: [doc 14](../../docs/14_fan_packs.md). Requirements FN-1…FN-10: [doc 02 §6.1](../../docs/02_hardware_specification.md#61-requirements). Paths to core docs are relative to this folder.

## 1. Summary

| Item | Value |
|---|---|
| Status | **stub** (doc 14 §5) |
| Family | `pwm_4wire` |
| Duct | 4″ |
| Connection | B2 XH 6 pins 1–4; fan tap |
| Fleet sizes | 1–15 |
| Profile | [`fan.yaml`](fan.yaml) |

## 2. Maker data

| Requirement | Data |
|---|---|
| FN-2 operating point | More pressure than the budget class [E]; P–Q curve [U] |
| FN-5, FN-6, FN-7 | 4-wire PWM, tach 2 pulses/rev [E: class standard] |
| FN-8a power | Running current [U]: must be ≤ ≈ 2.2 A on the fan tap |
| Other | ≈ $46–51 [E] |

## 3. Connection and cable

As the high-pressure class: pins 1–4 of B2, fan tap, printed 120 mm → 4″ adapter.

## 4. Open items

| # | Item | Impact | Closes at | Plan |
|---|---|---|---|---|
| DL-1 | P–Q curve and running current from Delta's datasheet | Whether the class is designed or stays a stub | Design | Read the maker datasheet for the exact suffix |
| DL-2 | Starting and running current | Fan tap limits | FQ-4 | As the high-pressure class |

## 5. Qualification results

None yet (doc 14 §4).
