# 🚀 Install Latest NVIDIA Driver on Debian

These steps use the extrepo tool to easily enable the official NVIDIA repositories for the latest drivers.

1. Install extrepo Package
```
sudo apt install extrepo
```
2. Enable NVIDIA-CUDA Repository
```
sudo extrepo enable nvidia-cuda
```
3. Update Repository Configuration

Edit the extrepo configuration file `/etc/extrepo/config.yaml` and uncomment the lines for `contrib` and `non-free` components to allow the installation of proprietary drivers.

4. Install NVIDIA Drivers

  - **For Newer Cards**
  
  ```
  sudo apt install nvidia-open
  ```
  - **For Older Cards**
  
  ```
  sudo apt install nvidia-driver
  ```

5. Enable Secure Boot for the NVIDIA Module

Since the NVIDIA kernel module is built locally via DKMS, it is unsigned and will be blocked by Secure Boot until its public key is enrolled.

- **Import the DKMS Public Key**

This command registers the key for enrollment upon the next boot. You will be prompted to create a one-time password (passphrase). Remember this password; you will need it in the next step.

```
sudo mokutil --import /var/lib/dkms/mok.pub
```
- **Reboot and Enroll the Key**

* Reboot your system: sudo reboot
* During boot, the MOK Management screen will appear.
* Select "Enroll MOK" -> "Continue".
* Select "Yes" to enroll the key.
* Enter the password/passphrase you created in the mokutil step.
* Select "Reboot".

6. Verify Installation

After the system boots up successfully:

- **Check Secure Boot State**
```
sudo mokutil --sb-state
```
Output should say: **"SecureBoot enabled"**

- **Check NVIDIA Driver Status**
```
nvidia-smi
```
This command should run and display information about your NVIDIA GPU, confirming the driver is loaded.
