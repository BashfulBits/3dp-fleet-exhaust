# 12 V 120 × 38 mm 4-wire PWM (Wathai 12038 class) — fan pack

> Structure and the profile schema: [doc 13](../../docs/13_fan_packs.md). Requirements FN-1…FN-10: [doc 02 §6.1](../../docs/02_hardware_specification.md#61-requirements). Paths to core docs are relative to this folder.

## 1. Summary

| Item | Value |
|---|---|
| Status | **stub** (doc 13 §5) |
| Family | `pwm_4wire` |
| Duct | 4″ |
| Connection | B2 pins 1–4 through an inline 24 → 12 V adapter that passes PWM and tach and limits a short (D86) |
| Fleet sizes | 1–4 |
| Profile | [`fan.yaml`](fan.yaml) |

## 2. Maker data

| Requirement | Data |
|---|---|
| FN-2 operating point | No P–Q curve [U: retailer listing] |
| FN-3 shut-off | 1.17 inH₂O [U: retailer listing]: below the 1.4 inH₂O screen |
| FN-8a power | 12 V, 2.0 A [U]; only through the adapter |

## 3. Connection and cable

Pins 1–4 of B2 through the inline adapter (D86): the adapter takes 24 V from pin 2 and passes PWM and tach.

## 4. Open items

| # | Item | Impact | Closes at | Plan |
|---|---|---|---|---|
| WT-1 | P–Q curve, or a measured operating point | Below the FN-3 screen | FQ-5 | Measure shut-off and delivered flow |
| WT-2 | A named inline 24 → 12 V adapter that passes PWM and tach and limits a short | Wiring | Design | Pick and list one in this pack |

## 5. Qualification results

None yet (doc 13 §4).
