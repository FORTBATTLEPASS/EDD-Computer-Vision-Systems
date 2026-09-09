# 🚀 Phase 4 — Build, Test & Present (Days 16–20)

**PLTW Engineering Design & Development | Lynwood High School**
**Computer Vision Systems Unit**

This is the final phase of the unit. Your design is locked in from Phase 3
— now you build the real system, test it rigorously, and present it live
to the class. No new libraries should be needed here; everything you
installed in Phase 1 and every module you built in Phase 2 is your toolkit
for the final build.

> **Key:** 🔧 = Build sprint | 🧪 = Testing | 📓 = Notebook entry due | 🎤 = Presentation

***

## 📅 Phase 4 Schedule

| Day | Topic | Deliverables |
|---|---|---|
| **Day 16** | Build Sprint 1: core detection/tracking logic | `final_project.py` (v1) pushed 🔧 |
| **Day 17** | Build Sprint 2: output/action logic + integration | `final_project.py` (v2) pushed 🔧 |
| **Day 18** | Build Sprint 3: refinement, edge cases, error handling | `final_project.py` (v3) pushed 🔧 |
| **Day 19** | Final test protocol — 20+ trials, metrics table | Test results + reflection 🧪 📓 Entry #10 |
| **Day 20** | Presentations | Live demo, 5 min + 2 min Q&A 🎤 |

***

## Day 16: Build Sprint 1 — Core Detection/Tracking Logic

### Learning Objectives
- Translate your Phase 3 system architecture diagram into working code
- Implement the core CV logic identified in your design brief (color
  tracking, object detection, pose estimation, and/or face recognition)

### Getting Started

Create your final project script, building directly on whichever Phase 2
module(s) your design brief specifies:

```bash
touch ~/cv_project/scripts/final_project.py
```

Structure your script around the architecture diagram from Day 15:

```python
# final_project.py — Build Sprint 1
# Core detection/tracking logic only. Output/action logic comes in Sprint 2.

import cv2
from picamera2 import Picamera2

# TODO: import whichever Phase 2 module(s) your design uses
# e.g., import mediapipe as mp   OR   from tflite_runtime.interpreter import Interpreter

picam2 = Picamera2()
picam2.configure(picam2.create_preview_configuration(
    main={"format": "XRGB8888", "size": (640, 480)}
))
picam2.start()

try:
    while True:
        frame = picam2.capture_array()
        frame = cv2.cvtColor(frame, cv2.COLOR_BGRA2BGR)

        # TODO: run your core detection/tracking logic here
        # (adapt from color_tracking.py, object_detect.py, pose_angles.py,
        # or face_recognize.py depending on your design brief)

        cv2.imshow("Final Project", frame)
        if cv2.waitKey(20) & 0xFF == ord("q"):
            break

except KeyboardInterrupt:
    print("Interrupted by user")

finally:
    picam2.stop()
    cv2.destroyAllWindows()
```

### Focus for Today

Get the **core detection working reliably** before adding any output
logic — confirm your system correctly detects/tracks/recognizes its
target under normal conditions before layering on complexity.

Push before end of class:
```bash
git add scripts/final_project.py
git commit -m "Build Sprint 1: implement core detection logic for [your CV technique]

Co-authored-by: Partner Full Name <partner@email.com>"
git push
```

***

## Day 17: Build Sprint 2 — Output/Action Logic + Integration

### Learning Objectives
- Add the output/action layer defined in your system architecture (e.g.,
  on-screen display, counter, CSV logging, audio/visual alert)
- Integrate detection and output into a single working pipeline

### Focus for Today

Extend `final_project.py` with the action logic your design brief
promised — this is the part that makes your system actually *do*
something useful with a detection, not just display a bounding box.

Examples based on common design patterns from Phase 2:
- **Counter output** (like `pose_angles.py`'s rep counter) — display a
  running count on screen
- **Logging output** (like `detection_logger.py`) — write detection
  events with timestamps to a CSV in `data/`
- **Access/alert output** (like `face_recognize.py`) — trigger a visual
  or audio cue when a specific person/object is recognized

Test the **full pipeline end to end** today — detection feeding into
output — even if it's not polished yet. Polish and edge cases are
tomorrow's job.

Push before end of class:
```bash
git add scripts/final_project.py
git commit -m "Build Sprint 2: integrate output/action logic with detection pipeline

Co-authored-by: Partner Full Name <partner@email.com>"
git push
```

***

## Day 18: Build Sprint 3 — Refinement, Edge Cases, Error Handling

### Learning Objectives
- Identify and handle edge cases and failure modes
- Add defensive error handling so the system fails gracefully instead of
  crashing during the live demo

### Focus for Today

Revisit your Day 14 risk log — this is the day to actually address those
risks in code, not just on paper. Common refinements:

- **No detection in frame:** what does your system do when nothing is
  detected for several seconds? Does it crash, freeze, or handle it
  gracefully?
- **Multiple simultaneous detections:** if your design assumes one
  target, what happens with two? Does it need to?
- **Lighting/angle sensitivity:** if Phase 2 testing showed your
  technique struggles under certain conditions, can you mitigate it here
  (e.g., adjustable thresholds, a confidence floor, multi-frame
  smoothing)?
- **Camera/hardware errors:** wrap camera startup in a
  try/except so a loose ribbon cable produces a clear error message
  instead of a silent crash.

Example defensive pattern:

```python
try:
    picam2.start()
except Exception as e:
    print(f"Camera failed to start: {e}")
    print("Check the ribbon cable connection and try again.")
    exit(1)
```

This is also the day to clean up your code: remove debug print statements,
add comments explaining non-obvious logic, and make sure variable names
are clear — someone else (like your instructor) should be able to read
your script and understand what it does.

Push before end of class:
```bash
git add scripts/final_project.py
git commit -m "Build Sprint 3: add error handling and refine edge case behavior

Co-authored-by: Partner Full Name <partner@email.com>"
git push
```

***

## Day 19: Final Test Protocol — 20+ Trials

### Learning Objectives
- Design and run a rigorous test protocol for your finished system
- Report quantitative results (detection rate, false positive rate) and
  reflect on the overall design process

### Final Test Protocol

Run **at least 20 trials** of your finished system, varying conditions
systematically. Design your own protocol based on what matters for your
specific system, but structure it like this:

| Trial # | Condition Varied | Expected Result | Actual Result | Pass/Fail |
|---|---|---|---|---|
| 1 | Normal lighting, target at 1m | Detected | | |
| 2 | Normal lighting, target at 2m | Detected | | |
| 3 | Low lighting, target at 1m | Detected | | |
| ... | ... | ... | ... | ... |
| 20 | Target absent (negative control) | Not detected | | |

Make sure your 20+ trials cover:
- Multiple distances/angles
- Multiple lighting conditions
- At least one **negative control** (system correctly does *not* trigger
  when it shouldn't)
- Both partners' faces/objects/movements if relevant to your design

### Calculate Summary Metrics

From your trial data, calculate and report:
- **Detection rate:** (correct detections) / (total trials where target
  was present)
- **False positive rate:** (incorrect detections) / (total trials where
  target was absent)
- **Any patterns:** does performance degrade under specific conditions?

Save your full results table and summary metrics to
`docs/test_results.md`.

### Design Process Reflection

Write a reflection (this can be a shared team paragraph, distinct from
each partner's individual notebook entries) addressing:
- How did your final system compare to your original Day 13 design brief
  — did success criteria change, and why?
- What was the biggest technical obstacle, and how did you solve it?
- If you had two more weeks, what would you improve or add?

Push before end of class:
```bash
git add docs/test_results.md
git commit -m "Add final test protocol results and design process reflection

Co-authored-by: Partner Full Name <partner@email.com>"
git push
```

### 📓 Notebook Entry #10 (Due today)
- Final test protocol results (20+ trials) + metrics table
- Design process reflection

***

## Day 20: Presentations 🎤

### Format

**5 minutes presentation + 2-minute Q&A per team.**

### Presentation Structure

1. **Problem (30 sec)** — What problem does your system solve? Who is the
   user?
2. **Design Decisions (1 min)** — Walk through your Day 13 decision
   matrix: what concepts did you consider, and why did you choose this
   one? Show a concept sketch.
3. **Live Demo (2 min)** — Run your system live in front of the class.
   Narrate what is happening.
4. **Results (1 min)** — Share your Day 19 test metrics: detection rate,
   false positive rate, performance under different conditions. What
   worked? What didn't?
5. **Future Improvements (30 sec)** — If you had two more weeks, what
   would you improve or add?

### Presentation Expectations

- **Both partners must speak** for roughly equal time
- The demo must be **live** — not a pre-recorded video
- Slides are optional but not required; the demo is the main event
- Be prepared to answer technical questions such as:
  - "How did you choose your confidence threshold?"
  - "What does your system do if it gets a false positive?"
  - "Why did you choose TFLite over OpenCV for detection?"
  - "How did you calculate the right mounting distance for your camera?"

### Pre-Demo Checklist (run through this the morning of Day 20)

- [ ] `final_project.py` runs cleanly with a single command from a fresh
      terminal (test this — don't assume it still works from yesterday's
      session)
- [ ] Camera is mounted at the distance/angle calculated on Day 15
- [ ] Lighting in the presentation room has been checked against your
      Day 19 test conditions
- [ ] Both partners have run the live demo at least once today before
      presenting
- [ ] Repo is fully pushed — commit history, notebook scans, design docs,
      and test results are all visible on GitHub

***

## Final Grading Rubric Overview

| Category | Weight | Description |
|---|---|---|
| Engineering Notebook (×2, individual) | 30% | 10 required entries, dated, complete, uploaded to repo |
| Repository & Commit History | 20% | Structure followed, both partners co-authoring, descriptive messages |
| Design Deliverables | 20% | Design brief, decision matrix, Gantt chart, architecture diagram |
| Final System Performance | 15% | Meets stated design criteria; accuracy/reliability data presented |
| Presentation & Demo | 15% | Both partners speak, live demo runs, technical questions answered |

***

## End of Unit Checklist

- [ ] `final_project.py` complete, tested, and pushed
- [ ] `docs/test_results.md` with 20+ trial results and metrics
- [ ] Notebook Entry #10 complete for both partners
- [ ] All 10 notebook entries scanned and uploaded to `notebooks/`
- [ ] Presentation delivered on Day 20 — live demo, both partners
      speaking, Q&A handled

Congratulations — this completes the Computer Vision Systems unit.
