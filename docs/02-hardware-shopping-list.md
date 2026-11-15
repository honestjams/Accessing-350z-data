# Hardware shopping list

Prices are approximate (late 2026). UpRev prices are from uprev.com; others are typical street prices — check before buying.

## Tier 1 — Read data & build apps (~$150–250)

| Item | Why | Approx. cost |
|---|---|---|
| **OBDLink MX+** (Bluetooth) *or* **OBDLink SX/EX** (USB) | STN chip with solid ISO 9141 / KWP2000 K-line support and custom headers. NisScan tested and recommends OBDLink. MX+ works with both iPhone and Android, which matters for phone apps. USB is cheaper and simpler for a laptop or Raspberry Pi. | MX+ ~$120 · SX ~$40 · EX ~$60 |
| **Battery maintainer / power supply** (10 A+, e.g. CTEK) | Long logging sessions with the engine off, and **mandatory** while flashing the ECU. A voltage dip mid-flash can brick the ECM. | ~$60–100 |
| **Factory Service Manual (2003 350Z)** | Wiring diagrams, K-line/CAN pinouts, sensor specs. NICOclub hosts free FSM PDFs. | Free |
| **NisScan NDS II** (Windows or Android) | A known-good reference tool for Consult-II: live data, DTC read/clear, idle air volume learn, self-learn reset. Useful for checking our own code against. | ~$48 |

**Skip:** $10 generic "ELM327 v2.1" Bluetooth clones. Many can't do K-line properly, and the ones that can are flaky.

## Tier 2 — Tune the car (~$800–1,000)

| Item | Why | Approx. cost |
|---|---|---|
| **Wideband O2 controller + sensor** (Innovate MTX-L Plus or AEM X-Series) | The 2003 has **no factory wideband**, so you can't tune fuel safely without one. Innovate has a documented serial output (easy to read from Python). AEM X-Series has CAN and 0–5 V outputs. | $200–250 |
| Wideband bung welded into the exhaust | One-time shop job (or put it in an aftermarket Y-pipe/test pipe) | $40–100 |
| **UpRev tuning kit** | The established reflash platform for the VQ35DE. It explicitly supports 2003 MT and AT ROMs. Windows-only software. | Remote Tuning Kit **$575** (a pro tunes you remotely) · Self Tuning Kit **$675** · or pay a local UpRev tuner who has their own cable (license $175 + their labor) |
| Windows laptop | UpRev requires Windows 10+ and 8 GB RAM | — |

**About EcuTek:** Z1 Motorsports sells EcuTek packages for the 350Z with Bluetooth flashing, but I couldn't confirm that 2003 ROMs are supported. For this year, UpRev is the safe choice.

## Tier 3 — Deeper hacking (optional, ~$100–250)

| Item | Why | Approx. cost |
|---|---|---|
| **Tactrix OpenPort 2.0** (J2534) | Runs RomRaider (an open-source logger with Nissan K-line support) and EcuFlash (for reading and inspecting the ROM). It's the "open" path alongside UpRev. Clones are ~$30 but less reliable. | ~$170 genuine |
| **CAN interface** (CANable 2.0 / Raspberry Pi + MCP2515 HAT) + harness tap | Sniffs the internal 500 kbps CAN bus for high-rate broadcast data such as wheel speeds, RPM, and throttle. This is a reverse-engineering task. | $40–80 |
| **Raspberry Pi 5 + 12 V→5 V buck converter** | An always-on in-car logger or dash that uploads drives over Wi-Fi when you get home. | $100–130 |
| GPS module (10 Hz) | For lap/drag timing and syncing logs to location | $30 |

## Recommended buy order
1. OBDLink (MX+ if you want phone apps) + battery maintainer + FSM. **Start logging the stock car now, so we have a baseline.**
2. Wideband + bung. Log AFRs on the stock tune too, since that's useful baseline data.
3. Decide on the bolt-ons (see `03-improvement-plan.md`), then the UpRev kit or a pro tuner.
4. Tactrix / CAN gear once our own logger works and we want more.
