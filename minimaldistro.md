## 🐧 Minimal Linux Distro Build Guide: Sequential Summary

This guide details the steps to compile a custom Linux kernel and BusyBox userspace into a minimal, bootable disk image. The build environment is isolated using Docker, and the final image is tested with QEMU.

> **⚠️ Important Context Note:** The first command is run on the **HOST** machine. All other steps are executed **INSIDE THE DOCKER CONTAINER** unless explicitly marked as **HOST**.

---

## 1. ⚙️ Environment Setup & Dependencies

### Host Machine Step

* **Start Build Container:**
    ```bash
    docker run --privileged -it ubuntu
    ```

### Docker Container Steps

* **Install Build Tools:**
    ```bash
    apt update && apt install -y git vim make gcc libncurses-dev flex bison bc cpio libelf-dev libssl-dev syslinux dosfstools
    ```
* **Create Working Directories:**
    ```bash
    mkdir boot_files m
    ```

---

## 2. 🧱 Kernel Compilation (The Core)

This section compiles the Linux kernel and prepares the `bzImage` file.

1.  **Get Source:** Clones the latest kernel source and enters the directory.
    ```bash
    git clone --depth one [https://github.com/torvalds/linux.git](https://github.com/torvalds/linux.git) && cd linux
    ```
2.  **Configure:** **ACTION:** Use the menu to configure the kernel. Ensure the **64-bit kernel** option is selected. Exit and Save.
    ```bash
    make menuconfig
    ```
3.  **Compile:** Compiles the kernel using all available CPU cores.
    ```bash
    make -j$(nproc)
    ```
4.  **Copy Kernel:** Copies the compiled compressed kernel binary to the staging area.
    ```bash
    cp arch/x86/boot/bzImage ../boot_files/
    ```
5.  **Exit Directory:**
    ```bash
    cd ..
    ```

---

## 3. 📦 BusyBox and Initramfs Creation (The Userspace)

This creates the minimal root filesystem and the initial ramdisk image (`init.cpio`).

1.  **Get BusyBox Source:** Clones the source code and enters the directory.
    ```bash
    git clone --depth one [https://git.busybox.net/busybox](https://git.busybox.net/busybox) && cd busybox
    ```
2.  **Configure:** **ACTION:** Use the menu to configure BusyBox. Navigate to **Settings -> Build static binary** and **enable it**. Exit and Save.
    ```bash
    make menuconfig
    ```
3.  **Compile & Install:** Compiles BusyBox and installs the utilities statically into the ramdisk folder.
    ```bash
    make -j$(nproc)
    mkdir -p ../boot_files/initramfs
    make install CONFIG_PREFIX=../boot_files/initramfs
    ```
4.  **Create Init Script:** Creates the mandatory script that launches a shell on boot and makes it executable.
    ```bash
    cd ../boot_files/initramfs
    vim init
    # Init File Content: #!/bin/sh exec /bin/sh
    chmod +x init
    ```
5.  **Build Ramdisk:** Packs the ramdisk contents into the final compressed CPIO archive.
    ```bash
    find . | cpio -o -H newc > ../init.cpio
    ```
6.  **Exit Directory:**
    ```bash
    cd ..
    ```

---

## 4. 💾 Bootloader Setup & Disk Image Preparation

This combines the kernel and ramdisk into a 50MB bootable FAT image using Syslinux.

1.  **Create Disk Image:**
    ```bash
    dd if=/dev/zero of=boot bs=1M count=50
    ```
2.  **Format Filesystem:**
    ```bash
    mkfs -t fat boot
    ```
3.  **Install Bootloader:**
    ```bash
    syslinux boot
    ```
4.  **Transfer Files:** Mount the image, copy the kernel and ramdisk, then safely unmount.
    ```bash
    mount boot m
    cp bzImage init.cpio m/
    umount m
    ```

---

## 5. 🖥️ Testing with QEMU

### Host Machine Steps

1.  **Copy Image Out:** Copy the final bootable image to your host machine.
    ```bash
    docker cp <container_id>:/boot_files/boot .  # REPLACE <container_id>
    ```
2.  **Boot with QEMU:** Start the emulator using the image.
    ```bash
    qemu-system-x86_64 boot
    ```

### QEMU Prompt Step

* **Load Kernel & Ramdisk:** At the Syslinux prompt, instruct the bootloader to load the kernel and the initial ramdisk.
    ```
    /bzImage -initrd=/init.cpio
    ```

If successful, the kernel will boot, and you will be dropped into the minimal **BusyBox shell!**
