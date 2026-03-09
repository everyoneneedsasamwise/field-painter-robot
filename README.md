# FieldBot — Modular Outdoor Robot Platform 🤖

One robot. Three swappable modules. Paint fields, laser weeds, spray fence lines.

## The Problem

| Task | Current Solution | Cost | Time |
|------|-----------------|------|------|
| Paint a soccer field | Manual push liner or $15K+ Turf Tank | $15,000+ | 2-3 hours |
| Kill lawn weeds | Bend over and spray/pull, or chemicals everywhere | Your back | Endless |
| Spray fence lines | Walk with a pump sprayer in the heat | Your weekend | Monthly |

## The Solution

An open-source autonomous robot platform for **under $530** that handles all three.

```
┌─────────────────────────────────────────────┐
│          FIELDBOT BASE PLATFORM             │
│                                              │
│  🛞 2WD Differential Drive (pneumatic)      │
│  🛰️ RTK GPS (1-2cm accuracy)               │
│  🧠 Raspberry Pi 4                          │
│  🔋 Ryobi 18V Battery                      │
│  📱 Phone App (flight planner style)        │
│  🛡️ Obstacle Avoidance                     │
│                                              │
│  ┌─────────┐  ┌─────────┐  ┌─────────┐     │
│  │🎨 PAINT │  │⚡ LASER │  │🧪 SPRAY │     │
│  │ MODULE  │  │ MODULE  │  │ MODULE  │     │
│  │         │  │         │  │         │     │
│  │ Field   │  │ AI weed │  │ Blanket │     │
│  │ lines   │  │ killing │  │ herbicide│    │
│  │         │  │ (lawn + │  │ (fences, │    │
│  │ Soccer  │  │ garden) │  │ gravel,  │    │
│  │ Football│  │         │  │ rocks)   │    │
│  │ Custom  │  │ No chems│  │          │    │
│  └─────────┘  └─────────┘  └─────────┘     │
│       $383        +$145        +$0*          │
└─────────────────────────────────────────────┘
        * Spray shares paint hardware
```

## Modules

### 🎨 Paint Module — Autonomous Field Line Painter
- Paint regulation soccer, football, lacrosse, or custom designs
- Flight planner UI: overlay templates on satellite view of your field
- Pump + solenoid + adjustable spray nozzle
- Save field layouts with GPS coords — repaint the same lines perfectly every time
- **Add-on cost: Included in base ($383)**

### ⚡ Laser Module — AI-Powered Weed Killer
- 5.5W 445nm blue laser destroys weed growing points
- Pi Camera + Google Coral Edge TPU for real-time AI detection
- **Lawn mode:** Identifies broadleaf weeds (goatheads, dandelions, crabgrass) in grass
- **Garden mode:** Kills everything outside your mapped crop positions
- No chemicals, no soil disturbance, surgical precision
- ~20-40 weeds per minute
- **Add-on cost: ~$145**

### 🧪 Spray Module — Fence Line & Gravel Blanket Sprayer
- Draw spray zones on satellite view (fence lines, gravel, rock beds)
- Robot drives the path and blanket sprays everything
- Use vinegar, organic herbicide, or commercial products
- No AI needed — just kill everything in the zone
- Shares pump/solenoid/tank with paint module (just swap liquid + nozzle)
- **Add-on cost: $0** (uses paint hardware)

## Navigation

**Primary:** RTK GPS via Quectel LC29H modules — $60 rover + $60 base station = **1-2cm accuracy**. Or use free NTRIP/CORS correction services and skip the base station entirely.

**Secondary:** Wheel encoders + IMU for dead reckoning between GPS fixes. Sensor fusion via Extended Kalman Filter.

**Inspired by:** [OpenMower](https://github.com/ClemensElflein/OpenMower), [ArduMower](https://wiki.ardumower.de), drone flight planners (Mission Planner, QGroundControl).

**Flight Planner UI:** Satellite view of your field → overlay design template → tap corners → robot paints it. Like planning a drone mission, but for paint.

## Build Phases

| Phase | Description | Status |
|-------|-------------|--------|
| 1 | Rolling chassis (motors, wheels, frame, Pi) | 🔲 Not started |
| 2 | Navigation (RTK GPS, encoders, IMU, sensor fusion) | 🔲 Not started |
| 3 | Paint module (pump, solenoid, tank, nozzle) | 🔲 Not started |
| 4 | Software (path planning, templates, phone app) | 🔲 Not started |
| 5 | Laser weeding module (camera, AI, laser, pan/tilt) | 🔲 Not started |
| 6 | Spray module (wider nozzle, zone planning in app) | 🔲 Not started |

## Budget

| Configuration | Cost |
|--------------|------|
| Base + Paint only | $383 |
| Base + Paint + Spray | $383 (same hardware) |
| Base + Paint + Laser | $528 |
| Base + All Above + Trailer | $618 |
| **Full platform (everything)** | **~$618** |

See [BOM.md](docs/BOM.md) for full parts list with purchase links.

## Docs

- [BOM.md](docs/BOM.md) — Bill of Materials with sources and prices
- [NAVIGATION.md](docs/NAVIGATION.md) — RTK GPS architecture, LC29H setup, coordinate system
- [DESIGN.md](docs/DESIGN.md) — Detailed design decisions and tradeoffs
- [WIRING.md](docs/WIRING.md) — Wiring diagrams and pinouts
- [SOFTWARE.md](docs/SOFTWARE.md) — Software architecture and setup
- [APP.md](docs/APP.md) — Mobile app design (flight planner UI)
- [WEEDER.md](docs/WEEDER.md) — Laser weeding module (lawn + garden)
- [SPRAY-MODULE.md](docs/SPRAY-MODULE.md) — Blanket spray module (fence lines, gravel, rocks)
- [TRAILER.md](docs/TRAILER.md) — Tow-behind trailer (fertilizer, seed, ice melt, liquid spray bar)

## Commercial Comparison

| Product | What It Does | Price |
|---------|-------------|-------|
| Turf Tank | GPS field painting | $15,000-30,000 |
| Tertill | Solar weeding robot | $400 (barely works) |
| FarmBot | Garden CNC robot | $1,500-4,000 |
| Carbon Robotics LaserWeeder | Laser weeding | $1,500,000 |
| **FieldBot** | **All of the above** | **$383-528** |

## License

MIT — build one, sell one, modify it, go wild.

## Contributing

Issues and PRs welcome. This is a learn-in-public project.
