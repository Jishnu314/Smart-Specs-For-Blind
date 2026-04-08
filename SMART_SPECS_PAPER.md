# Smart Specs for the Blind
## A Multimodal Wearable Assistive Device for the Visually Impaired

**Author:** Jishnu K.  
**Department:** Department of Applied Electronics and Instrumentaion Engineering  
**Institution:** GECK 


## Abstract

Visual impairment significantly affects independent mobility, social interaction, and environmental awareness. Conventional assistive tools such as white canes and mobile applications address only isolated problems and often fail to provide a unified, hands-free experience. This paper presents **Smart Specs for the Blind**, a wearable assistive system built around a Raspberry Pi 4B and designed to support visually impaired users through real-time obstacle navigation, face recognition, object identification, barcode scanning, scene description, fall detection, and caregiver video calling. The system integrates a VL53L5CX depth sensor, MPU-6500 inertial measurement unit, Pi Camera, haptic motors, Firebase cloud services, and a Bluetooth Low Energy remote. A concurrent software architecture enables low-latency mode switching by preloading models at boot and maintaining always-on sensing threads in the background. The prototype demonstrates how edge AI, cloud services, and wearable sensing can be combined into a practical assistive platform that improves safety and user independence.

**Keywords:** assistive technology, wearable computing, visually impaired, obstacle detection, fall detection, face recognition, edge AI, Raspberry Pi

## 1. Introduction

Millions of people with visual impairment face difficulties in navigation, recognizing familiar people, reading product information, and calling for help during emergencies. Traditional tools such as white canes remain useful for close-range navigation but provide no semantic understanding of surroundings and no connectivity to caregivers. Smartphone-based tools add useful features such as OCR and scene interpretation, but they require a handheld interaction model that is inconvenient during movement.

Smart Specs for the Blind was developed to address this gap through a single wearable platform. The project combines sensing, computer vision, haptics, voice output, and caregiver connectivity into one coordinated device. The design emphasizes three goals:

- hands-free interaction
- real-time responsiveness
- integrated multimodal assistance

## 2. Problem Statement

Existing assistive devices typically solve only one part of the problem. A cane helps with near-ground obstacle detection, while a smartphone app may help with reading text or describing scenes. However, users still need separate solutions for face recognition, remote caregiver communication, and emergency fall response. This fragmentation increases cognitive load and reduces usability in real-world conditions.

The problem addressed in this work is the lack of an affordable, integrated, wearable assistive system that can support mobility, recognition, safety, and communication in a unified way.

## 3. Objectives

The main objectives of Smart Specs are:

- to provide obstacle awareness using depth sensing and haptic feedback
- to recognize known people and allow easy face enrollment
- to identify nearby objects and describe full scenes using AI
- to detect and decode barcodes for product assistance
- to monitor falls and notify caregivers automatically
- to enable remote support through video calling
- to allow simple hands-free control using a BLE remote

## 4. Literature Review

Research in assistive technology has produced several navigation and recognition aids for visually impaired users. Electronic travel aids based on sonar and ultrasonic sensing improved obstacle awareness, but they lacked semantic interpretation of the environment. GPS-based solutions improved route guidance but remain unreliable indoors and do not offer fine-grained local obstacle avoidance. Camera-based systems and commercial smart glasses introduced text reading and recognition features, but many remain expensive, fragmented, or dependent on a phone for operation.

Recent advances in lightweight computer vision models and cloud-based multimodal AI make it possible to create more capable assistive devices on embedded hardware. Even so, combining navigation, recognition, fall detection, and caregiver communication into a single wearable system remains an engineering challenge. Smart Specs contributes to this area by integrating these capabilities into one Raspberry Pi-based prototype.

## 5. Proposed System

Smart Specs is a glasses-mounted wearable assistive device controlled through a four-button BLE remote. The device continuously runs navigation and safety monitoring in the background while allowing the user to switch between multiple functional modes.

### Bluetooth Remote (ESP32)

| Button | Single Press | Double-tap |
|--------|-------------|------------|
| **1** | Toggle Face Recog ↔ Face Add | — |
| **2** | Toggle Object ID ↔ Barcode | — |
| **3** | Toggle Scene ↔ Video Call | — |
| **4 / H** | Trigger action in current mode | **Hard reset → Navigation** |

Pressing button 1/2/3 from any mode jumps directly to that group — no home step needed. Only button 4 can reset to Navigation, preventing accidental resets.
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



### 5.1 Operational Modes

**Navigation Mode:**  
The VL53L5CX depth sensor measures obstacle proximity across an 8x8 grid. Directional haptic feedback helps the user understand where obstacles are located.

```
Zone thresholds (configurable in navigation.py):
  DANGER  < 400 mm  → Haptic burst (all 3 motors) + urgent voice alert
  WARNING < 800 mm  → Directional haptic pulse + cue ("obstacle on left")
  CLEAR   ≥ 800 mm  → Silent
```
Haptic motors fire with PWM duty cycles proportional to proximity — BCM 17 = left, 27 = centre, 22 = right — giving directional tactile feedback independent of audio.
---
**Face Recognition Mode:**  
The camera detects and recognizes known faces using MediaPipe and MobileFaceNet-based embeddings.

## 👤 Face Recognition Pipeline

1. **MediaPipe** (`model_selection=0`, short-range < 2 m) detects and crops the face bounding box.
2. The crop is resized to **112×112** and passed through **MobileFaceNet** (ncnn) to produce a **128-dimensional L2-normalised embedding**.
3. The embedding is compared against all entries in `face_db.pkl` using **cosine similarity**.
4. If the best match exceeds the confidence threshold (default `0.6`), the person's name is spoken aloud.

**Face Enrollment Mode:**  
New people can be added through a guided multi-step face capture process.


### Enrolling a new face

1. Switch to **Face Add** mode (press Button 1 once or twice depending on current mode).
2. Press **H** to begin enrollment.
3. The system captures multiple frames, averages the embeddings, and saves to `face_db.pkl`.
4. Switching away from Face Add mode cancels an in-progress enrollment and announces "Enrollment cancelled."

---


**Object Identification Mode:**  
The user points the camera toward an object and receives an AI-generated spoken description.

**Barcode Scanner Mode:**  
The system detects barcodes and decodes them using image preprocessing and barcode localization support.

`bar_float16.tflite` (YOLOv8-nano, float16) detects the barcode bounding box in the frame. The decoded barcode value is looked up in `scanned_products.csv` (two-column CSV: `barcode,product_name`) and the product name is read aloud. If the value is not found locally, the raw barcode is announced.

```csv
8901234567890,Colgate Total Toothpaste 150g
4005808224708,Nivea Moisturising Cream 250ml
0012000161155,Pepsi 330ml Can
```

**Scene Description Mode:**  
A single command triggers a short spoken description of the surrounding environment.

**Video Call Mode:**  
The user can contact a caregiver through a LiveKit-based communication channel.

## 6. Hardware Components

The prototype combines the following hardware:

- Raspberry Pi 4B as the main processor
- VL53L5CX time-of-flight depth sensor for obstacle detection
- MPU-6500 IMU for motion and fall analysis
- Pi Camera for vision-based tasks
- three haptic motors for directional feedback
- microphone and speaker for audio interaction
- ESP32 BLE remote for mode switching and triggers

## 7. Software Architecture

The system is implemented in Python using a modular architecture.

- `core.py` initializes shared hardware resources such as camera, audio, haptics, I2C devices, and sensor drivers.
- `main.py` coordinates mode transitions and the primary runtime loop.
- `state.py` manages the state machine and button event handling.
- `tasks/navigation.py` handles obstacle sensing, haptic output, and fall detection.
- `tasks/vision_tasks.py` manages face recognition, enrollment, scene description, object identification, and barcode workflows.
- `tasks/comms.py` supports caregiver calling.
- `firebase_helper.py` manages Firestore synchronization and alert writes.
- `bluetooth_handler.py` manages communication with the BLE remote.

The architecture relies on concurrent background threads so that sensing and system services remain active even while the user is in a different mode. This design reduces delay during mode changes and improves responsiveness.

## 8. Working Principle

At startup, the system initializes the camera, sensors, haptics, audio stack, and AI resources. Models are loaded during boot rather than at mode-switch time. After initialization, the device announces readiness and begins in navigation mode.

The user interacts through the BLE remote. Buttons switch between modes, while the action button triggers the main task in the selected mode. Navigation and fall detection continue in the background regardless of the visible mode. If a fall is detected and the user does not respond, the system sends an alert through Firebase so that a caregiver can take action.


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
│  │8×8 ToF   │          │         │  (Mode FSM + Button Logic) │  │
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
## 9. Key Technical Contributions

The project demonstrates several practical engineering contributions:

- a zero-latency mode switching approach through boot-time model loading
- concurrent thread-based execution for sensing, communication, and AI tasks
- directional haptic navigation for intuitive obstacle awareness
- multimodal integration of local inference and cloud AI services
- caregiver support through alerting and live communication

## 10. Advantages

Smart Specs offers several benefits:

- integrated support for navigation, recognition, safety, and communication
- wearable and hands-free interaction model
- improved user independence
- emergency response support for caregivers
- modular software structure that can be extended in future versions

## 11. Limitations

The current prototype also has limitations:

- limited battery life due to Raspberry Pi power usage
- dependence on internet connectivity for some AI and communication features
- larger form factor compared with commercial eyewear
- recognition accuracy may vary with lighting and occlusion
- performance depends on sensor reliability and calibration

## 12. Applications

This system can be useful in:

- daily independent mobility
- social interaction and face recognition
- retail and supermarket assistance
- home and institutional care settings
- emergency support for elderly visually impaired users

## 13. Future Scope

Several improvements can strengthen the next version of the system:

- GPS-based outdoor navigation
- lighter hardware and better battery optimization
- fully offline scene understanding models
- voice or wake-word interaction
- improved companion app integration
- richer environment understanding through mapping and localization

## 14. Conclusion

Smart Specs for the Blind shows that a low-cost embedded platform can support a rich set of assistive features when hardware sensing, computer vision, haptics, cloud services, and communication tools are carefully integrated. The prototype moves beyond single-purpose assistive devices by offering an all-in-one wearable system that supports navigation, recognition, safety, and caregiver interaction. With further miniaturization and optimization, Smart Specs has the potential to become a practical and impactful assistive technology solution for visually impaired users.

## References

1. L. Kay, "A sonar aid to enhance spatial perception of the blind: Engineering design and evaluation," *Radio and Electronic Engineer*, 1974.
2. Microsoft, "Seeing AI," [https://www.microsoft.com/en-us/ai/seeing-ai](https://www.microsoft.com/en-us/ai/seeing-ai)
3. A. Buchs et al., "Increasing independent mobility of blind individuals using the EyeCane," *Experimental Brain Research*, 2015.
4. OrCam, "OrCam MyEye," [https://www.orcam.com](https://www.orcam.com)
5. X. Yang et al., "Real-time obstacle detection for visually impaired persons using deep learning and RGB-D cameras," *IEEE Sensors Journal*, 2022.
6. N. Noury et al., "Fall detection principles and methods," *IEEE EMBS Conference*, 2007.
