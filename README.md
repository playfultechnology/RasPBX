![Rasp Pi](https://img.shields.io/badge/Rasp&nbsp;Pi-5-brgreen)
![Rasp Pi OS](https://img.shields.io/badge/Debian-12&nbsp;(Bookworm)-brgreen)
![Asterisk](https://img.shields.io/badge/Asterisk-22.1.1-brgreen)
![FreePBX](https://img.shields.io/badge/FreePBX-17.0.19.23-brgreen)

# RasPBX
Installation of Asterisk and FreePBX GUI on Raspberry Pi

 - "[Asterisk](https://www.asterisk.org/)" is a PBX communication server
 - "[FreePBX](https://github.com/FreePBX)" is a front-end that exposes Asterisk functionality via a web-based GUI
 - "FreePBX Distro" (previously called "AsteriskNow") was a pre-configured bundle distribution of Asterisk/FreePBX system based on CentOS. But now discontinued.
 - "[RaspPBX](http://www.raspbx.org/)" is a pre-configured bundle distribution of Asterisk/FreePBX based on Raspbian Buster 10, Asterisk 16.13.0, and FreePBX 15.0.16.75. The latest version is from 2020-10-10, but is no longer supported.

So... this repository intends to document the process of creating an up-to-date Asterisk installation on a Raspberry Pi, based on latest releases as follows:
 - Asterisk 22 (2025-01-09)
 - FreePBX 17 (2024-08-02)
 - Rasp Pi OS, Debian 12 Bookworm (2024-11-19)

This guide is based on my own experience combined with snippets from various sources, primarily https://www.dslreports.com/forum/r30661088-PBX-FreePBX-for-the-Raspberry-Pi .

# Installation

## 1.) Install Raspberry Pi OS (20mins)
 - a.) Download the latest OS image from https://www.raspberrypi.com/software/operating-systems/ . I'm using [2024-11-19-raspios-bookworm-arm64-lite.img.xz](https://downloads.raspberrypi.com/raspios_lite_arm64/images/raspios_lite_arm64-2024-11-19/2024-11-19-raspios-bookworm-arm64-lite.img.xz)
 - b.) Burn the OS image onto an >8Gb SD card. I use [DiskImager](https://diskimager.org/) to do this (but you could alternatively use [Etcher](https://etcher.io/) etc.)
<img src="https://github.com/playfultechnology/RasPBX/blob/main/images/diskimager.jpg" alt="Disk Imager" />

## 2.) Configure OS in Windows (2mins)
 - a.) Once the OS image is burned, the SD card should automically be mounted in Windows Explorer as a partition named "bootfs"
 - b.) Create an empty file in the bootfs root directory named "ssh"
 - c.) We need to create a pi:raspberry user to enable remote access, so create a text file named "userconf" in the bootfs root directory containing the following:
```
pi:$6$c70VpvPsVNCG0YR5$l5vWWLsLko9Kj65gcQ8qvMkuOoRkEagI90qi3F/Y7rm8eNYZHW8CY6BOIKwMH7a3YYzZYL90zf304cAHLFaZE0
```
(that's the hash of the "raspberry" password)
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

## 4.) FreePBX Installation: Phase 1 (2mins)
 - a.) Use PuTTY to create a SSH connection to:
 ```
host: raspberrypi.local
username: pi
password: raspberry
```
- b.) Set executable permissions on the install script `chmod +x install`
- c.) Run the install script `sudo ./install`
<img src="https://github.com/playfultechnology/RasPBX/blob/main/images/putty1.jpg" alt="Putty" />

- d.) When prompted...:
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

## 5.) RaspPi OS Configuration (2mins)
 - a.) System Options->Set Hostname (1/S4) `RasPBX`
 - b.) Localisation Options->Locale (5/L1) `en_GB.UTF-8`
 - c.) Localisation Options->Timezone (5/L2) `Europe - London`
 - d.) Advanced Options->Expand Filesystem (6/A1)
 - e.) Finish/Reboot Now `No`

The Raspberry Pi will reboot, closing the connection to PuTTY.
<img src="https://github.com/playfultechnology/RasPBX/blob/main/images/rpisetup.jpg" alt="Rasp Pi OS" />

## 6.) FreePBX Installation: Phase 2 (5mins)
- a.) Create a new PuTTY connection, using the **new** hostname and/or root credentials set in the previous steps (you may receive a message saying cached key is out-of-date - update it)
e.g. 
 ```
host: raspbx.local
username: root
password: raspberry
```
- b.) Having connected successfully, FreePBX installation will continue automatically. 

<img src="https://github.com/playfultechnology/RasPBX/blob/main/images/freepbx2.jpg" alt="FreePBX Phase 2" />

The Raspberry Pi will reboot, closing the connection to PuTTY.

## 7.) FreePBX Installation: Phase 3 (30mins)
- a.) Restart the PuTTY connection using the same root credentials as in the previous step (right click on the PuTTY menu bar and select "Restart Session")
 ```
host: raspbx.local
login as: root
root@raspbx.local's password: raspberry
```
- b.) Confirm the selection made in 4d., as follows:
```
FreePBX Version: 17.0   (c)
Asterisk Version: 22    (d)
Edge Enabled: Yes       (y)
IPv6 Enabled: Yes       (n)
```
This step will take some time. Also, it does not appear to finish cleanly. My PuTTY connection timed out, with the following as the last message received:

<img src="https://github.com/playfultechnology/RasPBX/blob/main/images/putty2.jpg" alt="Putty" />

The Raspberry Pi will reboot, closing the connection to PuTTY.

## 7.) FreePBX Installation: Phase 4 (2mins)
- a.) Restart the PuTTY connection using the same root credentials as in the previous step for one last time (right click on the PuTTY menu bar and select "Restart Session")
 ```
host: raspbx.local
login as: root
root@raspbx.local's password: raspberry
```
- b.) The FreePBX installation should continue as in previous steps, and will complete with the message `FreePBX Installation Complete`


# Debugging
By default, Asterisk will run in the background. However, it can be useful to attach a remote console to the running process by calling the -r option. You can also specify a level of verbosity using the -v option.
- a.) Use PuTTY to log in to the Raspbery Pi that is running Asterisk, as the `root` user.
- b.) Issue the command `asterisk -rvvv` to connect a console to Asterisk
<img src="https://github.com/playfultechnology/RasPBX/blob/main/images/asterisk-rvvv.jpg" alt="Asterisk -rvvv" />

# FreePBX Configuration

## 8.) FreePBX Settings
- a.) Log in to the web GUI in a browser by going to `raspbx.local`
- b.) Assign an administrator account credentials. I'll choose `asterisk`, `asterisk`

<img src="https://github.com/playfultechnology/RasPBX/blob/main/images/raspbxlocal.jpg" alt="FreePBX Administration" />

- c.) Go to Asterisk - SIP Settings and click **Detect Network Settings**. Ensure that this fills in the Local Networks address (failure to do this will mean that any call will be cut-off after 30 seconds)
<img src="https://github.com/playfultechnology/RasPBX/blob/main/images/freepbxnat.jpg" alt="FreePBX NAT settings" />
- d.) **IMPORTANT** to save changes, click both the Submit button AND the Apply Config button!

## 9.) Create Extensions
An “extension” is the name for any destination on the Asterisk phone network – it defines the number that is dialled to call a physical telephone, a “virtual” software telephone, or access a “feature” service (like the speaking clock or an automated message line). Every number that you want players to be able to dial a number and have something happen needs to have an associated extension.
To create a new extension associated with a telephone, perform the following steps:
- a.) Go Connectivity->Extensions
- b.) Add Extension + Add New SIP [chan_pjsip] Extension
- c.) There are several options, but only three are required:
  - **User Extension** is the phone number that users will dial to call this extension
  - **Display Name** specifies the name associated with this extension within FreePBX, and will also be sent as CallerID for outgoing calls
  - **Secret** is the password used by devices that connect to this extension
```
User Extension: 1000
Display Name: Handset
Secret: 82f5d5f5e4410fc003bd4c120bb06b6c
```
- d.) **IMPORTANT** to save changes, click both the Submit button AND the Apply Config button!

# Connecting a Phone Handset to an Extension
To connect a regular analog phone to the network requires a VoIP phone adaptor. This section explains how to configure the Linksys PAP2T device to map the handset plugged into Line 1 to the extension created in the previous section. Note that if you use a different ATA your settings may differ slightly, but the general approach should remain the same.
- a.) Open a web browser and navigate to the web interface for your ATA device.
- b.) Log on to the management screen (click the Admin Login link at the top right of the page), and click on the Line 1 tab to configure the device plugged into Line 1.
- c.) Change the following settings:
  - **Proxy**: This should be set to the static IP address assigned to your Raspberry Pi running Asterisk. E.g. 192.168.1.33.
  - **Display Name**: This should match the “_Display Name_” set in the FreePBX extension configuration in the previous section.
  - **User ID**: This should match the “_User Extension_” set in the FreePBX extension
  - **Password**: This should match the “_Secret_” set in the FreePBX extension

<img src="https://github.com/playfultechnology/RasPBX/blob/main/images/linksys.jpg" alt="LinkSys Settings" />

# Creating a Virtual Softphone
You can also make and receive calls using a software client mapped to an extension. I’m using the free program MicroSIP to simulate a VOIP phone on my PC.
- a.) Download MicroSIP from https://www.microsip.org/
- b.) Set up account information as follows:
  - **Account Name**: This should match the “_Display Name_” set in the FreePBX extension
  - **SIP Server** and **SIP Proxy**: These should both be set to the static IP address assigned to your Raspberry Pi running Asterisk. E.g. 192.168.1.33.
  - **Username**: This should match the “_User Extension_” set in the FreePBX extension
  - **Domain**: This should be set to `localhost`
  - **Password**: This should match the “_Secret_” set in the FreePBX extension

<img src="https://github.com/playfultechnology/RasPBX/blob/main/images/microsip.jpg" alt="MicroSIP settings" />
As when setting up the ATA in the previous section, the Username and Password must match the Extension and Secret set up in Asterisk
The SIP Server and SIP Proxy should be the IP address assigned to the Raspberry Pi that is running Asterisk.
Domain should be set to “localhost” 


