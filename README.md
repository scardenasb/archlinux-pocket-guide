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
  - [Partition table](#partition-table)
  - [Change partition types](#change-partition-types)
  - [Format partitions](#format-partitions)

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
### Partition table
*You can see your disks with* 
```bash
fdisk -l
```
*Choose the one you want to format with*
```bash
fdisk /dev/your_chosen_disk
```

*Suggested partition table (UEFI With GPT)*
| Type | Available size | Size            | Recomended |
|------|:--------------:|:---------------:|:----------:|
| EFI  | -              | 100MiB - 550Mib | 550Mib     |
| SWAP | 2GB - 8GB      | Equal to RAM    | 2 * RAM    |
|      | 8GB - 64GB     | > 4GB           | 1.5 * RAM  |
|      | > 64GB         | > 4GB           | 1 * RAM    |
| /    | -              | > 15GB          | > 50GB     |
| HOME | -              | Remainder       | > /        |

### Change partition types
*Types table GPT (GUUID)*

| Type | Name             |
|------|------------------|
| EFI  | EFI SYSTEM       |
| SWAP | Linux swap       |
| /    | Linux filesystem |
| HOME | Linux filesystem |

### Format partitions
| Name             | Format Type | Command                                                           |
|------------------|:-----------:|:-----------------------------------------------------------------:|
| EFI System       | FAT 32      | `mkfs.fat -F32 /dev/your_efi_device`                              |
| Linux swap       | -           | `mkswap /dev/your_swap_device` and `swapon /dev/your_swap_device` |
| Linux filesystem | ext4        | `mkfs.ext4 /dev/your_filesystem_device`                           |
