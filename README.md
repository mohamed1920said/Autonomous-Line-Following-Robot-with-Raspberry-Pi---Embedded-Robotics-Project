# Autonomous Line-Following Robot with Raspberry Pi

This repository contains an early Python prototype for a two-sensor line-following robot. The intended controller reads left and right digital line sensors, drives two motors through direction outputs, and uses two LEDs to show steering state.

## Important status

`line following.py` is a design draft, not runnable Python in its current form. It contains syntax errors, misspelled GPIO calls, a GPIO pin conflict, and an incomplete shutdown path. Do not connect motors or test the robot on the floor until the issues in [Required corrections](#required-corrections) have been fixed and verified with the wheels raised.

## Intended control logic

The source appears to implement this truth table:

| Left sensor | Right sensor | Intended action | LEDs |
| --- | --- | --- | --- |
| Inactive | Inactive | Drive both motors forward | Both off |
| Active | Inactive | Stop the left motor and drive the right motor | Left on |
| Inactive | Active | Drive the left motor and stop the right motor | Right on |
| Active | Active | Stop both motors | Both on |

The actual meaning of an active/high or inactive/low sensor depends on the line-sensor modules, their sensitivity settings, and whether the line is dark or light. Verify the truth table on the bench before allowing movement.

## Declared BCM pin map

The script selects `GPIO.BCM` numbering.

| Function | BCM GPIO in source | Note |
| --- | ---: | --- |
| Left line sensor (`CapteurG`) | 14 | Digital input |
| Right line sensor (`CapteurD`) | 15 | Digital input, but conflicts with left LED |
| Left LED (`LEDG`) | 15 | Conflicts with the right sensor |
| Right LED (`LEDD`) | 7 | Digital output |
| Left motor input 1 | 18 | Motor-driver control input |
| Left motor input 2 | 23 | Motor-driver control input |
| Right motor input 1 | 8 | Motor-driver control input |
| Right motor input 2 | 1 | Motor-driver control input; verify availability on the selected Pi |

This table records what the current source declares; it is not a validated wiring plan. In particular, BCM GPIO 15 cannot be both the right-sensor input and the left-LED output. BCM GPIO 0/1 are also commonly reserved for Raspberry Pi HAT identification, so using GPIO 1 should be reviewed for the exact board and operating-system configuration.

BCM GPIO 14 and 15 are also the Raspberry Pi's primary UART transmit/receive pins in common configurations. Using them as line-sensor inputs can conflict with the serial console or another UART device. Prefer conflict-free GPIOs, or deliberately disable/remap the UART and verify the boot configuration before wiring the sensors.

## Expected hardware

- Raspberry Pi with a supported GPIO library
- Two digital IR line-sensor modules
- Two DC motors
- A dual H-bridge or two suitable motor drivers
- Two LEDs with current-limiting resistors
- Separate, correctly rated motor supply
- Common signal ground between the Pi and motor driver

Never power a DC motor directly from a Raspberry Pi GPIO pin. GPIO is for logic signals only. Confirm the motor driver's voltage, current, flyback protection, logic levels, and common-ground requirements before wiring.

## Required corrections

At minimum, the following source issues must be resolved:

1. Assign unique GPIO pins to `CapteurD` and `LEDG`.
2. Replace `GPIO.setuo(...)` with `GPIO.setup(...)`.
3. Replace every `GPIO.read(pin)` call with `GPIO.input(pin)`.
4. Repair the missing closing parentheses in both `elif` conditions.
5. Correct the indentation of `else` and the statements below it.
6. Replace `GPIO,output(...)` with `GPIO.output(...)`.
7. Place the main loop inside a `try` block before using `except KeyboardInterrupt`.
8. Define `stop_robot()` or explicitly drive every motor control output low during shutdown.
9. Put `GPIO.cleanup()` in a `finally` block so cleanup also runs after unexpected errors.
10. Confirm the motor direction for each H-bridge input and reverse pairs if necessary.

The fixed program should default to a stopped state, stop on exceptions or signal loss, and avoid a long blocking delay in the control loop. The current `time.sleep(0.8)` after each decision makes steering updates slow for a moving robot.

## Suggested bring-up sequence

1. Leave the motor supply disconnected.
2. Choose and document a conflict-free BCM pin map.
3. Run a small GPIO input test and record the sensor values over the line and background.
4. Test each LED with its resistor.
5. Connect the motor driver with the robot's wheels raised.
6. Verify stop, left motor, right motor, forward direction, and emergency interruption at low power.
7. Run the corrected truth table at a short control interval.
8. Perform the first floor trial at low speed in a clear area with a physical power cutoff available.

## Software setup after correction

On Raspberry Pi OS, use a virtual environment and install the GPIO package appropriate for the specific Pi model and OS image. The current script imports `RPi.GPIO`:

```bash
python3 -m venv .venv
source .venv/bin/activate
pip install RPi.GPIO
python3 "line following.py"
```

`RPi.GPIO` may not be the preferred backend on newer Raspberry Pi hardware. If using Raspberry Pi 5, consider a compatible `gpiozero`/`lgpio` or `gpiod` implementation and adapt the code deliberately rather than assuming identical behavior.

## Project limitations

- No corrected runnable implementation is included.
- No schematic, motor-driver model, power specification, or bill of materials is included.
- Sensor polarity and physical mounting geometry are undocumented.
- Motor speed is not controlled with PWM.
- There is no line-loss recovery, debounce/filtering, watchdog, or automated test.
- There is no dependency file or project license.

Treat this repository as a starting point for an embedded-robotics exercise, not as deployable robot control software.
