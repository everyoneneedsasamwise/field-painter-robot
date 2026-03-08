# Design Decisions & Tradeoffs

## Navigation: Why UWB Over Alternatives

### Options Evaluated

| Method | Accuracy | Range | Cost | Complexity | Chosen? |
|--------|----------|-------|------|------------|---------|
| RTK GPS | 2cm | Unlimited | $300+ (needs base station) | Medium | ❌ Over budget |
| UWB Triangulation | 10-30cm | 50-100m | $90 | Medium | ✅ Best fit |
| LiDAR + Reflectors | 2-5cm | 12-25m | $100-300 | High | ❌ Range too short |
| Encoders + IMU Only | N/A (relative) | N/A | $17 | Low | ❌ Drift too much |
| Visual (Camera) | Variable | Line of sight | $30 | Very High | ❌ Too complex |

### UWB Details
- **Module:** Decawave DWM3000 (or DWM1001 as alternative)
- **Protocol:** Two-Way Ranging (TWR) for distance measurement
- **Topology:** 4 anchors at known positions, 1 mobile tag
- **Update rate:** 10-50 Hz depending on configuration
- **Range:** Up to 100m line-of-sight (perfect for soccer field)
- **Accuracy:** 10-30cm typical, improved with more anchors

### Sensor Fusion Strategy
UWB gives absolute position but is noisy. Encoders + IMU give smooth relative motion but drift.

**Extended Kalman Filter (EKF)** fuses all three:
1. **Prediction step:** Use encoder speeds + IMU heading to predict next position
2. **Update step:** When UWB fix arrives, correct the prediction
3. Result: smooth, accurate trajectory with ~5-10cm effective accuracy

## Drive: Why 2WD Differential

### Options Evaluated

| Config | Pros | Cons | Chosen? |
|--------|------|------|---------|
| 2WD + caster | Simple, gentle on turf, precise turning | Less traction on hills | ✅ |
| 4WD skid steer | Good traction | Tears up turf on turns | ❌ |
| Tracks | Handles any terrain | Destroys grass, heavy, complex | ❌ |
| Ackermann (car-style) | Smooth at speed | Wide turn radius, complex | ❌ |

### Key specs
- **Wheel diameter:** 10" pneumatic (absorbs bumps)
- **Motor speed:** 30 RPM → ~0.4 m/s at the wheel
- **Painting speed:** 0.3-0.5 m/s is ideal for clean lines
- **Turning:** Zero-radius turns by spinning wheels opposite directions
- **Weight target:** Under 20 lbs empty, ~28 lbs with full paint tank

## Paint System: Pump + Solenoid vs Spray Can

### Options Evaluated

| Method | Pros | Cons | Chosen? |
|--------|------|------|---------|
| Pump + solenoid + nozzle | Refillable, adjustable, consistent | More plumbing | ✅ |
| Inverted spray can + servo | Dead simple | Limited paint, can pressure varies | ❌ |
| Gravity drip | Simplest possible | Uneven lines, weather dependent | ❌ |

### Line Width Control
- Adjustable flat fan nozzle: 2-4" line width
- Line width affected by: nozzle height, pressure, robot speed
- Calibration routine: paint test strip, measure, adjust

### Paint Type
- Water-based field marking paint (eco-friendly, non-toxic)
- Available at Home Depot, sports supply stores
- ~$15-25 per gallon
- One gallon covers approximately one full soccer field

## Frame Design

```
        FRONT
   ┌───────────────┐
   │  [UWB]  [US] [US]  │  ← Ultrasonic sensors
   │                     │
   │  ┌─────────────┐   │
   │  │ Electronics │   │  ← Weatherproof box (Pi, drivers, etc.)
   │  │    Box      │   │
   │  └─────────────┘   │
   │                     │
  [M]●               ●[M]  ← Drive motors + 10" wheels
   │                     │
   │    ┌─────────┐     │
   │    │  Paint  │     │  ← 1 gal tank (centered)
   │    │  Tank   │     │
   │    └────┬────┘     │
   │         │          │
   │      [NOZZLE]      │  ← Spray nozzle (paints behind robot)
   │         ●          │  ← Rear caster
   └───────────────────┘
        REAR

   Overall: ~24" x 18" x 12" tall
```

### Design Principles
1. **Paint behind the robot** — never drive over wet lines
2. **Center of gravity over drive wheels** — better traction
3. **Electronics elevated and enclosed** — protected from paint splatter and weather
4. **Quick-disconnect paint system** — easy cleaning
5. **Battery accessible** — swap without tools

## Field Templates

### Standard Soccer Field (FIFA)
- Field: 100-110m x 64-75m
- Center circle: 9.15m radius
- Penalty area: 40.32m x 16.5m
- Goal area: 18.32m x 5.5m
- Line width: 5" (12cm) per FIFA rules

### Other Supported Fields (Future)
- Football (American)
- Lacrosse
- Field hockey
- Custom designs (logos, text, patterns)

## Safety

- **E-stop button** — physical kill switch on the robot
- **Geofence** — software boundary, robot stops if UWB position exceeds field bounds + margin
- **Obstacle detection** — ultrasonics stop motors if obstacle within 50cm
- **Bumper** — physical contact kills motors immediately
- **Watchdog timer** — if Pi crashes, motor driver defaults to stopped
- **Low battery cutoff** — stops gracefully, saves progress for resume
