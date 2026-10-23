**bookclerk_library > entities**

# Module: entities

## Contents

**Modules**

- [`account_links`](#account_links) - SeaORM entity for the `account_links` table.
- [`accounts`](#accounts) - SeaORM entity for the `accounts` table.
- [`bookclerk_receipts`](#bookclerk_receipts) - SeaORM entity for durable atomic result receipts.
- [`books`](#books) - SeaORM entity for the `books` table.
- [`branch_titles`](#branch_titles) - SeaORM entity for `branch_titles`: on a library, the titles a branch
- [`claim_tickets`](#claim_tickets) - SeaORM entity for the `claim_tickets` table.
- [`cluster_identity`](#cluster_identity) - SeaORM entity for the singleton cluster identity and secret-root fingerprint.
- [`cluster_leases`](#cluster_leases) - SeaORM entity for `cluster_leases`: singleton maintenance leases.
- [`cluster_links`](#cluster_links) - SeaORM entity for `cluster_links`: on a host node, the owner clusters it
- [`configuration_audit`](#configuration_audit) - SeaORM entity for successful configuration commits.
- [`configuration_changes`](#configuration_changes) - SeaORM entity for the configuration change notice written with each commit.
- [`configuration_documents`](#configuration_documents) - SeaORM entity for one typed configuration document.
- [`domain_events`](#domain_events) - SeaORM entity for the durable `domain_events` outbox.
- [`embeddings`](#embeddings) - SeaORM entity for the `embeddings` table.
- [`encrypted_secrets`](#encrypted_secrets) - SeaORM entity for the `encrypted_secrets` table.
- [`event_deliveries`](#event_deliveries) - SeaORM entity for per-subscriber `event_deliveries`.
- [`event_outbox_stats`](#event_outbox_stats) - SeaORM entity for the singleton durable event-outbox counters.
- [`event_subscriber_nodes`](#event_subscriber_nodes) - SeaORM entity for live per-node `event_subscriber_nodes` registrations.
- [`ignored_titles`](#ignored_titles) - SeaORM entity for the `ignored_titles` table entity (composite primary key).
- [`job_secret_handoffs`](#job_secret_handoffs) - SeaORM entity for `job_secret_handoffs`: which store-account secrets a
- [`job_temp_paths`](#job_temp_paths) - SeaORM entity for scratch directories associated with a durable job.
- [`jobs`](#jobs) - SeaORM entity for the durable daemon `jobs` table.
- [`listening_progress`](#listening_progress) - SeaORM entity for the `listening_progress` table.
- [`node_credentials`](#node_credentials) - SeaORM entity for `node_credentials`: the public keys an enrolled host
- [`node_enrollment_links`](#node_enrollment_links) - SeaORM entity for `node_enrollment_links`: single-use links that enroll a
- [`node_status`](#node_status) - SeaORM entity for `node_status`: each node's latest daemon heartbeat.
- [`node_upload_sessions`](#node_upload_sessions) - SeaORM entity for `node_upload_sessions`: one title's streamed upload
- [`nodes`](#nodes) - SeaORM entity for durable node identity and liveness observations.
- [`oidc_auth_codes`](#oidc_auth_codes) - SeaORM entity for the `oidc_auth_codes` — short-lived authorization codes (PKCE).
- [`oidc_clients`](#oidc_clients) - SeaORM entity for the `oidc_clients` — registered OAuth/OIDC clients.
- [`oidc_refresh_tokens`](#oidc_refresh_tokens) - SeaORM entity for the `oidc_refresh_tokens` — refresh tokens bound to a User.
- [`oidc_rp_states`](#oidc_rp_states) - SeaORM entity for OIDC relying-party authorize state (PKCE + nonce).
- [`operator_sessions`](#operator_sessions) - SeaORM entity for the `operator_sessions` table.
- [`pending_joins`](#pending_joins) - SeaORM entity for `pending_joins`: on a host joining a library, one
- [`plugin_artifacts`](#plugin_artifacts) - SeaORM entity for `plugin_artifacts`: small version archives stored
- [`plugin_deployment_observations`](#plugin_deployment_observations) - SeaORM entity for observed plugin deployment state.
- [`plugin_deployments`](#plugin_deployments) - SeaORM entity for desired plugin deployment state.
- [`plugin_installs`](#plugin_installs) - SeaORM entity for `plugin_installs`: the active version per plugin.
- [`plugin_instances`](#plugin_instances) - SeaORM entity for a durable plugin instance.
- [`plugin_kv`](#plugin_kv) - SeaORM entity for the per-plugin durable key/value table `plugin_kv`.
- [`plugin_placements`](#plugin_placements) - SeaORM entity for `plugin_placements`: the scheduler's input per instance.
- [`plugin_schemas`](#plugin_schemas) - SeaORM entity for the `plugin_schemas` table.
- [`plugin_versions`](#plugin_versions) - SeaORM entity for `plugin_versions`: one installed version of a plugin.
- [`portal_identities`](#portal_identities) - SeaORM entity for the `portal_identities` table.
- [`portal_sessions`](#portal_sessions) - SeaORM entity for the `portal_sessions` table.
- [`remote_plugin_consents`](#remote_plugin_consents) - SeaORM entity for `remote_plugin_consents`: on a host node, this
- [`saved_filters`](#saved_filters) - SeaORM entity for the `saved_filters` table.
- [`security_audit_events`](#security_audit_events) - SeaORM entity for the `security_audit_events` table entity — elevate / impersonate / login audit.
- [`storage_scan_generations`](#storage_scan_generations) - Ownership row for one durable storage-scan generation.
- [`storage_scan_rows`](#storage_scan_rows) - SeaORM entity for paged storage-scan rows.
- [`title_files`](#title_files) - SeaORM entity for `title_files`: every file key a title's acquire or
- [`title_request_sources`](#title_request_sources) - SeaORM entity for the `title_request_sources` — per-storefront catalog/pricing snapshot for a wish.
- [`title_requests`](#title_requests) - SeaORM entity for the `title_requests` table.
- [`user_invites`](#user_invites) - SeaORM entity for the `user_invites` table entity — admin/operator invite tickets with role.
- [`user_preferences`](#user_preferences) - SeaORM entity for the `user_preferences` table.
- [`users`](#users) - SeaORM entity for the `users` table entity — first-party identity principal (#117).
- [`webauthn_challenges`](#webauthn_challenges) - SeaORM entity for in-flight WebAuthn ceremonies.
- [`webauthn_credentials`](#webauthn_credentials) - SeaORM entity for WebAuthn / passkey credentials.
- [`work_editions`](#work_editions) - SeaORM entity for the `work_editions` table entity (composite primary key).
- [`works`](#works) - SeaORM entity for the `works` table entity (text primary key).

---

## Module: account_links

SeaORM entity for the `account_links` table.



## Module: accounts

SeaORM entity for the `accounts` table.



## Module: bookclerk_receipts

SeaORM entity for durable atomic result receipts.



## Module: books

SeaORM entity for the `books` table.



## Module: branch_titles

SeaORM entity for `branch_titles`: on a library, the titles a branch
shared with it (`catalog.sync`; written by `control_plane::branch_catalog`).
Rows name the branch's node and its owner user only; nothing names the
branch's store accounts, users or own branches.



## Module: claim_tickets

SeaORM entity for the `claim_tickets` table.



## Module: cluster_identity

SeaORM entity for the singleton cluster identity and secret-root fingerprint.



## Module: cluster_leases

SeaORM entity for `cluster_leases`: singleton maintenance leases.



## Module: cluster_links

SeaORM entity for `cluster_links`: on a host node, the owner clusters it
enrolled into (written by `control_plane::record_cluster_link`).



## Module: configuration_audit

SeaORM entity for successful configuration commits.



## Module: configuration_changes

SeaORM entity for the configuration change notice written with each commit.



## Module: configuration_documents

SeaORM entity for one typed configuration document.



## Module: domain_events

SeaORM entity for the durable `domain_events` outbox.



## Module: embeddings

SeaORM entity for the `embeddings` table.



## Module: encrypted_secrets

SeaORM entity for the `encrypted_secrets` table.

Retained in the schema and entities per the multi-storefront design; the
encryption / secrets-file removal work lives in [`crate::secrets`](secrets.md).



## Module: event_deliveries

SeaORM entity for per-subscriber `event_deliveries`.



## Module: event_outbox_stats

SeaORM entity for the singleton durable event-outbox counters.



## Module: event_subscriber_nodes

SeaORM entity for live per-node `event_subscriber_nodes` registrations.



## Module: ignored_titles

SeaORM entity for the `ignored_titles` table entity (composite primary key).



## Module: job_secret_handoffs

SeaORM entity for `job_secret_handoffs`: which store-account secrets a
Host was handed with a job, per lease generation.

Recorded when the claim hands the secrets over and read when the Host
writes a refreshed token back, so a write-back is accepted only for a
secret that went out under that exact fence. Carries no secret material.



## Module: job_temp_paths

SeaORM entity for scratch directories associated with a durable job.



## Module: jobs

SeaORM entity for the durable daemon `jobs` table.



## Module: listening_progress

SeaORM entity for the `listening_progress` table.



## Module: node_credentials

SeaORM entity for `node_credentials`: the public keys an enrolled host
node proves itself with.



## Module: node_enrollment_links

SeaORM entity for `node_enrollment_links`: single-use links that enroll a
host node into this cluster. Only the hash of each link's secret is stored.



## Module: node_status

SeaORM entity for `node_status`: each node's latest daemon heartbeat.



## Module: node_upload_sessions

SeaORM entity for `node_upload_sessions`: one title's streamed upload
from a host node to an owner node (H-4 design, section 6). Written by
`control_plane::uploads`.



## Module: nodes

SeaORM entity for durable node identity and liveness observations.



## Module: oidc_auth_codes

SeaORM entity for the `oidc_auth_codes` — short-lived authorization codes (PKCE).



## Module: oidc_clients

SeaORM entity for the `oidc_clients` — registered OAuth/OIDC clients.



## Module: oidc_refresh_tokens

SeaORM entity for the `oidc_refresh_tokens` — refresh tokens bound to a User.



## Module: oidc_rp_states

SeaORM entity for OIDC relying-party authorize state (PKCE + nonce).



## Module: operator_sessions

SeaORM entity for the `operator_sessions` table.



## Module: pending_joins

SeaORM entity for `pending_joins`: on a host joining a library, one
sign-in in progress (written by `control_plane::create_pending_join`).
The PKCE verifier and the node keys are sealed in `encrypted_secrets`,
never stored here.



## Module: plugin_artifacts

SeaORM entity for `plugin_artifacts`: small version archives stored
in the library when no shared artifact location exists.



## Module: plugin_deployment_observations

SeaORM entity for observed plugin deployment state.



## Module: plugin_deployments

SeaORM entity for desired plugin deployment state.



## Module: plugin_installs

SeaORM entity for `plugin_installs`: the active version per plugin.



## Module: plugin_instances

SeaORM entity for a durable plugin instance.



## Module: plugin_kv

SeaORM entity for the per-plugin durable key/value table `plugin_kv`.



## Module: plugin_placements

SeaORM entity for `plugin_placements`: the scheduler's input per instance.



## Module: plugin_schemas

SeaORM entity for the `plugin_schemas` table.



## Module: plugin_versions

SeaORM entity for `plugin_versions`: one installed version of a plugin.



## Module: portal_identities

SeaORM entity for the `portal_identities` table.



## Module: portal_sessions

SeaORM entity for the `portal_sessions` table.



## Module: remote_plugin_consents

SeaORM entity for `remote_plugin_consents`: on a host node, this
install's operator consent for each owner plugin an owner cluster wants
to run here (written by `control_plane::remote_consents`).



## Module: saved_filters

SeaORM entity for the `saved_filters` table.



## Module: security_audit_events

SeaORM entity for the `security_audit_events` table entity — elevate / impersonate / login audit.



## Module: storage_scan_generations

Ownership row for one durable storage-scan generation.

`storage_scan_rows` are keyed only by `scan_id`. This row binds that id to
the storage instance and, when the scan runs inside a job, the job that
owns it. Cleanup uses the row to tell a live scan from an abandoned one.



## Module: storage_scan_rows

SeaORM entity for paged storage-scan rows.

Object and identity hits live here so a scan does not retain the inventory
in process memory. `kind` is `object` or `identity`.



## Module: title_files

SeaORM entity for `title_files`: every file key a title's acquire or
host upload wrote, per output destination (H-4 design, section 6).
Written by the acquire pipeline (`crate::LibraryStore::record_title_file`)
and by `control_plane::uploads`, which refuses an upload a key another
title holds and deletes an aborted upload's copy only while the key is
still its own.



## Module: title_request_sources

SeaORM entity for the `title_request_sources` — per-storefront catalog/pricing snapshot for a wish.



## Module: title_requests

SeaORM entity for the `title_requests` table.



## Module: user_invites

SeaORM entity for the `user_invites` table entity — admin/operator invite tickets with role.



## Module: user_preferences

SeaORM entity for the `user_preferences` table.



## Module: users

SeaORM entity for the `users` table entity — first-party identity principal (#117).



## Module: webauthn_challenges

SeaORM entity for in-flight WebAuthn ceremonies.



## Module: webauthn_credentials

SeaORM entity for WebAuthn / passkey credentials.



## Module: work_editions

SeaORM entity for the `work_editions` table entity (composite primary key).



## Module: works

SeaORM entity for the `works` table entity (text primary key).



