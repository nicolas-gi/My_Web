## Epitech project
![](/home/ngillard/Downloads/Epitech.png "Epitech")

# My_Web Tuto 

# First step: load keys & partionning the disk.
```
loadkeys fr
```

```
$ parted /dev/sdX
$ (parted) mklabel gpt
$ (parted) mkpart ESP fat32 1MiB 513MiB
$ (parted) set 1 boot on
$ (parted) mkpart primary ext4 513MiB 100%
```

### Second step : Create a physical volume under which we'll have volume group under which finaly logical volume.

```
$ pvcreate /dev/sda2 (sda2 because sda1 is for the boot)
$ vgcreate lvm /dev/sda2 (lvm required for the project)
```
```
$ lvcreate -L 9G lvm -n root-arch
$ lvcreate -L 5G lvm -n home-arch
$ lvcreate -L 500M lvm -n swap-arch
$ lvcreate -L 10G lvm -n root-deb
$ lvcreate -L 4.5G lvm -n home-deb
$ lvcreate -L 500M lvm -n swap-deb
```
## `lsblk` to list existing partition, you should format your boot partition :

`$ mkfs.fat -F32 /dev/sda1` 
 
## Then the others partitions :

```
$ mkfs.ext4 /dev/lvm/root-arch 
$ mkfs.ext4 /dev/lvm/home-arch 
$ mkfs.ext4 /dev/lvm/root-deb 
$ mkfs.ext4 /dev/lvm/home-deb 
$ mkswap /dev/lvm/swap-arch 
$ swapon /dev/lvm/swap-arch
```
## Mount partitions :

```
$ mount /dev/lvm/root-arch /mnt
$ mkdir -p /mnt/home-arch
$ mount /dev/lvm/home-arch /mnt/home-arch
$ mkdir -p /mnt/home-deb
$ mount /dev/lvm/home-deb /mnt/home-deb
$ mkdir -p /mnt/boot
$ mount /dev/sda1 /mnt/boot 
```
## install the base packages using pacstrap :
```
$ pacstrap /mnt base linux linux-firmware nano lvm2 ctlboot base base-devel
$ genfstab -U /mnt >> /mnt/etc/fstab 
```
## change root :
`$ arch-chroot /mnt /bin/bash`
## make a password :
`$ passwd` 
## set up localtime : 
`$ nano /etc/locale.gen`
go to "`#en_US.UTF-8`" & remove the '#'.
```
$ locale-gen
$ nano /etc/locale.conf -> LANG=en_US.UTF-8
$ tzselect
$ ln -s /usr/share/zoneinfo/Europe/Paris /etc/localtime
$ hwclock --systohc --utc
```
## initramfs :
```
$ nano /etc/mkinitcpio.conf
base : HOOKS=(base udev ... block lvm2 filesystem)
to : HOOks=(base systemd ... block lvm2 filesystem)
```
## Init linux & boot loader :
```
$ mkinitcpio -p linux 
$ bootctl install 
$ nano /boot/loader/loader.conf
```
```
default arch
timeout 3
editor 0
```
## create the Arch entry by doing :

`$ nano /boot/loader/entries/arch-lvm.conf`
```
title Arch Linux (LVM) 
linux /vmlinuz-linux 
initrd /initramfs-linux.img 
options root=/dev/lvm/root-arch rw 
```
## To finish :
```
Ctrl + D
$ umount -R /mnt 
$ reboot
```
## (no need to poweroff because Linux will boot using first what's given to him and not the ISO). Remove ISO(after checking it work properly) & reboot select arch linux.
