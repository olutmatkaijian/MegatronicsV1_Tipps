If you should run into the trouble of ruining your Megatronics v1 boards bootloader, rejoice in the fact that you can use a Megatronicsv3 bootloader as well! We have dumped the Megatronicsv3 bootloader to make the Megatronicsv1 board usable again, and it worked. The corresponding bootloader, if I am not mistaken, should even be available in Marlin firmware in `hardware/avr/<ver_num>/bootloaders/stk500boot_v2_mega2560.hex`. In section below there is described how to burn bootloader to v1 (step 11).


# Megatronics v3 for 3D Printing

1. USE LINUX. WINDOWS will give you an issue "Filename too long" as the linking step is a longer line in the Arduino IDE console than Windows can handle.
2. Find the boards.txt. Usually somewhere in `~/.arduino15/`, but I have also seen it in `/usr/share` - this may depend on which Linux you are using.
Alternatively: Open Arduino IDE -> Files -> Examples -> EEPROM -> eeprom_clear; Sketch -> Open in Folder (or CTRL + K).
3. Close the IDE
4. In the `boards.txt` (could also be in directories `packages/Arduino`) paste the following. This is important in newer Arduino IDE versions! Tested with 1.8.6 and 1.8.19, as of 01.01.2025!
```
  megatronics.name=Megatronics
  megatronics.build.board=AVR_ATMEGA2560
  megatronics.upload.protocol=wiring
  megatronics.upload.maximum_size=258048
  megatronics.upload.speed=115200
  megatronics.bootloader.low_fuses=0xFF
  megatronics.bootloader.high_fuses=0xDA
  megatronics.bootloader.extended_fuses=0xF5
  megatronics.bootloader.path=stk500v2
  megatronics.bootloader.file=stk500boot_v2_mega2560.hex
  megatronics.bootloader.unlock_bits=0x3F
  megatronics.bootloader.lock_bits=0x0F
  megatronics.build.mcu=atmega2560
  megatronics.build.f_cpu=16000000L
  megatronics.build.core=arduino
  megatronics.build.variant=mega
  megatronics.bootloader.tool=avrdude
  megatronics.upload.tool=avrdude
```
6. Save the file
7. Copy the file `stk500boot_v2_mega2560.hex` into the directory `hardware/avr/<ver_num>/bootloaders` - while it should not be necessary, Arduino IDE cannot find if I don't do this. File exists under `hardware/avr/<ver_num>/bootloaders/stk500v2`.
8. Open Arduino IDE
9. Select "Megatronics" from Tools->Board menu bar.
10. Set correct COM port (Usually "/dev/ttyUSB0" or similar) and programmer "AVRISP mkII"
11. Optional for v1 bootloader repair(?): Burn bootloader
12. Upload your firmware

Should you get `stk500v2_ReciveMessage(): timeout` when burning/uploading, check if you have other software running (e.g. Printrun) and connected to board.

# Megatronicsv3 FAN ISSUE WORKAROUND

The MEGATRONICSv3 board has an issue when used with Marlin Firmware: 
The transistor for the FAN0_PIN yields INVERTED PWM signal. This means effectively that if you set FAN_SPEED 0, it will instead set it to FAN_SPEED 255 and vice versa. 
To ensure correct behaviour of the HOTEND FAN, go into `Marlin-2.X.X.X/Marlin/src/gcode/temp/M106_M107.cpp` and change Line 91 to `thermalManager.set_fan_speed(pfan 255 - speed)` and Line 109 to `thermalManager.set_fan_speed(pfan, 255 - 0)`.
After these changes compile and upload, then connect to printer. `M106 P0 S0` should now reduce fan speed significantly while `M106 P0 S255` will give fan full power. `M107` will effectively work the same as `M106 P0 S0` because of weirdness in Marlin code, and thus never turn off the fan completely. 

__There is possibly an issue in the READ and WRITE macros for PINS in Marlin Firmware. Marlin Firmware does not use digitalRead or digitalWrite, but FastIO instead. This note only exists for debugging purposes.__

Issues with fan control can manifest in the following:
1. Blobs in your 3D prints: Since filament in hotend does not remain solid where it should, amount of filament extruded is inconsistent.
2. Clogged hotends: Filament becomes non-solid and creeps up the side of the feeding tube within in the hotend. Heat expansion / Capillay action may do the rest? Either way, a fat blob forms inbetween nozzle and hotend as way too much filament is melted too quickly.

Should you find either of the issues to be happening for you, CHECK IF FAN IS INVERTED WITH M106 P0 S255 (SHOULD BE FULL SPEED) AND M106 P0 S0 (SHOULD BE ALMOST OFF / OFF). 
