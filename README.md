# EE120-2AE8GB3
Findings of an EE 4GEE Wifi EE120-2AE8GB3

I bought a faulty 4GEE Wifi which wouldn't boot so storing findings here.

The device would show the 4 LEDs and then just flash power a few times before going dead. 

It was sold as having a battery problem, a multimeter showed 2.4v when it arrived which raised up overnight to over 3v so seemed it was half alive.

I tried finding UART pins with a sigrok fx2 & Pulseview but nothing seemed to be outputting readable data, same with a ttl device going from 300bps to 5000000bps. Possibly a result of it not booting fully.

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

The root password is standard oelinux123 judging by the hash in the passwd file.

From the dmesg logs it indicated a problem with mounting ubi1_0 rootfs which was stored in mtd15.

I could use adb to backup most of the mtd partitions and using ubidump (https://github.com/nlitsme/ubidump) and ubi_reader (https://github.com/jrspruitt/ubi_reader/) I could see volumes and extract files etc.

As the device wasn't fully booting it was half bricked, but luckily just enough was up to try reflashing ubi1_0 as follows.

cat /dev/ubi1_0 > backuptest
ubiupdatevol /dev/ubi1_0 backuptest

I used software reboot to ensure it had time to fully finish flashing and this time the device booted.

Now just battling with the hardcoded usernames and passwords - Wifi details are stored in /etc/hostapd.conf

ssid=Private Hire Cab
wpa_passphrase=supertaxi

Nice and easy, now just to find the default admin password location (it's more fun than resetting it to defaults - plus I don't even know if reset to defaults will even work due to the previous ubifs corruption).
