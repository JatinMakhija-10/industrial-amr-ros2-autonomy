# Deliverables Tracking — AMR Design & Development

> Complete status table of all 16 engineering deliverables produced across the 6-week internship sprint.

---

## Deliverables Summary Table

| # | Area | Deliverable | Description | Status | Target Date |
|---|------|-------------|-------------|--------|-------------|
| 1 | Drawings | Component Drawing Sheets | All chassis parts — 2D engineering sheets with GD&T | ✅ Complete | Jun 26 |
| 2 | Drawings | Full Assembly Drawing | Orthographic views; 271 mm confirmed height | ✅ Complete | Jun 26 |
| 3 | Drawings | Mounting Cross-Check Sheet | LiDAR footprint vs top plate hole pattern | 🔄 In Progress | End of Internship |
| 4 | Assembly SOP | Assembly Sequence Document | Step-by-step chassis build order | ✅ Complete | Jun 26 |
| 5 | CAD | Master Fusion 360 Assembly | Full multi-body parametric assembly | ✅ Complete | Jun 26 |
| 6 | CAD | STEP Export | Universal 3D exchange format for downstream tools | ✅ Complete | Jun 26 |
| 7 | Materials | Bill of Materials (BOM) | Full component list with pricing and sources | ✅ Complete | Jun 20 |
| 8 | Materials | Vendor Pricing Sheet | Three-vendor comparison for aluminium stock | ✅ Complete | Jun 14 |
| 9 | Electrical | 24V Schematic | Full power protection chain and sensor wiring | ✅ Complete | Jun 20 |
| 10 | Electrical | KiCad Project | Editable schematic + BOM CSV + PDF export | ✅ Complete | Jun 20 |
| 11 | Software | ROS 2 Workspace | `amr_robot` package with URDF, launch, config | ✅ Complete | Jun 26 |
| 12 | Software | Gazebo Simulation | Robot in warehouse world; diff drive verified | ✅ Complete | Jun 26 |
| 13 | Software | SLAM Map | `warehouse_map.pgm` and `warehouse_map.yaml` saved | ✅ Complete | Jun 26 |
| 14 | Software | Nav2 Stack | All lifecycle nodes active; RPP controller configured | ✅ Complete | Jun 26 |
| 15 | Software | Autonomous Navigation Test | 2D Goal Pose end-to-end warehouse navigation | 🔄 Pending final test run | Post-Handoff |
| 16 | Documentation | ROS 2 Simulation Report | Full pipeline documentation with screenshots | ✅ Complete | Jun 26 |

---

## Handoff State

The AMR project is at a well-defined, documented handoff point:

**Mechanical (Complete)**
- Full Fusion 360 assembly with confirmed 271 mm height
- 16 two-sheet drawing sets with GD&T callouts
- BOM costed at ~₹61,000 from confirmed Delhi vendors

**Software Simulation (Functional, one item pending)**
- ROS 2 workspace builds and runs
- Differential drive control verified
- SLAM map of warehouse saved
- Nav2 stack fully configured and nodes active
- Pending: end-to-end autonomous goal navigation test run

**Immediate Next Steps for Continuation**
1. Replace placeholder box URDF geometry with actual STL/DAE meshes exported from Fusion 360
2. Complete autonomous navigation test with 2D Goal Pose in warehouse world
3. Confirm physical track width measurement; update `diff_drive_controller.yaml`
4. Deploy ROS 2 on Raspberry Pi 5; connect to hub motor controllers via RS-485
5. Integrate physical LiDAR and IMU; validate sensor data against simulation baseline
6. Estimate battery runtime under combined motor + compute load
