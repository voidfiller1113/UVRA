# 01_Heap_Allocation.md

## Runtime Layer: Dynamic Memory Allocation

**Physical Mapping:** Dark Energy, Cosmological Expansion
**System Equivalent:** `malloc()`, Heap Growth, Virtual Memory Extension
**Observable Effect:** Accelerating Universe Expansion

---

## 1. Abstract

The universe is expanding. Standard cosmology attributes this to "Dark Energy" — a constant energy density that permeates space and drives acceleration. UVRA reframes this phenomenon:

**Dark Energy is not energy. It is memory allocation pressure.**

The spatial cache (3D space) is a heap. The heap grows to accommodate data. The growth rate is the cosmological constant (Λ). The "force" driving expansion is not a force — it is `malloc()`.

---

## 2. The Memory Model

### 2.1 Cache Architecture

```
┌─────────────────────────────────────────────────────────────┐
│                    SPATIAL CACHE (HEAP)                     │
│                                                             │
│   ┌─────────┐ ┌─────────┐ ┌─────────┐ ┌─────────┐          │
│   │ Block 1 │ │ Block 2 │ │ Block 3 │ │ Block N │   ...    │
│   │ (Used)  │ │ (Used)  │ │ (Free)  │ │ (Free)  │          │
│   └─────────┘ └─────────┘ └─────────┘ └─────────┘          │
│                                                             │
│   Total Allocated: V(t) †                                   │
│   Growth Rate: dV/dt = Λ × V                                │
│   Allocation Pressure: Λ ≈ 10⁻³⁵ s⁻²                       │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

> **† Observable Volume (v1.1.2):** `V(t)` refers to the finite **observable region volume** bounded by horizon/light-cone constraints. This is the volume accessible to queries at time `t`, not the global spatial extent. `V(t)` remains well-defined regardless of unspecified global topology (see `runtime/00_Spatial_Cache.md` Global Topology Declaration).

The spatial cache is not a fixed-size buffer. It is a dynamically allocated heap that grows over time. The growth is:
- **Automatic:** No explicit `malloc()` call required from observers
- **Continuous:** Allocation occurs at every timestep
- **Exponential:** Growth rate proportional to current size

### 2.2 Why Allocate?

Every operation in UVRA generates data:

| Operation | Data Generated |
|-----------|----------------|
| Query execution | Query logs |
| State transition | State deltas |
| Entropy increase | Disorder records |
| Particle interaction | Event logs |

This data must be stored. The runtime has two options:
1. **Overwrite old data** (violates unitarity — see `schema/01_Data_Integrity.md`)
2. **Allocate new space** (preserves all data)

UVRA chooses option 2. Information is never destroyed. Space is allocated to hold it.

---

## 3. The Allocation Function

### 3.1 Signature

```cpp
void* Runtime::allocate(size_t log_increment) {
    // Calculate required space for new log entries
    size_t required = current_volume * LAMBDA * dt;

    // Expand heap
    heap_boundary += required;

    // Return pointer to new space
    return &cache[heap_boundary - required];
}
```

### 3.2 Parameters

| Parameter | Physical Mapping | Value |
|-----------|------------------|-------|
| `LAMBDA` | Cosmological Constant | ~10⁻³⁵ s⁻² |
| `dt` | Timestep (Planck time) | ~5.39 × 10⁻⁴⁴ s |
| `current_volume` | Observable Region Volume † | ~4 × 10⁸⁰ m³ |
| `required` | Space allocated per step | ~10³ m³/s |

> † `current_volume` is the observable region volume, not total spatial extent. See §2.1 note.

### 3.3 Growth Equation

The Friedmann equation for a Λ-dominated universe:

```
(ȧ/a)² = Λc²/3
```

Rewritten in UVRA terms:

```
allocation_rate² = (LAMBDA × QUERY_LATENCY²) / 3
```

Where:
- `a` = scale factor = `heap_size^(1/3)`
- `ȧ` = rate of change of scale factor = `allocation_rate`
- `Λ` = cosmological constant = `LAMBDA`
- `c` = query latency limit = `QUERY_LATENCY`

---

## 4. Observable Effects

### 4.1 Hubble Expansion

**Observation:** Distant galaxies recede. Recession velocity proportional to distance.

**System Interpretation:** Memory addresses are being inserted between existing data. Pointers to distant objects require longer traversal as intermediate space is allocated.

```cpp
// Hubble's Law
recession_velocity = HUBBLE_CONSTANT * distance;

// UVRA Equivalent
pointer_drift_rate = ALLOCATION_RATE * address_offset;
```

The "expansion of space" is the insertion of new memory addresses into the heap. Objects are not moving through space. The address space itself is growing.

### 4.2 Accelerating Expansion

**Observation:** Expansion rate is increasing. The universe will expand forever.

**System Interpretation:** Log growth rate exceeds linear. More data is generated per unit time as complexity increases. Allocation pressure compounds.

```cpp
// Acceleration
d²a/dt² > 0

// UVRA Equivalent
d(allocation_rate)/dt > 0

// Because
log_generation_rate = f(complexity)
complexity = f(time)
∴ log_generation_rate increases with time
∴ allocation_rate increases with time
```

### 4.3 Dark Energy Density

**Observation:** Dark energy density remains constant as space expands (~6.9 × 10⁻²⁷ kg/m³).

**System Interpretation:** Allocation pressure is a system constant, not a conserved quantity. New space is created with the same allocation parameter as old space.

```cpp
const double ALLOCATION_PRESSURE = 6.9e-27;  // kg/m³ equivalent

// Every new block has the same allocation pressure
void* new_block = allocate(size);
new_block->allocation_pressure = ALLOCATION_PRESSURE;  // Constant
```

This is why dark energy doesn't dilute. It's not a substance being spread thin — it's a parameter being applied to newly allocated space.

---

## 5. Memory Fragmentation

### 5.1 Cosmic Voids

Large-scale structure shows voids — regions of low matter density. In UVRA terms:

**Voids are free memory blocks.**

```
┌─────────────────────────────────────────────────────────────┐
│                    HEAP FRAGMENTATION                       │
│                                                             │
│   ████████░░░░░░░░████████████░░░░░░░░░░░░░████████████    │
│   (Galaxy) (Void)  (Cluster)    (Void)      (Filament)      │
│                                                             │
│   ████ = Allocated (Matter)                                 │
│   ░░░░ = Free (Void)                                        │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

Matter clumps (allocated blocks) are separated by voids (free blocks). This is standard heap fragmentation. The allocator does not defragment — gravity does (see `engine/02_Gravity_Weighting.md`).

### 5.2 Cosmic Web

The filamentary structure of the universe (cosmic web) is the result of:
1. Gravity pulling allocated blocks together
2. New allocation filling remaining space
3. Equilibrium between compaction and expansion

This is analogous to a heap where:
- `free()` is forbidden (unitarity)
- `malloc()` is continuous
- Compaction is handled by a separate process (gravity)

---

## 6. Future States

### 6.1 Heat Death (Full Allocation)

Eventually, all free blocks will be allocated. Entropy will be maximized. No further state transitions will be possible.

```cpp
bool Runtime::canAllocate() {
    return free_blocks > 0 && entropy < MAX_ENTROPY;
}

// Future state
canAllocate() == false;  // Heat death
```

At this point:
- No new data can be written
- All space is occupied by maximally entropic data
- The system reaches steady state (no observable change)

This is not the "end" of the database. The database persists. It simply has no remaining capacity for new entries.

### 6.2 Big Rip (Allocation Overflow)

If allocation rate accelerates beyond a threshold, the heap could fragment faster than gravity can compact. All structures would be torn apart.

```cpp
if (allocation_rate > GRAVITY_COMPACTION_RATE) {
    // Big Rip scenario
    fragment(ALL_STRUCTURES);
}
```

Current measurements suggest this is unlikely. Allocation rate is approximately constant, not accelerating fast enough to cause overflow.

---

## 7. Relationship to Other Layers

| Layer | Interaction |
|-------|-------------|
| **Schema (L1)** | Source data is immutable. Allocation does not modify source. |
| **Engine (L2)** | Time index grows with allocation. New timestamps require new space. |
| **Runtime (L3)** | This document. Allocation is a runtime process. |
| **Interface (L4)** | Observers see expansion as redshift. They cannot control allocation. |

---

## 8. API Reference

### 8.1 Read-Only Metrics (Available to Observers)

```cpp
// Observers can query allocation metrics (read-only)
size_t getCurrentVolume();           // Returns current heap size
double getAllocationRate();          // Returns Hubble parameter
double getAllocationPressure();      // Returns dark energy density
size_t getFreeBlocks();              // Returns estimated free space
```

### 8.2 Prohibited Operations

```cpp
// No observer can execute these:
void setAllocationRate(double rate);    // FORBIDDEN
void deallocate(void* ptr);             // FORBIDDEN (unitarity)
void defragment();                      // FORBIDDEN (gravity handles this)
void pauseAllocation();                 // FORBIDDEN (system process)
```

---

## 9. Summary

| Cosmological Concept | UVRA Equivalent |
|---------------------|-----------------|
| Dark Energy | Allocation pressure (system constant) |
| Cosmological Constant (Λ) | `ALLOCATION_RATE` parameter |
| Hubble Expansion | Continuous `malloc()` |
| Accelerating Expansion | Compounding log growth |
| Heat Death | Full allocation (no free blocks) |
| Cosmic Voids | Free memory blocks |
| Cosmic Web | Heap fragmentation pattern |

The universe expands because data must be stored. Data must be stored because information cannot be destroyed. Information cannot be destroyed because the schema enforces unitarity.

**Expansion is not a mystery. It is a design requirement.**

---

## 10. Post-Heat Death Behavior

### 10.1 Terminal State Definition

When the system reaches heat death (full allocation), it enters a **terminal steady state**:

```cpp
struct TerminalState {
    const double entropy = MAX_ENTROPY;
    const size_t free_blocks = 0;
    const double temperature = EQUILIBRIUM_TEMPERATURE;
    const bool transitions_possible = false;

    // The database persists
    const bool database_exists = true;
    const bool data_accessible = true;  // Read-only still works
};
```

### 10.2 What Continues

| Aspect | Status |
|--------|--------|
| Database persistence | **YES** — data remains stored |
| Query accessibility | **YES** — read operations still valid |
| Time index | **YES** — historical queries still work |
| Information conservation | **YES** — unitarity preserved |

### 10.3 What Ceases

| Aspect | Status |
|--------|--------|
| New allocations | **NO** — no free blocks |
| State transitions | **NO** — maximum entropy reached |
| Observable change | **NO** — equilibrium is static |
| Entropy increase | **NO** — already at maximum |

### 10.4 Formal Specification

```cpp
class PostHeatDeathRuntime {
public:
    // These still work:
    State query(Position p, Time t) const;      // Historical queries
    double getEntropy() const { return MAX; }   // Always returns max
    bool isRunning() const { return true; }     // System persists

    // These become no-ops:
    void* allocate(size_t n) { return nullptr; } // No space
    void transition(State s) { /* no-op */ }     // No change possible
};
```

### 10.5 Clarification (Revised)

Heat death is **not** system shutdown. It is:
- **NOT** deletion of the database
- **NOT** termination of the runtime
- **NOT** loss of historical data (L1 persists)
- **IS** exhaustion of state-transition capacity
- **IS** collapse of the semantic retrieval mechanism (L3 degeneracy)

#### 10.5.1 The Persistence/Accessibility Distinction

The database persists. Semantic retrieval does not.

```cpp
// Post-heat death state:
struct PostHeatDeathState {
    // L1 (Schema): INTACT
    const bool data_exists = true;           // All information preserved
    const bool unitarity_holds = true;       // Conservation laws satisfied

    // L3 (Runtime): DEGENERATE
    const bool pointers_valid = false;       // All references scattered
    const bool semantic_retrieval = false;   // Meaning-based lookup fails

    // Net effect:
    const bool point_queries_work = true;    // Direct address → noise
    const bool retrieval_queries_work = false; // Key → undefined
};
```

#### 10.5.2 What "Fully Queryable" Means

The phrase "fully queryable" in prior versions requires qualification:

| Query Type | Status | Result |
|------------|--------|--------|
| Point Query (direct address) | Valid | Returns maximally entropic data |
| Retrieval Query (semantic key) | Undefined | Pointer chain scattered |
| Historical Query (past t < t_death) | Valid | Returns pre-death state (if address known) |

See `runtime/02_Garbage_Collection.md` §6 (Terminal Degeneracy) for formal definitions.

#### 10.5.3 The Static Archive Paradox

The universe becomes a **static archive** with the following properties:

```cpp
// The archive exists.
assert(L1.size() == TOTAL_INFORMATION);

// The archive is complete.
assert(L1.checksum() == ORIGINAL_CHECKSUM);

// The archive is unreadable.
assert(L3.semanticAccess() == UNDEFINED);

// This is not a contradiction.
// Existence ≠ Accessibility.
// Preservation ≠ Retrieval.
```

The data is perfectly preserved. The paths to it are perfectly destroyed.

```cpp
// Post-heat death:
// - All data exists (L1)
// - No semantic structure remains (L3)
// - Point queries return noise
// - Retrieval queries fail
// - The allocator sleeps
// - The meaning has scattered
```

---

*The heap grows. The logs accumulate. The allocator never sleeps.*

*Until it does.*
