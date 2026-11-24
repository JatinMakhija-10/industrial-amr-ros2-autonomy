# KiCad Electrical Schematic — AMR Power & Control

> This folder contains the complete KiCad 7 project for the AMR 24V power distribution and sensor interface schematic.

## Schematic Overview

The electrical architecture implements a layered protection and distribution system for a 24V direct-drive hub motor platform.

### Power Protection Chain

```
[24V Li-ion Battery Pack]
         │
         ▼
   [Blade Fuse — 30A]          ← Overcurrent protection for entire system
         │
         ▼
   [EMI Filter Module]          ← Suppresses motor switching transients from propagating upstream
         │
         ▼
   [Reverse Polarity MOSFET]   ← Prevents damage from accidental reverse battery connection
         │
         ▼
   [TVS Diode Array]            ← Clamps inductive kick spikes from motor switching (up to 60V peak)
         │
         ▼
   [Emergency Stop Switch]      ← DPDT — cuts motor power, keeps compute rail live
         │
         ▼
   [Safety Relay (NC)]          ← Software-controlled secondary motor cutoff
         │
    ┌────┴────┐
    │         │
    ▼         ▼
[Hub Motor  [Buck Converter]
 Controllers]  24V → 5V/5A
   (RS-485)       │
                  ▼
           [Raspberry Pi 5]
           [RPLiDAR A1M8]
           [BNO055 IMU]
```

### Communication Interfaces

| Interface | Connected Devices | Bus Spec |
|-----------|------------------|----------|
| RS-485 | Hub Motor Controllers (×2) | 120 Ω terminated at both ends; supports cable runs up to ~10 m |
| I2C (3.3V) | BNO055 IMU | 400 kHz fast mode; pull-up resistors on SBC GPIO |
| USB (Serial) | RPLiDAR A1M8 | CP2102 USB-Serial bridge; `/dev/ttyUSB0` |
| CSI | Camera Module | Direct RPi 5 camera connector |

### Key Design Decisions

**Why RS-485 for motors?**  
I2C and SPI are single-ended signals susceptible to common-mode noise. In a chassis with high-current motor switching, ground bounce and cable coupling corrupt single-ended bus signals. RS-485 uses differential signaling — only the voltage *difference* between D+ and D− is interpreted, rejecting common-mode noise up to ±7V. This makes it suitable for motor controller communication across the full chassis cable harness length.

**Why TVS diodes?**  
When a BLDC motor controller switches current through the motor windings, inductive kickback generates voltage spikes that can far exceed the 24V supply rail (measured up to 50–60V in similar platforms). TVS diodes clamp these transients before they reach the battery BMS or the SBC power input.

**Emergency Stop Architecture**  
The E-stop DPDT switch breaks the motor power rail while leaving the compute rail (buck converter output) energized. This means the SBC continues running, ROS 2 nodes stay alive, and the system can log its state and position at the moment of the emergency event. This is critical for diagnosing near-miss incidents in warehouse deployments.

## Project Files

| File | Description |
|------|-------------|
| `AMR_ElectronicDIa.kicad_sch` | Main KiCad 7 schematic file |
| `AMR_ElectronicDIa.kicad_pro` | KiCad project configuration |
| `AMR_ElectronicDIa.pdf` | Exported schematic PDF (ready for review) |
| `AMR_ElectronicDIa.svg` | Vector schematic export |
| `AMR_ElectronicDIa_BOM.csv` | Electrical BOM with component references |
| `AMR_Electrical_Schematic_Report.txt` | Schematic design report |
| `ERC.rpt` | Electrical Rules Check report |

## Viewing the Schematic

Open `AMR_ElectronicDIa.kicad_pro` in **KiCad 7** or later.  
To view without KiCad: use `AMR_ElectronicDIa.pdf`.
