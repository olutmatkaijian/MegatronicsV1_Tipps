For some reason the bootloader provided is absolute bumblef***. Appearantly what works is to dump a stock Marlin v3 board, then to flash the correct software. This will make the Marlin v1 usable again. More information will follow.

# Kept for documentation purposes (unimportant)
I am having some problems getting a Megatronics v1 board working correctly for my 3D-Printer. Here is a list to remove some common issues:

0. Use Linux or macOS (?). If you use Windows, for some reason you will end up with a "Filename too long" error as the linking step is a longer line in the console than Windows can handle.
1. Find the boards.txt: Open arduino IDE -> Files -> Examples -> EEPROM -> eeprom_clear 
2. Sketch -> Open in Folder or CRTL+K
3. Close the IDE
4. Find the file `boards.txt` in the folder `hardware`. It could be in the directories `packages/Arduino`.
5. Paste the following:

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

The added lines are important in newer Arduino-IDE versions. I have tried this with Arduino 1.8.6 IDE.

6. Save the file
7. Copy the file `stk500boot_v2_mega2560.hex` into the directory `hardware/avr/<ver_num>/bootloaders` (This should not be necessary, but for some reason the Arduino could not find it if I did not do this. The file should exist in hardware/avr/<ver_num>/bootloaders/stk500v2`)
8. Open the IDE
9. Select Megatronics
10. Set the correct com port and programmer
11. Burn bootloader.

So far this is all I got, now the bootloader seems to be installed "correctly" - at least it verifies to the correct hex value. However, I still cannot upload any Sketches yet. 

There's two more things I will test: A shorter USB cable, and pressing the reset button right after compilation of the firmware.
The issue with `stk500v2_ReciveMessage(): timeout` still remains, but the IDE seems to be correctly configured to notice the board.
