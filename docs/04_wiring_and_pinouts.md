# 04 - Wiring, Cables and Pinouts

> Design baseline, 2026-10-25. Connector reference designators (J-numbers) and exact connector part numbers are assigned in the Phase 2 schematic. This document fixes the **signals, cable types, pin assignments and grounding rules** the schematic must follow.

---

## 1. Power and signal distribution

```mermaid
flowchart TB
  IEC["Mains → IEC inlet<br>fuse + switch"] --> PSU["LRS-100-24<br>24 V"]
  PSU -->|"XT60 (INT)"| HC["Hub carrier<br>injection eFuse"]
  HC -->|"fan A port"| FAN["G4 fan<br>(own mains)"]
  HC ===|"CAT6 #1"| N1["Node 1<br>(AD5X #1)"]
  N1 ===|"CAT6 #2"| N2["Node 2<br>(AD5X #2)"]
  N2 ===|"CAT6 #3"| N3["Node 3<br>(C5, lite)"]
  BR["24 V brick<br>(optional)"] -.->|"POWER IN"| N3
```

Each AD5X also feeds its own node's heater input:

```mermaid
flowchart TB
  SP["Mains → smart plug"] --> AD["AD5X internal<br>24 V supply (O1)"]
  AD --> FU["ATO 10 A fuse<br>≤ 15 cm from terminals"]
  FU -->|"14 AWG ≤ 1.5 m"| N["Node heater supply<br>(internal XT60)"]
  SP -.-> BRK["24 V brick<br>(alternative, D22)"]
  BRK -.->|"HEATER DIN or barrel jack<br>(in place of the tap, D64)"| N
  N -.->|"shunt (§6a)"| BUS["Bus injection"]
```

RJ45 IN/OUT on every carrier; a TERM jumper on every carrier, fitted at the two ends of the chain (§3). Every carrier can inject 24 V onto the bus through its eFuse, from whichever inputs its INJECT jumpers select (§6a).

- **The hub PSU powers node logic only**, over CAT6.
- **Each AD5X's heater runs from that printer's own 24 V supply** (D12, doc 02 §3.1). The printer's Shelly plug removes both.
- **The G4 fan runs on its own mains power.** The hub only drives its control port (R1).

## 2. Cables

| Cable | From → to | Type | Conductors | Connectors | Length |
|---|---|---|---|---|---|
| **Local injection** (optional) | 24 V brick or PoE splitter → any node's POWER IN barrel, INJECT BAR fitted | Brick lead | +24 V, GND | 5.5 × 2.1 mm barrel (D66) | as needed |
| **Wireless-only node** (optional, no CAT6) | 24 V brick on an ordinary outlet (not the printer's plug) → node POWER IN, INJECT BAR fitted (§6b) | Brick lead | +24 V, GND | 5.5 × 2.1 mm barrel (D66) | as needed |
| **Bus** | Hub → node 1 → node 2 → node 3 | CAT6 patch, **distinct colour, labelled "24 V – NOT ETHERNET"** (R10) | 4 pairs (§3) | RJ45 | ~3 m, 1.5 m, 1.5 m |
| **Heater feed** | AD5X supply terminals → inline ATO 10 A (≤ 15 cm from the terminals) → strain relief → node-heat internal XT60 (inside the housing, D64) | 14 AWG 2-conductor | 24V_HEAT, GND | Supply terminal / XT60 | ≤ 1.5 m |
| **External heater feed** (optional, D22) | 24 V DIN-4 brick, **plugged in on the printer's own smart plug** → node-heat **HEATER DIN** jack, in place of the printer tap (D52, D64) | Brick lead | +24 V, GND (2 pins each) | 4-pin DIN | brick lead |
| **Barrel heater feed** (optional, D22, D64, D66) | 24 V barrel brick, ≥ 160 W and ≤ 10 A for full power (a 120 W brick at `heater_power` ≤ 95 W), **on the printer's own smart plug** → node-heat **HEATER barrel** jack (15 A), in place of the printer tap | Brick lead | +24 V, GND | 5.5 × 2.5 mm barrel | brick lead |
| **Heater element** | Node XT60 out (inside the housing, through a strain relief, D64) → PTC (via TCO) | Element leads + TCO in series on the + lead | 2 | XT60 | ≤0.5 m |
| **Heater fan** (separate leads, C8) or 5015 blower (fallback) | Node fan A → fan | 22–24 AWG | 24 V, GND, (tach), (PWM) | JST-XH 4 | ≤0.5 m |
| **Recirculation-duct blower** (AD5X, O3) | Node B1 → 5015 blower | 22–24 AWG | 24 V, switched GND (PWM and tach leads unused) | JST-XH 2 | ≤1 m |
| **KSD301 loop** | Node → heater-housing KSD301 → chamber-air KSD301 (in series) → node | 22–24 AWG, high-temp silicone | 2 | JST-XH 2 | ≤1 m |
| **NTCs** | Node → chamber air NTC, heater housing NTC | NTC leads | 2 each | JST-XH 2 | ≤0.5 m |
| **Sensor head** | Node sensor header 1 (switch channel 0) → sensor board S in chamber | Pre-made Qwiic cable (4-core 28 AWG, D69), ≤0.5 m (I²C at 100 kHz) | GND, 3.3 V, SDA, SCL | JST-SH 4, Qwiic pinout (D69) | ≤0.5 m |
| **ΔP tubes** | Carrier XGZP barbs → chamber / room | Silicone 2 × 4 mm | — | barbs | ≤0.5 m |
| **Servo** | Node → damper | Servo lead (extend if needed) | 5 V, GND, signal | 3-pin servo | ≤0.5 m |
| **Damper encoder** (D30) | Node sensor header 2 (switch channel 1) → sensor board E (AS5600) on the damper's shaft end | Qwiic cable | GND, 3.3 V, SDA, SCL | JST-SH 4, Qwiic pinout (D69) | ≤0.5 m |
| **Door** | Node → reed switch | 2-core | 2 | JST-XH 2 | ≤1 m |
| **Bed LED strip** | Node LED1 → WS2812 strip | 3-core 22 AWG | 5 V, DATA, GND | JST-XH 3 | ≤1 m |
| **AD5X USB link** | Node dev-board native USB-C → AD5X bottom internal USB header (pack) | USB 2.0, **VBUS not connected** (§5) | D+, D−, GND, shield | USB-C plug / 4-pin housing | ~0.5–1 m |
| **C5 USB link** (D24) | Node dev-board native USB-C → C5 external USB port (fallback: J34 via a hub board, §5) | USB 2.0, **VBUS not connected** (§5) | D+, D−, GND, shield | USB-C plug / USB-A plug | ≤ 1.5 m |
| **Trunk ΔP** | Hub sensor header 2 (switch channel 1) → sensor board P (XGZP6897D) at trunk | Qwiic cable, 500 mm stock (a generic 1 m cable if needed) | GND, 3.3 V, SDA, SCL | JST-SH 4, Qwiic pinout (D69) | ≤1 m |
| **Filter ΔP** | Hub on-board XGZP → filter housing taps | Silicone tube | — | barbs | ≤1 m |
| **Trunk flow** (D30) | Hub sensor header 3 (switch channel 2) → sensor board P (XGZP6897D) at the venturi (D33) | Qwiic cable, 500 mm stock (a generic 1 m cable if needed); silicone 2 × 4 mm from the venturi taps | GND, 3.3 V, SDA, SCL | JST-SH 4, Qwiic pinout (D69) | ≤1 m |
| **Low-flow anemometer** (D32) | Hub NTC 1, NTC 2 and B1 → C18 probe in the venturi | 2-core each | NTC leads; heater 24 V switched | JST-XH 2 ×3 | ≤1 m |

**Cable route:** the bus and heater cables follow the trunk duct along the wall, clipped to it with printed duct clips (doc 08), then drop to each printer with its branch.

## 3. RJ45 bus pinout (T568B wiring)

| RJ45 pin | T568B colour | Signal | Pair |
|---|---|---|---|
| 1 | white/orange | **CAN_H** | pair 2 (twisted) |
| 2 | orange | **CAN_L** | pair 2 |
| 3 | white/green | **Presence loop** (D52): GND at the IN jack, sensed at the OUT jack | pair 3 |
| 6 | green | **Presence loop** (D52): GND at the IN jack, sensed at the OUT jack | pair 3 |
| 4 | blue | +24 V logic | pair 1 |
| 5 | white/blue | +24 V logic | pair 1 |
| 7 | white/brown | GND | pair 4 |
| 8 | brown | GND | pair 4 |

**Rule (D17): both wires of every pair carry the same potential.** An Ethernet port has a transformer winding across each pair, so DC between the two wires of one pair (as on pins 3 and 6 in the earlier pinout) would drive amps through it. With equal potential per pair, any DC only appears *between* pairs, which a port's magnetics and coupling capacitors block. That's the same way PoE mode B carries power on pairs 4/5 (+) and 7/8 (−), with the same polarity as here.

- **Capacity:** two conductors for +24 V (≤ 0.75 A each, **≈ 1.5 A per segment, ~36 W**) and two for GND (pins 7/8, the same 0.75 A each). Pair 3 carries only the presence loop's sense current since D52; the two extra GND wires it used to be only lowered the ground drop, not the segment limit. **Kept at 1.5 A on purpose (D21):** a cheap copper-clad-aluminium or Cat5 patch cable swapped in later still runs inside its rating. Injection at the nodes (§6a) covers more load.
- **Load:** each bed strip is ~48 LEDs (D44), about 2.9 A at full white, so the firmware limits LED brightness by estimated current to **≤ 0.5 A at 5 V per node** (a few percent of the bus, ~0.1 A at 24 V). A current-limit switch on the LED chain (≈ 0.75 A, D56, doc 02 §2.2) backs that up in hardware. Status colours at a diffused, low brightness stay well under that. Three nodes then stay ≤ 1.2 A from the hub alone. Bus power injection (§6a) covers anything more.
- **Jacks are directional:** each node has an **IN** jack (toward the hub) and an **OUT** jack (away from it). The hub has one OUT jack. Pins 1, 2, 7, 8 pass straight through. Pins 3 and 6 don't: the IN jack ties them to GND through a resistor (D63), and the OUT jack senses them as the presence loop (§3a, D52); +24 V reaches the OUT jack only through the detect-before-power switch (§3a). CAN stubs on the board are ≤ 50 mm.
- **Termination (D27):** every carrier has a **TERM jumper**: a 2-pin 2.54 mm header in series with a **120 Ω 1 % resistor** (1206, 0.25 W) across CAN_H and CAN_L, beside the OUT jack. Fit the shunt at the **two physical ends** only: the hub, and the node with nothing in its OUT jack. Two 120 Ω in parallel give the **60 Ω** that ISO 11898 and Klipper's CAN guide require, so the bus also meets Klipper's rules if someone reuses it. Adding a node at the end means moving one shunt.
- **Termination check (commissioning):** with the bus powered off, meter CAN_H to CAN_L (pins 1–2) at any spare jack or a breakout: **54–66 Ω passes**; ~120 Ω means one end is missing, ~40 Ω means three are fitted. The firmware can't measure termination, but a node whose OUT jack detects nothing shows a dashboard reminder to confirm its TERM shunt.

**Bus parameters:** CAN 2.0A at **125 kbit/s**. The total bus length (~6 m) is far inside the limit.

### 3a. Safe if plugged into network gear (R10, D17)

| Protection | What it does |
|---|---|
| **Same potential per pair** (above) | No DC across any Ethernet transformer winding |
| **Detect-before-power on every OUT jack** (hub and nodes) | +24 V is off until a fleet node is detected. The port probes pins 4/5 vs 7/8 with ≤ 5 V at ≤ 1 mA and powers up only if it sees **a node's 10 kΩ ±5 % signature**, or **an already-powered fleet segment (21–28 V; 22 V before D64, which left a segment fed from a node several hops away, at ≈ 22.4–23 V, almost no margin)** from a node with local injection. It refuses an open port, an Ethernet port's ~150 Ω DC-coupled termination, a PoE device's 25 kΩ signature, a short, or anything else. Once on, power stays on only while the **presence loop** is closed (D52). On the OUT jack, pins 3 and 6 are tied together and pulled up weakly to 3.3 V, through a series resistor and clamp rated for 60 V on the pins. Every node's IN jack ties pins 3 and 6 to GND through a resistor equal to the OUT pull-up (D63), so the pair reads ≈ 1.6 V with an OUT on the far end (the OUT's loop reads that as closed, and the IN jack's blue link LED lights) and ≈ 0 V with another IN, a LAN or Ethernet jack or nothing. Both wires of the pair always sit at the same potential, so the D17 rule holds. Unplugging either end of the cable opens the loop: the port drops +24 V after a short debounce (~10 ms) and returns to probing. The loop doesn't depend on current, so it works the same in both directions: a node's injection feeding upstream (D13), or the whole chain feeding the hub when its supply fails (D35). An Ethernet port reads as open, because its pair-3 winding has no DC path to GND. The pull-up and clamp must not form a 19–26.5 kΩ PoE signature between pairs 1/2 and 3/6 [U]. Hardware window comparator + latch driving a **bidirectional current-limited switch** (back-to-back N-FETs, so injected power can flow both ways; off, their body diodes block both ways). The D35 zero-drift current sense (INA190 class) is dropped. Firmware can read the port state but isn't needed to keep it safe [U: circuit in Phase 2]. **Ratings (D57):** an SMBJ58CA on the OUT jack's +24 V pins, 60 V switch FETs, and a prober input rated for 60 V, so a 48–57 V injector plugged into OUT is refused and damages nothing |
| **Node IN jack** | 10 kΩ signature, an SMBJ58CA TVS and a 100 V reverse clamp on the jack side, and the **IN guard** (D56): two P-FETs back to back, which conduct both ways when on (never a series diode, so a node can feed power upstream, D35), block both ways when off, and turn off above **≈ 34 V**. Polyfuse and FETs rated 60 V. A passive PoE injector of 48–57 V, of either polarity, leaves the node off and damages nothing (the TVS's 58 V stand-off covers it); above 58 V is not covered [E]. Before D56 the IN path had no over-voltage cut-off (the eFuse's ~30 V cut-off guards only the injection inputs). A PoE switch sees an invalid signature (10 kΩ is outside PoE's 19–26.5 kΩ window), so it never powers the node |
| **Fault-tolerant CAN transceiver** (±58 V bus pins, TJA1051T/3 class) | Ethernet signals or stray PoE voltage on pins 1/2 can't damage it. A CAN driver feeding an Ethernet winding is short-circuit current-limited by the transceiver [E] |
| Distinct cable colour, "24 V – NOT ETHERNET" cable labels, **port names embossed on the node housing and hub enclosure** (R10), a **blue port-live LED** beside each OUT jack (D62), a **blue link LED** beside each IN jack (lit when the far end is an OUT, D63), a **hardware CAN activity LED** that blinks on other units' frames (D63; doc 02 §2.2), and an optional **bus-colour port bezel** (C20, doc 08) | Kept as the first line of defence. The OUT LED is lit exactly when +24 V is on that jack, which no Ethernet port shows; with the IN LED, an IN–IN, OUT–OUT or Ethernet mis-plug leaves at least one end dark |

OUT-jack power logic:

```mermaid
stateDiagram-v2
  direction TB
  [*] --> Probing
  Probing : Probing (+24 V off)
  Probing : probe 4/5 vs 7/8, ≤ 5 V, ≤ 1 mA
  Probing --> Powered : 10 kΩ node signature
  Probing --> Powered : far end at 21–28 V
  Probing --> Probing : anything else
  Powered : Powered (+24 V on)
  Powered : current-limited switch
  Powered --> Probing : presence loop opens
```

"Anything else" covers an open jack, an Ethernet port (~150 Ω), a PoE device (25 kΩ) and a short. 21–28 V at the far end means a fleet segment that is already powered; 21 V still rejects a 19–20 V laptop or USB-C supply [E].

**Result:** plugging any fleet jack into a switch, laptop or PoE port puts no voltage on it and damages neither side. A fleet jack **can't carry real Ethernet data**, because CAN uses pair 1/2, which 10/100 Ethernet needs. Wired LAN is a separate, standard jack on the hub instead (D18).

## 4. ESP32-S3 GPIO map (carrier, all roles)

Rules applied:
- The carrier's **BOOT button (GPIO 0)** is read after boot as the node's **CLEAR button** (doc 05 §4.1); nothing else on the carrier connects to GPIO 0. Holding it through a reset only enters the bootloader, with every output off and the latches kept in NVS.
- Strapping pins: **GPIO 0** is read only as the service button. **GPIO 3** (fan A enable) and **GPIO 45** (output B2) are outputs with pull-downs, so both are low at reset (D68): GPIO 45 low selects the 3.3 V flash supply the WROOM-1 modules use [V: ESP32-S3 datasheet strapping table], and GPIO 3 low selects pad JTAG on a board whose `STRAP_JTAG_SEL` eFuse is burned (doc 07 §11.8). Nothing may pull either pin high at reset. **GPIO 46 is used as an input only** (the `24V_HEAT` sense, D64): it is low whenever the high side is off, including at reset, and a normal boot (GPIO 0 high) ignores it, and it matters only when download mode is entered, where it must be low or floating, which it is because the high side is off in hardware [V: Espressif esptool boot-mode documentation]. Also avoid native USB (19, 20, the USB-C receptacle) and flash/PSRAM (26–34). **GPIOs 35–37** are free only on the quad-PSRAM N8R2 module (the octal-PSRAM R8 modules use them), and all three are used (GPIO 37 by D74, GPIO 35 and 36 by D75). **GPIOs 43 and 44** are UART0, brought out to the 1 × 3 console header (D72). **GPIO 38** is the LAN module's interrupt (D72) and **GPIO 48** the KSD301 loop sense (D74); a DevKitC's RGB LED used one of them. **GPIOs 39–42 are the JTAG pins** (D68) and carry nothing else.
- Analog inputs use **ADC1 only** (ADC2 is unusable while Wi-Fi is active).

| GPIO | Node-heat | Node-lite | Hub | Notes |
|---|---|---|---|---|
| 0 | **Service button** (the carrier's BOOT tact switch, D46, D72) | Same | Same | Input; 10 kΩ pull-up per Espressif's module reference. Held at power-up, it enters download mode as usual |
| 1 | Chamber-air NTC (ADC) | – (fitted, spare) | **Anemometer reference NTC** (D32) | ADC1_CH0. 47 kΩ pull-up, ≈ 4.6 kΩ with the RANGE 1 shunt on (node-heat, D47) |
| 2 | Heater-housing NTC (ADC) | – (fitted, spare) | **Anemometer heated NTC** (D32) | ADC1_CH1. RANGE 2 shunt as for GPIO 1 |
| 3 | **Fan A high-side enable** (D68, was GPIO 18) | – | Fan A enable (if used) | 10 kΩ pull-down, low at reset (strapping pin, above). The board identity moved to the carrier I/O controller (D68, below) |
| 4 | Heater current, LM2904 out (ADC, D50) | – | – | ADC1_CH3; sampled at a fixed rate by ADC1 continuous mode with DMA (below) |
| 5 | **Heater-rail voltage** (ADC, D75) | Same | Same; **hub supply** (D78) | ADC1_CH4. 100 kΩ / 9.1 kΩ divider, 100 nF at the pin. Back from the controller's PB0 (D75); AUX A moved to GPIO 36 |
| 6 | AUX B: spare, or **UART1 RX** from the printer (D48) | Same | **Acknowledge button** (edge interrupt, debounced; D49, was GPIO 5): the on-board tact switch, or a remote button on AUX B | Level shifter as GPIO 36; 5 V-tolerant at the connector. The on-board tact switch is on this line (D49). The external heater-input sense (D22) moved to the carrier I/O controller's ADC (D47) |
| 7 | **Tap-side heater voltage** (ADC, D64, D75) | Same | Same | ADC1_CH6. Divider as GPIO 5, on the internal XT60's side of the tap ideal diode. Back from the controller's PB1 (D75); the heater PWM moved to GPIO 35 |
| 8 | I²C SDA | I²C SDA | I²C SDA | On-board XGZP, carrier I/O controller (I²C target), 8-channel I²C switch; behind it, one channel per external sensor header 1–8 (header 1 = sensor head; header 2 = node damper encoder, or hub trunk XGZP; header 3 = hub trunk-flow XGZP, D30/D33; header 4 = optional OLED on any role, on its own channel, D48; D45, D54, doc 09 §3). While the ESP32 programs an MCU, GPIO 8 carries UART2 RX instead (doc 09 §4.1) |
| 9 | I²C SCL | I²C SCL | I²C SCL | UART2 TX while programming (doc 09 §4.1) |
| 10 | **LAN CS** (D45) | LAN CS | LAN CS | SPI2 IO MUX pin (no GPIO matrix), to the 2 × 5 LAN module socket (D72). Only a board with a W5500 module plugged in uses it |
| 11 | **LAN MOSI** (D45) | LAN MOSI | LAN MOSI | SPI2 IO MUX pin |
| 12 | **LAN SCK** (D45) | LAN SCK | LAN SCK | SPI2 IO MUX pin |
| 13 | **LAN MISO** (D45) | LAN MISO | LAN MISO | SPI2 IO MUX pin. The door input is read through the carrier I/O controller (doc 02 §2.4) |
| 14 | **Fan A PWM** (open-drain, D70) | – | **G4 fan control** (per R1) | LEDC. Floats while the ESP32 is in reset, so a 4-wire fan runs at full speed. LED chain 1 moved to the carrier I/O controller (PA5, D70, D71) |
| 15 | Heater enable charge-pump drive (square wave) | – | – | No wave → high-side off. **A plain GPIO toggled by the safety task, deliberately not a PWM peripheral** (doc 07 §2): a peripheral would keep the wave running if the safety task hung |
| 16 | **CAN TX** (D68, was GPIO 41) | CAN TX | CAN TX | TWAI, routed through the GPIO matrix; pull-up on the transceiver's TXD so the bus stays recessive while the pin floats at reset. Fan A's PWM moved to GPIO 14 (D70) |
| 17 | Fan A tach in | – | **G4 tach/health** (per R1) | Pull-up + clamp |
| 18 | **CAN RX** (D68, was GPIO 42) | CAN RX | CAN RX | TWAI. Fan A enable moved to GPIO 3 |
| 21 | **Servo PWM** (50 Hz) | Servo PWM | – | Pull-down, held low at boot (was GPIO 12). LED chain 2 was dropped from the fixed map (D45) |
| 35 | Heater low-side PWM (D75, was GPIO 7) | – | – | LEDC. 10 kΩ pull-down. N8R2 module only (rules above) |
| 36 | AUX A: spare, or **UART1 TX** to the printer (D48, D75, was GPIO 5) | Same | **Smoke dry-contact input** (edge interrupt; D49) | Through a 2N7002 level shifter to the AUX header (below), so digital only. N8R2 module only |
| 37 | **Tap ideal-diode disable** (D74, was the controller's PF1) | Same | Same | Output to the 2N7002 on the MX74700T's EN; the 4.7 kΩ gate pull-down keeps the diode on while the pin floats at reset. Driven only by the two-supply check (doc 02 §2.2). N8R2 module only (rules above) |
| 38 | **LAN interrupt** (W5500 INTn, D72) | Same | Same | Input, 10 kΩ pull-up, active low, falling-edge interrupt; idle high with no module fitted. Was a DevKitC RGB LED pin |
| 39 | **JTAG TCK** (MTCK, D68) | Same | Same | To the 1 × 6 JTAG header only. Output B1 moved to the carrier I/O controller (PA8) |
| 40 | **JTAG TDO** (MTDO, D68) | Same | Same | JTAG header only. Output B2 moved to GPIO 45 |
| 41 | **JTAG TDI** (MTDI, D68) | Same | Same | JTAG header only. CAN TX moved to GPIO 16 |
| 42 | **JTAG TMS** (MTMS, D68) | Same | Same | JTAG header only. CAN RX moved to GPIO 18 |
| 45 | Output B2: spare (D68, was GPIO 40) | – | **Buzzer** | Pull-down, low at reset (strapping pin, above) |
| 46 | **High-side output sense** (`24V_HEAT` through a divider and clamp, D64) | Same | Same | Digital input, strapping pin used as an input only (above). Low whenever the high side is off. Moved here from the carrier I/O controller by D64 so a welded high-side FET is seen directly by the safety task |
| 43 | **UART0 TX** (console header, D72) | Same | Same | 1 × 3 header with GPIO 44 and GND; the ROM bootloader and the ESP-IDF console use it |
| 44 | **UART0 RX** (console header, D72) | Same | Same | |
| 47 | **Carrier I/O controller interrupt** (D45, D54, doc 09 §2.3) | Same | Same | Falling-edge interrupt input, 10 kΩ pull-up; the controller pulses it low for ≤ 20 µs (open-drain) when any of its own inputs changes (door, group rail; doc 09 §2.3). The same net reaches the controller's reset pin through 2.2 kΩ with 470 nF to ground (D76): the ESP32 resets the controller by driving this pin low (open-drain) for 1 s, then releasing it, and never drives it high. The board status LED moved to the carrier I/O controller |
| 48 | **KSD301 loop sense** (D74, was the controller's PA10) | – (loop not wired) | – (loop not wired) | Digital input with a falling- and rising-edge interrupt, through the sense divider and clamp. Read directly by the safety task, so it doesn't depend on the controller. Was a DevKitC RGB LED pin |

**Heater current sense (D50).** The 5 mΩ low-side shunt feeds one half of an LM2904 (LCSC C18229, Preferred) as a non-inverting amplifier, gain 40 (39 kΩ / 1 kΩ, 1 %), powered from `+5V_PWR` through 10 Ω and 10 µF (D72; `+5V_LOGIC` before):
- **Input:** shunt → 1 kΩ → + input. A **330 kΩ from 3.3 V** to the + input adds a pedestal of ≈ 10 mV, so the output never sits on the 0 V rail when the amplifier's offset (up to ±7 mV) is negative. With the pedestal, zero current reads ≈ 0.4 V, away from the ADC's poor region near 0 V. A 10 nF at the + input filters PWM edges.
- **Output:** 1 kΩ in series into GPIO 4, because the output can reach ≈ 3.5 V when saturated; the resistor limits the current into the pin's clamp. The output clips at ≈ 12 A, so cold-PTC inrush peaks read as full scale.
- The unused half is tied off as a follower to GND.
- **Zero at boot:** before the safety task starts the charge pump, the high side is off in hardware (the gate driver has no enable) and the low side is off, so current is zero unless both switches have failed. The firmware averages GPIO 4 for ~100 ms and stores that as the zero. The zero also cancels the ADC's own offset. A zero outside the window that pedestal and offset can explain (≈ 0.12–0.68 V) is a fault: the heater stays locked out and the node alerts, since it means both switches or the amplifier have failed.
- **Re-zero:** only while both switches are commanded off, and an update that moves more than 50 mA from the boot zero is refused (and counted), so a slowly rising leak can't hide in the zero.
- After zeroing, the error left is drift (≈ 7 µV/°C, about 60 mA over a 40 °C swing) and ±6 % gain (shunt and resistors), against the 0.3 A L4 threshold (doc 05 §3.1) [E].

**Board identity (D46).** A 2 × 3 pin header with one shunt (the same 2.54 mm header and shunt as TERM) selects the board's role. It is read by the carrier I/O controller on **PA0** (ADC_IN0, D68; GPIO 3 before), which has 100 kΩ to 3.3 V, plus 100 nF to GND so the ADC sees a low source impedance. The shunt adds a resistor to GND (1 % parts) [E]:

| Shunt position | Resistor to GND | PA0 voltage | Read as |
|---|---|---|---|
| HEAT | 10 kΩ | ≈ 0.30 V | `node-heat` (below 0.65 V) |
| LITE | 47 kΩ | ≈ 1.06 V | `node-lite` (0.65–1.6 V) |
| HUB | 220 kΩ | ≈ 2.27 V | `hub` (1.6–2.8 V) |
| No shunt | — | 3.3 V | **unset** (above 2.8 V) |

The ESP32 reads it from the controller once at boot, after the controller starts (doc 09 §4.2). If the shunt and the role stored in NVS disagree (doc 07 §2), or the shunt is unset, the board starts in a **safe hold**. If the controller can't be reached, the board runs the stored role with an "identity unchecked" alert, and the heater stays off until the identity has been read once since boot (D75). In safe hold, the heater enable stays off, the outputs stay off and the LEDs work, and a "role mismatch" fault is raised. Since D47 every carrier carries the same parts, so the shunt is what makes a board a given role. The firmware also checks the **RANGE shunts** against it: at boot, after the heater has been off, each NTC must agree with the sensor head's SHT40 to within a few °C. A RANGE shunt left in the wrong position reads tens of degrees off, and raises the same mismatch fault and safe hold. The **ILIM shunt** (eFuse limit, doc 02 §2.2) can't be read by the firmware; commissioning checks it, and a missing one gives the lower, safe limit.

**Service button (D46).** The carrier's BOOT tact switch (GPIO 0, D72), read after boot:
- **Short press:** opens Wi-Fi provisioning for 10 minutes.
- **Hold ≥ 10 s:** factory reset of NVS (Wi-Fi credentials, node ID, calibration), refused while the heater is on. After the reset, the role comes back from the shunt.

**AUX header (D48).** One JST-XH 5 replaces the two spare-input headers. Each contact has its own GND pin, so two pre-crimped pigtails can share one housing with no splice:

| Pin | Signal | Hub | UART printer link (`marlin_serial`, doc 12) |
|---|---|---|---|
| 1 | 3.3 V (header group rail, ≈ 1 A limit, D57) | – | not connected |
| 2 | GND | Smoke contact return | Printer GND |
| 3 | A: GPIO 36 | Smoke dry contact | **TX**, node → printer RX |
| 4 | B: GPIO 6 | Remote acknowledge button (optional; the on-board switch is in parallel) | **RX**, printer TX → node |
| 5 | GND | Remote button return | – |

- **Levels:** each line runs through a 2N7002 shifter (gate to 3.3 V, 10 kΩ pull-up to 3.3 V on the ESP32 side, 4.7 kΩ to 3.3 V on the connector side). A 3.3 V printer works both ways. A 5 V printer's TX is blocked by the FET, so the ESP32 never sees 5 V; ≈ 0.4 mA flows back into the 3.3 V rail through the pull-up. The node's TX reaches a 5 V AVR at 3.3 V against a 3.0 V threshold, which works but with little margin [E]; for those boards doc 12 prefers the USB-host route.
- **Speed:** ≈ 0.5 µs rise on a 1 m cable [E], fine at 115200 and 250000 baud.
- **Peripheral:** UART1 through the GPIO matrix. UART0 (GPIOs 43, 44) stays the console.
- A and B are digital only: the shifter doesn't pass an analog level.
- **On-board button (D49):** a TS-1187A-B-A-B SMD tact switch (C318884, Basic, 1.5 mm actuator) from B's connector side to GND, on every carrier. It sits on the RX line on purpose: pressing it can't short the TX output. Its position on the board is fixed at layout so the C9 plunger lines up; the node housing (C5) has no plunger for it.

It is reached with the node housing open. Holding it at power-up still enters download mode, as on any ESP32 board.

**Expansion port (D77).** A 2 × 5 2.54 mm pin header on the carrier I/O controller's spare pins, for an odd SPI or UART sensor or analog voltages. Pin 1 is 3.3 V from the header group rail, pins 2 and 10 are GND, and pins 3–9 are P1 = PA1, P2 = PA4, P3 = PA6, P4 = PA7, P5 = PF0, D1 = PA9 and D2 = PA10 (functions, modes and commands in doc 09 §2.5). SPI uses P1 (SCK), P3 (MISO) and P4 (MOSI); the UART uses P2 (TX) and P5 (RX); P1–P4 read as analog inputs whenever their peripheral is off. Each signal has 1 kΩ in series and an ESD diode. The ESP32 reaches it over I²C, so it is for slow sensors only.

**Peripheral use: no bit-banged interfaces.** Every interface runs on an ESP32-S3 peripheral:

| Interface | Peripheral | Notes |
|---|---|---|
| CAN | TWAI | |
| I²C | I²C 0 | Sensor headers, on-board XGZP, carrier I/O controller |
| LAN | SPI2 on the IO MUX pins | GPIOs 10–13, to the W5500 module socket, and the module's interrupt on GPIO 38 (D72) |
| LED chain 1 (bed strip, or the hub panel) | **Carrier I/O controller TIM3 PWM with DMA** (PA5, D70, D71) | The controller renders the stream from colour and pattern commands sent over I²C (doc 09 §2.1). The ESP32's RMT drove it with DMA before D70 |
| MCU programming (carrier I/O controller, sensor-board MCUs) | **UART2** on GPIO 8/9 through the GPIO matrix, open-drain, 115200 8E1 | Only while programming; the I²C driver is detached for that time (doc 09 §4) |
| Servo, heater PWM, fan A PWM | LEDC | Three frequencies, inside LEDC's four timers. B1 PWM is a timer on the carrier I/O controller (D68) |
| Heater-enable charge pump | **Plain GPIO, toggled by the safety task** | Deliberately not a peripheral. A hardware PWM keeps running if the safety task hangs, which would defeat layer L5 (doc 05 §3.1, doc 07 §2) |
| Analog inputs (GPIOs 1, 2, 4, 5, 7 on node-heat; 1, 2, 5, 7 on the other roles) | **ADC1 continuous mode with DMA** | Scans every ADC1 input at a fixed rate, so the heater-current samples (GPIO 4) are evenly spaced without the CPU timing each read. ESP-IDF doesn't mix one-shot and continuous mode on one ADC unit [E], so all ADC1 inputs move to continuous mode together |
| Hub smoke dry contact (GPIO 36), hub acknowledge button (GPIO 6) (AUX) | **GPIO edge interrupts** | A press or an alarm is seen at once, not at the next poll. The button is debounced in software. Neither depends on I²C |
| UART printer link on a node (GPIOs 36, 6, AUX; D48) | **UART1** | Only with `link: marlin_serial`, `link_port: uart` (doc 12 §3.2) |
| Fan tach | PCNT | |

**RMT budget:** none since D70 (LED chain 1 is on the carrier I/O controller); all 4 TX channels are free for a second LED chain later.

**LAN module socket (D18, D45, D72):** the W5500 module needs four SPI GPIOs (SCK, MOSI, MISO, CS) and its interrupt on GPIO 38. They are **GPIOs 10–13 on every carrier**, the S3's IO MUX pins for SPI2, so the SPI runs on the peripheral directly (up to 80 MHz) and not through the GPIO matrix (40 MHz, with extra delay on MISO) [E: confirm in the S3 technical reference manual]. CAN and the servo output use the matrix and moved to GPIOs 41, 42 and 21 to make room. There are no 0 Ω links, and the same map applies to all roles. The default SPI clock is 20 MHz; the W5500's own limit is higher, and bring-up checks 20 and 40 MHz with the module on the socket [E: a plug-in module adds lead length]. The interrupt keeps the driver from polling the chip; the module's own RC resets it.

The KSD301 and charge-pump chain works in **hardware**. The carrier I/O controller only *reports* its state; the firmware can't override it.

## 5. Printer USB link (D3, D24)

Every Tier 2 printer (doc 12 §3.2) uses the same link; only the printer end differs, and that's in its printer pack.

| Item | Specification |
|---|---|
| Node end | The carrier's **USB-C** receptacle, wired to the module's native USB (D72). |
| Lead | USB 2.0, **D+, D−, GND and shield only. VBUS is left unconnected and insulated at both ends**, so the printer never powers the node and the node never back-feeds the printer (ARG-08). |
| Enumeration | The node appears as a USB CDC-ACM device, so the printer's kernel needs `cdc_acm`. The plugin opens `/dev/serial/by-id/usb-fleet_exhaust_node_<serial>-if00`, or finds the node by VID/PID when `serial:` is unset (doc 07 §3.1). |
| Carrier off, USB powered | The carrier's USB-C has no VBUS connection (D72), so a lead with VBUS can't power or back-feed anything; the ESP32 runs only from the carrier's own supply. The no-VBUS lead stays the first layer |
| VBUS-less enumeration | The ESP32-S3 USB device works with the board powered by the carrier and no VBUS on the cable [U: confirm on the prototype]. |
| Printer end rules | Prefer an **external** USB port. Internally, plug into a header (solder pins into a footprint's holes if needed), never wires onto pads (P-5). An internal hub board is a fallback only. |
| Ground | Shield connects at the printer end only. When the node heater runs from the printer's supply, part of the heater return shares this lead's ground (R11, §6). |

| Printer | Printer end |
|---|---|
| AD5X | Bottom internal USB header ([pack §4](../printers/flashforge_ad5x/README.md#4-usb-link)) |
| C5 Pro | External USB port, J34 fallback ([pack §4](../printers/flashforge_c5pro/README.md#4-usb-link)) |

## 6. Grounding rules

1. **The hub PSU's −V stays floating** (not bonded to earth), so the fleet logic ground has no path to mains earth of its own.
2. **Heater current stays in its printer cell.** It leaves the AD5X supply on the 14 AWG lead and returns on it. **No heater current flows in the CAT6** any more.
3. **USB-link ground share (R11):** each node's ground reaches its printer twice, via the heater return lead and via the USB GND, so part of the heater return can flow in the USB lead. With a 14 AWG return ≤ 1.5 m the share is ≲ 0.3 A [E], within a USB lead's rating, with a ground offset of ~50 mV. Measure it at commissioning; if it's higher or the link misbehaves, fit a USB isolator (ADuM3160-class) on the link.
4. **Between printers:** if the stock AD5X supplies bond −V to earth, the two printers' grounds also meet through mains earth and through the CAT6 chain. Only a small share of heater current can take that long path [E]. The offset stays far inside CAN's common-mode range (−2 to +7 V). The USB isolator fallback breaks this path too.
5. Each carrier joins logic GND and power GND at **one star point** near the input connectors. The heater current-sense shunt sits on the power side of that point.
6. Shields: CAT6 is unshielded (UTP). The USB lead's shield connects at the AD5X end only.
7. **Injected current** (§6a) returns over the CAT6 GND conductors (pins 7/8; pair 3 is the presence loop since D52) to the loads it feeds. Each source is limited by its eFuse, which keeps every conductor inside the ~0.75 A allowance.

## 6a. Bus power injection (D13)

| Item | Rule |
|---|---|
| Sources | Selected per board by the **INJECT jumpers** (INT, BAR), each through its own diode (INT has two, one on each side of the tap ideal diode, D64; since D80 the heater-rail one is an ideal diode and the others SS56 Schottkys), then the input guard, into the eFuse. Hub: INT (its PSU on the internal XT60, always). Node-heat: INT (the AD5X supply tap or an upgraded internal supply), or a DIN-4 or barrel heater brick in the HEATER DIN or HEATER barrel jack (D52, D64). Any node: BAR with a 24 V brick or PoE splitter. |
| Protection | Each source enters the bus through an eFuse with reverse-current blocking, so no supply is back-fed by another, and a short is current-limited at the source. |
| Load sharing | Diode-OR by voltage. Every source runs at its factory 24 V (no trim, D79), so at each point the highest one carries its stretch of the chain, up to its eFuse limit, and cable drop under load shifts load toward local sources. No sharing controller is needed. |
| Printer supply protection | If a printer's rail sags (bed + heater load), it falls below the bus voltage and its injection stops by itself, so the bus never takes power from a printer that needs it (R12). The ~1 A limit caps the most it can ever add to the printer supply at ~24 W. |
| Printers off | Their injection disappears. Idle nodes draw ~1–2 W each, which the hub covers alone, so nodes can still purge powered-off printers. |
| Sizing rule (D35) | Size for the worst moment: **every printer off (a power cut, or all cut in a fire) with the busiest node at its alarm peak** (servo moving, LEDs flashing). Hub PSU ≥ idle load of the whole chain + that peak + the peak of nodes on their own bricks only if they share the hub's outlet. Any single CAT6 segment ≤ ~1.5 A (two +24 V conductors, D17; deliberately not raised, D21). Three nodes: ~2 W idle each plus ~10 W for one alarm peak [E], well inside both. Checked by commissioning test 16 (doc 05 §9). |
| Bus voltage (D64) [E] | Since D80 the INT injection path drops ≈ 0.12–0.17 V at 1 A and ≈ 0.25 V at 1.5 A (ideal diode ≈ 0.02–0.03 V, guard, eFuse and its blocking FET; ≈ 0.7 V and ≈ 0.95 V with the Schottky before), and BAR ≈ 0.4 V more through its SS56. So the bus sits at ≈ 23.5–24.1 V at the hub (LRS-100-24, 24 V ±1 %), ≈ 23.75–23.9 V at a node injecting from its own 24.0 V supply on INT, and ≥ ≈ 22.7 V at a far node fed by the hub alone at an alarm peak (≥ ≈ 22.2 V on a ±3 % brick, ≥ ≈ 21.75 V on ±5 %), after ≈ 0.8 V of cable and per-node IN/OUT losses (≈ 1.1 V before D80's OUT switch FETs). INT sources lose the same drop, so sharing among them follows the supplies' own voltages; BAR sources sit ≈ 0.4 V lower and take over later. Every board measures its own bus voltage (doc 02 §2.2) and reports it with its injected current. The bucks run down to 4 V and the eFuse and buck turn-on points sit at ≤ ≈ 18 V, so only the bus-fed fans notice (B2, and fan A and B1 on the bus leg, D78): they run ≈ 1–9 % slower than at 24 V, which the characterise run measures as installed (doc 05 §3.2). |
| Hub supply lost (D35) | The hub's injection eFuse blocks reverse current, so its dead supply is never back-fed. The nodes whose printers are on carry the whole chain and the hub through the OUT switches (§3a); the hub enters back-fed mode (doc 07 §11.7). Load then: ~0.35 A idle, up to ~0.85 A in an alarm [E], against ~1 A per injecting node. |

## 6b. Power configurations (D35)

Jumpers only choose what **feeds** the bus; every board's logic runs from the bus. Only the heater inputs (internal XT60, HEATER DIN, HEATER barrel; one at a time) feed the heater; the POWER IN barrel never does (D64). Whenever the heater rail is live it also feeds fan A and B1; otherwise they run from the bus (D78). The rule that matters: **a node must have some source that survives its own printer's cut**: the CAT6 bus from the hub, or a brick on an ordinary outlet.

| Setup | INJECT | Outlet rules |
|---|---|---|
| Hub | INT (its PSU on the internal XT60, or a brick in a HEATER jack, D78) | Hub supply and the fan on ordinary outlets, **never on a printer's plug** |
| Wired node-heat, printer supply | INT | Logic survives the cut through the bus |
| Wired node-heat, heater brick (HEATER DIN or HEATER barrel jack) | INT | Heater brick **on that printer's plug** (doc 05 §9 test 23) |
| Wired node-lite | none, or BAR with a brick | Any brick on an ordinary outlet |
| Wireless-only node-heat (AD5X) | BAR (logic brick) + INT | **Two supplies:** logic brick in POWER IN on an ordinary outlet, heater supply on INT. The logic brick is never on the printer's plug |
| Wireless-only node-lite | BAR | Logic brick on an ordinary outlet |

A node that stays up when its heater supply drops out (on the bus or a barrel logic brick) sees the drop directly and doesn't reset with it (doc 05 §3.3).

Checked at commissioning (doc 05 §9 test 29) and continuously by the hub: when a printer's plug reads off in normal use, its node must stay online, or the hub alerts (doc 05 §4).

## 7. Hub wiring summary

| Connection | From | To |
|---|---|---|
| Mains | IEC inlet (fused, switched) | LRS-100-24 L/N/PE (PE to the PSU case and the inlet earth only) |
| 24 V logic | PSU +V/−V, lead with an XT60 | Carrier internal XT60 (INJECT INT fitted), CAT6 power via carrier |
| Buzzer | Carrier B2 | Active 24 V buzzer |
| Panel | LED1 → WS2812 ×2; acknowledge = the carrier's on-board tact switch (GPIO 6, AUX B) through the C9 ACK plunger, SERVICE plunger over the carrier's BOOT button (D49); sensor header 4 (switch channel 3) → optional OLED (D48) | Panel |
| Trunk ΔP | Sensor header 2 (switch channel 1) | Sensor board P (XGZP6897D), tubes to the trunk tap and room |
| Trunk flow (D30) | Sensor header 3 (switch channel 2) | Sensor board P (D33), tubes to the venturi inlet and throat taps |
| Low-flow anemometer (D32) | NTC 1 (GPIO 1), NTC 2 (GPIO 2), B1 (carrier I/O controller PA8, D68) | C18 probe: reference NTC at the venturi inlet, heated NTC + heater resistor in the throat |
| Filter ΔP | On-board XGZP barbs | Filter housing inlet and outlet taps |
| Room sensor | Sensor header 1 (switch channel 0) | Sensor head |
| Fan | Fan A (+ adapter per R1) | G4 control port |
| Smoke | AUX A (GPIO 36) | Optional relay-output smoke detector (dry contact) |
