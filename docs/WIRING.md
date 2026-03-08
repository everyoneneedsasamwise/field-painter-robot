# Wiring Guide

## Raspberry Pi 4 GPIO Pinout

```
                    3V3 [1 ] [2 ] 5V
          IMU SDA → GP2 [3 ] [4 ] 5V
          IMU SCL → GP3 [5 ] [6 ] GND
                    GP4 [7 ] [8 ] GP14
                    GND [9 ] [10] GP15
   Encoder L (A) → GP17[11] [12] GP18 ← Motor L FWD (L298N IN1)
   Encoder L (B) → GP27[13] [14] GND
   Encoder R (A) → GP22[15] [16] GP23 ← Motor L REV (L298N IN2)
                   3V3 [17] [18] GP24 ← Motor R FWD (L298N IN3)
       UWB MOSI → GP10[19] [20] GND
       UWB MISO → GP9 [21] [22] GP25 ← Motor R REV (L298N IN4)
       UWB SCLK → GP11[23] [24] GP8  ← UWB CS (CE0)
                    GND [25] [26] GP7
                    GP0 [27] [28] GP1
   Encoder R (B) → GP5 [29] [30] GND
   Solenoid Valve→ GP6 [31] [32] GP12 ← Motor L PWM (L298N ENA)
     Paint Pump  → GP13[33] [34] GND
  Ultrasonic L Trig→GP19[35] [36] GP16 ← Ultrasonic L Echo
  Ultrasonic R Trig→GP26[37] [38] GP20 ← Ultrasonic R Echo
                    GND [39] [40] GP21 ← Bumper Switch
```

## Power Distribution

```
 DeWalt 20V Battery
        │
        ▼
 ┌──────────────┐
 │ Adapter Plate│
 └──────┬───────┘
        │ 20V
        ├───────────────────────────┐
        ▼                           ▼
 ┌──────────────┐           ┌──────────────┐
 │  Buck Conv.  │           │  Buck Conv.  │
 │  20V → 12V  │           │  20V → 5V   │
 └──────┬───────┘           └──────┬───────┘
        │ 12V                      │ 5V
        ├────────┐                 ├──── Raspberry Pi (USB-C)
        ├────────┤                 ├──── UWB Module
        ├────────┤                 ├──── IMU
        │        │                 ├──── Ultrasonic Sensors
        ▼        ▼                 └──── Encoder Logic
    L298N    Solenoid
    Motor    + Pump
    Driver   (via relay/MOSFET)
```

## Component Wiring Details

### L298N Motor Driver

```
L298N           →  Connection
─────────────────────────────
12V             →  12V from buck converter
GND             →  Common ground
5V (output)     →  Can power small stuff (regulated from 12V)
IN1             →  Pi GP18 (Motor L Forward)
IN2             →  Pi GP23 (Motor L Reverse)
IN3             →  Pi GP24 (Motor R Forward)
IN4             →  Pi GP25 (Motor R Reverse)
ENA             →  Pi GP12 (PWM - Left speed)
ENB             →  Pi GP13 (PWM - Right speed) *see note
OUT1            →  Left Motor +
OUT2            →  Left Motor -
OUT3            →  Right Motor +
OUT4            →  Right Motor -
```

*Note: GP13 is shared with pump. Use GP19 for ENB if pump uses GP13, or use a separate GPIO for pump via MOSFET.*

### MPU-6050 IMU (I2C)

```
MPU-6050    →  Pi
───────────────────
VCC         →  3.3V
GND         →  GND
SDA         →  GP2 (SDA1)
SCL         →  GP3 (SCL1)
```

### UWB DWM3000 Tag (SPI)

```
DWM3000     →  Pi
───────────────────
VCC         →  3.3V
GND         →  GND
MOSI        →  GP10 (SPI0_MOSI)
MISO        →  GP9 (SPI0_MISO)
SCLK        →  GP11 (SPI0_SCLK)
CS          →  GP8 (SPI0_CE0)
IRQ         →  GP7 (optional interrupt)
RST         →  GP1 (optional reset)
```

### Hall Effect Wheel Encoders

```
Left Encoder   →  Pi
───────────────────────
VCC            →  5V
GND            →  GND
Signal A       →  GP17
Signal B       →  GP27 (for direction)

Right Encoder  →  Pi
───────────────────────
VCC            →  5V
GND            →  GND
Signal A       →  GP22
Signal B       →  GP5 (for direction)
```

### HC-SR04 Ultrasonic Sensors

```
Left HC-SR04   →  Pi
───────────────────────
VCC            →  5V
GND            →  GND
TRIG           →  GP19
ECHO           →  GP16 (use voltage divider! 5V→3.3V)

Right HC-SR04  →  Pi
───────────────────────
VCC            →  5V
GND            →  GND
TRIG           →  GP26
ECHO           →  GP20 (use voltage divider! 5V→3.3V)
```

⚠️ **IMPORTANT:** HC-SR04 Echo pin outputs 5V. Pi GPIO is 3.3V only! Use a voltage divider (1kΩ + 2kΩ) on each Echo line.

### Paint Solenoid + Pump

```
Solenoid Valve (12V)
───────────────────────
Use N-channel MOSFET (IRLZ44N) or relay module:
  Pi GP6 → MOSFET Gate (via 1kΩ resistor)
  MOSFET Source → GND
  MOSFET Drain → Solenoid (-)
  Solenoid (+) → 12V
  Flyback diode across solenoid (1N4007)

Paint Pump (12V) — same pattern:
  Pi GP13 → MOSFET Gate
  MOSFET controls pump power
  Flyback diode across pump
```

### Bumper Switch

```
Pi GP21 ← Switch ← GND
(Enable internal pull-up on GP21)
Pressed = LOW = obstacle contact
```

## Grounding

**All grounds must be connected together:**
- Pi GND
- L298N GND
- Buck converter GNDs
- Sensor GNDs
- Battery GND

Use a common ground bus bar or terminal strip.

## Connector Recommendations

| Connection | Connector Type |
|------------|---------------|
| Battery → buck converters | XT60 |
| Motors | Spade terminals or solder |
| Sensors | JST-XH or DuPont headers |
| Pi → driver board | DuPont jumper wires |
| Paint system | Quick-disconnect push fittings |

## Assembly Tips

1. Wire everything on a breadboard first — test before soldering
2. Label every wire (masking tape + Sharpie works)
3. Use ferrules on stranded wire going into screw terminals
4. Keep high-current (motor/pump) wires away from signal wires
5. Secure all connections — vibration will shake loose anything not locked down
