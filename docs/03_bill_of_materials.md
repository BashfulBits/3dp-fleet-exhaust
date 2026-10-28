# 03 - Bill of Materials

> System-level BOM for the **reference build** (2 × AD5X + 1 × C5 Pro), built bottom-up for the approved design (Full tier with the D8 sensor choices and the cost-reduction pass in [design_decisions.md](design_decisions.md#cost-reduction-pass-c1c9)). §1.2 shows how it scales to other fleets. Prices are typical US street / AliExpress / JLCPCB prices in late 2026, **[E] ±20 %**. The board-level JLCPCB BOM (part numbers and LCSC codes) is a Phase 2 deliverable.
>
> **Not in the total** (the reference build already had them; add them if you don't): VIVOSUN AeroZesh G4 fan, the AD5X enclosure kits, a room smoke alarm (mandatory, doc 05), and an optional CO alarm.

---

## 1. Cost summary

| Group | Est. cost |
|---|---|
| A. Electronics (boards, ESP32s, sensors) | $301 |
| B. Chamber heating and recirculation (2 × AD5X) | $63 |
| C. Dampers and user interface | $34 |
| D. Power distribution and cabling | $84 |
| E. Pneumatics and filtration | $86 |
| F. Enclosure modifications | $38 |
| G. Safety and smart home | $65 |
| **Total (cash)**, AD5X supplies reused | **≈ $670** |
| Contingency if O1 shows the AD5X supplies need upgrading: 2 replacement supplies (~$80), less the hub LRS-100 (a removed stock unit replaces it, −$17) | **+≈ $63 → ≈ $733** |
| Alternative to upgrading: external DIN-4 heater brick per AD5X (Mean Well GST160A24-R7B, D22) + DIN-4 jack fitted + 2-outlet adapter on its smart plug | ≈ $58 each, **+≈ $115 → ≈ $785** (no mains work in the printers) |
| Reduced-power alternative: barrel brick per AD5X (Mean Well GST120A24-P1M, heater capped ~100 W, D22) + 2-outlet adapter | ≈ $40 each |
| Optional 24 V brick per extra injection point (larger fleets, D13) or per **wireless-only node** (D16; saves that node's CAT6 cable) | ≈ $8 each |
| Contingency if R1 fails (the G4 can't be controlled, or stops when the hub goes silent): a fan with a documented interface, e.g. AC Infinity Cloudline 4″ with its controller port, or a 4-wire PWM EC inline fan (D41) | ≈ $100–150, **not in the total** |
| Filament for printed parts (~2 kg ASA/ABS + 0.3 kg TPU), if not already on hand | ≈ $40 |

```mermaid
pie showData
  title Cash cost by group (≈ $670)
  "A. Electronics" : 301
  "B. Heating" : 63
  "C. Dampers, UI" : 34
  "D. Power" : 84
  "E. Pneumatics" : 86
  "F. Enclosure" : 38
  "G. Safety" : 65
```

### 1.1 Why this is higher than the first $470–490 estimate

> **Updates 2026-10-25:** D11 (XGZP6897D trunk sensor) took the total from ≈ $570 to ≈ $550. D12 (heater power from each AD5X's own supply, no hub relay, chamber-air KSD301) took it to ≈ $515. D13 (bus power injection) took it to ≈ $527. D17 (RJ45 safety) took it to ≈ $536. D18 (hub LAN jack) and D19 (AD5X recirculation blowers) took it to ≈ $548. D20 (ENS160, −$15) and D22 (three isolated power inputs with jumper selection; DIN-4 jack as a population option, +$25) took it to ≈ $558. D24 (C5 USB lead) took it to ≈ $561. D30 (damper encoders, trunk flow venturi) took it to ≈ $573. D31 (SDP31 on the venturi) took it to ≈ $587. D32 (low-flow thermistor anemometer) took it to ≈ $589. D33 (back to an XGZP6897D on the venturi) took it to ≈ $575. D35 (two-way current sense on the OUT ports) took it to ≈ $577. D36–D37 (reset hold, ideal-diode dev-board feed) took it to ≈ $581. D42 (a separate universal sensor board, ordered on its own, replacing the break-off tab and the XGZP and AS5600 breakouts) took it to ≈ $609. D43 (the carrier's connectors itemised, plus the extra XT30 plugs) took it to ≈ $635. D44 (a denser, diffused LED strip) took it to ≈ $640. D45 (carrier I/O controller, 8-channel I²C switch, 7 headers) took it to ≈ $650 (including the on-carrier I/O expander). D47 (one carrier population: LAN on every board, ILIM and RANGE shunts) took it to ≈ $670. The rest of this section explains the original $570.

The first estimate used group-level figures. Building the BOM line by line added items those estimates **under-counted**:
- ΔP tubing, the AD5X USB leads, pre-crimped pigtails, fasteners and standoffs (~$35)
- JLC extended-part fees and shipping on the board order (~$20)
- a spare ESP32 board and spare servo, TCO and fuses (~$15)

The cost-reduction measures (C1–C9) still save about **$110** against the same design built the original way. It's the starting estimate that was low, not the savings.

**If you want to get closer to $500,** these are the remaining levers. None of them changes function:
- Buy the Shelly plugs as a multi-pack or on sale (~$10–15).
- Skip the spare ESP32 board (−$6).
- Source duct parts from a local hardware store's dryer-vent aisle (~$10).
- Use reflective bubble insulation instead of foil-faced foam (~$8).

### 1.2 Scaling to another fleet

The line items in §2 fall into three kinds. Re-count each against your own fleet:

| Kind | Scales with | Examples |
|---|---|---|
| Per printer | Every printer | Carrier board and ESP32 board, sensor board (S), damper and encoder (E), CAT6 segment, USB lead, smart plug, LED strip, port adapter |
| Per heated printer (node-heat) | Printers that get the PTC heater | Group B, the heater lead and XT30 pair, the enclosure mods in group F (pack-specific) |
| Per fleet | Once | Hub carrier, trunk sensors (P), filter housing and cartridge, trunk ducting, outlet hood, hub supply, the JLC order minimums (5 boards per design) |

JLCPCB builds 5 of each board per order, so a fleet of up to 4 printers plus the hub uses one carrier order; more printers need a second. Printer packs ([doc 12](12_porting_guide.md)) list their own parts.

## 2. Line items

### A. Electronics

| Item | Qty | Unit | Total | Notes |
|---|---|---|---|---|
| Universal carrier PCB, JLC fab + SMT/THT assembly (2-layer, 2 oz), incl. setup, extended-part fees and shipping share | 5 | — | $78 | 3 nodes, hub, spare; components per doc 02 §2.2 **except the connectors, which are the next line (D43)**. **One population on all 5 (D47)**: heater stage, fan A, B1/B2, NTC inputs, door input and IN protection on every board, so it's one assembly order and the spare fits any role. +$3 over the role-specific populations. Includes the AUX level shifters (2 × 2N7002 + 4 resistors, ≈ $0.05, D48). |
| Carrier connectors (schedule A.1) | 5 | $5.31 | $27 | D43. Board-mount connectors, THT. The barrel jack and INJECT/HEAT headers are in the eFuse line; the LAN MagJack in the LAN line. |
| Universal sensor board, variant S: SHT40 + ENS160 + 1.8 V LDO, JLC fab + SMT | 5 | $6.40 | $32 | D20, D42. Its own JLC order: parts ≈ $4 each plus setup and shipping share ≈ $12 (R9) [E]. |
| ESP32-S3 DevKitC-1-format board (N8R2/N16R8 clone) | 6 | $6 | $36 | 5 + 1 spare. |
| XGZP6897D differential pressure sensor (I²C 0x6D), on the carrier | 4 | $4 | $16 | 3 chamber (lowest range) + 1 filter on the hub board (±1 kPa class, so a loaded filter stays in range). Assembled by JLC if stocked, otherwise hand-soldered; the hub's range variant is fitted by hand. |
| Universal sensor board, variant P: XGZP6897D ±500 Pa class, trunk vacuum + trunk flow + 1 spare | 3 | $6 | $18 | D42. Trunk vacuum (D11, replaces the SDP810). ±1 % FS ≈ ±5 Pa at a −80 Pa setpoint is enough for a fan-speed loop; static auto-zero (doc 06 §4.1) removes the offset. |
| (second P board) | – | – | – | Trunk flow across the venturi (D30, D33; doc 06 §4.3). Same board and part as the trunk sensor. Upgrade if M4 shows > ~1 Pa residual after auto-zero: Sensirion SDP31, ~$20, same header. |
| Universal sensor board, variant E: AS5600 | 4 | $2 | $8 | D30, D42: one per damper, 1 spare. A 5-board JLC order (or bare boards with hand-soldered AS5600). |
| Sensor-header I/O: 8-channel I²C switch (TCA9548A) + 2 × CH32V003 (carrier controller and on-carrier I/O expander) + 7 P-FETs (AO3401 class) + per-header pull-ups and ESD | 5 | $2.10 | $11 | D45, doc 09 §6. Populated on every carrier for one assembly BOM. One I²C segment per header (fixed XGZP6897D addresses, bus capacitance); per-header power, interrupt and CH32 programming. Replaces the 2-channel TCA9543A. |
| 0.96″ I²C OLED | 1 | $3 | $3 | Hub panel, optional, on its own sensor header (D48). A node OLED is the same part, optional and not in the total. |
| Bus-injection eFuse with reverse blocking (TPS2663 class) + POWER IN barrel jack + INJECT/HEAT jumper headers + 3 Schottky, on the carrier | 5 | $2.80 | $14 | D13, D22; one per board, including extended-part share. |
| Detect-before-power OUT-port circuit (window comparator, latch, back-to-back N-FETs, current limit, bidirectional zero-drift current sense) | 5 | $2.20 | $11 | D17, D35 (two-way sense, +$0.40); one per board. The TJA1051T/3 transceiver costs about the same as the SN65HVD230 it replaces. |
| Carrier-power reset hold (2 × 2N7002, divider) + ideal-diode dev-board feed (TI LM66100DCKR, C2869734; alt. MX66100T, C53121238; replaces the Schottky) | 5 | $0.33 | $5 | D36, D37; one per board. Includes the ~$3 JLCPCB extended-part fee for the ideal diode [V]. |
| LAN jack parts: W5500 or DM9051 + RJ45 MagJack + 25 MHz crystal | 5 | $4 | $20 | D18; fitted on every carrier since D47 (was hub only), so any node can take a wired link. The W5500's extended-part fee was already paid for the hub. |
| Input isolation: 3 × ideal-diode controller (LM74700 class) + N-FET, on the carrier | 5 | $4.50 | $23 | D22; INT, BAR, DIN, populated on all 5 so a DIN jack can be added later. |
| | | | **$301** | |

**A.1 Carrier connectors, per board (D43).** Populated on all 5 boards for one assembly BOM; JLC THT assembly or hand-soldered. Prices are LCSC-class [E].

| Connector | Per board | Unit | Per board $ | Where (doc 04) |
|---|---|---|---|---|
| Shielded RJ45 jack | 2 | $0.70 | $1.40 | IN and OUT, fleet bus (both fitted on every board, D47) |
| XT30PW-M 90° PCB socket | 2 | $0.40 | $0.80 | Internal 24 V input, heater output |
| JST-XH 2.54 mm THT header, 2/3/4/5-pin | 15 | $0.10 | $1.50 | Sensor headers 1–6 (5-pin, D45), fan A, LED 1, NTC ×2, KSD301 loop, door, B1, B2, AUX (5-pin, D48; replaces the two spare-input headers) |
| 2.54 mm pin header + shunt | 9 | ≈ $0.09 | $0.81 | TERM, servo, board-identity 2 × 3 (D46), ILIM, RANGE 1, RANGE 2 (D47); spare shunts |
| 2 × 22-pin female header | 2 | $0.40 | $0.80 | ESP32-S3 dev board socket |
| | | | **$5.31** | × 5 boards ≈ $27 |

Not in this table, because they are priced elsewhere: the POWER IN barrel jack and INJECT/HEAT headers (eFuse line), the LAN MagJack (LAN line), the sensor boards' own JST-XH (their lines), the DIN-4 jack (population option), and the mating plugs (group D).

**Population option (not in the total):** 4-pin DIN power jack (Kycon KPJX-4S class, 7.5 A), ≈ $2.50 per board, THT. Footprint and housing opening on every carrier. Long lead time on the Kycon part; a Same Sky equivalent is listed.

### B. Chamber heating and recirculation (AD5X ×2)

| Item | Qty | Unit | Total | Notes |
|---|---|---|---|---|
| 24 V 150 W PTC heater, integrated fan, aluminium housing | 2 | $18 | $36 | C8, conditional on the bench stall test (R8). Fallback: bare PTC + 3-wire 5015 blower, +$8. |
| KSD301 normally-closed thermostat | 5 | $1 | $5 | Per AD5X: one on the heater housing (rating per O7) and one in chamber air (≈ 70 °C, D12, D26). 1 spare. |
| Thermal cut-off (TCO), rated per O7, DC-capable | 4 | $0.75 | $3 | 2 spares (one-shot). |
| ATO blade fuses 10 A | 5 | $0.40 | $2 | Printer supply taps and node inputs, plus spares. |
| NTC 100k (3950) with leads | 4 | $1 | $4 | Chamber air + heater housing per AD5X. |
| Sheet aluminium (0.5–1 mm) for heater shrouds | 1 | $5 | $5 | |
| 24 V 5015 blower (4-wire) | 2 | $4 | $8 | AD5X recirculation duct (O3, doc 08 E7). PWM lead left open runs it at full speed; node output B1 switches it. |
| | | | **$63** | |

### C. Dampers and user interface

| Item | Qty | Unit | Total | Notes |
|---|---|---|---|---|
| MG90S metal-gear servo | 4 | $3 | $12 | 3 + 1 spare. |
| Reed switch + magnets | 2 | $2 | $4 | AD5X doors. |
| Addressable RGB LED strip, WS2812-protocol, 5 V, ~160 LED/m, with a frosted or milky silicone diffuser, 1 m | 1 | $12 | $12 | D44. Bed-lip indicator strips (~0.3 m each × 3, ~48 LEDs each). SK6812 RGB and WS2812B-type parts both work. Prefer RGB over RGBW unless the firmware driver adds it. |
| Active 24 V buzzer, momentary button | 1 | $3 | $3 | Hub panel. |
| Diametric magnet (6 × 2.5 mm) | 4 | $0.75 | $3 | D30: one per damper, on the blade shaft (doc 06 §3); 1 spare. The AS5600 is on sensor board variant E (group A). |
| | | | **$34** | |

### D. Power distribution and cabling

| Item | Qty | Unit | Total | Notes |
|---|---|---|---|---|
| Mean Well LRS-100-24 (hub, logic only) | 1 | $17 | $17 | Genuine Mean Well. $0 if a stock AD5X supply is freed by an upgrade (doc 02 §3.1). |
| AD5X internal supplies | 2 | — | $0 | **Reused** in the baseline. Upgrade cost is the contingency in §1. |
| IEC C14 inlet, fused + switched | 1 | $5 | $5 | |
| Inline ATO fuse holders | 2 | $1.50 | $3 | At each AD5X supply's heater tap. |
| XT30 connector pairs (plug side) | 6 | $0.75 | $5 | D43. Mate with the carrier sockets: internal 24 V input on each node-heat and the hub (3), heater output on each node-heat (2), plus 1 spare. |
| CAT6 patch cables, **distinct colour** (1 × 3 m hub→node 1, 2 × 1.5 m node→node, 1 spare) | 4 | $3.75 | $15 | Daisy chain; labelled "24 V – not Ethernet" (R10). |
| 14 AWG 2-conductor cable | 3 m | $1.20 | $4 | Two short heater leads from each printer's supply tap (≤ 1.5 m). 14 AWG keeps the heater return share in the USB-link ground low (R11, doc 04 §6). |
| JST-XH pre-crimped pigtail kit (2/3/4-pin) + housings | 1 | $10 | $10 | No crimp tool needed. |
| Silicone tubing 2 × 4 mm | 5 m | $1.20 | $6 | ΔP sensor taps. |
| USB-C plug → 4-pin header lead, **VBUS not connected** | 2 | $3 | $6 | AD5X link (doc 04 §5). |
| USB-C plug → USB-A plug lead, **VBUS not connected** | 1 | $3 | $3 | C5 link via its external USB port (D24). A small USB 2.0 hub (~$8) keeps the port usable for a thumb drive; the J34 fallback adds a CH334F hub board (~$5). |
| M3 standoffs, screws, heat-set inserts, heat shrink, zip ties | 1 | $10 | $10 | |
| | | | **$84** | |

### E. Pneumatics and filtration

| Item | Qty | Unit | Total | Notes |
|---|---|---|---|---|
| 4″ semi-rigid aluminium duct, 25 ft | 1 | $20 | $20 | Trunk and branches. |
| 4″ wye | 2 | $8 | $16 | Fallback only: the default is two printed ModulAir 2:1 Ys (D34), filament only. Kept in the total until O13 clears the ModulAir licence. |
| 4″ worm-drive clamps | 12 | $0.85 | $10 | |
| Aluminium foil HVAC tape | 1 | $8 | $8 | |
| 4″ through-wall dryer vent hood with gravity flap | 1 | $15 | $15 | Backdraft protection. |
| HEPA + carbon combo cartridge (Levoit Core 300 class) | 1 | $15 | $15 | O2; consumable, ~6–12 months of carbon life. |
| Thermistor anemometer parts: 2 × 100k glass-bead NTC, 2.2 kΩ ½ W metal-film heater resistor, thermal epoxy, leads | 1 | $2 | $2 | D32, doc 06 §4.4. The printed probe is C18; the hub's NTC inputs and B1 output are already on the carrier. |
| | | | **$86** | |

### F. Enclosure modifications (AD5X ×2)

| Item | Qty | Unit | Total | Notes |
|---|---|---|---|---|
| EPDM D-profile gasket + closed-cell foam tape | 1 | $12 | $12 | Doors, lids, seams. |
| Insulation: foil-faced foam board or reflective bubble | 1 | $20 | $20 | Rear, sides, top. |
| PTFE couplers (PC4-M10) + grommets | 1 | $6 | $6 | Bowden pass-through plates. |
| | | | **$38** | |

### G. Safety and smart home

| Item | Qty | Unit | Total | Notes |
|---|---|---|---|---|
| Shelly Plus Plug US | 3 | $20 | $60 | Mains cut + metering (`shelly_node` backend recommended, or `shelly`). **$0 if you use plugs you already own via the `ha` backend** (D14), e.g. Matter plugs, at the robustness cost in doc 05 §3.5. |
| Pushover licence (one platform) | 1 | $5 | $5 | |
| | | | **$65** | |

## 3. Consumables and running cost

| Item | Interval | Cost |
|---|---|---|
| HEPA + carbon combo cartridge | ~6–12 months (carbon odour life), or sooner when the filter-ΔP alert fires | ~$15 each |
| Heater intake pads (carbon/HEPA, cut to size) | ~6 months | ~$5/yr |
| TCO | Only after a trip | <$1 |
| Electricity | Heaters ~100–130 W while holding temperature; fan ~5–27 W | — |
