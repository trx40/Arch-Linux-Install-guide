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
   
   I recommend making partitions for `/efi, swap, /root` and `/home`
   
5. Use a partition utility you are comfortable with, I will be using cfdisk
   ```
   cfdisk /dev/sdb
   ```
   
6. Setup your partition according to your needs, I will be using the following scheme:
   ```
   /dev/sdb1  EFI  300 MB
   /dev/sdb2  SWAP  8 GB
   /dev/sdb3  Root  40 GB
   /dev/sdb4  Home  70 GB
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
    reflector --latest 40 --download-timeout 10 --protocol http,https --sort rate --save /etc/pacman.d/mirrorlist
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
   ln -sf /usr/share/zoneinfo/Asia/Kolkata /etc/localtime
   ```

2. Configure the localization
   ```
   nano /etc/locale.gen
   ```

   Locate and uncomment `en_US.UTF-8 UTF-8`
   
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

## Adding your Windows boot entry to GRUB
In your favorite terminal enter:

    sudo fdisk -l

You should get a long return that includes something like this:

Device          Start        End    Sectors   Size Type
/dev/sdb1        2048     206847     204800   100M EFI System
/dev/sdb2      206848     239615      32768    16M Microsoft reserved
/dev/sdb3      239616  268675071  268435456   128G Microsoft basic data
/dev/sdb4   268675072 1679302655 1410627584 672.6G Microsoft basic data
/dev/sdb5  1951934464 1953521663    1587200   775M Windows recovery environment
/dev/sdb6  1679302656 1679917055     614400   300M EFI System
/dev/sdb7  1679917056 1696694271   16777216     8G Linux swap
/dev/sdb8  1696694272 1801551871  104857600    50G Linux filesystem
/dev/sdb9  1801551872 1951934463  150382592  71.7G Linux filesystem

    Get the UUID of the EFI partition
    ```
    sudo blkid /dev/sdb1 #(replace sdb1 with the correct partition for you)
    ```

Return: /dev/sdb1: UUID="4E18-B936" BLOCK_SIZE="512" TYPE="vfat" PARTLABEL="EFI system partition" PARTUUID="a794f5d7-beb1-4455-bd6a-74a122d6ab90"

    Grant yourself write permission to the `'40_custom'` file in `/etc/grub.d`

    Open the terminal (ctrl+alt+t) and run the following commands:
    ```
    cd /etc/grub.d
    sudo chmod o+w 40_custom
    ```
    
    Open the 40_custom file
    ```
    sudo nano ./40_custom
    ```

    Write the following at the bottom of the file and replace 4E18-B936 with the correct UUID:

menuentry 'Windows 11' {
    search --fs-uuid --no-floppy --set=root 4E18-B936
    chainloader (${root})/EFI/Microsoft/Boot/bootmgfw.efi
}

    Save the file and close the editor.

    Back in the terminal, remove write permissions.
    ```
    sudo chmod o-w 40_custom
    ```

    Update GRUB using
    ```
    grub-mkconfig -o /boot/grub/grub.cfg
    ```

    (Optional) You can confirm that your change was successful by going to /boot/grub/grub.cfg and checking lines 243-251. It should reflect your edits in the 40_custom file

    Reboot your computer `reboot`
