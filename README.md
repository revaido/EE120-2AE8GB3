# EE120-2AE8GB3
Findings of an EE 4GEE Wifi EE120-2AE8GB3

I bought a faulty 4GEE Wifi which wouldn't boot so storing findings here.

To access the ADB Shell remove the battery, hold down the reset button and then power on. 
Keep holding reset for over ten seconds and eventually the LEDs will start alternating.
You should now have a mass storage device visible in Linux.

Send the following command in Linux:

sg_raw /dev/sg2 16 f9 00 00 00 00 00 00 00 00 00 00 00 00 00 00 -v

Output should be

cmd to send: 
16 f9 00 00 00 00 00 00  00 00 00 00 00 00 00 00
NVMe Result=0x0

You should now have root access via adb shell.

The root password is standard oelinux123
