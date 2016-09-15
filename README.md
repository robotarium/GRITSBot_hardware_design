- [Overview](#overview)
- [Getting Started](#getting-started)
- [Hardware Specifications](#hardware-specifications)
- [Hardware Assembly](#hardware-assembly)

# Overview
This repository contains the GRITSBot's circuit board designs, bill of materials, hardware specifications, and 3D design files. This README file is meant to provide more detailed hardware specifications and a brief tutorial outlining how to get started with the hardware. Specifically, it explains how to program both the motor and the main board.

# Getting Started
The newest version of the GRITSBot features two microcontrollers - an Atmega168/328 (8MHz, 16/32 KB flash, 2 KB RAM) on the motor board and the ESP8266 (80/160 MHz, ~80kB DRAM (Data RAM), ~35kB IRAM (Instruction RAM)) on the main board. Both boards can be programmed through the Arduino IDE but require different toolchains (avr-gcc and Xtensa). Both boards have broken out pins to connect the programmer to. The motor board relies on SPI communication for programming, while the main board's ESP8266 chip has a built-in serial bootloader. Therefore the main board requires only a USB-to-TTL converter to program instead of a programmer for the motor board. Note that for programming, the robot has to be disassembled, i.e. the two boards have to be taken apart (the battery does not have to be disconnected, but it's advisable that the robot's power switch is in the OFF position).

## Motor board programming
The GRITSBot's motor board firmware is available [here](https://github.com/robotarium/GRITSBot_firmware). Before programming the motor board, download all required libraries from that link and copy them to the Arduino IDE's 'libraries' folder.

1. Connect the 6-pin ISP single row programming header (shown in the picture) to the female SPI header pins on the motor board (note that for size reasons, the programming connector on the motor board is a two-row 0.127 inch pitch connector and may require an adapter.)
2. Plug the programmer into the USB port on your host machine (ideally running Linux).
3. Programming the Atmega168/328 chip can be done in a variety of ways 
  * Use a tool called arduino-cmake, a CMake-based toolchain customized for the use of avrdude. The initial setup can be tricky but the tool itself is convenient.
  * Add a board definition to the Arduino IDE's boards.txt file that allows direct programming of the motor board through a programming device in the Arduino IDE.
  * If you have compiled your code already and have a .hex file ready to go, you can use avrdude directly as shown below.

An example of uploading a compiled hex file is shown in the line below, which uploads the "blink.hex" file to the flash memory of an Atmega328 chip through the USB port with the help of an STK500v2 programmer. In the call to _avrdude_ you can see the _-C_ parameter to include its configuration file _avrdude.conf_.
```
avrdude -C avrdude.conf -v -v -v -patmega328 -cstk500v2 -Pusb -Uflash:w:./blink.hex:i -Ulock:w:0x0F:m

```

## Main board programming 
Programming the main board is simpler, since it can be done via a serial adapter. However, attention has to be paid to the operating voltage of the main board, which is 3.3V. The ESP8266 chip is NOT 5V-tolerant and both the supply voltage as well as the serial data signals have to be 3.3V. The following steps assume that your Arduino IDE is set up according to [Sparkfun's tutorial](https://learn.sparkfun.com/tutorials/esp8266-thing-hookup-guide/installing-the-esp8266-arduino-addon). It also assumes that you have downloaded all the GRITSBot's firmware libraries into your Arduino/libraries folder. Then the following steps will program the main board. Note that the GRITSBot's main board firmware is available [here](https://github.com/robotarium/GRITSBot_firmware).

1. Set the "Bootmode" switch on the main board to "Flash" (this ensures that the GPIO0 of the ESP8266 is pulled to logic LOW).
2. Press the push botton on the board to reset the board into programming mode in case the board was powered on before.
3. Select *Generic ESP8266 Module* in the board list of the Arduino IDE (Upload speed can be as high as 921000 Baud, depending on your FTDI adapter).
4. Connect the FTDI serial adapter to the main board and plug the other end into a USB port. Make sure the adapter operates at 3.3V (the broken out pins on the main board contain TX, RX, VCC, and GND).
5. Upload code to the board through the Arduino IDE.

# Hardware Specifications
The robot consists of two individual boards that are stacked through a combination of male and female header pins. 

The main board is the central processing board and responsible for the high-level control of the robot, communication, and power supply. As such, it contains the main microcontroller and a WiFi transceiver. The following list summarizes the main components of the main board.

1. The ESP8266 microcontroller that also hosts the WiFi hardware and runs the WiFi stack in software. The ESP8266 chip contains an SPI-controlled EEPROM chip that enables the chip to be wirelessly reprogrammed. The newest version of the GRITSBot contains the ESP8266 12-E with 4 MB (32 MBit) of flash memory, which allows firmware upgrades over-the-air ([see Adafruit](https://www.adafruit.com/products/2491)).
2. MCP73831 LiPo battery charging chip
3. AP2112K-3.3V 3.3V voltage regulator (capable of providing 600 mA of current, which is necessary since the ESP8266 can have peak current consumptions of 300 mA)
4. MCP1640 step-up converter for powering the motors on the motor board (capable of providing 150 mA)
5. INA219 I2C-enabled current and voltage sensor 
6. ATECC108 encryption and authentication chip

The following two images show the top and the bottom layer of the main board PCV
![Top layer of the main board](https://github.com/robotarium/GRITSBot_hardware_design/blob/master/images/main_board_top.jpg)
![Bottom layer of the main board](https://github.com/robotarium/GRITSBot_hardware_design/blob/master/images/main_board_bottom.jpg)

The motor board contains all components related to locomotion including a microcontroller to control the stepper motors. An additional EEPROM chip allows storing calibration parameters and equips each motor board with a unique ID. Additionally, the motor board contains downward-facing infrared sensors for line-following applications as well as introspective sensors to measure the state of the motor board (e.g. currents and temperatures).

1. One Atmega168/328 microcontroller (the motor board firmware requires over 9 KB which excludes the Atmega88 chip)
2. Two LB1836M motor drivers
3. Two miniature stepper motors
4. One 24AA025UIDT-I/OT 2KBIT I2C EEPROM chip providing a unique ID to each motor board.
5. Two QRE1113 downward-facing infrared line sensors 
6. Two STLM20 temperature sensors measuring motor temperatures
7. Two ZXCT1009 current sensors measuring motor currents
6. Reverse polarity circuitry protecting charger input pins
7. Two LEDs for visual output and debugging

The following two images show the top and the bottom layer of the main board PCV
![Top layer of the motor board](https://github.com/robotarium/GRITSBot_hardware_design/blob/master/images/motor_board_top.jpg)
![Bottom layer of the motor board](https://github.com/robotarium/GRITSBot_hardware_design/blob/master/images/motor_board_bottom.jpg)

# Hardware Assembly
Assembly of the GRITSBot is fairly simple in the sense that few components have to be added once all the circuit boards have been populated with SMD components (services such as Seeedstudio or Circuithub can manufacture and populate the boards for a reasonable price). Manufacturers can also populate the throughhole components (i.e. the male and female header pins) but so far we have done that manually. The main and motor boards have matching male/female headers that can only be stacked in one way. The __assembly sequence__ contains the following steps. 

## Main board assembly
1. Solder a female Losi Micro Walkera 2-Pin Connector plug to the main board's battery pins (i.e. VBAT and neighboring GND pin)
2. Solder a 4-pin male programming header pin to the top of the main board (i.e. the pins TX, RX, Vcc, GND). Note that the top side of the board is the one with the ESP8266 12-E part.
3. Solder four 2-pin male header pins to all four corners on the bottom of the main board. In the schematic and board files, these pins are labeled JP\_RST, JP\_I2C, JP\_GND, and JP\_VCHRG\_IN.
4. Solder a 3-pin female header pin to the power pins in the center of the bottom of the board (labeled "MOTOR", containing the pins Vcc, Vdd, and GND).

## Motor board assembly
The motor board assembly contains a few more steps since the motors have to be soldered on besides the male and female header pins. In addition, the Qi charging coil and receiver circuitry have to be attached to the board. 

### Prepare and mount the motors and wheels
1. Tap the wheels with an M1.7 thread plug.
2. Screw one wheel onto each motor.
3. Solder both motors onto the motor board (a video showing this procedure will follow soon)

### Solder on the connectors
1. Solder four 2-pin female header pins to all four corners on the top of the motor board. In the schematic and board files, these pins are labeled JP\_RST, JP\_I2C, JP\_GND, and JP\_VCHRG.
2. Solder a 3-pin male header pin to the power pins in the center of the top of the motor board (containing the pins Vcc, Vdd, and GND).

### Prepare the Qi receiver parts
1. Disassemble the Qi receiver pad (see bill of materials specifying which receiver pad to purchase).
2. Desolder the USB connector and the coil from the receiver assembly.

### Attach the Qi receiver parts
1. Glue or tape the Qi receiver coil to the 3D part [Charging Coil Base](https://github.com/robotarium/GRITSBot_hardware_design/blob/master/3d_model/Charging_frame_base.stp).
2. Tape the Qi receiver circuitry to the top of the motor board.
3. Solder a wire to each output pin of the receiver circuitry (those pins should be labeled '+' and '-' or '5V' and 'Gnd').
4. Solder the other end of these wires to the following motor board pins: JP\_CHRG\_L and JP\_CHRG\_R
5. Cut the receiver coil wires to the right length and solder them to the input pins of the receiver circuitry. Those pins may not be labeled on the receiver circuitry, but attach them to the same pins that you desoldered them before.

## Robot assembly
2. Sandwich the battery between the motor and the main board and wrap the battery cable around the robot.
3. Stack the motor and the main board keeping the battery in place.
5. Connect battery to the main board.
