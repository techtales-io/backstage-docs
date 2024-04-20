<!-- markdownlint-disable MD013 -->
<!-- markdownlint-disable MD033 -->
<!-- markdownlint-disable MD046 -->

# Installation

## Create usb medium

```shell
dd bs=4M if=path/to/archlinux.iso of=/dev/sdx status=progress oflag=sync
```

## BIOS configuration

Enter BIOS and configure

- "System Configuration" > "SATA Operation": "AHCI"
- "Secure Boot" > "Secure Boot Enable": "Disabled"

Boot the Arch USB medium.

## Keymap

Set desired keymap

```shell
loadkeys de-latin1
```

## Partitioning and formatting

```shell
fdisk /dev/nvme0n1
```

Create partitions

| Size  | Type                 | Notes                |
| ----- | -------------------- | -------------------- |
| 512M  | EFI partition        |                      |
| -250G | Linux partition      | to be encrypted      |
| 250G  | Microsoft basic data | optional for Windows |

Format EFI, if needed

```shell
mkfs.fat -F32 /dev/nvme0n1p1
```

!!! note

Install Windows now (optional)

## Setup Internet Connection

```shell
iwctl –passphrase passphrase station wlan0 connect <SSID>
```

## Sync clock

```shell
timedatectl set-ntp true
```

## Format and encrypt btrfs

```shell
cryptsetup luksFormat --type=luks2 /dev/nvme0n1p2
cryptsetup open /dev/nvme0n1p2 luks
mkfs.btrfs -L luks /dev/mapper/luks
```

### Create btrfs subvolumes

```shell
mount -t btrfs /dev/mapper/luks /mnt
btrfs subvolume create /mnt/@root
btrfs subvolume create /mnt/@home
btrfs subvolume create /mnt/@snapshots
```

### Mount btrfs subvolumes

```shell
umount /mnt
mount -o subvol=@root /dev/mapper/luks /mnt
mkdir /mnt/{home,.snapshots}
mount -o subvol=@home /dev/mapper/luks /mnt/home
mount -o subvol=@snapshots /dev/mapper/luks /mnt/.snapshots
```

### Mount EFI partition

```shell
mkdir /mnt/boot
mount /dev/nvme0n1p1 /mnt/boot
```

## Install the base system and a few packages

```shell
pacstrap /mnt \
  base \
  base-devel \
  git \
  intel-ucode \
  linux \
  linux-firmware \
  networkmanager \
  network-manager-applet \
  networkmanager-openvpn \
  openssh \
  openvpn
```

## Generate fstab

```shell
genfstab -U /mnt >> /mnt/etc/fstab
```

Verify and adjust if necessary.

For btrfs filesystems consider:

- change "relatime" to "noatime" to reduce wear on SSD
- adding "discard" to enable continuous TRIM for SSD

## Enter the new system

```shell
arch-chroot /mnt
```

## Configure base system

### Setup time

```shell
rm /etc/localtime
ln -sf /usr/share/zoneinfo/Europe/Vienna /etc/localtime
hwclock --systohc
```

### Setup required locales

Uncomment needed locales, f.e. en_DK.UTF-8.

```shell
vim /etc/locale.gen
```

Generate locales.

```shell
locale-gen
```

Create needed files.

```shell
echo "LANG=en_DK.UTF-8" >> /etc/locale.conf
echo "KEYMAP=de-latin1" >> /etc/vconsole.conf
```

### Set hostname

Set hostname in /etc/hosts.

```shell
127.0.0.1   localhost
::1         localhost
127.0.1.1   <hostname>.localdomain <hostname>
```

### Setup user

Set root password first.

```shell
passwd
```

Add your non-root user.

```shell
useradd -m -g users -G wheel -s /bin/bash <username>
```

Set your user password.

```shell
passwd <username>
```

Grant sudo permissions to your user.

```shell
echo '<username> ALL=(ALL) ALL' > /etc/sudoers.d/<username>
```

### Configure mkinitcpio

> vim /etc/mkinitcpio.conf

```shell
MODULES=(i915)
HOOKS=(base systemd autodetect modconf block keyboard sd-vconsole sd-encrypt filesystems fsck)
```

Regenerate image.

```shell
mkinitcpio -p linux
```

### Setup bootloader

Setup systemd boot.

```shell
bootctl --path=/boot install
```

Copy uuid to new entries (2x).

```shell
cryptsetup luksUUID /dev/nvme0n1p2 >> /boot/loader/entries/arch.conf
cryptsetup luksUUID /dev/nvme0n1p2 >> /boot/loader/entries/emergency.conf
```

Adapt entries.

> vim /boot/loader/entries/arch.conf

```shell
title       Arch Linux
linux         /vmlinuz-linux
initrd      /intel-ucode.img
initrd      /initramfs-linux.img
options   rw luks.uuid=<uuid> luks.name=<uuid>=luks root=/dev/mapper/luks rootflags=subvol=@root quiet mem_sleep_default=deep
```

> vim /boot/loader/entries/emergency.conf

```shell
title         Emergency Mode
options     rw luks.uuid=<uuid> luks.name=<uuid>=luks root=/dev/mapper/luks rootflags=subvol=@root quiet mem_sleep_default=deep systemd.unit=rescue.target
```

Set default entry at start.

> vim /boot/loader/loader.conf

```shell
default arch.conf
timeout 10
editor  0
```

Exit and reboot.

### Configure NetworkManager

```shell
systemctl enable NetworkManager
systemctl start NetworkManager
nmtui
```

### Check keymap layout

```shell
localectl status
```

If X11 unknown, set.

```shell
sudo localectl --no-convert set-x11-keymap de
```

### Setup Nvidia (optional)

Install drivers.

```shell
yay -Sy nvidia-open
```

Blacklist nouveau.

> vim /etc/modprobe.d/nvidia.conf

```shell
blacklist nouveau
blacklist nouveaufb
blacklist lbm-nouveau
options nouveau modeset=0
alias nouveau off
alias nouveaufb off
alias lbm-nouveau off
```
