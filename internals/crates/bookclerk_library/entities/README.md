# bookclerk_library::entities

SeaORM entities mirroring the greenfield schema in [`crate::migrations`](../migrations.md).

One `DeriveEntityModel` per table. Column Rust types match the proxy/native
backends: integer columns are `i64`, reals `f64`, blobs `Vec<u8>`, and text
(including RFC 3339 timestamps) `String`. Timestamps stay as `String` to
match the `TEXT` columns without relying on SeaORM's proxy chrono decoding;
[`LibraryStore`](../README.md#librarystore) parses them into `chrono::DateTime<Utc>`
at the record boundary and uses these entities (`Entity::find`,
`ActiveModel`) for the majority of CRUD.

## Modules

- [`account_links`](account_links.md): SeaORM entity for the `account_links` table.
- [`accounts`](accounts.md): SeaORM entity for the `accounts` table.
- [`bookclerk_receipts`](bookclerk_receipts.md): SeaORM entity for durable atomic result receipts.
- [`books`](books.md): SeaORM entity for the `books` table.
- [`branch_titles`](branch_titles.md): SeaORM entity for `branch_titles`: on a library, the titles a branch shared with it (`catalog.sync`; written by `control_plane::branch_catalog`). Rows name the branch's node and its owner user only; nothing names the branch's store accounts, users or own branches.
- [`claim_tickets`](claim_tickets.md): SeaORM entity for the `claim_tickets` table.
- [`cluster_identity`](cluster_identity.md): SeaORM entity for the singleton cluster identity and secret-root fingerprint.
- [`cluster_leases`](cluster_leases.md): SeaORM entity for `cluster_leases`: singleton maintenance leases.
- [`cluster_links`](cluster_links.md): SeaORM entity for `cluster_links`: on a host node, the owner clusters it enrolled into (written by `control_plane::record_cluster_link`).
- [`configuration_audit`](configuration_audit.md): SeaORM entity for successful configuration commits.
- [`configuration_changes`](configuration_changes.md): SeaORM entity for the configuration change notice written with each commit.
- [`configuration_documents`](configuration_documents.md): SeaORM entity for one typed configuration document.
- [`contributors`](contributors.md): SeaORM entity for the `contributors` table (text primary key).
- [`credits`](credits.md): SeaORM entity for the `credits` table (composite primary key).
- [`discover_candidates`](discover_candidates.md): SeaORM entity for the `discover_candidates` table.
- [`discover_feeds`](discover_feeds.md): SeaORM entity for the `discover_feeds` table.
- [`discover_fetches`](discover_fetches.md): SeaORM entity for the `discover_fetches` table.
- [`discover_store_calls`](discover_store_calls.md): SeaORM entity for the `discover_store_calls` table.
- [`domain_events`](domain_events.md): SeaORM entity for the durable `domain_events` outbox.
- [`edition_files`](edition_files.md): SeaORM entity for `edition_files`: every file key an acquire or a host upload wrote, per output destination, and the edition it belongs to (library data model ADR, `docs/adr/library-data-model.md`, section 3). Written by the acquire pipeline (`crate::LibraryStore::record_edition_file`) and by `control_plane::uploads`, which refuses an upload a key another edition holds and deletes an aborted upload's copy only while the key is still its own.
- [`edition_identifiers`](edition_identifiers.md): SeaORM entity for the `edition_identifiers` table (composite primary key).
- [`editions`](editions.md): SeaORM entity for the `editions` table (text primary key).
- [`embeddings`](embeddings.md): SeaORM entity for the `embeddings` table.
- [`encrypted_secrets`](encrypted_secrets.md): SeaORM entity for the `encrypted_secrets` table.
- [`event_deliveries`](event_deliveries.md): SeaORM entity for per-subscriber `event_deliveries`.
- [`event_outbox_stats`](event_outbox_stats.md): SeaORM entity for the singleton durable event-outbox counters.
- [`event_subscriber_nodes`](event_subscriber_nodes.md): SeaORM entity for live per-node `event_subscriber_nodes` registrations.
- [`expressions`](expressions.md): SeaORM entity for the `expressions` table (text primary key).
- [`holdings`](holdings.md): SeaORM entity for the `holdings` table (composite primary key).
- [`ignored_titles`](ignored_titles.md): SeaORM entity for the `ignored_titles` table entity (composite primary key).
- [`job_secret_handoffs`](job_secret_handoffs.md): SeaORM entity for `job_secret_handoffs`: which store-account secrets a Host was handed with a job, per lease generation.
- [`job_temp_paths`](job_temp_paths.md): SeaORM entity for scratch directories associated with a durable job.
- [`jobs`](jobs.md): SeaORM entity for the durable daemon `jobs` table.
- [`listening_progress`](listening_progress.md): SeaORM entity for the `listening_progress` table.
- [`node_credentials`](node_credentials.md): SeaORM entity for `node_credentials`: the public keys an enrolled host node proves itself with.
- [`node_enrollment_links`](node_enrollment_links.md): SeaORM entity for `node_enrollment_links`: single-use links that enroll a host node into this cluster. Only the hash of each link's secret is stored.
- [`node_status`](node_status.md): SeaORM entity for `node_status`: each node's latest daemon heartbeat.
- [`node_upload_sessions`](node_upload_sessions.md): SeaORM entity for `node_upload_sessions`: one title's streamed upload from a host node to an owner node (H-4 design, section 6). Written by `control_plane::uploads`.
- [`nodes`](nodes.md): SeaORM entity for durable node identity and liveness observations.
- [`oidc_auth_codes`](oidc_auth_codes.md): SeaORM entity for the `oidc_auth_codes` — short-lived authorization codes (PKCE).
- [`oidc_clients`](oidc_clients.md): SeaORM entity for the `oidc_clients` — registered OAuth/OIDC clients.
- [`oidc_refresh_tokens`](oidc_refresh_tokens.md): SeaORM entity for the `oidc_refresh_tokens` — refresh tokens bound to a User.
- [`oidc_rp_states`](oidc_rp_states.md): SeaORM entity for OIDC relying-party authorize state (PKCE + nonce).
- [`operator_sessions`](operator_sessions.md): SeaORM entity for the `operator_sessions` table.
- [`pending_joins`](pending_joins.md): SeaORM entity for `pending_joins`: on a host joining a library, one sign-in in progress (written by `control_plane::create_pending_join`). The PKCE verifier and the node keys are sealed in `encrypted_secrets`, never stored here.
- [`plugin_artifacts`](plugin_artifacts.md): SeaORM entity for `plugin_artifacts`: small version archives stored in the library when no shared artifact location exists.
- [`plugin_cron_cursors`](plugin_cron_cursors.md): SeaORM entity for `plugin_cron_cursors`: the last claimed fire of each plugin cron schedule.
- [`plugin_deployment_observations`](plugin_deployment_observations.md): SeaORM entity for observed plugin deployment state.
- [`plugin_deployments`](plugin_deployments.md): SeaORM entity for desired plugin deployment state.
- [`plugin_installs`](plugin_installs.md): SeaORM entity for `plugin_installs`: the active version per plugin.
- [`plugin_instances`](plugin_instances.md): SeaORM entity for a durable plugin instance.
- [`plugin_kv`](plugin_kv.md): SeaORM entity for the per-plugin durable key/value table `plugin_kv`.
- [`plugin_placements`](plugin_placements.md): SeaORM entity for `plugin_placements`: the scheduler's input per instance.
- [`plugin_schemas`](plugin_schemas.md): SeaORM entity for the `plugin_schemas` table.
- [`plugin_versions`](plugin_versions.md): SeaORM entity for `plugin_versions`: one installed version of a plugin.
- [`portal_identities`](portal_identities.md): SeaORM entity for the `portal_identities` table.
- [`portal_sessions`](portal_sessions.md): SeaORM entity for the `portal_sessions` table.
- [`remote_plugin_consents`](remote_plugin_consents.md): SeaORM entity for `remote_plugin_consents`: on a host node, this install's operator consent for each owner plugin an owner cluster wants to run here (written by `control_plane::remote_consents`).
- [`resolution_conflicts`](resolution_conflicts.md): SeaORM entity for the `resolution_conflicts` table (composite primary key).
- [`resolution_overrides`](resolution_overrides.md): SeaORM entity for the `resolution_overrides` table (composite primary key).
- [`saved_filters`](saved_filters.md): SeaORM entity for the `saved_filters` table.
- [`security_audit_events`](security_audit_events.md): SeaORM entity for the `security_audit_events` table entity — elevate / impersonate / login audit.
- [`series`](series.md): SeaORM entity for the `series` table (text primary key).
- [`series_entries`](series_entries.md): SeaORM entity for the `series_entries` table (composite primary key).
- [`storage_scan_generations`](storage_scan_generations.md): Ownership row for one durable storage-scan generation.
- [`storage_scan_rows`](storage_scan_rows.md): SeaORM entity for paged storage-scan rows.
- [`store_listings`](store_listings.md): SeaORM entity for the `store_listings` table (text primary key).
- [`title_request_sources`](title_request_sources.md): SeaORM entity for the `title_request_sources` — per-storefront catalog/pricing snapshot for a wish.
- [`title_requests`](title_requests.md): SeaORM entity for the `title_requests` table.
- [`user_invites`](user_invites.md): SeaORM entity for the `user_invites` table entity — admin/operator invite tickets with role.
- [`user_preferences`](user_preferences.md): SeaORM entity for the `user_preferences` table.
- [`users`](users.md): SeaORM entity for the `users` table entity — first-party identity principal (#117).
- [`webauthn_challenges`](webauthn_challenges.md): SeaORM entity for in-flight WebAuthn ceremonies.
- [`webauthn_credentials`](webauthn_credentials.md): SeaORM entity for WebAuthn / passkey credentials.
- [`work_identifiers`](work_identifiers.md): SeaORM entity for the `work_identifiers` table (composite primary key).
- [`works`](works.md): SeaORM entity for the `works` table (text primary key).
