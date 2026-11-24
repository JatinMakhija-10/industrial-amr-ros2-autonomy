# Fabrication Engineering & Drawing Package Index

> All manufacturing drawings were produced in Autodesk Fusion 360 and exported as formal 2-sheet drawing sets.
> Drawing standard: ASME Y14.5 (GD&T).  
> Total deliverables: **16 two-sheet drawing sets** covering all structural chassis components.

---

## Drawing Package Overview

Each drawing set follows a strict two-sheet structure:

| Sheet | Content |
|-------|---------|
| **Sheet 1/2** | Parts list (BOM sub-entries), raw stock dimensions, material callout, surface finish notes, DFM and fabrication instructions |
| **Sheet 2/2** | Orthographic projection views (front, top, side, isometric), sectional cuts where required, critical datum references, GD&T feature control frames |

---

## Component Drawing Index

| # | Drawing File | Component | Material | Fabrication Method | Key Dimensions / Features |
|---|-------------|-----------|----------|--------------------|--------------------------|
| 1 | `base plate.pdf` | Chassis Base Plate | Al 6061-T6, 3 mm | Precision Laser Cut | 318.0 × 347.5 mm primary envelope; motor mount bolt pattern |
| 2 | `chasis bottom plate.pdf` | Chassis Bottom Structural Plate | Al 6061-T6, 3 mm | Laser Cut | Main datum reference; caster mounting holes |
| 3 | `chasis.pdf` | Full Chassis Assembly Drawing | Multi-material | Assembly Reference | Ortho views; 271.0 mm confirmed height; complete BOM |
| 4 | `front.pdf` | Front Structural Panel | Al 5052-H32, 2 mm | Laser Cut + Press Brake | 22.8°, 36.05°, 53.95°, 150° chamfer angles; collision mitigation geometry |
| 5 | `backpart.pdf` | Chassis Back Part | Al 5052-H32, 2 mm | Laser Cut + Bending | 160° arc; R10, R3.77, R1.9 radii; 370.1 mm span |
| 6 | `left panel.pdf` | Left Side Enclosure Panel | Al 5052-H32, 2 mm | Laser Cut + Bending | Ø7.54 mm cable pass-through; 150° chamfer angle; 89.96 × 55.04 mm profile |
| 7 | `rugth panel.pdf` | Right Side Enclosure Panel | Al 5052-H32, 2 mm | Laser Cut + Bending | P/N 001-BTR2-Mk1; curved profile; 366.32 × 62.38 mm; 112.8° and 157.2° arcs |
| 8 | `righlt chassis fillet.pdf` | Right Chassis Fillet | Al 6061-T6 / Steel | CNC Mill | P/N 001-BTR2-Mk1; arcs 30°–157.2°; 349.61 and 346.83 mm reference dims |
| 9 | `Middle Bridge Section.pdf` | Middle Structural Bridge | Al 6061-T6 | Laser Cut + Mill | Internal chassis stiffener; prevents top-plate flex under sensor payload |
| 10 | `Front Face Plate.pdf` | Front Face Plate | Al 5052-H32, 2 mm | Laser Cut | Camera bracket and sensor routing apertures |
| 11 | `Side plate.pdf` | Side Reinforcement Plate | Al 6061-T6 | Laser Cut | Additional lateral stiffening member |
| 12 | `base.pdf` | Base Sub-Assembly Reference | Multi | Reference Drawing | Ground-level component packaging envelope |
| 13 | `Motor Mount.pdf` | Hub Motor Mounting Bracket | IS 2062 Steel, 4 mm | Laser Cut + CNC Mill | Right-angle bracket; 12 N·m stall torque rating; integrated axle clamp |
| 14 | `IMU Mount.pdf` | IMU Sensor Mounting Bracket | Al 6061-T6, 2 mm | CNC Mill | Precision planar surface for BNO055 alignment; vibration-isolated standoffs |
| 15 | `BATTERY MOUNT Drawing.pdf` | Battery Retention Cradle | IS 2062 Steel, 2 mm | Sheet Metal Forming | Low-CoG rear mounting; dynamic clamp for 24V Li-ion pack |
| 16 | `Drawings.pdf` | Master Assembly Drawing Set | Full Assembly | Reference | Complete orthographic views; BOM consolidated |

---

## DFM Notes Applied Across All Drawings

- **Laser Kerf**: 0.20 mm compensation applied to all hole true-positions for 3 mm aluminium
- **Bend Radii**: Minimum R = 1.5t (4.5 mm for 3 mm sheet); laser relief notches at all internal corners
- **Chamfer Strategy**: 30° outer corner chamfers (not 45°) — recovers +18% internal usable volume
- **Fastener Standard**: M3, M4, M5 ISO metric threads; A2-70 stainless steel hardware throughout
- **Material Nesting**: Multiple parts designed to nest within standard 1200 × 2400 mm sheet stock
- **Vibration Isolation**: Electronics mounting points use shore-50A rubber anti-vibration standoffs

---

## Tolerance Callouts (Critical Interfaces)

| Interface | Tolerance Class | Value |
|-----------|----------------|-------|
| Motor axle bore true-position | ISO 2768-m | ±0.1 mm |
| LiDAR mount face flatness | ISO 1101 | ≤ 0.05 mm |
| Chassis base plate hole pattern | ISO 2768-m | ±0.08 mm |
| Panel bend angle | ISO 2768-m | ±0.5° |
| Assembly height (overall) | Reference | 271.0 mm ± 0.5 mm |
