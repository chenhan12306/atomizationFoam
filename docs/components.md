# Components (Public APIs)

This section summarizes the main public classes and their key operations. See the generated API reference for the full list of methods and signatures.

## Foam::droplet
Spherical droplet parcel.

- Construction
  - `droplet(const polyMesh&, const vector& position, label cellI, scalar d, const vector& U)`
  - Stream constructor for parallel transfer
- Accessors (mutable references)
  - `scalar& d()` – diameter
  - `vector& U()` – velocity
  - `scalar& nParticle()` – number of physical droplets per parcel
  - `scalar& y()` – sphericity deviation; `scalar& yDot()` – its rate of change
- Tracking and interactions
  - `bool move(dropletCloud&, trackingData&, scalar dt)`
  - `bool hitPatch(...)`, `void hitProcessorPatch(...)`, `void hitWallPatch(...)`
  - `void transformProperties(const tensor& T)` / `transformProperties(const vector& separation)`
- I/O
  - `static void readFields(Cloud<droplet>&)` / `static void writeFields(const Cloud<droplet>&)`

Example (conceptual):
```cpp
// Given mesh, initial position and velocity
droplet p(mesh, position, cellI, initialDiameter, initialVelocity);
```

## Foam::dropletCloud
Cloud of droplets and drivers for collision and breakup.

- Construction
  - `dropletCloud(const fvMesh&, const dimensionedVector& g, const word& name="dropletCloud", bool readFields=true)`
- Access
  - `const fvMesh& mesh() const`
  - `scalar rhop() const`, `scalar mup() const`, `scalar sigma() const`
  - `vectorField& source()` – momentum source accumulator
  - `const volVectorField& momentumSource()`
  - Face-zone helpers: `const labelList& faceZoneIDs()`
  - Access droplet data buffers for post-processing
- Operations
  - `void move()` – advances all droplets
  - `void inject(vector position, scalar diameter, vector velocity)` – injects a droplet parcel

Example (conceptual):
```cpp
dropletCloud cloud(mesh, g);
cloud.inject({0,0,0}, 200e-6, {1,0,0});
cloud.move();
```

## Foam::collisionModel
Collision/coalescence modeling for droplets.

- Construction: `collisionModel(const fvMesh&)`
- Operation: `void update(dropletCloud& cloud, scalar dt)` – processes collisions over `dt`

## Foam::breakupModel
Secondary breakup modeling for droplets with multiple model options.

- Construction: `breakupModel(const fvMesh&)`
- Operation: `void update(dropletCloud& cloud, droplet::trackingData& td, scalar dt)`

Models supported (configure in `constant/cloudProperties`): `ReitzDiwakar`, `ETAB`, `PilchErman`.

## Foam::phaseCoupling
Couples VoF-resolved droplets to Lagrangian parcels.

- Construction
  - `phaseCoupling(const volVectorField& U, volScalarField& alpha, dropletCloud& cloud)`
- Operations
  - `void update()` – detect VoF droplets and convert to parcels based on thresholds
  - `volVectorField source()` – momentum source for carrier phase
  - `volScalarField damping()` – damping field for stabilization/removal

Example integration pattern in a solver loop (simplified):
```cpp
phaseCoupling coupling(U, alpha1, cloud);
while (runTime.run())
{
    coupling.update();   // Inject from VoF
    cloud.move();        // Advance parcels
    // ... solve alpha, U, p ...
}
```

## Application: atomizationFoam
A two-phase VoF solver with isoAdvector, extended with droplet coupling. Key calls (see `applications/atomizationFoam/atomizationFoam.C`):

- In each PIMPLE iteration:
  - `coupling.update();` – inject droplets from VoF
  - `cloud.move();` – move droplets and apply models