## Arch linux Fresh Install

### Set Brazilian ABNT keyboard:
```
loadkeys br-abnt2
```

### Verify the boot mode (if exists then is UEFI mode)
```
ls /sys/firmware/efi/efivars
```
### Connect to the internet (plug the cable for this version)
```
ping archlinux.org
```

### Partition the disks
```
fdisk -l

fdisk /dev/the_disk_to_be_partitioned (sdX)

	m  (help menu)
	g  (create new empty GPT table)
```

### Format the partitions
```
mkfs.ext4 /dev/root_partition
mkfs.vfat /dev/boot_partition
```
### Mount the file systems

Mount the root volume to /mnt. For example, if the root volume is /dev/root_partition:
```
# mkdir -p /mnt
```

### Install essential packages

Use the pacstrap(8) script to install the base package, Linux kernel and firmware for common hardware:
```
pacstrap /mnt base linux linux-firmware
```
(alternatively)
```
pacstrap /mnt base linux-zen linux-zen-headers
```
###  Chroot
Change root into the new system:
```
arch-chroot /mnt
```

Run hwclock(8) to generate /etc/adjtime:
```
hwclock --systohc
```
### Localization
```	
pacman -S vim
```
Edit /etc/locale.gen and uncomment 
```
	en_US.UTF-8 UTF-8 
	pt_BR.UTF-8 UTF-8
```
and other needed locales. Generate the locales by running:
```
locale-gen
```
### Network configuration. create the hostname file:
```
vim /etc/hostname

	myhostname
```

Add matching entries to hosts(5):
```
vim /etc/hosts

	127.0.0.1	localhost
	::1			localhost
```

### Initramfs
```
# mkinitcpio -Pv
```

### Root password

Set the root password:
```
passwd
```
### Configure the system Fstab
Generate an fstab file (use -U or -L to define by UUID or labels, respectively):
```
genfstab -U /mnt >> /mnt/etc/fstab
```
## Dual boot loader (GRUB)
```
pacman -S grub efibootmgr
mkdir -p /esp
fdisk -l /dev/sdX (where X disk with windows 10/11)
```
Locate "EFI System" partition and
```
mount /dev/sdXn /esp
```
(where n is the number of "EFI System")
```
grub-install --target=x86_64-efi --efi-directory=/esp --bootloader-id=GRUB
```
```
grub-mkconfig -o /boot/grub/grub.cfg
```

### Installing Some Essential Packages:

Note: to search for a package by name, use: pacman -Ss <AUR_package_name> 
```
pacman -Syyuu

pacman -S 
  syslinux sudo wget curl man  \
  networkmanager pulseaudio-alsa ntfs-3g dosfstools \
  mtools exfat-utils un{rar,zip} zip p7zip \
  base-devel git man openssh \
  plasma sddm packagekit-qt6 xorg xorg-server \
  dolphin kate vlc xarchiver 
```

### Softwares
```
pacman -S firefox plasma-browser-integration firefox-ublock-origin git gitg ufw meld
```
