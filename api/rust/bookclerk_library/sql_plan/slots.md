**bookclerk_library > sql_plan > slots**

# Module: sql_plan::slots

## Contents

**Functions**

- [`event_inflight_slot`](#event_inflight_slot) - Slot key that serializes in-flight event deliveries for one plugin class.
- [`lock_serialization_slot`](#lock_serialization_slot) - Inserts the slot row if missing, then bumps it (transaction-scoped write lock).

**Constants**

- [`JOB_QUEUE_SLOT`](#job_queue_slot) - Slot key for job admit / claim / scratch-quota serialization.

---

## bookclerk_library::sql_plan::slots::JOB_QUEUE_SLOT

*Constant*: `&str`

Slot key for job admit / claim / scratch-quota serialization.



## bookclerk_library::sql_plan::slots::event_inflight_slot

*Function*

Slot key that serializes in-flight event deliveries for one plugin class.

```rust
fn event_inflight_slot(plugin_id: &str, resource_class: &str) -> String
```



## bookclerk_library::sql_plan::slots::lock_serialization_slot

*Function*

Inserts the slot row if missing, then bumps it (transaction-scoped write lock).

Canonical SQL uses `?` placeholders. Host transport never lowers; adapters
realize `INSERT OR IGNORE` at the execute edge. This helper must run on the
caller's transaction, not a nested `BEGIN`. `in_process` of `false` is
canonical leftover SQL (production proxy). `true` physically lowers once
on the opened adapter connection.

# Errors

Returns `crate::LibraryError::Orm` when either statement fails.

```rust
fn lock_serialization_slot<C>(db: &C, in_process: bool, slot_key: &str) -> crate::error::Result<()>
```



