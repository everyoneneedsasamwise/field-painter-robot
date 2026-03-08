# WeedBot — Laser Weeding Module

## Overview

A swappable tool module for the FieldBot platform that autonomously identifies and kills weeds using a focused blue laser. Works in both lawn and garden modes.

**Primary target:** Goatheads (puncturevine / Tribulus terrestris) — flat-growing weed with central crown, ideal for laser treatment.

## How It Works

```
Camera scans ground (30 FPS)
        │
        ▼
AI model identifies weeds
(species, location, size)
        │
        ▼
Pan/tilt servos aim laser
at weed growing point (meristem)
        │
        ▼
445nm blue laser fires
for 0.5-1.5 seconds
        │
        ▼
Weed meristem destroyed
(plant dies within 24-48 hours)
        │
        ▼
Robot moves to next target
```

## Why Blue Laser (445nm)?

- Plants absorb blue light maximally at 435-450nm (chlorophyll absorption peak)
- 445nm blue diode lasers are cheap and efficient
- 5-10W optical power is sufficient to destroy weed meristems
- Focused beam = surgical precision, doesn't damage surrounding grass/crops
- No chemicals, no soil disturbance, no residue

## Two Operating Modes

### 🌿 Lawn Mode
- Robot mows a systematic pattern across the entire lawn (like a Roomba)
- Camera scans continuously for broadleaf weeds
- AI distinguishes weeds from grass:
  - Goatheads: flat rosette pattern, serrated leaflets, distinctive shape
  - Dandelions: broad leaves, rosette, yellow flowers
  - Crabgrass: wide blades, spreading from central point
  - Clover: trifoliate leaves
- Laser targets the growing crown/meristem of each weed
- Skips grass completely

### 🥬 Garden Mode
- User maps garden beds and crop locations in the app
- Robot knows exactly where crops are planted (GPS coordinates)
- Everything NOT in a crop zone = weed → laser it
- Can also use AI to identify specific crop species as extra safety
- Drives between rows, scans both sides
- Safe zone buffer around known crop positions (configurable, default 3")

## Hardware — Laser Module Add-on

### Core Components

| Part | Cost | Notes |
|------|------|-------|
| 5.5W 445nm Blue Laser Module (TTL) | $50-60 | Focusable, 12V, PWM power control |
| 2x MG996R Servo (pan + tilt) | $12 | Metal gear, high torque for aiming |
| Pan/Tilt Bracket | $8 | 2-axis mount for laser + camera |
| Pi Camera Module 3 | $25 | Wide angle, 12MP, autofocus |
| Google Coral USB Accelerator | $25 | Edge TPU for real-time AI inference |
| Laser safety glasses (445nm OD5+) | $15 | **MANDATORY for anyone nearby** |
| Laser interlock switch | $5 | Emergency cutoff |
| **Total add-on cost** | **~$140-150** | |

### Safety Components (Non-negotiable)

| Part | Purpose |
|------|---------|
| Key switch interlock | Laser can't fire without key |
| Emergency stop button | Instant laser kill |
| Tilt sensor / IMU check | Laser disables if robot tips |
| Software watchdog | Laser auto-off if camera/AI crashes |
| Proximity sensor check | Laser disables if obstacle detected within 2m |
| Beam containment skirt | Fabric/rubber skirt around laser area blocks stray reflections |
| OD5+ safety glasses | For operator — included in BOM |

## Laser Specifications

| Parameter | Value |
|-----------|-------|
| Wavelength | 445nm (blue) |
| Optical power | 5-5.5W (real optical, not "input watts") |
| Spot size at focus | ~1-2mm |
| Working distance | 15-30cm (from robot to ground) |
| Dwell time per weed | 0.5-1.5 seconds |
| Weeds per minute | ~20-40 (depending on density) |
| Power supply | 12V DC (from robot main supply) |
| Control | TTL on/off + PWM power level |
| Duty cycle | Pulsed (only on during targeting) |

### Kill Mechanism
The laser heats the meristem (growing point) to 60-80°C, denaturing proteins and destroying the growth cells. The weed can't regrow from the crown and dies within 24-48 hours. Roots may persist briefly but without the crown producing energy, the plant starves.

**For goatheads specifically:** The central growing crown is an ideal target — flat, exposed, usually visible. One 1-second burst kills the plant. Existing seed pods on the ground won't be affected (they're already hardened), but preventing new growth stops new seeds from forming.

## AI / Computer Vision

### Architecture

```
Pi Camera (30 FPS, 1080p)
        │
        ▼
Pre-processing (crop, resize to 320x320)
        │
        ▼
YOLOv8-nano (on Coral Edge TPU)
  - Detect weed bounding boxes
  - Classify species
  - ~30ms per frame = real-time
        │
        ▼
Post-processing
  - Calculate meristem position (center of rosette)
  - Convert pixel coords → servo angles
  - Filter false positives (confidence threshold)
        │
        ▼
Servo controller aims laser
        │
        ▼
Laser fires (TTL high) for calculated dwell time
```

### Training Data
- Start with existing weed datasets (DeepWeeds, PlantVillage)
- Fine-tune with photos of YOUR specific weeds (goatheads, dandelions, etc.)
- Take 200-500 photos of each weed species in your yard with the Pi camera
- Label with bounding boxes using LabelStudio or Roboflow (free)
- Train YOLOv8-nano → export to TFLite → deploy to Coral

### Species the AI Should Learn

**Lawn mode (kill these):**
- Goathead / puncturevine (Tribulus terrestris) — PRIMARY TARGET
- Dandelion (Taraxacum)
- Crabgrass (Digitaria)
- Clover (Trifolium) — optional, some people like it
- Bindweed (Convolvulus)
- Spurge (Euphorbia)

**Lawn mode (protect these):**
- Kentucky bluegrass
- Tall fescue
- Perennial ryegrass

**Garden mode:**
- Everything not at a known crop GPS coordinate = weed
- Optional crop identification as safety layer

## Software Integration

### New App Screens

**Weed Map:**
- Shows satellite/map view of lawn with detected weed locations (red dots)
- Heat map of weed density by area
- Historical tracking (are weeds decreasing over time?)
- Photo gallery of detected weeds

**Garden Planner:**
- Map garden beds on satellite view (like field designer)
- Mark crop positions with species labels
- Set safe zones around each crop
- Import planting maps from seed spacing charts

**Weeding Session:**
- Start/stop/pause
- Live camera feed with AI overlay (bounding boxes on detected weeds)
- Kill counter (weeds zapped this session)
- Estimated completion time
- Battery remaining

### API Extensions

```python
# New endpoints for weeding module
POST /weed/start          # Start weeding session (lawn or garden mode)
POST /weed/pause          # Pause session
POST /weed/stop           # Stop session
GET  /weed/stats          # Kill count, area covered, time elapsed
GET  /weed/map            # Weed location data for map view
GET  /weed/camera         # Live camera stream with AI overlay
POST /weed/train          # Upload training images for AI model
POST /weed/calibrate      # Calibrate laser aim vs camera offset
```

## Operating Procedure

### First-Time Setup
1. Mount laser module on robot (quick-connect)
2. Calibrate camera-to-laser offset (built-in calibration routine)
3. Walk the yard with the app, mark any areas to SKIP (flower beds, decorative plants)
4. Set operating schedule (optional: automated runs via cron)

### Typical Weeding Session (Lawn)
1. Place robot at starting position
2. Open app → Weed Mode → Lawn
3. Robot drives systematic mowing pattern
4. Camera scans, AI detects, laser fires
5. Session ends when area is covered or battery low
6. Review weed map + kill count in app

### Goathead Battle Plan
Goatheads are prolific — a single plant produces hundreds of spiny seeds. Strategy:
1. **Early spring:** Run weeder weekly when plants are small (easier to kill, no seeds yet)
2. **Summer:** Increase frequency to 2-3x per week during peak growth
3. **Fall:** Focus on any remaining plants before they seed
4. **Year 2:** Dramatically fewer goatheads (seed bank depletes over time)
5. **Ongoing:** Monthly maintenance runs

The key is killing them BEFORE they produce seeds. A single goathead plant can produce 200-5000 seeds that remain viable for 4-5 years. Consistent laser treatment breaks the cycle.

## Shared Platform Architecture

```
FieldBot Base Platform
├── Chassis (wheels, motors, frame)
├── RTK GPS Navigation
├── Raspberry Pi Brain
├── Battery (Ryobi 18V)
├── Phone App
│
├── Tool Mount (quick-connect, rear of robot)
│   │
│   ├── 🎨 PAINT MODULE
│   │   ├── Paint tank
│   │   ├── Pump + solenoid
│   │   └── Spray nozzle
│   │
│   ├── ⚡ LASER WEED MODULE
│   │   ├── Pi Camera + Coral TPU
│   │   ├── Pan/tilt servo mount
│   │   ├── 5.5W blue laser
│   │   └── Safety interlock
│   │
│   └── 🔮 FUTURE MODULES
│       ├── 💧 Precision watering
│       ├── 🌱 Seed planter
│       └── 📷 Crop scout / health monitor
```

## Safety Warnings ⚠️

**A 5W laser can cause PERMANENT EYE DAMAGE instantly.**

1. **Never look at the beam** — even reflections off shiny surfaces
2. **OD5+ safety glasses** are mandatory for anyone within 50 feet
3. **Beam containment skirt** keeps the laser aimed downward
4. **Interlock system** prevents firing when robot is tilted, moved, or obstacles detected
5. **No operation when people/pets are in the yard** — set up a perimeter
6. **Key switch** prevents unauthorized operation
7. **Software watchdog** kills laser if any sensor fails
8. The robot should have clear WARNING labels and possibly a flashing light when laser is active
9. **Check local regulations** — some areas may regulate outdoor laser use

## Budget Summary

| Item | Already Have | Need to Buy |
|------|-------------|-------------|
| Robot base platform | — | $383 (FieldBot BOM) |
| Laser weeding module | — | ~$140-150 |
| **Total for both capabilities** | | **~$523-533** |

If built as weeder-only (no paint), the base + laser module = ~$475 (skip paint pump/solenoid/tank).
