# WLED ESP32-S3 Controller Board

A compact, low-idle-power ESP32-S3 development board designed as a drop-in controller for WLED-driven LED installations. The board accepts both 5V via pin headers and USB-C (USB 2.0), exposes the full ESP32-S3 GPIO pinout on standard 2.54 mm headers, and deliberately omits an LDO regulator in favor of a switching buck converter to minimize idle power draw.

## Overview

<img width="2364" height="1290" alt="image" src="https://github.com/user-attachments/assets/4af36e47-3d4d-416c-9104-cebd4418cb70" />


This board was created because I wanted to create my own dev board with the focus on low power consumption even in idle.

## Motivation & Design Decisions

- **Buck converter instead of LDO**: The AP63203WU is a 2A synchronous step-down regulator. Compared to a classic AMS1117/ME6211-style LDO, it wastes virtually no power as heat at light loads, making it ideal for always-on WLED setups.
- **USB-C (USB 2.0, 16-pin)**: Full USB-C receptacle with proper CC resistors (R2/R3, 5.1 kΩ) for correct 5V/500mA enumeration and ESD protection via USBLC6-2SC6.
- **Schottky reverse-polarity diode (D1, SS34)**: Protects the board when power is fed through the 5V pin headers. 3A continuous rating with low forward voltage to minimize drop.
- **Polyfuse (F1)**: Resettable 1A/30V PTC fuse on the USB path for short-circuit protection — no need to replace fuses after a mishap.
- **Reset + Boot buttons**: Physical access to BOOT and RESET for easy firmware flashing without needing any external tool.
- **Easy iron solderable**: 5 small 1mm pins to access as better access to solder the backside of the ESP without the need of a heatgun or hotplate

## Pinout

<img width="2364" height="1290" alt="image" src="https://github.com/user-attachments/assets/4eecd702-ebc6-4b39-a57d-5f4077f40dd4" />

The board breaks out all ESP32-S3-WROOM-1 GPIOs across four 9-pin 2.54 mm headers (J1–J4). Power rails (5V and 3.3V) are available directly on the headers.

## Schematic

<img width="1656" height="1236" alt="image" src="https://github.com/user-attachments/assets/f6a674aa-dcd3-42a2-9c24-10c702995e10" />


## PCB Layout

<img width="871" height="977" alt="image" src="https://github.com/user-attachments/assets/24ce2f5f-0f16-4eac-af81-ad7a06bcc9db" />

***

## Bill of Materials

> LCSC links point to the exact parts used. Where no LCSC link is listed, the part is a generic 0805 passive available from any supplier.

| Ref | Qty | Value / Part | Package | Key Specs | LCSC |
|-----|-----|-------------|---------|-----------|------|
| U1 | 1 | ESP32-S3-WROOM-1 | SMD 25.5×18 mm | Xtensa LX7 dual-core, 2.4 GHz Wi-Fi + BT5, 16 MB flash, 8 MB PSRAM | [C2913202](https://www.lcsc.com/product-detail/C2913202.html) |
| U2 | 1 | USBLC6-2SC6 | SOT-23-6 | ESD protection, 17V clamp, 5A Ipp — guards USB-C data lines | [C7519](https://www.lcsc.com/product-detail/C7519.html) |
| U3 | 1 | AP63203WU | TSOT-23-6 | 2A sync buck, 3.3V fixed output, 1.1 MHz, no LDO heat waste | [C780769](https://www.lcsc.com/product-detail/C780769.html) |
| L1 | 1 | 4.7 µH | 1210 | 2.5A / 3.1A saturation, 115 mΩ DCR — buck converter output inductor | [C48945753](https://www.lcsc.com/product-detail/C48945753.html) |
| D1 | 1 | SS34 | SMA (DO-214AC) | 40V / 3A Schottky, reverse-polarity protection on 5V pin rail | [C8678](https://www.lcsc.com/product-detail/C8678.html) |
| F1 | 1 | Polyfuse 1A | 1206 | PTC resettable fuse 30V / 1A — USB input overcurrent protection | [C5358568](https://www.lcsc.com/product-detail/C5358568.html) |
| J2 | 1 | USB-C Receptacle | HRO TYPE-C-31-M-12 | USB 2.0, 16-pin, through-hole reinforced | — |
| J1,J3,J4,J5 | 4 | 1×9 Pin Header | 2.54 mm | GPIO breakout headers | [C41430882](https://www.lcsc.com/product-detail/C41430882.html) |
| C1,C3,C6,C9 | 4 | 100 nF | 0805 | Decoupling caps for power rails | — |
| C2,C7,C8 | 3 | 22 µF | 0805 | Bulk capacitance, X5R ceramic 6.3V | [C307530](https://www.lcsc.com/product-detail/C307530.html) |
| C5 | 1 | 10 µF | 0805 | Buck output filter capacitor | — |
| R1 | 1 | 10 kΩ | 0805 | Pull-up resistor | — |
| R2,R3 | 2 | 5.1 kΩ | 0805 | USB-C CC pull-down resistors (required for correct 5V/500mA enumeration) | — |
| R4,R5 | 2 | 22 Ω | 0805 | USB D+/D− series termination resistors | — |
| SW1 | 1 | Reset | 5.2×5.2 mm SMD tact | Hardware reset | — |
| SW2 | 1 | Boot | 5.2×5.2 mm SMD tact | Puts ESP32-S3 into download mode | — |

***
