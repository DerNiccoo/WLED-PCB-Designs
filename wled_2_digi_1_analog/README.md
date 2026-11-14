# WLED LED Controller Board

A 4-channel WLED controller board built around the custom [ESP32-S3 dev board](../wled_esp_board/README.md). It supports **two digital LED strips** (e.g. WS2812B / SK6812 via level-shifted data lines) and **one analog/PWM-dimmed strip** (5-channel RGBWW), with individually fused power lines per strip output. Total sustained current capacity is ~10A, with 3–4A recommended per strip.

## Overview

<img width="2364" height="1290" alt="image" src="https://github.com/user-attachments/assets/489bfd48-8d9e-4e52-b717-9b9e58d0c3d9" />
<img width="2364" height="1290" alt="image" src="https://github.com/user-attachments/assets/d13bb4f6-be94-4120-8c79-d8eed5db6016" />


This board was designed to be a clean, self-contained controller unit. It plugs the ESP32-S3 board into a socket, handles all power routing, level shifting, MOSFET switching, and fusing in one place. Two power inputs are available, but only **one should be active at a time** — a Schottky diode (D2) and TVS diode (D1) protect the board when switching sources.

## Motivation & Design Decisions

- **Two power inputs, one at a time**: A 5.5×2.1 mm barrel jack (J1) and a screw terminal input are both present for flexibility (bench supply vs. PSU). The Schottky diode D2 (SS34) on one path prevents backfeed between sources.
- **TVS protection (D1, SMBJ26A)**: A 26V transient voltage suppressor clamps voltage spikes on the main 5V bus — important with longer LED strip wiring that can generate inductive kickback.
- **Individually fused power lines**: Each LED strip's 5V supply line runs through a dedicated ATO blade fuse holder (FH2/FH3/FH4). This isolates fault conditions to a single channel. ATO fuses are cheap, universally available, and hot-swappable.
- **Level shifter (U1, 74AHCT125)**: The ESP32-S3 runs at 3.3V. WS2812B and similar LEDs expect 5V logic. The 74AHCT125 quad buffer shifts both DATA lines and clock signals to proper 5V levels reliably, without needing a dedicated 3.3V-compatible LED strip.
- **N-channel MOSFETs for PWM dimming (Q1, Q6–Q9, IRLR7843TR)**: Five low-side N-channel MOSFETs handle the analog RGBWW channels. The IRLR7843TR has an extremely low RDS(on) (< 3 mΩ), so it barely heats up even at 3–4A per channel.
- **P-channel MOSFETs for high-side switching (Q2, Q5)**: High-side switches on the digital strip power rails, driven by NPN transistors (Q3/Q4, MMBT4401). This allows the ESP32-S3 to cut power to strips entirely, useful for preventing LED ghost-lighting when WLED is in an "off" state.
- **5V buck for board logic (U2, AP63205WU)**: Derives a clean regulated 5V from the main bus specifically for the 74AHCT125 and the ESP32-S3 board's own regulator input. Uses a 4.7 µH / 1210 inductor (L1) on the switching output.
- **Screw terminals in 2- and 3-pin configurations**: Instead of using hard-to-find 1- or 5-pin terminals, standard KF301 2-pin, 3-pin and 4-pin blocks (5.0 mm pitch) are used throughout.

## ESP32-S3 Pinout (Signal Mapping)

The controller board uses the following signals from the ESP32-S3 board headers:

<img width="743" height="761" alt="image" src="https://github.com/user-attachments/assets/4e357297-155b-4618-b4ad-3499a751109e" />

### Digital LED Strip Signals

| Signal | ESP32-S3 GPIO | Description |
|--------|--------------|-------------|
| DATA1 | IO_16 | Digital data line — Strip 1 (e.g. WS2812B) |
| DATA2 | IO_18 | Digital data line — Strip 2 |
| CLK1 | IO_15 | Clock — Strip 1 (for SPI-based LEDs like APA102) |
| CLK2 | IO_17 | Clock — Strip 2 |
| HS_Enabled_1 | IO_6 | High-side enable — cuts 5V to Strip 1 |
| HS_Enabled_2 | IO_7 | High-side enable — cuts 5V to Strip 2 |

### Analog / PWM Strip Signals

| Signal | ESP32-S3 GPIO | Description |
|--------|--------------|-------------|
| A_CH1 | IO_11 | PWM channel 1 (e.g. Red) |
| A_CH2 | IO_12 | PWM channel 2 (e.g. Green) |
| A_CH3 | IO_9 | PWM channel 3 (e.g. Blue) |
| A_CH4 | IO_5 | PWM channel 4 (e.g. Warm White) |
| A_CH5 | IO_4 | PWM channel 5 (e.g. Cold White) |

## Schematic

<img width="1965" height="520" alt="image" src="https://github.com/user-attachments/assets/5f4303a7-7de2-47a5-9027-dec80ba56052" />
<img width="2075" height="771" alt="image" src="https://github.com/user-attachments/assets/b80a80f1-d02d-47b1-88f1-23a45b13e978" />
<img width="1657" height="784" alt="image" src="https://github.com/user-attachments/assets/d4323f73-7ec9-4dd2-ae20-5e588d5fc268" />


## PCB Layout

<img width="1908" height="874" alt="image" src="https://github.com/user-attachments/assets/dedf7c96-bdb5-4393-a50c-0bac8fd8f99f" />

***

## Bill of Materials

> LCSC links point to the exact parts used. Where no LCSC link is listed, the part is a generic passive available from stock.

| Ref | Qty | Value / Part | Package | Key Specs | LCSC |
|-----|-----|-------------|---------|-----------|------|
| ESP32-S3-Board1 | 1 | WLED ESP32-S3 Board | Custom header socket | See [esp_board README](../wled_esp_board/README.md) | — |
| U1 | 1 | 74AHCT125 | SOIC-14 | Quad 3-state buffer, 5V output — level shifts 3.3V ESP data/clock to 5V | [C554502](https://www.lcsc.com/product-detail/C554502.html) |
| U2 | 1 | AP63205WU | TSOT-23-6 | 2A synchronous buck, 5V fixed — powers logic rail from main bus | [C2071056](https://www.lcsc.com/product-detail/C2071056.html) |
| Q1,Q6–Q9 | 5 | IRLR7843TR (UMW) | TO-252 | N-Ch, 30V, 161A rated, <3 mΩ RDS(on) — analog PWM low-side switches | [C7499399](https://www.lcsc.com/product-detail/C7499399.html) |
| Q2,Q5 | 2 | AO4407A (or equiv.) | SOIC-8 | P-Ch, 30V, 12A — high-side power switch for digital strips | [C16072](https://www.lcsc.com/product-detail/C16072.html) |
| Q3,Q4 | 2 | MMBT4401LT1G | SOT-23 | NPN, 40V / 600mA — drives gate of P-Ch MOSFETs from 3.3V GPIO | [C31776](https://www.lcsc.com/product-detail/C31776.html) |
| D1 | 1 | SMBJ26A | SMB (DO-214AA) | 26V TVS, 14.25A Ipp — bus transient suppression | [C19077580](https://www.lcsc.com/product-detail/C19077580.html) |
| D2 | 1 | SS34 | SMA (DO-214AC) | 40V / 3A Schottky — reverse-polarity / backfeed protection | [C8678](https://www.lcsc.com/product-detail/C8678.html) |
| FH2,FH3,FH4 | 3 | ATO Fuse Holder | Through-hole clip | One per strip power line — accepts standard ATO blade fuses | [C717032](https://www.lcsc.com/product-detail/C717032.html) |
| L1 | 1 | 4.7 µH | 1210 | 2.5A / 3.1A saturation, 115 mΩ DCR — buck output inductor | [C48945753](https://www.lcsc.com/product-detail/C48945753.html) |
| C1 | 1 | 470 µF / 35V | Radial D10mm | Electrolytic bulk cap — main bus reservoir | [C106703](https://www.lcsc.com/product-detail/C106703.html) |
| C3 | 1 | 10 µF | 0805 | Buck output filter | — |
| C4,C6,C7 | 3 | 100 nF | 0805 | Decoupling caps | — |
| C8,C9 | 2 | 22 µF | 0805 | Ceramic X5R 6.3V — local bulk caps | [C307530](https://www.lcsc.com/product-detail/C307530.html) |
| J1 | 1 | Barrel Jack 5.5/2.1 mm | Horizontal THT | Primary power input (center positive) | — |
| J2 | 1 | Screw Terminal 3-pin | KF301 5.0 mm | Secondary / alternative power input + GND | — |
| J3,J4 | 2 | Screw Terminal 4-pin | KF301 5.0 mm | LED strip output connectors (2× split 4-pin instead of 8-pin) | — |
| U3–U5 | 3 | Screw Terminal 2-pin | KF301 5.0 mm | Analog strip channel outputs | — |
| R2,R7,R11,R13,R15,R17,R19 | 7 | 100 kΩ | 0805 | Gate pull-down / bias resistors | [C149504](https://www.lcsc.com/product-detail/C149504.html) |
| R3,R6 | 2 | 1 kΩ | 0805 | NPN base resistors | — |
| R4,R5,R8,R9 | 4 | 33 Ω | 0805 | Signal series termination (level shifter outputs) | [C17634](https://www.lcsc.com/product-detail/C17634.html) |
| R10,R12,R14,R16,R18 | 5 | 10 Ω | 0805 | Gate resistors for N-Ch MOSFETs | [C17415](https://www.lcsc.com/product-detail/C17415.html) |

***

## Power Budget

| Source | Max Recommended | Notes |
|--------|----------------|-------|
| Total board input | 10A | Limited by wiring and screw terminal rating |
| Per digital strip | 3–4A | Fused individually at FH2/FH3 |
| Analog strip per channel | 3–4A | Fused at FH4; 5 channels share one fuse |

> Always size your ATO fuses to the **strip's** maximum draw, not the board's maximum. A 3A fuse per digital strip and a 5A fuse for the analog strip are good starting points.
