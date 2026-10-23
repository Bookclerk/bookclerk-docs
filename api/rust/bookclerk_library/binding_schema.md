**bookclerk_library > binding_schema**

# Module: binding_schema

## Contents

**Functions**

- [`apply_binding_bootstrap`](#apply_binding_bootstrap) - Applies host-owned binding bootstrap when the binding is uninitialized.
- [`binding_bootstrap_plan`](#binding_bootstrap_plan) - Statements to apply for `state`, or `None` when the binding already matches.

---

## bookclerk_library::binding_schema::apply_binding_bootstrap

*Function*

Applies host-owned binding bootstrap when the binding is uninitialized.

Matching `SchemaState::Unreleased` is a no-op. Restore that rewrites
`bookclerk_schema_migrations` as ordinary rows will take this no-op path and will not
re-run plugin-owned DDL.

# Errors

Returns when schema apply or the binding state machine fails closed.

```rust
fn apply_binding_bootstrap(db: &sea_orm::DatabaseConnection) -> crate::error::Result<()>
```



## bookclerk_library::binding_schema::binding_bootstrap_plan

*Function*

Statements to apply for `state`, or `None` when the binding already matches.

# Errors

Returns when `state` is a checksum/base mismatch or an unexpected frozen
binding (this binary has no frozen binding plan).

```rust
fn binding_bootstrap_plan(state: &crate::schema_state::SchemaState) -> crate::error::Result<Option<Vec<String>>>
```



