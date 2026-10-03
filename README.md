# SIDLAK III Autonomous System — ROS 2 Workspace

ROS 2 Humble autonomy stack for the **SIDLAK III Autonomous System**, targeting the **NVIDIA Jetson Orin Nano Super Developer Kit** and validated through a **Gazebo Digital Twin**.

The workspace is organized into modular subsystems covering robot description, sensors, perception, localization, planning, supervision, command arbitration, vehicle interface, and simulation.

---

## 📁 Workspace Structure

    sidlak_ws/
    └── src/
        ├── sidlak_msgs/                  # Custom ROS 2 message definitions
        │   ├── msg/
        │   │   ├── LineDetection.msg     # Line detection interface (~15 Hz)
        │   │   └── ObstacleArray.msg     # Obstacle detection interface (~10 Hz)
        │   └── CMakeLists.txt
        │
        ├── sidlak_bringup/               # System-level launch and visualization
        │   ├── launch/
        │   │   └── sidlak_core.launch.py # Robot State Publisher + RViz2
        │   └── rviz/
        │       └── sidlak_default.rviz
        │
        ├── sidlak_description/           # Robot model and TF configuration
        │   ├── urdf/
        │   │   └── sidlak.urdf
        │   └── meshes/
        │
        ├── sidlak_sensors/               # Sensor drivers
        │   └── src/
        │       ├── unilidar_node.cpp      # Unitree L2 LiDAR driver
        │       └── camera_node.cpp        # Intel depth camera driver
        │
        ├── sidlak_perception/            # Perception pipeline
        │   └── src/
        │       ├── pointcloud_to_laserscan.cpp
        │       ├── line_detection_node.cpp
        │       └── obstacle_detector_node.cpp
        │
        ├── sidlak_localization/           # Localization and state estimation
        │   └── src/
        │       ├── point_lio_node.cpp
        │       └── ekf_node.cpp
        │
        ├── sidlak_planning/               # Path planning and control
        │   └── src/
        │       ├── planner_server.cpp
        │       └── controller_server.cpp
        │
        ├── sidlak_supervision/            # Mission logic and safety supervision
        │   └── src/
        │       └── mission_fsm_node.cpp
        │
        ├── sidlak_arbitration/            # Command arbitration and vehicle interface
        │   └── src/
        │       ├── arbitrator_node.cpp    # Safety gatekeeper and speed limiting
        │       └── sidlak_can_bridge.cpp  # SocketCAN ↔ ESP32 VCS @ 20 Hz
        │
        └── sidlak_simulation/             # Gazebo Digital Twin
            ├── launch/
            │   └── gazebo_sim.launch.py
            └── worlds/
                └── silesia_ring_basic.world

---

## 🧩 System Architecture

The SIDLAK III autonomy stack is divided into functional layers:

    ┌─────────────────────────────────────────────────────┐
    │                 Mission Supervision                 │
    │              Mission FSM / Safety Logic             │
    └───────────────────────┬─────────────────────────────┘
                            │
    ┌───────────────────────▼─────────────────────────────┐
    │                    Planning                          │
    │             Nav2 / Smac Hybrid-A* / Control         │
    └───────────────────────┬─────────────────────────────┘
                            │
    ┌───────────────────────▼─────────────────────────────┐
    │                   Localization                       │
    │                 Point-LIO / EKF Fusion              │
    └───────────────────────┬─────────────────────────────┘
                            │
    ┌───────────────────────▼─────────────────────────────┐
    │                    Perception                        │
    │      LiDAR Processing / YOLO / OpenCV / Obstacles   │
    └───────────────────────┬─────────────────────────────┘
                            │
    ┌───────────────────────▼─────────────────────────────┐
    │                     Sensors                          │
    │              Unitree L2 / Intel Depth Camera        │
    └───────────────────────┬─────────────────────────────┘
                            │
    ┌───────────────────────▼─────────────────────────────┐
    │              Vehicle Interface                       │
    │              Arbitration / SocketCAN                │
    └─────────────────────────────────────────────────────┘

### Simulation Data Flow

    Gazebo Digital Twin
            │
            ▼
    Sensor Simulation
            │
            ▼
    Perception ──────► Localization
            │                │
            └───────┬────────┘
                    ▼
                Planning
                    │
                    ▼
               Arbitration
                    │
                    ▼
            Simulated Vehicle

---

# 🛠️ Developer Quick Reference

## 1. Build the Workspace

Navigate to the workspace root:

    cd ~/sidlak_ws

### Build All Packages

    colcon build --symlink-install

### Build a Specific Package

    colcon build \
      --packages-select sidlak_bringup \
      --symlink-install

Replace `sidlak_bringup` with the package being modified.

### Source the Workspace

After building:

    source install/setup.bash

> **Important:** Every new terminal running ROS 2 commands must source the ROS 2 and SIDLAK workspaces.

For convenience, add the following to `~/.bashrc`:

    source /opt/ros/humble/setup.bash
    source ~/sidlak_ws/install/setup.bash

---

# 🚀 2. Step 0 — Robot Description & RViz2 Verification

The first verification step is to confirm that the robot description and TF tree load correctly.

Launch the core system:

    ros2 launch sidlak_bringup sidlak_core.launch.py

The core launch file is expected to start:

- Robot State Publisher
- RViz2
- SIDLAK III robot description
- Configured TF relationships

### Expected Result

RViz2 should open with the SIDLAK III model visible.

Verify that the major frames are connected correctly:

    base_link
    ├── lidar_link
    ├── camera_link
    └── ...

---

# 🔍 3. ROS 2 Graph Inspection

### List Active Nodes

    ros2 node list

### List Active Topics

    ros2 topic list

### Inspect a Topic

    ros2 topic info /unilidar/cloud

### Inspect Topic Data

    ros2 topic echo /unilidar/cloud

> For high-volume topics such as point clouds, avoid leaving `topic echo` running unnecessarily.

---

# 📡 4. Verify Topic Frequencies

Check the LiDAR point-cloud publishing rate:

    ros2 topic hz /unilidar/cloud

Check the custom perception interfaces:

    ros2 topic hz /line_detection
    ros2 topic hz /obstacles

> Update these topic names if the actual node configuration uses different names or namespaces.

### Target Interface Rates

| Interface | Target Rate |
|---|---:|
| Line Detection | ~15 Hz |
| Obstacle Detection | ~10 Hz |
| CAN Vehicle Interface | ~20 Hz |

---

# 🧭 5. Verify TF Connectivity

Check the transform between the robot base and LiDAR:

    ros2 run tf2_ros tf2_echo base_link lidar_link

Check the camera transform:

    ros2 run tf2_ros tf2_echo base_link camera_link

A valid transform should continuously produce translation and rotation data.

### Generate the TF Tree

    ros2 run tf2_tools view_frames

This can be used to identify:

- Missing transforms
- Disconnected frames
- Unexpected parent-child relationships
- Duplicate frame publishers

---

# 🧪 6. Recommended Verification Sequence

Bring up and verify the system incrementally:

    1. Workspace builds
           ↓
    2. Robot description loads
           ↓
    3. TF tree is connected
           ↓
    4. Sensors publish data
           ↓
    5. Perception produces detections
           ↓
    6. Localization produces pose
           ↓
    7. Planner produces paths
           ↓
    8. Controller produces commands
           ↓
    9. Arbitration validates commands
           ↓
    10. CAN bridge communicates with VCS

This approach isolates failures before higher-level autonomy is enabled.

---

# 🛡️ Safety-Critical Components

## `sidlak_supervision`

Responsible for:

- Mission state management
- Mission transitions
- Safety conditions
- Emergency and supervisory overrides

## `sidlak_arbitration`

Acts as the final command gatekeeper before commands reach the vehicle interface.

Responsibilities include:

- Command validation
- Speed limiting
- Safety overrides
- Command arbitration
- Vehicle-interface communication

### Command Flow

    Planner / Controller
            │
            ▼
    Mission / Safety Supervisor
            │
            ▼
    Command Arbitrator
            │
            ├── Safety checks
            ├── Speed limits
            └── Override logic
            │
            ▼
    SocketCAN Bridge
            │
            ▼
        ESP32 VCS

---

# 🖥️ Gazebo Digital Twin

Launch the simulation environment:

    ros2 launch sidlak_simulation gazebo_sim.launch.py

Simulation world:

    silesia_ring_basic.world

The Digital Twin should be used to validate the autonomy pipeline before deployment to the physical vehicle.

### Simulation Validation Flow

    Gazebo
      ↓
    Sensor Topics
      ↓
    TF
      ↓
    Perception
      ↓
    Localization
      ↓
    Planning
      ↓
    Control
      ↓
    Arbitration

---

# 📦 Package Responsibilities

| Package | Responsibility |
|---|---|
| `sidlak_msgs` | Custom ROS 2 interfaces |
| `sidlak_bringup` | System launch and visualization |
| `sidlak_description` | URDF, meshes, and TF structure |
| `sidlak_sensors` | LiDAR and camera drivers |
| `sidlak_perception` | Object, obstacle, and line perception |
| `sidlak_localization` | Point-LIO and EKF state estimation |
| `sidlak_planning` | Path planning and vehicle control |
| `sidlak_supervision` | Mission FSM and safety supervision |
| `sidlak_arbitration` | Command validation and vehicle interface |
| `sidlak_simulation` | Gazebo Digital Twin |

---

# 🧰 Diagnostic Commands

## Check ROS 2 Distribution

    echo $ROS_DISTRO

Expected:

    humble

## Check ROS Environment

    printenv | grep ROS

## Inspect a Node

    ros2 node info /<node_name>

## Inspect Custom Message Definitions

    ros2 interface show sidlak_msgs/msg/LineDetection

    ros2 interface show sidlak_msgs/msg/ObstacleArray

## List SIDLAK Packages

    ros2 pkg list | grep sidlak

## Inspect Launch Arguments

    ros2 launch sidlak_bringup sidlak_core.launch.py --show-args

---

# 🧹 Clean Rebuild

If the workspace enters an inconsistent build state, perform a clean rebuild:

    cd ~/sidlak_ws

    rm -rf build/ install/ log/

    colcon build --symlink-install

    source install/setup.bash

> **Note:** A clean rebuild should be used when necessary rather than as the default development workflow.

---

# ⚙️ Development Workflow

For normal development:

    cd ~/sidlak_ws

    # Modify source/configuration
    # ...

    colcon build --symlink-install

    source install/setup.bash

    ros2 launch sidlak_bringup sidlak_core.launch.py

For package-specific development:

    colcon build \
      --packages-select <package_name> \
      --symlink-install

    source install/setup.bash

Example:

    colcon build \
      --packages-select sidlak_perception \
      --symlink-install

    source install/setup.bash

---

# ✅ Step 0 Acceptance Checklist

Before proceeding to sensor and autonomy integration, verify:

- [ ] Workspace builds without errors
- [ ] `sidlak_bringup` launches successfully
- [ ] Robot model appears in RViz2
- [ ] `base_link` exists
- [ ] Sensor frames exist
- [ ] TF tree is connected
- [ ] No unexpected TF warnings appear
- [ ] Required ROS 2 packages are discoverable
- [ ] Gazebo Digital Twin launches successfully
- [ ] Sensor topics appear in simulation
- [ ] Expected topic frequencies are verified

Once these checks pass, proceed to sensor-driver and perception validation.

---

# 🎯 System Bring-Up Philosophy

SIDLAK III should be validated **incrementally from the lowest-level interfaces upward**.

    Hardware / Simulation
            ↓
    Sensor Drivers
            ↓
    TF / Robot Description
            ↓
    Perception
            ↓
    Localization
            ↓
    Planning
            ↓
    Supervision
            ↓
    Arbitration
            ↓
    Vehicle Interface

Each layer should be independently observable through ROS 2 topics, services, transforms, diagnostics, and visualization before being relied upon as an input to the next layer.

---

# 📚 Quick Command Reference

| Task | Command |
|---|---|
| Build workspace | `colcon build --symlink-install` |
| Build package | `colcon build --packages-select <package> --symlink-install` |
| Source workspace | `source install/setup.bash` |
| Launch core | `ros2 launch sidlak_bringup sidlak_core.launch.py` |
| Launch simulation | `ros2 launch sidlak_simulation gazebo_sim.launch.py` |
| List nodes | `ros2 node list` |
| List topics | `ros2 topic list` |
| Topic info | `ros2 topic info <topic>` |
| Echo topic | `ros2 topic echo <topic>` |
| Check frequency | `ros2 topic hz <topic>` |
| Inspect node | `ros2 node info <node>` |
| Inspect TF | `ros2 run tf2_ros tf2_echo <parent> <child>` |
| Generate TF tree | `ros2 run tf2_tools view_frames` |
| List SIDLAK packages | `ros2 pkg list \| grep sidlak` |
| Inspect message | `ros2 interface show <package>/msg/<Message>` |

---

# 🏁 SIDLAK III Development Principle

> **Build → Source → Verify → Isolate → Integrate → Validate**

Keep each subsystem independently testable and observable before integrating it into the complete autonomy stack.

---

## Workspace Information

| Component | Specification |
|---|---|
| ROS 2 | Humble |
| Target Platform | NVIDIA Jetson Orin Nano Super Developer Kit |
| Simulation | Gazebo Digital Twin |
| Vehicle Controller | ESP32 VCS |
| Vehicle Interface | SocketCAN |
| LiDAR | Unitree L2 |
| Depth Camera | Intel |
| Localization | Point-LIO + EKF |
| Planning | Nav2 / Smac Hybrid-A* |
| Perception | YOLO / OpenCV / LiDAR |
| Visualization | RViz2 |

---

## Repository Status

This repository contains the core ROS 2 workspace for the SIDLAK III Autonomous System.

Development should proceed subsystem-by-subsystem, with each component validated independently in simulation before integration with the physical vehicle.
EOF
