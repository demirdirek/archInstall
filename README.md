efivar -l
|-> check efi shit, there is gotta be lots of things
lsblk
gdisk /dev/sdb
|-> x
|-> z
|-> Y
|-> Y
lsblk
|-> check hardisk

cgdisk /dev/sdb
    1st partition # boot
    |-> enter
    |-> 1024MiB
    |-> EF00
    |-> boot

    2nd partition # swap
    |-> enter
    |-> 16GiB
    |-> 8200
    |-> swap

    3th partition # root
    |-> enter
    |-> 40GiB
    |-> 8300
    |-> root

    4th partition # home
    |-> enter
    |-> enter
    |-> enter
    |-> home

    |-> hit write
        |-> type yes
    |-> hit quit

lsblk 
|-> check partitions
mkfs.fat -F32 /dev/*boot partition*
mkswap /dev/*swap partition*
swapon /dev/*swap partition*
mkfs.ext4 /dev/*root partition*
|-> yes
mkfs.ext4 /dev/*home partition*
|-> yes
lsblk
|-> check formatting

mkdir /mnt/boot
mkdir /mnt/home
mount /dev/*root partition* /mnt
mount /dev/*boot partition* /mnt/boot
mount /dev/*home partition* /mnt/home
lsblk
|-> check mounting

cp /etc/pacman.d/mirrorlist /etc/pacman.d/mirrorlist.backup
sudo pacman -Sy pacman-contrib
rankmirrors -n 6 /etc/pacman.d/mirrorlist.backup > /etc/pacman.d/mirrorlist
|-> its gotta blink a bit
nano /etc/pacman.d/mirrorlist
|-> check mirror list
pacstrap -K /mnt base linux linux-firmware base-devel
genfstab -U -p /mnt >> /mnt/etc/fstab
nano /mnt/etc/fstab
|-> check every partition

arch-chroot /mnt
sudo pacman -S nano bash-completion
nano /etc/local.gen
|-> find your local
local-gen
echo LANG=en_US.UTF-8 > /etc/local.conf
export LANG=en_US.UTF-8
ls /usr/share/zoneinfo/Europe/Istanbul
|-> when you find your local change:
|-> ln -s /usr/share/zoneinfo/Europe/Istanbul > /etc/localtime
hwclock --systohc --utc
echo overlord > /etc/hostname
systemctl enable fstrim.timer
|-> should create symlink
nano /etc/pacman.conf
|-> ctrl+W, multilib
|-> delete [multilib] and "Include" # tags
sudo pacman -Sy
|-> should download multilib package

passwd
|-> enter root password
useradd -m -g users -G wheel,storage,power -s /bin/bash *username*
passwd *username*
|-> enter user password

EDITOR=nano visudo
|-> find "%wheel" and uncomment that
|-> create script to bottom
###
Defaults rootpw
###

mount -t efivarfs efivarfs /sys/firmware/efi/efivars/
ls /sys/firmware/efi/efivars/
|->  to check efi gabagush
bootctl install 

nano /boot/loader/entries/arch.conf
|-> write script
###
title Arch
linux /vmlinuz-linux
initrd /initramfs/linux.img
###

echo "options root=PARTUUID=$(blkid -s PARTUUID -o value /dev/*root partition*) rw" >> /boot/loader/entries/arch.conf
nano /boot/loader/entries/arch.conf
|-> you should see #options root=PARTUUID=*ID* rw#

sudo pacman -S dhcpcd
sudo pacman -S networkmanager

iplink
sudo systemctl enable dhcpcd@*network card from iplink command*.service
|-> should create symlink
sudo systemctl enable NetworkManager.service
|-> should create symlink

sudo pacman -S linux-headers
#dynamic kernel modual system#
sudo pacman -S nvidia-dkms libglvnd nvidia-utils opencl-nvidia lib32-libglvnd lib32-nvidia-utils lib32-opencl nvidia nvidia-settings
sudo nano /etc/mkinitcpio.conf
|-> inside modules
###
MODULES=(nvidia nvidia_modeset nvidia_uvm nvidia_drm)
###
sudo nano /boot/loader/enteries/arch.conf
###
|-> you should see #options root=PARTUUID=*ID* rw nvidia-drm.modeset=1 
###
sudo mkdir /etc/pacman.d/hooks
sudo nano /etc/pacman.d/hooks/nvidia.hook
###
[Trigger]
Operation=Install
Operation=Upgrade
Operation=Remove
Type=Package
Target=nvidia

[Action]
Depends=mkinitcpio
When=PostTransaction
Exec=/usr/bin/mkinitcpio -P
###
exit
umount -R /mnt
reboot

#INSTALL PLASMA OR SOME SHIT#
sudo pacman -S xorg-server xorg-apps xorg-xinit xorg-twm xorg-xclock xterm
sudo pacman -S plasma sddm
sudo systemctl enable sddm.service
reboot

sudo pacman -S steam firefox discord dolphin kate vlc

DONE