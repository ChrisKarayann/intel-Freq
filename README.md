# intel-Freq

**Power generator frequency metering & manipulation device built on ESP8266 / Arduino**

---

## Overview

intel-Freq is an embedded system that continuously measures the AC mains frequency of a power generator and displays it on a 16×2 I²C LCD screen.  
When the measured frequency drifts outside the acceptable 49–51 Hz window the display immediately shows an **ERROR** banner so the operator can take corrective action.  
A servo motor is coupled to the generator's throttle or governor: the operator can command the servo **up** or **down** from two push-buttons, or switch the system into **Auto** mode and let the firmware hold the throttle at the last position.

---

## Features

| Feature | Details |
|---|---|
| Frequency measurement | Reads HIGH and LOW pulse widths with `pulseIn()` and derives frequency from the measured period |
| Accuracy | Applies a small empirical correction factor (`low_t × 0.0101`) to compensate for asymmetric signal timing |
| Acceptable band | 49 Hz – 51 Hz (standard 50 Hz grid tolerance ±1 Hz) |
| Out-of-range alert | Full-screen **ERROR** message plus live frequency readout |
| I²C LCD output | 16×2 display shows frequency (3 decimal places), control mode, and motor direction |
| Manual / Auto mode | Toggle switch selects whether button inputs are active (**CTRL:M**) or locked (**CTRL:A**) |
| Servo motor control | Three states — rotate UP (`+`), rotate DOWN (`-`), hold neutral (`\|`) |
| Serial debug output | High/Low pulse widths, switch states and button states printed each cycle |

---

## Hardware Requirements

- **Microcontroller** – ESP8266 or compatible Arduino board (the firmware uses standard Arduino API)
- **Frequency input signal** – Square wave derived from the generator's AC output (e.g. via an optocoupler / voltage divider to bring the signal to 3.3 V / 5 V logic)
- **16×2 I²C LCD** – I²C address `0x3F`, controlled via the `LiquidCrystal_I2C` library
- **Servo motor** – Standard hobby servo (signal wire on pin 7)
- **Push-buttons** (×2) – UP and DOWN, wired to GND (internal pull-ups used)
- **Toggle switch** – Auto / Manual selector, wired to GND (internal pull-up used)

---

## Pin Assignment

| Pin | Direction | Description |
|-----|-----------|-------------|
| **8** | INPUT | Frequency signal input (square wave from generator) |
| **9** | INPUT_PULLUP | Auto / Manual toggle switch |
| **4** | INPUT_PULLUP | Motor UP button |
| **2** | INPUT_PULLUP | Motor DOWN button |
| **7** | OUTPUT | Servo motor PWM signal |
| SDA / SCL | I²C | LCD display (hardware I²C pins of the board) |

> All button/switch inputs are **active-LOW**: connect one terminal to the pin and the other to **GND**.

---

## Libraries

Install the following libraries through the **Arduino Library Manager** (or your preferred method) before compiling:

| Library | Purpose |
|---|---|
| `Wire` | I²C communication (built-in) |
| `LiquidCrystal_I2C` | 16×2 I²C LCD driver |
| `Servo` | PWM servo control (built-in) |

---

## How It Works

### Frequency Measurement

```
high_t  = pulseIn(freq_input, HIGH);   // µs the signal is HIGH
low_t   = pulseIn(freq_input, LOW);    // µs the signal is LOW

t_period = (high_t + low_t + low_t * 0.0101) / 1000;   // period in ms
frequency = 1000 / t_period;                             // frequency in Hz
```

The small `0.0101` factor is an empirical correction for timing asymmetry in the measurement circuit.

### LCD Display Layout

**Normal operation (49–51 Hz)**

```
FREQ: 50.012 Hz
CTRL:M  MOTOR: |
```

**Out-of-range**

```
ERROR  FREQ: 48.7
```

### Control Modes

| Mode | LCD label | Behaviour |
|------|-----------|-----------|
| Manual | `CTRL:M` | UP/DOWN buttons actively command the servo |
| Auto | `CTRL:A` | Buttons are read but mode indication changes; servo holds position |

### Servo Positions

| Button pressed | Servo angle | LCD symbol |
|---|---|---|
| UP button (pin 4 = LOW) | 108° | `+` |
| DOWN button (pin 2 = LOW) | 72° | `-` |
| Neither | 90° (neutral) | `\|` |

### Update Rate

The main loop delays **1000 ms** (`timeD`) between readings, giving a 1 Hz display refresh rate.

---

## Getting Started

1. **Wire** the hardware according to the pin table above.
2. **Install** the required libraries.
3. **Open** `esp8266-code` (the Arduino IDE requires the sketch filename to match its parent folder — rename the file to `intel-Freq.ino` and place it inside an `intel-Freq/` folder).
4. **Select** your board (e.g. *NodeMCU 1.0* or the appropriate Arduino variant) and COM port.
5. **Upload** the sketch.
6. **Open** the Serial Monitor at the default baud rate to view debug output (pulse widths, switch and button states).

---

## Project Files

| File | Description |
|---|---|
| `esp8266-code` | Main Arduino sketch (`InteliFreq.ino`) |
| `inteli freQ.pdf` | Project documentation / schematic |
| `README.md` | This file |

---

## License

This project is provided as-is for educational and personal use.  
Author: **chris** — created 2018-01-20.

