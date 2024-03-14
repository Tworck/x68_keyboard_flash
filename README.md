# x68 Keyboard Flash Guide

A repository to ensure that the xd85 keyboard can be flashed using a arduino pro micro after bricking the bootloader

## Issue

I ran into the case where my x68 keyboard from XIUDI stopped working
properly. I belive it came from a static discharge from myself. The
result was that I was unable to use the Windows key and that my
backspace button was switched with the back slash "\". When i tried to
reflash the firmware with QMK Toolbox the issue did not go away. I have
read that I need to clear the EEPROM which I also did using QMK Toolbox.
Trying to flash the keyboard afterwards resulted in a memory issue
similar to this

```
>>> dfu-programmer atmega32u4 flash /Users/macuser/Documents/hotdox/mykeymap_v2.hex
    Checking memory from 0x0 to 0x58FF...  Not blank at 0x1.
    The target memory for the program is not blank.
    Use --force flag to override this error check.
```

Further research resulted in the fact that I need to load a new
bootloader onto my keyboard. This is tricky if you had no clue about
low level microcontroller etc. (me) but here is the gist

## Prerequisites

1. Go to [QMK Configurator](https://config.qmk.fm/)
   - Select your keyboard and design your layout(in my case xiudi/xd68)
   - Compile the keyboard firmware (that is for later)
2. Get [QMK Toolbox] (https://github.com/qmk/qmk_toolbox/releases) (there is a link on the configurator page)
3. Install QMK MSYS following instructions at [QMK Documentation](https://docs.qmk.fm/#/newbs_getting_started).
4. Using an Arduino Pro you will need to check what MCU/ chip you have.
   My case was trickier as it was a 5V/16MHz Pro Micro which requires a
   specific firmware (QMK: "To use a 5V/16MHz Pro Micro as an ISP flashing tool, you will
   first need to load a special firmware onto it that emulates a hardware
   ISP flasher.")
   - Get this firmware from [here](https://github.com/qmk/qmk_firmware/blob/master/util/pro_micro_ISP_B6_10.hex)
5. Have the ISP flashing guide ready just in case [ISP Flashing Guide](https://github.com/qmk/qmk_firmware/blob/master/docs/isp_flashing_guide.md)
6. your need to wire some cables to your keyboards ISPC pinouts on the
   back of the board. Just female pin connectors to connect to the GPIO
   of the Arduino.

## Flashing the Arduino

1. Load the ISP firmware onto your Arduino using QMK Toolbox.
   - Select the "pro_micro_ISP_B6_10.hex" file.
   - Plug in the Arduino and note the COM port from QMK Toolbox.
   - Reset the Arduino by shorting the GND to Reset Pin.
   - After you shorted those connectors QMK Toolbox will give you the
     option to flash by pressing the "flash" button.
   - Great job

Alternatively use QMK MSYS here and run:

```
avrdude.exe -p atmega32u4 -c avr109 -U flash:w:"YOURPATH\pro_micro_ISP_B6_10.hex":i -P COM6
```

(Replace COM6 with your specific port)

## Preparing the Keyboard

- Connect your keyboard and clear the EEPROM by pressing the reset
  button of the keyboard and then pressing the Clear EEPROM on QMK Toolbox

## Flashing the Keyboard with the New Bootloader

1. Now it is time to connect the ISP device (Arduino) to your keyboard
   ISPC pins. You can use the connectivity table from the QMK ISP guide.
2. Your can find images of the ISPC pinout online (i.e. [image of ISPC PINOUT](https://www.instructables.com/Adding-ICSP-header-to-your-ArduinoAVR-board/))
   Note: For the XIUDI xd68 the if you flip the keyboard
   - The upper row reads left to right:
     VCC MOSI GND
   - The lower row read left to right:
     MISO SCLK RESET

3. Connection guidline:
    | Arduino Pin | Keyboard Pin |
    |-------------|--------------|
    | VCC         | VCC          |
    | GND         | GND          |
    | 10 (B6)     | RESET        |
    | 15 (B1)     | SCLK         |
    | 16 (B2)     | MOSI         |
    | 14 (B3)     | MISO         |


4. You need to start QMK MSYS now
5. Run:
    ```
    avrdude.exe -p atmega32u4 -c avrisp -U flash:w:"YOURPATH\bootloader_atmega32u4_1.0.0.hex":ic -P COM8 -v -v -v -v
    ```
    (Replace COM8 with your specific port)

## Flashing the Keyboard with New Firmware

1. Return to QMK Toolbox.
2. Connect your keyboard.
3. Select your compiled firmware (e.g., xd68_firmware.hex).
4. Press the reset button on the keyboard.
5. Flash the keyboard by clicking the "Flash" button.
6. Your keyboard should now be functional!
