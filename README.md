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

## 1.) Download Raspberry Pi OS
 - Download from https://www.raspberrypi.com/software/operating-systems/
 - I'm using 2024-11-19-raspios-bookworm-arm64-lite.img.xz

## 2.) Burn OS image onto SD card
 - I'm using DiskImager from https://diskimager.org/
 - Use a SD card >8Gb
<img src="https://github.com/playfultechnology/RasPBX/blob/main/images/diskimager.jpg" alt="Disk Imager" />

## 3.) 
