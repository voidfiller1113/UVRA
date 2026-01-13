# 03_Schema_Interface.md

## Engine Layer: Formal Contract with Schema (L1)

**Interface Type:** Read-Only View
**Contract Version:** 1.1.1
**Dependency Direction:** L2 → L1 (unidirectional)

---

## 1. Abstract

This document formalizes the interface contract between the Engine (L2) and the Schema (L1). The Engine does not own, create, or modify Schema primitives. It maintains a **Read-Only View** of an immutable data source.

This contract ensures:
- Structural prohibition of L1 mutation by L2
- Deterministic query semantics
- Race-condition freedom across all observers

---

## 2. Access Model

### 2.1 Read-Only View

```cpp
class SchemaView {
private:
    const Schema& _schema;  // Immutable reference

public:
    SchemaView(const Schema& s) : _schema(s) {}

    // All access methods are const-qualified
    Primitive get(PrimitiveID id) const;
    FieldValue sample(Position p, Time t) const;
    bool exists(PrimitiveID id) const;

    // Mutation is structurally impossible
    // void set(...)  — NOT DECLARED
    // void delete(...) — NOT DECLARED
    // void update(...) — NOT DECLARED
};
```

The Engine receives a `const` reference to the Schema. Mutation methods do not exist in the interface. This is not a policy; it is a structural constraint enforced at the type level.

### 2.2 Access Pattern

```
┌─────────────────────────────────────────────────────────────┐
│                         L1: SCHEMA                          │
│                    (Immutable Data Store)                   │
│                                                             │
│   ┌─────────────────────────────────────────────────────┐   │
│   │              Vector Primitives                      │   │
│   │         Strings | Branes | Fields                   │   │
│   └─────────────────────────────────────────────────────┘   │
│                            ▲                                │
│                            │ const& (read-only)             │
│                            │                                │
└────────────────────────────┼────────────────────────────────┘
                             │
┌────────────────────────────┼────────────────────────────────┐
│                            │                                │
│                    L2: ENGINE                               │
│                 (Indexing & Logic)                          │
│                                                             │
│   ┌─────────────────────────────────────────────────────┐   │
│   │  SchemaView view;  // Read-only accessor            │   │
│   │                                                     │   │
│   │  // Can: query, index, traverse, sample             │   │
│   │  // Cannot: create, modify, delete                  │   │
│   └─────────────────────────────────────────────────────┘   │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

---

## 3. Assumptions (A1–A3)

The Engine operates under the following assumptions about the Schema. These are not verified at runtime; they are axiomatic preconditions.

### A1: Primitive Immutability

```cpp
// ASSUMPTION A1: Once a primitive exists, its value never changes.
// Formal: ∀ p ∈ Primitives, ∀ t₁ < t₂ : value(p, t₁) = value(p, t₂)

assert(schema.get(id, t1) == schema.get(id, t2));  // Always true
```

Primitives are eternal and unchanging. The Engine may cache primitive values indefinitely without invalidation concerns.

### A2: Unitary Transitions

```cpp
// ASSUMPTION A2: All state transitions are unitary (reversible).
// Formal: ∀ U ∈ Transitions : U†U = I

Operator U = getTransition(t1, t2);
assert(U.adjoint() * U == Identity);  // Always true
```

The Engine can traverse the time index in either direction. Forward and reverse queries are equally valid.

### A3: Append-Only History (Append-only Immutability)

```cpp
// ASSUMPTION A3: History is append-only. Past states are never overwritten.
// Formal: ∀ t < t_now : state(t) is immutable

State past = schema.getState(t_past);
// past is guaranteed to never change, regardless of future operations
```

The Engine may rely on historical queries returning consistent results across all time.

> **Clarification (v1.1.1):** "Append-only Immutability" means existing records cannot be modified or deleted, but the **System** (not the Engine, not Observers) may append new states. The Engine's Read-Only View does not grant access to the append mechanism. Appends are triggered by Runtime (L3) via `malloc` pressure. See `schema/01_Data_Integrity.md` §2.4 for the formal definition.

---

## 4. Guarantees (G1–G3)

Given the above assumptions, the Engine provides the following guarantees to higher layers.

### G1: Query Determinism

```cpp
// GUARANTEE G1: Identical queries yield identical results.
// Formal: query(p, t, o) = query(p, t, o) for all invocations

Result r1 = engine.query(position, time, observable);
Result r2 = engine.query(position, time, observable);
assert(r1 == r2);  // Always true
```

Determinism is unconditional. There is no cache invalidation, no stale reads, no eventual consistency. The same query always returns the same result.

### G2: Race-Condition Freedom

```cpp
// GUARANTEE G2: Concurrent queries from multiple observers never conflict.
// Formal: ∀ observers O₁, O₂ : query(O₁) ∥ query(O₂) is safe

// No locks required
// No transaction isolation required
// No optimistic concurrency control required

parallel_for(observers, [&](Observer o) {
    engine.query(o.position, o.time, o.observable);  // Safe
});
```

Because L1 is immutable and L2 is read-only, there are no write conflicts. Concurrency is trivially safe.

### G3: Reverse Traversal

```cpp
// GUARANTEE G3: The time index supports bidirectional traversal.
// Formal: ∀ t₁, t₂ : traverse(t₁ → t₂) and traverse(t₂ → t₁) are both valid

State s1 = engine.getState(t1);
State s2 = engine.evolve(s1, t1, t2);  // Forward
State s1_recovered = engine.evolve(s2, t2, t1);  // Reverse

assert(s1 == s1_recovered);  // Guaranteed by A2
```

Reverse traversal enables:
- Historical reconstruction
- Counterfactual queries
- Entropy accounting

---

## 5. Prohibited Operations

The following operations are structurally prohibited at the interface level:

| Operation | Status | Reason |
|-----------|--------|--------|
| `schema.create(primitive)` | NOT DECLARED | L2 cannot create L1 data |
| `schema.update(id, value)` | NOT DECLARED | L1 is immutable |
| `schema.delete(id)` | NOT DECLARED | Violates unitarity |
| `schema.transaction(...)` | NOT DECLARED | No transactions needed (no writes) |
| `schema.lock(id)` | NOT DECLARED | No locks needed (immutable) |

These are not runtime errors. The methods do not exist. Attempting to call them is a compile-time error.

---

## 6. Interface Definition (IDL)

```idl
interface SchemaView {
    // Primitive access
    readonly Primitive getPrimitive(PrimitiveID id);
    readonly bool primitiveExists(PrimitiveID id);

    // Field sampling
    readonly FieldValue sampleField(Position p, Time t, FieldType f);
    readonly FieldValue[] sampleFields(Position p, Time t, FieldType[] fs);

    // State access
    readonly State getState(Time t);
    readonly StateRange getStateRange(Time t_start, Time t_end);

    // Metadata
    readonly Time getMinTime();  // Returns t_planck (not t=0)
    readonly Time getMaxTime();  // Returns t_now (current index boundary)
};
```

All methods are `readonly`. The interface has no mutators.

---

## 7. Error Conditions

### 7.1 Valid Errors

| Error | Condition | Recovery |
|-------|-----------|----------|
| `OUT_OF_BOUNDS` | Query at t < t_planck | Return boundary value |
| `NOT_FOUND` | Primitive ID does not exist | Return null/empty |
| `PRECISION_EXCEEDED` | Query precision below Planck scale | Clamp and warn (see below) |

#### 7.1.1 PRECISION_EXCEEDED Semantics (v1.1.2)

When a query requests precision finer than the Planck scale:

```cpp
Result handlePrecisionExceeded(Query q) {
    // 1. Clamp precision to Planck scale
    // - If requested precision < PLANCK_TIME, set precision = PLANCK_TIME (coarsen to Planck granularity)
    // - If requested time < T_PLANCK, set time = T_PLANCK (minimum queryable)
    Time clamped_t = clamp(q.time, T_PLANCK, resolution: PLANCK_TIME);
    Position clamped_p = clamp(q.position, resolution: PLANCK_LENGTH);

    // 2. Execute query at clamped precision
    Result r = executeQuery(clamped_t, clamped_p, q.observable);

    // 3. Attach warning (query completes successfully)
    r.warning = PRECISION_EXCEEDED;
    r.actual_precision = PLANCK_SCALE;

    return r;  // Valid result, not an error
}
```

**Behavior:**
- Precision is clamped **up** to `t_planck` (or Planck scale) for execution
- Query returns the normal result at the clamped precision
- `PRECISION_EXCEEDED` is a **warning**, not a fatal error
- Query completes successfully (no undefined behavior)

This ensures that sub-Planck queries degrade gracefully rather than failing.

### 7.2 Impossible Errors

| Error | Why Impossible |
|-------|----------------|
| `WRITE_CONFLICT` | No writes exist |
| `DEADLOCK` | No locks exist |
| `STALE_READ` | Data is immutable |
| `ROLLBACK_FAILED` | No transactions exist |

---

## 8. Relationship to Other Documents

| Document | Relationship |
|----------|--------------|
| `schema/00_Vector_Primitives.md` | Defines the primitives this interface accesses |
| `schema/01_Data_Integrity.md` | Defines the unitarity constraint (A2) |
| `engine/00_Time_Index.md` | Uses this interface for temporal queries |
| `docs/entropy_projection.md` | Defines E_engine using index traversal |

---

## 9. Summary

| Aspect | Specification |
|--------|---------------|
| Access Mode | Read-Only (const reference) |
| Mutation | Structurally prohibited |
| Concurrency | Trivially safe (no writes) |
| Determinism | Unconditional |
| Reversibility | Guaranteed (unitary assumption) |

The Engine does not modify the Schema. The Engine cannot modify the Schema. This is not a policy. It is the architecture.

---

*The Engine reads. The Engine indexes. The Engine never writes.*
