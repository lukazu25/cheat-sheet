# 🐧 Minimal Linux Distro Build Guide

This guide details the steps to compile a custom Linux kernel and BusyBox userspace into a minimal, bootable disk image using Docker and QEMU.

> 📝 **Note:** The first command (`docker run`) is executed on your host machine. All subsequent commands should be run inside the Docker container unless explicitly noted.

---

## 1. ⚙️ Environment Setup & Dependencies

| Context | Command | Purpose |
| :--- | :--- | :--- |
| **HOST** | `docker run --privileged -it ubuntu` | Starts a privileged Ubuntu container for the build environment. |
| **DOCKER** | `apt update` | Updates the package list. |
| **DOCKER** | `apt install git vim make gcc libncurses-dev flex bison bc cpio libelf-dev libssl-dev syslinux dosfstools` | Installs essential tools (compiler, kernel libs, Syslinux) for the entire process. |
| **DOCKER** | `mkdir boot_files` | Creates the central directory for the kernel and ramdisk. |
| **DOCKER** | `mkdir m` | Creates a temporary mount point for the boot image. |

---

## 2. 🧱 Kernel Compilation

| Context | Command | Purpose |
| :--- | :--- | :--- |
| **DOCKER** | `git clone --depth one https://github.com/torvalds/linux.git` | Clones the latest Linux kernel source (shallow). |
| **DOCKER** | `cd linux` | Enters the source directory. |
| **DOCKER** | `make menuconfig` | **ACTION:** Configure the kernel. Ensure the **64-bit kernel** option is selected, then Exit and Save. |
| **DOCKER** | `make -j$(nproc)` | Compiles the kernel using the **maximum number of available CPU cores** for faster compilation. |
| **DOCKER** | `cp arch/x86/boot/bzImage ../boot_files/` | Copies the compiled compressed kernel binary. |
| **DOCKER** | `cd ..` | Returns to the main directory. |

---

## 3. 📦 BusyBox and Initramfs Creation

| Context | Command | Purpose |
| :--- | :--- | :--- |
| **DOCKER** | `git clone --depth one https://git.busybox.net/busybox` | Clones the BusyBox source code. |
| **DOCKER** | `cd busybox` | Enters the BusyBox directory. |
| **DOCKER** | `make menuconfig` | **ACTION:** Configure BusyBox. Navigate to **Settings -> Build static binary** and **enable it**. Exit and Save. |
| **DOCKER** | `make -j$(nproc)` | Compiles BusyBox using the **maximum number of available CPU cores**. |
| **DOCKER** | `mkdir -p ../boot_files/initramfs` | Creates the directory structure for the initial ramdisk. |
| **DOCKER** | `make install CONFIG_PREFIX=../boot_files/initramfs` | Installs compiled BusyBox utilities into the ramdisk structure. |
| **DOCKER** | `cd ../boot_files/initramfs` | Moves into the initramfs structure. |
| **DOCKER** | `vim init` | Creates the mandatory **init** script. |
| **DOCKER** | **Init File Content:**<br>```#!/bin/sh exec /bin/sh``` | This script launches a shell immediately upon execution. |
| **DOCKER** | `chmod +x init` | Makes the script executable. |
| **DOCKER** | `find . | cpio -o -H newc > ../init.cpio` | Creates the final compressed ramdisk file (`init.cpio`). |
| **DOCKER** | `cd ..` | Moves back up to `boot_files`. |

---

## 4. 💾 Bootloader Setup (Syslinux)

| Context | Command | Purpose |
| :--- | :--- | :--- |
| **DOCKER** | `dd if=/dev/zero of=boot bs=1M count=50` | Creates a 50MB file (`boot`) for the disk image. |
| **DOCKER** | `mkfs -t fat boot` | Formats the image with the FAT filesystem. |
| **DOCKER** | `syslinux boot` | Installs the Syslinux bootloader onto the image. |
| **DOCKER** | `mount boot m` | Mounts the image file to transfer boot files. |
| **DOCKER** | `cp bzImage init.cpio m/` | Copies the kernel and ramdisk into the mounted boot image. |
| **DOCKER** | `umount m` | Unmounts the image file. |

---

## 5. 🖥️ Testing with QEMU

| Context | Command | Purpose |
| :--- | :--- | :--- |
| **HOST** | `docker cp <container_id>:/boot_files/boot .` | **REPLACE `<container_id>`** with your actual Docker ID. Copies the bootable image to your host machine. |
| **HOST** | `qemu-system-x86_64 boot` | Boots the image using the QEMU emulator. |
| **QEMU** | **At Syslinux Prompt:**<br>`/bzImage -initrd=/init.cpio` | Instructs Syslinux to load the kernel and the initial ramdisk. |

---

If the process is successful, the kernel will boot, and you will be dropped into the minimal BusyBox shell!


