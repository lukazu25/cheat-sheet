# 🛠️ XLibre Installation Guide

XLibre is a fork of the X.Org Server and replaces it entirely. You **MUST** install the corresponding XLibre-compatible video and input drivers.

---

## 1. Debian / Ubuntu / Mint

### 1. Install `extrepo` and enable the XLibre repository
```bash
sudo apt install extrepo
sudo extrepo enable xlibre
```

### 2. Update package list
```bash
sudo apt update
```

### 3. Install the XLibre metapackage
```bash
sudo apt install xlibre
```

---

## 2. Arch / Artix / Manjaro

### 1. Uninstall conflicting X.Org packages (Use with caution)
```bash
sudo pacman -Rdd xorg-server xorg-server-common
```

### 2. Install dependencies
```bash
sudo pacman -S base-devel git meson
```

### 3. Install XLibre components (AUR)
```bash
yay -S xlibre-xserver-bootstrap
yay -S xlibre-input-libinput
yay -S xlibre-xserver
```

### 4. Install video drivers (Intel example)
```bash
yay -S xlibre-xf86-video-intel
```

---

## 3. Fedora / Red Hat / CentOS

### 1. Enable the XLibre COPR repository
```bash
sudo dnf copr enable kkofler/xlibre-xserver
```

### 2. Update and replace packages
```bash
sudo dnf upgrade
```

### 3. Install missing drivers (AMD example)
```bash
sudo dnf install xlibre-xf86-video-amdgpu
```

---

## 4. Gentoo Linux

### 1. Add the XLibre overlay
```bash
sudo eselect repository enable xlibre
sudo emerge --sync
```

### 2. Accept keywords for live ebuilds  
Add to: `/etc/portage/package.accept_keywords/xlibre`
```
*/*::xlibre **
```

### 3. Replace X.Org with XLibre
```bash
sudo emerge -va --nodeps --oneshot x11-base/xlibre-server::xlibre
```

### 4. Install XLibre-compatible drivers (libinput example)
```bash
sudo emerge -va x11-base/xlibre-input-libinput::xlibre
```

---

## ⚠️ Proprietary NVIDIA Driver Users (All Distros)

Create `/etc/X11/xorg.conf.d/20-nvidia.conf`:
```conf
Section "ServerFlags"
    Option "IgnoreABI" "true"
EndSection
```

> **NOTE:** Starting with version **25.0.0.16**, the proprietary NVIDIA driver is autodetected and requires **no configuration**.

---

# 🛠️ Compile XLibre From Source

Compiling XLibre from source.

---

## 1. Install Build Dependencies

### Debian / Ubuntu / Mint
```bash
sudo apt install \
  build-essential git meson ninja-build pkg-config xutils-dev \
  libx11-dev libxext-dev libxfixes-dev libxcursor-dev libxrender-dev \
  libxi-dev libfontenc-dev libdrm-dev libudev-dev \
  x11proto-dev xtrans-dev libpixman-1-dev libxkbcommon-x11-dev \
  libxfont-dev libxcvt-dev libepoxy-dev \
  x11proto-present-dev libxkbfile-dev libxshmfence-dev \
  libbsd-dev x11proto-xf86dri-dev \
  libgl1-mesa-dev libglu1-mesa-dev libgbm-dev \
  libxcb-shape0-dev libxcb-util-dev libxcb-icccm4-dev
```

### Fedora / Red Hat / CentOS
```bash
sudo dnf install @development-tools git meson ninja-build pkg-config \
  xorg-x11-util-macros \
  libX11-devel libXext-devel libXfixes-devel libXcursor-devel libXrender-devel \
  libXi-devel libfontenc-devel \
  libdrm-devel systemd-devel \
  pixman-devel libxkbfile-devel libxcvt-devel \
  mesa-libGL-devel mesa-libgbm-devel \
  libxcb-devel libxcb-util-devel libxcb-icccm-devel \
  libepoxy-devel xorg-x11-server-devel
```

### Arch / Artix / Manjaro
```bash
sudo pacman -S --needed base-devel git meson ninja pkg-config \
  libx11 libxext libxfixes libxcursor libxrender libxi libfontenc libdrm systemd \
  pixman libxkbfile libxcvt \
  mesa libgl \
  libxcb xcb-util xcb-util-image xcb-util-keysyms xcb-util-renderutil xcb-util-wm \
  libepoxy
```

---

## 2. Compile and Install XLibre Server

### 1. Clone the source
```bash
git clone git clone https://github.com/X11Libre/xserver.git
cd xserver
```

### 2. Switch to the XLibre branch
```bash
git checkout <xlibre-branch-name>
```

### 3. Configure the build
```bash
mkdir build
cd build
meson setup .. --prefix=/usr
```

### 4. Build and install
```bash
ninja -j$(nproc)
sudo ninja install
```

---

## 3. Compile and Install Input Driver (xf86-input-libinput)

### 1. Clone the driver
```bash
cd ../..
git clone https://github.com/X11Libre/xf86-input-libinput.git
cd xf86-input-libinput
```

### 2. Configure
```bash
mkdir build
cd build
meson setup .. --prefix=/usr
```
The specific instruction regarding the installation prefix can be summarized as follows:

If you installed the XLibre server to a non-standard location (e.g., `/usr/local/xlibre` instead of the standard `/usr`), you **MUST** set the `PKG_CONFIG_PATH` environment variable before running the `meson setup` command. This variable tells the Meson build system where to look for the XLibre dependency files (specifically, the `.pc` files used by pkg-config).

**Example for a non-standard prefix** (`/usr/local/xlibre`):
```bash
PKG_CONFIG_PATH=/usr/local/xlibre/lib/x86_64-linux-gnu/pkgconfig meson setup .. --prefix /usr/local/xlibre
```
For a **standard prefix** (`/usr`), the command remains simply:
```bash
meson setup .. --prefix=/usr
```

### 3. Build & install
```bash
ninja -j$(nproc)
sudo ninja install
```

## 4. Compile and Install AMD Driver (xf86-video-amdgpu)

### 1. Clone the driver
```bash
cd ..
git clone https://github.com/X11Libre/xf86-video-amdgpu.git
cd xf86-video-amdgpu
```

### 2. Configure
```bash
./autogen.sh
./configure --prefix=/usr
```

### 3. Build & install
```bash
make -j$(nproc)
sudo make install
```

---

# 🚀 Final Step

Reboot your system:
```bash
sudo reboot
```
