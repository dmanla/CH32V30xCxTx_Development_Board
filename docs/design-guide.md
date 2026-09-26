# CH32V307VCT6 Hardware Design Guide

All part-specific values are taken from the official *CH32V303/305/307 Datasheet V2.7*
and the WCH reference design `CH32V307V-EVT-R1` (openwch/ch32v307 → `SCHPCB/CH32V307V-R1-1v0`).
Where a number is not stated by WCH, it is flagged as *practice* (community/EVT precedent, verify in your review).

---

## 1. Power architecture

The chip has **four independent supply domains** — do not merge them blindly in CAD:

| Rail | Range | Powers | Notes |
|---|---|---|---|
| `VDD` | 2.4–3.6 V | Some I/O, internal LDO, **built-in USB HS PHY and Ethernet PHY** | 3.3 V nominal |
| `VIO` | 2.4–3.6 V | Most I/O **and the Ethernet module I/O**; sets the GPIO high level | Must never exceed VDD; tie to 3V3 |
| `VDDA` | 2.4–3.6 V | ADC, DAC, OPA, PLL, HSI; must match VIO level | Must be ≥ 2.4 V when ADC is used |
| `VBAT` | 1.8–3.6 V | RTC, LSE, backup registers via internal power switch | Tie to 3V3 if no battery |

- Built-in **POR/PDR**: rising threshold 1.9–2.4 V. No external reset supervisor required.
- **PVD** (2.39–3.31 V selectable) available in software for brown-out warning.
- LQFP-100 supply pins (datasheet Table 3-1): `VBAT=6`, `VDD_5=11`, `VDD_4=28`, `VDD_2=75`, `VDDA=22`, `VSSA=19`, `VREF-=20`, `VREF+=21`, `VSS_5=10`, `VSS_4=27`, `VSS_1=49`, `VIO_1=50`, `VSS_2=74`, `VSS_3=99`, `VIO_3=100`. Check Table 3-1 for every VDDx/VIOx group and give **each its own 100 nF**.
- Decoupling (*practice*, matches EVT): 100 nF at every VDD/VIO/VDDA/VBAT pin + one 4.7–10 µF bulk per rail; feed `VDDA` through a ferrite bead with 1 µF + 10 nF; 100 nF on VBAT.
- `VREF+`/`VREF-` exist **only on LQFP-100** (you have it). VREF+ ≤ VDDA; connect to VDDA with a 100 nF + 1 µF pair, or bring out to a header/precision reference for serious ADC work. VREF- → VSSA.
- 3.3 V generation (EVT uses a 1117-3.3 LDO): budget for **144 MHz full run ≈ 22 mA MCU** plus USB/Ethernet PHYs; an AMS1117 is fine for a dev board, a low-noise LDO (ME6211/RT9013 class) is *practice* for ADC-heavy designs.

## 2. Reset

- `NRST` = pin 14, active low, internal weak pull-up **30–50 kΩ (typ 40 k)**.
- Datasheet reference circuit: **0.1 µF NRST→GND** (Figure 4-7). EVT adds a 10 k pull-up and a reset button — follow the EVT.

## 3. Boot / ISP

Three boot modes selected by `BOOT0` (pin 94) and `BOOT1`/`PB2`:

| BOOT0 | BOOT1 | Mode |
|---|---|---|
| 0 | x | Run from CodeFlash |
| 1 | 0 | **System memory bootloader (ISP)** — reprograms flash over USART1 (PA9/PA10) or USB (USBFS, PA11/PA12) with WCHISPTool |
| 1 | 1 | Boot from SRAM |

- Bootloader (28 KB) is factory-cured in system memory.
- Devices *without* a BOOT0 pin pull it down internally; for packages that expose it, fit a **10 k pull-down + BOOT0 button to 3V3** (EVT scheme) so the board can be forced into ISP by holding BOOT0 while pressing reset.
- `BOOT1/PB2` is internally pulled down; keep it low (10 k to GND is fine) and free of strong external loads.

## 4. Clocks

| Source | Value | Recommendation |
|---|---|---|
| HSI (internal) | 8 MHz factory-trimmed | Default after reset; usable alone |
| LSI (internal) | ~40 kHz | IWDG / RTC fallback |
| **HSE** (OSC_IN=12, OSC_OUT=13) | 3–25 MHz crystal | 8 MHz as on EVT; 25 MHz max with ESR ≤ 60 Ω; recommended load ~30 pF total incl. stray (Figure 4-5) |
| LSE (PC14=8, PC15=9) | 32.768 kHz | Optional RTC; CL1=CL2 ≈ 12 pF, RS < 70 kΩ, startup ≤ 800 ms (Figure 4-6) |
| PLL | → SYSCLK up to 144 MHz | USBFS needs SYSCLK = 48/96/144 MHz |

- **Ethernet does not require an external 25 MHz crystal or a 50 MHz oscillator.** The integrated 10M PHY takes its reference from an internal PLL (PLL3) off the HSE; the 8 MHz-crystal EVT runs Ethernet fine (Zephyr exposes this as `internal-phy-pllmul`/`internal-phy-prediv`, "adjust so PHY clocks at ~60 MHz").
- `OSC_IN/OSC_OUT` double as PD0/PD1 on LQFP-100 (no remap needed there). If you skip HSE, the EVT still recommends keeping it — most examples assume 8 MHz HSE.

## 5. Debug & programming

- 2-wire SDI debug, enabled by default after reset: **SWDIO = PA13 (72), SWCLK = PA14 (76)**.
- Fit a 2×4 or 1×4 header: `3V3, GND, SWDIO, SWCLK` (+`NRST` optional, 5th pin). The official Arduino core uploads via **WCH-LinkE** + openocd_wch, so this header is mandatory for the Arduino story.
- Optional rescue interfaces (recommended for a dev board):
  - USART1 **PA9/PA10 (68/69)** on a header or CH340-class USB-serial bridge → WCHISPTool serial ISP.
  - USBFS connector → WCHISPTool USB ISP (hold BOOT0, press reset).
- EVT also wires LEDs on PA9/PA10 (USART1 TX1/RX1) — handy, but remember those pins are your ISP serial too; avoid loading them heavily.

## 6. USB

Two independent controllers:

| Module | Pins (LQFP-100) | PHY | Notes |
|---|---|---|---|
| USBFS / OTG_FS | DM=PA11 (70), DP=PA12 (71) | Internal FS transceiver | Internal 1.5 k D+ pull-up (software-controlled) — EVT adds no external pull-up. VBUS sense is optional (maps to PA9). 48 MHz clock must come from PLL → SYSCLK 48/96/144 MHz |
| USBHS | DM=PB6 (92), DP=PB7 (93) | **Built-in 480 Mbps HS PHY** | Needs 90 Ω differential pair routing; when USBHS is enabled, DVP_D5 relocates to PB3 and FSMC_NADV to PD2 (datasheet Notes 8–9) |

- For device-mode Type-C ports: 5.1 k CC1/CC2 pull-downs. (EVT footprints them as NC(5.1K); populate them if you want to be powered from a C-to-C charger/cable.)
- Add USBLC6-class TVS on both ports; route D+/D- as short 90 Ω pairs.

## 7. Ethernet (10M PHY on-chip)

- RMII wiring (LQFP-100): `TXP/TXN = PC8/PC9 (65/66)`, `RXP/RXN = PC6/PC7 (63/64)`, `MDIO = PA2 (25)`, `MDC = PC1 (16)`.
- Connect **TXP/TXN/RXP/RXN directly to the magnetics** (EVT uses an RJ45 jack with integrated transformer, HR911105A-class, plus series resistors on the pairs). `VIO` powers the Ethernet I/O — keep it at 3V3 and decouple it well; the PHY itself runs off VDD.
- Ethernet uses PA1 (`ETH_RMII_REF_CLK`) only when an **external** PHY is used; with the internal PHY it is free for other uses (it is also ADC_IN2/TIM2_CH3).
- **10 Mbps full/half duplex** only for the on-chip PHY; the MAC additionally supports an external Gigabit PHY via RGMII if you ever want that.
- Layout: 100 Ω differential, short, matched; Bob Smith termination (4 x 75 Ω + 1 nF/2 kV) at the magnetics; RJ45 at board edge, chassis/shield network per magnetics datasheet.

## 8. GPIO notes

- 80 I/Os, `FT` type pins are **5 V-tolerant** (abs max 5.5 V input). Analog pins (I/O/A class, e.g. PA1–PA7, PC0–PC5, PB0/PB1) are **not** 5 V-tolerant.
- Special pins to protect: `NRST` (14), `BOOT0` (94), `SWDIO/SWCLK` (72/76), `PC13/TAMPER` (7, weak drive — 30 pF max load in VBAT mode), `PA0/WKUP`.
- PC14/PC15 are LSE-only while the backup domain runs from VBAT; usable as GPIO only when backup is powered from VDD.
- The EVT R1 exposes **Arduino-UNO-style headers** (A0–A5 = PA0–PA5, D0/D1 = PA10/PA9 = USART1, plus D2–D13 and power rows). For Arduino-core compatibility, mirror that pin mapping on your headers so community variants/pinouts map 1:1.

## 9. PCB layout guidance

- 4 layers preferred: Top (signals) / GND / 3V3 power / Bottom (CHLinkNet-style reference build).
- Decoupling caps within 1–2 mm of each supply pin; solid return path under every high-speed net.
- USBHS and Ethernet pairs: 90 Ω / 100 Ω differential respectively, matched, with nearby GND vias at layer transitions.
- Crystal bank: tight, symmetric, ground-poured, away from ETH/USB and switching regulator.
- Keep the 5 V input trace, LDO, and magnetics away from the antenna-ish edges; add via stitching along Ethernet/USB pairs.
- Test points: NRST, BOOT0, PA13/PA14, PA9/PA10, 3V3, 5V — you will thank yourself during bring-up.

## 10. Absolute maximum ratings (short list)

- `VDD–VSS`, `VIO–VSS`, `VDDA–VSS`: –0.3 … 4.0 V
- Input on FT pins: ≤ 5.5 V
- Ethernet PHY differential pins have their own abs-max entry — keep line transients inside spec (use the magnetics' isolation).

## 11. Verified pin map (LQFP-100, excerpt)

| Function | Pin | LQFP-100 # |
|---|---|---|
| VBAT | VBAT | 6 |
| HSE | OSC_IN / OSC_OUT | 12 / 13 |
| LSE | PC14 / PC15 | 8 / 9 |
| NRST | NRST | 14 |
| VSSA / VREF- / VREF+ / VDDA | — | 19 / 20 / 21 / 22 |
| WKUP | PA0 | 23 |
| ETH | PA1 (REF_CLK, ext-PHY only) / PA2 (MDIO) / PC1 (MDC) | 24 / 25 / 16 |
| ETH PHY | PC6/RXP, PC7/RXN, PC8/TXP, PC9/TXN | 63 / 64 / 65 / 66 |
| USART1 | PA9 / PA10 | 68 / 69 |
| USBFS | PA11/DM, PA12/DP | 70 / 71 |
| SDI | PA13/SWDIO, PA14/SWCLK | 72 / 76 |
| VSS_2 / VDD_2 | — | 74 / 75 |
| USBHS | PB6/DM, PB7/DP | 92 / 93 |
| BOOT0 / BOOT1 | BOOT0 / PB2 | 94 / 37 |

Full map: datasheet Table 3-1. (A few VDD/VIO groups and PD-port rows are omitted here — check Table 3-1 while routing.)
