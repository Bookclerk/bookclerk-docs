# bookclerk_db_exec::begin_gate

A SeaORM proxy wrapper that refuses interactive transactions the way an
external database adapter does.

Production hosts reach every database through an adapter guest whose
session answers `begin()` with `unsupported`. A test that opens an engine
proxy directly gets a working `begin()` instead, so a host write that
opens an interactive transaction passes the test and fails in production.
Wrapping the engine proxy in a [`BeginGate`](#begingate) closes that gap.

| Item | Kind | Summary |
| --- | --- | --- |
| [`BeginGateHandle`](#begingatehandle) | struct | The test's handle on a `BeginGate`: the refusal switch and the count. |
| [`BeginGate`](#begingate) | struct | An engine proxy whose `begin()` can be refused. |
| [`with_batch_in_progress`](#with_batch_in_progress) | function | Runs `batch` with every gate open for the transaction it opens. |
| [`INTERACTIVE_TRANSACTIONS_REFUSED`](#interactive_transactions_refused) | constant | The message the host sees when a guest refuses `begin()`. |

## Structs

### BeginGateHandle

```rust
pub struct BeginGateHandle(/* private */);
```

The test's handle on a [`BeginGate`]: the refusal switch and the count.

#### BeginGateHandle::refuse

```rust
pub fn refuse(&self, refusing: bool)
```

Switches refusal on or off.

A gate starts open, so schema application and other setup can use
the engine's transactions; the test switches it to refusing before
running the code under test.

#### BeginGateHandle::refused

```rust
pub fn refused(&self) -> u32
```

Refused `begin()` calls so far.

Implements: `Clone`, `Debug`

### BeginGate

```rust
pub struct BeginGate {
    /* private fields */
}
```

An engine proxy whose `begin()` can be refused.

While refusing, a `begin()` from the code under test records the proxy's
fail-closed per-task fault, and the gate answers every statement inside
the transaction with [`INTERACTIVE_TRANSACTIONS_REFUSED`](#interactive_transactions_refused) (the adapters
refuse statements after a failed begin the same way, so none autocommits);
the matching commit or rollback clears the fault without touching the
engine, which never opened a transaction. Batches run inside [`with_batch_in_progress`](#with_batch_in_progress) pass
through: they are the adapter's atomic batches, not interactive
transactions.

#### BeginGate::new

```rust
pub fn new(inner: impl ProxyDatabaseTrait + 'static) -> (Self, BeginGateHandle)
```

Wraps `inner`. The gate starts open.

Implements: `Debug`, `ProxyDatabaseTrait`

## Functions

### with_batch_in_progress

```rust
pub async fn with_batch_in_progress<F, T>(batch: F) -> T
where
    F: Future<Output = T>,
```

Runs `batch` with every gate open for the transaction it opens.

## Constants

### INTERACTIVE_TRANSACTIONS_REFUSED

```rust
pub const INTERACTIVE_TRANSACTIONS_REFUSED: &str = "unsupported: interactive adapter transactions";
```

The message the host sees when a guest refuses `begin()`.

Matches the native-behind-workerd gateway's answer, so a test that
matches on it also matches production logs.
