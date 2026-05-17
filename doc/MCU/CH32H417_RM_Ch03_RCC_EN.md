# Chapter 3 — Reset and Clock Control (RCC)

Translated summary of `CH32H417_ReferenceManual_CN.PDF` V1.7, lines 935–2781 of `_extracted/rm_full.txt`. Original Chinese: `_extracted/ch03_rcc_cn.txt`.

Diagrams (Figure 3-N) are in the original PDF.

This chapter is huge (1847 lines). This translation covers the clock-tree fundamentals + the I2S/audio-relevant sections. For full register layouts, see the original PDF starting around page 30.

---

## 3.1 Key features

- Multiple reset sources
- Multiple clock sources with bus-level management
- HSE oscillator monitoring + Clock Security System (CSS)
- Independent reset/enable/disable per peripheral
- Internal clock output capability

---

## 3.2 Reset

Three reset types:

### 3.2.1 Power reset
Resets all registers except the backup domain (VBAT-powered).
Triggered by: POR (power-on) or PDR (power-down).

### 3.2.2 System reset
Resets all registers except `RCC_RSTSCKR` reset flags and backup domain. Check flags in `RCC_RSTSCKR` to identify source.

Triggers:
- Low level on NRST pin (external)
- WWDG counter end
- IWDG counter end
- Software reset (`SYSRST` bit in `PFIC_CFGR` or `PFIC_SCTLR`)
- USBPD reset (Hard Reset or Cable Reset when enabled)

### 3.2.3 Backup domain reset
Resets only backup domain registers (`RCC_BDCTLR` — RTC enable, LSE oscillator).
Triggers:
- VDD33 or VBAT power-on after both have been off
- `BDRST` bit set in `RCC_BDCTLR`

---

## 3.3 Clocks

### 3.3.1 System clock structure (Figure 3-2 / 3-3)

The clock tree is large. Key sources:

| Source | Frequency | Notes |
|--------|-----------|-------|
| **LSI RC** | 40 kHz | drives IWDG, LPTIM1/2 |
| **LSE OSC** | 32.768 kHz | external crystal (X32KI/X32KO — not populated on nanoCH32H417); drives RTC |
| **HSI RC** | 25 MHz¹ | internal, drives IWDG/LPTIM, fallback for CSS |
| **HSE OSC** | 25 MHz | external crystal (X2 on nanoCH32H417); drives SYS PLL, USB PLLs, ETH PLL, SerDes PLL |

¹ HSI frequency typically 25 MHz on H417; verify in datasheet electrical chapter.

### Multiple PLLs (one per high-speed peripheral)

| PLL | Purpose | Output |
|-----|---------|--------|
| **SYS PLL** | system clock (HCLK, CPU, peripherals) | `PLL_CLK` |
| **USBHS PLL** | USB 2.0 High-Speed | `USBHS_PLL_CLK` |
| **USBSS PLL** | USB 3.0 SuperSpeed | `USBSS_PLL_CLK` (125 MHz typ) |
| **ETH PLL** | Ethernet MAC/PHY | `ETH_PLL_CLK` (500 MHz / 20 MHz / 100 MHz / 125 MHz taps) |
| **SerDes PLL** | SerDes link | `SERDES_PLL_CLK` |

### SYS PLL — main system clock

```
HSE (25 MHz) → PLL_SRC_DIV (/1..../64) → PLLSRC mux → SYS PLL → PLLMUL (×4..×59 incl. ×8.5, ×9) → PLL_CLK
                                                                                              ↓
HSI (25 MHz) ──────────────────────────────────────────────→ SYSPLL_SEL mux → SYSCLK
                                                                                              ↑
                                                                                              HSE direct, HSI direct
```

**Note `×8.5` in the multiplier list** — fractional multiplier is supported. This is what makes 44.1 kHz family accurate on I2S (no PPM drift).

`PLLMUL` covers: ×4, ×6, ×7, ×8, **×8.5**, ×9, ..., ×59.

`HPRE` (HB prescaler) divides SYSCLK to produce `HCLK`: /1, /2, /4, ... /512 — feeds CPU, memory, DMA, HB bus.

### 3.3.6 I2S clock source

> Translation of lines 494–496:
> "The clock source for I2S and RNG comes from `PLL_CLK` or system clock (`SYSCLK`). I2S2, I2S3, and RNG can independently select the source via `I2S2SRC`, `I2S3SRC`, and `RNGSRC` bits of `RCC_CFGR2`."

Register `RCC_CFGR2`:

| Bit | Name | Function |
|-----|------|----------|
| 25 | `I2S3SRC` | 0 = SYSCLK, 1 = PLL_CLK |
| 24 | `I2S2SRC` | 0 = SYSCLK, 1 = PLL_CLK |
| 23 | `RNGSRC`  | 0 = SYSCLK, 1 = PLL_CLK |

**For audio**: always set `I2S2SRC = 1` (or `I2S3SRC = 1`) to use PLL_CLK — gives the cleanest, most stable I2S clock.

### 3.3.6 Clock Security System (CSS)

Switches automatically from HSE to HSI on HSE failure, fires an interrupt. Application can take corrective action.

---

## Audio-project recipe — clock setup

Goal: 24-bit/192 kHz I2S out using HSE 25 MHz crystal.

```
HSE 25 MHz
  ↓ PLL_SRC_DIV /1
  ↓ × PLLMUL (chosen to hit a frequency that divides cleanly to 192 kHz × 256 = 49.152 MHz family)
  → PLL_CLK
  ↓ I2S2SRC = 1
  → I2SxCLK
  ↓ /[(32*2) * ((2*I2SDIV) + ODD)]   for 32-bit channel, MCK off
  → Fs = 192 kHz
```

For 48k-family target (48/96/192 kHz):
- Need PLL_CLK = N × 12.288 MHz for clean division
- Typical: PLL_CLK = 245.76 MHz (= 12.288 × 20) — requires PLL math with HSE=25 MHz: not a clean integer
- Alternative: PLL_CLK = 196.608 MHz (= 12.288 × 16) — same issue with 25 MHz HSE

The accurate way needs a 24.576 MHz or 22.5792 MHz reference crystal — which the board does **not** have. With HSE=25 MHz and PLL only, you'll typically be a few hundred ppm off ideal Fs. **Acceptable for casual playback, not audiophile spec.**

For audiophile spec: add an external low-jitter audio clock (Crystek CCHD-957 or similar) feeding the I2S MCK directly, bypass the chip's clock generator for I2S.

### 44.1k family (44.1 / 88.2 / 176.4)
- Need PLL_CLK = N × 11.2896 MHz
- With ×8.5 fractional PLLMUL and HSE=25 MHz: closest fits are off by a few hundred ppm
- Same conclusion: external clock needed for true audiophile accuracy

Detailed register layouts (`RCC_CR`, `RCC_CFGR`, `RCC_CFGR2`, `RCC_CFGR3`, peripheral enable bits) — original PDF or `_extracted/ch03_rcc_cn.txt`.
