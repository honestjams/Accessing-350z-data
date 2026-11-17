# Nissan ECU RE wiki: notes for our 350Z

> Adapted from the Nissan ECU RE wiki pages [Connect](https://nissanecu.miraheze.org/wiki/Connect), [Rom_an_cf43d](https://nissanecu.miraheze.org/wiki/Rom_an_cf43d), [Ecu_hw](https://nissanecu.miraheze.org/wiki/Ecu_hw), [Firmware_re](https://nissanecu.miraheze.org/wiki/Firmware_re) and [Tools](https://nissanecu.miraheze.org/wiki/Tools) (edited 2021–2026), licensed **CC BY-SA 4.0**. This summary is a derivative and is shared under the same license. Pasted by the owner on 2026-11-17. ➕ = our annotations. Boot mode is in `wiki-bootmode.md`.

## 1. Connecting (Connect page) ✅ matches what we got from the nisprog source
- K-line carries two things:
  - **Generic OBD-II (J1979) over ISO 9141**: 5-baud slow init to functional address `0x33`. Basic DTC and freeze-frame only.
  - **"Consult-II" over ISO 14230**: what we use. **10.4 kbps, fast init, tester `0xFC` → ECU `0x10`, physical addressing.**
- StartCommunication exchange:
  ```
  TX: 81 10 FC 81 <cks>
  RX: 83 FC 10 C1 D5 8F <cks>      ← keybytes D5 8F
  ```
- The ECU supports **addressless (short) headers**. Keep-alive (TesterPresent) = `01 3E <cks>` → `01 7E <cks>`. ➕ The logger must send this when idle, or the session times out.
- Supported SIDs: standard `1A` (ID, with mfg option `0x81`), `21` (by local ID), **`22` (by common ID)**, `27`, `3E`, plus Nissan-specific **`A0`, `A3`, `AC`**.
- Some ECUs answer on a different address (e.g. one '04 Titan wanted functional `0x10`). nisprog/freediag have a **`fastprobe`** command to find it.
- "On more recent vehicles it is probably possible to [connect] through the OBD2 CAN lines. **These are not populated on some vehicles.**" ➕ Consistent with the FSM diagram: our OBD pins 6/14 are probably empty.

## 2. Live data via SID 22 "Common Identifiers" (Firmware_re) ⭐ new logging route
- `22 <CIDH> <CIDL>` returns 1–4 bytes of a live or stored value.
- **Support masks work like OBD Mode 1 PID 00:** `22 12 00` → bitmask of supported CIDs `12 01…12 20`; if `12 20` is supported, `22 12 20` → mask for `12 21…12 40`, and so on. Ranges seen: `11xx, 12xx, 13xx, 14xx, 15xx, 16xx`.
- In 6Z68A the **first `12xx` entry points to engine RPM**. Others point at RAM variables (`FFFF84xx`, `FFFF97xx`…).
- ➕ **Plan:** the logger first **enumerates the supported CIDs** from the masks, then maps meanings (RomRaider logger defs, wiki [SID22](https://nissanecu.miraheze.org/wiki/SID22) page, or matching values against NDS II / the FSM data monitor list). RomRaider's `AC` entry type `0x02 <CID>` streams CIDs in the fast `AC`/`21` loop.
- In CF43D the CID descriptor tables are at `0x4330…0x488C`, the **SID 21 local-ID table at `0x490C`**, and the CID support masks at `0xB563…0xB593`. ➕ In our dumped ROM, these tables tell us exactly what each ID reads in RAM.

## 3. CF43D: 2006 USDM 350Z ROM analysis (closest analysed relative)
SH7058, 1 MiB, FID `6GI5VN6`, LOADER60. ➕ Ours is SH7055 / 512 KiB / FID 705513, so addresses differ, but structure and naming carry over.
- Checksums: std sum/xor at `0x8C54`/`0x8C4C`. Alt sum/xor at `0x240C`/`0x2410`, covering `0x2414…0x4166`.
- Keyset: s27k `0x634D157A`, s36k `0x4B1D6294` (known keyset).
- `&EEPROM_read()` `0x4EAF4`, EEPROM port `0xFFFFF766`.
- Similar ROM: CM31C (930 bytes differ, all in the map area).
- Useful functions: `SID21_ReadDataLID 0x1A290` · `SID22_ReadDataCID 0x19D4C` · `SIDAC_readmulti 0x1B1C0` · `SIDA4_readsingle 0x1AB8C` · `SID27_secu 0x1AF30` · `SID34/36/37` · `SIDBF_RAMjump 0x1C184` · `cks_std 0x3034` · `DTC_set 0x62A0C` · `PL5_toggle 0xFAC2` (watchdog) · table readers `RDtable_*` `0x6EC4…0x7490` · interpolators `0x76B0…0x7924`.
- RAM: `SCI1_rxbuf FFFF8000`, `isomsg_data FFFF8003` (incoming 14230 payload; the SID is at FFFF8003), `DTC_curstatus FFFFB21C`, timer block `FFFF3BD4`.
- ROM end: **`ROMvalid_flag? 0xFFF80`, `ROMsignature "NHU\xF8" 0xFFF84`** (1 MiB ROM).

## 4. Firmware structure (Firmware_re)
- **Boot checks:** the reset code checks for the `NHU\xF8` signature and a "ROM written" flag near the ROM end. **If either fails, the ECU boots into a Nissan bootloader mode** that accepts a limited ISO 14230 command set (untested). ➕ Our checksum/validation tool must also verify these bytes before any flash. For a 512 KiB ROM expect them around `0x7FF80`/`0x7FF84`; confirm on our dump.
- **LOADER struct** (`" LOADERxx"`, `"DATABASE"`, CPU string like `SH705513N`) near the ROM start; **FID struct** (FID string, `"DATABASE"`, CPU+code). ➕ Our parser reads both to confirm ECU type before doing anything.
- Two SID trees (normal and rescue/production). Requests land in RAM at `0xFFFF8003`.
- No OS: big **call tables** of function pointers polled in a loop. These are good anchors for disassembly.
- Compiler: Renesas SHC. Watch for sign-extended RAM addresses (`mov.w` 0x8438 → `FFFF8438`), the `shll8` peripheral-base trick, delay slots, and args in r4–r7 with the return in r0.
- **Security "algo 01":** `fwd_01`/`rev_01` with a 32-bit scode. Used for SID27 key from seed, SID36 kernel payload decrypt, and factory firmware decrypt. **Each ROM has 3 keys** (s27k, s36k1, s36k2); s36k2 only matters for factory reflash. Find keys from known keyset lists, similar ECUIDs, `nisrom`, or manually (search for `mov #0x67` building a `67 01 …` response).
- **Kernel upload sequence** (what nisprog does): pad payload to 32 B → cks16 = 16-bit sum of bytes → encrypt with s36k1 → `27` seed/key → `34 80` → `36 XX YY 20 <32 bytes>` (XXYY increments) → `37 cks16` → `BF 00` check → `BF 01` jump. The payload must take over the WDT pulse train.
- **DTCs:** descriptor table of 12-byte entries (code like `0x00000605` for P0605), a 3-byte status per DTC in RAM, support mask bits, and option words (enable/report flags). ➕ Find them by searching for `00 00 06 05`. This is also how DTCs are disabled (e.g. rear O2 after a cat delete).
- **Fuel cut** limits: found via the second-last call table (~item 21–29); u16 values.
- **Map-finding (WinOLS demo):** view 16-bit as **HiLo**, colour by value, and cross-reference a defined ROM. **"Any 350z ROM can be cross referenced with CF48D… even an 03 Manual (CD002)."** Ignition trim maps are the easiest anchor (a High Temp Fuel Comp map sits right before them), then work outward. ➕ CF48D is the best-defined 350Z ROM (2,100+ maps) and Pytrex's CF170 already covers our likely family.

## 5. ECU hardware (Ecu_hw), for bench work and boot-mode recovery
| Type | MCU | Example ECUIDs | Notes |
|---|---|---|---|
| A | SH7058 | **CF43D (MY06 350Z)**, 6Z68A | EEPROM 93C66 · L9762 (regulators + CAN + K transceivers) · 789101 supervisor · 10 MHz xtal · **WDT = 250 Hz (2 ms high / 2 ms low) on PL5** · K-line on SCI1 (CPU pins 165/166) · test points: A = VBat/VBat/GND/GND/+5V; B = PB14, SCI1_TX, MD1, SCI1_RX, PL5 (via 1 kΩ), nRES (3 k pull-up) |
| **B** | **SH7055** | **EQ910 (MY05 X-Trail, Australia)** | More compact PCB, **MCU rotated 180°**, probably the same chips as A; EEPROM on the same PORTJ pins |
| C | SH7058 | ZD89A | |
| D | SH7055 0.18 µ | HR16DE ECUs | EEPROM M95320 on SCI2 |
| F | SH7055 0.35 µ | AU40B, 8U023 (FID 705507) | EEPROM 93C66 |
| E | dual SH7051 | older | |

➕ **Our 2003 350Z (FID 705513, "loader 10")** shares its FID with **EQ910 = type B**. So expect a **type-B board**, with signals equivalent to type A's (WDT likely 250 Hz on PL5). Confirm with continuity checks on the spare ECU before any boot-mode wiring. Outside markings to record: the MEC number on the label (CF43D = "MEC60-122").

## 6. Tools (Tools page)
freediag/nisprog (compiled builds: RomRaider `viewtopic.php?f=45&t=13259`) · IDA (+ fenugrec's `sh3.cfg` with Nissan secondary-IVT defs) · **Ghidra** (free) · Notepad++ (XML defs) · **WinOLS demo** (map hunting) · Renesas HEW (simulator, core only, no SCI/ATU/CAN) · **Renesas FDT** (boot-mode flashing).

## 7. Wiki pages still worth copying (same "Edit → copy" method)
[SID22](https://nissanecu.miraheze.org/wiki/SID22) (CID meanings) ⭐ · [C2_14230](https://nissanecu.miraheze.org/wiki/C2_14230) (Consult-II protocol) ⭐ · [Wdt](https://nissanecu.miraheze.org/wiki/Wdt) · [Checksums](https://nissanecu.miraheze.org/wiki/Checksums) · [Eeprom_code](https://nissanecu.miraheze.org/wiki/Eeprom_code)
