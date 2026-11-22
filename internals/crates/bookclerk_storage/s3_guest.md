# bookclerk_storage::s3_guest

A storage guest's `Destination` capability over an object store.

The `s3` and `r2` destination plugins store through an `S3Backend` and
the `b2` plugin through a `B2Backend` (each built its own way: endpoint,
credentials, change feed). They answer the plugin ABI the same way:
streamed writes with optional conditions, staged writes committed by
token, and the intent operations through [`crate::guest`](guest.md).

| Item | Kind | Summary |
| --- | --- | --- |
| [`ChangeSource`](#changesource) | trait | A change feed kept beside the backend rather than in it, for a feed whose calls are not `Send` (one over the guest's own bindings, such as the `b2` destination's webhook feed). |
| [`ObjectStoreDestination`](#objectstoredestination) | struct | The `Destination` capability of an object-store guest. |

## Traits

### ChangeSource

```rust
pub trait ChangeSource {
    fn changes<'life0, 'life1, 'async_trait>(
        &'life0 self,
        cursor: Option<&'life1 str>,
        limit: u32,
    ) -> Pin<Box<dyn Future<Output = Result<Option<ChangeFeedPage>, StorageError>> + 'async_trait>>
    where
        Self: 'async_trait,
        'life0: 'async_trait,
        'life1: 'async_trait,;
}
```

A change feed kept beside the backend rather than in it, for a feed
whose calls are not `Send` (one over the guest's own bindings, such as
the `b2` destination's webhook feed).

#### ChangeSource::changes

```rust
fn changes<'life0, 'life1, 'async_trait>(
    &'life0 self,
    cursor: Option<&'life1 str>,
    limit: u32,
) -> Pin<Box<dyn Future<Output = Result<Option<ChangeFeedPage>, StorageError>> + 'async_trait>>
where
    Self: 'async_trait,
    'life0: 'async_trait,
    'life1: 'async_trait,
```

One page of changes since `cursor`, or `None` when the feed is not
running now (the host lists instead).

##### Errors

Returns [`StorageError::InvalidCursor`](README.md#storageerror) for a cursor the feed does
not accept, and the feed's other failures.

Implemented by: `B2ChangeFeed`

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

#### ObjectStoreDestination::with_changes

```rust
pub fn with_changes(self, feed: Box<dyn ChangeSource>) -> Self
```

Answers `changes` from `feed` instead of the backend.

Implements: `Destination`
