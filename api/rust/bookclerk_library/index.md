# bookclerk_library

Canonical Bookclerk library database.

[`LibraryStore`](store.md#bookclerk_librarystorelibrarystore) is SeaORM-backed: every backend is a
[`sea_orm::DatabaseConnection`](https://docs.rs/sea_orm/latest/sea_orm/database/db_connection/struct.DatabaseConnection.html). Engine connect/migrate/proxy quirks live in
the database plugin (`bookclerk-plugin-database-sqlite` (and optional d1/postgres guests)); hosts open the store via
the external guest RPC. This crate owns the greenfield schema
([`migrations`](migrations.md)), typed [`entities`](entities.md), and the store API. See
`docs/database.md`.

## Modules

### [`bookclerk_library`](bookclerk_library.md)

*11 modules*

### [`atomic_ops`](atomic_ops.md)

*1 enum, 1 module, 1 struct*

### [`atomic_ops::atomic_status`](atomic_ops/atomic_status.md)

*9 constants*

### [`atomic_txn`](atomic_txn.md)

*2 traits*

### [`backend_migrate`](backend_migrate.md)

*1 function, 2 structs*

### [`backup`](backup.md)

*15 structs, 4 constants, 4 enums, 8 functions*

### [`backup::capture`](backup/capture.md)

*1 function*

### [`backup::encode`](backup/encode.md)

*1 constant, 1 enum*

### [`backup::repository`](backup/repository.md)

*2 structs*

### [`backup::restore`](backup/restore.md)

*2 functions*

### [`backup::schema`](backup/schema.md)

*7 functions*

### [`backup::verify`](backup/verify.md)

*1 function*

### [`binding_schema`](binding_schema.md)

*2 functions*

### [`control_plane`](control_plane.md)

*10 functions, 2 enums, 4 structs, 8 constants*

### [`control_plane::branch_catalog`](control_plane/branch_catalog.md)

*11 functions, 2 enums, 3 structs, 6 constants*

### [`control_plane::clerks`](control_plane/clerks.md)

*1 struct, 3 functions*

### [`control_plane::cluster_links`](control_plane/cluster_links.md)

*13 functions, 3 structs, 4 constants*

### [`control_plane::deployments`](control_plane/deployments.md)

*1 constant, 15 functions, 2 structs, 3 enums*

### [`control_plane::documents`](control_plane/documents.md)

*2 enums, 2 functions, 4 structs, 6 constants*

### [`control_plane::enrollment`](control_plane/enrollment.md)

*1 enum, 17 functions, 9 constants, 9 structs*

### [`control_plane::identity`](control_plane/identity.md)

*1 constant, 1 enum, 2 structs, 9 functions*

### [`control_plane::instance_config`](control_plane/instance_config.md)

*2 enums, 4 constants, 6 structs, 9 functions*

### [`control_plane::instances`](control_plane/instances.md)

*2 structs, 9 functions*

### [`control_plane::job_routing`](control_plane/job_routing.md)

*11 functions, 2 constants, 4 structs*

### [`control_plane::leases`](control_plane/leases.md)

*3 structs, 5 constants, 9 functions*

### [`control_plane::live`](control_plane/live.md)

*1 enum, 1 struct, 1 type alias, 2 functions*

### [`control_plane::node_queue`](control_plane/node_queue.md)

*1 constant, 1 enum, 6 functions*

### [`control_plane::node_status`](control_plane/node_status.md)

*4 structs, 6 constants, 7 functions*

### [`control_plane::output`](control_plane/output.md)

*1 enum, 1 struct, 6 constants, 6 functions*

### [`control_plane::ownership`](control_plane/ownership.md)

*3 functions*

### [`control_plane::pending_joins`](control_plane/pending_joins.md)

*2 constants, 3 structs, 5 functions*

### [`control_plane::placement`](control_plane/placement.md)

*15 functions, 2 enums, 5 structs, 7 constants*

### [`control_plane::remote_catalog`](control_plane/remote_catalog.md)

*3 structs, 4 functions*

### [`control_plane::remote_consents`](control_plane/remote_consents.md)

*1 constant, 1 enum, 3 structs, 7 functions*

### [`control_plane::remote_results`](control_plane/remote_results.md)

*1 constant, 1 enum, 1 function*

### [`control_plane::remote_work`](control_plane/remote_work.md)

*3 constants, 4 enums, 4 structs, 9 functions*

### [`control_plane::requirements_sweep`](control_plane/requirements_sweep.md)

*1 function, 1 struct, 4 constants*

### [`control_plane::routing`](control_plane/routing.md)

*1 constant, 1 struct, 2 functions*

### [`control_plane::schemas`](control_plane/schemas.md)

*1 enum, 3 structs, 7 functions*

### [`control_plane::secret`](control_plane/secret.md)

*1 struct, 2 functions*

### [`control_plane::semaphore`](control_plane/semaphore.md)

*1 enum, 2 constants, 2 structs, 8 functions*

### [`control_plane::trust`](control_plane/trust.md)

*1 type alias, 12 functions, 3 enums*

### [`control_plane::uploads`](control_plane/uploads.md)

*12 functions, 2 enums, 3 constants, 5 structs*

### [`control_plane::versions`](control_plane/versions.md)

*1 enum, 11 functions, 2 constants, 3 structs*

### [`db_atomic`](db_atomic.md)

*4 functions*

### [`email`](email.md)

*3 functions*

### [`entities`](entities.md)

*60 modules*

### [`entities::account_links`](entities/account_links.md)

*3 enums, 6 structs*

### [`entities::accounts`](entities/accounts.md)

*3 enums, 6 structs*

### [`entities::bookclerk_receipts`](entities/bookclerk_receipts.md)

*3 enums, 6 structs*

### [`entities::books`](entities/books.md)

*3 enums, 6 structs*

### [`entities::branch_titles`](entities/branch_titles.md)

*3 enums, 6 structs*

### [`entities::claim_tickets`](entities/claim_tickets.md)

*3 enums, 6 structs*

### [`entities::cluster_identity`](entities/cluster_identity.md)

*3 enums, 6 structs*

### [`entities::cluster_leases`](entities/cluster_leases.md)

*3 enums, 6 structs*

### [`entities::cluster_links`](entities/cluster_links.md)

*3 enums, 6 structs*

### [`entities::configuration_audit`](entities/configuration_audit.md)

*3 enums, 6 structs*

### [`entities::configuration_changes`](entities/configuration_changes.md)

*3 enums, 6 structs*

### [`entities::configuration_documents`](entities/configuration_documents.md)

*3 enums, 6 structs*

### [`entities::domain_events`](entities/domain_events.md)

*3 enums, 6 structs*

### [`entities::embeddings`](entities/embeddings.md)

*3 enums, 6 structs*

### [`entities::encrypted_secrets`](entities/encrypted_secrets.md)

*3 enums, 6 structs*

### [`entities::event_deliveries`](entities/event_deliveries.md)

*3 enums, 6 structs*

### [`entities::event_outbox_stats`](entities/event_outbox_stats.md)

*3 enums, 6 structs*

### [`entities::event_subscriber_nodes`](entities/event_subscriber_nodes.md)

*3 enums, 6 structs*

### [`entities::ignored_titles`](entities/ignored_titles.md)

*3 enums, 6 structs*

### [`entities::job_secret_handoffs`](entities/job_secret_handoffs.md)

*3 enums, 6 structs*

### [`entities::job_temp_paths`](entities/job_temp_paths.md)

*3 enums, 6 structs*

### [`entities::jobs`](entities/jobs.md)

*3 enums, 6 structs*

### [`entities::listening_progress`](entities/listening_progress.md)

*3 enums, 6 structs*

### [`entities::node_credentials`](entities/node_credentials.md)

*3 enums, 6 structs*

### [`entities::node_enrollment_links`](entities/node_enrollment_links.md)

*3 enums, 6 structs*

### [`entities::node_status`](entities/node_status.md)

*3 enums, 6 structs*

### [`entities::node_upload_sessions`](entities/node_upload_sessions.md)

*3 enums, 6 structs*

### [`entities::nodes`](entities/nodes.md)

*3 enums, 6 structs*

### [`entities::oidc_auth_codes`](entities/oidc_auth_codes.md)

*3 enums, 6 structs*

### [`entities::oidc_clients`](entities/oidc_clients.md)

*3 enums, 6 structs*

### [`entities::oidc_refresh_tokens`](entities/oidc_refresh_tokens.md)

*3 enums, 6 structs*

### [`entities::oidc_rp_states`](entities/oidc_rp_states.md)

*3 enums, 6 structs*

### [`entities::operator_sessions`](entities/operator_sessions.md)

*3 enums, 6 structs*

### [`entities::pending_joins`](entities/pending_joins.md)

*3 enums, 6 structs*

### [`entities::plugin_artifacts`](entities/plugin_artifacts.md)

*3 enums, 6 structs*

### [`entities::plugin_deployment_observations`](entities/plugin_deployment_observations.md)

*3 enums, 6 structs*

### [`entities::plugin_deployments`](entities/plugin_deployments.md)

*3 enums, 6 structs*

### [`entities::plugin_installs`](entities/plugin_installs.md)

*3 enums, 6 structs*

### [`entities::plugin_instances`](entities/plugin_instances.md)

*3 enums, 6 structs*

### [`entities::plugin_kv`](entities/plugin_kv.md)

*3 enums, 6 structs*

### [`entities::plugin_placements`](entities/plugin_placements.md)

*3 enums, 6 structs*

### [`entities::plugin_schemas`](entities/plugin_schemas.md)

*3 enums, 6 structs*

### [`entities::plugin_versions`](entities/plugin_versions.md)

*3 enums, 6 structs*

### [`entities::portal_identities`](entities/portal_identities.md)

*3 enums, 6 structs*

### [`entities::portal_sessions`](entities/portal_sessions.md)

*3 enums, 6 structs*

### [`entities::remote_plugin_consents`](entities/remote_plugin_consents.md)

*3 enums, 6 structs*

### [`entities::saved_filters`](entities/saved_filters.md)

*3 enums, 6 structs*

### [`entities::security_audit_events`](entities/security_audit_events.md)

*3 enums, 6 structs*

### [`entities::storage_scan_generations`](entities/storage_scan_generations.md)

*3 enums, 6 structs*

### [`entities::storage_scan_rows`](entities/storage_scan_rows.md)

*3 enums, 6 structs*

### [`entities::title_files`](entities/title_files.md)

*3 enums, 6 structs*

### [`entities::title_request_sources`](entities/title_request_sources.md)

*3 enums, 6 structs*

### [`entities::title_requests`](entities/title_requests.md)

*3 enums, 6 structs*

### [`entities::user_invites`](entities/user_invites.md)

*3 enums, 6 structs*

### [`entities::user_preferences`](entities/user_preferences.md)

*3 enums, 6 structs*

### [`entities::users`](entities/users.md)

*3 enums, 6 structs*

### [`entities::webauthn_challenges`](entities/webauthn_challenges.md)

*3 enums, 6 structs*

### [`entities::webauthn_credentials`](entities/webauthn_credentials.md)

*3 enums, 6 structs*

### [`entities::work_editions`](entities/work_editions.md)

*3 enums, 6 structs*

### [`entities::works`](entities/works.md)

*3 enums, 6 structs*

### [`error`](error.md)

*1 enum, 1 type alias*

### [`host_schema`](host_schema.md)

*12 functions, 2 constants, 2 structs*

### [`in_process_atomic`](in_process_atomic.md)

*1 struct*

### [`master_key`](master_key.md)

*1 enum, 18 functions, 2 constants, 2 structs*

### [`migrations`](migrations.md)

*1 enum, 1 struct, 24 functions, 4 constants*

### [`migrations::engine`](migrations/engine.md)

*1 enum, 4 functions*

### [`migrations::plan`](migrations/plan.md)

*1 constant, 1 enum, 2 structs, 6 functions*

### [`migrations::plugin`](migrations/plugin.md)

*16 functions, 2 constants, 4 structs*

### [`models`](models.md)

*10 enums, 12 constants, 19 functions, 44 structs*

### [`operator_token`](operator_token.md)

*1 enum, 2 constants, 7 functions*

### [`password`](password.md)

*2 functions*

### [`schema_state`](schema_state.md)

*1 enum, 2 constants*

### [`schema_walk`](schema_walk.md)

*1 struct, 2 functions*

### [`scope`](scope.md)

*1 struct*

### [`secrets`](secrets.md)

*14 functions, 2 modules, 3 structs, 6 constants*

### [`secrets::secret_account_type`](secrets/secret_account_type.md)

*4 constants*

### [`secrets::secret_kind`](secrets/secret_kind.md)

*9 constants*

### [`session_client`](session_client.md)

*1 function, 1 struct*

### [`sql_plan`](sql_plan.md)

*1 module, 1 struct, 11 functions*

### [`sql_plan::exec`](sql_plan/exec.md)

*5 functions*

### [`sql_plan::host_ir`](sql_plan/host_ir.md)

*1 function, 1 struct*

### [`sql_plan::interpret`](sql_plan/interpret.md)

*1 function, 1 struct*

### [`sql_plan::named`](sql_plan/named.md)

*2 functions*

### [`sql_plan::reply`](sql_plan/reply.md)

*1 function*

### [`sql_plan::slots`](sql_plan/slots.md)

*1 constant, 2 functions*

### [`sql_plan::vectors_typed`](sql_plan/vectors_typed.md)

*4 functions*

### [`store`](store.md)

*10 structs, 3 functions*

### [`store::books_page`](store/books_page.md)

*1 struct, 2 constants*

### [`store::event_outbox`](store/event_outbox.md)

*1 enum, 4 functions*

### [`store::plugin_databases`](store/plugin_databases.md)

*1 struct*

### [`store::plugin_kv`](store/plugin_kv.md)

*1 constant, 1 enum, 3 structs*

### [`store::title_files`](store/title_files.md)

*1 constant, 1 struct*

### [`text`](text.md)

*6 functions*

### [`token_hash`](token_hash.md)

*1 constant, 4 functions*

### [`wishlist_merge`](wishlist_merge.md)

*2 functions*

