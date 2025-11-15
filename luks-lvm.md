# 🛡️ LVM on LUKS Installation Guide 

This guide outlines the steps for creating an encrypted Linux system using LUKS on top of LVM for both UEFI and Legacy BIOS environments.

---

## 1. 💽 Disk Partitioning 

Target Disk: `/dev/sda`

- **UEFI**

| Partition | Size | Type Code | Purpose |
| :--- | :--- | :--- | :--- |
| **P1** | 500MB | `ef00` | **EFI System Partition (ESP)** (Mount point: `/boot/efi`) |
| **P2** | 1GB | `8300` | **Boot Partition** (Mount point: `/boot`) |
| **P3** | Rest | `8309` | **LUKS/LVM Container** |


- **Legacy BIOS/CSM**

| Partition | Size | Type Code | Purpose |
| :--- | :--- | :--- | :--- |
| **P1** | 5MB | `ef02` | **BIOS Boot** (if you use GPT instead of MBR) |
| **P2** | 1GB | `8300` | **Boot Partition** (Mount point: `/boot`) |
| **P3** | Rest | `8309` | **LUKS/LVM Container** |

---

## 2. 🔒 LUKS Encryption Setup

1.  **Format the LVM partition with LUKS1:**
    ```bash
    cryptsetup -s 512 luksFormat --type luks1 /dev/sda3
    ```
    *You will be prompted to enter and verify your passphrase.*

2.  **Open the LUKS container:**
    ```bash
    cryptsetup luksOpen /dev/sda3 slackpv0
    ```
    *The decrypted volume is now accessible at `/dev/mapper/slackpv0`.*

---

## 3. 🗄️ LVM Setup

1.  **Create a Physical Volume (PV):**
    ```bash
    pvcreate /dev/mapper/slackpv0
    ```

2.  **Create a Volume Group (VG):**
    ```bash
    vgcreate slack /dev/mapper/slackpv0
    ```

3.  **Create Logical Volumes (LVs):**
    ```bash
    # Swap LV (4GB)
    lvcreate -C y -L 4GB -n swap slack
    # Root LV (20GB)
    lvcreate -C n -L 50GB -n root slack
    # Home LV (Remaining space)
    lvcreate -C n -l 100%FREE -n home slack
    ```

4.  **Activate and Prepare Swap:**
    ```bash
    vgscan --mknodes
    vgchange -ay
    lvscan
    mkswap /dev/slack/swap
    ```

---

## 4. ⚙️ Post-Installation: Chroot and Initrd

**❗ IMPORTANT:** Do **NOT** reboot after the installation finishes.

1.  **Prepare for `chroot`:** (Assuming the new system is mounted at `/mnt`)
    ```bash
    cp -R /etc/resolv.conf /mnt/etc/resolv.conf
    cd /mnt
    chroot /mnt /bin/bash -l
    ```

2.  **Create the custom Initial RAM Disk (`initrd`):**
    The `initrd` must contain the necessary logic to prompt for the passphrase, open the LUKS container (`-C`), and mount the LVM root volume (`-r`).

    ```bash
    cd /boot
    rm initrd.gz # Remove existing initrd if present

    mkinitrd -c -k kernel_version -f ext4 -r /dev/slack/root -m jbd2:mbcache:ext4 -C /dev/sda3 -L -u -o /boot/initrd.gz
    ```
    *Replace `kernel_version` with your installed kernel (e.g., `6.1.0`).*

---

## 5. 💻 GRUB Bootloader Configuration

1.  **Get the LUKS Partition UUID:**
    Find the UUID of `/dev/sda3` using the `blkid` command:
    ```bash
    blkid /dev/sda3
    ```

2.  **Edit `/etc/default/grub`:**
    Modify these two lines, replacing the placeholder `UUID` with the actual UUID of `/dev/sda3`.

    ```bash
    GRUB_CMDLINE_LINUX="cryptdevice=UUID=xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxx:slackpv0 root=/dev/slack/root cryptkey=/slackpv.keyfile resume=/dev/slack/swap"
    GRUB_ENABLE_CRYPTODISK=y
    ```

3.  **Install GRUB (UEFI):**
    *(Ensure `/dev/sda2` is mounted as `/boot/efi` before this step)*
    ```bash
    grub-install --target=x86_64-efi --efi-directory=/boot/efi --bootloader-id=grub --recheck
    ```

    **Install GRUB (Legacy BIOS/CSM):**
    ```bash
    grub-install /dev/sda
    ```
    

4.  **Generate Configuration:**
    ```bash
    grub-mkconfig -o /boot/grub/grub.cfg
    ```

---

## 6. 🔄 Final Reboot

1.  **Exit the `chroot` environment:**
    ```bash
    exit
    ```

2.  **Reboot the System:**
    ```bash
    reboot
    ```

# 🔑 LVM on LUKS with Keyfile Setup Guide 

This process uses a keyfile stored securely within the initial RAM disk (`initrd.gz`) to automatically unlock the LUKS partition at boot.

---

## 1. 💽 Disk Partitioning 

Target Disk: `/dev/sda`

- **UEFI**

| Partition | Size | Type Code | Purpose |
| :--- | :--- | :--- | :--- |
| **P1** | 500MB | `ef00` | **EFI System Partition (ESP)** (Mount point: `/boot/efi`) |
| **P2** | 1GB | `8300` | **Boot Partition** (Mount point: `/boot`) |
| **P3** | Rest | `8309` | **LUKS/LVM Container** |


- **Legacy BIOS/CSM**

| Partition | Size | Type Code | Purpose |
| :--- | :--- | :--- | :--- |
| **P1** | 5MB | `ef02` | **BIOS Boot** (if you use GPT instead of MBR) |
| **P2** | 1GB | `8300` | **Boot Partition** (Mount point: `/boot`) |
| **P3** | Rest | `8309` | **LUKS/LVM Container** |

---

## 2. 🔒 LUKS Encryption Setup

1.  **Format the LVM partition with LUKS1:**
    ```bash
    cryptsetup -s 512 luksFormat --type luks1 /dev/sda3
    ```
    *You will be prompted to enter and verify your passphrase.*

2.  **Open the LUKS container:**
    ```bash
    cryptsetup luksOpen /dev/sda3 slackpv0
    ```
    *The decrypted volume is now accessible at `/dev/mapper/slackpv0`.*

---

## 3. 🗄️ LVM Setup

1.  **Create a Physical Volume (PV):**
    ```bash
    pvcreate /dev/mapper/slackpv0
    ```

2.  **Create a Volume Group (VG):**
    ```bash
    vgcreate slack /dev/mapper/slackpv0
    ```

3.  **Create Logical Volumes (LVs):**
    ```bash
    # Swap LV (4GB)
    lvcreate -C y -L 4GB -n swap slack
    # Root LV (50GB)
    lvcreate -C n -L 50GB -n root slack
    # Home LV (Remaining space)
    lvcreate -C n -l 100%FREE -n home slack
    ```

4.  **Activate and Prepare Swap:**
    ```bash
    vgscan --mknodes
    vgchange -ay
    lvscan
    mkswap /dev/slack/swap
    ```

---

## 4. ⚙️ Post-Installation: Chroot and Initrd

**❗ IMPORTANT:** Do **NOT** reboot after the installation finishes.

1.  **Prepare for `chroot`:** (Assuming the new system is mounted at `/mnt`)
    ```bash
    cp -R /etc/resolv.conf /mnt/etc/resolv.conf
    cd /mnt
    chroot /mnt /bin/bash -l
    ```

2.  **Create and add the Keyfile to LUKS:**
    This keyfile will be used to unlock the partition automatically.
    ```bash
    dd bs=512 count=4 if=/dev/urandom of=/slackpv.keyfile
    cryptsetup luksAddKey /dev/sda3 /slackpv.keyfile
    chmod 000 /slackpv.keyfile
    ```

3.  **Patch the `mkinitrd` Environment to handle the Keyfile:**
    These steps customize the script that runs inside the initial RAM disk, ensuring it uses the keyfile to unlock the drive early in the boot process.

    ```bash
    mkdir /tmp/initrd-tree
    tar xpzf /usr/share/mkinitrd/initrd-tree.tar.gz -C /tmp/initrd-tree

    cd /tmp/initrd-tree
    # Download the required patch (specific to this mkinitrd system)
    wget https://raw.githubusercontent.com/lukazu25/cheat-sheet/main/assets/patch.diff

    patch init < key_file_in_the_initrd_and_drive_unlocked_by_grub.diff
    mv /slackpv.keyfile ./ # Move the keyfile into the build directory

    tar cpzf /usr/share/mkinitrd/initrd-tree.tar.gz * # Re-package the patched environment
    ```

4.  **Create the new `initrd.gz`:**
    This command includes the keyfile (`-K /slackpv.keyfile`) in the final `initrd.gz` image.

    ```bash
    cd /boot
    rm initrd.gz # Remove existing initrd if present

    mkinitrd -c -k kernel_version -f ext4 -r /dev/slack/root -m jbd2:mbcache:ext4 -C /dev/sda3 -L -K /slackpv.keyfile -u -o /boot/initrd.gz
    ```

---

## 5. 💻 GRUB Bootloader Configuration

1.  **Get the LUKS Partition UUID:**
    Find the UUID of `/dev/sda3` using the `blkid` command:
    ```bash
    blkid /dev/sda3
    ```

2.  **Edit `/etc/default/grub`:**
    Modify these two lines, replacing the placeholder `UUID` with the actual UUID of `/dev/sda3`.

    ```bash
    GRUB_CMDLINE_LINUX="cryptdevice=UUID=xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxx:slackpv0 root=/dev/slack/root cryptkey=/slackpv.keyfile resume=/dev/slack/swap"
    GRUB_ENABLE_CRYPTODISK=y
    ```

3.  **Install GRUB (UEFI):**
    *(Ensure `/dev/sda2` is mounted as `/boot/efi` before this step)*
    ```bash
    grub-install --target=x86_64-efi --efi-directory=/boot/efi --bootloader-id=grub --recheck
    ```
 
    **Install GRUB (Legacy BIOS/CSM):**
    ```bash
    grub-install /dev/sda
    ```
       

5.  **Generate Configuration:**
    ```bash
    grub-mkconfig -o /boot/grub/grub.cfg
    ```

---

## 6. 🔄 Final Reboot

1.  **Exit the `chroot` environment:**
    ```bash
    exit
    ```

2.  **Reboot the System:**
    ```bash
    reboot
    ```
