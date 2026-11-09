# 6″ grow EC with a TRRS port (TerraBloom ECMF-150 class) — fan pack

> Structure and the profile schema: [doc 14](../../docs/14_fan_packs.md). Requirements FN-1…FN-10: [doc 02 §6.1](../../docs/02_hardware_specification.md#61-requirements). Paths to core docs are relative to this folder.

## 1. Summary

| Item | Value |
|---|---|
| Status | **stub** (doc 14 §5) |
| Family | `ec_trrs` |
| Duct | 6″ |
| Connection | B2 TRRS jack, or the maker's breakout cable on the terminal header |
| Fleet sizes | 5–15 (6″ kit) |
| Profile | [`fan.yaml`](fan.yaml) |

## 2. Maker data

| Requirement | Data |
|---|---|
| FN-2 operating point | 350 CFM, 2.01 inH₂O ≈ 500 Pa rated [V: maker manual]; curve [U] |
| FN-5 control | TRRS carries GND, +10 V, tach and 0–10 V/PWM (PWM 15–32 kHz at 10–12 V; maximum speed is +10 V joined to the input; 10 kΩ speed pot) [V: maker manual; U: which contact is which] |
| FN-6 no signal | **Does not start unless a controller is connected before power-up** [V: maker manual]: whether CTRL pulled to REF counts as one is [U] |
| FN-7 feedback | Tach [U: pulses/rev] |

## 3. Connection and cable

A TRRS cable to B2's TRRS jack, or the maker's breakout cable to the terminal header. PWM mode.

## 4. Open items

| # | Item | Impact | Closes at | Plan |
|---|---|---|---|---|
| TB-1 | TRRS contact positions | Wiring | VAL-X2 | Meter and capture with the maker's controller |
| TB-2 | Starts with CTRL pulled to REF in place of its controller (F3) | Whether it is suitable at all | VAL-X2, FQ-6 | Power-up with the hub unpowered |
| TB-3 | P–Q curve | FN-2 | FQ-5 | Measure |

## 5. Qualification results

None yet (doc 14 §4).
