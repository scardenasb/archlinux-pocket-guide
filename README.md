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
*Mount home*
```bash
mkdir /mnt/home
mount /dev/your_home_device /mnt/home
```
### Pacstrap command
*Run `pacstrap` command to install base system for arch onto recent mounted /mnt*

```bash
pacstrap /mnt base linux linux-firmware linux-headers
```
> base: base package
>
> linux: kernel `(can be linux-lts)`
>
> linux-firmware: firmware for common hardware `(can be linux-lts-firmware ?)`
>
> linux-headers `(can be linux-lts-headers)`

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

*Enable service to syncronize timezone*
```bash
systemctl enable systemd-timesyncd
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

`same as`
```bash
hostnamectl set-hostname your_hostname
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

### Enable boot configuration
```bash
vim /etc/mkinitcpio.conf
```
![image](https://user-images.githubusercontent.com/84429399/189509178-1d77a29d-3b85-4dda-8010-62c02a323080.png)

> *add `lvm2` between those words*

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

*and boot dependencies*
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

*Reboot*
```bash
exit
umount -R /mnt
`shutdown now` or `reboot`
```
> *-R to notice any busy partition*
*and reboot with `reboot` or `shutdown now` in VM's*

# Environment
## Network
*network manager*
```bash
pacman -S networkmanager
```

*active networkmanager*
```bash
systemctl enable NetworkManager
```

## System




## Graphic
### Video driver/configs
*xorg-server*
```bash
pacman -S xorg-server
```

*Video driver for Intel or amd*
```bash
pacman -S mesa
```

*Video driver for Nvidia*
```bash
pacman -S nvidia `(for linux kernel)`
pacman -S nvidia-lts `(for linux-lts kernel)`
```

*Video driver for Virtual Machine*
```bash
pacman -S virtualbox-guest-utils xf86-video-vmware
```

*Enable Vbox*
```bash
systemctl enable vboxservice
```

### Desktop environments
**GNOME**
```bash
pacman -S gnome
```
*gnome tweaks*
```bash
pacman -S gnome-tweaks
```

*Enable gnome display manager so it will display log-in screen in the boot*
```bash
systemctl enable gdm
```

*Reboot*
```bash
`shutdown` now or `reboot`
```
> **IMPORTANT after reboot system**
> *Re-config the system language in gnome desktop (It could say `Unspecified`)*

**PLASMA**
```bash
pacman -S plasma-meta kde-applications
```

**XFCE**
```bash
pacman -S 
```

**MATE**
```bash
pacman -S 
```




### Utils

### Security

### Develop

### 
