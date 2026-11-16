# 2003 350Z (Z33, VQ35DE): how the car talks

## The short version

| Question | Answer | Confidence |
|---|---|---|
| Engine / ECU family | VQ35DE "pre-Rev-Up", Hitachi ECM, flash-reprogrammable | High |
| Diagnostic link on the OBD port | **K-line (pin 7)**, KWP2000 / ISO 14230 framing, Nissan "Consult-II / DDL2" services | High (NisScan, UpRev guide, RomRaider source) |
| Generic OBD-II (emissions PIDs) | ISO 9141-2 / KWP2000 over the same K-line: slow, few PIDs | Medium |
| CAN on the OBD port? | **No for engine diagnostics.** The car *does* have an internal 500 kbps CAN bus between ECM, TCM, ABS, meter, IPDM, but on a 2003 it's not the diagnostic path | Medium — check pins 6/14 on your connector |
| Factory wideband O2? | **No** (2003 and early 2004 cars) — must add one for tuning | High (multiple tuner reports) |
| ROM IDs UpRev supports | 2003 MT: `CD000`–`CD003`; 2003 AT: `CD100`–`CD102` | High (uprev.com support table, 2026) |

The big implication: **cheap Bluetooth ELM327 clones are a gamble on this car.** They're built for CAN cars and often handle K-line poorly or not at all. Buy a quality STN-based adapter (OBDLink) or a proper K-line interface.

## Protocol details for writing our own logger

These come from RomRaider's open-source Nissan ("NCS") K-line implementation
(`src/main/java/com/romraider/io/protocol/ncs/iso14230/NCSProtocol.java`), which logs these ECUs.

**Physical layer**
- K-line, single-wire half-duplex UART, 0/12 V
- **10400 baud, 8N1**
- P3min (inter-request gap) = 5 ms in RomRaider's settings
- Wake-up: **ISO 14230 fast init** (25 ms low / 25 ms high pulse), then `StartCommunication`

**Frame format** (KWP2000)
```
Long header:  [0x80 + len] [target addr] [tester addr] [SID] [data...] [checksum]
Short header: [len] [SID] [data...] [checksum]
checksum = sum of all preceding bytes & 0xFF
```

**Services used**
| SID | Response | Purpose |
|---|---|---|
| `0x81` | `0xC1` | StartCommunication (sent right after fast init, long header) |
| `0x82` | `0xC2` | StopCommunication |
| `0x1A 0x81` | `0x5A` | Read ECU identification (gives ROM ID, e.g. `CD003`) |
| `0xAC 0x81 …` | `0xEC` | **Nissan-specific "load address"**: tell the ECU which parameters/RAM addresses you want in the stream |
| `0x21 0x81 0x04 0x01` | `0x61` | Read the loaded parameter block (`0x06` instead of `0x04` = fast-poll mode) |
| `0x7F` | — | Negative response code |

`0xAC` entries are typed: `0x01 xx` = Consult/standard parameter (local ID), `0x02 xx xx` = SID 0x22-style ID, `0x83 0xFF aa aa aa` = direct RAM address (3-byte, high bit set).

**Throughput caveat:** RomRaider users report only ~5–6 RAM parameters per request fit in a K-line packet, so the sample rate drops as you add channels. Plan for ~10–20 Hz with a small channel set.

**Addressing (confirmed by nisprog's documented Nissan settings):** `l2protocol iso14230 / initmode fast / testerid 0xfc / destaddr 0x10 / addrtype phys`
→ ECM = `0x10`, tester = `0xFC`. Example StartCommunication after fast init: `81 10 FC 81 0E` (checksum = sum & 0xFF).

**Still unknown (to discover on the car):**
- Parameter local-IDs and RAM addresses for this ROM (RPM, coolant, MAF V, timing, knock, IPW…). Sources: RomRaider logger definitions, NisScan register lists, and our own sniffing of NDS II / UpRev traffic.

## OBD connector wiring quirk (check yours!)
UpRev's "2003 G35 CAN re-wire" fix for early cars is: **move the pin in OBD connector slot 3 to slot 14.** On some early Nissans a bus line sits on a non-standard pin. This is what Australian forum posts mean by "a Brisbane shop runs another wire to the OBD2 port". Before buying anything else, look at which pins are populated in your connector (photo it). If pin 3 is populated and 14 isn't, that's the same situation. The fix is reversible.

## Market / ROM
QLD car = Australian-delivered (ADM) or a JDM Fairlady Z import. Check the build/compliance plate. ROM IDs in UpRev's table are mostly US. **Your ROM ID (read with `0x1A 0x81`) is the first thing we learn from the car.** It decides which map definitions exist.

## ECU location & CPU
- RHD/ADM: **behind the passenger airbag** (Haltech). The label shows 23710-xxxxx and an MEC number.
- Likely **SH7055, 512 KiB ROM** (the CD405 350Z ROM is 512 KiB). Confirm when dumping.
- Live-data parameter ID candidates: see `06-research-findings.md` §4.

## Other modules
- Per NisScan: on this generation the ECM uses Consult-II on pin 7; some other modules (BCM, ABS, SRS, TCM) may use Consult-I style Tx/Rx on pins 12/13.
- Internal CAN (500 kbps, 11-bit IDs) carries broadcast data (wheel speed, RPM, coolant, etc.). Sniffing it requires tapping CAN-H/CAN-L at the combo meter or ECM harness if the OBD port doesn't expose pins 6/14. No reliable public ID list for the 2003 exists, so this is a reverse-engineering task.

## Sources
- [NisScan — which NDS software / vehicle list / adaptors](https://nisscan.com/which-nds-software/)
- [RomRaider source (NCS ISO14230 protocol)](https://github.com/RomRaider/RomRaider)
- [nisprog USING.txt (Nissan K-line settings)](https://github.com/fenugrec/nisprog/blob/master/USING.txt)
- [UpRev 2003 G35 CAN re-wire (PDF)](https://uprev.com/wp-content/uploads/2023/05/2003-G35-CAN-re-wire.pdf)
- [UpRev support / supported vehicles & ROMs](https://www.uprev.com/support/)
- [UpRev Nissan Tuning Guide (PDF)](https://uprev.com/wp-content/uploads/2023/05/UpRev-Nissan-Tuning-Guide-Rev2.pdf)
- [pinouts.ru Nissan OBD2 table](https://pinouts.ru/CarElectronics/nissan_obd2_diag.shtml)
- [my350z: OBDII protocol thread](https://my350z.com/forum/engine-and-drivetrain/514694-obdii-protocol.html)
- [my350z: 2003 Z UpRev (no factory wideband)](https://my350z.com/forum/tuning/580545-2003-z-uprev.html)
- [m0agx: reading OBD2 over K-line without ELM327](https://matt2005.github.io/2019-12-28-repost-reading-obd2-data-without-elm327-part2)
- [ECUMaster forum: Nissan CAN 500k / 11-bit](https://www.ecumaster.com/forum/viewtopic.php?p=22892)
