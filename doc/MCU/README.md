# MCU documentation — CH32H417

Silicon-level docs (chip), distinct from board-level docs in `doc/EVT/PUB/`.

Sourced from [ch32-riscv-ug/CH32H417](https://github.com/ch32-riscv-ug/CH32H417) and [openwch/ch32h417](https://github.com/openwch/ch32h417).

## Files

| File | Content |
|------|---------|
| `CH32H417_Datasheet_EN.PDF` | Electrical specs, pinout, AC/DC. **Pin AF table lives here.** |
| `CH32H417_Datasheet_CN.PDF` | Chinese version |
| `CH32H417_ReferenceManual_CN.PDF` | Register reference — Chinese only |
| `architecture.png`, `system_block.jpg` | Block diagrams |
| `CH32H417_ReferenceManual_EN_TOC.md` | 47-chapter index, EN, with line numbers into `_extracted/rm_full.txt` |
| `CH32H417_RM_Ch03_RCC_EN.md` | Clocks, PLL, I2S clock source |
| `CH32H417_RM_Ch09_GPIO_AFIO_EN.md` | Pin modes, AF mux, power domains |
| `CH32H417_RM_Ch10_DMA_EN.md` | DMA + circular ping-pong |
| `CH32H417_RM_Ch23_SPI_I2S_EN.md` | I2S TX config + formulas |
| `CHRV3UFI_Library_EN.md` | WCH USB Host + MSC + FAT32 API |
| `_extracted/*.txt` | Full RM as grep-able plain text |

Register names and bit fields inside the PDFs are already English — translations target narrative + tables.

## Not yet public

- RM English (only CN)
- QingKeV5 Processor Manual
- Errata — watch ch32-riscv-ug

## Sources

- <https://github.com/ch32-riscv-ug/CH32H417>
- <https://github.com/openwch/ch32h417>
- <https://www.wch-ic.com/products/CH32H417.html>
