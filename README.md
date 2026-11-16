# Accessing 350Z Data

Read data from a 2003 Nissan 350Z (VQ35DE), build custom gauges and apps, and build our own tune from our own logs for good, reliable power.

## Docs
1. [How the car talks](docs/01-vehicle-and-protocols.md): K-line / KWP2000, addresses, protocol bytes, the OBD pin quirk
2. [Hardware shopping list](docs/02-hardware-shopping-list.md): AUD, in buy order
3. [Improvement plan](docs/03-improvement-plan.md): health baseline → NA + DIY tune → (maybe) boost
4. [DIY tuning workflow](docs/04-diy-tuning.md): nisprog/npkern, safety rules, milestones
5. [Dash architecture](docs/05-dash-architecture.md): Pi hub → 7" centre-cubby screen + iPhone, screen pages
6. [Research findings](docs/06-research-findings.md): ECU/CPU, CD405 map definition, nissutils, param IDs, cubby fit
7. [Flash runbook](docs/07-flash-runbook.md): ECU ID lookup → dump → edit → flash (nisprog/npkern, kernel `npk_SH7055_35`)

Reference data: [refs/nissan-stock-rom-list.md](refs/nissan-stock-rom-list.md)

## Roadmap
- [ ] **Phase 0**: KKL cable + charger. Read ECU ROM ID, scan codes, photo the OBD pins, health checks
- [ ] **Phase 1**: Python K-line logger (fast init → ECU ID → stream → CSV), checked against NisScan
- [ ] **Phase 2**: Bench ECU: dump / practice-flash / recover with nisprog. Dump the car's stock ROM ×3
- [ ] **Phase 3**: Pi hub + wideband + oil sensors + 7" cubby screen + iPhone web dash
- [ ] **Phase 4**: Map definition for our ROM, checksum tool, first safe edit (limiter)
- [ ] **Phase 5**: Fuel/timing/VTC tuning with logs + dyno

## Car profile
| Field | Value |
|---|---|
| Model | 2003 Nissan 350Z (Z33), VQ35DE |
| Market | Australian-delivered (ADM), QLD-registered |
| Transmission | Manual |
| Odometer | 243,000 km |
| Existing mods | Cat-back exhaust (stock headers / Y-pipe / cats), suspension, brakes |
| Screen location | Centre-stack storage cubby (7" screen) |
| ECU ROM ID | (read via `0x1A 0x81`) |
| Owner devices | iPhone, Mac, Windows laptop |
