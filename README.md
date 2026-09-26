# CH32V307 Dev Board

Open hardware design for a development board around the **WCH CH32V307VCT6**
(LQFP-100, QingKe RISC-V4F @ 144 MHz), planned as an **Arduino-compatible**
platform (official core: [openwch/arduino_core_ch32](https://github.com/openwch/arduino_core_ch32)).

## Repository layout

| Path | Purpose |
|---|---|
| `docs/design-guide.md` | Chip-specific hardware design guide (power, clocks, boot, debug, USB, Ethernet, layout) |
| `docs/schematic-checklist.md` | Item-by-item review list before releasing the schematic |
| `docs/bom.md` | Minimal-system BOM suggestion |

## Key resources

- Datasheet: *CH32V303_305_307 Datasheet* (V2.7, EN) — [WCH download page](https://www.wch-ic.com/downloads/CH32V307DS0_PDF.html)
- Reference design: [openwch/ch32v307](https://github.com/openwch/ch32v307) → `SCHPCB/CH32V307V-R1-1v0/` (EVT R1 schematic + PCB)
- Peripheral libraries & examples: same repo → `EVT/EXAM/`
- Official Arduino core: [openwch/arduino_core_ch32](https://github.com/openwch/arduino_core_ch32) (uploads via **WCH-LinkE** SWD, using [openwch/openocd_wch](https://github.com/openwch/openocd_wch))
- Programmer: WCH-Link/LinkE — [WCH-Link docs](https://github.com/openwch/ch32v307/tree/main/WCH-Link)
- ISP flashing tool: WCHISPTool (serial via USART1 or USB, entered with BOOT0 high)

## Chip facts (CH32V307VCT6)

- LQFP-100, 14 x 14 mm, 0.5 mm pitch; 80 GPIO; -40..85 °C; rated 3.3 V (VDD 2.4–3.6 V)
- 480 KB CodeFlash (configurable split; 256 KB zero-wait application area typical) + SRAM 64 KB default (configurable up to 128 KB); 28 KB factory bootloader
- USBFS (PA11/PA12), USBHS with built-in 480 Mbps PHY (PB6/PB7), Gigabit MAC + integrated 10M PHY (RMII, magnetics only)
- Boot via BOOT0/BOOT1; ISP bootloader reprograms flash over **USART1 (PA9/PA10)** or **USB**

## Roadmap

1. Schematic (use `docs/schematic-checklist.md`)
2. PCB layout (see layout section of the design guide)
3. Fabrication + assembly (see `docs/bom.md`)
4. Bring-up: power → SWD connect (WCH-LinkE) → blink → USB/ETH smoke tests
5. Arduino core bring-up: core install, custom variant/pinout, upload via WCH-LinkE
