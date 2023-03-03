- [Library Information](#orga36a3e1)
- [Stepper Motors](#orgc412be3)
- [Stepper Motor Controllers and Drivers](#org99351fd)
- [Communication](#org6b5c024)
- [Settings](#org25e79a9)
- [Examples](#org9c732dc)
- [Hardware Documentation](#orgb1ac219)

    <!-- This file is generated automatically from metadata -->
    <!-- File edits may be overwritten! -->


<a id="orga36a3e1"></a>

# Library Information

-   **Name:** TMC2209
-   **Version:** 8.0.7
-   **License:** BSD
-   **URL:** <https://github.com/janelia-arduino/TMC2209>
-   **Author:** Peter Polidoro
-   **Email:** peter@polidoro.io


## Description

The TMC2209 is an ultra-silent motor driver IC for two phase stepper motors with both UART serial and step and direction interfaces.

![img](./images/TMC2209.png)


<a id="orgc412be3"></a>

# Stepper Motors

From Wikipedia, the free encyclopedia:

A stepper motor, also known as step motor or stepping motor, is a brushless DC electric motor that divides a full rotation into a number of equal steps. The motor's position can be commanded to move and hold at one of these steps without any position sensor for feedback (an open-loop controller), as long as the motor is correctly sized to the application in respect to torque and speed.

[Wikipedia - Stepper Motor](https://en.wikipedia.org/wiki/Stepper_motor)


<a id="org99351fd"></a>

# Stepper Motor Controllers and Drivers

Stepper motors need both a controller and a driver. These may be combined into a single component or separated into multiple components that communicate with each other, as is the case with the TMC2209 stepper motor driver. One controller may be connected to more than one driver for coordinated multi-axis motion control.


## Stepper Motor Controller

A stepper motor controller is responsible for the commanding either the motor kinetics, the torque, or the motor kinematics, the position, speed, and acceleration of one or more stepper motors.


## Stepper Motor Driver

A stepper motor driver is responsible for commanding the electrical current through the motor coils as it changes with time to meet the requirements of the stepper motor controller.


## TMC2209 Stepper Motor Driver and Controller Combination

The TMC2209 is a stepper motor driver and it needs a stepper motor controller communicating with it.

The TMC2209 can be used as both a stepper motor driver and stepper motor controller combined, independent from a separate stepper motor controller, but it is limited to simple velocity control mode only, with no direct position or acceleration control.

In velocity control mode, the velocity of the motor is set by the method moveAtVelocity(int32\_t microsteps\_per\_period). The actual magnitude of the velocity depends on the TMC2209 clock frequency. The TMC2209 clock frequency (fclk) is normally 12 MHz if the internal clock is used, but can be between 4 and 16 MHz if an external clock is used.

In general: microsteps\_per\_second = microsteps\_per\_period \* (fclk Hz / 2^24)

Using internal 12 MHz clock (default): microsteps\_per\_second = microsteps\_per\_period \* 0.715 Hz

Crude position control can be performed in this simple velocity control mode by commanding the driver to move the motor at a velocity, then after a given amount of time commanding it to stop, but small delays in the system will cause position errors. Plus without acceleration control, the stepper motor may also slip when it attempts to jump to a new velocity value causing more position errors. For some applications, these position errors may not matter, making simple velocity control good enough to save the trouble and expense of adding a separate stepper controller.

Most of this library's examples use the simple velocity control mode to test the driver independently from a separate stepper motor controller, however in most real world applications a separate motor controller is needed, along with the TMC2209 and this library, for position and acceleration control.


### Microcontroller Stepper Motor Controller

One controller option is to use just a single microcontroller, communicating with the TMC2209 over both the UART serial interface and the step and direction interface.

![img](./images/microcontroller_controller_driver.png)


### TMC429 and Microcontroller Stepper Motor Controller

Another controller option is to use both a microcontroller and a separate step and direction controller, such as the TMC429.

![img](./images/TMC429_controller_driver.png)


<a id="org6b5c024"></a>

# Communication

The TMC2209 driver has two interfaces to communicate with a stepper motor controller, a UART serial interface and a step and direction interface.

The UART serial interface may be used for tuning and control options, for diagnostics, and for simple velocity commands.

The step and direction interface may be used for real-time position, velocity, and acceleration commands. The step and direction signals may be synchronized with the step and direction signals to other TMC2209 chips for coordinated multi-axis motion.


## UART Serial Interface

[Wikipedia - UART](https://en.wikipedia.org/wiki/Universal_asynchronous_receiver-transmitter)

The TMC2209 communicates over a UART serial port using a single wire interface, allowing bi-directional operation for full control and diagnostics. It can be driven by any standard microcontroller UART or even by bit banging in software.

The UART single wire interface allows control of the TMC2209 with any set of microcontroller UART serial TX and RX pins. The single serial signal is connected to both the TX pin and the RX pin, with a 1k resistor between the TX pin and the RX pin to separate them.

The microcontroller serial port must be specified during the TMC2209 setup.

For example:

```cpp

#include <Arduino.h>
#include <TMC2209.h>

// Instantiate TMC2209
TMC2209 stepper_driver;
HardwareSerial & serial_stream = Serial1;

void setup()
{
  stepper_driver.setup(serial_stream);
}

```

![img](./images/TMC2209_serial.png)


### Arduino Serial

[Arduino Serial Web Page](https://www.arduino.cc/reference/en/language/functions/communication/serial)

On some Arduino boards (e.g. Uno, Nano, Mini, and Mega) pins 0 and 1 are used for communication with the computer on the serial port named "Serial". Pins 0 and 1 cannot be used on these boards to communicate with the TMC2209. Connecting anything to these pins can interfere with that communication, including causing failed uploads to the board.

Arduino boards with additional serial ports, such as "Serial1" and "Serial2", can use those ports to communicate with the TMC2209.


### Teeny Serial

[Teensy Serial Web Page](https://www.pjrc.com/teensy/td_uart.html)

The Teensy boards have 1 to 8 hardware serial ports (Serial1 - Serial8), which may be used to connect to serial devices.

Unlike Arduino boards, the Teensy USB serial interface is not connected to pins 0 and 1, allowing pins 0 and 1 to be used to communicate with a TMC2209 using "Serial1".


### Serial Baud Rate

The serial baud rate is the speed of communication in bits per second of the UART serial port connected to the TMC2209.

In theory, baud rates from 9600 Baud to 500000 Baud or even higher (when using an external clock) may be used. No baud rate configuration on the chip is required, as the TMC2209 automatically adapts to the baud rate. In practice, it was found that the baud rate may range from 19200 to 500000 without errors.

The higher the baud rate the better, but microcontrollers have various UART serial abilities and limitations which affects the maximum baud allowed. The baud rate may be specified when setting up the stepper driver.

1.  Arduino

    The maximum serial baud rate on typical Arduino boards is 115200, so that is the default, but other values as low as 19200 may be used.
    
    [Arduino Serial Baud Rate Web Page](https://www.arduino.cc/en/Reference/SoftwareSerialBegin)

2.  Teensy

    Teensy UART baud rates can go higher than many typical Arduino boards, so 500k is a good setting to use, but other values as low as 19200 may be used.
    
    [Teensy Serial Baud Rate Web Page](https://www.pjrc.com/teensy/td_uart.html)
    
    ```cpp
    
    #include <Arduino.h>
    #include <TMC2209.h>
    
    // Instantiate TMC2209
    TMC2209 stepper_driver;
    HardwareSerial & serial_stream = Serial1;
    const long SERIAL1_BAUD_RATE = 500000;
    
    void setup()
    {
      stepper_driver.setup(Serial1,SERIAL1_BAUD_RATE);
    }
    
    ```


### Serial Addresses

More than one TMC2209 may be connected to a single serial port, if each TMC2209 is assigned a unique serial address. The default serial address is "SERIAL\_ADDRESS\_0". The serial address may be changed from "SERIAL\_ADDRESS\_0" using the TMC2209 hardware input pins MS1 and MS2, to "SERIAL\_ADDRESS\_1", "SERIAL\_ADDRESS\_2", or "SERIAL\_ADDRESS\_3".

The TMC2209 serial address must be specified during the TMC2209 setup, if it is not equal to the default of "SERIAL\_ADDRESS\_0".

For example:

```cpp

#include <Arduino.h>
#include <TMC2209.h>

// Instantiate the two TMC2209
TMC2209 stepper_driver_0;
const TMC2209::SerialAddress SERIAL_ADDRESS_0 = TMC2209::SERIAL_ADDRESS_0;
TMC2209 stepper_driver_1;
const TMC2209::SerialAddress SERIAL_ADDRESS_1 = TMC2209::SERIAL_ADDRESS_1;
const long SERIALX_BAUD_RATE = 115200;

void setup()
{
  // TMC2209::SERIAL_ADDRESS_0 is used by default if not specified
  stepper_driver_0.setup(Serial1,SERIALX_BAUD_RATE,SERIAL_ADDRESS_0);
  stepper_driver_1.setup(Serial1,SERIALX_BAUD_RATE,SERIAL_ADDRESS_1);
}

```

![img](./images/TMC2209_serial_address.png)


## Step and Direction Interface


### Microcontroller Stepper Motor Controller

The step and direction signals may be output from a microcontroller, using one output pin for the step signal and another output pin for the direction signal.


### TMC429 and Microcontroller Stepper Motor Controller

The step and direction signals may be output from a dedicated step and direction controller, such as the TMC429.

A library such as the Arduino TMC429 library may be used to control the step and direction output signals.

[Arduino TMC429 Library](https://github.com/janelia-arduino/TMC429)


<a id="org25e79a9"></a>

# Settings

The default settings for this library are not the same as the default settings for the TMC2209 chip during power up.

The default settings for this library were chosen to be as conservative as possible so that motors can be attached to the chip without worry that they will accidentally overheat from too much current before library settings can be changed.

These default settings may cause this library to not work properly with a particular motor until the settings are changed.

The driver starts off with the outputs disabled, with the motor current minimized, with analog current scaling disabled, and both the automatic current scaling and automatic gradient adaptation disabled, and the cool step feature disabled.

Change driver settings with care as they may cause the motors, wires, or driver to overheat and be damaged when the current is too high. The driver tends to protect itself and shutdown when it overheats, then reenable when the driver cools, which can result in odd jerky motor motion.


## Analog Current Scaling

Analog current scaling is disabled in this library by default, so a potentiometer connected to VREF will not set the current limit of the driver. Current settings are controlled by UART commands instead.

Use enableAnalogCurrentScaling() to allow VREF, the analog input of the driver, to be used for current control.

According to the datasheet, modifying VREF or the supply voltage invalidates the result of the automatic tuning process. So take care when attempting to use analog current scaling with automatic tuning at the same time.


## Automatic Tuning

The TMC2209 can operate in one of two modes, the voltage control mode and the current control mode.

In both modes, the driver uses PWM (pulse width modulation) to set the voltage on the motor coils, which then determines how much current flows through the coils. In voltage control mode, the driver sets the PWM based only on the driver settings and the velocity of the motor. In current control mode, the driver uses driver settings and the velocity to set the PWM as well, but in addition it also measures the current through the coils and adjusts the PWM automatically in order to maintain the proper current levels in the coils.

Voltage control mode is the default of this library.

The datasheet recommends using current control mode unless the motor type, the supply voltage, and the motor load, the operating conditions, are well known. This library uses voltage control mode by default, though, because there seem to be cases when the driver is unable to calibrate the motor properties properly and that can cause the motor to overheat before the settings are adjusted.

The datasheet explains how to make sure the driver performs the proper automatic tuning routine in order to use current control mode.

Use enableAutomaticCurrentScaling() to switch to current control mode instead.


### Voltage Control Mode

Use disableAutomaticCurrentScaling() to switch to voltage control mode and disable automatic tuning.

When automatic current scaling is disabled, the driver operates in a feed forward velocity-controlled mode and will not react to a change of the supply voltage or to events like a motor stall, but it provides a very stable amplitude.

When automatic tuning is disabled, the run current and hold current settings are not enforced by regulation but scale the PWM amplitude only. When automatic tuning is disabled, the PWM offset and PWM gradient values may need to be set manually in order to adjust the motor current to proper levels.


### Current Control Mode

Use enableAutomaticCurrentScaling() to switch to current control mode and enable automatic tuning.

Use enableAutomaticGradientAdaptation() when in current control mode to allow the driver to automatically adjust the pwm gradient value.

When the driver is in current control mode it measures the current and uses that feedback to automatically adjust the voltage when the velocity, voltage supply, or load on the motor changes. In order to respond properly to the current feedback, the driver must perform a calibration routine, an automatic tuning procedure, to measure the motor properties. This allows high motor dynamics and supports powering down the motor to very low currents.

Refer to the datasheet to see how to make the driver perform the automatic tuning procedure properly.


### PWM Offset

The PWM offset relates the motor current to the motor voltage when the motor is at standstill.

Use setPwmOffset(pwm\_amplitude) to change. pwm\_amplitude range: 0-255

In voltage control mode, increase the PWM offset to increase the motor current.

In current control mode, the pwm offset value is used for initialization only. The driver will calculate the pwm offset value automatically.


### PWM Gradient

The PWM gradient adjusts the relationship between the motor current to the motor voltage to compensate for the velocity-dependent motor back-EMF.

Use setPwmGradient(pwm\_amplitude) to change. pwm\_amplitude range: 0-255

In voltage control mode, increase the PWM gradient to increase the motor current if it decreases too much when the motor increases velocity.

In current control mode, the pwm gradient value is used for initialization only. The driver will calculate the pwm gradient value automatically.


### Run Current

The run current is used to scale the spinning motor current.

Use setRunCurrent(percent) to change. percent range: 0-100

In voltage control mode, the run current scales the PWM amplitude, but the current setting is not measured and adjusted when changes to the operating conditions occur. Use the PWM offset, the PWM gradient, and the run current all three to adjust the motor current.

In current control mode, setting the run current is the way to adjust the spinning motor current. The driver will measure the current and automatically adjust the voltage to maintain the run current, even with the operating conditions change. The PWM offset and the PWM gradient may be changed to help the automatic tuning procedure, but changing the run current alone is enough to adjust the motor current since the driver will adjust the offset and gradient automatically.


### Standstill Mode

The standstill mode determines how the motor will behave when the driver is commanded to be at zero velocity.

Use setStandstillMode(mode) to change. mode values: NORMAL, FREEWHEELING, STRONG\_BRAKING, BRAKING

1.  NORMAL

    In NORMAL mode, the driver actively holds the motor still using the hold current setting to scale the motor current.

2.  FREEWHEELING

    In FREEWHEELING mode, the motor is free to spin freely when the driver is set to zero velocity.

3.  STRONG\_BRAKING and BRAKING

    When the mode is either BRAKING, or STRONG\_BRAKING, the motor coils will be shorted inside the driver so the motor will tend to stay in one place even though current is not actively being driven into the coils.


### Hold Current

The hold current is used to scale the standstill motor current, based on the standstill mode and the hold delay settings.

Use setHoldCurrent(percent) to change. percent range: 0-100

In voltage control mode, the hold current scales the PWM amplitude, but the current setting is not measured and adjusted when changes to the operating conditions occur. Use the PWM offset and the hold current both to adjust the motor current.

In current control mode, setting the hold current is the way to adjust the spinning motor current. The driver will measure the current and automatically adjust the voltage to maintain the hold current, even with the operating conditions change. The PWM offset may be changed to help the automatic tuning procedure, but changing the hold current alone is enough to adjust the motor current since the driver will adjust the offset automatically.


<a id="org9c732dc"></a>

# Examples


## Wiring


### Teensy 4.0

![img](./images/TMC2209_teensy40.svg)


### Mega 2560

![img](./images/TMC2209_mega2560.svg)


### Wiring Documentation Source

<https://github.com/janelia-kicad/trinamic_wiring>


<a id="orgb1ac219"></a>

# Hardware Documentation


## Datasheets

[Datasheets](./datasheet)


## TMC2209 Stepper Driver Integrated Circuit

[Trinamic TMC2209-LA Web Page](https://www.trinamic.com/products/integrated-circuits/details/tmc2209-la)


## TMC429 Stepper Controller Integrated Circuit

[Trinamic TMC429 Web Page](https://www.trinamic.com/products/integrated-circuits/details/tmc429/)


## SilentStepStick Stepper Driver Board

[Trinamic TMC2209 SilentStepStick Web Page](https://www.trinamic.com/support/eval-kits/details/silentstepstick)


## BIGTREETECH TMC2209 V1.2 UART Stepper Motor Driver

[BIGTREETECH TMC2209 Web Page](https://www.biqu.equipment/products/bigtreetech-tmc2209-stepper-motor-driver-for-3d-printer-board-vs-tmc2208)


## Janelia Stepper Driver

[Janelia Stepper Driver Web Page](https://github.com/janelia-kicad/stepper_driver)