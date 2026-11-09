# 24 V 120 × 38 mm 4-wire PWM, budget (Jentech JC12038B24UC1 class) — fan pack

> Structure and the profile schema: [doc 14](../../docs/14_fan_packs.md). Requirements FN-1…FN-10: [doc 02 §6.1](../../docs/02_hardware_specification.md#61-requirements). Paths to core docs are relative to this folder.

## 1. Summary

| Item | Value |
|---|---|
| Status | **stub** (doc 14 §5) |
| Family | `pwm_4wire` |
| Duct | 4″ |
| Connection | B2 XH 6 pins 1–4; fan tap |
| Fleet sizes | 1–4, short low-loss runs only |
| Profile | [`fan.yaml`](fan.yaml) |

## 2. Maker data

| Requirement | Data |
|---|---|
| FN-2 operating point | ≈ 176 Pa at 50 CFM [V: maker datasheet and P–Q curve]: **below** the 250 Pa requirement; ≈ 40–60 CFM with a new filter on a short run (D88 [E]) |
| FN-3 shut-off | 1.03 inH₂O [V]: below the screen |
| FN-5, FN-6, FN-7 | As the high-pressure class: 4-wire PWM, full speed open, tach 2 pulses/rev [V] |
| FN-8a power | 24 V, 1.3 A [V] |
| FN-9 noise | 58 dB(A) at full speed [V]: loud there; normal running is well below it |

## 3. Connection and cable

As the high-pressure class: pins 1–4 of B2, fan tap. Printed 120 mm → 4″ adapter.

## 4. Open items

| # | Item | Impact | Closes at | Plan |
|---|---|---|---|---|
| JB-1 | Purge flow through the filter on a real short run | Below FN-2: a purge may run at the airflow budget's floor | FQ-5 | Measure delivered flow at the trunk setpoint with a filter at its alert; qualify only for the run length measured |

## 5. Qualification results

None yet (doc 14 §4).
