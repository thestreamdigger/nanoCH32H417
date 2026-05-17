# EXAMPLES — index of `doc/EVT/EXAM/`

All paths relative to `doc/EVT/EXAM/`. Each project ships V3F and V5F variants under `<example>/<name>/{V3F,V5F}/`. Pick **V5F** by default (main core, FPU).

Use this table as a lookup when asking Claude to implement a peripheral feature — it knows which reference to read first.

## Core

| Path | What |
|------|------|
| CPU/CoreMark | Benchmark — measures CoreMark/MHz |
| CPU/FPU | V5F FPU enable + basic float math |
| CPU/HSEM | Hardware semaphore between V5F and V3F |
| CPU/ICache | Instruction cache enable + measurement |
| CPU/IPC | Inter-processor comm V5F ↔ V3F |
| CPU/PFIC | Programmable Fast Interrupt Controller |
| CPU/PMP | Physical Memory Protection |
| CPU/SYSTICK | SysTick periodic interrupt |
| RCC/Get_CLK | Read system clock tree |
| RCC/HSI_Calibration | Trim internal RC oscillator |
| RCC/MCO | Master Clock Output on pin |

## GPIO / EXTI / IO peripherals

| Path | What |
|------|------|
| GPIO/GPIO_Toggle | Push-pull output blink (PB1) |
| EXTI/EXTI0 | External interrupt on pin |
| PIOC/PIOC_1_Wire | Programmable IO Controller — 1-Wire bit-bang |
| PIOC/PIOC_IIC | PIOC bit-bang I2C |
| PIOC/PIOC_NEC | PIOC NEC IR decode |
| PIOC/PIOC_UART | PIOC bit-bang UART |
| PIOC/PIOC_Single_Wire | half-duplex single-wire |

## Timers

| Path | What |
|------|------|
| TIM/TIM_Continuous | basic free-running timer |
| TIM/TIM_INT | timer with interrupt |
| TIM/TIM_DMA | timer-triggered DMA |
| TIM/PWM_Output | PWM generation |
| TIM/Input_Capture | edge-time measurement |
| TIM/Bothedge_Capture | rise + fall capture |
| TIM/Encoder | quadrature decoder |
| TIM/One_Pulse | one-shot |
| TIM/Synchro_Timer | master/slave timer sync |
| TIM/ComplementaryOutput_DeadTime | motor control PWM |
| TIM/Clock_Select | clock source switching |
| LPTIM/LPTIM_LP_WakeUp | low-power wake |
| LPTIM/PWM_OnePulse_SingleMode | LPTIM single pulse |
| IWDG/IWDG | independent watchdog |
| WWDG/WWDG | windowed watchdog |
| RTC/RTC_Calendar | calendar mode |
| RTC/RTC_Calibrations | calib drift |

## Analog

| Path | What |
|------|------|
| ADC/ADC_DMA | continuous ADC into DMA buffer |
| ADC/AnalogWatchdog | threshold interrupt |
| ADC/Auto_Injection | injected group |
| ADC/Discontinuous_mode | grouped conversions |
| ADC/DualADC_* (6 modes) | two ADCs simultaneous |
| ADC/TIM_Trigger | timer-triggered sample |
| DAC/DAC_Normal_OUT | basic DAC |
| DAC/DAC_DMA | DMA-fed DAC stream |
| DAC/DAC_Noise_Generation | noise output |
| DAC/DualDAC_SineWave | two-channel sine |
| DAC/DualDAC_Triangle | triangle wave |
| HSADC/HSADC | high-speed ADC (~10-bit, fast) |
| OPA/OPA | on-chip op-amp |
| OPA/CMP | analog comparator |

## DSP / audio

| Path | What |
|------|------|
| DFSDM/DFSDM_DualFliter | sigma-delta dual-filter |
| DFSDM/DFSDM_I2S_Audio | sigma-delta over I2S source |
| DFSDM/DFSDM_InternalADC | with internal ADC |
| DFSDM/DFSDM_TIMTrigger | timer-triggered |
| SAI/SAI_Play | serial audio I/F (TDM/I2S) |
| I2S/HostRx_SlaveTx | I2S host/slave pair |
| I2S/I2S_DMA | I2S into DMA |

## Communication

| Path | What |
|------|------|
| USART/USART_Polling | bare polled UART |
| USART/USART_Interrupt | IRQ-driven |
| USART/USART_DMA | DMA |
| USART/USART_HalfDuplex | RS485-style half-duplex |
| USART/USART_HardwareFlowControl | CTS/RTS |
| USART/USART_MultiProcessorCommunication | wake-on-addr |
| APPLICATION/SoftUART | bit-bang UART |
| SPI/2Lines_FullDuplex | full-duplex |
| SPI/1Lines_half-duplex | 1-wire SPI |
| SPI/SPI_DMA | with DMA |
| SPI/SPI_FLASH | external flash chip driver |
| SPI/SPI_LCD | LCD via SPI (matches FPC connector) |
| SPI/FullDuplex_HardNSS | hardware chip-select |
| I2C/I2C_7bit_Mode | basic master |
| I2C/I2C_10bit_Mode | 10-bit addressing |
| I2C/I2C_DMA | DMA |
| I2C/I2C_EEPROM | external EEPROM driver |
| I2C/I2C_PEC | packet error check |
| I3C/I3C_Controller_* | I3C master |
| I3C/I3C_DMA_Target | I3C peripheral with DMA |
| CAN/Networking | multi-node |
| CAN/TestMode | loopback |
| CAN/Time-triggered | TTCAN |
| SWPMI/SWPMI_SwpCard | single-wire protocol (smartcards) |
| SWPMI/SWPMI_MultiBUffering | multi-buffer |

## Storage

| Path | What |
|------|------|
| SDIO/SDIO_SD | SD card via SDIO (matches onboard slot) |
| SDIO/SDIO_eMMC | eMMC via SDIO |
| SDMMC/SDMMC_SD | SD card via SDMMC |
| SDMMC/SDMMC_eMMC | eMMC via SDMMC |
| SDMMC/SDMMC_eMMC_DDR | eMMC DDR mode |
| QSPI/QSPI_FLASH | quad-SPI flash |
| QSPI/QSPI_FLASH_DMA | with DMA |
| FLASH/FLASH_Program | self-program internal flash |
| FLASH/LoadInFlash | code-in-flash example |
| FMC/SRAM | external SRAM |
| FMC/NAND | NAND flash |
| FMC/NORFLASH | NOR flash |
| FMC/SDRAM_16bit / 32bit (+DMA) | external SDRAM |
| FMC/LCD / LCD_8bit | parallel LCD via FMC |
| IAP/USB_UART | bootloader (in-app prog) over USB or UART |

## Networking — Ethernet

| Path | What |
|------|------|
| ETH/DHCP | DHCP client |
| ETH/DNS | DNS resolution |
| ETH/IPRaw_PING | raw IP ping |
| ETH/MAC_RAW | raw MAC frames |
| ETH/TCPClient | TCP client |
| ETH/TCPServer | TCP server |
| ETH/UDPClient | UDP client |
| ETH/UDPServer | UDP server |
| ETH/MQTT | MQTT pubsub |
| ETH/WebServer | HTTP server |
| ETH/NetLib | network library |
| ETH/IoCHub | IoT cloud hub |

## USB

| Path | What |
|------|------|
| USBFS/DEVICE | USB 2.0 Full Speed device |
| USBFS/HOST | USB 2.0 FS host |
| USBHS/DEVICE | USB 2.0 High Speed device |
| USBHS/HOST | USB 2.0 HS host |
| USBSS/DEVICE | **USB 3.0 SuperSpeed device** (board's main feature) |
| USBPD/USBPD_SNK | USB-PD sink |
| USBPD/USBPD_SRC | USB-PD source |

## Graphics / display

| Path | What |
|------|------|
| LTDC/LTDC_Display | LCD-TFT controller (RGB parallel) |
| GPHA/GPHA_M2M_WithLCD | 2D graphics accel mem→mem with LCD |
| GPHA/GPHA_R2M | register→mem (fill) |
| DVP/DVP_TFTLCD | camera → LCD passthrough |
| DVP/DVP_UART | camera → UART stream |
| SPI/SPI_LCD | SPI LCD (FPC) |

## Specialty

| Path | What |
|------|------|
| SerDes/AcquireConfig | serializer/deserializer config |
| SerDes/FullDuxTrans | full-duplex transfer |
| UHSIF/UHSIF_SLAVE | Universal High-Speed Interface 500 MB/s slave |
| TKey/TKey | capacitive touch single key |
| TKey/TKey_8keys | 8-key touch matrix |
| TKey/TKEYLIB | touchkey library |
| RNG/RNG | true RNG |
| CRC/CRC_Calculation | hardware CRC |
| ECDC/Single_Time | error correction once |
| ECDC/RAM_Block | error correction block |
| DMA/DMA_MEM2MEM | mem-to-mem |
| DMA/DoubleBuffer_DMA | ping-pong buffer |
| DMA/TIM_DMA_SLEEP_MODE | low-power DMA |
| PWR/Sleep_Mode | CPU sleep |
| PWR/Stop_Mode | deep stop |
| PWR/PVD_VoltageJudger | brown-out detect |
| PWR/SEL_VIO18 | 1.8 V IO supply |
| PWR/VDD12_ExternPower | external 1.2 V core |

## SDK source — NOT examples

`SRC/` is the StdPeriph library itself, not a demo. Link your project against it:

```
SRC/Core/        core_riscv.{c,h}
SRC/Startup/     startup_ch32h417_v5f.S (or _v3f.S)
SRC/Ld/V5F/      Link_v5f.ld
SRC/Ld/V3F/      Link_v3f.ld
SRC/Debug/       debug.{c,h}             printf via UART1
SRC/Peripheral/  inc/ + src/             42 headers + 38 sources
```

See [BUILD.md](BUILD.md) for the Make recipe that pulls these in.
