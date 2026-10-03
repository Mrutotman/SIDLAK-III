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

### Simulation Data Flow
