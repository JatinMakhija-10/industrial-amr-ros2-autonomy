# Weekly Engineering Log — AMR Design & Development

> Six-week engineering sprint across mechanical CAD, software simulation, and system integration.
> Facility: IIT Delhi — Design & Manufacturing Laboratory (DML)

---

## Week 1 — Lab Orientation & Feasibility Assessment

**Focus**: Domain knowledge gathering, fabrication constraints, and initial scoping.

### Activities
- **Electrical Department Walkthrough**: Reviewed existing wiring conventions, wire gauge standards, and terminal block configurations used in prior lab projects. Benchmarked against AMR power requirements.
- **Mechanical Lab Tour**: Identified available fabrication processes — laser cutting (available in-house), CNC milling (external vendor required), and manual sheet metal bending (in-house press brake).
- **DML Parts Audit**: Catalogued in-house components (motors, casters, standoffs) against initial BOM. Identified gaps requiring procurement.
- **Drive Architecture Decision**: Evaluated mecanum wheel, omnidirectional, and differential drive options. Selected differential drive for minimal mechanical complexity, well-understood kinematics, and direct compatibility with `diff_drive_controller` in ROS 2.

### Key Outcomes
- Laser cutting feasible in-house for all aluminium panels (3 mm and 2 mm sheet).
- CNC milling required for LiDAR mount plate (external vendor, Okhla industrial area).
- Three candidate hub motor specifications shortlisted for sizing analysis.

---

## Week 2 — Vendor Research & CAD Foundations

**Focus**: Material procurement and chassis geometry definition.

### Activities
- **Vendor Visits**: Canvassed aluminium suppliers in Chandni Chowk and Pahadganj. Comparative pricing:
  - Chandni Chowk: ₹400/kg (6061-T6)
  - Pahadganj Supplier A: ₹415/kg (5052-H32)
  - Pahadganj Supplier B: ₹465/kg (mixed alloy, premium finish)
- **CAD Start — Base Plate**: Established Fusion 360 assembly structure. Defined master coordinate origin at rear-lower-left chassis corner. Base plate outer envelope sketched at 318 × 347.5 mm; held at basic geometry pending motor and caster position confirmation.
- **Hub Motor Study**: Confirmed wheel outer diameter at 100.7 mm. Differential drive layout fixed with two driven wheels on the lateral axis (track width: 280 mm) and two passive casters longitudinally.
- **Component Footprints**: Raspberry Pi 5, RPLiDAR A1M8, battery pack, and camera footprints placed on base sketch to verify spatial feasibility.

### Key Outcomes
- Material selection finalized: 6061-T6 for structural members, 5052-H32 for enclosure panels, IS 2062 for motor brackets and battery cradle.
- BOM pricing baseline established at approximately ₹61,000 total.

---

## Week 3 — Full CAD Modelling & Faculty Consultation

**Focus**: Multi-body assembly build-out and component integration.

### Day-by-Day CAD Progress

**Day 1 (4 June)**: Base plate and motor mount concept initiated. Side-panel-integrated motor bracket selected over welded approach for rigidity.

**Day 2 (5 June)**: Top plate and base plate profiles finalized. Chamfered corners added. Manufacturer CAD files imported: RPLiDAR A1M8, BNO055 IMU, Raspberry Pi 5. All components positioned and constrained in assembly.

**Day 3 (6 June)**: Side panels modelled with 100.7 mm wheel cutouts (5 mm radial clearance). Caster brackets designed and integrated. Front camera bracket positioned at downward angle for floor obstacle detection.

**Day 4 (7 June)**: Hub motor wheels mated to chassis; left and right wheel joints defined as continuous rotation joints. All primary components present: chassis, wheels, motor mounts, top plate, LiDAR, Raspberry Pi 5, camera, IMU.

**Day 5 (8 June — CAD Finalization)**: Full assembly interference detection run. Two minor clashes found and resolved. Final assembly height confirmed: **271.0 mm** from side view measurement in Fusion 360.

### Critical Design Issue — Caster Clearance Conflict
During full assembly review (Day 2–3), a geometric interference was identified between the caster swivel envelope and the inner fold edge of the side panel. The individual component models appeared clear — the conflict only emerged in the constrained multi-body assembly context.

**Root Cause**: Initial caster mounting bracket offset did not account for the bend radius of the 2 mm 5052-H32 panel (R = 3 mm), which reduced the effective clearance zone.

**Resolution**: Bracket standoff offset increased by 14.5 mm; 30° relief chamfers added at the bracket-to-base interface. Swivel clearance re-verified through full 360° rotation envelope before finalising the drawing.

### Faculty Consultation
Consulted with faculty advisors regarding sensor selection. RPLiDAR A1M8 confirmed as primary navigation sensor over costlier S2E variant; BNO055 IMU confirmed for attitude estimation. Delivery timelines for both components obtained.

---

## Week 4 — Engineering Drawings & Electrical Architecture

**Focus**: Production drawing generation and power system design.

### Drawing Package
- **16 two-sheet drawing sets** produced covering all structural components.
- Drawing standard: ASME Y14.5 GD&T.
- Key learning: Fabrication drawings require explicit tolerance callouts and manufacturing notes that academic drawings typically omit. Laser kerf offsets (0.20 mm for 3 mm aluminium) and CNC toolpath offsets must be specified on the drawing, not assumed by the vendor.
- Material nesting exercise: Arranged 6 parts on a single 1200 × 2400 mm sheet, reducing per-part material cost by approximately 22% versus individual sheet orders.

### Electrical Architecture (High Level)
- 24V Li-ion battery pack feeds through a protection chain: fuse → EMI filter → reverse-polarity MOSFET → TVS diode → emergency stop → safety relay.
- E-stop configured to cut motor power while keeping the SBC compute rail live — allows continuous telemetry and position logging through emergency events.
- RS-485 selected for hub motor controller communication (noise immunity over longer cable runs versus I2C/SPI). Bus terminated with 120 Ω resistors at both ends.
- Schematic went through two revisions: EMI filter and TVS diode added after reviewing motor controller datasheet protection recommendations.

---

## Week 5 — ROS 2 Workspace & Simulation Bring-Up

**Focus**: Digital twin construction and hardware abstraction validation.

### ROS 2 Workspace Setup
- Workspace created at `~/amr_ws` with package `amr_robot`.
- Package structure: `urdf/`, `meshes/`, `launch/`, `config/`, `worlds/`, `rviz/`.
- All dependencies installed: `ros2_control`, `diff_drive_controller`, `joint_state_broadcaster`, `gazebo_ros2_control`, `slam_toolbox`, `nav2_bringup`.

### URDF Build (Incremental, with RViz verification at each step)
1. Base chassis box body
2. Left and right wheel joints (continuous, velocity interfaces)
3. Front and rear caster spheres (fixed joints, near-zero friction)
4. LiDAR cylinder (fixed joint, top plate mount)
5. IMU box (fixed joint, chassis surface)
6. `ros2_control` hardware interface tags and Gazebo plugin tags

### Key Debugging Issue — Controller Joint Name Mismatch
After spawning in Gazebo and loading controllers, `diff_drive_controller` reported **ACTIVE** state but wheels did not respond to velocity commands.

**Root Cause**: The `left_wheel_names` and `right_wheel_names` arrays in `diff_drive_controller.yaml` used `left_wheel_joint` while the URDF defined the joint as `leftwheel_joint` (no underscore in one variant). ROS 2 control accepts the YAML silently even when joint names do not match URDF definitions — no error log entry is generated.

**Resolution**: Exact string comparison of URDF joint name attributes against YAML controller parameter arrays. Names aligned to `left_wheel_joint` and `right_wheel_joint` throughout.

**Lesson**: In `ros2_control`, joint name mismatches are silent failures. Always verify by echoing `/joint_states` and confirming all expected joint names appear after controller load.

### Simulation Verification
- Robot spawned in empty Gazebo world; all TF frames published correctly.
- Teleop keyboard control verified in all motion directions (forward, backward, left arc, right arc, spot turn).
- LiDAR `/scan` topic confirmed at 10 Hz with 720 samples.
- IMU `/imu` topic confirmed at 100 Hz.

---

## Week 6 — SLAM, Nav2, and Documentation Handoff

**Focus**: Autonomous navigation validation and documentation package completion.

### SLAM Mapping — Warehouse World
Custom Gazebo world built: 20 × 20 m floor, four boundary walls, 12 storage racks in four columns with aisle spacing, three support pillars.

`slam_toolbox` launched in async mode alongside simulation. Drove robot through warehouse aisles using teleop.

**Initial Map Quality Issue**: Default scan matching parameters produced stitching artifacts in the aisle regions — identical rack geometries caused the scan matcher to generate false positive loop closures, pulling the map topology incorrectly.

**Parameter Tuning Applied**:
- Reduced `minimum_travel_distance` from 0.5 m to 0.15 m — triggers scan updates at tighter spatial intervals during cornering.
- Reduced `minimum_travel_heading` from 0.5 rad to 0.174 rad — eliminates rotational lag.
- Tightened `link_match_minimum_response_fine` from 0.6 to 0.8 — prevents false-positive cross-aisle loop closures.
- Reduced `loop_search_maximum_distance` from 8.0 m to 4.0 m — restricts hypothesis search to adjacent aisles only.

**Result**: Clean occupancy grid with correct aisle topology. Map saved as `warehouse_map.pgm` and `warehouse_map.yaml`.

### Nav2 Configuration
- Global planner: NavFn (Dijkstra over static costmap).
- Local controller: Regulated Pure Pursuit — smooth velocity profile through narrow aisles; curvature-proportional deceleration.
- Costmaps: 2-layer (Obstacle + Inflation, radius 0.55 m).
- All Nav2 lifecycle nodes confirmed active: `bt_navigator`, `controller_server`, `planner_server`, `recoveries_server`.

### Deliverables Completed
All 16 deliverables across six work areas produced and documented for project handoff. The AMR design is at a well-defined handoff state: CAD complete, simulation running, and physical build plan clear.
