**bookclerk_db_exec**

# Module: bookclerk_db_exec

## Contents

**Modules**

- [`guest_receipt`](#guest_receipt) - Guest-typed durable receipt finalize (same-transaction payload capture).
- [`proxy_txn`](#proxy_txn) - Fail-closed acknowledgements for SeaORM's infallible proxy transaction hooks.
- [`sql_v1`](#sql_v1) - Shared Bookclerk SQL v1 execution vectors (binding DDL + portable helpers).

**Functions**

- [`set_positional_result_columns`](#set_positional_result_columns) - Records engine column metadata for the in-flight query (SQLite rusqlite path).
- [`take_positional_result_columns`](#take_positional_result_columns) - Takes engine column metadata recorded by the SQLite adapter, if any.

---

## Module: guest_receipt

Guest-typed durable receipt finalize (same-transaction payload capture).

Host wrap lives in `bookclerk-library`; adapters run
[`guest_receipt_finalize_stmts`](guest_receipt.md#bookclerk_db_execguest_receiptguest_receipt_finalize_stmts) in the same atomic transaction before COMMIT.



## Module: proxy_txn

Fail-closed acknowledgements for SeaORM's infallible proxy transaction hooks.

`ProxyDatabaseTrait::{begin, commit, rollback}` return `()`. A failed
`BEGIN` that is only logged leaves later statements in autocommit; a failed
`COMMIT` still looks like success to `sea_orm::DatabaseTransaction::commit`.
Proxies record a per-task fault instead and refuse subsequent statements.



## bookclerk_db_exec::set_positional_result_columns

*Function*

Records engine column metadata for the in-flight query (SQLite rusqlite path).

Prefers the request-scoped [`ExecBudget`](proxy_txn.md#bookclerk_db_execproxy_txnexecbudget) so metadata survives `spawn_blocking`.

```rust
fn set_positional_result_columns(columns: Vec<bookclerk_plugin_abi::DbColumn>)
```



## Module: sql_v1

Shared Bookclerk SQL v1 execution vectors (binding DDL + portable helpers).

Hosts emit these canonical statements. Adapters lower at execute: every
backend rewrites `INSERT OR IGNORE` to unique/PK `ON CONFLICT DO NOTHING`;
Postgres also applies binding DDL type rewrites and helper lowering.
Every name in `GuestSqlPolicy::binding_owned` portable functions
(except denied `hex`) appears here.



## bookclerk_db_exec::take_positional_result_columns

*Function*

Takes engine column metadata recorded by the SQLite adapter, if any.

```rust
fn take_positional_result_columns() -> Option<Vec<bookclerk_plugin_abi::DbColumn>>
```



