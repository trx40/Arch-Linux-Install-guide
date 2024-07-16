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
   
7. Format the partitions
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
   
8. Mount your partitions
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
   
9. Configure mirrors using reflector
    ```
    reflector --latest 40 reflector --download-timeout 10 --protocol http,https --sort rate --save /etc/pacman.d/mirrorlist
    ```

## Install the Arch base system

1. Synchronize the repositories
    ```
    pacman -Sy
    ```

2. Bootstrap the system
   ```
   pacstrap /mnt base base-devel linux linux-firmware sudo nano ntfs-3g networkmanager git curl kate dolphin firefox
   ```

3. Generate the fstab file
   >The fstab file can be used to define how disk partitions, various other block devices, or remote file systems should be mounted into the file system.

   ```
   genfstab -U /mnt >> /mnt/etc/fstab
   ```

4. Login to the newly created system
   ```
   arch-chroot /mnt
   ```

## Configure the system

1. Configure time-zone
   ```
   ln -sf /usr/share/zoneinfo/Asia/Dhaka /etc/localtime
   ```

2. Configure the localization
   ```
   nano /etc/locale.gen
   ```

   Locate and uncomment en_US.UTF-8 UTF-8
   
   Execute the following command
   ```
   locale-gen
   ```

   Set default localization
   ```
   LANG=en_US.UTF-8
   ```

3. Configure the Hostname
   Enter your hostname in the `/etc/hostname` file
   ```
   Arch
   ```

   Configure the hosts file
   Open `/etc/hosts` file
   ```
   127.0.0.1  localhost
   ::1        localhost
   127.0.1.1  Arch
   ```

4. Enable the NetworkManager service
   ```
   systemctl enable NetworkManager
   ```
5. Make your user
   ```
   useradd -m -G wheel tejas
   ```

   Setup user password
   ```
   passwd tejas
   ```

6. Enable `sudo` privileges for your user by editiing the `/etc/sudoers` file
   Locate and uncomment the following line:
   ```
   # %wheel ALL=(ALL) ALL
   ```

7. Install the system specific microcode
   ```
   # for amd processors
   pacman -S amd-ucode
   
   # for intel processors
   pacman -S intel-ucode
   ```

8. Install and configure the bootloader
   We will be using GRUB (GRand Unified Bootloader)
   If you have a dual boot system you will also need `os-prober` to detect other OS bootloader
   ```
   pacman -S grub efibootmgr os-prober
   ```
   If your EFI partition and Microsoft EFI are on different partitions you will need to add it manually to GRUB we will do it later

9. Make and mount the `/boot/efi` partition
    ```
    mkdir /boot/efi
    ```

    ```
    mount /dev/sda1 /boot/efi
    ```

10. Install GRUB
    ```
    grub-install --target=x86_64-efi --bootloader-id=Arch
    ```

11. If you're installing alongside other operating systems, you'll have to enable `os-prober` before generating the configuration file. To do so, open the `/etc/default/grub` file in nano text editor. Locate the following line and uncomment it:
    ```
    #GRUB_DISABLE_OS_PROBER=false
    ```
    Now execute the following command to generate the configuration file:
    ```
    grub-mkconfig -o /boot/grub/grub.cfg
    ```

## Final steps
 You have installed the base system but we still need the Display server and graphics driver for using the operating system.

 1. Install the Xorg server
    ```
    pacman -S xorg-server
    ```

2. Install the graphic driver/s
   ```
   # for nvidia graphics processing unit
   pacman -S nvidia nvidia-utils

   # for amd discreet and integrated graphics processing unit
   pacman -S xf86-video-amdgpu

   # for intel integrated graphics processing unit
   pacman -S xf86-video-intel
   ```

3. Install the Desktop Environment
   You can choose either GNOME or KDE
   GNOME:
   ```
   pacman -S gnome
   ```
   Enable the desktop environment
   ```
   systemctl enable gdm
   ```

   KDE:
   ```
   pacman -S plasma
   ```

   Enable the desktop environment
   ```
   systemctl enable sddm
   ```

You have finally installed your system now we will unmount and reboot into your shiny new Arch Linux Desktop

Exit the arch-chroot environment:
```
exit
```

Unmount the root partition
```
umount -R /mnt
```

Now reboot the machine
```
reboot
```

