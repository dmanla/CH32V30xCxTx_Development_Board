# Minimal-system BOM (CH32V307VCT6 dev board)

Baseline follows the WCH EVT R1 + community reference builds. Values marked *alt* are optional depending on features you keep.

| Ref | Part | Value / Part # | Purpose | Notes |
|---|---|---|---|---|
| U1 | MCU | CH32V307VCT6 | LQFP-100 | WCH official channels; check date/lot for the SDIO/PB14-PB15 errata note in datasheet Notes 7–8 |
| U2 | LDO 3.3 V | AMS1117-3.3 (EVT) | 5V→3V3 | *alt:* ME6211C33 / RT9013-33 for lower noise |
| Y1 | Crystal | 8 MHz, HC-49S/SMD-3225, ±20 ppm, CL ~20 pF | HSE | ESR ≤ 60 Ω at 25 MHz; any ≤ 25 MHz within 3–25 MHz |
| Y2 | Crystal | 32.768 kHz, 12.5 pF | LSE/RTC | Optional |
| C-DEC | Caps | 100 nF X7R 0402/0603 | One per VDD/VIO/VDDA/VBAT pin | Count from Table 3-1 (≈8–10 pcs) |
| C-BULK | Caps | 10 µF (2–4 pcs) | 5V & 3V3 bulk + LDO | |
| C-A | Caps | 1 µF + 10 nF + 100 nF | VDDA / VREF+ / VBAT | |
| C-CL | Caps | per crystal CL | HSE & LSE load caps | Compute from CL formula |
| FB1 | Ferrite bead | 600 Ω @ 100 MHz | VDDA feed | |
| R-NRST | Resistor | 10 kΩ | NRST pull-up | Optional (internal ~40 k) but EVT-fitted |
| R-BOOT | Resistor | 10 kΩ | BOOT0 pull-down | |
| R-CC | Resistors | 5.1 kΩ x2 | Type-C CC1/CC2 (device) | EVT fits as NC; populate for C-C power |
| R-ETH | Resistors | per magnetics datasheet | series + Bob Smith (75 Ω x4, 1 nF/2 kV) | |
| SW1 | Button | 2-pad SMD | NRST | |
| SW2 | Button | 2-pad SMD | BOOT0 (ISP) | |
| SW3 | Button | 2-pad SMD | USER (PA0/WKUP recommended) | |
| J1/J2 | USB-C | 16-pin, 5 V | USBFS (PA11/PA12), USBHS (PB6/PB7) | Receptacle only |
| J3 | RJ45 | HR911105A / HY911105A | Ethernet, magnetics + LEDs | Keep EVT-compatible |
| J3 | Header | 2.54 mm 1x04–1x06 | WCH-LinkE SWD | 3V3/GND/SWDIO/SWCLK(+NRST) |
| J4 | Header | 2.54 mm 1x02 | USART1 PA9/PA10 | *alt:* CH340N USB-serial |
| D1 | LED | green + 1 k | Power | |
| D2/D3 | LED | + 1 k | User / PA9-PA10 (EVT) | Optional |
| TVS1/TVS2 | ESD | USBLC6-2SC6 | USB ports | |
| H1 | Arduino headers | UNO-style | A0–A5 = PA0–PA5, D0–D13 | Match EVT mapping for core compatibility |

## Sources

- CH32V307VCT6: WCH official distributors/LCSC; avoid gray-market for first spin
- Magnetics: HanRun HR911105A (EVT) or Pulse/Abracon equivalents with 10Base-T isolation
- WCH-LinkE debugger (~$4) if not put onboard like the EVT (CH549G-based)
