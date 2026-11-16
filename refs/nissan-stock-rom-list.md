# Nissan stock ROM list — 350Z and related rows

Source: RomRaider forum "Nissan stock ROM list" (fenugrec). Pasted by the owner on 2026-11-16, since the forum blocks automated access. Only the relevant rows are kept.

| Model | Year | Engine | ECUID | HWID | FID IC | Loader | Checksum | IVT2 | ROM download | Notes |
|---|---|---|---|---|---|---|---|---|---|---|
| 350Z Z33 | 03 | VQ35DE | **CD002** | 3GILXN05 | 705513 | 10, 705513 | std,alt | 0x1000 | [id=29704](https://www.romraider.com/forum/download/file.php?id=29704) | Decrypted from repro file; MT ASCD TCS (USDM) |
| 350Z Z33 | 03 | VQ35DE | **CD007** | 3GILXN05 | 705513 | 10, 705513 | std,alt | 0x1000 | — | MT |
| 350Z | ? | VQ35DE? | **CD405** | 2WLGHD10 | 705513 | 10, 705513 | std,alt | 0x1000 | [id=22617](https://www.romraider.com/forum/download/file.php?id=22617) | NisScan EcuFlash def labels it JDM MT 2003 |
| 350Z | 03 | VQ35DE | **CD415** | 2WLGHD10 | 705513 | 10, 705513 | std,alt | 0x1000 | [id=26895](https://www.romraider.com/forum/download/file.php?id=26895) | JDM AT, 7055F40 topcode |
| 350Z | ? | ? | **CD416** | 2WLM1D13 | 705513 | 10, 705513 | std,alt | 0x1000 | [id=28022](https://www.romraider.com/forum/download/file.php?id=28022) | Non-OBD II Hitachi MEC31-571 |
| 350Z | 03 | | **CD700** | 4TTZVEM41N1 | 705513 | 10, 705513 | std,alt | 0x1000 | [id=28675](https://www.romraider.com/forum/download/file.php?id=28675) | market not stated |
| 350Z | 04 | | **CE860** | 4TTZVEM43N3 | 705513 | 10, 705513 | std,alt | 0x1000 | [id=29776](http://romraider.com/forum/download/file.php?id=29776) | 5AT |
| 350Z | 05 | VQ35DE | CF000 | 5ZVSUNH | 705821 | 50, 705521 | std,alt | 0x1000 | [id=26058](https://www.romraider.com/forum/download/file.php?id=26058) | Touring, non-RevUp, MT |
| 350Z | 05 | | CF065 | 5ZVSVNJ | 705821 | 50, 705521 | std,alt | 0x1000 | [id=26105](https://www.romraider.com/forum/download/file.php?id=26105) | JDM; posted ROM corrupt, fixing IVT1 first entries solves it |
| G35 | 03 | | AC200 | 3GIJ7N04 | 705513 | 10, 705513 | std,alt | 0x1000 | — | 6MT USDM (same ECU family) |
| Skyline V35 | 04 | VQ35DE | AM900 | 2WLJ6D11 | 705513 | 10, 705513 | std,alt | 0x1000 | — | 6MT JP import (same family as CD405) |
| G35/V35 | 04 | VQ35DE | AC405 | 5ZVP8N00 | 705821 | 50, 705521 | std,alt | 0x1000 | [id=20246](https://www.romraider.com/forum/download/file.php?id=20246) | USDM 6MT coupe |

## Checksums (from the list notes + nissutils source)
- There are at least three types: **std**, **alt** and **alt2**. All use the same algorithm: a **32-bit addition and a 32-bit XOR of every big-endian 32-bit word** in the covered area, excluding the two words where the stored sum and XOR live. They differ only in the area covered and which locations are skipped.
- 2003 350Z ROMs use **std + alt**. Both must be valid before flashing.
- Fixing them (nissutils): `nisckfix1` writes 3 u32 correction values so the sum and XOR match the stored originals. `nisckfix2` rewrites the stored values instead.
- Some later ROMs also have a RIPEMD-160 hash. **Not** the 705513 family.
- Wiki (blocks automated reads): https://nissanecu.miraheze.org/wiki/Checksums · https://nissanecu.miraheze.org/wiki/Firmware_re#Checksums

## FID 705513 decoded (nissutils `nissan_romdefs.c`)
`FIDIC "SH705513"`, `ROMsize 512 KiB`, `ktype NPK_7055_35` (**SH7055, 350 nm flash → npkern 7055-350nm kernel**), `RAMF_header 0xFFFF8000`, features `STDCKS | ALTCKS | IVT2`.
