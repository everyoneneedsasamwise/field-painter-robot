# Fertilizer/Spreader Trailer Module

## Overview

A tow-behind trailer with swappable hopper configurations for granular spreading and liquid spraying. Connects to the robot via a hitch pin and powered wire harness.

## Trailer Design

```
     ROBOT                          TRAILER
┌──────────────┐              ┌──────────────────┐
│              │              │                  │
│         ●caster    hitch    │  ┌────────────┐  │
│              ├──────────────┤  │   HOPPER   │  │
│              │   tongue     │  │  (5 gal)   │  │
│  [M]●    ●[M]│              │  └─────┬──────┘  │
└──────────────┘              │    servo gate    │
                              │        │         │
                              │   ┌────▼─────┐   │
                              │   │ SPINNING │   │
                              │   │   DISC   │   │
                              │   └──────────┘   │
                              │                  │
                              │   ●          ●   │
                              └──────────────────┘
                                 passive wheels
```

## Towing Dynamics — Driving With a Trailer

A differential drive robot with a trailer is NOT the same as without one. The trailer changes everything about turning.

### The Problem

```
Without trailer:              With trailer:
Robot pivots in place         Trailer jackknifes if you
        ↻                     try to pivot in place!

                              ┌─────┐
                              │ROBOT│ ↻
                              └──┬──┘
                                 │ ← tongue swings wide
                              ┌──▼──┐
                              │TRAIL│ → crashes into things
                              └─────┘
```

### Driving Rules (Trailer Mode)

**1. No zero-radius turns**
- Normal mode: robot spins in place by running wheels opposite directions
- Trailer mode: MINIMUM turn radius enforced based on tongue length
- Min radius = tongue length × 1.5 (rule of thumb)
- With 18" tongue → min turn radius ~27"

**2. Wider turns, planned ahead**
- Path planner must account for trailer sweep
- Inside corner of turn sweeps tighter than the robot (off-tracking)
- Buffer distance from obstacles increases in trailer mode

**3. Reverse is complex**
- Backing up with a trailer is inherently unstable
- Robot should AVOID reversing with trailer attached
- If reverse is needed: very slow, very short distances, with constant heading correction
- Better: plan paths that don't require reversing (loop patterns)

**4. Speed limits**
- Max speed reduced in trailer mode (0.2 m/s vs 0.4 m/s)
- Slower = more stable, less risk of jackknife
- Better spreading coverage anyway (more granules per sq ft)

### Software Changes for Trailer Mode

```python
class TrailerController:
    """Modified path following for towing."""
    
    def __init__(self, tongue_length_m=0.45):
        self.tongue_length = tongue_length_m
        self.min_turn_radius = tongue_length_m * 1.5
        self.max_speed = 0.2  # m/s (reduced from 0.4)
        self.trailer_mode = True
    
    def plan_path(self, waypoints):
        """Re-plan path with trailer constraints."""
        # 1. Replace sharp corners with arcs (min radius)
        # 2. Add approach/exit tangents to each waypoint
        # 3. Eliminate reversing segments
        # 4. Widen obstacle buffers by trailer width
        # 5. Use Dubins path or Reeds-Shepp curves
        pass
    
    def calculate_turn(self, current_heading, target_heading):
        """Enforce minimum turn radius."""
        # No spinning in place!
        # Calculate arc length needed at min_turn_radius
        # Return left/right wheel speeds for smooth arc
        pass
    
    def pure_pursuit_trailer(self, position, path):
        """Modified pure pursuit with longer lookahead."""
        # Increase lookahead distance (smoother curves)
        # Account for trailer off-tracking on turns
        # Adjust wheel speeds to maintain stable tow
        pass
```

### Path Patterns (Trailer-Safe)

**Lawn coverage pattern:**
```
Standard mowing pattern but with rounded turns:

    ──────────────╮
                  │  ← wide U-turn (no sharp corners)
    ──────────────╯
    ╭──────────────
    │
    ╰──────────────
                  ╭─
    ──────────────╯
```

Turn radius at ends = max(min_turn_radius, 0.7m)

**Driveway/sidewalk pattern:**
```
Follow the path outline with no reversing:

    ┌──────────────────┐
    │  ╭──╮  ╭──╮  ╭──│
    │  │  │  │  │  │  │
    │  ╰──╯  ╰──╯  ╰──│
    │     serpentine    │
    └──────────────────┘
```

## Powered Trailer Hookup

### Wire Harness — 6-Pin Connector

The trailer needs power and control signals from the robot. Use a 6-pin aviation connector (GX16-6) for a weatherproof, quick-disconnect connection.

```
Pin 1: 12V Power      (from robot buck converter)
Pin 2: Ground          (common)
Pin 3: Gate Servo PWM  (Pi GPIO → servo signal)
Pin 4: Disc Motor PWM  (Pi GPIO → motor driver)
Pin 5: Disc Motor DIR  (forward/reverse spin)
Pin 6: Hopper Sensor   (weight/level sensor, analog)
```

### Connector Choice
- **GX16-6 Aviation Connector** (~$5/pair on Amazon)
- Waterproof, keyed (can't plug in wrong), rated for 5A per pin
- Screw-lock so it won't vibrate loose
- Mount: female on robot rear, male on trailer tongue

```
Robot rear panel:          Trailer tongue:
  ┌───────────┐              ┌───────────┐
  │  ◉ GX16-6 │──── cable ───│ ◉ GX16-6 │
  │  (female)  │              │  (male)   │
  └───────────┘              └───────────┘
```

### Hitch + Connector Combined

```
Side view:
                    ┌─── GX16 connector
                    │
Robot ══════╗   ╔═══╧═══╗══════ Trailer tongue
            ║   ║       ║
            ╚═══╣  PIN  ╠═══╝
                ║       ║
                ╚═══════╝
                    │
                    └─── Hitch pin (allows rotation)
```

The hitch pin allows the trailer to rotate freely for turns.
The GX16 cable has enough slack (12") to allow full rotation without binding.

## Gate Control System

### Granular Mode — Servo Gate

```
           ┌─────────────┐
           │   HOPPER     │
           │   (5 gal)    │
           │              │
           └──────┬───────┘
                  │ funnel narrows to 2" opening
           ┌──────▼───────┐
           │  ┌─────────┐ │
           │  │  GATE   │←├── MG996R Servo
           │  │ (slide) │ │   PWM from Pin 3
           │  └─────────┘ │
           └──────┬───────┘
                  │ granules fall
           ┌──────▼───────┐
           │  SPINNING    │←── DC Motor
           │    DISC      │    PWM from Pin 4
           └──────────────┘
              flings outward 3-8 ft
```

**Gate positions:**
- **Closed (0°):** No flow — traveling between zones
- **Narrow (30°):** Light application — maintenance feed
- **Medium (60°):** Standard application — fertilizer label rate
- **Full open (90°):** Heavy application — new lawn, heavy feed

The servo gives precise, variable flow control. The Pi adjusts gate position based on:
- Robot speed (slower = less gate opening needed)
- Application rate setting in app (lbs per 1000 sq ft)
- Granule size (coarse vs fine needs different opening)

### Liquid Mode — Same Pump System

For liquid fertilizer trailer config, the pump and solenoid mount ON the trailer instead of the robot:

```
           ┌─────────────┐
           │  LIQUID TANK │
           │   (5 gal)    │
           └──────┬───────┘
                  │
           ┌──────▼───────┐
           │    PUMP       │←── Power from Pin 1
           │  (12V diag)   │    Control from Pin 4
           └──────┬───────┘
                  │
           ┌──────▼───────┐
           │  SOLENOID     │←── Control from Pin 3
           └──────┬───────┘
                  │
        ┌─────────┼─────────┐
        ▼         ▼         ▼
    [nozzle]  [nozzle]  [nozzle]
        ←── spray bar (36" wide) ──→
```

The spray bar with 3 nozzles gives ~36" coverage width — much wider than the single robot nozzle.

## Hopper Level Sensing

Don't run empty and waste a pass. Simple options:

| Sensor | Cost | How |
|--------|------|-----|
| Load cell (HX711) | $5 | Weighs the hopper — knows exactly how much is left |
| Ultrasonic (in hopper) | $3 | Measures distance to granule surface |
| Simple float switch | $2 | Just tells you "low" when nearly empty |

**Recommendation:** Load cell ($5). Mount under the hopper. Gives precise weight → app shows remaining product AND calculates if you have enough for the planned area before you start.

Signal comes back to Pi on Pin 6 (analog via ADC on the trailer's own small board, or digitized via I2C).

## Trailer BOM

| Part | Cost | Notes |
|------|------|-------|
| Aluminum angle frame (24"x16") | $12 | Home Depot |
| 2x 6" pneumatic wheels | $15 | Garden cart style |
| Axle rod + bearings | $8 | 12mm rod |
| Hitch pin + brackets | $5 | Hardware store |
| GX16-6 Aviation Connector pair | $5 | Amazon |
| 5 gal bucket + funnel mod | $8 | 3D print funnel insert |
| MG996R Servo (gate) | $6 | Amazon |
| DC Motor + spinning disc | $10 | Amazon |
| Small motor driver (MOSFET) | $4 | Amazon |
| HX711 Load Cell | $5 | Amazon |
| Wire harness (18AWG, 3ft) | $4 | Amazon |
| Hardware (bolts, brackets) | $8 | Home Depot |
| **Total** | **~$90** |

## App Integration

### Trailer Detection
- App detects when trailer connector is live (Pin 6 reads a value)
- Automatically switches to trailer driving mode
- Enforces minimum turn radius
- Reduces max speed
- Shows trailer-specific UI

### Spreader Settings
```
┌─────────────────────────┐
│ 🌱 SPREADER SETTINGS    │
│                         │
│ Product: Scott's Turf B │
│ Rate: 2.87 lbs/1000sqft │
│ Bag size: 15 lbs        │
│                         │
│ Hopper level: ████░ 78% │
│ Coverage remaining:     │
│   ~4,100 sq ft          │
│                         │
│ Area planned: 3,200sqft │
│ ✅ Enough product       │
│                         │
│ [START SPREADING]       │
└─────────────────────────┘
```

### Product Library
Pre-programmed settings for common products:
- Scott's Turf Builder (various)
- Milorganite
- GrubEx
- Grass seed (by species)
- Ice melt (by type)
- Custom (enter rate manually)

Each entry stores: application rate, granule size, recommended gate opening, disc speed.

## Safety

- **Fertilizer dust:** Robot handles it, not you
- **Even application:** Servo gate + speed control prevents burn spots
- **No double-coverage:** GPS tracks exactly where product was applied
- **Rain awareness:** App warns if rain is forecast (washes away granular before absorption)
- **Ice melt mode:** Check surface temperature, only spread where needed
