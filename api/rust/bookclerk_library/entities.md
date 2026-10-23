# bookclerk_library::entities

SeaORM entities mirroring the greenfield schema in [`crate::migrations`](migrations.md).

One `DeriveEntityModel` per table. Column Rust types match the proxy/native
backends: integer columns are `i64`, reals `f64`, blobs `Vec<u8>`, and text
(including RFC 3339 timestamps) `String`. Timestamps stay as `String` to
match the `TEXT` columns without relying on SeaORM's proxy chrono decoding;
[`LibraryStore`](index.md#librarystore) parses them into `chrono::DateTime<Utc>`
at the record boundary and uses these entities (`Entity::find`,
`ActiveModel`) for the majority of CRUD.

## Modules

- [`account_links`](entities/account_links.md): SeaORM entity for the `account_links` table.
- [`accounts`](entities/accounts.md): SeaORM entity for the `accounts` table.
- [`bookclerk_receipts`](entities/bookclerk_receipts.md): SeaORM entity for durable atomic result receipts.
- [`books`](entities/books.md): SeaORM entity for the `books` table.
- [`branch_titles`](entities/branch_titles.md): SeaORM entity for `branch_titles`: on a library, the titles a branch shared with it (`catalog.sync`; written by `control_plane::branch_catalog`). Rows name the branch's node and its owner user only; nothing names the branch's store accounts, users or own branches.
- [`claim_tickets`](entities/claim_tickets.md): SeaORM entity for the `claim_tickets` table.
- [`cluster_identity`](entities/cluster_identity.md): SeaORM entity for the singleton cluster identity and secret-root fingerprint.
- [`cluster_leases`](entities/cluster_leases.md): SeaORM entity for `cluster_leases`: singleton maintenance leases.
- [`cluster_links`](entities/cluster_links.md): SeaORM entity for `cluster_links`: on a host node, the owner clusters it enrolled into (written by `control_plane::record_cluster_link`).
- [`configuration_audit`](entities/configuration_audit.md): SeaORM entity for successful configuration commits.
- [`configuration_changes`](entities/configuration_changes.md): SeaORM entity for the configuration change notice written with each commit.
- [`configuration_documents`](entities/configuration_documents.md): SeaORM entity for one typed configuration document.
- [`domain_events`](entities/domain_events.md): SeaORM entity for the durable `domain_events` outbox.
- [`embeddings`](entities/embeddings.md): SeaORM entity for the `embeddings` table.
- [`encrypted_secrets`](entities/encrypted_secrets.md): SeaORM entity for the `encrypted_secrets` table.
- [`event_deliveries`](entities/event_deliveries.md): SeaORM entity for per-subscriber `event_deliveries`.
- [`event_outbox_stats`](entities/event_outbox_stats.md): SeaORM entity for the singleton durable event-outbox counters.
- [`event_subscriber_nodes`](entities/event_subscriber_nodes.md): SeaORM entity for live per-node `event_subscriber_nodes` registrations.
- [`ignored_titles`](entities/ignored_titles.md): SeaORM entity for the `ignored_titles` table entity (composite primary key).
- [`job_secret_handoffs`](entities/job_secret_handoffs.md): SeaORM entity for `job_secret_handoffs`: which store-account secrets a Host was handed with a job, per lease generation.
- [`job_temp_paths`](entities/job_temp_paths.md): SeaORM entity for scratch directories associated with a durable job.
- [`jobs`](entities/jobs.md): SeaORM entity for the durable daemon `jobs` table.
- [`listening_progress`](entities/listening_progress.md): SeaORM entity for the `listening_progress` table.
- [`node_credentials`](entities/node_credentials.md): SeaORM entity for `node_credentials`: the public keys an enrolled host node proves itself with.
- [`node_enrollment_links`](entities/node_enrollment_links.md): SeaORM entity for `node_enrollment_links`: single-use links that enroll a host node into this cluster. Only the hash of each link's secret is stored.
- [`node_status`](entities/node_status.md): SeaORM entity for `node_status`: each node's latest daemon heartbeat.
- [`node_upload_sessions`](entities/node_upload_sessions.md): SeaORM entity for `node_upload_sessions`: one title's streamed upload from a host node to an owner node (H-4 design, section 6). Written by `control_plane::uploads`.
- [`nodes`](entities/nodes.md): SeaORM entity for durable node identity and liveness observations.
- [`oidc_auth_codes`](entities/oidc_auth_codes.md): SeaORM entity for the `oidc_auth_codes` — short-lived authorization codes (PKCE).
- [`oidc_clients`](entities/oidc_clients.md): SeaORM entity for the `oidc_clients` — registered OAuth/OIDC clients.
- [`oidc_refresh_tokens`](entities/oidc_refresh_tokens.md): SeaORM entity for the `oidc_refresh_tokens` — refresh tokens bound to a User.
- [`oidc_rp_states`](entities/oidc_rp_states.md): SeaORM entity for OIDC relying-party authorize state (PKCE + nonce).
- [`operator_sessions`](entities/operator_sessions.md): SeaORM entity for the `operator_sessions` table.
- [`pending_joins`](entities/pending_joins.md): SeaORM entity for `pending_joins`: on a host joining a library, one sign-in in progress (written by `control_plane::create_pending_join`). The PKCE verifier and the node keys are sealed in `encrypted_secrets`, never stored here.
- [`plugin_artifacts`](entities/plugin_artifacts.md): SeaORM entity for `plugin_artifacts`: small version archives stored in the library when no shared artifact location exists.
- [`plugin_deployment_observations`](entities/plugin_deployment_observations.md): SeaORM entity for observed plugin deployment state.
- [`plugin_deployments`](entities/plugin_deployments.md): SeaORM entity for desired plugin deployment state.
- [`plugin_installs`](entities/plugin_installs.md): SeaORM entity for `plugin_installs`: the active version per plugin.
- [`plugin_instances`](entities/plugin_instances.md): SeaORM entity for a durable plugin instance.
- [`plugin_kv`](entities/plugin_kv.md): SeaORM entity for the per-plugin durable key/value table `plugin_kv`.
- [`plugin_placements`](entities/plugin_placements.md): SeaORM entity for `plugin_placements`: the scheduler's input per instance.
- [`plugin_schemas`](entities/plugin_schemas.md): SeaORM entity for the `plugin_schemas` table.
- [`plugin_versions`](entities/plugin_versions.md): SeaORM entity for `plugin_versions`: one installed version of a plugin.
- [`portal_identities`](entities/portal_identities.md): SeaORM entity for the `portal_identities` table.
- [`portal_sessions`](entities/portal_sessions.md): SeaORM entity for the `portal_sessions` table.
- [`remote_plugin_consents`](entities/remote_plugin_consents.md): SeaORM entity for `remote_plugin_consents`: on a host node, this install's operator consent for each owner plugin an owner cluster wants to run here (written by `control_plane::remote_consents`).
- [`saved_filters`](entities/saved_filters.md): SeaORM entity for the `saved_filters` table.
- [`security_audit_events`](entities/security_audit_events.md): SeaORM entity for the `security_audit_events` table entity — elevate / impersonate / login audit.
- [`storage_scan_generations`](entities/storage_scan_generations.md): Ownership row for one durable storage-scan generation.
- [`storage_scan_rows`](entities/storage_scan_rows.md): SeaORM entity for paged storage-scan rows.
- [`title_files`](entities/title_files.md): SeaORM entity for `title_files`: every file key a title's acquire or host upload wrote, per output destination (H-4 design, section 6). Written by the acquire pipeline (`crate::LibraryStore::record_title_file`) and by `control_plane::uploads`, which refuses an upload a key another title holds and deletes an aborted upload's copy only while the key is still its own.
- [`title_request_sources`](entities/title_request_sources.md): SeaORM entity for the `title_request_sources` — per-storefront catalog/pricing snapshot for a wish.
- [`title_requests`](entities/title_requests.md): SeaORM entity for the `title_requests` table.
- [`user_invites`](entities/user_invites.md): SeaORM entity for the `user_invites` table entity — admin/operator invite tickets with role.
- [`user_preferences`](entities/user_preferences.md): SeaORM entity for the `user_preferences` table.
- [`users`](entities/users.md): SeaORM entity for the `users` table entity — first-party identity principal (#117).
- [`webauthn_challenges`](entities/webauthn_challenges.md): SeaORM entity for in-flight WebAuthn ceremonies.
- [`webauthn_credentials`](entities/webauthn_credentials.md): SeaORM entity for WebAuthn / passkey credentials.
- [`work_editions`](entities/work_editions.md): SeaORM entity for the `work_editions` table entity (composite primary key).
- [`works`](entities/works.md): SeaORM entity for the `works` table entity (text primary key).
