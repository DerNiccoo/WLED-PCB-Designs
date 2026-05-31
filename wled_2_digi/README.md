# WLED Dual Digital LED Controller Board

A compact WLED controller board built around the custom [ESP32-S3 dev board](../wled_esp_board/README.md), designed specifically for **two digital LED strips**. It keeps the same core ideas as the larger controller board [larger controller board](../wled_2_digi_1_analog/README.md) — dual power input options, per-output fusing, 5V level shifting, and high-side strip power switching — but removes the analog LED section to reduce board size and complexity.

## Overview

<img width="2364" height="1290" alt="image" src="https://github.com/user-attachments/assets/ea267075-e8a3-4a39-bbc2-569ccdb83818" />
<img width="2364" height="1290" alt="image" src="https://github.com/user-attachments/assets/35927406-bd7b-48e9-a9d1-68ac8be063f2" />

This board targets installations that only need addressable LED strips such as WS2812B, SK6812, or clocked chipsets like APA102. By dropping the analog MOSFET section, the layout becomes smaller, easier to route, and better suited for builds where two digital outputs are enough.

## Motivation & Design Decisions

- **Smaller than the mixed-signal controller**: This version keeps the useful infrastructure from the larger board but focuses purely on digital strips. That means less PCB area, fewer power components, and simpler wiring.
- **Two power input options**: The board can be powered either through a 5.5×2.1 mm barrel jack or a screw terminal input. Only **one source should be used at a time**.
- **Per-strip fused outputs**: Each digital strip has its own ATO blade fuse holder. A fault on one output should not take down the other strip.
- **TVS + Schottky protection**: An SMBJ26A TVS diode clamps input transients on the main power rail, while the SS34 Schottky diode helps prevent reverse current and backfeed issues.
- **5V level shifting with 74AHCT125**: The ESP32-S3 uses 3.3V logic, while many 5V LED strips prefer a stronger logic-high level. The 74AHCT125 provides reliable 5V logic for both DATA and CLOCK lines.
- **High-side strip switching**: Each strip power rail can be switched on and off using a P-channel MOSFET driven by an NPN transistor stage. This helps fully cut power to the strips and can avoid unwanted standby glow.
- **Simple screw terminal strategy**: Standard, easy-to-source terminal blocks are used instead of harder-to-find multi-position variants.

## ESP32-S3 Pin Mapping

The board uses the same ESP32-S3 signal assignment as the main controller platform for the digital channels:

| Signal | ESP32-S3 GPIO | Description |
|--------|---------------|-------------|
| DATA1 | IO_16 | Digital data output for LED Strip 1 |
| DATA2 | IO_18 | Digital data output for LED Strip 2 |
| CLK1 | IO_15 | Optional clock output for Strip 1 |
| CLK2 | IO_17 | Optional clock output for Strip 2 |
| HS_Enabled_1 | IO_6 | High-side power enable for Strip 1 |
| HS_Enabled_2 | IO_7 | High-side power enable for Strip 2 |

This allows the board to support both single-wire addressable LEDs and clocked digital LEDs, depending on the WLED configuration and connected strip type.

## Schematic

<img width="2030" height="979" alt="image" src="https://github.com/user-attachments/assets/f3182f77-9263-43d1-b509-78a4dc1db6ad" />


## PCB Layout

<img width="1892" height="1141" alt="image" src="https://github.com/user-attachments/assets/7d941704-5e47-400e-83c1-d90f4367ceea" />

***

## Bill of Materials

> LCSC links point to the matching parts from the current cart where available. Generic passives and connectors without a clear cart match are left without a direct link.

| Ref | Qty | Value / Part | Package | Key Specs | LCSC |
|-----|-----|-------------|---------|-----------|------|
| ESP32-S3-Board1 | 1 | WLED ESP32-S3 Board | Custom header socket | Main controller module used as plug-in base | — |
| U1 | 1 | 74AHCT125 | SOIC-14 | Quad buffer / level shifter, used for 5V DATA/CLOCK outputs | [C554502](https://www.lcsc.com/product-detail/C554502.html) |
| U2 | 1 | AP63205WU | TSOT-23-6 | 2A synchronous buck regulator, fixed 5V output | [C2071056](https://www.lcsc.com/product-detail/C2071056.html) |
| Q2,Q5 | 2 | AO4407A (or equivalent to AP15P04S) | SOIC-8 | P-channel MOSFET, 30V / 12A, used for high-side strip power switching | [C16072](https://www.lcsc.com/product-detail/C16072.html) |
| Q3,Q4 | 2 | MMBT4401LT1G | SOT-23 | NPN transistor, used to drive the P-channel MOSFET gates | [C31776](https://www.lcsc.com/product-detail/C31776.html) |
| D1 | 1 | SMBJ26A | SMB (DO-214AA) | TVS diode for input surge suppression | [C19077580](https://www.lcsc.com/product-detail/C19077580.html) |
| D2 | 1 | SS34 | SMA (DO-214AC) | 40V / 3A Schottky diode for protection / power-path isolation | [C8678](https://www.lcsc.com/product-detail/C8678.html) |
| FH2,FH3 | 2 | ATO Fuse Holder | Through-hole | One fuse holder per digital strip power line | [C717032](https://www.lcsc.com/product-detail/C717032.html) |
| L1 | 1 | 4.7 µH | 1210 | Buck converter inductor, 2.5A / 3.1A saturation class | [C48945753](https://www.lcsc.com/product-detail/C48945753.html) |
| C1 | 1 | 470 µF / 35V | Radial D10mm | Bulk input capacitor for the main power rail | [C106703](https://www.lcsc.com/product-detail/C106703.html) |
| C3 | 1 | 10 µF | 0805 | Regulator output / local bulk capacitance | — |
| C4,C6,C7 | 3 | 100 nF | 0805 | Local decoupling capacitors | — |
| C8,C9 | 2 | 22 µF | 0805 | Ceramic bulk capacitors, X5R 6.3V | [C307530](https://www.lcsc.com/product-detail/C307530.html) |
| J1 | 1 | Barrel Jack 5.5×2.1 mm | Horizontal THT | Main DC input connector | — |
| J3,J4 | 2 | Screw Terminal 4-pin | 5.0 mm pitch | LED strip output connectors | — |
| U3 | 1 | Screw Terminal 2-pin | 5.0 mm pitch | Alternative power input or auxiliary connection | — |
| R2,R7 | 2 | 100 kΩ | 0805 | Gate pull-down / bias resistors | [C149504](https://www.lcsc.com/product-detail/C149504.html) |
| R3,R6 | 2 | 1 kΩ | 0805 | Base resistors for transistor drivers | — |
| R4,R5,R8,R9 | 4 | 33 Ω | 0805 | Series resistors on logic outputs | [C17634](https://www.lcsc.com/product-detail/C17634.html) |

***

## Power Handling

| Path | Recommended Current | Notes |
|------|---------------------|-------|
| Total board input | Up to 10A | Depends on terminal, wiring, copper, and cooling |
| Per digital strip output | 3–4A | Each output is fused individually |

Each strip output has its own fuse holder, which makes troubleshooting easier and improves fault isolation in real installations.
