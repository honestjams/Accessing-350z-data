# Research findings (round 3)

Confidence: ✅ confirmed from a primary source · 🟡 strong lead, verify on the car · ❓ open

## 1. Your ECU
| Finding | Conf. | Source |
|---|---|---|
| US 2003 manual ECM = **23710-CD003** (supersedes CD000/CD001/CD002). Service label MEC31-5xx. | ✅ | Nissan parts listings |
| ADM part number **not published anywhere online**. Read the sticker on the ECU. | ❓ | — |
| **On RHD (Australian) cars the ECU sits behind the passenger airbag.** Haltech notes ADM tacho is on ECU pin 110. | ✅ | Haltech VQ35DE support page |
| A 350Z "**CD405**" ROM is **512 KiB → SH7055 CPU**. Its EcuFlash definition is tagged JDM / MT / 2003 / SH7055, internal ID string `2WLGHD10` at 0x3188. CD405 is also in UpRev's 2003 MT ROM list. | ✅ file / 🟡 for your car | NisScan forum "350Z bin file" |
| **CD405 may be the ADM (or JDM-shared) ROM.** The people mapping it were NisScan's Tom (Australia) and a **Queensland** owner. Tom said the 350GT maps were "tuned for high octane Japanese fuel which I can not get in Australia". | 🟡 | same thread |

**Action:** pull the passenger airbag/glovebox trim, photograph the ECU label (23710-xxxxx + MEC number), and we'll know whether we're on CD405 or something else.

## 2. Map definition already started for CD405 (EcuFlash, by NisScan's Tom)
Download: `https://nisscan.com/phpBB3/download/file.php?id=46` (`2WLGHD10.xml`, goes in EcuFlash `rommetadata`). 72 table entries (values + axes). Main tables:

| Address | Table (as named) | Notes from the thread |
|---|---|---|
| 0x6749 / 0x6849 / 0x6949 | Timing / Timing 1 / Timing 2 (8-bit 16×16) | "Ign low det add timing". Values >128 = more knock-aggressive. (value − 128) ≈ 0.5° per unit added to base. **Not** raw degrees BTDC. |
| 0x6C09 | Fuel Compensation (8-bit 16×16) | |
| 0x6E19 | Fuel Target / Target AFR (8-bit 8×8) | |
| 0x6F99 | Intake CAM Advance (8-bit 16×16) | VTC |
| 0x7574 / 0x7774 / 0x9C88 / 0x9E88 | MAP_1/2/4/5 (16-bit 16×16) | Probably base ignition timing (identical copies) |
| 0x7974 / 0x7DF4 | ETC1 / ETC2 (16-bit 16×16) | Electronic throttle |
| 0xA088 | MAP_6 | Probably cruise throttle |
| 0x6E99 / 0x6ED9 | unknown 8×8 | "Big impact on fuel" |
| 0x913E | ? | Candidate MAF curve (64×1, 16-bit) |

16-bit data is big-endian. The forum posters preferred **WinOLS (demo)** over EcuFlash for exploring tables. **Nobody reported a successful 350Z flash in that thread.** One owner couldn't get EcuFlash to connect through a Blitz cable.

## 3. Open-source tooling: nissutils (fenugrec, GPLv3)
The companion to nisprog. **This fills the "checksum" and "find the maps" gaps.**
- `nisrom`: analyses a ROM dump (ECUID, checksum locations, keyset detection, ROM layout heuristics)
- `nisckfix1` / `nisckfix2`: **fix the standard Nissan checksum** (sum + xor words at `cks_s`/`cks_x`)
- `romdb/keysets.csv`: **92 known security keysets** (s27k / s36k). nisprog's `guesskey` tries them.
- `ghidra_helpers/`: `nissan_load.py` + register/vector tables for **SH7055 (180 nm and 350 nm)** and SH7058, for disassembling our ROM in Ghidra
- `findrefs` / `findcallargs`: locate table references in code

Plan: dump ROM → `nisrom` → open in Ghidra with these helpers → confirm the CD405 table addresses (or find ours) → edit → `nisckfix` → `nisrom` re-check → flash on the bench ECU first.

## 4. Live-data parameter IDs (for our logger)
- RomRaider's Nissan (NCS) logger definitions **exist**, but only as a zip on the RomRaider forum, which blocks automated access. **You can download it manually in a browser** (logger defs zip, `logger_*_EN_*.xml`) and drop it in `refs/`. Then I can parse the NCS section.
- The Consult-I register numbering (older 14-pin cars, from Nissboard/consultDash) is a **🟡 hypothesis** for the `0xAC … 0x01 <id>` "standard parameter" entries:

| Reg | Parameter | Scaling (Consult-I) |
|---|---|---|
| 0x00/0x01 | RPM (MSB/LSB) | ×12.5 rpm |
| 0x08 | Coolant temp | raw − 50 °C |
| 0x0B | Vehicle speed | ×2 km/h |
| 0x0C | Battery voltage | ×0.08 V |
| 0x0D | Throttle position | (verify) |
| 0x14/0x15 | Injector pulse width | /100 ms |
| 0x16 | Ignition timing | 110 − raw °BTDC |
| 0x17 | AAC/IACV | /2 % |
| 0x1A | A/F alpha (trim) | raw % |

  **Verify each against NDS II's display on the car before trusting it.** Fallback: once the ROM is disassembled, we read any RAM variable directly with the `0x83 FF aa aa aa` entry type.

## 5. Centre cubby (2003)
- ✅ On a **2003**, the cubby's rear pocket unscrews, and those same screws can hold a 7" monitor. Several owners call it a "drop-in". **2004+ cubbies are harder**, so you're lucky to have a 2003.
- ✅ A **182 × 117 × 35 mm** monitor was confirmed to fit.
- ⚠️ The **Raspberry Pi Touch Display 2 is 189.5 × 120 mm** (official docs; ~500 nits per SparkFun). That's 7.5 mm wider than the known-good size, so **probably too big without trimming.** Choose a 7" panel ≤ ~182 × 117 mm, or a 6.5"–7" bare panel with a custom bezel.

## 6. Sensor hardware details
- ✅ VQ35DE oil filter stud: **M20×1.5**. Sandwich plates (Mishimoto, Z1 billet) have **1/8" NPT ports** for pressure and temperature senders.
- ✅ The factory oil pressure switch is **1/8 BSPT** (not NPT), so mind the adapter if you tee off it.
- 🟡 AEM X-Series wideband (30-0300 gauge / 30-0310 inline) on AEMnet CAN: lambda is a 16-bit unsigned value at **0.0001 λ/bit**, system volts 8-bit at 0.1 V/bit, plus status bits (LSU4.9 detected, data valid, sensor fault). The message ID is configurable. A 120 Ω termination is required. Get the exact ID/byte layout from the manual (pp. 10–13) when we wire it.

## Sources
- [Nissan parts: 23710-CD003](https://www.nissanpartsdeal.com/parts/nissan-engine-control~23710-cd003.html) · [Nissan USA ECM list (CD005/CD103)](https://parts.nissanusa.com/v-nissan-350z/engine--control-modules) · [FS1: 23710-CD000](https://www.fs1inc.com/23710-cd000-2003-nissan-350z-3-5l-pcm-engine-computer-ecm-ecu-programmed-plug-play.html)
- [Haltech: VQ35DE (ADM tacho pin, ECU location)](https://support.haltech.com/portal/en/kb/articles/vq35de-engine)
- [NisScan forum: 350Z bin file / CD405 definition](https://nisscan.com/phpBB3/viewtopic.php?t=120&start=10) · [NisScan NDS I (ID display, params)](https://nisscan.com/?p=2627) · [NisScan: MAF g/s on VQ](https://nisscan.com/phpBB3/viewtopic.php?p=99)
- [fenugrec/nissutils](https://github.com/fenugrec/nissutils) · [fenugrec/nisprog](https://github.com/fenugrec/nisprog) · [fenugrec/npkern](https://github.com/fenugrec/npkern)
- [RomRaider: Nissan stock ROM list](https://www.romraider.com/forum/viewtopic.php?f=35&t=11098) · [RomRaider: open-source Nissan flashing tutorial](https://romraider.com/forum/viewtopic.php?f=64&t=14867) · [RomRaider Logger XML guide](https://mailserver.romraider.com/index.php/RomRaider/LoggerXMLReferenceGuide) (forum pages block automated reads; open them in a browser)
- [Nissboard (Consult-I registers)](https://github.com/matiasmenares/Nissboard) · [consultDash](https://github.com/gregsqueeb/consultDash)
- [my350z: 2003 cubby screen fit](https://my350z.com/forum/audio-and-video/509688-question-about-2003-cubby-for-screen-2.html) · [my350z: 7in LCD](https://my350z.com/forum/audio-and-video/320309-7in-lcd.html) · [Raspberry Pi Touch Display 2 docs](https://raspberrypi.com/documentation/accessories/touch-display-2.html) · [SparkFun listing](https://www.sparkfun.com/raspberry-pi-touch-display-2.html)
- [Z1 billet sandwich plate](https://www.z1motorsports.com/engine-oil/z1-motorsports/z1-billet-oil-filter-sandwich-plate-p-58225.html) · [Mishimoto sandwich plate](https://conceptzperformance.com/mishimoto-mmop-tps-oil-filter-sandwich-plate-adapter-universal_p_9167.php) · [350z-uk: oil pressure sensor (BSPT)](https://www.350z-uk.com/topic/130763-oil-pressure-sensor-install/)
- [AEM 30-0300 manual (AEMnet)](https://www.manualslib.com/manual/1273740/Aem-30-0300.html?page=10)
