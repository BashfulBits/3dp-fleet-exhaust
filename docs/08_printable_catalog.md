# 08 - Printable Parts Catalogue

> Design baseline, 2026-10-25. Phase 2 work: verify each existing model (link, licence, fit to the real parts), then design the custom parts. **Policy:** existing models first; custom parts only where they're genuinely better.

---

## 1. Material rules

| Location / duty | Material | Why |
|---|---|---|
| Inside a heated chamber, on the heater module, or in exhaust air ≥ 50 °C | **ASA or ABS** (PC acceptable) | HDT ≈ 95–100 °C; PETG (HDT ≈ 65–70 °C) creeps at 55 °C (ARG-12) |
| Outside the chamber, near the warm enclosure wall (node enclosure, flange outer face) | ASA preferred; **PETG-CF acceptable** if it's a brand with a stated HDT ≥ 75 °C | Surface may reach ~40–45 °C |
| Room temperature (hub enclosure, filter housing, duct clips) | **PETG** or PETG-CF | Cheap, tough |
| Seals and gaskets | **TPU 95A** | |
| Light pipe | Clear PETG | |

All hot parts print on the **C5 Pro** (enclosed), using its internal filtration plus this system's exhaust once it's installed.

**CAD deliverables for custom parts:** parametric source (CadQuery) + **STEP** (imports into Fusion) + STL, in `hardware/cad/<part>/`. Remixes keep the original licence and attribution.

## 2. Assembly map

```mermaid
flowchart TB
  subgraph ad5x["AD5X ×2"]
    C1["C1 port module<br>+ C8 grommet"] --> E3a["E3 damper<br>+ C5 node housing"]
  end
  subgraph c5pro["C5 Pro"]
    C7["C7 shroud<br>(from E8)"] --> E3b["E3 damper<br>+ C5 node housing"]
  end
  E3a -->|"branch"| C14["ModulAir 2:1 Ys ×2<br>(D34)"]
  E3b -->|"branch"| C14
  C14 --> C11["C11 filter housing<br>+ C12 trunk ΔP mount"]
  C11 --> C17["C17 flow venturi"]
```

- **Also in each AD5X:** E1 kit parts (ASA), E2 IFS pass-through plate, E7 recirculation duct, C2 makeup inlet, C3 heater mount, C4 sensor-head cover, C6 door reed mounts.
- **Also on the C5:** C4 sensor-head cover.
- **On every damper:** C16 encoder mount (D30).
- **On the wall:** E5 duct clips with cable channel, C9 hub enclosure with C10 light pipe.

## 3. Existing models to use (core; verify in Phase 2)

| # | Part | Candidate source | Material | Verify |
|---|---|---|---|---|
| E3 | 100 mm motorised butterfly damper | Printables [#1023387](https://www.printables.com/model/1023387) (Tinkerer4000) | ASA, TPU edge seal | MG90S fit; blade clearance with a TPU edge seal; add one if missing (remix) |
| E5 | 4″ duct clips / raceway | Printables/MakerWorld "4 inch duct clip" (choose one with a cable channel) | PETG | Holds the bus cable alongside the duct |
| E6 | 5015 heater shroud (**only if C8 fails**) | Printables [#1472826](https://www.printables.com/model/1472826), [#1471103](https://www.printables.com/model/1471103) | ASA + aluminium liner | Element size; metal liner around the PTC |

**Removed from the old catalogue:**

| Old part | Why removed |
|---|---|
| Window sash plate (#151240) | The outlet is a through-wall vent hood (doc 06) |
| Drybox optical motion block | Drybox sensors deferred |
| C5 Pro USB hub bracket | Not needed: the C5 link uses its external USB port (D24). Verify and use it only if the J34 fallback is needed |
| C5 rear 4-hole flange | The C5's port is a perforated side port; replaced by C7 |
| ESP32 DIN shield enclosures (#1556383, #258431) | The hub now holds the PSU, inlet and carrier; replaced by C9 |

## 3a. Printer-pack parts

Parts that belong to one printer are defined in its printer pack (doc 12 §2). IDs stay as listed here.

| # | Part | Pack |
|---|---|---|
| E1 | AD5X enclosure kit printed parts | [AD5X §6](../printers/flashforge_ad5x/README.md#6-printed-parts) |
| E2 | IFS/Bowden pass-through plate | [AD5X §6](../printers/flashforge_ad5x/README.md#6-printed-parts) |
| E4 | AD5X exhaust flange (base for C1) | [AD5X §6](../printers/flashforge_ad5x/README.md#6-printed-parts) |
| E7 | AD5X recirculation duct + 5015 blower | [AD5X §6](../printers/flashforge_ad5x/README.md#6-printed-parts) |
| C1 | AD5X exhaust port module (port adapter) | [AD5X §6](../printers/flashforge_ad5x/README.md#6-printed-parts) |
| C6 | Door reed + magnet mounts | [AD5X §6](../printers/flashforge_ad5x/README.md#6-printed-parts) |
| E8 | C5 Pro exhaust-port adaptor (base for C7) | [C5 Pro §6](../printers/flashforge_c5pro/README.md#6-printed-parts) |
| C7 | C5 Pro side-port shroud (port adapter) | [C5 Pro §6](../printers/flashforge_c5pro/README.md#6-printed-parts) |

## 3b. Damper reference designs (D29; final choice at Phase 2 design)

E3 stays the baseline branch damper. These are recorded for comparison; the damper's mechanism, model and remix are decided at Phase 2 design time, after the bleed-resolution bench test (M1a, doc 06 §10).

| Ref | Design | What it is | Fit to our needs | Licence | Recommendation |
|---|---|---|---|---|---|
| E3 (baseline) | Printables [#1023387](https://www.printables.com/model/1023387) | 100 mm butterfly, MG90S | Native 100 mm and MG90S, so the C5 node housing and servo lead work as specified. Weak points: a butterfly passes most of its flow in the first ~20–30° of opening, so the −3 Pa bleed lives in a few degrees of an MG90S with ~1° deadband; a TPU edge seal may need adding for a tight close | [U: not yet checked] | **Keep as baseline.** Check the licence, and check bleed resolution on the bench (M1a); if it's too coarse, look at a sleeve-type design (below) or a smaller bleed blade |
| R1 | Nerdiy **PrintAirPipe servo valve** ([Hackaday project](https://hackaday.io/project/195522-printairpipe-3d-printable-air-duct-system), [servo valve log](https://hackaday.io/project/195522/log/229135-printairpipe-servo-valve), [160 mm on Printables](https://www.printables.com/model/1280629-printairpipe-160mm-servo-ventil-by-nerdiyde), [125 mm on Cults3D](https://cults3d.com/en/3d-model/tool/printairpipe-125mm-servo-valve-3d-druckbar-stl-dateien)) | Servo valve segment for Nerdiy's printable duct system: MG90 servo, its own ESP32 running ESPHome, touch button and open/closed LED. Sold as STL files | 125 mm and 160 mm (80 mm and 50 mm systems, with adapters to 100 mm); no native 100 mm valve found. The built-in ESP32 duplicates our node. Blade type and closed leakage not documented | Hardware **CC BY-NC-SA 4.0**, code **AGPL-3.0**; the Cults3D listing uses a personal-use licence | **Reference only; don't adopt.** NC and AGPL mean link-only under our licence policy (doc 12 §8), and we'd strip out half of it. Ideas worth taking: a local open/closed indicator on the damper, and their airflow-sensor segment as a design reference |
| R2 | MakerWorld **"Three way air diverter valve – 4 inch / 100 mm"** (Marcus_yallow, [#2528476](https://makerworld.com/en/models/2528476-three-way-air-diverter-valve-4inch-100mm)) | Manual rotary-core diverter: turning the inner core moves flow between two outlets, and part-turns throttle it continuously | Native 100 mm. A rotating sleeve uncovers port area roughly in proportion to rotation, which would give finer control near closed than a butterfly. Manual as published; a servo drive, the higher friction of a sliding seal (MG90S torque), and closed leakage are unknown | [U: not readable; MakerWorld blocked the request] | **Reference for an alternative mechanism** if E3's bleed resolution falls short. Check the licence before any remix |
| R3 | MakerWorld damper tee ([#2954045](https://makerworld.com/models/2954045)) | Described in search results as a three-way tee with built-in damper markings and a stepless manual cutoff | Possibly 100 mm; not confirmed | [U: page not readable] | **Unverified lead.** Open it at design time; treat like R2 |

## 3c. Duct interface: ModulAir (D34; extend at Phase 2 design)

The duct system builds on **[ModulAir](https://makerworld.com/en/collections/7342813-modulair-modular-air-system)**, a printed modular air system for 3D-printer exhaust. Its **Large (L)** size is 4″ / 100 mm and joins parts with a **QuickLock bayonet** (a 1/10 turn) sealed by a printed gasket. Every joint that is swapped, holds a sensor or sits at the printer end uses this interface; the long runs stay semi-rigid aluminium duct on ModulAir hose adapters.

**Existing ModulAir parts to use** [U: confirm sizes, print settings and licence, O13]:

| Part | ModulAir item | Use here |
|---|---|---|
| 2:1 Y | MA-L-Y-2L ([MakerWorld #2523640](https://makerworld.com/models/2523640)) | **Branch junction:** two in a chain for three printers (replaces the 4″ wyes and C14). 3:1 and larger don't fit a 256 mm plate |
| Gasket | MA-L-SEAL | On every joint (TPU) |
| 100 mm hose adapters | [MakerWorld #2376673](https://makerworld.com/models/2376673) (clockwise and counter-clockwise, match the duct spiral) | Ends of every flex-duct run |
| Inline fan / rigid pipe adapter | MA-L-PIPE-102 | Hub spool to the G4 fan |
| Bambu X1/P1 adapter | MA-L-BL-X1-P1 ([MakerWorld #2523279](https://makerworld.com/models/2523279)) | Reference for printer packs; a Bambu pack can use it as its port adapter |

**Extensions to design for this project** (ModulAir-L compatible, published as remixes under ModulAir's licence if it allows, otherwise as our own compatible interface):

| Extension | Replaces or extends | Notes |
|---|---|---|
| Port adapters with an L bayonet | C1 (AD5X), C7 (C5 Pro) | The printer-pack interface (doc 12 §5) |
| E3 damper with L bayonet ends | E3 | Remix; the C5 node housing and AS5600 mount (C16) stay on the damper body |
| Filter housing ports | C11 | L bayonets on inlet and outlet; the bayonet lid stays. The **trunk ΔP tap (C12)** sits on the inlet side, upstream of the cartridge, so the trunk loop never sees filter loading (doc 06 §4) |
| **Hub instrument spool** | C17, C18 | A straight run of L-bayonet sections between the filter and the fan: **calibration slot** (a plain spacer the length of a reference venturi), **venturi** with the anemometer and reference-NTC ports (C17, C18), then the fan adapter. Sensor boards (variant P, doc 02 §2.3) clip onto their sections |
| Reference venturis (10 mm, 15 mm) | C18 | Same length as the calibration spacer, so they swap in without re-cutting duct (doc 06 §4.4) |
| Blank cap | — | Closes an unused Y leg or port |

**Joint rules** (doc 06 §1.1): every bayonet joint gets its gasket; joints in the hub spool are checked in the duct leak test (M10); the joint between a reference venturi and the main venturi is foil-taped during calibration.

## 4. Custom parts (core)

| # | Part | Material | Key requirements |
|---|---|---|---|
| C2 | **Makeup inlet with gravity flap** | ASA (TPU hinge or pin hinge) | Low on the front or side panel; slotted grille; the flap opens under purge suction and stays mostly closed at −3 Pa; size set at commissioning (doc 06 §10) |
| C3 | **Heater module mount + intake filter cap** | ASA; sheet-aluminium shroud (DXF template) | Holds the integrated-fan PTC module (or the bare PTC + 5015 per E6) low at the rear, blowing along the floor away from the bed. ≥ 10 mm air gap between the aluminium shroud and any plastic. KSD301 + TCO clamp points on the shroud (doc 05 §3.2). The filter cap takes a cut carbon/HEPA pad on the intake side |
| C4 | **Sensor-head cover** | ASA | Vented, snap-on, M2 mount; places the sensor head at the exhaust port out of the heater jet, a short cable run from the node housing. **Also holds the chamber-air KSD301** in free air beside the head (D12) |
| C5 | **Node housing** | ASA | **Attaches to the E3 damper body** at the exhaust port, on both the AD5X and the C5 (D19), with a thermal break (air gap or standoffs) from the duct wall. Carrier + dev board. **Outside face:** 2 × RJ45, POWER IN barrel, DIN-4 opening for the DIN pigtail's panel jack (D52). **Printer face, toward the C8 grommet:** internal 24 V input XT30, heater-output XT30, JST row, servo, USB-C. **Port names embossed beside each opening** (IN, OUT, POWER IN, 24 V IN (INT), HEATER SUPPLY, HEATER OUT, USB, and "FLEET BUS – NOT ETHERNET" by the RJ45s; R10); a **2.5–3 mm window in the wall** in front of the carrier's port-live LED beside the OUT jack, so the side-view LED shows with no light pipe (D62; a short clear or white printed pipe is optional); optional C20 bezels; every opening takes a C15 blanking plug when unused; a captive printed **CLEAR plunger** over the dev board's BOOT button, recessed so it can't be pressed by accident and labelled CLEAR (doc 05 §4.1); ΔP barb routed through; vents; ESP32 antenna end facing away from the printer frame (R6) |
| C8 | **Cable grommet** | TPU | Split grommet in the C1 port module for the heater feed from the printer's supply tap, the heater lead, sensor cable, NTC leads, USB lead and ΔP tube; seals |
| C9 | **Hub enclosure** | PETG | Wall-mount; holds the LRS-100-24 (or a reused stock AD5X supply; vented per its spacing), IEC inlet module, carrier. Front panel: light pipe (C10), captive **ACK plunger** over the carrier's on-board acknowledge switch (D49), a recessed **SERVICE plunger** over the dev board's BOOT button (Wi-Fi provisioning and NVS reset, D46), labelled and recessed like the node's CLEAR plunger, buzzer grille, OLED window. The carrier mounts behind the front panel so both plungers land on their switches. Cutouts for the fleet RJ45 with a window for its port-live LED (D62), the LAN RJ45 (D18), the POWER IN barrel and DIN-4 openings (blanked unless used; the PSU feeds the internal XT30, D22), fan control lead, ΔP tubes, with **embossed port names** (R10). Mains section physically separated from low-voltage, with a cover |
| C10 | **Panel light pipe** | Clear PETG | Diffuses 1–2 WS2812s into a visible status lens |
| C11 | **Filter housing** | PETG (body), TPU (gasket) | Cylindrical, sized to the chosen combo cartridge (Levoit Core 300 class); ModulAir L bayonet inlet and outlet (D34); bayonet lid with TPU gasket, tool-free; **two barbed ΔP taps** (upstream and downstream of the cartridge); wall brackets |
| C12 | **Trunk ΔP sensor mount** | PETG | Clamps the sensor board P (XGZP6897D) to the trunk; short tube to a tap upstream of the filter; room-reference port |
| C13 | **Dev-board hold-down clip** | PETG | Retains the ESP32 board in its headers against vibration and cable pull |
| C14 | **Branch junction** | PETG | **Superseded by two chained ModulAir 2:1 Ys** (§3c, D34). A one-piece 3-into-1 or larger manifold doesn't fit a 256 mm build plate; two metal 4″ wyes remain the fallback |
| C15 | **Blanking plugs** (set) | ASA (node housings), PETG (hub) | Snap-in plugs for any unused opening in C5 or C9: DIN-4 (where no pigtail is fitted), POWER IN barrel, LAN RJ45, the OUT RJ45 at the end of the chain, the IN RJ45 on a wireless-only node. Each is embossed with its port name, keeps dust out, and stops an unused jack being mistaken for a free one. A small tether loop holds the plug on the housing while the port is in use (D22) |
| C16 | **Damper encoder mount** (D30) | ASA | Cap holding a 6 × 2.5 mm diametric magnet on the blade shaft's free end (opposite the servo), and a bracket holding the sensor board E (AS5600) 1–2 mm from it, centred on the shaft. Remix onto the chosen damper body (doc 08 §3b) |
| C17 | **Trunk flow venturi** (D30) | PETG | ModulAir L bayonet ends (D34), 50 mm throat, smooth bore, ring of 4 taps at inlet and throat with barbs; fits between the filter housing and the wall sleeve (doc 06 §4.3). Print in two halves along the axis or vertically in sections; sand the throat. Ports for the C18 probe in the throat (downstream of the throat taps) and for the reference NTC at the inlet |
| C18 | **Anemometer probe** (D32) | ASA | Plug for the C17 throat port that carries the heated element across the centreline on its leads, the body kept ≥ 10 mm from any plastic; a small plug for the reference NTC at the inlet. Also the **10 mm and 15 mm reference venturis** (PETG, L bayonet ends, taps as C17) used only for calibration (doc 06 §4.4) |
| C19 | **OLED bezel** (D48, optional) | PETG (hub), ASA (node) | Front bezel and clip for a 0.96″ I²C OLED module on a sensor-header cable (≤ 1 m), so a node's display can sit at the printer's front rather than on the housing at the exhaust port. The hub panel's OLED uses the C9 front panel. Look for an existing 0.96″ OLED case first (policy above) |
| C20 | **Fleet-bus port bezel** (D62, optional) | PETG (hub), ASA (node) | A thin snap-on ring around each fleet RJ45 opening (IN, OUT), printed as its own part so a single-colour printer can still make the housing and print only the ring in the bus colour (the colour of the fleet cable; blue suggested, to match the port-live LED), or leave it off. Not printed on the LAN RJ45, which keeps the housing colour. No multi-colour print needed |

## 5. Phase 2 verification checklist (per existing model)

1. The link resolves; record the exact URL and version.
2. The licence allows remixing and sharing (this project is open source).
3. Dimensions fit the real part (kit panel thickness, MG90S body, cartridge size, duct OD).
4. It prints in the required material without redesign.
5. Record the decision (use as is / remix / replace with custom) in this table.
