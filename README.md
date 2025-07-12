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

| Component | Recommended                        | Details                                               |
| --------- | ---------------------------------- | ----------------------------------------------------- |
| Sensor    | ADXL355/356 (I²C) or IEPE 100 mV/g | 3-axis, sampling ≥ 10kHz                              |
| MCU       | ESP32-S3, STM32F411                | Handles sampling + pre-FFT (fixed-point or CMSIS-DSP) |
| Gateway   | Raspberry Pi 4/5, 4GB+ RAM + SSD   | Runs Docker stack for edge services                   |
| Power     | 5V/3A + UPS HAT (for Pi)           | Optional backup for stable runtime                    |

---

## 🧱 Software Stack Overview

### Embedded Firmware

* Language: **C / C++**
* Frameworks: `ESP-IDF`, `STM32 HAL + CMSIS-DSP`
* Functionality:

  * High-speed ADC sampling
  * Windowing (Hanning, Blackman)
  * FFT (512\~1024 pts)
  * RMS, Peak-Peak
  * MQTT JSON publish

### Edge Gateway (Raspberry Pi)

| Layer          | Tech                                  | Notes                                            |
| -------------- | ------------------------------------- | ------------------------------------------------ |
| OS             | Ubuntu Server / Pi OS Lite 64-bit     | Headless setup with SSH access                   |
| MQTT Broker    | Mosquitto 2.0 (Docker)                | TLS, authentication, v5 support                  |
| Processing     | Python 3.12 + asyncio + numpy + scipy | Real-time processing + alert logic               |
| Time-series DB | InfluxDB 3.0 (Docker)                 | High-speed ingest via Line Protocol or FlightSQL |
| Dashboard      | Grafana 11 OSS                        | JSON panel import, threshold alerts, twin view   |
| Local GUI      | PyQt6/pyqtgraph OR C# WinForms/WPF    | Optional, for HMI or operator dashboard          |

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

| Service     | URL                    | Default Credentials           |
| ----------- | ---------------------- | ----------------------------- |
| Grafana     | http\://\<PI\_IP>:3000 | admin / admin                 |
| InfluxDB    | http\://\<PI\_IP>:8086 | In-browser UI or Flux queries |
| MQTT Broker | mqtt://\<PI\_IP>:1883  | Via Mosquitto or Paho         |

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

> This is published to MQTT topic `vib/sensor/<id>` at 10\~100Hz.

---

## 📊 Grafana Dashboards

* **Machine Overview**: Line graph of RMS, FFT peak hold
* **Digital Twin**: Model turns green → amber → red with alarm
* **Trend Panel**: Time-based history for diagnostics
* **System Panel**: Pi stats, network, MQTT traffic

---

## 🚨 Alert Logic

Alert thresholds are configurable in `thresholds.json` and applied in real-time:

```python
if rms > RMS_LIMIT:
    publish("alerts/crit", {"level": "red", "rms": rms})
elif fft_peak > VIB_THRESH:
    publish("alerts/warn", {"level": "amber", "fft": fft_peak})
```

Alerts are also visible in the Grafana alert panel or forwarded via webhook/email.

---

## 🖥️ GUI App (Optional)

We support **three** front‑end options; choose one or mix:

### A. PyQt6 GUI (Python)

* Real‑time charting with **pyqtgraph**
* FFT waterfall & RMS gauges
* Digital‑Twin panel (SVG/QtQuick) bound to MQTT data

### B. C# WPF

* LiveCharts 2 / ScottPlot for charts
* MQTT client (MQTTnet)
* Twin state model via XAML `DataTrigger` (green/amber/red)

### C. **Qt 6 C++ Module** (NEW)

> Suitable khi bạn cần hiệu năng cao, native trên Linux/Windows và dễ cross‑compile cho thiết bị ARM.

| Item         | Library / Tool                                                            | Ghi chú                                        |
| ------------ | ------------------------------------------------------------------------- | ---------------------------------------------- |
| UI Framework | **Qt 6.7 Widgets / QtQuick**                                              | Widgets dễ bảo trì; QtQuick + QML cho Twin 3‑D |
| Charting     | **Qt Charts** (LGPL)                                                      | `QLineSeries`, `QXYSeries` for FFT & RMS       |
| MQTT         | **Qt Mqtt** module                                                        | `QMqttClient` subscribe `vib/#`                |
| 3‑D Model    | **Qt 3D** hoặc import glTF via QtQuick3D                                  | Mô hình máy quay đổi màu theo cảnh báo         |
| Build        | **CMake** + `find_package(Qt6 COMPONENTS Widgets Charts Mqtt 3DCore ...)` | Cross‑compile to aarch64                       |

Minimal example:

```cpp
#include <QtWidgets>
#include <QtMqtt>

int main(int argc, char *argv[]) {
    QApplication app(argc, argv);
    QMqttClient client;
    client.setHostname("localhost");
    client.setPort(1883);
    client.setUsername("edgeuser");
    client.setPassword("pass");
    client.connectToHost();

    QChartView *view = new QChartView(new QChart);
    // append data points in slot connected to client.messageReceived()

    QWidget w; QVBoxLayout l(&w); l.addWidget(view);
    w.show();
    return app.exec();
}
```

Add new folder `gui/qtcpp/` containing full Qt C++ source & CMakeLists.txt.

### A. PyQt6 GUI

* Real-time chart (pyqtgraph)
* FFT + RMS numeric
* Twin Panel with live state

### B. C# WPF

* Realtime WPF charting (LiveCharts or ScottPlot)
* MQTT background worker
* Twin state model using XAML

---

## 🧑‍🔧 Development Notes

* Python formatting: `black`, lint: `ruff`
* C#: Use `dotnet format`, `StyleCop`
* Recommended IDEs:

  * VS Code (Python, PyQt)
  * STM32CubeIDE
  * Visual Studio 2022 (C# GUI)

---

## 🧠 Future Extensions

* Train anomaly model (PyTorch) → export ONNX → infer on edge
* Use multiple sensors per axis for redundancy
* Add temperature/IMU data
* Export data to cloud (Azure/AWS/GCP)
* OTA firmware update via Mender

---

## 🤝 Contribution

1. Fork repository
2. Submit PR from feature branch
3. Use conventional commits: `feat:`, `fix:`, `docs:`
4. Include test case if modifying processor

---

## 📄 License

MIT - see `LICENSE` file.
