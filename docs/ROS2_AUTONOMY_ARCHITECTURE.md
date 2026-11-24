# Autonomous Navigation & ROS 2 Architecture Specification

## 1. Autonomy Stack Architecture

The software architecture is engineered on **ROS 2 Humble Hawksbill** running in Ubuntu 22.04 LTS, with end-to-end simulation validation in **Gazebo Classic (11.10.2)** and state visualization via **RViz2**. The software pipeline decouples hardware abstraction (`ros2_control`), state estimation, simultaneous localization and mapping (SLAM), and path execution (Nav2).

```
                      +------------------------------------------+
                      |         2D Goal Pose / Mission           |
                      +--------------------+---------------------+
                                           |
+------------------------------------------v------------------------------------------+
| Nav2 Navigation Stack                                                                |
|                                                                                      |
|  +-----------------------+     Global Path      +---------------------------------+  |
|  | NavFn Global Planner  | -------------------> | Regulated Pure Pursuit (RPP)    |  |
|  | (Dijkstra / A*)       |                      | Local Controller                |  |
|  +-----------^-----------+                      +----------------+----------------+  |
|              |                                                   |                   |
|  +-----------+-----------+                      +----------------v----------------+  |
|  | Global Costmap        |                      | Local Costmap (Rolling Window)  |  |
|  | (Static + Obstacle    |                      | (Obstacle + 0.55m Inflation)    |  |
|  |  + 0.55m Inflation)   |                      +----------------^----------------+  |
|  +-----------^-----------+                                       |                   |
+--------------|---------------------------------------------------|-------------------+
               |                                                   |
      +--------+--------+                                 +--------+--------+
      |  /map (SLAM)    |                                 |  /scan (LiDAR)  |
      +--------^--------+                                 +--------^--------+
               |                                                   |
+--------------+--------------+                           +--------+--------+
| SLAM Toolbox                |                           | RPLiDAR A1M8    |
| (Async Graph SLAM + Ceres)  |                           | (12m, 360°, 10Hz|
+--------------^--------------+                           +-----------------+
               |
      +--------+--------------------------+
      |                                   |
+-----+-----------------------+  +--------+--------------------+
| Odometry (/odom)            |  | IMU Fusion (/imu)           |
+-----^-----------------------+  +--------^--------------------+
      |                                   |
+-----+-----------------------------------+--------------------+
| ros2_control Hardware Interface Layer                        |
| - DiffDriveController (Closed-loop velocity tracking)        |
| - JointStateBroadcaster (50 Hz encoder state publisher)      |
+-----------------------------^--------------------------------+
                              |
+-----------------------------+--------------------------------+
| Gazebo / Physical Actuation                                  |
| Left Wheel Joint | Right Wheel Joint                         |
+--------------------------------------------------------------+
```

---

## 2. Kinematics & ros2_control Hardware Layer

### 2.1 Differential Drive Kinematics
The platform obeys non-holonomic unicycle kinematics. Forward and inverse velocity transforms relate robot chassis planar twist $(\dot{x}, \dot{y}, \dot{\theta})$ to left and right wheel angular velocities $(\omega_L, \omega_R)$:

$$\begin{bmatrix} v \\ \omega \end{bmatrix} = \begin{bmatrix} \frac{R}{2} & \frac{R}{2} \\ -\frac{R}{L} & \frac{R}{L} \end{bmatrix} \begin{bmatrix} \omega_L \\ \omega_R \end{bmatrix}$$

$$\omega_L = \frac{v - \frac{\omega L}{2}}{R}, \quad \omega_R = \frac{v + \frac{\omega L}{2}}{R}$$

Where:
- Wheel Radius $R = 0.05035\text{ m}$ ($100.7\text{ mm}$ diameter)
- Wheel Separation Baseline (Track Width) $L = 0.280\text{ m}$

### 2.2 Controller Configuration & Debugging Insights
The low-level actuation interface is managed by `ros2_control` via `diff_drive_controller/DiffDriveController` and `joint_state_broadcaster/JointStateBroadcaster` running at $50\text{ Hz}$.

> [!NOTE]
> **Key Integration Insight**: In `ros2_control`, joint name discrepancies between the URDF `<joint>` tags and the YAML controller parameter arrays fail silently without hard crashes (controllers report `ACTIVE` while actuators receive null setpoints). Strict alignment between URDF definitions (`left_wheel_joint`, `right_wheel_joint`) and controller YAML declarations resolved this latency during hardware-in-the-loop setup.

---

## 3. Sensor Suite & TF2 Coordinate Frames

### 3.1 Sensor Pipeline
1. **RPLiDAR A1M8 2D Laser Range Scanner**:
   - Angular Range: $360^\circ$ ($-\pi \text{ to } +\pi\text{ rad}$)
   - Angular Resolution: $0.5^\circ$ (720 samples/revolution)
   - Range Envelope: $0.15\text{ m}$ to $12.0\text{ m}$
   - Scan Rate: $10\text{ Hz}$
   - Noise Model: Gaussian distribution ($\mu = 0.0\text{ m}$, $\sigma = 0.01\text{ m}$)
2. **9-DOF Inertial Measurement Unit (IMU)**:
   - Output Rate: $100\text{ Hz}$
   - Data Output: 3D Angular velocities, linear accelerations, and absolute quaternion orientation on `/imu`.

### 3.2 Coordinate Transform (TF2) Tree
The kinematic tree conforms strictly to REP-105:
$$\text{map} \longrightarrow \text{odom} \longrightarrow \text{base\_footprint} \longrightarrow \text{base\_link} \longrightarrow \{\text{laser\_frame}, \text{imu\_link}, \text{left\_wheel\_link}, \text{right\_wheel\_link}\}$$

---

## 4. 2D Graph SLAM (slam_toolbox)

### 4.1 Graph Optimization & Loop Closure
The mapping engine utilizes `slam_toolbox` in asynchronous mode. The robot constructs a pose graph where nodes represent robot poses $(x, y, \theta)$ and edges denote odometric and scan-matching spatial constraints:

$$E(x) = \sum_{ij} e_{ij}(x_i, x_j)^T \Omega_{ij} e_{ij}(x_i, x_j)$$

Non-linear least squares optimization is resolved via the **Ceres Solver** utilizing `SPARSE_NORMAL_CHOLESKY` factorizations and `LEVENBERG_MARQUARDT` trust-region optimization.

### 4.2 Parameter Optimization for Industrial Aisles
Standard out-of-the-box SLAM configurations suffer from severe scan-stitching distortion in long, repetitive warehouse corridors due to lack of distinct geometric features. The following parameters were tuned:

| Parameter | Default Value | Tuned Production Value | Justification |
| :--- | :--- | :--- | :--- |
| `minimum_travel_distance` | $0.50\text{ m}$ | **$0.15\text{ m}$** | Triggers scan matching at high spatial resolution during tight cornering |
| `minimum_travel_heading` | $0.50\text{ rad}$ | **$0.174\text{ rad}$ ($10^\circ$)** | Eliminates rotational lag during sharp turning phases |
| `link_match_minimum_response_fine`| $0.60$ | **$0.80$** | Prevents false-positive loop closures caused by identical aisle rack rows |
| `loop_search_maximum_distance` | $8.0\text{ m}$ | **$4.0\text{ m}$** | Restricts loop closure hypothesis search space to adjacent aisles |
| `map_update_interval` | $5.0\text{ s}$ | **$2.0\text{ s}$** | High-frequency occupancy grid updates on `/map` |

---

## 5. Nav2 Autonomous Navigation Pipeline

### 5.1 Global Planning: NavFn Planner
Computes globally optimal, collision-free paths from current localization pose $(x_0, y_0)$ to destination $(x_g, y_g)$ using Dijkstra's algorithm across the static costmap representation.

### 5.2 Local Trajectory Tracking: Regulated Pure Pursuit (RPP)
Unlike legacy DWA or TEB planners that can generate erratic high-frequency velocity oscillations in narrow industrial corridors, the **Regulated Pure Pursuit Controller** provides smooth, predictable path tracking:
- **Curvature Velocity Scaling**: Automatically decelerates the robot when approaching tight turns:
  $$v_{\text{regulated}} = v_{\text{max}} \cdot \frac{r_{\text{turn}}}{r_{\text{min}}}$$
- **Proximity-Based Deceleration**: Dynamically scales velocity inversely proportional to obstacle proximity via the local costmap inflation decay.
- **Adaptive Lookahead Distance**: Dynamically scales lookahead horizon between $0.3\text{ m}$ and $0.9\text{ m}$ based on real-time linear speed.

### 5.3 Costmap Configuration & Obstacle Inflation
Both Global and Local Costmaps incorporate two functional layers:
1. **ObstacleLayer**: Real-time laser raytracing ($12\text{ m}$ clearing, $10\text{ m}$ marking) detecting dynamic obstacles, pallets, and personnel.
2. **InflationLayer**: Creates an exponential repulsive cost gradient with an **inflation radius of $0.55\text{ m}$** (covering robot bounding radius $0.25\text{ m}$ plus a safety buffer of $0.30\text{ m}$):
   $$\text{cost} = \exp\left(-\alpha \cdot (d - r_{\text{robot}})\right) \times 254$$
   Guarantees that path planners favor spacious center-aisle trajectories over wall-adjacent routes.

---

## 6. Simulation Environment (Industrial Warehouse World)

A custom Gazebo simulation environment was modeled to simulate real-world logistics facilities:
- **Dimensions**: $20.0\text{ m} \times 20.0\text{ m}$ ($400\text{ m}^2$ operational floor)
- **Racks**: 12 industrial pallet racks arranged in four columns of three, forming narrow transit aisles ($1.8\text{ m}$ clearance)
- **Obstacles**: 3 cylindrical structural support pillars ($\varnothing 0.70\text{ m}$)
- **Occupancy Grid Deliverable**: Processed at $0.05\text{ m/pixel}$ ($400 \times 400$ grid) saved as `warehouse_map.pgm` and `warehouse_map.yaml`.
