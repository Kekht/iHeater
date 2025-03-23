# iHeater Configuration for Klipper

This repository contains configuration files for the iHeater chamber heater for 3D printers based on Klipper firmware and the iHeater control board. The configuration is designed to manage the chamber heating and fans using the iHeater microcontroller.

![PCB](img/PCB_r2.png)

## Table of Contents

- [Requirements](#requirements)
- [Preparation](#preparation)
- [Flashing Firmware to iHeater](#flashing-firmware-to-iheater)
- [Klipper Configuration](#klipper-configuration)
  - [Connecting the `iHeater` MCU](#1-connecting-the-iheater-mcu)
- [Usage](#usage)
  - [Chamber Heating Control Commands](#chamber-heating-control-commands)
  - [Automation and Logic](#automation-and-logic)
- [Notes](#notes)
- [License](#license)

## Requirements

- **Hardware:**
  - iHeater Control Board
  - NTC 100K 3950 Thermistors (2 pcs)
  - PTC Heating Element 220V 100W, for chamber
  - 7530 220V Fan, for air circulation
  - Thermal fuse KSD9700 or equivalent (220V 5A 130°C)

- **Software:**
  - Klipper (latest version)
  - Configured and working host with Klipper

## Preparation

1. **Assemble the hardware:**
   - Connect the heating element and fans to the iHeater board.
   - Connect the KSD thermal fuse to the appropriate header.
   - Install thermistors into the chamber and connect them to the correct MCU pins.
   - Ensure the pin connections match the configuration file.

2. **Install configuration files:**
   - Copy `iHeater.cfg` into your Klipper configuration directory.

## Flashing Firmware to iHeater

1. **Build Klipper firmware for STM32F042:**

    ```sh
    cd klipper/
    make menuconfig
    ```

2. **In menuconfig, select the following:**

    - Enable extra low-level configuration options
    - Micro-controller Architecture (STMicroelectronics STM32)
    - Processor model (STM32F042)
    - Bootloader offset (8KiB bootloader)
    - Clock Reference (Internal clock)
    - Communication interface (USB (on PA9/PA10))

3. **Disable unnecessary options:**

    ```
    [*] Support GPIO "bit-banging" devices
    [ ] Support LCD devices
    [ ] Support thermocouple MAX sensors
    [ ] Support adxl accelerometers
    [ ] Support lis2dw and lis3dh 3-axis accelerometers
    [ ] Support MPU accelerometers
    [ ] Support HX711 and HX717 ADC chips
    [ ] Support ADS 1220 ADC chip
    [ ] Support ldc1612 eddy current sensor
    [ ] Support angle sensors
    [*] Support software based I2C "bit-banging"
    [ ] Support software based SPI "bit-banging"
    ```

4. Save and exit the menu.

5. **Compile the firmware:**

    ```sh
    make clean
    make
    ```

    Expected output:
    ```
    Creating hex file out/klipper.bin
    ```

6. **Flashing firmware to iHeater:**

    If needed, install pyserial:

    ```sh
    sudo apt install python3-serial
    ```

    This example uses Katapult bootloader.

- Connect iHeater to your computer while holding the Mode button to enter flashing mode.

- Check for the device:

    ```sh
    ls /dev/serial/by-id/
    ```

    Expected output:
    ```
    usb-katapult_stm32f042x6_0C0018000D53304347373020-if00
    ```

- Replace with your actual ID and run:

    ```sh
    python3 ~/katapult/scripts/flashtool.py -d /dev/serial/by-id/usb-katapult_stm32f042x6_... -f out/klipper.bin
    ```

    Expected output:

    ```
    Flashing '/home/pi/klipper/out/klipper.bin'...

    [##################################################]
    
    Write complete: 20 pages

    Verifying (block count = 319)...

    [##################################################]

    Verification Complete: SHA = 8A3DDF39A0E70B684DC6BAF74EF8F089EBDD6C18

    Flash Success
    ```

- Verify:

    ```sh
    ls /dev/serial/by-id/
    ```

    Expected result:
    ```
    usb-Klipper_stm32f042x6_0C0018000D53304347373020-if00
    ```

    iHeater is now ready to work with Klipper.

## Pin Configuration

| Pin    | Alias       | Function                          |
|--------|-------------|-----------------------------------|
| PA0    | TH1         | Heater thermistor                 |
| PA1    | HEATER      | Heater control                    |
| PA2    | FAN         | Fan control                       |
| PA3    | TH0         | Chamber thermistor                |
| PA4    | MODE        | Mode button                       |
| PA5    | LED3        | LED 3                             |
| PA6    | LED2        | LED 2                             |
| PA7    | LED1        | LED 1                             |

## Klipper Configuration

Copy the `iHeater.cfg` configuration file into the same folder as `printer.cfg` and include it using:

```ini
[include iHeater.cfg]
```

### 1. Connecting the `iHeater` MCU

Update the `iHeater.cfg` file with the correct serial ID:

```ini
[mcu iHeater]
serial: usb-Klipper_stm32f042x6_0C0018000D53304347373020-if00
```

## Usage

### Chamber Heating Control Commands

- Set chamber temperature:

    ```gcode
    M141 S60  ; Set chamber temperature to 60°C
    ```

- Wait until chamber reaches target temperature:

    ```gcode
    M191 S60  ; Wait for chamber to reach 60°C
    ```

- Stop chamber heating:

    ```gcode
    M141 S0   ; Disable chamber heating
    ```

- Add `M141 S0` to the end of your slicer's G-code to safely turn off the heater.

## Notes

- **Safety:**
  - Ensure all wiring is correct and secure.
  - Make sure `min_temp` and `max_temp` values are appropriate for your hardware.

- **Hardware check:**
  - Test the heater and fan before regular use.
  - Monitor temperatures during initial testing.

- **PID Tuning:**
  - Perform PID calibration if needed for accurate temperature control.

## License

This project is licensed under the MIT License. See the LICENSE file for details.

> ⚠️ **Warning:** Using heating elements and temperature control involves risks of fire and equipment damage. Always follow manufacturer guidelines and observe safety precautions.