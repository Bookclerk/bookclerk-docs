# bookclerk_library::control_plane

Database-backed control plane: node identity and typed configuration.

Bootstrap material (database target, `master.key`, `node-identity.json`) is
read before these rows exist. After enrollment, `core.events` is the
authority for `[events]` and `core.output` for the shared `[output]`
table. Other `config.toml` sections stay transitional.

See `docs/adr/control-plane.md`.

| Item | Kind | Summary |
| --- | --- | --- |
| [`BranchTitle`](#branchtitle) | struct | One title a branch shared, as the owner's Settings → Branches lists it. |
| [`BranchTitleCounts`](#branchtitlecounts) | struct | Counts of one branch's shared titles. |
| [`CatalogSessionOpened`](#catalogsessionopened) | struct | An opened catalog session (`begin_catalog`). |
| [`LibraryContents`](#librarycontents) | struct | What library data an install holds of its own. |
| [`ClusterLink`](#clusterlink) | struct | One owner cluster this install enrolled into. |
| [`ClusterLinkKeys`](#clusterlinkkeys) | struct | The node keys of one cluster link: the Ed25519 signing seed and the X25519 box secret. Zeroized on drop; `Debug` shows only the public keys. |
| [`NewClusterLink`](#newclusterlink) | struct | What enrollment returned, to record with `record_cluster_link`. |
| [`DeploymentObservation`](#deploymentobservation) | struct | One node's observation of a deployment. |
| [`PluginDeployment`](#plugindeployment) | struct | One desired deployment row. |
| [`DocumentKey`](#documentkey) | struct | Address of one configuration document. |
| [`StoredDocument`](#storeddocument) | struct | One stored document before domain parsing. |
| [`VerifiedClusterOwner`](#verifiedclusterowner) | struct | Proof that a user was an active Owner when the write was authorized. |
| [`VerifiedNodeOwner`](#verifiednodeowner) | struct | Proof that a user owned a node when the write was authorized. |
| [`EnrolledNode`](#enrollednode) | struct | Result of a successful redeem. |
| [`EnrollmentLink`](#enrollmentlink) | struct | One link as listings show it. The secret and its hash are not included. |
| [`EnrollmentSecret`](#enrollmentsecret) | struct | A link's plaintext secret. Shown once at mint and never stored; `Debug` does not print it. |
| [`EnrollmentToken`](#enrollmenttoken) | struct | A parsed enrollment token. |
| [`HostEnrollmentSettings`](#hostenrollmentsettings) | struct | The host-enrollment switch as stored, or its default when no document was ever written. |
| [`HostEnrollmentSettingsV1`](#hostenrollmentsettingsv1) | struct | Typed `core.host_enrollment` document, schema version 1. |
| [`MintedLink`](#mintedlink) | struct | A freshly minted link and its secret, which is returned only here. |
| [`NodeCredential`](#nodecredential) | struct | A host node's stored credential. |
| [`RedeemRequest`](#redeemrequest) | struct | What a host submits to redeem a link. |
| [`NodeIdentity`](#nodeidentity) | struct | Bootstrap identity loaded from `NODE_IDENTITY_FILE`. |
| [`NodeRecord`](#noderecord) | struct | Durable node row plus the latest observation columns. |
| [`InstanceBindingGrant`](#instancebindinggrant) | struct | Whether the plugin-instance config grant includes `config` or `secrets`. |
| [`InstanceConfigDocument`](#instanceconfigdocument) | struct | A parsed instance document plus its revision. |
| [`InstanceSecretRefV1`](#instancesecretrefv1) | struct | A name in `encrypted_secrets`. The ciphertext is not part of the document. |
| [`PluginInstanceConfigV1`](#plugininstanceconfigv1) | struct | Schema-1 plugin instance configuration. |
| [`PrunedInstanceConfig`](#prunedinstanceconfig) | struct | One instance document rewritten to fit the pinned schema. |
| [`ResolvedInstanceBindings`](#resolvedinstancebindings) | struct | `CONFIG` and `SECRETS` payloads for one instance. |
| [`PluginInstance`](#plugininstance) | struct | One `plugin_instances` row. |
| [`PluginInstanceId`](#plugininstanceid) | struct | Stable id of one plugin instance. |
| [`JobClaimFilter`](#jobclaimfilter) | struct | What a Host is willing to claim: the job kinds it runs and the plugin keys its operator consented to. Both must admit a job. |
| [`JobClaimLimits`](#jobclaimlimits) | struct | Cluster-wide permit counts a routed claim enforces. |
| [`JobClaimRequest`](#jobclaimrequest) | struct | One routed claim attempt by one worker. |
| [`RoutedJobClaim`](#routedjobclaim) | struct | A claimed job and the semaphore permits it holds. |
| [`ClusterLease`](#clusterlease) | struct | A held cluster lease. |
| [`LeaseFence`](#leasefence) | struct | Proof, checked against the database, that this holder still holds a lease. Call `LeaseFence::check` before each destructive step. |
| [`SingletonDuty`](#singletonduty) | struct | One periodic duty that exactly one live node runs per interval. |
| [`LiveRevision`](#liverevision) | struct | One topic's revision: its row count and newest change stamp. |
| [`GuestReservation`](#guestreservation) | struct | CPU and memory one guest may use at most: its grant's ceilings after the node's `[plugins.jail]` limits. |
| [`NodeHeartbeat`](#nodeheartbeat) | struct | One heartbeat's measurements, before budget and accounting are applied. |
| [`NodeSampler`](#nodesampler) | struct | Samples capacity and measured use for heartbeats. Keeps the previous CPU counters so each sample reports utilization since the last one. |
| [`NodeStatus`](#nodestatus) | struct | A node's latest heartbeat with budget, accounting, and availability. |
| [`OutputSettingsV1`](#outputsettingsv1) | struct | Typed `core.output` document, schema version 1. |
| [`NewPendingJoin`](#newpendingjoin) | struct | What a joining daemon records when it starts a sign-in. |
| [`PendingJoin`](#pendingjoin) | struct | A pending join as stored (no secrets). |
| [`TakenJoin`](#takenjoin) | struct | A consumed pending join with its secrets. `Debug` prints no secret. |
| [`NodeCandidate`](#nodecandidate) | struct | One node as the planner sees it: identity, liveness, and the budget headroom its last heartbeat reported. |
| [`PlacementPlan`](#placementplan) | struct | What the planner decided for one instance. |
| [`PlacementSpec`](#placementspec) | struct | What an operator asks for: how many nodes, and which may qualify. |
| [`PluginPlacement`](#pluginplacement) | struct | A stored placement: the operator's intent plus the last pass's status. |
| [`ScheduleReport`](#schedulereport) | struct | What one scheduler pass changed. |
| [`RemoteDeploymentEntry`](#remotedeploymententry) | struct | One deployment a Host should run, with what it runs. |
| [`RemoteInstance`](#remoteinstance) | struct | A deployed instance a Host asks about, with the node's trust. |
| [`RemoteObservation`](#remoteobservation) | struct | One observation a Host reports. |
| [`RemoteApproval`](#remoteapproval) | struct | What an operator approves. |
| [`RemoteArtifactRef`](#remoteartifactref) | struct | The archive an owner asks this host to fetch: the version's artifact for this host's target. |
| [`RemotePluginConsent`](#remotepluginconsent) | struct | One owner plugin of one owner cluster, as this install sees it. |
| [`JobSecret`](#jobsecret) | struct | One unsealed secret handed with a job. The value is zeroized on drop and never shown by `Debug`. |
| [`JobSecretRef`](#jobsecretref) | struct | Names one store-account secret a job is handed. Carries no secret material. |
| [`RemoteClaimSettings`](#remoteclaimsettings) | struct | What a Host's claim settles on: the routing and lease settings of the owner node serving it. |
| [`RemoteJobClaim`](#remotejobclaim) | struct | A job claimed for a Host, with the secrets to seal for it. |
| [`RequirementsSweepReport`](#requirementssweepreport) | struct | What one sweep did. |
| [`RouteRank`](#routerank) | struct | This node's position among the live nodes running one instance. |
| [`PinnedPluginSchema`](#pinnedpluginschema) | struct | One `plugin_schemas` row. |
| [`SchemaPin`](#schemapin) | struct | Inputs for `pin_plugin_schema`. |
| [`SchemaPinOutcome`](#schemapinoutcome) | struct | Outcome of `pin_plugin_schema`: the row that is now pinned and how it relates to the local manifest. |
| [`ClusterSecret`](#clustersecret) | struct | Cluster id and fingerprint of the secret root aligned for this process. |
| [`SemaphorePermit`](#semaphorepermit) | struct | One held permit: a slot lease of a semaphore. |
| [`SemaphoreRequest`](#semaphorerequest) | struct | One semaphore a caller wants a permit of. |
| [`TitleNames`](#titlenames) | struct | The names a title's acquire writes on one destination, as the owner plans them, all without their extension. |
| [`UploadDestination`](#uploaddestination) | struct | One output destination of the receiving owner node an upload is granted for, with where the title may be written there. |
| [`UploadFinish`](#uploadfinish) | struct | How the job's closing upload completes it. |
| [`UploadGrantRequest`](#uploadgrantrequest) | struct | What a Host asks to upload for one title. |
| [`UploadSession`](#uploadsession) | struct | One open upload, as the upload endpoint sees it. |
| [`PluginArtifact`](#pluginartifact) | struct | One downloadable artifact of a version. |
| [`PluginInstall`](#plugininstall) | struct | The active version pointer of one plugin key. |
| [`PluginVersion`](#pluginversion) | struct | One installed version of a plugin. |
| [`EventsSettingsV1`](#eventssettingsv1) | struct | Typed `[events]` document, schema version 1. |
| [`NodeRuntimeSettingsV1`](#noderuntimesettingsv1) | struct | Typed node-scoped runtime document, schema version 1. |
| [`ConfigurationDocument`](#configurationdocument) | struct | A parsed domain document plus its revision metadata. |
| [`ControlPlaneSession`](#controlplanesession) | struct | Identity, secret root, and the authoritative events document for this process. |
| [`CatalogAccept`](#catalogaccept) | enum | Outcome of `accept_catalog_batch`. |
| [`CatalogBegin`](#catalogbegin) | enum | Outcome of `begin_catalog`. |
| [`DeploymentReplace`](#deploymentreplace) | enum | Outcome of `replace_plugin_deployment`. |
| [`DeploymentStatus`](#deploymentstatus) | enum | Observed deployment status. |
| [`DesiredState`](#desiredstate) | enum | What a deployment row asks its node to do, and who decided it. |
| [`ConfigActor`](#configactor) | enum | Who is allowed to write a configuration document. |
| [`ReplaceOutcome`](#replaceoutcome) | enum | Result of a compare-and-swap replacement. |
| [`EnrollmentActor`](#enrollmentactor) | enum | Who is acting on enrollment links and node credentials. |
| [`NodeKind`](#nodekind) | enum | How a node joined the cluster. |
| [`InstanceConfigReplace`](#instanceconfigreplace) | enum | Outcome of `replace_instance_config`. |
| [`SettingValue`](#settingvalue) | enum | One scalar setting. Nested objects and arrays are not representable. |
| [`LiveTopic`](#livetopic) | enum | A topic of the live socket. |
| [`NodeMessage`](#nodemessage) | enum | One message on a `node.<node_id>` queue. The body is the JSON of this enum (`{"type": "deployment.changed", ...}`). |
| [`OutputReplace`](#outputreplace) | enum | Outcome of `replace_output`. |
| [`PlacementWrite`](#placementwrite) | enum | Outcome of `put_plugin_placement`. |
| [`Replicas`](#replicas) | enum | How many nodes should run a placed instance. |
| [`RemoteApprover`](#remoteapprover) | enum | Who gives an approval. |
| [`ScanAccept`](#scanaccept) | enum | Outcome of `accept_scan_results`. |
| [`RemoteFinish`](#remotefinish) | enum | Outcome of `finish_remote_job`. |
| [`RemoteOutcome`](#remoteoutcome) | enum | How a Host's job ended. |
| [`RemoteRenew`](#remoterenew) | enum | Outcome of `renew_remote_job`. |
| [`WriteBack`](#writeback) | enum | Outcome of `write_back_job_secret`. |
| [`SchemaPinChange`](#schemapinchange) | enum | How `pin_plugin_schema` changed the row. |
| [`PermitAcquire`](#permitacquire) | enum | Outcome of `acquire_permits`. |
| [`Claimant`](#claimant) | enum | How the claiming node reaches the job queue. |
| [`NodeTrust`](#nodetrust) | enum | What a node is trusted with, resolved from the database. |
| [`ScanRequester`](#scanrequester) | enum | Who asked for a scan. |
| [`UploadCommit`](#uploadcommit) | enum | Outcome of `commit_upload`. |
| [`UploadGrant`](#uploadgrant) | enum | Outcome of `grant_upload`. |
| [`AddVersion`](#addversion) | enum | What `add_plugin_version` did. |
| [`EventsReplace`](#eventsreplace) | enum | Outcome of `replace_events`. |
| [`NodeRuntimeReplace`](#noderuntimereplace) | enum | Outcome of `replace_node_runtime`. |
| [`accept_catalog_batch`](#accept_catalog_batch) | function | Records one batch of `node_id`'s shared titles under `fence`, then the cursor after it. Every title is checked before any is written; a title this library already holds for the branch's owner is skipped. With `done` (the last batch of a pass) while a pass from the start is under way, titles the pass did not name are withdrawn (detached); `done` on a resumed incremental pass changes nothing more. |
| [`begin_catalog`](#begin_catalog) | function | Opens (or reopens) `node_id`'s catalog sync with the settings the branch reports: records them on its node row, bumps its catalog fence (frames under an older fence are lost from now on), and answers where to resume. A change of either setting restarts the pass from the start, so titles the new setting excludes are withdrawn when the pass ends; so does a session `FULL_CATALOG_PASS_EVERY` after the last full pass. With sharing off, every title the branch shared is marked detached (`stopped_sharing`). |
| [`branch_provenance`](#branch_provenance) | function | The provenance text of titles shared by `node_id`. |
| [`branch_title_counts`](#branch_title_counts) | function | `BranchTitleCounts` of `node_id`. |
| [`catalog_fence_live`](#catalog_fence_live) | function | True while `fence` is `node_id`'s live catalog fence and it may share. |
| [`detach_branch_titles`](#detach_branch_titles) | function | Marks every title `node_id` shares detached with `reason`; answers how many changed. Files already delivered stay where they are. |
| [`forwardable_branch_titles`](#forwardable_branch_titles) | function | On a branch: the titles its own branches shared with it that it may pass on (still shared, files delivered), oldest first. |
| [`grant_catalog_upload`](#grant_catalog_upload) | function | Opens an upload of one shared title's files, bound to `node_id`'s catalog `fence` and received by `receiving_node_id`. |
| [`list_branch_titles`](#list_branch_titles) | function | Titles branches shared, of one branch or all, oldest first, at most `limit`. |
| [`pending_deliveries`](#pending_deliveries) | function | Up to `limit` of `node_id`'s shared titles whose files have not arrived (or changed since). |
| [`remove_detached_branch_titles`](#remove_detached_branch_titles) | function | Removes detached entries ("from a branch that left" and the like), of one branch or of every one; answers how many. Only the library entries go: files already delivered stay in this library's outputs. |
| [`library_contents`](#library_contents) | function | Counts the library data this install holds of its own. |
| [`owner_nodes_visible_to`](#owner_nodes_visible_to) | function | The owner nodes (`kind = owner`: this one and its clerks) `actor` may see, by the rule `remove_owner_node` removes them: the operator or an active Owner sees every one; another active user only the ones they own; an unknown or disabled user none. |
| [`remove_owner_node`](#remove_owner_node) | function | Removes owner node `node_id` from the library's node list, in one transaction: its `nodes` row (and with it its labels and credentials), status, event-subscriber registrations, secret handoffs, and plugin deployments with their observations. Only a node that is not running may go, and never `this_node`. The liveness check and the removal are one step: the node's status row is deleted only if it still holds the stale heartbeat that was checked, so a heartbeat that lands in between refuses the removal. A node that was only cut off from the database for longer than its liveness window can still be removed while it runs; its next heartbeat writes a status row again, but not its node row. Removing a node does not take back the database credentials or DEK it held; rotate those when it is not trusted any more. |
| [`decode_spki_pin`](#decode_spki_pin) | function | Decodes an SPKI pin: base64url (no padding) of a SHA-256 digest. |
| [`join_cycle`](#join_cycle) | function | This library's own cluster id when `parent_ancestry` names it: joining that library would make a cycle (A ← B ← A, A ← B ← C ← A). `None` when it does not, or this install has no cluster identity yet. |
| [`join_overflows_ancestry`](#join_overflows_ancestry) | function | True when joining a library that reports `parent_ancestry` would give this library more ancestors (itself and every library above it) than `bookclerk_node_proto::MAX_ANCESTRY`: `library_ancestry` would then leave the farthest out, and a library joining this one later could no longer be checked for a loop through them. An ancestry cut at that limit by the library itself always holds exactly the limit, so this refuses it too. Such a join is refused rather than cut short. |
| [`leave_cluster_link`](#leave_cluster_link) | function | Leaves an owner cluster on this side: marks the link `CLUSTER_LINK_LEFT` and deletes its sealed node keys, so this install can never sign as that node again. Answers the link as it now stands, or `None` when there is no such link. |
| [`library_ancestry`](#library_ancestry) | function | This library's ancestry (what `GET /api/node/v1/info` reports): its own cluster id, then for each active link the owner's cluster id and the ancestry that owner reported, without repeats, at most `bookclerk_node_proto::MAX_ANCESTRY` ids. Empty when this install has no cluster identity yet. A join that would take it past that limit is refused (`join_overflows_ancestry`); should a library above grow its ancestry later, the list is cut at exactly the limit, which in turn refuses every join under it. |
| [`list_cluster_links`](#list_cluster_links) | function | Every cluster link, oldest enrollment first. |
| [`load_cluster_link`](#load_cluster_link) | function | One cluster link. |
| [`load_cluster_link_keys`](#load_cluster_link_keys) | function | Unseals the node keys of an active `link`. |
| [`mark_cluster_link_revoked`](#mark_cluster_link_revoked) | function | Marks an **active** link still enrolled as `remote_node_id` `CLUSTER_LINK_REVOKED` (the owner closed with 4403 on that node's connection). A link this install already left stays `left`, and one a later join enrolled as another node keeps its new enrollment. |
| [`record_cluster_link`](#record_cluster_link) | function | Seals `keys` and records the link, replacing an earlier link to the same cluster (a re-enrollment) as `active`. |
| [`set_cluster_link_ancestry`](#set_cluster_link_ancestry) | function | Records the owner library's ancestry as it reported it (`info` at join, `catalog.session` on every sync). Ids that are not plain cluster ids are refused; at most `bookclerk_node_proto::MAX_ANCESTRY` are kept. |
| [`set_cluster_link_sharing`](#set_cluster_link_sharing) | function | Sets a link's two sharing settings: "Share this branch's books with *library*" and "Include books from my own branches". Answers false when no such link exists. The running catalog sync reports them to the owner on its next pass. |
| [`set_cluster_link_state`](#set_cluster_link_state) | function | Sets a link's state. Answers false when no such link exists. |
| [`bounded_observation_detail`](#bounded_observation_detail) | function | Truncates detail to 512 bytes and drops ASCII control characters. |
| [`delete_plugin_deployment`](#delete_plugin_deployment) | function | Deletes an absent deployment and its observations. |
| [`ensure_plugin_deployment`](#ensure_plugin_deployment) | function | Inserts `(instance, node, desired)` when that pair has no row. |
| [`instance_has_deployment`](#instance_has_deployment) | function | True when any deployment row names `instance_id`. |
| [`list_all_deployments`](#list_all_deployments) | function | Every deployment row in the cluster. |
| [`list_deployments_for_instance`](#list_deployments_for_instance) | function | Every deployment row for `instance_id`, on any node. |
| [`list_deployments_for_node`](#list_deployments_for_node) | function | Every deployment row targeting `node_id`, present or absent. |
| [`list_observations_for_deployment`](#list_observations_for_deployment) | function | Every observation row for `deployment_id`, from any node. |
| [`list_present_deployments_for_node`](#list_present_deployments_for_node) | function | Present deployments whose `node_id` is `node_id`. |
| [`load_deployment`](#load_deployment) | function | Loads one deployment. |
| [`load_deployment_for_node`](#load_deployment_for_node) | function | The deployment of `instance_id` on `node_id`, if any. |
| [`load_observation`](#load_observation) | function | Loads the observation for one deployment on one node. |
| [`present_plugin_keys_for_node`](#present_plugin_keys_for_node) | function | Canonical plugin keys with a present deployment on `node_id`. |
| [`replace_plugin_deployment`](#replace_plugin_deployment) | function | Replaces desired state when `expected_revision` is current. |
| [`upsert_observation`](#upsert_observation) | function | Upserts one local observation. |
| [`audit_count`](#audit_count) | function | Counts audit rows for one namespace. |
| [`change_count`](#change_count) | function | Counts committed change notices for one namespace. |
| [`authorized_enrollment_link`](#authorized_enrollment_link) | function | The link a branch or clerk sign-in redeems through: its id and secret are derived from the authorization code and the PKCE verifier, so a retry of the same exchange (same code, verifier and keys) finds the same link and answers from its receipt, and a wrong verifier can never redeem it. |
| [`check_redeem_request`](#check_redeem_request) | function | Checks what a redeem would refuse as malformed before anything is written: its keys (canonical Curve25519 points) and `software_version`. A sign-in's redeem runs this before it spends its code, so a malformed request leaves the code for a corrected retry. |
| [`format_enrollment_token`](#format_enrollment_token) | function | Formats `bcn1.<link_id>.<secret>[.<spki_pin>]`. |
| [`list_enrollment_links`](#list_enrollment_links) | function | Links `actor` may see, newest first: every link for the operator or an Owner, the caller's own links for anyone else. |
| [`load_host_enrollment`](#load_host_enrollment) | function | Reads the cluster's host-enrollment switch. A missing document means the default: enabled. |
| [`load_node_credential`](#load_node_credential) | function | Reads a host node's credential. |
| [`mint_enrollment_link`](#mint_enrollment_link) | function | Mints a link that enrolls one host node for `for_user_id`. |
| [`parse_enrollment_token`](#parse_enrollment_token) | function | Parses a token, or a whole link whose fragment is the token. |
| [`record_authorized_enrollment_link`](#record_authorized_enrollment_link) | function | Records the link an approved branch sign-in redeems through (`authorized_enrollment_link`), for `for_user_id`, the user who signed in and consented. Called only after the authorization code's client, redirect and PKCE verifier were checked, before the code is consumed, so a failed write leaves the code usable; the redeem itself is `redeem_enrollment_link`, the same canonical batch a minted link takes. |
| [`record_node_connected`](#record_node_connected) | function | Stamps `last_connected_at` after a host node authenticates. Returns false (and stamps nothing) when the credential is missing or revoked, so a revoke that lands mid-handshake still refuses the connection. |
| [`record_node_left`](#record_node_left) | function | A host announced it is leaving (`node.leaving`): revokes its credential at once and drops its live status, so it stops counting as a node without waiting for missed heartbeats, and tells every node so the connection is closed wherever it is held. Answers false when the node has no unrevoked credential; for one already revoked, the cleanup is still finished, so a retried leave completes one that failed part way. |
| [`redeem_enrollment_link`](#redeem_enrollment_link) | function | Redeems a link: one canonical batch marks it redeemed, inserts a host `nodes` row owned by the link's user, inserts its `node_credentials`, and records a receipt. |
| [`redeem_enrollment_link_as`](#redeem_enrollment_link_as) | function | `redeem_enrollment_link` for a node of `kind`: the same canonical batch, recording `kind` on the `nodes` row. |
| [`revoke_enrollment_link`](#revoke_enrollment_link) | function | Revokes an unredeemed link. Returns the link as stored afterwards. |
| [`revoke_node_credential`](#revoke_node_credential) | function | Revokes a host node's credential and tells every node, so the node holding that host's connection can close it. |
| [`set_host_enrollment_enabled`](#set_host_enrollment_enabled) | function | Turns host enrollment on or off for the cluster. |
| [`verify_cluster_owner`](#verify_cluster_owner) | function | Returns a `ConfigActor::ClusterOwner` when `user_id` is an active Owner. |
| [`current_process_incarnation`](#current_process_incarnation) | function | This process's incarnation. Stable until the process exits. |
| [`encode_node_identity_file`](#encode_node_identity_file) | function | The `NODE_IDENTITY_FILE` contents `install_node_identity` writes, for a caller that stages the file itself (a clerk applying its bootstrap as one recoverable step) with mode `0600`. |
| [`install_node_identity`](#install_node_identity) | function | Writes the identity file for a node id the cluster already recorded, bound to `cluster_id`, replacing any file there atomically. A clerk joining a library takes the node id the library minted for it at redeem, so its first heartbeat lands on that `nodes` row. |
| [`list_node_ids`](#list_node_ids) | function | Lists node ids, sorted. |
| [`list_nodes`](#list_nodes) | function | Every node row, sorted by id. |
| [`load_node`](#load_node) | function | Loads one node row. |
| [`load_or_create_node_identity`](#load_or_create_node_identity) | function | Loads the identity file or creates a new node id. |
| [`node_identity_path`](#node_identity_path) | function | Path of the node identity file. |
| [`node_target`](#node_target) | function | Bookclerk artifact target this binary runs as (`linux-x64-gnu`, `macos-arm64`, …). Matches `bookclerk_plugin_catalog::host_bookclerk_target` (a plugin-host test pins the two together). |
| [`import_instance_config_if_absent`](#import_instance_config_if_absent) | function | Inserts the document when it is absent. |
| [`load_instance_config`](#load_instance_config) | function | Loads and parses the instance config document. |
| [`parse_setting_text`](#parse_setting_text) | function | Parses an operator-typed value for `key` according to the schema. |
| [`reconcile_instance_configs_to_schema`](#reconcile_instance_configs_to_schema) | function | Rewrites every instance document of `plugin_key` to what `schema` still accepts: settings whose key is gone, became a secret, or whose value no longer fits (a type or enum change), and secret refs whose key is gone or became plain, are dropped. |
| [`replace_instance_config`](#replace_instance_config) | function | Replaces the instance document when `expected_revision` is current. |
| [`resolve_instance_bindings`](#resolve_instance_bindings) | function | Resolves `CONFIG` and `SECRETS` for `instance_id` against `schema`. |
| [`seal_instance_secret`](#seal_instance_secret) | function | Seals one instance secret with the cluster DEK. |
| [`set_secret_ref`](#set_secret_ref) | function | Adds a secret ref for `key` whose sealed row is named `key`, replacing an existing ref for the same key. |
| [`set_setting`](#set_setting) | function | Sets `key` to `value` in `settings`, or removes it when `value` is `None`. |
| [`create_node_local_instance`](#create_node_local_instance) | function | Inserts a node-local instance of `plugin_key` owned by `node_id` (H-2): it adds to the cluster's outputs on that node only, and a deployment of it to any other node, or a placement, is refused. |
| [`create_plugin_instance`](#create_plugin_instance) | function | Inserts a new instance for `plugin_key`. |
| [`default_instance_for_key`](#default_instance_for_key) | function | The instance a host routes to when a job does not pin one: the `plugin_instance_defaults` claim for `plugin_key`, else the oldest row. |
| [`delete_plugin_instance`](#delete_plugin_instance) | function | Deletes an instance that has no deployments. Its configuration document, audit rows, and sealed secrets are left for retention cleanup. |
| [`ensure_plugin_instance`](#ensure_plugin_instance) | function | Returns the existing instance for `plugin_key`, or inserts one. |
| [`find_plugin_instance_by_key`](#find_plugin_instance_by_key) | function | Earliest instance row for `plugin_key`, if any. |
| [`list_all_plugin_instances`](#list_all_plugin_instances) | function | Every instance row, ordered by plugin key then creation time. |
| [`list_plugin_instances_for_key`](#list_plugin_instances_for_key) | function | Every instance row for `plugin_key`, oldest first. |
| [`load_plugin_instance`](#load_plugin_instance) | function | Loads one instance by id. |
| [`account_semaphore`](#account_semaphore) | function | Name of the cluster semaphore for one store account. |
| [`claim_routed_job`](#claim_routed_job) | function | Claims the best ready job this node can run, under the cluster semaphores. |
| [`class_semaphore`](#class_semaphore) | function | Name of the cluster semaphore for one resource class. |
| [`common_deployment_nodes`](#common_deployment_nodes) | function | Nodes where every instance in `requires` has a deployment that should run. |
| [`instance_semaphore`](#instance_semaphore) | function | Name of the cluster semaphore for one plugin instance. |
| [`job_permit_holder_prefix`](#job_permit_holder_prefix) | function | The holder prefix of the semaphore permits a routed claim of `job_id` by `owner` takes when the claim runs at lease `generation`: `<owner>/<job_id>/g<generation>/`. |
| [`job_plugin_keys`](#job_plugin_keys) | function | The plugin keys `job` runs, or `None` when the job alone does not say. |
| [`job_semaphores`](#job_semaphores) | function | The cluster semaphores `job` counts against under `limits`. |
| [`job_source_plugin_key`](#job_source_plugin_key) | function | The one plugin whose source guest runs `job`, or `None` when the job alone does not name exactly one. |
| [`node_can_run_job`](#node_can_run_job) | function | True when `node_id` can run a job that requires `requires`. |
| [`resolve_job_requirements`](#resolve_job_requirements) | function | The instances a new job must find on its claiming node. |
| [`acquire_lease`](#acquire_lease) | function | Acquires or renews `name` for `holder` for `ttl`. |
| [`acquire_lease_on`](#acquire_lease_on) | function | `acquire_lease` on a bare connection (schema migration runs before a `LibraryStore` exists). |
| [`list_leases`](#list_leases) | function | Every lease row, held or expired, by name. |
| [`load_lease`](#load_lease) | function | The current row of lease `name`, held or expired. |
| [`load_lease_on`](#load_lease_on) | function | `load_lease` on a bare connection. |
| [`release_lease`](#release_lease) | function | Gives up a held lease so another holder may take it at once. |
| [`release_lease_on`](#release_lease_on) | function | `release_lease` on a bare connection. |
| [`renew_lease`](#renew_lease) | function | Fencing check: extends a lease this holder still holds. |
| [`renew_lease_on`](#renew_lease_on) | function | `renew_lease` on a bare connection. |
| [`changed_topics`](#changed_topics) | function | The topics whose revision differs between `before` and `after`. |
| [`live_revisions`](#live_revisions) | function | Reads every topic's revision. |
| [`is_node_consumer_id`](#is_node_consumer_id) | function | True when a catalog / delivery `plugin_id` is a node consumer rather than a plugin. |
| [`is_valid_node_id`](#is_valid_node_id) | function | True when `node_id` can name a node: `node.<node_id>` is a valid queue name. Minted ids are lowercase UUIDs, which always qualify. |
| [`node_consumer_id`](#node_consumer_id) | function | Catalog id of a node's own consumer (`node:<node_id>`). |
| [`node_queue_name`](#node_queue_name) | function | Queue a node consumes: `node.<node_id>`. Registration accepts only node ids for which this is a valid queue name (`is_valid_node_id`). |
| [`publish_node_message`](#publish_node_message) | function | Publishes `message` onto `node.<node_id>`. |
| [`publish_to_all_nodes`](#publish_to_all_nodes) | function | Publishes `message` onto every registered node's queue. |
| [`instance_demand`](#instance_demand) | function | What one more guest of `instance_id` is expected to need: the largest ceiling any node recorded for it, or `None` when no node ran it yet. |
| [`list_node_statuses`](#list_node_statuses) | function | Every node's latest heartbeat, by node id. |
| [`load_node_status`](#load_node_status) | function | One node's latest heartbeat, if its daemon ever ran. |
| [`next_availability`](#next_availability) | function | Next availability average (parts per million) after `gap_ms` since the previous heartbeat. |
| [`node_reservations`](#node_reservations) | function | Sum of the ceilings of the guests running on `node_id`, and how many run. A running guest with no recorded ceiling counts as the grant defaults. |
| [`record_deployment_reservation`](#record_deployment_reservation) | function | Records the ceilings of the guest a node just started for a deployment. |
| [`record_node_heartbeat`](#record_node_heartbeat) | function | Records this node's heartbeat: its measurements, its budget from `node.runtime`, the reservations of its running guests, and the updated availability. |
| [`import_output_if_absent`](#import_output_if_absent) | function | Inserts `[output]` when the cluster document is absent. |
| [`is_output_setting_key`](#is_output_setting_key) | function | True when `key` is a `core.output` setting or alias. |
| [`load_output`](#load_output) | function | Loads the cluster output document. |
| [`overlay_output`](#overlay_output) | function | Copies the authoritative output document onto `config` for `cluster_id`. |
| [`portable_output_seed`](#portable_output_seed) | function | Copies `[output]` with a `widevine_cdm` under `files_dir` made relative again. |
| [`replace_output`](#replace_output) | function | Replaces the cluster output document. |
| [`nodes_owned_by`](#nodes_owned_by) | function | Nodes `user_id` owns, sorted by id. |
| [`set_node_owner`](#set_node_owner) | function | Assigns `node_id` to `owner` (a user id), or clears its owner. |
| [`verify_node_owner`](#verify_node_owner) | function | Returns a `ConfigActor::NodeOwner` for `user_id` on `node_id` when the node's stored owner is that user. |
| [`create_pending_join`](#create_pending_join) | function | Records a sign-in in progress: seals `verifier` and `keys` and writes the row under `hash_token(state)`. Expired rows are purged first. |
| [`finish_pending_join`](#finish_pending_join) | function | Ends the pending join for `state`: deletes its row and sealed material. |
| [`open_pending_join`](#open_pending_join) | function | Reads the pending join for `state` without consuming it: its facts and unsealed secrets, so a redeem that fails for a reason a retry can fix (the library unreachable, or its answer lost) can be tried again with the same keys. `None` for an unknown, finished or expired state. |
| [`purge_expired_pending_joins`](#purge_expired_pending_joins) | function | Deletes every expired pending join and its sealed material. Answers how many rows went. |
| [`take_pending_join`](#take_pending_join) | function | Consumes the pending join for `state`: answers its facts and secrets once and deletes both rows. `None` for an unknown, already consumed, or expired state. |
| [`undeploy_instance`](#undeploy_instance) | function | Stops `instance_id` everywhere: drops its placement (handing its rows back) and sets every running deployment `absent`. Returns how many deployments were stopped. Uninstall runs this for each instance before it clears the active version. |
| [`delete_plugin_placement`](#delete_plugin_placement) | function | Deletes an instance's placement and hands its deployments back to the operator: each `placed` row becomes `present` and each `unplaced` row `absent`, so every node keeps doing what it does now. Returns false when there was no placement. |
| [`effective_node_labels`](#effective_node_labels) | function | A node's operator labels plus `node=<id>` and `target=<target>`. |
| [`ensure_hand_managed`](#ensure_hand_managed) | function | Fails when the scheduler owns `instance_id`'s deployments, so a hand-written enable or disable is not silently undone on its next pass. |
| [`labels_satisfy`](#labels_satisfy) | function | True when `labels` satisfy every constraint. |
| [`list_plugin_placements`](#list_plugin_placements) | function | Every placement, by instance id. |
| [`live_node_ids`](#live_node_ids) | function | Nodes whose daemon is live: its last heartbeat in `node_status` is younger than three heartbeat intervals. |
| [`load_plugin_placement`](#load_plugin_placement) | function | The placement of `instance_id`, if it has one. |
| [`plan_placement`](#plan_placement) | function | Plans one instance. |
| [`put_plugin_placement`](#put_plugin_placement) | function | Writes placement intent for an instance. |
| [`schedule_placements`](#schedule_placements) | function | One scheduler pass: plans every placement against the nodes' last heartbeats and writes the deployments the plans call for. Call it only while holding the `SCHEDULER_LEASE`. |
| [`set_node_labels`](#set_node_labels) | function | Replaces a node's operator labels and wakes the scheduler. |
| [`validate_constraint`](#validate_constraint) | function | Checks a constraint: a label, optionally prefixed with `!` to exclude it. |
| [`validate_label`](#validate_label) | function | Checks the label grammar: `name` or `name=value`, lowercase letters, digits, and `._/-`, the name starting with a letter or digit, at most 63 bytes. |
| [`validate_node_label`](#validate_node_label) | function | Checks an operator label: the grammar of `validate_label`, and not one of the implicit names every node already carries. |
| [`artifact_served_to_node`](#artifact_served_to_node) | function | True when the archive `archive_sha256` is stored in the library and is an artifact of the active version of a plugin deployed to `node_id`. |
| [`record_remote_observation`](#record_remote_observation) | function | Records a Host's observation of one of its deployments. Answers `false` (and writes nothing) for a deployment that does not target the node, a status a Host may not report, or a node trusted with nothing. |
| [`remote_deployments_for_node`](#remote_deployments_for_node) | function | The deployments that target `node_id` and should run, in deployment-id order. Empty for a node trusted with nothing. |
| [`remote_instance_for_node`](#remote_instance_for_node) | function | `instance_id` when it has a deployment on `node_id` that should run and the node is trusted with something; `None` otherwise (an unknown instance and one deployed elsewhere read the same). |
| [`approve_remote_plugin`](#approve_remote_plugin) | function | Stores an approval, but only when the recorded request is still the one the operator reviewed: a request that changed in between (a new version, a wider grant, another archive) answers `LibraryError::Conflict` and nothing is written. An automatic approval (`RemoteApprover::Platform`) is likewise refused while a revoke is recorded (an operator approval clears it) and over an operator's approval, which it would replace along with its narrowing. |
| [`approved_remote_payload_digests`](#approved_remote_payload_digests) | function | Payload-root digests of every owner tree an operator approval still covers, which a sweep of the shared version cache must keep: owner versions have no local deployment row to reference them. |
| [`list_remote_plugin_consents`](#list_remote_plugin_consents) | function | Every consent row, of one cluster when `cluster_id` is given, by cluster then plugin key. |
| [`load_remote_plugin_consent`](#load_remote_plugin_consent) | function | The consent row for `plugin_key` of `cluster_id`. |
| [`record_remote_plugin_request`](#record_remote_plugin_request) | function | Records what the owner asks to run here: `version` of `plugin_key` with the owner's approved grant, fetched from `artifact`. A stored approval is kept; whether it still covers the new request is decided when the grant is evaluated (and an approval of another archive never does, `RemotePluginConsent::approved_artifact_matches`). Writes nothing when the request is unchanged. |
| [`remote_request_digest`](#remote_request_digest) | function | The digest an operator approves a request by: SHA-256 (lowercase hex) over the requested version, the owner's grant exactly as recorded and the archive's URL, digest and payload-root digest, each length-prefixed, so any change to any of them gives another digest. |
| [`revoke_remote_plugin`](#revoke_remote_plugin) | function | Clears the approval of `plugin_key` of `cluster_id`, keeping its request, and records the revoke so no automatic approval brings it back. Answers the row, or `None` when nothing is recorded. |
| [`accept_scan_results`](#accept_scan_results) | function | Validates one batch of a Host's scan rows and writes it to the library. |
| [`claim_remote_job`](#claim_remote_job) | function | Claims the best job `node_id` may run and unseals its secrets. |
| [`finish_remote_job`](#finish_remote_job) | function | Completes or fails the job `node_id` holds at `generation`, then gives its semaphore permits back. |
| [`job_secret_refs`](#job_secret_refs) | function | The store-account secrets `job` needs on a Host. |
| [`live_remote_job`](#live_remote_job) | function | The job `node_id` holds at `generation`, while that fence is the job's live lease and the node is still trusted with the job; `None` when it is lost. |
| [`release_remote_job`](#release_remote_job) | function | Gives back the job `node_id` holds at `generation` without running it: the job returns to `pending`, and its semaphore permits and handoff rows are released. |
| [`remote_job_fence`](#remote_job_fence) | function | The fence a Host holds for `job_id` at lease generation `generation`. |
| [`remote_lease_owner`](#remote_lease_owner) | function | Lease owner of every job `node_id` runs over the node API. |
| [`renew_remote_job`](#renew_remote_job) | function | Extends the lease of the job `node_id` holds at `generation`, and its semaphore permits, recording `progress` when given. |
| [`write_back_job_secret`](#write_back_job_secret) | function | Stores a refreshed store token a Host writes back. |
| [`sweep_job_requirements`](#sweep_job_requirements) | function | Re-resolves the requirements of every pending routed job (see the module docs for the rules). |
| [`rank_nodes`](#rank_nodes) | function | Ranks `running` nodes for `demand`, best first (most room, then node id). |
| [`route_ranks`](#route_ranks) | function | For each claim id that is a plugin instance id, `node_id`'s rank among the live nodes whose guest of that instance is up (`running` or `healthy`). Ids that are not instance ids (plugin keys, aliases, node consumers) are left out: they are not routed. A node missing from the list (no heartbeat yet) ranks after every live node. |
| [`list_plugin_schemas`](#list_plugin_schemas) | function | Every pinned schema, ordered by plugin key. |
| [`load_plugin_schema`](#load_plugin_schema) | function | The pinned schema for `plugin_key`, if any node has pinned one. |
| [`pin_plugin_schema`](#pin_plugin_schema) | function | Pins `pin` according to the version rule in the module docs. |
| [`pin_version_schema`](#pin_version_schema) | function | Pins the schema of the plugin's active release. |
| [`require_plugin_schema`](#require_plugin_schema) | function | Loads the pinned schema or fails closed with an operator-facing message. |
| [`schema_fingerprint`](#schema_fingerprint) | function | Hex SHA-256 of the schema's canonical JSON. Two manifests that declare the same settings fingerprint the same, whatever their bytes. |
| [`stale_against_pin`](#stale_against_pin) | function | Whether the pinned row for `plugin_key` is newer than `local_version`. |
| [`align_cluster_root`](#align_cluster_root) | function | Ensures this process's DEK is the cluster secret root. |
| [`load_cluster_row`](#load_cluster_row) | function | Reads the singleton cluster row when present. |
| [`acquire_permit`](#acquire_permit) | function | Takes one permit of `name` for `holder`, valid for `ttl`. |
| [`acquire_permits`](#acquire_permits) | function | Takes one permit of every semaphore in `requests`, or none. |
| [`live_permits`](#live_permits) | function | Live permits of `name` among its first `max_slots` slots (expired slots are not counted). |
| [`permits_held_by`](#permits_held_by) | function | Permits of `name` among its first `max_slots` slots whose holder starts with `holder_prefix`, expired ones included. |
| [`release_permits`](#release_permits) | function | Gives back every permit in `permits` so the slots are free at once. |
| [`renew_fenced_lease`](#renew_fenced_lease) | function | Extends `lease` by `ttl` only while the row still carries its holder and token. Unlike `acquire_lease`, it never takes a lease over. |
| [`renew_permits`](#renew_permits) | function | Extends every permit in `permits` by `ttl` while its holder and token still own the slot. |
| [`semaphore_slot_lease_name`](#semaphore_slot_lease_name) | function | Lease name of slot `slot` of semaphore `name`. |
| [`accounts_owned_by`](#accounts_owned_by) | function | Store accounts linked to `user_id` through any of the user's identities. |
| [`book_matches_title`](#book_matches_title) | function | True when `id` names `book` (uuid, product id, ISBN or ASIN, any case), as an acquire's title filter matches. |
| [`claimant_may_run`](#claimant_may_run) | function | True when a `claimant` may run `job` at all, whatever it is trusted with: a forced or PDF-only acquire runs only on a node that holds the library database, since the host acquire path does neither. |
| [`handoff_routed_job`](#handoff_routed_job) | function | Re-checks a claimed job just before its secrets are handed to the claiming node. |
| [`job_item_accounts`](#job_item_accounts) | function | The store accounts a scan or acquire touches: its items. |
| [`job_trusted`](#job_trusted) | function | True when a node with `trust` may receive `job`. |
| [`node_may_claim_job`](#node_may_claim_job) | function | True when `node_id` may claim `job` right now. |
| [`node_trust`](#node_trust) | function | Resolves what `node_id` is trusted with right now. |
| [`plan_acquire`](#plan_acquire) | function | Plans the acquire jobs one request admits. |
| [`plan_scan`](#plan_scan) | function | Plans the scan jobs one request admits. |
| [`remote_node_may_view_job`](#remote_node_may_view_job) | function | True when a remote node may read `job`'s state. |
| [`remote_node_requester`](#remote_node_requester) | function | Whom library work a remote node asks for runs as (a `scan.request` over the node API): the user who owns the node, while the node is trusted with anything. `None` for a node that is trusted with nothing (revoked, ownerless, a disabled owner) or unknown, which may ask for nothing. |
| [`abort_job_uploads`](#abort_job_uploads) | function | Aborts every open upload `node_id` was granted under the job's lease generation `generation` (the job finished or its fence was lost, so none of them can commit); answers the aborted sessions. |
| [`abort_upload`](#abort_upload) | function | Aborts an open upload (its fence was lost, its commit was refused for a lost fence, or the destinations changed). Answers the session as it stood when this call aborted it, its receipts final (no file is recorded on an aborted upload), so the receiving node can delete every copy it received; `None` when it was not open. |
| [`authorize_upload`](#authorize_upload) | function | The upload `token` opens: `upload_id`'s row when the token's hash matches, the token has not expired, and the upload was neither aborted nor (unless `allow_committed`) committed. |
| [`commit_upload`](#commit_upload) | function | Commits an upload whose every file was received. |
| [`extend_job_uploads`](#extend_job_uploads) | function | Extends the token of every open upload `node_id` holds under the job's lease generation `generation`, after that lease was renewed; answers how many were extended. |
| [`grant_upload`](#grant_upload) | function | Opens an upload of one title of the acquire job `node_id` holds at `generation`, to be received by `receiving_node_id`. |
| [`job_titles`](#job_titles) | function | The owner's library rows the acquire `job` run by `node_id` covers: of the job's plugins (`job_plugin_keys`), of the accounts its account selector names (`job_accounts`, so a job naming its account by label or in another case covers that account's titles) and its title when it names them, and, for a Host trusted only with its owner's accounts, one of that user's. `None` when the node is trusted with nothing. |
| [`load_upload`](#load_upload) | function | Loads one upload by id. |
| [`names_book`](#names_book) | function | True when `book` is one of the titles `title` names. |
| [`record_upload_file`](#record_upload_file) | function | Records that file `index` reached every destination and extends the upload's token. Answers false when the upload is no longer open (it was aborted meanwhile: the caller deletes the copies it just wrote). |
| [`upload_fence_live`](#upload_fence_live) | function | True while the upload's fence is the job's live lease for its node and the node is still trusted with the job. |
| [`upload_owns_key`](#upload_owns_key) | function | True while `key` on `destination` is still `session`'s own: its recorded latest writer is this upload. A later upload of the title, an acquire run on this node, or another title may have written the key since; the receiving node then leaves the file alone when it cleans up after `session`, so a stale abort never deletes a replacement's file. |
| [`activate_version`](#activate_version) | function | Moves the active pointer of `plugin_key` to `version` (first install, update, or rollback), pins the version's settings schema, and wakes every node. Pointing at the version that is already active is a no-op. |
| [`add_plugin_version`](#add_plugin_version) | function | Records a version. Installing the same `(plugin_key, version)` again with the same artifact digests is a no-op (`AddVersion::Existing`); different digests under an installed version are refused, so a version never changes content behind a node's cache. |
| [`approve_version_grant`](#approve_version_grant) | function | Records the operator-approved consent grant on a version. |
| [`clear_install`](#clear_install) | function | Clears the active pointer (`plugins uninstall`). Version rows stay so the plugin can be re-activated; nodes retire guests whose plugin has no active version and sweep the cache. |
| [`list_installs`](#list_installs) | function | Every active pointer. |
| [`list_versions`](#list_versions) | function | Every version of `plugin_key` (or of every plugin), newest install first. |
| [`load_active_version`](#load_active_version) | function | The active version of `plugin_key`, when a pointer names one. |
| [`load_artifact_blob`](#load_artifact_blob) | function | Reads an archive stored by `put_artifact_blob`. |
| [`load_install`](#load_install) | function | The active pointer of `plugin_key`. |
| [`load_version`](#load_version) | function | One version row. |
| [`put_artifact_blob`](#put_artifact_blob) | function | Stores a small archive and returns its `blob:sha256:<hex>` URL. |
| [`bootstrap_control_plane`](#bootstrap_control_plane) | function | Enrolls this files directory and imports `[events]` and `[output]` when their documents are absent. |
| [`overlay_core_documents`](#overlay_core_documents) | function | Copies every authoritative cluster document in `session` onto `config`. |
| [`overlay_events`](#overlay_events) | function | Copies the authoritative events document onto `config` for `cluster_id`. |
| [`load_events`](#load_events) | function | Loads the cluster events document. |
| [`import_events_if_absent`](#import_events_if_absent) | function | Inserts `[events]` when the cluster document is absent. |
| [`replace_events`](#replace_events) | function | Replaces the cluster events document. |
| [`load_node_runtime`](#load_node_runtime) | function | Loads one node's runtime document. |
| [`import_node_runtime_if_absent`](#import_node_runtime_if_absent) | function | Inserts a node runtime document when it is absent. |
| [`replace_node_runtime`](#replace_node_runtime) | function | Replaces one node's runtime document. |
| [`update_node_runtime`](#update_node_runtime) | function | Applies `edit` to a node's runtime document (creating it with defaults when the node never wrote one) under compare-and-swap, retrying a few times on a concurrent edit. Budgets reach the scheduler with the node's next heartbeat. |
| [`LiveRevisions`](#liverevisions) | type alias | Every topic's revision, in `LiveTopic::ALL` order. |
| [`AccountKey`](#accountkey) | type alias | A store account, keyed as in `accounts`: `(plugin_key, account_id)`. |
| [`BRANCH_TITLE_DETACHED`](#branch_title_detached) | constant | `branch_titles.state` of a title from a branch that stopped sharing, left, or withdrew it. |
| [`BRANCH_TITLE_SHARED`](#branch_title_shared) | constant | `branch_titles.state` of a title its branch still shares. |
| [`DETACHED_LEFT`](#detached_left) | constant | Detach reason: the branch left the library (or was revoked). |
| [`DETACHED_STOPPED_SHARING`](#detached_stopped_sharing) | constant | Detach reason: the branch turned "Share this branch's books" off. |
| [`DETACHED_WITHDRAWN`](#detached_withdrawn) | constant | Detach reason: a full pass of the branch's catalog no longer named it. |
| [`FULL_CATALOG_PASS_EVERY`](#full_catalog_pass_every) | constant | How long a branch's catalog sync resumes incrementally before the library asks for a full pass again, so titles a branch no longer has (removed, or from a sub-branch that left) are withdrawn within a day. |
| [`CLUSTER_LINK_ACTIVE`](#cluster_link_active) | constant | `cluster_links.state` of a link the host client connects. |
| [`CLUSTER_LINK_KEYS_NAME`](#cluster_link_keys_name) | constant | `encrypted_secrets.name` of a link's node keys. |
| [`CLUSTER_LINK_LEFT`](#cluster_link_left) | constant | `cluster_links.state` after this install left the cluster. |
| [`CLUSTER_LINK_REVOKED`](#cluster_link_revoked) | constant | `cluster_links.state` once the owner revoked this node (close 4403). |
| [`MAX_OBSERVATION_DETAIL_BYTES`](#max_observation_detail_bytes) | constant | Maximum UTF-8 size of an observation detail string. |
| [`CLUSTER_SCOPE_ID`](#cluster_scope_id) | constant | Cluster scope instance. There is one cluster document per namespace. |
| [`CLUSTER_SCOPE_TYPE`](#cluster_scope_type) | constant | Scope type stored for cluster documents. |
| [`MAX_CONFIGURATION_DOCUMENT_BYTES`](#max_configuration_document_bytes) | constant | Maximum UTF-8 size of `document_json`. |
| [`NODE_SCOPE_TYPE`](#node_scope_type) | constant | Scope type stored for node documents. |
| [`PLUGIN_INSTANCE_CONFIG_NAMESPACE`](#plugin_instance_config_namespace) | constant | Namespace of a plugin instance's typed configuration document. |
| [`PLUGIN_INSTANCE_SCOPE_TYPE`](#plugin_instance_scope_type) | constant | Scope type stored for one plugin instance's configuration. |
| [`AUTHORIZED_LINK_CREATED_BY`](#authorized_link_created_by) | constant | Prefix of the `created_by` audit id of a link a sign-in created. |
| [`DEFAULT_LINK_TTL`](#default_link_ttl) | constant | Default lifetime of a link. |
| [`ENROLLMENT_TOKEN_PREFIX`](#enrollment_token_prefix) | constant | Version tag that starts every enrollment token. |
| [`HOST_ENROLLMENT_NAMESPACE`](#host_enrollment_namespace) | constant | Namespace of the cluster's host-enrollment switch. |
| [`HOST_ENROLLMENT_SCHEMA_VERSION`](#host_enrollment_schema_version) | constant | Schema version of `HostEnrollmentSettingsV1`. |
| [`MAX_LINK_LABEL_BYTES`](#max_link_label_bytes) | constant | Longest `label` stored on a link. |
| [`MAX_LINK_TTL`](#max_link_ttl) | constant | Longest lifetime a link may be minted with. |
| [`MIN_LINK_TTL`](#min_link_ttl) | constant | Shortest lifetime a link may be minted with. |
| [`NODE_PUBLIC_KEY_LEN`](#node_public_key_len) | constant | Size of an Ed25519 or X25519 public key. |
| [`NODE_IDENTITY_FILE`](#node_identity_file) | constant | Bootstrap file under `$BOOKCLERK_FILES_DIR` that stores the stable node id. |
| [`MAX_INSTANCE_SECRET_REFS`](#max_instance_secret_refs) | constant | Maximum secret refs in one schema-1 document. |
| [`MAX_INSTANCE_SETTINGS`](#max_instance_settings) | constant | Maximum settings in one schema-1 document. |
| [`MAX_INSTANCE_SETTING_BYTES`](#max_instance_setting_bytes) | constant | Maximum UTF-8 size of one string setting. |
| [`PLUGIN_INSTANCE_CONFIG_SCHEMA_VERSION`](#plugin_instance_config_schema_version) | constant | Domain schema of `PluginInstanceConfigV1`. |
| [`ACCOUNT_SEMAPHORE_LIMIT`](#account_semaphore_limit) | constant | Permits an account semaphore grants: one job per store account at a time. |
| [`ROUTED_CLAIM_CANDIDATES`](#routed_claim_candidates) | constant | Pending candidates read per page of a routed claim attempt (the claim pages on until it claims a job or the ready rows run out). |
| [`EVENT_RETENTION_LEASE`](#event_retention_lease) | constant | Lease of the event retention duty (terminal deliveries, empty events, stale subscriber catalog rows). |
| [`JOB_RETENTION_LEASE`](#job_retention_lease) | constant | Lease of the job retention duty (terminal `jobs` rows). |
| [`KV_EXPIRY_LEASE`](#kv_expiry_lease) | constant | Lease of the plugin KV expiry duty. |
| [`MIGRATION_LEASE`](#migration_lease) | constant | Lease a schema migration runs under (see `crate::apply_host_schema_with_batch_opts`). |
| [`SESSION_EXPIRY_LEASE`](#session_expiry_lease) | constant | Lease of the session duty (expired operator sessions, OIDC RP states, WebAuthn challenges). |
| [`NODE_CONSUMER_PREFIX`](#node_consumer_prefix) | constant | Catalog `plugin_id` prefix of a node's own consumer (`node:<node_id>`). |
| [`AVAILABILITY_FULL_PPM`](#availability_full_ppm) | constant | Full availability in parts per million. |
| [`AVAILABILITY_TAU`](#availability_tau) | constant | Time constant of the availability average (about a day). |
| [`DEFAULT_GUEST_CPU_MILLICORES`](#default_guest_cpu_millicores) | constant | CPU a guest is assumed to need when no ceiling is known (the grant default, 80% of a core). |
| [`DEFAULT_GUEST_MEMORY_BYTES`](#default_guest_memory_bytes) | constant | Memory a guest is assumed to need when no ceiling is known (the grant default). |
| [`NODE_HEARTBEAT_INTERVAL`](#node_heartbeat_interval) | constant | How often a daemon heartbeats. |
| [`NODE_LIVE_INTERVALS`](#node_live_intervals) | constant | A node whose last heartbeat is older than this many intervals is not live. |
| [`MAX_OUTPUT_REPLACEMENT_RULES`](#max_output_replacement_rules) | constant | Maximum number of explicit path replacement rules. |
| [`MAX_OUTPUT_TEMPLATE_BYTES`](#max_output_template_bytes) | constant | Maximum byte length of a naming template. |
| [`OUTPUT_NAMESPACE`](#output_namespace) | constant | Namespace for the migrated shared `[output]` domain. |
| [`OUTPUT_SCHEMA_VERSION`](#output_schema_version) | constant | Schema version of `OutputSettingsV1`. |
| [`OUTPUT_SETTING_ALIASES`](#output_setting_aliases) | constant | Write-only aliases `OutputSettingsV1::set` also accepts. |
| [`OUTPUT_SETTING_KEYS`](#output_setting_keys) | constant | Every `output.*` key `OutputSettingsV1::get` and `OutputSettingsV1::set` accept, in display order. |
| [`MAX_JOIN_NAME_BYTES`](#max_join_name_bytes) | constant | Longest branch or library name stored. |
| [`PENDING_JOIN_TTL`](#pending_join_ttl) | constant | How long a sign-in may take before its pending row is refused. |
| [`MAX_NODE_LABELS`](#max_node_labels) | constant | Most operator labels one node may carry. |
| [`MAX_PLACEMENT_CONSTRAINTS`](#max_placement_constraints) | constant | Most constraints one placement may carry. |
| [`MAX_REPLICAS`](#max_replicas) | constant | Largest replica count an operator may ask for. |
| [`NODE_LABEL`](#node_label) | constant | Implicit label name every node carries with its own id. |
| [`SCHEDULER_LEASE`](#scheduler_lease) | constant | Name of the cluster lease the scheduler runs under. |
| [`SCHEDULER_LEASE_TTL`](#scheduler_lease_ttl) | constant | How long a scheduler lease lasts without renewal. |
| [`TARGET_LABEL`](#target_label) | constant | Implicit label name every node carries with its artifact target. |
| [`REMOTE_CONSENT_PLATFORM`](#remote_consent_platform) | constant | `approved_by` of an automatic approval: a bundled platform plugin whose request stays inside the platform envelope. |
| [`MAX_SCAN_RESULT_ROWS`](#max_scan_result_rows) | constant | Most rows one results batch may carry. |
| [`MAX_WRITE_BACK_BYTES`](#max_write_back_bytes) | constant | Largest store token a Host may write back. |
| [`REMOTE_LEASE_OWNER_PREFIX`](#remote_lease_owner_prefix) | constant | Prefix of the lease owner of a job a Host runs. |
| [`SECRETS_UNAVAILABLE`](#secrets_unavailable) | constant | Error kind recorded when a job's secrets cannot be unsealed for a Host. |
| [`REQUIREMENTS_GRACE`](#requirements_grace) | constant | How long a pending job's requirements may stay unsettled before the sweep acts on them. Several scheduler passes (20 s apart) and guest restarts fit inside it. |
| [`REQUIREMENTS_SWEEP_LEASE`](#requirements_sweep_lease) | constant | Cluster lease the daemon holds while it sweeps. |
| [`REQUIREMENTS_SWEEP_LEASE_TTL`](#requirements_sweep_lease_ttl) | constant | Lease length; the holder renews it on every pass. |
| [`UNROUTABLE_ERROR_KIND`](#unroutable_error_kind) | constant | `error_kind` of a job the sweep failed. |
| [`ROUTE_DEFER_PER_RANK`](#route_defer_per_rank) | constant | How long each rank waits behind the one above it. |
| [`MAX_SEMAPHORE_LIMIT`](#max_semaphore_limit) | constant | Largest limit a semaphore honours; larger limits are clamped. |
| [`SEMAPHORE_LEASE_PREFIX`](#semaphore_lease_prefix) | constant | Prefix of every semaphore slot lease name. |
| [`UPLOAD_NAME_WILDCARD`](#upload_name_wildcard) | constant | In a `TitleNames::part_patterns` pattern: one or more characters of a file name (no `/`), where a part or chapter title goes. Storage keys hold no control character, so the marker never clashes with one. |
| [`UPLOAD_NUMBER_WILDCARD`](#upload_number_wildcard) | constant | In a `TitleNames::part_patterns` pattern: one or more ASCII digits, where a part or chapter number goes. |
| [`UPLOAD_TOKEN_TTL_SECS`](#upload_token_ttl_secs) | constant | How long a granted upload's token is accepted past its latest extension (the grant, an accepted file, or a renewal of the job's lease). |
| [`ARTIFACT_BLOB_SCHEME`](#artifact_blob_scheme) | constant | Artifact URL scheme of an archive stored in `plugin_artifacts`. |
| [`MAX_ARTIFACT_BLOB_BYTES`](#max_artifact_blob_bytes) | constant | Largest archive stored in `plugin_artifacts` (base64 stays under D1's row cap). |
| [`EVENTS_NAMESPACE`](#events_namespace) | constant | Namespace for the migrated `[events]` domain. |
| [`NODE_RUNTIME_NAMESPACE`](#node_runtime_namespace) | constant | Namespace for node-scoped runtime settings. |
| [`EVENTS_SCHEMA_VERSION`](#events_schema_version) | constant | Schema version of `EventsSettingsV1`. |
| [`NODE_RUNTIME_SCHEMA_VERSION`](#node_runtime_schema_version) | constant | Schema version of `NodeRuntimeSettingsV1`. |
| [`MAX_EVENTS_RETENTION_DAYS`](#max_events_retention_days) | constant | Upper bound for event retention fields, in days. |
| [`MAX_EVENTS_CONCURRENCY`](#max_events_concurrency) | constant | Upper bound for `[events].concurrency`. |
| [`DEFAULT_CPU_BUDGET_PERCENT`](#default_cpu_budget_percent) | constant | Default CPU budget: leave a fifth of the cores to the rest of the node. |
| [`DEFAULT_MEMORY_BUDGET_PERCENT`](#default_memory_budget_percent) | constant | Default memory budget: leave a quarter of the memory to the rest of the node. |

## Structs

### BranchTitle

```rust
pub struct BranchTitle {
    pub uuid: String,
    pub provenance: String,
    pub node_id: String,
    pub owner_user_id: Option<i64>,
    pub store: String,
    pub product_id: String,
    pub title: String,
    pub authors: Option<String>,
    pub primary_key: String,
    pub delivered: bool,
    pub state: String,
    pub detached_reason: Option<String>,
    pub updated_at: String,
}
```

One title a branch shared, as the owner's Settings → Branches lists it.

**Fields**

- `uuid`: Stable id.
- `provenance`: `node:<node_id>`.
- `node_id`: The branch's node id.
- `owner_user_id`: The user it belongs to here (the branch's owner).
- `store`: Store id.
- `product_id`: Product id.
- `title`: Display title.
- `authors`: Authors.
- `primary_key`: Storage key of the primary audio file.
- `delivered`: True once the current manifest's files arrived.
- `state`: `shared` or `detached`.
- `detached_reason`: `stopped_sharing`, `left` or `withdrawn`.
- `updated_at`: RFC 3339 last change.

Implements: `Clone`, `Debug`, `Eq`, `From<Model>`, `PartialEq`, `Serialize`

### BranchTitleCounts

```rust
pub struct BranchTitleCounts {
    pub shared: u64,
    pub delivered: u64,
    pub detached: u64,
}
```

Counts of one branch's shared titles.

**Fields**

- `shared`: Titles it shares now.
- `delivered`: Of those, titles whose files arrived.
- `detached`: Titles marked detached (it stopped sharing, left, or withdrew them).

Implements: `Clone`, `Copy`, `Debug`, `Default`, `Eq`, `PartialEq`, `Serialize`

### CatalogSessionOpened

```rust
pub struct CatalogSessionOpened {
    pub fence: i64,
    pub cursor: String,
    pub wanted: Vec<CatalogRef>,
}
```

An opened catalog session ([`begin_catalog`](#begin_catalog)).

**Fields**

- `fence`: The new catalog fence.
- `cursor`: Where the branch resumes (empty: from the start).
- `wanted`: Titles recorded earlier whose files have not arrived.

Implements: `Clone`, `Debug`, `Eq`, `PartialEq`

### LibraryContents

```rust
pub struct LibraryContents {
    pub users: u64,
    pub accounts: u64,
    pub books: u64,
    pub libraries_joined: u64,
}
```

What library data an install holds of its own.

**Fields**

- `users`: Users of any role.
- `accounts`: Store accounts.
- `books`: Books.
- `libraries_joined`: Libraries this install is a branch of (not left).

#### LibraryContents::is_empty

```rust
pub fn is_empty(&self) -> bool
```

True when the install holds no library data: it may join as a clerk.

#### LibraryContents::reasons

```rust
pub fn reasons(&self) -> Vec<String>
```

Why the install is not empty, for the operator.

Implements: `Clone`, `Copy`, `Debug`, `Default`, `Eq`, `PartialEq`, `Serialize`

### ClusterLink

```rust
pub struct ClusterLink {
    pub owner_cluster_id: String,
    pub owner_origin: String,
    pub remote_node_id: String,
    pub spki_pin: Option<String>,
    pub state: String,
    pub enrolled_at: String,
    pub share_books: bool,
    pub include_branches: bool,
    pub ancestry: Vec<String>,
}
```

One owner cluster this install enrolled into.

**Fields**

- `owner_cluster_id`: Owner cluster id.
- `owner_origin`: Owner origin as [`bookclerk_node_proto::canonical_origin`](../bookclerk_node_proto/index.md#canonical_origin) spells it.
- `remote_node_id`: Node id the owner minted for this install.
- `spki_pin`: SPKI pin of the owner's self-signed certificate, when the link
  carried one.
- `state`: [`CLUSTER_LINK_ACTIVE`](#cluster_link_active), [`CLUSTER_LINK_REVOKED`](#cluster_link_revoked), or
  [`CLUSTER_LINK_LEFT`](#cluster_link_left).
- `enrolled_at`: RFC 3339 enrollment time.
- `share_books`: "Share this branch's books with *library*" (on by default).
- `include_branches`: "Include books from my own branches" (on by default; off shares
  only books acquired on this install itself).
- `ancestry`: The owner library's ancestry as it last reported it: its own
  cluster id first, then each library above it.

#### ClusterLink::is_active

```rust
pub fn is_active(&self) -> bool
```

True when the host client should connect this link.

Implements: `Clone`, `Debug`, `Eq`, `From<Model>`, `PartialEq`, `Serialize`

### ClusterLinkKeys

```rust
pub struct ClusterLinkKeys {
    /* private fields */
}
```

The node keys of one cluster link: the Ed25519 signing seed and the
X25519 box secret. Zeroized on drop; `Debug` shows only the public keys.

#### ClusterLinkKeys::generate

```rust
pub fn generate() -> Self
```

Fresh random keys from the OS RNG.

#### ClusterLinkKeys::from_bytes

```rust
pub fn from_bytes(sign_seed: &[u8; 32], box_secret: &[u8; 32]) -> Self
```

Keys from known bytes (tests, and reading the sealed row back).

#### ClusterLinkKeys::sign_seed

```rust
pub fn sign_seed(&self) -> &[u8; 32]
```

The Ed25519 signing seed.

#### ClusterLinkKeys::box_secret

```rust
pub fn box_secret(&self) -> &[u8; 32]
```

The X25519 box secret.

#### ClusterLinkKeys::sign_public_key

```rust
pub fn sign_public_key(&self) -> [u8; 32]
```

The Ed25519 public key the owner stores.

#### ClusterLinkKeys::box_public_key

```rust
pub fn box_public_key(&self) -> [u8; 32]
```

The X25519 public key the owner seals job secrets to.

Implements: `Clone`, `Debug`

### NewClusterLink

```rust
pub struct NewClusterLink {
    pub owner_cluster_id: String,
    pub owner_origin: String,
    pub remote_node_id: String,
    pub spki_pin: Option<String>,
}
```

What enrollment returned, to record with [`record_cluster_link`](#record_cluster_link).

**Fields**

- `owner_cluster_id`: Owner cluster id from the enroll answer.
- `owner_origin`: Owner origin the host enrolled through.
- `remote_node_id`: Node id from the enroll answer.
- `spki_pin`: SPKI pin from the enrollment link.

Implements: `Clone`, `Debug`, `Eq`, `PartialEq`

### DeploymentObservation

```rust
pub struct DeploymentObservation {
    pub deployment_id: String,
    pub node_id: String,
    pub incarnation: String,
    pub status: DeploymentStatus,
    pub detail: String,
    pub applied_config_revision: Option<i64>,
    pub observed_at: String,
}
```

One node's observation of a deployment.

**Fields**

- `deployment_id`: Deployment id.
- `node_id`: Observing node.
- `incarnation`: Process incarnation.
- `status`: Last status.
- `detail`: Bounded detail. Empty when healthy.
- `applied_config_revision`: Config revision applied by spawn, if a spawn has happened.
- `observed_at`: RFC 3339 observation time.

Implements: `Clone`, `Debug`, `Eq`, `PartialEq`

### PluginDeployment

```rust
pub struct PluginDeployment {
    pub deployment_id: String,
    pub plugin_instance_id: PluginInstanceId,
    pub node_id: String,
    pub desired: DesiredState,
    pub revision: i64,
    pub updated_at: String,
    pub updated_by: String,
}
```

One desired deployment row.

**Fields**

- `deployment_id`: Deployment id.
- `plugin_instance_id`: Instance to run.
- `node_id`: Target node.
- `desired`: What the node should do, and whose decision it is.
- `revision`: Compare-and-swap revision.
- `updated_at`: RFC 3339 commit time.
- `updated_by`: Audit identity of the last desired-state writer.

Implements: `Clone`, `Debug`, `Eq`, `PartialEq`

### DocumentKey

```rust
pub struct DocumentKey {
    pub scope_type: String,
    pub scope_id: String,
    pub namespace: String,
}
```

Address of one configuration document.

**Fields**

- `scope_type`: `cluster`, `node`, or `plugin_instance`.
- `scope_id`: `singleton`, a node id, or a plugin instance id.
- `namespace`: Domain namespace.

#### DocumentKey::cluster

```rust
pub fn cluster(namespace: &str) -> Self
```

Cluster-scoped domain.

#### DocumentKey::node

```rust
pub fn node(node_id: &str, namespace: &str) -> Self
```

Node-scoped domain.

#### DocumentKey::plugin_instance

```rust
pub fn plugin_instance(instance_id: &str) -> Self
```

Plugin-instance configuration document.

The address is `(plugin_instance, instance_id, config)`. It does not
include a plugin key, capability, alias, or node id.

Implements: `Clone`, `Debug`, `Eq`, `PartialEq`

### StoredDocument

```rust
pub struct StoredDocument {
    pub key: DocumentKey,
    pub schema_version: i64,
    pub revision: i64,
    pub document_json: String,
    pub updated_at: String,
    pub updated_by: String,
    pub write_operation_id: String,
}
```

One stored document before domain parsing.

**Fields**

- `key`: Document address.
- `schema_version`: Domain schema version column.
- `revision`: Compare-and-swap revision.
- `document_json`: JSON body.
- `updated_at`: RFC 3339 commit time.
- `updated_by`: Audit identity of the last writer.
- `write_operation_id`: Operation id of the write that produced this revision.

Implements: `Clone`, `Debug`, `Eq`, `PartialEq`

### VerifiedClusterOwner

```rust
pub struct VerifiedClusterOwner {
    /* private fields */
}
```

Proof that a user was an active Owner when the write was authorized.

Only [`super::verify_cluster_owner`](#verify_cluster_owner) constructs one.

#### VerifiedClusterOwner::user_id

```rust
pub fn user_id(&self) -> i64
```

Owner's user id.

Implements: `Clone`, `Debug`, `Eq`, `PartialEq`

### VerifiedNodeOwner

```rust
pub struct VerifiedNodeOwner {
    /* private fields */
}
```

Proof that a user owned a node when the write was authorized.

Only [`super::verify_node_owner`](#verify_node_owner) constructs one.

#### VerifiedNodeOwner::user_id

```rust
pub fn user_id(&self) -> i64
```

Owning user id.

#### VerifiedNodeOwner::node_id

```rust
pub fn node_id(&self) -> &str
```

Node this owner may administer.

Implements: `Clone`, `Debug`, `Eq`, `PartialEq`

### EnrolledNode

```rust
pub struct EnrolledNode {
    pub cluster_id: String,
    pub node_id: String,
    pub owner_user_id: i64,
    pub replayed: bool,
}
```

Result of a successful redeem.

**Fields**

- `cluster_id`: Owner cluster id.
- `node_id`: Node id minted for the host.
- `owner_user_id`: User who owns the node.
- `replayed`: True when this answered a retry of an earlier successful redeem.

Implements: `Clone`, `Debug`, `Eq`, `PartialEq`, `Serialize`

### EnrollmentLink

```rust
pub struct EnrollmentLink {
    pub link_id: String,
    pub for_user_id: i64,
    pub label: String,
    pub created_by: String,
    pub created_at: String,
    pub expires_at: String,
    pub redeemed_at: Option<String>,
    pub redeemed_node_id: Option<String>,
    pub revoked_at: Option<String>,
}
```

One link as listings show it. The secret and its hash are not included.

**Fields**

- `link_id`: Public link id.
- `for_user_id`: User the enrolled node will belong to.
- `label`: Free-form label.
- `created_by`: Audit id of the minter.
- `created_at`: RFC 3339 mint time.
- `expires_at`: RFC 3339 expiry.
- `redeemed_at`: RFC 3339 redeem time.
- `redeemed_node_id`: Node the redeem created.
- `revoked_at`: RFC 3339 revoke time.

#### EnrollmentLink::state

```rust
pub fn state(&self, now: DateTime<Utc>) -> &'static str
```

`pending`, `redeemed`, `revoked`, or `expired` at `now`.

Implements: `Clone`, `Debug`, `Eq`, `From<Model>`, `PartialEq`, `Serialize`

### EnrollmentSecret

```rust
pub struct EnrollmentSecret(/* private */);
```

A link's plaintext secret. Shown once at mint and never stored; `Debug`
does not print it.

#### EnrollmentSecret::from_text

```rust
pub fn from_text(secret: String) -> Self
```

Wraps a secret a host submitted.

#### EnrollmentSecret::expose

```rust
pub fn expose(&self) -> &str
```

The secret text (base64url, no padding).

Implements: `Clone`, `Debug`, `Eq`, `PartialEq`

### EnrollmentToken

```rust
pub struct EnrollmentToken {
    pub link_id: String,
    pub secret: EnrollmentSecret,
    pub spki_pin: Option<String>,
}
```

A parsed enrollment token.

**Fields**

- `link_id`: Link id.
- `secret`: Link secret.
- `spki_pin`: SPKI pin of the owner's self-signed certificate, when present.

Implements: `Clone`, `Debug`, `Eq`, `PartialEq`

### HostEnrollmentSettings

```rust
pub struct HostEnrollmentSettings {
    pub body: HostEnrollmentSettingsV1,
    pub revision: Option<i64>,
}
```

The host-enrollment switch as stored, or its default when no document
was ever written.

**Fields**

- `body`: Effective settings.
- `revision`: Stored revision; `None` while the default applies.

Implements: `Clone`, `Debug`, `Eq`, `PartialEq`

### HostEnrollmentSettingsV1

```rust
pub struct HostEnrollmentSettingsV1 {
    pub enabled: bool,
}
```

Typed `core.host_enrollment` document, schema version 1.

**Fields**

- `enabled`: When false, no link may be minted or redeemed.

Implements: `Clone`, `Debug`, `Default`, `Deserialize<'de>`, `Eq`, `PartialEq`, `Serialize`

### MintedLink

```rust
pub struct MintedLink {
    pub link: EnrollmentLink,
    pub secret: EnrollmentSecret,
}
```

A freshly minted link and its secret, which is returned only here.

**Fields**

- `link`: Stored link.
- `secret`: Plaintext secret; put it in the URL fragment and drop it.

#### MintedLink::token

```rust
pub fn token(&self, spki_pin: Option<&str>) -> Zeroizing<String>
```

The `bcn1.<link_id>.<secret>[.<spki_pin>]` token.

#### MintedLink::url

```rust
pub fn url(&self, origin: &str, spki_pin: Option<&str>) -> Zeroizing<String>
```

`<origin>/enroll#<token>`: the secret stays in the fragment.

Implements: `Clone`, `Debug`

### NodeCredential

```rust
pub struct NodeCredential {
    pub node_id: String,
    pub sign_public_key: Vec<u8>,
    pub box_public_key: Vec<u8>,
    pub link_id: String,
    pub enrolled_at: String,
    pub revoked_at: Option<String>,
    pub last_connected_at: Option<String>,
}
```

A host node's stored credential.

**Fields**

- `node_id`: Host node.
- `sign_public_key`: Ed25519 public signing key.
- `box_public_key`: X25519 public box key.
- `link_id`: Link that enrolled it.
- `enrolled_at`: RFC 3339 enrollment time.
- `revoked_at`: RFC 3339 revoke time.
- `last_connected_at`: RFC 3339 time of the last authenticated connection.

Implements: `Clone`, `Debug`, `Eq`, `From<Model>`, `PartialEq`, `Serialize`

### RedeemRequest

```rust
pub struct RedeemRequest {
    pub link_id: String,
    pub secret: EnrollmentSecret,
    pub sign_public_key: Vec<u8>,
    pub box_public_key: Vec<u8>,
    pub software_version: String,
}
```

What a host submits to redeem a link.

**Fields**

- `link_id`: Link id from the token.
- `secret`: Link secret from the token.
- `sign_public_key`: Ed25519 public signing key (32 bytes).
- `box_public_key`: X25519 public box key (32 bytes).
- `software_version`: Bookclerk version of the host.

Implements: `Clone`, `Debug`, `Eq`, `PartialEq`

### NodeIdentity

```rust
pub struct NodeIdentity {
    pub node_id: String,
    pub cluster_id: Option<String>,
}
```

Bootstrap identity loaded from [`NODE_IDENTITY_FILE`](#node_identity_file).

**Fields**

- `node_id`: Stable node id.
- `cluster_id`: Cluster binding, once enrollment has succeeded.

Implements: `Clone`, `Debug`, `Eq`, `PartialEq`

### NodeRecord

```rust
pub struct NodeRecord {
    pub node_id: String,
    pub cluster_id: String,
    pub created_at: String,
    pub incarnation: String,
    pub heartbeat_at: String,
    pub software_version: String,
    pub schema_state: String,
    pub compatible: bool,
    pub logical_cpus: Option<i64>,
    pub cpu_max_quota_us: Option<i64>,
    pub cpu_max_period_us: Option<i64>,
    pub memory_max_bytes: Option<i64>,
    pub memory_current_bytes: Option<i64>,
    pub memory_anon_bytes: Option<i64>,
    pub files_dir_free_bytes: Option<i64>,
    pub scratch_bytes: Option<i64>,
    pub target: String,
    pub labels: Vec<String>,
    pub kind: NodeKind,
    pub owner_user_id: Option<i64>,
    pub share_books: Option<bool>,
    pub include_branches: Option<bool>,
}
```

Durable node row plus the latest observation columns.

**Fields**

- `node_id`: Stable node id.
- `cluster_id`: Cluster id stored with the node.
- `created_at`: RFC 3339 creation time. Heartbeats do not change it.
- `incarnation`: Process incarnation from the latest heartbeat.
- `heartbeat_at`: RFC 3339 heartbeat time.
- `software_version`: Software version from the latest heartbeat.
- `schema_state`: Schema state display from the latest heartbeat.
- `compatible`: True when the reporting binary accepted the library schema.
- `logical_cpus`: `available_parallelism`, empty when the probe failed.
- `cpu_max_quota_us`: Process cgroup `cpu.max` quota in microseconds. Empty when unlimited or missing.
- `cpu_max_period_us`: Process cgroup `cpu.max` period in microseconds. Empty when unlimited or missing.
- `memory_max_bytes`: Process cgroup `memory.max` in bytes. Empty when unlimited or missing.
- `memory_current_bytes`: Process cgroup `memory.current` in bytes.
- `memory_anon_bytes`: Process cgroup `memory.stat` field `anon`.
- `files_dir_free_bytes`: Free bytes on the filesystem that holds the files directory.
- `scratch_bytes`: Byte size of the acquire scratch tree under the download cache.
- `target`: Bookclerk artifact target of the reporting binary; empty on a row
  written before targets were recorded.
- `labels`: Operator-assigned placement labels, sorted.
- `kind`: How the node joined the cluster.
- `owner_user_id`: User who owns this node and may administer it, when one is assigned.
- `share_books`: For a branch: "Share this branch's books" as it last reported it;
  empty until it does.
- `include_branches`: For a branch: "Include books from my own branches", as reported.

Implements: `Clone`, `Debug`, `Eq`, `PartialEq`

### InstanceBindingGrant

```rust
pub struct InstanceBindingGrant {
    pub config: bool,
    pub secrets: bool,
}
```

Whether the plugin-instance config grant includes `config` or `secrets`.

Resolved from the plugin key's grant before this function runs.

**Fields**

- `config`: Grant includes the `config` binding.
- `secrets`: Grant includes the `secrets` binding.

Implements: `Clone`, `Copy`, `Debug`, `Eq`, `PartialEq`

### InstanceConfigDocument

```rust
pub struct InstanceConfigDocument {
    pub body: PluginInstanceConfigV1,
    pub revision: i64,
    pub schema_version: i64,
    pub instance_id: String,
}
```

A parsed instance document plus its revision.

**Fields**

- `body`: Parsed body.
- `revision`: Compare-and-swap revision.
- `schema_version`: Domain schema version. Schema 1 is the only version this binary applies.
- `instance_id`: Instance id (`scope_id`).

Implements: `Clone`, `Debug`, `PartialEq`

### InstanceSecretRefV1

```rust
pub struct InstanceSecretRefV1 {
    pub key: String,
    pub name: String,
}
```

A name in `encrypted_secrets`. The ciphertext is not part of the document.

**Fields**

- `key`: Key in the `SECRETS` object delivered to the guest.
- `name`: `encrypted_secrets.name` for this instance.

Implements: `Clone`, `Debug`, `Deserialize<'de>`, `Eq`, `PartialEq`, `Serialize`

### PluginInstanceConfigV1

```rust
pub struct PluginInstanceConfigV1 {
    pub settings: BTreeMap<String, SettingValue>,
    pub secret_refs: Vec<InstanceSecretRefV1>,
}
```

Schema-1 plugin instance configuration.

**Fields**

- `settings`: Operator settings. Keys match `^[a-z][a-z0-9_]{0,63}$`.
- `secret_refs`: Secret names. Values stay in `encrypted_secrets`.

#### PluginInstanceConfigV1::empty

```rust
pub fn empty() -> Self
```

Empty settings and no secret refs.

#### PluginInstanceConfigV1::validate

```rust
pub fn validate(&self, schema: &SettingsSchema) -> Result<()>
```

Rejects documents whose settings or secret refs the plugin does not
declare, or whose values do not fit the declared types.

##### Errors

Returns an error naming the first offending key.

#### PluginInstanceConfigV1::validate_shape

```rust
pub fn validate_shape(&self) -> Result<()>
```

Rejects documents this binary will not store regardless of schema:
key grammar, scalar kinds, counts, and size.

Reads use this so a document written under an older manifest still
loads; spawn drops the keys the current schema no longer declares.

##### Errors

Returns an error when a key, value, or count fails.

#### PluginInstanceConfigV1::to_json

```rust
pub fn to_json(&self) -> Result<String>
```

Canonical JSON body stored in `document_json`.

##### Errors

Returns an error when the body cannot be serialized.

#### PluginInstanceConfigV1::settings_json

```rust
pub fn settings_json(&self) -> Value
```

Settings object delivered as the `CONFIG` payload.

Implements: `Clone`, `Debug`, `Deserialize<'de>`, `PartialEq`, `Serialize`

### PrunedInstanceConfig

```rust
pub struct PrunedInstanceConfig {
    pub instance_id: String,
    pub removed_keys: Vec<String>,
    pub revision: i64,
}
```

One instance document rewritten to fit the pinned schema.

**Fields**

- `instance_id`: Instance whose document changed.
- `removed_keys`: Keys dropped from the document: settings the schema no longer
  declares, declares as secrets, or whose value no longer fits the
  entry's kind; secret refs whose key is gone or now plain.
- `revision`: Revision after the rewrite.

Implements: `Clone`, `Debug`, `Eq`, `PartialEq`

### ResolvedInstanceBindings

```rust
pub struct ResolvedInstanceBindings {
    pub config: ExtensibleConfig,
    pub secrets: ExtensibleConfig,
    pub config_revision: i64,
    pub dropped_keys: Vec<String>,
}
```

`CONFIG` and `SECRETS` payloads for one instance.

**Fields**

- `config`: Schema defaults merged with the document, as `application/json`.
- `secrets`: Granted secret values as `application/json`, or an empty payload.
- `config_revision`: Document revision applied by this resolution.
- `dropped_keys`: Document keys left out of `config`: ones the schema no longer
  declares, and ones whose stored value no longer fits the entry's kind
  (the default applied instead).

Implements: `Clone`, `Debug`, `Eq`, `PartialEq`

### PluginInstance

```rust
pub struct PluginInstance {
    pub id: PluginInstanceId,
    pub plugin_key: String,
    pub display_name: String,
    pub created_at: String,
    pub created_by: String,
    pub owner_node_id: Option<String>,
}
```

One `plugin_instances` row.

**Fields**

- `id`: Stable id.
- `plugin_key`: Canonical plugin key. Immutable after create.
- `display_name`: Operator-facing name; empty for the default instance.
- `created_at`: RFC 3339 insert time.
- `created_by`: Audit identity of the creator.
- `owner_node_id`: The node that owns this instance (a node-local output, H-2); it may
  be deployed only there. `None` for cluster instances.

Implements: `Clone`, `Debug`, `Eq`, `PartialEq`

### PluginInstanceId

```rust
pub struct PluginInstanceId(/* private */);
```

Stable id of one plugin instance.

A hyphenated UUID version 4. It does not parse as a plugin key.

#### PluginInstanceId::mint

```rust
pub fn mint() -> Self
```

Mints a new random id.

#### PluginInstanceId::parse

```rust
pub fn parse(raw: &str) -> Result<Self>
```

Parses a hyphenated UUID.

##### Errors

Returns an error when `raw` is not a UUID.

#### PluginInstanceId::as_str

```rust
pub fn as_str(&self) -> &str
```

Hyphenated UUID text stored in the database.

Implements: `Clone`, `Debug`, `Display`, `Eq`, `Hash`, `Ord`, `PartialEq`, `PartialOrd`

### JobClaimFilter

```rust
pub struct JobClaimFilter {
    pub kinds: BTreeSet<String>,
    pub plugin_keys: BTreeSet<String>,
}
```

What a Host is willing to claim: the job kinds it runs and the plugin
keys its operator consented to. Both must admit a job.

**Fields**

- `kinds`: Job kinds (`scan`, `acquire`, ...).
- `plugin_keys`: Canonical plugin keys.

#### JobClaimFilter::is_empty

```rust
pub fn is_empty(&self) -> bool
```

True when the filter admits nothing.

#### JobClaimFilter::admits

```rust
pub async fn admits(&self, store: &LibraryStore, job: &JobRecord) -> Result<bool>
```

True when `job` is of an admitted kind, names the account it runs
for, every plugin it runs ([`job_plugin_keys`](#job_plugin_keys)) is admitted, and its
one source plugin can be told ([`job_source_plugin_key`](#job_source_plugin_key)), so the
Host can select the guest it runs. The account is required even
when the job is pinned to an instance: a Host is handed only that
account's secrets, and a job of every account has none to hand.

##### Errors

Returns an error when a read fails.

Implements: `Clone`, `Debug`, `Default`, `Eq`, `PartialEq`

### JobClaimLimits

```rust
pub struct JobClaimLimits {
    pub class: u32,
    pub plugin_instance: u32,
}
```

Cluster-wide permit counts a routed claim enforces.

**Fields**

- `class`: Running jobs of the claimed resource class, cluster-wide.
- `plugin_instance`: Running jobs that require one plugin instance, cluster-wide.

Implements: `Clone`, `Copy`, `Debug`, `Eq`, `PartialEq`

### JobClaimRequest

```rust
pub struct JobClaimRequest<'a> {
    pub resource_class: JobResourceClass,
    pub node_id: &'a str,
    pub claimant: Claimant,
    pub owner: &'a str,
    pub lease_secs: u64,
    pub limits: JobClaimLimits,
    pub filter: Option<&'a JobClaimFilter>,
}
```

One routed claim attempt by one worker.

**Fields**

- `resource_class`: Resource class the worker serves.
- `node_id`: The claiming node (`node-identity.json`). Empty when unknown, in
  which case only jobs without requirements are claimable.
- `claimant`: How the node reaches the queue; decides how an unknown node id is
  trusted (see [`Claimant`](#claimant)).
- `owner`: Worker lease owner written on the job row.
- `lease_secs`: Lease length; permits are taken for the same time.
- `limits`: Semaphore limits.
- `filter`: The claimer's own narrowing; `None` claims every job it is trusted
  with and can run.

Implements: `Clone`, `Debug`

### RoutedJobClaim

```rust
pub struct RoutedJobClaim {
    pub job: JobRecord,
    pub permits: Vec<SemaphorePermit>,
    pub resumed: bool,
}
```

A claimed job and the semaphore permits it holds.

**Fields**

- `job`: The running job (its fence is on the record).
- `permits`: Permits to renew with the job heartbeat and release at the end.
- `resumed`: True when this claim resumed a suspended job, so it spent no
  attempt; a release at handoff restores that state.

Implements: `Clone`, `Debug`

### ClusterLease

```rust
pub struct ClusterLease {
    pub name: String,
    pub holder: String,
    pub token: i64,
    pub expires_at_ms: i64,
}
```

A held cluster lease.

**Fields**

- `name`: Lease name.
- `holder`: Holder (a node id).
- `token`: Fencing token; a new holder always gets a larger one.
- `expires_at_ms`: Expiry in Unix milliseconds.

#### ClusterLease::is_held_at

```rust
pub fn is_held_at(&self, now_ms: i64) -> bool
```

True when the lease has not expired at `now_ms`.

Implements: `Clone`, `Debug`, `Eq`, `PartialEq`

### LeaseFence

```rust
pub struct LeaseFence<'a> {
    /* private fields */
}
```

Proof, checked against the database, that this holder still holds a
lease. Call [`LeaseFence::check`](#leasefencecheck) before each destructive step.

#### LeaseFence::new

```rust
pub fn new(store: &'a LibraryStore, lease: ClusterLease, ttl: Duration) -> Self
```

A fence for a lease acquired with `ttl`.

#### LeaseFence::lease

```rust
pub fn lease(&self) -> &ClusterLease
```

The lease as of the last successful check.

#### LeaseFence::check

```rust
pub async fn check(&mut self) -> Result<()>
```

Renews the lease, or fails when this holder no longer holds it.

##### Errors

Returns [`LibraryError::Conflict`](index.md#libraryerror) when the lease expired or another
holder took it, or the write error.

Implements: `Debug`

### SingletonDuty

```rust
pub struct SingletonDuty {
    /* private fields */
}
```

One periodic duty that exactly one live node runs per interval.

Every node keeps one per duty and calls [`SingletonDuty::poll`](#singletondutypoll) more
often than the lease TTL. `poll` acquires or renews the lease and hands
back a [`LeaseFence`](#leasefence) only when this node holds it and the duty is due:
at once after taking the lease, then every `interval`.

#### SingletonDuty::new

```rust
pub fn new(
    lease_name: impl Into<String>,
    holder: impl Into<String>,
    ttl: Duration,
    interval: Duration,
) -> Self
```

A duty under lease `lease_name` for `holder`.

#### SingletonDuty::lease_name

```rust
pub fn lease_name(&self) -> &str
```

The lease name.

#### SingletonDuty::poll

```rust
pub async fn poll<'a>(
    &mut self,
    store: &'a LibraryStore,
) -> Result<Option<LeaseFence<'a>>>
```

Takes or renews the lease; returns a fence when the duty should run
now on this node.

##### Errors

Returns an error when the lease read or write fails.

#### SingletonDuty::mark_ran

```rust
pub fn mark_ran(&mut self)
```

Records that the duty ran (successfully or not) so the next run waits
a full interval.

Implements: `Clone`, `Debug`

### LiveRevision

```rust
pub struct LiveRevision {
    pub rows: u64,
    pub newest: Option<String>,
}
```

One topic's revision: its row count and newest change stamp.

**Fields**

- `rows`: Rows behind the topic.
- `newest`: The newest change stamp (RFC 3339 text or a sequence number), or a
  digest of the rows for a table without one.

Implements: `Clone`, `Debug`, `Default`, `Eq`, `PartialEq`

### GuestReservation

```rust
pub struct GuestReservation {
    pub memory_bytes: i64,
    pub cpu_millicores: i64,
}
```

CPU and memory one guest may use at most: its grant's ceilings after the
node's `[plugins.jail]` limits.

**Fields**

- `memory_bytes`: Memory ceiling in bytes.
- `cpu_millicores`: CPU ceiling in millicores.

#### GuestReservation::default_guest

```rust
pub fn default_guest() -> Self
```

The grant defaults, used when nothing better is known.

#### GuestReservation::plus

```rust
pub fn plus(self, other: Self) -> Self
```

Component-wise sum.

#### GuestReservation::max

```rust
pub fn max(self, other: Self) -> Self
```

Component-wise maximum.

Implements: `Clone`, `Copy`, `Debug`, `Default`, `Eq`, `PartialEq`, `Serialize`

### NodeHeartbeat

```rust
pub struct NodeHeartbeat {
    pub incarnation: String,
    pub started_at: String,
    pub interval: Duration,
    pub cpu_millicores: i64,
    pub memory_bytes: Option<i64>,
    pub cpu_used_millicores: Option<i64>,
    pub memory_available_bytes: Option<i64>,
    pub files_dir_free_bytes: Option<i64>,
}
```

One heartbeat's measurements, before budget and accounting are applied.

**Fields**

- `incarnation`: Daemon process incarnation.
- `started_at`: RFC 3339 time the daemon started.
- `interval`: Promised heartbeat interval.
- `cpu_millicores`: Detected CPU capacity in millicores.
- `memory_bytes`: Detected memory in bytes.
- `cpu_used_millicores`: Measured CPU in use, in millicores.
- `memory_available_bytes`: Measured memory available to new work.
- `files_dir_free_bytes`: Free bytes on the files directory's filesystem.

Implements: `Clone`, `Debug`, `Eq`, `PartialEq`

### NodeSampler

```rust
pub struct NodeSampler {
    /* private fields */
}
```

Samples capacity and measured use for heartbeats. Keeps the previous CPU
counters so each sample reports utilization since the last one.

#### NodeSampler::new

```rust
pub fn new(files_dir: &Path) -> Self
```

A sampler for this process.

#### NodeSampler::with_cgroup

```rust
pub fn with_cgroup(files_dir: &Path, cgroup_dir: Option<PathBuf>) -> Self
```

A sampler reading cgroup files from `cgroup_dir` (tests point it at
fake controller files).

#### NodeSampler::sample

```rust
pub fn sample(&mut self) -> NodeHeartbeat
```

Reads capacity and use now. The first sample has no CPU use yet.
Blocking: small `/proc`, cgroup, and `statvfs` reads.

Implements: `Debug`

### NodeStatus

```rust
pub struct NodeStatus {
    pub node_id: String,
    pub incarnation: String,
    pub started_at: String,
    pub heartbeat_at: String,
    pub heartbeat_ms: i64,
    pub interval_ms: i64,
    pub cpu_millicores: i64,
    pub memory_bytes: Option<i64>,
    pub cpu_budget_millicores: i64,
    pub memory_budget_bytes: Option<i64>,
    pub schedulable: bool,
    pub cpu_used_millicores: Option<i64>,
    pub memory_available_bytes: Option<i64>,
    pub files_dir_free_bytes: Option<i64>,
    pub reserved: GuestReservation,
    pub running_guests: i64,
    pub availability_ppm: i64,
    pub online_since_ms: i64,
}
```

A node's latest heartbeat with budget, accounting, and availability.

**Fields**

- `node_id`: Reporting node.
- `incarnation`: Daemon process incarnation.
- `started_at`: RFC 3339 daemon start.
- `heartbeat_at`: RFC 3339 heartbeat time.
- `heartbeat_ms`: Heartbeat time in Unix milliseconds.
- `interval_ms`: Promised interval in milliseconds.
- `cpu_millicores`: Detected CPU capacity in millicores.
- `memory_bytes`: Detected memory.
- `cpu_budget_millicores`: CPU the scheduler may commit, in millicores.
- `memory_budget_bytes`: Memory the scheduler may commit.
- `schedulable`: False when an operator drained the node.
- `cpu_used_millicores`: Measured CPU in use.
- `memory_available_bytes`: Measured memory available.
- `files_dir_free_bytes`: Free bytes on the files directory's filesystem.
- `reserved`: Ceilings of the guests running here.
- `running_guests`: Guests running here.
- `availability_ppm`: Recent online fraction in parts per million.
- `online_since_ms`: Unix milliseconds the current online stretch began.

#### NodeStatus::is_live

```rust
pub fn is_live(&self, now_ms: i64) -> bool
```

True while the last heartbeat is younger than
[`NODE_LIVE_INTERVALS`](#node_live_intervals) intervals.

#### NodeStatus::cpu_headroom_millicores

```rust
pub fn cpu_headroom_millicores(&self) -> i64
```

CPU still uncommitted under the budget, in millicores. Measured use
counts when it is higher than the reservations (work outside
Bookclerk on a shared machine).

#### NodeStatus::memory_headroom_bytes

```rust
pub fn memory_headroom_bytes(&self) -> Option<i64>
```

Memory still uncommitted under the budget and actually available on
the machine; empty when the node could not detect its memory.

Implements: `Clone`, `Debug`, `Eq`, `PartialEq`, `Serialize`

### OutputSettingsV1

```rust
pub struct OutputSettingsV1 {
    pub format: OutputFormat,
    pub widevine: bool,
    pub xhe_aac: bool,
    pub widevine_cdm: Option<PathBuf>,
    pub widevine_cdm_provider: Option<String>,
    pub naming_profile: NamingProfile,
    pub folder_template: Option<String>,
    pub file_template: Option<String>,
    pub chapter_file_template: Option<String>,
    pub chapter_title_template: Option<String>,
    pub download_cover: bool,
    pub download_pdf: bool,
    pub create_cue: bool,
    pub fixup_metadata: bool,
    pub chapter_json: ChapterJsonMode,
    pub save_metadata_json: bool,
    pub cover_size: String,
    pub chapter_layout: String,
    pub overwrite_existing: bool,
    pub multi_destination: MultiDestinationMode,
    pub bad_book_action: BadBookAction,
    pub split_mp3_max_mb: u32,
    pub minimum_file_duration_minutes: u32,
    pub combine_nested_chapter_titles: bool,
    pub merge_opening_and_end_credits: bool,
    pub strip_unabridged: bool,
    pub strip_audible_brand_audio: bool,
    pub download_clips_bookmarks: bool,
    pub retain_aax_file: bool,
    pub download_speed_limit_kbps: u32,
    pub lame: LameConfig,
    pub max_sample_rate: Option<u32>,
    pub creation_time: FileTimestampMode,
    pub last_write_time: FileTimestampMode,
    pub path_sanitization: PathSanitizationMode,
    pub replacement_characters: Vec<ReplacementRule>,
    pub max_filename_length: u32,
}
```

Typed `core.output` document, schema version 1.

Field meanings match [`OutputConfig`](../bookclerk_config/index.md#outputconfig). Unknown fields are rejected, inside
the nested `lame` and `replacement_characters` objects too, so a document
written by a newer binary under the same schema version fails closed
instead of being silently truncated on the next write.

**Fields**

- `format`: Post-acquire packaging format.
- `widevine`: Prefer Widevine/CENC.
- `xhe_aac`: Prefer xHE-AAC on the Widevine path when offered.
- `widevine_cdm`: Optional `.wvd` path; relative paths resolve under each host's files dir.
- `widevine_cdm_provider`: Remote L3 CDM provider (`None` = AudibleCdm, empty/`off` disables).
- `naming_profile`: Named path-template preset.
- `folder_template`: Folder-path template override.
- `file_template`: File-stem template override.
- `chapter_file_template`: Per-chapter file-stem template override.
- `chapter_title_template`: Embedded chapter-title template.
- `download_cover`: Save cover JPEG alongside audio.
- `download_pdf`: Download companion PDF when available.
- `create_cue`: Write a `.cue` sidecar.
- `fixup_metadata`: Embed tags, cover, and chapters natively.
- `chapter_json`: Chapter JSON sidecars.
- `save_metadata_json`: Persist raw catalog API JSON.
- `cover_size`: Cover image size (`500`, `1215`, another pixel size, or `native`).
- `chapter_layout`: Preferred chapter API layout (`tree`, `flat`, or `both`).
- `overwrite_existing`: Re-acquire when media already exists at the destination.
- `multi_destination`: Policy when only some enabled destinations have a title.
- `bad_book_action`: Operator action when a title fails to acquire.
- `split_mp3_max_mb`: Max MP3 part size in MiB for `split_mp3_by_size`.
- `minimum_file_duration_minutes`: Drop chapter splits shorter than this many minutes.
- `combine_nested_chapter_titles`: Flatten nested chapter titles.
- `merge_opening_and_end_credits`: Merge opening/end credit chapters into neighbours.
- `strip_unabridged`: Strip an "Unabridged" suffix from titles used in naming.
- `strip_audible_brand_audio`: Trim Audible brand intro/outro audio.
- `download_clips_bookmarks`: Download Audible clips/bookmarks sidecars.
- `retain_aax_file`: Keep the encrypted download.
- `download_speed_limit_kbps`: Fetch speed cap in KB/s (`0` = unlimited).
- `lame`: LAME encoder knobs.
- `max_sample_rate`: Optional output sample-rate ceiling in Hz.
- `creation_time`: File creation timestamp policy.
- `last_write_time`: File last-write timestamp policy.
- `path_sanitization`: Sanitization profile used when no explicit rules are set.
- `replacement_characters`: Explicit find/replace rules for path segments.
- `max_filename_length`: Max length per path segment (`0` disables truncation).

#### OutputSettingsV1::from_config

```rust
pub fn from_config(config: &OutputConfig) -> Result<Self>
```

Copies the transitional `[output]` table, rejecting values the domain forbids.

The deprecated `save_chapter_json` flag is folded into
[`Self::chapter_json`](#outputsettingsv1), empty templates become unset, and the host-local
`in_progress` directory is not part of the document.

##### Errors

Returns an error when the table fails [`Self::validate`](#outputsettingsv1validate).

#### OutputSettingsV1::from_seed_lenient

```rust
pub fn from_seed_lenient(config: &OutputConfig) -> (Self, Vec<&'static str>)
```

Builds the document from a seed `[output]` table (TOML / environment)
for the first import, keeping every valid value and replacing each
field that fails [`Self::validate`](#outputsettingsv1validate) with its default.

Loading `config.toml` does not apply the document's bounds, so an
install upgrading from file-based output settings can hold a value
such as `max_filename_length = 2048` that ran before. Refusing it
would stop the host from starting; the replaced fields are returned
so the caller can log each one. Not used on a stored document, which
is validated strictly.

##### Arguments

* `config` - The seed `[output]` table.

##### Returns

The document, and the names of the fields that were replaced.

#### OutputSettingsV1::apply_to

```rust
pub fn apply_to(&self, config: &mut OutputConfig, files_dir: Option<&Path>)
```

Writes the document onto the in-memory `[output]` table.

`in_progress` is left as it was. A relative `widevine_cdm` resolves
under `files_dir` when one is given, the same rule `config.toml` paths
follow.

#### OutputSettingsV1::validate

```rust
pub fn validate(&self) -> Result<()>
```

Rejects values outside the domain bounds.

##### Errors

Returns an error naming the first field that is out of range.

#### OutputSettingsV1::get

```rust
pub fn get(&self, key: &str) -> Option<String>
```

Formats one `output.*` key for display.

Enums print their stored (snake_case) name; an unset optional value
prints as an empty string. A disabled `widevine_cdm_provider` (stored
empty) prints as `off`, which [`Self::set`](#outputsettingsv1set) reads back as disabled.

#### OutputSettingsV1::set

```rust
pub fn set(&mut self, key: &str, value: &str) -> Result<()>
```

Assigns one `output.*` key from its string form, then validates.

Unlike `-o key=value` overrides, an unparseable value is an error,
not a silent default. An empty value clears an optional field.
`replacement_characters` takes a JSON array of `{find, replace}`.

##### Errors

Returns an error for an unknown key, an unparseable value, or a value
the domain rejects. `self` may be partially changed on error; callers
edit a clone.

Implements: `Clone`, `Debug`, `Default`, `Deserialize<'de>`, `Eq`, `PartialEq`, `Serialize`

### NewPendingJoin

```rust
pub struct NewPendingJoin {
    pub owner_origin: String,
    pub spki_pin: Option<String>,
    pub owner_cluster_id: String,
    pub library_name: String,
    pub branch_name: String,
    pub redirect_uri: String,
    pub allow_clerk: bool,
}
```

What a joining daemon records when it starts a sign-in.

**Fields**

- `owner_origin`: Owner origin the sign-in goes to.
- `spki_pin`: SPKI pin the operator confirmed for a self-signed owner.
- `owner_cluster_id`: Owner cluster id `GET /api/node/v1/info` reported.
- `library_name`: Library name the owner reported.
- `branch_name`: Name this install joins as.
- `redirect_uri`: Redirect URI of the authorization request.
- `allow_clerk`: True when the operator offered to join as a clerk (the install was
  empty); the callback may then apply a clerk bootstrap.

Implements: `Clone`, `Debug`, `Eq`, `PartialEq`

### PendingJoin

```rust
pub struct PendingJoin {
    pub owner_origin: String,
    pub spki_pin: Option<String>,
    pub owner_cluster_id: String,
    pub library_name: String,
    pub branch_name: String,
    pub redirect_uri: String,
    pub allow_clerk: bool,
    pub expires_at: String,
}
```

A pending join as stored (no secrets).

**Fields**

- `owner_origin`: Owner origin.
- `spki_pin`: Confirmed SPKI pin.
- `owner_cluster_id`: Owner cluster id.
- `library_name`: Library name.
- `branch_name`: Branch name.
- `redirect_uri`: Redirect URI.
- `allow_clerk`: True when the operator offered to join as a clerk.
- `expires_at`: RFC 3339 expiry.

Implements: `Clone`, `Debug`, `Eq`, `From<Model>`, `PartialEq`

### TakenJoin

```rust
pub struct TakenJoin {
    pub join: PendingJoin,
    pub verifier: Zeroizing<String>,
    pub keys: ClusterLinkKeys,
}
```

A consumed pending join with its secrets. `Debug` prints no secret.

**Fields**

- `join`: The stored facts.
- `verifier`: The PKCE verifier.
- `keys`: The node keys generated for the join.

Implements: `Debug`

### NodeCandidate

```rust
pub struct NodeCandidate {
    pub node_id: String,
    pub labels: BTreeSet<String>,
    pub target: String,
    pub live: bool,
    pub schedulable: bool,
    pub cpu_headroom_millicores: i64,
    pub memory_headroom_bytes: Option<i64>,
    pub availability_ppm: i64,
}
```

One node as the planner sees it: identity, liveness, and the budget
headroom its last heartbeat reported.

**Fields**

- `node_id`: Node id.
- `labels`: Effective labels ([`effective_node_labels`](#effective_node_labels)).
- `target`: Artifact target; empty when unknown.
- `live`: True when the node's daemon heartbeated recently.
- `schedulable`: False when an operator drained the node.
- `cpu_headroom_millicores`: CPU still uncommitted under the node's budget, in millicores
  (measured use counts when it exceeds the reservations).
- `memory_headroom_bytes`: Memory still uncommitted under the budget and free on the machine;
  empty when the node could not detect its memory.
- `availability_ppm`: Recent online fraction in parts per million.

#### NodeCandidate::from_status

```rust
pub fn from_status(node: &NodeRecord, status: Option<&NodeStatus>, now_ms: i64) -> Self
```

A node as of its last heartbeat (`None`: its daemon never ran, so it
is not live).

#### NodeCandidate::fits

```rust
pub fn fits(&self, demand: GuestReservation) -> bool
```

True when one more guest needing `demand` fits the memory budget.
Memory is the hard limit (a guest over it is killed); CPU only ranks.

#### NodeCandidate::over_budget

```rust
pub fn over_budget(&self) -> bool
```

True when the node has committed more memory than its budget, or
more than the machine has free.

#### NodeCandidate::score

```rust
pub fn score(&self, demand: GuestReservation) -> i64
```

How many more guests like `demand` the node has room for, in
thousandths, weighted by availability (an always-on node counts in
full, a node that is never on counts half). Larger is better.

#### NodeCandidate::take

```rust
pub fn take(&mut self, demand: GuestReservation)
```

Commits `demand` on this node for the rest of a pass.

Implements: `Clone`, `Debug`, `Eq`, `PartialEq`

### PlacementPlan

```rust
pub struct PlacementPlan {
    pub add: Vec<String>,
    pub remove: Vec<String>,
    pub placed: u32,
    pub detail: String,
}
```

What the planner decided for one instance.

**Fields**

- `add`: Nodes that should gain a replica.
- `remove`: Nodes whose replica should be removed.
- `placed`: Replicas placed after the plan applies.
- `detail`: Why fewer than requested are placed; empty when satisfied.

Implements: `Clone`, `Debug`, `Default`, `Eq`, `PartialEq`

### PlacementSpec

```rust
pub struct PlacementSpec {
    pub replicas: Replicas,
    pub constraints: Vec<String>,
}
```

What an operator asks for: how many nodes, and which may qualify.

**Fields**

- `replicas`: Nodes that should run the instance.
- `constraints`: Label constraints every chosen node must satisfy.

#### PlacementSpec::normalized

```rust
pub fn normalized(&self) -> Result<Self>
```

Validated copy with constraints trimmed, sorted, and deduplicated.

##### Errors

Returns an error for a replica count over [`MAX_REPLICAS`](#max_replicas), too many
constraints, or a constraint outside the label grammar.

Implements: `Clone`, `Debug`, `Default`, `Eq`, `PartialEq`

### PluginPlacement

```rust
pub struct PluginPlacement {
    pub plugin_instance_id: PluginInstanceId,
    pub spec: PlacementSpec,
    pub revision: i64,
    pub updated_at: String,
    pub updated_by: String,
    pub placed: u32,
    pub detail: String,
    pub scheduled_at: String,
}
```

A stored placement: the operator's intent plus the last pass's status.

**Fields**

- `plugin_instance_id`: Placed instance.
- `spec`: Requested replicas and constraints.
- `revision`: Compare-and-swap revision of the intent.
- `updated_at`: RFC 3339 time of the last intent write.
- `updated_by`: Audit identity of the last intent writer.
- `placed`: Replicas the last scheduler pass placed.
- `detail`: Why the last pass placed fewer than requested; empty when satisfied.
- `scheduled_at`: RFC 3339 time the scheduler last changed the status; empty before
  the first pass.

Implements: `Clone`, `Debug`, `Eq`, `PartialEq`

### ScheduleReport

```rust
pub struct ScheduleReport {
    pub placements: usize,
    pub added: usize,
    pub removed: usize,
    pub conflicts: usize,
    pub handed_back: usize,
}
```

What one scheduler pass changed.

**Fields**

- `placements`: Placements considered.
- `added`: Deployments set `placed`.
- `removed`: Deployments set `unplaced`.
- `conflicts`: Deployment writes that lost a compare-and-swap (retried next pass).
- `handed_back`: Scheduler-owned rows of instances with no placement, handed back to
  the operator (a pass that raced a `--manual` or uninstall).

Implements: `Clone`, `Debug`, `Default`, `Eq`, `PartialEq`

### RemoteDeploymentEntry

```rust
pub struct RemoteDeploymentEntry {
    pub deployment: PluginDeployment,
    pub plugin_key: String,
    pub default_instance: bool,
    pub version: Option<PluginVersion>,
}
```

One deployment a Host should run, with what it runs.

**Fields**

- `deployment`: The desired row (targets the Host and should run).
- `plugin_key`: The instance's plugin key.
- `default_instance`: True when the instance is its plugin's default instance, which
  unpinned jobs use.
- `version`: The plugin's active version; `None` when nothing is installed.

Implements: `Clone`, `Debug`, `Eq`, `PartialEq`

### RemoteInstance

```rust
pub struct RemoteInstance {
    pub deployment: PluginDeployment,
    pub plugin_key: String,
    pub version: Option<PluginVersion>,
    pub trust: NodeTrust,
}
```

A deployed instance a Host asks about, with the node's trust.

**Fields**

- `deployment`: The deployment on the Host.
- `plugin_key`: The instance's plugin key.
- `version`: The plugin's active version.
- `trust`: What the node is trusted with now. Instance secrets belong to the
  cluster's operator (instances carry no owner yet), so only
  [`NodeTrust::All`](#nodetrust) receives them.

Implements: `Clone`, `Debug`, `Eq`, `PartialEq`

### RemoteObservation

```rust
pub struct RemoteObservation<'a> {
    pub deployment_id: &'a str,
    pub status: DeploymentStatus,
    pub detail: &'a str,
    pub applied_config_revision: Option<i64>,
}
```

One observation a Host reports.

**Fields**

- `deployment_id`: Deployment id.
- `status`: Observed status.
- `detail`: Detail (bounded on write).
- `applied_config_revision`: Config revision the Host applied, if any.

Implements: `Clone`, `Debug`, `Eq`, `PartialEq`

### RemoteApproval

```rust
pub struct RemoteApproval<'a> {
    pub cluster_id: &'a str,
    pub plugin_key: &'a str,
    pub version: &'a str,
    pub reviewed_request_json: &'a str,
    pub reviewed_artifact: RemoteArtifactRef<'a>,
    pub grant_json: &'a str,
    pub approver: RemoteApprover<'a>,
}
```

What an operator approves.

**Fields**

- `cluster_id`: Owner cluster id.
- `plugin_key`: Canonical plugin key.
- `version`: The requested version the operator reviewed.
- `reviewed_request_json`: The owner's grant the operator reviewed, exactly as recorded.
- `reviewed_artifact`: The archive the operator reviewed, exactly as recorded.
- `grant_json`: The grant consented to (already checked to be no wider than the
  reviewed request).
- `approver`: Who approves.

Implements: `Clone`, `Debug`, `Eq`, `PartialEq`

### RemoteArtifactRef

```rust
pub struct RemoteArtifactRef<'a> {
    pub url: &'a str,
    pub archive_sha256: &'a str,
    pub payload_root_sha256: &'a str,
}
```

The archive an owner asks this host to fetch: the version's artifact for
this host's target.

**Fields**

- `url`: Where the archive is fetched from (`blob:sha256:` over the owner
  link, or `https:`).
- `archive_sha256`: SHA-256 (lowercase hex) of the archive.
- `payload_root_sha256`: Payload-root digest of the tree the archive unpacks to: the key of
  this host's version cache, whose hit skips the fetch and the archive
  check, so it is part of what is approved.

#### RemoteArtifactRef::of

```rust
pub fn of(artifact: &'a PluginArtifact) -> Self
```

The consent view of a version artifact.

##### Arguments

* `artifact` - The artifact this host would materialize.

Implements: `Clone`, `Copy`, `Debug`, `Eq`, `PartialEq`

### RemotePluginConsent

```rust
pub struct RemotePluginConsent {
    pub owner_cluster_id: String,
    pub plugin_key: String,
    pub requested_version: String,
    pub requested_grant_json: String,
    pub requested_artifact_url: String,
    pub requested_artifact_sha256: String,
    pub requested_artifact_payload_sha256: String,
    pub version: Option<String>,
    pub approved_request_json: Option<String>,
    pub approved_artifact_url: Option<String>,
    pub approved_artifact_sha256: Option<String>,
    pub approved_artifact_payload_sha256: Option<String>,
    pub grant_json: Option<String>,
    pub approved_at: Option<String>,
    pub approved_by: Option<String>,
    pub revoked_at: Option<String>,
    pub updated_at: String,
}
```

One owner plugin of one owner cluster, as this install sees it.

**Fields**

- `owner_cluster_id`: Owner cluster id.
- `plugin_key`: Canonical plugin key.
- `requested_version`: Version the owner deploys here now.
- `requested_grant_json`: The owner's approved grant for that version (JSON).
- `requested_artifact_url`: URL of the archive the owner deploys to this host.
- `requested_artifact_sha256`: SHA-256 of that archive.
- `requested_artifact_payload_sha256`: Payload-root digest of the tree that archive unpacks to.
- `version`: Version this install's operator approved.
- `approved_request_json`: The owner's grant when the operator approved (JSON).
- `approved_artifact_url`: The archive URL when the operator approved.
- `approved_artifact_sha256`: The archive SHA-256 when the operator approved.
- `approved_artifact_payload_sha256`: The payload-root digest when the operator approved.
- `grant_json`: The grant the operator consented to (JSON).
- `approved_at`: RFC 3339 approval time.
- `approved_by`: Who approved ([`REMOTE_CONSENT_PLATFORM`](#remote_consent_platform) for an automatic approval).
- `revoked_at`: RFC 3339 time the operator revoked the approval; cleared only by an
  operator approval.
- `updated_at`: RFC 3339 time of the last write.

#### RemotePluginConsent::is_approved

```rust
pub fn is_approved(&self) -> bool
```

True when an approval is stored. Whether it still covers the
current request is the grant algebra's call.

#### RemotePluginConsent::is_revoked

```rust
pub fn is_revoked(&self) -> bool
```

True when the operator revoked the approval and has not approved
again: no automatic approval is given.

#### RemotePluginConsent::request_digest

```rust
pub fn request_digest(&self) -> String
```

The digest of the current request ([`remote_request_digest`](#remote_request_digest)); an
approval must name it.

#### RemotePluginConsent::requested_artifact

```rust
pub fn requested_artifact(&self) -> RemoteArtifactRef<'_>
```

The archive the current request asks this host to fetch.

#### RemotePluginConsent::approved_artifact_matches

```rust
pub fn approved_artifact_matches(&self, artifact: RemoteArtifactRef<'_>) -> bool
```

True when an approval is stored for exactly `artifact` (URL, archive
digest and payload-root digest). The current request's archive must
pass this before it is fetched or a cached tree is reused; an owner
that changed any of them needs approval again.

##### Arguments

* `artifact` - The archive about to be fetched.

Implements: `Clone`, `Debug`, `Eq`, `From<Model>`, `PartialEq`, `Serialize`

### JobSecret

```rust
pub struct JobSecret {
    pub secret_ref: JobSecretRef,
    pub value: Zeroizing<Vec<u8>>,
}
```

One unsealed secret handed with a job. The value is zeroized on drop and
never shown by `Debug`.

**Fields**

- `secret_ref`: Which row the value belongs to.
- `value`: The unsealed bytes.

Implements: `Clone`, `Debug`

### JobSecretRef

```rust
pub struct JobSecretRef {
    pub kind: String,
    pub provider: String,
    pub account_id: String,
    pub name: String,
}
```

Names one store-account secret a job is handed. Carries no secret
material.

**Fields**

- `kind`: Secret kind (`source_auth`, `widevine`, ...).
- `provider`: Plugin key that owns the secret.
- `account_id`: Store account id.
- `name`: Logical secret name.

Implements: `Clone`, `Debug`, `Eq`, `Hash`, `Ord`, `PartialEq`, `PartialOrd`

### RemoteClaimSettings

```rust
pub struct RemoteClaimSettings {
    pub lease_secs: u64,
    pub limits: JobClaimLimits,
}
```

What a Host's claim settles on: the routing and lease settings of the
owner node serving it.

**Fields**

- `lease_secs`: Lease length; permits are taken for the same time.
- `limits`: Cluster semaphore limits.

Implements: `Clone`, `Copy`, `Debug`

### RemoteJobClaim

```rust
pub struct RemoteJobClaim {
    pub job: JobRecord,
    pub secrets: Vec<JobSecret>,
    pub plugin_keys: Vec<String>,
    pub source_plugin_key: Option<String>,
    pub resumed: bool,
}
```

A job claimed for a Host, with the secrets to seal for it.

**Fields**

- `job`: The running job (its fence is on the record).
- `secrets`: The account secrets the job needs, unsealed.
- `plugin_keys`: The plugin keys the job runs ([`job_plugin_keys`](#job_plugin_keys)); empty when the
  job alone does not say (only possible without a claim filter).
- `source_plugin_key`: The one plugin whose source guest runs the job
  ([`job_source_plugin_key`](#job_source_plugin_key)); always set under a claim filter.
- `resumed`: The claim resumed a suspended job ([`RoutedJobClaim::resumed`](#routedjobclaim)).
  Pass it back to [`release_remote_job`](#release_remote_job) if the claim is given back.

Implements: `Debug`

### RequirementsSweepReport

```rust
pub struct RequirementsSweepReport {
    pub examined: u32,
    pub rewritten: u32,
    pub unsettled: u32,
    pub settled: u32,
    pub failed: u32,
    pub lost: u32,
}
```

What one sweep did.

**Fields**

- `examined`: Pending jobs whose requirements were re-resolved.
- `rewritten`: Jobs whose requirement set was rewritten.
- `unsettled`: Jobs newly marked, or still, unsettled within the grace period.
- `settled`: Jobs whose unsettled mark was cleared without another change.
- `failed`: Jobs failed as unroutable after the grace period.
- `lost`: Writes that lost their compare-and-swap (the job was claimed).

#### RequirementsSweepReport::changed

```rust
pub fn changed(&self) -> bool
```

True when the sweep wrote anything.

Implements: `Clone`, `Debug`, `Default`, `Eq`, `PartialEq`

### RouteRank

```rust
pub struct RouteRank {
    pub rank: usize,
    pub nodes: Vec<String>,
}
```

This node's position among the live nodes running one instance.

**Fields**

- `rank`: 0 for the best node.
- `nodes`: Live nodes running the instance, best first.

#### RouteRank::defer

```rust
pub fn defer(&self) -> Duration
```

How long this node lets a delivery wait before claiming it.

Implements: `Clone`, `Debug`, `Eq`, `PartialEq`

### PinnedPluginSchema

```rust
pub struct PinnedPluginSchema {
    pub plugin_key: String,
    pub manifest_id: String,
    pub schema_sha256: String,
    pub package_version: String,
    pub schema: SettingsSchema,
    pub recorded_at: String,
    pub recorded_by_node: String,
}
```

One `plugin_schemas` row.

**Fields**

- `plugin_key`: Canonical plugin key.
- `manifest_id`: Manifest id (display alias) at pin time.
- `schema_sha256`: Fingerprint of the normalized schema ([`schema_fingerprint`](#schema_fingerprint)).
- `package_version`: Semver package version that produced the pinned schema.
- `schema`: Declared settings.
- `recorded_at`: RFC 3339 time of the last pin.
- `recorded_by_node`: Node that recorded the last pin.

Implements: `Clone`, `Debug`, `Eq`, `PartialEq`

### SchemaPin

```rust
pub struct SchemaPin<'a> {
    pub plugin_key: &'a str,
    pub manifest_id: &'a str,
    pub package_version: &'a str,
    pub schema: &'a SettingsSchema,
    pub node_id: &'a str,
}
```

Inputs for [`pin_plugin_schema`](#pin_plugin_schema).

**Fields**

- `plugin_key`: Canonical plugin key.
- `manifest_id`: Manifest id (display alias).
- `package_version`: Semver package version of the local manifest.
- `schema`: Declared settings.
- `node_id`: Node recording the pin.

Implements: `Clone`, `Debug`

### SchemaPinOutcome

```rust
pub struct SchemaPinOutcome {
    pub schema: PinnedPluginSchema,
    pub change: SchemaPinChange,
}
```

Outcome of [`pin_plugin_schema`]: the row that is now pinned and how it
relates to the local manifest.

**Fields**

- `schema`: The pinned row. For [`SchemaPinChange::Stale`](#schemapinchange) this is the newer pin
  another node wrote, not the local manifest.
- `change`: What happened to the row.

Implements: `Clone`, `Debug`, `Eq`, `PartialEq`

### ClusterSecret

```rust
pub struct ClusterSecret {
    pub cluster_id: String,
    pub secret_fingerprint: String,
}
```

Cluster id and fingerprint of the secret root aligned for this process.

The unwrapped DEK stays in the process cache (`configure_master_key_with`).
Callers that need the key use [`crate::require_master_key`](master_key.md#require_master_key).

**Fields**

- `cluster_id`: Cluster id stored beside the fingerprint.
- `secret_fingerprint`: SHA-256 hex of the DEK.

Implements: `Clone`, `Debug`

### SemaphorePermit

```rust
pub struct SemaphorePermit {
    pub semaphore: String,
    pub slot: u32,
    pub lease: ClusterLease,
}
```

One held permit: a slot lease of a semaphore.

**Fields**

- `semaphore`: Semaphore name.
- `slot`: Slot index below the limit the permit was taken under.
- `lease`: The slot lease (holder, fencing token, expiry).

Implements: `Clone`, `Debug`, `Eq`, `PartialEq`

### SemaphoreRequest

```rust
pub struct SemaphoreRequest {
    pub name: String,
    pub limit: u32,
}
```

One semaphore a caller wants a permit of.

**Fields**

- `name`: Semaphore name (for example `jobs:network` or `account:<key>:<id>`).
- `limit`: Permits the semaphore grants cluster-wide (at least 1).

#### SemaphoreRequest::new

```rust
pub fn new(name: impl Into<String>, limit: u32) -> Self
```

A request for one permit of `name` under `limit`.

Implements: `Clone`, `Debug`, `Eq`, `Ord`, `PartialEq`, `PartialOrd`

### TitleNames

```rust
pub struct TitleNames {
    pub audio_stems: Vec<String>,
    pub part_patterns: Vec<String>,
}
```

The names a title's acquire writes on one destination, as the owner
plans them, all without their extension.

**Fields**

- `audio_stems`: The planned audio key of a single-file acquire (one per folder
  variant).
- `part_patterns`: Part and chapter keys, with [`UPLOAD_NAME_WILDCARD`](#upload_name_wildcard) and
  [`UPLOAD_NUMBER_WILDCARD`](#upload_number_wildcard) where the source's part or chapter title
  and number go: those are not known before the acquire runs.

#### TitleNames::admits

```rust
pub fn admits(&self, key: &str) -> bool
```

True when `key` is one of the title's audio keys with any extension
(a replacement of higher quality may change it) or a sidecar of one:
the audio stem, a `.` and any suffix (`.cue`, `.jpg`, `.pdf`,
`.chapters.flat.json`, `.m4b.bookclerk-meta.json`).

#### TitleNames::admits_audio

```rust
pub fn admits_audio(&self, key: &str) -> bool
```

True when `key` is one of the title's audio keys with any extension.

Implements: `Clone`, `Debug`, `Default`, `Eq`, `PartialEq`

### UploadDestination

```rust
pub struct UploadDestination {
    pub id: String,
    pub title_folders: Vec<String>,
    pub title_names: Option<TitleNames>,
}
```

One output destination of the receiving owner node an upload is
granted for, with where the title may be written there.

The owner computes the scope itself, from its own library row of the
granted title and that destination's naming options, exactly as a local
acquire would name the title; the Host's keys are never trusted to say
where a title belongs. A folder only this title's files use admits any
key below it; a folder several titles share (or the destination's
root) admits only the title's own planned names ([`TitleNames`](#titlenames)), so a
Host trusted with one title cannot claim another title's file there.

**Fields**

- `id`: The destination's id (`local`, `s3`).
- `title_folders`: The folders the title's files may be written under on this
  destination: a key must lie below one of them. An empty folder is
  the destination's root and admits only top-level keys. Empty means
  nothing may be written there.
- `title_names`: The title's planned names, when its folder is shared with other
  titles (or is the root): every key must then also be one of them.
  `None` when the folder is the title's own.

#### UploadDestination::admits

```rust
pub fn admits(&self, key: &str) -> bool
```

True when `key` lies in this destination's scope for the title.

#### UploadDestination::admits_primary

```rust
pub fn admits_primary(&self, key: &str) -> bool
```

True when `key` may be the title's primary audio key here: in scope
([`Self::admits`](#uploaddestinationadmits)) and, in a shared folder, one of the title's
planned audio keys with any extension ([`TitleNames::admits_audio`](#titlenamesadmits_audio)).

Implements: `Clone`, `Debug`, `Eq`, `PartialEq`

### UploadFinish

```rust
pub struct UploadFinish<'a> {
    pub detail: &'a str,
    pub download_episodes: bool,
}
```

How the job's closing upload completes it.

**Fields**

- `detail`: Outcome text recorded as the job's progress.
- `download_episodes`: Whether the receiving node acquires podcast episodes (its
  `library.download_episodes`); an episode it would skip does not
  keep the job open.

Implements: `Clone`, `Copy`, `Debug`

### UploadGrantRequest

```rust
pub struct UploadGrantRequest<'a> {
    pub book_uuid: &'a str,
    pub primary_key: &'a str,
    pub pdf_key: Option<&'a str>,
    pub files: &'a [UploadFile],
    pub destinations: &'a [UploadDestination],
}
```

What a Host asks to upload for one title.

**Fields**

- `book_uuid`: Library id of the title. Empty for the job's closing upload, which
  carries no files and only completes the job.
- `primary_key`: Storage key of the title's primary audio file on the primary
  destination (one of `files`' copies there).
- `pdf_key`: Storage key of the companion PDF on the primary destination (one of
  `files`' copies there), when uploaded.
- `files`: Every file, in upload order.
- `destinations`: The receiving owner node's output destinations, the primary first,
  each with the title's key scope there. Each must receive at least
  one file and every copy must name one of them and lie in its scope,
  so each destination is written only under the title's own keys (a
  closing upload passes none).

Implements: `Clone`, `Copy`, `Debug`

### UploadSession

```rust
pub struct UploadSession {
    pub upload_id: String,
    pub node_id: String,
    pub job_id: String,
    pub generation: i64,
    pub receiving_node_id: String,
    pub book_uuid: String,
    pub primary_key: String,
    pub pdf_key: Option<String>,
    pub destinations: Vec<String>,
    pub files: Vec<UploadFile>,
    pub received: BTreeSet<usize>,
    pub state: String,
    pub catalog_digest: Option<String>,
}
```

One open upload, as the upload endpoint sees it.

**Fields**

- `upload_id`: Upload id.
- `node_id`: Host node it was granted to.
- `job_id`: Acquire job.
- `generation`: Lease generation it is bound to.
- `receiving_node_id`: Owner node that receives the files.
- `book_uuid`: Library id of the title (empty for the job's closing upload).
- `primary_key`: Storage key of the primary audio file.
- `pdf_key`: Storage key of the companion PDF, when uploaded.
- `destinations`: The receiving node's destination ids at grant, the primary first;
  the files are written only while the node still has exactly these.
- `files`: The declared files.
- `received`: Indexes of the files every destination has.
- `state`: `open`, `committed` or `aborted`.
- `catalog_digest`: For a branch's catalog upload ([`Self::is_catalog`](#uploadsessionis_catalog)): the title's
  manifest digest at grant.

#### UploadSession::is_catalog

```rust
pub fn is_catalog(&self) -> bool
```

True for a branch's catalog upload, bound to the node's catalog
fence instead of a job's.

#### UploadSession::complete

```rust
pub fn complete(&self) -> bool
```

True once every declared file was received.

Implements: `Clone`, `Debug`

### PluginArtifact

```rust
pub struct PluginArtifact {
    pub target: String,
    pub url: String,
    pub format: String,
    pub archive_root: String,
    pub archive_sha256: String,
    pub payload_root_sha256: String,
    pub executable: String,
}
```

One downloadable artifact of a version.

**Fields**

- `target`: Bookclerk target (`linux-x64-gnu`, …; `any` for workerd packages).
- `url`: Where nodes fetch the archive: `https:`, `file:`, or
  `blob:sha256:<hex>`.
- `format`: Archive format (`tar.gz` or `zip`).
- `archive_root`: Path inside the archive to the plugin root.
- `archive_sha256`: Lowercase hex SHA-256 of the archive bytes.
- `payload_root_sha256`: Payload-root digest of the extracted plugin tree; the cache directory
  is named after it.
- `executable`: Relative path of the executable inside the plugin root.

Implements: `Clone`, `Debug`, `Deserialize<'de>`, `Eq`, `PartialEq`, `Serialize`

### PluginInstall

```rust
pub struct PluginInstall {
    pub plugin_key: String,
    pub version: String,
    pub revision: i64,
    pub updated_at: String,
    pub updated_by: String,
}
```

The active version pointer of one plugin key.

**Fields**

- `plugin_key`: Canonical plugin key.
- `version`: Active version.
- `revision`: Compare-and-swap revision.
- `updated_at`: RFC 3339 time of the last move.
- `updated_by`: Audit identity of the last writer.

Implements: `Clone`, `Debug`, `Eq`, `PartialEq`

### PluginVersion

```rust
pub struct PluginVersion {
    pub plugin_key: String,
    pub version: String,
    pub manifest_id: String,
    pub manifest_toml: String,
    pub settings: SettingsSchema,
    pub artifacts: Vec<PluginArtifact>,
    pub publisher_verified: bool,
    pub grant_json: String,
    pub coordinate: String,
    pub installed_at: String,
    pub installed_by: String,
}
```

One installed version of a plugin.

**Fields**

- `plugin_key`: Canonical plugin key.
- `version`: Semver package version.
- `manifest_id`: Manifest id (display alias).
- `manifest_toml`: `plugin.toml` text.
- `settings`: Settings schema the version declares.
- `artifacts`: Per-target artifacts.
- `publisher_verified`: True when the publisher was verified at install time.
- `grant_json`: Approved consent grant (JSON), empty until an operator approves it.
- `coordinate`: Package coordinate the version was resolved from; `update` resolves
  newer versions from it. Empty for a local archive.
- `installed_at`: RFC 3339 install time.
- `installed_by`: Audit identity of the publisher.

#### PluginVersion::artifact_for

```rust
pub fn artifact_for(&self, target: &str) -> Option<&PluginArtifact>
```

The artifact for `target`, falling back to a target-independent
(`any`) artifact.

Implements: `Clone`, `Debug`, `Eq`, `PartialEq`

### EventsSettingsV1

```rust
pub struct EventsSettingsV1 {
    pub retention_days: u64,
    pub dead_letter_retention_days: u64,
    pub concurrency: u32,
}
```

Typed `[events]` document, schema version 1.

**Fields**

- `retention_days`: Days to keep acked and rejected deliveries.
- `dead_letter_retention_days`: Days to keep dead-lettered deliveries.
- `concurrency`: Local delivery workers and the cluster in-flight cap.

#### EventsSettingsV1::validate

```rust
pub fn validate(&self) -> Result<()>
```

Rejects values outside the domain bounds.

##### Errors

Returns an error when a field is outside its allowed range.

#### EventsSettingsV1::from_config

```rust
pub fn from_config(config: &EventsConfig) -> Result<Self>
```

Copies the transitional `[events]` table, rejecting values the domain forbids.

##### Errors

Returns an error when the table fails [`Self::validate`](#eventssettingsv1validate).

#### EventsSettingsV1::apply_to

```rust
pub fn apply_to(&self, config: &mut EventsConfig)
```

Writes the document onto the in-memory events table.

Implements: `Clone`, `Debug`, `Deserialize<'de>`, `Eq`, `PartialEq`, `Serialize`

### NodeRuntimeSettingsV1

```rust
pub struct NodeRuntimeSettingsV1 {
    pub label: String,
    pub cpu_budget_percent: u8,
    pub memory_budget_percent: u8,
    pub schedulable: bool,
}
```

Typed node-scoped runtime document, schema version 1.

Desired state for one node, editable from any node in the cluster: an
operator label, and the budget the scheduler may commit on this node.
Budgets are shares of the capacity the node detects at heartbeat
(CPU cores and memory, the tighter of the machine and its cgroup).

**Fields**

- `label`: Operator label shown for this node. Empty is allowed.
- `cpu_budget_percent`: Share of detected CPU cores the scheduler may commit to plugin
  guests on this node (1..=100).
- `memory_budget_percent`: Share of detected memory the scheduler may commit (1..=100).
- `schedulable`: False drains the node: the scheduler places nothing new here and
  moves its replicas elsewhere. Hand-managed deployments still run.

#### NodeRuntimeSettingsV1::new

```rust
pub fn new() -> Self
```

Empty label, default budgets, schedulable.

#### NodeRuntimeSettingsV1::validate

```rust
pub fn validate(&self) -> Result<()>
```

Rejects control characters, labels longer than 64 bytes, and budgets
outside 1..=100.

##### Errors

Returns an error when a field is out of range.

Implements: `Clone`, `Debug`, `Default`, `Deserialize<'de>`, `Eq`, `PartialEq`, `Serialize`

### ConfigurationDocument

```rust
pub struct ConfigurationDocument<T> {
    pub body: T,
    pub revision: i64,
    pub schema_version: i64,
    pub updated_at: String,
    pub updated_by: String,
    pub scope_type: String,
    pub scope_id: String,
    pub namespace: String,
}
```

A parsed domain document plus its revision metadata.

**Fields**

- `body`: Parsed body.
- `revision`: Compare-and-swap revision.
- `schema_version`: Domain schema version.
- `updated_at`: RFC 3339 commit time.
- `updated_by`: Audit identity of the last writer.
- `scope_type`: Scope type (`cluster` or `node`).
- `scope_id`: Scope id (`singleton` or a node id).
- `namespace`: Domain namespace.

Implements: `Clone`, `Debug`, `Eq`, `PartialEq`

### ControlPlaneSession

```rust
pub struct ControlPlaneSession {
    pub node: NodeRecord,
    pub cluster_id: String,
    pub events: ConfigurationDocument<EventsSettingsV1>,
    pub output: ConfigurationDocument<OutputSettingsV1>,
    pub node_runtime: ConfigurationDocument<NodeRuntimeSettingsV1>,
}
```

Identity, secret root, and the authoritative events document for this process.

**Fields**

- `node`: Registered node.
- `cluster_id`: Cluster id from the secret root.
- `events`: Authoritative `[events]` document.
- `output`: Authoritative shared `[output]` document.
- `node_runtime`: This node's runtime document. Other nodes' documents are not included.

Implements: `Clone`, `Debug`

## Enums

### CatalogAccept

```rust
pub enum CatalogAccept {
    Accepted { upserted: u64, skipped: u64, wanted: Vec<CatalogRef> },
    Lost,
    Refused(&'static str),
}
```

Outcome of [`accept_catalog_batch`](#accept_catalog_batch).

**Variants**

- `Accepted`: The batch was recorded.
  - `upserted`: Titles recorded.
  - `skipped`: Titles skipped because this library already holds them for the
    branch's owner (no echo back).
  - `wanted`: Titles whose files this library wants.
- `Lost`: The fence is not live (or the branch no longer shares or is no
  longer trusted); nothing was recorded.
- `Refused`: The batch is not allowed; nothing was recorded.

Implements: `Clone`, `Debug`, `Eq`, `PartialEq`

### CatalogBegin

```rust
pub enum CatalogBegin {
    Opened(CatalogSessionOpened),
    Refused(&'static str),
}
```

Outcome of [`begin_catalog`](#begin_catalog).

**Variants**

- `Opened`: The session is open under a new fence.
- `Refused`: The node is not a branch this library trusts.

Implements: `Clone`, `Debug`, `Eq`, `PartialEq`

### DeploymentReplace

```rust
pub enum DeploymentReplace {
    Applied(PluginDeployment),
    Conflict { current_revision: i64 },
    Replayed { revision: i64 },
}
```

Outcome of [`replace_plugin_deployment`](#replace_plugin_deployment).

**Variants**

- `Applied`: The desired row is the committed body.
- `Conflict`: `expected_revision` was not current. No observation row was written.
  - `current_revision`: Revision currently stored.
- `Replayed`: This operation id already committed at `revision`.
  - `revision`: Revision of the original commit.

Implements: `Clone`, `Debug`, `Eq`, `PartialEq`

### DeploymentStatus

```rust
pub enum DeploymentStatus {
    Materialized,
    Running,
    Healthy,
    Error,
    Retired,
}
```

Observed deployment status.

**Variants**

- `Materialized`: The node has the plugin's tree (materialized from the installed
  version, or host-local); the spawn comes next.
- `Running`: Session spawn returned. Health is not confirmed on this tick.
- `Healthy`: Health succeeded with the applied config revision.
- `Error`: Install, grant, config, secrets, spawn, or health failed.
- `Retired`: Desired state is `absent`; the node stopped the guest.

#### DeploymentStatus::as_str

```rust
pub fn as_str(self) -> &'static str
```

Stored text.

#### DeploymentStatus::parse

```rust
pub fn parse(raw: &str) -> Result<Self>
```

Parses stored text.

##### Errors

Returns an error when `raw` is not one of the five statuses.

Implements: `Clone`, `Copy`, `Debug`, `Eq`, `PartialEq`

### DesiredState

```rust
pub enum DesiredState {
    Present,
    Absent,
    Placed,
    Unplaced,
}
```

What a deployment row asks its node to do, and who decided it.

Operators write [`DesiredState::Present`](#desiredstate) and [`DesiredState::Absent`](#desiredstate)
(`plugins deploy|undeploy`, the Settings switch). The placement scheduler
writes [`DesiredState::Placed`](#desiredstate) and [`DesiredState::Unplaced`](#desiredstate) for an
instance it owns. A node runs the instance for `present` and `placed`
and retires it for `absent` and `unplaced`; the split only records whose
decision the row carries, so "the operator turned this off" never reads
like "the scheduler put the replica elsewhere".

**Variants**

- `Present`: An operator enabled the instance on this node.
- `Absent`: An operator disabled the instance on this node.
- `Placed`: The scheduler placed a replica on this node.
- `Unplaced`: The scheduler holds no replica on this node (scaled away, node no
  longer qualifies, or not chosen).

#### DesiredState::ALL

```rust
pub const ALL: [Self; 4] = _
```

Every state, in stored-text order.

#### DesiredState::as_str

```rust
pub fn as_str(self) -> &'static str
```

Stored text.

#### DesiredState::parse

```rust
pub fn parse(raw: &str) -> Result<Self>
```

Parses stored or API text.

##### Errors

Returns an error when `raw` names no state.

#### DesiredState::runs

```rust
pub fn runs(self) -> bool
```

True when the node should run the instance.

#### DesiredState::is_scheduled

```rust
pub fn is_scheduled(self) -> bool
```

True for the scheduler's states.

#### DesiredState::operator

```rust
pub fn operator(enabled: bool) -> Self
```

The operator state for an enable (`true`) or disable switch.

#### DesiredState::scheduled

```rust
pub fn scheduled(placed: bool) -> Self
```

The scheduler state for a replica kept (`true`) or not.

#### DesiredState::handed_back

```rust
pub fn handed_back(self) -> Self
```

The operator state that keeps what this row does today: `placed`
becomes `present` and `unplaced` becomes `absent` when an instance
is handed back from the scheduler.

Implements: `Clone`, `Copy`, `Debug`, `Display`, `Eq`, `Hash`, `Ord`, `PartialEq`, `PartialOrd`

### ConfigActor

```rust
pub enum ConfigActor {
    Operator { id: String },
    Administrator { id: String },
    Member { id: String },
    Bootstrap,
    Scheduler,
    NodeOwner(VerifiedNodeOwner),
    ClusterOwner(VerifiedClusterOwner),
}
```

Who is allowed to write a configuration document.

**Variants**

- `Operator`: Control-plane operator (CLI or operator token).
  - `id`: Stable audit id (`cli`, `operator`, …).
- `Administrator`: Portal administrator. Cannot write cluster or node configuration.
  - `id`: User id recorded if a write is attempted.
- `Member`: Portal member. Cannot write configuration.
  - `id`: User id recorded if a write is attempted.
- `Bootstrap`: Process startup. May import a missing document and may rewrite an
  instance document to a newer pinned schema; it is never constructed
  for an API caller.
- `Scheduler`: The placement scheduler (one daemon, under the `scheduler` lease). It
  writes only `placed` / `unplaced` deployments; it is never
  constructed for an API caller.
- `NodeOwner`: The user who owns one node. Writes that node's `node.runtime`
  document only. Built by [`super::verify_node_owner`](#verify_node_owner), which checks
  `nodes.owner_user_id`, so a caller cannot claim a node it does not own.
- `ClusterOwner`: An active portal Owner. Writes the cluster's host-enrollment switch
  (`core.host_enrollment`) only; every other document stays an
  operator write. Built by [`super::verify_cluster_owner`](#verify_cluster_owner), which reads
  the user's stored role and status.

Implements: `Clone`, `Debug`, `Eq`, `PartialEq`

### ReplaceOutcome

```rust
pub enum ReplaceOutcome {
    Applied(StoredDocument),
    Conflict { current_revision: i64 },
    Replayed { revision: i64 },
}
```

Result of a compare-and-swap replacement.

**Variants**

- `Applied`: The document is the committed body.
- `Conflict`: The expected revision lost. No document, audit, or change row was written.
  - `current_revision`: Revision currently stored.
- `Replayed`: This `operation_id` already committed. The document was not modified again.
  - `revision`: Revision stored on the original receipt.

Implements: `Clone`, `Debug`, `Eq`, `PartialEq`

### EnrollmentActor

```rust
pub enum EnrollmentActor {
    Operator { id: String },
    User { user_id: i64 },
}
```

Who is acting on enrollment links and node credentials.

A `User` is resolved from the database on every call: the role and
status the library reads, not what the caller claims, decide what it may
do.

**Variants**

- `Operator`: Control-plane operator (CLI or operator token).
  - `id`: Stable audit id (`cli`, `operator`, …).
- `User`: A signed-in portal user.
  - `user_id`: Calling user.

Implements: `Clone`, `Debug`, `Eq`, `PartialEq`

### NodeKind

```rust
pub enum NodeKind {
    Owner,
    Host,
}
```

How a node joined the cluster.

**Variants**

- `Owner`: Shares the library database and `master.key`. Every node is one today.
- `Host`: Keeps its own database and operator, and enrolled to contribute work.

#### NodeKind::as_str

```rust
pub fn as_str(self) -> &'static str
```

Stored text.

#### NodeKind::parse

```rust
pub fn parse(raw: &str) -> Self
```

Parses stored text; anything unknown reads as [`Self::Owner`](#nodekind).

Implements: `Clone`, `Copy`, `Debug`, `Deserialize<'de>`, `Eq`, `PartialEq`, `Serialize`

### InstanceConfigReplace

```rust
pub enum InstanceConfigReplace {
    Applied(InstanceConfigDocument),
    Conflict { current_revision: i64 },
    Replayed { revision: i64 },
}
```

Outcome of [`replace_instance_config`](#replace_instance_config).

**Variants**

- `Applied`: The new document is committed.
- `Conflict`: `expected_revision` was not current. No audit row was written.
  - `current_revision`: Revision currently stored.
- `Replayed`: This operation id already committed at `revision`.
  - `revision`: Revision of the original commit.

Implements: `Clone`, `Debug`, `PartialEq`

### SettingValue

```rust
pub enum SettingValue {
    Bool(bool),
    Number(Number),
    String(String),
}
```

One scalar setting. Nested objects and arrays are not representable.

**Variants**

- `Bool`: JSON boolean.
- `Number`: Finite JSON number.
- `String`: JSON string, at most 1024 bytes.

Implements: `Clone`, `Debug`, `Deserialize<'de>`, `PartialEq`, `Serialize`

### LiveTopic

```rust
pub enum LiveTopic {
    Library,
    Jobs,
    Deliveries,
    Nodes,
    Config,
    Consents,
    Clusters,
    Users,
}
```

A topic of the live socket.

**Variants**

- `Library`: The library's titles (`books`).
- `Jobs`: Jobs (`jobs`).
- `Deliveries`: Event deliveries (`event_deliveries`).
- `Nodes`: Node heartbeats (`node_status`).
- `Config`: Cluster configuration documents (`configuration_changes`).
- `Consents`: Owner-plugin consent on this host (`remote_plugin_consents`).
- `Clusters`: Owner clusters this host joined, and their state (`cluster_links`).
- `Users`: Users (`users`).

#### LiveTopic::ALL

```rust
pub const ALL: [Self; 8] = _
```

Every topic.

#### LiveTopic::as_str

```rust
pub fn as_str(self) -> &'static str
```

The topic's wire name.

Implements: `Clone`, `Copy`, `Debug`, `Eq`, `Hash`, `Ord`, `PartialEq`, `PartialOrd`

### NodeMessage

```rust
pub enum NodeMessage {
    DeploymentChanged { deployment_id: String, plugin_instance_id: String, desired: String, revision: i64 },
    InstallChanged { plugin_key: String, version: String, revision: i64 },
    PlacementChanged { subject: String, revision: i64 },
    ConfigurationChanged { scope_type: String, scope_id: String, namespace: String, revision: i64 },
    NodeRevoked { node_id: String },
}
```

One message on a `node.<node_id>` queue. The body is the JSON of this
enum (`{"type": "deployment.changed", ...}`).

**Variants**

- `DeploymentChanged`: A deployment targeting this node was created, replaced, or deleted.
  - `deployment_id`: Deployment row id.
  - `plugin_instance_id`: Instance the deployment runs.
  - `desired`: `present`, `absent`, or `deleted`.
  - `revision`: Revision after the write (0 for a delete).
- `InstallChanged`: The active version of a plugin moved (install, update, rollback,
  uninstall) or its consent grant changed.
  - `plugin_key`: Canonical plugin key.
  - `version`: Active version; empty when the pointer was cleared.
  - `revision`: Pointer revision (0 for a grant change or a clear).
- `PlacementChanged`: Placement intent, or a fact it depends on, changed: an instance's
  replicas or constraints, or a node's labels. Only the node holding
  the scheduler lease acts on it; every other node ignores it.
  - `subject`: `instance:<id>` or `node:<id>`.
  - `revision`: Placement revision (0 for a node label change or a delete).
- `ConfigurationChanged`: A configuration document changed (cluster settings or an instance
  document).
  - `scope_type`: Document scope type.
  - `scope_id`: Document scope id.
  - `namespace`: Document namespace.
  - `revision`: Revision after the write.
- `NodeRevoked`: A host node's credential was revoked. The node holding that host's
  connection closes it; every claim, renew, and upload re-reads
  `node_credentials.revoked_at` as well, so a lost message does not
  keep the host connected to work.
  - `node_id`: Revoked host node.

Implements: `Clone`, `Debug`, `Deserialize<'de>`, `Eq`, `PartialEq`, `Serialize`

### OutputReplace

```rust
pub enum OutputReplace {
    Applied(Box<ConfigurationDocument<OutputSettingsV1>>),
    Conflict { current_revision: i64 },
    Replayed { revision: i64 },
}
```

Outcome of [`replace_output`](#replace_output).

**Variants**

- `Applied`: The new document is committed.
- `Conflict`: `expected_revision` was not current. The stored document is unchanged.
  - `current_revision`: Revision currently stored.
- `Replayed`: This operation id already committed at `revision`.
  - `revision`: Revision of the original commit.

Implements: `Clone`, `Debug`, `Eq`, `PartialEq`

### PlacementWrite

```rust
pub enum PlacementWrite {
    Applied(PluginPlacement),
    Conflict { current_revision: i64 },
}
```

Outcome of [`put_plugin_placement`](#put_plugin_placement).

**Variants**

- `Applied`: The intent is stored at the returned revision.
- `Conflict`: `expected_revision` was not current (0 when no placement exists).
  - `current_revision`: Revision currently stored.

Implements: `Clone`, `Debug`, `Eq`, `PartialEq`

### Replicas

```rust
pub enum Replicas {
    Count(u32),
    All,
}
```

How many nodes should run a placed instance.

**Variants**

- `Count`: Exactly this many nodes (zero keeps the placement and stops every
  replica).
- `All`: Every node that qualifies and has room; calls then go to the best of
  them. Suits nodes that come and go.

Implements: `Clone`, `Copy`, `Debug`, `Default`, `Deserialize<'de>`, `Display`, `Eq`, `FromStr`, `Hash`, `PartialEq`, `Serialize`

### RemoteApprover

```rust
pub enum RemoteApprover<'a> {
    Operator(&'a str),
    Platform,
}
```

Who gives an approval.

**Variants**

- `Operator`: This install's operator, by audit identity (never
  [`REMOTE_CONSENT_PLATFORM`](#remote_consent_platform)). Clears a recorded revoke.
- `Platform`: The automatic approval of a bundled platform plugin. Refused while
  a revoke is recorded.

#### RemoteApprover::audit_identity

```rust
pub fn audit_identity(&self) -> &str
```

The audit identity stored as `approved_by`.

Implements: `Clone`, `Copy`, `Debug`, `Eq`, `PartialEq`

### ScanAccept

```rust
pub enum ScanAccept {
    Accepted { upserted: u64 },
    Lost,
    Refused(&'static str),
}
```

Outcome of [`accept_scan_results`](#accept_scan_results).

**Variants**

- `Accepted`: The batch was written.
  - `upserted`: Rows written.
- `Lost`: The fence no longer owns the job, or the node is no longer trusted
  with it; nothing was written.
- `Refused`: The batch is not allowed; nothing was written (the reason is
  stable).

Implements: `Clone`, `Copy`, `Debug`, `Eq`, `PartialEq`

### RemoteFinish

```rust
pub enum RemoteFinish {
    Done,
    Lost,
    Refused(&'static str),
}
```

Outcome of [`finish_remote_job`](#finish_remote_job).

**Variants**

- `Done`: The job row was finalized and its permits given back.
- `Lost`: The fence no longer owns the job.
- `Refused`: The request is not allowed for this job (the reason is stable).

Implements: `Clone`, `Copy`, `Debug`, `Eq`, `PartialEq`

### RemoteOutcome

```rust
pub enum RemoteOutcome<'a> {
    Complete(Option<&'a str>),
    Fail { kind: &'a str, message: &'a str },
}
```

How a Host's job ended.

**Variants**

- `Complete`: The job succeeded; the text is recorded as its progress.
- `Fail`: The job failed.
  - `kind`: Stable error kind.
  - `message`: Operator-facing message.

Implements: `Clone`, `Copy`, `Debug`

### RemoteRenew

```rust
pub enum RemoteRenew {
    Renewed { cancel: bool },
    Lost,
}
```

Outcome of [`renew_remote_job`](#renew_remote_job).

**Variants**

- `Renewed`: The lease and its permits were extended.
  - `cancel`: The operator asked to cancel the job.
- `Lost`: The fence no longer owns the job, a permit was taken over, or the
  node is no longer trusted with the job.

Implements: `Clone`, `Copy`, `Debug`, `Eq`, `PartialEq`

### WriteBack

```rust
pub enum WriteBack {
    Written,
    Refused(&'static str),
}
```

Outcome of [`write_back_job_secret`](#write_back_job_secret).

**Variants**

- `Written`: The secret was re-sealed and stored.
- `Refused`: Refused; nothing was written (the reason is stable).

Implements: `Clone`, `Copy`, `Debug`, `Eq`, `PartialEq`

### SchemaPinChange

```rust
pub enum SchemaPinChange {
    Unchanged,
    Pinned,
    Upgraded { previous_version: String },
    Repinned,
    Stale { local_version: String },
    RolledBack { previous_version: String },
}
```

How [`pin_plugin_schema`](#pin_plugin_schema) changed the row.

**Variants**

- `Unchanged`: The row already matched the local manifest and version.
- `Pinned`: No node had pinned this plugin before.
- `Upgraded`: The local package is newer than the pin; the row now carries it.
  Callers prune instance documents to the new schema.
  - `previous_version`: Version the row carried before this pin.
- `Repinned`: Same version, different manifest (an unreleased edit); re-pinned in
  place. Callers prune instance documents as for an upgrade.
- `Stale`: The local package is older than the pin. The row was not touched and
  the plugin must not be spawned by this node.
  - `local_version`: The local manifest's version.
- `RolledBack`: The active release moved to an older version (a rollback); the row
  follows it. Callers prune instance documents as for an upgrade.
  - `previous_version`: Version the row carried before this pin.

Implements: `Clone`, `Debug`, `Eq`, `PartialEq`

### PermitAcquire

```rust
pub enum PermitAcquire {
    Granted(Vec<SemaphorePermit>),
    Blocked(String),
}
```

Outcome of [`acquire_permits`](#acquire_permits).

**Variants**

- `Granted`: Every requested permit is held.
- `Blocked`: The named semaphore had no free slot; nothing is held.

Implements: `Clone`, `Debug`, `Eq`, `PartialEq`

### Claimant

```rust
pub enum Claimant {
    Local,
    Remote,
}
```

How the claiming node reaches the job queue.

**Variants**

- `Local`: A worker in this process, which holds the library database itself.
  A node id without a `nodes` row is trusted as such a worker.
- `Remote`: A node that authenticated over the node API. It must have a `nodes`
  row and, unless it is an owner node, a live node credential.

Implements: `Clone`, `Copy`, `Debug`, `Default`, `Eq`, `PartialEq`

### NodeTrust

```rust
pub enum NodeTrust {
    All,
    OwnedBy(i64),
    Nothing,
}
```

What a node is trusted with, resolved from the database.

**Variants**

- `All`: Every job: an owner node, or a Host owned by an active Owner or
  Administrator.
- `OwnedBy`: Only jobs whose secrets all belong to this user.
- `Nothing`: No job at all.

Implements: `Clone`, `Debug`, `Eq`, `PartialEq`

### ScanRequester

```rust
pub enum ScanRequester {
    Operator,
    User(i64),
}
```

Who asked for a scan.

**Variants**

- `Operator`: Operator authority: the operator token or session, the daemon's own
  scheduler, or a daemon with authentication turned off.
- `User`: A signed-in user; their role is read from the database.

Implements: `Clone`, `Copy`, `Debug`, `Eq`, `PartialEq`

### UploadCommit

```rust
pub enum UploadCommit {
    Committed { job_completed: bool },
    Lost,
    Refused(&'static str),
}
```

Outcome of [`commit_upload`](#commit_upload).

**Variants**

- `Committed`: The title is committed (now or by an earlier identical commit).
  - `job_completed`: The job is finished.
- `Lost`: The fence was lost before the commit; nothing was written.
- `Refused`: Not allowed (the reason is stable).

Implements: `Clone`, `Copy`, `Debug`, `Eq`, `PartialEq`

### UploadGrant

```rust
pub enum UploadGrant {
    Granted { upload_id: String, token: Zeroizing<String>, expires_at: DateTime<Utc>, superseded: Vec<UploadSession> },
    Lost,
    Refused(&'static str),
}
```

Outcome of [`grant_upload`](#grant_upload).

**Variants**

- `Granted`: The upload is open.
  - `upload_id`: Upload id.
  - `token`: The bearer token (zeroized on drop; seal it before it leaves
    the process).
  - `expires_at`: When the token stops being accepted unless extended.
  - `superseded`: Earlier open uploads of the same title by the same job, aborted
    by this grant, as they stood; the receiving node deletes what
    each had received.
- `Lost`: The fence no longer owns the job, or the node is no longer trusted
  with it.
- `Refused`: Not allowed (the reason is stable).

Implements: `Debug`

### AddVersion

```rust
pub enum AddVersion {
    Created(PluginVersion),
    Existing(PluginVersion),
}
```

What [`add_plugin_version`](#add_plugin_version) did.

**Variants**

- `Created`: A new version row was written.
- `Existing`: The same `(plugin_key, version)` with the same artifacts already
  existed; nothing was written.

#### AddVersion::version

```rust
pub fn version(&self) -> &PluginVersion
```

The version row, created or existing.

Implements: `Clone`, `Debug`, `Eq`, `PartialEq`

### EventsReplace

```rust
pub enum EventsReplace {
    Applied(ConfigurationDocument<EventsSettingsV1>),
    Conflict { current_revision: i64 },
    Replayed { revision: i64 },
}
```

Outcome of [`replace_events`](#replace_events).

**Variants**

- `Applied`: The new document is committed.
- `Conflict`: `expected_revision` was not current. The stored document is unchanged.
  - `current_revision`: Revision currently stored.
- `Replayed`: This operation id already committed at `revision`.
  - `revision`: Revision of the original commit.

Implements: `Clone`, `Debug`, `Eq`, `PartialEq`

### NodeRuntimeReplace

```rust
pub enum NodeRuntimeReplace {
    Applied(ConfigurationDocument<NodeRuntimeSettingsV1>),
    Conflict { current_revision: i64 },
    Replayed { revision: i64 },
}
```

Outcome of [`replace_node_runtime`](#replace_node_runtime).

**Variants**

- `Applied`: The new document is committed.
- `Conflict`: `expected_revision` was not current.
  - `current_revision`: Revision currently stored.
- `Replayed`: This operation id already committed at `revision`.
  - `revision`: Revision of the original commit.

Implements: `Clone`, `Debug`, `Eq`, `PartialEq`

## Functions

### accept_catalog_batch

```rust
pub async fn accept_catalog_batch(
    store: &LibraryStore,
    node_id: &str,
    fence: i64,
    cursor: &str,
    done: bool,
    titles: &[CatalogTitle],
) -> Result<CatalogAccept>
```

Records one batch of `node_id`'s shared titles under `fence`, then the
cursor after it. Every title is checked before any is written; a title
this library already holds for the branch's owner is skipped. With
`done` (the last batch of a pass) while a pass from the start is under
way, titles the pass did not name are withdrawn (detached); `done` on
a resumed incremental pass changes nothing more.

Every write runs in one transaction, each statement gated on the fence
still being live, the branch still sharing and its credential live: a
concurrent revoke, leave or stop-sharing either lands after the whole
batch (and detaches what it recorded) or the batch records nothing.

#### Errors

Returns an error when a read or write fails.

### begin_catalog

```rust
pub async fn begin_catalog(
    store: &LibraryStore,
    node_id: &str,
    share_books: bool,
    include_branches: bool,
) -> Result<CatalogBegin>
```

Opens (or reopens) `node_id`'s catalog sync with the settings the
branch reports: records them on its node row, bumps its catalog fence
(frames under an older fence are lost from now on), and answers where
to resume. A change of either setting restarts the pass from the start,
so titles the new setting excludes are withdrawn when the pass ends; so
does a session [`FULL_CATALOG_PASS_EVERY`](#full_catalog_pass_every) after the last full pass.
With sharing off, every title the branch shared is marked detached
(`stopped_sharing`).

#### Errors

Returns an error when a read or write fails.

### branch_provenance

```rust
pub fn branch_provenance(node_id: &str) -> String
```

The provenance text of titles shared by `node_id`.

### branch_title_counts

```rust
pub async fn branch_title_counts(
    store: &LibraryStore,
    node_id: &str,
) -> Result<BranchTitleCounts>
```

[`BranchTitleCounts`](#branchtitlecounts) of `node_id`.

#### Errors

Returns the read error.

### catalog_fence_live

```rust
pub async fn catalog_fence_live(
    store: &LibraryStore,
    node_id: &str,
    fence: i64,
) -> Result<bool>
```

True while `fence` is `node_id`'s live catalog fence and it may share.

#### Errors

Returns an error when a read fails.

### detach_branch_titles

```rust
pub async fn detach_branch_titles(
    store: &LibraryStore,
    node_id: &str,
    reason: &str,
) -> Result<u64>
```

Marks every title `node_id` shares detached with `reason`; answers how
many changed. Files already delivered stay where they are.

#### Errors

Returns the write error.

### forwardable_branch_titles

```rust
pub async fn forwardable_branch_titles(store: &LibraryStore) -> Result<Vec<Model>>
```

On a branch: the titles its own branches shared with it that it may
pass on (still shared, files delivered), oldest first.

#### Errors

Returns the read error.

### grant_catalog_upload

```rust
pub async fn grant_catalog_upload(
    store: &LibraryStore,
    node_id: &str,
    fence: i64,
    receiving_node_id: &str,
    store_id: &str,
    product_id: &str,
    files: &[UploadFile],
    destinations: &[String],
) -> Result<UploadGrant>
```

Opens an upload of one shared title's files, bound to `node_id`'s
catalog `fence` and received by `receiving_node_id`.

Granted only while the fence is live, for a title the branch shares,
when every file is copied to each of `destinations` (this node's, the
primary first) under its manifest key, the files are exactly the
recorded manifest, no key is one another title here already uses
(`key_in_use`: the other title's audio, PDF or primary stem;
`key_of_another_book`: any of a library book's keys or their
sidecars), and no key lies in or below a folder holding another title's
audio (`folder_of_another_book`: a library book's, or another branch's
delivered title's). The keys are then claimed for this upload
in `title_files`, refused when another title holds one there.

#### Errors

Returns an error when a read or the insert fails.

### list_branch_titles

```rust
pub async fn list_branch_titles(
    store: &LibraryStore,
    node_id: Option<&str>,
    limit: u64,
) -> Result<Vec<BranchTitle>>
```

Titles branches shared, of one branch or all, oldest first, at most
`limit`.

#### Errors

Returns the read error.

### pending_deliveries

```rust
pub async fn pending_deliveries(
    store: &LibraryStore,
    node_id: &str,
    limit: usize,
) -> Result<Vec<CatalogRef>>
```

Up to `limit` of `node_id`'s shared titles whose files have not
arrived (or changed since).

#### Errors

Returns the read error.

### remove_detached_branch_titles

```rust
pub async fn remove_detached_branch_titles(
    store: &LibraryStore,
    node_id: Option<&str>,
) -> Result<u64>
```

Removes detached entries ("from a branch that left" and the like), of
one branch or of every one; answers how many. Only the library entries
go: files already delivered stay in this library's outputs.

#### Errors

Returns the write error.

### library_contents

```rust
pub async fn library_contents(store: &LibraryStore) -> Result<LibraryContents>
```

Counts the library data this install holds of its own.

#### Errors

Returns the read error.

### owner_nodes_visible_to

```rust
pub async fn owner_nodes_visible_to(
    store: &LibraryStore,
    actor: &EnrollmentActor,
) -> Result<Vec<NodeRecord>>
```

The owner nodes (`kind = owner`: this one and its clerks) `actor` may
see, by the rule [`remove_owner_node`](#remove_owner_node) removes them: the operator or an
active Owner sees every one; another active user only the ones they own;
an unknown or disabled user none.

#### Errors

Returns the read error.

### remove_owner_node

```rust
pub async fn remove_owner_node(
    store: &LibraryStore,
    actor: &EnrollmentActor,
    node_id: &str,
    this_node: &str,
) -> Result<()>
```

Removes owner node `node_id` from the library's node list, in one
transaction: its `nodes` row (and with it its labels and credentials),
status, event-subscriber registrations, secret handoffs, and plugin
deployments with their observations. Only a node that is not
running may go, and never `this_node`. The liveness check and the
removal are one step: the node's status row is deleted only if it still
holds the stale heartbeat that was checked, so a heartbeat that lands
in between refuses the removal. A node that was only cut off from the
database for longer than its liveness window can still be removed while
it runs; its next heartbeat writes a status row again, but not its node
row. Removing a node does not take back the
database credentials or DEK it held; rotate those when it is not
trusted any more.

The operator or an Owner may remove any owner node; another user only
one they own.

#### Errors

Returns [`LibraryError::NotFound`](index.md#libraryerror) for an unknown node or one the
caller may not see, [`LibraryError::Invalid`](index.md#libraryerror) for a host node (revoke
it instead) or this node, [`LibraryError::Conflict`](index.md#libraryerror) while it is live,
and the write error.

### decode_spki_pin

```rust
pub fn decode_spki_pin(pin: &str) -> Result<[u8; 32]>
```

Decodes an SPKI pin: base64url (no padding) of a SHA-256 digest.

#### Errors

Returns [`LibraryError::Invalid`](index.md#libraryerror) for anything else.

### join_cycle

```rust
pub async fn join_cycle(
    store: &LibraryStore,
    parent_ancestry: &[String],
) -> Result<Option<String>>
```

This library's own cluster id when `parent_ancestry` names it: joining
that library would make a cycle (A ← B ← A, A ← B ← C ← A). `None` when
it does not, or this install has no cluster identity yet.

#### Errors

Returns the read error.

### join_overflows_ancestry

```rust
pub async fn join_overflows_ancestry(
    store: &LibraryStore,
    parent_ancestry: &[String],
) -> Result<bool>
```

True when joining a library that reports `parent_ancestry` would give
this library more ancestors (itself and every library above it) than
[`bookclerk_node_proto::MAX_ANCESTRY`]: [`library_ancestry`](#library_ancestry) would then
leave the farthest out, and a library joining this one later could no
longer be checked for a loop through them. An ancestry cut at that
limit by the library itself always holds exactly the limit, so this
refuses it too. Such a join is refused rather than cut short.

#### Errors

Returns the read error.

### leave_cluster_link

```rust
pub async fn leave_cluster_link(
    store: &LibraryStore,
    owner_cluster_id: &str,
    expected_node_id: Option<&str>,
) -> Result<Option<ClusterLink>>
```

Leaves an owner cluster on this side: marks the link
[`CLUSTER_LINK_LEFT`](#cluster_link_left) and deletes its sealed node keys, so this install
can never sign as that node again. Answers the link as it now stands,
or `None` when there is no such link.

This only records the departure. Telling the owner (`node.leaving`,
which revokes the node there) is the caller's, before the keys go:
`bookclerk_node_client::announce_leaving` or the running client. A
running daemon stops the link's client when it next re-reads the link.
Joining again needs a new sign-in or invite.

Only one enrollment is left: `expected_node_id` when given (the node the
caller told the owner about), else the one read here. A link a join
enrolled as another node, before this call or while it runs, is refused,
so a new enrollment is never marked left with its keys still stored or
left without the owner having been told.

#### Arguments

* `store` - Library store holding the link.
* `owner_cluster_id` - Owner cluster the link joins.
* `expected_node_id` - The node the caller means to leave, when it read
  the link earlier (to send `node.leaving`); `None` leaves whatever node
  the link names now.

#### Errors

Returns [`LibraryError::Invalid`](index.md#libraryerror) when the link was re-enrolled as
another node, and the read or write error.

### library_ancestry

```rust
pub async fn library_ancestry(store: &LibraryStore) -> Result<Vec<String>>
```

This library's ancestry (what `GET /api/node/v1/info` reports): its own
cluster id, then for each active link the owner's cluster id and the
ancestry that owner reported, without repeats, at most
[`bookclerk_node_proto::MAX_ANCESTRY`](../bookclerk_node_proto/index.md#max_ancestry) ids. Empty when this install has
no cluster identity yet. A join that would take it past that limit is
refused ([`join_overflows_ancestry`](#join_overflows_ancestry)); should a library above grow its
ancestry later, the list is cut at exactly the limit, which in turn
refuses every join under it.

A clerk shares its library's database, so it reports the same ancestry:
it is the same node of the tree.

#### Errors

Returns the read error.

### list_cluster_links

```rust
pub async fn list_cluster_links(store: &LibraryStore) -> Result<Vec<ClusterLink>>
```

Every cluster link, oldest enrollment first.

#### Errors

Returns the read error.

### load_cluster_link

```rust
pub async fn load_cluster_link(
    store: &LibraryStore,
    owner_cluster_id: &str,
) -> Result<Option<ClusterLink>>
```

One cluster link.

#### Errors

Returns the read error.

### load_cluster_link_keys

```rust
pub async fn load_cluster_link_keys(
    store: &LibraryStore,
    link: &ClusterLink,
) -> Result<ClusterLinkKeys>
```

Unseals the node keys of an active `link`.

#### Errors

Returns [`LibraryError::Forbidden`](index.md#libraryerror) when the link is not active (a
revoked or departed node never signs again),
[`LibraryError::NotFound`](index.md#libraryerror) when the key row is missing, and
[`LibraryError::Invalid`](index.md#libraryerror) when it does not hold two 32-byte keys.

### mark_cluster_link_revoked

```rust
pub async fn mark_cluster_link_revoked(
    store: &LibraryStore,
    owner_cluster_id: &str,
    remote_node_id: &str,
) -> Result<bool>
```

Marks an **active** link still enrolled as `remote_node_id`
[`CLUSTER_LINK_REVOKED`](#cluster_link_revoked) (the owner closed with 4403 on that node's
connection). A link this install already left stays `left`, and one a
later join enrolled as another node keeps its new enrollment.

#### Arguments

* `store` - Library store holding the link.
* `owner_cluster_id` - Owner cluster the link joins.
* `remote_node_id` - The node the revoked connection ran as.

#### Returns

Whether a row changed: `false` when the link is gone, no longer active,
or enrolled as another node.

#### Errors

Returns the write error.

### record_cluster_link

```rust
pub async fn record_cluster_link(
    store: &LibraryStore,
    link: &NewClusterLink,
    keys: &ClusterLinkKeys,
) -> Result<ClusterLink>
```

Seals `keys` and records the link, replacing an earlier link to the same
cluster (a re-enrollment) as `active`.

The key row is written first: a link row never exists without keys, and
a key row left by a failed write is replaced by the next enrollment.
When the re-enrollment minted a different node id, the earlier node's
key row is deleted once the new link row is stored, so the sealed
private keys of a node the owner no longer knows are not kept.

#### Errors

Returns [`LibraryError::Invalid`](index.md#libraryerror) for a malformed id, origin, or pin, an
error when no master key is configured, and the write error.

### set_cluster_link_ancestry

```rust
pub async fn set_cluster_link_ancestry(
    store: &LibraryStore,
    owner_cluster_id: &str,
    ancestry: &[String],
) -> Result<bool>
```

Records the owner library's ancestry as it reported it (`info` at join,
`catalog.session` on every sync). Ids that are not plain cluster ids are
refused; at most [`bookclerk_node_proto::MAX_ANCESTRY`](../bookclerk_node_proto/index.md#max_ancestry) are kept.

#### Errors

Returns [`LibraryError::Invalid`](index.md#libraryerror) for a malformed ancestry, and the
write error.

### set_cluster_link_sharing

```rust
pub async fn set_cluster_link_sharing(
    store: &LibraryStore,
    owner_cluster_id: &str,
    share_books: bool,
    include_branches: bool,
) -> Result<bool>
```

Sets a link's two sharing settings: "Share this branch's books with
*library*" and "Include books from my own branches". Answers false when
no such link exists. The running catalog sync reports them to the owner
on its next pass.

#### Errors

Returns the write error.

### set_cluster_link_state

```rust
pub async fn set_cluster_link_state(
    store: &LibraryStore,
    owner_cluster_id: &str,
    state: &str,
    remote_node_id: Option<&str>,
) -> Result<bool>
```

Sets a link's state. Answers false when no such link exists.

#### Arguments

* `store` - Library store holding the link.
* `owner_cluster_id` - Owner cluster the link joins.
* `state` - One of [`CLUSTER_LINK_ACTIVE`](#cluster_link_active), [`CLUSTER_LINK_REVOKED`](#cluster_link_revoked) or
  [`CLUSTER_LINK_LEFT`](#cluster_link_left).
* `remote_node_id` - When set, only a link still enrolled as this node is
  changed, so a client running under an earlier enrollment cannot touch the
  link that replaced it. `None` changes the link whatever its node.

#### Errors

Returns [`LibraryError::Invalid`](index.md#libraryerror) for an unknown state, and the write
error.

### bounded_observation_detail

```rust
pub fn bounded_observation_detail(detail: &str) -> String
```

Truncates detail to 512 bytes and drops ASCII control characters.

### delete_plugin_deployment

```rust
pub async fn delete_plugin_deployment(
    store: &LibraryStore,
    actor: &ConfigActor,
    deployment_id: &str,
) -> Result<()>
```

Deletes an absent deployment and its observations.

#### Errors

Returns an error when the actor may not write, the row is still present,
or the delete fails.

### ensure_plugin_deployment

```rust
pub async fn ensure_plugin_deployment(
    store: &LibraryStore,
    actor: &ConfigActor,
    instance_id: &PluginInstanceId,
    node_id: &str,
    desired: DesiredState,
) -> Result<PluginDeployment>
```

Inserts `(instance, node, desired)` when that pair has no row.

A second call returns the original row and does not bump the revision.

#### Errors

Returns an error when the actor may not write `desired`, the instance is
missing, or the insert fails.

### instance_has_deployment

```rust
pub async fn instance_has_deployment(
    store: &LibraryStore,
    instance_id: &PluginInstanceId,
) -> Result<bool>
```

True when any deployment row names `instance_id`.

Key-based spawn must not pick this instance. The deployment reconciler
passes the instance id and the already resolved bindings.

#### Errors

Returns an error when the read fails.

### list_all_deployments

```rust
pub async fn list_all_deployments(store: &LibraryStore) -> Result<Vec<PluginDeployment>>
```

Every deployment row in the cluster.

#### Errors

Returns an error when the read fails.

### list_deployments_for_instance

```rust
pub async fn list_deployments_for_instance(
    store: &LibraryStore,
    instance_id: &PluginInstanceId,
) -> Result<Vec<PluginDeployment>>
```

Every deployment row for `instance_id`, on any node.

#### Errors

Returns an error when the read fails.

### list_deployments_for_node

```rust
pub async fn list_deployments_for_node(
    store: &LibraryStore,
    node_id: &str,
) -> Result<Vec<PluginDeployment>>
```

Every deployment row targeting `node_id`, present or absent.

#### Errors

Returns an error when the read fails.

### list_observations_for_deployment

```rust
pub async fn list_observations_for_deployment(
    store: &LibraryStore,
    deployment_id: &str,
) -> Result<Vec<DeploymentObservation>>
```

Every observation row for `deployment_id`, from any node.

#### Errors

Returns an error when the read fails.

### list_present_deployments_for_node

```rust
pub async fn list_present_deployments_for_node(
    store: &LibraryStore,
    node_id: &str,
) -> Result<Vec<PluginDeployment>>
```

Present deployments whose `node_id` is `node_id`.

Rows for any other node are not returned.

#### Errors

Returns an error when the read fails.

### load_deployment

```rust
pub async fn load_deployment(
    store: &LibraryStore,
    deployment_id: &str,
) -> Result<Option<PluginDeployment>>
```

Loads one deployment.

#### Errors

Returns an error when the read fails.

### load_deployment_for_node

```rust
pub async fn load_deployment_for_node(
    store: &LibraryStore,
    instance_id: &PluginInstanceId,
    node_id: &str,
) -> Result<Option<PluginDeployment>>
```

The deployment of `instance_id` on `node_id`, if any.

#### Errors

Returns an error when the read fails.

### load_observation

```rust
pub async fn load_observation(
    store: &LibraryStore,
    deployment_id: &str,
    node_id: &str,
) -> Result<Option<DeploymentObservation>>
```

Loads the observation for one deployment on one node.

#### Errors

Returns an error when the read fails.

### present_plugin_keys_for_node

```rust
pub async fn present_plugin_keys_for_node(
    store: &LibraryStore,
    node_id: &str,
) -> Result<Vec<String>>
```

Canonical plugin keys with a present deployment on `node_id`.

#### Errors

Returns an error when the read fails.

### replace_plugin_deployment

```rust
pub async fn replace_plugin_deployment(
    store: &LibraryStore,
    actor: &ConfigActor,
    deployment_id: &str,
    expected_revision: i64,
    desired: DesiredState,
    operation_id: &str,
) -> Result<DeploymentReplace>
```

Replaces desired state when `expected_revision` is current.

The batch carries a revision predicate and a `replacePluginDeployment`
receipt. It does not update observation rows.

The scheduler writes only its own states, with one exception: it may
hand a row back (`placed` to `present`, `unplaced` to `absent`) when the
row's placement is gone, finishing a deletion a racing pass outran.

#### Errors

Returns an error when the actor may not write `desired`, the deployment
is missing, or the operation id was already used for a different body.

### upsert_observation

```rust
pub async fn upsert_observation(
    store: &LibraryStore,
    observation: &DeploymentObservation,
) -> Result<()>
```

Upserts one local observation.

A failed upsert does not roll back an install. The caller retries on the next tick.

#### Errors

Returns an error when the write fails.

### audit_count

```rust
pub async fn audit_count(store: &LibraryStore, namespace: &str) -> Result<u64>
```

Counts audit rows for one namespace.

#### Errors

Returns an error when the read fails.

### change_count

```rust
pub async fn change_count(store: &LibraryStore, namespace: &str) -> Result<u64>
```

Counts committed change notices for one namespace.

#### Errors

Returns an error when the read fails.

### authorized_enrollment_link

```rust
pub fn authorized_enrollment_link(
    code: &str,
    verifier: &str,
    kind: NodeKind,
) -> (String, EnrollmentSecret)
```

The link a branch or clerk sign-in redeems through: its id and secret are
derived from the authorization code and the PKCE verifier, so a retry of
the same exchange (same code, verifier and keys) finds the same link and
answers from its receipt, and a wrong verifier can never redeem it.

The secret also binds the node kind the code was approved for, so a link
recorded for one kind never redeems as the other, even when the first
redeem failed and left the link unredeemed.

The id is `hex(SHA-256("bookclerk-branch-link-v1\0" || code))[..32]`; the
secret is base64url `SHA-256("bookclerk-branch-secret-v1\0" || code ||
"\0" || verifier || "\0" || kind)`.

### check_redeem_request

```rust
pub fn check_redeem_request(req: &RedeemRequest) -> Result<()>
```

Checks what a redeem would refuse as malformed before anything is
written: its keys (canonical Curve25519 points) and `software_version`. A
sign-in's redeem runs this before it spends its code, so a malformed
request leaves the code for a corrected retry.

#### Errors

Returns [`LibraryError::Invalid`](index.md#libraryerror) for malformed keys or a version that is
too long or holds control characters.

### format_enrollment_token

```rust
pub fn format_enrollment_token(
    link_id: &str,
    secret: &EnrollmentSecret,
    spki_pin: Option<&str>,
) -> Zeroizing<String>
```

Formats `bcn1.<link_id>.<secret>[.<spki_pin>]`.

### list_enrollment_links

```rust
pub async fn list_enrollment_links(
    store: &LibraryStore,
    actor: &EnrollmentActor,
) -> Result<Vec<EnrollmentLink>>
```

Links `actor` may see, newest first: every link for the operator or an
Owner, the caller's own links for anyone else.

#### Errors

Returns an error when the actor is not an active user or the read fails.

### load_host_enrollment

```rust
pub async fn load_host_enrollment(store: &LibraryStore) -> Result<HostEnrollmentSettings>
```

Reads the cluster's host-enrollment switch. A missing document means the
default: enabled.

#### Errors

Returns an error when the read fails or the stored document has an
unsupported schema or an invalid body.

### load_node_credential

```rust
pub async fn load_node_credential(
    store: &LibraryStore,
    node_id: &str,
) -> Result<Option<NodeCredential>>
```

Reads a host node's credential.

#### Errors

Returns an error when the read fails.

### mint_enrollment_link

```rust
pub async fn mint_enrollment_link(
    store: &LibraryStore,
    actor: &EnrollmentActor,
    for_user_id: i64,
    label: &str,
    ttl: Option<Duration>,
) -> Result<MintedLink>
```

Mints a link that enrolls one host node for `for_user_id`.

`ttl` defaults to 24 hours and must be 5 minutes to 7 days. The insert
is predicated on the enrollment switch revision read as enabled, so a
disable that commits while the link is minted wins.

#### Errors

Returns [`LibraryError::Forbidden`](index.md#libraryerror) when enrollment is disabled, the
actor is disabled, or a non-Owner mints for someone else;
[`LibraryError::NotFound`](index.md#libraryerror) / [`LibraryError::Invalid`](index.md#libraryerror) when the target
user is unknown or disabled or the ttl or label is out of range; and the
write error when the insert fails.

### parse_enrollment_token

```rust
pub fn parse_enrollment_token(raw: &str) -> Result<EnrollmentToken>
```

Parses a token, or a whole link whose fragment is the token.

The secret is registered for redaction, so logs and diagnostics scrub
it in the process that parsed it.

#### Errors

Returns [`LibraryError::Invalid`](index.md#libraryerror) when the text is not a `bcn1` token.

### record_authorized_enrollment_link

```rust
pub async fn record_authorized_enrollment_link(
    store: &LibraryStore,
    link_id: &str,
    secret: &EnrollmentSecret,
    for_user_id: i64,
    label: &str,
) -> Result<()>
```

Records the link an approved branch sign-in redeems through
([`authorized_enrollment_link`](#authorized_enrollment_link)), for `for_user_id`, the user who signed
in and consented. Called only after the authorization code's client,
redirect and PKCE verifier were checked, before the code is consumed, so
a failed write leaves the code usable; the redeem itself is
[`redeem_enrollment_link`](#redeem_enrollment_link), the same canonical batch a minted link takes.

A second call for the same link id leaves the first row as it is.

#### Errors

Returns [`LibraryError::Forbidden`](index.md#libraryerror) while host enrollment is off or the
user is not active, [`LibraryError::Invalid`](index.md#libraryerror) for a malformed label, and
the write error.

### record_node_connected

```rust
pub async fn record_node_connected(store: &LibraryStore, node_id: &str) -> Result<bool>
```

Stamps `last_connected_at` after a host node authenticates. Returns
false (and stamps nothing) when the credential is missing or revoked,
so a revoke that lands mid-handshake still refuses the connection.

#### Errors

Returns an error when the update fails.

### record_node_left

```rust
pub async fn record_node_left(store: &LibraryStore, node_id: &str) -> Result<bool>
```

A host announced it is leaving (`node.leaving`): revokes its credential
at once and drops its live status, so it stops counting as a node
without waiting for missed heartbeats, and tells every node so the
connection is closed wherever it is held. Answers false when the node
has no unrevoked credential; for one already revoked, the cleanup is
still finished, so a retried leave completes one that failed part way.

#### Errors

Returns the write error.

### redeem_enrollment_link

```rust
pub async fn redeem_enrollment_link(
    store: &LibraryStore,
    req: &RedeemRequest,
) -> Result<EnrolledNode>
```

Redeems a link: one canonical batch marks it redeemed, inserts a host
`nodes` row owned by the link's user, inserts its `node_credentials`,
and records a receipt.

The batch's predicates re-check the link hash, expiry, single use,
revocation, the user's status, and that the enrollment switch is still
the revision read as enabled, so a concurrent disable or second redeem
fails closed. A retry with the same link, secret, and keys after a lost
reply returns the same node (`replayed`).

#### Errors

Returns [`LibraryError::Invalid`](index.md#libraryerror) for malformed keys,
[`LibraryError::Forbidden`](index.md#libraryerror) when enrollment is disabled or the link does
not redeem (one message for every reason), and the write error when the
batch fails.

### redeem_enrollment_link_as

```rust
pub async fn redeem_enrollment_link_as(
    store: &LibraryStore,
    req: &RedeemRequest,
    kind: NodeKind,
) -> Result<EnrolledNode>
```

[`redeem_enrollment_link`](#redeem_enrollment_link) for a node of `kind`: the same canonical
batch, recording `kind` on the `nodes` row.

A [`super::NodeKind::Owner`](#nodekind) node (a clerk, which joined with a sign-in
an Owner approved as such) shares the library's database, so it gets no
`node_credentials` row: it never dials the node socket. The kind is part
of the request hash, so a retry of a branch redeem as a clerk (or the
other way round) is refused as a spent link.

#### Errors

As [`redeem_enrollment_link`](#redeem_enrollment_link).

### revoke_enrollment_link

```rust
pub async fn revoke_enrollment_link(
    store: &LibraryStore,
    actor: &EnrollmentActor,
    link_id: &str,
) -> Result<EnrollmentLink>
```

Revokes an unredeemed link. Returns the link as stored afterwards.

#### Errors

Returns [`LibraryError::NotFound`](index.md#libraryerror) when the link is unknown or not the
caller's (the two read the same), [`LibraryError::Conflict`](index.md#libraryerror) when it was
already redeemed, and the write error when the update fails.

### revoke_node_credential

```rust
pub async fn revoke_node_credential(
    store: &LibraryStore,
    actor: &EnrollmentActor,
    node_id: &str,
) -> Result<NodeCredential>
```

Revokes a host node's credential and tells every node, so the node
holding that host's connection can close it.

The operator and an Owner may revoke any host node; any other active
user may revoke the nodes they own. Revoking twice keeps the first time.

#### Errors

Returns [`LibraryError::NotFound`](index.md#libraryerror) when the node is unknown or not the
caller's (the two read the same), [`LibraryError::Invalid`](index.md#libraryerror) when the
node has no host credential (an owner node), and the write error when
the update fails.

### set_host_enrollment_enabled

```rust
pub async fn set_host_enrollment_enabled(
    store: &LibraryStore,
    actor: &ConfigActor,
    enabled: bool,
) -> Result<ConfigurationDocument<HostEnrollmentSettingsV1>>
```

Turns host enrollment on or off for the cluster.

Only the operator or a verified Owner ([`verify_cluster_owner`](#verify_cluster_owner)) may write
it. Retries a few times on a concurrent edit.

#### Errors

Returns [`LibraryError::Forbidden`](index.md#libraryerror) for any other actor, and the write
error when the document cannot be stored.

### verify_cluster_owner

```rust
pub async fn verify_cluster_owner(
    store: &LibraryStore,
    user_id: i64,
) -> Result<ConfigActor>
```

Returns a [`ConfigActor::ClusterOwner`](#configactor) when `user_id` is an active Owner.

#### Errors

Returns [`LibraryError::Forbidden`](index.md#libraryerror) for anyone else (including an unknown
user), and the read error when the lookup fails.

### current_process_incarnation

```rust
pub fn current_process_incarnation() -> &'static str
```

This process's incarnation. Stable until the process exits.

### encode_node_identity_file

```rust
pub fn encode_node_identity_file(node_id: &str, cluster_id: &str) -> Result<Vec<u8>>
```

The [`NODE_IDENTITY_FILE`](#node_identity_file) contents [`install_node_identity`](#install_node_identity) writes, for
a caller that stages the file itself (a clerk applying its bootstrap as
one recoverable step) with mode `0600`.

#### Errors

Returns [`LibraryError::Invalid`](index.md#libraryerror) for a node id that is not a UUID or an
empty cluster id, and the encode error.

### install_node_identity

```rust
pub fn install_node_identity(
    files_dir: &Path,
    node_id: &str,
    cluster_id: &str,
) -> Result<NodeIdentity>
```

Writes the identity file for a node id the cluster already recorded,
bound to `cluster_id`, replacing any file there atomically. A clerk
joining a library takes the node id the library minted for it at
redeem, so its first heartbeat lands on that `nodes` row.

#### Errors

Returns [`LibraryError::Invalid`](index.md#libraryerror) for a node id that is not a UUID or an
empty cluster id, and the write error.

### list_node_ids

```rust
pub async fn list_node_ids(store: &LibraryStore) -> Result<Vec<String>>
```

Lists node ids, sorted.

#### Errors

Returns an error when the read fails.

### list_nodes

```rust
pub async fn list_nodes(store: &LibraryStore) -> Result<Vec<NodeRecord>>
```

Every node row, sorted by id.

#### Errors

Returns an error when the read fails.

### load_node

```rust
pub async fn load_node(store: &LibraryStore, node_id: &str) -> Result<NodeRecord>
```

Loads one node row.

#### Errors

Returns an error when the row is missing or the read fails.

### load_or_create_node_identity

```rust
pub fn load_or_create_node_identity(files_dir: &Path) -> Result<NodeIdentity>
```

Loads the identity file or creates a new node id.

Concurrent creators of the same path share the winner's node id.
Copying the file copies the node id. A new file is a distinct node.

#### Errors

Returns an error when the file is unreadable, has an unknown version, or
cannot be created.

### node_identity_path

```rust
pub fn node_identity_path(files_dir: &Path) -> PathBuf
```

Path of the node identity file.

### node_target

```rust
pub fn node_target() -> &'static str
```

Bookclerk artifact target this binary runs as (`linux-x64-gnu`,
`macos-arm64`, …). Matches `bookclerk_plugin_catalog::host_bookclerk_target`
(a plugin-host test pins the two together).

### import_instance_config_if_absent

```rust
pub async fn import_instance_config_if_absent(
    store: &LibraryStore,
    actor: &ConfigActor,
    instance_id: &PluginInstanceId,
    schema: &SettingsSchema,
    body: &PluginInstanceConfigV1,
    operation_id: &str,
) -> Result<InstanceConfigDocument>
```

Inserts the document when it is absent.

An existing row is returned and the seed is not validated again.

#### Errors

Returns an error when the actor may not import, the body is invalid, or the write fails.

### load_instance_config

```rust
pub async fn load_instance_config(
    store: &LibraryStore,
    instance_id: &PluginInstanceId,
) -> Result<InstanceConfigDocument>
```

Loads and parses the instance config document.

An unknown `schema_version` fails the read. The stored JSON is not rewritten.

#### Errors

Returns an error when the row is missing, the schema is unknown, or the body is invalid.

### parse_setting_text

```rust
pub fn parse_setting_text(
    schema: &SettingsSchema,
    key: &str,
    text: &str,
) -> Result<SettingValue>
```

Parses an operator-typed value for `key` according to the schema.

Enum and string settings take the text as given; `bool` accepts
`true`/`false`; `integer` a signed decimal. Secrets are rejected here:
seal them with [`seal_instance_secret`](#seal_instance_secret) and add a ref.

#### Errors

Returns an error when `key` is unknown or `text` does not fit its kind.

### reconcile_instance_configs_to_schema

```rust
pub async fn reconcile_instance_configs_to_schema(
    store: &LibraryStore,
    plugin_key: &str,
    schema: &SettingsSchema,
) -> Result<Vec<PrunedInstanceConfig>>
```

Rewrites every instance document of `plugin_key` to what `schema` still
accepts: settings whose key is gone, became a secret, or whose value no
longer fits (a type or enum change), and secret refs whose key is gone or
became plain, are dropped.

Enrollment runs this on every pass, so a document that lost a race or
was written under an earlier schema is repaired on the next restart
rather than staying invalid. A document that already fits is not
rewritten; defaults for new keys are not written (`CONFIG` resolution
applies them at spawn). One instance's failure (a compare-and-swap
conflict because another node wrote the document meanwhile, or an
unreadable row) is logged and skipped so the others are still handled.

#### Errors

Returns an error when the instance list cannot be read.

### replace_instance_config

```rust
pub async fn replace_instance_config(
    store: &LibraryStore,
    actor: &ConfigActor,
    instance_id: &PluginInstanceId,
    schema: &SettingsSchema,
    expected_revision: i64,
    body: &PluginInstanceConfigV1,
    operation_id: &str,
) -> Result<InstanceConfigReplace>
```

Replaces the instance document when `expected_revision` is current.

#### Errors

Returns an error when validation, authorization, or the compare-and-swap fails.

### resolve_instance_bindings

```rust
pub async fn resolve_instance_bindings(
    store: &LibraryStore,
    instance_id: &PluginInstanceId,
    grant: &InstanceBindingGrant,
    schema: &SettingsSchema,
) -> Result<ResolvedInstanceBindings>
```

Resolves `CONFIG` and `SECRETS` for `instance_id` against `schema`.

`CONFIG` is the schema's defaults overlaid with the document's settings.
Keys the schema does not declare, and values that no longer fit their
entry, are dropped and reported; a required key with neither a usable
value nor a default fails, as does a required secret with no ref. A
missing document, a dangling secret ref, a secret that fails to unseal, a
missing `config` grant for a plugin that declares settings, or secret refs
without a `secrets` grant fails resolution. An empty ref list with no
`secrets` grant yields the empty secrets payload.

#### Errors

Returns an error when the document or a secret ref cannot be resolved.

### seal_instance_secret

```rust
pub async fn seal_instance_secret(
    store: &LibraryStore,
    instance_id: &PluginInstanceId,
    plugin_key: &str,
    name: &str,
    plaintext: &str,
) -> Result<()>
```

Seals one instance secret with the cluster DEK.

The plaintext is not written to `configuration_documents` or `configuration_audit`.

#### Errors

Returns an error when the value is empty or the seal fails.

### set_secret_ref

```rust
pub fn set_secret_ref(body: &mut PluginInstanceConfigV1, key: &str)
```

Adds a secret ref for `key` whose sealed row is named `key`, replacing an
existing ref for the same key.

### set_setting

```rust
pub fn set_setting(
    body: &mut PluginInstanceConfigV1,
    key: &str,
    value: Option<SettingValue>,
)
```

Sets `key` to `value` in `settings`, or removes it when `value` is `None`.

The CLI and API use this for one-key edits; the full document still goes
through [`replace_instance_config`](#replace_instance_config) with the pinned schema.

### create_node_local_instance

```rust
pub async fn create_node_local_instance(
    store: &LibraryStore,
    actor: &ConfigActor,
    plugin_key: &str,
    node_id: &str,
    display_name: &str,
) -> Result<PluginInstance>
```

Inserts a node-local instance of `plugin_key` owned by `node_id` (H-2):
it adds to the cluster's outputs on that node only, and a deployment of
it to any other node, or a placement, is refused.

#### Errors

Returns an error when the actor may not create an instance, the node id
is malformed or has no `nodes` row, or the insert fails.

### create_plugin_instance

```rust
pub async fn create_plugin_instance(
    store: &LibraryStore,
    actor: &ConfigActor,
    plugin_key: &str,
    display_name: &str,
) -> Result<PluginInstance>
```

Inserts a new instance for `plugin_key`.

A second call mints a different id even when `plugin_key` matches.

#### Errors

Returns an error when the actor may not create an instance or the insert fails.

### default_instance_for_key

```rust
pub async fn default_instance_for_key(
    store: &LibraryStore,
    plugin_key: &str,
) -> Result<Option<PluginInstance>>
```

The instance a host routes to when a job does not pin one: the
`plugin_instance_defaults` claim for `plugin_key`, else the oldest row.

#### Errors

Returns an error when the read fails.

### delete_plugin_instance

```rust
pub async fn delete_plugin_instance(
    store: &LibraryStore,
    actor: &ConfigActor,
    id: &PluginInstanceId,
) -> Result<()>
```

Deletes an instance that has no deployments. Its configuration document,
audit rows, and sealed secrets are left for retention cleanup.

#### Errors

Returns an error when the actor may not write, a deployment still
references the instance, or the delete fails.

### ensure_plugin_instance

```rust
pub async fn ensure_plugin_instance(
    store: &LibraryStore,
    actor: &ConfigActor,
    plugin_key: &str,
) -> Result<PluginInstance>
```

Returns the existing instance for `plugin_key`, or inserts one.

GraphicAudio startup uses this so a second process start keeps the original id.
Concurrent calls, including two connections to the same file, share one
inserted row. The claim is one canonical host batch (no interactive
`BEGIN`), so D1 and other non-interactive adapters can run it. A lock or
uniqueness failure retries that batch. Explicit [`create_plugin_instance`](#create_plugin_instance)
can still mint another row for the same key; that path does not claim the
default enrollment.

#### Errors

Returns an error when the actor may not create an instance or the write fails.

### find_plugin_instance_by_key

```rust
pub async fn find_plugin_instance_by_key(
    store: &LibraryStore,
    plugin_key: &str,
) -> Result<Option<PluginInstance>>
```

Earliest instance row for `plugin_key`, if any.

Oldest-wins. Prefer [`default_instance_for_key`](#default_instance_for_key), which honours the
`plugin_instance_defaults` claim and falls back to this order.

#### Errors

Returns an error when the read fails.

### list_all_plugin_instances

```rust
pub async fn list_all_plugin_instances(
    store: &LibraryStore,
) -> Result<Vec<PluginInstance>>
```

Every instance row, ordered by plugin key then creation time.

#### Errors

Returns an error when the read fails.

### list_plugin_instances_for_key

```rust
pub async fn list_plugin_instances_for_key(
    store: &LibraryStore,
    plugin_key: &str,
) -> Result<Vec<PluginInstance>>
```

Every instance row for `plugin_key`, oldest first.

#### Errors

Returns an error when the read fails.

### load_plugin_instance

```rust
pub async fn load_plugin_instance(
    store: &LibraryStore,
    id: &PluginInstanceId,
) -> Result<Option<PluginInstance>>
```

Loads one instance by id.

#### Errors

Returns an error when the read fails or `id` is not a UUID.

### account_semaphore

```rust
pub fn account_semaphore(plugin_key: &str, account_id: &str) -> String
```

Name of the cluster semaphore for one store account.

### claim_routed_job

```rust
pub async fn claim_routed_job(
    store: &LibraryStore,
    request: &JobClaimRequest<'_>,
) -> Result<Option<RoutedJobClaim>>
```

Claims the best ready job this node can run, under the cluster semaphores.

Pages through the ready pending jobs of the class, best first
([`ROUTED_CLAIM_CANDIDATES`](#routed_claim_candidates) per page); skips those the node is not
trusted with ([`super::node_trust`](#node_trust)), those whose requirements this node
does not meet, and those whose account or instance semaphore is full;
takes the permits of the first one left and claims it with
[`LibraryStore::claim_job_by_id`](index.md#librarystoreclaim_job_by_id) at the lease generation of the record
it evaluated, so a job whose requirements were rewritten after this node
checked them is not claimed on the stale check. The permit holder is
[`job_permit_holder_prefix`](#job_permit_holder_prefix) for the generation the claim runs at plus a
random suffix, so permits left over from an earlier claim of the same
job by the same owner never pass for this claim's. A lost claim gives
the permits back and moves on. It stops at the first claim, when the
rows run out, or when the class semaphore itself is full (no further
candidate could run).

#### Returns

The claimed job and its permits, or `None` when nothing could be claimed.

#### Errors

Returns an error when a read or write fails; permits taken for the
failing attempt are released on a best-effort basis.

### class_semaphore

```rust
pub fn class_semaphore(class: JobResourceClass) -> String
```

Name of the cluster semaphore for one resource class.

### common_deployment_nodes

```rust
pub async fn common_deployment_nodes(
    store: &LibraryStore,
    requires: &[String],
) -> Result<BTreeSet<String>>
```

Nodes where every instance in `requires` has a deployment that should run.

#### Errors

Returns an error when a read fails.

### instance_semaphore

```rust
pub fn instance_semaphore(instance_id: &str) -> String
```

Name of the cluster semaphore for one plugin instance.

### job_permit_holder_prefix

```rust
pub fn job_permit_holder_prefix(owner: &str, job_id: &str, generation: i64) -> String
```

The holder prefix of the semaphore permits a routed claim of `job_id`
by `owner` takes when the claim runs at lease `generation`:
`<owner>/<job_id>/g<generation>/`.

Every permit of that claim's holder starts with it (a random suffix
follows), and the trailing `/` keeps one generation's prefix from
matching another's (`g1/` is not a prefix of `g10/`). A worker that has
to find the claim's permits again by name, such as a Host renewing
through another owner node, matches on this prefix, so a still-live
permit left over from an earlier claim of the same job is never taken
for the current one.

### job_plugin_keys

```rust
pub async fn job_plugin_keys(
    store: &LibraryStore,
    job: &JobRecord,
) -> Result<Option<BTreeSet<String>>>
```

The plugin keys `job` runs, or `None` when the job alone does not say.

Only a scan or acquire says: the plugin of the instance it is pinned to
and of each instance it requires, plus every plugin holding an
`accounts` row with the account it names (within the pinned plugin when
pinned). An unpinned acquire that names no account runs the plugins of
the library rows it matches. An unpinned scan of every account, an
acquire matching no row, any other kind, and a job naming an unknown
instance or account answer `None`.

#### Errors

Returns an error when a read fails.

### job_semaphores

```rust
pub async fn job_semaphores(
    store: &LibraryStore,
    job: &JobRecord,
    limits: JobClaimLimits,
) -> Result<Vec<SemaphoreRequest>>
```

The cluster semaphores `job` counts against under `limits`.

Every job takes its class semaphore and one per required instance. Scan
and acquire also take one per store account they touch: the payload's
account when it names one (limited to the pinned instance's plugin when
pinned), otherwise every account of the pinned plugin, or every account.

#### Errors

Returns an error when a read fails.

### job_source_plugin_key

```rust
pub async fn job_source_plugin_key(
    store: &LibraryStore,
    job: &JobRecord,
) -> Result<Option<String>>
```

The one plugin whose source guest runs `job`, or `None` when the job
alone does not name exactly one.

Only a scan or acquire has one: the plugin of the instance it is pinned
to, or else the one plugin holding an `accounts` row the account it
names resolves to (by id or label, ignoring ASCII case). Required
instances ([`crate::JobPayload::requires`](index.md#jobpayload), the destinations an acquire
writes) never count. An unpinned job of every account, or one whose
account name matches rows under several plugins, answers
`None`, as do other kinds and unknown instances or accounts.

#### Errors

Returns an error when a read fails.

### node_can_run_job

```rust
pub async fn node_can_run_job(
    store: &LibraryStore,
    node_id: &str,
    requires: &[String],
) -> Result<bool>
```

True when `node_id` can run a job that requires `requires`.

Each instance needs a deployment on the node that should run and an
observation from the node with the guest up. An empty list is always
satisfied; an unknown node or an unparsable id never is.

#### Errors

Returns an error when a read fails.

### resolve_job_requirements

```rust
pub async fn resolve_job_requirements(
    store: &LibraryStore,
    kind: JobKind,
    payload: &JobPayload,
    node_bound_plugin_keys: &BTreeSet<String>,
) -> Result<Vec<String>>
```

The instances a new job must find on its claiming node.

`pinned` is the instance the job was pinned to; it is kept only when it
is a valid instance id with at least one deployment that should run, so a
job pinned to an undeployed instance is claimed anywhere and fails there
as before rather than waiting forever. For [`JobKind::Acquire`](index.md#jobkind), every
cluster instance of a plugin key in `node_bound_plugin_keys` that is
deployed to run somewhere is added: the acquire writes to that output, so
it must run where the output's disk is. A node-local instance (H-2,
`owner_node_id` set) is not: it adds to the outputs of its own node only,
which copies what it acquires to it, and requiring it would leave every
acquire claimable on that one node alone, or nowhere.

The required instances must share at least one node where all of them
are deployed to run; otherwise no node could ever claim the job, so it
is refused here instead of being left pending. Required instances are
never dropped to make a node fit: an acquire must write to every
deployed node-bound output.

#### Errors

Returns [`LibraryError::Conflict`](index.md#libraryerror) when the required instances have no
common deployment node, or an error when a read fails.

### acquire_lease

```rust
pub async fn acquire_lease(
    store: &LibraryStore,
    name: &str,
    holder: &str,
    ttl: Duration,
) -> Result<Option<ClusterLease>>
```

Acquires or renews `name` for `holder` for `ttl`.

Returns `None` when another holder's lease has not expired, or when a
concurrent acquire won. Renewing an unexpired lease keeps the token;
taking an expired (or released) lease bumps it, even when `holder` is
the one that held it.

#### Errors

Returns an error when the read or write fails.

### acquire_lease_on

```rust
pub async fn acquire_lease_on<C: ConnectionTrait>(
    db: &C,
    name: &str,
    holder: &str,
    ttl: Duration,
) -> Result<Option<ClusterLease>>
```

[`acquire_lease`](#acquire_lease) on a bare connection (schema migration runs before a
[`LibraryStore`](index.md#librarystore) exists).

#### Errors

Returns an error when the read or write fails.

### list_leases

```rust
pub async fn list_leases(store: &LibraryStore) -> Result<Vec<ClusterLease>>
```

Every lease row, held or expired, by name.

#### Errors

Returns an error when the read fails.

### load_lease

```rust
pub async fn load_lease(store: &LibraryStore, name: &str) -> Result<Option<ClusterLease>>
```

The current row of lease `name`, held or expired.

#### Errors

Returns an error when the read fails.

### load_lease_on

```rust
pub async fn load_lease_on<C: ConnectionTrait>(
    db: &C,
    name: &str,
) -> Result<Option<ClusterLease>>
```

[`load_lease`](#load_lease) on a bare connection.

#### Errors

Returns an error when the read fails.

### release_lease

```rust
pub async fn release_lease(store: &LibraryStore, lease: &ClusterLease) -> Result<()>
```

Gives up a held lease so another holder may take it at once.

The row stays (holder and token remain visible to status reads) with an
expiry of zero.

#### Errors

Returns an error when the write fails.

### release_lease_on

```rust
pub async fn release_lease_on<C: ConnectionTrait>(
    db: &C,
    lease: &ClusterLease,
) -> Result<()>
```

[`release_lease`](#release_lease) on a bare connection.

#### Errors

Returns an error when the write fails.

### renew_lease

```rust
pub async fn renew_lease(
    store: &LibraryStore,
    lease: &ClusterLease,
    ttl: Duration,
) -> Result<Option<ClusterLease>>
```

Fencing check: extends a lease this holder still holds.

Succeeds only while the row names `lease.holder` and `lease.token` and
has not expired. Returns the extended lease, or `None` when the lease
expired or another holder took it; the caller must then stop before any
further destructive work.

#### Errors

Returns an error when the write fails.

### renew_lease_on

```rust
pub async fn renew_lease_on<C: ConnectionTrait>(
    db: &C,
    lease: &ClusterLease,
    ttl: Duration,
) -> Result<Option<ClusterLease>>
```

[`renew_lease`](#renew_lease) on a bare connection.

#### Errors

Returns an error when the write fails.

### changed_topics

```rust
pub fn changed_topics(before: &LiveRevisions, after: &LiveRevisions) -> Vec<LiveTopic>
```

The topics whose revision differs between `before` and `after`.

### live_revisions

```rust
pub async fn live_revisions(store: &LibraryStore) -> Result<LiveRevisions>
```

Reads every topic's revision.

#### Errors

Returns the read error.

### is_node_consumer_id

```rust
pub fn is_node_consumer_id(plugin_id: &str) -> bool
```

True when a catalog / delivery `plugin_id` is a node consumer rather than
a plugin.

### is_valid_node_id

```rust
pub fn is_valid_node_id(node_id: &str) -> bool
```

True when `node_id` can name a node: `node.<node_id>` is a valid queue
name. Minted ids are lowercase UUIDs, which always qualify.

### node_consumer_id

```rust
pub fn node_consumer_id(node_id: &str) -> String
```

Catalog id of a node's own consumer (`node:<node_id>`).

### node_queue_name

```rust
pub fn node_queue_name(node_id: &str) -> String
```

Queue a node consumes: `node.<node_id>`. Registration accepts only node
ids for which this is a valid queue name ([`is_valid_node_id`](#is_valid_node_id)).

### publish_node_message

```rust
pub async fn publish_node_message(
    store: &LibraryStore,
    node_id: &str,
    message: &NodeMessage,
) -> Result<()>
```

Publishes `message` onto `node.<node_id>`.

#### Errors

Returns an error when the outbox insert fails.

### publish_to_all_nodes

```rust
pub async fn publish_to_all_nodes(
    store: &LibraryStore,
    message: &NodeMessage,
) -> Result<()>
```

Publishes `message` onto every registered node's queue.

#### Errors

Returns an error when the node list or an outbox insert fails.

### instance_demand

```rust
pub async fn instance_demand(
    store: &LibraryStore,
    instance_id: &PluginInstanceId,
) -> Result<Option<GuestReservation>>
```

What one more guest of `instance_id` is expected to need: the largest
ceiling any node recorded for it, or `None` when no node ran it yet.

#### Errors

Returns an error when a read fails.

### list_node_statuses

```rust
pub async fn list_node_statuses(store: &LibraryStore) -> Result<Vec<NodeStatus>>
```

Every node's latest heartbeat, by node id.

#### Errors

Returns an error when the read fails.

### load_node_status

```rust
pub async fn load_node_status(
    store: &LibraryStore,
    node_id: &str,
) -> Result<Option<NodeStatus>>
```

One node's latest heartbeat, if its daemon ever ran.

#### Errors

Returns an error when the read fails.

### next_availability

```rust
pub fn next_availability(previous_ppm: i64, gap_ms: i64, interval_ms: i64) -> i64
```

Next availability average (parts per million) after `gap_ms` since the
previous heartbeat.

The node counts as online for the whole gap when it is within the live
window, else for one interval (it stopped right after the last
heartbeat). The average decays with [`AVAILABILITY_TAU`](#availability_tau).

### node_reservations

```rust
pub async fn node_reservations(
    store: &LibraryStore,
    node_id: &str,
) -> Result<(GuestReservation, i64)>
```

Sum of the ceilings of the guests running on `node_id`, and how many run.
A running guest with no recorded ceiling counts as the grant defaults.

#### Errors

Returns an error when a read fails.

### record_deployment_reservation

```rust
pub async fn record_deployment_reservation(
    store: &LibraryStore,
    deployment_id: &str,
    node_id: &str,
    reservation: GuestReservation,
) -> Result<()>
```

Records the ceilings of the guest a node just started for a deployment.

#### Errors

Returns an error when the update fails.

### record_node_heartbeat

```rust
pub async fn record_node_heartbeat(
    store: &LibraryStore,
    node_id: &str,
    heartbeat: &NodeHeartbeat,
) -> Result<NodeStatus>
```

Records this node's heartbeat: its measurements, its budget from
`node.runtime`, the reservations of its running guests, and the updated
availability.

#### Errors

Returns an error when a read or the upsert fails.

### import_output_if_absent

```rust
pub async fn import_output_if_absent(
    store: &LibraryStore,
    actor: &ConfigActor,
    seed: &OutputConfig,
    operation_id: &str,
) -> Result<ConfigurationDocument<OutputSettingsV1>>
```

Inserts `[output]` when the cluster document is absent.

An existing document is parsed and returned without reading the seed, so
an obsolete or out-of-range TOML/environment value cannot fail startup or
replace the stored document.

#### Errors

Returns an error when the stored document is invalid, or when the document
is absent and the actor may not import. An absent document is seeded with
[`OutputSettingsV1::from_seed_lenient`]: an out-of-range seed field is
logged and replaced by its default rather than failing startup.

### is_output_setting_key

```rust
pub fn is_output_setting_key(key: &str) -> bool
```

True when `key` is a `core.output` setting or alias.

### load_output

```rust
pub async fn load_output(
    store: &LibraryStore,
) -> Result<ConfigurationDocument<OutputSettingsV1>>
```

Loads the cluster output document.

#### Errors

Returns an error when the document is missing, unsupported, or invalid.

### overlay_output

```rust
pub fn overlay_output(
    config: &mut Config,
    output: &ConfigurationDocument<OutputSettingsV1>,
    cluster_id: &str,
)
```

Copies the authoritative output document onto `config` for `cluster_id`.

Revisions are only comparable within one cluster. Callers that switch
databases pass the new cluster id explicitly.

### portable_output_seed

```rust
pub fn portable_output_seed(seed: &OutputConfig, files_dir: &Path) -> OutputConfig
```

Copies `[output]` with a `widevine_cdm` under `files_dir` made relative again.

`Config::load` resolves a relative CDM path under the loading host's files
directory. The shared document keeps it relative, so every host rebases it
under its own `BOOKCLERK_FILES_DIR` in [`OutputSettingsV1::apply_to`](#outputsettingsv1apply_to). A
path outside `files_dir` (or equal to it) is kept as written.

#### Arguments

* `seed` - The loaded `[output]` table.
* `files_dir` - The loading host's files directory.

### replace_output

```rust
pub async fn replace_output(
    store: &LibraryStore,
    actor: &ConfigActor,
    expected_revision: i64,
    body: &OutputSettingsV1,
    operation_id: &str,
) -> Result<OutputReplace>
```

Replaces the cluster output document.

#### Errors

Returns an error when validation, authorization, or the compare-and-swap fails.

### nodes_owned_by

```rust
pub async fn nodes_owned_by(store: &LibraryStore, user_id: i64) -> Result<Vec<NodeRecord>>
```

Nodes `user_id` owns, sorted by id.

#### Errors

Returns an error when the read fails.

### set_node_owner

```rust
pub async fn set_node_owner(
    store: &LibraryStore,
    actor: &ConfigActor,
    node_id: &str,
    owner: Option<i64>,
) -> Result<NodeRecord>
```

Assigns `node_id` to `owner` (a user id), or clears its owner.

The owner must be an active user. Any role may own a node: owning one
lets the user administer it, while the work the node is trusted with
still follows the user's role.

#### Errors

Returns [`LibraryError::Forbidden`](index.md#libraryerror) unless the actor is an operator,
[`LibraryError::NotFound`](index.md#libraryerror) when the node or user is unknown,
[`LibraryError::Invalid`](index.md#libraryerror) when the user is disabled, and the write error
when the update fails.

### verify_node_owner

```rust
pub async fn verify_node_owner(
    store: &LibraryStore,
    user_id: i64,
    node_id: &str,
) -> Result<ConfigActor>
```

Returns a [`ConfigActor::NodeOwner`](#configactor) for `user_id` on `node_id` when the
node's stored owner is that user.

#### Errors

Returns [`LibraryError::Forbidden`](index.md#libraryerror) when the node is unknown or owned by
someone else, and the read error when the lookup fails.

### create_pending_join

```rust
pub async fn create_pending_join(
    store: &LibraryStore,
    join: &NewPendingJoin,
    state: &str,
    verifier: &str,
    keys: &ClusterLinkKeys,
) -> Result<PendingJoin>
```

Records a sign-in in progress: seals `verifier` and `keys` and writes the
row under `hash_token(state)`. Expired rows are purged first.

#### Errors

Returns [`LibraryError::Invalid`](index.md#libraryerror) for a malformed origin, pin, name or
verifier, an error when no master key is configured, and the write
error.

### finish_pending_join

```rust
pub async fn finish_pending_join(store: &LibraryStore, state: &str) -> Result<bool>
```

Ends the pending join for `state`: deletes its row and sealed material.

#### Returns

Whether this call removed the row (`false` when another call already
did, or there was none).

#### Errors

Returns the write error.

### open_pending_join

```rust
pub async fn open_pending_join(
    store: &LibraryStore,
    state: &str,
) -> Result<Option<TakenJoin>>
```

Reads the pending join for `state` without consuming it: its facts and
unsealed secrets, so a redeem that fails for a reason a retry can fix
(the library unreachable, or its answer lost) can be tried again with
the same keys. `None` for an unknown, finished or expired state.

The caller serializes attempts for one state and calls
[`finish_pending_join`](#finish_pending_join) once the outcome is final.

#### Errors

Returns the read error, or [`LibraryError::Invalid`](index.md#libraryerror) when the sealed
material is malformed.

### purge_expired_pending_joins

```rust
pub async fn purge_expired_pending_joins(store: &LibraryStore) -> Result<u64>
```

Deletes every expired pending join and its sealed material. Answers how
many rows went.

#### Errors

Returns the read or write error.

### take_pending_join

```rust
pub async fn take_pending_join(
    store: &LibraryStore,
    state: &str,
) -> Result<Option<TakenJoin>>
```

Consumes the pending join for `state`: answers its facts and secrets once
and deletes both rows. `None` for an unknown, already consumed, or
expired state.

#### Errors

Returns the read or write error, or [`LibraryError::Invalid`](index.md#libraryerror) when the
sealed material is malformed.

### undeploy_instance

```rust
pub async fn undeploy_instance(
    store: &LibraryStore,
    actor: &ConfigActor,
    instance_id: &PluginInstanceId,
) -> Result<usize>
```

Stops `instance_id` everywhere: drops its placement (handing its rows
back) and sets every running deployment `absent`. Returns how many
deployments were stopped. Uninstall runs this for each instance before
it clears the active version.

#### Errors

Returns an error when the actor is not an operator, a write fails, or a
row keeps losing its compare-and-swap ([`LibraryError::Conflict`](index.md#libraryerror)); the
caller must then not treat the instance as stopped.

### delete_plugin_placement

```rust
pub async fn delete_plugin_placement(
    store: &LibraryStore,
    actor: &ConfigActor,
    instance_id: &PluginInstanceId,
) -> Result<bool>
```

Deletes an instance's placement and hands its deployments back to the
operator: each `placed` row becomes `present` and each `unplaced` row
`absent`, so every node keeps doing what it does now. Returns false when
there was no placement.

The placement row goes first, so a scheduler pass that starts now finds
no placement and leaves these rows alone. A pass already in flight skips
an instance whose placement is gone, and a row it still manages to write
is handed back by the next pass (see [`schedule_placements`](#schedule_placements)).

#### Errors

Returns an error when the actor is not an operator, a write fails, or a
row keeps losing its compare-and-swap ([`LibraryError::Conflict`](index.md#libraryerror)).

### effective_node_labels

```rust
pub fn effective_node_labels(node: &NodeRecord) -> BTreeSet<String>
```

A node's operator labels plus `node=<id>` and `target=<target>`.

### ensure_hand_managed

```rust
pub async fn ensure_hand_managed(
    store: &LibraryStore,
    instance_id: &PluginInstanceId,
) -> Result<()>
```

Fails when the scheduler owns `instance_id`'s deployments, so a
hand-written enable or disable is not silently undone on its next pass.

#### Errors

Returns a conflict naming the placement when one exists.

### labels_satisfy

```rust
pub fn labels_satisfy(labels: &BTreeSet<String>, constraints: &[String]) -> bool
```

True when `labels` satisfy every constraint.

### list_plugin_placements

```rust
pub async fn list_plugin_placements(store: &LibraryStore) -> Result<Vec<PluginPlacement>>
```

Every placement, by instance id.

#### Errors

Returns an error when the read fails.

### live_node_ids

```rust
pub async fn live_node_ids(store: &LibraryStore) -> Result<BTreeSet<String>>
```

Nodes whose daemon is live: its last heartbeat in `node_status` is
younger than three heartbeat intervals.

#### Errors

Returns an error when the read fails.

### load_plugin_placement

```rust
pub async fn load_plugin_placement(
    store: &LibraryStore,
    instance_id: &PluginInstanceId,
) -> Result<Option<PluginPlacement>>
```

The placement of `instance_id`, if it has one.

#### Errors

Returns an error when the read fails.

### plan_placement

```rust
pub fn plan_placement(
    spec: &PlacementSpec,
    version_targets: Option<&BTreeSet<String>>,
    demand: GuestReservation,
    nodes: &[NodeCandidate],
    present: &BTreeSet<String>,
) -> PlacementPlan
```

Plans one instance.

`version_targets` is the active release's artifact targets (`None` for a
plugin with no release), `demand` what one guest of the instance needs,
and `present` the nodes that already run it.

A node qualifies when it is live, schedulable, satisfies the
constraints, and has an artifact for its target. Replicas stay where
they run while their node qualifies. New replicas go to the qualifying
nodes with the most room for this demand, and only to nodes where its
memory fits the budget. A scale-down drops the replicas on the nodes
with the least room first. When a replica's node is over its memory
budget and another qualifying node has room, the replica moves there
(one move per pass, so the cluster does not thrash).

### put_plugin_placement

```rust
pub async fn put_plugin_placement(
    store: &LibraryStore,
    actor: &ConfigActor,
    instance_id: &PluginInstanceId,
    spec: &PlacementSpec,
    expected_revision: Option<i64>,
) -> Result<PlacementWrite>
```

Writes placement intent for an instance.

`expected_revision` is `None` (or `Some(0)`) to create a placement and the
current revision to replace one. Every node is woken so the scheduler,
wherever it runs, applies the change.

#### Errors

Returns an error when the actor is not an operator, the spec is invalid,
the instance is missing, or the write fails.

### schedule_placements

```rust
pub async fn schedule_placements(store: &LibraryStore) -> Result<ScheduleReport>
```

One scheduler pass: plans every placement against the nodes' last
heartbeats and writes the deployments the plans call for. Call it only
while holding the [`SCHEDULER_LEASE`](#scheduler_lease).

Headroom is tracked across the pass: a replica placed for one instance
is committed against its node before the next instance is planned.

#### Errors

Returns an error when a read fails or a deployment write is refused.

### set_node_labels

```rust
pub async fn set_node_labels(
    store: &LibraryStore,
    actor: &ConfigActor,
    node_id: &str,
    labels: &[String],
) -> Result<NodeRecord>
```

Replaces a node's operator labels and wakes the scheduler.

#### Errors

Returns an error when the actor is not an operator, a label is invalid or
reserved, there are too many labels, or the node is unknown.

### validate_constraint

```rust
pub fn validate_constraint(constraint: &str) -> Result<()>
```

Checks a constraint: a label, optionally prefixed with `!` to exclude it.

#### Errors

Returns an error when the label part does not fit the grammar.

### validate_label

```rust
pub fn validate_label(label: &str) -> Result<()>
```

Checks the label grammar: `name` or `name=value`, lowercase letters,
digits, and `._/-`, the name starting with a letter or digit, at most 63
bytes.

#### Errors

Returns an error when `label` does not fit the grammar.

### validate_node_label

```rust
pub fn validate_node_label(label: &str) -> Result<()>
```

Checks an operator label: the grammar of [`validate_label`](#validate_label), and not one
of the implicit names every node already carries.

#### Errors

Returns an error for a malformed or reserved label.

### artifact_served_to_node

```rust
pub async fn artifact_served_to_node(
    store: &LibraryStore,
    node_id: &str,
    archive_sha256: &str,
) -> Result<bool>
```

True when the archive `archive_sha256` is stored in the library and is
an artifact of the active version of a plugin deployed to `node_id`.

#### Errors

Returns an error when a read fails.

### record_remote_observation

```rust
pub async fn record_remote_observation(
    store: &LibraryStore,
    node_id: &str,
    incarnation: &str,
    observation: &RemoteObservation<'_>,
) -> Result<bool>
```

Records a Host's observation of one of its deployments. Answers `false`
(and writes nothing) for a deployment that does not target the node, a
status a Host may not report, or a node trusted with nothing.

#### Errors

Returns an error when a read or the write fails.

### remote_deployments_for_node

```rust
pub async fn remote_deployments_for_node(
    store: &LibraryStore,
    node_id: &str,
) -> Result<Vec<RemoteDeploymentEntry>>
```

The deployments that target `node_id` and should run, in deployment-id
order. Empty for a node trusted with nothing.

#### Errors

Returns an error when a read fails.

### remote_instance_for_node

```rust
pub async fn remote_instance_for_node(
    store: &LibraryStore,
    node_id: &str,
    instance_id: &str,
) -> Result<Option<RemoteInstance>>
```

`instance_id` when it has a deployment on `node_id` that should run and
the node is trusted with something; `None` otherwise (an unknown
instance and one deployed elsewhere read the same).

#### Errors

Returns an error when a read fails.

### approve_remote_plugin

```rust
pub async fn approve_remote_plugin(
    store: &LibraryStore,
    approval: &RemoteApproval<'_>,
) -> Result<RemotePluginConsent>
```

Stores an approval, but only when the recorded request is still the
one the operator reviewed: a request that changed in between (a new
version, a wider grant, another archive) answers [`LibraryError::Conflict`](index.md#libraryerror) and nothing
is written. An automatic approval ([`RemoteApprover::Platform`](#remoteapprover)) is
likewise refused while a revoke is recorded (an operator approval
clears it) and over an operator's approval, which it would replace
along with its narrowing.

#### Errors

Returns [`LibraryError::NotFound`](index.md#libraryerror) when no request is recorded,
[`LibraryError::Conflict`](index.md#libraryerror) when it changed or an automatic approval
meets a revoke or an operator's approval, [`LibraryError::Invalid`](index.md#libraryerror) for a malformed field (an
operator identity of [`REMOTE_CONSENT_PLATFORM`](#remote_consent_platform) among them), and the
write error.

### approved_remote_payload_digests

```rust
pub async fn approved_remote_payload_digests(
    store: &LibraryStore,
) -> Result<BTreeSet<String>>
```

Payload-root digests of every owner tree an operator approval still
covers, which a sweep of the shared version cache must keep: owner
versions have no local deployment row to reference them.

#### Errors

Returns the read error.

### list_remote_plugin_consents

```rust
pub async fn list_remote_plugin_consents(
    store: &LibraryStore,
    cluster_id: Option<&str>,
) -> Result<Vec<RemotePluginConsent>>
```

Every consent row, of one cluster when `cluster_id` is given, by
cluster then plugin key.

#### Errors

Returns an error when the read fails.

### load_remote_plugin_consent

```rust
pub async fn load_remote_plugin_consent(
    store: &LibraryStore,
    cluster_id: &str,
    plugin_key: &str,
) -> Result<Option<RemotePluginConsent>>
```

The consent row for `plugin_key` of `cluster_id`.

#### Errors

Returns an error when the read fails.

### record_remote_plugin_request

```rust
pub async fn record_remote_plugin_request(
    store: &LibraryStore,
    cluster_id: &str,
    plugin_key: &str,
    version: &str,
    requested_grant_json: &str,
    artifact: RemoteArtifactRef<'_>,
) -> Result<RemotePluginConsent>
```

Records what the owner asks to run here: `version` of `plugin_key` with
the owner's approved grant, fetched from `artifact`. A stored approval
is kept; whether it still covers the new request is decided when the
grant is evaluated (and an approval of another archive never does,
[`RemotePluginConsent::approved_artifact_matches`](#remotepluginconsentapproved_artifact_matches)). Writes nothing
when the request is unchanged.

#### Errors

Returns [`LibraryError::Invalid`](index.md#libraryerror) for an empty or oversized field or a
grant that is not a JSON object, and the write error.

### remote_request_digest

```rust
pub fn remote_request_digest(
    version: &str,
    requested_grant_json: &str,
    artifact: RemoteArtifactRef<'_>,
) -> String
```

The digest an operator approves a request by: SHA-256 (lowercase hex)
over the requested version, the owner's grant exactly as recorded and
the archive's URL, digest and payload-root digest, each length-prefixed,
so any change to any of them gives another digest.

#### Arguments

* `version` - The requested version.
* `requested_grant_json` - The owner's grant as recorded.
* `artifact` - The archive this host would fetch.

#### Returns

64 lowercase hex characters.

### revoke_remote_plugin

```rust
pub async fn revoke_remote_plugin(
    store: &LibraryStore,
    cluster_id: &str,
    plugin_key: &str,
) -> Result<Option<RemotePluginConsent>>
```

Clears the approval of `plugin_key` of `cluster_id`, keeping its
request, and records the revoke so no automatic approval brings it
back. Answers the row, or `None` when nothing is recorded.

#### Errors

Returns an error when a read or the write fails.

### accept_scan_results

```rust
pub async fn accept_scan_results(
    store: &LibraryStore,
    node_id: &str,
    job_id: &str,
    generation: i64,
    plugin_key: &str,
    books: Vec<NewBook>,
) -> Result<ScanAccept>
```

Validates one batch of a Host's scan rows and writes it to the library.

`books` are the rows as the guest produced them; their `uuid`,
`plugin_key` and `source` are ignored and set here (`source` from the
account row), so a Host cannot address an existing row by id or label
its rows as another plugin's.

#### Errors

Returns an error when a read or a write fails.

### claim_remote_job

```rust
pub async fn claim_remote_job(
    store: &LibraryStore,
    node_id: &str,
    resource_class: JobResourceClass,
    settings: RemoteClaimSettings,
    filter: Option<&JobClaimFilter>,
) -> Result<Option<RemoteJobClaim>>
```

Claims the best job `node_id` may run and unseals its secrets.

The claim is [`claim_routed_job`](#claim_routed_job) as a [`Claimant::Remote`](#claimant) with the
lease owner [`remote_lease_owner`](#remote_lease_owner); [`handoff_routed_job`](#handoff_routed_job) then re-checks
trust, and only then are the job's account secrets
([`job_secret_refs`](#job_secret_refs)) unsealed. The refs handed are recorded under the
claim's lease generation before the secrets are returned, and are the
only ones [`write_back_job_secret`](#write_back_job_secret) accepts under that fence. A job whose
secrets cannot be unsealed is failed with [`SECRETS_UNAVAILABLE`](#secrets_unavailable) (so it
retries with backoff and any node holding the database may take it) and
`None` is returned.

`filter` is the Host's own narrowing ([`JobClaimFilter`](#jobclaimfilter)): only jobs of
its kinds whose every plugin it consented to, and whose one source
plugin can be told, are claimed.

#### Errors

Returns an error when a read or write fails. Permits taken for a failed
attempt are released, and a claim whose plugin keys could not be read
or whose handoff could not be recorded is given back unspent (permits
included), on a best-effort basis.

### finish_remote_job

```rust
pub async fn finish_remote_job(
    store: &LibraryStore,
    node_id: &str,
    job_id: &str,
    generation: i64,
    limits: JobClaimLimits,
    outcome: RemoteOutcome<'_>,
) -> Result<RemoteFinish>
```

Completes or fails the job `node_id` holds at `generation`, then gives
its semaphore permits back.

A failure is always accepted under a live fence, so the job retries
promptly. A completion also needs the node to still be trusted with the
job, and is refused for an acquire, which completes only when its
upload is committed.

#### Errors

Returns an error when a read or write fails.

### job_secret_refs

```rust
pub async fn job_secret_refs(
    store: &LibraryStore,
    job: &JobRecord,
) -> Result<Vec<JobSecretRef>>
```

The store-account secrets `job` needs on a Host.

For a scan or acquire that names an account: every `integration` row of
`encrypted_secrets` for each store account the name reaches, by id or
label and ignoring case, as the executor resolves it (the same accounts
the trust predicate checked), limited to the pinned instance's plugin
when the job is pinned to one. Every other job is handed no secrets.
Sorted, without duplicates.

#### Errors

Returns an error when a read fails.

### live_remote_job

```rust
pub async fn live_remote_job(
    store: &LibraryStore,
    node_id: &str,
    job_id: &str,
    generation: i64,
) -> Result<Option<JobRecord>>
```

The job `node_id` holds at `generation`, while that fence is the job's
live lease and the node is still trusted with the job; `None` when it
is lost.

#### Errors

Returns an error when a read fails.

### release_remote_job

```rust
pub async fn release_remote_job(
    store: &LibraryStore,
    node_id: &str,
    job_id: &str,
    generation: i64,
    resumed: bool,
    limits: JobClaimLimits,
) -> Result<RemoteFinish>
```

Gives back the job `node_id` holds at `generation` without running it:
the job returns to `pending`, and its semaphore permits and handoff rows
are released.

For a Host whose operator no longer consents to the job's plugin, and
for a `work` answer that cannot reach the Host (its connection was
superseded or is closing), so the job is claimable again at once instead
of only after its lease expires. `resumed` is the claim's
[`RemoteJobClaim::resumed`]: a resumed claim gets its resume flag back,
any other its attempt.

The attempt is not spent, unless the job carries a checkpoint (it may
have resumed, which spent none, or saved progress under this claim), in
which case it is kept as spent and the job resumes from the checkpoint.
Nothing is refunded twice: a release never lowers the count below what
the claim raised it by. The release is accepted from a node that is no
longer trusted, since it only gives work back.

#### Returns

[`RemoteFinish::Done`](#remotefinish) when the claim was given back;
[`RemoteFinish::Lost`](#remotefinish) when the fence no longer owns the job.

#### Errors

Returns an error when a read or write fails.

### remote_job_fence

```rust
pub fn remote_job_fence(node_id: &str, job_id: &str, generation: i64) -> JobFence
```

The fence a Host holds for `job_id` at lease generation `generation`.

### remote_lease_owner

```rust
pub fn remote_lease_owner(node_id: &str) -> String
```

Lease owner of every job `node_id` runs over the node API.

### renew_remote_job

```rust
pub async fn renew_remote_job(
    store: &LibraryStore,
    node_id: &str,
    job_id: &str,
    generation: i64,
    settings: RemoteClaimSettings,
    progress: Option<&str>,
) -> Result<RemoteRenew>
```

Extends the lease of the job `node_id` holds at `generation`, and its
semaphore permits, recording `progress` when given.

Trust and the node credential are re-read first: a node that is no
longer trusted with the job (revoked, owner demoted, account unlinked)
gets [`RemoteRenew::Lost`](#remoterenew), and the lease then expires as for a crashed
worker. A fence that expired and was reclaimed is lost as well, as is a
job whose permit another holder took over. Permits are checked before
the job lease is extended, so a renewal that answers `Lost` never pushes
the reclaim out; should a permit be taken over between that check and
its renewal, the just-extended lease is cut back to expire at once.

#### Errors

Returns an error when a read or write fails.

### write_back_job_secret

```rust
pub async fn write_back_job_secret(
    store: &LibraryStore,
    node_id: &str,
    job_id: &str,
    generation: i64,
    secret_ref: &JobSecretRef,
    value: &[u8],
) -> Result<WriteBack>
```

Stores a refreshed store token a Host writes back.

Accepted only when all of these hold, checked immediately before the
write: the node still has a live credential and is still trusted with
the job; `generation` is the job's live lease generation for this node;
`secret_ref` was recorded as handed with the job under that generation
(a secret created for the account after the handoff is refused); and it
is still one of the job's secrets ([`job_secret_refs`](#job_secret_refs)), so it still
exists and its account is still linked. The value is re-sealed with this
node's DEK under the same identity.

#### Errors

Returns an error when a read, the seal, or the write fails.

### sweep_job_requirements

```rust
pub async fn sweep_job_requirements(
    store: &LibraryStore,
    node_bound_plugin_keys: &BTreeSet<String>,
    grace: Duration,
    now: DateTime<Utc>,
) -> Result<RequirementsSweepReport>
```

Re-resolves the requirements of every pending routed job (see the module
docs for the rules).

`node_bound_plugin_keys` are the plugin keys of node-bound outputs. A
recorded instance whose key is missing from them is treated as no longer
required, so the caller must pass a set that does not depend on what the
sweeping node has installed, and must not sweep with a set it could not
read. `grace` is normally [`REQUIREMENTS_GRACE`](#requirements_grace); `now` is the sweep's
clock.

#### Errors

Returns an error when a read or write fails; jobs handled before the
failure keep their writes.

### rank_nodes

```rust
pub fn rank_nodes(running: &[NodeCandidate], demand: GuestReservation) -> Vec<String>
```

Ranks `running` nodes for `demand`, best first (most room, then node id).

### route_ranks

```rust
pub async fn route_ranks(
    store: &LibraryStore,
    node_id: &str,
    claim_ids: &[String],
) -> Result<BTreeMap<String, RouteRank>>
```

For each claim id that is a plugin instance id, `node_id`'s rank among
the live nodes whose guest of that instance is up (`running` or
`healthy`). Ids that are not instance ids (plugin keys, aliases, node
consumers) are left out: they are not routed. A node missing from the
list (no heartbeat yet) ranks after every live node.

#### Errors

Returns an error when a read fails.

### list_plugin_schemas

```rust
pub async fn list_plugin_schemas(store: &LibraryStore) -> Result<Vec<PinnedPluginSchema>>
```

Every pinned schema, ordered by plugin key.

#### Errors

Returns an error when the read fails or a row cannot be decoded.

### load_plugin_schema

```rust
pub async fn load_plugin_schema(
    store: &LibraryStore,
    plugin_key: &str,
) -> Result<Option<PinnedPluginSchema>>
```

The pinned schema for `plugin_key`, if any node has pinned one.

#### Errors

Returns an error when the read fails or the stored JSON is not a schema
this binary understands.

### pin_plugin_schema

```rust
pub async fn pin_plugin_schema(
    store: &LibraryStore,
    pin: &SchemaPin<'_>,
) -> Result<SchemaPinOutcome>
```

Pins `pin` according to the version rule in the module docs.

An unchanged row is left alone, including `recorded_at`. The insert and
the update are both conditional on the row read just before, and a lost
race re-reads and decides again, so a concurrent newer pin is never
overwritten by an older one.

#### Errors

Returns an error when `package_version` is not semver, the schema fails
to serialize, the write fails, or the row keeps changing underneath.

### pin_version_schema

```rust
pub async fn pin_version_schema(
    store: &LibraryStore,
    pin: &SchemaPin<'_>,
) -> Result<SchemaPinOutcome>
```

Pins the schema of the plugin's active release.

The pinned schema of a released plugin is the active release's schema,
so an older version is pinned too ([`SchemaPinChange::RolledBack`](#schemapinchange))
instead of being refused as stale.

#### Errors

As [`pin_plugin_schema`](#pin_plugin_schema).

### require_plugin_schema

```rust
pub async fn require_plugin_schema(
    store: &LibraryStore,
    plugin_key: &str,
) -> Result<PinnedPluginSchema>
```

Loads the pinned schema or fails closed with an operator-facing message.

#### Errors

Returns [`LibraryError::NotFound`](index.md#libraryerror) when no node has pinned the plugin yet.

### schema_fingerprint

```rust
pub fn schema_fingerprint(schema: &SettingsSchema) -> Result<String>
```

Hex SHA-256 of the schema's canonical JSON. Two manifests that declare
the same settings fingerprint the same, whatever their bytes.

#### Errors

Returns an error when the schema fails to serialize.

### stale_against_pin

```rust
pub async fn stale_against_pin(
    store: &LibraryStore,
    plugin_key: &str,
    local_version: &str,
) -> Result<Option<PinnedPluginSchema>>
```

Whether the pinned row for `plugin_key` is newer than `local_version`.

`None` when nothing is pinned yet. Used by the reconciler before a spawn
so a node that was not the one to upgrade the pin still refuses to run an
older package against documents written for the newer schema.

#### Errors

Returns an error when the read fails or `local_version` is not semver.

### align_cluster_root

```rust
pub async fn align_cluster_root(
    store: &LibraryStore,
    files_dir: &Path,
    password: Option<&str>,
) -> Result<ClusterSecret>
```

Ensures this process's DEK is the cluster secret root.

* Existing fingerprint: the local key must match. A missing file is not minted.
* No fingerprint and sealed secrets present: the local key must unseal one
  of them, then the fingerprint is recorded. A missing or wrong key fails
  without writing a new root.
* No fingerprint and no sealed secrets: the local key (minted when absent)
  is recorded. Concurrent initializers: one fingerprint wins; a loser that
  minted a different key deletes that new file and fails.

On success the accepted DEK is installed in the process cache. The returned
value is only the cluster id and fingerprint.

#### Errors

Returns an error when bootstrap material is missing or does not match the
database. Existing rows are not updated on failure.

### load_cluster_row

```rust
pub async fn load_cluster_row(store: &LibraryStore) -> Result<Option<Model>>
```

Reads the singleton cluster row when present.

#### Errors

Returns an error when the read fails.

### acquire_permit

```rust
pub async fn acquire_permit(
    store: &LibraryStore,
    name: &str,
    limit: u32,
    holder: &str,
    ttl: Duration,
) -> Result<Option<SemaphorePermit>>
```

Takes one permit of `name` for `holder`, valid for `ttl`.

Tries the free or expired slots below `limit` in order; each attempt is
a fenced compare-and-swap, so a concurrent acquirer that wins a slot
makes this one move on. `holder` should be unique per piece of work
(it is also how a renewal proves ownership).

Live permits other holders keep on slots at or above `limit` (left over
from a higher limit) count against it, so after a limit drop no permit
is granted until the live count is under the new limit.

#### Returns

The permit, or `None` when `limit` live permits are held by others.

#### Errors

Returns an error when a read or write fails.

### acquire_permits

```rust
pub async fn acquire_permits(
    store: &LibraryStore,
    requests: &[SemaphoreRequest],
    holder: &str,
    ttl: Duration,
) -> Result<PermitAcquire>
```

Takes one permit of every semaphore in `requests`, or none.

Requests are taken in name order (duplicates merge to the smallest
limit). When one semaphore is full, the permits already taken are
released and the blocked name is returned.

#### Errors

Returns an error when a read or write fails; permits taken before the
failure are released on a best-effort basis.

### live_permits

```rust
pub async fn live_permits(
    store: &LibraryStore,
    name: &str,
    max_slots: u32,
) -> Result<Vec<SemaphorePermit>>
```

Live permits of `name` among its first `max_slots` slots (expired slots
are not counted).

#### Errors

Returns an error when the read fails.

### permits_held_by

```rust
pub async fn permits_held_by(
    store: &LibraryStore,
    name: &str,
    max_slots: u32,
    holder_prefix: &str,
) -> Result<Vec<SemaphorePermit>>
```

Permits of `name` among its first `max_slots` slots whose holder starts
with `holder_prefix`, expired ones included.

A worker that holds permits only by name (a host that reconnected to
another owner node) finds them again with this. An expired slot nobody
took over is still returned, since [`renew_fenced_lease`](#renew_fenced_lease) still renews
it; a slot another holder took is not.

#### Errors

Returns an error when the read fails.

### release_permits

```rust
pub async fn release_permits(
    store: &LibraryStore,
    permits: &[SemaphorePermit],
) -> Result<()>
```

Gives back every permit in `permits` so the slots are free at once.

A permit whose fence was lost is left alone (the release matches no row).

#### Errors

Returns an error when a write fails.

### renew_fenced_lease

```rust
pub async fn renew_fenced_lease(
    store: &LibraryStore,
    lease: &ClusterLease,
    ttl: Duration,
) -> Result<bool>
```

Extends `lease` by `ttl` only while the row still carries its holder and
token. Unlike [`acquire_lease`](#acquire_lease), it never takes a lease over.

A lease that has expired but that nobody has taken is still renewed: the
unchanged token proves no other holder ran in between.

#### Returns

`false` when another holder took the lease (the fence is lost).

#### Errors

Returns an error when the write fails.

### renew_permits

```rust
pub async fn renew_permits(
    store: &LibraryStore,
    permits: &[SemaphorePermit],
    ttl: Duration,
) -> Result<bool>
```

Extends every permit in `permits` by `ttl` while its holder and token
still own the slot.

#### Returns

`false` when any permit was taken over (its fence is lost); the
remaining permits are still renewed.

#### Errors

Returns an error when a write fails.

### semaphore_slot_lease_name

```rust
pub fn semaphore_slot_lease_name(name: &str, slot: u32) -> String
```

Lease name of slot `slot` of semaphore `name`.

### accounts_owned_by

```rust
pub async fn accounts_owned_by(
    store: &LibraryStore,
    user_id: i64,
) -> Result<BTreeSet<AccountKey>>
```

Store accounts linked to `user_id` through any of the user's identities.

#### Errors

Returns an error when a read fails.

### book_matches_title

```rust
pub fn book_matches_title(book: &BookRecord, id: &str) -> bool
```

True when `id` names `book` (uuid, product id, ISBN or ASIN, any case),
as an acquire's title filter matches.

### claimant_may_run

```rust
pub fn claimant_may_run(claimant: Claimant, job: &JobRecord) -> bool
```

True when a `claimant` may run `job` at all, whatever it is trusted
with: a forced or PDF-only acquire runs only on a node that holds the
library database, since the host acquire path does neither.

### handoff_routed_job

```rust
pub async fn handoff_routed_job(
    store: &LibraryStore,
    request: &JobClaimRequest<'_>,
    claim: RoutedJobClaim,
) -> Result<Option<RoutedJobClaim>>
```

Re-checks a claimed job just before its secrets are handed to the
claiming node.

The claim is returned unchanged only when both hold:

- the claim still owns the job: the row is `running` under the claim's
  fence (same lease owner and generation), that owner is
  `request.owner`, and the lease has not expired. A claim that was
  reclaimed, finished, released, or left to expire is not handed over;
- the node is still trusted with the job ([`node_may_claim_job`](#node_may_claim_job)).

Ownership is checked again after the trust reads, so a lease lost while
they ran is refused too.

Otherwise `None` is returned, so nothing is unsealed. When the claim was
taken by `request.owner`, the job goes back to `pending` without spending
an attempt if the claim's fence still owns the row (a row another fence
owns is left alone), and the claim's permits are released (a permit taken
over since holds a new token and is left alone too). A claim with another
lease owner is refused without touching the row or its permits.

#### Errors

Returns an error when a read or write fails. A failure while releasing
leaves the lease to expire, after which the job is redelivered.

### job_item_accounts

```rust
pub async fn job_item_accounts(
    store: &LibraryStore,
    job: &JobRecord,
) -> Result<Option<BTreeSet<AccountKey>>>
```

The store accounts a scan or acquire touches: its items.

- A scan that names an account touches every `accounts` row with that
  id (within the pinned instance's plugin). A scan of every account is
  `None`: what it touches is not known until it runs.
- An acquire touches the named account's rows, plus the account of every
  library row its filters match (title id, account, pinned plugin),
  whatever the row's acquire status. That is a superset of the titles it
  will fetch, so a node must be trusted with at least everything it
  fetches.
- Anything else, and a job pinned to an unknown instance, is `None`.

#### Errors

Returns an error when a read fails.

### job_trusted

```rust
pub async fn job_trusted(
    store: &LibraryStore,
    trust: &NodeTrust,
    job: &JobRecord,
) -> Result<bool>
```

True when a node with `trust` may receive `job`.

A node trusted only with one user's work receives a scan or acquire
that names an account (`owned_job_account`) only when every store
account the name reaches is linked to that user.

#### Errors

Returns an error when a read fails.

### node_may_claim_job

```rust
pub async fn node_may_claim_job(
    store: &LibraryStore,
    node_id: &str,
    claimant: Claimant,
    job: &JobRecord,
) -> Result<bool>
```

True when `node_id` may claim `job` right now.

#### Errors

Returns an error when a read fails.

### node_trust

```rust
pub async fn node_trust(
    store: &LibraryStore,
    node_id: &str,
    claimant: Claimant,
) -> Result<NodeTrust>
```

Resolves what `node_id` is trusted with right now.

#### Errors

Returns an error when a read fails.

### plan_acquire

```rust
pub async fn plan_acquire(
    store: &LibraryStore,
    requester: ScanRequester,
    account: Option<&str>,
    titles: &[String],
    plugin_instance_id: Option<&str>,
) -> Result<Vec<JobPayload>>
```

Plans the acquire jobs one request admits.

`titles` are title ids (uuid, ASIN, ISBN, product id); each becomes its
own job, so each title can go to any node trusted with it. `account`
narrows to one store account and `plugin_instance_id` pins every job.

- Operator authority, an Owner or an Administrator: one job per title,
  or one job for everything pending (it touches many accounts, and only
  nodes trusted with every one of them claim it; splitting it per
  account would repeat the storage listing for every account).
- Any other active user: a named account must be theirs, and so must
  every library row a named title matches. A title named without an
  account becomes one job per account id it is in, since a node trusted
  only with its owner's work takes only jobs that name an account. With
  neither, the request becomes one job per account they wholly own, so
  each can go to a node trusted with it; the plan is empty when they own
  none.

#### Errors

Returns [`LibraryError::Forbidden`](index.md#libraryerror) when the user is unknown or disabled,
or names an account or title that is not theirs (unknown ones read the
same), [`LibraryError::Invalid`](index.md#libraryerror) for a malformed instance id, and the
read error when a read fails.

### plan_scan

```rust
pub async fn plan_scan(
    store: &LibraryStore,
    requester: ScanRequester,
    account: Option<&str>,
    plugin_instance_id: Option<&str>,
) -> Result<Vec<JobPayload>>
```

Plans the scan jobs one request admits.

`account` names one store account; `plugin_instance_id` pins the scan to
one instance (and so to its plugin's accounts).

- Operator authority, an Owner or an Administrator: a named account is
  one job as asked. A scan of every account becomes one job per distinct
  scan-enabled account id (within the pinned plugin), so the planner can
  route each to any node trusted with that account. When the id also
  names a scan-disabled account on another plugin, its job is pinned to
  the default instance of each plugin where it is scan-enabled, so the
  disabled account is never touched. With no scan-enabled
  account the request stays one unsplit job, which only fully trusted
  nodes may claim and which reports that nothing was scanned.
- Any other active user: a named account must be theirs, and a scan of
  every account covers only the scan-enabled accounts they own. An
  account id that another user also holds on a different plugin is left
  out, since its job would touch both. The plan is empty when the user
  owns nothing to scan.

#### Errors

Returns [`LibraryError::Forbidden`](index.md#libraryerror) when the user is unknown or disabled,
or names an account that is not theirs (an unknown account reads the
same), [`LibraryError::Invalid`](index.md#libraryerror) for a malformed instance id, and the
read error when a read fails.

### remote_node_may_view_job

```rust
pub async fn remote_node_may_view_job(
    store: &LibraryStore,
    node_id: &str,
    job: &JobRecord,
) -> Result<bool>
```

True when a remote node may read `job`'s state.

A node trusted with everything reads every job. A node trusted only
with its owner's work reads any scan or acquire that names an account
and touches only its owner's accounts ([`job_item_accounts`](#job_item_accounts), within a
pinned job's plugin), including a pinned one it could not
claim itself, since its owner may queue those through it
([`plan_scan`](#plan_scan)). A node trusted with nothing reads none.

#### Errors

Returns an error when a read fails.

### remote_node_requester

```rust
pub async fn remote_node_requester(
    store: &LibraryStore,
    node_id: &str,
) -> Result<Option<ScanRequester>>
```

Whom library work a remote node asks for runs as (a `scan.request` over
the node API): the user who owns the node, while the node is trusted
with anything. `None` for a node that is trusted with nothing (revoked,
ownerless, a disabled owner) or unknown, which may ask for nothing.

#### Errors

Returns an error when a read fails.

### abort_job_uploads

```rust
pub async fn abort_job_uploads(
    store: &LibraryStore,
    node_id: &str,
    job_id: &str,
    generation: i64,
) -> Result<Vec<UploadSession>>
```

Aborts every open upload `node_id` was granted under the job's lease
generation `generation` (the job finished or its fence was lost, so
none of them can commit); answers the aborted sessions.

#### Errors

Returns an error when a read or write fails.

### abort_upload

```rust
pub async fn abort_upload(
    store: &LibraryStore,
    upload_id: &str,
) -> Result<Option<UploadSession>>
```

Aborts an open upload (its fence was lost, its commit was refused for
a lost fence, or the destinations changed). Answers the session as it
stood when this call aborted it, its receipts final (no file is
recorded on an aborted upload), so the receiving node can delete every
copy it received; `None` when it was not open.

#### Errors

Returns an error when a read or write fails.

### authorize_upload

```rust
pub async fn authorize_upload(
    store: &LibraryStore,
    upload_id: &str,
    token: &str,
    allow_committed: bool,
) -> Result<Option<UploadSession>>
```

The upload `token` opens: `upload_id`'s row when the token's hash
matches, the token has not expired, and the upload was neither aborted
nor (unless `allow_committed`) committed.

#### Errors

Returns an error when the read fails.

### commit_upload

```rust
pub async fn commit_upload(
    store: &LibraryStore,
    upload_id: &str,
    finish: Option<UploadFinish<'_>>,
    limits: JobClaimLimits,
) -> Result<UploadCommit>
```

Commits an upload whose every file was received.

A title's upload marks the title acquired under its primary key (and
its PDF, when uploaded) and publishes `book_acquired`; it never
completes the job, so `finish` must be `None`. The job's closing upload
(no title) must carry `finish`, and completes the job with its detail
as progress and gives its semaphore permits back, but only when no
title of the job is left unacquired and the job's cancel was not
requested; otherwise it is refused (`titles_remaining`,
`job_cancelled`) and nothing is written.

One canonical batch, every statement gated on the upload still being
open, the job still running under this upload's unexpired fence, the node's
credential still being live, the node still trusted with the title
(`commit_trust_gate`), and the upload still holding every key it was
granted (`upload_holds_its_keys`); trust and the keys are also
re-checked just before. A key another title or an acquire took since
the grant refuses the commit (`key_taken`) and records nothing. A repeat
of a commit that already landed answers [`UploadCommit::Committed`](#uploadcommit)
again without writing.

#### Errors

Returns an error when a read or the batch fails.

### extend_job_uploads

```rust
pub async fn extend_job_uploads(
    store: &LibraryStore,
    node_id: &str,
    job_id: &str,
    generation: i64,
) -> Result<u64>
```

Extends the token of every open upload `node_id` holds under the job's
lease generation `generation`, after that lease was renewed; answers
how many were extended.

#### Errors

Returns an error when the write fails.

### grant_upload

```rust
pub async fn grant_upload(
    store: &LibraryStore,
    node_id: &str,
    job_id: &str,
    generation: i64,
    receiving_node_id: &str,
    req: &UploadGrantRequest<'_>,
) -> Result<UploadGrant>
```

Opens an upload of one title of the acquire job `node_id` holds at
`generation`, to be received by `receiving_node_id`.

Granted only when all of these hold: the fence is the job's live lease
for this node and the node is still trusted with the job; the job is an
acquire; the title exists, is one of the job's plugins and accounts (and
the job's title, when it names one), and for a Host trusted only with
its owner's accounts is one of that user's; and the manifest is well
formed, every key inside the title's own scope on its destination
([`UploadDestination`](#uploaddestination)) and none another title's: neither another
title's recorded audio, PDF or audio sidecar (`claims_another_books_key`)
nor any key recorded for another title on that destination
(`claim_upload_keys`, which then claims every key for this upload). An
earlier open upload of the same title by the same job is aborted: the
newest grant wins, and one overtaken by a newer grant of the title
while it was being made is refused (`superseded`) instead. A
grant refused or failed once claiming began claims nothing in the end:
the session row is inserted first and aborted again, and every key
claimed goes back to its prior holder (`undo_grant`).

#### Errors

Returns an error when a read or the insert fails.

### job_titles

```rust
pub async fn job_titles(
    store: &LibraryStore,
    node_id: &str,
    job: &JobRecord,
) -> Result<Option<Vec<BookRecord>>>
```

The owner's library rows the acquire `job` run by `node_id` covers: of
the job's plugins ([`job_plugin_keys`](#job_plugin_keys)), of the accounts its account
selector names (`job_accounts`, so a job naming its account by label
or in another case covers that account's titles) and its title when it
names them, and, for a Host trusted only with its owner's accounts, one
of that user's. `None` when the node is trusted with nothing.

#### Errors

Returns an error when a read fails.

### load_upload

```rust
pub async fn load_upload(
    store: &LibraryStore,
    upload_id: &str,
) -> Result<Option<UploadSession>>
```

Loads one upload by id.

#### Errors

Returns an error when the read fails or the row is malformed.

### names_book

```rust
pub fn names_book(title: &str, book: &BookRecord) -> bool
```

True when `book` is one of the titles `title` names.

### record_upload_file

```rust
pub async fn record_upload_file(
    store: &LibraryStore,
    upload_id: &str,
    index: usize,
) -> Result<bool>
```

Records that file `index` reached every destination and extends the
upload's token. Answers false when the upload is no longer open (it was
aborted meanwhile: the caller deletes the copies it just wrote).

#### Errors

Returns an error when a read or write fails.

### upload_fence_live

```rust
pub async fn upload_fence_live(
    store: &LibraryStore,
    session: &UploadSession,
) -> Result<bool>
```

True while the upload's fence is the job's live lease for its node and
the node is still trusted with the job.

#### Errors

Returns an error when a read fails.

### upload_owns_key

```rust
pub async fn upload_owns_key(
    store: &LibraryStore,
    session: &UploadSession,
    destination: &str,
    key: &str,
) -> Result<bool>
```

True while `key` on `destination` is still `session`'s own: its
recorded latest writer is this upload. A later upload of the title, an
acquire run on this node, or another title may have written the key
since; the receiving node then leaves the file alone when it cleans up
after `session`, so a stale abort never deletes a replacement's file.

#### Errors

Returns an error when the read fails.

### activate_version

```rust
pub async fn activate_version(
    store: &LibraryStore,
    actor: &ConfigActor,
    plugin_key: &str,
    version: &str,
    node_id: &str,
) -> Result<(PluginInstall, SchemaPinOutcome)>
```

Moves the active pointer of `plugin_key` to `version` (first install,
update, or rollback), pins the version's settings schema, and wakes every
node. Pointing at the version that is already active is a no-op.

#### Errors

Returns an error when the actor may not write, the version does not
exist, or a write fails.

### add_plugin_version

```rust
pub async fn add_plugin_version(
    store: &LibraryStore,
    actor: &ConfigActor,
    version: PluginVersion,
) -> Result<AddVersion>
```

Records a version. Installing the same `(plugin_key, version)` again
with the same artifact digests is a no-op ([`AddVersion::Existing`](#addversion));
different digests under an installed version are refused, so a version
never changes content behind a node's cache.

`installed_at` / `installed_by` are stamped here.

#### Errors

Returns an error when the actor may not install, the version is
malformed, the version exists with other content, or the write fails.

### approve_version_grant

```rust
pub async fn approve_version_grant(
    store: &LibraryStore,
    actor: &ConfigActor,
    plugin_key: &str,
    version: &str,
    grant_json: &str,
) -> Result<PluginVersion>
```

Records the operator-approved consent grant on a version.

#### Errors

Returns an error when the actor may not write, the version is missing,
or the write fails.

### clear_install

```rust
pub async fn clear_install(
    store: &LibraryStore,
    actor: &ConfigActor,
    plugin_key: &str,
) -> Result<bool>
```

Clears the active pointer (`plugins uninstall`). Version rows stay so the
plugin can be re-activated; nodes retire guests whose plugin has no
active version and sweep the cache.

#### Errors

Returns an error when the actor may not write or the delete fails.

### list_installs

```rust
pub async fn list_installs(store: &LibraryStore) -> Result<Vec<PluginInstall>>
```

Every active pointer.

#### Errors

Returns an error when the read fails.

### list_versions

```rust
pub async fn list_versions(
    store: &LibraryStore,
    plugin_key: Option<&str>,
) -> Result<Vec<PluginVersion>>
```

Every version of `plugin_key` (or of every plugin), newest install first.

#### Errors

Returns an error when the read or decode fails.

### load_active_version

```rust
pub async fn load_active_version(
    store: &LibraryStore,
    plugin_key: &str,
) -> Result<Option<PluginVersion>>
```

The active version of `plugin_key`, when a pointer names one.

#### Errors

Returns an error when a read fails or the pointer names a missing row.

### load_artifact_blob

```rust
pub async fn load_artifact_blob(
    store: &LibraryStore,
    sha256: &str,
) -> Result<Option<Vec<u8>>>
```

Reads an archive stored by [`put_artifact_blob`](#put_artifact_blob).

#### Errors

Returns an error when the read fails or the stored bytes do not match
their digest.

### load_install

```rust
pub async fn load_install(
    store: &LibraryStore,
    plugin_key: &str,
) -> Result<Option<PluginInstall>>
```

The active pointer of `plugin_key`.

#### Errors

Returns an error when the read fails.

### load_version

```rust
pub async fn load_version(
    store: &LibraryStore,
    plugin_key: &str,
    version: &str,
) -> Result<Option<PluginVersion>>
```

One version row.

#### Errors

Returns an error when the read or decode fails.

### put_artifact_blob

```rust
pub async fn put_artifact_blob(store: &LibraryStore, bytes: &[u8]) -> Result<String>
```

Stores a small archive and returns its `blob:sha256:<hex>` URL.

#### Errors

Returns an error when the archive exceeds [`MAX_ARTIFACT_BLOB_BYTES`](#max_artifact_blob_bytes) or
the write fails.

### bootstrap_control_plane

```rust
pub async fn bootstrap_control_plane(
    store: &LibraryStore,
    files_dir: &Path,
    scratch_dir: &Path,
    password: Option<&str>,
    seed: &Config,
) -> Result<ControlPlaneSession>
```

Enrolls this files directory and imports `[events]` and `[output]` when
their documents are absent.

`seed` is the effective `config.toml` plus environment configuration; only
its `events` and `output` tables are read, and only for a document that
does not exist yet. After this returns, callers must treat
[`ControlPlaneSession::events`](#controlplanesession) and [`ControlPlaneSession::output`](#controlplanesession) as
authority (see [`overlay_core_documents`](#overlay_core_documents)). A later TOML load or
environment override must not replace them without a compare-and-swap
write.

`scratch_dir` is [`bookclerk_config::Config::download_cache_dir`]: the
heartbeat records the byte size of its `acquire` and `acquire-pdf` trees.

#### Errors

Returns an error when the secret root, cluster binding, schema, or a core
document cannot be established.

### overlay_core_documents

```rust
pub fn overlay_core_documents(config: &mut Config, session: &ControlPlaneSession)
```

Copies every authoritative cluster document in `session` onto `config`.

### overlay_events

```rust
pub fn overlay_events(
    config: &mut Config,
    events: &ConfigurationDocument<EventsSettingsV1>,
    cluster_id: &str,
)
```

Copies the authoritative events document onto `config` for `cluster_id`.

Revisions are only comparable within one cluster. Callers that switch
databases pass the new cluster id explicitly.

### load_events

```rust
pub async fn load_events(
    store: &LibraryStore,
) -> Result<ConfigurationDocument<EventsSettingsV1>>
```

Loads the cluster events document.

#### Errors

Returns an error when the document is missing, unsupported, or invalid.

### import_events_if_absent

```rust
pub async fn import_events_if_absent(
    store: &LibraryStore,
    actor: &ConfigActor,
    seed: &EventsConfig,
    operation_id: &str,
) -> Result<ConfigurationDocument<EventsSettingsV1>>
```

Inserts `[events]` when the cluster document is absent.

An existing document is parsed and returned without reading the seed, so an
obsolete or out-of-range TOML/environment value cannot fail startup.

#### Errors

Returns an error when the stored document is invalid, or when the document
is absent and the seed is invalid or the actor may not import.

### replace_events

```rust
pub async fn replace_events(
    store: &LibraryStore,
    actor: &ConfigActor,
    expected_revision: i64,
    body: &EventsSettingsV1,
    operation_id: &str,
) -> Result<EventsReplace>
```

Replaces the cluster events document.

#### Errors

Returns an error when validation, authorization, or the compare-and-swap fails.

### load_node_runtime

```rust
pub async fn load_node_runtime(
    store: &LibraryStore,
    node_id: &str,
) -> Result<ConfigurationDocument<NodeRuntimeSettingsV1>>
```

Loads one node's runtime document.

#### Errors

Returns an error when the document is missing, unsupported, or invalid.

### import_node_runtime_if_absent

```rust
pub async fn import_node_runtime_if_absent(
    store: &LibraryStore,
    actor: &ConfigActor,
    node_id: &str,
    body: &NodeRuntimeSettingsV1,
    operation_id: &str,
) -> Result<ConfigurationDocument<NodeRuntimeSettingsV1>>
```

Inserts a node runtime document when it is absent.

#### Errors

Returns an error when the body is invalid or the actor may not import.

### replace_node_runtime

```rust
pub async fn replace_node_runtime(
    store: &LibraryStore,
    actor: &ConfigActor,
    node_id: &str,
    expected_revision: i64,
    body: &NodeRuntimeSettingsV1,
    operation_id: &str,
) -> Result<NodeRuntimeReplace>
```

Replaces one node's runtime document.

#### Errors

Returns an error when validation, authorization, or the compare-and-swap fails.

### update_node_runtime

```rust
pub async fn update_node_runtime(
    store: &LibraryStore,
    actor: &ConfigActor,
    node_id: &str,
    edit: impl Fn(&mut NodeRuntimeSettingsV1),
) -> Result<ConfigurationDocument<NodeRuntimeSettingsV1>>
```

Applies `edit` to a node's runtime document (creating it with defaults
when the node never wrote one) under compare-and-swap, retrying a few
times on a concurrent edit. Budgets reach the scheduler with the node's
next heartbeat.

#### Errors

Returns an error when the actor may not write, the edited body is
invalid, or the document keeps changing underneath.

## Type aliases

### LiveRevisions

```rust
pub type LiveRevisions = Vec<(LiveTopic, LiveRevision)>;
```

Every topic's revision, in [`LiveTopic::ALL`](#livetopicall) order.

### AccountKey

```rust
pub type AccountKey = (String, String);
```

A store account, keyed as in `accounts`: `(plugin_key, account_id)`.

## Constants

### BRANCH_TITLE_DETACHED

```rust
pub const BRANCH_TITLE_DETACHED: &str = "detached";
```

`branch_titles.state` of a title from a branch that stopped sharing,
left, or withdrew it.

### BRANCH_TITLE_SHARED

```rust
pub const BRANCH_TITLE_SHARED: &str = "shared";
```

`branch_titles.state` of a title its branch still shares.

### DETACHED_LEFT

```rust
pub const DETACHED_LEFT: &str = "left";
```

Detach reason: the branch left the library (or was revoked).

### DETACHED_STOPPED_SHARING

```rust
pub const DETACHED_STOPPED_SHARING: &str = "stopped_sharing";
```

Detach reason: the branch turned "Share this branch's books" off.

### DETACHED_WITHDRAWN

```rust
pub const DETACHED_WITHDRAWN: &str = "withdrawn";
```

Detach reason: a full pass of the branch's catalog no longer named it.

### FULL_CATALOG_PASS_EVERY

```rust
pub const FULL_CATALOG_PASS_EVERY: Duration = _;
```

How long a branch's catalog sync resumes incrementally before the
library asks for a full pass again, so titles a branch no longer has
(removed, or from a sub-branch that left) are withdrawn within a day.

### CLUSTER_LINK_ACTIVE

```rust
pub const CLUSTER_LINK_ACTIVE: &str = "active";
```

`cluster_links.state` of a link the host client connects.

### CLUSTER_LINK_KEYS_NAME

```rust
pub const CLUSTER_LINK_KEYS_NAME: &str = "node_keys";
```

`encrypted_secrets.name` of a link's node keys.

### CLUSTER_LINK_LEFT

```rust
pub const CLUSTER_LINK_LEFT: &str = "left";
```

`cluster_links.state` after this install left the cluster.

### CLUSTER_LINK_REVOKED

```rust
pub const CLUSTER_LINK_REVOKED: &str = "revoked";
```

`cluster_links.state` once the owner revoked this node (close 4403).

### MAX_OBSERVATION_DETAIL_BYTES

```rust
pub const MAX_OBSERVATION_DETAIL_BYTES: usize = 512;
```

Maximum UTF-8 size of an observation detail string.

### CLUSTER_SCOPE_ID

```rust
pub const CLUSTER_SCOPE_ID: &str = "singleton";
```

Cluster scope instance. There is one cluster document per namespace.

### CLUSTER_SCOPE_TYPE

```rust
pub const CLUSTER_SCOPE_TYPE: &str = "cluster";
```

Scope type stored for cluster documents.

### MAX_CONFIGURATION_DOCUMENT_BYTES

```rust
pub const MAX_CONFIGURATION_DOCUMENT_BYTES: usize = _;
```

Maximum UTF-8 size of `document_json`.

### NODE_SCOPE_TYPE

```rust
pub const NODE_SCOPE_TYPE: &str = "node";
```

Scope type stored for node documents.

### PLUGIN_INSTANCE_CONFIG_NAMESPACE

```rust
pub const PLUGIN_INSTANCE_CONFIG_NAMESPACE: &str = "config";
```

Namespace of a plugin instance's typed configuration document.

### PLUGIN_INSTANCE_SCOPE_TYPE

```rust
pub const PLUGIN_INSTANCE_SCOPE_TYPE: &str = "plugin_instance";
```

Scope type stored for one plugin instance's configuration.

### AUTHORIZED_LINK_CREATED_BY

```rust
pub const AUTHORIZED_LINK_CREATED_BY: &str = "oauth:";
```

Prefix of the `created_by` audit id of a link a sign-in created.

### DEFAULT_LINK_TTL

```rust
pub const DEFAULT_LINK_TTL: Duration = _;
```

Default lifetime of a link.

### ENROLLMENT_TOKEN_PREFIX

```rust
pub const ENROLLMENT_TOKEN_PREFIX: &str = "bcn1";
```

Version tag that starts every enrollment token.

### HOST_ENROLLMENT_NAMESPACE

```rust
pub const HOST_ENROLLMENT_NAMESPACE: &str = "core.host_enrollment";
```

Namespace of the cluster's host-enrollment switch.

### HOST_ENROLLMENT_SCHEMA_VERSION

```rust
pub const HOST_ENROLLMENT_SCHEMA_VERSION: i64 = 1;
```

Schema version of [`HostEnrollmentSettingsV1`](#hostenrollmentsettingsv1).

### MAX_LINK_LABEL_BYTES

```rust
pub const MAX_LINK_LABEL_BYTES: usize = 64;
```

Longest `label` stored on a link.

### MAX_LINK_TTL

```rust
pub const MAX_LINK_TTL: Duration = _;
```

Longest lifetime a link may be minted with.

### MIN_LINK_TTL

```rust
pub const MIN_LINK_TTL: Duration = _;
```

Shortest lifetime a link may be minted with.

### NODE_PUBLIC_KEY_LEN

```rust
pub const NODE_PUBLIC_KEY_LEN: usize = 32;
```

Size of an Ed25519 or X25519 public key.

### NODE_IDENTITY_FILE

```rust
pub const NODE_IDENTITY_FILE: &str = "node-identity.json";
```

Bootstrap file under `$BOOKCLERK_FILES_DIR` that stores the stable node id.

### MAX_INSTANCE_SECRET_REFS

```rust
pub const MAX_INSTANCE_SECRET_REFS: usize = 16;
```

Maximum secret refs in one schema-1 document.

### MAX_INSTANCE_SETTINGS

```rust
pub const MAX_INSTANCE_SETTINGS: usize = 32;
```

Maximum settings in one schema-1 document.

### MAX_INSTANCE_SETTING_BYTES

```rust
pub const MAX_INSTANCE_SETTING_BYTES: usize = 1024;
```

Maximum UTF-8 size of one string setting.

### PLUGIN_INSTANCE_CONFIG_SCHEMA_VERSION

```rust
pub const PLUGIN_INSTANCE_CONFIG_SCHEMA_VERSION: i64 = 1;
```

Domain schema of [`PluginInstanceConfigV1`](#plugininstanceconfigv1).

### ACCOUNT_SEMAPHORE_LIMIT

```rust
pub const ACCOUNT_SEMAPHORE_LIMIT: u32 = 1;
```

Permits an account semaphore grants: one job per store account at a time.

### ROUTED_CLAIM_CANDIDATES

```rust
pub const ROUTED_CLAIM_CANDIDATES: u64 = 16;
```

Pending candidates read per page of a routed claim attempt (the claim
pages on until it claims a job or the ready rows run out).

### EVENT_RETENTION_LEASE

```rust
pub const EVENT_RETENTION_LEASE: &str = "maintenance.events";
```

Lease of the event retention duty (terminal deliveries, empty events,
stale subscriber catalog rows).

### JOB_RETENTION_LEASE

```rust
pub const JOB_RETENTION_LEASE: &str = "maintenance.jobs";
```

Lease of the job retention duty (terminal `jobs` rows).

### KV_EXPIRY_LEASE

```rust
pub const KV_EXPIRY_LEASE: &str = "maintenance.kv";
```

Lease of the plugin KV expiry duty.

### MIGRATION_LEASE

```rust
pub const MIGRATION_LEASE: &str = "schema.migrate";
```

Lease a schema migration runs under (see
[`crate::apply_host_schema_with_batch_opts`](index.md#apply_host_schema_with_batch_opts)).

### SESSION_EXPIRY_LEASE

```rust
pub const SESSION_EXPIRY_LEASE: &str = "maintenance.sessions";
```

Lease of the session duty (expired operator sessions, OIDC RP states,
WebAuthn challenges).

### NODE_CONSUMER_PREFIX

```rust
pub const NODE_CONSUMER_PREFIX: &str = "node:";
```

Catalog `plugin_id` prefix of a node's own consumer (`node:<node_id>`).

### AVAILABILITY_FULL_PPM

```rust
pub const AVAILABILITY_FULL_PPM: i64 = 1_000_000;
```

Full availability in parts per million.

### AVAILABILITY_TAU

```rust
pub const AVAILABILITY_TAU: Duration = _;
```

Time constant of the availability average (about a day).

### DEFAULT_GUEST_CPU_MILLICORES

```rust
pub const DEFAULT_GUEST_CPU_MILLICORES: i64 = 800;
```

CPU a guest is assumed to need when no ceiling is known (the grant
default, 80% of a core).

### DEFAULT_GUEST_MEMORY_BYTES

```rust
pub const DEFAULT_GUEST_MEMORY_BYTES: i64 = _;
```

Memory a guest is assumed to need when no ceiling is known (the grant
default).

### NODE_HEARTBEAT_INTERVAL

```rust
pub const NODE_HEARTBEAT_INTERVAL: Duration = _;
```

How often a daemon heartbeats.

### NODE_LIVE_INTERVALS

```rust
pub const NODE_LIVE_INTERVALS: i64 = 3;
```

A node whose last heartbeat is older than this many intervals is not live.

### MAX_OUTPUT_REPLACEMENT_RULES

```rust
pub const MAX_OUTPUT_REPLACEMENT_RULES: usize = 128;
```

Maximum number of explicit path replacement rules.

### MAX_OUTPUT_TEMPLATE_BYTES

```rust
pub const MAX_OUTPUT_TEMPLATE_BYTES: usize = 1024;
```

Maximum byte length of a naming template.

### OUTPUT_NAMESPACE

```rust
pub const OUTPUT_NAMESPACE: &str = "core.output";
```

Namespace for the migrated shared `[output]` domain.

### OUTPUT_SCHEMA_VERSION

```rust
pub const OUTPUT_SCHEMA_VERSION: i64 = 1;
```

Schema version of [`OutputSettingsV1`](#outputsettingsv1).

### OUTPUT_SETTING_ALIASES

```rust
pub const OUTPUT_SETTING_ALIASES: &[&str] = _;
```

Write-only aliases [`OutputSettingsV1::set`](#outputsettingsv1set) also accepts.

`output.save_chapter_json` takes a boolean or a chapter-JSON mode;
`output.split_files_by_chapter = true` selects the per-chapter MP3 format.

### OUTPUT_SETTING_KEYS

```rust
pub const OUTPUT_SETTING_KEYS: &[&str] = _;
```

Every `output.*` key [`OutputSettingsV1::get`](#outputsettingsv1get) and [`OutputSettingsV1::set`](#outputsettingsv1set)
accept, in display order.

### MAX_JOIN_NAME_BYTES

```rust
pub const MAX_JOIN_NAME_BYTES: usize = 64;
```

Longest branch or library name stored.

### PENDING_JOIN_TTL

```rust
pub const PENDING_JOIN_TTL: Duration = _;
```

How long a sign-in may take before its pending row is refused.

### MAX_NODE_LABELS

```rust
pub const MAX_NODE_LABELS: usize = 32;
```

Most operator labels one node may carry.

### MAX_PLACEMENT_CONSTRAINTS

```rust
pub const MAX_PLACEMENT_CONSTRAINTS: usize = 16;
```

Most constraints one placement may carry.

### MAX_REPLICAS

```rust
pub const MAX_REPLICAS: u32 = 64;
```

Largest replica count an operator may ask for.

### NODE_LABEL

```rust
pub const NODE_LABEL: &str = "node";
```

Implicit label name every node carries with its own id.

### SCHEDULER_LEASE

```rust
pub const SCHEDULER_LEASE: &str = "scheduler";
```

Name of the cluster lease the scheduler runs under.

### SCHEDULER_LEASE_TTL

```rust
pub const SCHEDULER_LEASE_TTL: Duration = _;
```

How long a scheduler lease lasts without renewal.

### TARGET_LABEL

```rust
pub const TARGET_LABEL: &str = "target";
```

Implicit label name every node carries with its artifact target.

### REMOTE_CONSENT_PLATFORM

```rust
pub const REMOTE_CONSENT_PLATFORM: &str = "platform";
```

`approved_by` of an automatic approval: a bundled platform plugin whose
request stays inside the platform envelope.

### MAX_SCAN_RESULT_ROWS

```rust
pub const MAX_SCAN_RESULT_ROWS: usize = 200;
```

Most rows one results batch may carry.

### MAX_WRITE_BACK_BYTES

```rust
pub const MAX_WRITE_BACK_BYTES: usize = _;
```

Largest store token a Host may write back.

### REMOTE_LEASE_OWNER_PREFIX

```rust
pub const REMOTE_LEASE_OWNER_PREFIX: &str = "host:";
```

Prefix of the lease owner of a job a Host runs.

### SECRETS_UNAVAILABLE

```rust
pub const SECRETS_UNAVAILABLE: &str = "secrets_unavailable";
```

Error kind recorded when a job's secrets cannot be unsealed for a Host.

### REQUIREMENTS_GRACE

```rust
pub const REQUIREMENTS_GRACE: Duration = _;
```

How long a pending job's requirements may stay unsettled before the
sweep acts on them. Several scheduler passes (20 s apart) and guest
restarts fit inside it.

### REQUIREMENTS_SWEEP_LEASE

```rust
pub const REQUIREMENTS_SWEEP_LEASE: &str = "jobs:requirements-sweep";
```

Cluster lease the daemon holds while it sweeps.

### REQUIREMENTS_SWEEP_LEASE_TTL

```rust
pub const REQUIREMENTS_SWEEP_LEASE_TTL: Duration = _;
```

Lease length; the holder renews it on every pass.

### UNROUTABLE_ERROR_KIND

```rust
pub const UNROUTABLE_ERROR_KIND: &str = "unroutable";
```

`error_kind` of a job the sweep failed.

### ROUTE_DEFER_PER_RANK

```rust
pub const ROUTE_DEFER_PER_RANK: Duration = _;
```

How long each rank waits behind the one above it.

### MAX_SEMAPHORE_LIMIT

```rust
pub const MAX_SEMAPHORE_LIMIT: u32 = 256;
```

Largest limit a semaphore honours; larger limits are clamped.

### SEMAPHORE_LEASE_PREFIX

```rust
pub const SEMAPHORE_LEASE_PREFIX: &str = "sem:";
```

Prefix of every semaphore slot lease name.

### UPLOAD_NAME_WILDCARD

```rust
pub const UPLOAD_NAME_WILDCARD: char = '\u{1}';
```

In a [`TitleNames::part_patterns`](#titlenames) pattern: one or more characters of a
file name (no `/`), where a part or chapter title goes. Storage keys
hold no control character, so the marker never clashes with one.

### UPLOAD_NUMBER_WILDCARD

```rust
pub const UPLOAD_NUMBER_WILDCARD: char = '\u{2}';
```

In a [`TitleNames::part_patterns`](#titlenames) pattern: one or more ASCII digits,
where a part or chapter number goes.

### UPLOAD_TOKEN_TTL_SECS

```rust
pub const UPLOAD_TOKEN_TTL_SECS: i64 = _;
```

How long a granted upload's token is accepted past its latest
extension (the grant, an accepted file, or a renewal of the job's
lease).

### ARTIFACT_BLOB_SCHEME

```rust
pub const ARTIFACT_BLOB_SCHEME: &str = "blob:sha256:";
```

Artifact URL scheme of an archive stored in `plugin_artifacts`.

### MAX_ARTIFACT_BLOB_BYTES

```rust
pub const MAX_ARTIFACT_BLOB_BYTES: usize = 1_048_576;
```

Largest archive stored in `plugin_artifacts` (base64 stays under
D1's row cap).

### EVENTS_NAMESPACE

```rust
pub const EVENTS_NAMESPACE: &str = "core.events";
```

Namespace for the migrated `[events]` domain.

### NODE_RUNTIME_NAMESPACE

```rust
pub const NODE_RUNTIME_NAMESPACE: &str = "node.runtime";
```

Namespace for node-scoped runtime settings.

### EVENTS_SCHEMA_VERSION

```rust
pub const EVENTS_SCHEMA_VERSION: i64 = 1;
```

Schema version of [`EventsSettingsV1`](#eventssettingsv1).

### NODE_RUNTIME_SCHEMA_VERSION

```rust
pub const NODE_RUNTIME_SCHEMA_VERSION: i64 = 1;
```

Schema version of [`NodeRuntimeSettingsV1`](#noderuntimesettingsv1).

### MAX_EVENTS_RETENTION_DAYS

```rust
pub const MAX_EVENTS_RETENTION_DAYS: u64 = 3650;
```

Upper bound for event retention fields, in days.

### MAX_EVENTS_CONCURRENCY

```rust
pub const MAX_EVENTS_CONCURRENCY: u32 = 32;
```

Upper bound for `[events].concurrency`.

### DEFAULT_CPU_BUDGET_PERCENT

```rust
pub const DEFAULT_CPU_BUDGET_PERCENT: u8 = 80;
```

Default CPU budget: leave a fifth of the cores to the rest of the node.

### DEFAULT_MEMORY_BUDGET_PERCENT

```rust
pub const DEFAULT_MEMORY_BUDGET_PERCENT: u8 = 75;
```

Default memory budget: leave a quarter of the memory to the rest of the node.
