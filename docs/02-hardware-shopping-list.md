# Hardware shopping list (QLD, DIY tune, custom screens)

Prices are **rough AUD estimates** (late 2026). Check local suppliers: Core Electronics, Little Bird, Jaycar, eBay AU.

## Tier 1 — Connect, log, and tune the ECU (~$150–300)

| Item | Why | ~AUD |
|---|---|---|
| **FTDI-based "KKL VAG-COM 409.1" USB cable** (dumb K-line) | **The core tool.** nisprog/npkern (open-source Nissan ROM dump + reflash), NisScan NDS II, and our own Python logger all work through it. Get a genuine FTDI FT232 chip, not CH340. Buy 2 (a spare in the glovebox). | $25–40 ea |
| **Battery charger / power supply, 10 A+** (CTEK MXS 10 or similar) | Mandatory for ROM dumps and flashing. nisprog: *"If the battery dies at the wrong moment, you'll have to unbrick the ECU!"* | $150–200 |
| **NisScan NDS II (Windows)** | NisScan is an Australian company. It's a known-good Consult-II tool: DTCs, live data, idle air volume learn, self-learn reset. We use it as the reference to check our own code against. | ~$48 |
| **Spare 350Z ECU** (wrecker/eBay, same part-number prefix as yours) | A bench "practice" ECU. Dump and flash this one first, so mistakes never strand the car. | $80–200 |
| Bench harness / 12 V bench supply + OBD pigtail | To power the spare ECU on the desk and talk K-line to it | $40–80 |
| **Factory Service Manual** (2003 Z33) | Pinouts, ECM connector, sensor specs | free (NICOclub) |

Optional: **Tactrix OpenPort 2.0** (J2534, ~$250 genuine). Needed **only** for RomRaider's built-in Nissan K-line logger (confirmed in the RomRaider 1.1.0 release notes; its ELM327 support is CAN-only). Our own logger, NisScan and nisprog all use the KKL cable, so this can wait.

Optional: **OBDLink MX+** (~$180). It works with iPhone via Bluetooth and is nice for quick checks with off-the-shelf apps. It **can't** flash, and our hub will own the K-line connection anyway, so it's a convenience, not a requirement.

## Tier 2 — Tuning sensors (~$450–900)

| Item | Why | ~AUD |
|---|---|---|
| **Wideband O2: AEM X-Series UEGO (with CAN + 0–5 V out)** or Innovate MTX-L Plus (serial out) | No factory wideband on this car, and you can't tune fuel without one. The AEM's CAN output plugs straight into our hub. | $350–450 |
| Wideband bung welded (pre-cat, or in the Y-pipe) | Any exhaust shop | $50–100 |
| **Oil pressure sender** (0–150 psi, 0.5–4.5 V) + **oil temp sender** (NTC) + **oil filter sandwich plate** (M20×1.5 stud, 1/8" NPT ports; factory pressure switch is 1/8 BSPT) | Reliability monitoring at 243k km. The factory oil pressure gauge is a crude sender. | $150–250 |
| Knock monitoring (optional): Plex Knock Monitor or det-cans | An independent check on detonation while learning to tune | $100–800 |
| **Dyno time** (Brisbane/Gold Coast, 2–3 hrs) | DIY doesn't mean tuning WOT on public roads. Hire load-bearing dyno time once part-throttle logs are clean. | $150–250/hr |

## Tier 3 — The custom dash (~$400–800)

See `05-dash-architecture.md` for how these fit together.

| Item | Why | ~AUD |
|---|---|---|
| **Raspberry Pi 5 (4–8 GB)** + case with fan | The "hub": owns the K-line connection, reads the wideband and sensors, logs everything, and feeds every screen | $120–170 |
| **Automotive Pi power supply with ignition sense + safe shutdown** | Survives cranking dips. Shuts the Pi down cleanly at key-off so the SD card doesn't get corrupted. | $60–120 |
| **CAN HAT** (Waveshare 2-CH CAN HAT or PiCAN) | Reads the AEM wideband over CAN and runs a private gauge bus. Can later sniff the car's internal bus. | $40–90 |
| ADC board (ADS1115) | Reads the oil pressure/temp senders | $15 |
| **7" screen in the centre cubby** (DSI or HDMI, IPS, touch, ≤ ~182 × 117 × 35 mm module) | The main display. On a 2003 the cubby pocket unscrews and a 7" fits without cutting. Software-drawn gauges mean any layout and multiple pages. **Measure your cubby before buying.** Choose a panel ≤ ~182 × 117 mm (known to fit a 2003). ⚠️ The Raspberry Pi Touch Display 2 is 189.5 × 120 mm, so it probably needs trimming. A Waveshare-style 7" 1024×600 IPS panel or a bare 6.5"–7" panel with a printed bezel is safer. | $90–180 |
| Mounting: 3D-printed bezel/bracket to replace the cubby pocket | A clean factory look. We can design it in OpenSCAD once you send measurements. | $10–60 |
| *(Optional, later)* 2× round ESP32-S3 2.1" gauges | For a second location (A-pillar/steering-column pod). Only two 52 mm-class gauges fit the cubby side by side, so the 7" is the better use of that space. | $55–75 ea |
| 10 Hz GPS (USB) | Lap timing, plus a road-based "virtual dyno" from speed and acceleration | $30–50 |
| Industrial microSD or a USB SSD | Survives heat and power cuts better | $30–60 |

**iPhone:** no hardware needed. The Pi serves a web dashboard over its own Wi-Fi. A native Swift app (built on your Mac) can come later.

## Buy order
1. **Now:** 2× KKL cable + battery charger + NDS II. Read the ROM ID, scan codes, log the stock car with NDS II. Photograph the OBD connector pins.
2. **Then:** spare ECU + bench setup → practise dump/flash with nisprog.
3. **Then:** wideband + oil sensors → real baseline logs.
4. **In parallel:** measure the cubby → Pi hub + 7" screen → first working dash page.
