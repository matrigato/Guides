## Arch Linux installation guide

_**IMPORTANT:** This is what worked for me. It should be enough in most cases, but there is no guarantee this will work for you. I wrote this guide mostly to myself. Most of it is taken directly from the [Arch Wiki](https://wiki.archlinux.org/index.php/Installation_Guide), with some modifications to fix some issues I had faced in previous installations and to make the whole process easier. The Wiki is still the best source of information on the subject and this guide is not meant to replace it. Follow at your own risk._

### Intro

Arch Linux is one of the most popular minimalist Linux distributions. It is versatile and bleeding edge, focusing on simplicity, with a default installation that covers only a minimal base system and expects the end user to configure and use it. This makes it popular among DIY enthusiasts and hardcore Linux users.

Arch Linux is a rolling release distro and has its own package manager - pacman. Aiming to provide a cutting-edge operating system, Arch never misses out to have an up-to-date repository. The fact that it provides a minimal base system gives you a choice to install it even on low-end hardware and then install only the required packages over it.

Also, its one of the most popular OS for learning Linux from scratch. If you like to experiment with a DIY attitude, you should give Arch Linux a try. It’s what many Linux users consider a core Linux experience.

In this guide, we will be installing a basic Arch Linux system using the full disk to a computer or virtual machine.

## Pre-installation

### Prerequisites

You will need:

* A x86_64 (i.e. 64 bit) computer or VM
* At least 512 MB of RAM (recommended 2 GB)
* At least 1 GB of free disk space (recommended 20 GB)
* An internet connection
* A USB drive with minimum 2 GB of storage capacity (not needed for VM)

You could do a minimal installation with the minimum requirements, but the recommended values are better for basic usage and running a GUI.

### 1. Download the ISO

Download the Arch ISO from the [Arch Linux official website](https://www.archlinux.org/download/). You could download it from one of the mirrors listed, but it is recommended to do a BitTorrent Download.

### 2. Create a live USB

_Note: If you are installing Arch Linux on a VM, skip this step and boot directly into the ISO image._

#### In Linux

If you are on Linux, you can use any of the following commands to create a live USB. Replace `/path/to/archlinux.iso` with the path where you have downloaded the ISO file, and replacing `/dev/sdx` with your drive, e.g. `/dev/sdb` (Do not append a partition number, so do not use something like `/dev/sdb1`) in the example below. You can get your drive information using `lsblk` or `fdisk -l` command. When running `lsblk`, also make sure the drive isn't mounted, and unmount it if necessary.

_**Warning:** This will destroy all data on `/dev/sdx`. If you accidentally indicate your HD, you may be in trouble._

* using `cat`:

```
cat path/to/archlinux.iso > /dev/sdx
```

* using `cp`:

```
cp path/to/archlinux.iso /dev/sdx
```

* using `dd`:

```
dd bs=4M if=/path/to/archlinux.iso of=/dev/sdx status=progress && sync
```

#### In Windows

On Windows, the recommended tool to create a live USB is Rufus. [This guide](https://itsfoss.com/live-usb-antergos/) shows how to create a live USB of Antergos Linux using Rufus. Since Antergos is based on Arch, you can follow the same tutorial.

### 3. Boot up Arch Linux

Shut down your PC, insert your USB and boot your system. To enter a boot menu, keep pressing its key while booting. Which key you need to press depends on your system, it can be something like F2, F10, F12 or ESC.

Once you boot from your live USB, select Boot Arch Linux (x86_64). After some checks, you should be logged with the root user. If something went wrong and it asks you for your login, just type `root`.

### 4. Preparing to install

#### 4.1. Set the Keyboard Layout

The default console keymap is US. Available layouts can be listed with:

```
ls /usr/share/kbd/keymaps/**/*.map.gz
```

To modify the layout, append a corresponding file name to `loadkeys`, omitting path and file extension. For example, to set a [German](https://en.wikipedia.org/wiki/File:KB_Germany.svg) keyboard layout:

```
loadkeys de-latin1
```

Console fonts are located in `/usr/share/kbd/consolefonts/` and can likewise be set with `setfont`.

#### 4.2. Verify the boot mode

If UEFI mode is enabled on an UEFI motherboard (most modern ones), Archiso will boot Arch Linux accordingly via systemd-boot. To verify this, list the efivars directory:

```
ls /sys/firmware/efi/efivars
```

If the directory does not exist, the system may be booted in BIOS mode. If it exists, the system is in UEFI mode. Remember which mode your system is in.

#### 4.3. Connect to the internet

For future reference, it is possible to list network interfaces with:

```
ip link
```

The first letters in an interface's name represent its function: `en` for wired/Ethernet, `wl` for wireless/WLAN. `lo` is the loop device and is not used in making network connections.

If you are using a wired connection, you should already be connected to the Internet. The installation image enables dhcpcd (for Dynamic IP address) for wired network devices on boot.

If you are using a wireless connection, running `wifi-menu` and configuring the connection should be enough for now. If your network isn't listed, try closing and opening `wifi-menu` until it appears. Keep in mind that this is only temporary, we will install NetworkManager in the actual system later.

You can check your internet connection by using the ping command:

```
ping -c5 google.com
```

It may need some seconds to connect if you use it immediately after setting up the connection, so it doesn't always work on the first try.

#### 4.4. Update the system clock

Use `timedatectl` to ensure the system clock is accurate:

```
timedatectl set-ntp true
```

To check the service status, use `timedatectl status`.

### 5. Partition the disks, create and mount the file systems

When recognized by the live system, disks are assigned to a block device such as `/dev/sda` or `/dev/nvme0n1`. To identify these devices, use `lsblk` or `fdisk -l`. Results ending in `rom`, `loop` or `airoot` may be ignored.

```
fdisk -l
```
_Note: This guide will assume that `/dev/sda` is the disk you wish to partition. Important to note is that the name convention for Raspberry PI drive storage usually is `/dev/mmcblk0` and for some types of hardware RAID cards can be `/dev/cciss`. Change it if you are using another._

The following partitions are **required** for a chosen device:

* One partition for the root directory `/`.
* If UEFI is enabled, an EFI system partition.

This guide won't cover separating `/` into optional partitions like `/home`, but will cover the creation of a swap partition (acts like extra RAM on disk, but is slower than the main RAM, useful to avoid running out of memory). If you don't know what size your swap partition should be, [It's FOSS has some useful suggestions](https://itsfoss.com/swap-size/).

Run the following command:

```
cfdisk /dev/sda
```

In `cfdisk` use the arrow keys to navigate and the enter key to select. Now follow the instructions for your boot mode:

#### UEFI

In UEFI mode, if you are greeted by a screen asking you to select the label type, in most cases you should choose `gpt`. If this doesn't happen, the device already has a partition table.

In the new screen, the partitions are listed. In a new system, you should only have "Free space". If this is not a new system and you want to overwrite older partitions, remember to `Delete` them.

Now we will create the partitions. First, select `New`. You will be prompted to enter the partition size. Type 512M and press enter. Select `Type` from the bottom menu and choose `EFI System` partition type. I'll assume this EFI System partition is `/dev/sda1`.

Now, let's create the swap partition. Do almost the same thing as before: Select "Free space", `New`, type desired swap size, select `Type` and choose `Linux swap`. I'll assume this swap partition is `/dev/sda2`.

Finally, the root partition, your main one. Select `New` -> Size: rest of free space -> `Type` as `Linux filesystem`. I'll assume this partition is `/dev/sda3`.

Write the changes to the drive by selecting `Write` and typing `yes`. Now exit by selecting `Quit`.

Now we can create the file system for the EFI system and root partitions. You should create a `FAT32` file system for `/dev/sda1` (EFI) and an `ext4` one for `/dev/sda3` (root).

```
mkfs.fat -F32 /dev/sda1
mkfs.ext4 /dev/sda3
```

We should also initialize and mount the swap:

```
mkswap /dev/sda2
swapon /dev/sda2
```

Now mount the file system on the root partition to `/mnt`:

```
mount /dev/sda3 /mnt
mkdir /mnt/boot
mkdir /mnt/boot/efi
mount /dev/sda1 /mnt/boot/efi
```

If you are using a more complex partitioning scheme, create any remaining mount points (such as /mnt/home) with `mkdir` and mount their corresponding partitions. `genfstab` will later detect mounted file systems and swap space.

#### BIOS

In BIOS mode, if you are greeted by a screen asking you to select the label type, in most cases you should choose `dos`. If this doesn't happen, the device already has a partition table.

In the new screen, the partitions are listed. In a new system, you should only have "Free space". If this is not a new system and you want to overwrite older partitions, remember to `Delete` them.

Now we will create the partitions. Let's start with the root partition, your main one. Select `New`. You will be prompted to enter the partition size. Be sure to leave enough room to create another partition for your swap space by subtracting it from the total.

Next, you will be asked if the partition should be primary or extended. Select `primary`. Now make the partition bootable by selecting `Bootable`. For this guide, I'll assume this partition is `/dev/sda1`.

Now, let's create the swap partition. Do almost the same thing as before: Select "Free space", `New`, use the remainder of the space on the drive, `primary`. However, you shouldn't make this partition bootable. This one has an extra step: The partition type needs to be changed from `83 Linux` to `82 Linux swap / Solaris`. Select `Type` on the swap partition and select `82 Linux swap / Solaris`. I'll assume this partition is `/dev/sda2`.

Write the changes to the drive by selecting `Write` and typing `yes`. Now exit by selecting `Quit`.

Now we can create the file system for the root partition, `/dev/sda1`. Here we will use the `ext4` file system:

```
mkfs.ext4 /dev/sda1
```

We should also initialize and mount the swap:

```
mkswap /dev/sda2
swapon /dev/sda2
```

Now mount the file system on the root partition to `/mnt`:

```
mount /dev/sda1 /mnt
```

If you are using a more complex partitioning scheme, create any remaining mount points (such as /mnt/home) with `mkdir` and mount their corresponding partitions. `genfstab` will later detect mounted file systems and swap space.

## Installation

As a last step before installation, you may want to edit `/etc/pacman.d/mirrorlist` and move the mirrors closest to you to the top of the list, usually the ones from your country. Mirrors on top are given priority, and one close to you can speed up package downloads. This file will be copied to the new system by `pacstrap`, so it is worth getting right.

Now, read this whole section before doing it. The basic install uses the `pacstrap` script to install the `base` package, Linux kernel and firmware for common hardware:

```
pacstrap /mnt base linux linux-firmware
```

However, the `base` package does not include all tools from the live installation, so installing other packages may be necessary for a fully functional base system. In particular, consider installing:

* userspace utilities for the management of file systems that will be used on the system,
* utilities for accessing RAID or LVM partitions,
* specific firmware for other devices not included in `linux-firmware`,
* software necessary for networking: `dhcpcd` for DHCP, `networkmanager` for WiFi, etc.,
* a text editor,
* packages for accessing documentation in man and info pages: `man-db`, `man-pages` and `texinfo`.

To install other packages or package groups, append the names to the `pacstrap` command above (space separated) or use `pacman` while chrooted into the new system. For comparison, packages available in the live system can be found in [packages.x86_64](https://gitlab.archlinux.org/archlinux/archiso/-/blob/master/configs/releng/packages.x86_64).

My personal recommendation (if using ethernet) is:

```
pacstrap /mnt base base-devel linux linux-firmware man-db man-pages vim nano dhcpcd
```

I would also add `networkmanager` if I needed WiFi (I think you wound't need `dhcpcd` then, but I'm not sure). Of course, you won't need `base-devel` if you don't need tools for building (compiling and linking), or `dhcpcd` if you are using a static IP intead of DHCP. These are just my recommendations, install whatever you want. If you mess something up now, like not installing `dhcpcd` when you needed it and end up without internet connection after finishing the installation, you can boot though the installation USB again, mount the root partition, chroot into it and install the needed packages through `pacman`.

## Configure the system

### Fstab

Generate an fstab file (use `-U` or `-L` to define by UUID or labels, respectively):

```
genfstab -U /mnt >> /mnt/etc/fstab
```

Check the resulting `/mnt/etc/fstab` file, and edit it in case of errors.

### Chroot

Change root into the new system:

```
arch-chroot /mnt
```

### Time zone

Set the time zone. Replace `Region` and `City` with the corresponding ones found by autocomplete (pressing TAB):

```
ln -sf /usr/share/zoneinfo/Region/City /etc/localtime
```

Don't worry about it not being already updated when running `timedatectl status`, this should take effect after rebooting.

Run `hwclock` to generate `/etc/adjtime`:

```
hwclock --systohc
```

This command assumes the hardware clock is set to UTC, which I guess may cause issues when dual-booting with Windows. Maybe setting ntp (covered by the guide later) should be enough to fix it.

### Localization

Edit `/etc/locale.gen` and uncomment `en_US.UTF-8 UTF-8` and other needed locales. Generate the locales by running:

```
locale-gen
```

Create the `locale.conf` file, and set the `LANG` variable accordingly (assuming English US, change as needed):

```
echo LANG=en_US.UTF-8 > /etc/locale.conf
export LANG=en_US.UTF-8
```

If you set the keyboard layout, make the changes persistent in `vconsole.conf` (German keyboard example):

```
echo KEYMAP=de-latin1 > /etc/vconsole.conf
```

Both locale and timezone settings can be changed later, when you are using your Arch Linux system.

### Network configuration

Create the hostname file with the desired name for your computer in the network. Replace `myhostname` with the name you chose:

```
echo myhostname > /etc/hostname
```

Add matching entries to hosts (run `nano /etc/hosts`):

```
127.0.0.1	localhost
::1		localhost
127.0.1.1	myhostname.localdomain	myhostname
```

If the system has a permanent IP address, it should be used instead of 127.0.1.1.

Complete the network configuration for the newly installed environment, that includes installing your preferred network management software.

### Root password

Create a root password:

```
passwd
```

### Boot loader

You can probably skip this if you are configuring a dual boot in a system that already has rEFInd installed, "it just werks". If that's not the case, this will show you how to install GRUB.

This is on of the most important steps and is needed to actually be able to boot the system. If you forgot to do it for some reason, there is still hope, you just need to chroot into the root partition after mounting it with the installation USB. The steps are different for BIOS and UEFI:

#### UEFI

Make sure that you are still using arch-chroot. Install required packages:

```
pacman -S grub efibootmgr
```

Install grub:

```
grub-install --target=x86_64-efi --bootloader-id=GRUB --efi-directory=/boot/efi
grub-mkconfig -o /boot/grub/grub.cfg
```

#### BIOS

Install grub package:

```
pacman -S grub
```

Install grub, make sure you use your disk name if it isn't `/dev/sda` (don’t put the disk number `sda1`, just the disk name `sda`):

```
grub-install /dev/sda
grub-mkconfig -o /boot/grub/grub.cfg
```

## Reboot
Exit the chroot environment by typing `exit` or pressing `Ctrl+d`.

Optionally manually unmount all the partitions with `umount -R /mnt`: this allows noticing any "busy" partitions, and finding the cause with `fuser`.

Finally, shutdown the machine by typing `shutdown now` or `poweroff`: any partitions still mounted will be automatically unmounted by systemd. Remember to remove the installation media, turn on the computer and then login into the new system with the root account.

## Post-installation

The first thing to do in your new system is to test the internet connection:

```
ping -c5 google.com
```

If it fails because you need to enable DHCP, run this:

```
systemctl start dhcpcd
systemctl enable dhcpcd
```

If you need to use WiFi, enable `networkmanager` and configure it through `nmtui`:

```
systemctl start NetworkManager.service
systemctl enable NetworkManager.service
nmtui
```

Also remember to enable ntp:

```
timedatectl set-ntp true
```

(TODO: create user, manage sudo, GUI, etc.)
