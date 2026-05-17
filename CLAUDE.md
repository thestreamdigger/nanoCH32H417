# CLAUDE.md — nanoCH32H417

Fork of [wuxx/nanoCH32H417](https://github.com/wuxx/nanoCH32H417), enriched for CLI vibecoding.

## Hardware

**CH32H417QEU6** — RISC-V V5F @ 400 MHz + V3F @ 150 MHz, 960 KB Flash, 896 KB SRAM (128 KB ITCM + 256 KB DTCM).

Onboard: WCHLink-E (CH32V305), USB 3.0 SS Type-A, USB 2.0 Type-C OTG/PD, Ethernet 100M, microSD (SDIO), FPC-12P SPI LCD, 25 MHz HSE. **No user LED, no LSE populated.**

Buttons: RESET, MODE (BOOT0), IAP.

## Targets

| Target | Use |
|--------|-----|
| **V5F** @ 400 MHz, FPU | main app, USB SS, ETH |
| **V3F** @ 150 MHz | coprocessor / low power |

EVT examples ship both. `Merge.Bin` from V5F gets flashed.

## Workflow

CLI build via `riscv-none-elf-gcc` + Make + `wlink` — see [BUILD.md](BUILD.md). MounRiver Studio is the official IDE alternative.

## Files

| File | Content |
|------|---------|
| [PINOUT.md](PINOUT.md) | silkscreen → MCU pin map |
| [EXAMPLES.md](EXAMPLES.md) | index of 47 peripherals in `doc/EVT/EXAM/` |
| [BUILD.md](BUILD.md) | toolchain + Makefile + flash |
| [COMMUNITY.md](COMMUNITY.md) | upstream, ecosystem, vendors |
| [PROJECT_FLAC_PLAYER.md](PROJECT_FLAC_PLAYER.md) | active project: USB FLAC → I2S |
| [doc/MCU/](doc/MCU/) | datasheets + RM + EN translations |

## SDK layout

`doc/EVT/EXAM/SRC/`:
- `Core/` — core_riscv.{c,h}
- `Startup/` — `startup_ch32h417_v5f.S`, `_v3f.S`
- `Ld/V5F/` + `Ld/V3F/` — linker scripts
- `Debug/` — `printf` via UART1
- `Peripheral/inc/` (42) + `src/` (38)

## Conventions

- Code text in English; `snake_case` files, `PascalCase` classes, `UPPER_CASE` constants
- No inline comments
- `VERSION` macro in `src/version.h`

## Search Chinese RM

```bash
pdftotext doc/MCU/CH32H417_ReferenceManual_CN.PDF - | grep -i 'i2s\|pll\|usb_ss'
```

## Key Learnings

Fill as discovered. Watch for:
- HSE 25 MHz vs 44.1k-family clock fit (likely needs external TCXO for audiophile)
- WCHLink-E shares PA13/PA14 with header — cut SB5/SB6 for external SWD
- USB SS PHY needs VDD12A decoupling
- V5F FPU enable in startup

## Compaction

Preserve: pinout table, active example, build errors, flash port, `wlink probe` output.
