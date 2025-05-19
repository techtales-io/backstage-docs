<!-- markdownlint-disable MD013 -->
<!-- markdownlint-disable MD033 -->
<!-- markdownlint-disable MD046 -->

# Btrfs

## Mount Btrfs volumes

```shell
cryptsetup open /dev/nvme0n1p2 luks
mount -o subvol=@root /dev/mapper/luks /mnt
mount -o subvol=@home /dev/mapper/luks /mnt/home
mount /dev/nvme0n1p1 /mnt/boot
```
