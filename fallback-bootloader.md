# 💡 Fallback Bootloader 

When you run the standard grub-install command with a bootloader ID (e.g., `--bootloader-id=grub`), it does two things:

1. It places the GRUB EFI binary at a vendor-specific path on the EFI System Partition (ESP), such as `/EFI/grub/grubx64.efi`
2. It uses the efibootmgr utility to register a new entry, named "grub," in the UEFI firmware's non-volatile RAM (NVRAM).


The Problem: When NVRAM Entries Fail

The problem comes from hardware where the custom NVRAM entry may be lost or ignored:

* NVRAM Corruption/Reset: If you update the firmware, unplug the motherboard's battery (CMOS reset), or sometimes even unplug the drive, the custom NVRAM entry for "grub" can be deleted.
* "Blind" Booting: Some UEFI firmware is older or poorly implemented. If it fails to read the NVRAM entries, it will look for a single, hardcoded default path to boot from the disk.

This default path is:

* \EFI\BOOT\BOOTX64.EFI (on the ESP)

**The Solution: The Fallback Bootloader**

By performing the copy operation to this hardcoded path, you ensure that if the custom "grub" entry is lost, the firmware can still find a working bootloader (your GRUB file) at the standard fallback location.

**Conclusion**

You should always run these commands on a UEFI installation using GRUB

```
mkdir -p /boot/efi/EFI/boot
```

```
cp /boot/efi/EFI/grub/grubx64.efi /boot/efi/EFI/boot/bootx64.efi
```

This is a small, easy step that provides maximum compatibility and redundancy against common UEFI firmware issues.
