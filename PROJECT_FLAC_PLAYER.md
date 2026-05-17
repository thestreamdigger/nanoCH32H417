# PROJECT — FLAC USB → I2S Player

```
USB pendrive → USBHS Host → CHRV3UFI (MSC+FAT32) → dr_flac → I2S DMA → PCM5102A → 3.5mm
```

## Pinout (I2S2 path)

| Function | MCU | AF | Silk |
|----------|-----|-----|-------|
| WS (LRCK) | PB12 | AF5 | LCD FPC pin 8 |
| CK (BCK)  | PB13 | AF5 | LCD FPC pin 9 |
| SD (DATA) | PC1  | AF5 | J10:C1 |
| USB HS D+ / D- | PB8 / PB9 | — | Type-A connector |
| 3V3 / GND | — | — | J6:3V3, J5:GND |

PB12/PB13 conflict with LCD. If LCD needed, use I2S3 (PA13/14/15) and cut SB5/SB6 to free SWD pins.

## PCM5102A wiring

| DAC | Source |
|-----|--------|
| VIN | 3V3 |
| GND | GND |
| BCK | PB13 |
| LCK | PB12 |
| DIN | PC1 |
| SCK | GND (internal PLL) |
| FMT, DEMP | GND |
| XSMT | 3V3 (unmute after init) |

## Software stack

| Layer | Source |
|-------|--------|
| Startup, linker | `doc/EVT/EXAM/SRC/{Startup,Ld/V5F}/` |
| Peripheral HAL | `doc/EVT/EXAM/SRC/Peripheral/` |
| USB Host driver | `doc/EVT/EXAM/USBHS/HOST/Host_UDisk_Exams/Common/USB_Host/` |
| USB MSC + FAT32 | `doc/EVT/EXAM/USBHS/HOST/Udisk_Lib/libRV3UFI.a` |
| FLAC decoder | `dr_flac.h` (fetch from mackron/dr_libs) |

## Reference docs

| Topic | Path |
|-------|------|
| Clock / PLL | `doc/MCU/CH32H417_RM_Ch03_RCC_EN.md` |
| GPIO / AF | `doc/MCU/CH32H417_RM_Ch09_GPIO_AFIO_EN.md` |
| DMA | `doc/MCU/CH32H417_RM_Ch10_DMA_EN.md` |
| I2S | `doc/MCU/CH32H417_RM_Ch23_SPI_I2S_EN.md` |
| USB+FAT lib | `doc/MCU/CHRV3UFI_Library_EN.md` |
| Datasheet | `doc/MCU/CH32H417_Datasheet_EN.PDF` |
| I2S DMA reference example | `doc/EVT/EXAM/I2S/I2S_DMA/V5F/` |
| USB UDisk reference example | `doc/EVT/EXAM/USBHS/HOST/Host_UDisk_Exams/V5F/` |

## Boot sequence

```c
int main(void) {
    SystemAndCoreClockUpdate();
    USART_Printf_Init(115200);

    init_i2s2_pins();
    init_i2s2(I2S_AudioFreq_44k, I2S_DataFormat_24b);
    init_dma_i2s_tx();             /* DMA1_Ch5, circular, HT+TC IRQ */

    init_usb_host();
    CHRV3vPacketSize = 512;        /* HS / SS device */
    CHRV3LibInit();

    while (CHRV3DiskStatus < DISK_MOUNTED) UDisk_USBH_DiskReady();

    char filename[64];
    if (find_first_flac(filename) != 0) while (1);

    strcpy((char *)mCmdParam.Open.mPathName, filename);
    if (CHRV3FileOpen() != ERR_SUCCESS) while (1);

    play_loop();
}
```

## DMA refill ISR (ping-pong)

```c
#define I2S_BUF_FRAMES 1024
static uint16_t i2s_buf[I2S_BUF_FRAMES * 2 * 2];   /* stereo, double-buffered */
static volatile int half_to_fill = -1;

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
```

## Gotchas to confirm on hardware

- Sample rate switch (44.1 → 96 → 192) requires I2S disable → reconfigure → enable; expect ~10 ms glitch
- 24-bit hi-res: `drflac_read_pcm_frames_s32` + `I2S_DataFormat_24b`, pack into 32-bit slots
- Half-buffer = 4 KB → ~23 ms @ 44.1 / ~5 ms @ 192. Grow if FLAC decode jitter underruns
- PCM5102A has no register interface → software volume in the s16/s32 stream

## Project layout

```
flac-usb-i2s/
├── Makefile         (per BUILD.md)
├── User/
│   ├── main.c
│   ├── audio_pipe.c   I2S init + DMA refill
│   ├── usb_host.c     CHRV3 init + file iter
│   ├── flac_io.c      dr_flac ↔ CHRV3 bridge
│   └── system_ch32h417.c
└── lib/
    ├── dr_flac.h
    └── libRV3UFI.a
```

## First milestones

1. Toolchain validated: flash GPIO_Toggle, blink external LED on PB0
2. I2S_DMA example unmodified + PCM5102A → fixed sine
3. Host_UDisk_Exams unmodified → read a known text file
4. Combine: play 44.1/16 PCM. Then FLAC decode. Then 24/192.
