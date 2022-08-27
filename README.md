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
- [Boot config](#boot-loader)
  - [Boot loader](#boot-loader)
  - [Boot directory](#boot-directory)
- [Environment](#environment)

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
*Run `pacstrap` command to install base system for arch onto recent mounted /mnt*

```bash
pacstrap /mnt base linux linux-firmware
```
> base: base package
> linux: kernel
> linux-firmware: firmware for common hardware

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

> *if there is no `vi` installed, install vim or nano with `pacman -S vim_or_nano`*


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

*Set this user as member of the wheel group (to let him use `sudo` instead of re-loggin as root)*
```bash
usermod -aG wheel user_name_you_want
```

> *you can add more groups separated with commas and no spaces, like `usermod -aG wheel,storage,optical`*

*install `sudo` with*
```bash
pacman -S sudo
```

*allow in sudo config the wheel group.*
```bash
visudo
```
![image](https://user-images.githubusercontent.com/84429399/187020460-5e860d0c-d467-4f01-8a01-97cdc6ce7903.png)

## Boot config
### Boot loader
*install a [boot loader](https://wiki.archlinux.org/title/Arch_boot_process#Boot_loader), in this case i choose grub*
```bash
sudo pacman -S grub
```

*an boot dependencies*
```bash
pacman -S efibootmgr dosfstools os-prober mtools
```
### Boot directory
```bash
mkdir /boot/EFI
```
*mount efi partition here*
```bash
mount /dev/your_efi_device /boot/EFI
```

### Install grub (to replace grub-legacy one)
```bash
grub-install --target=x86_64-efi --bootloader-id=grub_uefi --recheck
```

*make grub config file*

```bash
grub-mkconfig -o /boot/grub/grub.cfg
```
## Reboot
```bash
umount -R /mnt
```
> *-R to notice any busy partition*
*and reboot with `reboot`*

## Environment
### Network
*network manager*
```bash
pacman -S networkmanager
```

*active networkmanager*
```bash
systemctl enable NetworkManager
```

### Graphic

### Utils

### Security

### Develop

### 
