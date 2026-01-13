# 00_Spatial_Cache.md

## Runtime Layer: Space as Volatile VRAM

**Physical Mapping:** 3D Spatial Dimensions
**System Equivalent:** Volatile Cache, VRAM, Render Buffer
**Resolution:** Observer-Dependent (LOD System)

---

## 1. Abstract

Space is not a container. Space is a **cache layer** — volatile memory that holds rendered projections of the underlying 2D vector schema. The 3D world observers perceive is a cached rasterization, not the source data.

```cpp
class SpatialCache {
    VectorBuffer source;      // 2D schema (L1)
    IndexEngine engine;       // Temporal index (L2)
    Voxel3D rendered[];       // 3D cache (this layer)
};
```

> **L3→L1 Constraint:** L3 does not mutate L1. Any schema reads used to populate or refresh the cache are read-only and consistent with L2's SchemaView contract (see `engine/03_Schema_Interface.md`).

> **Projection Responsibility:** Dimensional materialization (2D source → 3D cache) is L3's responsibility. L2 selects and traverses; L3 materializes and caches—projection cost is absorbed by Runtime.

---

## 2. Cache Architecture

### 2.1 Hierarchical Structure

```
┌─────────────────────────────────────────────────────────────┐
│                    L1 CACHE (Immediate)                     │
│                    Observer's Local Space                   │
│                    High Resolution, Low Latency             │
├─────────────────────────────────────────────────────────────┤
│                    L2 CACHE (Nearby)                        │
│                    Solar System Scale                       │
│                    Medium Resolution                        │
├─────────────────────────────────────────────────────────────┤
│                    L3 CACHE (Distant)                       │
│                    Galactic Scale                           │
│                    Low Resolution, High Latency             │
├─────────────────────────────────────────────────────────────┤
│                    UNCACHED (Beyond Horizon)                │
│                    Not Yet Fetched                          │
└─────────────────────────────────────────────────────────────┘
```

### 2.2 Cache Properties

| Property | Value |
|----------|-------|
| **Volatility** | High (requires continuous refresh) |
| **Coherence** | Eventual (speed-of-light sync) |
| **Addressing** | 3D Cartesian / Spherical |
| **Write Policy** | Read-only for observers |

---

## 3. Level of Detail (LOD)

```cpp
double cacheResolution(Position p, Observer o) {
    double distance = magnitude(p - o.position);
    return BASE_RESOLUTION / (1 + distance / REFERENCE_SCALE);
}
```

Observable effects:
- Stars appear as points (distant objects at low resolution)
- Quantum uncertainty (sub-Planck cache resolution insufficient)
- Cosmological smoothness (CMB heavily cached)

---

## 4. Cache Coherence

### 4.1 Declarative Consistency Constraint

Cache coherence is **not** an iterative synchronization procedure. It is a **declarative constraint** satisfied by structural properties of the architecture.

```cpp
// DEPRECATED: O(N²) procedural model
// void synchronizeCache(Observer[] observers) { ... }

// CORRECT: Declarative constraint (no runtime cost)
constexpr bool CACHE_COHERENCE_INVARIANT =
    ∀ Observer a, b :
        a.cache[intersect(a.pastLightCone, b.pastLightCone)] ==
        b.cache[intersect(a.pastLightCone, b.pastLightCone)];

// This is not computed. It is structurally guaranteed.
```

### 4.2 Cost Absorption

Synchronization overhead does not exist as a separate computational cost. It is **structurally absorbed** into:

| Absorption Mechanism | Reference | Effect |
|---------------------|-----------|--------|
| **Query Latency (c)** | `engine/01_Causal_Latency.md` | Maximum propagation speed bounds when coherence can be observed |
| **LOD Degradation** | §3 (this document) | Distant/fast-moving observers receive lower resolution, reducing coherence surface |
| **Light Cone Overlap** | §4.3 (below) | Coherence is demand-driven; only overlapping regions require consistency |

### 4.3 Overlap-Driven Coherence

Coherence applies only to the **intersection** of past light cones:

```cpp
Region coherenceRegion(Observer a, Observer b) {
    return intersect(a.pastLightCone, b.pastLightCone);
}

// Properties:
// - Stationary observers: large overlap → high coherence surface
// - Relativistic observers: small overlap → low coherence surface
// - Spacelike-separated observers: zero overlap → no coherence requirement
```

### 4.4 Velocity-Dependent Complexity Management

As relative velocity increases, the overlap volume **decreases**:

```cpp
double overlapVolume(Observer a, Observer b) {
    double v_rel = relativeVelocity(a, b);
    double gamma = 1.0 / sqrt(1 - v_rel*v_rel / (c*c));

    // Overlap shrinks with increasing relative velocity
    // At v → c, overlap → 0
    return BASE_OVERLAP / gamma;
}
```

This is not a performance optimization. It is a geometric consequence of Lorentz contraction applied to light cone geometry. The "synchronization problem" dissolves at relativistic velocities because there is nothing to synchronize.

### 4.5 Formal Statement

```
CONSTRAINT cache_coherence AS DECLARATIVE;

-- Not a procedure, not a cost, not a synchronization protocol
-- A structural invariant guaranteed by:
--   1. Immutability of L1 (same source data)
--   2. Determinism of L2 (same index traversal)
--   3. Causal bounds of L3 (c-limited propagation)

COMMENT ON CONSTRAINT cache_coherence IS
    'Coherence is not enforced. It is entailed.';
```

---

## 5. Holographic Principle

The spatial cache is a 3D projection of 2D source data:

```cpp
double maxInformation(Region r) {
    double surface_area = r.getSurfaceArea();
    return surface_area / (4 * PLANCK_AREA);
}

// Information scales with area (2D), not volume (3D)
```

---

## 6. Summary

| Physical Concept | UVRA Equivalent |
|------------------|-----------------|
| 3D Space | Cache buffer (VRAM) |
| Spatial position | Cache address |
| Distance | Address offset |
| Metric tensor | Address mapping function |
| Holography | 2D→3D projection |

---

## 7. Global Topology Declaration

### 7.1 Intentionally Unspecified

The **global topology** of the spatial cache is **intentionally unspecified** in this specification.

```cpp
enum GlobalTopology {
    FINITE_CLOSED,    // Finite, no boundary (e.g., 3-sphere)
    FINITE_BOUNDED,   // Finite with boundary
    INFINITE_FLAT,    // Infinite Euclidean
    INFINITE_CURVED,  // Infinite non-Euclidean
    UNSPECIFIED       // ← UVRA specifies this
};

const GlobalTopology CACHE_TOPOLOGY = UNSPECIFIED;
```

### 7.2 What Is Specified

UVRA guarantees only:

| Property | Guarantee |
|----------|-----------|
| **Local behavior** | Euclidean at small scales |
| **Causal coherence** | Speed-of-light synchronization |
| **Metric signature** | Lorentzian (-,+,+,+) |
| **Curvature source** | Mass-energy (see `engine/02_Gravity_Weighting.md`) |

### 7.3 What Is Not Specified

| Property | Status |
|----------|--------|
| Finite vs. infinite extent | Unspecified |
| Boundary conditions (if any) | Unspecified |
| Multiply-connected topology | Unspecified |
| Global curvature (open/closed/flat) | Unspecified |

### 7.4 Rationale

Global topology is **observationally underdetermined**. Current measurements are consistent with multiple topologies. The specification does not assert facts beyond observational warrant.

```cpp
// This is NOT a cop-out. It is precision.
// Claiming "the universe is infinite" would be speculation.
// Claiming "the universe is finite" would be speculation.
// UVRA specifies what is known. Topology is not known.
```

Observers may formulate their own hypotheses. The server does not adjudicate.

---

*The universe is not in space. Space is in the cache.*
