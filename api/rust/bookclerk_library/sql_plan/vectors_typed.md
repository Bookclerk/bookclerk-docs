**bookclerk_library > sql_plan > vectors_typed**

# Module: sql_plan::vectors_typed

## Contents

**Functions**

- [`run_typed_conn_vectors`](#run_typed_conn_vectors) - Runs the shared contract suite through the host stamp + adapter execute path.
- [`run_typed_contract_vectors`](#run_typed_contract_vectors) - Native typed contract suite (`ExecuteRequest` → `ExecuteReply`).
- [`run_typed_request_vectors`](#run_typed_request_vectors) - Runs the shared contract suite through a typed `ExecuteRequest` callback.
- [`stamp_typed_vector`](#stamp_typed_vector) - Desugars and stamps proofs for one vector, recording DDL in `catalog`.

---

## bookclerk_library::sql_plan::vectors_typed::run_typed_conn_vectors

*Function*

Runs the shared contract suite through the host stamp + adapter execute path.

# Panics

Panics when a vector fails.

```rust
fn run_typed_conn_vectors(db: &sea_orm::DatabaseConnection, connect: bookclerk_plugin_abi::DbCapabilities)
```



## bookclerk_library::sql_plan::vectors_typed::run_typed_contract_vectors

*Function*

Native typed contract suite (`ExecuteRequest` → `ExecuteReply`).

Every reply is validated against its request before the vector's own
assertions run.

```rust
fn run_typed_contract_vectors<F, Fut>(connect: bookclerk_plugin_abi::DbCapabilities, row_cap: u32, run: F)
```



## bookclerk_library::sql_plan::vectors_typed::run_typed_request_vectors

*Function*

Runs the shared contract suite through a typed `ExecuteRequest` callback.

`advertised_cap` is the adapter's negotiated `maxResultRows`. The harness does
not wrap SQL or post-reject oversized results.

# Panics

Panics when a vector fails.

```rust
fn run_typed_request_vectors<F, Fut, E>(connect: bookclerk_plugin_abi::DbCapabilities, advertised_cap: u32, run: F)
```



## bookclerk_library::sql_plan::vectors_typed::stamp_typed_vector

*Function*

Desugars and stamps proofs for one vector, recording DDL in `catalog`.

Adapter-boundary tests (guest execute) must stamp against the host catalog
plus tables created by earlier vectors.

# Errors

Returns when typecheck fails.

```rust
fn stamp_typed_vector(req: bookclerk_plugin_abi::ExecuteRequest, catalog: & mut bookclerk_plugin_abi::SqlTypeEnv) -> Result<bookclerk_plugin_abi::AdapterExecuteRequest, String>
```



