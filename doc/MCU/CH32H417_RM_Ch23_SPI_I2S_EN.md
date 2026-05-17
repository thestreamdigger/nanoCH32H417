# Chapter 23 — Serial Peripheral Interface (SPI / I2S)

Translated from `CH32H417_ReferenceManual_CN.PDF` V1.7, lines 22779–23985 of `_extracted/rm_full.txt`. Original Chinese: `_extracted/ch23_spi_i2s_cn.txt`.

Diagrams referenced (Figure 23-N) are in the original PDF — not reproduced here.

The chip has 4 SPI peripherals (SPI1/2/3/4). Each SPI can be reconfigured as an I2S interface. SPI uses 3-wire synchronous serial; I2S adds support for 4 audio standards (Philips, MSB-aligned, LSB-aligned, PCM).

---

## 23.1 Key features

### 23.1.1 SPI features
- Full-duplex synchronous serial
- Single-wire half-duplex mode
- Master, slave, multi-slave modes
- 8-bit or 16-bit data structure
- Max clock = FHCLK / 2
- MSB-first or LSB-first data order
- Hardware or software NSS pin control
- Hardware CRC verification on TX/RX
- DMA on TX/RX buffers
- Programmable clock phase and polarity

### 23.1.2 I2S features
- Simplex (one-direction) communication
- Master and Slave modes
- **16-bit, 24-bit and 32-bit data formats**
- **Audio sample rate range 8 kHz – 562.2 kHz**
- Programmable clock polarity
- Standards: Philips I2S, MSB-aligned, LSB-aligned, PCM
- DMA on TX/RX buffers
- Master clock (MCK) output to external audio devices

> Note: HAL constants `I2S_AudioFreq_*` go up to `_192k`. The silicon supports up to 562.2 kHz — beyond 192k requires writing the registers directly (no HAL constant).

---

## 23.3 I2S Functional Description

### 23.3.1 Overview

Setting bit `I2SMOD` in register `I2SCFGR` enables I2S mode. SPI module is then used as an I2S audio interface. I2S shares 3 pins with SPI:

- **WS** (word select, frame): mapped to NSS pin. Master mode: output. Slave mode: input.
- **CK** (serial clock, BCK): mapped to SCK pin. Master: output. Slave: input.
- **SD** (serial data): mapped to MOSI pin.

For external audio devices that need a master clock:

- **MCK**: master clock, **independently mapped** (separate pin). Active when I2S is in master mode AND `MCKOE` bit of `I2SPR` register = 1. Output frequency is **256 × Fs** where Fs is the audio sample rate.

In master mode, I2S uses its own clock generator to produce communication clocks. The same generator drives the master clock output.

I2S mode adds two registers:
- `I2SPR` — clock generator config (divider, MCKOE, ODD)
- `I2SCFGR` — general I2S config (audio standard, master/slave, data format, frame packing, clock polarity)

In I2S mode, registers `CTLR1` and all `CRCR` registers are NOT used. The `SSOE` bit in `CTLR2` and `MODF`, `CRCERR` bits in `STATR` are also unused. I2S uses the same `DATAR` register as SPI for data transfer.

### 23.3.2 Audio standards

Four supported, selected via `I2SSTD[1:0]` + `PCMSYNC` in `I2SCFGR`:

| Standard | Code | Notes |
|----------|------|-------|
| Philips I2S | `I2SSTD=00` | WS toggles 1 clock before MSB; TX changes data on falling edge of CK, RX samples on rising edge; WS transitions on falling edge |
| MSB-aligned | `I2SSTD=01` | left-justified |
| LSB-aligned | `I2SSTD=10` | right-justified |
| PCM short frame | `I2SSTD=11`, `PCMSYNC=0` | |
| PCM long frame | `I2SSTD=11`, `PCMSYNC=1` | |

Detailed timing diagrams in the original PDF (Figures 23-4 through 23-14).

### 23.3.3 Clock generator

I2S bitrate determines the data rate and serial clock frequency:

```
I2S bitrate = bits_per_channel × num_channels × sample_rate (Fs)
```

For stereo 16-bit:  `bitrate = 16 × 2 × Fs`
For stereo 32-bit:  `bitrate = 32 × 2 × Fs`

**Clock source**: `I2SxCLK` comes from system clock tree — selectable between `SYSCLK` or `PLL_CLK` (and on some chip variants `PLL3 VCO`, i.e. `2 × PLL3CLK`). Selection is via `RCC_CFGR2`:
- `I2S2SRC` bit → I2S2 source (0 = SYSCLK, 1 = PLL_CLK)
- `I2S3SRC` bit → I2S3 source (0 = SYSCLK, 1 = PLL_CLK)

**Sample rate formula** (master mode):

| Channel length | MCLK output | Formula |
|----------------|-------------|---------|
| 16-bit | enabled (MCKOE=1) | `Fs = I2SxCLK / [(16*2) * ((2*I2SDIV) + ODD) * 8]` |
| 32-bit | enabled (MCKOE=1) | `Fs = I2SxCLK / [(32*2) * ((2*I2SDIV) + ODD) * 4]` |
| 16-bit | disabled (MCKOE=0) | `Fs = I2SxCLK / [(16*2) * ((2*I2SDIV) + ODD)]` |
| 32-bit | disabled (MCKOE=0) | `Fs = I2SxCLK / [(32*2) * ((2*I2SDIV) + ODD)]` |

Where:
- `I2SDIV[7:0]` — 8-bit linear divider value, in `I2SPR` register
- `ODD` — 1-bit fractional add, in `I2SPR` register
- Effective divider = `(2 × I2SDIV) + ODD` → can produce odd integer divisions
- MCKOE multiplier (×8 for 16-bit, ×4 for 32-bit) keeps MCK at 256 × Fs

### 23.3.4 I2S Master mode

Master generates serial clock (CK), word select (WS), and optionally MCK.

#### 23.3.4.1 Configuration steps

1. Set `I2SDIV[7:0]` and `ODD` in `I2SPR` to match target Fs (use formula above).
2. Set `CKPOL` to define idle clock level.
3. If MCK output is needed for external DAC/ADC, set `MCKOE = 1` in `I2SPR`.
4. Set `I2SMOD = 1` in `I2SCFGR` to enable I2S mode.
5. Set `I2SSTD[1:0]` + `PCMSYNC` to choose audio standard.
6. Set `CHLEN` to choose channel data length (16 or 32 bit).
7. Set `I2SCFG[1:0]` to choose master mode + direction (TX or RX).
8. Optionally configure interrupts/DMA via `CR2`.
9. Set `I2SE = 1` in `I2SCFGR` to start.
10. Configure pins WS and CK as alternate function output. If `MCKOE=1`, also configure MCK pin.

#### 23.3.4.2 Transmit flow

Writing a 16-bit half-word to TX buffer starts transmission. Assume first write = left channel. When data moves from buffer to shift register, `TXE` flag = 1 → write right channel data. The `CHSIDE` flag tells you which channel is currently in transit (valid when `TXE = 1`).

A complete frame requires both L and R data. Partial frames (e.g. left only) are not allowed.

MSB-first transmission via MOSI/SD pin. `TXE = 1` triggers IRQ if `TXEIE` in `CR2` is set.

For continuous audio, write next sample to `DATAR` before current transmission completes. To disable I2S cleanly: wait for `TXE = 1` AND `BSY = 0`, then clear `I2SE`.

#### 23.3.4.3 Receive flow

Same as TX except `I2SCFG[1:0]` selects master RX. Data always received in 16-bit packets — for 24/32-bit data this requires 2 buffer reads per sample.

`RXNE = 1` after each buffer fill → triggers IRQ if `RXNEIE` in `CR2` is set. Reading `DATAR` clears `RXNE`. `OVR` flag = 1 if new data arrives before previous read → IRQ if `ERRIE` is set.

To disable cleanly:
- 16→32 expansion + LSB-aligned: wait for (n-1)-th `RXNE = 1`, wait 17 I2S clock cycles, clear `I2SE`
- 16→32 expansion + MSB/I2S/PCM: wait for last `RXNE = 1`, wait 1 I2S clock cycle, clear `I2SE`
- All other combos: wait (n-1)-th `RXNE = 1`, wait 1 I2S clock cycle, clear `I2SE`

Note: `BSY` stays low during transmission.

### 23.3.5 I2S Slave mode

Slave can also be TX or RX. Configuration matches master except no clock setup is needed — both CK and WS come from external master.

Steps:
1. Set `I2SMOD = 1` to activate I2S.
2. Set `I2SSTD[1:0]` for standard.
3. Set `DATLEN[1:0]` for data bit count, `CHLEN` for channel length.
4. Set `I2SCFG[1:0]` for slave mode + direction.
5. Configure interrupts/DMA via `CR2` if needed.
6. Set `I2SE = 1`.

---

## 23.4 Register map (overview only)

Full register layouts (offsets, bit fields, reset values) are in the original PDF starting around page 410, with bit-field names already in English.

| Register | Function |
|----------|----------|
| `SPI_CTLR1` | SPI control 1 (BR[2:0], CPHA, CPOL, MSTR, SPE, etc.) |
| `SPI_CTLR2` | SPI/I2S control 2 (TXEIE, RXNEIE, ERRIE, SSOE, TXDMAEN, RXDMAEN) |
| `SPI_STATR` | Status (BSY, OVR, MODF, CRCERR, TXE, RXNE, CHSIDE) |
| `SPI_DATAR` | Data register (shared SPI/I2S) |
| `SPI_CRCR` | SPI CRC polynomial (SPI mode only) |
| `SPI_RCRCR` | RX CRC value |
| `SPI_TCRCR` | TX CRC value |
| `SPI_I2SCFGR` | I2S config (I2SMOD, I2SE, I2SCFG[1:0], PCMSYNC, I2SSTD[1:0], CKPOL, DATLEN[1:0], CHLEN) |
| `SPI_I2SPR` | I2S prescaler (MCKOE, ODD, I2SDIV[7:0]) |

---

## Quick recipe — Master TX, stereo, 24-bit, audio DAC

```c
/* 1. Enable clocks */
RCC_HB1PeriphClockCmd(RCC_HB1Periph_SPI2, ENABLE);
RCC_HB2PeriphClockCmd(RCC_HB2Periph_GPIOB | RCC_HB2Periph_GPIOC | RCC_HB2Periph_AFIO, ENABLE);

/* 2. Pick I2S clock source — PLL_CLK preferred for accuracy */
RCC->CFGR2 |= (1 << 24);   /* I2S2SRC = 1 → PLL_CLK */

/* 3. Configure pins (I2S2 example) */
GPIO_PinAFConfig(GPIOB, GPIO_PinSource12, GPIO_AF5);  /* WS  */
GPIO_PinAFConfig(GPIOB, GPIO_PinSource13, GPIO_AF5);  /* CK  */
GPIO_PinAFConfig(GPIOC, GPIO_PinSource1,  GPIO_AF5);  /* SD  */
/* (configure as AF push-pull, very high speed) */

/* 4. I2S config */
I2S_InitTypeDef cfg = {
    .I2S_Mode       = I2S_Mode_MasterTx,
    .I2S_Standard   = I2S_Standard_Phillips,
    .I2S_DataFormat = I2S_DataFormat_24b,
    .I2S_MCLKOutput = I2S_MCLKOutput_Disable, /* or _Enable + extra MCK pin */
    .I2S_AudioFreq  = I2S_AudioFreq_192k,    /* HAL ceiling; raw regs go higher */
    .I2S_CPOL       = I2S_CPOL_Low,
};
I2S_Init(SPI2, &cfg);

/* 5. Wire DMA on TX */
SPI_I2S_DMACmd(SPI2, SPI_I2S_DMAReq_Tx, ENABLE);

/* 6. Start */
I2S_Cmd(SPI2, ENABLE);
```

For external DACs needing MCLK (e.g. ES9018), set `I2S_MCLKOutput_Enable` and route the additional MCK pin (chip-specific, check pinmux table in datasheet).

PCM5102A works **without MCK** thanks to its internal PLL — use `_Disable`.

---

## Tradução pendente

This translation covers I2S sections that matter for an audio playback project. **Not yet translated** from this chapter:
- 23.2 SPI Functional Description (lines 64–229 of `_extracted/ch23_spi_i2s_cn.txt`)
- 23.3.6+ I2S DMA modes detail
- Full register bit-by-bit reset values

Generate on demand from `_extracted/ch23_spi_i2s_cn.txt` using the workflow in [CH32H417_ReferenceManual_EN_TOC.md](CH32H417_ReferenceManual_EN_TOC.md#on-demand-translation-workflow).
