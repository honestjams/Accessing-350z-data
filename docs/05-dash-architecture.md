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
                     Wi-Fi AP (UDP/WebSocket)     │ HDMI/DSI       │ (later) private CAN
                    ┌───────────┴───────┐         │                │
               ESP32-S3 round      iPhone /       Main 7–8.8"      Wired gauges
               gauges (×3)         iPad web       screen           (more robust)
                                   dashboard
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
