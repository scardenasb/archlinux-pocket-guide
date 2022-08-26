# Guide to install archlinux (Virtual Machine)

## Table of contents
- [Preinstallation](#preinstallation)
  - [Download iso](#download-iso)
  - [Verify signature](#verify-signature)
- [Previous setup](#previous-setup)
  - [Keyboard layout](#keyboard-layout)
  - [Test connection](#test-connection)
  - [Check boot mode](#check-boot-mode)
  - [Update system clock](#update-system-clock)
- [Partition (fdisk)](#partition-fdisk)

<br></br>

## PREINSTALLATION

### Download iso
*Choose a server to [Download](https://archlinux.org/download/) your iso.*
### Verify signature
*Check the signature to avoid malicious iso.*
```bash
gpg --keyserver-options auto-key-retrieve --verify archlinux-version-x86_64.iso.sig
```
*or in a current arch installation*
```bash
pacman-key -v archlinux-version-x86_64.iso.sig
```
## PREVIOUS SETUP
### Keyboard layout
*To list avilables keyboard layouts,* `| less` *in case you can't scroll the terminal, use vim commands to move around.*
```bash
ls /usr/share/kbd/keymaps/**/*.map.gz | less
```
*then set your keyboard layout, example*
```bash
loadkeys dvorak
```

### Test connection
*To check if your [network interface](https://wiki.archlinux.org/title/Network_interface) is listed and enabled.*
```bash
ip link
```
*then verify your connection*
```bash
ping archlinux.org
```
### Check boot mode
```bash
ls /sys/firmware/efi/efivars
```
### Update system clock
*To check if the system clock is accurate.*
```bash
timedatectl set-ntp true
```
*then check system status.*
```bash
timedatectl status
```

## PARTITION (fdisk)
