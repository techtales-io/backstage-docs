<!-- markdownlint-disable MD013 -->
<!-- markdownlint-disable MD033 -->
<!-- markdownlint-disable MD046 -->

# Installation

## Create usb medium

```console
dd bs=4M if=path/to/archlinux.iso of=/dev/sdx status=progress oflag=sync
```

## BIOS configuration

Enter BIOS and configure

- "System Configuration" > "SATA Operation": "AHCI"
- "Secure Boot" > "Secure Boot Enable": "Disabled"

Boot the Arch USB medium.

## Keymap

Set desired keymap

```console
loadkeys de-latin1
```

## Partitioning and formatting

```console
fdisk /dev/nvme0n1
```

Create partitions

| Size  | Type                 | Notes                |
| ----- | -------------------- | -------------------- |
| 512M  | EFI partition        |                      |
| -250G | Linux partition      | to be encrypted      |
| 250G  | Microsoft basic data | optional for Windows |

Format EFI, if needed

```console
mkfs.fat -F32 /dev/nvme0n1p1
```

!!! note

    Install Windows now (optional)

## Setup Internet Connection

```console
iwctl –passphrase passphrase station wlan0 connect <SSID>
```

## Sync clock

```console
timedatectl set-ntp true
```

## Format and encrypt btrfs

```console
cryptsetup luksFormat --type=luks2 /dev/nvme0n1p2
cryptsetup open /dev/nvme0n1p2 luks
mkfs.btrfs -L luks /dev/mapper/luks
```

### Create btrfs subvolumes

```console
mount -t btrfs /dev/mapper/luks /mnt
btrfs subvolume create /mnt/@root
btrfs subvolume create /mnt/@home
btrfs subvolume create /mnt/@snapshots
```

### Mount btrfs subvolumes

```console
umount /mnt
mount -o subvol=@root /dev/mapper/luks /mnt
mkdir /mnt/{home,.snapshots}
mount -o subvol=@home /dev/mapper/luks /mnt/home
mount -o subvol=@snapshots /dev/mapper/luks /mnt/.snapshots
```

### Mount EFI partition

```console
mkdir /mnt/boot
mount /dev/nvme0n1p1 /mnt/boot
```

## Install the base system and a few packages

```console
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

```console
genfstab -U /mnt >> /mnt/etc/fstab
```

Verify and adjust if necessary.

For btrfs filesystems consider:

- change "relatime" to "noatime" to reduce wear on SSD
- adding "discard" to enable continuous TRIM for SSD

## Enter the new system

```console
arch-chroot /mnt
```
