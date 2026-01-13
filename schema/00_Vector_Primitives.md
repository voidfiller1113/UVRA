# 00_Vector_Primitives.md

## Schema Layer: Fundamental Geometric Primitives

**Physical Mapping:** Strings, Branes, Quantum Fields
**Data Type:** 2D Vector (Resolution-Independent)
**Storage Class:** Immutable

---

## 1. Abstract

All data in UVRA originates from **2D vector primitives**. These are not 3D objects projected onto a plane — they are the source geometry from which higher-dimensional representations are derived.

The universe is not rasterized. There are no pixels. The source data is vector-based, meaning it can be queried at arbitrary precision without loss of fidelity.

---

## 2. Primitive Types

### 2.1 String Primitive

```cpp
struct StringPrimitive {
    double tension;           // Energy per unit length (T)
    Complex[] modes;          // Vibrational modes (quantized)
    Topology topology;        // OPEN | CLOSED
    Dimension embedding_dim;  // Dimension of target space (typically 10/11)
};
```

**Physical Mapping:** Fundamental strings in string theory.
**Resolution:** Planck length (l_p ≈ 1.6 × 10⁻³⁵ m) — not a pixel size, but an index granularity.

### 2.2 Brane Primitive

```cpp
struct BranePrimitive {
    uint8_t dimensionality;   // p-brane dimension (0 = point, 1 = string, 2 = membrane, ...)
    Tensor charge;            // Ramond-Ramond charge
    EmbeddingMap embedding;   // Map from brane worldvolume to target space
};
```

**Physical Mapping:** D-branes, M-branes.
**Special Case:** 3-brane defines the observable spatial cache.

### 2.3 Field Primitive

```cpp
struct FieldPrimitive {
    SpacetimePoint domain;    // Point in target space
    Complex value;            // Field amplitude
    SpinorIndex spin;         // Spin representation (0, 1/2, 1, 3/2, 2)
    GaugeIndex gauge;         // Gauge group representation
};
```

**Physical Mapping:** Quantum fields (scalar, spinor, vector, graviton).

---

## 3. Vector vs. Raster

### 3.1 Why Vector?

| Property | Vector | Raster |
|----------|--------|--------|
| Resolution | Infinite (query-dependent) | Fixed (pixel-dependent) |
| Scaling | Lossless | Lossy (aliasing) |
| Storage | Compact (equations) | Large (pixel arrays) |
| Precision | Arbitrary | Limited by bit depth |

Reality exhibits vector properties:
- No aliasing at small scales (quantum effects are smooth)
- Arbitrary zoom reveals structure (fractals, self-similarity)
- Fundamental constants have arbitrary precision

### 3.2 Apparent Quantization

Observers perceive quantization (discrete energy levels, Planck units). This is not rasterization. It is **index granularity**.

```
Source:  Continuous vector field
Index:   Discrete buckets (Planck scale)
Query:   Returns value at bucket resolution
Result:  Appears quantized (eigenvalues)
```

The source data is continuous. The query interface discretizes it.

---

## 4. Coordinate System

### 4.1 Target Space

Primitives exist in a high-dimensional target space:

```cpp
const int TARGET_DIMENSIONS = 11;  // M-theory compactification

struct TargetPoint {
    double coords[TARGET_DIMENSIONS];
    // coords[0..3]: Spacetime (Minkowski)
    // coords[4..10]: Compactified (Calabi-Yau)
};
```

### 4.2 Compactification

Extra dimensions are compactified at Planck scale:

```cpp
double compactRadius = PLANCK_LENGTH;  // ~10⁻³⁵ m

// Only coords[0..3] are macroscopically observable
// coords[4..10] curl into a compact manifold
```

Observers query a 4D projection. The full 11D data is accessible but requires specialized queries (high-energy collisions).

---

## 5. Data Integrity

Vector primitives are **immutable**. Once written, they cannot be modified. This ensures:

1. **Unitarity:** Quantum evolution is reversible (see `01_Data_Integrity.md`)
2. **Consistency:** No race conditions between observers
3. **Auditability:** Complete history preserved

```cpp
// FORBIDDEN
void modifyPrimitive(StringPrimitive* s, double new_tension);

// ALLOWED
StringPrimitive createPrimitive(double tension);
```

---

## 6. Query Interface

Observers access primitives through the Engine layer (L2). Direct schema access is prohibited.

```cpp
// Observer query (via Engine)
QueryResult result = engine.query(
    position: {x, y, z, t},
    precision: PLANCK_SCALE,
    fields: {ELECTRIC, MAGNETIC, GRAVITATIONAL}
);

// Returns interpolated field values at specified position
// Precision bounded by index granularity, not source resolution
```

---

## 7. Summary

| Concept | UVRA Definition |
|---------|-----------------|
| Strings | 1D vector primitives with vibrational modes |
| Branes | p-dimensional vector primitives |
| Fields | Point-valued vector primitives |
| Quantization | Index granularity, not source discretization |
| Resolution | Infinite (query-dependent) |

The schema stores vectors. The engine indexes them. The runtime caches them. The interface renders them.

---

## 8. Definition of "2D Source"

### 8.1 Semantic Clarification

Throughout UVRA, the term "2D" refers to the **dimensionality of the source manifold**, not a spatial embedding. The "2D Source" is:

- **NOT** a plane embedded in 3D space
- **NOT** a projection of 3D onto 2D
- **IS** an abstract information manifold with two intrinsic dimensions

### 8.2 Formal Definition

```cpp
// The "2D" in "2D Vector Primitives" refers to worldsheet dimensionality
struct Worldsheet {
    double sigma;  // Spatial parameter (0 ≤ σ ≤ π for open strings)
    double tau;    // Temporal parameter (worldsheet time)

    // These are INTRINSIC coordinates
    // They do not reference any embedding space
};
```

The worldsheet is a 2-dimensional manifold parameterized by `(σ, τ)`. It is the **domain** of string/brane physics, not an object floating in higher-dimensional space.

### 8.3 Relationship to Higher Dimensions

```
┌─────────────────────────────────────────────────────────────┐
│                    DIMENSIONAL HIERARCHY                    │
│                                                             │
│   2D Worldsheet (σ, τ)                                      │
│        │                                                    │
│        │ Embedding map X^μ(σ, τ)                            │
│        ▼                                                    │
│   10D/11D Target Space (X⁰, X¹, ..., X⁹, [X¹⁰])            │
│        │                                                    │
│        │ Compactification                                   │
│        ▼                                                    │
│   4D Macroscopic Spacetime (t, x, y, z)                    │
│        │                                                    │
│        │ Spatial projection                                 │
│        ▼                                                    │
│   3D Spatial Cache (x, y, z) at fixed t                    │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

The 2D source is **fundamental**. Higher dimensions are derived via:
1. **Embedding maps** (worldsheet → target space)
2. **Compactification** (target space → 4D spacetime)
3. **Spatial slicing** (4D spacetime → 3D cache at time t)

### 8.4 Why 2D?

The 2D source has special properties:

| Property | 2D Advantage |
|----------|--------------|
| Conformal invariance | Worldsheet theory is conformally invariant |
| Holography | 2D boundary encodes bulk (AdS/CFT) |
| Anomaly cancellation | Critical dimension constraints |
| Modular invariance | Consistent quantum theory |

These are not arbitrary choices. They are mathematical necessities for consistent quantum gravity.

### 8.5 Common Misconceptions

| Misconception | Correction |
|---------------|------------|
| "2D means flat plane" | 2D means intrinsically 2-dimensional manifold |
| "2D is embedded in 3D" | 2D is the source; 3D is a projection |
| "2D is a simplification" | 2D is fundamental; 3D is derived |
| "Observers live in 2D" | Observers query a 3D cache projected from 2D |

---

*The source is continuous. The query is discrete. The confusion is yours.*
