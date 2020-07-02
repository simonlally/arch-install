# ARCH LINUX INSTALLATION v1.0.2
updated as of 2/20/2020
## This is not meant to replace the installation guide on archlinux.org but meant to be used in tandem.
(0) check that we are booted into a UEFI environment

`ls /sys/firmware/efi`

if the directory is empty or doesn’t exist then we are not booted into a UEFI environment, if there’s files there you’re good to go


## Pre-installation
Check internet
`ping 1.1.1.1`


Enabling WiFi (if you are using ethernet you can skip this) :

`ip addr`

`wifi-menu <WiFi adapter name>`
And then connect your WiFi information

(1) partitioning
Show installed discs on system with
`lsblk`

## Partitioning and formatting
Layout: 300M for EFI partition (EFI System type), swap partition size of installed ram (Linux swap), and rest of the drive goes to arch, cfdisk will automatically make it ext4, so no need to change it

To start formatting:
`cfdisk /dev/sda`

Assuming /dev/sda1 is your EFI partition, /dev/sda2 is your swap partition and /dev/sda3 is your filesystem continue and you have these partitions setup, continue to format:

Now the partitions are created but not formatted
For efi
`mkfs.fat -F32 /dev/sda1`

For swap
`mkswap /dev/sda2`

For our main partition
`mkfs.ext4 /dev/sda3`

Let’s start installing!

Mount and chroot.

First enable swap partition

`swapon /dev/sda2`

## Installation

Now mount our main partition so we can write to it

`mount /dev/sda3 /mnt`

install the base arch system

`pacstrap /mnt base linux linux-firmware`

Once the base is installed generate an FStab [file that tells linux where our disks and partitions are, and what they are used for]

`genfstab -U -p /mnt /mnt/etc/fstab`

Almost there...

Chroot time

`arch-chroot /mnt`

Set hostname, choose whatever you want for name and omit the quotation marks

`echo “name” > /etc/hostname`

Set the locale (you can substitute nano for vim)

`vim /etc/locale.gen`

Go in and uncomment the EN_US lines and save the file (there are two of them, remove the # symbols)

Generate the conf with

`locale-gen`

`echo LANG=en_US.UTF-8 > /etc/locale.conf`

`export LANG=en_US.UTF-8`

Set the time zone , remove a local time zone file if it already exists

`rm /etc/localtime`

To check time zones:
ls /usr/share/zoneinfo

ls /usr/share/zoneinfo/America

Link it

`ln -s /usr/share/zoneinfo/America/New_York /etc/localtime`

`hwclock —-systohc —-utc`

Upgrade packages w/ pacman if you haven’t already

`pacman -Syu`

set rootpw

`passwd`

Add a user

`useradd -mg users -G wheel,storage,power -s /bin/bash <name>`

`passwd <name>`

install sudo

`pacman -S sudo`

Allow sudo when we boot into the installation

`EDITOR=nano visudo` or `visudo` if you are comfortable with vi

Uncomment `%wheel ALL=(ALL) ALL`

Save and exit

## Grub installation

Let’s install grub

`pacman -S grub efibootmgr dosfstools mtools`

Make the EFI directory and mount it

`mkdir /boot/EFI`

`mount /dev/sda1 /boot/EFI`

`grub-install —-target=x86_64-efi —-bootloader-id=grub_uefi —-recheck`

`grub-mkconfig -o /boot/grub/grub.cfg`

download dhcpcd and enable the service so we have internet when we reboot into our new system
`pacman -S dhcpcd`
now enable it:
`systemctl enable dhcpcd.service`

I also recommend you download a texteditor as the arch base package doesn't come with one.  
`pacman -S vim`

`exit`

## Post-installation

now reboot into your new OS! see my guide for installing a desktop environment.

`reboot`
