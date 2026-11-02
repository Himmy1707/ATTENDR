# ATTENDR

**Classroom Analysis System** — automate attendance with PTZ cameras and face recognition, and analyse in-class student behaviour with YOLOv8.

> Capstone Project (COMP4913) · The Hong Kong Polytechnic University · Department of Computing  
> Author: **YUE Lok Him** · Supervisor: Dr LOU Wei  
> Contact: [lokhimworkemail@gmail.com](mailto:lokhimworkemail@gmail.com)

<p align="center">
  <img src="captureScreen/ui_homepage.png" alt="ATTENDR Home Page" width="820"/>
</p>

---

## Overview

ATTENDR is a classroom analysis system for university lectures and labs. It combines a **PTZ (pan-tilt-zoom) camera**, **face recognition**, and **YOLOv8 behaviour detection** so lecturers can:

1. Take attendance automatically by scanning seat points with the camera  
2. Record the class session  
3. Generate behaviour reports (e.g. listening, raising hand, standing, group discussion)  
4. Review past session records through a web dashboard  

---

## Features

| Feature | Description |
|--------|-------------|
| **Automated attendance** | PTZ camera visits each seat point; face recognition marks students present / late / absent |
| **Seat-point mapping** | Lecturers define seat locations with live preview and a face-size guide frame |
| **Smart seat routing** | Seat points are sorted before saving to follow a shorter camera path |
| **Auto-zoom** | After preset zoom, an extra auto-zoom step improves face capture success |
| **Behaviour analysis** | YOLOv8 classifies classroom behaviours from recorded video |
| **Web dashboard** | Lecturer UI for login, courses, attendance, reports, and exports |
| **PTZ control** | Remote VISCA control: pan, tilt, zoom, home |

---

## Core Functions

ATTENDR focuses on three main functions:

### 1. Automate Attendance Tracking

The PTZ camera moves through seat points via VISCA, captures each student's face, runs face detection / recognition, and records attendance. The loop continues until every mapped seat has been checked.

```
PTZ Camera → Student Face Tracking (VISCA) → Face Detection → Attendance Recording
                              ↑________________________ loop: next student _________________|
```

<p align="center">
  <img src="captureScreen/function1.png" alt="Function 1: Automate Attendance Tracking" width="820"/>
</p>

### 2. Student Behavior Monitoring

While (or after) class, the system records classroom video through the VISCA-controlled PTZ camera, then runs YOLOv8 trained on a custom classroom dataset to detect student behaviours and produce a per-student behaviour summary.

```
PTZ Camera → Record student behavior (VISCA) → Student Behavior Detection (YOLOv8) → Behavior summary
```

<p align="center">
  <img src="captureScreen/function2.png" alt="Function 2: Student Behavior Monitoring" width="820"/>
</p>

### 3. Class Monitoring System

A web dashboard that brings the classroom data together for lecturers:

- Taught courses and their details  
- Attendance records  
- Behaviour summaries for each student  

<p align="center">
  <img src="captureScreen/function3.png" alt="Function 3: Class Monitoring System" width="820"/>
</p>

---

## Challenge & Solution

### Challenge: Seat-point order is not the shortest path

When lecturers add seat points, the save order is often random (wherever they clicked). If attendance simply follows that insertion order, the camera path becomes inefficient — zigzagging and backtracking across the room.

**Problem:** camera visit order = seat insertion order → inefficient attendance route.

<p align="center">
  <img src="captureScreen/challenge.png" alt="Challenge: inefficient seat-point order" width="820"/>
</p>

### Solution: Sort seat points before saving

A self-designed sorting algorithm reorders seat points before they are saved, so attendance follows a shorter, more predictable camera path:

1. **From bottom to top**  
2. **One row at a time**  
3. **Right to left**  

Example: add order `5 → 2 → 1 → 4 → 6 → 3` becomes attendance order `6 → 5 → 4 → 3 → 2 → 1`.

<p align="center">
  <img src="captureScreen/howifixthechallenge.png" alt="Solution: sorted seat-point path for attendance" width="820"/>
</p>

---

## System Architecture

<p align="center">
  <img src="captureScreen/systemArchitecture.png" alt="System Architecture" width="720"/>
</p>

**Pipeline (high level):**

```
PTZ Camera (RTSP + VISCA)
        │
        ▼
 Backend  ──►  MySQL
   ├─ Face recognition
   ├─ YOLOv8 behaviour model
   └─ Attendance & report APIs
        │
        ▼
 Web Frontend (lecturer dashboard)
```

---

## Screenshots

| Login | Home |
|:-----:|:----:|
| <img src="captureScreen/ui_loginpage.png" width="360"/> | <img src="captureScreen/ui_homepage.png" width="360"/> |

| Take Attendance | Add Seat Points |
|:---------------:|:---------------:|
| <img src="captureScreen/ui_takeattendancepage.png" width="360"/> | <img src="captureScreen/ui_addingseatpointpage.png" width="360"/> |

| Behaviour Report | Past Records |
|:----------------:|:------------:|
| <img src="captureScreen/ui_generatingbehaviorreportpage.png" width="360"/> | <img src="captureScreen/ui_viewingpastrecordpage_upperpart.png" width="360"/> |

**Design diagrams**

| Use Case | Activity |
|:--------:|:--------:|
| <img src="captureScreen/useCaseDiagram.png" width="360"/> | <img src="captureScreen/activityDiagram.png" width="360"/> |

**Behaviour model results**

<p align="center">
  <img src="captureScreen/behaviordetectionresult_confusionmatrix.png" alt="Confusion Matrix" width="420"/>
  &nbsp;
  <img src="captureScreen/behaviordetectionresult_numberofimg.png" alt="Dataset size" width="280"/>
</p>

---

## Tech Stack

| Layer | Technology |
|------|------------|
| Frontend | HTML5, CSS3, JavaScript |
| Backend | Python, Flask |
| Database | MySQL |
| Computer Vision | OpenCV, face recognition (dlib), Ultralytics YOLOv8 |
| Camera | PTZ camera over RTSP + VISCA |
| Dataset tooling | Roboflow (labelling), Ultralytics (training) |

---

## Camera Installation (lab reference)

Validated setup for a laboratory room (adjust per classroom size):

| Parameter | Reference value |
|-----------|-----------------|
| Camera height | **180 cm** |
| Distance to farthest desk | **400 cm** |
| Distance to nearest desk | **~40 cm** |
| Wide-angle coverage (farthest row) | **~450 cm** |

<p align="center">
  <img src="captureScreen/camerainstallation.png" alt="Camera installation" width="640"/>
</p>

<p align="center">
  <img src="captureScreen/camerainstallation2.png" width="320"/>
  &nbsp;
  <img src="captureScreen/camerainstallation3.png" width="320"/>
</p>

> Note: SH-HK10U max useful zoom is about **500 cm**. Larger theatres may need a camera with stronger optical zoom.

---

## Behaviour Classes (YOLOv8)

The classroom behaviour model was trained on **7,702** labelled images (≈6,117 train / 1,553 val) with classes such as:

- Head down writing / reading  
- Head up listening  
- Turning head  
- Raising hand  
- Standing  
- Group discussion  
- Teacher guidance  

Reported validation performance (project evaluation): roughly **mAP@0.5 ≈ 0.91**, precision ≈ 0.89, recall ≈ 0.84.

Attendance uses **face recognition** rather than per-course YOLO face models, reducing lag from ~5–6 s to ~0.3–0.4 s per match in evaluation.

---

## Typical Workflow

1. **Login** as lecturer  
2. On **Home**, open an active course session  
3. **Add Seat Location** — aim the PTZ camera, align the face frame, save seat points  
4. **Start Recording** / **Start Attendance** — camera scans seats and marks attendance  
5. **End Class** — stop recording and open the behaviour report page  
6. **Start Analysis** — run YOLOv8 on the recorded video and save the report  
7. Review later under **Past Courses** (export supported)

---

## Documentation

| File / folder | Content |
|---------------|---------|
| `information.txt` | Capstone **final report** (full text) |
| `visca.txt` | VISCA protocol command reference |
| `captureScreen/` | UI screenshots, UML diagrams, camera install photos, model result charts |

---

## Privacy & Ethics

This system processes **facial biometric data** and classroom video. Any real deployment should obtain informed consent and follow institutional / legal privacy rules. This repository is an **academic / portfolio demonstration** and does not include private enrollment photos or classroom recordings.

---

## Acknowledgements

- Capstone supervisors and examiners at PolyU Department of Computing  
- Ultralytics YOLOv8, Roboflow, OpenCV, and the face-recognition / dlib community  
- Technical support for lab booking and camera networking during user testing  

---

## License

All rights reserved unless otherwise stated. Please credit the author if you reference this work for teaching or research.
