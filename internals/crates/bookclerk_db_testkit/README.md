# bookclerk_db_testkit

Contract conformance test kit for code that talks to a Bookclerk database
plugin.

Production hosts reach every database through an external adapter guest
that refuses interactive transactions and enforces the limits it
advertised. A plain in-memory SQLite connection does neither, so a test
that passes on one can still fail on the hosted demo. This kit opens an
in-memory library whose connection behaves like the guest:

- `begin()` on the connection is refused the way the adapter refuses it,
  and the refusal is counted ([`TestDatabase::refused_begins`](#testdatabaserefused_begins)).
- Every host batch goes through an executor that applies the
  [`Profile`](../bookclerk_library/test_support.md#profile)'s capability advertisement on the request and on the reply,
  answering a typed `limit_exceeded` error.
- Lost replies and lock contention can be injected
  ([`TestDatabase::lose_next_replies`](#testdatabaselose_next_replies), [`TestDatabase::lock_next_requests`](#testdatabaselock_next_requests)).

[`fixtures`](fixtures.md) generates realistic rows near the caps, and [`paging`](paging.md) checks
that a paged listing returns what the full listing returns. Ordinary
SeaORM reads through [`TestDatabase::connection`](#testdatabaseconnection) are held to the engine
proxy's first-party caps, as on the shipped sqlite adapter.

## Re-exports

- [`binding::BindingDatabase`](binding.md#bindingdatabase)
- [`bookclerk_library::test_support::Profile`](../bookclerk_library/test_support.md#profile)
- [`bookclerk_library::test_support::INTERACTIVE_TRANSACTIONS_REFUSED`](../bookclerk_db_exec/begin_gate.md#interactive_transactions_refused)

## Modules

- [`binding`](binding.md): A plugin-owned database that behaves like a granted `[[databases]]` binding on a deployed host.
- [`fixtures`](fixtures.md): Realistic row contents sized against a capability advertisement.
- [`paging`](paging.md): Checks that a paged listing returns what the full listing returns.

## Structs

### TestDatabase

```rust
pub struct TestDatabase {
    /* private fields */
}
```

An in-memory library that behaves like an external database plugin.

Open one with [`TestDatabase::open`](#testdatabaseopen) and run the code under test against
[`TestDatabase::store`](#testdatabasestore).

#### TestDatabase::open

```rust
pub async fn open(profile: Profile) -> Result<Self>
```

Opens an empty in-memory library with the host schema under `profile`.

##### Errors

Returns the engine or schema error when the database cannot be opened.

#### TestDatabase::open_all

```rust
pub async fn open_all() -> Result<Vec<Self>>
```

Opens one database per `Profile::ALL` entry.

##### Errors

Returns the first open error.

#### TestDatabase::profile

```rust
pub fn profile(&self) -> Profile
```

The enforced profile.

#### TestDatabase::capabilities

```rust
pub fn capabilities(&self) -> &DbCapabilities
```

The enforced capability advertisement.

#### TestDatabase::store

```rust
pub fn store(&self) -> &LibraryStore
```

The library store under test.

#### TestDatabase::connection

```rust
pub fn connection(&self) -> &DatabaseConnection
```

The begin-refusing connection, for reads a test makes directly.

#### TestDatabase::refused_begins

```rust
pub fn refused_begins(&self) -> u32
```

How many interactive transactions the code under test tried to open.

A host write that works on an external adapter leaves this at zero.

#### TestDatabase::batch_sizes

```rust
pub fn batch_sizes(&self) -> Vec<usize>
```

Statement counts of every batch sent so far, in arrival order,
including batches a fault turned away.

#### TestDatabase::execute_batch

```rust
pub async fn execute_batch(
    &self,
    request: ExecuteRequest,
) -> Result<ExecuteReply, PluginError>
```

Runs one host-authored batch through the profile; see
`ProfileStore::execute_batch`.

##### Errors

Returns the typed `limit_exceeded` error when the request or the
reply exceeds a cap, `invalid_params` when the batch does not
typecheck against the host schema, and `unavailable` for an injected
fault or an engine failure.

#### TestDatabase::lose_next_replies

```rust
pub fn lose_next_replies(&self, n: u32)
```

The next `n` batches are applied but their replies are lost: the
caller sees `unavailable`, the way a dropped RPC reply looks, and a
retry with the same operation id must not repeat the write.

#### TestDatabase::lock_next_requests

```rust
pub fn lock_next_requests(&self, n: u32)
```

The next `n` batches are refused with SQLite lock contention before
anything is applied, the way a busy file lock answers.

Implements: `Debug`
