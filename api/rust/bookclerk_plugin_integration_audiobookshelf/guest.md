**bookclerk_plugin_integration_audiobookshelf > guest**

# Module: guest

## Contents

**Structs**

- [`AbsGuestState`](#absgueststate) - Shared guest state for the ABS external plugin process.

**Functions**

- [`guest_authenticate_user`](#guest_authenticate_user) - Username/password login against ABS (when allowed).
- [`guest_diagnose`](#guest_diagnose) - Runs connectivity checks and returns human-readable diagnose lines (libraries listed).
- [`guest_event_poll`](#guest_event_poll) - Drains newly observed ABS users queued by the background watch loop.
- [`guest_health`](#guest_health) - Probes ABS connectivity and returns the typed health reply for the host status strip.
- [`guest_on_event`](#guest_on_event) - Handle host-forwarded integration events (e.g. book_acquired → scan).
- [`guest_scan_library`](#guest_scan_library) - Asks Audiobookshelf to rescan the configured library (post-acquire hook).
- [`guest_start`](#guest_start) - Start ABS guest background work (user watch → queue for [`guest_event_poll`](guest.md#bookclerk_plugin_integration_audiobookshelfguestguest_event_poll)).
- [`guest_sync_listening`](#guest_sync_listening) - Collect listening progress as typed ABI rows (host upserts).

---

## bookclerk_plugin_integration_audiobookshelf::guest::AbsGuestState

*Struct*

Shared guest state for the ABS external plugin process.

**Methods:**

- `fn from_config_json(config: &Value) -> Self` - Build guest state from opaque spawn-config / settings JSON.
- `fn from_config(config: AudiobookshelfConfig) -> Self` - Build guest state from a typed ABS config.



## bookclerk_plugin_integration_audiobookshelf::guest::guest_authenticate_user

*Function*

Username/password login against ABS (when allowed).

# Errors

Returns an error when the operation fails.

```rust
fn guest_authenticate_user(state: &tokio::sync::Mutex<AbsGuestState>, username: &str, password: &str) -> bookclerk_integrations::Result<bookclerk_integrations::ExternalUser>
```



## bookclerk_plugin_integration_audiobookshelf::guest::guest_diagnose

*Function*

Runs connectivity checks and returns human-readable diagnose lines (libraries listed).

# Errors

Returns an error when the operation fails.

```rust
fn guest_diagnose(state: &tokio::sync::Mutex<AbsGuestState>) -> bookclerk_integrations::Result<Vec<String>>
```



## bookclerk_plugin_integration_audiobookshelf::guest::guest_event_poll

*Function*

Drains newly observed ABS users queued by the background watch loop.

Call from the host's event-poll RPC so user create/update notifications can
be applied without blocking the watch task.

```rust
fn guest_event_poll(state: &tokio::sync::Mutex<AbsGuestState>) -> bookclerk_plugin_sdk::EventPollResult
```



## bookclerk_plugin_integration_audiobookshelf::guest::guest_health

*Function*

Probes ABS connectivity and returns the typed health reply for the host status strip.

Misconfiguration (no client) yields `ok: false` with a detail message rather
than an RPC error so diagnose/health UIs can render it.

# Errors

Returns an error when the operation fails.

```rust
fn guest_health(state: &tokio::sync::Mutex<AbsGuestState>) -> bookclerk_integrations::Result<bookclerk_plugin_sdk::HealthOk>
```



## bookclerk_plugin_integration_audiobookshelf::guest::guest_on_event

*Function*

Handle host-forwarded integration events (e.g. book_acquired → scan).

# Errors

Returns an error when the operation fails.

```rust
fn guest_on_event(state: &tokio::sync::Mutex<AbsGuestState>, params: &serde_json::Value) -> bookclerk_integrations::Result<()>
```



## bookclerk_plugin_integration_audiobookshelf::guest::guest_scan_library

*Function*

Asks Audiobookshelf to rescan the configured library (post-acquire hook).

# Arguments

* `state` - Shared guest state with ABS client + config.
* `force` - When true, request a forced rescan from ABS.

# Errors

Returns an error when the operation fails.

```rust
fn guest_scan_library(state: &tokio::sync::Mutex<AbsGuestState>, force: bool) -> bookclerk_integrations::Result<()>
```



## bookclerk_plugin_integration_audiobookshelf::guest::guest_start

*Function*

Start ABS guest background work (user watch → queue for [`guest_event_poll`](guest.md#bookclerk_plugin_integration_audiobookshelfguestguest_event_poll)).

# Errors

Returns an error when the operation fails.

```rust
fn guest_start(state: std::sync::Arc<tokio::sync::Mutex<AbsGuestState>>) -> bookclerk_integrations::Result<()>
```



## bookclerk_plugin_integration_audiobookshelf::guest::guest_sync_listening

*Function*

Collect listening progress as typed ABI rows (host upserts).

# Errors

Returns an error when the operation fails.

```rust
fn guest_sync_listening(state: &tokio::sync::Mutex<AbsGuestState>) -> bookclerk_integrations::Result<Vec<bookclerk_plugin_sdk::ListeningProgress>>
```



