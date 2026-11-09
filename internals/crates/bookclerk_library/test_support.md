# bookclerk_library::test_support

A library store that behaves like one opened on an external database
plugin, for tests.

Production hosts reach every database through an adapter guest that
refuses interactive transactions and enforces the limits it advertised. A
plain engine connection does neither, so a test that passes on one can
still fail on the hosted demo. [`ProfileStore`](#profilestore) wraps an engine proxy in
a [`BeginGate`](../bookclerk_db_exec/begin_gate.md#begingate) and routes every host batch through a [`ProfileExec`](#profileexec)
that applies a [`Profile`](#profile)'s capability advertisement.

This module is the engine-neutral core; the `bookclerk-db-testkit` crate
opens it on in-memory SQLite and adds fixtures.

| Item | Kind | Summary |
| --- | --- | --- |
| [`ProfileExec`](#profileexec) | struct | Stands in for a database adapter's `executeAtomic`. |
| [`ProfileStore`](#profilestore) | struct | A library store opened on an engine proxy that behaves like an external database plugin under one `Profile`. |
| [`Profile`](#profile) | enum | A capability advertisement a database plugin may present to the host. |

## Re-exports

- [`bookclerk_db_exec::begin_gate::INTERACTIVE_TRANSACTIONS_REFUSED`](../bookclerk_db_exec/begin_gate.md#interactive_transactions_refused)

## Structs

### ProfileExec

```rust
pub struct ProfileExec {
    /* private fields */
}
```

Stands in for a database adapter's `executeAtomic`.

Each batch is admitted against the profile's capabilities the way the
adapter SDK admits it, run as one engine transaction under the profile's
result caps, and its reply checked the way the host checks a guest reply.
Injected faults are consumed one batch at a time.

#### ProfileExec::new

```rust
pub fn new(db: DatabaseConnection, caps: DbCapabilities) -> Self
```

Wraps `db` with `caps`.

#### ProfileExec::batch_sizes

```rust
pub fn batch_sizes(&self) -> Vec<usize>
```

Statement counts of every batch so far, including batches a fault
turned away.

#### ProfileExec::lose_next_replies

```rust
pub fn lose_next_replies(&self, n: u32)
```

Arms `n` lost replies: the batch is applied, the caller sees
`unavailable`.

#### ProfileExec::lock_next_requests

```rust
pub fn lock_next_requests(&self, n: u32)
```

Arms `n` lock-contention refusals: nothing is applied, the caller
sees `unavailable` naming `SQLITE_BUSY`.

Implements: `Debug`, `TypedAtomicExec`

### ProfileStore

```rust
pub struct ProfileStore {
    /* private fields */
}
```

A library store opened on an engine proxy that behaves like an external
database plugin under one [`Profile`](#profile).

#### ProfileStore::open

```rust
pub async fn open(
    engine: impl ProxyDatabaseTrait + 'static,
    backend: DbBackend,
    profile: Profile,
) -> Result<Self>
```

Opens `engine` behind a begin gate, applies the host schema and wires
a store that enforces `profile`.

`backend` is the SQL dialect the engine proxy speaks, which SeaORM
needs to render statements; the kit opens sqlite-shaped engines.

##### Errors

Returns the engine or schema error when the database cannot be opened.

#### ProfileStore::profile

```rust
pub fn profile(&self) -> Profile
```

The enforced profile.

#### ProfileStore::into_store

```rust
pub fn into_store(self) -> LibraryStore
```

The store alone, for fixtures that only need a store that behaves
like one on an external plugin. The gate keeps refusing `begin()`.

#### ProfileStore::capabilities

```rust
pub fn capabilities(&self) -> &DbCapabilities
```

The enforced capability advertisement.

#### ProfileStore::store

```rust
pub fn store(&self) -> &LibraryStore
```

The library store under test.

#### ProfileStore::connection

```rust
pub fn connection(&self) -> &DatabaseConnection
```

The begin-refusing connection, for reads a test makes directly.

#### ProfileStore::refused_begins

```rust
pub fn refused_begins(&self) -> u32
```

How many interactive transactions the code under test tried to open.

A host write that works on an external adapter leaves this at zero.

#### ProfileStore::batch_sizes

```rust
pub fn batch_sizes(&self) -> Vec<usize>
```

Statement counts of every batch sent so far, in arrival order,
including batches a fault turned away.

#### ProfileStore::lose_next_replies

```rust
pub fn lose_next_replies(&self, n: u32)
```

The next `n` batches are applied but their replies are lost: the
caller sees `unavailable`, the way a dropped RPC reply looks, and a
retry with the same operation id must not repeat the write.

#### ProfileStore::lock_next_requests

```rust
pub fn lock_next_requests(&self, n: u32)
```

The next `n` batches are refused with SQLite lock contention before
anything is applied, the way a busy file lock answers.

#### ProfileStore::execute_batch

```rust
pub async fn execute_batch(
    &self,
    request: ExecuteRequest,
) -> Result<ExecuteReply, PluginError>
```

Runs one host-authored batch through the profile the way a store
write does: stamped against the host schema, admitted against the
advertisement, run as one transaction, its reply checked.

##### Errors

Returns the typed `limit_exceeded` error when the request or the
reply exceeds a cap, `invalid_params` when the batch does not
typecheck against the host schema, and `unavailable` for an injected
fault or an engine failure.

Implements: `Debug`

## Enums

### Profile

```rust
pub enum Profile {
    FirstParty,
    D1,
    Minimum,
}
```

A capability advertisement a database plugin may present to the host.

Host and plugin database code is expected to pass under every profile:
what works on the shipped adapters must also work when a plugin
advertises only the host minimums.

**Variants**

- `FirstParty`: The shipped sqlite and Postgres adapters: 256 KiB inline results,
  streamed reads, the first-party statement and bind caps.
- `D1`: The Cloudflare D1 adapter: D1's bind, payload and lowered-SQL caps,
  no consistent backup read, 8 MiB streamed results.
- `Minimum`: The smallest advertisement the host connects to
  (`DbCapabilities::host_minimum`): 4,096-byte results, 1,024-byte
  cells, one row per statement, no streamed reads.

#### Profile::ALL

```rust
pub const ALL: [Profile; 3] = _
```

Every profile, in the order conformance runs report them.

#### Profile::name

```rust
pub fn name(self) -> &'static str
```

Stable name used in test output and the plugin tooling
(`first-party`, `d1`, `minimum`).

#### Profile::parse

```rust
pub fn parse(name: &str) -> Option<Self>
```

Parses [`Self::name`](#profilename).

#### Profile::capabilities

```rust
pub fn capabilities(self) -> DbCapabilities
```

The capability advertisement this profile enforces.

Implements: `Clone`, `Copy`, `Debug`, `Eq`, `Hash`, `PartialEq`
