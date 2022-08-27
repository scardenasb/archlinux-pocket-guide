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
- [Mount partitions](#mount-partitions)
  - [Pacstrap command](#pacstrap-command)
- [Configure the system](#configure-the-system)
  - [Create the file system table](#create-the-file-system-table-fstab)
  - [Set time zone](#set-time-zone)
  - [Set hardware clock](#set-hardware-clock)
- [Localization](#localization)
- [Network configuration](#network-configuration)
- [Users and passwords](#users-and-passwords)

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

| Type | Name                  |
|------|-----------------------|
| EFI  | EFI SYSTEM            |
| SWAP | Linux swap            |
| /    | Linux x86-64 root (/) |
| HOME | Linux filesystem      |

### Format partitions
| Name                  | Format Type | Command                                                           |
|-----------------------|:-----------:|:-----------------------------------------------------------------:|
| EFI System            | FAT 32      | `mkfs.fat -F32 /dev/your_efi_device`                              |
| Linux swap            | -           | `mkswap /dev/your_swap_device` and `swapon /dev/your_swap_device` |
| Linux x86-64 root (/) | ext4        | `mkfs.ext4 /dev/your_root_device`                                 |
| Linux filesystem      | ext4        | `mkfs.ext4 /dev/your_filesystem_device`                           |

## Mount partitions
*Mount root (/)*
```bash
mount /dev/your_root_device /mnt
```
*Mount*
### Pacstrap command
*Run `pacstrap` command to install base system for arch onto recent /mnt mounted*

```bash
pacstrap /mnt base linux linux-firmware
```
- base: base package
- linux: kernel
- linux-firmware: firmware for common hardware

## Configure the system
### Create the file system table (fstab)
```bash
genfstab -U /mnt >> /mnt/etc/fstab
```
### Set time zone
*To get into de big mount*
```bash
arch-chroot /mnt
```
*To check de available zones*
```bash
ls /usr/share/zoneinfo
```

*To create the symlink*
```bash
ln -sF /usr/share/YOUR_REGION/YOUR_CITY /etc/localtime
```

### Set hardware clock
```bash
hwclock --systohc
```

### Localization
*Edit `/etc/locale.gen` (uncomment line)*
```bash
vi /etc/locale.gen
```
*uncomment yours, for US ->*
![image](https://user-images.githubusercontent.com/84429399/187018409-255ffffb-4cef-4de3-b013-ef4e11d72f02.png)

```bash
locale-gen
```

*if there is no `vi` installed, install vim or nano with `pacman -S vim_or_nano`*


### Network configuration
*Create the file `hostname` and write a line with your hostname (example archvm)*
```bash
vim /etc/hostname
```

*set hosts with*
```bash
vim /etc/hosts
```

*add this 3 lines, use tabs.*
```
127.0.0.1 localhost
::1       localhost
127.0.1.1 yourhostname.localdomain  yourhostname
```

## Users and passwords
*Set the password to the root user*
```bash
passwd
```

*Create new user and set his password*
```bash
useradd user_name_you_want
```

```bash
passwd user_name_you_want
```

*Check if this user is member of the wheel group (to let him use `sudo` instead of re-loggin as root)*
```bash
usermod -aG wheel user_name_you_want
```

*you can add more groups separated with commas and no spaces, like `usermod -aG wheel,storage,optical`*
