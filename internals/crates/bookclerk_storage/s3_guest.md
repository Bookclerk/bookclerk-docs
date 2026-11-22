# bookclerk_storage::s3_guest

A storage guest's `Destination` capability over an object store.

The `s3` and `r2` destination plugins store through an `S3Backend` and
the `b2` plugin through a `B2Backend` (each built its own way: endpoint,
credentials, change feed). They answer the plugin ABI the same way:
streamed writes with optional conditions, staged writes committed by
token, and the intent operations through [`crate::guest`](guest.md).

## Structs

### ObjectStoreDestination

```rust
pub struct ObjectStoreDestination<B> {
    /* private fields */
}
```

The `Destination` capability of an object-store guest.

#### ObjectStoreDestination::new

```rust
pub fn new(backend: B) -> Self
```

Serves `backend` as a destination.

Implements: `Destination`
