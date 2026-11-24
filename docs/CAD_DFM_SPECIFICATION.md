# Mechanical Architecture & Design for Manufacturing (DFM) Specification

## 1. System Overview & Design Philosophy

The Autonomous Mobile Robot (AMR) mechanical platform is engineered for high-duty-cycle industrial intralogistics, material handling, and autonomous warehouse transport. The structural framework balances structural rigidity, low tare weight, modular subsystem mounting, and strict Design for Manufacturing (DFM) guidelines for cost-effective sheet metal fabrication and CNC milling.

```
                              +--------------------------+
                              |   RPLiDAR A1M8 (Top)     |
                              +-------------+------------+
                                            |
                              +-------------v------------+
                              |   CNC Top Plate (6061-T6)|
                              +-------------+------------+
                                            |
         +----------------------------------+----------------------------------+
         |                                                                     |
+--------v---------+                                                  +--------v---------+
| Left Panel (5052)|                                                  |Right Panel (5052)|
+--------+---------+                                                  +--------+---------+
         |                                                                     |
         |         +----------------------------------------+                  |
         +-------->| Primary Chassis Base Plate (6061-T6)   |<-----------------+
                   | 318 mm x 347.5 mm x 3 mm               |
                   +-------------------+--------------------+
                                       |
                   +-------------------+--------------------+
                   |                                        |
          +--------v--------+                      +--------v--------+
          | Left Hub Motor  |                      | Right Hub Motor |
          | Dia: 100.7 mm   |                      | Dia: 100.7 mm   |
          +-----------------+                      +-----------------+
                   |                                        |
          +--------v--------+                      +--------v--------+
          |  Front Caster   |                      |   Rear Caster   |
          |  Stability Ball |                      |  Stability Ball |
          +-----------------+                      +-----------------+
```

---

## 2. Core Dimensional & Mass Specifications

| Parameter | Value | Engineering Verification Method |
| :--- | :--- | :--- |
| **Total Assembly Height** | `271.0 mm` | Orthographic CAD side elevation & coordinate origin verification |
| **Chassis Footprint** | `318.0 mm (L) × 347.5 mm (W)` | Laser-cut base plate envelope |
| **Drive Wheel Diameter** | `100.7 mm (Radius: 50.35 mm)` | Precision hub motor tire outer diameter |
| **Track Width (Wheel Separation)**| `280.0 mm` | Symmetrical wheel center-to-center baseline |
| **Ground Clearance** | `25.0 mm` | Caster & hub motor axle mounting offset |
| **Tare Weight (Structure + Payload)**| `~14.5 kg` | Mass properties computation (Fusion 360 material densities) |
| **Rated Payload Capacity** | `35.0 kg` | Finite Element structural yield analysis on 6061-T6 base |
| **Drive Architecture** | Differential Drive (2 Active Hub Motors + 2 Passive Casters) | Non-holonomic unicycle kinematics |

---

## 3. Kinematic Architecture & Structural Stability

### 3.1 4-Point Ground Contact Stability
To achieve maximum tipping stability without redundant suspension complexities, the chassis employs a diamond-configured 4-point ground contact layout:
- **Two Lateral Drive Wheels**: Positioned along the robot's transverse center-of-mass axis ($x = 0$), carrying $\approx 70\%$ of the normal force to maintain traction coefficient ($\mu \ge 1.0$) under rapid acceleration and braking.
- **Dual Passive Spherical Casters**: Mounted longitudinally at front ($x = +120\text{ mm}$) and rear ($x = -120\text{ mm}$).
- **Tipping Stability Threshold**: The longitudinal caster baseline provides a critical tip-over angle $\theta_{\text{tip}} > 28^\circ$, preventing pitch instability during deceleration phases ($a_{\text{max}} = 1.5\text{ m/s}^2$).

### 3.2 Caster Clearance Engineering Iteration
During initial multi-part assembly checks in Autodesk Fusion 360, a critical geometric interference was identified:
- *Defect*: The initial caster bracket mounting flange clashed with the inner fold edge of the 5052-H32 side enclosure panel during maximum swivel sweep.
- *Root Cause*: Caster offset plane did not account for sheet metal bend radii ($R = 3.0\text{ mm}$).
- *Design Resolution*: The caster mounting bracket geometry was revised with an increased lateral standoff offset ($+14.5\text{ mm}$) and integrated 30° relief chamfers, achieving zero-interference through 360° swivel envelope while preserving base plate structural rigidity.

---

## 4. Materials Selection & Engineering Trade-Offs

| Component | Material Grade | Mechanical Properties | Engineering Justification |
| :--- | :--- | :--- | :--- |
| **Chassis Base Plate** | **Aluminium 6061-T6** | Yield: 276 MPa, Ultimate: 310 MPa, Density: $2.70\text{ g/cm}^3$ | High strength-to-weight ratio; resists deflection under heavy battery pack and industrial payloads. |
| **Top LiDAR Plate** | **Aluminium 6061-T6** | Surface Hardness: 95 HB, High Machinability | Precision CNC machined surface to ensure planar coplanarity ($\le 0.05\text{ mm}$) for LiDAR optical axis. |
| **Side & Front Enclosure Panels** | **Aluminium 5052-H32** | Yield: 193 MPa, Elongation: 12% | Exceptional sheet metal formability without cracking on 90° bends; high marine/industrial corrosion resistance. |
| **Motor Mounting Brackets** | **Structural Steel IS 2062** | Yield: 250 MPa, Tensile: 410 MPa | High fatigue threshold to withstand cyclic hub motor torque spikes ($T_{\text{stall}} = 12\text{ N}\cdot\text{m}$) and vibrational loads. |
| **Battery Retention Cradle** | **IS 2062 Cold Rolled Steel** | High Ductility & Damping | Low center-of-gravity retention tray ensuring secure clamping of 24V 120Ah Li-ion pack under dynamic shock loads. |

---

## 5. Design for Manufacturing (DFM) & Tolerances

### 5.1 Laser Cutting & Sheet Metal Bending
- **Kerf Allowance**: Compensated for a laser beam kerf width of $0.20\text{ mm}$ on 3 mm aluminium sheet to maintain hole true-positioning within $\pm 0.08\text{ mm}$.
- **Bend Radii & Relief**: Minimum bend radius maintained at $R = 1.5 \times t$ ($4.5\text{ mm}$) with laser-cut relief notches at corners to eliminate stress concentration tears.
- **Chamfer Strategy**: Outer corners feature 30° chamfers rather than 45° to reclaim $+18\%$ usable internal compartment volume while eliminating hazardous sharp corners in warehouse operating zones.

### 5.2 CNC Machining & Fasteners
- **Hole Patterns**: Standardized on M3, M4, and M5 ISO metric thread forms with stainless steel (A2-70) hardware.
- **Vibration Dampening**: Base-mounted electronics (Single Board Computer and IMU) utilize rubber anti-vibration standoffs (shore 50A) to decouple motor-induced harmonic vibrations.
- **Thermal Management**: Top and side panels feature passive convective ventilation slots positioned away from top particulate entry zones.

---

## 6. Engineering Drawing Package (GD&T)

All components are documented in a formal 16-deliverable, two-sheet manufacturing drawing package conforming to ASME Y14.5 standards:
- **Sheet 1**: Part Bill of Materials, raw stock dimensions, surface finish specifications, and DFM notes.
- **Sheet 2**: First-angle / third-angle projection orthographic views, sectional cuts, critical datum references, and GD&T tolerance callouts.

Detailed drawings are archived in [`cad/drawings/`](../cad/drawings/).
