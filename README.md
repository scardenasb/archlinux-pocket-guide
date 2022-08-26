# Guide to install archlinux (Virtual Machine)

## Table of contents
- [Preinstallation](#preinstallation)
  - [Download iso](#download-iso)
  - [Verify signature](#verify-signature)
- [Previous setup](#previous-setup)
  - [List keyboard layout](#list-keyboard-layout)
  - [Set keyboard layout](#set-keyboard-layout)
  - [Test connection](#test-connection)
  - [Check boot mode](#check-boot-mode)
  - [Set clock](#set-clock)
- [Partition (fdisk)](#partition-fdisk)
<br></br>
## Preinstallation
#### Download iso
#### Verify signature
> *To avoid malicious iso.*
> ```
> gpg --keyserver-options auto-key-retrieve --verify archlinux-version-x86_64.iso.sig
> ```
> or in a current arch installation
> ```
> pacman-key -v archlinux-version-x86_64.iso.sig
> ```
## Previous setup
#### List keyboard layout
```
ls /usr/share/kbd/keymaps/**/*.map.gz | less
```
#### Set keyboard layout
```
loadkeys dvorak
```
or
```
loadkeys de-latin1
```
#### Test connection
#### Check boot mode
#### Set clock

## Partition (fdisk)
