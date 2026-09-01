# LilyGo IoT Multi-Function PCB

A custom 4-layer KiCad PCB design integrating an ESP32 Wi-Fi/Bluetooth microcontroller with a SIM7600G-H 4G LTE Cat-4 cellular module, comprehensive power management, GPS tracking, GPRS connectivity, USB-C charging, LiPo battery management, environmental sensing, and USB-to-UART programming support.

---

## Table of Contents

- [Overview](#overview)
- [PCB Preview](#pcb-preview)
- [Schematic Overview](#schematic-overview)
- [Key Features](#key-features)
- [System Architecture](#system-architecture)
- [Schematic Sheets](#schematic-sheets)
- [Bill of Materials Summary](#bill-of-materials-summary)
- [Power System](#power-system)
- [Connectivity](#connectivity)
- [PCB Stackup & Design Rules](#pcb-stackup--design-rules)
- [Gerber & Manufacturing Files](#gerber--manufacturing-files)
- [Custom Libraries](#custom-libraries)
- [Datasheets](#datasheets)
- [Repository Structure](#repository-structure)
- [Getting Started](#getting-started)

---

## Overview

This project is a compact, feature-rich IoT development board built around the **ESP32-WROOM-32** and the **SIM7600G-H R2** 4G LTE module. It is designed for applications requiring cellular connectivity (4G/GPRS), GPS positioning, local Wi-Fi/Bluetooth, environmental sensing, and flexible power management from USB-C or a LiPo battery.

The board is designed in **KiCad** and targets JLCPCB / LCSC manufacturing and assembly (full BOM with LCSC part numbers included).

---

## PCB Preview

### Front Side (3D Render)

![Front 3D Render](PCB%20info/Front%203d.jpg)

### Back Side (3D Render)

![Back 3D Render](PCB%20info/back%203d.jpg)

### PCB Layout (Top View)

![PCB Layout](PCB%20info/Shem%201.jpg)

---

## Schematic Overview

The full schematic (all sheets combined) is available as:

- 📄 [`PCB info/schematic.pdf`](PCB%20info/schematic.pdf) — Full combined schematic (all sheets)

Individual sheet PDFs:

| Sheet | Description | PDF |
|---|---|---|
| Root | Top-level schematic, ESP32, USB-UART, interfaces | *(see full schematic)* |
| BUCK\_CONVERTER\_3V3\_2A | 3.3 V / 2 A step-down (TPS56339DDCR) | [`schematic-BUCK_CONVERTER_3V3_2A_TPS56339DDCR.pdf`](PCB%20info/schematic-BUCK_CONVERTER_3V3_2A_TPS56339DDCR.pdf) |
| LIPO\_BAT\_CHARGER\_TP4056 | LiPo charger and protection circuit | [`schematic-LIPO_BAT_CHARGER_TP4056.pdf`](PCB%20info/schematic-LIPO_BAT_CHARGER_TP4056.pdf) |
| MT3608\_5V\_BOOST\_CONVERTOR | 5 V boost converter (MT3608) | [`schematic-MT3608_5V_BOOST_CONVERTOR.pdf`](PCB%20info/schematic-MT3608_5V_BOOST_CONVERTOR.pdf) |
| POWER\_PATH\_SELECTOR | Automatic USB/Battery power path switching | [`schematic-POWER_PATH_SELECTOR.pdf`](PCB%20info/schematic-POWER_PATH_SELECTOR.pdf) |
| SIM808\_GPRS\_GPS\_MODULE | SIM7600G-H R2 4G/GPS module and support circuitry | [`schematic-SIM808_GPRS_GPS_MODULE.pdf`](PCB%20info/schematic-SIM808_GPRS_GPS_MODULE.pdf) |

---

## Key Features

| Feature | Details |
|---|---|
| **MCU** | ESP32-WROOM-32D (Wi-Fi 802.11 b/g/n + Bluetooth 4.2 / BLE) |
| **Cellular** | SIM7600G-H R2 — 4G LTE Cat-4 / 3G / 2G, global bands |
| **GPS / GNSS** | Integrated via SIM7600G-H R2 (GPS + GLONASS) |
| **USB Interface** | USB-C receptacle (GCT USB4105), USB 2.0 |
| **USB-to-UART** | CH340T (SSOP-20) for ESP32 programming |
| **LiPo Charging** | TP4056 — up to 1 A charge rate, with protection |
| **3.3 V Rail** | TPS56339DDCR buck converter, 2 A output |
| **5 V Boost** | MT3608 step-up converter for 5 V peripherals |
| **Buck-Boost** | TPS630250RNCR — efficient intermediate rail |
| **Power Path** | Automatic USB / battery selector (NCE6005AR P-FETs) |
| **Environmental Sensor** | AM2302 (DHT22) — Temperature & Humidity |
| **PIR Sensor** | AS312 passive infrared motion detector |
| **SIM Card Slot** | GCT SIM7155-6-1-14-A nano/micro SIM connector |
| **GPS Antenna** | U.FL / IPEX connector (GT-RF1125A-01G) |
| **GPRS Antenna** | U.FL / IPEX connector (GT-RF1125A-01G) |
| **Level Shifting** | RS0108YQ20 8-channel bidirectional level shifter |
| **ESD Protection** | SP0502BAHT (USB lines), ESD0504F (multi-line) |
| **Reset / Boot** | Dedicated tactile push buttons (SKRPACE010) |
| **Status LEDs** | Power, Charging, Network status, RGB indicator |
| **Crystal** | 12 MHz SMD crystal (Y1) for CH340T |
| **PCB Layers** | 4-layer stackup (F.Cu / In1.Cu / In2.Cu / B.Cu) |
| **Board outline** | Edge cuts defined; antenna keep-out zone present |

---

## System Architecture

```
                          ┌──────────────────────────────────────────┐
                          │              USB-C (J3)                  │
                          │         GCT USB4105-GF-A                 │
                          └───────────────┬──────────────────────────┘
                                          │ VBUS (5V)
                          ┌───────────────▼──────────────────────────┐
                          │       Power Path Selector                │
                          │  NCE6005AR P-FET switches (Q8, Q9)       │
                          │  Auto USB / LiPo selection               │
                          └────────┬──────────────┬──────────────────┘
                                   │              │
                    ┌──────────────▼──┐    ┌──────▼──────────────────┐
                    │  TP4056 LiPo    │    │   MT3608 Boost (U2)     │
                    │  Charger (U4)   │    │   VBAT → 5 V            │
                    └──────────────┬──┘    └──────┬──────────────────┘
                                   │ VBAT         │ 5V rail
                    ┌──────────────▼──────────────▼──────────────────┐
                    │           TPS56339DDCR Buck (U10)              │
                    │              → 3.3 V / 2 A                     │
                    └──────────────┬──────────────────────────────────┘
                                   │ 3.3 V
                    ┌──────────────▼──────────────────────────────────┐
                    │         ESP32-WROOM-32D (U6)                   │
                    │    Wi-Fi / BLE / GPIO / UART / SPI / I2C       │
                    └──────┬──────────────┬──────────────────────────┘
                           │ UART         │ I2C / GPIO
          ┌────────────────▼──┐    ┌──────▼────────────────────────────┐
          │   CH340T (U8)     │    │   RS0108YQ20 Level Shifter (IC1)  │
          │   USB-UART Bridge │    │   3.3V ↔ 5V level translation     │
          └────────────────┬──┘    └──────┬────────────────────────────┘
                           │              │
                     USB-C (J3)    ┌──────▼──────────────────────────────┐
                  (programming)    │     SIM7600G-H R2 (IC4)             │
                                   │  4G LTE / 3G / 2G / GPS / GNSS     │
                                   │  GPS Antenna (AE4) + GPRS (AE3)    │
                                   │  SIM Card (J1 — SIM7155)           │
                                   └─────────────────────────────────────┘
                                   
          ┌────────────────────┐   ┌──────────────────┐
          │  AM2302 Sensor     │   │  AS312 PIR (IC2)  │
          │  Temp + Humidity   │   │  Motion Detection │
          └────────────────────┘   └──────────────────┘
```

---

## Schematic Sheets

The schematic is organized into hierarchical sheets:

### 1. Root Sheet
Top-level sheet containing:
- ESP32-WROOM-32D microcontroller and decoupling
- CH340T USB-to-UART bridge with 12 MHz crystal
- RS0108YQ20 bidirectional level shifter
- Reset and Boot buttons
- AM2302 temperature/humidity sensor
- AS312 PIR motion sensor
- Status LEDs (Power, Charging, Network, RGB)
- ESD protection diodes

### 2. BUCK_CONVERTER_3V3_2A_TPS56339DDCR
- Texas Instruments TPS56339DDCR synchronous buck regulator
- Input: up to ~5.5 V (battery or boosted USB)
- Output: 3.3 V @ 2 A for ESP32 and logic
- Teardrop-optimized copper pours

### 3. LIPO_BAT_CHARGER_TP4056
- TP4056 single-cell LiPo/Li-Ion charger IC
- Charge current configurable via R-prog resistor
- Charging indicator LED (D8 — Red) and Done indicator (D24 — Green)
- SS54 Schottky diode for reverse protection
- 4.2 V Zener clamp (D28) for overvoltage protection
- Large bulk capacitors (220 µF electrolytic on BAT+ rail)

### 4. MT3608_5V_BOOST_CONVERTOR
- MT3608 2 A boost converter (U2, SOT-23-6)
- Boosts VBAT (~3.7 V) to 5 V for USB and module power
- Inductor: 1 µH power bead (L2)
- SS54 freewheeling diode

### 5. POWER_PATH_SELECTOR
- Automatic switchover between USB VBUS and LiPo battery
- P-channel MOSFETs NCE6005AR (Q8, Q9) in back-to-back configuration
- Prevents battery discharge when USB is connected
- Handles reverse current blocking

### 6. SIM808_GPRS_GPS_MODULE (SIM7600G-H R2)
- SIM7600G-H R2 global-band 4G LTE Cat-4 module (IC4)
- GPS/GLONASS integrated receiver
- GPRS antenna connector AE3 (U.FL / IPEX)
- GPS antenna connector AE4 (U.FL / IPEX)
- SIM card connector J1 (GCT SIM7155-6-1-14-A)
- TPS630250RNCR buck-boost for module power rail
- Large 220 µF tantalum bulk capacitors for surge current

---

## Bill of Materials Summary

> Full BOM: [`Manufacturing files/BOM.csv`](Manufacturing%20files/BOM.csv)  
> Schematic netlist CSV: [`schematic.csv`](schematic.csv)

### ICs & Modules

| Ref | Part | Description |
|---|---|---|
| U6 | ESP32-WROOM-32D-N4 | Wi-Fi + BT MCU module |
| IC4 | SIM7600G-H R2 | 4G LTE / GPS module |
| U4 | TP4056 | LiPo battery charger |
| U10 | TPS56339DDCR | 3.3 V / 2 A buck regulator |
| U2 | MT3608 | 5 V boost converter |
| IC3 | TPS630250RNCR | Buck-boost regulator |
| IC1 | RS0108YQ20 | 8-ch bidirectional level shifter |
| IC2 | AS312 | PIR motion sensor |
| U8 | CH340T (SSOP-20) | USB-to-UART bridge |
| U3 | AM2302 | DHT22 temperature & humidity sensor |

### Connectors

| Ref | Part | Description |
|---|---|---|
| J3 | USB4105-GF-A | USB-C receptacle (USB 2.0) |
| J1 | SIM7155-6-1-14-A | SIM card connector |
| AE3 | GT-RF1125A-01G | GPRS / LTE antenna (U.FL) |
| AE4 | GT-RF1125A-01G | GPS antenna (U.FL) |

### Semiconductors & Passives (Key)

| Ref | Value/Part | Description |
|---|---|---|
| Q8, Q9 | NCE6005AR | P-channel power MOSFETs (power path) |
| Q15, Q16 | SI2302 | N-channel MOSFETs |
| Q10, Q11 | MMBT3904 | NPN signal transistors |
| Q1 | BC817 | NPN transistor |
| D1, D17 | SS54 | 5 A Schottky diodes |
| D10–D12 | B5819W | 1 A Schottky diodes |
| D13 | BAT54C | Dual Schottky (SOT-23) |
| D28 | BZT52C4V3 | 4.3 V Zener clamp |
| D3 | SMLVN6RGBFU1 | RGB LED |
| D15, D16 | SP0502BAHT | USB ESD protection |
| D25 | ESD0504F | Multi-line ESD protection |
| Y1 | 12 MHz | SMD crystal (2520 4-pin) for CH340T |
| L1 | 22 µH | Chilisin power inductor (buck) |
| L2 | 1 µH | Chilisin power inductor (boost) |
| L5 | 4.7 µH | Chilisin power inductor (buck-boost) |
| C31, C35 | 220 µF | Electrolytic bulk caps |
| C23, C67 | 220 µF | Tantalum bulk caps (cellular surge) |

Total unique component lines in BOM: **~82 line items**

---

## Power System

```
USB-C VBUS (5 V)
     │
     ├──► Power Path Selector (NCE6005AR)
     │         │
     │    ┌────┴──────────────────────────────────────────┐
     │    │ When USB present: powers system, charges LiPo │
     │    │ When no USB: LiPo powers system               │
     │    └────────────────────────────────────────────────┘
     │
     ├──► TP4056 LiPo Charger ──► LiPo Battery (BAT+ / BAT-)
     │
     └──► MT3608 Boost ──► 5 V internal rail
               │
               └──► TPS56339DDCR Buck ──► 3.3 V (ESP32, logic, peripherals)
                         │
                         └──► TPS630250RNCR Buck-Boost ──► SIM7600G-H power rail
```

| Rail | Regulator | Max Current | Load |
|---|---|---|---|
| 5 V boost | MT3608 | 2 A | Module VCC, USB |
| 3.3 V | TPS56339DDCR | 2 A | ESP32, logic ICs, sensors |
| Module VCC | TPS630250RNCR | — | SIM7600G-H R2 |
| VBAT | TP4056 output | 1 A charge | LiPo cell |

---

## Connectivity

| Interface | Notes |
|---|---|
| **4G LTE / GPRS** | SIM7600G-H R2, global bands, SMA/U.FL antenna |
| **GPS / GNSS** | Integrated in SIM7600G-H R2, dedicated U.FL antenna |
| **Wi-Fi** | ESP32 802.11 b/g/n (PCB trace antenna on module) |
| **Bluetooth** | ESP32 BT Classic 4.2 + BLE |
| **USB** | USB-C, USB 2.0, via CH340T UART bridge for programming |
| **Temperature/Humidity** | AM2302 (DHT22) single-wire protocol |
| **PIR Motion** | AS312 digital output |
| **SIM** | Nano/micro SIM via GCT connector |

---

## PCB Stackup & Design Rules

| Parameter | Value |
|---|---|
| **Layers** | 4 (F.Cu, In1.Cu, In2.Cu, B.Cu) |
| **Min track width** | 0.2 mm |
| **Min clearance** | 0.129 mm |
| **Min via diameter** | 0.6 mm |
| **Via drill** | 0.3 mm |
| **Min annular ring** | 0.15 mm |
| **Min hole-to-hole** | 0.25 mm |
| **Min copper-edge clearance** | 0.5 mm |
| **Solder mask clearance** | 0.005 mm |
| **Track widths used** | 0.19, 0.25, 0.3, 0.6 mm |
| **Teardrops** | Enabled on PTH pads, SMD pads, and vias |

> ⚠️ **Antenna Keep-Out Zone**: A copper-free keep-out zone is defined above the ESP32 module and near the cellular/GPS antenna connectors. Do not route traces or place copper fills in this area.

---

## Gerber & Manufacturing Files

### Gerber Files (`Gerber/`)

| File | Layer |
|---|---|
| `schematic-F_Cu.gbr` | Front copper |
| `schematic-B_Cu.gbr` | Back copper |
| `schematic-In1_Cu.gbr` | Inner layer 1 |
| `schematic-In2_Cu.gbr` | Inner layer 2 |
| `schematic-F_Mask.gbr` | Front solder mask |
| `schematic-B_Mask.gbr` | Back solder mask |
| `schematic-F_Paste.gbr` | Front solder paste |
| `schematic-B_Paste.gbr` | Back solder paste |
| `schematic-F_Silkscreen.gbr` | Front silkscreen |
| `schematic-B_Silkscreen.gbr` | Back silkscreen |
| `schematic-Edge_Cuts.gbr` | Board outline |
| `schematic-PTH.drl` | Plated through-hole drill file |
| `schematic-NPTH.drl` | Non-plated through-hole drill file |
| `schematic-job.gbrjob` | Gerber job file |

> 📦 Pre-zipped package: [`Gerber.zip`](Gerber.zip)

### Manufacturing Files (`Manufacturing files/`)

| File | Description |
|---|---|
| [`BOM.csv`](Manufacturing%20files/BOM.csv) | Full BOM with LCSC part numbers for JLCPCB SMT |
| [`Gerber.zip`](Manufacturing%20files/Gerber.zip) | Gerber archive for board fabrication |
| `placement/schematic-all-pos.csv` | Component placement / pick-and-place file |

---

## Custom Libraries (`lib/`)

Custom KiCad footprint and symbol libraries used in this project:

| Library | Part |
|---|---|
| `LIB_AM2302` | DHT22 / AM2302 temperature & humidity sensor |
| `LIB_AS312` | AS312 PIR motion sensor |
| `LIB_AXP2101` | AXP2101 power management IC |
| `LIB_RS0108YQ20` | RS0108YQ20 level shifter |
| `LIB_SIM7080G` | SIM7080G NB-IoT / Cat-M module |
| `LIB_SIM7600G-H_R2` | SIM7600G-H R2 4G module |
| `LIB_SKRPACE010` | SKRPACE010 tactile switch |
| `LIB_TP4056` | TP4056 LiPo charger |
| `LIB_TPS630250RNCR` | TPS630250RNCR buck-boost regulator |
| `Antenna conector` | U.FL / IPEX antenna connector |
| `SIM7155_6_1_14_A` | GCT SIM card connector |

---

## Datasheets (`Datasheet/`)

| Document | Description |
|---|---|
| [`AXP2101.pdf`](Datasheet/AXP2101.pdf) | AXP2101 PMIC datasheet |
| [`T-SIM7080G_Schematic.pdf`](Datasheet/T-SIM7080G_Schematic.pdf) | LilyGo T-SIM7080G reference schematic |
| [`TDM-2202_TDM-2205_ESP32-X7600X_schematic.pdf`](Datasheet/TDM-2202_TDM-2205_ESP32-X7600X_schematic.pdf) | ESP32 + SIM7600 reference design |
| [`sim7600 schematic.pdf`](Datasheet/sim7600%20schematic.pdf) | SIM7600 series module schematic reference |
| [`tps630250.pdf`](Datasheet/tps630250.pdf) | TPS630250 buck-boost regulator datasheet |

---

## Repository Structure

```
lilygo/
├── README.md                    ← This file
├── schematic.csv                ← Component netlist export
├── Gerber.zip                   ← Fabrication-ready Gerber archive
│
├── PCB info/                    ← Visual references
│   ├── Front 3d.jpg             ← 3D render — front
│   ├── back 3d.jpg              ← 3D render — back
│   ├── Shem 1.jpg               ← PCB layout screenshot
│   ├── schematic.pdf            ← Full combined schematic PDF
│   ├── schematic-BUCK_CONVERTER_3V3_2A_TPS56339DDCR.pdf
│   ├── schematic-LIPO_BAT_CHARGER_TP4056.pdf
│   ├── schematic-MT3608_5V_BOOST_CONVERTOR.pdf
│   ├── schematic-POWER_PATH_SELECTOR.pdf
│   └── schematic-SIM808_GPRS_GPS_MODULE.pdf
│
├── schematic/                   ← KiCad project files
│   ├── schematic.kicad_pro      ← KiCad project settings
│   ├── schematic.kicad_pcb      ← PCB layout file
│   ├── schematic.kicad_sch      ← Root schematic sheet
│   ├── BUCK_CONVERTER_3V3_2A_TPS56339DDCR.kicad_sch
│   ├── LIPO_BAT_CHARGER_TP4056.kicad_sch
│   ├── MT3608_5V_BOOST_CONVERTOR.kicad_sch
│   ├── POWER_PATH_SELECTOR.kicad_sch
│   └── SIM808_GPRS_GPSS_MODULE.kicad_sch
│
├── Gerber/                      ← Individual Gerber layer files
├── Manufacturing files/         ← JLCPCB production package
│   ├── BOM.csv                  ← BOM with LCSC part numbers
│   ├── Gerber.zip
│   └── placement/
│       └── schematic-all-pos.csv ← Pick-and-place file
│
├── lib/                         ← Custom KiCad libraries
│   ├── LIB_AM2302/
│   ├── LIB_AS312/
│   ├── LIB_SIM7600G-H_R2/
│   ├── LIB_TP4056/
│   ├── LIB_TPS630250RNCR/
│   └── ...
│
├── Datasheet/                   ← Reference datasheets
└── placement/                   ← Placement CSV (root-level copy)
```

---

## Getting Started

### Prerequisites

- [KiCad 7 or later](https://www.kicad.org/download/) (project was created with KiCad 7/8)
- LCSC / JLCPCB account for ordering

### Opening the Project

1. Clone or download this repository.
2. Open KiCad and select **File → Open Project**.
3. Navigate to `schematic/schematic.kicad_pro` and open it.
4. The schematic editor and PCB layout editor will be available from the KiCad project manager.

> **Note**: Ensure the `lib/` directory is accessible to KiCad for custom footprints and symbols. If prompted, add `lib/` as a project-level footprint library path in **Preferences → Manage Footprint Libraries**.

### Ordering the PCB (JLCPCB)

1. Upload [`Gerber.zip`](Gerber.zip) (or [`Manufacturing files/Gerber.zip`](Manufacturing%20files/Gerber.zip)) to [JLCPCB](https://jlcpcb.com).
2. Select **4-layer** stackup.
3. For SMT assembly, also upload:
   - [`Manufacturing files/BOM.csv`](Manufacturing%20files/BOM.csv)
   - [`Manufacturing files/placement/schematic-all-pos.csv`](Manufacturing%20files/placement/schematic-all-pos.csv)
4. Review component availability and substitutions as needed.

### Programming the ESP32

The board includes an on-board CH340T USB-to-UART bridge. To program:

1. Connect a USB-C cable to **J3**.
2. The CH340T will enumerate as a serial COM port.
3. Use the Arduino IDE, ESP-IDF, or PlatformIO targeting **ESP32-WROOM-32**.
4. Hold **BOOT** button while pressing **RESET** to enter download mode if needed.

---

*PCB designed in KiCad. Manufactured via JLCPCB. All LCSC part numbers included in BOM for one-stop sourcing.*
