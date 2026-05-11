# Sign-Based Command Recognition and Execution for Autonomous Robot Navigation

**Team Members:**
- Ty Carlisle - tycarlis@buffalo.edu

**Course:** IE 482 - Spring 2026, University at Buffalo

---

## Motivation / Overview

This project extends the UB Racer autonomous robot car platform with a real-time sign recognition system. Using a combination of deep learning (YOLOv8) and classical computer vision (OpenCV), the robot camera feed is processed every frame to detect two sign types:

- **STOP sign** → robot halts completely for 3 seconds, then resumes
- **YIELD sign** → robot slows to 75% throttle for 3 seconds, then resumes full speed

After either event, a 5-second cooldown prevents the same sign from triggering again immediately. The system was developed and validated in dev mode using a laptop webcam and printed signs, using the UB Racer client framework for camera streaming, motor control, and browser-based live monitoring.

---

## Demonstration

> **📹 YouTube Demo:** 
> 
> [https://youtu.be/E39QX8WRWbU](https://youtu.be/E39QX8WRWbU)

### STOP Sign Detection
<img width="1341" height="532" alt="image" src="https://github.com/user-attachments/assets/4214e14c-3ec4-4bcb-81f5-d8a470d5d4de" />

### YIELD Sign Detection
<img width="1337" height="541" alt="image" src="https://github.com/user-attachments/assets/1f951287-02c2-4a4a-9323-96ecafe40d53" />

### What the video shows
The demo video shows the full pipeline running in dev mode against a laptop webcam. A printed STOP sign is held up, the robot immediately halts (simulated via `drive(0,0)` commands logged to the browser), and the 3-second countdown banner is visible on the camera feed. A printed YIELD sign is then shown - the robot slows, the cooldown kicks in, and scanning resumes automatically. The live browser UI (client notices, session panel, e-stop) is visible throughout.

---

## Installation Instructions

### Prerequisites
- Python 3.12
- The UB Racer `ub-code` class package (provides `ub_camera` and `ub_utils`)
- Git

### 1 - Clone the repository

```bash
git clone <REPO_URL>
cd spring2026/Projects/ub_racer/client/python
```

### 2 - Install the UB Racer class package

```bash
pip install ub-code
```

> If you already have it installed, make sure it is up to date:
> ```bash
> pip install --upgrade ub-code
> ```

### 3 - Install project dependencies

```bash
pip install -r ../requirements.txt
```

This installs:

| Package | Purpose |
|---|---|
| `fastapi` / `uvicorn` | Client web server (server.py) |
| `python-socketio` | Communication with the host server |
| `websockets` | WebSocket client/server |
| `cryptography` | Auto-generated SSL certificates |
| `aiofiles` / `aiohttp` | Async file and HTTP I/O |
| `ultralytics` | YOLOv8 model for STOP sign detection |

### 4 - Download the YOLO model

The YOLOv8n model (~6 MB) downloads automatically the **first time** `controller.py` runs. You can also pre-download it manually:

```bash
python -c "from ultralytics import YOLO; YOLO('yolov8n.pt')"
```

No GPU required - the model runs on CPU.

---

## How to Run the Code

### Dev mode (no car or host required - webcam only)

Use this to test sign detection with your own camera.

**Terminal 1 - start the web server:**
```bash
cd python
python server.py --dev
```
The server will print a URL like `https://192.168.x.x:8443`. Open it in your browser and accept the self-signed certificate warning.

**Terminal 2 - start the controller:**
```bash
python controller.py --dev
```

**In the browser:**
1. Enter any username and log in
2. In the **Dev Session** card, set **Camera URL or Device** to `0` (built-in webcam) or `1` (external webcam)
3. Click **Start Dev Session**
4. Click **▶ Enable** (E-Stop button) to enable driving
5. Open `https://<YOUR_IP>:8000/stream.mjpg` in a new tab and accept the certificate - this unlocks the camera feed in the main UI

> The camera stream runs on port 8000 with its own self-signed certificate. You must accept it separately before the feed will display.

### Normal mode (host server + physical car)

**Terminal 1:**
```bash
python server.py --host https://HOST_IP:HOST_PORT
```

**Terminal 2:**
```bash
python controller.py
```

Then open the browser URL, log in, and use the **Join Queue** button to request a car.

---

## How the Detection Works

### STOP sign - YOLOv8 (deep learning)

`controller.py` loads `yolov8n.pt` at startup - a YOLOv8 nano model pretrained on the 80-class COCO dataset. COCO class 11 is "stop sign". Every 3rd camera frame is passed through the model at 416×416 resolution with a confidence threshold of 0.45. The cached result is returned on skipped frames so the pipeline never stalls.

### YIELD sign - OpenCV shape detection

COCO does not include a yield sign class, so yield detection uses classical computer vision:
1. HSV masking isolates red pixels (two hue ranges to handle red's wrap-around in HSV)
2. Morphological closing fills gaps in the sign face
3. `cv2.approxPolyDP` approximates each red contour to a polygon
4. A contour with exactly 3 vertices, convex shape, and roughly square aspect ratio is classified as a yield sign

### State machine

```
SCANNING → (STOP detected) → STOPPED (3s) → COOLDOWN (5s) → SCANNING
SCANNING → (YIELD detected) → YIELDING (3s, 75% throttle) → COOLDOWN (5s) → SCANNING
```

---

## Milestones / Schedule Checklist

- [x] See what is already available for sign detection
- [x] Complete proposal document - *Due March 31*
- [x] Review and document existing class assignment code for camera detection and motor control - *TC, April 4*
- [x] Set up OpenCV pipeline for image preprocessing - *TC, April 9*
- [x] ~~Integrate Tesseract OCR~~ → **replaced by YOLOv8** (no external binary required) - *TC, April 14*
- [x] Combine sign detection with robot control logic - *TC, April 18*
- [x] Create progress report - *Due April 21*
- [x] Test full pipeline in dev mode with printed signs - *TC, April 25*
- [x] Add yield sign detection and 3-state machine - *TC, May 1*
- [x] Create final presentation - *Due May 5*
- [x] Provide system documentation (this README) - *Due May 15*

---

## Measures of Success

- [x] Detection pipeline successfully isolates sign regions from the camera feed
- [x] YOLOv8 correctly identifies a STOP sign in real-time video
- [x] Robot halts for 3 seconds upon STOP sign detection
- [x] Robot slows to 75% throttle for 3 seconds upon YIELD sign detection
- [x] 5-second cooldown prevents repeated triggering from the same sign
- [x] System documented clearly enough that a classmate could replicate the setup using this README
- [ ] Pipeline tested on the physical robot car in the lab environment

---

## Status Update - May 10, 2026

The full detection pipeline is complete and validated in dev mode. I pivoted away from Tesseract OCR early on - it required a system-level binary install and was slow (~300 ms/frame), which would have blocked the WebSocket keepalive and caused disconnects. Instead I landed on YOLOv8n for STOP signs (pretrained on COCO, runs ~50 ms/frame on CPU with no binary dependencies) and OpenCV contour analysis for YIELD signs (COCO doesn't include yield). Both detections are working reliably on camera with printed signs. The state machine handles the 3-second halt/slow and 5-second cooldown cleanly. Physical robot testing on the lab car was not completed due to network connectivity issues - this is documented in Future Work below.

---

## References

### Helpful
- [Ultralytics YOLOv8 Documentation](https://docs.ultralytics.com/) - model loading, inference API, class filtering
- [COCO Dataset Class List](https://cocodataset.org/#explore) - confirmed "stop sign" is class 11; confirmed yield sign is absent
- [OpenCV Documentation - contour approximation](https://docs.opencv.org/4.x/dd/d49/tutorial_py_contour_features.html) - `approxPolyDP`, `isContourConvex`
- [OpenCV HSV Color Ranges](https://docs.opencv.org/4.x/df/d9d/tutorial_py_colorspaces.html) - understanding red hue wrap-around at 0/180
- [UB Racer Framework README](python/) - callback API, dev mode usage, racerlib methods

### Tried but less helpful
- Tesseract OCR (pytesseract) - too slow for real-time use and requires a system binary; ultimately not used
- Template matching (`cv2.matchTemplate`) - too sensitive to scale and angle changes for a moving camera

---

## Future Work

### Physical robot testing
The pipeline was validated entirely in dev mode. Connecting to the lab car requires being on the correct lab network with the host server running. This should be the next step - the code is ready; it is a network/access issue.

### Custom YOLO model for yield signs
The current yield detector uses red triangle shape detection, which works well in controlled conditions but could false-positive on other red triangular objects. Training a small YOLO model on a traffic sign dataset (e.g. [GTSRB](https://benchmark.ini.rub.de/) or a [Roboflow traffic signs dataset](https://roboflow.com/)) would make yield detection as robust as stop detection.

### Additional sign types
The architecture supports adding new signs with minimal code changes. Natural next additions:
- Speed limit signs (requires OCR or a custom model to read the number)
- Pedestrian crossing signs
- One-way / do-not-enter signs

### Confidence threshold tuning
The STOP sign threshold is set to 0.45. At longer distances or in low light the model confidence drops. Adaptive thresholding (e.g. lower threshold when the car is moving slowly) could improve reliability.

### Auto re-queue after session
Uncommenting `conn.join()` in `on_session_end` would allow the robot to automatically re-enter the queue after each session, enabling fully autonomous lap runs without manual intervention.

---

## Repository Structure

```
ub_racer/client/
├── README.md                   ← this file
├── images/                     ← screenshots referenced above
│   ├── stop_detection.png
│   └── yield_detection.png
├── html/                       Browser UI (index.html + index.js)
├── python/
│   ├── server.py               Client web server
│   ├── controller.py           Sign detection + driving logic (main project file)
│   └── lib/
│       └── racerlib.py         UB Racer backend library (do not edit)
├── requirements.txt            Python dependencies
└── messages.md                 WebSocket message protocol reference
```
