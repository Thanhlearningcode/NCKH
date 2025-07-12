# Yocto Build Guide – Raspberry Pi 4 IoT Edge Image

> This guide walks you through building a custom Yocto image (kirkstone branch) for **Raspberry Pi 4 Model B (64‑bit)** that includes Mosquitto, Python 3 runtime, Qt 6 (Qt Charts, Qt MQTT), and your custom vibration‑monitoring GUI application.

---

## 1  📦 Prerequisites (Build Host)

| Requirement                                                               | Recommended                                                |
| ------------------------------------------------------------------------- | ---------------------------------------------------------- |
| OS                                                                        | Ubuntu 22.04 LTS 64‑bit (or Debian 11)                     |
| RAM                                                                       | ≥ 16 GB (physical)                                         |
| Disk                                                                      | ≥ 100 GB free (fast SSD)                                   |
| Packages                                                                  | \`sudo apt install -y gawk wget git diffstat unzip zstd \\ |
| chrpath socat cpio python3 python3-pip python3-subunit mesa-common-dev \\ |                                                            |
| libsdl1.2-dev xterm gcc-multilib build-essential locales\`                |                                                            |

```bash
# verify locale
sudo locale-gen en_US.UTF-8
sudo update-locale LANG=en_US.UTF-8
```

---

## 2  📂 Clone Poky & Layers

```bash
mkdir ~/yocto && cd ~/yocto

# Poky (Yocto kirkstone LTS)
git clone -b kirkstone git://git.yoctoproject.org/poky

# Raspberry Pi BSP layer
git clone -b kirkstone https://github.com/raspberrypi/meta-raspberrypi.git

# OpenEmbedded layers for extra packages
git clone -b kirkstone https://github.com/openembedded/meta-openembedded.git

# Qt 6 layer
git clone -b 6.7 https://code.qt.io/yocto/meta-qt6.git
```

```
~/yocto/
├─ poky/
├─ meta-raspberrypi/
├─ meta-openembedded/
└─ meta-qt6/
```

---

## 3  🛠️ Initialize Build Environment

```bash
cd poky
source oe-init-build-env build-rpi
```

The prompt changes to `build-rpi` directory containing `conf/`.

### 3.1  Edit `conf/bblayers.conf`

Append the absolute paths of extra layers:

```conf
BBLAYERS ?= " \
  ${TOPDIR}/../poky/meta \
  ${TOPDIR}/../poky/meta-poky \
  ${TOPDIR}/../poky/meta-yocto-bsp \
  ${TOPDIR}/../meta-raspberrypi \
  ${TOPDIR}/../meta-openembedded/meta-oe \
  ${TOPDIR}/../meta-openembedded/meta-python \
  ${TOPDIR}/../meta-openembedded/meta-networking \
  ${TOPDIR}/../meta-qt6 \
  "
```

### 3.2  Edit `conf/local.conf`

```conf
# MACHINE selection
MACHINE ?= "raspberrypi4-64"

# 64‑bit AArch64 tuning
TUNE_FEATURES_append = " crc neon cortexa72"

# Enable systemd & Wi‑Fi firmware
DISTRO_FEATURES:append = " systemd wifi"
VIRTUAL-RUNTIME_init_manager = "systemd"
DISTRO_FEATURES_BACKFILL_CONSIDERED = "sysvinit"

# Image packages
IMAGE_INSTALL:append = " \
    mosquitto \
    mosquitto-clients \
    python3 \
    python3-pip \
    python3-numpy python3-scipy \
    python3-paho-mqtt \
    qtbase qtcharts qtquick3d qtmqtt \
    "

# Optional: include your GUI app recipe
IMAGE_INSTALL:append = " rotor-gui "

# SSH enabled default
EXTRA_IMAGE_FEATURES = "debug-tweaks ssh-server-openssh"

# Resize rootfs on first boot (raspi)
ENABLE_SPI = "1"
ENABLE_I2C = "1"
```

---

## 4  📜 Add Custom GUI App Recipe

Create directory `meta-rotor/recipes-apps/rotor-gui/` and files:

**`rotor-gui.bb`**

```bitbake
SUMMARY = "Rotating Machinery GUI"
LICENSE = "MIT"
SRC_URI = "git://github.com/yourorg/rotor-vib-edge.git;protocol=https;branch=main"

S = "${WORKDIR}/git/gui/qtcpp"

inherit qt6-cmake pkgconfig

DEPENDS += "qtbase qtcharts qtmqtt qtquick3d"

FILES:${PN} += "/usr/bin/rotor-gui"

do_install() {
    install -d ${D}${bindir}
    install -m 0755 ${B}/rotor-gui ${D}${bindir}/
}
```

(Add `meta-rotor` to `BBLAYERS`.)

**Systemd service**: place at `recipes-apps/rotor-gui/rotor-gui.service`

```ini
[Unit]
Description=Rotating Machinery GUI
After=network.target mosquitto.service

[Service]
ExecStart=/usr/bin/rotor-gui
Restart=always

[Install]
WantedBy=multi-user.target
```

And reference in recipe:

```bitbake
SYSTEMD_SERVICE:${PN} = "rotor-gui.service"
```

---

## 5  🏃 Build Image

```bash
bitbake-layers add-layer ../meta-rotor
bitbake core-image-minimal        # test build (optional)
bitbake edge-iot-image            # your custom image name
```

*First build may take hours.*

Output image located at:

```
build-rpi/tmp/deploy/images/raspberrypi4-64/edge-iot-image-raspberrypi4-64.wic.bz2
```

Flash to SD:

```bash
bzcat edge-iot-image-*.wic.bz2 | sudo dd of=/dev/sdX bs=8M status=progress
```

---

## 6  🚀 First Boot Checklist

1. Insert SD, power Pi.
2. Discover via `ssh root@raspberrypi4`. Password default: `root` (or set in `local.conf`).
3. Verify services:

```bash
systemctl status mosquitto
systemctl status rotor-gui
```

4. MQTT broker listening on `1883`. Connect GUI (or Grafana) via LAN.

---

## 7  🔄 OTA Updates (Optional)

* Integrate **SWUpdate** or **Mender** layers for A/B partition updates.
* GUI app can update via package feed or docker if containerized.

---

## 8  📚 References

* Yocto Project Mega‑Manual – kirkstone
* meta-raspberrypi README
* meta‑qt6 layer docs
