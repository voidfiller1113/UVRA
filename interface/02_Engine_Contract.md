# 02_Engine_Contract.md

## Interface Layer: L4→L2 Delegation Contract

**Contract Type:** Read-Only Delegation
**Version:** 1.1.2
**Dependency Direction:** L4 → L2 (unidirectional)

---

## 1. Abstract

This document formalizes the contract between the Interface (L4) and the Engine (L2). The Render API (L4) delegates all state retrieval and sampling operations to the Engine (L2). Observers interact exclusively with L4; direct L2 access is prohibited.

---

## 2. Contract Properties

### 2.1 Read-Only

```cpp
// All L4→L2 calls are const-qualified
class EngineContract {
    Result query(Position p, Time t, Observable o) const;
    State sample(QuantumSystem s, Basis b) const;
    FieldValue getField(Position p, Time t, FieldType f) const;

    // No mutators exist in this contract
};
```

The contract exposes no write operations. L4 cannot modify L2 state.

### 2.2 Deterministic

```cpp
// Identical inputs yield identical outputs
assert(engine.query(p, t, o) == engine.query(p, t, o));  // Always true
```

Query results are deterministic given identical parameters. There is no hidden state, no randomness at the contract level.

### 2.3 Permission-Gated

```cpp
struct QueryConstraints {
    Region allowed_region;  // Past light cone of observer
    TimeRange allowed_time; // t_planck to t_now

    bool validate(Query q, Observer o) const {
        return q.position.within(o.pastLightCone()) &&
               q.time >= T_PLANCK &&
               q.time <= o.t_now;
    }
};
```

Queries are constrained by:
- **Spatial:** Observer's past light cone
- **Temporal:** `t_planck ≤ t ≤ t_now` (no future queries for observers)

### 2.4 Indirect Schema Access

L4 does not access L1 (Schema) directly. The delegation chain is:

```
Observer → L4 (Interface) → L2 (Engine) → L1 (Schema)
                ↓                ↓
            Contract          SchemaView
           (this doc)     (03_Schema_Interface.md)
```

L4 queries L2. L2 queries L1 via `SchemaView`. Observers never touch L1 or L2 directly.

---

## 3. Prohibited Access Patterns

| Pattern | Status | Reason |
|---------|--------|--------|
| Observer → L2 (direct) | **PROHIBITED** | L4 is the only public entry point |
| Observer → L1 (direct) | **PROHIBITED** | Schema is not exposed |
| L4 → L1 (direct) | **PROHIBITED** | Must go through L2 |
| Observer → Kernel | **PROHIBITED** | Kernel is private scope |

```cpp
// FORBIDDEN: Direct L2 access
void Observer::directEngineCall() {
    engine.query(...);  // ERROR: Observers cannot call L2 directly
}

// CORRECT: Access via L4
void Observer::properQuery() {
    interface.renderAPI.query(...);  // L4 delegates to L2 internally
}
```

---

## 4. Contract Guarantees

### 4.1 No Mutation

This contract does not grant mutation capabilities to L4 or Observers. The entire query path is read-only:

```
Observer (read) → L4 (read) → L2 (read) → L1 (immutable)
```

### 4.2 No Kernel Exposure

The Kernel (private scope) is not exposed through this contract. Query trigger origin, system initialization, and persistence mechanisms remain inaccessible.

### 4.3 Consistent Semantics

Query semantics are inherited from L2. See `engine/03_Schema_Interface.md` for:
- Error conditions (`OUT_OF_BOUNDS`, `PRECISION_EXCEEDED`)
- Determinism guarantees
- Concurrency safety

---

## 5. API Delegation

| L4 Method | L2 Delegate | Description |
|-----------|-------------|-------------|
| `RenderAPI.query()` | `Engine.executeQuery()` | State retrieval |
| `RenderAPI.measure()` | `Engine.sample()` | Quantum sampling |
| `RenderAPI.describe()` | `Engine.getStateDescription()` | State metadata |

All L4 public methods delegate to corresponding L2 internal methods.

---

## 6. Summary

| Aspect | Specification |
|--------|---------------|
| Access Mode | Read-Only delegation |
| Entry Point | L4 is the only public interface |
| Direct L2 Access | Prohibited for Observers |
| Mutation | Not granted |
| Kernel Exposure | Not granted |
| Determinism | Guaranteed |

Observers query through L4. L4 delegates to L2. L2 reads from L1. The chain is unidirectional, read-only, and deterministic.

---

*The interface delegates. The engine executes. The observer receives.*
