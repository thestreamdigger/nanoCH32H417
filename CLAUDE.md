# CLAUDE.md — nanoCH32H417

Vibecoding context for the MuseLab nanoCH32H417 board (WCH CH32H417QEU6 dual-core RISC-V).

Fork of [wuxx/nanoCH32H417](https://github.com/wuxx/nanoCH32H417), enriched for headless CLI workflow with Claude Code.

## Hardware

MCU: **CH32H417QEU6** — RISC-V V5F @ 400 MHz + V3F @ 150 MHz, 960 KB Flash, 896 KB SRAM (128 KB ITCM + 256 KB DTCM)

Onboard:
- WCHLink-E debugger (CH32V305, SWD + UART, Type-C)
- USB 3.0 Type-A SS Host/Device (5 Gbps)
- USB 2.0 Type-C OTG + PD
- Ethernet RJ45 100M (HanRun HR911105A)
- microSD slot (SDIO)
- FPC-12P SPI LCD header (ili9341/st7789 compatible)
- 25 MHz HSE, no LSE populated
- 4 × 13-pin GPIO headers + 25-pin PAD block

Buttons: **RESET** (NRST), **MODE** (BOOT0 select), **IAP** (WCHLink firmware update)

## Targets

Two RISC-V cores, pick one per build:

| Target | Core | Use |
|--------|------|-----|
| **V5F** | V5F @ 400 MHz, FPU | main app, USB SS, ETH, performance |
| **V3F** | V3F @ 150 MHz | secondary / coprocessor / low power |

EVT/EXAM/ projects ship with both. `Merge.Bin` from V5F is what gets flashed.

## Workflow

Two trilhas:

- **Trilha A — MounRiver Studio (official)**: open `.wvproj`, Build, Flash. Closed loop, hard to drive from LLM.
- **Trilha B — CLI (vibecoding)**: `riscv-none-elf-gcc` + Make + `wlink`. Headless, reproducible. See [BUILD.md](BUILD.md).

Default to Trilha B unless an example uses features that require MRS-only libs.

## Pinout

Silkscreen uses short form: `D0` = PD0, `A14` = PA14, etc. Letters = port (A..F), number = pin.

Full physical → logical map: [PINOUT.md](PINOUT.md)

## SDK

StdPeriph library is versioned in `doc/EVT/EXAM/SRC/`:

```
SRC/
  Core/        core_riscv.c/.h         RISC-V CSR + interrupt vec
  Startup/     startup_ch32h417_v5f.S  boot V5F
               startup_ch32h417_v3f.S  boot V3F
  Ld/          Link_v5f.ld, Link_v3f.ld
  Debug/       debug.c/.h              printf via UART1
  Peripheral/  inc/ (42 headers) + src/ (38 .c)
```

This is what every EXAM/ project links against. Reference for any peripheral driver work.

## Examples

47 peripherals demonstrated in `doc/EVT/EXAM/`. Index: [EXAMPLES.md](EXAMPLES.md)

## References

### In this repo (chip-level)
- [doc/MCU/CH32H417_Datasheet_EN.PDF](doc/MCU/CH32H417_Datasheet_EN.PDF) — electrical specs, pinout, AC/DC characteristics
- [doc/MCU/CH32H417_Datasheet_CN.PDF](doc/MCU/CH32H417_Datasheet_CN.PDF) — Chinese version
- [doc/MCU/CH32H417_ReferenceManual_CN.PDF](doc/MCU/CH32H417_ReferenceManual_CN.PDF) — register reference, clock tree, every peripheral (CN only — EN not published yet)
- [doc/MCU/README.md](doc/MCU/README.md) — index + reading order

### In this repo (board-level)
- `doc/EVT/PUB/CH32H417 Evaluation Board Reference-EN.pdf` — WCH eval board manual (not nanoCH32H417)
- `hardware/nanoCH32H417.pdf` — nanoCH32H417 v1.0 schematic
- `doc/EVT/EXAM/SRC/` — StdPeriph library source (42 headers + 38 .c)

### External (still on the web, not mirrored here)
- **QingKeV5 Processor Manual** — not yet public (chip too new)
- **QingKeV3 Processor Manual** — [wch-ic.com](https://www.wch-ic.com/downloads/QingKeV3_Processor_Manual_PDF.html), behind SPA, manual browser download
- **Errata** — not yet published; watch [ch32-riscv-ug/CH32H417](https://github.com/ch32-riscv-ug/CH32H417)
- [openwch/ch32h417](https://github.com/openwch/ch32h417) — official WCH upstream
- [ch32-rs/wlink](https://github.com/ch32-rs/wlink) — open Rust flash tool, supports H417

### Searching the Chinese RM
The Reference Manual is Chinese-only for now, but register names and bit fields are in English inside the PDF. To grep:
```bash
pdftotext doc/MCU/CH32H417_ReferenceManual_CN.PDF - | grep -i 'i2s\|spi\|pll\|usb_ss'
```

## Conventions

- Code text in English
- snake_case files, PascalCase classes, UPPER_CASE constants
- No inline comments (code must read itself)
- Docstrings only for protocol specs and non-obvious math
- `VERSION` macro in `src/version.h` per project

## Key Learnings

Empty — fill as gotchas appear. Examples to watch for:
- HSE 25 MHz vs HSI clock tree quirks during PLL setup
- USB SS PHY power sequencing (VDD12A requires separate decoupling)
- SerDes RX/TX pair routing on EXAM/SerDes example
- V5F FPU enable flag in startup
- Boot0 (MODE button) levels at reset
- WCHLink-E shares PA13/PA14 with header — disconnect via SB5/SB6 for external SWD
- No user LED — must wire one externally for blink tests

## Compaction

Preserve: pinout table, current example being adapted, build errors, flash port (`/dev/ttyACM*`), `wlink probe` output.
