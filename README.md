![Rasp Pi](https://img.shields.io/badge/Rasp&nbsp;Pi-5-brgreen)
![Rasp Pi OS](https://img.shields.io/badge/Debian-12&nbsp;(Bookworm)-brgreen)
![Asterisk](https://img.shields.io/badge/Asterisk-22-brgreen)
![FreePBX](https://img.shields.io/badge/FreePBX-17-brgreen)


# RasPBX Background
Installation of Asterisk and FreePBX GUI on Raspberry Pi

 - "[Asterisk](https://www.asterisk.org/)" is a PBX communication server
 - "[FreePBX](https://github.com/FreePBX)" is a front-end that exposes Asterisk functionality via a web-based GUI
 - "FreePBX Distro" (previously called "AsteriskNow") was a pre-configured bundle distribution of Asterisk/FreePBX system based on CentOS. But now discontinued.
 - "[RaspPBX](http://www.raspbx.org/)" is a pre-configured bundle distribution of Asterisk/FreePBX based on Raspbian Buster 10, Asterisk 16.13.0, and FreePBX 15.0.16.75. The latest version is from 2020-10-10, but is no longer supported.

So... this repository intends to document the process of creating an up-to-date Asterisk installation on a Raspberry Pi, based on latest releases as follows:
 - Asterisk 22 (2025-01-09)
 - FreePBX 17 (2024-08-02)
 - Rasp Pi OS, Debian 12 Bookworm (2024-11-19)

This is based on snippets from various sources, primarily https://www.dslreports.com/forum/r30661088-PBX-FreePBX-for-the-Raspberry-Pi .

## 1.) Install Raspberry Pi OS (20mins)
 - a.) Download the latest OS image from https://www.raspberrypi.com/software/operating-systems/ . I'm using [2024-11-19-raspios-bookworm-arm64-lite.img.xz](https://downloads.raspberrypi.com/raspios_lite_arm64/images/raspios_lite_arm64-2024-11-19/2024-11-19-raspios-bookworm-arm64-lite.img.xz)
 - b.) Burn the OS image onto an >8Gb SD card. I use [DiskImager](https://diskimager.org/) to do this (but you could alternatively use [Etcher](https://etcher.io/) etc.)
<img src="https://github.com/playfultechnology/RasPBX/blob/main/images/diskimager.jpg" alt="Disk Imager" />

## 2.) Configure OS in Windows (2mins)
 - a.) Once the OS image is burned, the SD card should automically be mounted in Windows Explorer as a partition named "bootfs"
 - b.) Create an empty file in the bootfs root directory named "ssh"
 - c.) Create a text file named "userconf" in the bootfs root directory containing the following:
```
pi:$6$c70VpvPsVNCG0YR5$l5vWWLsLko9Kj65gcQ8qvMkuOoRkEagI90qi3F/Y7rm8eNYZHW8CY6BOIKwMH7a3YYzZYL90zf304cAHLFaZE0
```

<img src="https://github.com/playfultechnology/RasPBX/blob/main/images/bootfs.jpg" alt="BootFS" />

## 3.) Copy Installation script to Raspberry Pi (2mins)
 - a.) Insert the SD card into the Raspberry Pi (connected to your LAN via ethernet cable) and power on
 - b.) Use [WinSCP](https://winscp.net/eng/index.php) to create an SFTP connection to:
 ```
host: raspberrypi.local
username: pi
password: raspberry
```
 - c.) Copy `install` and `install.tar.gz` from this repository to the /home/pi directory.
<img src="https://github.com/playfultechnology/RasPBX/blob/main/images/winscp.jpg" alt="WinSCP" />

## 4.) FreePBX Installation: Phase 1 (3mins)
 - a.) Use PuTTY to create a SSH connection to:
 ```
host: raspberrypi.local
username: pi
password: raspberry
```
 - b.) Set executable permissions on the install script `chmod +x install`
 - c.) Run the install script `sudo ./install`
<img src="https://github.com/playfultechnology/RasPBX/blob/main/images/putty1.jpg" alt="Putty" />
 - d.) When prompted:
   - Set pi user password
   - Set root user password
   - Select FreePBX version
   - Select Asterisk version
   - Use Edge?
   - Disable IPv6 option? ('No' recommended)
 - e.) Review selections as follows:
```
FreePBX Version: 17.0   (c)
Asterisk Version: 22    (d)
Edge Enabled: Yes       (y)
IPv6 Enabled: Yes       (n)
```

## 5.) RaspPi OS Configuration (3mins)
 - a.) System Options->Set Hostname (1/S4) `RasPBX`
 - b.) Localisation Options->Locale (5/L1) `en_GB.UTF-8`
 - c.) Localisation Options->Timezone (5/L2) `Europe - London`
 - d.) Advanced Options->Expand Filesystem (6/A1)
 - e.) Finish/Reboot Now `No`

The Raspberry Pi will reboot, closing the connection to PuTTY.
<img src="https://github.com/playfultechnology/RasPBX/blob/main/images/rpisetup.jpg" alt="Rasp Pi OS" />

