# Improvement plan — 2003 350Z (VQ35DE)

Stock specs: about 287 hp / 274 lb-ft at the crank for the manual (less for the AT), roughly 240–250 whp on a dyno, 6,600 rpm redline.

## Stage 0 — Health baseline (do this first, before any power mods)
More power only makes existing problems worse. Check these, and log the results in this repo:

- [ ] **Scan all modules for DTCs** (ECM, TCM, ABS, BCM)
- [ ] **Oil consumption**: early VQ35DEs (2003–05) are known to burn oil. Track it over ~1,000 mi. More than about 1 qt per 1,000 mi is a red flag for adding boost.
- [ ] **Compression / leak-down test**
- [ ] **Crank & cam position sensors**: there was a 2003 recall over poorly soldered sensor boards. Confirm it was done (a Nissan dealer can check by VIN).
- [ ] **Timing chain**: listen for a whine or rattle (secondary chain guides and tensioners). This is an interference engine, so a chain failure wrecks the valves.
- [ ] Valve cover / plenum gaskets, steering rack leaks
- [ ] MT: grinding into 3rd (synchros), clutch slave cylinder. AT: cooler lines.
- [ ] Spark plugs (100k mi interval), coil packs, MAF cleaning, fresh fluids

## Stage 1 — "Make the stock car better" (biggest return per dollar)
These often matter more than horsepower:
- **Tires** — the single biggest improvement to grip, braking and feel.
- **Brake fluid + performance pads**
- **Alignment** (a mild performance alignment)
- **Oil cooler** if you plan to track it. VQ35DE oil temperatures get high on track. **Our logger can measure this before you spend money.**

## Stage 2 — Naturally aspirated power + tune (~+20–35 whp, realistic)
| Mod | Notes |
|---|---|
| **UpRev tune** | This is the core. It lets you edit fuel targets, MAF scaling, ignition timing, knock thresholds, cam phasing (VTC), throttle mapping, **rev limit** (a modest raise from 6,600), and the **speed limiter**. It also cleans up drivability. |
| Cat-back exhaust | Mostly sound, but worth a few whp once tuned |
| Intake | A few whp at most. Forum dyno results are inconsistent. |
| Y-pipe + high-flow cats | A real gain on the VQ. **Use emissions-legal (CARB EO) cats if your state inspects.** Test pipes are illegal on the road in the US. |
| Headers | DE owners have reached ~253 whp with headers + intake + exhaust + UpRev tune, and ~269 whp after a larger intake and throttle body were added and the car was retuned |

Realistic NA ceiling for a 2003 DE on bolt-ons + tune: **~260–275 whp**.

## Stage 3 — Forced induction (~350–420 whp)
- Options: Vortech V-3 supercharger (~412 hp / 7.5–8.5 psi on a 2003 per the vendor), Stillen (Vortech-based), or single/twin turbo kits.
- **Stock internals are generally considered safe to about 400 whp.** People run more, but how long the engine lasts becomes a gamble.
- Requires: larger injectors, a fuel pump, a wideband, a proper tune (UpRev), and ideally a healthy-oil-consumption engine (see Stage 0). Budget $6–10k all-in.

## Where our software helps
1. **Baseline logger**: record RPM, load, MAF V, timing, knock retard, coolant/oil temperatures, IPW and AFR on the stock car.
2. **Health monitor**: track oil and coolant temperatures on drives or the track, and alert on knock events or fuel trims drifting.
3. **Before/after comparison**: use the same road, the same gear and a pull-logging app to measure what each mod actually did (a poor man's dyno from acceleration + vehicle mass).
4. **Tune-review tooling**: overlay wideband AFR against target and knock against timing, which shows a tuner where to look.

## Sources
- [MotoIQ: Project 350Z ECU tuning with UpRev](https://motoiq.com/project-350z-ecu-tuning-with-church-automotive-and-uprev)
- [my350z: AdminTuning intake + NWP TB dyno (DE, UpRev)](https://my350z.com/forum/intake-exhaust/610139-dyno-results-admintuning-3-5-intake-and-nwp-tb-vs-stillen-intake.html)
- [MotoIQ: Supercharging the VQ35DE with Vortech](https://motoiq.com/supercharging-the-nissan-vq35de-with-vortech)
- [JEGS: Vortech V-3 Si kit for 2003 350Z](https://www.jegs.com/i/vortech-933-4nz218-010l-v-3-sci-trim-nissan-supercharger-kit)
- [8020 Automotive: VQ35DE turbo guide](https://8020automotive.com/ultimate-nissan-350z-turbo-upgrade-guide/)
- [my350z: How much power will a stock VQ35DE hold?](https://my350z.com/forum/engine-and-drivetrain/407783-how-much-power-will-a-stock-vq35de-hold-printthread.html)
- [8020 Automotive: VQ35DE common problems](https://8020automotive.com/the-3-most-common-nissan-vq35de-engine-problems/)
- [au7o.io: 350Z known issues](https://au7o.io/known-issues/nissan-350z)
- [UpRev Nissan Tuning Guide](https://uprev.com/wp-content/uploads/2023/05/UpRev-Nissan-Tuning-Guide-Rev2.pdf)
