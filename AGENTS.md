# AGENTS.md

## Cursor Cloud specific instructions

### Overview

This is **atomizationFoam**, a C++ CFD solver for spray atomization built on OpenFOAM v2212. It consists of:

- `src/libAtomization/` — Lagrangian droplet library (shared lib `libAtomization.so`)
- `applications/atomizationFoam/` — The solver executable
- `run/crossFlow/` — Example simulation case

### Prerequisites

OpenFOAM v2212 is installed via the `openfoam2212-default` apt package at `/usr/lib/openfoam/openfoam2212`. The environment **must** be sourced before any build or run command:

```bash
source /usr/lib/openfoam/openfoam2212/etc/bashrc
```

This is already appended to `~/.bashrc`, so new interactive shells will have it automatically. For non-interactive shells (e.g. `Shell` tool calls), you must source it explicitly at the start of each command.

### Build

```bash
source /usr/lib/openfoam/openfoam2212/etc/bashrc 2>/dev/null
export FOAM_EXTRA_LDFLAGS="-Wl,-rpath-link,$FOAM_LIBBIN"
cd /workspace && ./Allwmake
```

**Gotcha**: On Ubuntu 24.04, the GNU linker requires `-Wl,-rpath-link,$FOAM_LIBBIN` to resolve transitive shared-library dependencies during linking. Without `FOAM_EXTRA_LDFLAGS`, the solver link step fails with undefined-reference errors from `libsurfMesh`, `libmeshTools`, etc. This env var is already set in `~/.bashrc`.

### Run (example case)

See `docs/usage.md` for full instructions. Quick single-process test run:

```bash
source /usr/lib/openfoam/openfoam2212/etc/bashrc 2>/dev/null
cd run/crossFlow
./Allclean 2>/dev/null; blockMesh && snappyHexMesh -overwrite && transformPoints -scale "(0.1 0.1 0.1)" && topoSet && atomizationFoam
```

For the full parallel run (4 MPI ranks), use `./Allrun` which additionally calls `decomposePar`, `runParallel`, `reconstructParMesh`, and `reconstructPar`.

### Lint / Tests

This is a pure C++ OpenFOAM project; there are no linter or automated test suites configured. Code correctness is verified by building (`./Allwmake`) and running the solver on the example case.

### Notes

- No Docker, databases, or network services are required.
- Build artifacts go to `~/OpenFOAM/ubuntu-v2212/platforms/linux64GccDPInt32Opt/{lib,bin}`.
- The harmless warning `export: '/usr/lib/openfoam/openfoam2212/etc/bashrc': not a valid identifier` appears when sourcing the OpenFOAM environment; it can be ignored.
