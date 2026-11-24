# End-to-End System Architecture & Hardware-Software Integration

## 1. High-Level System Architecture

The Autonomous Mobile Robot (AMR) integrates high-performance mechanical packaging, real-time actuation, deterministic sensor fusion, and ROS 2 compute:

```
+-----------------------------------------------------------------------------------------+
|                                     COMPUTE LAYER                                       |
|  Single Board Computer (Raspberry Pi 5 / Industrial Embedded Linux)                      |
|  - ROS 2 Humble Navigation Stack (Nav2 + SLAM Toolbox + ros2_control)                   |
|  - Real-Time Controller Loops & TF2 Spatial Transform Trees                             |
+-------------------+----------------------------+-----------------------------+----------+
                    |                            |                             |
                    | USB 3.0 / Serial           | I2C (3.3V)                  | RS-485 (Differential Bus)
                    |                            |                             | (120Ω Terminated)
+-------------------v---+                +-------v--------+            +-------v----------+
|  RPLiDAR A1M8 Scanner |                |  9-DOF IMU     |            | Dual Hub Motor   |
|  (2D Laser Mapping)   |                |  (Orientation) |            | Controllers      |
+-----------------------+                +----------------+            +-------+----------+
                                                                               |
                                                                       +-------v----------+
                                                                       | 2x BLDC Hub      |
                                                                       | Motors + Encoders|
                                                                       +------------------+
```

---

## 2. Sensor & Actuation Interfacing (High Level)

### 2.1 RS-485 Differential Actuator Bus
- **Noise Immunity**: Industrial differential signaling (RS-485) was selected over I2C/SPI for motor controller communication to prevent EMI corruption across extended chassis wire harnesses.
- **Bus Termination**: Terminated with $120\,\Omega$ parallel resistance at line extremities to suppress high-frequency signal reflections and ring-back.

### 2.2 Core Sensor Interfaces
- **Primary Range Sensing**: RPLiDAR A1M8 over serial-to-USB link, streaming raw laser scans (`/scan`) at $10\text{ Hz}$.
- **Inertial Fusion**: 9-DOF IMU communicating via high-speed I2C at $400\text{ kHz}$, providing angular rates and linear accelerations (`/imu`) at $100\text{ Hz}$.

### 2.3 Power & Safety Overview (Condensed)
- **Power Rail**: 24V isolated Li-ion battery bus supplying direct power to high-torque hub motor drivers.
- **Compute Power Conversion**: High-efficiency step-down DC-DC buck converter ($24\text{V} \to 5\text{V}/5\text{A}$) providing clean, regulated power to the SBC.
- **Hardware Emergency Stop (E-Stop)**: Physical hardware emergency cutoff circuit immediately disconnects motor actuation while keeping the compute rail and sensor states alive for continuous diagnostic telemetry.

---

## 3. Digital Twin to Physical Hardware Deployment Workflow

```
1. CAD Mechanical Model (Autodesk Fusion 360)
   ├── Multi-body assembly validation
   ├── Center-of-Mass & Inertia Tensor computation
   └── Mass & dimensional extraction
          │
          ▼
2. ROS 2 Robot Description (URDF / Xacro)
   ├── Kinematic links, joints, and continuous limits
   └── Gazebo physics, friction, and inertial definitions
          │
          ▼
3. Digital Twin Simulation (Gazebo Classic + RViz2)
   ├── Closed-loop differential drive verification (diff_drive_controller)
   ├── Asynchronous SLAM mapping & loop closure (slam_toolbox)
   └── Autonomous path planning & trajectory tracking (Nav2)
          │
          ▼
4. Physical Deployment (Embedded Linux Target)
   ├── Real-time serial/RS-485 actuator node bridging
   ├── Physical sensor calibration (LiDAR datum & IMU bias compensation)
   └── Field waypoint navigation in warehouse facility
```
