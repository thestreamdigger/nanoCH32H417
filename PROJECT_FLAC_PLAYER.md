# Project — FLAC USB → I2S Player

Pipeline:
```
USB pendrive → USB HS Host → CHRV3UFI (MSC+FAT32) → FLAC decoder → I2S DMA → PCM5102A DAC → Line-out
```

This file is the **bridge from docs to code**. Every decision below is grounded in this fork's documentation.

## Hardware bill of materials

| Item | Source | Cost | Notes |
|------|--------|------|-------|
| nanoCH32H417 v1.0 | MuseLab (Aliexpress/Tindie) | ~US$18 | this board |
| USB pendrive FAT32 | any | varies | format as FAT32, ≤32 GB recommended |
| USB-A to USB-A male/female cable | any | — | the board's USB-3.0-SS Type-A is the host port |
| PCM5102A I2S DAC module | Aliexpress | ~US$5 | standard breakout, has 3.5 mm jack |
| 4 jumper wires | any | — | DAC: VCC, GND, BCK, LCK, DIN |
| (optional) Crystek CCHD-957 12.288 MHz | Digikey | ~US$30 | for audiophile clock — bypass MCU jitter |

## Pin assignment

Verified via [PINOUT.md](PINOUT.md) + [Ch09_GPIO_AFIO_EN](doc/MCU/CH32H417_RM_Ch09_GPIO_AFIO_EN.md) + [Ch23_SPI_I2S_EN](doc/MCU/CH32H417_RM_Ch23_SPI_I2S_EN.md).

| Function | Silkscreen | MCU pin | AF | Where |
|----------|------------|---------|-----|-------|
| I2S2 WS (LRCK) | LCD FPC pin 8 (CS) | PB12 | AF5 | FPC connector — **conflicts with LCD use** |
| I2S2 CK (BCK)  | LCD FPC pin 9 (SCK) | PB13 | AF5 | FPC connector |
| I2S2 SD (DATA) | J10:C1 | PC1 | AF5 | bottom-right header |
| (USB HS) D+ | dedicated | PB8 | — | Type-A connector |
| (USB HS) D- | dedicated | PB9 | — | Type-A connector |
| 3V3 (DAC supply) | J6:3V3 | — | — | top-right header |
| GND | J5:GND or J10:GND | — | — | corners |

Wire the PCM5102A:
- VIN ← 3V3 (J6)
- GND ← GND (J5/J10)
- BCK ← PB13 (LCD FPC pin 9 / break the FPC and pull this signal out, OR don't use LCD)
- LCK ← PB12 (LCD FPC pin 8)
- DIN ← PC1 (J10:C1)
- SCK (DAC) ← GND (PCM5102A uses internal PLL, no MCK needed)
- FMT ← GND (I2S mode)
- DEMP ← GND (de-emphasis off)
- XSMT → 3V3 (un-mute, after init)

If you also want the LCD, switch I2S2 → I2S3 (PA13/PA14/PA15 path) and cut SB5/SB6 to free PA13/PA14 from the WCHLink-E. Trade-off: lose onboard SWD debug, must use external SWD via the WCHLink-E pads.

## Software stack

All ingredients exist in this fork:

| Layer | Source | Purpose |
|-------|--------|---------|
| Startup + Linker | `doc/EVT/EXAM/SRC/Startup/`, `doc/EVT/EXAM/SRC/Ld/V5F/` | Boot + memory map |
| Peripheral HAL | `doc/EVT/EXAM/SRC/Peripheral/` | RCC, GPIO, SPI/I2S, DMA drivers |
| USB Host (HW) | `doc/EVT/EXAM/USBHS/HOST/Host_UDisk_Exams/Common/USB_Host/` | USBHS controller driver |
| USB Host (FS lib) | `doc/EVT/EXAM/USBHS/HOST/Udisk_Lib/libRV3UFI.a` + `CHRV3UFI.h` | MSC + FAT32, [API in CHRV3UFI_Library_EN.md](doc/MCU/CHRV3UFI_Library_EN.md) |
| FLAC decoder | `dr_flac.h` (single header, public domain, fetch separately) | software decode |
| Application | new code, this project | glue |

## Reference docs (in this fork)

| Topic | Doc |
|-------|-----|
| Clock setup, PLL | [CH32H417_RM_Ch03_RCC_EN.md](doc/MCU/CH32H417_RM_Ch03_RCC_EN.md) |
| GPIO + AF | [CH32H417_RM_Ch09_GPIO_AFIO_EN.md](doc/MCU/CH32H417_RM_Ch09_GPIO_AFIO_EN.md) |
| DMA (for I2S TX) | [CH32H417_RM_Ch10_DMA_EN.md](doc/MCU/CH32H417_RM_Ch10_DMA_EN.md) |
| I2S TX, formats, recipe | [CH32H417_RM_Ch23_SPI_I2S_EN.md](doc/MCU/CH32H417_RM_Ch23_SPI_I2S_EN.md) |
| USB Host + FAT lib | [CHRV3UFI_Library_EN.md](doc/MCU/CHRV3UFI_Library_EN.md) |
| Datasheet (electrical, pin AF table) | [doc/MCU/CH32H417_Datasheet_EN.PDF](doc/MCU/CH32H417_Datasheet_EN.PDF) |
| Working I2S DMA example | `doc/EVT/EXAM/I2S/I2S_DMA/V5F/` |
| Working USB UDisk example | `doc/EVT/EXAM/USBHS/HOST/Host_UDisk_Exams/V5F/` |

## Architecture

```
                            ┌──────────────────────────────────┐
                            │            V5F (400 MHz)         │
                            │  ┌──────────┐  ┌──────────────┐  │
USB pendrive ─[USBHS]──────►│  │ USB Host │→ │ FLAC decoder │  │
                            │  │  + CHRV3 │  │  (dr_flac)   │  │
                            │  │  + FAT32 │  └──────┬───────┘  │
                            │  └──────────┘         │          │
                            │                       ▼          │
                            │              ┌────────────────┐  │
                            │              │  ring buffer   │  │
                            │              │ (stereo 24-bit)│  │
                            │              └────────┬───────┘  │
                            │                       │          │
                            │                ┌──────▼──────┐   │
                            │ DMA1_Ch5 ◄─────│  I2S2 (SPI2)│   │
                            │                └──────┬──────┘   │
                            └───────────────────────│──────────┘
                                                    │ BCK + LCK + DIN
                                                    ▼
                                              PCM5102A → 3.5mm jack
```

## Boot sequence

```c
int main(void) {
    SystemAndCoreClockUpdate();       // 1. lock clocks (HSE 25 MHz → PLL → 400 MHz)
    Delay_Init();
    USART_Printf_Init(115200);        // 2. debug UART via WCHLink-E
    printf("[INFO] CH32H417 FLAC player V5F %d MHz\r\n", SystemCoreClock / 1000000);

    /* 3. GPIO for I2S2 (PB12/PB13/PC1, AF5) — see Ch09 */
    init_i2s2_pins();

    /* 4. I2S2 master TX, 24-bit, 44.1 kHz initial — see Ch23 */
    init_i2s2(I2S_AudioFreq_44k, I2S_DataFormat_24b);

    /* 5. DMA1_Ch5 for I2S2 TX, circular, HT+TC IRQ — see Ch10 */
    init_dma_i2s_tx();

    /* 6. USB Host + CHRV3UFI — see CHRV3UFI doc */
    init_usb_host();
    CHRV3vPacketSize = 512;
    CHRV3LibInit();

    /* 7. Wait for pendrive */
    while (CHRV3DiskStatus < DISK_MOUNTED) {
        UDisk_USBH_DiskReady();
    }
    printf("[OK] disk ready, FAT%d\r\n", CHRV3vDiskFat);

    /* 8. Find first .flac file in root */
    char filename[64];
    if (find_first_flac(filename) != 0) {
        printf("[ERR] no flac files\r\n");
        while (1);
    }

    /* 9. Open + stream */
    strcpy((char *)mCmdParam.Open.mPathName, filename);
    if (CHRV3FileOpen() != ERR_SUCCESS) {
        printf("[ERR] cannot open %s\r\n", filename);
        while (1);
    }
    printf("[OK] playing %s (%lu bytes)\r\n", filename, CHRV3vFileSize);

    play_loop();   // until EOF or disk removed
}
```

## Critical path — `play_loop` and the DMA refill ISR

This is where the engineering happens.

```c
#define I2S_BUF_FRAMES    1024                            // half-buffer size
#define I2S_BUF_SAMPLES   (I2S_BUF_FRAMES * 2)            // stereo
static uint16_t i2s_buf[I2S_BUF_SAMPLES * 2];             // double-buffered

static volatile int half_to_fill = -1;                    // -1 = none, 0 = first, 1 = second

void DMA1_Channel5_IRQHandler(void) {
    if (DMA_GetITStatus(DMA1_IT_HT5)) {
        DMA_ClearITPendingBit(DMA1_IT_HT5);
        half_to_fill = 0;
    }
    if (DMA_GetITStatus(DMA1_IT_TC5)) {
        DMA_ClearITPendingBit(DMA1_IT_TC5);
        half_to_fill = 1;
    }
}

static void play_loop(void) {
    drflac *flac = drflac_open_file(filename, NULL);      // ← needs file callback bridging to CHRV3
    if (!flac) { printf("[ERR] flac open\r\n"); return; }

    /* Adjust I2S sample rate to file */
    set_i2s_sample_rate(flac->sampleRate);
    printf("[OK] %luHz %dch %dbit\r\n", flac->sampleRate, flac->channels, flac->bitsPerSample);

    /* Prefill both halves */
    drflac_read_pcm_frames_s16(flac, I2S_BUF_FRAMES * 2, (int16_t *)i2s_buf);

    /* Start DMA */
    DMA_Cmd(DMA1_Channel5, ENABLE);
    SPI_I2S_DMACmd(SPI2, SPI_I2S_DMAReq_Tx, ENABLE);
    I2S_Cmd(SPI2, ENABLE);

    /* Refill loop */
    for (;;) {
        if (half_to_fill == 0) {
            drflac_read_pcm_frames_s16(flac, I2S_BUF_FRAMES,
                                       (int16_t *)i2s_buf);
            half_to_fill = -1;
        }
        if (half_to_fill == 1) {
            drflac_read_pcm_frames_s16(flac, I2S_BUF_FRAMES,
                                       (int16_t *)(i2s_buf + I2S_BUF_SAMPLES));
            half_to_fill = -1;
        }
        /* TODO: handle EOF, user input, disk removal */
    }

    drflac_close(flac);
}
```

## Open issues / decisions

1. **Sample rate switching mid-playback**: when one file is 44.1 and next is 96, must re-init I2S clock. `I2S_Cmd(SPI2, DISABLE)` → reconfigure → `I2S_Cmd(SPI2, ENABLE)` is the path. Audible glitch ~10 ms.
2. **24-bit vs 16-bit decoding**: `dr_flac` exposes `drflac_read_pcm_frames_s32` for true 24/32-bit. Use that for hi-res — pack into 32-bit I2S slots, set `I2S_DataFormat_24b` or `_32b`.
3. **Concurrent DMA**: I2S TX (DMA1_Ch5) and USB transfers (separate DMA channels) can run in parallel — both have hardware DMA engines, just verify arbitration priority.
4. **Buffer size**: 1024 frames × 2 ch × 2 bytes = 4 KB half-buffer = ~23 ms @ 44.1 kHz / ~5 ms @ 192 kHz. Increase if FLAC decode jitter causes underruns at high sample rates.
5. **Power**: PCM5102A consumes ~30 mA; comfortable on the 3V3 LDO (rated 500 mA).
6. **Volume control**: PCM5102A is hardware-fixed; software volume in the s16/s32 stream is the cleanest approach (multiply samples in the refill path).
7. **UI**: no display in this v1 — playback is "insert pendrive, plays first .flac, repeat on EOF". Future: re-enable LCD on I2S3 path + UI from the IAP button.

## Build

See [BUILD.md](BUILD.md). Project layout suggestion:

```
flac-usb-i2s/
├── Makefile
├── User/
│   ├── main.c
│   ├── audio_pipe.c       # I2S init + DMA refill
│   ├── usb_host.c         # CHRV3 init + file iteration
│   ├── flac_io.c          # dr_flac ↔ CHRV3 byte read bridge
│   └── system_ch32h417.c  # from EXAM
├── lib/
│   ├── dr_flac.h          # fetch from https://github.com/mackron/dr_libs
│   └── libRV3UFI.a        # copy from EXAM/USBHS/HOST/Udisk_Lib
└── (build/)
```

## Next steps when board arrives

1. **Validate environment**: install xPack RISC-V GCC + wlink (BUILD.md), flash GPIO_Toggle to confirm toolchain works.
2. **Run I2S_DMA example unmodified** on V5F. Hook PCM5102A. Validate audio with a fixed sine wave.
3. **Run Host_UDisk_Exams** unmodified. Validate FAT32 read with a known text file.
4. **Combine**: skeleton main.c per above. First milestone — play a known 44.1/16 PCM stream from a file. Then add FLAC decode. Then hi-res.

## Memory budget vs Datasheet limits

| Resource | Used | Available | Margin |
|----------|------|-----------|--------|
| Flash | est. 80 KB (StdPeriph + CHRV3 + dr_flac + glue) | 960 KB | 90 % free |
| SRAM data | est. 50 KB (buffers + lib state) | 896 KB | 94 % free |
| ITCM | 0 | 128 KB | reserved for hot paths if needed |
| DTCM | 0 | 256 KB | reserved for low-latency buffers if needed |

Plenty of headroom for adding LCD + UI + EQ later.
