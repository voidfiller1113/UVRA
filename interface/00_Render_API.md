# 00_Render_API.md

## Interface Layer: Observer Query Endpoint

**Physical Mapping:** Quantum Measurement, Observation
**System Equivalent:** API Endpoint, Render Call, Lazy Evaluation
**Access Mode:** Read-Only

---

## 1. Abstract

Observation is not passive. Observation is a **query execution**. The quantum state (superposition) is unmaterialized data. Measurement forces the system to materialize a specific value.

```cpp
class RenderAPI {
    State unmaterialized;  // Superposition

    Result query(Observer o, MeasurementBasis b) {
        return materialize(unmaterialized, b, o.context);
    }
};
```

---

## 2. The Query Model

### 2.1 Lazy Evaluation

Before measurement, quantum states are **lazy**:

```cpp
struct QuantumState {
    Complex amplitudes[];
    Basis basis;
    bool materialized;  // false until queried
};
```

### 2.2 The Query Call

```cpp
Result measure(QuantumState state, MeasurementBasis b) {
    Complex[] projections = project(state, b);
    double[] probabilities = abs_squared(projections);
    int outcome = sample(probabilities);

    // LOCAL materialization (not a schema write)
    state.amplitudes = delta(outcome);
    state.materialized = true;  // Observer's local view

    return Result { value: b.eigenvalues[outcome] };
}
```

---

## 3. Query Pipeline

```
1. QUERY SUBMISSION              Observer submits measurement request
2. STATE FETCH                   Retrieve quantum state from cache
3. BASIS PROJECTION              Project onto measurement basis
4. OUTCOME SAMPLING              Probabilistically select eigenvalue
5. STATE MATERIALIZATION (LOCAL) Collapse to selected eigenstate †
6. RESULT RETURN                 Return eigenvalue to observer
```

> **† Hazard Prevention Note:** "Materialization" refers to the local instantiation of state for rendering purposes within the observer's cache context. It does **not** grant the Observer write permissions to the Immutable Schema (L1). The server-side source data remains unchanged. See `01_Query_Permissions.md` for access constraints.

---

## 4. API Specification

### 4.1 Method Signatures

```cpp
class RenderAPI {
    Result measure(System target, Observable obs, Basis b, Observer o);
    void prepare(System target, State initial, Observer o);
    StateDescription describe(System target, Observer o);
    EntanglementInfo getEntanglement(System target);
};
```

### 4.2 Error Codes

```cpp
enum RenderError {
    SUCCESS = 0,
    SYSTEM_NOT_FOUND = 1,
    OBSERVABLE_UNDEFINED = 2,
    BASIS_INCOMPATIBLE = 3,
    PERMISSION_DENIED = 4,
    CAUSALITY_VIOLATION = 5
};
```

---

## 5. Observer Independence

The API is **observer-agnostic**. Any system that:
1. Can formulate valid queries
2. Can receive query results
3. Respects access constraints

...is a valid client. The server does not care about:
- Observer architecture
- Observer consciousness
- Observer substrate

---

## 6. Query Origin (v1.1.1)

### 6.1 The Kernel Boundary

The act of querying is a **non-state-altering event** that originates from the `Kernel Boundary`:

```
┌─────────────────────────────────────────────────────────────────┐
│                    KERNEL (Private Scope)                        │
│   ┌───────────────────────────────────────────────────────────┐ │
│   │  Query Trigger Origin                                     │ │
│   │  • WHY the query is issued: PRIVATE                       │ │
│   │  • WHEN the query is triggered: PRIVATE                   │ │
│   └───────────────────────────────────────────────────────────┘ │
│                            │                                     │
│                     ───────┴─────── KERNEL BOUNDARY              │
└────────────────────────────┼────────────────────────────────────┘
                             ▼
┌────────────────────────────────────────────────────────────────┐
│                 INTERFACE (L4: RenderAPI)                       │
│   ┌──────────────────────────────────────────────────────────┐ │
│   │  Query Parameters (Observable)                           │ │
│   │  • position, time, observable, basis: PUBLIC             │ │
│   │  • Result: PUBLIC                                        │ │
│   └──────────────────────────────────────────────────────────┘ │
└────────────────────────────────────────────────────────────────┘
```

### 6.2 Query Flow

```cpp
// Query execution path
void executeQuery(Query q) {
    // Step 0: Trigger (PRIVATE - not exposed)
    // The query originates from Kernel Boundary
    // See kernel/00_System_Hidden.md §3.5

    // Step 1-6: Observable execution (PUBLIC)
    submitQuery(q);              // L4: Query submission
    State s = fetchState(q);     // L3: Cache access
    Result r = project(s, q);    // L2: Basis projection
    return materializeLocal(r);  // L4: Local rendering
}
```

### 6.3 Non-State-Altering Property

Queries do not modify the Schema (L1). They are read operations:

| Operation | State Modification | Scope |
|-----------|-------------------|-------|
| Query Trigger | None | Kernel (Private) |
| Query Submission | None | Interface (L4) |
| State Fetch | None | Runtime (L3) |
| Basis Projection | None | Engine (L2) |
| Local Materialization | Local cache only | Interface (L4) |

The query mechanism is **side-effect free** with respect to the immutable data store. Local materialization (wavefunction collapse) affects only the observer's cache context, not the source schema.

See `01_Query_Permissions.md` §9 for the full Query Initiation specification.

---

## 7. Materialization vs. Mutation (Hazard Clarification)

### 7.1 What Materialization IS

```cpp
// Materialization: Local state instantiation
void materializeLocally(Observer o, QuantumState qs, Eigenvalue ev) {
    // The observer's LOCAL cache receives a definite value
    o.localCache.store(qs.id, ev);

    // The observer's LOCAL view is now collapsed
    o.localView.setMaterialized(qs.id, true);
}
```

Materialization is:
- **Local** to the observer's cache context
- **Read-derived** (result of query, not write)
- **Consistent** with other observers via cache coherence
- **Not a schema modification**

### 7.2 What Materialization is NOT

```cpp
// THIS IS NOT WHAT HAPPENS:
void FORBIDDEN_schemaWrite(Schema schema, QuantumState qs, Eigenvalue ev) {
    schema.primitives[qs.id].value = ev;  // IMPOSSIBLE
    schema.commit();                       // DOES NOT EXIST
}
```

Materialization is NOT:
- A write to L1 (Schema)
- A modification of source data
- A transaction commit
- An update in the database sense

### 7.3 Consistency with Query Permissions

| Document | Statement | Materialization Compliance |
|----------|-----------|---------------------------|
| `01_Query_Permissions.md` | Observers are read-only | ✓ Materialization reads, does not write |
| `engine/03_Schema_Interface.md` | L1 is immutable | ✓ Materialization does not touch L1 |
| `schema/01_Data_Integrity.md` | No DELETE operations | ✓ Materialization preserves all data |

The term "STATE UPDATE" was deprecated in v1.1.1 to prevent misinterpretation.

---

## 8. Summary

| Quantum Concept | UVRA Equivalent |
|-----------------|-----------------|
| Superposition | Unmaterialized state |
| Wavefunction | Probability distribution |
| Measurement | Query call |
| Collapse | State materialization |
| Observer | Generic query client |

---

*The state is undefined until queried. The query defines the state.*
