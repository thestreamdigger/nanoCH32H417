# PINOUT — nanoCH32H417 v1.0

Verified against top-view photo (silkscreen) + `hardware/nanoCH32H417.pdf` (schematic).

## Notation

Silkscreen drops the `P` prefix to save space:
- `D0` on board = **PD0** in firmware
- `A14` = **PA14**, `C12` = **PC12**, `F2` = **PF2**, etc.
- Letters: **A=PA, B=PB, C=PC, D=PD, E=PE, F=PF**

## Headers — 4 segments × 13 pins

Board edges, viewed with USB-FS at top-left (matching the top-view photo).

### J5 — top-left segment (corner GND → LCD FPC)

| Silk | MCU | Alt functions (primary use) |
|------|-----|------------------------------|
| GND  | —   | ground |
| D0   | PD0 | GPIO |
| C12  | PC12 | GPIO |
| C11  | PC11 | GPIO |
| A14  | PA14 | **SWCLK** (shared with WCHLink-E via SB6) |
| A15  | PA15 | GPIO |
| C10  | PC10 | GPIO |
| A13  | PA13 | **SWDIO** (shared with WCHLink-E via SB5) |
| A8   | PA8  | GPIO / TIM1_CH1 |
| C9   | PC9  | GPIO |
| C8   | PC8  | GPIO |
| C7   | PC7  | GPIO |
| C6   | PC6  | GPIO |

### J6 — top-right segment (LCD FPC → corner F13)

| Silk | MCU | Alt functions |
|------|-----|---------------|
| 3V3  | —   | 3.3 V out |
| F2   | PF2 | GPIO |
| D14  | PD14 | GPIO |
| D12  | PD12 | GPIO |
| D15  | PD15 | GPIO |
| D13  | PD13 | GPIO |
| D11  | PD11 | GPIO |
| D10  | PD10 | GPIO |
| E15  | PE15 | GPIO |
| E14  | PE14 | GPIO |
| E13  | PE13 | GPIO |
| E12  | PE12 | GPIO |
| F13  | PF13 | GPIO |

### J9 — bottom-left segment (corner 5V → SD slot)

| Silk | MCU | Alt functions |
|------|-----|---------------|
| 5V   | —   | 5 V in / out (VBUS) |
| D1   | PD1 | GPIO |
| D2   | PD2 | GPIO |
| D3   | PD3 | GPIO |
| D4   | PD4 | GPIO |
| D5   | PD5 | GPIO |
| D6   | PD6 | GPIO |
| D7   | PD7 | GPIO |
| F3   | PF3 | GPIO |
| F4   | PF4 | GPIO |
| F5   | PF5 | GPIO |
| E0   | PE0 | GPIO |
| B4   | PB4 | **CC2** (USB-C config channel 2) |

### J10 — bottom-right segment (SD slot → corner F12)

| Silk | MCU | Alt functions |
|------|-----|---------------|
| GND  | —   | ground |
| A3   | PA3 | **ADC3 / OP3_N1** |
| A4   | PA4 | **DAC1 / ADC4 / OP3_O1** |
| A5   | PA5 | **DAC2 / ADC5 / OP1_O1** |
| A6   | PA6 | **ADC6 / OP1_P1** |
| A7   | PA7 | **ADC7 / OP1_N1** |
| C1   | PC1 | **ADC11 / HSADC1** |
| C5   | PC5 | **ADC15** |
| B0   | PB0 | **ADC8 / OP1_P0 / CMP_P0** |
| B1   | PB1 | **ADC9 / OP1_N0 / OP2_O1 / CMP_N0** |
| B2   | PB2 | **CMP_P1** |
| F11  | PF11 | **OP2_P1** |
| F12  | PF12 | **OP2_N1** |

## Fixed-function blocks (not on headers)

### USB-FS Type-C (top-left) — USB 2.0 OTG + PD
| Signal | MCU |
|--------|-----|
| OTG_FS_DP | PA12 |
| OTG_FS_DM | PA11 |
| VBUS sense | PA9 |
| ID | PA10 |
| CC1 | PB3 |
| CC2 | PB4 (exposed on J9!) |

### USB-3.0-SS Type-A (bottom-right)
| Signal | MCU pin # |
|--------|-----------|
| USBHS_DP | PB8 (pin 120) |
| USBHS_DN | PB9 (pin 121) |
| SS_TXA  | pin 124 |
| SS_TXB  | pin 123 |
| SS_RXA  | pin 127 |
| SS_RXB  | pin 126 |
| VDD12A  | pin 125 (dedicated supply) |

### Ethernet (top-right) — internal SerDes + external PHY in HR911105A
| Signal | MCU |
|--------|-----|
| MDIR_P | PE6 |
| MDIR_N | PE5 |
| MDIT_P | PE3 |
| MDIT_N | PE4 |
| LED0   | (debugger-side) |
| LED1   | (debugger-side) |

### microSD slot (SDIO 4-bit)
| Signal | MCU |
|--------|-----|
| SD_CMD  | PB10 |
| SD_CLK  | PB11 |
| SD_D0   | PE8 |
| SD_D1   | PE9 |
| SD_D2   | PE10 |
| SD_D3   | PE11 |

### LCD FPC-12P (SPI)
| Pin | Signal | MCU |
|-----|--------|-----|
| 1 | GND | — |
| 2 | LEDK | — |
| 3 | LEDA | — |
| 4 | VDD | 3V3 |
| 5,6,12 | GND | — |
| 7 | DC | PD9 |
| 8 | CS | PB12 |
| 9 | SCK | PB13 |
| 10 | SDA (MOSI) | PB15 |
| 11 | RESET | PD8 |

### Crystals
| Net | MCU | Notes |
|-----|-----|-------|
| HSE 25 MHz | XI / XO (pins 23, 24) | dedicated, not on header |
| LSE 32.768 kHz | X32KI / X32KO | footprint X1 unpopulated |

### WCHLink-E debug pads (board center, near MCU)
Five exposed pads for using the onboard CH32V305 as **external** SWD programmer:
`SWDIO • SWCLK • TX • RX • 5V`

To free PA13/PA14 for header use, cut **SB5** (SWDIO) and/or **SB6** (SWCLK).

## ADC / DAC quick map

| Channel | MCU | Header |
|---------|-----|--------|
| DAC1 | PA4 | J10:A4 |
| DAC2 | PA5 | J10:A5 |
| ADC3 | PA3 | J10:A3 |
| ADC4 | PA4 | J10:A4 |
| ADC5 | PA5 | J10:A5 |
| ADC6 | PA6 | J10:A6 |
| ADC7 | PA7 | J10:A7 |
| ADC8 | PB0 | J10:B0 |
| ADC9 | PB1 | J10:B1 |
| ADC11 | PC1 | J10:C1 |
| ADC15 | PC5 | J10:C5 |

Lower ADC channels (ADC0–ADC2, ADC10, ADC12–ADC14) are on PA0–PA2 / PC0 / PC2–PC4, **not** routed to expansion headers (used internally for board features).

HSADC channels (HSADC0–HSADC6) are on PC0–PC3 / PF8–PF10 (also internal).

## Power

- **5V** pin on J9: input (USB VBUS pass-through) or output (when board self-powered via WCHLink Type-C)
- **3V3** pin on J6: regulator output (ME6211C33M LDO), ~500 mA
- **GND** on J5 and J10

## LED situation

No user-controllable LED. The four visible LEDs are:
- D1 (blue) — VBUS_CC power indicator
- D2 (green) — 3V3 rail
- D3 (blue) — WCHLink-E connected
- D4 (red) — WCHLink-E activity

For blink tests, wire an external LED + resistor to any GPIO. Recommended: J10:B0 (PB0) — close to GND and easy to probe.

## Quick reference for prompts

When asking Claude to drive a pin, use the **silkscreen label** to avoid ambiguity:

> "Toggle B0 at 1 Hz" — clear: PB0 on J10
> "Read ADC on A6" — clear: PA6 (ADC6) on J10
> "SPI LCD on the FPC" — clear: PB12/13/15 + PD8/9

Avoid bare "GPIO 5" or "pin 12" — silkscreen is the source of truth.
