# Custom dash & gauges architecture

**Key fact:** off-the-shelf digital dashes (Haltech, AEM, etc.) that read "OBD" expect a CAN-OBD car. **Your ECU speaks K-line**, so they can't read it directly. That's why we build a hub: it translates the K-line data and feeds whatever screens you buy.

```
                         ┌──────────────── Raspberry Pi 5 "hub" ────────────────┐
 ECU ──K-line── KKL USB ─┤ logger: KWP2000 stream (RPM, load, MAF, timing,      │
                         │         knock, IPW, trims, coolant, IAT, TPS, speed) │
 AEM wideband ──CAN──────┤ CAN HAT                                              │
 Oil P / Oil T ─analog───┤ ADS1115                                              │
 GPS ──USB───────────────┤                                                      │
                         │ → merge + timestamp → SQLite/CSV logs (data/)        │
                         │ → alarms (oil P low vs RPM, AFR lean, knock, temps)  │
                         │ → publish ~20 Hz                                     │
                         └──────┬─────────────────┬────────────────┬────────────┘
                                  │ DSI/HDMI              │ Wi-Fi AP (WebSocket)
                                  │                       │
                        7" touch screen in the     iPhone web dashboard
                        centre cubby (main dash)   (+ optional ESP32 round
                                                    gauges elsewhere, later)
```

## Design decisions
- **The hub owns the ECU connection.** K-line allows one tester at a time, so phones and screens get data from the hub, not from the car. That also means one log file holds everything, time-aligned.
- **Wi-Fi first, CAN later** for the gauges. With Wi-Fi the screens only need power wires, so it's fast to prototype. A private wired CAN bus (ESP32-S3 TWAI + SN65HVD230 transceiver) is the robust upgrade once the layouts are settled. **Never put our gauges on the car's own CAN bus.**
- **Graceful degradation:** if the hub dies, the car drives normally. The ESP32 gauges show "NO DATA" rather than a frozen value.
- **iPhone:** a web dashboard served by the Pi (add to Home Screen) means no App Store and no Mac build step. A native SwiftUI app is possible later.
- **QLD heat:** dash-top temperatures in summer can exceed the ratings of consumer electronics. Mount the Pi under the dash or in the centre console, not on top. Use high-brightness IPS panels with auto-dimming (light sensor or ECU headlight signal).
- **Power:** use an ignition-switched supply with hold-up, plus a clean shutdown at key-off. Fuse it at the source.

## Refresh-rate budget
K-line at 10,400 baud with ~5–6 parameters per request → realistically **~10–20 Hz** for a small set and lower for a big one. Plan two tiers:
- **Fast set** (RPM, TPS, timing, knock, AFR from CAN) for gauges and shift light
- **Slow set** (temps, trims, voltages) polled round-robin

Wideband and analog sensors don't go over K-line, so they can update at 50–100 Hz.

## The centre cubby install (2003)
- On a 2003 the cubby pocket is screwed in. Remove the centre console trim, unscrew the pocket, and replace it with a **3D-printed bezel** holding the 7" panel. Owners report a 7" fits with no cutting, and an ~182 × 117 × 35 mm module was confirmed to fit an '03.
- **Before buying a panel, measure:** opening width × height, usable depth behind the opening, and where the harness and HVAC ducting sit behind it.
- The Pi probably won't fit behind the screen. Mount it in the centre console or passenger-side kick area and run a DSI ribbon (≤ ~20 cm is safest) or a short HDMI + USB-touch cable.
- The recessed cubby shades the screen from the QLD sun better than a dash-top mount. Add an anti-glare screen protector.

## Screen pages (swipe or tap to change)
```
┌─────────────────────── STREET ───────────────────────┐   ┌─────────────────────── TUNE ─────────────────────────┐
│   ╭─────╮                                             │   │ RPM 4820  LOAD 1.82  MAF 3.41V  IPW 9.8ms             │
│  ╱  RPM  ╲     OIL P  62 psi   OIL T  104°C            │   │ AFR 12.6 (tgt 12.6)  TIMING 28.5°  KNOCK 0.0°         │
│ │  4820   │    COOL   88°C     AFR    14.7             │   │ STFT B1 +1.6 B2 -0.8   LTFT B1 +3.1 B2 +2.4           │
│  ╲  ▮▮▮▮ ╱     VOLTS  14.1     IAT    34°C             │   │ ┌ AFR vs target ─────────────────────────────────┐   │
│   ╰─────╯      [shift light bar across top]            │   │ │~~~~~~~~~~~~~~ live graph ~~~~~~~~~~~~~~~~~~~~~~│   │
│  ⚠ alerts banner: OIL PRESSURE LOW / KNOCK / LEAN      │   │ └────────────────────────────────────────────────┘   │
└───────────────────────────────────────────────────────┘   │ [● REC]  [MARK]  pull detected: 3rd 2500→6600       │
                                                            └───────────────────────────────────────────────────────┘
┌──────────────────────── TRACK ────────────────────────┐   ┌────────────────────── HEALTH ────────────────────────┐
│ LAP 1:32.48  Δ -0.41   best 1:31.97                    │   │ Knock events this drive: 0   Max oil T: 112°C        │
│ OIL T / COOL / OIL P big tiles, colour-coded           │   │ Trim drift (30 days) ▁▂▂▃  DTCs: none                 │
│ Peak-hold G, speed, gear                               │   │ Oil P @ idle hot: 18 psi (baseline 20)                │
└───────────────────────────────────────────────────────┘   └───────────────────────────────────────────────────────┘
```
