# COMMUNITY

CH32H417 community is thin (chip launched Dec 2025). General WCH RISC-V (CH32V) is where most traffic lives — apply to H417 by analogy.

## Official WCH

| Resource | URL |
|----------|-----|
| WCH product page (EN) | <https://www.wch-ic.com/products/CH32H417.html> |
| WCH product page (CN) | <https://www.wch.cn/products/CH32H417.html> |
| Official repo (EVT mirror) | <https://github.com/openwch/ch32h417> |
| openwch org (all WCH repos) | <https://github.com/openwch> |
| MounRiver Studio IDE | <http://www.mounriver.com/> |

## Community

| Resource | URL | Notes |
|----------|-----|-------|
| awesome-ch32 (index) | <https://github.com/ch32-rs/awesome-ch32> | start here for anything CH32 |
| ch32fun (cnlohr) | <https://github.com/cnlohr/ch32fun> | most active open stack; **no H417 yet** |
| ch32-rs/ch32-hal | <https://github.com/ch32-rs/ch32-hal> | Rust HAL + Embassy; **no H417 yet** |
| ch32-rs/ch32-data | <https://github.com/ch32-rs/ch32-data> | chip DB; adding H417 here unlocks Rust |
| ch32-rs/wlink | <https://github.com/ch32-rs/wlink> | flash tool, **H417 supported** |
| ch32-riscv-ug/CH32H417 | <https://github.com/ch32-riscv-ug/CH32H417> | datasheets EN+CN + errata (cloned into `doc/MCU/`) |
| Community-PIO-CH32V | <https://github.com/Community-PIO-CH32V> | PlatformIO; H417 not yet |
| #ch32fun Discord | invite via @cnlohr DM | most active real-time chat |
| ch32v.com | <https://www.ch32v.com/> | forum, intermittent |

## Peer chips for porting

When H417 docs are silent, port from the closest cousin:

| Chip | Repo | Why |
|------|------|-----|
| CH32V307 | <https://github.com/openwch/ch32v307> | closest predecessor (V4F + ETH + USB HS) |
| CH32V305 | <https://github.com/openwch/ch32v305> | WCHLink-E debugger on this board uses it |
| CH32V003 | <https://github.com/openwch/ch32v003> | most tutorials, ch32fun primary target |

## News (chip is new — watch for updates)

- [CNX Software — WCH tag](https://www.cnx-software.com/news/wch/)
- [CH32H417 launch (CNX, Jan 2026)](https://www.cnx-software.com/2026/01/03/wch-ch32h417-dual-core-risc-v-mcu-offers-usb-3-0-500mb-s-uhsif-and-fast-ethernet-interfaces/)
- [Hackaday.io — CH32H417 EVB page](https://hackaday.io/page/399709)

## Vendors

- MuseLab (this board): [AliExpress](https://www.aliexpress.com/item/1005005033298927.html) · [Tindie](https://www.tindie.com/products/johnnywu/nanoch32h417-development-board/)
- WCH eval board: [AnalogLamb](https://analoglamb.com/product/wch-ch32h417-usb3-0-development-board/)

## Multi-AI

Gemini Pro/Flash available as MCP tool (`mcp__gemini-cli__ask-gemini`). Use for >200 KB doc analysis or second-opinion review. Config + gotchas: `~/.claude/rules/mcp-stack.md`.
