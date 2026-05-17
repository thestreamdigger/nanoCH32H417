# Chapter 10 — Direct Memory Access (DMA)

Concise translation of `CH32H417_ReferenceManual_CN.PDF` V1.7, lines 8454–9259. Original: `_extracted/rm_full.txt`.

DMA enables peripheral↔memory and memory↔memory transfers without CPU. Critical for the audio project (I2S TX) and USB transfers.

---

## Key features

- **2 DMA controllers**: DMA1 + DMA2, **8 channels each** (16 total)
- **Software-mappable request lines** — any peripheral request can be routed to any channel via `CHANNELx_MUX` bits
- **Software trigger** for memory-to-memory transfers
- **Circular buffer** mode (essential for continuous audio)
- **4 priority levels** (highest / high / medium / low) — ties broken by channel number (lower = higher)
- Modes: peripheral→memory, memory→peripheral, memory↔memory
- Sources/destinations: Flash, SRAM, peripheral SRAM, HB peripherals
- **Max transfer count per channel: 65,535**
- Data widths: **8, 16, 32, 256 bit**, with automatic alignment when source/dest widths differ

---

## Transfer modes

| Mode | `MEM2MEM` | `DIR` | Use case |
|------|-----------|-------|----------|
| Peripheral → Memory | 0 | 0 | ADC read, UART RX, **USB RX** |
| Memory → Peripheral | 0 | 1 | DAC write, UART TX, **I2S TX**, USB TX |
| Memory → Memory | 1 | — | mem copy (software-triggered, no circular mode) |

---

## Configuration steps

1. Set `DMA_PADDRx` = peripheral register address (for mem2mem: source data address)
2. Set `DMA_MADDRx` = memory buffer address (select via `FLAG_CUR_MEM` bit)
3. Set `DMA_CNTRx` = number of transfers (max 65535) — must be set with `EN=0`
4. Configure `DMA_CFGRx`:
   - `MEM2MEM`, `DIR` per table above
   - `PINC`, `MINC` → enable peripheral/memory address auto-increment
   - `PSIZE[1:0]`, `MSIZE[1:0]` → data widths (8/16/32/256)
   - `PL[1:0]` → priority
   - `CIRC` → circular mode
   - `HTIE`, `TCIE`, `TEIE` → enable half-transfer / transfer-complete / transfer-error IRQs
5. Set `EN = 1` to start

**Important**: `CFGR` bits (CIRC, MINC/PINC, etc.) can only be written when the channel is disabled (`EN=0`).

---

## Status flags (DMA_INTFR register)

| Flag | Set when | Use case |
|------|----------|----------|
| `GIFx` | global interrupt for channel x | check first to identify which channel fired |
| `HTIFx` | half-transfer (count ≤ initial/2) | **ping-pong refill** — fill first half while second plays |
| `TCIFx` | transfer complete (count = 0) | refill second half, or chain |
| `TEIFx` | transfer error (reserved-region access) | error handling — channel auto-disables |

Clear via writes to `DMA_INTFCR` register.

---

## Circular mode for continuous audio

When `CIRC = 1`:
- On count reaching 0, `DMA_CNTRx` auto-reloads to its initial value
- Internal peripheral + memory addresses also reload from `DMA_PADDRx` / `DMA_MADDRx`
- Transfer continues until `EN` is cleared

This is the **standard pattern for I2S audio streaming**:

```c
/* Allocate a stereo buffer big enough for N samples */
int32_t i2s_buffer[N * 2];   /* N stereo frames, 32-bit slot for 24-bit data */

/* DMA: mem → SPI2->DATAR, circular, 16-bit transfers */
DMA_InitTypeDef d = {
    .DMA_PeripheralBaseAddr = (uint32_t)&SPI2->DATAR,
    .DMA_MemoryBaseAddr     = (uint32_t)i2s_buffer,
    .DMA_DIR                = DMA_DIR_PeripheralDST,  /* mem → peripheral */
    .DMA_BufferSize         = N * 2 * 2,              /* 16-bit half-words */
    .DMA_PeripheralInc      = DMA_PeripheralInc_Disable,
    .DMA_MemoryInc          = DMA_MemoryInc_Enable,
    .DMA_PeripheralDataSize = DMA_PeripheralDataSize_HalfWord,
    .DMA_MemoryDataSize     = DMA_MemoryDataSize_HalfWord,
    .DMA_Mode               = DMA_Mode_Circular,
    .DMA_Priority           = DMA_Priority_High,
    .DMA_M2M                = DMA_M2M_Disable,
};
DMA_Init(DMA1_Channel5, &d);     /* SPI2 TX request — see channel map */

/* Enable HT + TC interrupts for ping-pong refill */
DMA_ITConfig(DMA1_Channel5, DMA_IT_HT | DMA_IT_TC, ENABLE);
NVIC_EnableIRQ(DMA1_Channel5_IRQn);

/* Start */
DMA_Cmd(DMA1_Channel5, ENABLE);
SPI_I2S_DMACmd(SPI2, SPI_I2S_DMAReq_Tx, ENABLE);
I2S_Cmd(SPI2, ENABLE);
```

In the ISR:
```c
void DMA1_Channel5_IRQHandler(void) {
    if (DMA_GetITStatus(DMA1_IT_HT5)) {       /* first half done */
        DMA_ClearITPendingBit(DMA1_IT_HT5);
        refill_first_half(i2s_buffer);        /* decoder produces samples */
    }
    if (DMA_GetITStatus(DMA1_IT_TC5)) {       /* second half done */
        DMA_ClearITPendingBit(DMA1_IT_TC5);
        refill_second_half(i2s_buffer + N);
    }
}
```

---

## Data-width auto-alignment

When source width ≠ destination width, DMA handles padding/truncation:
- **Source smaller** (e.g. 8→16): high bits zero-padded
- **Source larger** (e.g. 32→8): high bits truncated
- **Little-endian convention** throughout

For I2S TX with 24-bit data:
- Pack 24-bit samples into 32-bit slots
- `MSIZE = 32-bit`, `PSIZE = 16-bit` (SPI/I2S DATAR is 16-bit)
- DMA sends LSB half-word first (data), then MSB half-word (extension) → matches I2S 24-bit-in-32-bit-channel format

---

## Channel request map (DMA1)

The hardware request line for each peripheral is selected via `CHANNELx_MUX` bits in the DMA controller. Exact mapping table is in the original PDF (~page 150). For the audio project, the common assignments are:

| Peripheral | Direction | Typical DMA1 channel |
|------------|-----------|----------------------|
| SPI2/I2S2 RX | per→mem | Channel 4 |
| SPI2/I2S2 TX | mem→per | **Channel 5** |
| SPI3/I2S3 RX | per→mem | Channel 1 |
| SPI3/I2S3 TX | mem→per | Channel 2 |

> Verify exact channel via `CHANNELx_MUX` in `EXAM/I2S/I2S_DMA/Common/hardware.c`.
