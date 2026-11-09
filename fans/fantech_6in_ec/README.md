# 6″ mains EC, 0–10 V input with +10 V reference (Fantech prioAir 6 EC, FG 6M EC / FGC 6M EC) — fan pack

> Structure and the profile schema: [doc 13](../../docs/13_fan_packs.md). Requirements FN-1…FN-10: [doc 02 §6.1](../../docs/02_hardware_specification.md#61-requirements). Paths to core docs are relative to this folder.

## 1. Summary

| Item | Value |
|---|---|
| Status | **designed** (doc 13 §5) |
| Family | `ec_0_10v` |
| Duct | 6″ |
| Connection | B2 4-pole terminal header: red → REF, yellow → CTRL, blue → GND, white → TACH; B2 pin 2 not connected |
| Fleet sizes | 5–15 (6″ kit) |
| Profile | [`fan.yaml`](fan.yaml) |

## 2. Maker data

| Requirement | Data |
|---|---|
| FN-2 operating point | prioAir 6 EC: ≈ 434 / 395 / 355 Pa at 50 / 100 / 150 CFM [V: maker catalogue]; FG 6M EC: ≈ 432 / 386 / 326 Pa [V] |
| FN-3 shut-off | FG 6M EC 1.92 inH₂O [V] |
| FN-4 duct | 6″, on the 6″ kit |
| FN-5 control | "0…10V or PWM" input with its own +10 V reference (red, 1.1 mA, isolated) [V: prioAir IOM] |
| FN-6 no signal | Red joined to yellow is the maker's maximum-speed hookup [V: IOM]; on the CTRL buffer the fan sees ≈ 9.4 V with the hub off: ≈ 94 % if it reads the level linearly [U] |
| FN-7 feedback | White tach, open collector, 1 pulse/rev, 10 mA [V: IOM] |
| FN-8b power | Own mains; keeps running through a hub-supply failure |
| Isolation | Control side isolated from mains [V: prioAir IOM] |
| Other | ≈ $274 list (prioAir 6 EC) [V]; alone in the 6″ kit it beats one high-pressure 120 mm fan by ≈ 25–30 CFM [E, D95] |

## 3. Connection and cable

Bare leads to B2's terminal header (or pins 1, 3, 5 and 6 of the XH 6). B2 pin 2 not connected. PWM mode (DC jumper off). The CTRL buffer draws ≤ 0.45 mA of the fan's 1.1 mA reference (D97).

## 4. Open items

| # | Item | Impact | Closes at | Plan |
|---|---|---|---|---|
| FE-1 | Speed on the CTRL buffer's ≈ 9.4 V with the hub off (F3) | Whether the fan is at full speed or ≈ 94 % during a hub outage | FQ-6 | Measure the speed with the hub unpowered; ≥ 90 % of rated is enough |

## 5. Qualification results

None yet (doc 13 §4).
