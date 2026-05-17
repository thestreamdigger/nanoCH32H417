# CHRV3UFI — USB Host + FAT12/16/32 Library (English reference)

Translated reference for the WCH CHRV3UFI library, shipped pre-built as `libRV3UFI.a` plus header `CHRV3UFI.h`. This is the **critical finding** for the FLAC player project: WCH already provides the complete USB Host Mass Storage + FAT filesystem stack — no TinyUSB or FatFs port needed.

Source location in this fork: `doc/EVT/EXAM/USBHS/HOST/Udisk_Lib/`
Working example using it: `doc/EVT/EXAM/USBHS/HOST/Host_UDisk_Exams/`

## What it does

- USB Host with **MSC class** (Mass Storage Class) — talks to USB pendrives, SD card readers, external HDDs
- Supports all USB speeds: **FS (12 Mbps), HS (480 Mbps), SS (5 Gbps)**
- Works through internal root-hub OR external USB hub
- **Built-in FAT12 / FAT16 / FAT32** filesystem driver
- **Long-filename** support (`UDisk_Func_LongName.c` companion)
- File and directory operations (open / read / write / create / erase / locate)
- Both sector-mode and byte-mode access

Library version: **CHRV3UFI V1.0** (last update 2020.05, header rev 2014.09.09)

## Library files

| File | Purpose |
|------|---------|
| `libRV3UFI.a` | Precompiled archive (RV32 GCC 8.20 compatible) — black box |
| `CHRV3UFI.h`  | Public API: constants, structs, function prototypes |
| `CH32H417UFI.c` | H417-specific glue code (bridges library to USBHS hardware driver) |
| `CHRV3UF_README.TXT` | Brief readme (GBK encoded) |

## Application-side companion files

In `Host_UDisk_Exams/Common/Host_UDisk/`:

| File | Purpose |
|------|---------|
| `UDisk_HW.c` | Hardware init: USBHS clocks, root-hub, port detection |
| `UDisk_Operation.h` | Application-level helpers |
| `Udisk_Func_BasicOp.c` | Demo: create + write + read + modify files |
| `UDisk_Func_CreatDir.c` | Demo: create subdirectories |
| `UDisk_Func_LongName.c` | Demo: long-filename (LFN) handling |

In `Host_UDisk_Exams/Common/USB_Host/`:

| File | Purpose |
|------|---------|
| `ch32h417_usbhs_host.c/.h` | USBHS controller driver (host mode, low-level) |
| `usb_host_config.h` | Buffer sizes, endpoint config |

## API surface

### Error codes

| Code | Symbol | Meaning |
|------|--------|---------|
| 0x00 | `ERR_SUCCESS` | OK |
| 0x13 | `ERR_USB_CONNECT_LS` | Low-speed USB device connect detected |
| 0x15 | `ERR_USB_CONNECT` | USB device connect event, disk attached |
| 0x16 | `ERR_USB_DISCON` | USB device disconnect event |
| 0x17 | `ERR_USB_BUF_OVER` | Transfer data corrupted or buffer overflow |
| 0x1F | `ERR_USB_DISK_ERR` | USB storage operation failed (init: unsupported device; r/w: damaged or disconnected disk) |
| 0x20–0x2F | `ERR_USB_TRANSFER` | NAK / STALL / various transfer errors |
| 0x41 | `ERR_OPEN_DIR` | Directory at given path was opened |
| 0x42 | `ERR_MISS_FILE` | File at given path not found (possibly wrong filename) |
| 0x43 | `ERR_FOUND_NAME` | Wildcard match found (path in command buffer); open to use |
| 0x81 | `ERR_CHRV3_ERROR` | Hardware error — may need to reset CHRV3 |
| 0x83 | `ERR_STATUS_ERR` | Disk state error — disk connecting / disconnecting |
| 0x84 | `ERR_HUB_PORT_FREE` | External HUB attached but its port has no disk |
| 0x91 | `ERR_MBR_ERROR` | MBR invalid — disk not partitioned / not formatted |
| 0x92 | `ERR_TYPE_ERROR` | Unsupported partition type (only FAT12/16/BigDOS/FAT32) |
| 0xA1 | `ERR_BPB_ERROR` | Not formatted or wrong params — reformat using Windows defaults |
| 0xA2 | `ERR_TOO_LARGE` | Abnormal format + capacity > 4 GB, or > 250 GB |
| 0xA3 | `ERR_FAT_ERROR` | Filesystem unsupported (only FAT12/16/32) |
| 0xB1 | `ERR_DISK_FULL` | Insufficient free space — defragment |
| 0xB2 | `ERR_FDT_OVER` | Too many files in directory (FAT12/16 root limited to ~500) |
| 0xB3 | `ERR_MISS_DIR` | Subdirectory in path not found |
| 0xB4 | `ERR_FILE_CLOSE` | File closed — reopen if needed |

### Disk states (`CHRV3DiskStatus` global)

| Value | Symbol | Meaning |
|-------|--------|---------|
| 0x00 | `DISK_UNKNOWN` | Uninitialized |
| 0x01 | `DISK_DISCONNECT` | No disk connected / detached |
| 0x02 | `DISK_CONNECT` | Disk connected but not initialized |
| 0x04 | `DISK_USB_ADDR` | USB address assigned, USB not yet configured |
| 0x05 | `DISK_MOUNTED` | Disk mounted, filesystem not analyzed yet |
| 0x10 | `DISK_READY` | **Filesystem analyzed and supported — ready for file ops** |
| 0x12 | `DISK_OPEN_ROOT` | Root directory open, sector mode |
| 0x13 | `DISK_OPEN_DIR` | Subdirectory open, sector mode |
| 0x14 | `DISK_OPEN_FILE` | File open, sector mode |
| 0x15 | `DISK_OPEN_FILE_B` | File open, byte mode |

### FAT type (`CHRV3vDiskFat` global)

| Value | Symbol |
|-------|--------|
| 0 | `DISK_FS_UNKNOWN` |
| 1 | `DISK_FAT12` |
| 2 | `DISK_FAT16` |
| 3 | `DISK_FAT32` |

### Key functions

```c
uint8_t CHRV3LibInit(void);              // Init library, returns 0 on success
uint8_t CHRV3GetVer(void);               // Library version
uint8_t CHRV3DiskConnect(void);          // Probe disk + update state
uint8_t CHRV3DiskReady(void);            // Query disk ready
uint8_t CHRV3DiskQuery(void);            // Total + free sector count

uint8_t CHRV3FileOpen(void);             // Open file or enumerate (path in mCmdParam.Open.mPathName)
uint8_t CHRV3FileClose(void);            // Close current file
uint8_t CHRV3FileCreate(void);           // Create + open (overwrites if exists)
uint8_t CHRV3FileErase(void);            // Delete file and close
uint8_t CHRV3FileQuery(void);            // Read attributes of current file
uint8_t CHRV3FileModify(void);           // Update attributes

uint8_t CHRV3FileLocate(void);           // Seek by sector (mCmdParam.Locate.mSectorOffset)
uint8_t CHRV3FileRead(void);             // Read N sectors (mCmdParam.Read)
uint8_t CHRV3FileWrite(void);            // Write N sectors

uint8_t CHRV3ByteLocate(void);           // Seek by byte
uint8_t CHRV3ByteRead(void);             // Read N bytes (mCmdParam.ByteRead)
uint8_t CHRV3ByteWrite(void);            // Write N bytes

void    CHRV3SaveVariable(void);         // Save/restore lib state (multi-disk switching)
void    CHRV3DirtyBuffer(void);          // Invalidate disk buffer (force re-read)
```

### Globals you'll touch

```c
extern volatile uint8_t CHRV3IntStatus;    // Latest interrupt status
extern volatile uint8_t CHRV3DiskStatus;   // Disk + file state (see table above)
extern uint8_t  CHRV3vDiskFat;             // FAT type 1/2/3
extern uint8_t  CHRV3vSecPerClus;          // Sectors per cluster
extern uint16_t CHRV3vSectorSize;          // Sector size in bytes (typ 512)
extern uint16_t CHRV3vPacketSize;          // USB max packet 64@FS / 512@HS/SS (you set this)
extern uint32_t CHRV3vFileSize;            // Current file size
extern uint32_t CHRV3vCurrentOffset;       // Current byte offset

extern uint8_t  *pDISK_BASE_BUF;           // Pointer to sector buffer (you allocate)
extern uint8_t  *pDISK_FAT_BUF;            // Pointer to FAT buffer (you allocate)
extern uint32_t *pTX_DMA_A_REG;            // TX DMA address register (you wire)
extern uint32_t *pRX_DMA_A_REG;            // RX DMA address register
extern uint16_t *pTX_LEN_REG;              // TX length register
extern uint16_t *pRX_LEN_REG;              // RX length register

extern CMD_PARAM_I mCmdParam;              // Command parameter union (set fields, then call function)
```

### `mCmdParam` — the universal parameter passing struct

C-style union, you set the relevant struct's fields then call the matching function:

```c
mCmdParam.Open.mPathName       // CHRV3FileOpen — null-terminated path, max 64 bytes
mCmdParam.Create.mPathName     // CHRV3FileCreate
mCmdParam.Erase.mPathName      // CHRV3FileErase

mCmdParam.Modify.mFileSize     // CHRV3FileModify — 0xFFFFFFFF = no change
mCmdParam.Modify.mFileDate     // 0xFFFF = no change; use MAKE_FILE_DATE(y,m,d)
mCmdParam.Modify.mFileTime     // use MAKE_FILE_TIME(h,m,s)
mCmdParam.Modify.mFileAttr     // 0xFF = no change

mCmdParam.Locate.mSectorOffset // CHRV3FileLocate — 0 = beginning, 0xFFFFFFFF = end
mCmdParam.ByteLocate.mByteOffset // CHRV3ByteLocate — same convention

mCmdParam.Read.mSectorCount    // CHRV3FileRead
mCmdParam.Read.mDataBuffer

mCmdParam.ByteRead.mByteCount  // CHRV3ByteRead
mCmdParam.ByteRead.mByteBuffer

mCmdParam.Query.mTotalSector   // CHRV3DiskQuery returns here
mCmdParam.Query.mFreeSector
```

## Typical flow (from `Udisk_Func_BasicOp.c`)

```c
/* 1. Init phase — done once at boot */
CHRV3vPacketSize = 512;          // HS or SS device — for FS use 64
pDISK_BASE_BUF = MY_DATA_BUF;    // your 512+ byte sector buffer
CHRV3LibInit();
USBHS_Host_Init();               // hardware: clocks, root-hub, IRQ

/* 2. Main loop — wait for disk */
while (1) {
    UDisk_USBH_DiskReady();       // polls connect + mounts FS
    if (CHRV3DiskStatus >= DISK_MOUNTED) {
        printf("Disk mounted\r\n");

        /* 3. Open file */
        strcpy((char *)mCmdParam.Open.mPathName, "/SONG.FLAC");
        if (CHRV3FileOpen() == ERR_SUCCESS) {

            /* 4. Read in chunks */
            uint8_t chunk[512];
            while (CHRV3vCurrentOffset < CHRV3vFileSize) {
                mCmdParam.ByteRead.mByteCount = 512;
                mCmdParam.ByteRead.mByteBuffer = chunk;
                if (CHRV3ByteRead() != ERR_SUCCESS) break;
                /* feed chunk to FLAC decoder → I2S DMA */
                feed_decoder(chunk, mCmdParam.ByteRead.mActCnt);
            }

            /* 5. Close */
            mCmdParam.Close.mUpdateLen = 0;  // 1 if you wrote
            CHRV3FileClose();
        }
    }
    /* Else: disk dropped — handle hot-unplug */
}
```

### Path conventions
- Separator: `\` or `/` (both accepted)
- Drive letter optional: `C:\DIR\FILE.EXT` or `/DIR/FILE.EXT`
- Wildcard `*` for enumeration mode
- Max 64 bytes including null terminator (`MAX_PATH_LEN`)

### File time/date helpers
```c
mCmdParam.Modify.mFileTime = MAKE_FILE_TIME(13, 45, 30);  // 13:45:30
mCmdParam.Modify.mFileDate = MAKE_FILE_DATE(2026, 5, 17); // 2026-05-17
```

## Important caveats for the FLAC player

1. **`CHRV3vPacketSize`** must be set **before** `CHRV3LibInit()`:
   - 64 bytes for USB FS
   - 512 bytes for USB HS / SS
   The board's USB-3.0-SS Type-A connector typically negotiates HS for USB 2.0 pendrives — use 512.

2. **`pDISK_BASE_BUF`** must point to a **4-byte aligned** buffer of at least `CHRV3vSectorSize` bytes (typ 512, can be 2048/4096 for high-capacity drives). Use `__attribute__((aligned(4)))`.

3. **Sub-class quirks**: `CHRV3vSubClassIs6` global — set to 1 if your stick reports subclass 6, 0 otherwise. Most modern sticks need 0.

4. **Polling vs interrupt**: the library uses synchronous calls with internal NAK retry. For a FLAC player with concurrent I2S DMA, dedicate the V5F core to USB+decode and let I2S run autonomously on DMA. Or use the V3F as USB+decode and V5F for DSP/UI.

5. **Re-entrancy**: NOT re-entrant. Don't call CHRV3 functions from ISR.

6. **Read throughput**: in practice ~5–15 MB/s sustained from typical USB 2.0 HS sticks. Far exceeds FLAC bitrate (worst case ~5 Mbps for 24/192 stereo).

## Known limits

- **No exFAT** — pendrives over 32 GB formatted as exFAT will fail. Reformat as FAT32.
- **Partition < 4 GB recommended** unless using default Windows format.
- **FAT12/16 root directory** capped at ~500 files. FAT32 has no such limit.
- **No journaling, no UTF-8 native** (LFN supports Unicode-16 internally).

## Quick choice for the player

For a FLAC pendrive player:
- Format USB stick as **FAT32**
- Use **byte-mode read** (`CHRV3ByteRead`) — easier to pace into FLAC decoder
- Read chunks of 4 KB (`mByteCount = 4096`) — aligns with FAT32 cluster boundary
- Keep `pDISK_BASE_BUF` size at 4096 for performance
