
# Yocto Build Guide for Raspberry Pi 4 – Edge Server for Vibration Monitoring

## 📌 Why Yocto?
Yocto Project enables you to create a **custom Linux distribution** tailored for embedded systems. For an industrial IoT gateway like Raspberry Pi 4 used in rotating machinery monitoring, Yocto gives you:
- Full control over packages: Only include what you need (Mosquitto, Qt, Python)
- Small image size, fast boot (typically < 500 MB)
- Predictable builds with version control (BitBake recipes)
- Easy OTA (Over-The-Air) integration for future update (SWUpdate, Mender)
- Secure, production-ready filesystem (read-only root, systemd)

---

## 🏗️ Yocto Build Process (for Raspberry Pi 4 – 64-bit)

### 🔧 1. Install Build Dependencies
```bash
sudo apt update
sudo apt install -y gawk wget git diffstat unzip zstd chrpath socat cpio python3 python3-pip python3-subunit mesa-common-dev libsdl1.2-dev xterm gcc-multilib build-essential locales
```

---

### 📁 2. Clone Layers
```bash
mkdir ~/yocto && cd ~/yocto

git clone -b kirkstone git://git.yoctoproject.org/poky
git clone -b kirkstone https://github.com/raspberrypi/meta-raspberrypi.git
git clone -b kirkstone https://github.com/openembedded/meta-openembedded.git
git clone -b 6.7 https://code.qt.io/yocto/meta-qt6.git
```

---

### ⚙️ 3. Initialize Build Environment
```bash
cd poky
source oe-init-build-env build-rpi
```

---

### 📝 4. Configure `bblayers.conf`
Add these entries:
```
${TOPDIR}/../meta-raspberrypi
${TOPDIR}/../meta-openembedded/meta-oe
${TOPDIR}/../meta-openembedded/meta-python
${TOPDIR}/../meta-openembedded/meta-networking
${TOPDIR}/../meta-qt6
```

---

### 🧩 5. Configure `local.conf`
```conf
MACHINE = "raspberrypi4-64"
DISTRO_FEATURES:append = " systemd wifi"
VIRTUAL-RUNTIME_init_manager = "systemd"

IMAGE_INSTALL:append = " mosquitto python3 python3-pip python3-paho-mqtt python3-numpy qtbase qtcharts qtmqtt qtquick3d"

EXTRA_IMAGE_FEATURES = "debug-tweaks ssh-server-openssh"
```

---

### 🧪 6. Add GUI App Recipe (Optional)
Create `meta-rotor/recipes-apps/rotor-gui/rotor-gui.bb`
```bitbake
SUMMARY = "GUI Qt C++ App for Vibration Monitoring"
LICENSE = "MIT"
SRC_URI = "git://github.com/yourorg/rotor-vib-edge.git;protocol=https"

S = "${WORKDIR}/git/gui/qtcpp"

inherit qt6-cmake pkgconfig

DEPENDS += "qtbase qtcharts qtmqtt qtquick3d"

do_install() {
  install -d ${D}${bindir}
  install -m 0755 ${B}/rotor-gui ${D}${bindir}/
}
```

Also provide `rotor-gui.service` and enable it via `SYSTEMD_SERVICE:${PN}`.

---

### 🧱 7. Build Image
```bash
bitbake-layers add-layer ../meta-rotor
bitbake core-image-minimal  # or your custom .bb image
```

Output image (flash to SD/SSD):
```
tmp/deploy/images/raspberrypi4-64/*.wic.bz2
```

---

## ✅ Flash & Run
```bash
bzcat edge-iot-image-*.wic.bz2 | sudo dd of=/dev/sdX bs=8M status=progress
```
On first boot: verify services like `mosquitto`, `rotor-gui` with `systemctl`.

---

## 📘 Notes
- Do not build Yocto directly on Pi. Use PC (16 GB+ RAM).
- After boot, you can SSH, run GUI on HDMI or remotely.
- Ideal for research, lab, and industrial field deployment.

