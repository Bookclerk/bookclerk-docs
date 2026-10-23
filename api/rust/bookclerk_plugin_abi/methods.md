**bookclerk_plugin_abi > methods**

# Module: methods

## Contents

**Modules**

- [`authenticate_user`](#authenticate_user) - Validate a username/password against a remote IdP (Cap'n Proto
- [`catalog_detail`](#catalog_detail) - Fetch rich catalog detail for one product id.
- [`clients`](#clients) - Plugin-provided OIDC client templates (Cap'n Proto `Oidc.clients`).
- [`copy`](#copy) - Copy an object within an output destination.
- [`delete`](#delete) - Delete an object key from an output destination.
- [`describe`](#describe) - Return the guest's declared CLI schema ([`crate::CliSchema`](generated.md#bookclerk_plugin_abigeneratedclischema); Cap'n Proto
- [`diagnose`](#diagnose) - Human-readable diagnostic lines for `bookclerk plugins diagnose`.
- [`event`](#event) - Deliver one ordered batch of domain events (Cap'n Proto `EventConsumer.event`).
- [`exists`](#exists) - Test whether an object key exists in an output destination.
- [`expand_candidates`](#expand_candidates) - Expand related catalog candidates from a seed title.
- [`fetch_title`](#fetch_title) - Download/decrypt one title into the host cache directory.
- [`get`](#get) - Read a small object from an output destination.
- [`health`](#health) - Connectivity / configuration probe.
- [`invoke`](#invoke) - Invoke a declared plugin CLI command (Cap'n Proto `PluginCli.invoke`).
- [`list`](#list) - List object keys under a prefix in an output destination.
- [`list_accounts`](#list_accounts) - List source accounts known to the guest (host usually prefers DB rows).
- [`list_deals`](#list_deals) - List current storefront deals / promotions.
- [`login`](#login) - Password-style source login; host seals returned credentials.
- [`login_complete`](#login_complete) - Finish interactive OAuth after the operator completes the browser flow.
- [`login_start`](#login_start) - Begin interactive OAuth (returns browser URL + session id).
- [`names`](#names) - Flat `UPPER_SNAKE` aliases matching historical `protocol::methods` usage.
- [`poll_events`](#poll_events) - Poll for plugin-observed external users after [`start`](methods/start.md).
- [`publish`](#publish) - Append one guest event to the host outbox (Cap'n Proto `EventPublisher.publish`).
- [`purchase_hint`](#purchase_hint) - Resolve a purchase URL / price hint for a catalog product.
- [`put`](#put) - Put a small object (cover/sidecar) into an output destination.
- [`queue`](#queue) - Deliver one ordered batch of queue messages (Cap'n Proto `QueueConsumer.queue`).
- [`scan`](#scan) - Scan a source storefront library; host upserts returned [`crate::ScanBook`](generated.md#bookclerk_plugin_abigeneratedscanbook) rows.
- [`scan_library`](#scan_library) - Trigger an integration library scan against a remote server (e.g. ABS).
- [`search_catalog`](#search_catalog) - Search a storefront catalog for purchase / discovery UIs.
- [`send`](#send) - Send one message into a granted queue (Cap'n Proto `QueueProducer.send`).
- [`send_batch`](#send_batch) - Send a batch of messages into a granted queue (Cap'n Proto
- [`shutdown`](#shutdown) - Graceful guest teardown before the host closes the transport.
- [`start`](#start) - Start background watchers (Cap'n Proto `RemoteLibrary.start`).
- [`stop`](#stop) - Stop background watchers started by [`start`](methods/start.md) (Cap'n Proto `RemoteLibrary.stop`).
- [`sync_listening`](#sync_listening) - Pull listening-progress snapshots; host upserts tagged with the plugin id.

**Constants**

- [`METHOD_NAMES`](#method_names) - All known capability method names for discovery, docs, and schema checks.

---

## bookclerk_plugin_abi::methods::METHOD_NAMES

*Constant*: `&[&str]`

All known capability method names for discovery, docs, and schema checks.

Order is stable for readability; `scripts/gen-plugin-abi.py --check`
verifies this list against `schema/plugin.capnp`.



## Module: authenticate_user

Validate a username/password against a remote IdP (Cap'n Proto
`Oidc.authenticateUser`) and return an external user.

Params: [`crate::AuthenticateUserParams`](generated.md#bookclerk_plugin_abigeneratedauthenticateuserparams).



## Module: catalog_detail

Fetch rich catalog detail for one product id.

Params: [`crate::CatalogDetailParams`](generated.md#bookclerk_plugin_abigeneratedcatalogdetailparams). Result: [`crate::CatalogHit`](generated.md#bookclerk_plugin_abigeneratedcataloghit).



## Module: clients

Plugin-provided OIDC client templates (Cap'n Proto `Oidc.clients`).



## Module: copy

Copy an object within an output destination.

Typed Cap'n Proto `Destination.copy` (`crate::Destination::copy`).



## Module: delete

Delete an object key from an output destination.

Typed Cap'n Proto `Destination.delete` (`crate::Destination::delete`).



## Module: describe

Return the guest's declared CLI schema ([`crate::CliSchema`](generated.md#bookclerk_plugin_abigeneratedclischema); Cap'n Proto
`PluginCli.describe`). Also the root `PluginWorker.describe` identity call.



## Module: diagnose

Human-readable diagnostic lines for `bookclerk plugins diagnose`.

Result: [`crate::DiagnoseResult`](generated.md#bookclerk_plugin_abigenerateddiagnoseresult).



## Module: event

Deliver one ordered batch of domain events (Cap'n Proto `EventConsumer.event`).



## Module: exists

Test whether an object key exists in an output destination.

Wire name kept for consent grants; guests implement existence via
`crate::Destination::head` (`None` means the key is missing).



## Module: expand_candidates

Expand related catalog candidates from a seed title.

Params: [`crate::ExpandCandidatesParams`](generated.md#bookclerk_plugin_abigeneratedexpandcandidatesparams). Result: list of [`crate::CatalogHit`](generated.md#bookclerk_plugin_abigeneratedcataloghit).



## Module: fetch_title

Download/decrypt one title into the host cache directory.

Params: [`crate::FetchTitleParams`](generated.md#bookclerk_plugin_abigeneratedfetchtitleparams). Result: [`crate::PlainFetch`](generated.md#bookclerk_plugin_abigeneratedplainfetch).



## Module: get

Read a small object from an output destination.

Typed Cap'n Proto `Destination.get` (`crate::Destination::get`).



## Module: health

Connectivity / configuration probe.

Result: [`crate::HealthOk`](rpc_types.md#bookclerk_plugin_abirpc_typeshealthok).



## Module: invoke

Invoke a declared plugin CLI command (Cap'n Proto `PluginCli.invoke`).

Params: [`crate::CliInvokeParams`](generated.md#bookclerk_plugin_abigeneratedcliinvokeparams). Result: [`crate::CliInvokeResult`](generated.md#bookclerk_plugin_abigeneratedcliinvokeresult).



## Module: list

List object keys under a prefix in an output destination.

Typed Cap'n Proto `Destination.list` (`crate::Destination::list`).



## Module: list_accounts

List source accounts known to the guest (host usually prefers DB rows).




## Module: list_deals

List current storefront deals / promotions.

Params: [`crate::ListDealsParams`](generated.md#bookclerk_plugin_abigeneratedlistdealsparams).



## Module: login

Password-style source login; host seals returned credentials.

Params: [`crate::LoginParams`](generated.md#bookclerk_plugin_abigeneratedloginparams). Result: [`crate::LoginResult`](generated.md#bookclerk_plugin_abigeneratedloginresult).



## Module: login_complete

Finish interactive OAuth after the operator completes the browser flow.

Params: [`crate::LoginCompleteParams`](generated.md#bookclerk_plugin_abigeneratedlogincompleteparams). Result: [`crate::LoginResult`](generated.md#bookclerk_plugin_abigeneratedloginresult).



## Module: login_start

Begin interactive OAuth (returns browser URL + session id).

Params: [`crate::LoginParams`](generated.md#bookclerk_plugin_abigeneratedloginparams). Result: [`crate::LoginStartResult`](generated.md#bookclerk_plugin_abigeneratedloginstartresult).



## Module: names

Flat `UPPER_SNAKE` aliases matching historical `protocol::methods` usage.

Prefer the namespaced modules (`login_start::NAME`) in new code; these
constants remain for call sites that imported a single `methods` prelude.



## Module: poll_events

Poll for plugin-observed external users after [`start`](methods/start.md).

Result: [`crate::EventPollResult`](generated.md#bookclerk_plugin_abigeneratedeventpollresult).



## Module: publish

Append one guest event to the host outbox (Cap'n Proto `EventPublisher.publish`).

Params: [`crate::PluginEvent`](generated.md#bookclerk_plugin_abigeneratedpluginevent). Result: [`crate::PublishOk`](generated.md#bookclerk_plugin_abigeneratedpublishok).



## Module: purchase_hint

Resolve a purchase URL / price hint for a catalog product.

Params: [`crate::PurchaseHintParams`](generated.md#bookclerk_plugin_abigeneratedpurchasehintparams). Result: [`crate::PurchaseHint`](generated.md#bookclerk_plugin_abigeneratedpurchasehint).



## Module: put

Put a small object (cover/sidecar) into an output destination.

Typed Cap'n Proto `Destination.put` (`crate::Destination::put`).



## Module: queue

Deliver one ordered batch of queue messages (Cap'n Proto `QueueConsumer.queue`).

Params: [`crate::QueueBatch`](generated.md#bookclerk_plugin_abigeneratedqueuebatch). Result: one [`crate::QueueResult`](generated.md#bookclerk_plugin_abigeneratedqueueresult) per message.



## Module: scan

Scan a source storefront library; host upserts returned [`crate::ScanBook`](generated.md#bookclerk_plugin_abigeneratedscanbook) rows.

Params: [`crate::ScanParams`](generated.md#bookclerk_plugin_abigeneratedscanparams). Result: [`crate::ScanSummary`](generated.md#bookclerk_plugin_abigeneratedscansummary).



## Module: scan_library

Trigger an integration library scan against a remote server (e.g. ABS).

Params: [`crate::ScanLibraryParams`](generated.md#bookclerk_plugin_abigeneratedscanlibraryparams).



## Module: search_catalog

Search a storefront catalog for purchase / discovery UIs.

Params: [`crate::SearchCatalogParams`](generated.md#bookclerk_plugin_abigeneratedsearchcatalogparams). Result: list of [`crate::CatalogHit`](generated.md#bookclerk_plugin_abigeneratedcataloghit).



## Module: send

Send one message into a granted queue (Cap'n Proto `QueueProducer.send`).

Params: [`crate::QueueSendMessage`](generated.md#bookclerk_plugin_abigeneratedqueuesendmessage). Result: [`crate::QueueSendOk`](generated.md#bookclerk_plugin_abigeneratedqueuesendok).



## Module: send_batch

Send a batch of messages into a granted queue (Cap'n Proto
`QueueProducer.sendBatch`).

Params: a list of [`crate::QueueSendMessage`](generated.md#bookclerk_plugin_abigeneratedqueuesendmessage) plus a default delay.
Result: [`crate::QueueSendOk`](generated.md#bookclerk_plugin_abigeneratedqueuesendok).



## Module: shutdown

Graceful guest teardown before the host closes the transport.



## Module: start

Start background watchers (Cap'n Proto `RemoteLibrary.start`).



## Module: stop

Stop background watchers started by [`start`](methods/start.md) (Cap'n Proto `RemoteLibrary.stop`).



## Module: sync_listening

Pull listening-progress snapshots; host upserts tagged with the plugin id.

Result: [`crate::SyncListeningResult`](generated.md#bookclerk_plugin_abigeneratedsynclisteningresult).



