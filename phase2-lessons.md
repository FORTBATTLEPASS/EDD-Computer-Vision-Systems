# 🔧 Phase 2 — CV Skill Building (Days 5–12)

**PLTW Engineering Design & Development | Lynwood High School**
**Computer Vision Systems Unit**

In this phase you'll build four core computer vision skill modules —
color tracking, object detection, pose estimation, and face recognition.
Your environment was fully set up in Phase 1, so **no new library
installations should be required** in this phase. If a script throws an
import error, check that `cv_env` is active before assuming a package is
missing.

> **Key:** 📓 = Notebook entry due | 🔧 = Hands-on lab

***

## 📅 Phase 2 Schedule

| Day | Topic | Deliverables |
|---|---|---|
| **Day 5** | Module A: Color tracking with HSV masking and trackbars | `color_tracking.py` 🔧 |
| **Day 6** | Module A: Shape detection with contours and Hough transforms | `shape_detection.py` 🔧 📓 Entry #3 |
| **Day 7** | Module B: TFLite object detection — model setup and inference | `object_detect.py` 🔧 |
| **Day 8** | Module B: Filtering detections and logging to CSV | `detection_logger.py` + CSV output 🔧 📓 Entry #4 |
| **Day 9** | Module C: Pose estimation with MediaPipe — 33 keypoints | `pose_basic.py` 🔧 |
| **Day 10** | Module C: Joint angle calculation and gesture logic | `pose_angles.py` 🔧 📓 Entry #5 |
| **Day 11** | Module D: Face detection with Haar cascades + Face Mesh | `face_detect.py` 🔧 |
| **Day 12** | Module D: Face recognition — known vs. unknown classifier | `face_recognize.py` 🔧 📓 Entry #6 |

***

## Module A: Color Tracking & Shape Detection (Days 5–6)

### Day 5: Color Tracking with HSV Masking and Trackbars

#### Learning Objectives
- Build an HSV mask to isolate a specific color range
- Use OpenCV trackbars to tune HSV thresholds interactively
- Track the largest contour of the masked color in real time

#### Script: `color_tracking.py`

```python
import cv2
import numpy as np
from picamera2 import Picamera2

def nothing(x):
    pass

cv2.namedWindow("Trackbars")
cv2.createTrackbar("H Min", "Trackbars", 0, 179, nothing)
cv2.createTrackbar("H Max", "Trackbars", 179, 179, nothing)
cv2.createTrackbar("S Min", "Trackbars", 100, 255, nothing)
cv2.createTrackbar("S Max", "Trackbars", 255, 255, nothing)
cv2.createTrackbar("V Min", "Trackbars", 100, 255, nothing)
cv2.createTrackbar("V Max", "Trackbars", 255, 255, nothing)

picam2 = Picamera2()
picam2.configure(picam2.create_preview_configuration(
    main={"format": "XRGB8888", "size": (640, 480)}
))
picam2.start()

try:
    while True:
        frame = picam2.capture_array()
        frame = cv2.cvtColor(frame, cv2.COLOR_BGRA2BGR)
        hsv = cv2.cvtColor(frame, cv2.COLOR_BGR2HSV)

        h_min = cv2.getTrackbarPos("H Min", "Trackbars")
        h_max = cv2.getTrackbarPos("H Max", "Trackbars")
        s_min = cv2.getTrackbarPos("S Min", "Trackbars")
        s_max = cv2.getTrackbarPos("S Max", "Trackbars")
        v_min = cv2.getTrackbarPos("V Min", "Trackbars")
        v_max = cv2.getTrackbarPos("V Max", "Trackbars")

        lower = np.array([h_min, s_min, v_min])
        upper = np.array([h_max, s_max, v_max])
        mask = cv2.inRange(hsv, lower, upper)

        contours, _ = cv2.findContours(mask, cv2.RETR_EXTERNAL, cv2.CHAIN_APPROX_SIMPLE)
        if contours:
            largest = max(contours, key=cv2.contourArea)
            if cv2.contourArea(largest) > 500:
                x, y, w, h = cv2.boundingRect(largest)
                cv2.rectangle(frame, (x, y), (x + w, y + h), (0, 255, 0), 2)

        cv2.imshow("Frame", frame)
        cv2.imshow("Mask", mask)

        if cv2.waitKey(20) & 0xFF == ord("q"):
            break

except KeyboardInterrupt:
    print("Interrupted by user")

finally:
    picam2.stop()
    cv2.destroyAllWindows()
```

Push before end of class:
```bash
git add scripts/color_tracking.py
git commit -m "Add HSV color tracking with live trackbar tuning

Co-authored-by: Partner Full Name <partner@email.com>"
git push
```

### Day 6: Shape Detection with Contours and Hough Transforms

#### Learning Objectives
- Detect geometric shapes (circles, rectangles, triangles) using contour
  approximation
- Use the Hough Circle Transform to detect circular objects specifically

#### Script: `shape_detection.py`

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
        blurred = cv2.GaussianBlur(gray, (5, 5), 0)
        edges = cv2.Canny(blurred, 50, 150)

        contours, _ = cv2.findContours(edges, cv2.RETR_EXTERNAL, cv2.CHAIN_APPROX_SIMPLE)

        for contour in contours:
            if cv2.contourArea(contour) < 300:
                continue
            approx = cv2.approxPolyDP(contour, 0.02 * cv2.arcLength(contour, True), True)
            x, y = approx[0][0]
            sides = len(approx)

            if sides == 3:
                shape = "Triangle"
            elif sides == 4:
                shape = "Rectangle"
            elif sides > 6:
                shape = "Circle"
            else:
                shape = f"{sides}-gon"

            cv2.drawContours(frame, [approx], -1, (0, 255, 0), 2)
            cv2.putText(frame, shape, (x, y - 10),
                        cv2.FONT_HERSHEY_SIMPLEX, 0.6, (0, 255, 0), 2)

        cv2.imshow("Shape Detection", frame)
        cv2.imshow("Edges", edges)

        if cv2.waitKey(20) & 0xFF == ord("q"):
            break

except KeyboardInterrupt:
    print("Interrupted by user")

finally:
    picam2.stop()
    cv2.destroyAllWindows()
```

### 📓 Notebook Entry #3 (Due today)
- Color tracking + shape detection lab results
- Screenshots of both scripts working
- Conditions tested (lighting, object color/shape)
- Failure cases (what didn't get detected, and your hypothesis why)

***

## Module B: TFLite Object Detection (Days 7–8)

### Day 7: Model Setup and Inference

#### Learning Objectives
- Load a pretrained TensorFlow Lite object detection model
- Run inference on live camera frames and draw bounding boxes with labels

#### Setup

```bash
mkdir -p ~/cv_project/models
cd ~/cv_project/models
wget https://storage.googleapis.com/download.tensorflow.org/models/tflite/coco_ssd_mobilenet_v1_1.0_quant_2018_06_29.zip -O model.zip
unzip model.zip
mv detect.tflite model.tflite
mv labelmap.txt labels.txt
```

#### Script: `object_detect.py`

```python
import cv2
import numpy as np
from picamera2 import Picamera2
from tflite_runtime.interpreter import Interpreter

MODEL_PATH = "models/model.tflite"
LABELS_PATH = "models/labels.txt"

with open(LABELS_PATH, "r") as f:
    labels = [line.strip() for line in f.readlines()]

interpreter = Interpreter(model_path=MODEL_PATH)
interpreter.allocate_tensors()
input_details = interpreter.get_input_details()
output_details = interpreter.get_output_details()
input_height = input_details[0]['shape'][1]
input_width = input_details[0]['shape'][2]

picam2 = Picamera2()
picam2.configure(picam2.create_preview_configuration(
    main={"format": "XRGB8888", "size": (640, 480)}
))
picam2.start()

try:
    while True:
        frame = picam2.capture_array()
        frame = cv2.cvtColor(frame, cv2.COLOR_BGRA2BGR)
        h, w, _ = frame.shape

        resized = cv2.resize(frame, (input_width, input_height))
        input_data = np.expand_dims(resized, axis=0)

        interpreter.set_tensor(input_details[0]['index'], input_data)
        interpreter.invoke()

        boxes = interpreter.get_tensor(output_details[0]['index'])[0]
        classes = interpreter.get_tensor(output_details[1]['index'])[0]
        scores = interpreter.get_tensor(output_details[2]['index'])[0]

        for i in range(len(scores)):
            if scores[i] > 0.5:
                ymin, xmin, ymax, xmax = boxes[i]
                x1, y1, x2, y2 = int(xmin * w), int(ymin * h), int(xmax * w), int(ymax * h)
                label = labels[int(classes[i])]

                cv2.rectangle(frame, (x1, y1), (x2, y2), (0, 255, 0), 2)
                cv2.putText(frame, f"{label} {scores[i]:.2f}", (x1, y1 - 10),
                            cv2.FONT_HERSHEY_SIMPLEX, 0.6, (0, 255, 0), 2)

        cv2.imshow("Object Detection", frame)
        if cv2.waitKey(20) & 0xFF == ord("q"):
            break

except KeyboardInterrupt:
    print("Interrupted by user")

finally:
    picam2.stop()
    cv2.destroyAllWindows()
```

### Day 8: Filtering Detections and Logging to CSV

#### Learning Objectives
- Filter detections by confidence threshold and target label(s)
- Log detection events with timestamps to a CSV file for later analysis

#### Script: `detection_logger.py`

```python
import cv2
import csv
import numpy as np
from datetime import datetime
from picamera2 import Picamera2
from tflite_runtime.interpreter import Interpreter

MODEL_PATH = "models/model.tflite"
LABELS_PATH = "models/labels.txt"
LOG_PATH = "data/detections.csv"
CONFIDENCE_THRESHOLD = 0.5
TARGET_LABELS = None  # e.g., {"person", "bird"} to filter, or None for all

with open(LABELS_PATH, "r") as f:
    labels = [line.strip() for line in f.readlines()]

interpreter = Interpreter(model_path=MODEL_PATH)
interpreter.allocate_tensors()
input_details = interpreter.get_input_details()
output_details = interpreter.get_output_details()
input_height = input_details[0]['shape'][1]
input_width = input_details[0]['shape'][2]

with open(LOG_PATH, "w", newline="") as f:
    writer = csv.writer(f)
    writer.writerow(["timestamp", "label", "confidence"])

picam2 = Picamera2()
picam2.configure(picam2.create_preview_configuration(
    main={"format": "XRGB8888", "size": (640, 480)}
))
picam2.start()

try:
    while True:
        frame = picam2.capture_array()
        frame = cv2.cvtColor(frame, cv2.COLOR_BGRA2BGR)
        h, w, _ = frame.shape

        resized = cv2.resize(frame, (input_width, input_height))
        input_data = np.expand_dims(resized, axis=0)

        interpreter.set_tensor(input_details[0]['index'], input_data)
        interpreter.invoke()

        boxes = interpreter.get_tensor(output_details[0]['index'])[0]
        classes = interpreter.get_tensor(output_details[1]['index'])[0]
        scores = interpreter.get_tensor(output_details[2]['index'])[0]

        for i in range(len(scores)):
            if scores[i] > CONFIDENCE_THRESHOLD:
                label = labels[int(classes[i])]
                if TARGET_LABELS and label not in TARGET_LABELS:
                    continue

                ymin, xmin, ymax, xmax = boxes[i]
                x1, y1, x2, y2 = int(xmin * w), int(ymin * h), int(xmax * w), int(ymax * h)
                cv2.rectangle(frame, (x1, y1), (x2, y2), (0, 255, 0), 2)
                cv2.putText(frame, f"{label} {scores[i]:.2f}", (x1, y1 - 10),
                            cv2.FONT_HERSHEY_SIMPLEX, 0.6, (0, 255, 0), 2)

                with open(LOG_PATH, "a", newline="") as f:
                    writer = csv.writer(f)
                    writer.writerow([datetime.now().isoformat(), label, round(float(scores[i]), 2)])

        cv2.imshow("Detection Logger", frame)
        if cv2.waitKey(20) & 0xFF == ord("q"):
            break

except KeyboardInterrupt:
    print("Interrupted by user")

finally:
    picam2.stop()
    cv2.destroyAllWindows()
```

### 📓 Notebook Entry #4 (Due today)
- TFLite detection accuracy table across **10 trials** with varying
  conditions (distance, lighting, object type/angle)

***

## Module C: Pose Estimation with MediaPipe (Days 9–10)

### Day 9: 33 Keypoints with `pose_basic.py`

#### Learning Objectives
- Understand keypoint-based pose detection using MediaPipe's 33-point
  body landmark model
- Draw the detected skeleton over a live camera feed

#### Script: `pose_basic.py`

```python
import cv2
import mediapipe as mp
from picamera2 import Picamera2

mp_pose = mp.solutions.pose
mp_drawing = mp.solutions.drawing_utils

pose = mp_pose.Pose(
    min_detection_confidence=0.5,
    min_tracking_confidence=0.5
)

picam2 = Picamera2()
picam2.configure(picam2.create_preview_configuration(
    main={"format": "XRGB8888", "size": (640, 480)}
))
picam2.start()

try:
    while True:
        frame = picam2.capture_array()
        frame = cv2.cvtColor(frame, cv2.COLOR_BGRA2BGR)

        rgb_frame = cv2.cvtColor(frame, cv2.COLOR_BGR2RGB)
        results = pose.process(rgb_frame)

        if results.pose_landmarks:
            mp_drawing.draw_landmarks(
                frame,
                results.pose_landmarks,
                mp_pose.POSE_CONNECTIONS
            )

        cv2.imshow("Pose Estimation", frame)
        if cv2.waitKey(20) & 0xFF == ord("q"):
            break

except KeyboardInterrupt:
    print("Interrupted by user")

finally:
    picam2.stop()
    cv2.destroyAllWindows()
```

**Discuss:** what are the 33 landmarks MediaPipe tracks? Why does the
script convert BGR → RGB before passing frames to MediaPipe?

### Day 10: Joint Angle Calculation and Gesture Logic

#### Learning Objectives
- Calculate the angle at a joint using three landmark points
- Use that angle to build simple gesture/rep-counting logic

#### Script: `pose_angles.py`

```python
import cv2
import math
import mediapipe as mp
from picamera2 import Picamera2

mp_pose = mp.solutions.pose
mp_drawing = mp.solutions.drawing_utils

pose = mp_pose.Pose(
    min_detection_confidence=0.5,
    min_tracking_confidence=0.5
)

picam2 = Picamera2()
picam2.configure(picam2.create_preview_configuration(
    main={"format": "XRGB8888", "size": (640, 480)}
))
picam2.start()


def calculate_angle(a, b, c):
    a, b, c = (a.x, a.y), (b.x, b.y), (c.x, c.y)
    radians = math.atan2(c[1] - b[1], c[0] - b[0]) - \
              math.atan2(a[1] - b[1], a[0] - b[0])
    angle = abs(math.degrees(radians))
    if angle > 180:
        angle = 360 - angle
    return angle


counter = 0
stage = None

try:
    while True:
        frame = picam2.capture_array()
        frame = cv2.cvtColor(frame, cv2.COLOR_BGRA2BGR)
        rgb_frame = cv2.cvtColor(frame, cv2.COLOR_BGR2RGB)
        results = pose.process(rgb_frame)

        if results.pose_landmarks:
            mp_drawing.draw_landmarks(frame, results.pose_landmarks, mp_pose.POSE_CONNECTIONS)
            landmarks = results.pose_landmarks.landmark

            shoulder = landmarks[mp_pose.PoseLandmark.RIGHT_SHOULDER]
            elbow = landmarks[mp_pose.PoseLandmark.RIGHT_ELBOW]
            wrist = landmarks[mp_pose.PoseLandmark.RIGHT_WRIST]
            angle = calculate_angle(shoulder, elbow, wrist)

            h, w, _ = frame.shape
            elbow_px = (int(elbow.x * w), int(elbow.y * h))
            cv2.putText(frame, f"{int(angle)} deg", elbow_px,
                        cv2.FONT_HERSHEY_SIMPLEX, 0.8, (0, 255, 0), 2, cv2.LINE_AA)

            if angle > 160:
                stage = "down"
            if angle < 60 and stage == "down":
                stage = "up"
                counter += 1

        cv2.rectangle(frame, (0, 0), (200, 80), (245, 117, 16), -1)
        cv2.putText(frame, "REPS", (15, 20), cv2.FONT_HERSHEY_SIMPLEX, 0.6, (0, 0, 0), 1, cv2.LINE_AA)
        cv2.putText(frame, str(counter), (10, 65), cv2.FONT_HERSHEY_SIMPLEX, 1.5, (255, 255, 255), 2, cv2.LINE_AA)
        cv2.putText(frame, "STAGE", (100, 20), cv2.FONT_HERSHEY_SIMPLEX, 0.6, (0, 0, 0), 1, cv2.LINE_AA)
        cv2.putText(frame, str(stage), (100, 65), cv2.FONT_HERSHEY_SIMPLEX, 1, (255, 255, 255), 2, cv2.LINE_AA)

        cv2.imshow("Pose Angles", frame)
        if cv2.waitKey(20) & 0xFF == ord("q"):
            break

except KeyboardInterrupt:
    print("Interrupted by user")

finally:
    picam2.stop()
    cv2.destroyAllWindows()
```

### 📓 Notebook Entry #5 (Due today)
- Keypoint diagram
- Angle calculation method (show the `atan2` trig formula used)
- Rep counter test results (accuracy across multiple trials, varying
  distance/lighting)

***

## Module D: Face Detection & Recognition (Days 11–12)

### Day 11: Face Detection with Haar Cascades + Face Mesh

#### Learning Objectives
- Detect a face bounding box using a Haar cascade classifier
- Compare that to MediaPipe's Face Mesh for dense facial landmark tracking
- Understand that detection ("a face is here") is different from
  recognition ("this face belongs to a specific person")

#### Script: `face_detect.py`

```python
import cv2
import mediapipe as mp
from picamera2 import Picamera2

face_cascade = cv2.CascadeClassifier(cv2.data.haarcascades + "haarcascade_frontalface_default.xml")

mp_face_mesh = mp.solutions.face_mesh
mp_drawing = mp.solutions.drawing_utils
mp_drawing_styles = mp.solutions.drawing_styles

face_mesh = mp_face_mesh.FaceMesh(
    max_num_faces=2,
    min_detection_confidence=0.5,
    min_tracking_confidence=0.5
)

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

        faces = face_cascade.detectMultiScale(gray, 1.1, 5)
        for (x, y, w, h) in faces:
            cv2.rectangle(frame, (x, y), (x + w, y + h), (255, 0, 0), 2)

        rgb_frame = cv2.cvtColor(frame, cv2.COLOR_BGR2RGB)
        results = face_mesh.process(rgb_frame)

        if results.multi_face_landmarks:
            for face_landmarks in results.multi_face_landmarks:
                mp_drawing.draw_landmarks(
                    frame, face_landmarks, mp_face_mesh.FACEMESH_TESSELATION,
                    landmark_drawing_spec=None,
                    connection_drawing_spec=mp_drawing_styles.get_default_face_mesh_tesselation_style()
                )

        cv2.imshow("Face Detection", frame)
        if cv2.waitKey(20) & 0xFF == ord("q"):
            break

except KeyboardInterrupt:
    print("Interrupted by user")

finally:
    picam2.stop()
    cv2.destroyAllWindows()
```

### Day 12: Face Recognition — Known vs. Unknown Classifier

#### Learning Objectives
- Build a reference library of known faces
- Classify a detected face as a known individual or "Unknown"
- Reflect on the ethical implications of face recognition technology

#### Setup: Add Reference Photos

```bash
mkdir -p ~/cv_project/images/known_faces
# Add one clear photo per person, named firstname_lastname.jpg
```

#### Script: `face_recognize.py`

```python
import cv2
import face_recognition
import os
from picamera2 import Picamera2

KNOWN_FACES_DIR = os.path.expanduser("~/cv_project/images/known_faces")

known_encodings = []
known_names = []

for filename in os.listdir(KNOWN_FACES_DIR):
    if filename.lower().endswith((".jpg", ".jpeg", ".png")):
        path = os.path.join(KNOWN_FACES_DIR, filename)
        image = face_recognition.load_image_file(path)
        encodings = face_recognition.face_encodings(image)
        if encodings:
            known_encodings.append(encodings[0])
            name = os.path.splitext(filename)[0].replace("_", " ").title()
            known_names.append(name)

print(f"Loaded {len(known_names)} known face(s): {known_names}")

picam2 = Picamera2()
picam2.configure(picam2.create_preview_configuration(
    main={"format": "XRGB8888", "size": (640, 480)}
))
picam2.start()

try:
    while True:
        frame = picam2.capture_array()
        frame = cv2.cvtColor(frame, cv2.COLOR_BGRA2BGR)
        rgb_frame = cv2.cvtColor(frame, cv2.COLOR_BGR2RGB)

        small_frame = cv2.resize(rgb_frame, (0, 0), fx=0.5, fy=0.5)
        face_locations = face_recognition.face_locations(small_frame)
        face_encodings = face_recognition.face_encodings(small_frame, face_locations)

        for (top, right, bottom, left), face_encoding in zip(face_locations, face_encodings):
            matches = face_recognition.compare_faces(known_encodings, face_encoding, tolerance=0.6)
            name = "Unknown"
            if True in matches:
                name = known_names[matches.index(True)]

            top, right, bottom, left = top * 2, right * 2, bottom * 2, left * 2
            cv2.rectangle(frame, (left, top), (right, bottom), (0, 255, 0), 2)
            cv2.putText(frame, name, (left, top - 10),
                        cv2.FONT_HERSHEY_SIMPLEX, 0.8, (0, 255, 0), 2, cv2.LINE_AA)

        cv2.imshow("Face Recognition", frame)
        if cv2.waitKey(20) & 0xFF == ord("q"):
            break

except KeyboardInterrupt:
    print("Interrupted by user")

finally:
    picam2.stop()
    cv2.destroyAllWindows()
```

**Discuss:** why downscale the frame before running `face_recognition`?
What does `tolerance=0.6` control? What happens with poor lighting,
glasses, or an angled face — test and log these cases.

### 📓 Notebook Entry #6 (Due today)
- Face recognition accuracy table (multiple trials: lighting, angle,
  accessories, both partners)
- **Individual** ethics reflection paragraph covering: false
  positive/negative risk in different real-world contexts, consent,
  documented bias in face recognition accuracy across demographics (see
  NIST's face recognition vendor test reports), and what your own limited
  test data can and cannot prove

***

## End of Phase 2 Checklist

- [ ] `color_tracking.py` and `shape_detection.py` pushed
- [ ] `object_detect.py` and `detection_logger.py` pushed with CSV output
- [ ] `pose_basic.py` and `pose_angles.py` pushed
- [ ] `face_detect.py` and `face_recognize.py` pushed
- [ ] Notebook Entries #3–#6 complete for both partners

You've now built all four core CV skill modules. Phase 3 shifts from
learning techniques to designing your final project system.
