**bookclerk_library > sql_plan > named**

# Module: sql_plan::named

## Contents

**Functions**

- [`compile_claim_event_delivery`](#compile_claim_event_delivery) - Compiles a host-side CAS claim for one pending `event_deliveries` row.
- [`compile_named_request`](#compile_named_request) - Compiles a named atomic request into canonical Bookclerk SQL.

---

## bookclerk_library::sql_plan::named::compile_claim_event_delivery

*Function*

Compiles a host-side CAS claim for one pending `event_deliveries` row.

# Errors

Returns `DbErr::Custom` when wrapping the plan fails.

```rust
fn compile_claim_event_delivery(operation_id: &str, delivery_id: &str, owner: &str, lease_secs: i64, plugin_id: &str, resource_class: &str, max_in_flight: i64, now: &str) -> std::result::Result<super::CompiledAtomic, sea_orm::DbErr>
```



## bookclerk_library::sql_plan::named::compile_named_request

*Function*

Compiles a named atomic request into canonical Bookclerk SQL.

Adapters lower placeholders and functions at execute time.

# Errors

Returns `DbErr::Custom` when the named operation is missing or invalid.

```rust
fn compile_named_request(operation_id: &str, params: &crate::atomic_ops::DbAtomicParams, now: &str) -> std::result::Result<super::CompiledAtomic, sea_orm::DbErr>
```



