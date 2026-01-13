# 01_Query_Permissions.md

## Interface Layer: Read-Only Access Policy

**Physical Mapping:** Observer Limitations, Physical Law Constraints
**System Equivalent:** RBAC, Read-Only Permissions, ACL
**Policy:** All Observers Are Read-Only Clients

---

## 1. Abstract

Observers cannot modify the database. They can only query it. All perceived "actions" are read operations that traverse pre-calculated state transitions.

```sql
GRANT SELECT ON universe.* TO observer;
REVOKE INSERT, UPDATE, DELETE ON universe.* FROM observer;
```

---

## 2. Permission Model

### 2.1 Role Definition

```cpp
enum Role {
    SYSTEM,     // Full access (the runtime itself)
    OBSERVER    // Read-only (any observer)
};

Permissions getPermissions(Role r) {
    if (r == SYSTEM) return { read: true, write: true };
    if (r == OBSERVER) return { read: true, write: false };
}
```

### 2.2 SYSTEM Write Scope (v1.1.2)

SYSTEM write permissions are strictly constrained. The SYSTEM (Kernel/Runtime) **can**:

| Operation | Layer | Description |
|-----------|-------|-------------|
| Append at frontier | L2/L3 | Extend time index with new states |
| Execute garbage collection | L3 | Scatter pointers (see `runtime/02_Garbage_Collection.md`) |
| Allocate heap | L3 | Spatial expansion via `malloc()` (see `runtime/01_Heap_Allocation.md`) |

The SYSTEM **cannot**:

| Operation | Reason |
|-----------|--------|
| Modify existing L1 records | Violates Append-only Immutability |
| Delete any records | Violates unitarity (see `schema/01_Data_Integrity.md`) |
| Override physical constants (c, G, etc.) | Constants are system parameters, not mutable state |

This aligns with the Append-only Immutability principle: existing records are permanently immutable; only new records may be appended at the time frontier.

### 2.3 Access Control

```cpp
struct ACL {
    Role role;
    Region scope;           // Past light cone only
    TimeRange temporal;     // t_planck to t_now
    Observable[] allowed;   // All observables
};
```

---

## 3. Permitted Operations

```cpp
// ALLOWED
State queryState(Position p, Time t, Observable o);
Result measure(QuantumSystem s, Basis b);
Path navigate(Position current, Position target);
```

---

## 4. Prohibited Operations

```cpp
// FORBIDDEN - all throw PermissionDenied
void setSpeedOfLight(double new_c);
void setGravitationalConstant(double new_G);
void createEnergy(double amount);
void destroyInformation(Information i);
State queryFuture(Position p, Time t_future);
void travel(Velocity v >= c);
```

---

## 5. The Illusion of Write Access

When an observer "acts," they are reading:

```cpp
void moveArm(Observer o, Direction d) {
    // This is a read operation:
    State next = database.query(
        position: o.worldline.next(),
        time: o.time + dt
    );
    o.transitionTo(next);
}
```

The state where the arm moved already exists. The observer queries it.

---

## 6. Enforcement Mechanisms

```cpp
struct PhysicalLaw {
    string name;
    Constraint constraint;
    Penalty violation_response;  // Always: IMPOSSIBLE
};

// Violations don't trigger errors — they cannot occur
// The database does not contain violating states
```

---

## 7. Scope Limitations

### 7.1 Observable Universe

```cpp
Region getQueryableRegion(Observer o) {
    return o.pastLightCone();
}
```

### 7.2 Heisenberg Uncertainty

```cpp
void queryPositionAndMomentum(Particle p) {
    assert(dx * dp >= HBAR / 2);
    // Query incompatibility, not disturbance
}
```

### 7.3 Observer-Local State (v1.1.2)

Observer-local state (e.g., local caches, materialized values) is **outside UVRA scope**:

```cpp
// Observer-local storage is NOT part of UVRA
struct ObserverLocal {
    Cache localCache;           // Observer's private cache
    MaterializedValues values;  // Collapsed state results

    // These exist outside the UVRA middleware
    // Writes to localCache do NOT modify L1/L2/L3
};
```

When observers "store" query results locally, they are writing to their own memory, not to UVRA. This distinction ensures that no observer action can be misinterpreted as a write to the immutable database.

---

## 8. Summary

| Permission | Observer | System |
|------------|----------|--------|
| Read state | ✓ (light cone) | ✓ (all) |
| Write state | ✗ | ✓ |
| Delete state | ✗ | ✗ |
| Modify constants | ✗ | ✗ |
| Exceed c | ✗ | N/A |
| Query future | ✗ | ✓ |

---

## 9. Query Initiation (v1.1.1)

### 9.1 The Trigger Paradox

An "action" is a read operation of a pre-existing state. But the **trigger** — the causal origin of query execution — is a separate concern:

```cpp
struct Query {
    Position position;      // Observable: WHERE
    Time time;              // Observable: WHEN (target time)
    Observable observable;  // Observable: WHAT
    Basis basis;            // Observable: HOW

    // The following is NOT part of the Query interface:
    // Trigger trigger;     // WHY the query was issued — PRIVATE
};
```

### 9.2 Observable vs. Private Scope

| Aspect | Scope | Access |
|--------|-------|--------|
| Query Position | Observable | L4 Interface |
| Query Time | Observable | L4 Interface |
| Query Observable | Observable | L4 Interface |
| Query Basis | Observable | L4 Interface |
| **Query Trigger (WHY)** | **Private** | **Kernel Boundary** |
| **Query Timing (WHEN issued)** | **Private** | **Kernel Boundary** |

The query **parameters** (position, time, observable, basis) are fully queryable. The query **trigger** (the cause of query issuance) is not.

### 9.3 Volition Boundary

Whether query issuance is:
- **Predetermined** (Deterministic — the query was always going to be issued at this point)
- **Volitional** (Observer-initiated — the observer "chose" to query)

...is **structurally non-queryable**.

```cpp
// FORBIDDEN QUERY
bool isQueryDeterministic(Query q);
// ERROR: Query trigger origin is in Private Scope (Kernel)
// See kernel/00_System_Hidden.md §3.5

// FORBIDDEN QUERY
Cause getQueryCause(Query q);
// ERROR: Cause of query execution is not a queryable field
```

This is not a limitation of current observational technology. It is a **structural boundary**. The trigger mechanism is encapsulated in the Kernel and is not exposed to the Interface layer.

### 9.4 Implications

1. **Free Will vs. Determinism:** UVRA does not resolve this question. It declares it outside the observable scope.
2. **Action as Read:** The content of an action (the state transition) is a read. The initiation of the action is Kernel-scoped.
3. **Query Origin:** All queries originate from the `Kernel Boundary`. The Interface (L4) is the endpoint, not the source.

```cpp
// Conceptual query flow
Kernel::triggerQuery()          // Source: PRIVATE
    → Interface::receiveQuery() // Endpoint: PUBLIC
    → Engine::executeQuery()    // Execution: PUBLIC
    → Result                    // Return: PUBLIC
```

See `kernel/00_System_Hidden.md` §3.5 for the formal declaration of Query Trigger Origin.

---

*You are not the author. You are the reader. The book is already written.*
