# Arduino-MAX30100

[![Build Status](https://travis-ci.org/oxullo/Arduino-MAX30100.svg?branch=master)](https://travis-ci.org/oxullo/Arduino-MAX30100)

Arduino library for the Maxim Integrated MAX30100 oximetry / heart rate sensor.

![MAX30100](http://www.mouser.com/images/microsites/Maxim_MAX30100.jpg)

## Hardware

This library has been tested with the MikroElektronika Heart rate click daughterboard:

http://www.mikroe.com/click/heart-rate/

along with an Arduino UNO r3. Any Arduino supporting the Wire library should work.

The only required connection to the sensor is the I2C bus (SDA, SCL lines, pulled up).

An example which shows a possible way to wire up the sensor is shown in
[extras/arduino-wiring.pdf](extras/arduino-wiring.pdf)

Note: The schematics above shows also how to wire up the interrupt line, which is
currently not used by the library.

### Pull-ups

Since the I2C interface is clocked at 400kHz, make sure that the SDA/SCL lines are pulled
up by 4,7kOhm or less resistors.

## Architecture

The library offers a low-level driver class, MAX30100.
This component allows for low level communication with the device.

A rather simple but working implementation of the heart rate and SpO2 calculation
can be found in the PulseOximeter class.

This high level class sets up the sensor and data processing pipelines in order to
offer a very simple interface to the data:

 * Sampling frequency set to 100Hz
 * 1600uS pulse width, full sampling 16bit dynamic
 * IR LED current set to 50mA
 * Heart-rate + SpO2 mode

The PulseOximeter class is not optimised for battery-based projects.

## Examples

The included examples show how to use the PulseOximeter class:

 * MAX30100_Minimal: a minimal example that dumps human-readable results via serial
 * MAX30100_Debug: used in conjunction with the Processing pde "rolling_graph" (extras folder), to show the sampled data at various processing stages
 * MAX30100_RawData: demonstrates how to access raw data from the sensor
 * MAX30100_Tester: this sketch helps to find out potential issues with the sensor

## Troubleshooting

Run the MAX30100_Tester example to inspect the state of your rig.
When run with a properly connected sensor, it should print:

```
Initializing MAX30100..Success
Enabling HR/SPO2 mode..done.
Configuring LEDs biases to 50mA..done.
Lowering the current to 7.6mA..done.
Shutting down..done.
Resuming normal operation..done.
Sampling die temperature..done, temp=24.94C
All test pass. Press any key to go into sampling loop mode
```

Pressing any key, a data stream with the raw values from the photodiode sampling red
and infrared is presented.
With no finger on the sensor, both values should be close to zero and jump up when
a finger is positioned on top of the sensor.


Typical issues when attempting to run the examples:

### I2C error or garbage data

In particular when the tester fails with:

```
Initializing MAX30100..FAILED: I2C error
```

This is likely to be caused by an improper pullup setup for the I2C lines.
Make sure to use 4,7kOhm resistors, checking if the breakout board in use is equipped
with pullups.

### Sketchy beat frequency readouts

The beat detector uses the IR LED to track the heartbeat. The IR LED is biased
by default at 50mA on all examples, excluding the Tester (which sets it to 7.6mA).
This value is somehow critical and it must be experimented with.

The current can be adjusted using PulseOximeter::setIRLedCurrent().
Check the _MAX30100_Minimal_ example.
