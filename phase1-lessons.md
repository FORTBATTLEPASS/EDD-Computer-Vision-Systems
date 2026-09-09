# Phase 1 Lessons — Environment Setup (Revised)

These lessons replace the original Phase 1 content. The original version
had students install libraries one at a time and troubleshoot version
conflicts as they appeared (numpy/opencv/mediapipe ABI mismatches, protobuf
errors, picamera2 venv visibility, etc.). That was valuable for
understanding *why* dependency management matters, but it cost significant
class time. This revision front-loads the correct, tested installation
order so students spend lesson time on computer vision concepts instead of
environment debugging.

---

## Phase 1: Environment Setup (Day 1–2)

### Learning Objectives
- Understand what a virtual environment is and why it isolates project
  dependencies
- Understand why version compatibility matters between core libraries
  (NumPy, OpenCV, MediaPipe)
- Successfully run a live camera feed through OpenCV

### Day 1: Imaging and First Boot

1. Each student pair images their own SD card using **Raspberry Pi Imager**:
   - OS: **Raspberry Pi OS (64-bit)** — Bookworm
   - Click the gear icon and set:
     - Unique hostname (e.g., `<lastname_initial><period>`)
     - Unique username/password
     - Enable SSH
2. Boot the Pi, connect via SSH from a lab computer.
3. Discuss as a class: *why does each Pi need a unique hostname?* (Preview
   of networking/identity concepts that will matter later with RPi Connect.)

### Day 2: Running the Bootstrap Script

Rather than installing libraries one at a time and hitting errors, students
run a single tested bootstrap script and **read what it's doing** as it
runs — this is where the "why" gets taught, just without the trial-and-error
cost.

```bash
git clone https://github.com/YOUR_ORG/YOUR_TEACHER_REPO.git ~/setup
cd ~/setup
bash bootstrap_pi.sh
```

**Classroom discussion points while it runs (~5–10 minutes):**
- Why do we pin `numpy<2.0`? (NumPy 2.0 changed its internal binary
  interface — older compiled libraries like OpenCV and Picamera2 can crash
  if a newer NumPy is installed on top of them.)
- Why use `--system-site-packages` when creating the virtual environment?
  (Some libraries, like Picamera2, are tied closely to the Pi's OS and
  camera drivers — they're easiest to install system-wide via `apt` rather
  than through `pip` inside an isolated venv.)
- Why pin `mediapipe==0.10.18` specifically? (Newer versions changed their
  Python API; this version matches the lesson code students will use.)

**Manual steps after the script finishes:**
1. `rpi-connect signin` — link the device to the class Raspberry Pi Connect
   account (requires a browser).
2. `sudo raspi-config` → enable Wayland (Wayfire/Labwc) and Desktop
   Autologin (needed for remote screen sharing later).
3. Reboot and verify:
   ```bash
   which python3
   python3 ~/Documents/scripts/pose_basic.py
   ```

**Note on the camera:** Bookworm auto-detects a connected camera module —
there is no "enable camera" toggle needed in `raspi-config`, unlike older
Raspberry Pi OS releases. Confirm detection with:
```bash
rpicam-hello --list-cameras
```

### Engineering Notebook Entry #1 (Day 3, per original rubric)
- BASH setup log (what the bootstrap script did, in the student's own words)
- Camera pipeline sketch
- HSV reflection paragraph

---

## Why This Revision Matters

The original pilot run of this curriculum required reverting a partially
broken environment back to a fresh Bookworm image and rebuilding the
Python environment correctly, discovering each dependency conflict one at
a time. This revision captures the exact working order (documented in the
teacher setup repo's `bootstrap_pi.sh` and `TEACHER_README.md`) so that
future runs of this class start from a known-good state on Day 1, rather
than partway through Phase 1.
