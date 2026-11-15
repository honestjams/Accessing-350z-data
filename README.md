# Accessing 350Z Data

Read data from a 2003 Nissan 350Z (VQ35DE), build apps that talk to the car, and tune it for more power — safely.

## Docs
1. [How the car talks](docs/01-vehicle-and-protocols.md) — K-line / KWP2000 / Consult-II, protocol bytes, what's still unknown
2. [Hardware shopping list](docs/02-hardware-shopping-list.md) — what to buy, in what order
3. [Improvement plan](docs/03-improvement-plan.md) — health baseline → bolt-ons + tune → forced induction

## Roadmap
- [ ] **Phase 0** – Buy Tier 1 hardware, scan codes, record ECU ROM ID, baseline health checks
- [ ] **Phase 1** – Python K-line logger (fast init → ECU ID → stream params → CSV), checked against NisScan
- [ ] **Phase 2** – Wideband integration, a live dashboard (phone or Pi), drive/track logs stored in `data/`
- [ ] **Phase 3** – Analysis tools: pull detection, AFR vs target, knock reports, before/after mod comparisons
- [ ] **Phase 4** – Tune (UpRev), optional CAN-bus reverse engineering

## Car profile (fill in)
| Field | Value |
|---|---|
| Transmission | MT / AT ? |
| Trim | ? |
| Mileage | ? |
| ECU ROM ID | (read via `0x1A 0x81`) |
| Existing mods | ? |
| State (emissions) | ? |
