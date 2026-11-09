# 6″ grow EC with a USB-C-shaped port (AC Infinity Cloudline T6 class; Vivosun) — fan pack

> Structure and the profile schema: [doc 14](../../docs/14_fan_packs.md). Requirements FN-1…FN-10: [doc 02 §6.1](../../docs/02_hardware_specification.md#61-requirements). Paths to core docs are relative to this folder.

## 1. Summary

| Item | Value |
|---|---|
| Status | **stub** (doc 14 §5) |
| Family | `ec_usbc` |
| Duct | 6″ |
| Connection | B2 USB-C "FAN 10 V" jack; SWAP set per model |
| Fleet sizes | 5–15 (6″ kit) |
| Profile | [`fan.yaml`](fan.yaml) |

## 2. Maker data

| Requirement | Data |
|---|---|
| FN-2 operating point | 503 Pa rated; ≈ 459 / 415 / 362 Pa at 50 / 100 / 150 CFM with the FG curve shape assumed [U] |
| FN-5 control | VBUS +10 V from the fan, D+ PWM (pulled down against the fan's 10 V), D− tach [U: two hobby projects, not the makers; they disagree on which data line is PWM] |
| FN-6 no signal | Full speed with the cable at the jack and the hub off [U] |
| FN-7 feedback | Tach 2–3 pulses/rev [U] |
| Isolation | Control side isolated from mains [U: meter it, FQ-2] |

## 3. Connection and cable

An ordinary USB-C cable to B2's "FAN 10 V" jack. SWAP straight or crossed, found in FQ-3 (D98) and recorded here. A wrong setting damages nothing.

## 4. Open items

| # | Item | Impact | Closes at | Plan |
|---|---|---|---|---|
| AC-1 | Port mapping: which data line is PWM and which tach, and the levels | Whether the jack works with this fan (R1) | VAL-X2 | Logic analyser on the maker's controller; then FQ-3 sets SWAP |
| AC-2 | Full speed with the hub off (F3) | Silent-hub behaviour | FQ-6 | As FQ-6 |
| AC-3 | P–Q curve | FN-2 | FQ-5 | Measure shut-off and delivered flow |

## 5. Qualification results

None yet (doc 14 §4).
