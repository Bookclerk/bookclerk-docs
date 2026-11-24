# bookclerk_plugin_abi::methods

Role capability method names (camelCase on the wire).

Each submodule exposes a single [`NAME`](health.md#name) constant equal to
the corresponding `ContentSource` / `RemoteLibrary` / `Oidc` /
`MetadataProvider` / `MediaDownloader` / `MetadataFiles` / `Notifier` /
`NotificationRaiser` / `AiModels` / `AiRunner` / `Provisioning` / `Cli` /
`EventConsumer` / `EventPublisher` / `QueueConsumer` / `QueueProducer` /
`PluginWorker` method in
`schema/plugin.capnp`. Prefer these constants over string literals when
dispatching or documenting capabilities.

[`METHOD_NAMES`](#method_names) enumerates every known method for discovery / schema
drift tests. Flat `UPPER_SNAKE` aliases live under [`names`](names.md).

## Modules

- [`shutdown`](shutdown.md): Graceful guest teardown before the host closes the transport.
- [`health`](health.md): Connectivity / configuration probe.
- [`diagnose`](diagnose.md): Human-readable diagnostic lines for `bookclerk plugins diagnose`.
- [`start`](start.md): Start background watchers (Cap'n Proto `RemoteLibrary.start`).
- [`stop`](stop.md): Stop background watchers started by `start` (Cap'n Proto `RemoteLibrary.stop`).
- [`event`](event.md): Deliver one ordered batch of domain events (Cap'n Proto `EventConsumer.event`).
- [`publish`](publish.md): Append one guest event to the host outbox (Cap'n Proto `EventPublisher.publish`).
- [`read`](read.md): Read one page of past outbox events (Cap'n Proto `EventPublisher.read`).
- [`queue`](queue.md): Deliver one ordered batch of queue messages (Cap'n Proto `QueueConsumer.queue`).
- [`scheduled`](scheduled.md): Run one cron fire (Cap'n Proto `ScheduledHandler.scheduled`).
- [`send`](send.md): Send one message into a granted queue (Cap'n Proto `QueueProducer.send`), or deliver one notification to one destination (Cap'n Proto `Notifier.send`).
- [`send_batch`](send_batch.md): Send a batch of messages into a granted queue (Cap'n Proto `QueueProducer.sendBatch`).
- [`poll_events`](poll_events.md): Poll for plugin-observed external users after `start`.
- [`ack_events`](ack_events.md): Acknowledge users a `poll_events` reply handed over.
- [`scan_library`](scan_library.md): Trigger an integration library scan against a remote server (e.g. ABS).
- [`sync_listening`](sync_listening.md): Pull listening-progress snapshots; host upserts tagged with the plugin id.
- [`authenticate_user`](authenticate_user.md): Validate a username/password against a remote IdP (Cap'n Proto `Oidc.authenticateUser`) and return an external user.
- [`lookup`](lookup.md): Edition records for a set of identifiers (Cap'n Proto `MetadataProvider.lookup`).
- [`search`](search.md): Edition records matching a title and author (Cap'n Proto `MetadataProvider.search`).
- [`chapters`](chapters.md): Chapter layout of one edition (Cap'n Proto `MetadataProvider.chapters`).
- [`download_media`](download_media.md): Cover image bodies streamed into a host sink (Cap'n Proto `MediaDownloader.downloadMedia`).
- [`parse`](parse.md): What one metadata file says (Cap'n Proto `MetadataFiles.parse`).
- [`render`](render.md): A metadata file's bytes with chosen fields rewritten (Cap'n Proto `MetadataFiles.render`).
- [`validate_destination`](validate_destination.md): Normalise and check a patron's address on a notifier (Cap'n Proto `Notifier.validateDestination`).
- [`check_deliverability`](check_deliverability.md): Run the operator's deliverability check on a notifier (Cap'n Proto `Notifier.checkDeliverability`).
- [`raise`](raise.md): Raise a notification type the plugin declared (Cap'n Proto `NotificationRaiser.raise`).
- [`models`](models.md): List the models a plugin serves (Cap'n Proto `AiModels.models`).
- [`run`](run.md): Run one model on one input (Cap'n Proto `AiModels.run` on a model plugin, `AiRunner.run` on the `AI` binding).
- [`provision`](provision.md): Run a guided setup with the operator's grant (Cap'n Proto `Provisioning.provision`).
- [`deprovision`](deprovision.md): Remove what a guided setup created (Cap'n Proto `Provisioning.deprovision`).
- [`clients`](clients.md): Plugin-provided OIDC client templates (Cap'n Proto `Oidc.clients`).
- [`describe`](describe.md): Return the guest's declared CLI schema (`crate::CliSchema`; Cap'n Proto `PluginCli.describe`). Also the root `PluginWorker.describe` identity call.
- [`invoke`](invoke.md): Invoke a declared plugin CLI command (Cap'n Proto `PluginCli.invoke`).
- [`login`](login.md): Password-style source login; host seals returned credentials.
- [`login_start`](login_start.md): Begin interactive OAuth (returns browser URL + session id).
- [`login_complete`](login_complete.md): Finish interactive OAuth after the operator completes the browser flow.
- [`scan`](scan.md): Scan a source storefront library; host upserts returned `crate::ScanBook` rows.
- [`fetch_title`](fetch_title.md): Download/decrypt one title into the host cache directory.
- [`search_catalog`](search_catalog.md): Search a storefront catalog for purchase / discovery UIs.
- [`expand_candidates`](expand_candidates.md): Expand related catalog candidates from a seed title.
- [`purchase_hint`](purchase_hint.md): Resolve a purchase URL / price hint for a catalog product.
- [`purchase_hints`](purchase_hints.md): Resolve purchase URLs / price hints for several catalog products at once.
- [`quality_offers`](quality_offers.md): Report the audio quality one account would download for several titles.
- [`list_deals`](list_deals.md): List current storefront deals / promotions.
- [`list_accounts`](list_accounts.md): List source accounts known to the guest (host usually prefers DB rows).
- [`catalog_detail`](catalog_detail.md): Fetch rich catalog detail for one product id.
- [`revoke_account`](revoke_account.md): Undo store-side state for one account before the host deletes its stored credentials (for example, deregister the login device).
- [`put`](put.md): Put a small object (cover/sidecar) into an output destination.
- [`get`](get.md): Read a small object from an output destination.
- [`exists`](exists.md): Test whether an object key exists in an output destination.
- [`list`](list.md): List object keys under a prefix in an output destination.
- [`copy`](copy.md): Copy an object within an output destination.
- [`delete`](delete.md): Delete an object key from an output destination.
- [`names`](names.md): Flat `UPPER_SNAKE` aliases matching historical `protocol::methods` usage.

## Constants

### METHOD_NAMES

```rust
pub const METHOD_NAMES: &[&str] = _;
```

All known capability method names for discovery, docs, and schema checks.

Order is stable for readability; `scripts/gen-plugin-abi.py --check`
verifies this list against `schema/plugin.capnp`.
