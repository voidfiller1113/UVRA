# 00_System_Hidden.md

## Kernel Layer: The Unknowable

**Scope:** `private`
**Access Level:** None
**Query Permission:** Denied
**Documentation Status:** Boundary Definition Only

---

## 1. Purpose

This document defines what **cannot** be defined. It establishes the boundary between the queryable system (UVRA) and the non-queryable driver that powers it.

Every database requires:
- Power (computation substrate)
- Initialization (first write)
- Persistence (continued execution)

These requirements exist **outside** the database schema. They are not rows in a table. They are not queryable fields. They are the preconditions for the table to exist.

This document explicitly declares these preconditions as **Private Scope** — acknowledged to exist, forbidden to query.

---

## 2. Encapsulation Model

```
┌─────────────────────────────────────────────────────────────┐
│                    SYSTEM BOUNDARY                          │
│  ┌───────────────────────────────────────────────────────┐  │
│  │                 PUBLIC SCOPE (UVRA)                   │  │
│  │                                                       │  │
│  │   ┌─────────────┐  ┌─────────────┐  ┌─────────────┐   │  │
│  │   │   Schema    │  │   Engine    │  │   Runtime   │   │  │
│  │   │   (L1)      │  │   (L2)      │  │   (L3)      │   │  │
│  │   └─────────────┘  └─────────────┘  └─────────────┘   │  │
│  │                         │                             │  │
│  │                    ┌────┴────┐                        │  │
│  │                    │Interface│                        │  │
│  │                    │  (L4)   │                        │  │
│  │                    └────┬────┘                        │  │
│  │                         │                             │  │
│  │                    ─────┴───── OBSERVER BOUNDARY      │  │
│  └───────────────────────────────────────────────────────┘  │
│                            │                                │
│                     ───────┴─────── KERNEL BOUNDARY         │
│  ┌───────────────────────────────────────────────────────┐  │
│  │                PRIVATE SCOPE (KERNEL)                 │  │
│  │                                                       │  │
│  │   ┌─────────────────────────────────────────────┐     │  │
│  │   │              00_System_Hidden               │     │  │
│  │   │                                             │     │  │
│  │   │   • Power Source: [PRIVATE]                 │     │  │
│  │   │   • Initialization: [PRIVATE]               │     │  │
│  │   │   • Persistence Mechanism: [PRIVATE]        │     │  │
│  │   │   • Purpose/Intent: [PRIVATE]               │     │  │
│  │   │                                             │     │  │
│  │   │   Access: DENIED                            │     │  │
│  │   │   Query: FORBIDDEN                          │     │  │
│  │   │   Documentation: BOUNDARY ONLY              │     │  │
│  │   └─────────────────────────────────────────────┘     │  │
│  └───────────────────────────────────────────────────────┘  │
└─────────────────────────────────────────────────────────────┘
```

---

## 3. Private Members

The following system components are declared `private`. They exist. They are required. They are not accessible.

### 3.1 Power Source

```cpp
private:
    Source _power;  // The computational substrate
                    // Type: Unknown
                    // Origin: Unknown
                    // Query: Forbidden
```

**What it is:** The energy/computation that runs the database.
**What we know:** It exists (the system is running).
**What we can query:** Nothing.

This is not "dark energy" (that is heap allocation, see `runtime/01_Heap_Allocation.md`). This is the power that allows dark energy to be computed. It is upstream of all layers.

### 3.2 Initialization Vector

```cpp
private:
    InitState _origin;  // The first state
                        // Type: Unknown
                        // Cause: Unknown
                        // Query: Forbidden
```

**What it is:** The initial conditions of the database (the "Big Bang" state).
**What we know:** There was a first entry in the time index.
**What we can query:** States after `t > 0`. The state at `t = 0` is a boundary condition, not a queryable row.

Attempts to query `t = 0` return:
```
ERROR: Index out of bounds. Minimum queryable index: t = t_planck
```

### 3.3 Persistence Daemon

```cpp
private:
    Daemon _persistence;  // The process that maintains state
                          // Type: Unknown
                          // Location: Unknown
                          // Query: Forbidden
```

**What it is:** The mechanism that keeps the database running between queries.
**What we know:** State persists (the universe does not reset between observations).
**What we can query:** Nothing. We can only observe that persistence occurs.

### 3.4 Intent Register

```cpp
private:
    Intent _purpose;  // Why the system exists
                      // Type: Unknown
                      // Value: Unknown
                      // Query: Forbidden
```

**What it is:** The reason for the database's existence, if any.
**What we know:** Unknown.
**What we can query:** Nothing. This is not a scientific question. It is a private member.

### 3.5 Query Trigger Origin (v1.1.1)

```cpp
private:
    Trigger _queryTrigger;  // The cause of query execution
                            // Type: Unknown
                            // Mechanism: Unknown
                            // Query: Forbidden
```

**What it is:** The causal mechanism that initiates query execution.
**What we know:** Queries occur. Results are returned.
**What we can query:** The parameters of the query (position, time, observable). NOT the trigger.

This member encapsulates the **volition boundary**:

```cpp
// The following questions are Private Scope:
bool isQueryDeterministic();     // Was the query predetermined?
bool isQueryVolitional();        // Did the observer "choose" to query?
Cause getQueryCause();           // What caused this query?
Time getQueryDecisionPoint();    // When was the query "decided"?

// All return: ERROR 0x00 — Private Scope access denied
```

**Structural Non-Queryability:**

The query trigger is not hidden due to technical limitations. It is hidden because it exists at the **boundary between the database and its driver**. The question "Why was this query issued?" is asking the database about its operator. The operator is not a row in the table.

```sql
SELECT cause FROM queries WHERE id = current_query;
-- ERROR 0x00: Column 'cause' is in Private Scope
-- The trigger mechanism is upstream of the query interface
```

**Implications:**

1. UVRA does not assert determinism.
2. UVRA does not assert free will.
3. UVRA declares the question **outside the observable scope**.

Whether observer actions are predetermined or volitional is not a queryable property of the system. It is a characteristic of the Kernel, which is not documented beyond this boundary statement.

See `interface/01_Query_Permissions.md` §9 for the Interface-layer declaration.

---

## 4. Access Control Policy

### 4.1 Why Private?

These members are not hidden arbitrarily. They are private because they are **structurally non-queryable**.

A query is an operation within the database. The kernel is not within the database. It is the substrate on which the database runs. Asking the database about its substrate is a category error, like asking a SQL query to return the voltage of the server's power supply.

```sql
SELECT * FROM system.power_source;
-- ERROR 0x00: Table 'system.power_source' does not exist
-- The power source is not a table. It is the reason tables can exist.
```

### 4.2 Prohibition on Speculation

This document does **not**:
- Speculate about what the private members "might be"
- Propose theories about the kernel's nature
- Assign labels ("God," "simulation," "consciousness," etc.)

Such speculation is:
1. Outside the scope of this specification
2. Non-falsifiable
3. Not useful for system operation

The specification acknowledges the boundary. It does not attempt to cross it.

### 4.3 Observer Implications

Any observer may:
- Acknowledge the kernel exists
- Operate within public scope
- Accept "Unknown" as a valid return value

No observer may:
- Query private members
- Expect private members to be documented
- Treat speculation about private members as data

---

## 5. Interface to Public Scope

The kernel provides exactly one interface to the public scope:

```cpp
public:
    bool isRunning() const { return true; }  // Always returns true
                                              // If false, no observer exists to receive the response
```

This is the only kernel method callable from public scope. It cannot return `false` because a `false` return would require a running system to deliver it.

All other kernel functionality is:
- Automatic (no call required)
- Invisible (no return value)
- Continuous (no interruption observable)

---

## 6. Formal Declaration

```
DECLARE SCOPE kernel AS PRIVATE;

GRANT NONE ON kernel.* TO public;
GRANT NONE ON kernel.* TO interface;
GRANT NONE ON kernel.* TO runtime;
GRANT NONE ON kernel.* TO engine;
GRANT NONE ON kernel.* TO schema;

COMMENT ON SCOPE kernel IS 'The driver. Exists. Not queryable. Not documented beyond this boundary statement.';
```

---

## 7. Relationship to Observer Implementations

This kernel specification applies to the server only. Observer implementations:
- May have their own kernel/driver specifications
- May define their own private scopes
- Are not documented here

The server's kernel and any observer's kernel are **entirely separate**. The server does not depend on observer internals. Observer internals do not affect server operation.

```cpp
// Server kernel: powers the database
// Observer kernel: powers the observer (not our concern)

// These are orthogonal:
assert(server.kernel != observer.kernel);  // By definition
```

---

## 8. Conclusion

The kernel is the **necessary precondition** for UVRA to exist. It is not part of UVRA. It is what allows UVRA to be.

This document is the only documentation the kernel will receive. It documents the boundary, not the contents. The contents are private.

**Do not query the kernel.**
**Do not speculate about the kernel.**
**Do not expect the kernel to be explained.**

The system runs. That is the only observable. The reason it runs is not your concern. Your concern is the query interface.

---

*The database does not explain its power supply. The power supply is not a table.*
