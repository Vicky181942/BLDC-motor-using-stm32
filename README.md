# BLDC Motor Controller

An open-source, STM32-based sensored brushless (BLDC) motor controller. Designed by **Vigneshwar G**, KiCad EDA project (v10.0.1), top-level sheet `BLDC_4.sch`, current revision **4.5**.

<img width="1141" height="700" alt="bldc 3D view" src="https://github.com/user-attachments/assets/f5129ba4-a1b3-488f-b85b-90715b0a0e02" />
<img width="580" height="761" alt="bldc pcb view" src="https://github.com/user-attachments/assets/9b5cbdae-5838-4255-b5ad-bcf935de6c8f" />
<img width="1067" height="752" alt="bldc schm" src="https://github.com/user-attachments/assets/c2084240-57ac-4054-a708-53a5bcd2b672" />

*Assembled prototype (M3/M4 mini-QFN driver + 3x MOSFET half-bridge layout, STM32 daughter area, connector header on top edge).*

---

## 1. Overview

This project is a 3-phase, hall-sensored BLDC motor driver built around an **STM32F4-series microcontroller (64-pin LQFP)**. It integrates:

- Gate-driver + power-MOSFET half-bridge stage for 3-phase commutation
- Hall-effect sensor inputs (with hardware filtering) for commutation feedback
- CAN bus interface for vehicle/system-level communication
- USB (Mini-B) interface for configuration/firmware/debug
- I²C motor/board temperature sensing
- Analog current sense and fault/over-current protection
- Servo-style PWM output and status LEDs (green/red)
- Wide input voltage range power stage with reverse/ESD protection notes on the input connector

The schematic is split into hierarchical sheets, which map directly to the project layout:

| Sheet file | Function |
|---|---|
| `BLDC_4.sch` | Top-level: interconnects all sub-sheets |
| `STM32F4_64LQFP.sch` | MCU: STM32F4 (LQFP64), all peripheral routing |
| `Power.sch` | Gate driver ("Mosfet driver") stage |
| `mosfets.sch` | Power MOSFET half-bridges ("Power MOSFETS") |
| `hall_filters.sch` | Hall sensor / encoder input filtering |
| *(USB sub-block)* | Mini-USB shield connector + ESD protection |
| *(CAN sub-block)* | CAN transceiver + bus connector (P101/CANBUS) |
| *(I²C temp sub-block)* | On-board I²C temperature sensor (PWR_COMM) |

---

## 2. Key Specifications

| Parameter | Value / Notes |
|---|---|
| MCU | STM32F4 family, LQFP-64 package |
| Input voltage (V_SUPPLY) | 0–60 V (per schematic caution note — see §5) |
| Phase outputs | 3-phase (PHASE_1, PHASE_2, PHASE_3) |
| Commutation feedback | 3x Hall sensors (HALL_1/2/3) + motor temp input |
| Current/voltage sense | 3x per-phase sense lines (SENS1–3), bus current sense (BR_SO1/BR_SO2), DC-CAL |
| Protection | Hardware `FAULT` line from gate driver stage |
| Communication | CAN (CAN_RX/CAN_TX), USB (USB_DM/USB_DP) |
| Auxiliary I/O | Analog input (AN_IN), external ADC, SERVO PWM output, status LEDs (green/red), I²C temp bus (SDA/SCL/ALERT) |
| Gate drive rails | Bootstrap/high-side supply per phase (H1_V5/H2_V5/H3_V5), low-side drive (H1_LOW/H2_LOW/H3_LOW) |
| CAD tool | KiCad EDA 10.0.1 |
| Board size | A4 schematic sheet |

---

## 3. Architecture / Signal Flow

```
                         ┌────────────────────┐
   V_SUPPLY (0–60V) ────▶│   Power Input       │
                         │  (decoupling caps,  │
                         │   reverse/EMI note) │
                         └─────────┬───────────┘
                                   │
      USB (Mini-B) ───┐            │             ┌── CAN (CANH/CANL) ── P101
      + ESD protect   │            │             │
                       ▼            ▼             ▼
                 ┌─────────────────────────────────────┐
                 │        STM32F4 MCU (LQFP64)          │
                 │  AN_IN, ADC_EXT, TX_SDA/RX_SCL,      │
                 │  HALL_1/2/3, TEMP_MOTOR,             │
                 │  USB_DM/DP, CAN_RX/TX,               │
                 │  EN_GATE, H1D/L1D..H3D/L3D,          │
                 │  SENS1-3, FAULT, BR_SO1/2, DC_CAL,   │
                 │  SERVO, LED_GREEN, LED_RED           │
                 └───────────┬───────────────────────────┘
                             │  gate drive commands
                             ▼
                  ┌─────────────────────┐
                  │   Mosfet Driver      │  (Power.sch)
                  │  EN_GATE, H1-3/L1-3, │
                  │  SENS1-3, FAULT,     │
                  │  BR_SO1/2, DC_CAL    │
                  └──────────┬────────────┘
                             │  H1_V5/H2_V5/H3_V5, H*_LOW, SH*_A/B
                             ▼
                  ┌─────────────────────┐
                  │  Power MOSFETS        │ (mosfets.sch)
                  │  M_H1/N_L1 ... M_H3/  │
                  │  N_L3 half-bridges    │
                  └──────────┬────────────┘
                             ▼
                 PHASE_1 / PHASE_2 / PHASE_3 → Motor

   Hall sensors / TEMP_IN ──▶ hall_filters.sch ──▶ MCU (HALL_1/2/3, TEMP_MOTOR)
   I²C temp sensor (SDA/SCL/ALERT) ──▶ MCU (PWR_COMM)
```

---

## 4. Bill of Materials (BOM) — fill in from your source BOM

| Ref / Block | Function | Part number | Datasheet |
|---|---|---|---|
| U_MCU | Microcontroller, LQFP64 | **STM32F405RGT6** *(confirm exact suffix/variant from BOM)* | [ST — STM32F405xx/STM32F407xx datasheet](https://www.st.com/resource/en/datasheet/dm00037051.pdf) |
| U_GATEDRV | 3-phase gate driver | *TBD — fill in from BOM* | *(add link once part is confirmed)* |
| Q_H1/L1, Q_H2/L2, Q_H3/L3 | Power MOSFETs (half-bridge) | *TBD — fill in from BOM* | *(add link once part is confirmed)* |
| U_CAN | CAN transceiver | *TBD — fill in from BOM* | *(add link once part is confirmed)* |
| U_TEMP | I²C temperature sensor | *TBD — fill in from BOM* | *(add link once part is confirmed)* |
| J_USB | Mini-USB-B receptacle | e.g. **CCMUSBB-32005-201**-style Mini-USB-B SMT receptacle | [Mini-USB-B connector datasheet reference](https://www.datasheets.com/en/part-details/ccmusbb-32005-201-infineon-technologies-ag-37951400) |
| J_CAN (P101) | CAN bus connector | *TBD — fill in from BOM* | — |
| J_MOTOR | 3-phase motor output | *TBD — fill in from BOM* | — |
| J_PWR (V_SUPPLY) | Power input connector | *TBD — fill in from BOM* | — |


---

## 5. Design Notes (from schematic annotations)

- **Input supply**: schematic marks `V_SUPPLY` for **0–60 V** and explicitly calls out: *"needs external decoupling caps to avoid high voltage transients produced by the inductive wiring while switching the FETs. Also critical for EMI/RF compliance."* Make sure adequate bulk + high-frequency decoupling is placed close to the input connector before powering the board.
- **USB block**: has ESD protection noted at the USB data lines; a note also flags *"mount OR if used"* near the USB shield/ground strapping — double check this jumper/option before assembly if you plan to use USB.
- **Servo output**: a 100R series resistor is called out as needed *"if used on servo output"* — only populate if you're driving a servo-style PWM consumer from this header.
- **Gate driver stage**: exposes a hardware `FAULT` signal back to the MCU and per-phase current sense (`SENS1-3`) plus bus-side sense (`BR_SO1/BR_SO2`, `DC_CAL`) for over-current protection and current-based control loops.
- **Hall sensor filtering**: hall and motor-temperature lines are routed through a dedicated filter sheet (`hall_filters.sch`) before reaching the MCU, reducing switching noise pickup on these sensitive lines.

---

## 6. Repository Structure (suggested)

```
.
├── docs/                  # Datasheets, board photos, schematic exports
│   ├── datasheets/
│   └── board_photo.jpg
├── hardware/               # KiCad project
│   ├── BLDC_4.sch
│   ├── STM32F4_64LQFP.sch
│   ├── Power.sch
│   ├── mosfets.sch
│   ├── hall_filters.sch
│   └── BLDC_4.kicad_pcb
├── firmware/                # STM32 firmware (if included)
└── README.md
```

Adjust to match your actual folder layout — the schematic file names above (`Power.sch`, `mosfets.sch`, `hall_filters.sch`, `STM32F4_64LQFP.sch`) are taken directly from the sheet references in `BLDC_4.sch`.

---

## 7. Datasheets

Add PDFs (or links) for every part used, in `docs/datasheets/`. Confirmed so far:

- **STM32F4 (STM32F405xx/STM32F407xx) family datasheet** — STMicroelectronics: https://www.st.com/resource/en/datasheet/dm00037051.pdf
- **Mini-USB-B SMT receptacle** (connector family referenced by the schematic's `MINI-USB-SHIELD-32005-201` label): https://www.datasheets.com/en/part-details/ccmusbb-32005-201-infineon-technologies-ag-37951400

Still to add once BOM part numbers are confirmed:
- Gate driver IC datasheet
- Power MOSFET datasheet
- CAN transceiver datasheet
- I²C temperature sensor datasheet

---

## 8. Revision

- **Title**: BLDC Driver
- **File**: `BLDC_4.sch`
- **Sheet**: 1/7 (top level)
- **Size**: A4
- **Rev**: 4.5
- **Date**: 2026-08-20
- **EDA**: KiCad E.D.A. 10.0.1
- **Author**: Vigneshwar G

---

