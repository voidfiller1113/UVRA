# Entropy Projection

## Cross-Layer Semantic Resolution

**Document Type:** Clarification Addendum
**Version:** 1.1
**Status:** Normative

---

## 1. Abstract

The term "Entropy" appears across multiple UVRA layers with seemingly different meanings. This document resolves the semantic ambiguity by defining **Entropy** as a single abstract quantity **E** that projects differently onto each layer.

There is one entropy. There are three projections.

---

## 2. The Problem

Entropy is referenced in:
- `schema/01_Data_Integrity.md` — as information accessibility
- `engine/00_Time_Index.md` — as temporal ordering constraint
- `runtime/02_Garbage_Collection.md` — as cache coherence decay

These usages appear inconsistent. They are not. They are **projections** of the same underlying quantity onto different layer-specific semantics.

---

## 3. Formal Definition

### 3.1 Abstract Entropy (E)

Let **E** be the abstract entropy of the system at time **t**:

```
E(t) : Time → ℝ⁺
```

**E** is:
- Monotonically non-decreasing: `dE/dt ≥ 0`
- Bounded above: `E(t) ≤ E_max`
- Defined for all `t ≥ t_planck`

**E** is not directly observable. Observers access layer-specific projections.

### 3.2 Projection Operators

Each layer defines a projection operator that maps **E** onto layer-specific semantics:

```cpp
// Projection operators
E_schema = π_schema(E);   // Information accessibility
E_engine = π_engine(E);   // Index ordering constraint
E_runtime = π_runtime(E); // Cache coherence decay
```

These projections are not independent values. They are **views** of the same quantity.

---

## 4. Layer Projections

### 4.1 E_schema: Information Accessibility (L1)

```cpp
// Schema projection: How accessible is stored information?
double E_schema = π_schema(E);

// Interpretation:
// Low E_schema  → Information concentrated, easily retrievable
// High E_schema → Information scattered, retrieval cost high
```

**Physical Mapping:** Thermodynamic entropy as information-theoretic measure.

**Formal Definition:**

```
E_schema(t) = -∑ᵢ pᵢ(t) log pᵢ(t)
```

Where `pᵢ` is the probability distribution over microstates.

**Reference:** See `schema/01_Data_Integrity.md` § Entropy and Information.

### 4.2 E_engine: Index Ordering Constraint (L2)

```cpp
// Engine projection: What direction does the index flow?
double E_engine = π_engine(E);

// Interpretation:
// E_engine defines the arrow of time
// Queries in +t direction see E_engine increasing
// Queries in -t direction see E_engine decreasing
```

**Physical Mapping:** Temporal asymmetry, arrow of time.

**Formal Definition:**

```
E_engine(t₂) > E_engine(t₁)  ⟺  t₂ > t₁
```

The engine uses entropy to **order** the time index. Increasing entropy defines "later."

**Reference:** See `engine/00_Time_Index.md` § The Arrow of Time.

### 4.3 E_runtime: Cache Coherence Decay (L3)

```cpp
// Runtime projection: How degraded is cache coherence?
double E_runtime = π_runtime(E);

// Interpretation:
// Low E_runtime  → Cache entries highly coherent, pointers direct
// High E_runtime → Cache entries invalidated, pointers scattered
```

**Physical Mapping:** Decoherence, thermalization, heat flow.

**Formal Definition:**

```
E_runtime(t) = ∫₀ᵗ decay_rate(τ) dτ
```

Where `decay_rate` is the rate of cache invalidation.

**Reference:** See `runtime/02_Garbage_Collection.md`.

---

## 5. Invariance Theorem

### 5.1 Monotonic Correspondence

**Theorem:** All three projections are monotonically related.

```
∀ t₁ < t₂ :
    E_schema(t₁) ≤ E_schema(t₂)  ∧
    E_engine(t₁) ≤ E_engine(t₂)  ∧
    E_runtime(t₁) ≤ E_runtime(t₂)
```

**Proof Sketch:** All projections derive from the same abstract **E**, which is monotonically non-decreasing. Projection operators preserve monotonicity.

### 5.2 Consistency Constraint

**Corollary:** If any projection decreases, all projections must decrease (which is forbidden).

```cpp
void checkConsistency(Time t1, Time t2) {
    assert(t2 > t1);

    // All projections must increase together
    assert(E_schema(t2) >= E_schema(t1));
    assert(E_engine(t2) >= E_engine(t1));
    assert(E_runtime(t2) >= E_runtime(t1));

    // Violation of any implies violation of all
    // Which implies violation of Second Law
    // Which is impossible
}
```

---

## 6. Projection Diagram

```
                         ┌─────────────────────┐
                         │   Abstract Entropy  │
                         │         E(t)        │
                         │                     │
                         │   dE/dt ≥ 0         │
                         │   E ≤ E_max         │
                         └──────────┬──────────┘
                                    │
              ┌─────────────────────┼─────────────────────┐
              │                     │                     │
              ▼                     ▼                     ▼
    ┌─────────────────┐   ┌─────────────────┐   ┌─────────────────┐
    │    π_schema     │   │    π_engine     │   │    π_runtime    │
    │                 │   │                 │   │                 │
    │   E_schema      │   │   E_engine      │   │   E_runtime     │
    │                 │   │                 │   │                 │
    │  Information    │   │  Index Order    │   │  Cache Decay    │
    │  Accessibility  │   │  Constraint     │   │  Rate           │
    └─────────────────┘   └─────────────────┘   └─────────────────┘
           │                     │                     │
           ▼                     ▼                     ▼
    ┌─────────────────┐   ┌─────────────────┐   ┌─────────────────┐
    │   L1: SCHEMA    │   │   L2: ENGINE    │   │   L3: RUNTIME   │
    │                 │   │                 │   │                 │
    │  "How hard to   │   │  "Which way     │   │  "How stale     │
    │   retrieve?"    │   │   is forward?"  │   │   is the cache?"|
    └─────────────────┘   └─────────────────┘   └─────────────────┘
```

---

## 6.1 Negentropy Definition (v1.1.2)

**Negentropy** is occasionally referenced in discussions of information organization. To prevent ambiguity, UVRA defines negentropy as a **derived quantity**, not an independent projection.

### Definition

```cpp
// Negentropy is the complement of entropy relative to E_max
double Negentropy_schema(Time t) {
    return E_MAX - E_schema(t);
}
```

**Formal:**

```
Negentropy(t) = E_max - E(t)
```

Where:
- `E_max` = maximum entropy (heat death value)
- `E(t)` = abstract entropy at time `t`

### Properties

| Aspect | Specification |
|--------|---------------|
| Type | Derived quantity (not independent projection) |
| Relationship | `Negentropy(t) + E(t) = E_max` (constant sum) |
| Monotonicity | Non-increasing: `dNegentropy/dt ≤ 0` |
| Boundary Values | `Negentropy(t_planck) = E_max - E_min`, `Negentropy(t_heat_death) = 0` |

### Normative Usage

When any UVRA document references "negentropy," it refers to this definition:

```cpp
// Canonical negentropy mapping
Negentropy(t) ≡ E_MAX - E(t)
```

Negentropy is **not** a fourth projection. It is an alias for the remaining capacity for entropy increase.

> **Normative Equivalence:** UVRA identifies `TOTAL_INFORMATION` with `E_MAX`. By information conservation, `TOTAL_INFORMATION` is constant; at heat death, `E(t) = E_MAX = TOTAL_INFORMATION`.

---

## 7. Usage Guidelines

### 7.1 When to Use Each Projection

| Context | Use | Reason |
|---------|-----|--------|
| Information theory | E_schema | Measures retrieval difficulty |
| Temporal queries | E_engine | Defines time direction |
| Cache management | E_runtime | Measures coherence loss |
| Cross-layer analysis | E (abstract) | Underlying invariant |

### 7.2 Common Mistakes

| Mistake | Correction |
|---------|------------|
| Treating E_schema, E_engine, E_runtime as independent | They are projections of one quantity |
| Expecting different monotonicity | All projections share monotonicity |
| Mixing layer semantics | Use layer-appropriate projection |

---

## 8. Boundary Values

| Condition | E_schema | E_engine | E_runtime |
|-----------|----------|----------|-----------|
| t = t_planck | Minimal | Minimal | Minimal |
| t = t_now | Current | Current | Current |
| t → heat_death | E_max | E_max | E_max |

At heat death, all projections reach their maximum. The system is maximally entropic in all senses simultaneously.

---

## 9. Summary

| Aspect | Specification |
|--------|---------------|
| Entropy | Single abstract quantity E |
| Projections | Three: E_schema, E_engine, E_runtime |
| Relationship | Monotonically coupled |
| Independence | None (all derive from E) |
| Consistency | Guaranteed by shared source |

There is no "entropy ambiguity." There is one entropy with three views.

---

*One entropy. Three projections. Zero ambiguity.*
