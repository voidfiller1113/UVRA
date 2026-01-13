# 02_Gravity_Weighting.md

## Engine Layer: Mass as Index Weighting

**Physical Mapping:** Gravitation, Spacetime Curvature
**System Equivalent:** Index Priority, Search Path Curvature
**Algorithm:** Weighted Graph Traversal

---

## 1. Abstract

Mass is not a "stuff" that "attracts" other stuff. Mass is **index weighting**. High-mass regions have higher priority in the search index, curving query paths toward them.

Gravity is not a force. It is an artifact of weighted index traversal.

```cpp
struct IndexNode {
    Position location;
    double mass_weight;
    IndexNode* neighbors[];
};

// Queries naturally route through high-weight nodes
```

> **Weight Field Clarification:** `mass_weight` is an L2 routing/lookup parameter. It may reflect aggregate structural effects and is not required to correspond 1:1 with locally observable L1 primitives. A "dark matter" effect corresponds to the residual `ΔW = mass_weight - visible_mass`—index weight contributions present in L2 that lack a direct L1-rendered counterpart at that location. This is an Engine-level abstraction, not missing data in L1.

---

## 2. The Weighting Model

### 2.1 Mass as Priority

```cpp
double indexPriority(IndexNode node) {
    return G * node.mass_weight / (c² * distance_to_query);
}
```

### 2.2 Geodesic Paths

The "straightest" path through a weighted index is a geodesic:

```cpp
Path findGeodesic(Position start, Position end) {
    return dijkstra(start, end, weightFunction: (node) => node.mass_weight);
}
```

Objects follow geodesics because geodesics are the natural query paths through weighted space.

---

## 3. Spacetime Curvature

### 3.1 Metric Tensor as Weight Map

```cpp
// Schwarzschild metric near mass M
double g_tt = -(1 - 2*G*M / (c²*r));
double g_rr = 1 / (1 - 2*G*M / (c²*r));

// These weights determine path costs
```

### 3.2 Einstein's Field Equations

```cpp
Tensor updateIndexWeights(Tensor energy_momentum) {
    // G_μν = (8πG/c⁴) T_μν
    return (8 * PI * G / pow(c, 4)) * energy_momentum;
}
```

---

## 4. Black Holes

### 4.1 Event Horizon

At the Schwarzschild radius, index weights become infinite:

```cpp
double indexWeight(Position p, Mass m) {
    double r = distance(p, m);
    if (r <= schwarzschildRadius(m)) {
        return INFINITY;
    }
    return G * m / (c² * r);
}
```

### 4.2 Query Trapping

Queries inside the event horizon cannot escape — all paths route further inward.

---

## 5. Summary

| Physical Concept | UVRA Equivalent |
|------------------|-----------------|
| Mass | Index weight |
| Gravity | Weighted path routing |
| Spacetime curvature | Index weight gradient |
| Geodesic | Weighted shortest path |
| Black hole | Infinite weight region |

---

*Mass bends the index. The index bends the path. The path is called gravity.*
