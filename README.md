<div align="left">



**ProVision : AI-Powered Clinical Patient Monitoring System**

`v2.0` · Python 3.10+ · PyQt6 · YOLOv8 · MediaPipe · Flask

[![Python](https://img.shields.io/badge/Python-3.10%2B-blue?style=flat-square)](https://python.org)
[![PyQt6](https://img.shields.io/badge/UI-PyQt6-41cd52?style=flat-square)](https://pypi.org/project/PyQt6/)
[![Flask](https://img.shields.io/badge/Web-Flask-000?style=flat-square&logo=flask)](https://flask.palletsprojects.com/)
[![YOLOv8](https://img.shields.io/badge/AI-YOLOv8-00c853?style=flat-square)](https://ultralytics.com/)
[![MediaPipe](https://img.shields.io/badge/AI-MediaPipe-FF6F00?style=flat-square)](https://mediapipe.dev/)
[![License](https://img.shields.io/badge/License-Proprietary-red?style=flat-square)]()

</div>

---

Authors:

**Lead/Software Developer :** Edgar Joerenz B. Pondales - Robotics Engineering

**Co/Hardware Developer :** Ali R. Goyena - Computer Science

## Overview

**ProVision OS** is a production-grade, AI-powered patient monitoring desktop application built for clinical environments. It continuously streams live video from ESP32-CAM nodes over a hospital wireless network, runs real-time computer vision inference for fall detection and seizure monitoring, and surfaces actionable telemetry to clinical staff through a premium dark-mode PyQt6 interface.

The system is dual-target: it runs natively on **Linux Mint x86_64** (primary deployment target) and is structurally compatible for future deployment on **Raspberry Pi 5 ARM64** nodes.

---

## Key Features

| Feature | Description |
|---|---|
| 🎥 **Live Multi-Camera Monitoring** | Hardware-accelerated MJPEG stream ingestion via decoupled `QThread` workers. Bounded frame buffer (2 frames) prevents memory runaway. Streams at ~30 FPS. |
| 🧠 **YOLOv8 Person Detection** | Real-time multi-person bounding box projection using `yolov8n.pt`. Click-to-lock patient selection with tracking-loss telemetry bubbling. |
| 🦴 **MediaPipe Pose & Hand Estimation** | Full-body skeletal overlays, fall angle computation, hand landmark extraction, and gesture chord recognition per locked patient. |
| 🚨 **Fall & Seizure Detection** | Oscillation frequency analysis and angular velocity thresholds trigger atomic alerts. Configurable via validated settings sliders. |
| 🤙 **15-Chord Hand Gesture System** | Maps thumb-to-finger contact combinations (bitmask 1–15) to custom patient intent messages. Persistently stored in `gesture_responses.json`. |
| 🎬 **4-Second Alert Video Clips** | Automatically slices ±2 seconds of raw frames around each detection event into an MP4 file without dropping the live stream. |
| 📊 **Metrics & Analytics Dashboard** | 24-hour bar charts, hourly pie breakdowns, and 1-month alert trend line graphs powered by Matplotlib embedded in Qt. |
| 🌐 **Flask Web Companion** | Cookie-authenticated web dashboard at `http://localhost:5000` with live camera status, alert logs, and the ESP32 hardware handshake endpoint. |
| 📁 **Clinical Reports (DOCX)** | One persistent DOCX report per patient/camera — overwritten on update, not versioned — including demographics, caregiver assignments, and alert history. |
| 🔒 **Secure Clinician Authentication** | Username + password login with bcrypt-hashed credentials stored in `webapp_users.json`. Shared store between desktop and web companion. |

---

## Architecture

```
ProVisionOS/
├── main.py                      # Entry point — boots managers, web server, login, dashboard
├── preboot.py                   # Graphical preboot diagnostics (storage, models, network, cameras)
├── run.sh                       # Self-contained launcher — venv, deps, Qt platform, launch
├── requirements.txt
│
├── app/
│   ├── config/
│   │   ├── settings_manager.py  # Atomic settings persistence + gesture map storage
│   │   └── state_manager.py     # Thread-safe state: patients, cameras, alerts, discovery queue
│   │
│   ├── services/
│   │   └── camera_worker.py     # QThread — stream decode → YOLO → MediaPipe → signals
│   │
│   ├── tracking/
│   │   ├── yolo_detector.py     # YOLOv8n multi-person bounding boxes
│   │   ├── pose_estimator.py    # MediaPipe Pose + Hands landmark extraction
│   │   ├── seizure_detector.py  # Oscillation frequency / shake amplitude analysis
│   │   └── gesture_manager.py  # 15-chord thumb-touch bitmask gesture detector
│   │
│   ├── alerts/
│   │   ├── alert_manager.py     # Cooldown-gated alert registration and logging
│   │   └── video_slicer.py      # 4-second MP4 clip compiler (daemon thread)
│   │
│   ├── cameras/
│   │   └── handshake_server.py  # ESP32-CAM HTTP discovery handshake handler
│   │
│   ├── reports/
│   │   ├── docx_builder.py      # Patient DOCX report generator
│   │   └── print_handler.py     # OS-level native print dialog wrapper
│   │
│   ├── analytics/               # Aggregation helpers for charts
│   │
│   ├── ui/
│   │   ├── bootscreen.py        # Split-panel clinician login / register window
│   │   ├── main_window.py       # QMainWindow shell — sidebar + QStackedWidget
│   │   ├── live_monitoring.py   # Live camera grid + executive KPI dashboard
│   │   ├── clinical_directory.py# Patient profiles, caregiver directory, user settings
│   │   ├── incident_center.py   # Alert log table + 4-second MP4 playback viewer
│   │   ├── metrics_analytics.py # Matplotlib embedded chart panels
│   │   ├── weekly_reports.py    # Patient history timeline, DOCX export, purge
│   │   └── settings_view.py     # Sliders, discovery queue accept/reject, gesture map
│   │
│   ├── web/
│   │   ├── app.py               # Flask WebServer daemon thread
│   │   └── auth.py              # Login/register/validate against webapp_users.json
│   │
│   └── utils/
│       └── helpers.py           # Log writer, path resolvers, filename sanitizer
│
├── data/                        # Runtime-generated data (gitignored)
│   ├── models/yolov8n.pt
│   ├── settings.json
│   ├── gesture_responses.json
│   ├── cameras.json
│   ├── profiles.json
│   ├── caregivers.json
│   ├── alerts_log.json
│   ├── reports/                 # Per-patient DOCX files
│   └── alerts/videos/           # Alert MP4 clips
│
├── firmware/
│   └── ESPPROVISION.ino         # ESP32-CAM Arduino firmware source
│
├── assets/                      # UI images and boot screen assets
├── deployment/                  # Platform-specific deployment configs
├── scripts/                     # Utility and migration scripts
└── tests/
    └── test_pipeline.py
```

---

## Navigation — 6 Macro Sections

The sidebar drives a `QStackedWidget` with six distinct clinical workspaces:

| # | Section | Purpose |
|---|---|---|
| 1 | **Live Monitoring Workspace** | Camera grid, patient lock indicators, CV overlays, KPI tiles, 1-month trend graph |
| 2 | **Clinical Directory** | Patient demographics, caregiver directory (name + phone), clinician user profile |
| 3 | **Incident Center & Playback** | Chronological alert log, filter controls, MP4 frame-by-frame clip playback |
| 4 | **Metrics & Live Analytics** | 24h bar charts, hourly pie charts, monthly alert frequency trends |
| 5 | **Weekly & Daily Reports** | Patient history timeline, DOCX generation, OS print dialog, record purge |
| 6 | **Infrastructure & Settings** | ESP32 discovery queue (accept ✓ / reject ✗), threshold sliders, gesture map editor |

---

## Getting Started

### Prerequisites

- Python 3.10+
- `libxcb-cursor0` (required by PyQt6 on Linux)
- An active display server (X11 / Wayland)

```bash
sudo apt-get install libxcb-cursor0
```

### 1. Launch (recommended)

The launcher script handles everything automatically:

```bash
chmod +x run.sh
./run.sh
```

This will:
1. Create the Python virtualenv at `.venv/` if not present
2. Install all pip dependencies from `requirements.txt`
3. Copy `yolov8n.pt` from the legacy PROVISION_OS path (if present) or let ultralytics auto-download on first use
4. Verify Qt display initialization (`xcb` with fallback to `offscreen`)
5. Launch `main.py`

### 2. Web Companion

Once running, the web dashboard is available at:

```
http://localhost:5000
```

Default login credentials (first run):
```
Username: admin
Password: admin123
```

### 3. Connecting ESP32-CAM Nodes

Flash the firmware from `firmware/ESPPROVISION.ino` onto each ESP32-CAM. The device will automatically call the handshake endpoint:

```
GET http://<host-ip>:5000/report?ip=<device-ip>&type=cam
```

The node then appears in the **Infrastructure & Settings** discovery queue, where you accept (✓) or reject (✗) it. Accepting immediately prompts a rename dialog before registering the camera.

---

## Configuration & Settings

All settings are persisted atomically to `data/settings.json` using a `.tmp`-swap write pattern to prevent corruption on unexpected shutdowns.

### UI-Configurable Sliders

| Setting | Key | Default |
|---|---|---|
| AI Body Scan Smoothing | `WINDOW_SECONDS` | `4.0` |
| Camera Mounting Angle | `FALL_ANGLE_THRESHOLD_DEG` | `55.0°` |
| Fall Detection Timeout | `CONFIRM_WINDOW` | `1.0 s` |
| Seizure Intensity Threshold | `PEAK_TO_PEAK_MIN` | `0.05` |

Changes are **staged** in UI and only committed on clicking **Apply Settings** (transactional commit pattern).

### Gesture Map

Stored in `data/gesture_responses.json`. Maps bitmask IDs `1–15` (thumb-to-finger chord combinations) to clinical response strings:

```json
{
  "1":  "Patient wants to Urinate",
  "2":  "Patient needs Water",
  "3":  "Patient requests Assistance",
  "4":  "Patient is in Pain",
  "8":  "Emergency Alert",
  "15": "All fingers touched"
}
```

---

## Platform Compatibility

| Target | Status | Notes |
|---|---|---|
| Linux Mint 21+ (x86_64) | ✅ Primary | Full feature support. Qt xcb display. |
| Ubuntu 22.04+ (x86_64) | ✅ Supported | Identical dependency chain. |
| Raspberry Pi 5 (ARM64) | 🟡 Planned | Requires `mediapipe==0.10.14` ARM wheel. High-memory model compilation on first run. |
| macOS | ⚠️ Partial | PyQt6 supported; ESP32 hotspot features untested. |
| Windows | ❌ Not tested | — |

---

## Dependencies

| Package | Version | Purpose |
|---|---|---|
| `PyQt6` | ≥6.4.0 | Desktop GUI framework |
| `flask` | ≥2.0.0 | Web companion dashboard server |
| `ultralytics` | ≥8.0.0 | YOLOv8 person detection |
| `mediapipe` | ≥0.10.0 | Pose estimation + hand landmarks |
| `opencv-python-headless` | ≥4.5.0 | MJPEG frame decode, MP4 writing |
| `numpy` | ≥1.20.0 | Array operations |
| `matplotlib` | ≥3.5.0 | Analytics charts embedded in Qt |
| `python-docx` | ≥0.8.11 | Clinical DOCX report generation |
| `pillow` | ≥9.0.0 | Image conversion utilities |
| `requests` | ≥2.25.0 | HTTP requests to ESP32 nodes |
| `werkzeug` | ≥2.0.0 | Flask WSGI server utilities |

---

## Data & Privacy

- All patient data is stored **locally** in `data/` — nothing leaves the device.
- Credentials are stored as bcrypt hashes in `data/webapp_users.json`.
- Alert video clips are stored in `data/alerts/videos/` as MP4 files.
- DOCX reports are stored in `data/reports/` (one file per patient, overwritten on update).

> **⚠ Warning:** The `data/` directory contains protected health information (PHI). Ensure the host system has appropriate access controls and disk encryption in clinical deployments.

---

## License

Proprietary. All rights reserved. Not licensed for redistribution or commercial use without explicit written permission.

---

<div align="center">
<sub>Built for clinical environments · ProVision OS v2.0</sub>
</div>
