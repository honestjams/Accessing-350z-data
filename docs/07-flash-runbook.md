# Runbook: dump & flash a 2003 350Z ECU (nisprog / npkern)

Adapted from the RomRaider community tutorial "how to flash your Nissan ECU with opensource tools" (2018, written for 2002–2010 Nissans and originally 350Z-specific). The owner pasted it on 2026-11-16. **Our additions are marked ➕.** The tutorial is from 2018, so check that the nisprog version you download matches these commands (newer builds may name devices differently, e.g. `setdev 7055` instead of `setdev 1`).

## 0. Identify the ECU (do this first, no tools needed)
1. Go to **https://nissan.epc-data.com**, enter your VIN / frame number, then open **D: Engine electrical → ENGINE CONTROL MODULE**.
2. The last characters of the OEM part number (23710-**xxxxx**) are your **ECUID**.
3. Find it in `refs/nissan-stock-rom-list.md` and note the **FID IC**.
   - Every known 2003 350Z ROM has FID **705513**, so expect the **SH7055 350 nm** kernel.
   - ➕ We expect an ADM car to be **CD7xx/CD8xx**. If the ECUID isn't in the list, the ECU "has not been defined yet". You can still dump it, but we'll have to build the map definition ourselves.

➕ Keep your VIN out of the repo and out of chat. Just record the ECUID.

## 1. What you need
| Item | Notes |
|---|---|
| Windows laptop | nisprog is a Windows command-line tool |
| **VAG-COM KKL 409.1 cable** | The tutorial uses the CH340 driver (http://www.wch.cn/download/CH341SER_EXE.html). ➕ FTDI-chip cables are generally more reliable for K-line timing. Either should work, and having a spare of each is cheap insurance. |
| nisprog + npkern | RomRaider forum thread `viewtopic.php?f=64&t=13259` |
| **nissandefs.xml** (Nissan ROM definitions for RomRaider) | RomRaider forum thread `viewtopic.php?f=64&t=13325`. ➕ Please fetch this one so we can check whether CD7xx is defined. |
| RomRaider + **32-bit Java** (x86) | https://www.romraider.com/index.php/RomRaider/Download. ➕ v1.1.0 (Nov 2025) checked: the installer **bundles no Nissan ROM or logger definitions**. Its release notes say Nissan **K-line logging needs a J2534 cable** (Tactrix OpenPort 2.0); ELM327 support is CAN-only. The editor works without any cable. |
| ➕ Battery charger/supply on the car | Mandatory |
| ➕ Spare ECU on a bench harness | Practise every step here first |

## 2. Prepare nisprog
1. Copy the `nisprog` folder to `C:\nisprog\`.
2. Plug in the cable. In **Devices and Printers** (or Device Manager), note the COM port, e.g. `USB-Serial COM3`.
3. Edit `C:\nisprog\nisprog.ini`. Under `# !!! Set COM port number here`, change the port number.
4. Add these lines after the existing ones (for FID 705513):
   ```
   setdev 1
   runkernel C:\nisprog\npk_SH7055_35.bin
   ```
   Kernel rule from the tutorial:
   - 7058 → `setdev 2` + `npk_SH7058.bin`
   - 7055 with last two digits **≤ 13** → `npk_SH7055_35.bin`
   - 7055 with last two digits **> 13** → `npk_SH7055_18.bin`

   Ours is 705513, so `_35` is correct. This matches nissutils (`NPK_7055_35`).

## 3. Dump the stock ROM
1. ➕ Battery charger connected. Laptop on mains power, sleep disabled.
2. Ignition **ON**, engine **off**. Turn off all accessories (dome light, fan, head unit).
3. Connect the cable, then run `C:\nisprog\nisprog.exe`.
4. **Sanity check:** when the kernel is running, **the radiator fans run.** If they don't, stop, close nisprog and fix communication first. Read the log.
5. `dumpmem my_rom.bin 0 524288` (512 KiB for SH7055).
6. **Not done yet:** `stopkernel` (fans stop) → `npdisc` → `quit` → ignition off → unplug.
7. ➕ Repeat the dump **three times** (`stock_1.bin`, `stock_2.bin`, `stock_3.bin`). All three SHA-256 hashes must match. Commit them to `roms/stock/` with the ECUID and date.

## 4. Edit
1. RomRaider: run `run.bat` → Definitions Manager → **Add** `nissandefs.xml`.
2. **File → Open Image** → your dump. If your ECUID is defined, the maps appear in the left panel.
3. Make changes **based on logs and dyno data only.**
   - The tutorial recommends **Nissan DataScan II** (paid) for logging.
   - The UpRev Nissan Tuning Guide explains what each table does.
4. Save as a new file name. ➕ Then:
   - verify the std + alt checksums with our checker (or nissutils `nisrom`);
   - `diff` against stock, so only the intended bytes changed;
   - commit with a note on what changed and why.

## 5. Flash
1. Repeat steps 3.1–3.4 (kernel running, fans on).
2. `flrom YOUR_ROM_NAME.bin`. ➕ Choose **practice mode (`p`) first**, then run it again for real, flashing only the changed blocks.
3. **If you suspect flash errors, do NOT power off.** You can still re-flash the original ROM or fix the communication problem. Read `USING.txt` and ask on the forum.
4. `stopkernel` → `npdisc` → `quit` → unplug → ignition off.
5. Start the car, watch idle, then log a gentle drive before anything spirited.

➕ Do everything in this runbook on the **bench ECU** first, including a deliberately interrupted flash and its recovery, before touching the car.
