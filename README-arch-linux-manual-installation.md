# Arch Linux MANUAL Installation (step-by-step)
A fast guide to install archlinux manually (from scratch) ->
based on: https://wiki.archlinux.org/title/installation_guide

based on: https://omarine.org/blog/preparing-to-install-grub-on-the-gpt-drive/

### download arch linux from official repo
https://archlinux.org/download/

### (optional but necessary) -> Download PGP key `.sig  file` and check autenticity:
```sh
pacman-key -v archlinux-<version>-.iso.sig
``` 

## Create usb media
https://wiki.archlinux.org/title/USB_flash_installation_medium

replace `X` with your sdX usb media `/dev/sdX` 

create with `cat`
```sh
cat archlinux-<version>-.iso > /dev/sdX
````
or create with `dd`
```sh
dd bs=4M if=path/to/archlinux-<version>-.iso of=/dev/sdX conv=fsync oflag=direct status=progress
```

----------------------

## BOOT USB media (mode UEFI)

Now, `its time to boot usb` archlinux media

----------------------


## Commands sequence to proceed with manual installation


### load keyboard layout - brasilian (ABNT2)
```sh
loadkeys br-abnt2
```

### verify archlinux boot mode (UEFI mode)
```sh
ls /sys/firmware/efi/efivars
```
*if apear things in this command then OK! is UEFI mode.

### connect to internet
https://wiki.archlinux.org/title/Network_configuration

Ensure your network interface is listed and enabled, for example:
```sh
ip link
```
For wireless and WWAN, make sure the card is not blocked with rfkill.
    
    Connect to the network:

        1) Ethernet—plug in the cable. (Just plug the cable)

        2) Wi-Fi—authenticate to the wireless network using `iwctl`**. (For Wifi connection)
        
        3) Mobile broadband modem—connect to the mobile network with the mmcli utility.

**See: https://wiki.archlinux.org/title/Iwd#iwctl

#### 1) iwctl

connecting to Wifi device using iwctl:

Fast shorcut command (need know SSID and Wifi password previously)

```sh
iwctl --passphrase passphrase station device connect SSID
```

First, if you do not know your wireless device name, list all Wi-Fi devices:

```sh
[iwd]# device list
[iwd]# device wlan0 show
```

Then, to initiate a scan for networks (note that this command will not output anything):

```sh
[iwd]# station device scan
```

You can then list all available networks:

```sh
[iwd]# station device get-networks
```

Finally, to connect to a network:

```sh
[iwd]# station device connect SSID
```

#### Connect to a network using WPS/WSC

If your network is configured such that you can connect to it by pressing a button (Wikipedia:Wi-Fi Protected Setup), check first that your network device is also capable of using this setup procedure.

```sh
[iwd]# wsc list
```

Then, provided that your device appeared in the above list,

```sh
[iwd]# wsc device push-button
```

Check if internet connection is working:
```sh
ping archlinux.org
``` 

### update system clock
```sh
timedatectl set-ntp true
```

### partition the disks
https://wiki.archlinux.org/title/Fdisk

change `X` with disk letter to be parted

```sh
fdisk /dev/sdX
```

**Remember:** for dual boot it's necessary to `create a EFI partition` to install grub boot manager

```
--> fdisk /dev/the_disk_to_be_partitioned (sdX)
    
	m)  --> help menu
	l)  --> list know partition types
	n)  --> add a new partition
	    for example - select partition number: 19 Linux Swap (create swapfile is preefered)
	    for example - select partition number: 23 Linux root (x86-64)
	    for example - select partition number: 41 Linux home    
    w)  --> write the created partition table on selected disk (sdx)
```
----> Note: create a root / partition and a home /home partion is strongly recommended

### format the partitions
```sh
mkfs.ext4 /dev/root_partition
mkfs.ext4 /dev/home_partition
mkfs.fat -F 32 /dev/efi_system_partition
```

### mount the file systems

Mount the root volume to /mnt. For example, if the root volume is /dev/root_partition:
```sh
mount /dev/root_partition /mnt
```

For UEFI systems, mount the EFI system partition:
```sh
mount --mkdir /dev/efi_system_partition /mnt/boot
```

## Pacstrap install essential packages
https://wiki.archlinux.org/title/Kernel

Use the pacstrap script to install the base package, Linux kernel and firmware for common hardwares
```sh
pacstrap /mnt base linux linux-firmware
```

(alternatively) -> zen kernel
```sh
pacstrap /mnt base linux-zen linux-zen-headers
```

### mount others endpoints in `/mnt/endpoint` in this case is `/home`
```sh
mount /dev/home_partition /mnt/home
```

## Generate system `/etc/fstab` for automount

Generate an fstab file (use -U or -L to define by UUID or labels, respectively):
```sh
genfstab -U /mnt >> /mnt/etc/fstab
```
Check the resulting /mnt/etc/fstab file, and edit it in case of errors. 
Important: may you need add mannualy other partitions to fstab file. For example the /home partition for this tutorial
--> https://wiki.archlinux.org/title/persitent_block_device_naming

#### (optionaly) -> mounting from fstab file

Mounting home partition using fstab:
```sh
	-> list UUID and /dev partitions)
	lsblk -f

	vim /mnt/etc/fstab
	
	-> set UUID manualy on fstab file	
	UUID=xxxx /home ext4 defaults 0 2

	-> mount using fstab file
	mount -a
```

## Instalation

```sh
arch-chroot /mnt
```

### select yout favorite mirrors -> choose that with small latency for example
```sh
vim /etc/pacman.d/mirrorlist
```

````
[Brazilian Mirrors]
	Server = http://mirror.ufscar.br/archlinux/$repo/os/$arch
	Server = http://archlinux.c3sl.ufpr.br/$repo/os/$arch
	Server = http://www.caco.ic.unicamp.br/archlinux/$repo/os/$arch
	Server = https://www.caco.ic.unicamp.br/archlinux/$repo/os/$arch
	Server = http://linorg.usp.br/archlinux/$repo/os/$arch
	Server = http://pet.inf.ufsc.br/mirrors/archlinux/$repo/os/$arch
	Server = http://archlinux.pop-es.rnp.br/$repo/os/$arch
	Server = http://mirror.ufam.edu.br/archlinux/$repo/os/$arch
	Server = http://br.mirror.archlinux-br.org/$repo/os/$arch
````

### Create SwapFile (prefer this)
https://wiki.archlinux.org/title/Swap#Swap_file

Creating swapfile (if you dont create a swap partition)
The performance of a swapfile or a swappartition is not diferent
```sh
dd if=/dev/zero of=/swapfile bs=1M count=4096 status=progress
```
Note: Using dd to allocate a swap file is the most portable solution, see swapon(8) Files with holes for details.

Set the right permissions (a world-readable swap file is a huge local vulnerability):
```sh
chmod 600 /swapfile
```

After creating the correctly sized file, format it to swap:
```sh
mkswap /swapfile
```

If you created a partition for swap, initialize it with mkswap(8):
```sh
mkswap /dev/swap_partition
```

Activate the swap file:
```sh
swapon /swapfile
```

### install some necessary packages for this moment
```sh
pacman -S vim grub efibootmgr os-prober
```

Finally, edit the fstab configuration to add an entry for the swap file:
```sh
vim /etc/fstab
```
add this:
```sh
/swapfile none swap defaults 0 0
```

## Time zone
Set the time zone:

(model)
```sh
ln -sf /usr/share/zoneinfo/Region/City /etc/localtime
```

(example)
```sh
ln -sf /usr/share/zoneinfo/America/Sao_Paulo /etc/localtime
```

### run hwclock to generate `etc/adjtime`:
```sh
hwclock --systohc
```
This command assumes the hardware clock is set to UTC. See System time#Time standard for details.

## Localization
Edit /etc/locale.gen and uncomment 
```
en_`US.UTF-8 UTF-8 
pt_BR.UTF-8 UTF-8
```
and other needed locales. Generate the locales by running:
```sh
locale-gen
```

Create the locale.conf file, and set the LANG variable accordingly:
```sh
vim /etc/locale.conf`
LANG=en_US.UTF-8
```

If you set the keyboard layout, make the changes persistent in vconsole.conf(5):
```sh
vim /etc/vconsole.conf
KEYMAP=br-abnt2
```

## Network host configuration

Create the hostname file:
```sh
echo hostname > /etc/hostname
```

Add matching entries to hosts(5):
```sh
vim /etc/hosts

127.0.0.1	localhost
::1		localhost
127.0.1.1	myhostname.localdomain	myhostname
```

If the system has a permanent IP address, it should be used instead of 127.0.1.1.
Complete the network configuration for the newly installed environment, that may include installing suitable network management software.

## Initramfs
Creating a new initramfs is usually not required, because mkinitcpio was run on installation of the kernel package with pacstrap.
For LVM, system encryption or RAID, modify mkinitcpio.conf(5) and recreate the initramfs image:

```sh
mkinitcpio -Pv
```

### root password
Set the root password:
```
passwd
```

## Boot loader (GRUB)
https://wiki.archlinux.org/title/GRUB

### install grub efi file on `/boot` mounted ÙEFI partition`

```sh
grub-install --target=x86_64-efi --efi-directory=/boot --bootloader-id=GRUB
```

(this command under for MBR bootable usb media)

```sh
grub-install --target=i386-pc /devsdX
```

### generate `grub.cfg` file
```sh
grub-mkconfig -o /boot/grub/grub.cfg
```

## Create user and enable services on boot
```sh
#-------------------------------------------------
# Create main user
#-------------------------------------------------
useradd -m -g wheel -G users -s /bin/bash <yourusername>
passwd -d root
passwd -d <yourusername>
groupadd plugdev
usermod -aG plugdev <yourusername>
```

## Installing some essential packages:

Note: to search for a package by name, use: pacman -Ss <AUR_package_name> 
```sh
pacman -Syyuu
```

some essential packages (for me)
```sh
pacman -S 
	syslinux sudo fakeroot patch make wget curl man openvpn \
	networkmanager pulseaudio-alsa ntfs-3g dosfstools \
	mtools exfat-utils un{rar,zip} zip p7zip \
	base-devel multilib-devel git openssh \
	plasma sddm packagekit-qt5 xorg xorg-server \
	alacrity dolphin kate vlc xarchiver
```

### add username to sudoers and enable boot `sddm` and `networkmanager`

```
#-------------------------------------------------
# Add all real users to sudoers
#-------------------------------------------------
echo '%wheel ALL=(ALL) ALL' | sudo tee -a ./etc/sudoers > /dev/null

#-------------------------------------------------
# Enable services on boot
#-------------------------------------------------
sudo systemctl --root=. enable NetworkManager
sudo systemctl --root=. enable sddm


## Install the especific Vulkan Video Driver (Intel, Nvidia or AMD)
https://wiki.archlinux.org/title/Xorg_(Portugu%C3%AAs)

https://wiki.archlinux.org/title/KDE_(Portugu%C3%AAs)

https://www.edivaldobrito.com.br/como-instalar-o-kde-plasma-5-no-arch-linux-e-derivados/

-> What is my Video Hardware Manufacturer?

https://www.cyberciti.biz/faq/linux-tell-which-graphics-vga-card-installed/

```sh
lspci

(Intel)
pacman -S vulkan-intel intel-media-driver intel-ucode xf86-video-intel

(AMD)
	RADV (part of Mesa project)
	pacman -S vulkan-radeon

	AMDVLK Open (maintained by AMD)
	pacman -S amdvlk

	MDVLK Closed (maintained by AMD)
	pacman -S vulkan-amdgpu-proAUR 

(Nvidia)
	pacman -S nvidia-utils
```

--------------------

## It's time to Reboot 
(if all steps done looks good) and start the new archlinux system

--------------------

## Softwares
```sh
pacman -S 
	firefox plasma-browser-integration firefox-ublock-origin\
	git gitg ufw meld
	docker docker-compose
```

-> for docker use only:
```sh
	sudo systemctl enable docker
	sudo systemctl start docker
	sudo groupadd docker
	sudo usermod -aG docker <youruser>
	reboot
	docker run hello-world
```

## More optional own tasty programs (maybe some dependences are needed):

(WPS Office)
```sh
git clone https://aur.archlinux.org/wps-office.git
makepkg
pacman -U <wps-office.*.zst>
pacman -U <wps-office-mime.*.zst>
```

(Google Chrome)
```sh
git clone https://aur.archlinux.org/google-chrome.git
makepkg
pacman -U <google-chrome.*.zst>
```
