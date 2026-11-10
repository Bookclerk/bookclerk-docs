# bookclerk_plugin_database_d1::adapter

The `AdapterDatabaseSession` the D1 guest serves.

It lives in the library, not the guest binary, so tests can run the
shared adapter conformance suite (`bookclerk-db-testkit`) against the
session the guest ships, on a mocked D1 HTTP API.

## Structs

### D1Session

```rust
pub struct D1Session {
    /* private fields */
}
```

D1 adapter session: typed atomic batches over the D1 HTTP API. D1 has no
interactive transactions, consistent backup reads or unit restore, so
those calls answer `unsupported`.

#### D1Session::new

```rust
pub fn new(dedicated: Option<D1Proxy>) -> Self
```

A session on `dedicated`, or on the shared library proxy when `None`.

Implements: `AdapterDatabaseSession`
