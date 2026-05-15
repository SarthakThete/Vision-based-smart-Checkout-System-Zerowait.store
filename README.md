<div align="center">

<img src="https://img.shields.io/badge/ZeroWait-Smart%20Checkout-c8442a?style=for-the-badge&logo=raspberry-pi&logoColor=white" />

# ZeroWait — Autonomous IoT Smart Checkout System

**A modular, vision-first autonomous checkout ecosystem.**  
*No queues. No cashiers. No friction.*

[![Python](https://img.shields.io/badge/Python-3.11-3776AB?style=flat-square&logo=python&logoColor=white)](https://python.org)
[![Flask](https://img.shields.io/badge/Flask-2.x-000000?style=flat-square&logo=flask&logoColor=white)](https://flask.palletsprojects.com)
[![Raspberry Pi](https://img.shields.io/badge/Raspberry%20Pi-5-A22846?style=flat-square&logo=raspberry-pi&logoColor=white)](https://raspberrypi.com)
[![ESP32](https://img.shields.io/badge/ESP32-240MHz-E7352C?style=flat-square&logo=espressif&logoColor=white)](https://espressif.com)
[![Roboflow](https://img.shields.io/badge/Roboflow-Vision%20AI-purple?style=flat-square)](https://roboflow.com)
[![MQTT](https://img.shields.io/badge/MQTT-Mosquitto%20v2-660066?style=flat-square&logo=mqtt&logoColor=white)](https://mosquitto.org)
[![License](https://img.shields.io/badge/License-MIT-green?style=flat-square)](LICENSE)

<br/>

> **MIT World Peace University — SY B.Tech Autonomous Systems Project 2024–25**  
> **Saksham** `1032232749` · **Soham** `1032232422` · **Sarthak** `1032232607`

</div>

---

## Table of Contents

- [Overview](#overview)
- [Live Demo](#live-demo)
- [System Architecture](#system-architecture)
- [Hardware Components](#hardware-components)
- [Software Stack](#software-stack)
- [IoT & Network Layer](#iot--network-layer)
- [Computer Vision Pipeline](#computer-vision-pipeline)
- [Payment Integration](#payment-integration)
- [Performance Metrics](#performance-metrics)
- [Repository Structure](#repository-structure)
- [Getting Started](#getting-started)
  - [Prerequisites](#prerequisites)
  - [Hub Server Setup](#hub-server-setup)
  - [Raspberry Pi 5 (Frame) Setup](#raspberry-pi-5-frame-setup)
  - [ESP32 Firmware Flash](#esp32-firmware-flash)
- [Configuration](#configuration)
- [API Reference](#api-reference)
- [Future Scope](#future-scope)
- [Team](#team)
- [License](#license)

---

## Overview

**ZeroWait** is an end-to-end autonomous retail checkout ecosystem that replaces traditional POS queues with a real-time, computer-vision and IoT-driven self-checkout experience.

A customer places an item on the checkout tray. Within milliseconds:
1. The **load cell** detects a weight change and the **ESP32** fires a telemetry packet over **MQTT**
2. The **Raspberry Pi 5 edge node** captures a **2304×1296 camera frame** and sends it to the **Roboflow Cloud API**
3. The AI model classifies the item (99.1% mAP@50) and the **mass-visual correlation engine** cross-references the vision label against the weight to prevent spoofing
4. The item is added to the customer's cart on the **browser-native kiosk UI**
5. The customer taps **"Checkout & Pay"** — a **Razorpay** overlay handles payment in under 2 seconds


No cashier. No RFID tags. No proprietary hardware lock-in.

---

## Live Demo

> Real prototype running at MIT World Peace University — April 2025

<div align="center">

| Kiosk UI on iPad | Prototype in Action |
|:---:|:---:|
| ![ZeroWait Kiosk UI on iPad showing live camera feed with Your Bill panel and Pay Now button]![Kiosk UI](docs/images/img1.jpeg) | ![Student placing a grocery item onto the ZeroWait smart checkout tray with kiosk iPad visible]![Prototype](docs/images/img2.jpeg) |
| *Live 800×450 feed + bill panel on zerowait.store* | *Item placed on tray — ESP32 fires MQTT weight event* |

</div>

---

## System Architecture

### Full-Stack Software & Cloud Ecosystem

![ZeroWait Comprehensive Full-Stack Software and Cloud Ecosystem Diagram showing Edge Infrastructure with ESP32 and Kiosk, Store Network with MQTT and AI, and Payment and Billing Services via Razorpay and SMTP]![Architecture](docs/images/img4.jpeg)
```
┌─────────────────────────────────────────────────────────────────────┐
│                        SENSING LAYER (Edge)                          │
│  TAL220 Load Cell → HX711 24-bit ADC → ESP32 (MQTT Publisher)       │
│  Pi Camera 3 IMX708 (2304×1296 RAW) → Raspberry Pi 5 CSI-2          │
└──────────────────────────┬──────────────────────────────────────────┘
                           │ MQTT over Wi-Fi (2.4 GHz)
                           ▼
┌─────────────────────────────────────────────────────────────────────┐
│                    PROCESSING LAYER (Pi 5 Edge)                      │
│  Mosquitto Broker → Flask Web Streamer → MJPEG (800×450 relay)      │
│  V4L2 Video Engine → Roboflow Cloud API → AI Inference Engine        │
│  Mass-Visual Correlation → Cart State Manager                        │
└──────────────────────────┬──────────────────────────────────────────┘
                           │ HTTP Keep-Alive / WebSocket
                           ▼
┌─────────────────────────────────────────────────────────────────────┐
│                  APPLICATION LAYER (zerowait.store)                  │
│  Flask Hub Server → Kiosk Portal (React.js + Canvas)                │
│  Admin Portal → Inventory Dashboard → Transaction Logs               │
│  Razorpay Gateway → Webhook → S3 + PostgreSQL → SMTP Email           │
└─────────────────────────────────────────────────────────────────────┘
```

### Three-Layer Breakdown

| Layer | Components | Responsibility |
|---|---|---|
| **Edge / Sensing** | TAL220, HX711, ESP32, Pi Camera 3, Raspberry Pi 5 | Real-time capture & telemetry |
| **Processing** | Pi 5, Mosquitto, Flask Streamer, Roboflow API | Inference, weight correlation, video relay |
| **Application** | zerowait.store, Razorpay, S3, PostgreSQL, SMTP | Dashboard, payment, billing, analytics |

---

## Hardware Components

![ZeroWait Smart Checkout Advanced System Specifications and Architecture — showing Raspberry Pi 5 Edge AI Gateway, ESP32 Wireless Sensor Node, TAL220 Precise Weight Sensing Unit with HX711, and Pi Camera Module 3 Item Recognition Unit]![Hardware](docs/images/img3.jpeg)
### A — Edge AI Gateway: Raspberry Pi 5

| Spec | Value |
|---|---|
| SoC | Broadcom BCM2712 |
| CPU | Quad-core Arm Cortex-A76 @ 2.4 GHz |
| GPU | VideoCore VII |
| RAM | 4 GB / 8 GB LPDDR4X-4267 SDRAM |
| I/O Controller | Dedicated RP1 (manages CSI-2 + MQTT concurrently without thermal throttle) |
| Camera Interface | Dual CSI-2 Ports (MIPI 2-lane) |

The RP1 I/O controller is critical — it decouples camera DMA from CPU load, allowing concurrent 24 FPS video capture and MQTT message processing with CPU utilization held at ~28%.

### B — Wireless Sensor Node: ESP32

| Spec | Value |
|---|---|
| CPU | Xtensa® Dual-core 32-bit LX6 @ 240 MHz |
| Wi-Fi | 802.11 b/g/n (2.4 GHz) with WPA3 security |
| Bluetooth | BLE 4.2 / 5.0 |
| HX711 Interface | Serial (CLK + DOUT), hardware-timed sampling |
| MQTT Client | Paho, QOS1, Keep-alive 30s |
| Internal Buffer | Circular buffer for weight packet queuing |

> **Note on 2.4 GHz band:** The ESP32 is intentionally configured on the 2.4 GHz band to ensure stable communication in environments where a 5 GHz mobile hotspot band may fail or have reduced range.

### C — Precise Weight Sensing Unit

**TAL220 Load Cell (Cantilever Beam Design)**

The TAL220 is a single-point load cell operating on the **Cantilever Beam principle**. When a load is applied:
1. The beam deflects proportionally to the applied force
2. **Strain gauges** bonded to the beam surface change resistance (Wheatstone Bridge configuration)
3. The micro-volt differential output is amplified by the **HX711** 24-bit ADC (128× gain, temperature-compensated NTC)
4. Calibrated gram values (±0.01g resolution) are transmitted at 10 Hz via the ESP32

### D — Item Recognition Unit: Pi Camera Module 3

| Spec | Value |
|---|---|
| Sensor | Sony IMX708 — 11.9 MP |
| FOV | 120° (Widest) |
| Autofocus | Powered Phase-Detection AF |
| Interface | MIPI CSI-2 (2-lane) |
| Capture Resolution | 2304 × 1296 RAW BGR (4-channel) |
| Relay Resolution | 800 × 450 (72% bandwidth reduction) |

---

## Software Stack

### Dual-Portal Flask Architecture

```
zerowait.store (Central Hub)
├── /kiosk/<frame_id>          ← Customer-facing kiosk portal
│   ├── Live MJPEG feed        ← <canvas> WebSocket stream
│   ├── Real-time cart         ← Dynamic item detection updates
│   └── Razorpay checkout      ← JS overlay trigger
│
└── /admin                     ← Password-protected merchant portal
    ├── Shop management        ← Multi-store registration
    ├── Frame registration     ← Pi 5 pairing via device token
    ├── API key generation     ← Per-frame secure keys
    ├── Inventory dashboard    ← Add/edit items, pricing per kg or unit
    └── Transaction logs       ← GraphQL-queried, PostgreSQL-backed
```

### Technology Stack

| Layer | Technology |
|---|---|
| Backend | Python 3.11, Flask 2.x |
| Frontend | React.js, HTML5 Canvas, WebSocket |
| Database | PostgreSQL (transactions), Amazon S3 (PDF receipts) |
| Vision AI | Roboflow Cloud API (YOLOv8 backbone) |
| Messaging | Mosquitto MQTT v2.0, Secure WebSockets |
| Payments | Razorpay Standard Checkout + Webhooks |
| Email | Python `smtplib`, Gmail SMTP Relay |
| Inference Registry | Roboflow Inference Model Registry v2.1 |

---

## IoT & Network Layer

### IoT Architecture — 3-Layer System Diagram

[IoT-based ZeroWait Smart Checkout Diagram showing Sensing Layer with Weight Sensor Unit and Pi Camera, Processing and Data Gateway Layer with ESP32 and Raspberry Pi 5, and Application and Communication Layer with Mosquitto Broker, zerowait.store Dashboard, and Cloud Data Lake]![IoT Diagram](docs/images/img5.jpeg)
### MQTT Data Flow

```
ESP32 (Publisher)
    │
    │  Topic: zerowait/shop/<id>/weight
    │  Payload: { "grams": 205.3, "stable": true, "ts": 1720000000 }
    │  QOS: 1  |  Keep-alive: 30s
    ▼
Mosquitto Broker v2.0
    │  Authentication/ACL  |  Persistent Sessions
    │  Secure WebSockets (WSS)
    ▼
Raspberry Pi 5 (Subscriber)
    │  Triggers Roboflow inference on weight-change event
    │  Correlates vision label ↔ weight delta
    ▼
zerowait.store Hub  →  Kiosk UI  →  Cart Update
```

### Mass-Visual Correlation (Anti-Theft Engine)

The system implements a dual-verification strategy to prevent **spoofing attacks** (e.g., placing a heavy item that visually resembles a light item):

```python
# Pseudocode — correlation logic
def verify_item(vision_label: str, weight_delta_g: float) -> VerificationResult:
    expected_weight = inventory.get_weight_range(vision_label)
    tolerance = 0.15  # 15% tolerance band

    if abs(weight_delta_g - expected_weight.nominal) / expected_weight.nominal <= tolerance:
        return VerificationResult.APPROVED
    else:
        return VerificationResult.ALERT  # triggers anti-theft flag
```

### Video Relay Pipeline & Latency Optimization

| Stage | Resolution | Technique | Impact |
|---|---|---|---|
| Capture | 2304 × 1296 | Pi Camera 3 RAW | Max FOV |
| Encode | — | V4L2 Hardware (VideoCore VII) | CPU-offloaded |
| Downscale | 800 × 450 | `cv2.resize()` pre-relay | −72% bandwidth |
| Transport | — | HTTP Keep-Alive (persistent TCP) | −800ms → −90ms latency |
| Display | — | `<canvas>` WebSocket stream | Browser-native, 24 FPS |

```python
# HTTP persistent session — eliminates per-frame TCP handshake
session = requests.Session()
session.headers.update({'Connection': 'keep-alive'})

def relay_frame(frame):
    small = cv2.resize(frame, (800, 450))
    _, buf = cv2.imencode('.jpg', small, [cv2.IMWRITE_JPEG_QUALITY, 80])
    session.post(f"{HUB_URL}/frame/{FRAME_ID}", data=buf.tobytes())
```

---

## Computer Vision Pipeline

### Roboflow Model — Grocery Detection

| Metric | Value |
|---|---|
| **mAP@50** | **99.1%** |
| **Precision** | **99.4%** |
| **Recall** | **97.3%** |
| **F1 Score** | **98.3%** |
| Backbone | YOLOv8 |
| Quantisation | INT8 via ONNX Runtime |
| Acceleration | CUDA / Vulkan (Pi 5 VideoCore VII) |
| Dataset | 100+ SKU categories, real-world grocery shelves |
| Training Platform | Roboflow (annotation + augmentation + training) |

### Inference Flow

```
Pi Camera Frame (2304×1296)
        │
        ▼  cv2.resize → (800×450)
Roboflow Cloud API
        │  model_id: 'grocery-detection/5'
        ▼
[{ label: "Apple", confidence: 0.97, bbox: [...] },
 { label: "Milk",  confidence: 0.99, bbox: [...] }]
        │
        ▼  Mass-Visual Correlation Check
Cart State Manager → Kiosk UI Update
```

---

## Payment Integration

### Razorpay Flow

```
1. Customer taps "Checkout & Pay"
        │
        ▼
2. Razorpay Standard Checkout overlay (UPI / Cards / NetBanking / Wallets)
        │  256-bit TLS | PCI-DSS compliant
        ▼
3. Payment captured → Razorpay fires webhook: POST /razorpay/webhook
        │  HMAC-SHA256 signature verification
        ▼
4. Backend: pi5_receipt_commit()
        ├── Cart cleared
        ├── Order written to PostgreSQL
        ├── PDF receipt uploaded to Amazon S3
        └── HTML email dispatched via SMTP (smartcheckoutbilling@gmail.com)
```

### Webhook Handler (Simplified)

```python
@app.route('/razorpay/webhook', methods=['POST'])
def razorpay_webhook():
    payload = request.get_data(as_text=True)
    received_sig = request.headers.get('X-Razorpay-Signature')

    expected_sig = hmac.new(
        RAZORPAY_WEBHOOK_SECRET.encode(),
        payload.encode(), hashlib.sha256
    ).hexdigest()

    if not hmac.compare_digest(received_sig, expected_sig):
        abort(400)

    event = request.json
    if event['event'] == 'payment.captured':
        pi5_receipt_commit(event['payload']['payment']['entity'])

    return jsonify(status='ok'), 200
```

---

## Performance Metrics

| Metric | Measured Value | Notes |
|---|---|---|
| Model mAP@50 | 99.1% | Roboflow Cloud API |
| Precision | 99.4% | True Positive Rate |
| Recall | 97.3% | Detection Coverage |
| F1 Score | 98.3% | Harmonic Mean |
| Stream Latency (before) | ~800 ms | Without optimisations |
| Stream Latency (after) | ~90 ms | HTTP Keep-Alive + downscale |
| Inference Time | ~150 ms | Cloud API round-trip |
| Pi 5 CPU Usage | ~28% | Down from 95% pre-optimisation |
| Bandwidth (before) | ~18 MB/s | 2304×1296 relay |
| Bandwidth (after) | ~5 MB/s | 800×450 relay (−72%) |
| Frame Rate | 24 FPS | Stable, hardware-encoded |
| Weight Resolution | 24-bit (±0.01g) | HX711 @ 128× gain |
| Payment Checkout | < 2 seconds | Tap to Razorpay confirmation |

---

## Repository Structure

```
zerowait/
│
├── docs/
│   └── images/
│       ├── img1.jpeg           # Hardware specs (Pi 5, ESP32, Load Cell, Camera)
│       ├── img2.jpeg           # Full-stack software & cloud ecosystem diagram
│       ├── img3.jpeg           # Live kiosk UI demo on iPad
│       ├── img4.jpeg          # Prototype in action — item placement demo
│       └── img5.jpeg          # IoT 3-layer architecture diagram
│
├── hub/                        # Central Flask hub server (zerowait.store)
│   ├── app.py
│   ├── models.py
│   ├── routes/
│   │   ├── admin.py
│   │   ├── kiosk.py
│   │   └── payment.py
│   ├── mqtt_handler.py
│   ├── vision.py
│   ├── billing.py
│   └── templates/
│       ├── kiosk.html
│       └── admin/
│
├── frame/                      # Raspberry Pi 5 edge node
│   ├── camera_relay.py
│   ├── mqtt_subscriber.py
│   ├── inference_engine.py
│   └── config.py
│
├── esp32/                      # ESP32 firmware
│   ├── zerowait_esp32.ino
│   └── boot.py
│
├── scripts/
│   ├── setup_hub.sh
│   ├── setup_frame.sh
│   └── mosquitto.conf
│
├── requirements.txt
├── .env.example
└── README.md
```

---

## Getting Started

### Prerequisites

| Component | Requirement |
|---|---|
| Hub Server | Ubuntu 22.04+ / Any Linux (Python 3.11+) |
| Raspberry Pi 5 | Raspberry Pi OS Bookworm (64-bit) |
| ESP32 | Arduino IDE 2.x or PlatformIO |
| Python | 3.11+ |
| Node.js | 18+ (for React frontend build) |
| Accounts | Roboflow (API key), Razorpay (key + secret), Gmail SMTP |

### Hub Server Setup

```bash
# 1. Clone the repository
git clone https://github.com/<your-org>/zerowait.git
cd zerowait/hub

# 2. Create virtual environment
python3.11 -m venv venv
source venv/bin/activate

# 3. Install dependencies
pip install -r requirements.txt

# 4. Configure environment
cp ../.env.example .env
nano .env

# 5. Initialise database
flask db upgrade

# 6. Start the hub
flask run --host=0.0.0.0 --port=5000
```

### Raspberry Pi 5 (Frame) Setup

```bash
# 1. Enable camera
sudo raspi-config  # Interface Options → Camera → Enable

# 2. Install dependencies
cd zerowait/frame
pip install -r requirements.txt
sudo apt install -y mosquitto-clients python3-picamera2

# 3. Configure frame
cp config.example.py config.py
nano config.py  # Set FRAME_ID, HUB_URL, API_KEY

# 4. Start the frame agent
python camera_relay.py &
python mqtt_subscriber.py &
```

### ESP32 Firmware Flash

```cpp
// Key parameters to set before flashing:
const char* ssid         = "YOUR_HOTSPOT_SSID";  // 2.4 GHz band only
const char* mqtt_server  = "YOUR_PI5_IP";
const int   SAMPLE_RATE  = 10;                    // Hz
const float CALIB_FACTOR = 420.0983;              // Calibrate per load cell
```

```bash
# Arduino IDE:
# 1. Open esp32/zerowait_esp32.ino
# 2. Board Manager → install Espressif ESP32
# 3. Library Manager → install PubSubClient, HX711_ADC
# 4. Set parameters above → Flash
```

---

## Configuration

```env
# ── Hub Server ──────────────────────────────────────
FLASK_SECRET_KEY=your_flask_secret_key_here
DATABASE_URL=postgresql://user:password@localhost/zerowait

# ── Roboflow ────────────────────────────────────────
ROBOFLOW_API_KEY=your_roboflow_api_key
ROBOFLOW_MODEL_ID=grocery-detection/5

# ── Razorpay ────────────────────────────────────────
RAZORPAY_KEY_ID=rzp_live_xxxxxxxxxxxx
RAZORPAY_KEY_SECRET=your_razorpay_secret
RAZORPAY_WEBHOOK_SECRET=your_webhook_secret

# ── SMTP / Email ────────────────────────────────────
SMTP_EMAIL=smartcheckoutbilling@gmail.com
SMTP_APP_PASSWORD=your_gmail_app_password

# ── AWS S3 ──────────────────────────────────────────
AWS_ACCESS_KEY=your_aws_access_key
AWS_SECRET_KEY=your_aws_secret_key
AWS_S3_BUCKET=zerowait-receipts

# ── MQTT Broker ─────────────────────────────────────
MQTT_BROKER_HOST=localhost
MQTT_BROKER_PORT=8883
MQTT_USERNAME=zerowait
MQTT_PASSWORD=your_mqtt_password
```

---

## API Reference

### Frame Registration
```http
POST /api/v1/frames/register
Authorization: Bearer <admin_token>

{ "shop_id": "shop_abc123", "frame_name": "Checkout Lane 1" }
```

### Cart Update (Frame → Hub)
```http
POST /api/v1/frames/<frame_id>/cart/add
X-API-Key: <frame_api_key>

{ "label": "Apple", "confidence": 0.97, "weight_g": 205.3, "price": 20.00 }
```

### Initiate Checkout
```http
POST /api/v1/frames/<frame_id>/checkout/initiate
X-API-Key: <frame_api_key>

→ { "razorpay_order_id": "order_Nxxxxx", "amount": 24700, "currency": "INR" }
```

---

## Future Scope

- **Multi-Item Detection** — Detect multiple overlapping items simultaneously on a single tray
- **On-Device Inference** — INT8 ONNX model on Pi 5 VideoCore VII (target: 50ms, zero cloud dependency)
- **Offline Mode** — Local SQLite fallback for mission-critical retail environments
- **UPS Integration** — Battery autonomy for power interruptions
- **Open Frame SDK** — Third-party detection models and plugin marketplace
- **Scale to 1000+ Stores** — Centralized multi-store inventory and analytics

---

## Team

| Name | PRN | Role |
|---|---|---|
| **Saksham** | `1032232749` | Hardware & IoT Lead — ESP32, HX711, MQTT Layer, Load Cell Calibration |
| **Soham** | `1032232422` | Backend & Cloud Lead — Flask Hub, Razorpay Webhooks, PostgreSQL, S3 |
| **Sarthak** | `1032232607` | Vision & Frontend Lead — Roboflow Training, Pi Camera Pipeline, React.js Kiosk UI |

> MIT World Peace University · SY B.Tech · Autonomous Systems Project · 2024–25

---

## License

```
MIT License — Copyright (c) 2025 ZeroWait (Saksham, Soham, Sarthak)
MIT World Peace University

Permission is hereby granted, free of charge, to any person obtaining a copy
of this software and associated documentation files (the "Software"), to deal
in the Software without restriction, including without limitation the rights
to use, copy, modify, merge, publish, distribute, sublicense, and/or sell
copies of the Software.
```

---

<div align="center">

**Built with ❤️ at MIT World Peace University**

`zerowait.store` · *The Future is Frictionless*

</div>
