# 4″ EC with a TRRS port (TerraBloom ECMF-100 class) — fan pack

> Structure and the profile schema: [doc 13](../../docs/13_fan_packs.md). Requirements FN-1…FN-10: [doc 02 §6.1](../../docs/02_hardware_specification.md#61-requirements). Paths to core docs are relative to this folder.

## 1. Summary

| Item | Value |
|---|---|
| Status | **stub** (doc 13 §5) |
| Family | `ec_trrs` |
| Duct | 4″ |
| Connection | As the ECMF-150 |
| Fleet sizes | 1–4; marginal |
| Profile | [`fan.yaml`](fan.yaml) |

## 2. Maker data

| Requirement | Data |
|---|---|
| FN-2 operating point | 160 CFM, 1.34 inH₂O ≈ 334 Pa rated [V: maker manual]; ≈ 254 / 136 Pa at 50 / 100 CFM, FG curve shape assumed [E]: leaves nothing for leakage at a loaded filter |
| FN-5, FN-6, FN-7 | As the ECMF-150 |
| Other | ≈ $78 [V]. Neither 4″ EC holds a 50 CFM purge through a filter at its alert, even at four printers (D95 [E]); a high-pressure 120 mm fan does better for a fraction of the cost |

## 3. Connection and cable

As the ECMF-150.

## 4. Open items

| # | Item | Impact | Closes at | Plan |
|---|---|---|---|---|
| TC-1 | As TB-1 and TB-2 on the ECMF-150 | Wiring, F3 | VAL-X2, FQ-6 | Shared with the ECMF-150 pack |
| TC-2 | Delivered flow at a loaded filter | Marginal on FN-2 | FQ-5 | Qualify only for the run measured |

## 5. Qualification results

None yet (doc 13 §4).
