# Sel4 on Raspberry pi
The sel4 microkernel supports the following Raspberry Pi platforms:
- RPI 3B
- RPI 3B+
- RPI 4B (1GB, 2GB, 4GB, 8GB)

Additional details can be found at: https://docs.sel4.systems/Hardware/

Building and running a sel4 based system requires a number of steps, here we go through all the steps to see a sel4 system run on a RPI 4B 4GB, similiar if not identical steps are done on other RPI platforms.

## Building
sel4/Microkit need to know how much memory a device has at compile time, as such when building sel4 based images (programs to run on your RPI), you need to select the correct platform settings.

For the sel4 build system (this only applies if building the microkernel yourself) the minimal defines are below, as seen in this sel4Test build step for a RPI4B 4GB:
```bash
../init-build.sh -DPLATFORM=rpi4 -DAARCH64=1 -DRPI4_MEMORY=4096
```

For Microkit, the build system is left completely up to the user, but common convention will have your build arguments look like this:
```bash
make BUILD_DIR=build MICROKIT_BOARD=rpi4b_4gb MICROKIT_CONFIG=debug MICROKIT_SDK=./../microkit-sdk-2.0.1
```
Note that you only need to select the correct MICROKIT_BOARD, Microkit builds sel4 for you with appropriate settings.
Usually the resulting image will be 'loader.img', found in your build directory.

## Filesystem
Once you have built the image(s) you want to run, you will need to setup the correct files on a SD card, which the RPI will use to boot.

Starting with an empty SD card, you will need to create a FAT32 boot partition, this partition is where the RPI firmware will search
for a boot-config, a bootloader, device tree files, etc.<br>
If you want Linux (if its running as a VM in your sel4 system) to access storage on the SD card, you will need to add another partition
with a compatible format like ext4.

//relevant files
//link github
//say in folder have minimum files for a rpi4b

//build u boot
```bash
git clone https://github.com/u-boot/u-boot.git u-boot
cd u-boot
make CROSS_COMPILE=aarch64-linux-gnu- rpi_4_defconfig
make CROSS_COMPILE=aarch64-linux-gnu-
```
//drag in

//config.txt
//drag in images

## Input/Output
U-boot can be used over the UART (serial) of the RPI 4B or using a keyboard and monitor plugged directly into the RPI.<br>
The UART is the preferred method; on the RPI 4B the sel4 microkernel outputs all debug info to the UART, and many systems (like Microkit) 
will also use the UART for both debug and regular output (like from printf), where serial output cannot be seen from the u-boot console.

To use the primary UART on the RPI 4B, you will need to plug in serial pins to pins 6 (GND), 8 (TXD) and 10 (RXD). 
If you want to see the serial output on your computer, you will require a serial-to-USB adapter; plug in the serial pins to the RPI as above, 
plug in the usb to your computer, verify the adapter is recognised as a COM port with valid drivers, and then run a monitoring program (for example PUTTY).<br>

For PUTTY, use the following settings:
- Connection Type: Serial
- Speed: 115200
- Everything else: Leave as default

## Running
Plug in the RPI, it will now boot into u-boot. Press any key, as prompted by u-boot, to cancel automatic booting, you now have access to a command line.

To see a list of files detected on the SD card:
```bash
fatls mmc 0
```

sel4 images built for RPI 4B need to be loaded at a specific address in memory. 
You can use these commands to select, load and run an image:
``` bash
fatload mmc 0 0x10000000 <SYSTEM IMAGE>
go 0x10000000
```
If you have followed all the steps correctly, you should now see output from your image, this will typically be sel4 setup info, followed by your program output. 

