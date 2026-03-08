# FieldBot Mobile App — "Flight Planner for Field Painting"

## Overview

React Native app (iOS + Android) inspired by drone flight planners (Mission Planner, QGroundControl). Design field layouts on a satellite view, overlay templates, tap corners, and let the robot paint it autonomously.

## Screens

### 1. 🏠 Home / Dashboard
- Robot connection status (WiFi or BLE)
- Battery level, paint level estimate
- Quick-start buttons for saved templates
- Recent job history

### 2. 🏟️ Field Designer (Flight Planner Style)
- **Satellite Map View:**
  - Opens to current GPS location (Mapbox/Google Maps tiles)
  - Pinch to zoom, see the actual field from above
  - Tap 4 corners of the real field → defines boundary
  - Satellite imagery shows existing features (goals, fences, etc.)
- **Template Overlay:**
  - Pre-built regulation templates: Soccer (FIFA, MLS, youth), Football (NCAA, NFL, HS), Lacrosse, Field Hockey, Rugby
  - Template overlays on satellite view — drag/rotate/scale to fit
  - Semi-transparent so you can align with real-world features
  - Multi-sport overlay (paint multiple sports on same field)
- **Custom Designer:**
  - Drop lines, arcs, circles, rectangles on the satellite view
  - Import SVG files → auto-scale to field dimensions
  - Set line width per element
  - Mirror/repeat tools
- **Logo Mode:**
  - Import image → trace to paintable paths
  - Center circle logos, end zone text, etc.
- **Save & Share:**
  - Save field layouts with GPS coordinates
  - Share templates with other FieldBot users
  - Re-use same layout next time (robot returns to exact GPS positions)

### 3. 📡 Field Setup (GPS Calibration)
- Walk-through wizard:
  1. "Set up base station on tripod at field edge" → confirm signal
  2. "Walk to corner 1 with phone/robot" → record GPS coord
  3. "Walk to corner 2" → record GPS coord
  4. Repeat for remaining corners
  5. App auto-calculates field dimensions + orientation
  6. "Place robot at home position" → confirm origin
- Live RTK status: No Fix → GPS Fix → Float → **RTK Fix** ✅
- Base station signal strength + satellite count
- NTRIP connection status (if using CORS instead of own base)
- Accuracy indicator: shows current precision in cm

### 4. 🎮 Live Control
- **Map View:**
  - Real-time robot position on field overlay
  - Planned path shown (gray)
  - Completed path shown (green)
  - Current heading indicator
  - Obstacle markers (red dots)
- **Controls:**
  - ▶️ Start / ⏸️ Pause / ⏹️ Stop
  - Manual override joystick (for repositioning)
  - Paint on/off toggle (manual mode)
  - Speed slider (0.1 - 0.5 m/s)
- **Stats Bar:**
  - Progress: 47% complete
  - ETA: 12 min remaining
  - Distance painted: 340m / 720m
  - Paint remaining: ~60%
  - Battery: 72%

### 5. 📊 Job History
- List of completed/paused jobs
- Resume interrupted jobs
- Time, paint used, distance
- Before/after photos (manual upload)

### 6. ⚙️ Settings
- Robot WiFi/BLE pairing
- Unit preferences (meters/feet)
- Paint calibration (flow rate, line width test)
- Motor calibration (wheel diameter, base width)
- UWB anchor positions (manual override)
- Firmware update (OTA to Pi)
- Export/import field templates

## Tech Stack

```
├── React Native (Expo)
│   ├── expo-router          # Navigation
│   ├── react-native-canvas  # Field designer
│   ├── react-native-ble-plx # Bluetooth connection
│   ├── react-native-svg     # Field rendering
│   └── zustand              # State management
│
├── Communication
│   ├── WebSocket (WiFi mode) — real-time position streaming
│   └── BLE (field mode) — no WiFi infrastructure needed
│
└── Robot API (Flask on Pi)
    ├── GET  /status          # Battery, position, state
    ├── GET  /anchors         # UWB anchor status
    ├── POST /job/start       # Upload template + start
    ├── POST /job/pause       # Pause current job
    ├── POST /job/resume      # Resume from last position
    ├── POST /job/stop        # Cancel job
    ├── POST /manual/move     # Joystick commands
    ├── POST /manual/paint    # Paint on/off
    ├── POST /calibrate       # Run calibration routine
    └── WS   /stream          # Live position + telemetry
```

## Communication Protocol

### WiFi Mode (Primary)
- Robot runs Flask server + WebSocket
- Phone connects to robot's WiFi AP or same network
- WebSocket streams position at 10Hz
- REST API for commands

### BLE Mode (Field Fallback)
- No WiFi infrastructure needed in the middle of a field
- BLE range: ~50-100m (enough for most fields)
- Custom GATT services:
  - Control Service: start/stop/pause/manual
  - Telemetry Service: position, battery, progress (notify)
  - Config Service: upload template data

### Message Format (WebSocket)

```json
// Robot → App (10Hz telemetry)
{
  "type": "telemetry",
  "ts": 1709901234.567,
  "pos": { "x": 12.45, "y": -8.32 },
  "heading": 1.57,
  "speed": 0.35,
  "battery_v": 18.2,
  "painting": true,
  "progress": 0.47,
  "obstacles": [],
  "state": "running"
}

// App → Robot (commands)
{
  "type": "command",
  "action": "start",
  "template": { ... },
  "speed": 0.4
}

// App → Robot (manual joystick)
{
  "type": "manual",
  "left": 0.8,    // -1.0 to 1.0
  "right": 0.6,
  "paint": false
}
```

## Field Designer Details

### Canvas Interaction
- **Pinch to zoom** on the field
- **Tap** to place points
- **Drag** to draw lines/move elements
- **Long press** to select/edit/delete
- **Two-finger rotate** to orient field

### Snapping & Guides
- Snap to grid (configurable: 1m, 0.5m, 0.1m)
- Snap to regulation dimensions (auto-suggest)
- Perpendicular/parallel guides
- Symmetry mode (mirror across center line)

### Template Format

```json
{
  "name": "Soccer - FIFA Regulation",
  "version": 1,
  "dimensions": { "length": 100, "width": 64 },
  "unit": "meters",
  "line_width": 0.12,
  "elements": [
    {
      "type": "rectangle",
      "id": "perimeter",
      "x": 0, "y": 0,
      "width": 100, "height": 64
    },
    {
      "type": "line",
      "id": "center_line",
      "x1": 50, "y1": 0,
      "x2": 50, "y2": 64
    },
    {
      "type": "circle",
      "id": "center_circle",
      "cx": 50, "cy": 32,
      "radius": 9.15
    },
    {
      "type": "rectangle",
      "id": "penalty_area_left",
      "x": 0, "y": 13.84,
      "width": 16.5, "height": 40.32
    }
  ]
}
```

## App Folder Structure

```
app/
├── src/
│   ├── screens/
│   │   ├── HomeScreen.tsx
│   │   ├── DesignerScreen.tsx
│   │   ├── SetupScreen.tsx
│   │   ├── LiveControlScreen.tsx
│   │   ├── HistoryScreen.tsx
│   │   └── SettingsScreen.tsx
│   ├── components/
│   │   ├── FieldCanvas.tsx        # SVG field renderer
│   │   ├── Joystick.tsx           # Manual control
│   │   ├── AnchorStatus.tsx       # UWB anchor cards
│   │   ├── TelemetryBar.tsx       # Stats display
│   │   ├── TemplateCard.tsx       # Template selector
│   │   └── RobotIndicator.tsx     # Position dot + heading
│   ├── services/
│   │   ├── bluetooth.ts           # BLE connection manager
│   │   ├── websocket.ts           # WiFi WebSocket client
│   │   ├── robotApi.ts            # REST API wrapper
│   │   └── templateStore.ts       # Template CRUD
│   ├── hooks/
│   │   ├── useRobotConnection.ts  # Auto-connect WiFi/BLE
│   │   ├── useTelemetry.ts        # Live position state
│   │   └── useFieldDesigner.ts    # Canvas interaction logic
│   ├── templates/
│   │   ├── soccer-fifa.json
│   │   ├── soccer-youth.json
│   │   ├── football-ncaa.json
│   │   └── football-hs.json
│   └── utils/
│       ├── pathOptimizer.ts       # TSP waypoint ordering
│       ├── svgImporter.ts         # SVG → template converter
│       └── units.ts               # Meters ↔ feet conversion
├── app.json
├── package.json
└── tsconfig.json
```

## Future Features (v2+)
- **Multi-robot support** — paint different sections simultaneously
- **Photo overlay** — drone/satellite image of field, align template on top
- **Schedule jobs** — "Repaint every Tuesday at 6 AM"
- **Wear tracking** — AI estimates when lines need repainting based on usage
- **Team branding** — template marketplace with team logos/colors
- **Weather integration** — warn if rain expected within 2 hours
