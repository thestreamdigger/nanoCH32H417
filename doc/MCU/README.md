# MCU documentation — CH32H417

Official WCH chip-level docs imported from [ch32-riscv-ug/CH32H417](https://github.com/ch32-riscv-ug/CH32H417) and [openwch/ch32h417](https://github.com/openwch/ch32h417).

These are the **silicon-level** references — distinct from the board-level docs in `doc/EVT/PUB/` (which describe the WCH eval board, not the MCU).

## Files

| File | Size | Language | Content |
|------|------|----------|---------|
| `CH32H417_Datasheet_EN.PDF` | 1.8 MB | English | electrical specs, pinout (all packages), AC/DC characteristics, abs max ratings |
| `CH32H417_Datasheet_CN.PDF` | 2.0 MB | Chinese | same as EN, sometimes more detailed appendices |
| `CH32H417_ReferenceManual_CN.PDF` | 7.9 MB | Chinese only (EN not published yet) | register-level reference: clock tree, PLL, every peripheral, memory map, boot ROM |
| `architecture.png` | 210 KB | — | block diagram of MCU |
| `system_block.jpg` | 337 KB | — | system-level peripheral grouping diagram |

## What's missing

The following docs are **not yet publicly available** (chip released Dec 2025):

- **Reference Manual EN** — only Chinese version exists. Use translation tools when needed (the register names and bit fields are in English inside the PDF; only narrative text is CN).
- **QingKeV5 Processor Manual** — V5 core ISA + interrupt + debug spec. Not on the public WCH download portal yet. Older cores V2/V3/V4 manuals exist as references.
- **QingKeV3 Processor Manual** — exists, behind WCH SPA. Download manually from [wch-ic.com](https://www.wch-ic.com/downloads/QingKeV3_Processor_Manual_PDF.html) (requires JS-enabled browser).
- **Errata** — not yet published. Check [ch32-riscv-ug/CH32H417](https://github.com/ch32-riscv-ug/CH32H417) periodically.

## Reading order for vibecoding

1. **`architecture.png` + `system_block.jpg`** — orientation (already saw `CH32H417_Resource.jpg` in `doc/`; these add subsystem groupings)
2. **`CH32H417_Datasheet_EN.PDF`** — pinout, packages, electrical limits. **Read before writing pin-config code.**
3. **`CH32H417_ReferenceManual_CN.PDF`** — chapter-by-chapter as needed:
   - Chapter on **RCC/clocks** before any PLL setup
   - Chapter on **I2S/SPI** for the audio project
   - Chapter on **USB SS** for USB 3.0 work
   - Chapter on **SerDes** for Ethernet
4. Use `pdftotext` for grep-able search: `pdftotext CH32H417_ReferenceManual_CN.PDF - | grep -i "i2s\|i²s"`

## Sources

- [ch32-riscv-ug/CH32H417](https://github.com/ch32-riscv-ug/CH32H417) — datasheet EN/CN + RM CN
- [openwch/ch32h417](https://github.com/openwch/ch32h417) — official WCH (points to wch-ic.com for PDFs)
- [wch.cn — CH32H417](https://www.wch.cn/products/CH32H417.html) — direct vendor page
- [wch-ic.com — CH32H417](https://www.wch-ic.com/products/CH32H417.html) — English vendor page
