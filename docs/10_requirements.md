# 10 - Requirements

> **Status:** back-derived from the Phase 1 design baseline (D1–D93), 2026-11-07. The design came first; this doc states the requirements it meets, so a builder can check a fleet against them and a change can be checked against what it must not break. Each requirement traces to the priority it serves (P-1…P-8, doc 01 §2), the decisions that meet it, and the commissioning test that proves it (doc 05 §9). Limits of use and example systems at those limits: [doc 11](11_use_envelope.md).
>
> **Evidence key:** [V] verified against a primary source · [E] engineering estimate · [U] needs a measurement on the real hardware. A number marked [E] or [U] here is a design target until commissioning confirms it.

---

## 1. How to read this doc

- **Shall** is a requirement; **should** is a goal that may be traded off and says what it's traded against.
- IDs are grouped: **F** function, **S** safety, **I** integration, **E** electrical, **M** mechanical and pneumatic, **N** non-functional. An ID never changes once published; a dropped requirement is struck through, not reused.
- "Verified by" names a doc 05 §9 test where one exists, otherwise a commissioning measurement (doc 06 §10) or a design review.
- Printer-specific values (chamber cap, supply rating, port shape, link details) are not requirements of the core. Each printer pack states them and shows they meet the pack contract (doc 12 §6).

## 2. Function

| ID | Requirement | From | Met by | Verified by |
|---|---|---|---|---|
| F-1 | Every printing chamber shall be held below room pressure, at a setpoint of −2 to −5 Pa (−3 Pa while heating), so leaks flow into it | P-3 | D5, D7; doc 06 §4 | Commissioning ΔP readings (doc 06 §10) |
| F-2 | Fumes shall leave only through one shared duct that is below room pressure along its whole indoor run and vents outdoors | — | D6; doc 06 §1, §9 | Duct leak test M10 (doc 06 §1.1) |
| F-3 | The exhaust shall pass a particle filter and a carbon layer before the outlet | — | D6; doc 06 §8 | Filter ΔP reading on the dashboard |
| F-4 | A printer with no chamber heater shall be heatable to its pack's cap, up to 60 °C, for ABS, ASA, PA and PC; a sealed, insulated kit enclosure shall reach the cap with margin at a 15.5 °C room | P-7 | D7, D12, D26; doc 02 §3, doc 06 §7 | Characterise run (test 24) |
| F-5 | `M191` on a Tier 2 printer shall block print start until the chamber reaches its target, like a native Klipper heater, with a timeout and a choice to continue or pause | P-1 | D3; doc 07 §3.2, §4 | Test 26 |
| F-6 | A printer that heats its own chamber shall keep its heater and fans; the system shall add bleed, purge, fire detection and the mains cut and drive the printer's own circulation modes | P-1 | D39; doc 12 §4 (`klipper.modes`) | Pack commissioning (doc 12 §6 item 7) |
| F-7 | Cooling materials (PLA, PETG, TPU) shall run with exhaust bleed and no heat, below the material policy's chamber limit | — | doc 06 §6 | Commissioning prints |
| F-8 | Every print end, cancel or error shall start a purge; the purge shall cool at a slicer-set ramp and end only when temperature, time, air-change and VOC criteria are all met | — | D5; doc 06 §5.2 | Commissioning prints |
| F-9 | Only one purge shall run at a time; waiting chambers shall stay contained at the heated setpoint | — | doc 06 §5.1 | Two-printer purge at commissioning |
| F-10 | The end of a purge shall be signalled locally (bed LED) and by a push | — | doc 07 §7 | Commissioning prints |
| F-11 | A print started without the macros (touchscreen, vendor app) shall still get bleed and purge under a default policy that never heats | P-1 | doc 07 §5 | Commissioning prints |
| F-12 | Each chamber's temperature, humidity, VOC and pressure, the filter loading, duct flow and fan health shall be visible on a dashboard served by the hub without a cloud service | — | doc 07 §8 | Test 22 |
| F-13 | Push alerts shall go out only when someone's action could help or a print has finished; slow trends (filter loading, room VOC, maintenance) shall go into a daily digest | P-4 | doc 07 §7 | Notification review at commissioning |
| F-14 | The system should publish its readings to Home Assistant over MQTT, as an option only | — | doc 07 §9 | — |

## 3. Safety

| ID | Requirement | From | Met by | Verified by |
|---|---|---|---|---|
| S-1 | Nothing in the system shall shut down Klipper or cancel a print. The only action that ends a print is the mains cut of S-6, and the only one that pauses it is the print hold of a locked-out heater | P-2 | D3, D27; doc 05 §1 S-1, §4.1 | Tests 10, 26 |
| S-2 | Each heater shall stop on a single fault without firmware help: firmware stop, crashed MCU, welded FET on either side, overheated housing or chamber air, or a lost link | P-3 | D12; doc 05 §3.1 (layers L1–L5) | Tests 1–4, 6, 14 |
| S-3 | A node-heat chamber shall never exceed its pack's cap + 5 °C, and the cap shall not exceed 60 °C or the weakest enclosure material's limit | — | D26; doc 05 §3.2 | Tests 3, 24 |
| S-4 | A verified heater runaway (a double fault) shall cut that printer's mains within ~10 s | P-3 | D12, D14; doc 05 §3.4 | Test 5 |
| S-5 | A corroborated fire (two independent signals) shall cut that printer's mains and page; a single signal shall only alert | P-4 | D39; doc 05 §5 | Test 13 |
| S-6 | A mains cut shall latch, re-assert itself within 5 s if the plug is switched back on from any app, and survive a reboot | P-3 | D16, D27; doc 07 §11.1 | Tests 19, 25 |
| S-7 | The hardware heater layers shall work with no Wi-Fi, no hub, no Home Assistant and no printer firmware; the mains cut shall need only the node's own access point or Wi-Fi to reach the plug | P-3 | D12, D15, D16 | Tests 1–4, 15, 22 |
| S-8 | An unpowered or crashed unit shall leave a safe state: heater off, damper holding, exhaust fan running | P-3 | F3; doc 05 §2 | Tests 1, 7 |
| S-9 | The exhaust fan shall keep running when the hub goes silent (doc 02 FN-6); a fan that can't is not a supported exhaust fan | P-3 | F3, D10, D88; doc 02 §6.1 | Test 7 |
| S-10 | An open door shall get more extraction, not less | — | F1; doc 05 §6 | Test 12 |
| S-11 | A plug that can't be reached shall raise an alert within ~1 min, and each plug shall also enforce its own power limit | P-3 | D14, D16 | Tests 15, 18 |
| S-12 | Loss of the hub supply shall leave the hub running on printer power where any printer is on, with a push | P-3 | D35; doc 05 §2 | Test 30 |
| S-13 | Heater power shall come from the printer's own plug, so cutting the printer also cuts its heater | P-3 | D12, D22 | Test 23 |
| S-14 | A heater-supply fault shall not reset the printer's mainboard: the node limits its heater when the printer's rail sags | P-2 | D64; doc 05 §3.3, R12 | Test 8 |
| S-15 | The system shall not replace a room smoke alarm, and the docs shall require one | — | doc 05 S-6 | Design review |
| S-16 | Monitoring features (venturi, anemometer, thermal model, leak drift, heater step test) shall never feed a safety layer | — | D41; doc 05 S-7 | Design review |

## 4. Integration

| ID | Requirement | From | Met by | Verified by |
|---|---|---|---|---|
| I-1 | The system shall not replace or rebuild a printer's firmware, and shall not need a Klipper secondary MCU | P-1 | D1, D3 | Design review |
| I-2 | Any printer that can be connected to a 4″ duct shall be supportable at Tier 0 (plug power only), with closed firmware | — | D25; doc 07 §6.5, doc 12 §3.2 | Pack commissioning |
| I-3 | A Klipper printer shall get the full feature set over a USB-serial link that doesn't depend on the router; a printer whose kernel lacks `cdc_acm` shall fall back to its network API with no other change | P-3 | D3, D24; doc 07 §6.1 | Test 10 |
| I-4 | Unplugging a printer link shall be an alert, never a print failure, and it shall reconnect by itself | P-2 | D3 | Test 10 |
| I-5 | Everything printer-specific shall live in a printer pack; the core shall name no printer model | — | D25; doc 12 | Leak and name check on every commit |
| I-6 | Slicer G-code shall drive modes, chamber targets and purge parameters (`FLEET_EXHAUST`, `FLEET_PURGE`, `M141`, `M191`) | — | doc 07 §3.3 | Commissioning prints |
| I-7 | Smart plugs from more than one ecosystem shall work as the mains cut (Shelly, Tasmota, any plug through Home Assistant) | — | D14, D73; doc 07 §10 | Tests 15, 18 |
| I-8 | Firmware updates shall be signed, refused during a print, and roll back by themselves from a bad image | — | D16; doc 07 §11.2 | Test 20 |

## 5. Electrical

| ID | Requirement | From | Met by | Verified by |
|---|---|---|---|---|
| E-1 | One carrier PCB design, assembled one way, shall serve every role (node-heat, node-lite, hub, combined), chosen by a shunt | P-7 | D9, D46, D47, D84 | Design review |
| E-2 | No screw terminals and no wires soldered to pads; every field connection shall be a locking connector or a jack | P-5 | P-5; doc 04 | Design review |
| E-3 | A node shall keep running when its printer is switched off or cut, so it can still purge that printer | P-3 | D4, D35; doc 04 §6b | Test 29 |
| E-4 | Each CAT6 segment shall carry ≤ ~1.5 A (~36 W), so an ordinary patch cable stays inside its rating | — | D17, D21 | Test 16 |
| E-5 | Any node may inject bus power; no supply shall be back-fed by another, and the bus shall never draw from a sagging printer rail | — | D13; doc 04 §6a | Test 16 |
| E-6 | The hub supply shall carry the whole chain with every printer off and the busiest node at its alarm peak | — | D35; doc 04 §6a | Test 16 |
| E-7 | The bus jack shall harm nothing when plugged into a switch, laptop or PoE port, and vice versa | — | D17 | Test 21 |
| E-8 | Hub↔node coordination shall survive the loss of any one link medium: CAN with an ESP-NOW backup | P-3 | D16 | Test 17 |
| E-9 | The fleet shall grow to 15 nodes without changing message IDs | — | D25; doc 07 §6.2 | Design review |
| E-10 | A printer USB link shall carry no VBUS, and the heater's ground share in the USB lead shall be ≤ 0.3 A | — | D72, R11; doc 04 §5 | Tests 9, 31 |

## 6. Mechanical and pneumatic

| ID | Requirement | From | Met by | Verified by |
|---|---|---|---|---|
| M-1 | Duct 4″ throughout; every joint gasketed; total duct leakage < 0.5 CFM | — | D34; doc 06 §1.1, §2 | Duct leak test M10 |
| M-2 | The exhaust fan shall meet doc 02 §6.1 (FN-1…FN-9): ≥ 50 CFM at ≥ ≈ 250 Pa at full speed (≥ ≈ 330 Pa on long runs), 4″ connection, 4-wire PWM or 0–10 V control, full speed with no signal, and on the board's fan tap 24 V at ≤ ≈ 2.2 A running and a start inside ≈ 2.7 A. The framework lists qualifying products; each build chooses one | — | D85–D88, D91; doc 02 §6 | Flow measurement (doc 06 §10); test 7 |
| M-3 | Bleed shall cost little heat and noise: 1–3 CFM per printer at −3 Pa | P-6 | D7; doc 06 §2, §7 | Commissioning ΔP and flow |
| M-4 | Each printer shall meet the core at one standard interface: a port adapter to the damper's bayonet port, with the node housing on the damper outside the frame | — | D19, D34; doc 12 §5 | Pack review |
| M-5 | A chamber with a makeup inlet shall take its makeup air low and exhaust high; a gravity or spring flap shall close the inlet when the chamber isn't drawn | — | D89, D90; doc 06 §1 | Crack test where fitted (doc 06 §5.2) |
| M-6 | A damper that can't move shall be detected within 10 s and its servo released | — | D30 | Test 27 |
| M-7 | Printed parts should come from existing open models wherever they fit | P-8 | doc 08 | Design review |

## 7. Non-functional

| ID | Requirement | From | Met by | Verified by |
|---|---|---|---|---|
| N-1 | The system should be no louder than the printers: low-speed bleed most of the time, full flow only during a purge | P-6 | doc 06 §2 | Listening check at commissioning |
| N-2 | Cost should be trimmed without losing a safety layer; every trim is recorded with what it gave up | P-7 | design_decisions cost pass | Design review |
| N-3 | Firmware, hardware and docs shall be open source under the licences of doc 12 §8, with REUSE labelling | P-8 | doc 12 §8 | REUSE lint |
| N-4 | A build shall be staged so each layer is proven before the next depends on it, and heated unattended printing shall wait for its tests | — | D41; doc 01 §8 | Stage gates |
| N-5 | Alerts shall have priorities (normal, high, emergency, digest), and an unacknowledged critical alert shall escalate to the hub's panel and buzzer after 15 min | P-4 | doc 07 §7 | Notification review |
