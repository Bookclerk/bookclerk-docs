**bookclerk_db_guest > host_session**

# Module: host_session

## Contents

**Structs**

- [`BoundGuestHostAdapterSession`](#boundguesthostadaptersession) - Host-private envelope session bound to one dedicated connection (named bindings).
- [`GuestHostAdapterSession`](#guesthostadaptersession) - In-process SeaORM transaction bridge for host `begin` RPC (shared library connection).

**Functions**

- [`host_session`](#host_session) - Returns the default host adapter session on the shared guest connection.
- [`host_session_on`](#host_session_on) - Host envelope session that persists guest receipts on `conn`.

---

## bookclerk_db_guest::host_session::BoundGuestHostAdapterSession

*Struct*

Host-private envelope session bound to one dedicated connection (named bindings).

**Trait Implementations:**

- **HostAdapterDatabaseSession**
  - `fn begin(self: &'life0 Self, isolation: IsolationReq) -> ::core::pin::Pin<Box<dyn ::core::future::Future>>`
  - `fn execute(self: &'life0 Self, request: AdapterExecuteRequest) -> ::core::pin::Pin<Box<dyn ::core::future::Future>>`



## bookclerk_db_guest::host_session::GuestHostAdapterSession

*Struct*

In-process SeaORM transaction bridge for host `begin` RPC (shared library connection).

**Unit Struct**

**Trait Implementations:**

- **HostAdapterDatabaseSession**
  - `fn begin(self: &'life0 Self, isolation: IsolationReq) -> ::core::pin::Pin<Box<dyn ::core::future::Future>>`
  - `fn execute(self: &'life0 Self, request: AdapterExecuteRequest) -> ::core::pin::Pin<Box<dyn ::core::future::Future>>`



## bookclerk_db_guest::host_session::host_session

*Function*

Returns the default host adapter session on the shared guest connection.

```rust
fn host_session() -> GuestHostAdapterSession
```



## bookclerk_db_guest::host_session::host_session_on

*Function*

Host envelope session that persists guest receipts on `conn`.

```rust
fn host_session_on(conn: sea_orm::DatabaseConnection) -> BoundGuestHostAdapterSession
```



