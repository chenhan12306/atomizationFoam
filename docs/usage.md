# Usage

## Prerequisites
- A working OpenFOAM environment (tested with v2212)
- Dependencies required by your OpenFOAM installation (isoAdvector etc.)

Ensure the OpenFOAM environment is sourced in your shell (varies by installation):

```bash
source $WM_PROJECT_DIR/etc/bashrc
```

## Build

From the repository root:

```bash
./Allwmake
```

This compiles:
- `applications/atomizationFoam` (the solver)
- `src/libAtomization` (the Lagrangian droplet library)

## Run the example (crossFlow)

```bash
cd run/crossFlow
./Allclean 2>/dev/null || true
./Allrun
```

The script will:
- Mesh (blockMesh, snappyHexMesh), scale, and set topology sets
- Decompose, run `atomizationFoam` in parallel, and reconstruct results

## Key dictionaries

- `system/controlDict`
  - `application atomizationFoam;`
  - Time controls (`startTime`, `endTime`, `deltaT`, write controls, Co limits)

- `constant/cloudProperties`
  - `phaseName` (e.g., `water`)
  - `collision on|off`
  - `breakupModel` (e.g., `ReitzDiwakar`, `ETAB`, `PilchErman`)
  - `faceZones ( ... )` for post-processing
  - `phaseCoupling { ... }`
    - `active on|off`
    - `startTime <scalar>`
    - `nInterval <label>` (interval between coupling updates)
    - `alphaLimit <scalar>` (threshold for VoF cell detection)
    - `dMax <scalar>` (max injected droplet diameter)
    - `sphericity <scalar>` (max allowed sphericity)

- `system/fvSchemes` and `system/fvSolution`
  - Standard OpenFOAM scheme and solver settings, with isoAdvector controls for `alpha.*`

## Generate documentation

```bash
cd docs
doxygen Doxyfile
xdg-open build/html/index.html || open build/html/index.html || true
```

## Typical workflow in a custom case

1. Prepare mesh and boundary conditions
2. Tune `constant/cloudProperties` for droplet physics and coupling behavior
3. Adjust `system/*` for time-stepping, schemes, and linear solvers
4. Run `atomizationFoam`, post-process droplets and fields