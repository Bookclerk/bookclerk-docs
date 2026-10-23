# bookclerk_node_proto

Wire protocol between a Bookclerk host node and its owner cluster.

A host connects to `GET /api/node/v1/connect` with the WebSocket
subprotocol [`SUBPROTOCOL`](#subprotocol) and proves itself with the Ed25519 key it
enrolled (H-4 design, section 3):

1. host → owner: [`Body::Hello`](#body) `{node_id, incarnation,
   software_version, resume}`
2. owner → host: [`Body::Challenge`](#body) `{nonce, cluster_id}`
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

A claim may carry a [`ClaimFilter`](#claimfilter) (the kinds the host runs and the
plugins its operator consented to); a job handed back unrun goes with
[`Body::Release`](#body). To act on wakes the host reads
[`Body::DeploymentsList`](#body) and [`Body::InstanceConfigGet`](#body) (`SECRETS`
sealed with [`seal_instance_secrets`](#seal_instance_secrets)), reports
[`Body::Observation`](#body)s, and fetches stored archives over HTTP at
[`ARTIFACTS_PATH`](#artifacts_path) with a request signed by [`sign_artifact_request`](#sign_artifact_request).

A scan run on a host returns its rows in fenced [`Body::ScanResults`](#body)
batches; the owner validates and writes them. An acquire asks
[`Body::AcquirePlan`](#body) for its titles, then for each title asks
[`Body::UploadGrant`](#body) and streams the files over HTTP at
[`UPLOADS_PATH`](#uploads_path) with the token sealed in [`Body::UploadGranted`](#body); the
owner streams them on to its output destinations without staging them,
and the job completes when the last title's upload commits.

A branch shares the titles of its own library with the library it
joined ([`Body::CatalogBegin`](#body), [`Body::CatalogSync`](#body),
[`Body::CatalogUpload`](#body); see [`CatalogTitle`](#catalogtitle)): titles carry no account
or user field, and files reuse the upload endpoint.

A host that leaves sends [`Body::NodeLeaving`](#body); the owner revokes the
node at once. Joining with a sign-in instead of a link is the OAuth
code + PKCE flow of the built-in client [`BRANCH_CLIENT_ID`](#branch_client_id) ([`OwnerInfo`](#ownerinfo),
[`OAuthEnrollRequest`](#oauthenrollrequest)). An Owner of the library may instead let the
install join as a clerk (an owner node, [`ConnectionType::Clerk`](#connectiontype)): its
redeem answer carries a [`ClerkBootstrap`](#clerkbootstrap) sealed to the node's box key.

This crate is pure: no I/O, no clock, no randomness. The daemon (owner)
and the host client share it so both sides sign and check the same
bytes.

| Item | Kind | Summary |
| --- | --- | --- |
| [`CatalogFile`](#catalogfile) | struct | One file of a `CatalogTitle`. |
| [`CatalogMetadata`](#catalogmetadata) | struct | Bibliographic metadata of a `CatalogTitle`. |
| [`CatalogRef`](#catalogref) | struct | A title named by store and product id. |
| [`CatalogTitle`](#catalogtitle) | struct | One title, as a branch shares it. |
| [`ClerkBootstrap`](#clerkbootstrap) | struct | What a clerk receives at redeem: the `[database]` bootstrap and the DEK, for the node id the library recorded (`kind = owner`). `Debug` prints no secret. |
| [`AcquireTarget`](#acquiretarget) | struct | One title an acquire job should fetch on a host (`Body::AcquireTargets`). |
| [`ClaimFilter`](#claimfilter) | struct | What a host is willing to claim (`claim.filter`). |
| [`FileCopy`](#filecopy) | struct | Where one uploaded file is written: a destination and its key there. |
| [`Frame`](#frame) | struct | One frame: the envelope and its body. |
| [`HeldFence`](#heldfence) | struct | A lease the host still holds when it reconnects (`hello.resume`). |
| [`JobStatusEntry`](#jobstatusentry) | struct | One job's state in `Body::JobStatuses`. |
| [`PlanDestination`](#plandestination) | struct | One output destination of the owner node that answered an acquire plan (`Body::AcquireTargets`). |
| [`RemoteArtifact`](#remoteartifact) | struct | One downloadable artifact of a `RemoteVersion`. |
| [`RemoteDeployment`](#remotedeployment) | struct | One deployment that targets a host (`deployments.items`). |
| [`RemoteVersion`](#remoteversion) | struct | The active version of a plugin a host is asked to run. |
| [`ScanImport`](#scanimport) | struct | The owner's library import settings a host scan runs with. |
| [`UploadAnswer`](#uploadanswer) | struct | Answer of the upload endpoint (every status carries one). |
| [`UploadCommitRequest`](#uploadcommitrequest) | struct | Body of `POST <UPLOADS_PATH><upload_id>/commit`. |
| [`UploadFile`](#uploadfile) | struct | One file of an upload (`Body::UploadGrant`). Its content crosses the network once and is written to every copy, each destination under the key that destination's own naming options give it. |
| [`WorkJob`](#workjob) | struct | A job handed to a host. |
| [`OAuthEnrollAnswer`](#oauthenrollanswer) | struct | `POST /api/node/v1/enroll/oauth` answer. |
| [`OAuthEnrollRequest`](#oauthenrollrequest) | struct | `POST /api/node/v1/enroll/oauth` body. Keys are base64url without padding. |
| [`OwnerInfo`](#ownerinfo) | struct | `GET /api/node/v1/info` answer. |
| [`SecretEntry`](#secretentry) | struct | One secret in a bundle. |
| [`SecretRef`](#secretref) | struct | Names one `encrypted_secrets` row of a store account. It carries no secret material. |
| [`SecretValue`](#secretvalue) | struct | Secret bytes: zeroized on drop, redacted in `Debug`, and base64url on the wire. |
| [`ArtifactRequest`](#artifactrequest) | struct | What a signed artifact request covers. |
| [`AuthContext`](#authcontext) | struct | What a handshake signature covers. |
| [`ClerkDatabase`](#clerkdatabase) | enum | The library's database, as a clerk needs it to connect. |
| [`ConnectionType`](#connectiontype) | enum | How a joining install connects to a library. |
| [`Body`](#body) | enum | What a frame says, tagged by `type`. |
| [`FrameError`](#frameerror) | enum | Why a frame was not accepted. |
| [`KeyError`](#keyerror) | enum | Why a public key was refused. |
| [`SealError`](#sealerror) | enum | Why a sealed box could not be made or opened. |
| [`AuthError`](#autherror) | enum | Why a handshake signature was refused. |
| [`is_ancestry`](#is_ancestry) | function | True when every id in `ancestry` is a plain cluster id and there are at most `MAX_ANCESTRY`. |
| [`is_cluster_id`](#is_cluster_id) | function | True for a plain cluster id in an ancestry. |
| [`is_product_id`](#is_product_id) | function | True for a product id: 1 to 256 bytes, no whitespace at the ends and no control characters. |
| [`is_store_id`](#is_store_id) | function | True for a store id: a short lowercase token. |
| [`clerk_bootstrap_aad`](#clerk_bootstrap_aad) | function | Associated data of a clerk bootstrap for `node_id` of `cluster_id`: the domain tag, then each field as a 4-byte big-endian length and its bytes. |
| [`open_clerk_bootstrap`](#open_clerk_bootstrap) | function | Opens a bootstrap made by `seal_clerk_bootstrap` with the joining node's box secret, for the cluster and node ids the redeem answered. |
| [`seal_clerk_bootstrap`](#seal_clerk_bootstrap) | function | Seals `bootstrap` to the joining node's box key `recipient`, as base64url text for the redeem answer. |
| [`is_destination_id`](#is_destination_id) | function | True when `id` is a destination id: a short lowercase token. |
| [`is_storage_key`](#is_storage_key) | function | True when `key` is a relative storage key: no empty, `.` or `..` segment, no backslash or control character, not absolute, at most `MAX_UPLOAD_KEY_BYTES`, and never a Windows path that leaves the destination's root (`colon_is_safe`). |
| [`is_branch_name`](#is_branch_name) | function | True for a branch name: 1 to `MAX_BRANCH_NAME_BYTES` bytes, no control characters. |
| [`is_branch_redirect_uri`](#is_branch_redirect_uri) | function | True for a redirect URI `BRANCH_CLIENT_ID` may use: an absolute `https` URL, or an `http` URL whose host is a loopback IP literal or `localhost` (RFC 8252 section 7.3), with no user info and no fragment. |
| [`is_library_name`](#is_library_name) | function | True for a library name: 1 to `MAX_LIBRARY_NAME_BYTES` bytes, no control characters. |
| [`pkce_challenge`](#pkce_challenge) | function | The RFC 7636 S256 challenge of `verifier`: base64url (no padding) of its SHA-256. |
| [`box_public_key`](#box_public_key) | function | The X25519 public key of a 32-byte secret (clamped per RFC 7748). |
| [`sign_public_key`](#sign_public_key) | function | The Ed25519 public key of a 32-byte signing seed. |
| [`validate_box_public_key`](#validate_box_public_key) | function | Validates an X25519 public box key (a Montgomery u-coordinate). |
| [`validate_sign_public_key`](#validate_sign_public_key) | function | Validates an Ed25519 public signing key. |
| [`instance_secrets_aad`](#instance_secrets_aad) | function | Associated data of `instance_id`'s `SECRETS` handed to `node_id` at document `revision`. |
| [`open`](#open) | function | Opens a box sealed to the public key of `recipient_secret`. |
| [`open_instance_secrets`](#open_instance_secrets) | function | Opens a payload made by `seal_instance_secrets` with the node's box secret. |
| [`open_secret_update`](#open_secret_update) | function | Opens a write-back made by `seal_secret_update` with the owner's per-connection box secret. |
| [`open_secrets`](#open_secrets) | function | Opens a bundle made by `seal_secrets` with the node's box secret. |
| [`open_upload_token`](#open_upload_token) | function | Opens a token made by `seal_upload_token` with the node's box secret. |
| [`seal`](#seal) | function | Seals `plaintext` to `recipient` (an X25519 public key) with the ephemeral secret `ephemeral_secret`, binding `aad`. |
| [`seal_instance_secrets`](#seal_instance_secrets) | function | Seals an instance's `SECRETS` payload (JSON bytes) for `node_id`'s box key `recipient`, bound to the instance and document revision, as base64url text for an `instance_config` frame. |
| [`seal_secret_update`](#seal_secret_update) | function | Seals a refreshed token to the owner's per-connection box key `owner_box_key` (from `welcome`), bound to the node, job, fence and ref, as base64url text for a `secret.update` frame. |
| [`seal_secrets`](#seal_secrets) | function | Seals `entries` for `node_id`'s box key `recipient`, bound to `job_id` and `fence`, as base64url text for a `work` frame. |
| [`seal_upload_token`](#seal_upload_token) | function | Seals an upload token for `node_id`'s box key `recipient`, bound to the job, fence and upload, as base64url text for an `upload.granted` frame. The token is a bearer credential for the upload endpoint, so it never travels as frame text. |
| [`secret_update_aad`](#secret_update_aad) | function | Associated data of a token write-back from `node_id` for `secret_ref`, handed with `job_id` under `fence`. |
| [`secrets_aad`](#secrets_aad) | function | Associated data of the secret bundle handed to `node_id` for `job_id` under lease generation `fence`: the domain tag, then each field as a 4-byte big-endian length and its bytes. |
| [`upload_token_aad`](#upload_token_aad) | function | Associated data of the token for upload `upload_id`, granted to `node_id` for `job_id` under lease generation `fence`. |
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
| [`CATALOG_ITEM`](#catalog_item) | constant | Item kind of the catalog fence (`lost.item`), and the job id a catalog upload's token is sealed for. |
| [`MAX_ANCESTRY`](#max_ancestry) | constant | Most cluster ids an ancestry may list. |
| [`MAX_CATALOG_BATCH_TITLES`](#max_catalog_batch_titles) | constant | Most titles one `crate::Body::CatalogSync` batch may carry. |
| [`MAX_CATALOG_CURSOR_BYTES`](#max_catalog_cursor_bytes) | constant | Longest cursor. |
| [`MAX_CATALOG_TITLE_BYTES`](#max_catalog_title_bytes) | constant | Largest serialized title in a batch. |
| [`MAX_CATALOG_WANTED`](#max_catalog_wanted) | constant | Most titles one answer may say are wanted. |
| [`CLERK_BOOTSTRAP_AAD_DOMAIN`](#clerk_bootstrap_aad_domain) | constant | Domain tag that starts the associated data of a clerk bootstrap. |
| [`CLERK_BOOTSTRAP_VERSION`](#clerk_bootstrap_version) | constant | Version of `ClerkBootstrap`. |
| [`CLERK_GRANTED_SCOPE`](#clerk_granted_scope) | constant | The scope a clerk code is granted (stored with the code at consent). |
| [`CLERK_OFFER_PARAM`](#clerk_offer_param) | constant | Authorize-request parameter by which a joining install offers to join as a clerk (`clerk_offer=1`): it is empty, so its own data may be replaced. Without it the consent page offers a branch only. |
| [`DEK_LEN`](#dek_len) | constant | Length of the data-encryption key. |
| [`ITEM_JOB`](#item_job) | constant | Work item kind for jobs. |
| [`MAX_DESTINATION_ID_BYTES`](#max_destination_id_bytes) | constant | Longest destination id. |
| [`MAX_SCAN_BATCH_ROWS`](#max_scan_batch_rows) | constant | Most rows one `Body::ScanResults` batch may carry. |
| [`MAX_SCAN_ROW_BYTES`](#max_scan_row_bytes) | constant | Largest serialized row in a `Body::ScanResults` batch. |
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
| [`INSTANCE_SECRETS_AAD_DOMAIN`](#instance_secrets_aad_domain) | constant | Domain tag that starts the associated data of an instance's `SECRETS`. |
| [`SEALED_BOX_INFO`](#sealed_box_info) | constant | HKDF info string of a sealed box. |
| [`SEALED_BOX_OVERHEAD`](#sealed_box_overhead) | constant | Bytes a sealed box adds to its plaintext (ephemeral key and tag). |
| [`SECRETS_AAD_DOMAIN`](#secrets_aad_domain) | constant | Domain tag that starts the associated data of a secret bundle. |
| [`SECRET_UPDATE_AAD_DOMAIN`](#secret_update_aad_domain) | constant | Domain tag that starts the associated data of a token write-back. |
| [`UPLOAD_TOKEN_AAD_DOMAIN`](#upload_token_aad_domain) | constant | Domain tag that starts the associated data of an upload token. |
| [`ARTIFACT_AUTH_SCHEME`](#artifact_auth_scheme) | constant | `Authorization` scheme of a signed artifact request: `Bookclerk-Node <node_id> <issued_at> <sig>`. |
| [`ARTIFACT_DOMAIN`](#artifact_domain) | constant | Domain tag that starts every signed artifact request. |
| [`ARTIFACT_REQUEST_MAX_SKEW_SECS`](#artifact_request_max_skew_secs) | constant | How far a signed artifact request's time may be from the owner's clock. |
| [`AUTH_DOMAIN`](#auth_domain) | constant | Domain tag that starts every signed transcript. |
| [`NONCE_LEN`](#nonce_len) | constant | Length of a challenge nonce. |
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

### CatalogFile

```rust
pub struct CatalogFile {
    pub key: String,
    pub size: u64,
    pub content_type: Option<String>,
}
```

One file of a [`CatalogTitle`](#catalogtitle).

**Fields**

- `key`: Relative storage key (the same on every destination of the
  receiving library).
- `size`: Size in bytes.
- `content_type`: MIME type.

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
}
```

A title named by store and product id.

**Fields**

- `store`: Store id.
- `product_id`: Product id.

Implements: `Clone`, `Debug`, `Deserialize<'de>`, `Eq`, `Hash`, `Ord`, `PartialEq`, `PartialOrd`, `Serialize`

### CatalogTitle

```rust
pub struct CatalogTitle {
    pub store: String,
    pub product_id: String,
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

Why the title is malformed, or `None`: the store and product id, a
non-empty bounded title and bounded metadata, between one and
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

### ClerkBootstrap

```rust
pub struct ClerkBootstrap {
    pub v: u32,
    pub cluster_id: String,
    pub node_id: String,
    pub database: ClerkDatabase,
    pub dek: SecretValue,
}
```

What a clerk receives at redeem: the `[database]` bootstrap and the DEK,
for the node id the library recorded (`kind = owner`). `Debug` prints no
secret.

**Fields**

- `v`: [`CLERK_BOOTSTRAP_VERSION`](#clerk_bootstrap_version).
- `cluster_id`: The library's cluster id.
- `node_id`: The node id recorded for this clerk.
- `database`: How to reach the library's database.
- `dek`: The library's 32-byte data-encryption key.

Implements: `Clone`, `Debug`, `Deserialize<'de>`, `Eq`, `PartialEq`, `Serialize`

### AcquireTarget

```rust
pub struct AcquireTarget {
    pub book_uuid: String,
    pub plugin_key: String,
    pub account_id: String,
    pub book: Value,
    pub parent: Option<Value>,
}
```

One title an acquire job should fetch on a host
([`Body::AcquireTargets`](#body)).

**Fields**

- `book_uuid`: Library id of the title.
- `plugin_key`: Plugin key of the store that sells it.
- `account_id`: Store account that owns it.
- `book`: The owner's library row (JSON), for naming and metadata.
- `parent`: The owner's row for the title's podcast parent (JSON), when the
  title is an episode whose parent is in the owner's library: a
  destination that saves episodes to the parent's folder names them
  from it, as an acquire on the owner would.

Implements: `Clone`, `Debug`, `Deserialize<'de>`, `Eq`, `PartialEq`, `Serialize`

### ClaimFilter

```rust
pub struct ClaimFilter {
    pub kinds: Vec<String>,
    pub plugins: Vec<String>,
}
```

What a host is willing to claim (`claim.filter`).

**Fields**

- `kinds`: Job kinds the host runs.
- `plugins`: Plugin keys the host's operator consented to.

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

Implements: `Clone`, `Debug`, `Deserialize<'de>`, `Eq`, `PartialEq`, `Serialize`

### PlanDestination

```rust
pub struct PlanDestination {
    pub id: String,
    pub options: Value,
}
```

One output destination of the owner node that answered an acquire plan
([`Body::AcquireTargets`](#body)).

**Fields**

- `id`: Destination id on that owner node (its output kind, such as `local`
  or `s3`; one destination per kind).
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
- `grant_json`: The grant the cluster's operator approved (JSON); empty while the
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

### UploadAnswer

```rust
pub struct UploadAnswer {
    pub status: String,
    pub reason: Option<String>,
    pub job_completed: bool,
}
```

Answer of the upload endpoint (every status carries one).

**Fields**

- `status`: Stable outcome: `received`, `committed`, `lost`, `refused`,
  `misdirected`, `unauthorized`, `unavailable`.
- `reason`: Why, for `refused`.
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

#### UploadFile::is_well_formed

```rust
pub fn is_well_formed(&self) -> bool
```

True when the file has between one and [`MAX_UPLOAD_COPIES`](#max_upload_copies)
copies, each naming a different destination id
([`is_destination_id`](#is_destination_id)) and a storage key ([`is_storage_key`](#is_storage_key)); the
size is within [`MAX_UPLOAD_FILE_BYTES`](#max_upload_file_bytes); and the digest is
lowercase hex SHA-256.

#### UploadFile::key_on

```rust
pub fn key_on(&self, destination: &str) -> Option<&str>
```

This file's key on `destination`, if it is written there.

Implements: `Clone`, `Debug`, `Deserialize<'de>`, `Eq`, `PartialEq`, `Serialize`

### WorkJob

```rust
pub struct WorkJob {
    pub id: String,
    pub kind: String,
    pub payload: Value,
    pub attempt: i64,
    pub scan_import: Option<ScanImport>,
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

Implements: `Clone`, `Debug`, `Deserialize<'de>`, `Eq`, `PartialEq`, `Serialize`

### OAuthEnrollAnswer

```rust
pub struct OAuthEnrollAnswer {
    pub cluster_id: String,
    pub node_id: String,
    pub connection: ConnectionType,
    pub bootstrap: Option<String>,
}
```

`POST /api/node/v1/enroll/oauth` answer.

**Fields**

- `cluster_id`: The library's cluster id.
- `node_id`: The node id recorded for the joining install.
- `connection`: The connection type recorded (`branch` when left out).
- `bootstrap`: For a clerk: the [`crate::ClerkBootstrap`](#clerkbootstrap) sealed to the node's box
  key ([`crate::seal_clerk_bootstrap`](#seal_clerk_bootstrap)).

Implements: `Clone`, `Debug`, `Deserialize<'de>`, `Eq`, `PartialEq`, `Serialize`

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
    pub cluster_id: String,
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

- `cluster_id`: The library's cluster id.
- `library_name`: A name to show for the library.
- `origin`: The library's canonical origin: the one the node handshake signs and
  the authorize endpoint lives under.
- `transport`: The node API transport mode (`acme`, `proxy`, `self_signed`,
  `loopback`).
- `enrollment`: Whether the library lets new branches join (`core.host_enrollment`).
- `spki_fingerprint`: In `self_signed` mode: base64url SHA-256 of the certificate's
  SubjectPublicKeyInfo, which the joining operator confirms against
  the owner's Settings → Branches page.
- `software_version`: The library's Bookclerk version.
- `ancestry`: The library's ancestry: its own cluster id first, then the cluster
  id of every library above it up to the root. A joining library
  refuses a parent whose ancestry names its own cluster id, so a
  tree of libraries never becomes a cycle.

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

### ArtifactRequest

```rust
pub struct ArtifactRequest<'a> {
    pub cluster_id: &'a str,
    pub node_id: &'a str,
    pub owner_origin: &'a str,
    pub archive_sha256: &'a str,
    pub issued_at: i64,
}
```

What a signed artifact request covers.

**Fields**

- `cluster_id`: Owner cluster id.
- `node_id`: Requesting node id.
- `owner_origin`: Owner origin as [`canonical_origin`](#canonical_origin) spells it.
- `archive_sha256`: Lowercase hex SHA-256 of the archive asked for.
- `issued_at`: Unix seconds when the host signed the request.

Implements: `Clone`, `Copy`, `Debug`, `Eq`, `PartialEq`

### AuthContext

```rust
pub struct AuthContext<'a> {
    pub cluster_id: &'a str,
    pub node_id: &'a str,
    pub incarnation: &'a str,
    pub nonce: &'a [u8; 32],
    pub owner_origin: &'a str,
}
```

What a handshake signature covers.

**Fields**

- `cluster_id`: Owner cluster id from the challenge.
- `node_id`: Node id from the hello.
- `incarnation`: Incarnation from the hello.
- `nonce`: Nonce from the challenge.
- `owner_origin`: Owner origin as [`canonical_origin`](#canonical_origin) spells it. The host uses the
  origin it dialed; the owner uses its own configured origin, so a
  signature relayed through another server does not verify.

Implements: `Clone`, `Copy`, `Debug`, `Eq`, `PartialEq`

## Enums

### ClerkDatabase

```rust
pub enum ClerkDatabase {
    Postgres { url: SecretValue },
    D1 { account_id: String, database_id: String, api_base: String },
}
```

The library's database, as a clerk needs it to connect.

**Variants**

- `Postgres`: PostgreSQL: the connection URL (it carries credentials).
  - `url`: `postgres://user:password@host/db`.
- `D1`: Cloudflare D1: the database's ids. The API token stays
  environment-only bootstrap on each node and is never sent.
  - `account_id`: Cloudflare account id.
  - `database_id`: D1 database UUID.
  - `api_base`: Cloudflare API base.

#### ClerkDatabase::plugin

```rust
pub fn plugin(&self) -> &'static str
```

The `[database].plugin` id (`postgres`, `d1`).

Implements: `Clone`, `Debug`, `Deserialize<'de>`, `Eq`, `PartialEq`, `Serialize`

### ConnectionType

```rust
pub enum ConnectionType {
    Branch,
    Clerk,
}
```

How a joining install connects to a library.

**Variants**

- `Branch`: A host node: its own database, work over the node socket (default).
- `Clerk`: An owner node: shares the library's database and DEK.

#### ConnectionType::as_str

```rust
pub fn as_str(self) -> &'static str
```

Stable text (`branch`, `clerk`).

#### ConnectionType::parse

```rust
pub fn parse(raw: &str) -> Option<Self>
```

Parses `branch` or `clerk` (case-insensitive); anything else is
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

### Body

```rust
pub enum Body {
    Hello { node_id: String, incarnation: String, software_version: String, resume: Vec<HeldFence> },
    Challenge { nonce: String, cluster_id: String },
    Auth { sig: String },
    Welcome { node_id: String, cluster_id: String, box_key: String },
    Resumed { renewed: Vec<HeldFence>, lost: Vec<HeldFence>, unchecked: Vec<HeldFence>, lease_secs: Option<u64> },
    Error { code: String, message: String },
    Heartbeat { started_at: String, interval_secs: u64, cpu_millicores: i64, memory_bytes: Option<i64>, cpu_used_millicores: Option<i64>, memory_available_bytes: Option<i64>, files_dir_free_bytes: Option<i64> },
    Claim { item: String, class: Option<String>, filter: Option<ClaimFilter> },
    Work { item: String, job: WorkJob, fence: i64, lease_secs: u64, sealed_secrets: Option<String>, plugin_keys: Vec<String>, source_plugin_key: Option<String>, resumed: bool },
    NoWork { retry_after_secs: u64 },
    Renew { item: String, job_id: String, fence: i64, progress: Option<String> },
    Renewed { lease_secs: u64, cancel: bool },
    Lost { item: String, job_id: String, fence: i64 },
    Complete { job_id: String, fence: i64, detail: Option<String> },
    Fail { job_id: String, fence: i64, error_kind: String, message: String },
    Release { job_id: String, fence: i64, reason: String, resumed: bool },
    DeploymentsList {  },
    Deployments { items: Vec<RemoteDeployment> },
    InstanceConfigGet { instance_id: String },
    InstanceConfig { instance_id: String, revision: i64, config: Value, sealed_secrets: Option<String>, secrets_withheld: bool },
    Observation { deployment_id: String, status: String, detail: String, applied_config_revision: Option<i64> },
    SecretUpdate { job_id: String, fence: i64, secret_ref: SecretRef, sealed_value: String },
    ScanResults { job_id: String, fence: i64, plugin_key: String, batch: u64, books: Vec<Value> },
    ScanAccepted { batch: u64, upserted: u64 },
    AcquirePlan { job_id: String, fence: i64 },
    AcquireTargets { targets: Vec<AcquireTarget>, destinations: Vec<PlanDestination> },
    UploadGrant { job_id: String, fence: i64, book_uuid: String, primary_key: String, pdf_key: Option<String>, files: Vec<UploadFile> },
    UploadGranted { upload_id: String, sealed_token: String, expires_in_secs: u64 },
    ScanRequest { account: Option<String>, plugin_instance_id: Option<String> },
    ScanPlanned { created: Vec<String>, duplicates: Vec<String>, queue_full: u64 },
    JobStatus { job_ids: Vec<String> },
    JobStatuses { jobs: Vec<JobStatusEntry> },
    Wake { message: Value },
    Ack {  },
    NodeLeaving {  },
    CatalogBegin { share_books: bool, include_branches: bool },
    CatalogSession { fence: i64, cursor: String, ancestry: Vec<String>, destinations: Vec<String>, wanted: Vec<CatalogRef> },
    CatalogSync { fence: i64, batch: u64, cursor: String, done: bool, titles: Vec<CatalogTitle> },
    CatalogAccepted { batch: u64, upserted: u64, skipped: u64, wanted: Vec<CatalogRef> },
    CatalogUpload { fence: i64, store: String, product_id: String, files: Vec<UploadFile> },
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
  - `cluster_id`: Owner cluster id (also signed).
- `Auth`: Host → owner: proof of key possession.
  - `sig`: Ed25519 signature over [`crate::auth_transcript`](#auth_transcript), base64url
    without padding.
- `Welcome`: Owner → host: the handshake succeeded.
  - `node_id`: Authenticated node id.
  - `cluster_id`: Owner cluster id.
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
- `Claim`: Host → owner: pull one work item. Answered with [`Body::Work`](#body) or
  [`Body::NoWork`](#body).
  - `item`: [`ITEM_JOB`](#item_job) (deliveries are not served yet).
  - `class`: Job resource class; `network` when absent.
  - `filter`: The host's own narrowing: only jobs of these kinds whose every
    plugin its operator consented to. Absent claims everything the
    node is trusted with.
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
  consents to the job's plugin). The job returns to pending without
  spending the attempt. Answered with [`Body::Accepted`](#body) or
  [`Body::Lost`](#body).
  - `job_id`: Job id.
  - `fence`: Lease generation the host holds.
  - `reason`: Stable reason (`awaiting_host_consent`, ...), for the log.
  - `resumed`: The `resumed` flag of the job's `work` frame, so a resumed
    claim is given back resumable.
- `DeploymentsList`: Host → owner: the deployments that target this host and should run.
  Answered with [`Body::Deployments`](#body).
- `Deployments`: Owner → host: the answer to [`Body::DeploymentsList`](#body).
  - `items`: One entry per deployment, in deployment-id order.
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
- `ScanAccepted`: Owner → host: a results batch was written.
  - `batch`: The batch number.
  - `upserted`: Rows written.
- `AcquirePlan`: Host → owner: what an acquire job should fetch here. Answered with
  [`Body::AcquireTargets`](#body), [`Body::Lost`](#body), or `error`.
  - `job_id`: The acquire job.
  - `fence`: Lease generation the host holds.
- `AcquireTargets`: Owner → host: the titles to acquire and how to package them.
  - `targets`: Titles still to acquire, each one of the job's accounts.
  - `destinations`: Every output destination deployed on the answering owner node,
    the primary first, each with its own packaging and naming
    options, so the host names each destination's files as a local
    acquire would.
- `UploadGrant`: Host → owner: ask to upload one title's files. Answered with
  [`Body::UploadGranted`](#body), [`Body::Lost`](#body), or `error`.
  - `job_id`: The acquire job.
  - `fence`: Lease generation the host holds.
  - `book_uuid`: Library id of the title the files are for (one of the job's
    targets).
  - `primary_key`: Storage key of the title's primary audio file on the primary
    destination (one of `files`' copies there).
  - `pdf_key`: Storage key of the title's companion PDF on the primary
    destination (one of `files`' copies there), when one is
    uploaded.
  - `files`: Every file, in upload order. Empty, with an empty `book_uuid`,
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
- `ScanRequest`: Host → owner: queue a library scan on the owner cluster for the
  node's owner (H-5). The owner's scan planner runs as that user: an
  Owner or Administrator scanning every account gets one job per
  scan-enabled account, routed across the cluster's nodes and hosts;
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
- `NodeLeaving`: Host → owner: this host is leaving the cluster (its operator chose
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
  - `ancestry`: The library's ancestry: its own cluster id, then each library
    above it, up to the root.
  - `destinations`: The library node's output destination ids, the primary first;
    a catalog upload copies each file to every one under its key.
  - `wanted`: Titles recorded earlier whose files have not arrived yet.
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
  - `files`: The title's files, in upload order, each copied to every
    destination of the session under its manifest key.
- `Accepted`: Owner → host: the request with this frame id was applied.
- `Unknown`: A frame type this side does not know.

Implements: `Clone`, `Debug`, `Deserialize<'de>`, `Eq`, `PartialEq`, `Serialize`

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

## Functions

### is_ancestry

```rust
pub fn is_ancestry(ancestry: &[String]) -> bool
```

True when every id in `ancestry` is a plain cluster id and there are at
most [`MAX_ANCESTRY`](#max_ancestry).

### is_cluster_id

```rust
pub fn is_cluster_id(id: &str) -> bool
```

True for a plain cluster id in an ancestry.

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

### clerk_bootstrap_aad

```rust
pub fn clerk_bootstrap_aad(cluster_id: &str, node_id: &str) -> Vec<u8>
```

Associated data of a clerk bootstrap for `node_id` of `cluster_id`: the
domain tag, then each field as a 4-byte big-endian length and its bytes.

### open_clerk_bootstrap

```rust
pub fn open_clerk_bootstrap(
    recipient_secret: &[u8; 32],
    cluster_id: &str,
    node_id: &str,
    sealed: &str,
) -> Result<ClerkBootstrap, SealError>
```

Opens a bootstrap made by [`seal_clerk_bootstrap`](#seal_clerk_bootstrap) with the joining
node's box secret, for the cluster and node ids the redeem answered.

#### Errors

Returns [`SealError::Open`](#sealerror) when the box was sealed to another key, for
another cluster or node, or was altered, and [`SealError::Bundle`](#sealerror) when
its content is malformed or names other ids.

### seal_clerk_bootstrap

```rust
pub fn seal_clerk_bootstrap(
    recipient: &[u8],
    ephemeral_secret: &[u8; 32],
    bootstrap: &ClerkBootstrap,
) -> Result<String, SealError>
```

Seals `bootstrap` to the joining node's box key `recipient`, as
base64url text for the redeem answer.

#### Errors

Returns [`SealError::Key`](#sealerror) for an invalid recipient key and
[`SealError::Bundle`](#sealerror) for a bootstrap that is not version 1 or whose DEK
is not 32 bytes.

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

## Constants

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

Most cluster ids an ancestry may list.

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

### CLERK_BOOTSTRAP_AAD_DOMAIN

```rust
pub const CLERK_BOOTSTRAP_AAD_DOMAIN: &[u8] = b"bookclerk-node.v1 clerk-bootstrap";
```

Domain tag that starts the associated data of a clerk bootstrap.

### CLERK_BOOTSTRAP_VERSION

```rust
pub const CLERK_BOOTSTRAP_VERSION: u32 = 1;
```

Version of [`ClerkBootstrap`](#clerkbootstrap).

### CLERK_GRANTED_SCOPE

```rust
pub const CLERK_GRANTED_SCOPE: &str = "node:enroll node:clerk";
```

The scope a clerk code is granted (stored with the code at consent).

### CLERK_OFFER_PARAM

```rust
pub const CLERK_OFFER_PARAM: &str = "clerk_offer";
```

Authorize-request parameter by which a joining install offers to join as
a clerk (`clerk_offer=1`): it is empty, so its own data may be replaced.
Without it the consent page offers a branch only.

### DEK_LEN

```rust
pub const DEK_LEN: usize = 32;
```

Length of the data-encryption key.

### ITEM_JOB

```rust
pub const ITEM_JOB: &str = "job";
```

Work item kind for jobs.

### MAX_DESTINATION_ID_BYTES

```rust
pub const MAX_DESTINATION_ID_BYTES: usize = 64;
```

Longest destination id.

### MAX_SCAN_BATCH_ROWS

```rust
pub const MAX_SCAN_BATCH_ROWS: usize = 200;
```

Most rows one [`Body::ScanResults`](#body) batch may carry.

### MAX_SCAN_ROW_BYTES

```rust
pub const MAX_SCAN_ROW_BYTES: usize = _;
```

Largest serialized row in a [`Body::ScanResults`](#body) batch.

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
