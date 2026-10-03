# SIDLAK III ROS 2 Workspace Structure

This document outlines the directory tree for the `sidlak_ws` ROS 2 Humble workspace. The structure maps directly to the functional subsystems defined in the system architecture, including the Gazebo Digital Twin and the removal of the RL components.

```text
sidlak_ws/
├── build/                        # Compiled binaries (auto-generated)
├── install/                      # Installation files and setup scripts (auto-generated)
├── log/                          # Build and execution logs (auto-generated)
└── src/                          # Source code directory
    │
    ├── sidlak_msgs/              # 1. FOUNDATION: Custom Interface Contract
    │   ├── msg/
    │   │   ├── LineDetection.msg # 15 Hz line tracking interface
    │   │   └── ObstacleArray.msg # 10 Hz obstacle interface
    │   ├── CMakeLists.txt
    │   └── package.xml
    │
    ├── sidlak_bringup/           # 2. FOUNDATION: Launch and Visualization
    │   ├── launch/
    │   │   └── sidlak_core.launch.py
    │   ├── rviz/
    │   │   └── sidlak_default.rviz
    │   ├── CMakeLists.txt
    │   └── package.xml
    │
    ├── sidlak_description/       # 3. FOUNDATION: URDF and TF Frames
    │   ├── urdf/
    │   │   └── sidlak.urdf       # Single source of truth for physical dimensions & sensors
    │   ├── CMakeLists.txt
    │   └── package.xml
    │
    ├── sidlak_sensors/           # 4. SUBSYSTEM: Sensor Drivers
    │   ├── src/
    │   │   ├── unilidar_node.cpp # Unitree L2 LiDAR and IMU driver
    │   │   └── camera_node.cpp   # Intel depth sensing camera driver
    │   ├── CMakeLists.txt
    │   └── package.xml
    │
    ├── sidlak_perception/        # 5. SUBSYSTEM: Vision and LiDAR Processing
    │   ├── src/
    │   │   ├── pointcloud_to_laserscan.cpp
    │   │   ├── line_detection_node.cpp
    │   │   └── obstacle_detector_node.cpp
    │   ├── CMakeLists.txt
    │   └── package.xml
    │
    ├── sidlak_localization/      # 6. SUBSYSTEM: Point-LIO and EKF Fusion
    │   ├── src/
    │   │   ├── point_lio_node.cpp
    │   │   └── ekf_node.cpp
    │   ├── CMakeLists.txt
    │   └── package.xml
    │
    ├── sidlak_planning/          # 7. SUBSYSTEM: Motion Planning
    │   ├── src/
    │   │   ├── planner_server.cpp
    │   │   └── controller_server.cpp
    │   ├── CMakeLists.txt
    │   └── package.xml
    │
    ├── sidlak_supervision/       # 8. SUBSYSTEM: Mission State Management
    │   ├── src/
    │   │   └── mission_fsm_node.cpp
    │   ├── CMakeLists.txt
    │   └── package.xml
    │
    ├── sidlak_arbitration/       # 9. SUBSYSTEM: Vehicle Interface & Safety Gate
    │   ├── src/
    │   │   ├── arbitrator_node.cpp
    │   │   └── sidlak_can_bridge.cpp
    │   ├── CMakeLists.txt
    │   └── package.xml
    │
    └── sidlak_simulation/        # 10. DIGITAL TWIN: Gazebo Environment
        ├── launch/
        │   └── gazebo_sim.launch.py
        ├── worlds/
        │   └── silesia_ring_basic.world
        ├── src/
        │   └── sim_vehicle_interface.cpp
        ├── CMakeLists.txt
        └── package.xml
```