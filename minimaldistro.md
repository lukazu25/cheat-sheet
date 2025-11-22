# 🐧 Minimal Linux Distro Build Guide

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
    mkdir distro mounted
    ```

---

## 2. 🧱 Kernel Compilation (The Core)

This section compiles the Linux kernel and prepares the `bzImage` file.

1.  **Get Source:** Clones the latest kernel source and enters the directory.
    ```bash
    git clone --depth one https://github.com/torvalds/linux.git && cd linux
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
    cp arch/x86/boot/bzImage ../distro/
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
    git clone --depth 1 https://git.busybox.net/busybox && cd busybox
    ```
2.  **Configure:** **ACTION:** Use the menu to configure BusyBox. Navigate to **Settings -> Build static binary** and **enable it**. Exit and Save.
    ```bash
    make menuconfig
    ```
3.  **Compile & Install:** Compiles BusyBox and installs the utilities statically into the ramdisk folder.
    ```bash
    make -j$(nproc)
    mkdir -p ../distro/initramfs
    make install CONFIG_PREFIX=../distro/initramfs
    ```
4.  **Create Init Script:** Creates the mandatory script that launches a shell on boot and makes it executable.
    ```bash
    cd ../distro/initramfs
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
    mount boot mounted
    cp bzImage init.cpio mounted/
    umount mounted
    ```

---

## 5. 🖥️ Testing with QEMU

### Host Machine Steps

1.  **Copy Image Out:** Copy the final bootable image to your host machine.
    ```bash
    docker cp <container_id>:/distro/boot .  # REPLACE <container_id>
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

# 🛠️ Making Linux Distro with Buildroot

This guide contains the commands and steps used to create a simple Linux system for embedded purposes using the Buildroot tool.

---

## 1. ⚙️ Environment Setup & Dependencies

1.  **Update and Install Dependencies (Ubuntu/Debian):**
    ```bash
    sudo apt update
    sudo apt install wget build-essential libncurses-dev bc unzip bzip2 libelf-dev libssl-dev extlinux 
    ```

2.  **Download and Extract Buildroot:**
    ```bash
    # Get the latest version link from the Buildroot website
    wget [http://buildroot.org/downloads/](http://buildroot.org/downloads/) 
    # Replace [buildroot-file.tar.gz] with the name of the downloaded file
    tar xf [buildroot-file.tar.gz]
    rm [buildroot-file.tar.gz]

    cd [buildroot-directory]
    ```

---

## 2. 🧱 Configuration & Compilation (The Core Build)

1.  **Configure System (Buildroot menuconfig):** Launches the interactive configuration menu.
    ```bash
    make menuconfig
    ```
2.  **Configuration Actions:**
    * Navigate to **Target options** and change the **Target Architecture** to **x86\_64**.
    * Navigate to **Kernel** and select **build a kernel**.
    * Enter **Kernel configuration** and select **Use the architecture default configuration**.
    * Exit and **Save** the new configuration.
3.  **Compile System:** Starts the entire build process.
    ```bash
    make -j$(nproc)
    ```

---

## 3. 📦 Root Filesystem Preparation

1.  **Navigate and Copy Build Artifacts:**
    ```bash
    cd output/images
    cp bzImage rootfs.tar ~
    cd ~
    ```
2.  **Create Assembly Directory and Extract Rootfs:**
    ```bash
    mkdir distro
    mv bzImage rootfs.tar distro
    cd distro
    tar xf rootfs.tar
    rm rootfs.tar
    cd ..
    ```

---

## 4. 💾 Bootloader Setup & Disk Image Preparation

1.  **Create Disk Image and Mount Point:**
    ```bash
    truncate -s 100M boot.img
    mkdir mounted
    ```
2.  **Format Filesystem:**
    ```bash
    mkfs boot.img 
    ```
3.  **Mount and Install Bootloader:**
    ```bash
    sudo mount boot.img mounted
    sudo extlinux --install mounted
    ```
4.  **Transfer Files:**
    ```bash
    sudo cp -r distro/* mounted
    ```
5.  **Unmount Image:**
    ```bash
    sudo umount mounted
    ```

---

## 5. 🖥️ Testing with QEMU 

1.  **Boot with QEMU:**
    ```bash
    qemu-system-x86_64 -hda boot.img
    ```

2.  **Load Kernel & Root Device (At the 'boot:' prompt):**
    ```
    /bzImage root=/dev/sda
    ```
    (Log in as **`root`** when the system finishes booting.)
    
