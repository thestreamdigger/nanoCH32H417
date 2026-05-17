# CH32H417 Reference Manual — English TOC

Index for `CH32H417_ReferenceManual_CN.PDF` V1.7 (879 pages, Chinese-only). Manual covers CH32H417, H416, H415. Device characteristics (electrical, packages) → `CH32H417_Datasheet_EN.PDF`.

### Core overview

| Core | ISA | Stack levels | HW IRQ nesting | Fast IRQ channels | Pipeline | Ext. instructions | I-cache | Memory protection | Debug |
|------|-----|--------------|----------------|-------------------|----------|-------------------|---------|-------------------|-------|
| QingKe V3F | IMABCF-X | 2 | 2 | 4 | 3 stage | addr or instr vec | yes | — | 1/2-wire |
| QingKe V5F | IMABCF-X | 8 | 8 | 4 | 7–9 stage | addr or instr vec | yes | yes¹ | 1/2-wire |

¹ Only chips with batch code 5th digit ≠ 0 support memory protection.

### Register bit attribute legend

| Code | Meaning |
|------|---------|
| RF | Read-only, fixed value |
| RO | Read-only, changed by hardware |
| RZ | Read-only, auto-cleared after read |
| WO | Write-only (unreadable) |
| WA | Write-only, writable in secure mode |
| WZ | Write-only, auto-cleared after write |
| RW | Read+Write |
| RWA | Read+Write in secure mode |
| RW1 | Read, write 1 to set |
| RW0 | Read, write 0 to set |
| RW1T | Read, write 1 to toggle |
| RW1Z | Read, write 1 to clear |
| SC | Self-clearing |

## Chapter index

Line numbers point into `_extracted/rm_full.txt` (50,036 lines).

| # | EN title | Original (CN) | Line | Notes |
|---|----------|---------------|------|-------|
| 1 | Memory and Bus Architecture | 存储器和总线架构 | 50 | dual-core bus matrix, DMA, SRAM map |
| 2 | Power Control (PWR) | 电源控制 | 614 | LDO, BOR, PVD, sleep/stop/standby |
| 3 | Reset and Clock Control (RCC) | 复位和时钟控制 | 935 | **PLL, clock tree — read for I2S setup** |
| 4 | Central Processing Unit (CPU) | 中央处理器 | 2782 | V5F + V3F cores, PFIC, HSEM, IPC |
| 5 | CRC | 循环冗余校验 | 6536 | hardware CRC32 |
| 6 | Real-Time Clock (RTC) | 实时时钟 | 6641 | calendar, alarm, tamper |
| 7 | Independent Watchdog (IWDG) | 独立看门狗 | 6890 | LSI-clocked |
| 8 | Windowed Watchdog (WWDG) | 窗口看门狗 | 7032 | HSI-clocked |
| 9 | GPIO and Alternate Functions (GPIO/AFIO) | GPIO 及其复用功能 | 7208 | AF mux, pin remap |
| 10 | DMA Controller | 直接存储器访问控制 | 8454 | multi-channel DMA |
| 11 | ADC | 模拟/数字转换 | 9260 | dual 12-bit 5 Msps |
| 12 | High-Speed ADC (HSADC) | 高速模拟/数字转换 | 10650 | 10-bit 20 Msps |
| 13 | TouchKey Detection (TKEY) | 触摸按键检测 | 10915 | 16-channel capacitive |
| 14 | Advanced Timer (ADTM) | 高级定时器 | 11050 | motor control, dead-time |
| 15 | General-Purpose Timer (GPTM) | 通用定时器 | 12695 | PWM, capture, encoder |
| 16 | Basic Timer (BCTM) | 基本定时器 | 14241 | simple time base |
| 17 | Low-Power Timer (LPTIM) | 低功耗定时器 | 14492 | wakeup-capable |
| 18 | DAC | 数字/模拟转换 | 15331 | dual 12-bit |
| 19 | Universal Sync/Async Receiver-Transmitter (USART) | 通用同步异步收发器 | 15984 | UART/USART/SmartCard/IrDA |
| 20 | Serial/Parallel Converter + Transceiver (SerDes) | 串并互转控制器及收发器 | 16788 | high-speed isolated link |
| 21 | I2C (Two-Wire Communication Bus) | 两线通信总线 | 17238 | 100k/400k/Fast+ |
| 22 | I3C Bus | I3C 总线 | 18184 | MIPI I3C |
| 23 | Serial Peripheral Interface (SPI/I2S) | 串行外设接口 | 22779 | **SPI + I2S — read for audio project** |
| 24 | USB PD Controller (USBPD) | USB PD 控制器 | 23986 | Type-C power delivery |
| 25 | USB High-Speed Host/Device (USBHS) | USB 高速主机/设备控制器 | 24301 | 480 Mbps USB 2.0 HS |
| 26 | USB Full-Speed Host/Device (USBFS/OTG_FS) | USB 全速主机/设备控制器 | 25395 | 12 Mbps USB 2.0 FS |
| 27 | USB 3.0 SuperSpeed Host/Device (USBSS) | USB 3.0 超高速主机/设备控制器 | 26313 | 5 Gbps — flagship feature |
| 28 | Controller Area Network (CAN) | 控制器局域网 | 28431 | 3× CAN 2.0 B / CAN FD |
| 29 | Digital Camera Interface (DVP) | 数字图像接口 | 30104 | parallel camera input |
| 30 | Random Number Generator (RNG) | 随机数发生器 | 30584 | true RNG |
| 31 | Ethernet (ETH) | 以太网收发器 | 30725 | 10/100M MAC + PHY |
| 32 | SDIO Interface | SDIO 接口 | 34165 | SD card 1/4-bit |
| 33 | SD/eMMC Controller (SDMMC) | SD/EMMC 控制器 | 35538 | eMMC + SD, DDR mode |
| 34 | Programmable Protocol I/O Microcontroller (PIOC) | 可编程协议 I/O 微控制器 | 36360 | custom protocol bit-bang |
| 35 | Op-Amp (OPA) and Comparator (CMP) | 运算放大器和比较器 | 36821 | analog amplifiers |
| 36 | Serial Audio Interface (SAI) | 串行音频接口 | 37258 | TDM/I2S/AC97 audio |
| 37 | QSPI Interface (QuadSPI) | QSPI 接口 | 38812 | quad-SPI for ext. flash |
| 38 | Single-Wire Protocol Master (SWPMI) | 单线协议主接口 | 39939 | smartcard, NFC |
| 39 | Electronic Signature (ESIG) | 电子签名 | 41122 | unique chip ID, factory data |
| 40 | Flexible Memory Controller (FMC) | 灵活存储控制器 | 41195 | external SRAM/NOR/NAND/SDRAM |
| 41 | Cryptographic Module (ECDC) | 加密模块 | 44563 | AES, DES, hash |
| 42 | Digital Filter for Sigma-Delta Modulators (DFSDM) | 数字滤波器 ΣΔ 调制器 | 44896 | high-res sigma-delta input |
| 43 | LCD-TFT Display Controller (LTDC) | LCD-TFT 显示控制器 | 47072 | RGB parallel LCD |
| 44 | Graphics Hardware Accelerator (GPHA) | 图形处理硬件加速器 | 47961 | 2D blit, alpha blend |
| 45 | Debug Support (DBG) | 调试支持 | 49120 | DBGMCU, ITM |
| 46 | Flash Memory and User Option Bytes (FLASH) | 闪存及用户选择字 | 49283 | program/erase, option bytes |
| 47 | Universal High-Speed Interface (UHSIF) | 通用高速接口 | 50027 | 500 MB/s WCH proprietary link |

## Extract a chapter on demand

```bash
awk 'NR>=22779 && NR<23986' doc/MCU/_extracted/rm_full.txt > /tmp/chapter.txt
```

Register tables are already English in the PDF — only narrative needs translation.
