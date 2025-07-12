# Rotating Machinery Vibration – IoT Edge Platform

## 📖 Overview
A full-stack edge-to-cloud solution that monitors and diagnoses vibration on rotating assets (motors, pumps, gearboxes) in real-time.  
It collects multi-axis accelerometer data, performs on-device FFT & anomaly detection, streams results via MQTT, stores time-series in InfluxDB, and presents a live "Digital Twin" dashboard with alarm status.

Designed for industrial equipment diagnostics, research in predictive maintenance, and final-year engineering projects.

---

## 🏗️ System Architecture
```txt
[Sensors 3-axis ACC] → STM32/ESP32 MCU ──► MQTT over Wi-Fi/ETH ──► Raspberry Pi IoT Gateway
                                                 │
                                   ┌─────────────┴─────────────┐
                                   │   Edge Services (Docker)  │
                                   │  • Mosquitto (MQTT v5)    │
                                   │  • Python Signal Worker   │
                                   │  • InfluxDB 3.0           │
                                   │  • Grafana Dashboard      │
                                   └─────────────┬─────────────┘
                                                 │
                Local GUI (PyQt / C#) ◄──────────┘  (WebSocket/REST)
```

### Optional Cloud Extension:
```txt
Raspberry Pi → MQTT Bridge → AWS IoT Core / Azure IoT Hub
                                   ↓
                          Cloud Storage + Cloud Dashboard
```

---

## 🔩 Hardware Requirements
| Component | Recommended | Details |
|-----------|-------------|---------|
| Sensor | ADXL355/356 (I²C) or IEPE 100 mV/g | 3-axis, sampling ≥ 10kHz |
| MCU | ESP32-S3, STM32F411 | Handles sampling + pre-FFT (fixed-point or CMSIS-DSP) |
| Gateway | Raspberry Pi 4/5, 4GB+ RAM + SSD | Runs Docker stack for edge services |
| Power | 5V/3A + UPS HAT (for Pi) | Optional backup for stable runtime |

---

## 🧱 Software Stack Overview

### Embedded Firmware
- Language: **C / C++**
- Frameworks: `ESP-IDF`, `STM32 HAL + CMSIS-DSP`
- Functionality:
  - High-speed ADC sampling
  - Windowing (Hanning, Blackman)
  - FFT (512~1024 pts)
  - RMS, Peak-Peak
  - MQTT JSON publish

### Edge Gateway (Raspberry Pi)
| Layer | Tech | Notes |
|-------|------|-------|
| OS | Ubuntu Server / Pi OS Lite 64-bit | Headless setup with SSH access |
| MQTT Broker | Mosquitto 2.0 (Docker) | TLS, authentication, v5 support |
| Processing | Python 3.12 + asyncio + numpy + scipy | Real-time processing + alert logic |
| Time-series DB | InfluxDB 3.0 (Docker) | High-speed ingest via Line Protocol or FlightSQL |
| Dashboard | Grafana 11 OSS | JSON panel import, threshold alerts, twin view |
| Local GUI | PyQt6/pyqtgraph OR C# WinForms/WPF | Optional, for HMI or operator dashboard |

---

## ⚙️ Setup Guide – Raspberry Pi
### 1. Install OS and Docker
```bash
sudo apt update && sudo apt full-upgrade -y
curl -fsSL https://get.docker.com | sh
sudo usermod -aG docker $USER
sudo reboot
```

### 2. Clone and Launch Services
```bash
git clone https://github.com/yourorg/rotor-vib-edge.git
cd rotor-vib-edge
sudo docker compose up -d
```

### 3. Access Services
| Service | URL | Default Credentials |
|--------|-----|---------------------|
| Grafana | http://<PI_IP>:3000 | admin / admin |
| InfluxDB | http://<PI_IP>:8086 | In-browser UI or Flux queries |
| MQTT Broker | mqtt://<PI_IP>:1883 | Via Mosquitto or Paho |

---

## 📂 Repository Structure
```
rotor-vib-edge/
├─ firmware/                 # Firmware code (ESP32/STM32)
│   ├─ esp32/                # ESP-IDF project
│   └─ stm32/                # STM32CubeMX + CMSIS-DSP
├─ edge/                    # Edge infrastructure (Docker)
│   ├─ docker-compose.yml
│   ├─ mosquitto/
│   ├─ grafana/
│   └─ influxdb/
├─ worker/                  # Python-based data processor
│   ├─ main.py              # Main async loop
│   └─ thresholds.json      # Dynamic thresholds config
├─ gui/                     # GUI app (PyQt or C#)
│   ├─ pyqt/                # PyQt6 + pyqtgraph GUI
│   └─ csharp/              # C# WPF interface (optional)
│   └─ qtcpp/               # Qt C++ GUI
└─ README.md
```

---

## 🧪 Firmware Sample JSON Packet
```json
{
  "ts": 1720838400123,
  "ax": [1.02, -0.98, 0.25],
  "rms": 0.12,
  "fft": [0.01, 0.03, 0.06, 0.04],
  "rpm": 1480
}
```
> Published to MQTT topic `vib/sensor/<id>` at 10~100Hz.

---

## 📊 Grafana Dashboards
- **Machine Overview**: RMS, FFT peak hold
- **Digital Twin**: Model turns green → amber → red with alarm
- **Trend Panel**: Historical diagnostics
- **System Panel**: Pi stats, network, MQTT traffic

---

## 🚨 Alert Logic
Configurable in `thresholds.json`:
```python
if rms > RMS_LIMIT:
    publish("alerts/crit", {"level": "red", "rms": rms})
elif fft_peak > VIB_THRESH:
    publish("alerts/warn", {"level": "amber", "fft": fft_peak})
```

---

## 🖥️ GUI App (Optional)
We support **three** front-end options:

### A. PyQt6 GUI
- Real-time chart with **pyqtgraph**
- FFT waterfall & RMS gauges
- Twin panel (SVG/QtQuick) bound to MQTT

### B. C# WPF
- LiveCharts / ScottPlot
- MQTT client via MQTTnet
- Twin state via XAML DataTriggers

### C. Qt 6 C++ Module
| Item | Library / Tool | Notes |
|------|----------------|-------|
| UI | Qt 6 Widgets / QtQuick | QtQuick + 3D twin |
| Chart | Qt Charts | For FFT & RMS |
| MQTT | Qt Mqtt | `QMqttClient` subscribe `vib/#` |
| Build | CMake | Cross-compile to ARM |

Minimal example:
```cpp
#include <QtWidgets>
#include <QtMqtt>

int main(int argc, char *argv[]) {
    QApplication app(argc, argv);
    QMqttClient client;
    client.setHostname("192.168.1.100");
    client.setPort(1883);
    client.connectToHost();

    QChartView view(new QChart);
    QWidget w;
    QVBoxLayout l(&w);
    l.addWidget(&view);
    w.show();
    return app.exec();
}
```

---

## 🧑‍🔧 Development Notes
- Python: `black` + `ruff`
- C#: `dotnet format`, StyleCop
- IDEs: VS Code, STM32CubeIDE, Visual Studio

---

## 🧠 Future Extensions
- Edge ML (PyTorch → ONNX)
- Redundant sensors
- Temperature/IMU integration
- Cloud export
- OTA via Mender

---

## 🤝 Contribution
1. Fork
2. PR with conventional commits
3. Include tests

---

## 📄 License
MIT
