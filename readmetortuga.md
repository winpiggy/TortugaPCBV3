# 🚀 Tortuga TVC Flight Computer

**A custom flight-computer PCB that actively steers a model rocket by gimballing its motor, logs the flight, and fires its own parachute charge.**

*Made for [Grounded](https://grounded.hackclub.com) by Hack Club.*

```
                 ┌─────────────────────────┐
                 │   POWER ON / SELF-TEST  │
                 │  sensors + SD card OK?  │
                 └────────────┬────────────┘
                              ▼
                 ┌─────────────────────────┐
                 │      PAD (DISARMED)     │
                 │  relay open, logging    │
                 └────────────┬────────────┘
                              │ Bluetooth "ARM"
                              ▼
                 ┌─────────────────────────┐
                 │       PAD (ARMED)       │
                 │ relay closed, LED+buzzer│
                 └────────────┬────────────┘
                              │ launch acceleration detected
                              ▼
                 ┌─────────────────────────┐
                 │     TAKEOFF / ASCENT    │◄──┐
                 │ read IMU → gimbal servos│   │ TVC control loop
                 │ keep rocket vertical    │───┘
                 └────────────┬────────────┘
                              │ altitude stops rising (BMP280)
                              ▼
                 ┌─────────────────────────┐
                 │         APOGEE          │
                 │  MOSFET fires pyro →    │
                 │  parachute deploys      │
                 └────────────┬────────────┘
                              ▼
                 ┌─────────────────────────┐
                 │         DESCENT         │
                 │  servos centered, log   │
                 └────────────┬────────────┘
                              │ altitude steady near ground
                              ▼
                 ┌─────────────────────────┐
                 │         LANDED          │
                 │ close log, buzzer beacon│
                 └─────────────────────────┘
```

---

## What is it?

Most model rockets fly on fins alone and pop their parachute with a fixed motor delay. **Tortuga** is different. It uses **thrust vector control (TVC)**. Like a real orbital rocket, it steers by tilting its motor on a two-axis gimbal, using live sensor data, many times a second.

This PCB is the rocket's brain. It:

1. **Senses** orientation with an MPU6050 IMU (gyro + accelerometer) and altitude with a BMP280 barometer.
2. **Steers** by driving the pitch and roll gimbal servos to keep the rocket pointed straight up.
3. **Tracks the flight** with a state machine: `PAD → TAKEOFF → ASCENT → APOGEE → DESCENT → LANDED`.
4. **Deploys the parachute** by firing an ejection charge through a MOSFET pyro channel when it detects apogee.
5. **Logs everything** to a microSD card, so every flight can be analyzed afterward.

## Why I made it

I started with a breadboard and a tangle of jumper wires. That's fine on a bench, but a rocket shakes, spins and pulls Gs, so loose wires stop working. Things got more serious once I added a pyro channel, because a wiring fault there could set off the ejection charge at the wrong time. I needed a real board: compact, solid, and with the safety circuit built into the copper.

I'm also entering the flight software in a science fair. A clean, repeatable PCB means the hardware stays the same from flight to flight, so my data is actually comparable.

<!-- TODO: add a sentence or two in your own words: what got you into rocketry, what you want to learn -->

## How it works

### Sensing → Control → Actuation

```
 MPU6050 (attitude) ─┐
                     ├─► Teensy 4.1 ──► Pitch servo ─┐
 BMP280 (altitude) ──┘   control loop ──► Roll servo ─┴─► Motor gimbal
                              │
                              ├──► microSD (flight log)
                              ├──► LED + buzzer (status)
                              └──► Pyro channel ──► Parachute
```

The Teensy 4.1 reads the IMU and works out how far the rocket has tilted off vertical. It then commands the gimbal servos to tilt the motor and push the rocket back upright. A software limit caps how far the gimbal can deflect. The barometer tells the flight computer when the rocket has stopped climbing (apogee), and that's when the parachute fires.

### Pyro safety: two locks in series

Firing a pyro charge at the wrong moment is the most dangerous failure on the board, so I designed the circuit with **two independent switches in series**:

```
Battery + ──► Arming relay ──► E-match ──► MOSFET ──► GND
                  ▲                          ▲
         Bluetooth "ARM" cmd        Flight computer "FIRE"
```

- **The arming relay (SRD-05VDC-SL-C)** stays physically open until I arm the rocket over **Bluetooth**, once it's on the pad and everyone is clear. Until then, no current can reach the igniter, even if the MOSFET glitches on.
- **The MOSFET** only turns on when the state machine decides it's time to deploy.
- **The LED and buzzer** confirm the armed state, so you can always tell whether the board is live.

## Hardware

| Function | Part | Interface |
|---|---|---|
| Microcontroller | Teensy 4.1 (600 MHz Cortex-M7) | — |
| IMU | MPU6050 breakout | I²C |
| Barometer | BMP280 breakout | I²C |
| Data logging | Teensy 4.1 onboard microSD | SDIO |
| Gimbal | 2 × micro servo (pitch and roll) | PWM |
| Pyro switch | N-channel MOSFET | GPIO |
| Pyro arming | SRD-05VDC-SL-C relay | GPIO |
| Wireless arming | Bluetooth module | UART |
| Feedback | Status LED + buzzer | GPIO |

**Design constraints:**
- Fits in a **76 mm** airframe avionics bay
- The whole rocket has to stay under about **595 g** on an **Estes F15-4** motor, so every gram on the board counts
- This revision uses breakout boards so I can debug and swap parts easily. A bare-chip revision comes next.

## Screenshots

### Schematic
![Schematic](docs/images/schematic.png)
<!-- TODO: export the schematic as an image -->

### PCB layout
![PCB layout](docs/images/pcb-layout.png)
<!-- TODO: screenshot of the KiCad PCB editor -->

## Firmware

The firmware is written in C++ with PlatformIO in VS Code, using Adafruit libraries for the sensors. It features:
- A flight state machine (pad → takeoff → ascent → apogee → descent → landed)
- microSD data logging
- Bluetooth arming with LED and buzzer confirmation
- An adjustable gimbal deflection limit
- All pin assignments in one `pins.h` file

<!-- TODO: link to the firmware folder or repo -->

## Repository structure

```
tortuga-flight-computer/
├── README.md
├── LICENSE
├── PCB/
│   ├── tortuga.kicad_pro
│   ├── tortuga.kicad_sch
│   └── tortuga.kicad_pcb
├── production/
│   └── gerbers.zip          ← Gerber + drill files for the fab
├── firmware/                ← PlatformIO project
└── docs/images/             ← Screenshots used in this README
```

## Building one

1. Order the PCB from JLCPCB or PCBWay by uploading `production/gerbers.zip`.
2. Solder the headers for the Teensy and the sensor breakouts. Then solder the relay, MOSFET and passives.
3. Flash the firmware with PlatformIO.
4. **Before you connect an igniter,** use a multimeter to check that the pyro output reads open while the board is disarmed. Then test it with an LED in place of the e-match.

> ⚠️ Rocketry involves energetic materials. Follow the NAR/Tripoli safety codes, install the ejection charge last, and arm only from a safe distance.

## Roadmap

- [x] Breadboard prototype
- [x] Flight-computer PCB using breakout boards
- [x] MOSFET pyro channel with relay arming
- [x] Bluetooth arming
- [ ] Fabricate and assemble this board ← **what the Grounded grant pays for**
- [ ] Bench test the TVC loop and pyro channel
- [ ] First powered flight on an Estes F15-4
- [ ] Bare-chip revision (sensors soldered directly to the PCB, smaller and lighter)

## Bill of Materials

| Component | Qty | Price (USD) | Link |
|---|---|---|---|
| Custom PCB (JLCPCB, 5 pcs) | 5 | `TODO` | `TODO` |
| Teensy 4.1 | 1 | `TODO` | `TODO` |
| MPU6050 breakout | 1 | `TODO` | `TODO` |
| BMP280 breakout | 1 | `TODO` | `TODO` |
| Micro servo (gimbal) | 2 | `TODO` | `TODO` |
| SRD-05VDC-SL-C relay | 1 | `TODO` | `TODO` |
| N-channel MOSFET | 1 | `TODO` | `TODO` |
| Bluetooth module | 1 | `TODO` | `TODO` |
| LED + resistor | 1 | `TODO` | `TODO` |
| Buzzer | 1 | `TODO` | `TODO` |
| Headers / connectors | — | `TODO` | `TODO` |
| microSD card | 1 | `TODO` | `TODO` |
| Shipping | — | `TODO` | — |
| **Total** | | **`TODO`** | |

## License

Released under the [MIT License](LICENSE).

---

*Slack: @camemmert*
