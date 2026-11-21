# bookclerk_storage::s3_guest

A storage guest's `Destination` capability over an [`S3Backend`](README.md#s3backend).

The `s3`, `r2` and `b2` destination plugins all store through an
[`S3Backend`](README.md#s3backend) (each built its own way: endpoint, credentials, change
feed), and answer the plugin ABI the same way: streamed writes with
optional conditions, staged writes committed by token, and the intent
operations through [`crate::guest`](guest.md).

## Structs

### ObjectStoreDestination

```rust
pub struct ObjectStoreDestination {
    /* private fields */
}
```

The `Destination` capability of an object-store guest.

#### ObjectStoreDestination::new

```rust
pub fn new(backend: S3Backend) -> Self
```

Serves `backend` as a destination.

Implements: `Destination`
