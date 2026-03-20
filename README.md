<div align="center">

<img src="https://img.shields.io/badge/VisionSense-v1.0.0-1D9E75?style=for-the-badge" alt="version">

# 👓 VisionSense

### Assistive Smart Spectacles for the Visually Impaired

*Real-time object detection → spatial audio feedback → independent navigation*

[![Python](https://img.shields.io/badge/Python-3.10+-3572A5?style=flat-square&logo=python&logoColor=white)](https://python.org)
[![YOLOv8](https://img.shields.io/badge/YOLOv8-Ultralytics-FF6F00?style=flat-square)](https://ultralytics.com)
[![Raspberry Pi 4](https://img.shields.io/badge/Raspberry%20Pi-4-C51A4A?style=flat-square&logo=raspberrypi&logoColor=white)](https://raspberrypi.org)
[![License: MIT](https://img.shields.io/badge/License-MIT-green?style=flat-square)](LICENSE)
[![Stars](https://img.shields.io/github/stars/accessibility-ai/VisionSense?style=flat-square&color=yellow)](https://github.com/accessibility-ai/VisionSense/stargazers)

</div>

---

## 🌍 Overview

**VisionSense** is an open-source assistive technology project that transforms a pair of smart spectacles into an intelligent navigation / assistive companion for visually impaired users.

By combining **edge AI object detection**, **depth estimation**, and **spatial audio synthesis**, VisionSense converts the visual world into an intuitive audio landscape — helping users:

- 🚶 **Navigate safely** through indoor and outdoor environments
- 🚧 **Avoid obstacles** in real time with priority hazard alerts
- 🗺️ **Understand their surroundings** via natural language scene descriptions
- 🔑 **Operate independently** without relying on a sighted companion

> *"Person ahead, 2 metres. Door to your left. Stairs down — 3 steps."*

---

## 📐 System Architecture

```
┌─────────────────────────────────────────────────────────┐
│                   VisionSense Pipeline                  │
│                                                         │
│  ┌──────────┐    ┌──────────┐    ┌──────────────────┐  │
│  │  Stereo  │───▶│ YOLOv8n  │───▶│  Scene Composer  │  │
│  │  Camera  │    │ (Edge)   │    │  + Depth Model   │  │
│  └──────────┘    └──────────┘    └────────┬─────────┘  │
│                                           │             │
│                                  ┌────────▼─────────┐  │
│                                  │   Audio Engine   │  │
│                                  │  Spatial TTS +   │  │
│                                  │  Priority Alerts │  │
│                                  └────────┬─────────┘  │
│                                           │             │
│                                  ┌────────▼─────────┐  │
│                                  │  Bone Conduction │  │
│                                  │   Headphones     │  │
│                                  └──────────────────┘  │
└─────────────────────────────────────────────────────────┘
```

| Component | Technology | Details |
|-----------|-----------|---------|
| Compute | Raspberry Pi 5 (8GB) | ARM Cortex-A76 @ 2.4GHz |
| Camera | IMX219 Stereo Module | 30fps, 1080p, 120° FoV |
| Detection | YOLOv8n (ONNX) | 80 COCO classes, ~45ms/frame |
| Depth | MiDaS v2.1 Small | Monocular depth estimation |
| Audio | pyttsx3 + pygame | Spatial stereo panning + TTS |
| Output | Bone conduction speakers | Leaves ears open for ambient sound |
| Battery | 10,000mAh LiPo | ~6 hours continuous use |

---

## ✨ Features

### Core Detection
- **Real-time object recognition** at 30fps with sub-100ms end-to-end latency
- **80+ object classes** — people, vehicles, furniture, animals, everyday items
- **Confidence thresholding** to suppress false positives

### Spatial Audio System
- **Stereo panning** — detected objects pan left/right based on their horizontal position in frame
- **Distance-based volume** — near objects are louder; far objects are softer
- **Priority queue** — hazardous objects (vehicles, stairs, doors) interrupt the audio queue
- **Natural TTS descriptions** — full sentence output: *"Bicycle approaching from the right"*

### Intelligent Scene Understanding
- **Scene summarisation** — periodic full-scene narration every 5 seconds
- **Obstacle alerts** — immediate audio burst for sudden hazards entering the frame
- **Persistent tracking** — objects are tracked across frames to avoid repetitive announcements

### Power & Performance
- **Low-power mode** — adaptive frame skipping based on battery level
- **Thermal throttle guard** — reduces inference rate if CPU temperature exceeds 75°C
- **Offline-first** — all inference runs on-device, no internet required

---

## 🛠 Hardware Requirements

```
Required:
  - Raspberry Pi 5 (4GB or 8GB recommended)
  - IMX219 camera module (or USB stereo camera)
  - Bone conduction headphones (3.5mm or Bluetooth)
  - MicroSD card (32GB+, Class 10)
  - 5V/5A USB-C power supply or LiPo battery HAT

Optional:
  - Waveshare AI HAT+ (NPU acceleration, ~3x faster inference)
  - GPS module (for outdoor waypoint narration)
  - Ultrasonic HC-SR04 (supplemental close-range detection < 30cm)
```

---

## ⚡ Quickstart

### 1. Clone the repository

```bash
git clone https://github.com/accessibility-ai/VisionSense.git
cd VisionSense
```

### 2. Install dependencies

```bash
# System packages
sudo apt update && sudo apt install -y python3-pip libportaudio2 espeak

# Python packages
pip install -r requirements.txt
```

### 3. Download models

```bash
python scripts/download_models.py
# Downloads yolov8n.onnx (~6MB) and midas_v21_small.onnx (~68MB)
```

### 4. Run VisionSense

```bash
# Standard mode
python detect.py --model models/yolov8n.onnx --audio spatial

# With depth estimation
python detect.py --model models/yolov8n.onnx --depth --audio spatial

# Low-power mode (longer battery life)
python detect.py --model models/yolov8n.onnx --low-power --fps 15
```

### 5. (Optional) Run as a systemd service on boot

```bash
sudo cp scripts/visionsense.service /etc/systemd/system/
sudo systemctl enable visionsense
sudo systemctl start visionsense
```

---

## 📂 Project Structure

```
VisionSense/
│
├── firmware/                   # Raspberry Pi setup & GPIO
│   ├── setup.sh                # One-shot Pi configuration script
│   ├── camera_init.py          # Camera module initialisation
│   └── gpio_buttons.py         # Physical button controls (vol, pause)
│
├── models/                     # AI model weights
│   ├── yolov8n.onnx            # Object detection (download via script)
│   ├── midas_v21_small.onnx    # Depth estimation (download via script)
│   └── labels.txt              # COCO class names
│
├── audio_engine/               # Spatial audio & TTS
│   ├── spatial_audio.py        # Stereo panning & volume logic
│   ├── tts_engine.py           # Text-to-speech wrapper (pyttsx3)
│   ├── priority_queue.py       # Hazard alert priority system
│   └── scene_composer.py       # Natural language scene descriptions
│
├── tests/                      # Test suite
│   ├── test_detection.py       # Unit tests for detect.py
│   ├── test_audio.py           # Audio engine tests
│   └── fixtures/               # Sample frames & expected outputs
│
├── scripts/
│   ├── download_models.py      # Model downloader
│   └── visionsense.service     # systemd unit file
│
├── detect.py                   # Main detection loop
├── audio_feedback.py           # Audio pipeline entry point
├── config.yaml                 # All tunable parameters
├── requirements.txt
├── LICENSE
└── README.md
```

---

## ⚙️ Configuration

All parameters are tunable via `config.yaml`:

```yaml
detection:
  model: models/yolov8n.onnx
  confidence_threshold: 0.45
  iou_threshold: 0.5
  target_fps: 30
  input_size: [640, 480]

audio:
  mode: spatial            # spatial | mono | verbose
  tts_rate: 160            # words per minute
  pan_strength: 0.85       # 0.0 = center, 1.0 = hard pan
  min_announce_gap: 2.5    # seconds between repeat announcements
  priority_classes:        # always interrupt for these
    - person
    - car
    - bicycle
    - stairs
    - door

depth:
  enabled: true
  model: models/midas_v21_small.onnx
  near_threshold: 0.8      # objects closer than this trigger alerts

power:
  low_power_mode: false
  low_power_fps: 12
  thermal_limit_celsius: 75
```

---

## 🧪 Running Tests

```bash
# Run all tests
pytest tests/ -v

# Run with coverage report
pytest tests/ --cov=. --cov-report=term-missing

# Test audio engine only
pytest tests/test_audio.py -v
```

---

## 🗺 Roadmap

- [x] YOLOv8 real-time detection pipeline
- [x] Spatial stereo audio engine
- [x] Distance-based volume scaling
- [x] Priority hazard alert system
- [x] Natural TTS scene descriptions
- [ ] GPT-powered richer scene narration
- [ ] GPS integration for outdoor waypoints
- [ ] Companion mobile app (iOS/Android)
- [ ] Multi-language TTS support
- [ ] Custom wake-word activation ("Hey Vision")
- [ ] Sign & text recognition (OCR)
- [ ] Face recognition for familiar people

---

## 🤝 Contributing

Contributions are welcome and appreciated!

1. Fork the repository
2. Create your feature branch: `git checkout -b feat/my-feature`
3. Commit your changes: `git commit -m 'feat: add my feature'`
4. Push to the branch: `git push origin feat/my-feature`
5. Open a Pull Request

Please read [CONTRIBUTING.md](CONTRIBUTING.md) and ensure all tests pass before submitting.

---

## 📄 License

This project is licensed under the **MIT License** — see [LICENSE](LICENSE) for details.

---

## 🙏 Acknowledgements

- [Ultralytics YOLOv8](https://github.com/ultralytics/ultralytics) — object detection backbone
- [Intel MiDaS](https://github.com/isl-org/MiDaS) — monocular depth estimation
- [pyttsx3](https://github.com/nateshmbhat/pyttsx3) — offline text-to-speech
- The visually impaired community — for feedback, testing, and inspiration

---

<div align="center">

Built with ❤️ for accessibility · [Report a Bug](https://github.com/accessibility-ai/VisionSense/issues) · [Request a Feature](https://github.com/accessibility-ai/VisionSense/issues)

</div>
