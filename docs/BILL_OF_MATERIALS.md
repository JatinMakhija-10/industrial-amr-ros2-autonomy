# AMR Component Bill of Materials

> Complete hardware selection with procurement sources, rationale, and cost breakdown.  
> Total approximate BOM cost: **₹61,000** (sourced from robu.in and Delhi local vendors).

---

## Compute & Control

| # | Component | Model Selected | Backup Option | Interface | Justification |
|---|-----------|---------------|---------------|-----------|---------------|
| 1 | Single Board Computer (SBC) | Raspberry Pi 5 — 8 GB RAM | Industrial CM5-based Revolution Pi | USB 3.0, CSI, GPIO, I2C | Native ROS 2 Humble support; sufficient CPU headroom for SLAM + Nav2 simultaneously |

## Perception Sensors

| # | Component | Model Selected | Backup Option | Interface | Key Specs |
|---|-----------|---------------|---------------|-----------|-----------|
| 2 | 2D LiDAR | RPLiDAR A1M8 | RPLIDAR S2E 360° (30m) | USB (Serial) | 360°, 12 m range, 10 Hz, 8000 samples/s |
| 3 | 9-DOF IMU | Adafruit BNO055 Breakout | 7Semi BNO085 Qwiic Breakout | I2C (3.3V) | Absolute orientation, gravity-compensated accel |
| 4 | Stereo Camera | Arducam 1MP×2 Synchronized (OV9281) | Luxonis OAK-D Lite | CSI / USB | Global shutter, synchronized dual stream for depth |

## Actuation

| # | Component | Model Selected | Interface | Key Specs |
|---|-----------|---------------|-----------|-----------|
| 5 | Hub Motors | Direct-Drive Brushless Hub (×2) | RS-485 | Wheel OD: 100.7 mm, stall torque: 12 N·m |
| 6 | Motor Controller | Dedicated FOC controller per motor | RS-485 bus | Closed-loop velocity + position control |

## Power System

| # | Component | Model Selected | Backup Option | Specs |
|---|-----------|---------------|---------------|-------|
| 7 | Main Battery | 24V 120Ah Li-ion Smart Battery | 24V 100Ah Grade-A Li-ion | Integrated BMS, RS-485 state reporting |
| 8 | Buck Converter | XL4015 + LM2596S combo | DC-DC 7–24V→5V 4A module | 24V→5V @ 5A for SBC compute rail |

## Communication & Interfacing

| # | Component | Model | Justification |
|---|-----------|-------|---------------|
| 9 | RS-485 Adapter | Waveshare Industrial USB-to-RS-485 | 120Ω termination, industrial noise immunity, motor controller compatibility |
| 10 | Emergency Stop | DPDT 6-Pin 16mm Red Button | Hardware motor cutoff while keeping compute rail live |

---

## Procurement & Pricing Notes

- **Local Delhi Vendors**: Aluminium sheet stock sourced from Chandni Chowk (₹400/kg) and Pahadganj (₹415–465/kg depending on 6061-T6 vs 5052-H32 alloy grade).
- **Online Procurement**: Electronics sourced from [robu.in](https://robu.in).
- **Standard Sheet Stock**: 1200 × 2400 mm; multiple chassis parts nested on a single sheet to minimize material waste and unit cost.
- **CNC Milling**: LiDAR top plate machined externally (DML lab laser cutter used for panel profiles).
