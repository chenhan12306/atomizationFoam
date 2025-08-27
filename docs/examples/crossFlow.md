# Example: crossFlow

This example demonstrates meshing, running `atomizationFoam` in parallel, and reconstructing results.

## Run

```bash
cd run/crossFlow
./Allrun
```

What happens:
- Mesh generation: `blockMesh`, `snappyHexMesh -overwrite`, scaling, `topoSet`
- Decomposition: `decomposePar`
- Parallel run: `mpirun -np <N> atomizationFoam -parallel` (via `runParallel`)
- Reconstruction: `reconstructParMesh`, `reconstructPar`

## Key controls

- `system/controlDict`: simulation time, write controls, Co numbers
- `constant/cloudProperties`: droplet physics, coupling thresholds
- `system/fvSchemes` and `system/fvSolution`: numerical schemes and solvers

Adjust `constant/cloudProperties` to explore the effect of `alphaLimit`, `dMax`, `sphericity`, and collision/breakup toggles.