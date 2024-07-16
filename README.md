# Arch-Linux-Install-guide
The complete Arch Linux install guide. Ext4 + grub
## Getting started

 - Make a bootable flash drive for [Arch Linux ISO](https://archlinux.org/download/) using [Rufus](https://rufus.ie/en/).
 - Make sure you are using UEFI and not BIOS booting.

## Installation

1. To verify the boot mode, enter the following command:
   ```
   ls /sys/firmware/efi/efivars
   ```
2. Enable NTP to synchronize system clock
   ```
   timedatectl set-ntp true
   ```
3. Get an overview of your disks and plan your partition scheme
   ```
   lsblk
   ```
   I recommend making partitions for /efi, swap, /root and /home
4. Use a partition utility you are comfortable with, I will be using cfdisk
   ```
   cfdisk /dev/sdb
   ```
5. Setup your partition according to your needs, I will be using the following scheme:
   ```
   /dev/sdb1  EFI  300MB
   /dev/sdb2  SWAP  8GB
   /dev/sdb3  Root  40GB
   /dev/sdb4  Home  70GB
   ```
   Write changes to disk and exit
6. Format the partitions
   EFI
   ```
   mkfs.fat -F32 /dev/sdb1
   ```
   SWAP
   ```
   mkswap /dev/sdb2
   ```
   Root
   ```
   mkfs.ext4 /dev/sdb3
   ```
   Home
   ```
   mkfs.ext4 /dev/sdb4
   ```
7. Mount your partitions
   SWAP
   ```
   swapon /dev/sdb2
   ```
   Root
   ```
   mount /dev/sdb3 /mnt
   ```
   Home
   ```
   mkdir -p /mnt/home

   mount /dev/sdb4 /mnt/home
   ```
