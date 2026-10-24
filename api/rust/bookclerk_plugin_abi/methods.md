# bookclerk_plugin_abi::methods

Role capability method names (camelCase on the wire).

Each submodule exposes a single [`NAME`](methods/health.md#name) constant equal to
the corresponding `ContentSource` / `RemoteLibrary` / `Oidc` / `Cli` /
`EventConsumer` / `EventPublisher` / `QueueConsumer` / `QueueProducer` /
`PluginWorker` method in
`schema/plugin.capnp`. Prefer these constants over string literals when
dispatching or documenting capabilities.

[`METHOD_NAMES`](#method_names) enumerates every known method for discovery / schema
drift tests. Flat `UPPER_SNAKE` aliases live under [`names`](methods/names.md).

## Modules

- [`shutdown`](methods/shutdown.md): Graceful guest teardown before the host closes the transport.
- [`health`](methods/health.md): Connectivity / configuration probe.
- [`diagnose`](methods/diagnose.md): Human-readable diagnostic lines for `bookclerk plugins diagnose`.
- [`start`](methods/start.md): Start background watchers (Cap'n Proto `RemoteLibrary.start`).
- [`stop`](methods/stop.md): Stop background watchers started by `start` (Cap'n Proto `RemoteLibrary.stop`).
- [`event`](methods/event.md): Deliver one ordered batch of domain events (Cap'n Proto `EventConsumer.event`).
- [`publish`](methods/publish.md): Append one guest event to the host outbox (Cap'n Proto `EventPublisher.publish`).
- [`queue`](methods/queue.md): Deliver one ordered batch of queue messages (Cap'n Proto `QueueConsumer.queue`).
- [`send`](methods/send.md): Send one message into a granted queue (Cap'n Proto `QueueProducer.send`).
- [`send_batch`](methods/send_batch.md): Send a batch of messages into a granted queue (Cap'n Proto `QueueProducer.sendBatch`).
- [`poll_events`](methods/poll_events.md): Poll for plugin-observed external users after `start`.
- [`scan_library`](methods/scan_library.md): Trigger an integration library scan against a remote server (e.g. ABS).
- [`sync_listening`](methods/sync_listening.md): Pull listening-progress snapshots; host upserts tagged with the plugin id.
- [`authenticate_user`](methods/authenticate_user.md): Validate a username/password against a remote IdP (Cap'n Proto `Oidc.authenticateUser`) and return an external user.
- [`clients`](methods/clients.md): Plugin-provided OIDC client templates (Cap'n Proto `Oidc.clients`).
- [`describe`](methods/describe.md): Return the guest's declared CLI schema (`crate::CliSchema`; Cap'n Proto `PluginCli.describe`). Also the root `PluginWorker.describe` identity call.
- [`invoke`](methods/invoke.md): Invoke a declared plugin CLI command (Cap'n Proto `PluginCli.invoke`).
- [`login`](methods/login.md): Password-style source login; host seals returned credentials.
- [`login_start`](methods/login_start.md): Begin interactive OAuth (returns browser URL + session id).
- [`login_complete`](methods/login_complete.md): Finish interactive OAuth after the operator completes the browser flow.
- [`scan`](methods/scan.md): Scan a source storefront library; host upserts returned `crate::ScanBook` rows.
- [`fetch_title`](methods/fetch_title.md): Download/decrypt one title into the host cache directory.
- [`search_catalog`](methods/search_catalog.md): Search a storefront catalog for purchase / discovery UIs.
- [`expand_candidates`](methods/expand_candidates.md): Expand related catalog candidates from a seed title.
- [`purchase_hint`](methods/purchase_hint.md): Resolve a purchase URL / price hint for a catalog product.
- [`list_deals`](methods/list_deals.md): List current storefront deals / promotions.
- [`list_accounts`](methods/list_accounts.md): List source accounts known to the guest (host usually prefers DB rows).
- [`catalog_detail`](methods/catalog_detail.md): Fetch rich catalog detail for one product id.
- [`revoke_account`](methods/revoke_account.md): Undo store-side state for one account before the host deletes its stored credentials (for example, deregister the login device).
- [`put`](methods/put.md): Put a small object (cover/sidecar) into an output destination.
- [`get`](methods/get.md): Read a small object from an output destination.
- [`exists`](methods/exists.md): Test whether an object key exists in an output destination.
- [`list`](methods/list.md): List object keys under a prefix in an output destination.
- [`copy`](methods/copy.md): Copy an object within an output destination.
- [`delete`](methods/delete.md): Delete an object key from an output destination.
- [`names`](methods/names.md): Flat `UPPER_SNAKE` aliases matching historical `protocol::methods` usage.

## Constants

### METHOD_NAMES

```rust
pub const METHOD_NAMES: &[&str] = _;
```

All known capability method names for discovery, docs, and schema checks.

Order is stable for readability; `scripts/gen-plugin-abi.py --check`
verifies this list against `schema/plugin.capnp`.
