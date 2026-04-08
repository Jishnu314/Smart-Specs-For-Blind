You're right, my apologies! Here's your full README:

---

<div align="center">

# 👓 Smart Specs

### Open-Source Assistive Spectacles for the Visually Impaired

*Real-time computer vision · Spatial audio · Edge AI · Wearable*

[![Python](https://img.shields.io/badge/Python-3.10+-3572A5?style=flat-square&logo=python&logoColor=white)](https://python.org)
[![Raspberry Pi](https://img.shields.io/badge/Raspberry%20Pi-4-C51A4A?style=flat-square&logo=raspberrypi&logoColor=white)](https://raspberrypi.org)
[![Gemini](https://img.shields.io/badge/Gemini-2.5%20Flash-4285F4?style=flat-square&logo=google&logoColor=white)](https://ai.google.dev)
[![License: MIT](https://img.shields.io/badge/License-MIT-22c55e?style=flat-square)](LICENSE)

</div>

---

## 🌍 Overview

**Smart Specs** is an assistive wearable device that gives visually impaired users an intelligent, real-time understanding of their surroundings. Mounted in a pair of spectacles, it fuses a camera, an 8×8 Time-of-Flight depth sensor, an IMU, and speakers into a single pipeline that turns the visual world into spoken audio.

The user navigates through **7 modes** — each focused on a specific task — switched instantly via a 3-button Bluetooth remote (ESP32). All AI inference runs **fully on-device**; an internet connection is only needed for Gemini-powered scene and object descriptions.

> *"Move right, stop, path clear"*

---

## ✨ Features at a Glance

| Mode | What it does |
|------|-------------|
| 🧭 **Navigation** | Continuous obstacle detection via VL53L5CX 8×8 ToF depth grid + haptic & audio warnings |
| 👤 **Face Recognition** | Identifies enrolled people by name using MobileFaceNet (ncnn) + MediaPipe |
| ➕ **Face Add** | Enroll a new face live — multi-sample capture, averaged embedding, saved to `face_db.pkl` |
| 📦 **Object Identifier** | Press H → Gemini AI describes the object held in front of the camera in plain English |
| 📊 **Barcode Scanner** | TFLite YOLOv8n locates the barcode region; result looked up in `scanned_products.csv` |
| 🖼 **Scene Mode** | Press H → Gemini describes the full scene in one clear spoken sentence |
| 📞 **Video Call** | Streams live camera frames to a remote viewer over a peer connection |

---

## 🏗 System Architecture

```
┌──────────────────────────────────────────────────────────────────┐
│                         Smart Specs Runtime                      │
│                                                                  │
│  ┌──────────┐   ┌───────────┐   ┌──────────────────────────┐   │
│  │  Camera  │──▶│  core.py  │──▶│         main.py          │   │
│  │ 640×480  │   │ (Hardware)│   │  (Conductor / main loop) │   │
│  └──────────┘   └─────┬─────┘   └────────────┬─────────────┘   │
│                        │                      │                  │
│  ┌──────────┐          │         ┌────────────▼──────────────┐  │
│  │VL53L5CX  │──────────┤         │        state.py           │  │
│  │8×8 ToF   │          │         │  (Mode FSM + Button Logic)│  │
│  └──────────┘          │         └────────────┬──────────────┘  │
│                        │                      │                  │
│  ┌──────────┐          │         ┌────────────▼──────────────┐  │
│  │ MPU-6500 │──────────┤         │        models.py          │  │
│  │   IMU    │          │         │  MobileFaceNet · TFLite   │  │
│  └──────────┘          │         │  MediaPipe · Gemini       │  │
│                        │         └────────────┬──────────────┘  │
│  ┌──────────┐          │                      │                  │
│  │  Haptic  │◀─────────┘         ┌────────────▼──────────────┐  │
│  │  Motors  │                    │  tasks/  (per-mode logic)  │  │
│  └──────────┘                    │  navigation · vision_tasks │  │
│                                  │  comms                     │  │
│  ┌──────────┐                    └────────────────────────────┘  │
│  │ ESP32-BT │──▶ bluetooth_handler.py ──▶ state.handle_button()  │
│  │  Remote  │                                                    │
│  └──────────┘                                                    │
└──────────────────────────────────────────────────────────────────┘
```

---

## 🔩 Hardware Requirements

### Required

| Component | Spec | Notes |
|-----------|------|-------|
| Raspberry Pi 4 | 8 GB | Main compute unit |
| Camera Module | IMX219 or USB cam | 640×480, connected on `/dev/video0` |
| VL53L5CX | 8×8 ToF sensor | I2C bus 1 — navigation depth grid |
| MPU-6500 | IMU, I2C addr `0x68` | Step detection & tilt compensation |
| Haptic motors | 3× ERM/LRA motors | BCM GPIO 17 (L), 27 (C), 22 (R) |
| Speakers | 3.5 mm or BT | Leaves ears open for ambient sound |
| ESP32 remote | BLE, advertised as `ESP32-REMOTE` | 3 mode buttons + 1 trigger/home |

### Optional

- **Waveshare AI HAT+** — NPU acceleration, ~3× faster TFLite inference
- **10,000 mAh LiPo + HAT** — ~6 hours continuous runtime
- **GPS module** — future outdoor waypoint narration

---

## ⚡ Quickstart

### 1 · Clone

```bash
git clone https://github.com/your-username/smart-specs.git
cd smart-specs
```

### 2 · Install system dependencies

```bash
sudo apt update && sudo apt install -y \
  python3-pip libportaudio2 espeak \
  libatlas-base-dev libjasper-dev libhdf5-dev
```

### 3 · Install Python packages

```bash
pip install -r requirements.txt
```

### 4 · Set your Gemini API key

Open `models.py` and replace the placeholder:

```python
GENAI_API_KEY = "YOUR_GEMINI_API_KEY_HERE"
```

> Get a free key at [aistudio.google.com](https://aistudio.google.com).
> For production use, load it from an environment variable instead — see [Security Notes](#️-security-notes).

### 5 · Place model files

```
smart-specs/
├── models/
│   ├── mobilefacenet.param     ← MobileFaceNet ncnn param file
│   └── mobilefacenet.bin       ← MobileFaceNet ncnn weights
└── bar_float16.tflite          ← YOLOv8-nano barcode detector (TFLite float16)
```

### 6 · Run

```bash
python main.py
```

### 7 · (Optional) Autostart on boot

```bash
sudo cp scripts/smart-specs.service /etc/systemd/system/
sudo systemctl enable smart-specs
sudo systemctl start smart-specs
```

---

## 📂 Project Structure

```
smart-specs/
│
├── main.py                  # Conductor — main loop, display, mode callbacks
├── state.py                 # Mode finite state machine + button/double-tap logic
├── core.py                  # Hardware init (camera, I2C, ToF, IMU, haptics, audio)
├── models.py                # AI model loader — all models loaded once at boot
├── bluetooth_handler.py     # BLE listener for ESP32 remote (Bleak, auto-reconnect)
│
├── tasks/
│   ├── navigation.py        # Obstacle detection via ToF depth grid + audio/haptic
│   ├── vision_tasks.py      # Face recog, face add, object ID, barcode, scene
│   └── comms.py             # Video call frame streaming
│
├── models/
│   ├── mobilefacenet.param  # Face embedding model (ncnn)
│   └── mobilefacenet.bin
│
├── face_db.pkl              # Enrolled face embeddings (auto-created on first enroll)
├── scanned_products.csv     # barcode → product name lookup table
├── bar_float16.tflite       # YOLOv8-nano barcode region detector
│
├── scripts/
│   └── smart-specs.service  # systemd unit file for autostart
│
├── requirements.txt
├── LICENSE
└── README.md
```

---

## 🎮 Controls

### Bluetooth Remote (ESP32)

| Button | Single Press | Double-tap |
|--------|-------------|------------|
| **1** | Toggle Face Recog ↔ Face Add | — |
| **2** | Toggle Object ID ↔ Barcode | — |
| **3** | Toggle Scene ↔ Video Call | — |
| **4 / H** | Trigger action in current mode | **Hard reset → Navigation** |

Pressing button 1/2/3 from any mode jumps directly to that group — no home step needed. Only button 4 can reset to Navigation, preventing accidental resets.

### Keyboard Fallback (dev / testing)

| Key | Action |
|-----|--------|
| `1` / `2` / `3` | Switch mode group |
| `4` or `H` / `h` | Trigger (double-tap within 0.55 s = hard reset) |
| `ESC` | Quit |

### Per-Mode Trigger (Button 4 — single press)

| Mode | What H does |
|------|------------|
| Navigation | No-op |
| Face Recognition | Announces "Scanning for faces." |
| Face Add | Starts enrollment; if in progress, announces status |
| Object Identifier | Sends cropped ROI to Gemini for identification |
| Barcode | Announces "Point camera at a barcode." |
| Scene | Sends full frame to Gemini for scene description |
| Video Call | Confirms active; auto-exits to Nav if call already dropped |

---

## 🧠 AI Models

| Model | Framework | Task | Runs |
|-------|-----------|------|------|
| MobileFaceNet | ncnn (ARM-optimised) | 128-dim face embedding | On-device |
| MediaPipe Face Detection | MediaPipe | Face bounding box, short-range | On-device |
| YOLOv8n (barcode) | TFLite float16 | Barcode region detection | On-device |
| Gemini 2.5 Flash | Google Cloud API | Object ID + Scene description | Cloud |

All models except Gemini run **entirely on-device** with no network required. Gemini is only called when the user explicitly triggers Object ID or Scene mode.

All models are loaded **once at boot** by `models.load_all()` so mode switching is instantaneous with zero load latency.

---

## 🧭 Navigation Pipeline

The VL53L5CX fires an 8×8 grid of IR pulses at 15 Hz. `navigation.py` partitions the grid into three horizontal zones (left / centre / right) and computes the median distance per zone:

```
Zone thresholds (configurable in navigation.py):
  DANGER  < 400 mm  → Haptic burst (all 3 motors) + urgent voice alert
  WARNING < 800 mm  → Directional haptic pulse + cue ("obstacle on left")
  CLEAR   ≥ 800 mm  → Silent
```

Haptic motors fire with PWM duty cycles proportional to proximity — BCM 17 = left, 27 = centre, 22 = right — giving directional tactile feedback independent of audio.

The navigation thread runs continuously in the background gated by `state.nav_active_event`. When any other mode is active, the thread suspends full obstacle audio but keeps a low-power haptic sentry running.

---

## 👤 Face Recognition Pipeline

1. **MediaPipe** (`model_selection=0`, short-range < 2 m) detects and crops the face bounding box.
2. The crop is resized to **112×112** and passed through **MobileFaceNet** (ncnn) to produce a **128-dimensional L2-normalised embedding**.
3. The embedding is compared against all entries in `face_db.pkl` using **cosine similarity**.
4. If the best match exceeds the confidence threshold (default `0.6`), the person's name is spoken aloud.

### Enrolling a new face

1. Switch to **Face Add** mode (press Button 1 once or twice depending on current mode).
2. Press **H** to begin enrollment.
3. The system captures multiple frames, averages the embeddings, and saves to `face_db.pkl`.
4. Switching away from Face Add mode cancels an in-progress enrollment and announces "Enrollment cancelled."

---

## 📊 Barcode Scanner

`bar_float16.tflite` (YOLOv8-nano, float16) detects the barcode bounding box in the frame. The decoded barcode value is looked up in `scanned_products.csv` (two-column CSV: `barcode,product_name`) and the product name is read aloud. If the value is not found locally, the raw barcode is announced.

To extend the product database, append rows to `scanned_products.csv`:

```csv
8901234567890,Colgate Total Toothpaste 150g
4005808224708,Nivea Moisturising Cream 250ml
0012000161155,Pepsi 330ml Can
```

---

## 🔵 Bluetooth Remote Protocol

The ESP32 remote advertises as **`ESP32-REMOTE`** and sends single-byte BLE notifications on characteristic UUID `12345678-1234-5678-1234-56789abcdef1`.

| Byte | Button |
|------|--------|
| `0x01` | Button 1 — Mode group A |
| `0x02` | Button 2 — Mode group B |
| `0x03` | Button 3 — Mode group C |
| `0x04` | Button 4 — Home / Trigger |

`bluetooth_handler.py` uses **Bleak** with exponential back-off reconnection (starts at 2 s, doubles each failed attempt, capped at 30 s, resets to 2 s after each successful session). All double-tap and mode-reset logic lives entirely in `state.py` — the BT handler only delivers raw press events as fast as possible.

---

## ⚙️ Key Configuration

No config file yet — parameters are constants inside each module:

| Parameter | Module | Default | Description |
|-----------|--------|---------|-------------|
| `GENAI_API_KEY` | `models.py` | `""` | Gemini API key |
| `DB_PATH` | `models.py` | `~/Desktop/SPEC/face_db.pkl` | Face database path |
| `BARCODE_MODEL_PATH` | `models.py` | `bar_float16.tflite` | TFLite barcode model |
| `HAPTIC_PINS` | `core.py` | `[17, 27, 22]` | BCM GPIO for L/C/R haptic |
| `_H_DOUBLE_TAP_WINDOW` | `state.py` | `0.55 s` | Max gap between two H presses |
| `DEVICE_NAME` | `bluetooth_handler.py` | `ESP32-REMOTE` | BLE advertised device name |
| `CHAR_UUID` | `bluetooth_handler.py` | `12345678-...` | BLE notify characteristic UUID |

---

## 📦 requirements.txt

```
opencv-python
numpy
mediapipe
tflite-runtime
ncnn
bleak
smbus2
RPi.GPIO
pyaudio
google-genai
Pillow
```

---

## ⚠️ Security Notes

`del.py` and `models.py` contain hardcoded API keys for development convenience. **Never commit real API keys to a public repository.**

Use environment variables in production:

```bash
export GEMINI_API_KEY="your-key-here"
```

```python
# In models.py
import os
GENAI_API_KEY = os.environ.get("GEMINI_API_KEY", "")
```

---

## 🗺 Roadmap

- [x] 7-mode state machine with BLE remote
- [x] VL53L5CX 8×8 ToF navigation with directional haptic + audio
- [x] MobileFaceNet face recognition (ncnn, fully on-device)
- [x] Live face enrollment with `face_db.pkl` persistence
- [x] YOLOv8n TFLite barcode detection + CSV product lookup
- [x] Gemini-powered object identification
- [x] Gemini-powered scene description
- [x] Video call frame streaming
- [x] Bluetooth remote with double-tap hard reset
- [ ] Centralised `config.yaml` for all tunable parameters
- [ ] OCR for reading signs, labels, and printed text
- [ ] Richer offline TTS voices (Coqui / Piper TTS)
- [ ] GPS waypoint narration for outdoor navigation
- [ ] Companion mobile app (Android / iOS)
- [ ] Multi-language TTS support
- [ ] Wake-word activation ("Hey Specs")
- [ ] Low-power adaptive frame-skip based on battery level

---

## 🤝 Contributing

1. Fork the repository
2. Create a feature branch: `git checkout -b feat/my-feature`
3. Commit with a clear message: `git commit -m 'feat: add IMU-based tilt compensation'`
4. Push and open a Pull Request

Please ensure `python -m pytest tests/` passes before submitting.

---

## 📄 License

MIT License — see [LICENSE](LICENSE) for details.

---

## 🙏 Acknowledgements

- [Tencent ncnn](https://github.com/Tencent/ncnn) — high-performance ARM neural network inference
- [Ultralytics YOLOv8](https://github.com/ultralytics/ultralytics) — barcode region detection backbone
- [MediaPipe](https://mediapipe.dev) — Google's face detection pipeline
- [Bleak](https://github.com/hbldh/bleak) — cross-platform async BLE client for Python
- [Google Gemini](https://ai.google.dev) — scene understanding and object identification
- [STMicroelectronics VL53L5CX](https://www.st.com/en/imaging-and-photonics-solutions/vl53l5cx.html) — multizone ToF ranging sensor

---

<div align="center">

Built with ❤️ for accessibility · [Report a Bug](../../issues) · [Request a Feature](../../issues)

</div>
