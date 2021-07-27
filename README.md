# EE120-2AE8GB3 - fixing a faulty 4GEE Wifi
Findings of an EE 4GEE Wifi EE120-2AE8GB3 (known as a MW120VB-2XE1000 for update purposes).

I bought a faulty 4GEE Wifi which wouldn't boot so storing findings here. 

Sorry if it's a mess but I normally do this sort of thing without publishing online.

The device would show the 4 LEDs and then just flash the battery LED a few times before going dead. 

When connecting to a PC it was coming up as a TPST Recovery Port with USB Vendor ID_1BBB & Product ID_007A.

It was sold as having a battery problem, a multimeter showed 2.4v when it arrived which raised up overnight to over 3v so seemed it was half alive as trickle charging.

I tried finding UART pins with a sigrok fx2 & Pulseview but nothing seemed to be outputting readable data at any speed, same with a ttl device going from 300bps to 5000000bps and various other settings, ie 8N1, 7E1, 7O1. Possibly a result of it not booting fully. There were data bursts on two pins but nothing would decode it.

I had a play with the hardware and found a way to get it to present the SCSI storage so you could change modes.

# Accessing ADB shell
To access the ADB Shell remove the battery, hold down the reset button and then power on. 
Keep holding reset for over ten seconds and eventually the LEDs will start alternating.
You should now have a mass storage device visible in Linux.

I found the TCL Switch Tool for Windows and then found great info on Alex Studer's blog here - https://alex.studer.dev/2021/01/04/mw41-1

To switch the device into the right mode you just send the following command in Linux:

sg_raw /dev/sg2 16 f9 00 00 00 00 00 00 00 00 00 00 00 00 00 00 -v

Output should be like follows:
cmd to send: 
16 f9 00 00 00 00 00 00  00 00 00 00 00 00 00 00
NVMe Result=0x0

You should now have root access via adb shell.

The root password is standard oelinux123 judging by the hash in the passwd file.

From the dmesg logs it indicated a problem with mounting ubi1_0 rootfs which was stored in mtd15.

I could use adb pull to backup most of the mtd partitions and using the file command and then ubidump (https://github.com/nlitsme/ubidump) and ubi_reader (https://github.com/jrspruitt/ubi_reader/) I could see volumes and extract files etc.

As the device wasn't fully booting it was half bricked, but luckily just enough was up to try reflashing ubi1_0 as follows.

cat /dev/ubi1_0 > backuptest

ubiupdatevol /dev/ubi1_0 backuptest

The ubifs volume extracted directly from ubi1_0 was larger than that extracted from the mtd15 dump aklthough both had the same crc. The device was knackered so balls to the wall time as I understand EE and Alcatel support aren't great for the EE120 devices for whatever reason and replace with other spec devices.

I used software reboot to ensure it had time to fully finish flashing and this time the device booted.

# WiFi Configuration location
Now just battled past the usernames and passwords which were all changed by the previous owner - Wifi details are stored in /etc/hostapd.conf

ssid=Private Hire Cab
wpa_passphrase=supertaxi

# Admin u/p location
The login password for the web interface is stored in a sqlite db as follows:

![image](https://user-images.githubusercontent.com/32154290/127005315-88f95511-af9b-4e77-b6b3-4fdc57dbe7ad.png)

Once I had access I reset the settings to my likings and now could get into the interface to kick off an update.

# Firmware over the air details
At this point a file named _fota.log was created in /cache and when the update was confirmed via the GUI temporary staging files are created in /cache/downloaded until complete when they are put together as update.zip which is then applied over a reboot.

You can easily recreate the URL checked with wget as follows (using a made up IMEI):

wget "http://g2master-sa-east.tctmobile.com/check.php?id=356508090059911&curef=MW120VB-2XE1000&fv=020015&type=FIRMWARE&mode=2&cltp=1006&cktd=0" --user-agent="GOTU Client v1.2.7 Mifi_FOTA"

You get an HTTP 204 if no updates are available or a HTTP 200 if an update is available, for example:

└─# cat 'check.php?id=356508090059911&curef=MW120VB-2XE1000&fv=020015&type=FIRMWARE&mode=2&cltp=1006&cktd=0'
<?xml version="1.0" encoding="utf-8"?>
<GOTU><UPDATE_DESC>9</UPDATE_DESC><ENCODING_ERROR>0</ENCODING_ERROR><CUREF>MW120VB-2XE1000</CUREF><VERSION><TYPE>2</TYPE><FV>020015</FV><TV>020018</TV><SVN>VER_020018</SVN><RELEASE_INFO><year>2018</year><month>09</month><day>11</day><hour>14</hour><minute>25</minute><second>52</second><timezone>GMT 8</timezone><publisher>peng.z</publisher></RELEASE_INFO></VERSION><FIRMWARE><FW_ID>325843</FW_ID><FILESET_COUNT>1</FILESET_COUNT><FILESET><FILE><FILENAME>update.zip</FILENAME><FILE_ID>353493</FILE_ID><SIZE>16685759</SIZE><CHECKSUM>1a83ca8c5f84149af024ae3f96d9726319860d2f</CHECKSUM><FILE_VERSION>10</FILE_VERSION><INDEX>0</INDEX></FILE></FILESET></FIRMWARE><SPOP_LIST><SPOP_NB>0</SPOP_NB></SPOP_LIST><DESCRIPTION/></GOTU>

At this point the device rebooted and took a few minutes before booting with the new firmware.

Quite happy with that and now have a working EE120 & EE40 both bought cheaply as faulty.

# Hidden features
The EE120 on firmware 02.00.18 appears to be the same as James Mac White has writte in his great EE70 Github Wiki here - https://github.com/jamesmacwhite/hh70-ee/wiki/Hidden-features-and-settings
