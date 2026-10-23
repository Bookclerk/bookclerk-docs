# bookclerk_plugin_integration_audiobookshelf::guest

Guest-process helpers for the external Audiobookshelf integration plugin.

Opaque config JSON (from `[integrations.audiobookshelf]` / spawn config) drives
the HTTP client. User-watch queues newly observed users for
[`guest_event_poll`](#guest_event_poll) — the host owns claim-ticket / portal workflows.

| Item | Kind | Summary |
| --- | --- | --- |
| [`AbsGuestState`](#absgueststate) | struct | Shared guest state for the ABS external plugin process. |
| [`guest_start`](#guest_start) | function | Start ABS guest background work (user watch → queue for `guest_event_poll`). |
| [`guest_event_poll`](#guest_event_poll) | function | Drains newly observed ABS users queued by the background watch loop. |
| [`guest_health`](#guest_health) | function | Probes ABS connectivity and returns the typed health reply for the host status strip. |
| [`guest_diagnose`](#guest_diagnose) | function | Runs connectivity checks and returns human-readable diagnose lines (libraries listed). |
| [`guest_scan_library`](#guest_scan_library) | function | Asks Audiobookshelf to rescan the configured library (post-acquire hook). |
| [`guest_sync_listening`](#guest_sync_listening) | function | Collect listening progress as typed ABI rows (host upserts). |
| [`guest_authenticate_user`](#guest_authenticate_user) | function | Username/password login against ABS (when allowed). |
| [`guest_on_event`](#guest_on_event) | function | Handle host-forwarded integration events (e.g. book_acquired → scan). |

## Structs

### AbsGuestState

```rust
pub struct AbsGuestState {
    /* private fields */
}
```

Shared guest state for the ABS external plugin process.

#### AbsGuestState::from_config_json

```rust
pub fn from_config_json(config: &Value) -> Self
```

Build guest state from opaque spawn-config / settings JSON.

#### AbsGuestState::from_config

```rust
pub fn from_config(config: AudiobookshelfConfig) -> Self
```

Build guest state from a typed ABS config.

## Functions

### guest_start

```rust
pub async fn guest_start(state: Arc<Mutex<AbsGuestState>>) -> Result<()>
```

Start ABS guest background work (user watch → queue for [`guest_event_poll`](#guest_event_poll)).

#### Errors

Returns an error when the operation fails.

### guest_event_poll

```rust
pub async fn guest_event_poll(state: &Mutex<AbsGuestState>) -> EventPollResult
```

Drains newly observed ABS users queued by the background watch loop.

Call from the host's event-poll RPC so user create/update notifications can
be applied without blocking the watch task.

### guest_health

```rust
pub async fn guest_health(state: &Mutex<AbsGuestState>) -> Result<HealthOk>
```

Probes ABS connectivity and returns the typed health reply for the host status strip.

Misconfiguration (no client) yields `ok: false` with a detail message rather
than an RPC error so diagnose/health UIs can render it.

#### Errors

Returns an error when the operation fails.

### guest_diagnose

```rust
pub async fn guest_diagnose(state: &Mutex<AbsGuestState>) -> Result<Vec<String>>
```

Runs connectivity checks and returns human-readable diagnose lines (libraries listed).

#### Errors

Returns an error when the operation fails.

### guest_scan_library

```rust
pub async fn guest_scan_library(state: &Mutex<AbsGuestState>, force: bool) -> Result<()>
```

Asks Audiobookshelf to rescan the configured library (post-acquire hook).

#### Arguments

* `state` - Shared guest state with ABS client + config.
* `force` - When true, request a forced rescan from ABS.

#### Errors

Returns an error when the operation fails.

### guest_sync_listening

```rust
pub async fn guest_sync_listening(
    state: &Mutex<AbsGuestState>,
) -> Result<Vec<ListeningProgress>>
```

Collect listening progress as typed ABI rows (host upserts).

#### Errors

Returns an error when the operation fails.

### guest_authenticate_user

```rust
pub async fn guest_authenticate_user(
    state: &Mutex<AbsGuestState>,
    username: &str,
    password: &str,
) -> Result<ExternalUser>
```

Username/password login against ABS (when allowed).

#### Errors

Returns an error when the operation fails.

### guest_on_event

```rust
pub async fn guest_on_event(state: &Mutex<AbsGuestState>, params: &Value) -> Result<()>
```

Handle host-forwarded integration events (e.g. book_acquired → scan).

#### Errors

Returns an error when the operation fails.
