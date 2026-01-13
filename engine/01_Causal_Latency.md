# 01_Causal_Latency.md

## Engine Layer: Light Speed as Query Latency

**Physical Mapping:** Speed of Light (c), Causality Limit
**System Equivalent:** Maximum Query Propagation Speed
**Value:** 299,792,458 m/s

---

## 1. Abstract

The speed of light is not a property of photons. It is the **maximum query propagation speed** through the spatial index. No query can return results faster than `c` allows the index traversal to complete.

```cpp
const double QUERY_LATENCY_LIMIT = 299792458;  // m/s

double minQueryTime(Position source, Position target) {
    return distance(source, target) / QUERY_LATENCY_LIMIT;
}
```

---

## 2. The Latency Model

### 2.1 Index Traversal

When an observer at position `P₀` queries data at position `P₁`:

```
Observer (P₀) ──── Query ────→ Data (P₁)
                    │
                    │  Traversal Time = d/c
                    │
                    ▼
              Response arrives
              at t₀ + d/c
```

### 2.2 Why c is Finite

The index is not infinitely fast. Each node in the spatial index has processing time:

```cpp
// node_latency × node_density = 1/c
```

If `c` were infinite, the entire database would be queryable instantaneously from any point, violating causality.

---

## 3. Observable Effects

### 3.1 Light Cones

The set of queryable points forms a cone in spacetime:

- **Past Light Cone:** Events that could have sent queries reaching P by now
- **Future Light Cone:** Events that P's queries can reach

### 3.2 Causal Isolation

```cpp
bool canInfluence(Event a, Event b) {
    double spatial_dist = distance(a.position, b.position);
    double temporal_dist = abs(a.t - b.t);

    return spatial_dist <= temporal_dist * c;
}
```

### 3.3 Relativity of Simultaneity

Different observers disagree on what is "simultaneous" due to finite query latency combined with relative motion.

---

## 4. Entanglement and Latency

### 4.1 The Apparent Paradox

Entangled particles show correlated measurements instantaneously, seemingly violating `c`.

### 4.2 The Resolution

The correlation is **pre-indexed**:

```cpp
struct EntangledPair {
    CorrelationKey shared_index;  // Pre-established at creation
};

// No query travels from A to B
// The correlation was in the index all along
```

No information travels faster than `c`. The correlation is a database join, not a signal.

---

## 5. Summary

| Physical Concept | UVRA Equivalent |
|------------------|-----------------|
| Speed of light | Maximum query latency |
| Light cone | Queryable region boundary |
| Causality | Query ordering constraint |
| Entanglement | Pre-indexed correlation |
| Event horizon | Query timeout boundary |

---

*You cannot query faster than the index allows. The index allows c.*
