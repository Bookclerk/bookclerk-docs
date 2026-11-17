# bookclerk_plugin_integration_audiobookshelf::guest

Guest-process helpers for the external Audiobookshelf integration plugin.

Opaque config JSON (from `[integrations.audiobookshelf]` / spawn config) drives
the HTTP client. The user watch reports users created on the server from
[`guest_event_poll`](#guest_event_poll), which the host calls on its own schedule; the host
owns claim-ticket and portal workflows.

| Item | Kind | Summary |
| --- | --- | --- |
| [`AbsGuestState`](#absgueststate) | struct | Shared guest state for the ABS external plugin process. |
| [`guest_start`](#guest_start) | function | Seeds the user watch with the users the server already has. |
| [`guest_event_poll`](#guest_event_poll) | function | Reports the ABS users created since the last poll. |
| [`guest_health`](#guest_health) | function | Probes ABS connectivity and returns the typed health reply for the host status strip. |
| [`guest_diagnose`](#guest_diagnose) | function | Runs connectivity checks and returns human-readable diagnose lines (libraries listed). |
| [`guest_scan_library`](#guest_scan_library) | function | Asks Audiobookshelf to rescan the configured library (post-acquire hook). |
| [`guest_sync_listening`](#guest_sync_listening) | function | Collect listening progress as typed ABI rows (host upserts). |
| [`guest_authenticate_user`](#guest_authenticate_user) | function | Username/password login against ABS (when allowed). |
| [`guest_on_event`](#guest_on_event) | function | Handle host-forwarded integration events (e.g. book_acquired → scan). |
| [`guest_ensure_push`](#guest_ensure_push) | function | Opens the pushed-event socket when none is open, so the host learns of a new ABS user as soon as the server announces it. |
| [`guest_push_frame`](#guest_push_frame) | function | Answers one frame pushed on the ABS socket. |

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

#### AbsGuestState::from_bindings_json

```rust
pub fn from_bindings_json(config: &Value, secrets: &Value) -> Self
```

Build guest state from the `open` bindings: `CONFIG` plus `SECRETS`.

The host keeps secret settings out of `CONFIG`: an `api_key` set with
`bookclerk plugins config set audiobookshelf api_key --secret` arrives
only in `SECRETS`, so it fills `api_key` when `CONFIG` has none.

##### Arguments

* `config` - The `CONFIG` binding's JSON (instance settings).
* `secrets` - The `SECRETS` binding's JSON (unsealed secret settings).

#### AbsGuestState::from_config

```rust
pub fn from_config(config: AudiobookshelfConfig) -> Self
```

Build guest state from a typed ABS config.

## Functions

### guest_start

```rust
pub async fn guest_start(
    state: Arc<Mutex<AbsGuestState>>,
    memory: Option<&DatabaseBinding>,
) -> Result<()>
```

Seeds the user watch with the users the server already has.

Runs each time the guest starts. With the plugin database, only the
first start ever records users, and the users a server has then are
never reported; without it, every start seeds its in-memory set again.
A failed seed is logged; the next [`guest_event_poll`](#guest_event_poll) then seeds the
empty set instead and reports nobody.

#### Arguments

* `state` - Shared guest state with the ABS client and config.
* `memory` - The plugin's `LISTENING_STATE` database, when granted.

#### Errors

Returns an error when the integration has no usable client.

### guest_event_poll

```rust
pub async fn guest_event_poll(
    state: &Mutex<AbsGuestState>,
    memory: Option<&DatabaseBinding>,
) -> Result<EventPollResult>
```

Reports the ABS users created since the last poll.

Lists the server's users and returns those the watch has not reported
(see [`crate::watch`](watch.md)). When the watch was never seeded (the seed at
start failed), this poll seeds it and reports nobody.

#### Arguments

* `state` - Shared guest state with the ABS client and config.
* `memory` - The plugin's `LISTENING_STATE` database, when granted.

#### Errors

Returns an error when the client is missing, ABS cannot list its users,
or the watch database fails.

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
    memory: Option<&DatabaseBinding>,
) -> Result<Vec<ListeningProgress>>
```

Collect listening progress as typed ABI rows (host upserts).

With the `LISTENING_STATE` database only rows that changed since the
last sync come back, and each user's whole snapshot once a day (see
[`collect_listening_changes`](README.md#collect_listening_changes)); without it every row does.

#### Errors

Returns an error when ABS cannot list its users.

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

### guest_ensure_push

```rust
pub async fn guest_ensure_push(
    state: &Mutex<AbsGuestState>,
    sockets: Option<&dyn Sockets>,
) -> Result<bool>
```

Opens the pushed-event socket when none is open, so the host learns of a
new ABS user as soon as the server announces it.

Does nothing without the `SOCKETS` binding, with the user watch off, or
without an API key. A socket that closes is not reopened at once: the
next start or poll opens it again, so a server that keeps refusing the
key is retried at the poll's pace rather than in a loop. The host answers
the server's Engine.IO ping itself, so an open socket costs no wakes.

#### Arguments

* `state` - Shared guest state with the ABS config.
* `sockets` - The plugin's `SOCKETS` binding, when granted.

#### Returns

`true` when a socket was opened by this call.

#### Errors

Returns the host's error when the open sockets cannot be listed or the
socket cannot be opened.

### guest_push_frame

```rust
pub async fn guest_push_frame(
    state: &Mutex<AbsGuestState>,
    sockets: &dyn Sockets,
    event: &SocketMessage,
) -> Result<bool>
```

Answers one frame pushed on the ABS socket.

Sends the handshake replies (joining the namespace, then authenticating
with the API key), closes the socket when the server rejects the key,
and reports whether the frame announced a new user.

#### Arguments

* `state` - Shared guest state with the API key.
* `sockets` - The plugin's `SOCKETS` binding.
* `event` - The pushed frame.

#### Returns

`true` when the server announced a user created on it.

#### Errors

Returns the host's error when a reply cannot be sent.
