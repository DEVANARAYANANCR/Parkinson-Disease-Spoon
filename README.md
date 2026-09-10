# Active Stabilising Spoon for Parkinson's Disease

A low cost, Arduino based active stabilising spoon that detects involuntary hand
tremors (typically 4–7 Hz in Parkinson's disease) and drives a two-axis servo
mechanism in real time to cancel them out, while still letting the user make
deliberate movements, like tilting the spoon to scoop food.



Built as a mini project at the Division of Electronics Engineering, School of
Engineering, Cochin University of Science and Technology (CUSAT), Kochi.

## Media



## 🎥 Device Demonstration

<p align="center">
<b>Active Stabilising Spoon in Operation</b>
</p>

https://github.com/user-attachments/assets/4f2f02c5-af17-4c0e-89d2-98a593eb1e00



## How it works

1. An **MPU6050** IMU (accelerometer + gyroscope) reports pitch/roll angles over I²C.
2. Raw angles are smoothed with a moving average filter.
3. A **complementary low pass filter** splits the smoothed signal into intent and tremor components.
4. A **PD controller** (proportional + derivative) computes a corrective angle from the isolated tremor signal.
5. Two **servo motors** (pitch + roll axis) apply the correction to keep the spoon head level.
6. A **push button** (hold ≥700 ms) triggers a pre programmed scoop sequence, then hands control back to stabilisation.
7. The RGB LED shows state: white = calibrating, green = stabilising, blue = scoop mode.

## Hardware

| Component            | Notes                                  |
|-----------------------|-----------------------------------------|
| Arduino Nano          | ATmega328P, central controller          |
| MPU6050               | 6-DOF IMU, I²C                          |
| SG90 micro servo × 2  | Pitch + roll axis actuation             |
| TP4056 module         | Li-ion charging                         |
| DC-DC buck converter  | Regulates battery voltage to a stable rail |
| Li-ion cell           | Portable power source                   |
| Push button            | Scoop-mode trigger                      |
| RGB LED               | Status indicator                        |

Approximate total component cost: **~₹1100**.

### Pinout

| Pin | Function              |
|-----|------------------------|
| A4  | MPU6050 SDA            |
| A5  | MPU6050 SCL            |
| D9  | Servo 1 (pitch axis)   |
| D10 | Servo 2 (roll axis)    |
| D2  | Push button (INPUT_PULLUP, active LOW) |
| D3  | RGB LED: Red           |
| D4  | RGB LED: Green         |
| D5  | RGB LED: Blue          |

## Firmware

Firmware lives in [`code/stabilising_spoon.ino`](code/stabilising_spoon.ino).

### Requirements

1. [Arduino IDE](https://www.arduino.cc/) (or Arduino CLI)
2. Board: Arduino Nano (ATmega328P, old or new bootloader depending on your board)
3. Library: [`MPU6050_tockn`](https://github.com/tockn/MPU6050_tockn). Install via **Library Manager → search "MPU6050_tockn"**
4. Built in `Wire.h` and `Servo.h` (ship with the IDE)

### Flashing

1. Open `code/stabilising_spoon.ino` in the Arduino IDE.
2. Select **Tools → Board → Arduino Nano**, and the correct **Processor** (Old Bootloader if uploads fail on a clone board).
3. Select the correct **Port**.
4. Upload.
5. Open the Serial Monitor at **115200 baud** to watch calibration and live tremor/correction data.

### First-time calibration of your unit

Servo horns do not seat at the exact same spline position across builds, so the two
constants at the top of the firmware, `S1_CENTER` and `S2_CENTER`, need to match
**your** physical assembly:

1. On power-up, the firmware immediately drives both servos to `S1_CENTER`/`S2_CENTER` before anything else runs, so the spoon should hold a fixed position from the instant it powers on.
2. If the spoon head isn't level at that position, adjust `S1_CENTER`/`S2_CENTER` in the firmware and re-upload until it is.
3. Leave the spoon still for ~5 seconds after power on. This is the gyro calibration + settle window (white LED).
4. Once the LED turns green, stabilisation is active.

### Tuning parameters

| Constant     | Purpose                                                                 |
|--------------|--------------------------------------------------------------------------|
| `GAIN`       | Proportional gain on the isolated tremor signal. Too high → oscillation/overshoot. |
| `KD`         | Derivative gain (damping). Too high → jittery response.                 |
| `DEADBAND`   | Ignores tremor readings below this magnitude (sensor noise floor).      |
| `LP_ALPHA`   | Cutoff of the intent/tremor separation filter (~1 Hz by default). Lower = slower motions are still treated as "intent". |
| `AVG_N`      | Moving-average window size for raw sensor smoothing.                     |
| `MAX_OFFSET` | Safety limit: servo will never move more than this many degrees from center. |

Tune `GAIN` first with `KD` at 0, increasing until the response is fast but not
oscillating, then raise `KD` slightly to damp any remaining overshoot.

## Repository structure

```
.
├── code/
│   └── stabilising_spoon.ino   # Arduino Nano firmware
├── images/
│   ├── Circuit_board.jpeg
│   ├── Device_operation_video.mp4
│   └── Device_top_view.jpeg
└── README.md
```

## Future scope

1. Bluetooth logging of tremor frequency/amplitude for clinician review
2. Interchangeable utensil heads (fork, deeper soup spoon)
3. Per user adaptive tuning of `GAIN`/`KD`/`LP_ALPHA`
4. Waterproof, dishwasher safe enclosure
5. Wireless charging

## Team

Dania Abdulla, Deeraj P Menon, Devadath K, Devanarayanan C R,
B.Tech Electronics and Communication Engineering, CUSAT.
