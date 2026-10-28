# Design Decisions and Trade Studies

The decision record (D1–D46), the cost-reduction pass (C1–C9) and the trade studies (T1–T3) behind the design. Docs 01–08 describe the result. Decisions are numbered in the order they were made; a later row can amend or supersede an earlier one, and says so.

**Decisions recorded so far**

| # | Decision | Date |
|---|---|---|
| D1 | ~~Klipper secondary MCU (RP2040) on the AD5X USB header.~~ **Superseded 2026-10-25** by D3: the choice of stock Flashforge firmware + zmod which rules out a version-matched Klipper MCU on the C5 Pro and makes it risky on the AD5X. | 2026-10-24 |
| D2 | AD5X enclosures are the **official Flashforge AD5X Enclosure Kit**, so **mods to the kit are allowed**. | 2026-10-24 |
| D3 | **Standalone ESP32-S3 printer nodes** on all three printers, plus the `fleet_exhaust` zmod plugin with a klippy extra. **AD5X link: USB-serial (own protocol) on the bottom internal USB header, data + GND only.** C5 Pro link: Wi-Fi/Moonraker (**amended by D24:** USB-serial too). | 2026-10-25 |
| D4 | **Central 24 V bus** (LRS-450-24) with a **wired CAN** link; hub heater-bus cut. *Power part superseded by D12.* | 2026-10-25 |
| D5 | **Dampers on the printer nodes** (T1 option A′, which supersedes the T1 recommendation of B) with cascaded chamber-ΔP / trunk-vacuum control. | 2026-10-25 |
| D6 | **O2 combo filter** (T2), fan at the wall outlet. **No provision for a future deep-carbon canister.** | 2026-10-25 |
| D7 | **Enclosure mods including insulation** (T3); 150 W PTC heater. | 2026-10-25 |
| D8 | **Full tier**, except pressure sensors: **XGZP6897D** for chamber and filter, **SDP810-500 Pa** for the trunk (trunk amended by D11). Shelly plugs kept. | 2026-10-25 |
| D9 | **Universal carrier PCB + plug-in ESP32-S3 DevKitC-1-format board**, one design for nodes and hub; cost items C1–C9 below (C8 integrated-fan PTC conditional on a bench stall test). | 2026-10-25 |
| D10 | **First Phase 2 task: reverse-engineer the Vivosun G4 control interface** (fallback: a fan with a documented interface). | 2026-10-25 |
| D11 | **Trunk sensor: XGZP6897D (±500 Pa class) instead of the SDP810** (amends D8; −$19 net of the I²C switches). A 2-channel I²C switch on the carrier resolves the fixed 0x6D address (supersedes C7). All five pressure sensors are auto-zeroed together in a hub-coordinated static window (doc 06 §4.1). | 2026-10-25 |
| D12 | **No hub heater relay; heater power from each AD5X's own internal 24 V supply** (reused if O1 shows headroom, else upgraded; a freed stock unit can be the hub supply). Hub PSU is logic-only (LRS-100-24). Adds a **chamber-air KSD301** in the hardware enable chain and a **two-signal heater-runaway trigger for the per-printer Shelly cut**. Rationale: every fault the relay covered is a runaway the mains cut also handles, and the system scales to N printers with only CAT6. | 2026-10-25 |
| D13 | **Bus power injection at the nodes:** every carrier has a 24 V POWER IN feeding the bus (input selection by INJECT jumpers since D22) through a reverse-blocking, current-limited eFuse; node-heat injects from its heater tap; any node can take a 24 V brick; hub trimmed to ≈ 24.5 V for voltage-based sharing. Removes the CAT6 segment current as a chain-length limit (+$12). | 2026-10-25 |
| D14 | **Mains-cut plug backends:** per printer, `shelly` (direct local RPC, recommended), `ha` (any plug HA can switch, via the HA REST API, e.g. existing Matter plugs), or `none`. One firmware interface; read-back after every cut; 1-minute health poll for all. The `ha` path is less fail-safe (it depends on the HA server and the plug's integration) but still beneficial. | 2026-10-25 |
| D15 | **`shelly_node` backend (recommended):** each node hosts a private WPA2 access point (AP+STA) that its printer's Shelly joins as its primary network, with home Wi-Fi as the plug's fallback. Node-local cut on RW-a; hub cuts via CAN `MAINS_CUT`; hub cuts directly if the node dies. HA plugs require a confirmation step in HA before switching. | 2026-10-25 |
| D16 | **Robustness pack:** ESP-NOW link alongside CAN (backup, or the only link on a **wireless-only node** with a 24 V brick); Shelly plug-local power limit; latched state in NVS (lockouts, cut latch, fire, purge resume); staged, signed OTA refused during prints; CAN bus-off recovery and error localisation; hub fallback access point `fleet-hub` (dashboard and plug fallback when the router is down). No new cabling, no hardware cost. | 2026-10-25 |
| D17 | **RJ45 bus safe against network gear:** pinout with equal potential on both wires of every pair (1/2 CAN, 3/6 GND, 4/5 +24 V, 7/8 GND, PoE mode-B polarity); detect-before-power on every OUT jack (10 kΩ node signature); 30 V over-voltage cut-off and 60 V-rated input on the IN jack; ±58 V CAN transceiver. Segment capacity drops to ~1.5 A, covered by D13 injection (+$9). Wired Ethernet through a fleet jack is not possible (O9). | 2026-10-25 |
| D18 | **Wired LAN, option A** (closes O9): a standard 10/100 LAN jack (SPI Ethernet chip, W5500 or DM9051, + RJ45 MagJack + crystal) as a footprint on every carrier, populated on the hub only (+$4), so the dashboard, MQTT and HA reach the hub without Wi-Fi. Fleet jacks stay CAN with the D17 safety circuit. No on-board PoE: a board or the whole bus can run from PoE through an off-the-shelf 802.3af/at splitter with 24 V barrel output into POWER IN. | 2026-10-25 |
| D19 | **Node housing on the branch damper; AD5X recirculation duct** (closes R6, O2, O3): each node's electronics sit in a printed housing attached to its branch damper at the printer's exhaust port, with a grommeted pass-through for cables into the printer. On the AD5X the port, damper and node assembly replaces one of the kit's printed parts, so the polycarbonate isn't cut; on the C5 the cables pass through the existing side exhaust port. Port names embossed on the housing (R10). IFS beside the enclosure (O2). A printed recirculation duct with a 5015 blower is required on each AD5X (O3, doc 08 E7; +$8). | 2026-10-25 |
| D20 | **ENS160 on every sensor head** (closes R3): replaces the SGP41. Rated −40 to 85 °C (SGP41: 50 °C), so the chamber sensor reads through the whole purge; documented registers with raw element resistances; ≈ $4–6 vs $7–9. Needs a 1.8 V LDO on the tab. Node and hub cross-calibrated on raw resistances at idle (doc 06 §5.3). The BME688 was rejected because its full algorithm (BSEC) is closed source. −$15. | 2026-10-25 |
| D21 | **Bus current stays at ≈ 1.5 A per CAT6 segment** (two +24 V conductors at 0.75 A, D17), not raised to ~2 A, so a cheap copper-clad-aluminium or Cat5 patch cable swapped in later still runs inside its rating. Injection at the nodes (D13) covers higher loads. | 2026-10-25 |
| D22 | **Modular power inputs (option C):** every carrier has three isolated 24 V inputs plus the CAN bus: the **internal XT30** on the inside face, the system input for any supply inside the housing (printer supply tap, upgraded internal supply, the hub PSU); a **5.5 × 2.5 mm barrel POWER IN** (replaces the POWER IN XT30; logic bricks, PoE splitters, or a ≤ 120 W brick running the heater capped at ~100 W via a HEAT jumper); and a **4-pin DIN jack** (KPJX-4S class, 7.5 A) for a full-power brick (Mean Well GST160A24/GST220A24 class), as a **population option** with footprint, housing opening and isolation circuit on every board. Ideal-diode OR (LM74700 class) onto the heater rail; an **INJECT jumper header** (one Schottky per input) selects which supplies feed downstream nodes. A heater brick must be on the printer's smart plug; the node locks the heater out if the plug is off but an external heater input is live. A 10 A barrel-only design (option B) was rejected: about $2 per board cheaper, but no trustworthy 24 V barrel bricks above 5 A exist. Printed **blanking plugs** (doc 08 C15) close any unused opening. +$25. | 2026-10-25 |
| D23 | **Adopted from FLOOK32** (schreider/flook32, GPLv3; ghzserg/flook32_plugin): an auto-characterise run that proposes firmware thresholds for the operator to accept (doc 05 §3.2); three-stage retry-then-lockout escalation for recoverable faults, never for runaway, welded FET or fire (doc 05 §4); the zmod plugin's "DO NOT EDIT, copy to user.cfg" config convention, explicit-only settings push and USB auto-discovery (doc 07 §3.1); an alert-only learned thermal model (doc 07 §11.5); fault echoes with trends in the Klipper console (doc 07 §7). No cost. | 2026-10-26 |
| D24 | **C5 Pro on USB-serial too:** node 3 uses the same own-protocol USB link and VBUS-less lead as the AD5X nodes, with Moonraker over Wi-Fi as the backup, so no printer depends on the router. Printer end: the C5's **external USB port** first, **J34** through a hub board as the fallback. The old objections to a C5 USB link (from an earlier design review) applied to a Klipper secondary MCU (version matching, shutdown on unplug, VBUS power) and don't apply to this link. Depends on `cdc_acm` in the C5 kernel (O10); without it the C5 stays on Moonraker only. +$3. | 2026-10-26 |
| D25 | **Printer packs for other printers** (doc 12): the design splits into a core that every printer shares and a **printer pack** per model under `printers/` (profile, README, port adapter and other parts, host install variant, slicer snippets). Printer-specific behaviour comes from a versioned **`profile.yaml`**, not code. A **link contract** with three tiers: 0 `power_only` (print state from **plug power**, doc 07 §6.5; no heater), 1 network API (`moonraker` now; OctoPrint, PrusaLink and Bambu MQTT left to whoever needs them), 2 the USB host plugin, packaged once for any Klipper (zmod and stock-Klipper install variants). A **standard mechanical interface** (100 mm spigot to E3 + C5) so a new printer needs only a port adapter. A **safety contract** packs can't weaken. Node IDs widen to **1–15** (16-ID CAN blocks). Licensing: code MIT, hardware CERN-OHL-P-2.0, docs and CAD CC BY 4.0. The AD5X and C5 specifics moved into `printers/flashforge_ad5x/` and `printers/flashforge_c5pro/`. No hardware or cost change. | 2026-10-26 |
| D26 | **Chamber caps:** AD5X raised from 55 °C to **60 °C operating, 65 °C firmware hard limit, chamber-air KSD301 ≈ 70 °C, RW-b at 70 °C**, assuming the sealed, insulated enclosure (gaskets or RTV on the seams) and every printed part in the chamber in ASA/ABS (HDT ≈ 95 °C, so 60 °C keeps the 30 °C margin). The heat budget still closes at 60 °C (~120 W loss against 150 W plus bed heat). The stock toolhead and cable-chain plastics are the remaining unknown (O11). Node-heat limits now derive from `chamber.cap_c`: hard limit cap + 5 °C, KSD301 and RW-b cap + 10 °C, cap never above 60 °C with a node heater. **C5 Pro** cap 65 °C, Flashforge's rating for its own heater; node-heat limits don't apply there. No cost change. | 2026-10-26 |
| D27 | **Conventions from similar open-source projects** (Nevermore Controller, Klipper's CAN guide, Moonraker, Beacon/Shake&Tune/klipper_z_calibration, Kalico, BTT Panda Breath, DIY fire cutoffs): (1) the Klipper side in its own small repo `fleet-exhaust-klipper`, tagged `vX.Y.Z` for Moonraker's stable channel, with a re-runnable installer that keeps Klipper from reporting "dirty" and detects Kalico; (2) node readings in Moonraker through `[sensor]` over MQTT, with per-job history fields; (3) **fault levels 0–4** with separate Acknowledge and Clear, a **print hold** (`PAUSE`) for heated jobs while the heater is locked out, and a latched cut that re-cuts a plug switched back on (doc 05 §4.1; UI details O12); (4) USB `-if00` fixed as the protocol, `-if02` a log console; (5) VOC baseline learning frozen during prints, purges and dirty air; (6) a **TERM jumper** with 120 Ω on every carrier, fitted at the two chain ends for 60 Ω, plus a commissioning check; (7) required vs optional profile keys, optional ones left unset; (8) extruder target as a print-state cross-check; (9) notification URLs in Apprise format (Pushover and ntfy in the hub); (10) REUSE/SPDX labelling, KiBot and Interactive HTML BOM in CI, LCSC fields. Nevermore is AGPL-3.0: ideas only. No cost change. | 2026-10-26 |
| D28 | **Clearing latched faults** (O12 points 1–2): a heater lockout can be cleared on the dashboard (PIN) **or** with `FLEET_CLEAR` on the printer. A power cut needs **on-site** confirmation: the dashboard inspection checklist, then the node's CLEAR button held 3 s at the printer within 2 minutes. The button is the dev board's existing BOOT button (GPIO 0, read after boot) behind a recessed plunger in the node housing, so there's no BOM change. Wording and screens stay open for the Phase 2 mock-up (O12). | 2026-10-26 |
| D29 | **Damper reference designs** (doc 08 §3b): E3 (100 mm butterfly, MG90S) stays the baseline, with a bleed-resolution bench test (M1a). Nerdiy's PrintAirPipe servo valve is reference only: no native 100 mm valve, a built-in ESP32 that duplicates the node, and CC BY-NC-SA / AGPL licences. MakerWorld rotary-core diverters are kept as an alternative mechanism with finer control near closed, in case the butterfly's bleed resolution proves too coarse on the bench. The mechanism and model are chosen at Phase 2 design time. | 2026-10-26 |
| D30 | **Damper angle encoders and a trunk flow venturi.** Each damper gets an **AS5600** magnetic encoder on the blade shaft (node I²C header 2, ~$1.50 each), so closed is confirmed with the fan off and a stuck or slipped blade is caught directly; the chamber-ΔP check stays as the backup (amends C3, which dropped servo feedback because ΔP proves movement, true only while air moves). The hub gets a printed **venturi** between the filter and the fan, read by a second ±500 Pa XGZP6897D on a new I²C header 3 (the switch's unused channel), ~$6 (**amended by D31:** an SDP31 instead; **D33** went back to the XGZP). Chosen over a thermal anemometer: cheaper, no heated element to foul in fume-laden air, and auto-zeroed with the other sensors; neither measures bleed flow in a 4″ trunk. Commissioning sweeps give per-branch angle–flow curves, which give purge air changes and filter resistance. Per-branch flow sensors stay an option on node header 3. +$12. | 2026-10-26 |
| D31 | **SDP31 on the trunk venturi** (amends D30; **reverted by D33**). A Sensirion SDP31 replaces the XGZP6897D on the venturi: its error is ~0.2 Pa at zero plus ~3 % of the reading, so flow stays readable down to ~5 CFM (the combined bleed of several printers) instead of ~15 CFM. Its own I²C address means it needs no switch channel; it stays on header 3 for wiring. The node chamber sensors stay XGZP. +$14. | 2026-10-26 |
| D32 | **Low-flow measurement without new sensors on the nodes.** (1) **Leak drift:** each node tracks the damper angle that holds −3 Pa against its commissioning baseline. (2) **Heater step test:** once per long heated print, −3 → −5 Pa for 15 min; the extra heater power gives the bleed and the heat lost per CFM. (3) A **printed thermistor anemometer** in the venturi throat at the hub (heated 100k NTC on a resistor heater driven from B1, reference NTC at the inlet, on the hub's otherwise unused NTC inputs), covering ~0.5–10 CFM where the venturi can't. Calibrated by static zero, continuous overlap with the venturi, and temporary reference venturis at commissioning (M9; 10 mm and 15 mm since D33). A bag test was considered and not adopted. +$2. | 2026-10-26 |
| D33 | **Back to an XGZP6897D on the venturi** (reverts D31). With the anemometer covering 0.5–15 CFM and printed 10 mm and 15 mm reference venturis for its commissioning calibration, the SDP31's only remaining edge was a lower in-service overlap window (9–30 vs 20–40 CFM) and less reliance on auto-zero, not worth $14. The SDP31 stays the upgrade on the same header if M4 shows more than ~1 Pa residual after auto-zero. −$14. | 2026-10-26 |
| D34 | **Duct interface: build on ModulAir** ([collection](https://makerworld.com/en/collections/7342813-modulair-modular-air-system)). Its Large (4″) QuickLock bayonet with a printed gasket becomes the standard joint at printer ports, dampers, the filter housing and a hub **instrument spool** (trunk tap ring, calibration slot, venturi, fan adapter), and its 2:1 Y (two chained) is the branch junction; 3:1 and larger don't fit a 256 mm plate. ModulAir already has Bambu and inline-fan adapters. Long runs stay semi-rigid aluminium duct on ModulAir hose adapters. Phase 2 extends the library for this project (doc 08 §3c). Leaks are inward only, but matter for flow measurement: gaskets on every bayonet, and a duct leak test (M10, < 0.5 CFM) whose result is subtracted from branch calibrations. Magnet couplers considered, not used. Licence check: O13. | 2026-10-26 |
| D35 | **Power paths when a printer is cut.** Jumpers only choose what feeds the bus; logic always runs from it. (1) **Fire and runaway order:** heater off and damper close (or purge) are commanded first, the plug cut ~1 s later without waiting, so a node that dies at the cut has already moved its damper. (2) **Sizing:** hub PSU sized for every printer off with the busiest node at alarm peak; test 16 extended. (3) **Configurations:** a table of jumper and outlet rules (doc 04 §6b), hub PSU and fan never on a printer's plug, wireless-only AD5X nodes need a logic brick plus printer heater power; the hub checks every normal plug-off (node must stay online); test 29. (4) **OUT switch:** held on by current in **either** direction (bidirectional zero-drift sense, +$0.40 a board), since normal D13 sharing already sends power upstream and a forward-only sense would cycle the port; direction is readable, node IN protection is a P-FET. (5) **Hub supply loss:** the hub keeps running on printer power, alerts, and enters back-fed mode (LED cap, one servo at a time, siren and push before any cut); test 30. +$2. | 2026-10-26 |
| D36 | **Carrier-power reset hold.** Two 2N7002-class FETs hold the dev board's EN pin low unless the carrier's `+5V_LOGIC` is above ~4.3 V, so a dev board powered from its USB socket (an ordinary cable with VBUS, or a laptop) can't drive its pins into an unpowered carrier; it doubles as a brown-out reset. No bypass jumper (one left fitted would silently defeat it); bench work powers the carrier or pulls the dev board. The Schottky into the dev board's 5 V pin stays (it already blocks USB 5 V from the carrier rails and the buck; **replaced by D37**); a carrier USB socket was rejected (impedance-controlled routing through 2.54 mm headers, connector cost); USB ground stays connected (R11). ~$0.10 a board. | 2026-10-26 |
| D37 | **Dev-board feed: integrated ideal diode** (replaces the Schottky from D36): TI LM66100 (C2869734), alternate Maxinmicro MX66100T (C53121238). Blocks the dev board's USB 5 V from the carrier whenever the output is above the input and drops ~40 mV, keeping a clone AMS1117's headroom. Discrete versions rejected: a gate-to-ground P-FET conducts both ways; a P-FET switched by a "carrier above 4.3 V" detector latches on (USB back-feeds the carrier rail and holds the detector on); a two-PNP discrete ideal diode works but adds matching, temperature and stability unknowns for ~$3. **Dev boards must have a Schottky on each USB VBUS** (Espressif's v1.1 has 1N5819HW), so the carrier's 5 V and its noise never reach a USB host; checked at incoming inspection (test 31). ~+$4 including the extended-part fee. | 2026-10-26 |
| D38 | **Mains-cut plugs restore their last state after a power loss** (resolves doc 05 vs doc 07). A cut plug was off, so it stays off after a mains outage; a power blip doesn't leave running printers off; the latched re-cut covers anything else. Test 25 checks the setting. | 2026-10-26 |
| D39 | **Fire signals on printers that heat their own chamber** (node-lite, e.g. the C5 Pro). F-a uses the printer's chamber sensor and chamber-heater duty over the link, falling back to the sensor head's SHT40 with the any-time threshold; F-b compares the sensor head with the printer's chamber sensor; F-c, F-d and F-e are unchanged. Two-signal corroboration holds with or without the link. The printer's own heater runaway is left to its firmware (doc 05 §5). | 2026-10-26 |
| D40 | **RW-a response: immediate mains cut** (closes O8). Pause-and-stall was not adopted: it relied on `PAUSE` holding the bed, on the TCO tripping in a fan stall, and on leaving a double-faulted heater powered unattended (doc 05 §3.4). | 2026-10-26 |
| D41 | **Staged Phase 2 bring-up** (doc 01 §8): de-risk, exhaust core, heating, monitoring extras, each gated by its commissioning tests. **Monitoring never feeds a safety layer** (doc 05 S-7), so stage 3 can slip. The R1 fallback fan (≈ $100–150) is listed as a BOM contingency, outside the total. Also from the 2026-10-26 review: the trunk ΔP tap stays upstream of the filter (doc 08 §3c corrected), node damper behaviour on hub loss is open item O14, and the hub's move to a 24 V desktop brick is decided with O1. | 2026-10-26 |
| D42 | **Two boards: carrier and a universal sensor board** (amends D9, D20; closes the break-off-tab assumption). JLCPCB takes one design per board and has no mouse-bite panel of different designs ([their panelization rules](https://jlcpcb.com/help/article/pcb-panelization)), so the sensor head can't ride on the carrier panel. One ~30 × 20 mm sensor board, one set of Gerbers and three populations: **S** (SHT40 + ENS160 + LDO), **P** (XGZP6897D) and **E** (AS5600), replacing the sensor-head tab and the bought XGZP and AS5600 breakouts. Each variant is its own order of 5 (S JLC-assembled; P and E JLC or hand-soldered). About +$28 (≈ $609). | 2026-10-26 |
| D43 | **Carrier connectors itemised in the BOM.** The BOM listed the barrel jack, MagJack and cable-side parts but not the board-mount RJ45s, XT30 sockets, JST-XH headers or dev-board sockets. Schedule A.1 (doc 03) adds them: ≈ $4.85 per board, plus 2 more XT30 plugs. +≈ $26 (≈ $635). Counts follow doc 02 §2.2 and doc 04 §2; the connector list is checked again against the Phase 2 schematic. | 2026-10-26 |
| D44 | **Bed-lip LED strip: ~160 LED/m with a diffuser** (was 60/m), for a smoother, softer indicator along the build-plate lip. ~48 LEDs per printer, so the firmware current limit (≤ 0.5 A at 5 V per node) replaces a fixed brightness cap. Strip type: any WS2812-protocol 5 V RGB (SK6812 or WS2812B-type). Silicone diffusers also tolerate heat better than bare adhesive strips; the mounting spot and temperature are still to confirm for the AD5X chamber. +$5 (≈ $640). | 2026-10-26 |
| D45 | **Carrier I/O controller and per-header I²C segments** (amends D11, D30, D42; doc 09). The carrier gets up to **6 external sensor headers plus an internal expander slot**; each external header is a **JST-XH 5** (3.3 V, GND, SDA, SCL, INT/SWIO). A **TCA9548A** 8-channel switch replaces the TCA9543A, so each header is its own I²C segment. That fixes bus capacitance (seven cables on one segment ≈ 315 pF + the carrier [E]), clashing fixed addresses, and a stuck sensor blocking the bus. A **CH32V003** on the carrier switches each header's 3.3 V through its own P-FET (on by default) and takes each header's INT on its own EXTI line. It is also the **SWIO programmer** for any CH32V003 on a sensor board, using timer PWM/capture + DMA as the released host code does. The ESP32 programs the carrier CH32 over **one GPIO**, which is also its interrupt line. 18 of 18 pins used; no shift register. Headers 2–7 rely on single-pin output + capture **[U] (O15)**; the fallback is 5 two-pin headers. The CH32 is optional: without it every header is on and I²C works. **Slot 7 is an internal on-carrier I/O expander** (a second CH32V003, programmed like a sensor board) that takes the door reed, the KSD301 and 24V_HEAT sense lines and the status LED, and LED chain 2 is dropped from the fixed map. That frees five ESP32 GPIOs (13, 21, 41, 42, 47). CAN and the servo output (matrix peripherals) move onto three of them, which puts the LAN SPI on the S3's IO MUX pins (GPIO 10–13); the fifth is the carrier CH32 line, driven by the **RMT peripheral** [U] (bit-banged only as a fallback); the LED chain also runs on the RMT, with DMA. No interface is bit-banged. So **all roles share one pin map with no 0 Ω links and the LAN footprint can be fitted on any role** (amends D18). Six external headers. +≈ $10 (≈ $650). | 2026-10-27 |
| D46 | **Board identity shunt and service button** (amends D45; doc 04 §4). The role is set in hardware by a **2 × 3 pin header and shunt** (HEAT, LITE, HUB) on a resistor divider read by **GPIO 3** (ADC1_CH2, a strapping pin that matters only if an eFuse selects JTAG). At boot the firmware checks it against the role stored in NVS. A mismatch or an unset shunt starts a safe hold with a fault: no heater enable and no outputs. That catches wrong-role firmware and population errors. A shunt was chosen over a DIP switch because it reuses the TERM header part (no new part, no JLC fee) and is changed by hand. **GPIO 0**, the dev board's BOOT button, is read after boot as a **service button**: a short press opens Wi-Fi provisioning; holding it 10 s or more factory-resets NVS, refused while the heater is on. GPIOs 45 and 46 stay unused. +≈ $0.50 (total ≈ $650). | 2026-10-27 |

## Cost-reduction pass (C1–C9)

Applied after the first design was approved, before docs 01–08 were written (D9). Together they took the first estimate from ≈ $580 to ≈ $470–490; later decisions added function and cost (doc 03 §1.1).

| # | Change | Saving [E] | Status |
|---|---|---|---|
| C1 | Sensor head as a break-off tab of the carrier PCB | ~$15 | Superseded by D42 (separate sensor board) |
| C2 | SHT40 instead of SHT45 | ~$9 | In force |
| C3 | Drop the servo feedback tap; the chamber-ΔP loop proves the damper moved | ~$12 | Amended by D30 (AS5600 encoders) |
| C4 | CAT6 patch cable for logic 24 V + CAN + GND, plus a separate 16 AWG XT30 heater lead to heated nodes only | ~$15, no crimp tool | In force |
| C5 | Heater-bus cut on the board's own high-side switch | ~$5 | Superseded by D12 (no hub heater relay) |
| C6 | Watchdog as an RC charge-pump on a toggling pin | ~$1 | In force |
| C7 | One I²C bus at the hub | ~$0 | Superseded by D11 (2-channel I²C switch) |
| C8 | Integrated-fan PTC module, only if stall detection validates without a tach | ~$10–15 | Conditional on a bench test; fallback is a separate 3-wire blower (+$8) |
| C9 | Ducting from hardware-store dryer-vent parts | ~$10–20 | In force; joints follow D34 (ModulAir) |

---

## T1. Where are the branch dampers controlled from?

### What a damper has to do

| Event | Required damper action | Timing need |
|---|---|---|
| Print starts (hot material) | Open to the **bleed** position | Seconds; not critical |
| Print running | Hold bleed; trim for chamber temperature or pressure | Slow |
| Print ends / cancelled / errors | **Purge** (wide open) for N minutes, then close | Seconds |
| Door opened | Open toward purge (keep fumes in) | ≤1–2 s matters |
| Smoke / fire signature | **Close** (protect the shared carbon and the other branches) while mains are cut | ≤1 s matters |
| Printer powered off or crashed with fumes inside | Still able to purge | Must work **without** that printer's Klipper |
| Hub / Wi-Fi down | Fail to a defined, safe position | — |
| Two printers purge together | Serialize, or accept reduced flow | Coordination |

### Options

| | **A. Printer-node drives its own damper** | **B. Hub drives all dampers, wired** | **C. Hub drives all dampers via a tiny node per damper** | **D. Hybrid**: AD5X nodes drive their own, hub drives the C5's |
|---|---|---|---|---|
| **How** | Servo on the AD5X RP2040 (`[servo]`). The C5 Pro needs its own node via the J34 USB-hub interposer | 3-conductor cable from the hub to each damper (3–6 m), 24 V or 12 V feed with a local 5 V regulator at the servo | ESP32-C3 per damper (ESP-NOW/Wi-Fi) or an RS-485 drop | AD5X: as A. C5: as B |
| **Electronics inside the C5 Pro** | **Yes**: hub interposer, firmware matched to unknown FF Klipper (no Klipper 13), alpha zmod, VBUS power risk | **None** | None | None |
| **Door-open response** | Local and fast (door switch on the same MCU) | Via Moonraker over Wi-Fi (~0.2–1 s typical), or wire the door switch to the hub too | Via Moonraker or local input on the node | Mixed |
| **Purge after printer crash / power-off** | **Not possible** (the node is part of the dead Klipper) | **Yes** | Yes | AD5X no, C5 yes |
| **Fire response** | Klipper on the failing printer must act; it may be the thing failing | **Hub acts independently** (close damper + cut mains) | Yes, if the node is alive | Mixed |
| **Does a damper fault kill a print?** | **Yes**: servo stall browns out the aux MCU, Klipper shuts down | No | No | AD5X yes |
| **Wi-Fi loss** | Dampers keep following Klipper, which is good. Hub holds trunk pressure closed-loop | Hub keeps last state, holds trunk vacuum, and fails printing branches to *bleed* | Node fail-state is configurable | Mixed |
| **Purge queue / concurrency** | Needs a network handshake anyway (hub ↔ printers) | Trivial: one controller | Trivial | Partial |
| **Where pneumatic logic lives** | 3 Klipper configs + hub | **One place** | One place + small firmwares | Split |
| **Wiring** | Short servo lead at each printer | 3 long runs (bundle with duct; a C-clip raceway already exists in the catalogue) | Power only at each damper | 1 long run |
| **Cost delta** | Lowest for AD5X. C5 needs node PCB + hub interposer (~$15) | +~$10 cable/connectors, local 5 V regs | +~$5–8 per ESP32-C3 node plus firmware | ~same as B |
| **Firmware burden** | Klipper macros on 3 printers, two different Klipper versions | Hub firmware only | Hub + node firmware | Both |

### Actuator choice (independent of A–D)

| Actuator | Pros | Cons |
|---|---|---|
| **MG90S hobby servo** (current) | $3, simple, and the printed damper (Printables #1023387) is designed for it | No position feedback; jitters/hums when holding; nylon/metal gears creep at 60 °C; **no defined fail position** |
| **Servo with feedback tap** (pot wire to ADC) | Hub *knows* the damper actually moved, so jams and stalls are detected | Small mod, one extra wire |
| **28BYJ-48 stepper + endstop** | Holds with no power, precise, homing gives a known zero | Slower, needs a driver, bulkier |
| **24 V spring-return HVAC actuator** (Belimo-style) | **Fails to a defined position on power loss** (the commercial standard) | $40–120 each, needs a damper with a standard shaft |

### Recommendation

**B (hub-wired) with a feedback-capable servo**, and a **cut-power fail state = last position plus hub watchdog**. Upgrade to spring-return actuators later if you want true hardware fail-safe.

The deciding argument is that the safety-critical damper events (fire, purge after crash or power-off, coordinated purges) are exactly the ones a printer-side controller **can't** handle, because the printer may be the failed component. B also deletes all C5 Pro electronics. Option A's one advantage, fast local door response, is recovered by wiring each door switch to the hub as well (or reading it via Moonraker; ~0.5 s is fine for fume containment).

**What would change my mind:** if the cable routes from the printers to the hub are impractical (different rooms, no path), C (wireless node per damper) becomes the better B.

---

## T2. Outdoor venting: HEPA-only vs HEPA + carbon

### What comes out of the printers

| Emission | Main materials | Captured by HEPA? | Captured by carbon? |
|---|---|---|---|
| Ultrafine particles (10–100 nm) | All, highest for ABS/ASA | **Yes** (H13 ≥ 99.95 % at MPPS) | No |
| Styrene | ABS, ASA (less) | No | **Yes** |
| Caprolactam | PA6/PA6-CF | No | Yes |
| Other VOCs (ethylbenzene, aldehydes) | Various | No | Yes (aldehydes poorly) |

Literature emission rates for single desktop printers are roughly **10–200 µg/min styrene (ABS)** and up to roughly **200 µg/min caprolactam (nylon)**, with large spread between filaments and nozzle temperatures. [E: order-of-magnitude, not your printers]

### Concentration at the outdoor outlet [E]

- **During printing (bleed):** 3 printers × ~100 µg/min styrene into ~5 CFM total bleed (0.14 m³/min) ≈ **2 mg/m³ ≈ 0.5 ppm** at the outlet. Styrene's odour threshold is roughly **0.05–0.3 ppm**, so it's **smellable within ~1 m of the outlet**, and dilution makes it imperceptible a few metres away in any breeze.
- **During a purge** (~60–100 CFM): about 15–20× more dilute. Below the odour threshold almost immediately.
- **Particles outdoors:** a few printers' UFP output is trivial against outdoor background (traffic, cooking). HEPA outdoors is mostly a **courtesy and duct-cleanliness** measure, not a health requirement.

**Conclusion:** carbon matters outdoors **only if the outlet is near a window, door, patio, air intake, or a neighbour's space.** Otherwise both are optional and HEPA is the better of the two to keep.

### Carbon life [E]

At ~100 µg/min per printer, 3 printers, 1,000 hot-material print-hours per printer per year, about **18 g styrene per year**. Working capacity of activated carbon for styrene, de-rated for humidity, is roughly 5–10 % w/w.

| Carbon bed | Carbon mass | Est. styrene life |
|---|---|---|
| Barrel "combo" cartridge (Levoit Core 300 class) | ~100–200 g pellets | ~**6 months to 1 year** |
| 4″ × 12–14″ grow-room carbon canister | ~1.5–2.5 kg | **many years** (media ageing and humidity dominate) |

### Pressure drop and fan headroom [E/U]

At ~100 CFM through 4″: H13 pleated barrel ≈ 80–200 Pa, a thin carbon pellet layer adds ≈ 20–60 Pa, and a deep 4″ carbon canister adds ≈ 40–100 Pa. A Cloudline-class 4″ EC fan has **limited static pressure** [U: depends on exact model]. Every filter stage directly cuts purge flow. For the bleed (~5 CFM), pressure drop is negligible.

### Options

| | **O0. Prefilter only** (reference) | **O1. HEPA only** | **O2. Combo barrel (HEPA + thin carbon)** | **O3. HEPA + deep carbon canister** |
|---|---|---|---|---|
| Particles out | Most UFPs pass | ≥99.95 % captured | ≥99.95 % | ≥99.95 % |
| Odour at outlet | Smellable at ~1 m | Smellable at ~1 m | Clean ~6–12 months, then like O1 | Clean for years |
| Pressure drop (100 CFM) | ~10 Pa | ~80–200 Pa | ~100–250 Pa | ~150–300 Pa |
| Purge flow | Best | Good | Good | Lowest; may need a stronger fan (Cloudline T4 class) |
| Consumable cost | ~$5/yr | ~$15–25 per cartridge | ~$15–25 per cartridge (**same price as O1**) | +$50–80 canister, years of life |
| Fire load in duct | None | None (glass fibre) | Small | **~2 kg carbon**; needs the fire-damper logic (close + cut mains) |
| Printed canister design | Simple | Barrel housing | **Same barrel housing as O1** | Barrel housing + commercial canister in series |

### Recommendation

**O2: a printed barrel housing taking a standard HEPA + carbon combo cartridge.** Combo cartridges cost the same as HEPA-only ones and fit the same housing, so you get first-year odour control for free. Design the trunk so an **O3 deep-carbon canister can be added in series later** (a straight 4″ section with couplers) if anyone ever complains about odour near the outlet.

Two inputs would change this:
1. **Outlet location**: next to a patio, door, or neighbour? Then go straight to O3.
2. **Exact fan model**: if it's a low-static model, the filter choice may be limited by purge flow.

---

## T3. AD5X official enclosure kit: suggested mods (for review)

The kit uses **polycarbonate panels** (good to ~120 °C), so the enclosure itself handles a 50–60 °C chamber. The weak points for this project are **leakage** (P1/P2: every cm² of gap costs ~18 W of heater power at −35 Pa) and **heat-sensitive printed parts**.

| Mod | Why | Effort |
|---|---|---|
| Print all kit printed parts in **ASA/ABS** (not PETG/PLA) | HDT; corners and hinges creep at 55 °C | Material choice only |
| **Seal the IFS / Bowden entry window** (e.g. the "AD5X Bowden Window Remix" on MakerWorld, or a custom PTFE-pass-through plate) | Probably the largest single leak | Small print |
| **Door and lid gaskets** (EPDM D-profile or printed TPU) + foam tape on panel seams | Cuts leakage area by an order of magnitude | Low |
| **Exhaust port high on the rear panel**, 100 mm spigot | Hot fumes rise; a high exit plus low makeup leakage gives clean displacement | Cut PC panel + printed flange (custom) |
| **Small makeup inlet low on the front/side with a gravity flap or filter** | Gives purge air a defined path instead of pulling through the IFS slot; closed during heated prints | Small print |
| **Heater + recirc module mounted low at the rear**, blowing across the floor away from the bed | Avoids hot jet on the part; keeps hot air away from the PC panel near the element | Custom shroud (ASA) |
| **Door reed switch** | For the door-open purge logic | Tiny print + magnet |
| **Cable/USB pass-through grommet** | For the aux board lead and heater power, sealed | Small print |

*Open question for you:* is the IFS on top of the enclosure (AMS-style top mount) or beside it? That decides where the filament pass-through and its seal go.
