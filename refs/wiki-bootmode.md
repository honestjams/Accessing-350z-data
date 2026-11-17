# SH boot mode: last-resort ECU recovery

> Adapted from the Nissan ECU RE wiki page **[Bootmode](https://nissanecu.miraheze.org/wiki/Bootmode)** (last edited 12 June 2026), licensed **CC BY-SA 4.0**. This file is a derivative and is shared under the same license. Pasted by the owner on 2026-11-17; summarised and annotated (➕) for our 350Z.

## What it is
The SH705x MCU's built-in **boot mode** can reflash the chip **even if the main ROM ("user MAT") is corrupted**, e.g. after a failed K-line flash. It needs **physical access to the PCB**: open the ECU case, remove the conformal coating, and connect to test points.

➕ For us this is the **safety net behind nisprog**: a bricked ECU is recoverable, but only with bench work. So the order of operations matters even more (backups first, bench practice first).

## Wiring
- Two sets of 6 test points on the PCB back (often mirrored on the component side):
  - **Power:** VBat, GND, +5 V
  - **Signals:** **WDT** (usually PL5), **MD1**, **SCI1_TX**, **SCI1_RX**, **nRES**
- Test-point locations vary by [hardware version](https://nissanecu.miraheze.org/wiki/Ecu_hw). ➕ We need our board's layout (Ecu_hw page, or trace it on the spare ECU).
- **nRES** is pulled up on the PCB and other chips (e.g. the 789101 supervisor) may pull it low, so **never hard-wire nRES to Vcc.**
- The **WDT signal must be supplied artificially**, or the supervisor IC keeps resetting the MCU. Measure the real PL5 pulse train first and make sure your generator reproduces it.
- SCI1 may be shared with the K-line transceiver (on 6Z68A, RX goes to the L9762 through 3.3 kΩ, so two transmitters on the line is OK).

## Sequence
1. Hold **nRES → GND** (through ≤ 22 Ω).
2. Hold **MD1 → GND** (through ≤ 22 Ω).
3. Power everything up and start the **WDT pulse train**.
4. Release nRES.
5. Verify: good WDT pulses, and nRES stays at 5 V for at least 2–3 s.

⚠️ **Entering programming mode automatically ERASES the whole ROM**, whatever tool you use (FDT, EcuFlash…). **Never do it without a verified backup of the original ROM.**

### Safe "is it alive?" test (doesn't enter programming mode)
Serial at **19200 bps 8N1** (TTL levels, on SCI1):
1. Send `0x00` repeatedly (≤ 30×) until `0x00` comes back.
2. Send `0x55` (completes bit-rate adjustment).
3. Expect **`0xE6`** ("boot OK").
4. Send `0x20` (Supported Device Inquiry). Example reply: `30 0F 01 0D 30 36 30 31 48 44 36 34 46 37 30 35 38 DC`, which contains ASCII "0601" + "HD64F7058". ➕ Ours should report an **SH7055** part.
5. Beyond this, follow the datasheet's "Serial communication interface specification for Boot Mode" exactly. Careless commands can erase the chip.

## Practical rules
- Verify continuity from each test point to the MD1, PL5, SCI1 and nRES pins.
- Remove the conformal coating before soldering.
- **Never** solder directly to MCU pins or try to lift them. Use the test points.
- **No pulling or shearing force on pads.** They will lift or tear. ➕ Use flexible wire and strain relief (hot glue or Kapton).

## Links
- [Nissan-specific unbricking (RomRaider)](http://www.romraider.com/forum/viewtopic.php?f=7&t=12821)
- RomRaider threads [7136](http://www.romraider.com/forum/viewtopic.php?f=7&t=7136), [7384](http://www.romraider.com/forum/viewtopic.php?t=7384); [NASIOC thread](http://forums.nasioc.com/forums/showthread.php?t=2200443) (mostly Subaru; similar but not identical)
- Wiki: [Wdt](https://nissanecu.miraheze.org/wiki/Wdt) (WDT signal details) and [Ecu_hw](https://nissanecu.miraheze.org/wiki/Ecu_hw) (board versions/test points). ➕ **Still needed.**

## ➕ What this means for our plan
1. **Read-only ROM dump first** (AC/21, no kernel) + nisprog dump, both verified. Without a backup, boot-mode recovery is impossible because entering it erases the chip.
2. **Rehearse boot mode on the spare ECU** (open it, find the test points, measure WDT, run the "is it alive?" test) **before** the first real flash of the car ECU. Then a brick is a bad afternoon, not a dead car.
3. **Recovery kit** (~AUD $60–120): 5 V USB-TTL serial adapter (FTDI) · small MCU (Arduino Nano / ESP32) to generate the WDT pulse train · cheap 8-channel logic analyser (to capture the real PL5 signal) · 22 Ω resistors · fine wire, flux, fine-tip iron · Kapton tape.
