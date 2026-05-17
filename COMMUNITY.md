# COMMUNITY — where to ask, what to read

Resources for the CH32H417 specifically, and the wider WCH RISC-V (CH32) ecosystem, since the H417 community is still small (chip released Dec 2025).

When in doubt: post on the most active channel that fits the question. CH32H417-specific traffic is thin — for general WCH RISC-V questions, the broader CH32V community usually answers.

---

## Tier 1 — official WCH

| Resource | URL | Notes |
|----------|-----|-------|
| WCH product page (EN) | <https://www.wch-ic.com/products/CH32H417.html> | datasheet, RM, app notes (SPA — needs JS browser) |
| WCH product page (CN) | <https://www.wch.cn/products/CH32H417.html> | same content in Chinese, sometimes updated first |
| WCH downloads (EN) | <https://www.wch-ic.com/downloads/category/27.html> | category 27 = product manuals |
| WCH official repo (H417) | <https://github.com/openwch/ch32h417> | mirror of EVT + links to datasheet downloads |
| openwch organization | <https://github.com/openwch> | all WCH-published reference repos (CH32V003, V103, V203, V305, V307, X035, L103, H417, etc.) |
| MounRiver Studio (official IDE) | <http://www.mounriver.com/> | toolchain + SDK, Windows/Linux/Mac |

---

## Tier 2 — active community (English)

### Curated index — start here

| Resource | URL | Why |
|----------|-----|-----|
| **awesome-ch32** | <https://github.com/ch32-rs/awesome-ch32> | curated list of everything CH32 — first stop for any new question |
| **CH32 RISC-V User Group** (GitHub org) | <https://github.com/ch32-riscv-ug> | community mirror with errata, GPIO defs, datasheets including for H417 |

### Frameworks / HALs

| Resource | URL | Status for H417 |
|----------|-----|------|
| **ch32fun** (cnlohr) | <https://github.com/cnlohr/ch32fun> | minimal open stack — most active CH32 community project. **Does not yet support H417** (V0/V1/V2/V3/X0/L1 only). Most likely route to add H417 → PR welcome. |
| **ch32-rs/ch32-hal** | <https://github.com/ch32-rs/ch32-hal> | Rust HAL with Embassy. **No H417 support yet.** |
| **ch32-rs/ch32-metapac** | <https://github.com/ch32-rs/ch32-metapac> | Rust PAC generator. Eventually generates H417 once `ch32-data` adds it. |
| **ch32-rs/ch32-data** | <https://github.com/ch32-rs/ch32-data> | structured chip DB (registers, peripherals). Contributing H417 here unlocks Rust + future tools. |
| **ch32-rs/wlink** | <https://github.com/ch32-rs/wlink> | open Rust WCH-Link tool. **H417 supported** for flash. |
| **Community-PIO-CH32V** (org) | <https://github.com/Community-PIO-CH32V> | PlatformIO platform definitions. CH32V support good; H417 not yet. |
| **arduino_core_ch32_riscv** | <https://github.com/ch32-riscv-ug/arduino_core_ch32_riscv_noneos> | Arduino IDE core, brings EVT examples to Arduino. May or may not work on H417. |

### Chat / forum

| Channel | URL | Status |
|---------|-----|--------|
| **`#ch32fun` Discord** (cnlohr) | invite via [Twitter @cnlohr DM](https://x.com/cnlohr) or GitHub issue on [ch32fun](https://github.com/cnlohr/ch32fun) | **most active real-time CH32 chat.** Private invite, must ping cnlohr. |
| **ch32v.com forum** | <https://www.ch32v.com/> | listed as central hub; intermittent reachability — verify before relying on |
| **r/RISCV** | <https://www.reddit.com/r/RISCV/> | broad RISC-V, occasional WCH threads |
| **r/embedded** | <https://www.reddit.com/r/embedded/> | general — CH32 occasionally |
| **GitHub Issues** on cnlohr/ch32fun, ch32-rs/* | each repo's issues tab | best for technical questions on those tools |

### Podcasts / interviews

| Source | URL |
|--------|-----|
| The Amp Hour #637 — CH32V003…fun! with CNLohr | <https://theamphour.com/637-ch32v003-fun-with-cnlohr/> |
| The Amp Hour #667 — Long Distance with CNLohr | <https://theamphour.com/667-long-distance-with-cnlohr-a/> |

---

## Tier 3 — news / coverage

Useful for tracking the H417's evolution, new tooling announcements, related WCH releases.

| Outlet | URL |
|--------|-----|
| **CNX Software — WCH tag** | <https://www.cnx-software.com/news/wch/> |
| CNX — CH32H417 launch article | <https://www.cnx-software.com/2026/01/03/wch-ch32h417-dual-core-risc-v-mcu-offers-usb-3-0-500mb-s-uhsif-and-fast-ethernet-interfaces/> |
| LinuxGizmos — CH32H417 | <https://linuxgizmos.com/ch32h417-dual-core-risc-v-mcu-offers-usb-ethernet-and-serdes-support/> |
| Electronics-Lab — CH32H417 | <https://www.electronics-lab.com/ch32h417-dual-core-risc-v-mcu-supports-usb-3-2-serdes-usb-pd-and-10-100m-ethernet/> |
| Adafruit blog — WCH/CH32 tag | <https://blog.adafruit.com/?s=ch32> |
| Hackaday.io — CH32H417 EVB | <https://hackaday.io/page/399709> |
| Grokipedia — CH32H417 | <https://grokipedia.com/page/ch32h417> |
| RISC-V International on WCH | <https://riscv.org/blog/wch-risc-v-microcontrollers-can-now-be-programmed-with-the-arduino-ide/> |

---

## Tier 4 — vendors (boards + parts)

| Vendor | URL | What |
|--------|-----|------|
| MuseLab (manufacturer) | <https://www.muselab-tech.com/> | nanoCH32H417 maker; also makes nano-V003/V203/V317, ESP32-C3/C6 boards |
| AliExpress (MuseLab) | <https://www.aliexpress.com/item/1005005033298927.html> | nanoCH32H417 |
| Tindie (MuseLab — johnnywu) | <https://www.tindie.com/products/johnnywu/nanoch32h417-development-board/> | nanoCH32H417 |
| AnalogLamb (alt board) | <https://analoglamb.com/product/wch-ch32h417-usb3-0-development-board/> | WCH official eval board, alternative form factor |

---

## Tier 5 — peer projects (porting sources)

When H417-specific docs don't cover something, the closest cousins are often good ports of reference.

| Chip | Repo | Why useful |
|------|------|------------|
| **CH32V307** | <https://github.com/openwch/ch32v307> | closest predecessor — single-core V4F + ETH + USB HS. Lots of code reusable. |
| **CH32V305** | <https://github.com/openwch/ch32v305> | smaller V4F + USB HS. Used as **WCHLink-E debugger on the nanoCH32H417 itself**. |
| **CH32V003** | <https://github.com/openwch/ch32v003> | tiny but most-loved by community — best tutorials, ch32fun targets it first |
| **CH32X035** | <https://github.com/openwch/ch32x035> | USB-C PD reference |

`ch32fun` and `ch32-hal` both follow this pattern: they support the closer cousins first, H417 will land later.

---

## Tier 6 — chip-specific docs hosted by others (mirrors)

| Resource | URL | Why |
|----------|-----|-----|
| **ch32-riscv-ug/CH32H417** (datasheets EN+CN, RM CN) | <https://github.com/ch32-riscv-ug/CH32H417> | already cloned into [`doc/MCU/`](doc/MCU/) of this fork |
| QingKeV2 manual mirror | <https://ch405-labs.com/content/files/2023/11/QingKeV2_Processor_Manual.PDF> | older core — useful baseline for understanding QingKe ISA |
| QingKeV4 manual link | <https://www.wch-ic.com/downloads/QingKeV4_Processor_Manual_PDF.html> | one generation older than V5F — read this if V5F isn't published yet |
| Olimex WCH-Link User Manual mirror | <https://www.olimex.com/Products/RISC-V/WCH/WCH-LinkE/resources/WCH-LinkUserManual.PDF> | direct PDF (no JS needed) |

---

## Multi-AI workflow (Gemini as second opinion)

This environment has the **`gemini-cli` MCP server** registered globally — Gemini Pro/Flash is available as a tool from inside Claude Code. Use it for:

- **Long docs**: hand the entire Chinese RM PDF to Gemini Pro (1M+ context) for a holistic explanation that wouldn't fit in Claude's context
- **SPA / JS-gated pages**: Gemini has native Google Search and can resolve wch-ic.com landing pages that block direct curl
- **Cross-check**: ask "review this I2S init code, does it look right?" to a different model

Setup (one-time, already done):
```bash
npm install -g gemini-mcp-tool
claude mcp add -s user gemini-cli "C:\Users\Admin\AppData\Roaming\npm\gemini-mcp.cmd"
# restart Claude Code to load tools
```

Verify: `claude mcp list` should show `gemini-cli: ... - ✓ Connected`.

See [`~/.claude/rules/mcp-stack.md`](../../../.claude/rules/mcp-stack.md) for detailed MCP usage notes (model selection, quota behavior, troubleshooting).

## How to use this list

**Quick technical question** (HAL function, register meaning, build flag):
1. grep `doc/MCU/` translations and SDK source in this fork
2. cnlohr/ch32fun GitHub Issues — search before opening new
3. ch32-rs/awesome-ch32 — see if a working example exists elsewhere
4. `#ch32fun` Discord (need invite)

**Software porting question** (Arduino, Rust, PlatformIO):
1. Check if your target framework supports H417 yet (table in Tier 2)
2. If not — port from CH32V307 (closest cousin) and contribute back
3. Open an issue on the relevant repo (`ch32-hal`, `ch32-pio-projects`, etc.)

**Hardware-level question** (electrical, jitter, USB SS PHY layout):
1. WCH official: <https://www.wch.cn/contact_us.html> — email support, EN possible but slow
2. RISC-V Reddit for second opinions
3. CNX Software comments — sometimes WCH engineers respond

**Need a tool**:
- Flash via WCHLink-E: [`ch32-rs/wlink`](https://github.com/ch32-rs/wlink)
- Flash via USB DFU: `wchisp` (Rust)
- Build without MRS: [BUILD.md](BUILD.md) — xPack riscv-none-elf-gcc + Makefile

**Want to contribute**:
- Add H417 to `ch32fun` — biggest ecosystem unlock
- Add H417 to `ch32-data` → unlocks Rust HAL automatically
- Write a tutorial — there's basically zero EN material on this chip; you'd be the first

---

## What's missing in the ecosystem (as of May 2026)

Acknowledging the gaps so we know what's NOT there:
- ❌ No Arduino core confirmed for H417
- ❌ No PlatformIO board definition
- ❌ No Rust HAL support
- ❌ No `ch32fun` support
- ❌ No official Errata published yet
- ❌ No QingKeV5 Processor Manual published yet
- ❌ No English Reference Manual (Chinese only — see `doc/MCU/` for our translations)
- ❌ No Buildroot / Yocto / Zephyr support
- ❌ FreeRTOS port not officially published by WCH

Each of these is a low-hanging-fruit contribution opportunity if you want to become an early visible member of this community.
