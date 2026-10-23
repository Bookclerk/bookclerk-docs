**bookclerk_plugin_host > destinations**

# Module: destinations

## Contents

**Functions**

- [`build_acquire_destinations`](#build_acquire_destinations) - Build destination backends for acquire from the deployed output plugins.
- [`build_storage_backend`](#build_storage_backend) - Build listing/storage backends from the deployed output plugins.

---

## bookclerk_plugin_host::destinations::build_acquire_destinations

*Function*

Build destination backends for acquire from the deployed output plugins.

# Errors

Returns an error when no destination is deployed on this host or two
instances of one kind are deployed.

```rust
fn build_acquire_destinations(config: &bookclerk_config::Config, destinations: &crate::host::DestinationRegistry) -> bookclerk_acquire::Result<bookclerk_acquire::AcquireDestinations>
```



## bookclerk_plugin_host::destinations::build_storage_backend

*Function*

Build listing/storage backends from the deployed output plugins.

# Errors

Returns an error when no destination is deployed or two instances of one
kind are deployed.

```rust
fn build_storage_backend(destinations: &crate::host::DestinationRegistry) -> bookclerk_storage::Result<Box<dyn StorageBackend>>
```



