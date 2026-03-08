# Bill of Materials (BOM)

Total estimated cost: **~$437** (with ~$63 buffer to $500)

## 🧠 Brain & Navigation — $162

| Part | Qty | Unit Price | Total | Source | Notes |
|------|-----|-----------|-------|--------|-------|
| Raspberry Pi 4 (4GB) | 1 | $55 | $55 | [PiShop.us](https://www.pishop.us) / Amazon | Main controller |
| Makerfabs UWB DWM3000 Module | 5 | $18 | $90 | [Makerfabs](https://www.makerfabs.com/uwb-dwm3000.html) | 4 anchors + 1 tag |
| MPU-6050 IMU (GY-521) | 1 | $5 | $5 | Amazon | Gyro + accelerometer |
| Hall Effect Wheel Encoder Kit | 2 | $6 | $12 | Amazon | Speed + distance tracking |

### UWB Anchor Setup
- 4 anchors mounted on stakes/tripods at field corners
- Each needs power: small USB battery pack or 3xAA holder ($5-10 extra)
- Tag module mounts on robot, connects to Pi via SPI

## ⚡ Drive System — $88

| Part | Qty | Unit Price | Total | Source | Notes |
|------|-----|-----------|-------|--------|-------|
| JGB37-520 12V 30RPM Gear Motor | 2 | $20 | $40 | Amazon | High torque, low speed |
| L298N Dual Motor Driver | 1 | $8 | $8 | Amazon | Handles 2A per channel |
| 10" Pneumatic Tire + 12mm Hub | 2 | $15 | $30 | Amazon / Harbor Freight | Absorbs bumps, no turf damage |
| 3" Swivel Caster (rubber) | 1 | $10 | $10 | Amazon / Home Depot | Rear passive wheel |

### Motor Sizing Notes
- 30 RPM with 10" wheels ≈ 0.4 m/s (1.3 ft/s) — good painting speed
- Need sufficient torque to handle grass/bumps: JGB37-520 provides ~10 kg·cm
- If too slow, can go 60RPM but paint quality may suffer

## 🎨 Paint System — $58

| Part | Qty | Unit Price | Total | Source | Notes |
|------|-----|-----------|-------|--------|-------|
| 12V Solenoid Valve (normally closed) | 1 | $12 | $12 | Amazon | On/off paint flow |
| 12V Diaphragm Pump (45 PSI) | 1 | $15 | $15 | Amazon | Pressurizes paint line |
| Adjustable Flat Fan Spray Tip | 2 | $4 | $8 | Amazon | 2-4" line width |
| 1 Gallon HDPE Tank | 1 | $10 | $10 | Hardware store | Paint reservoir |
| Silicone Tubing (3/8" ID, 6ft) | 1 | $8 | $8 | Amazon | Tank → pump → valve → nozzle |
| Inline Filter/Strainer | 1 | $5 | $5 | Amazon | Prevents clogs |

### Paint Notes
- Use field marking paint (water-based, non-toxic) — available at hardware stores
- Dilute if needed for spray consistency
- Flow rate calibration: adjust pump duty cycle + nozzle size
- Clean system with water after each use

## 🔋 Power — $64

| Part | Qty | Unit Price | Total | Source | Notes |
|------|-----|-----------|-------|--------|-------|
| DeWalt 20V 5Ah Battery | 1 | $40 | $40 | Amazon / Home Depot | Or use existing if you have one |
| DeWalt Battery Adapter Plate | 1 | $12 | $12 | Amazon | Wired output from battery |
| LM2596 Buck Converter (adj) | 1 | $6 | $6 | Amazon | 20V → 12V for motors/pump |
| LM2596 Buck Converter (5V) | 1 | $6 | $6 | Amazon | 20V → 5V for Pi |

### Power Budget
| Component | Voltage | Current (max) | Watts |
|-----------|---------|---------------|-------|
| Raspberry Pi 4 | 5V | 3A | 15W |
| 2x Drive Motors | 12V | 2A total | 24W |
| Paint Pump | 12V | 1.5A | 18W |
| Solenoid Valve | 12V | 0.5A | 6W |
| Sensors + UWB | 5V | 0.5A | 2.5W |
| **Total** | | | **~65W** |

DeWalt 20V 5Ah = 100Wh → **~1.5 hours runtime** at full draw. Enough for a soccer field.

## 🛡️ Obstacle Avoidance — $8

| Part | Qty | Unit Price | Total | Source | Notes |
|------|-----|-----------|-------|--------|-------|
| HC-SR04 Ultrasonic Sensor | 2 | $2.50 | $5 | Amazon | Front left + right |
| Microswitch (bumper) | 2 | $1.50 | $3 | Amazon | Physical last-resort stop |

## 🔧 Frame & Hardware — $57

| Part | Qty | Unit Price | Total | Source | Notes |
|------|-----|-----------|-------|--------|-------|
| 1" Aluminum Square Tube (8ft) | 2 | $8 | $16 | Home Depot | Main frame rails |
| Aluminum Angle (4ft) | 2 | $5 | $10 | Home Depot | Cross braces, motor mounts |
| Weatherproof Project Box (large) | 1 | $12 | $12 | Amazon | Electronics enclosure |
| Bolts, nuts, washers (assorted) | 1 | $10 | $10 | Home Depot | M6 + M8 for frame |
| Wiring kit (14-22 AWG, connectors) | 1 | $9 | $9 | Amazon | XT60, spade, JST connectors |

### Frame Design
- Rectangular base: ~24" x 18"
- Motors mount at front, caster at rear
- Paint tank centered for balance
- Electronics box on top, protected from spray
- Nozzle points down between rear wheels (paints behind the robot)

## 📊 Cost Summary

| Category | Cost |
|----------|------|
| Brain & Navigation | $162 |
| Drive System | $88 |
| Paint System | $58 |
| Power | $64 |
| Obstacle Avoidance | $8 |
| Frame & Hardware | $57 |
| **TOTAL** | **$437** |
| **Budget remaining** | **$63** |

## Substitutions to Save Money

- Already have a DeWalt battery? Save $40
- Use Arduino Mega instead of Pi: save $30 (but harder software dev)
- 3D print motor mounts + brackets: save $10-15 on hardware
- Smaller wheels (8"): save $10 but worse on bumps
