# Molecular Dynamics Simulation of Laser-Assisted Cold Spray Deposition of Aluminum Particle on Aluminum Substrate Using EAM Potential

<p align="center">
  <img src="https://img.shields.io/badge/LAMMPS-MD%20Simulation-blue?style=for-the-badge&logo=gnu&logoColor=white"/>
  <img src="https://img.shields.io/badge/Cold%20Spray-Laser--Assisted%20Al%20Deposition-orange?style=for-the-badge"/>
  <img src="https://img.shields.io/badge/EAM-Al99%20Potential-purple?style=for-the-badge"/>
  <img src="https://img.shields.io/badge/Impact-500%20m%2Fs-red?style=for-the-badge"/>
  <img src="https://img.shields.io/badge/Laser%20Heating-600%20K-yellow?style=for-the-badge"/>
  <img src="https://img.shields.io/badge/OVITO-Visualization-green?style=for-the-badge"/>
  <img src="https://img.shields.io/badge/Ubuntu-WSL-red?style=for-the-badge&logo=ubuntu&logoColor=white"/>
  <img src="https://img.shields.io/badge/License-CC%20BY%204.0-lightgrey?style=for-the-badge"/>
</p>

<p align="center">
  A fully atomistic <b>molecular dynamics simulation of laser-assisted Cold Spray deposition
  of an Aluminum (Al) nanoparticle onto an Al substrate</b> using LAMMPS.
  A spherical Al particle (r = 10 Å) is launched at 500 m/s toward a laser-pre-heated Al
  substrate, capturing the full impact sequence — substrate laser heating to 600 K, particle
  approach, high-strain-rate deformation at impact, and post-impact cooling — all resolved
  at the angstrom scale.
  The simulation uses the <i>Al99 EAM/alloy potential</i>, NVT substrate thermostats with
  staged temperature ramp, per-atom von Mises stress and hydrostatic pressure fields, and
  multi-phase trajectory output for OVITO visualization.
</p>

<img width="1600" height="1200" alt="laser assisted cold spray" src="https://github.com/user-attachments/assets/c93d986a-5ebd-4399-9c21-aa4fa9a0b04b" />

---

## Why This is a Laser-Assisted Cold Spray Simulation

This simulation captures the **laser-assisted** variant of cold spray in two physically distinct stages:

- **Laser pre-heating** — the top substrate layer (z = 0 to 5 Å, the `laser_zone`) is ramped
  from 300 K to 600 K via NVT thermostat before particle impact, reproducing the thermal
  softening introduced by laser irradiation of the substrate surface. The bulk substrate
  remains at 300 K throughout, creating a thermal gradient that mimics experimental
  laser-spot heating.

- **Impact under elevated surface temperature** — the particle impacts a substrate whose
  top surface is at 600 K, reducing the critical velocity threshold and promoting plastic
  deformation and bonding. After impact, the laser zone is cooled back to 300 K in a
  controlled ramp, mimicking laser shut-off.

This contrasts with conventional cold spray MD simulations where the substrate is
uniformly thermalized at room temperature and no staged thermal field is applied.

---

## Simulation Overview

| Property | Value |
|----------|-------|
| Substrate | Aluminum — FCC, a₀ = 4.05 Å |
| Particle | Aluminum — FCC, a₀ = 4.05 Å |
| Potential | Al99 EAM/alloy (`Al99.eam.alloy`) |
| Box size | 100 × 100 × 100 Å (−50 to 50 in X/Y; −20 to 80 in Z) |
| Boundary conditions | Periodic in X/Y; shrink-wrap in Z |
| Substrate atoms | ~18,000 (estimated) |
| Particle atoms | ~354 (sphere r = 10 Å) |
| Timestep | 1 fs (0.001 ps) |
| Substrate ensemble | NVT — bulk at 300 K; laser zone ramped 300 → 600 → 600 → 300 K |
| Particle ensemble | NVE (full dynamics post-equilibration) |
| Impact velocity | −5.0 Å/ps = 500 m/s (downward, −Z) |
| Total simulation time | ~16 ps (1000 + 2000 + 2000 + 2000 + 4000 + 4000 + 3000 steps) |

---

## System Geometry

```
z =  80 Å  ┌─────────────────────────────┐  ← box top (shrink-wrap)
            │                             │
z =  40 Å  │          ●●●●●              │
            │        ●●●●●●●●●           │  ← Al particle (sphere r = 10 Å)
            │          ●●●●●              │  centre at z = 40, surface at z = 30
            │                             │
            │   (gap ≈ 25 Å travel zone)  │
            │                             │
z =   5 Å  ├═════════════════════════════┤  ← laser-heated zone top (600 K)
            │                             │
z =   0 Å  │   laser_zone (300→600 K)    │  ← top substrate: laser-heated layer
            │                             │
z =  −5 Å  │  ───────────────────────────│  ← substrate bulk (300 K)
            │                             │
z = −15 Å  │  ═══════════════════════════│  ← frozen bottom anchor layer
            │                             │
z = −20 Å  └─────────────────────────────┘  ← box bottom (shrink-wrap)
```

| Component | Atom type | Region | Z range | Temperature |
|-----------|-----------|--------|---------|-------------|
| Al particle | 1 | sphere (0, 0, 40), r = 10 Å | 30–50 Å | NVE |
| Laser-heated zone | 1 | block, z = 0 to 5 Å | 0 to 5 Å | 300 → 600 K |
| Al substrate bulk | 1 | substrate minus laser zone | −20 to 0 Å | 300 K |
| Frozen bottom anchor | 1 | block, z = −20 to −15 Å | −20 to −15 Å | Fixed |

---

## Simulation Phases

```
Energy Minimization (CG)
      ↓
Phase 0 — Initial Equilibration  (1,000 steps = 1 ps)
          Full substrate (bulk + laser zone) at 300 K; particle carries NVE
      ↓
Phase 1 — Laser Heating Ramp  (2,000 steps = 2 ps)
          Laser zone ramped 300 → 600 K; substrate bulk held at 300 K
      ↓
Phase 2 — Laser Temperature Hold  (2,000 steps = 2 ps)
          Laser zone maintained at 600 K; thermal gradient established
      ↓
      [Particle velocity already set: −5.0 Å/ps throughout above phases]
      ↓
Phase 3 — Approach  (2,000 steps = 2 ps)
          Particle travels toward heated substrate surface
      ↓
Phase 4 — Impact  (4,000 steps = 4 ps, dense output every 50 steps)
          High-strain-rate deformation; Al–Al bonding across hot interface
      ↓
Phase 5 — Post-Impact Relaxation  (4,000 steps = 4 ps)
          Kinetic energy redistribution; compressive stress wave propagates
      ↓
Phase 6 — Cooling Phase (Laser Off)  (3,000 steps = 3 ps)
          Laser zone cooled 600 → 300 K; system approaches new equilibrium
```

---

## Atom Types and Color Coding

| Type | Element | OVITO Color | Role |
|------|---------|-------------|------|
| 1 | Al | 🔴 Red | Substrate (mobile + frozen anchor) + particle |

Since this is a homogeneous-material simulation (Al–Al), differentiate groups in OVITO
using **Color Coding → by `v_von_mises`** to reveal the stress field, or use
**Slice at z = 5 Å** and **Common Neighbor Analysis (CNA)** to identify the disordered
impact/bonding zone at the original laser-heated surface.

---

## Repository Structure

```
LaserColdSpray_Al/
│
├── in.laser_coldspray_Al.lammps         # Main LAMMPS input script
├── Al99.eam.alloy                        # EAM/alloy potential file (required)
├── README.md                             # This file
│
└── output/                               # Generated on run
    ├── deposition.lammpstrj                   # Approach + post-impact (500-step frames)
    ├── vonmises.lammpstrj                     # Von Mises stress, simplified (500-step frames)
    ├── impact_detail.lammpstrj                # Impact phase, dense output (50-step frames)
    ├── final_deposited.data                   # Final LAMMPS data file
    └── deposition.restart                     # Binary restart file
```

---

## Requirements

- LAMMPS (7 Feb 2024 or newer, with MANYBODY package): https://www.lammps.org
- EAM potential file `Al99.eam.alloy` — place in the same directory as the input script
- OVITO for visualization: https://www.ovito.org

---

## Installation

```bash
# Ubuntu / WSL
sudo apt update && sudo apt install -y lammps
```

---

## Running the Simulation

```bash
# Single-core run
lmp -in in.laser_coldspray_Al.lammps

# Parallel run (recommended)
mpirun -np 4 lmp -in in.laser_coldspray_Al.lammps
```

---

## Simulation Parameters

### Thermostat and Integrator Assignment

| Group | Fix | Temperature | Notes |
|-------|-----|-------------|-------|
| `bottom` (frozen anchor) | `setforce 0.0 0.0 0.0` | — | Permanently frozen; acts as rigid anvil |
| `heated` (laser zone) | `nvt` 300 → 600 → 600 → 300 K | Staged ramp | Tdamp = 0.1 ps; simulates laser on/off cycle |
| `substrate_bulk` (Al mobile) | `nvt` 300 K | 300 K | Tdamp = 0.1 ps; constant background temperature |
| `particle` (Al) | `nve` throughout | None (adiabatic) | Kinetic impulse only; no thermostat during impact |

The `mobile` group (all atoms except `bottom`) uses a `compute mobile_temp` for thermo
monitoring. The particle is initialized with velocity −5.0 Å/ps at the start and evolves
under NVE throughout all phases.

### Laser Heating Parameters

| Parameter | Value |
|-----------|-------|
| Laser zone | z = 0 to 5 Å (top substrate layer) |
| Initial temperature | 300 K |
| Ramp target | 600 K (over 2,000 steps) |
| Hold temperature | 600 K (2,000 steps during approach) |
| Cool-down target | 300 K (over 3,000 steps, post-impact) |
| Thermostat damping | 0.1 ps |

### Impact Parameters

| Parameter | Value |
|-----------|-------|
| Particle velocity | −5.0 Å/ps (downward) = 500 m/s |
| Particle radius | 10 Å |
| Particle centre (start) | (0, 0, 40 Å) |
| Substrate top surface (laser zone) | z = 5 Å |
| Gap at start | ~25 Å |
| Approach phase | 2,000 steps (2 ps) |

### Stress Output

Per-atom stress tensor components and derived fields are computed for all atoms:

| Variable | Compute | Description |
|----------|---------|-------------|
| `sxx … syz` | `c_stress_atom[1..6]` | Virial stress components (bar·Å³) |
| `von_mises` | Derived from stress components | Equivalent von Mises stress (bar) |
| `pressure` | −(sxx + syy + szz) / 3 | Hydrostatic pressure (bar) |

Final summary statistics (max/avg von Mises, max/min pressure) are printed to the log at
the end of the cooling phase.

### Neighbor List

| Parameter | Value |
|-----------|-------|
| Skin distance | 2.0 Å |
| Update | every step, check yes |

---

## Key Script Features

| Feature | Implementation |
|---------|---------------|
| Laser-assisted heating | `laser_zone` NVT group ramped 300 → 600 K before impact; cooled post-impact |
| Thermal gradient | Laser zone at 600 K; substrate bulk pinned at 300 K via separate NVT fix |
| Staged output | Three dump files: approach/relaxation (500-step), von Mises simplified (500-step), impact dense (50-step) |
| Bottom boundary | `setforce 0.0 0.0 0.0` on bottom 5 Å of substrate — rigid anvil |
| Mobile temperature monitor | `compute mobile_temp mobile temp` — tracks true kinetic temperature excluding frozen atoms |
| Von Mises stress | Per-atom variable derived from full virial stress tensor — marks plastic deformation front |
| Hydrostatic pressure | Per-atom variable — tensile (negative) regions reveal adhesion zone at interface |
| Final analysis block | `run 0` after cooling evaluates and prints max/avg von Mises and max/min pressure |
| Performance target | < 30 min runtime, < 5 GB RAM — substrate thickness and box size optimized accordingly |

---

## Visualization in OVITO

### Access trajectory from Windows (WSL users)

```
\\wsl$\Ubuntu\home\<username>\output\impact_detail.lammpstrj
```

### Recommended modifiers

1. **Color Coding → Color by `v_von_mises`** — map per-atom von Mises stress; peak values mark the plastic deformation front at the impact zone
2. **Color Coding → Color by `v_pressure`** — hydrostatic pressure field; tensile (negative) regions beneath the impact crater indicate adhesion bonding
3. **Common Neighbor Analysis (CNA)** — FCC (green) vs. disordered/surface (white/red) atoms reveal the extent of the deformed bonding zone
4. **Polyhedral Template Matching (PTM)** — identify amorphous Al regions at the contact zone driven by high-strain-rate deformation
5. **Slice at z = 5 Å** — cross-section view of the original laser-heated surface to isolate the deposition interface
6. **Color Coding → by Particle Type** — since all atoms are Al (type 1), combine with a **Select → Expression** filter (`Position.Z > 28`) to visually isolate the particle from the substrate

---

## What to Expect

### After Initial Equilibration (Phase 0)
The Al substrate thermalizes to 300 K with random thermal motion visible.
The Al particle carries NVE dynamics and begins moving toward the substrate.
Thermo output shows `c_mobile_temp ≈ 300 K` uniformly across the system.

### During Laser Heating (Phases 1–2)
The laser zone (z = 0 to 5 Å) heats from 300 K to 600 K over 2 ps, then holds.
The substrate bulk remains at 300 K. A thermal gradient forms across the substrate surface,
softening the top layer and reducing the effective yield strength at the impact site.

### During Approach (Phase 3)
The particle travels toward the heated substrate surface at 500 m/s.
No significant stress or deformation yet — the surface temperature remains elevated at 600 K.
The `impact_detail` dump begins after this phase for high-resolution capture.

### During Impact (Phase 4)
This is the most physically rich phase. The Al particle contacts the 600 K substrate surface
and undergoes severe plastic deformation. The elevated surface temperature promotes enhanced
bonding relative to room-temperature cold spray. Von Mises stress peaks at the Al–Al contact
zone; a compressive stress wave propagates into the substrate. Dense output (every 50 steps)
captures the deformation front evolution.

### After Post-Impact Relaxation (Phase 5)
The particle has spread laterally and bonded to the substrate surface. Residual compressive
stress beneath the deposition site and tensile stress at the adhesion periphery are visible
in the per-atom pressure field. The disordered bonding zone at z ≈ 5 Å is identifiable via CNA.

### During Cooling (Phase 6)
The laser zone cools from 600 K back to 300 K over 3 ps, simulating laser shut-off.
The system approaches mechanical and thermal equilibrium. Residual stress locked into the
deposit reflects the thermal contraction of the formerly heated zone.

---

## Common Errors and Fixes

| Error | Cause | Fix |
|-------|-------|-----|
| `Al99.eam.alloy` not found | Potential file missing from run directory | Place `Al99.eam.alloy` in same folder as input script |
| Temperature explosion on impact | No thermostat on substrate bulk | Ensure `fix 3 substrate_bulk nvt` is active before particle impact |
| `compute` undefined variable error | Stress compute defined after variable reference | Place `compute stress_atom` before all `variable` definitions |
| Laser zone group empty | `laser_heated` region not intersecting substrate atoms | Verify `region laser_zone block` Z bounds overlap substrate top layer |
| `unfix 2` crash | Fix already removed in a prior `unfix` call | Track fix lifecycle; only unfix what is currently active |
| Atoms wrap through boundary | Using `p p p` instead of `p p s` in Z | Set `boundary p p s` to allow Z expansion during impact |
| `Lost atoms` | Atoms ejected beyond shrink-wrap Z range | Increase Z upper bound (currently 80 Å) |
| Stress units confusion | `stress/atom` returns bar·Å³ (virial), not pressure directly | Divide by per-atom volume if converting to pressure units |
| Group stale after impact | Region-based groups become invalid as atoms move | Use group definitions before energy minimization; avoid re-grouping mid-run |

---

## Extending the Simulation

| Extension | What to Change |
|-----------|---------------|
| Higher laser temperature | Raise NVT target in `heated` fix (e.g. 800 K for warm spray regime) |
| Larger laser spot | Expand `laser_zone` region in X/Y (e.g. `block -30 30 -30 30 0 5`) |
| Higher impact velocity | Increase particle velocity magnitude (e.g. `−8.0 Å/ps` = 800 m/s) |
| Larger particle | Increase sphere radius (e.g. 20 Å); adjust particle centre Z to maintain gap |
| Heterogeneous system | Change particle to Cu (type 2); update `pair_coeff` with CuAlW EAM potential |
| Multiple particle impacts | Re-run from `deposition.restart`; reposition a new particle above the deposit |
| Oblique laser + impact | Tilt particle velocity (x and z components); shift laser zone off-centre |
| Add oxide layer | Create thin Al₂O₃ region at z = 5–8 Å; add O–Al pair coefficients |
| Quantify bonding | Use `compute rdf` on atoms in the interface region (z = 0–10 Å) |
| Stress–strain curve | Reduce `compute stress/atom` over substrate slice vs. simulation time |
| Thermal diffusion analysis | `compute msd` on original laser-zone atoms to quantify heat-driven displacement |

---

## Citation

If you use this simulation in your research, please cite:

```bibtex
@software{mishra_2026_laser_coldspray,
  author    = {Mishra, Akshansh},
  title     = {Molecular Dynamics Simulation of Laser-Assisted Cold Spray Deposition},
  year      = {2026},
  publisher = {Zenodo},
  doi       = {10.5281/zenodo.20809176},
  url       = {https://doi.org/10.5281/zenodo.20809176}
}
```

Plain text citation:

> Mishra, A. (2026). *Molecular Dynamics Simulation of Laser-Assisted Cold Spray Deposition*. Zenodo. https://doi.org/10.5281/zenodo.20809176

---

## Author

**Akshansh Mishra**
GitHub: https://github.com/akshansh11

---

## License

<p>
<a rel="license" href="http://creativecommons.org/licenses/by/4.0/">
<img alt="Creative Commons Licence" style="border-width:0" src="https://i.creativecommons.org/l/by/4.0/88x31.png"/>
</a>
<br/>
This work is licensed under a
<a rel="license" href="http://creativecommons.org/licenses/by/4.0/">Creative Commons Attribution 4.0 International License</a>.
</p>

You are free to:

- **Share** — copy and redistribute the material in any medium or format
- **Adapt** — remix, transform, and build upon the material for any purpose, including commercially

Under the following terms:

- **Attribution** — You must give appropriate credit to Akshansh Mishra and provide a link to this repository

Copyright 2026 Akshansh Mishra.
