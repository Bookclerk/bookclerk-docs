**bookclerk_plugin_host > authority**

# Module: authority

## Contents

**Functions**

- [`apply_grant_store`](#apply_grant_store) - Reconcile every live session against a loaded **persisted** grant store.
- [`authority_revision`](#authority_revision) - Canonical SHA-256 of security-relevant **effective** authority.
- [`fence_plugin_key`](#fence_plugin_key) - Fences every live session for `plugin_key` (grant/authority change).
- [`fence_stale_grant_revisions`](#fence_stale_grant_revisions) - Fences live sessions whose **persisted** [`crate::grant_revision`](consent.md#bookclerk_plugin_hostconsentgrant_revision) is not current.
- [`fence_stale_sessions`](#fence_stale_sessions) - Fences live sessions for `plugin_key` whose revision is not `current_revision`.
- [`is_fenced`](#is_fenced) - True when this cancel flag has been fenced.
- [`live_session_count`](#live_session_count) - Sessions currently registered with the process-wide authority table.
- [`notify_grants_changed`](#notify_grants_changed) - Wake the grant watcher in **this** process (same-process save / tests).
- [`reconcile_grants_from_disk`](#reconcile_grants_from_disk) - Load `plugin-grants.json` and fence sessions that no longer match.
- [`register_session`](#register_session) - Registers a live session and returns its cancel flag.
- [`register_session_revisions`](#register_session_revisions) - Registers a live session with distinct persisted vs effective revisions.
- [`register_session_with_shutdown`](#register_session_with_shutdown) - Registers a live session whose shutdown hook runs the instant it is fenced.
- [`spawn_grant_watcher`](#spawn_grant_watcher) - Spawn the daemon grant watcher (fire-and-forget).
- [`unregister_session`](#unregister_session) - Drops a previously registered cancel flag from the live set.
- [`watch_grants_loop`](#watch_grants_loop) - Watch `plugin-grants.json` until the process exits.

**Constants**

- [`GRANT_WATCH_INTERVAL`](#grant_watch_interval) - How often the daemon re-reads `plugin-grants.json` when no in-process notify fires.

**Type Aliases**

- [`SessionShutdown`](#sessionshutdown) - Shutdown hook invoked the moment a live session is fenced.

---

## bookclerk_plugin_host::authority::GRANT_WATCH_INTERVAL

*Constant*: `std::time::Duration`

How often the daemon re-reads `plugin-grants.json` when no in-process notify fires.



## bookclerk_plugin_host::authority::SessionShutdown

*Type Alias*: `std::sync::Arc<dyn Fn>`

Shutdown hook invoked the moment a live session is fenced.



## bookclerk_plugin_host::authority::apply_grant_store

*Function*

Reconcile every live session against a loaded **persisted** grant store.

Missing keys and [`crate::grant_revision`](consent.md#bookclerk_plugin_hostconsentgrant_revision) mismatches are fenced. Effective
[`authority_revision`](authority.md#bookclerk_plugin_hostauthorityauthority_revision) (host overlays, clamped budgets) is **not** compared
to the persisted digest. Host-config overlay changes fence through
`crate::ExecutorIdentity::configuration_revision`.

```rust
fn apply_grant_store(store: &crate::consent::PluginGrantStore)
```



## bookclerk_plugin_host::authority::authority_revision

*Function*

Canonical SHA-256 of security-relevant **effective** authority.

Hash the grant **after** manifest ∩ operator ∩ host policy and host-controlled
overlays (TCP/CIDR implied by configured destinations, clamped budgets).
Omits `approved_at` and display aliases except the PluginKey. Distinct from
[`crate::grant_revision`](consent.md#bookclerk_plugin_hostconsentgrant_revision) and `crate::ExecutorIdentity::configuration_revision`.

```rust
fn authority_revision(grant: &crate::consent::PluginGrant) -> String
```



## bookclerk_plugin_host::authority::fence_plugin_key

*Function*

Fences every live session for `plugin_key` (grant/authority change).

```rust
fn fence_plugin_key(plugin_key: &str)
```



## bookclerk_plugin_host::authority::fence_stale_grant_revisions

*Function*

Fences live sessions whose **persisted** [`crate::grant_revision`](consent.md#bookclerk_plugin_hostconsentgrant_revision) is not current.

```rust
fn fence_stale_grant_revisions(plugin_key: &str, current_grant_revision: &str)
```



## bookclerk_plugin_host::authority::fence_stale_sessions

*Function*

Fences live sessions for `plugin_key` whose revision is not `current_revision`.

Pass an empty `current_revision` to fence every session for the key.
Each fenced session is cancelled **and** its shutdown hook is invoked
immediately (vat `Work::Shutdown`, which kills the child and mediated
sockets). Hooks run after the live-table lock is released.

```rust
fn fence_stale_sessions(plugin_key: &str, current_revision: &str)
```



## bookclerk_plugin_host::authority::is_fenced

*Function*

True when this cancel flag has been fenced.

```rust
fn is_fenced(flag: &std::sync::Arc<std::sync::atomic::AtomicBool>) -> bool
```



## bookclerk_plugin_host::authority::live_session_count

*Function*

Sessions currently registered with the process-wide authority table.

```rust
fn live_session_count() -> usize
```



## bookclerk_plugin_host::authority::notify_grants_changed

*Function*

Wake the grant watcher in **this** process (same-process save / tests).

Cross-process CLI writes are observed by polling the grants file.

```rust
fn notify_grants_changed()
```



## bookclerk_plugin_host::authority::reconcile_grants_from_disk

*Function*

Load `plugin-grants.json` and fence sessions that no longer match.

Unreadable / malformed files are left untouched for this tick so a
torn write cannot mass-fence. `PluginGrantStore::save` writes atomically.

```rust
fn reconcile_grants_from_disk(files_dir: &std::path::Path)
```



## bookclerk_plugin_host::authority::register_session

*Function*

Registers a live session and returns its cancel flag.

Drop the session (or call [`unregister_session`](authority.md#bookclerk_plugin_hostauthorityunregister_session)) when the vat exits.
Sessions registered this way are still cancelled on grant change, but
have no shutdown hook — prefer [`register_session_with_shutdown`](authority.md#bookclerk_plugin_hostauthorityregister_session_with_shutdown) for
product vats so fencing does not wait for the next RPC.

```rust
fn register_session(plugin_key: &str, revision: &str) -> std::sync::Arc<std::sync::atomic::AtomicBool>
```



## bookclerk_plugin_host::authority::register_session_revisions

*Function*

Registers a live session with distinct persisted vs effective revisions.

`grant_revision` is the digest of the **stored** operator grant. `revision`
is [`authority_revision`](authority.md#bookclerk_plugin_hostauthorityauthority_revision) of the **effective** runtime grant (after host
overlays). The grant watcher compares grant revisions only.

```rust
fn register_session_revisions(plugin_key: &str, grant_revision: &str, revision: &str, shutdown: SessionShutdown) -> std::sync::Arc<std::sync::atomic::AtomicBool>
```



## bookclerk_plugin_host::authority::register_session_with_shutdown

*Function*

Registers a live session whose shutdown hook runs the instant it is fenced.

Call this **after** the vat work channel exists so `shutdown` can send
`Work::Shutdown` without racing spawn.

```rust
fn register_session_with_shutdown(plugin_key: &str, revision: &str, shutdown: SessionShutdown) -> std::sync::Arc<std::sync::atomic::AtomicBool>
```



## bookclerk_plugin_host::authority::spawn_grant_watcher

*Function*

Spawn the daemon grant watcher (fire-and-forget).

# Returns

Stop flag; set it to end the loop (tests). Production callers leak the task
until process exit.

```rust
fn spawn_grant_watcher(files_dir: std::path::PathBuf) -> std::sync::Arc<std::sync::atomic::AtomicBool>
```



## bookclerk_plugin_host::authority::unregister_session

*Function*

Drops a previously registered cancel flag from the live set.

```rust
fn unregister_session(flag: &std::sync::Arc<std::sync::atomic::AtomicBool>)
```



## bookclerk_plugin_host::authority::watch_grants_loop

*Function*

Watch `plugin-grants.json` until the process exits.

In-process [`notify_grants_changed`](authority.md#bookclerk_plugin_hostauthoritynotify_grants_changed) wakes immediately; CLI writes in
another process are picked up within [`GRANT_WATCH_INTERVAL`](authority.md#bookclerk_plugin_hostauthoritygrant_watch_interval).

```rust
fn watch_grants_loop(files_dir: std::path::PathBuf, stop: std::sync::Arc<std::sync::atomic::AtomicBool>)
```



