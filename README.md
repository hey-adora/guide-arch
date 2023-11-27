# Requirements

- AMD GPU
- UEFI
- Make sure to turn off [secure](https://www.top-password.com/blog/wp-content/uploads/2019/09/disable-secure-boot.jpg)/[fast](https://key.wpxbox.com/img/2021/04/Fastboot-UEFI-BIOS.jpg) boot in your UEFI settings.

# Features

- EXT4
- [Swap](https://wiki.archlinux.org/title/swap)
- Gnome
- [Wifi](https://wiki.archlinux.org/title/iwd) and [Bluetooth](https://wiki.archlinux.org/title/bluetooth)
- [AMD GPU](https://wiki.archlinux.org/title/AMDGPU)

# Index

1. [Pre-installation](#pre-installation)
	1. [Double check](#double-check)
	2. [Connecting to wifi (optional)](#connecting-to-wifi-optional)
	3. [Confirm that you have Internet conenction](#confirm-that-you-have-internet-conenction)
	4. [Update kerings if installer outdated (optional)](#update-kerings-if-installer-outdated-optional)
2. [Formatting](#formatting)
	1.  [Create partitions](#create-partitions)
	2.  [Mount partitions](#mount-partitions)
3. [Installation](#installation)
	1. [Base install](#base-install)
	2. [Bootloader setup](#bootloader-setup)
	3. [Configuration setup](#configuration-setup)
	4. [Install the rest of the packages](#install-the-rest-of-the-packages)
	5. [User setup](#user-setup)
	6. [AUR setup](#aur-setup)
4. [Post installation](#post-installation)
	1. [Reboot](#reboot)


# Pre-installation

### Double check

1. Confirm that its [**EFI**](https://wiki.archlinux.org/title/GRUB)
```sh
ls /sys/firmware/efi/efivars
```
>If you see huge text output, its efi. Else if you get error, make sure you selected to boot with UEFI via [bootmenu](https://1.bp.blogspot.com/-tZaLcISTlBo/XqQTKpWfoLI/AAAAAAAAHmY/x18ibHarXjoUcsJGN_xjXzUyBPqRJe_XQCLcBGAsYHQ/s1600/step_1.jpg).

### Connecting to wifi (optional)

1. Enter iwctl shell
```sh
iwctl
```

2. Show wifi devices
```sh
device list
```

3. Scan for wifis
```sh
station wlan0 scan
```

4. Show wifis
```sh
station wlan0 get-networks
```

5. Connect to wifi
```sh
station wlan0 connect "name"
```

6. Leave iwctl shell
```sh
exit
```

### Confirm that you have Internet conenction

1. Confirm that you have internet connection
```bash
ping google.com
```

### Update kerings if installer outdated (optional)

1. Update kerings
```bash
pacman -Sy archlinux-keyring
```
>This is to prevent errors when using pacman later, that can accur if you are not using the latest installer. (skip if you just now flashed .iso)

# Formatting

### Create partitions.

1. Check where you want to install disk **NAME** and **SIZE**
```bash
lsblk
```

2. Carefully choose disk and format it with
```sh
mkfs.ext4 /dev/nvme0n1
```
>**Be sure to replace nvme0n1 with your disk name from `lsblk` if its not the same**

3. Calculate how much you want to have [swap](https://wiki.archlinux.org/title/swap) (i recommend 20gb at least). So for example if `lsblk` shows that u have 465G size disk, subtract 1 for boot partition, 20 for swap parition. So you will have left **444G for main** parition, **1G for boot** partition and **20G for swap** partition

4. Run partitioning tool
```bash
gdisk /dev/nvme0n1
```

5. Create 1G boot partition
	1. For "Command (? for help)" write  `n` and press `enter`
	2. For "Partition number" just press `enter`
	3. For "First sector ..." just press `enter`
	4. For "Last sector ..." write `+1G` and press `enter`
	5. For "Hex code or GUID ..." write `ef00` and press `enter`

6. Create 444G main partition
	1. For "Command (? for help)" write  `n` and press `enter`
	2. For "Partition number" just press `enter`
	3. For "First sector ..." just press `enter`
	4. For "Last sector ..." write `+444G` and press `enter`
	5. For "Hex code or GUID ..." just press `enter`

7. Create 20G swap partition
	1. For "Command (? for help)" write  `n` and press `enter`
	2. For "Partition number" just press `enter`
	3. For "First sector ..." just press `enter`
	4. For "Last sector ..." just press `enter`
	5. For "Hex code or GUID ..." write `8200` and press `enter`

8. Write `w` and press `enter`

9. Format the boot partition
```bash
mkfs.fat -F32 /dev/nvme0n1p1
```

10. Format the main partition
```bash
mkfs.ext4 /dev/nvme0n1p2
```

11. Format the swap partitoin
```bash
mkswap /dev/nvme0n1p3
```

### Mount partitions.

1. Mount the main partition
```bash
mount /dev/nvme0n1p2 /mnt
```

2. Create boot folder
```bash
mkdir /mnt/boot
```

3. Mount boot partitoin
```bash
mount /dev/nvme0n1p1 /mnt/boot
```

4. Mount swap partition
```bash
swapon /dev/nvme0n1p3
```

# Installation

### Base install

1. Install base pacakges
```bash
pacstrap /mnt base linux linux-firmware grub efibootmgr amd-ucode os-prober git nano 
```

2. Generate mount points
```bash
genfstab -U /mnt >> /mnt/etc/fstab
```

3. Enter installed linux environment
```bash
arch-chroot /mnt
```

### Bootloader setup

1. Create initial ramdisk environment
```bash
mkinitcpio -p linux
```

2. Install grub bootloader
```bash
grub-install --target=x86_64-efi --efi-directory=/boot --bootloader-id=GRUB
```

3. Generate grub bootloader config
```bash
grub-mkconfig -o /boot/grub/grub.cfg
```

### Configuration setup

1. Set timezone
```bash
ln -sf /usr/share/zoneinfo/Europe/Vilnius /etc/localtime
```
>You can list zones with `ls /usr/share/zoneinfo/` or with `ls /usr/share/zoneinfo/Europe/`

2. Sync hardware clock
```bash
hwclock --systohc
```

3. Uncomment **UTF-8** languages that you want by deleting `#` then save and exist
```bash
nano /etc/locale.gen
```
>For example `en_US.UTF-8`

4. Generate the languages that you selected
```bash
locale-gen
```

5. Set the default language 
```bash
echo "LANG=en_US.UTF-8" >> /etc/locale.conf
```

6. Set pc name
```bash
echo "angel-pc" >> /etc/hostname
```
>You can "angel-pc" to whatever you want, but don't write special characters or spaces.

7. Setup host file with `nano /etc/hosts` and write in it
```
127.0.0.1 localhost
::1       localhost
127.0.1.1 angel-pc.localdomain angel-pc
```
>Then save and exit

8. Enable multilib support with `nano /etc/pacman.conf` and uncomment by deleting `#` from:
```
[multilib]  
Include = /etc/pacman.d/mirrorlist
```
>Then save and exit

9. (Optional) multi-threading compiling with `nano /etc/makepkg.conf` by uncommenting line: 
```
MAKEFLAGS="-j2"
```
>To uncomment, delete `#` on specified line like previous times, then save and exit. Change the number `2` to how many threads you have on your cpu, you can check with `nproc`

### Install the rest of the packages

1. First update pacman database with
```bash
pacman -Sy
```

2. Install packages
```bash
pacman -S networkmanager network-manager-applet dialog wpa_supplicant mtools dosfstools reflector inetutils base-devel linux-headers   bluez bluez-utils xdg-utils xdg-user-dirs alsa-utils pipewire lib32-pipewire pipewire-alsa pipewire-pulse pipewire-jack   gnome gnome-extra gdm xdg-desktop-portal-gnome   xdg-desktop-portal fish xorg wayland qt5-wayland qt6-wayland breeze   mesa lib32-mesa xf86-video-amdgpu vulkan-radeon lib32-vulkan-radeon vulkan-radeon libva-mesa-driver lib32-libva-mesa-driver mesa-vdpau lib32-mesa-vdpau steam wine-staging lutris wine-mono wine-gecko gamemode lib32-gamemode  noto-fonts noto-fonts-cjk noto-fonts-emoji noto-fonts-extra ttf-liberation
```

3. Enable services (case sensitive)
```bash
systemctl enable NetworkManager
systemctl enable bluetooth
systemctl enable gdm
```

4. Set default shell for root (Optional)
```bash
chsh -s /bin/fish
```

### User setup

1. Set root password
```
passwd
```

2. Create user
```bash
useradd -mG wheel angel
```
>You can name it whatever you want, but don't use special characters or spaces.

3. Set password for angel user
```bash
passwd angel
```

4. Add sudo support for wheel group with `EDITOR=nano visudo` by uncommenting `%wheel ALL=(ALL) ALL` like this:
```
## Uncomment to allow members of group wheel to execute any command
%wheel ALL=(ALL) ALL
```
>Then save and exit.

5. Switch to your user
```bash
su angel
```

4. Set default shell for user (Optional)
```bash
chsh -s /bin/fish
```

6. Go to your home folder
```bash
cd ~
```

### AUR setup

1. Clone the repository locally
```bash
git clone https://aur.archlinux.org/yay
```

1. Enter the repository
```bash
cd yay
```

3. Install yay 
```bash
makepkg -si PKGBUILD
```

# Post installation

### Reboot

1. First exist your user by pressing `ctrl+d` then exist arch-chroot by pressing  `ctrl+d` again

2. Then unmount all partitions
```bash
umount -a
```
>**Errors are expected**, continue.

3. Reboot
```bash
reboot
```
