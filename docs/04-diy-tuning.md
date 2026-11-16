# DIY tuning workflow (no UpRev)

## The open-source toolchain
| Job | Tool | Notes |
|---|---|---|
| Dump / reflash the ROM | **nisprog + npkern** (fenugrec, GPL) | Open-source Nissan reflash over K-line. Supports SH7051/7055/7058 Nissan ECUs from ~2002 on. Works with a "dumb" KKL cable. Windows usage is documented. **Experimental — the author offers no guarantees.** |
| Live logging | **Our own Python logger** (KWP2000 `0xAC` + `0x21`), RomRaider for cross-checks | See `01-vehicle-and-protocols.md` |
| Existing start point | **CD405 EcuFlash definition** (NisScan forum) + **nissutils** (`nisrom`, Ghidra helpers for SH7055, 92 known keysets) | See `06-research-findings.md` |
| Finding maps in the ROM | Community definitions for the ROM ID (EcuFlash/RomRaider XML) if they exist; otherwise **nisrom** (fenugrec) + **Ghidra** (supports SuperH) | This is the real work if your ROM (ADM/JDM) has no public definition |
| Editing | Our own table editor (Python), or EcuFlash/RomRaider with a definition | |
| Checksums | **nissutils `nisckfix1/2`** fix the standard Nissan sum+xor checksum; `nisrom` re-checks | Must be correct before every flash | nisprog's docs say *"make ABSOLUTELY sure the checksum of the new ROM file is ok before reflashing"*. We'll write and test our own checksum verifier against the stock ROM. |

nisprog connection settings for these ECUs: `iso14230`, fast init, tester `0xFC`, ECU `0x10`, then `setdev 7055` or `7058` (identify your chip by opening the spare ECU).

## Safety rules
1. **Back up the stock ROM 3 times** (car ECU), compare the hashes, and store copies in this repo (`roms/stock/`) and off-site.
2. **Practise on the spare ECU first.** Do every new operation (dump, practice flash, real flash, recovery) on the bench before touching the car.
3. Battery charger connected, ignition ON, engine OFF, laptop on mains power, nothing else on the K-line (unplug the Pi hub).
4. Use nisprog's **practice mode** (`p`) before real writes. Flash only the changed blocks. The flash chips are rated for **~100 write cycles**, so batch your changes.
5. **If a flash goes wrong, don't power the ECU off.** Re-dump, verify, retry with the stock image.
6. One change per flash, a log before and after, and every ROM committed with a note on what changed and why.
7. WOT changes happen on a dyno, not the road.

## The loop
```
log stock  →  analyse (AFR vs target, knock, trims, timing)  →  change ONE thing
   ↑                                                                   ↓
commit ROM + logs  ←  verify with logs  ←  flash (bench-proven)  ←  checksum fix
```

## Milestones
1. Read ECU ID → know the ROM → search for an existing definition
2. Bench: dump the spare ECU → identify the CPU (7055/7058) → practice flash → recover from a deliberately aborted flash
3. Car: dump the stock ROM ×3
4. Map definition: locate fuel target, MAF, K-fuel, timing, VTC, rev/speed limit tables
5. First safe edit: speed limiter or rev limit (low risk, easy to verify) → prove the whole flash chain end to end
6. Fuel and timing work with the wideband + dyno
