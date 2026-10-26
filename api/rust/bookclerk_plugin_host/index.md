# bookclerk_plugin_host

External plugin host for Bookclerk (`bookclerk-plugin-host`).

Discovers staged guests under `$BOOKCLERK_FILES_DIR/plugins/` and spawns
them over the Workers RPC ABI through the mandatory `bookclerk-workerd`
front door (`api_version = 3` Cap'n Proto on stdio).

Production hosts never link ordinary plugin implementation crates.
First-party storefronts (Audible, Libro.fm, …) are staged guests, the
same path third-party plugins use. Database adapters own physical
lowering behind the typed `databaseAdapter` capability (`openSession`,
`dropUnit`); production hosts do not link `bookclerk-plugin-database-*`
crates to perform product behavior.

External plugins are **untrusted** relative to the host: the host never
passes `library.db` / `master.key` / the files-dir root, clears
secret-bearing env on spawn, and mediates credentials + library upserts.
Operators must `bookclerk plugins approve` domains/bindings before enable;
the same covering grant is required again at every external spawn and at
privileged delivery points (`config` / `secrets` / `work_fs` / `oauth`).
Durable identity is provenance-qualified `PluginKey`;
the manifest `id` is a display / CLI alias.

Host binaries should depend on **this** crate for registration — not on
individual store crates.

## Guest SDK

Third-party Rust plugins should depend on [`bookclerk_plugin_sdk`](../bookclerk_plugin_sdk/index.md), not this
crate. This host crate re-exports the protocol types for in-tree convenience.

See `docs/plugins/README.md`, `docs/adr/plugin-workers-rpc-workerd.md`, and
`docs/plugins/registry.md`.

| Item | Kind | Summary |
| --- | --- | --- |
| [`DeploymentRuntime`](#deploymentruntime) | trait | Install health and spawn for one deployment row. |
| [`CallbackProxy`](#callbackproxy) | struct | Active host callback proxy for one OAuth session. |
| [`GrantedEventConsumer`](#grantedeventconsumer) | struct | One approved event-consumer subscription (structural authority). |
| [`PendingStructural`](#pendingstructural) | struct | Newly requested structural capabilities awaiting operator consent. |
| [`PluginGrant`](#plugingrant) | struct | One approved grant snapshot for a provenance-qualified plugin. |
| [`PluginGrantStore`](#plugingrantstore) | struct | On-disk grant store. |
| [`AuthorizedLocalPackages`](#authorizedlocalpackages) | struct | Operator-placed archives keyed by canonical plugin key. |
| [`DeploymentSpawn`](#deploymentspawn) | struct | Spawn request after config resolution. |
| [`LiveDeploymentRuntime`](#livedeploymentruntime) | struct | Production runtime: spawn through the existing host adapters and register the session on the live registry. |
| [`LocalPackage`](#localpackage) | struct | Local archive the reconciler may install when the ledger has no row. |
| [`DiscoveredPlugin`](#discoveredplugin) | struct | A discovered plugin ready to spawn. |
| [`EnrolledPlugin`](#enrolledplugin) | struct | One plugin after enrollment on this node. |
| [`EventOutbox`](#eventoutbox) | struct | Host-owned facts a session needs before it can hand a guest an `EVENTS` binding: the outbox store plus the producer allowlist. |
| [`OutboxEventPublisher`](#outboxeventpublisher) | struct | `EventPublisher` server for one invocation: validates a guest `PluginEvent` and appends it to the library outbox. |
| [`DatabaseRegistry`](#databaseregistry) | struct | Long-lived external database plugin for the active `[database].plugin`. |
| [`DestinationRegistry`](#destinationregistry) | struct | Long-lived external output plugins loaded at host startup. |
| [`ExternalDatabase`](#externaldatabase) | struct | External database backend spawned for `[database].plugin`. |
| [`ExternalIntegration`](#externalintegration) | struct | External integration backed by a discovered plugin binary. |
| [`ExternalSource`](#externalsource) | struct | External content source backed by a discovered plugin binary. |
| [`HostControlPlane`](#hostcontrolplane) | struct | Open control plane for this process. |
| [`NodeEnrollment`](#nodeenrollment) | struct | Result of `enroll_node`: the bootstrap half of a host control plane. |
| [`PresentDeployment`](#presentdeployment) | struct | A deployment the database wants present on this node, joined with the instance row and the local install it will run. |
| [`AddedVersion`](#addedversion) | struct | What `add_package_version` recorded. |
| [`InstallRequest`](#installrequest) | struct | An operator's `install`: what `bookclerk plugins install` and `POST /api/config/plugin-versions` both install. |
| [`InstalledVersion`](#installedversion) | struct | What `install_version` did. |
| [`VersionRequest`](#versionrequest) | struct | One package to install. |
| [`PreparedOpen`](#preparedopen) | struct | Bindings passed to `PluginWorker.open`, plus the spawn config table. |
| [`KvNamespaces`](#kvnamespaces) | struct | Host-owned facts a session needs before it can hand a guest its KV bindings: the library store plus the granted rows. |
| [`LibraryKvNamespace`](#librarykvnamespace) | struct | `KvNamespace` server for one `(plugin_key, namespace)` on the library. |
| [`OutboxQueueProducer`](#outboxqueueproducer) | struct | `QueueProducer` server for one binding of one invocation: validates a guest `QueueSendMessage` and appends it to the library outbox. |
| [`QueueOutbox`](#queueoutbox) | struct | Host-owned facts a session needs before it can hand a guest its queue producer bindings: the outbox store plus the granted `(binding, queue)` pairs. |
| [`BookclerkPackageMetadata`](#bookclerkpackagemetadata) | struct | `[package.metadata.bookclerk]` published on crates.io. |
| [`PluginCatalogEntry`](#plugincatalogentry) | struct | One catalog hit from crates.io (or a curated index). |
| [`PluginCrateName`](#plugincratename) | struct | Parsed crates.io crate name for a Bookclerk plugin. |
| [`RemoteNarrowing`](#remotenarrowing) | struct | What a host operator takes out of an owner plugin's request. |
| [`ExecutorIdentity`](#executoridentity) | struct | Expanded executor identity. Pooling is an optimization; correctness must not depend on a PID surviving. |
| [`PluginSession`](#pluginsession) | struct | Host-side plugin session (one jailed child + one vat thread). |
| [`PluginStorage`](#pluginstorage) | struct | `StorageBackend` over a plugin destination capability (streams, fail-closed scalars). |
| [`RpcBackupOps`](#rpcbackupops) | struct | Send-safe backup primitives over a vat-held adapter session (or its open txn). |
| [`SessionServices`](#sessionservices) | struct | Host services a session hands its guest as `Bindings` on every `PluginWorker.open`. |
| [`SpawnPlan`](#spawnplan) | struct | Everything the spawn path needs to start one guest. |
| [`WorkerdFrontDoor`](#workerdfrontdoor) | struct | Resolved `bookclerk-workerd` launcher and the pinned `workerd` beside it. |
| [`SpawnHealth`](#spawnhealth) | enum | Result of spawn plus the health RPC. |
| [`PluginError`](#pluginerror) | enum | Failures from discovery, RPC, consent, or plugin adapters. |
| [`Activate`](#activate) | enum | When installing moves the active pointer. |
| [`RemoteApprovalError`](#remoteapprovalerror) | enum | Why `approve_remote_request` refused, by cause, so a caller (the operator API) can tell "review again" from "fix the input" from "retry later". |
| [`RemoteConsentState`](#remoteconsentstate) | enum | How this host stands on one owner plugin's request. |
| [`GuestRuntimeKind`](#guestruntimekind) | enum | Process tree the jail will contain, as reported in `crate::ExecutorIdentity`. |
| [`SpawnTransport`](#spawntransport) | enum | Which transport a spawn uses to reach the guest. |
| [`apply_grant_store`](#apply_grant_store) | function | Reconcile every live session against a loaded **persisted** grant store. |
| [`authority_revision`](#authority_revision) | function | Canonical SHA-256 of security-relevant **effective** authority. |
| [`fence_plugin_key`](#fence_plugin_key) | function | Fences every live session for `plugin_key` (grant/authority change). |
| [`fence_stale_grant_revisions`](#fence_stale_grant_revisions) | function | Fences live sessions whose **persisted** `crate::grant_revision` is not current. |
| [`fence_stale_sessions`](#fence_stale_sessions) | function | Fences live sessions for `plugin_key` whose revision is not `current_revision`. |
| [`is_fenced`](#is_fenced) | function | True when this cancel flag has been fenced. |
| [`live_session_count`](#live_session_count) | function | Sessions currently registered with the process-wide authority table. |
| [`notify_grants_changed`](#notify_grants_changed) | function | Wake the grant watcher in **this** process (same-process save / tests). |
| [`reconcile_grants_from_disk`](#reconcile_grants_from_disk) | function | Load `plugin-grants.json` and fence sessions that no longer match. |
| [`register_session`](#register_session) | function | Registers a live session and returns its cancel flag. |
| [`register_session_revisions`](#register_session_revisions) | function | Registers a live session with distinct persisted vs effective revisions. |
| [`register_session_with_shutdown`](#register_session_with_shutdown) | function | Registers a live session whose shutdown hook runs the instant it is fenced. |
| [`spawn_grant_watcher`](#spawn_grant_watcher) | function | Spawn the daemon grant watcher (fire-and-forget). |
| [`unregister_session`](#unregister_session) | function | Drops a previously registered cancel flag from the live set. |
| [`watch_grants_loop`](#watch_grants_loop) | function | Watch `plugin-grants.json` until the process exits. |
| [`install_bundled_platform_plugins`](#install_bundled_platform_plugins) | function | Installs bundled platform plugins from beside the running executable into `files_dir`. |
| [`active_processes_for`](#active_processes_for) | function | Absolute Spec `active_processes` from runtime overhead + extra budget. |
| [`canonical_event_filter`](#canonical_event_filter) | function | Minified JSON for an event filter; `null` / missing → `None`. |
| [`consent_request`](#consent_request) | function | Build the consent request surface from a manifest and its PluginKey. |
| [`consent_request_alias`](#consent_request_alias) | function | Build the consent request surface from a manifest (alias only; tests). |
| [`consent_summary`](#consent_summary) | function | Human-readable consent lines for CLI/UI. |
| [`consumers_cover`](#consumers_cover) | function | True when every requested consumer is covered by some approved consumer. |
| [`cores_to_percent`](#cores_to_percent) | function | Convert cores to Spec/config percent-of-one-core, rounded to 0.01 core. |
| [`database_binding_name`](#database_binding_name) | function | The binding name when a grant entry is a named database binding. |
| [`effective_cpu_cores`](#effective_cpu_cores) | function | Clamp operator/manifest cores into `0.01..=``host_cpu_cores_max`. |
| [`effective_cpu_rate_percent`](#effective_cpu_rate_percent) | function | Resolved jail Spec CPU rate as percent of one logical CPU. |
| [`effective_disk_budget_bytes`](#effective_disk_budget_bytes) | function | Resolved disk budget in bytes for `data/` and `tmp/` checks. |
| [`effective_disk_mib`](#effective_disk_mib) | function | Resolved disk budget in MiB (default + host clamp). |
| [`effective_extra_processes`](#effective_extra_processes) | function | Clamp an operator/manifest extra-process budget (`0..=``PLUGIN_JAIL_EXTRA_PROCESSES_MAX`). |
| [`effective_grant`](#effective_grant) | function | Spawn/delivery grant: stored approval is authoritative, host-normalized. |
| [`effective_memory_mib`](#effective_memory_mib) | function | Resolved jail Spec memory ceiling in MiB. |
| [`format_cpu_cores`](#format_cpu_cores) | function | Format cores for operator-facing text (`0.80`). |
| [`grant_covers`](#grant_covers) | function | True when a stored grant is usable for enable/spawn of this PluginKey. |
| [`grant_has_binding`](#grant_has_binding) | function | True when `grant.bindings` contains `name` (case-insensitive). |
| [`grant_revision`](#grant_revision) | function | Canonical SHA-256 of a grant's security-relevant fields (not presentation). |
| [`grant_within_ceiling`](#grant_within_ceiling) | function | True when an existing grant stays within the manifest request surface. |
| [`granted_consumers_from_manifest`](#granted_consumers_from_manifest) | function | Consumers declared by `plugin.toml` as structural grant entries. |
| [`granted_database_bindings`](#granted_database_bindings) | function | Consented database binding names on a grant, in manifest-set order. |
| [`granted_jobs_from_manifest`](#granted_jobs_from_manifest) | function | Job trigger types declared by `[triggers].jobs`. |
| [`granted_queue_consumers`](#granted_queue_consumers) | function | Consented queue consumer names on a grant. |
| [`host_cpu_cores_max`](#host_cpu_cores_max) | function | Host max jail CPU in cores (`logical_cpus` as a 2 d.p. float). |
| [`host_cpu_rate_max`](#host_cpu_rate_max) | function | Host CPU rate ceiling in one-core percent units (`logical_cpus × 100`). |
| [`host_overlay_config_digest`](#host_overlay_config_digest) | function | SHA-256 of host config that participates in implied network overlays. |
| [`inject_workerd_grant_env`](#inject_workerd_grant_env) | function | Injects effective-grant overrides for `bookclerk-workerd` into a guest command. |
| [`is_platform_envelope`](#is_platform_envelope) | function | True when `grant` stays inside the platform installer envelope (deny network, only `config` / `work_fs` bindings): the request a Bookclerk platform plugin may receive without the operator being asked. |
| [`jail_process_overhead`](#jail_process_overhead) | function | Fixed payload occupancy for the launcher / primary guest tree. |
| [`network_compatible`](#network_compatible) | function | Returns true when a stored network mode can satisfy a requested network mode. |
| [`node_logical_cpus`](#node_logical_cpus) | function | Logical CPUs visible to this process (at least 1). |
| [`overlay_host_implied_network`](#overlay_host_implied_network) | function | Overlay TCP + CIDR implied by host configuration (not persisted). |
| [`pending_structural`](#pending_structural) | function | Structural capabilities the current manifest requests that the stored grant does not yet authorize. |
| [`percent_to_cores`](#percent_to_cores) | function | Convert Spec/config percent-of-one-core (100 = 1.00 core) to cores (2 d.p.). |
| [`queue_consumer_grant`](#queue_consumer_grant) | function | Grant token for consuming `queue`. |
| [`queue_producer_grant`](#queue_producer_grant) | function | Grant token for sending through the `[[queues.producers]]` binding `binding`. |
| [`reconcile_host_overlay_authority`](#reconcile_host_overlay_authority) | function | Recomputes effective overlay authority and fences sessions that no longer match. |
| [`require_binding`](#require_binding) | function | Fail closed when a delivery site needs a binding the covering grant lacks. |
| [`require_grant`](#require_grant) | function | Require an operator grant before enable **or** every external spawn. |
| [`spawn_config_for_grant`](#spawn_config_for_grant) | function | Spawn `config` payload: non-empty settings only when the grant includes `config`. |
| [`spawn_grant`](#spawn_grant) | function | Resolve the covering grant for spawn: verified platform auto-grant, else `require_grant`. |
| [`validate_approved_grant`](#validate_approved_grant) | function | Validates and normalizes an operator-supplied grant against host hard caps. |
| [`validate_described_capabilities`](#validate_described_capabilities) | function | Reject `describe()` capability claims that exceed the manifest and the covering grant. |
| [`validate_described_config_options`](#validate_described_config_options) | function | Validates `describe().configOptions` against the manifest `[[settings]]`. |
| [`windows_session_active_processes`](#windows_session_active_processes) | function | Windows outer session Job active-process cap: baseline 5 plus `extra`. |
| [`search_crates_io`](#search_crates_io) | function | Search crates.io for Bookclerk plugins. |
| [`load_authorized_local_packages`](#load_authorized_local_packages) | function | Loads operator-placed archives under `$FILES_DIR/plugin-packages/<name>/`. |
| [`reconcile_local_deployments`](#reconcile_local_deployments) | function | Reconciles every present deployment for `node_id`. |
| [`build_acquire_destinations`](#build_acquire_destinations) | function | Build destination backends for acquire from the deployed output plugins. |
| [`build_storage_backend`](#build_storage_backend) | function | Build listing/storage backends from the deployed output plugins. |
| [`discover_plugins`](#discover_plugins) | function | Discover plugins under the configured search directories. |
| [`first_party_database_kind`](#first_party_database_kind) | function | First-party database adapter kind when provenance + PluginKey match. |
| [`identity_matches_occupancy`](#identity_matches_occupancy) | function | True when `plugin_key` / `alias` is the occupant named by `spec`. |
| [`is_first_party_audiobookshelf`](#is_first_party_audiobookshelf) | function | True when this install is the verified Bookclerk Audiobookshelf integration. |
| [`is_first_party_local_output`](#is_first_party_local_output) | function | True when this install is the verified Bookclerk local-filesystem destination. |
| [`is_first_party_local_output_key`](#is_first_party_local_output_key) | function | True when `key` names the Bookclerk local-filesystem destination. |
| [`is_first_party_s3_output`](#is_first_party_s3_output) | function | True when this install is the verified Bookclerk S3 destination. |
| [`occupancy_matches_alias`](#occupancy_matches_alias) | function | True when occupancy `spec` names the display alias (or kind token) `alias`. |
| [`occupancy_names_alias`](#occupancy_names_alias) | function | True when occupancy `spec` names `alias` among `discovered` installs. |
| [`occupancy_spec`](#occupancy_spec) | function | Config occupancy selector (`plugin = "…"`), or `alias` when the field is empty. |
| [`plugin_matches_occupancy`](#plugin_matches_occupancy) | function | True when `plugin` is the occupant named by `spec` (PluginKey or alias). |
| [`plugin_search_dirs`](#plugin_search_dirs) | function | Resolve search roots: `BOOKCLERK_PLUGIN_DIRS` then `$FILES_DIR/plugins`. |
| [`resolve_plugin_ref`](#resolve_plugin_ref) | function | Resolves `spec` to a discovered plugin. |
| [`resolve_plugin_slot`](#resolve_plugin_slot) | function | Resolves `spec` among `plugins` without treating a vacant slot as an error. |
| [`settings_table`](#settings_table) | function | Bootstrap connection settings for a database adapter from `config.toml`. |
| [`settings_table_for`](#settings_table_for) | function | `settings_table` for one explicit handler `family`. |
| [`stamp_occupancy_plugin_key`](#stamp_occupancy_plugin_key) | function | Records `plugin` as the `[database].plugin` occupant. |
| [`upgrade_unique_alias_occupancy`](#upgrade_unique_alias_occupancy) | function | Rewrites a unique `[database].plugin` alias to the canonical PluginKey. |
| [`enroll_local_plugins`](#enroll_local_plugins) | function | Enrolls every discovered plugin on `node_id`. |
| [`enroll_one`](#enroll_one) | function | Enrolls one discovered plugin. See the module docs for the steps. |
| [`has_covering_grant`](#has_covering_grant) | function | True when the operator already consented to this plugin, or it is a verified platform guest that grants itself on first spawn. |
| [`is_bootstrap_only`](#is_bootstrap_only) | function | True when the plugin is a database adapter and nothing else: those stay bootstrap (`[database]` in `config.toml`) in this phase. |
| [`backup_adapter_id`](#backup_adapter_id) | function | Adapter id recorded on a backup (diagnostic; restore is capability-driven). |
| [`database_connect_bindings`](#database_connect_bindings) | function | Builds the `bookclerk_plugin_sdk::BindingValues` for the adapter's `PluginWorker.open` (then `databaseAdapter.openSession`). |
| [`export_registered_plugin_units`](#export_registered_plugin_units) | function | Exports every registered plugin-owned database binding as SeaORM sessions. |
| [`load_external_database`](#load_external_database) | function | Discover and spawn the external database plugin matching `[database].plugin`. |
| [`migrate_database_plugin`](#migrate_database_plugin) | function | Copy library data from one database plugin backend to another. |
| [`migrate_library_schema`](#migrate_library_schema) | function | Opens the active database guest without auto-apply and migrates to `target`. |
| [`open_library_store`](#open_library_store) | function | Open `bookclerk_library::LibraryStore` via the external database guest (required). |
| [`open_library_store_for_plugin`](#open_library_store_for_plugin) | function | Open the library for a specific `[database].plugin` id (ignoring the active config value). |
| [`restore_plugin_backup_units`](#restore_plugin_backup_units) | function | Restores plugin binding units onto the **current** adapter. |
| [`allow_credential_login`](#allow_credential_login) | function | `allow_credential_login` from the resolved `CONFIG`; absent means allowed. |
| [`enroll_node`](#enroll_node) | function | Bootstraps the control plane on an open library and enrolls every local plugin, without loading any guest. |
| [`open_host_control_plane`](#open_host_control_plane) | function | Opens the library database and the control plane for `config`. |
| [`open_host_control_plane_with_store`](#open_host_control_plane_with_store) | function | `open_host_control_plane` on a library that is already open. |
| [`refresh_present`](#refresh_present) | function | Loads an already-open host control plane's deployments again, for callers that changed desired state and want a fresh view without re-enrolling. |
| [`add_package_version`](#add_package_version) | function | Verifies `package`, records a `PluginVersion`, and moves the active pointer per `VersionRequest::activate`. |
| [`enroll_version`](#enroll_version) | function | Creates the default instance of a installed plugin with its schema defaults, as enrollment does for a host-local plugin. Idempotent. |
| [`grant_covers_version`](#grant_covers_version) | function | True when `grant` still covers everything `version` requests, so an update may carry an earlier approval forward. A widened request needs a fresh `plugins approve`. |
| [`host_artifact`](#host_artifact) | function | The version artifact this node runs. |
| [`install_plugin_key`](#install_plugin_key) | function | The plugin key an install installs under. |
| [`install_version`](#install_version) | function | Installs an operator's package as a version, carries an earlier approval forward when it still covers the new version (or approves when asked), and enrolls the plugin's default instance. |
| [`is_sha256_hex`](#is_sha256_hex) | function | True for a canonical SHA-256 digest: exactly 64 lowercase hex digits. |
| [`materialize_owner_version`](#materialize_owner_version) | function | `materialize_version` for a version an owner cluster deploys to this host: a `blob:` archive is read from the library as usual (the caller fetched it over the owner link and checked its digest), and any other URL is fetched only as an owner `https:` URL (`bookclerk_plugin_catalog::fetch_owner_artifact`: no `file:`, no local or private hosts, every redirect hop checked the same way). |
| [`materialize_version`](#materialize_version) | function | Makes the active version's tree available under `plugins/.versions/<payload digest>` and returns that path. |
| [`open_installed_plugin`](#open_installed_plugin) | function | Opens a materialized version tree as a spawnable plugin carrying the version's identity and consent grant. |
| [`sweep_version_cache`](#sweep_version_cache) | function | Removes cached digests not in `keep` and older than `grace`, plus stale version staging directories. Returns how many entries were removed. |
| [`version_cache_path`](#version_cache_path) | function | Cached tree of one payload-root digest. |
| [`version_cache_root`](#version_cache_root) | function | Cache root: `<files_dir>/plugins/.versions`. |
| [`version_consent_request`](#version_consent_request) | function | Consent the version's manifest requests (what `plugins approve` shows). |
| [`version_grant`](#version_grant) | function | The consent grant recorded on `version`. |
| [`configured_alias_occupants`](#configured_alias_occupants) | function | Occupancy (PluginKey, alias, path) under every configured discovery root. |
| [`install_from_manifest_with_configured_aliases`](#install_from_manifest_with_configured_aliases) | function | Catalog install after `reject_configured_alias_collision`. |
| [`install_local_archive_with_configured_aliases`](#install_local_archive_with_configured_aliases) | function | Local-archive install after `reject_configured_alias_collision`. |
| [`reject_configured_alias_collision`](#reject_configured_alias_collision) | function | Rejects installing `incoming_alias` when a different PluginKey already owns it anywhere in the configured discovery universe. |
| [`binding_grant_flags`](#binding_grant_flags) | function | `config` / `secrets` flags of the plugin's covering grant. |
| [`prepare_open_for_instance`](#prepare_open_for_instance) | function | Resolves open bindings for `instance_id` of `plugin`. |
| [`prepared_open_from_resolved`](#prepared_open_from_resolved) | function | Builds open bindings from a deployment's already resolved payloads. |
| [`guest_reservation`](#guest_reservation) | function | CPU and memory one guest may use at most on this host: its grant's ceilings after `[plugins.jail]`, the numbers its jail enforces. The cluster scheduler accounts host budgets against these. |
| [`plugin_data_dir`](#plugin_data_dir) | function | The private directory a plugin keeps state in. |
| [`family_keyword`](#family_keyword) | function | Family-specific crates.io keyword (`bookclerk-source`, …). |
| [`host_target_triple`](#host_target_triple) | function | Host target triple used when selecting release assets. |
| [`validate_plugin_id`](#validate_plugin_id) | function | Validate plugin id segment used in crate names and `plugin.toml`. |
| [`approve_remote_request`](#approve_remote_request) | function | Approves the request recorded from `cluster_id` for `plugin_key`, exactly as it was reviewed: `reviewed_digest` must be the recorded request's digest (`bookclerk_library::control_plane::RemotePluginConsent::request_digest`, shown with the request), and the consent is that request minus `narrowing` (`remote_consent`). The write is a compare-and-swap on the recorded request, so a request that changed after it was read is refused rather than approved unseen. |
| [`narrow_remote_grant`](#narrow_remote_grant) | function | The grant a guest runs with: `requested ∩ consent`. |
| [`remote_consent`](#remote_consent) | function | The consent an operator gives to `requested` with `narrowing` applied. |
| [`remote_consent_state`](#remote_consent_state) | function | Evaluates a stored consent row's grants: the current request (`requested_json`), the request as reviewed and the consent (both `None` while unapproved). |
| [`remote_consent_view`](#remote_consent_view) | function | The one JSON view of a consent row, shared by the operator API and the CLI so both answer the same fields: the request (version, grant, the archive this host would fetch, and its `request_digest`), the approval (`approved_version`, `grant`, `approved_at`, `approved_by`), `revoked_at`, `updated_at`, and the `state` (`remote_row_consent_state`). A stored grant that does not parse shows as `null`. |
| [`remote_platform_auto_approval`](#remote_platform_auto_approval) | function | True when a platform plugin's request may be approved without asking: the key is a Bookclerk platform key, this host's own install of that key is platform-bundled with the same payload digest as the owner's artifact (`local_bundled_payload`, from this host's provenance check), and the request stays inside the platform envelope (deny network, only `config` / `work_fs`). |
| [`remote_request_within`](#remote_request_within) | function | True when `requested` asks for nothing outside `reviewed`: the same plugin, every capability set a subset, network no wider (outbound only if reviewed outbound), undeclared redirects only if reviewed, and no larger budget. |
| [`remote_row_consent_state`](#remote_row_consent_state) | function | Evaluates a stored consent row: `remote_consent_state` over its grants, except that an approval of another archive (the owner changed the artifact URL or digest since the operator approved) needs approval again, whatever the grant. |
| [`plugin_instance_key`](#plugin_instance_key) | function | Isolation key: different accounts never share a plugin isolate. |
| [`note_spawn_stage`](#note_spawn_stage) | function | Record one secret-free startup line. |
| [`recent_spawn_diagnostics`](#recent_spawn_diagnostics) | function | Bounded startup log for the current process. |
| [`retry_abandoned_acl_journals`](#retry_abandoned_acl_journals) | function | Retry abandoned journals for every plugin under `files_dir/plugin-state`. |
| [`revoke_plugin_state_before_purge`](#revoke_plugin_state_before_purge) | function | Revoke this plugin's journal before `--purge-state` deletes the directory. |
| [`SessionShutdown`](#sessionshutdown) | type alias | Shutdown hook invoked the moment a live session is fenced. |
| [`Result`](#result) | type alias | Result alias for `PluginError`. |
| [`GuestDatabaseFactory`](#guestdatabasefactory) | type alias | Send-safe constructor for a per-binding `bookclerk_plugin_sdk::GuestDatabase`. |
| [`GRANT_WATCH_INTERVAL`](#grant_watch_interval) | constant | How often the daemon re-reads `plugin-grants.json` when no in-process notify fires. |
| [`GRANTS_FILE`](#grants_file) | constant | Filename under `$BOOKCLERK_FILES_DIR` for persisted grants. |
| [`GRANT_SCHEMA_VERSION`](#grant_schema_version) | constant | Current persisted `PluginGrant` document version. |
| [`KNOWN_HOST_BINDINGS`](#known_host_bindings) | constant | Host binding names operators may grant (widen or narrow). |
| [`PLUGIN_JAIL_ACTIVE_PROCESSES_MAX`](#plugin_jail_active_processes_max) | constant | Absolute OS jail PID ceiling (`overhead + extra`, inclusive). |
| [`PLUGIN_JAIL_CPU_CORES_DEFAULT`](#plugin_jail_cpu_cores_default) | constant | Default jail CPU as cores (`PLUGIN_JAIL_CPU_RATE_DEFAULT` / 100). |
| [`PLUGIN_JAIL_CPU_RATE_DEFAULT`](#plugin_jail_cpu_rate_default) | constant | Default jail Spec CPU rate percent for confined guests (one-core units). |
| [`PLUGIN_JAIL_CPU_RATE_MAX`](#plugin_jail_cpu_rate_max) | constant | Absolute ceiling used only when host CPU detection is unavailable in docs. |
| [`PLUGIN_JAIL_EXTRA_PROCESSES_DEFAULT`](#plugin_jail_extra_processes_default) | constant | Default **extra** processes/threads beyond launcher overhead. |
| [`PLUGIN_JAIL_EXTRA_PROCESSES_MAX`](#plugin_jail_extra_processes_max) | constant | Host hard cap for per-plugin extra process budget. |
| [`PLUGIN_JAIL_MEMORY_MIB_DEFAULT`](#plugin_jail_memory_mib_default) | constant | Default jail Spec memory ceiling (MiB) for confined guests. |
| [`PLUGIN_JAIL_MEMORY_MIB_MAX`](#plugin_jail_memory_mib_max) | constant | Host hard cap for per-plugin jail memory overrides (MiB). |
| [`PLUGIN_STATE_BUDGET_MIB_DEFAULT`](#plugin_state_budget_mib_default) | constant | Default per-plugin `data/` and `tmp/` disk budget (MiB each). |
| [`PLUGIN_STATE_BUDGET_MIB_MAX`](#plugin_state_budget_mib_max) | constant | Host hard cap for per-plugin disk budget overrides (MiB). |
| [`QUEUE_CONSUMER_BINDING_PREFIX`](#queue_consumer_binding_prefix) | constant | Grant `bindings` entry prefix for one `[[queues.consumers]]` row (`queue_consumer:<queue>`): the host may deliver that queue to the plugin. |
| [`QUEUE_PRODUCER_BINDING_PREFIX`](#queue_producer_binding_prefix) | constant | Grant `bindings` entry prefix for one `[[queues.producers]]` binding (`queue:<BINDING>`): the plugin may send through that `env` binding. |
| [`WINDOWS_SESSION_PROCESS_BASELINE`](#windows_session_process_baseline) | constant | Windows outer session Job processes before the granted extra budget. |
| [`WORKERD_GRANT_CPU_MS_ENV`](#workerd_grant_cpu_ms_env) | constant | Grant CPU budget (ms) for workerd logging / limit narrowing. |
| [`WORKERD_GRANT_DOMAINS_ENV`](#workerd_grant_domains_env) | constant | Comma-separated grant domain allowlist for workerd egress. |
| [`WORKERD_GRANT_NETWORK_MODE_ENV`](#workerd_grant_network_mode_env) | constant | Env keys consumed by `bookclerk-workerd` (`grant.rs`) at isolate start. |
| [`WORKERD_GRANT_POLICY_ENV`](#workerd_grant_policy_env) | constant | Canonical JSON `EgressPolicy` (preferred over piecemeal domain env vars). |
| [`WORKERD_GRANT_SUBREQUESTS_ENV`](#workerd_grant_subrequests_env) | constant | Grant subrequest budget injected into workerd `EGRESS_POLICY`. |
| [`AUTHORIZED_PACKAGE_DIR`](#authorized_package_dir) | constant | Directory under the files dir that holds operator-placed package archives. |
| [`VERSION_CACHE_DIR`](#version_cache_dir) | constant | Directory under `plugins/` holding materialized version trees. |
| [`VERSION_CACHE_GRACE`](#version_cache_grace) | constant | How long an unreferenced cached digest survives before the sweep. |
| [`KV_GRANT_BINDING`](#kv_grant_binding) | constant | Grant token that authorizes every `[[kv_namespaces]]` row of a plugin. |
| [`CRATE_NAME_PREFIX`](#crate_name_prefix) | constant | Required crates.io name prefix: `bookclerk-plugin-{family}-{id}`. |
| [`PRODUCT_KEYWORD`](#product_keyword) | constant | Shared crates.io keyword (`bookclerk`) expected on published plugin crates. |
| [`REGISTRY_KEYWORD`](#registry_keyword) | constant | Keyword every published plugin crate should include. |
| [`HOST_SHARED_ACCOUNT`](#host_shared_account) | constant | Registry-loaded source/integration guests (not a user account). |
| [`OPERATOR_ACCOUNT`](#operator_account) | constant | Isolation key: different accounts never share a plugin isolate. |
| [`WORKERD_BIN_ENV`](#workerd_bin_env) | constant | Override for the pinned Cloudflare `workerd` binary, honoured by `bookclerk-workerd` and forwarded to it by the host. |
| [`WORKERD_LAUNCHER_ENV`](#workerd_launcher_env) | constant | Override for the `bookclerk-workerd` launcher path (tests / packaging). |
| [`TEST_CHANNEL_IDENT_ENV`](#test_channel_ident_env) | constant | Set to `1` or `true` in the host process to give each proxy a channel tag. |
| [`TEST_CHANNEL_TAG_FILE`](#test_channel_tag_file) | constant | Session-directory file holding the tag when `TEST_CHANNEL_IDENT_ENV` is set. |
| [`TEST_INJECT_EXTRA_ENDPOINT_ENV`](#test_inject_extra_endpoint_env) | constant | Host-process description of one extra endpoint to add to the guest handoff. |
| [`TEST_OBSERVE_EXTRA_ENDPOINT_ENV`](#test_observe_extra_endpoint_env) | constant | Host-process request to publish candidate-endpoint observation metadata. |

## Re-exports

- [`bookclerk_plugin_sdk::protocol`](../bookclerk_plugin_sdk/protocol.md)
- [`bookclerk_plugin_sdk::methods`](../bookclerk_plugin_sdk/methods.md)
- [`bookclerk_plugin_sdk::CatalogHit`](../bookclerk_plugin_abi/generated.md#cataloghit)
- [`bookclerk_plugin_sdk::CliArg`](../bookclerk_plugin_abi/generated.md#cliarg)
- [`bookclerk_plugin_sdk::CliArgKind`](../bookclerk_plugin_abi/generated.md#cliargkind)
- [`bookclerk_plugin_sdk::CliArgSpec`](../bookclerk_plugin_abi/generated.md#cliargspec)
- [`bookclerk_plugin_sdk::CliCommandSpec`](../bookclerk_plugin_abi/generated.md#clicommandspec)
- [`bookclerk_plugin_sdk::CliInvokeParams`](../bookclerk_plugin_abi/generated.md#cliinvokeparams)
- [`bookclerk_plugin_sdk::CliInvokeResult`](../bookclerk_plugin_abi/generated.md#cliinvokeresult)
- [`bookclerk_plugin_sdk::CliSchema`](../bookclerk_plugin_abi/generated.md#clischema)
- [`bookclerk_plugin_sdk::EventPollResult`](../bookclerk_plugin_abi/generated.md#eventpollresult)
- [`bookclerk_plugin_sdk::ExpandCandidatesParams`](../bookclerk_plugin_abi/generated.md#expandcandidatesparams)
- [`bookclerk_plugin_sdk::ExternalUser`](../bookclerk_plugin_abi/generated.md#externaluser)
- [`bookclerk_plugin_sdk::FetchTitleParams`](../bookclerk_plugin_abi/generated.md#fetchtitleparams)
- [`bookclerk_plugin_sdk::ListeningProgress`](../bookclerk_plugin_abi/generated.md#listeningprogress)
- [`bookclerk_plugin_sdk::LoginCompleteParams`](../bookclerk_plugin_abi/generated.md#logincompleteparams)
- [`bookclerk_plugin_sdk::LoginParams`](../bookclerk_plugin_abi/generated.md#loginparams)
- [`bookclerk_plugin_sdk::LoginResult`](../bookclerk_plugin_abi/generated.md#loginresult)
- [`bookclerk_plugin_sdk::LoginStartResult`](../bookclerk_plugin_abi/generated.md#loginstartresult)
- [`bookclerk_plugin_sdk::PlainFetch`](../bookclerk_plugin_abi/generated.md#plainfetch)
- [`bookclerk_plugin_sdk::PlainPart`](../bookclerk_plugin_abi/generated.md#plainpart)
- [`bookclerk_plugin_sdk::PluginDescribe`](../bookclerk_plugin_abi/index.md#plugindescribe)
- [`bookclerk_plugin_sdk::PurchaseHint`](../bookclerk_plugin_abi/generated.md#purchasehint)
- [`bookclerk_plugin_sdk::PurchaseHintParams`](../bookclerk_plugin_abi/generated.md#purchasehintparams)
- [`bookclerk_plugin_sdk::ScanBook`](../bookclerk_plugin_abi/generated.md#scanbook)
- [`bookclerk_plugin_sdk::ScanParams`](../bookclerk_plugin_abi/generated.md#scanparams)
- [`bookclerk_plugin_sdk::ScanSummary`](../bookclerk_plugin_abi/generated.md#scansummary)
- [`bookclerk_plugin_sdk::SearchCatalogParams`](../bookclerk_plugin_abi/generated.md#searchcatalogparams)
- [`bookclerk_plugin_sdk::SourceAccount`](../bookclerk_plugin_abi/generated.md#sourceaccount)
- [`bookclerk_plugin_sdk::SyncListeningResult`](../bookclerk_plugin_abi/generated.md#synclisteningresult)
- [`bookclerk_plugin_sdk::PRODUCT_API_VERSION`](../bookclerk_plugin_abi/index.md#product_api_version)
- [`bookclerk_plugin_sdk::PROTOCOL_NAME`](../bookclerk_plugin_sdk/protocol.md#protocol_name)
- [`bookclerk_plugin_catalog::host_bookclerk_target`](../bookclerk_plugin_catalog/index.md#host_bookclerk_target)
- [`bookclerk_plugin_manifest::TcpGrant`](../bookclerk_plugin_manifest/egress.md#tcpgrant)
- [`bookclerk_plugin_sdk::JobCheckpoint`](../bookclerk_plugin_abi/index.md#jobcheckpoint)
- [`bookclerk_plugin_sdk::JobInvocationLease`](../bookclerk_plugin_abi/index.md#jobinvocationlease)
- [`bookclerk_plugin_sdk::JobOutcome`](../bookclerk_plugin_abi/index.md#joboutcome)
- [`manifest::embedded_logo_api_path`](../bookclerk_plugin_manifest/logo.md#embedded_logo_api_path)
- [`manifest::entrypoint_family`](../bookclerk_plugin_manifest/types.md#entrypoint_family)
- [`manifest::logo_content_type`](../bookclerk_plugin_manifest/logo.md#logo_content_type)
- [`manifest::validate_logo`](../bookclerk_plugin_manifest/logo.md#validate_logo)
- [`manifest::BindingCapabilities`](../bookclerk_plugin_manifest/types.md#bindingcapabilities)
- [`manifest::CapabilitiesManifest`](../bookclerk_plugin_manifest/types.md#capabilitiesmanifest)
- [`manifest::DatabaseBindingManifest`](../bookclerk_plugin_manifest/types.md#databasebindingmanifest)
- [`manifest::Entrypoint`](../bookclerk_plugin_abi/generated.md#entrypoint)
- [`manifest::EventConsumer`](../bookclerk_plugin_manifest/types.md#eventconsumer)
- [`manifest::EventProducer`](../bookclerk_plugin_manifest/types.md#eventproducer)
- [`manifest::EventsManifest`](../bookclerk_plugin_manifest/types.md#eventsmanifest)
- [`manifest::JailNetworkNeed`](../bookclerk_plugin_manifest/types.md#jailnetworkneed)
- [`manifest::LogoKind`](../bookclerk_plugin_manifest/logo.md#logokind)
- [`manifest::ModuleSpec`](../bookclerk_plugin_manifest/types.md#modulespec)
- [`manifest::NamedBinding`](../bookclerk_plugin_manifest/types.md#namedbinding)
- [`manifest::NetworkCapabilities`](../bookclerk_plugin_manifest/types.md#networkcapabilities)
- [`manifest::NetworkMode`](../bookclerk_plugin_manifest/types.md#networkmode)
- [`manifest::PluginFamily`](../bookclerk_plugin_manifest/types.md#pluginfamily)
- [`manifest::PluginManifest`](../bookclerk_plugin_manifest/types.md#pluginmanifest)
- [`manifest::PluginRuntimeKind`](../bookclerk_plugin_manifest/types.md#pluginruntimekind)
- [`manifest::TriggersManifest`](../bookclerk_plugin_manifest/types.md#triggersmanifest)
- [`manifest::WorkerdLimits`](../bookclerk_plugin_manifest/types.md#workerdlimits)
- [`manifest::WorkerdRuntimeManifest`](../bookclerk_plugin_manifest/types.md#workerdruntimemanifest)
- [`manifest::ALL_ENTRYPOINTS`](../bookclerk_plugin_manifest/types.md#all_entrypoints)
- [`manifest::MAX_EMBEDDED_LOGO_BYTES`](../bookclerk_plugin_manifest/logo.md#max_embedded_logo_bytes)

## Traits

### DeploymentRuntime

```rust
pub trait DeploymentRuntime: Send + Sync {
    fn spawn_and_health<'life0, 'life1, 'async_trait>(
        &'life0 self,
        request: &'life1 DeploymentSpawn,
    ) -> Pin<Box<dyn Future<Output = SpawnHealth> + Send + 'async_trait>>
    where
        Self: 'async_trait,
        'life0: 'async_trait,
        'life1: 'async_trait,;
    fn guest_still_running<'life0, 'life1, 'async_trait>(
        &'life0 self,
        plugin_instance_id: &'life1 str,
    ) -> Pin<Box<dyn Future<Output = bool> + Send + 'async_trait>>
    where
        Self: 'async_trait,
        'life0: 'async_trait,
        'life1: 'async_trait,;
    fn guest_root<'life0, 'life1, 'async_trait>(
        &'life0 self,
        plugin_instance_id: &'life1 str,
    ) -> Pin<Box<dyn Future<Output = Option<PathBuf>> + Send + 'async_trait>>
    where
        Self: 'async_trait,
        'life0: 'async_trait,
        'life1: 'async_trait,;
    fn retire_instance<'life0, 'life1, 'async_trait>(
        &'life0 self,
        plugin_instance_id: &'life1 str,
    ) -> Pin<Box<dyn Future<Output = ()> + Send + 'async_trait>>
    where
        Self: 'async_trait,
        'life0: 'async_trait,
        'life1: 'async_trait,;
    fn tracked_instance_ids<'life0, 'async_trait>(
        &'life0 self,
    ) -> Pin<Box<dyn Future<Output = Vec<String>> + Send + 'async_trait>>
    where
        Self: 'async_trait,
        'life0: 'async_trait,;
}
```

Install health and spawn for one deployment row.

#### DeploymentRuntime::spawn_and_health

```rust
fn spawn_and_health<'life0, 'life1, 'async_trait>(
    &'life0 self,
    request: &'life1 DeploymentSpawn,
) -> Pin<Box<dyn Future<Output = SpawnHealth> + Send + 'async_trait>>
where
    Self: 'async_trait,
    'life0: 'async_trait,
    'life1: 'async_trait,
```

Spawn the guest and run health. Health is the deployment observation;
there is no pre-commit gate and nothing to roll back.

#### DeploymentRuntime::guest_still_running

```rust
fn guest_still_running<'life0, 'life1, 'async_trait>(
    &'life0 self,
    plugin_instance_id: &'life1 str,
) -> Pin<Box<dyn Future<Output = bool> + Send + 'async_trait>>
where
    Self: 'async_trait,
    'life0: 'async_trait,
    'life1: 'async_trait,
```

True when the guest spawned for `plugin_instance_id` is still running.

The default is true so a runtime that does not track processes keeps
the healthy-observation skip. [`LiveDeploymentRuntime`](#livedeploymentruntime) checks the
session.

#### DeploymentRuntime::guest_root

```rust
fn guest_root<'life0, 'life1, 'async_trait>(
    &'life0 self,
    plugin_instance_id: &'life1 str,
) -> Pin<Box<dyn Future<Output = Option<PathBuf>> + Send + 'async_trait>>
where
    Self: 'async_trait,
    'life0: 'async_trait,
    'life1: 'async_trait,
```

Tree the running guest for `plugin_instance_id` was spawned from.

`None` (the default) means unknown and never forces a respawn.

#### DeploymentRuntime::retire_instance

```rust
fn retire_instance<'life0, 'life1, 'async_trait>(
    &'life0 self,
    plugin_instance_id: &'life1 str,
) -> Pin<Box<dyn Future<Output = ()> + Send + 'async_trait>>
where
    Self: 'async_trait,
    'life0: 'async_trait,
    'life1: 'async_trait,
```

Stops a guest this process is running for `plugin_instance_id`.

Used when a storefront is disabled. The default does nothing so a
runtime that does not track processes still compiles. The deployment
row is left in place so a later enable can start it again.

#### DeploymentRuntime::tracked_instance_ids

```rust
fn tracked_instance_ids<'life0, 'async_trait>(
    &'life0 self,
) -> Pin<Box<dyn Future<Output = Vec<String>> + Send + 'async_trait>>
where
    Self: 'async_trait,
    'life0: 'async_trait,
```

Instance ids this process is still tracking.

The default is empty. [`LiveDeploymentRuntime`](#livedeploymentruntime) returns the guest map
so a deployment row that disappeared can be retired.

Implemented by: `LiveDeploymentRuntime`

## Structs

### CallbackProxy

```rust
pub struct CallbackProxy {
    pub public_base: String,
    pub ipc_endpoint: String,
    /* private fields */
}
```

Active host callback proxy for one OAuth session.

**Fields**

- `public_base`: Browser-facing origin (`http://127.0.0.1:<port>`) advertised to the guest.
- `ipc_endpoint`: Unix socket path or Windows pipe name the guest LoginServer connects to.

#### CallbackProxy::start

```rust
pub async fn start(
    callback_bind: Option<&str>,
    scratch: &Path,
    package_sid: Option<&str>,
) -> Result<Self>
```

Bind the browser TCP listener and the guest IPC endpoint, then spawn
the accept/forward loop.

##### Errors

Returns when `callback_bind` is not a socket address, the TCP listener
cannot bind, the scratch directory cannot be created, or the platform
IPC endpoint cannot be created.

#### CallbackProxy::bind_addr

```rust
pub fn bind_addr(&self) -> SocketAddr
```

Bound TCP address the browser (and guest) should use for the callback.

Implements: `Drop`

### GrantedEventConsumer

```rust
pub struct GrantedEventConsumer {
    pub event_type: String,
    pub schema_versions: Vec<u32>,
    pub supports_suspend: bool,
    pub filter: Option<String>,
}
```

One approved event-consumer subscription (structural authority).

Identity includes the event type plus the security-relevant subscription
properties that change what the plugin receives: schema versions, suspend
support, and the host-owned payload filter. Operational knobs
(`resource_class`, `max_retries`) are **not** part of authority.

**Fields**

- `event_type`: Versioned event type (`book_acquired`, …).
- `schema_versions`: Schema versions this grant may deliver (sorted, unique).
- `supports_suspend`: Whether `EventResult::Suspended` is authorized for this type.
- `filter`: Canonical JSON of the host-owned payload filter, when present.

#### GrantedEventConsumer::from_manifest

```rust
pub fn from_manifest(consumer: &EventConsumer) -> Self
```

Structural consumer snapshot from a `[[events.consumers]]` row.

#### GrantedEventConsumer::covers

```rust
pub fn covers(&self, requested: &Self) -> bool
```

True when `requested` is the same type with equal-or-narrower schemas,
equal-or-narrower suspend, and the same filter.

#### GrantedEventConsumer::intersect

```rust
pub fn intersect(&self, requested: &Self) -> Option<Self>
```

Intersection used when a stored grant already covers the current manifest.

Implements: `Clone`, `Debug`, `Deserialize<'de>`, `Eq`, `Ord`, `PartialEq`, `PartialOrd`, `Serialize`

### PendingStructural

```rust
pub struct PendingStructural {
    pub entrypoints: BTreeSet<String>,
    pub producers: BTreeSet<String>,
    pub consumers: BTreeSet<GrantedEventConsumer>,
    pub jobs: BTreeSet<String>,
    pub bindings: BTreeSet<String>,
    pub compatibility_flags: BTreeSet<String>,
}
```

Newly requested structural capabilities awaiting operator consent.

**Fields**

- `entrypoints`: Entrypoints declared by the current manifest but not granted.
- `producers`: Event producers declared by the current manifest but not granted.
- `consumers`: Event consumers declared by the current manifest but not granted.
- `jobs`: Job triggers declared by the current manifest but not granted.
- `bindings`: Host / database bindings declared by the current manifest but not granted.
- `compatibility_flags`: Compatibility flags declared by the current manifest but not granted.

#### PendingStructural::is_empty

```rust
pub fn is_empty(&self) -> bool
```

True when the current manifest introduces no unapproved structural caps.

#### PendingStructural::summary_lines

```rust
pub fn summary_lines(&self) -> Vec<String>
```

CLI/UI lines for capabilities waiting on consent.

Implements: `Clone`, `Debug`, `Default`, `Eq`, `PartialEq`

### PluginGrant

```rust
pub struct PluginGrant {
    pub schema_version: u32,
    pub plugin_key: String,
    pub plugin_id: String,
    pub entrypoints: BTreeSet<String>,
    pub producers: BTreeSet<String>,
    pub consumers: BTreeSet<GrantedEventConsumer>,
    pub jobs: BTreeSet<String>,
    pub network_mode: String,
    pub domains: BTreeSet<String>,
    pub manifest_domains: BTreeSet<String>,
    pub operator_added_domains: BTreeSet<String>,
    pub operator_denied_domains: BTreeSet<String>,
    pub tcp: BTreeSet<TcpGrant>,
    pub manifest_tcp: BTreeSet<TcpGrant>,
    pub operator_added_tcp: BTreeSet<TcpGrant>,
    pub operator_denied_tcp: BTreeSet<TcpGrant>,
    pub address_cidrs: BTreeSet<String>,
    pub manifest_cidrs: BTreeSet<String>,
    pub operator_added_cidrs: BTreeSet<String>,
    pub operator_denied_cidrs: BTreeSet<String>,
    pub allow_undeclared_public_redirects: bool,
    pub operator_allow_undeclared_public_redirects: Option<bool>,
    pub bindings: BTreeSet<String>,
    pub compatibility_flags: BTreeSet<String>,
    pub cpu_ms: Option<u32>,
    pub subrequests: Option<u32>,
    pub disk_mib: Option<u32>,
    pub memory_mib: Option<u32>,
    pub cpu_rate_percent: Option<u32>,
    pub extra_processes: Option<u32>,
    pub approved_at: String,
}
```

One approved grant snapshot for a provenance-qualified plugin.

**Fields**

- `schema_version`: Persisted grant document version.
  
  `0` (or omitted JSON) is a legacy document. Empty structural sets
  approve nothing at every version; a newer manifest cannot acquire
  entrypoints, producers, consumers, jobs, or bindings by request alone.
- `plugin_key`: Canonical [`bookclerk_plugin_catalog::PluginKey`](../bookclerk_plugin_catalog/index.md#pluginkey).
- `plugin_id`: Display / CLI alias from `plugin.toml` (not security authoritative).
- `entrypoints`: Approved named entrypoints (wire names: `storefront`, `storage`,
  `databaseAdapter`, `remoteLibrary`, `cli`, `oidc`).
- `producers`: Approved event types the plugin may publish through its events binding.
- `consumers`: Approved event-consumer subscriptions (type + schema + suspend + filter).
- `jobs`: Approved `[triggers].jobs` types the plugin may handle.
- `network_mode`: Approved network mode: `deny` or `outbound`.
- `domains`: Approved initial outbound domain patterns (**workerd** allowlist).
  
  Effective set: manifest request ∪ operator additions − operator denials.
- `manifest_domains`: Network destinations requested by the installed manifest (not operator-added).
- `operator_added_domains`: Operator-added destinations beyond the current manifest request.
  
  Survive upgrades of the same [`Self::plugin_key`](#plugingrant). Do not transfer across provenance.
- `operator_denied_domains`: Operator-denied destinations (even if the manifest still lists them).
- `tcp`: Effective raw TCP grants (manifest ∪ operator additions − denials).
- `manifest_tcp`: TCP grants requested by the installed manifest.
- `operator_added_tcp`: Operator-added TCP grants; survive upgrades of the same PluginKey.
- `operator_denied_tcp`: Operator-denied TCP grants (even if the current manifest still lists them).
  
  Same PluginKey upgrades must not silently restore a TCP host the operator
  removed. Host spawn overlays (postgres URL, D1, ABS, S3) may still add
  destinations from operator config after this set is applied.
- `address_cidrs`: Effective CIDR grants beyond the public Internet.
- `manifest_cidrs`: CIDRs requested by the installed manifest.
- `operator_added_cidrs`: Operator-added CIDRs; survive upgrades of the same PluginKey.
- `operator_denied_cidrs`: Operator-denied CIDRs (even if the current manifest still lists them).
  
  Same PluginKey upgrades must not silently restore an address-space grant
  the operator removed. Host spawn overlays (loopback postgres/ABS) may
  still add CIDRs from operator config after this set is applied.
- `allow_undeclared_public_redirects`: Effective undeclared-public-redirect permission.
- `operator_allow_undeclared_public_redirects`: Explicit operator override for undeclared public redirects (`None` = follow manifest).
- `bindings`: Approved host binding names (`config`, `secrets`, `oauth`, …).
- `compatibility_flags`: Approved workerd compatibility flags from the consent snapshot.
- `cpu_ms`: Optional workerd CPU budget override in milliseconds.
- `subrequests`: Optional workerd outbound fetch / subrequest budget override.
- `disk_mib`: Optional per-plugin disk budget for `data/` and `tmp/` (MiB each).
  
  Applies to **native and workerd** guests. Unset →
  [`PLUGIN_STATE_BUDGET_MIB_DEFAULT`](#plugin_state_budget_mib_default); clamped to
  [`PLUGIN_STATE_BUDGET_MIB_MAX`](#plugin_state_budget_mib_max).
- `memory_mib`: Optional jail Spec memory ceiling (MiB) for **native and workerd**.
- `cpu_rate_percent`: Optional jail Spec CPU rate as percent of one logical CPU for **native**
  guests (`1..=`[`host_cpu_rate_max`](#host_cpu_rate_max); values above 100 request multi-core
  bandwidth). Workerd guests omit this and use `cpu_ms` plus the host jail
  CPU ceiling instead.
- `extra_processes`: Optional **extra** process/thread budget beyond launcher overhead (**native**).
  
  Workerd guests omit this; the host applies default extra headroom only.
  Absolute Spec `active_processes` = overhead(runtime) + this value.
- `approved_at`: RFC 3339 time when the operator approved this grant.

#### PluginGrant::allows_event_consumer

```rust
pub fn allows_event_consumer(&self, event_type: &str, schema_version: u32) -> bool
```

True when this grant authorizes delivering `event_type` at `schema_version`.

#### PluginGrant::allows_queue_consumer

```rust
pub fn allows_queue_consumer(&self, queue: &str) -> bool
```

True when this grant authorizes delivering messages from `queue`.

#### PluginGrant::allows_queue_producer

```rust
pub fn allows_queue_producer(&self, binding: &str) -> bool
```

True when this grant authorizes sending through the queue producer
`binding`.

#### PluginGrant::allows_job

```rust
pub fn allows_job(&self, job_type: &str) -> bool
```

True when this grant authorizes handling `job_type`.

#### PluginGrant::empty

```rust
pub fn empty() -> Self
```

Empty grant used as a struct-update base for tests and new fields.

#### PluginGrant::egress_policy

```rust
pub fn egress_policy(&self) -> EgressPolicy
```

Canonical typed network policy for workerd and the native socket proxy.

Implements: `Clone`, `Debug`, `Deserialize<'de>`, `Eq`, `PartialEq`, `Serialize`

### PluginGrantStore

```rust
pub struct PluginGrantStore {
    pub grants: Vec<PluginGrant>,
}
```

On-disk grant store.

**Fields**

- `grants`: Persisted per-plugin grant snapshots.

#### PluginGrantStore::path

```rust
pub fn path(files_dir: &Path) -> PathBuf
```

Absolute path to `plugin-grants.json` under `files_dir`.

#### PluginGrantStore::load

```rust
pub fn load(files_dir: &Path) -> Result<Self>
```

Loads grants from disk, or an empty store when the file is missing.

Operator-selected `files_dir` may contain `..` spelling (resolved via
canonicalize). Escaping or dangling leaf symlinks at `plugin-grants.json`
are hard errors; a true missing file yields an empty store.

##### Arguments

* `files_dir` - Bookclerk files directory that owns `plugin-grants.json`.

##### Returns

Parsed [`PluginGrantStore`](#plugingrantstore), defaulting when the file does not exist.

##### Errors

Returns [`PluginError`](#pluginerror) when the file cannot be read or parsed.

#### PluginGrantStore::save

```rust
pub fn save(&self, files_dir: &Path) -> Result<()>
```

Writes this grant store to `plugin-grants.json` under `files_dir`.

Uses unique same-dir `create_new` staging and Unix `rename` / Windows
`MoveFileExW` so a leaf symlink at the final path is replaced as a
directory entry (not followed). Cleanup removes only staging this call
created.

##### Arguments

* `files_dir` - Bookclerk files directory that owns `plugin-grants.json`.

##### Errors

Returns [`PluginError`](#pluginerror) when serialization or the write fails.

#### PluginGrantStore::get_by_plugin_key

```rust
pub fn get_by_plugin_key(&self, plugin_key: &str) -> Option<&PluginGrant>
```

Returns the grant whose canonical PluginKey matches `plugin_key`.

Does not fall back to display aliases — spawn and privilege checks must
not inherit another provenance's grant.

#### PluginGrantStore::get

```rust
pub fn get(&self, plugin_key: &str) -> Option<&PluginGrant>
```

Returns the grant for `plugin_key` (canonical), or an unambiguous alias.

Display / CLI helper only. Spawn, overlay, and other privilege checks
must call [`Self::get_by_plugin_key`](#plugingrantstoreget_by_plugin_key) so an alias cannot inherit another
provenance's grant.

##### Arguments

* `plugin_key` - Canonical PluginKey text, or a display alias.

##### Returns

A reference to the matching [`PluginGrant`](#plugingrant), or `None`.

#### PluginGrantStore::upsert

```rust
pub fn upsert(&mut self, grant: PluginGrant)
```

Inserts or replaces the grant for `grant.plugin_key`.

Incoming grants with a PluginKey match only that key: they never
overwrite a different provenance, and never collapse onto a keyless
same-alias legacy row. A keyless incoming grant (legacy files / tests)
matches only other keyless rows by alias so it cannot replace a
provenance-qualified grant.

##### Arguments

* `grant` - Full consent snapshot to persist in memory (call [`Self::save`](#plugingrantstoresave) to flush).

Implements: `Clone`, `Debug`, `Default`, `Deserialize<'de>`, `Eq`, `PartialEq`, `Serialize`

### AuthorizedLocalPackages

```rust
pub struct AuthorizedLocalPackages {
    /* private fields */
}
```

Operator-placed archives keyed by canonical plugin key.

A key that two packages both resolve to is removed from the map and
recorded in [`Self::is_conflicted`](#authorizedlocalpackagesis_conflicted). Neither package is installed.

#### AuthorizedLocalPackages::get

```rust
pub fn get(&self, key: &str) -> Option<&LocalPackage>
```

Package for `key`, when exactly one authorized archive resolved to it.

#### AuthorizedLocalPackages::is_conflicted

```rust
pub fn is_conflicted(&self, key: &str) -> bool
```

True when two archives resolved to `key` and neither was kept.

#### AuthorizedLocalPackages::is_empty

```rust
pub fn is_empty(&self) -> bool
```

True when no archive was loaded and no key collided.

#### AuthorizedLocalPackages::keys

```rust
pub fn keys(&self) -> Keys<'_, String, LocalPackage>
```

Canonical keys that have exactly one package.

#### AuthorizedLocalPackages::insert

```rust
pub fn insert(&mut self, key: String, package: LocalPackage)
```

Inserts `package` under `key`, clearing a previous conflict on that key.

Implements: `Clone`, `Debug`, `Default`

### DeploymentSpawn

```rust
pub struct DeploymentSpawn {
    pub plugin_instance_id: String,
    pub plugin_key: String,
    pub plugin_root: Option<PathBuf>,
    pub config: ExtensibleConfig,
    pub secrets: ExtensibleConfig,
    pub config_revision: i64,
    pub installed: Option<Arc<PluginVersion>>,
}
```

Spawn request after config resolution.

**Fields**

- `plugin_instance_id`: Instance this deployment runs. Not derived from the plugin key.
- `plugin_key`: Canonical plugin key.
- `plugin_root`: Installed tree, when discovery or install produced one.
- `config`: Granted `CONFIG` payload.
- `secrets`: Granted `SECRETS` payload.
- `config_revision`: Document revision the spawn applies.
- `installed`: Installed version the tree was materialized from; `None` for a
  host-local tree.

Implements: `Clone`, `Debug`

### LiveDeploymentRuntime

```rust
pub struct LiveDeploymentRuntime {
    pub config: Arc<RwLock<Config>>,
    pub store: Arc<RwLock<LibraryStore>>,
    pub sources: Arc<RwLock<SourceRegistry>>,
    pub integrations: Arc<RwLock<IntegrationRegistry>>,
    pub destinations: Arc<RwLock<DestinationRegistry>>,
    /* private fields */
}
```

Production runtime: spawn through the existing host adapters and register
the session on the live registry.

**Fields**

- `config`: Live daemon config.
- `store`: Open library, used for instance bindings.
- `sources`: Storefront sessions the reconciler owns.
- `integrations`: Integration sessions the reconciler owns.
- `destinations`: Destination sessions the reconciler owns.

#### LiveDeploymentRuntime::new

```rust
pub fn new(
    config: Arc<RwLock<Config>>,
    store: Arc<RwLock<LibraryStore>>,
    sources: Arc<RwLock<SourceRegistry>>,
    integrations: Arc<RwLock<IntegrationRegistry>>,
    destinations: Arc<RwLock<DestinationRegistry>>,
) -> Self
```

Runtime bound to the daemon's live registries.

#### LiveDeploymentRuntime::reconcile_lock

```rust
pub fn reconcile_lock(&self) -> &Mutex<()>
```

Lock a caller holds for the duration of one reconcile pass.

#### LiveDeploymentRuntime::set_integration_context

```rust
pub fn set_integration_context(&self, ctx: IntegrationContext)
```

Stores the daemon lifecycle context used when a deployment starts an integration.

##### Panics

Panics when the integration-context lock is poisoned.

#### LiveDeploymentRuntime::retire_plugin_instance

```rust
pub async fn retire_plugin_instance(&self, plugin_instance_id: &str)
```

Stops every registry entry for `plugin_instance_id` and drops the guest.

Sources, integrations, and storage slots all hold the session. Removing
only the source leaves an integration claiming events or a destination
still accepting writes. Other instances of the same plugin key stay.

#### LiveDeploymentRuntime::forget_guest

```rust
pub fn forget_guest(&self, plugin_instance_id: &str)
```

Drops the tracked guest for `plugin_instance_id` without taking the source registry.

Config reload already holds that registry. Dropping this entry stops the
process once the registry no longer keeps the session.

##### Panics

Panics when the deployment guest lock is poisoned.

#### LiveDeploymentRuntime::opened_config_json

```rust
pub fn opened_config_json(&self, plugin_instance_id: &str) -> Option<Value>
```

`CONFIG` JSON the live `open` call delivered for `plugin_instance_id`.

##### Panics

Panics when the deployment guest lock is poisoned.

#### LiveDeploymentRuntime::tracked_guest_pid

```rust
pub fn tracked_guest_pid(&self, plugin_instance_id: &str) -> Option<u32>
```

Native guest pid for `plugin_instance_id`, when this process spawned it.

##### Panics

Panics when the deployment guest lock is poisoned.

Implements: `DeploymentRuntime`

### LocalPackage

```rust
pub struct LocalPackage {
    pub archive: PathBuf,
    pub manifest: BookclerkPackageManifest,
}
```

Local archive the reconciler may install when the ledger has no row.

**Fields**

- `archive`: Archive path (`tar.gz` or `zip`).
- `manifest`: Install-grade package manifest for `archive`.

Implements: `Clone`, `Debug`

### DiscoveredPlugin

```rust
pub struct DiscoveredPlugin {
    pub manifest: PluginManifest,
    pub root: PathBuf,
    pub command: PathBuf,
    pub identity: PluginInstallIdentity,
    pub version_grant: Option<PluginGrant>,
}
```

A discovered plugin ready to spawn.

**Fields**

- `manifest`: Parsed `plugin.toml` for this install directory.
- `root`: Directory containing `plugin.toml` (cwd + relative `command` base).
- `command`: Absolute path to the plugin executable.
- `identity`: Host-evaluated provenance-qualified identity.
- `version_grant`: Consent grant recorded on the cluster release this tree was
  materialized from; `None` for host-local trees, whose grant lives in
  `plugin-grants.json`.

#### DiscoveredPlugin::new

```rust
pub fn new(manifest: PluginManifest, root: PathBuf, command: PathBuf) -> Self
```

Builds a discovered plugin, evaluating content hashes and provenance.

Incomplete synthetic trees should use [`Self::for_test`](#discoveredpluginfor_test) instead of
weakening production evaluation.

##### Panics

Panics when the install tree cannot be evaluated. Prefer [`Self::try_new`](#discoveredplugintry_new).

#### DiscoveredPlugin::try_new

```rust
pub fn try_new(
    manifest: PluginManifest,
    root: PathBuf,
    command: PathBuf,
    files_dir: Option<&Path>,
) -> Result<Self>
```

Production constructor: fail closed on malformed receipts, hash errors,
and unrecognized install receipts.

##### Errors

Returns when identity evaluation fails closed.

#### DiscoveredPlugin::for_test

```rust
pub fn for_test(manifest: PluginManifest, root: PathBuf, command: PathBuf) -> Self
```

Explicit test/dev constructor for incomplete plugin trees.

Does not hash the tree or consult receipts. Never grants platform trust.

#### DiscoveredPlugin::from_release

```rust
pub fn from_release(
    manifest: PluginManifest,
    root: PathBuf,
    plugin_key: &str,
    version: &str,
    payload_root_sha256: &str,
    archive_sha256: &str,
    grant: PluginGrant,
) -> Result<Self>
```

Builds a plugin from a tree materialized from a cluster release.

The caller has already verified the tree's payload-root digest
against the release row; identity, version, and the consent grant
come from that row, not from a receipt or the host grant file.

##### Errors

Returns an error when the spawn command cannot be resolved, the
manifest cannot be hashed, or `plugin_key` is malformed.

#### DiscoveredPlugin::alias

```rust
pub fn alias(&self) -> &str
```

Display / CLI alias (`plugin.toml` id).

#### DiscoveredPlugin::plugin_key

```rust
pub fn plugin_key(&self) -> &PluginKey
```

Provenance-qualified key.

Implements: `Clone`, `Debug`

### EnrolledPlugin

```rust
pub struct EnrolledPlugin {
    pub plugin: DiscoveredPlugin,
    pub instance: PluginInstance,
    pub deployment: PluginDeployment,
    pub schema: PinnedPluginSchema,
}
```

One plugin after enrollment on this node.

**Fields**

- `plugin`: Discovered install.
- `instance`: Default instance for the plugin key.
- `deployment`: This node's deployment of the default instance.
- `schema`: Pinned settings schema.

Implements: `Clone`, `Debug`

### EventOutbox

```rust
pub struct EventOutbox {
    /* private fields */
}
```

Host-owned facts a session needs before it can hand a guest an `EVENTS`
binding: the outbox store plus the producer allowlist.

#### EventOutbox::new

```rust
pub fn new(
    store: LibraryStore,
    plugin_id: &str,
    declared: impl IntoIterator<Item = String>,
    granted: &BTreeSet<String>,
) -> Option<Self>
```

Builds the outbox hook for `plugin_id`, or `None` when no producer is
both declared and granted (the guest then sees no `EVENTS` binding).

#### EventOutbox::publisher

```rust
pub fn publisher(&self, invocation: &Invocation) -> OutboxEventPublisher
```

Publisher bound to one `PluginWorker.open` invocation (account scope
and default correlation / causation come from `invocation`).

Implements: `Clone`

### OutboxEventPublisher

```rust
pub struct OutboxEventPublisher {
    /* private fields */
}
```

[`EventPublisher`](../bookclerk_plugin_abi/index.md#eventpublisher) server for one invocation: validates a guest
[`PluginEvent`](../bookclerk_plugin_abi/generated.md#pluginevent) and appends it to the library outbox.

#### OutboxEventPublisher::spec

```rust
pub fn spec(&self, event: PluginEvent) -> Result<PublishDomainEventSpec, PluginError>
```

Validates `event` and builds the outbox row the host will insert.

##### Errors

* `forbidden` when `eventType` is not a consented producer.
* `payload_too_large` when `payload` exceeds [`MAX_EVENT_PAYLOAD_BYTES`](../bookclerk_plugin_abi/index.md#max_event_payload_bytes).
* `invalid_params` when `payload` is not UTF-8 JSON.

Implements: `EventPublisher`

### DatabaseRegistry

```rust
pub struct DatabaseRegistry {
    /* private fields */
}
```

Long-lived external database plugin for the active `[database].plugin`.

#### DatabaseRegistry::active

```rust
pub fn active(&self) -> Option<Arc<ExternalDatabase>>
```

Currently active external database guest for `[database].plugin`.

Implements: `Clone`, `Default`

### DestinationRegistry

```rust
pub struct DestinationRegistry {
    /* private fields */
}
```

Long-lived external output plugins loaded at host startup.

#### DestinationRegistry::s3

```rust
pub fn s3(&self) -> Option<Arc<dyn StorageBackend>>
```

External S3 output backend, when exactly one is loaded.

Two deployed S3 instances return `None`. Acquire uses
[`Self::require_s3`](#destinationregistryrequire_s3) so that case fails closed instead of falling
through to `[output.s3]`.

#### DestinationRegistry::require_s3

```rust
pub fn require_s3(&self) -> Result<Option<Arc<dyn StorageBackend>>, String>
```

The S3 backend, or an error when more than one deployed instance owns it.

##### Errors

Returns an error when two deployed S3 instances would otherwise overwrite
one slot.

#### DestinationRegistry::local

```rust
pub fn local(&self) -> Option<Arc<dyn StorageBackend>>
```

External local-filesystem output backend, when exactly one is loaded.

#### DestinationRegistry::require_local

```rust
pub fn require_local(&self) -> Result<Option<Arc<dyn StorageBackend>>, String>
```

The local backend, or an error when more than one deployed instance owns it.

##### Errors

Returns an error when two deployed local instances would otherwise
overwrite one slot.

#### DestinationRegistry::settings_for

```rust
pub fn settings_for(&self, kind: OutputBackendKind) -> Option<DestinationSettings>
```

Key layout settings of the deployed destination of `kind`, when exactly
one is deployed.

#### DestinationRegistry::plugin_session

```rust
pub fn plugin_session(
    &self,
    plugin_id: &str,
    account_id: &str,
) -> Option<Arc<PluginSession>>
```

Plugin session for `plugin_id` and `account_id`, when that guest was loaded.

`plugin_id` may be the canonical PluginKey (how sessions are stored) or
a display alias. Aliases are installation-unique; two occupants sharing
an alias is invalid state and fails closed rather than returning a twin.

#### DestinationRegistry::require_plugin_session

```rust
pub fn require_plugin_session(
    &self,
    plugin_id: &str,
    account_id: &str,
) -> Result<Arc<PluginSession>, String>
```

[`Self::plugin_session`](#destinationregistryplugin_session) that reports same-key ambiguity.

##### Errors

Returns an error when no guest is loaded, or when more than one session
matches `plugin_id`.

#### DestinationRegistry::reattach_owned

```rust
pub fn reattach_owned(&self, candidate: &mut Self, deployed_instances: &BTreeSet<String>)
```

Copies deployed sessions from this registry onto `candidate`.

`deployed_instances` is the set of present plugin instance ids. Reload
builds `candidate` without those plugin keys. Only sessions the
reconciler bound to one of those ids are copied, including the S3 or
local backend they own. A transitional session for the same key is left
behind.

Implements: `Clone`, `Default`

### ExternalDatabase

```rust
pub struct ExternalDatabase {
    /* private fields */
}
```

External database backend spawned for `[database].plugin`.

#### ExternalDatabase::spawn

```rust
pub async fn spawn(plugin: &DiscoveredPlugin, config: &Config) -> PluginResult<Self>
```

Spawn and describe a database plugin (connection happens later).

##### Errors

Returns an error when the operation fails.

#### ExternalDatabase::spawn_with

```rust
pub async fn spawn_with(
    plugin: &DiscoveredPlugin,
    config: &Config,
    services: SessionServices,
) -> PluginResult<Self>
```

[`Self::spawn`](#externaldatabasespawn) with explicit session services (spawn transport, …).

Product callers use [`Self::spawn`](#externaldatabasespawn); this exists so transport
diagnostics can compare the front door against direct native.

##### Errors

Returns an error when the guest cannot start or negotiate.

#### ExternalDatabase::connect

```rust
pub async fn connect(
    &self,
    config: &Config,
) -> Result<(DatabaseConnection, DbCapabilities), DbErr>
```

Open the library connection through the guest (`database.openSession`).

##### Errors

Returns an error when the operation fails.

#### ExternalDatabase::connect_without_migrate

```rust
pub async fn connect_without_migrate(
    &self,
    config: &Config,
) -> Result<(DatabaseConnection, DbCapabilities), DbErr>
```

Open the library connection without applying host schema (CLI migrate / version).

##### Errors

Returns an error when the guest cannot open a session.

#### ExternalDatabase::library_backup_ops

```rust
pub fn library_backup_ops(&self) -> SharedAdapterBackupOps
```

Adapter snapshot/identity/restore hooks for the library session.

#### ExternalDatabase::binding_backup_ops

```rust
pub fn binding_backup_ops(&self, key: impl Into<String>) -> SharedAdapterBackupOps
```

Adapter snapshot/identity/restore hooks for one named binding session.

#### ExternalDatabase::migrate_to

```rust
pub async fn migrate_to(
    &self,
    db: &DatabaseConnection,
    caps: &DbCapabilities,
    target: i64,
    opts: SchemaApplyOptions,
) -> Result<SchemaWalk, DbErr>
```

Migrates toward `target` using guest `executeAtomic` batches (CLI migrate / downgrade).

##### Errors

Returns when capability flags are unknown or a schema batch fails.

#### ExternalDatabase::open_binding_databases

```rust
pub async fn open_binding_databases(
    &self,
    config: &Config,
    store: &LibraryStore,
    owner_plugin_id: &str,
    bindings: &[String],
    owner: &PluginSession,
) -> PluginResult<Vec<(String, GuestDatabaseFactory)>>
```

Opens (provisioning on first use) isolated sessions for the named
plugin database bindings of `owner_plugin_id`.

Backend-native units: SQLite gets a file per binding under
`<files_dir>/plugin-databases/<plugin-key-fs-id>/<BINDING>.db`, PostgreSQL a
dedicated database (`pb_` + 32 hex of the `(plugin, binding)` digest),
and D1 its own database (`bookclerk-pb-` + the same 32 hex). Third-party
adapters advertising `pluginDatabases` receive `binding` plus a
collision-resistant `instanceId` on the public
[`bookclerk_plugin_abi::DatabaseAdapterConfig`](../bookclerk_plugin_abi/generated.md#databaseadapterconfig). Each provisioned unit
is recorded in the `plugin_databases` registry (an existing row wins so
a re-open never re-targets a binding).

##### Errors

Fails closed when the adapter does not advertise
`DbCapabilities::plugin_databases`, when provisioning fails, or when
the registry cannot be recorded.

#### ExternalDatabase::open_binding_seaorm

```rust
pub async fn open_binding_seaorm(
    &self,
    config: &Config,
    owner_plugin_id: &str,
    binding: &str,
    unit_ref: &str,
    provision: bool,
) -> PluginResult<(DatabaseConnection, DbCapabilities)>
```

Opens one plugin binding as a SeaORM connection through the guest session.

Physical provisioning is adapter-owned (`provision`). Logical identity is
`(plugin_id, binding)`. Does not switch on sqlite/postgres/d1 beyond
building the adapter's own connect context.

##### Errors

Returns when the guest cannot open the binding or the proxy fails.

#### ExternalDatabase::drop_provisioned_unit

```rust
pub async fn drop_provisioned_unit(
    config: &Config,
    adapter_plugin_key: &str,
    unit_ref: &str,
) -> PluginResult<()>
```

Physically deletes a provisioned binding unit. The registry row is the
caller's to remove **after** this returns success.

Spawns the **original** adapter PluginKey and calls `Database.dropUnit`.
SQLite unlinks the file and journal sidecars. PostgreSQL issues
`DROP DATABASE`. D1 deletes the Cloudflare database by name. Adapters
that do not implement `dropUnit` fail closed so a registry row cannot
outlive a unit the host cannot prove is gone.

##### Errors

Returns when the adapter is not installed, spawn fails, credentials
are missing, or the guest refuses the delete.

#### ExternalDatabase::adapter_plugin_key

```rust
pub fn adapter_plugin_key(&self) -> &str
```

Canonical PluginKey of this adapter guest.

#### ExternalDatabase::first_party_kind

```rust
pub fn first_party_kind(&self) -> Option<DatabasePluginKind>
```

Verified first-party adapter kind, if any.

Implements: `Clone`

### ExternalIntegration

```rust
pub struct ExternalIntegration {
    /* private fields */
}
```

External integration backed by a discovered plugin binary.

#### ExternalIntegration::spawn_prepared

```rust
pub async fn spawn_prepared(
    plugin: &DiscoveredPlugin,
    config: &Config,
    services: SessionServices,
    prepared: PreparedOpen,
    allow_credential_login: bool,
) -> Result<Self>
```

Spawn with bindings already resolved for one plugin instance.

Does not look up an instance by plugin key.

##### Errors

Returns an error when the guest cannot start or `open` fails.

#### ExternalIntegration::bind_plugin_instance

```rust
pub fn bind_plugin_instance(&mut self, plugin_instance_id: &str)
```

Records the plugin instance id used to recognize this guest on reload.

#### ExternalIntegration::check_health

```rust
pub async fn check_health(&self) -> Result<()>
```

Integration health RPC.

Guests that do not export `remoteLibrary` are ready once `open` has
succeeded. Event, job, OIDC, and CLI-only integrations use that
describe/open check. This function calls remote-library health only
when that entrypoint is exported.

##### Errors

Returns an error when remote-library health reports not ok.

Implements: `Integration`

### ExternalSource

```rust
pub struct ExternalSource {
    /* private fields */
}
```

External content source backed by a discovered plugin binary.

#### ExternalSource::spawn_prepared

```rust
pub async fn spawn_prepared(
    plugin: &DiscoveredPlugin,
    config: &Config,
    services: SessionServices,
    prepared: PreparedOpen,
) -> Result<Self>
```

Spawn with bindings already resolved for one plugin instance.

Does not look up an instance by plugin key. See
[`crate::prepare_open_for_instance`](#prepare_open_for_instance) for how the bindings are built.

##### Errors

Returns an error when the guest cannot start or `open` fails.

#### ExternalSource::with_handed_credentials

```rust
pub fn with_handed_credentials(self, credentials: Vec<AccountCredential>) -> Self
```

Uses only `credentials` (an owner cluster handed them with a job)
for every guest call: no library is read for credentials, and
`ContentSource::scan` is refused in favour of
[`Self::scan_handed`](#externalsourcescan_handed).

#### ExternalSource::scan_handed

```rust
pub async fn scan_handed(&self, opts: ScanOptions) -> Result<Vec<ScanBook>>
```

Scans with the handed credentials and answers the guest's rows
unwritten (the owner validates and writes them).

`opts.accounts` narrows the scan to those account ids; empty means
every handed account.

##### Errors

Returns an error when this source has no handed credentials, the
grant lacks the `secrets` binding, or the guest fails.

#### ExternalSource::bind_plugin_instance

```rust
pub fn bind_plugin_instance(&mut self, plugin_instance_id: &str)
```

Records the plugin instance id used as this source's registry address.

#### ExternalSource::check_health

```rust
pub async fn check_health(&self) -> Result<()>
```

Storefront health RPC.

##### Errors

Returns an error when the guest has no storefront or health reports not ok.

#### ExternalSource::guest_pid

```rust
pub fn guest_pid(&self) -> Option<u32>
```

Native guest pid, when the session recorded one.

Implements: `ContentSource`

### HostControlPlane

```rust
pub struct HostControlPlane {
    pub config: Config,
    pub database_registry: DatabaseRegistry,
    pub store: LibraryStore,
    pub session: ControlPlaneSession,
    pub enrolled: Vec<EnrolledPlugin>,
    pub present: Vec<PresentDeployment>,
    pub default_instances: HashMap<String, String>,
}
```

Open control plane for this process.

**Fields**

- `config`: Bootstrap config with the database-backed domains overlaid.
- `database_registry`: Database adapters discovered for bootstrap.
- `store`: Open library.
- `session`: Node identity, cluster id, and core documents.
- `enrolled`: Plugins enrolled on this pass.
- `present`: Deployments desired present on this node, in deployment-id order.
- `default_instances`: Default instance id per plugin key, for routing a bare key.

#### HostControlPlane::node_id

```rust
pub fn node_id(&self) -> &str
```

This node's id.

#### HostControlPlane::load_sources

```rust
pub async fn load_sources(&self, services: &SessionServices) -> Result<SourceRegistry>
```

Spawns every present storefront deployment and registers it under its
instance id, with the plugin's default instance recorded for routing.

A guest that fails to start is logged and skipped so the rest of the
command still runs.

##### Errors

Returns an error when the files directory is unavailable.

#### HostControlPlane::load_integrations

```rust
pub async fn load_integrations(
    &self,
    services: &SessionServices,
) -> Result<IntegrationRegistry>
```

Spawns every present integration deployment.

##### Errors

Returns an error when the files directory is unavailable.

#### HostControlPlane::load_destinations

```rust
pub async fn load_destinations(&self) -> Result<DestinationRegistry>
```

Spawns every present storage deployment.

##### Errors

Returns an error when a present destination guest cannot start: a
destination the database wants here that does not come up would
otherwise silently drop acquire output.

### NodeEnrollment

```rust
pub struct NodeEnrollment {
    pub config: Config,
    pub session: ControlPlaneSession,
    pub enrolled: Vec<EnrolledPlugin>,
}
```

Result of [`enroll_node`]: the bootstrap half of a host control plane.

**Fields**

- `config`: Bootstrap config with the database-backed domains overlaid.
- `session`: Node identity, cluster id, and core documents.
- `enrolled`: Plugins enrolled on this pass.

### PresentDeployment

```rust
pub struct PresentDeployment {
    pub deployment: PluginDeployment,
    pub instance: PluginInstance,
    pub plugin: Option<DiscoveredPlugin>,
}
```

A deployment the database wants present on this node, joined with the
instance row and the local install it will run.

**Fields**

- `deployment`: Desired row.
- `instance`: Instance the row runs.
- `plugin`: Install discovered on this node, when there is one.

Implements: `Clone`, `Debug`

### AddedVersion

```rust
pub struct AddedVersion {
    pub version: PluginVersion,
    pub created: bool,
    pub pointer: Option<PluginInstall>,
}
```

What [`add_package_version`](#add_package_version) recorded.

**Fields**

- `version`: The version row.
- `created`: True when this call wrote the row (false for an identical re-install).
- `pointer`: Pointer after activation, when this call moved it.

Implements: `Clone`, `Debug`

### InstallRequest

```rust
pub struct InstallRequest {
    pub package: BookclerkPackageManifest,
    pub coordinate: PackageCoordinate,
    pub archive: Option<PathBuf>,
    pub target: Option<String>,
    pub offline: bool,
    pub activate: bool,
    pub approve: bool,
}
```

An operator's `install`: what `bookclerk plugins install` and
`POST /api/config/plugin-versions` both install.

**Fields**

- `package`: Resolved package manifest.
- `coordinate`: Where the package came from. A local archive's coordinate is
  path-derived and is not recorded on the version.
- `archive`: Local archive to install instead of fetching the coordinate.
- `target`: Install only this target instead of every target in the package.
- `offline`: Refuse network fetches.
- `activate`: Make the version active even when another version already is.
- `approve`: Record the requested consent as approved when no earlier approval
  carries forward.

Implements: `Clone`, `Debug`

### InstalledVersion

```rust
pub struct InstalledVersion {
    pub version: PluginVersion,
    pub created: bool,
    pub active: bool,
    pub instance: PluginInstance,
}
```

What [`install_version`](#install_version) did.

**Fields**

- `version`: The installed (and maybe approved) version.
- `created`: False when this exact version was already installed.
- `active`: True when the version is the plugin's active version afterwards.
- `instance`: The plugin's default instance.

Implements: `Clone`, `Debug`

### VersionRequest

```rust
pub struct VersionRequest {
    pub plugin_key: String,
    pub package: BookclerkPackageManifest,
    pub archive: Option<PathBuf>,
    pub target: Option<String>,
    pub coordinate: String,
    pub offline: bool,
    pub publisher_verified: bool,
    pub grant: Option<PluginGrant>,
    pub activate: Activate,
}
```

One package to install.

**Fields**

- `plugin_key`: Canonical plugin key the version is recorded under.
- `package`: Install-grade package manifest.
- `archive`: Local archive. `None` fetches every target's artifact from its URL
  and verifies the manifest digests.
- `target`: Target the local archive is built for (default: this host's).
- `coordinate`: Package coordinate recorded for `update`; empty for a local archive.
- `offline`: Refuse remote downloads (a local archive needs none).
- `publisher_verified`: Publisher verification result (recorded on the version).
- `grant`: Consent grant approved at install time, if any.
- `activate`: When to move the active pointer.

Implements: `Clone`, `Debug`

### PreparedOpen

```rust
pub struct PreparedOpen {
    pub bindings: BindingValues,
    pub spawn_config_table: Value,
    pub granted_config: Value,
    pub config_revision: i64,
}
```

Bindings passed to `PluginWorker.open`, plus the spawn config table.

**Fields**

- `bindings`: Values for `PluginWorker.open`.
- `spawn_config_table`: JSON object passed into [`crate::PluginSession::spawn_with`](#pluginsessionspawn_with).
  
  The session still applies [`crate::spawn_config_for_grant`](#spawn_config_for_grant).
- `granted_config`: JSON object the guest receives when the grant includes `config`.
- `config_revision`: Document revision this open applies.

Implements: `Clone`, `Debug`

### KvNamespaces

```rust
pub struct KvNamespaces {
    /* private fields */
}
```

Host-owned facts a session needs before it can hand a guest its KV
bindings: the library store plus the granted rows.

#### KvNamespaces::new

```rust
pub fn new(
    store: LibraryStore,
    plugin_key: &str,
    manifest: &PluginManifest,
    grant: &PluginGrant,
) -> Option<Self>
```

Builds the KV hook for `plugin_key`, or `None` when the manifest
declares no row or the grant lacks [`KV_GRANT_BINDING`](#kv_grant_binding).

#### KvNamespaces::binding_names

```rust
pub fn binding_names(&self) -> Vec<String>
```

Granted binding names in manifest order.

#### KvNamespaces::namespaces

```rust
pub fn namespaces(&self) -> Vec<(String, String, Arc<dyn KvNamespace>)>
```

Namespaces shaped for `HostBindings::kv` as `(binding, namespace, kv)`.

Implements: `Clone`

### LibraryKvNamespace

```rust
pub struct LibraryKvNamespace {
    /* private fields */
}
```

[`KvNamespace`](../bookclerk_plugin_abi/index.md#kvnamespace) server for one `(plugin_key, namespace)` on the library.

#### LibraryKvNamespace::new

```rust
pub fn new(store: LibraryStore, plugin_key: &str, row: &KvNamespaceManifest) -> Self
```

Builds one namespace directly (tests and in-process hosts).

#### LibraryKvNamespace::namespace

```rust
pub fn namespace(&self) -> &str
```

Namespace this binding is scoped to.

Implements: `KvNamespace`

### OutboxQueueProducer

```rust
pub struct OutboxQueueProducer {
    /* private fields */
}
```

[`QueueProducer`](../bookclerk_plugin_abi/index.md#queueproducer) server for one binding of one invocation: validates a
guest [`QueueSendMessage`](../bookclerk_plugin_abi/generated.md#queuesendmessage) and appends it to the library outbox.

#### OutboxQueueProducer::queue

```rust
pub fn queue(&self) -> &str
```

Queue this producer sends to.

#### OutboxQueueProducer::spec

```rust
pub fn spec(
    &self,
    message: QueueSendMessage,
    default_delay: u32,
) -> Result<PublishDomainEventSpec, PluginError>
```

Validates `message` and builds the outbox row the host will insert.
`default_delay` applies when the message carries no delay of its own.

##### Errors

* `payload_too_large` when `body` exceeds [`MAX_EVENT_PAYLOAD_BYTES`](../bookclerk_plugin_abi/index.md#max_event_payload_bytes).
* `invalid_params` when `contentType` is unknown, a `json` body is not
  UTF-8 JSON, or a `text` body is not UTF-8.

Implements: `QueueProducer`

### QueueOutbox

```rust
pub struct QueueOutbox {
    /* private fields */
}
```

Host-owned facts a session needs before it can hand a guest its queue
producer bindings: the outbox store plus the granted `(binding, queue)`
pairs.

#### QueueOutbox::new

```rust
pub fn new(
    store: LibraryStore,
    plugin_id: &str,
    manifest: &PluginManifest,
    grant: &PluginGrant,
) -> Option<Self>
```

Builds the outbox hook for `plugin_id`, or `None` when no producer row
is both declared and granted (the guest then sees no queue bindings).

#### QueueOutbox::binding_names

```rust
pub fn binding_names(&self) -> Vec<String>
```

Granted binding names in manifest order.

#### QueueOutbox::producers

```rust
pub fn producers(
    &self,
    invocation: &Invocation,
) -> Vec<(String, String, Arc<dyn QueueProducer>)>
```

Producers bound to one `PluginWorker.open` invocation, shaped for
`HostBindings::queues` as `(binding, queue, producer)`.

Implements: `Clone`

### BookclerkPackageMetadata

```rust
pub struct BookclerkPackageMetadata {
    pub api_version: u32,
    pub family: PluginFamily,
    pub id: String,
    pub display_name: Option<String>,
    pub artifact_base_url: Option<String>,
    pub artifact_url: Option<String>,
    pub archive_root: Option<String>,
    pub min_host: Option<String>,
}
```

`[package.metadata.bookclerk]` published on crates.io.

**Fields**

- `api_version`: Plugin ABI / package metadata API version (must be >= 1).
- `family`: Plugin family encoded in the crates.io name.
- `id`: Plugin id (`[a-z0-9_]{2,32}`), unique across kinds in one host namespace.
- `display_name`: Optional human-readable name for catalog UI.
- `artifact_base_url`: Template with `{tag}`, `{version}`, `{target}`, `{crate}` placeholders.
  
  Any HTTPS host works (GitHub/GitLab/Codeberg releases, S3/R2, CDN,
  self-hosted). Combined with the conventional archive filename unless
  [`Self::artifact_url`](#bookclerkpackagemetadata) is set.
- `artifact_url`: Full download URL template (overrides base + conventional filename).
  
  Placeholders: `{tag}`, `{version}`, `{target}`, `{crate}`, `{ext}`.
- `archive_root`: Optional path inside the release archive where `plugin.toml` lives.
- `min_host`: Optional minimum Bookclerk host version required to run this plugin.

#### BookclerkPackageMetadata::validate_against_crate_name

```rust
pub fn validate_against_crate_name(&self, crate_name: &str) -> Result<()>
```

Ensure metadata matches the crate naming taxonomy.

##### Errors

Returns an error when the operation fails.

#### BookclerkPackageMetadata::artifact_download_url

```rust
pub fn artifact_download_url(
    &self,
    crate_name: &str,
    version: &str,
    target: &str,
) -> Option<String>
```

Build a download URL for a release asset.

Prefers the `artifact_url` metadata field when set; otherwise
`{artifact_base_url}/{crate}-{version}-{target}.{ext}` with `{ext}` =
`zip` for Windows targets and `tar.gz` otherwise.

Implements: `Clone`, `Debug`, `Deserialize<'de>`, `Eq`, `PartialEq`, `Serialize`

### PluginCatalogEntry

```rust
pub struct PluginCatalogEntry {
    pub crate_name: String,
    pub version: String,
    pub description: Option<String>,
    pub downloads: u64,
    pub documentation: Option<String>,
    pub repository: Option<String>,
    pub homepage: Option<String>,
    pub parsed: Option<PluginCrateName>,
    pub metadata: Option<BookclerkPackageMetadata>,
}
```

One catalog hit from crates.io (or a curated index).

**Fields**

- `crate_name`: crates.io package name (`bookclerk-plugin-{family}-{id}`).
- `version`: Published crate / release version string.
- `description`: Short crates.io description when the index provides one.
- `downloads`: Download count from the registry index, when known.
- `documentation`: Documentation URL from crate metadata, when present.
- `repository`: Source repository URL from crate metadata, when present.
- `homepage`: Project homepage URL from crate metadata, when present.
- `parsed`: Parsed from crate name when it matches the taxonomy.
- `metadata`: Present when crates.io returned Cargo.toml metadata (install phase).

Implements: `Clone`, `Debug`, `Deserialize<'de>`, `Eq`, `PartialEq`, `Serialize`

### PluginCrateName

```rust
pub struct PluginCrateName {
    pub family: PluginFamily,
    pub id: String,
}
```

Parsed crates.io crate name for a Bookclerk plugin.

**Fields**

- `family`: Plugin family encoded in the crates.io name.
- `id`: Plugin id (`[a-z0-9_]{2,32}`), unique across kinds in one host namespace.

#### PluginCrateName::crate_name

```rust
pub fn crate_name(&self) -> String
```

Format as `bookclerk-plugin-{family}-{id}`.

#### PluginCrateName::parse

```rust
pub fn parse(name: &str) -> Result<Self>
```

Parse `bookclerk-plugin-{family}-{id}`.

##### Errors

Returns an error when the operation fails.

Implements: `Clone`, `Debug`, `Deserialize<'de>`, `Display`, `Eq`, `PartialEq`, `Serialize`

### RemoteNarrowing

```rust
pub struct RemoteNarrowing {
    pub deny_domains: BTreeSet<String>,
    pub deny_bindings: BTreeSet<String>,
    pub deny_network: bool,
}
```

What a host operator takes out of an owner plugin's request.

**Fields**

- `deny_domains`: Outbound domain patterns to deny.
- `deny_bindings`: Host bindings to deny (`secrets`, `oauth`, `kv`, ...).
- `deny_network`: Turn outbound network off altogether.

Implements: `Clone`, `Debug`, `Default`, `Eq`, `PartialEq`

### ExecutorIdentity

```rust
pub struct ExecutorIdentity {
    pub plugin_id: String,
    pub artifact_digest: String,
    pub version: String,
    pub role: String,
    pub account_id: String,
    pub configuration_revision: String,
    pub grant_revision: String,
    pub authority_revision: String,
    pub runtime_backend: String,
    pub compatibility_date: String,
}
```

Expanded executor identity. Pooling is an optimization; correctness must not
depend on a PID surviving.

**Fields**

- `plugin_id`: Provenance-qualified [`bookclerk_plugin_catalog::PluginKey`](../bookclerk_plugin_catalog/index.md#pluginkey) (canonical text).
- `artifact_digest`: Artifact payload-root digest (or install-path stand-in when unsigned/dev).
- `version`: Manifest version.
- `role`: Role (`destination`, `source`, `database`, `integration`).
- `account_id`: Account / principal.
- `configuration_revision`: Configuration revision (installed `plugin.toml` SHA-256).
- `grant_revision`: Grant revision (persisted operator consent; revocation changes this).
- `authority_revision`: Effective runtime authority (host overlays + clamped budgets).
- `runtime_backend`: `workerd` / `native-behind-workerd` / `native-direct`
  ([`GuestRuntimeKind::label`](#guestruntimekindlabel)).
- `compatibility_date`: Workerd compatibility date when applicable.

#### ExecutorIdentity::from_plugin

```rust
pub fn from_plugin(plugin: &DiscoveredPlugin, account_id: &str) -> Self
```

Builds an identity from a discovered plugin and account on the default
(front-door) transport.

#### ExecutorIdentity::from_plugin_on

```rust
pub fn from_plugin_on(
    plugin: &DiscoveredPlugin,
    account_id: &str,
    transport: SpawnTransport,
) -> Self
```

Builds an identity for `plugin` spawned over `transport`.

#### ExecutorIdentity::from_plugin_with_runtime

```rust
pub fn from_plugin_with_runtime(
    plugin: &DiscoveredPlugin,
    account_id: &str,
    runtime: GuestRuntimeKind,
) -> Self
```

Builds an identity whose `runtime_backend` is the resolved launcher tree.

#### ExecutorIdentity::with_overlay_config

```rust
pub fn with_overlay_config(self, config: &Config) -> Self
```

Folds overlay-relevant host config into [`Self::configuration_revision`](#executoridentity).

Changing a Postgres URL / D1 API origin / S3 endpoint / Audiobookshelf
URL must not reuse a session that was spawned under the old overlay.

#### ExecutorIdentity::with_grant_revision

```rust
pub fn with_grant_revision(self, grant: &PluginGrant) -> Self
```

Fills persisted [`Self::grant_revision`](#executoridentity) and effective
[`Self::authority_revision`](#executoridentity) from a grant snapshot.

When host overlays apply, pass the persisted grant to
[`Self::with_persisted_and_effective`](#executoridentitywith_persisted_and_effective).

#### ExecutorIdentity::with_persisted_and_effective

```rust
pub fn with_persisted_and_effective(
    self,
    persisted: &PluginGrant,
    effective: &PluginGrant,
) -> Self
```

Fills revisions from a persisted operator grant and the effective runtime grant.

#### ExecutorIdentity::session_key

```rust
pub fn session_key(&self) -> String
```

Stable session key. Distinct PIDs with the same key are the same logical
instance; pooling must use this key, not a PID.

Implements: `Clone`, `Debug`, `Eq`, `Hash`, `PartialEq`

### PluginSession

```rust
pub struct PluginSession {
    /* private fields */
}
```

Host-side plugin session (one jailed child + one vat thread).

#### PluginSession::spawn

```rust
pub async fn spawn(
    plugin: &DiscoveredPlugin,
    config: &Config,
    config_table: Value,
) -> Result<Self>
```

Spawns a plugin guest and connects Cap'n Proto on stdio.

##### Errors

Fails when the child cannot start, describe fails, or `apiVersion` is not 2.

#### PluginSession::spawn_for_account

```rust
pub async fn spawn_for_account(
    plugin: &DiscoveredPlugin,
    config: &Config,
    config_table: Value,
    account_id: &str,
) -> Result<Self>
```

[`Self::spawn`](#pluginsessionspawn) keyed by `(plugin_id, account_id)` so different accounts
never share a plugin isolate.

##### Errors

Fails when the child cannot start, describe fails, or negotiation fails.

#### PluginSession::spawn_for_account_with_env

```rust
pub async fn spawn_for_account_with_env(
    plugin: &DiscoveredPlugin,
    config: &Config,
    config_table: Value,
    account_id: &str,
    extra_env: &[(&str, OsString)],
) -> Result<Self>
```

[`Self::spawn_for_account`](#pluginsessionspawn_for_account) with transport-private extra environment.

##### Errors

Fails when the child cannot start, describe fails, or negotiation fails.

#### PluginSession::spawn_with

```rust
pub async fn spawn_with(
    plugin: &DiscoveredPlugin,
    config: &Config,
    config_table: Value,
    account_id: &str,
    extra_env: &[(&str, OsString)],
    services: SessionServices,
) -> Result<Self>
```

[`Self::spawn_for_account_with_env`](#pluginsessionspawn_for_account_with_env) plus the host services the guest
may receive as bindings (`EVENTS` outbox, …) and the spawn transport.

This is the one place every product spawn passes through: the
[`SpawnPlan`](#spawnplan) resolved here decides that a `runtime = "native"` manifest
is fronted by `bookclerk-workerd` (host-spawned sibling jails joined by
inherited links) unless `services.spawn_transport` opted into the
diagnostic direct transport.

##### Errors

Fails when the front door (`bookclerk-workerd` + pinned `workerd`) is
missing, the child cannot start, describe fails, or negotiation fails.

#### PluginSession::instance_key

```rust
pub fn instance_key(&self) -> &str
```

Isolation instance key (`plugin_id:account_id`).

#### PluginSession::account_id

```rust
pub fn account_id(&self) -> &str
```

Account scope carried on every `PluginWorker.open` invocation.

#### PluginSession::session_key

```rust
pub fn session_key(&self) -> &str
```

Expanded executor session key (artifact, role, grant revision, …).

#### PluginSession::guest_pid

```rust
pub fn guest_pid(&self) -> Option<u32>
```

Native guest PID (sibling jail) or the single child when there is none.

#### PluginSession::gateway_pid

```rust
pub fn gateway_pid(&self) -> Option<u32>
```

Gateway / Cap'n Proto child PID for native-behind-workerd, when known.

#### PluginSession::guest_running

```rust
pub fn guest_running(&self) -> bool
```

True when the guest process, and the gateway when one was spawned, are
still running.

#### PluginSession::session_dir

```rust
pub fn session_dir(&self) -> Option<&Path>
```

Host-owned gateway session directory, when this session has a sibling.

#### PluginSession::limits

```rust
pub fn limits(&self) -> ScalarLimits
```

Negotiated scalar limits.

#### PluginSession::supports_server_copy

```rust
pub fn supports_server_copy(&self) -> bool
```

True when the guest accepted `storage.copy`.

#### PluginSession::id

```rust
pub fn id(&self) -> &str
```

Provenance-qualified PluginKey (canonical text).

#### PluginSession::alias

```rust
pub fn alias(&self) -> &str
```

Manifest display / CLI alias (`plugin.toml` `id`).

#### PluginSession::data_dir

```rust
pub fn data_dir(&self) -> &Path
```

Guest data directory.

#### PluginSession::open

```rust
pub async fn open(&self, values: BindingValues) -> Result<()>
```

Opens the session's primary entrypoints with the granted binding
values (`CONFIG` / `SECRETS`).

Idempotent for equal values; different values re-open. Entrypoint
calls that run before `open` use empty bindings.

##### Errors

Returns a plugin error when `PluginWorker.open` fails.

#### PluginSession::describe

```rust
pub async fn describe(&self) -> Result<PluginDescribe>
```

Calls `PluginWorker.describe`.

##### Errors

Returns a plugin error when the RPC fails.

#### PluginSession::stream_copy

```rust
pub async fn stream_copy(&self, job_id: &str, from: &str, to: &str) -> Result<JobOutcome>
```

Runs the stream-copy job handler on the guest.

##### Errors

Returns a plugin error when the handler fails.

#### PluginSession::stream_copy_with_cancel

```rust
pub async fn stream_copy_with_cancel(
    &self,
    lease: JobInvocationLease,
    from: &str,
    to: &str,
    cancel: Arc<AtomicBool>,
    progress: Option<(LibraryStore, JobFence)>,
) -> Result<JobOutcome>
```

[`Self::stream_copy`](#pluginsessionstream_copy) raced against a host cancel/fence flag.

When `progress` is set, reports are persisted with
`LibraryStore::set_job_progress`; a lost fence surfaces as cancellation.

##### Errors

Returns a plugin error when the handler fails or the fence is lost.

#### PluginSession::stream_copy_with_databases

```rust
pub async fn stream_copy_with_databases(
    &self,
    lease: JobInvocationLease,
    from: &str,
    to: &str,
    cancel: Arc<AtomicBool>,
    progress: Option<(LibraryStore, JobFence)>,
    databases: Vec<(String, GuestDatabaseFactory)>,
) -> Result<JobOutcome>
```

[`Self::stream_copy_with_cancel`](#pluginsessionstream_copy_with_cancel) with named plugin database bindings.

Each `(name, factory)` pair becomes an isolated `GuestDatabase` on the
per-job `PluginWorker.open` bindings; factories run on the vat thread.

##### Errors

Returns a plugin error when the handler fails or the fence is lost.

#### PluginSession::describe_snapshot

```rust
pub fn describe_snapshot(&self) -> &PluginDescribe
```

Snapshot from the guest `describe()` call at spawn.

#### PluginSession::grant

```rust
pub fn grant(&self) -> &PluginGrant
```

Covering consent grant from spawn.

#### PluginSession::require_binding

```rust
pub fn require_binding(&self, name: &str) -> Result<()>
```

Fail closed when a delivery site needs an ungranted binding.

##### Errors

Returns an error when the binding is missing from the grant.

#### PluginSession::scratch_dir

```rust
pub fn scratch_dir(&self) -> &Path
```

Guest TMPDIR / scratch directory.

#### PluginSession::guest_ipc_dir

```rust
pub fn guest_ipc_dir(&self) -> Option<&Path>
```

Directory the guest may use for pathname sockets, when this session has one.

#### PluginSession::package_sid

```rust
pub fn package_sid(&self) -> Option<&str>
```

AppContainer package SID when the guest is jailed on Windows.

#### PluginSession::spawn_config

```rust
pub fn spawn_config(&self) -> &Value
```

Spawn config JSON captured at spawn.

#### PluginSession::has_entrypoint

```rust
pub fn has_entrypoint(&self, entrypoint: Entrypoint) -> bool
```

True when the guest exported `entrypoint` in `describe()`.

#### PluginSession::consumes_event

```rust
pub fn consumes_event(&self, event_type: &str) -> bool
```

True when the guest's default entrypoint consumes `event_type`.

#### PluginSession::consumes_events

```rust
pub fn consumes_events(&self) -> bool
```

True when the guest's default entrypoint consumes any event type.

#### PluginSession::consumes_queues

```rust
pub fn consumes_queues(&self) -> bool
```

True when the guest's default entrypoint consumes at least one
`[[queues.consumers]]` queue.

#### PluginSession::storefront

```rust
pub async fn storefront<T, F, Fut>(&self, call: F) -> Result<T>
where
    T: Send + 'static,
    F: FnOnce(Box<dyn ContentSource>) -> Fut + Send + 'static,
    Fut: Future<Output = Result<T, PluginError>> + 'static,
```

One typed `storefront` method on the opened primary entrypoints.

`call` runs on the vat thread; its output crosses back to the caller's
runtime.

##### Errors

Returns a plugin error when the guest did not export `storefront` or
the method fails.

#### PluginSession::remote_library

```rust
pub async fn remote_library<T, F, Fut>(&self, call: F) -> Result<T>
where
    T: Send + 'static,
    F: FnOnce(Box<dyn RemoteLibrary>) -> Fut + Send + 'static,
    Fut: Future<Output = Result<T, PluginError>> + 'static,
```

One typed `remoteLibrary` method on the opened primary entrypoints.

##### Errors

Returns a plugin error when the guest did not export `remoteLibrary`
or the method fails.

#### PluginSession::remote_library_cancelable

```rust
pub async fn remote_library_cancelable<T, F, Fut>(
    &self,
    cancel: Arc<AtomicBool>,
    call: F,
) -> Result<T>
where
    T: Send + 'static,
    F: FnOnce(Box<dyn RemoteLibrary>) -> Fut + Send + 'static,
    Fut: Future<Output = Result<T, PluginError>> + 'static,
```

Typed `remoteLibrary` method aborted when `cancel` is set.

##### Errors

Returns a plugin error when the RPC fails or is cancelled.

#### PluginSession::deliver_events

```rust
pub async fn deliver_events(
    &self,
    batch: Vec<DomainEvent>,
    cancel: Arc<AtomicBool>,
) -> Result<Vec<EventResult>>
```

Delivers one ordered event batch to the guest `eventConsumer` trigger
(`EventConsumer.event`), aborted when `cancel` is set (fence loss).

Returns exactly one [`EventResult`](../bookclerk_plugin_abi/index.md#eventresult) per input event.

##### Errors

Returns a plugin error when the guest did not export an event
consumer, the RPC fails, or the delivery is cancelled.

#### PluginSession::deliver_queue

```rust
pub async fn deliver_queue(
    &self,
    batch: QueueBatch,
    cancel: Arc<AtomicBool>,
) -> Result<Vec<QueueResult>>
```

Delivers one ordered batch from a `[[queues.consumers]]` queue to the
guest's `queue(batch)` trigger.

Returns exactly one [`bookclerk_plugin_abi::QueueResult`](../bookclerk_plugin_abi/generated.md#queueresult) per message.

##### Errors

Returns a plugin error when the grant does not cover the queue, the
guest did not export a queue consumer, the RPC fails, or the delivery
is cancelled.

#### PluginSession::cli_describe

```rust
pub async fn cli_describe(&self) -> Result<CliSchema>
```

Guest CLI schema.

##### Errors

Returns a plugin error when the RPC fails.

#### PluginSession::cli_invoke

```rust
pub async fn cli_invoke(&self, params: CliInvokeParams) -> Result<CliInvokeResult>
```

Guest CLI invoke.

##### Errors

Returns a plugin error when the RPC fails.

#### PluginSession::oidc_clients

```rust
pub async fn oidc_clients(&self) -> Result<Vec<OidcClientTemplate>>
```

Plugin-provided OIDC authorization-server client templates
(`Oidc.clients`); empty when the guest exports no `oidc` entrypoint.

##### Errors

Returns a plugin error when the RPC fails.

#### PluginSession::oidc_authenticate_user

```rust
pub async fn oidc_authenticate_user(
    &self,
    params: AuthenticateUserParams,
) -> Result<ExternalUser>
```

Verifies remote credentials through the guest `oidc` entrypoint
(`Oidc.authenticateUser`).

##### Errors

Returns a plugin error when the guest exports no `oidc` entrypoint or
the RPC fails.

#### PluginSession::database_migrations

```rust
pub async fn database_migrations(&self, binding: &str) -> Result<Vec<PluginMigration>>
```

Complete ordered plugin-owned migration sequence for one named binding.

##### Errors

Returns a plugin error when the RPC fails.

#### PluginSession::db_open

```rust
pub async fn db_open(&self, values: BindingValues) -> Result<()>
```

Opens the library adapter session (held on the vat until drop).

Performs a dedicated `PluginWorker.open` whose bindings carry the
host-private connect params in `values`, then `openSession` on the
returned `databaseAdapter` entrypoint.

##### Errors

Returns a plugin error when `open` / `openSession` fails or the guest
exports no `databaseAdapter`.

#### PluginSession::db_drop_unit

```rust
pub async fn db_drop_unit(&self, values: BindingValues, unit_ref: &str) -> Result<()>
```

Physically drops one provisioned binding unit via `Database.dropUnit`.

Opens the adapter factory with `values` (library connect params) and
does not retain a session. Used by `plugins db drop`.

##### Errors

Returns a plugin error when `open` / `dropUnit` fails or the guest
exports no `databaseAdapter`.

#### PluginSession::db_open_binding

```rust
pub async fn db_open_binding(
    &self,
    name: &str,
    values: BindingValues,
) -> Result<DbCapabilities>
```

Opens an isolated adapter session for one named plugin database binding.

Negotiates [`bookclerk_plugin_sdk::DbCapabilities`](../bookclerk_plugin_abi/db_execute.md#dbcapabilities) on the binding
session itself (not the library session).

##### Errors

Returns a plugin error when the guest rejects the open or the binding
session fails host capability minima.

#### PluginSession::db_execute_binding_request

```rust
pub async fn db_execute_binding_request(
    &self,
    name: &str,
    request: AdapterExecuteRequest,
    cancel: Arc<AtomicBool>,
) -> Result<ExecuteReply>
```

Typed execute on a named plugin database binding session.

##### Errors

Returns a plugin error when the binding is not open or the guest
rejects the call.

#### PluginSession::db_execute_binding_envelope_request

```rust
pub async fn db_execute_binding_envelope_request(
    &self,
    name: &str,
    envelope: AdapterExecuteRequest,
    cancel: Arc<AtomicBool>,
) -> Result<ExecuteReply>
```

Host-private envelope execute on a named plugin database binding.

Forwards [`bookclerk_plugin_abi::AdapterExecuteRequest`](../bookclerk_plugin_abi/index.md#adapterexecuterequest) so the adapter
persists guest receipt payloads before COMMIT.

##### Errors

Returns a plugin error when the binding is not open, the guest rejects
the call, or `cancel` is set.

#### PluginSession::db_capabilities

```rust
pub async fn db_capabilities(&self) -> Result<DbCapabilities>
```

Typed `AdapterDatabaseSession.capabilities`.

##### Errors

Returns a plugin error when the guest rejects the call.

#### PluginSession::db_bootstrap

```rust
pub async fn db_bootstrap(&self) -> Result<DbBootstrap>
```

Typed `AdapterDatabaseSession.bootstrap`.

##### Errors

Returns a plugin error when the guest rejects the call or lacks `bootstrap`.

#### PluginSession::db_execute_request

```rust
pub async fn db_execute_request(
    &self,
    request: AdapterExecuteRequest,
    cancel: Arc<AtomicBool>,
) -> Result<ExecuteReply>
```

Typed `AdapterDatabaseSession.execute`.

##### Errors

Returns a plugin error when the guest rejects the call or `cancel` is set.

#### PluginSession::db_execute_envelope_request

```rust
pub async fn db_execute_envelope_request(
    &self,
    envelope: AdapterExecuteRequest,
    cancel: Arc<AtomicBool>,
) -> Result<ExecuteReply>
```

Typed `HostAdapterDatabaseSession.execute`.

##### Errors

Returns a plugin error when the guest rejects the call or `cancel` is set.

#### PluginSession::db_txn_execute_request

```rust
pub async fn db_txn_execute_request(
    &self,
    request: AdapterExecuteRequest,
    cancel: Arc<AtomicBool>,
) -> Result<ExecuteReply>
```

Typed `AdapterTransaction.execute` on the vat-held open transaction.

##### Errors

Returns a plugin error when no transaction is open, the guest rejects
the call, or `cancel` is set.

#### PluginSession::db_begin

```rust
pub async fn db_begin(&self, isolation: IsolationReq) -> Result<()>
```

Begin a vat-held transaction.

##### Errors

Returns a plugin error when begin fails.

#### PluginSession::db_commit

```rust
pub async fn db_commit(&self) -> Result<()>
```

Commit the vat-held transaction.

##### Errors

Returns a plugin error when commit fails.

#### PluginSession::db_rollback

```rust
pub async fn db_rollback(&self) -> Result<()>
```

Roll back the vat-held transaction.

##### Errors

Returns a plugin error when rollback fails.

#### PluginSession::db_begin_binding

```rust
pub async fn db_begin_binding(&self, name: &str, isolation: IsolationReq) -> Result<()>
```

Begin a transaction on a named plugin database binding.

##### Errors

Returns when the binding is not open or begin fails.

#### PluginSession::db_commit_binding

```rust
pub async fn db_commit_binding(&self, name: &str) -> Result<()>
```

Commit a named plugin database binding transaction.

##### Errors

Returns when no binding transaction is open or commit fails.

#### PluginSession::db_rollback_binding

```rust
pub async fn db_rollback_binding(&self, name: &str) -> Result<()>
```

Roll back a named plugin database binding transaction.

##### Errors

Returns when no binding transaction is open or rollback fails.

#### PluginSession::db_txn_execute_binding_request

```rust
pub async fn db_txn_execute_binding_request(
    &self,
    name: &str,
    request: AdapterExecuteRequest,
    cancel: Arc<AtomicBool>,
) -> Result<ExecuteReply>
```

Typed execute on the vat-held binding transaction.

##### Errors

Returns when no binding transaction is open or the guest rejects the call.

#### PluginSession::db_export_identity

```rust
pub async fn db_export_identity(
    &self,
    binding: Option<&str>,
) -> Result<Vec<DbIdentityHighWater>>
```

Identity high-water from the adapter (open txn if any).

##### Errors

Returns when the session is closed or the guest rejects the call.

#### PluginSession::db_import_identity

```rust
pub async fn db_import_identity(
    &self,
    binding: Option<&str>,
    rows: Vec<DbIdentityHighWater>,
) -> Result<()>
```

Restore identity high-water into the adapter.

##### Errors

Returns when the session is closed or the guest rejects the call.

#### PluginSession::db_list_user_relations

```rust
pub async fn db_list_user_relations(&self, binding: Option<&str>) -> Result<Vec<String>>
```

User-visible relation names from the adapter.

##### Errors

Returns when the session is closed or the guest rejects the call.

#### PluginSession::db_prepare_unit_restore

```rust
pub async fn db_prepare_unit_restore(&self, binding: Option<&str>) -> Result<()>
```

Prepare the open restore transaction.

##### Errors

Returns when the session is closed or the guest rejects the call.

#### PluginSession::db_drop_user_relations

```rust
pub async fn db_drop_user_relations(
    &self,
    binding: Option<&str>,
    names: Vec<String>,
) -> Result<()>
```

Drop named user relations.

##### Errors

Returns when the session is closed or the guest rejects the call.

#### PluginSession::db_assert_restore_constraints

```rust
pub async fn db_assert_restore_constraints(&self, binding: Option<&str>) -> Result<()>
```

Fail closed when restore FK checks still fail.

##### Errors

Returns when the session is closed or the guest rejects the call.

Implements: `Drop`

### PluginStorage

```rust
pub struct PluginStorage {
    /* private fields */
}
```

[`StorageBackend`](../bookclerk_storage/index.md#storagebackend) over a plugin destination capability (streams, fail-closed scalars).

#### PluginStorage::new

```rust
pub fn new(session: Arc<PluginSession>) -> Self
```

Wraps a connected session after [`PluginSession::open`](#pluginsessionopen) granted the
`storage` entrypoint.

Implements: `Clone`, `StorageBackend`

### RpcBackupOps

```rust
pub struct RpcBackupOps {
    /* private fields */
}
```

Send-safe backup primitives over a vat-held adapter session (or its open txn).

#### RpcBackupOps::library

```rust
pub fn library(session: Arc<PluginSession>) -> Self
```

Library adapter session (no named binding).

#### RpcBackupOps::binding

```rust
pub fn binding(session: Arc<PluginSession>, name: impl Into<String>) -> Self
```

Named plugin-database binding (`plugin_id/binding`).

#### RpcBackupOps::shared

```rust
pub fn shared(self) -> SharedAdapterBackupOps
```

Shared handle stored on backup/restore options.

Implements: `AdapterBackupOps`

### SessionServices

```rust
pub struct SessionServices {
    pub event_outbox: Option<LibraryStore>,
    pub spawn_transport: SpawnTransport,
    pub selected_instance_id: Option<String>,
    pub local_output_root: Option<PathBuf>,
    pub deployed_endpoint: Option<Option<String>>,
}
```

Host services a session hands its guest as `Bindings` on every
`PluginWorker.open`.

Nothing here is guest-visible on its own: the session still gates each
binding on the manifest declaration plus the covering consent grant.

**Fields**

- `event_outbox`: Library store whose outbox backs the `EVENTS` binding. `None` (the
  default) never exposes `EVENTS`, even to a granted producer.
- `spawn_transport`: How the guest is reached. The default fronts every guest with
  `bookclerk-workerd`; [`SpawnTransport::DirectNativeDiagnostic`](#spawntransport) is for
  tests and diagnostics only and no product binary selects it.
- `selected_instance_id`: Plugin instance id selected by CLI scan or acquire.
  
  `None` uses the only instance document for a plugin key, or transitional
  file settings when that key has no document. Several documents require
  this id. An id selects only the plugin key it belongs to.
- `local_output_root`: Writable root for a deployed local destination.
  
  Set from the deployed local destination's instance document; it is the
  jail write grant. `None` grants no output directory.
- `deployed_endpoint`: Network authority from the deployed instance document: the S3
  `endpoint` or the Audiobookshelf `base_url`.
  
  `Some(Some(url))` grants that host in addition to the manifest's fixed
  TCP list; `Some(None)` records that the document names none; `None`
  means this spawn is not a deployed instance of such a plugin.

#### SessionServices::with_event_outbox

```rust
pub fn with_event_outbox(store: LibraryStore) -> Self
```

Services with the library outbox attached.

#### SessionServices::from_outbox

```rust
pub fn from_outbox(store: Option<&LibraryStore>) -> Self
```

Services with the library outbox attached when `store` is present.

#### SessionServices::direct_native_diagnostic

```rust
pub fn direct_native_diagnostic() -> Self
```

Default services on the direct native diagnostic transport.

For shell-probe jail tests and transport benchmarks: the host speaks
Cap'n Proto to the native guest's stdio with no `bookclerk-workerd`.

Implements: `Clone`, `Default`

### SpawnPlan

```rust
pub struct SpawnPlan {
    pub launcher: PathBuf,
    pub args: Vec<String>,
    pub native_backend: Option<PathBuf>,
    pub workerd_bin: Option<PathBuf>,
    pub runtime: GuestRuntimeKind,
}
```

Everything the spawn path needs to start one guest.

**Fields**

- `launcher`: Program the jail execs: `bookclerk-workerd`, or the native command on
  the diagnostic transport.
- `args`: Arguments after `launcher` (manifest `args`; diagnostic transport only).
- `native_backend`: Native executable the host spawns as the sibling guest. `None` for
  author isolates and direct native.
- `workerd_bin`: Pinned `workerd` the launcher will spawn (must stay readable in the jail).
- `runtime`: Process tree this plan produces.

#### SpawnPlan::resolve

```rust
pub fn resolve(plugin: &DiscoveredPlugin, transport: SpawnTransport) -> Result<Self>
```

Resolves the launch for `plugin` on `transport` using the host's front door.

##### Errors

Fails when the front door is required but `bookclerk-workerd` or the
pinned `workerd` cannot be found, when a native manifest declares `args`
(the launcher spawns the backend with none), or when a `workerd` manifest
asks for the diagnostic transport.

#### SpawnPlan::resolve_with

```rust
pub fn resolve_with(
    plugin: &DiscoveredPlugin,
    transport: SpawnTransport,
    locate: impl FnOnce() -> Result<WorkerdFrontDoor>,
) -> Result<Self>
```

[`Self::resolve`](#spawnplanresolve) with an injected front-door locator (called only when
the plan needs `bookclerk-workerd`).

##### Errors

Same as [`Self::resolve`](#spawnplanresolve); locator failures are wrapped in the
front-door refusal message.

#### SpawnPlan::fronted_by_workerd

```rust
pub fn fronted_by_workerd(&self) -> bool
```

True when `bookclerk-workerd` is the program the jail execs.

#### SpawnPlan::executable_reads

```rust
pub fn executable_reads(&self) -> Vec<PathBuf>
```

Executable paths the jail must let the launcher tree read and exec.

For [`GuestRuntimeKind::NativeBehindWorkerd`](#guestruntimekind) this is the union of
[`Self::gateway_executable_reads`](#spawnplangateway_executable_reads) and [`Self::guest_executable_reads`](#spawnplanguest_executable_reads).

#### SpawnPlan::gateway_executable_reads

```rust
pub fn gateway_executable_reads(&self) -> Vec<PathBuf>
```

Executables the gateway jail may read: `bookclerk-workerd` and `workerd`.

Direct native uses the guest command as the (only) launcher.

#### SpawnPlan::guest_executable_reads

```rust
pub fn guest_executable_reads(&self) -> Vec<PathBuf>
```

Executables the native sibling jail may read (the backend only).

Implements: `Clone`, `Debug`, `Eq`, `PartialEq`

### WorkerdFrontDoor

```rust
pub struct WorkerdFrontDoor {
    pub launcher: PathBuf,
    pub workerd_bin: PathBuf,
}
```

Resolved `bookclerk-workerd` launcher and the pinned `workerd` beside it.

**Fields**

- `launcher`: `bookclerk-workerd` executable the jail execs.
- `workerd_bin`: Pinned Cloudflare `workerd` the launcher spawns.

#### WorkerdFrontDoor::locate

```rust
pub fn locate() -> Result<Self>
```

Locates the front door for this host process.

Launcher: [`WORKERD_LAUNCHER_ENV`](#workerd_launcher_env) when set (must be a file), else
beside the current executable (or its `deps/` parent for cargo test
binaries), else on `PATH`. `workerd`: [`WORKERD_BIN_ENV`](#workerd_bin_env) when set
(must be a file), else beside the launcher.

##### Errors

Returns an error naming what was looked for and where when either
binary is missing.

#### WorkerdFrontDoor::locate_in

```rust
pub fn locate_in(dir: &Path) -> Result<Self>
```

Locates the front door looking **only** in `dir` (no environment
overrides, no `PATH`). A test seam for missing-launcher planning.

##### Errors

Returns an error when `dir` lacks `bookclerk-workerd` or `workerd`.

Implements: `Clone`, `Debug`, `Eq`, `PartialEq`

## Enums

### SpawnHealth

```rust
pub enum SpawnHealth {
    Healthy,
    SpawnFailed { detail: String },
    HealthFailed { detail: String },
}
```

Result of spawn plus the health RPC.

**Variants**

- `Healthy`: Health succeeded.
- `SpawnFailed`: The guest did not start. No config revision was applied.
  - `detail`: Bounded later by the observation writer.
- `HealthFailed`: The guest started and health failed. The install tree stays.
  - `detail`: Bounded later by the observation writer.

Implements: `Clone`, `Debug`, `Eq`, `PartialEq`

### PluginError

```rust
pub enum PluginError {
    Message(String),
    InstanceSelection(String),
    Conflict(String),
    NotFound(String),
    Unavailable(String),
    Io(Error),
    Json(Error),
    Toml(Error),
    Manifest(Error),
    Other(Error),
    Abi { code: String, message: String },
}
```

Failures from discovery, RPC, consent, or plugin adapters.

**Variants**

- `Message`: Operator-facing error text with no structured code.
- `InstanceSelection`: Instance selection failed closed: missing, wrong plugin key, or ambiguous.
  
  Callers match this variant instead of the message text.
- `Conflict`: The request conflicts with cluster state (an alias owned by another
  plugin, a version installed with different content).
- `NotFound`: A plugin, version, or row the request names does not exist.
- `Unavailable`: Backend or plugin transport is temporarily unreachable; retry the same
  idempotency key when the caller still holds consume-once material.
- `Io`: Filesystem or process I/O failure.
- `Json`: JSON encode/decode failure on the RPC wire or grant file.
- `Toml`: TOML decode failure (settings tables, legacy paths).
- `Manifest`: Invalid or unsupported `plugin.toml` / package manifest.
- `Other`: Catch-all for otherwise unclassified failures.
- `Abi`: Guest ABI error with a preserved wire `code` (including `unknown`).
  - `code`: Wire error code (`not_found`, `unknown`, …).
  - `message`: Operator-facing explanation.

#### PluginError::message

```rust
pub fn message(msg: impl Into<String>) -> Self
```

Builds a [`PluginError::Message`](#pluginerror) from any displayable string.

##### Arguments

* `msg` - Operator-facing explanation; must not embed secrets.

##### Returns

A message-only [`PluginError`](#pluginerror).

#### PluginError::conflict

```rust
pub fn conflict(msg: impl Into<String>) -> Self
```

Builds a [`PluginError::Conflict`](#pluginerror).

#### PluginError::unavailable

```rust
pub fn unavailable(msg: impl Into<String>) -> Self
```

Builds a structured unavailable error (lost RPC, timeout, incomplete reply).

#### PluginError::instance_selection

```rust
pub fn instance_selection(msg: impl Into<String>) -> Self
```

Instance id, plugin key, or alias cannot select exactly one instance.

#### PluginError::is_instance_selection

```rust
pub fn is_instance_selection(&self) -> bool
```

True when spawn must stop instead of skipping this guest.

#### PluginError::from_abi

```rust
pub fn from_abi(code: Option<&str>, message: impl Into<String>) -> Self
```

Maps a guest ABI error, preserving [`unavailable`](#pluginerrorunavailable)
so atomic retries can classify without string matching.

#### PluginError::is_ambiguous_transport

```rust
pub fn is_ambiguous_transport(&self) -> bool
```

True when an atomic-execute caller should retry the same operation id.

Relies on the structured [`Self::Unavailable`](#pluginerror) variant (guest ABI
`unavailable`, RPC timeout, or a closed guest) rather than matching
`D1 HTTP` strings: permanent 4xx from D1 is `internal` / `invalid_params`.

Implements: `Debug`, `Display`, `Error`, `From<Error>`, `From<PluginError>`

### Activate

```rust
pub enum Activate {
    Always,
    IfNone,
    Never,
}
```

When installing moves the active pointer.

**Variants**

- `Always`: Always point at the new version (`update`, `--activate`).
- `IfNone`: Only when the plugin has no active version yet (first install).
- `Never`: Never; install only.

Implements: `Clone`, `Copy`, `Debug`, `Eq`, `PartialEq`

### RemoteApprovalError

```rust
pub enum RemoteApprovalError {
    NotRequested(String),
    Conflict(String),
    Invalid(String),
    Store(String),
}
```

Why [`approve_remote_request`](#approve_remote_request) refused, by cause, so a caller (the
operator API) can tell "review again" from "fix the input" from "retry
later".

**Variants**

- `NotRequested`: No request is recorded for that cluster and plugin.
- `Conflict`: The recorded request is not the reviewed one (its digest differs),
  or it changed between the read and the write (the store's
  compare-and-swap), or an automatic approval met a revoke: review it
  again.
- `Invalid`: The recorded request does not parse, the narrowing names something
  the request does not ask for, or a field is malformed.
- `Store`: The store failed (a fault that may pass).

Implements: `Debug`, `Display`, `Error`, `From<LibraryError>`

### RemoteConsentState

```rust
pub enum RemoteConsentState {
    Consented(Box<PluginGrant>),
    Pending,
    NeedsReapproval,
    Invalid,
}
```

How this host stands on one owner plugin's request.

**Variants**

- `Consented`: Approved and the request is within what was reviewed: the guest
  runs with this grant.
- `Pending`: Never approved (or revoked): waiting for the host's operator.
- `NeedsReapproval`: Approved once, but the request now asks for more than was reviewed
  or names another archive.
- `Invalid`: The request or the stored approval does not parse; nothing runs.

#### RemoteConsentState::grant

```rust
pub fn grant(&self) -> Option<&PluginGrant>
```

The grant to run with, when consented.

#### RemoteConsentState::as_str

```rust
pub fn as_str(&self) -> &'static str
```

Stable text for observations and listings.

Implements: `Clone`, `Debug`, `Eq`, `PartialEq`

### GuestRuntimeKind

```rust
pub enum GuestRuntimeKind {
    Workerd,
    NativeBehindWorkerd,
    NativeDirect,
}
```

Process tree the jail will contain, as reported in [`crate::ExecutorIdentity`](#executoridentity).

**Variants**

- `Workerd`: `bookclerk-workerd` + the pinned `workerd` running author modules.
- `NativeBehindWorkerd`: `bookclerk-workerd` + `workerd` (control plane) + the native guest.
- `NativeDirect`: The native guest alone, spoken to directly (diagnostic transport).

#### GuestRuntimeKind::label

```rust
pub fn label(self) -> &'static str
```

Stable label used in executor identities and logs.

#### GuestRuntimeKind::process_overhead

```rust
pub fn process_overhead(self) -> u32
```

Payload occupancy before guest-owned children.

`NativeDirect`: the guest (1). `Workerd`: `bookclerk-workerd` plus the
pinned `workerd` (2). `NativeBehindWorkerd`: gateway payload (2) plus
the native guest (1) = 3. This count does **not** include the two
`bookclerk-jail` supervisors. The Windows outer session Job adds those
supervisors ([`crate::consent::WINDOWS_SESSION_PROCESS_BASELINE`](#windows_session_process_baseline)); Linux
`pids.max` stays on a thread budget and must not reuse the Windows
process total.

#### GuestRuntimeKind::fronted_by_workerd

```rust
pub fn fronted_by_workerd(self) -> bool
```

True when `bookclerk-workerd` is the program the jail execs.

#### GuestRuntimeKind::for_manifest

```rust
pub fn for_manifest(runtime: PluginRuntimeKind, transport: SpawnTransport) -> Self
```

Runtime kind a manifest resolves to under `transport`.

Implements: `Clone`, `Copy`, `Debug`, `Eq`, `Hash`, `PartialEq`

### SpawnTransport

```rust
pub enum SpawnTransport {
    WorkerdFrontDoor,
    DirectNativeDiagnostic,
}
```

Which transport a spawn uses to reach the guest.

**Variants**

- `WorkerdFrontDoor`: Product path: `bookclerk-workerd` fronts every guest, native or isolate.
- `DirectNativeDiagnostic`: Host ↔ native Cap'n Proto on the guest's own stdio, no isolate.
  
  Diagnostic only: shell-probe jail tests and transport benchmarks. A
  `runtime = "workerd"` manifest cannot use it (there is no native guest
  to talk to) and the product binaries never select it.

Implements: `Clone`, `Copy`, `Debug`, `Default`, `Eq`, `PartialEq`

## Functions

### apply_grant_store

```rust
pub fn apply_grant_store(store: &PluginGrantStore)
```

Reconcile every live session against a loaded **persisted** grant store.

Missing keys and [`crate::grant_revision`](#grant_revision) mismatches are fenced. Effective
[`authority_revision`](#authority_revision) (host overlays, clamped budgets) is **not** compared
to the persisted digest. Host-config overlay changes fence through
[`crate::ExecutorIdentity::configuration_revision`](#executoridentity).

### authority_revision

```rust
pub fn authority_revision(grant: &PluginGrant) -> String
```

Canonical SHA-256 of security-relevant **effective** authority.

Hash the grant **after** manifest ∩ operator ∩ host policy and host-controlled
overlays (TCP/CIDR implied by configured destinations, clamped budgets).
Omits `approved_at` and display aliases except the PluginKey. Distinct from
[`crate::grant_revision`](#grant_revision) and [`crate::ExecutorIdentity::configuration_revision`](#executoridentity).

### fence_plugin_key

```rust
pub fn fence_plugin_key(plugin_key: &str)
```

Fences every live session for `plugin_key` (grant/authority change).

### fence_stale_grant_revisions

```rust
pub fn fence_stale_grant_revisions(plugin_key: &str, current_grant_revision: &str)
```

Fences live sessions whose **persisted** [`crate::grant_revision`](#grant_revision) is not current.

### fence_stale_sessions

```rust
pub fn fence_stale_sessions(plugin_key: &str, current_revision: &str)
```

Fences live sessions for `plugin_key` whose revision is not `current_revision`.

Pass an empty `current_revision` to fence every session for the key.
Each fenced session is cancelled **and** its shutdown hook is invoked
immediately (vat `Work::Shutdown`, which kills the child and mediated
sockets). Hooks run after the live-table lock is released.

### is_fenced

```rust
pub fn is_fenced(flag: &Arc<AtomicBool>) -> bool
```

True when this cancel flag has been fenced.

### live_session_count

```rust
pub fn live_session_count() -> usize
```

Sessions currently registered with the process-wide authority table.

### notify_grants_changed

```rust
pub fn notify_grants_changed()
```

Wake the grant watcher in **this** process (same-process save / tests).

Cross-process CLI writes are observed by polling the grants file.

### reconcile_grants_from_disk

```rust
pub fn reconcile_grants_from_disk(files_dir: &Path)
```

Load `plugin-grants.json` and fence sessions that no longer match.

Unreadable / malformed files are left untouched for this tick so a
torn write cannot mass-fence. [`PluginGrantStore::save`](#plugingrantstoresave) writes atomically.

### register_session

```rust
pub fn register_session(plugin_key: &str, revision: &str) -> Arc<AtomicBool>
```

Registers a live session and returns its cancel flag.

Drop the session (or call [`unregister_session`](#unregister_session)) when the vat exits.
Sessions registered this way are still cancelled on grant change, but
have no shutdown hook — prefer [`register_session_with_shutdown`](#register_session_with_shutdown) for
product vats so fencing does not wait for the next RPC.

### register_session_revisions

```rust
pub fn register_session_revisions(
    plugin_key: &str,
    grant_revision: &str,
    revision: &str,
    shutdown: SessionShutdown,
) -> Arc<AtomicBool>
```

Registers a live session with distinct persisted vs effective revisions.

`grant_revision` is the digest of the **stored** operator grant. `revision`
is [`authority_revision`](#authority_revision) of the **effective** runtime grant (after host
overlays). The grant watcher compares grant revisions only.

### register_session_with_shutdown

```rust
pub fn register_session_with_shutdown(
    plugin_key: &str,
    revision: &str,
    shutdown: SessionShutdown,
) -> Arc<AtomicBool>
```

Registers a live session whose shutdown hook runs the instant it is fenced.

Call this **after** the vat work channel exists so `shutdown` can send
`Work::Shutdown` without racing spawn.

### spawn_grant_watcher

```rust
pub fn spawn_grant_watcher(files_dir: PathBuf) -> Arc<AtomicBool>
```

Spawn the daemon grant watcher (fire-and-forget).

#### Returns

Stop flag; set it to end the loop (tests). Production callers leak the task
until process exit.

### unregister_session

```rust
pub fn unregister_session(flag: &Arc<AtomicBool>)
```

Drops a previously registered cancel flag from the live set.

### watch_grants_loop

```rust
pub async fn watch_grants_loop(files_dir: PathBuf, stop: Arc<AtomicBool>)
```

Watch `plugin-grants.json` until the process exits.

In-process [`notify_grants_changed`](#notify_grants_changed) wakes immediately; CLI writes in
another process are picked up within [`GRANT_WATCH_INTERVAL`](#grant_watch_interval).

### install_bundled_platform_plugins

```rust
pub async fn install_bundled_platform_plugins(files_dir: &Path)
```

Installs bundled platform plugins from beside the running executable into `files_dir`.

A no-op when no `plugins/` directory sits beside the executable (development
builds) or this process already tried `files_dir`. The hashing and copying
run on the blocking pool. Failures are logged and do not stop startup:
discovery then reports the missing or untrusted plugin the same way it
would without a bundle.

When something has to be installed the sync takes the files directory's
[`bookclerk_plugin_catalog::PluginMutationLock`](../bookclerk_plugin_catalog/index.md#pluginmutationlock), so the first call for a
files directory must not run while this process holds that lock.

#### Arguments

* `files_dir` - Host `$BOOKCLERK_FILES_DIR` that owns `plugins/` and the install ledger.

### active_processes_for

```rust
pub fn active_processes_for(runtime: GuestRuntimeKind, extra: u32) -> u32
```

Absolute Spec `active_processes` from runtime overhead + extra budget.

### canonical_event_filter

```rust
pub fn canonical_event_filter(filter: &Option<Value>) -> Option<String>
```

Minified JSON for an event filter; `null` / missing → `None`.

### consent_request

```rust
pub fn consent_request(manifest: &PluginManifest, plugin_key: &PluginKey) -> PluginGrant
```

Build the consent request surface from a manifest and its PluginKey.

### consent_request_alias

```rust
pub fn consent_request_alias(manifest: &PluginManifest) -> PluginGrant
```

Build the consent request surface from a manifest (alias only; tests).

### consent_summary

```rust
pub fn consent_summary(grant: &PluginGrant) -> Vec<String>
```

Human-readable consent lines for CLI/UI.

### consumers_cover

```rust
pub fn consumers_cover(
    existing: &BTreeSet<GrantedEventConsumer>,
    requested: &BTreeSet<GrantedEventConsumer>,
) -> bool
```

True when every requested consumer is covered by some approved consumer.

### cores_to_percent

```rust
pub fn cores_to_percent(cores: f64) -> u32
```

Convert cores to Spec/config percent-of-one-core, rounded to 0.01 core.

### database_binding_name

```rust
pub fn database_binding_name(binding: &str) -> Option<&str>
```

The binding name when a grant entry is a named database binding.

### effective_cpu_cores

```rust
pub fn effective_cpu_cores(value: Option<f64>) -> f64
```

Clamp operator/manifest cores into `0.01..=`[`host_cpu_cores_max`](#host_cpu_cores_max).

### effective_cpu_rate_percent

```rust
pub fn effective_cpu_rate_percent(value: Option<u32>) -> u32
```

Resolved jail Spec CPU rate as percent of one logical CPU.

Clamped to `1..=`[`host_cpu_rate_max`](#host_cpu_rate_max) (100 × logical CPUs).

### effective_disk_budget_bytes

```rust
pub fn effective_disk_budget_bytes(grant: Option<&PluginGrant>) -> u64
```

Resolved disk budget in bytes for `data/` and `tmp/` checks.

### effective_disk_mib

```rust
pub fn effective_disk_mib(value: Option<u32>) -> u32
```

Resolved disk budget in MiB (default + host clamp).

### effective_extra_processes

```rust
pub fn effective_extra_processes(value: Option<u32>) -> u32
```

Clamp an operator/manifest extra-process budget (`0..=`[`PLUGIN_JAIL_EXTRA_PROCESSES_MAX`](#plugin_jail_extra_processes_max)).

### effective_grant

```rust
pub fn effective_grant(existing: &PluginGrant, requested: &PluginGrant) -> PluginGrant
```

Spawn/delivery grant: stored approval is authoritative, host-normalized.

Structural capabilities stay at the stored operator snapshot (narrowed).
Network destinations re-evaluate the current manifest plus persisted
operator additions/denials.

### effective_memory_mib

```rust
pub fn effective_memory_mib(value: Option<u32>) -> u32
```

Resolved jail Spec memory ceiling in MiB.

### format_cpu_cores

```rust
pub fn format_cpu_cores(cores: f64) -> String
```

Format cores for operator-facing text (`0.80`).

### grant_covers

```rust
pub fn grant_covers(existing: &PluginGrant, requested: &PluginGrant) -> bool
```

True when a stored grant is usable for enable/spawn of this PluginKey.

Identity match is enough. Extra capabilities on a newer manifest stay
pending until the operator approves them; [`effective_grant`](#effective_grant) intersects.

### grant_has_binding

```rust
pub fn grant_has_binding(grant: &PluginGrant, name: &str) -> bool
```

True when `grant.bindings` contains `name` (case-insensitive).

### grant_revision

```rust
pub fn grant_revision(grant: &PluginGrant) -> String
```

Canonical SHA-256 of a grant's security-relevant fields (not presentation).

### grant_within_ceiling

```rust
pub fn grant_within_ceiling(existing: &PluginGrant, requested: &PluginGrant) -> bool
```

True when an existing grant stays within the manifest request surface.

Used for **platform auto-grants** (installer envelope). Operator approvals
are not limited to this subset — see [`validate_approved_grant`](#validate_approved_grant).

### granted_consumers_from_manifest

```rust
pub fn granted_consumers_from_manifest(
    manifest: &PluginManifest,
) -> BTreeSet<GrantedEventConsumer>
```

Consumers declared by `plugin.toml` as structural grant entries.

### granted_database_bindings

```rust
pub fn granted_database_bindings(grant: &PluginGrant) -> Vec<String>
```

Consented database binding names on a grant, in manifest-set order.

### granted_jobs_from_manifest

```rust
pub fn granted_jobs_from_manifest(manifest: &PluginManifest) -> BTreeSet<String>
```

Job trigger types declared by `[triggers].jobs`.

### granted_queue_consumers

```rust
pub fn granted_queue_consumers(grant: &PluginGrant) -> Vec<String>
```

Consented queue consumer names on a grant.

### host_cpu_cores_max

```rust
pub fn host_cpu_cores_max() -> f64
```

Host max jail CPU in cores (`logical_cpus` as a 2 d.p. float).

### host_cpu_rate_max

```rust
pub fn host_cpu_rate_max() -> u32
```

Host CPU rate ceiling in one-core percent units (`logical_cpus × 100`).

### host_overlay_config_digest

```rust
pub fn host_overlay_config_digest(config: &Config) -> String
```

SHA-256 of host config that participates in implied network overlays.

Changing a Postgres URL or D1 API origin must mint a new executor
`configuration_revision` and fence live sessions. Plugin endpoints (S3,
Audiobookshelf) come from instance documents, not `config.toml`.

### inject_workerd_grant_env

```rust
pub fn inject_workerd_grant_env(cmd: &mut Command, grant: &PluginGrant)
```

Injects effective-grant overrides for `bookclerk-workerd` into a guest command.

Must be called after `env_clear` (explicit `BOOKCLERK_*` is blocked from
inheritance). Domains are always set (possibly empty) so a subset approval
cannot fall back to the full manifest allowlist.

#### Arguments

* `cmd` - Guest process command being prepared for spawn.
* `grant` - Effective covering grant for this plugin.

### is_platform_envelope

```rust
pub fn is_platform_envelope(grant: &PluginGrant) -> bool
```

True when `grant` stays inside the platform installer envelope (deny
network, only `config` / `work_fs` bindings): the request a Bookclerk
platform plugin may receive without the operator being asked.

### jail_process_overhead

```rust
pub fn jail_process_overhead(runtime: GuestRuntimeKind) -> u32
```

Fixed payload occupancy for the launcher / primary guest tree.

Workerd isolate: `bookclerk-workerd` plus the `workerd` child (2).
Native behind workerd: those two plus the native guest (3). Direct native
(diagnostic transport): the guest (1). The Windows outer Job adds the two
jail supervisors on top of the native-behind payload; see
[`WINDOWS_SESSION_PROCESS_BASELINE`](#windows_session_process_baseline).

### network_compatible

```rust
pub fn network_compatible(existing: &str, requested: &str) -> bool
```

Returns true when a stored network mode can satisfy a requested network mode.

#### Arguments

* `existing` - Network mode already approved by the operator.
* `requested` - Network mode requested by the current plugin manifest.

#### Returns

`true` when the modes match, or when an existing `deny` grant is stricter
than a requested `outbound` grant.

### node_logical_cpus

```rust
pub fn node_logical_cpus() -> u32
```

Logical CPUs visible to this process (at least 1).

### overlay_host_implied_network

```rust
pub fn overlay_host_implied_network(
    grant: &mut PluginGrant,
    plugin: &DiscoveredPlugin,
    config: &Config,
    discovered: &[DiscoveredPlugin],
)
```

Overlay TCP + CIDR implied by host configuration (not persisted).

Not operator-invented structural authority. The operator already configured
the destination URL; spawn injects matching `EgressPolicy` so the nested
native guest can `connect()` it. Covers:

- active Postgres `[database.postgres].url`
- active D1 `[database.d1].api_base`
- Audiobookshelf `[integrations.audiobookshelf].base_url`
- S3 `[output.s3].endpoint` (custom/MinIO)

Occupancy is uniquified with [`crate::resolve_plugin_slot`]: a bare alias
overlays only the unique occupant, and a PluginKey never overlays a twin.

No-op unless the covering grant is outbound.

#### Arguments

* `grant` - Effective covering grant (mutated in place).
* `plugin` - Guest being spawned.
* `config` - Host config (URL already env-applied).
* `discovered` - Installs used to uniquify occupancy (must include `plugin`).

### pending_structural

```rust
pub fn pending_structural(
    existing: &PluginGrant,
    requested: &PluginGrant,
) -> PendingStructural
```

Structural capabilities the current manifest requests that the stored grant
does not yet authorize.

### percent_to_cores

```rust
pub fn percent_to_cores(percent: u32) -> f64
```

Convert Spec/config percent-of-one-core (100 = 1.00 core) to cores (2 d.p.).

### queue_consumer_grant

```rust
pub fn queue_consumer_grant(queue: &str) -> String
```

Grant token for consuming `queue`.

### queue_producer_grant

```rust
pub fn queue_producer_grant(binding: &str) -> String
```

Grant token for sending through the `[[queues.producers]]` binding `binding`.

### reconcile_host_overlay_authority

```rust
pub fn reconcile_host_overlay_authority(config: &Config)
```

Recomputes effective overlay authority and fences sessions that no longer match.

Host-config destination changes do not touch `plugin-grants.json`, so the
grant watcher cannot see them. Config reload must call this.

A deployed S3 session is recomputed with the document endpoint recorded at
registration. That path does not read `[output.s3].endpoint`, so a settings
reload cannot cancel an in-flight upload because the TOML endpoint differs
or is unset. Transitional sessions still overlay the TOML endpoint.

A revision mismatch fences that session only. Several vats may share one
PluginKey with different effective revisions, and cancelling the key would
kill a deployed upload when a transitional session's TOML endpoint changes.

### require_binding

```rust
pub fn require_binding(grant: &PluginGrant, name: &str) -> Result<()>
```

Fail closed when a delivery site needs a binding the covering grant lacks.

#### Errors

Returns an error when the operation fails.

### require_grant

```rust
pub fn require_grant(files_dir: &Path, plugin: &DiscoveredPlugin) -> Result<PluginGrant>
```

Require an operator grant before enable **or** every external spawn.

#### Arguments

* `files_dir` - Bookclerk files directory containing `plugin-grants.json`.
* `manifest` - Plugin manifest whose id must have a stored grant.

#### Returns

Effective [`PluginGrant`](#plugingrant) (operator approval, host-normalized).

#### Errors

Returns [`PluginError`](#pluginerror) when no grant exists for the plugin id.

### spawn_config_for_grant

```rust
pub fn spawn_config_for_grant(grant: &PluginGrant, config_table: Value) -> Value
```

Spawn `config` payload: non-empty settings only when the grant includes `config`.

### spawn_grant

```rust
pub fn spawn_grant(files_dir: &Path, plugin: &DiscoveredPlugin) -> Result<PluginGrant>
```

Resolve the covering grant for spawn: verified platform auto-grant, else [`require_grant`](#require_grant).

#### Errors

Returns an error when the operation fails.

### validate_approved_grant

```rust
pub fn validate_approved_grant(
    approved: &PluginGrant,
    baseline: &PluginGrant,
) -> Result<PluginGrant>
```

Validates and normalizes an operator-supplied grant against host hard caps.

The manifest `baseline` supplies identity defaults and suggested values.
Operators may **narrow** structural capabilities (entrypoints, producers,
host bindings, flags) and **widen or narrow** network destinations.
Inventing a structural capability the package did not declare is rejected.
Workerd / disk limits clamp to host maximums.

#### Arguments

* `approved` - Operator-supplied grant draft.
* `baseline` - Consent request generated from the current manifest.

#### Returns

A normalized grant with plugin identity and a fresh `approved_at`.

#### Errors

Returns [`PluginError`](#pluginerror) when identity mismatches, network mode is invalid,
a binding is unknown, or a domain pattern cannot be normalized.

#### Panics

Panics when an internal invariant does not hold.

### validate_described_capabilities

```rust
pub fn validate_described_capabilities(
    manifest: &PluginManifest,
    grant: &PluginGrant,
    described: &PluginCapabilities,
    portal_auth_mode: PortalAuthMode,
) -> Result<()>
```

Reject `describe()` capability claims that exceed the manifest and the
covering grant.

The guest's typed [`PluginCapabilities`](../bookclerk_plugin_abi/generated.md#plugincapabilities) must not widen what
`plugin.toml` declares: no extra entrypoints, event consumers, producers,
job types, database bindings, or named bindings. Every advertised
entrypoint and producer must also be present in the operator grant, and
OAuth storefronts need the `oauth` binding.

#### Arguments

* `manifest` - Installed manifest for the plugin.
* `grant` - Effective covering grant.
* `described` - Capability block the guest returned from `describe()`.
* `portal_auth_mode` - Storefront connect mode from `describe()`.

#### Errors

Returns an error naming the first widening capability or missing grant.

### validate_described_config_options

```rust
pub fn validate_described_config_options(
    manifest: &PluginManifest,
    described: &[ConfigOption],
) -> Result<()>
```

Validates `describe().configOptions` against the manifest `[[settings]]`.

The manifest is the authority for the settings a plugin takes. An empty
list means the guest defers to the manifest. A non-empty list must name
the same keys with the same enum value ids (labels may differ), so a guest
cannot widen or narrow its own writable keys at runtime.

#### Arguments

* `manifest` - Installed manifest for the plugin.
* `described` - `configOptions` the guest returned from `describe()`.

#### Errors

Returns an error naming the first key or value that differs.

### windows_session_active_processes

```rust
pub fn windows_session_active_processes(extra: u32) -> u32
```

Windows outer session Job active-process cap: baseline 5 plus `extra`.

`extra` is clamped to [`PLUGIN_JAIL_EXTRA_PROCESSES_MAX`](#plugin_jail_extra_processes_max). The sum is
clamped to [`PLUGIN_JAIL_ACTIVE_PROCESSES_MAX`](#plugin_jail_active_processes_max). Inner jail caps are not
inflated by this baseline: the gateway stays at 2 and the guest stays at
1 plus the same extra budget.

### search_crates_io

```rust
pub fn search_crates_io(
    query: Option<&str>,
    per_page: u32,
) -> Result<Vec<PluginCatalogEntry>>
```

Search crates.io for Bookclerk plugins.

Uses keyword [`REGISTRY_KEYWORD`](#registry_keyword) plus optional free text, then keeps hits
whose crate name starts with [`CRATE_NAME_PREFIX`](#crate_name_prefix) and parses cleanly.

#### Errors

Returns an error when the operation fails.

### load_authorized_local_packages

```rust
pub fn load_authorized_local_packages(files_dir: &Path) -> Result<AuthorizedLocalPackages>
```

Loads operator-placed archives under `$FILES_DIR/plugin-packages/<name>/`.

Each directory holds `package.json` (a [`BookclerkPackageManifest`](../bookclerk_plugin_catalog/index.md#bookclerkpackagemanifest))
and `archive.tar.gz`. Artifact URLs must be `file:` paths inside that
directory. The reconciler does not download plugins. A missing directory
is an empty map.

#### Errors

Returns an error when `plugin-packages` itself cannot be read. One bad
package is skipped and does not abort the rest of the directory.

### reconcile_local_deployments

```rust
pub async fn reconcile_local_deployments(
    store: &LibraryStore,
    config: &Config,
    node_id: &str,
    packages: &AuthorizedLocalPackages,
    runtime: &dyn DeploymentRuntime,
) -> Result<()>
```

Reconciles every present deployment for `node_id`.

#### Errors

Returns an error when the deployment list cannot be read. A single row's
install, spawn, or observation failure is stored on that row and does not
stop the others.

### build_acquire_destinations

```rust
pub fn build_acquire_destinations(
    config: &Config,
    destinations: &DestinationRegistry,
) -> Result<AcquireDestinations>
```

Build destination backends for acquire from the deployed output plugins.

#### Errors

Returns an error when no destination is deployed on this host or two
instances of one kind are deployed.

### build_storage_backend

```rust
pub fn build_storage_backend(
    destinations: &DestinationRegistry,
) -> Result<Box<dyn StorageBackend>>
```

Build listing/storage backends from the deployed output plugins.

#### Errors

Returns an error when no destination is deployed or two instances of one
kind are deployed.

### discover_plugins

```rust
pub fn discover_plugins(config: &Config) -> Result<Vec<DiscoveredPlugin>>
```

Discover plugins under the configured search directories.

Accepts either:
- `$dir/plugin.toml` (single plugin at root), or
- `$dir/<name>/plugin.toml` (one plugin per subdirectory).

Duplicate [`PluginKey`](../bookclerk_plugin_catalog/index.md#pluginkey) values are a hard error. Duplicate aliases from
different PluginKeys are also a hard error (invalid installation state).

#### Errors

Returns [`PluginError`](#pluginerror) on duplicate keys, duplicate aliases, missing binaries, or I/O failures.

### first_party_database_kind

```rust
pub fn first_party_database_kind(plugin: &DiscoveredPlugin) -> Option<DatabasePluginKind>
```

First-party database adapter kind when provenance + PluginKey match.

`PlatformBundled` (platform sqlite) and `VerifiedStaged` (crates.io /
platform postgres and D1) both qualify. `LocalDevelopment` and `Modified`
never do. Alias text (`id = "sqlite"`) is not consulted.

### identity_matches_occupancy

```rust
pub fn identity_matches_occupancy(plugin_key: &str, alias: &str, spec: &str) -> bool
```

True when `plugin_key` / `alias` is the occupant named by `spec`.

A parseable PluginKey never falls through to an alias comparison, so a
miss cannot inherit another provenance's grant or session. Does not
detect corrupt duplicate aliases — callers that load or privilege-check
must require a unique match ([`resolve_plugin_slot`](#resolve_plugin_slot)).

#### Arguments

* `plugin_key` - Canonical [`PluginKey`](../bookclerk_plugin_catalog/index.md#pluginkey) text.
* `alias` - Manifest display id.
* `spec` - Occupancy selector from [`occupancy_spec`](#occupancy_spec) or a job/CLI id.

### is_first_party_audiobookshelf

```rust
pub fn is_first_party_audiobookshelf(plugin: &DiscoveredPlugin) -> bool
```

True when this install is the verified Bookclerk Audiobookshelf integration.

Only this plugin receives the host-implied network grant for the server
its instance document names (`base_url`). The live manifest alias is not
consulted: an alias twin is a different PluginKey with its own consent.

### is_first_party_local_output

```rust
pub fn is_first_party_local_output(plugin: &DiscoveredPlugin) -> bool
```

True when this install is the verified Bookclerk local-filesystem destination.

### is_first_party_local_output_key

```rust
pub fn is_first_party_local_output_key(key: &PluginKey) -> bool
```

True when `key` names the Bookclerk local-filesystem destination.

This is the key half of [`is_first_party_local_output`]: it does not see
an install, so it cannot check provenance. Use it only where matching
more plugins than the verified one is safe, such as routing acquires to
the nodes that hold their output disks when discovery is unavailable.

#### Arguments

* `key` - Durable plugin identity to classify.

#### Returns

`true` for the first-party local destination package on a first-party
provenance scheme.

### is_first_party_s3_output

```rust
pub fn is_first_party_s3_output(plugin: &DiscoveredPlugin) -> bool
```

True when this install is the verified Bookclerk S3 destination.

### occupancy_matches_alias

```rust
pub fn occupancy_matches_alias(spec: &str, alias: &str) -> bool
```

True when occupancy `spec` names the display alias (or kind token) `alias`.

Bare kind tokens (`pg` ↔ `postgres`) match without discovery. A PluginKey
occupant matches only when it is a host-controlled first-party package
whose **package name** maps to that alias — never via a `#fragment`.
Third-party keys need [`occupancy_names_alias`](#occupancy_names_alias) plus discovery.

A bare alias does **not** match a PluginKey string passed as `alias`.

#### Arguments

* `spec` - Occupancy selector from config (`plugin = "…"`).
* `alias` - Display alias or first-party kind token being compared.

### occupancy_names_alias

```rust
pub fn occupancy_names_alias(
    spec: &str,
    alias: &str,
    discovered: &[DiscoveredPlugin],
) -> bool
```

True when occupancy `spec` names `alias` among `discovered` installs.

Uses [`occupancy_matches_alias`](#occupancy_matches_alias) first, then resolves `spec` to exactly one
PluginKey. Corrupt duplicate aliases fail closed (`false`).

### occupancy_spec

```rust
pub fn occupancy_spec<'a>(plugin_field: &'a str, alias: &'a str) -> &'a str
```

Config occupancy selector (`plugin = "…"`), or `alias` when the field is empty.

Occupancy is how host config names the guest that may use a singleton
slot (`[database].plugin`, `[output.s3].plugin`, `[sources.<id>].plugin`).
An empty field still means the display alias, which
[`resolve_plugin_slot`](#resolve_plugin_slot) accepts only when exactly one install uses it.

#### Arguments

* `plugin_field` - Occupancy string from config (`plugin = "…"`).
* `alias` - Display alias used when `plugin_field` is empty.

### plugin_matches_occupancy

```rust
pub fn plugin_matches_occupancy(plugin: &DiscoveredPlugin, spec: &str) -> bool
```

True when `plugin` is the occupant named by `spec` (PluginKey or alias).

Does not detect corrupt duplicate aliases — loaders must call [`resolve_plugin_slot`](#resolve_plugin_slot).

#### Arguments

* `plugin` - Candidate install.
* `spec` - Occupancy selector from [`occupancy_spec`](#occupancy_spec).

### plugin_search_dirs

```rust
pub fn plugin_search_dirs(config: &Config) -> Vec<PathBuf>
```

Resolve search roots: `BOOKCLERK_PLUGIN_DIRS` then `$FILES_DIR/plugins`.

### resolve_plugin_ref

```rust
pub fn resolve_plugin_ref<'a>(
    plugins: &'a [DiscoveredPlugin],
    spec: &str,
) -> Result<&'a DiscoveredPlugin>
```

Resolves `spec` to a discovered plugin.

`spec` may be a canonical [`PluginKey`](../bookclerk_plugin_catalog/index.md#pluginkey) or an alias unique in this host
plugin namespace. In valid installed state an alias maps to exactly one
PluginKey. Duplicate aliases fail closed rather than picking first or last.

#### Errors

Returns an error when `spec` matches nothing, or when corrupt disk state
contains duplicate keys or aliases.

### resolve_plugin_slot

```rust
pub fn resolve_plugin_slot<'a>(
    plugins: &'a [DiscoveredPlugin],
    spec: &str,
) -> Result<Option<&'a DiscoveredPlugin>>
```

Resolves `spec` among `plugins` without treating a vacant slot as an error.

`None` means nothing installed matches. An ambiguous alias is an error
so callers fail closed instead of spawning every twin or last-write-wins.

#### Arguments

* `plugins` - Already-filtered family candidates.
* `spec` - Occupancy selector from [`occupancy_spec`](#occupancy_spec).

#### Errors

Returns [`PluginError`](#pluginerror) when two installs share the alias (or PluginKey).

### settings_table

```rust
pub fn settings_table(config: &Config, plugin: &DiscoveredPlugin) -> Table
```

Bootstrap connection settings for a database adapter from `config.toml`.

Database adapters are the one plugin family whose configuration stays in
the bootstrap file: the process needs it to open the library before the
database-backed control plane exists. Every other family receives its
settings from the plugin instance document.

### settings_table_for

```rust
pub fn settings_table_for(
    config: &Config,
    plugin: &DiscoveredPlugin,
    family: PluginFamily,
) -> Table
```

[`settings_table`](#settings_table) for one explicit handler `family`.

Only `PluginFamily::Database` has a
`config.toml` table; every other family returns an empty table.

### stamp_occupancy_plugin_key

```rust
pub fn stamp_occupancy_plugin_key(
    config: &mut Config,
    plugin: &DiscoveredPlugin,
) -> Result<()>
```

Records `plugin` as the `[database].plugin` occupant.

Database adapters are the bootstrap exception: they stay selected in
`config.toml` because the connection is needed before the database can be
read. Every other plugin is enabled by a `plugin_deployments` row.

#### Errors

Returns an error when `plugin` is not a database adapter.

### upgrade_unique_alias_occupancy

```rust
pub fn upgrade_unique_alias_occupancy(
    config: &mut Config,
    discovered: &[DiscoveredPlugin],
)
```

Rewrites a unique `[database].plugin` alias to the canonical PluginKey.

An ambiguous alias or vacant slot is left unchanged so loaders still fail
closed. An empty `discovered` slice is a no-op so a settings discovery
timeout cannot wipe occupancy.

### enroll_local_plugins

```rust
pub async fn enroll_local_plugins(
    store: &LibraryStore,
    config: &Config,
    node_id: &str,
) -> Result<Vec<EnrolledPlugin>>
```

Enrolls every discovered plugin on `node_id`.

A plugin that fails to enroll is logged and skipped so one bad install
does not keep the others from starting.

#### Errors

Returns an error when discovery or the grant store cannot be read.

### enroll_one

```rust
pub async fn enroll_one(
    store: &LibraryStore,
    node_id: &str,
    plugin: &DiscoveredPlugin,
    grants: &PluginGrantStore,
) -> Result<EnrolledPlugin>
```

Enrolls one discovered plugin. See the module docs for the steps.

#### Errors

Returns an error when a schema, instance, document, or deployment write fails.

### has_covering_grant

```rust
pub fn has_covering_grant(plugin: &DiscoveredPlugin, grants: &PluginGrantStore) -> bool
```

True when the operator already consented to this plugin, or it is a
verified platform guest that grants itself on first spawn.

### is_bootstrap_only

```rust
pub fn is_bootstrap_only(plugin: &DiscoveredPlugin) -> bool
```

True when the plugin is a database adapter and nothing else: those stay
bootstrap (`[database]` in `config.toml`) in this phase.

### backup_adapter_id

```rust
pub fn backup_adapter_id(plugin_id: &str) -> String
```

Adapter id recorded on a backup (diagnostic; restore is capability-driven).

### database_connect_bindings

```rust
pub fn database_connect_bindings(
    config: &Config,
    plugin: &DiscoveredPlugin,
    session: &PluginSession,
) -> Result<BindingValues>
```

Builds the [`bookclerk_plugin_sdk::BindingValues`](../bookclerk_plugin_abi/index.md#bindingvalues) for the adapter's
`PluginWorker.open` (then `databaseAdapter.openSession`).

Used by the CLI diagnose probe and mirrors [`ExternalDatabase::connect`](#externaldatabaseconnect).

#### Errors

Returns an error when plugin data paths, secrets, or context encoding fail.

### export_registered_plugin_units

```rust
pub async fn export_registered_plugin_units(
    ext: &ExternalDatabase,
    config: &Config,
    library: &DatabaseConnection,
) -> Result<Vec<PreparedPluginUnit>>
```

Exports every registered plugin-owned database binding as SeaORM sessions.

A missing `plugin_databases` table (uninitialized library) is treated as an
empty registry. Fails closed if any registered binding cannot be opened.

#### Errors

Returns when a binding is inaccessible.

### load_external_database

```rust
pub async fn load_external_database(config: &Config) -> Result<DatabaseRegistry>
```

Discover and spawn the external database plugin matching `[database].plugin`.

Platform plugins bundled beside the host executable are installed into the
files directory first ([`crate::install_bundled_platform_plugins`](#install_bundled_platform_plugins)).

Guests are required: when the matching database plugin is missing or fails to
start, [`open_library_store`](#open_library_store) returns an error (no in-process engine).

#### Errors

Returns an error when the operation fails.

### migrate_database_plugin

```rust
pub async fn migrate_database_plugin(
    config: &Config,
    from_plugin: &str,
    to_plugin: &str,
    opts: &BackendMigrateOptions,
) -> Result<BackendMigrateSummary>
```

Copy library data from one database plugin backend to another.

#### Errors

Returns an error when the operation fails.

### migrate_library_schema

```rust
pub async fn migrate_library_schema(
    config: &Config,
    target: i64,
    opts: SchemaApplyOptions,
) -> Result<SchemaWalk>
```

Opens the active database guest without auto-apply and migrates to `target`.

Uses guest `executeAtomic` batches (the same path as daemon connect).

#### Errors

Returns when the guest cannot start, connect, or apply schema.

### open_library_store

```rust
pub async fn open_library_store(
    config: &Config,
    registry: &DatabaseRegistry,
) -> Result<LibraryStore>
```

Open [`bookclerk_library::LibraryStore`](../bookclerk_library/index.md#librarystore) via the external database guest (required).

#### Errors

Returns an error when the operation fails.

### open_library_store_for_plugin

```rust
pub async fn open_library_store_for_plugin(
    config: &Config,
    plugin_id: &str,
) -> Result<LibraryStore>
```

Open the library for a specific `[database].plugin` id (ignoring the active config value).

#### Errors

Returns an error when the operation fails.

### restore_plugin_backup_units

```rust
pub async fn restore_plugin_backup_units(
    ext: &ExternalDatabase,
    config: &Config,
    library: &DatabaseConnection,
    repo: &BackupRepository,
    units: &[BackupUnit],
) -> Result<()>
```

Restores plugin binding units onto the **current** adapter.

Each unit is replaced independently. A later failure leaves earlier units
already replaced (no cross-database atomicity). Plugin migrations are not
run; version rows restore as captured data. Registry rows are rebound to
the target adapter's physical placement.

Each `open_binding_seaorm` session must itself advertise `atomicUnitRestore`.
Restore binds, payload, and request limits come from that session.

#### Errors

Returns when provisioning or canonical restore fails. Partial completion is
reported in the error message.

### allow_credential_login

```rust
pub fn allow_credential_login(granted_config: &Value) -> bool
```

`allow_credential_login` from the resolved `CONFIG`; absent means allowed.

### enroll_node

```rust
pub async fn enroll_node(config: &Config, store: &LibraryStore) -> Result<NodeEnrollment>
```

Bootstraps the control plane on an open library and enrolls every local
plugin, without loading any guest.

The daemon's config reload uses this: guests stay with the deployment
reconciler, which picks up the enrolled deployments on its next pass.

#### Errors

Returns an error when the secret root or node identity cannot be
established, or enrollment fails as a whole.

### open_host_control_plane

```rust
pub async fn open_host_control_plane(config: &Config) -> Result<HostControlPlane>
```

Opens the library database and the control plane for `config`.

#### Errors

Returns an error when the database cannot be opened, the secret root or
node identity cannot be established, or enrollment fails as a whole.

### open_host_control_plane_with_store

```rust
pub async fn open_host_control_plane_with_store(
    config: &Config,
    database_registry: DatabaseRegistry,
    store: LibraryStore,
) -> Result<HostControlPlane>
```

[`open_host_control_plane`](#open_host_control_plane) on a library that is already open.

The daemon's config reload uses this so it does not reopen the database.

#### Errors

Returns an error when bootstrap or enrollment fails.

### refresh_present

```rust
pub async fn refresh_present(
    store: &LibraryStore,
    node_id: &str,
    enrolled: &[EnrolledPlugin],
) -> Result<(Vec<PresentDeployment>, HashMap<String, String>)>
```

Loads an already-open host control plane's deployments again, for callers
that changed desired state and want a fresh view without re-enrolling.

#### Errors

Returns an error when the deployment table cannot be read.

### add_package_version

```rust
pub async fn add_package_version(
    store: &LibraryStore,
    actor: &ConfigActor,
    files_dir: &Path,
    node_id: &str,
    request: VersionRequest,
) -> Result<AddedVersion>
```

Verifies `package`, records a [`PluginVersion`](../bookclerk_library/control_plane.md#pluginversion), and moves the active
pointer per [`VersionRequest::activate`](#versionrequest).

A local archive's digest is computed here (it is the operator's own
file); a remote artifact must match the digest its package manifest
declares. Archives up to the library blob cap are stored in the
database when no shared URL exists, so every host can materialize them.

#### Errors

Returns an error for a platform key (platform guests ship with the
host), a malformed or yanked package, a digest mismatch, a tree that
disagrees with the package, or a library write failure.

### enroll_version

```rust
pub async fn enroll_version(
    store: &LibraryStore,
    actor: &ConfigActor,
    version: &PluginVersion,
) -> Result<PluginInstance>
```

Creates the default instance of a installed plugin with its schema
defaults, as enrollment does for a host-local plugin. Idempotent.

#### Errors

Returns an error when an instance or document write fails.

### grant_covers_version

```rust
pub fn grant_covers_version(grant: &PluginGrant, version: &PluginVersion) -> bool
```

True when `grant` still covers everything `version` requests, so an
update may carry an earlier approval forward. A widened request needs a
fresh `plugins approve`.

### host_artifact

```rust
pub fn host_artifact(version: &PluginVersion) -> Result<&PluginArtifact>
```

The version artifact this node runs.

#### Errors

Returns an error naming the host target when the version has none for
it, so placement can exclude this host.

### install_plugin_key

```rust
pub async fn install_plugin_key(
    store: &LibraryStore,
    files_dir: &Path,
    coordinate: &PackageCoordinate,
    package: &BookclerkPackageManifest,
) -> Result<String>
```

The plugin key an install installs under.

A registry coordinate names a stable key. A local archive's path does
not: a new version of an already installed plugin arrives as another
file, so its key is the key its alias is already installed under, and
only a first install takes the path-derived key.

#### Errors

Returns an error when the coordinate yields no key or the alias is
installed under several keys.

### install_version

```rust
pub async fn install_version(
    store: &LibraryStore,
    actor: &ConfigActor,
    files_dir: &Path,
    node_id: &str,
    request: InstallRequest,
) -> Result<InstalledVersion>
```

Installs an operator's package as a version, carries an earlier
approval forward when it still covers the new version (or approves when
asked), and enrolls the plugin's default instance.

One alias names one plugin: a version may not take an alias another
plugin's active version already answers to.

#### Errors

Returns an error when the key cannot be derived, the alias belongs to
another plugin, or installing, approval, or enrollment fails.

### is_sha256_hex

```rust
pub fn is_sha256_hex(raw: &str) -> bool
```

True for a canonical SHA-256 digest: exactly 64 lowercase hex digits.

Every digest that names a path (the version cache) or is compared
against a computed digest is held to this form, so an owner- or
registry-supplied value such as `..`, an absolute path, or one with a
separator can never select a directory outside the cache.

### materialize_owner_version

```rust
pub async fn materialize_owner_version(
    store: &LibraryStore,
    files_dir: &Path,
    version: &PluginVersion,
) -> Result<PathBuf>
```

[`materialize_version`](#materialize_version) for a version an owner cluster deploys to this
host: a `blob:` archive is read from the library as usual (the caller
fetched it over the owner link and checked its digest), and any other
URL is fetched only as an owner `https:` URL
([`bookclerk_plugin_catalog::fetch_owner_artifact`]: no `file:`, no
local or private hosts, every redirect hop checked the same way).

A tree already cached under the artifact's payload digest is reused
without a fetch or an archive check, so the caller must have checked
the whole artifact (URL, archive digest and payload digest) against the
host operator's approval first
(`RemotePluginConsent::approved_artifact_matches`).

#### Arguments

* `store` - This host's library.
* `files_dir` - This host's files dir.
* `version` - The owner's version, with the grant this host runs it with.

#### Errors

As [`materialize_version`](#materialize_version), and when the owner's URL is refused.

### materialize_version

```rust
pub async fn materialize_version(
    store: &LibraryStore,
    files_dir: &Path,
    version: &PluginVersion,
) -> Result<PathBuf>
```

Makes the active version's tree available under
`plugins/.versions/<payload digest>` and returns that path.

A cached digest is reused as is; otherwise the archive is fetched (from
the library blob store, a `file:` path, or HTTPS), its digest checked,
extracted under `.staging`, the payload-root digest checked against the
version, and renamed into place under the plugin mutation lock.

#### Errors

Returns an error when the version has no artifact for this host, a
digest is not 64 lowercase hex digits ([`is_sha256_hex`](#is_sha256_hex); checked before
any path is formed or a cached tree reused), the artifact cannot be
fetched, or a digest does not match.

### open_installed_plugin

```rust
pub fn open_installed_plugin(
    root: &Path,
    version: &PluginVersion,
) -> Result<DiscoveredPlugin>
```

Opens a materialized version tree as a spawnable plugin carrying the
version's identity and consent grant.

#### Errors

Returns an error when the tree's manifest is unreadable or names another
plugin, or the version has no approved grant yet.

### sweep_version_cache

```rust
pub fn sweep_version_cache(
    files_dir: &Path,
    keep: &BTreeSet<String>,
    grace: Duration,
) -> Result<usize>
```

Removes cached digests not in `keep` and older than `grace`, plus stale
version staging directories. Returns how many entries were removed.

#### Errors

Returns an error when the mutation lock cannot be taken.

### version_cache_path

```rust
pub fn version_cache_path(files_dir: &Path, payload_root_sha256: &str) -> Result<PathBuf>
```

Cached tree of one payload-root digest.

#### Errors

Returns an error when `payload_root_sha256` is not a canonical SHA-256
([`is_sha256_hex`](#is_sha256_hex)): it is joined onto the cache root, so anything else
could name a directory outside it.

### version_cache_root

```rust
pub fn version_cache_root(files_dir: &Path) -> PathBuf
```

Cache root: `<files_dir>/plugins/.versions`.

### version_consent_request

```rust
pub fn version_consent_request(version: &PluginVersion) -> Result<PluginGrant>
```

Consent the version's manifest requests (what `plugins approve` shows).

#### Errors

Returns an error when the recorded `plugin.toml` or plugin key no longer
parses.

### version_grant

```rust
pub fn version_grant(version: &PluginVersion) -> Result<PluginGrant>
```

The consent grant recorded on `version`.

#### Errors

Returns an error when no operator has approved the version yet or the
recorded grant is malformed.

### configured_alias_occupants

```rust
pub fn configured_alias_occupants(
    config: &Config,
) -> Result<Vec<(PluginKey, String, PathBuf)>>
```

Occupancy (PluginKey, alias, path) under every configured discovery root.

Walks [`plugin_search_dirs`](#plugin_search_dirs) without hashing payloads. A receipt PluginKey
is preferred; otherwise `PluginKey::from_install_path`.

#### Errors

Returns when a `plugin.toml` cannot be read or parsed, or a PluginKey
cannot be formed for an occupant.

### install_from_manifest_with_configured_aliases

```rust
pub fn install_from_manifest_with_configured_aliases(
    config: &Config,
    lock: &PluginMutationLock,
    manifest: &BookclerkPackageManifest,
    coordinate: &PackageCoordinate,
    opts: &InstallOptions,
) -> Result<InstallOutcome>
```

Catalog install after [`reject_configured_alias_collision`](#reject_configured_alias_collision).

`lock` must already cover `opts.plugins_root` so alias preflight and the
catalog transaction share one host-local mutation lock.

#### Errors

Returns a catalog error when the lock does not cover this plugins root,
preflight rejects the alias, or install fails.

### install_local_archive_with_configured_aliases

```rust
pub fn install_local_archive_with_configured_aliases(
    config: &Config,
    lock: &PluginMutationLock,
    archive: &Path,
    manifest: &BookclerkPackageManifest,
    opts: &InstallOptions,
) -> Result<InstallOutcome>
```

Local-archive install after [`reject_configured_alias_collision`](#reject_configured_alias_collision).

`lock` must already cover `opts.plugins_root`.

#### Errors

Returns a catalog error when the lock does not cover this plugins root,
preflight rejects the alias, or install fails.

### reject_configured_alias_collision

```rust
pub fn reject_configured_alias_collision(
    config: &Config,
    incoming_key: &PluginKey,
    incoming_alias: &str,
) -> Result<()>
```

Rejects installing `incoming_alias` when a different PluginKey already owns
it anywhere in the configured discovery universe.

Same PluginKey remains eligible for update/replace at the **same** alias.
`--replace` cannot seize a foreign alias or rename an installed alias.

#### Errors

Returns [`PluginError`](#pluginerror) when the alias is occupied by another PluginKey or
occupancy cannot be read.

### binding_grant_flags

```rust
pub fn binding_grant_flags(grant: &PluginGrant) -> InstanceBindingGrant
```

`config` / `secrets` flags of the plugin's covering grant.

### prepare_open_for_instance

```rust
pub async fn prepare_open_for_instance(
    store: &LibraryStore,
    files_dir: &Path,
    plugin: &DiscoveredPlugin,
    instance_id: &PluginInstanceId,
) -> Result<PreparedOpen>
```

Resolves open bindings for `instance_id` of `plugin`.

The covering grant comes from `plugin-grants.json`; the settings schema
from the installed manifest. A missing document, a dangling secret ref, or
a missing required setting is an error. Document keys the installed
manifest no longer declares are dropped and logged.

#### Errors

Returns an error when the grant cannot be loaded, the instance is not this
plugin's, or its document cannot be resolved.

### prepared_open_from_resolved

```rust
pub fn prepared_open_from_resolved(
    config: ExtensibleConfig,
    secrets: ExtensibleConfig,
    config_revision: i64,
) -> PreparedOpen
```

Builds open bindings from a deployment's already resolved payloads.

Does not look up an instance by plugin key.

### guest_reservation

```rust
pub fn guest_reservation(
    manifest: &PluginManifest,
    grant: Option<&PluginGrant>,
    jail: &PluginsJailConfig,
) -> GuestReservation
```

CPU and memory one guest may use at most on this host: its grant's
ceilings after `[plugins.jail]`, the numbers its jail enforces. The
cluster scheduler accounts host budgets against these.

### plugin_data_dir

```rust
pub fn plugin_data_dir(config: &Config, plugin: &DiscoveredPlugin) -> Result<PathBuf>
```

The private directory a plugin keeps state in.

State is keyed by [`bookclerk_plugin_catalog::PluginKey`](../bookclerk_plugin_catalog/index.md#pluginkey) (`plugin-state/<fs_id>/data`), not the
display alias. Invalid keys cannot be constructed.

#### Errors

Returns an error when the operation fails.

### family_keyword

```rust
pub fn family_keyword(family: PluginFamily) -> &'static str
```

Family-specific crates.io keyword (`bookclerk-source`, …).

### host_target_triple

```rust
pub fn host_target_triple() -> &'static str
```

Host target triple used when selecting release assets.

### validate_plugin_id

```rust
pub fn validate_plugin_id(id: &str) -> Result<()>
```

Validate plugin id segment used in crate names and `plugin.toml`.

Delegates to [`bookclerk_plugin_manifest::validate_plugin_id`](../bookclerk_plugin_manifest/id.md#validate_plugin_id) (strict
`[a-z0-9_]{2,32}` grammar; unique across kinds in one host plugin namespace).

#### Errors

Returns an error when the operation fails.

### approve_remote_request

```rust
pub async fn approve_remote_request(
    store: &LibraryStore,
    cluster_id: &str,
    plugin_key: &str,
    reviewed_digest: &str,
    narrowing: &RemoteNarrowing,
    approver: RemoteApprover<'_>,
) -> Result<RemotePluginConsent, RemoteApprovalError>
```

Approves the request recorded from `cluster_id` for `plugin_key`,
exactly as it was reviewed: `reviewed_digest` must be the recorded
request's digest
(`bookclerk_library::control_plane::RemotePluginConsent::request_digest`,
shown with the request), and the consent is that request minus
`narrowing` ([`remote_consent`](#remote_consent)). The write is a compare-and-swap on the
recorded request, so a request that changed after it was read is
refused rather than approved unseen.

#### Arguments

* `store` - This host's library.
* `cluster_id` - Owner cluster id.
* `plugin_key` - Canonical plugin key.
* `reviewed_digest` - Digest of the request as the approver reviewed it.
* `narrowing` - What the approver takes out of the request.
* `approver` - Who approves (an automatic approval never overrides an
  operator's revoke).

#### Errors

Returns [`RemoteApprovalError::NotRequested`](#remoteapprovalerror) when nothing is recorded,
[`RemoteApprovalError::Conflict`](#remoteapprovalerror) when the recorded request is not the
reviewed one (its digest differs) or the store refuses the write (a
request that changed in between, or an automatic approval of a revoked
one), [`RemoteApprovalError::Invalid`](#remoteapprovalerror) when it does not parse or the
narrowing names something it does not ask for, and
[`RemoteApprovalError::Store`](#remoteapprovalerror) when a read or write fails.

### narrow_remote_grant

```rust
pub fn narrow_remote_grant(requested: &PluginGrant, consent: &PluginGrant) -> PluginGrant
```

The grant a guest runs with: `requested ∩ consent`.

Every structural set and network destination is intersected; network
is outbound only when both are; undeclared redirects only when both
allow; budgets take the smaller. Destinations the request asks for but
the result lacks are recorded as operator denials, so the spawn path's
manifest re-evaluation ([`crate::effective_grant`](#effective_grant)) keeps them out.

### remote_consent

```rust
pub fn remote_consent(
    requested: &PluginGrant,
    narrowing: &RemoteNarrowing,
) -> Result<PluginGrant>
```

The consent an operator gives to `requested` with `narrowing` applied.

#### Errors

Returns an error when the narrowing names a domain or binding the
request does not ask for (it would deny nothing, and a typo must not
read as a narrowing that did not happen).

### remote_consent_state

```rust
pub fn remote_consent_state(
    plugin_key: &str,
    requested_json: &str,
    reviewed_json: Option<&str>,
    consent_json: Option<&str>,
) -> RemoteConsentState
```

Evaluates a stored consent row's grants: the current request
(`requested_json`), the request as reviewed and the consent (both
`None` while unapproved).

### remote_consent_view

```rust
pub fn remote_consent_view(row: &RemotePluginConsent) -> Value
```

The one JSON view of a consent row, shared by the operator API and the
CLI so both answer the same fields: the request (version, grant, the
archive this host would fetch, and its `request_digest`), the approval
(`approved_version`, `grant`, `approved_at`, `approved_by`), `revoked_at`,
`updated_at`, and the `state` ([`remote_row_consent_state`](#remote_row_consent_state)). A stored
grant that does not parse shows as `null`.

### remote_platform_auto_approval

```rust
pub fn remote_platform_auto_approval(
    requested: &PluginGrant,
    owner_payload_root_sha256: &str,
    local_bundled_payload: Option<&str>,
) -> bool
```

True when a platform plugin's request may be approved without asking:
the key is a Bookclerk platform key, this host's own install of that key
is platform-bundled with the same payload digest as the owner's
artifact (`local_bundled_payload`, from this host's provenance check),
and the request stays inside the platform envelope (deny network, only
`config` / `work_fs`).

### remote_request_within

```rust
pub fn remote_request_within(reviewed: &PluginGrant, requested: &PluginGrant) -> bool
```

True when `requested` asks for nothing outside `reviewed`: the same
plugin, every capability set a subset, network no wider (outbound only
if reviewed outbound), undeclared redirects only if reviewed, and no
larger budget.

### remote_row_consent_state

```rust
pub fn remote_row_consent_state(row: &RemotePluginConsent) -> RemoteConsentState
```

Evaluates a stored consent row: [`remote_consent_state`](#remote_consent_state) over its
grants, except that an approval of another archive (the owner changed
the artifact URL or digest since the operator approved) needs approval
again, whatever the grant.

#### Arguments

* `row` - The consent row as stored.

### plugin_instance_key

```rust
pub fn plugin_instance_key(plugin_id: &str, account_id: &str) -> String
```

Isolation key: different accounts never share a plugin isolate.

### note_spawn_stage

```rust
pub fn note_spawn_stage(message: &str)
```

Record one secret-free startup line.

The line is a `bookclerk::spawn` tracing event and a byte-capped ring
entry. Raw stderr is written only when [`spawn_diag_stderr_enabled`](../bookclerk_sandbox/index.md#spawn_diag_stderr_enabled) is
set (`BOOKCLERK_SPAWN_DIAG=1|true|stderr`). Session challenges are stripped.
Plugin stderr is not stored here; the per-session tail is separate.

### recent_spawn_diagnostics

```rust
pub fn recent_spawn_diagnostics() -> String
```

Bounded startup log for the current process.

A deadline handler prints this before exiting. The snapshot is a second
capped copy of the ring, not a join of unbounded strings.

### retry_abandoned_acl_journals

```rust
pub fn retry_abandoned_acl_journals(files_dir: &Path)
```

Retry abandoned journals for every plugin under `files_dir/plugin-state`.

Host startup calls this so a crash does not wait for the next spawn of the
same plugin. A live session's owner lock is left alone.

### revoke_plugin_state_before_purge

```rust
pub fn revoke_plugin_state_before_purge(
    state_root: &Path,
    discard_rejected: bool,
) -> Result<(), String>
```

Revoke this plugin's journal before `--purge-state` deletes the directory.

`discard_rejected` deletes `session-*.json.rejected` with plugin state.
A readable journal that is still present is never discarded that way.

#### Errors

Returns an error when a journal file is still present, when a live session
holds one, or when a rejected record remains and `discard_rejected` is
false. Purge then leaves plugin-state in place.

## Type aliases

### SessionShutdown

```rust
pub type SessionShutdown = Arc<dyn Fn() + Send + Sync>;
```

Shutdown hook invoked the moment a live session is fenced.

### Result

```rust
pub type Result<T> = Result<T, PluginError>;
```

Result alias for [`PluginError`](#pluginerror).

### GuestDatabaseFactory

```rust
pub type GuestDatabaseFactory = Arc<dyn Fn(Arc<AtomicBool>, u64) -> Arc<dyn GuestDatabase> + Send + Sync>;
```

Send-safe constructor for a per-binding [`bookclerk_plugin_sdk::GuestDatabase`](../bookclerk_plugin_abi/index.md#guestdatabase).

`GuestDatabase` trait objects are `?Send`, so named plugin database
bindings cross into the vat task as factories and are constructed on the
vat thread just before the per-job `PluginWorker.open`. The factory
receives the job cancel flag and the host lease deadline so binding
execute can abort and cap `deadlineUnixMs`.

## Constants

### GRANT_WATCH_INTERVAL

```rust
pub const GRANT_WATCH_INTERVAL: Duration = _;
```

How often the daemon re-reads `plugin-grants.json` when no in-process notify fires.

### GRANTS_FILE

```rust
pub const GRANTS_FILE: &str = "plugin-grants.json";
```

Filename under `$BOOKCLERK_FILES_DIR` for persisted grants.

### GRANT_SCHEMA_VERSION

```rust
pub const GRANT_SCHEMA_VERSION: u32 = 2;
```

Current persisted [`PluginGrant`](#plugingrant) document version.

This constant versions the on-disk grant representation. Missing / `0` JSON
is a legacy document. Empty structural sets are fail-closed at **every**
schema version: they approve nothing and do not acquire capabilities merely
because a newer manifest requests them.

### KNOWN_HOST_BINDINGS

```rust
pub const KNOWN_HOST_BINDINGS: &[&str] = _;
```

Host binding names operators may grant (widen or narrow).

### PLUGIN_JAIL_ACTIVE_PROCESSES_MAX

```rust
pub const PLUGIN_JAIL_ACTIVE_PROCESSES_MAX: u32 = 64;
```

Absolute OS jail PID ceiling (`overhead + extra`, inclusive).

### PLUGIN_JAIL_CPU_CORES_DEFAULT

```rust
pub const PLUGIN_JAIL_CPU_CORES_DEFAULT: f64 = 0.80;
```

Default jail CPU as cores (`PLUGIN_JAIL_CPU_RATE_DEFAULT` / 100).

### PLUGIN_JAIL_CPU_RATE_DEFAULT

```rust
pub const PLUGIN_JAIL_CPU_RATE_DEFAULT: u32 = 80;
```

Default jail Spec CPU rate percent for confined guests (one-core units).

### PLUGIN_JAIL_CPU_RATE_MAX

```rust
pub const PLUGIN_JAIL_CPU_RATE_MAX: u32 = 100;
```

Absolute ceiling used only when host CPU detection is unavailable in docs.

Prefer [`host_cpu_rate_max`](#host_cpu_rate_max) at runtime (`logical_cpus × 100`).

### PLUGIN_JAIL_EXTRA_PROCESSES_DEFAULT

```rust
pub const PLUGIN_JAIL_EXTRA_PROCESSES_DEFAULT: u32 = 2;
```

Default **extra** processes/threads beyond launcher overhead.

### PLUGIN_JAIL_EXTRA_PROCESSES_MAX

```rust
pub const PLUGIN_JAIL_EXTRA_PROCESSES_MAX: u32 = 62;
```

Host hard cap for per-plugin extra process budget.

### PLUGIN_JAIL_MEMORY_MIB_DEFAULT

```rust
pub const PLUGIN_JAIL_MEMORY_MIB_DEFAULT: u32 = 512;
```

Default jail Spec memory ceiling (MiB) for confined guests.

### PLUGIN_JAIL_MEMORY_MIB_MAX

```rust
pub const PLUGIN_JAIL_MEMORY_MIB_MAX: u32 = 4096;
```

Host hard cap for per-plugin jail memory overrides (MiB).

### PLUGIN_STATE_BUDGET_MIB_DEFAULT

```rust
pub const PLUGIN_STATE_BUDGET_MIB_DEFAULT: u32 = 512;
```

Default per-plugin `data/` and `tmp/` disk budget (MiB each).

### PLUGIN_STATE_BUDGET_MIB_MAX

```rust
pub const PLUGIN_STATE_BUDGET_MIB_MAX: u32 = 4096;
```

Host hard cap for per-plugin disk budget overrides (MiB).

### QUEUE_CONSUMER_BINDING_PREFIX

```rust
pub const QUEUE_CONSUMER_BINDING_PREFIX: &str = "queue_consumer:";
```

Grant `bindings` entry prefix for one `[[queues.consumers]]` row
(`queue_consumer:<queue>`): the host may deliver that queue to the plugin.

### QUEUE_PRODUCER_BINDING_PREFIX

```rust
pub const QUEUE_PRODUCER_BINDING_PREFIX: &str = "queue:";
```

Grant `bindings` entry prefix for one `[[queues.producers]]` binding
(`queue:<BINDING>`): the plugin may send through that `env` binding.

### WINDOWS_SESSION_PROCESS_BASELINE

```rust
pub const WINDOWS_SESSION_PROCESS_BASELINE: u32 = 5;
```

Windows outer session Job processes before the granted extra budget.

Both `bookclerk-jail` supervisors, `bookclerk-workerd`, the pinned
`workerd`, and the native guest. Payload overhead stays 3
([`GuestRuntimeKind::process_overhead`](#guestruntimekindprocess_overhead)); this baseline is only the outer
Windows Job. Linux `pids.max` counts threads and must not use this number.

### WORKERD_GRANT_CPU_MS_ENV

```rust
pub const WORKERD_GRANT_CPU_MS_ENV: &str = "BOOKCLERK_WORKERD_GRANT_CPU_MS";
```

Grant CPU budget (ms) for workerd logging / limit narrowing.

### WORKERD_GRANT_DOMAINS_ENV

```rust
pub const WORKERD_GRANT_DOMAINS_ENV: &str = "BOOKCLERK_WORKERD_GRANT_DOMAINS";
```

Comma-separated grant domain allowlist for workerd egress.

### WORKERD_GRANT_NETWORK_MODE_ENV

```rust
pub const WORKERD_GRANT_NETWORK_MODE_ENV: &str = "BOOKCLERK_WORKERD_GRANT_NETWORK_MODE";
```

Env keys consumed by `bookclerk-workerd` (`grant.rs`) at isolate start.

### WORKERD_GRANT_POLICY_ENV

```rust
pub const WORKERD_GRANT_POLICY_ENV: &str = "BOOKCLERK_WORKERD_GRANT_POLICY";
```

Canonical JSON [`EgressPolicy`](../bookclerk_plugin_manifest/egress.md#egresspolicy) (preferred over piecemeal domain env vars).

### WORKERD_GRANT_SUBREQUESTS_ENV

```rust
pub const WORKERD_GRANT_SUBREQUESTS_ENV: &str = "BOOKCLERK_WORKERD_GRANT_SUBREQUESTS";
```

Grant subrequest budget injected into workerd `EGRESS_POLICY`.

### AUTHORIZED_PACKAGE_DIR

```rust
pub const AUTHORIZED_PACKAGE_DIR: &str = "plugin-packages";
```

Directory under the files dir that holds operator-placed package archives.

### VERSION_CACHE_DIR

```rust
pub const VERSION_CACHE_DIR: &str = ".versions";
```

Directory under `plugins/` holding materialized version trees.

### VERSION_CACHE_GRACE

```rust
pub const VERSION_CACHE_GRACE: Duration = _;
```

How long an unreferenced cached digest survives before the sweep.

### KV_GRANT_BINDING

```rust
pub const KV_GRANT_BINDING: &str = "plugin_kv";
```

Grant token that authorizes every `[[kv_namespaces]]` row of a plugin.

### CRATE_NAME_PREFIX

```rust
pub const CRATE_NAME_PREFIX: &str = "bookclerk-plugin-";
```

Required crates.io name prefix: `bookclerk-plugin-{family}-{id}`.

### PRODUCT_KEYWORD

```rust
pub const PRODUCT_KEYWORD: &str = "bookclerk";
```

Shared crates.io keyword (`bookclerk`) expected on published plugin crates.

### REGISTRY_KEYWORD

```rust
pub const REGISTRY_KEYWORD: &str = "bookclerk-plugin";
```

Keyword every published plugin crate should include.

### HOST_SHARED_ACCOUNT

```rust
pub const HOST_SHARED_ACCOUNT: &str = "host";
```

Registry-loaded source/integration guests (not a user account).

### OPERATOR_ACCOUNT

```rust
pub const OPERATOR_ACCOUNT: &str = "operator";
```

Isolation key: different accounts never share a plugin isolate.

### WORKERD_BIN_ENV

```rust
pub const WORKERD_BIN_ENV: &str = "BOOKCLERK_WORKERD_BIN";
```

Override for the pinned Cloudflare `workerd` binary, honoured by
`bookclerk-workerd` and forwarded to it by the host.

### WORKERD_LAUNCHER_ENV

```rust
pub const WORKERD_LAUNCHER_ENV: &str = "BOOKCLERK_PLUGIN_WORKERD";
```

Override for the `bookclerk-workerd` launcher path (tests / packaging).

### TEST_CHANNEL_IDENT_ENV

```rust
pub const TEST_CHANNEL_IDENT_ENV: &str = "BOOKCLERK_TEST_CHANNEL_IDENT";
```

Set to `1` or `true` in the host process to give each proxy a channel tag.

The variable is not copied into the guest. The tag is written under the
gateway session directory as [`TEST_CHANNEL_TAG_FILE`](#test_channel_tag_file) and answered only
by that session's proxy. Production leaves it unset.

### TEST_CHANNEL_TAG_FILE

```rust
pub const TEST_CHANNEL_TAG_FILE: &str = "channel-tag";
```

Session-directory file holding the tag when [`TEST_CHANNEL_IDENT_ENV`](#test_channel_ident_env) is set.

### TEST_INJECT_EXTRA_ENDPOINT_ENV

```rust
pub const TEST_INJECT_EXTRA_ENDPOINT_ENV: &str = "BOOKCLERK_TEST_INJECT_EXTRA_ENDPOINT";
```

Host-process description of one extra endpoint to add to the guest handoff.

Unix: `fd:<n>` of a socket in this process. Windows:
`handle:<read>,handle:<write>` of the guest pipe ends in this process.
The variable is not copied into the guest. Production leaves it unset, so
the guest preserve list stays [`bookclerk_sandbox::GUEST_PROXY_FD`](../bookclerk_sandbox/index.md#guest_proxy_fd) only.
Setting it transfers the endpoint. It does not, by itself, publish the
candidate metadata the guest uses to observe that slot.

### TEST_OBSERVE_EXTRA_ENDPOINT_ENV

```rust
pub const TEST_OBSERVE_EXTRA_ENDPOINT_ENV: &str = "BOOKCLERK_TEST_OBSERVE_EXTRA_ENDPOINT";
```

Host-process request to publish candidate-endpoint observation metadata.

`1` or `true` names the fixed Unix slot, and on Windows the handles from
[`TEST_INJECT_EXTRA_ENDPOINT_ENV`](#test_inject_extra_endpoint_env). A Windows `handle:<read>,handle:<write>`
value names that candidate without transferring it. The variable is not
copied into the guest. Unset means the guest reports `not-run`, which is
not proof the endpoint is absent.
