# Software Architecture

## Overview

All software runs on the Raspberry Pi. Python 3.11+ for everything.

```
┌─────────────────────────────────────────────┐
│                Phone App (Future)            │
│         (React Native or Web App)            │
│   - Load/design field templates              │
│   - Set anchor positions                     │
│   - Start/stop/pause                         │
│   - Live position + progress view            │
└─────────────────┬───────────────────────────┘
                  │ WebSocket / BLE
┌─────────────────▼───────────────────────────┐
│              Main Controller (Pi)            │
│                                              │
│  ┌──────────┐ ┌──────────┐ ┌──────────────┐ │
│  │   Path   │ │Navigation│ │    Paint     │ │
│  │ Planner  │→│ + Fusion │→│  Controller  │ │
│  └──────────┘ └──────────┘ └──────────────┘ │
│       │            │              │          │
│  ┌────▼────┐ ┌─────▼─────┐ ┌─────▼───────┐ │
│  │  Field  │ │  Sensor   │ │   Motor     │ │
│  │Templates│ │  Drivers  │ │   Driver    │ │
│  └─────────┘ └───────────┘ └─────────────┘ │
└──────────────────────────────────────────────┘
```

## Module Breakdown

### 1. Field Templates (`fieldbot/templates/`)
- Pre-built templates: soccer, football, lacrosse, etc.
- SVG import for custom designs
- Coordinate system: origin at field center, meters

```python
# Example: soccer field template
class SoccerField:
    length = 100  # meters
    width = 64    # meters
    line_width = 0.12  # meters (FIFA: 12cm / ~5 inches)
    
    def get_paths(self) -> list[Path]:
        """Return all line segments and arcs to paint."""
        return [
            # Perimeter
            Rectangle(0, 0, self.length, self.width),
            # Center line
            Line(self.length/2, 0, self.length/2, self.width),
            # Center circle
            Circle(self.length/2, self.width/2, radius=9.15),
            # Penalty areas, goal areas, etc.
            ...
        ]
```

### 2. Path Planner (`fieldbot/planner/`)
Takes field template → generates optimized waypoint sequence.

**Responsibilities:**
- Convert template shapes to discrete waypoints
- Optimize travel order (minimize unpainted travel — TSP-like)
- Handle curves (interpolate arcs into short segments)
- Generate paint on/off commands at each waypoint

```python
class PathPlanner:
    def plan(self, template: FieldTemplate, resolution_cm=2) -> list[Waypoint]:
        """Convert template to waypoints with paint on/off."""
        # 1. Discretize all paths into points
        # 2. Solve ordering (nearest-neighbor heuristic)
        # 3. Return ordered waypoints with paint state
        pass

@dataclass
class Waypoint:
    x: float        # meters from origin
    y: float        # meters from origin
    heading: float   # radians
    paint: bool      # spray on/off
```

### 3. Navigation & Sensor Fusion (`fieldbot/nav/`)
Fuses UWB + encoders + IMU into a single position estimate.

**Extended Kalman Filter (EKF):**
- State vector: [x, y, heading, velocity]
- Prediction: encoder distance + IMU heading change
- Update: UWB position fix (when available)

```python
class Navigator:
    def __init__(self):
        self.ekf = ExtendedKalmanFilter(state_dim=4, meas_dim=2)
        self.uwb = UWBDriver()
        self.encoders = EncoderDriver()
        self.imu = IMUDriver()
    
    def get_position(self) -> Position:
        """Return fused position estimate."""
        # 1. Read encoders → predict motion
        # 2. Read IMU → update heading
        # 3. Read UWB → correct position
        # 4. Return EKF state
        pass
```

### 4. Motor Controller (`fieldbot/motors/`)
PID control for following waypoint paths.

**Pure Pursuit Algorithm:**
- Looks ahead on the path by a configurable distance
- Calculates steering (wheel speed differential) to reach lookahead point
- Smooth, stable path following even with noisy position

```python
class MotorController:
    def __init__(self):
        self.left = Motor(pin_fwd=17, pin_rev=18, pin_pwm=12)
        self.right = Motor(pin_fwd=22, pin_rev=23, pin_pwm=13)
        self.lookahead = 0.5  # meters
    
    def follow_path(self, current_pos, waypoints):
        """Pure pursuit path following."""
        target = find_lookahead_point(current_pos, waypoints, self.lookahead)
        curvature = calculate_curvature(current_pos, target)
        left_speed, right_speed = differential_drive(curvature)
        self.left.set_speed(left_speed)
        self.right.set_speed(right_speed)
```

### 5. Paint Controller (`fieldbot/paint/`)
Simple on/off control with timing compensation.

```python
class PaintController:
    def __init__(self, solenoid_pin=24, pump_pin=25):
        self.solenoid = GPIO(solenoid_pin)
        self.pump = GPIO(pump_pin)
        self.lead_time_ms = 50  # open valve slightly before reaching paint point
    
    def start_painting(self):
        self.pump.on()
        time.sleep(self.lead_time_ms / 1000)
        self.solenoid.on()
    
    def stop_painting(self):
        self.solenoid.off()
        self.pump.off()
```

### 6. Safety Manager (`fieldbot/safety/`)

```python
class SafetyManager:
    def check(self) -> bool:
        """Returns False if robot should stop."""
        # Ultrasonic obstacle check
        if self.ultrasonic_left.distance_cm() < 50:
            return False
        if self.ultrasonic_right.distance_cm() < 50:
            return False
        # Bumper check
        if self.bumper.pressed():
            return False
        # Geofence check
        if not self.geofence.contains(self.nav.position):
            return False
        # Battery check
        if self.battery.voltage() < MIN_VOLTAGE:
            return False
        return True
```

## Communication

### Phase 1: SSH + CLI
- SSH into Pi, run commands manually
- `python -m fieldbot paint --template soccer --anchors corners.json`

### Phase 2: Web UI
- Flask/FastAPI server on Pi
- React web app on phone (same WiFi)
- WebSocket for live position streaming

### Phase 3: Phone App (Stretch)
- React Native app
- BLE connection (no WiFi needed in field)
- Offline field design + template library

## Configuration

```yaml
# config.yaml
robot:
  wheel_diameter_m: 0.254    # 10 inches
  wheel_base_m: 0.45         # distance between wheels
  max_speed_mps: 0.4
  
navigation:
  uwb_update_hz: 20
  encoder_ticks_per_rev: 360
  imu_update_hz: 100
  
paint:
  line_width_m: 0.12         # 5 inches
  pump_pressure_psi: 30
  lead_time_ms: 50
  
safety:
  obstacle_distance_cm: 50
  geofence_margin_m: 2.0
  min_battery_v: 15.0

anchors:
  # Set during field setup — positions in meters from origin
  - id: "A1"
    x: -50.0
    y: -32.0
  - id: "A2"
    x: 50.0
    y: -32.0
  - id: "A3"
    x: 50.0
    y: 32.0
  - id: "A4"
    x: -50.0
    y: 32.0
```

## Dependencies

```
# requirements.txt
RPi.GPIO>=0.7.0
smbus2>=0.4.0       # I2C (IMU)
spidev>=3.6         # SPI (UWB)
numpy>=1.24.0       # EKF math
scipy>=1.10.0       # Path interpolation
pyyaml>=6.0         # Config
svgpathtools>=1.6.0 # SVG import
flask>=3.0.0        # Web UI (Phase 2)
websockets>=12.0    # Live streaming (Phase 2)
```
