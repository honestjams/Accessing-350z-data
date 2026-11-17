# Improvement plan — reliable power, DIY tune (manual, 243,000 km)

Goal: **good, reliable power** with a tune we build ourselves from our own logs.

Rough stock numbers: ~206 kW at the crank (ADM manual), roughly **175–185 rwkW** (≈240–250 whp) on a dyno. Your existing exhaust probably adds a little.

## Stage 0 — Health baseline (non-negotiable at 243k km)
A tune exposes weaknesses. A tired engine on a good tune is still a tired engine. Log every result in `data/health/`.

| Check | Why it matters for tuning |
|---|---|
| **Compression + leak-down test** | Decides whether forced induction is ever on the table |
| **Oil consumption** over 1,500 km | Early VQ35DEs burn oil. More than ~1 L per 1,500 km means fix it before adding power. |
| **Smoke test for vacuum/intake leaks** | This is a MAF car: any unmetered air makes the fuel tuning lie |
| **Injector flow test + clean** (or a matched set) | At 243k km, uneven injectors = one lean cylinder you can't see on a single wideband |
| **Fuel pressure + pump + filter** | Lean at the top end is how VQs melt pistons |
| **Narrowband O2 sensors, MAF, coil packs, plugs** | Old O2 sensors corrupt fuel trims, and the trims are our main data |
| **Timing chain guides/tensioners** (listen for whine/rattle) | Interference engine |
| **Crank/cam sensors**, PCV, plenum gaskets, thermostat | Known Z33 items |
| Clutch, engine/gearbox mounts | Extra torque finds weak mounts and slipping clutches |

**Factory pass/fail numbers** (from the FSM, see `refs/fsm-ec-2003-350z.md`): warm idle MAF 1.1–1.5 V / 2–6 g/s · base fuel schedule 2.5–3.5 ms · injector pulse 2.0–3.0 ms · timing 13–18° BTDC at idle · A/F alpha 54–155 % at 2,000 rpm · fuel pressure ≈ 350 kPa · idle 650 ± 50 rpm.

**Our first software deliverable helps here directly:** logging fuel trims (LTFT/STFT) per bank at idle, cruise and load shows vacuum leaks, injector or O2 problems before we touch the tune.

## Stage 1 — Naturally aspirated + DIY tune (target ~195–205 rwkW / ~265–275 whp)
Order of work:
1. **Learn the stock tune from logs first.** Map where the ECU runs closed loop vs open loop, how far knock retard pulls timing, and what the AFR does at WOT. The stock DE runs very rich at WOT, so there is safe power in fuel and timing.
2. Mechanical: the car has a **cat-back only**, so the stock headers, Y-pipe and cats are still in place. That's where the remaining exhaust gains are. **Y-pipe + high-flow cats** is the best NA mechanical gain on a VQ; headers come next, and an intake adds a little. **Tip:** have the wideband bung welded into the new Y-pipe/cat pipe in the same exhaust-shop visit. Log the stock car *before* the exhaust work, so we can measure what it actually gained.
3. Tune, in small logged steps:
   - **WOT fuel targets**: lean the rich stock WOT mixture toward a safe ~12.5–12.8:1 AFR
   - **Ignition timing**: add advance gradually on 98 RON, watching knock retard. Back off at the first sign of knock.
   - **MAF scaling**: required after any intake or MAF-housing change
   - **VTC (intake cam timing)**: real midrange gains, best tested on a dyno
   - **Rev limit**: modest only (~6,900–7,000) on a 243k km DE with stock valve springs
   - Throttle map, speed limiter, cooling fan temperatures
4. **Dyno sessions** for the WOT work. Cruise/part-throttle tuning can be done from road logs.

## Stage 2 — Forced induction (only if Stage 0 is clean): ~250–280 rwkW
- A low-boost (~6 psi) centrifugal supercharger is the most "reliable power" option on a high-km DE
- Stock internals are generally considered safe to ~400 whp (~300 rwkW). On a 243k km engine, aim well under that.
- **Honest warning:** DIY-tuning a MAF-based stock ECU for boost is a big jump. It needs larger injectors (K-fuel constant and latency changes), MAF rescaling or a larger housing, and boost-aware timing. Do NA first. By then we'll know the ROM well enough to judge it.

## Alongside power: reliability monitors we build
- Oil temp / oil pressure alarms (screen flash + chime), especially oil pressure vs RPM
- Knock-retard counter per drive, with a warning when it trends up
- Fuel trim drift tracking (early warning of leaks, injector or O2 aging)
- Coolant temp and IAT on track/hot QLD days
- AFR vs target at WOT, flagged if it's ever leaner than the limit
