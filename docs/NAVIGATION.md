# Navigation System — RTK GPS

## Architecture Change: UWB → RTK GPS

After research into the Quectel LC29H module ($60), we're switching primary navigation from UWB to RTK GPS.

**Why:**
- 1-2cm accuracy (vs 10-30cm for UWB)
- Unlimited range (vs 50-100m for UWB)
- Well-established ecosystem (NTRIP, CORS, ArduPilot, ROS)
- Cheaper for the accuracy level ($120 for rover + base)
- Massive open-source community (OpenMower, ArduMower, ArduPilot)

## Hardware

### Rover (on robot)
- **Module:** Quectel LC29H(DA) — Waveshare breakout board
- **Price:** ~$60
- **Interface:** UART (serial) to Raspberry Pi
- **Antenna:** Included patch antenna (good enough) or upgrade to external L1/L2 antenna ($15-30)
- **Update rate:** 1-10 Hz (configurable)
- **Accuracy:** 1-2cm with RTK fix

### Base Station (stationary at field edge)
- **Module:** Quectel LC29H(BS) — Waveshare breakout board
- **Price:** ~$60
- **Interface:** UART to Raspberry Pi Zero or ESP32
- **Setup:** Mount on tripod at known position, streams RTCM corrections to rover
- **Communication:** WiFi (Pi Zero → robot Pi) or radio (LoRa for longer range)

### Alternative: Free NTRIP Service (no base station needed)
- **CORS Network:** US has CORS stations — free RTCM corrections via internet
- **RTK2GO:** Community-run free NTRIP caster
- **Requirement:** Internet connection on the robot (phone hotspot or WiFi)
- **Caveat:** Base station needs to be within ~30km for best accuracy
- **Check coverage:** https://geodesy.noaa.gov/CORS_Map/
- **Utah coverage:** Multiple CORS stations in the Wasatch Front area

### Recommended Setup
| Component | Part | Cost |
|-----------|------|------|
| Rover GPS | LC29H(DA) Waveshare | $60 |
| Base GPS | LC29H(BS) Waveshare | $60 |
| Base controller | ESP32 or Pi Zero W | $10-35 |
| Base tripod | Camera tripod | $15 (or use existing) |
| Base battery | USB power bank | $10 (or use existing) |
| **Total** | | **$120-180** |

If CORS coverage is good in your area, skip the base station and save $60-85.

## How RTK Works

```
    GPS Satellites (L1 + L2 bands)
         │              │
         ▼              ▼
   ┌──────────┐   ┌──────────┐
   │  Base    │   │  Rover   │
   │ LC29H-BS│   │ LC29H-DA │
   │ (known  │   │ (on robot)│
   │ position)│   │          │
   └────┬─────┘   └────┬─────┘
        │               │
        │  RTCM3 corrections
        │  (via WiFi/radio)
        └───────────────┘
                │
                ▼
        Rover computes position
        relative to base = 1-2cm accuracy
```

**Key concept:** Both receivers see the same satellites. The base knows its exact position, so it can calculate errors in the satellite signals. It sends these corrections (RTCM format) to the rover. The rover subtracts the errors = centimeter accuracy.

**Fix types:**
- **No fix:** No position (startup, bad sky view)
- **GPS fix:** Standard GPS (~2-3m accuracy)
- **RTK Float:** Getting corrections, converging (~30-50cm)
- **RTK Fix:** Full correction applied (**1-2cm accuracy**) ← what we want

Time to RTK fix: typically 30-90 seconds after power on with good sky view.

## LC29H Module Variants (IMPORTANT)

These look identical but do completely different things:
| Variant | Function | Use |
|---------|----------|-----|
| LC29H(AA) | Standard GPS + augmentation | ❌ NOT for RTK |
| LC29H(DA) | RTK Rover | ✅ Robot |
| LC29H(BS) | RTK Base Station | ✅ Field edge |

**Double check when ordering!** They're all called "LC29H" — verify the suffix.

## Sensor Fusion (Same as Before)

RTK GPS replaces UWB but the fusion strategy stays the same:

```
RTK GPS (1-2cm, 5-10 Hz)
     +
Wheel Encoders (continuous, relative)
     +
IMU / MPU-6050 (heading, 100 Hz)
     ↓
Extended Kalman Filter
     ↓
Fused Position (smooth, accurate)
```

The EKF smooths out GPS jitter between fixes and keeps the robot on course during brief GPS dropouts.

## Coordinate System

### GPS → Local Field Coordinates
1. **Survey the field:** Walk to each corner with the rover, record GPS coords
2. **Define local frame:** Pick one corner as origin, compute rotation to align field axes
3. **Transform:** All field template coordinates (meters from origin) map to GPS coords
4. **The app does this automatically** — user just walks to corners and taps

### Projection
GPS coords (lat/lon) → local ENU (East-North-Up) in meters:
- Use Haversine or UTM projection
- At soccer field scale (<200m), simple flat-earth approximation is fine
- 1° latitude ≈ 111,000 meters
- 1° longitude ≈ 111,000 × cos(latitude) meters
- At Utah (~41°N): 1° lon ≈ 83,700 meters

## Integration with Flight Planner App

The GPS approach unlocks the satellite view overlay feature:
1. App shows satellite imagery of current location
2. User taps field corners → GPS coords recorded
3. Field template overlaid on satellite view → drag/rotate/scale to fit
4. Template waypoints auto-converted to GPS coordinates
5. Robot follows GPS waypoints to paint

This is exactly how drone flight planners (Mission Planner, QGroundControl) work.

## Open Source References

### OpenMower (github.com/ClemensElflein/OpenMower)
- Full RTK GPS mower, open source
- Uses Pi 4 + ArduSimple F9P (more expensive) or compatible RTK boards
- ROS-based navigation stack
- Has path planning, sensor fusion, obstacle avoidance
- We can reference their nav code

### ArduMower Sunray (wiki.ardumower.de)
- Arduino-based RTK mower firmware
- Sunray firmware has built-in RTK support
- Simpler than OpenMower (no ROS)
- Good reference for motor control + path following

### ArduPilot / ArduRover
- Full autopilot firmware, supports rovers
- Has LC29H support already
- Mission Planner is exactly the flight planner concept
- Could potentially run ArduRover on the Pi directly
