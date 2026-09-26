# Schematic review checklist (CH32V307VCT6)

Tick every line before releasing the schematic to layout. "EVT" = WCH CH32V307V-EVT-R1 reference design.

## Power

- [ ] Every `VDDx`, `VIOx`, `VDDA`, `VBAT` pin located from datasheet Table 3-1 and individually decoupled (100 nF)
- [ ] `VIO` tied to the same 3V3 net as `VDD` (VIO must never exceed VDD) and decoupled
- [ ] `VDDA` fed via ferrite + 1 µF + 10 nF; `VSSA` star-connected to GND
- [ ] `VREF+` ≤ VDDA: RC-decoupled to VDDA (1 µF + 100 nF), `VREF-` to VSSA
- [ ] `VBAT` → 3V3 + 100 nF (or coin cell + blocking diode + 100 nF)
- [ ] Bulk caps on 5V and 3V3 rails; LDO input/output caps per its datasheet
- [ ] Power path: USB 5V / barrel or header 5V with a protection diode or mux (EVT uses a supply switch)

## Reset / boot

- [ ] `NRST`: 0.1 µF to GND (+ 10 k pull-up and button, EVT scheme)
- [ ] `BOOT0`: 10 k pull-down + button to 3V3 (hold + reset = ISP)
- [ ] `BOOT1/PB2`: pulled low, not used for anything that drives it high at reset

## Clocks

- [ ] 8 MHz crystal (or chosen 3–25 MHz) with CL1=CL2 from `CL = (CL1·CL2)/(CL1+CL2) + Cstray`; 25 MHz needs ESR ≤ 60 Ω
- [ ] 32.768 kHz crystal with ~12 pF caps if RTC is wanted (else leave PC14/PC15 free)
- [ ] Crystals placed close to pins, guarded by GND (layout item, verify in layout review)

## Debug / programming (critical for the Arduino plan)

- [ ] 4–6 pin header: 3V3, GND, SWDIO (PA13), SWCLK (PA14), NRST — WCH-LinkE compatible
- [ ] USART1 PA9/PA10 routed to a header or USB-serial bridge (serial ISP rescue)
- [ ] No pull-ups/LEDs on PA13/PA14 stronger than the debug spec; keep them dedicated

## USB

- [ ] USBFS: PA11/PA12 to connector with D+/D- paired; **no external 1.5 k pull-up** (internal); VBUS connected if you want sensing (PA9) or left unsensed
- [ ] USBHS: PB6/PB7 to second connector if used; reserve PB3/PD2 knowledge (they move when USBHS is on)
- [ ] Type-C: 5.1 k CC1/CC2 pull-downs for device ports
- [ ] TVS/ESD parts on both USB connectors

## Ethernet

- [ ] PC6–PC9 to magnetics with series resistors (EVT-style); Bob Smith termination network present
- [ ] RJ45 with integrated magnetics (HR911105A-class), LED nets wired to spare GPIO
- [ ] MDIO=PA2, MDC=PC1 routed; PA1 left free (internal PHY clocking)

## GPIO hygiene

- [ ] No analog-class pins (PA0–PA7, PB0/PB1, PC0–PC5) treated as 5 V-tolerant
- [ ] PA9/PA10 LEDs (if copied from EVT) don't prevent serial ISP
- [ ] PC13 load ≤ 30 pF if VBAT operation matters
- [ ] Arduino-UNO-style header mapping (A0–A5 = PA0–PA5, D0/D1 = PA10/PA9) for core compatibility

## General

- [ ] Every unconnected NC pin left floating or per datasheet
- [ ] Test points: 3V3, 5V, GND, NRST, BOOT0, SWD, PA9/PA10
- [ ] Silkscreen: pin-1 markers, connector labels, ISP instructions
- [ ] ERC/DRC clean; all nets named; no hidden power flags issues
