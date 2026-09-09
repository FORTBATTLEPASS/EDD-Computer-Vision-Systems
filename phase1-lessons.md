# 🔧 Phase 1 — Foundations (Days 1–4)

**PLTW Engineering Design & Development | Lynwood High School**
**Computer Vision Systems Unit**

This phase gets your Raspberry Pi environment fully working and introduces
the camera, OpenCV basics, and the first design-thinking step of the unit.

> **Key:** 📓 = Notebook entry due | 🔧 = Hands-on lab | 🎨 = Design deliverable

***

## 📅 Phase 1 Schedule

| Day | Topic | Deliverables |
|---|---|---|
| **Day 1** | BASH review, SSH, virtual environment setup, OS update | Repo created, venv activated, camera tested 🔧 |
| **Day 2** | Camera optics: focal length, FOV, sensor specs, capture modes | FOV calculation table, image captures at multiple resolutions 🔧 📓 Entry #1 |
| **Day 3** | Intro to OpenCV: live feed, image ops, HSV color space | `day3_annotated.py` pushed to repo 🔧 📓 Entry #1 (cont.) |
| **Day 4** | EDD design process: problem identification, concept sketches | 3 concept sketches per partner 🎨 📓 Entry #2 |

***

## Day 1: BASH Review, SSH, Environment Setup

### Learning Objectives
- Navigate a Linux filesystem confidently with BASH
- Connect to your Pi remotely over SSH
- Understand what a Python virtual environment is and why we isolate
  project dependencies inside one
- Confirm your camera hardware is detected and working

### Imaging Your Pi (before class, or first 10 minutes)

Each partner team images their own SD card using **Raspberry Pi Imager**:

- OS: **Raspberry Pi OS (64-bit)** — Bookworm
- Click the gear icon (OS customization) and set:
  - A unique hostname for your Pi
  - A unique username and password
  - Enable SSH

> **Note:** on Bookworm, there is **no separate "enable camera" step**
> like older Raspberry Pi OS releases required — the camera is
> auto-detected by libcamera as soon as it's connected. You'll confirm
> this later in the lab.

### BASH Review

Practice these commands with your partner — you'll use all of them daily
for the rest of the unit:

```bash
pwd                     # print working directory
ls -la                  # list files, including hidden
cd ~/Documents          # change directory
mkdir scripts           # make a new directory
touch test.py           # create an empty file
cat test.py              # print file contents
nano test.py             # edit a file in terminal
history                 # see recent commands
```

### Connecting via SSH

From your lab computer:

```bash
ssh <your-username>@<your-hostname>.local
```

If `.local` doesn't resolve, find your Pi's IP address on your router's
device list and use that instead:

```bash
ssh <your-username>@<PI_IP>
```

### Hands-On Lab: Environment Setup

Run the following on your Pi. This installs everything you'll need for
the **entire unit** in one pass — you will not need to install anything
new for Phases 1–3.

```bash
# System update
sudo apt update && sudo apt full-upgrade -y

# Install RPi Connect (not offered as a Raspberry Pi Imager option on Bookworm)
sudo apt install -y rpi-connect
rpi-connect on

# Confirm camera detection (Bookworm auto-detects — no manual enable needed)
rpicam-hello --list-cameras

# Install system-level packages
sudo apt install -y \
  python3-full python3-venv python3-pip \
  python3-opencv python3-picamera2 \
  libatlas-base-dev libhdf5-dev libgtk-3-0 libcap-dev \
  libcamera-dev libkms++-dev libfmt-dev libdrm-dev ffmpeg git

# Create your project folders (matches the required repo structure)
mkdir -p ~/cv_project/{scripts,models,images/known_faces,data,notebooks/,design/concept_sketches,docs}
cd ~/cv_project

# Create the virtual environment (system-site-packages lets it see picamera2/opencv)
python3 -m venv --system-site-packages cv_env
echo "source ~/cv_project/cv_env/bin/activate" >> ~/.bashrc
source ~/.bashrc

# Pin numpy to a 1.x version — required so OpenCV, Picamera2, and later
# libraries in this unit don't crash from a NumPy 2.x binary mismatch
pip install --upgrade pip wheel
pip install "numpy>=1.25,<2.0" --force-reinstall

# Install MediaPipe (pinned version, no-deps to avoid pulling in numpy 2 / duplicate opencv)
pip install "mediapipe==0.10.18" --no-deps
pip install "protobuf>=4.25.3,<5" --force-reinstall
pip install absl-py flatbuffers sounddevice matplotlib --no-deps
pip install cffi pycparser contourpy cycler fonttools kiwisolver packaging pyparsing python-dateutil six

# Install remaining unit libraries
pip install tflite-runtime face_recognition
```

**Why we pin these specific versions (discuss as a class):**
- `numpy<2.0` — NumPy 2.0 changed its internal binary interface. Libraries
  already compiled against NumPy 1.x (like the system `opencv` and
  `picamera2` packages) can crash or throw `ImportError` if a newer NumPy
  is installed on top of them.
- `--system-site-packages` on the venv — Picamera2 is tied closely to the
  Pi's camera drivers and is easiest installed system-wide via `apt`
  rather than `pip` inside an isolated venv.
- `mediapipe==0.10.18` — this is the version whose Python API
  (`mp.solutions.pose`, `mp.solutions.face_detection`, etc.) matches every
  script you'll write in this unit. Newer MediaPipe releases changed this
  API.

### Set Up Your Repo

```bash
cd ~/cv_project
git init
git add .
git commit -m "Initial repo structure"
git branch -M main
git remote add origin https://github.com/YOUR_USERNAME/YOUR_REPO_NAME.git
git push -u origin main
```

### Set Up Co-Author Commit Alias

```bash
alias gcp='git commit --template ~/.git_coauthor_template'
echo "
Co-authored-by: Partner Full Name <partner@email.com>" > ~/.git_coauthor_template
```

### Test the Camera

```bash
source ~/cv_project/cv_env/bin/activate
python3 -c "import numpy as np, cv2, mediapipe as mp; print('numpy', np.__version__, 'cv2', cv2.__version__, 'mediapipe', mp.__version__)"
```

Then capture a quick test image:

```bash
rpicam-jpeg -o ~/cv_project/images/test.jpg
```

### Manual Steps (finish before end of Day 1)

**Sign in to RPi Connect** (requires a browser on any device):

```bash
rpi-connect signin
```

**Enable Wayland compositor + Desktop Autologin** (needed later for RPi
Connect screen sharing to work):

```bash
sudo raspi-config
```
- **Advanced Options → Wayland** → choose **Wayfire** (or **Labwc**)
- Back to the main menu (don't exit yet)
- **System Options → Boot / Auto Login** → choose **Desktop Autologin**
- Exit and reboot

### End of Day 1 Checklist
- [ ] Repo created and pushed with correct folder structure
- [ ] `cv_env` activates automatically on login
- [ ] Camera detected via `rpicam-hello --list-cameras`
- [ ] Test image successfully captured
- [ ] RPi Connect signed in and Wayland/autologin configured

***

## Day 2: Camera Optics — FOV, Focal Length, Sensor Specs

### Learning Objectives
- Understand focal length, field of view (FOV), and sensor size
  relationships
- Capture images at multiple resolutions and compare quality/performance
  tradeoffs

### Key Concepts

The **field of view** of your Camera Module V2 depends on its fixed focal
length and sensor size. You'll use this relationship again in Phase 3 when
calculating mounting distance for your final project.

\[
FOV = 2 \arctan\left(\frac{d}{2f}\right)
\]

where \(d\) is the sensor dimension (width or height) and \(f\) is the
focal length.

### Hands-On Lab

Capture test images at a few different resolutions and note file size and
capture time:

```bash
rpicam-jpeg -o ~/cv_project/images/res_640x480.jpg --width 640 --height 480
rpicam-jpeg -o ~/cv_project/images/res_1280x720.jpg --width 1280 --height 720
rpicam-jpeg -o ~/cv_project/images/res_1920x1080.jpg --width 1920 --height 1080
```

Build a table in your notebook: resolution vs. file size vs. capture
time vs. subjective image quality.

### 📓 Notebook Entry #1 (Due Day 3, started today)
- FOV calculation table (show your math using the formula above)
- Image captures at multiple resolutions attached/described

***

## Day 3: Intro to OpenCV — Live Feed, Image Ops, HSV

### Learning Objectives
- Display a live camera feed using OpenCV and Picamera2
- Perform basic image operations (grayscale, blur, thresholding)
- Understand the HSV color space and why it's preferred over RGB for
  color-based detection

### Script: `day3_annotated.py`

```python
import cv2
from picamera2 import Picamera2

picam2 = Picamera2()
picam2.configure(picam2.create_preview_configuration(
    main={"format": "XRGB8888", "size": (640, 480)}
))
picam2.start()

try:
    while True:
        frame = picam2.capture_array()
        frame = cv2.cvtColor(frame, cv2.COLOR_BGRA2BGR)

        gray = cv2.cvtColor(frame, cv2.COLOR_BGR2GRAY)
        blurred = cv2.GaussianBlur(frame, (15, 15), 0)
        hsv = cv2.cvtColor(frame, cv2.COLOR_BGR2HSV)

        cv2.imshow("Original", frame)
        cv2.imshow("Grayscale", gray)
        cv2.imshow("Blurred", blurred)
        cv2.imshow("HSV", hsv)

        if cv2.waitKey(20) & 0xFF == ord("q"):
            break

except KeyboardInterrupt:
    print("Interrupted by user")

finally:
    picam2.stop()
    cv2.destroyAllWindows()
```

Push this script to your repo before end of class:

```bash
git add scripts/day3_annotated.py
git commit -m "Add day 3 OpenCV feed with grayscale, blur, and HSV views

Co-authored-by: Partner Full Name <partner@email.com>"
git push
```

### 📓 Notebook Entry #1 (continued — due today)
- BASH setup log (what you ran on Day 1 and why, in your own words)
- Camera pipeline sketch (draw: camera → Picamera2 → OpenCV → display)
- HSV reflection paragraph: why does HSV separate color information more
  usefully than RGB for detecting a specific color under changing light?

***

## Day 4: EDD Design Process — Problem ID & Concept Sketches

### Learning Objectives
- Apply the engineering design process to identify a real-world problem
  your CV system could solve
- Produce initial concept sketches, individually, before settling on a
  team direction

### Activity

Review the example final system concepts in the main README (smart
birdfeeder, security door system, fitness rep counter, etc.) — but you are
not limited to these.

**Individually** (not with your partner yet), sketch **3 concept ideas**
for a computer vision system that solves a problem you care about. For
each sketch, note:
- What problem does it solve, and who is the user?
- Which CV technique might it use (you'll learn these in Phase 2 —
  color tracking, object detection, pose estimation, or face recognition)?
- What would "the system works" look like in a live demo?

### 📓 Notebook Entry #2 (Due today)
- Problem statement (one paragraph, individually written)
- 3 annotated concept sketches (hand-drawn, in your physical notebook)
- Partner comparison paragraph: after sharing sketches with your partner,
  what similarities and differences did you notice in your ideas?

***

## End of Phase 1 Checklist

- [ ] Repo structure fully created and pushed
- [ ] `cv_env` working with pinned numpy/opencv/mediapipe versions
- [ ] Camera tested at multiple resolutions
- [ ] `day3_annotated.py` pushed with co-authored commit
- [ ] Notebook Entries #1 and #2 complete for both partners
- [ ] RPi Connect signed in, Wayland + autologin configured

You're now ready for Phase 2, where you'll build the four core computer
vision skill modules used throughout the rest of the unit.
