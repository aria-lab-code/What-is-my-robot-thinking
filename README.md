## Table of Contents

- [Abstract](#Abstract)
- [System Requirements](#system-requirements)
- [Installation](#installation)
- [Quick Start](#quick-start)
- [Package Structure](#package-structure)
- [Launch Files](#launch-files)
- [Key Components](#key-components)

---

## Abstract

Assistive robots operating under shared autonomy must balance user control with autonomous assistance. Because robot actions depend on internal intent inference that is not directly observable, mismatches between inferred and intended goals can undermine coordination and trust. We investigate how interface-level transparency—feedback modality (visual vs. auditory) and information richness (sparse vs. rich)—shapes interaction in a vision-based shared autonomy system. In a user study (N=25) across two assistive manipulation tasks, we evaluate how these designs influence coordination and trust. Providing feedback significantly improves intent alignment and reduces corrective intervention, indicating that making the inferred goal legible accelerates convergence in shared control. Participants preferred visual over auditory feedback, while preferences for sparse versus rich information depended on task complexity. We also found that revealing the full belief distribution did not consistently improve alignment or trust. Together, these findings indicate that effective transparency enhances coordination primarily through goal legibility, while trust depends on task-appropriate information exposure rather than maximal disclosure. Based on these results, we outline guidelines for designing transparent shared autonomy systems.

---


## System Requirements

### Hardware
- **Robot**: Kinova Gen3 robotic arm (6 or 7 DOF)
- **Gripper**: Robotiq 2F-85 gripper
- **Cameras**: 3x Intel RealSense D435/D455 cameras
  - End-effector camera
  - Environment recording camera
  - Scene camera
- **Controller**: Xbox controller (or compatible joystick)
- **Compute**: Ubuntu Linux machine with NVIDIA GPU

### Software
- Ubuntu 18.04 or 20.04
- ROS Melodic or Noetic
- Python 3.6+
- CUDA-compatible GPU drivers (for YOLOv11)

### Dependencies
- `kortex_driver` - Kinova ROS driver
- `realsense2_camera` - Intel RealSense ROS wrapper
- `cv_bridge` - OpenCV-ROS bridge
- Python packages: `numpy`, `opencv-python`, `ultralytics` (YOLOv11), `torch`

---

## Installation

1. **Create a catkin workspace** (if you don't have one):
   ```bash
   mkdir -p ~/catkin_ws/src
   cd ~/catkin_ws/src
   ```

2. **Clone this repository**:
   ```bash
   git clone https://github.com/Atharv-B/What-is-my-robot-thinking.git
   ```

3. **Install ROS dependencies**:
   ```bash
   cd ~/catkin_ws
   rosdep install --from-paths src --ignore-src -r -y
   ```

4. **Install Python dependencies**:
   ```bash
   pip3 install numpy opencv-python ultralytics torch torchvision
   ```

5. **Build the package**:
   ```bash
   cd ~/catkin_ws
   catkin_make
   source devel/setup.bash
   ```

6. **Download YOLO models**:
   - Place `yolo11n.pt` and `yolocustom.pt` in the package root directory

---

### Robot Connection
Connect the Kinova arm via Ethernet and configure the IP address (default: `192.168.1.10`).

### Controller Setup
Connect Xbox controller via USB. Verify device path (default: `/dev/input/js1`).

---

## Quick Start

### Basic Teleoperation
```bash
roslaunch trust_and_transparency direct_teleop.launch
```

### Run User Study Experiment
1. **Initialize participant session**:
   ```bash
   python src/user_study/user_study.py
   ```
   This generates a unique participant ID and randomizes treatment order.

2. **Launch experiment**:
   ```bash
   roslaunch trust_and_transparency trust_feedback.launch \
       task:=sorting \
       treatment:=C \
       feedback_type:=visual_rich
   ```

### Available Parameters
- `task`: `sorting`, `shelving`, `familiarity`, 
- `treatment`: Experiment condition identifier (e.g., `A`, `B`, `C`)
- `feedback_type`: `visual_rich`, `visual_sparse`, `auditory_rich`, `auditory_sparse`
- `ip_address`: Robot IP (default: `192.168.1.10`)

---

## Package Structure

```
trust_and_transparency/
├── launch/                          # ROS launch files
│   ├── direct_teleop.launch        # Basic teleoperation
│   ├── trust_feedback.launch       # Main experiment launch file
│   ├── familiarity.launch          # Familiarity task
│   └── trust_and_transparency_visual_feedback.launch
├── msg/                             # Custom ROS messages
│   ├── CentroidConfidence.msg      # Object detection with confidence
│   └── CentroidConfidenceArray.msg # Array of detections
├── src/                             # Python source code
│   ├── constants.py                # Task configurations and parameters
│   ├── ActuatorModel.py            # Robot kinematics model
│   ├── ForwardKinematics.py        # FK solver
│   ├── yolov11_for_scene.py        # Scene object detection
│   ├── test_yolov11.py             # Wrist camera detection
│   ├── vosa_for_trust.py           # VOSA autonomy (shelving)
│   ├── vosa_top_down.py            # VOSA autonomy (sorting)
│   ├── cam_to_world.py             # Camera transform publisher
│   ├── scene_bb_from_centroids.py  # Bounding box visualization
│   ├── viz_feedback.py             # Feedback visualizer
│   ├── goal_alignment_logger.py    # Goal alignment tracking
│   ├── record.py                   # Video recording
│   ├── auditory/                   # Auditory feedback modules
│   │   ├── tts.py                  # Text-to-speech
│   │   └── auditory_rich.py        # Rich auditory feedback
│   └── user_study/                 # User study utilities
│       ├── user_study.py           # Participant management
│       ├── joy_logger.py           # Joystick input logging
│       ├── robot_logger.py         # Robot state logging
│       ├── cam_logger.py           # Camera recording
│       ├── audio_logger.py         # Audio feedback logging
│       └── goal_logger.py          # Goal tracking
├── include/                         # C++ headers (if any)
├── CMakeLists.txt                  # Build configuration
├── package.xml                     # ROS package manifest
└── README.md                       # This file
```

---

## Launch Files

### `trust_feedback.launch`
Main launch file for user study experiments. Starts:
- Kinova robot driver
- All three RealSense cameras
- YOLO object detectors (wrist + scene)
- VOSA autonomy system
- Feedback visualization
- Data logging nodes

### `direct_teleop.launch`
Simplified teleoperation without feedback systems. Useful for:
- Testing robot connectivity
- Manual manipulation tasks
- Hardware debugging

### `familiarity.launch`
Specific configuration for familiarity training tasks.

---

## Key Components

### Object Detection
- **YOLOv11**: Custom trained model for task-specific objects
- **Dual detection**: Wrist camera (manipulation) + scene camera (awareness)
- **Confidence scoring**: Each detection includes confidence metric

### VOSA (Vision Only Shared Autonomy)
- Assists operator by suggesting optimal grasp locations
- Computes goal alignment scores
- Adapts to different task types (top-down sorting vs. shelving)

### Feedback Modalities
- **Visual Rich**: All detected objects with bounding boxes and confidence
- **Visual Sparse**: Only highest confidence object highlighted
- **Auditory Rich**: Verbal descriptions of all detected objects
- **Auditory Sparse**: Alerts for highest confidence object only

---

### Running Trials
```bash
roslaunch trust_and_transparency trust_feedback.launch \
    task:=sorting \
    treatment:=A \
    feedback_type:=visual_rich
```

