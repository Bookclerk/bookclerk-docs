**bookclerkd > event_worker**

# Module: event_worker

## Contents

**Functions**

- [`start_event_runtime`](#start_event_runtime) - Dispatch undispatched outbox rows and run configured delivery workers.
- [`upsert_event_subscriber_catalog`](#upsert_event_subscriber_catalog) - Upsert this node's discovered + loaded integration subscriptions.

---

## bookclerkd::event_worker::start_event_runtime

*Function*

Dispatch undispatched outbox rows and run configured delivery workers.

```rust
fn start_event_runtime(state: std::sync::Arc<crate::api::AppState>)
```



## bookclerkd::event_worker::upsert_event_subscriber_catalog

*Function*

Upsert this node's discovered + loaded integration subscriptions.

Catalog `plugin_id` is the plugin instance id when the loaded integration
has one. Otherwise it is the provenance-qualified PluginKey. Two instances
of one key each get their own row so event delivery does not collapse them.

```rust
fn upsert_event_subscriber_catalog(state: &crate::api::AppState)
```



