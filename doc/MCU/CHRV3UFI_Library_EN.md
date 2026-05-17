# CHRV3UFI — USB Host + FAT12/16/32 (WCH library)

Pre-built MSC class driver + FAT filesystem from WCH. No TinyUSB or FatFs needed.

- Source: `doc/EVT/EXAM/USBHS/HOST/Udisk_Lib/` (`libRV3UFI.a` + `CHRV3UFI.h`)
- Working example: `doc/EVT/EXAM/USBHS/HOST/Host_UDisk_Exams/`
- Supports USB FS/HS/SS, root-hub + external hub, FAT12/16/32, long filenames
- Sector-mode and byte-mode access
- Version: V1.0 (2020.05)

## Error codes (sample)

| Code | Symbol | Meaning |
|------|--------|---------|
| 0x00 | `ERR_SUCCESS` | OK |
| 0x15 | `ERR_USB_CONNECT` | Disk attached |
| 0x16 | `ERR_USB_DISCON` | Disk removed |
| 0x1F | `ERR_USB_DISK_ERR` | Read/write failed |
| 0x42 | `ERR_MISS_FILE` | File not found |
| 0x81 | `ERR_CHRV3_ERROR` | HW error — reset CHRV3 |
| 0x91–0xA3 | MBR/BPB/FAT errors | Reformat the disk |
| 0xB1 | `ERR_DISK_FULL` | No space |
| 0xB3 | `ERR_MISS_DIR` | Path subdirectory missing |

Full list in `CHRV3UFI.h`.

## Disk state machine (`CHRV3DiskStatus`)

| Value | Symbol |
|-------|--------|
| 0x00 | `DISK_UNKNOWN` |
| 0x01 | `DISK_DISCONNECT` |
| 0x02 | `DISK_CONNECT` |
| 0x04 | `DISK_USB_ADDR` |
| 0x05 | `DISK_MOUNTED` |
| 0x10 | `DISK_READY` *(file ops OK)* |
| 0x12–0x15 | `DISK_OPEN_*` |

## Key functions

```c
uint8_t CHRV3LibInit(void);
uint8_t CHRV3DiskConnect(void);
uint8_t CHRV3DiskReady(void);
uint8_t CHRV3FileOpen(void);
uint8_t CHRV3FileClose(void);
uint8_t CHRV3FileCreate(void);
uint8_t CHRV3FileErase(void);
uint8_t CHRV3FileLocate(void);     /* by sector */
uint8_t CHRV3FileRead(void);
uint8_t CHRV3FileWrite(void);
uint8_t CHRV3ByteLocate(void);     /* by byte */
uint8_t CHRV3ByteRead(void);
uint8_t CHRV3ByteWrite(void);
```

## `mCmdParam` — parameter union

Set the field for the call you're about to make:

```c
mCmdParam.Open.mPathName        /* CHRV3FileOpen — null-terminated path, ≤64 B */
mCmdParam.Read.mSectorCount     /* CHRV3FileRead */
mCmdParam.Read.mDataBuffer
mCmdParam.ByteRead.mByteCount   /* CHRV3ByteRead */
mCmdParam.ByteRead.mByteBuffer
mCmdParam.Locate.mSectorOffset  /* 0 = head, 0xFFFFFFFF = tail */
mCmdParam.Query.mTotalSector    /* return from CHRV3DiskQuery */
mCmdParam.Query.mFreeSector
```

Path: `/` or `\` separator, drive letter optional, `*` wildcard for enumeration.

## Globals you set

```c
extern uint16_t CHRV3vPacketSize;  /* set before CHRV3LibInit(): 64=FS, 512=HS/SS */
extern uint8_t  *pDISK_BASE_BUF;   /* 4-byte-aligned buffer ≥ sector size */
extern uint8_t  CHRV3vSubClassIs6; /* 0 for most modern sticks */
```

## Typical flow

```c
CHRV3vPacketSize = 512;
pDISK_BASE_BUF = MY_DATA_BUF;       /* aligned(4), ≥512 B */
CHRV3LibInit();
USBHS_Host_Init();

while (1) {
    UDisk_USBH_DiskReady();
    if (CHRV3DiskStatus >= DISK_MOUNTED) {
        strcpy((char *)mCmdParam.Open.mPathName, "/SONG.FLAC");
        if (CHRV3FileOpen() == ERR_SUCCESS) {
            uint8_t chunk[4096];
            while (CHRV3vCurrentOffset < CHRV3vFileSize) {
                mCmdParam.ByteRead.mByteCount = sizeof chunk;
                mCmdParam.ByteRead.mByteBuffer = chunk;
                if (CHRV3ByteRead() != ERR_SUCCESS) break;
                feed_decoder(chunk, mCmdParam.ByteRead.mActCnt);
            }
            mCmdParam.Close.mUpdateLen = 0;
            CHRV3FileClose();
        }
    }
}
```

## Gotchas

- Not re-entrant — never call from ISR
- USB stick must be FAT12/16/32 (no exFAT) — reformat sticks >32 GB
- FAT32 cluster-aligned 4 KB reads give best throughput
- Throughput ~5–15 MB/s sustained on USB HS — plenty for FLAC
- `__attribute__((aligned(4)))` on data buffers is mandatory
