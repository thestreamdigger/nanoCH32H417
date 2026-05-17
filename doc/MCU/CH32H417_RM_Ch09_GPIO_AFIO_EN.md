# Chapter 9 — GPIO and Alternate Function I/O (GPIO/AFIO)

Concise translation of `CH32H417_ReferenceManual_CN.PDF` V1.7, lines 7208–8453. Original: `_extracted/rm_full.txt`.

The chip has **6 GPIO ports** (PA, PB, PC, PD, PE, PF) totaling **95 pins**. Most pins are software-configurable as digital input/output or alternate-function peripheral I/O.

---

## Power domains

Pin behavior and supply voltage depend on the power domain:

| Domain | Voltage | Pins | Notes |
|--------|---------|------|-------|
| **VDD33** | 3.3 V fixed | PA9–PA12, PC13–PC15, PE3–PE6 | PC13–PC15 auto-switch to VBAT on VDD33 loss |
| **VDDIO** | 3.3 V (supports 1.8 / 2.5 / 3.3 V) | PA0–PA8, PB2–PB7, PB15, PC4–PC5, PD8, PE2, PF6–PF10 | external supply choice |
| **VIO18** | 1.2 / 1.8 / 2.5 / 3.3 V (internal regulator, **dynamic switch**) | PA13–PA15, PB0–PB1, PB10–PB14, PC0–PC3, PC6–PC12, PD0–PD7, PD9–PD15, PE0–PE1, PE7–PE15, PF0–PF5, PF11–PF14 | **high-speed pins** — used for I2S, USB, SPI |

Internal voltage regulator on VIO18 pins is configured via `PWR_CTLR` register bits [12:9] (see PWR chapter).

> Most pins exposed on the **nanoCH32H417 expansion headers** are VIO18 (high-speed). PA13/PA14 (SWDIO/SWCLK) are VDD33.

---

## Pin modes

Each pin: one of these modes:

| Mode | Use |
|------|-----|
| **Floating input** | high-impedance, no pull |
| **Pull-up input** | weak pull to VDD |
| **Pull-down input** | weak pull to GND |
| **Analog input** | ADC, comparator, OPA |
| **Push-pull output** | standard digital out |
| **Open-drain output** | needs external pull-up |
| **AF push-pull** | alternate function, push-pull |
| **AF open-drain** | alternate function, open-drain |

Configuration registers (per pin):
- `GPIOx_CFGLR` / `GPIOx_CFGHR` — mode + speed
- `GPIOx_INDR` — input data (read-only)
- `GPIOx_OUTDR` — output data
- `GPIOx_BSHR` / `GPIOx_BCR` — atomic set/reset
- `GPIOx_LCKR` — lock configuration until next reset
- `GPIOx_AFLR` / `GPIOx_AFHR` — alternate function select (AF0..AF15)

---

## Alternate Functions

Every pin has a 16-input AF mux (AF0–AF15). One AF selected at a time per pin → no peripheral conflict.

After reset: AF0 selected on all pins.

To use a peripheral on a pin:
1. Configure pin mode = AF (push-pull or open-drain per peripheral need)
2. Set the AF code in `GPIOx_AFLR` (pins 0–7) or `GPIOx_AFHR` (pins 8–15)
3. Enable the peripheral clock (RCC)

**The exact AF mapping per pin is in the `CH32H417_Datasheet_EN.PDF`** — search the alternate function table. Examples relevant to this project:

| Pin | AF | Function | Used by |
|-----|-----|----------|---------|
| PB12 | AF5 | I2S2_WS | I2S audio out |
| PB13 | AF5 | I2S2_CK | I2S audio out |
| PC1  | AF5 | I2S2_SD | I2S audio out |
| PB15 | AF5 | I2S2_MCK | I2S master clock (optional) |
| PA13 | AF1 | I2S3_SD | I2S3 audio (alt path, conflicts SWDIO) |
| PA14 | AF1 | I2S3_CK | (conflicts SWCLK) |
| PA15 | AF6 | I2S3_WS | I2S3 audio |
| PB12 | AF5 | SPI2_NSS | SPI2 (same pin as I2S2_WS — mutually exclusive) |
| PB13 | AF5 | SPI2_SCK | SPI2 |
| PB15 | AF5 | SPI2_MOSI | SPI2 |
| PB8/PB9 | dedicated | USB2.0 D+/D- (USBHS_DP/DM) | USB Host high-speed |

> The same pin name (e.g. PB12) supports multiple peripherals via different AF codes. Pick AF code = peripheral you want.

---

## External Interrupts (EXTI)

All GPIO can be EXTI sources, but with a constraint:

**Each EXTI line `EXTIx` can map to pin `x` of exactly ONE port at a time.**

Example: `EXTI1` can be PA1 OR PB1 OR PC1 OR PD1 OR PE1 — pick one.

Mapping controlled by `AFIO_EXTICRx` registers.

---

## Lock mechanism (`GPIOx_LCKR`)

Optional: after a specific magic write sequence, the selected pin configuration freezes until next reset. Use for safety-critical signals you don't want code mistakes to perturb.

---

## Quick recipe — configure I2S2 output pins

```c
/* Enable clocks */
RCC_HB2PeriphClockCmd(RCC_HB2Periph_GPIOB | RCC_HB2Periph_GPIOC | RCC_HB2Periph_AFIO, ENABLE);
RCC_HB1PeriphClockCmd(RCC_HB1Periph_SPI2, ENABLE);

/* PB12 WS, PB13 CK — AF5 */
GPIO_PinAFConfig(GPIOB, GPIO_PinSource12, GPIO_AF5);
GPIO_PinAFConfig(GPIOB, GPIO_PinSource13, GPIO_AF5);

GPIO_InitTypeDef g = {
    .GPIO_Mode  = GPIO_Mode_AF_PP,
    .GPIO_Speed = GPIO_Speed_Very_High,    /* max edge rate for 192 kHz */
    .GPIO_Pin   = GPIO_Pin_12 | GPIO_Pin_13,
};
GPIO_Init(GPIOB, &g);

/* PC1 SD — AF5 */
GPIO_PinAFConfig(GPIOC, GPIO_PinSource1, GPIO_AF5);
g.GPIO_Pin = GPIO_Pin_1;
GPIO_Init(GPIOC, &g);
```

Full per-pin AF mapping table is in `CH32H417_Datasheet_EN.PDF`, not in the RM.
