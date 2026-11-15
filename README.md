# Accessing 350Z Data

Read data from a 2003 Nissan 350Z (VQ35DE), build custom gauges and apps, and build our own tune from our own logs for good, reliable power.

## Docs
1. [How the car talks](docs/01-vehicle-and-protocols.md): K-line / KWP2000, addresses, protocol bytes, the OBD pin quirk
2. [Hardware shopping list](docs/02-hardware-shopping-list.md): AUD, in buy order
3. [Improvement plan](docs/03-improvement-plan.md): health baseline → NA + DIY tune → (maybe) boost
4. [DIY tuning workflow](docs/04-diy-tuning.md): nisprog/npkern, safety rules, milestones
5. [Dash architecture](docs/05-dash-architecture.md): Pi hub → round ESP32 gauges, main screen, iPhone

## Roadmap
- [ ] **Phase 0**: KKL cable + charger. Read ECU ROM ID, scan codes, photo the OBD pins, health checks
- [ ] **Phase 1**: Python K-line logger (fast init → ECU ID → stream → CSV), checked against NisScan
- [ ] **Phase 2**: Bench ECU: dump / practice-flash / recover with nisprog. Dump the car's stock ROM ×3
- [ ] **Phase 3**: Pi hub + wideband + oil sensors + first round gauge + iPhone web dash
- [ ] **Phase 4**: Map definition for our ROM, checksum tool, first safe edit (limiter)
- [ ] **Phase 5**: Fuel/timing/VTC tuning with logs + dyno

## Car profile
| Field | Value |
|---|---|
| Model | 2003 Nissan 350Z (Z33), VQ35DE |
| Market | QLD-registered: ADM or JDM import? (check the build plate) |
| Transmission | Manual |
| Odometer | 243,000 km |
| Existing mods | Aftermarket exhaust (details?), suspension, brakes |
| ECU ROM ID | (read via `0x1A 0x81`) |
| Owner devices | iPhone, Mac, Windows laptop |
