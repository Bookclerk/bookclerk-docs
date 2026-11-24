# bookclerk_node_proto

Wire protocol between a Bookclerk branch node and its library.

A host connects to `GET /api/node/v1/connect` with the WebSocket
subprotocol [`SUBPROTOCOL`](#subprotocol) and proves itself with the Ed25519 key it
enrolled (H-4 design, section 3):

1. host → owner: [`Body::Hello`](#body) `{node_id, incarnation,
   software_version, resume}`
2. owner → host: [`Body::Challenge`](#body) `{nonce, library_id}`
3. host → owner: [`Body::Auth`](#body) `{sig}`, an Ed25519 signature over
   [`auth_transcript`](#auth_transcript)
4. owner → host: [`Body::Welcome`](#body), or a close with
   [`CLOSE_AUTH_FAILED`](#close_auth_failed) / [`CLOSE_REVOKED`](#close_revoked).

Frames are JSON text of at most [`MAX_FRAME_BYTES`](#max_frame_bytes); binary frames are a
protocol error because bulk data travels over HTTP.

After the handshake, a hello that listed held fences is answered with
[`Body::Resumed`](#body). Work is pulled: the host sends [`Body::Claim`](#body) and
gets [`Body::Work`](#body) (the job, its fence, and its secrets sealed to the
node's box key with [`seal_secrets`](#seal_secrets)) or [`Body::NoWork`](#body); it then
sends [`Body::Renew`](#body), [`Body::Complete`](#body) or [`Body::Fail`](#body) under that
fence, and may write a refreshed store token back with
[`Body::SecretUpdate`](#body). The owner forwards the host's `node.<id>` queue
as [`Body::Wake`](#body) frames, each answered with [`Body::Ack`](#body).

A claim may carry a [`ClaimFilter`](#claimfilter) (the kinds the host runs, whose work
it takes ([`ClaimScope`](#claimscope)), and the
plugins its operator consented to); a job handed back unrun goes with
[`Body::Release`](#body). To act on wakes the host reads
[`Body::DeploymentsList`](#body) and [`Body::InstanceConfigGet`](#body) (`SECRETS`
sealed with [`seal_instance_secrets`](#seal_instance_secrets)), reports
[`Body::Observation`](#body)s, and fetches stored archives over HTTP at
[`ARTIFACTS_PATH`](#artifacts_path) with a request signed by [`sign_artifact_request`](#sign_artifact_request).

A scan run on a host returns its rows in fenced [`Body::ScanResults`](#body)
batches; the owner validates and writes them. Each account's pass then
reports the ids a delta pass found gone ([`Body::ScanRemoved`](#body)) and how
it ended ([`Body::ScanPassEnd`](#body)), so the owner keeps the account's sync
token ([`WorkJob::scan_tokens`](#workjob) hands it back) and prunes after a
complete full pass. An acquire asks
[`Body::AcquirePlan`](#body) for its titles, then for each title asks
[`Body::UploadGrant`](#body) and streams the files over HTTP at
[`UPLOADS_PATH`](#uploads_path) with the token sealed in [`Body::UploadGranted`](#body) (a file
larger than [`UPLOAD_PART_BYTES`](#upload_part_bytes) as consecutive [`UploadPart`](#uploadpart)s); the
owner streams them on to its output destinations without staging them,
and the job completes when the last title's upload commits.

A branch shares the titles of its own library with the library it
joined ([`Body::CatalogBegin`](#body), [`Body::CatalogSync`](#body),
[`Body::CatalogUpload`](#body); see [`CatalogTitle`](#catalogtitle)): titles carry no account
or user field, and files reuse the upload endpoint.

A library asks its branches whether they can supply works it knows
([`LookupWork`](#lookupwork), by strong identifiers, never titles): it stores the
questions and wakes the branch, which pulls them with
[`Body::CatalogQuestions`](#body) and answers with [`Body::CatalogFound`](#body).

A member's request for a book a branch listed is a job on the library
pinned to that branch ([`BranchRequest`](#branchrequest)): the branch claims it with
[`Body::RequestsClaim`](#body) and reports with [`Body::RequestFinish`](#body); the
book itself travels up through the catalog sync.

A window (a branch whose link takes no work) reads the library's books
with [`Body::WindowBooks`](#body), page by page ([`Body::WindowPage`](#body),
[`WindowWork`](#windowwork), each edition a [`WindowEdition`](#windowedition) marked when it is the
branch owner's), and merges them by identifier with the other libraries
it joined.

A host that leaves sends [`Body::NodeLeaving`](#body); the owner revokes the
node at once. Joining with a sign-in instead of a link is the OAuth
code + PKCE flow of the built-in client [`BRANCH_CLIENT_ID`](#branch_client_id) ([`OwnerInfo`](#ownerinfo),
[`OAuthEnrollRequest`](#oauthenrollrequest)). An Owner of the library may instead let the
install join as a shared branch (a shared node, [`ConnectionType::SharedBranch`](#connectiontype)): its
redeem answer carries a [`SharedBranchBootstrap`](#sharedbranchbootstrap) sealed to the node's box key.
A branch's redeem answer may carry the signing-in user's own store
accounts as an [`AccountBundle`](#accountbundle) sealed the same way, when the user
chose to bring them.

An install can also sign in to a library as a person rather than join
it as a node: a clerk ([`CLERK_CLIENT_ID`](#clerk_client_id), [`ClerkTokenAnswer`](#clerktokenanswer)),
whose session token travels as `Authorization: Bookclerk-Clerk`
([`parse_clerk_authorization`](#parse_clerk_authorization)). With it the clerk holds the clerk
socket at [`CLERK_CONNECT_PATH`](#clerk_connect_path) ([`ClerkFrame`](#clerkframe), [`LibraryFrame`](#libraryframe)):
changed topics and pages of the library's books, never work.

This crate is pure: no I/O, no clock, no randomness. The daemon (owner)
and the host client share it so both sides sign and check the same
bytes.

| Item | Kind | Summary |
| --- | --- | --- |
| [`AccountBundle`](#accountbundle) | struct | The store accounts a library hands a joining branch, for one person. `Debug` prints no secret. |
| [`CarriedAccount`](#carriedaccount) | struct | One store account in an `AccountBundle`. |
| [`CarriedSecret`](#carriedsecret) | struct | One secret of a `CarriedAccount`: an `encrypted_secrets` row of the account, by kind and name. |
| [`CatalogFile`](#catalogfile) | struct | One file of a `CatalogTitle`. |
| [`CatalogMetadata`](#catalogmetadata) | struct | Bibliographic metadata of a `CatalogTitle`. |
| [`CatalogRef`](#catalogref) | struct | A title named by store, product id and marketplace: the identity of a `CatalogTitle` on both ends of a catalog sync. |
| [`CatalogTitle`](#catalogtitle) | struct | One title, as a branch shares it. |
| [`ClerkAuthorizeParams`](#clerkauthorizeparams) | struct | The authorization request a clerk consent is for (the query `/oidc/authorize` forwards to the library's `/clerk/approve` page). |
| [`ClerkTokenAnswer`](#clerktokenanswer) | struct | `POST /api/clerk/v1/token` answer. |
| [`ClerkTokenRequest`](#clerktokenrequest) | struct | `POST /api/clerk/v1/token` body: exactly these three strings, each once, and nothing else. |
| [`ClerkUser`](#clerkuser) | struct | The person a clerk session signs in as. |
| [`StoredClerkCode`](#storedclerkcode) | struct | What the library stored with an approved code, as the token exchange checks it. |
| [`ClerkCopy`](#clerkcopy) | struct | What a `copy.granted` frame carries. |
| [`CopyToken`](#copytoken) | struct | A copy token as a clerk socket frame carries it: the bearer of the download endpoint (`crate::COPY_AUTH_SCHEME`). Its `Debug` prints no token, and the text is zeroized when dropped. |
| [`CopyFile`](#copyfile) | struct | One file of a copy (`crate::Body::CopyGranted`). |
| [`CopyGrantedFrame`](#copygrantedframe) | struct | The fields of a `crate::Body::CopyGranted`, borrowed for `copy_granted_problem`. |
| [`AcquireTarget`](#acquiretarget) | struct | One title an acquire job should fetch on a host (`Body::AcquireTargets`). |
| [`CallNumber`](#callnumber) | struct | A title's short call number as the owner names it: 1 to `MAX_CALL_NUMBER_BYTES` ASCII letters and digits, so it is safe in any storage key segment. Deserializing anything else fails, which refuses the whole frame. |
| [`ClaimFilter`](#claimfilter) | struct | What a host is willing to claim (`claim.filter`). |
| [`FileCopy`](#filecopy) | struct | Where one uploaded file is written: a destination and its key there. |
| [`Frame`](#frame) | struct | One frame: the envelope and its body. |
| [`HeldFence`](#heldfence) | struct | A lease the host still holds when it reconnects (`hello.resume`). |
| [`JobStatusEntry`](#jobstatusentry) | struct | One job's state in `Body::JobStatuses`. |
| [`PlanDestination`](#plandestination) | struct | One output destination of the shared node that answered an acquire plan (`Body::AcquireTargets`). |
| [`RemoteArtifact`](#remoteartifact) | struct | One downloadable artifact of a `RemoteVersion`. |
| [`RemoteDeployment`](#remotedeployment) | struct | One deployment that targets a host (`deployments.items`). |
| [`RemoteVersion`](#remoteversion) | struct | The active version of a plugin a host is asked to run. |
| [`ScanImport`](#scanimport) | struct | The owner's library import settings a host scan runs with. |
| [`ScanToken`](#scantoken) | struct | The sync token an owner holds for one account a host scan covers. |
| [`TargetTitle`](#targettitle) | struct | A title an `AcquireTarget` names, as the owner's library holds it: the store listing's identity and the metadata a host names the title's files from and seeds its scratch library with. The store and account are the target's own. |
| [`UploadAnswer`](#uploadanswer) | struct | Answer of the upload endpoint (every status carries one). |
| [`UploadCommitRequest`](#uploadcommitrequest) | struct | Body of `POST <UPLOADS_PATH><upload_id>/commit`. |
| [`UploadFile`](#uploadfile) | struct | One file of an upload (`Body::UploadGrant`). Its content crosses the network once and is written to every copy, each destination under the key that destination's own naming options give it. |
| [`UploadProbe`](#uploadprobe) | struct | What a node's probe found in an audio file it uploads (`UploadFile::probe`). The media type travels as `UploadFile::content_type`. |
| [`WorkJob`](#workjob) | struct | A job handed to a host. |
| [`OAuthEnrollAnswer`](#oauthenrollanswer) | struct | `POST /api/node/v1/enroll/oauth` answer. |
| [`OAuthEnrollRequest`](#oauthenrollrequest) | struct | `POST /api/node/v1/enroll/oauth` body. Keys are base64url without padding. |
| [`OwnerInfo`](#ownerinfo) | struct | `GET /api/node/v1/info` answer. |
| [`LookupAnswer`](#lookupanswer) | struct | A branch's answer to one `LookupWork`. |
| [`LookupEdition`](#lookupedition) | struct | One edition a branch can supply. |
| [`LookupIdentifier`](#lookupidentifier) | struct | One identifier of the work a question is about. |
| [`LookupWork`](#lookupwork) | struct | One question: "can you supply this work?". |
| [`BranchOrigin`](#branchorigin) | struct | A branch's public origin: `http` or `https`, a host, an optional port and nothing else (no user, path, query or fragment), at most `MAX_BRANCH_ORIGIN_BYTES` bytes, in the form `Url` serializes it (lowercase scheme and host, default port dropped). |
| [`BranchRequest`](#branchrequest) | struct | One request a branch claimed: "send this library that book". |
| [`SecretEntry`](#secretentry) | struct | One secret in a bundle. |
| [`SecretRef`](#secretref) | struct | Names one `encrypted_secrets` row of a store account. It carries no secret material. |
| [`SecretValue`](#secretvalue) | struct | Secret bytes: zeroized on drop, redacted in `Debug`, and base64url on the wire. |
| [`SharedBranchBootstrap`](#sharedbranchbootstrap) | struct | What a shared branch receives at redeem: the `[database]` bootstrap and the DEK, for the node id the library recorded (`kind = shared`). `Debug` prints no secret. |
| [`SharedBranchCredential`](#sharedbranchcredential) | struct | The database plugin's own credential, for a plugin whose connection settings don't carry it (D1's API token; a Postgres URL carries its own). Whatever the plugin, the shared branch keeps it sealed with the DEK in its master key store (file, OS keychain or `systemd-creds`), tagged with the plugin it belongs to. `Debug` prints no secret. |
| [`ArtifactRequest`](#artifactrequest) | struct | What a signed artifact request covers. |
| [`AuthContext`](#authcontext) | struct | What a handshake signature covers. |
| [`UploadPart`](#uploadpart) | struct | A part of a declared file: at least one byte, inside the file. |
| [`WindowCopy`](#windowcopy) | struct | The copy of an edition's primary audio file a library holds, as its probe of the file found it: enough for a reader to decide whether to take the library's copy or fetch its own, without asking again. |
| [`WindowCursor`](#windowcursor) | struct | Where a page of the person's books starts: an opaque key the library handed out as the previous page's `next`, or empty for the first page. |
| [`WindowEdition`](#windowedition) | struct | One edition of a `WindowWork`: the edition as the library holds it, and whether it is the person's own. |
| [`WindowWork`](#windowwork) | struct | One work the library holds for the window's person. |
| [`ClerkFrame`](#clerkframe) | enum | A frame from a clerk to a library. |
| [`ClerkSocketError`](#clerksocketerror) | enum | Why the library did not answer a `books` or `copy.grant` frame. |
| [`ClerkTopic`](#clerktopic) | enum | Something a clerk hears about on the socket. Each names the routes to read again; the socket itself carries no row data. |
| [`LibraryFrame`](#libraryframe) | enum | A frame from a library to a clerk. |
| [`CopyAnswer`](#copyanswer) | enum | What the head of a download's answer says follows it (`parse_copy_answer`). |
| [`CopyRange`](#copyrange) | enum | What a download's `Range` header asks for (`parse_copy_range`). |
| [`CopyRole`](#copyrole) | enum | What a file of a copy is, as the library recorded it when it wrote the file. |
| [`Body`](#body) | enum | What a frame says, tagged by `type`. |
| [`ClaimScope`](#claimscope) | enum | Whose work a host takes from a library (`claim.filter.scope`), as the branch operator set it for that link. |
| [`FrameError`](#frameerror) | enum | Why a frame was not accepted. |
| [`KeeperOutcome`](#keeperoutcome) | enum | What a library did with a keeper's `Body::AccountsUpdate`. |
| [`KeyError`](#keyerror) | enum | Why a public key was refused. |
| [`Availability`](#availability) | enum | How a branch can supply an edition. |
| [`LookupFormat`](#lookupformat) | enum | A file format a branch has for an edition. |
| [`LookupScheme`](#lookupscheme) | enum | Which kind of identifier a `LookupIdentifier` is. Only identifiers that name one edition or one work are asked about; anything else does not parse. |
| [`RequestOutcome`](#requestoutcome) | enum | What a branch did with a claimed request. |
| [`SealError`](#sealerror) | enum | Why a sealed box could not be made or opened. |
| [`ConnectionType`](#connectiontype) | enum | How a joining install connects to a library. |
| [`SharedBranchDatabase`](#sharedbranchdatabase) | enum | The library's database, as a shared branch needs it to connect. |
| [`AuthError`](#autherror) | enum | Why a handshake signature was refused. |
| [`TunnelExposure`](#tunnelexposure) | enum | What public traffic may reach. |
| [`TunnelRefusal`](#tunnelrefusal) | enum | Why a tunnel request was refused. |
| [`UploadRange`](#uploadrange) | enum | What a `Content-Range` on an upload `PUT` asks for. |
| [`account_bundle_aad`](#account_bundle_aad) | function | Associated data of the account bundle for `node_id` of `library_id`: the domain tag, then each field as a 4-byte big-endian length and its bytes. |
| [`account_offer_aad`](#account_offer_aad) | function | Associated data of an account offer from `node_id` to its library `library_id`: `ACCOUNT_OFFER_AAD_DOMAIN`, then the two ids as in `account_bundle_aad`. |
| [`account_reconcile_aad`](#account_reconcile_aad) | function | Associated data of a library's sign-in answering a keeper's first push from `node_id`: `ACCOUNT_RECONCILE_AAD_DOMAIN`, then the two ids as in `account_bundle_aad`. |
| [`check_account_bundle`](#check_account_bundle) | function | Checks `bundle` against every rule `parse_account_bundle` applies. |
| [`open_account_bundle`](#open_account_bundle) | function | Opens a bundle made by `seal_account_bundle` with the joining node's box secret, for the library and node ids the redeem answered. |
| [`open_account_offer`](#open_account_offer) | function | Opens an offer made by `seal_account_offer` with the library's owner box secret, for the library id and the node id of the authenticated connection it arrived on. |
| [`open_account_reconcile`](#open_account_reconcile) | function | Opens a library's sign-in made by `seal_account_reconcile` with the branch's box secret, for the library it joined and its own node id. |
| [`parse_account_bundle`](#parse_account_bundle) | function | Parses the opened bytes of an account bundle for `node_id` of `library_id` and checks it (`check_account_bundle`). Sans-IO: the branch runs it on what `open` returns, and the fuzz target runs it on arbitrary bytes. |
| [`seal_account_bundle`](#seal_account_bundle) | function | Seals `bundle` to the joining node's box key `recipient`, as base64url text for the redeem answer. |
| [`seal_account_offer`](#seal_account_offer) | function | Seals a branch's `bundle` to its library's owner box key `recipient` (`crate::Body::Welcome`), for `crate::Body::AccountsAdd`. The box is bound with `account_offer_aad`, so a joining bundle never opens as an offer, nor the other way round. |
| [`seal_account_reconcile`](#seal_account_reconcile) | function | Seals a library's own sign-in of one account (`bundle`) to the keeping branch's box key `recipient`, for the `sealed` of an `crate::Body::AccountsUpdated` with `crate::KeeperOutcome::Reconcile`. Bound with `account_reconcile_aad`. |
| [`is_ancestry`](#is_ancestry) | function | True when every id in `ancestry` is a plain library id and there are at most `MAX_ANCESTRY`. |
| [`is_library_id`](#is_library_id) | function | True for a plain library id in an ancestry. |
| [`is_marketplace`](#is_marketplace) | function | True for a marketplace: empty, or up to `MAX_MARKETPLACE_BYTES` lowercase ASCII letters and digits. |
| [`is_product_id`](#is_product_id) | function | True for a product id: 1 to 256 bytes, no whitespace at the ends and no control characters. |
| [`is_store_id`](#is_store_id) | function | True for a store id: a short lowercase token. |
| [`clerk_exchange_matches`](#clerk_exchange_matches) | function | True when `request` redeems `stored`: the code belongs to `CLERK_CLIENT_ID`, the redirect URI is the same, the method is S256 and the trimmed verifier hashes to the challenge (compared in constant time). |
| [`clerk_request_problem`](#clerk_request_problem) | function | Why a clerk authorization request is malformed, or `None` when the library may show its consent page. |
| [`is_clerk_name`](#is_clerk_name) | function | True for a clerk name: 1 to `MAX_CLERK_NAME_BYTES` bytes, no control characters. |
| [`is_clerk_token`](#is_clerk_token) | function | True for a session token: exactly `CLERK_TOKEN_LEN` base64url characters. |
| [`is_s256_challenge`](#is_s256_challenge) | function | True for an S256 code challenge: exactly 43 base64url characters. |
| [`parse_clerk_authorization`](#parse_clerk_authorization) | function | The session token in an `Authorization` header value, when the value is exactly `Bookclerk-Clerk <token>`. |
| [`parse_clerk_query`](#parse_clerk_query) | function | Parses an authorize query string (the raw query `/oidc/authorize` received), sans-IO so the fuzz target drives the same code. |
| [`parse_clerk_token_answer`](#parse_clerk_token_answer) | function | Reads the library's `POST /api/clerk/v1/token` answer on the clerk. |
| [`parse_clerk_token_request`](#parse_clerk_token_request) | function | Reads a `POST /api/clerk/v1/token` body. |
| [`is_topic_name`](#is_topic_name) | function | True when `name` could be a topic name: 1 to `MAX_CLERK_TOPIC_BYTES` bytes of lowercase ASCII letters, digits, `_` and `.`. |
| [`parse_clerk_frame`](#parse_clerk_frame) | function | Reads a frame a clerk sent: at most `MAX_CLERK_FRAME_BYTES`, one JSON object with `v` (1), a known `type`, and exactly that type's fields, each once (a field written as `null` counts as absent). |
| [`parse_library_frame`](#parse_library_frame) | function | Reads a frame a library sent: at most `crate::MAX_FRAME_BYTES`, one JSON object with `v` (1), a known `type`, and exactly that type's fields, each once. |
| [`copy_answer_head_end`](#copy_answer_head_end) | function | Finds where the head of a download's answer ends in the bytes received so far, however the library's writes were cut into reads. |
| [`copy_grant_problem`](#copy_grant_problem) | function | Why a `crate::Body::CopyGrant` is malformed, or `None`: a store id, a marketplace (empty when the store has none) and a product id. |
| [`copy_granted_problem`](#copy_granted_problem) | function | Why a `crate::Body::CopyGranted` is malformed, or `None`: a copy id (`is_copy_id`), a non-empty sealed token of base64url text, a life of 1 to `MAX_COPY_TOKEN_SECS` seconds, a well-formed copy, and 1 to `MAX_COPY_FILES` well-formed files with distinct names, exactly one of them the primary audio (`CopyRole::Audio`). |
| [`is_copy_id`](#is_copy_id) | function | True for a copy id: a lowercase hyphenated UUID. |
| [`parse_copy_answer`](#parse_copy_answer) | function | Reads the head of the answer to a download of a file of `size` bytes that asked for the bytes from `asked_from` on (0 for the whole file). |
| [`parse_copy_range`](#parse_copy_range) | function | Reads the `Range` header of a download of a file of `size` bytes. |
| [`deployments_page_problem`](#deployments_page_problem) | function | Why a branch would refuse one `deployments` page answering `deployments.list {after}`, or `None` when it takes it: every entry's deployment id sorts byte-wise after `after` and after the entry before it (so no deployment is listed twice), and `next`, when present, sorts after `after` and is not before the last entry. A page that breaks this would let a library keep a branch paging forever or list a deployment twice, so the branch drops the whole listing and retries. |
| [`is_destination_id`](#is_destination_id) | function | True when `id` is a destination id: a short lowercase token. |
| [`is_storage_key`](#is_storage_key) | function | True when `key` is a relative storage key: no empty, `.` or `..` segment, no backslash or control character, not absolute, at most `MAX_UPLOAD_KEY_BYTES`, and never a Windows path that leaves the destination's root (`colon_is_safe`). |
| [`scan_pass_end_problem`](#scan_pass_end_problem) | function | Why a `Body::ScanPassEnd` the owner received is malformed, or `None` when it is one a host could send: a non-blank account id and a sync token of at most `MAX_SCAN_SYNC_TOKEN_BYTES` with no control characters. |
| [`scan_removed_problem`](#scan_removed_problem) | function | Why a `Body::ScanRemoved` the owner received is malformed, or `None` when it is one a host could send: a non-blank account id, at most `MAX_SCAN_REMOVED_IDS` product ids, each non-blank, at most `MAX_SCAN_PRODUCT_ID_BYTES` and free of control characters. |
| [`target_destinations_problem`](#target_destinations_problem) | function | Why `target`'s destination list (`AcquireTarget::destinations`) does not fit the plan it came with, or `None`: every id it names is one of `plan`'s destinations, named once, in the plan's order. |
| [`is_branch_name`](#is_branch_name) | function | True for a branch name: 1 to `MAX_BRANCH_NAME_BYTES` bytes, no control characters. |
| [`is_branch_redirect_uri`](#is_branch_redirect_uri) | function | True for a redirect URI `BRANCH_CLIENT_ID` may use: an absolute `https` URL, or an `http` URL whose host is a loopback IP literal or `localhost` (RFC 8252 section 7.3), with no user info and no fragment. |
| [`is_library_name`](#is_library_name) | function | True for a library name: 1 to `MAX_LIBRARY_NAME_BYTES` bytes, no control characters. |
| [`pkce_challenge`](#pkce_challenge) | function | The RFC 7636 S256 challenge of `verifier`: base64url (no padding) of its SHA-256. |
| [`box_public_key`](#box_public_key) | function | The X25519 public key of a 32-byte secret (clamped per RFC 7748). |
| [`sign_public_key`](#sign_public_key) | function | The Ed25519 public key of a 32-byte signing seed. |
| [`validate_box_public_key`](#validate_box_public_key) | function | Validates an X25519 public box key (a Montgomery u-coordinate). |
| [`validate_sign_public_key`](#validate_sign_public_key) | function | Validates an Ed25519 public signing key. |
| [`found_problem`](#found_problem) | function | Why a `crate::Body::CatalogFound` is malformed, or `None`: at most `MAX_LOOKUP_WORKS` works named in all, each once, every answer well-formed and every missing work a correlation key. |
| [`is_work_ref`](#is_work_ref) | function | True for a correlation key: 1 to `MAX_WORK_REF_BYTES` ASCII letters, digits and `-`. |
| [`questions_problem`](#questions_problem) | function | Why a list of questions is malformed, or `None`: at most `MAX_LOOKUP_WORKS` well-formed questions, each about a different work. |
| [`claimed_problem`](#claimed_problem) | function | Why a list of claimed requests is malformed, or `None`: at most `MAX_CLAIMED_REQUESTS` well-formed requests, each for a different job. |
| [`finish_problem`](#finish_problem) | function | The problem with a `requests.finish {job, generation}` a library received, or `None` when both are ones it could have handed out. |
| [`is_request_job`](#is_request_job) | function | True for a request's job id: 1 to `MAX_REQUEST_JOB_BYTES` ASCII letters, digits, `-` and `_`. |
| [`copy_token_aad`](#copy_token_aad) | function | Associated data of the token for copy `copy_id`, granted to `node_id`. |
| [`instance_secrets_aad`](#instance_secrets_aad) | function | Associated data of `instance_id`'s `SECRETS` handed to `node_id` at document `revision`. |
| [`open`](#open) | function | Opens a box sealed to the public key of `recipient_secret`. |
| [`open_copy_token`](#open_copy_token) | function | Opens a token made by `seal_copy_token` with the node's box secret. |
| [`open_instance_secrets`](#open_instance_secrets) | function | Opens a payload made by `seal_instance_secrets` with the node's box secret. |
| [`open_secret_update`](#open_secret_update) | function | Opens a write-back made by `seal_secret_update` with the owner's per-connection box secret. |
| [`open_secrets`](#open_secrets) | function | Opens a bundle made by `seal_secrets` with the node's box secret. |
| [`open_upload_token`](#open_upload_token) | function | Opens a token made by `seal_upload_token` with the node's box secret. |
| [`seal`](#seal) | function | Seals `plaintext` to `recipient` (an X25519 public key) with the ephemeral secret `ephemeral_secret`, binding `aad`. |
| [`seal_copy_token`](#seal_copy_token) | function | Seals a copy token for `node_id`'s box key `recipient`, bound to the node and the copy, as base64url text for a `copy.granted` frame. The token is a bearer credential for the download endpoint, so it never travels as frame text. |
| [`seal_instance_secrets`](#seal_instance_secrets) | function | Seals an instance's `SECRETS` payload (JSON bytes) for `node_id`'s box key `recipient`, bound to the instance and document revision, as base64url text for an `instance_config` frame. |
| [`seal_secret_update`](#seal_secret_update) | function | Seals a refreshed token to the owner's per-connection box key `owner_box_key` (from `welcome`), bound to the node, job, fence and ref, as base64url text for a `secret.update` frame. |
| [`seal_secrets`](#seal_secrets) | function | Seals `entries` for `node_id`'s box key `recipient`, bound to `job_id` and `fence`, as base64url text for a `work` frame. |
| [`seal_upload_token`](#seal_upload_token) | function | Seals an upload token for `node_id`'s box key `recipient`, bound to the job, fence and upload, as base64url text for an `upload.granted` frame. The token is a bearer credential for the upload endpoint, so it never travels as frame text. |
| [`secret_update_aad`](#secret_update_aad) | function | Associated data of a token write-back from `node_id` for `secret_ref`, handed with `job_id` under `fence`. |
| [`secrets_aad`](#secrets_aad) | function | Associated data of the secret bundle handed to `node_id` for `job_id` under lease generation `fence`: the domain tag, then each field as a 4-byte big-endian length and its bytes. |
| [`upload_token_aad`](#upload_token_aad) | function | Associated data of the token for upload `upload_id`, granted to `node_id` for `job_id` under lease generation `fence`. |
| [`open_shared_branch_bootstrap`](#open_shared_branch_bootstrap) | function | Opens a bootstrap made by `seal_shared_branch_bootstrap` with the joining node's box secret, for the library and node ids the redeem answered. |
| [`parse_shared_branch_bootstrap`](#parse_shared_branch_bootstrap) | function | Reads an opened bootstrap for `library_id` and `node_id`. |
| [`seal_shared_branch_bootstrap`](#seal_shared_branch_bootstrap) | function | Seals `bootstrap` to the joining node's box key `recipient`, as base64url text for the redeem answer. |
| [`shared_branch_bootstrap_aad`](#shared_branch_bootstrap_aad) | function | Associated data of a shared branch bootstrap for `node_id` of `library_id`: the domain tag, then each field as a 4-byte big-endian length and its bytes. |
| [`artifact_transcript`](#artifact_transcript) | function | The bytes a host signs for an artifact request: the domain tag, then each field as a 4-byte big-endian length and its bytes. |
| [`auth_transcript`](#auth_transcript) | function | The bytes a host signs: the domain tag, then each field as a 4-byte big-endian length and its bytes. |
| [`canonical_origin`](#canonical_origin) | function | The ASCII origin (`scheme://host[:port]`) of an http(s) URL, with the default port and any path dropped and the host lowercased. |
| [`decode_nonce`](#decode_nonce) | function | Decodes a challenge nonce. |
| [`encode_nonce`](#encode_nonce) | function | Encodes a nonce as base64url without padding. |
| [`parse_artifact_authorization`](#parse_artifact_authorization) | function | Splits an artifact `Authorization` header into node id, issue time and signature. |
| [`sign_artifact_request`](#sign_artifact_request) | function | The `Authorization` header value of a signed artifact request. |
| [`sign_auth`](#sign_auth) | function | Signs a handshake; the result is base64url without padding. |
| [`verify_artifact_request`](#verify_artifact_request) | function | Verifies a signed artifact request with the node's enrolled key and checks its time against `now` (unix seconds). |
| [`verify_auth`](#verify_auth) | function | Verifies a handshake signature with the node's enrolled public key. Uses strict verification (no malleable or small-order signatures). |
| [`normalize_tunnel_path`](#normalize_tunnel_path) | function | The path the router matches for `raw`: the router trims trailing slashes and keeps exactly one leading slash, so `//api/node/` routes as `/api/node`. |
| [`tunnel_refusal`](#tunnel_refusal) | function | Whether a tunnel request for `path` is refused. |
| [`parse_upload_range`](#parse_upload_range) | function | Reads a `Content-Range` header value on an upload `PUT` of a declared file of `size` bytes. |
| [`upload_parts`](#upload_parts) | function | The parts a host sends a file of `size` bytes in, each at most `part_bytes` (at least 1) long, in order; empty when the file fits in one request (`size <= part_bytes`), which is then sent whole. |
| [`window_page_problem`](#window_page_problem) | function | Why a `crate::Body::WindowPage` is malformed, or `None`: at most `MAX_WINDOW_WORKS` well-formed works with distinct grouping keys, and a next cursor, when there is one, past `after` (the cursor the window asked with), so paging always moves forward. |
| [`ACCOUNT_BUNDLE_AAD_DOMAIN`](#account_bundle_aad_domain) | constant | Domain tag that starts the associated data of an account bundle. |
| [`ACCOUNT_BUNDLE_VERSION`](#account_bundle_version) | constant | Version of an `AccountBundle`. |
| [`ACCOUNT_OFFER_AAD_DOMAIN`](#account_offer_aad_domain) | constant | Domain tag that starts the associated data of an account offer: a branch's `AccountBundle` sent up to its library (`crate::Body::AccountsAdd`). A distinct tag keeps a bundle sealed in one direction from opening as the other. |
| [`ACCOUNT_RECONCILE_AAD_DOMAIN`](#account_reconcile_aad_domain) | constant | Domain of the associated data a library's own sign-in is sealed with when it answers a keeper's first push with `crate::KeeperOutcome::Reconcile` (`seal_account_reconcile`), so it never opens as a joining bundle or an offer. |
| [`MAX_ACCOUNT_BUNDLE_BYTES`](#max_account_bundle_bytes) | constant | Largest opened bundle accepted, in bytes. A bundle at every limit above would be far larger; this keeps the redeem answer small. |
| [`MAX_ACCOUNT_SECRETS`](#max_account_secrets) | constant | Most secrets one carried account holds. |
| [`MAX_BUNDLE_ACCOUNTS`](#max_bundle_accounts) | constant | Most accounts one bundle carries. |
| [`MAX_CARRIED_ACCOUNT_ID_BYTES`](#max_carried_account_id_bytes) | constant | Longest store account id carried, in bytes. |
| [`MAX_CARRIED_FIELD_BYTES`](#max_carried_field_bytes) | constant | Longest source alias, marketplace, secret kind or label carried, in bytes. |
| [`MAX_CARRIED_PLUGIN_KEY_BYTES`](#max_carried_plugin_key_bytes) | constant | Longest plugin key carried, in bytes. |
| [`MAX_CARRIED_SECRET_BYTES`](#max_carried_secret_bytes) | constant | Largest secret value carried, in bytes. |
| [`CATALOG_ITEM`](#catalog_item) | constant | Item kind of the catalog fence (`lost.item`), and the job id a catalog upload's token is sealed for. |
| [`MAX_ANCESTRY`](#max_ancestry) | constant | Most library ids an ancestry may list. |
| [`MAX_CATALOG_BATCH_TITLES`](#max_catalog_batch_titles) | constant | Most titles one `crate::Body::CatalogSync` batch may carry. |
| [`MAX_CATALOG_CURSOR_BYTES`](#max_catalog_cursor_bytes) | constant | Longest cursor. |
| [`MAX_CATALOG_FILE_VERSION_BYTES`](#max_catalog_file_version_bytes) | constant | Longest `CatalogFile::version` a title may carry, in bytes. |
| [`MAX_CATALOG_TITLE_BYTES`](#max_catalog_title_bytes) | constant | Largest serialized title in a batch. |
| [`MAX_CATALOG_WANTED`](#max_catalog_wanted) | constant | Most titles one answer may say are wanted. |
| [`MAX_MARKETPLACE_BYTES`](#max_marketplace_bytes) | constant | Longest marketplace. |
| [`CLERK_AUTH_SCHEME`](#clerk_auth_scheme) | constant | The `Authorization` scheme a clerk sends its session token under. |
| [`CLERK_CALLBACK_PATH`](#clerk_callback_path) | constant | The clerk's own web app page a clerk sign-in's redirect comes back to (`<clerk origin>/clerk/callback?code=…&state=…`). |
| [`CLERK_CLIENT_ID`](#clerk_client_id) | constant | The built-in public OAuth client a clerk signs in through. It is never stored in the library's clients table. |
| [`CLERK_CODE_TTL_SECS`](#clerk_code_ttl_secs) | constant | Seconds an authorization code for `CLERK_CLIENT_ID` lives. |
| [`CLERK_IDLE_DAYS`](#clerk_idle_days) | constant | Days a clerk session stays signed in without being used. Each use moves the expiry forward again. |
| [`CLERK_SCOPE`](#clerk_scope) | constant | The only scope `CLERK_CLIENT_ID` may ask for. |
| [`CLERK_TOKEN_LEN`](#clerk_token_len) | constant | Length of a session token: base64url (no padding) of 32 random bytes. |
| [`CLERK_TOKEN_PATH`](#clerk_token_path) | constant | `POST`: redeem an approved clerk sign-in for a session token. |
| [`MAX_CLERK_ANSWER_BYTES`](#max_clerk_answer_bytes) | constant | Largest `POST /api/clerk/v1/token` answer a clerk reads. |
| [`MAX_CLERK_DISPLAY_NAME_BYTES`](#max_clerk_display_name_bytes) | constant | Longest display name a clerk keeps from a token answer. |
| [`MAX_CLERK_NAME_BYTES`](#max_clerk_name_bytes) | constant | Longest clerk name sent. |
| [`MAX_CLERK_STATE_BYTES`](#max_clerk_state_bytes) | constant | Longest `state` an authorize request may carry. |
| [`CLERK_CONNECT_PATH`](#clerk_connect_path) | constant | Path of the clerk socket. |
| [`CLERK_SOCKET_VERSION`](#clerk_socket_version) | constant | The `v` every frame carries. |
| [`CLERK_SUBPROTOCOL`](#clerk_subprotocol) | constant | The WebSocket subprotocol a clerk must offer. |
| [`MAX_CLERK_FRAME_BYTES`](#max_clerk_frame_bytes) | constant | Largest frame a library accepts from a clerk, in bytes; a `books` frame is under 120. |
| [`MAX_CLERK_TOPICS`](#max_clerk_topics) | constant | Most topic names one `hello` or `changed` frame may carry, known or not. |
| [`MAX_CLERK_TOPIC_BYTES`](#max_clerk_topic_bytes) | constant | Longest topic name, in bytes. |
| [`COPIES_PATH`](#copies_path) | constant | Path prefix of the download endpoint: the files of copy `<copy_id>` are at `COPIES_PATH<copy_id>/files/<n>`. |
| [`COPY_AUTH_SCHEME`](#copy_auth_scheme) | constant | `Authorization` scheme of the download endpoint. |
| [`MAX_COPY_ANSWER_HEAD_BYTES`](#max_copy_answer_head_bytes) | constant | Longest head of a download's answer a reader takes, in bytes: the status line and headers, before the blank line. |
| [`MAX_COPY_FILES`](#max_copy_files) | constant | Most files one `crate::Body::CopyGranted` may list. |
| [`MAX_COPY_FILE_BYTES`](#max_copy_file_bytes) | constant | Largest `CopyFile::size` (1 TiB): above any audiobook file. |
| [`MAX_COPY_NAME_BYTES`](#max_copy_name_bytes) | constant | Longest `CopyFile::name`, in bytes. |
| [`MAX_COPY_TOKEN_SECS`](#max_copy_token_secs) | constant | Longest a copy token may be stated to live, in seconds (one day). |
| [`ITEM_JOB`](#item_job) | constant | Work item kind for jobs. |
| [`MAX_CALL_NUMBER_BYTES`](#max_call_number_bytes) | constant | Longest call number (see `CallNumber`). |
| [`MAX_DESTINATION_ID_BYTES`](#max_destination_id_bytes) | constant | Longest destination id. |
| [`MAX_PROBE_BITRATE_KBPS`](#max_probe_bitrate_kbps) | constant | Highest bitrate an `UploadProbe` may state, in kilobits per second: above any audiobook encoding, so a forged value cannot win by size alone. |
| [`MAX_SCAN_BATCH_ROWS`](#max_scan_batch_rows) | constant | Most rows one `Body::ScanResults` batch may carry. |
| [`MAX_SCAN_PRODUCT_ID_BYTES`](#max_scan_product_id_bytes) | constant | Longest product id a `Body::ScanRemoved` batch may carry. |
| [`MAX_SCAN_REMOVED_IDS`](#max_scan_removed_ids) | constant | Most product ids one `Body::ScanRemoved` batch may carry (a scan page's own cap). |
| [`MAX_SCAN_ROW_BYTES`](#max_scan_row_bytes) | constant | Largest serialized row in a `Body::ScanResults` batch. |
| [`MAX_SCAN_SYNC_TOKEN_BYTES`](#max_scan_sync_token_bytes) | constant | Longest sync token a `Body::ScanPassEnd` or `ScanToken` may carry (a scan page's own cap). |
| [`MAX_SCAN_TOKENS`](#max_scan_tokens) | constant | Most `ScanToken`s one `WorkJob` may carry. |
| [`MAX_STATUS_JOBS`](#max_status_jobs) | constant | Most job ids one `Body::JobStatus` may ask about. |
| [`MAX_UPLOAD_COPIES`](#max_upload_copies) | constant | Most destinations one uploaded file may be written to. |
| [`MAX_UPLOAD_FILES`](#max_upload_files) | constant | Most files one upload may declare. |
| [`MAX_UPLOAD_FILE_BYTES`](#max_upload_file_bytes) | constant | Largest file one upload may declare (8 GiB). |
| [`MAX_UPLOAD_KEY_BYTES`](#max_upload_key_bytes) | constant | Longest storage key an upload may declare. |
| [`AUTHORIZE_PATH`](#authorize_path) | constant | The owner's authorization endpoint (path). |
| [`BRANCH_CLIENT_ID`](#branch_client_id) | constant | The built-in public OAuth client a joining branch signs in through. It is never stored in the owner's clients table. |
| [`BRANCH_CODE_TTL_SECS`](#branch_code_ttl_secs) | constant | Seconds an authorization code for `BRANCH_CLIENT_ID` lives. |
| [`BRANCH_SCOPE`](#branch_scope) | constant | The only scope `BRANCH_CLIENT_ID` may ask for. |
| [`ENROLL_OAUTH_PATH`](#enroll_oauth_path) | constant | `POST`: redeem an approved branch sign-in. |
| [`INFO_PATH`](#info_path) | constant | `GET`: what a library tells a joining install before sign-in. |
| [`JOIN_CALLBACK_PATH`](#join_callback_path) | constant | Path of the joining install's own callback (the SPA route the browser returns to, and the CLI's loopback listener). |
| [`MAX_BRANCH_NAME_BYTES`](#max_branch_name_bytes) | constant | Longest branch name sent. |
| [`MAX_LIBRARY_NAME_BYTES`](#max_library_name_bytes) | constant | Longest library name accepted: a library is named by its public host, and a DNS name runs to 253 bytes. |
| [`PUBLIC_KEY_LEN`](#public_key_len) | constant | Length of either public key. |
| [`MAX_LOOKUP_EDITIONS`](#max_lookup_editions) | constant | Most editions one `LookupAnswer` may list. |
| [`MAX_LOOKUP_IDENTIFIERS`](#max_lookup_identifiers) | constant | Most identifiers one `LookupWork` may carry. |
| [`MAX_LOOKUP_WORKS`](#max_lookup_works) | constant | Most works one `crate::Body::CatalogAsk` or `crate::Body::CatalogFound` may name. |
| [`MAX_WORK_REF_BYTES`](#max_work_ref_bytes) | constant | Longest correlation key (`LookupWork::work`). |
| [`MAX_BRANCH_ORIGIN_BYTES`](#max_branch_origin_bytes) | constant | Longest origin a branch may report, in bytes. |
| [`ITEM_REQUEST`](#item_request) | constant | The work item name of a request in a `crate::Body::Lost` answer. |
| [`MAX_CLAIMED_REQUESTS`](#max_claimed_requests) | constant | Most requests one `crate::Body::RequestsClaimed` may carry. |
| [`MAX_REQUEST_JOB_BYTES`](#max_request_job_bytes) | constant | Longest job id a request may carry. |
| [`COPY_TOKEN_AAD_DOMAIN`](#copy_token_aad_domain) | constant | Domain of the associated data of a sealed copy token. |
| [`INSTANCE_SECRETS_AAD_DOMAIN`](#instance_secrets_aad_domain) | constant | Domain tag that starts the associated data of an instance's `SECRETS`. |
| [`SEALED_BOX_INFO`](#sealed_box_info) | constant | HKDF info string of a sealed box. |
| [`SEALED_BOX_OVERHEAD`](#sealed_box_overhead) | constant | Bytes a sealed box adds to its plaintext (ephemeral key and tag). |
| [`SECRETS_AAD_DOMAIN`](#secrets_aad_domain) | constant | Domain tag that starts the associated data of a secret bundle. |
| [`SECRET_UPDATE_AAD_DOMAIN`](#secret_update_aad_domain) | constant | Domain tag that starts the associated data of a token write-back. |
| [`UPLOAD_TOKEN_AAD_DOMAIN`](#upload_token_aad_domain) | constant | Domain tag that starts the associated data of an upload token. |
| [`DEK_LEN`](#dek_len) | constant | Length of the data-encryption key. |
| [`MAX_BOOTSTRAP_CREDENTIAL_BYTES`](#max_bootstrap_credential_bytes) | constant | The longest database credential (a Postgres URL, a D1 API token) a bootstrap may carry, in bytes. |
| [`MAX_BOOTSTRAP_CREDENTIAL_ID_BYTES`](#max_bootstrap_credential_id_bytes) | constant | The longest provider id for a database credential a bootstrap may carry, in bytes. |
| [`MAX_SHARED_BRANCH_BOOTSTRAP_BYTES`](#max_shared_branch_bootstrap_bytes) | constant | The longest opened bootstrap accepted, in bytes. |
| [`SHARED_BRANCH_BOOTSTRAP_AAD_DOMAIN`](#shared_branch_bootstrap_aad_domain) | constant | Domain tag that starts the associated data of a shared branch bootstrap. |
| [`SHARED_BRANCH_BOOTSTRAP_VERSION`](#shared_branch_bootstrap_version) | constant | Version of `SharedBranchBootstrap`. |
| [`SHARED_BRANCH_GRANTED_SCOPE`](#shared_branch_granted_scope) | constant | The scope a shared branch code is granted (stored with the code at consent). |
| [`SHARED_BRANCH_OFFER_PARAM`](#shared_branch_offer_param) | constant | Authorize-request parameter by which a joining install offers to join as a shared branch (`shared_branch_offer=1`): it is empty, so its own data may be replaced. Without it the consent page offers a branch only. |
| [`ARTIFACT_AUTH_SCHEME`](#artifact_auth_scheme) | constant | `Authorization` scheme of a signed artifact request: `Bookclerk-Node <node_id> <issued_at> <sig>`. |
| [`ARTIFACT_DOMAIN`](#artifact_domain) | constant | Domain tag that starts every signed artifact request. |
| [`ARTIFACT_REQUEST_MAX_SKEW_SECS`](#artifact_request_max_skew_secs) | constant | How far a signed artifact request's time may be from the owner's clock. |
| [`AUTH_DOMAIN`](#auth_domain) | constant | Domain tag that starts every signed transcript. |
| [`NONCE_LEN`](#nonce_len) | constant | Length of a challenge nonce. |
| [`BRANCH_API_PATHS`](#branch_api_paths) | constant | API paths `TunnelExposure::Branches` serves. Each also covers the paths below it. |
| [`LEGACY_CONTROL_PATHS`](#legacy_control_paths) | constant | Legacy control routes outside `/api/` that `TunnelExposure::Branches` refuses. Each also covers the paths below it. |
| [`OPERATOR_ONLY_PATHS`](#operator_only_paths) | constant | Paths no tunnel request reaches: the operator-token sign-in, the loopback tray handoff, operator elevation and the operator's first-Owner bootstrap. Each also covers the paths below it. |
| [`UPLOAD_PART_BYTES`](#upload_part_bytes) | constant | Largest part a host sends (64 MiB): under the request-body caps of the relays an install can sit behind (100 MB on Cloudflare's Free plan). |
| [`MAX_COPY_CONTENT_TYPE_BYTES`](#max_copy_content_type_bytes) | constant | Longest `WindowCopy::content_type`, in bytes. |
| [`MAX_COPY_VERSION_BYTES`](#max_copy_version_bytes) | constant | Longest `WindowCopy::version`, in bytes. |
| [`MAX_WINDOW_WORKS`](#max_window_works) | constant | Most works one `crate::Body::WindowPage` may carry. |
| [`ARTIFACTS_PATH`](#artifacts_path) | constant | Path prefix of the owner's artifact endpoint: `GET /api/node/v1/artifacts/<archive_sha256>` with a signed `Authorization` (`sign_artifact_request`) streams one stored archive to a host that runs it. |
| [`UPLOADS_PATH`](#uploads_path) | constant | Path prefix of the owner's upload endpoint (H-4 design, section 6): `PUT\|HEAD <prefix><upload_id>/files/<n>` streams one declared file and `POST <prefix><upload_id>/commit` commits the title, each with `Authorization: Bookclerk-Upload <token>` (`UPLOAD_AUTH_SCHEME`). |
| [`UPLOAD_AUTH_SCHEME`](#upload_auth_scheme) | constant | `Authorization` scheme of the upload endpoint. |
| [`SUBPROTOCOL`](#subprotocol) | constant | WebSocket subprotocol a host must offer. |
| [`FRAME_VERSION`](#frame_version) | constant | Frame envelope version (`v`). |
| [`MAX_FRAME_BYTES`](#max_frame_bytes) | constant | Largest frame either side accepts (1 MiB). |
| [`PING_INTERVAL_SECS`](#ping_interval_secs) | constant | Seconds between pings from each side. |
| [`IDLE_TIMEOUT_SECS`](#idle_timeout_secs) | constant | Seconds without any inbound frame before a side closes the connection. |
| [`SEND_QUEUE_FRAMES`](#send_queue_frames) | constant | Frames a side queues for sending before it stops forwarding wakes and waits for the peer to read. |
| [`HANDSHAKE_TIMEOUT_SECS`](#handshake_timeout_secs) | constant | Seconds the owner waits for each handshake frame. |
| [`CLOSE_AUTH_FAILED`](#close_auth_failed) | constant | Close code: the handshake failed (unknown node, bad signature, or a malformed or late handshake frame). |
| [`CLOSE_REVOKED`](#close_revoked) | constant | Close code: the node's credential is revoked. |
| [`CLOSE_SUPERSEDED`](#close_superseded) | constant | Close code: a newer authenticated connection for the same node replaced this one. |
| [`CLOSE_PROTOCOL`](#close_protocol) | constant | Close code: a frame broke the protocol after the handshake (binary, oversized, malformed, or the wrong version). |

## Re-exports

- [`ed25519_dalek::SigningKey`](https://docs.rs/ed25519_dalek/latest/ed25519_dalek/signing/struct.SigningKey.html)

## Structs

### AccountBundle

```rust
pub struct AccountBundle {
    pub v: u32,
    pub library_id: String,
    pub node_id: String,
    pub accounts: Vec<CarriedAccount>,
}
```

The store accounts a library hands a joining branch, for one person.
`Debug` prints no secret.

**Fields**

- `v`: [`ACCOUNT_BUNDLE_VERSION`](#account_bundle_version).
- `library_id`: The library id.
- `node_id`: The node id the library minted for the branch.
- `accounts`: The person's accounts, each with its secrets.

Implements: `Clone`, `Debug`, `Deserialize<'de>`, `Eq`, `PartialEq`, `Serialize`

### CarriedAccount

```rust
pub struct CarriedAccount {
    pub plugin_key: String,
    pub account_id: String,
    pub source: String,
    pub marketplace: String,
    pub label: Option<String>,
    pub secrets: Vec<CarriedSecret>,
}
```

One store account in an [`AccountBundle`](#accountbundle).

**Fields**

- `plugin_key`: Provenance-qualified plugin key of the storefront that owns the
  account.
- `account_id`: Store account id, unique within its plugin key.
- `source`: Display alias of the storefront (`libro`, `chirp`, ...).
- `marketplace`: Store marketplace or locale code (`us`, `uk`, ...).
- `label`: The account's display label, when it has one.
- `secrets`: The account's sealed secrets, unsealed for the box.

Implements: `Clone`, `Debug`, `Deserialize<'de>`, `Eq`, `PartialEq`, `Serialize`

### CarriedSecret

```rust
pub struct CarriedSecret {
    pub kind: String,
    pub name: String,
    pub value: SecretValue,
}
```

One secret of a [`CarriedAccount`]: an `encrypted_secrets` row of the
account, by kind and name.

**Fields**

- `kind`: Secret kind (`source_auth`, ...).
- `name`: Logical secret name within the account.
- `value`: The secret bytes.

Implements: `Clone`, `Debug`, `Deserialize<'de>`, `Eq`, `PartialEq`, `Serialize`

### CatalogFile

```rust
pub struct CatalogFile {
    pub key: String,
    pub size: u64,
    pub content_type: Option<String>,
    pub version: Option<String>,
}
```

One file of a [`CatalogTitle`](#catalogtitle).

**Fields**

- `key`: Relative storage key (the same on every destination of the
  receiving library).
- `size`: Size in bytes.
- `content_type`: MIME type.
- `version`: An opaque marker that changes when the file's content changes (the
  branch's storage change signature, entity tag or modification
  time), at most [`MAX_CATALOG_FILE_VERSION_BYTES`](#max_catalog_file_version_bytes). It is part of the
  manifest digest, so a file replaced in place under the same key and
  size is sent again. Left out when the branch's storage reports none.

Implements: `Clone`, `Debug`, `Deserialize<'de>`, `Eq`, `Ord`, `PartialEq`, `PartialOrd`, `Serialize`

### CatalogMetadata

```rust
pub struct CatalogMetadata {
    pub title: String,
    pub subtitle: Option<String>,
    pub authors: Option<String>,
    pub narrators: Option<String>,
    pub series: Option<String>,
    pub series_index: Option<String>,
    pub asin: Option<String>,
    pub isbn: Option<String>,
    pub publisher: Option<String>,
    pub length_minutes: Option<i64>,
    pub content_kind: Option<String>,
    pub language: Option<String>,
    pub published_at: Option<String>,
}
```

Bibliographic metadata of a [`CatalogTitle`](#catalogtitle).

**Fields**

- `title`: Display title.
- `subtitle`: Subtitle.
- `authors`: Authors as the store lists them.
- `narrators`: Narrators.
- `series`: Series name.
- `series_index`: Position in the series.
- `asin`: Amazon ASIN.
- `isbn`: ISBN.
- `publisher`: Publisher.
- `length_minutes`: Runtime in minutes.
- `content_kind`: `book`, `episode`, ...
- `language`: Language code.
- `published_at`: Publication date text.

Implements: `Clone`, `Debug`, `Default`, `Deserialize<'de>`, `Eq`, `PartialEq`, `Serialize`

### CatalogRef

```rust
pub struct CatalogRef {
    pub store: String,
    pub product_id: String,
    pub marketplace: String,
}
```

A title named by store, product id and marketplace: the identity of a
[`CatalogTitle`](#catalogtitle) on both ends of a catalog sync.

One store product may be sold in several marketplaces (an ASIN on
Audible US and on Audible UK), and each is a title of its own, so the
marketplace is part of the identity.

**Fields**

- `store`: Store id.
- `product_id`: Product id.
- `marketplace`: Lower-cased marketplace, as the title names it ([`is_marketplace`](#is_marketplace));
  empty when the store has none. Left out of the frame when empty.

#### CatalogRef::is_well_formed

```rust
pub fn is_well_formed(&self) -> bool
```

True when every part is well formed: a store id ([`is_store_id`](#is_store_id)), a
product id ([`is_product_id`](#is_product_id)) and a marketplace
([`is_marketplace`](#is_marketplace)). A receiver refuses a reference that is not, so
the parts it stores together never run into one another.

Implements: `Clone`, `Debug`, `Deserialize<'de>`, `Eq`, `Hash`, `Ord`, `PartialEq`, `PartialOrd`, `Serialize`

### CatalogTitle

```rust
pub struct CatalogTitle {
    pub store: String,
    pub product_id: String,
    pub marketplace: String,
    pub metadata: CatalogMetadata,
    pub primary_key: String,
    pub pdf_key: Option<String>,
    pub files: Vec<CatalogFile>,
}
```

One title, as a branch shares it.

**Fields**

- `store`: The store that sells it, as the store's plain id (`audible`,
  `libro`, ...).
- `product_id`: The store's product id (ASIN, ISBN, ...).
- `marketplace`: Lower-cased marketplace of the store listing (`us`, `uk`); empty
  when the store has none. The receiving library scopes the ASIN to
  it, so the title meets the same edition there.
- `metadata`: Bibliographic metadata.
- `primary_key`: Storage key of the primary audio file (one of `files`).
- `pdf_key`: Storage key of the companion PDF (one of `files`), when shared.
- `files`: Every file of the title.

#### CatalogTitle::reference

```rust
pub fn reference(&self) -> CatalogRef
```

The title's [`CatalogRef`](#catalogref).

#### CatalogTitle::problem

```rust
pub fn problem(&self) -> Option<&'static str>
```

Why the title is malformed, or `None`: the store, product id and
marketplace, a non-empty bounded title and bounded metadata, between one and
[`MAX_UPLOAD_FILES`](#max_upload_files) files with distinct storage keys within
[`MAX_UPLOAD_FILE_BYTES`](#max_upload_file_bytes), and the primary key (and PDF key) among
them.

#### CatalogTitle::manifest_digest

```rust
pub fn manifest_digest(&self) -> String
```

Lowercase hex SHA-256 of the title's manifest (primary key, PDF key
and files, sorted by key): equal digests mean the same files.

Implements: `Clone`, `Debug`, `Deserialize<'de>`, `Eq`, `PartialEq`, `Serialize`

### ClerkAuthorizeParams

```rust
pub struct ClerkAuthorizeParams {
    pub client_id: String,
    pub redirect_uri: String,
    pub response_type: String,
    pub scope: Option<String>,
    pub state: Option<String>,
    pub code_challenge: String,
    pub code_challenge_method: Option<String>,
    pub clerk_name: Option<String>,
}
```

The authorization request a clerk consent is for (the query
`/oidc/authorize` forwards to the library's `/clerk/approve` page).

**Fields**

- `client_id`: Must be [`CLERK_CLIENT_ID`](#clerk_client_id).
- `redirect_uri`: Where the code goes: loopback or https.
- `response_type`: Must be `code`.
- `scope`: Must be [`CLERK_SCOPE`](#clerk_scope).
- `state`: Echoed on the redirect.
- `code_challenge`: PKCE S256 challenge.
- `code_challenge_method`: Must be `S256`.
- `clerk_name`: The name the clerk gives itself, shown on the consent page
  and in the user's session list.

Implements: `Clone`, `ComposeSchema`, `Debug`, `Deserialize<'de>`, `Eq`, `IntoParams`, `PartialEq`, `ToSchema`

### ClerkTokenAnswer

```rust
pub struct ClerkTokenAnswer {
    pub token: String,
    pub library_id: String,
    pub user: ClerkUser,
    pub idle_secs: i64,
}
```

`POST /api/clerk/v1/token` answer.

**Fields**

- `token`: The session token, sent as `Authorization: Bookclerk-Clerk <token>`.
  Shown once; the library keeps only its hash.
- `library_id`: The library id.
- `user`: The signed-in person on that library.
- `idle_secs`: Seconds the session stays signed in unused; each use resets it.

Implements: `Clone`, `ComposeSchema`, `Debug`, `Deserialize<'de>`, `Eq`, `PartialEq`, `Serialize`, `ToSchema`

### ClerkTokenRequest

```rust
pub struct ClerkTokenRequest {
    pub code: String,
    pub code_verifier: String,
    pub redirect_uri: String,
}
```

`POST /api/clerk/v1/token` body: exactly these three strings, each
once, and nothing else.

**Fields**

- `code`: The single-use code from the redirect.
- `code_verifier`: The PKCE verifier whose S256 challenge the authorize request sent.
- `redirect_uri`: The redirect URI the authorize request named.

Implements: `Clone`, `ComposeSchema`, `Debug`, `Deserialize<'de>`, `Eq`, `PartialEq`, `Serialize`, `ToSchema`

### ClerkUser

```rust
pub struct ClerkUser {
    pub id: i64,
    pub display_name: Option<String>,
}
```

The person a clerk session signs in as.

**Fields**

- `id`: The user's id on the library.
- `display_name`: The user's display name on the library, when set.

Implements: `Clone`, `ComposeSchema`, `Debug`, `Deserialize<'de>`, `Eq`, `PartialEq`, `Serialize`, `ToSchema`

### StoredClerkCode

```rust
pub struct StoredClerkCode<'a> {
    pub client_id: &'a str,
    pub redirect_uri: &'a str,
    pub method: &'a str,
    pub challenge: &'a str,
}
```

What the library stored with an approved code, as the token exchange
checks it.

**Fields**

- `client_id`: The OAuth client the code was approved for.
- `redirect_uri`: The redirect URI the authorize request named.
- `method`: The PKCE method the authorize request named.
- `challenge`: The PKCE challenge the authorize request sent.

Implements: `Clone`, `Copy`, `Debug`, `Eq`, `PartialEq`

### ClerkCopy

```rust
pub struct ClerkCopy {
    pub store: String,
    pub marketplace: String,
    pub product_id: String,
    pub copy_id: String,
    pub token: CopyToken,
    pub expires_in_secs: u64,
    pub copy: WindowCopy,
    pub files: Vec<CopyFile>,
}
```

What a `copy.granted` frame carries.

**Fields**

- `store`: Store id of the listing the clerk asked with.
- `marketplace`: That listing's marketplace; empty for a store without marketplaces.
- `product_id`: That listing's product id.
- `copy_id`: Copy id; the files are at `COPIES_PATH<copy_id>/files/<n>`.
- `token`: The bearer token for the download endpoint.
- `expires_in_secs`: Seconds the token stays valid.
- `copy`: What the copy is, as the library's page states it.
- `files`: The copy's files; a file's index here is its `<n>`.

#### ClerkCopy::problem

```rust
pub fn problem(&self) -> Option<&'static str>
```

Why the grant is malformed, or `None`: a listing that passes
[`copy_grant_problem`](#copy_grant_problem) and the rest passing
[`copy_granted_problem`](#copy_granted_problem), the token held to the same rule as a
node's sealed one (1 to 1024 base64url characters).

Implements: `Clone`, `Debug`, `Eq`, `PartialEq`

### CopyToken

```rust
pub struct CopyToken(/* private */);
```

A copy token as a clerk socket frame carries it: the bearer of the
download endpoint ([`crate::COPY_AUTH_SCHEME`](#copy_auth_scheme)). Its `Debug` prints no
token, and the text is zeroized when dropped.

#### CopyToken::new

```rust
pub fn new(token: &str) -> Self
```

Wraps `token`.

#### CopyToken::expose

```rust
pub fn expose(&self) -> &str
```

The token text, for the `Authorization` header.

Implements: `Clone`, `Debug`, `Deserialize<'de>`, `Eq`, `PartialEq`, `Serialize`

### CopyFile

```rust
pub struct CopyFile {
    pub name: String,
    pub role: CopyRole,
    pub size: u64,
    pub content_type: Option<String>,
    pub probe: Option<UploadProbe>,
}
```

One file of a copy ([`crate::Body::CopyGranted`](#body)).

**Fields**

- `name`: The file's path below the title's folder on the library, `/`
  separated ([`crate::is_storage_key`](#is_storage_key), at most
  [`MAX_COPY_NAME_BYTES`](#max_copy_name_bytes)). The reader places it under its own
  naming, so this is a hint to what the file is, never a path to
  write to as given.
- `role`: What the file is.
- `size`: Exact size in bytes, at most [`MAX_COPY_FILE_BYTES`](#max_copy_file_bytes).
- `content_type`: The media type the library's probe found, for a probed audio file.
- `probe`: What the library's probe found in an audio file; absent for other
  files and for audio it has not probed.

#### CopyFile::is_well_formed

```rust
pub fn is_well_formed(&self) -> bool
```

True when the name is a relative path within its length limit, the
size is within [`MAX_COPY_FILE_BYTES`](#max_copy_file_bytes), the media type (if stated)
is a media type token and the probe (if stated) is well formed.

Implements: `Clone`, `Debug`, `Deserialize<'de>`, `Eq`, `PartialEq`, `Serialize`

### CopyGrantedFrame

```rust
pub struct CopyGrantedFrame<'a> {
    pub copy_id: &'a str,
    pub sealed_token: &'a str,
    pub expires_in_secs: u64,
    pub copy: &'a WindowCopy,
    pub files: &'a [CopyFile],
}
```

The fields of a [`crate::Body::CopyGranted`](#body), borrowed for
[`copy_granted_problem`](#copy_granted_problem).

**Fields**

- `copy_id`: The copy id.
- `sealed_token`: The sealed token text.
- `expires_in_secs`: Seconds the token stays valid.
- `copy`: What the copy is.
- `files`: The copy's files.

Implements: `Clone`, `Copy`, `Debug`

### AcquireTarget

```rust
pub struct AcquireTarget {
    pub holding_id: String,
    pub plugin_key: String,
    pub account_id: String,
    pub title: TargetTitle,
    pub parent: Option<TargetTitle>,
    pub call_number: Option<CallNumber>,
    pub parent_call_number: Option<CallNumber>,
    pub destinations: Option<Vec<String>>,
}
```

One title an acquire job should fetch on a host
([`Body::AcquireTargets`](#body)).

**Fields**

- `holding_id`: The owner's account holding of the title (`holdings.id`): the id the
  host's upload of it names.
- `plugin_key`: Plugin key of the store that sells it.
- `account_id`: Store account that owns it.
- `title`: The title as the owner's library holds it, for naming and metadata.
- `parent`: The title's podcast parent as the owner's library holds it, when
  the title is an episode whose show an account of the same store
  holds there: a destination that saves episodes to the parent's
  folder names them from it, as an acquire on the owner would.
- `call_number`: The title's call number on the owner (the short id of its edition
  there), which the `<call number>` naming tag writes. A host acquires
  into a scratch library whose edition ids differ, so it names with
  this one to plan the same keys the owner grants.
- `parent_call_number`: The podcast parent's call number on the owner, for a destination that
  saves episodes to the parent's folder.
- `destinations`: The ids of the plan's destinations whose filter takes this title, in
  the plan's order: the title's target set, which the host writes and
  uploads to and nowhere else. An empty list means no destination
  takes it, so the host fails the title without fetching it. `None`
  comes from an owner that predates destination filters and means
  every destination of the plan.

Implements: `Clone`, `Debug`, `Deserialize<'de>`, `Eq`, `PartialEq`, `Serialize`

### CallNumber

```rust
pub struct CallNumber(/* private */);
```

A title's short call number as the owner names it: 1 to
[`MAX_CALL_NUMBER_BYTES`](#max_call_number_bytes) ASCII letters and digits, so it is safe in any
storage key segment. Deserializing anything else fails, which refuses
the whole frame.

#### CallNumber::as_str

```rust
pub fn as_str(&self) -> &str
```

The id as text.

Implements: `Clone`, `Debug`, `Deserialize<'de>`, `Eq`, `From<CallNumber>`, `PartialEq`, `Serialize`, `TryFrom<String>`

### ClaimFilter

```rust
pub struct ClaimFilter {
    pub kinds: Vec<String>,
    pub plugins: Vec<String>,
    pub scope: ClaimScope,
}
```

What a host is willing to claim (`claim.filter`).

**Fields**

- `kinds`: Job kinds the host runs.
- `plugins`: Plugin keys the host's operator consented to.
- `scope`: Whose work the host takes. Absent means [`ClaimScope::Own`](#claimscope).

Implements: `Clone`, `Debug`, `Default`, `Deserialize<'de>`, `Eq`, `PartialEq`, `Serialize`

### FileCopy

```rust
pub struct FileCopy {
    pub destination: String,
    pub key: String,
}
```

Where one uploaded file is written: a destination and its key there.

**Fields**

- `destination`: A [`PlanDestination::id`](#plandestination).
- `key`: Storage key on that destination.

Implements: `Clone`, `Debug`, `Deserialize<'de>`, `Eq`, `Ord`, `PartialEq`, `PartialOrd`, `Serialize`

### Frame

```rust
pub struct Frame {
    pub v: u32,
    pub id: String,
    pub body: Body,
}
```

One frame: the envelope and its body.

**Fields**

- `v`: Envelope version; always [`FRAME_VERSION`](#frame_version).
- `id`: Request id, echoed by the answer.
- `body`: The body, tagged by `type`.

#### Frame::new

```rust
pub fn new(id: impl Into<String>, body: Body) -> Self
```

A version-1 frame.

#### Frame::parse

```rust
pub fn parse(text: &str) -> Result<Self, FrameError>
```

Parses a text frame, checking size and version.

##### Errors

Returns [`FrameError`](#frameerror) for an oversized, malformed, or wrong-version
frame.

#### Frame::to_text

```rust
pub fn to_text(&self) -> String
```

Serializes the frame as JSON text.

Implements: `Clone`, `Debug`, `Deserialize<'de>`, `Eq`, `PartialEq`, `Serialize`

### HeldFence

```rust
pub struct HeldFence {
    pub kind: String,
    pub id: String,
    pub fence: i64,
}
```

A lease the host still holds when it reconnects (`hello.resume`).

**Fields**

- `kind`: `job` or `delivery`.
- `id`: Job id or delivery id.
- `fence`: Lease generation the host holds.

Implements: `Clone`, `Debug`, `Deserialize<'de>`, `Eq`, `PartialEq`, `Serialize`

### JobStatusEntry

```rust
pub struct JobStatusEntry {
    pub id: String,
    pub kind: String,
    pub state: String,
    pub progress: Option<String>,
    pub error_kind: Option<String>,
    pub error_message: Option<String>,
}
```

One job's state in [`Body::JobStatuses`](#body).

**Fields**

- `id`: Job id.
- `kind`: Job kind (`scan`, `acquire`, ...).
- `state`: `pending`, `running`, `succeeded`, `failed`, or `cancelled`.
- `progress`: Progress or result text.
- `error_kind`: Structured error kind when it failed or was cancelled.
- `error_message`: Error text when it failed or was cancelled.

Implements: `Clone`, `ComposeSchema`, `Debug`, `Deserialize<'de>`, `Eq`, `PartialEq`, `Serialize`, `ToSchema`

### PlanDestination

```rust
pub struct PlanDestination {
    pub id: String,
    pub options: Value,
}
```

One output destination of the shared node that answered an acquire plan
([`Body::AcquireTargets`](#body)).

**Fields**

- `id`: Destination id on that shared node: its storage plugin's id, such
  as `local`, `s3` or `r2` (one destination per plugin).
- `options`: That destination's `DownloadOptions` (JSON): its naming templates,
  replacement characters and path limits.

Implements: `Clone`, `Debug`, `Deserialize<'de>`, `Eq`, `PartialEq`, `Serialize`

### RemoteArtifact

```rust
pub struct RemoteArtifact {
    pub target: String,
    pub url: String,
    pub format: String,
    pub archive_root: String,
    pub archive_sha256: String,
    pub payload_root_sha256: String,
    pub executable: String,
}
```

One downloadable artifact of a [`RemoteVersion`](#remoteversion).

**Fields**

- `target`: Bookclerk target (`linux-x64-gnu`, `any`, ...).
- `url`: `https:` URL, or `blob:sha256:<hex>` for an archive the owner stores
  (fetched from the owner's artifact endpoint, [`crate::ARTIFACTS_PATH`](#artifacts_path)).
- `format`: Archive format.
- `archive_root`: Path inside the archive to the plugin root.
- `archive_sha256`: SHA-256 of the archive.
- `payload_root_sha256`: Payload-root digest of the extracted tree.
- `executable`: Executable inside the plugin root.

Implements: `Clone`, `Debug`, `Deserialize<'de>`, `Eq`, `PartialEq`, `Serialize`

### RemoteDeployment

```rust
pub struct RemoteDeployment {
    pub deployment_id: String,
    pub instance_id: String,
    pub plugin_key: String,
    pub desired: String,
    pub revision: i64,
    pub default_instance: bool,
    pub version: Option<RemoteVersion>,
}
```

One deployment that targets a host (`deployments.items`).

**Fields**

- `deployment_id`: Deployment id.
- `instance_id`: Plugin instance the deployment runs.
- `plugin_key`: The instance's plugin key.
- `desired`: Desired state (`present`, `placed`).
- `revision`: Desired-row revision.
- `default_instance`: True for the plugin's default instance (unpinned jobs use it).
- `version`: The plugin's active version; absent when none is installed.

Implements: `Clone`, `Debug`, `Deserialize<'de>`, `Eq`, `PartialEq`, `Serialize`

### RemoteVersion

```rust
pub struct RemoteVersion {
    pub version: String,
    pub manifest_id: String,
    pub manifest_toml: String,
    pub settings: Value,
    pub artifacts: Vec<RemoteArtifact>,
    pub grant_json: String,
    pub publisher_verified: bool,
}
```

The active version of a plugin a host is asked to run.

**Fields**

- `version`: Semver version.
- `manifest_id`: Manifest id (display alias).
- `manifest_toml`: `plugin.toml` text.
- `settings`: Settings schema the version declares (JSON).
- `artifacts`: Per-target artifacts.
- `grant_json`: The grant the library's operator approved (JSON); empty while the
  version is unapproved, in which case nothing runs.
- `publisher_verified`: True when the publisher was verified at install.

Implements: `Clone`, `Debug`, `Deserialize<'de>`, `Eq`, `PartialEq`, `Serialize`

### ScanImport

```rust
pub struct ScanImport {
    pub import_episodes: bool,
    pub import_plus_titles: bool,
}
```

The owner's library import settings a host scan runs with.

**Fields**

- `import_episodes`: Import podcast episodes.
- `import_plus_titles`: Import subscription (Plus) titles.

Implements: `Clone`, `Copy`, `Debug`, `Deserialize<'de>`, `Eq`, `PartialEq`, `Serialize`

### ScanToken

```rust
pub struct ScanToken {
    pub account_id: String,
    pub sync_token: String,
}
```

The sync token an owner holds for one account a host scan covers.

**Fields**

- `account_id`: The store account.
- `sync_token`: The token, at most [`MAX_SCAN_SYNC_TOKEN_BYTES`](#max_scan_sync_token_bytes).

Implements: `Clone`, `Debug`, `Deserialize<'de>`, `Eq`, `PartialEq`, `Serialize`

### TargetTitle

```rust
pub struct TargetTitle {
    pub product_id: String,
    pub source: String,
    pub marketplace: String,
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
    pub is_abridged: bool,
    pub content_kind: String,
    pub purchased_at: Option<String>,
    pub published_at: Option<String>,
}
```

A title an [`AcquireTarget`](#acquiretarget) names, as the owner's library holds it:
the store listing's identity and the metadata a host names the title's
files from and seeds its scratch library with. The store and account
are the target's own.

**Fields**

- `product_id`: The store's product id of the listing (what the store fetches).
- `source`: The store's source name (`audible`, `librofm`, ...), as the
  listing's plugin reports it.
- `marketplace`: The store marketplace the listing is sold in (`us`, `uk`, ...).
- `asin`: The edition's ASIN, when it has one.
- `isbn`: The edition's ISBN, when it has one.
- `title`: The listing's title.
- `subtitle`: The listing's subtitle.
- `authors`: Authors, comma-separated, as the listing gives them.
- `narrators`: Narrators, comma-separated, as the listing gives them.
- `series`: The series name, when the title is in one.
- `series_index`: The title's position in its series, as text (`1`, `2.5`).
- `series_product_id`: The store's product id of the series, or of an episode's show.
- `publisher`: The publisher.
- `categories`: Categories, comma-separated, as the listing gives them.
- `length_minutes`: Running time in whole minutes.
- `is_abridged`: Whether the listing is abridged.
- `content_kind`: `book`, `podcast` or `episode`.
- `purchased_at`: When the account bought it, RFC 3339.
- `published_at`: When it was published, RFC 3339.

Implements: `Clone`, `Debug`, `Deserialize<'de>`, `Eq`, `PartialEq`, `Serialize`

### UploadAnswer

```rust
pub struct UploadAnswer {
    pub status: String,
    pub reason: Option<String>,
    pub offset: Option<u64>,
    pub job_completed: bool,
}
```

Answer of the upload endpoint (every status carries one).

**Fields**

- `status`: Stable outcome: `received`, `partial` (a part that is not the
  file's last was taken), `committed`, `lost`, `refused`,
  `misdirected`, `unauthorized`, `unavailable`.
- `reason`: Why, for `refused`, and `resume` for an `unavailable` part that
  does not continue the open file at [`Self::offset`](#uploadanswer).
- `offset`: For `partial`, and an `unavailable` part with reason `resume`: the
  offset of the next byte of the file the owner needs.
- `job_completed`: For `committed`: the job is finished.

Implements: `Clone`, `ComposeSchema`, `Debug`, `Default`, `Deserialize<'de>`, `Eq`, `PartialEq`, `Serialize`, `ToSchema`

### UploadCommitRequest

```rust
pub struct UploadCommitRequest {
    pub finish: bool,
    pub detail: Option<String>,
}
```

Body of `POST <UPLOADS_PATH><upload_id>/commit`.

**Fields**

- `finish`: Complete the job with this commit (the job's closing upload).
- `detail`: Outcome text recorded as the job's progress when `finish` is set.

Implements: `Clone`, `ComposeSchema`, `Debug`, `Default`, `Deserialize<'de>`, `Eq`, `PartialEq`, `Serialize`, `ToSchema`

### UploadFile

```rust
pub struct UploadFile {
    pub copies: Vec<FileCopy>,
    pub size: u64,
    pub sha256: String,
    pub content_type: Option<String>,
    pub probe: Option<UploadProbe>,
}
```

One file of an upload ([`Body::UploadGrant`](#body)). Its content crosses the
network once and is written to every copy, each destination under the
key that destination's own naming options give it.

**Fields**

- `copies`: Where the file is written; at most one copy per destination.
- `size`: Exact size in bytes.
- `sha256`: Lowercase hex SHA-256 of the content.
- `content_type`: MIME type to store with it.
- `probe`: What the sender's probe found in an audio file, so the library
  keeps it on the file's row without reading the file back; absent
  for other files and for audio the sender could not probe.

#### UploadFile::is_well_formed

```rust
pub fn is_well_formed(&self) -> bool
```

True when the file has between one and [`MAX_UPLOAD_COPIES`](#max_upload_copies)
copies, each naming a different destination id
([`is_destination_id`](#is_destination_id)) and a storage key ([`is_storage_key`](#is_storage_key)); the
size is within [`MAX_UPLOAD_FILE_BYTES`](#max_upload_file_bytes); the digest is lowercase
hex SHA-256; and the probe, if there is one, is well formed
([`UploadProbe::is_well_formed`](#uploadprobeis_well_formed)).

#### UploadFile::key_on

```rust
pub fn key_on(&self, destination: &str) -> Option<&str>
```

This file's key on `destination`, if it is written there.

Implements: `Clone`, `Debug`, `Deserialize<'de>`, `Eq`, `PartialEq`, `Serialize`

### UploadProbe

```rust
pub struct UploadProbe {
    pub bitrate_kbps: Option<u32>,
    pub lossless: bool,
    pub source_store: Option<String>,
}
```

What a node's probe found in an audio file it uploads
([`UploadFile::probe`](#uploadfile)). The media type travels as
[`UploadFile::content_type`](#uploadfile).

The library stores these values as the sender states them: the sender
is a node it trusts with the title, and they decide only which of two
copies of one edition is kept.

**Fields**

- `bitrate_kbps`: Average audio bitrate in kilobits per second, 1 to
  [`MAX_PROBE_BITRATE_KBPS`](#max_probe_bitrate_kbps); absent when the file states no running
  time.
- `lossless`: True for a lossless encoding (FLAC, ALAC).
- `source_store`: Plain id of the storefront the copy was downloaded from
  ([`crate::is_store_id`](#is_store_id)), when the sender knows it.

#### UploadProbe::is_well_formed

```rust
pub fn is_well_formed(&self) -> bool
```

True when the bitrate, if stated, is 1 to
[`MAX_PROBE_BITRATE_KBPS`](#max_probe_bitrate_kbps) and the storefront, if named, is a store
id ([`crate::is_store_id`](#is_store_id)).

Implements: `Clone`, `Debug`, `Deserialize<'de>`, `Eq`, `PartialEq`, `Serialize`

### WorkJob

```rust
pub struct WorkJob {
    pub id: String,
    pub kind: String,
    pub payload: Value,
    pub attempt: i64,
    pub scan_import: Option<ScanImport>,
    pub for_node_owner: bool,
    pub scan_tokens: Vec<ScanToken>,
}
```

A job handed to a host.

**Fields**

- `id`: Job id.
- `kind`: Job kind (`scan`, `acquire`, ...).
- `payload`: Kind-specific payload, as the owner stores it.
- `attempt`: Claims of this job so far, this one included.
- `scan_import`: For a scan: which titles the owner's library imports. The owner
  writes every row a host returns, so the host scans with the
  owner's settings, never its own. Absent for other kinds.
- `for_node_owner`: True when every store account the job touches belongs to the user
  who owns the node: the branch's own books. A branch keeps its own
  copy of another member's book only when its operator asked for
  that. Absent means false.
- `scan_tokens`: For a scan: the sync token the owner holds for each handed account
  whose last complete pass is recent enough to build on. The host's
  guest runs a delta pass from it; an account without one runs a full
  pass. At most [`MAX_SCAN_TOKENS`](#max_scan_tokens) entries. Absent means none.

Implements: `Clone`, `Debug`, `Deserialize<'de>`, `Eq`, `PartialEq`, `Serialize`

### OAuthEnrollAnswer

```rust
pub struct OAuthEnrollAnswer {
    pub library_id: String,
    pub node_id: String,
    pub connection: ConnectionType,
    pub bootstrap: Option<String>,
    pub accounts: Option<String>,
}
```

`POST /api/node/v1/enroll/oauth` answer.

**Fields**

- `library_id`: The library id.
- `node_id`: The node id recorded for the joining install.
- `connection`: The connection type recorded (`branch` when left out).
- `bootstrap`: For a shared branch: the [`crate::SharedBranchBootstrap`](#sharedbranchbootstrap) sealed to the node's box
  key ([`crate::seal_shared_branch_bootstrap`](#seal_shared_branch_bootstrap)), as base64url without padding.
- `accounts`: For a branch whose user chose "Bring my store connections to this
  branch": the user's own store accounts as a
  [`crate::AccountBundle`](#accountbundle) sealed to the node's box key
  ([`crate::seal_account_bundle`](#seal_account_bundle)), as base64url without padding.
  Left out when the user did not choose it or had none to bring.

Implements: `Clone`, `ComposeSchema`, `Debug`, `Deserialize<'de>`, `Eq`, `PartialEq`, `Serialize`, `ToSchema`

### OAuthEnrollRequest

```rust
pub struct OAuthEnrollRequest {
    pub code: String,
    pub code_verifier: String,
    pub redirect_uri: String,
    pub sign_pk: String,
    pub box_pk: String,
    pub branch_name: String,
    pub software_version: String,
    pub connection: ConnectionType,
}
```

`POST /api/node/v1/enroll/oauth` body. Keys are base64url without
padding.

**Fields**

- `code`: The authorization code from the callback.
- `code_verifier`: The PKCE verifier.
- `redirect_uri`: The redirect URI the authorization was requested with.
- `sign_pk`: Ed25519 public signing key.
- `box_pk`: X25519 public box key.
- `branch_name`: The name this branch joins as (the node's label).
- `software_version`: The joining install's Bookclerk version.
- `connection`: The connection type the code was approved for. It must match the
  one bound to the code at consent; left out for a branch.

Implements: `Clone`, `ComposeSchema`, `Debug`, `Deserialize<'de>`, `Eq`, `PartialEq`, `Serialize`, `ToSchema`

### OwnerInfo

```rust
pub struct OwnerInfo {
    pub library_id: String,
    pub library_name: String,
    pub origin: String,
    pub transport: String,
    pub enrollment: bool,
    pub spki_fingerprint: Option<String>,
    pub software_version: String,
    pub ancestry: Vec<String>,
}
```

`GET /api/node/v1/info` answer.

**Fields**

- `library_id`: The library id.
- `library_name`: A name to show for the library.
- `origin`: The library's canonical origin: the one the node handshake signs and
  the authorize endpoint lives under.
- `transport`: The node API transport mode (`acme`, `proxy`, `self_signed`,
  `loopback`).
- `enrollment`: Whether the library lets new branches join (`core.branch_enrollment`).
- `spki_fingerprint`: In `self_signed` mode: base64url SHA-256 of the certificate's
  SubjectPublicKeyInfo, which the joining operator confirms against
  the owner's Settings → Branches page.
- `software_version`: The library's Bookclerk version.
- `ancestry`: The library's ancestry: its own library id first, then the library
  id of every library above it up to the root. A joining library
  refuses a parent whose ancestry names its own library id, so a
  tree of libraries never becomes a cycle.

Implements: `Clone`, `ComposeSchema`, `Debug`, `Deserialize<'de>`, `Eq`, `PartialEq`, `Serialize`, `ToSchema`

### LookupAnswer

```rust
pub struct LookupAnswer {
    pub work: String,
    pub editions: Vec<LookupEdition>,
}
```

A branch's answer to one [`LookupWork`](#lookupwork).

**Fields**

- `work`: The question's [`LookupWork::work`](#lookupwork).
- `editions`: What the branch can supply; 1 to [`MAX_LOOKUP_EDITIONS`](#max_lookup_editions) editions.

#### LookupAnswer::problem

```rust
pub fn problem(&self) -> Option<&'static str>
```

Why the answer is malformed, or `None`: a correlation key, and 1 to
[`MAX_LOOKUP_EDITIONS`](#max_lookup_editions) well-formed editions with distinct
(store, marketplace, product id).

Implements: `Clone`, `Debug`, `Deserialize<'de>`, `Eq`, `PartialEq`, `Serialize`

### LookupEdition

```rust
pub struct LookupEdition {
    pub store: String,
    pub marketplace: String,
    pub product_id: String,
    pub availability: Availability,
    pub formats: Vec<LookupFormat>,
    pub metadata: CatalogMetadata,
}
```

One edition a branch can supply.

**Fields**

- `store`: The store that sells it, as the store's plain id (`audible`,
  `libro`, ...).
- `marketplace`: Lower-cased marketplace of the store listing (`us`); empty when the
  store has none.
- `product_id`: The store's product id (ASIN, ISBN, ...).
- `availability`: Whether the branch holds it or can borrow it.
- `formats`: Formats the branch has; empty when unknown.
- `metadata`: Bibliographic metadata, as the branch's library has it.

#### LookupEdition::problem

```rust
pub fn problem(&self) -> Option<&'static str>
```

Why the edition is malformed, or `None`: a store id, product id and
marketplace within their rules, distinct formats, a non-empty
bounded title and bounded metadata.

Implements: `Clone`, `Debug`, `Deserialize<'de>`, `Eq`, `PartialEq`, `Serialize`

### LookupIdentifier

```rust
pub struct LookupIdentifier {
    pub scheme: LookupScheme,
    pub value: String,
    pub marketplace: String,
}
```

One identifier of the work a question is about.

**Fields**

- `scheme`: The scheme.
- `value`: The normalised value (an ASIN, an ISBN-13, an OpenLibrary id).
- `marketplace`: Lower-cased marketplace for an ASIN; empty for every other scheme.

#### LookupIdentifier::is_well_formed

```rust
pub fn is_well_formed(&self) -> bool
```

True when the value is 1 to 64 ASCII letters, digits and `-`, and
the marketplace is set exactly for an ASIN, as up to 16 lowercase
ASCII letters and digits.

Implements: `Clone`, `Debug`, `Deserialize<'de>`, `Eq`, `Hash`, `Ord`, `PartialEq`, `PartialOrd`, `Serialize`

### LookupWork

```rust
pub struct LookupWork {
    pub work: String,
    pub identifiers: Vec<LookupIdentifier>,
}
```

One question: "can you supply this work?".

**Fields**

- `work`: The asking library's work id, echoed in the answer; opaque to the
  branch.
- `identifiers`: The work's identifiers; at least one, at most
  [`MAX_LOOKUP_IDENTIFIERS`](#max_lookup_identifiers).

#### LookupWork::problem

```rust
pub fn problem(&self) -> Option<&'static str>
```

Why the question is malformed, or `None`: a correlation key
([`is_work_ref`](#is_work_ref)) and 1 to [`MAX_LOOKUP_IDENTIFIERS`](#max_lookup_identifiers) distinct
well-formed identifiers.

Implements: `Clone`, `Debug`, `Deserialize<'de>`, `Eq`, `PartialEq`, `Serialize`

### BranchOrigin

```rust
pub struct BranchOrigin(/* private */);
```

A branch's public origin: `http` or `https`, a host, an optional port and
nothing else (no user, path, query or fragment), at most
[`MAX_BRANCH_ORIGIN_BYTES`](#max_branch_origin_bytes) bytes, in the form [`Url`](https://docs.rs/url/latest/url/struct.Url.html) serializes it
(lowercase scheme and host, default port dropped).

#### BranchOrigin::parse

```rust
pub fn parse(raw: &str) -> Option<Self>
```

Parses the origin a branch reported.

Surrounding whitespace and one trailing `/` are ignored, as in the
branch's own `integrations.public_origin` setting.

##### Arguments

* `raw` - The `public_origin` text from a heartbeat frame.

##### Returns

The normalized origin, or `None` when the text is not a plain
`http(s)` origin within the size limit.

#### BranchOrigin::as_str

```rust
pub fn as_str(&self) -> &str
```

The origin as text, such as `https://branch.example:8443`.

#### BranchOrigin::into_string

```rust
pub fn into_string(self) -> String
```

The origin as owned text.

Implements: `Clone`, `Debug`, `Eq`, `PartialEq`

### BranchRequest

```rust
pub struct BranchRequest {
    pub job: String,
    pub generation: i64,
    pub work: LookupWork,
    pub store: String,
    pub marketplace: String,
    pub product_id: String,
}
```

One request a branch claimed: "send this library that book".

**Fields**

- `job`: The library's job id; the branch echoes it when it finishes.
- `generation`: The job's lease generation; together with the job id and the
  branch's node id it is the fence the finish is checked against.
- `work`: The work, by identifier, as a catalog question names it.
- `store`: The store of the listing the branch answered with, as its plain id
  (`audible`).
- `marketplace`: Lower-cased marketplace of that listing; empty when the store has
  none.
- `product_id`: The store's product id of that listing.

#### BranchRequest::problem

```rust
pub fn problem(&self) -> Option<&'static str>
```

Why the request is malformed, or `None`: a job id
([`is_request_job`](#is_request_job)), a positive generation, a well-formed work, and
a store id, marketplace and product id within the catalog rules.

Implements: `Clone`, `Debug`, `Deserialize<'de>`, `Eq`, `PartialEq`, `Serialize`

### SecretEntry

```rust
pub struct SecretEntry {
    pub secret_ref: SecretRef,
    pub value: SecretValue,
}
```

One secret in a bundle.

**Fields**

- `secret_ref`: Which row the value belongs to.
- `value`: The unsealed value.

Implements: `Clone`, `Debug`, `Deserialize<'de>`, `Eq`, `PartialEq`, `Serialize`

### SecretRef

```rust
pub struct SecretRef {
    pub kind: String,
    pub provider: String,
    pub account_id: String,
    pub name: String,
}
```

Names one `encrypted_secrets` row of a store account. It carries no
secret material.

**Fields**

- `kind`: Secret kind (`source_auth`, `widevine`, ...).
- `provider`: Plugin key that owns the secret.
- `account_id`: Store account id.
- `name`: Logical secret name within that account.

Implements: `Clone`, `Debug`, `Deserialize<'de>`, `Eq`, `Hash`, `Ord`, `PartialEq`, `PartialOrd`, `Serialize`

### SecretValue

```rust
pub struct SecretValue(/* private */);
```

Secret bytes: zeroized on drop, redacted in `Debug`, and base64url on
the wire.

#### SecretValue::new

```rust
pub fn new(bytes: Vec<u8>) -> Self
```

Wraps secret bytes.

#### SecretValue::expose

```rust
pub fn expose(&self) -> &[u8]
```

The secret bytes.

#### SecretValue::len

```rust
pub fn len(&self) -> usize
```

Length in bytes.

#### SecretValue::is_empty

```rust
pub fn is_empty(&self) -> bool
```

True when empty.

Implements: `Clone`, `Debug`, `Deserialize<'de>`, `Eq`, `PartialEq`, `Serialize`

### SharedBranchBootstrap

```rust
pub struct SharedBranchBootstrap {
    pub v: u32,
    pub library_id: String,
    pub node_id: String,
    pub database: SharedBranchDatabase,
    pub database_credential: Option<SharedBranchCredential>,
    pub dek: SecretValue,
}
```

What a shared branch receives at redeem: the `[database]` bootstrap and the DEK,
for the node id the library recorded (`kind = shared`). `Debug` prints no
secret.

**Fields**

- `v`: [`SHARED_BRANCH_BOOTSTRAP_VERSION`](#shared_branch_bootstrap_version).
- `library_id`: The library id.
- `node_id`: The node id recorded for this shared branch.
- `database`: How to reach the library's database.
- `database_credential`: The database plugin's own credential, when `database` doesn't carry
  it. Never sent with a Postgres `database`, whose URL is its
  credential; left out when the library has none to give (a D1
  library whose token is configured only where it runs).
- `dek`: The library's 32-byte data-encryption key.

Implements: `Clone`, `Debug`, `Deserialize<'de>`, `Eq`, `PartialEq`, `Serialize`

### SharedBranchCredential

```rust
pub struct SharedBranchCredential {
    pub secret: SecretValue,
    pub secret_id: Option<String>,
}
```

The database plugin's own credential, for a plugin whose connection
settings don't carry it (D1's API token; a Postgres URL carries its
own). Whatever the plugin, the shared branch keeps it sealed with the DEK
in its master key store (file, OS keychain or `systemd-creds`), tagged
with the plugin it belongs to. `Debug` prints no secret.

**Fields**

- `secret`: The secret: 1 to [`MAX_BOOTSTRAP_CREDENTIAL_BYTES`](#max_bootstrap_credential_bytes) bytes of UTF-8
  with no whitespace or control character.
- `secret_id`: The provider's id for the secret, when the guided setup minted it
  (so a later setup on the shared branch can replace it): 1 to
  [`MAX_BOOTSTRAP_CREDENTIAL_ID_BYTES`](#max_bootstrap_credential_id_bytes) ASCII letters, digits, `-` and
  `_`.

Implements: `Clone`, `Debug`, `Deserialize<'de>`, `Eq`, `PartialEq`, `Serialize`

### ArtifactRequest

```rust
pub struct ArtifactRequest<'a> {
    pub library_id: &'a str,
    pub node_id: &'a str,
    pub owner_origin: &'a str,
    pub archive_sha256: &'a str,
    pub issued_at: i64,
}
```

What a signed artifact request covers.

**Fields**

- `library_id`: Library id.
- `node_id`: Requesting node id.
- `owner_origin`: Owner origin as [`canonical_origin`](#canonical_origin) spells it.
- `archive_sha256`: Lowercase hex SHA-256 of the archive asked for.
- `issued_at`: Unix seconds when the host signed the request.

Implements: `Clone`, `Copy`, `Debug`, `Eq`, `PartialEq`

### AuthContext

```rust
pub struct AuthContext<'a> {
    pub library_id: &'a str,
    pub node_id: &'a str,
    pub incarnation: &'a str,
    pub nonce: &'a [u8; 32],
    pub owner_origin: &'a str,
}
```

What a handshake signature covers.

**Fields**

- `library_id`: Library id from the challenge.
- `node_id`: Node id from the hello.
- `incarnation`: Incarnation from the hello.
- `nonce`: Nonce from the challenge.
- `owner_origin`: Owner origin as [`canonical_origin`](#canonical_origin) spells it. The host uses the
  origin it dialed; the owner uses its own configured origin, so a
  signature relayed through another server does not verify.

Implements: `Clone`, `Copy`, `Debug`, `Eq`, `PartialEq`

### UploadPart

```rust
pub struct UploadPart {
    pub offset: u64,
    pub length: u64,
}
```

A part of a declared file: at least one byte, inside the file.

**Fields**

- `offset`: Offset of the part's first byte in the file.
- `length`: Bytes in the part; at least 1.

#### UploadPart::end

```rust
pub fn end(&self) -> u64
```

Offset just past the part's last byte.

#### UploadPart::is_last

```rust
pub fn is_last(&self, size: u64) -> bool
```

Whether the part ends the file of `size` bytes.

#### UploadPart::content_range

```rust
pub fn content_range(&self, size: u64) -> String
```

The `Content-Range` value naming this part of a file of `size`
bytes, as [`parse_upload_range`](#parse_upload_range) reads it.

Implements: `Clone`, `Copy`, `Debug`, `Eq`, `PartialEq`

### WindowCopy

```rust
pub struct WindowCopy {
    pub source_store: Option<String>,
    pub content_type: String,
    pub bitrate_kbps: Option<u32>,
    pub lossless: bool,
    pub version: String,
}
```

The copy of an edition's primary audio file a library holds, as its
probe of the file found it: enough for a reader to decide whether to
take the library's copy or fetch its own, without asking again.

A hint, like [`WindowEdition::yours`]: the library checks what it hands
out when a copy is asked for.

**Fields**

- `source_store`: The storefront the library's copy was downloaded from (a store id);
  absent when the library does not know. It can differ from the
  edition's own store when the same edition is sold by several.
- `content_type`: The file's media type, such as `audio/mp4`; 1 to
  [`MAX_COPY_CONTENT_TYPE_BYTES`](#max_copy_content_type_bytes) bytes of a `type/subtype` token.
- `bitrate_kbps`: Average bitrate in kbps, 1 to [`crate::MAX_PROBE_BITRATE_KBPS`](#max_probe_bitrate_kbps);
  absent when the file was not probed or states no running time.
- `lossless`: True when the audio encoding is lossless; false also for a file
  that was not probed.
- `version`: An opaque token that changes whenever the library's file is
  replaced: 1 to [`MAX_COPY_VERSION_BYTES`](#max_copy_version_bytes) ASCII letters, digits and
  `-`. A reader keeps it to notice a newer copy.

#### WindowCopy::is_well_formed

```rust
pub fn is_well_formed(&self) -> bool
```

True when every field is within its documented shape.

Implements: `Clone`, `Debug`, `Deserialize<'de>`, `Eq`, `PartialEq`, `Serialize`

### WindowCursor

```rust
pub struct WindowCursor(/* private */);
```

Where a page of the person's books starts: an opaque key the library
handed out as the previous page's `next`, or empty for the first page.

A cursor is empty or, like a grouping key ([`is_work_ref`](#is_work_ref)), at most
[`crate::lookup::MAX_WORK_REF_BYTES`](#max_work_ref_bytes) bytes of ASCII letters, digits and `-`. Cursors
order as strings, and each page's `next` sorts after the cursor it
answers, so paging always moves forward. On the wire it is a JSON
string; anything else is refused.

#### WindowCursor::start

```rust
pub fn start() -> Self
```

The cursor of the first page.

#### WindowCursor::parse

```rust
pub fn parse(text: &str) -> Option<Self>
```

Reads a cursor: `None` unless `text` is empty or a well-formed key.

#### WindowCursor::is_start

```rust
pub fn is_start(&self) -> bool
```

True for the first page's cursor.

#### WindowCursor::as_str

```rust
pub fn as_str(&self) -> &str
```

The cursor's text.

Implements: `Clone`, `Debug`, `Default`, `Deserialize<'de>`, `Display`, `Eq`, `From<WindowCursor>`, `Hash`, `Ord`, `PartialEq`, `PartialOrd`, `Serialize`, `TryFrom<String>`

### WindowEdition

```rust
pub struct WindowEdition {
    pub edition: LookupEdition,
    pub yours: bool,
    pub copy: Option<WindowCopy>,
}
```

One edition of a [`WindowWork`]: the edition as the library holds it,
and whether it is the person's own.

**Fields**

- `edition`: The edition, with every format any of the library's copies has.
- `yours`: True when the person holds it through a store account of their own
  on the library; false for a book only other people there hold.
  A hint for the reader: anything the library hands out on the
  strength of it is checked again there.
- `copy`: The copy of the edition's audio the library holds, when it has
  acquired one and knows its media type; absent otherwise.

Implements: `Clone`, `Debug`, `Deserialize<'de>`, `Eq`, `PartialEq`, `Serialize`

### WindowWork

```rust
pub struct WindowWork {
    pub work: String,
    pub identifiers: Vec<LookupIdentifier>,
    pub editions: Vec<WindowEdition>,
}
```

One work the library holds for the window's person.

**Fields**

- `work`: The library's own work id (or another key unique within the page)
  that groups these editions; opaque to the window and never compared
  across libraries.
- `identifiers`: The work's identifiers in the library, strong ones first; at most
  [`MAX_LOOKUP_IDENTIFIERS`](#max_lookup_identifiers), and empty when the library has none.
- `editions`: The library's editions of the work; 1 to [`MAX_LOOKUP_EDITIONS`](#max_lookup_editions),
  each [`Availability::Owned`](#availability) (the library holds it on a store
  account) or, on a clerk's page, [`Availability::Reachable`](#availability) (only a
  branch of the library lists it; never the person's and with no
  copy).

#### WindowWork::problem

```rust
pub fn problem(&self) -> Option<&'static str>
```

Why the work is malformed, or `None`: a grouping key
([`is_work_ref`](#is_work_ref)), at most [`MAX_LOOKUP_IDENTIFIERS`](#max_lookup_identifiers) distinct
well-formed identifiers, and 1 to [`MAX_LOOKUP_EDITIONS`](#max_lookup_editions) well-formed
editions, with distinct (store, marketplace, product id) and, where
one is stated, a well-formed copy ([`WindowCopy::is_well_formed`](#windowcopyis_well_formed)).
An edition is [`Availability::Owned`](#availability), or [`Availability::Reachable`](#availability)
with neither a copy nor [`WindowEdition::yours`](#windowedition).

Implements: `Clone`, `Debug`, `Deserialize<'de>`, `Eq`, `PartialEq`, `Serialize`

## Enums

### ClerkFrame

```rust
pub enum ClerkFrame {
    Books { after: WindowCursor },
    CopyGrant { store: String, marketplace: String, product_id: String },
}
```

A frame from a clerk to a library.

**Variants**

- `Books`: `books`: the page of the library's books after `after` (empty for
  the first page).
  - `after`: The cursor of the previous page.
- `CopyGrant`: `copy.grant`: ask for the library's copy of one edition, named by
  its store listing; the person is the card's.
  - `store`: Store id of the edition's listing.
  - `marketplace`: The listing's marketplace; empty for a store without
    marketplaces.
  - `product_id`: The listing's product id.

#### ClerkFrame::encode

```rust
pub fn encode(&self) -> String
```

The frame's JSON text, as a clerk sends it.

Implements: `Clone`, `Debug`, `Eq`, `PartialEq`

### ClerkSocketError

```rust
pub enum ClerkSocketError {
    Invalid,
    Unavailable,
    NotFound,
}
```

Why the library did not answer a `books` or `copy.grant` frame.

**Variants**

- `Invalid`: The clerk sent a frame the library could not read.
- `Unavailable`: The library could not read the books just now; ask again later.
- `NotFound`: `copy.grant` only: the person may not take that edition, or the
  library holds no copy of it. One code for both.

Implements: `Clone`, `Copy`, `Debug`, `Deserialize<'de>`, `Eq`, `PartialEq`, `Serialize`

### ClerkTopic

```rust
pub enum ClerkTopic {
    Library,
    Discover,
}
```

Something a clerk hears about on the socket. Each names the routes to
read again; the socket itself carries no row data.

**Variants**

- `Library`: The library's books changed: read them again.
- `Discover`: The person's stored Discover shelves may have changed.

#### ClerkTopic::ALL

```rust
pub const ALL: [Self; 2] = _
```

Every topic.

#### ClerkTopic::as_str

```rust
pub fn as_str(self) -> &'static str
```

The topic's wire name.

#### ClerkTopic::from_name

```rust
pub fn from_name(name: &str) -> Option<Self>
```

The topic named `name`, or `None` for a name this version does not
know.

Implements: `Clone`, `Copy`, `Debug`, `Deserialize<'de>`, `Eq`, `Hash`, `Ord`, `PartialEq`, `PartialOrd`, `Serialize`

### LibraryFrame

```rust
pub enum LibraryFrame {
    Hello { topics: BTreeSet<ClerkTopic> },
    Changed { topics: BTreeSet<ClerkTopic> },
    BooksPage { after: WindowCursor, works: Vec<WindowWork>, next: Option<WindowCursor> },
    CopyGranted(Box<ClerkCopy>),
    Error { code: ClerkSocketError },
}
```

A frame from a library to a clerk.

**Variants**

- `Hello`: `hello`: the topics this card hears about, sent once.
  - `topics`: The topics, without names this version does not know.
- `Changed`: `changed`: these topics changed.
  - `topics`: The topics, without names this version does not know.
- `BooksPage`: `books.page`: one page of the library's books, the card's person's
  own marked ([`crate::WindowEdition::yours`](#windowedition)).
  - `after`: The cursor the clerk asked with.
  - `works`: At most [`crate::MAX_WINDOW_WORKS`](#max_window_works) works.
  - `next`: The cursor of the next page; `None` on the last page.
- `CopyGranted`: `copy.granted`: the copy may be downloaded.
- `Error`: `error`: a `books` or `copy.grant` frame was not answered.
  - `code`: Why.

#### LibraryFrame::encode

```rust
pub fn encode(&self) -> String
```

The frame's JSON text, as a library sends it.

Implements: `Clone`, `Debug`, `Eq`, `PartialEq`

### CopyAnswer

```rust
pub enum CopyAnswer {
    Body { offset: u64, length: u64 },
    Status(u16),
}
```

What the head of a download's answer says follows it
([`parse_copy_answer`](#parse_copy_answer)).

**Variants**

- `Body`: `length` bytes of the file follow, from byte `offset` to its end:
  `offset + length` is the file's size, and `offset` is 0 or the
  byte the reader asked to resume from.
  - `offset`: First byte of the file the body holds.
  - `length`: Number of bytes that follow.
- `Status`: The library answered with this status and no file bytes: a refusal
  (401, 404), an unsatisfiable range (416) or a fault.

Implements: `Clone`, `Copy`, `Debug`, `Eq`, `PartialEq`

### CopyRange

```rust
pub enum CopyRange {
    Whole,
    Part { offset: u64, length: u64 },
    Unsatisfiable,
}
```

What a download's `Range` header asks for ([`parse_copy_range`](#parse_copy_range)).

**Variants**

- `Whole`: The whole file (answer 200): no usable range was named.
- `Part`: `length` bytes from `offset` (answer 206); `length` is at least 1
  and the range lies within the file.
  - `offset`: First byte.
  - `length`: Number of bytes.
- `Unsatisfiable`: A well-formed range that starts at or past the end of the file
  (answer 416).

Implements: `Clone`, `Copy`, `Debug`, `Eq`, `PartialEq`

### CopyRole

```rust
pub enum CopyRole {
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

What a file of a copy is, as the library recorded it when it wrote the
file.

**Variants**

- `Audio`: The edition's primary audio file.
- `Part`: One part of a multi-part audio edition.
- `Chapter`: One chapter file.
- `Pdf`: The companion PDF.
- `Cover`: The cover image.
- `Sidecar`: A metadata sidecar.
- `Transcript`: A transcript.
- `Epub`: The ebook.

Implements: `Clone`, `Copy`, `Debug`, `Deserialize<'de>`, `Eq`, `Ord`, `PartialEq`, `PartialOrd`, `Serialize`

### Body

```rust
pub enum Body {
    Hello { node_id: String, incarnation: String, software_version: String, resume: Vec<HeldFence> },
    Challenge { nonce: String, library_id: String },
    Auth { sig: String },
    Welcome { node_id: String, library_id: String, box_key: String },
    Resumed { renewed: Vec<HeldFence>, lost: Vec<HeldFence>, unchecked: Vec<HeldFence>, lease_secs: Option<u64> },
    Error { code: String, message: String },
    Heartbeat { started_at: String, interval_secs: u64, cpu_millicores: i64, memory_bytes: Option<i64>, cpu_used_millicores: Option<i64>, memory_available_bytes: Option<i64>, files_dir_free_bytes: Option<i64>, public_origin: Option<String>, budget_cpu_millicores: Option<i64>, budget_memory_bytes: Option<i64> },
    Claim { item: String, class: Option<String>, filter: Option<ClaimFilter> },
    Work { item: String, job: WorkJob, fence: i64, lease_secs: u64, sealed_secrets: Option<String>, plugin_keys: Vec<String>, source_plugin_key: Option<String>, resumed: bool },
    NoWork { retry_after_secs: u64 },
    Renew { item: String, job_id: String, fence: i64, progress: Option<String> },
    Renewed { lease_secs: u64, cancel: bool },
    Lost { item: String, job_id: String, fence: i64 },
    Complete { job_id: String, fence: i64, detail: Option<String> },
    Fail { job_id: String, fence: i64, error_kind: String, message: String },
    Release { job_id: String, fence: i64, reason: String },
    DeploymentsList { after: Option<String> },
    Deployments { items: Vec<RemoteDeployment>, next: Option<String> },
    InstanceConfigGet { instance_id: String },
    InstanceConfig { instance_id: String, revision: i64, config: Value, sealed_secrets: Option<String>, secrets_withheld: bool },
    Observation { deployment_id: String, status: String, detail: String, applied_config_revision: Option<i64> },
    SecretUpdate { job_id: String, fence: i64, secret_ref: SecretRef, sealed_value: String },
    ScanResults { job_id: String, fence: i64, plugin_key: String, batch: u64, books: Vec<Value> },
    ScanAccepted { batch: u64, upserted: u64 },
    ScanRemoved { job_id: String, fence: i64, plugin_key: String, account_id: String, batch: u64, product_ids: Vec<String> },
    ScanPassEnd { job_id: String, fence: i64, plugin_key: String, account_id: String, sync_token: String, complete: bool, full: bool },
    AcquirePlan { job_id: String, fence: i64, after: Option<String> },
    AcquireTargets { targets: Vec<AcquireTarget>, destinations: Vec<PlanDestination>, next: Option<String> },
    UploadGrant { job_id: String, fence: i64, holding_id: String, primary_key: String, pdf_key: Option<String>, files: Vec<UploadFile> },
    UploadGranted { upload_id: String, sealed_token: String, expires_in_secs: u64 },
    ScanRequest { account: Option<String>, plugin_instance_id: Option<String> },
    ScanPlanned { created: Vec<String>, duplicates: Vec<String>, queue_full: u64 },
    JobStatus { job_ids: Vec<String> },
    JobStatuses { jobs: Vec<JobStatusEntry> },
    Wake { message: Value },
    Ack {  },
    NodeLeaving {  },
    CatalogBegin { share_books: bool, include_branches: bool },
    CatalogSession { fence: i64, cursor: String, ancestry: Vec<String>, destinations: Vec<String>, wanted: Vec<CatalogRef>, members_books: bool },
    CatalogSync { fence: i64, batch: u64, cursor: String, done: bool, titles: Vec<CatalogTitle> },
    CatalogAccepted { batch: u64, upserted: u64, skipped: u64, wanted: Vec<CatalogRef> },
    CatalogUpload { fence: i64, store: String, product_id: String, marketplace: String, files: Vec<UploadFile> },
    AccountsAdd { sealed: String },
    AccountsAdded { added: u64, already_here: u64, taken: u64, failed: u64, not_allowed: bool },
    AccountsRemove { plugin_key: String, account_id: String },
    AccountsUpdate { sealed: String, revision: i64, keeps_own: bool },
    AccountsUpdated { outcome: KeeperOutcome, sealed: Option<String> },
    CatalogQuestions {  },
    CatalogAsk { works: Vec<LookupWork> },
    CatalogFound { answers: Vec<LookupAnswer>, missing: Vec<String> },
    RequestsClaim {  },
    RequestsClaimed { requests: Vec<BranchRequest> },
    RequestFinish { job: String, generation: i64, outcome: RequestOutcome },
    WindowBooks { after: WindowCursor },
    WindowPage { works: Vec<WindowWork>, next: Option<WindowCursor> },
    CopyGrant { store: String, marketplace: String, product_id: String },
    CopyGranted { copy_id: String, sealed_token: String, expires_in_secs: u64, copy: WindowCopy, files: Vec<CopyFile> },
    Accepted {  },
    Unknown,
}
```

What a frame says, tagged by `type`.

**Variants**

- `Hello`: Host → owner: first frame.
  - `node_id`: Node id the owner minted at enrollment.
  - `incarnation`: Host process incarnation (new on each start).
  - `software_version`: Host's Bookclerk version.
  - `resume`: Leases the host still holds from an earlier connection.
- `Challenge`: Owner → host: the nonce to sign.
  - `nonce`: 32 random bytes, base64url without padding.
  - `library_id`: Library id (also signed).
- `Auth`: Host → owner: proof of key possession.
  - `sig`: Ed25519 signature over [`crate::auth_transcript`](#auth_transcript), base64url
    without padding.
- `Welcome`: Owner → host: the handshake succeeded.
  - `node_id`: Authenticated node id.
  - `library_id`: Library id.
  - `box_key`: The owner's X25519 box key for this connection, base64url
    without padding. The host seals token write-backs to it
    (`crate::seal_secret_update`), so no secret travels as frame
    text either way.
- `Resumed`: Owner → host, right after [`Body::Welcome`](#body) when the hello listed
  fences: which leases the owner renewed, which are lost, and which it
  could not check now. The host aborts lost work; every later frame
  under a lost fence is refused.
  - `renewed`: Leases renewed for this connection.
  - `lost`: Leases that expired, were taken over, or are no longer trusted.
  - `unchecked`: Leases the owner could not check (a transient read or write
    failure). They are neither renewed nor lost: the host keeps the
    work and renews them as usual, and that `renew` decides.
  - `lease_secs`: The lease, in seconds, the `renewed` fences now run for, counted
    from the host's `hello`. The owner's lease length may have
    changed while the host was away, so the host counts this rather
    than its original grant; absent when nothing was renewed.
- `Error`: Either side: a request was not served. The connection stays open.
  - `code`: Stable code (`unsupported`, `invalid`, ...).
  - `message`: Human-readable detail.
- `Heartbeat`: Host → owner: the host's own measurements, recorded in
  `node_status` like any node's heartbeat. Answered with
  [`Body::Accepted`](#body).
  - `started_at`: RFC 3339 time the host daemon started.
  - `interval_secs`: Promised heartbeat interval in seconds.
  - `cpu_millicores`: Detected CPU capacity in millicores.
  - `memory_bytes`: Detected memory in bytes.
  - `cpu_used_millicores`: Measured CPU in use, in millicores.
  - `memory_available_bytes`: Measured memory available to new work.
  - `files_dir_free_bytes`: Free bytes on the host's files directory.
  - `public_origin`: The branch's `integrations.public_origin`, the address other
    installs reach it at; absent for a branch that has none (a
    local branch). The library keeps it only when
    [`crate::BranchOrigin::parse`](#branchoriginparse) accepts it.
  - `budget_cpu_millicores`: The CPU, in millicores, of the branch's own budget it gives
    this library (its `node.runtime` budget cut to this library's
    share of the machine). The library records the smaller of this
    and its own budget for the node. Absent from a branch that
    reports none.
  - `budget_memory_bytes`: The memory, in bytes, of the branch's own budget it gives this
    library, like `budget_cpu_millicores`.
- `Claim`: Host → owner: pull one work item. Answered with [`Body::Work`](#body) or
  [`Body::NoWork`](#body).
  - `item`: [`ITEM_JOB`](#item_job) (deliveries are not served yet).
  - `class`: Job resource class; `network` when absent.
  - `filter`: The host's own narrowing: only jobs of these kinds whose every
    plugin its operator consented to, and whose work ([`ClaimScope`](#claimscope)).
    Absent claims everything the node is trusted with.
- `Work`: Owner → host: a claimed job, its lease, and its sealed secrets.
  - `item`: [`ITEM_JOB`](#item_job).
  - `job`: The job.
  - `fence`: Lease generation; every later frame for the job names it.
  - `lease_secs`: Lease length; renew well within it.
  - `sealed_secrets`: The job's secrets sealed to the node's box key, bound to the
    node, job and fence (`crate::seal_secrets`). Absent when the job
    needs none.
  - `plugin_keys`: Plugin keys the job runs; the host runs it only when it still
    consents to every one.
  - `source_plugin_key`: The one plugin of `plugin_keys` whose source guest runs the
    job (the others are destinations it writes); the owner hands a
    host only jobs that name one.
  - `resumed`: The claim resumed a suspended job: no attempt was spent, and the
    job's checkpoint applies.
- `NoWork`: Owner → host: nothing to claim now.
  - `retry_after_secs`: Seconds to wait before claiming again.
- `Renew`: Host → owner: extend a lease. Answered with [`Body::Renewed`](#body) or
  [`Body::Lost`](#body).
  - `item`: [`ITEM_JOB`](#item_job).
  - `job_id`: Job id.
  - `fence`: Lease generation the host holds.
  - `progress`: Progress text to record.
- `Renewed`: Owner → host: the lease was extended.
  - `lease_secs`: Lease length from now.
  - `cancel`: The operator asked to cancel; stop and report `fail`.
- `Lost`: Owner → host: the fence no longer owns the item; abort it and drop
  its secrets.
  - `item`: [`ITEM_JOB`](#item_job).
  - `job_id`: Job id.
  - `fence`: The fence that was refused.
- `Complete`: Host → owner: the job finished. Answered with [`Body::Accepted`](#body) or
  [`Body::Lost`](#body).
  - `job_id`: Job id.
  - `fence`: Lease generation the host holds.
  - `detail`: Outcome text recorded as the job's progress.
- `Fail`: Host → owner: the job failed. Answered with [`Body::Accepted`](#body) or
  [`Body::Lost`](#body).
  - `job_id`: Job id.
  - `fence`: Lease generation the host holds.
  - `error_kind`: Stable error kind.
  - `message`: Operator-facing message (never a secret).
- `Release`: Host → owner: give a job back unrun (its operator no longer
  consents to the job's plugin, or its claim timed out before the
  `work` answer arrived). The job returns to pending without spending
  the attempt; whether the claim was a resume is the owner's own
  record, never the host's. Answered with [`Body::Accepted`](#body) or
  [`Body::Lost`](#body).
  - `job_id`: Job id.
  - `fence`: Lease generation the host holds.
  - `reason`: Stable reason (`awaiting_host_consent`, `claim_abandoned`, ...),
    for the log.
- `DeploymentsList`: Host → owner: one page of the deployments that target this host and
  should run. Answered with [`Body::Deployments`](#body).
  - `after`: The `next` of the previous page; absent for the first page.
- `Deployments`: Owner → host: one page of the answer to [`Body::DeploymentsList`](#body),
  as many entries as fit a frame.
  - `items`: One entry per deployment, in deployment-id order, each after
    the request's `after`.
  - `next`: Where the next page starts (the last entry's deployment id);
    absent on the last page.
- `InstanceConfigGet`: Host → owner: `CONFIG` and `SECRETS` of an instance deployed to this
  host. Answered with [`Body::InstanceConfig`](#body), or `error {code:
  "not_found"}` for an instance not deployed here.
  - `instance_id`: Plugin instance id.
- `InstanceConfig`: Owner → host: the answer to [`Body::InstanceConfigGet`](#body).
  - `instance_id`: Plugin instance id.
  - `revision`: Instance document revision these values apply.
  - `config`: `CONFIG`: schema defaults overlaid with the document.
  - `sealed_secrets`: `SECRETS` as a JSON object, sealed to the node's box key and bound
    to the node, instance and revision
    (`crate::seal_instance_secrets`). Absent when the instance has
    none or they were withheld.
  - `secrets_withheld`: True when the instance has secrets this node is not trusted
    with (it receives them only when its owner may see every job's
    secrets).
- `Observation`: Host → owner: the host's observation of one of its deployments
  (`materialized`, or `error` with a detail such as
  `awaiting_host_consent`). Answered with [`Body::Accepted`](#body).
  - `deployment_id`: Deployment id.
  - `status`: `materialized` or `error`.
  - `detail`: Bounded detail.
  - `applied_config_revision`: Instance document revision the host applied.
- `SecretUpdate`: Host → owner: write back a refreshed store token. Accepted only for a
  secret handed out with the job under this still-live fence.
  - `job_id`: Job the secret was handed with.
  - `fence`: Lease generation it was handed under.
  - `secret_ref`: The secret row.
  - `sealed_value`: The new value sealed to the connection's owner box key
    ([`Body::Welcome`](#body)), bound to the node, job, fence and ref
    (`crate::seal_secret_update`).
- `ScanResults`: Host → owner: one batch of a scan's book rows (decision 6). The
  owner checks the fence, the node's trust with the job, and every
  row's account before it writes any row to its library, on every
  batch. Answered with [`Body::ScanAccepted`](#body), [`Body::Lost`](#body), or
  `error {code: "refused"}`.
  - `job_id`: Job the rows belong to.
  - `fence`: Lease generation the host holds.
  - `plugin_key`: Plugin key whose guest produced the rows (one of the job's).
  - `batch`: Batch number, from 1, for the logs.
  - `books`: Book rows as the guest's `ScanBook` (camelCase JSON objects);
    at most [`MAX_SCAN_BATCH_ROWS`](#max_scan_batch_rows).
- `ScanAccepted`: Owner → host: a results batch was written, or a
  [`Body::ScanRemoved`](#body) or [`Body::ScanPassEnd`](#body) was applied.
  - `batch`: The batch number (`0` for a pass end).
  - `upserted`: Rows written, or for a removals batch or a pass end the holdings
    removed.
- `ScanRemoved`: Host → owner: product ids a delta pass reported gone from one
  account, sent after the job's rows and before that account's
  [`Body::ScanPassEnd`](#body). The owner checks the fence, the node's trust
  and the account as for a row naming it, then removes those
  holdings. Answered with [`Body::ScanAccepted`](#body), [`Body::Lost`](#body), or
  `error {code: "refused"}`.
  - `job_id`: Job the pass belongs to.
  - `fence`: Lease generation the host holds.
  - `plugin_key`: The job's source plugin key.
  - `account_id`: The account the titles left.
  - `batch`: Batch number, from 1, for the logs.
  - `product_ids`: At most [`MAX_SCAN_REMOVED_IDS`](#max_scan_removed_ids) product ids.
- `ScanPassEnd`: Host → owner: how one account's pass ended, after all its rows and
  removals. The owner stores the sync token for the account's next
  pass and, after a complete full pass, removes the account's
  holdings this job's rows did not list. Answered with
  [`Body::ScanAccepted`](#body), [`Body::Lost`](#body), or `error {code: "refused"}`.
  - `job_id`: Job the pass belongs to.
  - `fence`: Lease generation the host holds.
  - `plugin_key`: The job's source plugin key.
  - `account_id`: The account the pass scanned.
  - `sync_token`: The plugin's token for the account's next pass, at most
    [`MAX_SCAN_SYNC_TOKEN_BYTES`](#max_scan_sync_token_bytes); empty when it keeps none.
  - `complete`: The pass listed every title the account holds.
  - `full`: The pass was full: it was sent no sync token, or restarted as a
    full pass after its token was refused.
- `AcquirePlan`: Host → owner: what an acquire job should fetch here. Answered with
  [`Body::AcquireTargets`](#body), [`Body::Lost`](#body), or `error`.
  - `job_id`: The acquire job.
  - `fence`: Lease generation the host holds.
  - `after`: Only titles whose holding id sorts after this one: the last
    target this run already tried, so titles that failed earlier in
    the run never fill the plan's cap. Absent for the first plan.
- `AcquireTargets`: Owner → host: the titles to acquire and how to package them.
  - `targets`: Titles still to acquire, each one of the job's accounts, in
    holding-id order and each after the plan's `after`.
  - `destinations`: Every output destination deployed on the answering shared node,
    the primary first, each with its own packaging and naming
    options, so the host names each destination's files as a local
    acquire would.
  - `next`: Present when the owner stopped before it looked at every title
    of the job (each plan looks at a bounded number, and titles
    that need no fetch add no target): ask again with this holding
    id as `after`, even when `targets` is empty. Absent, a plan
    with no new target means no title is left to try.
- `UploadGrant`: Host → owner: ask to upload one title's files. Answered with
  [`Body::UploadGranted`](#body), [`Body::Lost`](#body), or `error`.
  - `job_id`: The acquire job.
  - `fence`: Lease generation the host holds.
  - `holding_id`: The account holding of the title the files are for
    (`holdings.id` on the owner; one of the job's targets).
  - `primary_key`: Storage key of the title's primary audio file on the primary
    destination (one of `files`' copies there).
  - `pdf_key`: Storage key of the title's companion PDF on the primary
    destination (one of `files`' copies there), when one is
    uploaded.
  - `files`: Every file, in upload order. Empty, with an empty `holding_id`,
    for the job's closing upload, whose commit only completes the
    job.
- `UploadGranted`: Owner → host: the upload is open.
  - `upload_id`: Upload id; the files go to `UPLOADS_PATH<upload_id>/files/<n>`.
  - `sealed_token`: The bearer token for the upload endpoint, sealed to the node's
    box key and bound to the node, job, fence and upload
    (`crate::seal_upload_token`).
  - `expires_in_secs`: Seconds the token stays valid unless extended: the owner
    extends it while the job's fence is live (each renewal and each
    accepted file).
- `ScanRequest`: Host → owner: queue a library scan on the library for the
  node's owner (H-5). The owner's scan planner runs as that user: an
  Owner or Administrator scanning every account gets one job per
  scan-enabled account, routed across the library's nodes and hosts;
  anyone else scans only the accounts they own. Answered with
  [`Body::ScanPlanned`](#body) or `error {code: "forbidden" | "invalid" |
  "unavailable"}`.
  - `account`: Scan only this store account (even when not scan-enabled).
  - `plugin_instance_id`: Pin every planned job to this plugin instance.
- `ScanPlanned`: Owner → host: what a [`Body::ScanRequest`](#body) queued.
  - `created`: New jobs.
  - `duplicates`: Jobs that were already pending or running.
  - `queue_full`: Planned jobs the queue had no room for.
- `JobStatus`: Host → owner: the state of jobs this node may see (H-5). A job the
  node's owner could not have queued, or an unknown id, is left out.
  Answered with [`Body::JobStatuses`](#body).
  - `job_ids`: Job ids, at most [`MAX_STATUS_JOBS`](#max_status_jobs).
- `JobStatuses`: Owner → host: the jobs of a [`Body::JobStatus`](#body).
  - `jobs`: One entry per visible job, in request order.
- `Wake`: Owner → host: a message from the host's `node.<id>` queue. The host
  answers [`Body::Ack`](#body) with the same frame id once it has acted.
  - `message`: The queue message (`{"type": "deployment.changed", ...}`).
- `Ack`: Host → owner: the wake with this frame id was handled.
- `NodeLeaving`: Host → owner: this host is leaving the library (its operator chose
  Leave library). The owner revokes the node's credential and stops
  counting it as live at once, instead of waiting for missed
  heartbeats, answers [`Body::Accepted`](#body), and closes the connection.
- `CatalogBegin`: Branch → library: open (or reopen) this branch's catalog sync with
  the link's two settings. The library records them on the branch's
  node row and bumps its catalog fence, so frames under an older
  fence are lost. Answered with [`Body::CatalogSession`](#body) or `error`.
  - `share_books`: "Share this branch's books with *library*".
  - `include_branches`: "Include books from my own branches".
- `CatalogSession`: Library → branch: the answer to [`Body::CatalogBegin`](#body).
  - `fence`: The catalog fence every later `catalog.*` frame names.
  - `cursor`: Resume after this cursor (empty: from the start).
  - `ancestry`: The library's ancestry: its own library id, then each library
    above it, up to the root.
  - `destinations`: The library node's output destination ids, the primary first;
    a catalog upload copies each file to every one under its key.
  - `wanted`: Titles recorded earlier whose files have not arrived yet.
  - `members_books`: The library's "Acquire all of this branch's members' books":
    when true, the branch also sends the books its members acquired
    that it lists to this library, not only its owner's. Absent
    (older libraries) reads as false.
- `CatalogSync`: Branch → library: one batch of shared titles (no account or user
  fields; see [`crate::CatalogTitle`](#catalogtitle)). Answered with
  [`Body::CatalogAccepted`](#body), [`Body::Lost`](#body) (`item = "catalog"`), or
  `error {code: "refused"}`.
  - `fence`: The catalog fence.
  - `batch`: Batch number, from 1, for the logs.
  - `cursor`: The cursor after this batch's last title; the library resumes
    after it.
  - `done`: True for the last batch of a pass. When the library's pass
    from an empty cursor is under way (a resumed pass included),
    titles it did not name are withdrawn; otherwise it changes
    nothing more.
  - `titles`: The titles; at most [`crate::MAX_CATALOG_BATCH_TITLES`](#max_catalog_batch_titles).
- `CatalogAccepted`: Library → branch: a catalog batch was recorded.
  - `batch`: The batch number.
  - `upserted`: Titles recorded.
  - `skipped`: Titles not recorded because this library already holds them
    (no echo back).
  - `wanted`: Titles whose files the library wants.
- `CatalogUpload`: Branch → library: open an upload of one recorded title's files.
  Answered with [`Body::UploadGranted`](#body) (the token sealed for job
  [`crate::CATALOG_ITEM`](#catalog_item) under the catalog fence), [`Body::Lost`](#body), or
  `error`.
  - `fence`: The catalog fence.
  - `store`: Store id of the title.
  - `product_id`: Product id of the title.
  - `marketplace`: Marketplace of the title ([`crate::CatalogRef::marketplace`](#catalogref));
    empty, and left out of the frame, when the store has none.
  - `files`: The title's files, in upload order, each copied to every
    destination of the session under its manifest key.
- `AccountsAdd`: Branch → library: add the branch person's store connections to this
  library ("Add my connections to *library*"). `sealed` is an
  [`crate::AccountBundle`](#accountbundle) sealed to the connection's owner box key
  ([`Body::Welcome`](#body)) and bound to the library and node ids
  ([`crate::seal_account_offer`](#seal_account_offer)). The library keeps them for the
  user its node row names as the branch's owner, under that user's
  account rule. Answered with [`Body::AccountsAdded`](#body) or `error`.
  - `sealed`: The sealed bundle, base64url without padding.
- `AccountsAdded`: Library → branch: what became of an [`Body::AccountsAdd`](#body).
  - `added`: Accounts now connected here for the person.
  - `already_here`: Accounts the person had already connected here.
  - `taken`: Accounts connected to someone else here; left as they are.
  - `failed`: Accounts that could not be kept.
  - `not_allowed`: The library's operator does not let the person connect store
    accounts there; nothing was kept.
- `AccountsRemove`: Branch → library: the person disconnected this store account
  everywhere. The library unlinks it from the branch owner's
  identities when it came from this branch's person, and takes it
  back once no link holds it. Answered with [`Body::Accepted`](#body) or
  `error`.
  - `plugin_key`: Provenance-qualified plugin key of the storefront.
  - `account_id`: Store account id.
- `AccountsUpdate`: Branch → library: the newest sign-in of one single-keeper store
  account this branch keeps ("one refresh keeper" in the library
  catalog network ADR). `sealed` is an [`crate::AccountBundle`](#accountbundle) of
  exactly that account, sealed and bound like [`Body::AccountsAdd`](#body);
  `revision` is the keeper's revision of the sign-in (milliseconds
  since the Unix epoch of its newest secret write there, above 0).
  The library replaces its copy only when the account is its
  branch owner's, shared with this branch, and `revision` is above
  the last one it applied from it. Answered with
  [`Body::AccountsUpdated`](#body) or `error`.
  - `sealed`: The sealed one-account bundle, base64url without padding.
  - `revision`: The keeper's revision of the sign-in.
  - `keeps_own`: True when the branch was answered [`KeeperOutcome::Reconcile`](#keeperoutcome)
    for this account and keeps its own sign-in, which is newer than
    the one the library sent it: the library then applies the push
    rather than answering with its own sign-in again. Left out when
    false.
- `AccountsUpdated`: Library → branch: what became of an [`Body::AccountsUpdate`](#body).
  - `outcome`: What the library did with the sign-in.
  - `sealed`: With [`KeeperOutcome::Reconcile`](#keeperoutcome) only: the library's own
    sign-in of the account, a one-account [`crate::AccountBundle`](#accountbundle)
    sealed to the branch's box key
    ([`crate::seal_account_reconcile`](#seal_account_reconcile)), base64url without padding.
- `CatalogQuestions`: Branch → library: send the catalog questions waiting for this
  branch (see [`crate::LookupWork`](#lookupwork)). Answered with
  [`Body::CatalogAsk`](#body), or `error {code: "refused"}` when the node is
  not a branch this library trusts.
- `CatalogAsk`: Library → branch: the questions waiting for it, at most
  [`crate::MAX_LOOKUP_WORKS`](#max_lookup_works); empty when none is.
  - `works`: The questions.
- `CatalogFound`: Branch → library: answers to questions it was asked; a question
  with no answer means the branch cannot supply that work. Answered
  with [`Body::Accepted`](#body) or `error`.
  - `answers`: The answers, at most [`crate::MAX_LOOKUP_WORKS`](#max_lookup_works).
  - `missing`: Works of the batch the branch looked up and cannot supply, so
    the library stops asking until the answer expires.
- `RequestsClaim`: Branch → library: claim the interlibrary requests waiting for this
  branch (see [`crate::BranchRequest`](#branchrequest)). Answered with
  [`Body::RequestsClaimed`](#body), or `error {code: "refused"}` when the
  node is not a branch this library trusts.
- `RequestsClaimed`: Library → branch: the requests now leased to it, at most
  [`crate::MAX_CLAIMED_REQUESTS`](#max_claimed_requests); empty when none waits.
  - `requests`: The requests now leased to the asking branch, at most [`crate::MAX_CLAIMED_REQUESTS`](#max_claimed_requests).
- `RequestFinish`: Branch → library: what it did with one claimed request. Answered
  with [`Body::Accepted`](#body), [`Body::Lost`](#body) (item
  [`crate::ITEM_REQUEST`](#item_request)) when the lease is no longer the branch's,
  or `error`.
  - `job`: The request's job id.
  - `generation`: The lease generation it was claimed under.
  - `outcome`: What the branch did.
- `WindowBooks`: Window → library: send the next page of the books this library
  holds for the window's person (the node's owner). Answered with
  [`Body::WindowPage`](#body), or `error {code: "refused"}` when the node is
  not a branch this library trusts.
  - `after`: The `next` cursor of the previous page; empty (or absent) for
    the first page.
- `WindowPage`: Library → window: one page of the person's books.
  - `works`: The works, at most [`crate::MAX_WINDOW_WORKS`](#max_window_works).
  - `next`: The cursor to ask with for the next page; absent on the last.
- `CopyGrant`: Reader → library: ask for the copy this library holds of one
  edition, named by its store listing, for the node's owner
  (`copy_grant_problem`). Answered with [`Body::CopyGranted`](#body), or `error`
  with `refused` (not a branch this library trusts), `not_found` (the
  person cannot take the edition, or the library holds no copy of it),
  `invalid`, or `unavailable` (the library cannot read its storage
  right now).
  - `store`: Store id of the edition's listing.
  - `marketplace`: The listing's marketplace; empty (or absent) for a store
    without marketplaces.
  - `product_id`: The listing's product id.
- `CopyGranted`: Library → reader: the copy may be downloaded.
  - `copy_id`: Copy id; the files are at `COPIES_PATH<copy_id>/files/<n>`.
  - `sealed_token`: The bearer token for the download endpoint, sealed to the
    node's box key and bound to the node and the copy
    (`crate::seal_copy_token`).
  - `expires_in_secs`: Seconds the token stays valid.
  - `copy`: What the copy is, as the library's page states it.
  - `files`: The copy's files; a file's index here is its `<n>`.
- `Accepted`: Owner → host: the request with this frame id was applied.
- `Unknown`: A frame type this side does not know.

Implements: `Clone`, `Debug`, `Deserialize<'de>`, `Eq`, `PartialEq`, `Serialize`

### ClaimScope

```rust
pub enum ClaimScope {
    Own,
    AllMembers,
}
```

Whose work a host takes from a library (`claim.filter.scope`), as the
branch operator set it for that link.

The library still hands only work the node is trusted with: a scope
can narrow that trust, never widen it.

**Variants**

- `Own`: Only work for the store accounts of the user who owns the node.
- `AllMembers`: Work for any member of the library the node is trusted with
  ("Download for other members of *library*").

#### ClaimScope::is_own

```rust
pub fn is_own(&self) -> bool
```

True for [`ClaimScope::Own`](#claimscope), the default left off the wire.

Implements: `Clone`, `Copy`, `Debug`, `Default`, `Deserialize<'de>`, `Eq`, `PartialEq`, `Serialize`

### FrameError

```rust
pub enum FrameError {
    TooLarge,
    Malformed,
    Version(u32),
}
```

Why a frame was not accepted.

**Variants**

- `TooLarge`: More than [`MAX_FRAME_BYTES`](#max_frame_bytes).
- `Malformed`: Not a JSON frame of the expected shape.
- `Version`: `v` is not [`FRAME_VERSION`](#frame_version).

Implements: `Debug`, `Display`, `Eq`, `Error`, `PartialEq`

### KeeperOutcome

```rust
pub enum KeeperOutcome {
    Applied,
    Stale,
    NotShared,
    Reconcile,
}
```

What a library did with a keeper's [`Body::AccountsUpdate`](#body).

**Variants**

- `Applied`: The library's copy now holds the pushed sign-in.
- `Stale`: The library already applied this revision or a newer one from the
  branch; nothing changed.
- `NotShared`: The account is not the branch owner's here, or not shared with
  this branch; nothing changed. The keeper stops pushing it here.
- `Reconcile`: The branch's first push of the account, and the library refreshed
  its own sign-in after it sent the account to the branch: nothing
  changed, and the answer carries the library's sign-in. The branch
  takes it when its own copy is still the one the library sent, and
  otherwise pushes its own again with `keeps_own`.

Implements: `Clone`, `Copy`, `Debug`, `Deserialize<'de>`, `Eq`, `PartialEq`, `Serialize`

### KeyError

```rust
pub enum KeyError {
    Length,
    NotOnCurve,
    NonCanonical,
    WeakPoint,
}
```

Why a public key was refused.

**Variants**

- `Length`: Not 32 bytes.
- `NotOnCurve`: Not a point on the curve.
- `NonCanonical`: A non-canonical encoding of a point.
- `WeakPoint`: A small-order or mixed-order point.

Implements: `Debug`, `Display`, `Eq`, `Error`, `PartialEq`

### Availability

```rust
pub enum Availability {
    Owned,
    Reachable,
}
```

How a branch can supply an edition.

**Variants**

- `Owned`: The branch's own library holds it (its owner's or a member's book).
- `Reachable`: The branch can borrow it from another library it joined, which
  allows onward lending.

Implements: `Clone`, `Copy`, `Debug`, `Deserialize<'de>`, `Eq`, `PartialEq`, `Serialize`

### LookupFormat

```rust
pub enum LookupFormat {
    Audio,
    Pdf,
}
```

A file format a branch has for an edition.

**Variants**

- `Audio`: The audiobook itself.
- `Pdf`: The companion PDF.

Implements: `Clone`, `Copy`, `Debug`, `Deserialize<'de>`, `Eq`, `Ord`, `PartialEq`, `PartialOrd`, `Serialize`

### LookupScheme

```rust
pub enum LookupScheme {
    Asin,
    Isbn,
    OpenlibraryEdition,
    OpenlibraryWork,
}
```

Which kind of identifier a [`LookupIdentifier`](#lookupidentifier) is. Only identifiers
that name one edition or one work are asked about; anything else does
not parse.

**Variants**

- `Asin`: Amazon ASIN of an edition in one marketplace (strong).
- `Isbn`: ISBN-13 of an edition (weak: one ISBN can sit on several editions).
- `OpenlibraryEdition`: OpenLibrary edition id (strong).
- `OpenlibraryWork`: OpenLibrary work id (names the work).

#### LookupScheme::as_str

```rust
pub fn as_str(self) -> &'static str
```

The scheme as the library model stores it (`edition_identifiers.scheme`).

#### LookupScheme::parse

```rust
pub fn parse(text: &str) -> Option<Self>
```

Parses stored scheme text; `None` for any other.

#### LookupScheme::is_scoped

```rust
pub fn is_scoped(self) -> bool
```

True for the one scheme scoped to a marketplace (ASIN).

Implements: `Clone`, `Copy`, `Debug`, `Deserialize<'de>`, `Eq`, `Hash`, `Ord`, `PartialEq`, `PartialOrd`, `Serialize`

### RequestOutcome

```rust
pub enum RequestOutcome {
    Accepted,
    NotListed,
    Failed,
}
```

What a branch did with a claimed request.

**Variants**

- `Accepted`: The branch took it: the book is queued or already on the branch,
  and reaches the library through the branch's catalog sync.
- `NotListed`: The branch no longer lists the book to this library (nobody here
  holds it, the holder opted out, or sharing is off). Final: the
  library does not ask again.
- `Failed`: The branch could not take it now (a local read or write failed).
  The library retries it while the job has attempts left.

#### RequestOutcome::as_str

```rust
pub fn as_str(self) -> &'static str
```

The outcome as the library records it on the job.

Implements: `Clone`, `Copy`, `Debug`, `Deserialize<'de>`, `Eq`, `PartialEq`, `Serialize`

### SealError

```rust
pub enum SealError {
    Key,
    Degenerate,
    Open,
    Bundle,
}
```

Why a sealed box could not be made or opened.

**Variants**

- `Key`: The recipient key is not a valid box key.
- `Degenerate`: Key agreement produced the all-zero point.
- `Open`: The box is too short, not base64url, or does not authenticate.
- `Bundle`: The opened bundle is not a JSON list of secrets.

Implements: `Debug`, `Display`, `Eq`, `Error`, `PartialEq`

### ConnectionType

```rust
pub enum ConnectionType {
    Branch,
    SharedBranch,
}
```

How a joining install connects to a library.

**Variants**

- `Branch`: A branch node: its own database, work over the node socket (default).
- `SharedBranch`: A shared node: shares the library's database and DEK.

#### ConnectionType::as_str

```rust
pub fn as_str(self) -> &'static str
```

Stable text (`branch`, `shared_branch`).

#### ConnectionType::parse

```rust
pub fn parse(raw: &str) -> Option<Self>
```

Parses `branch` or `shared_branch` (case-insensitive); anything else is
`None`.

#### ConnectionType::is_branch

```rust
pub fn is_branch(&self) -> bool
```

True for [`Self::Branch`](#connectiontype) (serde: the field is left out).

#### ConnectionType::granted_scope

```rust
pub fn granted_scope(self) -> &'static str
```

The scope a code for this connection type is granted.

Implements: `Clone`, `ComposeSchema`, `Copy`, `Debug`, `Default`, `Deserialize<'de>`, `Eq`, `PartialEq`, `Serialize`, `ToSchema`

### SharedBranchDatabase

```rust
pub enum SharedBranchDatabase {
    Postgres { url: SecretValue },
    D1 { account_id: String, database_id: String, api_base: String },
}
```

The library's database, as a shared branch needs it to connect.

**Variants**

- `Postgres`: PostgreSQL: the connection URL (it carries credentials).
  - `url`: `postgres://user:password@host/db`.
- `D1`: Cloudflare D1: the database's ids. Its API token travels as the
  bootstrap's [`SharedBranchCredential`](#sharedbranchcredential).
  - `account_id`: Cloudflare account id.
  - `database_id`: D1 database UUID.
  - `api_base`: Cloudflare API base.

#### SharedBranchDatabase::plugin

```rust
pub fn plugin(&self) -> &'static str
```

The `[database].plugin` id (`postgres`, `d1`).

Implements: `Clone`, `Debug`, `Deserialize<'de>`, `Eq`, `PartialEq`, `Serialize`

### AuthError

```rust
pub enum AuthError {
    Key,
    Malformed,
    Mismatch,
}
```

Why a handshake signature was refused.

**Variants**

- `Key`: The stored public key is invalid.
- `Malformed`: The signature is not base64url of 64 bytes.
- `Mismatch`: The signature does not verify.

Implements: `Debug`, `Display`, `Eq`, `Error`, `PartialEq`

### TunnelExposure

```rust
pub enum TunnelExposure {
    Branches,
    Library,
}
```

What public traffic may reach.

**Variants**

- `Branches`: Only what branches and clerks need.
- `Library`: The whole library, web app included.

Implements: `Clone`, `Copy`, `Debug`, `Eq`, `PartialEq`

### TunnelRefusal

```rust
pub enum TunnelRefusal {
    OperatorOnly,
    NotExposed,
}
```

Why a tunnel request was refused.

**Variants**

- `OperatorOnly`: The path is for the operator on this machine only.
- `NotExposed`: The exposure the operator chose does not include the path.

#### TunnelRefusal::as_str

```rust
pub fn as_str(self) -> &'static str
```

Stable code for the response body and logs.

Implements: `Clone`, `Copy`, `Debug`, `Eq`, `PartialEq`

### UploadRange

```rust
pub enum UploadRange {
    Part(UploadPart),
    Status,
}
```

What a `Content-Range` on an upload `PUT` asks for.

**Variants**

- `Part`: The body is this part of the file.
- `Status`: No body: answer with the offset the owner needs next
  (`bytes */<size>`).

#### UploadRange::status_header

```rust
pub fn status_header(size: u64) -> String
```

The status query's `Content-Range` value for a file of `size`
bytes.

Implements: `Clone`, `Copy`, `Debug`, `Eq`, `PartialEq`

## Functions

### account_bundle_aad

```rust
pub fn account_bundle_aad(library_id: &str, node_id: &str) -> Vec<u8>
```

Associated data of the account bundle for `node_id` of `library_id`:
the domain tag, then each field as a 4-byte big-endian length and its
bytes.

### account_offer_aad

```rust
pub fn account_offer_aad(library_id: &str, node_id: &str) -> Vec<u8>
```

Associated data of an account offer from `node_id` to its library
`library_id`: [`ACCOUNT_OFFER_AAD_DOMAIN`](#account_offer_aad_domain), then the two ids as in
[`account_bundle_aad`](#account_bundle_aad).

### account_reconcile_aad

```rust
pub fn account_reconcile_aad(library_id: &str, node_id: &str) -> Vec<u8>
```

Associated data of a library's sign-in answering a keeper's first push
from `node_id`: [`ACCOUNT_RECONCILE_AAD_DOMAIN`](#account_reconcile_aad_domain), then the two ids as in
[`account_bundle_aad`](#account_bundle_aad).

### check_account_bundle

```rust
pub fn check_account_bundle(
    bundle: &AccountBundle,
    library_id: &str,
    node_id: &str,
) -> Result<(), SealError>
```

Checks `bundle` against every rule [`parse_account_bundle`](#parse_account_bundle) applies.

#### Errors

Returns [`SealError::Bundle`](#sealerror) for a wrong version or ids, too many
accounts or secrets, an empty or oversized field or value, a repeated
account, or a repeated secret within an account.

### open_account_bundle

```rust
pub fn open_account_bundle(
    recipient_secret: &[u8; 32],
    library_id: &str,
    node_id: &str,
    sealed: &str,
) -> Result<AccountBundle, SealError>
```

Opens a bundle made by [`seal_account_bundle`](#seal_account_bundle) with the joining node's
box secret, for the library and node ids the redeem answered.

#### Errors

Returns [`SealError::Open`](#sealerror) when the box was sealed to another key, for
another library or node, or was altered, and [`SealError::Bundle`](#sealerror) when
its content is refused by [`parse_account_bundle`](#parse_account_bundle).

### open_account_offer

```rust
pub fn open_account_offer(
    recipient_secret: &[u8; 32],
    library_id: &str,
    node_id: &str,
    sealed: &str,
) -> Result<AccountBundle, SealError>
```

Opens an offer made by [`seal_account_offer`](#seal_account_offer) with the library's owner
box secret, for the library id and the node id of the authenticated
connection it arrived on.

#### Errors

As [`open_account_bundle`](#open_account_bundle).

### open_account_reconcile

```rust
pub fn open_account_reconcile(
    recipient_secret: &[u8; 32],
    library_id: &str,
    node_id: &str,
    sealed: &str,
) -> Result<AccountBundle, SealError>
```

Opens a library's sign-in made by [`seal_account_reconcile`](#seal_account_reconcile) with the
branch's box secret, for the library it joined and its own node id.

#### Errors

As [`open_account_bundle`](#open_account_bundle).

### parse_account_bundle

```rust
pub fn parse_account_bundle(
    plaintext: &[u8],
    library_id: &str,
    node_id: &str,
) -> Result<AccountBundle, SealError>
```

Parses the opened bytes of an account bundle for `node_id` of
`library_id` and checks it ([`check_account_bundle`](#check_account_bundle)). Sans-IO: the
branch runs it on what [`open`](#open) returns, and the fuzz target runs it on
arbitrary bytes.

#### Errors

Returns [`SealError::Bundle`](#sealerror) for bytes over
[`MAX_ACCOUNT_BUNDLE_BYTES`](#max_account_bundle_bytes), bytes that are not a bundle, and a bundle
the check refuses.

### seal_account_bundle

```rust
pub fn seal_account_bundle(
    recipient: &[u8],
    ephemeral_secret: &[u8; 32],
    bundle: &AccountBundle,
) -> Result<String, SealError>
```

Seals `bundle` to the joining node's box key `recipient`, as base64url
text for the redeem answer.

#### Errors

Returns [`SealError::Key`](#sealerror) for an invalid recipient key and
[`SealError::Bundle`](#sealerror) for a bundle [`check_account_bundle`](#check_account_bundle) refuses for
its own ids, or whose JSON is over [`MAX_ACCOUNT_BUNDLE_BYTES`](#max_account_bundle_bytes).

### seal_account_offer

```rust
pub fn seal_account_offer(
    recipient: &[u8],
    ephemeral_secret: &[u8; 32],
    bundle: &AccountBundle,
) -> Result<String, SealError>
```

Seals a branch's `bundle` to its library's owner box key `recipient`
([`crate::Body::Welcome`](#body)), for [`crate::Body::AccountsAdd`](#body). The box is
bound with [`account_offer_aad`](#account_offer_aad), so a joining bundle never opens as an
offer, nor the other way round.

#### Errors

As [`seal_account_bundle`](#seal_account_bundle).

### seal_account_reconcile

```rust
pub fn seal_account_reconcile(
    recipient: &[u8],
    ephemeral_secret: &[u8; 32],
    bundle: &AccountBundle,
) -> Result<String, SealError>
```

Seals a library's own sign-in of one account (`bundle`) to the keeping
branch's box key `recipient`, for the `sealed` of an
[`crate::Body::AccountsUpdated`](#body) with
[`crate::KeeperOutcome::Reconcile`](#keeperoutcome). Bound with
[`account_reconcile_aad`](#account_reconcile_aad).

#### Errors

As [`seal_account_bundle`](#seal_account_bundle).

### is_ancestry

```rust
pub fn is_ancestry(ancestry: &[String]) -> bool
```

True when every id in `ancestry` is a plain library id and there are at
most [`MAX_ANCESTRY`](#max_ancestry).

### is_library_id

```rust
pub fn is_library_id(id: &str) -> bool
```

True for a plain library id in an ancestry.

### is_marketplace

```rust
pub fn is_marketplace(marketplace: &str) -> bool
```

True for a marketplace: empty, or up to [`MAX_MARKETPLACE_BYTES`](#max_marketplace_bytes)
lowercase ASCII letters and digits.

### is_product_id

```rust
pub fn is_product_id(product_id: &str) -> bool
```

True for a product id: 1 to 256 bytes, no whitespace at the ends and no
control characters.

### is_store_id

```rust
pub fn is_store_id(store: &str) -> bool
```

True for a store id: a short lowercase token.

### clerk_exchange_matches

```rust
pub fn clerk_exchange_matches(
    stored: &StoredClerkCode<'_>,
    request: &ClerkTokenRequest,
) -> bool
```

True when `request` redeems `stored`: the code belongs to
[`CLERK_CLIENT_ID`](#clerk_client_id), the redirect URI is the same, the method is S256
and the trimmed verifier hashes to the challenge (compared in constant
time).

#### Arguments

* `stored` - The approved code's record.
* `request` - The token request.

#### Returns

Whether the exchange may mint a library card.

### clerk_request_problem

```rust
pub fn clerk_request_problem(params: &ClerkAuthorizeParams) -> Option<&'static str>
```

Why a clerk authorization request is malformed, or `None` when the
library may show its consent page.

#### Arguments

* `params` - The parsed request ([`parse_clerk_query`](#parse_clerk_query)).

#### Returns

A short reason, or `None`.

### is_clerk_name

```rust
pub fn is_clerk_name(name: &str) -> bool
```

True for a clerk name: 1 to [`MAX_CLERK_NAME_BYTES`](#max_clerk_name_bytes) bytes, no control
characters.

### is_clerk_token

```rust
pub fn is_clerk_token(token: &str) -> bool
```

True for a session token: exactly [`CLERK_TOKEN_LEN`](#clerk_token_len) base64url
characters.

### is_s256_challenge

```rust
pub fn is_s256_challenge(raw: &str) -> bool
```

True for an S256 code challenge: exactly 43 base64url characters.

### parse_clerk_authorization

```rust
pub fn parse_clerk_authorization(value: &[u8]) -> Option<&str>
```

The session token in an `Authorization` header value, when the value is
exactly `Bookclerk-Clerk <token>`.

The scheme is matched case-insensitively (RFC 9110 section 11.1); one
space separates it from the token; the token must pass
[`is_clerk_token`](#is_clerk_token). Anything else, including an operator `Bearer`
token, is `None`.

#### Arguments

* `value` - The raw header value bytes.

#### Returns

The token, borrowed from `value`, or `None`.

### parse_clerk_query

```rust
pub fn parse_clerk_query(raw: &str) -> Option<ClerkAuthorizeParams>
```

Parses an authorize query string (the raw query `/oidc/authorize`
received), sans-IO so the fuzz target drives the same code.

#### Arguments

* `raw` - The query string, without the `?`.

#### Returns

The parameters, or `None` when a parameter repeats, a required one is
missing, or the shape is wrong. Unknown parameters are ignored.

### parse_clerk_token_answer

```rust
pub fn parse_clerk_token_answer(body: &[u8]) -> Option<ClerkTokenAnswer>
```

Reads the library's `POST /api/clerk/v1/token` answer on the clerk.

The answer comes from another machine, so it is checked before any of
it is kept: at most [`MAX_CLERK_ANSWER_BYTES`](#max_clerk_answer_bytes) of UTF-8 (all of it,
unknown keys included), one JSON object with no known key written twice, `token` a session token ([`is_clerk_token`](#is_clerk_token)),
`library_id` a library id ([`crate::is_library_id`](#is_library_id)), `user.id` a
positive integer, `user.display_name` (optional) at most
[`MAX_CLERK_DISPLAY_NAME_BYTES`](#max_clerk_display_name_bytes) bytes with no control characters, and
`idle_secs` from 1 second to a year. Keys it does not know are ignored,
so a newer library can add to the answer.

#### Arguments

* `body` - The answer body bytes.

#### Returns

The answer, or `None` when it breaks any rule.

### parse_clerk_token_request

```rust
pub fn parse_clerk_token_request(body: &[u8]) -> Option<ClerkTokenRequest>
```

Reads a `POST /api/clerk/v1/token` body.

#### Arguments

* `body` - The request body bytes, from another machine.

#### Returns

The request when the body is one JSON object with exactly `code`,
`code_verifier` and `redirect_uri`, each a string written once, or
`None`.

### is_topic_name

```rust
pub fn is_topic_name(name: &str) -> bool
```

True when `name` could be a topic name: 1 to [`MAX_CLERK_TOPIC_BYTES`](#max_clerk_topic_bytes)
bytes of lowercase ASCII letters, digits, `_` and `.`.

### parse_clerk_frame

```rust
pub fn parse_clerk_frame(text: &[u8]) -> Option<ClerkFrame>
```

Reads a frame a clerk sent: at most [`MAX_CLERK_FRAME_BYTES`](#max_clerk_frame_bytes), one JSON
object with `v` (1), a known `type`, and exactly that type's fields,
each once (a field written as `null` counts as absent).

- `books`: `after`, a string [`WindowCursor`](#windowcursor).
- `copy.grant`: `store`, `product_id` and, for a store with
  marketplaces, `marketplace`, passing [`copy_grant_problem`](#copy_grant_problem).

#### Arguments

* `text` - The frame's bytes.

#### Returns

The frame, or `None` when it breaks any rule.

### parse_library_frame

```rust
pub fn parse_library_frame(text: &[u8]) -> Option<LibraryFrame>
```

Reads a frame a library sent: at most [`crate::MAX_FRAME_BYTES`](#max_frame_bytes), one
JSON object with `v` (1), a known `type`, and exactly that type's fields,
each once.

- `hello` and `changed`: `topics`, at most [`MAX_CLERK_TOPICS`](#max_clerk_topics) names,
  each [`is_topic_name`](#is_topic_name); names this version does not know are dropped.
- `books.page`: `after` (a [`WindowCursor`](#windowcursor)), `works`, and `next` unless it is
  the last page, passing [`crate::window_page_problem`](#window_page_problem).
- `copy.granted`: `store`, `product_id`, `copy_id`, `token`,
  `expires_in_secs`, `copy`, `files` and, for a store with
  marketplaces, `marketplace`, passing [`ClerkCopy::problem`](#clerkcopyproblem).
- `error`: a [`ClerkSocketError`](#clerksocketerror) `code`.

#### Arguments

* `text` - The frame's bytes.

#### Returns

The frame, or `None` when it breaks any rule.

### copy_answer_head_end

```rust
pub fn copy_answer_head_end(received: &[u8]) -> Result<Option<usize>, &'static str>
```

Finds where the head of a download's answer ends in the bytes
received so far, however the library's writes were cut into reads.

The head ends before the first blank line (CRLF CRLF). A reader calls
this on everything it has received after each read: the head is the
first `n` bytes, [`parse_copy_answer`](#parse_copy_answer) reads it, and the body starts
four bytes after it. The answer depends only on the bytes, never on
where the reads fell: once it is `Some(n)` or an error it stays that
for every longer run of the same bytes.

#### Returns

`Ok(Some(n))` when the head is the first `n` bytes, `Ok(None)` when
the blank line has not arrived yet and still may.

#### Errors

Returns a short reason when the head would be longer than
[`MAX_COPY_ANSWER_HEAD_BYTES`](#max_copy_answer_head_bytes).

### copy_grant_problem

```rust
pub fn copy_grant_problem(
    store: &str,
    marketplace: &str,
    product_id: &str,
) -> Option<&'static str>
```

Why a [`crate::Body::CopyGrant`](#body) is malformed, or `None`: a store id, a
marketplace (empty when the store has none) and a product id.

### copy_granted_problem

```rust
pub fn copy_granted_problem(frame: &CopyGrantedFrame<'_>) -> Option<&'static str>
```

Why a [`crate::Body::CopyGranted`](#body) is malformed, or `None`: a copy id
([`is_copy_id`](#is_copy_id)), a non-empty sealed token of base64url text, a life of
1 to [`MAX_COPY_TOKEN_SECS`](#max_copy_token_secs) seconds, a well-formed copy, and 1 to
[`MAX_COPY_FILES`](#max_copy_files) well-formed files with distinct names, exactly one
of them the primary audio ([`CopyRole::Audio`](#copyrole)).

### is_copy_id

```rust
pub fn is_copy_id(id: &str) -> bool
```

True for a copy id: a lowercase hyphenated UUID.

### parse_copy_answer

```rust
pub fn parse_copy_answer(
    head: &[u8],
    asked_from: u64,
    size: u64,
) -> Result<CopyAnswer, &'static str>
```

Reads the head of the answer to a download of a file of `size` bytes
that asked for the bytes from `asked_from` on (0 for the whole file).

`head` is the status line and the headers, each line ended by CRLF or
the end of the head, without the blank line that ends it. A reader
writes the body where this says, so the head must state exactly the
bytes that were asked for:

- `200` with `Content-Length` equal to `size`: the whole file, also
  when a range was asked for (a server may ignore one);
- `206` with `Content-Range: bytes <asked_from>-<size - 1>/<size>` and
  the matching `Content-Length`, only when `asked_from` is above 0 and
  inside the file;
- any other status from 100 to 599: no file bytes. A `416` that
  states a `Content-Range` must state the unsatisfied form,
  `bytes */<size>`.

A header's name is an HTTP token (letters, digits and
``!#$%&'*+-.^_`|~``).

#### Errors

Returns a short reason when the head is longer than
[`MAX_COPY_ANSWER_HEAD_BYTES`](#max_copy_answer_head_bytes), is not printable ASCII lines, has no
`HTTP/1.x` status line, states a transfer encoding, repeats or
misstates `Content-Length` or `Content-Range`, or answers `200` or
`206` with other bytes than the ones asked for.

### parse_copy_range

```rust
pub fn parse_copy_range(header: &str, size: u64) -> CopyRange
```

Reads the `Range` header of a download of a file of `size` bytes.

One byte range is honoured: `bytes=A-B` (clamped to the file's end),
`bytes=A-` and the suffix form `bytes=-N` (the last `N` bytes). Each
number is 1 to 19 decimal digits. Anything else (another unit, several
ranges, spaces, signs, `A` past `B`) is ignored and reads the whole
file, as HTTP allows a server to do.

### deployments_page_problem

```rust
pub fn deployments_page_problem(
    after: Option<&str>,
    items: &[RemoteDeployment],
    next: Option<&str>,
) -> Option<&'static str>
```

Why a branch would refuse one `deployments` page answering
`deployments.list {after}`, or `None` when it takes it: every entry's
deployment id sorts byte-wise after `after` and after the entry before
it (so no deployment is listed twice), and `next`, when present, sorts
after `after` and is not before the last entry. A page that breaks this
would let a library keep a branch paging forever or list a deployment
twice, so the branch drops the whole listing and retries.

#### Arguments

* `after` - The cursor the branch asked with (`None` for the first
  page).
* `items` - The page's entries.
* `next` - The page's next cursor.

### is_destination_id

```rust
pub fn is_destination_id(id: &str) -> bool
```

True when `id` is a destination id: a short lowercase token.

### is_storage_key

```rust
pub fn is_storage_key(key: &str) -> bool
```

True when `key` is a relative storage key: no empty, `.` or `..`
segment, no backslash or control character, not absolute, at most
[`MAX_UPLOAD_KEY_BYTES`](#max_upload_key_bytes), and never a Windows path that leaves the
destination's root (`colon_is_safe`).

### scan_pass_end_problem

```rust
pub fn scan_pass_end_problem(account_id: &str, sync_token: &str) -> Option<&'static str>
```

Why a [`Body::ScanPassEnd`](#body) the owner received is malformed, or `None`
when it is one a host could send: a non-blank account id and a sync
token of at most [`MAX_SCAN_SYNC_TOKEN_BYTES`](#max_scan_sync_token_bytes) with no control
characters.

### scan_removed_problem

```rust
pub fn scan_removed_problem(
    account_id: &str,
    product_ids: &[String],
) -> Option<&'static str>
```

Why a [`Body::ScanRemoved`](#body) the owner received is malformed, or `None`
when it is one a host could send: a non-blank account id, at most
[`MAX_SCAN_REMOVED_IDS`](#max_scan_removed_ids) product ids, each non-blank, at most
[`MAX_SCAN_PRODUCT_ID_BYTES`](#max_scan_product_id_bytes) and free of control characters.

### target_destinations_problem

```rust
pub fn target_destinations_problem(
    target: &AcquireTarget,
    plan: &[PlanDestination],
) -> Option<&'static str>
```

Why `target`'s destination list ([`AcquireTarget::destinations`](#acquiretarget)) does
not fit the plan it came with, or `None`: every id it names is one of
`plan`'s destinations, named once, in the plan's order.

#### Arguments

* `target` - One target of an `acquire.targets` answer.
* `plan` - That answer's destinations, the primary first.

#### Returns

`None` when the list fits (or is absent), otherwise a short reason.

### is_branch_name

```rust
pub fn is_branch_name(name: &str) -> bool
```

True for a branch name: 1 to [`MAX_BRANCH_NAME_BYTES`](#max_branch_name_bytes) bytes, no control
characters.

### is_branch_redirect_uri

```rust
pub fn is_branch_redirect_uri(raw: &str) -> bool
```

True for a redirect URI [`BRANCH_CLIENT_ID`](#branch_client_id) may use: an absolute
`https` URL, or an `http` URL whose host is a loopback IP literal or
`localhost` (RFC 8252 section 7.3), with no user info and no fragment.

### is_library_name

```rust
pub fn is_library_name(name: &str) -> bool
```

True for a library name: 1 to [`MAX_LIBRARY_NAME_BYTES`](#max_library_name_bytes) bytes, no
control characters.

### pkce_challenge

```rust
pub fn pkce_challenge(verifier: &str) -> String
```

The RFC 7636 S256 challenge of `verifier`: base64url (no padding) of its
SHA-256.

### box_public_key

```rust
pub fn box_public_key(secret: &[u8; 32]) -> [u8; 32]
```

The X25519 public key of a 32-byte secret (clamped per RFC 7748).

### sign_public_key

```rust
pub fn sign_public_key(seed: &[u8; 32]) -> [u8; 32]
```

The Ed25519 public key of a 32-byte signing seed.

### validate_box_public_key

```rust
pub fn validate_box_public_key(raw: &[u8]) -> Result<(), KeyError>
```

Validates an X25519 public box key (a Montgomery u-coordinate).

#### Errors

Returns [`KeyError`](#keyerror) when the key is the wrong length, on the twist,
not canonical, or not in the prime-order subgroup.

### validate_sign_public_key

```rust
pub fn validate_sign_public_key(raw: &[u8]) -> Result<VerifyingKey, KeyError>
```

Validates an Ed25519 public signing key.

#### Errors

Returns [`KeyError`](#keyerror) when the key is the wrong length, not a canonical
curve point, or not in the prime-order subgroup.

### found_problem

```rust
pub fn found_problem(answers: &[LookupAnswer], missing: &[String]) -> Option<&'static str>
```

Why a [`crate::Body::CatalogFound`](#body) is malformed, or `None`: at most
[`MAX_LOOKUP_WORKS`](#max_lookup_works) works named in all, each once, every answer
well-formed and every missing work a correlation key.

### is_work_ref

```rust
pub fn is_work_ref(work: &str) -> bool
```

True for a correlation key: 1 to [`MAX_WORK_REF_BYTES`](#max_work_ref_bytes) ASCII letters,
digits and `-`.

### questions_problem

```rust
pub fn questions_problem(works: &[LookupWork]) -> Option<&'static str>
```

Why a list of questions is malformed, or `None`: at most
[`MAX_LOOKUP_WORKS`](#max_lookup_works) well-formed questions, each about a different work.

### claimed_problem

```rust
pub fn claimed_problem(requests: &[BranchRequest]) -> Option<&'static str>
```

Why a list of claimed requests is malformed, or `None`: at most
[`MAX_CLAIMED_REQUESTS`](#max_claimed_requests) well-formed requests, each for a different job.

### finish_problem

```rust
pub fn finish_problem(job: &str, generation: i64) -> Option<&'static str>
```

The problem with a `requests.finish {job, generation}` a library
received, or `None` when both are ones it could have handed out.

### is_request_job

```rust
pub fn is_request_job(job: &str) -> bool
```

True for a request's job id: 1 to [`MAX_REQUEST_JOB_BYTES`](#max_request_job_bytes) ASCII
letters, digits, `-` and `_`.

### copy_token_aad

```rust
pub fn copy_token_aad(node_id: &str, copy_id: &str) -> Vec<u8>
```

Associated data of the token for copy `copy_id`, granted to `node_id`.

### instance_secrets_aad

```rust
pub fn instance_secrets_aad(node_id: &str, instance_id: &str, revision: i64) -> Vec<u8>
```

Associated data of `instance_id`'s `SECRETS` handed to `node_id` at
document `revision`.

### open

```rust
pub fn open(
    recipient_secret: &[u8; 32],
    aad: &[u8],
    sealed: &[u8],
) -> Result<Zeroizing<Vec<u8>>, SealError>
```

Opens a box sealed to the public key of `recipient_secret`.

#### Errors

Returns [`SealError::Open`](#sealerror) when the box is short, was sealed to another
key or other associated data, or was altered.

### open_copy_token

```rust
pub fn open_copy_token(
    recipient_secret: &[u8; 32],
    node_id: &str,
    copy_id: &str,
    sealed: &str,
) -> Result<SecretValue, SealError>
```

Opens a token made by [`seal_copy_token`](#seal_copy_token) with the node's box secret.

#### Errors

Returns [`SealError::Open`](#sealerror) when it was not sealed to this node for
this copy.

### open_instance_secrets

```rust
pub fn open_instance_secrets(
    recipient_secret: &[u8; 32],
    node_id: &str,
    instance_id: &str,
    revision: i64,
    sealed: &str,
) -> Result<SecretValue, SealError>
```

Opens a payload made by [`seal_instance_secrets`](#seal_instance_secrets) with the node's box
secret.

#### Errors

Returns [`SealError::Open`](#sealerror) when it was not sealed to this node for this
instance and revision.

### open_secret_update

```rust
pub fn open_secret_update(
    owner_box_secret: &[u8; 32],
    node_id: &str,
    job_id: &str,
    fence: i64,
    secret_ref: &SecretRef,
    sealed: &str,
) -> Result<SecretValue, SealError>
```

Opens a write-back made by [`seal_secret_update`](#seal_secret_update) with the owner's
per-connection box secret.

#### Errors

Returns [`SealError::Open`](#sealerror) when it was not sealed to this key for this
node, job, fence and ref.

### open_secrets

```rust
pub fn open_secrets(
    recipient_secret: &[u8; 32],
    node_id: &str,
    job_id: &str,
    fence: i64,
    sealed: &str,
) -> Result<Vec<SecretEntry>, SealError>
```

Opens a bundle made by [`seal_secrets`](#seal_secrets) with the node's box secret.

#### Errors

Returns [`SealError::Open`](#sealerror) when the box does not open for this node,
job and fence, and [`SealError::Bundle`](#sealerror) when its content is malformed.

### open_upload_token

```rust
pub fn open_upload_token(
    recipient_secret: &[u8; 32],
    node_id: &str,
    job_id: &str,
    fence: i64,
    upload_id: &str,
    sealed: &str,
) -> Result<SecretValue, SealError>
```

Opens a token made by [`seal_upload_token`](#seal_upload_token) with the node's box secret.

#### Errors

Returns [`SealError::Open`](#sealerror) when it was not sealed to this node for this
job, fence and upload.

### seal

```rust
pub fn seal(
    recipient: &[u8],
    ephemeral_secret: &[u8; 32],
    aad: &[u8],
    plaintext: &[u8],
) -> Result<Vec<u8>, SealError>
```

Seals `plaintext` to `recipient` (an X25519 public key) with the
ephemeral secret `ephemeral_secret`, binding `aad`.

`ephemeral_secret` must be fresh random bytes for every box; reusing it
for two plaintexts to one recipient reuses the key and nonce.

#### Errors

Returns [`SealError::Key`](#sealerror) for an invalid recipient key and
[`SealError::Degenerate`](#sealerror) when key agreement fails.

### seal_copy_token

```rust
pub fn seal_copy_token(
    recipient: &[u8],
    ephemeral_secret: &[u8; 32],
    node_id: &str,
    copy_id: &str,
    token: &[u8],
) -> Result<String, SealError>
```

Seals a copy token for `node_id`'s box key `recipient`, bound to the
node and the copy, as base64url text for a `copy.granted` frame. The
token is a bearer credential for the download endpoint, so it never
travels as frame text.

#### Errors

Returns [`SealError`](#sealerror) when the recipient key is invalid.

### seal_instance_secrets

```rust
pub fn seal_instance_secrets(
    recipient: &[u8],
    ephemeral_secret: &[u8; 32],
    node_id: &str,
    instance_id: &str,
    revision: i64,
    secrets_json: &[u8],
) -> Result<String, SealError>
```

Seals an instance's `SECRETS` payload (JSON bytes) for `node_id`'s box
key `recipient`, bound to the instance and document revision, as
base64url text for an `instance_config` frame.

#### Errors

Returns [`SealError`](#sealerror) when the recipient key is invalid.

### seal_secret_update

```rust
pub fn seal_secret_update(
    owner_box_key: &[u8],
    ephemeral_secret: &[u8; 32],
    node_id: &str,
    job_id: &str,
    fence: i64,
    secret_ref: &SecretRef,
    value: &[u8],
) -> Result<String, SealError>
```

Seals a refreshed token to the owner's per-connection box key
`owner_box_key` (from `welcome`), bound to the node, job, fence and
ref, as base64url text for a `secret.update` frame.

#### Errors

Returns [`SealError`](#sealerror) when the owner key is invalid.

### seal_secrets

```rust
pub fn seal_secrets(
    recipient: &[u8],
    ephemeral_secret: &[u8; 32],
    node_id: &str,
    job_id: &str,
    fence: i64,
    entries: &[SecretEntry],
) -> Result<String, SealError>
```

Seals `entries` for `node_id`'s box key `recipient`, bound to `job_id`
and `fence`, as base64url text for a `work` frame.

#### Errors

Returns [`SealError`](#sealerror) when the recipient key is invalid.

### seal_upload_token

```rust
pub fn seal_upload_token(
    recipient: &[u8],
    ephemeral_secret: &[u8; 32],
    node_id: &str,
    job_id: &str,
    fence: i64,
    upload_id: &str,
    token: &[u8],
) -> Result<String, SealError>
```

Seals an upload token for `node_id`'s box key `recipient`, bound to the
job, fence and upload, as base64url text for an `upload.granted` frame.
The token is a bearer credential for the upload endpoint, so it never
travels as frame text.

#### Errors

Returns [`SealError`](#sealerror) when the recipient key is invalid.

### secret_update_aad

```rust
pub fn secret_update_aad(
    node_id: &str,
    job_id: &str,
    fence: i64,
    secret_ref: &SecretRef,
) -> Vec<u8>
```

Associated data of a token write-back from `node_id` for `secret_ref`,
handed with `job_id` under `fence`.

### secrets_aad

```rust
pub fn secrets_aad(node_id: &str, job_id: &str, fence: i64) -> Vec<u8>
```

Associated data of the secret bundle handed to `node_id` for `job_id`
under lease generation `fence`: the domain tag, then each field as a
4-byte big-endian length and its bytes.

### upload_token_aad

```rust
pub fn upload_token_aad(
    node_id: &str,
    job_id: &str,
    fence: i64,
    upload_id: &str,
) -> Vec<u8>
```

Associated data of the token for upload `upload_id`, granted to
`node_id` for `job_id` under lease generation `fence`.

### open_shared_branch_bootstrap

```rust
pub fn open_shared_branch_bootstrap(
    recipient_secret: &[u8; 32],
    library_id: &str,
    node_id: &str,
    sealed: &str,
) -> Result<SharedBranchBootstrap, SealError>
```

Opens a bootstrap made by [`seal_shared_branch_bootstrap`](#seal_shared_branch_bootstrap) with the joining
node's box secret, for the library and node ids the redeem answered.

#### Errors

Returns [`SealError::Open`](#sealerror) when the box was sealed to another key, for
another library or node, or was altered, and [`SealError::Bundle`](#sealerror) when
its content is malformed or names other ids.

### parse_shared_branch_bootstrap

```rust
pub fn parse_shared_branch_bootstrap(
    plaintext: &[u8],
    library_id: &str,
    node_id: &str,
) -> Result<SharedBranchBootstrap, SealError>
```

Reads an opened bootstrap for `library_id` and `node_id`.

#### Arguments

* `plaintext` - the bootstrap JSON, as [`open_shared_branch_bootstrap`](#open_shared_branch_bootstrap)
  opened it.
* `library_id`, `node_id` - the ids the redeem answered.

#### Errors

[`SealError::Bundle`](#sealerror) for bytes over [`MAX_SHARED_BRANCH_BOOTSTRAP_BYTES`](#max_shared_branch_bootstrap_bytes),
bytes that are not a bootstrap, another version, a DEK that is not 32
bytes, other ids, a Postgres URL or credential secret that is empty,
longer than [`MAX_BOOTSTRAP_CREDENTIAL_BYTES`](#max_bootstrap_credential_bytes) or holds anything but
UTF-8 without whitespace or control characters, a credential id longer than
[`MAX_BOOTSTRAP_CREDENTIAL_ID_BYTES`](#max_bootstrap_credential_id_bytes) or holding anything but ASCII
letters, digits, `-` and `_`, or a credential sent with a Postgres
database.

### seal_shared_branch_bootstrap

```rust
pub fn seal_shared_branch_bootstrap(
    recipient: &[u8],
    ephemeral_secret: &[u8; 32],
    bootstrap: &SharedBranchBootstrap,
) -> Result<String, SealError>
```

Seals `bootstrap` to the joining node's box key `recipient`, as
base64url text for the redeem answer.

#### Errors

Returns [`SealError::Key`](#sealerror) for an invalid recipient key and
[`SealError::Bundle`](#sealerror) for a bootstrap that is not version 1 or whose DEK
is not 32 bytes.

### shared_branch_bootstrap_aad

```rust
pub fn shared_branch_bootstrap_aad(library_id: &str, node_id: &str) -> Vec<u8>
```

Associated data of a shared branch bootstrap for `node_id` of `library_id`: the
domain tag, then each field as a 4-byte big-endian length and its bytes.

### artifact_transcript

```rust
pub fn artifact_transcript(request: &ArtifactRequest<'_>) -> Vec<u8>
```

The bytes a host signs for an artifact request: the domain tag, then
each field as a 4-byte big-endian length and its bytes.

### auth_transcript

```rust
pub fn auth_transcript(ctx: &AuthContext<'_>) -> Vec<u8>
```

The bytes a host signs: the domain tag, then each field as a 4-byte
big-endian length and its bytes.

### canonical_origin

```rust
pub fn canonical_origin(raw: &str) -> Option<String>
```

The ASCII origin (`scheme://host[:port]`) of an http(s) URL, with the
default port and any path dropped and the host lowercased.

### decode_nonce

```rust
pub fn decode_nonce(raw: &str) -> Option<[u8; 32]>
```

Decodes a challenge nonce.

### encode_nonce

```rust
pub fn encode_nonce(nonce: &[u8; 32]) -> String
```

Encodes a nonce as base64url without padding.

### parse_artifact_authorization

```rust
pub fn parse_artifact_authorization(header: &str) -> Option<(&str, i64, &str)>
```

Splits an artifact `Authorization` header into node id, issue time and
signature.

### sign_artifact_request

```rust
pub fn sign_artifact_request(key: &SigningKey, request: &ArtifactRequest<'_>) -> String
```

The `Authorization` header value of a signed artifact request.

### sign_auth

```rust
pub fn sign_auth(key: &SigningKey, ctx: &AuthContext<'_>) -> String
```

Signs a handshake; the result is base64url without padding.

### verify_artifact_request

```rust
pub fn verify_artifact_request(
    public_key: &[u8],
    request: &ArtifactRequest<'_>,
    sig: &str,
    now: i64,
) -> Result<(), AuthError>
```

Verifies a signed artifact request with the node's enrolled key and
checks its time against `now` (unix seconds).

#### Errors

Returns [`AuthError`](#autherror) when the key is invalid, the signature is
malformed or does not verify, or the time is out of range
([`AuthError::Mismatch`](#autherror)).

### verify_auth

```rust
pub fn verify_auth(
    public_key: &[u8],
    ctx: &AuthContext<'_>,
    sig: &str,
) -> Result<(), AuthError>
```

Verifies a handshake signature with the node's enrolled public key.
Uses strict verification (no malleable or small-order signatures).

#### Errors

Returns [`AuthError`](#autherror) when the key is invalid, the signature is
malformed, or it does not verify.

### normalize_tunnel_path

```rust
pub fn normalize_tunnel_path(raw: &str) -> String
```

The path the router matches for `raw`: the router trims trailing slashes
and keeps exactly one leading slash, so `//api/node/` routes as
`/api/node`.

#### Arguments

* `raw` - The request target's path, without the query.

#### Returns

`raw` without leading or trailing slashes, behind one leading slash.

### tunnel_refusal

```rust
pub fn tunnel_refusal(path: &str, exposure: TunnelExposure) -> Option<TunnelRefusal>
```

Whether a tunnel request for `path` is refused.

#### Arguments

* `path` - The path from [`normalize_tunnel_path`](#normalize_tunnel_path).
* `exposure` - What the operator chose to expose.

#### Returns

`None` to serve the request, or why it is refused.

### parse_upload_range

```rust
pub fn parse_upload_range(header: &[u8], size: u64) -> Option<UploadRange>
```

Reads a `Content-Range` header value on an upload `PUT` of a declared
file of `size` bytes.

#### Arguments

* `header` - the header value as sent.
* `size` - the file's declared size from the upload's manifest.

#### Returns

[`UploadRange::Part`](#uploadrange) for exactly `bytes <first>-<last>/<size>`, with
canonical decimal numbers, `first <= last < size` and the declared size;
[`UploadRange::Status`](#uploadrange) for exactly `bytes */<size>` with the declared
size; `None` for anything else, including a part of an empty file
(which is always sent whole).

### upload_parts

```rust
pub fn upload_parts(size: u64, part_bytes: u64) -> Vec<UploadPart>
```

The parts a host sends a file of `size` bytes in, each at most
`part_bytes` (at least 1) long, in order; empty when the file fits in
one request (`size <= part_bytes`), which is then sent whole.

### window_page_problem

```rust
pub fn window_page_problem(
    after: &WindowCursor,
    works: &[WindowWork],
    next: Option<&WindowCursor>,
) -> Option<&'static str>
```

Why a [`crate::Body::WindowPage`](#body) is malformed, or `None`: at most
[`MAX_WINDOW_WORKS`](#max_window_works) well-formed works with distinct grouping keys, and a
next cursor, when there is one, past `after` (the cursor the window
asked with), so paging always moves forward.

## Constants

### ACCOUNT_BUNDLE_AAD_DOMAIN

```rust
pub const ACCOUNT_BUNDLE_AAD_DOMAIN: &[u8] = b"bookclerk-node.v1 account-bundle";
```

Domain tag that starts the associated data of an account bundle.

### ACCOUNT_BUNDLE_VERSION

```rust
pub const ACCOUNT_BUNDLE_VERSION: u32 = 1;
```

Version of an [`AccountBundle`](#accountbundle).

### ACCOUNT_OFFER_AAD_DOMAIN

```rust
pub const ACCOUNT_OFFER_AAD_DOMAIN: &[u8] = b"bookclerk-node.v1 account-offer";
```

Domain tag that starts the associated data of an account offer: a
branch's [`AccountBundle`](#accountbundle) sent up to its library
([`crate::Body::AccountsAdd`](#body)). A distinct tag keeps a bundle sealed in
one direction from opening as the other.

### ACCOUNT_RECONCILE_AAD_DOMAIN

```rust
pub const ACCOUNT_RECONCILE_AAD_DOMAIN: &[u8] = b"bookclerk-node.v1 account-reconcile";
```

Domain of the associated data a library's own sign-in is sealed with
when it answers a keeper's first push with
[`crate::KeeperOutcome::Reconcile`](#keeperoutcome) ([`seal_account_reconcile`](#seal_account_reconcile)), so it
never opens as a joining bundle or an offer.

### MAX_ACCOUNT_BUNDLE_BYTES

```rust
pub const MAX_ACCOUNT_BUNDLE_BYTES: usize = _;
```

Largest opened bundle accepted, in bytes. A bundle at every limit above
would be far larger; this keeps the redeem answer small.

### MAX_ACCOUNT_SECRETS

```rust
pub const MAX_ACCOUNT_SECRETS: usize = 16;
```

Most secrets one carried account holds.

### MAX_BUNDLE_ACCOUNTS

```rust
pub const MAX_BUNDLE_ACCOUNTS: usize = 64;
```

Most accounts one bundle carries.

### MAX_CARRIED_ACCOUNT_ID_BYTES

```rust
pub const MAX_CARRIED_ACCOUNT_ID_BYTES: usize = 256;
```

Longest store account id carried, in bytes.

### MAX_CARRIED_FIELD_BYTES

```rust
pub const MAX_CARRIED_FIELD_BYTES: usize = 128;
```

Longest source alias, marketplace, secret kind or label carried, in
bytes.

### MAX_CARRIED_PLUGIN_KEY_BYTES

```rust
pub const MAX_CARRIED_PLUGIN_KEY_BYTES: usize = 256;
```

Longest plugin key carried, in bytes.

### MAX_CARRIED_SECRET_BYTES

```rust
pub const MAX_CARRIED_SECRET_BYTES: usize = _;
```

Largest secret value carried, in bytes.

### CATALOG_ITEM

```rust
pub const CATALOG_ITEM: &str = "catalog";
```

Item kind of the catalog fence (`lost.item`), and the job id a catalog
upload's token is sealed for.

### MAX_ANCESTRY

```rust
pub const MAX_ANCESTRY: usize = 64;
```

Most library ids an ancestry may list.

### MAX_CATALOG_BATCH_TITLES

```rust
pub const MAX_CATALOG_BATCH_TITLES: usize = 100;
```

Most titles one [`crate::Body::CatalogSync`](#body) batch may carry.

### MAX_CATALOG_CURSOR_BYTES

```rust
pub const MAX_CATALOG_CURSOR_BYTES: usize = 512;
```

Longest cursor.

### MAX_CATALOG_FILE_VERSION_BYTES

```rust
pub const MAX_CATALOG_FILE_VERSION_BYTES: usize = 128;
```

Longest [`CatalogFile::version`](#catalogfile) a title may carry, in bytes.

### MAX_CATALOG_TITLE_BYTES

```rust
pub const MAX_CATALOG_TITLE_BYTES: usize = _;
```

Largest serialized title in a batch.

### MAX_CATALOG_WANTED

```rust
pub const MAX_CATALOG_WANTED: usize = 200;
```

Most titles one answer may say are wanted.

### MAX_MARKETPLACE_BYTES

```rust
pub const MAX_MARKETPLACE_BYTES: usize = 16;
```

Longest marketplace.

### CLERK_AUTH_SCHEME

```rust
pub const CLERK_AUTH_SCHEME: &str = "Bookclerk-Clerk";
```

The `Authorization` scheme a clerk sends its session token under.

### CLERK_CALLBACK_PATH

```rust
pub const CLERK_CALLBACK_PATH: &str = "/clerk/callback";
```

The clerk's own web app page a clerk sign-in's redirect comes back to
(`<clerk origin>/clerk/callback?code=…&state=…`).

### CLERK_CLIENT_ID

```rust
pub const CLERK_CLIENT_ID: &str = "bookclerk-clerk";
```

The built-in public OAuth client a clerk signs in through. It
is never stored in the library's clients table.

### CLERK_CODE_TTL_SECS

```rust
pub const CLERK_CODE_TTL_SECS: i64 = 60;
```

Seconds an authorization code for [`CLERK_CLIENT_ID`](#clerk_client_id) lives.

### CLERK_IDLE_DAYS

```rust
pub const CLERK_IDLE_DAYS: i64 = 90;
```

Days a clerk session stays signed in without being used. Each use
moves the expiry forward again.

### CLERK_SCOPE

```rust
pub const CLERK_SCOPE: &str = "library:member";
```

The only scope [`CLERK_CLIENT_ID`](#clerk_client_id) may ask for.

### CLERK_TOKEN_LEN

```rust
pub const CLERK_TOKEN_LEN: usize = 43;
```

Length of a session token: base64url (no padding) of 32 random bytes.

### CLERK_TOKEN_PATH

```rust
pub const CLERK_TOKEN_PATH: &str = "/api/clerk/v1/token";
```

`POST`: redeem an approved clerk sign-in for a session token.

### MAX_CLERK_ANSWER_BYTES

```rust
pub const MAX_CLERK_ANSWER_BYTES: usize = _;
```

Largest `POST /api/clerk/v1/token` answer a clerk reads.

### MAX_CLERK_DISPLAY_NAME_BYTES

```rust
pub const MAX_CLERK_DISPLAY_NAME_BYTES: usize = 128;
```

Longest display name a clerk keeps from a token answer.

### MAX_CLERK_NAME_BYTES

```rust
pub const MAX_CLERK_NAME_BYTES: usize = 64;
```

Longest clerk name sent.

### MAX_CLERK_STATE_BYTES

```rust
pub const MAX_CLERK_STATE_BYTES: usize = 512;
```

Longest `state` an authorize request may carry.

### CLERK_CONNECT_PATH

```rust
pub const CLERK_CONNECT_PATH: &str = "/api/clerk/v1/connect";
```

Path of the clerk socket.

### CLERK_SOCKET_VERSION

```rust
pub const CLERK_SOCKET_VERSION: u8 = 1;
```

The `v` every frame carries.

### CLERK_SUBPROTOCOL

```rust
pub const CLERK_SUBPROTOCOL: &str = "bookclerk-clerk.v1";
```

The WebSocket subprotocol a clerk must offer.

### MAX_CLERK_FRAME_BYTES

```rust
pub const MAX_CLERK_FRAME_BYTES: usize = _;
```

Largest frame a library accepts from a clerk, in bytes; a `books` frame
is under 120.

### MAX_CLERK_TOPICS

```rust
pub const MAX_CLERK_TOPICS: usize = 16;
```

Most topic names one `hello` or `changed` frame may carry, known or not.

### MAX_CLERK_TOPIC_BYTES

```rust
pub const MAX_CLERK_TOPIC_BYTES: usize = 32;
```

Longest topic name, in bytes.

### COPIES_PATH

```rust
pub const COPIES_PATH: &str = "/api/copies/v1/";
```

Path prefix of the download endpoint: the files of copy `<copy_id>`
are at `COPIES_PATH<copy_id>/files/<n>`.

### COPY_AUTH_SCHEME

```rust
pub const COPY_AUTH_SCHEME: &str = "Bookclerk-Copy";
```

`Authorization` scheme of the download endpoint.

### MAX_COPY_ANSWER_HEAD_BYTES

```rust
pub const MAX_COPY_ANSWER_HEAD_BYTES: usize = _;
```

Longest head of a download's answer a reader takes, in bytes: the
status line and headers, before the blank line.

### MAX_COPY_FILES

```rust
pub const MAX_COPY_FILES: usize = 1000;
```

Most files one [`crate::Body::CopyGranted`](#body) may list.

### MAX_COPY_FILE_BYTES

```rust
pub const MAX_COPY_FILE_BYTES: u64 = _;
```

Largest [`CopyFile::size`](#copyfile) (1 TiB): above any audiobook file.

### MAX_COPY_NAME_BYTES

```rust
pub const MAX_COPY_NAME_BYTES: usize = 512;
```

Longest [`CopyFile::name`](#copyfile), in bytes.

### MAX_COPY_TOKEN_SECS

```rust
pub const MAX_COPY_TOKEN_SECS: u64 = _;
```

Longest a copy token may be stated to live, in seconds (one day).

### ITEM_JOB

```rust
pub const ITEM_JOB: &str = "job";
```

Work item kind for jobs.

### MAX_CALL_NUMBER_BYTES

```rust
pub const MAX_CALL_NUMBER_BYTES: usize = 16;
```

Longest call number (see [`CallNumber`](#callnumber)).

### MAX_DESTINATION_ID_BYTES

```rust
pub const MAX_DESTINATION_ID_BYTES: usize = 64;
```

Longest destination id.

### MAX_PROBE_BITRATE_KBPS

```rust
pub const MAX_PROBE_BITRATE_KBPS: u32 = 20_000;
```

Highest bitrate an [`UploadProbe`](#uploadprobe) may state, in kilobits per second:
above any audiobook encoding, so a forged value cannot win by size alone.

### MAX_SCAN_BATCH_ROWS

```rust
pub const MAX_SCAN_BATCH_ROWS: usize = 200;
```

Most rows one [`Body::ScanResults`](#body) batch may carry.

### MAX_SCAN_PRODUCT_ID_BYTES

```rust
pub const MAX_SCAN_PRODUCT_ID_BYTES: usize = 512;
```

Longest product id a [`Body::ScanRemoved`](#body) batch may carry.

### MAX_SCAN_REMOVED_IDS

```rust
pub const MAX_SCAN_REMOVED_IDS: usize = 500;
```

Most product ids one [`Body::ScanRemoved`](#body) batch may carry (a scan
page's own cap).

### MAX_SCAN_ROW_BYTES

```rust
pub const MAX_SCAN_ROW_BYTES: usize = _;
```

Largest serialized row in a [`Body::ScanResults`](#body) batch.

### MAX_SCAN_SYNC_TOKEN_BYTES

```rust
pub const MAX_SCAN_SYNC_TOKEN_BYTES: usize = 4096;
```

Longest sync token a [`Body::ScanPassEnd`](#body) or [`ScanToken`](#scantoken) may carry
(a scan page's own cap).

### MAX_SCAN_TOKENS

```rust
pub const MAX_SCAN_TOKENS: usize = 256;
```

Most [`ScanToken`](#scantoken)s one [`WorkJob`](#workjob) may carry.

### MAX_STATUS_JOBS

```rust
pub const MAX_STATUS_JOBS: usize = 64;
```

Most job ids one [`Body::JobStatus`](#body) may ask about.

### MAX_UPLOAD_COPIES

```rust
pub const MAX_UPLOAD_COPIES: usize = 16;
```

Most destinations one uploaded file may be written to.

### MAX_UPLOAD_FILES

```rust
pub const MAX_UPLOAD_FILES: usize = 512;
```

Most files one upload may declare.

### MAX_UPLOAD_FILE_BYTES

```rust
pub const MAX_UPLOAD_FILE_BYTES: u64 = _;
```

Largest file one upload may declare (8 GiB).

### MAX_UPLOAD_KEY_BYTES

```rust
pub const MAX_UPLOAD_KEY_BYTES: usize = 1024;
```

Longest storage key an upload may declare.

### AUTHORIZE_PATH

```rust
pub const AUTHORIZE_PATH: &str = "/oidc/authorize";
```

The owner's authorization endpoint (path).

### BRANCH_CLIENT_ID

```rust
pub const BRANCH_CLIENT_ID: &str = "bookclerk-branch";
```

The built-in public OAuth client a joining branch signs in through. It
is never stored in the owner's clients table.

### BRANCH_CODE_TTL_SECS

```rust
pub const BRANCH_CODE_TTL_SECS: i64 = 60;
```

Seconds an authorization code for [`BRANCH_CLIENT_ID`](#branch_client_id) lives.

### BRANCH_SCOPE

```rust
pub const BRANCH_SCOPE: &str = "node:enroll";
```

The only scope [`BRANCH_CLIENT_ID`](#branch_client_id) may ask for.

### ENROLL_OAUTH_PATH

```rust
pub const ENROLL_OAUTH_PATH: &str = "/api/node/v1/enroll/oauth";
```

`POST`: redeem an approved branch sign-in.

### INFO_PATH

```rust
pub const INFO_PATH: &str = "/api/node/v1/info";
```

`GET`: what a library tells a joining install before sign-in.

### JOIN_CALLBACK_PATH

```rust
pub const JOIN_CALLBACK_PATH: &str = "/join/callback";
```

Path of the joining install's own callback (the SPA route the browser
returns to, and the CLI's loopback listener).

### MAX_BRANCH_NAME_BYTES

```rust
pub const MAX_BRANCH_NAME_BYTES: usize = 64;
```

Longest branch name sent.

### MAX_LIBRARY_NAME_BYTES

```rust
pub const MAX_LIBRARY_NAME_BYTES: usize = 253;
```

Longest library name accepted: a library is named by its public host,
and a DNS name runs to 253 bytes.

### PUBLIC_KEY_LEN

```rust
pub const PUBLIC_KEY_LEN: usize = 32;
```

Length of either public key.

### MAX_LOOKUP_EDITIONS

```rust
pub const MAX_LOOKUP_EDITIONS: usize = 8;
```

Most editions one [`LookupAnswer`](#lookupanswer) may list.

### MAX_LOOKUP_IDENTIFIERS

```rust
pub const MAX_LOOKUP_IDENTIFIERS: usize = 16;
```

Most identifiers one [`LookupWork`](#lookupwork) may carry.

### MAX_LOOKUP_WORKS

```rust
pub const MAX_LOOKUP_WORKS: usize = 100;
```

Most works one [`crate::Body::CatalogAsk`](#body) or
[`crate::Body::CatalogFound`](#body) may name.

### MAX_WORK_REF_BYTES

```rust
pub const MAX_WORK_REF_BYTES: usize = 64;
```

Longest correlation key ([`LookupWork::work`](#lookupwork)).

### MAX_BRANCH_ORIGIN_BYTES

```rust
pub const MAX_BRANCH_ORIGIN_BYTES: usize = 255;
```

Longest origin a branch may report, in bytes.

### ITEM_REQUEST

```rust
pub const ITEM_REQUEST: &str = "request";
```

The work item name of a request in a [`crate::Body::Lost`](#body) answer.

### MAX_CLAIMED_REQUESTS

```rust
pub const MAX_CLAIMED_REQUESTS: usize = 16;
```

Most requests one [`crate::Body::RequestsClaimed`](#body) may carry.

### MAX_REQUEST_JOB_BYTES

```rust
pub const MAX_REQUEST_JOB_BYTES: usize = 96;
```

Longest job id a request may carry.

### COPY_TOKEN_AAD_DOMAIN

```rust
pub const COPY_TOKEN_AAD_DOMAIN: &[u8] = b"bookclerk-node.v1 copy-token";
```

Domain of the associated data of a sealed copy token.

### INSTANCE_SECRETS_AAD_DOMAIN

```rust
pub const INSTANCE_SECRETS_AAD_DOMAIN: &[u8] = b"bookclerk-node.v1 instance-secrets";
```

Domain tag that starts the associated data of an instance's `SECRETS`.

### SEALED_BOX_INFO

```rust
pub const SEALED_BOX_INFO: &[u8] = b"bookclerk-node.v1 sealed-box";
```

HKDF info string of a sealed box.

### SEALED_BOX_OVERHEAD

```rust
pub const SEALED_BOX_OVERHEAD: usize = _;
```

Bytes a sealed box adds to its plaintext (ephemeral key and tag).

### SECRETS_AAD_DOMAIN

```rust
pub const SECRETS_AAD_DOMAIN: &[u8] = b"bookclerk-node.v1 job-secrets";
```

Domain tag that starts the associated data of a secret bundle.

### SECRET_UPDATE_AAD_DOMAIN

```rust
pub const SECRET_UPDATE_AAD_DOMAIN: &[u8] = b"bookclerk-node.v1 secret-update";
```

Domain tag that starts the associated data of a token write-back.

### UPLOAD_TOKEN_AAD_DOMAIN

```rust
pub const UPLOAD_TOKEN_AAD_DOMAIN: &[u8] = b"bookclerk-node.v1 upload-token";
```

Domain tag that starts the associated data of an upload token.

### DEK_LEN

```rust
pub const DEK_LEN: usize = 32;
```

Length of the data-encryption key.

### MAX_BOOTSTRAP_CREDENTIAL_BYTES

```rust
pub const MAX_BOOTSTRAP_CREDENTIAL_BYTES: usize = 4096;
```

The longest database credential (a Postgres URL, a D1 API token) a
bootstrap may carry, in bytes.

### MAX_BOOTSTRAP_CREDENTIAL_ID_BYTES

```rust
pub const MAX_BOOTSTRAP_CREDENTIAL_ID_BYTES: usize = 128;
```

The longest provider id for a database credential a bootstrap may carry,
in bytes.

### MAX_SHARED_BRANCH_BOOTSTRAP_BYTES

```rust
pub const MAX_SHARED_BRANCH_BOOTSTRAP_BYTES: usize = _;
```

The longest opened bootstrap accepted, in bytes.

### SHARED_BRANCH_BOOTSTRAP_AAD_DOMAIN

```rust
pub const SHARED_BRANCH_BOOTSTRAP_AAD_DOMAIN: &[u8] = b"bookclerk-node.v1 shared-branch-bootstrap";
```

Domain tag that starts the associated data of a shared branch bootstrap.

### SHARED_BRANCH_BOOTSTRAP_VERSION

```rust
pub const SHARED_BRANCH_BOOTSTRAP_VERSION: u32 = 1;
```

Version of [`SharedBranchBootstrap`](#sharedbranchbootstrap).

### SHARED_BRANCH_GRANTED_SCOPE

```rust
pub const SHARED_BRANCH_GRANTED_SCOPE: &str = "node:enroll node:shared_branch";
```

The scope a shared branch code is granted (stored with the code at consent).

### SHARED_BRANCH_OFFER_PARAM

```rust
pub const SHARED_BRANCH_OFFER_PARAM: &str = "shared_branch_offer";
```

Authorize-request parameter by which a joining install offers to join as
a shared branch (`shared_branch_offer=1`): it is empty, so its own data may be replaced.
Without it the consent page offers a branch only.

### ARTIFACT_AUTH_SCHEME

```rust
pub const ARTIFACT_AUTH_SCHEME: &str = "Bookclerk-Node";
```

`Authorization` scheme of a signed artifact request:
`Bookclerk-Node <node_id> <issued_at> <sig>`.

### ARTIFACT_DOMAIN

```rust
pub const ARTIFACT_DOMAIN: &[u8] = b"bookclerk-node-artifact-v1";
```

Domain tag that starts every signed artifact request.

### ARTIFACT_REQUEST_MAX_SKEW_SECS

```rust
pub const ARTIFACT_REQUEST_MAX_SKEW_SECS: i64 = 300;
```

How far a signed artifact request's time may be from the owner's clock.

### AUTH_DOMAIN

```rust
pub const AUTH_DOMAIN: &[u8] = b"bookclerk-node-auth-v1";
```

Domain tag that starts every signed transcript.

### NONCE_LEN

```rust
pub const NONCE_LEN: usize = 32;
```

Length of a challenge nonce.

### BRANCH_API_PATHS

```rust
pub const BRANCH_API_PATHS: [&str; 13] = _;
```

API paths [`TunnelExposure::Branches`](#tunnelexposure) serves. Each also covers the paths
below it.

### LEGACY_CONTROL_PATHS

```rust
pub const LEGACY_CONTROL_PATHS: [&str; 5] = _;
```

Legacy control routes outside `/api/` that
[`TunnelExposure::Branches`](#tunnelexposure) refuses. Each also covers the paths below
it.

### OPERATOR_ONLY_PATHS

```rust
pub const OPERATOR_ONLY_PATHS: [&str; 6] = _;
```

Paths no tunnel request reaches: the operator-token sign-in, the loopback
tray handoff, operator elevation and the operator's first-Owner
bootstrap. Each also covers the paths below it.

### UPLOAD_PART_BYTES

```rust
pub const UPLOAD_PART_BYTES: u64 = _;
```

Largest part a host sends (64 MiB): under the request-body caps of the
relays an install can sit behind (100 MB on Cloudflare's Free plan).

### MAX_COPY_CONTENT_TYPE_BYTES

```rust
pub const MAX_COPY_CONTENT_TYPE_BYTES: usize = 64;
```

Longest [`WindowCopy::content_type`](#windowcopy), in bytes.

### MAX_COPY_VERSION_BYTES

```rust
pub const MAX_COPY_VERSION_BYTES: usize = 64;
```

Longest [`WindowCopy::version`](#windowcopy), in bytes.

### MAX_WINDOW_WORKS

```rust
pub const MAX_WINDOW_WORKS: usize = 100;
```

Most works one [`crate::Body::WindowPage`](#body) may carry.

### ARTIFACTS_PATH

```rust
pub const ARTIFACTS_PATH: &str = "/api/node/v1/artifacts/";
```

Path prefix of the owner's artifact endpoint: `GET
/api/node/v1/artifacts/<archive_sha256>` with a signed `Authorization`
([`sign_artifact_request`](#sign_artifact_request)) streams one stored archive to a host that
runs it.

### UPLOADS_PATH

```rust
pub const UPLOADS_PATH: &str = "/api/node/v1/uploads/";
```

Path prefix of the owner's upload endpoint (H-4 design, section 6):
`PUT|HEAD <prefix><upload_id>/files/<n>` streams one declared file and
`POST <prefix><upload_id>/commit` commits the title, each with
`Authorization: Bookclerk-Upload <token>` ([`UPLOAD_AUTH_SCHEME`](#upload_auth_scheme)).

### UPLOAD_AUTH_SCHEME

```rust
pub const UPLOAD_AUTH_SCHEME: &str = "Bookclerk-Upload";
```

`Authorization` scheme of the upload endpoint.

### SUBPROTOCOL

```rust
pub const SUBPROTOCOL: &str = "bookclerk-node.v1";
```

WebSocket subprotocol a host must offer.

### FRAME_VERSION

```rust
pub const FRAME_VERSION: u32 = 1;
```

Frame envelope version (`v`).

### MAX_FRAME_BYTES

```rust
pub const MAX_FRAME_BYTES: usize = _;
```

Largest frame either side accepts (1 MiB).

### PING_INTERVAL_SECS

```rust
pub const PING_INTERVAL_SECS: u64 = 20;
```

Seconds between pings from each side.

### IDLE_TIMEOUT_SECS

```rust
pub const IDLE_TIMEOUT_SECS: u64 = 60;
```

Seconds without any inbound frame before a side closes the connection.

### SEND_QUEUE_FRAMES

```rust
pub const SEND_QUEUE_FRAMES: usize = 64;
```

Frames a side queues for sending before it stops forwarding wakes and
waits for the peer to read.

### HANDSHAKE_TIMEOUT_SECS

```rust
pub const HANDSHAKE_TIMEOUT_SECS: u64 = 10;
```

Seconds the owner waits for each handshake frame.

### CLOSE_AUTH_FAILED

```rust
pub const CLOSE_AUTH_FAILED: u16 = 4401;
```

Close code: the handshake failed (unknown node, bad signature, or a
malformed or late handshake frame).

### CLOSE_REVOKED

```rust
pub const CLOSE_REVOKED: u16 = 4403;
```

Close code: the node's credential is revoked.

### CLOSE_SUPERSEDED

```rust
pub const CLOSE_SUPERSEDED: u16 = 4409;
```

Close code: a newer authenticated connection for the same node replaced
this one.

### CLOSE_PROTOCOL

```rust
pub const CLOSE_PROTOCOL: u16 = 4400;
```

Close code: a frame broke the protocol after the handshake (binary,
oversized, malformed, or the wrong version).
