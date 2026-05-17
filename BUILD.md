# BUILD — CLI workflow (Trilha B)

Headless build + flash without MounRiver Studio. Optimized for vibecoding (LLM can run every step).

## Toolchain

| Tool | Purpose | Install |
|------|---------|---------|
| **riscv-none-elf-gcc** 14.x | RV32 cross-compile | xPack |
| **wlink** | flash + GDB server via WCHLink-E | `cargo install wlink` |
| **wchisp** | flash via USB DFU (MODE+RESET) | `cargo install wchisp` |
| **gdb-multiarch** | source debug | apt |
| **make** / **ninja** | build driver | apt |

### Install riscv-none-elf-gcc (xPack)

```bash
wsl -d Debian -- bash -lc "
  npm i -g xpm@latest
  xpm install --global @xpack-dev-tools/riscv-none-elf-gcc@latest
  echo 'export PATH=\$HOME/.local/xPacks/@xpack-dev-tools/riscv-none-elf-gcc/14.2.0-3.1/.content/bin:\$PATH' >> ~/.bashrc
"
```

Verify:
```bash
wsl -d Debian -- bash -ic "riscv-none-elf-gcc --version"
```

### Install wlink (flash via onboard WCHLink-E)

```bash
wsl -d Debian -- bash -lc "cargo install wlink"
```

USB permission (Debian):
```bash
wsl -d Debian -u root -- bash -c "
  echo 'SUBSYSTEM==\"usb\", ATTRS{idVendor}==\"1a86\", ATTRS{idProduct}==\"8010\", MODE=\"0666\"' \
    > /etc/udev/rules.d/99-wchlink.rules
  udevadm control --reload-rules
"
```

USB passthrough to WSL (Windows side, admin):
```powershell
usbipd list                       # find WCHLink-E (VID 1a86 PID 8010)
usbipd bind --busid <BUSID>
usbipd attach --wsl --busid <BUSID>
```

## Compiler flags

From `EXAM/*/V5F/*.wvproj` and `EXAM/*/V3F/*.wvproj`:

| Target | -march | -mabi |
|--------|--------|-------|
| V5F (no FPU) | `rv32imacxw_zicsr` | `ilp32` |
| V5F (with FPU) | `rv32imafcxw_zicsr` | `ilp32f` |
| V3F | `rv32imacxw_zicsr` | `ilp32` |

`xw` = WCH custom compressed extension (provided by riscv-none-elf-gcc 12+).

## Minimal Makefile

Drop into a new project root next to a `User/` folder with `main.c`. Points to SDK in this fork.

```makefile
PROJECT  ?= app
TARGET   ?= V5F                     # V5F or V3F
SDK      := doc/EVT/EXAM/SRC

CROSS    := riscv-none-elf-
CC       := $(CROSS)gcc
AS       := $(CROSS)gcc -x assembler-with-cpp
OBJCOPY  := $(CROSS)objcopy
SIZE     := $(CROSS)size

ifeq ($(TARGET),V5F)
  MARCH := rv32imacxw_zicsr
  MABI  := ilp32
  STARTUP := $(SDK)/Startup/startup_ch32h417_v5f.S
  LDSCRIPT := $(SDK)/Ld/V5F/Link_v5f.ld
else
  MARCH := rv32imacxw_zicsr
  MABI  := ilp32
  STARTUP := $(SDK)/Startup/startup_ch32h417_v3f.S
  LDSCRIPT := $(SDK)/Ld/V3F/Link_v3f.ld
endif

CFLAGS  := -march=$(MARCH) -mabi=$(MABI) -msmall-data-limit=8 \
           -msave-restore -Os -fmessage-length=0 -fsigned-char \
           -ffunction-sections -fdata-sections -Wall -g
CFLAGS  += -I User -I $(SDK)/Core -I $(SDK)/Debug -I $(SDK)/Peripheral/inc
LDFLAGS := -T $(LDSCRIPT) -nostartfiles -Xlinker --gc-sections \
           -Wl,-Map,build/$(PROJECT).map --specs=nano.specs --specs=nosys.specs

SRC_C := $(wildcard User/*.c) \
         $(wildcard $(SDK)/Core/*.c) \
         $(wildcard $(SDK)/Debug/*.c) \
         $(wildcard $(SDK)/Peripheral/src/*.c)
SRC_S := $(STARTUP)

OBJ_C := $(patsubst %.c,build/%.o,$(SRC_C))
OBJ_S := $(patsubst %.S,build/%.o,$(SRC_S))
OBJS  := $(OBJ_C) $(OBJ_S)

ELF := build/$(PROJECT).elf
BIN := build/$(PROJECT).bin
HEX := build/$(PROJECT).hex

all: $(ELF) $(BIN) $(HEX)
	@$(SIZE) $(ELF)

build/%.o: %.c
	@mkdir -p $(dir $@)
	$(CC) $(CFLAGS) -c -o $@ $<

build/%.o: %.S
	@mkdir -p $(dir $@)
	$(AS) $(CFLAGS) -c -o $@ $<

$(ELF): $(OBJS)
	$(CC) $(CFLAGS) $(LDFLAGS) -o $@ $(OBJS)

$(BIN): $(ELF)
	$(OBJCOPY) -O binary $< $@

$(HEX): $(ELF)
	$(OBJCOPY) -O ihex $< $@

flash: $(BIN)
	wlink flash $(BIN)

erase:
	wlink erase

reset:
	wlink reset

gdb-server:
	wlink gdb

clean:
	rm -rf build

.PHONY: all flash erase reset gdb-server clean
```

Build + flash:
```bash
wsl -d Debian -- bash -ic "cd /mnt/c/dev/ideas/embedded/<your-project> && make && make flash"
```

## Flash workflows

### Via WCHLink-E (preferred, onboard)
```bash
wlink probe                   # verify board detected
wlink flash build/app.bin
wlink reset
```

### Via USB DFU (no WCHLink needed)
Hold **MODE** button, press **RESET**, release MODE → board enumerates as DFU.
```bash
wchisp probe
wchisp flash build/app.bin
```

## Serial monitor

WCHLink-E exposes UART on the same USB. From WSL:
```bash
tio /dev/ttyACM0 -b 115200
```

(Default baud per `USART_Printf_Init(115200)` in WCH examples.)

## GDB

Two terminals:

Terminal A — GDB server:
```bash
wlink gdb
```

Terminal B — client:
```bash
riscv-none-elf-gdb build/app.elf -ex "target remote :3333"
```

## Quick start — copy an example

```bash
wsl -d Debian -- bash -ic "
  cp -r doc/EVT/EXAM/GPIO/GPIO_Toggle/V5F/User my-blink/User
  cp doc/EVT/EXAM/GPIO/GPIO_Toggle/Common/hardware.* my-blink/User/
  cp BUILD.md/Makefile-template my-blink/Makefile   # write Makefile per above
  cd my-blink && make && make flash
"
```

## Known caveats

- **WCH `xw` extension**: only newer riscv-none-elf-gcc (xPack 12+) knows it. If unknown, drop `xw` from `-march` (slight code-size hit, otherwise OK).
- **No `--specs=nano.specs`**: omit if newlib-nano is missing — falls back to full newlib (larger).
- **DFU window short**: after `MODE+RESET`, you have ~5 s to start `wchisp`.
- **`wlink` and `wchisp` are mutually exclusive** while one runs.
- **USB passthrough**: re-attach `usbipd` after every WSL restart.
