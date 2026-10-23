**bookclerk_acquire > destinations**

# Module: destinations

## Contents

**Structs**

- [`AcquireDestination`](#acquiredestination) - One configured output destination for an acquire run.
- [`AcquireDestinations`](#acquiredestinations) - Ordered set of acquire destinations built from the deployed destination plugins.

**Functions**

- [`destination_id`](#destination_id) - The id of the output destination of `kind`, as plans, upload manifests

---

## bookclerk_acquire::destinations::AcquireDestination

*Struct*

One configured output destination for an acquire run.

**Fields:**
- `kind: bookclerk_config::OutputBackendKind` - Destination backend kind (`local`, `s3`, …).
- `backend: Box<dyn StorageBackend>` - Constructed storage backend for this destination.
- `options: bookclerk_source::DownloadOptions` - Source download options (quality, chapter prefs, …).



## bookclerk_acquire::destinations::AcquireDestinations

*Struct*

Ordered set of acquire destinations built from the deployed destination plugins.

**Fields:**
- `items: Vec<AcquireDestination>` - Configured acquire destinations in evaluation order.
- `primary: bookclerk_config::OutputBackendKind` - Index of the primary destination used for library status keys.
- `multi_destination: bookclerk_config::MultiDestinationMode` - Whether multiple destinations are enabled for this run.

**Methods:**

- `fn len(self: &Self) -> usize` - Number of destinations.
- `fn is_empty(self: &Self) -> bool` - Returns true when no destinations are deployed.
- `fn primary_destination(self: &Self) -> &AcquireDestination` - Returns the primary destination used for library storage keys.
- `fn destination(self: &Self, kind: OutputBackendKind) -> Option<&AcquireDestination>` - Returns the destination whose kind equals `kind`, if configured.
- `fn listing_backend(self: &Self) -> Result<Box<dyn StorageBackend>>` - Clone destination backends into a listing/match storage handle.
- `fn into_listing_backend(self: Self) -> Result<Box<dyn StorageBackend>>` - Consumes this set and returns a listing backend over the primary destination.



## bookclerk_acquire::destinations::destination_id

*Function*

The id of the output destination of `kind`, as plans, upload manifests
and `title_files` name it: its lowercase kind name (a node deploys at
most one destination per kind).

```rust
fn destination_id(kind: bookclerk_config::OutputBackendKind) -> &'static str
```



