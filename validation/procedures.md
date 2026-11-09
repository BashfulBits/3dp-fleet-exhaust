# Validation Procedures

> **Living procedure** (see [README](README.md)). Each test says what it proves, what it runs on, the equipment beyond a powered board and the dashboard, the steps and the pass criteria, and which open items it closes. Edit a procedure when a run shows it is wrong or unclear, and say so in the run's result file. Test IDs never change; a dropped test is struck through, not reused.
>
> **Test mode** is the firmware's mode for injecting faults and signals (doc 07 §8): it injects faults and signals that can't be made safely by hand. **Probe header** is the 1.27 mm ESP32 header fitted on bring-up boards (doc 04 §4a).

---

## P · Power and bus

### VAL-P1 Board supply and fan feed (was test 35, D94)
*Run on:* every bring-up board, each revision. *Equipment:* bench supply, a 24 V brick, a board-powered fan on B2 where the role has one, meters on each input.

A bench supply on the HEATER barrel and a brick on POWER IN. Meter the current into each.
1. Set the bench supply 1 V above the brick: the board, the bus share and the fan draw from POWER IN, and the bench supply carries only what the heater takes.
2. Unplug POWER IN with the fan at full: the board doesn't reset, the heater rail takes over, and the fan keeps running. Plug it back in: the load moves back to POWER IN. Scope the bus during the change-over: it must not dip below the eFuse turn-on point (closes the doc 02 §2.2.1 [U] on the 20 V margin).
3. Fan A and B1 run from the bus with the heater supply off; on a node-heat the firmware turns both off with no stall logged, and on a node-lite role they keep running.
4. Short fan A's output through a 0.1 Ω load for 5 s: the 0.75 A fuse opens, the bus and the chain stay up, and it recovers once the short is removed. Repeat on B1.
5. With POWER IN unplugged and the board on the bus, raise the bench supply to 36 V: the input guard keeps it off the bus, the heater stays off, and "wrong supply on heater input" arrives.
6. On a hub fed by a brick in the HEATER barrel, unplug the brick: the "hub supply lost" push arrives (as VAL-P6). Repeat with the brick on POWER IN and with a supply on the internal XT60.
7. With a tap live, plug the highest-voltage brick the jacks accept into a jack (current-limited bench supply standing in for it): record the reverse voltage on the tap ideal diode's FET against its rating (doc 02 §2.2.1; the FET choice itself is a schematic decision, doc 15 §2).

*Pass:* every step as written.

### VAL-P2 Bus power sharing (was test 16, D13, D35)
*Run on:* a chain of the hub and three nodes, each revision. *Equipment:* meter, 1 Ω power resistor, bench supply.

1. With all LEDs at full brightness and servos moving, read each node's bus voltage and injected current on the dashboard (D64); check every node against a meter. No source exceeds its limit, and no supply is back-fed. Switch a printer off: its node stays up from the hub.
2. Switch every printer's plug off and repeat the full-load step with one node in an alarm pattern (D35): the hub supply alone holds every node's bus above 21 V, the OUT window floor (doc 04 §3a; ≥ ≈ 21.75 V expected on a ±5 % hub supply, D80), and no segment exceeds ~1.5 A.
3. Short the bus at the far node through a 1 Ω load: every eFuse limits.
4. Inject 0.1–0.6 A from a bench supply through one eFuse and compare the IMON reading with the meter (closes the doc 02 §2.2.2 [U] below 0.6 A; record the error and whether the 90 % alert still works at that end).
5. Run step 2 as the installation's automatic bus load check (INS-5) and confirm it reports the same numbers (O19).

*Pass:* steps 1–3 as written; step 4 recorded; step 5 agrees within 0.1 V and 0.05 A.

### VAL-P3 RJ45 into network gear and wrong supplies (was test 21, D17, R10)
*Run on:* every revision. *Equipment:* laptop, PoE switch, 48 V passive injector, meter, oscilloscope.

1. Plug the hub's jack and a node's OUT jack into a laptop or switch port, metering pins 4/5 vs 7/8 first: **0 V**.
2. Repeat on a PoE switch port: neither side powers the other. Read the resistance between pairs 1/2 and 3/6 from the carrier: outside the 19–26.5 kΩ PoE signature band (doc 04 §3a).
3. Plug a node's IN jack into a passive 48 V PoE injector: the node stays off undamaged (over-voltage cut-off).
4. Hot-plug a 48 V brick into a live node at 24 V, scoping the guard's gate and the bus current: the guard opens within ≈ 10 µs and the node survives; repeat ten times (doc 02 §2.2.10 [U]).

*Pass:* steps 1–4 as written; no part warmer than its neighbours afterwards.

### VAL-P4 USB back-power and enumeration (was test 31, D72)
*Run on:* every bring-up board. *Equipment:* laptop, an ordinary USB-C cable with VBUS, a USB-C charger.

1. With the carrier unpowered, plug the cable from a laptop into the carrier's USB-C. Nothing starts (no USB device appears) and `+5V_PWR` and `+3V3` read < 0.3 V, since the receptacle's VBUS pin isn't connected.
2. Power the carrier: the ESP32 enumerates on the same cable as a composite device with two CDC interfaces, `-if00` (protocol) and `-if02` (log) (doc 07 §6.1).
3. Repeat step 2 through a cable with VBUS cut (the printer link's lead, doc 04 §5): it still enumerates.

*Pass:* as written. Closes the doc 04 §5 and doc 07 §6.1 [U]s.

### VAL-P5 A node survives its printer's cut (was test 29, D35)
*Run on:* one node of each power configuration (doc 04 §6b), each revision.

1. With the hub and every other printer on, switch that printer's plug off from the Shelly app. The node stays online and moves its damper to a commanded angle (encoder confirms).
2. Repeat for a wireless-only node.
3. Move that node's logic brick to the printer's plug (test only) and switch it off: the hub raises "node powered from the printer's plug" within 10 s.

*Pass:* as written. Steps 1–2 become the per-printer check in INS-6.

### VAL-P6 Hub supply loss (was test 30, D35)
*Run on:* a hub with one node-heat, each revision. *Equipment:* meter on the hub supply output.

1. With one node-heat printer on and idle, unplug the hub supply's mains. The hub stays up, the "hub supply lost" push arrives, and the dashboard shows back-fed mode.
2. Trigger a fire test (test mode) on another printer: siren and push come before its cut.
3. Switch the feeding printer off: the hub goes down, its klippy extra reports the link drop, and on restoring hub mains the hub boots and reconnects with no action.
4. Meter the hub supply's output throughout: it is never back-fed.

### VAL-P7 Automatic CAN termination and port LEDs (D62, D63, D96)
*Run on:* every revision. *Equipment:* meter, logic analyser on CAN.

1. One powered board with an empty OUT jack: ≈ 120 Ω between CAN_H and CAN_L. Plug a powered node into OUT: the first board's termination switches off, the far one's on (doc 04 §3).
2. Chains of 1, 2, 3 and 4 boards: every heartbeat reports exactly two terminations, and the hub alerts when a cable is pulled mid-chain until the chain settles.
3. Run a 24 h soak at full CAN traffic: error counters stay at zero.
4. Port LEDs: OUT port-live lights only with a node detected; IN link lights only with an OUT on the far end; the activity LED blinks at ≈ 30 ms on / 30 ms off on a busy bus and never stays solid (doc 02 §2.2.9 [U]: tune the RC values here).

---

## H · Heater safety chain

All H tests run on every node-heat bring-up board, each revision, with the heater in a test shroud and a thermocouple on the housing.

### VAL-H1 ESP32 in reset stops the heater (was test 1, L5)
Heat at 50 %, then hold the ESP32 in reset (EN low: the probe header's EN pin to GND, doc 04 §4a): the heater must stop. Scope the charge-pump output and the high-side gate: both collapse within the charge pump's hold-up time.

### VAL-H2 Static charge-pump pin stops the heater (was test 2, L5)
1. Hold GPIO 15 static high, then low (test firmware build): the heater must stop each time. Scope the time from the last edge to zero heater current.
2. Run the installation self-check's heater-chain step (INS-4: a short low-duty pulse with the charge pump stopped) on a good board: it passes. Lift the charge pump's coupling capacitor (rework log entry): the heater can't start, and the self-check reports it (O19).

### VAL-H3 KSD301 loop opens the chain (was test 3, D74)
1. Open the KSD301 loop at the housing thermostat, then at the chamber-air thermostat: the heater must stop each time; the ESP32 reports it from the loop sense on GPIO 48.
2. Heat the chamber-air KSD301 with a hot-air gun beside a thermocouple and confirm it opens near its rating, and the housing KSD301 likewise.
3. With the loop open, the INS-4 self-check reports "heater loop open" and refuses to heat (O19).

### VAL-H4 Welded FETs and the shunt zero (was test 4, L4, D50)
1. Command the heater off with a jumper simulating a welded low-side FET: the node turns the high side off and locks the heater out (L4).
2. With the heater off, tie the `24V_HEAT` node to the rail through a resistor to simulate a welded high side: the node locks the heater out within about a second and alerts, and the sense reads low again once the tie is removed and the node is cleared. Run the INS-4 self-check with the tie in place: it fails (O19).
3. Reboot with a bench supply injecting 3 A through the shunt (beyond the ±1.4 A the offset can explain), heater unplugged: the boot zero is refused and the heater stays locked out. With nothing injected, the zeroed reading is within ±0.1 A.

### VAL-H5 Runaway → mains cut (was test 5, RW-a, RW-b)
In test mode, inject "current with both switches off" plus a rising temperature: that printer's plug is cut within ~10 s, by the node with `shelly_node`, otherwise by the hub (RW-a). Repeat with two chamber sensors over 70 °C (RW-b). Repeat on each mains-cut backend.

### VAL-H6 Heater bench: temperatures, stall and FET drive (was test 11 and the doc 05 §3.2 procedure; O7, R8)
*Equipment:* thermocouples on the housing at the KSD301 mounting point and on the TCO, clamp meter, oscilloscope.

1. Run the chosen element at 100 % with normal airflow until steady. Record the housing temperature at the KSD301 mounting point (T_normal) and the steady current.
2. Block the fan. Record T_stall and the PTC current drop; the node must report the stall within 30 s (R8). If it doesn't, the C8 module is rejected and the fallback (bare PTC + 3-wire blower) is validated instead.
3. Set the ratings by doc 05 §3.2: KSD301 = T_normal + ~15 °C and below the nearest printed part's HDT; TCO = KSD301 + ~20 °C; firmware housing limit = T_normal + 10 °C. Record them in doc 02 §3 (closes O7).
4. Measure the low-side FET's resistance at the 3.3 V gate drive at full current (V_DS / I_D); above ≈ 12 mΩ, add the 5 V gate stage (doc 02 §2.2.10).
5. Measure the module fan's current (doc 02 §2.2.7, doc 05 §3.3).

### VAL-H7 Gain watchdog (was test 14, L3)
Disconnect the element (heater commanded with no rise) and confirm the L3 trip. Repeat after a characterise run (PQ-7) with its accepted gain-watchdog values.

### VAL-H8 Rail reading, throttle and learned supply cap (was test 8 in part; D64, D75, O18)
*Equipment:* bench supply with an adjustable current limit, meter.

1. **Untrimmed accuracy (O18):** on every bring-up board, at 20, 24 and 26 V and at room temperature and in a 50 °C box, compare the heater-rail reading with a meter. Record the spread.
2. **Throttle:** feed the tap from the bench supply and lower it through 23.0 V while heating: duty falls and the rail never goes below the throttle point by more than the reading error.
3. **Learned cap:** set the bench supply's current limit below the heater's draw: the supply drops out, the node sets the cap at 90 % of the proven level (half `heater_power` if none) and alerts; each further drop steps it down; Clear restores it.
4. **Boot inference:** cut the supply mid-heat with the node on the bench supply alone: at the next boot it infers the drop from the "heater on" flag.

*Closes O18:* untrimmed spread within ±0.2 V → the trim goes (doc 05 §3.3, D75 amended in a new D row). Otherwise propose an automatic trim against the bus reading.

### VAL-H9 USB ground share on the reference printer (was test 9, R11)
*Run on:* the reference node-heat printer (AD5X pack), heater fed from the printer's supply tap with a 14 AWG ≤ 1.5 m lead. *Equipment:* DC clamp meter or a shunt in the USB lead's ground.

1. Heat at 100 % with the bed at temperature: measure the current in the USB lead's ground. ≤ 0.3 A.
2. Run for 1 h: the link's error counters stay at zero.
3. Repeat with a 2.5 m lead to find the margin.

*Pass:* ≤ 0.3 A at the rule's lead (closes R11 for packs that follow the rule; they check link errors only, PQ-6). Over 0.3 A: the USB isolator becomes required on a shared supply.

### VAL-H10 Heater supplies (was test 23 in part; D22, D64, D94)
1. **Two-supply check** on a node-heat that uses the tap: plug a brick into an open jack as well (test only), first one about 1.5 V below the tap (a bench supply), then one above it: each time the node refuses to heat, or stops within a second if heating, and alerts "two heater supplies connected"; switch the printer's plug off: the printer's display stays dark (the brick doesn't back-feed it). Remove the brick: heating is allowed again.
2. **Brick on the wrong outlet:** with a heater brick on another outlet (test only), switch the printer's plug off: the node locks the heater out and alerts "heater supply not on the printer's plug".
3. **Shutter:** the C21 shutter leaves only one heater jack open, and covers both on a node that uses the tap.

The per-printer half, "the heater rail drops with the printer's plug", runs automatically in INS-6.

### VAL-H11 NTC inputs across their range (D97)
Replace each NTC with decade-box values for 10–70 °C (chamber air) and 20–120 °C (housing): every reading within the doc 02 §2.2.8 error budget after the rail scaling. Settle the pull-up value inside 20–27 kΩ.

---

## C · Carrier I/O controller and ESP32

### VAL-C1 ROM-bootloader programming and the LAN module (O15, D54, D72)
*Run on:* PY32F030 and PY32F003 dev boards and an ESP32-S3 dev board, **before carrier layout**; repeated on rev A. *Equipment:* logic analyser.

1. From the ESP32's GPIO 8/9 in open-drain UART mode at 115200 8E1: sync, Get, Read, Erase, Write and Go, against a blank part, a running part after a reset command, and a part after a watchdog reset (doc 09 §4).
2. With an XGZP6897D and the I²C switch on the same lines: neither answers or holds the bus during the traffic.
3. Repeat through a switch channel and a 1 m cable.
4. Capture PA9 and PA14 while the bootloader runs: record whether it drives them (doc 09 §2.1).
5. A W5500 module on its socket: link up and a 10-minute transfer at 20 MHz and at 40 MHz SPI.

*Closes O15:* all of 1–3 pass → the bootloader path stands; any fail → the SWD-pad fallback (doc 09 §4.5) before layout.

### VAL-C2 Controller hang and a stuck I²C bus (was test 32, D75)
1. While heating, hold the carrier I/O controller in reset (NRST to GND on its SWD header) for 2 min. The "carrier I/O controller silent" alert arrives, the heater keeps regulating, B1 runs at full duty and the I²C sensors keep reading. Release it: the ESP32 reprograms or restarts it and the alert clears.
2. Halt the controller with SDA held low: record whether its I²C peripheral times out by itself, and confirm the ESP32's bus-recovery clocking frees the bus (doc 02 §2.4).

### VAL-C3 Controller reset line (was test 33, D76)
1. With the heater off, run `IOC_RESET`: the controller restarts, the ESP32 runs the boot flow again and the alert clears within a few seconds.
2. Toggle the door input about 1000 times in a minute: the controller must not reset.
3. Halt the controller at an SWD breakpoint with `IOC_HOLD` clear: the ESP32 resets it once (not more than once per 10 min) and the alert is logged.

### VAL-C4 Expansion port (was test 34, D77)
1. Jumper P2 to P5 and run a UART loopback at 115200: the bytes sent come back.
2. Jumper P3 to P4 and run an SPI transfer: the bytes sent come back.
3. Apply 0 V and 3.3 V to each of P1–P4 and read both within the ADC tolerance.
4. D1 and D2 read high with nothing connected, including while the controller is held in reset with the reset line (VAL-C3).

---

## F · Firmware behaviour

Run on a hub and two nodes (one node-heat), on each firmware release.

### VAL-F1 CAN loss (was test 6)
Unplug CAN to a heating node with ESP-NOW disabled (test mode): the hub alerts within 10 s; the node turns its heater off at 30 s.

### VAL-F2 Hub loss (was test 7)
Kill the hub (hold its ESP32 in reset, EN low): heaters off at 30 s, and the printer `_NOTIFY` fires. The fan half of this test (the fan keeps running, F3) belongs to each fan model and is FQ-6.

### VAL-F3 ESP-NOW failover and radio range (was test 17, D16)
1. Unplug CAN from a heating node mid-print. Traffic continues on ESP-NOW, the heater keeps running, and a digest alert is raised.
2. Disable ESP-NOW too (test mode): the heater goes off at 30 s.
3. Node in its C5 housing on a damper beside a metal-framed printer: record RSSI to the hub and to its plug at 2, 5 and 10 m (doc 02 §2.1, R6). These numbers set the alert threshold INS-4 uses.

### VAL-F4 Latches survive a reboot (was test 19)
Reboot a node during a heater lockout (it must stay locked out) and during a purge (it must resume).

### VAL-F5 OTA (was test 20)
Push a deliberately bad image (rollback within 60 s) and try an update during a print (refused).

### VAL-F6 Router off (was test 22)
Switch the router off: the dashboard is reachable on `fleet-hub`, Shelly plugs rejoin the hub network, ESP-NOW carries on, and alarms show on the panel and LEDs.

### VAL-F7 Latched cut holds; level-4 Clear (was test 25, D27)
1. With a cut latched in test mode, switch the plug on from the Shelly app, then from Moonraker: it's cut again within 5 s each time with an Emergency push.
2. Confirm the plug's power-on behaviour is restore last state (D38) and Moonraker's `on_when_job_queued` is off, as the installation wizard sets them (INS-6).
3. Do a level-4 Clear: the dashboard step alone doesn't release the plug, the button step after 2 minutes doesn't either, and both in time do; the printer comes back at level 3 with the heater locked out.

### VAL-F8 Fire corroboration (was test 13, D39)
Inject two signals (test mode) and confirm the full response order, including the plug cut. Inject one signal and confirm it only alerts. On the reference node-lite printer (C5 Pro pack), repeat with the printer link unplugged: F-a falls back to the SHT40. Each pack with a native heater repeats that last step (PQ-9).

### VAL-F9 Door logic (was test 12, F1)
Open the door during a heated print, with each door source (node reed, link objects): damper to purge, alert at 60 s. Each pack checks its own door source (PQ-5).

### VAL-F10 Print hold (was test 26, D27)
On the reference node-heat printer: lock the node out, start a job with `M191`: the print pauses during the chamber wait with the reason on screen. `RESUME` runs it unheated (logged); `FLEET_CLEAR` then `RESUME` on a second job heats normally. A job without `M191` isn't held. Each pack repeats it on its own Klipper (PQ-8).

### VAL-F11 Purge queue (doc 06 §5.1, F-9)
End prints on two nodes within a minute of each other: one purge runs at a time, the waiting chamber stays at the heated setpoint until it gets the token, and the token passes on `PURGE_RELEASE`. Kill the token holder mid-purge (ESP32 reset): the hub reclaims the token and the waiting node purges.

### VAL-F12 Notifications (doc 07 §7, F-13, N-5)
Trigger one event of each priority in test mode: normal, high and emergency go out as pushes with their priority; slow trends go only into the daily digest. Leave a critical alert unacknowledged: after 15 min it escalates to the hub's panel and buzzer.

---

## S · Mains-cut plugs

Run once per backend (`shelly_node`, `shelly`, `tasmota_node`, `tasmota`, `ha`) and per plug model the docs name, on each firmware release.

### VAL-S1 Reachability, node-direct cut and fallbacks (was test 15, D14, D15)
1. Take a plug off Wi-Fi (and, with `ha`, stop HA): the hub raises "plug unreachable" within ~1 min.
2. Issue a test cut on the plug and confirm the read-back.
3. `shelly_node` and `tasmota_node`: switch the router off and confirm the node still cuts its plug (RW-a test mode).
4. Hold the node's ESP32 in reset and time how long the plug takes to rejoin its fallback network and accept a hub cut (doc 07 §10.1 [U]). Tasmota: time the switch back to AP1 when the node returns (doc 07 §10.3 [U]).
5. Read back the restore-after-power-loss setting the firmware writes (`PowerOnState`, Shelly `initial_state`) and confirm it holds after a mains blip (doc 07 §10.3 [E]).

### VAL-S2 Plug-local power limit (was test 18, D16)
1. On each plug, set the limit below a known test load and confirm the plug trips by itself; restore the limit and confirm the hub's read-back.
2. Record the exact fields or commands that worked on the device (Shelly `power_limit`, `current_limit`; Tasmota `MaxPower`, `MaxPowerHold`, `MaxPowerWindow`) against doc 07 §10.3 and §10.5.

### VAL-S3 BLE backup cut (was test 36, D102, O17)
1. Keep the plug off the node's AP (point its `sta` at a dummy network), then issue a test cut from the node: it must arrive over BLE, with read-back, inside the hub's 5 s window. Record the connect-to-off time.
2. Check whether the plug's HTTP authentication applies over BLE, and how long `BLE.StartPairing` stays open.
3. Restore the plug's `sta` and confirm the next cut goes over Wi-Fi again.

*Closes O17:* the default for `ble_backup` is decided from 1–2.

---

## A · Air path and sensors

### VAL-A1 Pressure-sensor offset and drift (was M4, R4)
*Run on:* all six XGZP6897D positions of a hub and three nodes, plus a sensor in a temperature box. Offset and noise in the static window; auto-zero stability over a day; offset vs temperature (doc 06 §4.1). *Sets:* the deadband, the Z4 limits and the plausibility limit; whether the venturi keeps the XGZP or gets an SDP31 (> ~1 Pa residual).

### VAL-A2 Damper bleed resolution (was M1a, D29)
*Run on:* the E3 damper (and any candidate from doc 08 §3b) in a test duct with a chamber box, **before the damper design is frozen**. Step the damper 1° at a time from closed while holding trunk ΔP; record encoder angle and chamber ΔP per step, and closed leakage. *Sets:* whether E3 holds −3 Pa smoothly or the design needs a finer mechanism.

### VAL-A3 Damper encoder and stall (was test 27, D30)
1. With the fan off, hold each damper blade while the servo is commanded to move: the node raises "damper stuck" within 10 s and stops the servo pulses: the servo goes limp and the node's 5 V current drops back. If a digital servo keeps holding without pulses, note it; that servo model isn't suitable, and doc 03 names only models that pass.
2. Loosen the horn screw so the horn slips: the node raises it too.
3. Unplug the encoder: the node falls back to ΔP-only detection and posts a digest item.
4. Run the INS-4 damper sweep with the horn loose and with the encoder unplugged: it fails each time (O19).

### VAL-A4 VOC purge criterion (was M6, R3)
On the reference printers: purge timing and VOC behaviour over ten ABS prints; cross-calibration of node and hub ENS160 at idle. *Sets:* the VOC end margin, or the fallback to temperature + time + air changes (doc 06 §5.3).

### VAL-A5 Low-flow anemometer: protection and calibration (was test 28 and M9, D32)
1. With the fan off, block the probe airflow and command full heater duty in test mode: the firmware holds the element at ≤ 80 °C and ΔT ≤ 30 K. Unplug the heated NTC: the heater turns off and a digest item is posted.
2. Static zero, then the 10 mm and 15 mm reference venturis in turn in the hub spool's calibration slot, fan stepped through its lowest speeds with one branch open; record reference flow vs element conductance from ~0.5 to 5 CFM, on three probes.
3. If the three curves agree within the doc 06 §4.4 accuracy, publish one **default curve** in the firmware and drop the reference venturis from installation (D104 proposal; an installation then runs only the automatic static zero). Otherwise keep a per-installation calibration and say so in doc 13.

### VAL-A6 Venturi spot check (was M8, spot check)
With one branch open at several flows, compare the venturi reading with a vane anemometer at the outlet. *Sets:* the venturi coefficient in firmware.

### VAL-A7 ModulAir parts: fit and leakage (O13 fit, first M10)
Print a Y, two hose adapters and a gasket to doc 08 §3c's settings. Assemble a test run with the hub spool, close every damper, hold the trunk at its setpoint and read the low-flow anemometer (VAL-A5 calibrated): < 0.5 CFM. Disturb and reseat each joint; repeat. *Closes:* the fit half of O13.

---

## X · Exhaust-fan port

### VAL-X1 B2 fan port (D95, D97, D98)
*Run on:* every revision. *Equipment:* oscilloscope, a 4-wire PWM fan, a 0–10 V fan or a resistor model of one (1.1 mA reference), clamp meter.

1. PWM on pin 4 at 0, 50 and 100 %: correct frequency and duty; tach read on pin 3 at the configured pulses per revolution.
2. CTRL buffer: high ≈ 9.4 V into 100 kΩ, ≈ 9.1 V at 1 mA, ≈ 7.9 V at 5 mA; short CTRL to GND for 10 s: the 10 mA limit holds and REF survives (doc 02 §2.2.7, D97 [U]).
3. Fails to full: with the board unpowered and with the ESP32 held in reset, CTRL sits at REF and pin 4 floats.
4. Each external jack (USB-C "FAN 10 V", TRRS, terminal header) carries REF, CTRL, TACH and GND to the right contacts, with SWAP straight and crossed; a laptop charger on the USB-C jack feeds only the CTRL buffer.
5. Fan tap: start a 2.2 A fan from cold ten times; scope the starting current against the tap eFuse's ≈ 2.7 A limit.

### VAL-X2 Mapping an undocumented fan control port (R1, D10)
*Run on:* one sample of each fan family with a proprietary controller, before anyone writes its fan pack. *Equipment:* logic analyser, oscilloscope, meter, the fan's own controller.

1. With the fan on its mains and its controller connected, meter every contact to the control ground: find the reference, the speed input and the tach. Confirm the control side is isolated from mains (SELV).
2. Capture the controller's signal at several speeds: PWM frequency and levels, or the 0–10 V level; tach pulses per revolution against an optical tachometer.
3. Disconnect the controller with the fan running, then power the fan with no controller: record whether it runs, stops or refuses to start (F3).
4. Connect the port to B2 through the matching jack or an adapter: speed follows the duty, and the fan runs at full speed with the hub held in reset.

*Result:* the pinout, levels and F3 behaviour go into the fan's pack (doc 14) as [V] facts, and R1 closes for that family. A family that fails step 3 or 4 is *not suitable*.
