# Pre-build research: what we know before writing the tools

Sources are mostly **source code** (nisprog, freediag, RomRaider) and the Pytrex definitions, so they're far more reliable than forum posts.

## 1. The K-line conversation, byte by byte ✅ (nisprog `nis_backend.c`, `np_cli.c`; freediag `diag_l0_dumb.c`)

| Step | Request | Positive reply | Notes |
|---|---|---|---|
| Wake-up | 25 ms low / 25 ms high on K | — | Either a serial **break of 25 ms**, or freediag's trick: **set 360 baud, send `0x00`** (9 low bits × 2.78 ms ≈ 25 ms), then back to 10400. Wait >300 ms idle before the first init, >55 ms after a stop. |
| Start comms | `81` (long header `81 10 FC 81 0E`) | `C1 …` | ECU `0x10`, tester `0xFC` |
| ECU ID | `1A 81` | `5A …` (≥7 bytes) | Gives the ROM ID, e.g. CD700 |
| Load addresses | `AC 81` + up to **12 ×** `83 aa aa aa aa` (4-byte address) | `EC 81` | No security access needed |
| Read them | `21 81 04 01` | `61 81 <n bytes>` | One byte per loaded address. `06` instead of `04` = fast poll (RomRaider). |
| Diag session (programming) | `10 85 14` | `50 …` | Used by nisprog before flashing |
| Security access | `27 01` → seed, `27 02 <key>` | `67 …` | **Only for kernel/flash**, not for logging |
| Kernel upload | `34 80`, `36 …`, `37 <cks>`, `BF 00/01` (RAM jump) | `74`, `76`, `77`, `FF` | nisprog does this. We don't reimplement it. |
| Stop | `82` | `C2` | |

**Negative responses worth handling** (`7F <sid> <nrc>`): `0x94` engine running · **`0x95` electrical loads present** (why the tutorial says switch off lights, fan and head unit) · `0x93` additional conditions not met · `0x90`/`0x91` bad SID36 sequence / bad SID37 checksum.

**Cable quirk:** a KKL cable is half-duplex, so **every byte we send echoes back** and must be discarded before reading the reply (nisprog's `dumbopts 0x48` = manual break + echo removal).

**Throughput:** each `AC` request carries ≤12 addresses, so logging ~12 single-byte values costs about (5×12+4) + 6 bytes out plus 4+12 back per cycle. At 10400 baud that's realistically ~10–20 Hz for a small set.

## 2. A risk-free ROM backup is possible ✅
nisprog's `dumpmem` **without the kernel loaded** falls back to `dump_fast()`, which is just `AC`/`21` reads: **no security unlock, no flash mode, nothing written.** It runs at about 100 B/s, so the 512 KiB ROM takes **~1.5 hours**. Our own Python tool can do the same.
➡️ **New first step:** take a read-only ROM backup with our own tool, cross-check it against a nisprog read-only dump, and only then ever load a kernel.

## 3. How the ECU decides ignition timing ✅ (A2L template descriptions) + 🟡 (forums)
```
final advance ≈ MBT base advance   (MBT Flame Speed Map, by load × RPM)
              + trim map           (High Octane Ignition Timing Map, or High Detonation map when knock is learned)
              + compensations      (cold, idle, accel retard K1, decel advance K2Y, …)
              − knock retard       (only inside the knock window)
```
- Trim-map values are stored as **(x − 64)°**. The **top bit (0x80) is the knock-window flag** for that cell (mask 0x7F = timing, 0x80 = window). This matches NisScan's ">128" observation on CD405.
- Inside the knock window, knock → timing pulled and **held** retarded, then slowly added back. Outside it, knock is largely ignored unless a knock flag trips, at which point the ECU runs the **High Detonation** map.
- The ECU **learns**: the first pull after a change isn't representative. Log several pulls (VQ37 tuners report timing creeping back over consecutive runs).
- Load axis = **Cylinder Filling Up Efficiency** (×0.78125 %), RPM axis ×50.
- Target AFR cells at **14.7 = closed loop**. Target AFR scaling is `1881.6 / x`.
- MAF tuning advice from the definitions author: with a stock-diameter intake, prefer **MAF Factor + Fuel Compensation** over editing the MAF table.

Full table reference (81 tables, with addresses, scaling, axes, original Nissan names and descriptions): **`refs/cf170-tables.md`**.

## 4. Live-data parameters
- RomRaider's Nissan logger names parameters `P2` coolant, `P11` IAT, `P17` battery V, `P4389`/`P4390` learned fuel trim B1/B2, `E173`/`E174` mixture-learning trim/count. The **byte IDs live in RomRaider's logger XML** (forum `viewtopic.php?f=8&t=1642`), which we still need.
- **Plan B needs no IDs:** the `AC 81 83 <addr>` form reads any RAM address. Once we have the ROM, disassembling it (Ghidra + nissutils helpers) finds where RPM, load, timing, knock retard etc. live in RAM. That's more work but fully under our control.

## 4b. CAN broadcast data 🟡 (FSM EC-30, corrected by EC-631/632)
The EC-30 table lists the OBD connector as a CAN unit, **but the EC-631/632 wiring diagrams show no CAN wires to OBD pins 6/14**, so check the connector. The reliable tap is **meter/A/C amp connector M48 pin 1 (CAN-H) / pin 11 (CAN-L)** behind the centre stack. The ECM broadcasts engine speed, coolant, accelerator position and fuel consumption, and other units broadcast vehicle speed, fuel level, lamps and doors. **The dash should read these passively over CAN** (fast, zero ECM load) and use K-line polling only for what CAN doesn't carry (timing, MAF, injector PW, trims, cam angle, knock via RAM). That needs a CAN interface on the hub (already in the Tier 3 list) and an ID-sniffing session.

## 4c. ⭐ SID 22 common identifiers (wiki Firmware_re/Connect; see `refs/wiki-notes.md`)
`22 12 00` returns a **bitmask of supported live-data CIDs** (like OBD Mode 1 PID 00), chained through `12 20`, `12 40`… across ranges `11xx–16xx`. The logger can therefore **discover what the ECU supports by itself**. Meanings come from RomRaider's logger XML, the wiki SID22 page, or by matching values against NDS II / FSM data-monitor readings. The **keep-alive** is `01 3E` → `01 7E` (short header), and must be sent when idle.

## 5. Tool architecture decision
| Job | Approach | Why |
|---|---|---|
| Logging, ECU ID, read-only ROM dump | **Our Python tool** (pyserial + KKL cable) | Low risk, runs on Mac/Linux/Pi, feeds the dash |
| ROM checksums, table view/edit, diff | **Our Python tool**, driven by the Pytrex definitions | Testable offline. Independent check of RomRaider. |
| **Flashing** | **Wrap nisprog (Windows)**, don't reimplement | Highest-risk step. nisprog is battle-tested; a homemade flasher is how ECUs get bricked. |

## 6. Bench ECU harness
UpRev sells "Bench-Top harness" variants (Gen1, Gen3, HR & VHR, Juke) at $175 USD. 🟡 **Gen1 likely fits 2003–2006 VQ35DE ECUs** (confirm with UpRev). It's only a wiring loom, so it works with nisprog. Otherwise build one from the FSM ECM pinout (BAT, IGN, grounds, K-line).

## 7. Still open (owner can fetch in a browser)
| Item | Where | Why |
|---|---|---|
| RomRaider **logger definition**: `logger_v370.zip` (+ `logger.dtd`) | romraider.com forum `viewtopic.php?f=8&t=1642` (attachment; may need a free login) | Parameter byte IDs + scalings. Nissan K-line params were added in **v350**, Nissan CAN in **v370**. We want `logger_METRIC_EN_v370.xml`. |
| ~~2003 350Z FSM, EC section~~ ✅ received (EC-1…198); see `refs/fsm-ec-2003-350z.md`. Optional: last part with SDS (≈EC-650) | — | — |
| Nissan ECU wiki (CC BY-SA; behind a bot check, so copy pages by hand) | ✅ Bootmode, Connect, Rom_an_cf43d, Ecu_hw, Firmware_re, Tools (saved: `refs/wiki-bootmode.md`, `refs/wiki-notes.md`). Still useful: **[SID22](https://nissanecu.miraheze.org/wiki/SID22)** (CID meanings), **[C2_14230](https://nissanecu.miraheze.org/wiki/C2_14230)**, **[Wdt](https://nissanecu.miraheze.org/wiki/Wdt)**: watchdog pulse spec for boot mode · **2. [Rom_an_cf43d](https://nissanecu.miraheze.org/wiki/Rom_an_cf43d)**: '06 350Z VQ35DE ROM analysis (RAM variables, structures) · **3. [Connect](https://nissanecu.miraheze.org/wiki/Connect)** · **4. [Ecu_hw](https://nissanecu.miraheze.org/wiki/Ecu_hw)**: hardware versions, bench pinouts · **5. [Firmware_re](https://nissanecu.miraheze.org/wiki/Firmware_re)**: checksums, keys, algorithms · 6. [Tools](https://nissanecu.miraheze.org/wiki/Tools) · 7. [Eeprom_dumps](https://nissanecu.miraheze.org/wiki/Eeprom_dumps) | Recovery plan, RAM addresses for logging, bench wiring |
| RomRaider unbricking tutorial | `viewtopic.php?f=7&t=12821` ("Nissan ECU unbricking tutorial with shbootmode") | Step-by-step recovery |
| ECU ID | EPC VIN lookup | Confirms CD700/CD800 |

## Sources
[fenugrec/nisprog](https://github.com/fenugrec/nisprog) · [fenugrec/freediag](https://github.com/fenugrec/freediag) · [RomRaider source](https://github.com/RomRaider/RomRaider) · [Pytrex/NissanDefinitions](https://github.com/Pytrex/NissanDefinitions) · [RomRaider: Ignition timing — how it really works](https://www.romraider.com/forum/viewtopic.php?t=15017) · [RomRaider: VQ35DE timing drop-off](https://www.romraider.com/forum/viewtopic.php?t=16312) · [RomRaider: 91 vs 94 octane, will ECU adapt (VQ37)](https://www.romraider.com/forum/viewtopic.php?t=17389) · [TwinTurbo.net: Z32 knock control](http://twinturbo.net/nissan/300zx/forums/technical/view/1127497/Z32-Knock-Control---Do-you-know-your-ABCs.html) · [Link ECU forum: VQ35DE](https://forums.linkecu.com/topic/9100-nissan-vq35de/) · [Nissan ECU wiki: CF48D analysis](https://nissanecu.miraheze.org/wiki/Rom_an_cf48d) · [Nissan ECU Tuning Basics (PDF)](https://www.xenonzcar.com/z31/images/nistune/nissan_ecu_tuning_basics.pdf) · [UpRev shop (bench harnesses)](https://www.uprev.com/shop/)
