# Troubleshooting Guide & Known Debugging Solutions

> This document captures real debugging issues encountered during development with their root causes and confirmed solutions. Each entry represents actual time lost and recovered — preserved here for whoever continues this project.

---

## ROS 2 / Gazebo Issues

### Issue 1: `diff_drive_controller` Active but Wheels Not Moving

**Symptom**: After spawning the robot and loading controllers, `ros2 control list_controllers` shows `diff_drive_controller` as `active`. Publishing to `/diff_drive_controller/cmd_vel_unstamped` produces no motion. No error in `ros2 topic echo /joint_states`.

**Root Cause**: Joint name mismatch between URDF `<joint name="...">` attributes and the `left_wheel_names` / `right_wheel_names` arrays in `diff_drive_controller.yaml`. ROS 2 control loads silently without throwing errors when names do not match.

**Confirmed Fix**:
```bash
# Check what joint names are actually being published
ros2 topic echo /joint_states --once

# Compare against your YAML
grep -A5 "left_wheel_names" ros2_ws/src/amr_robot/config/diff_drive_controller.yaml
```
Ensure exact string match including underscores, hyphens, and case. In this project, the correct names are `left_wheel_joint` and `right_wheel_joint`.

---

### Issue 2: Gazebo Fails to Launch — GLX / OpenGL Renderer Error

**Symptom**: `gzserver` starts but `gzclient` exits immediately. Terminal shows: `[Err] [RenderEngine.cc] Can't open display: GLX not available`.

**Environment**: Ubuntu 22.04 on a system with NVIDIA GPU but `nouveau` drivers active, or running inside a VM/container without hardware OpenGL passthrough.

**Fix**:
```bash
# Option A: Use software rendering (slower but functional)
export LIBGL_ALWAYS_SOFTWARE=1
ros2 launch amr_robot amr_simulation.launch.py

# Option B: Run headless (server only, no GUI)
export GAZEBO_HEADLESS=1
gzserver ros2_ws/src/amr_robot/worlds/warehouse.world
```

---

### Issue 3: `dpkg` Lock Prevents `apt install`

**Symptom**: `sudo apt install ros-humble-...` fails with: `E: Could not get lock /var/lib/dpkg/lock-frontend`.

**Cause**: `unattended-upgrades` running in background on Ubuntu 22.04 LTS.

**Fix**:
```bash
# Wait for unattended-upgrades to finish
sudo systemctl stop unattended-upgrades
# Then run apt
sudo apt install ros-humble-nav2-bringup
# Re-enable after
sudo systemctl start unattended-upgrades
```

---

### Issue 4: SLAM Map Has Stitching Artifacts in Warehouse Aisles

**Symptom**: Occupancy grid on `/map` shows duplicated or misaligned aisle sections. Rack rows appear offset or doubled when driving through long parallel corridors.

**Root Cause**: Default `slam_toolbox` parameters are tuned for general indoor environments with varied geometry. Uniform rack layouts lack distinctive scan features, causing the scan matcher to generate false-positive loop closures across identical aisle profiles.

**Confirmed Fix** (parameters in `config/slam_toolbox_params.yaml`):
```yaml
minimum_travel_distance: 0.15        # was 0.50 — tighter update interval
minimum_travel_heading: 0.174        # was 0.50 — ~10 degrees
link_match_minimum_response_fine: 0.80  # was 0.60 — stricter match threshold
loop_search_maximum_distance: 4.0    # was 8.0 — restrict search to adjacent aisles
```

Also important: **drive slowly through aisles** when building the initial map. At speeds above ~0.4 m/s, scan overlap between consecutive poses decreases and gaps appear in the occupancy grid.

---

### Issue 5: `spawn_entity.py` Fails — Robot Description Not Published

**Symptom**: `spawn_entity.py` times out waiting for `/robot_description` topic. Robot never appears in Gazebo.

**Root Cause**: `robot_state_publisher` node not started before `spawn_entity.py`, or `xacro` failed silently due to a syntax error in the URDF.

**Diagnosis**:
```bash
# Check if robot_description is being published
ros2 topic echo /robot_description --once

# Test xacro directly
xacro ros2_ws/src/amr_robot/urdf/amr_robot.urdf.xacro
```

**Fix**: Verify xacro parses without errors. In the launch file, `spawn_entity` is registered as an `OnProcessExit` event handler after `robot_state_publisher` completes startup — confirm this ordering is correct.

---

## CAD / Mechanical Issues

### Issue 6: Caster Swivel Interference in Full Assembly

**Symptom**: Individual component models show no interference, but full assembly clash detection reports a collision between caster swivel body and side panel inner fold.

**Root Cause**: Initial caster mounting bracket was designed without accounting for the 3 mm bend radius of the 2 mm 5052-H32 side panel. The effective inner face of the panel is displaced inward by the bend allowance, reducing clearance.

**Fix**: Increased lateral standoff offset on caster bracket by 14.5 mm and added 30° relief chamfers at the bracket-to-base interface. Re-ran full 360° swivel envelope check in Fusion 360 Interference Detection.

**Lesson**: Always run Fusion 360 `Inspect → Interference` on the **fully constrained** assembly, not on sub-assemblies. Clearance conflicts often only emerge in the complete system context.

---

## Physical Hardware Notes (For Future Deployment)

### RS-485 Bus Termination
Both ends of the RS-485 motor controller bus **must** have 120 Ω termination resistors installed between D+ and D−. Without termination, signal reflections at the line ends cause intermittent communication errors that are difficult to reproduce and diagnose — the bus will appear functional under low-speed testing but fail under rapid command sequences.

### Track Width Calibration
The URDF currently uses `wheel_separation: 0.280` m. When deploying on physical hardware, measure the actual distance between the contact patches of the left and right wheels and update this value. An incorrect track width does not cause crashes — it causes the robot to arc when commanded to drive straight, and Nav2 localization drift increases as a result.

### IMU Bias Compensation
On first power-up, allow the BNO055 IMU to complete its auto-calibration cycle (all four calibration status values reach 3/3) before commanding any motion. Run:
```bash
ros2 topic echo /imu | grep -A2 "orientation_covariance"
```
Covariance values drop significantly when calibration completes.
