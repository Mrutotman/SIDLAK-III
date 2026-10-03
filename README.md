# SIDLAK III Autonomous System - ROS 2 Workspace

This workspace hosts the complete ROS 2 Humble autonomy stack for the SIDLAK III Autonomous System, designed for the NVIDIA Jetson Orin Nano Super Developer Kit and tested via a Gazebo Digital Twin architecture.

---

## 📁 Workspace Folder Structure

```text
sidlak_ws/src/
├── sidlak_msgs/                  # Custom message definitions (LineDetection, ObstacleArray)
│   ├── msg/
│   │   ├── LineDetection.msg     # 15 Hz interface contract
│   │   └── ObstacleArray.msg     # 10 Hz interface contract
│   └── CMakeLists.txt
├── sidlak_bringup/               # Central launch skeleton and configurations
│   ├── launch/
│   │   └── sidlak_core.launch.py # Automatically starts Robot State Publisher and RViz2
│   └── rviz/
│       └── sidlak_default.rviz
├── sidlak_description/           # URDF and TF frames (Static sensor offsets)
│   ├── urdf/
│   │   └── sidlak.urdf
│   └── meshes/
├── sidlak_sensors/               # Sensor Drivers subsystem (Unitree L2 LiDAR, Intel Depth Camera)
│   └── src/
│       ├── unilidar_node.cpp
│       └── camera_node.cpp
├── sidlak_perception/            # Perception subsystem (Pointcloud to laserscan, YOLO/OpenCV)
│   └── src/
│       ├── pointcloud_to_laserscan.cpp
│       ├── line_detection_node.cpp
│       └── obstacle_detector_node.cpp
├── sidlak_localization/          # Localization subsystem (Point-LIO & EKF fusion)
│   └── src/
│       ├── point_lio_node.cpp
│       └── ekf_node.cpp
├── sidlak_planning/              # Planning subsystem (Nav2 Costmap, Smac Hybrid-A*)
│   └── src/
│       ├── planner_server.cpp
│       └── controller_server.cpp
├── sidlak_supervision/           # Supervision subsystem (Mission FSM & safety overrides)
│   └── src/
│       └── mission_fsm_node.cpp
├── sidlak_arbitration/           # Arbitration and vehicle interface
│   └── src/
│       ├── arbitrator_node.cpp   # Command gatekeeper (speed limits, safety override)
│       └── sidlak_can_bridge.cpp # SocketCAN interface to ESP32 VCS at 20 Hz
└── sidlak_simulation/            # Gazebo Digital Twin environment
    ├── launch/
    │   └── gazebo_sim.launch.py
    └── worlds/
        └── silesia_ring_basic.world
🛠️ Essential Developer Cheat Sheet & Commands
1. Workspace Rebuilding & Testing
Use these commands inside your workspace directory (~/sidlak_ws) whenever you modify code, launch files, or descriptions. --symlink-install is enabled so Python launch files and scripts update instantly without full recompilation.

Bash
# Navigate to workspace
cd ~/sidlak_ws

# Rebuild all packages
colcon build --symlink-install

# Rebuild a single specific package (e.g., bringup)
colcon build --packages-select sidlak_bringup --symlink-install

# Source the environment (Required in every new terminal tab)
source install/setup.bash
2. Running Step 0 (Foundations & RViz2 Verification)
Launch the robot state publisher and automatically open RViz2 to verify the TF tree:

Bash
ros2 launch sidlak_bringup sidlak_core.launch.py
3. Debugging & Verification Tools
Bash
# Check active nodes in the ROS 2 graph
ros2 node list

# Check active topics and publishing frequencies
ros2 topic list
ros2 topic hz /unilidar/cloud

# Echo specific transform data to verify URDF link connections
ros2 run tf2_ros tf2_echo base_link lidar_link
