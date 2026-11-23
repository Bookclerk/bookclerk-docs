# bookclerk_library

Canonical Bookclerk library database.

[`LibraryStore`](#librarystore) is SeaORM-backed: every backend is a
[`sea_orm::DatabaseConnection`](https://docs.rs/sea_orm/latest/sea_orm/database/db_connection/struct.DatabaseConnection.html). Engine connect/migrate/proxy quirks live in
the database plugin (`bookclerk-plugin-database-sqlite` (and optional d1/postgres guests)); hosts open the store via
the external guest RPC. This crate owns the greenfield schema
([`migrations`](migrations.md)), typed [`entities`](entities/README.md), and the store API. See
`docs/server/database.md`.

| Item | Kind | Summary |
| --- | --- | --- |
| [`AtomicTxnBackend`](#atomictxnbackend) | trait | Backend that runs `crate::LibraryStore` named security operations as a single guest atomic batch. |
| [`TypedAtomicExec`](#typedatomicexec) | trait | Runs a host-authorized typed `bookclerk_plugin_abi::ExecuteRequest` as one guest `executeAtomic`. |
| [`NamedAtomicRunner`](#namedatomicrunner) | trait | Runs one compiled named plan as a single atomic batch. |
| [`DbAtomicResult`](#dbatomicresult) | struct | Host-interpreted result of a named atomic operation. |
| [`BackendMigrateOptions`](#backendmigrateoptions) | struct | Options for a backend-to-backend library copy. |
| [`BackendMigrateSummary`](#backendmigratesummary) | struct | Per-table row counts copied (or that would be copied in dry-run mode). |
| [`BackupRepository`](#backuprepository) | struct | On-disk backup repository rooted at `files_dir/backups`. |
| [`BackupListEntry`](#backuplistentry) | struct | Catalog entry used by `bookclerk db backup list`. |
| [`BackupManifest`](#backupmanifest) | struct | On-disk recovery-point metadata (`manifests/<id>.json`). |
| [`BackupOutcome`](#backupoutcome) | struct | Result of publishing a recovery point. |
| [`BackupRequest`](#backuprequest) | struct | Inputs for an automatic or CLI backup. |
| [`BackupUnit`](#backupunit) | struct | One logical database in a recovery point (library or plugin binding). |
| [`CanonicalDatabaseSchema`](#canonicaldatabaseschema) | struct | Admitted canonical schema used to export or restore one database unit. |
| [`CanonicalExportOpts`](#canonicalexportopts) | struct | Options for a consistent canonical export. |
| [`CanonicalRestoreOpts`](#canonicalrestoreopts) | struct | Negotiated limits for parameterized canonical restore. |
| [`CanonicalTableSchema`](#canonicaltableschema) | struct | One admitted `CREATE TABLE` plus its parsed IR. |
| [`IdentityHighWater`](#identityhighwater) | struct | Identity high-water for one generated column. |
| [`PreparedPluginUnit`](#preparedpluginunit) | struct | Open plugin binding captured through the active adapter session. |
| [`RestorePlan`](#restoreplan) | struct | Library restore result plus plugin units for the host to apply. |
| [`SchemaBackupOpts`](#schemabackupopts) | struct | Optional in-place backup taken before applying ups or downs. |
| [`ValidatedBackup`](#validatedbackup) | struct | Recovery point whose objects, schema, and typed rows have been verified. |
| [`Missing`](#missing) | struct | What a `LibraryError::NotFound` could not find: the kind of row and, when the caller named one, its id. |
| [`SchemaApplyOptions`](#schemaapplyoptions) | struct | Options for `apply_host_schema_with_options`. |
| [`SchemaBatch`](#schemabatch) | struct | Canonical schema apply batch: already-separate host statements plus marker. |
| [`InProcessSqlite`](#inprocesssqlite) | struct | Runs compiled named plans on a local SeaORM SQLite connection. |
| [`PluginMigrationHistory`](#pluginmigrationhistory) | struct | Ordered durable plugin migration history. |
| [`PluginMigrationSequence`](#pluginmigrationsequence) | struct | Proven, ordered plugin migration sequence. |
| [`ProvenPluginMigration`](#provenpluginmigration) | struct | One proven registered migration. |
| [`AccountLinkRecord`](#accountlinkrecord) | struct | Link between a portal identity and a bookstore account. |
| [`AccountRecord`](#accountrecord) | struct | Account row stored in the Bookclerk DB. |
| [`AcquireStatusCounts`](#acquirestatuscounts) | struct | Held work counts per `AcquireStatus`, read in one grouped query (see `LibraryStore::count_works_by_status`). |
| [`BranchRequestTarget`](#branchrequesttarget) | struct | What a `JobKind::BranchRequest` asks its branch for: a work of this library, and the store listing the branch answered for it (`control_plane::catalog_lookup`). The branch is told the work by its identifiers, never by this id. |
| [`ClaimTicketRecord`](#claimticketrecord) | struct | Claim ticket metadata (token plaintext is never stored). |
| [`DomainEventRecord`](#domaineventrecord) | struct | Immutable outbox envelope. |
| [`EmbeddingRecord`](#embeddingrecord) | struct | Stored embedding vector metadata (blob fetched separately when needed). |
| [`EnqueueJobSpec`](#enqueuejobspec) | struct | Admission request for `crate::LibraryStore::enqueue_job`. |
| [`EventCatalogSubscription`](#eventcatalogsubscription) | struct | One `[capabilities.events.subscriptions]` row persisted in `event_subscriber_nodes`. |
| [`EventDeliveryFence`](#eventdeliveryfence) | struct | Claim fence for an event delivery (mirrors `JobFence`). |
| [`EventDeliveryMetrics`](#eventdeliverymetrics) | struct | Operator-visible delivery-queue counters for `GET /api/status`. |
| [`EventDeliveryRecord`](#eventdeliveryrecord) | struct | Per-subscriber delivery row. |
| [`EventStreamPage`](#eventstreampage) | struct | One page of `EventStreamRead`. |
| [`EventStreamRead`](#eventstreamread) | struct | One page request over the domain-event outbox (`EventPublisher.read`). |
| [`EventSubscriber`](#eventsubscriber) | struct | Subscriber snapshot used when creating deliveries for one event. |
| [`EventSubscriberCatalogRecord`](#eventsubscribercatalogrecord) | struct | Library-authoritative catalog row after collapsing live nodes by `plugin_id`. |
| [`EventSubscriberNodeRecord`](#eventsubscribernoderecord) | struct | Live per-node catalog row (`event_subscriber_nodes`). |
| [`EventWakeGrant`](#eventwakegrant) | struct | Host-derived wake grant: schema versions plus an intersected payload filter. |
| [`GlobalQueueEntry`](#globalqueueentry) | struct | Aggregated global request-queue entry (one work, many wishers). |
| [`JobClaimCursor`](#jobclaimcursor) | struct | Where the next page of routed-claim candidates starts (keyset paging). |
| [`JobClaimPage`](#jobclaimpage) | struct | One page of routed-claim candidates. |
| [`JobFence`](#jobfence) | struct | Lease identity a worker must present to mutate a running job. |
| [`JobPayload`](#jobpayload) | struct | JSON command envelope stored on a job row. |
| [`JobRecord`](#jobrecord) | struct | Durable job row returned to the daemon API and worker. |
| [`JobTempPath`](#jobtemppath) | struct | Scratch path registered against a job for crash cleanup. |
| [`ListeningProgressRecord`](#listeningprogressrecord) | struct | Listening progress snapshot from an external player (e.g. AudioBookshelf). |
| [`OidcClientRecord`](#oidcclientrecord) | struct | Registered OIDC authorization-server client (Bookclerk as IdP). |
| [`OperatorSessionRecord`](#operatorsessionrecord) | struct | Durable operator session metadata (hashed token is never exposed). |
| [`PendingWakeProgress`](#pendingwakeprogress) | struct | How far one claimed wake tick got. |
| [`PortalIdentity`](#portalidentity) | struct | Portal identity bound to an external provider user (e.g. ABS). |
| [`PortalSessionRecord`](#portalsessionrecord) | struct | Portal session row for session lists (includes hash for current-session match). |
| [`PublishDomainEventSpec`](#publishdomaineventspec) | struct | Producer-side domain event (or queue message) to persist in the outbox. |
| [`QueueWisher`](#queuewisher) | struct | Compact wisher shown on the global queue (avatar + label). |
| [`SecurityAuditEvent`](#securityauditevent) | struct | Security audit event (elevate / impersonate / login / provision). |
| [`StoredPasskey`](#storedpasskey) | struct | Stored WebAuthn credential (passkey JSON plus the label shown in Settings). |
| [`TasteHolder`](#tasteholder) | struct | A storefront account that holds a work, as a `TasteTitle` lists it. |
| [`TasteTitle`](#tastetitle) | struct | One work the library holds, as Discover reads it for taste and seeds. |
| [`TitleRequestRecord`](#titlerequestrecord) | struct | Personal wishlist row (also contributes to the shared global queue while open). |
| [`TitleRequestSourceRecord`](#titlerequestsourcerecord) | struct | Per-storefront catalog/pricing snapshot attached to a wishlist row. |
| [`UserIntegrationHint`](#userintegrationhint) | struct | Storefront / integration connection summary for admin user lists. |
| [`UserInviteRecord`](#userinviterecord) | struct | Invite ticket for provisioning a User (token plaintext never stored). |
| [`UserListeningHint`](#userlisteninghint) | struct | Recent unfinished listening hint for admin user lists. |
| [`UserPreferences`](#userpreferences) | struct | Per-user GUI / Discover preferences (operator or portal identity). |
| [`UserPresenceExtras`](#userpresenceextras) | struct | Presence + connection extras for administrator user management. |
| [`UserRecord`](#userrecord) | struct | First-party Bookclerk user (security principal for portal paths). |
| [`UserSsoPicture`](#userssopicture) | struct | IdP-supplied avatar available as a profile-picture choice. |
| [`UserTitleState`](#usertitlestate) | struct | One user's own state for one work (`user_title_state`). |
| [`WishlistPurchaseHint`](#wishlistpurchasehint) | struct | Snapshotted purchase link/price for a wishlist title. |
| [`WishlistStoreEdition`](#wishliststoreedition) | struct | Storefront edition key for wishlist / queue payloads. |
| [`WorkRecord`](#workrecord) | struct | Canonical work spanning one or more ownership rows / editions. |
| [`NamedAtomicBackend`](#namedatomicbackend) | struct | An `AtomicTxnBackend` that compiles every named operation and hands the plan to a `NamedAtomicRunner`. |
| [`SchemaWalk`](#schemawalk) | struct | Result of walking a host migration plan from one version to another. |
| [`SessionClientInfo`](#sessionclientinfo) | struct | Parsed client presentation for a session row. |
| [`FacetValue`](#facetvalue) | struct | One value of a facet value list. |
| [`FacetValuesPage`](#facetvaluespage) | struct | One page of a facet value list. |
| [`LibraryWorkRow`](#libraryworkrow) | struct | One row of the page index, as list views show it. |
| [`UnheldItem`](#unhelditem) | struct | What a catalog item the library does not hold says about itself, read by `WorkScope::admits_unheld`. People are comma- or semicolon-separated, genres as the page index splits them. |
| [`WorkCursor`](#workcursor) | struct | Where the next page of a `WorkSort` order starts: the sort values of the last row returned. |
| [`WorkPage`](#workpage) | struct | One page of library works. |
| [`WorkPageQuery`](#workpagequery) | struct | Filters and position of one page of library works. |
| [`WorkScope`](#workscope) | struct | The part of the library a patron may see, as the page index applies it. |
| [`EditionDetail`](#editiondetail) | struct | One edition of a work with its expression and listings. |
| [`EditionListing`](#editionlisting) | struct | One store listing of an edition. |
| [`ListingHolding`](#listingholding) | struct | One holding of a listing. |
| [`WorkDetail`](#workdetail) | struct | A top-level work with everything its detail view shows. |
| [`WorkSearchDoc`](#worksearchdoc) | struct | The searchable text of one held top-level work. |
| [`WorkTag`](#worktag) | struct | One library tag on a work. |
| [`WorkTagsBody`](#worktagsbody) | struct | `PUT /api/library/works/{id}/tags` body: the tags an Administrator sets. |
| [`WorkTagsRequest`](#worktagsrequest) | struct | A validated `WorkTagsBody`: at most `MAX_WORK_TAGS` distinct tags (by `facet_key`), trimmed, each 1 to `MAX_WORK_TAG_BYTES` bytes, in the order sent with the first spelling kept. |
| [`CatalogListing`](#cataloglisting) | struct | One title a store's catalog answer reported. |
| [`CatalogListingRecord`](#cataloglistingrecord) | struct | A stored catalog listing, as Discover and the wishlist read it. |
| [`ListingDetail`](#listingdetail) | struct | The store's own text for a listing, beyond the title the resolver sees. |
| [`ListingOffer`](#listingoffer) | struct | One price a store quoted for one listing and audience. |
| [`RecordedListings`](#recordedlistings) | struct | What `LibraryStore::record_catalog_listings` stored for one answer. |
| [`Chapter`](#chapter) | struct | One chapter of a checked list. |
| [`ChapterClaim`](#chapterclaim) | struct | One stored chapter claim, without its list. |
| [`ChapterClaimWrite`](#chapterclaimwrite) | struct | A claim to record (`LibraryStore::record_chapter_claim`). |
| [`ChapterDigest`](#chapterdigest) | struct | Lower-case hex SHA-256 naming one stored chapter list (`ChapterList::digest`). |
| [`ChapterInput`](#chapterinput) | struct | One chapter as a source states it, before it is checked. |
| [`ChapterList`](#chapterlist) | struct | A checked chapter list, in pre-order with starts never decreasing. |
| [`ChapterSpan`](#chapterspan) | struct | One chapter as a player seeks between them: where it starts and ends on the edition's timeline (`ChapterList::spans`). |
| [`EditionChapters`](#editionchapters) | struct | The chapters an edition resolved to. |
| [`ChapterEditBody`](#chaptereditbody) | struct | `PUT /api/library/editions/{id}/chapters` body: the edition's chapters as an Administrator sets them, in pre-order. An empty list clears them. |
| [`ChapterEditEntry`](#chaptereditentry) | struct | One chapter of an edit, as JSON carries it. |
| [`ChapterPinBody`](#chapterpinbody) | struct | `PUT /api/library/editions/{id}/chapters/pin` body: the claim to pin. |
| [`ChapterPinRequest`](#chapterpinrequest) | struct | A checked pin request. |
| [`TitleQuery`](#titlequery) | struct | A title someone asks about (a wish, a catalog card): what the resolver needs to say which work it belongs to. |
| [`WorkEnrichment`](#workenrichment) | struct | Enrichment fields for `LibraryStore::set_work_enrichment`. |
| [`ListeningIdentifiers`](#listeningidentifiers) | struct | The identifiers a listening item names its book by, as the provider reported them (any case and punctuation; normalised here). |
| [`ListeningMatch`](#listeningmatch) | struct | What one listening item resolved to in the library. |
| [`ListeningProgressWrite`](#listeningprogresswrite) | struct | One stored listening row a batched write produced. |
| [`ConflictCursor`](#conflictcursor) | struct | Where a conflicts page stopped: the last row's key, in key order. |
| [`ResolutionConflict`](#resolutionconflict) | struct | One conflict the resolver recorded for an Administrator. |
| [`AccountPaceState`](#accountpacestate) | struct | One account's pacing state as stored. |
| [`DownloadId`](#downloadid) | struct | One store download's ledger identity. |
| [`Bookmark`](#bookmark) | struct | A place one person marked in one edition. |
| [`NewPlaybackSession`](#newplaybacksession) | struct | A play session to open (`LibraryStore::open_playback_session`). |
| [`PlaybackChange`](#playbackchange) | struct | The result of a sync or of setting progress outright. |
| [`PlaybackEdition`](#playbackedition) | struct | What the player needs to know of an edition before it plays (`LibraryStore::playback_edition`). |
| [`PlaybackMs`](#playbackms) | struct | A length of time on a recording's timeline, in milliseconds: a position, a duration or time listened. At most `MAX_PLAYBACK_MS`. |
| [`PlaybackProgress`](#playbackprogress) | struct | How far one person has listened to one edition. |
| [`PlaybackProgressEdit`](#playbackprogressedit) | struct | Progress to set outright (`LibraryStore::set_playback_progress`): a seek from a page with no session, Mark finished, or Start over. |
| [`PlaybackSession`](#playbacksession) | struct | One stretch of listening on one player. |
| [`PlaybackSync`](#playbacksync) | struct | What a player reports about a playing session (`LibraryStore::sync_playback_session`). |
| [`ProgressCursor`](#progresscursor) | struct | A page position of `LibraryStore::list_playback_progress`: the last row of the page before. |
| [`BookmarkEditBody`](#bookmarkeditbody) | struct | `PUT /api/playback/bookmarks/{id}` body: what to change. A field left out keeps its stored value. |
| [`BookmarkEditRequest`](#bookmarkeditrequest) | struct | A checked change to a bookmark. |
| [`NewBookmarkBody`](#newbookmarkbody) | struct | `POST /api/library/editions/{id}/bookmarks` body. |
| [`NewBookmarkRequest`](#newbookmarkrequest) | struct | A checked bookmark to add. |
| [`OpenSessionBody`](#opensessionbody) | struct | `POST /api/playback/sessions` body. |
| [`OpenSessionRequest`](#opensessionrequest) | struct | A checked request to open a play session. |
| [`PlaybackSyncBody`](#playbacksyncbody) | struct | `POST /api/playback/sessions/{id}/sync` and `.../close` body: where the listener is. |
| [`ProgressEditBody`](#progresseditbody) | struct | `PUT /api/playback/progress/{edition_id}` body: what to set. A field left out keeps its stored value. |
| [`TrackPath`](#trackpath) | struct | The file a stream request names (`parse_track_path`). |
| [`NewPlayerCard`](#newplayercard) | struct | A new player card (`LibraryStore::mint_player_card`). |
| [`PlayerCard`](#playercard) | struct | A live player card, as an access token resolves to it. |
| [`PlayerCardTokens`](#playercardtokens) | struct | What signing a card in or refreshing it hands out, as digests: the caller holds the tokens themselves. |
| [`PlayerPassword`](#playerpassword) | struct | A player password as its owner sees it in Account settings. |
| [`PlayerFacet`](#playerfacet) | struct | One value of the page index on an item: its key and how it is written. |
| [`PlayerItem`](#playeritem) | struct | What the player API shows of one item (`LibraryStore::player_items`). |
| [`PluginDatabaseRecord`](#plugindatabaserecord) | struct | One provisioned plugin database binding registry row. |
| [`KvEntry`](#kventry) | struct | One live key/value row. |
| [`KvListPage`](#kvlistpage) | struct | One page of `LibraryStore::kv_list`. |
| [`KvWriteSpec`](#kvwritespec) | struct | A write plus the namespace limits it must satisfy. |
| [`PassStart`](#passstart) | struct | How the next pass of one account starts. |
| [`QualityCheckState`](#qualitycheckstate) | struct | Where the quality-upgrade check of one account stands. |
| [`ScanState`](#scanstate) | struct | Where one account's scan stands. |
| [`UserStateEdit`](#userstateedit) | struct | A change to one user's state for one work, applied by `LibraryStore::edit_user_title_state`. |
| [`WorkSplitBody`](#worksplitbody) | struct | `POST /api/library/works/{id}/split` body: the expressions to move. |
| [`WorkSplitRequest`](#worksplitrequest) | struct | A validated `WorkSplitBody`: 1 to `MAX_SPLIT_EXPRESSIONS` distinct expression ids, each 1 to `MAX_SPLIT_EXPRESSION_ID_BYTES` bytes, sorted. |
| [`AcquireTitle`](#acquiretitle) | struct | One title an account holds, as acquire reads it from the library model. |
| [`AcquireTitlePage`](#acquiretitlepage) | struct | One page of `LibraryStore::acquire_titles_page`. |
| [`AcquiredTitleKey`](#acquiredtitlekey) | struct | One account's title of an edition acquired with a stored file, without its metadata. |
| [`AcquiredTitleKeyPage`](#acquiredtitlekeypage) | struct | One page of `AcquiredTitleKey`s. |
| [`AudioStream`](#audiostream) | struct | What a probe found about the audio stream of one file, besides its `FileProbe`: the encoding, channels, sample rate and the running time a player sees. |
| [`CandidateRef`](#candidateref) | struct | One title a store returned for a subject, as written: the shared listing it is, plus why the subject got it. |
| [`ChangedObject`](#changedobject) | struct | One object whose row changed after a reader's last sequence. |
| [`DestinationFeed`](#destinationfeed) | struct | How one destination's outside changes are followed. |
| [`EditionAcquisition`](#editionacquisition) | struct | An edition's acquire state (`edition_acquisitions`). |
| [`EditionAsin`](#editionasin) | struct | An ASIN a catalog match found for an edition. |
| [`EditionCopy`](#editioncopy) | struct | One edition's copy at one destination. |
| [`EditionEnrichment`](#editionenrichment) | struct | What an enricher fills on one edition; `None` fields are left alone. |
| [`EditionFile`](#editionfile) | struct | Who holds a recorded file key, and what the file is. |
| [`EnrichCandidate`](#enrichcandidate) | struct | One held edition, with what an enricher needs to look it up and the fields it may fill. |
| [`EnrichCandidatePage`](#enrichcandidatepage) | struct | One page of `LibraryStore::enrich_candidates_page`. |
| [`FileProbe`](#fileprobe) | struct | What a probe found in one stored audio file, and where the copy came from (`edition_files`). |
| [`HeldIdentifiers`](#heldidentifiers) | struct | Which of a set of store identifiers belong to titles the library holds. |
| [`LibraryStore`](#librarystore) | struct | Handle to the Bookclerk library database. |
| [`LostCopies`](#lostcopies) | struct | What `LibraryStore::lose_copies` did with the stored copies whose file is gone. |
| [`NewBook`](#newbook) | struct | One title as a library sync (or an import) reports it: the account that holds it, the store listing it describes and the store's text. |
| [`NewListeningProgress`](#newlisteningprogress) | struct | Input for upserting listening progress. |
| [`NewTitleRequest`](#newtitlerequest) | struct | Input for creating a title request / wishlist row. |
| [`NewTitleRequestSource`](#newtitlerequestsource) | struct | Input for upserting a per-storefront wishlist snapshot. |
| [`ProbedAudioFile`](#probedaudiofile) | struct | One audio file of an edition as `LibraryStore::edition_audio_files` reads it back. |
| [`QueueRank`](#queuerank) | struct | The taste ranking of one queue entry, as the ranking job computed it. |
| [`SavedFilterRecord`](#savedfilterrecord) | struct | Saved Lucene-style quick filter. |
| [`ScannedTitle`](#scannedtitle) | struct | One title a storefront scan reported. |
| [`SeenObject`](#seenobject) | struct | One object as a listing or a change feed reports it. |
| [`StoreCallHit`](#storecallhit) | struct | One listing a cached storefront call answered. |
| [`StoredCandidate`](#storedcandidate) | struct | One stored storefront candidate of a subject: the shared listing joined with what is personal to the subject. |
| [`StoredDiscoverTaste`](#storeddiscovertaste) | struct | One subject's stored Discover taste. |
| [`StoredFeed`](#storedfeed) | struct | One subject's stored, ranked Discover feed. |
| [`StoredQueueRank`](#storedqueuerank) | struct | One stored queue ranking row. |
| [`TitleHolding`](#titleholding) | struct | The edition a library title sells and the account holding it is held by: the key its acquire writes are recorded under. |
| [`UserBookFields`](#userbookfields) | struct | Partial update for user-defined book fields. |
| [`WishWork`](#wishwork) | struct | The work a wish is for, read from a store listing in the shared pool. |
| [`NewVector`](#newvector) | struct | One vector to store. |
| [`SpaceCoverage`](#spacecoverage) | struct | How many work vectors one space holds. |
| [`VectorSpace`](#vectorspace) | struct | What made a set of vectors. Vectors of two different spaces are never compared. |
| [`NotifyCursor`](#notifycursor) | struct | Where a page of the delivery log or an inbox starts: the last row of the previous page (newest first). |
| [`NotifyPage`](#notifypage) | struct | One page of a newest-first listing. |
| [`NotifyQueueLimits`](#notifyqueuelimits) | struct | Queue limits a `notify_send` admission uses (the daemon's `[jobs]`). |
| [`DbAtomicParams`](#dbatomicparams) | enum | Named atomic library operation compiled by the host into a generic plan. |
| [`CanonicalObject`](#canonicalobject) | enum | One immutable canonical object (hashed before compression). |
| [`BackupReason`](#backupreason) | enum | Why a backup was written. |
| [`BackupResolve`](#backupresolve) | enum | How `resolve_backup_spec` located a recovery point. |
| [`CanonicalRestoreKind`](#canonicalrestorekind) | enum | How restore applies DDL companions. |
| [`DatabaseUnitKind`](#databaseunitkind) | enum | Logical kind of one database unit in a recovery point. |
| [`EntityKind`](#entitykind) | enum | The kind of thing a `LibraryError::NotFound` could not find. |
| [`LibraryError`](#libraryerror) | enum | Failures from library store, schema apply, secrets, or config. |
| [`AcquireStatus`](#acquirestatus) | enum | Acquire / download state for a title. |
| [`EnqueueOutcome`](#enqueueoutcome) | enum | Result of admitting a job into the durable queue. |
| [`EventStreamStart`](#eventstreamstart) | enum | Where one event stream read starts. |
| [`JobKind`](#jobkind) | enum | Kind of durable daemon work stored in the `jobs` table. |
| [`JobResourceClass`](#jobresourceclass) | enum | Concurrency class that bounds how many jobs of this type run at once. |
| [`JobState`](#jobstate) | enum | Lifecycle state of a durable job row. |
| [`JobTrigger`](#jobtrigger) | enum | Who admitted the job (API vs periodic scheduler). |
| [`PortalSessionKind`](#portalsessionkind) | enum | Where a user's session token is presented. |
| [`PublishDomainEventOutcome`](#publishdomaineventoutcome) | enum | Result of `crate::LibraryStore::publish_domain_event`. |
| [`RequestStatus`](#requeststatus) | enum | Wishlist row status (`open` while wishlisted; `cancelled` after un-wishlist). |
| [`UserRole`](#userrole) | enum | First-party user role (`owner` \| `administrator` \| `member`). |
| [`UserStatus`](#userstatus) | enum | First-party user status. |
| [`SchemaState`](#schemastate) | enum | Durable Bookclerk library schema state. |
| [`FacetValuesOf`](#facetvaluesof) | enum | Whose values a facet value list reads. |
| [`WorkFacet`](#workfacet) | enum | A filter value list of the page index (`library_work_facets`). |
| [`WorkFilter`](#workfilter) | enum | One filter on a page of library works. |
| [`WorkSort`](#worksort) | enum | How a page of library works is ordered. |
| [`HoldingHolder`](#holdingholder) | enum | Who holds a listing. |
| [`WorkTagSource`](#worktagsource) | enum | Where a library tag came from (`work_tags.source`). |
| [`OfferAudience`](#offeraudience) | enum | Who a `ListingOffer` price is for. |
| [`ChapterCheck`](#chaptercheck) | enum | How strictly `ChapterList::new` reads its input. |
| [`ChapterSource`](#chaptersource) | enum | Who states a chapter claim, as a writer names it. |
| [`ChapterSourceKind`](#chaptersourcekind) | enum | The kind of source a chapter claim came from (`source_kind`). |
| [`ChaptersOutcome`](#chaptersoutcome) | enum | What a chapter write changed. |
| [`ClaimQuality`](#claimquality) | enum | Where a machine source ranks within its kind (`quality`). |
| [`MarkerKind`](#markerkind) | enum | What a store says its markers are. |
| [`ProviderQuality`](#providerquality) | enum | How a provider found the edition, and how exact its offsets are. |
| [`EventPruneStep`](#eventprunestep) | enum | One delete of `LibraryStore::prune_event_deliveries`, in run order. |
| [`PinOutcome`](#pinoutcome) | enum | What `LibraryStore::pin_listing` did. |
| [`DownloadStarter`](#downloadstarter) | enum | Who started a download, for the ledger. |
| [`PlaybackEvent`](#playbackevent) | enum | A host event about someone's listening, published after the write it describes (`LibraryStore::publish_playback_event`). |
| [`PlayerKind`](#playerkind) | enum | What kind of player opened a play session. |
| [`PlayerKindParam`](#playerkindparam) | enum | The kind of first-party player that opens a session. Outside player apps open theirs through the player API. |
| [`KvPutOutcome`](#kvputoutcome) | enum | Why a write was refused, so the binding can map it onto a wire code. |
| [`RecordPruneStep`](#recordprunestep) | enum | One delete of `LibraryStore::prune_record_step`, in run order. |
| [`FieldEdit`](#fieldedit) | enum | One field of a `UserStateEdit`. |
| [`SplitOutcome`](#splitoutcome) | enum | What `LibraryStore::split_work` did. |
| [`CopyState`](#copystate) | enum | Where an edition's copy at one destination stands. |
| [`DiscoverSubject`](#discoversubject) | enum | Whose Discover shelves a stored row belongs to. |
| [`FeedState`](#feedstate) | enum | The state of a destination's change feed. |
| [`FileOwner`](#fileowner) | enum | Who a recorded file key belongs to. |
| [`FileRole`](#filerole) | enum | What a recorded file is. |
| [`HoldingSelection`](#holdingselection) | enum | Which holdings of one account a removal takes. |
| [`NodeHoldingSelection`](#nodeholdingselection) | enum | Which holdings of one branch (node) a removal takes. |
| [`SeedScope`](#seedscope) | enum | Which seeds a subject's candidates and feed were built from. |
| [`migrate_library_backend`](#migrate_library_backend) | function | Copy the whole library from `source` into `dest`. |
| [`restore_backup_unit`](#restore_backup_unit) | function | Destructively replaces one logical database with `unit`. Never merges. |
| [`library_ddl_for_schema_state`](#library_ddl_for_schema_state) | function | Library DDL that matches `state`, not necessarily this binary's latest pack. |
| [`sort_tables_by_foreign_keys`](#sort_tables_by_foreign_keys) | function | FK-safe table order (parents before children). Cycles fail closed. |
| [`verify_recovery_point`](#verify_recovery_point) | function | Parses the manifest, rejects unsupported formats, and verifies every referenced object **before** the caller starts a destructive restore. |
| [`admit_canonical_schema`](#admit_canonical_schema) | function | Parses and fully admits a diagnostic/imported canonical SQL script. |
| [`admit_canonical_statements`](#admit_canonical_statements) | function | Admits already-separated canonical `CREATE TABLE` / `CREATE INDEX` statements. |
| [`apply_admitted_sql`](#apply_admitted_sql) | function | Applies admitted canonical DDL plus binding companions (tests / bootstrap). |
| [`archive_backup`](#archive_backup) | function | Packages one recovery point (manifest + referenced objects) as `.tar.gz`. |
| [`backup_library`](#backup_library) | function | Writes a crash-safe recovery point under `files_dir/backups/`. |
| [`extract_backup_archive`](#extract_backup_archive) | function | Extracts a `.tar.gz` recovery-point archive into `dest` (a backups root). |
| [`filter_library_pack_ddl`](#filter_library_pack_ddl) | function | Library migration packs may include seed `INSERT`s. Those are table data (captured from rows), not schema objects. This keeps admitted `CREATE TABLE` / `CREATE INDEX` and skips seed DML. Any other statement fails closed. |
| [`library_canonical_schema`](#library_canonical_schema) | function | Admitted library schema for this binary's current pack (frozen ups + unreleased). |
| [`library_canonical_schema_for_state`](#library_canonical_schema_for_state) | function | Admitted library schema for `state`. |
| [`list_backups`](#list_backups) | function | Lists published recovery points oldest-first. |
| [`plugin_canonical_schema_from_ddl_catalog`](#plugin_canonical_schema_from_ddl_catalog) | function | Rebuilds plugin schema from durable `bookclerk_sql_ddl` on an open connection. |
| [`prune_automatic_backups`](#prune_automatic_backups) | function | Deletes oldest automatic pre-migrate recovery points. Never deletes `manual`. |
| [`resolve_backup_spec`](#resolve_backup_spec) | function | Resolves a recovery-point id, timestamp prefix, or archive path. |
| [`restore_backup`](#restore_backup) | function | Restores the library unit after verifying **all** objects and units. |
| [`restore_backup_in_repo`](#restore_backup_in_repo) | function | Restores from an already-open repository (extracted archives). |
| [`apply_binding_bootstrap`](#apply_binding_bootstrap) | function | Applies host-owned binding bootstrap when the binding is uninitialized. |
| [`binding_bootstrap_plan`](#binding_bootstrap_plan) | function | Statements to apply for `state`, or `None` when the binding already matches. |
| [`db_atomic_operation_id`](#db_atomic_operation_id) | function | Caller-owned idempotency key for an atomic attempt. |
| [`db_atomic_request_hash`](#db_atomic_request_hash) | function | SHA-256 hex digest of the idempotency-relevant fields of `op`. |
| [`execute_db_atomic`](#execute_db_atomic) | function | Runs a compiled named atomic as one native SQL transaction. |
| [`execute_named_atomic`](#execute_named_atomic) | function | Compiles `params` and runs the plan as one native SQL transaction. |
| [`apply_host_schema`](#apply_host_schema) | function | Applies pending host-authored DDL from the canonical migration plan. |
| [`apply_host_schema_with_batch`](#apply_host_schema_with_batch) | function | Applies schema using `run_batch` (typed `executeAtomic`) for each version. |
| [`apply_host_schema_with_batch_opts`](#apply_host_schema_with_batch_opts) | function | Applies schema with backups using `run_batch` for each version. |
| [`apply_host_schema_with_options`](#apply_host_schema_with_options) | function | Applies pending host schema with optional in-place backups. |
| [`current_schema_state`](#current_schema_state) | function | Reads explicit `SchemaState` in the Bookclerk host/bootstrap namespace. |
| [`current_schema_state_in`](#current_schema_state_in) | function | Reads explicit `SchemaState` for one ledger `namespace`. |
| [`current_schema_version`](#current_schema_version) | function | Frozen revision when the database is `SchemaState::Frozen`; otherwise `0`. |
| [`ensure_restore_target_is_replaceable`](#ensure_restore_target_is_replaceable) | function | Fails closed when this binary cannot interpret the target database's schema history well enough to drop every unknown table before restore. |
| [`migrate_host_schema_to`](#migrate_host_schema_to) | function | Migrates toward `target`, including reversible downs for CLI rollback. |
| [`migrate_host_schema_to_with_batch`](#migrate_host_schema_to_with_batch) | function | Migrates toward `target` using `run_batch` (typed `executeAtomic`) per version. |
| [`require_schema_migrations`](#require_schema_migrations) | function | Requires the `schemaMigrations` versioning contract from typed `bookclerk_plugin_abi::DbCapabilities`. |
| [`schema_state_compatible`](#schema_state_compatible) | function | True when `state` is exactly the schema this binary applies: a start against it neither migrates nor fails closed. |
| [`apply_plugin_migrations`](#apply_plugin_migrations) | function | Applies the pending registered suffix. Idempotent when history already matches. |
| [`history_from_execute_reply`](#history_from_execute_reply) | function | Parses journal rows from a typed select reply. |
| [`load_plugin_migration_history`](#load_plugin_migration_history) | function | Reads the host-private plugin migration journal. |
| [`next_pending_plugin_migration`](#next_pending_plugin_migration) | function | Next registered migration after a verified durable prefix, if any. |
| [`plugin_apply_statements`](#plugin_apply_statements) | function | One atomic apply unit: serialization slot, registered ops, host journal append. |
| [`plugin_binding_type_env`](#plugin_binding_type_env) | function | Binding catalog after host bootstrap plus schema ops from the first `applied` registered migrations. |
| [`plugin_history_session_matches`](#plugin_history_session_matches) | function | True when a binding session's captured history digest still matches durable history. |
| [`plugin_journal_has_entry`](#plugin_journal_has_entry) | function | True when `history` already contains this `(ordinal, id, checksum)` row. |
| [`plugin_journal_select_request`](#plugin_journal_select_request) | function | Typed SELECT used by RPC hosts to read the plugin migration journal. |
| [`prove_plugin_migration_sequence`](#prove_plugin_migration_sequence) | function | Proves a complete registration with one evolving `SqlTypeEnv`. |
| [`require_history_prefix`](#require_history_prefix) | function | Fail closed unless durable history is an exact prefix of `registered`. |
| [`sql_string_literal`](#sql_string_literal) | function | SQL string literal with `'` doubled. |
| [`branch_request_dedup_key`](#branch_request_dedup_key) | function | Dedup key of a `JobKind::BranchRequest`: one active request per `(branch node, work)`. |
| [`catalog_subscribers_for_event`](#catalog_subscribers_for_event) | function | Enabled catalog rows that match `event` type, schema version, and filter. |
| [`collapse_live_subscriber_nodes`](#collapse_live_subscriber_nodes) | function | Collapse live per-node rows into one catalog record per `plugin_id`. |
| [`content_kind_from_classic`](#content_kind_from_classic) | function | Map classic EF `ContentType` to bookclerk `content_kind`. |
| [`content_kind_to_classic`](#content_kind_to_classic) | function | Classic EF `ContentType` enum value. |
| [`event_filter_matches`](#event_filter_matches) | function | True when `filter` is absent/empty or every filter key equals the payload object value. |
| [`event_matches_wake_grants`](#event_matches_wake_grants) | function | True when an event matches persisted wake grants, or legacy type+filter rows. |
| [`intersect_event_filters`](#intersect_event_filters) | function | Merge declared and requested payload filters; `None` on a same-key conflict. |
| [`is_downloadable`](#is_downloadable) | function | True when the title can be downloaded (classic `WithoutParents`). |
| [`is_valid_queue_name`](#is_valid_queue_name) | function | True when `name` is `a-z0-9*` and at most `MAX_QUEUE_NAME_LEN` chars. |
| [`job_backoff_run_after`](#job_backoff_run_after) | function | Next `run_after` after a failed attempt, exponential from a 5s base (capped). |
| [`normalize_theme`](#normalize_theme) | function | Canonical appearance preference; unknown values become `system`. |
| [`plugin_scheduled_dedup_key`](#plugin_scheduled_dedup_key) | function | Dedup key of a `JobKind::PluginScheduled` fire: one active fire per `(instance, cron)`. |
| [`portal_prefs_key`](#portal_prefs_key) | function | Subject key for a portal identity (legacy; prefer `user_prefs_key`). |
| [`push_queue_wisher`](#push_queue_wisher) | function | Adds `wisher` when they are not already present (capped at `MAX_QUEUE_WISHERS`). |
| [`queue_name_or_bus`](#queue_name_or_bus) | function | Resolves an optional queue name: empty means `CORE_EVENTS_QUEUE`. |
| [`subscription_matches_event`](#subscription_matches_event) | function | True when one catalog subscription matches `event` type, schema, class, and filter. |
| [`user_prefs_key`](#user_prefs_key) | function | Subject key for a first-party user. |
| [`wake_grants_from_subscriptions`](#wake_grants_from_subscriptions) | function | Build wake grants from declared subscriptions for `wake_type`. |
| [`plan_schema_walk`](#plan_schema_walk) | function | Plans ups or downs from `from` toward `to` using `plan`. |
| [`plan_schema_walk_from_state`](#plan_schema_walk_from_state) | function | Plans a frozen walk from an explicit `SchemaState`. |
| [`classify_session_client`](#classify_session_client) | function | Classify a User-Agent (or missing UA for API/bearer clients). |
| [`compile_claim_event_delivery`](#compile_claim_event_delivery) | function | Compiles a host-side CAS claim for one pending `event_deliveries` row. |
| [`compile_named_request`](#compile_named_request) | function | Compiles a named atomic request into canonical Bookclerk SQL. |
| [`interpret_typed_exec`](#interpret_typed_exec) | function | Maps a typed `ExecuteReply` onto a host `DbAtomicResult`. |
| [`validate_execute_reply`](#validate_execute_reply) | function | Rejects a guest `ExecuteReply` that does not echo the sent request or exceeds negotiated caps. |
| [`validate_streamed_execute_reply`](#validate_streamed_execute_reply) | function | `validate_execute_reply` for a reply reassembled from `executeStreamed` frames. |
| [`facet_key`](#facet_key) | function | The key a facet value or tag is stored and matched under: trimmed and lower-cased, so "Fantasy" and "fantasy " are one value. |
| [`sort_title`](#sort_title) | function | The sort key of a title: lower case, with one leading English article dropped, so "The Hobbit" sorts under h. |
| [`normalize_work_tags`](#normalize_work_tags) | function | Trims, drops empties and duplicates (by `facet_key`, first spelling kept) and checks the bounds. |
| [`parse_work_tags_request`](#parse_work_tags_request) | function | Parses and validates a `PUT /api/library/works/{id}/tags` body from its raw bytes, sans IO, the way the route reads it: JSON into `WorkTagsBody`, then `WorkTagsBody::validate`. |
| [`apply_user_states`](#apply_user_states) | function | Applies the state of `users` to `titles`, merged per work. |
| [`copy_source_id`](#copy_source_id) | function | The source id a `ChapterSource::File` claim names one stored copy by: `copy-` and the first 32 hex digits of the SHA-256 of the destination and the copy's audio keys in sorted order, so the id has a fixed length whatever the keys are and does not depend on their play order. |
| [`resolve_chapters`](#resolve_chapters) | function | Picks the claim an edition's chapters resolve to. |
| [`parse_chapter_edit_request`](#parse_chapter_edit_request) | function | Parses and checks a chapter edit body from its raw bytes, sans IO, the way the route reads it. |
| [`parse_chapter_pin_request`](#parse_chapter_pin_request) | function | Parses and checks a pin body from its raw bytes, sans IO. |
| [`copy_retry_delay`](#copy_retry_delay) | function | The delay before the next background attempt of a copy that has now failed `attempts` times (1 for the first failure): `COPY_RETRY_BASE_DELAY` doubled per earlier failure, capped at `COPY_RETRY_MAX_DELAY`. |
| [`deliver_after_for`](#deliver_after_for) | function | RFC 3339 earliest delivery time for `spec`; empty when it may deliver now. |
| [`prepare_publish_domain_event`](#prepare_publish_domain_event) | function | Mint an event id when empty and reject oversized or blank publish fields. |
| [`fallback_work_key`](#fallback_work_key) | function | Local fallback when callers do not supply a discover `work_map_key`. |
| [`inject_dispatch_page_failures`](#inject_dispatch_page_failures) | function | Fail after the next `n` successful dispatch pages (tests only). |
| [`call_number`](#call_number) | function | The `<call number>` of edition `edition_id`: its first 40 bits as 8 lower-case Crockford base32 characters (no `i`, `l`, `o` or `u`). An id that is not a UUID falls back to its first 8 ASCII letters and digits. Returns `None` when that leaves nothing. |
| [`edition_id_for_listing`](#edition_id_for_listing) | function | The id of an edition first seen through the store listing `key`: a version 5 UUID over its plugin key, marketplace and product id. Two nodes (or one library rebuilt from a fresh scan) that resolve the same listing first give its edition the same id, and so the same `<call number>` in file names. |
| [`holding_from_book`](#holding_from_book) | function | The holding a flat library row describes. |
| [`listing_from_book`](#listing_from_book) | function | The structured listing a flat library row describes, for paths that only have a `NewBook` (imports, seeds, tests). |
| [`reaches_end`](#reaches_end) | function | True when `position` in a recording `duration` long counts as having reached its end: within the last one percent, or within the last thirty seconds of a recording longer than a minute. |
| [`parse_bookmark_edit_request`](#parse_bookmark_edit_request) | function | Parses and validates a `PUT /api/playback/bookmarks/{id}` body from its raw bytes, sans IO. |
| [`parse_new_bookmark_request`](#parse_new_bookmark_request) | function | Parses and validates a `POST /api/library/editions/{id}/bookmarks` body from its raw bytes, sans IO. |
| [`parse_open_session_request`](#parse_open_session_request) | function | Parses and validates a `POST /api/playback/sessions` body from its raw bytes, sans IO: JSON into `OpenSessionBody`, then `OpenSessionBody::validate`. |
| [`parse_playback_sync_request`](#parse_playback_sync_request) | function | Parses and validates a sync or close body from its raw bytes, sans IO, the way the routes read it: JSON into `PlaybackSyncBody`, then `PlaybackSyncBody::validate`. |
| [`parse_progress_cursor`](#parse_progress_cursor) | function | Parses the `cursor` of `GET /api/playback/progress`, sans IO. |
| [`parse_progress_edit_request`](#parse_progress_edit_request) | function | Parses and validates a `PUT /api/playback/progress/{edition_id}` body from its raw bytes, sans IO. |
| [`parse_track_path`](#parse_track_path) | function | Parses the two path parts of `GET /api/playback/sessions/{id}/tracks/{n}`, sans IO. |
| [`progress_cursor_token`](#progress_cursor_token) | function | The opaque token of a progress cursor: unpadded URL-safe Base64 of the row's change time, a `\|`, and its edition id. |
| [`plan_pass`](#plan_pass) | function | Decides how the next pass of an account starts. |
| [`set_dispatch_chunk_for_test`](#set_dispatch_chunk_for_test) | function | Override subscribers-per-plan for dispatch paging tests. |
| [`parse_work_split_request`](#parse_work_split_request) | function | Parses and validates a `POST /api/library/works/{id}/split` body from its raw bytes, sans IO, the way the route reads it: JSON into `WorkSplitBody`, then `WorkSplitBody::validate`. |
| [`derive_claim_password_fingerprint`](#derive_claim_password_fingerprint) | function | Stable, non-secret fingerprint of an invite/reset password for idempotency. |
| [`derive_claim_session_token`](#derive_claim_session_token) | function | Derive the portal session token for a claim ticket plus browser nonce. |
| [`hash_token`](#hash_token) | function | SHA-256 hex digest of a raw ticket/session token for durable storage. |
| [`parse_claim_redeem_nonce`](#parse_claim_redeem_nonce) | function | Accept a browser-generated claim-redeem nonce. |
| [`apply_merged_sources`](#apply_merged_sources) | function | Apply merged bib / editions / hints onto a wishlist row from its source snapshots. |
| [`pick_better_description`](#pick_better_description) | function | Prefer HTML blurbs over plain text; otherwise the longer string wins. |
| [`Result`](#result) | type alias | Result alias for `LibraryError`. |
| [`InProcessSqliteAtomic`](#inprocesssqliteatomic) | type alias | The `crate::AtomicTxnBackend` over a local SQLite connection. |
| [`LEFT_BEHIND_TABLES`](#left_behind_tables) | constant | Tables the copy leaves behind on purpose, each with the reason. |
| [`CHUNK_TARGET_UNCOMPRESSED_BYTES`](#chunk_target_uncompressed_bytes) | constant | Target uncompressed JSON size for one table-data chunk. |
| [`BACKUPS_DIR`](#backups_dir) | constant | Subdirectory of `$BOOKCLERK_FILES_DIR` holding the backup repository. |
| [`BACKUP_FORMAT_VERSION`](#backup_format_version) | constant | Manifest format written by this binary. |
| [`BACKUP_RETENTION`](#backup_retention) | constant | Automatic `pre-migrate` recovery points retained after prune. |
| [`LIBRARY_SKIP_TABLES`](#library_skip_tables) | constant | Library registry rows are environment-local (`unit_ref` is not portable). |
| [`MIGRATION_LEASE_TTL`](#migration_lease_ttl) | constant | How long a schema migration lease lasts without renewal. The migrator renews it every third of this while it works. |
| [`MIGRATION_LEASE_WAIT`](#migration_lease_wait) | constant | How long a starter waits for another starter's migration before it fails closed. Longer than `MIGRATION_LEASE_TTL`, so a migrator that died is replaced by a waiter rather than failing every waiter. |
| [`BOOKCLERK_SCHEMA_NAMESPACE`](#bookclerk_schema_namespace) | constant | Ledger namespace for host library schema and Bookclerk-owned binding bootstrap. |
| [`MAX_PLUGIN_MIGRATION_APPLY_ATTEMPTS`](#max_plugin_migration_apply_attempts) | constant | Bounded retries for **one** expected plugin migration apply unit. |
| [`CORE_EVENTS_QUEUE`](#core_events_queue) | constant | Queue that carries the domain-event bus (`[events]` producers and consumers). |
| [`EDITION_ACQUIRE_CHANGED`](#edition_acquire_changed) | constant | Host event type published with an edition's acquire state change other than becoming acquired (payload `editionId`, `titleId`, `status`). |
| [`EVENT_DELIVERY_DEFAULT_RETENTION_SECS`](#event_delivery_default_retention_secs) | constant | How long a delivery may wait, due but never leased, before the host expires it when its consumer declares no `message_retention_period`: one day. A consumer that is deployed but not running stops accumulating work after this, and catches up through the event stream instead. |
| [`EVENT_DELIVERY_MAX_ATTEMPTS`](#event_delivery_max_attempts) | constant | Default max delivery attempts before dead-letter. |
| [`EVENT_RESOURCE_CLASS_NETWORK`](#event_resource_class_network) | constant | Default delivery resource class (`network`). |
| [`EVENT_SUBSCRIBER_HEARTBEAT_TTL_SECS`](#event_subscriber_heartbeat_ttl_secs) | constant | Live catalog rows quieter than this many seconds are ignored at dispatch. |
| [`HOST_CONSUMER_PREFIX`](#host_consumer_prefix) | constant | Prefix of subscriber ids the daemon itself consumes under (`host:discover`, `host:search-index:<node>`). Their deliveries never expire. |
| [`INGEST_BATCH_FINISHED`](#ingest_batch_finished) | constant | Host event type published when an ingest batch (an upload or a storage intake) reaches a final state (payload `batchId`, `state`, `source`, `intakeInstance`, `detail`). |
| [`JOB_PAYLOAD_VERSION`](#job_payload_version) | constant | Current job command envelope version stored in `JobPayload::v`. |
| [`LISTENING_SYNCED`](#listening_synced) | constant | Host event published when a listening sync changed what Discover reads from a stored row: its owner, matched book or work, authors, or how far they got (started, quarter, finished). |
| [`MAX_QUEUE_NAME_LEN`](#max_queue_name_len) | constant | Longest queue name. |
| [`MAX_QUEUE_WISHERS`](#max_queue_wishers) | constant | Maximum distinct people serialized on a global-queue or Discover card. |
| [`NODE_QUEUE_PREFIX`](#node_queue_prefix) | constant | Prefix of the per-node queues (`node.<node_id>`) node subsystems consume. |
| [`OPERATOR_PREFS_KEY`](#operator_prefs_key) | constant | Subject key for the shared operator account. |
| [`QUEUE_CONTENT_TYPE_BYTES`](#queue_content_type_bytes) | constant | Queue message body is raw bytes stored as base64 text. |
| [`QUEUE_CONTENT_TYPE_JSON`](#queue_content_type_json) | constant | Queue message body is UTF-8 JSON text. |
| [`QUEUE_CONTENT_TYPE_TEXT`](#queue_content_type_text) | constant | Queue message body is plain UTF-8 text. |
| [`WISHLIST_CHANGED`](#wishlist_changed) | constant | Host event published when a wishlist entry is added or changes status, or an open entry is deleted with its user. |
| [`SCHEMA_STATE_FROZEN`](#schema_state_frozen) | constant | On-disk `bookclerk_schema_migrations.state` for a frozen plan version. |
| [`SCHEMA_STATE_UNRELEASED`](#schema_state_unreleased) | constant | On-disk `bookclerk_schema_migrations.state` for an unreleased development pack. |
| [`EVENT_STREAM_SCAN_FACTOR`](#event_stream_scan_factor) | constant | Rows one `LibraryStore::read_event_stream` call may examine per event it may return. |
| [`EVENT_STREAM_SETTLE_SECS`](#event_stream_settle_secs) | constant | Seconds an outbox row must age before `LibraryStore::read_event_stream` returns it. `created_at` is stamped before the insert commits, so a publication can commit after a newer one; a read that stops short of the last few seconds never moves its cursor past a row that has yet to appear. Newer events still reach subscribers through delivery. |
| [`LIBRARY_INDEX_LAYOUT`](#library_index_layout) | constant | Layout of `library_work_rows` this binary writes. A library with no finished build of this layout is rebuilt; bump it when a change to how rows are derived needs every row rewritten. |
| [`MAX_WORK_TAGS`](#max_work_tags) | constant | Most tags an Administrator may set on one work. With the delete this fits one batch of the smallest statement count a database plugin may advertise (40). |
| [`MAX_WORK_TAG_BYTES`](#max_work_tag_bytes) | constant | Longest tag, in UTF-8 bytes after trimming: the longest value the page index keeps under a facet, and the longest value a scope rule matches. |
| [`CATALOG_LISTING_REFRESH`](#catalog_listing_refresh) | constant | How long a catalog listing counts as current: a store answer for a listing seen more recently than this refreshes only its prices, not its resolution or text, so overlapping tastes cost one resolve a day. |
| [`COPY_RUNTIME_TOLERANCE_MS`](#copy_runtime_tolerance_ms) | constant | Most a stored copy's running time may differ from the resolved list's before the copy plays the chapters read from its own files, in milliseconds (`docs/adr/edition-chapters.md`, decision 1). |
| [`MAX_CHAPTERS`](#max_chapters) | constant | Most chapters one list holds, the same cap as a metadata file's (`maxMetadataFileChapters` in `plugin.capnp`). |
| [`MAX_CHAPTER_DEPTH`](#max_chapter_depth) | constant | Deepest nesting stored; 0 is a top-level chapter. A machine source's deeper chapter is lifted to this depth; an edit's is refused. |
| [`MAX_CHAPTER_START_MS`](#max_chapter_start_ms) | constant | Latest chapter start stored, in milliseconds: sixty days, the playback core's longest position (`crate::MAX_PLAYBACK_MS`). |
| [`MAX_CHAPTER_TITLE_BYTES`](#max_chapter_title_bytes) | constant | Longest chapter title stored, in UTF-8 bytes. A machine source's longer title is cut at a character boundary; an edit's is refused. |
| [`MAX_CHAPTER_EDIT_BYTES`](#max_chapter_edit_bytes) | constant | Largest chapter edit body the route reads, in bytes: room for `MAX_CHAPTERS` chapters with titles at `MAX_CHAPTER_TITLE_BYTES`, some of them escaped. |
| [`MAX_CHAPTER_PIN_BYTES`](#max_chapter_pin_bytes) | constant | Largest pin body the route reads, in bytes. |
| [`MAX_CHAPTER_SOURCE_ID_BYTES`](#max_chapter_source_id_bytes) | constant | Longest `source_id` a pin names, in bytes; a PluginKey is far shorter. |
| [`MAX_CONFLICT_PAGE`](#max_conflict_page) | constant | Largest conflicts page `LibraryStore::list_resolution_conflicts` reads. |
| [`MAX_BOOKMARKS_PER_EDITION`](#max_bookmarks_per_edition) | constant | Most bookmarks one person keeps in one edition. |
| [`MAX_BOOKMARK_TITLE_BYTES`](#max_bookmark_title_bytes) | constant | Longest bookmark title, in bytes (well inside the minimum database profile's cell size). |
| [`MAX_PLAYBACK_MS`](#max_playback_ms) | constant | The longest position or duration the player stores: sixty days, far past any recording, and small enough that sums of two stay exact in every database profile's integer. |
| [`MAX_PLAYER_NAME_BYTES`](#max_player_name_bytes) | constant | Longest player name a session stores, in bytes. |
| [`MAX_EDITION_ID_BYTES`](#max_edition_id_bytes) | constant | Longest edition id a session is opened for, in bytes; a real one is a 36-byte UUID. |
| [`MAX_PLAYBACK_BODY_BYTES`](#max_playback_body_bytes) | constant | Largest body the other playback parsers read, in bytes: room for an escaped bookmark title or a long player name. |
| [`MAX_PLAYBACK_SYNC_BYTES`](#max_playback_sync_bytes) | constant | Largest sync body the routes read, in bytes; a real one is under 100. |
| [`MAX_PLAYBACK_TRACKS`](#max_playback_tracks) | constant | Most tracks a play session lists: a stream path's index is below it. |
| [`MAX_PROGRESS_CURSOR_BYTES`](#max_progress_cursor_bytes) | constant | Longest progress cursor token the route reads, in bytes; a real one is about 100. |
| [`MAX_PLAYER_PASSWORDS`](#max_player_passwords) | constant | Most player passwords one patron keeps. |
| [`MAX_PLAYER_PASSWORD_LABEL_BYTES`](#max_player_password_label_bytes) | constant | Longest player password label, in bytes. |
| [`PLAYER_ACCESS_TTL_SECS`](#player_access_ttl_secs) | constant | Lifetime of a player card's access token. |
| [`PLAYER_REFRESH_GRACE_SECS`](#player_refresh_grace_secs) | constant | How long the other refresh tokens of a card keep working after one of them is used: long enough for two refreshes that raced with the same token to both succeed, and for a phone whose refresh answer was lost while it slept to try again (the apps expect ten minutes). |
| [`PLAYER_REFRESH_TTL_DAYS`](#player_refresh_ttl_days) | constant | Lifetime of a player card's refresh token; each refresh starts it again. |
| [`KV_LIST_DEFAULT_LIMIT`](#kv_list_default_limit) | constant | Default page size for `LibraryStore::kv_list` when the caller passes 0. |
| [`AUDIT_RETENTION_DAYS`](#audit_retention_days) | constant | Days `security_audit_events` and `configuration_audit` rows are kept. |
| [`CONFIGURATION_CHANGE_RETENTION_DAYS`](#configuration_change_retention_days) | constant | Days `configuration_changes` notices are kept. They only signal that a document changed; the document and its audit row hold the content. |
| [`STORAGE_SCAN_STALE_DAYS`](#storage_scan_stale_days) | constant | Days without a heartbeat after which a jobless storage-scan generation (a CLI scan) is treated as abandoned. |
| [`TOKEN_HISTORY_DAYS`](#token_history_days) | constant | Days an expired claim ticket, user invite, or node enrollment link stays listed (as expired or redeemed) before it is deleted. |
| [`UPLOAD_HISTORY_DAYS`](#upload_history_days) | constant | Days a committed or aborted host upload is kept past its token's expiry. |
| [`FULL_PASS_INTERVAL`](#full_pass_interval) | constant | Longest time since the last complete pass before the host asks for a full pass again, so a store whose deltas miss edits or removals is corrected. |
| [`RESUME_CURSOR_TTL`](#resume_cursor_ttl) | constant | Longest time a stopped pass's cursor is resumed; an older one restarts the pass (store cursors expire). |
| [`MAX_SPLIT_EXPRESSIONS`](#max_split_expressions) | constant | Most expressions one split moves: the batch's widest statement binds each id twice, and the minimum database profile allows 32 binds. |
| [`MAX_SPLIT_EXPRESSION_ID_BYTES`](#max_split_expression_id_bytes) | constant | Longest expression id a split body may name, in bytes; ids are UUIDs. |
| [`ACQUIRED_KEYS_MAX_PAGE`](#acquired_keys_max_page) | constant | Upper clamp for the editions one `LibraryStore::acquired_title_keys_page` read covers. |
| [`ACQUIRE_WRITER`](#acquire_writer) | constant | The `writer` of a key an acquire run on this node wrote. |
| [`COPY_RETRY_BASE_DELAY`](#copy_retry_base_delay) | constant | Delay before the first background retry of a failed copy; each later failure doubles it, up to `COPY_RETRY_MAX_DELAY`. |
| [`COPY_RETRY_MAX_DELAY`](#copy_retry_max_delay) | constant | The longest delay between two background retries of one copy. |
| [`HOLDING_REMOVED`](#holding_removed) | constant | Event type published when a holding is removed. |
| [`MAX_COPY_ATTEMPTS`](#max_copy_attempts) | constant | Failed attempts after which a pending copy turns `CopyState::Failed` and is no longer retried in the background. |
| [`MAX_QUEUE_REASONS`](#max_queue_reasons) | constant | Most taste reasons one stored row keeps, so a row stays small whatever the ranking explains. |
| [`OBJECT_TOMBSTONE_DAYS`](#object_tombstone_days) | constant | Days a removed object's tombstone is kept for readers that page changes. |
| [`VECTOR_KIND_TEXT`](#vector_kind_text) | constant | `embeddings.target_kind` of a plain text's vector; the target id is the SHA-256 hex of the text. |
| [`VECTOR_KIND_WORK`](#vector_kind_work) | constant | `embeddings.target_kind` of a work's vector; the target id is `works.id`. |
| [`MAX_NOTIFY_PAGE`](#max_notify_page) | constant | Most rows one page of the delivery log or the inbox returns. |
| [`CLAIM_REDEEM_NONCE_HEX_LEN`](#claim_redeem_nonce_hex_len) | constant | Hex length of a browser-generated claim-redeem nonce (32 bytes). |

## Re-exports

- [`bookclerk_keystore::CREDENTIAL_NAME` as `SYSTEMD_CREDENTIAL_NAME`](../bookclerk_keystore/README.md#credential_name)
- [`bookclerk_plugin_abi::AdapterBackupOps`](../bookclerk_plugin_abi/backup_ops.md#adapterbackupops)
- [`bookclerk_plugin_abi::GuestSqlPolicy`](../bookclerk_plugin_abi/guest_sql.md#guestsqlpolicy)
- [`bookclerk_plugin_abi::SharedAdapterBackupOps`](../bookclerk_plugin_abi/backup_ops.md#sharedadapterbackupops)
- [`bookclerk_resolve::Abridgement`](../bookclerk_resolve/kinds.md#abridgement)
- [`bookclerk_resolve::Entitlement`](../bookclerk_resolve/kinds.md#entitlement)
- [`bookclerk_resolve::ExpressionKind`](../bookclerk_resolve/kinds.md#expressionkind)
- [`bookclerk_resolve::MediaKind`](../bookclerk_resolve/kinds.md#mediakind)
- [`bookclerk_resolve::WorkKind`](../bookclerk_resolve/kinds.md#workkind)
- [`bookclerk_storefront::text::decode_html_entities`](../bookclerk_storefront/text.md#decode_html_entities)
- [`bookclerk_storefront::text::decode_html_entities_cow`](../bookclerk_storefront/text.md#decode_html_entities_cow)
- [`bookclerk_storefront::text::decode_html_entities_in_place`](../bookclerk_storefront/text.md#decode_html_entities_in_place)
- [`bookclerk_storefront::text::decode_html_entities_opt`](../bookclerk_storefront/text.md#decode_html_entities_opt)
- [`bookclerk_storefront::text::decode_html_entities_opt_in_place`](../bookclerk_storefront/text.md#decode_html_entities_opt_in_place)
- [`bookclerk_storefront::text::str_maybe_html_entity`](../bookclerk_storefront/text.md#str_maybe_html_entity)
- [`email::gravatar_hash`](email.md#gravatar_hash)
- [`master_key::configure_master_key`](master_key.md#configure_master_key)
- [`master_key::configure_master_key_with`](master_key.md#configure_master_key_with)
- [`master_key::discard_minted_master_key`](master_key.md#discard_minted_master_key)
- [`master_key::encode_master_key_file`](master_key.md#encode_master_key_file)
- [`master_key::export_master_key`](master_key.md#export_master_key)
- [`master_key::import_master_key`](master_key.md#import_master_key)
- [`master_key::inspect_master_key`](master_key.md#inspect_master_key)
- [`master_key::install_master_key`](master_key.md#install_master_key)
- [`master_key::key_store_preference`](master_key.md#key_store_preference)
- [`master_key::master_key_envelope`](master_key.md#master_key_envelope)
- [`master_key::master_key_fingerprint`](master_key.md#master_key_fingerprint)
- [`master_key::master_key_is_wrapped`](master_key.md#master_key_is_wrapped)
- [`master_key::master_key_path`](master_key.md#master_key_path)
- [`master_key::move_master_key`](master_key.md#move_master_key)
- [`master_key::prepare_master_key_dir`](master_key.md#prepare_master_key_dir)
- [`master_key::require_master_key`](master_key.md#require_master_key)
- [`master_key::resolve_master_key_detailed`](master_key.md#resolve_master_key_detailed)
- [`master_key::set_key_store_preference`](master_key.md#set_key_store_preference)
- [`master_key::set_keychain_vault`](master_key.md#set_keychain_vault)
- [`master_key::set_systemd_creds_vault`](master_key.md#set_systemd_creds_vault)
- [`master_key::store_master_key`](master_key.md#store_master_key)
- [`master_key::uncache_master_key`](master_key.md#uncache_master_key)
- [`master_key::warn_unwrapped_master_key`](master_key.md#warn_unwrapped_master_key)
- [`master_key::wrap_master_key`](master_key.md#wrap_master_key)
- [`master_key::KeyStorePreference`](master_key.md#keystorepreference)
- [`master_key::MasterKey`](master_key.md#masterkey)
- [`master_key::MasterKeyFormat`](master_key.md#masterkeyformat)
- [`master_key::MasterKeyResolution`](master_key.md#masterkeyresolution)
- [`master_key::StoredMasterKey`](master_key.md#storedmasterkey)
- [`master_key::SystemdCredsFactory`](master_key.md#systemdcredsfactory)
- [`master_key::AUTH_PASSWORD_ENV` as `MASTER_KEY_AUTH_PASSWORD_ENV`](master_key.md#auth_password_env)
- [`master_key_recovery::record_recovery_copy`](master_key_recovery.md#record_recovery_copy)
- [`master_key_recovery::recovery_copy_state`](master_key_recovery.md#recovery_copy_state)
- [`master_key_recovery::recovery_record_path`](master_key_recovery.md#recovery_record_path)
- [`master_key_recovery::RecoveryCopyChoice`](master_key_recovery.md#recoverycopychoice)
- [`master_key_recovery::RecoveryCopyState`](master_key_recovery.md#recoverycopystate)
- [`migrations::binding_bootstrap_type_env`](migrations.md#binding_bootstrap_type_env)
- [`migrations::host_migration_plan`](migrations.md#host_migration_plan)
- [`migrations::min_supported_schema_version`](migrations.md#min_supported_schema_version)
- [`migrations::LIBRARY_LEASES_DDL`](migrations.md#library_leases_ddl)
- [`migrations::SCHEMA_MIGRATIONS_DDL`](migrations.md#schema_migrations_ddl)
- [`migrations::SCHEMA_VERSION`](migrations.md#schema_version)
- [`models::is_episode`](../bookclerk_storefront/content_kind.md#is_episode)
- [`models::is_podcast_parent`](../bookclerk_storefront/content_kind.md#is_podcast_parent)
- [`operator_token::read_or_create_operator_token`](operator_token.md#read_or_create_operator_token)
- [`operator_token::resolve_operator_token`](operator_token.md#resolve_operator_token)
- [`operator_token::rotate_operator_token`](operator_token.md#rotate_operator_token)
- [`operator_token::ResolveOperatorToken`](operator_token.md#resolveoperatortoken)
- [`password::hash_password`](password.md#hash_password)
- [`password::verify_password`](password.md#verify_password)
- [`proxy_txn::consume_begin_injection`](../bookclerk_db_exec/proxy_txn.md#consume_begin_injection)
- [`proxy_txn::consume_commit_injection`](../bookclerk_db_exec/proxy_txn.md#consume_commit_injection)
- [`proxy_txn::current_exec_budget`](../bookclerk_db_exec/proxy_txn.md#current_exec_budget)
- [`proxy_txn::inject_atomic_interrupt`](../bookclerk_db_exec/proxy_txn.md#inject_atomic_interrupt)
- [`proxy_txn::inject_atomic_interrupt_after`](../bookclerk_db_exec/proxy_txn.md#inject_atomic_interrupt_after)
- [`proxy_txn::inject_begin_failures`](../bookclerk_db_exec/proxy_txn.md#inject_begin_failures)
- [`proxy_txn::inject_commit_failures`](../bookclerk_db_exec/proxy_txn.md#inject_commit_failures)
- [`proxy_txn::inject_savepoint_release_failures`](../bookclerk_db_exec/proxy_txn.md#inject_savepoint_release_failures)
- [`proxy_txn::inject_savepoint_rollback_failures`](../bookclerk_db_exec/proxy_txn.md#inject_savepoint_rollback_failures)
- [`proxy_txn::is_txn_broken`](../bookclerk_db_exec/proxy_txn.md#is_txn_broken)
- [`proxy_txn::note_begin_failed`](../bookclerk_db_exec/proxy_txn.md#note_begin_failed)
- [`proxy_txn::note_commit_failed`](../bookclerk_db_exec/proxy_txn.md#note_commit_failed)
- [`proxy_txn::query_rows_seen`](../bookclerk_db_exec/proxy_txn.md#query_rows_seen)
- [`proxy_txn::take_txn_fault`](../bookclerk_db_exec/proxy_txn.md#take_txn_fault)
- [`proxy_txn::txn_broken_err`](../bookclerk_db_exec/proxy_txn.md#txn_broken_err)
- [`proxy_txn::AtomicInterruptKind`](../bookclerk_db_exec/proxy_txn.md#atomicinterruptkind)
- [`proxy_txn::AtomicInterruptPhase`](../bookclerk_db_exec/proxy_txn.md#atomicinterruptphase)
- [`proxy_txn::ExecBudget`](../bookclerk_db_exec/proxy_txn.md#execbudget)
- [`scope::RefreshKeeping`](scope.md#refreshkeeping)
- [`scope::SourceScope`](scope.md#sourcescope)
- [`secrets::b64_string_to_bytes`](secrets.md#b64_string_to_bytes)
- [`secrets::build_sealed_record`](secrets.md#build_sealed_record)
- [`secrets::bytes_to_b64_string`](secrets.md#bytes_to_b64_string)
- [`secrets::delete_secret`](secrets.md#delete_secret)
- [`secrets::delete_secrets_for_account`](secrets.md#delete_secrets_for_account)
- [`secrets::get_secret`](secrets.md#get_secret)
- [`secrets::list_secrets`](secrets.md#list_secrets)
- [`secrets::unseal_secret`](secrets.md#unseal_secret)
- [`secrets::upsert_secret`](secrets.md#upsert_secret)
- [`secrets::EncryptedSecretRecord`](secrets.md#encryptedsecretrecord)
- [`secrets::SecretStore`](secrets.md#secretstore)
- [`secrets::FORMAT_SEALED_V1`](secrets.md#format_sealed_v1)
- [`sql_plan::authorize_typed_request`](sql_plan/README.md#authorize_typed_request)
- [`sql_plan::execute_guest_atomic_with`](sql_plan/README.md#execute_guest_atomic_with)
- [`sql_plan::proxy_insert_returning_identity`](sql_plan/README.md#proxy_insert_returning_identity)
- [`sql_plan::proxy_read_kind`](sql_plan/README.md#proxy_read_kind)
- [`sql_plan::proxy_write_kind`](sql_plan/README.md#proxy_write_kind)
- [`sql_plan::unquote_sea_identifiers`](sql_plan/README.md#unquote_sea_identifiers)
- [`sql_plan::validate_plan`](sql_plan/README.md#validate_plan)
- [`sql_plan::wake_page_for_max_binds`](sql_plan/README.md#wake_page_for_max_binds)
- [`sql_plan::AtomicSession`](../bookclerk_db_exec/README.md#atomicsession)
- [`sql_plan::CompiledAtomic`](sql_plan/README.md#compiledatomic)

## Modules

- [`column_read`](column_read.md): Column reads that stay correct on the host's proxied store connection.
- [`control_plane`](control_plane.md): Database-backed control plane: node identity and typed configuration.
- [`email`](email.md): Contact-email normalization and library-backed validation.
- [`entities`](entities/README.md): SeaORM entities mirroring the greenfield schema in `crate::migrations`.
- [`library_link`](library_link.md): Named-atomic parameters for writing a resolved listing.
- [`master_key`](master_key.md): Data-encryption key (DEK) for `encrypted_secrets`.
- [`master_key_recovery`](master_key_recovery.md): What an install's operator did about a recovery copy of its key.
- [`migrations`](migrations.md): Host library schema: frozen plan steps plus the unreleased development pack.
- [`notifications`](notifications.md): Notifications: the router's rules and the shapes the library stores (`docs/adr/notifications.md`, `docs/server/notifications.md`).
- [`operator_token`](operator_token.md): Durable operator API token in `encrypted_secrets`.
- [`pacing`](pacing.md): Download pacing per store account: the limits, the operator's override, and the decision whether a download may start now.
- [`password`](password.md): Argon2id password hashing for first-party local login.
- [`proxy_txn`](proxy_txn.md): Fail-closed transaction injects re-exported from `bookclerk_db_exec`.
- [`quality`](quality.md): When a store's offer is a better file than the one the library holds.
- [`scope`](scope.md): Plugin-scoped library access.
- [`secrets`](secrets.md): Encrypted secrets store — DB-backed credential storage.
- [`sql_plan`](sql_plan/README.md): Host-owned generic SQL atomic plans for database adapters. Host-owned SQL atomic plans for database plugins.
- [`test_support`](test_support/README.md): A library store that behaves like one opened on an external database plugin, for tests.
- [`atomic_status`](atomic_status.md): Application status strings produced by host plan interpretation.
- [`secret_account_type`](secret_account_type.md): Ownership namespace for `encrypted_secrets.account_type`.
- [`secret_kind`](secret_kind.md): Well-known `kind` values for the `encrypted_secrets` table.

## Traits

### AtomicTxnBackend

```rust
pub trait AtomicTxnBackend: Send + Sync {
    fn delete_user<'life0, 'async_trait>(
        &'life0 self,
        id: i64,
    ) -> Pin<Box<dyn Future<Output = Result<()>> + Send + 'async_trait>>
    where
        Self: 'async_trait,
        'life0: 'async_trait,;
    fn set_user_status<'life0, 'async_trait>(
        &'life0 self,
        id: i64,
        status: UserStatus,
    ) -> Pin<Box<dyn Future<Output = Result<UserRecord>> + Send + 'async_trait>>
    where
        Self: 'async_trait,
        'life0: 'async_trait,;
    fn set_user_password_hash<'life0, 'life1, 'async_trait>(
        &'life0 self,
        id: i64,
        password_hash: Option<&'life1 str>,
    ) -> Pin<Box<dyn Future<Output = Result<UserRecord>> + Send + 'async_trait>>
    where
        Self: 'async_trait,
        'life0: 'async_trait,
        'life1: 'async_trait,;
    fn set_user_role<'life0, 'async_trait>(
        &'life0 self,
        id: i64,
        role: UserRole,
    ) -> Pin<Box<dyn Future<Output = Result<UserRecord>> + Send + 'async_trait>>
    where
        Self: 'async_trait,
        'life0: 'async_trait,;
    fn redeem_claim_ticket_to_session<'life0, 'life1, 'life2, 'life3, 'life4, 'life5, 'async_trait>(
        &'life0 self,
        token_hash: &'life1 str,
        session_hash: &'life2 str,
        expires_at: DateTime<Utc>,
        client: Option<&'life3 SessionClientInfo>,
        new_password_hash: Option<&'life4 str>,
        password_fingerprint: Option<&'life5 str>,
    ) -> Pin<Box<dyn Future<Output = Result<PortalIdentity>> + Send + 'async_trait>>
    where
        Self: 'async_trait,
        'life0: 'async_trait,
        'life1: 'async_trait,
        'life2: 'async_trait,
        'life3: 'async_trait,
        'life4: 'async_trait,
        'life5: 'async_trait,;
    fn take_oidc_rp_state<'life0, 'life1, 'async_trait>(
        &'life0 self,
        state_hash: &'life1 str,
    ) -> Pin<Box<dyn Future<Output = Result<Option<(String, String, String, String, Option<i64>)>>> + Send + 'async_trait>>
    where
        Self: 'async_trait,
        'life0: 'async_trait,
        'life1: 'async_trait,;
    fn take_webauthn_challenge<'life0, 'life1, 'life2, 'async_trait>(
        &'life0 self,
        challenge_id: &'life1 str,
        kind: &'life2 str,
    ) -> Pin<Box<dyn Future<Output = Result<Option<(Option<i64>, String)>>> + Send + 'async_trait>>
    where
        Self: 'async_trait,
        'life0: 'async_trait,
        'life1: 'async_trait,
        'life2: 'async_trait,;
    fn enqueue_job<'life0, 'async_trait>(
        &'life0 self,
        spec: EnqueueJobSpec,
    ) -> Pin<Box<dyn Future<Output = Result<EnqueueOutcome>> + Send + 'async_trait>>
    where
        Self: 'async_trait,
        'life0: 'async_trait,;
    fn claim_next_job<'life0, 'life1, 'life2, 'async_trait>(
        &'life0 self,
        resource_class: JobResourceClass,
        owner: &'life1 str,
        lease_secs: u64,
        operation_id: &'life2 str,
    ) -> Pin<Box<dyn Future<Output = Result<Option<JobRecord>>> + Send + 'async_trait>>
    where
        Self: 'async_trait,
        'life0: 'async_trait,
        'life1: 'async_trait,
        'life2: 'async_trait,;
    fn reserve_job_temp_path<'life0, 'life1, 'life2, 'async_trait>(
        &'life0 self,
        job_id: &'life1 str,
        path: &'life2 str,
        reserved_bytes: u64,
        quota_bytes: u64,
    ) -> Pin<Box<dyn Future<Output = Result<()>> + Send + 'async_trait>>
    where
        Self: 'async_trait,
        'life0: 'async_trait,
        'life1: 'async_trait,
        'life2: 'async_trait,;
    fn confirm_totp_enrollment<'life0, 'life1, 'async_trait>(
        &'life0 self,
        user_id: i64,
        record: &'life1 EncryptedSecretRecord,
    ) -> Pin<Box<dyn Future<Output = Result<()>> + Send + 'async_trait>>
    where
        Self: 'async_trait,
        'life0: 'async_trait,
        'life1: 'async_trait,;
    fn disable_user_totp<'life0, 'async_trait>(
        &'life0 self,
        user_id: i64,
    ) -> Pin<Box<dyn Future<Output = Result<()>> + Send + 'async_trait>>
    where
        Self: 'async_trait,
        'life0: 'async_trait,;
    fn publish_domain_event<'life0, 'async_trait>(
        &'life0 self,
        spec: PublishDomainEventSpec,
    ) -> Pin<Box<dyn Future<Output = Result<PublishDomainEventOutcome>> + Send + 'async_trait>>
    where
        Self: 'async_trait,
        'life0: 'async_trait,;
    fn set_acquire_status<'life0, 'life1, 'life2, 'life3, 'async_trait>(
        &'life0 self,
        holding: &'life1 TitleHolding,
        status: AcquireStatus,
        storage_key: Option<&'life2 str>,
        error_message: Option<&'life3 str>,
        event: PublishDomainEventSpec,
    ) -> Pin<Box<dyn Future<Output = Result<()>> + Send + 'async_trait>>
    where
        Self: 'async_trait,
        'life0: 'async_trait,
        'life1: 'async_trait,
        'life2: 'async_trait,
        'life3: 'async_trait,;
    fn dispatch_event_deliveries<'life0, 'life1, 'life2, 'life3, 'async_trait>(
        &'life0 self,
        event_id: &'life1 str,
        subscribers: &'life2 [EventSubscriber],
        operation_id: &'life3 str,
        mark_dispatched: bool,
    ) -> Pin<Box<dyn Future<Output = Result<u32>> + Send + 'async_trait>>
    where
        Self: 'async_trait,
        'life0: 'async_trait,
        'life1: 'async_trait,
        'life2: 'async_trait,
        'life3: 'async_trait,;
    fn link_listing<'life0, 'async_trait>(
        &'life0 self,
        params: ListingLinkParams,
    ) -> Pin<Box<dyn Future<Output = Result<String>> + Send + 'async_trait>>
    where
        Self: 'async_trait,
        'life0: 'async_trait,;
    fn write_credits<'life0, 'async_trait>(
        &'life0 self,
        params: CreditBatchParams,
    ) -> Pin<Box<dyn Future<Output = Result<()>> + Send + 'async_trait>>
    where
        Self: 'async_trait,
        'life0: 'async_trait,;
    fn claim_event_delivery<'life0, 'life1, 'life2, 'life3, 'life4, 'life5, 'async_trait>(
        &'life0 self,
        delivery_id: &'life1 str,
        owner: &'life2 str,
        lease_secs: u64,
        operation_id: &'life3 str,
        plugin_id: &'life4 str,
        resource_class: &'life5 str,
        max_in_flight: u32,
    ) -> Pin<Box<dyn Future<Output = Result<Option<EventDeliveryRecord>>> + Send + 'async_trait>>
    where
        Self: 'async_trait,
        'life0: 'async_trait,
        'life1: 'async_trait,
        'life2: 'async_trait,
        'life3: 'async_trait,
        'life4: 'async_trait,
        'life5: 'async_trait,;
}
```

Backend that runs [`crate::LibraryStore`](#librarystore) named security operations as a
single guest atomic batch.

Implementations must preserve the same fail-closed semantics as the SeaORM
path: last-owner refusals mutate nothing; a failed claim redeem must not
consume the ticket or write a password/session; consume-once OIDC/WebAuthn
rows must not be observable by a concurrent caller; TOTP enroll/disable must
keep `encrypted_secrets` and `users.totp_enabled` in the same transaction.

#### AtomicTxnBackend::delete_user

```rust
fn delete_user<'life0, 'async_trait>(
    &'life0 self,
    id: i64,
) -> Pin<Box<dyn Future<Output = Result<()>> + Send + 'async_trait>>
where
    Self: 'async_trait,
    'life0: 'async_trait,
```

Delete a first-party user and personal data (last-owner guarded).

#### AtomicTxnBackend::set_user_status

```rust
fn set_user_status<'life0, 'async_trait>(
    &'life0 self,
    id: i64,
    status: UserStatus,
) -> Pin<Box<dyn Future<Output = Result<UserRecord>> + Send + 'async_trait>>
where
    Self: 'async_trait,
    'life0: 'async_trait,
```

Set user status (`active` / `disabled`; last-owner guarded).

#### AtomicTxnBackend::set_user_password_hash

```rust
fn set_user_password_hash<'life0, 'life1, 'async_trait>(
    &'life0 self,
    id: i64,
    password_hash: Option<&'life1 str>,
) -> Pin<Box<dyn Future<Output = Result<UserRecord>> + Send + 'async_trait>>
where
    Self: 'async_trait,
    'life0: 'async_trait,
    'life1: 'async_trait,
```

Set or clear the Argon2id password hash and bump `security_version`.

#### AtomicTxnBackend::set_user_role

```rust
fn set_user_role<'life0, 'async_trait>(
    &'life0 self,
    id: i64,
    role: UserRole,
) -> Pin<Box<dyn Future<Output = Result<UserRecord>> + Send + 'async_trait>>
where
    Self: 'async_trait,
    'life0: 'async_trait,
```

Set user role (last-owner guarded on demotion).

#### AtomicTxnBackend::redeem_claim_ticket_to_session

```rust
fn redeem_claim_ticket_to_session<'life0, 'life1, 'life2, 'life3, 'life4, 'life5, 'async_trait>(
    &'life0 self,
    token_hash: &'life1 str,
    session_hash: &'life2 str,
    expires_at: DateTime<Utc>,
    client: Option<&'life3 SessionClientInfo>,
    new_password_hash: Option<&'life4 str>,
    password_fingerprint: Option<&'life5 str>,
) -> Pin<Box<dyn Future<Output = Result<PortalIdentity>> + Send + 'async_trait>>
where
    Self: 'async_trait,
    'life0: 'async_trait,
    'life1: 'async_trait,
    'life2: 'async_trait,
    'life3: 'async_trait,
    'life4: 'async_trait,
    'life5: 'async_trait,
```

Consume a claim ticket, optionally set a first password, mint a session.

#### AtomicTxnBackend::take_oidc_rp_state

```rust
fn take_oidc_rp_state<'life0, 'life1, 'async_trait>(
    &'life0 self,
    state_hash: &'life1 str,
) -> Pin<Box<dyn Future<Output = Result<Option<(String, String, String, String, Option<i64>)>>> + Send + 'async_trait>>
where
    Self: 'async_trait,
    'life0: 'async_trait,
    'life1: 'async_trait,
```

Consume a one-time OIDC RP state. `Ok(None)` if missing or expired.

Tuple is `(provider_id, pkce_verifier, nonce, purpose, user_id)`.

#### AtomicTxnBackend::take_webauthn_challenge

```rust
fn take_webauthn_challenge<'life0, 'life1, 'life2, 'async_trait>(
    &'life0 self,
    challenge_id: &'life1 str,
    kind: &'life2 str,
) -> Pin<Box<dyn Future<Output = Result<Option<(Option<i64>, String)>>> + Send + 'async_trait>>
where
    Self: 'async_trait,
    'life0: 'async_trait,
    'life1: 'async_trait,
    'life2: 'async_trait,
```

Consume a one-time WebAuthn challenge. `Ok(None)` if missing or expired.

Tuple is `(user_id, state_json)`.

#### AtomicTxnBackend::enqueue_job

```rust
fn enqueue_job<'life0, 'async_trait>(
    &'life0 self,
    spec: EnqueueJobSpec,
) -> Pin<Box<dyn Future<Output = Result<EnqueueOutcome>> + Send + 'async_trait>>
where
    Self: 'async_trait,
    'life0: 'async_trait,
```

Admit a durable job in one atomic transaction.

#### AtomicTxnBackend::claim_next_job

```rust
fn claim_next_job<'life0, 'life1, 'life2, 'async_trait>(
    &'life0 self,
    resource_class: JobResourceClass,
    owner: &'life1 str,
    lease_secs: u64,
    operation_id: &'life2 str,
) -> Pin<Box<dyn Future<Output = Result<Option<JobRecord>>> + Send + 'async_trait>>
where
    Self: 'async_trait,
    'life0: 'async_trait,
    'life1: 'async_trait,
    'life2: 'async_trait,
```

Claim the next ready job; `operation_id` makes a lost response replay-safe.

#### AtomicTxnBackend::reserve_job_temp_path

```rust
fn reserve_job_temp_path<'life0, 'life1, 'life2, 'async_trait>(
    &'life0 self,
    job_id: &'life1 str,
    path: &'life2 str,
    reserved_bytes: u64,
    quota_bytes: u64,
) -> Pin<Box<dyn Future<Output = Result<()>> + Send + 'async_trait>>
where
    Self: 'async_trait,
    'life0: 'async_trait,
    'life1: 'async_trait,
    'life2: 'async_trait,
```

Reserve scratch-quota bytes for `path` on `job_id`.

#### AtomicTxnBackend::confirm_totp_enrollment

```rust
fn confirm_totp_enrollment<'life0, 'life1, 'async_trait>(
    &'life0 self,
    user_id: i64,
    record: &'life1 EncryptedSecretRecord,
) -> Pin<Box<dyn Future<Output = Result<()>> + Send + 'async_trait>>
where
    Self: 'async_trait,
    'life0: 'async_trait,
    'life1: 'async_trait,
```

Promote a sealed TOTP secret to `primary` and set `totp_enabled`.

#### AtomicTxnBackend::disable_user_totp

```rust
fn disable_user_totp<'life0, 'async_trait>(
    &'life0 self,
    user_id: i64,
) -> Pin<Box<dyn Future<Output = Result<()>> + Send + 'async_trait>>
where
    Self: 'async_trait,
    'life0: 'async_trait,
```

Delete TOTP secrets and clear `totp_enabled`.

#### AtomicTxnBackend::publish_domain_event

```rust
fn publish_domain_event<'life0, 'async_trait>(
    &'life0 self,
    spec: PublishDomainEventSpec,
) -> Pin<Box<dyn Future<Output = Result<PublishDomainEventOutcome>> + Send + 'async_trait>>
where
    Self: 'async_trait,
    'life0: 'async_trait,
```

Persist a domain event in the outbox.

#### AtomicTxnBackend::set_acquire_status

```rust
fn set_acquire_status<'life0, 'life1, 'life2, 'life3, 'async_trait>(
    &'life0 self,
    holding: &'life1 TitleHolding,
    status: AcquireStatus,
    storage_key: Option<&'life2 str>,
    error_message: Option<&'life3 str>,
    event: PublishDomainEventSpec,
) -> Pin<Box<dyn Future<Output = Result<()>> + Send + 'async_trait>>
where
    Self: 'async_trait,
    'life0: 'async_trait,
    'life1: 'async_trait,
    'life2: 'async_trait,
    'life3: 'async_trait,
```

Write an edition's acquire state for the account holding that made
the write, and publish its event, in one batch.

Only the holding that acquired an edition, while it exists, changes
it.

##### Errors

Returns [`crate::LibraryError::NotFound`](#libraryerror) when the holding no longer
holds the edition, or the database error.

#### AtomicTxnBackend::dispatch_event_deliveries

```rust
fn dispatch_event_deliveries<'life0, 'life1, 'life2, 'life3, 'async_trait>(
    &'life0 self,
    event_id: &'life1 str,
    subscribers: &'life2 [EventSubscriber],
    operation_id: &'life3 str,
    mark_dispatched: bool,
) -> Pin<Box<dyn Future<Output = Result<u32>> + Send + 'async_trait>>
where
    Self: 'async_trait,
    'life0: 'async_trait,
    'life1: 'async_trait,
    'life2: 'async_trait,
    'life3: 'async_trait,
```

Create deliveries for `subscribers`. `mark_dispatched` finishes the parent event.

#### AtomicTxnBackend::link_listing

```rust
fn link_listing<'life0, 'async_trait>(
    &'life0 self,
    params: ListingLinkParams,
) -> Pin<Box<dyn Future<Output = Result<String>> + Send + 'async_trait>>
where
    Self: 'async_trait,
    'life0: 'async_trait,
```

Write one resolved listing as one guarded batch.

Returns the batch status: `ok`, or `retry` when another writer
changed the rows the plan was resolved against.

#### AtomicTxnBackend::write_credits

```rust
fn write_credits<'life0, 'async_trait>(
    &'life0 self,
    params: CreditBatchParams,
) -> Pin<Box<dyn Future<Output = Result<()>> + Send + 'async_trait>>
where
    Self: 'async_trait,
    'life0: 'async_trait,
```

Write one batch of contributors, credits, series and series entries.

#### AtomicTxnBackend::claim_event_delivery

```rust
fn claim_event_delivery<'life0, 'life1, 'life2, 'life3, 'life4, 'life5, 'async_trait>(
    &'life0 self,
    delivery_id: &'life1 str,
    owner: &'life2 str,
    lease_secs: u64,
    operation_id: &'life3 str,
    plugin_id: &'life4 str,
    resource_class: &'life5 str,
    max_in_flight: u32,
) -> Pin<Box<dyn Future<Output = Result<Option<EventDeliveryRecord>>> + Send + 'async_trait>>
where
    Self: 'async_trait,
    'life0: 'async_trait,
    'life1: 'async_trait,
    'life2: 'async_trait,
    'life3: 'async_trait,
    'life4: 'async_trait,
    'life5: 'async_trait,
```

CAS-claim one pending delivery after the host has filtered eligibility.

Implemented by: `NamedAtomicBackend<R>`

### TypedAtomicExec

```rust
pub trait TypedAtomicExec: Send + Sync {
    fn execute_typed<'life0, 'async_trait>(
        &'life0 self,
        envelope: AdapterExecuteRequest,
    ) -> Pin<Box<dyn Future<Output = Result<ExecuteReply, PluginError>> + Send + 'async_trait>>
    where
        Self: 'async_trait,
        'life0: 'async_trait,;
}
```

Runs a host-authorized typed [`bookclerk_plugin_abi::ExecuteRequest`](../bookclerk_plugin_abi/db_execute.md#executerequest) as one guest `executeAtomic`.

First-party hosts attach this alongside [`AtomicTxnBackend`](#atomictxnbackend) so granted job
sessions do not round-trip through the SeaORM proxy (`BEGIN` + nested
`query`/`execute`). In-process tests leave it unset and run the same
request on the local connection.

#### TypedAtomicExec::execute_typed

```rust
fn execute_typed<'life0, 'async_trait>(
    &'life0 self,
    envelope: AdapterExecuteRequest,
) -> Pin<Box<dyn Future<Output = Result<ExecuteReply, PluginError>> + Send + 'async_trait>>
where
    Self: 'async_trait,
    'life0: 'async_trait,
```

Executes `req` as one typed atomic batch.

##### Errors

Returns a plugin ABI error when validation, transport, or the engine
rejects the batch.

Implemented by: `ProfileExec`

### NamedAtomicRunner

```rust
pub trait NamedAtomicRunner: Send + Sync {
    fn run_compiled<'life0, 'async_trait>(
        &'life0 self,
        compiled: CompiledAtomic,
        operation_id: String,
    ) -> Pin<Box<dyn Future<Output = Result<DbAtomicResult>> + Send + 'async_trait>>
    where
        Self: 'async_trait,
        'life0: 'async_trait,;
}
```

Runs one compiled named plan as a single atomic batch.

A runner is the transport under a [`NamedAtomicBackend`]: it receives the
plan the backend compiled and answers with the interpreted result. It
must apply the whole plan or none of it, and must return application
outcomes (`notFound`, `lastOwner`, `empty`, …) as the result's status,
not as errors.

#### NamedAtomicRunner::run_compiled

```rust
fn run_compiled<'life0, 'async_trait>(
    &'life0 self,
    compiled: CompiledAtomic,
    operation_id: String,
) -> Pin<Box<dyn Future<Output = Result<DbAtomicResult>> + Send + 'async_trait>>
where
    Self: 'async_trait,
    'life0: 'async_trait,
```

Runs `compiled` under `operation_id`, the idempotency key a retry
after a lost reply resumes.

##### Errors

Returns [`LibraryError::LimitExceeded`](#libraryerror) when the plan or its result
exceeds a negotiated cap, [`LibraryError::Unavailable`](#libraryerror) when the reply
was lost and the batch may have been applied, and another
[`LibraryError`](#libraryerror) when the engine refused the batch.

Implemented by: `InProcessSqlite`, `ProfileAtomic`

## Structs

### DbAtomicResult

```rust
pub struct DbAtomicResult {
    pub status: String,
    pub payload: Option<Value>,
    pub operation_id: String,
    pub replayed: bool,
    pub receipt_created_at: Option<String>,
    pub timing: Option<DbTiming>,
}
```

Host-interpreted result of a named atomic operation.

**Fields**

- `status`: Application outcome (`ok`, `empty`, `notFound`, `lastOwner`, …).
- `payload`: Op-specific record when `status` is `ok`, or the existing record a
  `duplicate` folded into (`enqueueJob`, `publishDomainEvent`).
- `operation_id`: Echo of the request `operationId`.
- `replayed`: True when this result was loaded from a durable receipt.
- `receipt_created_at`: RFC 3339 timestamp when the receipt was first written.
- `timing`: Handler/engine timing. Not hashed for idempotency.

#### DbAtomicResult::ok

```rust
pub fn ok(payload: JsonValue) -> Self
```

Successful operation with a JSON payload.

#### DbAtomicResult::ok_unit

```rust
pub fn ok_unit() -> Self
```

Successful operation with no payload (`deleteUser`).

#### DbAtomicResult::with_status

```rust
pub fn with_status(status: &str) -> Self
```

Application failure or empty consume-once result (no payload).

Implements: `Clone`, `Debug`, `Deserialize<'de>`, `PartialEq`, `Serialize`

### BackendMigrateOptions

```rust
pub struct BackendMigrateOptions {
    pub dry_run: bool,
    pub force: bool,
}
```

Options for a backend-to-backend library copy.

**Fields**

- `dry_run`: When true, report counts only — do not write to the destination.
- `force`: Allow copying into a destination that already has library rows.

Implements: `Clone`, `Debug`

### BackendMigrateSummary

```rust
pub struct BackendMigrateSummary {
    pub tables: BTreeMap<String, usize>,
    pub dry_run: bool,
}
```

Per-table row counts copied (or that would be copied in dry-run mode).

**Fields**

- `tables`: Map of table name → rows copied (or counted in dry-run).
  `plugin_artifacts` counts archives; their chunks are not counted.
- `dry_run`: When true, counts rows without writing to the destination.

#### BackendMigrateSummary::total_rows

```rust
pub fn total_rows(&self) -> usize
```

Sums row counts across all tables in the summary, leaving out
`plugin_artifacts`, which counts archives rather than rows.

Implements: `Clone`, `Debug`, `Default`, `Serialize`

### BackupRepository

```rust
pub struct BackupRepository {
    /* private fields */
}
```

On-disk backup repository rooted at `files_dir/backups`.

#### BackupRepository::open

```rust
pub fn open(files_dir: &Path) -> Result<Self>
```

Opens (creating) the repository under `files_dir/backups`.

##### Errors

Returns when the backups directory cannot be created.

#### BackupRepository::open_root

```rust
pub fn open_root(root: &Path) -> Result<Self>
```

Opens `root` as a backups repository (`manifests/` + `objects/`).

##### Errors

Returns when the directories cannot be created.

#### BackupRepository::root

```rust
pub fn root(&self) -> &Path
```

Repository root (`…/backups`).

#### BackupRepository::lock_exclusive

```rust
pub fn lock_exclusive(&self) -> Result<RepoLockGuard<'_>>
```

Exclusive lock covering object publication and GC/prune.

Same-thread reentry is allowed so [`Self::put_object`](#backuprepositoryput_object) can nest under a
backup that already holds the lock. Other threads wait. Other processes
block on `backups/.lock`.

##### Errors

Returns when the lock file cannot be created or flocked.

#### BackupRepository::put_object

```rust
pub fn put_object(&self, object: &CanonicalObject) -> Result<String>
```

Writes `object` if missing. Returns the uncompressed SHA-256 hex.

The write uses a unique same-directory temp file, fsyncs it, then
installs with a no-clobber hard link. An existing object is reused only
after its bytes verify; a truncated or corrupt file is replaced.

##### Errors

Returns when encoding, compression, or the filesystem write fails.

#### BackupRepository::get_object

```rust
pub fn get_object(&self, digest: &str) -> Result<CanonicalObject>
```

Loads and verifies one object by uncompressed digest.

##### Errors

Returns when the object is missing, corrupt, or the digest mismatches.

#### BackupRepository::object_exists

```rust
pub fn object_exists(&self, digest: &str) -> bool
```

True when an object file exists (does not imply the bytes are valid).

#### BackupRepository::publish_manifest

```rust
pub fn publish_manifest(&self, manifest: &BackupManifest) -> Result<PathBuf>
```

Publishes `manifest` only after all referenced objects exist **and**
verify.

Writes JSON then a sibling `.sha256` of the exact file bytes. Incomplete
staging uses `manifests/.staging-<id>.json` and is ignored by list/GC.

##### Errors

Returns when a referenced object is missing or corrupt, or the
filesystem write fails.

#### BackupRepository::read_manifest

```rust
pub fn read_manifest(&self, id: &str) -> Result<BackupManifest>
```

Reads and integrity-checks a published manifest.

##### Errors

Returns when the file is missing, JSON is malformed, the sidecar hash
mismatches, or the embedded id does not match the filename.

#### BackupRepository::list_manifests

```rust
pub fn list_manifests(&self) -> Vec<BackupManifest>
```

Lists published recovery-point manifests oldest-first. Staging files
and unreadable manifests are skipped.

#### BackupRepository::list_manifests_strict

```rust
pub fn list_manifests_strict(&self) -> Result<Vec<BackupManifest>>
```

Lists every published recovery-point manifest, failing if any cannot
be verified. Staging files (dot-prefixed) are ignored.

Use this for prune/GC. [`Self::list_manifests`](#backuprepositorylist_manifests) stays best-effort for
non-destructive listing.

##### Errors

Returns when the manifests directory cannot be read, a published
`*.json` fails [`Self::read_manifest`](#backuprepositoryread_manifest), or an unexpected non-sidecar
file is present.

#### BackupRepository::delete_manifest

```rust
pub fn delete_manifest(&self, id: &str) -> Result<bool>
```

Deletes one published manifest (not its objects). Returns whether a
file was removed.

##### Errors

Returns when the id is unsafe or unlink fails for a reason other than
not found.

#### BackupRepository::gc_unreferenced_objects

```rust
pub fn gc_unreferenced_objects(&self) -> Result<usize>
```

Deletes objects not referenced by any retained published manifest.

Reachability is computed from a **strict** enumeration of published
manifests. An unreadable retained manifest fails closed instead of
treating its objects as garbage.

##### Errors

Returns when a published manifest cannot be verified, a directory
cannot be read, or an unreferenced object cannot be unlinked.

Implements: `Clone`, `Debug`

### BackupListEntry

```rust
pub struct BackupListEntry {
    pub manifest: BackupManifest,
}
```

Catalog entry used by `bookclerk db backup list`.

**Fields**

- `manifest`: Parsed manifest.

Implements: `Clone`, `Debug`

### BackupManifest

```rust
pub struct BackupManifest {
    pub format_version: u32,
    pub id: String,
    pub created_at: String,
    pub schema_state: String,
    pub schema_checksum: Option<String>,
    pub frozen_version: Option<i64>,
    pub sql_contract_version: u32,
    pub app_version: String,
    pub reason: BackupReason,
    pub migrate_to: i64,
    pub include_plugin_databases: bool,
    pub units: Vec<BackupUnit>,
}
```

On-disk recovery-point metadata (`manifests/<id>.json`).

**Fields**

- `format_version`: Catalog format (`1` for content-addressed canonical objects).
- `id`: Stable recovery-point id (UUID).
- `created_at`: RFC 3339 UTC timestamp.
- `schema_state`: [`SchemaState::display`](#schemastatedisplay) at capture.
- `schema_checksum`: SHA-256 recorded on the library schema state marker.
- `frozen_version`: Frozen revision when the source library was frozen.
- `sql_contract_version`: Bookclerk SQL contract of the library unit.
- `app_version`: Bookclerk version that wrote the backup.
- `reason`: Operator vs automatic.
- `migrate_to`: Target frozen version a following migrate step was heading toward.
- `include_plugin_databases`: When true, every registered plugin binding was required at capture.
- `units`: Logical database units (library first).

#### BackupManifest::referenced_objects

```rust
pub fn referenced_objects(&self) -> BTreeSet<String>
```

Union of every object digest referenced by this recovery point.

#### BackupManifest::library_unit

```rust
pub fn library_unit(&self) -> Result<&BackupUnit>
```

Library unit from this manifest.

##### Errors

Returns when the library unit is missing or duplicated.

#### BackupManifest::plugin_units

```rust
pub fn plugin_units(&self) -> Vec<&BackupUnit>
```

Plugin binding units.

Implements: `Clone`, `Debug`, `Deserialize<'de>`, `Serialize`

### BackupOutcome

```rust
pub struct BackupOutcome {
    pub dir: PathBuf,
    pub manifest: BackupManifest,
}
```

Result of publishing a recovery point.

**Fields**

- `dir`: Repository root (`files_dir/backups`).
- `manifest`: Parsed manifest.

Implements: `Clone`, `Debug`

### BackupRequest

```rust
pub struct BackupRequest {
    pub files_dir: PathBuf,
    pub schema_state: SchemaState,
    pub reason: BackupReason,
    pub to_version: i64,
    pub include_plugin_databases: bool,
    pub consistent_backup_read: bool,
    pub backend_at_capture: String,
    pub max_result_rows: u32,
    pub max_result_bytes: u32,
    pub max_atomic_result_bytes: u32,
    pub plugin_units: Vec<PreparedPluginUnit>,
    pub adapter: Option<SharedAdapterBackupOps>,
    pub in_process: bool,
}
```

Inputs for an automatic or CLI backup.

**Fields**

- `files_dir`: `$BOOKCLERK_FILES_DIR`.
- `schema_state`: Durable library schema state at capture. [`SchemaState::Uninitialized`](#schemastate) skips.
- `reason`: Operator vs pre-migrate.
- `to_version`: Version the upcoming migrate step will target (`0` for manual).
- `include_plugin_databases`: Include every registered plugin-owned database binding.
- `consistent_backup_read`: Guest advertised consistent backup read.
- `backend_at_capture`: Adapter id at capture (diagnostic; not used to branch restore).
- `max_result_rows`: Negotiated `maxResultRows` (clamped).
- `max_result_bytes`: Negotiated `maxResultBytes` for one SELECT page.
- `max_atomic_result_bytes`: Negotiated `maxAtomicResultBytes` (capture page budget is the min of both).
- `plugin_units`: Plugin sessions prepared by the host (must match the registry when inclusion is on).
- `adapter`: Library adapter hooks. `None` uses the in-process SDK (tests).
- `in_process`: True when leftover SQL is physically lowered on the opened connection.

Implements: `Clone`, `Debug`

### BackupUnit

```rust
pub struct BackupUnit {
    pub kind: DatabaseUnitKind,
    pub plugin_id: Option<String>,
    pub binding: Option<String>,
    pub backend_at_capture: String,
    pub sql_contract_version: u32,
    pub schema_object: String,
    pub identity_object: String,
    pub tables: Vec<BackupTable>,
    pub plugin_schema_namespace: Option<String>,
    pub plugin_schema_state: Option<String>,
    pub plugin_schema_checksum: Option<String>,
}
```

One logical database in a recovery point (library or plugin binding).

**Fields**

- `kind`: Library vs plugin binding.
- `plugin_id`: Plugin id when [`DatabaseUnitKind::PluginBinding`](#databaseunitkind).
- `binding`: Binding name when [`DatabaseUnitKind::PluginBinding`](#databaseunitkind).
- `backend_at_capture`: Adapter that captured this unit (diagnostic only; restore is cross-adapter).
- `sql_contract_version`: Bookclerk SQL grammar/ABI contract, not a library schema revision.
- `schema_object`: SHA-256 of the schema object.
- `identity_object`: SHA-256 of the identity object.
- `tables`: User-visible tables (not catalog/identity companions).
- `plugin_schema_namespace`: Plugin id when [`DatabaseUnitKind::PluginBinding`](#databaseunitkind) (owner identity).
- `plugin_schema_state`: [`crate::PluginMigrationHistory::display`](#pluginmigrationhistorydisplay) at capture (`history@{n}+{digest}`).
- `plugin_schema_checksum`: Digest of the complete ordered `(migration_id, checksum)` journal.

#### BackupUnit::restore_kind

```rust
pub fn restore_kind(&self) -> CanonicalRestoreKind
```

Restore companion kind for this unit.

#### BackupUnit::referenced_objects

```rust
pub fn referenced_objects(&self) -> BTreeSet<String>
```

Object digests this unit needs to restore.

Implements: `Clone`, `Debug`, `Deserialize<'de>`, `Eq`, `PartialEq`, `Serialize`

### CanonicalDatabaseSchema

```rust
pub struct CanonicalDatabaseSchema {
    pub sql_contract_version: u32,
    pub tables: Vec<CanonicalTableSchema>,
    pub indexes: Vec<CreateIndexSchema>,
}
```

Admitted canonical schema used to export or restore one database unit.

**Fields**

- `sql_contract_version`: Bookclerk SQL grammar/ABI contract, not a library schema revision.
- `tables`: `CREATE TABLE` statements in FK-safe order after [`crate::sort_tables_by_foreign_keys`](#sort_tables_by_foreign_keys).
- `indexes`: `CREATE INDEX` statements (after tables).

#### CanonicalDatabaseSchema::schema_sql

```rust
pub fn schema_sql(&self) -> Vec<String>
```

Ordered canonical DDL (tables then indexes).

#### CanonicalDatabaseSchema::table_names

```rust
pub fn table_names(&self) -> Vec<String>
```

Folded user table names in CREATE order.

Implements: `Clone`, `Debug`, `Eq`, `PartialEq`

### CanonicalExportOpts

```rust
pub struct CanonicalExportOpts {
    pub consistent_backup_read: bool,
    pub skip_tables: BTreeSet<String>,
    pub max_result_rows: u32,
    pub max_result_bytes: u32,
    pub max_atomic_result_bytes: u32,
    pub chunk_target_bytes: usize,
    pub adapter: Option<SharedAdapterBackupOps>,
    pub in_process: bool,
}
```

Options for a consistent canonical export.

**Fields**

- `consistent_backup_read`: Guest advertised `DbCapabilities::supports_consistent_backup_read`.
- `skip_tables`: Folded table names omitted from row capture (library `plugin_databases`).
- `max_result_rows`: Page size for `SELECT … LIMIT` (honors adapter `maxResultRows`).
- `max_result_bytes`: Encoded-byte budget for one SELECT page (`maxResultBytes`).
- `max_atomic_result_bytes`: Encoded-byte budget for one atomic reply (`maxAtomicResultBytes`).
- `chunk_target_bytes`: Uncompressed JSON target for one table-data chunk.
- `adapter`: Adapter snapshot/identity hooks. `None` uses the in-process SDK (tests).
- `in_process`: True when leftover SQL is physically lowered on the opened connection.

#### CanonicalExportOpts::from_caps

```rust
pub fn from_caps(caps: &DbCapabilities) -> Self
```

Page-size and result-byte limits from negotiated adapter capabilities.

#### CanonicalExportOpts::page_byte_budget

```rust
pub fn page_byte_budget(&self) -> usize
```

Effective encoded-byte budget for one capture SELECT page.

Implements: `Clone`, `Debug`, `Default`

### CanonicalRestoreOpts

```rust
pub struct CanonicalRestoreOpts {
    pub atomic_unit_restore: bool,
    pub max_binds: u32,
    pub max_payload_bytes: u32,
    pub max_request_bytes: u32,
    pub adapter: Option<SharedAdapterBackupOps>,
    pub in_process: bool,
}
```

Negotiated limits for parameterized canonical restore.

**Fields**

- `atomic_unit_restore`: Guest advertised `atomicUnitRestore`.
- `max_binds`: Maximum bound parameters per INSERT (`maxBinds`).
- `max_payload_bytes`: Maximum UTF-8 bytes of SQL plus binds per statement (`maxPayloadBytes`).
- `max_request_bytes`: Maximum encoded bytes of one [`bookclerk_plugin_abi::ExecuteRequest`](../bookclerk_plugin_abi/db_execute.md#executerequest).
- `adapter`: Adapter restore hooks. `None` uses the in-process SDK (tests).
- `in_process`: True when leftover SQL is physically lowered on the opened connection.

#### CanonicalRestoreOpts::from_caps

```rust
pub fn from_caps(caps: &DbCapabilities) -> Result<Self>
```

Restore limits copied from negotiated adapter capabilities.

The versioning contract is checked with [`require_schema_migrations`](#require_schema_migrations),
never inferred from SeaORM [`sea_orm::DbBackend`](https://docs.rs/sea_orm/latest/sea_orm/database/db_connection/type.DbBackend.html). Host policy requires
`schemaMigrations`.

##### Errors

Returns when `caps` is not a known versioning contract (`schemaMigrations`
missing).

Implements: `Clone`, `Debug`, `Default`

### CanonicalTableSchema

```rust
pub struct CanonicalTableSchema {
    pub create_sql: String,
    pub parsed: CreateTableSchema,
}
```

One admitted `CREATE TABLE` plus its parsed IR.

**Fields**

- `create_sql`: Original canonical SQL (no trailing semicolon).
- `parsed`: Parsed columns, constraints, and identity.

Implements: `Clone`, `Debug`, `Eq`, `PartialEq`

### IdentityHighWater

```rust
pub struct IdentityHighWater {
    pub column: String,
    pub last: i64,
}
```

Identity high-water for one generated column.

**Fields**

- `column`: Identity column name.
- `last`: Highest generated or stored value that must not be reused.

Implements: `Clone`, `Debug`, `Deserialize<'de>`, `Eq`, `PartialEq`, `Serialize`

### PreparedPluginUnit

```rust
pub struct PreparedPluginUnit {
    pub plugin_id: String,
    pub binding: String,
    pub backend_at_capture: String,
    pub db: DatabaseConnection,
    pub max_result_rows: u32,
    pub max_result_bytes: u32,
    pub max_atomic_result_bytes: u32,
    pub adapter: Option<SharedAdapterBackupOps>,
}
```

Open plugin binding captured through the active adapter session.

**Fields**

- `plugin_id`: Owning plugin id.
- `binding`: Binding name.
- `backend_at_capture`: Adapter that produced this unit (diagnostic).
- `db`: SeaORM connection opened via the active adapter (not a native dump).
- `max_result_rows`: Negotiated `maxResultRows` for this binding session.
- `max_result_bytes`: Negotiated `maxResultBytes` for this binding session.
- `max_atomic_result_bytes`: Negotiated `maxAtomicResultBytes` for this binding session.
- `adapter`: Binding-scoped adapter hooks. `None` uses the in-process SDK (tests).

Implements: `Clone`, `Debug`

### RestorePlan

```rust
pub struct RestorePlan {
    pub manifest: BackupManifest,
    pub plugin_units: Vec<BackupUnit>,
}
```

Library restore result plus plugin units for the host to apply.

Plugin databases are restored separately after this returns. A failure
while restoring a later unit leaves earlier units replaced (no bundle-level
atomicity). Each unit still uses complete replacement semantics.

**Fields**

- `manifest`: Verified manifest.
- `plugin_units`: Plugin units to restore onto the target adapter.

Implements: `Clone`, `Debug`

### SchemaBackupOpts

```rust
pub struct SchemaBackupOpts {
    pub files_dir: PathBuf,
    pub include_plugin_databases: bool,
    pub consistent_backup_read: bool,
    pub backend_at_capture: String,
    pub max_result_rows: u32,
    pub max_result_bytes: u32,
    pub max_atomic_result_bytes: u32,
    pub plugin_units: Vec<PreparedPluginUnit>,
    pub adapter: Option<SharedAdapterBackupOps>,
}
```

Optional in-place backup taken before applying ups or downs.

**Fields**

- `files_dir`: `$BOOKCLERK_FILES_DIR` root for `backups/`.
- `include_plugin_databases`: Include every registered plugin-owned database binding.
- `consistent_backup_read`: Guest advertised consistent backup read.
- `backend_at_capture`: Adapter id at capture (diagnostic).
- `max_result_rows`: Negotiated page size.
- `max_result_bytes`: Negotiated encoded SELECT-page budget.
- `max_atomic_result_bytes`: Negotiated encoded atomic-reply budget.
- `plugin_units`: Plugin sessions prepared by the host when inclusion is requested.
- `adapter`: Library adapter hooks. `None` uses the in-process SDK (tests).

Implements: `Clone`, `Debug`, `Default`

### ValidatedBackup

```rust
pub struct ValidatedBackup {
    pub manifest: BackupManifest,
    pub library: BackupUnit,
    pub plugin_units: Vec<BackupUnit>,
}
```

Recovery point whose objects, schema, and typed rows have been verified.

**Fields**

- `manifest`: Parsed, supported manifest.
- `library`: Library unit metadata (objects already verified in the repository).
- `plugin_units`: Plugin binding units (already verified).

Implements: `Clone`, `Debug`

### Missing

```rust
pub struct Missing {
    pub entity: EntityKind,
    pub id: Option<String>,
}
```

What a [`LibraryError::NotFound`](#libraryerror) could not find: the kind of row and,
when the caller named one, its id.

Displays as `<entity> <id> not found`, or `<entity> not found` without
an id, so an error body names the missing thing rather than presuming a
book.

**Fields**

- `entity`: The kind of row.
- `id`: The id the caller asked for, when it named one.

Implements: `Clone`, `Debug`, `Display`, `Eq`, `PartialEq`

### SchemaApplyOptions

```rust
pub struct SchemaApplyOptions {
    pub backup: Option<SchemaBackupOpts>,
}
```

Options for [`apply_host_schema_with_options`](#apply_host_schema_with_options).

**Fields**

- `backup`: When set, write a recovery point before applying DDL to a non-empty database.

Implements: `Clone`, `Debug`, `Default`

### SchemaBatch

```rust
pub struct SchemaBatch {
    pub statements: Vec<String>,
}
```

Canonical schema apply batch: already-separate host statements plus marker.

Adapters lower each statement at execution; they must not re-split SQL.

**Fields**

- `statements`: Ordered SQL strings; the last statement is the version/state marker.

#### SchemaBatch::from_statements_and_marker

```rust
pub fn from_statements_and_marker(
    stmts: impl IntoIterator<Item = impl Into<String>>,
    marker: impl Into<String>,
) -> Self
```

Builds a batch from an ordered statement list plus a trailing marker.

#### SchemaBatch::unreleased

```rust
pub fn unreleased(
    stmts: impl IntoIterator<Item = impl Into<String>>,
    checksum: &str,
) -> Self
```

Unreleased development pack plus `bookclerk_schema_migrations` unreleased row.

Implements: `Clone`, `Debug`, `Eq`, `PartialEq`

### InProcessSqlite

```rust
pub struct InProcessSqlite {
    /* private fields */
}
```

Runs compiled named plans on a local SeaORM SQLite connection.

Implements: `Clone`, `Debug`, `NamedAtomicRunner`

### PluginMigrationHistory

```rust
pub struct PluginMigrationHistory {
    pub entries: Vec<PluginJournalEntry>,
}
```

Ordered durable plugin migration history.

**Fields**

- `entries`: Journal rows in ordinal order.

#### PluginMigrationHistory::digest

```rust
pub fn digest(&self) -> String
```

SHA-256 digest of the complete ordered `(migration_id, checksum)` list.

#### PluginMigrationHistory::len

```rust
pub fn len(&self) -> usize
```

Number of applied journal rows.

#### PluginMigrationHistory::is_empty

```rust
pub fn is_empty(&self) -> bool
```

True when no plugin migrations have been applied.

#### PluginMigrationHistory::display

```rust
pub fn display(&self) -> String
```

Operator-facing history label (`history@{n}+{digest}`).

Implements: `Clone`, `Debug`, `Default`, `Eq`, `PartialEq`

### PluginMigrationSequence

```rust
pub struct PluginMigrationSequence {
    pub migrations: Vec<ProvenPluginMigration>,
}
```

Proven, ordered plugin migration sequence.

**Fields**

- `migrations`: Registration-order migrations with checksums.

Implements: `Clone`, `Debug`, `Eq`, `PartialEq`

### ProvenPluginMigration

```rust
pub struct ProvenPluginMigration {
    pub id: String,
    pub operations: Vec<PluginMigrationOp>,
    pub checksum: String,
}
```

One proven registered migration.

**Fields**

- `id`: Opaque plugin-chosen identity.
- `operations`: Ordered operations.
- `checksum`: Deterministic checksum of `(id, classified ops)`.

Implements: `Clone`, `Debug`, `Eq`, `PartialEq`

### AccountLinkRecord

```rust
pub struct AccountLinkRecord {
    pub id: i64,
    pub identity_id: i64,
    pub plugin_key: String,
    pub account_id: String,
    pub source: String,
    pub created_at: DateTime<Utc>,
}
```

Link between a portal identity and a bookstore account.

**Fields**

- `id`: Surrogate primary key assigned by the database.
- `identity_id`: Foreign key to `portal_identities.id`.
- `plugin_key`: Provenance-qualified PluginKey that owns the account.
- `account_id`: Store or operator account id this row belongs to.
- `source`: Display alias of the owning plugin (`audible`, `libro`, …).
- `created_at`: RFC 3339 timestamp when the row was inserted.

Implements: `Clone`, `Debug`, `Deserialize<'de>`, `Serialize`

### AccountRecord

```rust
pub struct AccountRecord {
    pub id: i64,
    pub plugin_key: String,
    pub account_id: String,
    pub source: String,
    pub marketplace: String,
    pub label: Option<String>,
    pub scan_enabled: bool,
    pub connection_status: String,
    pub created_at: DateTime<Utc>,
    pub updated_at: DateTime<Utc>,
}
```

Account row stored in the Bookclerk DB.

**Fields**

- `id`: Surrogate primary key assigned by the database.
- `plugin_key`: Provenance-qualified PluginKey that owns this account. Any instance
  of that plugin, on any node, may act for it.
- `account_id`: Store or operator account id, unique within one plugin key.
- `source`: Display alias of the owning plugin (`audible`, `libro`, …).
- `marketplace`: Store marketplace / locale code (for example `us`, `uk`).
- `label`: Optional operator-facing display label.
- `scan_enabled`: When nonzero/true, scheduled scans include this account.
- `connection_status`: `active` or `revoked` (credentials and holdings removed; books retained).
- `created_at`: RFC 3339 timestamp when the row was inserted.
- `updated_at`: RFC 3339 timestamp when the row was last modified.

Implements: `Clone`, `Debug`, `Deserialize<'de>`, `Serialize`

### AcquireStatusCounts

```rust
pub struct AcquireStatusCounts {
    pub total: i64,
    pub not_acquired: i64,
    pub queued: i64,
    pub downloading: i64,
    pub acquired: i64,
    pub error: i64,
}
```

Held work counts per [`AcquireStatus`](#acquirestatus), read in one grouped query
(see `LibraryStore::count_works_by_status`).

**Fields**

- `total`: Every work, including any with a status this build does not know.
- `not_acquired`: Works in [`AcquireStatus::NotAcquired`](#acquirestatus).
- `queued`: Works in [`AcquireStatus::Queued`](#acquirestatus).
- `downloading`: Works in [`AcquireStatus::Downloading`](#acquirestatus).
- `acquired`: Works in [`AcquireStatus::Acquired`](#acquirestatus).
- `error`: Works in [`AcquireStatus::Error`](#acquirestatus).

Implements: `Clone`, `Copy`, `Debug`, `Default`, `Eq`, `PartialEq`

### BranchRequestTarget

```rust
pub struct BranchRequestTarget {
    pub work_id: String,
    pub store: String,
    pub marketplace: String,
    pub product_id: String,
}
```

What a [`JobKind::BranchRequest`](#jobkind) asks its branch for: a work of this
library, and the store listing the branch answered for it
(`control_plane::catalog_lookup`). The branch is told the work by its
identifiers, never by this id.

**Fields**

- `work_id`: This library's work id.
- `store`: Store of the branch listing, as its plain id (`audible`).
- `marketplace`: Lower-cased marketplace of the listing; empty when the store has
  none.
- `product_id`: The store's product id of the listing.

Implements: `Clone`, `Debug`, `Deserialize<'de>`, `Eq`, `PartialEq`, `Serialize`

### ClaimTicketRecord

```rust
pub struct ClaimTicketRecord {
    pub id: i64,
    pub token_hash: String,
    pub identity_id: Option<i64>,
    pub expires_at: DateTime<Utc>,
    pub redeemed_at: Option<DateTime<Utc>>,
    pub created_by: String,
    pub created_at: DateTime<Utc>,
}
```

Claim ticket metadata (token plaintext is never stored).

**Fields**

- `id`: Surrogate primary key assigned by the database.
- `token_hash`: SHA-256 hex digest of the opaque token (plaintext never stored).
- `identity_id`: Foreign key to `portal_identities.id`.
- `expires_at`: RFC 3339 expiry for the ticket, session, or code.
- `redeemed_at`: RFC 3339 time when the ticket/invite was redeemed, if any.
- `created_by`: Actor that created the row (user id or operator label).
- `created_at`: RFC 3339 timestamp when the row was inserted.

Implements: `Clone`, `Debug`, `Deserialize<'de>`, `Serialize`

### DomainEventRecord

```rust
pub struct DomainEventRecord {
    pub id: String,
    pub event_type: String,
    pub schema_version: i64,
    pub occurred_at: DateTime<Utc>,
    pub account_id: String,
    pub source: String,
    pub correlation_id: String,
    pub causation_id: String,
    pub dedup_key: String,
    pub payload: String,
    pub ordering_key: String,
    pub dispatch_state: String,
    pub created_at: DateTime<Utc>,
    pub wake_pending: bool,
    pub queue_name: String,
    pub content_type: String,
    pub deliver_after: Option<DateTime<Utc>>,
}
```

Immutable outbox envelope.

**Fields**

- `id`: Event id.
- `event_type`: Event type.
- `schema_version`: Payload schema version.
- `occurred_at`: Occurrence time.
- `account_id`: Tenant / account id.
- `source`: Producer plugin id (`audible`, …); empty when unknown.
- `correlation_id`: Trace correlation id.
- `causation_id`: Causing id.
- `dedup_key`: Producer idempotency key.
- `payload`: Bounded JSON payload.
- `ordering_key`: Producer FIFO key copied onto each delivery.
- `dispatch_state`: `pending` or `dispatched`.
- `created_at`: Insert time.
- `wake_pending`: True until a wake pass for this event has completed.
- `queue_name`: Queue this row belongs to (`core.events` for domain events).
- `content_type`: Body encoding of `payload` (`json`, `text`, `bytes`).
- `deliver_after`: Earliest delivery time for a delayed queue message.

#### DomainEventRecord::is_queue_message

```rust
pub fn is_queue_message(&self) -> bool
```

True for a plugin-owned queue message (not a domain event).

Implements: `Clone`, `Debug`, `Deserialize<'de>`, `Serialize`

### EmbeddingRecord

```rust
pub struct EmbeddingRecord {
    pub id: i64,
    pub target_kind: String,
    pub target_id: String,
    pub model: String,
    pub dims: i64,
    pub text_hash: String,
    pub created_at: DateTime<Utc>,
    pub updated_at: DateTime<Utc>,
}
```

Stored embedding vector metadata (blob fetched separately when needed).

**Fields**

- `id`: Surrogate primary key assigned by the database.
- `target_kind`: Embedding target kind (`book`, `work`, …).
- `target_id`: Id of the embedded target within `target_kind`.
- `model`: Embedding model identifier used to produce `vector`.
- `dims`: Dimensionality of the stored embedding vector.
- `text_hash`: Hash of the text that was embedded (skip re-embed when unchanged).
- `created_at`: RFC 3339 timestamp when the row was inserted.
- `updated_at`: RFC 3339 timestamp when the row was last modified.

Implements: `Clone`, `Debug`, `Deserialize<'de>`, `Serialize`

### EnqueueJobSpec

```rust
pub struct EnqueueJobSpec {
    pub kind: JobKind,
    pub payload: JobPayload,
    pub priority: i64,
    pub max_attempts: i64,
    pub max_pending: i64,
    pub run_after: Option<DateTime<Utc>>,
}
```

Admission request for [`crate::LibraryStore::enqueue_job`](#librarystoreenqueue_job).

**Fields**

- `kind`: Job kind to run.
- `payload`: Kind-specific filters and trigger metadata.
- `priority`: Higher values are claimed first (default 0).
- `max_attempts`: Maximum claims before a failure is terminal.
- `max_pending`: Global cap on pending+running rows; enqueue fails when at the cap.
- `run_after`: Optional delay before the job becomes claimable.

Implements: `Clone`, `Debug`

### EventCatalogSubscription

```rust
pub struct EventCatalogSubscription {
    pub event_type: String,
    pub schema_versions: Vec<u32>,
    pub supports_suspend: bool,
    pub resource_class: String,
    pub filter: Option<Value>,
    pub queue: String,
    pub max_retries: Option<u32>,
    pub dead_letter_queue: Option<String>,
    pub retry_delay_secs: u32,
    pub message_retention_secs: Option<u32>,
}
```

One `[capabilities.events.subscriptions]` row persisted in `event_subscriber_nodes`.

**Fields**

- `event_type`: Versioned event type (`book_acquired`, …).
- `schema_versions`: Schema versions this guest can consume (default `[1]`).
- `supports_suspend`: Whether `EventResult::Suspended` is supported for this type.
- `resource_class`: Concurrency class (default `"network"`).
- `filter`: Optional host-owned payload object filter (top-level key equality).
- `queue`: Queue this subscription consumes; empty means the domain-event bus.
  A plugin-owned queue matches by name only (no type, schema, or filter).
- `max_retries`: Redeliveries before dead-lettering; `None` uses the host default.
- `dead_letter_queue`: Queue that receives a message once `max_retries` is exhausted.
- `retry_delay_secs`: Seconds a retried message waits before redelivery.
- `message_retention_secs`: Seconds a delivery may wait, due but never leased, before the host
  expires it (the consumer's `message_retention_period`). `None` uses
  [`EVENT_DELIVERY_DEFAULT_RETENTION_SECS`](#event_delivery_default_retention_secs).

#### EventCatalogSubscription::new

```rust
pub fn new(event_type: impl Into<String>, schema_versions: Vec<u32>) -> Self
```

Build a type+schema subscription with default resource class and no filter.

#### EventCatalogSubscription::queue

```rust
pub fn queue(queue: impl Into<String>) -> Self
```

Build a plugin-owned queue subscription (`[[queues.consumers]]`).

#### EventCatalogSubscription::queue_name

```rust
pub fn queue_name(&self) -> &str
```

Queue this subscription consumes (`core.events` when unset).

#### EventCatalogSubscription::is_queue

```rust
pub fn is_queue(&self) -> bool
```

True for a plugin-owned queue subscription (not the domain-event bus).

#### EventCatalogSubscription::max_attempts

```rust
pub fn max_attempts(&self) -> Option<u32>
```

`max_attempts` for deliveries created from this subscription.

Implements: `Clone`, `Debug`, `Deserialize<'de>`, `PartialEq`, `Serialize`

### EventDeliveryFence

```rust
pub struct EventDeliveryFence {
    pub delivery_id: String,
    pub owner: String,
    pub generation: i64,
}
```

Claim fence for an event delivery (mirrors [`JobFence`](#jobfence)).

**Fields**

- `delivery_id`: Delivery id that was claimed.
- `owner`: Worker id stored in `lease_owner`.
- `generation`: `lease_generation` assigned by the successful claim.

Implements: `Clone`, `Debug`, `Deserialize<'de>`, `Serialize`

### EventDeliveryMetrics

```rust
pub struct EventDeliveryMetrics {
    pub pending: i64,
    pub running: i64,
    pub suspended: i64,
    pub dead_letter: i64,
    pub acked: i64,
    pub oldest_pending_age_secs: Option<i64>,
    pub retries_total: i64,
    pub suspensions_total: i64,
    pub dead_letters_total: i64,
    pub dispatch_latency_ms_avg: Option<i64>,
    pub handler_latency_ms_avg: Option<i64>,
}
```

Operator-visible delivery-queue counters for `GET /api/status`.

**Fields**

- `pending`: `pending` rows with no checkpoint (not suspended).
- `running`: Currently claimed rows.
- `suspended`: `pending` rows that hold a suspend checkpoint.
- `dead_letter`: Dead-lettered rows.
- `acked`: Successfully acked rows.
- `oldest_pending_age_secs`: Age in seconds of the oldest `pending` row, when any exist.
- `retries_total`: Durable retry outcomes (including reclaim-as-retry).
- `suspensions_total`: Durable suspend outcomes.
- `dead_letters_total`: Durable transitions into `dead_letter`.
- `dispatch_latency_ms_avg`: Average first-dispatch latency in milliseconds, when sampled.
- `handler_latency_ms_avg`: Average `onEvent` handler duration in milliseconds, when sampled.

Implements: `Clone`, `Debug`, `Default`, `Deserialize<'de>`, `Eq`, `PartialEq`, `Serialize`

### EventDeliveryRecord

```rust
pub struct EventDeliveryRecord {
    pub id: String,
    pub event_id: String,
    pub plugin_id: String,
    pub idempotency_key: String,
    pub state: String,
    pub attempt_count: i64,
    pub max_attempts: i64,
    pub lease_owner: Option<String>,
    pub lease_expires_at: Option<DateTime<Utc>>,
    pub lease_generation: i64,
    pub run_after: DateTime<Utc>,
    pub invocation_sequence: i64,
    pub resume_pending: bool,
    pub checkpoint_json: Option<String>,
    pub checkpoint_schema_version: i64,
    pub ordering_key: String,
    pub outcome: Option<String>,
    pub error_message: Option<String>,
    pub created_at: DateTime<Utc>,
    pub updated_at: DateTime<Utc>,
    pub cancel_requested: bool,
    pub resource_class: String,
    pub wake_event_type: String,
    pub wake_filter_json: String,
    pub wake_grants_json: String,
    pub queue_name: String,
}
```

Per-subscriber delivery row.

**Fields**

- `id`: Delivery id (`{event_id}:{plugin_id}`).
- `event_id`: Parent event id.
- `plugin_id`: Subscriber plugin id.
- `idempotency_key`: Stable idempotency key.
- `state`: Lifecycle state.
- `attempt_count`: Claim count (not incremented on resume).
- `max_attempts`: Max claims before dead-letter.
- `lease_owner`: Current lease owner.
- `lease_expires_at`: Lease expiry.
- `lease_generation`: Fence generation.
- `run_after`: Earliest claim time.
- `invocation_sequence`: Resume ordinal.
- `resume_pending`: Next claim should resume rather than increment attempt.
- `checkpoint_json`: Suspend checkpoint JSON.
- `checkpoint_schema_version`: Checkpoint schema version.
- `ordering_key`: FIFO key.
- `outcome`: Terminal outcome.
- `error_message`: Operator-facing error.
- `created_at`: Insert time.
- `updated_at`: Last-modified time.
- `cancel_requested`: Cooperative cancel flag for a running delivery.
- `resource_class`: Concurrency class (`network` today).
- `wake_event_type`: Event type that can wake this suspended delivery; empty = timestamp-only.
- `wake_filter_json`: Host-owned payload object filter JSON for event-wake; empty = type only.
- `wake_grants_json`: Host-derived wake grants JSON; empty falls back to type + `wake_filter_json`.
- `queue_name`: Queue of the parent event, copied at dispatch.

#### EventDeliveryRecord::fence

```rust
pub fn fence(&self) -> EventDeliveryFence
```

Fence for heartbeat / complete / fail / suspend.

Implements: `Clone`, `Debug`, `Deserialize<'de>`, `Serialize`

### EventStreamPage

```rust
pub struct EventStreamPage {
    pub events: Vec<DomainEventRecord>,
    pub last_scanned: Option<(String, String)>,
    pub more: bool,
    pub expired_event_ids: Vec<String>,
    pub truncated: bool,
}
```

One page of [`EventStreamRead`](#eventstreamread).

**Fields**

- `events`: Matching events, oldest first.
- `last_scanned`: Stored `(created_at, id)` of the last row scanned, the next page's
  start; `None` when nothing was scanned.
- `more`: True when more events may follow (the page filled or the scan budget
  ran out).
- `expired_event_ids`: Ids of returned events whose delivery to one of the subscriber ids
  expired unleased.
- `truncated`: True when the start position is older than the retention window.

Implements: `Clone`, `Debug`, `Default`

### EventStreamRead

```rust
pub struct EventStreamRead {
    pub start: EventStreamStart,
    pub subscriptions: Vec<EventCatalogSubscription>,
    pub account_id: Option<String>,
    pub plugin_key: String,
    pub retention_days: u64,
    pub settle_secs: u64,
    pub limit: usize,
    pub now: DateTime<Utc>,
}
```

One page request over the domain-event outbox (`EventPublisher.read`).

**Fields**

- `start`: Where the read starts.
- `subscriptions`: The reader's consented subscriptions, already narrowed to the types it
  asked for. An event is returned when any of them matches it.
- `account_id`: `Some` limits the read to one account's events; `None` reads every
  account, as delivery does for a host-wide invocation.
- `plugin_key`: The reader's PluginKey. Its deliveries are filed under this key and
  under each of its instance ids; they report which returned events it
  missed. Empty reports none.
- `retention_days`: `[events].retention_days`: nothing older is returned.
- `settle_secs`: Events stored less than this many seconds before `now` are left for a
  later read ([`crate::EVENT_STREAM_SETTLE_SECS`](#event_stream_settle_secs) in production), so a
  publication that commits behind a newer one is never skipped.
- `limit`: Most events to return (at least one).
- `now`: Current time, which fixes the retention cutoff.

Implements: `Clone`, `Debug`

### EventSubscriber

```rust
pub struct EventSubscriber {
    pub plugin_id: String,
    pub resource_class: String,
    pub max_attempts: Option<u32>,
}
```

Subscriber snapshot used when creating deliveries for one event.

**Fields**

- `plugin_id`: Plugin id that declared a matching subscription.
- `resource_class`: Concurrency class copied onto the delivery row.
- `max_attempts`: Claims before the delivery dead-letters; `None` uses the host default
  ([`EVENT_DELIVERY_MAX_ATTEMPTS`](#event_delivery_max_attempts)). Derived from the consumer's
  `max_retries + 1`.

#### EventSubscriber::plugin

```rust
pub fn plugin(plugin_id: impl Into<String>) -> Self
```

Catalog match for `plugin_id` with the default `network` resource class.

#### EventSubscriber::max_attempts_or_default

```rust
pub fn max_attempts_or_default(&self) -> i64
```

Effective `max_attempts` for a new delivery row.

Implements: `Clone`, `Debug`, `Default`, `Deserialize<'de>`, `Eq`, `PartialEq`, `Serialize`

### EventSubscriberCatalogRecord

```rust
pub struct EventSubscriberCatalogRecord {
    pub plugin_id: String,
    pub subscriptions: Vec<EventCatalogSubscription>,
    pub enabled: bool,
    pub updated_at: DateTime<Utc>,
}
```

Library-authoritative catalog row after collapsing live nodes by `plugin_id`.

**Fields**

- `plugin_id`: Canonical PluginKey of the subscriber (not the display alias).
- `subscriptions`: Union of subscriptions from live enabled nodes.
- `enabled`: Whether any live node has this plugin enabled.
- `updated_at`: Latest heartbeat among contributing live rows.

Implements: `Clone`, `Debug`, `Deserialize<'de>`, `PartialEq`, `Serialize`

### EventSubscriberNodeRecord

```rust
pub struct EventSubscriberNodeRecord {
    pub node_id: String,
    pub plugin_id: String,
    pub subscriptions: Vec<EventCatalogSubscription>,
    pub enabled: bool,
    pub heartbeat_at: DateTime<Utc>,
}
```

Live per-node catalog row (`event_subscriber_nodes`).

**Fields**

- `node_id`: Daemon node id.
- `plugin_id`: Canonical PluginKey of the subscriber (not the display alias).
- `subscriptions`: Declared subscriptions on this node.
- `enabled`: Whether this node wants matching events for `plugin_id`.
- `heartbeat_at`: Last heartbeat from this node.

Implements: `Clone`, `Debug`, `Deserialize<'de>`, `PartialEq`, `Serialize`

### EventWakeGrant

```rust
pub struct EventWakeGrant {
    pub schema_versions: Vec<u32>,
    pub filter: Option<Value>,
}
```

Host-derived wake grant: schema versions plus an intersected payload filter.

**Fields**

- `schema_versions`: Schema versions this grant may observe.
- `filter`: Intersected payload object filter; `None` / empty object matches any payload.

Implements: `Clone`, `Debug`, `Deserialize<'de>`, `PartialEq`, `Serialize`

### GlobalQueueEntry

```rust
pub struct GlobalQueueEntry {
    pub work_key: String,
    pub work_id: String,
    pub edition_id: Option<String>,
    pub expression_id: Option<String>,
    pub title: String,
    pub authors: Option<String>,
    pub asin: Option<String>,
    pub isbn: Option<String>,
    pub cover_url: Option<String>,
    pub description: Option<String>,
    pub subtitle: Option<String>,
    pub narrators: Option<String>,
    pub series: Option<String>,
    pub series_index: Option<String>,
    pub publisher: Option<String>,
    pub length_minutes: Option<i64>,
    pub published_at: Option<String>,
    pub genres: Option<String>,
    pub language: Option<String>,
    pub store_editions: Vec<WishlistStoreEdition>,
    pub purchase_hints: Vec<WishlistPurchaseHint>,
    pub wish_count: i64,
    pub wishers: Vec<QueueWisher>,
    pub sample_uuids: Vec<String>,
    pub first_requested_at: DateTime<Utc>,
    pub last_requested_at: DateTime<Utc>,
}
```

Aggregated global request-queue entry (one work, many wishers).

**Fields**

- `work_key`: Bibliographic card key of the newest wish (see
  [`TitleRequestRecord::work_key`](#titlerequestrecord)); clients match cards on it.
- `work_id`: The work every wish of this entry is for: the queue groups by it.
- `edition_id`: The edition the newest wish naming one is for (`editions.id`); the
  entry's title, identifiers and cover describe that edition.
- `expression_id`: That edition's expression (`expressions.id`), when the library
  places it: store listings of the same expression are the same
  recording, so Discover prices the wish from them only.
- `title`: Display title of the work or edition.
- `authors`: Comma-separated or JSON author list from the storefront.
- `asin`: Amazon ASIN when known; otherwise null.
- `isbn`: ISBN-10/13 when known; otherwise null.
- `cover_url`: HTTPS URL for cover art when known.
- `description`: Blurb / synopsis text (may contain HTML).
- `subtitle`: Optional subtitle from bibliographic metadata.
- `narrators`: Comma-separated or JSON narrator list when present.
- `series`: Series name when the title belongs to a series.
- `series_index`: Position within the series (storefront string form).
- `publisher`: Publisher name from metadata enrichment or the storefront.
- `length_minutes`: Runtime in whole minutes when the storefront reports it.
- `published_at`: Publication date string from the storefront or enrichment.
- `genres`: Merged genre / category list for display.
- `language`: BCP-47 or storefront language code when known.
- `store_editions`: Distinct storefront editions contributing to this wishlist item.
- `purchase_hints`: Purchase URLs and prices observed per storefront.
- `wish_count`: Number of users who requested this title.
- `wishers`: Distinct people who have this title on an open wishlist.
- `sample_uuids`: Sample title-request UUIDs represented in this aggregate row.
- `first_requested_at`: Earliest request timestamp in this aggregate group.
- `last_requested_at`: Most recent request timestamp in this aggregate group.

#### GlobalQueueEntry::push_wisher

```rust
pub fn push_wisher(&mut self, wisher: QueueWisher)
```

Adds `wisher` when they are not already present (capped at 16).

Duplicate people keep the richer picture and name fields.

##### Arguments

* `wisher` - Person who has this title on an open wishlist.

Implements: `Clone`, `Debug`, `Deserialize<'de>`, `Serialize`

### JobClaimCursor

```rust
pub struct JobClaimCursor {
    pub priority: i64,
    pub created_at: String,
    pub id: String,
}
```

Where the next page of routed-claim candidates starts (keyset paging).

Candidates are ordered by `priority DESC, created_at ASC, id ASC`; the
cursor is the last row of the previous page, so rows that leave the
pending state between pages never shift the window.

**Fields**

- `priority`: Priority of the last row read.
- `created_at`: `created_at` (RFC 3339 text) of the last row read.
- `id`: Id of the last row read.

Implements: `Clone`, `Debug`, `Eq`, `PartialEq`

### JobClaimPage

```rust
pub struct JobClaimPage {
    pub jobs: Vec<JobRecord>,
    pub next: Option<JobClaimCursor>,
}
```

One page of routed-claim candidates.

**Fields**

- `jobs`: Readable, not cancel-requested pending jobs, best first.
- `next`: Where the next page starts, or `None` when no row remains.

Implements: `Clone`, `Debug`, `Default`

### JobFence

```rust
pub struct JobFence {
    pub job_id: String,
    pub owner: String,
    pub generation: i64,
}
```

Lease identity a worker must present to mutate a running job.

**Fields**

- `job_id`: Job id that was claimed.
- `owner`: Worker id stored in `lease_owner`.
- `generation`: `lease_generation` assigned by the successful claim.

Implements: `Clone`, `Debug`, `Eq`, `PartialEq`

### JobPayload

```rust
pub struct JobPayload {
    pub v: u32,
    pub account: Option<String>,
    pub title: Option<String>,
    pub plugin_instance_id: Option<String>,
    pub requires: Vec<String>,
    pub trigger: JobTrigger,
    pub integration_id: Option<String>,
    pub force: bool,
    pub plugin_id: Option<String>,
    pub source_key: Option<String>,
    pub dest_key: Option<String>,
    pub checkpoint: Option<JobCheckpoint>,
    pub invocation_sequence: Option<u32>,
    pub resume_pending: bool,
    pub resumed_generation: Option<i64>,
    pub pdf_only: bool,
    pub paced: bool,
    pub requires_unsettled_since_ms: Option<i64>,
    pub cron: Option<String>,
    pub scheduled_time_ms: Option<i64>,
    pub subject: Option<String>,
    pub node_id: Option<String>,
    pub branch_request: Option<BranchRequestTarget>,
    pub ingest_batch: Option<String>,
    pub convert: Option<OutputCopy>,
    pub notify_delivery: Option<String>,
}
```

JSON command envelope stored on a job row.

**Fields**

- `v`: Envelope schema version; must equal [`JOB_PAYLOAD_VERSION`](#job_payload_version).
- `account`: Optional account filter (`None` = all eligible accounts).
- `title`: Optional title filter (uuid / ASIN / ISBN / product id).
- `plugin_instance_id`: Pins the plugin instance that serves the job. `None` routes to the
  plugin's default instance on the claiming host.
- `requires`: Plugin instance ids the claiming node must run, with its guest up.
  
  Recorded at admission (the pinned instance and any node-bound
  destination). Empty means any node may claim the job. The routed claim
  (`control_plane::claim_routed_job`) skips a job whose instances are not
  deployed and observed up on the claiming node, and the unrouted
  [`crate::LibraryStore::claim_next_job`](#librarystoreclaim_next_job) never claims a job that has any.
- `trigger`: Admission source (`api` or `scheduler`).
- `integration_id`: Integration plugin id for [`JobKind::IntegrationScan`](#jobkind).
- `force`: When true, an integration scan asks the remote to rescan, and an
  acquire re-downloads titles already acquired (run only by nodes that
  hold the library database).
- `plugin_id`: Destination plugin id for [`JobKind::PluginCopy`](#jobkind).
- `source_key`: Source object key for [`JobKind::PluginCopy`](#jobkind).
- `dest_key`: Destination object key for [`JobKind::PluginCopy`](#jobkind).
- `checkpoint`: Bounded checkpoint restored after `bookclerk_plugin_abi::JobOutcome::Suspended`.
- `invocation_sequence`: Resume ordinal; distinct from failure [`Self`](#jobpayload) attempt_count on the job row.
- `resume_pending`: One-shot: the next claim is a suspend-resume and must not consume a failure attempt.
  
  Cleared when that claim is taken. A leftover [`Self::checkpoint`](#jobpayload) after a
  retryable failure must not keep later claims in the resume path.
- `resumed_generation`: The lease generation of the claim that took [`Self::resume_pending`]:
  set by the claim itself, so a release of that claim knows, from the
  library's own record and not the claimant's word, that it spent no
  attempt. A value from an earlier claim never matches a later fence.
- `pdf_only`: For [`JobKind::Acquire`]: fetch only each title's companion PDF.
  
  Run only by nodes that hold the library database; a Host is never
  handed such a job (`control_plane::node_may_claim_job`).
- `paced`: For [`JobKind::Acquire`]: the host started this acquire itself
  (auto-acquire, quality upgrades, acquire-all, a request from another
  library), so each download waits for its store account's pace
  (`crate::pacing`). An acquire a person starts by hand leaves it
  false: its downloads are counted but never held.
- `requires_unsettled_since_ms`: Unix milliseconds when the requirements sweep first found this pending
  job's [`Self::requires`](#jobpayload) unsettled: no node runs all of them, or one of
  them is no longer deployed anywhere.
  
  Cleared when they settle again. The sweep fails the job (or, when only
  a dropped instance is involved, rewrites it) once it has been unsettled
  for its grace period, so a placement move that briefly leaves no common
  node does not fail queued work (`control_plane::sweep_job_requirements`).
- `cron`: For [`JobKind::PluginScheduled`]: the normalized `[triggers] crons`
  schedule that fired.
- `scheduled_time_ms`: For [`JobKind::PluginScheduled`]: the fire time the handler sees as
  `scheduledTime`, in Unix milliseconds (always a whole UTC minute).
- `subject`: For [`JobKind::DiscoverCandidates`](#jobkind) and [`JobKind::DiscoverFeed`]: the
  Discover subject key (`operator`, `user:<id>`, `portal:<id>`).
- `node_id`: The one node that may claim this job, for work on that node's own
  disk ([`JobKind::CacheSweep`](#jobkind), [`JobKind::SearchIndexRebuild`](#jobkind),
  [`JobKind::IngestBatch`](#jobkind)).
  
  `None` lets any node that meets [`Self::requires`](#jobpayload) claim it. The
  routed claim (`control_plane::claim_routed_job`) skips a job pinned
  to another node, the unrouted [`crate::LibraryStore::claim_next_job`](#librarystoreclaim_next_job)
  never claims a pinned job, and the requirements sweep fails one whose
  node has stopped heartbeating (`control_plane::sweep_job_requirements`).
- `branch_request`: For [`JobKind::BranchRequest`]: the work asked for and the branch
  listing it was asked from.
- `ingest_batch`: For [`JobKind::IngestBatch`]: the `ingest_batches.id` to run.
- `convert`: For [`JobKind::Acquire`]: instead of downloading, add this copy to
  the title's stored edition, made from the copy the library already
  holds (an administrator's convert). No store is asked, so the job is
  never paced; [`Self::force`](#jobpayload) makes the copy again when it exists.
  
  Run only by nodes that hold the library database, since the copy is
  made from the edition's recorded files.
- `notify_delivery`: For [`JobKind::NotifySend`]: the `notify_deliveries.id` to send.

Implements: `Clone`, `Debug`, `Default`, `Deserialize<'de>`, `Eq`, `PartialEq`, `Serialize`

### JobRecord

```rust
pub struct JobRecord {
    pub id: String,
    pub kind: JobKind,
    pub state: JobState,
    pub priority: i64,
    pub resource_class: JobResourceClass,
    pub payload: JobPayload,
    pub progress: Option<String>,
    pub attempt_count: i64,
    pub max_attempts: i64,
    pub run_after: DateTime<Utc>,
    pub lease_owner: Option<String>,
    pub lease_expires_at: Option<DateTime<Utc>>,
    pub dedup_key: String,
    pub error_kind: Option<String>,
    pub error_message: Option<String>,
    pub cancel_requested: bool,
    pub created_at: DateTime<Utc>,
    pub updated_at: DateTime<Utc>,
    pub started_at: Option<DateTime<Utc>>,
    pub finished_at: Option<DateTime<Utc>>,
    pub lease_generation: i64,
}
```

Durable job row returned to the daemon API and worker.

**Fields**

- `id`: Stable job id (`scan-{uuid}`, …).
- `kind`: Job kind (`scan`, `acquire`, `listen_sync`).
- `state`: Lifecycle state.
- `priority`: Higher values are claimed first.
- `resource_class`: Concurrency class.
- `payload`: Kind-specific filters and trigger metadata.
- `progress`: Human-readable progress string.
- `attempt_count`: Number of times a worker has claimed this row.
- `max_attempts`: Maximum claims before a failure becomes terminal.
- `run_after`: Earliest time a pending job may be claimed.
- `lease_owner`: Worker id that currently holds the lease.
- `lease_expires_at`: Lease expiry; stale running rows are reclaimed after this.
- `dedup_key`: Idempotency key among pending/running rows.
- `error_kind`: Structured error kind when failed or cancelled.
- `error_message`: Operator-facing error text.
- `cancel_requested`: Cooperative cancel flag.
- `created_at`: When the row was inserted.
- `updated_at`: When the row was last modified.
- `started_at`: When a worker first claimed the row.
- `finished_at`: When the row reached a terminal state.
- `lease_generation`: Per-claim generation used to fence heartbeat and finalization.

#### JobRecord::fence

```rust
pub fn fence(&self) -> Option<JobFence>
```

Lease identity for heartbeat and finalization, when this row is claimed.

Implements: `Clone`, `Debug`, `Deserialize<'de>`, `Serialize`

### JobTempPath

```rust
pub struct JobTempPath {
    pub id: i64,
    pub job_id: String,
    pub path: String,
    pub created_at: DateTime<Utc>,
    pub reserved_bytes: u64,
}
```

Scratch path registered against a job for crash cleanup.

**Fields**

- `id`: Surrogate primary key.
- `job_id`: Owning job id.
- `path`: Absolute filesystem path.
- `created_at`: When the path was registered.
- `reserved_bytes`: Bytes reserved against the job temp quota for this path.

Implements: `Clone`, `Debug`, `Deserialize<'de>`, `Serialize`

### ListeningProgressRecord

```rust
pub struct ListeningProgressRecord {
    pub id: i64,
    pub identity_id: Option<i64>,
    pub provider: String,
    pub external_user_id: String,
    pub work_id: String,
    pub edition_id: Option<String>,
    pub external_item_id: String,
    pub progress: Option<f64>,
    pub current_time_seconds: Option<f64>,
    pub duration_seconds: Option<f64>,
    pub is_finished: bool,
    pub last_listened_at: Option<DateTime<Utc>>,
    pub updated_at: DateTime<Utc>,
}
```

Listening progress snapshot from an external player (e.g. AudioBookshelf).

**Fields**

- `id`: Surrogate primary key assigned by the database.
- `identity_id`: Foreign key to `portal_identities.id`.
- `provider`: External identity or integration provider id (for example ABS).
- `external_user_id`: User id at the external provider.
- `work_id`: The work the item resolved to through its identifiers.
- `edition_id`: The edition the item's identifiers name, when exactly one does.
- `external_item_id`: Provider-native listening-progress item id.
- `progress`: Fractional progress 0.0–1.0 when the provider reports it.
- `current_time_seconds`: Current playback position within the title, in seconds.
- `duration_seconds`: Total duration in seconds when known.
- `is_finished`: Whether the listener marked the title finished (0/1 or bool).
- `last_listened_at`: RFC 3339 time of the last playback update.
- `updated_at`: RFC 3339 timestamp when the row was last modified.

Implements: `Clone`, `Debug`, `Deserialize<'de>`, `Serialize`

### OidcClientRecord

```rust
pub struct OidcClientRecord {
    pub id: i64,
    pub client_id: String,
    pub client_secret_hash: Option<String>,
    pub redirect_uris: Vec<String>,
    pub name: Option<String>,
    pub issue_refresh_token: bool,
    pub allowed_scopes: Vec<String>,
    pub enabled: bool,
    pub plugin_id: Option<String>,
    pub created_at: String,
}
```

Registered OIDC authorization-server client (Bookclerk as IdP).

**Fields**

- `id`: Surrogate primary key.
- `client_id`: Public client_id used at authorize / token.
- `client_secret_hash`: Hash of the client secret; `None` for public PKCE clients.
- `redirect_uris`: Allowed OAuth redirect URIs.
- `name`: Operator-facing display name.
- `issue_refresh_token`: When true, token responses include a refresh token.
- `allowed_scopes`: Scopes this client may be granted (`openid`, `profile`, `email`, …).
- `enabled`: When true, authorize and token endpoints accept this client.
- `plugin_id`: Plugin id that owns this client; `None` for operator-created clients.
- `created_at`: RFC 3339 timestamp when the row was inserted.

#### OidcClientRecord::has_secret

```rust
pub fn has_secret(&self) -> bool
```

True when a client secret hash is stored (confidential client).

#### OidcClientRecord::is_plugin_provided

```rust
pub fn is_plugin_provided(&self) -> bool
```

True when a plugin owns this client's identity and redirect URIs.

Implements: `Clone`, `Debug`

### OperatorSessionRecord

```rust
pub struct OperatorSessionRecord {
    pub id: i64,
    pub expires_at: DateTime<Utc>,
    pub created_at: DateTime<Utc>,
    pub last_used_at: Option<DateTime<Utc>>,
    pub elevated_from_user_id: Option<i64>,
    pub impersonating_user_id: Option<i64>,
    pub user_agent: Option<String>,
    pub device_type: Option<String>,
    pub client_label: Option<String>,
}
```

Durable operator session metadata (hashed token is never exposed).

**Fields**

- `id`: Surrogate primary key assigned by the database.
- `expires_at`: RFC 3339 expiry for the ticket, session, or code.
- `created_at`: RFC 3339 timestamp when the row was inserted.
- `last_used_at`: RFC 3339 time of the last authenticated use of this session.
- `elevated_from_user_id`: User id that elevated into this operator session, if any.
- `impersonating_user_id`: User id being impersonated by this operator session, if any.
- `user_agent`: Raw User-Agent captured at session mint (optional).
- `device_type`: Best-effort device class (`desktop` / `mobile` / `tablet` / `api`).
- `client_label`: Best-effort OS / client label (`Windows`, `Android`, `API`, …).

Implements: `Clone`, `Debug`, `Deserialize<'de>`, `Serialize`

### PendingWakeProgress

```rust
pub struct PendingWakeProgress {
    pub claimed: u32,
    pub still_pending: bool,
}
```

How far one claimed wake tick got.

**Fields**

- `claimed`: Events this tick claimed and processed one delivery page for.
- `still_pending`: True when another tick should run (saved cursor or remaining flags).

Implements: `Clone`, `Copy`, `Debug`, `Default`, `Eq`, `PartialEq`

### PortalIdentity

```rust
pub struct PortalIdentity {
    pub id: i64,
    pub provider: String,
    pub external_user_id: String,
    pub label: Option<String>,
    pub user_id: Option<i64>,
    pub created_at: DateTime<Utc>,
    pub picture_url: Option<String>,
}
```

Portal identity bound to an external provider user (e.g. ABS).

**Fields**

- `id`: Surrogate primary key assigned by the database.
- `provider`: External identity or integration provider id (for example ABS).
- `external_user_id`: User id at the external provider.
- `label`: Optional operator-facing display label.
- `user_id`: First-party user this external identity is linked to (Phase 1).
- `created_at`: RFC 3339 timestamp when the row was inserted.
- `picture_url`: Last HTTPS profile picture URL from the identity provider, when any.

Implements: `Clone`, `Debug`, `Deserialize<'de>`, `Serialize`

### PortalSessionRecord

```rust
pub struct PortalSessionRecord {
    pub id: i64,
    pub token_hash: String,
    pub created_at: String,
    pub expires_at: String,
    pub last_used_at: Option<String>,
    pub user_agent: Option<String>,
    pub device_type: Option<String>,
    pub client_label: Option<String>,
    pub kind: PortalSessionKind,
}
```

Portal session row for session lists (includes hash for current-session match).

**Fields**

- `id`: Surrogate primary key assigned by the database.
- `token_hash`: SHA-256 hex of the session token (never serialized to clients).
- `created_at`: RFC 3339 created timestamp.
- `expires_at`: RFC 3339 expiry timestamp.
- `last_used_at`: RFC 3339 last-used timestamp, when known.
- `user_agent`: Raw User-Agent captured at session mint (optional).
- `device_type`: Best-effort device class.
- `client_label`: Best-effort OS / client label; for a library card, the name the
  clerk gave itself.
- `kind`: Where the token is presented.

Implements: `Clone`, `Debug`

### PublishDomainEventSpec

```rust
pub struct PublishDomainEventSpec {
    pub id: String,
    pub event_type: String,
    pub schema_version: i64,
    pub account_id: String,
    pub source: String,
    pub correlation_id: String,
    pub causation_id: String,
    pub dedup_key: String,
    pub payload: String,
    pub ordering_key: String,
    pub queue_name: String,
    pub content_type: String,
    pub delay_seconds: u32,
}
```

Producer-side domain event (or queue message) to persist in the outbox.

A domain event leaves `queue_name` empty (the bus, [`CORE_EVENTS_QUEUE`](#core_events_queue))
and sets `event_type`. A queue message names its queue and leaves
`event_type` empty; `content_type` says how `payload` is encoded and
`delay_seconds` holds its deliveries back.

**Fields**

- `id`: Stable event id (UUID). Generated when empty.
- `event_type`: Event type (`book_acquired`); empty for queue messages.
- `schema_version`: Payload schema version.
- `account_id`: Tenant / account id.
- `source`: Producer plugin id (`audible`, …); empty when unknown.
- `correlation_id`: Trace correlation id.
- `causation_id`: Causing event or job id.
- `dedup_key`: Unique with `event_type` (duplicate publishes coalesce).
- `payload`: Bounded JSON payload (never media bytes).
- `ordering_key`: FIFO key copied onto each delivery (`book uuid` for `book_acquired`).
- `queue_name`: Queue name; empty means the domain-event bus ([`CORE_EVENTS_QUEUE`](#core_events_queue)).
- `content_type`: Body encoding (`json`, `text`, `bytes`); empty means `json`.
- `delay_seconds`: Seconds before the first delivery may be claimed (queue messages).

Implements: `Clone`, `Debug`, `Default`, `Deserialize<'de>`, `Serialize`

### QueueWisher

```rust
pub struct QueueWisher {
    pub user_id: Option<i64>,
    pub identity_id: Option<i64>,
    pub display_name: Option<String>,
    pub login_name: Option<String>,
    pub operator: bool,
    pub has_avatar: bool,
    pub avatar_source: Option<String>,
    pub gravatar_hash: Option<String>,
    pub picture_url: Option<String>,
    pub picture_path: Option<String>,
}
```

Compact wisher shown on the global queue (avatar + label).

**Fields**

- `user_id`: First-party `users.id` when the portal identity is linked.
- `identity_id`: Portal identity that created the wish (dedupe for unlinked identities).
- `display_name`: Display name, identity label, or `"Operator"`.
- `login_name`: Local login name when the first-party user has one.
- `operator`: True when the wish came from the operator token (no portal user).
- `has_avatar`: True when an uploaded avatar file exists (filled by the daemon).
- `avatar_source`: Explicit picture choice when the user has set one.
- `gravatar_hash`: SHA-256 hex of the contact email for Gravatar.
- `picture_url`: HTTPS picture from the wishing identity, when any.
- `picture_path`: Same-origin 128 px copy of `picture_url` for the web app to show
  (filled by the daemon; never stored).

#### QueueWisher::same_person

```rust
pub fn same_person(&self, other: &Self) -> bool
```

Returns true when both rows represent the same person for aggregation.

##### Arguments

* `other` - Candidate wisher to compare.

##### Returns

True for two operator wishes, the same `user_id`, or the same unlinked identity.

Implements: `Clone`, `ComposeSchema`, `Debug`, `Default`, `Deserialize<'de>`, `Eq`, `PartialEq`, `Serialize`, `ToSchema`

### SecurityAuditEvent

```rust
pub struct SecurityAuditEvent {
    pub id: i64,
    pub at: DateTime<Utc>,
    pub actor: String,
    pub action: String,
    pub detail_json: Option<String>,
}
```

Security audit event (elevate / impersonate / login / provision).

**Fields**

- `id`: Surrogate primary key assigned by the database.
- `at`: RFC 3339 timestamp of the audit event.
- `actor`: Actor label (user id, operator, or system).
- `action`: Audit action verb (for example `login`, `rotate_token`).
- `detail_json`: JSON object with structured event details (no secrets).

Implements: `Clone`, `Debug`, `Deserialize<'de>`, `Serialize`

### StoredPasskey

```rust
pub struct StoredPasskey {
    pub id: i64,
    pub credential_id: String,
    pub passkey_json: String,
    pub name: Option<String>,
}
```

Stored WebAuthn credential (passkey JSON plus the label shown in Settings).

**Fields**

- `id`: Surrogate primary key.
- `credential_id`: Base64url credential id.
- `passkey_json`: Serialized `webauthn_rs` passkey JSON.
- `name`: Label chosen at registration; `None` for rows created before names existed.

Implements: `Clone`, `Debug`

### TasteHolder

```rust
pub struct TasteHolder {
    pub plugin_key: String,
    pub account_id: String,
    pub source: String,
}
```

A storefront account that holds a work, as a [`TasteTitle`](#tastetitle) lists it.

**Fields**

- `plugin_key`: The account's plugin key.
- `account_id`: The account id at that plugin.
- `source`: The account's storefront id (`accounts.source`), lowercase.

Implements: `Clone`, `Debug`, `Deserialize<'de>`, `Eq`, `PartialEq`, `Serialize`

### TasteTitle

```rust
pub struct TasteTitle {
    pub work_id: String,
    pub edition_id: String,
    pub plugin_key: String,
    pub source: String,
    pub product_id: String,
    pub marketplace: String,
    pub asin: Option<String>,
    pub isbn: Option<String>,
    pub title: String,
    pub subtitle: Option<String>,
    pub authors: Option<String>,
    pub narrators: Option<String>,
    pub series: Option<String>,
    pub series_index: Option<String>,
    pub series_store_id: Option<String>,
    pub publisher: Option<String>,
    pub categories: Option<String>,
    pub subjects: Option<String>,
    pub description: Option<String>,
    pub language: Option<String>,
    pub cover_url: Option<String>,
    pub length_minutes: Option<i64>,
    pub published_at: Option<String>,
    pub is_abridged: bool,
    pub is_podcast: bool,
    pub purchased_at: Option<DateTime<Utc>>,
    pub acquired: bool,
    pub holders: Vec<TasteHolder>,
    pub is_finished: bool,
    pub rating_overall: Option<f32>,
    pub rating_performance: Option<f32>,
    pub rating_story: Option<f32>,
    pub tags: Option<String>,
}
```

One work the library holds, as Discover reads it for taste and seeds.

Bibliographic fields come from the work (credits, series, subjects,
description) and from a representative edition and store listing: one
whose edition has files, else the most recently bought. The person's
fields (`is_finished`, ratings, `tags`) start empty and are filled from
`user_title_state` for whichever people the reader asks about.

**Fields**

- `work_id`: The work (`works.id`); the title's identity.
- `edition_id`: The representative edition (`editions.id`).
- `plugin_key`: Plugin key of the representative listing's store.
- `source`: Storefront id of that store (an account's `source`), lowercase.
- `product_id`: The representative listing's product id at its store.
- `marketplace`: The representative listing's stored marketplace.
- `asin`: The representative edition's ASIN, when it has one.
- `isbn`: The representative edition's ISBN, when it has one.
- `title`: The work's title.
- `subtitle`: The edition's subtitle, when known.
- `authors`: Names credited on the work (authors first), comma-separated.
- `narrators`: Narrators credited on the work's expressions, comma-separated.
- `series`: The first series the work belongs to.
- `series_index`: The work's position in that series as printed (`"3"`, `"2.5"`).
- `series_store_id`: The representative store's own id for the series, when it gave one.
- `publisher`: The edition's first credited publisher.
- `categories`: Store categories (serialized), from the listing or the work.
- `subjects`: Subject tags of the work (serialized), when enriched.
- `description`: The work's synopsis (may contain HTML).
- `language`: The expression's language, when known.
- `cover_url`: Cover art URL of the edition, else of the work.
- `length_minutes`: The edition's running time in minutes, when known.
- `published_at`: The edition's publication date as given, when known.
- `is_abridged`: Whether the expression is abridged.
- `is_podcast`: Whether the work is a podcast show or episode.
- `purchased_at`: The newest purchase date of the representative listing's holders.
- `acquired`: Whether the representative edition has files at a destination.
- `holders`: The accounts that hold any edition of the work, by plugin and id.
- `is_finished`: Whether the people asked about finished the work.
- `rating_overall`: Their mean overall rating (0 to 5), when any gave one.
- `rating_performance`: Their mean narration rating (0 to 5), when any gave one.
- `rating_story`: Their mean story rating (0 to 5), when any gave one.
- `tags`: Their tags, space-separated, each once.

Implements: `Clone`, `Debug`, `Deserialize<'de>`, `PartialEq`, `Serialize`

### TitleRequestRecord

```rust
pub struct TitleRequestRecord {
    pub id: i64,
    pub uuid: String,
    pub identity_id: Option<i64>,
    pub title: String,
    pub authors: Option<String>,
    pub asin: Option<String>,
    pub isbn: Option<String>,
    pub notes: Option<String>,
    pub status: RequestStatus,
    pub work_key: String,
    pub work_id: String,
    pub edition_id: Option<String>,
    pub resolved_edition_id: Option<String>,
    pub cover_url: Option<String>,
    pub cover_path: Option<String>,
    pub sources: Vec<TitleRequestSourceRecord>,
    pub description: Option<String>,
    pub subtitle: Option<String>,
    pub narrators: Option<String>,
    pub series: Option<String>,
    pub series_index: Option<String>,
    pub publisher: Option<String>,
    pub length_minutes: Option<i64>,
    pub published_at: Option<String>,
    pub genres: Option<String>,
    pub language: Option<String>,
    pub store_editions: Vec<WishlistStoreEdition>,
    pub purchase_hints: Vec<WishlistPurchaseHint>,
    pub created_at: DateTime<Utc>,
    pub updated_at: DateTime<Utc>,
}
```

Personal wishlist row (also contributes to the shared global queue while open).

**Fields**

- `id`: Surrogate primary key assigned by the database.
- `uuid`: Stable UUID for this row (API / foreign-key identity).
- `identity_id`: Foreign key to `portal_identities.id`.
- `title`: Display title: the title of the wish's work.
- `authors`: Comma-separated authors: the work's author credits in billing order.
- `asin`: Amazon ASIN from the wish's storefront sources, when one carries it.
- `isbn`: ISBN-10/13 from the wish's storefront sources, when one carries it.
- `notes`: Free-form operator or requester notes.
- `status`: Lifecycle status for the row (user, request, …).
- `work_key`: Bibliographic card key (`isbn:…` / `asin:…` / `soft:…`) clients use
  to match a wish to a catalog card. Derived from the work's title and
  authors and the sources' identifiers on every read, never stored.
- `work_id`: The work the wish is for; wishes for one work are one queue entry.
- `edition_id`: The edition the wish named, when the library knows it.
- `resolved_edition_id`: Edition (`editions.id`) the library holds once the wishlist item is fulfilled.
- `cover_url`: HTTPS URL for cover art when known.
- `cover_path`: Same-origin path of this cover's processed images, set by the daemon
  when `cover_url` is an `https` URL. Append `/128`
  (thumbnail), `/360` (card) or `/full`, so a client can show covers
  without loading `cover_url` from another site.
- `sources`: Per-storefront snapshots (empty for legacy rows).
- `description`: Blurb / synopsis text (may contain HTML).
- `subtitle`: Optional subtitle from bibliographic metadata.
- `narrators`: Comma-separated or JSON narrator list when present.
- `series`: Series name when the title belongs to a series.
- `series_index`: Position within the series (storefront string form).
- `publisher`: Publisher name from metadata enrichment or the storefront.
- `length_minutes`: Runtime in whole minutes when the storefront reports it.
- `published_at`: Publication date string from the storefront or enrichment.
- `genres`: Merged genre / category list for display.
- `language`: BCP-47 or storefront language code when known.
- `store_editions`: Distinct storefront editions contributing to this wishlist item.
- `purchase_hints`: Purchase URLs and prices observed per storefront.
- `created_at`: RFC 3339 timestamp when the row was inserted.
- `updated_at`: RFC 3339 timestamp when the row was last modified.

#### TitleRequestRecord::hide_prices_and_links

```rust
pub fn hide_prices_and_links(&mut self)
```

Removes the store links and prices of every purchase hint, for a
patron whose "See prices and store links" member control is off.

Implements: `Clone`, `ComposeSchema`, `Debug`, `Deserialize<'de>`, `Serialize`, `ToSchema`

### TitleRequestSourceRecord

```rust
pub struct TitleRequestSourceRecord {
    pub id: i64,
    pub title_request_id: i64,
    pub source: String,
    pub product_id: String,
    pub title: Option<String>,
    pub subtitle: Option<String>,
    pub authors: Option<String>,
    pub narrators: Option<String>,
    pub series: Option<String>,
    pub series_index: Option<String>,
    pub asin: Option<String>,
    pub isbn: Option<String>,
    pub description: Option<String>,
    pub publisher: Option<String>,
    pub length_minutes: Option<i64>,
    pub published_at: Option<String>,
    pub categories: Option<String>,
    pub language: Option<String>,
    pub cover_url: Option<String>,
    pub url: Option<String>,
    pub price_cents: Option<i64>,
    pub currency: Option<String>,
    pub price_label: Option<String>,
    pub list_price_cents: Option<i64>,
    pub list_price_label: Option<String>,
    pub member_price_cents: Option<i64>,
    pub member_price_label: Option<String>,
    pub observed_at: Option<DateTime<Utc>>,
    pub created_at: DateTime<Utc>,
    pub updated_at: DateTime<Utc>,
}
```

Per-storefront catalog/pricing snapshot attached to a wishlist row.

**Fields**

- `id`: Surrogate primary key assigned by the database.
- `title_request_id`: Foreign key to `title_requests.id`.
- `source`: Content-source plugin id (`audible`, `libro`, …).
- `product_id`: Storefront product id (ASIN, ISBN, UUID, …).
- `title`: Display title of the work or edition.
- `subtitle`: Optional subtitle from bibliographic metadata.
- `authors`: Comma-separated or JSON author list from the storefront.
- `narrators`: Comma-separated or JSON narrator list when present.
- `series`: Series name when the title belongs to a series.
- `series_index`: Position within the series (storefront string form).
- `asin`: Amazon ASIN when known; otherwise null.
- `isbn`: ISBN-10/13 when known; otherwise null.
- `description`: Blurb / synopsis text (may contain HTML).
- `publisher`: Publisher name from metadata enrichment or the storefront.
- `length_minutes`: Runtime in whole minutes when the storefront reports it.
- `published_at`: Publication date string from the storefront or enrichment.
- `categories`: Storefront category / genre path list (serialized).
- `language`: BCP-47 or storefront language code when known.
- `cover_url`: HTTPS URL for cover art when known.
- `url`: Storefront product or purchase URL when known.
- `price_cents`: Observed price in minor currency units, when known.
- `currency`: ISO 4217 currency code for price fields.
- `price_label`: Storefront-formatted price string for display.
- `list_price_cents`: List/MSRP price in minor units, when known.
- `list_price_label`: Storefront-formatted list price for display.
- `member_price_cents`: Member/subscriber price in minor units, when known.
- `member_price_label`: Storefront-formatted member price for display.
- `observed_at`: RFC 3339 time when this storefront snapshot was observed.
- `created_at`: RFC 3339 timestamp when the row was inserted.
- `updated_at`: RFC 3339 timestamp when the row was last modified.

Implements: `Clone`, `ComposeSchema`, `Debug`, `Default`, `Deserialize<'de>`, `Serialize`, `ToSchema`

### UserIntegrationHint

```rust
pub struct UserIntegrationHint {
    pub source: String,
    pub account_id: String,
    pub label: Option<String>,
}
```

Storefront / integration connection summary for admin user lists.

**Fields**

- `source`: Content-source or integration plugin id (`audible`, `libro`, …).
- `account_id`: Store or operator account id this row belongs to.
- `label`: Optional operator-facing account label.

Implements: `Clone`, `ComposeSchema`, `Debug`, `Deserialize<'de>`, `Serialize`, `ToSchema`

### UserInviteRecord

```rust
pub struct UserInviteRecord {
    pub id: i64,
    pub token_hash: String,
    pub role: UserRole,
    pub login_name: Option<String>,
    pub display_name: Option<String>,
    pub expires_at: DateTime<Utc>,
    pub redeemed_at: Option<DateTime<Utc>>,
    pub created_by: String,
    pub created_at: DateTime<Utc>,
}
```

Invite ticket for provisioning a User (token plaintext never stored).

**Fields**

- `id`: Surrogate primary key assigned by the database.
- `token_hash`: SHA-256 hex digest of the opaque token (plaintext never stored).
- `role`: First-party role (`administrator` or `member`).
- `login_name`: Local username for password login, when set.
- `display_name`: Human-readable name shown in the UI.
- `expires_at`: RFC 3339 expiry for the ticket, session, or code.
- `redeemed_at`: RFC 3339 time when the ticket/invite was redeemed, if any.
- `created_by`: Actor that created the row (user id or operator label).
- `created_at`: RFC 3339 timestamp when the row was inserted.

Implements: `Clone`, `Debug`, `Deserialize<'de>`, `Serialize`

### UserListeningHint

```rust
pub struct UserListeningHint {
    pub title: Option<String>,
    pub provider: String,
    pub last_listened_at: DateTime<Utc>,
}
```

Recent unfinished listening hint for admin user lists.

**Fields**

- `title`: Title of the work being listened to, when the library knows it.
- `provider`: Integration / storefront provider id.
- `last_listened_at`: RFC 3339 time of the last playback update.

Implements: `Clone`, `ComposeSchema`, `Debug`, `Deserialize<'de>`, `Serialize`, `ToSchema`

### UserPreferences

```rust
pub struct UserPreferences {
    pub id: i64,
    pub subject_key: String,
    pub identity_id: Option<i64>,
    pub default_view: String,
    pub disabled_shelves: Vec<String>,
    pub discover_sort: String,
    pub discover_sort_dir: String,
    pub discover_language: Option<String>,
    pub discover_excluded_sources: Vec<String>,
    pub theme: String,
    pub updated_at: DateTime<Utc>,
}
```

Per-user GUI / Discover preferences (operator or portal identity).

**Fields**

- `id`: Surrogate primary key assigned by the database.
- `subject_key`: Preference subject key (`operator`, `user:<id>`, portal key).
- `identity_id`: Foreign key to `portal_identities.id`.
- `default_view`: Preferred library SPA view identifier (for example `grid` or `list`).
- `disabled_shelves`: Shelf kind ids to hide (`author`, `chirp_deals`, …). Empty = all on.
- `discover_sort`: Preferred Discover sort column (for example `title` or `added`).
- `discover_sort_dir`: Discover sort direction (`asc` / `desc`).
- `discover_language`: Preferred content language (`en`, `__all__`, …). `None` = browser default.
- `discover_excluded_sources`: Store ids to hide in Discover. Empty = all sources (including future).
- `theme`: Appearance preference (`system`, `light`, or `dark`).
- `updated_at`: RFC 3339 timestamp when the row was last modified.

#### UserPreferences::defaults_for

```rust
pub fn defaults_for(subject_key: &str, identity_id: Option<i64>) -> Self
```

Defaults when no row exists yet.

Implements: `Clone`, `Debug`, `Deserialize<'de>`, `Serialize`

### UserPresenceExtras

```rust
pub struct UserPresenceExtras {
    pub online: bool,
    pub listening: Option<UserListeningHint>,
    pub integrations: Vec<UserIntegrationHint>,
    pub last_active_at: Option<DateTime<Utc>>,
}
```

Presence + connection extras for administrator user management.

**Fields**

- `online`: True when the user has a non-expired portal session.
- `listening`: Most recent unfinished listen within the listening window, if any.
- `integrations`: Linked storefront / integration accounts across portal identities.
- `last_active_at`: Most recent *unexpired* portal session activity (RFC 3339), when known.

Implements: `Clone`, `Debug`, `Deserialize<'de>`, `Serialize`

### UserRecord

```rust
pub struct UserRecord {
    pub id: i64,
    pub role: UserRole,
    pub status: UserStatus,
    pub display_name: Option<String>,
    pub login_name: Option<String>,
    pub email: Option<String>,
    pub has_password: bool,
    pub security_version: i64,
    pub created_at: DateTime<Utc>,
    pub updated_at: DateTime<Utc>,
    pub last_seen_at: Option<DateTime<Utc>>,
    pub avatar_source: Option<String>,
    pub totp_enabled: bool,
}
```

First-party Bookclerk user (security principal for portal paths).

**Fields**

- `id`: Surrogate primary key assigned by the database.
- `role`: First-party role (`owner`, `administrator`, or `member`).
- `status`: Lifecycle status for the row (user, request, …).
- `display_name`: Human-readable name shown in the UI.
- `login_name`: Local username for password login, when set.
- `email`: Optional contact email (notifications / magic-link invites later).
- `has_password`: Whether a local password hash is stored for this user.
- `security_version`: Incremented to invalidate existing sessions after security changes.
- `created_at`: RFC 3339 timestamp when the row was inserted.
- `updated_at`: RFC 3339 timestamp when the row was last modified.
- `last_seen_at`: Last authenticated portal activity (RFC 3339); survives session expiry.
- `avatar_source`: Explicit picture choice (`monogram` / `gravatar` / `upload` / `sso:{id}`); `None` is auto.
- `totp_enabled`: True when a TOTP authenticator has been confirmed for local password login.

Implements: `Clone`, `Debug`, `Deserialize<'de>`, `Serialize`

### UserSsoPicture

```rust
pub struct UserSsoPicture {
    pub identity_id: i64,
    pub provider: String,
    pub picture_url: String,
    pub last_used_at: Option<DateTime<Utc>>,
}
```

IdP-supplied avatar available as a profile-picture choice.

**Fields**

- `identity_id`: `portal_identities.id` this picture belongs to.
- `provider`: Identity-broker id (`oidc:google`, `oidc:github`, …).
- `picture_url`: HTTPS URL to display (hotlinked from the IdP / CDN).
- `last_used_at`: Most recent portal session activity for this identity, when any.

Implements: `Clone`, `Debug`, `Deserialize<'de>`, `Serialize`

### UserTitleState

```rust
pub struct UserTitleState {
    pub user_id: i64,
    pub work_id: String,
    pub is_finished: bool,
    pub rating_overall: Option<f64>,
    pub rating_performance: Option<f64>,
    pub rating_story: Option<f64>,
    pub tags: Option<String>,
    pub updated_at: DateTime<Utc>,
}
```

One user's own state for one work (`user_title_state`).

**Fields**

- `user_id`: The user (`users.id`) the state belongs to.
- `work_id`: The work (`works.id`) the state is about.
- `is_finished`: Whether the user finished the work.
- `rating_overall`: The user's overall rating (0 to 5), when given.
- `rating_performance`: The user's narration rating (0 to 5), when given.
- `rating_story`: The user's story rating (0 to 5), when given.
- `tags`: The user's tags, as one serialized string, when any.
- `updated_at`: When the state was last written.

Implements: `Clone`, `Debug`, `Deserialize<'de>`, `PartialEq`, `Serialize`

### WishlistPurchaseHint

```rust
pub struct WishlistPurchaseHint {
    pub source: String,
    pub product_id: String,
    pub title: Option<String>,
    pub url: Option<String>,
    pub price_cents: Option<i64>,
    pub currency: Option<String>,
    pub price_label: Option<String>,
    pub list_price_cents: Option<i64>,
    pub list_price_label: Option<String>,
    pub member_price_cents: Option<i64>,
    pub member_price_label: Option<String>,
}
```

Snapshotted purchase link/price for a wishlist title.

**Fields**

- `source`: Content-source plugin id (`audible`, `libro`, …).
- `product_id`: Storefront product id (ASIN, ISBN, UUID, …).
- `title`: Display title of the work or edition.
- `url`: Storefront product or purchase URL when known.
- `price_cents`: Observed price in minor currency units, when known.
- `currency`: ISO 4217 currency code for price fields.
- `price_label`: Storefront-formatted price string for display.
- `list_price_cents`: List/MSRP price in minor units, when known.
- `list_price_label`: Storefront-formatted list price for display.
- `member_price_cents`: Member/subscriber price in minor units, when known.
- `member_price_label`: Storefront-formatted member price for display.

#### WishlistPurchaseHint::hide_price_and_link

```rust
pub fn hide_price_and_link(&mut self)
```

Removes the hint's store link and every price it carries, keeping
the store and product id, for a patron whose "See prices and store
links" member control is off.

Implements: `Clone`, `ComposeSchema`, `Debug`, `Default`, `Deserialize<'de>`, `Serialize`, `ToSchema`

### WishlistStoreEdition

```rust
pub struct WishlistStoreEdition {
    pub source: String,
    pub product_id: String,
}
```

Storefront edition key for wishlist / queue payloads.

**Fields**

- `source`: Content-source plugin id (`audible`, `libro`, …).
- `product_id`: Storefront product id (ASIN, ISBN, UUID, …).

Implements: `Clone`, `ComposeSchema`, `Debug`, `Deserialize<'de>`, `Eq`, `PartialEq`, `Serialize`, `ToSchema`

### WorkRecord

```rust
pub struct WorkRecord {
    pub id: String,
    pub kind: String,
    pub title: String,
    pub parent_work_id: Option<String>,
    pub credits: Vec<String>,
    pub narrators: Vec<String>,
    pub series: Vec<String>,
    pub description: Option<String>,
    pub subjects: Option<String>,
    pub categories: Option<String>,
    pub cover_url: Option<String>,
    pub created_at: DateTime<Utc>,
    pub updated_at: DateTime<Utc>,
}
```

Canonical work spanning one or more ownership rows / editions.

**Fields**

- `id`: Work id (UUID).
- `kind`: Stored work kind (`book`, `podcast_show`, `podcast_episode`, `drama`).
- `title`: Display title.
- `parent_work_id`: The show a podcast episode belongs to.
- `credits`: Names credited on the work itself (authors, …), in billing order.
- `narrators`: Narrators credited on the work's expressions, each named once.
- `series`: Names of the series the work belongs to.
- `description`: Blurb / synopsis text (may contain HTML).
- `subjects`: Subject / topic tags from enrichment (serialized).
- `categories`: Store category / genre path list (serialized).
- `cover_url`: HTTPS URL for cover art when known.
- `created_at`: When the row was inserted.
- `updated_at`: When the row was last modified.

Implements: `Clone`, `Debug`, `Deserialize<'de>`, `Serialize`

### NamedAtomicBackend

```rust
pub struct NamedAtomicBackend<R> {
    /* private fields */
}
```

An [`AtomicTxnBackend`](#atomictxnbackend) that compiles every named operation and hands the
plan to a [`NamedAtomicRunner`](#namedatomicrunner).

#### NamedAtomicBackend::on

```rust
pub fn on(db: DatabaseConnection) -> Self
```

The backend over `db`, an open and migrated library connection.

#### NamedAtomicBackend::new

```rust
pub fn new(runner: R) -> Self
```

Wraps `runner`.

Implements: `AtomicTxnBackend`, `Debug`

### SchemaWalk

```rust
pub struct SchemaWalk {
    pub from: i64,
    pub requested_to: i64,
    pub stopped_at: i64,
    pub ups: Vec<HostMigrationStep>,
    pub downs: Vec<HostMigrationStep>,
    pub blocked: bool,
}
```

Result of walking a host migration plan from one version to another.

**Fields**

- `from`: Database version before the walk.
- `requested_to`: Version the caller asked to reach.
- `stopped_at`: Version the walk can actually reach (may be higher than `requested_to`
  when an irreversible step blocks a downgrade).
- `ups`: Remaining `up` steps in order (empty when rolling back).
- `downs`: `down` steps newest-first (empty when upgrading).
- `blocked`: True when a missing `down` stopped a rollback short of `requested_to`.

#### SchemaWalk::is_noop

```rust
pub fn is_noop(&self) -> bool
```

True when no DDL remains to apply.

Implements: `Clone`, `Debug`, `Eq`, `PartialEq`

### SessionClientInfo

```rust
pub struct SessionClientInfo {
    pub user_agent: Option<String>,
    pub device_type: String,
    pub client_label: String,
}
```

Parsed client presentation for a session row.

**Fields**

- `user_agent`: Raw User-Agent (or synthetic `API` / empty).
- `device_type`: High-level device class: `desktop`, `mobile`, `tablet`, `api`, or `unknown`.
- `client_label`: Human label such as `Windows`, `macOS`, `Android`, `iPhone`, or `API`.

Implements: `Clone`, `Debug`, `Eq`, `PartialEq`

### FacetValue

```rust
pub struct FacetValue {
    pub key: String,
    pub value: String,
    pub works: u64,
}
```

One value of a facet value list.

**Fields**

- `key`: The value's [`facet_key`](#facet_key), the list's order and cursor.
- `value`: The value as shown (one of its spellings).
- `works`: How many held works have it.

Implements: `Clone`, `Debug`, `Eq`, `PartialEq`

### FacetValuesPage

```rust
pub struct FacetValuesPage {
    pub values: Vec<FacetValue>,
    pub next: Option<String>,
}
```

One page of a facet value list.

**Fields**

- `values`: The values, by key.
- `next`: The last key returned, where the next page starts; `None` on the
  last page.

Implements: `Clone`, `Debug`, `Eq`, `PartialEq`

### LibraryWorkRow

```rust
pub struct LibraryWorkRow {
    pub work_id: String,
    pub kind: WorkKind,
    pub title: String,
    pub authors: Option<String>,
    pub series: Option<String>,
    pub series_position: Option<String>,
    pub cover_edition_id: Option<String>,
    pub media_kinds: Vec<MediaKind>,
    pub acquire_status: AcquireStatus,
    pub edition_count: u64,
    pub holding_count: u64,
    pub added_at: String,
    pub length_minutes: Option<u64>,
    pub purchased_at: Option<String>,
}
```

One row of the page index, as list views show it.

**Fields**

- `work_id`: The work.
- `kind`: Book or podcast.
- `title`: Display title.
- `authors`: Authors (or hosts) joined for display.
- `series`: First series name.
- `series_position`: Position in [`Self::series`](#libraryworkrow).
- `cover_edition_id`: The edition whose cover the list shows.
- `media_kinds`: Media kinds of the work's editions, sorted.
- `acquire_status`: The furthest acquire state any edition reached.
- `edition_count`: How many editions the work has.
- `holding_count`: How many holdings the work's listings have.
- `added_at`: RFC 3339 time of the newest holding.
- `length_minutes`: Runtime in minutes of the longest held edition, when known.
- `purchased_at`: RFC 3339 time of the newest purchase a store reported.

Implements: `Clone`, `Debug`, `Eq`, `PartialEq`

### UnheldItem

```rust
pub struct UnheldItem<'a> {
    pub authors: Option<&'a str>,
    pub narrators: Option<&'a str>,
    pub series: Option<&'a str>,
    pub genres: Option<&'a str>,
    pub is_explicit: Option<bool>,
}
```

What a catalog item the library does not hold says about itself, read
by [`WorkScope::admits_unheld`](#workscopeadmits_unheld). People are comma- or
semicolon-separated, genres as the page index splits them.

**Fields**

- `authors`: The item's authors, as one comma-separated string.
- `narrators`: The item's narrators, as one comma-separated string.
- `series`: The series name, without its position.
- `genres`: Genre labels, separated by `,`, `;` or `|`.
- `is_explicit`: Whether a store marks the item explicit (adult) content; `None`
  when no store said.

Implements: `Clone`, `Copy`, `Debug`, `Default`, `Eq`, `PartialEq`

### WorkCursor

```rust
pub struct WorkCursor {
    pub sort: WorkSort,
    pub key: String,
    pub title: Option<String>,
    pub work_id: String,
}
```

Where the next page of a [`WorkSort`](#worksort) order starts: the sort values of
the last row returned.

**Fields**

- `sort`: The order the cursor belongs to; a cursor is refused under another.
- `key`: The last row's primary sort value (sort title, sort author, added-at
  stamp, series key, purchase stamp or runtime in minutes).
- `title`: The last row's sort title; set for [`WorkSort::Author`](#worksort) and
  [`WorkSort::Series`](#worksort), whose ties break on title.
- `work_id`: The last row's work id, the final tie-break.

#### WorkCursor::encode

```rust
pub fn encode(&self) -> String
```

The opaque token a client sends back: URL-safe base64 of the JSON.

#### WorkCursor::decode

```rust
pub fn decode(token: &str) -> Result<Self>
```

Parses a token from [`Self::encode`](#workcursorencode).

##### Errors

Returns [`LibraryError::Invalid`](#libraryerror) when the token is not one this
library issued.

Implements: `Clone`, `Debug`, `Deserialize<'de>`, `Eq`, `PartialEq`, `Serialize`

### WorkPage

```rust
pub struct WorkPage {
    pub works: Vec<LibraryWorkRow>,
    pub next: Option<WorkCursor>,
}
```

One page of library works.

**Fields**

- `works`: The rows, in the requested order.
- `next`: Where the next page starts; `None` on the last page.

Implements: `Clone`, `Debug`, `Eq`, `PartialEq`

### WorkPageQuery

```rust
pub struct WorkPageQuery {
    pub sort: WorkSort,
    pub media_kind: Option<MediaKind>,
    pub kind: Option<WorkKind>,
    pub status: Option<AcquireStatus>,
    pub account: Option<String>,
    pub filter: Option<WorkFilter>,
    pub scope: Option<WorkScope>,
    pub after: Option<WorkCursor>,
    pub limit: u64,
}
```

Filters and position of one page of library works.

**Fields**

- `sort`: Order of the page.
- `media_kind`: Only works with an edition of this media kind.
- `kind`: Only works of this kind.
- `status`: Only works whose furthest acquire state is this one.
- `account`: Only works held by this store account id (any plugin).
- `filter`: Only works that pass this filter.
- `scope`: Only works inside this patron scope; `None` for a caller who sees
  the whole library.
- `after`: Continue after this cursor.
- `limit`: Most rows to return; at least 1.

Implements: `Clone`, `Debug`, `Default`

### WorkScope

```rust
pub struct WorkScope {
    pub include: Vec<(WorkFacet, String)>,
    pub exclude: Vec<(WorkFacet, String)>,
    pub owner: Option<i64>,
}
```

The part of the library a patron may see, as the page index applies it.

A work passes when one of the patron's own store accounts holds it, or
when it matches no exclude value and, if there are include values,
matches at least one of them. Values are [`facet_key`](#facet_key)s of the page
index's facets.

**Fields**

- `include`: `(facet, key)` pairs a work must match one of; empty lets every
  work through.
- `exclude`: `(facet, key)` pairs that hide a work, whatever `include` says.
- `owner`: The patron (`users.id`) whose linked store accounts' books always
  pass; `None` for a caller with no accounts of their own.

#### WorkScope::sees_everything

```rust
pub fn sees_everything(&self) -> bool
```

True when the scope hides nothing.

#### WorkScope::admits_unheld

```rust
pub fn admits_unheld(&self, item: &UnheldItem<'_>) -> bool
```

True when a catalog item the library does not hold may be shown to
the patron: Discover shelves and storefront search results.

The item's own metadata stands in for the page index's facets, with
the same rule: no exclude value matches and, when there are include
values, one of them does. An item carries no library tag or
collection, so a scope that includes only by those admits no
catalog item, and excludes by them never hide one.

##### Arguments

* `item` - What the store listing says about the item.

#### WorkScope::key_set_id

```rust
pub fn key_set_id(&self) -> String
```

Content id of the scope's include and exclude values: lowercase hex
SHA-256 over its `(effect, facet, key)` rows, sorted and without
duplicates, each field length-prefixed, so two
scopes share an id exactly when they hold the same values.

Implements: `Clone`, `Debug`, `Default`, `Eq`, `PartialEq`

### EditionDetail

```rust
pub struct EditionDetail {
    pub id: String,
    pub work_id: String,
    pub expression_id: String,
    pub expression_kind: ExpressionKind,
    pub language: Option<String>,
    pub abridgement: Abridgement,
    pub media_kind: MediaKind,
    pub title: String,
    pub subtitle: Option<String>,
    pub published_at: Option<String>,
    pub duration_minutes: Option<i64>,
    pub cover_url: Option<String>,
    pub acquire_status: AcquireStatus,
    pub listings: Vec<EditionListing>,
}
```

One edition of a work with its expression and listings.

**Fields**

- `id`: Edition id.
- `work_id`: The work the edition realises: the work itself, or one of a show's
  episodes.
- `expression_id`: The expression the edition carries.
- `expression_kind`: How the expression realises the work.
- `language`: The expression's language tag, when known.
- `abridgement`: Whether the expression is abridged.
- `media_kind`: Carrier of the edition.
- `title`: The edition's title.
- `subtitle`: The edition's subtitle.
- `published_at`: Publication date as the store gave it.
- `duration_minutes`: Running time in minutes, for audio.
- `cover_url`: Cover image URL from the store, when known.
- `acquire_status`: The edition's acquire state.
- `listings`: The edition's listings, by id.

Implements: `Clone`, `Debug`, `PartialEq`

### EditionListing

```rust
pub struct EditionListing {
    pub id: String,
    pub plugin_key: String,
    pub marketplace: String,
    pub product_id: String,
    pub url: Option<String>,
    pub holdings: Vec<ListingHolding>,
}
```

One store listing of an edition.

**Fields**

- `id`: Listing id.
- `plugin_key`: The storefront plugin that lists it.
- `marketplace`: Store marketplace (`us`, `uk`, …), empty when the store has one.
- `product_id`: The store's product id (ASIN, ISBN, product UUID, …).
- `url`: The product page, when known.
- `holdings`: The listing's holdings, by id.

Implements: `Clone`, `Debug`, `Eq`, `PartialEq`

### ListingHolding

```rust
pub struct ListingHolding {
    pub id: String,
    pub holder: HoldingHolder,
    pub entitlement: Entitlement,
    pub purchased_at: Option<String>,
    pub added_at: String,
}
```

One holding of a listing.

**Fields**

- `id`: Holding id.
- `holder`: Who holds the listing.
- `entitlement`: On what terms the holder has it.
- `purchased_at`: RFC 3339 purchase time, when the store reported one.
- `added_at`: RFC 3339 time the holding was first recorded.

Implements: `Clone`, `Debug`, `Eq`, `PartialEq`

### WorkDetail

```rust
pub struct WorkDetail {
    pub id: String,
    pub kind: WorkKind,
    pub title: String,
    pub description: Option<String>,
    pub authors: Vec<String>,
    pub series: Option<String>,
    pub series_position: Option<String>,
    pub editions: Vec<EditionDetail>,
}
```

A top-level work with everything its detail view shows.

**Fields**

- `id`: Work id.
- `kind`: Book or podcast.
- `title`: Display title.
- `description`: Blurb or synopsis (may contain HTML).
- `authors`: Authors (hosts for a podcast), in billing order.
- `series`: First series name.
- `series_position`: Position in [`Self::series`](#workdetail).
- `editions`: Editions, by id; a show's episodes' editions included.

Implements: `Clone`, `Debug`, `PartialEq`

### WorkSearchDoc

```rust
pub struct WorkSearchDoc {
    pub work_id: String,
    pub title: String,
    pub other_titles: Vec<String>,
    pub authors: Vec<String>,
    pub narrators: Vec<String>,
    pub series: Vec<String>,
    pub identifiers: Vec<String>,
}
```

The searchable text of one held top-level work.

**Fields**

- `work_id`: The work (a podcast's show, never an episode).
- `title`: The work's title.
- `other_titles`: Distinct edition and listing titles and subtitles other than
  [`Self::title`](#worksearchdoc) (episode titles, for a show).
- `authors`: Authors (hosts, for a podcast), from the model's credits and the
  listings' author text.
- `narrators`: Narrators, from the listings' narrator text.
- `series`: Series names, from the model and the listings.
- `identifiers`: Product ids of the listings and identifier values (ISBN, ASIN and
  others) of the editions, lowercased.

Implements: `Clone`, `Debug`, `Default`, `Eq`, `PartialEq`

### WorkTag

```rust
pub struct WorkTag {
    pub value: String,
    pub source: WorkTagSource,
}
```

One library tag on a work.

**Fields**

- `value`: The tag as written.
- `source`: Where it came from.

Implements: `Clone`, `ComposeSchema`, `Debug`, `Eq`, `PartialEq`, `Serialize`, `ToSchema`

### WorkTagsBody

```rust
pub struct WorkTagsBody {
    /* private fields */
}
```

`PUT /api/library/works/{id}/tags` body: the tags an Administrator sets.

#### WorkTagsBody::validate

```rust
pub fn validate(self) -> Result<WorkTagsRequest>
```

Checks the body ([`normalize_work_tags`](#normalize_work_tags)).

##### Errors

Returns [`LibraryError::Invalid`](#libraryerror) for too many or too long tags.

Implements: `ComposeSchema`, `Debug`, `Deserialize<'de>`, `ToSchema`

### WorkTagsRequest

```rust
pub struct WorkTagsRequest {
    /* private fields */
}
```

A validated [`WorkTagsBody`]: at most [`MAX_WORK_TAGS`](#max_work_tags) distinct tags
(by [`facet_key`](#facet_key)), trimmed, each 1 to [`MAX_WORK_TAG_BYTES`](#max_work_tag_bytes) bytes, in
the order sent with the first spelling kept.

#### WorkTagsRequest::tags

```rust
pub fn tags(&self) -> &[String]
```

The tags as written, trimmed and distinct.

Implements: `Clone`, `Debug`, `Eq`, `PartialEq`

### CatalogListing

```rust
pub struct CatalogListing {
    pub draft: ListingDraft,
    pub detail: ListingDetail,
    pub offers: Vec<ListingOffer>,
}
```

One title a store's catalog answer reported.

**Fields**

- `draft`: What the resolver sees: key, kinds, title, identifiers, credits and
  series. Its plugin key and marketplace must be set.
- `detail`: The store's own text for it.
- `offers`: Prices the answer quoted (often none: search hits rarely price).

Implements: `Clone`, `Debug`

### CatalogListingRecord

```rust
pub struct CatalogListingRecord {
    pub id: String,
    pub plugin_key: String,
    pub marketplace: String,
    pub product_id: String,
    pub edition_id: String,
    pub work_id: Option<String>,
    pub expression_id: Option<String>,
    pub title: String,
    pub categories: Option<String>,
    pub detail: ListingDetail,
    pub asin: Option<String>,
    pub isbn: Option<String>,
    pub offers: Vec<ListingOffer>,
    pub observed_at: DateTime<Utc>,
}
```

A stored catalog listing, as Discover and the wishlist read it.

**Fields**

- `id`: `store_listings.id`.
- `plugin_key`: Provenance-qualified PluginKey of the store.
- `marketplace`: Lower-cased marketplace.
- `product_id`: The store's product id.
- `edition_id`: The edition the listing sells.
- `work_id`: That edition's work, when the rows are consistent.
- `expression_id`: That edition's expression (`expressions.id`), when the rows are
  consistent. Listings of one expression are the same recording or
  text, so their prices compare; listings of one work in different
  expressions (a narration and a full-cast dramatisation) do not.
- `title`: Title as the store lists it.
- `categories`: Store category labels.
- `detail`: The store's own text.
- `asin`: ASIN of the edition, preferring one scoped to the marketplace.
- `isbn`: An ISBN of the edition.
- `offers`: Stored prices, one per audience.
- `observed_at`: When a store last reported the listing.

#### CatalogListingRecord::offer

```rust
pub fn offer(&self, audience: OfferAudience) -> Option<&ListingOffer>
```

The stored offer for `audience`.

Implements: `Clone`, `Debug`

### ListingDetail

```rust
pub struct ListingDetail {
    pub subtitle: Option<String>,
    pub authors: Option<String>,
    pub narrators: Option<String>,
    pub series: Option<String>,
    pub series_index: Option<String>,
    pub series_product_id: Option<String>,
    pub description: Option<String>,
    pub publisher: Option<String>,
    pub published_at: Option<String>,
    pub length_minutes: Option<i64>,
    pub language: Option<String>,
    pub cover_url: Option<String>,
    pub url: Option<String>,
    pub rating_overall: Option<f64>,
    pub rating_count: Option<i64>,
    pub is_abridged: Option<bool>,
    pub is_explicit: Option<bool>,
}
```

The store's own text for a listing, beyond the title the resolver sees.

Display only: credits and series here are as the store prints them, and
nothing matches on them.

**Fields**

- `subtitle`: Subtitle.
- `authors`: Authors, comma-separated.
- `narrators`: Narrators, comma-separated.
- `series`: Series name.
- `series_index`: Position in the series (`1`, `2.5`).
- `series_product_id`: The store's own product id for the series or podcast show (an
  Audible series ASIN); store-specific, so it lives on the listing.
- `description`: Synopsis, kept whole.
- `publisher`: Publisher or imprint.
- `published_at`: Publication date as the store reports it.
- `length_minutes`: Runtime in whole minutes.
- `language`: Language code (`en`).
- `cover_url`: Cover image URL.
- `url`: Product page URL.
- `rating_overall`: Community rating (0 to 5).
- `rating_count`: How many ratings back the rating.
- `is_abridged`: Abridged (`true`), unabridged (`false`) or unknown.
- `is_explicit`: Explicit (adult) content (`true`), clean (`false`) or unknown.

#### ListingDetail::of_new_book

```rust
pub fn of_new_book(book: &NewBook) -> Self
```

The store text a scanned library title carries, for its listing.

Only the fields a scan reports are taken. A title the store marks
unabridged is left unknown, since a scan record cannot tell
unabridged from unreported.

#### ListingDetail::of_catalog_hit

```rust
pub fn of_catalog_hit(hit: &CatalogHit) -> Self
```

The store text of a storefront catalog record, as the store
reported it.

Everything the record carries is taken, including cover, synopsis,
product page and rating; a podcast episode's show product becomes
[`Self::series_product_id`](#listingdetail). HTML entities are left as the record
has them.

Implements: `Clone`, `Debug`, `Default`, `PartialEq`

### ListingOffer

```rust
pub struct ListingOffer {
    pub audience: OfferAudience,
    pub price_cents: Option<i64>,
    pub price_label: Option<String>,
    pub currency: Option<String>,
    pub observed_at: DateTime<Utc>,
}
```

One price a store quoted for one listing and audience.

**Fields**

- `audience`: Who pays it.
- `price_cents`: Price in minor units of [`Self::currency`](#listingoffer), when the store gave one.
- `price_label`: The store's formatted price (`$14.95`), when it gave one.
- `currency`: ISO 4217 currency code.
- `observed_at`: When the store quoted it.

#### ListingOffer::is_empty

```rust
pub fn is_empty(&self) -> bool
```

True when the offer carries neither a price nor a label.

Implements: `Clone`, `Debug`, `PartialEq`

### RecordedListings

```rust
pub struct RecordedListings {
    pub ids: Vec<Option<String>>,
    pub unresolved: usize,
}
```

What [`LibraryStore::record_catalog_listings`](#librarystorerecord_catalog_listings) stored for one answer.

**Fields**

- `ids`: One entry per input, in order: the listing id, or `None` for a title
  that was not stored.
- `unresolved`: How many valid titles could not be resolved this time (other writers
  kept changing their rows on every attempt). An answer with any is incomplete and should not be
  cached, so those titles are tried again.

Implements: `Clone`, `Debug`, `Default`, `Eq`, `PartialEq`

### Chapter

```rust
pub struct Chapter {
    pub title: String,
    pub start_ms: u64,
    pub depth: u32,
}
```

One chapter of a checked list.

Named `EditionChapter` in the OpenAPI documents, apart from the player
apps' `Chapter`.

**Fields**

- `title`: Chapter title; never empty, at most 512 bytes, no control
  characters.
- `start_ms`: Start, in milliseconds from the start of the edition's timeline.
- `depth`: Nesting depth; 0 for a top-level chapter, at most 7. A chapter at
  depth n + 1 is nested under the nearest earlier chapter at depth n.

Implements: `Clone`, `ComposeSchema`, `Debug`, `Eq`, `PartialEq`, `Serialize`, `ToSchema`

### ChapterClaim

```rust
pub struct ChapterClaim {
    pub source_kind: ChapterSourceKind,
    pub source_id: String,
    pub quality: ClaimQuality,
    pub digest: ChapterDigest,
    pub chapter_count: Option<u64>,
    pub runtime_ms: Option<u64>,
    pub input_digest: Option<ChapterDigest>,
    pub revision: i64,
    pub pinned_at: Option<String>,
    pub created_by_user_id: Option<i64>,
    pub updated_at: String,
}
```

One stored chapter claim, without its list.

**Fields**

- `source_kind`: The kind of source.
- `source_id`: Which source of that kind (a PluginKey; empty for a person's edit
  and for Bookclerk's processing).
- `quality`: Where a machine source ranks within its kind.
- `digest`: The claimed list.
- `chapter_count`: How many chapters the list has; `None` while its write is
  unfinished.
- `runtime_ms`: Length of the timeline the list was measured on, in milliseconds.
- `input_digest`: For a processed claim, the list it was derived from.
- `revision`: The edition's claim counter when this claim last changed.
- `pinned_at`: When an Administrator pinned it (RFC 3339).
- `created_by_user_id`: The person whose edit it is.
- `updated_at`: When it last changed (RFC 3339).

Implements: `Clone`, `Debug`, `Eq`, `PartialEq`

### ChapterClaimWrite

```rust
pub struct ChapterClaimWrite {
    pub source: ChapterSource,
    pub list: ChapterList,
    pub runtime_ms: Option<u64>,
}
```

A claim to record ([`LibraryStore::record_chapter_claim`](#librarystorerecord_chapter_claim)).

**Fields**

- `source`: Who states it.
- `list`: The chapters.
- `runtime_ms`: Length of the timeline the chapters were measured on, in
  milliseconds, when known.

Implements: `Clone`, `Debug`

### ChapterDigest

```rust
pub struct ChapterDigest(/* private */);
```

Lower-case hex SHA-256 naming one stored chapter list
([`ChapterList::digest`](#chapterlistdigest)).

#### ChapterDigest::parse

```rust
pub fn parse(stored: &str) -> Option<Self>
```

Reads a stored digest, refusing anything but 64 lower-case hex
characters.

#### ChapterDigest::as_str

```rust
pub fn as_str(&self) -> &str
```

The digest as stored.

Implements: `Clone`, `Debug`, `Display`, `Eq`, `Hash`, `Ord`, `PartialEq`, `PartialOrd`

### ChapterInput

```rust
pub struct ChapterInput {
    pub title: String,
    pub start_ms: u64,
    pub depth: u32,
}
```

One chapter as a source states it, before it is checked.

**Fields**

- `title`: Chapter title.
- `start_ms`: Start, in milliseconds from the start of the edition's timeline.
- `depth`: Nesting depth; 0 for a top-level chapter.

Implements: `Clone`, `Debug`, `Eq`, `PartialEq`

### ChapterList

```rust
pub struct ChapterList(/* private */);
```

A checked chapter list, in pre-order with starts never decreasing.

Every list satisfies: at most [`MAX_CHAPTERS`](#max_chapters) chapters; each title
non-empty, at most [`MAX_CHAPTER_TITLE_BYTES`](#max_chapter_title_bytes) bytes, free of control
characters and of leading or trailing white space; each start at most
[`MAX_CHAPTER_START_MS`](#max_chapter_start_ms) and no earlier than the one before; the first
depth 0 and each later depth at most one more than the one before and
at most [`MAX_CHAPTER_DEPTH`](#max_chapter_depth).

#### ChapterList::new

```rust
pub fn new(input: Vec<ChapterInput>, check: ChapterCheck) -> Result<Self>
```

Checks `input` into a list.

##### Arguments

* `input` - The chapters in pre-order.
* `check` - Whether to refuse or repair what breaks a rule.

##### Errors

Returns [`LibraryError::Invalid`](#libraryerror) naming the first chapter that
breaks a rule [`ChapterCheck`](#chaptercheck) does not repair.

#### ChapterList::chapters

```rust
pub fn chapters(&self) -> &[Chapter]
```

The chapters, in pre-order.

#### ChapterList::is_empty

```rust
pub fn is_empty(&self) -> bool
```

True when the list has no chapters.

#### ChapterList::spans

```rust
pub fn spans(&self, end_ms: Option<u64>) -> Vec<ChapterSpan>
```

The chapters as spans a player can seek between: in pre-order, each
ending where the next one starts, as Audiobookshelf flattens a
tree, and the last ending at `end_ms`.

##### Arguments

* `end_ms` - Where the timeline ends, in milliseconds. `None`, or a
  value before the last chapter's start, ends the last chapter at
  its own start.

##### Returns

One span per chapter, starts non-decreasing, every end at or after
its start.

#### ChapterList::digest

```rust
pub fn digest(&self) -> ChapterDigest
```

The list's digest: SHA-256 over each chapter's start (8 bytes, big
endian), depth (4 bytes), title length (4 bytes) and title bytes,
in order.

Implements: `Clone`, `Debug`, `Default`, `Eq`, `PartialEq`

### ChapterSpan

```rust
pub struct ChapterSpan {
    pub title: String,
    pub start_ms: u64,
    pub end_ms: u64,
    pub depth: u32,
}
```

One chapter as a player seeks between them: where it starts and ends
on the edition's timeline ([`ChapterList::spans`](#chapterlistspans)).

**Fields**

- `title`: Chapter title.
- `start_ms`: Start, in milliseconds from the start of the edition's timeline.
- `end_ms`: End, in milliseconds: the next chapter's start, or the end of the
  timeline for the last chapter; never before `start_ms`.
- `depth`: Nesting depth; 0 for a top-level chapter. A chapter at depth n + 1
  is nested under the nearest earlier chapter at depth n.

Implements: `Clone`, `ComposeSchema`, `Debug`, `Eq`, `PartialEq`, `Serialize`, `ToSchema`

### EditionChapters

```rust
pub struct EditionChapters {
    pub list: ChapterList,
    pub source_kind: ChapterSourceKind,
    pub source_id: String,
    pub runtime_ms: Option<u64>,
}
```

The chapters an edition resolved to.

**Fields**

- `list`: The chapters; empty when an Administrator cleared them.
- `source_kind`: The winning claim's source kind.
- `source_id`: The winning claim's source id.
- `runtime_ms`: Length of the timeline the list was measured on, in milliseconds.

Implements: `Clone`, `Debug`, `Eq`, `PartialEq`

### ChapterEditBody

```rust
pub struct ChapterEditBody {
    /* private fields */
}
```

`PUT /api/library/editions/{id}/chapters` body: the edition's chapters
as an Administrator sets them, in pre-order. An empty list clears them.

#### ChapterEditBody::validate

```rust
pub fn validate(self) -> Result<ChapterList>
```

Checks the chapters strictly ([`ChapterCheck::Strict`](#chaptercheck)).

##### Errors

Returns [`LibraryError::Invalid`](#libraryerror) naming the first chapter that
breaks a rule.

Implements: `ComposeSchema`, `Debug`, `Deserialize<'de>`, `ToSchema`

### ChapterEditEntry

```rust
pub struct ChapterEditEntry {
    /* private fields */
}
```

One chapter of an edit, as JSON carries it.

Implements: `ComposeSchema`, `Debug`, `Deserialize<'de>`, `ToSchema`

### ChapterPinBody

```rust
pub struct ChapterPinBody {
    /* private fields */
}
```

`PUT /api/library/editions/{id}/chapters/pin` body: the claim to pin.

#### ChapterPinBody::validate

```rust
pub fn validate(self) -> Result<ChapterPinRequest>
```

Checks the source id's length and characters.

##### Errors

Returns [`LibraryError::Invalid`](#libraryerror) for a source id over
[`MAX_CHAPTER_SOURCE_ID_BYTES`](#max_chapter_source_id_bytes) or with a control character.

Implements: `ComposeSchema`, `Debug`, `Deserialize<'de>`, `ToSchema`

### ChapterPinRequest

```rust
pub struct ChapterPinRequest {
    pub source_kind: ChapterSourceKind,
    pub source_id: String,
}
```

A checked pin request.

**Fields**

- `source_kind`: The claim's source kind.
- `source_id`: The claim's source id.

Implements: `Clone`, `Debug`, `Eq`, `PartialEq`

### TitleQuery

```rust
pub struct TitleQuery<'a> {
    pub title: &'a str,
    pub authors: Option<&'a str>,
    pub asin: Option<&'a str>,
    pub isbn: Option<&'a str>,
    pub series: Option<&'a str>,
    pub series_index: Option<&'a str>,
}
```

A title someone asks about (a wish, a catalog card): what the resolver
needs to say which work it belongs to.

**Fields**

- `title`: Display title.
- `authors`: Comma-separated authors, when known.
- `asin`: Audible ASIN, when known.
- `isbn`: ISBN, when known.
- `series`: Series name, when known.
- `series_index`: Position in the series, when known.

Implements: `Clone`, `Debug`, `Default`

### WorkEnrichment

```rust
pub struct WorkEnrichment {
    pub description: Option<String>,
    pub subjects: Option<String>,
    pub categories: Option<String>,
    pub cover_url: Option<String>,
}
```

Enrichment fields for [`LibraryStore::set_work_enrichment`](#librarystoreset_work_enrichment).

**Fields**

- `description`: Blurb / synopsis text.
- `subjects`: Subject tags (serialized).
- `categories`: Category labels (serialized).
- `cover_url`: HTTPS cover URL.

#### WorkEnrichment::of_new_book

```rust
pub fn of_new_book(book: &NewBook) -> Self
```

The fields a scanned library title offers its work: its store
categories (a scan reports no synopsis, subjects or cover).

#### WorkEnrichment::is_empty

```rust
pub fn is_empty(&self) -> bool
```

True when no field is set.

Implements: `Clone`, `Debug`, `Default`, `Eq`, `PartialEq`

### ListeningIdentifiers

```rust
pub struct ListeningIdentifiers {
    pub asin: Option<String>,
    pub isbn: Option<String>,
}
```

The identifiers a listening item names its book by, as the provider
reported them (any case and punctuation; normalised here).

**Fields**

- `asin`: Amazon / Audible ASIN, when the provider knows it.
- `isbn`: ISBN-10 or ISBN-13, when the provider knows it.

Implements: `Clone`, `Debug`, `Default`, `Eq`, `PartialEq`

### ListeningMatch

```rust
pub struct ListeningMatch {
    pub work_id: Option<String>,
    pub edition_id: Option<String>,
}
```

What one listening item resolved to in the library.

**Fields**

- `work_id`: Work the item's identifiers resolve to, when they name exactly one.
- `edition_id`: Edition carrying the identifier that decided the work, when exactly
  one edition carries it.

Implements: `Clone`, `Debug`, `Default`, `Eq`, `PartialEq`

### ListeningProgressWrite

```rust
pub struct ListeningProgressWrite {
    pub before: Option<ListeningProgressRecord>,
    pub after: NewListeningProgress,
}
```

One stored listening row a batched write produced.

**Fields**

- `before`: The stored row before this write; `None` when the write inserted it.
- `after`: The row as written: every report for its key merged over `before`
  the way [`LibraryStore::upsert_listening_progress`](#librarystoreupsert_listening_progress) merges one.

Implements: `Clone`, `Debug`

### ConflictCursor

```rust
pub struct ConflictCursor {
    pub listing_id: String,
    pub kind: String,
    pub subject_ids: String,
}
```

Where a conflicts page stopped: the last row's key, in key order.

**Fields**

- `listing_id`: The last row's listing id.
- `kind`: The last row's kind.
- `subject_ids`: The last row's stored subject ids (a JSON array).

#### ConflictCursor::encode

```rust
pub fn encode(&self) -> String
```

The cursor as an opaque URL-safe token.

#### ConflictCursor::decode

```rust
pub fn decode(token: &str) -> Result<Self>
```

Reads a token made by [`Self::encode`](#conflictcursorencode).

##### Errors

Returns [`LibraryError::Invalid`](#libraryerror) when the token is not one.

Implements: `Clone`, `Debug`, `Deserialize<'de>`, `Eq`, `PartialEq`, `Serialize`

### ResolutionConflict

```rust
pub struct ResolutionConflict {
    pub listing_id: String,
    pub kind: String,
    pub subject_ids: Vec<String>,
    pub plugin_key: String,
    pub product_id: String,
    pub title: String,
    pub edition_id: String,
    pub updated_at: String,
}
```

One conflict the resolver recorded for an Administrator.

**Fields**

- `listing_id`: The listing whose resolution conflicted.
- `kind`: `strong_identifiers`, `work_identifiers`, `ambiguous_work` or
  `work_kind_mismatch`.
- `subject_ids`: The edition ids (`strong_identifiers`) or work ids (the other kinds)
  involved, sorted.
- `plugin_key`: The listing's storefront PluginKey.
- `product_id`: The store's product id.
- `title`: The listing's title as the store reported it.
- `edition_id`: The edition the listing sells now.
- `updated_at`: RFC 3339 time the conflict was last seen.

Implements: `Clone`, `Debug`, `Eq`, `PartialEq`

### AccountPaceState

```rust
pub struct AccountPaceState {
    pub override_: PaceOverride,
    pub cooldown_until: Option<DateTime<Utc>>,
    pub window: DownloadWindow,
}
```

One account's pacing state as stored.

**Fields**

- `override_`: The operator's override of the storefront's pace.
- `cooldown_until`: End of the store's cool-down, when one is set (it may be past).
- `window`: Downloads in the last [`BUDGET_WINDOW`](pacing.md#budget_window), oldest first.

Implements: `Clone`, `Debug`, `Default`, `Eq`, `PartialEq`

### DownloadId

```rust
pub struct DownloadId(/* private */);
```

One store download's ledger identity.

It is made once, when a download is kept for recording, and every write
attempt of that download passes the same one, so a write that committed
but whose answer was lost inserts no second row when it is tried again.

#### DownloadId::new

```rust
pub fn new() -> Self
```

A fresh identity (a random UUID) for a new download.

#### DownloadId::as_str

```rust
pub fn as_str(&self) -> &str
```

The identity as the ledger's `id` column stores it.

Implements: `Clone`, `Debug`, `Default`, `Eq`, `Hash`, `Ord`, `PartialEq`, `PartialOrd`

### Bookmark

```rust
pub struct Bookmark {
    pub id: String,
    pub user_id: i64,
    pub edition_id: String,
    pub position: PlaybackMs,
    pub title: String,
    pub created_at: DateTime<Utc>,
    pub updated_at: DateTime<Utc>,
}
```

A place one person marked in one edition.

**Fields**

- `id`: Bookmark id (hyphenated UUID).
- `user_id`: The person who made it (`users.id`).
- `edition_id`: The edition it is in.
- `position`: The marked place.
- `title`: The person's note; may be empty.
- `created_at`: When it was made.
- `updated_at`: When it last changed.

Implements: `Clone`, `Debug`, `Eq`, `PartialEq`

### NewPlaybackSession

```rust
pub struct NewPlaybackSession<'a> {
    pub user_id: i64,
    pub edition_id: &'a str,
    pub serving_node_id: &'a str,
    pub destination: &'a str,
    pub files_digest: &'a str,
    pub client_kind: PlayerKind,
    pub client_name: &'a str,
}
```

A play session to open ([`LibraryStore::open_playback_session`](#librarystoreopen_playback_session)).

**Fields**

- `user_id`: The listener (`users.id`).
- `edition_id`: The edition to play.
- `serving_node_id`: This node's id: it streams the files.
- `destination`: The destination the files will be read from.
- `files_digest`: Digest of the edition's file rows on that destination
  ([`crate::control_plane::PlaybackPlan::digest`](control_plane.md#playbackplandigest)).
- `client_kind`: What kind of player asks.
- `client_name`: The player's own name; at most [`MAX_PLAYER_NAME_BYTES`](#max_player_name_bytes) bytes are
  kept.

Implements: `Clone`, `Copy`, `Debug`

### PlaybackChange

```rust
pub struct PlaybackChange {
    pub progress: PlaybackProgress,
    pub became_finished: bool,
}
```

The result of a sync or of setting progress outright.

**Fields**

- `progress`: The person's progress after the write.
- `became_finished`: True when this write is the one that reached the edition's end.

Implements: `Clone`, `Debug`, `Eq`, `PartialEq`

### PlaybackEdition

```rust
pub struct PlaybackEdition {
    pub edition_id: String,
    pub work_id: Option<String>,
    pub title: String,
    pub runtime: Option<PlaybackMs>,
}
```

What the player needs to know of an edition before it plays
([`LibraryStore::playback_edition`](#librarystoreplayback_edition)).

**Fields**

- `edition_id`: The edition.
- `work_id`: The work it belongs to; `None` only while the edition is being
  moved between works.
- `title`: The edition's display title.
- `runtime`: Its runtime as the catalog states it, to the minute; players
  report the exact one once they have loaded the audio.

Implements: `Clone`, `Debug`, `Eq`, `PartialEq`

### PlaybackMs

```rust
pub struct PlaybackMs(/* private */);
```

A length of time on a recording's timeline, in milliseconds: a
position, a duration or time listened. At most [`MAX_PLAYBACK_MS`](#max_playback_ms).

#### PlaybackMs::ZERO

```rust
pub const ZERO: Self = _
```

The start of a recording.

#### PlaybackMs::new

```rust
pub fn new(ms: u64) -> Option<Self>
```

`ms` as a timeline length.

##### Returns

`None` when `ms` is past [`MAX_PLAYBACK_MS`](#max_playback_ms).

#### PlaybackMs::get

```rust
pub fn get(self) -> u64
```

The length in milliseconds.

Implements: `Clone`, `Copy`, `Debug`, `Default`, `Eq`, `Hash`, `Ord`, `PartialEq`, `PartialOrd`

### PlaybackProgress

```rust
pub struct PlaybackProgress {
    pub user_id: i64,
    pub edition_id: String,
    pub position: PlaybackMs,
    pub duration: Option<PlaybackMs>,
    pub is_finished: bool,
    pub started_at: DateTime<Utc>,
    pub finished_at: Option<DateTime<Utc>>,
    pub updated_at: DateTime<Utc>,
}
```

How far one person has listened to one edition.

**Fields**

- `user_id`: The listener (`users.id`).
- `edition_id`: The edition.
- `position`: Where they are on the edition's timeline.
- `duration`: The edition's runtime as the last player reported it.
- `is_finished`: True once they reached the end of this edition.
- `started_at`: When they first played it.
- `finished_at`: When they last reached the end.
- `updated_at`: When the row last changed.

Implements: `Clone`, `Debug`, `Eq`, `PartialEq`

### PlaybackProgressEdit

```rust
pub struct PlaybackProgressEdit {
    pub position: Option<PlaybackMs>,
    pub duration: Option<PlaybackMs>,
    pub finished: Option<bool>,
}
```

Progress to set outright ([`LibraryStore::set_playback_progress`](#librarystoreset_playback_progress)): a
seek from a page with no session, Mark finished, or Start over.

**Fields**

- `position`: The position to store; `None` keeps it.
- `duration`: The runtime to store; `None` keeps it.
- `finished`: `Some(true)` marks the edition (and its work) finished,
  `Some(false)` clears the edition's flag only, `None` decides from
  the position and runtime.

Implements: `Clone`, `Copy`, `Debug`, `Eq`, `PartialEq`

### PlaybackSession

```rust
pub struct PlaybackSession {
    pub id: String,
    pub user_id: i64,
    pub edition_id: String,
    pub serving_node_id: String,
    pub destination: String,
    pub files_digest: String,
    pub client_kind: PlayerKind,
    pub client_name: String,
    pub start_position: PlaybackMs,
    pub position: PlaybackMs,
    pub listened: PlaybackMs,
    pub started_at: DateTime<Utc>,
    pub updated_at: DateTime<Utc>,
    pub closed_at: Option<DateTime<Utc>>,
}
```

One stretch of listening on one player.

**Fields**

- `id`: Session id (hyphenated UUID).
- `user_id`: The listener (`users.id`).
- `edition_id`: The edition being played.
- `serving_node_id`: The node that opened the session and streams its files.
- `destination`: The destination the files are read from.
- `files_digest`: Digest of the edition's file rows when the session opened.
- `client_kind`: What kind of player opened it.
- `client_name`: The player's own name for itself.
- `start_position`: Where the session started from.
- `position`: Where it was at the last sync.
- `listened`: Time spent listening in it, as the player counted.
- `started_at`: When it opened.
- `updated_at`: When it last synced.
- `closed_at`: When the player closed it; `None` while open.

Implements: `Clone`, `Debug`, `Eq`, `PartialEq`

### PlaybackSync

```rust
pub struct PlaybackSync {
    pub position: PlaybackMs,
    pub listened: PlaybackMs,
    pub duration: Option<PlaybackMs>,
    pub close: bool,
}
```

What a player reports about a playing session
([`LibraryStore::sync_playback_session`](#librarystoresync_playback_session)).

**Fields**

- `position`: Where the listener is now.
- `listened`: Time listened since the last sync of this session.
- `duration`: The recording's whole runtime, when the player knows it.
- `close`: True when this is the session's last report: it closes.

Implements: `Clone`, `Copy`, `Debug`, `Eq`, `PartialEq`

### ProgressCursor

```rust
pub struct ProgressCursor {
    pub updated_at: String,
    pub edition_id: String,
}
```

A page position of [`LibraryStore::list_playback_progress`]: the last
row of the page before.

**Fields**

- `updated_at`: That row's `updated_at`, as stored.
- `edition_id`: That row's edition.

Implements: `Clone`, `Debug`, `Eq`, `PartialEq`

### BookmarkEditBody

```rust
pub struct BookmarkEditBody {
    /* private fields */
}
```

`PUT /api/playback/bookmarks/{id}` body: what to change. A field left
out keeps its stored value.

#### BookmarkEditBody::validate

```rust
pub fn validate(self) -> Result<BookmarkEditRequest>
```

Checks the body.

##### Errors

Returns [`LibraryError::Invalid`](#libraryerror) for a position past
[`MAX_PLAYBACK_MS`](#max_playback_ms), or a title that is too long or has a control
character.

Implements: `ComposeSchema`, `Debug`, `Default`, `Deserialize<'de>`, `ToSchema`

### BookmarkEditRequest

```rust
pub struct BookmarkEditRequest {
    pub position: Option<PlaybackMs>,
    pub title: Option<String>,
}
```

A checked change to a bookmark.

**Fields**

- `position`: The new place; `None` keeps it.
- `title`: The new note, trimmed; `None` keeps it.

Implements: `Clone`, `Debug`, `Eq`, `PartialEq`

### NewBookmarkBody

```rust
pub struct NewBookmarkBody {
    /* private fields */
}
```

`POST /api/library/editions/{id}/bookmarks` body.

#### NewBookmarkBody::validate

```rust
pub fn validate(self) -> Result<NewBookmarkRequest>
```

Checks the body.

##### Errors

Returns [`LibraryError::Invalid`](#libraryerror) for a position past
[`MAX_PLAYBACK_MS`](#max_playback_ms), or a title that is too long or has a control
character.

Implements: `ComposeSchema`, `Debug`, `Deserialize<'de>`, `ToSchema`

### NewBookmarkRequest

```rust
pub struct NewBookmarkRequest {
    pub position: PlaybackMs,
    pub title: String,
}
```

A checked bookmark to add.

**Fields**

- `position`: The place to mark.
- `title`: The note, trimmed; may be empty.

Implements: `Clone`, `Debug`, `Eq`, `PartialEq`

### OpenSessionBody

```rust
pub struct OpenSessionBody {
    /* private fields */
}
```

`POST /api/playback/sessions` body.

#### OpenSessionBody::validate

```rust
pub fn validate(self) -> Result<OpenSessionRequest>
```

Checks the body and tidies the player's name.

##### Errors

Returns [`LibraryError::Invalid`](#libraryerror) for an empty edition id or one
over [`MAX_EDITION_ID_BYTES`](#max_edition_id_bytes).

Implements: `ComposeSchema`, `Debug`, `Deserialize<'de>`, `ToSchema`

### OpenSessionRequest

```rust
pub struct OpenSessionRequest {
    pub edition_id: String,
    pub player: PlayerKind,
    pub player_name: String,
}
```

A checked request to open a play session.

**Fields**

- `edition_id`: The edition to play, as sent.
- `player`: What kind of player asks.
- `player_name`: The player's name as stored: no control characters, trimmed, at
  most [`MAX_PLAYER_NAME_BYTES`](#max_player_name_bytes) bytes.

Implements: `Clone`, `Debug`, `Eq`, `PartialEq`

### PlaybackSyncBody

```rust
pub struct PlaybackSyncBody {
    /* private fields */
}
```

`POST /api/playback/sessions/{id}/sync` and `.../close` body: where
the listener is.

#### PlaybackSyncBody::validate

```rust
pub fn validate(self, close: bool) -> Result<PlaybackSync>
```

Checks the body: every length at most [`MAX_PLAYBACK_MS`](#max_playback_ms).

##### Arguments

* `close` - True for the `close` route: the report ends the session.

##### Errors

Returns [`LibraryError::Invalid`](#libraryerror) naming the field that is too
large.

Implements: `ComposeSchema`, `Debug`, `Deserialize<'de>`, `ToSchema`

### ProgressEditBody

```rust
pub struct ProgressEditBody {
    /* private fields */
}
```

`PUT /api/playback/progress/{edition_id}` body: what to set. A field
left out keeps its stored value.

#### ProgressEditBody::validate

```rust
pub fn validate(self) -> Result<PlaybackProgressEdit>
```

Checks the body: every length at most [`MAX_PLAYBACK_MS`](#max_playback_ms).

##### Errors

Returns [`LibraryError::Invalid`](#libraryerror) naming the field that is too
large.

Implements: `ComposeSchema`, `Debug`, `Default`, `Deserialize<'de>`, `ToSchema`

### TrackPath

```rust
pub struct TrackPath {
    pub session_id: String,
    pub index: usize,
}
```

The file a stream request names ([`parse_track_path`](#parse_track_path)).

**Fields**

- `session_id`: The play session's id: a hyphenated lowercase UUID.
- `index`: The track's index in the session's list, below
  [`MAX_PLAYBACK_TRACKS`](#max_playback_tracks).

Implements: `Clone`, `Debug`, `Eq`, `PartialEq`

### NewPlayerCard

```rust
pub struct NewPlayerCard<'a> {
    pub identity_id: i64,
    pub session_hash: &'a str,
    pub password_id: Option<&'a str>,
    pub label: &'a str,
    pub user_agent: Option<&'a str>,
    pub device_type: &'a str,
    pub tokens: PlayerCardTokens<'a>,
}
```

A new player card ([`LibraryStore::mint_player_card`](#librarystoremint_player_card)).

**Fields**

- `identity_id`: The portal identity the card signs in as.
- `session_hash`: SHA-256 hex of random bytes nobody is given: the session row's own
  token hash, unique per card.
- `password_id`: The player password it was made with, when it was.
- `label`: The app's name, shown in the session list.
- `user_agent`: The app's `User-Agent`, when it sent one.
- `device_type`: Best-effort device class (`mobile`, `tablet`, `desktop`, `api`).
- `tokens`: The card's first tokens.

Implements: `Clone`, `Copy`, `Debug`

### PlayerCard

```rust
pub struct PlayerCard {
    pub session_id: i64,
    pub identity: PortalIdentity,
}
```

A live player card, as an access token resolves to it.

**Fields**

- `session_id`: The card's `portal_sessions.id`.
- `identity`: The identity it signs in as.

Implements: `Clone`, `Debug`

### PlayerCardTokens

```rust
pub struct PlayerCardTokens<'a> {
    pub access_hash: &'a str,
    pub refresh_hash: &'a str,
}
```

What signing a card in or refreshing it hands out, as digests: the
caller holds the tokens themselves.

**Fields**

- `access_hash`: SHA-256 hex of the new access token.
- `refresh_hash`: SHA-256 hex of the new refresh token.

Implements: `Clone`, `Copy`, `Debug`

### PlayerPassword

```rust
pub struct PlayerPassword {
    pub id: String,
    pub user_id: i64,
    pub label: String,
    pub created_at: DateTime<Utc>,
    pub last_used_at: Option<DateTime<Utc>>,
}
```

A player password as its owner sees it in Account settings.

**Fields**

- `id`: Password id.
- `user_id`: The patron it signs in as.
- `label`: The name the patron gave it.
- `created_at`: When it was made.
- `last_used_at`: When it last signed an app in.

Implements: `Clone`, `Debug`, `Eq`, `PartialEq`

### PlayerFacet

```rust
pub struct PlayerFacet {
    pub key: String,
    pub value: String,
}
```

One value of the page index on an item: its key and how it is written.

**Fields**

- `key`: The value's key ([`crate::facet_key`](#facet_key)), what a filter matches.
- `value`: The value as shown.

Implements: `Clone`, `Debug`, `Eq`, `PartialEq`

### PlayerItem

```rust
pub struct PlayerItem {
    pub edition_id: String,
    pub work_id: String,
    pub title: String,
    pub subtitle: Option<String>,
    pub description: Option<String>,
    pub published_at: Option<String>,
    pub runtime_minutes: Option<u64>,
    pub language: Option<String>,
    pub abridged: bool,
    pub authors: Vec<PlayerFacet>,
    pub narrators: Vec<PlayerFacet>,
    pub series: Vec<PlayerFacet>,
    pub series_position: Option<String>,
    pub genres: Vec<String>,
    pub publishers: Vec<String>,
    pub isbn: Option<String>,
    pub asin: Option<String>,
    pub added_at: DateTime<Utc>,
    pub updated_at: DateTime<Utc>,
}
```

What the player API shows of one item ([`LibraryStore::player_items`](#librarystoreplayer_items)).

**Fields**

- `edition_id`: The edition: the item id.
- `work_id`: Its work.
- `title`: The edition's title.
- `subtitle`: Its subtitle.
- `description`: The work's description (may hold markup).
- `published_at`: The publication date as the store gave it.
- `runtime_minutes`: The runtime the catalog states, in whole minutes.
- `language`: The expression's language tag.
- `abridged`: True when the expression is abridged.
- `authors`: Authors, in the order the work's row lists them.
- `narrators`: Narrators.
- `series`: Series, the work's first series first.
- `series_position`: The work's place in its first series, as written.
- `genres`: Genres.
- `publishers`: Publishers.
- `isbn`: The edition's ISBN, when known.
- `asin`: The edition's ASIN, when known.
- `added_at`: When the library acquired it.
- `updated_at`: When its record last changed.

Implements: `Clone`, `Debug`, `PartialEq`

### PluginDatabaseRecord

```rust
pub struct PluginDatabaseRecord {
    pub plugin_id: String,
    pub binding: String,
    pub adapter_plugin_key: String,
    pub backend_kind: String,
    pub unit_ref: String,
    pub created_at: String,
}
```

One provisioned plugin database binding registry row.

**Fields**

- `plugin_id`: Owning plugin PluginKey (canonical text).
- `binding`: Binding name from `plugin.toml` `capabilities.bindings.databases`.
- `adapter_plugin_key`: Adapter PluginKey that provisioned the physical unit.
- `backend_kind`: Diagnostic adapter family (`sqlite`, `postgres`, `d1`, or custom).
- `unit_ref`: Backend-native unit: file path, schema name, or D1 database name.
- `created_at`: RFC 3339 provisioning time.

Implements: `Clone`, `Debug`, `Eq`, `PartialEq`

### KvEntry

```rust
pub struct KvEntry {
    pub key: String,
    pub value: Vec<u8>,
    pub metadata: String,
    pub expires_at: Option<DateTime<Utc>>,
}
```

One live key/value row.

**Fields**

- `key`: Key within the namespace.
- `value`: Value bytes.
- `metadata`: Opaque JSON metadata text; empty when none.
- `expires_at`: Absolute expiry, when set.

Implements: `Clone`, `Debug`, `Eq`, `PartialEq`

### KvListPage

```rust
pub struct KvListPage {
    pub keys: Vec<KvEntry>,
    pub list_complete: bool,
    pub cursor: String,
}
```

One page of [`LibraryStore::kv_list`](#librarystorekv_list).

**Fields**

- `keys`: Keys in lexical order (values are not loaded).
- `list_complete`: True when no further page exists.
- `cursor`: Opaque cursor for the next page; empty when complete.

Implements: `Clone`, `Debug`, `Default`, `Eq`, `PartialEq`

### KvWriteSpec

```rust
pub struct KvWriteSpec {
    pub value: Vec<u8>,
    pub metadata: String,
    pub expires_at: Option<DateTime<Utc>>,
    pub max_value_bytes: u32,
    pub max_keys: u32,
}
```

A write plus the namespace limits it must satisfy.

**Fields**

- `value`: Value bytes.
- `metadata`: Opaque JSON metadata text; empty when none.
- `expires_at`: Absolute expiry; `None` never expires.
- `max_value_bytes`: Largest value this namespace accepts.
- `max_keys`: Most live keys this namespace holds.

Implements: `Clone`, `Debug`, `Eq`, `PartialEq`

### PassStart

```rust
pub struct PassStart {
    pub cursor: String,
    pub sync_token: String,
    pub resumed: bool,
    pub pass_id: String,
}
```

How the next pass of one account starts.

**Fields**

- `cursor`: Cursor of the first page to ask for; empty starts at the beginning.
- `sync_token`: Sync token to send; empty asks for a full pass.
- `resumed`: True when the pass resumes one that stopped part way.
- `pass_id`: Id of the pass, stamped on every holding a page lists
  (`holdings.seen_pass`); a resumed pass keeps the id its earlier
  pages stamped.

Implements: `Clone`, `Debug`, `Eq`, `PartialEq`

### QualityCheckState

```rust
pub struct QualityCheckState {
    pub cursor: String,
    pub checked_at: Option<DateTime<Utc>>,
}
```

Where the quality-upgrade check of one account stands.

**Fields**

- `cursor`: Holding id the next check continues after; empty to start from the
  account's first holding.
- `checked_at`: When the check last ran; `None` before it first ran.

Implements: `Clone`, `Debug`, `Default`, `Eq`, `PartialEq`

### ScanState

```rust
pub struct ScanState {
    pub sync_token: String,
    pub resume_cursor: String,
    pub resume_token: String,
    pub last_complete_at: Option<DateTime<Utc>>,
    pub pass_id: String,
    pub updated_at: DateTime<Utc>,
}
```

Where one account's scan stands.

**Fields**

- `sync_token`: Opaque token from the end of the last pass; empty for none.
- `resume_cursor`: Cursor of a stopped pass's next page; empty when none.
- `resume_token`: Token the stopped pass started with.
- `last_complete_at`: When the last complete pass ended.
- `pass_id`: Id of the pass in progress or last run; empty before the first.
- `updated_at`: When the row was last written.

Implements: `Clone`, `Debug`, `Eq`, `From<Model>`, `PartialEq`

### UserStateEdit

```rust
pub struct UserStateEdit {
    pub is_finished: Option<bool>,
    pub rating_overall: FieldEdit<f64>,
    pub rating_performance: FieldEdit<f64>,
    pub rating_story: FieldEdit<f64>,
    pub tags: FieldEdit<String>,
}
```

A change to one user's state for one work, applied by
[`LibraryStore::edit_user_title_state`](#librarystoreedit_user_title_state).

**Fields**

- `is_finished`: The finished flag to store; `None` keeps it (a new row starts
  unfinished).
- `rating_overall`: Overall rating, 0 to 5.
- `rating_performance`: Narration rating, 0 to 5.
- `rating_story`: Story rating, 0 to 5.
- `tags`: Tags, whitespace-separated.

Implements: `Clone`, `Debug`, `Default`, `PartialEq`

### WorkSplitBody

```rust
pub struct WorkSplitBody {
    /* private fields */
}
```

`POST /api/library/works/{id}/split` body: the expressions to move.

#### WorkSplitBody::validate

```rust
pub fn validate(self) -> Result<WorkSplitRequest>
```

Checks the body: every id 1 to [`MAX_SPLIT_EXPRESSION_ID_BYTES`](#max_split_expression_id_bytes)
bytes, and 1 to [`MAX_SPLIT_EXPRESSIONS`](#max_split_expressions) distinct ids (a repeated
id counts once).

##### Errors

Returns [`LibraryError::Invalid`](#libraryerror) for an empty or overlong id, or a
count outside the bounds.

Implements: `ComposeSchema`, `Debug`, `Deserialize<'de>`, `ToSchema`

### WorkSplitRequest

```rust
pub struct WorkSplitRequest {
    /* private fields */
}
```

A validated [`WorkSplitBody`]: 1 to [`MAX_SPLIT_EXPRESSIONS`](#max_split_expressions) distinct
expression ids, each 1 to [`MAX_SPLIT_EXPRESSION_ID_BYTES`](#max_split_expression_id_bytes) bytes,
sorted.

#### WorkSplitRequest::expression_ids

```rust
pub fn expression_ids(&self) -> &[String]
```

The expressions to move, sorted and distinct.

Implements: `Clone`, `Debug`, `Eq`, `PartialEq`

### AcquireTitle

```rust
pub struct AcquireTitle {
    pub holding_id: String,
    pub edition_id: String,
    pub listing_id: String,
    pub plugin_key: String,
    pub source: String,
    pub account_id: String,
    pub marketplace: String,
    pub product_id: String,
    pub asin: Option<String>,
    pub isbn: Option<String>,
    pub title: String,
    pub subtitle: Option<String>,
    pub authors: Option<String>,
    pub narrators: Option<String>,
    pub series: Option<String>,
    pub series_index: Option<String>,
    pub series_product_id: Option<String>,
    pub publisher: Option<String>,
    pub categories: Option<String>,
    pub length_minutes: Option<i64>,
    pub language: Option<String>,
    pub is_abridged: bool,
    pub content_kind: &'static str,
    pub purchased_at: Option<DateTime<Utc>>,
    pub added_at: Option<DateTime<Utc>>,
    pub published_at: Option<DateTime<Utc>>,
    pub status: AcquireStatus,
    pub storage_key: Option<String>,
    pub acquired_by: Option<String>,
    pub pdf_status: AcquireStatus,
    pub pdf_storage_key: Option<String>,
    pub pdf_acquired_by: Option<String>,
}
```

One title an account holds, as acquire reads it from the library model.

**Fields**

- `holding_id`: `holdings.id` of the account's holding.
- `edition_id`: The edition the listing sells: the key the title's files and acquire
  state belong to.
- `listing_id`: `store_listings.id`.
- `plugin_key`: Provenance-qualified PluginKey of the store.
- `source`: Display alias of the store's plugin (`audible`, `libro`, …), as the
  account records it.
- `account_id`: The holding account's id at that store.
- `marketplace`: Lower-cased marketplace (empty for a store with one).
- `product_id`: The store's own product id: what its download and fetch APIs take.
- `asin`: The edition's ASIN, preferring one scoped to the marketplace.
- `isbn`: The edition's lowest ISBN, when it has one.
- `title`: Title as the store lists it.
- `subtitle`: Subtitle as the store lists it, else the edition's.
- `authors`: Authors as the store prints them, comma-separated.
- `narrators`: Narrators as the store prints them, comma-separated.
- `series`: Series name as the store lists it.
- `series_index`: Position in that series as the store writes it.
- `series_product_id`: The store's product id for the series or podcast show.
- `publisher`: Publisher as the store lists it.
- `categories`: Store category labels (serialized as the scan stored them).
- `length_minutes`: Running time in whole minutes, from the store or the edition.
- `language`: Language as the store lists it.
- `is_abridged`: True when the store or the expression marks it abridged.
- `content_kind`: What the title is, in the scan's `content_kind` labels: `episode` for a
  podcast episode, `podcast` for a show, `book` otherwise.
- `purchased_at`: When the account bought the title, when the store reported it.
- `added_at`: When this library first recorded the account's holding of the
  title; the added date when the store reports no purchase time.
- `published_at`: Publication time as the store reported it, else the edition's.
- `status`: The edition's acquire state; not acquired when no title of it was
  ever queued.
- `storage_key`: Primary destination key of the edition's audio once acquired.
- `acquired_by`: The account holding (`holdings.id`) whose title's write set the
  edition's audio state; `None` when no title of it was ever queued,
  or when that holding no longer exists (its account was revoked or
  its title relinked), so no title owns the state any more.
- `pdf_status`: The edition's companion PDF state.
- `pdf_storage_key`: Destination key of the companion PDF once acquired.
- `pdf_acquired_by`: The account holding (`holdings.id`) whose title's write set the
  PDF state; `None` as for [`Self::acquired_by`](#acquiretitle).

#### AcquireTitle::acquired_here

```rust
pub fn acquired_here(&self) -> bool
```

True when this title's holding set the edition's audio state: the
one whose writes change it once acquired.

#### AcquireTitle::pdf_acquired_here

```rust
pub fn pdf_acquired_here(&self) -> bool
```

True when this title's holding set the edition's companion PDF
state.

#### AcquireTitle::asin_or_isbn

```rust
pub fn asin_or_isbn(&self) -> &str
```

Naming and display fallback: ASIN, else ISBN, else the store's
product id.

Implements: `Clone`, `Debug`, `PartialEq`

### AcquireTitlePage

```rust
pub struct AcquireTitlePage {
    pub titles: Vec<AcquireTitle>,
    pub next: Option<String>,
}
```

One page of [`LibraryStore::acquire_titles_page`](#librarystoreacquire_titles_page).

**Fields**

- `titles`: The page's titles, in holding order.
- `next`: The cursor of the next page; `None` once the walk is done.

Implements: `Clone`, `Debug`, `PartialEq`

### AcquiredTitleKey

```rust
pub struct AcquiredTitleKey {
    pub holding_id: String,
    pub edition_id: String,
    pub source: String,
    pub plugin_key: String,
    pub account_id: String,
    pub marketplace: String,
    pub product_id: String,
    pub updated_at: DateTime<Utc>,
}
```

One account's title of an edition acquired with a stored file, without
its metadata.

**Fields**

- `holding_id`: The account holding (`holdings.id`): the id the full view is read
  by.
- `edition_id`: The acquired edition (`editions.id`).
- `source`: Display alias of the store plugin (`audible`, `libro`, ...), from
  the account; empty when the account row is gone.
- `plugin_key`: Provenance-qualified plugin key of the store.
- `account_id`: Store account that holds the title.
- `marketplace`: Store marketplace of the listing, as stored.
- `product_id`: Store product id of the listing.
- `updated_at`: Last change of the title: the latest of its holding, its listing and
  its edition's acquire state. An unparsable stored value counts as
  the time of the read.

Implements: `Clone`, `Debug`, `Eq`, `PartialEq`

### AcquiredTitleKeyPage

```rust
pub struct AcquiredTitleKeyPage {
    pub keys: Vec<AcquiredTitleKey>,
    pub next: Option<String>,
}
```

One page of [`AcquiredTitleKey`](#acquiredtitlekey)s.

**Fields**

- `keys`: Every account title of the page's editions, by edition then holding
  id.
- `next`: The cursor of the next page (an edition id), or `None` after the
  last.

Implements: `Clone`, `Debug`, `Default`, `Eq`, `PartialEq`

### AudioStream

```rust
pub struct AudioStream {
    pub codec: Option<AudioCodec>,
    pub format: AudioFormat,
    pub duration_ms: Option<u64>,
}
```

What a probe found about the audio stream of one file, besides its
[`FileProbe`]: the encoding, channels, sample rate and the running time
a player sees.

Each value is `None` when the file's headers do not state it; nothing
is decoded to find them.

**Fields**

- `codec`: The audio encoding.
- `format`: Channels, sample rate and, for AAC, the profile.
- `duration_ms`: The playable running time in milliseconds, as the container states
  it after edit lists and encoder priming; never zero
  ([`Self::is_well_formed`](#audiostreamis_well_formed)).

#### AudioStream::is_well_formed

```rust
pub fn is_well_formed(&self) -> bool
```

True when every stated value is in range: channels and a sample
rate above zero and within the tag reader's limits, an AAC profile
only on AAC, and a running time above zero that a database integer
holds and [`bookclerk_tags::MAX_DURATION_MS`](../bookclerk_tags/README.md#max_duration_ms) allows.

Implements: `Clone`, `Copy`, `Debug`, `Default`, `Eq`, `PartialEq`

### CandidateRef

```rust
pub struct CandidateRef {
    pub listing_id: String,
    pub origin: String,
    pub seed_title: Option<String>,
    pub seed_categories: Option<String>,
    pub starter_genre: Option<String>,
}
```

One title a store returned for a subject, as written: the shared
listing it is, plus why the subject got it.

**Fields**

- `listing_id`: The `store_listings` row.
- `origin`: How the store found it (`related to …`, `deals`).
- `seed_title`: The subject's title that seeded it, when a seed did.
- `seed_categories`: That seed's categories.
- `starter_genre`: Starter genre it was browsed from, for a thin taste.

Implements: `Clone`, `Debug`, `Eq`, `PartialEq`

### ChangedObject

```rust
pub struct ChangedObject {
    pub key: String,
    pub changed_seq: i64,
    pub removed: bool,
    pub size: u64,
}
```

One object whose row changed after a reader's last sequence.

**Fields**

- `key`: The object's key at the destination.
- `changed_seq`: The change sequence that changed the row.
- `removed`: True when the object is gone.
- `size`: Size in bytes when last seen.

Implements: `Clone`, `Debug`, `Eq`, `PartialEq`

### DestinationFeed

```rust
pub struct DestinationFeed {
    pub destination: String,
    pub seq: i64,
    pub feed_state: FeedState,
    pub feed_cursor: Option<String>,
    pub feed_error: Option<String>,
    pub feed_checked_at: Option<DateTime<Utc>>,
    pub listing_seq: Option<i64>,
    pub listing_cursor: Option<String>,
    pub listing_started_at: Option<DateTime<Utc>>,
    pub listed_at: Option<DateTime<Utc>>,
    pub next_listing_at: Option<DateTime<Utc>>,
    pub listing_error: Option<String>,
    pub objects: u64,
    pub checked_seq: i64,
    pub listing_base: Option<u64>,
    pub revision: i64,
}
```

How one destination's outside changes are followed.

**Fields**

- `destination`: The destination's plugin instance id.
- `seq`: The last change sequence number handed out.
- `feed_state`: The change feed's state.
- `feed_cursor`: The feed cursor to read from next; `None` before the first read.
- `feed_error`: The last feed failure.
- `feed_checked_at`: When the feed was last read.
- `listing_seq`: Sequence of the listing in progress; `None` between listings.
- `listing_cursor`: Cursor of the next page of the listing in progress.
- `listing_started_at`: When the listing in progress, or the last one, started.
- `listed_at`: When the last listing finished.
- `next_listing_at`: When the next scheduled listing is due.
- `listing_error`: Why the last listing failed or its removals were not applied.
- `objects`: Objects the destination held after the last listing or feed page.
- `checked_seq`: The last sequence the copy check read changes up to.
- `listing_base`: Objects known when the listing in progress started, which its
  removal guard is judged against; `None` between listings.
- `revision`: The row revision this value was read at (0 before the first save);
  [`LibraryStore::save_destination_feed`](#librarystoresave_destination_feed) refuses a save from an older
  one.

#### DestinationFeed::new

```rust
pub fn new(destination: &str) -> Self
```

A destination never followed before: no feed read, no listing.

#### DestinationFeed::next_seq

```rust
pub fn next_seq(&mut self) -> i64
```

Takes the next change sequence number.

#### DestinationFeed::listing_due

```rust
pub fn listing_due(&self, now: DateTime<Utc>) -> bool
```

Whether a listing should run at `now`: one is in progress, none has
finished yet, the feed is not healthy while a feed is in use, or the
scheduled one is due.

Implements: `Clone`, `Debug`, `Eq`, `PartialEq`

### EditionAcquisition

```rust
pub struct EditionAcquisition {
    pub edition_id: String,
    pub status: AcquireStatus,
    pub storage_key: Option<String>,
    pub error_message: Option<String>,
    pub holding_id: String,
    pub updated_at: String,
    pub pdf_status: AcquireStatus,
    pub pdf_storage_key: Option<String>,
    pub pdf_holding_id: Option<String>,
}
```

An edition's acquire state (`edition_acquisitions`).

**Fields**

- `edition_id`: The edition id.
- `status`: Where the edition's acquire stands; a status this build does not
  know reads as [`AcquireStatus::NotAcquired`](#acquirestatus).
- `storage_key`: Primary destination key of the edition's audio once acquired.
- `error_message`: Last acquire failure message.
- `holding_id`: The account holding (`holdings.id`) whose title's write set the
  state.
- `updated_at`: RFC 3339 time of that write.
- `pdf_status`: Where the edition's companion PDF acquire stands; a status this
  build does not know reads as [`AcquireStatus::NotAcquired`](#acquirestatus).
- `pdf_storage_key`: Destination key of the companion PDF once acquired.
- `pdf_holding_id`: The account holding (`holdings.id`) whose title's write set the PDF
  state; `None` until a PDF write happens.

Implements: `Clone`, `Debug`, `Eq`, `From<Model>`, `PartialEq`

### EditionAsin

```rust
pub struct EditionAsin {
    pub value: String,
    pub marketplace: String,
}
```

An ASIN a catalog match found for an edition.

**Fields**

- `value`: The ASIN as the catalog printed it (normalised when stored).
- `marketplace`: The marketplace the ASIN is sold in (such as `us`).

Implements: `Clone`, `Debug`, `Eq`, `PartialEq`

### EditionCopy

```rust
pub struct EditionCopy {
    pub edition_id: String,
    pub destination: String,
    pub state: CopyState,
    pub storage_key: Option<String>,
    pub attempts: u32,
    pub next_attempt_at: Option<DateTime<Utc>>,
    pub last_error: Option<String>,
}
```

One edition's copy at one destination.

**Fields**

- `edition_id`: The edition (`editions.id`).
- `destination`: The destination's plugin instance id.
- `state`: Where the copy stands.
- `storage_key`: The edition's primary audio key on the destination, once a write
  there was stored.
- `attempts`: Failed writes since the copy was last stored.
- `next_attempt_at`: When the next background attempt is due; `None` unless pending.
- `last_error`: The last write failure; `None` once stored.

#### EditionCopy::wants_attempt

```rust
pub fn wants_attempt(&self, now: DateTime<Utc>) -> bool
```

Whether an acquire at `now` should write this copy: a pending copy
whose delay has passed. A stored copy needs nothing, a failed or
missing one waits for an acquire of its title, and a pending one
waits out its delay.

Implements: `Clone`, `Debug`, `Eq`, `PartialEq`

### EditionEnrichment

```rust
pub struct EditionEnrichment {
    pub asin: Option<EditionAsin>,
    pub description: Option<String>,
    pub subjects: Option<String>,
    pub categories: Option<String>,
    pub cover_url: Option<String>,
    pub language: Option<String>,
    pub source: String,
    pub confidence: f64,
}
```

What an enricher fills on one edition; `None` fields are left alone.

**Fields**

- `asin`: An ASIN to record as the edition's identifier. An ASIN another
  edition already holds stays there.
- `description`: Description for the work, when it has none.
- `subjects`: Subjects for the work, when it has none.
- `categories`: Category labels for the work, when it has none.
- `cover_url`: Cover for the edition (and the work), when it has none.
- `language`: Language for the expression, when it has none.
- `source`: Who filled it, recorded as `editions.enrich_source`.
- `confidence`: How sure the enricher was, from 0.0 to 1.0.

Implements: `Clone`, `Debug`, `Default`, `PartialEq`

### EditionFile

```rust
pub struct EditionFile {
    pub owner: FileOwner,
    pub writer: String,
    pub role: FileRole,
    pub sha256: Option<String>,
}
```

Who holds a recorded file key, and what the file is.

**Fields**

- `owner`: The edition (or branch title) the file belongs to.
- `writer`: The host upload id that claimed the key last, or [`ACQUIRE_WRITER`](#acquire_writer).
- `role`: What the file is.
- `sha256`: Lowercase hex SHA-256 of the content, when known (host uploads).

Implements: `Clone`, `Debug`, `Eq`, `PartialEq`

### EnrichCandidate

```rust
pub struct EnrichCandidate {
    pub edition_id: String,
    pub work_id: Option<String>,
    pub title: String,
    pub subtitle: Option<String>,
    pub authors: Option<String>,
    pub narrators: Option<String>,
    pub duration_minutes: Option<i64>,
    pub marketplace: String,
    pub isbn: Option<String>,
    pub asin: Option<String>,
    pub description: Option<String>,
    pub subjects: Option<String>,
    pub cover_url: Option<String>,
    pub language: Option<String>,
    pub enrich_source: Option<String>,
}
```

One held edition, with what an enricher needs to look it up and the
fields it may fill.

**Fields**

- `edition_id`: `editions.id`.
- `work_id`: The edition's work (`works.id`), when its expression still names one.
- `title`: The edition's display title.
- `subtitle`: The edition's subtitle, else a listing's.
- `authors`: The credited authors as the first listing of the edition prints
  them, when it does.
- `narrators`: The credited narrators as the first listing of the edition prints
  them, when it does.
- `duration_minutes`: The edition's running time in whole minutes, else a listing's.
- `marketplace`: The marketplace of the edition's first listing (lower case, such as
  `us`), which scopes an ASIN found for it.
- `isbn`: The edition's ISBN (the smallest when it has several).
- `asin`: The edition's ASIN (the smallest when it has several), which a
  store listing or an earlier catalog match recorded.
- `description`: The work's description, else a listing's.
- `subjects`: The work's subjects.
- `cover_url`: The edition's cover, else a listing's.
- `language`: The expression's language, else a listing's.
- `enrich_source`: Who already filled the edition (`editions.enrich_source`).

Implements: `Clone`, `Debug`, `PartialEq`

### EnrichCandidatePage

```rust
pub struct EnrichCandidatePage {
    pub candidates: Vec<EnrichCandidate>,
    pub next: Option<String>,
}
```

One page of [`LibraryStore::enrich_candidates_page`](#librarystoreenrich_candidates_page).

**Fields**

- `candidates`: The page's held editions, in edition id order.
- `next`: The cursor of the next page; `None` once the walk is done.

Implements: `Clone`, `Debug`, `PartialEq`

### FileProbe

```rust
pub struct FileProbe {
    pub content_type: String,
    pub bitrate_kbps: Option<u32>,
    pub lossless: bool,
    pub source_store: Option<String>,
}
```

What a probe found in one stored audio file, and where the copy came
from (`edition_files`).

The same values decide which of two copies of one edition is kept
([`FileProbe::is_better_than`](#fileprobeis_better_than)), wherever the comparison runs.

**Fields**

- `content_type`: Media type of the file (`audio/mp4`, `audio/mpeg`, `audio/flac`).
- `bitrate_kbps`: Average audio bitrate in kilobits per second; `None` when the file
  states no running time to divide its size by.
- `lossless`: True for a lossless encoding (FLAC, ALAC).
- `source_store`: Plain id of the storefront the copy was downloaded from, when known.

#### FileProbe::MAX_CONTENT_TYPE_BYTES

```rust
pub const MAX_CONTENT_TYPE_BYTES: usize = 64
```

Longest media type a probe may carry, in bytes.

#### FileProbe::MAX_BITRATE_KBPS

```rust
pub const MAX_BITRATE_KBPS: u32 = 20_000
```

Highest bitrate a probe may carry, in kilobits per second: above any
audiobook encoding, so a forged value cannot win by size alone.

#### FileProbe::is_better_than

```rust
pub fn is_better_than(&self, other: &Self) -> bool
```

True when a copy with this probe should replace one with `other`
(library data model ADR, section 3): a lossless copy beats a lossy
one, and within the same kind the higher bitrate wins. A copy whose
bitrate is unknown never replaces one whose bitrate is known, and
equal copies do not replace each other.

#### FileProbe::is_well_formed

```rust
pub fn is_well_formed(&self) -> bool
```

True when the values are ones an honest probe can produce: a short
media type of visible ASCII, a bitrate within
[`Self::MAX_BITRATE_KBPS`](#fileprobemax_bitrate_kbps), and a storefront that is a store's plain
id (lowercase letters, digits, `_`, `-` and `.`, as node frames name
stores).

Implements: `Clone`, `Debug`, `Eq`, `PartialEq`

### HeldIdentifiers

```rust
pub struct HeldIdentifiers {
    pub asins: BTreeSet<String>,
    pub isbns: BTreeSet<String>,
    pub product_ids: BTreeSet<String>,
}
```

Which of a set of store identifiers belong to titles the library holds.

**Fields**

- `asins`: Upper-cased ASINs of held editions.
- `isbns`: ISBNs of held editions, as stored (canonical ISBN-13).
- `product_ids`: Store product ids of held listings.

Implements: `Clone`, `Debug`, `Default`, `Eq`, `PartialEq`

### LibraryStore

```rust
pub struct LibraryStore {
    /* private fields */
}
```

Handle to the Bookclerk library database.

Async API over a SeaORM [`DatabaseConnection`](https://docs.rs/sea_orm/latest/sea_orm/database/db_connection/struct.DatabaseConnection.html). `DatabaseConnection` is
cheaply cloneable (shared connection), so `LibraryStore` is `Clone`.

#### LibraryStore::scope

```rust
pub fn scope(
    &self,
    plugin_key: impl Into<String>,
    source_id: impl Into<String>,
) -> SourceScope
```

Plugin-scoped view — first-party and third-party sources use this.

`plugin_key` isolates rows; `source_id` is the display alias written
to the `source` column.

#### LibraryStore::acquire_title

```rust
pub async fn acquire_title(
    &self,
    plugin_key: &str,
    account_id: &str,
    marketplace: &str,
    product_id: &str,
) -> Result<Option<AcquireTitle>>
```

The title `account_id` holds at a store, as acquire reads it.

##### Arguments

* `plugin_key` - The store's PluginKey.
* `account_id` - The account at that store.
* `marketplace` - The store's marketplace (any case; blank for one).
* `product_id` - The store's product id.

##### Returns

`None` when the store's listing is not resolved yet or the account
does not hold it.

##### Errors

Returns [`LibraryError::Orm`](#libraryerror) when a read fails.

#### LibraryStore::acquire_titles_named

```rust
pub async fn acquire_titles_named(
    &self,
    account_id: Option<&str>,
    id: &str,
) -> Result<Vec<AcquireTitle>>
```

The titles accounts hold that `id` names, as acquire reads them, in
holding order: what an acquire job given one title visits.

`id` matches, ignoring ASCII case, a listing's store product id, an
ASIN or ISBN of the listing's edition (every listing of that
edition), or a holding id.

##### Arguments

* `account_id` - Only the titles of the accounts this id or label
  names (ignoring ASCII case), when set.
* `id` - The product id, ASIN, ISBN, or holding id.

##### Returns

Every matching account holding's title; empty when nothing matches.

##### Errors

Returns [`LibraryError::Orm`](#libraryerror) when a read fails.

#### LibraryStore::acquire_titles_of_listings

```rust
pub async fn acquire_titles_of_listings(
    &self,
    listing_ids: &[String],
) -> Result<Vec<AcquireTitle>>
```

The titles accounts hold of the store listings `listing_ids`, as
acquire reads them, in holding order: every account's copy of those
listings.

##### Arguments

* `listing_ids` - `store_listings.id` values.

##### Returns

One title per account holding of a listing; empty when none is held.

##### Errors

Returns [`LibraryError::Orm`](#libraryerror) when a read fails.

#### LibraryStore::held_titles_of_edition

```rust
pub async fn held_titles_of_edition(&self, edition_id: &str) -> Result<Vec<AcquireTitle>>
```

The held titles of edition `edition_id`: one per account holding of
any of its listings, in holding id order.

##### Errors

Returns [`LibraryError::Orm`](#libraryerror) when a read fails.

#### LibraryStore::acquire_title_of_holding

```rust
pub async fn acquire_title_of_holding(
    &self,
    holding_id: &str,
) -> Result<Option<AcquireTitle>>
```

The title of one account holding (`holdings.id`), as acquire reads
it.

##### Returns

`None` when no account holding has that id (a node holding, a
holding withdrawn, or an id this library never had) or its listing
is gone.

##### Errors

Returns [`LibraryError::Orm`](#libraryerror) when a read fails.

#### LibraryStore::accounts_holding_titles

```rust
pub async fn accounts_holding_titles(
    &self,
    account_id: Option<&str>,
    plugin_key: Option<&str>,
) -> Result<BTreeSet<(String, String)>>
```

The accounts that hold at least one title, as `(plugin_key,
account_id)`: what an acquire of every title in its scope touches.

##### Arguments

* `account_id` - Only the accounts this id or label names (ignoring
  ASCII case), when set.
* `plugin_key` - Only accounts on this plugin, when set.

##### Errors

Returns [`LibraryError::Orm`](#libraryerror) when a read fails.

#### LibraryStore::acquire_titles_page

```rust
pub async fn acquire_titles_page(
    &self,
    account_id: Option<&str>,
    after: Option<&str>,
    limit: u64,
) -> Result<AcquireTitlePage>
```

One page of the titles accounts hold, as acquire reads them, in
holding order: what reconcile and storage matching walk.

##### Arguments

* `account_id` - Only the titles of the accounts this id or label
  names (ignoring ASCII case), when set.
* `after` - The previous page's [`AcquireTitlePage::next`](#acquiretitlepage); `None`
  for the first page.
* `limit` - Most holdings the page reads (1 to 64, and at most the
  database's `maxResultRows`).

##### Returns

The page's titles and the cursor of the next page. A holding whose
listing is gone has no title, so a page can be short or empty while
more follow: walk until `next` is `None`.

##### Errors

Returns [`LibraryError::Orm`](#libraryerror) when a read fails.

#### LibraryStore::all_acquire_titles

```rust
pub async fn all_acquire_titles(
    &self,
    account_id: Option<&str>,
) -> Result<Vec<AcquireTitle>>
```

Every account title, in holding order: [`Self::acquire_titles_page`](#librarystoreacquire_titles_page)
walked to its end.

##### Arguments

* `account_id` - Only the titles of the accounts this id or label
  names (ignoring ASCII case), when set.

##### Errors

Returns [`LibraryError::Orm`](#libraryerror) when a read fails.

#### LibraryStore::acquired_title_keys_page

```rust
pub async fn acquired_title_keys_page(
    &self,
    after: Option<&str>,
    limit: u64,
) -> Result<AcquiredTitleKeyPage>
```

One keyset page of the account titles whose edition is acquired
with a stored audio file, identity columns only.

A page of editions whose result would exceed the database's result
limits is halved and read again.

##### Arguments

* `after` - The previous page's [`AcquiredTitleKeyPage::next`](#acquiredtitlekeypage), or
  `None` for the first page.
* `limit` - Editions wanted, clamped to
  1..=[`ACQUIRED_KEYS_MAX_PAGE`](#acquired_keys_max_page) and to the database's row limit.

##### Returns

Every account holding of up to `limit` acquired editions, in edition
then holding order. An acquired edition no account holds yields no
key; walk until `next` is `None`.

##### Errors

Returns [`LibraryError::Orm`](#libraryerror) when a read fails.

#### LibraryStore::record_catalog_listings

```rust
pub async fn record_catalog_listings(
    &self,
    listings: &[CatalogListing],
) -> Result<RecordedListings>
```

Stores what a store's catalog answer reported and returns each
title's listing id.

A title whose listing a store reported within
[`CATALOG_LISTING_REFRESH`](#catalog_listing_refresh) keeps its resolution and text, and only
its offers are written. Any other title is resolved into the book
model as a catalog listing ([`Self::record_catalog_listing`]: work,
expression, edition and listing row, no holding), then its store
text is filled. Text and offers go to the database plugin as atomic
host batches sized from its limits.

##### Arguments

* `listings` - The titles, in the store's order.

##### Returns

The listing ids, one per input in order. A title that does not form
a valid listing (a blank title or key, or a field over the listing
limits) gets `None`; so does one whose rows other writers kept
changing on every attempt, which is also counted in
[`RecordedListings::unresolved`](#recordedlistings) and does not stop the rest of the
answer. A stored title that was not placed stays stale, so the next
answer resolves it again.

##### Errors

Returns [`LibraryError`](#libraryerror) when a read or write of the answer's rows
fails, including the resolver's own reads and writes; titles before
the failure may already be stored.

#### LibraryStore::record_listing_offers

```rust
pub async fn record_listing_offers(&self, offers: &[(String, ListingOffer)]) -> Result<()>
```

Stores prices a store quoted for listings already in the pool.

##### Arguments

* `offers` - `(listing id, offer)` pairs. An offer older than the
  stored one for the same audience is ignored; one for a listing no
  longer stored fails its foreign key and is dropped with its batch,
  while the other batches are still written.

##### Errors

Returns the first batch's database error, after every batch was
tried.

#### LibraryStore::record_store_text

```rust
pub async fn record_store_text(
    &self,
    texts: &[(ListingKey, ListingDetail)],
) -> Result<usize>
```

Fills the store text of stored listings named by their store key, as
a store's answer for titles the library already lists would.

Each listing keeps a stored value its text leaves out, and a cover
in the text also becomes the cover of the edition the listing sells
when that edition has none.

##### Arguments

* `texts` - `(store key, store text)` pairs; a key with no stored
  listing is skipped.

A held work whose edition gained a cover has its page index row
refreshed, so the row can name that edition as its cover edition.

##### Returns

How many of `texts` named a stored listing.

##### Errors

Returns the database error of the listing read, of the first write
batch that failed (after every batch was tried), or of the row
refresh.

#### LibraryStore::catalog_listings

```rust
pub async fn catalog_listings(&self, ids: &[String]) -> Result<Vec<CatalogListingRecord>>
```

Reads stored listings by id, with their work, identifiers and offers.

Rows are read in chunks bounded by the backend's bind limit; a chunk
whose result would pass the database plugin's `maxResultBytes`
(long synopses) is split and read again.

##### Arguments

* `ids` - `store_listings` ids; duplicates are read once.

##### Returns

The listings found, in the order of `ids` (first occurrence); ids no
longer stored are left out.

##### Errors

Returns the database error.

#### LibraryStore::listing_expressions

```rust
pub async fn listing_expressions(
    &self,
    products: &[(String, String)],
) -> Result<HashMap<(String, String), BTreeSet<String>>>
```

The expressions (recordings) the shared listing pool placed each
store product in, in any marketplace.

A store's product id names one edition per marketplace, so a product
usually has one expression; a product listed in two marketplaces
under different editions has each of their expressions. A product
the pool never listed is left out.

##### Arguments

* `products` - `(plugin_key, product_id)` pairs, as stores report
  them (the product id is compared in its stored spelling).

##### Returns

Each pooled product's expression ids, keyed by the pair as given.

##### Errors

Returns the database error.

#### LibraryStore::prune_catalog_listings

```rust
pub async fn prune_catalog_listings(&self, cutoff: DateTime<Utc>) -> Result<u64>
```

Deletes catalog listings nothing refers to any more.

A listing is deleted when no store reported it since `cutoff` and
nobody holds it, no Discover subject has it as a candidate, no
cached storefront call lists it, no wish names it and no resolution
override names it. Its offers go with it. The edition, expression and work it
made stay, since other rows (a wish, a later scan) may use them.

##### Returns

How many listings were deleted.

##### Errors

Returns the database error.

#### LibraryStore::record_chapter_claim

```rust
pub async fn record_chapter_claim(
    &self,
    edition_id: &str,
    write: &ChapterClaimWrite,
) -> Result<ChaptersOutcome>
```

Records one source's chapters for an edition and resolves the
edition's chapters again.

The list is stored first when it is new. The claim then replaces
the source's earlier one; a claim that states the same list and
runtime as before changes nothing, so recording the same chapters
on every acquire costs one read.

##### Arguments

* `edition_id` - The edition (`editions.id`).
* `write` - The claim.

##### Returns

Whether the edition's chapters changed.

##### Errors

Returns [`LibraryError::NotFound`](#libraryerror) for an unknown edition, or the
database error.

#### LibraryStore::remove_chapter_edit

```rust
pub async fn remove_chapter_edit(&self, edition_id: &str) -> Result<ChaptersOutcome>
```

Removes an Administrator's edit of an edition's chapters, so they
resolve from the other sources again.

##### Returns

Whether the edition's chapters changed; [`ChaptersOutcome::Unchanged`](#chaptersoutcome)
when there was no edit.

##### Errors

Returns [`LibraryError::NotFound`](#libraryerror) for an unknown edition, or the
database error.

#### LibraryStore::pin_chapter_claim

```rust
pub async fn pin_chapter_claim(
    &self,
    edition_id: &str,
    target: Option<(ChapterSourceKind, &str)>,
    user_id: Option<i64>,
) -> Result<ChaptersOutcome>
```

Pins one of an edition's chapter claims, so it wins over every
other source, or removes the pin.

At most one claim is pinned: pinning one unpins the rest in the same
batch. A claim that is pinned or unpinned moves to the edition's
next revision.

##### Arguments

* `edition_id` - The edition.
* `target` - The claim to pin, by source kind and source id;
  `None` removes the pin.
* `user_id` - The Administrator; `None` for the operator.

##### Errors

Returns [`LibraryError::NotFound`](#libraryerror) for an unknown edition or a
claim it does not have, or the database error.

#### LibraryStore::chapter_claims

```rust
pub async fn chapter_claims(&self, edition_id: &str) -> Result<Vec<ChapterClaim>>
```

An edition's chapter claims, newest first, each with its list's
chapter count.

##### Errors

Returns the read error.

#### LibraryStore::edition_chapters

```rust
pub async fn edition_chapters(&self, edition_id: &str) -> Result<Option<EditionChapters>>
```

The chapters an edition resolved to; `None` when it has none.

##### Errors

Returns the read error, or [`LibraryError::Other`](#libraryerror) when the stored
list is missing or no longer passes its own checks.

#### LibraryStore::edition_asins

```rust
pub async fn edition_asins(&self, edition_id: &str) -> Result<Vec<(String, String)>>
```

The ASINs edition `edition_id` carries, as `(asin, marketplace)`:
strong ones first, then in value order. A provider's chapter layout
is asked for by the first.

##### Errors

Returns the read error.

#### LibraryStore::chapters_for_copy

```rust
pub async fn chapters_for_copy(
    &self,
    edition_id: &str,
    destination: &str,
    keys: &[String],
) -> Result<Option<EditionChapters>>
```

The chapters one stored copy plays: the edition's resolved list,
unless the copy's own running time is more than
[`COPY_RUNTIME_TOLERANCE_MS`](#copy_runtime_tolerance_ms) from the timeline that list was
measured on and the copy's files carried chapters of their own
([`ChapterSource::File`](#chaptersource)), which it then plays instead
(`docs/adr/edition-chapters.md`, decision 1).

The copy's running time is the sum of its probed files' durations;
when a file was never probed, or the list's runtime is unknown, the
resolved list stands.

##### Arguments

* `edition_id` - The edition (`editions.id`).
* `destination` - The destination the copy is read from.
* `keys` - The storage keys of the copy's audio files.

##### Errors

Returns the read error, or [`LibraryError::Other`](#libraryerror) when a stored
list is missing or no longer passes its own checks.

#### LibraryStore::chapter_list

```rust
pub async fn chapter_list(&self, digest: &ChapterDigest) -> Result<Option<ChapterList>>
```

A stored chapter list; `None` when no complete list has this
digest.

##### Errors

Returns the read error, or [`LibraryError::Other`](#libraryerror) when the entries
do not match the list's row or its digest.

#### LibraryStore::resolve_edition_chapters

```rust
pub async fn resolve_edition_chapters(&self, edition_id: &str) -> Result<ChaptersOutcome>
```

Resolves an edition's chapters from its claims and writes the
result, publishing `edition_updated` when the list changed.

The write is guarded on the highest claim revision the resolution
read; when a newer claim landed in between, the resolution is
computed again.

##### Errors

Returns the database error, or [`LibraryError::Conflict`](#libraryerror) when
claims kept changing for every attempt.

#### LibraryStore::destination_feed

```rust
pub async fn destination_feed(&self, destination: &str) -> Result<Option<DestinationFeed>>
```

How `destination`'s outside changes are followed; `None` before its
first sync pass (or for a stored state this build does not know).

##### Errors

Returns an error when the read fails.

#### LibraryStore::destination_feeds

```rust
pub async fn destination_feeds(&self) -> Result<Vec<DestinationFeed>>
```

Every destination's feed row, ordered by destination.

##### Errors

Returns an error when the read fails.

#### LibraryStore::save_destination_feed

```rust
pub async fn save_destination_feed(&self, feed: &mut DestinationFeed) -> Result<()>
```

Writes `feed`, replacing the destination's row, and moves
`feed.revision` to the revision written.

The write is a compare-and-swap on the revision `feed` was read at:
a first save inserts the row, and a later one updates it only while
nobody else saved since. A sync pass whose lease another node took
therefore fails at its next save instead of overwriting the new
holder's progress.

##### Errors

Returns [`LibraryError::Conflict`](#libraryerror) when the row changed since
`feed` was read, or the error of a failed write.

#### LibraryStore::record_seen_objects

```rust
pub async fn record_seen_objects(
    &self,
    destination: &str,
    seq: i64,
    objects: &[SeenObject],
    now: DateTime<Utc>,
) -> Result<u64>
```

Records that `destination` holds `objects`, as a listing page or a
feed page numbered `seq` reported them.

Each object is marked seen at `seq`. One that is new, was removed,
or whose size or signature differs from its row is recorded as
changed at `seq`; an unchanged one keeps its `changed_seq`. A row a
later page or listing already wrote is left as it is.

##### Returns

How many objects changed.

##### Errors

Returns an error when a read or a write fails.

#### LibraryStore::remove_seen_objects

```rust
pub async fn remove_seen_objects(
    &self,
    destination: &str,
    seq: i64,
    keys: &[String],
    now: DateTime<Utc>,
) -> Result<u64>
```

Records that the objects at `keys` are gone from `destination`, as
a feed page numbered `seq` reported. A key with no row, already
removed, or written by a later page or listing is left alone.

##### Returns

How many objects were marked removed.

##### Errors

Returns an error when a write fails.

#### LibraryStore::unlisted_object_count

```rust
pub async fn unlisted_object_count(
    &self,
    destination: &str,
    listing_seq: i64,
) -> Result<u64>
```

Objects `destination` is known to hold that the listing numbered
`listing_seq` has not seen: the ones [`Self::finish_listing`](#librarystorefinish_listing) would
mark removed.

##### Errors

Returns an error when the read fails.

#### LibraryStore::known_object_count

```rust
pub async fn known_object_count(&self, destination: &str) -> Result<u64>
```

Objects `destination` is known to hold now.

##### Errors

Returns an error when the read fails.

#### LibraryStore::finish_listing

```rust
pub async fn finish_listing(
    &self,
    destination: &str,
    listing_seq: i64,
    now: DateTime<Utc>,
) -> Result<u64>
```

Ends the listing numbered `listing_seq` of `destination`: every
object it did not see is marked removed at `listing_seq`.

##### Returns

How many objects were marked removed.

##### Errors

Returns an error when the write fails.

#### LibraryStore::keep_unlisted_objects

```rust
pub async fn keep_unlisted_objects(
    &self,
    destination: &str,
    listing_seq: i64,
) -> Result<()>
```

Marks every object `destination` holds as seen by the listing
numbered `listing_seq`, without recording a change: used when a
listing's removals are held back, so the next listing compares
against the same set.

##### Errors

Returns an error when the write fails.

#### LibraryStore::changed_objects

```rust
pub async fn changed_objects(
    &self,
    destination: &str,
    after: (i64, Option<&str>),
    limit: u64,
) -> Result<Vec<ChangedObject>>
```

Up to `limit` objects of `destination` whose rows changed after
`after`: a change sequence, or a sequence and the last key read in
it to continue a page. Ordered by sequence, then key.

##### Errors

Returns an error when the read fails.

#### LibraryStore::prune_object_tombstones

```rust
pub async fn prune_object_tombstones(&self, before: DateTime<Utc>) -> Result<u64>
```

Deletes tombstones of objects removed before `before`.

##### Returns

How many tombstones were deleted.

##### Errors

Returns an error when the delete fails.

#### LibraryStore::lose_copies

```rust
pub async fn lose_copies(
    &self,
    destination: &str,
    keys: &[String],
    action: MissingFileAction,
    now: DateTime<Utc>,
) -> Result<LostCopies>
```

Records that the files at `keys` are gone from `destination`.

Every stored copy whose primary audio key is one of them is looked
at. When another audio file of its edition is recorded at the
destination (`edition_files`) and the destination still holds it
(the file was moved or renamed), the copy now names that key.
Otherwise `action` decides:

- [`MissingFileAction::Restore`]: the copy turns pending, due at
  `now`, so the next copy retry writes it again.
- [`MissingFileAction::MarkMissing`]: the copy turns missing and is
  left alone. When no destination holds a stored copy of the edition
  any more, the edition is no longer acquired (its title's acquire
  status turns not acquired), so a member can ask for it again.

##### Errors

Returns an error when a read or a write fails.

#### LibraryStore::release_missing_editions

```rust
pub async fn release_missing_editions(&self, destination: &str) -> Result<u64>
```

Releases every edition with a missing copy at `destination` that is
still acquired although no destination holds a stored copy of it:
the state a failed release in [`Self::lose_copies`](#librarystorelose_copies) leaves behind,
since the copy is already missing and no later loss names it.

##### Returns

How many editions turned not acquired.

##### Errors

Returns an error when a read or a write fails.

#### LibraryStore::unlisted_copy_keys

```rust
pub async fn unlisted_copy_keys(&self, destination: &str) -> Result<(Vec<String>, u64)>
```

The primary audio keys of `destination`'s stored copies that have no
present object row there: files a finished listing did not see
(including ones that were gone before the destination was first
listed, which leave no tombstone), in key order.

##### Returns

The missing keys, and how many distinct keys the destination's stored
copies name.

##### Errors

Returns an error when a read fails.

#### LibraryStore::replace_discover_candidates

```rust
pub async fn replace_discover_candidates(
    &self,
    subject: &DiscoverSubject,
    source: &str,
    scope: SeedScope,
    rows: &[CandidateRef],
) -> Result<()>
```

Replaces every candidate one store returned for one subject.

Each row is upserted with this call's fetch time, then the store's
rows for the subject that this call did not write are deleted. Other
stores' rows for the subject are untouched.

The writes go to the database plugin as canonical host batches, each
one atomic request (one transaction) holding as many rows as the
plugin's `maxStatements` and `maxRequestBytes` allow, so a few hundred
candidates cost a handful of round trips instead of one per row. No
interactive transaction is used: the external database adapters only
take single statements and atomic batches, so `begin` is refused
there. The delete rides in the last batch. A reader between two
batches can see part of the new set next to old rows that are not
deleted yet.

##### Arguments

* `subject` - Whose candidates these are.
* `source` - Storefront plugin id that returned them.
* `scope` - Seeds that produced them.
* `rows` - The candidates, as listing references; a listing named
  twice keeps the last.

##### Errors

Returns the database error, including when a single row is larger
than one request may carry. Batches written before it stay; the old
rows are only deleted once every new row is stored.

#### LibraryStore::list_discover_candidates

```rust
pub async fn list_discover_candidates(
    &self,
    subject: &DiscoverSubject,
) -> Result<Vec<StoredCandidate>>
```

Lists every stored candidate of one subject, oldest store fetch first,
each joined with its listing from the shared pool.

Candidate rows are read in id pages, then their listings in chunks
that stay under the database plugin's result limit
([`LibraryStore::catalog_listings`](#librarystorecatalog_listings)), then the rows are ordered by
fetch time (ties keep id order). Rows whose scope text is unknown
(written by a newer release), or whose listing is gone, are skipped.

##### Errors

Returns the database error.

#### LibraryStore::discover_subject_exists

```rust
pub async fn discover_subject_exists(&self, subject: &DiscoverSubject) -> Result<bool>
```

Reports whether a Discover subject still exists.

The operator always exists; a user or portal subject exists while
its `users` or `portal_identities` row does. Jobs check this so a
fetch or ranking queued before the subject was deleted does not
write its rows back.

##### Errors

Returns the database error.

#### LibraryStore::forget_discover_subject

```rust
pub async fn forget_discover_subject(&self, subject: &DiscoverSubject) -> Result<()>
```

Deletes every stored Discover row of one subject (candidates, feed,
taste and fetch marker).

##### Errors

Returns the database error.

#### LibraryStore::put_discover_taste

```rust
pub async fn put_discover_taste(
    &self,
    subject: &DiscoverSubject,
    taste_json: &str,
    scope: SeedScope,
) -> Result<()>
```

Stores one subject's taste, replacing the previous one.

##### Arguments

* `subject` - Whose taste it is.
* `taste_json` - The taste, as the Discover crate encodes it.
* `scope` - Seeds it comes from.

##### Errors

Returns the database error.

#### LibraryStore::get_discover_taste

```rust
pub async fn get_discover_taste(
    &self,
    subject: &DiscoverSubject,
) -> Result<Option<StoredDiscoverTaste>>
```

Returns one subject's stored taste, when it has one.

##### Errors

Returns the database error.

#### LibraryStore::mark_discover_fetch

```rust
pub async fn mark_discover_fetch(
    &self,
    subject: &DiscoverSubject,
    at: DateTime<Utc>,
) -> Result<()>
```

Records that one subject's candidate fetch finished at `at`.

Recorded separately from the candidate rows, so a fetch that found
nothing new still counts as fresh for a day.

##### Errors

Returns the database error.

#### LibraryStore::newest_discover_candidate_fetch

```rust
pub async fn newest_discover_candidate_fetch(
    &self,
    subject: &DiscoverSubject,
) -> Result<Option<DateTime<Utc>>>
```

Returns when one subject's last candidate fetch finished.

##### Returns

`None` when no fetch has finished for the subject yet.

##### Errors

Returns the database error.

#### LibraryStore::get_discover_store_call

```rust
pub async fn get_discover_store_call(
    &self,
    call_key: &str,
    max_age: Duration,
) -> Result<Option<Vec<StoreCallHit>>>
```

Returns a cached storefront answer no older than `max_age`.

##### Arguments

* `call_key` - Key the caller derived from the store, call kind and query.
* `max_age` - Oldest answer still worth reusing.

##### Returns

The listings the call answered, in the store's order, or `None`
when absent or too old.

##### Errors

Returns the database error.

#### LibraryStore::get_discover_store_calls

```rust
pub async fn get_discover_store_calls(
    &self,
    call_keys: &[String],
    max_age: Duration,
) -> Result<HashMap<String, Vec<StoreCallHit>>>
```

[`Self::get_discover_store_call`](#librarystoreget_discover_store_call) for many calls at once, in a few
chunked reads instead of two per call.

A discover fetch asks one store dozens of calls (deals, each seed,
each starter genre); reading their cached answers up front saves two
database round trips per call. Like the single read, a call row is
read first and its hits after, so an answer being rewritten reads as
expired, never as part of its hits.

##### Arguments

* `call_keys` - Keys the caller derived; duplicates are read once.
* `max_age` - Oldest answer still worth reusing.

##### Returns

Each key with a fresh answer, mapped to its listings in the store's
order (at most 500 per key). Absent or expired keys are
left out.

##### Errors

Returns the database error.

#### LibraryStore::put_discover_store_call

```rust
pub async fn put_discover_store_call(
    &self,
    call_key: &str,
    source: &str,
    hits: &[StoreCallHit],
) -> Result<()>
```

Stores (or refreshes) one cached storefront answer: the call row and
the listings it answered, replacing the previous answer's hits.

The writes go as atomic host batches sized from the database
plugin's limits. The first batch marks the call as fetched long ago
and deletes the old hits; the new fetch time is written last, in the
final batch. A reader between batches, or after a batch fails, sees
an expired answer and asks the store again, never a fresh answer
with only part of its hits. Each write claims the call row with its
own random token, and its hits and fetch time apply only while the
row still carries that token, so two concurrent answers to one call
are never mixed: the later claim wins, and the earlier write adds
nothing after it.

##### Arguments

* `call_key` - The call's key.
* `source` - Storefront plugin id that answered.
* `hits` - The listings, in the store's order.

##### Errors

Returns the database error.

#### LibraryStore::prune_discover_store_calls

```rust
pub async fn prune_discover_store_calls(&self, cutoff: DateTime<Utc>) -> Result<u64>
```

Deletes cached storefront answers fetched before `cutoff`.

##### Returns

How many answers were deleted.

##### Errors

Returns the database error.

#### LibraryStore::put_discover_feed

```rust
pub async fn put_discover_feed(
    &self,
    subject: &DiscoverSubject,
    feed_json: &str,
    scope: SeedScope,
    candidates_fetched_at: Option<DateTime<Utc>>,
) -> Result<()>
```

Stores (or replaces) one subject's ranked feed.

One upsert writes the row without reading the old one first, so a
stored feed too large to read back cannot block its replacement.

##### Arguments

* `subject` - Whose feed it is.
* `feed_json` - The feed, before hidden shelves are filtered out.
* `scope` - Seeds it was ranked from.
* `candidates_fetched_at` - Newest candidate fetch it used, when any.

##### Errors

Returns the database error.

#### LibraryStore::get_discover_feed

```rust
pub async fn get_discover_feed(
    &self,
    subject: &DiscoverSubject,
) -> Result<Option<StoredFeed>>
```

Returns one subject's stored feed, when it has one.

##### Errors

Returns the database error.

#### LibraryStore::list_discover_feed_subjects

```rust
pub async fn list_discover_feed_subjects(&self) -> Result<Vec<DiscoverSubject>>
```

Lists every subject that has a stored feed, so a shared change (the
request queue) can re-rank them all.

Keys that do not parse as a [`DiscoverSubject`](#discoversubject) are skipped.

##### Errors

Returns the database error.

#### LibraryStore::set_copy_stored

```rust
pub async fn set_copy_stored(
    &self,
    edition_id: &str,
    destination: &str,
    storage_key: &str,
) -> Result<()>
```

Records that `destination` holds the complete files of `edition_id`,
with its primary audio at `storage_key` there. Clears any pending
retry.

##### Errors

Returns an error when the write fails.

#### LibraryStore::set_copy_pending

```rust
pub async fn set_copy_pending(
    &self,
    edition_id: &str,
    destination: &str,
    error: &str,
    now: DateTime<Utc>,
) -> Result<CopyState>
```

Records that writing `edition_id` to `destination` failed with
`error` at `now`. The copy turns pending with its next attempt due
after [`copy_retry_delay`](#copy_retry_delay), or failed once it has failed
[`MAX_COPY_ATTEMPTS`](#max_copy_attempts) times. A key an earlier stored copy recorded is
kept.

The attempt count is read, then written: the acquire job that writes
an edition holds the node's work lock, so two writes of one copy do
not race.

##### Returns

The copy's state after the failure.

##### Errors

Returns an error when the read or the write fails.

#### LibraryStore::record_copies_from_files

```rust
pub async fn record_copies_from_files(&self, edition_id: &str) -> Result<usize>
```

Records a stored copy of `edition_id` on each destination where its
primary audio file is recorded (`edition_files`, role `audio`) and
no copy row exists yet.

An edition acquired before copies were recorded has files but no
copy rows; its audio rows say where the acquire wrote it. Call this
only for an acquired edition: an acquire records each key just
before writing it, so a row alone does not prove the write
finished. Existing copy rows are never changed.

##### Returns

How many copies were recorded.

##### Errors

Returns an error when the read or a write fails.

#### LibraryStore::copies_of_editions

```rust
pub async fn copies_of_editions(&self, edition_ids: &[String]) -> Result<Vec<EditionCopy>>
```

Every recorded copy of the editions in `edition_ids`, ordered by
edition then destination. Rows in a state this build does not know
are left out.

##### Errors

Returns an error when the read fails.

#### LibraryStore::due_copy_editions

```rust
pub async fn due_copy_editions(
    &self,
    now: DateTime<Utc>,
    destinations: &[String],
    limit: u64,
) -> Result<Vec<String>>
```

Up to `limit` editions with a pending copy on one of `destinations`
whose next attempt is due at `now`, soonest due first, each named
once. A copy on a destination not in `destinations` (one no longer
deployed) is not due anywhere.

##### Errors

Returns an error when the read fails.

#### LibraryStore::defer_copy_retries

```rust
pub async fn defer_copy_retries(
    &self,
    edition_id: &str,
    destinations: &[String],
    now: DateTime<Utc>,
) -> Result<()>
```

Moves the due retries of `edition_id`'s pending copies on
`destinations` to their next delay from `now`, without counting an
attempt: called once an acquire for them is queued, so the next
pass does not queue another while that one waits. The acquire's own
outcome then records the copy as usual.

##### Errors

Returns an error when the read or a write fails.

#### LibraryStore::enrich_candidates_page

```rust
pub async fn enrich_candidates_page(
    &self,
    after: Option<&str>,
    limit: u64,
) -> Result<EnrichCandidatePage>
```

One page of the held editions an enricher walks, in edition id order.

##### Arguments

* `after` - The previous page's [`EnrichCandidatePage::next`](#enrichcandidatepage); `None`
  for the first page.
* `limit` - Most editions the page reads (1 to 200, and at most the
  database's `maxResultRows`).

##### Returns

The page's editions that a holding still holds, and the cursor of
the next page. An edition nobody holds is left out, so a page can be
short or empty while more follow: walk until `next` is `None`.

##### Errors

Returns [`LibraryError::Orm`](#libraryerror) when a read fails.

#### LibraryStore::enrich_edition

```rust
pub async fn enrich_edition(
    &self,
    edition_id: &str,
    fill: &EditionEnrichment,
) -> Result<()>
```

Fills what edition `edition_id` lacks from `fill` and records who
filled it.

The work takes the description, subjects and cover it lacks (a work
another edition already described keeps its own values), the
expression the language it lacks, and the edition the cover it
lacks. The edition's provenance is written last, so a write
that fails before it leaves the edition unmarked and the next run
tries again.

##### Arguments

* `edition_id` - The edition filled.
* `fill` - The values offered and who offers them.

##### Errors

Returns [`LibraryError::NotFound`](#libraryerror) when the edition is gone, or
[`LibraryError::Orm`](#libraryerror) when a read or write fails.

#### LibraryStore::edition_acquisition

```rust
pub async fn edition_acquisition(
    &self,
    edition_id: &str,
) -> Result<Option<EditionAcquisition>>
```

An edition's acquire state; `None` when no title of it was ever
queued or acquired.

##### Errors

Returns an error when the read fails.

#### LibraryStore::edition_of_holding

```rust
pub async fn edition_of_holding(&self, holding_id: &str) -> Result<Option<String>>
```

The edition holding `holding_id` (`holdings.id`) holds: the one its
listing sells; `None` when the holding or its listing is gone.

##### Errors

Returns an error when a read fails.

#### LibraryStore::file_owner_of_holding

```rust
pub async fn file_owner_of_holding(&self, holding_id: &str) -> Result<Option<FileOwner>>
```

The file owner of a node holding (`holdings.id`, a title a branch
shares here): the edition its listing sells.

##### Errors

Returns an error when a read fails.

#### LibraryStore::record_edition_file

```rust
pub async fn record_edition_file(
    &self,
    destination: &str,
    key: &str,
    owner: &FileOwner,
    role: FileRole,
) -> Result<bool>
```

Records that `owner` writes `key` on `destination` (an acquire run
on this node, just before the write).

The record is taken when the key is unrecorded, already `owner`'s,
or held by an owner no longer live (a removed edition, or a title a
branch shares here, whose keys an acquire on this node takes over).
Another live edition's key is refused, changing nothing: two
editions whose short ids collide can plan one chapter or sidecar
name the audio-name check does not see, and the write must not
replace that edition's file. Each record is one conditional write,
so of two acquires racing for one key only one wins.

##### Returns

`true` when the key is now `owner`'s; `false` when another live
edition holds it.

##### Errors

Returns an error when a write fails.

#### LibraryStore::record_edition_copy_file

```rust
pub async fn record_edition_copy_file(
    &self,
    destination: &str,
    key: &str,
    owner: &FileOwner,
    role: FileRole,
    copy: Option<&OutputCopy>,
) -> Result<bool>
```

[`Self::record_edition_file`](#librarystorerecord_edition_file) for a file of one library copy
(`output.copies`): the row also names the copy's format and quality,
or none when `copy` is `None`.

##### Returns

`true` when the key is now `owner`'s; `false` when another live
edition holds it.

##### Errors

Returns an error when a write fails.

#### LibraryStore::claim_edition_file

```rust
pub async fn claim_edition_file(
    &self,
    destination: &str,
    key: &str,
    owner: &FileOwner,
    writer: &str,
    role: FileRole,
    sha256: Option<&str>,
    expected: Option<&EditionFile>,
    title: &str,
    catalog: bool,
) -> Result<bool>
```

Claims `key` on `destination` for `owner`, with `writer` (a host
upload id) as its latest writer.

The claim succeeds when the key is unrecorded, already the owner's
(any account's row of the same edition), or recorded for an owner
that holds no key against this claim; answers false, changing
nothing, when another edition holds it. An edition holds its keys
against every claim while it exists. Each claim
is one conditional write, so of two uploads of different editions
racing for one key only one wins.

`expected` is the record the caller read before
([`Self::edition_file`](#librarystoreedition_file); `None` when unrecorded), which a refused
claim's release restores: the claim also answers false, changing
nothing, when the record is no longer that one (an acquire recorded
the key in between), so a release never removes a record the claim
did not replace.

##### Arguments

* `destination`, `key` - The file's place.
* `owner` - Who claims it.
* `writer` - The host upload id.
* `role`, `sha256` - What the file is and its declared digest.
* `expected` - The record read before the claim.
* `title` - What the upload writes for: the
  account holding (`holdings.id`) of a job upload, or the node holding
  (`holdings.id`) of a catalog upload. A key its edition already
  holds is taken over only when this upload or an earlier upload for
  the same title (or, for a catalog upload, an earlier catalog upload
  of the same branch) wrote it, never one an acquire or another
  account's upload wrote: the edition's files are shared, so one node cannot
  replace them. A job upload may also take over a key a branch's
  catalog upload wrote (`catalog` false), as an acquire on this node
  would replace a delivered copy with its own.
* `catalog` - True for a catalog upload's claim.

##### Errors

Returns an error when a write fails.

#### LibraryStore::release_edition_file

```rust
pub async fn release_edition_file(
    &self,
    destination: &str,
    key: &str,
    writer: &str,
    prior: Option<&EditionFile>,
) -> Result<()>
```

Gives back a claim `writer` made on `key` on `destination`: the
record returns to `prior` (who held it before the claim), or is
removed when `prior` is `None` (the claim created it). Changes
nothing once someone else has written the key since (its writer is
no longer `writer`), so a release never takes a key from a later
claim or acquire.

##### Errors

Returns an error when the write fails.

#### LibraryStore::count_edition_files_by

```rust
pub async fn count_edition_files_by(&self, writer: &str) -> Result<u64>
```

How many keys `writer` (a host upload id) holds, on every
destination together. An upload id is unique and claims keys for one
owner only, so this is how many keys that upload's title still holds
through it.

##### Errors

Returns an error when the read fails.

#### LibraryStore::edition_file

```rust
pub async fn edition_file(
    &self,
    destination: &str,
    key: &str,
) -> Result<Option<EditionFile>>
```

Who holds `key` on `destination`; `None` when it is unrecorded.

A row naming an owner kind this build does not know reads as held
by an edition no claim matches, so it is never taken over.

##### Errors

Returns an error when the read fails.

#### LibraryStore::set_edition_file_probe

```rust
pub async fn set_edition_file_probe(
    &self,
    destination: &str,
    key: &str,
    owner: &FileOwner,
    probe: &FileProbe,
) -> Result<bool>
```

Stores what a probe found in `owner`'s file at `key` on
`destination`, stamped with the time of the write.

The row is changed only while `owner` still holds the key, so a
probe that finishes after another edition took the key over
describes nothing.

##### Returns

`true` when the row was `owner`'s and now carries `probe`.

##### Errors

Returns an error when `probe` is not well formed
([`FileProbe::is_well_formed`](#fileprobeis_well_formed)) or the write fails.

#### LibraryStore::set_uploaded_file_probe

```rust
pub async fn set_uploaded_file_probe(
    &self,
    destination: &str,
    key: &str,
    writer: &str,
    probe: &FileProbe,
) -> Result<bool>
```

Stores what a probe found in the file the upload `writer` wrote at
`key` on `destination`, as [`Self::set_edition_file_probe`](#librarystoreset_edition_file_probe) does for
an owner.

The row is changed only while `writer` is still the upload that
claimed the key last, so a committed upload never describes a file
a later write replaced.

##### Returns

`true` when the row was `writer`'s and now carries `probe`.

##### Errors

Returns an error when `probe` is not well formed or the write fails.

#### LibraryStore::set_edition_file_stream

```rust
pub async fn set_edition_file_stream(
    &self,
    destination: &str,
    key: &str,
    owner: &FileOwner,
    stream: &AudioStream,
) -> Result<bool>
```

Stores what a probe found about the audio stream of `owner`'s file
at `key` on `destination` ([`AudioStream`](#audiostream)), beside the probe
[`Self::set_edition_file_probe`](#librarystoreset_edition_file_probe) stores.

The row is changed only while `owner` still holds the key, as for
the probe.

##### Returns

`true` when the row was `owner`'s and now carries `stream`.

##### Errors

Returns an error when `stream` is not well formed
([`AudioStream::is_well_formed`](#audiostreamis_well_formed)) or the write fails.

#### LibraryStore::edition_audio_files

```rust
pub async fn edition_audio_files(&self, edition_id: &str) -> Result<Vec<ProbedAudioFile>>
```

Every audio file (primary audio, parts and chapters) recorded for
edition `edition_id`, on every destination, with what the last probe
of each found ([`ProbedAudioFile`](#probedaudiofile)), in destination then key order.

##### Errors

Returns an error when the read fails.

#### LibraryStore::edition_file_probe

```rust
pub async fn edition_file_probe(
    &self,
    destination: &str,
    key: &str,
) -> Result<Option<FileProbe>>
```

What the last probe found in the file at `key` on `destination`;
`None` when the key is unrecorded or was never probed.

##### Errors

Returns an error when the read fails.

#### LibraryStore::edition_file_copy

```rust
pub async fn edition_file_copy(
    &self,
    destination: &str,
    key: &str,
) -> Result<Option<OutputCopy>>
```

The library copy (`output.copies`) the file at `key` on
`destination` belongs to; `None` when the key is unrecorded, belongs
to no one copy, or names a copy this build cannot read.

##### Errors

Returns an error when the read fails.

#### LibraryStore::audio_keys_of

```rust
pub async fn audio_keys_of(
    &self,
    destination: &str,
    owner: &FileOwner,
) -> Result<Vec<String>>
```

The audio keys (primary audio, parts and chapters) recorded for
`owner` on `destination`, in key order: every file a copy of the
edition's recording there needs.

##### Errors

Returns an error when the read fails.

#### LibraryStore::name_held_by_another_owner

```rust
pub async fn name_held_by_another_owner(
    &self,
    stem: &str,
    owner: &FileOwner,
) -> Result<bool>
```

True when another live edition's file is recorded, on any
destination, at the name `stem` (a planned key without its
extension): the key `stem` itself, `stem` with an extension in the
same folder (`stem.cue`), or one of its part files
(`stem — Part 1.m4b`). Another edition's files already live under
that name, so an acquire of `owner` is refused rather than overwrite
them. A longer name that only starts with `stem` (`stem Peaks.m4b`)
is not the same name. A branch title's file, or a file of an edition
no longer in the library, does not count: an acquire takes those
keys over, as [`LibraryStore::claim_edition_file`](#librarystoreclaim_edition_file) does.

##### Errors

Returns an error when the read fails.

#### LibraryStore::publish_domain_event

```rust
pub async fn publish_domain_event(
    &self,
    spec: PublishDomainEventSpec,
) -> Result<PublishDomainEventOutcome>
```

Persist a domain event. Duplicate `(account_id, source, event_type, dedup_key)`
coalesces. Wake processing is left to [`Self::process_pending_wakes`](#librarystoreprocess_pending_wakes).

##### Errors

Returns [`LibraryError::Orm`](#libraryerror) when the write fails, or
[`LibraryError::Other`](#libraryerror) when the payload exceeds 64 KiB.

#### LibraryStore::dispatch_event_deliveries

```rust
pub async fn dispatch_event_deliveries(
    &self,
    event_id: &str,
    subscribers: &[EventSubscriber],
    operation_id: &str,
) -> Result<u32>
```

Snapshot matching subscribers and create one delivery row per plugin.

Idempotent under replay: existing `(event_id, plugin_id)` rows are kept.
Marks the outbox row `dispatched` in the same transaction.

##### Errors

Returns [`LibraryError::Orm`](#libraryerror) when the write fails.

#### LibraryStore::next_undispatched_event

```rust
pub async fn next_undispatched_event(&self) -> Result<Option<DomainEventRecord>>
```

Next undispatched outbox row (oldest first).

##### Errors

Returns [`LibraryError::Orm`](#libraryerror) when the read fails.

#### LibraryStore::process_pending_wakes

```rust
pub async fn process_pending_wakes(
    &self,
    limit: u64,
    _owner: &str,
    lease_secs: u64,
) -> Result<PendingWakeProgress>
```

Claim up to `limit` `wake_pending` events and process one delivery page each.

Each claimed event is leased with a unique UUID token so two processors
cannot own the same slice (process `node_id` is not the fence — it is ABA
across restarts). Cursor release and finish require that token.

##### Errors

Returns [`LibraryError::Orm`](#libraryerror) when the read or wake write fails.

#### LibraryStore::lease_next_event_delivery

```rust
pub async fn lease_next_event_delivery(
    &self,
    owner: &str,
    lease_secs: u64,
    operation_id: &str,
    plugin_ids: &[String],
    max_in_flight: u32,
    node_id: &str,
) -> Result<Option<EventDeliveryRecord>>
```

Lease the next ready delivery with a fenced `pending` → `running` mutation.

A delivery is one subscriber's own copy of an event, so a lease never
takes work from another subscriber: it only picks which node runs this
copy. Jobs, by contrast, are claimed (see `docs/internals/jobs.md`).

When `node_id` is non-empty, this node’s own catalog must match the parent
event’s type, schema version, and filter. The host loads the catalog and
filters candidates in Rust, then CAS-updates one row (portable slot +
`UPDATE … WHERE id = ? AND state = pending`). An empty `node_id` keeps
plugin-id-only filtering (tests).

##### Errors

Returns [`LibraryError::Orm`](#libraryerror) when the claim write fails.

#### LibraryStore::lease_next_event_delivery_due_by

```rust
pub async fn lease_next_event_delivery_due_by(
    &self,
    owner: &str,
    lease_secs: u64,
    operation_id: &str,
    plugin_ids: &[String],
    max_in_flight: u32,
    node_id: &str,
    due_by: DateTime<Utc>,
) -> Result<Option<EventDeliveryRecord>>
```

[`Self::lease_next_event_delivery`](#librarystorelease_next_event_delivery) restricted to deliveries due at
or before `due_by` (`run_after <= due_by`).

A host that is not the best placed to run a plugin passes a cutoff in
the past, so it only picks up deliveries the better hosts left
waiting that long.

##### Errors

Returns [`LibraryError::Orm`](#libraryerror) when the claim write fails.

#### LibraryStore::release_unexecuted_event_delivery

```rust
pub async fn release_unexecuted_event_delivery(
    &self,
    fence: &EventDeliveryFence,
    restore_resume: bool,
) -> Result<bool>
```

Return a claimed delivery to `pending` without consuming an attempt.

Used when this worker cannot execute the subscriber (plugin not loaded).

##### Errors

Returns [`LibraryError::Orm`](#libraryerror) when the update fails.

#### LibraryStore::heartbeat_event_delivery

```rust
pub async fn heartbeat_event_delivery(
    &self,
    fence: &EventDeliveryFence,
    lease_secs: u64,
) -> Result<bool>
```

Refresh the lease when `fence` still owns the running generation.

##### Errors

Returns [`LibraryError::Orm`](#libraryerror) when the update fails.

#### LibraryStore::ack_event_delivery

```rust
pub async fn ack_event_delivery(&self, fence: &EventDeliveryFence) -> Result<bool>
```

Mark a fenced delivery `acked`.

##### Errors

Returns [`LibraryError::Orm`](#libraryerror) when the update fails.

#### LibraryStore::reject_event_delivery

```rust
pub async fn reject_event_delivery(
    &self,
    fence: &EventDeliveryFence,
    reason: &str,
) -> Result<bool>
```

Mark a fenced delivery permanently `rejected`.

##### Errors

Returns [`LibraryError::Orm`](#libraryerror) when the update fails.

#### LibraryStore::dead_letter_event_delivery

```rust
pub async fn dead_letter_event_delivery(
    &self,
    fence: &EventDeliveryFence,
    reason: &str,
) -> Result<bool>
```

Mark a fenced delivery `dead_letter`.

##### Errors

Returns [`LibraryError::Orm`](#libraryerror) when the update fails.

#### LibraryStore::retry_event_delivery

```rust
pub async fn retry_event_delivery(
    &self,
    fence: &EventDeliveryFence,
    run_after: DateTime<Utc>,
    reason: &str,
) -> Result<bool>
```

Return a fenced delivery to `pending` with backoff (retryable handler).

When `attempt_count` has already reached `max_attempts`, the delivery is
dead-lettered instead of looping forever on a guest that keeps returning
retry.

##### Errors

Returns [`LibraryError::Orm`](#libraryerror) when the update fails.

#### LibraryStore::suspend_event_delivery

```rust
pub async fn suspend_event_delivery(
    &self,
    fence: &EventDeliveryFence,
    checkpoint_json: &str,
    checkpoint_schema_version: i64,
    wake_at: DateTime<Utc>,
    wake_event_type: &str,
    wake_filter_json: &str,
    wake_grants_json: &str,
) -> Result<bool>
```

Suspend a fenced delivery (checkpoint + wake) without burning an attempt.

##### Errors

Returns [`LibraryError::Orm`](#libraryerror) when the update fails.

#### LibraryStore::fail_event_delivery

```rust
pub async fn fail_event_delivery(
    &self,
    fence: &EventDeliveryFence,
    reason: &str,
) -> Result<bool>
```

Fail a running delivery: retry with backoff or dead-letter at max attempts.

##### Errors

Returns [`LibraryError::Orm`](#libraryerror) when the update fails.

#### LibraryStore::reclaim_expired_event_deliveries

```rust
pub async fn reclaim_expired_event_deliveries(&self) -> Result<u32>
```

Reclaim expired running deliveries back to `pending`.

Rows with `cancel_requested` become `rejected` instead of pending so an
operator cancel survives a worker crash. A stored checkpoint restores
`resume_pending` so a crash during resume does not consume an attempt.

##### Errors

Returns [`LibraryError::Orm`](#libraryerror) when the update fails.

#### LibraryStore::get_domain_event

```rust
pub async fn get_domain_event(&self, id: &str) -> Result<Option<DomainEventRecord>>
```

Load one event envelope.

##### Errors

Returns [`LibraryError::Orm`](#libraryerror) when the read fails.

#### LibraryStore::get_event_delivery

```rust
pub async fn get_event_delivery(&self, id: &str) -> Result<Option<EventDeliveryRecord>>
```

Load one delivery.

##### Errors

Returns [`LibraryError::Orm`](#libraryerror) when the read fails.

#### LibraryStore::list_domain_events

```rust
pub async fn list_domain_events(&self, limit: u64) -> Result<Vec<DomainEventRecord>>
```

Recent outbox events, newest first.

##### Errors

Returns [`LibraryError::Orm`](#libraryerror) when the read fails.

#### LibraryStore::list_domain_events_for_queue

```rust
pub async fn list_domain_events_for_queue(
    &self,
    queue_name: &str,
    limit: u64,
) -> Result<Vec<DomainEventRecord>>
```

Recent outbox rows on one queue, newest first.

##### Errors

Returns [`LibraryError::Orm`](#libraryerror) when the read fails.

#### LibraryStore::list_queue_names

```rust
pub async fn list_queue_names(&self) -> Result<Vec<String>>
```

Distinct queue names that hold at least one outbox row, sorted.

##### Errors

Returns [`LibraryError::Orm`](#libraryerror) when the read fails.

#### LibraryStore::count_queue_deliveries

```rust
pub async fn count_queue_deliveries(&self, queue_name: &str, state: &str) -> Result<i64>
```

Delivery rows on one queue in `state`.

##### Errors

Returns [`LibraryError::Orm`](#libraryerror) when the read fails.

#### LibraryStore::list_event_deliveries_for_queue

```rust
pub async fn list_event_deliveries_for_queue(
    &self,
    queue_name: &str,
    state: Option<&str>,
    limit: u64,
) -> Result<Vec<EventDeliveryRecord>>
```

Deliveries filtered by optional state and queue, newest first.

##### Errors

Returns [`LibraryError::Orm`](#libraryerror) when the read fails.

#### LibraryStore::list_event_deliveries

```rust
pub async fn list_event_deliveries(
    &self,
    state: Option<&str>,
    limit: u64,
) -> Result<Vec<EventDeliveryRecord>>
```

Deliveries filtered by optional state, newest first.

##### Errors

Returns [`LibraryError::Orm`](#libraryerror) when the read fails.

#### LibraryStore::retry_dead_letter_delivery

```rust
pub async fn retry_dead_letter_delivery(&self, id: &str) -> Result<bool>
```

Re-queue a dead-lettered delivery for operator retry.

##### Errors

Returns [`LibraryError::Orm`](#libraryerror) when the update fails.

#### LibraryStore::acknowledge_dead_letter_delivery

```rust
pub async fn acknowledge_dead_letter_delivery(&self, id: &str) -> Result<bool>
```

Operator acknowledge: mark a dead-letter as rejected (no further retry).

##### Errors

Returns [`LibraryError::Orm`](#libraryerror) when the update fails.

#### LibraryStore::count_event_deliveries

```rust
pub async fn count_event_deliveries(&self, state: &str) -> Result<i64>
```

Count deliveries in `state`.

A peer writer on the same SQLite file can make this read return
`SQLITE_BUSY`. The read is repeated until the lock clears or the
attempt budget is spent. `state` is unchanged.

##### Errors

Returns [`LibraryError::Orm`](#libraryerror) when the read fails for a reason other
than file-lock contention, and [`LibraryError::Unavailable`](#libraryerror) when the
lock is still held after the retries.

#### LibraryStore::event_delivery_metrics

```rust
pub async fn event_delivery_metrics(&self) -> Result<EventDeliveryMetrics>
```

Operator-visible delivery-queue counters (pending vs suspended split).

Each read waits out a peer SQLite file lock the same way
[`Self::count_event_deliveries`](#librarystorecount_event_deliveries) does.

##### Errors

Returns [`LibraryError::Orm`](#libraryerror) when a read fails for a reason other than
file-lock contention, and [`LibraryError::Unavailable`](#libraryerror) when a lock is
still held after the retries.

#### LibraryStore::record_event_handler_latency

```rust
pub async fn record_event_handler_latency(&self, duration_ms: i64) -> Result<()>
```

Record one `onEvent` duration sample.

##### Errors

Returns [`LibraryError::Orm`](#libraryerror) when the write fails.

#### LibraryStore::list_domain_events_by_dispatch_state

```rust
pub async fn list_domain_events_by_dispatch_state(
    &self,
    dispatch_state: &str,
    limit: u64,
) -> Result<Vec<DomainEventRecord>>
```

Outbox envelopes in `dispatch_state`, oldest first.

##### Errors

Returns [`LibraryError::Orm`](#libraryerror) when the read fails.

#### LibraryStore::list_dispatched_events_page

```rust
pub async fn list_dispatched_events_page(
    &self,
    created_after: &str,
    after: Option<(&str, &str)>,
    limit: u64,
) -> Result<Vec<DomainEventRecord>>
```

Dispatched events newer than `created_after`, after optional `(created_at, id)` cursor.

##### Errors

Returns [`LibraryError::Orm`](#libraryerror) when the read fails.

#### LibraryStore::upsert_event_subscriber

```rust
pub async fn upsert_event_subscriber(
    &self,
    node_id: &str,
    plugin_id: &str,
    subscriptions: &[EventCatalogSubscription],
    enabled: bool,
) -> Result<()>
```

Heartbeat one node's plugin registration (does not delete other nodes).

##### Errors

Returns [`LibraryError::Orm`](#libraryerror) when the write fails.

#### LibraryStore::upsert_event_subscriber_at

```rust
pub async fn upsert_event_subscriber_at(
    &self,
    node_id: &str,
    plugin_id: &str,
    subscriptions: &[EventCatalogSubscription],
    enabled: bool,
    heartbeat_at: DateTime<Utc>,
) -> Result<()>
```

Heartbeat one node's plugin registration at an explicit timestamp.

##### Errors

Returns [`LibraryError::Orm`](#libraryerror) when the write fails.

#### LibraryStore::list_live_event_subscribers

```rust
pub async fn list_live_event_subscribers(
    &self,
) -> Result<Vec<EventSubscriberCatalogRecord>>
```

Live catalog collapsed by `plugin_id` using the default heartbeat TTL.

##### Errors

Returns [`LibraryError::Orm`](#libraryerror) when the read fails.

#### LibraryStore::list_live_event_subscribers_with_ttl

```rust
pub async fn list_live_event_subscribers_with_ttl(
    &self,
    ttl_secs: i64,
) -> Result<Vec<EventSubscriberCatalogRecord>>
```

Live catalog collapsed by `plugin_id` for a custom heartbeat TTL.

##### Errors

Returns [`LibraryError::Orm`](#libraryerror) when the read fails.

#### LibraryStore::list_live_event_subscriber_nodes

```rust
pub async fn list_live_event_subscriber_nodes(
    &self,
    ttl_secs: i64,
) -> Result<Vec<EventSubscriberNodeRecord>>
```

Live per-node rows whose heartbeat is within `ttl_secs`.

Read in pages of the adapter's `maxResultRows`, keyed by
`(plugin_id, node_id)`, so a catalog larger than one result still
reads on an adapter that does not paginate for the host.

##### Errors

Returns [`LibraryError::Orm`](#libraryerror) when the read fails.

#### LibraryStore::dispatch_catalog_matches

```rust
pub async fn dispatch_catalog_matches(
    &self,
    event: &DomainEventRecord,
    operation_id: &str,
) -> Result<u32>
```

Create deliveries from the live catalog for one event.

##### Errors

Returns [`LibraryError::Orm`](#libraryerror) when the write fails.

#### LibraryStore::reconcile_catalog_deliveries

```rust
pub async fn reconcile_catalog_deliveries(
    &self,
    created_after: DateTime<Utc>,
) -> Result<u32>
```

Late-join catalog deliveries for missing `(event_id, plugin_id)` pairs.

Pages an anti-join of dispatched events in the retention window that
lack a delivery for each live plugin. Unchanged catalogs with no missing
pairs issue bounded empty `SELECT`s and **zero** dispatch writes.

##### Errors

Returns [`LibraryError::Orm`](#libraryerror) when a read or write fails.

#### LibraryStore::request_event_delivery_cancel

```rust
pub async fn request_event_delivery_cancel(
    &self,
    id: &str,
) -> Result<Option<EventDeliveryRecord>>
```

Cancel a pending delivery or flag a running one for cooperative stop.

Pending (including suspended) rows become `rejected`. Running rows set
`cancel_requested` so the heartbeat loop aborts `onEvent`.

##### Errors

Returns [`LibraryError::Orm`](#libraryerror) when the update fails.

#### LibraryStore::event_delivery_cancel_requested

```rust
pub async fn event_delivery_cancel_requested(&self, id: &str) -> Result<bool>
```

True when a running delivery worker should abort after the current step.

##### Errors

Returns [`LibraryError::Orm`](#libraryerror) when the read fails.

#### LibraryStore::resume_event_delivery

```rust
pub async fn resume_event_delivery(&self, id: &str) -> Result<bool>
```

Wake a suspended delivery: `run_after = now` and `resume_pending = 1`.

##### Errors

Returns [`LibraryError::Orm`](#libraryerror) when the update fails.

#### LibraryStore::expire_unleased_event_deliveries

```rust
pub async fn expire_unleased_event_deliveries(
    &self,
    retention_days: u64,
    now: DateTime<Utc>,
) -> Result<u64>
```

Expire deliveries that waited, due but never leased, past their
consumer's retention period.

This is the Cloudflare Queues message retention period applied per
subscriber: a plugin instance that is deployed but not running stops
accumulating work, and catches up from the event stream once it runs.
A delivery is a candidate while it is `pending`, has never been
leased (`attempt_count = 0`), and is not suspended or resuming. It
expires once both its insert time and its `run_after` are older than
the retention of the matching subscription in the live catalog
(`message_retention_secs`, else
[`EVENT_DELIVERY_DEFAULT_RETENTION_SECS`](#event_delivery_default_retention_secs)), capped at
`retention_days`; a subscriber with no live catalog row waits the
full `retention_days`. The matching subscription is the first one of the
subscriber that matches the parent event in full (queue, type, schema
version and filter), as at dispatch. Host consumers (`host:` and node
consumers) never expire and are left out of the scan. Expired rows stay
until [`EventPruneStep::TerminalDeliveries`](#eventprunestep) removes them, so they
record which events a subscriber missed. About 10 000
candidates are examined per call (the budget is checked after each
page, so the last page can go past it); the next call resumes where this one
stopped and wraps around once it reaches the newest candidate.

##### Arguments

* `retention_days` - The events retention window, which caps every
  delivery retention.
* `now` - The time the retention periods are measured back from.

##### Returns

The number of deliveries moved to `expired`.

##### Errors

Returns [`LibraryError::Orm`](#libraryerror) when a read or the update fails.

#### LibraryStore::read_event_stream

```rust
pub async fn read_event_stream(&self, read: &EventStreamRead) -> Result<EventStreamPage>
```

Reads one page of past domain events for `EventPublisher.read`.

Scans the outbox in `(created_at, id)` order from `read.start`, never
before `read.retention_days`, and returns the events any of
`read.subscriptions` matches (type, schema version and filter, as
delivery does), limited to `read.account_id` when set. Queue messages
are never returned. One call examines at most
[`EVENT_STREAM_SCAN_FACTOR`](#event_stream_scan_factor) times `read.limit` rows, in pages that fit
the adapter's bind and result-row limits; when the budget runs out
first the page says `more` and its `last_scanned` resumes the scan.

##### Errors

Returns [`LibraryError::Orm`](#libraryerror) when a read fails.

#### LibraryStore::prune_event_deliveries

```rust
pub async fn prune_event_deliveries(
    &self,
    retention_days: u64,
    dead_letter_retention_days: u64,
) -> Result<u64>
```

Delete terminal deliveries past their retention windows, then empty events.

Acked, rejected and expired rows use `retention_days`. Dead letters use
`dead_letter_retention_days`. Dispatched parent events with no remaining
deliveries are removed only after the same `retention_days` cutoff.
Expired per-node catalog heartbeats are swept here too. Runs every
[`EventPruneStep`](#eventprunestep) in order; a caller that must recheck a lease between
deletes runs [`Self::prune_event_step`](#librarystoreprune_event_step) itself.

##### Errors

Returns [`LibraryError::Orm`](#libraryerror) when the delete fails.

#### LibraryStore::prune_event_step

```rust
pub async fn prune_event_step(
    &self,
    step: EventPruneStep,
    retention_days: u64,
    dead_letter_retention_days: u64,
) -> Result<u64>
```

Runs one delete of [`Self::prune_event_deliveries`](#librarystoreprune_event_deliveries).

##### Arguments

* `step` - Which rows to expire or delete.
* `retention_days` - Window for acked, rejected and expired
  deliveries and empty dispatched events, and the cap on every
  delivery retention period.
* `dead_letter_retention_days` - Window for dead letters.

##### Returns

The number of rows expired or removed.

##### Errors

Returns [`LibraryError::Orm`](#libraryerror) when the delete fails.

#### LibraryStore::mark_holdings_seen

```rust
pub async fn mark_holdings_seen(
    &self,
    plugin_key: &str,
    account_id: &str,
    pass_id: &str,
    product_ids: &[String],
) -> Result<()>
```

Stamps scan pass `pass_id` on the account's holdings of `product_ids`.

A holding is stamped whether or not its listing resolved on this
page, so a title the library model could not settle is never taken
for one that left the account.

##### Arguments

* `plugin_key` - The account's plugin (and the listings').
* `account_id` - The scanned account.
* `pass_id` - The pass in progress (`account_scan_state.pass_id`).
* `product_ids` - The product ids the page listed.

##### Errors

Returns an error when a batch fails.

#### LibraryStore::remove_account_holdings

```rust
pub async fn remove_account_holdings(
    &self,
    plugin_key: &str,
    account_id: &str,
    selection: HoldingSelection<'_>,
) -> Result<usize>
```

Removes the account's holdings that `selection` names, publishing
`holding_removed` for each in the batch that deletes it.

##### Arguments

* `plugin_key` - The account's plugin.
* `account_id` - The account.
* `selection` - Which of its holdings go.

##### Returns

How many holdings were selected for removal (a concurrent writer may
have kept or removed some of them first).

##### Errors

Returns an error when a read or a batch fails; holdings removed by
earlier batches stay removed.

#### LibraryStore::remove_node_holdings

```rust
pub async fn remove_node_holdings(
    &self,
    node_id: &str,
    selection: NodeHoldingSelection<'_>,
) -> Result<usize>
```

Removes the holdings of branch `node_id` that `selection` names,
publishing `holding_removed` for each in the batch that deletes it.
Files already delivered for them stay with their editions.

##### Returns

How many holdings were selected for removal.

##### Errors

Returns an error when a read or a batch fails; holdings removed by
earlier batches stay removed.

#### LibraryStore::refresh_library_works

```rust
pub async fn refresh_library_works(&self, work_ids: &[String]) -> Result<u64>
```

Refreshes the page index rows of `work_ids` (an episode refreshes its
show): writes each work's row (an unheld work's with a holding count
of 0, which pages skip) and deletes the row of a work that is gone.

##### Arguments

* `work_ids` - Works whose editions, listings or holdings changed.

##### Returns

How many rows were written or deleted (a write skipped because a
later refresh already wrote the row is not counted).

##### Errors

Returns [`LibraryError::Orm`](#libraryerror) or a batch error when a read or write
fails.

#### LibraryStore::rebuild_library_index_page

```rust
pub async fn rebuild_library_index_page(
    &self,
    after: Option<&str>,
) -> Result<Option<String>>
```

Refreshes one page of the rebuild: the works with an id after
`after`, in id order, as [`LibraryStore::refresh_library_works`](#librarystorerefresh_library_works)
does.

##### Arguments

* `after` - The last work id of the previous page; `None` to start.

##### Returns

The last work id of this page, or `None` when the walk is done.

##### Errors

Returns [`LibraryError::Orm`](#libraryerror) or a batch error when a read or write
fails.

#### LibraryStore::top_level_work_ids_page

```rust
pub async fn top_level_work_ids_page(
    &self,
    after: Option<&str>,
    limit: u64,
) -> Result<Vec<String>>
```

One page of top-level work ids (podcast shows, not their episodes)
in id order, for a walk over every work.

##### Arguments

* `after` - The last id of the previous page; `None` to start.
* `limit` - Most ids to return, further capped at the database's
  result-row limit.

##### Errors

Returns [`LibraryError::Orm`](#libraryerror) when the read fails.

#### LibraryStore::library_index_built

```rust
pub async fn library_index_built(&self) -> Result<bool>
```

True when the page index has a finished build of
[`LIBRARY_INDEX_LAYOUT`](#library_index_layout).

##### Errors

Returns [`LibraryError::Orm`](#libraryerror) when the read fails.

#### LibraryStore::drop_library_index_build

```rust
pub async fn drop_library_index_build(&self) -> Result<()>
```

Drops the finished build of [`LIBRARY_INDEX_LAYOUT`](#library_index_layout), so the next
heartbeat queues a rebuild of every row: for a change, such as the
library's collections, that the rows' derivation reads but no
work's events carry.

##### Errors

Returns a batch error when the write fails.

#### LibraryStore::finish_library_index_build

```rust
pub async fn finish_library_index_build(&self) -> Result<()>
```

Records a finished build of [`LIBRARY_INDEX_LAYOUT`](#library_index_layout).

##### Errors

Returns a batch error when the write fails.

#### LibraryStore::work_of_edition

```rust
pub async fn work_of_edition(&self, edition_id: &str) -> Result<Option<String>>
```

The works an edition belongs to (one, unless it was just moved).

##### Errors

Returns [`LibraryError::Orm`](#libraryerror) when the read fails.

#### LibraryStore::work_of_listing

```rust
pub async fn work_of_listing(&self, listing_id: &str) -> Result<Option<String>>
```

The work a listing's edition belongs to.

##### Errors

Returns [`LibraryError::Orm`](#libraryerror) when the read fails.

#### LibraryStore::library_works_by_ids

```rust
pub async fn library_works_by_ids(
    &self,
    ids: &[String],
    query: &WorkPageQuery,
) -> Result<Vec<LibraryWorkRow>>
```

The rows of `ids` that pass `query`'s filters, in the order of
`ids` (a search's relevance order); `query`'s order, cursor and
limit are not used. Unheld works and ids with no row are left out.

##### Arguments

* `ids` - Top-level work ids.
* `query` - The filters to apply.

##### Errors

Returns [`LibraryError::Orm`](#libraryerror) when a read fails.

#### LibraryStore::work_in_scope

```rust
pub async fn work_in_scope(&self, work_id: &str, scope: &WorkScope) -> Result<bool>
```

Whether a work is inside a patron scope: its top-level work (the
show, for an episode) is a held work of the page index that passes
`scope`.

##### Errors

Returns [`LibraryError::Orm`](#libraryerror) when a read fails.

#### LibraryStore::edition_in_scope

```rust
pub async fn edition_in_scope(
    &self,
    edition_id: &str,
    scope: Option<&WorkScope>,
) -> Result<bool>
```

Whether an edition is inside a patron scope: no scope admits every
edition, and a scope admits one whose work passes
[`Self::work_in_scope`](#librarystorework_in_scope). An edition that belongs to no work is
outside every scope, so a scoped patron never learns of it.

##### Arguments

* `edition_id` - The edition (`editions.id`).
* `scope` - The patron's filter, `None` when nothing is hidden.

##### Errors

Returns [`LibraryError::Orm`](#libraryerror) when a read fails; callers fail closed.

#### LibraryStore::library_facet_values

```rust
pub async fn library_facet_values(
    &self,
    of: FacetValuesOf,
    after: Option<&str>,
    limit: u64,
    scope: Option<&WorkScope>,
) -> Result<FacetValuesPage>
```

One page of the values a facet has across the held works, by key.

##### Arguments

* `of` - The facet, or the user whose tags to list.
* `after` - The `next` key of the previous page; `None` to start.
* `limit` - Most values to return; at least 1, further capped at the
  database's result-row limit.
* `scope` - Count only the works inside this patron scope; `None`
  for a caller who sees the whole library.

##### Errors

Returns [`LibraryError::Orm`](#libraryerror) when the read fails.

#### LibraryStore::library_works_page

```rust
pub async fn library_works_page(&self, query: &WorkPageQuery) -> Result<WorkPage>
```

One page of the library's works from the page index.

##### Arguments

* `query` - Order, filters, cursor and page size.

##### Errors

Returns [`LibraryError::Invalid`](#libraryerror) when the cursor belongs to another
order, or [`LibraryError::Orm`](#libraryerror) when the read fails.

#### LibraryStore::library_works_at

```rust
pub async fn library_works_at(
    &self,
    query: &WorkPageQuery,
    offset: u64,
    limit: u64,
    reversed: bool,
) -> Result<(Vec<LibraryWorkRow>, u64)>
```

One page of the library's works by position: `limit` rows from
`offset` in `query`'s order (reversed when `reversed`), and how
many rows the query matches in all. `query.after` and
`query.limit` are not read.

For callers that page by number, such as player apps; the library's
own pages use [`Self::library_works_page`](#librarystorelibrary_works_page), whose cursors hold
still while rows are added.

##### Arguments

* `query` - Order and filters.
* `offset` - Rows to skip.
* `limit` - Most rows to return.
* `reversed` - True to read the order backwards.

##### Errors

Returns [`LibraryError::Orm`](#libraryerror) when a read fails.

#### LibraryStore::owned_work_for

```rust
pub async fn owned_work_for(&self, query: &TitleQuery<'_>) -> Result<Option<String>>
```

The work in the library that `query` belongs to, when the library
holds one.

The query is resolved exactly as a scanned listing would be (shared
identifiers first, then the work's identifiers, then the work-level
title-and-author match), but nothing is written. A query that would
make a new work, or whose work nothing in the library holds, is not
owned: *Dune Messiah* is its own work beside *Dune*. A query that
cannot form a valid listing (a blank title, or a field over the
listing limits) matches no work, so it is not owned either.

##### Errors

Returns [`LibraryError`](#libraryerror) when a read fails.

#### LibraryStore::work_for

```rust
pub async fn work_for(&self, query: &TitleQuery<'_>) -> Result<Option<String>>
```

The existing work `query` belongs to, held by a library title or
not, when the resolver finds one; `None` when it would make a new
work. Nothing is written. A wish stores this id when it is created,
so the queue and the shelves compare works, not words.

##### Errors

Returns [`LibraryError`](#libraryerror) when a read fails. A query that cannot form
a valid listing (a blank title, or a field over the listing limits)
names no work.

#### LibraryStore::existing_edition_work

```rust
pub async fn existing_edition_work(&self, listing: &Listing) -> Result<Option<String>>
```

The work of the stored edition `listing` would join, without
writing anything: `Some` only when the resolver places it on an
edition this library already has (through a shared identifier), so
no new work, expression or edition, and no title match, is involved.

##### Errors

Returns [`LibraryError`](#libraryerror) when a read fails.

#### LibraryStore::work_for_wish

```rust
pub async fn work_for_wish(
    &self,
    query: &TitleQuery<'_>,
) -> Result<(String, Option<String>)>
```

The work a wish for `query` is for, made when the library has none.

The query is resolved exactly as [`Self::work_for`](#librarystorework_for) resolves it. When
that names an existing work (held by a library title or not), that is
the work. Otherwise a catalog work is written as one atomic batch:
the work, an expression and an edition carrying the wish's ASIN and
ISBN (a non-acquirable catalog edition, as the data model ADR keeps
catalog results), then their credits and series. No store listing or
holding is written, so the library does not hold it. A later wish
with the same identifiers joins that edition, and a later scan of the
book finds the work through the work matcher (same title, authors
and kind), so wishes and the scanned title share the work.

Every id is derived (the work's from its match key, the edition's
from the query, the expression's from both) and every row is
inserted with `INSERT OR IGNORE`, so two wishes for one new book made
at once (on one node or two) name the same rows.

##### Arguments

* `query` - The wished title, authors, identifiers and series.

##### Returns

The work, and the existing edition the query's identifiers reached
(`None` when they reached none).

##### Errors

Returns [`LibraryError::Invalid`](#libraryerror) when the query cannot form a valid
listing (a blank title, or a field over the listing limits), and
[`LibraryError`](#libraryerror) when a read or write fails.

#### LibraryStore::record_listing

```rust
pub async fn record_listing(
    &self,
    listing: &Listing,
    holding: &HoldingDraft,
) -> Result<Plan>
```

Resolves `listing` into the library and records `holding` of it.

##### Arguments

* `listing` - The validated listing a storefront reported.
* `holding` - Who holds it, and how.

##### Returns

The plan that was written (a no-op plan when nothing changed).

##### Errors

Returns [`LibraryError`](#libraryerror) when a read or write fails, when the stored
rows are inconsistent, or when other writers kept changing the rows
for three attempts.

#### LibraryStore::record_catalog_listing

```rust
pub async fn record_catalog_listing(&self, listing: &Listing) -> Result<Plan>
```

Resolves a catalog `listing` (a title a store sells that nobody in
the library holds) into the library: its work, expression, edition
and `store_listings` row, with no holding.

A later scan of the same product finds the same listing row and adds
only its holding, so a catalog listing and a held one never diverge.

##### Returns

The plan that was written (a no-op plan when nothing changed).

##### Errors

As [`Self::record_listing`](#librarystorerecord_listing).

#### LibraryStore::works_by_ids

```rust
pub async fn works_by_ids(&self, ids: &[String]) -> Result<BTreeMap<String, WorkRecord>>
```

Loads many works by id, with their work-level credits, narrators and
series, in chunked bulk reads.

##### Arguments

* `ids` - Work ids; unknown ids are left out of the result.

##### Returns

The works found, keyed by id.

##### Errors

Returns [`LibraryError::Orm`](#libraryerror) when a read fails.

#### LibraryStore::get_work

```rust
pub async fn get_work(&self, id: &str) -> Result<Option<WorkRecord>>
```

Loads a work by id, with its work-level credits and series.

##### Errors

Returns [`LibraryError::Orm`](#libraryerror) when a read fails.

#### LibraryStore::list_works_page

```rust
pub async fn list_works_page(
    &self,
    after_id: Option<&str>,
    limit: u64,
) -> Result<Vec<WorkRecord>>
```

Lists up to `limit` works with ids after `after_id`, in id order, with
their work-level credits and series.

A keyset page: pass the last id of one page as `after_id` of the
next, and stop at a short page. Unlike [`Self::list_works`](#librarystorelist_works) it never
reads the whole table at once.

##### Arguments

* `after_id` - Exclusive lower bound on `works.id`; `None` starts at
  the first work.
* `limit` - Page size, clamped to 1..=500.

##### Errors

Returns [`LibraryError::Orm`](#libraryerror) when a read fails.

#### LibraryStore::list_works

```rust
pub async fn list_works(&self) -> Result<Vec<WorkRecord>>
```

Lists every work by title, with its work-level credits and series.

##### Errors

Returns [`LibraryError::Orm`](#libraryerror) when a read fails.

#### LibraryStore::work_of_holding

```rust
pub async fn work_of_holding(&self, holding_id: &str) -> Result<Option<String>>
```

The work holding `holding_id` (`holdings.id`) reaches through its
listing's edition; `None` when the holding or its listing is gone.

##### Errors

Returns [`LibraryError::Orm`](#libraryerror) when a read fails.

#### LibraryStore::works_of_editions

```rust
pub async fn works_of_editions(
    &self,
    edition_ids: BTreeSet<String>,
) -> Result<BTreeMap<String, String>>
```

Each edition's work, through its expression, read in chunks that
fit the database's limits.

##### Returns

Edition id to work id; an edition that no longer exists, or whose
expression row is missing, is left out.

##### Errors

Returns [`LibraryError::Orm`](#libraryerror) when a read fails.

#### LibraryStore::set_work_enrichment

```rust
pub async fn set_work_enrichment(
    &self,
    work_id: &str,
    enrichment: &WorkEnrichment,
) -> Result<()>
```

Fills a work's enrichment fields; `None` keeps the stored value.

##### Arguments

* `work_id` - The work.
* `enrichment` - Description, subjects and cover found by enrichment.

##### Errors

Returns [`LibraryError::NotFound`](#libraryerror) when the work is gone, or
[`LibraryError::Orm`](#libraryerror) when the write fails.

#### LibraryStore::add_work_identifier

```rust
pub async fn add_work_identifier(
    &self,
    work_id: &str,
    scheme: IdentifierScheme,
    raw: &str,
) -> Result<()>
```

Attaches a work-level identifier enrichment found (an OpenLibrary
work id); a value another work already holds is left there.

##### Errors

Returns [`LibraryError::Orm`](#libraryerror) when the write fails.

#### LibraryStore::match_listening_items

```rust
pub async fn match_listening_items(
    &self,
    items: &[ListeningIdentifiers],
) -> Result<Vec<ListeningMatch>>
```

Resolves many listening items to works in a few bulk lookups.

The item's ASIN is tried first, then its ISBN; the first one that
names a work decides it. For that identifier, the work comes from the
editions whose `edition_identifiers` carry it (an ASIN in any
marketplace), used only when those editions agree on one work; the
edition is kept when exactly one carries it. Titles are never
compared.

##### Arguments

* `items` - Each item's identifiers.

##### Returns

One match per item, in the same order; an item no identifier
resolves has neither a work nor an edition.

##### Errors

Returns [`LibraryError::Orm`](#libraryerror) when a read fails.

#### LibraryStore::portal_identity_ids

```rust
pub async fn portal_identity_ids(
    &self,
    provider: &str,
    external_user_ids: &BTreeSet<String>,
) -> Result<BTreeMap<String, i64>>
```

Portal identity ids of many external users of one provider.

##### Arguments

* `provider` - Identity provider / integration id.
* `external_user_ids` - User ids at that provider.

##### Returns

`portal_identities.id` keyed by external user id; users with no
identity are absent.

##### Errors

Returns [`LibraryError::Orm`](#libraryerror) when a read fails.

#### LibraryStore::upsert_listening_progress_batch

```rust
pub async fn upsert_listening_progress_batch(
    &self,
    rows: &[NewListeningProgress],
) -> Result<Vec<ListeningProgressWrite>>
```

Stores many listening rows in a few chunked reads and atomic batches.

Reports for one `(provider, external_user_id, external_item_id)` are
merged in order over the stored row, exactly as successive calls to
[`Self::upsert_listening_progress`](#librarystoreupsert_listening_progress) would merge them: optional text
and the identity keep their stored value when a report leaves them
out, progress fields take the report's. Each row is written once.

The writes are split into atomic batches under the database plugin's
`maxStatements` and `maxRequestBytes`; a row's two statements always
share a batch. No interactive transaction is used, so a reader
between two batches can see part of the sync.

##### Arguments

* `rows` - Reports to store, in the order the provider sent them.

##### Returns

One write per distinct row key, in the order the keys first appear.

##### Errors

Returns the database error. Batches sent before a failing one stay
written.

#### LibraryStore::route_notification

```rust
pub async fn route_notification(
    &self,
    new: &NewNotification,
    limits: NotifyQueueLimits,
) -> Result<Routed>
```

Accepts a notification: stores it, writes one delivery-log row per
destination the routing rules pick and, for a patron, the inbox row,
in one batch; then enqueues one `notify_send` job per delivery.

##### Arguments

* `new` - The notification, already rendered.
* `limits` - The queue limits the send jobs are admitted under.

##### Returns

The stored id and the deliveries made.

##### Errors

[`LibraryError::Invalid`](#libraryerror) when the notification breaks a message
rule; [`LibraryError::NotFound`](#libraryerror) when its patron does not exist;
[`LibraryError::Conflict`](#libraryerror) when route replacements kept landing while
it was routed; an error when a read or the batch fails. A failed
enqueue is not an error: the delivery waits for the sweep.

#### LibraryStore::route_notification_to

```rust
pub async fn route_notification_to(
    &self,
    new: &NewNotification,
    destination_id: &str,
    limits: NotifyQueueLimits,
) -> Result<Routed>
```

Stores `new` with one delivery to destination `destination_id`
alone, whatever the routes say: an administrator's test of a
destination.

##### Errors

[`LibraryError::Invalid`](#libraryerror) when the notification breaks a message
rule; [`LibraryError::NotFound`](#libraryerror) when the destination does not
exist; an error when a read or the batch fails.

#### LibraryStore::user_holds_account

```rust
pub async fn user_holds_account(
    &self,
    user_id: i64,
    plugin_key: &str,
    account_id: &str,
) -> Result<bool>
```

Whether user `user_id` holds store account (`plugin_key`,
`account_id`) through one of their linked identities: the check a
plugin's patron raise must pass inside its invocation's account
scope.

##### Errors

An error when a read fails.

#### LibraryStore::enqueue_notify_send

```rust
pub async fn enqueue_notify_send(
    &self,
    delivery: &PendingDelivery,
    limits: NotifyQueueLimits,
) -> Result<Option<String>>
```

Enqueues the `notify_send` job of one delivery and records its id.

##### Returns

The job id, or `None` when the queue is full (the sweep tries again).

##### Errors

An error when the enqueue or the write fails.

#### LibraryStore::unscheduled_notify_deliveries

```rust
pub async fn unscheduled_notify_deliveries(
    &self,
    limit: u64,
) -> Result<Vec<PendingDelivery>>
```

Pending deliveries no send job was recorded for, oldest first: the
ones whose enqueue failed or found the queue full.

##### Errors

An error when the read fails.

#### LibraryStore::drop_orphaned_notify_deliveries

```rust
pub async fn drop_orphaned_notify_deliveries(&self) -> Result<u64>
```

Drops pending deliveries whose send job ended without recording a
result (an operator cancelled it, or it failed on a host error), so
the sweep does not send them again behind the operator's back and
the delivery log says what happened.

##### Returns

How many deliveries were dropped.

##### Errors

An error when the batch fails.

#### LibraryStore::stalled_notify_sends

```rust
pub async fn stalled_notify_sends(
    &self,
    ready_before: DateTime<Utc>,
    after: Option<&str>,
    limit: u64,
) -> Result<(Vec<StalledSend>, Option<String>)>
```

One page of send jobs still pending although they have been ready
to run since before `ready_before`, with the pending delivery and
notifier instance of each when it still has one. Whether a job is
stalled or only waiting behind other work depends on whether its
notifier runs anywhere, which the caller decides.

Pending send jobs are walked by id, `after` the last id of the
previous page (`None` for the first), at most `limit` rows a page;
the second value is the cursor of the next page, `None` after the
last. Only the id, readiness, lease generation and dedup key columns
are read, so a page stays narrow under the database's result-size
cap, and a page the database still refuses for its size is read
again at half the rows. Readiness is compared here rather than in
SQL because stored timestamps carry a varying number of fractional
digits, so they do not order as text.

The delivery comes from the job's dedup key (it names the delivery
the job was enqueued for), not from the delivery's `job_id`, so a
job whose link write failed is still matched with its delivery.

##### Errors

An error when a read fails.

#### LibraryStore::fail_stalled_notify_send

```rust
pub async fn fail_stalled_notify_send(
    &self,
    stalled: &StalledSend,
    reason: &str,
) -> Result<StallOutcome>
```

Cancels one stalled send job and fails its pending delivery in a
single atomic write, so a claim racing the sweep either wins (and
nothing changes) or loses (and the failure is counted toward the
destination's pause), and the cancel never lands without the count.
A delivery whose destination is already paused is dropped instead,
with no attempt or failure counted, as a send would drop it. Only
call this for a job whose notifier runs nowhere; a job that is only
waiting for a worker is not a notifier failure.

##### Returns

What happened; [`StallOutcome::NotStalled`](notifications.md#stalloutcome) when the job was
claimed, retried or ended since it was read.

##### Errors

An error when the batch fails.

#### LibraryStore::drop_notify_delivery_if_job_ended

```rust
pub async fn drop_notify_delivery_if_job_ended(&self, delivery_id: &str) -> Result<bool>
```

Drops a pending delivery no send job is linked to when a send job
for it already ended: its enqueue succeeded but the link write did
not, and the job was then cancelled or failed. Without this the
sweep would queue a second send behind the operator's back, since
the ended job no longer blocks a new one with the same dedup key.

##### Returns

Whether the delivery was dropped (and must not be enqueued again).

##### Errors

An error when the read or the batch fails.

#### LibraryStore::notify_send_work

```rust
pub async fn notify_send_work(&self, delivery_id: &str) -> Result<Option<SendWork>>
```

Everything the send job of `delivery_id` needs; `None` when the row
is gone (pruned, or its notification or destination removed).

##### Errors

An error when a read fails or a stored value is malformed.

#### LibraryStore::record_notify_delivered

```rust
pub async fn record_notify_delivered(
    &self,
    delivery_id: &str,
    provider_id: Option<&str>,
    fence: Option<&JobFence>,
) -> Result<bool>
```

Records a delivered send: the delivery is done and its destination's
failure count is cleared.

##### Returns

`false` when the delivery was no longer pending (a replayed job).

##### Arguments

* `fence` - The send job's claim; the write is skipped (and answers
  as if the delivery were no longer pending) once that claim has lost
  the job.

##### Errors

An error when the batch fails.

#### LibraryStore::record_notify_retry

```rust
pub async fn record_notify_retry(
    &self,
    delivery_id: &str,
    reason: &str,
    fence: Option<&JobFence>,
) -> Result<Option<i64>>
```

Records a send attempt the notifier asked to retry.

##### Returns

The attempts made, this one included; `None` when the delivery was
no longer pending.

##### Arguments

* `fence` - The send job's claim; the write is skipped (and answers
  as if the delivery were no longer pending) once that claim has lost
  the job.

##### Errors

An error when the batch or the read fails.

#### LibraryStore::record_notify_failed

```rust
pub async fn record_notify_failed(
    &self,
    delivery_id: &str,
    reason: &str,
    attempt: FailedAttempt,
    fence: Option<&JobFence>,
) -> Result<Option<FailureEffect>>
```

Records a send the notifier refused for good (or that ran out of
attempts): the delivery fails and its destination's failure count
rises, pausing it at [`PAUSE_AFTER_FAILURES`](notifications.md#pause_after_failures).

##### Returns

Whether this failure paused the destination; `None` when the
delivery was no longer pending.

##### Arguments

* `fence` - The send job's claim; the write is skipped (and answers
  as if the delivery were no longer pending) once that claim has lost
  the job.
* `attempt` - Whether this answer spends an attempt of its own.

##### Errors

An error when the batch or the read fails.

#### LibraryStore::record_notify_dropped

```rust
pub async fn record_notify_dropped(
    &self,
    delivery_id: &str,
    reason: &str,
    fence: Option<&JobFence>,
) -> Result<bool>
```

Records a delivery that was not sent (its destination is paused or
gone, or its notifier instance is gone). The destination's failure
count is untouched.

##### Returns

`false` when the delivery was no longer pending.

##### Arguments

* `fence` - The send job's claim; the write is skipped (and answers
  as if the delivery were no longer pending) once that claim has lost
  the job.

##### Errors

An error when the batch fails.

#### LibraryStore::create_notify_destination

```rust
pub async fn create_notify_destination(&self, new: &NewDestination) -> Result<Destination>
```

Creates a destination.

##### Errors

[`LibraryError::Invalid`](#libraryerror) for a bad label, a channel with an owner or
an address, a personal destination without an owner, or one past
the owner's [`MAX_ROUTES_PER_TYPE`](notifications.md#max_routes_per_type) addresses;
[`LibraryError::NotFound`](#libraryerror) when the owner does not exist; an error
when the batch fails.

#### LibraryStore::notify_destination

```rust
pub async fn notify_destination(&self, id: &str) -> Result<Option<Destination>>
```

One destination by id.

##### Errors

An error when the read fails or the row is malformed.

#### LibraryStore::notify_channels

```rust
pub async fn notify_channels(&self) -> Result<Vec<Destination>>
```

Every library channel, oldest first.

##### Errors

An error when the read fails.

#### LibraryStore::personal_destinations

```rust
pub async fn personal_destinations(&self, user_id: i64) -> Result<Vec<Destination>>
```

One patron's personal destinations, oldest first.

##### Errors

An error when the read fails.

#### LibraryStore::rename_notify_destination

```rust
pub async fn rename_notify_destination(&self, id: &str, label: &str) -> Result<()>
```

Renames a destination.

##### Errors

[`LibraryError::Invalid`](#libraryerror) for a bad label;
[`LibraryError::NotFound`](#libraryerror) when the destination does not exist.

#### LibraryStore::resume_notify_destination

```rust
pub async fn resume_notify_destination(&self, id: &str) -> Result<()>
```

Resumes a paused destination and clears its failure count.

##### Errors

[`LibraryError::NotFound`](#libraryerror) when the destination does not exist.

#### LibraryStore::delete_notify_destination

```rust
pub async fn delete_notify_destination(&self, id: &str) -> Result<()>
```

Deletes a destination with its routes, patron targets and delivery
log rows.

##### Errors

[`LibraryError::NotFound`](#libraryerror) when the destination does not exist.

#### LibraryStore::notify_routes

```rust
pub async fn notify_routes(&self) -> Result<BTreeMap<String, Vec<Route>>>
```

Every operational route, by type; an "In app" route comes first in
its type's list under [`IN_APP_DESTINATION`](notifications.md#in_app_destination).

##### Errors

An error when the read fails or a stored severity is malformed.

#### LibraryStore::set_notify_routes

```rust
pub async fn set_notify_routes(
    &self,
    notification_type: &str,
    routes: &[Route],
) -> Result<()>
```

Replaces the routes of one operational type. A route to
[`IN_APP_DESTINATION`](notifications.md#in_app_destination) is the staff inbox, not a stored destination.

##### Errors

[`LibraryError::Invalid`](#libraryerror) for too many routes, a repeated destination
or one that is not a channel; [`LibraryError::NotFound`](#libraryerror) when a
destination does not exist.

#### LibraryStore::patron_offers

```rust
pub async fn patron_offers(&self) -> Result<BTreeMap<String, Vec<PatronOffer>>>
```

Every patron offer, by type.

##### Errors

An error when the read fails.

#### LibraryStore::set_patron_offers

```rust
pub async fn set_patron_offers(
    &self,
    notification_type: &str,
    offers: &[PatronOffer],
) -> Result<()>
```

Replaces the instances offered to patrons for one patron type.

##### Errors

[`LibraryError::Invalid`](#libraryerror) for too many offers or a repeated instance.

#### LibraryStore::patron_choices

```rust
pub async fn patron_choices(&self, user_id: i64) -> Result<Vec<PatronChoice>>
```

One patron's choices, by type.

##### Errors

An error when a read fails.

#### LibraryStore::set_patron_choice

```rust
pub async fn set_patron_choice(&self, user_id: i64, choice: &PatronChoice) -> Result<()>
```

Stores one patron's choice for one patron type, replacing the last.

##### Errors

[`LibraryError::Invalid`](#libraryerror) for too many or repeated destinations, or,
when the choice is enabled, one on a notifier not offered for the
type;
[`LibraryError::NotFound`](#libraryerror) when a destination does not exist.

#### LibraryStore::clear_patron_choice

```rust
pub async fn clear_patron_choice(
    &self,
    user_id: i64,
    notification_type: &str,
) -> Result<()>
```

Drops one patron's choice for a type, so the defaults apply again.

##### Errors

An error when the batch fails.

#### LibraryStore::notify_deliveries

```rust
pub async fn notify_deliveries(
    &self,
    destination_id: Option<&str>,
    before: Option<&NotifyCursor>,
    limit: u64,
) -> Result<NotifyPage<DeliveryRecord>>
```

One page of the delivery log, newest first.

##### Arguments

* `destination_id` - Only this destination's rows, when set.
* `before` - Where the page starts (the previous page's last row).
* `limit` - Page size, clamped to `1..=`[`MAX_NOTIFY_PAGE`](#max_notify_page).

##### Errors

An error when a read fails or a stored value is malformed.

#### LibraryStore::inbox

```rust
pub async fn inbox(
    &self,
    user_id: i64,
    before: Option<&NotifyCursor>,
    limit: u64,
) -> Result<NotifyPage<InboxEntry>>
```

One page of a patron's inbox, newest first.

##### Arguments

* `user_id` - The patron.
* `before` - Where the page starts (the previous page's last entry;
  its id is the notification id).
* `limit` - Page size, clamped to `1..=`[`MAX_NOTIFY_PAGE`](#max_notify_page).

##### Errors

An error when a read fails or a stored value is malformed.

#### LibraryStore::inbox_unread

```rust
pub async fn inbox_unread(&self, user_id: i64) -> Result<u64>
```

How many entries in a patron's inbox are unread.

##### Errors

An error when the read fails.

#### LibraryStore::mark_inbox_read

```rust
pub async fn mark_inbox_read(
    &self,
    user_id: i64,
    notification_id: Option<&str>,
) -> Result<u64>
```

Marks one inbox entry read, or every unread one when
`notification_id` is `None`.

##### Returns

How many entries changed.

##### Errors

[`LibraryError::NotFound`](#libraryerror) when the named entry is not in this
patron's inbox; an error when the batch fails.

#### LibraryStore::prune_notifications

```rust
pub async fn prune_notifications(&self, now: DateTime<Utc>) -> Result<u64>
```

Deletes notifications older than [`NOTIFICATION_RETENTION_DAYS`](notifications.md#notification_retention_days) as
of `now`, with their inbox entries and delivery-log rows.

##### Returns

How many notifications were deleted.

##### Errors

An error when the batch fails; nothing is deleted then.

#### LibraryStore::pin_listing

```rust
pub async fn pin_listing(
    &self,
    listing_id: &str,
    edition_id: &str,
    user_id: Option<i64>,
) -> Result<PinOutcome>
```

Pins `listing_id` to `edition_id`: records the Administrator's
decision and moves the listing there, as one atomic batch.

Every statement requires both the listing and the edition to exist,
so an unknown id writes nothing. The listing's recorded conflicts are
dropped (the pin settles them). When the listing moves, the batch
publishes `edition_linked` for `edition_id` and `edition_updated` for
the edition it left; both are published only while this batch's pin
is the stored one.

##### Arguments

* `listing_id` - The listing (`store_listings.id`).
* `edition_id` - The edition it sells (`editions.id`).
* `user_id` - The Administrator (`users.id`); `None` for the operator.

##### Returns

What the pin did.

##### Errors

Returns the database error of a failed read or write.

#### LibraryStore::unpin_listing

```rust
pub async fn unpin_listing(&self, listing_id: &str) -> Result<bool>
```

Removes the pin on `listing_id`, if any.

The listing keeps selling the edition it was pinned to: the resolver
keeps a stored listing's edition, so a pin only ever moves a listing
when it is made.

##### Returns

Whether a pin was removed.

##### Errors

Returns the database error of a failed read or write.

#### LibraryStore::listing_pin

```rust
pub async fn listing_pin(&self, listing_id: &str) -> Result<Option<String>>
```

The edition `listing_id` is pinned to, or `None` when it has no pin.

##### Errors

Returns [`LibraryError::Orm`](#libraryerror) when the read fails.

#### LibraryStore::list_resolution_conflicts

```rust
pub async fn list_resolution_conflicts(
    &self,
    after: Option<&ConflictCursor>,
    limit: u64,
) -> Result<(Vec<ResolutionConflict>, Option<ConflictCursor>)>
```

One page of the conflicts the resolver recorded, in key order
(listing, kind, subjects), with each listing's store and title.

##### Arguments

* `after` - Where the previous page stopped; `None` starts at the
  first conflict.
* `limit` - Page size, clamped to 1..=[`MAX_CONFLICT_PAGE`](#max_conflict_page) and to
  the database plugin's result-row cap.

##### Returns

The page, and the cursor of the next page when this one is full.

##### Errors

Returns [`LibraryError::Orm`](#libraryerror) when a read fails, or
[`LibraryError::Invalid`](#libraryerror) when a stored subject list is not a JSON
array of strings.

#### LibraryStore::record_resolution_conflict_for_test

```rust
pub async fn record_resolution_conflict_for_test(
    &self,
    listing_id: &str,
    kind: &str,
    subject_ids: &[&str],
)
```

Records a resolver conflict for `listing_id`, as a scan that could
not decide would. Test support only.

##### Panics

When the write fails.

#### LibraryStore::account_pace_state

```rust
pub async fn account_pace_state(
    &self,
    plugin_key: &str,
    account_id: &str,
    now: DateTime<Utc>,
) -> Result<AccountPaceState>
```

Reads one account's pacing state at `now`.

##### Arguments

* `plugin_key` - The account's plugin.
* `account_id` - The account.
* `now` - The current time (the window ends here).

##### Errors

Returns [`LibraryError::Orm`](#libraryerror) when a read fails.

#### LibraryStore::set_account_pace_override

```rust
pub async fn set_account_pace_override(
    &self,
    plugin_key: &str,
    account_id: &str,
    override_: &PaceOverride,
) -> Result<()>
```

Stores the operator's override of an account's pace (the cool-down
is kept), and wakes parked paced acquires that may download on it so
they look at the new pace ([`Self::wake_paced_acquires`](#librarystorewake_paced_acquires)).

##### Errors

Returns [`LibraryError::Other`](#libraryerror) when the override is out of range
([`PaceOverride::validate`](pacing.md#paceoverridevalidate)), or an error when the batch fails.

#### LibraryStore::extend_account_cooldown

```rust
pub async fn extend_account_cooldown(
    &self,
    plugin_key: &str,
    account_id: &str,
    until: DateTime<Utc>,
) -> Result<()>
```

Records the store's cool-down for an account, unless one that ends
later is already recorded.

##### Errors

Returns an error when the batch fails.

#### LibraryStore::clear_account_cooldown

```rust
pub async fn clear_account_cooldown(
    &self,
    plugin_key: &str,
    account_id: &str,
) -> Result<()>
```

Clears an account's cool-down (the operator knows better), and wakes
parked paced acquires that may download on it
([`Self::wake_paced_acquires`](#librarystorewake_paced_acquires)).

##### Errors

Returns an error when the batch fails.

#### LibraryStore::record_account_download

```rust
pub async fn record_account_download(
    &self,
    id: &DownloadId,
    plugin_key: &str,
    account_id: &str,
    started_at: DateTime<Utc>,
    job_id: Option<&str>,
    starter: DownloadStarter,
) -> Result<()>
```

Records a download that reached the store, and prunes the account's
ledger rows older than [`LEDGER_RETENTION`](pacing.md#ledger_retention), in one batch.

Writing the same `id` again adds no second row, so a write whose
answer was lost may be retried.

##### Arguments

* `id` - The download's identity, the same for every write attempt.
* `plugin_key` - The account's plugin.
* `account_id` - The account.
* `started_at` - When the download started.
* `job_id` - The acquire job that ran it, if any.
* `starter` - Whether the host or a person started it.

##### Errors

Returns an error when the batch fails.

#### LibraryStore::playback_edition

```rust
pub async fn playback_edition(&self, edition_id: &str) -> Result<Option<PlaybackEdition>>
```

The edition a player asks about, with its work and catalog runtime.

##### Returns

`None` for an unknown edition.

##### Errors

Returns the read error.

#### LibraryStore::playback_progress

```rust
pub async fn playback_progress(
    &self,
    user_id: i64,
    edition_id: &str,
) -> Result<Option<PlaybackProgress>>
```

One person's progress in one edition.

##### Errors

Returns the read error.

#### LibraryStore::list_playback_progress

```rust
pub async fn list_playback_progress(
    &self,
    user_id: i64,
    include_finished: bool,
    after: Option<&ProgressCursor>,
    limit: u64,
) -> Result<(Vec<PlaybackProgress>, Option<ProgressCursor>)>
```

A page of one person's progress rows, most recently played first.

##### Arguments

* `user_id` - The listener.
* `include_finished` - False lists only editions not yet finished
  (Continue listening).
* `after` - The last row of the page before; `None` for the first.
* `limit` - Most rows to return.

##### Returns

The rows, and the cursor of the next page when there may be one.

##### Errors

Returns the read error.

#### LibraryStore::set_playback_progress

```rust
pub async fn set_playback_progress(
    &self,
    user_id: i64,
    edition_id: &str,
    work_id: Option<&str>,
    edit: &PlaybackProgressEdit,
) -> Result<PlaybackChange>
```

Sets one person's progress in one edition outright, with no
session: a seek from the book's page, Mark finished or Start over.

The row is created when the person has none. Finishing also sets
the work's finished flag when `work_id` is given. One atomic batch;
nothing is written for an unknown edition on an engine that checks
foreign keys, which is then the returned error.

##### Arguments

* `user_id` - The listener.
* `edition_id` - The edition.
* `work_id` - The edition's work, for the finished flag.
* `edit` - What to store.

##### Errors

Returns the database error.

#### LibraryStore::delete_playback_progress

```rust
pub async fn delete_playback_progress(
    &self,
    user_id: i64,
    edition_id: &str,
) -> Result<bool>
```

Forgets one person's progress in one edition. The work's finished
flag is left as it is.

##### Returns

True when a row was removed.

##### Errors

Returns the database error.

#### LibraryStore::open_playback_session

```rust
pub async fn open_playback_session(
    &self,
    new: NewPlaybackSession<'_>,
) -> Result<PlaybackSession>
```

Opens a play session and answers it.

The session starts where the person's progress is, or at the start
when they have none or had finished the edition (playing a finished
edition again starts it over and clears the edition's finished
flag; the work stays finished). One atomic batch.

##### Errors

Returns the database error, for example for an unknown edition on
an engine that checks foreign keys.

#### LibraryStore::playback_session

```rust
pub async fn playback_session(&self, id: &str) -> Result<Option<PlaybackSession>>
```

One play session by id.

##### Errors

Returns the read error.

#### LibraryStore::sync_playback_session

```rust
pub async fn sync_playback_session(
    &self,
    session: &PlaybackSession,
    work_id: Option<&str>,
    sync: &PlaybackSync,
) -> Result<Option<PlaybackChange>>
```

Records a player's report for an open session: the position, the
time listened since the last report and, with `close`, its end.

The person's progress moves with it. When the position reaches the
recording's end ([`reaches_end`](#reaches_end)) the edition is marked finished,
and so is its work when `work_id` is given. One atomic batch, in
which the progress only moves if the session was still open.

##### Arguments

* `session` - The session, as read just before.
* `work_id` - The edition's work, for the finished flag.
* `sync` - The player's report.

##### Returns

`None` when the session had already closed; nothing is written
then.

##### Errors

Returns the database error.

#### LibraryStore::publish_playback_event

```rust
pub async fn publish_playback_event(
    &self,
    event: PlaybackEvent,
    progress: &PlaybackProgress,
    work_id: Option<&str>,
    session_id: Option<&str>,
)
```

Publishes a playback host event, after the write it describes.

Best effort: the listening state is already committed, so a failed
publish is logged rather than undoing it.

Payload: `userId`, `editionId`, `workId` (or null), `sessionId` (or
null when progress was set outright), `positionMs` and `durationMs`
(or null while no player has reported the runtime).

##### Arguments

* `event` - Which event.
* `progress` - The listener's progress after the write.
* `work_id` - The edition's work.
* `session_id` - The play session, when the write came from one.

#### LibraryStore::list_bookmarks

```rust
pub async fn list_bookmarks(
    &self,
    user_id: i64,
    edition_id: &str,
) -> Result<Vec<Bookmark>>
```

One person's bookmarks in one edition, in timeline order.

##### Errors

Returns the read error.

#### LibraryStore::bookmark

```rust
pub async fn bookmark(&self, id: &str) -> Result<Option<Bookmark>>
```

One bookmark by id.

##### Errors

Returns the read error.

#### LibraryStore::add_bookmark

```rust
pub async fn add_bookmark(
    &self,
    user_id: i64,
    edition_id: &str,
    position: PlaybackMs,
    title: &str,
) -> Result<Option<Bookmark>>
```

Adds a bookmark for one person in one edition.

##### Arguments

* `user_id` - The person.
* `edition_id` - The edition.
* `position` - The place to mark.
* `title` - Their note; at most [`MAX_BOOKMARK_TITLE_BYTES`](#max_bookmark_title_bytes) bytes
  are kept.

##### Returns

The bookmark, or `None` when the person already keeps
[`MAX_BOOKMARKS_PER_EDITION`](#max_bookmarks_per_edition) in the edition.

##### Errors

Returns the database error.

#### LibraryStore::edit_bookmark

```rust
pub async fn edit_bookmark(
    &self,
    user_id: i64,
    id: &str,
    position: Option<PlaybackMs>,
    title: Option<&str>,
) -> Result<Option<Bookmark>>
```

Changes one of a person's bookmarks.

##### Arguments

* `user_id` - The person; another person's bookmark is not found.
* `id` - The bookmark.
* `position` - The new place; `None` keeps it.
* `title` - The new note; `None` keeps it.

##### Returns

The bookmark after the change, or `None` when the person has no
such bookmark.

##### Errors

Returns the database error.

#### LibraryStore::delete_bookmark

```rust
pub async fn delete_bookmark(&self, user_id: i64, id: &str) -> Result<bool>
```

Removes one of a person's bookmarks.

##### Returns

True when a bookmark of theirs was removed.

##### Errors

Returns the database error.

#### LibraryStore::insert_player_password

```rust
pub async fn insert_player_password(
    &self,
    user_id: i64,
    label: &str,
    password_hash: &str,
) -> Result<Option<PlayerPassword>>
```

Stores a new player password for `user_id`.

##### Arguments

* `user_id` - The patron.
* `label` - Its name; trimmed, and at most
  [`MAX_PLAYER_PASSWORD_LABEL_BYTES`](#max_player_password_label_bytes) bytes are kept.
* `password_hash` - SHA-256 hex of the generated password.

##### Returns

The password, or `None` when the patron already keeps
[`MAX_PLAYER_PASSWORDS`](#max_player_passwords).

##### Errors

Returns the database error.

#### LibraryStore::list_player_passwords

```rust
pub async fn list_player_passwords(&self, user_id: i64) -> Result<Vec<PlayerPassword>>
```

A patron's player passwords, newest first.

##### Errors

Returns the read error.

#### LibraryStore::delete_player_password

```rust
pub async fn delete_player_password(&self, user_id: i64, id: &str) -> Result<bool>
```

Removes one of a patron's player passwords and ends every card it
signed in.

##### Returns

True when a password of theirs was removed.

##### Errors

Returns the database error.

#### LibraryStore::use_player_password

```rust
pub async fn use_player_password(
    &self,
    password_hash: &str,
) -> Result<Option<PlayerPassword>>
```

The player password a sign-in presents, by digest, and its
patron; marks it used.

##### Returns

The password, or `None` for no such password.

##### Errors

Returns the database error.

#### LibraryStore::mint_player_card

```rust
pub async fn mint_player_card(&self, card: NewPlayerCard<'_>) -> Result<Option<i64>>
```

Makes a player card: its session row, its card row and its first
access and refresh tokens, in one batch.

##### Returns

The card's session id, or `None` when the player password it names
was removed meanwhile.

##### Errors

Returns the database error.

#### LibraryStore::player_card_by_access

```rust
pub async fn player_card_by_access(&self, access_hash: &str) -> Result<Option<PlayerCard>>
```

Resolves a player card's access token to its card and identity,
touching the session at most once a minute as a browser session is.

##### Arguments

* `access_hash` - SHA-256 hex of the access token.

##### Returns

`None` for an unknown or expired token or an ended card.

##### Errors

Returns the database error.

#### LibraryStore::refresh_player_card

```rust
pub async fn refresh_player_card(
    &self,
    refresh_hash: &str,
    tokens: PlayerCardTokens<'_>,
) -> Result<Option<PlayerCard>>
```

Trades a refresh token for a new access and refresh token, in one
batch. The presented token and every other refresh token of the
card keep working for [`PLAYER_REFRESH_GRACE_SECS`](#player_refresh_grace_secs) more, so two
refreshes racing with one token both succeed; the card's thirty
days start again.

##### Arguments

* `refresh_hash` - SHA-256 hex of the presented refresh token.
* `tokens` - Digests of the new pair.

##### Returns

The card, or `None` when the token does not refresh: unknown,
expired, or its card ended.

##### Errors

Returns the database error.

#### LibraryStore::end_player_card

```rust
pub async fn end_player_card(&self, token_hash: &str) -> Result<bool>
```

Ends the card a token (access or refresh) belongs to: the app
signed out.

##### Returns

True when a live card was ended.

##### Errors

Returns the database error.

#### LibraryStore::player_editions_of_works

```rust
pub async fn player_editions_of_works(
    &self,
    work_ids: &[String],
) -> Result<HashMap<String, String>>
```

The edition each work is listed by in the player API: of its audio
editions this library acquired, the one acquired last. A work with
none is left out.

##### Arguments

* `work_ids` - Works (top-level works of the page index).

##### Errors

Returns [`LibraryError::Orm`](#libraryerror) when a read fails.

#### LibraryStore::player_items

```rust
pub async fn player_items(&self, edition_ids: &[String]) -> Result<Vec<PlayerItem>>
```

The items of `edition_ids`, in the same order: audio editions this
library acquired. An unknown, unacquired or non-audio edition is
left out.

##### Errors

Returns [`LibraryError::Orm`](#libraryerror) when a read fails.

#### LibraryStore::player_items_page

```rust
pub async fn player_items_page(
    &self,
    query: &WorkPageQuery,
    offset: u64,
    limit: u64,
    reversed: bool,
) -> Result<(Vec<PlayerItem>, u64)>
```

The items of one page of works ([`Self::library_works_at`](#librarystorelibrary_works_at)), each
work by the edition [`Self::player_editions_of_works`](#librarystoreplayer_editions_of_works) picks, and
how many works the query matches in all. A work with no acquired
audio edition is left out of the page.

##### Arguments

* `query` - Order and filters of the page index.
* `offset` - Works to skip.
* `limit` - Most works to read.
* `reversed` - True to read the order backwards.

##### Errors

Returns [`LibraryError::Orm`](#libraryerror) when a read fails.

#### LibraryStore::list_playback_sessions

```rust
pub async fn list_playback_sessions(
    &self,
    user_id: i64,
    offset: u64,
    limit: u64,
) -> Result<(Vec<PlaybackSession>, u64)>
```

One person's play sessions, newest first: `limit` from `offset`,
and how many they have in all.

##### Errors

Returns [`LibraryError::Orm`](#libraryerror) when a read fails.

#### LibraryStore::list_user_bookmarks

```rust
pub async fn list_user_bookmarks(&self, user_id: i64, cap: u64) -> Result<Vec<Bookmark>>
```

Every bookmark one person made, at most `cap`, oldest first.

##### Errors

Returns [`LibraryError::Orm`](#libraryerror) when a read fails.

#### LibraryStore::record_plugin_database

```rust
pub async fn record_plugin_database(
    &self,
    plugin_id: &str,
    binding: &str,
    adapter_plugin_key: &str,
    backend_kind: &str,
    unit_ref: &str,
) -> Result<PluginDatabaseRecord>
```

Records one provisioned binding unit, keeping any existing row.

Returns the authoritative row (an earlier registration wins so a
re-open never silently re-targets a binding at a different unit).

##### Errors

Returns an error when the insert or lookup fails.

#### LibraryStore::get_plugin_database

```rust
pub async fn get_plugin_database(
    &self,
    plugin_id: &str,
    binding: &str,
) -> Result<Option<PluginDatabaseRecord>>
```

The registry row for `(plugin_id, binding)`, if provisioned.

##### Errors

Returns an error when the query fails.

#### LibraryStore::list_plugin_databases

```rust
pub async fn list_plugin_databases(
    &self,
    plugin_id: Option<&str>,
) -> Result<Vec<PluginDatabaseRecord>>
```

All registry rows, optionally filtered to one owner PluginKey.

##### Errors

Returns an error when the query fails.

#### LibraryStore::remove_plugin_databases

```rust
pub async fn remove_plugin_databases(
    &self,
    plugin_id: &str,
    binding: Option<&str>,
) -> Result<u64>
```

Deletes registry rows for a plugin (one binding, or all of them).

Removes only the registry entry; dropping the underlying unit is the
caller's responsibility (operator CLI / uninstall flow).

##### Errors

Returns an error when the delete fails.

#### LibraryStore::rebind_plugin_database

```rust
pub async fn rebind_plugin_database(
    &self,
    plugin_id: &str,
    binding: &str,
    adapter_plugin_key: &str,
    backend_kind: &str,
    unit_ref: &str,
) -> Result<PluginDatabaseRecord>
```

Inserts or updates placement for one `(plugin_id, binding)` on restore.

Logical identity is portable; `adapter_plugin_key` / `backend_kind` /
`unit_ref` are the **target** adapter's physical placement, never the
source's.

##### Errors

Returns when the upsert fails.

#### LibraryStore::kv_get

```rust
pub async fn kv_get(
    &self,
    plugin_key: &str,
    namespace: &str,
    key: &str,
) -> Result<Option<KvEntry>>
```

Reads one key; `None` when absent or expired.

##### Errors

Returns an error when the query fails.

#### LibraryStore::kv_put

```rust
pub async fn kv_put(
    &self,
    plugin_key: &str,
    namespace: &str,
    key: &str,
    spec: KvWriteSpec,
) -> Result<KvPutOutcome>
```

Writes one key, replacing any previous value, metadata, and expiry.

Enforces `spec.max_value_bytes` and `spec.max_keys` (expired rows do
not count and are reclaimed to make room).

##### Errors

Returns an error when the write fails.

#### LibraryStore::kv_delete

```rust
pub async fn kv_delete(
    &self,
    plugin_key: &str,
    namespace: &str,
    key: &str,
) -> Result<bool>
```

Deletes one key. Returns whether a live value existed.

##### Errors

Returns an error when the delete fails.

#### LibraryStore::kv_list

```rust
pub async fn kv_list(
    &self,
    plugin_key: &str,
    namespace: &str,
    prefix: &str,
    cursor: &str,
    limit: u32,
) -> Result<KvListPage>
```

Lists live keys in lexical order, `limit` per page (0 means
[`KV_LIST_DEFAULT_LIMIT`](#kv_list_default_limit)). `cursor` is the opaque value a previous
page returned; empty starts from the beginning.

##### Errors

Returns an error when the query fails or the cursor is malformed.

#### LibraryStore::kv_count

```rust
pub async fn kv_count(&self, plugin_key: &str, namespace: &str) -> Result<u64>
```

Live keys in one namespace.

##### Errors

Returns an error when the query fails.

#### LibraryStore::kv_prune_expired_in

```rust
pub async fn kv_prune_expired_in(&self, plugin_key: &str, namespace: &str) -> Result<u64>
```

Removes expired rows of one namespace. Returns the rows removed.

##### Errors

Returns an error when the delete fails.

#### LibraryStore::kv_prune_expired

```rust
pub async fn kv_prune_expired(&self) -> Result<u64>
```

Removes every expired row across all plugins and namespaces.

##### Errors

Returns an error when the delete fails.

#### LibraryStore::kv_delete_plugin

```rust
pub async fn kv_delete_plugin(&self, plugin_key: &str) -> Result<u64>
```

Removes every row of one plugin (retire / uninstall).

##### Errors

Returns an error when the delete fails.

#### LibraryStore::replace_request_queue_ranks

```rust
pub async fn replace_request_queue_ranks(&self, ranks: &[QueueRank]) -> Result<()>
```

Replaces the stored queue ranking with `ranks`.

Each row is upserted with this pass's time, then every row this pass
did not write (an entry whose wishes were all closed) is deleted. The
writes go to the database plugin as canonical atomic batches sized
from its advertised limits, never an interactive transaction; the
delete rides in the last batch. A reader between two batches can see
part of the new ranking next to old rows, which only shifts order.

##### Arguments

* `ranks` - One ranking per queue entry; a repeated `work_id` keeps the last.

##### Errors

Returns the database error. Batches written before it stay; old rows
are deleted only once every new row is stored.

#### LibraryStore::list_request_queue_ranks

```rust
pub async fn list_request_queue_ranks(&self) -> Result<Vec<StoredQueueRank>>
```

Lists every stored queue ranking row.

Rows are read in work id pages of a bounded size; a page whose result
would exceed the plugin's `maxResultBytes` is halved and read again.
A row whose reasons do not parse keeps its score with no reasons.

##### Errors

Returns the database error.

#### LibraryStore::prune_record_step

```rust
pub async fn prune_record_step(
    &self,
    step: RecordPruneStep,
    now: DateTime<Utc>,
) -> Result<u64>
```

Runs one retention delete as of `now` and answers how many rows
(for [`RecordPruneStep::StorageScans`](#recordprunestep), generations) it removed.

##### Arguments

* `step` - Which table and rule to apply.
* `now` - The instant the windows are measured back from; the
  maintenance duty passes the current time, tests a fixed one.

##### Errors

Returns [`LibraryError::Orm`](#libraryerror) when a read or delete fails; rows the
step had not reached yet are left for the next pass.

#### LibraryStore::quality_check_state

```rust
pub async fn quality_check_state(
    &self,
    plugin_key: &str,
    account_id: &str,
) -> Result<Option<QualityCheckState>>
```

Reads where an account's quality-upgrade check stands.

##### Returns

`None` when the account has no scan state yet (it was never
scanned, so it holds nothing to check).

##### Errors

Returns an error when the read fails.

#### LibraryStore::save_quality_check

```rust
pub async fn save_quality_check(
    &self,
    plugin_key: &str,
    account_id: &str,
    cursor: &str,
    at: DateTime<Utc>,
) -> Result<()>
```

Records that a quality-upgrade check of an account ran at `at` and
continues after holding `cursor` next time (empty starts over).

Writes only an existing row of an account that is not revoked; the
scan creates the row.

##### Errors

Returns an error when the batch fails.

#### LibraryStore::scan_state

```rust
pub async fn scan_state(
    &self,
    plugin_key: &str,
    account_id: &str,
) -> Result<Option<ScanState>>
```

Reads where an account's scan stands.

##### Errors

Returns an error when the read fails.

#### LibraryStore::save_scan_checkpoint

```rust
pub async fn save_scan_checkpoint(
    &self,
    plugin_key: &str,
    account_id: &str,
    cursor: &str,
    pass_token: &str,
    pass_id: &str,
) -> Result<()>
```

Records that a pass wrote a page and continues at `cursor`.

##### Arguments

* `plugin_key` - The account's plugin.
* `account_id` - The account.
* `cursor` - The next page's cursor.
* `pass_token` - The sync token the pass started with.
* `pass_id` - The pass's id ([`PassStart::pass_id`](#passstart)), kept so a
  resumed pass stamps the same one.

##### Errors

Returns an error when the batch fails.

#### LibraryStore::finish_scan_pass

```rust
pub async fn finish_scan_pass(
    &self,
    plugin_key: &str,
    account_id: &str,
    sync_token: &str,
    complete: bool,
) -> Result<()>
```

Records the end of a pass: stores the plugin's next sync token, drops
the resume cursor, and stamps `last_complete_at` for a complete pass.

##### Errors

Returns an error when the batch fails.

#### LibraryStore::reset_scan_state

```rust
pub async fn reset_scan_state(&self, plugin_key: &str, account_id: &str) -> Result<()>
```

Forgets an account's sync token and resume cursor (the plugin called
one invalid), so its next pass is full and starts at the beginning.

##### Errors

Returns an error when the batch fails.

#### LibraryStore::storage_scan_put_object

```rust
pub async fn storage_scan_put_object(
    &self,
    scan_id: &str,
    key: &str,
    size: u64,
    media_rank: u8,
    is_audio: bool,
) -> Result<()>
```

Inserts an object row. A repeat of the same key is a no-op.

##### Errors

Returns [`LibraryError::Orm`](#libraryerror) when the write fails.

#### LibraryStore::storage_scan_put_identity

```rust
pub async fn storage_scan_put_identity(
    &self,
    scan_id: &str,
    identity: &str,
    key: &str,
    size: u64,
    media_rank: u8,
) -> Result<()>
```

Inserts an identity → key row. Repeats are no-ops.

##### Errors

Returns [`LibraryError::Orm`](#libraryerror) when the write fails.

#### LibraryStore::storage_scan_identity_page

```rust
pub async fn storage_scan_identity_page(
    &self,
    scan_id: &str,
    identity: &str,
    after: Option<(i64, &str)>,
    limit: u64,
) -> Result<Vec<(String, i64)>>
```

One page of identity candidates, ordered by `(media_rank, object_key)`.

`after` is the last key already considered. The page is bounded by
`limit` (clamped to 1..=64). Callers try each key and request the next
page; this does not load every match.

##### Errors

Returns [`LibraryError::Orm`](#libraryerror) when the read fails.

#### LibraryStore::storage_scan_object_page

```rust
pub async fn storage_scan_object_page(
    &self,
    scan_id: &str,
    prefix: &str,
    after_key: Option<&str>,
    limit: u64,
) -> Result<Vec<String>>
```

One page of object keys at or above `prefix`, in `object_key` order.

The upper bound is the byte-wise successor of `prefix`, not a SQL `LIKE`
pattern, so wildcard characters in a storage key stay literal.

##### Errors

Returns [`LibraryError::Orm`](#libraryerror) when the read fails.

#### LibraryStore::storage_scan_claim

```rust
pub async fn storage_scan_claim(&self, scan_id: &str, key: &str) -> Result<()>
```

Marks an object claimed so unmatched-audio counts stay in the database.

##### Errors

Returns [`LibraryError::Orm`](#libraryerror) when the update fails.

#### LibraryStore::storage_scan_unclaimed_audio

```rust
pub async fn storage_scan_unclaimed_audio(&self, scan_id: &str) -> Result<u64>
```

Audio objects in the scan that no library row claimed.

##### Errors

Returns [`LibraryError::Orm`](#libraryerror) when the count fails.

#### LibraryStore::storage_scan_delete

```rust
pub async fn storage_scan_delete(&self, scan_id: &str) -> Result<()>
```

Deletes inventory rows and the generation ownership row for `scan_id`.

Both deletes go to the database plugin as one atomic batch, so a
failure leaves the generation and its rows in place and an apply
checkpoint cannot adopt an empty inventory.

##### Errors

Returns the batch error when the delete fails.

#### LibraryStore::storage_scan_register

```rust
pub async fn storage_scan_register(
    &self,
    scan_id: &str,
    instance_id: &str,
    job_id: Option<&str>,
) -> Result<()>
```

Binds `scan_id` to `instance_id` and an optional job.

A second register for the same id and instance refreshes the heartbeat.
A different instance fails closed so a reused id cannot adopt another scan.

##### Errors

Returns [`LibraryError::Orm`](#libraryerror) when the write fails, or
[`LibraryError::Other`](#libraryerror) when `instance_id` does not match the existing row.

#### LibraryStore::storage_scan_mark_complete

```rust
pub async fn storage_scan_mark_complete(&self, scan_id: &str) -> Result<()>
```

Marks a generation's inventory complete so a later checkpoint can adopt it.

##### Errors

Returns [`LibraryError::Orm`](#libraryerror) when the update fails.

#### LibraryStore::storage_scan_adoption

```rust
pub async fn storage_scan_adoption(&self, scan_id: &str) -> Result<Option<(String, bool)>>
```

Returns `(instance_id, completed)` when the generation row still exists.

##### Errors

Returns [`LibraryError::Orm`](#libraryerror) when the read fails.

#### LibraryStore::storage_scan_touch

```rust
pub async fn storage_scan_touch(&self, scan_id: &str) -> Result<()>
```

Refreshes the generation heartbeat after a page is stored.

##### Errors

Returns [`LibraryError::Orm`](#libraryerror) when the update fails.

#### LibraryStore::storage_scan_generation_count

```rust
pub async fn storage_scan_generation_count(&self) -> Result<u64>
```

How many generation rows exist. Tests use this to bound retained scans.

##### Errors

Returns [`LibraryError::Orm`](#libraryerror) when the count fails.

#### LibraryStore::storage_scan_reclaim_abandoned

```rust
pub async fn storage_scan_reclaim_abandoned(&self, stale_before: &str) -> Result<u64>
```

Deletes generations that are not live resumable work.

A generation with an active job is kept even when `updated_at` is older
than `stale_before`. A jobless generation is kept only while its
heartbeat is at least `stale_before`. Terminal, missing, and stale
jobless generations are removed with their inventory rows.

##### Errors

Returns [`LibraryError::Orm`](#libraryerror) when a read or delete fails.

#### LibraryStore::storage_scan_reclaim_if_terminal

```rust
pub async fn storage_scan_reclaim_if_terminal(&self, job_id: &str) -> Result<u64>
```

Deletes this job's inventories when the job is terminal or already gone.

Pending and running jobs, including a retry after one failed attempt,
are left untouched.

##### Errors

Returns [`LibraryError::Orm`](#libraryerror) when a read or delete fails.

#### LibraryStore::storage_scan_reclaim_terminal_page

```rust
pub async fn storage_scan_reclaim_terminal_page(
    &self,
    after_scan_id: Option<&str>,
    limit: u64,
) -> Result<(u64, Option<String>)>
```

Deletes up to `limit` generations whose job is terminal or missing.

Pending, running, and jobless generations are kept. `after_scan_id`
walks past a page of still-active rows so one sweep cannot stall on
them. Pass the returned cursor back; `None` means the walk wrapped.

##### Errors

Returns [`LibraryError::Orm`](#libraryerror) when a read or delete fails. A delete
error leaves that generation for a later retry.

#### LibraryStore::taste_titles

```rust
pub async fn taste_titles(&self) -> Result<Vec<TasteTitle>>
```

Every work the library holds, one [`TasteTitle`](#tastetitle) each, in work id
order, with no person's state applied.

A work is held when an account or a branch holds a listing of one
of its editions. [`TasteTitle::holders`](#tastetitle) lists the accounts that
hold it (a branch-held work has none).

##### Errors

Returns [`LibraryError::Orm`](#libraryerror) when a read fails.

#### LibraryStore::user_title_states_for_works

```rust
pub async fn user_title_states_for_works(
    &self,
    work_ids: &[String],
) -> Result<Vec<UserTitleState>>
```

The stored state of every user for the given works, read in chunks.

##### Arguments

* `work_ids` - Works whose state to read.

##### Returns

The stored rows, ordered by work then user.

##### Errors

Returns [`LibraryError::Orm`](#libraryerror) when a read fails.

#### LibraryStore::held_works

```rust
pub async fn held_works(&self, work_ids: &[String]) -> Result<BTreeSet<String>>
```

The works among `work_ids` the library holds: an account or a branch
holds a listing of one of their editions.

##### Arguments

* `work_ids` - Works to check.

##### Returns

The held ones.

##### Errors

Returns [`LibraryError::Orm`](#libraryerror) when a read fails.

#### LibraryStore::held_identifiers

```rust
pub async fn held_identifiers(
    &self,
    asins: &[String],
    isbns: &[String],
    product_ids: &[String],
) -> Result<HeldIdentifiers>
```

Which of `asins`, `isbns` and `product_ids` belong to a title the
library holds, so a ranking can drop candidates the library already
has without reading the whole library.

An ASIN or ISBN is held when it identifies an edition one of whose
listings an account or a branch holds; a product id is held when a
held listing carries it, on any store.

##### Arguments

* `asins` - ASINs to check, in any case.
* `isbns` - ISBNs to check, as stored (canonical ISBN-13).
* `product_ids` - Store product ids to check.

##### Returns

The held ones; ASINs upper-cased.

##### Errors

Returns [`LibraryError::Orm`](#libraryerror) when a read fails.

#### LibraryStore::set_user_title_state

```rust
pub async fn set_user_title_state(
    &self,
    user_id: i64,
    work_id: &str,
    fields: &UserBookFields,
) -> Result<()>
```

Records `fields` as `user_id`'s state for `work_id`.

Fields left `None` keep their stored value (a new row starts
unfinished, unrated and untagged). New tags also replace the work's
`user_work_tags` rows, which the library page filters by. The write
is one atomic batch.

##### Arguments

* `user_id` - The user (`users.id`).
* `work_id` - The work (`works.id`).
* `fields` - The fields to set.

##### Errors

Returns the database error, for example when the user or the work
does not exist on an engine that checks foreign keys.

#### LibraryStore::edit_user_title_state

```rust
pub async fn edit_user_title_state(
    &self,
    user_id: i64,
    work_id: &str,
    edit: &UserStateEdit,
) -> Result<Option<UserTitleState>>
```

Applies `edit` to `user_id`'s state for `work_id` and returns the
state as stored afterwards.

Unlike [`LibraryStore::set_user_title_state`](#librarystoreset_user_title_state), a field can be
cleared as well as kept or set. New tags also replace the work's
`user_work_tags` rows. The write is one atomic batch, and every
statement in it requires the work to exist, so an unknown work
writes nothing.

##### Arguments

* `user_id` - The user (`users.id`).
* `work_id` - The work (`works.id`).
* `edit` - What to change.

##### Returns

The stored state, or `None` when no such work exists.

##### Errors

Returns the database error.

#### LibraryStore::user_title_states

```rust
pub async fn user_title_states(&self, user_ids: &[i64]) -> Result<Vec<UserTitleState>>
```

Every work state of the given users, read in chunks.

##### Arguments

* `user_ids` - Users whose state to read.

##### Returns

The stored rows, ordered by user then work.

##### Errors

Returns [`LibraryError::Orm`](#libraryerror) when a read fails.

#### LibraryStore::user_title_states_of_works

```rust
pub async fn user_title_states_of_works(
    &self,
    user_id: i64,
    work_ids: &[String],
) -> Result<Vec<UserTitleState>>
```

One user's stored state for each of `work_ids`, read in chunks
that fit the database's limits.

##### Arguments

* `user_id` - The user (`users.id`).
* `work_ids` - Works whose state to read; duplicates are read once.

##### Returns

The stored rows, one per work the user has state for, ordered by
work.

##### Errors

Returns [`LibraryError::Orm`](#libraryerror) when a read fails.

#### LibraryStore::users_linked_to_account

```rust
pub async fn users_linked_to_account(
    &self,
    plugin_key: &str,
    account_id: &str,
) -> Result<Vec<i64>>
```

The users linked to a storefront account: the users of the portal
identities whose account links name it.

##### Arguments

* `plugin_key` - The account's plugin.
* `account_id` - The account id at that plugin.

##### Returns

Distinct user ids, ascending; empty when no linked identity has a
user.

##### Errors

Returns [`LibraryError::Orm`](#libraryerror) when a read fails.

#### LibraryStore::put_vectors

```rust
pub async fn put_vectors(
    &self,
    target_kind: &str,
    space: &VectorSpace,
    dims: i64,
    vectors: &[NewVector<'_>],
) -> Result<()>
```

Stores vectors of one kind in one space, replacing the vector a
target already has there.

##### Arguments

* `target_kind` - [`VECTOR_KIND_WORK`](#vector_kind_work) or [`VECTOR_KIND_TEXT`](#vector_kind_text).
* `space` - The space the vectors belong to.
* `dims` - Numbers in each vector.
* `vectors` - The vectors, one statement each.

##### Errors

Returns [`LibraryError::Orm`](#libraryerror) when a write fails; the vectors before
it are stored.

#### LibraryStore::vector_text_hashes

```rust
pub async fn vector_text_hashes(
    &self,
    target_kind: &str,
    space: &VectorSpace,
    target_ids: &[String],
) -> Result<HashMap<String, String>>
```

Stored text hashes of targets in one space, keyed by target id.
Targets with no vector there are absent.

Reads in chunks sized from the adapter's limits, so one call
replaces a lookup per target.

##### Errors

Returns [`LibraryError::Orm`](#libraryerror) when a read fails.

#### LibraryStore::vectors

```rust
pub async fn vectors(
    &self,
    target_kind: &str,
    space: &VectorSpace,
    target_ids: &[String],
) -> Result<HashMap<String, Vec<u8>>>
```

Stored vectors of targets in one space, keyed by target id, as
little-endian `f32` bytes. Targets with no vector there are absent.

Reads in chunks that narrow until each reply fits the adapter's
limits, so a space of long vectors is still read.

##### Errors

Returns [`LibraryError::Orm`](#libraryerror) when a read fails.

#### LibraryStore::work_vector_spaces

```rust
pub async fn work_vector_spaces(&self) -> Result<Vec<SpaceCoverage>>
```

The spaces that hold work vectors, with how many each holds.

##### Errors

Returns [`LibraryError::Orm`](#libraryerror) when the read fails.

#### LibraryStore::count_works

```rust
pub async fn count_works(&self) -> Result<u64>
```

How many works the library has.

##### Errors

Returns [`LibraryError::Orm`](#libraryerror) when the read fails.

#### LibraryStore::prune_orphan_work_vectors

```rust
pub async fn prune_orphan_work_vectors(&self) -> Result<u64>
```

Deletes work vectors (every space) whose work no longer exists, and
answers how many it removed.

##### Errors

Returns [`LibraryError::Orm`](#libraryerror) when the delete fails.

#### LibraryStore::prune_vectors_outside

```rust
pub async fn prune_vectors_outside(
    &self,
    keep: &VectorSpace,
    before: DateTime<Utc>,
) -> Result<u64>
```

Deletes every vector, of any kind, outside `keep` that was last
written or used before `before`, and answers how many it removed.

Called once `keep` holds a vector for every work, so the vectors of
a model the library moved away from do not stay. The age limit
leaves alone a space another node is still writing.

##### Errors

Returns [`LibraryError::Orm`](#libraryerror) when the delete fails.

#### LibraryStore::prune_text_vectors

```rust
pub async fn prune_text_vectors(&self, before: DateTime<Utc>) -> Result<u64>
```

Deletes the text vectors (every space) last used before `before`,
and answers how many it removed.

##### Errors

Returns [`LibraryError::Orm`](#libraryerror) when the delete fails.

#### LibraryStore::touch_text_vectors

```rust
pub async fn touch_text_vectors(
    &self,
    space: &VectorSpace,
    target_ids: &[String],
    stale: DateTime<Utc>,
) -> Result<()>
```

Marks text vectors of one space as used now, when their last use is
older than `stale`, so [`Self::prune_text_vectors`](#librarystoreprune_text_vectors) keeps what a
ranking still reads without a write per read.

##### Errors

Returns [`LibraryError::Orm`](#libraryerror) when a write fails.

#### LibraryStore::library_work_detail

```rust
pub async fn library_work_detail(&self, work_id: &str) -> Result<Option<WorkDetail>>
```

One top-level work with its editions, listings and holdings.

##### Arguments

* `work_id` - The work. A podcast episode's id reads as missing:
  its show is the work the library lists.

##### Returns

`None` when there is no such top-level work.

##### Errors

Returns [`LibraryError::Orm`](#libraryerror) when a read fails.

#### LibraryStore::edition_cover_urls

```rust
pub async fn edition_cover_urls(
    &self,
    edition_ids: &[String],
) -> Result<HashMap<String, String>>
```

The cover URLs of `edition_ids`, for a page of index rows (each
names its cover edition), chosen as `edition_cover` says: the
edition's own, else a listing's, else its work's (or show's).

##### Returns

Edition id to cover URL; editions with no cover are left out.

##### Errors

Returns [`LibraryError::Orm`](#libraryerror) when a read fails.

#### LibraryStore::user_title_state

```rust
pub async fn user_title_state(
    &self,
    user_id: i64,
    work_id: &str,
) -> Result<Option<UserTitleState>>
```

One user's own state for one work.

##### Arguments

* `user_id` - The user (`users.id`).
* `work_id` - The work (`works.id`).

##### Returns

`None` when the user recorded nothing for the work.

##### Errors

Returns [`LibraryError::Orm`](#libraryerror) when the read fails.

#### LibraryStore::work_search_document

```rust
pub async fn work_search_document(
    &self,
    work_id: &str,
) -> Result<(String, Option<WorkSearchDoc>)>
```

The search document of `work_id`'s top-level work.

##### Arguments

* `work_id` - A work; an episode stands for its show.

##### Returns

The top-level work's id and its document, or `None` for the document
when the work is gone or no longer held (its document is then
removed from the index).

##### Errors

Returns [`crate::LibraryError::Orm`](#libraryerror) when a read fails.

#### LibraryStore::work_search_documents

```rust
pub async fn work_search_documents(
    &self,
    work_ids: &[String],
) -> Result<Vec<(String, Option<WorkSearchDoc>)>>
```

The search documents of the top-level works of `work_ids`, read
together in a few chunked lookups.

##### Arguments

* `work_ids` - Works; an episode stands for its show.

##### Returns

One entry per distinct top-level work, in id order: its id and its
document, or `None` for the document when the work is gone or no
longer held (its document is then removed from the index).

##### Errors

Returns [`crate::LibraryError::Orm`](#libraryerror) when a read fails.

#### LibraryStore::split_work

```rust
pub async fn split_work(
    &self,
    work_id: &str,
    expression_ids: &[String],
    user_id: Option<i64>,
) -> Result<SplitOutcome>
```

Moves `expression_ids` out of `work_id` into a new work, as one
atomic batch.

Every write requires the new work, and the new work is inserted only
while every named expression still belongs to `work_id` and another
stays behind, so a concurrent change writes nothing rather than half
a split.

##### Arguments

* `work_id` - The work to split (`works.id`).
* `expression_ids` - The expressions to move, 1 to
  [`MAX_SPLIT_EXPRESSIONS`](#max_split_expressions); duplicates count once.
* `user_id` - The Administrator (`users.id`); `None` for the operator.

##### Returns

What the split did. [`SplitOutcome::Split`](#splitoutcome) is returned only when
the expressions were moved.

##### Errors

Returns the database error of a failed read or write, or
[`LibraryError::Conflict`](#libraryerror) when the work changed between the read
and the write.

#### LibraryStore::work_tags

```rust
pub async fn work_tags(&self, work_id: &str) -> Result<Vec<WorkTag>>
```

A work's library tags, every source, in key order (a key two sources
share is listed once per source).

##### Errors

Returns [`LibraryError::NotFound`](#libraryerror) for a missing work or an episode,
or the read error.

#### LibraryStore::set_work_tags

```rust
pub async fn set_work_tags(
    &self,
    work_id: &str,
    values: &[String],
) -> Result<Vec<WorkTag>>
```

Replaces the tags an Administrator set on a work (sidecar tags stay)
and refreshes the work's page index row, so the Library filter and
patron scopes see the change at once.

The replacement is one atomic batch. The caller checks who may set
tags.

##### Arguments

* `work_id` - A top-level work (`works.id`).
* `values` - The tags; blanks and repeats (ignoring case) are
  dropped.

##### Returns

The work's tags afterwards, every source.

##### Errors

Returns [`LibraryError::NotFound`](#libraryerror) for a missing work or an episode,
[`LibraryError::Invalid`](#libraryerror) for too many or too long tags, or the
read or write error.

#### LibraryStore::enqueue_job

```rust
pub async fn enqueue_job(&self, spec: EnqueueJobSpec) -> Result<EnqueueOutcome>
```

Admit a job in one transaction, coalescing onto an active dedup key.

##### Arguments

* `spec` - Kind, payload, priority, attempt cap, and pending cap.

##### Returns

[`EnqueueOutcome::Created`](#enqueueoutcome), [`EnqueueOutcome::Duplicate`](#enqueueoutcome), or
[`EnqueueOutcome::QueueFull`](#enqueueoutcome).

##### Errors

Returns [`LibraryError::Orm`](#libraryerror) when the database write fails.

#### LibraryStore::claim_next_job

```rust
pub async fn claim_next_job(
    &self,
    resource_class: JobResourceClass,
    owner: &str,
    lease_secs: u64,
    operation_id: &str,
) -> Result<Option<JobRecord>>
```

Claim the next ready job with a conditional `pending` → `running` mutation.

`operation_id` is the atomic / local-txn idempotency key: retrying a
lost response with the same id must not claim a different row.

##### Errors

Returns [`LibraryError::Orm`](#libraryerror) when the claim write fails.

#### LibraryStore::list_job_claim_candidates

```rust
pub async fn list_job_claim_candidates(
    &self,
    resource_class: JobResourceClass,
    after: Option<&JobClaimCursor>,
    limit: u64,
) -> Result<JobClaimPage>
```

One page of ready pending jobs of `resource_class`, best first, for a
routed claim.

Pages are keyset-ordered by `priority DESC, created_at ASC, id ASC`:
pass `None` for the first page and the previous page's
[`JobClaimPage::next`](#jobclaimpage) for the following one, until it is `None`. A
caller that skips every job of a page (for example jobs another node
must run) can therefore still reach the runnable rows behind them.

Each returned row is readable and not cancel-requested. On the way,
pending rows with an unknown class or an unreadable command become
`invalid_job`, and cancel-requested pending rows become `cancelled`,
exactly as [`Self::claim_next_job`](#librarystoreclaim_next_job) does. Every write is a CAS on
`state = 'pending'`, so a concurrent claimer is never overwritten.

##### Errors

Returns [`LibraryError::Orm`](#libraryerror) when a read or write fails.

#### LibraryStore::claim_job_by_id

```rust
pub async fn claim_job_by_id(
    &self,
    job_id: &str,
    expected_generation: i64,
    owner: &str,
    lease_secs: u64,
) -> Result<Option<JobRecord>>
```

Claims the pending job `job_id` for `owner` with a single CAS, but
only while its lease generation is still `expected_generation`.

The update matches `state = 'pending'` and `expected_generation`, the
lease generation of the record the caller routed on, so of several
concurrent claimers on any number of connections exactly one moves the
row to `running` (and bumps the generation that fences every later
write). A successful claim therefore always runs at
`expected_generation + 1`, which lets the caller name the claim's
generation (in its permit holders, say) before claiming. A row whose
payload was rewritten after the caller read it
([`Self::rewrite_pending_job_payload`](#librarystorerewrite_pending_job_payload) bumps the generation) is not
claimed, so a job never runs on requirements its claimer did not
check. No transaction is needed: a pending → running move does not
change the admission count.

##### Arguments

* `job_id` - The job to claim.
* `expected_generation` - `lease_generation` of the record the caller
  evaluated; a successful claim runs at `expected_generation + 1`.
* `owner` - Worker lease owner written on the row.
* `lease_secs` - Lease length.

##### Returns

The claimed record, or `None` when the row is missing, not pending,
not yet due, cancel-requested, unreadable, moved past
`expected_generation`, or another claimer won.

##### Errors

Returns [`LibraryError::Orm`](#libraryerror) when a read or write fails.

#### LibraryStore::list_pending_jobs_page

```rust
pub async fn list_pending_jobs_page(
    &self,
    after_id: Option<&str>,
    limit: u64,
) -> Result<(Vec<JobRecord>, Option<String>)>
```

One page of pending jobs in id order, for maintenance passes.

Pass `None` for the first page and the returned cursor for the next,
until it is `None`. Unlike the claim pages this ignores `run_after`
and the resource class; unreadable rows are skipped (the claim paths
mark them `invalid_job`).

##### Errors

Returns [`LibraryError::Orm`](#libraryerror) when the read fails.

#### LibraryStore::wake_paced_acquires

```rust
pub async fn wake_paced_acquires(&self, account_id: &str) -> Result<u32>
```

Makes parked paced acquires that may download on an account ready
to run now, after its pace was relaxed or its pause cleared.

A paced acquire parked by the pace ([`Self::defer_job`](#librarystoredefer_job)) waits until
the moment its account was due to download again. When a person or
the operator loosens the limit, the job must look again rather than
sleep out the old deadline. Every pending paced acquire that names
`account_id`, or names no account, and whose `run_after` is later
than now is moved to now; on its next claim it checks the pace and
parks again if it still has to wait. Each write is a compare-and-swap
on `state = 'pending'` and the generation it read, which it bumps, so
a job claimed meanwhile is left alone.

##### Returns

How many jobs were woken.

##### Errors

Returns [`LibraryError::Orm`](#libraryerror) when a read or write fails.

#### LibraryStore::rewrite_pending_job_payload

```rust
pub async fn rewrite_pending_job_payload(
    &self,
    job_id: &str,
    lease_generation: i64,
    payload: &JobPayload,
) -> Result<bool>
```

Replaces the payload of a pending job read at `lease_generation`.

A compare-and-swap on `state = 'pending'` and the generation, which it
also bumps: a claimer that read the row before this write loses its
own CAS (and re-reads), so a concurrent claim is never overwritten and
never runs on the replaced payload.

##### Returns

`true` when this write won; `false` when the row moved on.

##### Errors

Returns [`LibraryError::Orm`](#libraryerror) when the write fails, or
[`LibraryError::Other`](#libraryerror) when the payload cannot be serialized.

#### LibraryStore::fail_pending_job

```rust
pub async fn fail_pending_job(
    &self,
    job_id: &str,
    lease_generation: i64,
    error_kind: &str,
    error_message: &str,
) -> Result<bool>
```

Fails a pending job read at `lease_generation`, terminally.

A compare-and-swap on `state = 'pending'` and the generation, so a job
a worker claimed meanwhile is left alone. The row leaves the pending
count and frees its dedup key.

##### Returns

`true` when this write won; `false` when the row moved on.

##### Errors

Returns [`LibraryError::Orm`](#libraryerror) when the write fails.

#### LibraryStore::release_job_claim

```rust
pub async fn release_job_claim(&self, fence: &JobFence, keep_spent: bool) -> Result<bool>
```

Returns a job claimed under `fence` to `pending` without spending an
attempt, as if the claim had not happened.

Used when the claim is refused at handoff (the node is no longer
trusted with the job), and when a branch gives a claim back unrun.
Whether the claim resumed a suspended job is read from the row
([`JobPayload::resumed_generation`](#jobpayload), recorded by the claim), never
taken from the claimant: a resumed claim did not raise the attempt
count, so the one-shot resume flag is set again; otherwise the count
the claim added is taken back. `keep_spent` keeps the attempt spent
and the job resumable even for a claim that was not a resume (a
checkpointed job given back resumes from its checkpoint). The lease
generation stays bumped, so the released claimer's fence can never
write to the row again.

##### Returns

`true` when the row was released; `false` when `fence` no longer owns
the running generation.

##### Errors

Returns [`LibraryError::Orm`](#libraryerror) when a read or write fails, or
[`LibraryError::Other`](#libraryerror) when the stored payload is not valid JSON.

#### LibraryStore::heartbeat_job

```rust
pub async fn heartbeat_job(
    &self,
    fence: &JobFence,
    lease_secs: u64,
    progress: Option<&str>,
) -> Result<bool>
```

Refresh the lease when `fence` still owns the running generation.

##### Errors

Returns [`LibraryError::Orm`](#libraryerror) when the update fails.

#### LibraryStore::set_job_progress

```rust
pub async fn set_job_progress(&self, fence: &JobFence, progress: &str) -> Result<bool>
```

Persist progress when `fence` still owns the running generation.

##### Errors

Returns [`LibraryError::Orm`](#libraryerror) when the update fails.

#### LibraryStore::complete_job

```rust
pub async fn complete_job(&self, fence: &JobFence, progress: Option<&str>) -> Result<bool>
```

Mark a running job succeeded when `fence` still owns the generation.

##### Errors

Returns [`LibraryError::Orm`](#libraryerror) when the update fails.

#### LibraryStore::suspend_job

```rust
pub async fn suspend_job(
    &self,
    fence: &JobFence,
    checkpoint: &JobCheckpoint,
    wake_at: DateTime<Utc>,
) -> Result<bool>
```

Park a running job as `pending` with a checkpoint, to be claimed again after `wake_at`.

The CAS matches [`Self::complete_job`]: `state=running` and the caller's fence. On success the
lease is cleared so another replica can claim the next attempt. `payload.checkpoint` is
replaced with `checkpoint` (other payload fields are preserved).

##### Errors

Returns [`LibraryError::Orm`](#libraryerror) when the read or update fails, or [`LibraryError::Other`](#libraryerror)
when the stored payload is not valid JSON.

#### LibraryStore::defer_job

```rust
pub async fn defer_job(
    &self,
    fence: &JobFence,
    wake_at: DateTime<Utc>,
    progress: &str,
) -> Result<bool>
```

Parks a running job as `pending` until `wake_at` without spending an
attempt, with `progress` saying why it waits.

For a job that cannot go on yet and has nothing to resume from, such
as an acquire waiting for its store account's download pace
(`crate::pacing`). Unlike [`Self::suspend_job`](#librarystoresuspend_job) it drops any stored
checkpoint, so the next claim starts fresh. The CAS matches
[`Self::complete_job`]: `state=running` and the caller's fence.

##### Returns

`true` when the row was parked; `false` when `fence` no longer owns
the running generation.

##### Errors

Returns [`LibraryError::Orm`](#libraryerror) when the read or update fails, or
[`LibraryError::Other`](#libraryerror) when the stored payload is not valid JSON.

#### LibraryStore::checkpoint_running_job

```rust
pub async fn checkpoint_running_job(
    &self,
    fence: &JobFence,
    checkpoint: &JobCheckpoint,
) -> Result<bool>
```

Replaces `payload.checkpoint` while `fence` still owns the running job.

Unlike [`Self::suspend_job`](#librarystoresuspend_job), the row stays `running` and `resume_pending`
is left unchanged. A stale fence updates zero rows. Reclaim keeps the
payload, so the next claim observes the checkpoint.

##### Errors

Returns [`LibraryError::Orm`](#libraryerror) on database failure, or [`LibraryError::Other`](#libraryerror)
when the checkpoint exceeds [`bookclerk_plugin_abi::MAX_CHECKPOINT_BYTES`](../bookclerk_plugin_abi/README.md#max_checkpoint_bytes).

#### LibraryStore::fail_job

```rust
pub async fn fail_job(
    &self,
    fence: &JobFence,
    error_kind: &str,
    error_message: &str,
) -> Result<bool>
```

Fail a running job when `fence` still owns the generation.

##### Errors

Returns [`LibraryError::Orm`](#libraryerror) when the update fails.

#### LibraryStore::fail_job_without_retry

```rust
pub async fn fail_job_without_retry(
    &self,
    fence: &JobFence,
    error_kind: &str,
    error_message: &str,
) -> Result<bool>
```

Fail a running job for good when `fence` still owns the generation:
like [`Self::fail_job`](#librarystorefail_job), but the job does not go back to `pending`
whatever attempts it has left. For an outcome another try cannot
change (an interlibrary request its branch no longer lists).

##### Errors

Returns [`LibraryError::Orm`](#libraryerror) when the update fails.

#### LibraryStore::request_job_cancel

```rust
pub async fn request_job_cancel(&self, id: &str) -> Result<Option<JobRecord>>
```

Cancel a pending job immediately, or flag a running job for cooperative stop.

Pending and running are handled atomically against claim: a CAS miss
(the worker claimed, or the job left `running`) retries until the row
is terminal, cancelled, or flagged `cancel_requested`.

##### Returns

The updated record, or `None` when the id is unknown.

##### Errors

Returns [`LibraryError::Orm`](#libraryerror) when the update fails.

#### LibraryStore::job_cancel_requested

```rust
pub async fn job_cancel_requested(&self, id: &str) -> Result<bool>
```

True when a running worker should abort after the current step.

##### Errors

Returns [`LibraryError::Orm`](#libraryerror) when the read fails.

#### LibraryStore::reclaim_expired_leases

```rust
pub async fn reclaim_expired_leases(&self) -> Result<u32>
```

Reclaim expired running leases with a conditional `running` update.

##### Returns

Number of rows moved out of `running`.

##### Errors

Returns [`LibraryError::Orm`](#libraryerror) when the update fails.

#### LibraryStore::get_job

```rust
pub async fn get_job(&self, id: &str) -> Result<Option<JobRecord>>
```

Load one job by id, rejecting unreadable rows as `invalid_job`.

##### Errors

Returns [`LibraryError::Orm`](#libraryerror) when the read fails.

#### LibraryStore::list_jobs

```rust
pub async fn list_jobs(&self, limit: u64) -> Result<Vec<JobRecord>>
```

List recent jobs, newest first, capped at `limit` (minimum 1).

A peer writer on the same SQLite file (TRUNCATE journal) can make this
read return `SQLITE_BUSY`. The read is repeated until the lock clears
or the attempt budget is spent. `limit` is unchanged.

##### Errors

Returns [`LibraryError::Orm`](#libraryerror) when the read fails for a reason other
than file-lock contention, and [`LibraryError::Unavailable`](#libraryerror) when the
lock is still held after the retries.

#### LibraryStore::prune_terminal_jobs

```rust
pub async fn prune_terminal_jobs(&self, retention_days: u64) -> Result<u64>
```

Delete terminal jobs older than `retention_days`.

##### Errors

Returns [`LibraryError::Orm`](#libraryerror) when the delete fails.

#### LibraryStore::reserve_job_temp_path

```rust
pub async fn reserve_job_temp_path(
    &self,
    job_id: &str,
    path: &str,
    reserved_bytes: u64,
    quota_bytes: u64,
) -> Result<()>
```

Reserve `reserved_bytes` against the job temp quota and register `path`.

##### Errors

Returns [`LibraryError::Orm`](#libraryerror) when the write fails, or
[`LibraryError::Other`](#libraryerror) when the quota would be exceeded.

#### LibraryStore::register_job_temp_path

```rust
pub async fn register_job_temp_path(&self, job_id: &str, path: &str) -> Result<()>
```

Register a scratch path with a zero-byte reservation (legacy helper).

##### Errors

Returns [`LibraryError::Orm`](#libraryerror) when the insert fails.

#### LibraryStore::list_job_temp_paths

```rust
pub async fn list_job_temp_paths(&self, job_id: &str) -> Result<Vec<JobTempPath>>
```

List scratch paths registered for `job_id`.

##### Errors

Returns [`LibraryError::Orm`](#libraryerror) when the read fails.

#### LibraryStore::list_all_job_temp_paths

```rust
pub async fn list_all_job_temp_paths(&self) -> Result<Vec<JobTempPath>>
```

List every registered scratch path (used by the startup sweeper).

##### Errors

Returns [`LibraryError::Orm`](#libraryerror) when the read fails.

#### LibraryStore::unregister_job_temp_path

```rust
pub async fn unregister_job_temp_path(&self, job_id: &str, path: &str) -> Result<()>
```

Drop one scratch-path row after that path is gone from disk.

##### Errors

Returns [`LibraryError::Orm`](#libraryerror) when the delete fails.

#### LibraryStore::reserved_temp_bytes

```rust
pub async fn reserved_temp_bytes(&self) -> Result<u64>
```

Sum of reserved scratch bytes across all jobs.

##### Errors

Returns [`LibraryError::Orm`](#libraryerror) when the read fails.

#### LibraryStore::has_running_acquire_job

```rust
pub async fn has_running_acquire_job(&self) -> Result<bool>
```

True when any acquire job is currently running.

##### Errors

Returns [`LibraryError::Orm`](#libraryerror) when the read fails.

#### LibraryStore::reconcile_orphaned_acquire_rows

```rust
pub async fn reconcile_orphaned_acquire_rows(&self) -> Result<u32>
```

Mark orphaned `queued` / `downloading` edition acquire states as
`error` after a crash.

The state is written as the holding that queued it, with its event;
a state whose holding is gone (its account was revoked since) is
written on its own, with no title for an event to name.

##### Returns

Number of edition states updated.

##### Errors

Returns [`LibraryError::Orm`](#libraryerror) when a status write fails.

#### LibraryStore::count_active_jobs

```rust
pub async fn count_active_jobs(&self) -> Result<i64>
```

Count pending + running jobs (admission cap).

##### Errors

Returns [`LibraryError::Orm`](#libraryerror) when the read fails.

#### LibraryStore::job_admitted_since

```rust
pub async fn job_admitted_since(
    &self,
    dedup_key: &str,
    since: DateTime<Utc>,
) -> Result<bool>
```

True when a job with `dedup_key` was admitted at or after `since`
and is waiting, running or has succeeded.

Such a job started after `since`, so it read whatever had changed by
then. A retrying event consumer uses this to skip work an earlier
attempt already queued instead of admitting it again.

##### Arguments

* `dedup_key` - The [`JobKind::dedup_key`](#jobkinddedup_key) of the job.
* `since` - The moment the job must have been admitted after.

##### Errors

Returns [`LibraryError::Orm`](#libraryerror) when the read fails.

#### LibraryStore::job_is_active

```rust
pub async fn job_is_active(&self, dedup_key: &str) -> Result<bool>
```

True when a job with `dedup_key` is waiting or running.

A Discover page read uses this to say its shelves are being rebuilt
while an event-queued fetch or ranking has not finished.

##### Arguments

* `dedup_key` - The [`JobKind::dedup_key`](#jobkinddedup_key) of the job.

##### Errors

Returns [`LibraryError::Orm`](#libraryerror) when the read fails.

#### LibraryStore::from_connection

```rust
pub fn from_connection(db: DatabaseConnection) -> Self
```

Wrap an already-opened (and migrated) SeaORM connection.

Prefer the database plugin host (`open_library_store`) or, in tests,
`bookclerk_plugin_database_sqlite::open_memory` — this crate does not
open engine-specific connections.

#### LibraryStore::with_atomic_txn

```rust
pub fn with_atomic_txn(self, backend: Arc<dyn AtomicTxnBackend>) -> Self
```

Attach a guest atomic backend for named security operations.

Hosts attach this for every database plugin. Tests that open a local
connection leave it unset and run the same commands through
[`crate::execute_db_atomic`](#execute_db_atomic).

#### LibraryStore::with_typed_exec

```rust
pub fn with_typed_exec(self, backend: Arc<dyn TypedAtomicExec>) -> Self
```

Attach a guest `executeAtomic` backend for granted job SQL.

Hosts attach this for every database plugin so job sessions do not use
the SeaORM proxy. Tests that open a local connection leave it unset.

#### LibraryStore::with_max_binds

```rust
pub fn with_max_binds(self, max_binds: u32) -> Self
```

Records the guest's negotiated bind cap for wake `IN (…)` chunking.

#### LibraryStore::with_plan_limits

```rust
pub fn with_plan_limits(
    self,
    max_statements: u32,
    max_result_rows: u32,
    max_payload_bytes: u32,
) -> Self
```

Records negotiated statement / result / payload caps from connect.

#### LibraryStore::with_db_capabilities

```rust
pub fn with_db_capabilities(self, caps: DbCapabilities) -> Self
```

Records the full negotiated [`bookclerk_plugin_abi::DbCapabilities`](../bookclerk_plugin_abi/db_execute.md#dbcapabilities).

#### LibraryStore::db_capabilities

```rust
pub fn db_capabilities(&self) -> &DbCapabilities
```

Negotiated capability advertisement used for plan validation.

#### LibraryStore::execute_guest_atomic

```rust
pub async fn execute_guest_atomic(
    &self,
    req: ExecuteRequest,
    policy: &GuestSqlPolicy,
) -> Result<ExecuteReply, PluginError>
```

Runs guest-authored SQL after grammar, table-scope, and cap checks.

Hosts inject this into `JobRunner.job` as the granted
`bookclerk_plugin_abi::GuestDatabase::execute` path.
Job sessions pass a host-issued [`GuestSqlPolicy`](../bookclerk_plugin_abi/guest_sql.md#guestsqlpolicy)
(`books` by default).

##### Errors

Returns `bookclerk_plugin_abi::PluginError::invalid_params` when the
SQL is outside the guest grammar or allowlist, and an engine/transport
error when the batch fails.

#### LibraryStore::dispatch_chunk_size

```rust
pub fn dispatch_chunk_size(&self) -> usize
```

Subscribers packed into one dispatch plan (receipt overhead and the
live counter bump are ~13 statements).

#### LibraryStore::wake_page

```rust
pub fn wake_page(&self) -> u64
```

Wake delivery page size derived from negotiated `maxBinds`.

#### LibraryStore::connection

```rust
pub fn connection(&self) -> &DatabaseConnection
```

Borrow the underlying SeaORM connection (e.g. for [`crate::secrets`](secrets.md)).

#### LibraryStore::db

```rust
pub fn db(&self) -> &DatabaseConnection
```

Alias for [`Self::connection`](#librarystoreconnection) — convenience name used by credential modules.

#### LibraryStore::delete_elevated_operator_sessions_for_user

```rust
pub async fn delete_elevated_operator_sessions_for_user(
    &self,
    user_id: i64,
) -> Result<u64>
```

Revoke Operator sessions minted by this user's elevate-to-operator flow.

##### Errors

Returns an error when the operation fails.

#### LibraryStore::upsert_account

```rust
pub async fn upsert_account(
    &self,
    plugin_key: &str,
    account_id: &str,
    marketplace: &str,
    label: Option<&str>,
    scan_enabled: bool,
    source: &str,
) -> Result<AccountRecord>
```

Upsert an account (updates `scan_enabled` on conflict).

Accounts are keyed by `(plugin_key, account_id)`: the same store id may
exist under two plugins. `source` is the owning plugin's display alias.
Content sources should prefer [`crate::SourceScope::upsert_account`](scope.md#sourcescopeupsert_account),
which forces both.

##### Errors

Returns an error when the operation fails.

#### LibraryStore::ensure_account

```rust
pub async fn ensure_account(
    &self,
    plugin_key: &str,
    account_id: &str,
    marketplace: &str,
    label: Option<&str>,
    source: &str,
) -> Result<AccountRecord>
```

Ensure an account row exists without overwriting `scan_enabled`.

Content sources should prefer [`crate::SourceScope::ensure_account`](scope.md#sourcescopeensure_account).

##### Errors

Returns an error when the operation fails.

#### LibraryStore::remap_account_id

```rust
pub async fn remap_account_id(&self, plugin_key: &str, from: &str, to: &str) -> Result<()>
```

Remap `from` → `to` account ids within one plugin (books, holdings
and the account row).

Used when classic Libation stored an email `AccountId` and a later
login/scan discovers Audible `customer_id`. Holdings move to `to`
with `holding_removed` / `holding_added` events before the old
account row, whose foreign key would otherwise drop them, is deleted.

##### Errors

Returns an error when the operation fails.

#### LibraryStore::reconcile_account_id

```rust
pub async fn reconcile_account_id(
    &self,
    plugin_key: &str,
    canonical_id: &str,
    aliases: &[&str],
    marketplace: &str,
    label: Option<&str>,
    scan_enabled: bool,
    source: &str,
) -> Result<AccountRecord>
```

Remap any existing alias account ids onto `canonical_id`, then upsert.

##### Errors

Returns an error when the operation fails.

#### LibraryStore::get_account

```rust
pub async fn get_account(
    &self,
    plugin_key: &str,
    account_id: &str,
) -> Result<Option<AccountRecord>>
```

Loads one account by `(plugin_key, account_id)`, if present.

##### Errors

Returns a crate error when the database operation fails or inputs are invalid.

#### LibraryStore::find_accounts_by_id

```rust
pub async fn find_accounts_by_id(&self, account_id: &str) -> Result<Vec<AccountRecord>>
```

Every account row with this store id, across plugins.

##### Errors

Returns a crate error when the database operation fails.

#### LibraryStore::get_account_unique

```rust
pub async fn get_account_unique(&self, account_id: &str) -> Result<Option<AccountRecord>>
```

Resolves one account by store id, failing closed when two plugins own
that id.

##### Errors

Returns [`LibraryError::Conflict`](#libraryerror) when the id is ambiguous.

#### LibraryStore::list_accounts

```rust
pub async fn list_accounts(&self) -> Result<Vec<AccountRecord>>
```

Lists all store accounts in the library database.


##### Returns

`Result<Vec<AccountRecord>>` — `Ok` on success.

##### Errors

Returns a crate error when the database operation fails or inputs are invalid.

#### LibraryStore::list_accounts_for_plugin

```rust
pub async fn list_accounts_for_plugin(
    &self,
    plugin_key: &str,
) -> Result<Vec<AccountRecord>>
```

List the accounts of one storefront plugin, ordered by account id.

The filter runs in the query, so the read is bounded by that
plugin's accounts rather than the whole library's.

##### Arguments

* `plugin_key` - Canonical plugin key whose accounts to read.

##### Errors

Returns the read error.

#### LibraryStore::count_identities

```rust
pub async fn count_identities(&self) -> Result<i64>
```

Count library identity rows (SQL `COUNT`, not a full fetch).

Same cardinality as [`Self::count_accounts`](#librarystorecount_accounts). Prefer this when the caller
only needs a number for an operator-facing summary.

A peer writer on the same SQLite file (TRUNCATE journal) can make this
read return `SQLITE_BUSY`. The read is repeated until the lock clears
or the attempt budget is spent.

##### Errors

Returns [`LibraryError::Orm`](#libraryerror) when the read fails for a reason other
than file-lock contention, and [`LibraryError::Unavailable`](#libraryerror) when the
lock is still held after the retries.

#### LibraryStore::count_accounts

```rust
pub async fn count_accounts(&self) -> Result<i64>
```

Count account rows (SQL `COUNT`, not a full fetch).

Uses [`Self::count_identities`](#librarystorecount_identities), including its wait for a peer SQLite lock.

##### Errors

Returns [`LibraryError::Orm`](#libraryerror) when the read fails for a reason other
than file-lock contention, and [`LibraryError::Unavailable`](#libraryerror) when the
lock is still held after the retries.

#### LibraryStore::find_accounts

```rust
pub async fn find_accounts(&self, identifier: &str) -> Result<Vec<AccountRecord>>
```

Every account whose id or nickname (`label`) matches, case-insensitive.

##### Errors

Returns an error when the operation fails.

#### LibraryStore::find_account

```rust
pub async fn find_account(
    &self,
    identifier: &str,
    plugin_key: Option<&str>,
) -> Result<Option<AccountRecord>>
```

Resolve one account by id or nickname (`label`), case-insensitive.

`plugin_key` narrows the match to one plugin. Without it, a needle that
matches accounts of two plugins fails closed.

##### Errors

Returns [`LibraryError::Conflict`](#libraryerror) when the needle is ambiguous.

#### LibraryStore::set_scan_enabled

```rust
pub async fn set_scan_enabled(
    &self,
    plugin_key: &str,
    account_id: &str,
    scan_enabled: bool,
) -> Result<()>
```

Toggle whether an account is included in automatic library scans.

##### Errors

Returns an error when the operation fails.

#### LibraryStore::mark_connection_active

```rust
pub async fn mark_connection_active(
    &self,
    plugin_key: &str,
    account_id: &str,
) -> Result<()>
```

Mark bookstore credentials active again (after reconnect).

##### Errors

Returns an error when the operation fails.

#### LibraryStore::delete_account_secrets

```rust
pub async fn delete_account_secrets(
    &self,
    plugin_key: &str,
    account_id: &str,
) -> Result<()>
```

Delete all `encrypted_secrets` rows for an account (source auth + Widevine CDM).

##### Errors

Returns an error when the operation fails.

#### LibraryStore::revoke_credentials

```rust
pub async fn revoke_credentials(&self, plugin_key: &str, account_id: &str) -> Result<()>
```

Mark bookstore credentials revoked, remove the account's holdings
(with `holding_removed` for each) and forget its sync token, without
deleting the account, its flat library rows or acquired files.

The account no longer reaches the store, so the library no longer
holds its titles through it. A later reconnect starts a full scan
pass (no sync token), which adds the holdings back.

##### Errors

Returns an error when the operation fails.

#### LibraryStore::upsert_portal_identity

```rust
pub async fn upsert_portal_identity(
    &self,
    provider: &str,
    external_user_id: &str,
    label: Option<&str>,
) -> Result<PortalIdentity>
```

Create or fetch a portal identity for `(provider, external_user_id)`.

Ensures a first-party [`UserRecord`](#userrecord) (role `member`) is linked via
`portal_identities.user_id`.

##### Errors

Returns an error when the operation fails.

#### LibraryStore::create_user

```rust
pub async fn create_user(
    &self,
    role: UserRole,
    display_name: Option<&str>,
    password_hash: Option<&str>,
) -> Result<UserRecord>
```

Create a first-party user.

##### Errors

Returns an error when the operation fails.

#### LibraryStore::create_user_with_login

```rust
pub async fn create_user_with_login(
    &self,
    role: UserRole,
    display_name: Option<&str>,
    login_name: Option<&str>,
    password_hash: Option<&str>,
) -> Result<UserRecord>
```

Create a first-party user with optional local login_name and email.

##### Errors

Returns an error when the operation fails.

#### LibraryStore::create_user_with_profile

```rust
pub async fn create_user_with_profile(
    &self,
    role: UserRole,
    display_name: Option<&str>,
    login_name: Option<&str>,
    email: Option<&str>,
    password_hash: Option<&str>,
) -> Result<UserRecord>
```

Create a first-party user with login, email, and optional password hash.

##### Errors

Returns an error when the operation fails.

#### LibraryStore::get_user

```rust
pub async fn get_user(&self, id: i64) -> Result<Option<UserRecord>>
```

Look up a first-party user by id.

##### Errors

Returns an error when the operation fails.

#### LibraryStore::get_user_by_login_name

```rust
pub async fn get_user_by_login_name(&self, login_name: &str) -> Result<Option<UserRecord>>
```

Look up by local login_name (case-insensitive).

##### Errors

Returns an error when the operation fails.

#### LibraryStore::get_user_by_email

```rust
pub async fn get_user_by_email(&self, email: &str) -> Result<Option<UserRecord>>
```

Look up by email (case-insensitive).

##### Errors

Returns an error when the operation fails.

#### LibraryStore::get_user_password_hash

```rust
pub async fn get_user_password_hash(&self, id: i64) -> Result<Option<String>>
```

Raw password hash for verification (never expose via API).

##### Errors

Returns an error when the operation fails.

#### LibraryStore::list_users

```rust
pub async fn list_users(&self) -> Result<Vec<UserRecord>>
```

List all first-party users (admin tooling).

##### Errors

Returns an error when the operation fails.

#### LibraryStore::list_user_presence_extras

```rust
pub async fn list_user_presence_extras(
    &self,
    listening_within: Duration,
) -> Result<HashMap<i64, UserPresenceExtras>>
```

Presence, listening, and storefront links for administrator user lists.

`online` means any non-expired portal session. `listening` is the newest
unfinished progress row with `last_listened_at` within
`listening_within` (typical UI window: ~30 minutes). Used by
`GET /api/users`. Session activity (`last_active_at`) comes from
unexpired `portal_sessions.last_used_at`. Durable last-seen lives on
`users.last_seen_at` (survives logout) and is refreshed with session
use at most once per minute.

##### Errors

Returns an error when the operation fails.

#### LibraryStore::delete_user

```rust
pub async fn delete_user(&self, id: i64) -> Result<()>
```

Delete a first-party user and personal data.

Removes portal identities, account links, wishlist rows, claim tickets,
sessions, and prefs for the user. Acquired library books are retained.
Refuses to delete the last active owner. Publishes `wishlist_changed`
(status `deleted`) for each open wish removed with the user, so
Discover stops counting it for everyone else.

##### Errors

Returns an error when the operation fails.

#### LibraryStore::set_user_status

```rust
pub async fn set_user_status(&self, id: i64, status: UserStatus) -> Result<UserRecord>
```

Set user status (`active` / `disabled`).

Refuses to disable the last active owner.

##### Errors

Returns an error when the operation fails.

#### LibraryStore::set_user_display_name

```rust
pub async fn set_user_display_name(
    &self,
    id: i64,
    display_name: Option<&str>,
) -> Result<UserRecord>
```

Set display name (does not bump security_version).

##### Errors

Returns an error when the operation fails.

#### LibraryStore::set_user_password_hash

```rust
pub async fn set_user_password_hash(
    &self,
    id: i64,
    password_hash: Option<&str>,
) -> Result<UserRecord>
```

Set or clear Argon2id password hash; bumps `security_version`.

##### Errors

Returns an error when the operation fails.

#### LibraryStore::set_user_login_name

```rust
pub async fn set_user_login_name(
    &self,
    id: i64,
    login_name: Option<&str>,
) -> Result<UserRecord>
```

Set login_name (unique); does not bump security_version.

##### Errors

Returns an error when the operation fails.

#### LibraryStore::set_user_email

```rust
pub async fn set_user_email(&self, id: i64, email: Option<&str>) -> Result<UserRecord>
```

Set contact email (unique when set); does not bump security_version.

##### Errors

Returns an error when the operation fails.

#### LibraryStore::set_user_avatar_source

```rust
pub async fn set_user_avatar_source(
    &self,
    id: i64,
    source: Option<&str>,
) -> Result<UserRecord>
```

Set the user's picture source (`monogram` / `gravatar` / `upload` / `sso:{id}` / auto).

`None` or empty stores auto-resolve (upload, then last-used SSO, then Gravatar).

##### Errors

Returns an error when the user is missing or the write fails.

#### LibraryStore::set_portal_identity_picture

```rust
pub async fn set_portal_identity_picture(
    &self,
    provider: &str,
    external_user_id: &str,
    picture_url: Option<&str>,
) -> Result<()>
```

Store or clear the HTTPS picture URL on a portal identity.

Missing identities are ignored so callers can run this after a failed
lookup without an extra round-trip.

##### Errors

Returns an error when the write fails.

#### LibraryStore::list_user_sso_pictures

```rust
pub async fn list_user_sso_pictures(&self, user_id: i64) -> Result<Vec<UserSsoPicture>>
```

IdP pictures linked to `user_id`, with last portal-session activity.

Identities without a `picture_url` are omitted. `last_used_at` is the
latest session `last_used_at` or `created_at` for that identity.

##### Errors

Returns an error when the query fails.

#### LibraryStore::delete_portal_sessions_for_user

```rust
pub async fn delete_portal_sessions_for_user(&self, user_id: i64) -> Result<u64>
```

Delete all portal sessions for identities linked to `user_id`.

##### Errors

Returns an error when the operation fails.

#### LibraryStore::insert_user_invite

```rust
pub async fn insert_user_invite(
    &self,
    token_hash: &str,
    role: UserRole,
    login_name: Option<&str>,
    display_name: Option<&str>,
    expires_at: DateTime<Utc>,
    created_by: &str,
) -> Result<UserInviteRecord>
```

Insert a user invite (store hash only).

##### Errors

Returns an error when the operation fails.

#### LibraryStore::redeem_user_invite

```rust
pub async fn redeem_user_invite(&self, token_hash: &str) -> Result<UserInviteRecord>
```

Atomically redeem a user invite.

##### Errors

Returns an error when the operation fails.

#### LibraryStore::set_user_role

```rust
pub async fn set_user_role(&self, id: i64, role: UserRole) -> Result<UserRecord>
```

Set user role (`owner` / `administrator` / `member`).

Refuses to demote the last active owner.

##### Errors

Returns an error when the operation fails.

#### LibraryStore::count_administrators

```rust
pub async fn count_administrators(&self) -> Result<u64>
```

Count administrators (includes disabled).

##### Errors

Returns an error when the operation fails.

#### LibraryStore::count_active_administrators

```rust
pub async fn count_active_administrators(&self) -> Result<u64>
```

Count administrators with status `active`.

##### Errors

Returns an error when the operation fails.

#### LibraryStore::count_owners

```rust
pub async fn count_owners(&self) -> Result<u64>
```

Count owners (includes disabled) — bootstrap / inventory.

##### Errors

Returns an error when the operation fails.

#### LibraryStore::count_active_owners

```rust
pub async fn count_active_owners(&self) -> Result<u64>
```

Count owners with status `active` (last-owner demote/disable/delete guard).

##### Errors

Returns an error when the operation fails.

#### LibraryStore::ensure_users_bridged

```rust
pub async fn ensure_users_bridged(&self) -> Result<usize>
```

Backfill `portal_identities.user_id` and remap prefs `portal:{id}` → `user:{id}`.

Safe to call repeatedly after migrations. Does not link CLI-created store
accounts that have no `account_links` (avoids widening access).

##### Errors

Returns an error when the operation fails.

#### LibraryStore::ensure_local_portal_identity

```rust
pub async fn ensure_local_portal_identity(
    &self,
    user_id: i64,
    label: Option<&str>,
) -> Result<PortalIdentity>
```

Ensure a `local` portal identity exists for a first-party user.

##### Errors

Returns an error when the operation fails.

#### LibraryStore::get_portal_identity

```rust
pub async fn get_portal_identity(
    &self,
    provider: &str,
    external_user_id: &str,
) -> Result<Option<PortalIdentity>>
```

Loads a portal identity by primary key, if present.

##### Arguments

* `provider` - External provider id.
* `external_user_id` - User id at the external provider.

##### Returns

`Result<Option<crate::models::PortalIdentity>>` — `Ok` on success.

##### Errors

Returns a crate error when the database operation fails or inputs are invalid.

#### LibraryStore::get_portal_identity_by_id

```rust
pub async fn get_portal_identity_by_id(&self, id: i64) -> Result<Option<PortalIdentity>>
```

Look up portal identity by row id.

##### Errors

Returns an error when the operation fails.

#### LibraryStore::insert_claim_ticket

```rust
pub async fn insert_claim_ticket(
    &self,
    token_hash: &str,
    identity_id: Option<i64>,
    expires_at: DateTime<Utc>,
    created_by: &str,
) -> Result<ClaimTicketRecord>
```

Insert a claim ticket (store only the hash).

##### Errors

Returns an error when the operation fails.

#### LibraryStore::get_claim_ticket_by_hash

```rust
pub async fn get_claim_ticket_by_hash(
    &self,
    token_hash: &str,
) -> Result<Option<ClaimTicketRecord>>
```

Loads a claim ticket row by its token hash, if present.

##### Arguments

* `token_hash` - SHA-256 hex digest of the opaque token.

##### Returns

`Result<Option<crate::models::ClaimTicketRecord>>` — `Ok` on success.

##### Errors

Returns a crate error when the database operation fails or inputs are invalid.

#### LibraryStore::list_open_claim_tickets

```rust
pub async fn list_open_claim_tickets(&self) -> Result<Vec<ClaimTicketRecord>>
```

List unredeemed, unexpired claim tickets (newest first).

##### Errors

Returns an error when the operation fails.

#### LibraryStore::redeem_claim_ticket

```rust
pub async fn redeem_claim_ticket(&self, token_hash: &str) -> Result<ClaimTicketRecord>
```

Atomically mark a claim ticket redeemed.

Uses a single `UPDATE … WHERE redeemed_at IS NULL AND expires_at > now` so
concurrent redeemers cannot both succeed.

##### Errors

Returns an error when the operation fails.

#### LibraryStore::redeem_claim_ticket_to_session

```rust
pub async fn redeem_claim_ticket_to_session(
    &self,
    token_hash: &str,
    session_hash: &str,
    expires_at: DateTime<Utc>,
    client: Option<&SessionClientInfo>,
    new_password_hash: Option<&str>,
    password_fingerprint: Option<&str>,
) -> Result<PortalIdentity>
```

Consume a claim ticket, optionally set an unset local password, and mint
a portal session in one transaction.

`new_password_hash` is assigned only while the ticket-bound local user
still has no password. A failed consume rolls back password and session
writes. Missing password for an unset local user aborts without consuming
the ticket.

##### Arguments

* `token_hash` - SHA-256 hex digest of the claim ticket.
* `session_hash` - SHA-256 hex digest of the new portal session token.
* `expires_at` - RFC 3339 expiry for the minted session.
* `client` - Optional client metadata stored on the session row.
* `new_password_hash` - Argon2id hash to set when the local user has none.
* `password_fingerprint` - Stable HMAC of the plaintext password for
  atomic-execute idempotency; ignored by the mutation SQL.

##### Returns

The portal identity bound to the consumed ticket.

##### Errors

Returns a crate error when the ticket is invalid/consumed, a required
invite password is missing, or the database operation fails.

#### LibraryStore::insert_portal_session

```rust
pub async fn insert_portal_session(
    &self,
    token_hash: &str,
    identity_id: i64,
    expires_at: DateTime<Utc>,
) -> Result<()>
```

Create a portal session (hash only).

##### Errors

Returns an error when the operation fails.

#### LibraryStore::insert_portal_session_with_client

```rust
pub async fn insert_portal_session_with_client(
    &self,
    token_hash: &str,
    identity_id: i64,
    expires_at: DateTime<Utc>,
    client: Option<&SessionClientInfo>,
) -> Result<()>
```

Create a portal session with optional client metadata.

##### Errors

Returns an error when the operation fails.

#### LibraryStore::insert_clerk_session

```rust
pub async fn insert_clerk_session(
    &self,
    token_hash: &str,
    identity_id: i64,
    expires_at: DateTime<Utc>,
    label: &str,
) -> Result<i64>
```

Creates a clerk sign-in session: a token a clerk presents
as `Authorization: Bookclerk-Clerk`, never as a cookie.

##### Arguments

* `token_hash` - SHA-256 hex of the token (the token is never stored).
* `identity_id` - The user's portal identity.
* `expires_at` - First expiry; each use moves it forward
  ([`Self::get_clerk_session_identity`](#librarystoreget_clerk_session_identity)).
* `label` - The name the clerk gave itself, shown in the
  user's session list.

##### Returns

The new session's row id.

##### Errors

Returns an error when the insert fails.

#### LibraryStore::delete_clerk_session

```rust
pub async fn delete_clerk_session(&self, token_hash: &str) -> Result<bool>
```

Deletes a clerk sign-in session by token hash (the clerk signing
itself out). A browser session with that hash is left alone.

##### Errors

Returns an error when the delete fails.

#### LibraryStore::delete_portal_session

```rust
pub async fn delete_portal_session(&self, token_hash: &str) -> Result<bool>
```

Delete a portal session by token hash (logout / revoke).

##### Errors

Returns an error when the operation fails.

#### LibraryStore::list_portal_sessions_for_identity

```rust
pub async fn list_portal_sessions_for_identity(
    &self,
    identity_id: i64,
) -> Result<Vec<(i64, String, String)>>
```

List portal sessions for an identity (id, created_at, expires_at).

##### Errors

Returns an error when the operation fails.

#### LibraryStore::list_portal_session_records_for_identity

```rust
pub async fn list_portal_session_records_for_identity(
    &self,
    identity_id: i64,
) -> Result<Vec<PortalSessionRecord>>
```

List portal sessions with client metadata for session management UIs.

##### Errors

Returns an error when the operation fails.

#### LibraryStore::delete_portal_session_by_id_for_identity

```rust
pub async fn delete_portal_session_by_id_for_identity(
    &self,
    id: i64,
    identity_id: i64,
) -> Result<bool>
```

Delete a portal session by id only if it belongs to `identity_id`.

##### Errors

Returns an error when the operation fails.

#### LibraryStore::count_account_links_for_account

```rust
pub async fn count_account_links_for_account(
    &self,
    plugin_key: &str,
    account_id: &str,
) -> Result<u64>
```

Count account_links referencing `account_id`.

##### Errors

Returns an error when the operation fails.

#### LibraryStore::get_portal_session_identity

```rust
pub async fn get_portal_session_identity(
    &self,
    token_hash: &str,
) -> Result<Option<PortalIdentity>>
```

Resolve a valid browser (cookie) session to its identity.

Refreshes `portal_sessions.last_used_at` and `users.last_seen_at` at most
once per minute so administrator presence can tell active from idle
without a write on every request. `last_seen_at` is kept after logout.
A library card is not a browser session and resolves to
nothing here.

##### Errors

Returns an error when the operation fails.

#### LibraryStore::get_clerk_session_identity

```rust
pub async fn get_clerk_session_identity(
    &self,
    token_hash: &str,
    idle: Duration,
) -> Result<Option<PortalIdentity>>
```

Resolve a valid clerk sign-in session to its identity.

Like [`Self::get_portal_session_identity`](#librarystoreget_portal_session_identity), but only for a `client`
session, and each touch (at most once a minute) also moves its expiry
to `idle` from now, so a client in use stays signed in and one left
unused for `idle` is signed out.

##### Errors

Returns an error when the operation fails.

#### LibraryStore::clerk_session_person

```rust
pub async fn clerk_session_person(&self, token_hash: &str) -> Result<Option<i64>>
```

The person a library card is signed in as right now, read without
touching the session: a `clerk` session with that token hash that
has not expired, whose identity names a user who exists and is not
disabled.

A copy download asks this on every request
([`crate::control_plane::authorize_copy`](control_plane.md#authorize_copy)); it does not keep the
card signed in, which only the card's own use does
([`Self::get_clerk_session_identity`](#librarystoreget_clerk_session_identity)).

##### Arguments

* `token_hash` - SHA-256 hex of the card's token.

##### Errors

Returns an error when a read fails.

#### LibraryStore::insert_operator_session

```rust
pub async fn insert_operator_session(
    &self,
    token_hash: &str,
    expires_at: DateTime<Utc>,
) -> Result<()>
```

Create a durable operator session (hash only).

##### Errors

Returns an error when the operation fails.

#### LibraryStore::insert_operator_session_with_client

```rust
pub async fn insert_operator_session_with_client(
    &self,
    token_hash: &str,
    expires_at: DateTime<Utc>,
    client: Option<&SessionClientInfo>,
) -> Result<()>
```

Create a durable operator session with optional client metadata.

##### Errors

Returns an error when the operation fails.

#### LibraryStore::insert_elevated_operator_session

```rust
pub async fn insert_elevated_operator_session(
    &self,
    token_hash: &str,
    expires_at: DateTime<Utc>,
    elevated_from_user_id: i64,
) -> Result<()>
```

Create an elevated operator session (Administrator reauth → short TTL).

##### Errors

Returns an error when the operation fails.

#### LibraryStore::insert_elevated_operator_session_with_client

```rust
pub async fn insert_elevated_operator_session_with_client(
    &self,
    token_hash: &str,
    expires_at: DateTime<Utc>,
    elevated_from_user_id: i64,
    client: Option<&SessionClientInfo>,
) -> Result<()>
```

Create an elevated operator session with optional client metadata.

##### Errors

Returns an error when the operation fails.

#### LibraryStore::get_operator_session

```rust
pub async fn get_operator_session(
    &self,
    token_hash: &str,
) -> Result<Option<OperatorSessionRecord>>
```

Look up a valid operator session (touches `last_used_at`).

##### Errors

Returns an error when the operation fails.

#### LibraryStore::operator_session_valid

```rust
pub async fn operator_session_valid(&self, token_hash: &str) -> Result<bool>
```

Return whether a hashed operator session is still valid (and touch last_used).

##### Errors

Returns an error when the operation fails.

#### LibraryStore::set_operator_session_impersonating

```rust
pub async fn set_operator_session_impersonating(
    &self,
    token_hash: &str,
    user_id: Option<i64>,
) -> Result<OperatorSessionRecord>
```

Set or clear impersonation on an operator session.

##### Errors

Returns an error when the operation fails.

#### LibraryStore::delete_operator_session

```rust
pub async fn delete_operator_session(&self, token_hash: &str) -> Result<bool>
```

Delete an operator session by token hash (logout / revoke).

##### Errors

Returns an error when the operation fails.

#### LibraryStore::delete_operator_session_by_id

```rust
pub async fn delete_operator_session_by_id(&self, id: i64) -> Result<bool>
```

Delete an operator session by row id.

##### Errors

Returns an error when the operation fails.

#### LibraryStore::prune_expired_operator_sessions

```rust
pub async fn prune_expired_operator_sessions(&self) -> Result<u64>
```

Delete expired operator sessions (bounded cleanup).

##### Errors

Returns an error when the operation fails.

#### LibraryStore::list_operator_sessions

```rust
pub async fn list_operator_sessions(&self) -> Result<Vec<OperatorSessionRecord>>
```

List active operator sessions (newest first) for remote sign-out UI.

##### Errors

Returns an error when the operation fails.

#### LibraryStore::first_portal_identity_for_user

```rust
pub async fn first_portal_identity_for_user(
    &self,
    user_id: i64,
) -> Result<Option<PortalIdentity>>
```

First portal identity linked to a first-party user (for impersonation scoping).

##### Errors

Returns an error when the operation fails.

#### LibraryStore::list_portal_identities_for_user

```rust
pub async fn list_portal_identities_for_user(
    &self,
    user_id: i64,
) -> Result<Vec<PortalIdentity>>
```

All portal identities linked to a first-party user.

##### Errors

Returns an error when the operation fails.

#### LibraryStore::link_portal_identity_to_user

```rust
pub async fn link_portal_identity_to_user(
    &self,
    provider: &str,
    external_user_id: &str,
    user_id: i64,
    label: Option<&str>,
) -> Result<PortalIdentity>
```

Link `(provider, external_user_id)` to an existing user (SSO / invite_only).

Creates the identity row when missing. Refuses to steal an identity already
bound to a different user.

##### Errors

Returns an error when the operation fails.

#### LibraryStore::insert_oidc_rp_state

```rust
pub async fn insert_oidc_rp_state(
    &self,
    state_hash: &str,
    provider_id: &str,
    pkce_verifier: &str,
    nonce: &str,
    purpose: &str,
    user_id: Option<i64>,
    expires_at: DateTime<Utc>,
) -> Result<()>
```

Insert a short-lived OIDC RP authorize state row.

##### Errors

Returns an error when the operation fails.

#### LibraryStore::take_oidc_rp_state

```rust
pub async fn take_oidc_rp_state(
    &self,
    state_hash: &str,
) -> Result<Option<(String, String, String, String, Option<i64>)>>
```

Consume (delete) an OIDC RP state by hash if it has not expired.

Find + delete run in one transaction so concurrent callbacks cannot both
observe the same unused state.

##### Errors

Returns an error when the operation fails.

#### LibraryStore::insert_webauthn_credential

```rust
pub async fn insert_webauthn_credential(
    &self,
    user_id: i64,
    credential_id: &str,
    passkey_json: &str,
    name: Option<&str>,
) -> Result<()>
```

Store a WebAuthn credential for a user.

##### Arguments

* `user_id` - First-party user who owns the passkey.
* `credential_id` - Base64url credential id.
* `passkey_json` - Serialized `webauthn_rs` passkey.
* `name` - Label shown in Settings; empty becomes `None`.

##### Errors

Returns an error when the insert fails (including unique credential id).

#### LibraryStore::list_webauthn_credentials

```rust
pub async fn list_webauthn_credentials(&self, user_id: i64) -> Result<Vec<StoredPasskey>>
```

Passkeys registered to a user.

##### Errors

Returns an error when the operation fails.

#### LibraryStore::count_webauthn_credentials

```rust
pub async fn count_webauthn_credentials(&self, user_id: i64) -> Result<u64>
```

Count passkeys for a user.

##### Errors

Returns an error when the operation fails.

#### LibraryStore::get_webauthn_credential_by_cred_id

```rust
pub async fn get_webauthn_credential_by_cred_id(
    &self,
    credential_id: &str,
) -> Result<Option<(i64, i64, String)>>
```

Look up a passkey by credential id.

##### Errors

Returns an error when the operation fails.

#### LibraryStore::update_webauthn_credential

```rust
pub async fn update_webauthn_credential(&self, id: i64, passkey_json: &str) -> Result<()>
```

Replace passkey JSON after a successful assertion (counter update).

##### Errors

Returns an error when the operation fails.

#### LibraryStore::delete_webauthn_credential

```rust
pub async fn delete_webauthn_credential(&self, user_id: i64, id: i64) -> Result<bool>
```

Delete a passkey owned by `user_id`.

##### Errors

Returns an error when the operation fails.

#### LibraryStore::set_user_totp_enabled

```rust
pub async fn set_user_totp_enabled(&self, user_id: i64, enabled: bool) -> Result<()>
```

Set whether this user has a confirmed TOTP authenticator.

##### Arguments

* `user_id` - First-party user id.
* `enabled` - `true` after a successful enroll confirm; `false` on disable.

##### Errors

Returns [`LibraryError::NotFound`](#libraryerror) when the user does not exist.

#### LibraryStore::confirm_totp_enrollment

```rust
pub async fn confirm_totp_enrollment(&self, user_id: i64, secret_b32: &str) -> Result<()>
```

Promote a pending TOTP secret to primary and set `totp_enabled` in one transaction.

##### Errors

Returns [`LibraryError::NotFound`](#libraryerror) when the user does not exist.

#### LibraryStore::disable_user_totp

```rust
pub async fn disable_user_totp(&self, user_id: i64) -> Result<()>
```

Delete TOTP secrets and clear `totp_enabled` in one transaction.

##### Errors

Returns [`LibraryError::NotFound`](#libraryerror) when the user does not exist.

#### LibraryStore::insert_webauthn_challenge

```rust
pub async fn insert_webauthn_challenge(
    &self,
    challenge_id: &str,
    user_id: Option<i64>,
    kind: &str,
    state_json: &str,
    expires_at: DateTime<Utc>,
) -> Result<()>
```

Insert a WebAuthn ceremony state.

##### Errors

Returns an error when the operation fails.

#### LibraryStore::take_webauthn_challenge

```rust
pub async fn take_webauthn_challenge(
    &self,
    challenge_id: &str,
    kind: &str,
) -> Result<Option<(Option<i64>, String)>>
```

Consume a WebAuthn ceremony by id (transactional delete-and-return).

##### Errors

Returns an error when the operation fails.

#### LibraryStore::prune_expired_oidc_rp_states

```rust
pub async fn prune_expired_oidc_rp_states(&self) -> Result<u64>
```

Drop expired OIDC RP states (bounded cleanup).

##### Errors

Returns an error when the operation fails.

#### LibraryStore::prune_expired_webauthn_challenges

```rust
pub async fn prune_expired_webauthn_challenges(&self) -> Result<u64>
```

Drop expired WebAuthn challenges (bounded cleanup).

##### Errors

Returns an error when the operation fails.

#### LibraryStore::count_oidc_rp_states

```rust
pub async fn count_oidc_rp_states(&self) -> Result<u64>
```

Count pending OIDC RP states (for public-begin caps).

##### Errors

Returns an error when the operation fails.

#### LibraryStore::count_webauthn_challenges

```rust
pub async fn count_webauthn_challenges(&self) -> Result<u64>
```

Count pending WebAuthn challenges (for public-begin caps).

##### Errors

Returns an error when the operation fails.

#### LibraryStore::portal_session_created_at

```rust
pub async fn portal_session_created_at(
    &self,
    token_hash: &str,
) -> Result<Option<DateTime<Utc>>>
```

Created-at of a still-valid portal session (recent-auth checks).

##### Errors

Returns an error when the operation fails.

#### LibraryStore::set_portal_session_created_at

```rust
pub async fn set_portal_session_created_at(
    &self,
    token_hash: &str,
    created_at: DateTime<Utc>,
) -> Result<bool>
```

Overwrite `created_at` for a portal session (tests / recent-auth fixtures).

##### Errors

Returns an error when the operation fails.

#### LibraryStore::insert_security_audit_event

```rust
pub async fn insert_security_audit_event(
    &self,
    actor: &str,
    action: &str,
    detail_json: Option<&str>,
) -> Result<SecurityAuditEvent>
```

Appends one security audit event row.

##### Arguments

* `actor` - Actor label.
* `action` - Audit action verb.
* `detail_json` - `detail_json` argument for this call.

##### Returns

`Result<crate::models::SecurityAuditEvent>` — `Ok` on success.

##### Errors

Returns a crate error when the database operation fails or inputs are invalid.

#### LibraryStore::insert_oidc_client

```rust
pub async fn insert_oidc_client(
    &self,
    client_id: &str,
    client_secret_hash: Option<&str>,
    redirect_uris: &[String],
    name: Option<&str>,
    issue_refresh_token: bool,
    allowed_scopes: &[String],
    enabled: bool,
    plugin_id: Option<&str>,
) -> Result<OidcClientRecord>
```

Insert a new OIDC client (public clients may omit secret hash).

Custom (operator) clients pass `plugin_id = None` and typically
`enabled = true`. Plugin-owned templates pass the plugin id and start
disabled.

##### Errors

Returns an error when the operation fails.

#### LibraryStore::upsert_plugin_oidc_client

```rust
pub async fn upsert_plugin_oidc_client(
    &self,
    client_id: &str,
    plugin_id: &str,
    name: &str,
    redirect_uris: &[String],
    issue_refresh_token: bool,
    allowed_scopes: &[String],
) -> Result<OidcClientRecord>
```

Insert a plugin-owned OIDC client when missing; refresh redirect URIs when present.

New rows start **disabled**. Existing rows owned by the **same** plugin keep
`enabled`, name, scopes, and secret. Collisions with custom (operator) clients
or a different plugin's `client_id` are hard errors — this never mutates
an unowned row. Alias-era rows (`plugin_id = "audiobookshelf"`) are rewritten
to the provenance-qualified PluginKey (`…#audiobookshelf`) on the next
heartbeat from that guest; a different PluginKey with the same alias
cannot take the client over.

##### Errors

Returns [`LibraryError::Conflict`](#libraryerror) when `client_id` is already owned by a
custom client or another plugin.

#### LibraryStore::update_oidc_client

```rust
pub async fn update_oidc_client(
    &self,
    client_id: &str,
    redirect_uris: &[String],
    name: Option<&str>,
    issue_refresh_token: bool,
    allowed_scopes: &[String],
    enabled: bool,
    secret_hash: Option<Option<String>>,
) -> Result<Option<OidcClientRecord>>
```

Update an existing OIDC client without changing the secret unless `secret_hash` is `Some`.

Pass `Some(None)` as `secret_hash` to clear the secret (public PKCE). Pass `None` to keep.

##### Errors

Returns an error when the operation fails.

#### LibraryStore::set_oidc_client_secret

```rust
pub async fn set_oidc_client_secret(
    &self,
    client_id: &str,
    client_secret_hash: Option<&str>,
) -> Result<Option<OidcClientRecord>>
```

Replace the stored client-secret hash (or clear it).

##### Errors

Returns an error when the operation fails.

#### LibraryStore::delete_oidc_client

```rust
pub async fn delete_oidc_client(&self, client_id: &str) -> Result<bool>
```

Delete a registered OIDC client by `client_id`.

##### Errors

Returns an error when the operation fails.

#### LibraryStore::list_oidc_clients

```rust
pub async fn list_oidc_clients(&self) -> Result<Vec<OidcClientRecord>>
```

List registered OIDC clients, oldest first.

##### Errors

Returns an error when the operation fails.

#### LibraryStore::get_oidc_client

```rust
pub async fn get_oidc_client(&self, client_id: &str) -> Result<Option<OidcClientRecord>>
```

Loads a registered OIDC client by `client_id`, if present.

##### Errors

Returns a crate error when the database operation fails or inputs are invalid.

#### LibraryStore::insert_oidc_auth_code

```rust
pub async fn insert_oidc_auth_code(
    &self,
    code_hash: &str,
    client_id: &str,
    user_id: i64,
    redirect_uri: &str,
    code_challenge: &str,
    code_challenge_method: &str,
    scope: &str,
    expires_at: DateTime<Utc>,
) -> Result<()>
```

Persists a hashed OIDC authorization code with PKCE metadata.

##### Arguments

* `code_hash` - SHA-256 hex digest of the authorization code.
* `client_id` - OIDC client_id.
* `user_id` - First-party user id.
* `redirect_uri` - OAuth redirect URI.
* `code_challenge` - PKCE code_challenge.
* `code_challenge_method` - PKCE method (`S256` or `plain`).
* `scope` - OAuth scope string.
* `expires_at` - Expiry timestamp.

##### Returns

`Result<()>` — `Ok` on success.

##### Errors

Returns a crate error when the database operation fails or inputs are invalid.

#### LibraryStore::peek_oidc_auth_code

```rust
pub async fn peek_oidc_auth_code(
    &self,
    code_hash: &str,
) -> Result<Option<(String, i64, String, String, String, String)>>
```

Reads an unconsumed, unexpired auth code without consuming it; returns
(client_id, user_id, redirect_uri, challenge, method, scope). For a
caller that records what the code grants before it consumes the code,
so a failed write leaves the code usable for a retry.

##### Errors

Returns an error when the read fails.

#### LibraryStore::consume_oidc_auth_code

```rust
pub async fn consume_oidc_auth_code(
    &self,
    code_hash: &str,
) -> Result<Option<(String, i64, String, String, String, String)>>
```

Atomically consume an auth code; returns (client_id, user_id, redirect_uri, challenge, method, scope).

##### Errors

Returns an error when the operation fails.

#### LibraryStore::insert_oidc_refresh_token

```rust
pub async fn insert_oidc_refresh_token(
    &self,
    token_hash: &str,
    client_id: &str,
    user_id: i64,
    scope: &str,
    expires_at: DateTime<Utc>,
) -> Result<()>
```

Persists a hashed OIDC refresh token.

##### Arguments

* `token_hash` - SHA-256 hex digest of the opaque token.
* `client_id` - OIDC client_id.
* `user_id` - First-party user id.
* `scope` - OAuth scope string.
* `expires_at` - Expiry timestamp.

##### Returns

`Result<()>` — `Ok` on success.

##### Errors

Returns a crate error when the database operation fails or inputs are invalid.

#### LibraryStore::get_oidc_refresh_token

```rust
pub async fn get_oidc_refresh_token(
    &self,
    token_hash: &str,
) -> Result<Option<(String, i64, String)>>
```

Loads a refresh token row by hash, if present.

##### Arguments

* `token_hash` - SHA-256 hex digest of the opaque token.

##### Returns

`Result<Option<(String, i64, String)>>` — `Ok` on success.

##### Errors

Returns a crate error when the database operation fails or inputs are invalid.

#### LibraryStore::revoke_oidc_refresh_token

```rust
pub async fn revoke_oidc_refresh_token(&self, token_hash: &str) -> Result<bool>
```

Marks a refresh token revoked; returns whether a row was updated.

##### Arguments

* `token_hash` - SHA-256 hex digest of the opaque token.

##### Returns

`Result<bool>` — `Ok` on success.

##### Errors

Returns a crate error when the database operation fails or inputs are invalid.

#### LibraryStore::list_security_audit_events

```rust
pub async fn list_security_audit_events(
    &self,
    limit: u64,
) -> Result<Vec<SecurityAuditEvent>>
```

List recent security audit events (newest first).

##### Errors

Returns an error when the operation fails.

#### LibraryStore::link_account

```rust
pub async fn link_account(
    &self,
    identity_id: i64,
    plugin_key: &str,
    account_id: &str,
    source: &str,
) -> Result<AccountLinkRecord>
```

Link a bookstore account to a portal identity.

Account links are exclusive: one `(plugin_key, account_id)` may belong
to only one identity (`idx_account_links_account_exclusive`).

##### Errors

Returns an error when the operation fails.

#### LibraryStore::list_account_links

```rust
pub async fn list_account_links(&self, identity_id: i64) -> Result<Vec<AccountLinkRecord>>
```

Lists store accounts linked to a portal identity.

##### Arguments

* `identity_id` - Portal identity id.

##### Returns

`Result<Vec<crate::models::AccountLinkRecord>>` — `Ok` on success.

##### Errors

Returns a crate error when the database operation fails or inputs are invalid.

#### LibraryStore::unlink_account

```rust
pub async fn unlink_account(
    &self,
    identity_id: i64,
    plugin_key: &str,
    account_id: &str,
) -> Result<()>
```

Remove an account link row (does not delete the account).

##### Errors

Returns an error when the operation fails.

#### LibraryStore::upsert_book

```rust
pub async fn upsert_book(&self, book: &NewBook) -> Result<AcquireTitle>
```

Records a title an account holds, as a library sync lists it: the
listing it describes, resolved into works, expressions and editions,
the account's holding of it, the listing's store text and the fields
it offers its work.

The listing is derived from `book` ([`listing_from_book`](#listing_from_book)); a scan
that has the plugin's structured record uses
[`Self::upsert_scanned_title`](#librarystoreupsert_scanned_title) instead.

##### Returns

The account's title, as acquire reads it.

##### Errors

Returns [`LibraryError::Invalid`](#libraryerror) when `book` has no plugin key or
fails listing validation, [`LibraryError::NotFound`](#libraryerror) when the
holding was removed again before it could be read back, or the
database error.

#### LibraryStore::update_book_metadata

```rust
pub async fn update_book_metadata(&self, book: &NewBook) -> Result<()>
```

Updates a title's store text and the fields it offers its work,
without asserting that its account holds it.

For metadata from elsewhere (enrichment), which must not bring back
a holding a scan or a revoke removed: the listing is linked as a
catalog listing when it is not stored yet.

##### Errors

Returns [`LibraryError::Invalid`](#libraryerror) when `book` has no plugin key or
fails listing validation, or the database error.

#### LibraryStore::upsert_scanned_title

```rust
pub async fn upsert_scanned_title(&self, title: &ScannedTitle) -> Result<AcquireTitle>
```

Records a scanned title: the listing its plugin's structured record
describes (or, without one, the listing `title.book` describes), the
account's holding of it, its store text and the fields it offers
its work.

The listing is filed under the book's plugin, whatever plugin key it
carries.

##### Returns

The account's title, as acquire reads it.

##### Errors

As [`Self::upsert_book`](#librarystoreupsert_book).

#### LibraryStore::upsert_scanned_titles

```rust
pub async fn upsert_scanned_titles(&self, titles: &[ScannedTitle]) -> Result<usize>
```

Records a page of scanned titles as [`Self::upsert_scanned_title`](#librarystoreupsert_scanned_title)
records each one, with every title's listing resolved as one page
(`record_listing_page_with_details`), so titles the library already
holds as the store describes them cost no reads of their own.

A title that fails listing validation or cannot be resolved is
logged and skipped, so one bad title does not stop the rest of a
scan; the next scan tries it again.

##### Returns

How many of `titles` were recorded.

##### Errors

Returns [`LibraryError::Invalid`](#libraryerror) when a title has no plugin key;
nothing is written then.

#### LibraryStore::held_title

```rust
pub async fn held_title(
    &self,
    title_id: &str,
    account_id: &str,
) -> Result<Option<AcquireTitle>>
```

The title `account_id` holds that `title_id` names: its holding id,
the store's product id, or an ASIN or ISBN of its edition.

Prefers a holding id match, then a product id, then an ASIN, then
an ISBN. When two plugins hold the same account id and both match
at the same priority, the lookup fails closed; pass the plugin key
through [`Self::held_title_in`](#librarystoreheld_title_in) instead.

##### Errors

Returns [`LibraryError::Conflict`](#libraryerror) when the match is ambiguous, or
the database error.

#### LibraryStore::held_title_in

```rust
pub async fn held_title_in(
    &self,
    plugin_key: Option<&str>,
    title_id: &str,
    account_id: &str,
) -> Result<Option<AcquireTitle>>
```

[`Self::held_title`](#librarystoreheld_title) narrowed to one plugin key when `plugin_key` is
set.

##### Errors

Returns [`LibraryError::Conflict`](#libraryerror) when the match is ambiguous, or
the database error.

#### LibraryStore::has_held_titles

```rust
pub async fn has_held_titles(&self) -> Result<bool>
```

True when any account holds a title (SQL `EXISTS`-style read, not a
full fetch).

A peer writer on the same SQLite file (TRUNCATE journal) can make this
read return `SQLITE_BUSY`. The read is repeated until the lock clears
or the attempt budget is spent.

##### Errors

Returns [`LibraryError::Orm`](#libraryerror) when the read fails for a reason other
than file-lock contention, and [`LibraryError::Unavailable`](#libraryerror) when the
lock is still held after the retries.

#### LibraryStore::update_user_fields

```rust
pub async fn update_user_fields(
    &self,
    title_id: &str,
    account_id: &str,
    fields: &UserBookFields,
) -> Result<()>
```

Records user-defined fields (tags, ratings, finished) for every
person linked to the account that holds the title, on the title's
work, without touching scan metadata.

Each person keeps their own state, so members sharing the account
do not overwrite one another's; an account no person is linked to
records nothing.

##### Errors

Returns [`LibraryError::NotFound`](#libraryerror) when the account holds no title
`title_id` names, [`LibraryError::Conflict`](#libraryerror) when the name is
ambiguous, or the database error.

#### LibraryStore::set_pdf_status

```rust
pub async fn set_pdf_status(
    &self,
    title_id: &str,
    account_id: &str,
    status: AcquireStatus,
    pdf_storage_key: Option<&str>,
) -> Result<()>
```

Records the companion PDF state of the title `account_id` holds
that `title_id` names (see [`Self::held_title`](#librarystoreheld_title)), as
[`Self::set_holding_pdf_status`](#librarystoreset_holding_pdf_status) records it for that holding.

##### Arguments

* `title_id` - Holding id, product id, ASIN or ISBN of the title.
* `account_id` - Store account id.
* `status` - The new PDF state.
* `pdf_storage_key` - Object-storage key for the companion PDF.

##### Errors

Returns [`LibraryError::NotFound`](#libraryerror) when the account holds no such
title, [`LibraryError::Conflict`](#libraryerror) when the name is ambiguous, or the
database error.

#### LibraryStore::set_holding_pdf_status

```rust
pub async fn set_holding_pdf_status(
    &self,
    holding_id: &str,
    status: AcquireStatus,
    pdf_storage_key: Option<&str>,
) -> Result<()>
```

Records the companion PDF state account holding `holding_id`
(`holdings.id`) wrote for the edition its listing sells.

Once the PDF is acquired, only the holding that acquired it changes
it while that holding exists.

##### Arguments

* `holding_id` - The account holding that wrote the state.
* `status` - The new PDF state.
* `pdf_storage_key` - Object-storage key for the companion PDF.

##### Errors

Returns [`LibraryError::NotFound`](#libraryerror) when no account holding has that
id, or the database error.

#### LibraryStore::is_ignored

```rust
pub async fn is_ignored(&self, title_id: &str, account_id: &str) -> Result<bool>
```

Returns whether (`title_id`, `account_id`) is on the ignore list.

##### Arguments

* `title_id` - Product id / ASIN / ISBN used as the title key.
* `account_id` - Store account id.

##### Returns

`Result<bool>` — `Ok` on success.

##### Errors

Returns a crate error when the database operation fails or inputs are invalid.

#### LibraryStore::set_ignored

```rust
pub async fn set_ignored(
    &self,
    title_id: &str,
    account_id: &str,
    ignored: bool,
    reason: Option<&str>,
) -> Result<()>
```

Adds or removes a title from the per-account ignore list.

##### Arguments

* `title_id` - Product id / ASIN / ISBN used as the title key.
* `account_id` - Store account id.
* `ignored` - When true, add to the ignore list; when false, remove.
* `reason` - Optional operator note for why the title is ignored.

##### Returns

`Result<()>` — `Ok` on success.

##### Errors

Returns a crate error when the database operation fails or inputs are invalid.

#### LibraryStore::set_acquire_status

```rust
pub async fn set_acquire_status(
    &self,
    title_id: &str,
    account_id: &str,
    status: AcquireStatus,
    storage_key: Option<&str>,
    error_message: Option<&str>,
) -> Result<()>
```

Records the acquire state of the title `account_id` holds that
`title_id` names (see [`Self::held_title`](#librarystoreheld_title)), as
[`Self::set_holding_acquire_status`](#librarystoreset_holding_acquire_status) records it for that holding.

##### Arguments

* `title_id` - Holding id, product id, ASIN or ISBN of the title.
* `account_id` - Store account id.
* `status` - Acquire or request status value.
* `storage_key` - Object-storage key for the primary audio artifact.
* `error_message` - Optional failure message to persist.

##### Errors

Returns [`LibraryError::NotFound`](#libraryerror) when the account holds no such
title (or it stops holding the edition before the write),
[`LibraryError::Conflict`](#libraryerror) when the name is ambiguous, or the
database error.

#### LibraryStore::set_holding_acquire_status

```rust
pub async fn set_holding_acquire_status(
    &self,
    holding_id: &str,
    status: AcquireStatus,
    storage_key: Option<&str>,
    error_message: Option<&str>,
) -> Result<()>
```

Records the acquire state account holding `holding_id`
(`holdings.id`) wrote for the edition its listing sells.

The edition's `edition_acquisitions` row (read with
[`Self::edition_acquisition`](#librarystoreedition_acquisition)) and the state's event, which names
the holding, are written in one batch; an edition another holding
acquired keeps its state while that holding exists.

##### Arguments

* `holding_id` - The account holding that wrote the state.
* `status` - The new acquire state.
* `storage_key` - Object-storage key for the primary audio artifact.
* `error_message` - Optional failure message to persist.

##### Errors

Returns [`LibraryError::NotFound`](#libraryerror) when no account holding has that
id or it no longer holds the edition, or the database error.

#### LibraryStore::bulk_set_acquire_status

```rust
pub async fn bulk_set_acquire_status(
    &self,
    account: Option<&str>,
    asins: &[String],
    status: AcquireStatus,
) -> Result<u32>
```

Bulk-update acquire status (classic `set-status --force`).

Walks every account holding (only `account`'s when set) and writes
`status` for each title, keeping its edition's storage key. When
`asins` is non-empty, only titles whose holding id, product id, ISBN
or ASIN one of them names (ignoring ASCII case) are written.

##### Returns

How many titles were written.

##### Errors

Returns the first read or write error; titles written before it
stay written.

#### LibraryStore::list_saved_filters

```rust
pub async fn list_saved_filters(&self) -> Result<Vec<SavedFilterRecord>>
```

List saved quick-filter expressions.

##### Errors

Returns an error when the operation fails.

#### LibraryStore::upsert_saved_filter

```rust
pub async fn upsert_saved_filter(
    &self,
    name: &str,
    query: &str,
) -> Result<SavedFilterRecord>
```

Inserts or replaces a named saved library filter query.

##### Arguments

* `name` - Logical name (filter, secret, …).
* `query` - Saved filter query string.

##### Returns

`Result<SavedFilterRecord>` — `Ok` on success.

##### Errors

Returns a crate error when the database operation fails or inputs are invalid.

#### LibraryStore::get_saved_filter

```rust
pub async fn get_saved_filter(&self, name: &str) -> Result<Option<SavedFilterRecord>>
```

Loads a saved filter by name, if present.

##### Arguments

* `name` - Logical name (filter, secret, …).

##### Returns

`Result<Option<SavedFilterRecord>>` — `Ok` on success.

##### Errors

Returns a crate error when the database operation fails or inputs are invalid.

#### LibraryStore::delete_saved_filter

```rust
pub async fn delete_saved_filter(&self, name: &str) -> Result<()>
```

Deletes a saved filter by name.

##### Arguments

* `name` - Logical name (filter, secret, …).

##### Returns

`Result<()>` — `Ok` on success.

##### Errors

Returns a crate error when the database operation fails or inputs are invalid.

#### LibraryStore::count_works_by_status

```rust
pub async fn count_works_by_status(&self) -> Result<AcquireStatusCounts>
```

Counts the library's held works per acquire status in one grouped
read of the page index (`library_work_rows`), where each work has the
furthest state any of its editions reached.

##### Returns

The count for each [`AcquireStatus`](#acquirestatus) and the total. A row with a
status this build does not know is counted in the total only.

##### Errors

Returns [`LibraryError::Orm`](#libraryerror) when the read fails for a reason other
than file-lock contention, and [`LibraryError::Unavailable`](#libraryerror) when the
lock is still held after the retries.

#### LibraryStore::upsert_listening_progress

```rust
pub async fn upsert_listening_progress(
    &self,
    row: &NewListeningProgress,
) -> Result<ListeningProgressRecord>
```

Inserts or updates listening progress for a title.

##### Arguments

* `row` - `row` argument for this call.

##### Returns

`Result<ListeningProgressRecord>` — `Ok` on success.

##### Errors

Returns a crate error when the database operation fails or inputs are invalid.

#### LibraryStore::get_listening_progress

```rust
pub async fn get_listening_progress(
    &self,
    provider: &str,
    external_user_id: &str,
    external_item_id: &str,
) -> Result<Option<ListeningProgressRecord>>
```

Loads listening progress for one identity and book.

##### Arguments

* `provider` - External provider id.
* `external_user_id` - User id at the external provider.
* `external_item_id` - `external_item_id` argument for this call.

##### Returns

`Result<Option<ListeningProgressRecord>>` — `Ok` on success.

##### Errors

Returns a crate error when the database operation fails or inputs are invalid.

#### LibraryStore::list_listening_progress

```rust
pub async fn list_listening_progress(
    &self,
    external_user_id: Option<&str>,
) -> Result<Vec<ListeningProgressRecord>>
```

Lists listening progress rows for an identity.

##### Arguments

* `external_user_id` - User id at the external provider.

##### Returns

`Result<Vec<ListeningProgressRecord>>` — `Ok` on success.

##### Errors

Returns a crate error when the database operation fails or inputs are invalid.

#### LibraryStore::create_title_request

```rust
pub async fn create_title_request(
    &self,
    req: &NewTitleRequest,
) -> Result<TitleRequestRecord>
```

Creates a wishlist / title-request row.

##### Arguments

* `req` - Insert / update request payload.

##### Returns

`Result<TitleRequestRecord>` — `Ok` on success.

##### Errors

Returns a crate error when the database operation fails or inputs are invalid.

#### LibraryStore::create_title_request_for

```rust
pub async fn create_title_request_for(
    &self,
    req: &NewTitleRequest,
    work: Option<&WishWork>,
) -> Result<TitleRequestRecord>
```

Creates a wishlist / title-request row for a known work.

A wish that names a store edition the server looked up is for that
listing's work: its resolution in the shared pool, with no holding.
Without one, the wish's text and identifiers find or make the work
([`Self::work_for_wish`](#librarystorework_for_wish)).

##### Arguments

* `req` - Insert / update request payload.
* `work` - The work (and edition) of a pooled listing the wish
  names, or `None` to resolve it from `req`.

##### Returns

The new wish, or the caller's open wish for the same work.

##### Errors

Returns a crate error when the database operation fails or inputs are invalid.

#### LibraryStore::upsert_title_request_sources

```rust
pub async fn upsert_title_request_sources(
    &self,
    title_request_id: i64,
    sources: &[NewTitleRequestSource],
) -> Result<TitleRequestRecord>
```

Upsert per-storefront snapshots for a wishlist row, then return the merged record.

##### Errors

Returns an error when the operation fails.

#### LibraryStore::list_title_request_sources

```rust
pub async fn list_title_request_sources(
    &self,
    title_request_id: i64,
) -> Result<Vec<TitleRequestSourceRecord>>
```

Lists per-storefront snapshots for a title request.

##### Arguments

* `title_request_id` - `title_request_id` argument for this call.

##### Returns

`Result<Vec<TitleRequestSourceRecord>>` — `Ok` on success.

##### Errors

Returns a crate error when the database operation fails or inputs are invalid.

#### LibraryStore::find_open_wishlist

```rust
pub async fn find_open_wishlist(
    &self,
    identity_id: Option<i64>,
    work_id: &str,
) -> Result<Option<TitleRequestRecord>>
```

Open wish of this identity for `work_id`, if any.

##### Arguments

* `identity_id` - The wisher's portal identity; `None` for the operator.
* `work_id` - The work wished for.

##### Errors

Returns [`LibraryError::Orm`](#libraryerror) when a read fails.

#### LibraryStore::list_wishlist

```rust
pub async fn list_wishlist(
    &self,
    identity_id: Option<i64>,
) -> Result<Vec<TitleRequestRecord>>
```

Personal open wishlist for a portal identity, or operator (`identity_id` null).

##### Errors

Returns an error when the operation fails.

#### LibraryStore::list_global_request_queue

```rust
pub async fn list_global_request_queue(&self) -> Result<Vec<GlobalQueueEntry>>
```

Global request queue: open wishes grouped by their work.

Sorted by wish count as a simple default; Discover re-ranks with local
taste plus a heavy per-wisher boost for the Wishlist sidebar.

##### Errors

Returns an error when the operation fails.

#### LibraryStore::get_title_request_by_uuid

```rust
pub async fn get_title_request_by_uuid(
    &self,
    uuid: &str,
) -> Result<Option<TitleRequestRecord>>
```

Loads a wishlist title-request row by UUID, if present.

##### Arguments

* `uuid` - Book UUID.

##### Returns

`Result<Option<TitleRequestRecord>>` — `Ok` on success.

##### Errors

Returns a crate error when the database operation fails or inputs are invalid.

#### LibraryStore::list_title_requests

```rust
pub async fn list_title_requests(
    &self,
    status: Option<RequestStatus>,
) -> Result<Vec<TitleRequestRecord>>
```

Lists wishlist / title-request rows.

##### Arguments

* `status` - Acquire or request status value.

##### Returns

`Result<Vec<TitleRequestRecord>>` — `Ok` on success.

##### Errors

Returns a crate error when the database operation fails or inputs are invalid.

#### LibraryStore::update_title_request_status

```rust
pub async fn update_title_request_status(
    &self,
    uuid: &str,
    status: RequestStatus,
    resolved_edition_id: Option<&str>,
) -> Result<()>
```

Updates the lifecycle status of a title request.

##### Arguments

* `uuid` - Book UUID.
* `status` - Acquire or request status value.
* `resolved_edition_id` - Edition that fulfilled the request, when it is now held.

##### Returns

`Result<()>` — `Ok` on success.

##### Errors

Returns a crate error when the database operation fails or inputs are invalid.

#### LibraryStore::get_user_preferences

```rust
pub async fn get_user_preferences(
    &self,
    subject_key: &str,
) -> Result<Option<UserPreferences>>
```

Load per-user GUI / Discover preferences by subject key.

##### Errors

Returns an error when the operation fails.

#### LibraryStore::get_user_preferences_or_default

```rust
pub async fn get_user_preferences_or_default(
    &self,
    subject_key: &str,
    identity_id: Option<i64>,
) -> Result<UserPreferences>
```

Preferences for `subject_key`, or in-memory defaults when no row exists.

##### Errors

Returns an error when the operation fails.

#### LibraryStore::upsert_user_preferences

```rust
pub async fn upsert_user_preferences(
    &self,
    subject_key: &str,
    identity_id: Option<i64>,
    default_view: &str,
    disabled_shelves: &[String],
    discover_sort: &str,
    discover_sort_dir: &str,
    discover_language: Option<&str>,
    discover_excluded_sources: &[String],
    theme: &str,
) -> Result<UserPreferences>
```

Insert or replace preferences for a subject (operator or portal identity).

##### Errors

Returns an error when the operation fails.

Implements: `Clone`, `Debug`

### LostCopies

```rust
pub struct LostCopies {
    pub restoring: u64,
    pub missing: u64,
    pub not_acquired: u64,
}
```

What [`LibraryStore::lose_copies`](#librarystorelose_copies) did with the stored copies whose file
is gone.

**Fields**

- `restoring`: Copies that turned pending for the copy retry to write again
  (`output.missing_files = restore`).
- `missing`: Copies marked missing (`output.missing_files = mark_missing`).
- `not_acquired`: Editions that turned not acquired because no destination holds a
  stored copy of them any more.

Implements: `Clone`, `Copy`, `Debug`, `Default`, `Eq`, `PartialEq`

### NewBook

```rust
pub struct NewBook {
    pub product_id: String,
    pub plugin_key: String,
    pub source: String,
    pub account_id: String,
    pub asin: Option<String>,
    pub isbn: Option<String>,
    pub marketplace: String,
    pub title: String,
    pub authors: Option<String>,
    pub narrators: Option<String>,
    pub series: Option<String>,
    pub series_index: Option<String>,
    pub series_asin: Option<String>,
    pub purchased_at: Option<DateTime<Utc>>,
    pub publisher: Option<String>,
    pub length_minutes: Option<i64>,
    pub is_abridged: bool,
    pub content_kind: String,
    pub categories: Option<String>,
    pub subtitle: Option<String>,
    pub published_at: Option<DateTime<Utc>>,
}
```

One title as a library sync (or an import) reports it: the account
that holds it, the store listing it describes and the store's text.

**Fields**

- `product_id`: Storefront product id (ASIN, ISBN, UUID, …).
- `plugin_key`: Provenance-qualified PluginKey that owns the account. Forced by the
  scope on write.
- `source`: Display alias of the owning plugin (`audible`, `libro`, …).
- `account_id`: Store or operator account id this row belongs to.
- `asin`: Amazon ASIN when known; otherwise null.
- `isbn`: ISBN-10/13 when known; otherwise null.
- `marketplace`: Store marketplace / locale code (for example `us`, `uk`).
- `title`: Display title of the work or edition.
- `authors`: Comma-separated or JSON author list from the storefront.
- `narrators`: Comma-separated or JSON narrator list when present.
- `series`: Series name when the title belongs to a series.
- `series_index`: Position within the series (storefront string form).
- `series_asin`: Amazon series ASIN when the storefront exposes one.
- `purchased_at`: RFC 3339 purchase time from the storefront, when known.
- `publisher`: Publisher name from metadata enrichment or the storefront.
- `length_minutes`: Runtime in whole minutes when the storefront reports it.
- `is_abridged`: Whether the edition is abridged (0/1 or bool).
- `content_kind`: Title kind: `book`, `episode`, `podcast`, ….
- `categories`: Storefront category / genre path list (serialized).
- `subtitle`: Optional subtitle from bibliographic metadata.
- `published_at`: Publication date string from the storefront or enrichment.

#### NewBook::needs_html_entity_decode

```rust
pub fn needs_html_entity_decode(&self) -> bool
```

True when any human-readable field may contain an HTML entity.

#### NewBook::decode_html_entities

```rust
pub fn decode_html_entities(&mut self)
```

Decode HTML entities in human-readable metadata fields (titles, etc.).

#### NewBook::minimal

```rust
pub fn minimal(
    product_id: impl Into<String>,
    account_id: impl Into<String>,
    marketplace: impl Into<String>,
    title: impl Into<String>,
) -> Self
```

Minimal Audible book row with scan metadata defaults.

Sets `product_id` from the first argument, `asin = Some(product_id)`,
`isbn = None`, `uuid = None` (auto), and `plugin_key = source =
"audible"` (a [`crate::SourceScope`] replaces both on write).

Implements: `Clone`, `Debug`, `From<NewBook>`

### NewListeningProgress

```rust
pub struct NewListeningProgress {
    pub identity_id: Option<i64>,
    pub provider: String,
    pub external_user_id: String,
    pub work_id: String,
    pub edition_id: Option<String>,
    pub external_item_id: String,
    pub progress: Option<f64>,
    pub current_time_seconds: Option<f64>,
    pub duration_seconds: Option<f64>,
    pub is_finished: bool,
    pub last_listened_at: Option<DateTime<Utc>>,
}
```

Input for upserting listening progress.

**Fields**

- `identity_id`: Foreign key to `portal_identities.id`.
- `provider`: External identity or integration provider id (for example ABS).
- `external_user_id`: User id at the external provider.
- `work_id`: The work the item resolved to through its identifiers.
- `edition_id`: The edition the item's identifiers name, when exactly one does.
- `external_item_id`: Provider-native listening-progress item id.
- `progress`: Fractional progress 0.0–1.0 when the provider reports it.
- `current_time_seconds`: Current playback position within the title, in seconds.
- `duration_seconds`: Total duration in seconds when known.
- `is_finished`: Whether the listener marked the title finished (0/1 or bool).
- `last_listened_at`: RFC 3339 time of the last playback update.

Implements: `Clone`, `Debug`

### NewTitleRequest

```rust
pub struct NewTitleRequest {
    pub uuid: Option<String>,
    pub identity_id: Option<i64>,
    pub title: String,
    pub authors: Option<String>,
    pub asin: Option<String>,
    pub isbn: Option<String>,
    pub notes: Option<String>,
    pub status: RequestStatus,
    pub resolved_edition_id: Option<String>,
    pub cover_url: Option<String>,
}
```

Input for creating a title request / wishlist row.

**Fields**

- `uuid`: Stable UUID for this row (API / foreign-key identity).
- `identity_id`: Foreign key to `portal_identities.id`.
- `title`: Title the wisher asked for. It only finds (or makes) the wish's
  work and is not stored on the wish.
- `authors`: Comma-separated authors the wisher gave, used like `title`.
- `asin`: Amazon ASIN the wisher gave, used like `title`.
- `isbn`: ISBN-10/13 the wisher gave, used like `title`.
- `notes`: Free-form operator or requester notes.
- `status`: Lifecycle status for the row (user, request, …).
- `resolved_edition_id`: Edition (`editions.id`) the library holds once the wishlist item is fulfilled.
- `cover_url`: HTTPS URL for cover art when known.

#### NewTitleRequest::needs_html_entity_decode

```rust
pub fn needs_html_entity_decode(&self) -> bool
```

Returns true when the string still contains HTML character entities.

#### NewTitleRequest::decode_html_entities

```rust
pub fn decode_html_entities(&mut self)
```

Decode HTML entities in human-readable fields.

Implements: `Clone`, `Debug`

### NewTitleRequestSource

```rust
pub struct NewTitleRequestSource {
    pub source: String,
    pub product_id: String,
    pub title: Option<String>,
    pub subtitle: Option<String>,
    pub authors: Option<String>,
    pub narrators: Option<String>,
    pub series: Option<String>,
    pub series_index: Option<String>,
    pub asin: Option<String>,
    pub isbn: Option<String>,
    pub description: Option<String>,
    pub publisher: Option<String>,
    pub length_minutes: Option<i64>,
    pub published_at: Option<String>,
    pub categories: Option<String>,
    pub language: Option<String>,
    pub cover_url: Option<String>,
    pub url: Option<String>,
    pub price_cents: Option<i64>,
    pub currency: Option<String>,
    pub price_label: Option<String>,
    pub list_price_cents: Option<i64>,
    pub list_price_label: Option<String>,
    pub member_price_cents: Option<i64>,
    pub member_price_label: Option<String>,
    pub listing_id: Option<String>,
}
```

Input for upserting a per-storefront wishlist snapshot.

**Fields**

- `source`: Content-source plugin id (`audible`, `libro`, …).
- `product_id`: Storefront product id (ASIN, ISBN, UUID, …).
- `title`: Display title of the work or edition.
- `subtitle`: Optional subtitle from bibliographic metadata.
- `authors`: Comma-separated or JSON author list from the storefront.
- `narrators`: Comma-separated or JSON narrator list when present.
- `series`: Series name when the title belongs to a series.
- `series_index`: Position within the series (storefront string form).
- `asin`: Amazon ASIN when known; otherwise null.
- `isbn`: ISBN-10/13 when known; otherwise null.
- `description`: Blurb / synopsis text (may contain HTML).
- `publisher`: Publisher name from metadata enrichment or the storefront.
- `length_minutes`: Runtime in whole minutes when the storefront reports it.
- `published_at`: Publication date string from the storefront or enrichment.
- `categories`: Storefront category / genre path list (serialized).
- `language`: BCP-47 or storefront language code when known.
- `cover_url`: HTTPS URL for cover art when known.
- `url`: Storefront product or purchase URL when known.
- `price_cents`: Observed price in minor currency units, when known.
- `currency`: ISO 4217 currency code for price fields.
- `price_label`: Storefront-formatted price string for display.
- `list_price_cents`: List/MSRP price in minor units, when known.
- `list_price_label`: Storefront-formatted list price for display.
- `member_price_cents`: Member/subscriber price in minor units, when known.
- `member_price_label`: Storefront-formatted member price for display.
- `listing_id`: The shared pool's listing for this store edition, when the server
  looked it up at the store ([`crate::CatalogListing`](#cataloglisting)); `None` keeps
  what the row already names.

#### NewTitleRequestSource::needs_html_entity_decode

```rust
pub fn needs_html_entity_decode(&self) -> bool
```

Returns true when the string still contains HTML character entities.

#### NewTitleRequestSource::decode_html_entities

```rust
pub fn decode_html_entities(&mut self)
```

Decode HTML entities in human-readable metadata fields.

Implements: `Clone`, `Debug`, `Default`

### ProbedAudioFile

```rust
pub struct ProbedAudioFile {
    pub destination: String,
    pub storage_key: String,
    pub role: FileRole,
    pub copy: Option<OutputCopy>,
    pub probe: Option<FileProbe>,
    pub stream: AudioStream,
}
```

One audio file of an edition as [`LibraryStore::edition_audio_files`](#librarystoreedition_audio_files)
reads it back.

**Fields**

- `destination`: Output destination id (`local`, `s3`).
- `storage_key`: Storage key on that destination.
- `role`: Primary audio, a part or a chapter file.
- `copy`: The library copy (`output.copies`) the file belongs to; `None` for
  an upload or a copy this build cannot read.
- `probe`: What the last probe found; `None` until the file has been probed.
- `stream`: What the last probe found about the stream; all `None` until the
  file has been probed, or for a probe from before streams were
  stored.

Implements: `Clone`, `Debug`, `Eq`, `PartialEq`

### QueueRank

```rust
pub struct QueueRank {
    pub work_id: String,
    pub owned: bool,
    pub taste_score: f64,
    pub reasons: Vec<String>,
}
```

The taste ranking of one queue entry, as the ranking job computed it.

**Fields**

- `work_id`: The work (`works.id`) the entry's wishes are for, which identifies
  the entry.
- `owned`: True when the library already holds the entry's work.
- `taste_score`: Household taste score before the wish-count boost. A non-finite
  value is stored as 0.
- `reasons`: Taste reasons; at most [`MAX_QUEUE_REASONS`](#max_queue_reasons) are stored.

Implements: `Clone`, `Debug`, `PartialEq`

### SavedFilterRecord

```rust
pub struct SavedFilterRecord {
    pub id: i64,
    pub name: String,
    pub query: String,
    pub created_at: DateTime<Utc>,
    pub updated_at: DateTime<Utc>,
}
```

Saved Lucene-style quick filter.

**Fields**

- `id`: Surrogate primary key assigned by the database.
- `name`: Human-readable or logical name for this row.
- `query`: Saved library filter expression (UI search DSL).
- `created_at`: RFC 3339 timestamp when the row was inserted.
- `updated_at`: RFC 3339 timestamp when the row was last modified.

Implements: `Clone`, `Debug`

### ScannedTitle

```rust
pub struct ScannedTitle {
    pub book: NewBook,
    pub listing: Option<Listing>,
    pub is_explicit: Option<bool>,
}
```

One title a storefront scan reported.

**Fields**

- `book`: The scan's record of the title: the account, the store text and,
  without [`Self::listing`](#scannedtitle), the listing it describes.
- `listing`: The listing the plugin's structured record describes; `None` when
  only `book` is known.
- `is_explicit`: Whether the store marks the edition explicit (adult) content:
  `Some(true)` explicit, `Some(false)` clean, `None` unknown. Kept on
  the listing (`store_listings.is_explicit`).

Implements: `Clone`, `Debug`, `From<NewBook>`

### SeenObject

```rust
pub struct SeenObject {
    pub key: String,
    pub size: u64,
    pub signature: Option<String>,
    pub modified_at_ms: Option<u64>,
}
```

One object as a listing or a change feed reports it.

**Fields**

- `key`: The object's key at the destination.
- `size`: Object size in bytes.
- `signature`: The backend's opaque change signature, else its entity tag; `None`
  when it gave neither, and then a change of size alone counts as a
  change.
- `modified_at_ms`: Last modification time in UTC unix milliseconds, when known.

Implements: `Clone`, `Debug`, `Eq`, `PartialEq`

### StoreCallHit

```rust
pub struct StoreCallHit {
    pub listing_id: String,
    pub origin: String,
    pub rank_hint: Option<u32>,
}
```

One listing a cached storefront call answered.

**Fields**

- `listing_id`: The `store_listings` row the hit resolved to.
- `origin`: How the store found it for this call.
- `rank_hint`: The store's own 1-based rank for the hit, when it gave one.

Implements: `Clone`, `Debug`, `Eq`, `PartialEq`

### StoredCandidate

```rust
pub struct StoredCandidate {
    pub source: String,
    pub listing: CatalogListingRecord,
    pub seed_scope: SeedScope,
    pub origin: String,
    pub seed_title: Option<String>,
    pub seed_categories: Option<String>,
    pub starter_genre: Option<String>,
    pub fetched_at: DateTime<Utc>,
}
```

One stored storefront candidate of a subject: the shared listing joined
with what is personal to the subject.

**Fields**

- `source`: Storefront plugin id that returned the title (`audible`).
- `listing`: The listing, read from the shared pool.
- `seed_scope`: Seeds that produced the row.
- `origin`: How the store found it.
- `seed_title`: The subject's title that seeded it, when a seed did.
- `seed_categories`: That seed's categories.
- `starter_genre`: Starter genre it was browsed from, for a thin taste.
- `fetched_at`: When the store fetch that produced the row ran.

Implements: `Clone`, `Debug`

### StoredDiscoverTaste

```rust
pub struct StoredDiscoverTaste {
    pub taste_json: String,
    pub seed_scope: SeedScope,
    pub computed_at: DateTime<Utc>,
}
```

One subject's stored Discover taste.

**Fields**

- `taste_json`: The taste as JSON (`bookclerk_discover::StoredTaste`).
- `seed_scope`: Seeds the taste comes from.
- `computed_at`: When the taste was computed.

Implements: `Clone`, `Debug`, `Eq`, `PartialEq`

### StoredFeed

```rust
pub struct StoredFeed {
    pub feed_json: String,
    pub seed_scope: SeedScope,
    pub candidates_fetched_at: Option<DateTime<Utc>>,
    pub computed_at: DateTime<Utc>,
}
```

One subject's stored, ranked Discover feed.

**Fields**

- `feed_json`: The feed as JSON, before hidden shelves are filtered out.
- `seed_scope`: Seeds the feed was ranked from.
- `candidates_fetched_at`: Newest candidate fetch the feed used, when it used any.
- `computed_at`: When the feed was ranked.

Implements: `Clone`, `Debug`

### StoredQueueRank

```rust
pub struct StoredQueueRank {
    pub rank: QueueRank,
    pub ranked_at: DateTime<Utc>,
}
```

One stored queue ranking row.

**Fields**

- `rank`: The ranking.
- `ranked_at`: When the ranking pass that wrote the row ran.

Implements: `Clone`, `Debug`, `PartialEq`

### TitleHolding

```rust
pub struct TitleHolding {
    pub edition_id: String,
    pub holding_id: String,
}
```

The edition a library title sells and the account holding it is held
by: the key its acquire writes are recorded under.

**Fields**

- `edition_id`: The edition the title's listing sells.
- `holding_id`: The account's holding of that listing (`holdings.id`).

Implements: `Clone`, `Debug`, `Eq`, `PartialEq`

### UserBookFields

```rust
pub struct UserBookFields {
    pub tags: Option<String>,
    pub rating_overall: Option<f32>,
    pub rating_performance: Option<f32>,
    pub rating_story: Option<f32>,
    pub is_finished: Option<bool>,
}
```

Partial update for user-defined book fields.

**Fields**

- `tags`: Operator or storefront tags (serialized string).
- `rating_overall`: Overall user rating from the storefront, if any.
- `rating_performance`: Narration/performance rating from the storefront, if any.
- `rating_story`: Story rating from the storefront, if any.
- `is_finished`: Whether the listener marked the title finished (0/1 or bool).

Implements: `Clone`, `Debug`, `Default`

### WishWork

```rust
pub struct WishWork {
    pub work_id: String,
    pub edition_id: Option<String>,
}
```

The work a wish is for, read from a store listing in the shared pool.

**Fields**

- `work_id`: `works.id` of the listing's resolution.
- `edition_id`: `editions.id` the listing is an edition of.

Implements: `Clone`, `Debug`, `Eq`, `PartialEq`

### NewVector

```rust
pub struct NewVector<'a> {
    pub target_id: &'a str,
    pub vector: &'a [u8],
    pub text_hash: &'a str,
}
```

One vector to store.

**Fields**

- `target_id`: Id of the target within its kind.
- `vector`: The vector as little-endian `f32`s.
- `text_hash`: SHA-256 hex of the text that was embedded.

Implements: `Clone`, `Copy`, `Debug`

### SpaceCoverage

```rust
pub struct SpaceCoverage {
    pub space: VectorSpace,
    pub works: u64,
    pub newest: Option<DateTime<Utc>>,
}
```

How many work vectors one space holds.

**Fields**

- `space`: The space.
- `works`: Work vectors stored in it.
- `newest`: When the newest of them was written.

Implements: `Clone`, `Debug`, `Eq`, `PartialEq`

### VectorSpace

```rust
pub struct VectorSpace {
    pub model: String,
    pub revision: String,
    pub packs: String,
}
```

What made a set of vectors. Vectors of two different spaces are never
compared.

**Fields**

- `model`: What made the vectors: `@<plugin id>/<model id>` for a model a
  plugin serves, or the stand-in's own id.
- `revision`: The model's digest of everything that decides its output (weights,
  tokenizer, language packs); empty for the stand-in.
- `packs`: BCP 47 tags of the languages the model was loaded for, joined by
  `,`. Recorded beside the vectors; the revision already covers it,
  so it takes no part in comparing two spaces.

#### VectorSpace::same_as

```rust
pub fn same_as(&self, other: &Self) -> bool
```

Whether `other` is this space: the same model at the same revision.

Implements: `Clone`, `Debug`, `Eq`, `Hash`, `PartialEq`

### NotifyCursor

```rust
pub struct NotifyCursor {
    pub created_at: String,
    pub id: String,
}
```

Where a page of the delivery log or an inbox starts: the last row of the
previous page (newest first).

**Fields**

- `created_at`: RFC 3339 creation time of the last row read.
- `id`: Id of the last row read (the notification id for an inbox).

#### NotifyCursor::encode

```rust
pub fn encode(&self) -> String
```

The opaque token a page answer carries.

#### NotifyCursor::decode

```rust
pub fn decode(token: &str) -> Result<Self>
```

Reads a token made by [`Self::encode`](#notifycursorencode).

##### Errors

[`LibraryError::Invalid`](#libraryerror) when the token is not one: not base64url,
not the JSON shape, or a field over 128 bytes.

Implements: `Clone`, `Debug`, `Deserialize<'de>`, `Eq`, `PartialEq`, `Serialize`

### NotifyPage

```rust
pub struct NotifyPage<T> {
    pub items: Vec<T>,
    pub more: bool,
}
```

One page of a newest-first listing.

**Fields**

- `items`: The page's rows, at most the page size asked for.
- `more`: Whether at least one row follows the last one: only then does the
  listing hand out a cursor, so a full final page ends the walk.

Implements: `Clone`, `Debug`, `Eq`, `PartialEq`

### NotifyQueueLimits

```rust
pub struct NotifyQueueLimits {
    pub max_pending: i64,
    pub max_attempts: i64,
}
```

Queue limits a `notify_send` admission uses (the daemon's `[jobs]`).

**Fields**

- `max_pending`: Global cap on pending and running jobs.
- `max_attempts`: Claims before a failing send job stops.

Implements: `Clone`, `Copy`, `Debug`, `Eq`, `PartialEq`

## Enums

### DbAtomicParams

```rust
pub enum DbAtomicParams {
    DeleteUser { user_id: i64 },
    SetUserStatus { user_id: i64, status: String },
    SetUserPasswordHash { user_id: i64, password_hash: Option<String> },
    SetUserRole { user_id: i64, role: String },
    RedeemClaimTicket { token_hash: String, session_hash: String, expires_at: String, user_agent: Option<String>, device_type: Option<String>, client_label: Option<String>, new_password_hash: Option<String>, password_fingerprint: Option<String> },
    TakeOidcRpState { state_hash: String },
    TakeWebauthnChallenge { challenge_id: String, kind: String },
    EnqueueJob { kind: String, payload_json: String, priority: i64, max_attempts: i64, max_pending: i64, run_after: Option<String> },
    ClaimNextJob { resource_class: String, owner: String, lease_secs: i64 },
    ReserveJobTemp { job_id: String, path: String, reserved_bytes: i64, quota_bytes: i64 },
    ConfirmTotpEnrollment { user_id: i64, format: String, ciphertext: String, cipher_algorithm: Option<String>, cipher_nonce: Option<String>, kdf_algorithm: Option<String>, kdf_salt: Option<String>, kdf_m_cost: Option<i64>, kdf_t_cost: Option<i64>, kdf_p_cost: Option<i64>, created_at: String },
    DisableUserTotp { user_id: i64 },
    PublishDomainEvent { id: String, event_type: String, schema_version: i64, account_id: String, source: String, correlation_id: String, causation_id: String, dedup_key: String, payload: String, ordering_key: String, queue_name: String, content_type: String, deliver_after: String },
    SetAcquireStatus { holding_id: String, edition_id: String, status: String, storage_key: Option<String>, error_message: Option<String>, event_id: String, event_type: String, schema_version: i64, event_account_id: String, source: String, correlation_id: String, causation_id: String, dedup_key: String, payload: String, ordering_key: String },
    DispatchEventDeliveries { event_id: String, subscribers_json: String, mark_dispatched: bool },
    ClaimNextEventDelivery { owner: String, lease_secs: i64, plugin_ids_json: String, max_in_flight: i64 },
    LinkListing(Box<ListingLinkParams>),
    WriteCredits(Box<CreditBatchParams>),
}
```

Named atomic library operation compiled by the host into a generic plan.

**Variants**

- `DeleteUser`: Delete a first-party user and personal data (last-owner guarded).
  - `user_id`: `users.id` to delete.
- `SetUserStatus`: Set `users.status` (`active` / `disabled`; last-owner guarded).
  - `user_id`: `users.id` to update.
  - `status`: Canonical status string (`active` or `disabled`).
- `SetUserPasswordHash`: Set or clear the Argon2id password hash and bump `security_version`.
  - `user_id`: `users.id` to update.
  - `password_hash`: New hash, or `null` to clear.
- `SetUserRole`: Set `users.role` (last-owner guarded on demotion).
  - `user_id`: `users.id` to update.
  - `role`: Canonical role string (`owner` / `administrator` / `member`).
- `RedeemClaimTicket`: Consume a claim ticket, optionally set a first password, mint a session.
  - `token_hash`: SHA-256 hex digest of the claim ticket.
  - `session_hash`: SHA-256 hex digest of the new portal session token.
  - `expires_at`: RFC 3339 expiry for the minted session.
  - `user_agent`: Raw User-Agent captured at session mint.
  - `device_type`: Best-effort device class.
  - `client_label`: Best-effort OS / client label.
  - `new_password_hash`: Argon2id hash to set when the local user has none.
  - `password_fingerprint`: Domain-separated HMAC of the invite password for idempotency.
- `TakeOidcRpState`: Consume a one-time OIDC RP state (`DELETE` + expiry check).
  - `state_hash`: SHA-256 hex digest of the OAuth `state` parameter.
- `TakeWebauthnChallenge`: Consume a one-time WebAuthn challenge (`DELETE` + expiry check).
  - `challenge_id`: Public ceremony id returned to the browser.
  - `kind`: `register`, `login`, or `elevate`.
- `EnqueueJob`: Admit a durable job (dedup + pending cap + insert).
  - `kind`: Job kind wire string (`scan`, `acquire`, `listen_sync`, `integration_scan`).
  - `payload_json`: Versioned command envelope JSON.
  - `priority`: Higher values are claimed first.
  - `max_attempts`: Maximum claims before a failure is terminal.
  - `max_pending`: Global cap on pending+running rows.
  - `run_after`: Optional RFC 3339 delay before the job becomes claimable.
- `ClaimNextJob`: Claim the next ready job in `resource_class` for `owner`.
  - `resource_class`: Resource class wire string (`network`, …).
  - `owner`: Worker id stored as `lease_owner`.
  - `lease_secs`: Lease length in seconds.
- `ReserveJobTemp`: Reserve scratch-quota bytes for one job path.
  - `job_id`: Owning job id.
  - `path`: Absolute filesystem path.
  - `reserved_bytes`: Bytes to reserve for this path.
  - `quota_bytes`: Global reserved-bytes cap.
- `ConfirmTotpEnrollment`: Promote a sealed TOTP secret to `primary` and set `users.totp_enabled`.
  - `user_id`: `users.id` to enroll.
  - `format`: Payload format (`sealed-v1`).
  - `ciphertext`: Sealed TOTP secret bytes (`b64:…`).
  - `cipher_algorithm`: Cipher algorithm identifier, if any.
  - `cipher_nonce`: AEAD nonce (`b64:…`), if any.
  - `kdf_algorithm`: Legacy KDF algorithm, if any.
  - `kdf_salt`: Legacy KDF salt (`b64:…`), if any.
  - `kdf_m_cost`: Legacy Argon2 memory cost in KiB, if any.
  - `kdf_t_cost`: Legacy Argon2 time cost, if any.
  - `kdf_p_cost`: Legacy Argon2 parallelism, if any.
  - `created_at`: RFC 3339 timestamp to store as `created_at` on the primary row.
- `DisableUserTotp`: Delete TOTP secrets and clear `users.totp_enabled`.
  - `user_id`: `users.id` to disable.
- `PublishDomainEvent`: Persist a domain event in the outbox (dedup on eventType+dedupKey).
  - `id`: Stable event id (UUID).
  - `event_type`: Event type (`book_acquired`).
  - `schema_version`: Payload schema version.
  - `account_id`: Tenant / account id.
  - `source`: Producer plugin id; empty when unknown.
  - `correlation_id`: Trace correlation id.
  - `causation_id`: Causing event or job id.
  - `dedup_key`: Unique with `eventType`.
  - `payload`: Bounded JSON payload.
  - `ordering_key`: FIFO key copied onto deliveries.
  - `queue_name`: Queue name; empty means the domain-event bus (`core.events`).
  - `content_type`: Body encoding (`json`, `text`, `bytes`); empty means `json`.
  - `deliver_after`: RFC 3339 earliest delivery time; empty means now.
- `SetAcquireStatus`: Write an edition's acquire state for the account holding that made
  the write, and publish its event in the same batch.
  - `holding_id`: The account holding (`holdings.id`) that wrote the state; every
    statement is gated on it still holding `edition_id`.
  - `edition_id`: The edition whose `edition_acquisitions` row is written.
  - `status`: Acquire status string (`acquired`, `downloading`, …).
  - `storage_key`: Object-storage key for the primary audio artifact.
  - `error_message`: Optional failure message.
  - `event_id`: Event id when publishing; empty skips the outbox insert.
  - `event_type`: Event type when publishing (`book_acquired`).
  - `schema_version`: Payload schema version.
  - `event_account_id`: Tenant / account id on the event.
  - `source`: Producer plugin id; empty when unknown.
  - `correlation_id`: Trace correlation id.
  - `causation_id`: Causing event or job id.
  - `dedup_key`: Unique with `eventType`.
  - `payload`: Bounded JSON payload.
  - `ordering_key`: FIFO key copied onto deliveries.
- `DispatchEventDeliveries`: Create per-subscriber deliveries and mark the event dispatched.
  - `event_id`: Parent event id.
  - `subscribers_json`: JSON array of `{ "pluginId": "…" }` subscriber snapshots.
  - `mark_dispatched`: When true, this page is the last snapshot and may mark the parent dispatched.
- `ClaimNextEventDelivery`: Claim the next ready event delivery for `owner`.
  - `owner`: Worker id stored as `lease_owner`.
  - `lease_secs`: Lease length in seconds.
  - `plugin_ids_json`: JSON array of plugin ids this worker can execute (`[]` claims nothing).
  - `max_in_flight`: Library-wide max `running` deliveries per `(plugin_id, resource_class)`.
- `LinkListing`: Write one resolved store listing: new work, expression and edition,
  identifiers, the listing, its holding, conflicts and events.
  
  Answers `retry` when another writer stored the listing or claimed a
  strong identifier first.
- `WriteCredits`: Write contributors, credits, series and series entries (idempotent).

Implements: `Clone`, `Debug`, `Deserialize<'de>`, `PartialEq`, `Serialize`

### CanonicalObject

```rust
pub enum CanonicalObject {
    Schema { sql_contract_version: u32, statements: Vec<String> },
    TableChunk { table: String, columns: Vec<String>, rows: Vec<Vec<DbValue>> },
    Identity { entries: BTreeMap<String, IdentityHighWater> },
}
```

One immutable canonical object (hashed before compression).

**Variants**

- `Schema`: Admitted `CREATE TABLE` / `CREATE INDEX` in restore order.
  - `sql_contract_version`: Bookclerk SQL grammar/ABI contract.
  - `statements`: Canonical statements (tables in FK-safe order, then indexes).
- `TableChunk`: Bounded table rows in canonical order.
  - `table`: Folded table name.
  - `columns`: Column names in CREATE declaration order.
  - `rows`: Row-major `DbValue` cells.
- `Identity`: Identity high-water for generated columns.
  - `entries`: Folded table name → high-water.

Implements: `Clone`, `Debug`, `Deserialize<'de>`, `PartialEq`, `Serialize`

### BackupReason

```rust
pub enum BackupReason {
    Manual,
    PreMigrate,
}
```

Why a backup was written.

**Variants**

- `Manual`: Operator-created (`bookclerk db backup create`). Never pruned automatically.
- `PreMigrate`: Written immediately before a frozen schema walk.

Implements: `Clone`, `Copy`, `Debug`, `Default`, `Deserialize<'de>`, `Eq`, `PartialEq`, `Serialize`

### BackupResolve

```rust
pub enum BackupResolve {
    Id(String),
    Archive(PathBuf),
}
```

How [`resolve_backup_spec`](#resolve_backup_spec) located a recovery point.

**Variants**

- `Id`: Catalog id under `backups/manifests/`.
- `Archive`: `.tar.gz` archive of one recovery point.

Implements: `Clone`, `Debug`

### CanonicalRestoreKind

```rust
pub enum CanonicalRestoreKind {
    Library,
    PluginBinding,
}
```

How restore applies DDL companions.

**Variants**

- `Library`: Host library: Postgres identity companions only (no plugin catalog).
- `PluginBinding`: Plugin binding: catalog + Postgres identity companions.

Implements: `Clone`, `Copy`, `Debug`, `Eq`, `PartialEq`

### DatabaseUnitKind

```rust
pub enum DatabaseUnitKind {
    Library,
    PluginBinding,
}
```

Logical kind of one database unit in a recovery point.

**Variants**

- `Library`: Host library database.
- `PluginBinding`: Plugin-owned binding (`plugin_id` + `binding`).

Implements: `Clone`, `Copy`, `Debug`, `Deserialize<'de>`, `Eq`, `PartialEq`, `Serialize`

### EntityKind

```rust
pub enum EntityKind {
    Account,
    Book,
    Work,
    Edition,
    Listing,
    TitleRequest,
    User,
    PortalIdentity,
    OperatorSession,
    Passkey,
    OidcState,
    WebauthnChallenge,
    Node,
    EnrollmentLink,
    PendingJoin,
    LibraryIdentity,
    LibraryLink,
    LibraryNodeKeys,
    ClerkLink,
    ClerkCard,
    Configuration,
    PluginInstance,
    PluginDeployment,
    Placement,
    PluginVersion,
    VersionPointer,
    SettingsSchema,
    RemoteConsent,
    BranchRequest,
    DomainEvent,
    EventDelivery,
    Job,
    SavedFilter,
    ClaimTicket,
    NotifyDestination,
    Notification,
    MetadataClaim,
}
```

The kind of thing a [`LibraryError::NotFound`](#libraryerror) could not find.

Each variant displays as the lower-case phrase an error message uses,
such as `enrollment link`.

**Variants**

- `Account`: A store account, keyed by plugin and account id.
- `Book`: A library title: an account's holding of a store listing.
- `Work`: A work of the library data model.
- `Edition`: An edition of the library data model.
- `Listing`: A store listing of the library data model (`store_listings`).
- `TitleRequest`: A title request (wishlist entry).
- `User`: A user.
- `PortalIdentity`: A portal identity linked to a user.
- `OperatorSession`: An operator session.
- `Passkey`: A registered passkey.
- `OidcState`: An OpenID Connect relying-party sign-in state.
- `WebauthnChallenge`: A WebAuthn ceremony challenge.
- `Node`: A library node.
- `EnrollmentLink`: A node enrollment link.
- `PendingJoin`: A pending join of this host to another library.
- `LibraryIdentity`: This library's identity.
- `LibraryLink`: A link to another library.
- `LibraryNodeKeys`: The node keys a linked library published.
- `ClerkLink`: A library this install holds a library card for, as a clerk.
- `ClerkCard`: The sealed library card of a clerk link.
- `Configuration`: A configuration document, by namespace.
- `PluginInstance`: A plugin instance.
- `PluginDeployment`: A plugin deployment.
- `Placement`: A plugin instance's scheduler placement.
- `PluginVersion`: An installed plugin version.
- `VersionPointer`: A plugin's active version pointer.
- `SettingsSchema`: A plugin's pinned settings schema.
- `RemoteConsent`: A remote plugin consent.
- `BranchRequest`: A request from a linked library.
- `DomainEvent`: A domain event on the outbox.
- `EventDelivery`: An event delivery.
- `Job`: A job.
- `SavedFilter`: A saved library filter.
- `ClaimTicket`: A claim ticket.
- `NotifyDestination`: A notification destination.
- `Notification`: A notification (in an inbox or the delivery log).
- `MetadataClaim`: One source's metadata claim about a library subject.

#### EntityKind::ALL

```rust
pub const ALL: [Self; 37] = _
```

Every kind, in declaration order.

#### EntityKind::as_str

```rust
pub const fn as_str(self) -> &'static str
```

The phrase an error message names this kind with.

Implements: `Clone`, `Copy`, `Debug`, `Display`, `Eq`, `Hash`, `PartialEq`

### LibraryError

```rust
pub enum LibraryError {
    Schema(String),
    Orm(DbErr),
    Config(ConfigError),
    Io(Error),
    NotFound(Missing),
    LastAdministrator,
    LastOwner,
    InvalidEmail,
    Unavailable(String),
    Conflict(String),
    Forbidden(String),
    Invalid(String),
    LimitExceeded(LimitExceeded),
    Other(Error),
}
```

Failures from library store, schema apply, secrets, or config.

**Variants**

- `Schema`: Schema versioning, backup, or canonical SQL failure.
- `Orm`: SeaORM / database-plugin failure (`DbErr`).
- `Config`: Configuration load or validation failure.
- `Io`: Filesystem or other I/O failure.
- `NotFound`: A requested row was not found; the payload names what and which.
- `LastAdministrator`: Demoting or disabling the last active administrator is refused.
- `LastOwner`: Demoting, disabling, or deleting the last active owner is refused.
- `InvalidEmail`: Contact email failed library-backed syntax validation.
- `Unavailable`: Backend temporarily unreachable or an atomic RPC response was lost.
  
  Callers that still hold the original consume-once / session token should
  retry the same atomic operation id rather than minting a new one.
- `Conflict`: Identity or ownership conflict (for example a taken OIDC `client_id`).
- `Forbidden`: The caller may not perform this write (for example a configuration
  write without operator authority).
- `Invalid`: The request or stored document failed validation.
- `LimitExceeded`: A request or its result exceeded a negotiated SQL-v1 cap
  (`maxResultBytes`, `maxResultRows`, `maxRequestBytes`, ...).
  
  Not retryable as is: the caller narrows the request (fewer rows, fewer
  statements) or switches to a transport that fits.
- `Other`: Catch-all for otherwise unclassified failures.

#### LibraryError::not_found

```rust
pub fn not_found(entity: EntityKind, id: impl Display) -> Self
```

A [`Self::NotFound`](#libraryerror) for `entity` with the id `id`.

##### Arguments

* `entity` - The kind of row that is missing.
* `id` - The id that was asked for, as the caller names it.

#### LibraryError::missing

```rust
pub const fn missing(entity: EntityKind) -> Self
```

A [`Self::NotFound`](#libraryerror) for `entity` without an id (a singleton, or a
lookup by something other than an id).

#### LibraryError::from_db_err

```rust
pub fn from_db_err(err: DbErr) -> Self
```

Maps a SeaORM / adapter [`sea_orm::DbErr`](https://docs.rs/sea_orm/latest/sea_orm/error/enum.DbErr.html) onto a typed library error.

Busy, deadlock, and lost-commit tokens become [`Self::Unavailable`](#libraryerror).
Unique/duplicate-object tokens become [`Self::Conflict`](#libraryerror). Foreign-key,
CHECK, and NOT NULL stay [`Self::Orm`](#libraryerror). Everything else stays [`Self::Orm`](#libraryerror).

#### LibraryError::from_abi_err

```rust
pub fn from_abi_err(err: PluginError) -> Self
```

Maps an ABI [`bookclerk_plugin_abi::PluginError`](../bookclerk_plugin_abi/error.md#pluginerror) from an adapter or
admission check onto a typed library error.

A `limit_exceeded` error becomes [`Self::LimitExceeded`](#libraryerror); everything
else is [`Self::Other`](#libraryerror) with the same message.

#### LibraryError::limit

```rust
pub fn limit(&self) -> Option<LimitExceeded>
```

The exceeded cap when this error is (or wraps the message of) a
SQL-v1 limit error.

Matches [`Self::LimitExceeded`](#libraryerror) directly and parses the canonical
`limit exceeded: <name> is <cap>; used <n>` text out of
[`Self::Orm`](#libraryerror), [`Self::Unavailable`](#libraryerror) and [`Self::Other`](#libraryerror), so a limit
error survives a string hop through the adapter boundary.

#### LibraryError::is_schema_apply_retryable

```rust
pub fn is_schema_apply_retryable(&self) -> bool
```

True when host schema apply may retry after re-reading durable state.

Implements: `Debug`, `Display`, `Error`, `From<ConfigError>`, `From<DbErr>`, `From<Error>`, `From<LibraryError>`

### AcquireStatus

```rust
pub enum AcquireStatus {
    NotAcquired,
    Queued,
    Downloading,
    Acquired,
    Error,
}
```

Acquire / download state for a title.

**Variants**

- `NotAcquired`: Title has not been queued or downloaded yet.
- `Queued`: Title is waiting in the acquire queue.
- `Downloading`: Acquire pipeline is actively downloading this title.
- `Acquired`: Primary audio artifact is stored and library status is liberated.
- `Error`: Last acquire attempt failed; see the edition's `error_message`.

#### AcquireStatus::ALL

```rust
pub const ALL: [Self; 5] = _
```

Every status, in pipeline order (the `edition_acquisitions.status`
CHECK lists the same strings).

#### AcquireStatus::as_str

```rust
pub fn as_str(self) -> &'static str
```

Returns the canonical snake_case / lowercase wire string.

#### AcquireStatus::parse

```rust
pub fn parse(s: &str) -> Option<Self>
```

Parses the canonical wire string; returns `None` when unknown.

#### AcquireStatus::to_classic

```rust
pub fn to_classic(self) -> i32
```

Classic EF `LiberatedStatus` integer.

#### AcquireStatus::from_classic

```rust
pub fn from_classic(status: i64) -> Self
```

Parse classic EF `LiberatedStatus` integer.

Implements: `Clone`, `ComposeSchema`, `Copy`, `Debug`, `Default`, `Deserialize<'de>`, `Eq`, `PartialEq`, `Serialize`, `ToSchema`

### EnqueueOutcome

```rust
pub enum EnqueueOutcome {
    Created { id: String },
    Duplicate { existing_id: String },
    QueueFull,
}
```

Result of admitting a job into the durable queue.

**Variants**

- `Created`: A new row was inserted.
  - `id`: New job id.
- `Duplicate`: An equivalent pending/running job already exists.
  - `existing_id`: Existing job id that satisfies the dedup key.
- `QueueFull`: `pending + running` already equals `max_pending`.

Implements: `Clone`, `Debug`, `Eq`, `PartialEq`

### EventStreamStart

```rust
pub enum EventStreamStart {
    Oldest,
    Since(DateTime<Utc>),
    After { created_at: String, id: String },
}
```

Where one event stream read starts.

**Variants**

- `Oldest`: The oldest retained event.
- `Since`: The first event stored at or after this time.
- `After`: The first event after this stored `(created_at, id)` position.
  - `created_at`: Stored `created_at` text of the last row already read.
  - `id`: Event id of the last row already read.

Implements: `Clone`, `Debug`, `Eq`, `PartialEq`

### JobKind

```rust
pub enum JobKind {
    Scan,
    Acquire,
    ListenSync,
    IntegrationScan,
    PluginCopy,
    PluginScheduled,
    DiscoverCandidates,
    DiscoverFeed,
    DiscoverRequestQueue,
    DiscoverEmbed,
    CacheSweep,
    SearchIndexRebuild,
    LibraryIndexRebuild,
    BranchRequest,
    IngestBatch,
    NotifySend,
    ChapterAlign,
    Invalid,
}
```

Kind of durable daemon work stored in the `jobs` table.

**Variants**

- `Scan`: Library scan for one account or every scan-enabled account.
- `Acquire`: Acquire pending titles (optional title / account filter).
- `ListenSync`: Sync listening progress from capable integrations.
- `IntegrationScan`: Remote integration library scan (Audiobookshelf, …).
- `PluginCopy`: ABI `JobRunner` stream-copy vertical slice (`plugin_copy`).
- `PluginScheduled`: One fire of a plugin's `[triggers] crons` schedule
  (`plugin_scheduled`), delivered to its `scheduled(controller)`
  handler on a node that runs the instance.
- `DiscoverCandidates`: Fetch storefront candidates from one Discover subject's own seeds
  (`discover_candidates`), store by store.
- `DiscoverFeed`: Rank one Discover subject's stored candidates into shelves
  (`discover_feed`); no network.
- `DiscoverRequestQueue`: Rank the shared request queue against household taste and store the
  ranking (`discover_request_queue`); no network. One job serves
  everyone, since the queue is not personalized.
- `DiscoverEmbed`: Embed every work with the model one node serves for text embeddings
  (`discover_embed`), after the model lands there or changes; only
  the node named by [`JobPayload::node_id`](#jobpayload) claims it, since another
  node may not have the model yet.
- `CacheSweep`: Sweep one node's image caches and its orphaned job scratch
  (`cache_sweep`); only the node named by [`JobPayload::node_id`](#jobpayload)
  claims it.
- `SearchIndexRebuild`: Rebuild one node's full-text search index from the library
  (`search_index_rebuild`); only the node named by
  [`JobPayload::node_id`](#jobpayload) claims it.
- `LibraryIndexRebuild`: Rebuild the library's page index (`library_work_rows`) from the
  library model (`library_index_rebuild`); one for the whole library,
  any node claims it.
- `BranchRequest`: An interlibrary request (`branch_request`): send this library a
  book one of its branches listed. Pinned by
  [`JobPayload::node_id`](#jobpayload) to that branch, which claims it over the
  node socket (`requests.claim`) and never through the plugin job
  path; no node of this library runs it.
- `IngestBatch`: Identify and store one ingest batch (`ingest_batch`): uploaded files
  or storage references, held by the batch's patron. Pinned by
  [`JobPayload::node_id`](#jobpayload) to the node whose disk holds the staged
  bytes, and run only by a node that holds the library database.
- `NotifySend`: Send one notification to one destination (`notify_send`): the
  delivery-log row [`JobPayload::notify_delivery`](#jobpayload) names, through its
  notifier instance ([`JobPayload::plugin_instance_id`](#jobpayload), also in
  [`JobPayload::requires`](#jobpayload)), so only a node running that instance
  claims it.
- `ChapterAlign`: Align a metadata provider's chapter layout to one edition's stored
  copy (`chapter_align`): the edition's id is in [`JobPayload::title`](#jobpayload).
  Any node with a destination holding the copy claims it, since it
  reads the copy's audio from there; it runs only on a node that
  holds the library database.
- `Invalid`: Terminal placeholder after an unreadable persisted command is rejected.

#### JobKind::ALL

```rust
pub const ALL: [Self; 18] = _
```

Every kind, in declaration order; the SQL plans derive their kind
lists from it so a new kind cannot be left unclaimable.

#### JobKind::as_str

```rust
pub fn as_str(self) -> &'static str
```

Returns the canonical snake_case wire string.

#### JobKind::parse

```rust
pub fn parse(s: &str) -> Option<Self>
```

Parses the canonical wire string; returns `None` when unknown.

#### JobKind::resource_class

```rust
pub fn resource_class(self) -> JobResourceClass
```

Resource class used for admission and worker concurrency.

#### JobKind::dedup_key

```rust
pub fn dedup_key(self, payload: &JobPayload) -> String
```

Idempotency key unique among pending/running jobs of the same work.

Implements: `Clone`, `ComposeSchema`, `Copy`, `Debug`, `Deserialize<'de>`, `Eq`, `PartialEq`, `Serialize`, `ToSchema`

### JobResourceClass

```rust
pub enum JobResourceClass {
    Network,
    Media,
    Transcription,
    Indexing,
}
```

Concurrency class that bounds how many jobs of this type run at once.

**Variants**

- `Network`: Store HTTP / scan / acquire / listening-sync work.
- `Media`: Codec / remux work (reserved; media pool remains separate).
- `Transcription`: Speech-to-text / derived transcript work (reserved for #121).
- `Indexing`: Search-index / embedding work (reserved).

#### JobResourceClass::as_str

```rust
pub fn as_str(self) -> &'static str
```

Returns the canonical snake_case wire string.

#### JobResourceClass::parse

```rust
pub fn parse(s: &str) -> Option<Self>
```

Parses the canonical wire string; returns `None` when unknown.

#### JobResourceClass::ALL

```rust
pub const ALL: &'static [&'static str] = _
```

Canonical wire strings for every known class.

Implements: `Clone`, `Copy`, `Debug`, `Deserialize<'de>`, `Eq`, `PartialEq`, `Serialize`

### JobState

```rust
pub enum JobState {
    Pending,
    Running,
    Succeeded,
    Failed,
    Cancelled,
}
```

Lifecycle state of a durable job row.

**Variants**

- `Pending`: Admitted and waiting to be claimed.
- `Running`: Claimed by a worker that holds a live lease.
- `Succeeded`: Finished successfully.
- `Failed`: Exhausted retries or failed without retry.
- `Cancelled`: Cancelled by an operator before or during execution.

#### JobState::as_str

```rust
pub fn as_str(self) -> &'static str
```

Returns the canonical snake_case wire string.

#### JobState::parse

```rust
pub fn parse(s: &str) -> Option<Self>
```

Parses the canonical wire string; returns `None` when unknown.

#### JobState::is_active

```rust
pub fn is_active(self) -> bool
```

True when the job may still be claimed or is executing.

#### JobState::is_terminal

```rust
pub fn is_terminal(self) -> bool
```

True when the job will not run again.

Implements: `Clone`, `ComposeSchema`, `Copy`, `Debug`, `Deserialize<'de>`, `Eq`, `PartialEq`, `Serialize`, `ToSchema`

### JobTrigger

```rust
pub enum JobTrigger {
    Api,
    Scheduler,
}
```

Who admitted the job (API vs periodic scheduler).

**Variants**

- `Api`: Enqueued from the HTTP control plane, tray, or CLI daemon commands.
- `Scheduler`: Enqueued by the daemon interval scheduler.

#### JobTrigger::as_str

```rust
pub fn as_str(self) -> &'static str
```

Returns the canonical snake_case wire string.

#### JobTrigger::parse

```rust
pub fn parse(s: &str) -> Option<Self>
```

Parses the canonical wire string; returns `None` when unknown.

Implements: `Clone`, `Copy`, `Debug`, `Default`, `Deserialize<'de>`, `Eq`, `PartialEq`, `Serialize`

### PortalSessionKind

```rust
pub enum PortalSessionKind {
    Browser,
    Clerk,
    Player,
}
```

Where a user's session token is presented.

A token works only in its own place: a browser token only as the session
cookie, a library card only as `Authorization: Bookclerk-Clerk`, and a
player card's tokens only on the `/player` API.

**Variants**

- `Browser`: A browser's session cookie.
- `Clerk`: A library card: a clerk that shows the user's books from this
  library without joining it as a node.
- `Player`: A player library card: an audiobook player app signed in to the
  `/player` API. Its tokens live in `player_card_tokens`; the row's
  own token hash is never handed out.

#### PortalSessionKind::as_str

```rust
pub fn as_str(self) -> &'static str
```

Returns the column value.

#### PortalSessionKind::parse

```rust
pub fn parse(s: &str) -> Option<Self>
```

Parses the column value; returns `None` when unknown.

Implements: `Clone`, `ComposeSchema`, `Copy`, `Debug`, `Deserialize<'de>`, `Eq`, `PartialEq`, `Serialize`, `ToSchema`

### PublishDomainEventOutcome

```rust
pub enum PublishDomainEventOutcome {
    Created { id: String },
    Duplicate { existing_id: String },
}
```

Result of [`crate::LibraryStore::publish_domain_event`](#librarystorepublish_domain_event).

**Variants**

- `Created`: A new outbox row was inserted.
  - `id`: Event id.
- `Duplicate`: An equivalent `(event_type, dedup_key)` row already exists.
  - `existing_id`: Existing event id.

Implements: `Clone`, `Debug`, `Deserialize<'de>`, `Eq`, `PartialEq`, `Serialize`

### RequestStatus

```rust
pub enum RequestStatus {
    Open,
    Cancelled,
}
```

Wishlist row status (`open` while wishlisted; `cancelled` after un-wishlist).

**Variants**

- `Open`: Wishlist item is still open (not fulfilled/cancelled).
- `Cancelled`: Request cancelled by the requester or operator.

#### RequestStatus::as_str

```rust
pub fn as_str(self) -> &'static str
```

Returns the canonical snake_case / lowercase wire string.

#### RequestStatus::parse

```rust
pub fn parse(s: &str) -> Option<Self>
```

Parses the canonical wire string; returns `None` when unknown.

Implements: `Clone`, `ComposeSchema`, `Copy`, `Debug`, `Default`, `Deserialize<'de>`, `Eq`, `PartialEq`, `Serialize`, `ToSchema`

### UserRole

```rust
pub enum UserRole {
    Owner,
    Administrator,
    Member,
}
```

First-party user role (`owner` | `administrator` | `member`).

**Variants**

- `Owner`: Library super-user: may elevate to Operator after password re-auth.
- `Administrator`: Library / user administration without Operator elevation.
- `Member`: Library member with scoped portal access (no operator token).

#### UserRole::as_str

```rust
pub fn as_str(self) -> &'static str
```

Returns the canonical snake_case / lowercase wire string.

#### UserRole::parse

```rust
pub fn parse(s: &str) -> Option<Self>
```

Parses the canonical wire string; returns `None` when unknown.

#### UserRole::is_privileged

```rust
pub fn is_privileged(self) -> bool
```

Whether this role may provision users and manage the host library.

#### UserRole::can_elevate

```rust
pub fn can_elevate(self) -> bool
```

Whether this role may elevate to a short-lived Operator session.

#### UserRole::can_administer_owned_nodes

```rust
pub fn can_administer_owned_nodes(self) -> bool
```

Whether this role reaches the node administration routes even while
the user owns no node (Administrator, Owner). A Member reaches them
once they own a node: owning a node does not require a role.

Implements: `Clone`, `ComposeSchema`, `Copy`, `Debug`, `Deserialize<'de>`, `Display`, `Eq`, `PartialEq`, `Serialize`, `ToSchema`

### UserStatus

```rust
pub enum UserStatus {
    Active,
    Disabled,
}
```

First-party user status.

**Variants**

- `Active`: Account may sign in and use the portal.
- `Disabled`: Account is blocked from new sessions.

#### UserStatus::as_str

```rust
pub fn as_str(self) -> &'static str
```

Returns the canonical snake_case / lowercase wire string.

#### UserStatus::parse

```rust
pub fn parse(s: &str) -> Option<Self>
```

Parses the canonical wire string; returns `None` when unknown.

Implements: `Clone`, `ComposeSchema`, `Copy`, `Debug`, `Deserialize<'de>`, `Eq`, `PartialEq`, `Serialize`, `ToSchema`

### SchemaState

```rust
pub enum SchemaState {
    Uninitialized,
    Unreleased { base_version: i64, checksum: String },
    Frozen { version: i64, checksum: String },
}
```

Durable Bookclerk library schema state.

**Variants**

- `Uninitialized`: No Bookclerk bookkeeping and no host tables.
- `Unreleased`: Development pack applied on top of a frozen base (or pre-v1).
  - `base_version`: Highest frozen plan version this unreleased pack sits on (`0` = none).
  - `checksum`: SHA-256 hex of [`crate::migrations::unreleased_ops`](migrations.md#unreleased_ops).
- `Frozen`: A frozen [`crate::migrations::HostMigrationStep`](migrations.md#hostmigrationstep) is applied.
  - `version`: Frozen plan version (`>= 1`).
  - `checksum`: SHA-256 hex recorded for that frozen step.

#### SchemaState::display

```rust
pub fn display(&self) -> String
```

CLI / JSON display: `uninitialized`, `unreleased@base<n>+<checksum>`,
`frozen@<version>+<checksum>`.

#### SchemaState::frozen_version

```rust
pub fn frozen_version(&self) -> Option<i64>
```

Frozen revision when this state is [`Self::Frozen`](#schemastate); otherwise `None`.

Never use `0` as a stand-in for uninitialized or unreleased.

#### SchemaState::unreleased_base_version

```rust
pub fn unreleased_base_version(&self) -> Option<i64>
```

Frozen base recorded on an unreleased database (`0` = pre-v1).

#### SchemaState::checksum

```rust
pub fn checksum(&self) -> Option<&str>
```

Checksum when the database has applied Bookclerk schema.

Implements: `Clone`, `Debug`, `Display`, `Eq`, `PartialEq`

### FacetValuesOf

```rust
pub enum FacetValuesOf {
    Facet(WorkFacet),
    Tags(i64),
}
```

Whose values a facet value list reads.

**Variants**

- `Facet`: A facet of the page index.
- `Tags`: One user's tags (`users.id`).

Implements: `Clone`, `Copy`, `Debug`, `Eq`, `PartialEq`

### WorkFacet

```rust
pub enum WorkFacet {
    Author,
    Narrator,
    Series,
    Genre,
    Publisher,
    Source,
    Tag,
    Collection,
    Explicit,
}
```

A filter value list of the page index (`library_work_facets`).

**Variants**

- `Author`: Authors (hosts, for a podcast).
- `Narrator`: Narrators.
- `Series`: Series names.
- `Genre`: Genres (store categories).
- `Publisher`: Publishers.
- `Source`: Storefront plugins the work is held through.
- `Tag`: Library tags (`work_tags`): set by an Administrator or read from a
  sidecar, the same for everyone.
- `Collection`: Collections (`core.collections`) one of the work's files is in.
- `Explicit`: Whether a store marks one of the work's listings explicit (adult)
  content: the single value `explicit`, which a patron scope that
  hides explicit books excludes. Not offered as a Library filter.

#### WorkFacet::ALL

```rust
pub const ALL: [Self; 9] = _
```

Every facet, in the order a work's values are written.

#### WorkFacet::as_str

```rust
pub fn as_str(self) -> &'static str
```

The stored `facet` value.

Implements: `Clone`, `Copy`, `Debug`, `Deserialize<'de>`, `Eq`, `Hash`, `PartialEq`, `Serialize`

### WorkFilter

```rust
pub enum WorkFilter {
    Facet { facet: WorkFacet, value: String },
    Tag { user_id: Option<i64>, tag: String },
    Finished { user_id: Option<i64>, finished: bool },
}
```

One filter on a page of library works.

**Variants**

- `Facet`: Works with this value under a facet (matched by [`facet_key`](#facet_key)). An
  empty series value matches works in no series.
  - `facet`: The facet.
  - `value`: The value.
- `Tag`: Works the user tagged with this tag (matched by [`facet_key`](#facet_key)). With
  no user (an operator token), nothing matches.
  - `user_id`: The user (`users.id`) whose tags count.
  - `tag`: The tag.
- `Finished`: Works the user marked finished, or did not. With no user, no work is
  finished.
  - `user_id`: The user (`users.id`) whose state counts.
  - `finished`: `true` for finished works, `false` for the rest.

Implements: `Clone`, `Debug`, `Eq`, `PartialEq`

### WorkSort

```rust
pub enum WorkSort {
    Title,
    Author,
    Added,
    Series,
    Purchased,
    Length,
}
```

How a page of library works is ordered.

**Variants**

- `Title`: By title sort key, A to Z.
- `Author`: By the first author's sort name, then title.
- `Added`: Newest holding first ("recently added").
- `Series`: By first series name and position, then title; works in no series
  last.
- `Purchased`: Newest purchase first; works no store reported a purchase time for
  last.
- `Length`: Longest runtime first; works with no known runtime last.

Implements: `Clone`, `Copy`, `Debug`, `Default`, `Deserialize<'de>`, `Eq`, `PartialEq`, `Serialize`

### HoldingHolder

```rust
pub enum HoldingHolder {
    Account { plugin_key: String, account_id: String },
    Node { node_id: String },
    Patron { user_id: i64 },
}
```

Who holds a listing.

**Variants**

- `Account`: A store account, by plugin and account id.
  - `plugin_key`: The account's plugin.
  - `account_id`: The account id at that plugin.
- `Node`: A branch library's node, by node id.
  - `node_id`: The node id.
- `Patron`: A patron who uploaded the book, by user id.
  - `user_id`: The person's `users.id`.

Implements: `Clone`, `Debug`, `Eq`, `PartialEq`

### WorkTagSource

```rust
pub enum WorkTagSource {
    Admin,
    Sidecar,
}
```

Where a library tag came from (`work_tags.source`).

**Variants**

- `Admin`: Set in Bookclerk by an Owner, an Administrator or the operator.
- `Sidecar`: Read from a metadata file beside the book.

#### WorkTagSource::as_str

```rust
pub fn as_str(self) -> &'static str
```

The stored value.

Implements: `Clone`, `ComposeSchema`, `Copy`, `Debug`, `Eq`, `Ord`, `PartialEq`, `PartialOrd`, `Serialize`, `ToSchema`

### OfferAudience

```rust
pub enum OfferAudience {
    NonMember,
    Member,
}
```

Who a [`ListingOffer`](#listingoffer) price is for.

**Variants**

- `NonMember`: Someone without an account at the store: the price Discover shows
  unless the person has an account there.
- `Member`: A member or subscriber of the store.

#### OfferAudience::as_str

```rust
pub fn as_str(self) -> &'static str
```

The stored `listing_offers.audience` text.

#### OfferAudience::parse

```rust
pub fn parse(text: &str) -> Option<Self>
```

Parses stored audience text; `None` for text a newer release wrote.

Implements: `Clone`, `Copy`, `Debug`, `Eq`, `Hash`, `Ord`, `PartialEq`, `PartialOrd`

### ChapterCheck

```rust
pub enum ChapterCheck {
    Strict,
    Lenient,
}
```

How strictly [`ChapterList::new`](#chapterlistnew) reads its input.

**Variants**

- `Strict`: A person's edit: anything outside the rules is refused, so what is
  stored is exactly what they sent.
- `Lenient`: A machine source: control characters are dropped, titles trimmed
  and cut to [`MAX_CHAPTER_TITLE_BYTES`](#max_chapter_title_bytes), an empty title becomes
  "Chapter n", and a depth that skips a level or passes
  [`MAX_CHAPTER_DEPTH`](#max_chapter_depth) is lifted. Starts out of order and too many
  chapters are still refused, because no repair keeps their meaning.

Implements: `Clone`, `Copy`, `Debug`, `Eq`, `PartialEq`

### ChapterSource

```rust
pub enum ChapterSource {
    Person { user_id: Option<i64> },
    Processed { input: Option<ChapterDigest> },
    Provider { plugin: String, quality: ProviderQuality },
    Listing { plugin: String, kind: MarkerKind },
    PartNames { plugin: String },
    File { copy: String },
}
```

Who states a chapter claim, as a writer names it.

**Variants**

- `Person`: An Administrator's edit. One per edition: a newer edit replaces it.
  - `user_id`: The Administrator (`users.id`); `None` for the operator.
- `Processed`: Bookclerk's chapters aligned to the audio during acquire.
  - `input`: The claim whose titles and starts were aligned.
- `Provider`: A metadata provider's layout.
  - `plugin`: The provider's PluginKey.
  - `quality`: How it found the edition and whether its offsets are exact.
- `Listing`: A store's markers.
  - `plugin`: The storefront's PluginKey.
  - `kind`: What the store says the markers are.
- `PartNames`: The titles of a store's parts.
  - `plugin`: The storefront's PluginKey.
- `File`: The chapters embedded in one stored copy's files, as first
  imported.
  - `copy`: The copy ([`copy_source_id`](#copy_source_id)).

Implements: `Clone`, `Debug`, `Eq`, `PartialEq`

### ChapterSourceKind

```rust
pub enum ChapterSourceKind {
    Person,
    OutsideEdit,
    Processed,
    Provider,
    Listing,
    File,
    PartNames,
}
```

The kind of source a chapter claim came from (`source_kind`).

**Variants**

- `Person`: An Administrator's edit in Bookclerk.
- `OutsideEdit`: A change found outside Bookclerk: a file it did not write, or an
  edit in an integrated tool. Nothing records these yet.
- `Processed`: Bookclerk's own chapters, aligned to the audio during acquire.
- `Provider`: A metadata provider's chapter layout.
- `Listing`: The markers a store returned with the audio.
- `File`: Chapters read from a stored copy's files as first imported: an
  upload records them.
- `PartNames`: The titles of the parts a store delivered, one chapter per part.

#### ChapterSourceKind::ALL

```rust
pub const ALL: [Self; 7] = _
```

Every kind, in the order the stored values are documented.

#### ChapterSourceKind::as_str

```rust
pub fn as_str(self) -> &'static str
```

The stored spelling.

#### ChapterSourceKind::parse

```rust
pub fn parse(stored: &str) -> Option<Self>
```

Reads a stored spelling.

Implements: `Clone`, `ComposeSchema`, `Copy`, `Debug`, `Deserialize<'de>`, `Display`, `Eq`, `Hash`, `PartialEq`, `Serialize`, `ToSchema`

### ChaptersOutcome

```rust
pub enum ChaptersOutcome {
    Changed,
    Unchanged,
}
```

What a chapter write changed.

**Variants**

- `Changed`: The edition's chapters now come from another list.
- `Unchanged`: The claims changed but the edition plays the same list.

Implements: `Clone`, `Copy`, `Debug`, `Eq`, `PartialEq`

### ClaimQuality

```rust
pub enum ClaimQuality {
    None,
    StrongAccurate,
    StrongEstimated,
    Weak,
    Chapters,
    Tracks,
    Unknown,
}
```

Where a machine source ranks within its kind (`quality`).

**Variants**

- `None`: Not ranked within its kind: a person's edit, an outside edit,
  Bookclerk's own processing, a file, part names.
- `StrongAccurate`: A provider found the edition by a strong identifier and marks the
  offsets exact.
- `StrongEstimated`: A provider found the edition by a strong identifier; the offsets
  are estimates.
- `Weak`: A provider found the edition by a weak identifier or a search.
- `Chapters`: A store's markers that it says are chapters.
- `Tracks`: A store's markers that it says are track or part boundaries.
- `Unknown`: A store's markers it says nothing about.

#### ClaimQuality::ALL

```rust
pub const ALL: [Self; 7] = _
```

Every quality, in the order the stored values are documented.

#### ClaimQuality::as_str

```rust
pub fn as_str(self) -> &'static str
```

The stored spelling; empty for [`Self::None`](#claimquality).

#### ClaimQuality::parse

```rust
pub fn parse(stored: &str) -> Option<Self>
```

Reads a stored spelling.

Implements: `Clone`, `ComposeSchema`, `Copy`, `Debug`, `Eq`, `Hash`, `PartialEq`, `Serialize`, `ToSchema`

### MarkerKind

```rust
pub enum MarkerKind {
    Chapters,
    Tracks,
    Unknown,
}
```

What a store says its markers are.

**Variants**

- `Chapters`: Literary chapters.
- `Tracks`: Track or part boundaries.
- `Unknown`: The store did not say.

Implements: `Clone`, `Copy`, `Debug`, `Eq`, `PartialEq`

### ProviderQuality

```rust
pub enum ProviderQuality {
    StrongAccurate,
    StrongEstimated,
    Weak,
}
```

How a provider found the edition, and how exact its offsets are.

**Variants**

- `StrongAccurate`: Found by a strong identifier; offsets marked exact.
- `StrongEstimated`: Found by a strong identifier; offsets estimated.
- `Weak`: Found by a weak identifier or a search.

Implements: `Clone`, `Copy`, `Debug`, `Eq`, `PartialEq`

### EventPruneStep

```rust
pub enum EventPruneStep {
    ExpireUnleased,
    TerminalDeliveries,
    DeadLetters,
    StaleSubscriberNodes,
    EmptyEvents,
}
```

One delete of [`LibraryStore::prune_event_deliveries`](#librarystoreprune_event_deliveries), in run order.

**Variants**

- `ExpireUnleased`: Pending deliveries never leased within their retention period become
  `expired` (see [`LibraryStore::expire_unleased_event_deliveries`](#librarystoreexpire_unleased_event_deliveries)).
- `TerminalDeliveries`: Acked, rejected and expired deliveries past `retention_days`.
- `DeadLetters`: Dead letters past `dead_letter_retention_days`.
- `StaleSubscriberNodes`: Per-node subscriber heartbeats past their TTL.
- `EmptyEvents`: Dispatched events past `retention_days` with no deliveries left.

#### EventPruneStep::ALL

```rust
pub const ALL: [Self; 5] = _
```

Every step, in the order the deletes must run (deliveries before the
events they belong to).

Implements: `Clone`, `Copy`, `Debug`, `Eq`, `PartialEq`

### PinOutcome

```rust
pub enum PinOutcome {
    Moved { from: String },
    Unchanged,
    UnknownListing,
    UnknownEdition,
}
```

What [`LibraryStore::pin_listing`](#librarystorepin_listing) did.

**Variants**

- `Moved`: The listing now sells the pinned edition; it sold `from` before.
  - `from`: The edition the listing left.
- `Unchanged`: The listing already sold the pinned edition; the pin is recorded
  (or renewed) and nothing moved.
- `UnknownListing`: No listing has this id.
- `UnknownEdition`: No edition has this id.

Implements: `Clone`, `Debug`, `Eq`, `PartialEq`

### DownloadStarter

```rust
pub enum DownloadStarter {
    Host,
    Person,
}
```

Who started a download, for the ledger.

**Variants**

- `Host`: The host started it (auto-acquire, upgrades, acquire-all, requests
  from other libraries): held for the pace.
- `Person`: A person started it by hand: counted, never held.

Implements: `Clone`, `Copy`, `Debug`, `Eq`, `PartialEq`

### PlaybackEvent

```rust
pub enum PlaybackEvent {
    Started,
    Progress,
    Finished,
}
```

A host event about someone's listening, published after the write it
describes ([`LibraryStore::publish_playback_event`](#librarystorepublish_playback_event)).

**Variants**

- `Started`: A play session opened (`playback_started`).
- `Progress`: A play session closed, or progress was set outright
  (`playback_progress`). The position syncs of a playing session
  publish nothing.
- `Finished`: The listener reached the end of an edition (`playback_finished`).

#### PlaybackEvent::ALL

```rust
pub const ALL: [Self; 3] = _
```

Every event, in the order of a listen.

#### PlaybackEvent::as_str

```rust
pub fn as_str(self) -> &'static str
```

The event type plugins subscribe to.

Implements: `Clone`, `Copy`, `Debug`, `Eq`, `PartialEq`

### PlayerKind

```rust
pub enum PlayerKind {
    Web,
    Desktop,
    App,
}
```

What kind of player opened a play session.

**Variants**

- `Web`: The web app in a browser.
- `Desktop`: The desktop window's native engine.
- `App`: An outside player app, through the player API.

#### PlayerKind::ALL

```rust
pub const ALL: [Self; 3] = _
```

Every kind, in stored order (the `playback_sessions.client_kind`
CHECK list).

#### PlayerKind::as_str

```rust
pub fn as_str(self) -> &'static str
```

The stored spelling.

#### PlayerKind::parse

```rust
pub fn parse(stored: &str) -> Self
```

The kind a stored spelling names; [`Self::App`](#playerkind) for an unknown one.

Implements: `Clone`, `Copy`, `Debug`, `Eq`, `From<PlayerKindParam>`, `Hash`, `Ord`, `PartialEq`, `PartialOrd`

### PlayerKindParam

```rust
pub enum PlayerKindParam {
    Web,
    Desktop,
}
```

The kind of first-party player that opens a session. Outside player
apps open theirs through the player API.

**Variants**

- `Web`: The web app in a browser.
- `Desktop`: The desktop window.

Implements: `Clone`, `ComposeSchema`, `Copy`, `Debug`, `Default`, `Deserialize<'de>`, `Eq`, `From<PlayerKindParam>`, `PartialEq`, `ToSchema`

### KvPutOutcome

```rust
pub enum KvPutOutcome {
    Written,
    ValueTooLarge { len: usize, max: u32 },
    KeyLimitReached { max: u32 },
}
```

Why a write was refused, so the binding can map it onto a wire code.

**Variants**

- `Written`: The row was written.
- `ValueTooLarge`: `value` exceeds `max_value_bytes`.
  - `len`: Bytes offered.
  - `max`: Namespace cap.
- `KeyLimitReached`: The namespace already holds `max_keys` other live keys.
  - `max`: Namespace cap.

Implements: `Clone`, `Debug`, `Eq`, `PartialEq`

### RecordPruneStep

```rust
pub enum RecordPruneStep {
    OidcAuthCodes,
    OidcRefreshTokens,
    PortalSessions,
    ClaimTickets,
    UserInvites,
    EnrollmentLinks,
    UploadSessions,
    CopyGrants,
    StorageScans,
    SecurityAudit,
    ConfigurationAudit,
    ConfigurationChanges,
    Notifications,
}
```

One delete of [`LibraryStore::prune_record_step`](#librarystoreprune_record_step), in run order.

**Variants**

- `OidcAuthCodes`: OIDC authorization codes past their expiry (they are single-use
  and minutes long; consumed codes are refused by `consumed_at`
  until then).
- `OidcRefreshTokens`: OIDC refresh tokens that expired or were revoked; neither is ever
  accepted again.
- `PortalSessions`: User portal sessions past their expiry.
- `ClaimTickets`: Claim tickets expired more than [`TOKEN_HISTORY_DAYS`](#token_history_days) ago.
- `UserInvites`: User invites expired more than [`TOKEN_HISTORY_DAYS`](#token_history_days) ago.
- `EnrollmentLinks`: Node enrollment links expired more than [`TOKEN_HISTORY_DAYS`](#token_history_days) ago.
- `UploadSessions`: Committed or aborted uploads expired more than
  [`UPLOAD_HISTORY_DAYS`](#upload_history_days) ago.
- `CopyGrants`: Copy grants past their expiry; no download accepts one after it.
- `StorageScans`: Storage-scan generations whose job is terminal or gone, and jobless
  ones silent for [`STORAGE_SCAN_STALE_DAYS`](#storage_scan_stale_days), with their rows.
- `SecurityAudit`: Security audit events older than [`AUDIT_RETENTION_DAYS`](#audit_retention_days).
- `ConfigurationAudit`: Configuration audit rows older than [`AUDIT_RETENTION_DAYS`](#audit_retention_days).
- `ConfigurationChanges`: Configuration change notices older than
  [`CONFIGURATION_CHANGE_RETENTION_DAYS`](#configuration_change_retention_days).
- `Notifications`: Notifications older than
  [`crate::notifications::NOTIFICATION_RETENTION_DAYS`](notifications.md#notification_retention_days), with their
  inbox entries and delivery-log rows.

#### RecordPruneStep::ALL

```rust
pub const ALL: [Self; 13] = _
```

Every step, in the order a maintenance pass runs them.

Implements: `Clone`, `Copy`, `Debug`, `Eq`, `PartialEq`

### FieldEdit

```rust
pub enum FieldEdit<T> {
    Keep,
    Clear,
    Set(T),
}
```

One field of a [`UserStateEdit`](#userstateedit).

**Variants**

- `Keep`: Leave the stored value as it is.
- `Clear`: Remove the stored value.
- `Set`: Store this value.

Implements: `Clone`, `Debug`, `Default`, `PartialEq`

### SplitOutcome

```rust
pub enum SplitOutcome {
    Split { work_id: String },
    UnknownWork,
    BadCount,
    NotOfWork(Vec<String>),
    WouldEmpty,
}
```

What [`LibraryStore::split_work`](#librarystoresplit_work) did.

**Variants**

- `Split`: The expressions now belong to `work_id`, a new work.
  - `work_id`: The new work.
- `UnknownWork`: No work has this id.
- `BadCount`: No expression was named, or more than [`MAX_SPLIT_EXPRESSIONS`](#max_split_expressions).
- `NotOfWork`: These named expressions are not the work's.
- `WouldEmpty`: The split would leave the work with no expression.

Implements: `Clone`, `Debug`, `Eq`, `PartialEq`

### CopyState

```rust
pub enum CopyState {
    Stored,
    Pending,
    Failed,
    Missing,
}
```

Where an edition's copy at one destination stands.

**Variants**

- `Stored`: The destination holds the complete files.
- `Pending`: A write failed or the destination was unavailable; the copy is
  retried once its delay has passed.
- `Failed`: The copy failed [`MAX_COPY_ATTEMPTS`](#max_copy_attempts) times in a row and is no
  longer retried on its own.
- `Missing`: The destination's file disappeared outside Bookclerk and
  `output.missing_files` is `mark_missing`: the copy is left alone
  until an acquire of the title writes it again.

#### CopyState::ALL

```rust
pub const ALL: [Self; 4] = _
```

Every state, in stored order (the `edition_copies.state` CHECK list).

#### CopyState::as_str

```rust
pub fn as_str(self) -> &'static str
```

The stored spelling.

#### CopyState::parse

```rust
pub fn parse(stored: &str) -> Option<Self>
```

The state a stored spelling names; `None` for one this build does
not know.

Implements: `Clone`, `Copy`, `Debug`, `Eq`, `Hash`, `Ord`, `PartialEq`, `PartialOrd`

### DiscoverSubject

```rust
pub enum DiscoverSubject {
    Operator,
    User(i64),
    Portal(i64),
}
```

Whose Discover shelves a stored row belongs to.

The same key as `user_preferences.subject_key`: the operator, a
first-party user, or a portal identity with no user.

**Variants**

- `Operator`: The operator (token, session, or auth disabled); seeded from the household.
- `User`: A first-party user, by `users.id`.
- `Portal`: A portal identity that has no first-party user, by `portal_identities.id`.

#### DiscoverSubject::parse

```rust
pub fn parse(key: &str) -> Option<Self>
```

Parses a stored subject key (`operator`, `user:<id>`, `portal:<id>`).

##### Returns

`None` for any other text, including a non-numeric or negative id.

#### DiscoverSubject::key

```rust
pub fn key(&self) -> String
```

The subject's stored key, the inverse of [`Self::parse`](#discoversubjectparse).

Implements: `Clone`, `Debug`, `Eq`, `Hash`, `Ord`, `PartialEq`, `PartialOrd`

### FeedState

```rust
pub enum FeedState {
    Off,
    Healthy,
    Gap,
    Failing,
}
```

The state of a destination's change feed.

**Variants**

- `Off`: The destination's plugin has no change feed (or it is not turned
  on); scheduled listings alone notice outside changes.
- `Healthy`: The feed answered and said it missed nothing; listings run rarely.
- `Gap`: The feed said it may have missed changes; a listing runs at once.
- `Failing`: Reading the feed failed; a listing runs at once and the feed is
  tried again on the next pass.

#### FeedState::ALL

```rust
pub const ALL: [Self; 4] = _
```

Every state, in stored order (the `destination_feeds.feed_state`
CHECK list).

#### FeedState::as_str

```rust
pub fn as_str(self) -> &'static str
```

The stored spelling.

#### FeedState::parse

```rust
pub fn parse(stored: &str) -> Option<Self>
```

The state a stored spelling names; `None` for one this build does
not know.

Implements: `Clone`, `Copy`, `Debug`, `Eq`, `Hash`, `Ord`, `PartialEq`, `PartialOrd`

### FileOwner

```rust
pub enum FileOwner {
    Edition(String),
}
```

Who a recorded file key belongs to.

**Variants**

- `Edition`: An edition (`editions.id`): every holding of it shares the file.

#### FileOwner::kind

```rust
pub fn kind(&self) -> &'static str
```

The stored `owner_kind`.

#### FileOwner::id

```rust
pub fn id(&self) -> &str
```

The stored `owner_id`.

Implements: `Clone`, `Debug`, `Eq`, `Ord`, `PartialEq`, `PartialOrd`

### FileRole

```rust
pub enum FileRole {
    Audio,
    Part,
    Chapter,
    Pdf,
    Cover,
    Sidecar,
    Transcript,
    Epub,
}
```

What a recorded file is.

**Variants**

- `Audio`: The edition's primary audio file.
- `Part`: Another audio file of the edition: a part or a chapter file.
- `Chapter`: A chapter file of a title split by chapter.
- `Pdf`: The companion PDF.
- `Cover`: Cover art.
- `Sidecar`: Any other file beside the audio (`.cue`, chapter JSON, metadata).
- `Transcript`: A transcript or lyrics file.
- `Epub`: An EPUB.

#### FileRole::ALL

```rust
pub const ALL: [Self; 8] = _
```

Every role, in stored order (the `edition_files.role` CHECK list).

#### FileRole::as_str

```rust
pub fn as_str(self) -> &'static str
```

The stored spelling.

#### FileRole::parse

```rust
pub fn parse(stored: &str) -> Self
```

The role a stored spelling names; [`Self::Sidecar`](#filerole) for an unknown
one, the role that gates nothing.

#### FileRole::for_key

```rust
pub fn for_key(key: &str, primary: Option<&str>) -> Self
```

The role of the file at `key`, from its extension.

##### Arguments

* `key` - The storage key.
* `primary` - The edition's primary audio key, when known: another
  audio file is then a [`Self::Part`](#filerole). Without it every audio file
  is [`Self::Audio`](#filerole).

Implements: `Clone`, `Copy`, `Debug`, `Eq`, `Ord`, `PartialEq`, `PartialOrd`

### HoldingSelection

```rust
pub enum HoldingSelection<'a> {
    Products(&'a [String]),
    UnseenBy(&'a str),
    All,
}
```

Which holdings of one account a removal takes.

**Variants**

- `Products`: The account's holdings of these products of its plugin (a scan page's
  `removed` ids).
- `UnseenBy`: The account's holdings that the scan pass `pass_id` did not list.
- `All`: Every holding of the account (it was revoked).

Implements: `Clone`, `Debug`, `Eq`, `PartialEq`

### NodeHoldingSelection

```rust
pub enum NodeHoldingSelection<'a> {
    UnseenBy { pass: &'a str, fence: i64 },
    All,
    AllUnderFence(i64),
    Replaced { ids: &'a [String], fence: i64 },
    UnsharedBefore(&'a str),
}
```

Which holdings of one branch (node) a removal takes.

**Variants**

- `UnseenBy`: The node's holdings its catalog pass `pass` (the node's
  `catalog_pass_started_at`) did not name, while that pass is still
  the node's and `fence` still its catalog fence: a connection that
  resumes the pass keeps what it links, and ends the pass itself.
  - `pass`: The catalog pass id.
  - `fence`: The catalog fence of the batch that ended the pass.
- `All`: Every holding of the node (it left or was revoked).
- `AllUnderFence`: Every holding of the node while `fence` is still the node's catalog
  fence (it stopped sharing under that fence): a newer connection that
  shares again keeps what it links.
- `Replaced`: These holdings of the node, by id (ones a title's newer holding
  replaced), while `fence` is still the node's catalog fence: a newer
  connection may have taken them up again.
  - `ids`: The holdings' ids.
  - `fence`: The catalog fence the replacing batch was written under.
- `UnsharedBefore`: The node's holdings created before this stamp that still carry no
  share state: ones a catalog batch linked before finding its fence
  gone. A holding a live batch has given its share state since stays,
  and one newer than the stamp may belong to a batch still in flight.

Implements: `Clone`, `Copy`, `Debug`, `Eq`, `PartialEq`

### SeedScope

```rust
pub enum SeedScope {
    Own,
    Household,
}
```

Which seeds a subject's candidates and feed were built from.

**Variants**

- `Own`: The subject's own books, listening, ratings and wishlist.
- `Household`: The whole household library, used while the subject has no taste of
  their own (and always for the operator).

#### SeedScope::as_str

```rust
pub fn as_str(self) -> &'static str
```

Wire and column text (`own`, `household`).

#### SeedScope::parse

```rust
pub fn parse(text: &str) -> Option<Self>
```

Parses [`Self::as_str`](#seedscopeas_str) text; anything else is `None`.

Implements: `Clone`, `Copy`, `Debug`, `Eq`, `PartialEq`

## Functions

### migrate_library_backend

```rust
pub async fn migrate_library_backend(
    source: &LibraryStore,
    dest: &LibraryStore,
    opts: &BackendMigrateOptions,
) -> Result<BackendMigrateSummary>
```

Copy the whole library from `source` into `dest`.

The destination should be empty unless [`BackendMigrateOptions::force`](#backendmigrateoptions) is set.
Plugin artifacts are written first through the destination's own chunked
writer (they are content-addressed, so a second run rewrites nothing);
every other row is then copied in a single transaction on the
destination.

#### Arguments

* `source` - Existing library backend to read from.
* `dest` - Target backend to write into (must already be migrated).
* `opts` - Dry-run / force behavior.

#### Returns

Per-table row counts copied (or that would be copied when `dry_run`).

#### Errors

Returns [`LibraryError`](#libraryerror) when the destination is non-empty without `force`,
or when any read/write fails.

### restore_backup_unit

```rust
pub async fn restore_backup_unit(
    db: &DatabaseConnection,
    repo: &BackupRepository,
    unit: &BackupUnit,
    kind: CanonicalRestoreKind,
    opts: &CanonicalRestoreOpts,
    preserve_plugin_registry: bool,
) -> Result<()>
```

Destructively replaces one logical database with `unit`. Never merges.

Verifies every referenced object before the first `DROP`. Plugin restore
rebuilds catalog companions from admitted SQL. It does not run plugin-owned
migrations. When `preserve_plugin_registry` is true, library restore does
not drop or replay `plugin_databases` (environment-local placement).
Row inserts are parameterized canonical Bookclerk SQL (`?` + typed
`DbValue` binds); adapters lower placeholders at the execute edge.
SQLite restore defers foreign keys on the restore transaction and runs
`PRAGMA foreign_key_check` before commit. It never toggles
`PRAGMA foreign_keys` on the pool connection.

#### Errors

Returns when validation fails, the adapter cannot provide complete unit
replacement, a bind/payload/request budget is exceeded, or DDL/DML cannot
be applied.

### library_ddl_for_schema_state

```rust
pub fn library_ddl_for_schema_state(state: &SchemaState) -> Result<String>
```

Library DDL that matches `state`, not necessarily this binary's latest pack.

#### Errors

Returns when the database is uninitialized, checksums do not match, or a
frozen version is unknown to this binary.

### sort_tables_by_foreign_keys

```rust
pub fn sort_tables_by_foreign_keys(
    tables: Vec<CanonicalTableSchema>,
) -> Result<Vec<CanonicalTableSchema>>
```

FK-safe table order (parents before children). Cycles fail closed.

Column-level `REFERENCES` and table-level `FOREIGN KEY` both count.
Self-references are ignored. References to tables outside this schema
are ignored. Ready nodes keep original declaration order so a child named
`a_child` that references `z_parent` still waits for the parent.

#### Errors

Returns when the FK graph contains a cycle.

### verify_recovery_point

```rust
pub fn verify_recovery_point(repo: &BackupRepository, id: &str) -> Result<ValidatedBackup>
```

Parses the manifest, rejects unsupported formats, and verifies every
referenced object **before** the caller starts a destructive restore.

#### Errors

Returns when the manifest is malformed, an object is missing/corrupt, schema
is not fully admitted, or typed cells violate the contract.

### admit_canonical_schema

```rust
pub fn admit_canonical_schema(
    sql_contract_version: u32,
    sql: &str,
) -> Result<CanonicalDatabaseSchema>
```

Parses and fully admits a diagnostic/imported canonical SQL script.

Production capture, checksum, apply, and restore paths use already-separated
statement lists ([`admit_canonical_statements`](#admit_canonical_statements)). This helper exists for
tests and imported scripts that still arrive as one packable string.

#### Errors

Returns when the packer rejects `sql` or any statement is not admitted.

### admit_canonical_statements

```rust
pub fn admit_canonical_statements(
    sql_contract_version: u32,
    statements: &[&str],
) -> Result<CanonicalDatabaseSchema>
```

Admits already-separated canonical `CREATE TABLE` / `CREATE INDEX` statements.

#### Errors

Returns when any statement is not admitted Bookclerk SQL.

### apply_admitted_sql

```rust
pub async fn apply_admitted_sql(
    db: &DatabaseConnection,
    statements: &[&str],
    kind: CanonicalRestoreKind,
) -> Result<()>
```

Applies admitted canonical DDL plus binding companions (tests / bootstrap).

#### Errors

Returns when a statement fails.

### archive_backup

```rust
pub fn archive_backup(files_dir: &Path, id: &str, dest: &Path) -> Result<()>
```

Packages one recovery point (manifest + referenced objects) as `.tar.gz`.

#### Errors

Returns when the destination cannot be written or objects are missing.

### backup_library

```rust
pub async fn backup_library(
    db: &DatabaseConnection,
    req: &BackupRequest,
) -> Result<Option<BackupOutcome>>
```

Writes a crash-safe recovery point under `files_dir/backups/`.

Skips only [`SchemaState::Uninitialized`](#schemastate). The manifest is published only
after every referenced object exists. Pre-migrate backups prune older
automatic recovery points and garbage-collect unreferenced objects.
Manual backups are never auto-pruned.

#### Errors

Returns when the files dir cannot be written, capture fails, or a requested
plugin binding is missing from [`BackupRequest::plugin_units`](#backuprequest).

### extract_backup_archive

```rust
pub fn extract_backup_archive(archive: &Path, dest: &Path) -> Result<()>
```

Extracts a `.tar.gz` recovery-point archive into `dest` (a backups root).

Expansion is bounded by entry count, per-entry size, total expanded bytes,
and gzip-decoded tar stream size so a small archive cannot fill the disk.

#### Errors

Returns when the archive is missing, malformed, a path escapes `dest`, or
an extract budget would be exceeded.

### filter_library_pack_ddl

```rust
pub fn filter_library_pack_ddl(sql: &str) -> Result<String>
```

Library migration packs may include seed `INSERT`s. Those are table data
(captured from rows), not schema objects. This keeps admitted
`CREATE TABLE` / `CREATE INDEX` and skips seed DML. Any other statement
fails closed.

#### Errors

Returns when a non-empty statement is neither admitted DDL nor seed DML.

### library_canonical_schema

```rust
pub fn library_canonical_schema() -> Result<CanonicalDatabaseSchema>
```

Admitted library schema for this binary's current pack (frozen ups + unreleased).

#### Errors

Returns when current canonical SQL is not fully admitted.

### library_canonical_schema_for_state

```rust
pub fn library_canonical_schema_for_state(
    state: &SchemaState,
) -> Result<CanonicalDatabaseSchema>
```

Admitted library schema for `state`.

#### Errors

Returns when DDL cannot be resolved or is not fully admitted.

### list_backups

```rust
pub fn list_backups(files_dir: &Path) -> Result<Vec<BackupListEntry>>
```

Lists published recovery points oldest-first.

#### Errors

Returns when the repository cannot be opened.

### plugin_canonical_schema_from_ddl_catalog

```rust
pub async fn plugin_canonical_schema_from_ddl_catalog(
    db: &DatabaseConnection,
) -> Result<CanonicalDatabaseSchema>
```

Rebuilds plugin schema from durable `bookclerk_sql_ddl` on an open connection.

#### Errors

Returns when the DDL catalog is missing, incomplete, or malformed.

### prune_automatic_backups

```rust
pub fn prune_automatic_backups(files_dir: &Path) -> Result<usize>
```

Deletes oldest automatic pre-migrate recovery points. Never deletes `manual`.

#### Errors

Returns when a manifest cannot be unlinked.

### resolve_backup_spec

```rust
pub fn resolve_backup_spec(files_dir: &Path, spec: &str) -> Result<BackupResolve>
```

Resolves a recovery-point id, timestamp prefix, or archive path.

#### Errors

Returns when no complete backup matches or the spec is ambiguous.

### restore_backup

```rust
pub async fn restore_backup(
    db: &DatabaseConnection,
    files_dir: &Path,
    id: &str,
    opts: &CanonicalRestoreOpts,
) -> Result<RestorePlan>
```

Restores the library unit after verifying **all** objects and units.

Plugin units are returned for the host/CLI to restore onto the target
adapter. Restore replaces Bookclerk-visible schema for the library unit
and does not merge or auto-migrate. Plugin-owned migrations are not run.

The library `plugin_databases` registry is preserved when the recovery
point carries no plugin units (including `--include-plugin-databases` with
an empty registry at capture). When plugin units are present, library
restore clears the registry so leftover bindings cannot outlive the
captured set; the host/CLI rebinds while restoring those units.

#### Errors

Returns when the backup is unsupported, an object is missing/corrupt, or
library replace fails. Destructive SQL starts only after complete preflight.

### restore_backup_in_repo

```rust
pub async fn restore_backup_in_repo(
    db: &DatabaseConnection,
    repo: &BackupRepository,
    id: &str,
    opts: &CanonicalRestoreOpts,
) -> Result<RestorePlan>
```

Restores from an already-open repository (extracted archives).

Target [`crate::SchemaState`](#schemastate) is read from the canonical
`bookclerk_schema_migrations` rows. This function never infers a
versioning contract from [`sea_orm::DbBackend`](https://docs.rs/sea_orm/latest/sea_orm/database/db_connection/type.DbBackend.html).

#### Errors

Same as [`restore_backup`](#restore_backup).

### apply_binding_bootstrap

```rust
pub async fn apply_binding_bootstrap(db: &DatabaseConnection) -> Result<()>
```

Applies host-owned binding bootstrap when the binding is uninitialized.

Matching [`SchemaState::Unreleased`](#schemastate) is a no-op. Restore that rewrites
`bookclerk_schema_migrations` as ordinary rows will take this no-op path and will not
re-run plugin-owned DDL.

#### Errors

Returns when schema apply or the binding state machine fails closed.

### binding_bootstrap_plan

```rust
pub fn binding_bootstrap_plan(state: &SchemaState) -> Result<Option<Vec<String>>>
```

Statements to apply for `state`, or `None` when the binding already matches.

#### Errors

Returns when `state` is a checksum/base mismatch or an unexpected frozen
binding (this binary has no frozen binding plan).

### db_atomic_operation_id

```rust
pub fn db_atomic_operation_id(op: &DbAtomicParams) -> String
```

Caller-owned idempotency key for an atomic attempt.

Consume-once keys are derived from the operation so an HTTP/RPC retry
resumes the same receipt. Claim redeem includes the session hash; callers
must derive that session from the ticket plus browser nonce (see
[`crate::derive_claim_session_token`](#derive_claim_session_token)) so a new HTTP request after a lost
reply reuses this id. Other ops mint a UUID for this attempt.

### db_atomic_request_hash

```rust
pub fn db_atomic_request_hash(op: &DbAtomicParams) -> Result<String>
```

SHA-256 hex digest of the idempotency-relevant fields of `op`.

Claim redeem omits `expires_at`, client metadata, and the randomized Argon2
`new_password_hash`. Idempotency uses `password_fingerprint` (HMAC of the
plaintext password) so a retry with a fresh Argon2 salt still matches.

#### Errors

Returns an error when the operation fails.

### execute_db_atomic

```rust
pub async fn execute_db_atomic(
    db: &DatabaseConnection,
    compiled: CompiledAtomic,
) -> Result<DbAtomicResult>
```

Runs a compiled named atomic as one native SQL transaction.

#### Errors

Returns [`LibraryError`](#libraryerror) for engine failures. Application outcomes
(`lastOwner`, `empty`, …) are returned as [`DbAtomicResult`](#dbatomicresult), not errors.

### execute_named_atomic

```rust
pub async fn execute_named_atomic(
    db: &DatabaseConnection,
    operation_id: &str,
    params: &DbAtomicParams,
) -> Result<DbAtomicResult>
```

Compiles `params` and runs the plan as one native SQL transaction.

#### Errors

Returns [`LibraryError`](#libraryerror) for engine failures or invalid named operations.

### apply_host_schema

```rust
pub async fn apply_host_schema(db: &DatabaseConnection) -> Result<()>
```

Applies pending host-authored DDL from the canonical migration plan.

In-process adapters (sqlite tests, native postgres tests) infer the engine
from the opened connection. Production plugin-host uses
[`apply_host_schema_with_batch_opts`](#apply_host_schema_with_batch_opts) so adapters lower canonical DDL after
RPC.

#### Errors

Returns [`LibraryError`](#libraryerror) when a version read or DDL statement fails, the
database is newer than this binary, or a frozen checksum does not match.

### apply_host_schema_with_batch

```rust
pub async fn apply_host_schema_with_batch<F, Fut>(
    db: &DatabaseConnection,
    run_batch: F,
) -> Result<()>
where
    F: FnMut(Vec<String>) -> Fut,
    Fut: Future<Output = Result<()>>,
```

Applies schema using `run_batch` (typed `executeAtomic`) for each version.

#### Errors

Returns [`LibraryError`](#libraryerror) when a version read, DDL statement, or batch fails.

### apply_host_schema_with_batch_opts

```rust
pub async fn apply_host_schema_with_batch_opts<F, Fut>(
    db: &DatabaseConnection,
    opts: SchemaApplyOptions,
    run_batch: F,
) -> Result<()>
where
    F: FnMut(Vec<String>) -> Fut,
    Fut: Future<Output = Result<()>>,
```

Applies schema with backups using `run_batch` for each version.

#### Errors

Returns [`LibraryError`](#libraryerror) when a version read, backup, DDL, or batch fails.

### apply_host_schema_with_options

```rust
pub async fn apply_host_schema_with_options(
    db: &DatabaseConnection,
    opts: SchemaApplyOptions,
) -> Result<()>
```

Applies pending host schema with optional in-place backups.

#### Errors

Returns [`LibraryError`](#libraryerror) when a version read, backup, or DDL statement fails.

### current_schema_state

```rust
pub async fn current_schema_state(db: &DatabaseConnection) -> Result<SchemaState>
```

Reads explicit [`SchemaState`](#schemastate) in the Bookclerk host/bootstrap namespace.

An empty `bookclerk_schema_migrations` table is uninitialized; leftover SQLite
`PRAGMA user_version` is ignored.

#### Errors

Returns [`LibraryError::Schema`](#libraryerror) on malformed, partial, or contradictory markers.

### current_schema_state_in

```rust
pub async fn current_schema_state_in(
    db: &DatabaseConnection,
    namespace: &str,
) -> Result<SchemaState>
```

Reads explicit [`SchemaState`](#schemastate) for one ledger `namespace`.

Host library and binding bootstrap use [`BOOKCLERK_SCHEMA_NAMESPACE`](#bookclerk_schema_namespace).
Frozen checksums are verified against [`host_migration_plan`](migrations.md#host_migration_plan) only for
the Bookclerk namespace. Plugin-owned history uses `bookclerk_plugin_migrations`,
not this ledger.

#### Errors

Returns [`LibraryError::Schema`](#libraryerror) on malformed, partial, or contradictory markers.

### current_schema_version

```rust
pub async fn current_schema_version(db: &DatabaseConnection) -> Result<i64>
```

Frozen revision when the database is [`SchemaState::Frozen`](#schemastate); otherwise `0`.

This is **not** a discriminator for uninitialized vs unreleased. Prefer
[`current_schema_state`](#current_schema_state). `0` means “no frozen revision,” never “applied
development schema.”

#### Errors

Returns when the state query fails or the marker is malformed.

### ensure_restore_target_is_replaceable

```rust
pub async fn ensure_restore_target_is_replaceable(db: &DatabaseConnection) -> Result<()>
```

Fails closed when this binary cannot interpret the target database's
schema history well enough to drop every unknown table before restore.

Uninitialized targets are replaceable. Frozen history newer than this
binary, unreleased checksums this binary does not know, and unreadable
markers are not.

#### Errors

Returns [`LibraryError::Schema`](#libraryerror) when the target is newer or uninterpretable.

### migrate_host_schema_to

```rust
pub async fn migrate_host_schema_to(
    db: &DatabaseConnection,
    target: i64,
    opts: SchemaApplyOptions,
) -> Result<SchemaWalk>
```

Migrates toward `target`, including reversible downs for CLI rollback.

#### Errors

Returns [`LibraryError`](#libraryerror) when the walk cannot start, a backup fails, or DDL fails.
A blocked downgrade still applies every reversible step and returns the walk
(the caller treats `blocked` / `stopped_at != target` as failure).

### migrate_host_schema_to_with_batch

```rust
pub async fn migrate_host_schema_to_with_batch<F, Fut>(
    db: &DatabaseConnection,
    target: i64,
    opts: SchemaApplyOptions,
    run_batch: F,
) -> Result<SchemaWalk>
where
    F: FnMut(Vec<String>) -> Fut,
    Fut: Future<Output = Result<()>>,
```

Migrates toward `target` using `run_batch` (typed `executeAtomic`) per version.

A walk that writes anything (backup or DDL) runs under the
[`MIGRATION_LEASE`](control_plane.md#migration_lease) library lease, like a start that migrates: it waits
up to [`MIGRATION_LEASE_WAIT`](#migration_lease_wait) for another migrator, renews the lease
while it works, and checks it before every DDL batch. A no-op walk only
checks the database and takes no lease.

#### Errors

Returns [`LibraryError`](#libraryerror) when the walk cannot start, a backup fails, a
batch fails, another migrator holds the lease past the wait, or the lease
is lost mid-walk.

### require_schema_migrations

```rust
pub fn require_schema_migrations(caps: &DbCapabilities) -> Result<()>
```

Requires the `schemaMigrations` versioning contract from typed
[`bookclerk_plugin_abi::DbCapabilities`](../bookclerk_plugin_abi/db_execute.md#dbcapabilities).

The host always records versions in canonical `bookclerk_schema_migrations`
rows; canonical Bookclerk SQL is [`crate::migrations::current_canonical_schema`](migrations.md#current_canonical_schema)
and adapters lower it for the live connection backend at execution. Plugin
identity and diagnostic engine names are not consulted.

#### Errors

Returns [`LibraryError::Other`](#libraryerror) when `schemaMigrations` is missing.

### schema_state_compatible

```rust
pub fn schema_state_compatible(state: &SchemaState) -> bool
```

True when `state` is exactly the schema this binary applies: a start
against it neither migrates nor fails closed.

### apply_plugin_migrations

```rust
pub async fn apply_plugin_migrations(
    db: &DatabaseConnection,
    registered: &PluginMigrationSequence,
) -> Result<PluginMigrationHistory>
```

Applies the pending registered suffix. Idempotent when history already matches.

Outer loop walks durable history (exact-prefix). Each pending migration is
one short atomic unit: serialization-slot mutation + plugin operations +
journal append, with a bounded inner retry loop that does **not** consume
later migration capacity. There is no lock held across those independent
transactions. Concurrent hosts re-read durable history before applying;
uniqueness / conflict / an ambiguous reply causes another journal read.
Matching `(id, checksum)` at the expected ordinal is success.
Contradictory history fails closed.

#### Errors

Returns when prefix verification fails, apply exhausts retries, or the
journal is contradictory.

### history_from_execute_reply

```rust
pub fn history_from_execute_reply(reply: &ExecuteReply) -> Result<PluginMigrationHistory>
```

Parses journal rows from a typed select reply.

#### Errors

Returns when row shape or ordinals are invalid.

### load_plugin_migration_history

```rust
pub async fn load_plugin_migration_history(
    db: &DatabaseConnection,
) -> Result<PluginMigrationHistory>
```

Reads the host-private plugin migration journal.

#### Errors

Returns when the journal cannot be read or ordinals are not contiguous.

### next_pending_plugin_migration

```rust
pub fn next_pending_plugin_migration<'a>(
    history: &PluginMigrationHistory,
    registered: &'a PluginMigrationSequence,
) -> Result<Option<(usize, &'a ProvenPluginMigration)>>
```

Next registered migration after a verified durable prefix, if any.

Uses [`require_history_prefix`](#require_history_prefix) then indexes `registered[history.len()]`
rather than rebuilding the pending suffix vector.

#### Errors

Returns [`require_history_prefix`](#require_history_prefix) errors.

### plugin_apply_statements

```rust
pub fn plugin_apply_statements(
    ordinal: i64,
    migration: &ProvenPluginMigration,
) -> Vec<String>
```

One atomic apply unit: serialization slot, registered ops, host journal append.

### plugin_binding_type_env

```rust
pub fn plugin_binding_type_env(
    registered: &PluginMigrationSequence,
    applied: usize,
) -> SqlTypeEnv
```

Binding catalog after host bootstrap plus schema ops from the first `applied`
registered migrations.

Plugin apply stamps journal DML and suffix data ops against this evolving
environment. `applied` is the durable prefix length (host-private ordinal
count), not a plugin version.

### plugin_history_session_matches

```rust
pub fn plugin_history_session_matches(
    expected_digest: &str,
    observed: &PluginMigrationHistory,
) -> Result<()>
```

True when a binding session's captured history digest still matches durable history.

#### Errors

Returns when the digest differs (peer advanced or rewritten history).

### plugin_journal_has_entry

```rust
pub fn plugin_journal_has_entry(
    history: &PluginMigrationHistory,
    ordinal: i64,
    id: &str,
    checksum: &str,
) -> bool
```

True when `history` already contains this `(ordinal, id, checksum)` row.

### plugin_journal_select_request

```rust
pub fn plugin_journal_select_request(
    operation_id: impl Into<String>,
    deadline_unix_ms: u64,
) -> ExecuteRequest
```

Typed SELECT used by RPC hosts to read the plugin migration journal.

### prove_plugin_migration_sequence

```rust
pub fn prove_plugin_migration_sequence(
    migrations: Vec<PluginMigration>,
) -> Result<PluginMigrationSequence>
```

Proves a complete registration with one evolving [`SqlTypeEnv`](../bookclerk_plugin_abi/sql_types.md#sqltypeenv).

Schema operations update the environment. Data operations typecheck against
the current environment. The environment is not reset between migrations.

#### Errors

Returns when resource limits are exceeded, an id is invalid, ids repeat,
an operation is empty/malformed, or BookclerkSQL proof fails.
Transport/resource validation is
[`bookclerk_plugin_abi::require_plugin_migration_registration`](../bookclerk_plugin_abi/README.md#require_plugin_migration_registration);
this function is the authoritative semantic proof afterward.

### require_history_prefix

```rust
pub fn require_history_prefix(
    history: &PluginMigrationHistory,
    registered: &PluginMigrationSequence,
) -> Result<()>
```

Fail closed unless durable history is an exact prefix of `registered`.

#### Errors

Returns when an applied id/checksum/order does not match, or durable history
is longer than the current registration (older/incompatible plugin).

### sql_string_literal

```rust
pub fn sql_string_literal(value: &str) -> String
```

SQL string literal with `'` doubled.

### branch_request_dedup_key

```rust
pub fn branch_request_dedup_key(node_id: &str, work_id: &str) -> String
```

Dedup key of a [`JobKind::BranchRequest`]: one active request per
`(branch node, work)`.

### catalog_subscribers_for_event

```rust
pub fn catalog_subscribers_for_event(
    catalog: &[EventSubscriberCatalogRecord],
    event: &DomainEventRecord,
) -> Vec<EventSubscriber>
```

Enabled catalog rows that match `event` type, schema version, and filter.

### collapse_live_subscriber_nodes

```rust
pub fn collapse_live_subscriber_nodes(
    rows: &[EventSubscriberNodeRecord],
) -> Vec<EventSubscriberCatalogRecord>
```

Collapse live per-node rows into one catalog record per `plugin_id`.

`enabled` is true when any live row is enabled. Subscriptions are the union
of enabled live rows (disabled nodes do not contribute filters).

### content_kind_from_classic

```rust
pub fn content_kind_from_classic(content_type: i64) -> String
```

Map classic EF `ContentType` to bookclerk `content_kind`.

### content_kind_to_classic

```rust
pub fn content_kind_to_classic(content_kind: &str) -> i32
```

Classic EF `ContentType` enum value.

### event_filter_matches

```rust
pub fn event_filter_matches(filter: Option<&Value>, payload_json: &str) -> bool
```

True when `filter` is absent/empty or every filter key equals the payload object value.

### event_matches_wake_grants

```rust
pub fn event_matches_wake_grants(
    grants_json: &str,
    legacy_filter_json: &str,
    schema_version: i64,
    payload: &str,
) -> bool
```

True when an event matches persisted wake grants, or legacy type+filter rows.

### intersect_event_filters

```rust
pub fn intersect_event_filters(
    declared: Option<&Value>,
    requested: Option<&Value>,
) -> Option<Value>
```

Merge declared and requested payload filters; `None` on a same-key conflict.

### is_downloadable

```rust
pub fn is_downloadable(content_kind: &str) -> bool
```

True when the title can be downloaded (classic `WithoutParents`).

### is_valid_queue_name

```rust
pub fn is_valid_queue_name(name: &str) -> bool
```

True when `name` is `[a-z0-9][a-z0-9._-]*` and at most [`MAX_QUEUE_NAME_LEN`](#max_queue_name_len) chars.

Host-reserved names (`core.events`, `node.<id>`) satisfy this grammar; the
manifest validator, not the outbox, keeps plugins off those prefixes.

### job_backoff_run_after

```rust
pub fn job_backoff_run_after(attempt_count: i64, now: DateTime<Utc>) -> DateTime<Utc>
```

Next `run_after` after a failed attempt, exponential from a 5s base (capped).

### normalize_theme

```rust
pub fn normalize_theme(raw: &str) -> String
```

Canonical appearance preference; unknown values become `system`.

### plugin_scheduled_dedup_key

```rust
pub fn plugin_scheduled_dedup_key(plugin_instance_id: &str, cron: &str) -> String
```

Dedup key of a [`JobKind::PluginScheduled`](#jobkind) fire: one active fire per
`(instance, cron)`.

### portal_prefs_key

```rust
pub fn portal_prefs_key(identity_id: i64) -> String
```

Subject key for a portal identity (legacy; prefer [`user_prefs_key`](#user_prefs_key)).

### push_queue_wisher

```rust
pub fn push_queue_wisher(wishers: &mut Vec<QueueWisher>, wisher: QueueWisher)
```

Adds `wisher` when they are not already present (capped at [`MAX_QUEUE_WISHERS`](#max_queue_wishers)).

Duplicate people keep the richer picture and name fields.

#### Arguments

* `wishers` - Accumulated people for this work.
* `wisher` - Person who has this title on an open wishlist.

### queue_name_or_bus

```rust
pub fn queue_name_or_bus(name: &str) -> &str
```

Resolves an optional queue name: empty means [`CORE_EVENTS_QUEUE`](#core_events_queue).

### subscription_matches_event

```rust
pub fn subscription_matches_event(
    sub: &EventCatalogSubscription,
    event: &DomainEventRecord,
) -> bool
```

True when one catalog subscription matches `event` type, schema, class, and filter.

### user_prefs_key

```rust
pub fn user_prefs_key(user_id: i64) -> String
```

Subject key for a first-party user.

### wake_grants_from_subscriptions

```rust
pub fn wake_grants_from_subscriptions(
    subs: &[EventCatalogSubscription],
    wake_type: &str,
    requested_filter_json: &str,
) -> Result<Vec<EventWakeGrant>, &'static str>
```

Build wake grants from declared subscriptions for `wake_type`.

Empty `wake_type` yields no grants (timestamp-only). A type with no matching
subscription returns `"wake event type not subscribed"`. When every matching
subscription conflicts with the requested filter, returns
`"wake filter not within subscription"`.

#### Errors

Returns a static reason when the type is undeclared or the requested filter
cannot intersect any declared filter.

### plan_schema_walk

```rust
pub fn plan_schema_walk(
    plan: &[HostMigrationStep],
    from: i64,
    to: i64,
) -> Result<SchemaWalk>
```

Plans ups or downs from `from` toward `to` using `plan`.

Downgrades walk newest-first and stop at the last reversible version when
the next step has `down: None`. Versions newer than the compiled plan
cannot be reversed by this binary. Versions older than the first frozen
step in `plan` (except empty `from == 0`) cannot be upgraded by this
binary. An empty plan means no frozen schema versions exist.

#### Errors

Returns [`LibraryError::Schema`](#libraryerror) when `to` is below zero, `from` is ahead
of every compiled step (unknown newer schema), or `from` is below the
plan's first retained frozen version.

### plan_schema_walk_from_state

```rust
pub fn plan_schema_walk_from_state(
    plan: &[HostMigrationStep],
    state: &SchemaState,
    to: i64,
) -> Result<SchemaWalk>
```

Plans a frozen walk from an explicit [`SchemaState`](#schemastate).

[`SchemaState::Unreleased`](#schemastate) is never treated as empty `from == 0`. A future
freeze must not apply version-1 ups on top of an unreleased development
database — recreate (`cargo reset --yes`) unless a later explicit promote
path is added.

#### Errors

Returns [`LibraryError::Schema`](#libraryerror) when an unreleased database is asked to
become a frozen revision, or when [`plan_schema_walk`](#plan_schema_walk) fails.

### classify_session_client

```rust
pub fn classify_session_client(
    user_agent: Option<&str>,
    is_api: bool,
) -> SessionClientInfo
```

Classify a User-Agent (or missing UA for API/bearer clients).

#### Arguments

* `user_agent` - Optional `User-Agent` header value.
* `is_api` - When true (no browser UA / bearer-style client), label as API.

#### Panics

Panics when an internal invariant does not hold.

### compile_claim_event_delivery

```rust
pub fn compile_claim_event_delivery(
    operation_id: &str,
    delivery_id: &str,
    owner: &str,
    lease_secs: i64,
    plugin_id: &str,
    resource_class: &str,
    max_in_flight: i64,
    now: &str,
) -> Result<CompiledAtomic, DbErr>
```

Compiles a host-side CAS claim for one pending `event_deliveries` row.

#### Errors

Returns `DbErr::Custom` when wrapping the plan fails.

### compile_named_request

```rust
pub fn compile_named_request(
    operation_id: &str,
    params: &DbAtomicParams,
    now: &str,
) -> Result<CompiledAtomic, DbErr>
```

Compiles a named atomic request into canonical Bookclerk SQL.

Adapters lower placeholders and functions at execute time.

#### Errors

Returns `DbErr::Custom` when the named operation is missing or invalid.

### interpret_typed_exec

```rust
pub fn interpret_typed_exec(
    compiled: &CompiledAtomic,
    reply: &ExecuteReply,
    expected_hash: &str,
) -> DbAtomicResult
```

Maps a typed [`ExecuteReply`](../bookclerk_plugin_abi/db_execute.md#executereply) onto a host [`DbAtomicResult`](#dbatomicresult).

### validate_execute_reply

```rust
pub fn validate_execute_reply(
    req: &ExecuteRequest,
    reply: &ExecuteReply,
    caps: &DbCapabilities,
) -> Result<()>
```

Rejects a guest [`ExecuteReply`](../bookclerk_plugin_abi/db_execute.md#executereply) that does not echo the sent request or
exceeds negotiated caps.

A mismatch after the guest reports success is treated as
[`LibraryError::Unavailable`](#libraryerror) so callers retry the same `operationId`
rather than interpreting a truncated or mismatched envelope.

#### Errors

Returns [`LibraryError::Unavailable`](#libraryerror) when the echo, statement count or
result selection shape does not match the request, and
[`LibraryError::LimitExceeded`](#libraryerror) when a statement result or the whole
reply exceeds a negotiated cap (`maxResultRows`, `maxCellBytes`,
`maxResultBytes`, `maxAtomicResultBytes`). A reply read as a stream is
validated with [`validate_streamed_execute_reply`](#validate_streamed_execute_reply) instead.

### validate_streamed_execute_reply

```rust
pub fn validate_streamed_execute_reply(
    req: &ExecuteRequest,
    reply: &ExecuteReply,
    caps: &DbCapabilities,
) -> Result<()>
```

[`validate_execute_reply`](#validate_execute_reply) for a reply reassembled from
`executeStreamed` frames.

`caps` are the streamed caps (`DbCapabilities::streamed`), so the
byte budgets are the streamed budget and an exceeded one is reported
as `maxStreamedResultBytes` whatever its size, which keeps a caller
from retrying the same read in inline pieces.

#### Errors

As [`validate_execute_reply`](#validate_execute_reply).

### facet_key

```rust
pub fn facet_key(value: &str) -> String
```

The key a facet value or tag is stored and matched under: trimmed and
lower-cased, so "Fantasy" and "fantasy " are one value.

#### Arguments

* `value` - A value as shown or as a client sent it.

### sort_title

```rust
pub fn sort_title(title: &str) -> String
```

The sort key of a title: lower case, with one leading English article
dropped, so "The Hobbit" sorts under h.

#### Arguments

* `title` - The display title.

### normalize_work_tags

```rust
pub fn normalize_work_tags(values: &[String]) -> Result<Vec<(String, String)>>
```

Trims, drops empties and duplicates (by [`facet_key`](#facet_key), first spelling
kept) and checks the bounds.

#### Errors

Returns [`LibraryError::Invalid`](#libraryerror) for more than [`MAX_WORK_TAGS`](#max_work_tags) tags or
a tag longer than [`MAX_WORK_TAG_BYTES`](#max_work_tag_bytes).

### parse_work_tags_request

```rust
pub fn parse_work_tags_request(body: &[u8]) -> Result<WorkTagsRequest>
```

Parses and validates a `PUT /api/library/works/{id}/tags` body from its
raw bytes, sans IO, the way the route reads it: JSON into
[`WorkTagsBody`](#worktagsbody), then [`WorkTagsBody::validate`](#worktagsbodyvalidate).

#### Errors

Returns [`LibraryError::Invalid`](#libraryerror) for bytes that are not the body's JSON
shape (unknown or repeated fields, wrong types, a missing field, an
array instead of an object) or tags outside the bounds.

### apply_user_states

```rust
pub fn apply_user_states(
    titles: &mut [TasteTitle],
    states: &[UserTitleState],
    users: &[i64],
)
```

Applies the state of `users` to `titles`, merged per work.

A work is finished when any of the users finished it; each rating is
the mean of the users who gave one; tags are the users' tags joined,
each tag once. A title without state from any of `users` keeps the
unfinished, unrated, untagged default. `users` empty means every user
whose state is in `states` (the household).

#### Arguments

* `titles` - Titles to fill, as [`LibraryStore::taste_titles`](#librarystoretaste_titles) returns
  them.
* `states` - Stored state rows for the titles' works.
* `users` - Whose state counts; empty for everyone's.

### copy_source_id

```rust
pub fn copy_source_id(destination: &str, keys: &[String]) -> String
```

The source id a [`ChapterSource::File`](#chaptersource) claim names one stored copy by:
`copy-` and the first 32 hex digits of the SHA-256 of the destination
and the copy's audio keys in sorted order, so the id has a fixed length
whatever the keys are and does not depend on their play order.

### resolve_chapters

```rust
pub fn resolve_chapters(claims: &[ChapterClaim]) -> Option<&ChapterClaim>
```

Picks the claim an edition's chapters resolve to.

In order: the claim an Administrator pinned; the newest edit made in
Bookclerk; the newest outside edit; then the best machine source by
the chapters machine ranking (`docs/adr/edition-chapters.md`, decision
2), ties broken by source id. A claim whose list write is
unfinished is never picked, and an empty list is picked only from a
person, who cleared the chapters on purpose. The answer depends only on
the claims, never on the order they are given in.

### parse_chapter_edit_request

```rust
pub fn parse_chapter_edit_request(body: &[u8]) -> Result<ChapterList>
```

Parses and checks a chapter edit body from its raw bytes, sans IO, the
way the route reads it.

#### Errors

Returns [`LibraryError::Invalid`](#libraryerror) for a body over
[`MAX_CHAPTER_EDIT_BYTES`](#max_chapter_edit_bytes), bytes that are not the body's JSON shape, or
chapters that break a rule: more than [`MAX_CHAPTERS`](#max_chapters), a title that is
empty, padded, over [`MAX_CHAPTER_TITLE_BYTES`](#max_chapter_title_bytes) or holds a control
character, a start past [`MAX_CHAPTER_START_MS`](#max_chapter_start_ms) or before the one
above, or a depth that skips a level or passes [`MAX_CHAPTER_DEPTH`](#max_chapter_depth).

### parse_chapter_pin_request

```rust
pub fn parse_chapter_pin_request(body: &[u8]) -> Result<ChapterPinRequest>
```

Parses and checks a pin body from its raw bytes, sans IO.

#### Errors

Returns [`LibraryError::Invalid`](#libraryerror) for a body over
[`MAX_CHAPTER_PIN_BYTES`](#max_chapter_pin_bytes), bytes that are not the body's JSON shape, an
unknown source kind, or a source id [`ChapterPinBody::validate`](#chapterpinbodyvalidate)
refuses.

### copy_retry_delay

```rust
pub fn copy_retry_delay(attempts: u32) -> Duration
```

The delay before the next background attempt of a copy that has now
failed `attempts` times (1 for the first failure):
[`COPY_RETRY_BASE_DELAY`](#copy_retry_base_delay) doubled per earlier failure, capped at
[`COPY_RETRY_MAX_DELAY`](#copy_retry_max_delay).

### deliver_after_for

```rust
pub fn deliver_after_for(spec: &PublishDomainEventSpec, now: &str) -> String
```

RFC 3339 earliest delivery time for `spec`; empty when it may deliver now.

### prepare_publish_domain_event

```rust
pub fn prepare_publish_domain_event(
    spec: PublishDomainEventSpec,
) -> Result<PublishDomainEventSpec>
```

Mint an event id when empty and reject oversized or blank publish fields.

#### Errors

Returns [`LibraryError::Other`](#libraryerror) when the payload exceeds 64 KiB, when
`dedup_key` is blank, when a domain event (bus row) has no `event_type`
or a queue message has one, when the queue name or content type is not
recognised, or when `source` is not empty and fails the source grammar.

### fallback_work_key

```rust
pub fn fallback_work_key(
    title: &str,
    authors: Option<&str>,
    asin: Option<&str>,
    isbn: Option<&str>,
) -> String
```

Local fallback when callers do not supply a discover `work_map_key`.

Converts ISBN-10 → ISBN-13 when possible so 10/13 variants share a key.
Soft keys here are a simple lowercase fallback — Discover re-merges with
richer identity matching when ranking the global queue.

### inject_dispatch_page_failures

```rust
pub fn inject_dispatch_page_failures(n: u32)
```

Fail after the next `n` successful dispatch pages (tests only).

### call_number

```rust
pub fn call_number(edition_id: &str) -> Option<String>
```

The `<call number>` of edition `edition_id`: its first 40 bits as 8
lower-case Crockford base32 characters (no `i`, `l`, `o` or `u`). An id
that is not a UUID falls back to its first 8 ASCII letters and digits.
Returns `None` when that leaves nothing.

### edition_id_for_listing

```rust
pub fn edition_id_for_listing(key: &ListingKey) -> String
```

The id of an edition first seen through the store listing `key`: a
version 5 UUID over its plugin key, marketplace and product id. Two
nodes (or one library rebuilt from a fresh scan) that resolve the same
listing first give its edition the same id, and so the same
`<call number>` in file names.

### holding_from_book

```rust
pub fn holding_from_book(book: &NewBook) -> HoldingDraft
```

The holding a flat library row describes.

### listing_from_book

```rust
pub fn listing_from_book(book: &NewBook) -> Result<Listing, ListingError>
```

The structured listing a flat library row describes, for paths that
only have a [`NewBook`](#newbook) (imports, seeds, tests).

Scans that carry a plugin's structured record use that instead.

#### Errors

Returns [`resolve::ListingError`](../bookclerk_resolve/listing.md#listingerror) when the row fails listing validation.

### reaches_end

```rust
pub fn reaches_end(position: PlaybackMs, duration: PlaybackMs) -> bool
```

True when `position` in a recording `duration` long counts as having
reached its end: within the last one percent, or within the last
thirty seconds of a recording longer than a minute.

#### Arguments

* `position` - Where the listener is.
* `duration` - The recording's whole runtime; a zero duration is
  unknown and never finished.

### parse_bookmark_edit_request

```rust
pub fn parse_bookmark_edit_request(body: &[u8]) -> Result<BookmarkEditRequest>
```

Parses and validates a `PUT /api/playback/bookmarks/{id}` body from
its raw bytes, sans IO.

#### Errors

Returns [`LibraryError::Invalid`](#libraryerror) for a body over
[`MAX_PLAYBACK_BODY_BYTES`](#max_playback_body_bytes), bytes that are not the body's JSON shape,
a position past [`MAX_PLAYBACK_MS`](#max_playback_ms) or a refused title.

### parse_new_bookmark_request

```rust
pub fn parse_new_bookmark_request(body: &[u8]) -> Result<NewBookmarkRequest>
```

Parses and validates a `POST /api/library/editions/{id}/bookmarks`
body from its raw bytes, sans IO.

#### Errors

Returns [`LibraryError::Invalid`](#libraryerror) for a body over
[`MAX_PLAYBACK_BODY_BYTES`](#max_playback_body_bytes), bytes that are not the body's JSON shape,
a position past [`MAX_PLAYBACK_MS`](#max_playback_ms) or a refused title.

### parse_open_session_request

```rust
pub fn parse_open_session_request(body: &[u8]) -> Result<OpenSessionRequest>
```

Parses and validates a `POST /api/playback/sessions` body from its raw
bytes, sans IO: JSON into [`OpenSessionBody`](#opensessionbody), then
[`OpenSessionBody::validate`](#opensessionbodyvalidate).

#### Errors

Returns [`LibraryError::Invalid`](#libraryerror) for a body over
[`MAX_PLAYBACK_BODY_BYTES`](#max_playback_body_bytes), bytes that are not the body's JSON shape
(unknown or repeated fields, wrong types, a missing `edition_id`, an
unknown `player`, an array instead of an object) or an edition id that
is empty or too long.

### parse_playback_sync_request

```rust
pub fn parse_playback_sync_request(body: &[u8], close: bool) -> Result<PlaybackSync>
```

Parses and validates a sync or close body from its raw bytes, sans IO,
the way the routes read it: JSON into [`PlaybackSyncBody`](#playbacksyncbody), then
[`PlaybackSyncBody::validate`](#playbacksyncbodyvalidate).

#### Arguments

* `body` - The request body.
* `close` - True for the `close` route.

#### Errors

Returns [`LibraryError::Invalid`](#libraryerror) for a body over
[`MAX_PLAYBACK_SYNC_BYTES`](#max_playback_sync_bytes), bytes that are not the body's JSON shape
(unknown or repeated fields, wrong types, a missing `position_ms`, an
array instead of an object) or a length past [`MAX_PLAYBACK_MS`](#max_playback_ms).

### parse_progress_cursor

```rust
pub fn parse_progress_cursor(token: &str) -> Option<ProgressCursor>
```

Parses the `cursor` of `GET /api/playback/progress`, sans IO.

#### Arguments

* `token` - The query value as sent.

#### Returns

`None` unless `token` is at most [`MAX_PROGRESS_CURSOR_BYTES`](#max_progress_cursor_bytes) of
canonical unpadded URL-safe Base64 that decodes to UTF-8 text of a
change time (see [`progress_cursor_token`](#progress_cursor_token)), a `|` and an edition id
of 1 to [`MAX_EDITION_ID_BYTES`](#max_edition_id_bytes) bytes. An accepted token is exactly
what [`progress_cursor_token`](#progress_cursor_token) writes for the cursor it yields.

### parse_progress_edit_request

```rust
pub fn parse_progress_edit_request(body: &[u8]) -> Result<PlaybackProgressEdit>
```

Parses and validates a `PUT /api/playback/progress/{edition_id}` body
from its raw bytes, sans IO.

#### Errors

Returns [`LibraryError::Invalid`](#libraryerror) for a body over
[`MAX_PLAYBACK_BODY_BYTES`](#max_playback_body_bytes), bytes that are not the body's JSON shape
or a length past [`MAX_PLAYBACK_MS`](#max_playback_ms).

### parse_track_path

```rust
pub fn parse_track_path(session_id: &str, index: &str) -> Option<TrackPath>
```

Parses the two path parts of
`GET /api/playback/sessions/{id}/tracks/{n}`, sans IO.

#### Arguments

* `session_id` - The `{id}` part as sent.
* `index` - The `{n}` part as sent.

#### Returns

`None` unless `session_id` is a UUID in its canonical spelling (36
bytes, lowercase, hyphenated) and `index` is a decimal number below
[`MAX_PLAYBACK_TRACKS`](#max_playback_tracks) written with ASCII digits only and no leading
zero. Anything else names no session or track, and the route answers
404 without reading the library.

### progress_cursor_token

```rust
pub fn progress_cursor_token(cursor: &ProgressCursor) -> String
```

The opaque token of a progress cursor: unpadded URL-safe Base64 of
the row's change time, a `|`, and its edition id.

### plan_pass

```rust
pub fn plan_pass(state: Option<&ScanState>, now: DateTime<Utc>) -> PassStart
```

Decides how the next pass of an account starts.

A stopped pass whose cursor is younger than [`RESUME_CURSOR_TTL`](#resume_cursor_ttl) resumes
with the token and pass id it started with (a row without a pass id
starts over, since its earlier pages stamped no holdings). Otherwise a pass starts from the
beginning, as a delta from the stored token unless no complete pass ended
within [`FULL_PASS_INTERVAL`](#full_pass_interval), in which case it is full.

#### Arguments

* `state` - The stored row, `None` before the account's first pass.
* `now` - The current time.

### set_dispatch_chunk_for_test

```rust
pub fn set_dispatch_chunk_for_test(chunk: Option<usize>)
```

Override subscribers-per-plan for dispatch paging tests.

### parse_work_split_request

```rust
pub fn parse_work_split_request(body: &[u8]) -> Result<WorkSplitRequest>
```

Parses and validates a `POST /api/library/works/{id}/split` body from
its raw bytes, sans IO, the way the route reads it: JSON into
[`WorkSplitBody`](#worksplitbody), then [`WorkSplitBody::validate`](#worksplitbodyvalidate).

#### Errors

Returns [`LibraryError::Invalid`](#libraryerror) for bytes that are not the body's JSON
shape (unknown or repeated fields, wrong types, a missing field, an
array instead of an object) or ids outside the bounds.

### derive_claim_password_fingerprint

```rust
pub fn derive_claim_password_fingerprint(
    dek: &MasterKey,
    nonce: &str,
    password: &str,
) -> String
```

Stable, non-secret fingerprint of an invite/reset password for idempotency.

Argon2id storage hashes use a fresh salt on every POST, so they cannot be
the atomic request hash. This value is
`HMAC-SHA256(subkey, nonce || 0x00 || password)` where
`subkey = HMAC-SHA256(DEK, "bookclerk-claim-password-subkey-v1")`.
The randomized Argon2id hash is still what gets stored on the user row.

### derive_claim_session_token

```rust
pub fn derive_claim_session_token(
    dek: &MasterKey,
    raw_ticket: &str,
    nonce: &str,
) -> String
```

Derive the portal session token for a claim ticket plus browser nonce.

The raw session is
`HMAC-SHA256(DEK, "bookclerk-claim-session-v1:" || ticket || 0x00 || nonce)`
hex-encoded (64 characters). The same ticket, nonce, and process DEK always
yield the same session, so a browser retry after a lost atomic reply
reuses `redeemClaimTicket:{token}:{session}`. A different nonce (another
browser, or the magic link alone) cannot replay that receipt. The plaintext
bearer is never written to a receipt; only [`hash_token`](#hash_token) of this value is
stored.

#### Arguments

* `dek` - Process data-encryption key from [`crate::require_master_key`](master_key.md#require_master_key).
* `raw_ticket` - Plaintext claim ticket from the SPA / share URL.
* `nonce` - Validated 64-hex redeem nonce from [`parse_claim_redeem_nonce`](#parse_claim_redeem_nonce).

### hash_token

```rust
pub fn hash_token(raw: &str) -> String
```

SHA-256 hex digest of a raw ticket/session token for durable storage.

#### Arguments

* `raw` - Opaque plaintext token; callers must not persist this value.

#### Returns

Lowercase hex SHA-256 digest (64 characters).

### parse_claim_redeem_nonce

```rust
pub fn parse_claim_redeem_nonce(raw: &str) -> Result<&str>
```

Accept a browser-generated claim-redeem nonce.

The SPA persists a 32-byte random value as 64 hex characters across HTTP
retries (`sessionStorage`). Possession of the used magic-link URL without
this nonce cannot recover the committed session.

#### Errors

Returns an error when the value is missing, the wrong length, or not hex.

### apply_merged_sources

```rust
pub fn apply_merged_sources(row: &mut TitleRequestRecord)
```

Apply merged bib / editions / hints onto a wishlist row from its source snapshots.

Updates `row` in place from `row.sources`, clearing merged fields when no
sources remain.

#### Arguments

* `row` - Wishlist title-request record whose `sources` snapshots are merged.

### pick_better_description

```rust
pub fn pick_better_description(a: Option<&str>, b: Option<&str>) -> Option<String>
```

Prefer HTML blurbs over plain text; otherwise the longer string wins.

#### Arguments

* `a` - First candidate description (may be HTML or plain).
* `b` - Second candidate description.

#### Returns

The richer non-empty description, or `None` when both are empty.

## Type aliases

### Result

```rust
pub type Result<T> = Result<T, LibraryError>;
```

Result alias for [`LibraryError`](#libraryerror).

### InProcessSqliteAtomic

```rust
pub type InProcessSqliteAtomic = NamedAtomicBackend<InProcessSqlite>;
```

The [`crate::AtomicTxnBackend`](#atomictxnbackend) over a local SQLite connection.

## Constants

### LEFT_BEHIND_TABLES

```rust
pub const LEFT_BEHIND_TABLES: &[(&str, &str)] = _;
```

Tables the copy leaves behind on purpose, each with the reason.

Every other table of the library schema is copied; a test checks that
the two lists together cover the schema.

### CHUNK_TARGET_UNCOMPRESSED_BYTES

```rust
pub const CHUNK_TARGET_UNCOMPRESSED_BYTES: usize = _;
```

Target uncompressed JSON size for one table-data chunk.

256 KiB sits in a 128–512 KiB band: large enough to avoid pathological
per-object overhead on typical catalog rows, small enough for a small-VPS
working set and for `maxResultBytes` paging. A single row larger than the
target still occupies its own chunk.

### BACKUPS_DIR

```rust
pub const BACKUPS_DIR: &str = "backups";
```

Subdirectory of `$BOOKCLERK_FILES_DIR` holding the backup repository.

### BACKUP_FORMAT_VERSION

```rust
pub const BACKUP_FORMAT_VERSION: u32 = 1;
```

Manifest format written by this binary.

### BACKUP_RETENTION

```rust
pub const BACKUP_RETENTION: usize = 5;
```

Automatic `pre-migrate` recovery points retained after prune.

### LIBRARY_SKIP_TABLES

```rust
pub const LIBRARY_SKIP_TABLES: &[&str] = _;
```

Library registry rows are environment-local (`unit_ref` is not portable).

### MIGRATION_LEASE_TTL

```rust
pub const MIGRATION_LEASE_TTL: Duration = _;
```

How long a schema migration lease lasts without renewal. The migrator
renews it every third of this while it works.

### MIGRATION_LEASE_WAIT

```rust
pub const MIGRATION_LEASE_WAIT: Duration = _;
```

How long a starter waits for another starter's migration before it fails
closed. Longer than [`MIGRATION_LEASE_TTL`](#migration_lease_ttl), so a migrator that died is
replaced by a waiter rather than failing every waiter.

### BOOKCLERK_SCHEMA_NAMESPACE

```rust
pub const BOOKCLERK_SCHEMA_NAMESPACE: &str = "bookclerk";
```

Ledger namespace for host library schema and Bookclerk-owned binding bootstrap.

### MAX_PLUGIN_MIGRATION_APPLY_ATTEMPTS

```rust
pub const MAX_PLUGIN_MIGRATION_APPLY_ATTEMPTS: usize = 8;
```

Bounded retries for **one** expected plugin migration apply unit.

Independent of how many migrations are registered. A fresh binding may
apply the complete registration; this only caps retries of a single
expected `(ordinal, id, checksum)` while durable history is unchanged.

### CORE_EVENTS_QUEUE

```rust
pub const CORE_EVENTS_QUEUE: &str = "core.events";
```

Queue that carries the domain-event bus (`[events]` producers and consumers).

### EDITION_ACQUIRE_CHANGED

```rust
pub const EDITION_ACQUIRE_CHANGED: &str = "edition_acquire_changed";
```

Host event type published with an edition's acquire state change other
than becoming acquired (payload `editionId`, `titleId`, `status`).

### EVENT_DELIVERY_DEFAULT_RETENTION_SECS

```rust
pub const EVENT_DELIVERY_DEFAULT_RETENTION_SECS: u32 = 86_400;
```

How long a delivery may wait, due but never leased, before the host
expires it when its consumer declares no `message_retention_period`: one
day. A consumer that is deployed but not running stops accumulating work
after this, and catches up through the event stream instead.

### EVENT_DELIVERY_MAX_ATTEMPTS

```rust
pub const EVENT_DELIVERY_MAX_ATTEMPTS: i64 = 8;
```

Default max delivery attempts before dead-letter.

### EVENT_RESOURCE_CLASS_NETWORK

```rust
pub const EVENT_RESOURCE_CLASS_NETWORK: &str = "network";
```

Default delivery resource class (`network`).

### EVENT_SUBSCRIBER_HEARTBEAT_TTL_SECS

```rust
pub const EVENT_SUBSCRIBER_HEARTBEAT_TTL_SECS: i64 = 60;
```

Live catalog rows quieter than this many seconds are ignored at dispatch.

### HOST_CONSUMER_PREFIX

```rust
pub const HOST_CONSUMER_PREFIX: &str = "host:";
```

Prefix of subscriber ids the daemon itself consumes under (`host:discover`,
`host:search-index:<node>`). Their deliveries never expire.

### INGEST_BATCH_FINISHED

```rust
pub const INGEST_BATCH_FINISHED: &str = "ingest_batch_finished";
```

Host event type published when an ingest batch (an upload or a storage
intake) reaches a final state (payload `batchId`, `state`, `source`,
`intakeInstance`, `detail`).

### JOB_PAYLOAD_VERSION

```rust
pub const JOB_PAYLOAD_VERSION: u32 = 1;
```

Current job command envelope version stored in [`JobPayload::v`](#jobpayload).

### LISTENING_SYNCED

```rust
pub const LISTENING_SYNCED: &str = "listening_synced";
```

Host event published when a listening sync changed what Discover reads
from a stored row: its owner, matched book or work, authors, or how far
they got (started, quarter, finished).

Payload: `provider` (the integration that synced) and `identityIds` (the
portal identities whose rows changed, before or after; rows not tied to
an identity are not listed).

### MAX_QUEUE_NAME_LEN

```rust
pub const MAX_QUEUE_NAME_LEN: usize = 63;
```

Longest queue name.

### MAX_QUEUE_WISHERS

```rust
pub const MAX_QUEUE_WISHERS: usize = 16;
```

Maximum distinct people serialized on a global-queue or Discover card.

### NODE_QUEUE_PREFIX

```rust
pub const NODE_QUEUE_PREFIX: &str = "node.";
```

Prefix of the per-node queues (`node.<node_id>`) node subsystems consume.

### OPERATOR_PREFS_KEY

```rust
pub const OPERATOR_PREFS_KEY: &str = "operator";
```

Subject key for the shared operator account.

### QUEUE_CONTENT_TYPE_BYTES

```rust
pub const QUEUE_CONTENT_TYPE_BYTES: &str = "bytes";
```

Queue message body is raw bytes stored as base64 text.

### QUEUE_CONTENT_TYPE_JSON

```rust
pub const QUEUE_CONTENT_TYPE_JSON: &str = "json";
```

Queue message body is UTF-8 JSON text.

### QUEUE_CONTENT_TYPE_TEXT

```rust
pub const QUEUE_CONTENT_TYPE_TEXT: &str = "text";
```

Queue message body is plain UTF-8 text.

### WISHLIST_CHANGED

```rust
pub const WISHLIST_CHANGED: &str = "wishlist_changed";
```

Host event published when a wishlist entry is added or changes status, or
an open entry is deleted with its user.

Payload: `requestId` (the entry's uuid), `identityId` (the portal identity
that wished for it, or null) and `status` after the write (`open`,
`cancelled`, or `deleted`).

### SCHEMA_STATE_FROZEN

```rust
pub const SCHEMA_STATE_FROZEN: &str = "frozen";
```

On-disk `bookclerk_schema_migrations.state` for a frozen plan version.

### SCHEMA_STATE_UNRELEASED

```rust
pub const SCHEMA_STATE_UNRELEASED: &str = "unreleased";
```

On-disk `bookclerk_schema_migrations.state` for an unreleased development pack.

### EVENT_STREAM_SCAN_FACTOR

```rust
pub const EVENT_STREAM_SCAN_FACTOR: usize = 8;
```

Rows one [`LibraryStore::read_event_stream`](#librarystoreread_event_stream) call may examine per event
it may return.

### EVENT_STREAM_SETTLE_SECS

```rust
pub const EVENT_STREAM_SETTLE_SECS: u64 = 30;
```

Seconds an outbox row must age before [`LibraryStore::read_event_stream`](#librarystoreread_event_stream)
returns it. `created_at` is stamped before the insert commits, so a
publication can commit after a newer one; a read that stops short of the
last few seconds never moves its cursor past a row that has yet to
appear. Newer events still reach subscribers through delivery.

### LIBRARY_INDEX_LAYOUT

```rust
pub const LIBRARY_INDEX_LAYOUT: i64 = 2;
```

Layout of `library_work_rows` this binary writes. A library with no
finished build of this layout is rebuilt; bump it when a change to how
rows are derived needs every row rewritten.

- 2: every work with an edition names a cover edition, whose cover may
  come from a store listing or the work, so libraries indexed under 1
  (no cover edition unless the edition row had a cover) are rebuilt.

### MAX_WORK_TAGS

```rust
pub const MAX_WORK_TAGS: usize = 32;
```

Most tags an Administrator may set on one work. With the delete this
fits one batch of the smallest statement count a database plugin may
advertise (40).

### MAX_WORK_TAG_BYTES

```rust
pub const MAX_WORK_TAG_BYTES: usize = 256;
```

Longest tag, in UTF-8 bytes after trimming: the longest value the page
index keeps under a facet, and the longest value a scope rule matches.

### CATALOG_LISTING_REFRESH

```rust
pub const CATALOG_LISTING_REFRESH: Duration = _;
```

How long a catalog listing counts as current: a store answer for a
listing seen more recently than this refreshes only its prices, not
its resolution or text, so overlapping tastes cost one resolve a day.

### COPY_RUNTIME_TOLERANCE_MS

```rust
pub const COPY_RUNTIME_TOLERANCE_MS: u64 = 2_000;
```

Most a stored copy's running time may differ from the resolved list's
before the copy plays the chapters read from its own files, in
milliseconds (`docs/adr/edition-chapters.md`, decision 1).

### MAX_CHAPTERS

```rust
pub const MAX_CHAPTERS: usize = 2000;
```

Most chapters one list holds, the same cap as a metadata file's
(`maxMetadataFileChapters` in `plugin.capnp`).

### MAX_CHAPTER_DEPTH

```rust
pub const MAX_CHAPTER_DEPTH: u32 = 7;
```

Deepest nesting stored; 0 is a top-level chapter. A machine source's
deeper chapter is lifted to this depth; an edit's is refused.

### MAX_CHAPTER_START_MS

```rust
pub const MAX_CHAPTER_START_MS: u64 = crate::store::playback::MAX_PLAYBACK_MS;
```

Latest chapter start stored, in milliseconds: sixty days, the playback
core's longest position ([`crate::MAX_PLAYBACK_MS`](#max_playback_ms)).

### MAX_CHAPTER_TITLE_BYTES

```rust
pub const MAX_CHAPTER_TITLE_BYTES: usize = 512;
```

Longest chapter title stored, in UTF-8 bytes. A machine source's longer
title is cut at a character boundary; an edit's is refused.

### MAX_CHAPTER_EDIT_BYTES

```rust
pub const MAX_CHAPTER_EDIT_BYTES: usize = _;
```

Largest chapter edit body the route reads, in bytes: room for
[`MAX_CHAPTERS`](#max_chapters) chapters with titles at [`MAX_CHAPTER_TITLE_BYTES`](#max_chapter_title_bytes),
some of them escaped.

### MAX_CHAPTER_PIN_BYTES

```rust
pub const MAX_CHAPTER_PIN_BYTES: usize = 1024;
```

Largest pin body the route reads, in bytes.

### MAX_CHAPTER_SOURCE_ID_BYTES

```rust
pub const MAX_CHAPTER_SOURCE_ID_BYTES: usize = 256;
```

Longest `source_id` a pin names, in bytes; a PluginKey is far shorter.

### MAX_CONFLICT_PAGE

```rust
pub const MAX_CONFLICT_PAGE: u64 = 200;
```

Largest conflicts page [`LibraryStore::list_resolution_conflicts`](#librarystorelist_resolution_conflicts) reads.

### MAX_BOOKMARKS_PER_EDITION

```rust
pub const MAX_BOOKMARKS_PER_EDITION: u64 = 200;
```

Most bookmarks one person keeps in one edition.

### MAX_BOOKMARK_TITLE_BYTES

```rust
pub const MAX_BOOKMARK_TITLE_BYTES: usize = 200;
```

Longest bookmark title, in bytes (well inside the minimum database
profile's cell size).

### MAX_PLAYBACK_MS

```rust
pub const MAX_PLAYBACK_MS: u64 = _;
```

The longest position or duration the player stores: sixty days, far
past any recording, and small enough that sums of two stay exact in
every database profile's integer.

### MAX_PLAYER_NAME_BYTES

```rust
pub const MAX_PLAYER_NAME_BYTES: usize = 64;
```

Longest player name a session stores, in bytes.

### MAX_EDITION_ID_BYTES

```rust
pub const MAX_EDITION_ID_BYTES: usize = 128;
```

Longest edition id a session is opened for, in bytes; a real one is a
36-byte UUID.

### MAX_PLAYBACK_BODY_BYTES

```rust
pub const MAX_PLAYBACK_BODY_BYTES: usize = 4096;
```

Largest body the other playback parsers read, in bytes: room for an
escaped bookmark title or a long player name.

### MAX_PLAYBACK_SYNC_BYTES

```rust
pub const MAX_PLAYBACK_SYNC_BYTES: usize = 1024;
```

Largest sync body the routes read, in bytes; a real one is under 100.

### MAX_PLAYBACK_TRACKS

```rust
pub const MAX_PLAYBACK_TRACKS: usize = 512;
```

Most tracks a play session lists: a stream path's index is below it.

### MAX_PROGRESS_CURSOR_BYTES

```rust
pub const MAX_PROGRESS_CURSOR_BYTES: usize = 256;
```

Longest progress cursor token the route reads, in bytes; a real one is
about 100.

### MAX_PLAYER_PASSWORDS

```rust
pub const MAX_PLAYER_PASSWORDS: u64 = 20;
```

Most player passwords one patron keeps.

### MAX_PLAYER_PASSWORD_LABEL_BYTES

```rust
pub const MAX_PLAYER_PASSWORD_LABEL_BYTES: usize = 64;
```

Longest player password label, in bytes.

### PLAYER_ACCESS_TTL_SECS

```rust
pub const PLAYER_ACCESS_TTL_SECS: i64 = _;
```

Lifetime of a player card's access token.

### PLAYER_REFRESH_GRACE_SECS

```rust
pub const PLAYER_REFRESH_GRACE_SECS: i64 = _;
```

How long the other refresh tokens of a card keep working after one of
them is used: long enough for two refreshes that raced with the same
token to both succeed, and for a phone whose refresh answer was lost
while it slept to try again (the apps expect ten minutes).

### PLAYER_REFRESH_TTL_DAYS

```rust
pub const PLAYER_REFRESH_TTL_DAYS: i64 = 30;
```

Lifetime of a player card's refresh token; each refresh starts it again.

### KV_LIST_DEFAULT_LIMIT

```rust
pub const KV_LIST_DEFAULT_LIMIT: u32 = 100;
```

Default page size for [`LibraryStore::kv_list`](#librarystorekv_list) when the caller passes 0.

### AUDIT_RETENTION_DAYS

```rust
pub const AUDIT_RETENTION_DAYS: i64 = 365;
```

Days `security_audit_events` and `configuration_audit` rows are kept.

### CONFIGURATION_CHANGE_RETENTION_DAYS

```rust
pub const CONFIGURATION_CHANGE_RETENTION_DAYS: i64 = 30;
```

Days `configuration_changes` notices are kept. They only signal that a
document changed; the document and its audit row hold the content.

### STORAGE_SCAN_STALE_DAYS

```rust
pub const STORAGE_SCAN_STALE_DAYS: i64 = 1;
```

Days without a heartbeat after which a jobless storage-scan generation
(a CLI scan) is treated as abandoned.

### TOKEN_HISTORY_DAYS

```rust
pub const TOKEN_HISTORY_DAYS: i64 = 30;
```

Days an expired claim ticket, user invite, or node enrollment link stays
listed (as expired or redeemed) before it is deleted.

### UPLOAD_HISTORY_DAYS

```rust
pub const UPLOAD_HISTORY_DAYS: i64 = 1;
```

Days a committed or aborted host upload is kept past its token's expiry.

No request can use it after expiry; the day only spares a commit replay
racing the expiry a confusing "unknown upload". Open uploads are never
deleted here: an abort still needs the row to tell the receiving node
which copies to delete.

### FULL_PASS_INTERVAL

```rust
pub const FULL_PASS_INTERVAL: Duration = _;
```

Longest time since the last complete pass before the host asks for a full
pass again, so a store whose deltas miss edits or removals is corrected.

### RESUME_CURSOR_TTL

```rust
pub const RESUME_CURSOR_TTL: Duration = _;
```

Longest time a stopped pass's cursor is resumed; an older one restarts
the pass (store cursors expire).

### MAX_SPLIT_EXPRESSIONS

```rust
pub const MAX_SPLIT_EXPRESSIONS: usize = 12;
```

Most expressions one split moves: the batch's widest statement binds
each id twice, and the minimum database profile allows 32 binds.

### MAX_SPLIT_EXPRESSION_ID_BYTES

```rust
pub const MAX_SPLIT_EXPRESSION_ID_BYTES: usize = 64;
```

Longest expression id a split body may name, in bytes; ids are UUIDs.

### ACQUIRED_KEYS_MAX_PAGE

```rust
pub const ACQUIRED_KEYS_MAX_PAGE: u64 = 256;
```

Upper clamp for the editions one [`LibraryStore::acquired_title_keys_page`](#librarystoreacquired_title_keys_page)
read covers.

An acquisition row is a few short columns (well under 1 KiB on the wire),
so this many stay far below the 256 KiB inline result cap of an adapter
that does not stream.

### ACQUIRE_WRITER

```rust
pub const ACQUIRE_WRITER: &str = "acquire";
```

The `writer` of a key an acquire run on this node wrote.

### COPY_RETRY_BASE_DELAY

```rust
pub const COPY_RETRY_BASE_DELAY: Duration = _;
```

Delay before the first background retry of a failed copy; each later
failure doubles it, up to [`COPY_RETRY_MAX_DELAY`](#copy_retry_max_delay).

### COPY_RETRY_MAX_DELAY

```rust
pub const COPY_RETRY_MAX_DELAY: Duration = _;
```

The longest delay between two background retries of one copy.

### HOLDING_REMOVED

```rust
pub const HOLDING_REMOVED: &str = "holding_removed";
```

Event type published when a holding is removed.

### MAX_COPY_ATTEMPTS

```rust
pub const MAX_COPY_ATTEMPTS: u32 = 8;
```

Failed attempts after which a pending copy turns [`CopyState::Failed`](#copystate)
and is no longer retried in the background.

### MAX_QUEUE_REASONS

```rust
pub const MAX_QUEUE_REASONS: usize = 6;
```

Most taste reasons one stored row keeps, so a row stays small whatever
the ranking explains.

### OBJECT_TOMBSTONE_DAYS

```rust
pub const OBJECT_TOMBSTONE_DAYS: i64 = 30;
```

Days a removed object's tombstone is kept for readers that page changes.

### VECTOR_KIND_TEXT

```rust
pub const VECTOR_KIND_TEXT: &str = "text";
```

`embeddings.target_kind` of a plain text's vector; the target id is the
SHA-256 hex of the text.

### VECTOR_KIND_WORK

```rust
pub const VECTOR_KIND_WORK: &str = "work";
```

`embeddings.target_kind` of a work's vector; the target id is `works.id`.

### MAX_NOTIFY_PAGE

```rust
pub const MAX_NOTIFY_PAGE: u64 = 200;
```

Most rows one page of the delivery log or the inbox returns.

### CLAIM_REDEEM_NONCE_HEX_LEN

```rust
pub const CLAIM_REDEEM_NONCE_HEX_LEN: usize = 64;
```

Hex length of a browser-generated claim-redeem nonce (32 bytes).
