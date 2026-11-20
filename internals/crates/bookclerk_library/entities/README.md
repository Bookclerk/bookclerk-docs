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

- [`account_credential_revisions`](account_credential_revisions.md): SeaORM entity for the `account_credential_revisions` table: the newest sign-in of a single-keeper account that crossed each share (`control_plane::keeper`).
- [`account_links`](account_links.md): SeaORM entity for the `account_links` table.
- [`account_scan_state`](account_scan_state.md): SeaORM entity for `account_scan_state`: where each account's storefront scan stands between passes and between pages.
- [`account_shares`](account_shares.md): SeaORM entity for the `account_shares` table: where a store account travelled with its person (`control_plane::carried_accounts`).
- [`accounts`](accounts.md): SeaORM entity for the `accounts` table.
- [`bookclerk_receipts`](bookclerk_receipts.md): SeaORM entity for durable atomic result receipts.
- [`bookmarks`](bookmarks.md): SeaORM entity for `bookmarks`: places one person marked in one edition. Written by `crate::LibraryStore`'s playback methods (`store::playback`).
- [`books`](books.md): SeaORM entity for the `books` table.
- [`catalog_listing_optouts`](catalog_listing_optouts.md): SeaORM entity for `catalog_listing_optouts`: on a branch, the members whose books it does not list to the libraries it joined.
- [`catalog_lookups`](catalog_lookups.md): SeaORM entity for `catalog_lookups`: on a library, the catalog questions waiting for (or answered by) each branch (`control_plane::catalog_lookup`).
- [`claim_tickets`](claim_tickets.md): SeaORM entity for the `claim_tickets` table.
- [`clerk_links`](clerk_links.md): SeaORM entity for `clerk_links`: on a clerk, the libraries this install holds a library card for (written by `control_plane::record_clerk_link`).
- [`configuration_audit`](configuration_audit.md): SeaORM entity for successful configuration commits.
- [`configuration_changes`](configuration_changes.md): SeaORM entity for the configuration change notice written with each commit.
- [`configuration_documents`](configuration_documents.md): SeaORM entity for one typed configuration document.
- [`contributors`](contributors.md): SeaORM entity for the `contributors` table (text primary key).
- [`copy_grants`](copy_grants.md): SeaORM entity for `copy_grants`: one reader's leave to download the copy this library holds of one edition. Written by `control_plane::copies`.
- [`credits`](credits.md): SeaORM entity for the `credits` table (composite primary key).
- [`discover_candidates`](discover_candidates.md): SeaORM entity for the `discover_candidates` table.
- [`discover_feeds`](discover_feeds.md): SeaORM entity for the `discover_feeds` table.
- [`discover_fetches`](discover_fetches.md): SeaORM entity for the `discover_fetches` table.
- [`discover_request_queue`](discover_request_queue.md): SeaORM entity for the `discover_request_queue` table.
- [`discover_store_call_hits`](discover_store_call_hits.md): SeaORM entity for the `discover_store_call_hits` table (composite primary key).
- [`discover_store_calls`](discover_store_calls.md): SeaORM entity for the `discover_store_calls` table.
- [`discover_tastes`](discover_tastes.md): SeaORM entity for the `discover_tastes` table.
- [`domain_events`](domain_events.md): SeaORM entity for the durable `domain_events` outbox.
- [`edition_acquisitions`](edition_acquisitions.md): SeaORM entity for `edition_acquisitions`: each edition's acquire state.
- [`edition_files`](edition_files.md): SeaORM entity for `edition_files`: every file key an acquire or a host upload wrote, per output destination, and the edition it belongs to (library data model ADR, `docs/adr/library-data-model.md`, section 3). Written by the acquire pipeline (`crate::LibraryStore::record_edition_file`) and by `control_plane::uploads`, which refuses an upload a key another edition holds and deletes an aborted upload's copy only while the key is still its own.
- [`edition_identifiers`](edition_identifiers.md): SeaORM entity for the `edition_identifiers` table (composite primary key).
- [`editions`](editions.md): SeaORM entity for the `editions` table (text primary key).
- [`embeddings`](embeddings.md): SeaORM entity for the `embeddings` table.
- [`encrypted_secrets`](encrypted_secrets.md): SeaORM entity for the `encrypted_secrets` table.
- [`event_deliveries`](event_deliveries.md): SeaORM entity for per-subscriber `event_deliveries`.
- [`event_outbox_stats`](event_outbox_stats.md): SeaORM entity for the singleton durable event-outbox counters.
- [`event_subscriber_nodes`](event_subscriber_nodes.md): SeaORM entity for live per-node `event_subscriber_nodes` registrations.
- [`expressions`](expressions.md): SeaORM entity for the `expressions` table (text primary key).
- [`holdings`](holdings.md): SeaORM entity for the `holdings` table: who holds a store listing.
- [`ignored_titles`](ignored_titles.md): SeaORM entity for the `ignored_titles` table entity (composite primary key).
- [`job_secret_handoffs`](job_secret_handoffs.md): SeaORM entity for `job_secret_handoffs`: which store-account secrets a Host was handed with a job, per lease generation.
- [`job_temp_paths`](job_temp_paths.md): SeaORM entity for scratch directories associated with a durable job.
- [`jobs`](jobs.md): SeaORM entity for the durable daemon `jobs` table.
- [`library_identity`](library_identity.md): SeaORM entity for the singleton library identity and secret-root fingerprint.
- [`library_index_builds`](library_index_builds.md): SeaORM entity for `library_index_builds`: finished page index builds.
- [`library_leases`](library_leases.md): SeaORM entity for `library_leases`: singleton maintenance leases.
- [`library_links`](library_links.md): SeaORM entity for `library_links`: on a branch node, the libraries it enrolled into (written by `control_plane::record_library_link`).
- [`library_work_facets`](library_work_facets.md): SeaORM entity for `library_work_facets`: the page index's filter values.
- [`library_work_rows`](library_work_rows.md): SeaORM entity for `library_work_rows`: the library's page index.
- [`link_imports`](link_imports.md): SeaORM entity for `link_imports`: on a branch, the branch owner's books acquired for a library it joined and imported into this library as well (written by `control_plane::link_imports`). The catalog sync never sends a library a book recorded here for it.
- [`link_requests`](link_requests.md): SeaORM entity for `link_requests`: on a branch, the books a library it joined asked for through an interlibrary request (written by `control_plane::branch_requests`). The catalog sync sends that library a book recorded here for it, although the book is a member's.
- [`listening_progress`](listening_progress.md): SeaORM entity for the `listening_progress` table.
- [`listing_offers`](listing_offers.md): SeaORM entity for the `listing_offers` table (composite primary key).
- [`node_credentials`](node_credentials.md): SeaORM entity for `node_credentials`: the public keys an enrolled host node proves itself with.
- [`node_enrollment_links`](node_enrollment_links.md): SeaORM entity for `node_enrollment_links`: single-use links that enroll a branch node into this library. Only the hash of each link's secret is stored.
- [`node_status`](node_status.md): SeaORM entity for `node_status`: each node's latest daemon heartbeat.
- [`node_upload_sessions`](node_upload_sessions.md): SeaORM entity for `node_upload_sessions`: one title's streamed upload from a branch node to a shared node (H-4 design, section 6). Written by `control_plane::uploads`.
- [`nodes`](nodes.md): SeaORM entity for durable node identity and liveness observations.
- [`oidc_auth_codes`](oidc_auth_codes.md): SeaORM entity for the `oidc_auth_codes` — short-lived authorization codes (PKCE).
- [`oidc_clients`](oidc_clients.md): SeaORM entity for the `oidc_clients` — registered OAuth/OIDC clients.
- [`oidc_refresh_tokens`](oidc_refresh_tokens.md): SeaORM entity for the `oidc_refresh_tokens` — refresh tokens bound to a User.
- [`oidc_rp_states`](oidc_rp_states.md): SeaORM entity for OIDC relying-party authorize state (PKCE + nonce).
- [`operator_sessions`](operator_sessions.md): SeaORM entity for the `operator_sessions` table.
- [`patron_controls`](patron_controls.md): SeaORM entity for the `patron_controls` table: one Member's overrides of the library's default member controls, set by an Administrator.
- [`patron_scope_keys`](patron_scope_keys.md): SeaORM entity for the `patron_scope_keys` table: the include and exclude values of a patron scope as rows, so the Library filter joins them instead of binding one parameter per rule.
- [`patron_scopes`](patron_scopes.md): SeaORM entity for the `patron_scopes` table: a Member's own patron scope, set by an Administrator.
- [`pending_joins`](pending_joins.md): SeaORM entity for `pending_joins`: on a host joining a library, one sign-in in progress (written by `control_plane::create_pending_join`). The PKCE verifier and the node keys are sealed in `encrypted_secrets`, never stored here.
- [`playback_progress`](playback_progress.md): SeaORM entity for `playback_progress`: how far one person has listened to one edition. Written by `crate::LibraryStore`'s playback methods (`store::playback`).
- [`playback_sessions`](playback_sessions.md): SeaORM entity for `playback_sessions`: one stretch of listening by one person on one player. A session is also the leave to stream the edition's files from one destination, checked afresh on every request (`control_plane::copies::authorize_playback`).
- [`plugin_artifacts`](plugin_artifacts.md): SeaORM entity for `plugin_artifacts`: the header of a version archive stored in the library when no shared artifact location exists.
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
- [`remote_plugin_consents`](remote_plugin_consents.md): SeaORM entity for `remote_plugin_consents`: on a branch node, this install's operator consent for each owner plugin a library wants to run here (written by `control_plane::remote_consents`).
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
- [`user_title_state`](user_title_state.md): SeaORM entity for the `user_title_state` table.
- [`user_work_tags`](user_work_tags.md): SeaORM entity for `user_work_tags`: one user's tags on a work, one row per tag.
- [`users`](users.md): SeaORM entity for the `users` table entity — first-party identity principal (#117).
- [`webauthn_challenges`](webauthn_challenges.md): SeaORM entity for in-flight WebAuthn ceremonies.
- [`webauthn_credentials`](webauthn_credentials.md): SeaORM entity for WebAuthn / passkey credentials.
- [`window_book_parts`](window_book_parts.md): SeaORM entity for `window_book_parts`: one piece of a window book's stored edition and identifiers (`control_plane::window_books`).
- [`window_books`](window_books.md): SeaORM entity for `window_books`: on a branch or a clerk, the books of each library it reads, one row per edition (`control_plane::window_books`).
- [`work_identifiers`](work_identifiers.md): SeaORM entity for the `work_identifiers` table (composite primary key).
- [`works`](works.md): SeaORM entity for the `works` table (text primary key).
