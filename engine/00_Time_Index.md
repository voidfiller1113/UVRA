# 00_Time_Index.md

## Engine Layer: Time as Primary Key

**Physical Mapping:** Temporal Dimension, Block Universe
**Index Type:** B-Tree (Ordered, Immutable)
**Key Format:** `t` (Real Number, Planck-Time Resolution)

---

## 1. Abstract

Time is not a dimension through which things "move." Time is the **primary key** of the universal database. Every record is indexed by its temporal coordinate. The universe is a pre-calculated dataset; time is the index you query against.

```sql
CREATE INDEX idx_time ON universe_state (t);

-- All queries implicitly filter by time
SELECT * FROM universe_state WHERE t = :observer_now;
```

---

## 2. The Block Universe Model

### 2.1 All Times Exist

The database contains records for all values of `t`:

```
t = -∞  ←────────────────────────────────────→  t = +∞
        │          │         │         │
     t = 0    t_planck   t = now   t = heat_death
        │          │         │         │
  [BOUNDARY]    [ROW]     [ROW]      [ROW]
  (private)   (min queryable)
```

> **Boundary Condition (v1.1.2):** `t = 0` exists as a private boundary condition but is **non-queryable**. The minimum queryable time index is `t = t_planck`. Queries for `t < t_planck` return `OUT_OF_BOUNDS` (see `03_Schema_Interface.md` §7.1).

Past, present, and future are not ontologically different. They are index ranges.

### 2.2 No "Flow" of Time

Time does not "flow." The observer's query cursor advances through the index:

```cpp
class Observer {
    double t_cursor;  // Current position in time index

    void advanceCursor(double dt) {
        t_cursor += dt;
        // The database doesn't change
        // The cursor moves
    }
};
```

> **Note:** `t_cursor` is Observer-local state (see `interface/01_Query_Permissions.md` §7.3). Advancing the cursor does not write to any UVRA layer.

---

## 3. Index Structure

### 3.1 B-Tree Organization

```
                      [Root: t_mid]
                     /             \
           [t_early]               [t_late]
           /       \               /       \
    [t_p..t_1] [t_1..t_2]   [t_2..t_3] [t_3..t_now]
         ↑
    t_p = t_planck (minimum queryable)
```

Properties:
- **Ordered:** Records sorted by `t`
- **Balanced:** O(log n) lookup
- **Immutable:** No rebalancing (append-only)
- **Bounded:** Index starts at `t_planck`, not `t = 0`

### 3.2 Granularity

```cpp
const double PLANCK_TIME = 5.39e-44;  // seconds
const double INDEX_RESOLUTION = PLANCK_TIME;

// Queries below this resolution return interpolated values
```

---

## 4. Query Semantics

### 4.1 Point Query

```sql
SELECT * FROM universe_state WHERE t = 13.8e9 * YEAR;
```

Returns the complete state of the universe at that timestamp.

### 4.2 Range Query

```sql
SELECT * FROM universe_state WHERE t BETWEEN t_start AND t_end;
```

Returns a sequence of states (the "history" of that interval).

### 4.3 Observer-Relative Query

```sql
SELECT * FROM universe_state
WHERE t = :observer_t - distance(P, position) / c;
```

Due to query latency (see `01_Causal_Latency.md`), observers see past states of distant objects.

---

## 5. Immutability

### 5.1 No Updates

```sql
-- FORBIDDEN
UPDATE universe_state SET field_value = :new WHERE t = :past_time;

-- ERROR: Time index is immutable.
```

### 5.2 Append-Only

New records are appended at the advancing edge of time. Only the system (not observers) can append.

---

## 6. Special Relativity as Index Transformation

### 6.1 Lorentz Transformation

Different observers use different index mappings:

```cpp
Time transformTime(Time t, Position x, Velocity v) {
    double gamma = 1.0 / sqrt(1 - v²/c²);
    return gamma * (t - v * x / c²);
}
```

### 6.2 Time Dilation

Moving observers traverse the index at different rates:

```cpp
double properTimeIncrement(Velocity v, double coordinate_dt) {
    double gamma = 1.0 / sqrt(1 - v²/c²);
    return coordinate_dt / gamma;
}
```

---

## 7. The Arrow of Time

The index has a natural direction (low entropy → high entropy):

```cpp
assert(entropy(t + dt) >= entropy(t));
```

This defines the "arrow" of time. It is a boundary condition, not a law.

---

## 8. Summary

| Physical Concept | UVRA Equivalent |
|------------------|-----------------|
| Time | Primary key index |
| Present | Current query cursor |
| Past | Records at `t_planck ≤ t < t_cursor` |
| Future | Records at `t > t_cursor` |
| t = 0 | Private boundary (non-queryable) |
| Time dilation | Cursor rate variation |
| Arrow of time | Low-entropy boundary condition |

---

*The database contains all moments. You are querying one at a time.*
