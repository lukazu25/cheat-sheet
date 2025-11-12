# 💻 KVM/QEMU/Virt-Manager Setup Guide

---

## 🐧 Debian/Ubuntu/Mint

1. Check Virtualization Extension

Run this command to check if VT-x (Intel) or AMD-V (AMD) is enabled in your BIOS. The output should be greater than 0.

```
egrep -c '(vmx|svm)' /proc/cpuinfo
```
⚠️ Note: If the output is 0, you must go into your computer's BIOS/UEFI settings and enable the virtualization technology (often labeled as VT-x or AMD-V).



2. Install QEMU and Virtual Machine Manager

Install all the necessary packages for KVM, QEMU, libvirt, and networking.

```
sudo apt update
sudo apt install qemu-kvm qemu-system qemu-utils python3 python3-pip libvirt-clients libvirt-daemon-system bridge-utils virtinst libvirt-daemon virt-manager -y
```


3. Verify Libvirtd Service Status

Ensure the primary virtualization management service is running.

```
sudo systemctl status libvirtd.service
```


4. Start Default Network

Start and enable the default NAT-based virtual network for your VMs to access the internet.

```
sudo virsh net-start default
sudo virsh net-autostart default
sudo virsh net-list --all
```


5. Add User to Required Groups

Add your current user ($USER) to the necessary groups to manage VMs without needing sudo every time. You must log out and log back in for these changes to take effect.

```
sudo usermod -aG libvirt $USER
sudo usermod -aG libvirt-qemu $USER
sudo usermod -aG kvm $USER
sudo usermod -aG input $USER
sudo usermod -aG disk $USER
```



## 🐧 Arch Linux

1. Install QEMU and Virtual Machine Manager

Use pacman to install the core virtualization packages and network utilities.

```
sudo pacman -S qemu virt-manager virt-viewer dnsmasq vde2 bridge-utils openbsd-netcat ebtables iptables libguestfs
```


2. Configure Libvirt Daemon

Edit the libvirt configuration file to allow users in the libvirt group to manage VMs.

Edit `/etc/libvirt/libvirtd.conf` and ensure the following lines are set (uncomment them if they are commented out):

```
unix_sock_group = "libvirt"
unix_sock_rw_perms = "0770"
```


3. Start and Enable Libvirtd Service

Enable and start the libvirt service to run at boot.

```
sudo systemctl enable libvirtd.service
sudo systemctl start libvirtd.service
```


4. Add User to libvirt Group

Add your current user to the libvirt group.

```
sudo usermod -a -G libvirt $(whoami)
newgrp libvirt
```
💡 Tip: The newgrp libvirt command temporarily activates the new group membership in your current shell session. For a permanent change across all terminals, you should log out and log back in.
