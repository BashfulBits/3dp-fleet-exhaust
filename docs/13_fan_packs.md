# 13 - Fan Packs: Adding an Exhaust Fan

> Design baseline, 2026-11-09 (D105). How a bought exhaust fan becomes supported: what a fan pack is, the profile the hub reads, how a fan model is qualified, and the list of fan packs. The requirements every exhaust fan must meet stay in [doc 02 §6.1](02_hardware_specification.md#61-requirements) (FN-1…FN-10); this doc is how a product is shown to meet them. Printers have packs of their own ([doc 12](12_porting_guide.md)); fan packs follow the same pattern and are simpler.
>
> **Evidence key:** [V] verified against a primary source · [E] engineering estimate · [U] needs a measurement on the real hardware.

---

## 1. Core and fan packs

| Core (same for every fan) | Fan pack (one per fan model or class) |
|---|---|
| The requirements FN-1…FN-10 (doc 02 §6.1) | The maker's data against each requirement, with evidence |
| B2's port, its external jacks, the fan tap and the CTRL buffer (doc 02 §2.2.7, doc 04 §7a) | Which jack or pins the fan uses, any adapter or cable, the jumper settings (SWAP, DC) |
| The trunk loop, airflow budget, fan-fault alerts (doc 06 §4) | `fan.yaml`: what the hub needs to know (§3) |
| Validation of the port itself (`validation/`, VAL-X1) and of any undocumented control port (VAL-X2, R1) | Open items for that model and its qualification results (§4) |

A pack names a **model** where one product was checked, or a **class** where any fan with the same ratings and wiring behaves the same (for example "24 V 120 × 38 mm 4-wire PWM, ≥ 1.9 inH₂O shut-off"). A class pack lists the products known to belong to it.

## 2. Pack layout

```text
fans/
  _template/              # copy this to start a new fan
  <vendor>_<model>/
    fan.yaml              # what the hub needs to know (§3)
    README.md             # maker data, connection, settings, open items, results
    cad/                  # duct adapter or cable drawing, if the fan needs one
```

Every fan pack README has the same sections:

| § | Section | Contents |
|---|---|---|
| 1 | Summary | Status, family, duct size, connection, supply, fleet sizes, profile link |
| 2 | Maker data | Each FN requirement against the maker's data, tagged [V], [E] or [U], with sources |
| 3 | Connection and cable | Pins or jack, adapter or cable, isolation of the control side, jumper settings, printed duct adapter |
| 4 | Open items | `# | Item | Impact | Closes at | Plan`. *Closes at* is **Design** (desk work, before `designed`), an `FQ-n` test, or `VAL-X2` for a control port that must be mapped first |
| 5 | Qualification results | The FQ tests with dates, units, the build that measured them, and the numbers |

A pack records facts about the fan model, never about one owner's installation.

## 3. Fan profile (`fan.yaml`)

The hub loads the fan profile from the dashboard at installation (INS-2), as a node loads its printer profile. The schema is versioned (`schema: 1`) and unknown keys are rejected.

| Key | Type | Meaning |
|---|---|---|
| `id`, `name` | string | Pack folder name and display name |
| `status` | enum | `stub`, `designed` or `qualified` (§5). The dashboard warns on anything but `qualified` |
| `family` | enum | `pwm_4wire`, `ec_0_10v`, `ec_usbc`, `ec_trrs` (doc 02 §6.1 wiring families) |
| `duct_in` | number | 4 or 6: the duct the fan serves (FN-4), directly or through its adapter |
| `port` | enum | `b2_pins_1_4`, `b2_terminal`, `b2_usbc`, `b2_trrs`; `fan_a` for a second fan (FN-10) |
| `supply` | enum | `fan_tap` (B2 pin 2), `own_mains`, `own_brick`, `adapter_12v` (inline 24 → 12 V adapter, D86) |
| `tach_ppr` | number | Tach pulses per revolution; 0 if the fan has no tach |
| `rated_rpm` | number | Full-speed speed from FQ-3, for the fan-fault alert |
| `swap` | enum | `straight` or `crossed`, USB-C jack only (D98); otherwise absent |
| `dc_filter` | bool | The DC jumper (D95): a fan that reads 0–10 V but not PWM. Default false |
| `printers` | list | Fleet sizes the pack supports, `[min, max]`, from FN-2 and the airflow budget (doc 06 §4.5) |
| `second_fan_from` | number | Fleet size from which a second fan is needed (FN-10); absent if never |

Jumpers are hardware: the profile records what they must be, and the installation wizard shows it (INS-2). A wrong SWAP damages nothing (D98).

## 4. Fan qualification

A fan model is qualified by whoever owns one, on a carrier that passed validation, with **the system's own sensors and a multimeter only**: the trunk ΔP sensor, the venturi, the tach input and the dashboard's test modes. It is the same step as printer qualification (doc 12 §7) and sits between validation and installation (D104). A fan whose control port is undocumented first needs its port mapped in validation (VAL-X2, R1), which uses a logic analyser; its pack can't be `designed` until then.

| Test | What | Was |
|---|---|---|
| **FQ-1 Paper check** | Each FN requirement against the maker's data, with sources; a fan with no P–Q curve is screened by FN-3. Sets `status: designed`, or `stub` where a requirement rests on [U] data | doc 02 §6.2 statuses |
| **FQ-2 Wiring and isolation** | Meter the fan's cable or the pack's adapter pin by pin to the jack or pins in §3. For a mains fan: with the mains plug in and the control lead free, meter control GND to mains earth and to each mains pin (AC and DC ranges): no voltage, so the control side is SELV. Set the jumpers from §3 | doc 02 §6.1 shared-ground note |
| **FQ-3 Control and tach** | In test mode, command 0, 25, 50, 75 and 100 %: speed follows. On the USB-C jack find SWAP first (D98): straight, then crossed, until tach or speed follows. At 100 %, the tach reading matches the maker's rated speed within 10 % with the chosen `tach_ppr`; record `rated_rpm`. Note the lowest duty that keeps it turning | doc 02 §6.2 step 3 |
| **FQ-4 Start and current** (board-powered fans) | Start the fan from standstill at 100 % twenty times: the tap eFuse never trips (the dashboard logs a tap fault). Meter the running current at full speed in series with the fan's supply lead: ≤ ≈ 2.2 A (FN-8a) | doc 02 §6.2 step 2 |
| **FQ-5 Pressure and flow** | In the run it will serve (or a test run with the filter, venturi and hood): with every damper closed, read the trunk ΔP at full speed (shut-off, FN-3 screen); then open one branch and read the venturi flow at the trunk setpoint (FN-2, ≥ 50 CFM). Record filter ΔP vs fan speed with a new cartridge | M1, M2 (fan half) |
| **FQ-6 Fails to full speed** (F3, FN-6) | Full speed three ways: with the hub's B2 released in test mode (outputs at their reset state), with the hub unpowered (a fan on its own mains), and with the control cable unplugged where the fan allows it. A fan that stops in any of them is *not suitable* | test 7 (fan half) |
| **FQ-7 Second fan** (fans offered for FN-10) | Two fans in series: trunk flow at full speed with both, then with each stopped in turn; record the flow left with one fan down for the fan-fault alert | M12 |
| **FQ-8 Noise** | By ear at the nearest workspace: bleed speed is no louder than the printers (FN-9). Record the duty where it becomes intrusive | M7 (fan half) |

**Recording and status.** Record each test in the pack's §5 with the date, the unit's date code, the jumper settings and the numbers. When FQ-1…FQ-6 pass (and FQ-7 where offered), set `status: qualified`. One unit is enough; a second date code is welcome, since fans change between batches.

## 5. Fan pack list

**Status:** *stub* = the carrier is wired for it, but at least one requirement rests on unverified data [U]; use it only once its open checks pass (was "designed for, not validated"). *designed* = the maker's own data meets every requirement; only qualification is open (was "qualifies on paper"). *qualified* = FQ passed on at least one unit (was "validated"). **No fan is qualified yet.**

| Pack | Status | Printers | Connects to | Open before qualified |
|---|---|---|---|---|
| [24 V 120 × 38 mm 4-wire PWM, high pressure (Jentech JC12038B24UC2 class)](../fans/jentech_jc12038b24uc2/README.md) | designed | 1–15 (a second one on fan A from ≈ 9) | B2 pins 1–4, fan tap | Starting and running current (FN-8a) |
| [6″ mains EC, 0–10 V with +10 V reference (Fantech prioAir 6 EC, FG 6M EC)](../fans/fantech_6in_ec/README.md) | designed | 5–15 (6″ kit) | B2 terminal header | Speed on the CTRL buffer's ≈ 9.4 V with the hub off (F3) |
| [24 V 120 × 38 mm 4-wire PWM, budget (Jentech JC12038B24UC1 class)](../fans/jentech_jc12038b24uc1/README.md) | stub | 1–4, short runs | B2 pins 1–4, fan tap | Purge flow through the filter: ≈ 176 Pa at 50 CFM is below FN-2 |
| [24 V 120 × 38 mm 4-wire PWM, industrial (Delta PFB1224SE / HE class)](../fans/delta_pfb1224/README.md) | stub | 1–15 | B2 pins 1–4, fan tap | Running current, P–Q curve |
| [12 V 120 × 38 mm 4-wire PWM (Wathai 12038 class)](../fans/wathai_12038_12v/README.md) | stub | 1–4 | B2 pins 1–4 through a 24 → 12 V adapter | P–Q curve (below the FN-3 screen) |
| [6″ grow EC with a USB-C-shaped port (AC Infinity Cloudline T6 class; Vivosun)](../fans/ac_infinity_cloudline_t6/README.md) | stub | 5–15 (6″ kit) | B2 USB-C "FAN 10 V" | Port mapping (VAL-X2), SWAP, F3, P–Q curve |
| [6″ grow EC with a TRRS port (TerraBloom ECMF-150 class)](../fans/terrabloom_ecmf150/README.md) | stub | 5–15 (6″ kit) | B2 TRRS jack or the maker's breakout on the terminal header | Contact positions (VAL-X2); starts with CTRL pulled to REF (F3); P–Q curve |
| [4″ EC with a TRRS port (TerraBloom ECMF-100 class)](../fans/terrabloom_ecmf100/README.md) | stub; marginal | 1–4 | As the ECMF-150 | As the ECMF-150; ≈ 254 Pa at 50 CFM [E] leaves nothing for leakage at a loaded filter |

**Not suitable** (no pack; doc 02 §6.1 says why):

| Fan | Why |
|---|---|
| 4″ grow EC below FN-2 (AC Infinity Cloudline S4 class) | ≈ 185 Pa at 50 CFM [E] |
| RJ12-port grow fans (Spider Farmer, Mars Hydro, TrolMaster controllers) | No jack (an RJ12 plug fits the RJ45 bus jack). Mapping the port (VAL-X2) and an adapter could give it a stub pack |
| 120 × 25 mm case fans; on/off fans; 0–10 V inputs without a +10 V reference output | Too little pressure, or they stop when the hub is down (F3) |

## 6. Open items for fans

The fan items of the core register (doc 01 §7) now live here and in the packs.

| # | Item | Impact | Closes at | Plan |
|---|---|---|---|---|
| **R1** | Control interface and silent-hub behaviour of a fan with a proprietary controller (F3, D10) | Hub may not set speed; "hub dead → fan keeps running" may be impossible | VAL-X2 per fan family, then FQ-6 | First Phase 2 task for such a fan. Fallback: a fan with a documented interface (a mains EC fan with a +10 V reference, or a board-powered 4-wire PWM fan) |
| R2 | Fan static-pressure curve unpublished | Purge flow through the filter may be low | FQ-5 per pack | Screen against FN-2 and FN-3 (FQ-1); the airflow budget lowers a purge toward 20 CFM before it caps cooling (D95); drop the carbon layer before buying a bigger fan |

## 7. Adding a fan

1. Copy `fans/_template/` to `fans/<vendor>_<model>/` (or a class name). Fill in §1–§3 and `fan.yaml`.
2. Run FQ-1 on paper and set `status: stub` or `designed`. If the control port isn't published, open an issue for VAL-X2 first.
3. Run FQ-2…FQ-8 on a unit and record them; set `status: qualified`.
4. Add the pack to §5 and its BOM line to doc 03 if it is the example fleet's fan.

## 8. Sources

The maker documents and hobby projects behind the packs are listed in each pack and in doc 02 §6.4.
