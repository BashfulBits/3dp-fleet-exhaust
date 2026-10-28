# 09 - Sensor Bus and Carrier I/O Controller

> Design baseline, 2026-10-28 (D45, D54). How the carrier reaches up to eight external sensor boards through one I²C switch, with per-header power and interrupts, and how the ESP32 programs the carrier I/O controller and any sensor-board MCU through each part's ROM bootloader. Phase 2 builds it after the bench test in O15.

Evidence key, as in the other docs: **[V]** verified from a datasheet, reference manual or released source; **[E]** engineering estimate; **[U]** unverified, needs a test.

---

## 1. What it adds

| Function | How |
|---|---|
| **Up to 8 external sensor headers** per carrier, one per switch channel | JST-XH 5: 3.3 V, GND, SDA, SCL, INT |
| **Each header on its own I²C segment** | TCA9548A-compatible 8-channel switch (§3) |
| **Per-header power switching** | One P-FET per header, on by default (§2.2) |
| **Per-header interrupt** | Each header's INT pin goes to its own EXTI line on the carrier I/O controller (§2.3) |
| **Slow carrier I/O** | The same controller reads the door, the KSD301 and 24V_HEAT senses and the two heater-input ADC channels, and drives the board status LED (§2.4, doc 02 §2.4) |
| **In-system programming of every MCU** (blank, running or hung) | Each MCU's **ROM bootloader** on its own I²C pins, driven by the ESP32's UART through the GPIO matrix (§4). No programmer chip, no debug protocol and no extra ESP32 pin |

A sensor board with an MCU on it (D42: an ADC, I/O expander, light sensor or protocol bridge) is programmed through the header it is plugged into, with no pads and no separate programmer. A sensor board without one leaves the fifth pin as its sensor's interrupt output, or unconnected. The reference build's sensor boards S, P and E have no MCU.

**The carrier I/O controller is needed for heating, not for sensing.** If it is missing or not yet programmed, every header is powered (the gate pull-downs hold the FETs on) and the ESP32 still reads every sensor through the switch. Per-header interrupts, power cycling and the slow carrier I/O are lost, and since the heater-input readings come from it, **the heater is held off** (D47).

## 2. Carrier I/O controller (PY32F030)

### 2.1 Part and pin plan

**Part: PY32F030K28U6TR** (Puya, Cortex-M0+, QFN32 5 × 5 mm, 64 KB flash, 8 KB SRAM, 48 MHz rated, 12-bit ADC; LCSC C3018718, ≈ $0.41) [V: PY32F030 datasheet, JLC stock]. One part does what D45 split over two CH32V003s (the header controller and the slot-7 expander); D54 explains why. Pinout 2 (K2x) gives 28 I/O besides PF2-NRST and PF4-BOOT0 [V: datasheet fig. 3-3].

Three pin rules drive the plan:
- **I²C target on PA2/PA3.** PA2 is both I2C_SDA and USART1_TX, and PA3 both I2C_SCL and USART1_RX [V: datasheet pin table], and the ROM bootloader listens on that USART (§4.1). So the bootloader is reachable on the two I²C wires.
- **Bootloader pins carry only inputs or the status LED.** The bootloader also uses the USART on PA9/PA10 and PA14/PA15 [V: PY32F003 reference manual §3.6.2, same family], and may drive its TX candidates (PA9, PA14) while it runs [U]. Nothing on those pins may switch a header or signal the ESP32.
- **One EXTI line per interrupt.** EXTI lines are shared by pin number across ports, so the eight header INT lines and the door each take a different number.

| Function | Pins | Notes |
|---|---|---|
| I²C target (upstream) / bootloader UART | PA2 (SDA / TX), PA3 (SCL / RX) | **330 Ω in series with PA2** (§4.3) |
| Header INT 1–8 | PA0, PA1, PA4, PA5, PA6, PA7, PA8, PA11 | EXTI 0, 1, 4, 5, 6, 7, 8, 11 |
| Header P-FET gate 1–8 | PB2, PB3, PB4, PB5, PB6, PB7, PB8, PF0 | Gate pull-downs hold every header on while the pins float (§2.2) |
| Interrupt out to the ESP32 (GPIO 47) | PA12 | Open-drain, active low |
| Door reed | PA15 | EXTI 15; input only (a bootloader RX candidate, so harmless) |
| KSD301 loop sense, 24V_HEAT sense | PA10, PA9 | Inputs only |
| Heater-rail voltage, barrel heater-input sense | PB0 (ADC_IN8), PB1 (ADC_IN9) | 12-bit; ≈ 8 mV per step on the rail divider [E] |
| Board status LED | PA14 | If the bootloader drives PA14 as TX, the LED lights solid: a free "in bootloader" indicator |
| SWD test pads (bench debugging, §4.5) | PA13 (SWDIO), PA14 (SWCLK) | Pads only |
| Spare | PA13, PF1, PF3 | |

25 of 28 pins are used. The 32-pin part's other pinouts (K1x, K3x, K4x) move pins around, so the PCB uses K2x only [E].

### 2.2 Per-header power

- **P-FET** (HL3401 class, SOT-23, ≈ $0.04, a Preferred part, D51) from the carrier's 3.3 V to the header's 3.3 V pin, with a **gate pull-down** (100 kΩ [E]). With the controller in reset, in its bootloader, blank or absent, its pins float and every header is **on**. The controller drives the gate high to switch a header off.
- **Pull-ups go after the FET:** SDA, SCL (on that header's switch channel) and INT (10 kΩ [E]) all pull up to the header's switched 3.3 V. A header that is off is not back-powered through its pull-ups.
- **Firmware rule:** before switching a header off, the ESP32 deselects its switch channel. Otherwise SDA and SCL from the powered side would feed the unpowered board through its I/O clamp diodes. The controller masks that header's INT line while it is off, since the pull-up after the FET lets it fall.
- **Why every external header is switched:** the sensors on them (SHT40, ENS160, XGZP6897D, AS5600, an OLED) have no reset pin. One that latches up or holds SDA low after ESD or a brownout only recovers when its power is removed. Deselecting its channel isolates it but doesn't reset it. The cable is also the likeliest place for a fault.
- Every header line gets the same ESD protection as the carrier's other signal inputs (doc 02 §2.2).

### 2.3 Interrupt aggregation

Each header's INT pin is an EXTI input with its own line. Sensor boards drive it open-drain, active low. On any edge (and on a door change), the controller sets a bit in its pending register and pulls PA12 low, which is the ESP32's interrupt line on GPIO 47. The ESP32 reads and clears the pending register over I²C, then services whatever is flagged.

### 2.4 Carrier I/O (was the slot-7 expander)

The controller takes the slow, non-safety signals that would otherwise need ESP32 GPIOs, so every role has the same pin map and the same parts (D47), LAN included (doc 04 §4):

- **Takes:** the door reed, the KSD301 loop sense, the 24V_HEAT switched-output sense, the **heater-rail voltage** and the **barrel heater-input sense** (ADC), and the board status LED (doc 02 §2.4). LED chain 2 stays dropped from the fixed map.
- **Frees:** GPIOs 13, 21, 41, 42 and 47 (D45). CAN TX/RX and the servo output (all GPIO-matrix peripherals) use 41, 42 and 21, which leaves GPIOs 10–13, the S3's IO MUX pins for SPI2, to the LAN SPI (doc 04 §4). GPIO 47 is the controller's interrupt line.
- **Interrupt:** a door change reaches the ESP32 in a few milliseconds through §2.3.
- **Failure behaviour:** nothing it carries feeds a safety layer (S-7). A missing, silent or not-yet-programmed controller blanks the door input (the AD5X falls back to Moonraker), the KSD sense and the status LED, and removes the heater-input readings, so the firmware **holds the heater off** (and turns it off if the controller stops answering mid-heat). That is the safe direction.
- **No power switching of its own:** it is the carrier's own firmware, with no cable. A reset command over I²C or its hardware watchdog restarts it, and every reset lands in the ROM bootloader (§4.1), so it never needs a power cycle to recover.
- **It sits on the ESP32's main I²C bus,** not behind the switch. If it ever held SDA low, it would block the whole bus, which is why its firmware enables the watchdog at start-up, resets its I²C peripheral when the bus has seen no valid transaction for 1 s [E], and the ESP32 runs the usual nine-clock bus recovery before reporting a fault.

## 3. I²C bus load: one switch channel per header

The controller **does not** carry the sensor traffic. SDA and SCL run from the ESP32. Putting eight cables and boards on one segment would load it with roughly 8 × ~45 pF [E] plus the carrier's own load, which is too close to the 400 pF limit, even at 100 kHz. So each header gets its own segment:

| Upstream (ESP32 I²C, on the carrier) | TCA9548A channel → header |
|---|---|
| ESP32 SDA/SCL with the carrier's pull-ups; on-board XGZP6897D (0x6D); TCA9548A (0x70–0x77); carrier I/O controller (I²C target) | Channels 0–7 → headers 1–8 |

- **Load per segment:** one cable (≤ 1 m), one sensor board and one switch channel. That's well under 400 pF at 100 kHz [E].
- **Address clashes go away:** two boards with the same fixed address (two XGZP6897D at 0x6D, two SHT40 at 0x44) can be on any two headers. That is also why a sensor-board MCU never needs to translate addresses.
- **Firmware selects one channel at a time.** Enabling several channels at once adds their loads together and joins their address spaces.
- **Isolation during power cycling:** a deselected channel cuts SDA/SCL to a header that is switched off (§2.2).
- **Displays get their own channel (D48).** An OLED never shares a channel with a sensor; the default is header 4 (channel 3) on every role. The firmware selects the display channel only while it writes a frame (≤ 2 Hz) and deselects it straight after, so a display that locks up is selected as little as possible.
- **Stuck bus:** a sensor that holds SDA low only blocks its own channel. The ESP32 deselects it and switches its power off and on through the controller.
- **Alternative:** the ESP32-S3's second I²C controller could split the headers into two groups. That costs two more GPIOs. GPIO 5/6 are the only spare pair, and they are kept for the AUX header and its UART printer link (D48). It would also still share a segment between headers. The TCA9548A is preferred.
- **Cost:** a TCA9548A-compatible switch is ≈ $0.4–0.8 (FCA9548T fitted, D51).

## 4. Programming every MCU through its ROM bootloader

### 4.1 The mechanism

- **BOOT0 is strapped high** (10 kΩ to 3.3 V) on every PY32 in the system: the carrier I/O controller and any sensor-board MCU. The factory option byte has **nBOOT1 = 1** (production value `0x0155BEAA`, bit 15 set) [V: PY32F003 reference manual §4.4.1; E: the PY32F030 shares the layout, confirm in its manual], so BOOT0 high selects **system memory**: the factory bootloader, which can't be erased or overwritten [V: RM §3.6].
- So **every power-up and every reset lands in the bootloader**, and the part runs its application only after the ESP32 sends **Go** (0x08000000). A blank part, a running part told to reset, and a hung part reset by its watchdog all reach the same state. Nothing can brick it, short of the firmware writing the option bytes, which it never does.
- **Bootloader protocol** [V: puyaisp, MIT]: USART 8E1, 115200 baud (4800–1 M), sync byte 0x7F, ACK 0x79 / NACK 0x1F; commands Get 0x00, Read 0x11, Write 0x31, Erase 0x44, Go 0x21, Read-unprotect 0x92. It follows the STM32 USART bootloader pattern [E].
- **ESP32 side:** UART2 (UART1 belongs to the AUX printer link, D48), attached through the GPIO matrix to the **I²C pins (GPIO 8/9) in open-drain mode** only while programming: TX on SCL (→ PA3, RX) and RX on SDA (← PA2, TX). The I²C driver is detached for that time and reattached after. No extra pin, no RMT channels.

### 4.2 Boot flow, every ESP32 start

1. **Carrier controller first, all switch channels deselected.** The ESP32 sends the 0x7F sync on the UART.
   - ACK → the controller is in its bootloader. The ESP32 reads its image checksum (Read), writes the bundled image if it differs (Erase, Write, Read back), then sends Go.
   - No ACK → the application is already running (the ESP32 restarted on its own). The ESP32 returns to I²C, reads the controller's firmware version, and if it differs sends a "reset" command and starts again at step 1.
2. **Sensor-board MCUs,** one header at a time: select its switch channel, then the same sync / check / write / Go sequence through the switch. A header with no MCU simply doesn't answer the sync.
3. Only then do the power-input diagnostics run (D47). A new carrier takes a few seconds and needs no tools; heating is blocked until it finishes. If programming fails, the firmware raises **"I/O controller not programmed"**, kept apart from "heater rail absent", so a builder isn't sent looking for a supply fault.

The controller's image ships inside the ESP32's own firmware image, so an ESP32 OTA update (doc 07) carries the controller update with it.

**If Read is refused** (read protection active, which the factory doesn't set but a failed option-byte self-check would [V: RM §4.4]), the ESP32 sends Read-unprotect, which erases the flash, then writes the image as for a blank part.

**Mid-run resets.** A controller that resets while the ESP32 runs comes back in its bootloader, stops answering I²C, and the ESP32 sees that within one poll. It then runs step 1 again. The heater is held off until the readings return. A sensor-board MCU that resets only blocks its own channel; the ESP32 re-runs step 2 for that header.

### 4.3 Why the UART traffic is safe on the I²C bus [E]

The on-board XGZP6897D, the switch and (while running) the controller all see the UART traffic as I²C activity:
- The protocol is **half-duplex**: the ESP32 waits for each ACK before it sends again.
- While the ESP32 transmits, its data is on **SCL** and SDA stays high, so there is **no START** (SDA falling while SCL is high). Every I²C target ignores clocks without a START.
- While the bootloader replies, its data is on **SDA** and SCL stays high. That makes START and STOP conditions but **no clock pulses**, so no address is ever shifted in. Each reply ends with its stop bit, high, which reads as a STOP.
- So no target ever sees a valid address byte, and the switch never changes channel.

**Bus contention:** the bootloader drives PA2 push-pull, not open-drain. While the controller sits in its bootloader, an I²C low from the ESP32 on SDA would fight it. The **330 Ω on PA2** limits that to ≈ 10 mA and still lets the ESP32 pull SDA to ≈ 0.3 V, and the boot flow never does I²C on the main bus while the controller may be in its bootloader. Sensor-board MCUs sit behind a switch channel that is deselected whenever another header is in use.

**Through the switch:** the TCA9548A class is a set of pass FETs with no protocol logic, so a 115200-baud UART (8.7 µs bits against rise times of ≈ 0.4 µs with 10 kΩ and ~100 pF [E]) passes like slow I²C. The reference build has no MCU behind the switch, so this path only matters once a sensor board carries one.

### 4.4 Sensor-board MCUs

- Same conventions as the controller: **I²C target on PA2/PA3, BOOT0 strapped high, nothing that matters on PA9/PA10/PA14/PA15**. The default part is a **PY32F003F18P6TU** (TSSOP20, 64 KB / 8 KB, LCSC C5379864, ≈ $0.31) [V: datasheet, JLC stock]. It has one I²C; a board that relays to a sensor drives the sensor side in software.
- The fifth pin is the board's **INT** output (open-drain, active low) on any GPIO that isn't a bootloader pin.
- A board doesn't run on its own: after power-up it waits in its bootloader until the ESP32 sends Go (§4.2 step 2). That's fine here, because the ESP32 is the only host.

### 4.5 Bench debugging and fallback

The carrier keeps **four SWD test pads** (3.3 V, GND, PA13, PA14) for bench debugging with any CMSIS-DAP probe. They are also the fallback if the bootloader path fails O15: each controller is then programmed once by probe and updated over I²C by an application-level loader [E]. The PY32's SWD stays enabled unless firmware remaps PA13/PA14, and PA14 only drives the status LED.

## 5. Sensor board changes (amends D42)

- The connector is **JST-XH 5**: 3.3 V, GND, SDA, SCL, INT. The first four pins keep their order.
- On a board with an MCU, the fifth pin is its INT output (§4.4). On a board without one, it's the sensor's own interrupt output (open-drain) or left unconnected.
- The pull-ups stay on the carrier, after the header FET (§2.2). The sensor board still has none.

## 6. Cost per carrier [E]

| Part | Qty | ≈ Cost |
|---|---|---|
| PY32F030K28U6TR (QFN32) + 100 nF, BOOT0 pull-up, PA2 series resistor, status-LED resistor | 1 | $0.42 |
| TCA9548A-compatible switch (FCA9548T, D51) | 1 | $0.58 |
| HL3401-class P-FET + gate pull-down | 8 | $0.34 |
| Pull-ups (3 per header), single-line ESD diodes (H5VL10B class) | — | $0.23 |
| JST-XH 5 headers | 8 | $0.12 |
| **Total** | | **≈ $1.70** |

Every part above is fitted on every carrier, including headers 5–8 and their FETs (D47: one population, one assembly order). That is the same total as D45's two CH32V003s and seven headers: the second MCU, slot 7's FET, pull-ups and ESD go, and two external headers come in.

## 7. Open points

- **O15** (doc 01 §7): bench test of the ROM-bootloader path. On a PY32F030 and a PY32F003 board, from the ESP32's GPIO 8/9 in open-drain UART mode at 115200 8E1: sync, Get, Read, Erase, Write and Go, against a blank part, a running part after a reset command, and a part held in reset by a hung application and its watchdog. Also check: that the on-board XGZP6897D and the switch never respond during the traffic (§4.3); the PA2 contention current; whether the bootloader drives PA9/PA14; the same sequence through a switch channel and a 1 m header cable; and a scope check of edge times. The same session tests the LAN chip at 20 and 40 MHz on GPIOs 10–13.
