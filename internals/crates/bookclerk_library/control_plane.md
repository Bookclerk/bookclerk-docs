# bookclerk_library::control_plane

Database-backed control plane: node identity and typed configuration.

Bootstrap material (database target, `master.key`, `node-identity.json`) is
read before these rows exist. After enrollment, `core.events` is the
authority for `[events]` and `core.output` for the shared `[output]`
table. Other `config.toml` sections stay transitional.

See `docs/adr/control-plane.md`.

| Item | Kind | Summary |
| --- | --- | --- |
| [`AdminControlOverrides`](#admincontroloverrides) | struct | One Administrator's overrides: a value for some switches, and the library's default for the rest (an absent or `null` field). |
| [`AdminControls`](#admincontrols) | struct | A value for each switch: the library's default, or what applies to one Administrator. |
| [`AssignedAdminControls`](#assignedadmincontrols) | struct | A user's switches: the library's default, their overrides, and what applies to them. |
| [`DefaultSwitchChanges`](#defaultswitchchanges) | struct | Changes to some of the library's default switches: a new value for each named switch; an absent switch keeps its stored value. |
| [`OverrideSwitchChanges`](#overrideswitchchanges) | struct | Changes to some of one Administrator's overrides: `on` or `off` for them alone, or `default` to follow the library's default again; an absent switch keeps what it had. |
| [`BranchTitle`](#branchtitle) | struct | One title a branch shares, as the owner's Settings → Branches lists it. |
| [`BranchTitleCounts`](#branchtitlecounts) | struct | Counts of one branch's shared titles. |
| [`CatalogSessionOpened`](#catalogsessionopened) | struct | An opened catalog session (`begin_catalog`). |
| [`ForwardableBranchKey`](#forwardablebranchkey) | struct | The identity of one title a branch shares with this install, without its metadata or file manifest. |
| [`ForwardableBranchTitle`](#forwardablebranchtitle) | struct | A title one of this install's branches shares, ready to pass on. |
| [`MembersBooksBody`](#membersbooksbody) | struct | `PUT /api/config/branches/members-books` body: a branch, and whether this library acquires all of its members' books. |
| [`MembersBooksChoice`](#membersbookschoice) | struct | A validated `MembersBooksBody`. |
| [`BranchRequestAsk`](#branchrequestask) | struct | A validated `BranchRequestBody`: both ids trimmed and well formed. |
| [`BranchRequestBody`](#branchrequestbody) | struct | `POST /api/branch-requests` body: the work to ask a branch for, and optionally which branch. |
| [`BranchRequestPlan`](#branchrequestplan) | struct | The job a request for `work_id` becomes: which branch it goes to and what it asks for. |
| [`CacheSettingsV1`](#cachesettingsv1) | struct | Typed `core.cache` document, schema version 1. |
| [`CarriedImport`](#carriedimport) | struct | What `import_carried_accounts` did with each account it was handed. |
| [`MemberAccount`](#memberaccount) | struct | One store account a member connected here, as the join consent page lists it. Carries no secret. |
| [`AnswerReport`](#answerreport) | struct | What `record_answers` stored. |
| [`AskReport`](#askreport) | struct | What `ask_branches` did. |
| [`BranchListing`](#branchlisting) | struct | A `branch:<node>` listing and the work its edition belongs to. |
| [`BranchOffer`](#branchoffer) | struct | One branch a work is available through, as Discover and search show it. |
| [`ClerkLink`](#clerklink) | struct | One library this install holds (or held) a library card for. |
| [`LibraryCard`](#librarycard) | struct | A library card in plain form: zeroized on drop, redacted in `Debug`. |
| [`NewClerkLink`](#newclerklink) | struct | What a sign-in returned, to record with `record_clerk_link`. |
| [`LibraryCollection`](#librarycollection) | struct | One collection: a name and the storage key prefixes whose files belong to it. |
| [`LibraryCollections`](#librarycollections) | struct | The library's collections: the `core.collections` document and the body of `PUT /api/library/collections`. |
| [`AccountRef`](#accountref) | struct | One store account, by storefront plugin key and store account id. |
| [`AddConnectionsBody`](#addconnectionsbody) | struct | `POST /api/config/join/connections` body. |
| [`AddConnectionsRequest`](#addconnectionsrequest) | struct | An "Add my connections" request: the library it names (trimmed) and the accounts picked, between 1 and `MAX_ADD_CONNECTIONS` of them. |
| [`DisconnectBody`](#disconnectbody) | struct | `POST /api/config/join/connections/disconnect` body. |
| [`DisconnectRequest`](#disconnectrequest) | struct | A disconnect request: the account and whether to reach every linked library it was shared with. |
| [`CopyDownload`](#copydownload) | struct | What a download request may read (`authorize_copy`). |
| [`CopyListing`](#copylisting) | struct | The store listing a reader names an edition by. |
| [`CopyPlan`](#copyplan) | struct | The copy of one edition a holder may take. |
| [`CopyPlanFile`](#copyplanfile) | struct | One file of a planned copy, on one destination. |
| [`OpenedCopy`](#openedcopy) | struct | A grant just opened by `open_copy`. |
| [`PlaybackPlan`](#playbackplan) | struct | What a person may play of one edition (`plan_playback`). |
| [`PlaybackTrack`](#playbacktrack) | struct | One audio file a player streams for an edition, on one destination. |
| [`CronCursor`](#croncursor) | struct | One `plugin_cron_cursors` row. |
| [`DeploymentObservation`](#deploymentobservation) | struct | One node's observation of a deployment. |
| [`PluginDeployment`](#plugindeployment) | struct | One desired deployment row. |
| [`DestinationPlacementV1`](#destinationplacementv1) | struct | The body of a `core.destination` document. |
| [`DocumentKey`](#documentkey) | struct | Address of one configuration document. |
| [`StoredDocument`](#storeddocument) | struct | One stored document before domain parsing. |
| [`VerifiedLibraryOwner`](#verifiedlibraryowner) | struct | Proof that a user was an active Owner when the write was authorized. |
| [`VerifiedNodeOwner`](#verifiednodeowner) | struct | Proof that a user owned a node when the write was authorized. |
| [`VerifiedPatronAdministrator`](#verifiedpatronadministrator) | struct | Proof that a user was an active Owner or Administrator when a write of the library's patron defaults was authorized. |
| [`DownloadRequestAsk`](#downloadrequestask) | struct | A validated `DownloadRequestBody`: the work id trimmed and well formed. |
| [`DownloadRequestBody`](#downloadrequestbody) | struct | `POST /api/download-requests` body: the work to download. |
| [`DownloadRequestPlan`](#downloadrequestplan) | struct | What a download request for a work fetches. |
| [`BranchEnrollmentSettings`](#branchenrollmentsettings) | struct | The branch-enrollment switch as stored, or its default when no document was ever written. |
| [`BranchEnrollmentSettingsV1`](#branchenrollmentsettingsv1) | struct | Typed `core.branch_enrollment` document, schema version 1. |
| [`EnrolledNode`](#enrollednode) | struct | Result of a successful redeem. |
| [`EnrollmentLink`](#enrollmentlink) | struct | One link as listings show it. The secret and its hash are not included. |
| [`EnrollmentSecret`](#enrollmentsecret) | struct | A link's plaintext secret. Shown once at mint and never stored; `Debug` does not print it. |
| [`EnrollmentToken`](#enrollmenttoken) | struct | A parsed enrollment token. |
| [`MintedLink`](#mintedlink) | struct | A freshly minted link and its secret, which is returned only here. |
| [`NodeCredential`](#nodecredential) | struct | A branch node's stored credential. |
| [`RedeemRequest`](#redeemrequest) | struct | What a host submits to redeem a link. |
| [`NodeIdentity`](#nodeidentity) | struct | Bootstrap identity loaded from `NODE_IDENTITY_FILE`. |
| [`NodeRecord`](#noderecord) | struct | Durable node row plus the latest observation columns. |
| [`ExpandedEntry`](#expandedentry) | struct | One entry an archive member expanded into. |
| [`IngestBatch`](#ingestbatch) | struct | One stored batch. |
| [`IngestBatchId`](#ingestbatchid) | struct | A batch id: 32 lower-case hex digits, safe as one path segment. |
| [`IngestMember`](#ingestmember) | struct | One stored member. |
| [`IntakeRef`](#intakeref) | struct | An intake member's place in its storage instance. |
| [`ManifestFile`](#manifestfile) | struct | One file of a `POST /api/uploads` manifest, as sent. |
| [`MemberSpec`](#memberspec) | struct | One file a manifest names. |
| [`OutcomeUpdate`](#outcomeupdate) | struct | One member's outcome, as the job decided it. |
| [`ReviewItem`](#reviewitem) | struct | One stored book an Administrator should review: a member stored under a new local edition because nothing strong named one. |
| [`Sha256Hex`](#sha256hex) | struct | A lower-case hex SHA-256 digest a manifest declared for a file. |
| [`UploadManifest`](#uploadmanifest) | struct | `POST /api/uploads` body: the files of one batch. |
| [`UploadQuota`](#uploadquota) | struct | The upload limits a batch is admitted under (`uploads.*` settings). |
| [`IngressSettings`](#ingresssettings) | struct | The `core.ingress` document. |
| [`InstanceBindingGrant`](#instancebindinggrant) | struct | Whether the plugin-instance config grant includes `config` or `secrets`. |
| [`InstanceConfigDocument`](#instanceconfigdocument) | struct | A parsed instance document plus its revision. |
| [`InstanceSecretRefV1`](#instancesecretrefv1) | struct | A name in `encrypted_secrets`. The ciphertext is not part of the document. |
| [`PluginInstanceConfigV1`](#plugininstanceconfigv1) | struct | Schema-1 plugin instance configuration. |
| [`PrunedInstanceConfig`](#prunedinstanceconfig) | struct | One instance document rewritten to fit the pinned schema. |
| [`ResolvedInstanceBindings`](#resolvedinstancebindings) | struct | `CONFIG` and `SECRETS` payloads for one instance. |
| [`PluginInstance`](#plugininstance) | struct | One `plugin_instances` row. |
| [`PluginInstanceId`](#plugininstanceid) | struct | Stable id of one plugin instance. |
| [`JobClaimFilter`](#jobclaimfilter) | struct | What a Host is willing to claim: the job kinds it runs and the plugin keys its operator consented to. Both must admit a job. |
| [`JobClaimLimits`](#jobclaimlimits) | struct | Library-wide permit counts a routed claim enforces. |
| [`JobClaimRequest`](#jobclaimrequest) | struct | One routed claim attempt by one worker. |
| [`RoutedJobClaim`](#routedjobclaim) | struct | A claimed job and the semaphore permits it holds. |
| [`KeptShare`](#keptshare) | struct | One account this install shares with a library it joined, as the keeper sees it. |
| [`SignIn`](#signin) | struct | An account's current sign-in on its keeper (`sign_in_revision`). |
| [`LeaseFence`](#leasefence) | struct | Proof, checked against the database, that this holder still holds a lease. Call `LeaseFence::check` before each destructive step. |
| [`LibraryLease`](#librarylease) | struct | A held library lease. |
| [`SingletonDuty`](#singletonduty) | struct | One periodic duty that exactly one live node runs per interval. |
| [`CapacityBody`](#capacitybody) | struct | `POST /api/config/join/capacity` body: one library and the job cap to set for it. |
| [`JobCapRequest`](#jobcaprequest) | struct | A validated job cap change: the library it names (trimmed) and its new cap, `None` for an equal share. |
| [`LibraryLink`](#librarylink) | struct | One library this install enrolled into. |
| [`LibraryLinkKeys`](#librarylinkkeys) | struct | The node keys of one library link: the Ed25519 signing seed and the X25519 box secret. Zeroized on drop; `Debug` shows only the public keys. |
| [`LibraryLinkSettings`](#librarylinksettings) | struct | Every setting Settings → Library changes on one link at once; see `set_library_link_settings`. |
| [`LinkJobCap`](#linkjobcap) | struct | The most jobs a branch runs for one library at once, as its operator set it: 0 (runs nothing for that library) to `LinkJobCap::MAX`. |
| [`NewLibraryLink`](#newlibrarylink) | struct | What enrollment returned, to record with `record_library_link`. |
| [`SharingBody`](#sharingbody) | struct | `POST /api/config/join/sharing` body: one library and the settings to change for it. |
| [`SharingRequest`](#sharingrequest) | struct | A sharing settings change: the library it names (trimmed) and the settings to write, each `None` setting left as stored. |
| [`LinkImport`](#linkimport) | struct | One title the library committed, as the branch kept it. |
| [`OwnerShareFilter`](#ownersharefilter) | struct | Which of this install's acquired books the catalog sync sends the library `library_id`: the branch owner's own books, those held by a store account connected here to the branch's person (`super::branch_person`); the books that library asked for through an interlibrary request and this branch still lists to it (`still_listed_requests`); and, while that library acquires all of this branch's members' books, the books of every store account this branch lists to it (`listed_accounts`). Never one imported from that library (`books_imported_from_link`). With no single person, the owner's books are not among them. |
| [`TakenCopy`](#takencopy) | struct | A book of this install whose files came from a library it joined, by its store identity here. |
| [`LiveRevision`](#liverevision) | struct | One topic's revision: a counter that every write to the topic's table moves forward by at least one. |
| [`AssignedControls`](#assignedcontrols) | struct | A user's controls: the library's default, their overrides, and what applies to them. |
| [`ControlOverrides`](#controloverrides) | struct | One patron's overrides: a value for some controls; the rest (an absent or `null` field) follow the library's default for a Member and are allowed for an Administrator. |
| [`MemberControls`](#membercontrols) | struct | A value for each control: the library's default, or what applies to one user. |
| [`PutUserControls`](#putusercontrols) | struct | `PUT /api/users/{id}/controls` body. |
| [`CacheUsage`](#cacheusage) | struct | What a node's image caches hold, as its daemon reports them. |
| [`GuestReservation`](#guestreservation) | struct | CPU and memory one guest may use at most: its grant's ceilings after the node's `[plugins.jail]` limits. |
| [`NodeHeartbeat`](#nodeheartbeat) | struct | One heartbeat's measurements, before budget and accounting are applied. |
| [`NodeSampler`](#nodesampler) | struct | Samples capacity and measured use for heartbeats. Keeps the previous CPU counters so each sample reports utilization since the last one. |
| [`NodeStatus`](#nodestatus) | struct | A node's latest heartbeat with budget, accounting, and availability. |
| [`OutputSettingsV1`](#outputsettingsv1) | struct | Typed `core.output` document, schema version 1. |
| [`AssignedScope`](#assignedscope) | struct | A Member's scope and where it came from. |
| [`PatronScope`](#patronscope) | struct | Which part of the library a patron sees. |
| [`PutUserScope`](#putuserscope) | struct | `PUT /api/users/{id}/scope` body. |
| [`ScopeRule`](#scoperule) | struct | One rule of a scope: a field, one value of it, and its effect. |
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
| [`RemotePluginConsent`](#remotepluginconsent) | struct | One owner plugin of one library, as this install sees it. |
| [`HandedPassEnd`](#handedpassend) | struct | How a Host's pass of one account ended. |
| [`ScanSighting`](#scansighting) | struct | A product a Host's pass listed that has no library row to write (its record maps onto no flat row, an ebook for instance), so the owner only marks the account's holding of it seen by the pass. |
| [`JobSecret`](#jobsecret) | struct | One unsealed secret handed with a job. The value is zeroized on drop and never shown by `Debug`. |
| [`JobSecretRef`](#jobsecretref) | struct | Names one store-account secret a job is handed. Carries no secret material. |
| [`RemoteClaimSettings`](#remoteclaimsettings) | struct | What a Host's claim settles on: the routing and lease settings of the shared node serving it. |
| [`RemoteJobClaim`](#remotejobclaim) | struct | A job claimed for a Host, with the secrets to seal for it. |
| [`RequirementsSweepReport`](#requirementssweepreport) | struct | What one sweep did. |
| [`RouteRank`](#routerank) | struct | This node's position among the live nodes running one instance. |
| [`PinnedPluginSchema`](#pinnedpluginschema) | struct | One `plugin_schemas` row. |
| [`SchemaPin`](#schemapin) | struct | Inputs for `pin_plugin_schema`. |
| [`SchemaPinOutcome`](#schemapinoutcome) | struct | Outcome of `pin_plugin_schema`: the row that is now pinned and how it relates to the local manifest. |
| [`LibrarySecret`](#librarysecret) | struct | Library id and fingerprint of the secret root aligned for this process. |
| [`SemaphorePermit`](#semaphorepermit) | struct | One held permit: a slot lease of a semaphore. |
| [`SemaphoreRequest`](#semaphorerequest) | struct | One semaphore a caller wants a permit of. |
| [`LibraryContents`](#librarycontents) | struct | What library data an install holds of its own. |
| [`TemplateStrip`](#templatestrip) | struct | What `strip_install_state` removed. |
| [`TitleNames`](#titlenames) | struct | The names a title's acquire writes on one destination, as the owner plans them, all without their extension. |
| [`UploadDestination`](#uploaddestination) | struct | One output destination of the receiving shared node an upload is granted for, with where the title may be written there. |
| [`UploadFinish`](#uploadfinish) | struct | How the job's closing upload completes it. |
| [`UploadGrantRequest`](#uploadgrantrequest) | struct | What a Host asks to upload for one title. |
| [`UploadSession`](#uploadsession) | struct | One open upload, as the upload endpoint sees it. |
| [`PluginArtifact`](#pluginartifact) | struct | One downloadable artifact of a version. |
| [`PluginInstall`](#plugininstall) | struct | The active version pointer of one plugin key. |
| [`PluginVersion`](#pluginversion) | struct | One installed version of a plugin. |
| [`WindowEntry`](#windowentry) | struct | One work in the window's merged view. |
| [`WindowEntryEdition`](#windowentryedition) | struct | One edition of a `WindowEntry`, and the library that holds it. |
| [`EventsSettingsV1`](#eventssettingsv1) | struct | Typed `[events]` document, schema version 1. |
| [`NodeRuntimeSettingsV1`](#noderuntimesettingsv1) | struct | Typed node-scoped runtime document, schema version 1. |
| [`ConfigurationDocument`](#configurationdocument) | struct | A parsed domain document plus its revision metadata. |
| [`ControlPlaneSession`](#controlplanesession) | struct | Identity, secret root, and the authoritative events document for this process. |
| [`LinkRevoked`](#linkrevoked) | enum | What `revoke_account_link` did. |
| [`SignInLink`](#signinlink) | enum | What `link_signed_in_account` did. |
| [`AdminControl`](#admincontrol) | enum | One power an Administrator may be allowed or refused. |
| [`AdminSwitchChoice`](#adminswitchchoice) | enum | What a change does to one switch of one Administrator. |
| [`CatalogAccept`](#catalogaccept) | enum | Outcome of `accept_catalog_batch`. |
| [`CatalogBegin`](#catalogbegin) | enum | Outcome of `begin_catalog`. |
| [`RequestFinished`](#requestfinished) | enum | What `finish_branch_request` did. |
| [`Withdrawn`](#withdrawn) | enum | What `withdraw_person_account` did. |
| [`AnswersRecorded`](#answersrecorded) | enum | Outcome of `record_answers`. |
| [`ClerkLinkState`](#clerklinkstate) | enum | Whether this install still holds the card of a clerk link. |
| [`CopyHolder`](#copyholder) | enum | Who asks for a copy. |
| [`CopyPlanned`](#copyplanned) | enum | The answer of `plan_copy`. |
| [`CronFireClaim`](#cronfireclaim) | enum | What `claim_cron_fire` did. |
| [`DeploymentReplace`](#deploymentreplace) | enum | Outcome of `replace_plugin_deployment`. |
| [`DeploymentStatus`](#deploymentstatus) | enum | Observed deployment status. |
| [`DesiredState`](#desiredstate) | enum | What a deployment row asks its node to do, and who decided it. |
| [`ConfigActor`](#configactor) | enum | Who is allowed to write a configuration document. |
| [`ReplaceOutcome`](#replaceoutcome) | enum | Result of a compare-and-swap replacement. |
| [`EnrollmentActor`](#enrollmentactor) | enum | Who is acting on enrollment links and node credentials. |
| [`BranchRelationship`](#branchrelationship) | enum | How a node relates to the library, as people are told: a shared branch serves the library from its database, and a branch with its own database is a remote branch when it reports a public address (it is a core branch other installs can reach) and a local branch when it does not. |
| [`NodeKind`](#nodekind) | enum | How a node joined the library. |
| [`CreateRefusal`](#createrefusal) | enum | Why a batch was not created. |
| [`IngestSource`](#ingestsource) | enum | Where a batch's bytes come from. |
| [`IngestState`](#ingeststate) | enum | Where a batch is in its life. |
| [`ManifestError`](#manifesterror) | enum | Why a manifest was refused before any batch was made. |
| [`MemberOutcome`](#memberoutcome) | enum | What became of one member. |
| [`IngressExposure`](#ingressexposure) | enum | What public traffic from the ingress may reach. |
| [`OriginReport`](#originreport) | enum | What `record_started` did with a reported origin. |
| [`InstanceConfigReplace`](#instanceconfigreplace) | enum | Outcome of `replace_instance_config`. |
| [`SettingCell`](#settingcell) | enum | One cell of a table setting row: the scalar kinds only. |
| [`SettingValue`](#settingvalue) | enum | One setting value: a scalar, or the rows of a table setting. |
| [`KeeperUpdate`](#keeperupdate) | enum | What `apply_keeper_update` did. |
| [`LinkImportOutcome`](#linkimportoutcome) | enum | What `import_link_title` did. |
| [`LiveTopic`](#livetopic) | enum | A topic of the live socket. |
| [`MemberControl`](#membercontrol) | enum | One thing a patron may be allowed or refused. |
| [`NodeMessage`](#nodemessage) | enum | One message on a `node.<node_id>` queue. The body is the JSON of this enum (`{"type": "deployment.changed", ...}`). |
| [`OutputReplace`](#outputreplace) | enum | Outcome of `replace_output`. |
| [`ExplicitContent`](#explicitcontent) | enum | Whether a patron sees books marked as explicit (adult) content. |
| [`RuleEffect`](#ruleeffect) | enum | Whether a `ScopeRule` lets matching books in or keeps them out. |
| [`ScopeField`](#scopefield) | enum | The book field a `ScopeRule` matches one value of. |
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
| [`account_slot`](#account_slot) | function | The `bookclerk_slots` key that serializes writes to one store account's sign-in and link. |
| [`link_signed_in_account`](#link_signed_in_account) | function | Finishes an allowed store login: marks the account active with scans on and links it to `identity_id`, in one atomic batch under the account's write lock, only while the account's sign-in is stored and no other identity holds it. |
| [`revoke_account_link`](#revoke_account_link) | function | Revokes the account `identity_id` holds: in one atomic batch under the account's write lock, marks it `revoked` with scans off, deletes its integration secrets, forgets which branch kept its login for this library (its `node:` shares and their sign-in revisions), and removes the link. Every statement applies only while the identity's link exists, and the link goes last, so the batch refuses as a whole once the link is gone. |
| [`admin_control_overrides`](#admin_control_overrides) | function | The overrides stored for `user_id` (none when there is no row). |
| [`admin_may`](#admin_may) | function | Whether `user_id` may use `control` here. An unknown user may not. |
| [`administrator_switch_allows`](#administrator_switch_allows) | function | Whether an Administrator's switch lets `user_id` do `control`, for an action other roles are allowed or refused by their own rules: true for any role but Administrator (and for an active Administrator whose switch is on), false for an unknown user. |
| [`assigned_admin_controls`](#assigned_admin_controls) | function | The switches of `user`: the library's default, their overrides, and what applies to them. |
| [`change_admin_control_overrides`](#change_admin_control_overrides) | function | Changes only the named overrides of `user_id`: each attempt applies `changes` to the row it read and writes it back only if the row is still what it read, so a concurrent edit of another switch is kept rather than overwritten. |
| [`change_default_admin_controls`](#change_default_admin_controls) | function | Changes only the named default switches under compare-and-swap: each attempt applies `changes` to the stored default it read, so a concurrent edit of another switch is kept rather than overwritten. |
| [`default_admin_controls`](#default_admin_controls) | function | The library's default Administrator switches: the stored document, or every switch on. |
| [`effective_admin_controls`](#effective_admin_controls) | function | What applies to `user` given the library's default and their overrides: everything for an active Owner, the overrides over the default for an active Administrator, and nothing for anyone else (a user who is not active or a Member). |
| [`load_default_admin_controls`](#load_default_admin_controls) | function | Loads the library's default Administrator switches document. |
| [`set_admin_control_overrides`](#set_admin_control_overrides) | function | Replaces the overrides of `user_id`; with none, the Administrator follows the library's default for every switch again. |
| [`set_default_admin_controls`](#set_default_admin_controls) | function | Replaces the library's default Administrator switches under compare-and-swap, creating the document when absent and retrying a few times on a concurrent edit. |
| [`load_artifact_blob`](#load_artifact_blob) | function | Reads an archive stored by `put_artifact_blob`. |
| [`put_artifact_blob`](#put_artifact_blob) | function | Stores a small archive and returns its `blob:sha256:<hex>` URL. |
| [`accept_catalog_batch`](#accept_catalog_batch) | function | Records one batch of `node_id`'s shared titles under `fence`, then the cursor after it. Every title is checked before any is written; a title this library already holds for the branch's owner is skipped. Each other title is resolved into the library model and held by the branch (`holding_added` when new), and the holding's share state is written in one transaction with the cursor, each statement gated on the fence still being live, the branch still sharing and its credential live. A holding is stamped with the pass under way, and with `done` (the last batch of a pass) while a pass from the start is under way, the branch's holdings the pass did not name are removed (`withdrawn`) and the pass ends; `done` on a resumed incremental pass changes nothing more. |
| [`begin_catalog`](#begin_catalog) | function | Opens (or reopens) `node_id`'s catalog sync with the settings the branch reports: records them on its node row, bumps its catalog fence (frames under an older fence are lost from now on), and answers where to resume. A change of either setting restarts the pass from the start, so titles the new setting excludes are withdrawn when the pass ends; so does a session `FULL_CATALOG_PASS_EVERY` after the last full pass. With sharing off, every title the branch shared is removed (`stopped_sharing`). |
| [`branch_provenance`](#branch_provenance) | function | The provenance text of titles shared by `node_id`: the holder key of its node holdings. |
| [`branch_title_counts`](#branch_title_counts) | function | `BranchTitleCounts` of `node_id`. |
| [`catalog_fence_live`](#catalog_fence_live) | function | True while `fence` is `node_id`'s live catalog fence and it may share. |
| [`detach_branch_titles`](#detach_branch_titles) | function | Removes every holding of `node_id` (it stopped sharing, left or was revoked; `reason` is logged), each with `holding_removed`; answers how many went. Files already delivered stay with their editions. |
| [`forwardable_branch_title`](#forwardable_branch_title) | function | On a branch: the title shared through node holding `id` when it may still be passed on (still held, files of its current manifest delivered). |
| [`forwardable_branch_title_keys_page`](#forwardable_branch_title_keys_page) | function | On a branch: one keyset page of the titles its own branches share with it, identity columns only, by holding id. |
| [`grant_catalog_upload`](#grant_catalog_upload) | function | Opens an upload of one shared title's files, bound to `node_id`'s catalog `fence` and received by `receiving_node_id`. |
| [`list_branch_titles`](#list_branch_titles) | function | Titles branches share, of one branch or all, by holding id, at most `limit`. |
| [`listing_from_catalog_title`](#listing_from_catalog_title) | function | The listing a shared title describes, filed under `plugin_key`: its metadata, ASIN scoped to its marketplace, ISBN, and media kind from its content kind. |
| [`pending_deliveries`](#pending_deliveries) | function | Up to `limit` of `node_id`'s shared titles whose files have not arrived (or changed since), leaving out titles whose edition already has audio here from another writer. |
| [`remove_detached_branch_titles`](#remove_detached_branch_titles) | function | Forgets the files branches left behind (entries "from a branch that left" and the like), of one branch or of every one: the `edition_files` records its catalog uploads wrote for editions nothing here holds any more. Answers how many editions' entries went. The files themselves stay in this library's outputs. |
| [`parse_members_books_request`](#parse_members_books_request) | function | Parses and validates a `PUT /api/config/branches/members-books` body from its raw bytes, sans IO, the way the route reads it: JSON into `MembersBooksBody`, then `MembersBooksBody::validate`. |
| [`set_branch_members_books`](#set_branch_members_books) | function | Turns "Acquire all of this branch's members' books" on or off for branch `node_id`. |
| [`books_requested_by_link`](#books_requested_by_link) | function | On a branch: every book the library `library_id` asked for, by store identity, with when it last asked (an unreadable stamp reads as now, so the book is sent rather than skipped). Read in keyset pages no larger than the database plugin's `maxResultRows`. |
| [`claim_branch_requests`](#claim_branch_requests) | function | Claims the requests waiting for branch `node_id`, oldest first. |
| [`finish_branch_request`](#finish_branch_request) | function | Records what branch `node_id` did with the request `job_id` it claimed at lease `generation`. |
| [`notify_requests_queued`](#notify_requests_queued) | function | Wakes branch `node_id` after a request for it was admitted. Best effort: the branch also claims on every connection and on a timer. |
| [`parse_branch_request`](#parse_branch_request) | function | Parses and validates a `POST /api/branch-requests` body from its raw bytes, sans IO, the way the route reads it: JSON into `BranchRequestBody`, then `BranchRequestBody::validate`. |
| [`plan_branch_request`](#plan_branch_request) | function | Plans an interlibrary request for `work_id`. |
| [`record_link_request`](#record_link_request) | function | On a branch: records that the library `library_id` asked for `title` (by its holding account and store listing), so the catalog sync sends it there. Asking again moves the request to now. |
| [`is_cache_setting_key`](#is_cache_setting_key) | function | True when `key` is one of `CACHE_SETTING_KEYS`. |
| [`load_cache_settings`](#load_cache_settings) | function | Loads the library cache document. |
| [`update_cache_settings`](#update_cache_settings) | function | Applies `edit` to the library cache document under compare-and-swap, creating it with the defaults when absent and retrying a few times on a concurrent edit. Nodes pick the change up at their next heartbeat. |
| [`account_share_peers`](#account_share_peers) | function | Every peer this install shared the store account with, in either direction (`account_shares`): `node_peer` and `link_peer` values, each once, sorted. |
| [`accounts_to_carry`](#accounts_to_carry) | function | The store accounts `user_id` connected here that may travel to a branch (`member_accounts` whose plugin `shareable` accepts), each with its secrets unsealed. At most `MAX_BUNDLE_ACCOUNTS`; an account without a secret is left out (there is nothing to sign in with). |
| [`accounts_to_carry_where`](#accounts_to_carry_where) | function | `accounts_to_carry` limited to the accounts `keep` accepts, by `(plugin_key, account_id)`, applied before the bundle's count and size limits so a picked account is never crowded out by unpicked ones. |
| [`branch_person`](#branch_person) | function | The person on this install that store accounts brought from a library belong to: `preferred` (the Owner who elevated into the operator session that ran the join) while that user is still an active Owner, otherwise the only active Owner. `None` when neither settles it, for example on a new install with no Owner yet; the accounts are then not kept. |
| [`identity_may_connect_accounts`](#identity_may_connect_accounts) | function | Whether the portal identity `identity_id` may connect store accounts here: the rule of the user it belongs to (`member_may_connect_accounts`). An identity that belongs to no first-party user (an integration's external user) has no member rule and may connect, as before; an unknown identity may not. |
| [`import_carried_accounts`](#import_carried_accounts) | function | Imports the accounts a library sent with its person, for `user_id`, recording that they came from the library `library_id`. |
| [`import_offered_accounts`](#import_offered_accounts) | function | Imports the accounts a branch offered up its link ("Add my connections to *library*") for `user_id`, the branch's owner, recording that they came from the node `node_id`. |
| [`link_peer`](#link_peer) | function | The `account_shares.peer` of a library this install received accounts from. |
| [`member_accounts`](#member_accounts) | function | The store accounts `user_id` has connected here: linked to any of the user's portal identities and still connected (`connection_status` is `active`), sorted by plugin key and account id. |
| [`member_may_connect_accounts`](#member_may_connect_accounts) | function | Whether `user_id` may connect store accounts on this instance: an active user whose member controls allow it (`super::MemberControl::LinkAccounts`). |
| [`node_peer`](#node_peer) | function | The `account_shares.peer` of a branch node this library sent accounts to. |
| [`record_accounts_sent`](#record_accounts_sent) | function | Records that `accounts` were sent to the branch `node_id` (`account_shares`, direction `SHARE_SENT`), in atomic batches of at most 13 accounts; a repeat refreshes the time. |
| [`record_accounts_sent_to`](#record_accounts_sent_to) | function | Records that `accounts` were sent to `peer` (`node_peer` for a branch, `link_peer` for a library this install offered them to), as `record_accounts_sent` does. |
| [`revoke_unlinked_account`](#revoke_unlinked_account) | function | Takes back a store account a refused login stored: deletes its secrets and marks it `revoked` with scans off, in one atomic batch whose every statement applies only while no link holds the account. |
| [`withdraw_person_account`](#withdraw_person_account) | function | Disconnects a store account "everywhere" on this library for the person `user_id`: removes every link from their identities to the account and, in the same atomic batch, deletes its secrets and marks it `revoked` with scans off when no other link holds it. A revoked account then loses its holdings (each with a `holding_removed` event) and its scan checkpoints, as `LibraryStore::revoke_credentials` does. |
| [`answer_questions`](#answer_questions) | function | Answers questions a library sent this branch from this branch's own library. |
| [`ask_branches`](#ask_branches) | function | Asks every trusted branch about `work_ids`. |
| [`branch_listing_key`](#branch_listing_key) | function | The plugin key of listings answered by branch `node_id`. |
| [`branch_listings_for_works`](#branch_listings_for_works) | function | The current branch listings of `work_ids`: those a branch this library still trusts named within `BRANCH_LISTING_TTL`. |
| [`branch_offers_for_works`](#branch_offers_for_works) | function | The branches each of `work_ids` is currently available through. |
| [`catalog_listing_always`](#catalog_listing_always) | function | True when `user_id` is an active Owner of this branch, whose books are always listed whatever their opt-out says. |
| [`catalog_listing_opted_out`](#catalog_listing_opted_out) | function | True when member `user_id` keeps their books out of this branch's listings. |
| [`pending_questions`](#pending_questions) | function | The questions waiting for branch `node_id`, marked sent. |
| [`record_answers`](#record_answers) | function | Records branch `node_id`'s answers to the questions it was sent. |
| [`set_catalog_listing_opt_out`](#set_catalog_listing_opt_out) | function | Records whether member `user_id`'s books are kept out of what this branch lists to the libraries it joined. |
| [`title_for_request`](#title_for_request) | function | The title of this library that fulfils an interlibrary request: a title this branch would list for the request's work (`answer_questions`'s rules: the owner's always, members' only with `list_members_books` on and when they did not opt out) whose store, marketplace and product id are the request's. The first such holding wins. `None` when this branch no longer lists that edition to the library. |
| [`trusted_branches`](#trusted_branches) | function | The branches this library trusts, by node id. |
| [`works_of_interest`](#works_of_interest) | function | The works this library wants its branches asked about: those of open wishes (most recently changed first), then those of Discover candidates (most recently fetched first), at most `limit`. |
| [`works_of_listings`](#works_of_listings) | function | The work of each store listing in `listings`, by `(plugin key, product id)`, for listings the shared pool already holds. |
| [`forget_clerk_link`](#forget_clerk_link) | function | Deletes a signed-out link's row, so it leaves the list. Answers false when there was no such link. |
| [`list_clerk_links`](#list_clerk_links) | function | Every clerk link, oldest sign-in first. |
| [`load_clerk_card`](#load_clerk_card) | function | Unseals the card of an active `link`. |
| [`load_clerk_link`](#load_clerk_link) | function | One clerk link. |
| [`record_clerk_link`](#record_clerk_link) | function | Seals `card` and records the link as active, replacing an earlier sign-in to the same library. |
| [`sign_out_clerk_link`](#sign_out_clerk_link) | function | Marks a link signed out, deletes its card and drops the books read through it. Answers the link as it now stands, or `None` when there is no such link. |
| [`collections_revision`](#collections_revision) | function | The collections document's revision, 0 when it was never written. The page index rebuild compares it before and after its walk. |
| [`library_collections`](#library_collections) | function | The library's collections; none when the document was never written. |
| [`load_collections`](#load_collections) | function | Reads the stored collections document, if one was ever written. |
| [`parse_collections_request`](#parse_collections_request) | function | Parses and validates a `PUT /api/library/collections` body from its raw bytes, sans IO, the way the route reads it: JSON into `LibraryCollections`, then `LibraryCollections::validate`. |
| [`set_library_collections`](#set_library_collections) | function | Replaces the library's collections (compare-and-swap, audited) and, when they changed, drops the page index's finished build so every row is rebuilt with the new prefixes. |
| [`parse_add_connections_request`](#parse_add_connections_request) | function | Parses and validates a `POST /api/config/join/connections` body from its raw bytes, sans IO, the way the route reads it: JSON into `AddConnectionsBody`, then `AddConnectionsBody::validate`. |
| [`parse_disconnect_request`](#parse_disconnect_request) | function | Parses a `POST /api/config/join/connections/disconnect` body from its raw bytes, sans IO, the way the route reads it: JSON into `DisconnectBody`, then `DisconnectBody::into_request`. |
| [`authorize_copy`](#authorize_copy) | function | The files `token` may download for copy `copy_id`, or `None`. |
| [`authorize_playback`](#authorize_playback) | function | The tracks a play session may still stream, or `None`. |
| [`open_copy`](#open_copy) | function | Records that `holder` may download `plan`'s files from `destination` on node `serving_node_id`, and mints the token. |
| [`plan_copy`](#plan_copy) | function | Decides whether `holder` may take this library's copy of the edition `listing` names, and lists the copy's files. |
| [`plan_playback`](#plan_playback) | function | Decides whether `person` may play this library's copy of `edition_id`, and lists its tracks: the one check behind the playback manifest, a new play session and every stream request. |
| [`claim_cron_fire`](#claim_cron_fire) | function | Claims the fire at `fire_ms` for `(plugin_instance_id, cron)` and admits its `plugin_scheduled` job in the same batch. |
| [`ensure_cron_cursor`](#ensure_cron_cursor) | function | Creates the cursor at `initial_last_ms` unless one exists, and returns the stored cursor. |
| [`load_cron_cursor`](#load_cron_cursor) | function | Reads one cursor. |
| [`bounded_observation_detail`](#bounded_observation_detail) | function | Truncates detail to 512 bytes and drops ASCII control characters. |
| [`delete_plugin_deployment`](#delete_plugin_deployment) | function | Deletes an absent deployment and its observations. |
| [`ensure_plugin_deployment`](#ensure_plugin_deployment) | function | Inserts `(instance, node, desired)` when that pair has no row. |
| [`instance_has_deployment`](#instance_has_deployment) | function | True when any deployment row names `instance_id`. |
| [`list_all_deployments`](#list_all_deployments) | function | Every deployment row in the library. |
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
| [`destination_filter`](#destination_filter) | function | One destination's filter: the stored one, or a filter that takes every book. |
| [`destination_filters`](#destination_filters) | function | The stored filters of the given destinations, keyed by plugin instance id. |
| [`destination_key`](#destination_key) | function | The `core.destination` address for one plugin instance. |
| [`load_destination_placement`](#load_destination_placement) | function | Loads one destination's placement document. |
| [`set_destination_filter`](#set_destination_filter) | function | Replaces one destination's filter under compare-and-swap, creating the document when absent and retrying a few times on a concurrent edit. |
| [`audit_count`](#audit_count) | function | Counts audit rows for one namespace. |
| [`change_count`](#change_count) | function | Counts committed change notices for one namespace. |
| [`parse_download_request`](#parse_download_request) | function | Parses and validates a `POST /api/download-requests` body from its raw bytes, sans IO, the way the route reads it: JSON into `DownloadRequestBody`, then `DownloadRequestBody::validate`. |
| [`plan_download_request`](#plan_download_request) | function | Plans a download request for `work_id`. |
| [`authorized_enrollment_link`](#authorized_enrollment_link) | function | The link a branch or shared branch sign-in redeems through: its id and secret are derived from the authorization code and the PKCE verifier, so a retry of the same exchange (same code, verifier and keys) finds the same link and answers from its receipt, and a wrong verifier can never redeem it. |
| [`check_redeem_request`](#check_redeem_request) | function | Checks what a redeem would refuse as malformed before anything is written: its keys (canonical Curve25519 points) and `software_version`. A sign-in's redeem runs this before it spends its code, so a malformed request leaves the code for a corrected retry. |
| [`enrollment_link_brings_accounts`](#enrollment_link_brings_accounts) | function | Whether the user who approved the sign-in behind `link_id` chose to bring their store connections to the branch (`record_authorized_enrollment_link`). `false` for an unknown link and for every minted invite. |
| [`format_enrollment_token`](#format_enrollment_token) | function | Formats `bcn1.<link_id>.<secret>[.<spki_pin>]`. |
| [`list_enrollment_links`](#list_enrollment_links) | function | Links `actor` may see, newest first: every link for the operator or an Owner, the caller's own links for anyone else. |
| [`load_branch_enrollment`](#load_branch_enrollment) | function | Reads the library's branch-enrollment switch. A missing document means the default: enabled. |
| [`load_node_credential`](#load_node_credential) | function | Reads a branch node's credential. |
| [`mint_enrollment_link`](#mint_enrollment_link) | function | Mints a link that enrolls one branch node for `for_user_id`. |
| [`parse_enrollment_token`](#parse_enrollment_token) | function | Parses a token, or a whole link whose fragment is the token. |
| [`record_authorized_enrollment_link`](#record_authorized_enrollment_link) | function | Records the link an approved branch sign-in redeems through (`authorized_enrollment_link`), for `for_user_id`, the user who signed in and consented. Called only after the authorization code's client, redirect and PKCE verifier were checked, before the code is consumed, so a failed write leaves the code usable; the redeem itself is `redeem_enrollment_link`, the same canonical batch a minted link takes. |
| [`record_node_connected`](#record_node_connected) | function | Stamps `last_connected_at` after a branch node authenticates. Returns false (and stamps nothing) when the credential is missing or revoked, so a revoke that lands mid-handshake still refuses the connection. |
| [`record_node_left`](#record_node_left) | function | A host announced it is leaving (`node.leaving`): revokes its credential at once and drops its live status, so it stops counting as a node without waiting for missed heartbeats, and tells every node so the connection is closed wherever it is held. Answers false when the node has no unrevoked credential; for one already revoked, the cleanup is still finished, so a retried leave completes one that failed part way. |
| [`redeem_enrollment_link`](#redeem_enrollment_link) | function | Redeems a link: one canonical batch marks it redeemed, inserts a host `nodes` row owned by the link's user, inserts its `node_credentials`, and records a receipt. |
| [`redeem_enrollment_link_as`](#redeem_enrollment_link_as) | function | `redeem_enrollment_link` for a node of `kind`: the same canonical batch, recording `kind` on the `nodes` row. |
| [`revoke_enrollment_link`](#revoke_enrollment_link) | function | Revokes an unredeemed link. Returns the link as stored afterwards. |
| [`revoke_node_credential`](#revoke_node_credential) | function | Revokes a branch node's credential and tells every node, so the node holding that host's connection can close it. |
| [`set_branch_enrollment_enabled`](#set_branch_enrollment_enabled) | function | Turns branch enrollment on or off for the library. |
| [`verify_library_owner`](#verify_library_owner) | function | Returns a `ConfigActor::LibraryOwner` when `user_id` is an active Owner. |
| [`current_process_incarnation`](#current_process_incarnation) | function | This process's incarnation. Stable until the process exits. |
| [`encode_node_identity_file`](#encode_node_identity_file) | function | The `NODE_IDENTITY_FILE` contents `install_node_identity` writes, for a caller that stages the file itself (a shared branch applying its bootstrap as one recoverable step) with mode `0600`. |
| [`install_node_identity`](#install_node_identity) | function | Writes the identity file for a node id the library already recorded, bound to `library_id`, replacing any file there atomically. A shared branch joining a library takes the node id the library minted for it at redeem, so its first heartbeat lands on that `nodes` row. |
| [`list_node_ids`](#list_node_ids) | function | Lists node ids, sorted. |
| [`list_nodes`](#list_nodes) | function | Every node row, sorted by id. |
| [`load_node`](#load_node) | function | Loads one node row. |
| [`load_or_create_node_identity`](#load_or_create_node_identity) | function | Loads the identity file or creates a new node id. |
| [`node_identity_path`](#node_identity_path) | function | Path of the node identity file. |
| [`node_target`](#node_target) | function | Bookclerk artifact target this binary runs as (`linux-x64-gnu`, `macos-arm64`, …). Matches `bookclerk_plugin_catalog::host_bookclerk_target` (a plugin-host test pins the two together). |
| [`add_expanded_members`](#add_expanded_members) | function | Adds the entries archive member `parent_no` expanded into, numbered after the batch's last member, and marks the archive expanded, with `parent_reason` when the walk stopped early. |
| [`batch_exists`](#batch_exists) | function | Whether a batch row exists for `id`, in any state. |
| [`batch_members`](#batch_members) | function | Reads every member of a batch, in member order. |
| [`cancel_batch`](#cancel_batch) | function | Cancels a batch the job has not started. |
| [`create_intake_batch`](#create_intake_batch) | function | Creates an intake batch: files read in place from `instance`, held by `user_id`, run on `node_id`. The host creates these itself (the drop folder); the person must be active. |
| [`create_upload_batch`](#create_upload_batch) | function | Creates an upload batch for `user_id`, staged on `node_id`. |
| [`delete_finished_before`](#delete_finished_before) | function | Deletes final batches (and their members) that finished before `before`, at most one result page per call. |
| [`final_batches_on`](#final_batches_on) | function | Batches on `node_id` in a final state: their staged bytes are no longer needed. |
| [`finish_batch`](#finish_batch) | function | Ends a running batch as `state` (done or failed), with `detail` for a failure. |
| [`first_active_owner`](#first_active_owner) | function | The library's first active Owner: who holds what the operator uploads with no person of their own. |
| [`idle_upload_batches`](#idle_upload_batches) | function | Open upload batches on `node_id` idle since before `before`: the prune cancels them and removes their staged bytes. |
| [`load_batch`](#load_batch) | function | Reads one batch. |
| [`load_member`](#load_member) | function | Reads one member. |
| [`mark_committed`](#mark_committed) | function | Hands an open batch to its job: every member must be complete and the manifest's every file present. |
| [`parse_upload_manifest`](#parse_upload_manifest) | function | Parses a `POST /api/uploads` body into the batch's members, checking every path with the archive path policy. Sans-IO: the `upload_manifest` fuzz target drives this same function. |
| [`parse_upload_offset`](#parse_upload_offset) | function | Parses an `Upload-Offset` header: plain ASCII digits (no sign, space or leading `+`), at most 19 of them, and no larger than `i64::MAX`. |
| [`patron_holding_id`](#patron_holding_id) | function | The id of the patron holding `user_id` has of `listing_id`, if any. |
| [`record_ingest_acquired`](#record_ingest_acquired) | function | Marks an edition acquired by a patron holding after the ingest job wrote its files, and publishes `edition_acquire_changed` so the library's page index picks the book up. |
| [`record_received`](#record_received) | function | Records that `received` bytes of a member are staged, and that the batch was active now. |
| [`review_items`](#review_items) | function | Members stored for review, oldest batch first, after `after` (a `(batch id, member)` cursor), at most one result page. |
| [`set_member_outcomes`](#set_member_outcomes) | function | Records outcomes the job decided, `MEMBERS_PER_WRITE` per host batch, while the batch is running. |
| [`staged_bytes`](#staged_bytes) | function | Bytes `user_id` has in live upload batches (open, committed or running), which the staging quota counts. |
| [`start_batch`](#start_batch) | function | Starts (or, after a crash, resumes) the job's run of a batch. |
| [`user_batches`](#user_batches) | function | A person's batches, newest first, at most `limit`. |
| [`accept_reported_origin`](#accept_reported_origin) | function | Adopts the origin the active instance reported and is waiting on the operator. |
| [`activate_ingress`](#activate_ingress) | function | Makes `instance` the library's active ingress, replacing any other. |
| [`deactivate_ingress`](#deactivate_ingress) | function | Turns ingress off. The adopted origin is kept, so turning the same provider back on serves under the same name. |
| [`ingress_settings`](#ingress_settings) | function | The ingress settings: the stored document, or ingress off. |
| [`load_ingress_settings`](#load_ingress_settings) | function | Loads the `core.ingress` document. |
| [`record_started`](#record_started) | function | Records what `instance` reported from `Ingress.start`. |
| [`set_ingress_exposure`](#set_ingress_exposure) | function | Sets what public traffic may reach. |
| [`import_instance_config_if_absent`](#import_instance_config_if_absent) | function | Inserts the document when it is absent. |
| [`load_instance_config`](#load_instance_config) | function | Loads and parses the instance config document. |
| [`parse_setting_text`](#parse_setting_text) | function | Parses an operator-typed value for `key` according to the schema. |
| [`reconcile_instance_configs_to_schema`](#reconcile_instance_configs_to_schema) | function | Rewrites every instance document of `plugin_key` to what `schema` still accepts: settings whose key is gone, became a secret, or whose value no longer fits (a type or enum change), and secret refs whose key is gone or became plain, are dropped. Table rows are brought to the current columns with `bookclerk_plugin_abi::SettingSpec::normalize_value`: cells of removed columns are dropped and defaults of new columns written, which changes the document without removing a key. |
| [`replace_instance_config`](#replace_instance_config) | function | Replaces the instance document when `expected_revision` is current. |
| [`resolve_instance_bindings`](#resolve_instance_bindings) | function | Resolves `CONFIG` and `SECRETS` for `instance_id` against `schema`. |
| [`seal_instance_secret`](#seal_instance_secret) | function | Seals one instance secret with the library DEK. |
| [`set_secret_ref`](#set_secret_ref) | function | Adds a secret ref for `key` whose sealed row is named `key`, replacing an existing ref for the same key. |
| [`set_setting`](#set_setting) | function | Sets `key` to `value` in `settings`, or removes it when `value` is `None`. |
| [`create_node_local_instance`](#create_node_local_instance) | function | Inserts a node-local instance of `plugin_key` owned by `node_id` (H-2): it adds to the library's outputs on that node only, and a deployment of it to any other node, or a placement, is refused. |
| [`create_plugin_instance`](#create_plugin_instance) | function | Inserts a new instance for `plugin_key`. |
| [`default_instance_for_key`](#default_instance_for_key) | function | The instance a host routes to when a job does not pin one: the `plugin_instance_defaults` claim for `plugin_key`, else the oldest row. |
| [`delete_plugin_instance`](#delete_plugin_instance) | function | Deletes an instance that has no deployments. Its configuration document, audit rows, and sealed secrets are left for retention cleanup. |
| [`ensure_plugin_instance`](#ensure_plugin_instance) | function | Returns the existing instance for `plugin_key`, or inserts one. |
| [`find_plugin_instance_by_key`](#find_plugin_instance_by_key) | function | Earliest instance row for `plugin_key`, if any. |
| [`list_all_plugin_instances`](#list_all_plugin_instances) | function | Every instance row, ordered by plugin key then creation time. |
| [`list_plugin_instances_for_key`](#list_plugin_instances_for_key) | function | Every instance row for `plugin_key`, oldest first. |
| [`load_plugin_instance`](#load_plugin_instance) | function | Loads one instance by id. |
| [`account_semaphore`](#account_semaphore) | function | Name of the library semaphore for one store account. |
| [`claim_routed_job`](#claim_routed_job) | function | Claims the best ready job this node can run, under the library semaphores. |
| [`class_semaphore`](#class_semaphore) | function | Name of the library semaphore for one resource class. |
| [`common_deployment_nodes`](#common_deployment_nodes) | function | Nodes where every instance in `requires` has a deployment that should run. |
| [`instance_running_on_database_node`](#instance_running_on_database_node) | function | True when some live node that holds the library database runs `instance`: it has a deployment there that should run, the node's last observation says the guest is up (`running` or `healthy`), and the node's heartbeat is current (`super::node_status::NodeStatus::is_live`). A node holds the database when it is a shared branch (`super::NodeKind::Shared`) or has no `nodes` row (a worker in this process); a branch with its own database does not. |
| [`instance_semaphore`](#instance_semaphore) | function | Name of the library semaphore for one plugin instance. |
| [`job_permit_holder_prefix`](#job_permit_holder_prefix) | function | The holder prefix of the semaphore permits a routed claim of `job_id` by `owner` takes when the claim runs at lease `generation`: `<owner>/<job_id>/g<generation>/`. |
| [`job_plugin_keys`](#job_plugin_keys) | function | The plugin keys `job` runs, or `None` when the job alone does not say. |
| [`job_semaphores`](#job_semaphores) | function | The library semaphores `job` counts against under `limits`. |
| [`job_source_plugin_key`](#job_source_plugin_key) | function | The one plugin whose source guest runs `job`, or `None` when the job alone does not name exactly one. |
| [`node_can_run_job`](#node_can_run_job) | function | True when `node_id` can run a job that requires `requires`. |
| [`resolve_job_requirements`](#resolve_job_requirements) | function | The instances a new job must find on its claiming node. |
| [`apply_keeper_update`](#apply_keeper_update) | function | Applies a keeper branch's push of one account's newest sign-in. |
| [`confirmed_keeping_branch`](#confirmed_keeping_branch) | function | The branch that has taken over this account's sign-in for this library: its `keeping_branch`, once that branch has pushed a sign-in here (`accounts.update`). Until then the library still keeps the account itself (saves its own refreshes and takes write-backs), so a branch that was sent the account but could not keep it never leaves it unrefreshed. |
| [`keeping_branch`](#keeping_branch) | function | The branch that keeps this account for this library: the node of its earliest-recorded share with a branch, or `None` when no branch holds it (this library keeps it). |
| [`keeps_for_link`](#keeps_for_link) | function | True when this install keeps the account for the library it joined as `library_id` (it holds a share with that library). |
| [`kept_shares`](#kept_shares) | function | Every account this install shares with a library it joined, each library once, with the revision last pushed there. |
| [`record_pushed_revision`](#record_pushed_revision) | function | Records that the keeper sent `revision` of the account to the library `library_id` (whether it applied it or already had it), for the sign-in whose fingerprint is `observed` (`SignIn::fingerprint`). Never lowers the recorded revision. |
| [`sign_in_revision`](#sign_in_revision) | function | The keeper's view of an account's sign-in: when its newest secret was written here and a fingerprint of all of them, or `None` when it has no secret. |
| [`acquire_lease`](#acquire_lease) | function | Acquires or renews `name` for `holder` for `ttl`. |
| [`acquire_lease_on`](#acquire_lease_on) | function | `acquire_lease` on a bare connection (schema migration runs before a `LibraryStore` exists). |
| [`list_leases`](#list_leases) | function | Every lease row, held or expired, by name. |
| [`load_lease`](#load_lease) | function | The current row of lease `name`, held or expired. |
| [`load_lease_on`](#load_lease_on) | function | `load_lease` on a bare connection. |
| [`release_lease`](#release_lease) | function | Gives up a held lease so another holder may take it at once. |
| [`release_lease_on`](#release_lease_on) | function | `release_lease` on a bare connection. |
| [`renew_lease`](#renew_lease) | function | Fencing check: extends a lease this holder still holds. |
| [`renew_lease_on`](#renew_lease_on) | function | `renew_lease` on a bare connection. |
| [`decode_spki_pin`](#decode_spki_pin) | function | Decodes an SPKI pin: base64url (no padding) of a SHA-256 digest. |
| [`join_cycle`](#join_cycle) | function | This library's own library id when `parent_ancestry` names it: joining that library would make a cycle (A ← B ← A, A ← B ← C ← A). `None` when it does not, or this install has no library identity yet. |
| [`join_overflows_ancestry`](#join_overflows_ancestry) | function | True when joining a library that reports `parent_ancestry` would give this library more ancestors (itself and every library above it) than `bookclerk_node_proto::MAX_ANCESTRY`: `library_ancestry` would then leave the farthest out, and a library joining this one later could no longer be checked for a loop through them. An ancestry cut at that limit by the library itself always holds exactly the limit, so this refuses it too. Such a join is refused rather than cut short. |
| [`leave_library_link`](#leave_library_link) | function | Leaves a library on this side: marks the link `LIBRARY_LINK_LEFT` and deletes its sealed node keys, so this install can never sign as that node again. Answers the link as it now stands, or `None` when there is no such link. |
| [`library_ancestry`](#library_ancestry) | function | This library's ancestry (what `GET /api/node/v1/info` reports): its own library id, then for each active link the owner's library id and the ancestry that owner reported, without repeats, at most `bookclerk_node_proto::MAX_ANCESTRY` ids. Empty when this install has no library identity yet. A join that would take it past that limit is refused (`join_overflows_ancestry`); should a library above grow its ancestry later, the list is cut at exactly the limit, which in turn refuses every join under it. |
| [`list_library_links`](#list_library_links) | function | Every library link, oldest enrollment first. |
| [`load_library_link`](#load_library_link) | function | One library link. |
| [`load_library_link_keys`](#load_library_link_keys) | function | Unseals the node keys of an active `link`. |
| [`mark_library_link_revoked`](#mark_library_link_revoked) | function | Marks an **active** link still enrolled as `remote_node_id` `LIBRARY_LINK_REVOKED` (the owner closed with 4403 on that node's connection). A link this install already left stays `left`, and one a later join enrolled as another node keeps its new enrollment. |
| [`parse_job_cap_request`](#parse_job_cap_request) | function | Parses and validates a `POST /api/config/join/capacity` body from its raw bytes, sans IO, the way the route reads it: JSON into `CapacityBody`, then `CapacityBody::validate`. |
| [`parse_sharing_request`](#parse_sharing_request) | function | Parses a `POST /api/config/join/sharing` body from its raw bytes, sans IO, the way the route reads it: JSON into `SharingBody`, then `SharingBody::into_request`. |
| [`record_library_link`](#record_library_link) | function | Seals `keys` and records the link, replacing an earlier link to the same library (a re-enrollment) as `active`. |
| [`set_library_link_ancestry`](#set_library_link_ancestry) | function | Records the owner library's ancestry as it reported it (`info` at join, `catalog.session` on every sync). Ids that are not plain library ids are refused; at most `bookclerk_node_proto::MAX_ANCESTRY` are kept. |
| [`set_library_link_job_cap`](#set_library_link_job_cap) | function | Sets the most jobs this install runs for one library at once; `None` clears the cap, leaving the link its equal share of the machine. Answers false when no such link exists. The running client picks the new cap up on the next share update. |
| [`set_library_link_keep_copies`](#set_library_link_keep_copies) | function | Sets the link's "Keep copies on this computer" setting. |
| [`set_library_link_listing`](#set_library_link_listing) | function | Sets the link's "List my members' books to *library*" setting. |
| [`set_library_link_person`](#set_library_link_person) | function | Binds the link to the local user who joined the library through it. |
| [`set_library_link_settings`](#set_library_link_settings) | function | Sets a link's sharing, listing and work settings in one statement, so a failed write leaves every one of them as it was. |
| [`set_library_link_sharing`](#set_library_link_sharing) | function | Sets a link's two sharing settings: "Share this branch's books with *library*" and "Include books from my own branches". Answers false when no such link exists. The running catalog sync reports them to the owner on its next pass. |
| [`set_library_link_state`](#set_library_link_state) | function | Sets a link's state. Answers false when no such link exists. |
| [`set_library_link_work`](#set_library_link_work) | function | Sets a link's two work settings: "Download for other members of *library*" and "Keep copies of other members' books". Answers false when no such link exists. The running client claims under the new scope from its next consent check. |
| [`books_imported_from_link`](#books_imported_from_link) | function | Every book imported from the library `library_id`, by store identity: the books the catalog sync must not send back to it. Read in keyset pages no larger than the database plugin's `maxResultRows`. |
| [`import_link_title`](#import_link_title) | function | Imports a title the library committed into this install's own library (see the module docs). |
| [`record_copy_taken_from_link`](#record_copy_taken_from_link) | function | Records that this install took its copy of `book` from the library `library_id`, so the catalog sync never sends that library a book it provided itself (`books_imported_from_link`). A book already recorded for that library is left as it is; one taken from several libraries is recorded once for each. |
| [`changed_topics`](#changed_topics) | function | The topics whose revision differs between `before` and `after`. |
| [`live_revisions`](#live_revisions) | function | Reads every topic's revision: one batch of one-row reads of `live_topic_revisions`, one per topic, so it fits an adapter that returns a single row per statement. |
| [`assigned_member_controls`](#assigned_member_controls) | function | The controls of `user`: the library's default, their overrides, and what applies to them. |
| [`default_member_controls`](#default_member_controls) | function | The library's default member controls: the stored document, or every control allowed. |
| [`effective_controls`](#effective_controls) | function | What applies to `user` given the library's default and their overrides: nothing for a user who is not active, everything for an active Owner, the overrides over "everything allowed" for an active Administrator, and the overrides over the default for an active Member. |
| [`load_default_member_controls`](#load_default_member_controls) | function | Loads the library's default member controls document. |
| [`member_control_overrides`](#member_control_overrides) | function | The overrides stored for `user_id` (none when there is no row). |
| [`parse_member_controls`](#parse_member_controls) | function | Parses the body of `PUT /api/users/default-controls` from its raw bytes, sans IO, the way the route reads it. |
| [`parse_user_controls_request`](#parse_user_controls_request) | function | Parses the body of `PUT /api/users/{id}/controls` from its raw bytes, sans IO, the way the route reads it. |
| [`role_follows_default_controls`](#role_follows_default_controls) | function | Whether a role follows the library's default controls: Members do. An Administrator may do everything except the controls set for them alone, so a default meant for patrons never limits staff. |
| [`role_has_controls`](#role_has_controls) | function | Whether member controls ever limit a role: Members and Administrators can be limited; Owners never are. |
| [`set_default_member_controls`](#set_default_member_controls) | function | Replaces the library's default member controls under compare-and-swap, creating the document when absent and retrying a few times on a concurrent edit. |
| [`set_member_control_overrides`](#set_member_control_overrides) | function | Replaces the overrides of `user_id`; with none, the user follows the library's default for every control again. |
| [`set_member_control_overrides_for`](#set_member_control_overrides_for) | function | Replaces the overrides of `user`, as the caller read them when it checked that the actor may manage them; with none, the user follows the library's default for every control again. |
| [`user_may`](#user_may) | function | Whether `user_id` may do `control` here. An unknown user may not. |
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
| [`import_output_if_absent`](#import_output_if_absent) | function | Inserts `[output]` when the library document is absent. |
| [`is_output_setting_key`](#is_output_setting_key) | function | True when `key` is a `core.output` setting or alias. |
| [`load_output`](#load_output) | function | Loads the library output document. |
| [`overlay_output`](#overlay_output) | function | Copies the authoritative output document onto `config` for `library_id`. |
| [`portable_output_seed`](#portable_output_seed) | function | Copies `[output]` with a `widevine_cdm` under `files_dir` made relative again. |
| [`replace_output`](#replace_output) | function | Replaces the library output document. |
| [`nodes_owned_by`](#nodes_owned_by) | function | Nodes `user_id` owns, sorted by id. |
| [`set_node_owner`](#set_node_owner) | function | Assigns `node_id` to `owner` (a user id), or clears its owner. |
| [`verify_node_owner`](#verify_node_owner) | function | Returns a `ConfigActor::NodeOwner` for `user_id` on `node_id` when the node's stored owner is that user. |
| [`assigned_patron_scope`](#assigned_patron_scope) | function | The scope a user has: their own, or the library's default. |
| [`default_patron_scope`](#default_patron_scope) | function | The library's default scope: the stored document, or no restriction. |
| [`default_work_scope`](#default_work_scope) | function | The page index filter for a caller who is not a library user, such as a session whose identity is linked to nobody: the library default, with no store accounts of their own. `None` when the default hides nothing. |
| [`load_default_patron_scope`](#load_default_patron_scope) | function | Loads the library's default scope document. |
| [`parse_patron_scope`](#parse_patron_scope) | function | Parses the body of `PUT /api/users/default-scope` from its raw bytes, sans IO, the way the route reads it: JSON into `PatronScope`, then trimmed (`PatronScope::normalized`) and checked (`PatronScope::validate`) as `set_default_patron_scope` does. |
| [`parse_user_scope_request`](#parse_user_scope_request) | function | Parses the body of `PUT /api/users/{id}/scope` from its raw bytes, sans IO, the way the route reads it: JSON into `PutUserScope`, then the scope (when one is given) trimmed and checked as `set_patron_scope` does. |
| [`role_is_scoped`](#role_is_scoped) | function | Whether a role is ever scoped: Members are patrons, while Owners and Administrators always see the whole library. |
| [`scope_for_user`](#scope_for_user) | function | The scope that applies to `user`: `None` for an Owner or Administrator, who see everything, and the assigned scope for a Member. |
| [`set_default_patron_scope`](#set_default_patron_scope) | function | Replaces the library's default scope under compare-and-swap, creating the document when absent and retrying a few times on a concurrent edit. |
| [`set_patron_scope`](#set_patron_scope) | function | Gives a user their own scope, or with `None` returns them to the library's default. |
| [`verify_patron_administrator`](#verify_patron_administrator) | function | Checks that `user_id` is an active Owner or Administrator and returns the actor that may write the library's default scope. |
| [`work_scope_for_user`](#work_scope_for_user) | function | The page index filter for `user`: `None` when nothing is hidden from them (an Owner or Administrator, or a patron whose scope has no rules). |
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
| [`list_remote_plugin_consents`](#list_remote_plugin_consents) | function | Every consent row, of one library when `library_id` is given, by library then plugin key. |
| [`load_remote_plugin_consent`](#load_remote_plugin_consent) | function | The consent row for `plugin_key` of `library_id`. |
| [`record_remote_plugin_request`](#record_remote_plugin_request) | function | Records what the owner asks to run here: `version` of `plugin_key` with the owner's approved grant, fetched from `artifact`. A stored approval is kept; whether it still covers the new request is decided when the grant is evaluated (and an approval of another archive never does, `RemotePluginConsent::approved_artifact_matches`). Writes nothing when the request is unchanged. |
| [`remote_request_digest`](#remote_request_digest) | function | The digest an operator approves a request by: SHA-256 (lowercase hex) over the requested version, the owner's grant exactly as recorded and the archive's URL, digest and payload-root digest, each length-prefixed, so any change to any of them gives another digest. |
| [`revoke_remote_plugin`](#revoke_remote_plugin) | function | Clears the approval of `plugin_key` of `library_id`, keeping its request, and records the revoke so no automatic approval brings it back. Answers the row, or `None` when nothing is recorded. |
| [`accept_scan_pass_end`](#accept_scan_pass_end) | function | Records how a Host's pass of one account ended. |
| [`accept_scan_removed`](#accept_scan_removed) | function | Removes the holdings a Host's delta pass reported gone from one account. |
| [`accept_scan_results`](#accept_scan_results) | function | Validates one batch of a Host's scan rows and writes it to the library. |
| [`accept_scan_rows`](#accept_scan_rows) | function | Validates one batch of a Host's scan rows and writes it to the library, marking every listed product seen by the Host's pass. |
| [`handed_pass_id`](#handed_pass_id) | function | The pass id a Host's scan of `job_id` under lease generation `generation` stamps on the holdings its rows list (`holdings.seen_pass`). |
| [`handed_sync_token`](#handed_sync_token) | function | The sync token the owner hands with a Host scan of one account: the stored token while the account's last complete pass is recent enough to build on, otherwise empty (a full pass), as a local pass would start. |
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
| [`align_library_root`](#align_library_root) | function | Ensures this process's DEK is the library secret root. |
| [`load_library_row`](#load_library_row) | function | Reads the singleton library row when present. |
| [`acquire_permit`](#acquire_permit) | function | Takes one permit of `name` for `holder`, valid for `ttl`. |
| [`acquire_permits`](#acquire_permits) | function | Takes one permit of every semaphore in `requests`, or none. |
| [`live_permits`](#live_permits) | function | Live permits of `name` among its first `max_slots` slots (expired slots are not counted). |
| [`permits_held_by`](#permits_held_by) | function | Permits of `name` among its first `max_slots` slots whose holder starts with `holder_prefix`, expired ones included. |
| [`release_permits`](#release_permits) | function | Gives back every permit in `permits` so the slots are free at once. |
| [`renew_fenced_lease`](#renew_fenced_lease) | function | Extends `lease` by `ttl` only while the row still carries its holder and token. Unlike `acquire_lease`, it never takes a lease over. |
| [`renew_permits`](#renew_permits) | function | Extends every permit in `permits` by `ttl` while its holder and token still own the slot. |
| [`semaphore_slot_lease_name`](#semaphore_slot_lease_name) | function | Lease name of slot `slot` of semaphore `name`. |
| [`library_contents`](#library_contents) | function | Counts the library data this install holds of its own. |
| [`remove_shared_node`](#remove_shared_node) | function | Removes shared node `node_id` from the library's node list, in one transaction: its `nodes` row (and with it its labels and credentials), status, event-subscriber registrations, secret handoffs, and plugin deployments with their observations. Only a node that is not running may go, and never `this_node`. The liveness check and the removal are one step: the node's status row is deleted only if it still holds the stale heartbeat that was checked, so a heartbeat that lands in between refuses the removal. A node that was only cut off from the database for longer than its liveness window can still be removed while it runs; its next heartbeat writes a status row again, but not its node row. Removing a node does not take back the database credentials or DEK it held; rotate those when it is not trusted any more. |
| [`shared_nodes_visible_to`](#shared_nodes_visible_to) | function | The shared nodes (`kind = shared`: this one and its shared branches) `actor` may see, by the rule `remove_shared_node` removes them: the operator or an active Owner sees every one; another active user only the ones they own; an unknown or disabled user none. |
| [`strip_install_state`](#strip_install_state) | function | Removes this install's own state from the library so that copies of it can start as installs of their own. |
| [`accounts_owned_by`](#accounts_owned_by) | function | Store accounts linked to `user_id` through any of the user's identities. |
| [`acquire_titles_of_editions`](#acquire_titles_of_editions) | function | The library titles an acquire by edition queues: for each edition, one title (`books.uuid`) that sells it in an account `requester` may acquire for, and in `account` when one is named. |
| [`claimant_may_run`](#claimant_may_run) | function | True when a `claimant` may run `job` at all, whatever it is trusted with: a forced, PDF-only or paced acquire runs only on a node that holds the library database, since the host acquire path does none of them (a paced acquire checks and records the account's pace in the library database before each download, `crate::pacing`). An interlibrary request (`JobKind::BranchRequest`) is never run through the routed claim: only its branch claims it, over `requests.claim` (`super::claim_branch_requests`). An ingest batch and a notification send run only on a node that holds the library database. |
| [`handoff_routed_job`](#handoff_routed_job) | function | Re-checks a claimed job just before its secrets are handed to the claiming node. |
| [`job_for_node_owner`](#job_for_node_owner) | function | True when every store account `job` touches belongs to the user who owns `node_id`: the job is the branch owner's own work (`bookclerk_node_proto::WorkJob::for_node_owner`). |
| [`job_item_accounts`](#job_item_accounts) | function | The store accounts a scan or acquire touches: its items. |
| [`job_trusted`](#job_trusted) | function | True when a node with `trust` may receive `job`. |
| [`node_may_claim_job`](#node_may_claim_job) | function | True when `node_id` may claim `job` right now, whoever's work it is. |
| [`node_may_claim_job_in_scope`](#node_may_claim_job_in_scope) | function | True when `node_id` may claim `job` right now under a claim of `scope` (`node_trust_in_scope`). |
| [`node_trust`](#node_trust) | function | Resolves what `node_id` is trusted with right now. |
| [`node_trust_in_scope`](#node_trust_in_scope) | function | What `node_id` is trusted with for a claim of `scope`. |
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
| [`job_titles`](#job_titles) | function | The titles the acquire `job` run by `node_id` covers, read from the account holdings as a local acquire reads them: of the job's plugins (`job_plugin_keys`), of the accounts its account selector names (`job_accounts`, so a job naming its account by label or in another case covers that account's titles) and its title when it names one (`LibraryStore::acquire_titles_named`, matched as the executor matches it), and, for a Host trusted only with its owner's accounts, one of that user's. `None` when the node is trusted with nothing. |
| [`load_upload`](#load_upload) | function | Loads one upload by id. |
| [`record_upload_file`](#record_upload_file) | function | Records that file `index` reached every destination and extends the upload's token. Answers false when the upload is no longer open (it was aborted meanwhile: the caller deletes the copies it just wrote). |
| [`upload_fence_live`](#upload_fence_live) | function | True while the upload's fence is the job's live lease for its node and the node is still trusted with the job. |
| [`upload_owns_key`](#upload_owns_key) | function | True while `key` on `destination` is still `session`'s own: its recorded latest writer is this upload (an upload id is unique, and claims keys for its own title only). A later upload of the title, an acquire run on this node, or another title may have written the key since; the receiving node then leaves the file alone when it cleans up after `session`, so a stale abort never deletes a replacement's file. |
| [`activate_version`](#activate_version) | function | Moves the active pointer of `plugin_key` to `version` (first install, update, or rollback), pins the version's settings schema, and wakes every node. Pointing at the version that is already active is a no-op. |
| [`add_plugin_version`](#add_plugin_version) | function | Records a version. Installing the same `(plugin_key, version)` again with the same artifact digests is a no-op (`AddVersion::Existing`); different digests under an installed version are refused, so a version never changes content behind a node's cache. |
| [`approve_version_grant`](#approve_version_grant) | function | Records the operator-approved consent grant on a version. |
| [`clear_install`](#clear_install) | function | Clears the active pointer (`plugins uninstall`). Version rows stay so the plugin can be re-activated; nodes retire guests whose plugin has no active version and sweep the cache. |
| [`list_installs`](#list_installs) | function | Every active pointer. |
| [`list_versions`](#list_versions) | function | Every version of `plugin_key` (or of every plugin), newest install first. |
| [`load_active_version`](#load_active_version) | function | The active version of `plugin_key`, when a pointer names one. |
| [`load_install`](#load_install) | function | The active pointer of `plugin_key`. |
| [`load_version`](#load_version) | function | One version row. |
| [`clear_window_books`](#clear_window_books) | function | Drops every window book of library `parent_library_id` (its link downloads again, or was left). |
| [`discard_window_read`](#discard_window_read) | function | Drops the rows a read cut short by a settings change wrote (stamped `seen_at`), so a read that began before downloading was turned on cannot put back the list the change cleared. |
| [`finish_window_read`](#finish_window_read) | function | Ends a complete read of library `parent_library_id`'s window books: drops its rows that read (stamped `seen_at`) did not name. Rows are compared by stamp equality, not order, so a clock moved back between reads still drops what the new read left out. |
| [`person_books_page`](#person_books_page) | function | One page of this library's books for a person, as a clerk signed in with that person's library card reads them over the client socket (`GET /api/clerk/v1/connect`): the same works, grouping and limits as `window_books_page`, for the person the card belongs to rather than a branch node's owner. |
| [`record_window_page`](#record_window_page) | function | Stores one `window.page` from library `parent_library_id`, read under `seen_at` (`window_read_stamp`). |
| [`window_books_page`](#window_books_page) | function | One page of the books this library holds for branch `node_id`'s owner, for the `window.books` frame. |
| [`window_read_stamp`](#window_read_stamp) | function | A stamp for one complete read of a library's window books: pass it to every `record_window_page` of the read and then to `finish_window_read`. |
| [`window_view`](#window_view) | function | The merged view of every library this install is a branch of or holds a library card for: their books, one entry per work, sorted by title. |
| [`bootstrap_control_plane`](#bootstrap_control_plane) | function | Enrolls this files directory and imports `[events]` and `[output]` when their documents are absent. |
| [`overlay_core_documents`](#overlay_core_documents) | function | Copies every authoritative library document in `session` onto `config`. |
| [`overlay_events`](#overlay_events) | function | Copies the authoritative events document onto `config` for `library_id`. |
| [`load_events`](#load_events) | function | Loads the library events document. |
| [`import_events_if_absent`](#import_events_if_absent) | function | Inserts `[events]` when the library document is absent. |
| [`replace_events`](#replace_events) | function | Replaces the library events document. |
| [`load_node_runtime`](#load_node_runtime) | function | Loads one node's runtime document. |
| [`import_node_runtime_if_absent`](#import_node_runtime_if_absent) | function | Inserts a node runtime document when it is absent. |
| [`replace_node_runtime`](#replace_node_runtime) | function | Replaces one node's runtime document. |
| [`update_node_runtime`](#update_node_runtime) | function | Applies `edit` to a node's runtime document (creating it with defaults when the node never wrote one) under compare-and-swap, retrying a few times on a concurrent edit. Budgets reach the scheduler with the node's next heartbeat. |
| [`HeldBookKey`](#heldbookkey) | type alias | A book this library holds, by store identity: `(plugin_key, stored marketplace, stored product id, account_id)`. |
| [`LiveRevisions`](#liverevisions) | type alias | Every topic's revision, in `LiveTopic::ALL` order. |
| [`AccountKey`](#accountkey) | type alias | A store account, keyed as in `accounts`: `(plugin_key, account_id)`. |
| [`ADMIN_CONTROLS_NAMESPACE`](#admin_controls_namespace) | constant | Namespace of the library document that holds the library's default Administrator switches. |
| [`MAX_ARTIFACT_BLOB_BYTES`](#max_artifact_blob_bytes) | constant | Largest archive stored in the library, in bytes. |
| [`DETACHED_LEFT`](#detached_left) | constant | Removal reason: the branch left the library (or was revoked). |
| [`DETACHED_STOPPED_SHARING`](#detached_stopped_sharing) | constant | Removal reason: the branch turned "Share this branch's books" off. |
| [`DETACHED_WITHDRAWN`](#detached_withdrawn) | constant | Removal reason: a full pass of the branch's catalog no longer named it. |
| [`FORWARDABLE_KEYS_MAX_PAGE`](#forwardable_keys_max_page) | constant | Upper clamp for one `forwardable_branch_title_keys_page` read. |
| [`FULL_CATALOG_PASS_EVERY`](#full_catalog_pass_every) | constant | How long a branch's catalog sync resumes incrementally before the library asks for a full pass again, so titles a branch no longer has (removed, or from a sub-branch that left) are withdrawn within a day. |
| [`REQUEST_ACCEPTED`](#request_accepted) | constant | Progress text of a request its branch accepted. |
| [`REQUEST_BRANCH_FAILED`](#request_branch_failed) | constant | `error_kind` of a request its branch could not take now. |
| [`REQUEST_LEASE_SECS`](#request_lease_secs) | constant | How long a branch holds a claimed request before it goes back to pending. Taking a request is a few local reads and writes on the branch, so this is generous. |
| [`REQUEST_NOT_LISTED`](#request_not_listed) | constant | `error_kind` of a request its branch no longer lists. |
| [`REQUEST_WORK_GONE`](#request_work_gone) | constant | `error_kind` of a request whose work has nothing left to ask by (it was merged away, or lost every identifier) when its branch came for it. |
| [`CACHE_NAMESPACE`](#cache_namespace) | constant | Namespace of the library cache document. |
| [`CACHE_SCHEMA_VERSION`](#cache_schema_version) | constant | Schema version of `CacheSettingsV1`. |
| [`CACHE_SETTING_KEYS`](#cache_setting_keys) | constant | Settings keys the cache document owns, in display order. |
| [`SHARE_RECEIVED`](#share_received) | constant | `account_shares.direction` of an account this install received from a library it joined. |
| [`SHARE_SENT`](#share_sent) | constant | `account_shares.direction` of an account this library sent to a branch. |
| [`BRANCH_LISTING_PREFIX`](#branch_listing_prefix) | constant | Plugin key prefix of the listings branches answer with. |
| [`BRANCH_LISTING_TTL`](#branch_listing_ttl) | constant | How long a `branch:<node>` listing counts as current after the branch last named it: a day of answer life plus a day for the next question to be asked and answered. |
| [`LOOKUP_ANSWER_TTL`](#lookup_answer_ttl) | constant | How long a branch's answer stands: a question answered longer ago is asked again the next time `ask_branches` names its work. |
| [`LOOKUP_RESEND`](#lookup_resend) | constant | How long a sent question waits for its answer before it is sent again. |
| [`CLERK_CARD_NAME`](#clerk_card_name) | constant | `encrypted_secrets.name` of a clerk link's library card. |
| [`COLLECTIONS_NAMESPACE`](#collections_namespace) | constant | Namespace of the library document that holds the library's collections. |
| [`COLLECTIONS_SCHEMA_VERSION`](#collections_schema_version) | constant | Schema version of `LibraryCollections` in the document. |
| [`MAX_COLLECTIONS`](#max_collections) | constant | Most collections a library may define. |
| [`MAX_COLLECTIONS_JSON_BYTES`](#max_collections_json_bytes) | constant | Largest set of collections as compact JSON, in bytes: well inside the largest configuration document. |
| [`MAX_COLLECTION_NAME_BYTES`](#max_collection_name_bytes) | constant | Longest collection name, in UTF-8 bytes after trimming. |
| [`MAX_COLLECTION_PREFIXES`](#max_collection_prefixes) | constant | Most prefixes one collection may have. |
| [`MAX_COLLECTION_PREFIX_BYTES`](#max_collection_prefix_bytes) | constant | Longest prefix, in UTF-8 bytes after trimming and the closing `/`. |
| [`MAX_ADD_CONNECTIONS`](#max_add_connections) | constant | Most accounts one "Add my connections" request names: as many as one sealed account bundle carries. |
| [`COPY_TOKEN_TTL_SECS`](#copy_token_ttl_secs) | constant | Seconds a copy token is accepted after its grant. |
| [`CRON_FIRE_MAX_ATTEMPTS`](#cron_fire_max_attempts) | constant | Failure attempts a cron fire gets before it is terminal (the first run plus two retries with the queue's backoff). |
| [`MAX_OBSERVATION_DETAIL_BYTES`](#max_observation_detail_bytes) | constant | Maximum UTF-8 size of an observation detail string. |
| [`DESTINATION_NAMESPACE`](#destination_namespace) | constant | Namespace of a destination's host-owned placement document. |
| [`DESTINATION_SCHEMA_VERSION`](#destination_schema_version) | constant | Schema version of `DestinationPlacementV1`. |
| [`LIBRARY_SCOPE_ID`](#library_scope_id) | constant | Library scope instance. There is one library document per namespace. |
| [`LIBRARY_SCOPE_TYPE`](#library_scope_type) | constant | Scope type stored for library documents. |
| [`MAX_CONFIGURATION_DOCUMENT_BYTES`](#max_configuration_document_bytes) | constant | Maximum UTF-8 size of `document_json`. |
| [`NODE_SCOPE_TYPE`](#node_scope_type) | constant | Scope type stored for node documents. |
| [`PLUGIN_INSTANCE_CONFIG_NAMESPACE`](#plugin_instance_config_namespace) | constant | Namespace of a plugin instance's typed configuration document. |
| [`PLUGIN_INSTANCE_SCOPE_TYPE`](#plugin_instance_scope_type) | constant | Scope type stored for one plugin instance's configuration. |
| [`AUTHORIZED_LINK_CREATED_BY`](#authorized_link_created_by) | constant | Prefix of the `created_by` audit id of a link a sign-in created. |
| [`BRANCH_ENROLLMENT_NAMESPACE`](#branch_enrollment_namespace) | constant | Namespace of the library's branch-enrollment switch. |
| [`BRANCH_ENROLLMENT_SCHEMA_VERSION`](#branch_enrollment_schema_version) | constant | Schema version of `BranchEnrollmentSettingsV1`. |
| [`DEFAULT_LINK_TTL`](#default_link_ttl) | constant | Default lifetime of a link. |
| [`ENROLLMENT_TOKEN_PREFIX`](#enrollment_token_prefix) | constant | Version tag that starts every enrollment token. |
| [`MAX_LINK_LABEL_BYTES`](#max_link_label_bytes) | constant | Longest `label` stored on a link. |
| [`MAX_LINK_TTL`](#max_link_ttl) | constant | Longest lifetime a link may be minted with. |
| [`MIN_LINK_TTL`](#min_link_ttl) | constant | Shortest lifetime a link may be minted with. |
| [`NODE_PUBLIC_KEY_LEN`](#node_public_key_len) | constant | Size of an Ed25519 or X25519 public key. |
| [`NODE_IDENTITY_FILE`](#node_identity_file) | constant | Bootstrap file under `$BOOKCLERK_FILES_DIR` that stores the stable node id. |
| [`MAX_BATCH_FILES`](#max_batch_files) | constant | Most files one batch's manifest may name. |
| [`MAX_INTAKE_REF_BYTES`](#max_intake_ref_bytes) | constant | Longest intake storage key or signature a member may carry, in bytes; both are stored in one cell. |
| [`MAX_REASON_BYTES`](#max_reason_bytes) | constant | Longest outcome reason, in bytes. |
| [`INGRESS_NAMESPACE`](#ingress_namespace) | constant | Namespace of the library document that holds the ingress settings. |
| [`INGRESS_SCHEMA_VERSION`](#ingress_schema_version) | constant | Schema version of `IngressSettings`. |
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
| [`RECORD_RETENTION_LEASE`](#record_retention_lease) | constant | Lease of the record retention duty (expired credentials and tokens, finished uploads, abandoned scan inventories, old audit rows). |
| [`SESSION_EXPIRY_LEASE`](#session_expiry_lease) | constant | Lease of the session duty (expired operator sessions, OIDC RP states, WebAuthn challenges). |
| [`LIBRARY_LINK_ACTIVE`](#library_link_active) | constant | `library_links.state` of a link the host client connects. |
| [`LIBRARY_LINK_KEYS_NAME`](#library_link_keys_name) | constant | `encrypted_secrets.name` of a link's node keys. |
| [`LIBRARY_LINK_LEFT`](#library_link_left) | constant | `library_links.state` after this install left the library. |
| [`LIBRARY_LINK_REVOKED`](#library_link_revoked) | constant | `library_links.state` once the owner revoked this node (close 4403). |
| [`MEMBER_CONTROLS_NAMESPACE`](#member_controls_namespace) | constant | Namespace of the library document that holds the library's default member controls. |
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
| [`EXPLICIT_FACET_KEY`](#explicit_facet_key) | constant | The page index value a work marked explicit carries under `WorkFacet::Explicit`. |
| [`MAX_SCOPE_JSON_BYTES`](#max_scope_json_bytes) | constant | Largest scope as compact JSON, in bytes: one cell of the smallest reply the host accepts from a database plugin (`HOST_MIN_CELL_BYTES`), so a stored scope can be read back on every supported database. Sixteen rules fit when their values are short. |
| [`MAX_SCOPE_RULES`](#max_scope_rules) | constant | Most rules one scope may hold. |
| [`MAX_SCOPE_VALUE_BYTES`](#max_scope_value_bytes) | constant | Longest rule value, in UTF-8 bytes. |
| [`PATRON_SCOPE_NAMESPACE`](#patron_scope_namespace) | constant | Namespace of the library document that holds the library's default scope. |
| [`MAX_JOIN_NAME_BYTES`](#max_join_name_bytes) | constant | Longest branch or library name stored. |
| [`PENDING_JOIN_TTL`](#pending_join_ttl) | constant | How long a sign-in may take before its pending row is refused. |
| [`MAX_NODE_LABELS`](#max_node_labels) | constant | Most operator labels one node may carry. |
| [`MAX_PLACEMENT_CONSTRAINTS`](#max_placement_constraints) | constant | Most constraints one placement may carry. |
| [`MAX_REPLICAS`](#max_replicas) | constant | Largest replica count an operator may ask for. |
| [`NODE_LABEL`](#node_label) | constant | Implicit label name every node carries with its own id. |
| [`SCHEDULER_LEASE`](#scheduler_lease) | constant | Name of the library lease the scheduler runs under. |
| [`SCHEDULER_LEASE_TTL`](#scheduler_lease_ttl) | constant | How long a scheduler lease lasts without renewal. |
| [`TARGET_LABEL`](#target_label) | constant | Implicit label name every node carries with its artifact target. |
| [`REMOTE_CONSENT_PLATFORM`](#remote_consent_platform) | constant | `approved_by` of an automatic approval: a bundled platform plugin whose request stays inside the platform envelope. |
| [`MAX_SCAN_RESULT_ROWS`](#max_scan_result_rows) | constant | Most rows one results batch may carry. |
| [`MAX_WRITE_BACK_BYTES`](#max_write_back_bytes) | constant | Largest store token a Host may write back. |
| [`REMOTE_LEASE_OWNER_PREFIX`](#remote_lease_owner_prefix) | constant | Prefix of the lease owner of a job a Host runs. |
| [`SECRETS_UNAVAILABLE`](#secrets_unavailable) | constant | Error kind recorded when a job's secrets cannot be unsealed for a Host. |
| [`REQUIREMENTS_GRACE`](#requirements_grace) | constant | How long a pending job's requirements may stay unsettled before the sweep acts on them. Several scheduler passes (20 s apart) and guest restarts fit inside it. |
| [`REQUIREMENTS_SWEEP_LEASE`](#requirements_sweep_lease) | constant | Library lease the daemon holds while it sweeps. |
| [`REQUIREMENTS_SWEEP_LEASE_TTL`](#requirements_sweep_lease_ttl) | constant | Lease length; the holder renews it on every pass. |
| [`UNROUTABLE_ERROR_KIND`](#unroutable_error_kind) | constant | `error_kind` of a job the sweep failed. |
| [`ROUTE_DEFER_PER_RANK`](#route_defer_per_rank) | constant | How long each rank waits behind the one above it. |
| [`MAX_SEMAPHORE_LIMIT`](#max_semaphore_limit) | constant | Largest limit a semaphore honours; larger limits are clamped. |
| [`SEMAPHORE_LEASE_PREFIX`](#semaphore_lease_prefix) | constant | Prefix of every semaphore slot lease name. |
| [`UPLOAD_NAME_WILDCARD`](#upload_name_wildcard) | constant | In a `TitleNames::part_patterns` pattern: one or more characters of a file name (no `/`), where a part or chapter title goes. Storage keys hold no control character, so the marker never clashes with one. |
| [`UPLOAD_NUMBER_WILDCARD`](#upload_number_wildcard) | constant | In a `TitleNames::part_patterns` pattern: one or more ASCII digits, where a part or chapter number goes. |
| [`UPLOAD_TOKEN_TTL_SECS`](#upload_token_ttl_secs) | constant | How long a granted upload's token is accepted past its latest extension (the grant, an accepted file, or a renewal of the job's lease). |
| [`ARTIFACT_BLOB_SCHEME`](#artifact_blob_scheme) | constant | Artifact URL scheme of an archive stored in the library (`super::artifacts::put_artifact_blob`). |
| [`WINDOW_BOOKS_READ`](#window_books_read) | constant | Most holdings one `window_books_page` reads. |
| [`EVENTS_NAMESPACE`](#events_namespace) | constant | Namespace for the migrated `[events]` domain. |
| [`NODE_RUNTIME_NAMESPACE`](#node_runtime_namespace) | constant | Namespace for node-scoped runtime settings. |
| [`EVENTS_SCHEMA_VERSION`](#events_schema_version) | constant | Schema version of `EventsSettingsV1`. |
| [`NODE_RUNTIME_SCHEMA_VERSION`](#node_runtime_schema_version) | constant | Schema version of `NodeRuntimeSettingsV1`. |
| [`MAX_EVENTS_RETENTION_DAYS`](#max_events_retention_days) | constant | Upper bound for event retention fields, in days. |
| [`MAX_EVENTS_CONCURRENCY`](#max_events_concurrency) | constant | Upper bound for `[events].concurrency`. |
| [`DEFAULT_CPU_BUDGET_PERCENT`](#default_cpu_budget_percent) | constant | Default CPU budget: leave a fifth of the cores to the rest of the node. |
| [`DEFAULT_MEMORY_BUDGET_PERCENT`](#default_memory_budget_percent) | constant | Default memory budget: leave a quarter of the memory to the rest of the node. |

## Re-exports

- [`cache::DEFAULT_CACHE_BUDGET_BYTES`](../bookclerk_config/README.md#default_cache_budget_bytes)
- [`cache::DEFAULT_CACHE_UNVIEWED_DAYS`](../bookclerk_config/README.md#default_cache_unviewed_days)
- [`cache::MAX_CACHE_BUDGET_BYTES`](../bookclerk_config/README.md#max_cache_budget_bytes)
- [`cache::MAX_CACHE_UNVIEWED_DAYS`](../bookclerk_config/README.md#max_cache_unviewed_days)
- [`cache::MIN_CACHE_BUDGET_BYTES`](../bookclerk_config/README.md#min_cache_budget_bytes)

## Structs

### AdminControlOverrides

```rust
pub struct AdminControlOverrides {
    pub manage_members: Option<bool>,
    pub set_patron_limits: Option<bool>,
    pub library_scan: Option<bool>,
    pub run_jobs: Option<bool>,
    pub fix_catalog: Option<bool>,
    pub invite_branches: Option<bool>,
    pub approve_requests: Option<bool>,
}
```

One Administrator's overrides: a value for some switches, and the
library's default for the rest (an absent or `null` field).

**Fields**

- `manage_members`: Overrides [`AdminControls::manage_members`](#admincontrols).
- `set_patron_limits`: Overrides [`AdminControls::set_patron_limits`](#admincontrols).
- `library_scan`: Overrides [`AdminControls::library_scan`](#admincontrols).
- `run_jobs`: Overrides [`AdminControls::run_jobs`](#admincontrols).
- `fix_catalog`: Overrides [`AdminControls::fix_catalog`](#admincontrols).
- `invite_branches`: Overrides [`AdminControls::invite_branches`](#admincontrols).
- `approve_requests`: Overrides [`AdminControls::approve_requests`](#admincontrols).

#### AdminControlOverrides::get

```rust
pub fn get(&self, control: AdminControl) -> Option<bool>
```

The override of `control`, if any.

#### AdminControlOverrides::set

```rust
pub fn set(&mut self, control: AdminControl, value: Option<bool>)
```

Sets or clears (`None`) the override of `control`.

#### AdminControlOverrides::is_empty

```rust
pub fn is_empty(&self) -> bool
```

True when no switch is overridden.

#### AdminControlOverrides::apply

```rust
pub fn apply(&self, defaults: AdminControls) -> AdminControls
```

`defaults` with these overrides applied.

Implements: `Clone`, `ComposeSchema`, `Copy`, `Debug`, `Default`, `Deserialize<'de>`, `Eq`, `PartialEq`, `Serialize`, `ToSchema`

### AdminControls

```rust
pub struct AdminControls {
    pub manage_members: bool,
    pub set_patron_limits: bool,
    pub library_scan: bool,
    pub run_jobs: bool,
    pub fix_catalog: bool,
    pub invite_branches: bool,
    pub approve_requests: bool,
}
```

A value for each switch: the library's default, or what applies to one
Administrator.

**Fields**

- `manage_members`: May manage Members ([`AdminControl::ManageMembers`](#admincontrol)).
- `set_patron_limits`: May set patron limits ([`AdminControl::SetPatronLimits`](#admincontrol)).
- `library_scan`: May scan and acquire library-wide ([`AdminControl::LibraryScan`](#admincontrol)).
- `run_jobs`: May run the job queue ([`AdminControl::RunJobs`](#admincontrol)).
- `fix_catalog`: May fix the catalog ([`AdminControl::FixCatalog`](#admincontrol)).
- `invite_branches`: May invite branches ([`AdminControl::InviteBranches`](#admincontrol)).
- `approve_requests`: May approve interlibrary requests ([`AdminControl::ApproveRequests`](#admincontrol)).

#### AdminControls::all

```rust
pub fn all(value: bool) -> Self
```

Every switch set to `value`.

#### AdminControls::allows

```rust
pub fn allows(&self, control: AdminControl) -> bool
```

Whether `control` is allowed.

#### AdminControls::set

```rust
pub fn set(&mut self, control: AdminControl, value: bool)
```

Sets `control` to `value`.

Implements: `Clone`, `ComposeSchema`, `Copy`, `Debug`, `Default`, `Deserialize<'de>`, `Eq`, `PartialEq`, `Serialize`, `ToSchema`

### AssignedAdminControls

```rust
pub struct AssignedAdminControls {
    pub defaults: AdminControls,
    pub overrides: AdminControlOverrides,
    pub effective: AdminControls,
}
```

A user's switches: the library's default, their overrides, and what
applies to them.

**Fields**

- `defaults`: The library's default.
- `overrides`: The user's own overrides.
- `effective`: What applies to the user: everything for an active Owner, the
  default with the overrides applied for an active Administrator, and
  nothing for anyone else.

Implements: `Clone`, `Copy`, `Debug`, `Eq`, `PartialEq`

### DefaultSwitchChanges

```rust
pub struct DefaultSwitchChanges {
    pub manage_members: Option<bool>,
    pub set_patron_limits: Option<bool>,
    pub library_scan: Option<bool>,
    pub run_jobs: Option<bool>,
    pub fix_catalog: Option<bool>,
    pub invite_branches: Option<bool>,
    pub approve_requests: Option<bool>,
}
```

Changes to some of the library's default switches: a new value for
each named switch; an absent switch keeps its stored value.

**Fields**

- `manage_members`: New default for [`AdminControl::ManageMembers`](#admincontrol).
- `set_patron_limits`: New default for [`AdminControl::SetPatronLimits`](#admincontrol).
- `library_scan`: New default for [`AdminControl::LibraryScan`](#admincontrol).
- `run_jobs`: New default for [`AdminControl::RunJobs`](#admincontrol).
- `fix_catalog`: New default for [`AdminControl::FixCatalog`](#admincontrol).
- `invite_branches`: New default for [`AdminControl::InviteBranches`](#admincontrol).
- `approve_requests`: New default for [`AdminControl::ApproveRequests`](#admincontrol).

#### DefaultSwitchChanges::get

```rust
pub fn get(&self, control: AdminControl) -> Option<bool>
```

The change named for `control`, if any.

#### DefaultSwitchChanges::set

```rust
pub fn set(&mut self, control: AdminControl, value: Option<bool>)
```

Names (or, with `None`, stops naming) a change for `control`.

#### DefaultSwitchChanges::with

```rust
pub fn with(self, control: AdminControl, value: bool) -> Self
```

`control` changed to `value`, for building a change.

Implements: `Clone`, `ComposeSchema`, `Copy`, `Debug`, `Default`, `Deserialize<'de>`, `Eq`, `PartialEq`, `Serialize`, `ToSchema`

### OverrideSwitchChanges

```rust
pub struct OverrideSwitchChanges {
    pub manage_members: Option<AdminSwitchChoice>,
    pub set_patron_limits: Option<AdminSwitchChoice>,
    pub library_scan: Option<AdminSwitchChoice>,
    pub run_jobs: Option<AdminSwitchChoice>,
    pub fix_catalog: Option<AdminSwitchChoice>,
    pub invite_branches: Option<AdminSwitchChoice>,
    pub approve_requests: Option<AdminSwitchChoice>,
}
```

Changes to some of one Administrator's overrides: `on` or `off` for
them alone, or `default` to follow the library's default again; an
absent switch keeps what it had.

**Fields**

- `manage_members`: Change to [`AdminControl::ManageMembers`](#admincontrol).
- `set_patron_limits`: Change to [`AdminControl::SetPatronLimits`](#admincontrol).
- `library_scan`: Change to [`AdminControl::LibraryScan`](#admincontrol).
- `run_jobs`: Change to [`AdminControl::RunJobs`](#admincontrol).
- `fix_catalog`: Change to [`AdminControl::FixCatalog`](#admincontrol).
- `invite_branches`: Change to [`AdminControl::InviteBranches`](#admincontrol).
- `approve_requests`: Change to [`AdminControl::ApproveRequests`](#admincontrol).

#### OverrideSwitchChanges::get

```rust
pub fn get(&self, control: AdminControl) -> Option<AdminSwitchChoice>
```

The change named for `control`, if any.

#### OverrideSwitchChanges::set

```rust
pub fn set(&mut self, control: AdminControl, value: Option<AdminSwitchChoice>)
```

Names (or, with `None`, stops naming) a change for `control`.

#### OverrideSwitchChanges::with

```rust
pub fn with(self, control: AdminControl, value: AdminSwitchChoice) -> Self
```

`control` changed to `value`, for building a change.

Implements: `Clone`, `ComposeSchema`, `Copy`, `Debug`, `Default`, `Deserialize<'de>`, `Eq`, `PartialEq`, `Serialize`, `ToSchema`

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
    pub updated_at: String,
}
```

One title a branch shares, as the owner's Settings → Branches lists it.

**Fields**

- `uuid`: The node holding's id (a catalog upload's `holding_id`).
- `provenance`: `node:<node_id>`.
- `node_id`: The branch's node id.
- `owner_user_id`: The user it belongs to here (the branch's owner).
- `store`: Store id.
- `product_id`: Product id.
- `title`: Display title.
- `authors`: Authors.
- `primary_key`: Storage key of the primary audio file.
- `delivered`: True once the current manifest's files arrived.
- `updated_at`: RFC 3339 last change.

Implements: `Clone`, `Debug`, `Eq`, `PartialEq`, `Serialize`

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
- `detached`: Editions whose files it delivered that nothing in this library holds
  any more (it stopped sharing, left, or withdrew them, and no store
  account or other branch here holds them).

Implements: `Clone`, `ComposeSchema`, `Copy`, `Debug`, `Default`, `Eq`, `PartialEq`, `Serialize`, `ToSchema`

### CatalogSessionOpened

```rust
pub struct CatalogSessionOpened {
    pub fence: i64,
    pub cursor: String,
    pub wanted: Vec<CatalogRef>,
    pub members_books: bool,
}
```

An opened catalog session ([`begin_catalog`](#begin_catalog)).

**Fields**

- `fence`: The new catalog fence.
- `cursor`: Where the branch resumes (empty: from the start).
- `wanted`: Titles recorded earlier whose files have not arrived.
- `members_books`: This library's "Acquire all of this branch's members' books"
  ([`super::set_branch_members_books`](#set_branch_members_books)).

Implements: `Clone`, `Debug`, `Eq`, `PartialEq`

### ForwardableBranchKey

```rust
pub struct ForwardableBranchKey {
    pub id: String,
    pub store: String,
    pub product_id: String,
    pub updated_at: String,
    pub delivered: bool,
}
```

The identity of one title a branch shares with this install, without
its metadata or file manifest.

Read in keyset pages by [`forwardable_branch_title_keys_page`](#forwardable_branch_title_keys_page); the full
title is loaded only when it is needed, by [`forwardable_branch_title`](#forwardable_branch_title).

**Fields**

- `id`: The node holding's id; the keyset cursor of the page reader.
- `store`: Store id (`audible`, `libro`, ...).
- `product_id`: Store product id.
- `updated_at`: RFC 3339 last change, as stored.
- `delivered`: The files of the current manifest arrived here, so the title may be
  passed on. Rows without it are returned so a page's length still
  tells the caller whether more rows follow.

Implements: `Clone`, `Debug`, `Eq`, `PartialEq`

### ForwardableBranchTitle

```rust
pub struct ForwardableBranchTitle {
    pub id: String,
    pub title: CatalogTitle,
    pub updated_at: String,
}
```

A title one of this install's branches shares, ready to pass on.

**Fields**

- `id`: The node holding's id.
- `title`: The title as the branch shared it.
- `updated_at`: RFC 3339 last change.

Implements: `Clone`, `Debug`, `Eq`, `PartialEq`

### MembersBooksBody

```rust
pub struct MembersBooksBody {
    /* private fields */
}
```

`PUT /api/config/branches/members-books` body: a branch, and whether
this library acquires all of its members' books.

#### MembersBooksBody::validate

```rust
pub fn validate(self) -> Result<MembersBooksChoice>
```

Checks the body: a well-formed node id, with surrounding whitespace
removed.

##### Errors

Returns [`LibraryError::Invalid`](README.md#libraryerror) for a malformed node id.

Implements: `ComposeSchema`, `Debug`, `Deserialize<'de>`, `ToSchema`

### MembersBooksChoice

```rust
pub struct MembersBooksChoice {
    pub node_id: String,
    pub acquire: bool,
}
```

A validated [`MembersBooksBody`](#membersbooksbody).

**Fields**

- `node_id`: The branch's node id, trimmed: a valid node id.
- `acquire`: The new choice.

Implements: `Clone`, `Debug`, `Eq`, `PartialEq`

### BranchRequestAsk

```rust
pub struct BranchRequestAsk {
    pub work_id: String,
    pub node_id: Option<String>,
}
```

A validated [`BranchRequestBody`]: both ids trimmed and well formed.

**Fields**

- `work_id`: The work id: 1 to 64 ASCII letters, digits and `-`.
- `node_id`: The branch's node id, when the body named one: a valid node id.

Implements: `Clone`, `Debug`, `Eq`, `PartialEq`

### BranchRequestBody

```rust
pub struct BranchRequestBody {
    /* private fields */
}
```

`POST /api/branch-requests` body: the work to ask a branch for, and
optionally which branch.

#### BranchRequestBody::validate

```rust
pub fn validate(self) -> Result<BranchRequestAsk>
```

Checks the body: a well-formed work id and, when present, node id
(each with surrounding whitespace removed).

##### Errors

Returns [`LibraryError::Invalid`](README.md#libraryerror) for a malformed work or node id.

Implements: `ComposeSchema`, `Debug`, `Deserialize<'de>`, `ToSchema`

### BranchRequestPlan

```rust
pub struct BranchRequestPlan {
    pub node_id: String,
    pub payload: JobPayload,
}
```

The job a request for `work_id` becomes: which branch it goes to and
what it asks for.

**Fields**

- `node_id`: The branch node the job is pinned to.
- `payload`: The job's payload ([`JobKind::BranchRequest`](README.md#jobkind)).

Implements: `Clone`, `Debug`, `Eq`, `PartialEq`

### CacheSettingsV1

```rust
pub struct CacheSettingsV1 {
    pub budget_bytes: u64,
    pub unviewed_days: u32,
}
```

Typed `core.cache` document, schema version 1.

**Fields**

- `budget_bytes`: Disk space, in bytes, each node's image caches may use together
  unless the node overrides it (16 MiB to 1 TiB).
- `unviewed_days`: Days an entry may go unviewed before the sweep evicts it (1 to 3650).

#### CacheSettingsV1::validate

```rust
pub fn validate(&self) -> Result<()>
```

Rejects values outside the domain bounds.

##### Errors

Returns [`LibraryError::Invalid`](README.md#libraryerror) when a field is out of range.

#### CacheSettingsV1::get

```rust
pub fn get(&self, key: &str) -> Option<String>
```

The value of one settings key, as text.

##### Arguments

* `key` - One of [`CACHE_SETTING_KEYS`](#cache_setting_keys).

##### Returns

The value, or `None` for a key this document does not own.

#### CacheSettingsV1::set

```rust
pub fn set(&mut self, key: &str, value: &str) -> Result<()>
```

Parses `value` into the field `key` names and validates the result.

##### Arguments

* `key` - One of [`CACHE_SETTING_KEYS`](#cache_setting_keys).
* `value` - The new value as text: a whole number.

##### Errors

Returns [`LibraryError::Invalid`](README.md#libraryerror) for an unknown key, a value that is
not a whole number, or one outside the field's range.

#### CacheSettingsV1::budget_for

```rust
pub fn budget_for(&self, node: &NodeRuntimeSettingsV1) -> u64
```

The budget that applies on a node: its own override, else the
library budget.

##### Arguments

* `node` - The node's runtime document.

Implements: `Clone`, `ComposeSchema`, `Debug`, `Default`, `Deserialize<'de>`, `Eq`, `PartialEq`, `Serialize`, `ToSchema`

### CarriedImport

```rust
pub struct CarriedImport {
    pub imported: Vec<(String, String)>,
    pub already_here: Vec<(String, String)>,
    pub unrecorded: Vec<(String, String)>,
    pub taken: Vec<(String, String)>,
    pub failed: Vec<(String, String)>,
    pub refused_by_rule: bool,
}
```

What [`import_carried_accounts`](#import_carried_accounts) did with each account it was handed.

**Fields**

- `imported`: `(plugin_key, account_id)` of each account now connected here for
  the person.
- `already_here`: Accounts the person had already connected here; left as they are.
- `unrecorded`: Of [`Self::already_here`](#carriedimport), the accounts whose record that this peer
  holds them too could not be written (logged): they stay connected
  here, but a disconnect everywhere from here would not reach that
  peer until the same offer arrives again and records it.
- `taken`: Accounts connected to someone else here; refused.
- `failed`: Accounts that came without a secret, or whose write failed (logged);
  nothing of them was stored.
- `refused_by_rule`: This instance's operator does not let the person connect store
  accounts here; nothing was stored. A rule turned off while the
  import runs instead stops it: the accounts not yet kept land in
  [`Self::failed`](#carriedimport).

Implements: `Clone`, `Debug`, `Default`, `Eq`, `PartialEq`

### MemberAccount

```rust
pub struct MemberAccount {
    pub plugin_key: String,
    pub account_id: String,
    pub source: String,
    pub marketplace: String,
    pub label: Option<String>,
}
```

One store account a member connected here, as the join consent page
lists it. Carries no secret.

**Fields**

- `plugin_key`: Provenance-qualified plugin key of the storefront.
- `account_id`: Store account id.
- `source`: Display alias of the storefront.
- `marketplace`: Store marketplace or locale code.
- `label`: The account's display label, when it has one.

Implements: `Clone`, `Debug`, `Eq`, `PartialEq`

### AnswerReport

```rust
pub struct AnswerReport {
    pub works: u64,
    pub listings: u64,
    pub dropped: u64,
    pub expired: u64,
}
```

What [`record_answers`](#record_answers) stored.

**Fields**

- `works`: Works the answer settled (answered or "not here").
- `listings`: Branch listings stored or refreshed.
- `dropped`: Editions dropped: for a work not asked of this branch, or one the
  resolver did not place on a stored edition of the asked work.
- `expired`: Earlier branch listings the answer no longer names, now expired.

Implements: `Clone`, `Debug`, `Default`, `Eq`, `PartialEq`

### AskReport

```rust
pub struct AskReport {
    pub branches: usize,
    pub asked: u64,
}
```

What [`ask_branches`](#ask_branches) did.

**Fields**

- `branches`: Branches asked.
- `asked`: Questions added or asked again, over every branch.

Implements: `Clone`, `Debug`, `Default`, `Eq`, `PartialEq`

### BranchListing

```rust
pub struct BranchListing {
    pub listing_id: String,
    pub node_id: String,
    pub work_id: String,
    pub store: String,
    pub marketplace: String,
    pub product_id: String,
    pub observed_at: DateTime<Utc>,
}
```

A `branch:<node>` listing and the work its edition belongs to.

**Fields**

- `listing_id`: `store_listings.id`.
- `node_id`: The branch node it is available through.
- `work_id`: The work it supplies.
- `store`: The store the branch has it from (`audible`).
- `marketplace`: Lower-cased marketplace of the listing; empty when the store has
  none.
- `product_id`: The store's product id.
- `observed_at`: When the branch last named it.

Implements: `Clone`, `Debug`, `Eq`, `PartialEq`

### BranchOffer

```rust
pub struct BranchOffer {
    pub node_id: String,
    pub owner_name: Option<String>,
    pub store: String,
    pub product_id: String,
}
```

One branch a work is available through, as Discover and search show it.

**Fields**

- `node_id`: The branch node.
- `owner_name`: The branch owner's display name (or login name), for "available
  through Riley's branch"; `None` when the owner has neither.
- `store`: The store the branch has the book from (`audible`).
- `product_id`: The store's product id.

Implements: `Clone`, `Debug`, `Eq`, `PartialEq`

### ClerkLink

```rust
pub struct ClerkLink {
    pub library_id: String,
    pub origin: String,
    pub spki_pin: Option<String>,
    pub library_name: String,
    pub remote_user_id: i64,
    pub display_name: Option<String>,
    pub clerk_name: String,
    pub state: ClerkLinkState,
    pub created_at: String,
    pub updated_at: String,
}
```

One library this install holds (or held) a library card for.

**Fields**

- `library_id`: The library id.
- `origin`: The library's origin as [`bookclerk_node_proto::canonical_origin`](../bookclerk_node_proto/README.md#canonical_origin)
  spells it.
- `spki_pin`: SPKI pin of the library's self-signed certificate, when the person
  confirmed one.
- `library_name`: The library's name at sign-in.
- `remote_user_id`: The person's user id on the library.
- `display_name`: The person's display name on the library, when it has one.
- `clerk_name`: The name this install signed in as.
- `state`: Whether the card is still held.
- `created_at`: RFC 3339 sign-in time.
- `updated_at`: RFC 3339 time of the last change.

#### ClerkLink::is_active

```rust
pub fn is_active(&self) -> bool
```

True while the clerk reader should dial this library.

Implements: `Clone`, `Debug`, `Eq`, `From<Model>`, `PartialEq`, `Serialize`

### LibraryCard

```rust
pub struct LibraryCard(/* private */);
```

A library card in plain form: zeroized on drop, redacted in `Debug`.

#### LibraryCard::new

```rust
pub fn new(token: String) -> Result<Self>
```

Wraps a card a library issued.

##### Errors

Returns [`LibraryError::Invalid`](README.md#libraryerror) when `token` is not a card
([`bookclerk_node_proto::is_clerk_token`](../bookclerk_node_proto/README.md#is_clerk_token)).

#### LibraryCard::expose

```rust
pub fn expose(&self) -> &str
```

The card, for the `Authorization: Bookclerk-Clerk` header.

Implements: `Clone`, `Debug`

### NewClerkLink

```rust
pub struct NewClerkLink {
    pub library_id: String,
    pub origin: String,
    pub spki_pin: Option<String>,
    pub library_name: String,
    pub remote_user_id: i64,
    pub display_name: Option<String>,
    pub clerk_name: String,
}
```

What a sign-in returned, to record with [`record_clerk_link`](#record_clerk_link).

**Fields**

- `library_id`: The library id, as both its info and its token answer
  named it.
- `origin`: The origin the sign-in went to.
- `spki_pin`: SPKI pin the person confirmed, for a self-signed library.
- `library_name`: The library's name from its info.
- `remote_user_id`: The person's user id from the token answer.
- `display_name`: The person's display name from the token answer.
- `clerk_name`: The name this install signed in as.

Implements: `Clone`, `Debug`, `Eq`, `PartialEq`

### LibraryCollection

```rust
pub struct LibraryCollection {
    pub name: String,
    pub prefixes: Vec<String>,
}
```

One collection: a name and the storage key prefixes whose files belong
to it.

**Fields**

- `name`: The name people see and scope rules match, 1 to 64 bytes, distinct
  ignoring case.
- `prefixes`: Storage key prefixes, 1 to 8, each a folder relative to the
  destination's root (no leading `/`); a closing `/` is added when
  missing, so `kids` matches `kids/…` but not `kidsbooks/…`.

Implements: `Clone`, `ComposeSchema`, `Debug`, `Deserialize<'de>`, `Eq`, `PartialEq`, `Serialize`, `ToSchema`

### LibraryCollections

```rust
pub struct LibraryCollections {
    pub collections: Vec<LibraryCollection>,
}
```

The library's collections: the `core.collections` document and the
body of `PUT /api/library/collections`.

**Fields**

- `collections`: The collections, at most 32, in the order shown.

#### LibraryCollections::validate

```rust
pub fn validate(self) -> Result<Self>
```

Checks and normalizes the collections: names trimmed and distinct
ignoring case, prefixes trimmed with a closing `/` and distinct
within a collection, every bound held.

##### Errors

Returns [`LibraryError::Invalid`](README.md#libraryerror) naming the first bound broken.

#### LibraryCollections::names_for_keys

```rust
pub fn names_for_keys<'k>(
    &self,
    keys: impl IntoIterator<Item = &'k str> + Clone,
) -> Vec<&str>
```

The names of the collections any of `keys` belongs to, in the
collections' order.

##### Arguments

* `keys` - Storage keys of a book's files.

Implements: `Clone`, `ComposeSchema`, `Debug`, `Default`, `Deserialize<'de>`, `Eq`, `PartialEq`, `Serialize`, `ToSchema`

### AccountRef

```rust
pub struct AccountRef {
    pub plugin_key: String,
    pub account_id: String,
}
```

One store account, by storefront plugin key and store account id.

**Fields**

- `plugin_key`: Provenance-qualified plugin key of the storefront, as sent.
- `account_id`: Store account id, as sent.

Implements: `Clone`, `ComposeSchema`, `Debug`, `Deserialize<'de>`, `Eq`, `PartialEq`, `ToSchema`

### AddConnectionsBody

```rust
pub struct AddConnectionsBody {
    /* private fields */
}
```

`POST /api/config/join/connections` body.

#### AddConnectionsBody::validate

```rust
pub fn validate(self) -> Result<AddConnectionsRequest>
```

Checks the account count and returns the request.

##### Errors

Returns [`LibraryError::Invalid`](README.md#libraryerror) when the body names no account or
more than [`MAX_ADD_CONNECTIONS`](#max_add_connections).

Implements: `ComposeSchema`, `Debug`, `Deserialize<'de>`, `ToSchema`

### AddConnectionsRequest

```rust
pub struct AddConnectionsRequest {
    pub library_id: String,
    pub accounts: Vec<AccountRef>,
}
```

An "Add my connections" request: the library it names (trimmed) and the
accounts picked, between 1 and [`MAX_ADD_CONNECTIONS`](#max_add_connections) of them.

**Fields**

- `library_id`: The library (parent library id), surrounding whitespace removed.
- `accounts`: The accounts picked, in the order sent.

Implements: `Clone`, `Debug`, `Eq`, `PartialEq`

### DisconnectBody

```rust
pub struct DisconnectBody {
    /* private fields */
}
```

`POST /api/config/join/connections/disconnect` body.

#### DisconnectBody::into_request

```rust
pub fn into_request(self) -> DisconnectRequest
```

The request the body makes.

Implements: `ComposeSchema`, `Debug`, `Deserialize<'de>`, `ToSchema`

### DisconnectRequest

```rust
pub struct DisconnectRequest {
    pub account: AccountRef,
    pub everywhere: bool,
}
```

A disconnect request: the account and whether to reach every linked
library it was shared with.

**Fields**

- `account`: The account, as sent.
- `everywhere`: Also disconnect it in every linked library it was shared with.

Implements: `Clone`, `Debug`, `Eq`, `PartialEq`

### CopyDownload

```rust
pub struct CopyDownload {
    pub serving_node_id: String,
    pub destination: String,
    pub files: Vec<CopyPlanFile>,
}
```

What a download request may read ([`authorize_copy`](#authorize_copy)).

**Fields**

- `serving_node_id`: The node that granted the copy and serves its files.
- `destination`: The destination the files are read from.
- `files`: The files, in download order.

Implements: `Clone`, `Debug`, `Eq`, `PartialEq`

### CopyListing

```rust
pub struct CopyListing<'a> {
    pub store: &'a str,
    pub marketplace: &'a str,
    pub product_id: &'a str,
}
```

The store listing a reader names an edition by.

**Fields**

- `store`: Store id as the library's page states it (the store's alias).
- `marketplace`: Lowercase marketplace; empty for a store without marketplaces.
- `product_id`: The store's product id.

Implements: `Clone`, `Copy`, `Debug`, `Eq`, `PartialEq`

### CopyPlan

```rust
pub struct CopyPlan {
    pub person: i64,
    pub edition_id: String,
    pub copy: WindowCopy,
    pub destinations: BTreeMap<String, Vec<CopyPlanFile>>,
    /* private fields */
}
```

The copy of one edition a holder may take.

**Fields**

- `person`: The person the copy is for.
- `edition_id`: The edition.
- `copy`: What the copy is, as the library's page states it.
- `destinations`: The edition's files on each destination that holds its primary
  audio, by destination id; each list has the primary audio first,
  then the rest in key order. A file's place in its list is its
  download index.

Implements: `Clone`, `Debug`, `Eq`, `PartialEq`

### CopyPlanFile

```rust
pub struct CopyPlanFile {
    pub key: String,
    pub file: CopyFile,
}
```

One file of a planned copy, on one destination.

**Fields**

- `key`: Storage key on the destination.
- `file`: The file as the answer frame lists it, its size not yet known
  (zero): the daemon reads sizes from the destination.

Implements: `Clone`, `Debug`, `Eq`, `PartialEq`

### OpenedCopy

```rust
pub struct OpenedCopy {
    pub copy_id: String,
    pub token: Zeroizing<String>,
    pub expires_in_secs: u64,
}
```

A grant just opened by [`open_copy`](#open_copy).

**Fields**

- `copy_id`: The copy id.
- `token`: The bearer token; shown to the holder once and never stored.
- `expires_in_secs`: Seconds the token is accepted.

Implements: `Debug`, `Drop`

### PlaybackPlan

```rust
pub struct PlaybackPlan {
    pub person: i64,
    pub edition_id: String,
    pub destinations: BTreeMap<String, Vec<PlaybackTrack>>,
    /* private fields */
}
```

What a person may play of one edition ([`plan_playback`](#plan_playback)).

**Fields**

- `person`: The listener.
- `edition_id`: The edition.
- `destinations`: The edition's tracks on each destination that holds its primary
  audio, by destination id: the primary audio first, then its other
  parts in key order. A track's place in its list is its stream
  index. Chapter files of a title also stored whole are left out,
  since they repeat the primary audio.

#### PlaybackPlan::digest

```rust
pub fn digest(&self, destination: &str) -> Option<&str>
```

The digest of the edition's file rows on `destination`, or `None`
when the destination does not hold the edition.

Implements: `Clone`, `Debug`, `Eq`, `PartialEq`

### PlaybackTrack

```rust
pub struct PlaybackTrack {
    pub key: String,
    pub content_type: Option<String>,
    pub bitrate_kbps: Option<u32>,
    pub lossless: bool,
}
```

One audio file a player streams for an edition, on one destination.

**Fields**

- `key`: Storage key on the destination.
- `content_type`: Media type of the file, once probed.
- `bitrate_kbps`: Average bitrate in kilobits per second, once probed.
- `lossless`: True for a lossless encoding; false also for a file not probed.

Implements: `Clone`, `Debug`, `Eq`, `PartialEq`

### CronCursor

```rust
pub struct CronCursor {
    pub plugin_instance_id: String,
    pub cron: String,
    pub last_fire_ms: i64,
}
```

One `plugin_cron_cursors` row.

**Fields**

- `plugin_instance_id`: Plugin instance whose schedule this is.
- `cron`: Normalized cron source.
- `last_fire_ms`: Last claimed (or, for a fresh cursor, last skipped) fire time in Unix
  milliseconds.

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

### DestinationPlacementV1

```rust
pub struct DestinationPlacementV1 {
    pub filter: DestinationFilter,
}
```

The body of a `core.destination` document.

**Fields**

- `filter`: Which books the destination takes; no rules take every book.

Implements: `Clone`, `Debug`, `Default`, `Eq`, `PartialEq`, `Serialize`

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

- `scope_type`: `library`, `node`, or `plugin_instance`.
- `scope_id`: `singleton`, a node id, or a plugin instance id.
- `namespace`: Domain namespace.

#### DocumentKey::library

```rust
pub fn library(namespace: &str) -> Self
```

Library-scoped domain.

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

### VerifiedLibraryOwner

```rust
pub struct VerifiedLibraryOwner {
    /* private fields */
}
```

Proof that a user was an active Owner when the write was authorized.

Only [`super::verify_library_owner`](#verify_library_owner) constructs one.

#### VerifiedLibraryOwner::user_id

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

### VerifiedPatronAdministrator

```rust
pub struct VerifiedPatronAdministrator {
    /* private fields */
}
```

Proof that a user was an active Owner or Administrator when a write of
the library's patron defaults was authorized.

Only [`super::verify_patron_administrator`](#verify_patron_administrator) constructs one.

#### VerifiedPatronAdministrator::user_id

```rust
pub fn user_id(&self) -> i64
```

The user's id.

Implements: `Clone`, `Debug`, `Eq`, `PartialEq`

### DownloadRequestAsk

```rust
pub struct DownloadRequestAsk {
    pub work_id: String,
}
```

A validated [`DownloadRequestBody`]: the work id trimmed and well
formed.

**Fields**

- `work_id`: The work id: 1 to 64 ASCII letters, digits and `-`.

Implements: `Clone`, `Debug`, `Eq`, `PartialEq`

### DownloadRequestBody

```rust
pub struct DownloadRequestBody {
    /* private fields */
}
```

`POST /api/download-requests` body: the work to download.

#### DownloadRequestBody::validate

```rust
pub fn validate(self) -> Result<DownloadRequestAsk>
```

Checks the body: a well-formed work id (with surrounding whitespace
removed).

##### Errors

Returns [`LibraryError::Invalid`](README.md#libraryerror) for a malformed work id.

Implements: `ComposeSchema`, `Debug`, `Deserialize<'de>`, `ToSchema`

### DownloadRequestPlan

```rust
pub struct DownloadRequestPlan {
    pub edition_id: String,
    pub title: String,
}
```

What a download request for a work fetches.

**Fields**

- `edition_id`: The edition to download (`editions.id`).
- `title`: The acquire job's title: one account holding of that edition
  (`holdings.id`).

Implements: `Clone`, `Debug`, `Eq`, `PartialEq`

### BranchEnrollmentSettings

```rust
pub struct BranchEnrollmentSettings {
    pub body: BranchEnrollmentSettingsV1,
    pub revision: Option<i64>,
}
```

The branch-enrollment switch as stored, or its default when no document
was ever written.

**Fields**

- `body`: Effective settings.
- `revision`: Stored revision; `None` while the default applies.

Implements: `Clone`, `Debug`, `Eq`, `PartialEq`

### BranchEnrollmentSettingsV1

```rust
pub struct BranchEnrollmentSettingsV1 {
    pub enabled: bool,
}
```

Typed `core.branch_enrollment` document, schema version 1.

**Fields**

- `enabled`: When false, no link may be minted or redeemed.

Implements: `Clone`, `Debug`, `Default`, `Deserialize<'de>`, `Eq`, `PartialEq`, `Serialize`

### EnrolledNode

```rust
pub struct EnrolledNode {
    pub library_id: String,
    pub node_id: String,
    pub owner_user_id: i64,
    pub replayed: bool,
}
```

Result of a successful redeem.

**Fields**

- `library_id`: Library id.
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

A branch node's stored credential.

**Fields**

- `node_id`: Branch node.
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
    pub library_id: Option<String>,
}
```

Bootstrap identity loaded from [`NODE_IDENTITY_FILE`](#node_identity_file).

**Fields**

- `node_id`: Stable node id.
- `library_id`: Library binding, once enrollment has succeeded.

Implements: `Clone`, `Debug`, `Eq`, `PartialEq`

### NodeRecord

```rust
pub struct NodeRecord {
    pub node_id: String,
    pub library_id: String,
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
    pub acquire_members_books: bool,
}
```

Durable node row plus the latest observation columns.

**Fields**

- `node_id`: Stable node id.
- `library_id`: Library id stored with the node.
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
- `kind`: How the node joined the library.
- `owner_user_id`: User who owns this node and may administer it, when one is assigned.
- `share_books`: For a branch: "Share this branch's books" as it last reported it;
  empty until it does.
- `include_branches`: For a branch: "Include books from my own branches", as reported.
- `acquire_members_books`: For a branch: this library's "Acquire all of this branch's members'
  books" ([`super::set_branch_members_books`](#set_branch_members_books)).

Implements: `Clone`, `Debug`, `Eq`, `PartialEq`

### ExpandedEntry

```rust
pub struct ExpandedEntry {
    pub path: String,
    pub size: u64,
    pub outcome: MemberOutcome,
    pub reason: Option<String>,
}
```

One entry an archive member expanded into.

**Fields**

- `path`: Its path in the batch, or the sanitized name of a refused entry.
- `size`: Bytes written (0 for a refused entry).
- `outcome`: `Pending` for a written entry, `Refused` with a reason otherwise.
- `reason`: Why it was refused.

Implements: `Clone`, `Debug`, `Eq`, `PartialEq`

### IngestBatch

```rust
pub struct IngestBatch {
    pub id: IngestBatchId,
    pub user_id: i64,
    pub node_id: String,
    pub source: IngestSource,
    pub intake_instance: Option<String>,
    pub state: IngestState,
    pub file_count: u64,
    pub declared_bytes: u64,
    pub job_id: Option<String>,
    pub detail: Option<String>,
    pub created_at: DateTime<Utc>,
    pub updated_at: DateTime<Utc>,
    pub finished_at: Option<DateTime<Utc>>,
}
```

One stored batch.

**Fields**

- `id`: The batch id.
- `user_id`: The patron who holds its books.
- `node_id`: The node that holds its bytes and runs it.
- `source`: Upload or intake.
- `intake_instance`: For an intake batch, the storage instance it reads.
- `state`: Its state.
- `file_count`: Files the manifest named.
- `declared_bytes`: Bytes the manifest declared.
- `job_id`: Its job, once committed.
- `detail`: Why it failed.
- `created_at`: Creation time.
- `updated_at`: Last activity.
- `finished_at`: When it reached a final state.

Implements: `Clone`, `Debug`, `Eq`, `PartialEq`

### IngestBatchId

```rust
pub struct IngestBatchId(/* private */);
```

A batch id: 32 lower-case hex digits, safe as one path segment.

#### IngestBatchId::new

```rust
pub fn new() -> Self
```

A new random id.

#### IngestBatchId::parse

```rust
pub fn parse(raw: &str) -> Option<Self>
```

Parses an id, refusing anything but 32 lower-case hex digits.

#### IngestBatchId::as_str

```rust
pub fn as_str(&self) -> &str
```

The id as stored.

Implements: `Clone`, `Debug`, `Default`, `Deserialize<'de>`, `Display`, `Eq`, `Hash`, `Ord`, `PartialEq`, `PartialOrd`, `Serialize`

### IngestMember

```rust
pub struct IngestMember {
    pub member_no: u32,
    pub parent_no: Option<u32>,
    pub path: String,
    pub size: u64,
    pub sha256: Option<String>,
    pub received: u64,
    pub storage_key: Option<String>,
    pub signature: Option<String>,
    pub outcome: MemberOutcome,
    pub reason: Option<String>,
    pub edition_id: Option<String>,
}
```

One stored member.

**Fields**

- `member_no`: Its position in the batch.
- `parent_no`: The archive member it was expanded from.
- `path`: Its relative path (or a refused entry's sanitized name).
- `size`: Its size in bytes.
- `sha256`: The digest the manifest declared.
- `received`: Bytes staged so far.
- `storage_key`: For an intake member, its key.
- `signature`: For an intake member, its listed signature.
- `outcome`: What became of it.
- `reason`: Why it was refused or failed.
- `edition_id`: The edition it was stored under.

#### IngestMember::complete

```rust
pub fn complete(&self) -> bool
```

True once every declared byte is staged.

Implements: `Clone`, `Debug`, `Eq`, `PartialEq`

### IntakeRef

```rust
pub struct IntakeRef {
    pub key: String,
    pub signature: Option<String>,
}
```

An intake member's place in its storage instance.

**Fields**

- `key`: The key in the intake instance.
- `signature`: The signature the key was listed with, when the backend gives one.

Implements: `Clone`, `Debug`, `Eq`, `PartialEq`

### ManifestFile

```rust
pub struct ManifestFile {
    pub path: String,
    pub size: u64,
    pub sha256: Option<String>,
}
```

One file of a `POST /api/uploads` manifest, as sent.

**Fields**

- `path`: Relative path, folders kept (`Book/01 Chapter.mp3`); `\` reads as
  `/`. Refused when absolute, when it climbs out with `..`, or when it
  is too long or too deep.
- `size`: Size in bytes.
- `sha256`: Lower-case hex SHA-256 of the file, checked before it is stored.

Implements: `Clone`, `ComposeSchema`, `Debug`, `Deserialize<'de>`, `ToSchema`

### MemberSpec

```rust
pub struct MemberSpec {
    pub path: EntryPath,
    pub size: u64,
    pub sha256: Option<Sha256Hex>,
    pub intake: Option<IntakeRef>,
}
```

One file a manifest names.

**Fields**

- `path`: Its relative path in the batch.
- `size`: Its size in bytes.
- `sha256`: Its digest, when the client declared one.
- `intake`: For an intake batch, where the file is; `None` for an upload.

Implements: `Clone`, `Debug`, `Eq`, `PartialEq`

### OutcomeUpdate

```rust
pub struct OutcomeUpdate {
    pub member_no: u32,
    pub outcome: MemberOutcome,
    pub reason: Option<String>,
    pub edition_id: Option<String>,
}
```

One member's outcome, as the job decided it.

**Fields**

- `member_no`: The member.
- `outcome`: Its outcome.
- `reason`: The reason for a refusal or failure, cut to [`MAX_REASON_BYTES`](#max_reason_bytes).
- `edition_id`: The edition it was stored under.

Implements: `Clone`, `Debug`, `Eq`, `PartialEq`

### ReviewItem

```rust
pub struct ReviewItem {
    pub batch_id: IngestBatchId,
    pub member_no: u32,
    pub path: String,
    pub edition_id: Option<String>,
    pub user_id: i64,
}
```

One stored book an Administrator should review: a member stored under
a new local edition because nothing strong named one.

**Fields**

- `batch_id`: The batch.
- `member_no`: The member.
- `path`: Its path in the batch.
- `edition_id`: The local edition it was stored under.
- `user_id`: The patron who uploaded it.

Implements: `Clone`, `Debug`, `Eq`, `PartialEq`

### Sha256Hex

```rust
pub struct Sha256Hex(/* private */);
```

A lower-case hex SHA-256 digest a manifest declared for a file.

#### Sha256Hex::parse

```rust
pub fn parse(raw: &str) -> Option<Self>
```

Parses 64 hex digits, folding them to lower case.

#### Sha256Hex::as_str

```rust
pub fn as_str(&self) -> &str
```

The digest as stored.

Implements: `Clone`, `Debug`, `Deserialize<'de>`, `Eq`, `Hash`, `PartialEq`, `Serialize`

### UploadManifest

```rust
pub struct UploadManifest {
    pub files: Vec<ManifestFile>,
}
```

`POST /api/uploads` body: the files of one batch.

**Fields**

- `files`: Every file of the batch, in the order their numbers follow.

Implements: `Clone`, `ComposeSchema`, `Debug`, `Deserialize<'de>`, `ToSchema`

### UploadQuota

```rust
pub struct UploadQuota {
    pub max_file_bytes: u64,
    pub max_staged_bytes: u64,
}
```

The upload limits a batch is admitted under (`uploads.*` settings).

**Fields**

- `max_file_bytes`: Largest file, in bytes.
- `max_staged_bytes`: Most bytes one person may have staged in live upload batches.

Implements: `Clone`, `Copy`, `Debug`, `Eq`, `PartialEq`

### IngressSettings

```rust
pub struct IngressSettings {
    pub active_instance: Option<String>,
    pub exposure: IngressExposure,
    pub origin: Option<String>,
    pub reported_origin: Option<String>,
}
```

The `core.ingress` document.

**Fields**

- `active_instance`: The plugin instance serving the library's public address, as its
  hyphenated UUID; `None` when ingress is off.
- `exposure`: What public traffic may reach.
- `origin`: The adopted public origin, `https://<host>`; `None` until the active
  instance first reports one.
- `reported_origin`: The last origin the active instance reported when it differs from
  [`IngressSettings::origin`](#ingresssettings); waits for the operator to accept it.

#### IngressSettings::active

```rust
pub fn active(&self) -> Option<PluginInstanceId>
```

The active instance, parsed.

##### Returns

`None` when ingress is off or the stored id does not parse (a
document written by hand); either way no instance is active.

#### IngressSettings::serving_origin

```rust
pub fn serving_origin(&self) -> Option<&str>
```

The public origin to serve while ingress is on: the adopted origin of
the active instance, `None` when ingress is off.

Implements: `Clone`, `Debug`, `Default`, `Deserialize<'de>`, `Eq`, `PartialEq`, `Serialize`

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

Implements: `Clone`, `ComposeSchema`, `Debug`, `Deserialize<'de>`, `Eq`, `PartialEq`, `Serialize`, `ToSchema`

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
  entry's kind; secret refs whose key is gone or now plain. Empty when
  only table rows were brought to the current columns.
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
  be deployed only there. `None` for library instances.

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
    pub scope: ClaimScope,
}
```

What a Host is willing to claim: the job kinds it runs and the plugin
keys its operator consented to. Both must admit a job.

**Fields**

- `kinds`: Job kinds (`scan`, `acquire`, ...).
- `plugin_keys`: Canonical plugin keys.
- `scope`: Whose work the Host takes; it narrows what the node is trusted
  with ([`super::node_trust_in_scope`](#node_trust_in_scope)), never widens it.

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

Library-wide permit counts a routed claim enforces.

**Fields**

- `class`: Running jobs of the claimed resource class, library-wide.
- `plugin_instance`: Running jobs that require one plugin instance, library-wide.

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

#### JobClaimRequest::scope

```rust
pub fn scope(&self) -> ClaimScope
```

Whose work the claim takes: a Host's filter says
([`JobClaimFilter::scope`](#jobclaimfilter)); a claim without one takes every job
the node is trusted with.

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

### KeptShare

```rust
pub struct KeptShare {
    pub plugin_key: String,
    pub account_id: String,
    pub library_id: String,
    pub pushed: i64,
    pub observed: i64,
}
```

One account this install shares with a library it joined, as the keeper
sees it.

**Fields**

- `plugin_key`: Provenance-qualified plugin key of the storefront.
- `account_id`: Store account id.
- `library_id`: The library id.
- `pushed`: The revision last sent to that library; 0 before the first push.
- `observed`: The [`SignIn::fingerprint`](#signin) of the sign-in last sent; 0 before the
  first push.

#### KeptShare::next_revision

```rust
pub fn next_revision(&self, sign_in: &SignIn) -> Option<i64>
```

The revision to send for `sign_in`, or `None` when that sign-in was
already sent. A sign-in counts as new whenever its fingerprint
differs from the one last sent, and each push carries a revision
above the last one sent (its write time, when that is higher), so
the library takes it even after the clock went back.

Implements: `Clone`, `Debug`, `Eq`, `PartialEq`

### SignIn

```rust
pub struct SignIn {
    pub at: i64,
    pub fingerprint: i64,
}
```

An account's current sign-in on its keeper ([`sign_in_revision`](#sign_in_revision)).

**Fields**

- `at`: Milliseconds since the Unix epoch of the account's newest secret
  write here: the revision a push of it carries, unless that would
  not be above the last one sent.
- `fingerprint`: A non-zero fingerprint of every secret the account holds here (each
  one's kind, name, sealed value and write time). Every secret write
  seals with a fresh nonce, so any write changes it, even two in the
  same millisecond or one whose time went back.

Implements: `Clone`, `Copy`, `Debug`, `Eq`, `PartialEq`

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
pub fn new(store: &'a LibraryStore, lease: LibraryLease, ttl: Duration) -> Self
```

A fence for a lease acquired with `ttl`.

#### LeaseFence::lease

```rust
pub fn lease(&self) -> &LibraryLease
```

The lease as of the last successful check.

#### LeaseFence::check

```rust
pub async fn check(&mut self) -> Result<()>
```

Renews the lease, or fails when this holder no longer holds it.

##### Errors

Returns [`LibraryError::Conflict`](README.md#libraryerror) when the lease expired or another
holder took it, or the write error.

Implements: `Debug`

### LibraryLease

```rust
pub struct LibraryLease {
    pub name: String,
    pub holder: String,
    pub token: i64,
    pub expires_at_ms: i64,
}
```

A held library lease.

**Fields**

- `name`: Lease name.
- `holder`: Holder (a node id).
- `token`: Fencing token; a new holder always gets a larger one.
- `expires_at_ms`: Expiry in Unix milliseconds.

#### LibraryLease::is_held_at

```rust
pub fn is_held_at(&self, now_ms: i64) -> bool
```

True when the lease has not expired at `now_ms`.

Implements: `Clone`, `Debug`, `Eq`, `PartialEq`

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

### CapacityBody

```rust
pub struct CapacityBody {
    /* private fields */
}
```

`POST /api/config/join/capacity` body: one library and the job cap to
set for it.

#### CapacityBody::validate

```rust
pub fn validate(self) -> Result<JobCapRequest>
```

Checks the body: `max_jobs` must be present (`null` included) and at
most [`LinkJobCap::MAX`](#linkjobcapmax).

##### Errors

Returns [`LibraryError::Invalid`](README.md#libraryerror) when `max_jobs` is missing or above
the maximum.

Implements: `ComposeSchema`, `Debug`, `Deserialize<'de>`, `ToSchema`

### JobCapRequest

```rust
pub struct JobCapRequest {
    pub library_id: String,
    pub max_jobs: Option<LinkJobCap>,
}
```

A validated job cap change: the library it names (trimmed) and its new
cap, `None` for an equal share.

**Fields**

- `library_id`: The library (parent library id), surrounding whitespace removed.
- `max_jobs`: The cap to store; `None` clears it (an equal share).

Implements: `Clone`, `Debug`, `Eq`, `PartialEq`

### LibraryLink

```rust
pub struct LibraryLink {
    pub parent_library_id: String,
    pub owner_origin: String,
    pub remote_node_id: String,
    pub spki_pin: Option<String>,
    pub state: String,
    pub enrolled_at: String,
    pub share_books: bool,
    pub include_branches: bool,
    pub ancestry: Vec<String>,
    pub work_for_members: bool,
    pub keep_member_copies: bool,
    pub max_jobs: Option<LinkJobCap>,
    pub list_members_books: bool,
    pub keep_copies: bool,
    pub person_user_id: Option<i64>,
}
```

One library this install enrolled into.

**Fields**

- `parent_library_id`: Library id.
- `owner_origin`: Owner origin as [`bookclerk_node_proto::canonical_origin`](../bookclerk_node_proto/README.md#canonical_origin) spells it.
- `remote_node_id`: Node id the owner minted for this install.
- `spki_pin`: SPKI pin of the owner's self-signed certificate, when the link
  carried one.
- `state`: [`LIBRARY_LINK_ACTIVE`](#library_link_active), [`LIBRARY_LINK_REVOKED`](#library_link_revoked), or
  [`LIBRARY_LINK_LEFT`](#library_link_left).
- `enrolled_at`: RFC 3339 enrollment time.
- `share_books`: "Share this branch's books with *library*" (on by default).
- `include_branches`: "Include books from my own branches" (on by default; off shares
  only books acquired on this install itself).
- `ancestry`: The owner library's ancestry as it last reported it: its own
  library id first, then each library above it.
- `work_for_members`: "Download for other members of *library*" (off by default): with it
  off the branch takes only its owner's work from the library.
- `keep_member_copies`: "Keep copies of other members' books" (off by default): with it off
  a book acquired for another member goes only to the library.
- `max_jobs`: The operator's cap on jobs run for this library at once; `None`
  leaves the link its equal share of the machine.
- `list_members_books`: "List my members' books to *library*" (on by default): catalog
  lookups from that library are answered with the members' books
  (except those of members who opted out) as well as the owner's.
- `keep_copies`: "Keep copies on this computer" (on by default): whether the branch
  takes the library's copies of its person's books into its own
  storage. The branch takes the library's work and reads its books
  (`control_plane::window_books`) whether or not this is on.
- `person_user_id`: The local user who joined the library through this link, when one
  could be named: the only person who may add store connections to
  that library, or withdraw them from it, through this link.

#### LibraryLink::is_active

```rust
pub fn is_active(&self) -> bool
```

True when the host client should connect this link.

#### LibraryLink::claim_scope

```rust
pub fn claim_scope(&self) -> ClaimScope
```

Whose work the branch takes from this library.

Implements: `Clone`, `Debug`, `Eq`, `From<Model>`, `PartialEq`, `Serialize`

### LibraryLinkKeys

```rust
pub struct LibraryLinkKeys {
    /* private fields */
}
```

The node keys of one library link: the Ed25519 signing seed and the
X25519 box secret. Zeroized on drop; `Debug` shows only the public keys.

#### LibraryLinkKeys::generate

```rust
pub fn generate() -> Self
```

Fresh random keys from the OS RNG.

#### LibraryLinkKeys::from_bytes

```rust
pub fn from_bytes(sign_seed: &[u8; 32], box_secret: &[u8; 32]) -> Self
```

Keys from known bytes (tests, and reading the sealed row back).

#### LibraryLinkKeys::sign_seed

```rust
pub fn sign_seed(&self) -> &[u8; 32]
```

The Ed25519 signing seed.

#### LibraryLinkKeys::box_secret

```rust
pub fn box_secret(&self) -> &[u8; 32]
```

The X25519 box secret.

#### LibraryLinkKeys::sign_public_key

```rust
pub fn sign_public_key(&self) -> [u8; 32]
```

The Ed25519 public key the owner stores.

#### LibraryLinkKeys::box_public_key

```rust
pub fn box_public_key(&self) -> [u8; 32]
```

The X25519 public key the owner seals job secrets to.

Implements: `Clone`, `Debug`

### LibraryLinkSettings

```rust
pub struct LibraryLinkSettings {
    pub share_books: bool,
    pub include_branches: bool,
    pub list_members_books: Option<bool>,
    pub work_for_members: Option<bool>,
    pub keep_member_copies: Option<bool>,
    pub keep_copies: Option<bool>,
}
```

Every setting Settings → Library changes on one link at once; see
[`set_library_link_settings`](#set_library_link_settings).

**Fields**

- `share_books`: "Share this branch's books with *library*".
- `include_branches`: "Include books from my own branches".
- `list_members_books`: "List my members' books to *library*"; `None` keeps the stored value.
- `work_for_members`: "Download for other members of *library*"; `None` keeps the stored
  value.
- `keep_member_copies`: "Keep copies of other members' books"; `None` keeps the stored value.
- `keep_copies`: "Keep copies on this computer"; `None` keeps the stored value.

Implements: `Clone`, `Copy`, `Debug`, `Eq`, `PartialEq`

### LinkJobCap

```rust
pub struct LinkJobCap(/* private */);
```

The most jobs a branch runs for one library at once, as its operator
set it: 0 (runs nothing for that library) to [`LinkJobCap::MAX`](#linkjobcapmax).

#### LinkJobCap::MAX

```rust
pub const MAX: u8 = 16
```

The highest cap an operator may set: the most job slots a branch
ever counts its machine as having.

#### LinkJobCap::new

```rust
pub fn new(jobs: u32) -> Result<Self>
```

A cap of `jobs`.

##### Errors

Returns [`LibraryError::Invalid`](README.md#libraryerror) above [`Self::MAX`](#linkjobcapmax).

#### LinkJobCap::jobs

```rust
pub fn jobs(self) -> usize
```

The cap in jobs.

Implements: `Clone`, `Copy`, `Debug`, `Eq`, `Ord`, `PartialEq`, `PartialOrd`, `Serialize`

### NewLibraryLink

```rust
pub struct NewLibraryLink {
    pub parent_library_id: String,
    pub owner_origin: String,
    pub remote_node_id: String,
    pub spki_pin: Option<String>,
}
```

What enrollment returned, to record with [`record_library_link`](#record_library_link).

**Fields**

- `parent_library_id`: Library id from the enroll answer.
- `owner_origin`: Owner origin the host enrolled through.
- `remote_node_id`: Node id from the enroll answer.
- `spki_pin`: SPKI pin from the enrollment link.

Implements: `Clone`, `Debug`, `Eq`, `PartialEq`

### SharingBody

```rust
pub struct SharingBody {
    /* private fields */
}
```

`POST /api/config/join/sharing` body: one library and the settings to
change for it.

#### SharingBody::into_request

```rust
pub fn into_request(self) -> SharingRequest
```

The change the body asks for.

Implements: `ComposeSchema`, `Debug`, `Deserialize<'de>`, `ToSchema`

### SharingRequest

```rust
pub struct SharingRequest {
    pub library_id: String,
    pub settings: LibraryLinkSettings,
}
```

A sharing settings change: the library it names (trimmed) and the
settings to write, each `None` setting left as stored.

**Fields**

- `library_id`: The library (parent library id), surrounding whitespace removed.
- `settings`: The settings to write ([`set_library_link_settings`](#set_library_link_settings)).

Implements: `Clone`, `Debug`, `Eq`, `PartialEq`

### LinkImport

```rust
pub struct LinkImport<'a> {
    pub library_id: &'a str,
    pub book: &'a NewBook,
    pub primary_key: &'a str,
    pub pdf_key: Option<&'a str>,
    pub files: &'a [(&'a str, &'a str)],
}
```

One title the library committed, as the branch kept it.

**Fields**

- `library_id`: The library this branch acquired the title for.
- `book`: The title as the library sent it.
- `primary_key`: Storage key of the primary audio file, the same on every
  destination of the copy.
- `pdf_key`: Storage key of the companion PDF, when one was acquired.
- `files`: Every file of the copy: `(destination id, storage key)`, where the
  destination id is this install's (`local`, `s3`).

Implements: `Clone`, `Copy`, `Debug`

### OwnerShareFilter

```rust
pub struct OwnerShareFilter {
    /* private fields */
}
```

Which of this install's acquired books the catalog sync sends the
library `library_id`: the branch owner's own books, those held by a
store account connected here to the branch's person
([`super::branch_person`](#branch_person)); the books that library asked for through an
interlibrary request and this branch still lists to it
(`still_listed_requests`); and, while that library acquires all of this
branch's members' books, the books of every store account this branch
lists to it (`listed_accounts`). Never one imported from that library
([`books_imported_from_link`](#books_imported_from_link)). With no single person, the owner's
books are not among them.

#### OwnerShareFilter::load

```rust
pub async fn load(
    store: &LibraryStore,
    library_id: &str,
    members_books: bool,
) -> Result<Self>
```

Reads the filter for the library `library_id` now.

##### Arguments

* `members_books` - That library's "Acquire all of this branch's
  members' books", from its catalog session.

##### Errors

Returns an error when a read fails.

#### OwnerShareFilter::admits

```rust
pub fn admits(&self, key: &AcquiredTitleKey) -> bool
```

True when the book `key` names is sent.

#### OwnerShareFilter::admit_editions

```rust
pub fn admit_editions(&self, keys: Vec<AcquiredTitleKey>) -> Vec<AcquiredTitleKey>
```

The keys of `keys` that are sent, dropping every holding of an
edition when any of its holdings was imported from the library: a
copy imported for one account is the same file every other account
holding that edition would send back.

##### Arguments

* `keys` - Whole editions' keys, every account holding of each
  edition present, as one [`LibraryStore::acquired_title_keys_page`](README.md#librarystoreacquired_title_keys_page)
  returns them.

##### Returns

The admitted keys, in their order.

#### OwnerShareFilter::requested_at

```rust
pub fn requested_at(&self, key: &AcquiredTitleKey) -> Option<DateTime<Utc>>
```

When the library last asked for the book `key` names, if it asked
and this branch still lists it. The catalog sync orders a requested
book no earlier than this, so a book already past the sync's cursor
still goes up on the next pass.

Implements: `Clone`, `Debug`, `Default`

### TakenCopy

```rust
pub struct TakenCopy<'a> {
    pub plugin_key: &'a str,
    pub account_id: &'a str,
    pub marketplace: &'a str,
    pub product_id: &'a str,
}
```

A book of this install whose files came from a library it joined, by
its store identity here.

**Fields**

- `plugin_key`: Key of the plugin the book belongs to.
- `account_id`: Store account that owns the book.
- `marketplace`: The book's marketplace as its row states it; empty for a store
  without marketplaces.
- `product_id`: The store's product id.

Implements: `Clone`, `Copy`, `Debug`, `Eq`, `PartialEq`

### LiveRevision

```rust
pub struct LiveRevision(pub i64);
```

One topic's revision: a counter that every write to the topic's table
moves forward by at least one.

Implements: `Clone`, `Copy`, `Debug`, `Default`, `Eq`, `PartialEq`

### AssignedControls

```rust
pub struct AssignedControls {
    pub defaults: MemberControls,
    pub overrides: ControlOverrides,
    pub effective: MemberControls,
}
```

A user's controls: the library's default, their overrides, and what
applies to them.

**Fields**

- `defaults`: The library's default.
- `overrides`: The user's own overrides.
- `effective`: What applies to the user: everything for an active Owner or
  Administrator, nothing for a user who is not active, and the
  default with the overrides applied for an active Member.

Implements: `Clone`, `Copy`, `Debug`, `Eq`, `PartialEq`

### ControlOverrides

```rust
pub struct ControlOverrides {
    pub link_accounts: Option<bool>,
    pub download: Option<bool>,
    pub edit_metadata: Option<bool>,
    pub upload: Option<bool>,
    pub request_books: Option<bool>,
    pub see_prices: Option<bool>,
}
```

One patron's overrides: a value for some controls; the rest (an absent
or `null` field) follow the library's default for a Member and are
allowed for an Administrator.

**Fields**

- `link_accounts`: Overrides [`MemberControls::link_accounts`](#membercontrols).
- `download`: Overrides [`MemberControls::download`](#membercontrols).
- `edit_metadata`: Overrides [`MemberControls::edit_metadata`](#membercontrols).
- `upload`: Overrides [`MemberControls::upload`](#membercontrols).
- `request_books`: Overrides [`MemberControls::request_books`](#membercontrols).
- `see_prices`: Overrides [`MemberControls::see_prices`](#membercontrols).

#### ControlOverrides::get

```rust
pub fn get(&self, control: MemberControl) -> Option<bool>
```

The override of `control`, if any.

#### ControlOverrides::set

```rust
pub fn set(&mut self, control: MemberControl, value: Option<bool>)
```

Sets or clears (`None`) the override of `control`.

#### ControlOverrides::is_empty

```rust
pub fn is_empty(&self) -> bool
```

True when no control is overridden.

#### ControlOverrides::apply

```rust
pub fn apply(&self, defaults: MemberControls) -> MemberControls
```

`defaults` with these overrides applied.

Implements: `Clone`, `ComposeSchema`, `Copy`, `Debug`, `Default`, `Deserialize<'de>`, `Eq`, `PartialEq`, `Serialize`, `ToSchema`

### MemberControls

```rust
pub struct MemberControls {
    pub link_accounts: bool,
    pub download: bool,
    pub edit_metadata: bool,
    pub upload: bool,
    pub request_books: bool,
    pub see_prices: bool,
}
```

A value for each control: the library's default, or what applies to
one user.

**Fields**

- `link_accounts`: May connect store accounts ([`MemberControl::LinkAccounts`](#membercontrol)).
- `download`: May take copies of the library's files ([`MemberControl::Download`](#membercontrol)).
- `edit_metadata`: May change metadata ([`MemberControl::EditMetadata`](#membercontrol)).
- `upload`: May upload books ([`MemberControl::Upload`](#membercontrol)); off unless set.
- `request_books`: May ask branches for books ([`MemberControl::RequestBooks`](#membercontrol)).
- `see_prices`: May see prices and store links ([`MemberControl::SeePrices`](#membercontrol)).

#### MemberControls::all

```rust
pub fn all(value: bool) -> Self
```

Every control set to `value`.

#### MemberControls::allows

```rust
pub fn allows(&self, control: MemberControl) -> bool
```

Whether `control` is allowed.

#### MemberControls::set

```rust
pub fn set(&mut self, control: MemberControl, value: bool)
```

Sets `control` to `value`.

Implements: `Clone`, `ComposeSchema`, `Copy`, `Debug`, `Default`, `Deserialize<'de>`, `Eq`, `PartialEq`, `Serialize`, `ToSchema`

### PutUserControls

```rust
pub struct PutUserControls {
    pub overrides: ControlOverrides,
}
```

`PUT /api/users/{id}/controls` body.

**Fields**

- `overrides`: The controls set for this patron alone; an absent or `null`
  control follows the library's default for a Member and is allowed
  for an Administrator, so `{}` clears every override.

Implements: `Clone`, `ComposeSchema`, `Copy`, `Debug`, `Deserialize<'de>`, `Eq`, `PartialEq`, `Serialize`, `ToSchema`

### CacheUsage

```rust
pub struct CacheUsage {
    pub bytes: i64,
    pub budget_bytes: i64,
    pub swept_ms: Option<i64>,
}
```

What a node's image caches hold, as its daemon reports them.

**Fields**

- `bytes`: Bytes the caches hold now.
- `budget_bytes`: Bytes they may hold: the library's `cache.budget_bytes` or the
  node's override.
- `swept_ms`: When the last eviction sweep finished, in Unix milliseconds; `None`
  before the first since the daemon started.

Implements: `Clone`, `Copy`, `Debug`, `Default`, `Eq`, `PartialEq`

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

Implements: `Clone`, `ComposeSchema`, `Copy`, `Debug`, `Default`, `Eq`, `PartialEq`, `Serialize`, `ToSchema`

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
    pub cache: Option<CacheUsage>,
    pub public_origin: Option<String>,
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
- `cache`: The image caches' use; `None` from a node that keeps none (a
  branch).
- `public_origin`: The address other installs reach the node at, as a branch reports it
  (already parsed into a [`bookclerk_node_proto::BranchOrigin`](../bookclerk_node_proto/README.md#branchorigin) by the
  node API); `None` for a branch without one (a local branch) and for
  a shared node, which reports none.

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
    pub cache_bytes: Option<i64>,
    pub cache_budget_bytes: Option<i64>,
    pub cache_swept_ms: Option<i64>,
    pub public_origin: Option<String>,
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
- `memory_bytes`: Detected memory in bytes.
- `cpu_budget_millicores`: CPU the scheduler may commit, in millicores.
- `memory_budget_bytes`: Memory the scheduler may commit, in bytes.
- `schedulable`: False when an operator drained the node.
- `cpu_used_millicores`: Measured CPU in use, in millicores.
- `memory_available_bytes`: Measured memory available, in bytes.
- `files_dir_free_bytes`: Free bytes on the files directory's filesystem.
- `reserved`: Ceilings of the guests running here.
- `running_guests`: Guests running here.
- `availability_ppm`: Recent online fraction in parts per million.
- `online_since_ms`: Unix milliseconds the current online stretch began.
- `cache_bytes`: Bytes the node's image caches hold; `None` from a node that keeps
  none.
- `cache_budget_bytes`: Bytes the node's image caches may hold.
- `cache_swept_ms`: When the node's last cache sweep finished, in Unix milliseconds.
- `public_origin`: The address other installs reach this branch at, as it last
  reported it; `null` for a local branch and for a shared node. A
  branch with one is a remote branch of this library.

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

Implements: `Clone`, `ComposeSchema`, `Debug`, `Eq`, `PartialEq`, `Serialize`, `ToSchema`

### OutputSettingsV1

```rust
pub struct OutputSettingsV1 {
    pub format: OutputFormat,
    pub copies: Vec<OutputCopy>,
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
    pub missing_files: MissingFileAction,
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

Field meanings match [`OutputConfig`](../bookclerk_config/README.md#outputconfig). Unknown fields are rejected, inside
the nested `lame` and `replacement_characters` objects too, so a document
written by a newer binary under the same schema version fails closed
instead of being silently truncated on the next write.

**Fields**

- `format`: Post-acquire packaging format.
- `copies`: Further copies of every acquired book beside the main output, at
  most 8; empty keeps one copy. Each needs a quality that suits its
  format, and none may repeat another or the main output's copy.
- `widevine`: Prefer Widevine/CENC.
- `xhe_aac`: Prefer xHE-AAC on the Widevine path when offered.
- `widevine_cdm`: Optional `.wvd` path; relative paths resolve under each host's files dir.
- `widevine_cdm_provider`: Remote L3 CDM provider endpoint URL (`None` = the built-in AudibleCdm
  provider, empty/`off` disables provisioning).
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
- `missing_files`: What happens to a stored copy whose file disappears from its
  destination outside Bookclerk. Absent in documents written before
  the setting existed, which read as the default.
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

Implements: `Clone`, `ComposeSchema`, `Debug`, `Default`, `Deserialize<'de>`, `Eq`, `PartialEq`, `Serialize`, `ToSchema`

### AssignedScope

```rust
pub struct AssignedScope {
    pub scope: PatronScope,
    pub own: bool,
}
```

A Member's scope and where it came from.

**Fields**

- `scope`: The scope that applies.
- `own`: True when an Administrator gave the patron this scope; false when the
  patron follows the library's default.

Implements: `Clone`, `Debug`, `Eq`, `PartialEq`

### PatronScope

```rust
pub struct PatronScope {
    pub rules: Vec<ScopeRule>,
    pub explicit: ExplicitContent,
}
```

Which part of the library a patron sees.

**Fields**

- `rules`: Visibility rules, at most 16 and 1,024 bytes as JSON. Empty means
  every book is visible.
- `explicit`: Whether books marked explicit are visible; left out means `allow`.

#### PatronScope::unrestricted

```rust
pub fn unrestricted() -> Self
```

A scope that restricts nothing: no rules.

#### PatronScope::sees_everything

```rust
pub fn sees_everything(&self) -> bool
```

True when the scope hides no book: no rules, and explicit books
allowed.

#### PatronScope::facet_keys

```rust
pub fn facet_keys(&self, effect: RuleEffect) -> Vec<(WorkFacet, String)>
```

The rules with the given effect, as `(facet, facet key)` pairs for
the page index. Hiding explicit books adds an exclude pair on
[`WorkFacet::Explicit`](README.md#workfacet).

#### PatronScope::validate

```rust
pub fn validate(&self) -> Result<()>
```

Checks the scope's bounds.

##### Errors

Returns [`LibraryError::Invalid`](README.md#libraryerror) for more than [`MAX_SCOPE_RULES`](#max_scope_rules)
rules, an empty or over-long value, a rule given twice (the same
field, value and effect, compared without case), or a scope longer
than [`MAX_SCOPE_JSON_BYTES`](#max_scope_json_bytes) as JSON.

#### PatronScope::normalized

```rust
pub fn normalized(self) -> Self
```

The scope with each value trimmed, as it is stored.

Implements: `Clone`, `ComposeSchema`, `Debug`, `Default`, `Deserialize<'de>`, `Eq`, `PartialEq`, `Serialize`, `ToSchema`

### PutUserScope

```rust
pub struct PutUserScope {
    pub scope: Option<PatronScope>,
}
```

`PUT /api/users/{id}/scope` body.

**Fields**

- `scope`: The patron's own scope, or `null` to return them to the library's
  default.

Implements: `Clone`, `ComposeSchema`, `Debug`, `Deserialize<'de>`, `Eq`, `PartialEq`, `ToSchema`

### ScopeRule

```rust
pub struct ScopeRule {
    pub field: ScopeField,
    pub value: String,
    pub effect: RuleEffect,
}
```

One rule of a scope: a field, one value of it, and its effect.

**Fields**

- `field`: The field the rule matches.
- `value`: The value, compared without case and surrounding space (1 to 256
  bytes).
- `effect`: Include or exclude.

Implements: `Clone`, `ComposeSchema`, `Debug`, `Deserialize<'de>`, `Eq`, `PartialEq`, `Serialize`, `ToSchema`

### NewPendingJoin

```rust
pub struct NewPendingJoin {
    pub owner_origin: String,
    pub spki_pin: Option<String>,
    pub parent_library_id: String,
    pub library_name: String,
    pub branch_name: String,
    pub redirect_uri: String,
    pub allow_shared_branch: bool,
}
```

What a joining daemon records when it starts a sign-in.

**Fields**

- `owner_origin`: Owner origin the sign-in goes to.
- `spki_pin`: SPKI pin the operator confirmed for a self-signed owner.
- `parent_library_id`: Library id `GET /api/node/v1/info` reported.
- `library_name`: Library name the owner reported.
- `branch_name`: Name this install joins as.
- `redirect_uri`: Redirect URI of the authorization request.
- `allow_shared_branch`: True when the operator offered to join as a shared branch (the install was
  empty); the callback may then apply a shared branch bootstrap.

Implements: `Clone`, `Debug`, `Eq`, `PartialEq`

### PendingJoin

```rust
pub struct PendingJoin {
    pub owner_origin: String,
    pub spki_pin: Option<String>,
    pub parent_library_id: String,
    pub library_name: String,
    pub branch_name: String,
    pub redirect_uri: String,
    pub allow_shared_branch: bool,
    pub expires_at: String,
}
```

A pending join as stored (no secrets).

**Fields**

- `owner_origin`: Owner origin.
- `spki_pin`: Confirmed SPKI pin.
- `parent_library_id`: Library id.
- `library_name`: Library name.
- `branch_name`: Branch name.
- `redirect_uri`: Redirect URI.
- `allow_shared_branch`: True when the operator offered to join as a shared branch.
- `expires_at`: RFC 3339 expiry.

Implements: `Clone`, `Debug`, `Eq`, `From<Model>`, `PartialEq`

### TakenJoin

```rust
pub struct TakenJoin {
    pub join: PendingJoin,
    pub verifier: Zeroizing<String>,
    pub keys: LibraryLinkKeys,
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
  library's operator (instances carry no owner yet), so only
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
    pub library_id: &'a str,
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

- `library_id`: Library id.
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
    pub parent_library_id: String,
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

One owner plugin of one library, as this install sees it.

**Fields**

- `parent_library_id`: Library id.
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

### HandedPassEnd

```rust
pub struct HandedPassEnd<'a> {
    pub account_id: &'a str,
    pub sync_token: &'a str,
    pub complete: bool,
    pub full: bool,
}
```

How a Host's pass of one account ended.

**Fields**

- `account_id`: The account the pass scanned.
- `sync_token`: The token the plugin returned for the account's next pass; empty
  when it keeps none.
- `complete`: The pass listed every title the account holds.
- `full`: The pass was full (sent no sync token), so its rows are everything
  the account holds when it is also complete.

Implements: `Clone`, `Copy`, `Debug`, `Eq`, `PartialEq`

### ScanSighting

```rust
pub struct ScanSighting {
    pub account_id: String,
    pub product_id: String,
}
```

A product a Host's pass listed that has no library row to write (its
record maps onto no flat row, an ebook for instance), so the owner only
marks the account's holding of it seen by the pass.

**Fields**

- `account_id`: The account the pass listed it for.
- `product_id`: The storefront product id it listed.

Implements: `Clone`, `Debug`, `Eq`, `PartialEq`

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
shared node serving it.

**Fields**

- `lease_secs`: Lease length; permits are taken for the same time.
- `limits`: Library semaphore limits.

Implements: `Clone`, `Copy`, `Debug`

### RemoteJobClaim

```rust
pub struct RemoteJobClaim {
    pub job: JobRecord,
    pub secrets: Vec<JobSecret>,
    pub plugin_keys: Vec<String>,
    pub source_plugin_key: Option<String>,
    pub resumed: bool,
    pub for_node_owner: bool,
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
- `for_node_owner`: Every store account the job touches belongs to the user who owns
  the node ([`super::job_for_node_owner`](#job_for_node_owner)): the branch owner's own
  work.

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

### LibrarySecret

```rust
pub struct LibrarySecret {
    pub library_id: String,
    pub secret_fingerprint: String,
}
```

Library id and fingerprint of the secret root aligned for this process.

The unwrapped DEK stays in the process cache (`configure_master_key_with`).
Callers that need the key use [`crate::require_master_key`](master_key.md#require_master_key).

**Fields**

- `library_id`: Library id stored beside the fingerprint.
- `secret_fingerprint`: SHA-256 hex of the DEK.

Implements: `Clone`, `Debug`

### SemaphorePermit

```rust
pub struct SemaphorePermit {
    pub semaphore: String,
    pub slot: u32,
    pub lease: LibraryLease,
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
- `limit`: Permits the semaphore grants library-wide (at least 1).

#### SemaphoreRequest::new

```rust
pub fn new(name: impl Into<String>, limit: u32) -> Self
```

A request for one permit of `name` under `limit`.

Implements: `Clone`, `Debug`, `Eq`, `Ord`, `PartialEq`, `PartialOrd`

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
- `books`: Books: editions in the library model, held or not, so any book
  data the install holds counts.
- `libraries_joined`: Libraries this install is a branch of (not left).

#### LibraryContents::is_empty

```rust
pub fn is_empty(&self) -> bool
```

True when the install holds no library data: it may join as a shared branch.

#### LibraryContents::reasons

```rust
pub fn reasons(&self) -> Vec<String>
```

Why the install is not empty, for the operator.

Implements: `Clone`, `Copy`, `Debug`, `Default`, `Eq`, `PartialEq`, `Serialize`

### TemplateStrip

```rust
pub struct TemplateStrip {
    pub identity_rows: u64,
    pub leases: u64,
    pub node_status: u64,
}
```

What [`strip_install_state`](#strip_install_state) removed.

**Fields**

- `identity_rows`: `library_identity` rows deleted (0 or 1); each holds the library id and
  the fingerprint of its key, never the key.
- `leases`: Library lease rows deleted.
- `node_status`: Node heartbeat rows deleted.

Implements: `Clone`, `Copy`, `Debug`, `Eq`, `PartialEq`

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

One output destination of the receiving shared node an upload is
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
    pub holding_id: &'a str,
    pub primary_key: &'a str,
    pub pdf_key: Option<&'a str>,
    pub files: &'a [UploadFile],
    pub destinations: &'a [UploadDestination],
}
```

What a Host asks to upload for one title.

**Fields**

- `holding_id`: The title's holding (`holdings.id`): an account holding for a job
  upload, the node holding for a branch's catalog upload. Empty for the
  job's closing upload, which carries no files and only completes the
  job.
- `primary_key`: Storage key of the title's primary audio file on the primary
  destination (one of `files`' copies there).
- `pdf_key`: Storage key of the companion PDF on the primary destination (one of
  `files`' copies there), when uploaded.
- `files`: Every file, in upload order.
- `destinations`: The receiving shared node's output destinations, the primary first,
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
    pub holding_id: String,
    pub primary_key: String,
    pub pdf_key: Option<String>,
    pub destinations: Vec<String>,
    pub files: Vec<UploadFile>,
    pub received: BTreeSet<usize>,
    pub state: String,
    pub catalog_digest: Option<String>,
    pub catalog_edition: Option<String>,
}
```

One open upload, as the upload endpoint sees it.

**Fields**

- `upload_id`: Upload id.
- `node_id`: Branch node it was granted to.
- `job_id`: Acquire job.
- `generation`: Lease generation it is bound to.
- `receiving_node_id`: Shared node that receives the files.
- `holding_id`: The title's holding (`holdings.id`; empty for the job's closing
  upload).
- `primary_key`: Storage key of the primary audio file.
- `pdf_key`: Storage key of the companion PDF, when uploaded.
- `destinations`: The receiving node's destination ids at grant, the primary first;
  the files are written only while the node still has exactly these.
- `files`: The declared files.
- `received`: Indexes of the files every destination has.
- `state`: `open`, `committed` or `aborted`.
- `catalog_digest`: For a branch's catalog upload ([`Self::is_catalog`](#uploadsessionis_catalog)): the title's
  manifest digest at grant.
- `catalog_edition`: For a branch's catalog upload: the edition the holding's listing
  sold at grant. The upload writes under that edition's keys, so it
  is recorded as delivered only while the listing still sells it.

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

### WindowEntry

```rust
pub struct WindowEntry {
    pub key: String,
    pub work_id: Option<String>,
    pub editions: Vec<WindowEntryEdition>,
}
```

One work in the window's merged view.

**Fields**

- `key`: A key for the entry that stays the same while its rows do: the
  local work id when it has one, else its first edition's library,
  store, marketplace and product id.
- `work_id`: The work of this install's library the entry was placed on, if any.
- `editions`: Every edition held for the person, over every library, in library
  then store order.

#### WindowEntry::title

```rust
pub fn title(&self) -> &str
```

The entry's title: its first edition's.

#### WindowEntry::formats

```rust
pub fn formats(&self) -> Vec<LookupFormat>
```

The formats any library has, deduplicated.

Implements: `Clone`, `Debug`, `Eq`, `PartialEq`

### WindowEntryEdition

```rust
pub struct WindowEntryEdition {
    pub parent_library_id: String,
    pub owner_origin: String,
    pub edition: LookupEdition,
    pub yours: bool,
    pub copy: Option<WindowCopy>,
    pub work_ref: String,
}
```

One edition of a [`WindowEntry`](#windowentry), and the library that holds it.

**Fields**

- `parent_library_id`: The library (parent library id) that holds it for the person.
- `owner_origin`: The library's origin, for display.
- `edition`: The edition as that library sent it.
- `yours`: Whether the person holds it there through a store account of their
  own.
- `copy`: The copy of its audio that library holds, when it stated one.
- `work_ref`: That library's own key for the edition's work (its work id, or
  `edition-<id>` for an edition with no work there): what to ask the
  library for when only one of its branches lists the edition.

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
- `concurrency`: Local delivery workers and the library in-flight cap.

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

Implements: `Clone`, `ComposeSchema`, `Debug`, `Deserialize<'de>`, `Eq`, `PartialEq`, `Serialize`, `ToSchema`

### NodeRuntimeSettingsV1

```rust
pub struct NodeRuntimeSettingsV1 {
    pub label: String,
    pub cpu_budget_percent: u8,
    pub memory_budget_percent: u8,
    pub schedulable: bool,
    pub cache_budget_bytes: Option<u64>,
}
```

Typed node-scoped runtime document, schema version 1.

Desired state for one node, editable from any node in the library: an
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
- `cache_budget_bytes`: Disk space, in bytes, this node's image caches may use together,
  overriding the library's `core.cache` budget (16 MiB to 1 TiB).
  Absent means the library budget applies.

#### NodeRuntimeSettingsV1::new

```rust
pub fn new() -> Self
```

Empty label, default budgets, schedulable.

#### NodeRuntimeSettingsV1::validate

```rust
pub fn validate(&self) -> Result<()>
```

Rejects control characters, labels longer than 64 bytes, budgets
outside 1..=100, and a cache budget outside 16 MiB..=1 TiB.

##### Errors

Returns an error when a field is out of range.

Implements: `Clone`, `ComposeSchema`, `Debug`, `Default`, `Deserialize<'de>`, `Eq`, `PartialEq`, `Serialize`, `ToSchema`

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
- `scope_type`: Scope type (`library` or `node`).
- `scope_id`: Scope id (`singleton` or a node id).
- `namespace`: Domain namespace.

Implements: `Clone`, `Debug`, `Eq`, `PartialEq`

### ControlPlaneSession

```rust
pub struct ControlPlaneSession {
    pub node: NodeRecord,
    pub library_id: String,
    pub events: ConfigurationDocument<EventsSettingsV1>,
    pub output: ConfigurationDocument<OutputSettingsV1>,
    pub node_runtime: ConfigurationDocument<NodeRuntimeSettingsV1>,
}
```

Identity, secret root, and the authoritative events document for this process.

**Fields**

- `node`: Registered node.
- `library_id`: Library id from the secret root.
- `events`: Authoritative `[events]` document.
- `output`: Authoritative shared `[output]` document.
- `node_runtime`: This node's runtime document. Other nodes' documents are not included.

Implements: `Clone`, `Debug`

## Enums

### LinkRevoked

```rust
pub enum LinkRevoked {
    Revoked,
    NotLinked,
}
```

What [`revoke_account_link`](#revoke_account_link) did.

**Variants**

- `Revoked`: The account was marked revoked, its sign-in deleted and the
  identity's link removed.
- `NotLinked`: The identity held no link to the account when the batch ran (a
  concurrent revoke finished first); nothing changed.

Implements: `Clone`, `Copy`, `Debug`, `Eq`, `PartialEq`

### SignInLink

```rust
pub enum SignInLink {
    Linked,
    AlreadyYours,
    LinkedToSomeoneElse,
    SignInTakenBack,
}
```

What [`link_signed_in_account`](#link_signed_in_account) did.

**Variants**

- `Linked`: The account is now linked to the identity, active with scans on.
- `AlreadyYours`: The identity already held the link; the account is active again
  when its sign-in is stored.
- `LinkedToSomeoneElse`: Another identity holds the link; nothing changed.
- `SignInTakenBack`: The account's sign-in was taken back (a refused login of the same
  store account, or a revoke, ran after this login stored it), so
  nothing was linked. Signing in again connects it.

Implements: `Clone`, `Copy`, `Debug`, `Eq`, `PartialEq`

### AdminControl

```rust
pub enum AdminControl {
    ManageMembers,
    SetPatronLimits,
    LibraryScan,
    RunJobs,
    FixCatalog,
    InviteBranches,
    ApproveRequests,
}
```

One power an Administrator may be allowed or refused.

**Variants**

- `ManageMembers`: Invite Members, change their details, disable or delete them, reset
  their passwords and mint claim tickets. Off, the Administrator still
  sees the user list.
- `SetPatronLimits`: Set patron scopes and member controls, for one Member and the
  library's defaults. Off, the Administrator still reads them.
- `LibraryScan`: Scan and acquire for every store account, plus the legacy `/scan`
  and `/acquire`. Off, the Administrator scans and acquires only with
  the store accounts linked to them, like a Member.
- `RunJobs`: Cancel jobs; retry, acknowledge, cancel or resume event deliveries;
  run a listening sync. Off, the Administrator still watches jobs and
  queues.
- `FixCatalog`: Pin listings, split works and resolve conflicts.
- `InviteBranches`: Mint enrollment links for their own new nodes. Nodes they already
  own keep running.
- `ApproveRequests`: Approve interlibrary requests from branches and Members. Off,
  requests wait for an Owner.

#### AdminControl::ALL

```rust
pub const ALL: [Self; 7] = _
```

Every switch, in the order the editors list them.

#### AdminControl::as_str

```rust
pub fn as_str(self) -> &'static str
```

The switch's wire name (`manage_members`, `library_scan`, ...).

#### AdminControl::parse

```rust
pub fn parse(raw: &str) -> Option<Self>
```

Parses a wire name.

Implements: `Clone`, `ComposeSchema`, `Copy`, `Debug`, `Deserialize<'de>`, `Eq`, `Hash`, `Ord`, `PartialEq`, `PartialOrd`, `Serialize`, `ToSchema`

### AdminSwitchChoice

```rust
pub enum AdminSwitchChoice {
    On,
    Off,
    Default,
}
```

What a change does to one switch of one Administrator.

**Variants**

- `On`: Allow the switch for this Administrator, whatever the default says.
- `Off`: Refuse the switch for this Administrator, whatever the default says.
- `Default`: Follow the library's default again.

#### AdminSwitchChoice::value

```rust
pub fn value(self) -> Option<bool>
```

The stored override: `Some(true)`, `Some(false)`, or `None` to
follow the default.

Implements: `Clone`, `ComposeSchema`, `Copy`, `Debug`, `Deserialize<'de>`, `Eq`, `PartialEq`, `Serialize`, `ToSchema`

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
    branch's owner (no echo back), or because they formed no valid
    listing.
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

### RequestFinished

```rust
pub enum RequestFinished {
    Recorded,
    Lost,
    Refused,
}
```

What [`finish_branch_request`](#finish_branch_request) did.

**Variants**

- `Recorded`: The outcome was recorded on the job.
- `Lost`: The lease is no longer the branch's (it expired and moved on, the
  job was cancelled, or the job is not a request for this branch).
- `Refused`: The node is not a branch this library trusts.

Implements: `Clone`, `Copy`, `Debug`, `Eq`, `PartialEq`

### Withdrawn

```rust
pub enum Withdrawn {
    Unlinked,
    NotTheirs,
}
```

What [`withdraw_person_account`](#withdraw_person_account) did.

**Variants**

- `Unlinked`: The person's links to the account were removed (and the account
  taken back when no other link held it).
- `NotTheirs`: The person had no link to the account here; nothing changed.

Implements: `Clone`, `Copy`, `Debug`, `Eq`, `PartialEq`

### AnswersRecorded

```rust
pub enum AnswersRecorded {
    Recorded(AnswerReport),
    Refused(&'static str),
}
```

Outcome of [`record_answers`](#record_answers).

**Variants**

- `Recorded`: The answers were applied.
- `Refused`: The node is not a branch this library trusts.

Implements: `Clone`, `Debug`, `Eq`, `PartialEq`

### ClerkLinkState

```rust
pub enum ClerkLinkState {
    Active,
    SignedOut,
}
```

Whether this install still holds the card of a clerk link.

**Variants**

- `Active`: The card is held; the clerk reader dials the library.
- `SignedOut`: The library ended the card (or the person signed out here); the
  card is deleted and nothing dials.

#### ClerkLinkState::as_str

```rust
pub fn as_str(self) -> &'static str
```

The `clerk_links.state` spelling.

Implements: `Clone`, `Copy`, `Debug`, `Eq`, `PartialEq`, `Serialize`

### CopyHolder

```rust
pub enum CopyHolder<'a> {
    Node(&'a str),
    Card(&'a str),
}
```

Who asks for a copy.

**Variants**

- `Node`: A branch node, on its node socket; the person is its owner.
- `Card`: A clerk, on its clerk socket, named by the SHA-256 hex of its
  library card (`portal_sessions.token_hash`); the person is the
  card's.

Implements: `Clone`, `Copy`, `Debug`, `Eq`, `PartialEq`

### CopyPlanned

```rust
pub enum CopyPlanned {
    Refused,
    NotFound,
    Planned(CopyPlan),
}
```

The answer of [`plan_copy`](#plan_copy).

**Variants**

- `Refused`: The asker is not one this library gives copies to (frame code
  `refused`).
- `NotFound`: The person cannot take that edition, or the library holds no copy
  of it (frame code `not_found` for both).
- `Planned`: The copy and its files.

Implements: `Clone`, `Debug`, `Eq`, `PartialEq`

### CronFireClaim

```rust
pub enum CronFireClaim {
    Claimed { job_id: Option<String> },
    Lost,
}
```

What [`claim_cron_fire`](#claim_cron_fire) did.

**Variants**

- `Claimed`: This call moved the cursor and admitted the fire as job `job_id`, or
  moved the cursor while an earlier fire of the schedule was still
  active (`job_id` is `None`; the fire is skipped).
  - `job_id`: The new job, or `None` when the fire coalesced onto an active one.
- `Lost`: The cursor was not at `expected_last_ms` (another node claimed the
  fire, or this node's view is stale). Re-read with
  [`load_cron_cursor`](#load_cron_cursor).

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

Implements: `Clone`, `ComposeSchema`, `Copy`, `Debug`, `Display`, `Eq`, `Hash`, `Ord`, `PartialEq`, `PartialOrd`, `Serialize`, `ToSchema`

### ConfigActor

```rust
pub enum ConfigActor {
    Operator { id: String },
    Administrator { id: String },
    Member { id: String },
    Bootstrap,
    Scheduler,
    NodeOwner(VerifiedNodeOwner),
    LibraryOwner(VerifiedLibraryOwner),
    PatronAdministrator(VerifiedPatronAdministrator),
}
```

Who is allowed to write a configuration document.

**Variants**

- `Operator`: Control-plane operator (CLI or operator token).
  - `id`: Stable audit id (`cli`, `operator`, …).
- `Administrator`: Portal administrator. Cannot write library or node configuration.
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
- `LibraryOwner`: An active portal Owner. Writes the library's branch-enrollment switch
  (`core.branch_enrollment`) and the default Administrator switches
  (`core.admin_controls`) only; every other document stays an
  operator write. Built by [`super::verify_library_owner`](#verify_library_owner), which reads
  the user's stored role and status.
- `PatronAdministrator`: An active portal Owner or Administrator. Writes the library's
  patron defaults only: the default scope (`core.patron_scope`), the
  default member controls (`core.member_controls`) and the
  collections scopes match (`core.collections`). Built by
  [`super::verify_patron_administrator`](#verify_patron_administrator), which reads the user's
  stored role and status.

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

### BranchRelationship

```rust
pub enum BranchRelationship {
    SharedBranch,
    RemoteBranch,
    LocalBranch,
}
```

How a node relates to the library, as people are told: a shared branch
serves the library from its database, and a branch with its own
database is a remote branch when it reports a public address (it is a
core branch other installs can reach) and a local branch when it does
not.

**Variants**

- `SharedBranch`: Shares the library's database and keys ([`NodeKind::Shared`](#nodekind)).
- `RemoteBranch`: Its own install, reachable at the public address it reports.
- `LocalBranch`: Its own install with no public address; it dials the library and is
  never dialed.

#### BranchRelationship::of

```rust
pub fn of(kind: NodeKind, public_origin: Option<&str>) -> Self
```

The relationship of a node of `kind` whose latest heartbeat reported
`public_origin`. A branch counts as local until a heartbeat reports
an address.

##### Arguments

* `kind` - How the node joined.
* `public_origin` - The address its latest heartbeat reported, if any.

##### Returns

[`Self::SharedBranch`](#branchrelationship) for a shared node, whatever it reports;
otherwise [`Self::RemoteBranch`](#branchrelationship) with an address and
[`Self::LocalBranch`](#branchrelationship) without.

#### BranchRelationship::as_str

```rust
pub fn as_str(self) -> &'static str
```

Wire text (`shared_branch`, `remote_branch`, `local_branch`).

Implements: `Clone`, `ComposeSchema`, `Copy`, `Debug`, `Deserialize<'de>`, `Eq`, `PartialEq`, `Serialize`, `ToSchema`

### NodeKind

```rust
pub enum NodeKind {
    Shared,
    Branch,
}
```

How a node joined the library.

**Variants**

- `Shared`: Shares the library database and `master.key`. Every node is one today.
- `Branch`: Keeps its own database and operator, and enrolled to contribute work.

#### NodeKind::as_str

```rust
pub fn as_str(self) -> &'static str
```

Stored text.

#### NodeKind::parse

```rust
pub fn parse(raw: &str) -> Self
```

Parses stored text; anything unknown reads as [`Self::Shared`](#nodekind).

Implements: `Clone`, `ComposeSchema`, `Copy`, `Debug`, `Deserialize<'de>`, `Eq`, `PartialEq`, `Serialize`, `ToSchema`

### CreateRefusal

```rust
pub enum CreateRefusal {
    NotAllowed,
    Empty,
    TooManyFiles,
    FileTooLarge { member_no: usize },
    PathClash { member_no: usize },
    BadReference { member_no: usize },
    OverQuota { staged: u64, requested: u64, limit: u64 },
}
```

Why a batch was not created.

**Variants**

- `NotAllowed`: The person may not upload (the `upload` member control), is not
  active, or is unknown.
- `Empty`: The manifest names no file.
- `TooManyFiles`: The manifest names more than [`MAX_BATCH_FILES`](#max_batch_files) files.
- `FileTooLarge`: A file is larger than the per-file limit.
  - `member_no`: The file's position in the manifest.
- `PathClash`: Two files share a path, or a path folds onto another (case,
  Unicode form, or a file where a folder is).
  - `member_no`: The later file's position in the manifest.
- `BadReference`: An intake member lacks its storage reference, an upload member has
  one, or a reference is too long.
  - `member_no`: The file's position in the manifest.
- `OverQuota`: Admitting the batch would pass the person's staging quota.
  - `staged`: Bytes the person already has staged.
  - `requested`: Bytes this batch declares.
  - `limit`: The quota.

Implements: `Clone`, `Debug`, `Display`, `Eq`, `PartialEq`

### IngestSource

```rust
pub enum IngestSource {
    Upload,
    Intake,
}
```

Where a batch's bytes come from.

**Variants**

- `Upload`: Staged on the receiving node by the upload API.
- `Intake`: Read in place from an intake storage instance.

#### IngestSource::ALL

```rust
pub const ALL: &'static [Self] = _
```

Every value, in declaration order.

#### IngestSource::as_str

```rust
pub fn as_str(self) -> &'static str
```

The stored text.

#### IngestSource::parse

```rust
pub fn parse(raw: &str) -> Option<Self>
```

Parses stored text; `None` for anything else.

Implements: `Clone`, `ComposeSchema`, `Copy`, `Debug`, `Deserialize<'de>`, `Eq`, `Hash`, `PartialEq`, `Serialize`, `ToSchema`

### IngestState

```rust
pub enum IngestState {
    Open,
    Committed,
    Running,
    Done,
    Failed,
    Cancelled,
}
```

Where a batch is in its life.

**Variants**

- `Open`: Taking bytes; nothing identified yet.
- `Committed`: Every file arrived and the job is queued.
- `Running`: The job is identifying and storing it.
- `Done`: Every member has its outcome.
- `Failed`: The job gave up; members keep the outcomes they reached.
- `Cancelled`: Cancelled before the job started.

#### IngestState::ALL

```rust
pub const ALL: &'static [Self] = _
```

Every value, in declaration order.

#### IngestState::as_str

```rust
pub fn as_str(self) -> &'static str
```

The stored text.

#### IngestState::parse

```rust
pub fn parse(raw: &str) -> Option<Self>
```

Parses stored text; `None` for anything else.

#### IngestState::is_final

```rust
pub fn is_final(self) -> bool
```

True for a state the batch never leaves.

Implements: `Clone`, `ComposeSchema`, `Copy`, `Debug`, `Deserialize<'de>`, `Eq`, `Hash`, `PartialEq`, `Serialize`, `ToSchema`

### ManifestError

```rust
pub enum ManifestError {
    Malformed(String),
    Path { member_no: usize, refusal: PathRefusal },
    Digest { member_no: usize },
    TooManyFiles,
}
```

Why a manifest was refused before any batch was made.

**Variants**

- `Malformed`: The body is not the documented JSON.
- `Path`: A file's path is not usable.
  - `member_no`: The file's position.
  - `refusal`: Why.
- `Digest`: A file's digest is not 64 hex digits.
  - `member_no`: The file's position.
- `TooManyFiles`: The manifest names more than [`MAX_BATCH_FILES`](#max_batch_files) files.

Implements: `Clone`, `Debug`, `Display`, `Eq`, `PartialEq`

### MemberOutcome

```rust
pub enum MemberOutcome {
    Pending,
    Expanded,
    Matched,
    NeedsReview,
    Duplicate,
    Refused,
    Failed,
}
```

What became of one member.

**Variants**

- `Pending`: Not decided yet.
- `Expanded`: An archive whose entries joined the batch as their own members.
- `Matched`: Stored as part of an edition a strong identifier or ISBN named.
- `NeedsReview`: Stored as part of a new local edition an Administrator should
  review.
- `Duplicate`: Its edition already had files here; the existing copy was kept.
- `Refused`: Not stored, for the given reason.
- `Failed`: Storing it failed.

#### MemberOutcome::ALL

```rust
pub const ALL: &'static [Self] = _
```

Every value, in declaration order.

#### MemberOutcome::as_str

```rust
pub fn as_str(self) -> &'static str
```

The stored text.

#### MemberOutcome::parse

```rust
pub fn parse(raw: &str) -> Option<Self>
```

Parses stored text; `None` for anything else.

Implements: `Clone`, `ComposeSchema`, `Copy`, `Debug`, `Deserialize<'de>`, `Eq`, `Hash`, `PartialEq`, `Serialize`, `ToSchema`

### IngressExposure

```rust
pub enum IngressExposure {
    Branches,
    Library,
}
```

What public traffic from the ingress may reach.

**Variants**

- `Branches`: Only what branches and clerks need: the node API, sign-in and the
  clerk routes. The default, so turning ingress on never publishes the
  web app by surprise.
- `Library`: The whole library, web app included.

#### IngressExposure::as_str

```rust
pub fn as_str(self) -> &'static str
```

Stable spelling used by the CLI and the API.

Implements: `Clone`, `ComposeSchema`, `Copy`, `Debug`, `Default`, `Deserialize<'de>`, `Eq`, `Hash`, `PartialEq`, `Serialize`, `ToSchema`

### OriginReport

```rust
pub enum OriginReport {
    Adopted(IngressOrigin),
    Pending { adopted: String, reported: IngressOrigin },
    NotActive,
}
```

What [`record_started`](#record_started) did with a reported origin.

**Variants**

- `Adopted`: The origin is the library's public origin now (first report, or the
  same host again).
- `Pending`: The origin differs from the adopted one; it is recorded and waits for
  the operator.
  - `adopted`: The origin the library keeps serving under.
  - `reported`: The origin the plugin reported.
- `NotActive`: The reporting instance is not the active one; nothing was written.

Implements: `Clone`, `Debug`, `Eq`, `PartialEq`

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

### SettingCell

```rust
pub enum SettingCell {
    Bool(bool),
    Number(Number),
    String(String),
}
```

One cell of a table setting row: the scalar kinds only.

**Variants**

- `Bool`: JSON boolean.
- `Number`: Finite JSON number; an integer for integer columns.
- `String`: JSON string, at most 1024 bytes.

#### SettingCell::to_json

```rust
pub fn to_json(&self) -> Value
```

The cell as JSON.

Implements: `Clone`, `ComposeSchema`, `Debug`, `Deserialize<'de>`, `PartialEq`, `Serialize`, `ToSchema`

### SettingValue

```rust
pub enum SettingValue {
    Bool(bool),
    Number(Number),
    String(String),
    Rows(Vec<BTreeMap<String, SettingCell>>),
}
```

One setting value: a scalar, or the rows of a table setting.

**Variants**

- `Bool`: JSON boolean.
- `Number`: Finite JSON number: an integer kind stores an `i64`, so the schema
  is an unformatted `number` rather than a `double`.
- `String`: JSON string, at most 1024 bytes.
- `Rows`: Rows of a table setting: each row maps a column key to a scalar cell.
  Rows are records rather than positional lists, so a column added to
  the declaration later keeps existing rows valid.

#### SettingValue::to_json

```rust
pub fn to_json(&self) -> Value
```

The value as JSON, the form `CONFIG` and the API carry.

Implements: `Clone`, `ComposeSchema`, `Debug`, `Deserialize<'de>`, `PartialEq`, `Serialize`, `ToSchema`

### KeeperUpdate

```rust
pub enum KeeperUpdate {
    Applied,
    Stale,
    NotShared,
}
```

What [`apply_keeper_update`](#apply_keeper_update) did.

**Variants**

- `Applied`: The account's secrets here now hold the pushed sign-in.
- `Stale`: This revision, or a newer one, was already applied from the branch.
- `NotShared`: The account is not `user_id`'s here, or not shared with the branch.

Implements: `Clone`, `Copy`, `Debug`, `Eq`, `PartialEq`

### LinkImportOutcome

```rust
pub enum LinkImportOutcome {
    Imported(Box<AcquireTitle>),
    AccountNotHere,
    NoPerson,
    KeyHeldElsewhere,
}
```

What [`import_link_title`](#import_link_title) did.

**Variants**

- `Imported`: The title is in this library now, as this account's title.
- `AccountNotHere`: The store account that bought it is not connected here to the
  branch's person; nothing was written.
- `NoPerson`: This install has no single person a branch's books belong to (no
  active Owner, or several); nothing was written.
- `KeyHeldElsewhere`: Another live edition here holds the copy's primary file key, so the
  copy is not this title's: its holding stays (not acquired) and
  nothing else was recorded.

Implements: `Clone`, `Debug`

### LiveTopic

```rust
pub enum LiveTopic {
    Library,
    Jobs,
    Deliveries,
    Nodes,
    Config,
    Consents,
    Libraries,
    Users,
    Discover,
    Playback,
}
```

A topic of the live socket.

**Variants**

- `Library`: The library's titles (`books`).
- `Jobs`: Jobs (`jobs`).
- `Deliveries`: Event deliveries (`event_deliveries`).
- `Nodes`: Node heartbeats (`node_status`).
- `Config`: Library configuration documents (`configuration_changes`).
- `Consents`: Owner-plugin consent on this host (`remote_plugin_consents`).
- `Libraries`: Owner libraries this host joined, and their state (`library_links`).
- `Users`: Users (`users`).
- `Discover`: Stored Discover shelves (`discover_feeds`): a ranking job wrote or
  dropped someone's shelves.
- `Playback`: Listening progress and bookmarks (`playback_progress`, `bookmarks`):
  someone started, stopped or finished an edition, or changed a
  bookmark. Position syncs while a session plays do not move it.

#### LiveTopic::ALL

```rust
pub const ALL: [Self; 10] = _
```

Every topic.

#### LiveTopic::as_str

```rust
pub fn as_str(self) -> &'static str
```

The topic's wire name, also its key in `live_topic_revisions`.

#### LiveTopic::parse

```rust
pub fn parse(name: &str) -> Option<Self>
```

The topic for a wire name, or `None` for a name no topic has.

#### LiveTopic::stamp_column

```rust
pub fn stamp_column(self) -> Option<(&'static str, &'static str)>
```

The table behind the topic and its change-stamp column, which every
write to the table sets to the time of the write.

`None` for `library`, whose writes span the library model's tables,
for `library_links`, which has no change stamp, for
`configuration_changes`, an append-only log whose stamp is not
indexed, and for `playback`, which has two tables and moves only
on some of their writes; their writes bump on a probe or outright
instead.

Implements: `Clone`, `Copy`, `Debug`, `Eq`, `Hash`, `Ord`, `PartialEq`, `PartialOrd`

### MemberControl

```rust
pub enum MemberControl {
    LinkAccounts,
    Download,
    EditMetadata,
    Upload,
    RequestBooks,
    SeePrices,
}
```

One thing a patron may be allowed or refused.

**Variants**

- `LinkAccounts`: Connect a store account, or sign in to one again, here or by
  bringing it from another library. Accounts already connected keep
  working, and an Administrator can still remove them.
- `Download`: Take copies of the library's files: downloads to a branch or a
  clerk, streaming and library-card sync.
- `EditMetadata`: Change a book's metadata.
- `Upload`: Add books to the library by uploading files.
- `RequestBooks`: Ask a branch for a book it listed (interlibrary requests).
- `SeePrices`: See prices and store links in Discover and the wishlist.

#### MemberControl::ALL

```rust
pub const ALL: [Self; 6] = _
```

Every control, in the order the editors list them.

#### MemberControl::as_str

```rust
pub fn as_str(self) -> &'static str
```

The control's wire name (`link_accounts`, `download`, ...).

#### MemberControl::parse

```rust
pub fn parse(raw: &str) -> Option<Self>
```

Parses a wire name.

Implements: `Clone`, `ComposeSchema`, `Copy`, `Debug`, `Deserialize<'de>`, `Eq`, `Hash`, `Ord`, `PartialEq`, `PartialOrd`, `Serialize`, `ToSchema`

### NodeMessage

```rust
pub enum NodeMessage {
    DeploymentChanged { deployment_id: String, plugin_instance_id: String, desired: String, revision: i64 },
    InstallChanged { plugin_key: String, version: String, revision: i64 },
    PlacementChanged { subject: String, revision: i64 },
    ConfigurationChanged { scope_type: String, scope_id: String, namespace: String, revision: i64 },
    NodeRevoked { node_id: String },
    CatalogAsked {  },
    RequestsQueued {  },
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
- `ConfigurationChanged`: A configuration document changed (library settings or an instance
  document).
  - `scope_type`: Document scope type.
  - `scope_id`: Document scope id.
  - `namespace`: Document namespace.
  - `revision`: Revision after the write.
- `NodeRevoked`: A branch node's credential was revoked. The node holding that host's
  connection closes it; every claim, renew, and upload re-reads
  `node_credentials.revoked_at` as well, so a lost message does not
  keep the host connected to work.
  - `node_id`: Revoked branch node.
- `CatalogAsked`: Catalog questions wait for this branch
  (`control_plane::catalog_lookup`). The branch pulls them with
  `catalog.questions`; the message itself carries none.
- `RequestsQueued`: Interlibrary requests wait for this branch
  (`control_plane::branch_requests`). The branch claims them with
  `requests.claim`; the message itself carries none.

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

### ExplicitContent

```rust
pub enum ExplicitContent {
    Allow,
    Hide,
}
```

Whether a patron sees books marked as explicit (adult) content.

**Variants**

- `Allow`: Explicit books are visible (the default: explicit flags are
  incomplete, so hiding is a choice an Administrator makes).
- `Hide`: Books a store or metadata file marks explicit are hidden, whatever
  the include rules say; books with no flag stay visible.

#### ExplicitContent::is_allow

```rust
pub fn is_allow(&self) -> bool
```

True for [`Self::Allow`](#explicitcontent), which a stored scope leaves out.

Implements: `Clone`, `ComposeSchema`, `Copy`, `Debug`, `Default`, `Deserialize<'de>`, `Eq`, `Hash`, `Ord`, `PartialEq`, `PartialOrd`, `Serialize`, `ToSchema`

### RuleEffect

```rust
pub enum RuleEffect {
    Include,
    Exclude,
}
```

Whether a [`ScopeRule`](#scoperule) lets matching books in or keeps them out.

**Variants**

- `Include`: Matching books are visible (once any include rule exists, only books
  that match one are).
- `Exclude`: Matching books are hidden, whatever the include rules say.

Implements: `Clone`, `ComposeSchema`, `Copy`, `Debug`, `Deserialize<'de>`, `Eq`, `Hash`, `Ord`, `PartialEq`, `PartialOrd`, `Serialize`, `ToSchema`

### ScopeField

```rust
pub enum ScopeField {
    Genre,
    Series,
    Author,
    Narrator,
    Tag,
    Collection,
}
```

The book field a [`ScopeRule`](#scoperule) matches one value of.

**Variants**

- `Genre`: A genre (store category), such as "Kids".
- `Series`: A series name, such as "Discworld".
- `Author`: An author (a show's host, for a podcast).
- `Narrator`: A narrator.
- `Tag`: A library tag: one an Administrator set on the work, or one read
  from a sidecar file.
- `Collection`: A collection: a named part of the library, defined by storage key
  prefixes (`core.collections`). A book with no file is in none.

#### ScopeField::facet

```rust
pub fn facet(self) -> WorkFacet
```

The page index facet whose values the field matches.

Implements: `Clone`, `ComposeSchema`, `Copy`, `Debug`, `Deserialize<'de>`, `Eq`, `Hash`, `Ord`, `PartialEq`, `PartialOrd`, `Serialize`, `ToSchema`

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
  row and, unless it is a shared node, a live node credential.

Implements: `Clone`, `Copy`, `Debug`, `Default`, `Eq`, `PartialEq`

### NodeTrust

```rust
pub enum NodeTrust {
    All,
    OwnedBy(i64),
    AllOfOwner(i64),
    Nothing,
}
```

What a node is trusted with, resolved from the database.

**Variants**

- `All`: Every job: a shared node, or a Host owned by an active Owner or
  Administrator.
- `OwnedBy`: Only jobs whose secrets all belong to this user.
- `AllOfOwner`: Any job a node trusted with [`NodeTrust::All`](#nodetrust) may run, but only
  when it names an account and every store account that name reaches
  belongs to this user: an Owner's or Administrator's branch claiming
  only its owner's work ([`node_trust_in_scope`](#node_trust_in_scope)). Unlike
  [`NodeTrust::OwnedBy`](#nodetrust) the job's shape (an instance pin,
  requirements, its kind) does not refuse it. Only claims produce it;
  [`node_trust`](#node_trust) never does.
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

### account_slot

```rust
pub fn account_slot(plugin_key: &str, account_id: &str) -> String
```

The `bookclerk_slots` key that serializes writes to one store account's
sign-in and link.

#### Arguments

* `plugin_key` - The storefront's PluginKey.
* `account_id` - The store account id.

### link_signed_in_account

```rust
pub async fn link_signed_in_account(
    store: &LibraryStore,
    identity_id: i64,
    plugin_key: &str,
    account_id: &str,
    source: &str,
) -> Result<SignInLink>
```

Finishes an allowed store login: marks the account active with scans
on and links it to `identity_id`, in one atomic batch under the
account's write lock, only while the account's sign-in is stored and
no other identity holds it.

#### Arguments

* `store` - The library.
* `identity_id` - The portal identity the login ran for.
* `plugin_key` - The storefront's PluginKey.
* `account_id` - The store account the login signed in to.
* `source` - The storefront alias recorded on the link (display only).

#### Returns

What happened; see [`SignInLink`](#signinlink).

#### Errors

Returns the write error.

### revoke_account_link

```rust
pub async fn revoke_account_link(
    store: &LibraryStore,
    identity_id: i64,
    plugin_key: &str,
    account_id: &str,
) -> Result<LinkRevoked>
```

Revokes the account `identity_id` holds: in one atomic batch under the
account's write lock, marks it `revoked` with scans off, deletes its
integration secrets, forgets which branch kept its login for this
library (its `node:` shares and their sign-in revisions), and removes
the link. Every statement applies only while the identity's link
exists, and the link goes last, so the batch refuses as a whole once
the link is gone.

The best-effort store sign-out, and the removal of the account's
holdings and scan checkpoints ([`LibraryStore::revoke_credentials`](README.md#librarystorerevoke_credentials)),
run before it while the link still exists, so a failure there can be
retried.

#### Arguments

* `store` - The library.
* `identity_id` - The portal identity revoking its link.
* `plugin_key` - The storefront's PluginKey.
* `account_id` - The store account id.

#### Returns

What happened; see [`LinkRevoked`](#linkrevoked).

#### Errors

Returns the write error. The batch may have committed when the reply
was lost; retrying then answers [`LinkRevoked::NotLinked`](#linkrevoked).

### admin_control_overrides

```rust
pub async fn admin_control_overrides(
    store: &LibraryStore,
    user_id: i64,
) -> Result<AdminControlOverrides>
```

The overrides stored for `user_id` (none when there is no row).

#### Errors

Returns the read error, or an error when the stored row is invalid.

### admin_may

```rust
pub async fn admin_may(
    store: &LibraryStore,
    user_id: i64,
    control: AdminControl,
) -> Result<bool>
```

Whether `user_id` may use `control` here. An unknown user may not.

#### Errors

Returns a read or parse error; callers fail closed on it.

### administrator_switch_allows

```rust
pub async fn administrator_switch_allows(
    store: &LibraryStore,
    user_id: i64,
    control: AdminControl,
) -> Result<bool>
```

Whether an Administrator's switch lets `user_id` do `control`, for an
action other roles are allowed or refused by their own rules: true for
any role but Administrator (and for an active Administrator whose
switch is on), false for an unknown user.

#### Errors

Returns a read or parse error; callers fail closed on it.

### assigned_admin_controls

```rust
pub async fn assigned_admin_controls(
    store: &LibraryStore,
    user: &UserRecord,
) -> Result<AssignedAdminControls>
```

The switches of `user`: the library's default, their overrides, and
what applies to them.

#### Errors

Returns a read or parse error; callers fail closed on it.

### change_admin_control_overrides

```rust
pub async fn change_admin_control_overrides(
    store: &LibraryStore,
    user_id: i64,
    changes: &OverrideSwitchChanges,
) -> Result<AssignedAdminControls>
```

Changes only the named overrides of `user_id`: each attempt applies
`changes` to the row it read and writes it back only if the row is
still what it read, so a concurrent edit of another switch is kept
rather than overwritten.

The caller has already checked that the actor is the operator or an
Owner. With no override left, the row is deleted and the Administrator
follows the library's default for every switch.

#### Errors

Returns [`LibraryError::NotFound`](README.md#libraryerror) for an unknown user,
[`LibraryError::Conflict`](README.md#libraryerror) when the row keeps changing underneath, or
the read or write error.

### change_default_admin_controls

```rust
pub async fn change_default_admin_controls(
    store: &LibraryStore,
    actor: &ConfigActor,
    changes: &DefaultSwitchChanges,
) -> Result<ConfigurationDocument<AdminControls>>
```

Changes only the named default switches under compare-and-swap: each
attempt applies `changes` to the stored default it read, so a
concurrent edit of another switch is kept rather than overwritten.

#### Arguments

* `store` - The library database.
* `actor` - Who is writing: the operator or a verified Owner
  ([`super::verify_library_owner`](#verify_library_owner)).
* `changes` - The switches to change and their new values.

#### Errors

Returns an error when the actor may not write, or
[`LibraryError::Conflict`](README.md#libraryerror) when the document keeps changing underneath.

### default_admin_controls

```rust
pub async fn default_admin_controls(store: &LibraryStore) -> Result<AdminControls>
```

The library's default Administrator switches: the stored document, or
every switch on.

#### Errors

Returns the read or parse error.

### effective_admin_controls

```rust
pub fn effective_admin_controls(
    user: &UserRecord,
    defaults: AdminControls,
    overrides: AdminControlOverrides,
) -> AdminControls
```

What applies to `user` given the library's default and their
overrides: everything for an active Owner, the overrides over the
default for an active Administrator, and nothing for anyone else (a
user who is not active or a Member).

### load_default_admin_controls

```rust
pub async fn load_default_admin_controls(
    store: &LibraryStore,
) -> Result<Option<ConfigurationDocument<AdminControls>>>
```

Loads the library's default Administrator switches document.

#### Returns

The stored document, or `None` when nobody has written one: every
switch is then on.

#### Errors

Returns the read error, or an error when the stored document is invalid
or of an unknown schema.

### set_admin_control_overrides

```rust
pub async fn set_admin_control_overrides(
    store: &LibraryStore,
    user_id: i64,
    overrides: AdminControlOverrides,
) -> Result<AssignedAdminControls>
```

Replaces the overrides of `user_id`; with none, the Administrator
follows the library's default for every switch again.

The caller has already checked that the actor is the operator or an
Owner. One atomic batch updates the stored row, inserts it when there is
none, or deletes it when no switch is overridden.

#### Errors

Returns [`LibraryError::NotFound`](README.md#libraryerror) for an unknown user, or the read or
write error.

### set_default_admin_controls

```rust
pub async fn set_default_admin_controls(
    store: &LibraryStore,
    actor: &ConfigActor,
    controls: AdminControls,
) -> Result<ConfigurationDocument<AdminControls>>
```

Replaces the library's default Administrator switches under
compare-and-swap, creating the document when absent and retrying a few
times on a concurrent edit.

#### Arguments

* `store` - The library database.
* `actor` - Who is writing: the operator or a verified Owner
  ([`super::verify_library_owner`](#verify_library_owner)).
* `controls` - The new default.

#### Errors

Returns an error when the actor may not write, or
[`LibraryError::Conflict`](README.md#libraryerror) when the document keeps changing underneath.

### load_artifact_blob

```rust
pub async fn load_artifact_blob(
    store: &LibraryStore,
    sha256: &str,
) -> Result<Option<Vec<u8>>>
```

Reads an archive stored by [`put_artifact_blob`](#put_artifact_blob).

Chunks are read in order, as many per request as the database plugin's
result limits allow.

#### Returns

The archive bytes, or `None` when no archive with that digest is stored.

#### Errors

Returns an error when a read fails, a chunk is missing or the wrong
size, or the stored bytes do not match their digest.

### put_artifact_blob

```rust
pub async fn put_artifact_blob(store: &LibraryStore, bytes: &[u8]) -> Result<String>
```

Stores a small archive and returns its `blob:sha256:<hex>` URL.

The bytes go in chunks sized from the database plugin's advertised
limits, over as many atomic batches as those limits need; the header row
goes in the last one. Storing an archive that is already stored writes
nothing.

#### Errors

Returns an error when the archive exceeds [`MAX_ARTIFACT_BLOB_BYTES`](#max_artifact_blob_bytes) or
a write fails. Chunks from batches written before the failure stay but
are invisible without their header, and a retry reuses them.

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
this library already holds for the branch's owner is skipped. Each
other title is resolved into the library model and held by the branch
(`holding_added` when new), and the holding's share state is written
in one transaction with the cursor, each statement gated on the fence
still being live, the branch still sharing and its credential live. A
holding is stamped with the pass under way, and with `done` (the last
batch of a pass) while a pass from the start is under way, the
branch's holdings the pass did not name are removed (`withdrawn`) and
the pass ends; `done` on a resumed incremental pass changes nothing
more.

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
With sharing off, every title the branch shared is removed
(`stopped_sharing`).

#### Errors

Returns an error when a read or write fails.

### branch_provenance

```rust
pub fn branch_provenance(node_id: &str) -> String
```

The provenance text of titles shared by `node_id`: the holder key of
its node holdings.

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

Removes every holding of `node_id` (it stopped sharing, left or was
revoked; `reason` is logged), each with `holding_removed`; answers how
many went. Files already delivered stay with their editions.

#### Errors

Returns the read or write error.

### forwardable_branch_title

```rust
pub async fn forwardable_branch_title(
    store: &LibraryStore,
    id: &str,
) -> Result<Option<ForwardableBranchTitle>>
```

On a branch: the title shared through node holding `id` when it may
still be passed on (still held, files of its current manifest
delivered).

#### Arguments

* `store` - This install's library.
* `id` - The holding id, from a [`ForwardableBranchKey`](#forwardablebranchkey).

#### Returns

The title, or `None` when the holding is gone or not delivered.

#### Errors

Returns the read error.

### forwardable_branch_title_keys_page

```rust
pub async fn forwardable_branch_title_keys_page(
    store: &LibraryStore,
    after_id: Option<&str>,
    limit: u64,
) -> Result<Vec<ForwardableBranchKey>>
```

On a branch: one keyset page of the titles its own branches share with
it, identity columns only, by holding id.

Only keys with [`ForwardableBranchKey::delivered`](#forwardablebranchkey) set may be passed on.

#### Arguments

* `store` - This install's library.
* `after_id` - The last [`ForwardableBranchKey::id`](#forwardablebranchkey) of the previous
  page, or `None` for the first page.
* `limit` - Rows wanted, clamped to 1..=[`FORWARDABLE_KEYS_MAX_PAGE`](#forwardable_keys_max_page).

#### Returns

Up to `limit` keys in ascending id order; fewer than `limit` means the
last page.

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

Granted only while the fence is live, for a title the branch holds
here, when every file is copied to each of `destinations` (this node's,
the primary first) under its manifest key, the files are exactly the
shared manifest, the title's edition has no audio here from another
writer (`edition_has_files`), no key is one another title here already
uses (`key_in_use`: another edition's file, or a file beside one of its
audio files with the same stem; `key_of_another_title`: any of a
library book's keys or their sidecars), and no key lies in or below a
folder holding another title's audio (`folder_of_another_book`). The
keys are then claimed for this upload as the edition's in
`edition_files`, refused when another title holds one there.

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

Titles branches share, of one branch or all, by holding id, at most
`limit`.

#### Errors

Returns the read error.

### listing_from_catalog_title

```rust
pub fn listing_from_catalog_title(
    title: &CatalogTitle,
    plugin_key: &str,
) -> Result<Listing, ListingError>
```

The listing a shared title describes, filed under `plugin_key`: its
metadata, ASIN scoped to its marketplace, ISBN, and media kind from
its content kind.

#### Errors

Returns [`resolve::ListingError`](../bookclerk_resolve/listing.md#listingerror) when the title fails listing
validation.

### pending_deliveries

```rust
pub async fn pending_deliveries(
    store: &LibraryStore,
    node_id: &str,
    limit: usize,
) -> Result<Vec<CatalogRef>>
```

Up to `limit` of `node_id`'s shared titles whose files have not
arrived (or changed since), leaving out titles whose edition already
has audio here from another writer.

#### Errors

Returns the read error.

### remove_detached_branch_titles

```rust
pub async fn remove_detached_branch_titles(
    store: &LibraryStore,
    node_id: Option<&str>,
) -> Result<u64>
```

Forgets the files branches left behind (entries "from a branch that
left" and the like), of one branch or of every one: the
`edition_files` records its catalog uploads wrote for editions nothing
here holds any more. Answers how many editions' entries went. The files
themselves stay in this library's outputs.

#### Errors

Returns the read or write error.

### parse_members_books_request

```rust
pub fn parse_members_books_request(body: &[u8]) -> Result<MembersBooksChoice>
```

Parses and validates a `PUT /api/config/branches/members-books` body
from its raw bytes, sans IO, the way the route reads it: JSON into
[`MembersBooksBody`](#membersbooksbody), then [`MembersBooksBody::validate`](#membersbooksbodyvalidate).

#### Errors

Returns [`LibraryError::Invalid`](README.md#libraryerror) for bytes that are not the body's JSON
shape (unknown or repeated fields, wrong types, a missing field) or a
malformed node id.

### set_branch_members_books

```rust
pub async fn set_branch_members_books(
    store: &LibraryStore,
    node_id: &str,
    on: bool,
) -> Result<Option<bool>>
```

Turns "Acquire all of this branch's members' books" on or off for
branch `node_id`.

A change also clears the branch's catalog cursor and bumps its catalog
fence: the pass under way is lost, and the branch's next catalog
session (at most one sync interval later) starts from the start with
the new choice. Setting the value it already has changes nothing.

#### Arguments

* `node_id` - The branch's node id.
* `on` - The new choice.

#### Returns

`Some(on)` once stored, or `None` when `node_id` is not a branch (host
node) of this library.

#### Errors

Returns [`LibraryError::Unavailable`](README.md#libraryerror) when the branch's catalog fence
kept changing under the write, or the read or write error.

### books_requested_by_link

```rust
pub async fn books_requested_by_link(
    store: &LibraryStore,
    library_id: &str,
) -> Result<BTreeMap<HeldBookKey, DateTime<Utc>>>
```

On a branch: every book the library `library_id` asked for, by store
identity, with when it last asked (an unreadable stamp reads as now, so
the book is sent rather than skipped). Read in keyset pages no larger
than the database plugin's `maxResultRows`.

#### Errors

Returns an error when a read fails.

### claim_branch_requests

```rust
pub async fn claim_branch_requests(
    store: &LibraryStore,
    node_id: &str,
) -> Result<Option<Vec<BranchRequest>>>
```

Claims the requests waiting for branch `node_id`, oldest first.

A request is waiting while its job is pending, pinned to this branch,
and due. Each is leased to the branch ([`remote_lease_owner`](#remote_lease_owner)) for
[`REQUEST_LEASE_SECS`](#request_lease_secs). A request whose work has no identifier left to
ask by is failed (`work_gone`) instead of sent.

#### Returns

`None` when `node_id` is not a branch this library trusts; otherwise
at most [`MAX_CLAIMED_REQUESTS`](../bookclerk_node_proto/README.md#max_claimed_requests) claimed requests.

#### Errors

Returns a read or write error.

### finish_branch_request

```rust
pub async fn finish_branch_request(
    store: &LibraryStore,
    node_id: &str,
    job_id: &str,
    generation: i64,
    outcome: RequestOutcome,
) -> Result<RequestFinished>
```

Records what branch `node_id` did with the request `job_id` it claimed
at lease `generation`.

#### Errors

Returns a read or write error.

### notify_requests_queued

```rust
pub async fn notify_requests_queued(store: &LibraryStore, node_id: &str)
```

Wakes branch `node_id` after a request for it was admitted. Best
effort: the branch also claims on every connection and on a timer.

### parse_branch_request

```rust
pub fn parse_branch_request(body: &[u8]) -> Result<BranchRequestAsk>
```

Parses and validates a `POST /api/branch-requests` body from its raw
bytes, sans IO, the way the route reads it: JSON into
[`BranchRequestBody`](#branchrequestbody), then [`BranchRequestBody::validate`](#branchrequestbodyvalidate).

#### Errors

Returns [`LibraryError::Invalid`](README.md#libraryerror) for bytes that are not the body's JSON
shape (unknown or repeated fields, wrong types, a missing `work_id`) or a
malformed id.

### plan_branch_request

```rust
pub async fn plan_branch_request(
    store: &LibraryStore,
    work_id: &str,
    node_id: Option<&str>,
    trigger: JobTrigger,
) -> Result<BranchRequestPlan>
```

Plans an interlibrary request for `work_id`.

The listing comes from [`super::branch_listings_for_works`]: answers a
trusted branch gave within [`super::BRANCH_LISTING_TTL`](#branch_listing_ttl). With
`node_id`, only that branch's listings count; without, the first branch
in node order. Within a branch the first listing (store, product id)
is asked for; the branch picks which of its people's copies to send.

#### Arguments

* `work_id` - This library's work.
* `node_id` - The branch to ask, or `None` for any that lists the work.
* `trigger` - Who asked (`api` for a member, `scheduler` for an
  automatic request).

#### Errors

Returns [`LibraryError::NotFound`](README.md#libraryerror) when no trusted branch (or not the
named one) currently lists the work, or the read error.

### record_link_request

```rust
pub async fn record_link_request(
    store: &LibraryStore,
    library_id: &str,
    title: &AcquireTitle,
) -> Result<()>
```

On a branch: records that the library `library_id` asked for `title`
(by its holding account and store listing), so the catalog sync sends
it there. Asking again moves the request to now.

#### Errors

Returns an error when the write fails.

### is_cache_setting_key

```rust
pub fn is_cache_setting_key(key: &str) -> bool
```

True when `key` is one of [`CACHE_SETTING_KEYS`](#cache_setting_keys).

#### Arguments

* `key` - A dotted settings key such as `cache.budget_bytes`.

### load_cache_settings

```rust
pub async fn load_cache_settings(
    store: &LibraryStore,
) -> Result<Option<ConfigurationDocument<CacheSettingsV1>>>
```

Loads the library cache document.

#### Returns

The stored document, or `None` when nobody has written one yet: the
defaults of `CacheSettingsV1::default` then apply.

#### Errors

Returns an error when the database read fails or the stored document is
unsupported or invalid.

### update_cache_settings

```rust
pub async fn update_cache_settings(
    store: &LibraryStore,
    actor: &ConfigActor,
    edit: impl Fn(&mut CacheSettingsV1) -> Result<()>,
) -> Result<ConfigurationDocument<CacheSettingsV1>>
```

Applies `edit` to the library cache document under compare-and-swap,
creating it with the defaults when absent and retrying a few times on a
concurrent edit. Nodes pick the change up at their next heartbeat.

#### Arguments

* `store` - The library database.
* `actor` - Who is writing, for authorization and the audit log.
* `edit` - Changes the body; an error aborts without writing.

#### Errors

Returns the error `edit` returned, an error when the edited body is
invalid or the actor may not write, or [`LibraryError::Conflict`](README.md#libraryerror) when
the document keeps changing underneath.

### account_share_peers

```rust
pub async fn account_share_peers(
    store: &LibraryStore,
    plugin_key: &str,
    account_id: &str,
) -> Result<Vec<String>>
```

Every peer this install shared the store account with, in either
direction (`account_shares`): [`node_peer`](#node_peer) and [`link_peer`](#link_peer) values,
each once, sorted.

#### Errors

Returns the read error.

### accounts_to_carry

```rust
pub async fn accounts_to_carry(
    store: &LibraryStore,
    user_id: i64,
    shareable: &dyn Fn(&str) -> bool + Send + Sync,
) -> Result<Vec<CarriedAccount>>
```

The store accounts `user_id` connected here that may travel to a
branch ([`member_accounts`](#member_accounts) whose plugin `shareable` accepts), each
with its secrets unsealed. At most [`MAX_BUNDLE_ACCOUNTS`](../bookclerk_node_proto/README.md#max_bundle_accounts); an account
without a secret is left out (there is nothing to sign in with).

#### Arguments

* `store` - This library.
* `user_id` - The member who chose to bring their connections.
* `shareable` - Whether a plugin key's store credentials may be held by
  several instances (`credentials = "shareable"` in its `plugin.toml`).

#### Errors

Returns a read error, or the unseal error of a secret.

### accounts_to_carry_where

```rust
pub async fn accounts_to_carry_where(
    store: &LibraryStore,
    user_id: i64,
    keep: &dyn Fn(&str, &str) -> bool + Send + Sync,
) -> Result<Vec<CarriedAccount>>
```

[`accounts_to_carry`](#accounts_to_carry) limited to the accounts `keep` accepts, by
`(plugin_key, account_id)`, applied before the bundle's count and size
limits so a picked account is never crowded out by unpicked ones.

#### Arguments

* `store` - This library.
* `user_id` - The member whose connections travel.
* `keep` - Whether an account may travel (its storefront shareable,
  and picked when the person picks some).

#### Errors

Returns a read error, or the unseal error of a secret.

### branch_person

```rust
pub async fn branch_person(
    store: &LibraryStore,
    preferred: Option<i64>,
) -> Result<Option<i64>>
```

The person on this install that store accounts brought from a library
belong to: `preferred` (the Owner who elevated into the operator
session that ran the join) while that user is still an active Owner,
otherwise the only
active Owner. `None` when neither settles it, for example on a new
install with no Owner yet; the accounts are then not kept.

#### Errors

Returns the read error.

### identity_may_connect_accounts

```rust
pub async fn identity_may_connect_accounts(
    store: &LibraryStore,
    identity_id: i64,
) -> Result<bool>
```

Whether the portal identity `identity_id` may connect store accounts
here: the rule of the user it belongs to ([`member_may_connect_accounts`](#member_may_connect_accounts)).
An identity that belongs to no first-party user (an integration's
external user) has no member rule and may connect, as before; an
unknown identity may not.

#### Errors

Returns the read error.

### import_carried_accounts

```rust
pub async fn import_carried_accounts(
    store: &LibraryStore,
    user_id: i64,
    library_id: &str,
    accounts: &[CarriedAccount],
) -> Result<CarriedImport>
```

Imports the accounts a library sent with its person, for `user_id`,
recording that they came from the library `library_id`.

Refused whole, storing nothing, when this instance's rule does not let
the person connect accounts ([`member_may_connect_accounts`](#member_may_connect_accounts)). Each
batch checks the rule again, so a rule turned off midway stops the
accounts not yet kept. Each
other account is one atomic batch that inserts the account (scanning
on, connected), writes its secrets re-sealed with this install's key,
links it to the person's local identity and records the share. Every
statement is guarded on the account having no link yet, so an account
connected here before (to the person or to someone else) is never
overwritten, even when it is connected while the import runs.

#### Errors

Returns a read error from before any account is written. An account
whose own write fails is reported in [`CarriedImport::failed`](#carriedimport).

### import_offered_accounts

```rust
pub async fn import_offered_accounts(
    store: &LibraryStore,
    user_id: i64,
    node_id: &str,
    accounts: &[CarriedAccount],
) -> Result<CarriedImport>
```

Imports the accounts a branch offered up its link ("Add my connections
to *library*") for `user_id`, the branch's owner, recording that they
came from the node `node_id`.

The same rules as [`import_carried_accounts`](#import_carried_accounts) apply: refused whole
under this library's rule, each account in its own guarded batch, and
an account connected here before is never overwritten.

#### Errors

As [`import_carried_accounts`](#import_carried_accounts).

### link_peer

```rust
pub fn link_peer(library_id: &str) -> String
```

The `account_shares.peer` of a library this install received accounts
from.

### member_accounts

```rust
pub async fn member_accounts(
    store: &LibraryStore,
    user_id: i64,
) -> Result<Vec<MemberAccount>>
```

The store accounts `user_id` has connected here: linked to any of the
user's portal identities and still connected (`connection_status` is
`active`), sorted by plugin key and account id.

#### Errors

Returns the read error.

### member_may_connect_accounts

```rust
pub async fn member_may_connect_accounts(
    store: &LibraryStore,
    user_id: i64,
) -> Result<bool>
```

Whether `user_id` may connect store accounts on this instance: an
active user whose member controls allow it
([`super::MemberControl::LinkAccounts`](#membercontrol)).

#### Errors

Returns the read error.

### node_peer

```rust
pub fn node_peer(node_id: &str) -> String
```

The `account_shares.peer` of a branch node this library sent accounts to.

### record_accounts_sent

```rust
pub async fn record_accounts_sent(
    store: &LibraryStore,
    node_id: &str,
    accounts: &[CarriedAccount],
) -> Result<()>
```

Records that `accounts` were sent to the branch `node_id`
(`account_shares`, direction [`SHARE_SENT`](#share_sent)), in atomic batches of at
most 13 accounts; a repeat refreshes the time.

#### Errors

Returns the write error.

### record_accounts_sent_to

```rust
pub async fn record_accounts_sent_to(
    store: &LibraryStore,
    peer: &str,
    accounts: &[CarriedAccount],
) -> Result<()>
```

Records that `accounts` were sent to `peer` ([`node_peer`](#node_peer) for a
branch, [`link_peer`](#link_peer) for a library this install offered them to), as
[`record_accounts_sent`](#record_accounts_sent) does.

Sending an account to a library again also forgets which sign-in this
install last pushed there (`account_credential_revisions.observed`),
keeping the revision: the library may have forgotten its keeper in
between (the account was disconnected everywhere there), so the next
push pass sends the current sign-in again even when it hasn't changed,
which confirms this install as the keeper anew.

#### Errors

Returns the write error.

### revoke_unlinked_account

```rust
pub async fn revoke_unlinked_account(
    store: &LibraryStore,
    plugin_key: &str,
    account_id: &str,
) -> Result<()>
```

Takes back a store account a refused login stored: deletes its secrets
and marks it `revoked` with scans off, in one atomic batch whose every
statement applies only while no link holds the account.

The batch takes the account's write lock
([`super::account_slot`](#account_slot)) first. A link committed before
it (another login of the same store account that finished meanwhile)
keeps the account and its secrets as they are; an allowed login that
links after it finds the sign-in gone
([`super::SignInLink::SignInTakenBack`](#signinlink)) instead of linking an account
with no secret.

#### Errors

Returns the write error.

### withdraw_person_account

```rust
pub async fn withdraw_person_account(
    store: &LibraryStore,
    user_id: i64,
    plugin_key: &str,
    account_id: &str,
) -> Result<Withdrawn>
```

Disconnects a store account "everywhere" on this library for the
person `user_id`: removes every link from their identities to the
account and, in the same atomic batch, deletes its secrets and marks it
`revoked` with scans off when no other link holds it. A revoked account
then loses its holdings (each with a `holding_removed` event) and its
scan checkpoints, as [`LibraryStore::revoke_credentials`](README.md#librarystorerevoke_credentials) does.

An account another member is linked to keeps that member's link,
secrets and status.

#### Errors

Returns a read or write error.

### answer_questions

```rust
pub async fn answer_questions(
    store: &LibraryStore,
    questions: &[LookupWork],
    list_members_books: bool,
) -> Result<(Vec<LookupAnswer>, Vec<String>)>
```

Answers questions a library sent this branch from this branch's own
library.

Each question is mapped to a work here by identifier
(`work_for_question`); the answer lists the books of that work this
library holds whose holder may be listed (the owner's always; members'
only with `list_members_books` on and when they did not opt out), at
most [`MAX_LOOKUP_EDITIONS`](../bookclerk_node_proto/README.md#max_lookup_editions). A question with no such book is answered
as missing.

#### Arguments

* `questions` - The library's questions (already validated).
* `list_members_books` - The link's "List my members' books" setting.

#### Returns

The answers and the missing works, together naming every question.

#### Errors

Returns the read error.

### ask_branches

```rust
pub async fn ask_branches(store: &LibraryStore, work_ids: &[String]) -> Result<AskReport>
```

Asks every trusted branch about `work_ids`.

A work a branch was never asked about gets a new question; one whose
answer is older than [`LOOKUP_ANSWER_TTL`](#lookup_answer_ttl) is asked again; one waiting
for its answer, or answered recently, is left alone. A work id this
library does not have is skipped. Each branch with a new question gets a
`catalog.asked` wakeup.

#### Arguments

* `work_ids` - Works of this library to ask about.

#### Errors

Returns a read or write error.

### branch_listing_key

```rust
pub fn branch_listing_key(node_id: &str) -> String
```

The plugin key of listings answered by branch `node_id`.

### branch_listings_for_works

```rust
pub async fn branch_listings_for_works(
    store: &LibraryStore,
    work_ids: &[String],
) -> Result<Vec<BranchListing>>
```

The current branch listings of `work_ids`: those a branch this library
still trusts named within [`BRANCH_LISTING_TTL`](#branch_listing_ttl).

#### Errors

Returns the read error.

### branch_offers_for_works

```rust
pub async fn branch_offers_for_works(
    store: &LibraryStore,
    work_ids: &[String],
) -> Result<HashMap<String, Vec<BranchOffer>>>
```

The branches each of `work_ids` is currently available through.

Built from [`branch_listings_for_works`]: only answers a trusted branch
gave within [`BRANCH_LISTING_TTL`](#branch_listing_ttl) count. A work with no current branch
listing is left out of the map. Each work's offers are ordered by node,
then store and product id, with duplicates removed.

#### Errors

Returns the read error.

### catalog_listing_always

```rust
pub async fn catalog_listing_always(store: &LibraryStore, user_id: i64) -> Result<bool>
```

True when `user_id` is an active Owner of this branch, whose books are
always listed whatever their opt-out says.

#### Errors

Returns the read error.

### catalog_listing_opted_out

```rust
pub async fn catalog_listing_opted_out(store: &LibraryStore, user_id: i64) -> Result<bool>
```

True when member `user_id` keeps their books out of this branch's
listings.

#### Errors

Returns the read error.

### pending_questions

```rust
pub async fn pending_questions(
    store: &LibraryStore,
    node_id: &str,
) -> Result<Option<Vec<LookupWork>>>
```

The questions waiting for branch `node_id`, marked sent.

A question is waiting while it has no answer and was never sent or was
sent more than [`LOOKUP_RESEND`](#lookup_resend) ago. A work with no identifier to ask
by is marked answered (there is nothing to ask), so it does not come
back until it is asked again.

#### Returns

`None` when `node_id` is not a branch this library trusts; otherwise at
most [`MAX_LOOKUP_WORKS`](../bookclerk_node_proto/README.md#max_lookup_works) questions, oldest first.

#### Errors

Returns a read or write error.

### record_answers

```rust
pub async fn record_answers(
    store: &LibraryStore,
    node_id: &str,
    answers: &[LookupAnswer],
    missing: &[String],
) -> Result<AnswersRecorded>
```

Records branch `node_id`'s answers to the questions it was sent.

Only works with a question waiting for this branch are settled; an
answer for any other work is dropped. Each answered edition is stored
as a `branch:<node>` catalog listing when the resolver places it on a
stored edition of the asked work (through a shared identifier, never a
title); other editions are dropped. Branch listings of a settled work
the answer no longer names are expired, and the settled questions are
marked answered.

#### Arguments

* `answers` - Works the branch can supply.
* `missing` - Works it looked up and cannot supply.

#### Errors

Returns [`LibraryError::Invalid`](README.md#libraryerror) for a malformed answer, and a read or
write error.

### set_catalog_listing_opt_out

```rust
pub async fn set_catalog_listing_opt_out(
    store: &LibraryStore,
    user_id: i64,
    opted_out: bool,
) -> Result<()>
```

Records whether member `user_id`'s books are kept out of what this
branch lists to the libraries it joined.

#### Errors

Returns the write error.

### title_for_request

```rust
pub async fn title_for_request(
    store: &LibraryStore,
    request: &BranchRequest,
    list_members_books: bool,
) -> Result<Option<AcquireTitle>>
```

The title of this library that fulfils an interlibrary request: a
title this branch would list for the request's work
([`answer_questions`](#answer_questions)'s rules: the owner's always, members' only with
`list_members_books` on and when they did not opt out) whose store,
marketplace and product id are the request's. The first such holding
wins. `None` when this branch no longer lists that edition to the
library.

#### Arguments

* `request` - The claimed request (already validated).
* `list_members_books` - The link's "List my members' books" setting.

#### Errors

Returns the read error.

### trusted_branches

```rust
pub async fn trusted_branches(store: &LibraryStore) -> Result<Vec<String>>
```

The branches this library trusts, by node id.

#### Errors

Returns the read error.

### works_of_interest

```rust
pub async fn works_of_interest(store: &LibraryStore, limit: u64) -> Result<Vec<String>>
```

The works this library wants its branches asked about: those of open
wishes (most recently changed first), then those of Discover candidates
(most recently fetched first), at most `limit`.

#### Errors

Returns the read error.

### works_of_listings

```rust
pub async fn works_of_listings(
    store: &LibraryStore,
    listings: &[(String, String)],
) -> Result<HashMap<(String, String), String>>
```

The work of each store listing in `listings`, by `(plugin key, product
id)`, for listings the shared pool already holds.

Product ids are compared in their stored spelling
([`resolve::stored_product_id`](../bookclerk_resolve/listing.md#stored_product_id)); the marketplace is not compared, so a
product sold in two marketplaces maps to the work of either listing.
Branch listings (`branch:<node>`) are never matched. A listing the pool
does not hold, or whose edition has no work, is left out.

#### Arguments

* `store` - The library.
* `listings` - `(plugin key, product id)` pairs as the storefront
  reports them.

#### Errors

Returns the read error.

### forget_clerk_link

```rust
pub async fn forget_clerk_link(store: &LibraryStore, library_id: &str) -> Result<bool>
```

Deletes a signed-out link's row, so it leaves the list. Answers false
when there was no such link.

#### Errors

Returns [`LibraryError::Conflict`](README.md#libraryerror) while the link is active (sign out
first), and the write error.

### list_clerk_links

```rust
pub async fn list_clerk_links(store: &LibraryStore) -> Result<Vec<ClerkLink>>
```

Every clerk link, oldest sign-in first.

#### Errors

Returns the read error.

### load_clerk_card

```rust
pub async fn load_clerk_card(
    store: &LibraryStore,
    link: &ClerkLink,
) -> Result<LibraryCard>
```

Unseals the card of an active `link`.

The caller registers the card with the log redactor before using it.

#### Errors

Returns [`LibraryError::Forbidden`](README.md#libraryerror) when the link is signed out,
[`LibraryError::NotFound`](README.md#libraryerror) when the card row is missing, and
[`LibraryError::Invalid`](README.md#libraryerror) when it does not hold a card.

### load_clerk_link

```rust
pub async fn load_clerk_link(
    store: &LibraryStore,
    library_id: &str,
) -> Result<Option<ClerkLink>>
```

One clerk link.

#### Errors

Returns the read error.

### record_clerk_link

```rust
pub async fn record_clerk_link(
    store: &LibraryStore,
    link: &NewClerkLink,
    card: &LibraryCard,
) -> Result<ClerkLink>
```

Seals `card` and records the link as active, replacing an earlier
sign-in to the same library.

The card row is written first, so a link row never names a card that
is not stored; a card row left by a failed write is replaced by the
next sign-in.

#### Errors

Returns [`LibraryError::Invalid`](README.md#libraryerror) for a malformed library id, origin,
pin, name or user id, [`LibraryError::Conflict`](README.md#libraryerror) when this install is
already an active branch of that library (it reads that
library's books through the link instead), an error when no master key
is configured, and the write error.

### sign_out_clerk_link

```rust
pub async fn sign_out_clerk_link(
    store: &LibraryStore,
    library_id: &str,
) -> Result<Option<ClerkLink>>
```

Marks a link signed out, deletes its card and drops the books read
through it. Answers the link as it now stands, or `None` when there is
no such link.

Used when the library ends the card (close 4401 or a 401 answer) and
when the person signs out here.

#### Errors

Returns the read or write error.

### collections_revision

```rust
pub async fn collections_revision(store: &LibraryStore) -> Result<i64>
```

The collections document's revision, 0 when it was never written. The
page index rebuild compares it before and after its walk.

#### Errors

See [`load_collections`](#load_collections).

### library_collections

```rust
pub async fn library_collections(store: &LibraryStore) -> Result<LibraryCollections>
```

The library's collections; none when the document was never written.

#### Errors

See [`load_collections`](#load_collections).

### load_collections

```rust
pub async fn load_collections(
    store: &LibraryStore,
) -> Result<Option<ConfigurationDocument<LibraryCollections>>>
```

Reads the stored collections document, if one was ever written.

#### Errors

Returns the read error, or [`LibraryError::Invalid`](README.md#libraryerror) for a stored
document this binary can't read.

### parse_collections_request

```rust
pub fn parse_collections_request(body: &[u8]) -> Result<LibraryCollections>
```

Parses and validates a `PUT /api/library/collections` body from its raw
bytes, sans IO, the way the route reads it: JSON into
[`LibraryCollections`](#librarycollections), then [`LibraryCollections::validate`](#librarycollectionsvalidate).

#### Errors

Returns [`LibraryError::Invalid`](README.md#libraryerror) for bytes that are not the body's JSON
shape (unknown or repeated fields, wrong types, a missing field, an
array instead of an object) or collections outside the bounds.

### set_library_collections

```rust
pub async fn set_library_collections(
    store: &LibraryStore,
    actor: &ConfigActor,
    collections: LibraryCollections,
) -> Result<ConfigurationDocument<LibraryCollections>>
```

Replaces the library's collections (compare-and-swap, audited) and,
when they changed, drops the page index's finished build so every row
is rebuilt with the new prefixes.

The caller checks who may write them; `actor` must be the operator or a
verified Owner or Administrator.

#### Errors

Returns [`LibraryError::Invalid`](README.md#libraryerror) for collections outside the bounds,
[`LibraryError::Conflict`](README.md#libraryerror) when the document kept changing, or the write
error.

### parse_add_connections_request

```rust
pub fn parse_add_connections_request(body: &[u8]) -> Result<AddConnectionsRequest>
```

Parses and validates a `POST /api/config/join/connections` body from its
raw bytes, sans IO, the way the route reads it: JSON into
[`AddConnectionsBody`](#addconnectionsbody), then [`AddConnectionsBody::validate`](#addconnectionsbodyvalidate).

#### Errors

Returns [`LibraryError::Invalid`](README.md#libraryerror) for bytes that are not the body's JSON
shape (not an object, an account that is not an object, unknown or
duplicate fields, missing fields, wrong types) or that name no account
or more than [`MAX_ADD_CONNECTIONS`](#max_add_connections).

### parse_disconnect_request

```rust
pub fn parse_disconnect_request(body: &[u8]) -> Result<DisconnectRequest>
```

Parses a `POST /api/config/join/connections/disconnect` body from its
raw bytes, sans IO, the way the route reads it: JSON into
[`DisconnectBody`](#disconnectbody), then [`DisconnectBody::into_request`](#disconnectbodyinto_request).

#### Errors

Returns [`LibraryError::Invalid`](README.md#libraryerror) for bytes that are not the body's JSON
shape: not an object, unknown or duplicate fields, a missing
`plugin_key` or `account_id`, or a value of the wrong type (`everywhere`
is a boolean when present; `null` is refused).

### authorize_copy

```rust
pub async fn authorize_copy(
    store: &LibraryStore,
    copy_id: &str,
    token: &str,
) -> Result<Option<CopyDownload>>
```

The files `token` may download for copy `copy_id`, or `None`.

The token's hash must match and it must not have expired; and, read
afresh on every call, the holder must still be one this library gives
copies to for the same person, the person must still be allowed the
edition (`visible_to`), and the edition's files on the granted
destination must be the ones the grant listed. Every refusal is the
same `None`.

#### Errors

Returns the read error.

### authorize_playback

```rust
pub async fn authorize_playback(
    store: &LibraryStore,
    person: i64,
    edition_id: &str,
    destination: &str,
    files_digest: &str,
) -> Result<Option<Vec<PlaybackTrack>>>
```

The tracks a play session may still stream, or `None`.

Read afresh on every stream request: the person must still be allowed
the edition ([`plan_playback`](#plan_playback)), and the edition's files on
`destination` must be the ones the session's `files_digest` was made
from, so a replaced file ends the session's streams instead of
splicing two encodings.

#### Arguments

* `store` - The library.
* `person` - The session's listener.
* `edition_id` - The session's edition.
* `destination` - The destination the session streams from.
* `files_digest` - The digest the session stored when it opened.

#### Errors

Returns the read error.

### open_copy

```rust
pub async fn open_copy(
    store: &LibraryStore,
    holder: CopyHolder<'_>,
    plan: &CopyPlan,
    destination: &str,
    serving_node_id: &str,
) -> Result<Option<OpenedCopy>>
```

Records that `holder` may download `plan`'s files from `destination`
on node `serving_node_id`, and mints the token.

#### Returns

`None` when `plan` has no files on `destination`.

#### Errors

Returns the write error.

### plan_copy

```rust
pub async fn plan_copy(
    store: &LibraryStore,
    holder: CopyHolder<'_>,
    listing: CopyListing<'_>,
) -> Result<CopyPlanned>
```

Decides whether `holder` may take this library's copy of the edition
`listing` names, and lists the copy's files.

The holder must be one this library gives copies to ([`CopyHolder`](#copyholder));
the edition must be one the person may take (`visible_to`) and one
this library has acquired, with its primary audio recorded on at least
one destination.

#### Errors

Returns the read error.

### plan_playback

```rust
pub async fn plan_playback(
    store: &LibraryStore,
    person: i64,
    edition_id: &str,
) -> Result<Option<PlaybackPlan>>
```

Decides whether `person` may play this library's copy of
`edition_id`, and lists its tracks: the one check behind the playback
manifest, a new play session and every stream request.

The edition must be a title on one of this library's store accounts
that the person's patron scope shows them (`visible_to`), and one this
library has acquired, with its primary audio recorded on at least one
destination. The routes also check the scope before they answer, so
a hidden edition gets the same 404 as an unknown one.

The Download member control ([`super::MemberControl::Download`](#membercontrol)) is
not read: it stops taking copies to keep, and anyone who can see a
book may listen to it.

#### Arguments

* `store` - The library.
* `person` - The listener (`users.id`).
* `edition_id` - The edition.

#### Returns

The plan, or `None` for "you may not" and "there is no copy" alike.

#### Errors

Returns the read error.

### claim_cron_fire

```rust
pub async fn claim_cron_fire(
    store: &LibraryStore,
    plugin_instance_id: &str,
    cron: &str,
    expected_last_ms: i64,
    fire_ms: i64,
) -> Result<CronFireClaim>
```

Claims the fire at `fire_ms` for `(plugin_instance_id, cron)` and admits
its `plugin_scheduled` job in the same batch.

#### Arguments

* `expected_last_ms` - The cursor value this node read; the claim is a
  compare-and-swap on it.
* `fire_ms` - The fire time to claim; must be later than
  `expected_last_ms`.

#### Returns

[`CronFireClaim::Claimed`](#cronfireclaim) when this call moved the cursor, otherwise
[`CronFireClaim::Lost`](#cronfireclaim).

#### Errors

Returns an error when `fire_ms` does not advance the cursor, or when the
batch or the read-back fails.

### ensure_cron_cursor

```rust
pub async fn ensure_cron_cursor(
    store: &LibraryStore,
    plugin_instance_id: &str,
    cron: &str,
    initial_last_ms: i64,
) -> Result<CronCursor>
```

Creates the cursor at `initial_last_ms` unless one exists, and returns
the stored cursor.

#### Arguments

* `initial_last_ms` - The newest past fire time, so a new schedule waits
  for its next fire instead of firing at once.

#### Errors

Returns an error when the write or the read-back fails.

### load_cron_cursor

```rust
pub async fn load_cron_cursor(
    store: &LibraryStore,
    plugin_instance_id: &str,
    cron: &str,
) -> Result<Option<CronCursor>>
```

Reads one cursor.

#### Errors

Returns an error when the read fails.

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

Every deployment row in the library.

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

### destination_filter

```rust
pub async fn destination_filter(
    store: &LibraryStore,
    instance_id: &str,
) -> Result<DestinationFilter>
```

One destination's filter: the stored one, or a filter that takes every
book.

#### Errors

Returns the read or parse error.

### destination_filters

```rust
pub async fn destination_filters<'a>(
    store: &LibraryStore,
    instance_ids: impl IntoIterator<Item = &'a str>,
) -> Result<BTreeMap<String, DestinationFilter>>
```

The stored filters of the given destinations, keyed by plugin instance id.

Each document is read by its primary key, one row per read, so the
lookup stays inside the smallest result limits an adapter may advertise
however many destinations have stored a filter. Pass the destinations
deployed on this host; ids without a document are absent from the map
and take every book.

#### Arguments

* `store` - The library database.
* `instance_ids` - The destinations to read, by plugin instance id.

#### Errors

Returns the read error, or a parse error for any stored document; an
acquire fails closed on it rather than writing where a filter said not
to.

### destination_key

```rust
pub fn destination_key(instance_id: &str) -> DocumentKey
```

The `core.destination` address for one plugin instance.

### load_destination_placement

```rust
pub async fn load_destination_placement(
    store: &LibraryStore,
    instance_id: &str,
) -> Result<Option<ConfigurationDocument<DestinationPlacementV1>>>
```

Loads one destination's placement document.

#### Returns

The stored document, or `None` when nobody has written one: the
destination then takes every book.

#### Errors

Returns the read error, or an error when the stored document is invalid
or of an unknown schema.

### set_destination_filter

```rust
pub async fn set_destination_filter(
    store: &LibraryStore,
    actor: &ConfigActor,
    instance_id: &str,
    filter: DestinationFilter,
) -> Result<ConfigurationDocument<DestinationPlacementV1>>
```

Replaces one destination's filter under compare-and-swap, creating the
document when absent and retrying a few times on a concurrent edit.

#### Arguments

* `store` - The library database.
* `actor` - Who is writing, for authorization and the audit log.
* `instance_id` - The destination's plugin instance id.
* `filter` - The new filter; no rules take every book.

#### Errors

Returns [`LibraryError::Invalid`](README.md#libraryerror) for a filter that fails
`DestinationFilter::validate`, an error when the actor may not write,
or [`LibraryError::Conflict`](README.md#libraryerror) when the document keeps changing
underneath.

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

### parse_download_request

```rust
pub fn parse_download_request(body: &[u8]) -> Result<DownloadRequestAsk>
```

Parses and validates a `POST /api/download-requests` body from its raw
bytes, sans IO, the way the route reads it: JSON into
[`DownloadRequestBody`](#downloadrequestbody), then [`DownloadRequestBody::validate`](#downloadrequestbodyvalidate).

#### Errors

Returns [`LibraryError::Invalid`](README.md#libraryerror) for bytes that are not the body's JSON
shape (unknown or repeated fields, wrong types, a missing `work_id`) or a
malformed work id.

### plan_download_request

```rust
pub async fn plan_download_request(
    store: &LibraryStore,
    work_id: &str,
) -> Result<DownloadRequestPlan>
```

Plans a download request for `work_id`.

The work's editions are read in edition id order; the first one a store
account of this library holds is downloaded, through the account
holding [`super::acquire_titles_of_editions`](#acquire_titles_of_editions) picks for it. The
request is the library's own download, so any of its accounts may
serve it, as for an acquire the operator starts.

#### Arguments

* `work_id` - This library's work.

#### Errors

- [`LibraryError::NotFound`](README.md#libraryerror) when the work has no edition, or none on a
  store account of this library.
- [`LibraryError::Conflict`](README.md#libraryerror) when the library already has a copy of an
  edition of the work (the patron can take that one).
- [`LibraryError::Orm`](README.md#libraryerror) when a read fails.

### authorized_enrollment_link

```rust
pub fn authorized_enrollment_link(
    code: &str,
    verifier: &str,
    kind: NodeKind,
) -> (String, EnrollmentSecret)
```

The link a branch or shared branch sign-in redeems through: its id and secret are
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

Returns [`LibraryError::Invalid`](README.md#libraryerror) for malformed keys or a version that is
too long or holds control characters.

### enrollment_link_brings_accounts

```rust
pub async fn enrollment_link_brings_accounts(
    store: &LibraryStore,
    link_id: &str,
) -> Result<bool>
```

Whether the user who approved the sign-in behind `link_id` chose to
bring their store connections to the branch ([`record_authorized_enrollment_link`](#record_authorized_enrollment_link)).
`false` for an unknown link and for every minted invite.

#### Errors

Returns the read error.

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

### load_branch_enrollment

```rust
pub async fn load_branch_enrollment(
    store: &LibraryStore,
) -> Result<BranchEnrollmentSettings>
```

Reads the library's branch-enrollment switch. A missing document means the
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

Reads a branch node's credential.

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

Mints a link that enrolls one branch node for `for_user_id`.

`ttl` defaults to 24 hours and must be 5 minutes to 7 days. The insert
is predicated on the enrollment switch revision read as enabled, so a
disable that commits while the link is minted wins.

#### Errors

Returns [`LibraryError::Forbidden`](README.md#libraryerror) when enrollment is disabled, the
actor is disabled, or a non-Owner mints for someone else;
[`LibraryError::NotFound`](README.md#libraryerror) / [`LibraryError::Invalid`](README.md#libraryerror) when the target
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

Returns [`LibraryError::Invalid`](README.md#libraryerror) when the text is not a `bcn1` token.

### record_authorized_enrollment_link

```rust
pub async fn record_authorized_enrollment_link(
    store: &LibraryStore,
    link_id: &str,
    secret: &EnrollmentSecret,
    for_user_id: i64,
    label: &str,
    bring_accounts: bool,
) -> Result<()>
```

Records the link an approved branch sign-in redeems through
([`authorized_enrollment_link`](#authorized_enrollment_link)), for `for_user_id`, the user who signed
in and consented. Called only after the authorization code's client,
redirect and PKCE verifier were checked, before the code is consumed, so
a failed write leaves the code usable; the redeem itself is
[`redeem_enrollment_link`](#redeem_enrollment_link), the same canonical batch a minted link takes.

`bring_accounts` keeps the user's "Bring my store connections to this
branch" choice with the link, so a retried redeem of the same exchange
answers the same way ([`enrollment_link_brings_accounts`](#enrollment_link_brings_accounts)): it gathers
the user's shareable accounts again, while the user is still active
and the branch is not revoked.

A second call for the same link id leaves the first row as it is.

#### Errors

Returns [`LibraryError::Forbidden`](README.md#libraryerror) while branch enrollment is off or the
user is not active, [`LibraryError::Invalid`](README.md#libraryerror) for a malformed label, and
the write error.

### record_node_connected

```rust
pub async fn record_node_connected(store: &LibraryStore, node_id: &str) -> Result<bool>
```

Stamps `last_connected_at` after a branch node authenticates. Returns
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

Returns [`LibraryError::Invalid`](README.md#libraryerror) for malformed keys,
[`LibraryError::Forbidden`](README.md#libraryerror) when enrollment is disabled or the link does
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

A [`super::NodeKind::Shared`](#nodekind) node (a shared branch, which joined with a sign-in
an Owner approved as such) shares the library's database, so it gets no
`node_credentials` row: it never dials the node socket. The kind is part
of the request hash, so a retry of a branch redeem as a shared branch (or the
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

Returns [`LibraryError::NotFound`](README.md#libraryerror) when the link is unknown or not the
caller's (the two read the same), [`LibraryError::Conflict`](README.md#libraryerror) when it was
already redeemed, and the write error when the update fails.

### revoke_node_credential

```rust
pub async fn revoke_node_credential(
    store: &LibraryStore,
    actor: &EnrollmentActor,
    node_id: &str,
) -> Result<NodeCredential>
```

Revokes a branch node's credential and tells every node, so the node
holding that host's connection can close it.

The operator and an Owner may revoke any branch node; any other active
user may revoke the nodes they own. Revoking twice keeps the first time.

#### Errors

Returns [`LibraryError::NotFound`](README.md#libraryerror) when the node is unknown or not the
caller's (the two read the same), [`LibraryError::Invalid`](README.md#libraryerror) when the
node has no host credential (a shared node), and the write error when
the update fails.

### set_branch_enrollment_enabled

```rust
pub async fn set_branch_enrollment_enabled(
    store: &LibraryStore,
    actor: &ConfigActor,
    enabled: bool,
) -> Result<ConfigurationDocument<BranchEnrollmentSettingsV1>>
```

Turns branch enrollment on or off for the library.

Only the operator or a verified Owner ([`verify_library_owner`](#verify_library_owner)) may write
it. Retries a few times on a concurrent edit.

#### Errors

Returns [`LibraryError::Forbidden`](README.md#libraryerror) for any other actor, and the write
error when the document cannot be stored.

### verify_library_owner

```rust
pub async fn verify_library_owner(
    store: &LibraryStore,
    user_id: i64,
) -> Result<ConfigActor>
```

Returns a [`ConfigActor::LibraryOwner`](#configactor) when `user_id` is an active Owner.

#### Errors

Returns [`LibraryError::Forbidden`](README.md#libraryerror) for anyone else (including an unknown
user), and the read error when the lookup fails.

### current_process_incarnation

```rust
pub fn current_process_incarnation() -> &'static str
```

This process's incarnation. Stable until the process exits.

### encode_node_identity_file

```rust
pub fn encode_node_identity_file(node_id: &str, library_id: &str) -> Result<Vec<u8>>
```

The [`NODE_IDENTITY_FILE`](#node_identity_file) contents [`install_node_identity`](#install_node_identity) writes, for
a caller that stages the file itself (a shared branch applying its bootstrap as
one recoverable step) with mode `0600`.

#### Errors

Returns [`LibraryError::Invalid`](README.md#libraryerror) for a node id that is not a UUID or an
empty library id, and the encode error.

### install_node_identity

```rust
pub fn install_node_identity(
    files_dir: &Path,
    node_id: &str,
    library_id: &str,
) -> Result<NodeIdentity>
```

Writes the identity file for a node id the library already recorded,
bound to `library_id`, replacing any file there atomically. A shared branch
joining a library takes the node id the library minted for it at
redeem, so its first heartbeat lands on that `nodes` row.

#### Errors

Returns [`LibraryError::Invalid`](README.md#libraryerror) for a node id that is not a UUID or an
empty library id, and the write error.

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

### add_expanded_members

```rust
pub async fn add_expanded_members(
    store: &LibraryStore,
    id: &IngestBatchId,
    parent_no: u32,
    entries: &[ExpandedEntry],
    parent_reason: Option<&str>,
) -> Result<u32>
```

Adds the entries archive member `parent_no` expanded into, numbered
after the batch's last member, and marks the archive expanded, with
`parent_reason` when the walk stopped early.

Only the batch's running job calls this, so the numbering read here
cannot race another writer.

#### Returns

The number given to the first entry; the rest follow in order.

#### Errors

Returns a read or write error.

### batch_exists

```rust
pub async fn batch_exists(store: &LibraryStore, id: &IngestBatchId) -> Result<bool>
```

Whether a batch row exists for `id`, in any state.

#### Errors

Returns a read error.

### batch_members

```rust
pub async fn batch_members(
    store: &LibraryStore,
    id: &IngestBatchId,
) -> Result<Vec<IngestMember>>
```

Reads every member of a batch, in member order.

#### Errors

Returns a read error, or an error for a corrupt row.

### cancel_batch

```rust
pub async fn cancel_batch(store: &LibraryStore, id: &IngestBatchId) -> Result<bool>
```

Cancels a batch the job has not started.

#### Returns

`false` when the batch is running or already final.

#### Errors

Returns the write error.

### create_intake_batch

```rust
pub async fn create_intake_batch(
    store: &LibraryStore,
    user_id: i64,
    node_id: &str,
    instance: &str,
    specs: &[MemberSpec],
) -> Result<Result<IngestBatch, CreateRefusal>>
```

Creates an intake batch: files read in place from `instance`, held by
`user_id`, run on `node_id`. The host creates these itself (the drop
folder); the person must be active.

#### Returns

The new batch, open, or why it was refused. Its members are complete
from the start, so it can be committed at once.

#### Errors

Returns a read or write error.

### create_upload_batch

```rust
pub async fn create_upload_batch(
    store: &LibraryStore,
    user_id: i64,
    node_id: &str,
    specs: &[MemberSpec],
    quota: UploadQuota,
) -> Result<Result<IngestBatch, CreateRefusal>>
```

Creates an upload batch for `user_id`, staged on `node_id`.

The person must be allowed to upload (the `upload` member control) and
the batch must fit their staging quota; both are checked again inside
the write. The node's free-disk floor is the caller's to check.

#### Returns

The new batch, open and taking bytes, or why it was refused.

#### Errors

Returns a read or write error.

### delete_finished_before

```rust
pub async fn delete_finished_before(
    store: &LibraryStore,
    before: DateTime<Utc>,
) -> Result<usize>
```

Deletes final batches (and their members) that finished before
`before`, at most one result page per call.

#### Returns

How many batches were deleted.

#### Errors

Returns a read or write error.

### final_batches_on

```rust
pub async fn final_batches_on(
    store: &LibraryStore,
    node_id: &str,
) -> Result<Vec<IngestBatchId>>
```

Batches on `node_id` in a final state: their staged bytes are no
longer needed.

#### Errors

Returns a read error.

### finish_batch

```rust
pub async fn finish_batch(
    store: &LibraryStore,
    id: &IngestBatchId,
    state: IngestState,
    detail: Option<&str>,
) -> Result<()>
```

Ends a running batch as `state` (done or failed), with `detail` for a
failure.

#### Errors

Returns the write error, or an error when `state` is not final.

### first_active_owner

```rust
pub async fn first_active_owner(store: &LibraryStore) -> Result<Option<i64>>
```

The library's first active Owner: who holds what the operator uploads
with no person of their own.

#### Errors

Returns a read error.

### idle_upload_batches

```rust
pub async fn idle_upload_batches(
    store: &LibraryStore,
    node_id: &str,
    before: DateTime<Utc>,
) -> Result<Vec<IngestBatchId>>
```

Open upload batches on `node_id` idle since before `before`: the
prune cancels them and removes their staged bytes.

#### Errors

Returns a read error.

### load_batch

```rust
pub async fn load_batch(
    store: &LibraryStore,
    id: &IngestBatchId,
) -> Result<Option<IngestBatch>>
```

Reads one batch.

#### Errors

Returns a read error, or an error for a corrupt row.

### load_member

```rust
pub async fn load_member(
    store: &LibraryStore,
    id: &IngestBatchId,
    member_no: u32,
) -> Result<Option<IngestMember>>
```

Reads one member.

#### Errors

Returns a read error, or an error for a corrupt row.

### mark_committed

```rust
pub async fn mark_committed(
    store: &LibraryStore,
    id: &IngestBatchId,
    job_id: &str,
) -> Result<bool>
```

Hands an open batch to its job: every member must be complete and the
manifest's every file present.

#### Returns

`false` when the batch is not open or a file is incomplete.

#### Errors

Returns the write error.

### parse_upload_manifest

```rust
pub fn parse_upload_manifest(body: &[u8]) -> Result<Vec<MemberSpec>, ManifestError>
```

Parses a `POST /api/uploads` body into the batch's members, checking
every path with the archive path policy. Sans-IO: the
`upload_manifest` fuzz target drives this same function.

Duplicate and folding paths, sizes and quotas are checked when the
batch is created ([`create_upload_batch`](#create_upload_batch)).

#### Errors

Returns [`ManifestError`](#manifesterror) for a body that is not the documented JSON,
a path the policy refuses, a malformed digest or too many files.

### parse_upload_offset

```rust
pub fn parse_upload_offset(raw: &str) -> Option<u64>
```

Parses an `Upload-Offset` header: plain ASCII digits (no sign, space or
leading `+`), at most 19 of them, and no larger than `i64::MAX`.

### patron_holding_id

```rust
pub async fn patron_holding_id(
    store: &LibraryStore,
    listing_id: &str,
    user_id: i64,
) -> Result<Option<String>>
```

The id of the patron holding `user_id` has of `listing_id`, if any.

#### Errors

Returns a read error.

### record_ingest_acquired

```rust
pub async fn record_ingest_acquired(
    store: &LibraryStore,
    edition_id: &str,
    holding_id: &str,
    storage_key: &str,
) -> Result<bool>
```

Marks an edition acquired by a patron holding after the ingest job
wrote its files, and publishes `edition_acquire_changed` so the
library's page index picks the book up.

The holding must still hold the edition; an edition another live
holding already acquired keeps that holding's files.

#### Returns

`true` when the edition is now acquired by `holding_id`.

#### Errors

Returns the write error.

### record_received

```rust
pub async fn record_received(
    store: &LibraryStore,
    id: &IngestBatchId,
    member_no: u32,
    received: u64,
) -> Result<bool>
```

Records that `received` bytes of a member are staged, and that the
batch was active now.

#### Returns

`false` when the batch is no longer open or `received` passes the
member's size; nothing is written then.

#### Errors

Returns the write error.

### review_items

```rust
pub async fn review_items(
    store: &LibraryStore,
    after: Option<(&IngestBatchId, u32)>,
) -> Result<Vec<ReviewItem>>
```

Members stored for review, oldest batch first, after `after` (a
`(batch id, member)` cursor), at most one result page.

#### Errors

Returns a read error, or an error for a corrupt row.

### set_member_outcomes

```rust
pub async fn set_member_outcomes(
    store: &LibraryStore,
    id: &IngestBatchId,
    updates: &[OutcomeUpdate],
) -> Result<()>
```

Records outcomes the job decided, `MEMBERS_PER_WRITE` per host batch,
while the batch is running.

#### Errors

Returns the write error.

### staged_bytes

```rust
pub async fn staged_bytes(store: &LibraryStore, user_id: i64) -> Result<u64>
```

Bytes `user_id` has in live upload batches (open, committed or
running), which the staging quota counts.

#### Errors

Returns the read error.

### start_batch

```rust
pub async fn start_batch(
    store: &LibraryStore,
    id: &IngestBatchId,
    job_id: &str,
) -> Result<bool>
```

Starts (or, after a crash, resumes) the job's run of a batch.

#### Returns

`false` when the batch is neither committed nor running under
`job_id`: it was cancelled, or belongs to another job.

#### Errors

Returns the write error.

### user_batches

```rust
pub async fn user_batches(
    store: &LibraryStore,
    user_id: i64,
    limit: u64,
) -> Result<Vec<IngestBatch>>
```

A person's batches, newest first, at most `limit`.

#### Errors

Returns a read error, or an error for a corrupt row.

### accept_reported_origin

```rust
pub async fn accept_reported_origin(
    store: &LibraryStore,
    actor: &ConfigActor,
) -> Result<ConfigurationDocument<IngressSettings>>
```

Adopts the origin the active instance reported and is waiting on the
operator.

#### Returns

The document after the change; unchanged when nothing was pending.

#### Errors

Returns an error when the actor may not write or the write fails.

### activate_ingress

```rust
pub async fn activate_ingress(
    store: &LibraryStore,
    actor: &ConfigActor,
    instance: &PluginInstanceId,
) -> Result<ConfigurationDocument<IngressSettings>>
```

Makes `instance` the library's active ingress, replacing any other.

Switching to another instance keeps the adopted origin: the new instance's
first report then has to match it or wait for the operator, like any
change of name.

#### Arguments

* `store` - The library database.
* `actor` - The operator or a verified Owner.
* `instance` - The plugin instance to activate.

#### Errors

Returns an error when the actor may not write, the store refuses the
write, or the document keeps changing underneath.

### deactivate_ingress

```rust
pub async fn deactivate_ingress(
    store: &LibraryStore,
    actor: &ConfigActor,
) -> Result<ConfigurationDocument<IngressSettings>>
```

Turns ingress off. The adopted origin is kept, so turning the same
provider back on serves under the same name.

#### Errors

Returns an error when the actor may not write or the write fails.

### ingress_settings

```rust
pub async fn ingress_settings(store: &LibraryStore) -> Result<IngressSettings>
```

The ingress settings: the stored document, or ingress off.

#### Errors

Returns the read or parse error.

### load_ingress_settings

```rust
pub async fn load_ingress_settings(
    store: &LibraryStore,
) -> Result<Option<ConfigurationDocument<IngressSettings>>>
```

Loads the `core.ingress` document.

#### Returns

The stored document, or `None` when nobody has written one: ingress is
then off.

#### Errors

Returns the read error, or an error when the stored document is invalid
or of an unknown schema.

### record_started

```rust
pub async fn record_started(
    store: &LibraryStore,
    actor: &ConfigActor,
    instance: &PluginInstanceId,
    started: &IngressStarted,
) -> Result<OriginReport>
```

Records what `instance` reported from `Ingress.start`.

#### Arguments

* `store` - The library database.
* `actor` - The host writing on the plugin's behalf.
* `instance` - The instance that started.
* `started` - Its report, checked here with [`parse_ingress_origin`](../bookclerk_plugin_abi/origin.md#parse_ingress_origin).

#### Returns

Whether the origin was adopted, is waiting for the operator, or came from
an instance that is not active.

#### Errors

[`LibraryError::Invalid`](README.md#libraryerror) when the origin is not `https://` and a DNS
name, or the write error.

### set_ingress_exposure

```rust
pub async fn set_ingress_exposure(
    store: &LibraryStore,
    actor: &ConfigActor,
    exposure: IngressExposure,
) -> Result<ConfigurationDocument<IngressSettings>>
```

Sets what public traffic may reach.

#### Errors

Returns an error when the actor may not write or the write fails.

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
became plain, are dropped. Table rows are brought to the current columns
with [`bookclerk_plugin_abi::SettingSpec::normalize_value`]: cells of
removed columns are dropped and defaults of new columns written, which
changes the document without removing a key.

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

Seals one instance secret with the library DEK.

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
it adds to the library's outputs on that node only, and a deployment of
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

Name of the library semaphore for one store account.

### claim_routed_job

```rust
pub async fn claim_routed_job(
    store: &LibraryStore,
    request: &JobClaimRequest<'_>,
) -> Result<Option<RoutedJobClaim>>
```

Claims the best ready job this node can run, under the library semaphores.

Pages through the ready pending jobs of the class, best first
([`ROUTED_CLAIM_CANDIDATES`](#routed_claim_candidates) per page); skips those the node is not
trusted with ([`super::node_trust`](#node_trust)), those pinned to another node
([`crate::JobPayload::node_id`](README.md#jobpayload)), those whose requirements this node
does not meet, and those whose account or instance semaphore is full;
takes the permits of the first one left and claims it with
[`LibraryStore::claim_job_by_id`](README.md#librarystoreclaim_job_by_id) at the lease generation of the record
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

Name of the library semaphore for one resource class.

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

### instance_running_on_database_node

```rust
pub async fn instance_running_on_database_node(
    store: &LibraryStore,
    instance: &str,
    now_ms: i64,
) -> Result<bool>
```

True when some live node that holds the library database runs
`instance`: it has a deployment there that should run, the node's last
observation says the guest is up (`running` or `healthy`), and the
node's heartbeat is current ([`super::node_status::NodeStatus::is_live`](#nodestatusis_live)).
A node holds the database when it is a shared branch
([`super::NodeKind::Shared`](#nodekind)) or has no `nodes` row (a worker in this
process); a branch with its own database does not.

A job that requires only `instance` and runs only where the database is
(a notification send) is then claimable, however long it has waited for
a worker; when this is false, nothing can claim it until the instance
comes up on such a node. An unparsable id is never running.

#### Arguments

* `now_ms` - The caller's clock, in Unix milliseconds, for the
  heartbeat check.

#### Errors

Returns an error when a read fails.

### instance_semaphore

```rust
pub fn instance_semaphore(instance_id: &str) -> String
```

Name of the library semaphore for one plugin instance.

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
through another shared node, matches on this prefix, so a still-live
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

The library semaphores `job` counts against under `limits`.

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
instances ([`crate::JobPayload::requires`](README.md#jobpayload), the destinations an acquire
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
as before rather than waiting forever. For [`JobKind::Acquire`](README.md#jobkind), every
library instance of a plugin key in `node_bound_plugin_keys` that is
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

Returns [`LibraryError::Conflict`](README.md#libraryerror) when the required instances have no
common deployment node, or an error when a read fails.

### apply_keeper_update

```rust
pub async fn apply_keeper_update(
    store: &LibraryStore,
    user_id: i64,
    node_id: &str,
    account: &CarriedAccount,
    revision: i64,
) -> Result<KeeperUpdate>
```

Applies a keeper branch's push of one account's newest sign-in.

One atomic batch replaces the account's secrets (re-sealed with this
install's key) and records `revision`, every statement applied only
while the account is linked to `user_id` (the branch's owner), shared
with the branch `node_id`, and `revision` is above the last one applied
from it. So a push that lost a race with a newer one, or that arrives
after the person disconnected the account, changes nothing.

#### Errors

Returns [`LibraryError::Invalid`](README.md#libraryerror) for a revision that is not positive or
an account without secrets, a seal error, or a read or write error.

### confirmed_keeping_branch

```rust
pub async fn confirmed_keeping_branch(
    store: &LibraryStore,
    plugin_key: &str,
    account_id: &str,
) -> Result<Option<String>>
```

The branch that has taken over this account's sign-in for this library:
its [`keeping_branch`](#keeping_branch), once that branch has pushed a sign-in here
(`accounts.update`). Until then the library still keeps the account
itself (saves its own refreshes and takes write-backs), so a branch that
was sent the account but could not keep it never leaves it unrefreshed.

#### Errors

Returns the read error.

### keeping_branch

```rust
pub async fn keeping_branch(
    store: &LibraryStore,
    plugin_key: &str,
    account_id: &str,
) -> Result<Option<String>>
```

The branch that keeps this account for this library: the node of its
earliest-recorded share with a branch, or `None` when no branch holds
it (this library keeps it).

#### Errors

Returns the read error.

### keeps_for_link

```rust
pub async fn keeps_for_link(
    store: &LibraryStore,
    plugin_key: &str,
    account_id: &str,
    library_id: &str,
) -> Result<bool>
```

True when this install keeps the account for the library it joined as
`library_id` (it holds a share with that library).

#### Errors

Returns the read error.

### kept_shares

```rust
pub async fn kept_shares(store: &LibraryStore) -> Result<Vec<KeptShare>>
```

Every account this install shares with a library it joined, each library
once, with the revision last pushed there.

#### Errors

Returns the read error.

### record_pushed_revision

```rust
pub async fn record_pushed_revision(
    store: &LibraryStore,
    plugin_key: &str,
    account_id: &str,
    library_id: &str,
    revision: i64,
    observed: i64,
) -> Result<()>
```

Records that the keeper sent `revision` of the account to the library
`library_id` (whether it applied it or already had it), for the sign-in
whose fingerprint is `observed` ([`SignIn::fingerprint`](#signin)). Never lowers
the recorded revision.

#### Errors

Returns the write error.

### sign_in_revision

```rust
pub async fn sign_in_revision(
    store: &LibraryStore,
    plugin_key: &str,
    account_id: &str,
) -> Result<Option<SignIn>>
```

The keeper's view of an account's sign-in: when its newest secret was
written here and a fingerprint of all of them, or `None` when it has no
secret.

#### Errors

Returns the read error, or [`LibraryError::Invalid`](README.md#libraryerror) for a stored time
that does not parse.

### acquire_lease

```rust
pub async fn acquire_lease(
    store: &LibraryStore,
    name: &str,
    holder: &str,
    ttl: Duration,
) -> Result<Option<LibraryLease>>
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
) -> Result<Option<LibraryLease>>
```

[`acquire_lease`](#acquire_lease) on a bare connection (schema migration runs before a
[`LibraryStore`](README.md#librarystore) exists).

#### Errors

Returns an error when the read or write fails.

### list_leases

```rust
pub async fn list_leases(store: &LibraryStore) -> Result<Vec<LibraryLease>>
```

Every lease row, held or expired, by name.

#### Errors

Returns an error when the read fails.

### load_lease

```rust
pub async fn load_lease(store: &LibraryStore, name: &str) -> Result<Option<LibraryLease>>
```

The current row of lease `name`, held or expired.

#### Errors

Returns an error when the read fails.

### load_lease_on

```rust
pub async fn load_lease_on<C: ConnectionTrait>(
    db: &C,
    name: &str,
) -> Result<Option<LibraryLease>>
```

[`load_lease`](#load_lease) on a bare connection.

#### Errors

Returns an error when the read fails.

### release_lease

```rust
pub async fn release_lease(store: &LibraryStore, lease: &LibraryLease) -> Result<()>
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
    lease: &LibraryLease,
) -> Result<()>
```

[`release_lease`](#release_lease) on a bare connection.

#### Errors

Returns an error when the write fails.

### renew_lease

```rust
pub async fn renew_lease(
    store: &LibraryStore,
    lease: &LibraryLease,
    ttl: Duration,
) -> Result<Option<LibraryLease>>
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
    lease: &LibraryLease,
    ttl: Duration,
) -> Result<Option<LibraryLease>>
```

[`renew_lease`](#renew_lease) on a bare connection.

#### Errors

Returns an error when the write fails.

### decode_spki_pin

```rust
pub fn decode_spki_pin(pin: &str) -> Result<[u8; 32]>
```

Decodes an SPKI pin: base64url (no padding) of a SHA-256 digest.

#### Errors

Returns [`LibraryError::Invalid`](README.md#libraryerror) for anything else.

### join_cycle

```rust
pub async fn join_cycle(
    store: &LibraryStore,
    parent_ancestry: &[String],
) -> Result<Option<String>>
```

This library's own library id when `parent_ancestry` names it: joining
that library would make a cycle (A ← B ← A, A ← B ← C ← A). `None` when
it does not, or this install has no library identity yet.

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

### leave_library_link

```rust
pub async fn leave_library_link(
    store: &LibraryStore,
    parent_library_id: &str,
    expected_node_id: Option<&str>,
) -> Result<Option<LibraryLink>>
```

Leaves a library on this side: marks the link
[`LIBRARY_LINK_LEFT`](#library_link_left) and deletes its sealed node keys, so this install
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
* `parent_library_id` - Library the link joins.
* `expected_node_id` - The node the caller means to leave, when it read
  the link earlier (to send `node.leaving`); `None` leaves whatever node
  the link names now.

#### Errors

Returns [`LibraryError::Invalid`](README.md#libraryerror) when the link was re-enrolled as
another node, and the read or write error.

### library_ancestry

```rust
pub async fn library_ancestry(store: &LibraryStore) -> Result<Vec<String>>
```

This library's ancestry (what `GET /api/node/v1/info` reports): its own
library id, then for each active link the owner's library id and the
ancestry that owner reported, without repeats, at most
[`bookclerk_node_proto::MAX_ANCESTRY`](../bookclerk_node_proto/README.md#max_ancestry) ids. Empty when this install has
no library identity yet. A join that would take it past that limit is
refused ([`join_overflows_ancestry`](#join_overflows_ancestry)); should a library above grow its
ancestry later, the list is cut at exactly the limit, which in turn
refuses every join under it.

A shared branch shares its library's database, so it reports the same ancestry:
it is the same node of the tree.

#### Errors

Returns the read error.

### list_library_links

```rust
pub async fn list_library_links(store: &LibraryStore) -> Result<Vec<LibraryLink>>
```

Every library link, oldest enrollment first.

#### Errors

Returns the read error.

### load_library_link

```rust
pub async fn load_library_link(
    store: &LibraryStore,
    parent_library_id: &str,
) -> Result<Option<LibraryLink>>
```

One library link.

#### Errors

Returns the read error.

### load_library_link_keys

```rust
pub async fn load_library_link_keys(
    store: &LibraryStore,
    link: &LibraryLink,
) -> Result<LibraryLinkKeys>
```

Unseals the node keys of an active `link`.

#### Errors

Returns [`LibraryError::Forbidden`](README.md#libraryerror) when the link is not active (a
revoked or departed node never signs again),
[`LibraryError::NotFound`](README.md#libraryerror) when the key row is missing, and
[`LibraryError::Invalid`](README.md#libraryerror) when it does not hold two 32-byte keys.

### mark_library_link_revoked

```rust
pub async fn mark_library_link_revoked(
    store: &LibraryStore,
    parent_library_id: &str,
    remote_node_id: &str,
) -> Result<bool>
```

Marks an **active** link still enrolled as `remote_node_id`
[`LIBRARY_LINK_REVOKED`](#library_link_revoked) (the owner closed with 4403 on that node's
connection). A link this install already left stays `left`, and one a
later join enrolled as another node keeps its new enrollment.

#### Arguments

* `store` - Library store holding the link.
* `parent_library_id` - Library the link joins.
* `remote_node_id` - The node the revoked connection ran as.

#### Returns

Whether a row changed: `false` when the link is gone, no longer active,
or enrolled as another node.

#### Errors

Returns the write error.

### parse_job_cap_request

```rust
pub fn parse_job_cap_request(body: &[u8]) -> Result<JobCapRequest>
```

Parses and validates a `POST /api/config/join/capacity` body from its
raw bytes, sans IO, the way the route reads it: JSON into
[`CapacityBody`](#capacitybody), then [`CapacityBody::validate`](#capacitybodyvalidate).

#### Errors

Returns [`LibraryError::Invalid`](README.md#libraryerror) for bytes that are not the body's JSON
shape (unknown or duplicate fields, wrong types, numbers outside `u32`),
a missing `max_jobs`, or a cap above [`LinkJobCap::MAX`](#linkjobcapmax).

### parse_sharing_request

```rust
pub fn parse_sharing_request(body: &[u8]) -> Result<SharingRequest>
```

Parses a `POST /api/config/join/sharing` body from its raw bytes, sans
IO, the way the route reads it: JSON into [`SharingBody`](#sharingbody), then
[`SharingBody::into_request`](#sharingbodyinto_request).

#### Errors

Returns [`LibraryError::Invalid`](README.md#libraryerror) for bytes that are not the body's JSON
shape: not an object, unknown or duplicate fields, a missing
`library_id`, `share_books` or `include_branches`, or a value of the
wrong type.

### record_library_link

```rust
pub async fn record_library_link(
    store: &LibraryStore,
    link: &NewLibraryLink,
    keys: &LibraryLinkKeys,
) -> Result<LibraryLink>
```

Seals `keys` and records the link, replacing an earlier link to the same
library (a re-enrollment) as `active`.

The key row is written first: a link row never exists without keys, and
a key row left by a failed write is replaced by the next enrollment.
When the re-enrollment minted a different node id, the earlier node's
key row is deleted once the new link row is stored, so the sealed
private keys of a node the owner no longer knows are not kept. The link
is stored with no person bound (a re-enrollment unbinds it); the join
binds one with [`set_library_link_person`](#set_library_link_person).

#### Errors

Returns [`LibraryError::Invalid`](README.md#libraryerror) for a malformed id, origin, or pin, an
error when no master key is configured, and the write error.

### set_library_link_ancestry

```rust
pub async fn set_library_link_ancestry(
    store: &LibraryStore,
    parent_library_id: &str,
    ancestry: &[String],
) -> Result<bool>
```

Records the owner library's ancestry as it reported it (`info` at join,
`catalog.session` on every sync). Ids that are not plain library ids are
refused; at most [`bookclerk_node_proto::MAX_ANCESTRY`](../bookclerk_node_proto/README.md#max_ancestry) are kept.

#### Errors

Returns [`LibraryError::Invalid`](README.md#libraryerror) for a malformed ancestry, and the
write error.

### set_library_link_job_cap

```rust
pub async fn set_library_link_job_cap(
    store: &LibraryStore,
    parent_library_id: &str,
    max_jobs: Option<LinkJobCap>,
) -> Result<bool>
```

Sets the most jobs this install runs for one library at once; `None`
clears the cap, leaving the link its equal share of the machine.
Answers false when no such link exists. The running client picks the
new cap up on the next share update.

#### Errors

Returns the write error.

### set_library_link_keep_copies

```rust
pub async fn set_library_link_keep_copies(
    store: &LibraryStore,
    parent_library_id: &str,
    keep_copies: bool,
) -> Result<bool>
```

Sets the link's "Keep copies on this computer" setting.

#### Arguments

* `parent_library_id` - The library the link joined.
* `keep_copies` - The new setting: whether the branch takes the
  library's copies of its person's books.

#### Returns

Whether the link exists (`false` when there is no such link); setting
the value it already has still returns `true`.

#### Errors

Returns the write error.

### set_library_link_listing

```rust
pub async fn set_library_link_listing(
    store: &LibraryStore,
    parent_library_id: &str,
    list_members_books: bool,
) -> Result<bool>
```

Sets the link's "List my members' books to *library*" setting.

#### Arguments

* `parent_library_id` - The library the link joined.
* `list_members_books` - The new setting.

#### Returns

Whether the link exists (`false` when there is no such link); setting
the value it already has still returns `true`.

#### Errors

Returns the write error.

### set_library_link_person

```rust
pub async fn set_library_link_person(
    store: &LibraryStore,
    parent_library_id: &str,
    remote_node_id: &str,
    person_user_id: Option<i64>,
) -> Result<bool>
```

Binds the link to the local user who joined the library through it.

Only that person may add their store connections to the library, or
withdraw them from it, through this link.

#### Arguments

* `parent_library_id` - The library the link joined.
* `remote_node_id` - The node id that join enrolled as: a join another
  one replaced since binds nothing.
* `person_user_id` - The local user who joined (`None` unbinds it).

#### Returns

Whether the link exists as that node (`false` otherwise).

#### Errors

Returns the write error.

### set_library_link_settings

```rust
pub async fn set_library_link_settings(
    store: &LibraryStore,
    parent_library_id: &str,
    settings: LibraryLinkSettings,
) -> Result<bool>
```

Sets a link's sharing, listing and work settings in one statement, so a
failed write leaves every one of them as it was.

#### Arguments

* `parent_library_id` - The library the link joined.
* `settings` - The new values; each `None` keeps the stored one.

#### Returns

Whether the link exists (`false` when there is no such link); setting
the values it already has still returns `true`.

#### Errors

Returns the write error.

### set_library_link_sharing

```rust
pub async fn set_library_link_sharing(
    store: &LibraryStore,
    parent_library_id: &str,
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

### set_library_link_state

```rust
pub async fn set_library_link_state(
    store: &LibraryStore,
    parent_library_id: &str,
    state: &str,
    remote_node_id: Option<&str>,
) -> Result<bool>
```

Sets a link's state. Answers false when no such link exists.

#### Arguments

* `store` - Library store holding the link.
* `parent_library_id` - Library the link joins.
* `state` - One of [`LIBRARY_LINK_ACTIVE`](#library_link_active), [`LIBRARY_LINK_REVOKED`](#library_link_revoked) or
  [`LIBRARY_LINK_LEFT`](#library_link_left).
* `remote_node_id` - When set, only a link still enrolled as this node is
  changed, so a client running under an earlier enrollment cannot touch the
  link that replaced it. `None` changes the link whatever its node.

#### Errors

Returns [`LibraryError::Invalid`](README.md#libraryerror) for an unknown state, and the write
error.

### set_library_link_work

```rust
pub async fn set_library_link_work(
    store: &LibraryStore,
    parent_library_id: &str,
    work_for_members: bool,
    keep_member_copies: bool,
) -> Result<bool>
```

Sets a link's two work settings: "Download for other members of
*library*" and "Keep copies of other members' books". Answers false
when no such link exists. The running client claims under the new
scope from its next consent check.

#### Errors

Returns the write error.

### books_imported_from_link

```rust
pub async fn books_imported_from_link(
    store: &LibraryStore,
    library_id: &str,
) -> Result<BTreeSet<HeldBookKey>>
```

Every book imported from the library `library_id`, by store identity:
the books the catalog sync must not send back to it. Read in keyset
pages no larger than the database plugin's `maxResultRows`.

#### Errors

Returns an error when a read fails.

### import_link_title

```rust
pub async fn import_link_title(
    store: &LibraryStore,
    import: LinkImport<'_>,
) -> Result<LinkImportOutcome>
```

Imports a title the library committed into this install's own library
(see the module docs).

A file whose key another live edition here already holds is left
unrecorded (logged), as an acquire on this install would refuse it; when
that is the primary file, the title is not marked acquired
([`LinkImportOutcome::KeyHeldElsewhere`](#linkimportoutcome)); when it is the PDF, the PDF
is not marked acquired.

#### Errors

Returns an error when a read or write fails; the writes before it stay.

### record_copy_taken_from_link

```rust
pub async fn record_copy_taken_from_link(
    store: &LibraryStore,
    library_id: &str,
    book: TakenCopy<'_>,
) -> Result<()>
```

Records that this install took its copy of `book` from the library
`library_id`, so the catalog sync never sends that library a book it
provided itself ([`books_imported_from_link`](#books_imported_from_link)). A book already
recorded for that library is left as it is; one taken from several
libraries is recorded once for each.

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

Reads every topic's revision: one batch of one-row reads of
`live_topic_revisions`, one per topic, so it fits an adapter that
returns a single row per statement.

A topic without a counter row (a database the seed has not reached yet)
reads as revision 0.

#### Errors

Returns the read error.

### assigned_member_controls

```rust
pub async fn assigned_member_controls(
    store: &LibraryStore,
    user: &UserRecord,
) -> Result<AssignedControls>
```

The controls of `user`: the library's default, their overrides, and
what applies to them.

#### Errors

Returns a read or parse error; callers fail closed on it.

### default_member_controls

```rust
pub async fn default_member_controls(store: &LibraryStore) -> Result<MemberControls>
```

The library's default member controls: the stored document, or every
control allowed.

#### Errors

Returns the read or parse error.

### effective_controls

```rust
pub fn effective_controls(
    user: &UserRecord,
    defaults: MemberControls,
    overrides: ControlOverrides,
) -> MemberControls
```

What applies to `user` given the library's default and their
overrides: nothing for a user who is not active, everything for an
active Owner, the overrides over "everything allowed" for an active
Administrator, and the overrides over the default for an active Member.

### load_default_member_controls

```rust
pub async fn load_default_member_controls(
    store: &LibraryStore,
) -> Result<Option<ConfigurationDocument<MemberControls>>>
```

Loads the library's default member controls document.

#### Returns

The stored document, or `None` when nobody has written one: every
control is then allowed.

#### Errors

Returns the read error, or an error when the stored document is invalid
or of an unknown schema.

### member_control_overrides

```rust
pub async fn member_control_overrides(
    store: &LibraryStore,
    user_id: i64,
) -> Result<ControlOverrides>
```

The overrides stored for `user_id` (none when there is no row).

#### Errors

Returns the read error, or an error when the stored row is invalid.

### parse_member_controls

```rust
pub fn parse_member_controls(body: &[u8]) -> Result<MemberControls>
```

Parses the body of `PUT /api/users/default-controls` from its raw
bytes, sans IO, the way the route reads it.

#### Returns

The new default, with an absent control allowed.

#### Errors

Returns [`LibraryError::Invalid`](README.md#libraryerror) for bytes that are not one JSON
object of known controls with boolean values and no repeated key.

### parse_user_controls_request

```rust
pub fn parse_user_controls_request(body: &[u8]) -> Result<ControlOverrides>
```

Parses the body of `PUT /api/users/{id}/controls` from its raw bytes,
sans IO, the way the route reads it.

#### Returns

The patron's overrides, with an absent or `null` control following
the default.

#### Errors

Returns [`LibraryError::Invalid`](README.md#libraryerror) for bytes that are not one JSON
object with exactly the key `overrides`, itself an object of known
controls with boolean or `null` values, with no repeated key.

### role_follows_default_controls

```rust
pub fn role_follows_default_controls(role: UserRole) -> bool
```

Whether a role follows the library's default controls: Members do.
An Administrator may do everything except the controls set
for them alone, so a default meant for patrons never limits staff.

### role_has_controls

```rust
pub fn role_has_controls(role: UserRole) -> bool
```

Whether member controls ever limit a role: Members and Administrators
can be limited; Owners never are.

An Administrator's controls are set only by an Owner or the operator,
because an Administrator may manage Members alone.

### set_default_member_controls

```rust
pub async fn set_default_member_controls(
    store: &LibraryStore,
    actor: &ConfigActor,
    controls: MemberControls,
) -> Result<ConfigurationDocument<MemberControls>>
```

Replaces the library's default member controls under compare-and-swap,
creating the document when absent and retrying a few times on a
concurrent edit.

#### Arguments

* `store` - The library database.
* `actor` - Who is writing, for authorization and the audit log.
* `controls` - The new default.

#### Errors

Returns an error when the actor may not write, or
[`LibraryError::Conflict`](README.md#libraryerror) when the document keeps changing underneath.

### set_member_control_overrides

```rust
pub async fn set_member_control_overrides(
    store: &LibraryStore,
    user_id: i64,
    overrides: ControlOverrides,
) -> Result<AssignedControls>
```

Replaces the overrides of `user_id`; with none, the user follows the
library's default for every control again.

The caller has already checked that the actor may manage the user. The
write is guarded on the role read here ([`set_member_control_overrides_for`](#set_member_control_overrides_for)).

#### Errors

Returns [`LibraryError::NotFound`](README.md#libraryerror) for an unknown user, or the errors
of [`set_member_control_overrides_for`](#set_member_control_overrides_for).

### set_member_control_overrides_for

```rust
pub async fn set_member_control_overrides_for(
    store: &LibraryStore,
    user: &UserRecord,
    overrides: ControlOverrides,
) -> Result<(UserRecord, AssignedControls)>
```

Replaces the overrides of `user`, as the caller read them when it
checked that the actor may manage them; with none, the user follows the
library's default for every control again.

One atomic batch updates the stored row, inserts it when there is none,
or deletes it when no control is overridden. Every statement is guarded
on the user still having the role in `user`, so a role change made after
the caller authorized the write (an Owner promoting a Member to
Administrator, say) stops it rather than letting it through under the
old authorization.

#### Returns

The user as read after the write, with the controls that apply to them
computed from that same record, so a caller describing both never mixes
two roles.

#### Errors

Returns [`LibraryError::Conflict`](README.md#libraryerror) when the user did not have the role
in `user` as the batch ran (nothing was written; whether the role
changes again afterwards does not matter), [`LibraryError::NotFound`](README.md#libraryerror)
when the user is gone, or the read or write error.

### user_may

```rust
pub async fn user_may(
    store: &LibraryStore,
    user_id: i64,
    control: MemberControl,
) -> Result<bool>
```

Whether `user_id` may do `control` here. An unknown user may not.

#### Errors

Returns a read or parse error; callers fail closed on it.

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

Inserts `[output]` when the library document is absent.

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

Loads the library output document.

#### Errors

Returns an error when the document is missing, unsupported, or invalid.

### overlay_output

```rust
pub fn overlay_output(
    config: &mut Config,
    output: &ConfigurationDocument<OutputSettingsV1>,
    library_id: &str,
)
```

Copies the authoritative output document onto `config` for `library_id`.

Revisions are only comparable within one library. Callers that switch
databases pass the new library id explicitly.

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

Replaces the library output document.

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

Returns [`LibraryError::Forbidden`](README.md#libraryerror) unless the actor is an operator,
[`LibraryError::NotFound`](README.md#libraryerror) when the node or user is unknown,
[`LibraryError::Invalid`](README.md#libraryerror) when the user is disabled, and the write error
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

Returns [`LibraryError::Forbidden`](README.md#libraryerror) when the node is unknown or owned by
someone else, and the read error when the lookup fails.

### assigned_patron_scope

```rust
pub async fn assigned_patron_scope(
    store: &LibraryStore,
    user_id: i64,
) -> Result<AssignedScope>
```

The scope a user has: their own, or the library's default.

The role is not considered; see [`scope_for_user`](#scope_for_user) for the scope that
applies to a signed-in user.

#### Errors

Returns [`LibraryError::NotFound`](README.md#libraryerror) for an unknown user, or a read or
parse error.

### default_patron_scope

```rust
pub async fn default_patron_scope(store: &LibraryStore) -> Result<PatronScope>
```

The library's default scope: the stored document, or no restriction.

#### Errors

Returns the read or parse error.

### default_work_scope

```rust
pub async fn default_work_scope(store: &LibraryStore) -> Result<Option<WorkScope>>
```

The page index filter for a caller who is not a library user, such as a
session whose identity is linked to nobody: the library default, with
no store accounts of their own. `None` when the default hides nothing.

#### Errors

Returns a read or parse error; callers fail closed on it.

### load_default_patron_scope

```rust
pub async fn load_default_patron_scope(
    store: &LibraryStore,
) -> Result<Option<ConfigurationDocument<PatronScope>>>
```

Loads the library's default scope document.

#### Returns

The stored document, or `None` when nobody has written one: the default
is then [`PatronScope::unrestricted`](#patronscopeunrestricted).

#### Errors

Returns the read error, or an error when the stored document is invalid
or of an unknown schema.

### parse_patron_scope

```rust
pub fn parse_patron_scope(body: &[u8]) -> Result<PatronScope>
```

Parses the body of `PUT /api/users/default-scope` from its raw bytes,
sans IO, the way the route reads it: JSON into [`PatronScope`](#patronscope), then
trimmed ([`PatronScope::normalized`](#patronscopenormalized)) and checked
([`PatronScope::validate`](#patronscopevalidate)) as [`set_default_patron_scope`](#set_default_patron_scope) does.

#### Errors

Returns [`LibraryError::Invalid`](README.md#libraryerror) for bytes that are not the scope's JSON
shape (unknown or repeated fields, wrong types, a missing rule field) or
a scope that fails [`PatronScope::validate`](#patronscopevalidate).

### parse_user_scope_request

```rust
pub fn parse_user_scope_request(body: &[u8]) -> Result<Option<PatronScope>>
```

Parses the body of `PUT /api/users/{id}/scope` from its raw bytes, sans
IO, the way the route reads it: JSON into [`PutUserScope`](#putuserscope), then the
scope (when one is given) trimmed and checked as [`set_patron_scope`](#set_patron_scope)
does.

#### Returns

The patron's own scope, or `None` (`"scope": null`, or no `scope` key)
to return them to the default.

#### Errors

Returns [`LibraryError::Invalid`](README.md#libraryerror) for bytes that are not the body's JSON
shape or a scope that fails [`PatronScope::validate`](#patronscopevalidate).

### role_is_scoped

```rust
pub fn role_is_scoped(role: UserRole) -> bool
```

Whether a role is ever scoped: Members are patrons, while
Owners and Administrators always see the whole library.

### scope_for_user

```rust
pub async fn scope_for_user(
    store: &LibraryStore,
    user: &UserRecord,
) -> Result<Option<PatronScope>>
```

The scope that applies to `user`: `None` for an Owner or Administrator,
who see everything, and the assigned scope for a Member.

#### Errors

Returns a read or parse error.

### set_default_patron_scope

```rust
pub async fn set_default_patron_scope(
    store: &LibraryStore,
    actor: &ConfigActor,
    scope: PatronScope,
) -> Result<ConfigurationDocument<PatronScope>>
```

Replaces the library's default scope under compare-and-swap, creating the
document when absent and retrying a few times on a concurrent edit.

#### Arguments

* `store` - The library database.
* `actor` - Who is writing, for authorization and the audit log.
* `scope` - The new default.

#### Errors

Returns [`LibraryError::Invalid`](README.md#libraryerror) for a scope that fails
[`PatronScope::validate`](#patronscopevalidate), an error when the actor may not write, or
[`LibraryError::Conflict`](README.md#libraryerror) when the document keeps changing underneath.

### set_patron_scope

```rust
pub async fn set_patron_scope(
    store: &LibraryStore,
    user_id: i64,
    scope: Option<PatronScope>,
) -> Result<AssignedScope>
```

Gives a user their own scope, or with `None` returns them to the
library's default.

The caller has already checked that the actor may manage the user. One
atomic batch updates the stored row, or inserts it when there is none.

#### Errors

Returns [`LibraryError::NotFound`](README.md#libraryerror) for an unknown user,
[`LibraryError::Invalid`](README.md#libraryerror) for a scope that fails
[`PatronScope::validate`](#patronscopevalidate), or the write error.

### verify_patron_administrator

```rust
pub async fn verify_patron_administrator(
    store: &LibraryStore,
    user_id: i64,
) -> Result<ConfigActor>
```

Checks that `user_id` is an active Owner or Administrator and returns
the actor that may write the library's default scope.

#### Errors

Returns [`LibraryError::Forbidden`](README.md#libraryerror) for anyone else (including an unknown
user), and the read error when the lookup fails.

### work_scope_for_user

```rust
pub async fn work_scope_for_user(
    store: &LibraryStore,
    user: &UserRecord,
) -> Result<Option<WorkScope>>
```

The page index filter for `user`: `None` when nothing is hidden from
them (an Owner or Administrator, or a patron whose scope has no rules).

The filter names the user as its owner, so the books held by every
store account linked to them, connected or not, are the patron's own
and always pass.

#### Errors

Returns a read or parse error; callers fail closed on it.

### create_pending_join

```rust
pub async fn create_pending_join(
    store: &LibraryStore,
    join: &NewPendingJoin,
    state: &str,
    verifier: &str,
    keys: &LibraryLinkKeys,
) -> Result<PendingJoin>
```

Records a sign-in in progress: seals `verifier` and `keys` and writes the
row under `hash_token(state)`. Expired rows are purged first.

#### Errors

Returns [`LibraryError::Invalid`](README.md#libraryerror) for a malformed origin, pin, name or
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

Returns the read error, or [`LibraryError::Invalid`](README.md#libraryerror) when the sealed
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

Returns the read or write error, or [`LibraryError::Invalid`](README.md#libraryerror) when the
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
row keeps losing its compare-and-swap ([`LibraryError::Conflict`](README.md#libraryerror)); the
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
row keeps losing its compare-and-swap ([`LibraryError::Conflict`](README.md#libraryerror)).

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
(one move per pass, so the library does not thrash).

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
version, a wider grant, another archive) answers [`LibraryError::Conflict`](README.md#libraryerror) and nothing
is written. An automatic approval ([`RemoteApprover::Platform`](#remoteapprover)) is
likewise refused while a revoke is recorded (an operator approval
clears it) and over an operator's approval, which it would replace
along with its narrowing.

#### Errors

Returns [`LibraryError::NotFound`](README.md#libraryerror) when no request is recorded,
[`LibraryError::Conflict`](README.md#libraryerror) when it changed or an automatic approval
meets a revoke or an operator's approval, [`LibraryError::Invalid`](README.md#libraryerror) for a malformed field (an
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
    library_id: Option<&str>,
) -> Result<Vec<RemotePluginConsent>>
```

Every consent row, of one library when `library_id` is given, by
library then plugin key.

#### Errors

Returns an error when the read fails.

### load_remote_plugin_consent

```rust
pub async fn load_remote_plugin_consent(
    store: &LibraryStore,
    library_id: &str,
    plugin_key: &str,
) -> Result<Option<RemotePluginConsent>>
```

The consent row for `plugin_key` of `library_id`.

#### Errors

Returns an error when the read fails.

### record_remote_plugin_request

```rust
pub async fn record_remote_plugin_request(
    store: &LibraryStore,
    library_id: &str,
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

Returns [`LibraryError::Invalid`](README.md#libraryerror) for an empty or oversized field or a
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
    library_id: &str,
    plugin_key: &str,
) -> Result<Option<RemotePluginConsent>>
```

Clears the approval of `plugin_key` of `library_id`, keeping its
request, and records the revoke so no automatic approval brings it
back. Answers the row, or `None` when nothing is recorded.

#### Errors

Returns an error when a read or the write fails.

### accept_scan_pass_end

```rust
pub async fn accept_scan_pass_end(
    store: &LibraryStore,
    node_id: &str,
    job_id: &str,
    generation: i64,
    plugin_key: &str,
    end: HandedPassEnd<'_>,
) -> Result<ScanAccept>
```

Records how a Host's pass of one account ended.

A complete full pass listed every title the account holds, so the
account's holdings no row of this job and fence listed are removed
first. Then the sync token is stored and, for a complete pass, the
time of the last complete pass, as a local pass's end records them.

#### Returns

[`ScanAccept::Accepted`](#scanaccept) with the holdings removed in `upserted`, or why
nothing was recorded.

#### Errors

Returns an error when a read or a write fails.

### accept_scan_removed

```rust
pub async fn accept_scan_removed(
    store: &LibraryStore,
    node_id: &str,
    job_id: &str,
    generation: i64,
    plugin_key: &str,
    account_id: &str,
    product_ids: &[String],
) -> Result<ScanAccept>
```

Removes the holdings a Host's delta pass reported gone from one account.

#### Arguments

* `store` - The owner's library.
* `node_id` - The Host.
* `job_id` - The scan job.
* `generation` - The lease generation the Host holds.
* `plugin_key` - The job's source plugin.
* `account_id` - The account the ids left.
* `product_ids` - The ids, as `bookclerk_node_proto::scan_removed_problem`
  accepts them.

#### Returns

[`ScanAccept::Accepted`](#scanaccept) with the holdings removed in `upserted`, or why
nothing was removed.

#### Errors

Returns an error when a read or a write fails.

### accept_scan_results

```rust
pub async fn accept_scan_results(
    store: &LibraryStore,
    node_id: &str,
    job_id: &str,
    generation: i64,
    plugin_key: &str,
    books: Vec<impl Into<ScannedTitle>>,
) -> Result<ScanAccept>
```

Validates one batch of a Host's scan rows and writes it to the library.

The same as [`accept_scan_rows`](#accept_scan_rows) with no sightings beyond the rows.

#### Errors

Returns an error when a read or a write fails.

### accept_scan_rows

```rust
pub async fn accept_scan_rows(
    store: &LibraryStore,
    node_id: &str,
    job_id: &str,
    generation: i64,
    plugin_key: &str,
    books: Vec<impl Into<ScannedTitle>>,
    sightings: Vec<ScanSighting>,
) -> Result<ScanAccept>
```

Validates one batch of a Host's scan rows and writes it to the library,
marking every listed product seen by the Host's pass.

`books` are the rows as the guest produced them; their `uuid`,
`plugin_key` and `source` are ignored and set here (`source` from the
account row), so a Host cannot address an existing row by id or label
its rows as another plugin's. `sightings` are products the same batch
listed that map onto no row; they pass the same account checks and are
only marked seen, so a complete full pass's prune keeps them.

#### Errors

Returns an error when a read or a write fails.

### handed_pass_id

```rust
pub fn handed_pass_id(job_id: &str, generation: i64) -> String
```

The pass id a Host's scan of `job_id` under lease generation
`generation` stamps on the holdings its rows list
(`holdings.seen_pass`).

### handed_sync_token

```rust
pub async fn handed_sync_token(
    store: &LibraryStore,
    plugin_key: &str,
    account_id: &str,
) -> Result<String>
```

The sync token the owner hands with a Host scan of one account: the
stored token while the account's last complete pass is recent enough to
build on, otherwise empty (a full pass), as a local pass would start.

#### Errors

Returns an error when the read fails.

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

Returns [`LibraryError::NotFound`](README.md#libraryerror) when no node has pinned the plugin yet.

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

### align_library_root

```rust
pub async fn align_library_root(
    store: &LibraryStore,
    files_dir: &Path,
    password: Option<&str>,
) -> Result<LibrarySecret>
```

Ensures this process's DEK is the library secret root.

* Existing fingerprint: the local key must match. A missing file is not minted.
* No fingerprint and sealed secrets present: the local key must unseal one
  of them, then the fingerprint is recorded. A missing or wrong key fails
  without writing a new root.
* No fingerprint and no sealed secrets: the local key (minted when absent)
  is recorded. Concurrent initializers: one fingerprint wins; a loser that
  minted a different key deletes that new file and fails.

On success the accepted DEK is installed in the process cache. The returned
value is only the library id and fingerprint.

#### Errors

Returns an error when bootstrap material is missing or does not match the
database. Existing rows are not updated on failure.

### load_library_row

```rust
pub async fn load_library_row(store: &LibraryStore) -> Result<Option<Model>>
```

Reads the singleton library row when present.

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
another shared node) finds them again with this. An expired slot nobody
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
    lease: &LibraryLease,
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

### library_contents

```rust
pub async fn library_contents(store: &LibraryStore) -> Result<LibraryContents>
```

Counts the library data this install holds of its own.

#### Errors

Returns the read error.

### remove_shared_node

```rust
pub async fn remove_shared_node(
    store: &LibraryStore,
    actor: &EnrollmentActor,
    node_id: &str,
    this_node: &str,
) -> Result<()>
```

Removes shared node `node_id` from the library's node list, in one
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

The operator or an Owner may remove any shared node; another user only
one they own.

#### Errors

Returns [`LibraryError::NotFound`](README.md#libraryerror) for an unknown node or one the
caller may not see, [`LibraryError::Invalid`](README.md#libraryerror) for a branch node (revoke
it instead) or this node, [`LibraryError::Conflict`](README.md#libraryerror) while it is live,
and the write error.

### shared_nodes_visible_to

```rust
pub async fn shared_nodes_visible_to(
    store: &LibraryStore,
    actor: &EnrollmentActor,
) -> Result<Vec<NodeRecord>>
```

The shared nodes (`kind = shared`: this one and its shared branches) `actor` may
see, by the rule [`remove_shared_node`](#remove_shared_node) removes them: the operator or an
active Owner sees every one; another active user only the ones they own;
an unknown or disabled user none.

#### Errors

Returns the read error.

### strip_install_state

```rust
pub async fn strip_install_state(store: &LibraryStore) -> Result<TemplateStrip>
```

Removes this install's own state from the library so that copies of it
can start as installs of their own.

Deletes the library secret fingerprint, so the first daemon of each copy
records the key it mints instead of refusing it as a mismatch, plus the
library leases and node heartbeats left by the build. Library data,
plugin instances, deployments and the node rows stay. Run it only on a
library no daemon has open.

#### Errors

Returns an error, and changes nothing, when `encrypted_secrets`,
`operator_sessions`, `portal_sessions` or `claim_tickets` holds a row: a
template must never carry sealed secrets or sessions.
Returns the database error when a statement fails.

### accounts_owned_by

```rust
pub async fn accounts_owned_by(
    store: &LibraryStore,
    user_id: i64,
) -> Result<BTreeSet<AccountKey>>
```

Store accounts linked to `user_id` through any of the user's identities.

Read in keyset pages no larger than the database plugin's
`maxResultRows`.

#### Errors

Returns an error when a read fails.

### acquire_titles_of_editions

```rust
pub async fn acquire_titles_of_editions(
    store: &LibraryStore,
    requester: ScanRequester,
    account: Option<&str>,
    plugin_instance_id: Option<&str>,
    editions: &[String],
) -> Result<Vec<String>>
```

The library titles an acquire by edition queues: for each edition, one
title (`books.uuid`) that sells it in an account `requester` may
acquire for, and in `account` when one is named.

An edition's titles share its files, so acquiring one of them acquires
the edition. Each title is still planned by [`plan_acquire`](#plan_acquire), which
checks the requester again.

#### Arguments

* `store` - The library.
* `requester` - Who asks; a member is limited to their own accounts.
* `account` - When named (an id or label), only titles in that account.
* `plugin_instance_id` - When the acquire is pinned to a plugin
  instance, only titles on that instance's plugin, so the title picked
  is one the pinned plan keeps.
* `editions` - Edition ids (`editions.id`); blanks and repeats are
  skipped.

#### Returns

One title per edition, named by its account holding (`holdings.id`,
which an acquire job's title accepts), in the order the editions were
given.

#### Errors

- [`LibraryError::Invalid`](README.md#libraryerror) when every edition id given is blank, the
  pinned instance id does not parse, or an edition has no title in the
  library (or none in `account` or on the pinned plugin).
- [`LibraryError::Forbidden`](README.md#libraryerror) when an edition's titles are all in
  accounts the requester may not acquire for.
- [`LibraryError::Orm`](README.md#libraryerror) when a read fails.

### claimant_may_run

```rust
pub fn claimant_may_run(claimant: Claimant, job: &JobRecord) -> bool
```

True when a `claimant` may run `job` at all, whatever it is trusted
with: a forced, PDF-only or paced acquire runs only on a node that holds
the library database, since the host acquire path does none of them (a
paced acquire checks and records the account's pace in the library
database before each download, `crate::pacing`). An
interlibrary request ([`JobKind::BranchRequest`](README.md#jobkind)) is never run through
the routed claim: only its branch claims it, over `requests.claim`
(`super::claim_branch_requests`). An ingest batch and a notification
send run only on a node that holds the library database.

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
- the node is still trusted with the job under the claim's scope
  ([`node_may_claim_job_in_scope`](#node_may_claim_job_in_scope)).

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

### job_for_node_owner

```rust
pub async fn job_for_node_owner(
    store: &LibraryStore,
    node_id: &str,
    job: &JobRecord,
) -> Result<bool>
```

True when every store account `job` touches belongs to the user who
owns `node_id`: the job is the branch owner's own work
(`bookclerk_node_proto::WorkJob::for_node_owner`).

#### Errors

Returns an error when a read fails.

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
- An acquire touches the named account's rows, plus every account
  holding a title its filters match (the title id as
  [`LibraryStore::acquire_titles_named`](README.md#librarystoreacquire_titles_named) matches it, the account and the
  pinned plugin), whatever the edition's acquire status. That is a
  superset of the titles it will fetch, so a node must be trusted with
  at least everything it fetches.
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

True when `node_id` may claim `job` right now, whoever's work it is.

#### Errors

Returns an error when a read fails.

### node_may_claim_job_in_scope

```rust
pub async fn node_may_claim_job_in_scope(
    store: &LibraryStore,
    node_id: &str,
    claimant: Claimant,
    scope: ClaimScope,
    job: &JobRecord,
) -> Result<bool>
```

True when `node_id` may claim `job` right now under a claim of `scope`
([`node_trust_in_scope`](#node_trust_in_scope)).

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

### node_trust_in_scope

```rust
pub async fn node_trust_in_scope(
    store: &LibraryStore,
    node_id: &str,
    claimant: Claimant,
    scope: ClaimScope,
) -> Result<NodeTrust>
```

What `node_id` is trusted with for a claim of `scope`.

`ClaimScope::Own` narrows a Host trusted with every job
([`NodeTrust::All`](#nodetrust), an Owner's or Administrator's) to the work of the
user who owns it ([`NodeTrust::AllOfOwner`](#nodetrust), which still admits that
user's pinned jobs and jobs with requirements), as a branch takes only
its owner's work unless its
operator turned on "Download for other members". A scope never widens
trust, and a shared node, which shares the database, keeps every job.

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

Returns [`LibraryError::Forbidden`](README.md#libraryerror) when the user is unknown or disabled,
or names an account or title that is not theirs (unknown ones read the
same), [`LibraryError::Invalid`](README.md#libraryerror) for a malformed instance id, and the
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

Returns [`LibraryError::Forbidden`](README.md#libraryerror) when the user is unknown or disabled,
or names an account that is not theirs (an unknown account reads the
same), [`LibraryError::Invalid`](README.md#libraryerror) for a malformed instance id, and the
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
again without writing. A title's commit also records the store
download in its account's pace ledger (`account_downloads`), in the same
batch.

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
title's recorded audio, PDF or audio sidecar (`claims_another_titles_key`)
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
) -> Result<Option<Vec<AcquireTitle>>>
```

The titles the acquire `job` run by `node_id` covers, read from the
account holdings as a local acquire reads them: of the job's plugins
([`job_plugin_keys`](#job_plugin_keys)), of the accounts its account selector names
(`job_accounts`, so a job naming its account by label or in another
case covers that account's titles) and its title when it names one
([`LibraryStore::acquire_titles_named`](README.md#librarystoreacquire_titles_named), matched as the executor
matches it), and, for a Host trusted only with its owner's accounts,
one of that user's. `None` when the node is trusted with nothing.

Without a title the holdings are read in keyset pages
([`LibraryStore::acquire_titles_page`](README.md#librarystoreacquire_titles_page)), so no read loads the whole
library at once.

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
recorded latest writer is this upload (an upload id is unique, and
claims keys for its own title only). A later upload of the title, an
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

### clear_window_books

```rust
pub async fn clear_window_books(
    store: &LibraryStore,
    parent_library_id: &str,
) -> Result<u64>
```

Drops every window book of library `parent_library_id` (its link
downloads again, or was left).

#### Returns

How many rows were dropped.

#### Errors

Returns the write error.

### discard_window_read

```rust
pub async fn discard_window_read(
    store: &LibraryStore,
    parent_library_id: &str,
    seen_at: &str,
) -> Result<u64>
```

Drops the rows a read cut short by a settings change wrote (stamped
`seen_at`), so a read that began before downloading was turned on
cannot put back the list the change cleared.

#### Returns

How many rows were dropped.

#### Errors

Returns the write error.

### finish_window_read

```rust
pub async fn finish_window_read(
    store: &LibraryStore,
    parent_library_id: &str,
    seen_at: &str,
) -> Result<u64>
```

Ends a complete read of library `parent_library_id`'s window books: drops
its rows that read (stamped `seen_at`) did not name. Rows are compared
by stamp equality, not order, so a clock moved back between reads still
drops what the new read left out.

#### Returns

How many rows were dropped.

#### Errors

Returns the write error.

### person_books_page

```rust
pub async fn person_books_page(
    store: &LibraryStore,
    user_id: i64,
    after: &WindowCursor,
) -> Result<(Vec<WindowWork>, Option<WindowCursor>)>
```

One page of this library's books for a person, as a clerk signed in
with that person's library card reads them over the client
socket (`GET /api/clerk/v1/connect`): the same works, grouping and
limits as [`window_books_page`](#window_books_page), for the person the card belongs to
rather than a branch node's owner.

After the books on its store accounts, a clerk's pages go on with the
books only the library's branches hold, each edition
`Availability::Reachable`, for the person to ask the library for with
its member request route (`POST /api/branch-requests`).

Both parts follow the person's patron scope: only books it shows them
are listed, judged on this library, so a clerk never learns of the
rest.

#### Arguments

* `user_id` - The person: the card's user on this library.
* `after` - The cursor of the previous page (empty for the first).

#### Returns

The page's works and the cursor of the next page (`None` on the last).

#### Errors

Returns the read error.

### record_window_page

```rust
pub async fn record_window_page(
    store: &LibraryStore,
    parent_library_id: &str,
    seen_at: &str,
    works: &[WindowWork],
) -> Result<usize>
```

Stores one `window.page` from library `parent_library_id`, read under
`seen_at` ([`window_read_stamp`](#window_read_stamp)).

Each edition becomes (or refreshes) one `window_books` row. Each work is
placed on a work of this install's library when its strong identifiers
reach exactly one; none otherwise. An ISBN never places a work: one can
sit on editions of different works, and the local work joins rows from
every library in the view.

#### Arguments

* `works` - The page, already validated against the frame rules.

#### Returns

How many editions were stored.

#### Errors

Returns a read or write error.

### window_books_page

```rust
pub async fn window_books_page(
    store: &LibraryStore,
    node_id: &str,
    after: &WindowCursor,
) -> Result<Option<(Vec<WindowWork>, Option<WindowCursor>)>>
```

One page of the books this library holds for branch `node_id`'s owner,
for the `window.books` frame.

The library's titles (every account holding, each marked the person's
when it is on one of their accounts) are read in holding id order from
cursor `after` (at most [`WINDOW_BOOKS_READ`](#window_books_read) holdings, and
no more than one database result page); the next cursor is the last
holding id the page covered. Each title is grouped under its work here
(its key is the work id, or `edition-<edition id>` when the edition has
no work), with the work's identifiers; a work
lists at most [`MAX_LOOKUP_EDITIONS`](../bookclerk_node_proto/README.md#max_lookup_editions) editions (where two accounts hold
the same edition, one entry with every format either copy has, the
person's when either account is).
The page stops early once it would pass the frame budget or
[`MAX_WINDOW_WORKS`](../bookclerk_node_proto/README.md#max_window_works) works; the books after that come on the next page.

#### Arguments

* `node_id` - The asking node; its owner is the person.
* `after` - The cursor of the previous page (empty for the first).

#### Returns

`None` when `node_id` is not a branch this library trusts; otherwise
the page's works and the cursor of the next page (`None` on the last).

#### Errors

Returns the read error.

### window_read_stamp

```rust
pub fn window_read_stamp() -> String
```

A stamp for one complete read of a library's window books: pass it to
every [`record_window_page`](#record_window_page) of the read and then to
[`finish_window_read`](#finish_window_read).

### window_view

```rust
pub async fn window_view(store: &LibraryStore) -> Result<Vec<WindowEntry>>
```

The merged view of every library this install is a branch of or holds
a library card for: their books, one entry per work, sorted by title.

Every active link counts, and every active library card ([`crate::control_plane::list_clerk_links`](#list_clerk_links)). Rows
are the same work when they reach the same work of this library, share
a strong identifier (not an ISBN), or carry the same grouping key from
one library. A row whose stored edition is missing or no longer parses
is skipped.

#### Errors

Returns the read error.

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

Returns an error when the secret root, library binding, schema, or a core
document cannot be established.

### overlay_core_documents

```rust
pub fn overlay_core_documents(config: &mut Config, session: &ControlPlaneSession)
```

Copies every authoritative library document in `session` onto `config`.

### overlay_events

```rust
pub fn overlay_events(
    config: &mut Config,
    events: &ConfigurationDocument<EventsSettingsV1>,
    library_id: &str,
)
```

Copies the authoritative events document onto `config` for `library_id`.

Revisions are only comparable within one library. Callers that switch
databases pass the new library id explicitly.

### load_events

```rust
pub async fn load_events(
    store: &LibraryStore,
) -> Result<ConfigurationDocument<EventsSettingsV1>>
```

Loads the library events document.

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

Inserts `[events]` when the library document is absent.

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

Replaces the library events document.

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

### HeldBookKey

```rust
pub type HeldBookKey = (String, String, String, String);
```

A book this library holds, by store identity: `(plugin_key, stored
marketplace, stored product id, account_id)`.

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

### ADMIN_CONTROLS_NAMESPACE

```rust
pub const ADMIN_CONTROLS_NAMESPACE: &str = "core.admin_controls";
```

Namespace of the library document that holds the library's default
Administrator switches.

### MAX_ARTIFACT_BLOB_BYTES

```rust
pub const MAX_ARTIFACT_BLOB_BYTES: usize = 1_048_576;
```

Largest archive stored in the library, in bytes.

A larger local archive is referenced in place as a `file://` URL.

### DETACHED_LEFT

```rust
pub const DETACHED_LEFT: &str = "left";
```

Removal reason: the branch left the library (or was revoked).

### DETACHED_STOPPED_SHARING

```rust
pub const DETACHED_STOPPED_SHARING: &str = "stopped_sharing";
```

Removal reason: the branch turned "Share this branch's books" off.

### DETACHED_WITHDRAWN

```rust
pub const DETACHED_WITHDRAWN: &str = "withdrawn";
```

Removal reason: a full pass of the branch's catalog no longer named it.

### FORWARDABLE_KEYS_MAX_PAGE

```rust
pub const FORWARDABLE_KEYS_MAX_PAGE: u64 = 256;
```

Upper clamp for one [`forwardable_branch_title_keys_page`](#forwardable_branch_title_keys_page) read.

A key row is a handful of short columns, so this many stay far below the
256 KiB inline result cap of an adapter that does not stream.

### FULL_CATALOG_PASS_EVERY

```rust
pub const FULL_CATALOG_PASS_EVERY: Duration = _;
```

How long a branch's catalog sync resumes incrementally before the
library asks for a full pass again, so titles a branch no longer has
(removed, or from a sub-branch that left) are withdrawn within a day.

### REQUEST_ACCEPTED

```rust
pub const REQUEST_ACCEPTED: &str = "accepted by the branch; the book arrives with its catalog sync";
```

Progress text of a request its branch accepted.

### REQUEST_BRANCH_FAILED

```rust
pub const REQUEST_BRANCH_FAILED: &str = "branch_failed";
```

`error_kind` of a request its branch could not take now.

### REQUEST_LEASE_SECS

```rust
pub const REQUEST_LEASE_SECS: u64 = 300;
```

How long a branch holds a claimed request before it goes back to
pending. Taking a request is a few local reads and writes on the
branch, so this is generous.

### REQUEST_NOT_LISTED

```rust
pub const REQUEST_NOT_LISTED: &str = "not_listed";
```

`error_kind` of a request its branch no longer lists.

### REQUEST_WORK_GONE

```rust
pub const REQUEST_WORK_GONE: &str = "work_gone";
```

`error_kind` of a request whose work has nothing left to ask by (it was
merged away, or lost every identifier) when its branch came for it.

### CACHE_NAMESPACE

```rust
pub const CACHE_NAMESPACE: &str = "core.cache";
```

Namespace of the library cache document.

### CACHE_SCHEMA_VERSION

```rust
pub const CACHE_SCHEMA_VERSION: i64 = 1;
```

Schema version of [`CacheSettingsV1`](#cachesettingsv1).

### CACHE_SETTING_KEYS

```rust
pub const CACHE_SETTING_KEYS: &[&str] = _;
```

Settings keys the cache document owns, in display order.

### SHARE_RECEIVED

```rust
pub const SHARE_RECEIVED: &str = "received";
```

`account_shares.direction` of an account this install received from a
library it joined.

### SHARE_SENT

```rust
pub const SHARE_SENT: &str = "sent";
```

`account_shares.direction` of an account this library sent to a branch.

### BRANCH_LISTING_PREFIX

```rust
pub const BRANCH_LISTING_PREFIX: &str = "branch:";
```

Plugin key prefix of the listings branches answer with.

### BRANCH_LISTING_TTL

```rust
pub const BRANCH_LISTING_TTL: Duration = _;
```

How long a `branch:<node>` listing counts as current after the branch
last named it: a day of answer life plus a day for the next question to
be asked and answered.

### LOOKUP_ANSWER_TTL

```rust
pub const LOOKUP_ANSWER_TTL: Duration = _;
```

How long a branch's answer stands: a question answered longer ago is
asked again the next time [`ask_branches`](#ask_branches) names its work.

### LOOKUP_RESEND

```rust
pub const LOOKUP_RESEND: Duration = _;
```

How long a sent question waits for its answer before it is sent again.

### CLERK_CARD_NAME

```rust
pub const CLERK_CARD_NAME: &str = "library_card";
```

`encrypted_secrets.name` of a clerk link's library card.

### COLLECTIONS_NAMESPACE

```rust
pub const COLLECTIONS_NAMESPACE: &str = "core.collections";
```

Namespace of the library document that holds the library's collections.

### COLLECTIONS_SCHEMA_VERSION

```rust
pub const COLLECTIONS_SCHEMA_VERSION: i64 = 1;
```

Schema version of [`LibraryCollections`](#librarycollections) in the document.

### MAX_COLLECTIONS

```rust
pub const MAX_COLLECTIONS: usize = 32;
```

Most collections a library may define.

### MAX_COLLECTIONS_JSON_BYTES

```rust
pub const MAX_COLLECTIONS_JSON_BYTES: usize = _;
```

Largest set of collections as compact JSON, in bytes: well inside the
largest configuration document.

### MAX_COLLECTION_NAME_BYTES

```rust
pub const MAX_COLLECTION_NAME_BYTES: usize = 64;
```

Longest collection name, in UTF-8 bytes after trimming.

### MAX_COLLECTION_PREFIXES

```rust
pub const MAX_COLLECTION_PREFIXES: usize = 8;
```

Most prefixes one collection may have.

### MAX_COLLECTION_PREFIX_BYTES

```rust
pub const MAX_COLLECTION_PREFIX_BYTES: usize = 256;
```

Longest prefix, in UTF-8 bytes after trimming and the closing `/`.

### MAX_ADD_CONNECTIONS

```rust
pub const MAX_ADD_CONNECTIONS: usize = bookclerk_node_proto::MAX_BUNDLE_ACCOUNTS;
```

Most accounts one "Add my connections" request names: as many as one
sealed account bundle carries.

### COPY_TOKEN_TTL_SECS

```rust
pub const COPY_TOKEN_TTL_SECS: i64 = _;
```

Seconds a copy token is accepted after its grant.

### CRON_FIRE_MAX_ATTEMPTS

```rust
pub const CRON_FIRE_MAX_ATTEMPTS: i64 = 3;
```

Failure attempts a cron fire gets before it is terminal (the first run
plus two retries with the queue's backoff).

### MAX_OBSERVATION_DETAIL_BYTES

```rust
pub const MAX_OBSERVATION_DETAIL_BYTES: usize = 512;
```

Maximum UTF-8 size of an observation detail string.

### DESTINATION_NAMESPACE

```rust
pub const DESTINATION_NAMESPACE: &str = "core.destination";
```

Namespace of a destination's host-owned placement document.

### DESTINATION_SCHEMA_VERSION

```rust
pub const DESTINATION_SCHEMA_VERSION: i64 = 1;
```

Schema version of [`DestinationPlacementV1`](#destinationplacementv1).

### LIBRARY_SCOPE_ID

```rust
pub const LIBRARY_SCOPE_ID: &str = "singleton";
```

Library scope instance. There is one library document per namespace.

### LIBRARY_SCOPE_TYPE

```rust
pub const LIBRARY_SCOPE_TYPE: &str = "library";
```

Scope type stored for library documents.

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

### BRANCH_ENROLLMENT_NAMESPACE

```rust
pub const BRANCH_ENROLLMENT_NAMESPACE: &str = "core.branch_enrollment";
```

Namespace of the library's branch-enrollment switch.

### BRANCH_ENROLLMENT_SCHEMA_VERSION

```rust
pub const BRANCH_ENROLLMENT_SCHEMA_VERSION: i64 = 1;
```

Schema version of [`BranchEnrollmentSettingsV1`](#branchenrollmentsettingsv1).

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

### MAX_BATCH_FILES

```rust
pub const MAX_BATCH_FILES: usize = 10_000;
```

Most files one batch's manifest may name.

### MAX_INTAKE_REF_BYTES

```rust
pub const MAX_INTAKE_REF_BYTES: usize = 1_024;
```

Longest intake storage key or signature a member may carry, in bytes;
both are stored in one cell.

### MAX_REASON_BYTES

```rust
pub const MAX_REASON_BYTES: usize = 128;
```

Longest outcome reason, in bytes.

### INGRESS_NAMESPACE

```rust
pub const INGRESS_NAMESPACE: &str = "core.ingress";
```

Namespace of the library document that holds the ingress settings.

### INGRESS_SCHEMA_VERSION

```rust
pub const INGRESS_SCHEMA_VERSION: i64 = 1;
```

Schema version of [`IngressSettings`](#ingresssettings).

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
[`crate::apply_host_schema_with_batch_opts`](README.md#apply_host_schema_with_batch_opts)).

### RECORD_RETENTION_LEASE

```rust
pub const RECORD_RETENTION_LEASE: &str = "maintenance.records";
```

Lease of the record retention duty (expired credentials and tokens,
finished uploads, abandoned scan inventories, old audit rows).

### SESSION_EXPIRY_LEASE

```rust
pub const SESSION_EXPIRY_LEASE: &str = "maintenance.sessions";
```

Lease of the session duty (expired operator sessions, OIDC RP states,
WebAuthn challenges).

### LIBRARY_LINK_ACTIVE

```rust
pub const LIBRARY_LINK_ACTIVE: &str = "active";
```

`library_links.state` of a link the host client connects.

### LIBRARY_LINK_KEYS_NAME

```rust
pub const LIBRARY_LINK_KEYS_NAME: &str = "node_keys";
```

`encrypted_secrets.name` of a link's node keys.

### LIBRARY_LINK_LEFT

```rust
pub const LIBRARY_LINK_LEFT: &str = "left";
```

`library_links.state` after this install left the library.

### LIBRARY_LINK_REVOKED

```rust
pub const LIBRARY_LINK_REVOKED: &str = "revoked";
```

`library_links.state` once the owner revoked this node (close 4403).

### MEMBER_CONTROLS_NAMESPACE

```rust
pub const MEMBER_CONTROLS_NAMESPACE: &str = "core.member_controls";
```

Namespace of the library document that holds the library's default
member controls.

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

### EXPLICIT_FACET_KEY

```rust
pub const EXPLICIT_FACET_KEY: &str = "explicit";
```

The page index value a work marked explicit carries under
[`WorkFacet::Explicit`](README.md#workfacet).

### MAX_SCOPE_JSON_BYTES

```rust
pub const MAX_SCOPE_JSON_BYTES: usize = 1024;
```

Largest scope as compact JSON, in bytes: one cell of the smallest reply
the host accepts from a database plugin
([`HOST_MIN_CELL_BYTES`](../bookclerk_plugin_abi/db_execute.md#host_min_cell_bytes)),
so a stored scope can be read back on every supported database. Sixteen
rules fit when their values are short.

### MAX_SCOPE_RULES

```rust
pub const MAX_SCOPE_RULES: usize = 16;
```

Most rules one scope may hold.

### MAX_SCOPE_VALUE_BYTES

```rust
pub const MAX_SCOPE_VALUE_BYTES: usize = 256;
```

Longest rule value, in UTF-8 bytes.

### PATRON_SCOPE_NAMESPACE

```rust
pub const PATRON_SCOPE_NAMESPACE: &str = "core.patron_scope";
```

Namespace of the library document that holds the library's default
scope.

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

Name of the library lease the scheduler runs under.

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

Library lease the daemon holds while it sweeps.

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

Artifact URL scheme of an archive stored in the library
([`super::artifacts::put_artifact_blob`](#put_artifact_blob)).

### WINDOW_BOOKS_READ

```rust
pub const WINDOW_BOOKS_READ: u64 = 200;
```

Most holdings one [`window_books_page`](#window_books_page) reads.

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
