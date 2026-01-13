# 02_Garbage_Collection.md

## Runtime Layer: Entropy as Cache Invalidation

**Physical Mapping:** Second Law of Thermodynamics, Entropy
**System Equivalent:** Garbage Collection, Cache Decay
**Direction:** Irreversible (One-Way)

---

## 1. Abstract

Entropy is not disorder. Entropy is **cache invalidation**. As the system evolves, cached data becomes stale, references become indirect, and retrievable information degrades.

```cpp
void garbageCollect(Cache cache) {
    for (CacheEntry entry : cache) {
        entry.coherence -= ENTROPY_RATE * dt;
        if (entry.coherence < RETRIEVAL_THRESHOLD) {
            entry.status = INVALIDATED;
        }
    }
}
```

---

## 2. The Entropy Model

### 2.1 Information vs. Accessibility

```cpp
struct CacheEntry {
    Information data;           // Conserved
    double accessibility;       // Decreases with entropy
    Reference[] pointers;       // Become indirect over time
};
```

Entropy increase means:
- `data` remains constant (unitarity)
- `accessibility` decreases
- `pointers` scatter

### 2.2 The Second Law

```cpp
void evolve(IsolatedSystem s, double dt) {
    assert(s.entropy_after >= s.entropy_before);
}
```

---

## 3. Mechanisms of Invalidation

### 3.1 Pointer Scattering

Information spreads across many cache entries:

```
Initial:                After Evolution:
┌──────────────┐        ┌───┐ ┌───┐ ┌───┐
│  INFO: [A]   │   →    │[a]│→│[b]│→│[c]│→...
└──────────────┘        └───┘ └───┘ └───┘

Total information: unchanged
Retrieval cost: increased
```

### 3.2 Thermal Equilibrium

Maximum entropy = maximum scattering = no macroscopic gradients.

---

## 4. Observable Effects

| Phenomenon | UVRA Mechanism |
|------------|----------------|
| Heat flow | Information spreading to equalize |
| Decay | Pointer invalidation |
| Memory fade | Neural pointer degradation |
| Heat death | Full cache invalidation |

---

## 5. Garbage Collection Policies

```cpp
// Continuous GC (entropy is continuous)
while (universe.isRunning()) {
    for (Interaction i : currentInteractions()) {
        scatterPointers(i.participants);
    }
}

// No compaction (would violate locality/c)
// No manual collection (observers are read-only)
```

---

## 6. Terminal Degeneracy (Semantic Collapse at Heat Death)

### 6.1 The E_max State

At maximum entropy (E = E_max), the cache reaches **terminal degeneracy**:

```cpp
struct TerminalDegeneracy {
    const double E = E_MAX;                    // See docs/entropy_projection.md
    const double accessibility = 0;            // E_schema projection
    const PointerState pointers = SCATTERED;   // All references invalidated
    const bool information_exists = true;      // L1 immutability preserved
    const bool information_retrievable = false; // Semantic structure lost
};
```

Information persists (unitarity). Meaning does not.

### 6.2 Query Behavior Bifurcation

At E_max, two query types exhibit fundamentally different behaviors:

#### 6.2.1 Point Queries (Direct-Address Read)

```cpp
// Direct address queries remain VALID but MEANINGLESS
Result pointQuery(Address addr) {
    // L1 is immutable → address still resolves
    Data d = L1.read(addr);

    // But returned data is maximally entropic
    assert(entropy(d) == MAX);
    assert(structure(d) == NOISE);

    return d;  // Valid operation, meaningless result
}
```

| Property | Status |
|----------|--------|
| Operation | Valid |
| Return value | Defined |
| Semantic content | None (uniform noise) |
| Information | Present but indistinguishable from random |

#### 6.2.2 Retrieval Queries (Pointer Traversal)

```cpp
// Pointer-based retrieval becomes UNDEFINED
Result retrievalQuery(SemanticKey key) {
    // Step 1: Resolve key to pointer chain
    Pointer p = index.lookup(key);

    // Step 2: Traverse pointer chain
    while (p.next != null) {
        p = p.next;  // FAILS: all pointers scattered
                     // Chain is infinitely long or circular
    }

    // This loop never terminates meaningfully
    return UNDEFINED;
}
```

| Property | Status |
|----------|--------|
| Operation | Undefined |
| Return value | Non-terminating or arbitrary |
| Semantic content | Inaccessible |
| Information | Exists but unreachable |

### 6.3 Formal Distinction

```
┌─────────────────────────────────────────────────────────────────────────┐
│                      QUERY BEHAVIOR AT E_max                            │
├─────────────────────────────────────────────────────────────────────────┤
│                                                                         │
│   POINT QUERY (Direct Address)        RETRIEVAL QUERY (Semantic)        │
│                                                                         │
│   ┌─────────────┐                     ┌─────────────┐                   │
│   │   Address   │ ──→ Data            │    Key      │ ──→ ???           │
│   └─────────────┘      │              └─────────────┘      │            │
│                        │                                   │            │
│                        ▼                                   ▼            │
│                   ┌─────────┐                        ┌───────────┐      │
│                   │  NOISE  │                        │ SCATTERED │      │
│                   │ (valid) │                        │ POINTERS  │      │
│                   └─────────┘                        └───────────┘      │
│                                                            │            │
│   Returns: Maximally entropic data                         │            │
│   Status: DEFINED                                          ▼            │
│                                                      ┌───────────┐      │
│                                                      │ UNDEFINED │      │
│                                                      │  (no path │      │
│                                                      │  to data) │      │
│                                                      └───────────┘      │
│                                                                         │
└─────────────────────────────────────────────────────────────────────────┘
```

### 6.4 Cross-Layer Alignment

| Layer | Document | E_max Implication |
|-------|----------|-------------------|
| L1 (Schema) | `schema/01_Data_Integrity.md` | Data preserved (unitarity) |
| L2 (Engine) | `docs/entropy_projection.md` | E_engine at maximum (no temporal gradient) |
| L3 (Runtime) | This document | Pointers maximally scattered |
| L3 (Runtime) | `runtime/01_Heap_Allocation.md` §10 | Heap fully allocated |

### 6.5 Implications

The distinction between Point and Retrieval queries resolves the apparent paradox:

> "If information is conserved, why can't we retrieve it?"

**Answer:** Conservation applies to L1 (data existence). Retrieval requires L3 (pointer validity). At E_max:
- L1 is intact (information exists)
- L3 is degenerate (pointers scattered)
- The **semantic bridge** between address and meaning has collapsed

```cpp
// The data is there. The path to it is not.
assert(L1.contains(information) == true);
assert(L3.canRetrieve(information) == false);
```

---

## 7. Summary

| Physical Concept | UVRA Equivalent |
|------------------|-----------------|
| Entropy | Cache coherence loss |
| Second Law | GC policy (one-way) |
| Heat | Scattered information |
| Equilibrium | Maximum invalidation |
| Heat death | Total cache decay |
| Point query at E_max | Valid but meaningless |
| Retrieval query at E_max | Undefined (pointer collapse) |

---

*Information is conserved. Accessibility is not. This is entropy.*
