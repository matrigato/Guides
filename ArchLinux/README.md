## Arch Linux installation guide

### Intro

Arch Linux is one of the most popular minimalist Linux distributions. It is versatile and bleeding edge, focusing on simplicity, with a default installation that covers only a minimal base system and expects the end user to configure and use it. This makes it popular among DIY enthusiasts and hardcore Linux users.

Arch Linux is a rolling release distro and has its own package manager - pacman. Aiming to provide a cutting-edge operating system, Arch never misses out to have an up-to-date repository. The fact that it provides a minimal base system gives you a choice to install it even on low-end hardware and then install only the required packages over it.

Also, its one of the most popular OS for learning Linux from scratch. If you like to experiment with a DIY attitude, you should give Arch Linux a try. It’s what many Linux users consider a core Linux experience.

In this guide, we will be installing a basic Arch Linux system using the full disk to a computer or virtual machine.

## Installing Arch Linux

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

*Note: If you are installing Arch Linux on a VM, skip this step and boot directly into the ISO image.*

#### In Linux

If you are on Linux, you can use dd command to create a live USB. Replace `/path/to/archlinux.iso` with the path where you have downloaded the ISO file, and replacing `/dev/sdx` with your drive, e.g. `/dev/sdb`. (Do not append a partition number, so do not use something like `/dev/sdb1`) in the example below. You can get your drive information using `lsblk` or `fdisk -l` command.

**Warning:** This will destroy all data on `/dev/sdx`. If you accidentally indicate your HD, you may be in trouble.

```
dd bs=4M if=/path/to/archlinux.iso of=/dev/sdx status=progress && sync
```

#### In Windows

On Windows, the recommended tool to create a live USB is Rufus. [This guide](https://itsfoss.com/live-usb-antergos/) shows how to create a live USB of Antergos Linux using Rufus. Since Antergos is based on Arch, you can follow the same tutorial.

### 3. Boot up Arch Linux

Shut down your PC, insert your USB and boot your system. To enter a boot menu, keep pressing its key while booting. Which key you need to press depends on your system, it can be something like F2, F10, F12 or ESC.

Once you boot from your live USB, select Boot Arch Linux (x86_64). After some checks, you should be logged with the root user. If something went wrong and it asks you for your login, just type `root`.

### 4. Set the Keyboard Layout

The default console keymap is US. Available layouts can be listed with:

```
ls /usr/share/kbd/keymaps/**/*.map.gz
```
```
# ls /usr/share/kbd/keymaps/**/*.map.gz
```
