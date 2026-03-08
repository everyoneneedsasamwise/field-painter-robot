# FieldBot — Autonomous Line Painting Robot 🤖🎨

An open-source, GPS-free autonomous robot that paints lines on sports fields and custom designs. Built for under $500.

## Why?

Commercial field painters cost $5,000–$30,000+. Manual line painting takes hours and requires skill. FieldBot bridges the gap — affordable, automated, and accurate enough for regulation fields.

## Features (Planned)

- **Full soccer field coverage** (~100m x 64m)
- **UWB positioning** — cm-level accuracy with 4 corner stakes (no GPS subscription)
- **Custom designs** — load SVGs or field templates
- **Obstacle avoidance** — ultrasonic sensors + bumper
- **Phone control** — start/stop/monitor from your phone
- **Under $500** total build cost

## Architecture

```
┌─────────────┐     ┌──────────────┐     ┌─────────────┐
│  Phone App  │────▶│ Raspberry Pi │────▶│   Motors    │
│  (Design +  │     │  (Brain)     │     │  (L298N)    │
│   Control)  │     │              │     └─────────────┘
└─────────────┘     │  - Path Plan │     ┌─────────────┐
                    │  - Nav Fusion│────▶│ Paint Valve │
┌─────────────┐     │  - PID Ctrl  │     │ (Solenoid)  │
│ UWB Anchors │────▶│              │     └─────────────┘
│ (4 corners) │     │              │
└─────────────┘     └──────┬───────┘
                           │
              ┌────────────┼────────────┐
              ▼            ▼            ▼
        ┌──────────┐ ┌──────────┐ ┌──────────┐
        │ Encoders │ │   IMU    │ │Ultrasonic│
        │ (wheels) │ │(MPU-6050)│ │ (HC-SR04)│
        └──────────┘ └──────────┘ └──────────┘
```

## Navigation Strategy

**Primary:** RTK GPS via Quectel LC29H modules — $60 rover + $60 base station = **1-2cm accuracy**. Or use free NTRIP/CORS correction services and skip the base station entirely.

**Secondary:** Wheel encoders + IMU for dead reckoning between GPS fixes. Sensor fusion via Extended Kalman Filter.

**Inspired by:** [OpenMower](https://github.com/ClemensElflein/OpenMower), [ArduMower](https://wiki.ardumower.de), drone flight planners (Mission Planner, QGroundControl).

**Flight Planner UI:** Satellite view of your field → overlay design template → tap corners → robot paints it. Like planning a drone mission, but for paint.

## Build Phases

| Phase | Description | Status |
|-------|-------------|--------|
| 1 | Rolling chassis (motors, wheels, frame, Pi) | 🔲 Not started |
| 2 | Navigation (UWB, encoders, IMU, sensor fusion) | 🔲 Not started |
| 3 | Paint system (pump, solenoid, tank) | 🔲 Not started |
| 4 | Software (path planning, templates, phone app) | 🔲 Not started |

## Budget

Target: **Under $500**. See [BOM.md](docs/BOM.md) for full parts list.

## Docs

- [BOM.md](docs/BOM.md) — Bill of Materials with sources and prices
- [NAVIGATION.md](docs/NAVIGATION.md) — RTK GPS architecture, LC29H setup, coordinate system
- [DESIGN.md](docs/DESIGN.md) — Detailed design decisions and tradeoffs
- [WIRING.md](docs/WIRING.md) — Wiring diagrams and pinouts
- [SOFTWARE.md](docs/SOFTWARE.md) — Software architecture and setup
- [APP.md](docs/APP.md) — Mobile app design (flight planner UI)

## License

MIT — build one, sell one, modify it, go wild.

## Contributing

Issues and PRs welcome. This is a learn-in-public project.
