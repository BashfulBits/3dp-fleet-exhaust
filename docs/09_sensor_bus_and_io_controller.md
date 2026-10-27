# 09 - Sensor Bus and Carrier I/O Controller

> Design baseline, 2026-10-27 (D45). How the carrier reaches up to six external sensor boards and one on-carrier I/O expander. It covers per-header power and interrupts, the I²C bus layout, and how the ESP32 programs every CH32V003 in the system from one pin. Phase 2 builds it after the bench test in O15.

Evidence key, as in the other docs: **[V]** verified from a datasheet, reference manual or released source; **[E]** engineering estimate; **[U]** unverified, needs a test.

---

## 1. What it adds

| Function | How |
|---|---|
| **Up to 6 external sensor headers** per carrier (was 3), plus an **internal seventh slot** for an on-carrier I/O expander (§2.4) | JST-XH 5: 3.3 V, GND, SDA, SCL, INT/SWIO |
| **Each header on its own I²C segment** | TCA9548A 8-channel switch replaces the TCA9543A (§3) |
| **Per-header power switching** | One P-FET per header, on by default (§2.2) |
| **Per-header interrupt** | The INT/SWIO pin of each header goes to a true EXTI pin on a carrier CH32V003 |
| **In-system programming of every CH32V003** (blank, running or bricked) | The carrier CH32 is the SWIO programmer for its headers. The ESP32 programs the carrier CH32 over **one GPIO**, which is also the carrier CH32's interrupt line back to the ESP32 (§4) |

A sensor board with a CH32V003 on it (D42, used as an ADC, I/O expander, light sensor or I²C endpoint) is programmed through the header it is plugged into, with no programming pads and no separate programmer. A sensor board without a CH32 just leaves the fifth pin as its interrupt output, or unconnected.

**The carrier CH32 is optional.** If it isn't fitted, every header is powered (the gate pull-downs hold the FETs on) and the ESP32 still reads every sensor through the switch. Only per-header interrupts, power cycling and CH32 programming are lost.

## 2. Carrier CH32V003

### 2.1 Pin plan (TSSOP20 / QFN20: 18 I/O, all used)

The CH32V003 SWIO protocol is pulse-width coded. The data-1 low time is T to 4T, the data-0 low time is 6T to 64T, and the target answers a read by stretching the host's short pulse [V: QingKeV2 debug manual]. The released host code generates pulses with **timer PWM + DMA** and reads them with **timer input capture + DMA** [V: rv003usb `bitbang_rvswdio_ch32v003.h`, ch32fun]. So every header line must sit on a pin that can be both a timer output and a timer capture input. It must also be a separate EXTI line, so each header keeps its own interrupt. The pin plan meets both rules for seven header slots; slot 7 is internal (§2.4). The eighth EXTI line (5) has no free timer pin left.

| Header | INT/SWIO pin | Timer use when programming (remap) | EXTI line | P-FET gate |
|---|---|---|---|---|
| **1** | PD0 **and** PC3, joined on the board | out: TIM1 CH1N (RM00); in: TIM1 CH3 capture | 0 (PD0) | PA2 |
| **2** | PA1 | TIM1 CH2 out (RM00); capture IC1 ← TI2 | 1 | PC0 |
| **3** | PD2 | TIM1 CH1 out (RM00); capture IC2 ← TI1 | 2 | PC4 |
| **4** | PD3 | TIM2 CH2 out (RM00); capture IC1 ← TI2 | 3 | PC5 |
| **5** | PD4 | TIM2 CH1 out (RM00); capture IC2 ← TI1 | 4 | PD5 |
| **6** | PC6 | TIM1 CH1 out (RM01); capture IC2 ← TI1 | 6 | PD6 |
| **7** (internal, §2.4) | PC7 | TIM1 CH2 out (RM01); capture IC1 ← TI2 | 7 | PD7 |
| — | PC1 / PC2 | I²C1 SDA / SCL (default remap): the carrier CH32 is an I²C target on the ESP32's upstream bus | — | — |
| — | PD1 | The carrier CH32's **own SWIO**, also its interrupt output to the ESP32 (§4.1) | — | — |

Timer remap tables, I²C remap and EXTI port rules: [V: CH32V003 reference manual, tables 7-8 and 7-9; AFIO_EXTICR]. PD7 is a GPIO unless the NRST option byte is set [V: datasheet], so the option byte stays at its default.

**Single-pin mode [U].** Headers 2–7 use one pin for both output and capture. The pin is an alternate-function open-drain output, and its input path stays live. The reference manual says that in this mode the Schmitt trigger is on and the pin is sampled into the input register [V]. The capture channel then takes the **neighbouring** channel's input. CC1S = 10 maps IC1 on TI2, and CC2S = 10 maps IC2 on TI1 [V: RM, TIMx_CHCTLR1]. No released programmer uses this arrangement, so it's the subject of the O15 bench test. Header 1 has two pins joined (output and capture on different channels), which is how the released code works, so header 1 is the known-good fallback for the bench test.

**Remap at run time.** Headers 2–3 and 6–7 need different TIM1 remaps. A remap only affects pins set to alternate function, and only one header is programmed at a time, so the firmware sets the remap at the start of each programming session [E].

**If single-pin mode fails O15:** fall back to five slots (one is the internal expander, so four external), each with two joined pins on the default remaps. The pairs are PA1→PC4, PA2→PD2, PD0→PC3 (TIM1), and PD3→PD4, PC0→PD7 (TIM2). The other pins become the five gates. The PCB is laid out for the seven-header plan, with the five-header pairs reachable by 0 Ω links [E].

**DMA:** I²C1 TX and RX share DMA channels 6 and 7 with TIM1 CH3 and TIM2 CH2/CH4 [V: RM DMA request map]. So the carrier CH32's I²C target is **interrupt-driven, not DMA**.

### 2.2 Per-header power

- **P-FET** (AO3401 class, SOT-23, ≈ $0.02 [E]) from the carrier's 3.3 V to the header's 3.3 V pin, with a **gate pull-down** (100 kΩ [E]). With the carrier CH32 in reset, blank or absent, its pins float and every header is **on**. The CH32 drives the gate high to switch a header off.
- **Pull-ups go after the FET:** SDA, SCL (on that header's switch channel) and INT/SWIO (**2.2 kΩ**, for SWIO edge speed, §4.1a [E]) all pull up to the header's switched 3.3 V. A header that is off is not back-powered through its pull-ups.
- **Firmware rule:** before switching a header off, the ESP32 deselects its switch channel, and the carrier CH32 sets that header's INT/SWIO pin to input. Otherwise SDA and SCL from the powered side would feed the unpowered board through its I/O clamp diodes.
- Every header line gets the same ESD protection as the carrier's other signal inputs (doc 02 §2.2).

### 2.3 Interrupt aggregation

Each INT/SWIO pin is an EXTI input with its own line. Sensor boards drive it open-drain, active low. On any edge, the carrier CH32 sets a bit in its pending register and pulls PD1 low, which is the ESP32's single interrupt line. The ESP32 reads and clears the pending register over I²C, then services the header that is flagged.

### 2.4 On-carrier I/O expander (slot 7)

Slot 7 is wired like an external header (gated 3.3 V, its own switch channel, INT/SWIO on PC7 and EXTI 7) but its connector is a CH32V003 on the carrier, with no cable. It is programmed through the carrier CH32 with the flow in §4.2, so it needs no programming pads of its own. Its job is to free ESP32 GPIOs so that **every role has the same pin map** (doc 04 §4) and the LAN footprint is a population option on any role, not a hub-only one with 0 Ω links:

- **Takes:** the door reed, the KSD301 loop sense, the 24V_HEAT switched-output sense, the board status LED (doc 02 §2.4). LED chain 2 is dropped from the fixed map.
- **Frees:** GPIOs 13, 21, 41, 42 and 47. CAN TX/RX and the servo output (all GPIO-matrix peripherals) move onto 41, 42 and 21. That releases GPIOs 10–13, the S3's IO MUX pins for SPI2, to the LAN SPI (SCK, MOSI, MISO, CS), so it runs without the GPIO matrix (doc 04 §4). The fifth pin, 47, becomes the carrier CH32's SWIO/INT line (§4.1).
- **Interrupt:** the door reed changes the slot-7 INT line, so a door event reaches the ESP32 in a few milliseconds through the aggregation in §2.3.
- **Failure behaviour:** nothing it carries feeds a safety layer (S-7). A missing or dead expander blanks the door input (the AD5X falls back to Moonraker), the KSD sense and the status LED. A pre-heat check that reads the 24V_HEAT sense **blocks heating** when the expander is absent. That is the safe direction.
- **Costs:** one CH32V003 (≈ $0.30 with passives [E]), one fewer external header (six), and one more firmware image (a small I²C register map). The part is the same as the carrier CH32's, so there is no extra JLC fee.
- **Dependency on O15:** slot 7 uses single-pin SWIO capture. If O15 fails and the five-slot fallback is used, the expander takes one slot and four stay external. The reference build needs at most three external headers.

## 3. I²C bus load: one switch channel per header

The carrier CH32 **does not** carry I²C. SDA and SCL run from the ESP32. Putting seven cables and boards on one segment would load it with roughly 7 × ~45 pF [E] plus the carrier's own load, which is too close to the 400 pF limit, even at 100 kHz. So each header gets its own segment:

| Upstream (ESP32 I²C, on the carrier) | TCA9548A channel → header |
|---|---|
| ESP32 SDA/SCL with the carrier's pull-ups; on-board XGZP6897D (0x6D); TCA9548A (0x70–0x77); carrier CH32 I²C target | Channels 0–6 → headers 1–7; channel 7 spare |

- **Load per segment:** one cable (≤ 1 m), one sensor board and one switch channel. That's well under 400 pF at 100 kHz [E].
- **Address clashes go away:** two boards with the same fixed address (two XGZP6897D at 0x6D, two SHT40 at 0x44) can be on any two headers.
- **Firmware selects one channel at a time.** Enabling several channels at once adds their loads together and joins their address spaces.
- **Isolation during power cycling:** a deselected channel cuts SDA/SCL to a header that is switched off (§2.2).
- **Stuck bus:** a sensor that holds SDA low only blocks its own channel. The ESP32 deselects it and switches its power off and on through the carrier CH32.
- **Alternative:** the ESP32-S3's second I²C controller could split the headers into two groups. That costs two more GPIOs, which the node-heat carrier doesn't have (doc 04 §4), and still shares a segment between headers. The TCA9548A is preferred.
- **Cost:** the TCA9548A is ≈ $0.5–1 [E], in place of the TCA9543A (≈ $0.5 [E]).

## 4. Programming every CH32V003 from one ESP32 pin

### 4.1 The carrier CH32 (ESP32 → PD1)

- **The ESP32 drives SWIO with its RMT peripheral, not a bit-banged loop [U].** One RMT TX channel (open-drain) and one RMT RX channel share the line, which is how Espressif's `onewire_bus` driver shares one pin [E]. The details are in §4.1a. The CPU is free while a frame is on the wire, and no interrupts are held off, so a node keeps running its control loops during an update. The released ESP32-S2 programmer is the timing reference, but it bit-bangs with `nop` loops [V: cnlohr esp32s2-cookbook]. **Fallback if the RMT test (O15) fails:** that bit-banged code, run from a high-priority task with a short critical section per frame, and only while no heater is on (programming is rare and doesn't need to be fast).
- The **same pin** is the carrier CH32's interrupt output. **Firmware rule:** after reset, the carrier CH32 keeps its SDI (debug interface) **enabled** until the ESP32 writes an "arm INT" command over I²C. Only then does it set SWCFG = 100 to free PD1 as a GPIO [V: RM AFIO_PCFR1: the pin is always SWD after system reset]. A carrier CH32 whose firmware hangs before it is armed stays programmable. While it is unarmed, the ESP32 polls the pending register instead of waiting for the interrupt.
- **Reprogramming a running carrier CH32:** the ESP32 sends a "reboot" command over I²C, then sends SWIO halt requests repeatedly until the debug module answers. This is the same sequence minichlink uses to unbrick a part [V: minichlink `DefaultUnbrick`].
- **Last resort:** a carrier CH32 that disables SDI and stops answering I²C needs a power cycle of the whole carrier, or a WCH-LinkE on three pogo pads (3.3 V, GND, PD1). The pads are on the carrier for that reason. A guard in the CH32V003's 1920-byte boot area could remove this case [U].
- **ESP32 GPIO:** GPIO 47 (doc 04 §4), freed by moving the board status LED to the I/O expander (§2.4).

### 4.1a SWIO on the ESP32's RMT

**Why the RMT can do it.** A general single-wire protocol is hard on the RMT, because nothing in hardware turns the line around from transmit to receive, and re-arming the receiver from software takes microseconds. SWIO needs no turnaround: **the host starts every bit slot**, including the 32 data bits of a read. For each read bit the host sends a short low pulse and releases the line. The target leaves it high for a 1, or stretches the low for a 0 [V: QingKeV2 debug manual §2.1–2.2; rv003usb host code]. So the RX channel is armed **before** the TX channel starts, and it records every slot on the wire, the host's own pulses included. The firmware decodes only the data-bit slots of a read and ignores the rest. Neither channel needs the CPU mid-frame.

**Frame format [V: debug manual §2.1].** A "new packet" is a start bit (1), a 7-bit register address, a read/write bit (1 = write), 32 data bits (MSB first) and an optional even-parity bit, followed by the stop condition. The stop is a sustained high, not a bit slot. A "bypass packet" is a start bit of 0 plus 32 data bits, reusing the last address. One packet is at most 42 slots, and each slot is one RMT symbol (one low time plus one high time). A whole packet fits in one channel's 48-symbol block, so there is no refill interrupt and no DMA.

**Timing [V: debug manual §2.2].** The debug interface starts in **normal mode (2×)** after any bus reset (low > 256T):

| Element | Normal mode 2× (default after reset) | Fast mode 1× |
|---|---|---|
| Data 1 | low T–4T, high T–16T | low T–2T, high T–8T |
| Data 0 | low 6T–64T, high T–16T | low 4T–32T, high T–8T |
| Stop | high ≥ 18T | high ≥ 10T |

T is the target's debug-interface clock period. The manual gives that clock as 8 MHz on the CH32V003 (HSI 24 MHz ÷ 3), so T ≈ 125 ns. The released host code is consistent with that: its 1-bit low is ≈ 290 ns (2.3T) and its 0-bit low ≈ 890 ns (7T) [V: rv003usb]. **Stay in normal mode** and leave margin inside each window. The firmware never switches to fast mode, since the margins there are half as wide.

**Proposed symbols** (RMT at 40 MHz, 25 ns per tick [E]):

| Symbol | Low | High | Normal-mode check |
|---|---|---|---|
| Write 1 (and start) | 300 ns (2.4T) | 700 ns | T–4T ✓; high T–16T ✓ |
| Write 0 | 1.0 µs (8T) | 700 ns | 6T–64T ✓ |
| Read slot | 300 ns (2.4T), then released | 1.5 µs (12T) | Released high stays below the 18T stop ✓ |
| Stop | (last slot's low) | ≥ 4 µs (32T) | ≥ 18T ✓ |
| Bus reset | ≥ 40 µs low (320T) | — | > 256T ✓ |

- **Read decision:** a read-slot low under ≈ 600 ns is a 1, and a longer one is a 0. The released code uses the same split (583 ns) [V: rv003usb `reply_buffer[i] < 28` at 48 MHz]. The RX channel's idle timeout ends the capture on the stop condition, at about 3 µs, which is between the 1.5 µs slot high and the 4 µs stop.
- **Unbrick "halt hammer":** the halt-request writes (§4.2 step 3) can go out as an RMT loop transmission, so the ~10 µs repeat interval needs no CPU timing either [E].
- **Pull-up:** **2.2 kΩ** on GPIO 47's line, and on every header's INT/SWIO line (§2.2). The CH32's weak internal pull-up, or 10 kΩ on a cable, rises too slowly: a slow rise after a short "1" pulse lengthens the measured low and can push it past the 600 ns threshold [E]. The rise time is ≈ 0.85 × R × C: about 110 ns for 2.2 kΩ with 60 pF, against about 510 ns for 10 kΩ.
- **ESP-IDF details [U]:** the TX channel needs open-drain output and a loopback path, so that both channels see the pad. The flag names (`io_od_mode`, `io_loop_back`) and the channel set-up order have changed between ESP-IDF releases, so check them against the version used in Phase 2.
- **Note on the timing source:** an earlier draft of this plan used the fast-mode windows with T = 250 ns. That conflicts with both the manual (normal mode is the default) and the released code's timings. The table above follows the manual.

### 4.2 Sensor-board CH32s (carrier CH32 → header)

One flow works for a blank, running or bricked part:

1. The ESP32 sends the image to the carrier CH32 over I²C in 64-byte blocks, matching the CH32V003's fast page program [V: RM flash chapter]. The carrier CH32 has only 2 KB of SRAM, so it streams the image instead of buffering it.
2. The ESP32 deselects that header's switch channel. The carrier CH32 switches the header off (≈ 300 ms [V: minichlink power-off time]) and then on again.
3. While the target boots, the carrier CH32 sends halt requests every ~10 µs until DMSTATUS reads valid [V: minichlink].
4. The carrier CH32 erases, writes and verifies the flash through the debug module, using the released host sequence, and then resumes the target.

**Sensor-board firmware rule:** keep SDI enabled for a boot window of a few milliseconds before turning PD1 into the INT output. The carrier CH32 controls the power-up time exactly, so a short window is enough. A part that is factory-blank has SDI enabled with no code to disable it, so it needs no window at all.

**Sources:** rv003usb, ch32fun and minichlink are permissively licensed (MIT) [E: confirm each at Phase 2 before copying code].

## 5. Sensor board changes (amends D42)

- The connector becomes **JST-XH 5**: 3.3 V, GND, SDA, SCL, INT/SWIO. The first four pins keep their order.
- On a board with a CH32V003, the fifth pin is its PD1 (SWIO, and INT after the boot window). On a board without one, it's the sensor's own interrupt output (open-drain) or left unconnected.
- The pull-ups stay on the carrier, after the header FET (§2.2). The sensor board still has none.

## 6. Cost per carrier [E]

| Part | Qty | ≈ Cost |
|---|---|---|
| CH32V003F4P6 (TSSOP20) | 1 | $0.20 |
| TCA9548A (replaces the TCA9543A) | 1 | +$0.40 |
| AO3401-class P-FET + gate pull-down | 7 | $0.20 |
| Pull-ups (3 per header), ESD arrays | — | $0.40 |
| On-carrier I/O expander: CH32V003 + 100 nF + LED resistor (§2.4) | 1 | $0.30 |
| JST-XH 5 headers (4 more than before; all go from 4- to 5-pin) | 7 | $0.35 |
| **Total** | | **≈ $1.85** |

External headers 4–6 and their FETs can be left unfitted on a role that doesn't need them.

## 7. Open points

- **O15** (doc 01 §7): bench test of single-pin SWIO capture and of TIM1 CH1N as an SWIO output. This includes checking CH1N's polarity and dead-time settings, and whether IC3/IC4 can be mapped across like IC1/IC2 [U]. It also confirms that the I/O expander on slot 7 (§2.4) is programmable, and that the ESP32's SWIO line works on GPIO 47 through the RMT peripheral: TX and capture on one open-drain pin, against a blank and a running target, in normal mode with the §4.1a symbols, and with a scope check of edge times through a 1 m header cable. The same session tests the W5500 at 20 and 40 MHz on GPIOs 10–13.
